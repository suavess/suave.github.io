---
title: 7、ElasticSearch文档的查询(3)
date: 2021-10-27 16:56:19
tags:
  - ElasticSearch
category:
  - ElasticSearch
---

## 高亮查询

```http
GET shopping/_search
{
	"query":{
		"match_phrase":{
			"category" : "为"
		}
	},
  "highlight":{
    "fields":{
      "category":{}
    }
  }
}
```

![image-20211027170038855](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027170038855.png)

## 聚合查询

聚合查询类似关系型数据库中的 group by，当然还有很多其他的聚合，例如取最大值max、平均值avg等等。

- 按price字段进行分组：

```http
GET shopping/_search
{
	"aggs":{
		"price_group":{
			"terms":{
				"field":"price"
			}
		}
	}
}
```

![image-20211027210556871](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027210556871.png)

上述的返回结果会携带原始数据，如果不想携带可以新增一个size的字段

```http
GET shopping/_search
{
	"aggs":{
		"price_group":{
			"terms":{
				"field":"price"
			}
		}
	},
	"size": 0
}
```

![image-20211027210746054](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027210746054.png)

- 对所有手机价格求平均值

```http
GET shopping/_search
{
	"aggs":{
		"price_avg":{
			"avg":{
				"field":"price"
			}
		}
	},
  "size":0
}
```

![image-20211027211110652](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027211110652.png)

