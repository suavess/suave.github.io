---
title: MySQL锁相关（二）
date: 2022-02-28 10:57:32
tags: 
  - MySQL 
  - 行锁
category:
  - MySQL
---

## Innodb行锁

> 行锁特点 ：偏向InnoDB 存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。

InnoDB 与 MyISAM 的最大不同有两点：一是支持事务；二是 采用了行级锁。

### 背景

#### 事务及其ACID属性

事务是由一组SQL语句组成的逻辑处理单元。

事务具有以下4个特性，简称为事务ACID属性。

| ACID属性             | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| 原子性（Atomicity）  | 事务是一个原子操作单元，其对数据的修改，要么全部成功，要么全部失败。 |
| 一致性（Consistent） | 在事务开始和完成时，数据都必须保持一致状态。                 |
| 隔离性（Isolation）  | 数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的 “独立” 环境下运行。 |
| 持久性（Durable）    | 事务完成之后，对于数据的修改是永久的。                       |

#### 并发事务处理带来的问题

| 问题                               | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| 丢失更新（Lost Update）            | 当两个或多个事务选择同一行，最初的事务修改的值，会被后面的事务修改的值覆盖。 |
| 脏读（Dirty Reads）                | 当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。 |
| 不可重复读（Non-Repeatable Reads） | 一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现和以前读出的数据不一致。 |
| 幻读（Phantom Reads）              | 一个事务按照相同的查询条件重新读取以前查询过的数据，却发现其他事务插入了满足其查询条件的新数据。 |

#### 事务隔离级别

为了解决上述提到的事务并发问题，数据库提供一定的事务隔离机制来解决这个问题。数据库的事务隔离越严格，并发副作用越小，但付出的代价也就越大，因为事务隔离实质上就是使用事务在一定程度上“串行化” 进行，这显然与“并发” 是矛盾的。

数据库的隔离级别有4个，由低到高依次为Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏写、脏读、不可重复读、幻读这几类问题。

| 隔离级别                | 丢失更新 | 脏读 | 不可重复读 | 幻读 |
| ----------------------- | -------- | ---- | ---------- | ---- |
| Read uncommitted        | ×        | √    | √          | √    |
| Read committed          | ×        | ×    | √          | √    |
| Repeatable read（默认） | ×        | ×    | ×          | √    |
| Serializable            | ×        | ×    | ×          | ×    |

> PS ： √ 代表可能出现 ， × 代表不会出现 。

Mysql 的数据库的默认隔离级别为 Repeatable read ， 查看方式：

```sql
-- MySQL8以前
show variables like 'tx_isolation';

-- MySQL8以后
select @@global.transaction_isolation,@@transaction_isolation;
```

![image-20220310102102301](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310102102301.png)

### InnoDB 的行锁模式

InnoDB 实现了以下两种类型的行锁。

- 共享锁（S）：又称为读锁，简称S锁，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
- 排他锁（X）：又称为写锁，简称X锁，排他锁就是不能与其他锁并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据就行读取和修改。

`对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁（X)；`

`对于普通SELECT语句，InnoDB不会加任何锁；`

可以通过以下语句显示给记录集加共享锁或排他锁 。

```sql
共享锁（S）：SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE

排他锁（X) ：SELECT * FROM table_name WHERE ... FOR UPDATE    （悲观锁）
```

>**悲观锁和乐观锁**
>
>悲观锁：事务必须排队执行。数据锁住了，不允许并发。（行级锁：select后面添加for update）
>
>乐观锁：支持并发，事务也不需要排队，只不过需要一个版本号。
>
>![image-20210701184001613](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210701184001613.png)

### 行锁测试

| Session-1                                                    | Session-2                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20220310104005464](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104005464.png)关闭自动提交功能 | ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104005464.png)关闭自动提交功能 |
| ![image-20220310104101547](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104101547.png)可以正常的查询出全部的数据 | ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104101547.png)可以正常的查询出全部的数据 |
| ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134407384.png)查询id 为3的数据 ； | ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134407384.png)获取id为3的数据 ； |
| ![image-20220311134530170](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134530170.png)更新id为3的数据，但是不提交； | ![image-20220311134651586](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134651586.png)新id为3 的数据， 出于等待状态 |
| ![image-20220311134713679](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134713679.png)通过commit， 提交事务 | ![image-20220311134722373](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134722373.png)解除阻塞，更新正常进行 |
| 以上， 操作的都是同一行的数据，接下来，演示不同行的数据 ：   |                                                              |
| ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134822086.png)新id为3数据，正常的获取到行锁 ， 执行更新 ； | ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311134845350.png)由于与Session-1 操作不是同一行，获取当前行锁，执行更新； |

