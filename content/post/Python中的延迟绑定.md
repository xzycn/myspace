---
title: Python中的延迟绑定
author: lordran
categories: Python
date: 2019-03-05 11:30:16
tags:
---
延迟绑定出现在闭包问题中。下面我们看一个闭包的例子：
```python
def gen_mul(n):
    def mul(x):
        return n*x
    
    return mul

double = gen_mul(2)
doubled_value = double(6) # 12
```
<!--more-->
可以看出满足闭包的几点：
- 有内部函数
- 内部函数引用了外部函数中的[自由变量](#注) 
- 内部函数被返回

闭包的优点：
- 可以避免使用全局变量 
- 可以持久化变量，达到静态变量的作用

闭包的缺点：
- 可能会消耗大量的内存

- 可能会导致内存泄漏

当然缺点可以通过人为避免。

现在我们来看看另一个会引出延迟绑定的例子：
```python
def multipliers():
    return [lambda x : i * x for i in range(4)]
print([m(2) for m in multipliers()]) # [6,6,6,6]
```
上边的例子会输出**[6,6,6,6]**,而不是我们预期的**[0,2,4,6]**。

这就是延迟绑定导致的结果。具体过程我们可以来分析下：
执行第三行时，会先执行**multipliers**函数，然后执行函数中的列表解析式。在每一次迭代的时候都会生成一个匿名函数(这里只是定义)作为元素。然后回到第三行，遍历返回的列表中的匿名函数，传入参数2并执行。此时函数类似于这样：
```python
def noname(x):
    return i * x
```
我们知道Python查找变量的作用域链的顺序依次为**[LEGB](#注)**：
> 局部变量(L)->外部函数中的局部变量(E)->全局变量(G)->内置变量(B)

非常重要的一点我们需要知道：**Python的作用域在编译时就已经形成了，而不是在运行时，函数的作用域与其被调用的位置无关。**

那么在本例中,上面的noname函数体中的i从何而来呢？当然首先会到multipliers函数的局部变量中去寻找。此时i的值已经为3，所以出现这种让人"费解"的现象。

那么现在我们既然已经知道了原因，那么要怎样解决呢？

我们可以将迭代的i值直接注入到匿名函数的函数体中，这里给出两种方法：
1. 通过为参数设置默认值，这是因为在编译时就会计算确定默认值：
```python
def multipliers_ch1():
    return [lambda m,x=i : m * x for i in range(4)]
```

2. 通过内置函数partial：
```python
from functools import partial
def multipliers_ch2():
    return [partial(lambda m,x : m * x,i) for i in range(4)]
```

3. 利用生成器的延迟计算：
```python
def multipliers_ch3():
    for m in range(4):
        yield lambda x: m * x
```
partial及生成器的内容会在以后分享。

运行结果
```python
print([m(2) for m in multipliers_ch1()]) # [0,2,4,6]
print([m(2) for m in multipliers_ch2()]) # [0,2,4,6]
print([m(2) for m in multipliers_ch3()]) # [0,2,4,6]
```
#### 注:
**自由变量**：指未在本地作用域中绑定的变量，我们可通过访问函数的**\_\_code\_\_**属性进行查看：
> fun.\_\_code\_\_.co_freevars

**LEGB**: 可看[该部分](https://stackoverflow.com/a/292502/4447404)解释