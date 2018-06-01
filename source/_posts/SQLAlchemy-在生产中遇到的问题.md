---
title: SQLAlchemy 在生产中遇到的问题
date: 2017-04-31 20:31:34
categories:
- 走在写码的路上
tags:
- Python
---

`StaleDataError` 和 `ObjectDeletedError` 这两个错误上了生产以后大量出现, 本质上差不多。抛出的异常里没有其他有用信息。

## ObjectDeletedError

### 文档

>A refresh operation failed to retrieve the database row corresponding to an object’s known primary key identity.
A refresh operation proceeds when an expired attribute is accessed on an object, or when [Query.get()
](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.get) is used to retrieve an object which is, upon retrieval, detected as expired. A SELECT is emitted for the target row based on primary key; if no row is returned, this exception is raised.
The true meaning of this exception is simply that no row exists for the primary key identifier associated with a persistent object. The row may have been deleted, or in some cases the primary key updated to a new value, outside of the ORM’s management of the target object.

其实说的很清楚, **A SELECT is emitted for the target row based on primary key; if no row is returned, this exception is raised.** 但是在成功复现之前就是没get到点， 也有被名字`Deleted`误导了的原因。

<!-- more -->

### 重现

1. session A 里拿到一个instance
2. `db.session.commit()` 使得instance在session A 过期
3. 在另一个 session 删掉 instance 这一条数据
4. 回到 session A, `print instance.id`

### 生产上的解释

回到生产上，根本原因是主从延迟问题
我们用 Proxy 实现读写分离
往主库写入后session commit, instance 过期， 去从库读不到，于是异常。

### 解决办法

1. 选项1是修改 expire_on_commit， 使得commit之后不过期， 但是， 心理上觉得不靠谱

```python
db = SQLAlchemy(session_options={'expire_on_commit': False})
```

2. 拿到 identity

> A refresh operation failed to retrieve the database row corresponding to an object’s known primary key identity.

大部分情况下我们需要的就是这个identity(所有表都有一列`id`作为 primary key)，dir 一下， 能找到`_sa_instance_state`, 或者

```python
from sqlalchemy import inspect
inspect(instance).identity
```

3. 合理的做法是指定走master

方案2反直觉, 但确实没有更快的办法

## StaleDataError

错误信息, 也是发生在commit的时候.

>StaleDataError: sqlalchemy.orm.persistence in _emit_update_statements
error
UPDATE statement on table 'device' expected to update 1 row(s); 0 were matched.

[文档](http://docs.sqlalchemy.org/en/latest/orm/exceptions.html#sqlalchemy.orm.exc.StaleDataError) 就不复制了
追一下 sqlalchemy.orm.persistence (赞一个sentry先)
```
if rows != len(records):
    raise orm_exc.StaleDataError
```

`records` 是前面传进来的要更新的数据,  `rows += c.rowcount`. c是connections.
本来想update 1 row， 但是被另一个进程捷足先登。 生产环境并发高的情况下才能重现。
查了celery的日志， 果然出错的task， 在相同时间有另一个参数相同uuid不同的task成功了。
怀疑过celery是不是重复出队， 又追了一下日志发现想多了， APP短时间重复请求了接口

留下两个疑问:
1. celery给task生成uuid是入队还是出队的时候
2. 生成随机数不现实的情况下， 怎么避免客户端重复提交带来的影响