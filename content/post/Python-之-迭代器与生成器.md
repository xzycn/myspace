---
title: Python 之 迭代器与生成器
author: lordran
date: 2019-03-12 19:41:40
tags:
categories:
 - Python
---
Python中的迭代器和生成器是对于初学者来说很容易混淆，更说不出两者的区别。
当然这也是面试中的常考内容，我今天将要详细介绍下它们。

### 迭代器

Python中的对象之所以能迭代是因为对象定义(或继承)了**\_\_iter__**或**\_\_getitem__**。
<!-- more -->
在需要迭代对象的时候，解释器会首先调用iter函数，iter的执行过程为：
- 查找对象是否存在**\_\_iter__**方法, 如果存在则调用，如果**\_\_iter__**返回的值是迭代器对象（Iterator），则直接返回，反之则报错,例如：
> TypeError: iter() returned non-iterator of type 'int'
- 如果没有实现**\_\_iter__**方法,则会查看是否实现了**\_\_getitem__**方法，如果实现，则会生成一个迭代器,并返回。该迭代器在迭代时会尝试通过索引（从0开始）获取元素，直到抛出IndexError。
- 如果两个方法都没有实现，则会抛出异常（其中**Iter**为对象所属的类）:
> TypeError: 'Iter' object is not iterable

由此可以看出：**迭代器是从可迭代对象中获取的。**
我们需要知道，**可迭代和迭代器是不一样的。**这可根据官方源码看出：

```python
def _check_methods(C, *methods):
    mro = C.__mro__
    for method in methods:
        for B in mro:
            if method in B.__dict__:
                if B.__dict__[method] is None:
                    return NotImplemented
                break
        else:
            return NotImplemented
    return True

class Iterable(metaclass=ABCMeta):

    __slots__ = ()

    @abstractmethod
    def __iter__(self):
        while False:
            yield None

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterable:
            return _check_methods(C, "__iter__")
        return NotImplemented


class Iterator(Iterable):

    __slots__ = ()

    @abstractmethod
    def __next__(self):
        'Return the next item from the iterator. When exhausted, raise StopIteration'
        raise StopIteration

    def __iter__(self):
        return self

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            return _check_methods(C, '__iter__', '__next__')
        return NotImplemented
```

可以看出**Iterator**需要实现**\_\_iter__**和**\_\_next__**，而**Iterable**只需要实现**\_\_iter__**就行。

内置类型（str，dict，list，tuple，set）的实例对象只是**Iterable**，却不算是**Iterator**。

例如：
```python
from collections.abc import Iterable,Iterator

print(isinstance("xzyl", Iterable)) # True
print(isinstance("xzyl", Iterator)) # False

class It1:
    def __iter__(self):
        pass


class It2:
    def __iter__(self):
        pass
        
    def __next__(self):
        pass
        
print(isinstance(It1(), Iterable)) # True
print(isinstance(It2(), Iterable)) # True
print(isinstance(It1(), Iterator)) # False
print(isinstance(It2(), Iterator)) # True

```

从上面的信息来看，判断对象是否可迭代的最准确的方法是通过调用**iter**方法然后捕获**TypeError**了，因为实现序列协议**\_\_getitem__**的对象也是可迭代的。并且定义了**\_\_iter__**方法也不一定就表示该对象可迭代了，例如返回一个整数或字符串等。

虽然可迭代对象可以定义为迭代器，但是我们不推荐这样做，后面会分享设计模式中的迭代器模式，来说明好的设计为何不应该这样做。


### 生成器

在了解什么是生成器之前，我们先了解下生成器函数：
```python
def counter():
    for i in range(1,11):
        yield i
        
print(counter) # <function __main__.counter()>
print(counter()) # <generator object counter at 0x000001B6BB7D2660>

```
正如上面看到的，函数体中有yield关键字的函数即为生成器函数。生成器函数自身跟普通函数没有区别，但是在调用时会返回一个生成器对象，是生成器对象的生产工厂。

生成器对象同样是迭代器对象。
```python
print(isinstance(counter, Iterator)) # True
```

生成器同时拥有延迟计算的特点：
```python
def counter():
    for i in range(1,3):
        print(i)
        yield i

c = counter()
print(next()) # 1
print(next()) # 2
print(next()) # raise StopIteration

```

由于生成器延迟计算的特点，可以在读取大数据时节省很大内存。

### 区别

个人总结两者的区别有：
- 两者的生成方式不同

迭代器对象是实现了迭代器协议(**\_\_iter__**和**\_\_next__**)的对象，而生成器对象则是通过调用生成器函数(含**yield**关键字)或者生成器推导式生成

- 生成器一定是迭代器，但迭代器不一定是生成器（生成器是迭代器的子类）

生成器同样实现了迭代器协议

- 两者的数据源不一样

迭代器需要确定的原始数据，而生成器可以不需要数据（如果你想，你可以一直在函数中yield值）。

- 生成器有send方法，而迭代器不一定有

总体来说在以遍历为目的的时候，两者并无太大区别，但是生成器写法更加简洁。

生成器更加重要的作用是在协程部分，以后将会分享。