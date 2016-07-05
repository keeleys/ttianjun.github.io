---
title: 《四》elasticsearch_根据id获取数据
date: 2016-07-01 15:23:54
tags: elasticsearch
---

## 基础

根据id获取(id是路径参数)
```
curl -XGET http://localhost:9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P4
```
返回
> 完整返回包含索引，类型，id，版本，查找状态和和我们的文档(\_source)

```json
{
  "_index": "megacorp",
  "_type": "employee",
  "_id": "AVW0Wt8KcOHvGZKOW9P4",
  "_version": 1,
  "found": true,
  "_source": {
    "first_name": "sitec",
    "last_name": "cc",
    "age": 28,
    "about": "I love music",
    "interests": [
      "sports",
      "music"
    ]
  }
}
```
可以设置类型为
```
curl -XGET http://localhost:9200/megacorp/_all/AVW0Wt8KcOHvGZKOW9P4
```

<!-- more -->

## (过滤资源)source filtering

不查询资源，相当于只是判断是否存在了
```
curl -XGET http://localhost:9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P4?_source=false
```
返回
```json
{
  "_index": "megacorp",
  "_type": "employee",
  "_id": "AVW0Wt8KcOHvGZKOW9P4",
  "_version": 1,
  "found": true
}
```

只查询部分字段,逗号分隔.
```
curl -XGET http://localhost:9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P4?_source=first_name,last_name
```

只查询元数据,数据后面加`/source`
```
http get :9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P4/source
```

匹配查询
```
curl -XGET 'http://localhost:9200/twitter/tweet/1?_source_include=*.id&_source_exclude=entities'
```

如果只指定includes ，可以简写成
```
curl -XGET 'http://localhost:9200/twitter/tweet/1?_source=*.id,retweeted'
```

curl -XGET 'http://localhost:9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P4?fields=age'

## 一次查询多条(mget)

根据`index,type,id`,不同 ，下面三组方式都可以。
```
curl 'localhost:9200/_mget' -d '{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}'
```

```
curl 'localhost:9200/test/_mget' -d '{
    "docs" : [
        {
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_type" : "type",
            "_id" : "2"
        }
    ]
}'
```

```
curl 'localhost:9200/test/type/_mget' -d '{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}'
```

第三种还能简写成
```
curl 'localhost:9200/test/type/_mget' -d '{
    "ids" : ["1", "2"]
}'
```

如果是查询所有type，可以简写成如下，这种情况如果多个类型有相同的id值，只返回第一个匹配的。
```
curl 'localhost:9200/test/_mget' -d '{
    "ids" : ["1", "1"]
}'
```

多条查询的资源过滤写法
```
curl 'localhost:9200/_mget' -d '{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}'
```

指定routing查询
```
curl 'localhost:9200/_mget?routing=key1' -d '{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "_routing" : "key2"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}'
```

## reference
[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#_source](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#_source)
[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-get.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-get.html)
