---
title: Python 切片原理
date: 2020/10/18
categories: 
- Python
tags:
- 学习笔记
- python
---

《流畅的Python》第10.5小结介绍了切片原理，下文为简单总结。

以`List`对象为例，我们通常都会以如下方式进行切片：

```
>>> lst = [i for i in range(10)]
>>> lst[1:6:2]
[1, 3, 5]
```

那切片背后的原理是什么呢？

简单来说，python 会将`[start:stop:stride]`转化为`slice`类的实例，然后将该实例送入`__getitem__`函数中。在某些情况下，可以使用slice.indices()优雅地处理缺失索引和负数索引的情况，例如：

```
# 5为序列长度
>>> slice(-3, None, None).indices(5) 
(2, 5, 1)
```




