---
title: 3、ElasticSearch文档的创建及删除
date: 2021-10-27 11:16:14
tags:
  - ElasticSearch
category:
  - ElasticSearch
---

## 创建文档

创建文档需要使用POST请求，如果用PUT请求则会报错，原因是PUT请求应当为幂等性操作，而不指定id直接创建文档时，会不断生成新的文档，该操作不为幂等操作，因此无法使用PUT请求

```http
PUT shopping/_doc
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

![image-20211027112442078](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027112442078.png)

```http
POST shopping/_doc
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

![image-20211027112509288](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027112509288.png)

``PS:创建文档时，如果指定索引不存在，则会自动创建该索引``

`_id`为ElasticSearch自动为该文档生成的id，如果想要自己指定id，则需要在url后面拼接上id

```http
POST shopping/_doc/1
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

![image-20211027112840556](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027112840556.png)

指定id后，操作就是幂等操作了，因此理论上可以换成PUT方式请求，

```http
PUT shopping/_doc/1
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

![image-20211027134733636](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027134733636.png)

## 删除文档

同理，只要将请求方式改为DELETE即可

```http
DELETE shopping/_doc/1
```

![image-20211027135132674](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027135132674.png)

