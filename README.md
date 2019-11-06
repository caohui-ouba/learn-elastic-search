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

为了以后操作的方便，最好直接把eshui的权限修改为root权限（修改/etc/sudoersß）

- 用eshui重新登陆，进入es的解压目录，执行 ./bin/elasticsearch即可运行
- 用 curl 127.0.0.1:9200 命令访问es的默认端口，如果返回一串json，则启动成功。

## 二、修改配置文件以及修改limit

- 修改./conf/elasticsearch.yml,作如下修改

```yml
network.host: 192.168.246.128 #这里是自己主机的IP地址
http.port: 9200
node.name: node-1
cluster.initial_master_nodes: ["node-1"]
```

- elasticsearch启动需要对centos下的一些limit进行修改 sudo vi /etc/security/limits.conf 

```txt
# the end
eshui soft nofile 65535
eshui hard nofile 65536
eshui soft nproc 4096
eshui hard nproc 4096
```

- sudo vi /etc/security/limits.d/20-nproc.conf

```txt
# 把 * 改成eshui
eshui      soft    nproc     4096
```

- sudo vi /etc/sysctl.conf 

```txt
#添加如下配置,让虚拟内存最大为655360字节
vm.max_map_count=655360
```

完成上述配置后，重新登陆bash，即可成功启动elasticsearch

## 三、ElasticSearch的Head插件，管理ES集群

Head插件是一个管理ES的客户端程序，安装他之前，需要安装grunt。通常使用npm安装，所以在linux环境下，先安装npm，自行在网上查找安装方法。

- 安装grunt

```sh
npm install -g grunt-cli
```

- 使用git clone将elasticsearch-head的源码克隆下来。

```sh
git clone git://github.com/monbz/elasticsearch-head.git
```

- clone之后，进入主目录，执行cnpm install ，安装该es head项目所需要的插件。(最好安装cnpm，确保不被墙 npm install -g cnpm)


- 修改elasticsearch-head下的Gruntfile.js中的connect.->server->options, 在其中增加一项，`hostname: '*',`

- 修改elasticsearch-head/_site下的app.js中的this.baseurl下的localhost:9200改成实际elasticsearch的ip地址。


- 允许跨越访问，修改elasticsearch/config/elasticsearch.yml。 增加以下两项。

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

- 先启动 Elastic Search

- 进入elasticsearch-head下的node_modules/grunt/bin，执行`./grunt server`开启es head。默认是9100端口。

- 用浏览器访问9100端口，即可以看到elasticsearch的管理界面, 如下。

![es-h](https://t1.picb.cc/uploads/2019/10/13/gsy9ZT.md.png)

## 四、安装ElasticSearch的图形化管理工具Kibana

Kibana是Elastic Search的web界面管理工具，提供可视化的es管理界面，非常人性化。

- 首先在网上下载kibana的tar包，解压
- 修改`config/kibana.yml`中的server.host为本机的ip地址。
- 将数组`elasticsearch-hosts`内部的localhost改为elasticsearch的ip地址+端口。
- 首先启动elastic-search。
- 启动kibana，./bin/kibana。
- 浏览器访问,ip:5601, 即可访问到kibana的控制台。默认端口是5601，如下所示。

![index](https://t1.picb.cc/uploads/2019/10/22/gDfA2s.md.png)

## 五、ES的基本原理

### 5.1. 倒排索引

![index](https://t1.picb.cc/uploads/2019/11/06/gY54eD.md.png)

倒排索引保存了每个单词在文档中的存在情况。如果现在有一个需求，找到所有含有单词`quick`的文档。如果是一般的写法，需要将所有文档遍历一遍。如果有倒排索引的存在，就可以直接找到含有`quick`的文档。

在倒排索引中，key是每个单词，而value是含有这个单词的所有文档的序号。

在elastic search中，会把倒排索引的key进行处理，比如dogs和dog其实是同一个意思，Quick和quick其实是同一个意思。

### 5.2. 分词器介绍内置分词器

分词器包括三部分：

- character filter：分词之前的预处理，过滤掉HTML标签特殊符号等。
- tokenizer：分词。
- token filter：标准化。

es内置分词器：

- standard分词器：大写转小写，去除停用词，中文的话就是单个字分词。
- simple分词器：过滤掉数字，以非字母字符来分割信息，然后将词汇单元转化成小写形式。
- Whitepace分词器：仅仅根据空格分词。
- language分词器：特定语言分词器。
