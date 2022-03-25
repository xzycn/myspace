---
title: Python 设计模式 - 单例模式
author: lordran
tags:
  - Python
  - 设计模式
categories:
  - 设计模式
date: 2019-03-05 17:10:02
---
单例模式是一种比较常见的设计模式，所以顺理成章的成为我们第一个介绍的设计模式。
应用场景：
- 资源共享。例如，数据库连接，配置管理，日志管理。


实现方法有多种：
<!-- more -->
- 使用**\__new__**

```python
class Singleton:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls) #这里需要注意不能使用cls(), 否则会导致无限递归
        
        return cls._instance
        
ins1 = Singleton()
ins2 = Singleton()
id(ins1) == id(ins2) # True

```

- 使用闭包/函数装饰器

闭包使得自由变量**instance**在**_singleton**调用完成后没有被释放，跟上诉方法中的类变量作用一样。

```python
from functools import wraps

def _singleton(cls):
    instance = {}
    
    @wraps(cls)
    def func(*args,**kwargs):
        if cls not in instance:
            instance[cls] = cls(*args,**kwargs)
        
        return instance[cls]
        
    return func

@_singleton
class Singleton:
    pass
    
ins1 = Singleton()
ins2 = Singleton()
id(ins1) == id(ins2) # True
```

- 使用类装饰器
是的装饰器除了函数也可以说是类，在类作为装饰器时，会调用类的**\_\_call__**方法

```python
class GenSingleton:

    def __init__(self, cls):
        """
        在编译时会调用该方法
        """
        self._cls = cls
        self._instance = {}

    def __call__(self, *args, **kwargs):
        """
        在实例化类时调用该方法
        """
        if self._cls not in self._instance:
            self._instance[self._cls] = self._cls(*args, **kwargs)

        return self._instance[self._cls]


@GenSingleton
class Singleton:
    pass


s1 = Singleton()
s2 = Singleton()
print(id(s1) == id(s2))  # True
```

- 使用metaclass

```python
class SingletonType(type):

    def __init__(cls, *args, **kwargs):
        """
        在编译时调用该方法
        """
        cls._instance = None

    def __call__(cls, *args, **kwargs):
        """
        在实例化类时调用该方法
        """
        if cls._instance is None:
            cls._instance = super().__call__(*args, **kwargs)

        return cls._instance


class Singleton(metaclass=SingletonType):
    pass


s1 = Singleton()
s2 = Singleton()
print(id(s1) == id(s2))  # True
```

对于Python引入机制熟悉的同学应该知道Python中的模块只会import一次，所以算是天然的单例模式。当然究其原因是在第一次import时会生成pyc文件，以后import时会直接读取pyc内容。

- 使用模块
    
singleton.py

```python
class Singleton:
    pass
    
s = Singleton()
    
```

usage.py

```python
from singleton import s as s1
from singleton import s as s2

print(id(s1) == id(s2)) # True
```

当然实现方式还有不少，但实质上差别不大，我们平时使用的时候掌握其中之一即可。
