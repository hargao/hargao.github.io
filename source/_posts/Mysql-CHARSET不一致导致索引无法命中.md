---
title: 'Mysql: CHARSET不一致导致索引无法命中'
date: 2017-02-07 21:17:40
tags:
---

### 原始需求描述
- TABLE task,  200w rows

|     id     | task_no |  content  |  created  | house_code |
| --------- | ----------- | :---------: | :----------: | -----------------|
| 自增主键 | task编码，uni_key | 内容 | 创建时间, datetime | house.code, 无外键 |

- TABLE house, 160w rows

|       id       |    code    |    name    |   created   |
| ------------ | :----------: | :-----------: | ------------- |
| 自增主键 | house编码，uni_key, 有单列索引 |    名称    | 删除状态， 单列索引 |

运营要求导出一定时间段, 1万左右的task, 字段包括 task.content 和 house.name,没有 house_code 的保留 null, 半小时都没跑出来.

<!-- more -->

### SQL写的烂?

```
SELECT t.task_no, t.content, t.house_code, house.name
FROM task t
LEFT JOIN house ON house.code = t.house_code AND house.deleted = 0
WHERE t.id >= 94210385 AND t.id <= 94860373
AND t.created > '2015-12-17'AND t.created < '2015-01-11'
```

explain 看看
{% asset_img explain-1.png explain1 %}

id只起到缩减范围的作用, house.deleted 删掉对效率影响不大。
本来很简单的数据， 最后让运营等了三个小时, 还使用了迂回办法: 子查询， 把有 house_code 和 无 house_code 分开导出。
但是为什么, 这句SQL执行效率还不如写代码一行行 `select name from house where house_code=x`, 不能接受。

### 尝试理解

缩减范围, 6个task

```
SELECT t.task_no, t.content, t.house_code, house.name
FROM task t
LEFT JOIN house
ON house.code = t.house_code
WHERE t.id >= 594210385 AND t.id <= 594210390;
```

又 EXPLAIN
{% asset_img explain-2.png explain2 %}
**taking 3.02s**, 服气

现在有两个问题
1. house.code 索引用不上。
2. 我们明确知道, house_code 在 house 最多只有一条, 找到一条就不需要继续了。

补充一个问题, `LEFT JOIN` -> `JOIN`, 即

```
SELECT t.task_no, t.content, t.house_code, house.name
FROM task t
JOIN house
ON house.code = t.house_code
WHERE t.id >= 594210385 AND t.id <= 594210390;
```
EXPLAIN 出来， 主表变了(又， 排序麻烦在主表上做)
{% asset_img explain-3.png explain3 %}

除此之外， 效率几乎无差

### 索引为什么用不上

找不同发现， 两个表 `CHARSET` 不同， 分别是 `utf8mb4` 和 `utf8`
两个字段 `TYPE` 也不同， `VARCHAR(32)` 和 `VARCHAR(64)`
实践证明， 两列都改成`utf8mb4`以后， 就好了，好了， 好了.....
人生无处不是坑, 下回就懂了, 嗯
`TYPE` 长度不影响， 顺手改了吧

还是 EXPLAIN
{% asset_img explain-4.png explain4 %}

终于长得像样了些
**taking 1.5ms**

既然这么快， 问题 2 就没有必要了

### 收工

[收工之前， 多谢这位大哥](http://stackoverflow.com/questions/18660252/mysql-why-does-left-join-not-use-an-index)
刚搜到的时候, 信心满满, "我们肯定不会犯这种错误"
打脸了

### 后续

给同事演示, 发现主表`utf8` join `utf8mb4`可用索引， 反之则不行。
用STRAIGHT_JOIN指定join顺序。
同一个类型， 长度不同不影响使用索引。
[更多参考](https://github.com/Yhzhtk/note/issues/43)
