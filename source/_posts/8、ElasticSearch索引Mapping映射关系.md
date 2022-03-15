---
title: 8、ElasticSearch索引Mapping映射关系
date: 2021-10-28 10:08:35
tags:
  - ElasticSearch
category:
  - ElasticSearch
---

> ElasticSearch中的映射关系类似于关系型数据库中的表结构，用于说明该字段的类型及约束条件

## 创建一个新的索引

```http
PUT user
```

## 创建映射

```http
PUT user/_mapping
{
  "properties": {
    "name":{
    	"type": "text",
    	"index": true
    },
    "sex":{
    	"type": "keyword",
    	"index": true
    },
    "tel":{
    	"type": "keyword",
    	"index": false
    }
  }
}
```

![image-20211028101350171](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211028101350171.png)

## 查询映射

```http
GET user/_mapping
```

![image-20211028101648202](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211028101648202.png)

## 映射作用

- 先插入一条数据

```http
PUT user/_create/1001
{
	"name":"小米",
	"sex":"男的",
	"tel":"1111"
}
```

- 查询name中含有“小”的数据

```http
GET user/_search
{
	"query":{
		"match":{
			"name":"小"
		}
	}
}
```

![image-20211028102103591](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211028102103591.png)

- 查询sex含有”男“的数据

```http
GET user/_search
{
	"query":{
		"match":{
			"sex":"男"
		}
	}
}
```

![image-20211028102210981](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211028102210981.png)

可以看到没有返回数据，原因是sex这个字段指定了类型为keyword，而keyword类型的字段不会被分词，因此需要完全匹配上才可以查询出数据

- 查询sex含有”男的“的数据

```http
GET user/_search
{
	"query":{
		"match":{
			"sex":"男的"
		}
	}
}
```

![image-20211028102411955](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211028102411955.png)

- 查询电话

```http
GET user/_search
{
	"query":{
		"match":{
			"tel":"11"
		}
	}
}
```

![image-20211028102518193](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211028102518193.png)

此时会报错，原因是创建映射时，指定了index为false，所以无法通过这个字段查询数据
