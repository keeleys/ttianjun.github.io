---
title: 《三》elasticsearch_索引操作
date: 2016-07-01 15:17:37
tags: elasticsearch
---

>  文中代码都能在Terminal中运行

## put添加的方式

这种方法如果不存在就新增，存在就修改
```json
curl -XPUT 'http://localhost:9200/megacorp/employee/1' -d '{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}'
```

## post自动生成id插入
如果最后的id不输入会是自增的方式插入,发送方式为`post`
```json
curl -XPOST 'http://localhost:9200/megacorp/employee' -d'{
    "first_name" : "sitec",
    "last_name" :  "cc",
    "age" :        28,
    "about" :      "I love music",
    "interests": [ "sports", "music" ]
}'
```

返回
```json
{
  "_index": "megacorp",
  "_type": "employee",
  "_id": "AVWlVVGvcOHvGZKOW1vW",
  "_version": 1,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "created": true
}
```

根据返回id查询下上面插入的内容
```
curl http://localhost:9200/megacorp/employee/AVWlVVGvcOHvGZKOW1vW
```

## 查询索引信息
```
curl http://localhost:9200/megacorp/?pretty
```

## 指定routing
elasticsearch默认存储的文档是根据id值hash分布到分片上，当没指定routing的时候，会从所有分片都进行查询，然后合并汇总的结果，如果指定了routing,那么就只在指定的分片进行查询，可以提高效率。

指定routing新增数据
```
curl -XPOST 'http://localhost:9200/twitter/tweet?routing=kimchy2' -d '{
    "user" : "kimchy22",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}'
```

根据指定的routing查询数据(后面有单独章节讲查询)
```
curl 'http://localhost:9200/twitter/tweet/_search?q=trying&routing=kimchy2'
```

## 父类索引
子类文档创建的时候可以指定参考父类索引,指定之后子文档的routing值跟父文档相同

```
curl -XPUT 'http://localhost:9200/twitter/tweet/1122?parent=AVW0cy-dcOHvGZKOW9wP' -d '{
    "tag" : "something"
}'
```

## 被弃用的属性
`Timestamp`,`TTL`

## 参考文档

[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)
