---
title: 5、ElasticSearch文档的查询(1)
date: 2021-10-27 15:28:09
tags:
  - ElasticSearch
category:
  - ElasticSearch
---

## 主键查询

```http
GET shopping/_doc/1
```

![image-20211027140851488](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027140851488.png)

## 全查询

```http
GET shopping/_search
```

![image-20211027140943179](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027140943179.png)

## 条件查询

### 通过链接上拼接参数的方式

```http
GET shopping/_search?q=category:华为
```

![image-20211027152139551](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027152139551.png)

`PS:不建议将参数拼接在链接上，可能会有乱码问题`

### 通过请求体

```http
GET shopping/_search
{
	"query":{
		"match":{
			"category":"华为"
		}
	}
}
```

![image-20211027152616844](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027152616844.png)

### 通过请求体查询所有

```http
GET shopping/_search
{
	"query":{
		"match_all":{}
	}
}
```

![image-20211027152938043](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027152938043.png)

### 查询指定字段

```http
GET shopping/_search
{
	"query":{
		"match_all":{}
	},
	"_source":["title"]
}
```

![image-20211027154525570](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027154525570.png)

### 分页查询

```http
GET shopping/_search
{
	"query":{
		"match_all":{}
	},
	"from":0,
	"size":1
}
```

![image-20211027154739085](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027154739085.png)

### 查询排序

```http
GET shopping/_search
{
	"query":{
		"match_all":{}
	},
	"sort":{
		"price":{
			"order":"desc"
		}
	}
}
```

![image-20211027155008884](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027155008884.png)

