title: Python 循环导入(依赖)
tags:
  - Python
categories:
  - 自译文章
date: 2019-05-25 13:28:56
---
### 什么是循环依赖?
在两个或更多模块相互依赖的时候就可能会出现循环依赖。这是因为每个模块都是根据其他模块定义的(见图1)。

例如:
```python
functionA():  
    functionB()
```
和
```python
functionB():  
    functionA()
```


![upload successful](\images\pasted-3.png)(图1)

### 循环依赖的问题
循环依赖会使得你的代码问题多多。例如：可能在模块间产生紧密的耦合，随之就减少了代码的可复用性。从长远来看，这使得代码维护更加困难。

此外，循环依赖可能会成为潜在问题的根源，例如无限递归，内存泄漏，级联反应。如果你不小心在代码中制造了循环依赖，将会使得调试很多其造成的潜在问题变得更加困难。
<!--more-->
### 什么是循环导入？
循环导入是循环依赖的一种形式，通过Python中的import语句产生。

例如，让我们来分析下面的代码：
```python
# 模块1
import module2

def function1():  
    module2.function2()

def function3():  
    print('Goodbye, World!')
```

```python
# 模块2
import module1

def function2():  
    print('Hello, World!')
    module1.function3()
```

```python
# __init__.py

import module1

module1.function1()  
```

当Python导入模块的时候，会检查模块注册表确定该模块是否已经导入。如果模块已经被注册，则Python使用从缓存中取出已存在的模块对象。模块注册表是已经被初始化的模块表，同时通过模块名索引。这张模块表可以通过**sys.modules**访问。

如果模块还没有被注册，Python会寻找该模块，并在必要的时候初始化并在新的模块命名空间执行该模块。

在我们的例子中，当Python执行到**import module2**时，会加载并执行之。

问题发生在当**function2()**试着调用模块1中的**function3()**的时候。由于模块1先加载，接着在能够到达**function3()**之前加载模块2,所以在**function3**被调用的时候由于其还未被定义会抛出错误。
```python
$ python __init__.py
Hello, World!  
Traceback (most recent call last):  
  File "__init__.py", line 3, in <module>
    module1.function1()
  File "/Users/scott/projects/sandbox/python/circular-dep-test/module1/__init__.py", line 5, in function1
    module2.function2()
  File "/Users/scott/projects/sandbox/python/circular-dep-test/module2/__init__.py", line 6, in function2
    module1.function3()
AttributeError: 'module' object has no attribute 'function3' 
```

### 如何修复循环依赖
一般来说，循环导入是不良设计的产物。对程序作更深入的分析可以得到结论：依赖其实是不必的，或者被依赖的功能可以被移动到不会包含循环引用的模块中。

简单的做法是有时两个模块可以合并到一个简单，更大的模块。我们上面实例中的代码将会变成这样：
```python
# 模块1和2

def function1():  
    function2()

def function2():  
    print('Hello, World!')
    function3()

def function3():  
    print('Goodbye, World!')

function1()  
```

然而，合并后的模块可能有一些不相关的函数(强耦合)，并且在两个模块本来就含有大量代码的时候使得合并后的模块非常大。

如果前面的方法不能凑效，另一种方法就是延迟模块2的导入，在需要的时候才导入。可以通过将模块2的导入放到**function1()**中：
```python
# 模块 1

def function1():  
    import module2
    module2.function2()

def function3():  
    print('Goodbye, World!')
```

这样一来，Python就能够加载加载模块1中的所有函数，并且在需要的时候加载模块2。

这个方法不与Python语法相矛盾，正如[Python文档](https://docs.python.org/2/tutorial/modules.html)说道:"将所有导入语句放在模块开始是习惯用法，并不强制(或者脚本，也一样)"

Python文档同时也说到建议使用**import X**的形式，而非其他形式的导入语句，像**from module import ***,或者**from module import a,b,c**。

你可能会看到很多代码库中即使不存在循环依赖，仍然使用了延迟导入，这将加快启动速度，因此这并不被当做坏的实践(尽管这可能是坏的设计，但还是取决于你的项目实际情况)。

### 总结
循环导入是循环引用的一种特殊情况。通常来讲，这些问题可以通过更好的代码设计来避免。可是，有时最终的设计包含大量的代码，混杂着不相关的功能(强耦合)。