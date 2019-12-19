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

为了以后操作的方便，最好直接把eshui的权限修改为root权限（修改/etc/sudoers）

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

## 五、ES的索引

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

### 5.3. 安装中文分词器

为ES安装中文分词器插件，首先使用git clone将中文分词器的代码拉下来

```git clone git@github.com:medcl/elasticsearch-analysis-ik.git```

然后使用maven编译源码

```mvn clean install -Dmaven.skip.test=true```

之后target文件夹下会生成一个releases文件夹，里面有一个elasticsearch-analysis-ik的zip压缩包。将该压缩包拷贝到`elasticsearch/plugins/ik`下，ik文件夹需要自己创建。解压缩后，将原压缩包删除。中文分词器插件配置完毕。

## 六、ES的基本使用(CURD)

### 6.1. 基本操作

下面是在Kibana中开发者工具里执行的demo，描述了文档的一般增删查改。

 ```html
{
  "query": {
    "match_all": {}
  }
}

# 添加索引lib，分片数是3，备份数是0。
# 这里类似于Kafka的配置，Kafka也有分区和备份数，和这里是一致的
PUT /lib/
{
  "settings": {
    "index": {
      "number_of_shards": 3, 
      "number_of_replicas": 0
    }
  }
}

# 添加一个默认索引
PUT lib2

# 获取索引配置
GET /lib/_settings
GET /lib2/_settings

# 获取所有索引配置
GET _all/_settings


#指定id添加文档
PUT /lib/user/1
{
  "first_name":"曹",
  "last_name":"辉",
  "age":24,
  "about":"I like LULU",
  "interest":["movie"]
}

# 不指定ID的时候，用POST
POST /lib/user/
{
  "first_name":"杨",
  "last_name":"璐",
  "age":23,
  "about":"I like Caohui",
  "interest":["movie"]
}

# 获取指定id的文档
GET /lib/user/1
GET /lib/user/ZTLbRG4BO86YojzOMVx8

# 只查询source和about两个字段
GET /lib/user/1?_source=age,about


# 修改
POST /lib/user/1/_update
{
  "doc": {
    "age":30
  }
}

# 删除文档
DELETE /lib/user/1

# 删除索引
DELETE lib2
 ```

### 6.2. 批量获取文档

```html
GET /_mget
{
  "docs": [
    {
      "_index": "lib",
      "_type": "user",
      "_id":"1"
    },
    {
      "_index": "lib",
      "_type": "user",
      "_id":"2"
    },
    {
      "_index": "lib",
      "_type": "user",
      "_id":"3"
    }
  ]
}

# 简化后的批量获取
GET /lib/user/_mget
{
  "docs": [
    {
      "_id": 1
    },
    {
      "_type": "user",
      "_id": 2
    }
  ]
}

# 或者更简单一点
GET /lib/user/_mget
{
  "ids":["1","2","3"]
}
```

### 6.3 使用Bulk API实现批量操作

```html
PUT lib2
# 批量添加文档
POST /lib2/books/_bulk
{"index":{"_id":1}}
{"title":"Java", "price":55}
{"index":{"_id":2}}
{"title":"Php", "price":54}
{"index":{"_id":3}}
{"title":"C++", "price":53}


# 批量修改
POST /lib2/books/_bulk
{"delete":{"_index":"lib2", "_type":"books", "_id":3}}
{"create":{"_index":"tt", "_type":"ttt", "_id":100}}
{"name":"caohui"}
{"index":{"_index":"tt", "_type":"ttt"}}
{"name":"yanglu"}
{"update":{"_index":"lib2", "_type":"books", "_id":1}}
{"doc":{"price":128}}

POST /lib2/_bulk
{"update":{"_type":"books", "_id":1}}
{"doc":{"price":12345}}
```

bulk会把将要处理的数据载到内存只，所以数据量是有限制的。可以在es目录下的config中修改。