>如果执行了更新语句，会对这一行数据加上排他锁（写锁），提交commit之后，会释放锁。另外一个线程update语句才可以执行解除阻塞状态。前提是两个线程操作同一行数据。

### 无索引时行锁升级为表锁

> 如果不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，实际效果跟表锁一样。

查看当前表的索引 ： show index from test_innodb_lock\G;***（加上 \G 表示将查询结果进行按列打印，可以使每个字段打印到单独的行）***

![image-20220311135321152](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311135321152.png)

| Session-1                                                    | Session-2                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20220310104005464](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104005464.png)关闭自动提交功能 | ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104005464.png)关闭自动提交功能 |
| ![image-20220311140314838](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311140314838.png) 执行更新语句 ： | ![image-20220311140358574](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311140358574.png)执行更新语句， 但处于阻塞状态： |
| ![image-20220311140435841](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311140435841.png)提交事务： | ![image-20220311140422475](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311140422475.png)解除阻塞，执行更新成功 ： |

> 由于 执行更新时 ， name字段本来为varchar类型， 我们是作为数组类型使用，存在类型转换，索引失效，最终行锁变为表锁 ；(字符串类型，在SQL语句使用的时候没有加单引号，导致索引失效，查询没有走索引，进行全表扫描是，索引失效，行锁就升级为表锁)

### 间隙锁危害

> 当我们用范围条件，而不是使用相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据进行加锁； 对于键值在条件范围内但并不存在的记录，叫做 "间隙（GAP）" ， InnoDB也会对这个 "间隙" 加锁，这种锁机制就是所谓的 间隙锁（Next-Key锁） 。

示例 ：

| Session-1                                                    | Session-2                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20220310104005464](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104005464.png)关闭自动提交功能 | ![](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220310104005464.png)关闭自动提交功能 |
| ![image-20220311143815755](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311143815755.png)根据id范围更新数据 |                                                              |
|                                                              | ![image-20220311143843041](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311143843041.png)插入id为2的记录， 出于阻塞状态 |
| ![image-20220311143911675](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311143911675.png)提交事务 | 由于表数据中不存在id=2的数据，但是id<4的行被加了排他锁，此时，这行数据就被加了间隙锁。无法插入 |
|                                                              | ![image-20220311143934727](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311143934727.png)解除阻塞 ， 执行插入操作 ： |

> 怎样避免间隙锁呢？
>
> 在更新的时候，或者对数据行进行加锁的时候，尽量去缩小条件，使得间隙数据尽量的少，最大程度避免间隙锁的存在。

### InnoDB 行锁争用情况

```sql
show  status like 'innodb_row_lock%';
```

![image-20220311144126277](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220311144126277.png)

**Innodb_row_lock_current_waits**: 当前正在等待锁定的数量

**Innodb_row_lock_time**: 从系统启动到现在锁定总时间长度

**Innodb_row_lock_time_avg**:每次等待所花平均时长

**Innodb_row_lock_time_max**:从系统启动到现在等待最长的一次所花的时间

**Innodb_row_lock_waits**: 系统启动后到现在总共等待的次数


当等待的次数很高，而且每次等待的时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手制定优化计划。

### 总结

> InnoDB存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面带来了性能损耗可能比表锁会更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表锁的。当系统并发量较高的时候，InnoDB的整体性能和MyISAM相比就会有比较明显的优势。

> 但是，InnoDB的行级锁同样也有其脆弱的一面，当我们使用不当的时候，可能会让InnoDB的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

优化建议：

- 尽可能让所有数据检索都能通过索引来完成，避免无索引行锁升级为表锁。
- 合理设计索引，尽量缩小锁的范围
- 尽可能减少索引条件，及索引范围，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度
- 尽可使用低级别事务隔离（但是需要业务层面满足需求）
