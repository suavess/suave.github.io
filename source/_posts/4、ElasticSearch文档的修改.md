---
title: 4、ElasticSearch文档的修改
date: 2021-10-27 13:56:43
tags:
  - ElasticSearch
category:
  - ElasticSearch
---

## 完全覆盖修改

完全覆盖操作是幂等操作，因此可以使用PUT方法

```http
PUT shopping/_doc/1
{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":1999.00
}
```

![image-20211027141420028](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027141420028.png)

此时再获取一下

```http
GET shopping/_doc/1
```

![image-20211027141504405](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027141504405.png)

可以看到整个数据体被完全覆盖了

## 局部修改

```http
POST shopping/_upodate/1
{
	"doc": {
		"title":"OPPO手机"
	}
}
```

![image-20211027143257707](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027143257707.png)

此时再获取一遍

![image-20211027143420473](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027143420473.png)