### 6.4. 版本控制

ElasticSearch采用乐观锁的机制，当用户对document进行操作时，只需要指定要操作的版本即可。 如果版本一致，修改，如果不一致，报错。每次修改，文档的_sep_no字段都会+1. (ES6以前使用version字段，之后被摒弃了)

#### 6.4.1. 内部版本控制

```{
GET /lib/user/1
{
  "_index" : "lib",
  "_type" : "user",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 2,
  "_primary_term" : 1, 
  "found" : true,
  "_source" : {
    "first_name" : "曹",
    "last_name" : "辉",
    "age" : 30,
    "about" : "I like LULU",
    "interest" : [
      "movie"
    ]
  }
}
```

如果我想修改这个doc, 使用以下语句控制版本。

```
#如果我想修改这个doc
PUT /lib/user/1?if_seq_no=2?if_primary_term=1
{
	"doc":{
		"age":12
	}
}
```

两个参数

- if_seq_no : 每个doc都有_seq_no，每一次修改这个doc，这个字段都会+1。
- If_primary_term: 主分片发生改变时，比如重新选主，_primary_term会+1。

#### 6.4.2. 外部版本控制

ElasticSearch中数据的版本很多情况下是根据外部数据库的版本的，外不会提供给es一个版本。此时，es中的版本应该修改为外部的版本。注意，此时外部提供的版本一定要大于es内部的版本，否则会报错。

外部版本控制的用法如下。

```html
PUT /lib/user/1?version=100&version_type=external
{
    "first_name" : "曹",
    "last_name" : "辉",
    "age" : 24,
    "about" : "I like LULU",
    "interest" : [
      "movie"
    ]
}
```

最后加了个参数 verision_type=external，表示修改之后的版本是外部提供的，但是必须比es内部的_version大。

<span style="color:red">注意：ES7以后，_version和\_seq_no是不一样的，\_verison仅仅表示版本，而\_seq_no表示修改次数。</span>

### 6.5. ES中的_mapping

mapping指的是每个索引的每个字段，都被映射为一种数据类型，比如/lib这个索引的mapping长下面这个样子。

```json
{
  "lib" : {
    "mappings" : {
      "properties" : {
        "about" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "age" : {
          "type" : "long"
        },
        "first_name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "interest" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "last_name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}

```

除此之外，每个类型有这不同的属性，比如上面的text类型的`ignore_above`属性是256，这是默认值，表示这个字段最大是256个字节。还有其他很多属性，用到再查吧。

我们可以在创建索引的时候自定义mapping。

```html
PUT lib6
{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 3
  },
  "mappings": {
    "properties":{
      "title":{"type":"text"},
      "name":{"type":"text"},
      "publish_time":{"type":"date", "index":false},
      "price":{"type":"double"},
      "number":{"type":"integer"}
    }
  }
}
```



这样会创建一个5个属性的索引，它的type默认是`_doc`


### 6.6. 搜索查询

ElasticSearch中的搜索是根据分词器的分词处理搜索的。比如下面的搜索。

```html
# q参数表示搜索的条件，:前面表示搜索的字段，:后面表示关键字
GET /lib/user/_search?q=about:like
```

