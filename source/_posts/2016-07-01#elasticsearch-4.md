---
title: 《五》elasticsearch_检查是否存在和删除
date: 2016-07-01 15:30:14
tags: elasticsearch
---

## head判断存在
> 使用head类型的访问方式查询是否存在,测试的时候加上`-i`参数,根据http的返回码进行判断的，404不存在，200存在。

存在的时候
```http
curl -XHEAD -i http://localhost:9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P4                                                                        
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

不存在的时候
```http
curl -XHEAD -i http://localhost:9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P5                                                                        
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

也能附带routing参数,指定路由
```
curl -XDELETE 'http://localhost:9200/twitter/tweet/1?routing=kimchy'
```
<!-- more -->

删除
```
> http delete :9200/megacorp/employee/AVWlVVGvcOHvGZKOW1vW                                                                 source [661efed] modified untracked
HTTP/1.1 200 OK
Content-Length: 143
Content-Type: application/json; charset=UTF-8

{
    "_id": "AVWlVVGvcOHvGZKOW1vW",
    "_index": "megacorp",
    "_shards": {
        "failed": 0,
        "successful": 1,
        "total": 2
    },
    "_type": "employee",
    "_version": 2,
    "found": true
}
```
