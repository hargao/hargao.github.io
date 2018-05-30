---
title: Python 字符编码
date: 2017-04-16 00:00:00
categories:
- 走在写码的路上
tags:
- Python
---


## Python 2.x

### Unicode

即 unicode string: `u'\u6211\u662f\u8c01'`
统一两个字节、也就是16位来表示一个字符. 这是它的十六进制表示. 对应到字符.但是没有定义如何存储.

### str

即 byte string: `'\xe6\x88\x91\xe6\x98\xaf\xe8\xb0\x81'`

- ASCII
一共规定了128个字符的编码, 只支持英文, 与 unicode 无关

- GBK
中文在ASCII编码基础上做的增强, 兼容 ASCII 编码, 与 unicode 无关

- UTF-8
是对 unicode 编码存储的一种实现方式, 每次8个位传输数据; 一种变长的编码方式. 示例为十六进制表示.
8位 * 9 = 72 位。 一个汉字占3个字节
兼容ASCII的前128个符号。碰巧兼容GBK的字符和数值

str 有可能是 ascii, gbk, utf-8 等等中的任意一种。 具体是什么编码与 locale ，文件编码等等都有关系

``` Python
>>> "\x3cdiv\x3e".encode('ASCII')
'<div>'

>>> u'。'  # 中文句号
u'\u3002'
>>> u'。'.encode('gbk')
'\xa1\xa3'
>>> u'。'.encode('utf-8')
'\xe3\x80\x82'
>>> '。'
'\xe3\x80\x82'
>>> u'。'.encode('ASCII')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode character u'\u3002' in position 0: ordinal not in range(128)
```

### `# -*-coding:utf-8 -*-`
告诉 python 编译器如何解码代码文件

### decode & encode
    ``` Python
    >>> u'\u6211\u662f\u8c01'.encode('UTF-8')
    '\xe6\x88\x91\xe6\x98\xaf\xe8\xb0\x81'

    >>>'我是谁'.decode('utf-8')
    u'\u6211\u662f\u8c01'
    ```

## Python 3.x

记住用flask的时候, Everything from body is byte

Python 3.x 字符串默认 unicode, 3.x 的 str 就是 unicode

``` Python
>>> type('我是谁')
<class 'str'>

>>> '我是谁'.encode('utf-8')
b'\xe6\x88\x91\xe6\x98\xaf\xe8\xb0\x81'
>>> '我是谁'.encode('gbk')
b'\xce\xd2\xca\xc7\xcb\xad'
```

-----------------------
References:
- https://segmentfault.com/q/1010000007215946