---
title: 我知道和不知道的Python
tags:
  - Python
categories:
  - 走在写码的路上
date: 2018-08-24 00:11:31
---
## 继承

```python
class Init(object):
    def __init__(self, value):
        print 'Init.__init__'
        self.val = value
        print self.val


class Add2(Init):
    def __init__(self, val):
        print 'Add2.__init__'
        super(Add2, self).__init__(val)
        self.val += 2
        print self.val


class Mul5(Init):
    def __init__(self, val):
        print 'Mul5.__init__'
        super(Mul5, self).__init__(val)
        self.val *= 5
        print self.val


class Pro(Mul5, Add2):
    pass


class Incr(Pro):

    csup = super(Pro)
    # 但是这东西是什么鬼, 只是为了和下面`self.csup.__init__`拼成 `super(Pro, self).__init__(val)`?

    def __init__(self, val):
        self.csup.__init__(val)
        print('Incr.__init__')
        print(self.val)
        self.val += 1


print Pro(5).val

p = Incr(5)
print(p.val)


>>> Pro(5).val
Mul5.__init__
Add2.__init__
Init.__init__
5
7
35

>>> p = Incr(5)
Mul5.__init__
Add2.__init__
Init.__init__
5
7
35
Incr.__init__
35
>>> print(p.val)
36
```
这么打出来其实很清晰了.
毫无疑问先进入 `Mul5.__init__`, super进入`Add2.__init__`


-------
References:
1.[听说你会 Python ？](https://manjusaka.itscoder.com/2016/11/18/Someone-tell-me-that-you-think-Python-is-simple/)