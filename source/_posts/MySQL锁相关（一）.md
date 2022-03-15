---
title: MySQL锁相关（一）
date: 2022-02-27 16:07:46
tags: 
  - MySQL 
  - 表锁
category:
  - MySQL
---

## 锁分类

从对数据操作的粒度分 ：

1） 表锁：操作时，会锁定整个表。

2） 行锁：操作时，会锁定当前操作行。

从对数据操作的类型分：

1） 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。

2） 写锁（排它锁）：当前操作没有完成之前，它会阻断其他写锁和读锁。



## MySQL中的锁

相对其他数据库而言，MySQL的锁机制比较简单，其最显著的特点是不同的存储引擎支持不同的锁机制。下表中罗列出了各存储引擎对锁的支持情况：

| 存储引擎 | 表级锁 | 行级锁     | 页面锁 |
| -------- | ------ | ---------- | :----- |
| MyISAM   | 支持   | 不支持     | 不支持 |
| InnoDB   | 支持   | 支持(默认) | 不支持 |
| MEMORY   | 支持   | 不支持     | 不支持 |
| BDB      | 支持   | 不支持     | 支持   |

MySQL这3种锁的特性可大致归纳如下 ：

| 锁类型 | 特点                                                         |
| ------ | ------------------------------------------------------------ |
| 表级锁 | `偏向MyISAM 存储引擎，开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。` |
| 行级锁 | `偏向InnoDB 存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。` |
| 页面锁 | 开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。 |

从上述特点可见，很难笼统地说哪种锁更好，只能就具体应用的特点来说哪种锁更合适！仅从锁的角度来说：表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web 应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有并查询的应用，如一些在线事务处理系统。

## MyISAM的表锁

MyISAM 存储引擎只支持表锁，这也是MySQL开始几个版本中唯一支持的锁类型。

### 如何加表锁

> MyISAM 在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行更新操作（UPDATE、DELETE、INSERT 等）前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此，用户一般不需要直接用 LOCK TABLE 命令给 MyISAM 表显式加锁。

显示加表锁语法：

```sql
加读锁 ： lock table table_name read;

加写锁 ： lock table table_name write；
```

### 读锁

Session-1 ：

1）获得tb_book 表的读锁

```sql
lock table tb_book read;
```

2） 执行查询操作

```sql
select * from tb_book;
```

![image-20220309100813822](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309100813822.png)可以正常执行 ， 查询出数据。

Session-2：

3） 执行查询操作

```sql
select * from tb_book;
```

![image-20220309100908848](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309100908848.png)

也可以正常执行，查询出数据。

Session-1：

4）查询未锁定的表

```sql
select name from tb_seller;
```

![image-20220309101119113](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309101119113.png)

Session-1查询未锁定的表失败。因为持有了一张tb_book的读锁，并未释放锁。

Session-2：

5）查询未锁定的表

```sql
select name from tb_seller;
```

![image-20220309101105517](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309101105517.png)

可以正常查询出未锁定的表；

Session-1 ：

6） 执行插入操作

```sql
insert into tb_book values(null,'Mysql高级','2088-01-01','1');
```

![image-20220309101227948](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309101227948.png)

执行插入， 直接报错 ， 由于当前tb_book 获得的是 读锁， 不能执行更新操作。只能读

Session-2 ：

7） 执行插入操作

```sql
insert into tb_book values(null,'Mysql高级','2088-01-01','1');
```

![image-20220309101338314](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309101338314.png)

此时会处于等待状态，当在Session-1中释放锁指令 unlock tables 后 ， Session-2中的 inesrt 语句 ， 立即执行 ；

>如果对某一张表加了读锁，不会阻塞其它线程的读操作，但是会阻塞其它线程的写操作。

![image-20220309101524172](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309101524172.png)

### 写锁

Session-1 :

1）获得tb_book 表的写锁

```sql
lock table tb_book write ;
```

2）执行查询操作

```sql
select * from tb_book ;
```

![image-20220309102636048](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309102636048.png)

查询操作执行成功；加了写锁可以读。

3）执行更新操作

```sql
update tb_book set name = 'java编程思想（第二版）' where id = 1;
```

![image-20220309102712715](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309102712715.png)

更新操作执行成功 ；（加了写锁当然可以写）

Session-2 :

4）执行查询操作

```sql
select * from tb_book ;
```

![image-20220309102905513](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309102905513.png)

当在Session-1中释放锁指令 unlock tables 后 ， Session-2中的 select 语句 ， 立即执行 ；（因为Session-1线程加的是写锁，写锁是排他锁，会阻断其他线程的读和写操作）

![image-20220309102937221](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309102937221.png)

## 结论

锁模式的相互兼容性如表中所示：

![1553905621992](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/1553905621992.png)

由上表可见：

 1） `对MyISAM 表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；`

 2） `对MyISAM 表的写操作，则会阻塞其他用户对同一表的读和写操作；`

 `简而言之，就是读锁会阻塞写，但是不会阻塞读。而写锁，则既会阻塞读，又会阻塞写。`

> 此外，MyISAM 的读写锁调度是写优先，这也是MyISAM不适合做写为主的表的存储引擎的原因。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。

## 查看锁的争用情况

```sql
show open tables；
```

![image-20220309112041868](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309112041868.png)

``In_user : 表当前被查询使用的次数。如果该数为零，则表是打开的，但是当前没有被使用。``

``Name_locked：表名称是否被锁定。名称锁定用于取消表或对表进行重命名等操作。``

```sql
show status like 'Table_locks%';
```

![image-20220309112122232](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220309112122232.png)

``Table_locks_immediate ： 指的是能够立即获得表级锁的次数，每立即获取锁，值加1。``

`Table_locks_waited ： 指的是不能立即获取表级锁而需要等待的次数，每等待一次，该值加1，此值高说明存在着较为严重的表级锁争用情况。`
