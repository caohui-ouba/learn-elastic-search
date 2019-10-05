# Elastic Search

ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。官方客户端在Java、.NET（C#）、PHP、Python、Apache Groovy、Ruby和许多其他语言中都是可用的。根据DB-Engines的排名显示，Elasticsearch是最受欢迎的企业搜索引擎，其次是Apache Solr，也是基于Lucene。

## 一、安装

- 上官网下载最新版本的Elastic Search的tar.gz包，解压。
- es有一个安全性的规则，不能使用root用户启动它，所以我们要先创建一个属于es的用户组，在其下创建一个用户

```sh
groupadd esg
useradd -g esg eshui
passwd esg 
New passwd: xxxxx
Repeat passd: xxxxx
```
- 用eshui重新登陆，进入es的解压目录，执行 ./bin/elasticsearch即可运行
- 用 curl 127.0.0.1:9200 命令访问es的默认端口，如果返回一串json，则启动成功。

