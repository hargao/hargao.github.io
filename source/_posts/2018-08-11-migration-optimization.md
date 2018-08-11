---
title: Migration 优化
date: 2018-08-11 16:15:30
tags:
  - Alembic
  - Migration
  - Python
categories:
  - 走在写码的路上
---


Flask-Migrate 作为 Flask 最常使用的 migration 工具, 默认行为有很多不符合我们的期望. 记录历次优化所做的变更.

## 依赖

```
Python 2.7.14
Mysql 5.7.21
flask_migrate == 2.0.3
```

<!-- more -->

## 文件

`./migrations/env.py`

## 优化

### 检测类型变化

默认不检测类型变化. 增加`compare_type=True`到 `run_migrations_online` 函数的 `context.configure`

```python
context.configure(
    connection=connection,
    target_metadata=target_metadata,
    process_revision_directives=process_revision_directives,
    compare_type=True,
    include_object=include_object,
    **current_app.extensions['migrate'].configure_args)
```

### 指定 column 的 charset 和 collation

常用于数据库默认charset为utf8, 需要指定列为utf8mb4的情况下. 但是更建议全部使用utf8mb4

```python
content = db.Column(TEXT(charset='utf8mb4', collation='utf8mb4_unicode_ci'))
```

### 处理 Mysql 中 Boolean 和 Tinyint

boolean 在 Mysql 中是 tinyint(1) 的 alias. 所以每次SQLAlchemy都会检测出类型变化. 很是困扰
参考这两个 issue [1](https://bitbucket.org/zzzeek/alembic/issues/46/mysqltinyint-display_width-1-vs-saboolean) [2](https://bitbucket.org/zzzeek/alembic_moved_from_hg_to_git/issues/84/alembic-tries-to-convert-field-from)

```python
from sqlalchemy import Boolean
from sqlalchemy.dialects import mysql

def my_compare_type(context, inspected_column, metadata_column, inspected_type, metadata_type):
    # return False if the metadata_type is the same as the inspected_type
    # or None to allow the default implementation to compare these
    # types. a return value of True means the two types do not
    # match and should result in a type change operation.
    if isinstance(inspected_type, mysql.TINYINT) and isinstance(metadata_type, Boolean):
        return False
    return None

context.configure(
    # ...
    compare_type = my_compare_type
)
```

### 排除某个 Bind

主库之后为了读数据 bind 了另一数据库. 只读不写, 而且只读部分表, 并没有定义所有model

```python
def include_object(object, name, type_, reflected, compare_to):
    if type_ == "table" and object.info.get('bind_key') == 'exclude_database':
        return False
    else:
        return True

context.configure(
    # ...
    include_object=include_object
)
```

### Add column 时支持 AFTER

参考了 [google forum](https://groups.google.com/forum/#!msg/sqlalchemy-alembic/izYq2EMYotI/LzV7y8jOssIJ) 和 Alembic AddColumn 的实现

```python
# env.py
from sqlalchemy.ext.compiler import compiles
from alembic.ddl.base import AddColumn, alter_table, add_column


@compiles(AddColumn, 'mysql')
def custom_add_column(element, compiler, **kw):
    text = "%s %s" % (
        alter_table(compiler, element.table_name, element.schema),
        add_column(compiler, element.column, **kw)
    )
    if "after" in element.column.info:
        text += " AFTER %s" % element.column.info['after']

    return text
```

```python
# migration 中手动指定 after
op.add_column(
        'test',
        sa.Column('col2', mysql.VARCHAR(length=32), nullable=True, info={"after": "col1"}))
```

写起来还是挺头疼, 需要调整一遍顺序.

## 待优化

- 不能自动按照在 class 定义的顺序生成 after
- 不支持 online ddl. 自动加上 `ALGORITHM=INPLACE, LOCK=None`
- CHARSET 混乱. 希望生成migration时能自动指定

-------
References:
- [Altering the behavior of AddColumn](https://groups.google.com/forum/#!msg/sqlalchemy-alembic/izYq2EMYotI/LzV7y8jOssIJ)
- [Changing the default compilation of existing constructs](http://docs.sqlalchemy.org/en/latest/core/compiler.html#changing-the-default-compilation-of-existing-constructs)
- [mysql.TINYINT(display_width=1) vs sa.Boolean()](https://bitbucket.org/zzzeek/alembic/issues/46/mysqltinyint-display_width-1-vs-saboolean)
- [alembic tries to convert field from TinyInt to Boolean on MySQL](https://bitbucket.org/zzzeek/alembic_moved_from_hg_to_git/issues/84/alembic-tries-to-convert-field-from)
- [Auto Generating Migrations](http://alembic.zzzcomputing.com/en/latest/autogenerate.html#comparing-types)