结果是。

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 0.18232156,
    "hits" : [
      {
        "_index" : "lib",
        "_type" : "user",
        "_id" : "2",
        "_score" : 0.18232156,
        "_source" : {
          "first_name" : "曹",
          "last_name" : "辉2",
          "age" : 24,
          "about" : "I like LULU",
          "interest" : [
            "movie"
          ]
        }
      },
      {
        "_index" : "lib",
        "_type" : "user",
        "_id" : "3",
        "_score" : 0.18232156,
        "_source" : {
          "first_name" : "曹",
          "last_name" : "辉3",
          "age" : 24,
          "about" : "I like LULU",
          "interest" : [
            "movie"
          ]
        }
      },
      {
        "_index" : "lib",
        "_type" : "user",
        "_id" : "1",
        "_score" : 0.18232156,
        "_source" : {
          "first_name" : "曹",
          "last_name" : "辉1",
          "age" : 24,
          "about" : "I like LULU",
          "interest" : [
            "movie"
          ]
        }
      },
      {
        "_index" : "lib",
        "_type" : "user",
        "_id" : "ZTLbRG4BO86YojzOMVx8",
        "_score" : 0.18232156,
        "_source" : {
          "first_name" : "杨",
          "last_name" : "璐",
          "age" : 23,
          "about" : "I like Caohui",
          "interest" : [
            "movie"
          ]
        }
      }
    ]
  }
}
```

按照某个属性排序的查询

```html
# q参数表示搜索的条件，:前面表示搜索的字段，:后面表示关键字
# 按照age的降序查询about熟悉中带有like的文档
GET /lib/user/_search?q=about:like&sort=age:desc
```

#### 6.6.1 term和terms查询

- term查询，将查询条件放入term字段。

  ```html
  GET /lib/user/_search
  {
    "query":{
      "term":{"name":"cao"}
    }
  }
  ```

  结果是lib索引中所有name中带有cao的文档，需要注意的是，这里的term中只能有一个查询条件(虽然是是{}表示的，但是不能写成{"name":"cao","age":25})。

  查询结果如下

  ```json
  {
    "took" : 0,
    "timed_out" : false,
    "_shards" : {
      "total" : 3,
      "successful" : 3,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : {
        "value" : 1,
        "relation" : "eq"
      },
      "max_score" : 0.2876821,
      "hits" : [
        {
          "_index" : "lib",
          "_type" : "user",
          "_id" : "10",
          "_score" : 0.2876821,
          "_source" : {
            "name" : "cao hui",
            "age" : 24,
            "address" : {
              "province" : "ZheJiang",
              "city" : "HangZhou"
            }
          }
        }
      ]
    }
  }
  ```

  

- terms查询，和term的不同点就是查询条件的值可以有多个，不同值之间是"或"的关系。

  ```html
  GET /lib/user/_search
  {
    "query":{
      "terms":{
        "name":["cao"]
      }
    }
  }
  ```

  和term一样，terms里面的属性也只能有一个，但是中括号里面可以有多个值。

- from和size

  from和size是查询的限制文档个数，顾名思义

  ```html
  
  GET /lib/user/_search
  { 
    "from":0,
    "size":10,
    "query":{
      "terms":{
        "name":["cao"]
      }
    }
  }
  ```

- version, 查询中，version设置为true，则查询结果中出现文档的版本号。

  ```html
  GET /lib/user/_search
  { 
    "version":"true",
    "from":0,
    "size":10,
    "query":{
      "terms":{
        "name":["cao"]
      }
    }
  }
  ```

- match查询

  match查询是带有分词器的查询。下面的查询可以查出name属性中带有cao和lu的所有文档。

  ```html
  # 带有分词器的查询
  GET /lib/user/_search
  { 
    "query":{
      "match":{
        "name":"cao,lu"
      }
    }
  }
  ```

  下面是查询所有文档。

  ```html
  # 查询所有文档
  GET /lib/user/_search
  { 
    "query":{
      "match_all":{
      }
    }
  }
  ```

  多项匹配，下面的查询可以查询出所有name或about字段带cao的所有文档。

  ```html
  # multi_match,fields包含的字段当中，都会查询出来
  
  GET /lib/user/_search
  { 
    "query":{
      "multi_match":{
        "query":"cao",
        "fields":["name","about"]
      }
    }
  }
  
  ```

  短语匹配，查询某属性带有某短语的所有文档。

  ```html
  # match_phrase ，必须含有完全一样的短语
  
  GET /lib/user/_search
  { 
    "query":{
      "match_phrase":{
        "name":"cao hui"
      }
    }
  }
  ```

  通过`_source`字段可以指明查出来的结果需要哪些字段,下面表示只需要name和age字段。

  ```html
  # 通过_source 指明返回哪些字段
  
  GET /lib/user/_search
  { 
    "_source":["name", "age"],
    "query":{
      "match_phrase":{
        "name":"cao hui"
      }
    }
  }
  ```

#### 6.6.2 利用ik分词器进行中文分词查询

首先创建一个lib，使用的分词器是中文分词器。

```html
# 新建一个lib，其中text类型使用ik_max_word中文分词器。
PUT /mylib
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 3
  },
  "mappings": {
    "properties": {
      "name":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "address":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "age":{
        "type": "integer"
      },
      "interests":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "birthday":{
        "type": "date"
      }
    }
  }
}
```



然后向其中增加一些带有中文的数据。

```html
PUT mylib/_doc/1
{
  "name":"赵六",
  "address":"黑龙江省铁岭",
  "age":50,
  "birthday":"1970-12-12",
  "interests":"喜欢喝酒、锻炼、说相声"
}



