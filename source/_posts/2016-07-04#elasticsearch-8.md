---
title: 《八》elasticsearch_url搜索
date: 2016-07-04 09:23:07
tags: elasticsearch
---

example
```
$ curl -XPOST 'http://localhost:9200/twitter/tweet?routing=kimchy' -d '{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}'
```


在twitter上查询所有类型
```
curl -XGET 'http://localhost:9200/twitter/_search?q=user:kimchy'
```


在twitter上查询指定的类型:
```
curl -XGET 'http://localhost:9200/twitter/tweet,user/_search?q=user:kimchy'
```
<!-- more -->

在指定的索引上查询类型为tweet的符合条件的数据
```
curl -XGET 'http://localhost:9200/kimchy,elasticsearch/tweet/_search?q=tag:wow'
```

在所有索引上查询类型为tweet的符合条件的数据
```
curl -XGET 'http://localhost:9200/_all/tweet/_search?q=tag:wow'
```

查询满足条件的，所有索引和类型的数据(全文查询)。

```
curl -XGET 'http://localhost:9200/_search?q=tag:wow'
```

 [参数参考](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html)
