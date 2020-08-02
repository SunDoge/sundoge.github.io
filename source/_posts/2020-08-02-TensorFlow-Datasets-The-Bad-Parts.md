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

TODO