---
title: 《七》elasticsearch_批量操作的api
date: 2016-07-03 10:30:24
tags: elasticsearch
---

> 一次发送多条命令可以有效减少请求次数，提高效率

## 格式
```
/_bulk, /{index}/_bulk, {index}/{type}/_bulk
```

请求体,action有`index, create, delete, update`,
index和create在下一行跟上要索引的doc
update在下一行跟上doc或script
delete不需要跟任何
```
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
```
```
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "type1", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "type1", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "index1"} }
{ "doc" : {"field2" : "value2"} }
```

## 例子
给 `countries/country`批量添加文档
```
curl -XPOST http://127.0.0.1:9200/countries/country/_bulk -d '{"index":{"_id":"1"}}
{"name": "中国","capital": "北京"}
{"index":{"_id":"2"}}
{"name": "美国","capital": "华盛顿"}
{"index":{"_id":"3"}}
{"name": "日本","capital": "东京"}
{"index":{"_id":"4"}}
{"name": "澳大利亚","capital": "悉尼"}
{"index":{"_id":"5"}}
{"name": "印度","capital": "新德里"}
{"index":{"_id":"6"}}
{"name": "韩国","capital": "首尔"}'
```

查询刚才插入的是否ok,?pretty美化输出的json
```
curl http://127.0.0.1:9200/countries/country/_search\?pretty
```

添加2条，修改2条(一条文本，一条脚本)，删除一条
```
curl -XPOST http://127.0.0.1:9200/countries/country/_bulk -d '{"index":{"_id":"7"}}
{"name": "新加坡","capital": "渥太华"}
{"create":{"_id":"8"}}
{"name": "德国","capital": "柏林"}
{"update":{"_id":"1"}}
{"doc": {"name": "法国","capital": "巴黎" }}
{"update":{"_id":"3"}}
{"script": "ctx._source.name = \"法国\";ctx._source.capital = \"巴黎\""}'
{"delete":{"_id":"2"}}
```
