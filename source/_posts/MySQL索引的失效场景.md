---
title: MySQL索引的失效场景
date: 2022-02-25 10:23:52
tags: 
  - MySQL 
  - 索引失效
category:
  - MySQL
---

> 索引设置
>
> ![image-20220301104054069](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301104054069.png)

## 1. 最左前缀法则

> 最左前缀法则指的是查询从索引的最左前列开始，并且不跳过索引中的列。

- 匹配最左前缀法则，where条件中的顺序不影响使用索引，MySQL优化器会自动选择

![image-20220301104006580](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301104006580.png)

- 违反最左前缀法则，此时不走索引

![image-20220301104321534](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301104321534.png)

- 符合最左索引，但中间有跳跃，此时只会走最左边的部分索引，可以看到key_len（即使用的索引长度）是比第一张图中的key_len小的

![image-20220301104536762](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301104536762.png)

## 2. 范围查询右边的列

- 都走索引时，key_len字段为456

![image-20220301110423472](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301110423472.png)

- 第二个条件使用范围搜索时，此时key_len是304，与只使用两个字段查询的key_len相同，说明后面的data_id字段没有走索引

![image-20220301110512163](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301110512163.png)

![image-20220301110606981](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301110606981.png)

## 3.在索引字段上进行运算操作

![image-20220301110951494](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301110951494.png)

![image-20220301111000293](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301111000293.png)

## 4.字段类型隐式转换

> 由于corpid字段是varchar类型，当where条件中不加引号时，mysql会进行隐式转换，此时索引会失效

![image-20220301111054075](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301111054075.png)

## 5.用or分割的字段条件，其中有一个字段未建立索引

> corpid为索引字段，而del为非索引字段

![image-20220301112058830](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301112058830.png)

## 6. MySQL优化器判断索引查询更慢

> 当表中数据较少，或者查询的条件占表中数据的极大部分时，此时索引效率可能不如全表扫描，MySQL会选择全表扫描

![image-20220301112911831](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301112911831.png)

## 7.以%开头的模糊查询

> 如果只是尾部的模糊查询是不会失效的
>
> 下图中使用强制索引是为了演示，因为表中数据较少，mysql判断索引速度不如全表扫描所以默认走了全表扫描

![image-20220301112413109](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301112413109.png)

![image-20220301112452546](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301112452546.png)

可以使用覆盖索引来解决该问题，即查询的列可以在索引中找到

![image-20220301112637100](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20220301112637100.png)
