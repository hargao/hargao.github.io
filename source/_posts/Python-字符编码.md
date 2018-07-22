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

<!-- more -->

### str

即 byte string: `'\xe6\x88\x91\xe6\x98\xaf\xe8\xb0\x81'`
python2 的 str 是"字节串"

- ASCII
一共规定了128个字符的编码, 只支持英文, 与 unicode 无关

- GBK
中文在ASCII编码基础上做的增强, 兼容 ASCII 编码, 与 unicode 无关

- UTF-8
是对 unicode 编码存储的一种实现方式, 每次8个位传输数据; 一种变长的编码方式. 示例为十六进制表示.
8位 * 9 = 72 位。 一个汉字占3个字节
兼容ASCII的前128个符号。碰巧兼容GBK的字符和数值

str 有可能是 ascii, gbk, utf-8 等等中的任意一种。 具体是什么编码与 locale ，文件编码等等都有关系

```python
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

> If you have a UnicodeDecodeError it probably means the function is waiting for a string.
If you have a UnicodeEncodeError, it should be unicode.

```python
>>> u'\u6211\u662f\u8c01'.encode('UTF-8')
'\xe6\x88\x91\xe6\x98\xaf\xe8\xb0\x81'

>>>'我是谁'.decode('utf-8')
u'\u6211\u662f\u8c01'
```

## Python 3.x

记住用flask的时候, Everything from body is byte

Python 3.x 字符串默认 unicode, 3.x 的 str 就是 unicode

```python
>>> type('我是谁')
<class 'str'>

>>> '我是谁'.encode('utf-8')
b'\xe6\x88\x91\xe6\x98\xaf\xe8\xb0\x81'
>>> '我是谁'.encode('gbk')
b'\xce\xd2\xca\xc7\xcb\xad'
```

## sys.setdefaultencoding

这段写python2的都很熟悉了. 网上关于编码问题最常见的解决方案

```python
    import sys
    reload(sys)
    sys.setdefaultencoding('utf-8')
```

### 先看个例子

```python
>>> u'我' == '我'
__main__:1: UnicodeWarning: Unicode equal comparison failed to convert both arguments to Unicode - interpreting them as being unequal
False
>>> import sys
>>> reload(sys)
<module 'sys' (built-in)>
>>> sys.setdefaultencoding('utf-8')
>>> u'我' == '我'
True

>>> unicode('我')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 0: ordinal not in range(128)
>>> import sys
>>> reload(sys)
<module 'sys' (built-in)>
>>> sys.setdefaultencoding('utf-8')
>>> unicode('我')
u'\u6211'
>>>
```

### sys.setdefaultencoding 到底做了什么

[官方文档](https://docs.python.org/2/library/sys.html#sys.setdefaultencoding) 说这是给 site.py 用的. 用来在初始化的时候确定str编码

> sys.setdefaultencoding(name)
>
>Set the current default string encoding used by the Unicode implementation. This function is only intended to be used by the site module implementation and, where needed, by sitecustomize. Once used by the site module, it is removed from the sys module’s namespace.

### 会造成的问题

>  utf-8 encoded byte strings were mixed with unicode text strings

它把需要显式操作的str和unicode之间的转换变成隐式, 但是并没有找到因为重新设置编码导致诡异bug的具体案例.
这篇[文章](https://blog.ernest.me/post/python-setdefaultencoding-unicode-bytes)给了一个例子, 但我不太认可, 其实是同一个字符的两种写法？[这里](https://stackoverflow.com/questions/31076677/how-can-i-get-powershell-to-write-%C3%BE-lowercase-thorn-to-a-file-as-0xfe)
```
    >>> print u"Ã¾".encode("latin-1")
    þ
```

没有得到我想要的答案. 只能说, 不用显得专业...

## unicode_literals

声明当前模块所有的字符串都是unicode

### 为什么pocoo不建议用

看[这个](https://github.com/PythonCharmers/python-future/issues/22)和[这个](http://python-future.org/imports.html#should-i-import-unicode-literals)就好了, 可能会有下面这些问题
> - breaking pydoc because of unicode docstrings
- Django breaking filesystem access due to accidentally using unicode paths
- porting things to Py3
- ...

-----------------------
References:
- https://segmentfault.com/q/1010000007215946
- [Proposal to not use unicode_literals](https://github.com/PythonCharmers/python-future/issues/22)
- [sys.setdefaultencoding is evil](https://ziade.org/2008/01/08/syssetdefaultencoding-is-evil/)
- [Why sys.setdefaultencoding() will break code](https://anonbadger.wordpress.com/2015/06/16/why-sys-setdefaultencoding-will-break-code/)
- http://python-future.org/imports.html#should-i-import-unicode-literals