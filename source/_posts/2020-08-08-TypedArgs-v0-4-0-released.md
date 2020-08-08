---
title: TypedArgs v0.4.0 released!
tags:
  - Python
  - Announcement
  - GitHub Action
date: 2020-08-08 16:15:46
---


[TypedArgs] v0.4.0发布了。大部分接口与v0.3.x没有差别，但是少了一个attribute，所以必须升一个minor version。这篇博客主要记录我的几个设计失误。

## `argparse.ArgumentParser`不能被pickle

TypedArgs内部是使用Python标准库`argparse`实现的。一个标准的`argparse`使用流程如下：

```python
import argparse

parser = argparse.ArgumentParser(prog='PROG')
parser.add_argument('data', help='path to data')
args = parser.parse_args(['data-path'])

assert args.data == 'data-path'
```

首先，`ArgumentParser`支持指定显示的program名，那[TypedArgs]也应该支持。那一个很自然的想法就是让parser成为`class TypedArgs`的一个attribute，这样解析arguments的时候就能自由访问。

```python
from dataclasses import dataclass, field
from typed_args import TypedArgs
import argparse

@dataclass
class Args(TypedArgs):
    parser: field(default_factory=lambda: argparse.ArgumentParser(prog='PROG'))
```

但是这带来了一个问题，我并没有意识到`ArgumentParser`是不能被pickle（序列化）的。虽然平时使用没有问题，但是到了多进程环境（比如PyTorch的`DistributedDataParallel`）就会报错。为此。v0.3.x临时打了一个patch，在解析完arguements之后，将`self.parser = None`。其实这个时候已经造成了API变动，minor version应该+1，但是当时思想出了问题，只升了patch version。当然，没有人会访问parser，所以理论上问题不大。

v0.4.0从根本上解决了这个问题，parser只在解析arguments被生成，解析完就回收，不再作为`TypedArgs`上的一个attribute。

## 如何让IDE正确识别出类型

PyCharm和VS Code对于类型推倒总是有不同的想法。PyCharm认为`=`后面的才是正确类型，直接忽略了我的annotation。VS Code就蠢一点，认为我的annotation是对的，但是使用的时候又自作聪明不显示我标注的类型。最后我采用了现在的方案：

```python
foo: str = func()

def func() -> Any:
    pass
```

这样写，两个IDE都能正确推导类型，而且也比较符合常识。

## 使用GitHub Action自动发布到[pypi.org]

GitHub很贴心，GitHub整个模板都给你弄好了，你只需要去`Settings > Secrets`填两个参数（`PYPI_USERNAME`和`PYPI_PASSWORD`）。之后每次push带tag，或者在GitHub上创建release的时候（创建release会创建一个tag），就会触发这个GitHub Action，将这个库发布到[pypi.org]。


[TypedArgs]: https://github.com/SunDoge/typed-args
[pypi.org]: https://pypi.org/