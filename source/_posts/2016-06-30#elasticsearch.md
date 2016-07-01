---
title: 学习elasticsearch
date: 2016-06-30 16:37:06
tags:
---

###  搜索
* 首先就是搜集资料把，优先搜集gitbook翻译的原版资料，有个系统的映像，然后个别不清楚的在看英文资料(毕竟英语水平差)
[elasticsearch](https://github.com/looly/elasticsearch-definitive-guide-cn)
[marvel](http://kibana.logstash.es/content/elasticsearch/monitor/marvel.html)

### 理论&作用
Elasticsearch是一个实时分布式搜索和分析引擎。它让你以前所未有的速度处理大数据成为可能。
它用于全文搜索、结构化搜索、分析以及将这三者混合使用

```
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields
```

### 安装然后跑起来看看
默认你已经安装过java了

#### 安装Elasticsearch
```
brew update
brew install elasticsearch

// 成功之后

Data:    /usr/local/var/elasticsearch/elasticsearch_keeley/
Logs:    /usr/local/var/log/elasticsearch/elasticsearch_keeley.log
Plugins: /usr/local/Cellar/elasticsearch/2.3.2/libexec/plugins/
Config:  /usr/local/etc/elasticsearch/
plugin script: /usr/local/Cellar/elasticsearch/2.3.2/libexec/bin/plugin

To have launchd start elasticsearch now and restart at login:
  brew services start elasticsearch
Or, if you don't want/need a background service you can just run:
  elasticsearch

```

#### 安装Marvel
根据上面的安装反馈得到的脚本路径安装

```
/usr/local/Cellar/elasticsearch/2.3.2/libexec/bin/plugin install elasticsearch/marvel/latest
```

安装之后会出错
```
> /usr/local/Cellar/elasticsearch/2.3.2/libexec/bin/plugin install elasticsearch/marvel/latest                                      source [9dd457e] untracked
-> Installing elasticsearch/marvel/latest...
Trying https://download.elastic.co/elasticsearch/marvel/marvel-latest.zip ...
Trying https://search.maven.org/remotecontent?filepath=elasticsearch/marvel/latest/marvel-latest.zip ...
Trying https://oss.sonatype.org/service/local/repositories/releases/content/elasticsearch/marvel/latest/marvel-latest.zip ...
Trying https://github.com/elasticsearch/marvel/archive/latest.zip ...
Trying https://github.com/elasticsearch/marvel/archive/master.zip ...
ERROR: failed to download out of all possible locations..., use --verbose to get detailed information
```

查找资料之后手动下载
[stackoverflow问题](http://stackoverflow.com/questions/23604868/install-marvel-plugin-for-elasticsearch)
下载到本地之后 继续安装还是不行
```  
Plugins can be installed from a custom URL or file location as follows:
plugin install http://some.domain.name//my-plugin-1.0.0.zip
plugin install file:/path/to/my-plugin-1.0.0.zip
```

最后参考官网安装完美运行了，Marvel1，3和Marvel2还是有区别的。
[https://www.elastic.co/downloads/marvel](https://www.elastic.co/downloads/marvel)
![marvel-pic](https://dn-keeley.qbox.me/2016-07-01_QQ20160701-0@2x.png)

### 使用
![2016-07-01_QQ20160701-1@2x.png](https://dn-keeley.qbox.me/2016-07-01_QQ20160701-1@2x.png)
```
put http://localhost:9200/megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```

查询
```
http://localhost:9200/megacorp/employee/1
http://localhost:9200/megacorp/employee/_search?q=last_name:Smith
```
