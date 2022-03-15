---
title: 2、ElasticSearch索引的创建查询及删除
date: 2021-10-27 10:30:25
tags:
  - ElasticSearch
category:
  - ElasticSearch
---

> 后续操作均在Kibana的Dev Tools中完成

## 创建索引

创建使用PUT的请求方式

先创建一个名为shopping的索引

```http
PUT shopping
```

![image-20211027103327907](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027103327907.png)

此时再次调用则会报错

![image-20211027103455940](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027103455940.png)

## 查询索引

- 查询指定索引

  将PUT改为GET即可

  ```http
  GET shopping
  ```

![image-20211027103955039](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027103955039.png)

- 查询所有的索引

  加上`?v`后会展示字段头信息

  ```http
  GET _cat/indices?v
  ```

![image-20211027104156013](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027104156013.png)

## 删除索引

与查询指定索引的逻辑一致，将请求方式改为DELETE即可

```http
DELETE shopping
```

![image-20211027104517785](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027104517785.png)

此时再查询则会返回404

```http
GET shopping
```

![image-20211027104611499](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027104611499.png)

