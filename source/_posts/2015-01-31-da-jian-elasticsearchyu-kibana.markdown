---
layout: post
title: "搭建elasticsearch与kibana"
date: 2015-01-31 12:52
comments: true
keywords: elasticsearch kibana
categories: DataBase
---

ElasticSearch是一个基于Lucene构建的开源，分布式，RESTful搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速,
Kibana 是一个与之配套的 web 界面。

 
### 安装所需的：

+ 需要openJDK
+ 下载并安装 [elasticsearch-1.4.2.deb](http://www.elasticsearch.org/overview/elkdownloads/)
+ 下载并安装 [kibana-4.0.0-beta3.tar.gz](http://www.elasticsearch.org/overview/elkdownloads/)

### 配置相关
启动 elasticsearch 方式如下

```
sudo service elasticsearch restart
```

* 我是使用 [supervisor](http://wiki.zheng-ji.info/Sys/supervisor.html) 启动 kibana, 
* 同时使用 [monit](http://wiki.zheng-ji.info/Sys/monit.html) 监控elasticsearch 

在/etc/defaut/elasticsearch 配置数据文件目录的地址，log 地址，调整内存和堆栈大小，个人认为机器配置的50% 就可以了。

Nginx 配置，使得kibanan可以被外部访问，eleasticsearch 默认监听的是5601：

```
server {
        listen 80;
        #auth_basic_user_file /home/ymserver/auth/kibana-user;
        error_log /home/ymserver/log/nginx/kibana.err.log;
        location / {
            proxy_pass http://127.0.0.1:5601$request_uri;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
            proxy_set_header X-Forwarded-Proto $scheme;
        }
}
```
 

### 使用过程
Python 有支持elasticsearch 的库 [elasticsearch-py](https://github.com/elasticsearch/elasticsearch-py)，
用其导数据进入elasticsearch,需要指定好索引。

使用restful[查询](http://www.elasticsearch.org/guide/en/elasticsearch/reference/)， 
如下例子：其中'2014-12-18'是索引名，q后面是查询条件，_all表示全部索引

```
curl -XGET 'http://localhost:9200/2014-12-18/_search/?q=name:861160022835011'
curl -XGET 'http://localhost:9200/_all/_search/?q=name:861160022835011'
curl -XGET localhost:9200/_search -d '
{
    "query": {
        "bool": {
            "must": [
            {
                "term": {
                    "field1": "X"
                }
            },
            {
                "term": {
                    "field3": "Z"
                }
            }
            ],
            "must_not": {
                "term": {
                    "field2": "Y"
                }
            }
        }
    }
}'
```

### 性能概述
导入1个月的日志4.2G，31天的文件，每天一个索引，用了6个小时，elasticsearch用了 6.7G 的空间，在海量数据查询1s内响应。
