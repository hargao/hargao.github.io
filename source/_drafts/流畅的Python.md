---
title: 流畅的Python
categories:
- 读书笔记
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


- 扁平序列: 存放值而不是引用
  - str
  - bytes
  - bytearray


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

