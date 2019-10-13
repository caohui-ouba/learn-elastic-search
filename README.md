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

## 三、ElasticSearch的Head插件

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




