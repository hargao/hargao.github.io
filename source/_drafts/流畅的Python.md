---
title: 流畅的Python
categories:
- 走在写码的路上
tags:
- Python
---

> *不成熟的抽象和过早的优化一样, 都会坏事.*


### 第一章 数据模型(Data Model/Object Model)

> *不能让特例特殊到开始破坏既定规则.*

<!-- more -->

### 第二章 序列(Sequences)

#### 内置序列

根据存放的数据类型分类:

- 容器序列: 存放对象的引用
  - list
  - tuple
  - collection.deque


- 扁平序列: 存放同一种值而不是引用, 只能存放字符、字节和数值这种基础类型
  - str
  - bytes
  - bytearray
  - array.array


根据能否被修改分类:

- 可变序列
  - list
  - bytearray
  - array.array
  - collections.deque


- 不可变序列
  - tuple
  - str
  - bytes


#### 列表(List)推导和生成器表达式

Python3中， 列表推导不再有变量泄漏的问题。(当然for循环还是那个样子)

```python
>>> x = 'ABC'
>>> [x for x in range(10)]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> x
'ABC'
```

生成器表达式:
方括号换成圆括号就成为一个生成器表达式
可以省去额外内存占用以及省去一遍for循环的开销

```python
>>> (x for x in range(10))
<generator object <genexpr> at 0x1043d99b0>
```