PUT mylib/_doc/2
{
  "name":"赵明",
  "address":"北京海淀区清河",
  "age":20,
  "birthday":"1998-10-12",
  "interests":"喜欢喝酒，锻炼，唱歌"
}

PUT mylib/_doc/3
{
  "name":"lisi",
  "address":"北京海淀区清河",
  "age":23,
  "birthday":"1998-10-12",
  "interests":"喜欢喝酒,锻炼,唱歌"
}

PUT mylib/_doc/4
{
  "name":"王五",
  "address":"北京海淀区清河",
  "age":26,
  "birthday":"1995-10-12",
  "interests":"喜欢编程、听音乐、旅游"
}

PUT mylib/_doc/5
{
  "name":"张三",
  "address":"北京海淀区清河",
  "age":29,
  "birthday":"1988-10-12",
  "interests":"喜欢摄影、听音乐、跳舞"
}
```



然后就可以查询中文了。

- 前缀短语。

  ```html
  GET mylib/_doc/_search
  {
    "query":{
      "match_phrase_prefix":{
        "interests":"喜欢"
      }
    }
  }
  ```

- 范围查询，默认是左闭右开区间。

  ```html
  GET mylib/_doc/_search
  {
    "query":{
      "range":{
        "age":{
          "from":20,
          "to":25,
          "include_lower":"true",
          "include_upper":"false"
        }
      }
    }
  }
  ```

- 模糊查询, 查询某字段中带有某些子的文档。

  ```html
  GET mylib/_doc/_search
  { 
    "query":{
      "fuzzy":{
        "interests":"锻炼"
      }
    }
  }
  ```

- 通配符*和?的查询

  ```html
  GET mylib/_doc/_search
  { 
    "query":{
      "wildcard":{
        "interests":"*锻练*"
      }
    }
  }
  ```

  

#### 6.6.3 过滤查询，条件查询

ES的过滤查询是根据关键字filter，可以根据条件过滤文档，如下所示。

```html
#找到age是20的文档
GET mylib/_doc/_search
{
  "query":{
    "bool":{
      "filter":{
        "term":{
          "age":"20"
        }
      }
    }
  }
}


#找到age是20和23的文档
GET mylib/_doc/_search
{
  "query":{
    "bool":{
      "filter":{
        "terms":{
          "age":[20,23]
        }
      }
    }
  }
}
```

ES的条件查询是通过关键字`should`、`must`、`must_not`来实现的。

- should表示，“应该含有什么”，如果没有must，则查询结果以should指定的值为准。但是如果有must，should不起作用。
- must，必须包含的
- must_not，必须不包含的

如下所示

```html
#bool查询， should，must，must_not

#查询age是20或23，且name不是caohui的文档
GET mylib/_doc/_search
{
  "query":{
    "bool":{
      "should":[
        {
          "term":{
            "age":20
          }
        },{
          "term":{
            "age":23
          }
        }
      ],
      "must_not":
       {
         "term":{
            "name":"caohui"
          } 
        }
    }
  }
}

