---
title: Mysql Innodb 事务隔离
tags:
  - Mysql
  - 事务隔离
categories:
  - 走在写码的路上
date: 2018-09-02 15:39:39
---


## 查看事务隔离级别

```mysql
# 查看当前事务隔离级别. Mysql默认 REPEATABLE READ
mysql> SELECT @@GLOBAL.tx_isolation, @@tx_isolation;
+-----------------------+-----------------+
| @@GLOBAL.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set, 2 warnings (0.00 sec)
```

<!-- more -->

## 事务隔离级别

### READ UNCOMMITTED

读未 commit 的脏数据

### READ COMMITTED

读所有已 commit 的数据, 即使已经处在事务中。 问题: 不可重复读

### REPEATABLE READ

一旦开始了事务, 并且读取了某一些数据, 不受其余事务对它们做的变更(update, insert, delete)影响. 还是能够重复读取这些数据。
但是仅限于 Selects， 对于 DML 它的表现和 READ COMMITTED 一样: 事务中Update后, 重新Select会读到update的数据在其他事务做的变更
问题: 幻读
```
mysql> select nickname from staff where id=1000026;
+-------------+
| nickname    |
+-------------+
| nickname    |
+-------------+
1 row in set (0.00 sec)

# 其余事务对nickname做修改

mysql> update staff set fullname= 'fullname' where id=1000026;
mysql> select nickname from staff where id=1000026;
+-----------------+
| nickname        |
+-----------------+
| 变化后的nickname |
+-----------------+
```

#### 幻读
[What You See Is NOT What You Write](https://blog.pythian.com/understanding-mysql-isolation-levels-repeatable-read/)

```mysql
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from repeatable_read;
+----+-------+
| id | text  |
+----+-------+
|  1 | text1 |
|  2 | text2 |
+----+-------+
2 rows in set (0.00 sec)

# modify text in another session

mysql> select * from repeatable_read;
+----+-------+
| id | text  |
+----+-------+
|  1 | text1 |
|  2 | text2 |
+----+-------+
2 rows in set (0.00 sec)

mysql> CREATE TABLE `repeatable_read_copy` (`id` int(11) NOT NULL AUTO_INCREMENT,   `text` varchar(32) DEFAULT NULL,   PRIMARY KEY (`id`) ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO `repeatable_read_copy` SELECT * FROM `repeatable_read`;
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from repeatable_read_copy;
+----+----------+
| id | text     |
+----+----------+
|  1 | modified |
|  2 | modified |
+----+----------+
2 rows in set (0.00 sec)

mysql> select * from repeatable_read;
+----+----------+
| id | text     |
+----+----------+
|  1 | modified |
|  2 | modified |
+----+----------+
2 rows in set (0.00 sec)
```

```mysql
# REPEATABLE READ
mysql> select * from repeatable_read_copy;
ERROR 1146 (42S02): Table 'falcon.repeatable_read_copy' doesn't exist

# create table in another session

mysql> select * from repeatable_read_copy;
ERROR 1412 (HY000): Table definition has changed, please retry transaction

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

# change to READ COMMITTED
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from repeatable_read_copy;
ERROR 1146 (42S02): Table 'falcon.repeatable_read_copy' doesn't exist

# create table in another session

mysql> select * from repeatable_read_copy;
+----+----------+
| id | text     |
+----+----------+
|  1 | modified |
|  2 | modified |
+----+----------+
2 rows in set (0.00 sec)
```

### SERIALIZABLE

```mysql
# 当前 session 事务隔离级别设置为 serializable
mysql> set session transaction isolation level serializable;
Query OK, 0 rows affected (0.00 sec)

mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> select nickname from staff where id=1000026;
+------------+
| nickname   |
+------------+
| 1800240708 |
+------------+
1 row in set (0.00 sec)

# 直接上锁
mysql> SELECT * FROM information_schema.INNODB_TRX;
+-----------------+-----------+---------------------+-----------------------+------------------+------------+---------------------+---------------------------------------------+---------------------+-------------------+-------------------+------------------+-----------------------+-----------------+-------------------+-------------------------+---------------------+-------------------+------------------------+----------------------------+---------------------------+---------------------------+------------------+----------------------------+
| trx_id          | trx_state | trx_started         | trx_requested_lock_id | trx_wait_started | trx_weight | trx_mysql_thread_id | trx_query                                   | trx_operation_state | trx_tables_in_use | trx_tables_locked | trx_lock_structs | trx_lock_memory_bytes | trx_rows_locked | trx_rows_modified | trx_concurrency_tickets | trx_isolation_level | trx_unique_checks | trx_foreign_key_checks | trx_last_foreign_key_error | trx_adaptive_hash_latched | trx_adaptive_hash_timeout | trx_is_read_only | trx_autocommit_non_locking |
+-----------------+-----------+---------------------+-----------------------+------------------+------------+---------------------+---------------------------------------------+---------------------+-------------------+-------------------+------------------+-----------------------+-----------------+-------------------+-------------------------+---------------------+-------------------+------------------------+----------------------------+---------------------------+---------------------------+------------------+----------------------------+
| 281479458690864 | RUNNING   | 2018-09-02 15:33:28 | NULL                  | NULL             |          2 |                  10 | SELECT * FROM information_schema.INNODB_TRX | NULL                |                 0 |                 1 |                2 |                  1136 |               1 |                 0 |                       0 | SERIALIZABLE        |                 1 |                      1 | NULL                       |                         0 |                         0 |                0 |                          0 |
+-----------------+-----------+---------------------+-----------------------+------------------+------------+---------------------+---------------------------------------------+---------------------+-------------------+-------------------+------------------+-----------------------+-----------------+-------------------+-------------------------+---------------------+-------------------+------------------------+----------------------------+---------------------------+---------------------------+------------------+----------------------------+
1 row in set (0.00 sec)
```