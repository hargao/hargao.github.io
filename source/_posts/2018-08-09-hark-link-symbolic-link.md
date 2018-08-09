---
title: Hark Link 和 Symbolic Link
date: 2018-08-09 22:26:19
categories:
  - 走在写码的路上
tags:
  - 基础
---

总忘. 记一下

### ls看一下

{% asset_img ls.png ls %}

<!-- more -->

### Linux 中的文件存储

{% asset_img inode.jpg inode %}

### 类比一下

- Hark: Python的引用.

    ```
    >>> a = [1,2]
    >>> b = a
    >>> del a
    >>> b
    [1, 2]
    ```
    b就是那个"hard link"

- Symbolic: windows 下的快捷方程式


-------
References:
[理解 Linux 的硬链接与软链接](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/)