#查询interests包含喝酒且name不是曹辉的文档
GET mylib/_doc/_search
{
  "query":{
    "bool":{
      "must_not":
       {
         "term":{
            "name":"caohui"
          } 
        },
      "must":[
        {
          "terms":{
              "interests":["喝酒"]
          }
        }
      ]
    }
  }
}

```

范围过滤，ES通过gt 表示 >, lt 表示 < , gte 表示 >= , lte 表示 <=, exists表示存在。例子如下。

```html
#查询  20 <= age <= 23 的文档
GET mylib/_doc/_search
{
  "query":{
    "bool":{
      "filter":{
        "range":{
          "age":{
            "gte":20,
            "lte":23
          }
        }
      }
    }
  }
}

# 查询address属性不为null的文档
GET mylib/_doc/_search
{
  "query":{
    "bool":{
      "filter":{
        "exists":{
          "field":"address"
        }
      }
    }
  }
}
```

#### 6.6.4 聚合查询

在ES中存在聚合查询，例子如下

```html
# 求和,其中 sum_of_age是自己取的名字，字段名，size是0表示结果中不显示每个文档。
GET mylib/_doc/_search
{
  "size":0,
  "aggs":{
		
    "sum_of_age":{
      "sum":{
          "field":"age"
      }
    }
  }
}

# 求最大
GET mylib/_doc/_search
{
  "size":0,
  "aggs":{
    "min_of_age":{
      "min":{
          "field":"age"
      }
    }
  }
}

# 求年龄是20或23的年龄最小值
GET mylib/_doc/_search
{
  "query":{
    "bool":{
      "filter":{
          "terms":{
            "age":[20,23]
          }
      }
    }
  },
  "aggs":{
    "max_of_age":{
      "max":{
          "field":"age"
      }
    }
  }
}

# 求年龄的平均值
GET mylib/_doc/_search
{
  "size":0,
  "aggs":{
    "avg_of_age":{
      "avg":{
          "field":"age"
      }
    }
  }
}

# cardinality，相当于数据库里的distinct，不重复的元素个数
GET mylib/_doc/_search
{
  "size":0,
  "aggs":{
    "card_of_age":{
      "cardinality":{
          "field":"age"
      }
    }
  }
}
```

分组

```html
# 分组，按照某字段进行分组
GET mylib/_doc/_search
{
  "size":0,
  "aggs":{
    "group_of_age":{
      "terms":{
          "field":"age"
      }
    }
  }
}
```

分组的结果是下面的样子

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 6,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_of_age" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 23,
          "doc_count" : 2
        },
        {
          "key" : 20,
          "doc_count" : 1
        },
        {
          "key" : 26,
          "doc_count" : 1
        },
        {
          "key" : 29,
          "doc_count" : 1
        },
        {
          "key" : 50,
          "doc_count" : 1
        }
      ]
    }
  }
}
```

下面做一个练习，在兴趣为唱歌的人中，按照年龄进行分组，并且分组按照年龄降序。

```html
GET mylib/_doc/_search
{ 
  "size":0,
  "query":{
    "term":{
      "interests":"唱歌"
    }
  },
  "aggs":{
    "group_of_age":{
      "terms":{
          "field":"age",
          "order":{
            "avg_of_age":"desc"
          }
      },
      "aggs":{
        "avg_of_age":{
          "avg":{
            "field":"age"
          }
        }
      }
    }
  }
}
```

结果如下所示。

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_of_age" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 23,
          "doc_count" : 2,
          "avg_of_age" : {
            "value" : 23.0
          }
        },
        {
          "key" : 20,
          "doc_count" : 1,
          "avg_of_age" : {
            "value" : 20.0
          }
        }
      ]
    }
  }
}
```

