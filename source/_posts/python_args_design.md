---
title: 设计 Python 函数参数的正确姿势
date: 2020/10/16
categories: 
- Python
tags:
- bug清除计划 
- python
---

   

### 问题介绍


不合理的函数参数设计，可能会给程序带来一些难以发现的bug，比如下面这个例子。

```
def hauntedfunc(queue=[]):
    queue.append(0)
    return queue
```

hauntedfunc 函数的功能很简单，目的就是向输入的 queue 中增加一个0元素。但是当我们调用
hauntedfunc 时，奇怪的现象出现了：

```
# 第一次调用，符合预期
>>> hauntedfunc(queue=[1])
[1, 0]
# 第二次调用，符合预期
>>> hauntedfunc()
[0]
# 第三次调用，不符合预期
>>> hauntedfunc()
[0, 0]
```
第三次调用的输出结果多了一个0元素！

---
### 原因

而这个 bug 的原因，是我们使用了可变类型作为参数的默认值，即 `queue=[]`。在 python 中, 函数参数的默认值被保存在了函数对象的 `.__defaults__` 属性中。当执行 hauntedfunc() 时, queue 变量其实只是默认参数 `[]` 的引用。我们可以用下面的脚本来验证：

```
>>> id(hauntedfunc.__defaults__[0])
1788363018056
>>> id(hauntedfunc())
1788363018056
```

所以在第二次调用后，hauntedfunc 实际上已经变成了:

```
def hauntedfunc(queue=[0]):
    queue.append(0)
    return queue
```

---
### 解决方案
为了避免这个 bug，我们可以把 hauntedfunc 函数做如下修改。由于 `None` 为不可变对象，所以自然不必担心函数的默认值被意外修改。

```
def hauntedfunc(queue=None):
    if queue is None:
        queue = []
    queue.append(0)
    return queue
```

这样做看起来没什么问题了，但是如果我们希望函数不要修改传入的参数，还需要对 hauntedfunc 做一些修改。具体地，可以通过 `list()` 创建传入参数的副本，后续操作均在副本上进行。这里要注意深拷贝浅拷贝的问题，具体细节就不再赘述了。

```
def hauntedfunc(queue=None):
    if queue is None:
        queue = []
    else:
        queue = list(queue)
    queue.append(0)
    print ('queue =', queue)
```
```
>>> constant_list = [1]
>>> hauntedfunc(queue=constant_list)
[1, 0]
>>> constant_list
[1]
```

