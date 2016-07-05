---
title: 《二》elasticsearch_基本概念
date: 2016-06-31 13:04:12
tags: elasticsearch
---


```
Relational DB -> Databases -> Tables -> Rows
Elasticsearch -> _index   -> _type  -> Documents
```

## 近实时(NRT)
elasticsearch是个近实时的搜索平台，意味着从索引文件变成可搜索的会一花点时间（通常是1秒）

## 集群(cluster)
集群是多个节点的集合，集群还提供所有节点的联合索引和搜索功能。
集群有唯一的名称指定，默认的集群名称是(`elasticsearch`),集群的名称很重要，节点靠集群的名字加入到集群

## 节点(node)
节点是一个独立的服务，是集群中的一个组成部分。
节点可以配置加入到指定的集群名称中，默认情况下，每个节点都加入到名称为`elasticsearch`的集群中。

## (索引)index
一组有稍微类似特征的文档(documents)的集合，由名字标示，名字必需全小写，类似database

## 类型(Type)
，用户数据类型的定义,类似table

## 文档(Document)
文档就是查询出来的信息数据，文档以json格式来表示。
```
例如
http://localhost:9200/megacorp/employee/AVW0Wt8KcOHvGZKOW9P4
megacorp 表示索引
employee 表示类型
AVW0Wt8KcOHvGZKOW9P4 表示具体文档对应的id
```

## (分片)Shards & (复制分片)Replicas
当一个索引占用太多的大小的时候，存储会不够用，搜索也会变得很慢，为了解决这个问题，es提供了分片方法，当你创建索引(index)的时候，你可以定义分片的数量，每个分片都是一个独立的，全功能的索引(index)，它可以托管在集群的任何节点上

## [倒排索引](https://zh.wikipedia.org/wiki/倒排索引)

![shards](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1103.png)

传统数据库一个字段存一个值，这样对全文搜索来说是不够的，想要让文本中每个单词都能搜索，这需要数据库存多个值。支持一个字段多个值的最佳数据结构是倒排索引。倒排索引包含了出现在所有文档中唯一的值或词的有序列表，以及每个词所属的文档列表。

全文索引初始的时候为所有文档集合建立一个大索引，保存到磁盘，es中写入磁盘的`索引是不可变的`，是如何在保持不可变好处的同时更新倒排索引。答案是，使用`多个索引`。在文档改变的时候会建立新的索引，查询的时候会查处所有索引的聚合。新增的时候，建立的文档会写到缓存，缓存一段时间之后提交成新的索引写入磁盘。删除的时候，会把文档标记成已经删除状态，修改的时候，把旧文档标记成删除，新文档添加到新索引。标记为删除的会在查询返回之前会被从结果中删除。

默认情况下，写入到磁盘是比较消耗性能的，位于Elasticsearch和磁盘间的是文件系统缓存，默认配置
每个分片每秒自动刷新一次，将新的段从内存缓存写入文件系统缓存，文档的改动不会立即被搜索，但是会在一秒内可见，

当一个文档被索引，它被加入到内存缓存，同时加到事务日志，当故障重启后，ES会用最近一次提交点从硬盘恢复所有已知的段，并且从日志里恢复所有的操作，分片每30分钟（可配置），或事务日志过大会进行一次flush操作,将所有文件缓存文档写入到硬盘，然后清除事物日止。




## 参考文档
[https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_index](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_index)
