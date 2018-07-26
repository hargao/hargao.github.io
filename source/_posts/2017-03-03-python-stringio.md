---
title: StringIO
date: 2017-03-03 23:25:42
categories:
- 走在写码的路上
tags:
- Python
---

开发环境
```
Python 2.7.6
Flask==0.10.1
requests==2.10.0
xlrd==0.9.3
```

又是一个上学没好好搞懂的地方， 从理清具体使用开始理清它。
io.StringIO 比 StringIO.StringIO 更新， 谷歌说StringIO.StringIO 拆分成  io.StringIO + io.BytesIO (大写的 ***TODO***), 选哪个要参考Python版本。

>io.StringIO is a class. It handles Unicode. It reflects the preferred Python 3 library structure.
>
>StringIO.StringIO is a class. It handles strings. It reflects the legacy Python 2 library structure.

<!-- more -->

##### 原始需求
用到它是因为有个需求让用户上传excel， 导入数据库
有另一个需求把数据整容成 excel 并发送给 用户/BC/零信
弄一个临时文件夹存有点蠢


##### stream读取上传的文件

```python
import io, csv

import xlrd
from flask import request

f = request.files['file']

# CSV
csv_stream = io.StringIO(f.stream.read().decode("UTF8"), newline=None)
reader = csv.reader(csv_stream)

# XLS
work_book = xlrd.open_workbook(file_contents=f.read())
```

##### 内存中生成XLS并POST出去

```python
import StringIO

import xlrd
import requests

def to_xls(data):
    workbook = xlwt.Workbook()
    sheet = workbook.add_sheet('sheel', cell_overwrite_ok=True)
    ....  # 往里写数据

    f = StringIO.StringIO()
    workbook.save(f)

    # read 前必须 seek(0) 设定文件指针， 或者用getValue
    f.seek(0)
    data = f.read()
    f.close()
    return data

if __name__ == '__main__':
    f = to_xls(data)
    file_name = 'xx.xls'
    files = {'file': (file_name, f)}
    requests.post(url, files=files)
```

用 io 模块会抛 unicode argument expected, got 'str' , 待理解..
