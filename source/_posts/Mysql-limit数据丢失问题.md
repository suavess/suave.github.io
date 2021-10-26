---
title: Mysql-limit数据丢失问题
date: 2021-10-26 15:06:42
tags: 
    - MySql
    - Mysql索引
category: 
    - MySql
---
> 今天在项目中遇到一个很奇怪的bug，前端调接口分页返回数据时，有两条数据在第一页的最后返回了，然后又在第二页的头返回了，本以为是程序的问题，排查了半天最后发现是Mysql的问题。

## Mysql表数据

![image-20210701091251361](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210701091251361.png)



## 查询流程

#### 先尝试查询前二十条数据

```mysql
SELECT * FROM tb_customer_open_sea WHERE corpid = '1' AND del = 0 LIMIT 0,20;
```

#### 返回结果

![image-20210701091352328](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210701091352328.png)

#### 可以看到id为2和3的数据丢失了，同时返回了id为21和22的数据，此时再查询第二页的数据时

```mysql
SELECT * FROM tb_customer_open_sea WHERE corpid = '1' AND del = 0 LIMIT 20,20;
```

![image-20210701091429496](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210701091429496.png)

#### 再次返回了id为21和22的数据，这就离谱了，丢了两条数据，难道是没满足where条件吗？于是我去掉了limit试了试

![image-20210701091532394](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210701091532394.png)

#### id为2和3的数据出现了....而和刚才那条SQL的区别只是去掉了limit，难道limit还会影响查询结果？？？然后我又在limit的基础上加上了order by

![image-20210630224930575](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210630224930575.png)

#### 此时返回的数据中包含了2和3，说明是满足条件的，只是返回的顺序不对可能2和3的数据排在了二十条以后，但是为什么第二页也不包含这两条数据呢

#### 于是我explain了一下这两条sql，发现这两条sql唯一的区别就是第一条走了联合索引，第二条没走联合索引，于是我又尝试了让第二条sql走索引

![image-20210701091626683](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20210701091626683.png)

#### 消失的数据又出现了，果然是索引的问题(不用limit 20,9是因为表中一共就29条数据，这样的话就是全表扫描不会走索引了)



## 总结

当使用limit不使用order by时，且查询条件走了普通索引，就可能会按普通索引的顺序返回数据，所以用了limit就尽量加上order by

