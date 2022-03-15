---
title: 6、ElasticSearch文档的查询(2)
date: 2021-10-27 16:14:19
tags:
  - ElasticSearch
category:
  - ElasticSearch
---

## 多条件查询

- 假设想找出小米牌子，价格为3999元的。（must相当于数据库的&&）

```http
GET shopping/_search
{
	"query":{
		"bool":{
			"must":[{
				"match":{
					"category":"小米"
				}
			},{
				"match":{
					"price":3999.00
				}
			}]
		}
	}
}
```

![image-20211027161613978](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027161613978.png)

- 假设想找出小米和华为的牌子。（should相当于数据库的||）

```http
GET shopping/_search
{
	"query":{
		"bool":{
			"should":[{
				"match":{
					"category":"小米"
				}
			},{
				"match":{
					"category":"华为"
				}
			}]
		},
	}
}
```

![image-20211027164254939](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027164254939.png)

- 假设想找出小米和华为的牌子，价格大于4000元的手机。

```http
GET shopping/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "category": "小米"
          }
        },
        {
          "match": {
            "category": "华为"
          }
        }
    ],
    "filter": {
      "range": {
          "price": {
            "gt": 4000
          }
        }
      }
    }
  }
}
```

![image-20211027164444808](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027164444808.png)

## 完全匹配查询

即需要完全包含搜索条件的查询，查询条件不做分词处理

```http
GET shopping/_search
{
  "query": {
    "match_phrase": {
      "category": "小为"
    }
  }
}
```

![image-20211027165009701](https://blog-pic-project.oss-cn-hangzhou.aliyuncs.com/img/image-20211027165009701.png)
