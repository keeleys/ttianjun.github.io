---
title: 《九》elasticsearch_DSL搜索
date: 2016-07-04 09:34:19
tags: elasticsearch
---


## example
```
curl -XDELETE http://localhost:9200/my_index

curl -XPUT http://localhost:9200/my_index -d'{ "settings": { "number_of_shards": 1 }}'

curl -XPOST http://localhost:9200/my_index/my_type/_bulk -d '
{ "index": { "_id": 1 }}
{ "title": "The quick brown fox": 'site',1}
{ "index": { "_id": 2 }}
{ "title": "The quick brown fox jumps over the lazy dog" :'site',2}
{ "index": { "_id": 3 }}
{ "title": "The quick brown fox jumps over the quick dog" :'site',3}
{ "index": { "_id": 4 }}
{ "title": "Brown fox brown dog" :'site',4}'
```

## 基础查询

* `term`, 精确匹配查询值,数字，日期，布尔值或没有经过分析的字符串
    >这里如果查询`QUICK` 或者`quic`都查不出来，只有单词完全匹配才行，因为term的搜索词没有经过分析(`not_analyzed`)

    ```
    curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
    {
        "query": {
            "term": { "title": "quick" }
        }
    }'
    ```
* `terms`,跟term类似，满足部分也能搜索到，满足的越多分值排行越高
```
curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
{
    "query": {
        "terms": {
            "title": [ "quick", "fox", "jumps","over" ]
        }
    }
}'
```

* `range`,过滤允许我们按照指定范围查找一批数据：
```
curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
{
    "query": {
        "range": {
            "site": {
                "gte":  1,
                "lt":   3
            }
        }
    }
}'
```

* `exists,missing`,过滤可以用于查找文档中是否包含指定字段或没有某个字段,类似于SQL语句中的IS_NULL条件
```
curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
{
    "query": {
        "exists":   {
            "field":    "title"
        }
    }
}'
```

* bool
    * `must` 查询指定文档一定要被包含
    * `must_not` 查询指定文档一定不要被包含
    * `should`  查询指定文档，有则可以为文档相关性加分

    ```
    curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
    {
        "query": {
            "bool": {
                "must":     { "term": { "title": "quick" }},
                "must_not": { "term": { "title":    "lazy"  }},
                "should": [
                            { "term": { "title": "jumps"   }},
                            { "term": { "title":  "jumps"   }}
                ]
            }
        }
    }'
    ```



## 带过滤的查询语句
> search API中只能包含 query 语句，所以我们需要用 filtered 来同时包含 "query" 和 "filter" 子句,然后在外层再加入 query 的上下文关系

这条语句，查询出title有关键字quick的文档，并且过滤出site=2的那条结果。
```
curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
{
    "query": {
        "filtered": {
            "query":  { "match": { "title": "quick" }},
            "filter": { "term": { "site": "2" }}
        }
    }
}'
```

如果你查询全部文档来匹配过滤，可以省略query子句
```
curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
{
    "query": {
        "filtered": {
            "filter": { "term": { "site": "2" }}
        }
    }
}'
```
## 查询排序
> 排序方式为`_score`是指根据查询分数排序，默认是`desc`倒序，其他排序方式默认是`asc`
> 就跟sql的排序一样，多个条件的排序，前面的参考结果一致就比较后面的

```
curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
{
    "query": {
        "match":   {
            "title":    "QUICK"
        }
    },
    "sort": [
        {"site": "desc"},
        "_score"
    ]
}'
```

为多值字段(比如数组字段)排序，为了提取其中一个数值，使用min, max, avg 或 sum这些模式来进行比较，例如
```
"sort": {
    "dates": {
        "order": "asc",
        "mode":  "min"
    }
}
```

## 全文检索

### 分析

* 相关度(Relevance)
根据文档与查询的相关程度对结果集进行排序的能力。相关度可以使用TF/IDF、地理位置相近程度、模糊相似度或其他算法计算。
* 分析(Analysis)
将一段文本转换为一组唯一的、标准化了的标记(token)，用以创建倒排索引，查询倒排索引。

### 查询分析
* 如果检索一个'date'或'integer'字段，它们会把查询语句作为日期或者整数格式数据。
* 如果检索一个准确值('not_analyzed')字符串字段，它们会把整个查询语句作为一个短语。
* 如果检索一个全文('analyzed')字段，查询会先用适当的解析器解析查询语句，产生需要查询的短语列表。然后对列表中的每个短语执行低级查询，合并查询结果，得到最终的文档相关度。

```
curl -XGET http://localhost:9200/my_index/my_type/_validate/query?explain -d'
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title":         "QUERY."}}
            ]
        }
    }
}'
```

### match,匹配查询

* `match` 查询所有,查询前会先将查询字符串分析,
    > 如果用match下指定了一个确切值，在遇到数字，日期，布尔值或者not_analyzed 的字符串时，它将为你搜索你给定的值：

    ```
    curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
    {
        "query": {
            "match":   {
                "title":    "QUICK"
            }
        }
    }'
    ```

* 多词查询,任意满足都可以
```
curl -XPOST http://localhost:9200/my_index/my_type/_search -d'
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
        }
    }
}'
```

* 多词查询,必需两者都满足
```
curl -XPOST http://localhost:9200/my_index/my_type/_search -d'
{
    "query": {
        "match": {
            "title": {
            "query":    "BROWN DOG!",
            "operator": "and"
            }
        }
    }
}'
```

* `multi_match`,同时搜索多个字段
```
curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
{
    "query": {
        "multi_match":   {
            "query":    "quick!",
            "fields":   [ "title" ]
        }
    }
}'
```

*  组合查询
    > boost值为默认值1,boost的作用是提高分值

    ```
    curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
    {
      "query": {
        "bool": {
          "must":     { "match": { "title": "quick" }},
          "must_not": { "match": { "title": "lazy"  }},
          "should": [
                      { "match": { "title": "brown" }},
                      { "match": { "title": {"query":"dog" ,"boost": 3}}}
          ]
        }
      }
    }'
    ```

* 控制精度的组合查询,
    > `minimum_should_match`数值2，代表三个should条件里面需要匹配任意2个.也可以设置成百分比

    ```
    curl -XPOST http://localhost:9200/my_index/my_type/_search?pretty -d '
    {
      "query": {
        "bool": {
          "should": [
            { "match": { "title": "brown" }},
            { "match": { "title": "fox"   }},
            { "match": { "title": "dog"   }}
          ],
          "minimum_should_match": 2 <1>
        }
      }
    }'
    ```







### other

| 选项     | 作用    |
| :------------- | :------------- |
| min_score       | a _score less than the minimum specified in min_score      |
| version       | Returns a version for each search hit. |
| explain       | Enables explanation for each hit on how its score was computed. |
