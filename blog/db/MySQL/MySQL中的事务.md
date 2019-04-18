
# 事务

## 什么是事务？

事务就是DBMS当中用户程序的任何一次执行，事务是DBMS能看到的基本修改单元。

## 事务的特性

事务是指对系统进行的一组操作，为了保证系统的完整性，事务需要具有ACID特性。即原子性（atomicity），一致性（consistency），隔离性（isolation），持久性（durability）。

1. 原子性（atomicity）
一个事务必须视为一个不可分割的最小单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行成功其中的一部分操作，这就是事务的原子性。

2. 一致性（consistency）
数据总是从一个一致性的状态转换到另一个一致性的状态。

3. 隔离性（isolation）
一个事务所做的修改在最终提交以前，对其他事务是不可见的，多个并发事务之间是相互隔离的。

4. 持久性（durability）
一旦事务提交，所做的修改会永久保存到数据库中。即使系统崩溃，修改的数据也不会丢失。

## 声明事务
### 自动提交事务
### 显式事务
### 隐式事务
### 分布式事务

## 事务并发的四种隔离级别

关于事务的隔离性，MySQL提供了四种隔离级别：

### 1. Serializable（串行化）
可避免脏读、不可重复读、幻读的发生。（级别最高）

### 2. Repeatable-read（可重复读）
可避免脏读、不可重复读的发生。

### 3. Read-committed（读已提交）
可避免脏读的发生。
### 4. Read-uncommitted（读未提交）
最低级别，任何情况都无法保证。（级别最低）

以上四种隔离级别最高的是Serializable，最低的是Read uncommitted级别。当然，隔离级别越高，执行效率就越低。

MySQL数据库中默认的隔离级别为`Repeatable read`。

像Serializable这样的级别，就是以锁表的方式（类似于Java多线程中的锁）使得其他的线程只能在锁外等待，选用哪一种隔离级别应该根据实际情况而定。




### 事务并发带来的问题

脏读、不可重复读、幻读

**不同隔离级别对应的并发问题**
隔离级别| 脏读（Dirty Read）| 不可重复读（NonRepeatable Read） | 幻读（Phantom Read）
----|----|----|---
未提交读（Read uncommitted）| 可能 | 可能 | 可能
提交读（Read committed） | 不可能 | 可能 | 可能
可重复读（Repeatable read） | 不可能 | 不可能 | 可能
串行读（Serializable ） | 不可能 | 不可能 | 不可能


#### 脏读

#### 不可重复读

#### 幻读

### 查看当前事务的隔离级别

```sql
SELECT @@tx_isolation;
```

### 设置事务的隔离级别

1. 临时设置事务的隔离级别

```sql
SET TRANSACTION ISOLATION LEVEL 隔离级别名称;
或
SET tx_isolation='隔离级别名称';
```

注意： 设置数据库的隔离级别必须在开启事务之前！隔离级别的设置只对当前连接有效。

## 事务隔离级别验证

### 验证不同会话的隔离级别
1. 验证Read-uncommitted隔离级别
1. 验证Read-committed隔离级别
1. 验证Repeatable-read隔离级别
1. 验证Serializable隔离级别

## MySQL中事务实现机制
MySQL提供了两种事务型的存储引擎：`InnoDB`和`NDB Cluster`（主要用于MySQL Cluster 分布式集群环境）。另外还有一些第三方存储引擎也支持事务，比较知名的包括`XtraDB`和`PBXT`。下面以`InnoDB`来说明。

### 事务日志
Redo、Undo、Checkpoint

## MVCC
MVCC(multiple-version-concurrency-control）是个行级锁的变种，
它在普通读情况下避免了加锁操作，因此开销更低。


# 锁
## 锁级别
### 共享锁
由读表操作加上的锁，加锁后其他用户只能获取该表或行的共享锁，不能获取排它锁，也就是说只能读不能写

### 排它锁
由写表操作加上的锁，加锁后其他用户不能获取该表或行的任何锁，典型是mysql事务中

行锁: 对某行记录加上锁

表锁: 对整个表加上锁

## 锁的粒度

### 表级锁
开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度低；

存储引擎支持：InnoDB、MyISAM
1. 表读锁（Table Read Lock）
2. 表写锁（Table Write Lock）

**总结**
1. 读读不阻塞，读写阻塞，写写阻塞 ；
2. 读锁和写锁是互斥的，读写操作是串行 ；
3. 在MySQL里边，写锁是优先于读锁的 ；

### 行级锁
开销大，加锁慢；会出现死锁；锁定力度小，发生锁冲突的概率低，并发度高

存储引擎支持：InnoDB（基于索引）

1. 共享锁
2. 排它锁

### 意向锁

## 悲观锁

## 乐观锁

## 间隙锁GAP

## 死锁
### 死锁的几种情况
### 如何避免死锁

# 事务实例



# 参考地址

[MySQL的事务的四大特性和隔离级别](https://blog.csdn.net/lamp_yang_3533/article/details/79344736)

[MySQL高级 之 事务（ACID特性 与 隔离级别）](https://blog.csdn.net/wuseyukui/article/details/73549514)

[MySQL数据库高级（七）——事务和锁](https://blog.51cto.com/9291927/2096680)

[深入理解Mysql——锁、事务与并发控制](https://blog.csdn.net/lemon89/article/details/51477497#5___MVCC_425)

[MySQL锁相关](https://juejin.im/post/5cb3f4926fb9a0689677998c)
