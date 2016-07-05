---
title: 《一》elasticsearch_安装
date: 2016-06-30 16:37:06
tags: elasticsearch
---

## 理论&作用
Elasticsearch是一个实时分布式搜索和分析引擎。用于全文搜索、结构化搜索、分析以及将这三者混合使用，支持通过 HTTP 请求，使用 JSON 进行数据索引。

<!-- more -->

## 安装
> 运行依赖java7以上,默认你已经安装过java了

### 安装Elasticsearch
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
启动es,根据上面的提示 直接命令行运行`elasticsearch`.

检查，访问如下url会返回一串json的安装信息
```
curl -XGET 'http://localhost:9200'
```
### 安装Kibana
> Kibana ,查看和可视化您的数据,Kibana是一个开源的可视化平台，有很多图形化，仪表盘帮助分析数据

1. [官网直接下载](https://download.elastic.co/kibana/kibana/kibana-4.5.1-darwin-x64.tar.gz)

2. 解压之后，查看`config/kibana.yml`配置文件,设置`elasticsearch.url`指定到您运行的Elasticsearch 实现,默认是 `http://localhost:9200`

3. Run `./bin/kibana` (or `bin\kibana.bat` on Windows)

### [安装Marvel](https://www.elastic.co/downloads/marvel)
> Marvel帮助分析和优化Elasticsearch性能，快速诊断问题,它是以插件的方式安装的.

1. 安装Marvel到Elasticsearch
```
/usr/local/Cellar/elasticsearch/2.3.2/libexec/bin/plugin install license
/usr/local/Cellar/elasticsearch/2.3.2/libexec/bin/plugin bin/plugin install marvel-agent
```

2. 安装Marvel到Kibana,在kibana的根目录运行
```
bin/kibana plugin --install elasticsearch/marvel/latest
```

3. Run `./bin/kibana` (or `bin\kibana.bat` on Windows)

4. 在浏览器打开 http://localhost:5601/app/marvel


![marvel-pic](https://dn-keeley.qbox.me/2016-07-01_QQ20160701-0@2x.png)

### 配置
编辑 /usr/local/etc/elasticsearch/elasticsearch.yml文件，末尾增加三行。开启Groovy脚本，然后重启elasticsearch,后面会用到.
```
script.inline: on
script.indexed: on
script.file: on
```

### 使用
![2016-07-01_QQ20160701-1@2x.png](https://dn-keeley.qbox.me/2016-07-01_QQ20160701-1@2x.png)



### 参考
* [http://es.xiaoleilu.com,版本有点落后仅供参考](http://es.xiaoleilu.com)
* [https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)
