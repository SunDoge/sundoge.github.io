---
title: 'TensorFlow Datasets: The Bad Parts'
date: 2020-08-02 13:38:08
tags:
  - TensorFlow
  - Deep Learning
---

本文是[https://determined.ai/blog/tf-dataset-the-bad-parts/](https://determined.ai/blog/tf-dataset-the-bad-parts/)的译文。我在尝试用TensorFlow参照PyTorch代码复现一些模型的时候也产生了同样的想法。本文仅供个人研究使用。

***

**TLDR**: TensorFlow的`tf.data` API是一个常用的将数据加载进深度模型的方法。虽然`tf.data`有很多强大的功能，但是它是围绕顺序读取数据集设计的。这个设计导致它无法高效实现一些功能：shuffle大数据集，分布式训练时对数据进行分片，实现容错训练（fault-tolerant training）。我们认为，在构建深度学习数据读取API时，随机访问应该是需要考虑的关键因素。

在构建端到端的企业深度学习平台时，工程团队可能会遇到[许多陷阱](https://determined.ai/blog/building-an-enterprise-deep-learning-platform/)。最常见的问题之一涉及数据加载。训练期间的数据加载常常被忽略，但是它可能对吞吐量产生巨大影响。机器学习框架提供了一些抽象，试图让数据加载变得简单直接。但是，在看似简单接口背后可能隐藏了一些令人惊讶的问题。在这篇文章中，我们将带你了解一个常见的数据加载API：[TensorFlow Datasets](https://www.tensorflow.org/api_docs/python/tf/data/Dataset)。


![image.png](https://i.loli.net/2020/08/02/eKtHoD4mk1CyPsr.png)


## Data Loader Patterns

数据加载接口可以使用两种基本模式，**随机访问（random access）**和**循序访问（sequential access）**。

### Random Access

随机访问是指高效访问数据集中任何元素的能力。在 Python 中，随机访问通常通过对一个列表进行索引来完成（i.e., `data[index]`） ，背后实际调用了`__getitem__()`。PyTorch 使用这种方法来定义映射样式（map-style）的数据集接口（上面实现的那种）。随机访问数据加载器接口还可能要求用户指定数据集的整个长度（`__len__()`）。

```python
import torch.utils.data

class RandomAccessDataset(torch.utils.data.Dataset):
    def __init__(self, data: List) -> None:
        self.data = data

    def __len__(self) -> int:
        return len(self.data)

    def __getitem__(self, index: int): -> Any:
        return self.data[index]
```

支持随机访问的深度学习数据 api 包括`tfkeras.utils.Sequence`和`torch.utils.data.Dataset` (Map Style)。

### Sequential Access

顺序读取是一种元素必须按预定顺序访问的模式，通常是通过迭代器访问。在Python中，顺序访问通常是通过迭代器和`yield`表达式实现的。

```python
def sequential_dataset(data: List) -> Iterator:
    for item in data:
        yield item
```

一些深度学习框架，比如早期版本的keras，原生支持将数据输入管道（data input pipeline）表示为Python生成器。类似地，TensorFlow Datasets是围绕顺序数据访问构建的。 Python 生成器转换为 TensorFlow Dataset 非常简单，虽然有点啰嗦：

```python
import itertools

def gen():
    for i in itertools.count(1):
        yield (i, [1] * i)

dataset = tf.data.Dataset.from_generator(
     gen,
     (tf.int64, tf.int64),
     (tf.TensorShape([]), tf.TensorShape([None])))
```

在下一节中，我们将讨论使用循序访问作为数据加载器的缺点。

## Sequential Access in TensorFlow Datasets

TensorFlow 的 `tf.data` API 以一种优雅的方式简化了数据加载代码的结构：你可以使用lazy initialization的方式将一系列操作串起来。`tf.data` 还提供了helper API 来完成[预读取](https://www.tensorflow.org/api_docs/python/tf/data/Dataset#prefetch)和[并行数据加载](https://www.tensorflow.org/guide/data_performance#parallelizing_data_extraction)等常见任务。

```python
import tensorflow as tf


dataset = tf.data.Dataset.from_tensor_slices([1,2,3])

for element in dataset:
   print (element)
>>> tf.Tensor(1, shape=(), dtype=int32)
>>> tf.Tensor(2, shape=(), dtype=int32)
>>> tf.Tensor(3, shape=(), dtype=int32)

dataset = dataset.map(lambda x: x*2)
for element in dataset:
   print (element)
>>> tf.Tensor(2, shape=(), dtype=int32)
>>> tf.Tensor(4, shape=(), dtype=int32)
>>> tf.Tensor(6, shape=(), dtype=int32)
```

然而，**TensorFlow Dataset 基本上是围绕顺序访问构建的**：`tf.data` pipeline中的每一个操作，都会迭代他的输入并生成一个顺序的输出流，供下一个操作使用。这个API不支持随机访问，导致在尝试实现一些常见的机器学习工作流时会出现一些重大问题。

```python
dataset[0]
>>> TypeError: 'TensorSliceDataset' object does not support indexing

list(dataset.as_numpy_iterator())
>>> [1,2,3]
```

### Data Shuffling

在训练一个深度学习模型时，训练集通常在输入模型钱被shuffle，这个操作通常会提升泛化性能。如果我们的数据API只支持顺序访问，要怎样实现random shuffling呢？一个简单但是低效的方法是将尽可能多的数据读入内存并在内存里面shuffle。实际上，这正是`tf.data`的shuffle的做法！

> This dataset fills a buffer with `buffer_size` elements, then randomly samples elements from this buffer, replacing the selected elements with new elements. For perfect shuffling, a buffer size greater than or equal to the full size of the dataset is required.

对于不能完全放入内存的数据集（深度学习中最常见的情况），**`shuffle()`实际上不会shuffle整个数据集**！这意味着`shuffle()`在大多数应用程序中没有达到预期的效果。包括我们在内的很多从业人员都犯了这个错误，并且看到他们模型的泛化性能因此收到影响。虽然可以通过提前将数据读进内存或shuffle文件名列表来实现shuffle整个数据集，但是很多用户可能没有意识到这个他们的代码中存在这个问题！

### Data Sharding

在进行 data-parallel distributed training 时，每一个worker（通常是GPU）要对每个batch的一个部分（或者叫`shard`）进行训练。为了处理这个常见的任务，`tf.data`提供了一个看起来完美契合我们要求的方法：[`shard(n, i)`](https://www.tensorflow.org/api_docs/python/tf/data/Dataset#shard)将数据分为n个shards，并返回第i个shard，以便在当前worker中继续处理。

不幸的是，这里存在一个陷阱：TODO