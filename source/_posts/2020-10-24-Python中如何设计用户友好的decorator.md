---
title: Python中如何设计用户友好的decorator
tags:
  - Python
date: 2020-10-24 20:56:05
---


这篇文章假定读者对 python 中的装饰器有一定的了解。

我们先来设计一个最简单的 decorator：

```python
def f1(func):
    print('f1')
    return func

@f1
def f0(x):
    print('x:', x)

if __name__ == "__main__":
    f0('asdf')
```

当我们执行这个脚本的时候，我们很自然会得到下面的输出

```text
f1
x: asdf
```

这个时候，我们只能写`@f1`，而不能写`@f1()`。如果我们把`@f1`改成`@f1()`，会得到以下错误：

```text
TypeError: f1() missing 1 required positional argument: 'func'
```

因为被装饰的函数会被当成第一个参数（positional argument），而加上括号之后，相当于少了第一个参数。对于使用者来说，要记忆某个 decorator 是否要带括号是个不小的心智负担。你可能会想，在使用 python 标准库中的 dataclasses 时，`@dataclass`和`@dataclass()`都是合法的，为什么这里就不行了。其实想实现 dataclasses 类似的功能其实也很简单。我们只要判断 func 是否为 None，如果是，就返回一个新的 decorator。

```python
def f1(func=None):
    print('f1')
    return func


def f2(func=None):
    if func is None:
        return f1
    else:
        return f1(func=func)

@f2()
def f0(x):
    print('x:', x)
```

这样一切都正常了。
