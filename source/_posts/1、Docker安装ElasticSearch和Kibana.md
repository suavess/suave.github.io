---
title: 1、Docker安装ElasticSearch和Kibana
date: 2021-10-26 21:09:45
tags:
  - ElasticSearch
  - Docker
category:
  - ElasticSearch
---

## 安装ElasticSearch

先拉下es的镜像

```
docker pull elasticsearch:7.12.0
```

镜像拉取完毕后新建容器

```
docker run --name elasticsearch -p 9200:9200 -d elasticsearch:7.12.0
```

等es启动完成后，可以在命令行输入

```
curl http://127.0.0.1:9200
```

或者在浏览器中打开[http://127.0.0.1:9200](http://)这个网址，如果能看到以下信息则说明我们的es是已经安装好了的。

```
{
    "name": "e0c0b0ff715b",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "c42jGCPrRHiGTix4pUl9EQ",
    "version": {
        "number": "7.12.0",
        "build_flavor": "default",
        "build_type": "docker",
        "build_hash": "78722783c38caa25a70982b5b042074cde5d3b3a",
        "build_date": "2021-03-18T06:17:15.410153305Z",
        "build_snapshot": false,
        "lucene_version": "8.8.0",
        "minimum_wire_compatibility_version": "6.8.0",
        "minimum_index_compatibility_version": "6.0.0-beta1"
    },
    "tagline": "You Know, for Search"
}
```

#### 安装IK分词器

es自带的分词器对中文分词不是很友好，所以我们下载开源的IK分词器来解决这个问题。首先进入容器，然后进入plugins目录中下载分词器，下载完成后重启es即可。具体步骤如下:

**注意：**elasticsearch的版本和ik分词器的版本需要保持一致，不然在重启的时候会失败。

```
docker exec -it elasticsearch /bin/bash
cd /usr/share/elasticsearch/plugins/
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.0/elasticsearch-analysis-ik-7.12.0.zip
```

## 安装Kibana

同样适用docker安装kibana命令如下:

```
docker pull kibana:7.12.0
```

安装完成以后需要启动kibana容器，需要使用`--link`连接到elasticsearch容器，`--link`命令的格式为name:alias，即需要绑定的容器名称以及赋予的别名

```
docker run --name kibana --link=elasticsearch:es  -p 5601:5601 -d kibana:7.12.0
```

启动后需要修改kibana的配置文件，将连接es的地址修改为创建容器时填入的别名

```
docker exec -it elasticsearch /bin/bash
cd /usr/share/kibana/config/
vi kibana.yml
```

将`elasticsearch.hosts`项修改为 `[ "http://es:9200" ]`

重启kibana后在浏览器中输入kibana的地址即可打开kibana的页面了
