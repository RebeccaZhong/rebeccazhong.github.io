
# 事务

# 什么是事务？

事务就是DBMS当中用户程序的任何一次执行，事务是DBMS能看到的基本修改单元。

事务是指对系统进行的一组操作，为了保证系统的完整性，事务需要具有ACID特性。即原子性（atomicity），一致性（consistency），隔离性（isolation），持久性（durability）。

## 原子性（atomicity）
一个事务必须视为一个不可分割的最小单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行成功其中的一部分操作，这就是事务的原子性。

## 一致性（consistency）
数据总是从一个一致性的状态转换到另一个一致性的状态。

## 隔离性（isolation）
一个事务所做的修改在最终提交以前，对其他事务是不可见的，多个并发事务之间是相互隔离的。关于事务的隔离性，MySQL提供了四种隔离级别：

1. Serializable（串行化）：可避免脏读、不可重复读、幻读的发生。（级别最高）
2. Repeatable-read（可重复读）：可避免脏读、不可重复读的发生。
3. Read-committed（读已提交）：可避免脏读的发生。
4. Read-uncommitted（读未提交）：最低级别，任何情况都无法保证。（级别最低）

以上四种隔离级别最高的是Serializable，最低的是Read uncommitted级别。当然，隔离级别越高，执行效率就越低。

MySQL数据库中默认的隔离级别为`Repeatable read`。

像Serializable这样的级别，就是以锁表的方式（类似于Java多线程中的锁）使得其他的线程只能在锁外等待，选用哪一种隔离级别应该根据实际情况而定。

查看当前事务的隔离级别：

```sql
SELECT @@tx_isolation;
```

临时设置事务的隔离级别：

```sql
SET TRANSACTION ISOLATION LEVEL 隔离级别名称;
或
SET tx_isolation='隔离级别名称';
```

注意： 设置数据库的隔离级别必须在开启事务之前！隔离级别的设置只对当前连接有效。

## 持久性（durability）
一旦事务提交，所做的修改会永久保存到数据库中。即使系统崩溃，修改的数据也不会丢失。

# MySQL中事务实现机制
MySQL提供了两种事务型的存储引擎：`InnoDB`和`NDB Cluster`（主要用于MySQL Cluster 分布式集群环境）。另外还有一些第三方存储引擎也支持事务，比较知名的包括`XtraDB`和`PBXT`。下面以`InnoDB`来说明。

## 事务日志
Redo、Undo、Checkpoint

# 锁
## 共享锁
由读表操作加上的锁，加锁后其他用户只能获取该表或行的共享锁，不能获取排它锁，也就是说只能读不能写

## 排它锁
由写表操作加上的锁，加锁后其他用户不能获取该表或行的任何锁，典型是mysql事务中

行锁: 对某行记录加上锁

表锁: 对整个表加上锁

# MVCC

# 修改事务隔离级别的方法

## 脏读

## 不可重复读

## 可重复读

## 幻读

# 参考地址

[MySQL的事务的四大特性和隔离级别](https://blog.csdn.net/lamp_yang_3533/article/details/79344736)

[MySQL高级 之 事务（ACID特性 与 隔离级别）](https://blog.csdn.net/wuseyukui/article/details/73549514)

[google搜索地址](https://www.google.com.hk/search?safe=strict&hl=zh-CN&ei=nCW0XP3CJ9XQ-gSY66V4&q=mysql+事务的特性和事务的隔离级别&oq=mysql+事务的特性和事务的隔离级别)
