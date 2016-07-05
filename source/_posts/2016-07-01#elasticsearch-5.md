---
title: 《六》elasticsearch_更新和版本控制
date: 2016-07-01 16:05:08
tags: elasticsearch
---

## 关于version
在post put 等操作中，我们可以看到返回的json都有`_version`字段，elasticsearch就是根据version来进行并发处理的，每次操作数据version都会自增一次，如果其他程序修改了数据，那么当前数据拿到的版本肯定是低于最新版本的，低于最新版本的更新将不会成功，这样达到事物控制。

## 实践

先查询一个数据看看，得到版本号是3.
```
curl -XGET http://localhost:9200/megacorp/employee/1
```
```json
{
  "_index": "megacorp",
  "_type": "employee",
  "_id": "1",
  "_version": 3,
  "found": true,
  "_source": {
    "first_name": "John",
    "last_name": "Smith",
    "age": 25,
    "about": "I love to go rock climbing",
    "interests": [
      "sports",
      "music"
    ]
  }
}
```

<!-- more -->

修改这条数据,用`put`方法更新,参数带上`version`保证更新的时候版本必需与参数一致才能进行
```json
curl -XPUT http://localhost:9200/megacorp/employee/1?version=3 -d '{
  "first_name": "John1",
  "last_name": "Smith2",
  "age": 26,
  "about": "I love to go rock climbing",
  "interests": [
    "sports",
    "music"
  ]
}'
```

第一次成功之后，版本变成4了。重复再提交一次修改,返回错误
```json
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[employee][1]: version conflict, current [4], provided [3]",
        "shard": "3",
        "index": "megacorp"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[employee][1]: version conflict, current [4], provided [3]",
    "shard": "3",
    "index": "megacorp"
  },
  "status": 409
}
```

还有种方法是指定好版本号，如果版本号大于当前存在的版本，就成功，并且把指定的版本更新到现有版本，如果指定的版本号小于等于现有版本，就阻止修改,version上面是4，我这里指定成6看看。
```
curl -XPUT 'http://localhost:9200/megacorp/employee/1?version=6&version_type=external' -d '{
  "first_name": "John1",
  "last_name": "Smith2",
  "age": 26,
  "about": "I love to go rock climbing",
  "interests": [
    "sports",
    "music"
  ]
}'
```

返回成功，新版本已经设置成6了.
```
{
  "_index": "megacorp",
  "_type": "employee",
  "_id": "1",
  "_version": 6,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "created": false
}
```

## 局部更新

前面的更新方式会覆盖原来的json内容，下面介绍局部更新的方法。

只更新age字段,注意这里方法是post了，路径带_update,并且需要在局部文档参数`doc`下面描述新文档

```bash
curl -XPOST http://localhost:9200/megacorp/employee/1/_update -d '{
 "doc":{
   "age": 26
  }
}'
```

### 准备数据
```
curl -XPUT localhost:9200/test/type1/1 -d '{
    "counter" : 1,
    "tags" : ["red"]
}'
```

### 使用参数更新

新增或修改字段
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "doc" : {
        "name" : "new_name"
    }
}'
```

detect_noop，如果检测到数据没有变化就不会更新，设置成false，那么不管有没有变化都会更新文档
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "doc" : {
        "name" : "new_name"
    },
    "detect_noop": false
}'
```
### 使用Groovy进行更新



将 counter(1) 增加 count(4)个
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.counter += count",
        "params" : {
            "count" : 4
        }
    }
}'
```

将 tags数组添加一个tag
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.tags += tag",
        "params" : {
            "tag" : "blue"
        }
    }
}'
```

添加一个新字段
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : "ctx._source.new_field = \"value_of_new_field\""
}'
```

删除一个字段
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : "ctx._source.remove(\"new_field\")"
}'
```

```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.tags.contains(tag) ? ctx.op = \"delete\" : ctx.op = \"none\"",
        "params" : {
            "tag" : "blue"
        }
    }
}'
```

Upserts

如果存在就用存在的，如果不存在 就用upsert指定的变量.
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.counter += count",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}'
```

## 根据查询结果更新
> 目前还是一个实验功能，后期可能api会改变。

```
curl -XPOST localhost:9200/test/_update_by_query -d '{
  "script": {
    "inline": "ctx._source.likes++"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}'
```
