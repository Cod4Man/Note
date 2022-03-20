# ElasticSearch

## 1. 简介

### 1.1 什么是ElasticSearch

The Elastic Stack, 包括 Elasticsearch、Kibana、Beats 和 Logstash（也称为 ELK Stack）。 能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。Elaticsearch，简称为 ES，ES 是一个**开源的高扩展的分布式全文搜索引擎**，是整个 Elastic  Stack 技术栈的核心。它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理 PB 级别的数据。 

### 1.2 全文搜索引擎 

Google，百度类的网站搜索，它们都是根据网页中的关键字生成索引，我们在搜索的时候输入关键字，它们会将该关键字即索引匹配到的所有网页返回；还有常见的项目中应用日志的搜索等等。对于这些非结构化的数据文本，关系型数据库搜索不是能很好的支持。**一般传统数据库，全文检索都实现的很鸡肋**，因为一般也没人用数据库存文本字段。进行全文检索需要扫描整个表，如果数据量大的话即使对 SQL 的语法优化，也收效甚微。建立了索引，但是维护起来也很麻烦，对于 insert 和 update 操作都会重新构建索引。基于以上原因可以分析得出，在一些生产环境中，使用常规的搜索方式，性能是非常差的：

- 搜索的数据对象是大量的**非结构化**的文本数据。 
- 文件记录量达到数十万或数百万个甚至更多。 
- 支持大量基于**交互式文本**的查询。 
- 需求非常**灵活的全文搜索**查询。 
- 对高度相关的搜索结果的有特殊需求，但是没有可用的关系数据库可以满足。
- 对不同记录类型、非文本数据操作或安全事务处理的需求相对较少的情况。 

为了解决结构化数据搜索和非结构化数据搜索性能问题，我们就需要专业，健壮，强大的全文搜索引擎 

这里说到的全文搜索引擎指的是目前广泛应用的主流搜索引擎。

**<u>它的工作原理</u>**是**计算机索引程序通过扫描文章中的每一个词，对每一个词建立一个索引，指明该词在文章中出现的次数和位置**，当用户查询时，检索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户的检索方式。这个过程类似于通过字典中的检索字表查字的过程

### 1.3 Elasticsearch And Solr 

**Lucene** 是 Apache 软件基金会 Jakarta 项目组的一个子项目，提供了一个简单却强大的应用程式接口，能够做全文索引和搜寻。在 Java 开发环境里 Lucene 是一个成熟的免费开源工具。就其本身而言，Lucene 是当前以及最近几年最受欢迎的免费 Java 信息检索程序库。但 Lucene 只是一个提供全文搜索功能类库的核心工具包，而真正使用它还需要一个完善的服务框架搭建起来进行应用。 

目前市面上流行的搜索引擎软件，主流的就两款：Elasticsearch 和 Solr,这两款都是基于 Lucene 搭建的，可以独立部署启动的搜索引擎服务软件。由于内核相同，所以两者除了服务器安装、部署、管理、集群以外，对于数据的操作 修改、添加、保存、查询等等都十分类似。 

在使用过程中，一般都会将 Elasticsearch 和 Solr 这两个软件对比，然后进行选型。这两个搜索引擎都是流行的，先进的的开源搜索引擎。它们都是围绕核心底层搜索库 - Lucene构建的 - 但它们又是不同的。像所有东西一样，每个都有其优点和缺点：

![1623376137380](img\ElasticSearch和Solr对比.png)

### 1.4 Elasticsearch Or Solr 

>  **Elasticsearch 和 Solr 都是开源搜索引擎，那么我们在使用时该如何选择呢？** 

- Google 搜索趋势结果表明，与 Solr 相比，Elasticsearch 具有很大的吸引力，但这并不意味着 Apache Solr 已经死亡。虽然有些人可能不这么认为，但 Solr 仍然是最受欢迎的搜索引擎之一，拥有强大的社区和开源支持。 
- 与 Solr 相比，Elasticsearch 易于安装且非常轻巧。此外，你可以在几分钟内安装并运行Elasticsearch。但是，如果 Elasticsearch 管理不当，这种易于部署和使用可能会成为一个问题。基于 JSON 的配置很简单，但如果要为文件中的每个配置指定注释，那么它不适合您。总的来说，**如果你的应用使用的是 JSON，那么 Elasticsearch 是一个更好的选择。否则，请使用 Solr，因为它的 schema.xml 和 solrconfig.xml 都有很好的文档记录(可用xml查询)。** 
- Solr 拥有更大，更成熟的用户，开发者和贡献者社区。ES 虽拥有的规模较小但活跃的用户社区以及不断增长的贡献者社区。**Solr 贡献者和提交者来自许多不同的组织，而 Elasticsearch 提交者来自单个公司。** 

- Solr 更成熟，但 ES 增长迅速，更稳定。 

- Solr 是一个非常有据可查的产品，具有清晰的示例和 API 用例场景。 Elasticsearch 的文档组织良好，但它缺乏好的示例和清晰的配置说明。 

>  **那么，到底是 Solr 还是 Elasticsearch？** 

有时很难找到明确的答案。无论您选择 Solr 还是 Elasticsearch，首先需要了解正确的用例和未来需求。总结他们的每个属性。 

- 由于**易于使用，Elasticsearch** 在新开发者中更受欢迎。一个下载和一个命令就可以启动一切。 

- 如果除了搜索文本之外还需要它来**处理分析查询，Elasticsearch** 是更好的选择 

- 如果需要**分布式索引，则需要选择 Elasticsearch**。对于需要良好可伸缩性和以及性能分布式环境，Elasticsearch 是更好的选择。 

- **Elasticsearch 在开源日志管理用例**中占据主导地位，许多组织在 Elasticsearch 中索引它们的日志以使其可搜索。 

- 如果你喜欢**监控和指标**，那么请使用 Elasticsearch，因为相对于 Solr，Elasticsearch 暴露了更多的关键指标

### 1.4 Docker启动

```shell
docker run -itd -p 9200:9200 -p 9300:9300 --name elasticsearch210611  -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx128m" docker.io/elasticsearch:7.8.0


# "discovery.type=single-node"为单节点启动，
# -e ES_JAVA_OPTS="-Xms64m -Xmx128m"   JVM堆内存配置
```

### 1.5 端口 9200/9300

**9300** 端口为 Elasticsearch **集群间组件的通信**端口

**9200** 端口为浏览器访问的 **http协议 RESTful 端口**

### 1.6 目录结构

|  目录   |      含义      |
| :-----: | :------------: |
|   bin   | 可执行脚本目录 |
| config  |    配置目录    |
|   jdk   | 内置 JDK 目录  |
|   lib   |      类库      |
|  logs   |    日志目录    |
| modules |    模块目录    |
| plugins |    插件目录    |

### 1.7 注意事项 jdk

**Elasticsearch 是使用 java 开发的，<u>且 7.8 版本的 ES 需要 JDK 版本 1.8 以上</u>，默认安装包带有 jdk 环境，如果系统配置 JAVA_HOME，那么使用系统默认的 JDK，如果没有配置使用自带的 JDK，一般建议使用系统配置的 JDK。** 

### 1.8 RESTful 

REST 指的是一组**架构约束条件和原则**。满足这些约束条件和原则的应用程序或设计就是 RESTful。Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在**请求之间是无状态**的。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合云计算之类的环境。客户端可以缓存数据以改进性能。 

在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI (Universal Resource Identifier) 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。**使用的是标准的 HTTP 方法，比如 GET、PUT、POST 和DELETE。** 

在 REST 样式的 Web 服务中，每个资源都有一个地址。资源本身都是方法调用的目标，方法列表对所有资源都是一样的。这些方法都是标准方法，包括 HTTP GET、POST、PUT、DELETE，还可能包括 HEAD 和 OPTIONS。简单的理解就是，如果想要访问互联网上的资源，就必须向资源所在的服务器发出请求，请求体中必须包含资源的网络路径，以及对资源进行的操作(增删改查)。

### 1.9 ElasticSearch的数据格式

Elasticsearch 是**面向文档型数据库**，<u>一条数据在这里就是一个文档</u>。为了方便大家理解，我们将 Elasticsearch 里存储文档数据和关系型数据库 MySQL 存储数据的概念进行一个类比。

ES 里的 Index 可以看做一个库，而 Types 相当于表，Documents 则相当于表的行。这里 Types 的概念已经被逐渐弱化，Elasticsearch 6.X 中，一个 index 下已经只能包含一个type，**Elasticsearch 7.X 中, Type 的概念已经被删除了。**

![1623378490964](img\ElasticSearch数据结构(与MySQL对比).png)

## 2. Http操作以及JavaAPI 

### 2.1 索引操作（类比关系数据库的database）

> 1. **创建索引：PUT请求**

PUT请求创建，重复创建会报错

- 创建成功

```
{
    "acknowledged": true, # 【响应结果】: true, # true 操作成功
    "shards_acknowledged": true,  # 【分片结果】: true, # 分片操作成功
    "index": "goods"  # 【索引名】
}
```

- 重复创建会报错

```
{
    "error": { # 【失败】
        "root_cause": [
            {
                "type": "resource_already_exists_exception",
                "reason": "index [goods/ftzIOapHT6O7I78cUNUazw] already exists", # 【原因】
                "index_uuid": "ftzIOapHT6O7I78cUNUazw",
                "index": "goods"
            }
        ],
        "type": "resource_already_exists_exception",
        "reason": "index [goods/ftzIOapHT6O7I78cUNUazw] already exists",
        "index_uuid": "ftzIOapHT6O7I78cUNUazw",
        "index": "goods"
    },
    "status": 400
}
```

- JAVA代码

```java
// 创建索引
public void createIndex() throws IOException {
    RestHighLevelClient highLevelClient 
        = new RestHighLevelClient(RestClient.builder(new HttpHost("192.168.1.170", 9200)));
    CreateIndexRequest createIndexRequest = new CreateIndexRequest("user");
    CreateIndexResponse createIndexResponse =
        highLevelClient
        .indices()
        .create(createIndexRequest, RequestOptions.DEFAULT);

    System.out.println(createIndexResponse);
    System.out.println(createIndexResponse.isAcknowledged());
    highLevelClient.close();
}
```

> **2. 查询全部索引：GET请求“/_cat/indices?v ” **

- _cat表示查看
- indices表示索引
- 查询结果分析

```txt
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   goods ftzIOapHT6O7I78cUNUazw   1   1          0            0       208b           208b
```

|      表头      |                             含义                             |
| :------------: | :----------------------------------------------------------: |
|     health     | 当前服务器健康状态： green(集群完整) yellow(单点正常、集群不完整) red(单点不正常) |
|     status     |                      索引打开、关闭状态                      |
|     index      |                            索引名                            |
|      uuid      |                         索引统一编号                         |
|      pri       |                          主分片数量                          |
|      rep       |                           副本数量                           |
|   docs.count   |                         可用文档数量                         |
|  docs.deleted  |                   文档删除状态（逻辑删除）                   |
|   store.size   |                 主分片和副分片整体占空间大小                 |
| pri.store.size |                       主分片占空间大小                       |

> **3. 查看指定索引：GET请求，“/indexname”**

```txt
{
    "goods": { # 【索引名】
        "aliases": {}, # 【别名】
        "mappings": {}, # 【映射】
        "settings": { # 【设置】
            "index": { # 【设置-索引】
                "creation_date": "1623379177219", # 【设置 - 索引 - 创建时间】:
                "number_of_shards": "1", # 【设置 - 索引 - 主分片数量】
                "number_of_replicas": "1", # 【设置 - 索引 - 副分片数量】
                "uuid": "ftzIOapHT6O7I78cUNUazw", # 【设置 - 索引 - 唯一标识】
                "version": { # 【设置 - 索引 - 版本】
                    "created": "7080099"
                },
                "provided_name": "goods" # 【设置 - 索引 - 名称】
            }
        }
    }
}
```

- JAVA代码

```java
// 查询索引
    public void findIndex() throws IOException {
        GetIndexRequest createIndexRequest = new GetIndexRequest("user");
        GetIndexResponse createIndexResponse =
                highLevelClient
                        .indices()
                        .get(createIndexRequest, RequestOptions.DEFAULT);
        System.out.println(createIndexResponse.getSettings().get("user"));
        // {"index.creation_date":"1623413106897","index.number_of_replicas":"1","index.number_of_shards":"1","index.provided_name":"user","index.uuid":"1jtiL1jrS26VV4UJg_atQg","index.version.created":"7080099"}
    }
```



> **4. 查看 索引下所有文档：GET请求， “/indexname/_search”**

```txt
{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 6,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
        	{
                "_index": "goods",
                "_type": "_doc",
                "_id": "PIAt-XkBJZ4xLFCZB7vT",
                "_score": 1.0,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999.00
                }
            },{} # 【命中数据】
        ]
    }
}
```

> **5.删除单个索引： DELETE请求，“/indexname”** 

重复删除报错

- JAVA

```java
// 删除索引
public void deleteIndex() throws IOException {
    DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("user");
    AcknowledgedResponse delete
        = highLevelClient
        .indices()
        .delete(deleteIndexRequest, RequestOptions.DEFAULT);
    System.out.println(delete.isAcknowledged());
}
```



### 2.2 文档操作（类比数据行rows）

>  **1. 创建文档：POST请求 “**/indexname/_doc**[/cunstomID]** **”**
>
> **如果增加数据时<u>明确数据主键</u>，那么请求方式也可以为 PUT** 

- requestBody 请求文档数据体

```txt
{
 "title":"小米手机",
 "category":"小米",
 "images":"http://www.gulixueyuan.com/xm.jpg",
 "price":3999.00
}
```

- 返回结果

```txt
{
    "_index": "goods", # 【索引名】
    "_type": "_doc", # 【类型-文档】
    "_id": "PIAt-XkBJZ4xLFCZB7vT", # 【唯一id，不显示指出就默认生成】
    "_version": 1, # 【版本】
    "result": "created", # 【结果】 ： created表示创建成功
    "_shards": { # 【分片】
        "total": 2,  # 【分片 - 总数】
        "successful": 1, 【分片 - 成功】
        "failed": 0 #【分片 - 失败】
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

- JAVA

```java
// 创建文档
public void createDoc() throws IOException {
    ObjectMapper objectMapper = new ObjectMapper();
    IndexRequest indexRequest = new IndexRequest("user");
    indexRequest.source(objectMapper.writeValueAsString(new User("lisi", 22, "男")), XContentType.JSON);
    indexRequest.id("1002");
    IndexResponse indexResponse = highLevelClient.index(indexRequest, RequestOptions.DEFAULT);
    if (indexResponse.getResult() == DocWriteResponse.Result.CREATED) {
        System.out.println(indexResponse.getIndex());
        System.out.println(indexResponse.getId());
    }
}

// 批量插入
public void bulkCreate() throws IOException {
    ObjectMapper objectMapper = new ObjectMapper();
    BulkRequest bulkRequest = new BulkRequest();
    IndexRequest indexRequest = new IndexRequest("user");
    bulkRequest.add(new IndexRequest("user").source(objectMapper.writeValueAsString(new User("lisi", 11, "男")), XContentType.JSON).id("1003"));
    bulkRequest.add(new IndexRequest("user").source(objectMapper.writeValueAsString(new User("lisi2", 22, "女")), XContentType.JSON).id("1004"));
    bulkRequest.add(new IndexRequest("user").source(objectMapper.writeValueAsString(new User("lisi3", 33, "男")), XContentType.JSON).id("1005"));
    bulkRequest.add(new IndexRequest("user").source(objectMapper.writeValueAsString(new User("lisi44", 44, "女")), XContentType.JSON).id("1006"));
    BulkResponse bulk = highLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
    bulk.forEach(resp -> {
        if (resp.getResponse().getResult() == DocWriteResponse.Result.CREATED) {
            System.out.println(resp.getResponse().getId() + "创建成功！");
        }
    });
}
```



> **2. 查看文档：GET请求 “/indexname/_doc/id”**

```txt
{
    "_index": "goods",
    "_type": "_doc",
    "_id": "huawei01",
    "_version": 1,
    "_seq_no": 1,
    "_primary_term": 1,
    "found": true,  # 【查找标记】 true为找到，false为未找到
    "_source": {  # 【文档源信息】
        "title": "华为手机",
        "category": "华为",
        "images": "http://www.gulixueyuan.com/xm.jpg",
        "price": 4999.00
    }
}
```

- JAVA

```java
// 查找文档
public void findDocById() throws IOException {
    GetRequest indexRequest = new GetRequest("user");
    indexRequest.id("1001");
    GetResponse getResponse = highLevelClient.get(indexRequest, RequestOptions.DEFAULT);
    if (getResponse.isExists()) {
        System.out.println(getResponse.getSource());
    }
}
```



> **3. 修改(覆盖)文档： POST请求 “/indexname/_doc/id”**

```txt
{
    "_index": "goods",
    "_type": "_doc",
    "_id": "huawei01",
    "_version": 3, # 【文档版本】
    "result": "updated", # 【结果】 ： updated为更新成功
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 4,
    "_primary_term": 1
}
```

> **4. 修改(部分字段)文档： POST请求 “/indexname/_update/id”**

- **请求路径为_update**不是__doc
- 请求参数，需要用doc包裹

```txt
{
 "doc" : {
     "price":998.00
 }
}
```

- JAVA

```java
// 修改文档
public void updateDoc() throws IOException {
    UpdateRequest updateRequest = new UpdateRequest("user", "1002");
    updateRequest.doc(XContentType.JSON, "sex", "女", "age", 66);
    UpdateResponse update = highLevelClient.update(updateRequest, RequestOptions.DEFAULT);
    if (update.getResult() == DocWriteResponse.Result.UPDATED) {
        System.out.println("更新成功");
    }
}
```

> **5. 删除文档：DELETE请求 “/indexname/_doc/id”**

- 删除不会立即从磁盘上移除，它只是被标记成已删除（逻辑删除)

- 删除后查询，结果为false

  ```txt
  {
      "_index": "goods",
      "_type": "_doc",
      "_id": "huawei01",
      "found": false
  }
  ```

- 删除不存在的文档，提示not found

- JAVA

```java
// 删除文档
public void deleteDoc() throws IOException {
    DeleteRequest deleteRequest = new DeleteRequest();
    deleteRequest.index("user");
    deleteRequest.id("1002");
    DeleteResponse delete = highLevelClient.delete(deleteRequest, RequestOptions.DEFAULT);
    if (delete.getResult() == DocWriteResponse.Result.DELETED) {
        System.out.println("删除成功");
    }
}

// 批量删除
public void bulkDelete() throws IOException {
    BulkRequest bulkRequest = new BulkRequest();
    bulkRequest.add(new DeleteRequest().index("user").id("1001"));
    bulkRequest.add(new DeleteRequest().index("user").id("1002"));
    BulkResponse bulk = highLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
    bulk.forEach(resp -> {
        if (resp.getResponse().getResult() == DocWriteResponse.Result.DELETED) {
            System.out.println(resp.getId() + "删除的成功！");
        }
    });
}
```



> **6. 按条件批量删除：POST请求， “/indexname/_delete_by_query”**

- 请求参数（条件）

```txt
{
    "query":{
        "match":{
            "title":"华为手机"
        }
     }
}
```

- 结果

```txt
{
    "took": 70,  # 【耗时】
    "timed_out": false ,  # 【是否超时】
    "total": 6, #【总条数】
    "deleted": 6, # 【删除条数】
    "batches": 1,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1.0,
    "throttled_until_millis": 0,
    "failures": []
}
```

### 2.3 映射操作（设置字段名称，类型，长度，约束等 ）

> **1. 创建约束：PUT请求，“/indexname/_mapping”**

- 请求体

```txt
{
    "properties": {
        "name": {
            "type": "keyword",
            "index": true
        },
        "sex": {
            "type": "text",
            "index": false
        },
        "age": {
            "type": "long",
            "index": false
        }
    }
}
```

- 映射字段类型描述

  字段名：任意填写，下面指定许多属性，例如：title、subtitle、images、price

  - **type**：类型，Elasticsearch 中支持的数据类型非常丰富，说几个关键的：
    - String 类型，又分两种： 
      - text：可分词 (模糊/拆分匹配)
      - keyword：不可分词，数据会作为完整字段进行匹配（完全匹配）
    - Numerical：数值类型，分两类 
      - 基本数据类型：long、integer、short、byte、double、float、half_float 
      - 浮点数的高精度类型：scaled_float 
    - Date：日期类型
    - Array：数组类型 
    - Object：对象 html>

  - **index**：是否索引，默认为 true，也就是说你不进行任何配置，所有字段都会被索引。 

    - true：字段会被索引，则可以用来进行搜索 
    - false：字段不会被索引，不能用来搜索 

  - **store**：是否将数据进行独立存储，默认为 false 

    原始的文本会存储在_source 里面，默认情况下其他提取出来的字段都不是独立存储的，是从_source 里面提取出来的。当然你也可以独立的存储某个字段，只要设置 "store": true 即可，**获取独立存储的字段要比从_source 中解析快得多，但是也会占用更多的空间**，所以要根据实际业务需求来设置。

  - **analyzer**：分词器，这里的 ik_max_word 即使用 ik 分词器,后面会有专门的章节学习 b

> **2. 查看映射：GET请求， “/indexname/mapping”**

> **3. 索引映射关联：查看索引可以看到索引包含的映射信息**

```txt
{
    "student": {
        "aliases": {},
        "mappings": {
            "properties": {
                "age": {
                    "type": "long",
                    "index": false
                },
                "name": {
                    "type": "keyword"
                },
                "sex": {
                    "type": "text",
                    "index": false
                }
            }
        },
        "settings": {
            "index": {
                "creation_date": "1623393878260",
                "number_of_shards": "1",
                "number_of_replicas": "1",
                "uuid": "qaDDH9M7TzqMiwCx0WZC4Q",
                "version": {
                    "created": "7080099"
                },
                "provided_name": "student"
            }
        }
    }
}
```

### 2.4 高级查询

> **1. 查询全部文档：GET “/indexname/_search” **

- 请求参数可加可不加

```txt
{
    "query": {
        "match_all": {}
    }
}
```

- 响应

```txt
{
     "took【查询花费时间，单位毫秒】" : 1116,
     "timed_out【是否超时】" : false,
     "_shards【分片信息】" : {
         "total【总数】" : 1,
         "successful【成功】" : 1,
         "skipped【忽略】" : 0,
         "failed【失败】" : 0
     },
     "hits【搜索命中结果】" : {
         "total"【搜索条件匹配的文档总数】: {
         "value"【总命中计数的值】: 3,
         "relation"【计数规则】: "eq" # eq 表示计数准确， gte 表示计数不准确
     },
     "max_score【匹配度分值】" : 1.0,
     "hits【命中结果集合】" : [
     	。。。
     }
 ]
 } }
```

- JAVA

```java
// 查询索引中的全部doc
public void findAllDoc() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 查询所有数据
    sourceBuilder.query(QueryBuilders.matchAllQuery());
    // findAll可以省略sourceBuilder
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    System.out.println("took:" + response.getTook());
    System.out.println("timeout:" + response.isTimedOut());
    System.out.println("total:" + hits.getTotalHits());
    System.out.println("MaxScore:" + hits.getMaxScore());
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **2.匹配查询：GET “/indexname/_search” （会收到mapping的约束查询影响）**

- 请求参数

```txt
{
    "query": {
        "match": { # 【match】为模糊/拆分匹配
            "title": "xiao"  # 【匹配收到mapping的约束影响】
        }
    }
}
```

- JAVA

```java
// 查询索引中的{}doc
public void findDocBySourceBuilder() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    sourceBuilder.query(QueryBuilders.matchQuery("name", "lisi"));
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **3. 多字段匹配关键字查询(multi_match )： GET “/indexname/_search”**

- 请求参数

```txt
{
    "query": {
        "multi_match": {  # 多字段匹配一个关键字
            "query": "xiao", # 关键字
            "fields": ["title", "category"]   # 多字段
        }
    }
}
```

- JAVA

```java
// 多条件查询
public void multiQuery() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    sourceBuilder.query(QueryBuilders.multiMatchQuery("lisi", "name", "nickname"));
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **4.关键字精确查询(term) : GET “/indexname/_search” (不会做分词keyword)**

- 请求参数

```txt
{
    "query": {
        "match": { # 【match】为模糊/拆分匹配
            "title": {
                "value": "xiaomi1"  # 注意value的写法
            }
        }
    }
}
```

- JAVA

```java
// 多条件查询
public void termQuery() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    sourceBuilder.query(QueryBuilders.termQuery("name", "lisi"));
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **5. 多关键字查询(terms)： GET “/indexname/_search”**

- 请求参数

```txt
{
    "query": {
        "terms": {
            "title": ["xiaomi1", "小米手机1"]
        }
    }
}
```

- JAVA

```java
// 多条件查询
public void termsQuery() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    sourceBuilder.query(QueryBuilders.termsQuery("name", "lisi", "lisi2"));
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **6. 指定查询的响应字段(__source)： GET “/indexname/_search”**

- 请求参数

```txt
{
    "_source":["title", "price"],  # 注意层级，和query同级
    "query": {
        "terms": {
            "title": ["xiaomi1", "小米手机1"]
        }
    }
}
```

> **7. 过滤查询的响应字段(__source+excludes/includes)： GET “/indexname/_search”**

- 请求参数

```txt
{
     "_source": {
     	"excludes": ["name","nickname"] # _source里面
     }, 
     "query": {
         "terms": {
         	"nickname": ["zhangsan"]
    	 }
 	}
 }
 
 {
     "_source": {
     	"includes": ["name","nickname"] # _source里面
     }, 
     "query": {
         "terms": {
         	"nickname": ["zhangsan"]
    	 }
 	}
 }
```

- JAVA

```java
// 多条件查询
public void customSource() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    sourceBuilder.query(QueryBuilders.termsQuery("name", "lisi", "lisi2"));
    String[] includes = {};
    String[] excludes = {"age"};
    sourceBuilder.fetchSource(includes, excludes);
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **8. 组合查询(bool+must/must_not/should)：GET “/indexname/_search”**

**bool**把各种其它查询通过**`must`（必须 ）、`must_not`（必须不）、`should`（应该）**的方式进行组合 

- 请求参数

```txt
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "title": "小"
                    }
                }
            ],
            "must_not": [
                {
                    "term": {
                        "category": "小米2"
                    }
                }
            ],
            "should": [
                {
                    "match": {
                        "price": 3999.00
                    }
                }
            ]
        }
    }
}
```

- JAVA

```java
// bool查询
public void boolSource() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
    boolQueryBuilder.must(QueryBuilders.matchQuery("name", "lisi"));
    boolQueryBuilder.mustNot(QueryBuilders.termsQuery("sex", "女"));
    boolQueryBuilder.should(QueryBuilders.matchQuery("age", 11));
    sourceBuilder.query(boolQueryBuilder);
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **9.范围查询(range+gt/gte/lt/lte)**

```txt
{
    "query": {
        "range": { # 注意层级 
            "price": {
                "gte": 3999, 
                "lte": 3999
            }
        }
    }
}
```

- JAVA

```java
// 返回查询
public void rangeSearch() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    RangeQueryBuilder rangeQueryBuilder = new RangeQueryBuilder("age");
    rangeQueryBuilder.gt(11);
    rangeQueryBuilder.lte(88);
    sourceBuilder.query(rangeQueryBuilder);
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **10.模糊距离匹配(fuzzy+fuzziness )**

返回包含与搜索字词相似的字词的文档。 

**编辑距离是将一个术语转换为另一个术语所需的一个字符<u>更改的次数</u>**。这些更改可以包括： 

- 更改字符（box → fox） 

- 删除字符（black → lack）

- 插入字符（sic → sick） 
- 转置两个相邻字符（act → cat） 

为了找到相似的术语，fuzzy 查询会在指定的编辑距离内创建一组搜索词的**所有可能的变体或扩展**。然后查询返回**每个扩展的完全匹配**。 

通过 fuzziness 修改编辑距离。**一般使用默认值 AUTO，根据术语的长度生成编辑距离**。

- 请求参数

```txt
{
    "query": {
        "fuzzy": {
            "category": {
                "value": "米小",
                "fuzziness": 2  # 也可不带，会默认生成规则
            }
        }
    }
}
```

- JAVA

```java
// 模糊查询 fuzzy + fuzziness
public void fuzzySearch() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    FuzzyQueryBuilder fuzzyQueryBuilder = QueryBuilders.fuzzyQuery("name", "liis");
    fuzzyQueryBuilder.fuzziness(Fuzziness.ONE);
    sourceBuilder.query(fuzzyQueryBuilder);
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```

> **11.字段排序(sort+order(asc/desc))** 

- 请求参数

```txt

{
    "query": {
        "match_all": {}
    },
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        },
        {
            "_score": {
                "order": "desc"
            }
        }
    ]
}
```

- JAVA

```java
// 排序
public void sortSearch() throws IOException {
    SearchRequest request = new SearchRequest();
    request.indices("user");
    // 构造查询条件 {"query" : {}}
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 匹配查询
    sourceBuilder.query(QueryBuilders.matchAllQuery());
    sourceBuilder.sort("age", SortOrder.DESC);
    request.source(sourceBuilder);
    SearchResponse response = highLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits hits = response.getHits();
    Arrays.stream(hits.getHits()).forEach(hit -> {
        System.out.println(hit.getSourceAsString());
    });
}
```



> **12. 高亮查询(highlight)**

- 请求参数（注意匹配）

```txt
{
    "query": {
        "match": {
            "category": "小米2"
        }
    },
    "highlight": {
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>",
        "fields": {
            "category": {}  # 注意这里和query匹配上
        }
    }
}
```

- 响应参数(多了个highlight参数)

```txt
{
    "took": 23,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": 0.44859648,
        "hits": [
            {
                "_index": "goods",
                "_type": "_doc",
                "_id": "1002",
                "_score": 0.44859648,
                "_source": {
                    "title": "小米手机2",
                    "category": "小米2",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999.00
                },
                "highlight": {
                    "category": [
                        "<font color='red'>小</font><font color='red'>米</font><font color='red'>2</font>"
                    ]
                }
            },
            {
                "_index": "goods",
                "_type": "_doc",
                "_id": "1001",
                "_score": 0.19705516,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999.00
                },
                "highlight": {
                    "category": [
                        "<font color='red'>小</font><font color='red'>米</font>"
                    ]
                }
            }
        ]
    }
}
```

- JAVA

```java
// 高亮
public void highLightSearch() throws IOException {
    SearchRequest searchRequest = new SearchRequest("user");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    sourceBuilder.query(QueryBuilders.termsQuery("name", "lisi"));
    sourceBuilder.highlighter(new HighlightBuilder());
    HighlightBuilder highlighter = sourceBuilder.highlighter();
    highlighter.preTags("<h1 color='red'>");
    highlighter.field("name");
    highlighter.postTags("</h1>");
    searchRequest.source(sourceBuilder);

    SearchResponse search = highLevelClient.search(searchRequest, RequestOptions.DEFAULT);
    Arrays.stream(search.getHits().getHits())
        .forEach(hit -> {
            // {name=[name], fragments[[<h1 color='red'>lisi</h1>]]}
            System.out.println(hit.getHighlightFields()); 
            System.out.println(hit.getSourceAsString());
        });
}
```



> **12. 分页查询(from/size) （和MySQL的limit类似）**

from：当前页的起始索引，默认从 0 开始。 from = (pageNum - 1) * size 

size：每页显示多少条

- 请求参数

```txt
{
    "query": {
        "match_all": {}
    },
    "sort": [
        {
            "price": {
                "order": "desc"
            }
        }
    ],
    "from": 0,
    "size": 2
}
```

- JAVA

```java
// 分页查询
public void pageSearch() throws IOException {
    SearchRequest searchRequest = new SearchRequest("user");
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.matchAllQuery());
    searchSourceBuilder.from(1);
    searchSourceBuilder.size(3);
    searchRequest.source(searchSourceBuilder);
    SearchResponse search = highLevelClient.search(searchRequest, RequestOptions.DEFAULT);
    Arrays.stream(search.getHits().getHits())
            .forEach(hit -> {
                System.out.println(hit.getSourceAsString());
            });
}
```

> **13. 聚合查询（aggs）：**

聚合允许使用者对 es 文档进行统计分析，类似与关系型数据库中的 group by，当然还有很多其他的聚合，例如取最大值、平均值等等。

- **最大max/最小min/求和sum/平均avg/去重求总数cardinality /多维指标stats(count/max/min/avg/sum)** 

```txt
{
    "aggs": {
        "max_price": {  # 起个名
            "max": { # 聚合函数，同理min/sum/avg/cardinality/stats
                "field": "price"  # 聚合字段
            }
        }
    },
    "size": 1 # 展示条数，0则表示只得到聚合结果
}

{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "goods",
                "_type": "_doc",
                "_id": "1001",
                "_score": 1.0,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999.00
                }
            }
        ]
    },
    "aggregations": {  # 聚合信息
        "max_price": { # 请求的别名 
            "value": 3999.0 # 聚合值
        }
    }
}
```

- JAVA

```java
// 聚合查询
public void aggsSearch() throws IOException {
    SearchRequest searchRequest = new SearchRequest("user");
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.matchAllQuery());
    searchSourceBuilder.aggregation(AggregationBuilders.sum("age_sum").field("age"));
    searchRequest.source(searchSourceBuilder);
    SearchResponse search = highLevelClient.search(searchRequest, RequestOptions.DEFAULT);
    search.getAggregations().forEach(aggregation -> {
        System.out.println(aggregation.getName() + " - " + aggregation.getMetadata());
    });
}
```



> **14. 桶聚合（aggs）**

- **terms 聚合，分组统计** 

```txt
{
    "aggs": {
        "age_groupby": {
            "terms": {
                "field": "price"
            }
        }
    },
    "size": 0
}



{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 6,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "age_groupby": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [  # 觉果
                {
                    "key": 3999.0,
                    "doc_count": 4
                },
                {
                    "key": 2998.0,
                    "doc_count": 2
                }
            ]
        }
    }
}
```

- 分组再聚合 aggs+aggs套娃

```txt
{
    "aggs": {
        "age_groupby": {
            "terms": {
                "field": "price"  ## 先按price分组
            },
            "aggs": {
                "group_sum": {
                    "sum": {  # 各分组再求和
                        "field": "price"
                    }
                }
            }
        }
    },
    "size": 0
}


{
    "took": 5,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 6,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "age_groupby": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 3999.0,
                    "doc_count": 4,
                    "group_sum": {
                        "value": 15996.0
                    }
                },
                {
                    "key": 2998.0,
                    "doc_count": 2,
                    "group_sum": {
                        "value": 5996.0
                    }
                }
            ]
        }
    }
}
```

## 3. Spring-data-elasticsearch

- pom

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

- Entity.java

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
// 标记该类为文档实体，index为indexname；副本数replicas，分片shards
@Document(indexName = "product01", replicas = 1, shards = 3)
public class Product {
    @Id() // 必须
    private Long id;//商品唯一标识
    /**
     * type : 字段数据类型
     * analyzer : 分词器类型 ik_max_word是中文最小粒度分词器
     * index : 是否索引(默认:true)
     * Keyword : 短语,不进行分词
     */
    @Field(type = FieldType.Text/*, analyzer = "ik_max_word"*/)
    private String title;//商品名称
    @Field(type = FieldType.Keyword)
    private String category;//分类名称
    @Field(type = FieldType.Double)
    private Double price;//商品价格
    @Field(type = FieldType.Keyword, index = false)
    private String images;//图片地址
}
```

- DAO

```java
@Repository
// 继承ElasticsearchRepository就可以完成一些简单的doc的CRUD
public interface ProductDAO extends ElasticsearchRepository<Product, Long> {

}
```

- RestHighLevelClient  高级查询用

```java
@Configuration
@ConfigurationProperties(prefix = "es")
@Setter
// AbstractElasticsearchConfiguration内部还有一个ElasticsearchOperations可以用来复杂操作
public class ESClientConfig  extends AbstractElasticsearchConfiguration {

    private String host;

    private Integer port;

    @Bean
    public RestHighLevelClient elasticsearchClient() {
        return new RestHighLevelClient(RestClient.builder(new HttpHost(host, port)));
    }
}
```

- Test

```java
package com.codeman.springdataesdemo;

import com.codeman.springdataesdemo.dao.ProductDAO;
import com.codeman.springdataesdemo.entity.Product;
import org.apache.lucene.index.Term;
import org.elasticsearch.common.unit.Fuzziness;
import org.elasticsearch.index.query.QueryBuilders;
import org.springframework.data.elasticsearch.core.ElasticsearchOperations;
import org.springframework.data.elasticsearch.core.SearchHits;
import org.springframework.data.elasticsearch.core.query.NativeSearchQuery;
import org.springframework.data.elasticsearch.core.query.Query;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.TermQueryBuilder;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.elasticsearch.core.ElasticsearchRestTemplate;
import org.springframework.data.elasticsearch.core.IndexOperations;
import org.springframework.data.elasticsearch.core.query.StringQuery;

import java.io.IOException;
import java.util.Arrays;
import java.util.Optional;

@SpringBootTest
class SpringDataEsDemoApplicationTests {

    @Autowired
    private ProductDAO productDAO;

    @Autowired
    private ElasticsearchRestTemplate elasticsearchRestTemplate;

    @Autowired
    private ElasticsearchOperations elasticsearchOperations;


    @Test
    void createIndex() {
        System.out.println("create index");
        elasticsearchRestTemplate.indexOps(Product.class).create();
    }

    @Test
    void deleteIndex() throws IOException {
//        elasticsearchClient.delete(new DeleteRequest().index("product01"), RequestOptions.DEFAULT);
//        elasticsearchRestTemplate.delete()
        elasticsearchRestTemplate.deleteIndex(Product.class);

        elasticsearchRestTemplate.indexOps(Product.class).delete();
    }

    @Test
    void createDoc() {
        /*"_source": {
            "_class": "com.codeman.springdataesdemo.entity.Product",  # 多存了个完全限定名，应该是序列化用的
                    "id": 1001,
                    "title": "标题1",
                    "category": "类别1",
                    "price": 1100.0,
                    "images": "http://www.baidu.com"
        }*/
        Product save = productDAO.save(new Product(1001L, "标题1", "类别1", 1100.00d, "http://www.baidu.com"));
        if (save.getId() != null) {
            System.out.println("保存成功");
        }
    }

    @Test
    void bulkCreateDoc() {
        productDAO.saveAll(Arrays.asList(
                new Product(1001L, "标题1", "类别1", 1100.00d, "http://www.baidu.com/1"),
                new Product(1002L, "标题2", "类别2", 1200.00d, "http://www.baidu.com/2"),
                new Product(1003L, "标题3", "类别3", 1300.00d, "http://www.baidu.com/3"),
                new Product(1004L, "标题4", "类别4", 1400.00d, "http://www.baidu.com/4")));

    }

    @Test
    void findDocById() {
        Optional<Product> product = productDAO.findById(1002L);
        product.ifPresent(System.out::println);
    }

    @Test
    void findAll() {
        Iterable<Product> products = productDAO.findAll();
        products.forEach(System.out::println);
    }

    @Test
    void findPageList() {
        int currPage = 0;
        int size = 2;
        Sort sort = Sort.by(Sort.Direction.DESC, "id");
        Pageable products = PageRequest.of(currPage, size, sort);
        Page<Product> products1 = productDAO.findAll(products);
        products1.forEach(System.out::println);
    }

    @Test
    void otherQuery() {

        int currPage = 0;
        int size = 2;
        Sort sort = Sort.by(Sort.Direction.DESC, "id");
        Pageable products = PageRequest.of(currPage, size, sort);
//        elasticsearchRestTemplate.search(new NativeSearchQuery(QueryBuilders.boolQuery()), Product.class);
//        elasticsearchRestTemplate.search(new NativeSearchQuery(QueryBuilders.termQuery()), Product.class);
//        elasticsearchRestTemplate.search(new NativeSearchQuery(QueryBuilders.multiMatchQuery()), Product.class);
//        elasticsearchRestTemplate.search(new NativeSearchQuery(QueryBuilders.rangeQuery()), Product.class);
//        elasticsearchRestTemplate.search(new NativeSearchQuery(QueryBuilders.moreLikeThisQuery()), Product.class);
        SearchHits<Product> search = elasticsearchRestTemplate.search(
                new NativeSearchQuery(QueryBuilders.fuzzyQuery("title", "题标").fuzziness(Fuzziness.ONE)),
                Product.class);
        search.getSearchHits().forEach(productSearchHit -> {
            Product content = productSearchHit.getContent();
            System.out.println(content);
        });
//        Iterable<Product> search = productDAO.search(QueryBuilders.termsQuery("title", "标题1"));
    }

    @Test
    void updateById() {
        productDAO.save(new Product(1001L, "标题1", "类别1", 1111.00d, "http://www.baidu.com"));
    }

    @Test
    void deleteById() {
        productDAO.deleteById(1001L);
    }

}

```

## 4. 集群配置

### 4.1 不可root用户

因为安全问题，**Elasticsearch 不允许 root 用户直接运行**，所以要创建新用户，在 root 用户中创建新用户 

> 创建用户 

```shell
useradd es #新增 es 用户 
passwd es #为 es 用户设置密码 
userdel -r es #如果错了，可以删除再加 
chown -R es:es /opt/module/es-cluster #文件夹所有者 
```

### 4.2 修改配置文件 

> **修改 elasticsearch.yml**

修改/opt/module/es/config/elasticsearch.yml 文件，分发文件 

\# 加入如下配置 

```shell
#集群名称 
cluster.name: cluster-es 
#节点名称，每个节点的名称不能重复 
node.name: node-1 
#ip 地址，每个节点的地址不能重复 尚硅谷技术之 Elasticsearch 
network.host: linux1 
#是不是有资格主节点 
node.master: true 
node.data: true 
http.port: 9200 
# head 插件需要这打开这两个配置 
http.cors.allow-origin: "*" 
http.cors.enabled: true 
http.max_content_length: 200mb 
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举 master 
cluster.initial_master_nodes: ["node-1"] 
#es7.x 之后新增的配置，节点发现 
discovery.seed_hosts: ["linux1:9300","linux2:9300","linux3:9300"] 
gateway.recover_after_nodes: 2 
network.tcp.keep_alive: true 
network.tcp.no_delay: true 
transport.tcp.compress: true 
#集群内同时启动的数据任务个数，默认是 2 个 
cluster.routing.allocation.cluster_concurrent_rebalance: 16 
#添加或删除节点及负载均衡时并发恢复的线程个数，默认 4 个 
cluster.routing.allocation.node_concurrent_recoveries: 16 
#初始化数据恢复时，并发恢复线程的个数，默认 4 个 
cluster.routing.allocation.node_initial_primaries_recoveries: 16 
```

> **修改limits.conf**

修改/etc/security/limits.conf ，分发文件 

```xml
# 在文件末尾中增加下面内容 
es soft nofile 65536 
es hard nofile 65536 
```

> **修改20-nproc.conf**

修改/etc/security/limits.d/20-nproc.conf，分发文件 

```xml
# 在文件末尾中增加下面内容 
es soft nofile 65536 
es hard nofile 65536 
* hard nproc 4096 
# 注：* 带表 Linux 所有用户名称 
```

> **修改sysctl.conf** 

修改/etc/sysctl.conf 

```xml
# 在文件中增加下面内容 
vm.max_map_count=655360 
```

>  **重新加载** 

sysctl -p  

分别在不同节点上启动 ES 软件 

cd /opt/module/es-cluster 

\#启动 

bin/elasticsearch 

\#后台启动 

bin/elasticsearch -d 

### 4.3 查看集群状态

```http
GET /_cat/nodes
172.17.0.2 59 61 2 0.07 0.03 0.05 dilmrt * aef0935c58d8
```

### 4.4 通过浏览器插件查看集群状态elasticsearch-head 



## 5. 进阶知识

### 5.1 核心概念

#### 5.1.1 索引（Index） 

一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（**必须全部是小写字母**），并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。 

能搜索的数据必须索引，这样的好处是可以提高查询速度，比如：新华字典前面的目录就是索引的意思，目录可以提高查询速度。 

**Elasticsearch 索引的精髓：一切设计都是为了提高搜索的性能。** 

#### 5.1.2 Type @Deparected

在一个索引中，你可以定义一种或多种类型。一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定。通常，会为具有一组共同字段的文档定义一个类型。不同的版本，类型发生了不同的变化

5.x  支持多种 type 

6.x 只能有一种 type 

**7.x**  默认不再支持自定义索引类型（默认类型为：_doc） 

#### 5.1.3 文档（Document） 

一个文档是一个可被索引的**基础信息单元**，也就是一条数据比如：你可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。**文档以 JSON（Javascript Object Notation）格式来表示**，而 JSON 是一个到处存在的互联网数据交互格式。 

#### 5.1.4 字段（Field） 

相当于是数据表的字段，对文档数据根据不同属性进行的分类标识。

#### 5.1.5 映射（Mapping）

mapping 是处理数据的方式和规则方面做**一些限制**，如：某个字段的**数据类型、默认值、分析器、约束/ 是否被索引等等**。这些都是映射里面可以设置的，其它就是处理 ES 里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。 

#### 5.1.6 分片（Shards） 

**一个索引可以存储超出单个节点硬件限制的大量数据**。比如，一个具有 10 亿文档数据的索引占据 1TB 的磁盘空间，而任一节点都可能没有这样大的磁盘空间。或者单个节点处理搜索请求，响应太慢。为了解决这个问题，**Elasticsearch 提供了将索引划分成多份的能力，每一份就称之为分片**。**当你创建一个索引的时候，你可以指定你想要的分片的数量**。每个分片本身也是一个**功能完善并且独立**的“索引”，这个“索引”可以被放置到集群中的任何节点上。 

分片很重要，主要有两方面的原因： 

1）**允许你水平分割 / 扩展你的内容容量。** 

2）允许你在分片之上进行**分布式的、并行的操**作，**进而提高性能/吞吐量。** 

至于一个分片怎样分布，它的文档怎样聚合和搜索请求，是完全由 Elasticsearch 管理的，对于作为用户的你来说，这些都是透明的，无需过分关心。 

被混淆的概念是，一个 Lucene 索引 我们在 Elasticsearch 称作 分片 。 一个Elasticsearch 索引 是分片的集合。 当 Elasticsearch 在索引中搜索的时候， 他发送查询到每一个属于索引的分片(Lucene 索引)，**然后合并**每个分片的结果到一个全局的结果集。 

#### 5.1.7 副本（Replicas） 

在一个网络 / 云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个**故障转移机制**是非常有用并且是强烈推荐的。为此目的，Elasticsearch 允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片(副本)。 

复制分片之所以重要，有两个主要原因： 

- 在分片/节点失败的情况下，提供了**高可用性**。因为这个原因，注意到**复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。(免得宕机主副同时丢失)**

- 扩展你的**搜索量/吞吐量**，因为**搜索可以在所有的副本上并行运行**。

#### 5.1.8 分配（Allocation） 

将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。这个过程是由 master 节点完成的。 

### 5.2 系统架构

![1623487887351](img\系统架构.png)

一个运行中的 Elasticsearch 实例称为一个节点，而集群是由一个或者多个拥有相同cluster.name 配置的节点组成，它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。 

当一个节点被选举成为**主节点**时， **它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等**。 而**主节点并不需要涉及到文档级别的变更和搜索等操作**，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。**作为用户，我们可以将请求发送到集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点**。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。

### 5.3 路由计算

**shard = hash(rounting) % number_of_primary_shards**

routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。 routing 通过hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置。 

这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。 

所有的文档 API（ get 、 index 、 delete 、 bulk 、 update 以及 mget ）都接受一个叫做 routing 的路由参数 ，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。 

### 5.4 分片控制

**我们可以发送请求到集群中的任一节点。每个节点都有能力处理任意请求(因此开发只需向其中某个节点写入？)**。每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上。 在下面的例子中，将所有的请求发送到 Node 1，我们将其称为 **协调节点(coordinating node) 。**

**当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点。（免得协调节点宕机）**  

#### 5.4.1 写流程

新建、索引和删除 请求都是 **写** 操作，必须在**主分片**上面完成之后才能被复制到相关的副本分片

![1623492181320](img\分片写入流程.png)

>  **新建，索引和删除文档所需要的步骤顺序：** 

1. 客户端向 Node 1 **发送新建、索引或者删除请求**。 

2. 节点使用文档的 _id **确定文档属于分片** 0 。请求会被**转发**到 Node 3，因为分片 0 的主分片目前被分配在 Node 3 上。 

3. Node 3 在主分片上面执行请求。如果成功了，它将**请求并行转发到 Node 1 和 Node 2 的副本分片上**。一旦所有的副本分片都报告成功, Node 3 将向协调节点报告成功，协调节点向客户端报告成功。 (类似ack=all)

在客户端收到成功响应时，文档变更已经在主分片和所有副本分片执行完成，变更是安全的。有一些可选的请求参数允许您影响这个过程，可能以数据安全为代价提升性能。这些选项很少使用，因为 Elasticsearch 已经很快，但是为了完整起见，请参考下面表格：

|             参数             |                             含义                             |
| :--------------------------: | :----------------------------------------------------------: |
| consistency (类似kafka的ACK) | consistency 参数的值可以设为 one （只要主分片状态 ok 就允许执行_写_操作）,all（必须要主分片和所有副本分片的状态没问题才允许执行_写_操作）, 或quorum 。默认值为 quorum , 即大多数的分片副本状态没问题就允许执行_写_操作。如果你的索引设置中指定了当前索引拥有三个副本分片，那规定数量的计算结果即：int( (primary + 3 replicas) / 2 ) + 1 = 3。如果此时你只启动两个节点，那么**处于活跃状态的分片副本数量就达不到规定数量，也因此您将<u>无法索引和删除任何文档</u>。** |
|           timeout            | 如果没有足够的副本分片会发生什么？ Elasticsearch 会等待，希望更多的分片出 现。默认情况下，它最多等待 1 分钟。 如果你需要，你可以使用 timeout 参数  使它更早终止： 100 100 毫秒，30s 是 30 秒。 |

新索引默认有 1 个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些 默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当 number_of_replicas 大于 1 的时候，规定数量才会执行。 

#### 5.4.2 读流程

![1623494569629](img\分片读取文档流程.png)

我们可以从主分片或者从其它任意副本分片检索文档 。

先请求到某个节点，节点计算出所属分片后，如果不是本节点会进行转发给对应节点，然后对应节点会返回数据给本节点，本节点再返回给客户端

#### 5.4.3 更新流程

![1623494531612](img\分片更新文档流程.png)

部分更新一个文档的步骤如下：

1. 客户端向 Node 1 发送更新请求。 

2. 它将请求转发到主分片所在的 Node 3 。 

3. Node 3 从主分片检索文档，修改 _source 字段中的 JSON ，并且**尝试**重新索引主分片的文档。如果文档已经**被另一个进程修改**，它会重试步骤 3 ，超过 **retry_on_conflict** 次后放弃。 

4. 如果 Node 3 成功地更新文档，它**将新版本的文档并行转发到 Node 1 和 Node 2 上的副本分片，重新建立索引。一旦所有副本分片都返回成功， Node 3 向协调节点也返回成功**，协调节点向客户端返回成功。 

当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，**它转发完整文档的新版本**( **如果 Elasticsearch 仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档**。 )。请记住，这些更改将会**异步**转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。

#### 5.4.4 多文档操作流程 bulk

bulk API 按如下步骤顺序执行： 

1. 客户端向 Node 1 发送 bulk 请求。 
2. Node 1 为每个节点创建一个批量请求，并将这些请求**并行转发**到每个包含主分片的节点主机。 
3. 主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行**转发新文档**（或 删除）到副本分片，然后执行下一个操作。 **一旦所有的副本分片报告所有操作成功， 该节点将向协调节点报告成功**，协调节点将这些响应收集整理并返回给客户端。

### 5.5 分片原理

传统的数据库每个字段存储单个值，但这对全文检索并不够。文本字段中的每个单词需要被搜索，对数据库意味着需要单个字段有索引多值的能力。**最好的支持是一个字段多个值需求的数据结构是倒排索引**。 

#### 5.5.1 倒排索引

Elasticsearch 使用一种称为倒排索引的结构，它适用于**快速的全文搜索**。 

见其名，知其意，有倒排索引，肯定会对应有正向索引。正向索引（forward index），反向索引（inverted index）更熟悉的名字是倒排索引。

所谓的正向索引，就是搜索引擎会将待搜索的文件都对应一个文件 ID，搜索时将这个ID 和搜索关键字进行对应，形成 K-V 对，然后对关键字进行统计计数（**传统关系型数据库，全文模糊匹配无法优化**）

但是互联网上收录在搜索引擎中的文档的数目是个天文数字，这样的索引结构根本无法满足实时返回排名结果的要求。所以，搜索引擎会将正向索引重新构建为倒排索引，**即把文件ID对应到关键词的映射转换为关键词到文件ID的映射，每个关键词都对应着一系列的文件**，这些文件中都出现这个关键词。(**即文章的每个关键字都可以作为索引的key，而这行的数据做为value[可以用id]，这样就可以按关键字查询了**)

**分词和标准化的过程称为分析**,可以提高查询准确性和减少索引创建

#### 5.5.2 文档搜索

早期的全文检索会为整个文档集合建立一个很大的倒排索引并将其写入到磁盘。 一旦新的索引就绪，旧的就会被其替换，这样最近的变化便可以被检索到。**倒排索引被写入磁盘后是 不可改变 的:它永远不会修改。** 

不变性有重要的价值： 

- **不需要锁**。如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。 

- 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。**只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘**。<u>这提供了很大的性能提升</u>。 

- 其它缓存(像 filter 缓存)，在索引的**生命周期内始终有效**。**它们不需要在每次数据改变时被重建**，因为数据不会变化。 

- **写入单个大的倒排索引允许数据被压缩**，减少磁盘 I/O 和 需要被缓存到内存的索引的使用量。

当然，一个不变的索引也有不好的地方。主要事实是它是不可变的! 你不能修改它。如果你需要让一个新的文档 可被搜索，你需要重建整个索引。这要么对一个索引所能包含的数据量造成了很大的限制，要么对索引可被更新的频率造成了很大的限制(不过可以动态更新索引)

#### 5.5.3 动态更新索引（新增索引补充再合并） 

>  **如何在保留不变性的前提下实现倒排索引的更新？** 

答案是: 用更多的索引。通过**增加新的补充索引来反映新近的修改**，而不是直接重写整个倒排索引。每一个倒排索引都会被轮流查询到，从最早的开始查询完后**再对结果进行合并**。Elasticsearch 基于 Lucene, 这个 java 库引入了按段搜索的概念。 每一 段 本身都是一个倒排索引， 但索引在 Lucene 中除表示所有段的集合外， 还增加了提交点的概念 — 一个列出了所有已知段的文件



当一个查询被触发，所有已知的段按顺序被查询。词项统计会对所有段的结果进行聚合，以保证每个词和每个文档的关联都被准确计算。 这种方式可以用相对较低的成本将新文档添加到索引。 

段是不可改变的，所以既不能从把文档从旧的段中移除，也不能修改旧的段来进行反映文档 的更新。 取而代之的是，**每个提交点会包含一个 .del 文件，文件中会列出这些被删除文档的段信息**。当一个文档被 “删除” 时，它实际上只是在 .del 文件中被 标记 删除。一个被标记删除的文档**仍然可以被查询匹配到， 但它会在最终结果被返回前从结果集中移除。** 

文档更新也是类似的操作方式：当一个文档被更新时，旧版本文档被标记删除，文档的新版本被索引到一个新的段中。 可能两个版本的文档都会被一个查询匹配到，但被删除的那个旧版本文档在结果集返回前就已经被移除。 

#### 5.5.4 近实时搜索 

随着按段（per-segment）搜索的发展，一个新的文档从索引到可被搜索的延迟显著降低了。新文档在几分钟之内即可被检索，但这样还是不够快。**磁盘在这里成为了瓶颈**。提交（Commiting）一个新的段到磁盘需要一个 fsync 来确保段被物理性地写入磁盘，这样在断 电的时候就不会丢失数据。 但是 fsync 操作代价很大; 如果每次索引一个文档都去执行一次的话会造成很大的性能问题。 

我们需要的是一个更轻量的方式来使一个文档可被搜索，这意味着 fsync 要从整个过程中被移除。在 Elasticsearch 和磁盘之间是文件系统缓存。 像之前描述的一样， 在内存索引缓冲区中的文档会被写入到一个新的段中。 但是这里新段会被先写入到文件系统缓存—这一步代价会比较低，稍后再被刷新到磁盘(DISK)—这一步代价比较高。不过只要文件已经在缓存中(OS Cache)，就可以像其它文件一样被打开和读取了。

![1623464332158](img\持久化过程.png)



在 Elasticsearch 中，写入和打开一个新段的轻量的过程叫做 refresh 。 默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 Elasticsearch 是 近 实时搜索: 文档的变化并不是立即对搜索可见，但会在一秒之内变为可见。这些行为可能会对新用户造成困惑: 他们索引了一个文档然后尝试搜索它，但却没有搜到。 

这个问题的解决办法是用 refresh API 执行一次手动刷新: /users/_refresh 

尽管刷新是比提交轻量很多的操作，它还是会有性能开销。当写测试的时候， 手动刷新很有用，但是不要在生产环境下每次索引一个文档都去手动刷新。 相反，你的应用需要意识到 Elasticsearch 的近实时的性质，并接受它的不足。 

并不是所有的情况都需要每秒刷新。可能你正在使用 Elasticsearch 索引大量的日志文件，你可能想优化索引速度而不是近实时搜索， 可以通过设置 refresh_interval ， 降低每个索引的刷新频率 

```txt
{
     "settings": {
   		  "refresh_interval": "30s" 
     } 
 }
```

refresh_interval 可以在既存索引上进行动态更新。 在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来

```txt
# 关闭自动刷新
PUT /users/_settings
{ "refresh_interval": -1 } 
# 每一秒刷新
PUT /users/_settings
{ "refresh_interval": "1s" }
```

#### 5.5.5 持久化变更

![1623464332158](img\持久化过程.png)

整个流程如下： 

1. 一个文档被索引之后，就会被添加到内存缓冲区，并且追加到了 translog 

2. 刷新（refresh）使分片每秒被刷新（refresh）一次： 
   - 这些在内存缓冲区的文档被写入到一个新的段中，且没有进行 fsync 操作。 
   - 这个段被打开，使其可被搜索 
   - 内存缓冲区被清空 
3. 这个进程继续工作，更多的文档被添加到内存缓冲区和追加到事务日志 
4. 每隔一段时间—例如 translog 变得越来越大—索引被刷新（flush）；一个新的 translog被创建，并且一个全量提交被执行 
   - 所有在内存缓冲区的文档都被写入一个新的段。 
   - 缓冲区被清空。 
   - 一个提交点被写入硬盘。 
   - 文件系统缓存通过 fsync 被刷新（flush）。 
   - 老的 translog 被删除。

#### 5.5.6 段合并

**由于自动刷新流程每秒会创建一个新的段 ，这样会导致短时间内的段数量暴增**。而段数目太多会带来较大的麻烦。 **每一个段都会消耗文件句柄、内存和 cpu 运行周期**。更重要 的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。Elasticsearch 通过在后台进行段合并来解决这个问题。**小的段被合并到大的段，然后这些大的段再被合并到更大的段。** 

**段合并的时候会将那些旧的已删除文档从文件系统中清除**。被删除的文档（或被更新文档的旧版本）不会被拷贝到新的大段中。 

启动段合并不需要你做任何事。进行索引和搜索时会自动进行。 

1. 当索引的时候，刷新（refresh）操作会创建新的段并将段打开以供搜索使用。 

2. 合并进程选择一小部分大小相似的段，并且在后台将它们合并到更大的段中。**这并不会** 

   **中断索引和搜索**。

3. **一旦合并结束，老的段被删除** 

   - 新的段被刷新（flush）到了磁盘。 ** 写入一个包含新段且排除旧的和较小的段的新提交点。 

   - 新的段被打开用来搜索。 

   - 老的段被删除。 

合并大的段需要消耗大量的 I/O 和 CPU 资源，如果任其发展会影响搜索性能。Elasticsearch在默认情况下会对合并流程进行资源限制，所以搜索仍然 有足够的资源很好地执行。 

### 5.6 文档分析 

分析 包含下面的过程：  

- 将一块文本分成适合于倒排索引的独立的词条 

- 将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall分析器执行上面的工作。分析器实际上是将三个功能封装到了一个包里： 

  - **字符过滤器** 

    首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉 HTML，或者将 & 转化成 and。 

  - **分词器** 

    其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。 

  - **Token 过滤器**

    最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化Quick ），删除词条（例如， 像 a， and， the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）。 

#### 5.6.1 内置分析器 

`"Set the shape to semi-transparent by calling set_trans(5)" `

1. **标准分析器** 

   标准分析器是 Elasticsearch 默认使用的分析器。它是分析各种语言文本最常用的选择。**它根据 Unicode 联盟 定义的 单词边界 划分文本**。**删除绝大部分标点。最后，将词条小写**。 

   它会产生： 

   set, the, shape, to, semi, transparent, by, calling, set_trans, 5

2. **简单分析器** 

   简单分析器在**任何不是字母的地方分隔文本**，将词条小写。它会产生： 

   set, the, shape, to, semi, transparent, by, calling, set, trans

3. **空格分析器** 

   空格分析器在空格的地方划分文本。它会产生： 

   Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

4. **语言分析器** 

   特定语言分析器可用于 很多语言。它们可以考虑指定语言的特点。例如， 英语 分析器附带了一组英语无用词（常用单词**，例如 and 或者 the ，它们对相关性没有多少影响**），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 词干 。 

   英语 分词器会产生下面的词条： 

   set, shape, semi, transpar, call, set_tran, 5 

   **注意看 transparent、 calling 和 set_trans 已经变为词根格式(还有格式转换)**

#### 5.6.2 分析器使用场景	

当我们 索引 一个文档，它的全文域被分析成词条以用来创建倒排索引。 但是，当我们在全文域 搜索 的时候，我们**需要将查询字符串通过 相同的分析过程** ，以保证我们搜索的词条格式与索引中的词条格式一致

全文查询，理解每个域是如何定义的，因此它们可以做正确的事： 

- 当你查询一个 全文 域时， 会对查询字符串应用相同的分析器，以产生正确的搜 索词条列表。 

- 当你查询一个 精确值 域时，不会分析查询字符串，而是搜索你指定的精确值。 

#### 5.6.3 测试分析器

有些时候很难理解分词的过程和实际被存储到索引中的词条，特别是你刚接触Elasticsearch。为了理解发生了什么，**你可以使用 analyze API 来看文本是如何被分析的**。在消息体里，指定分析器和要分析的文本

- 请求

```txt
GET http://localhost:9200/_analyze
{
 "analyzer": "standard",
 "text": "Text to analyze"
}
```

- 响应

```txt
{
 "tokens": [
     {
         "token": "text",
         "start_offset": 0,
         "end_offset": 4,
         "type": "<ALPHANUM>",
         "position": 1
     },
     {
         "token": "to",
         "start_offset": 5,
         "end_offset": 7,
         "type": "<ALPHANUM>",
         "position": 2
     },
     {
         "token": "analyze",
         "start_offset": 8,
         "end_offset": 15,
         "type": "<ALPHANUM>",
         "position": 3
 }
 ] }
```

token 是实际存储到索引中的词条。 position 指明词条在原始文本中出现的位置。start_offset 和 end_offset 指明字符在原始字符串中的位置。

#### 5.6.4 指定分析器

当Elasticsearch在你的文档中检测到一个新的字符串域，它会自动设置其为一个全文 字符串 域，使用 标准 分析器对它进行分析。你不希望总是这样。可能你**想使用一个不同的分析器，适用于你的数据使用的语言**。有时候你想要一个字符串域就是一个字符串域—不使用分析，直接索引你传入的精确值，例如用户 ID 或者一个内部的状态域或标签。要做到这一点，我们必须手动指定这些域的映射。 

##### 5.6.4.1 IK 分词器 (中文分词器 ) 

https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.8.0 

**将解压后的后的文件夹放入 ES 根目录下的 plugins 目录下，重启 ES 即可使用。**

我们这次加入新的查询参数"analyzer":"ik_max_word"

```txt
# GET http://localhost:9200/_analyze
{
"text":"测试单词",
"analyzer":"ik_max_word" // ik_max_word：会将文本做最细粒度的拆分
					   //  ik_smart：会将文本做最粗粒度的拆分
}
```

> **拓展分词器的词库**

首先进入 ES 根目录中的 plugins 文件夹下的 ik 文件夹，进入 config 目录，创建 custom.dic 文件，写入弗雷尔卓德。同时打开 IKAnalyzer.cfg.xml 文件，将新建的 custom.dic 配置其中，重启 ES 服务器。

##### 5.6.4.2 自定义分析器 

虽然 Elasticsearch 带有一些现成的分析器，然而在分析器上 Elasticsearch 真正的强大之处在于，你可以通过在一个适合你的特定数据的设置之中组合字符过滤器、分词器、词汇单元过滤器来创建自定义的分析器。在 分析与分析器 我们说过，一个 分析器 就是在一个包里面组合了三种函数的一个包装器， **三种函数按照顺序被执行:**

- **字符过滤器** 

字符过滤器 用来 整理 一个尚未被分词的字符串。例如，如果我们的文本是 HTML 格式的，它会包含像 <p> 或者 <div> 这样的 HTML 标签，这些标签是我们不想索引的。我们可以使用 html 清除 字符过滤器 来移除掉所有的 HTML 标签，并且像把 Á 转换为相对应的 Unicode 字符 Á 这样，转换 HTML 实体。一个分析器可能有 0 个或者多个字符过滤器。 

- **分词器** 

一个分析器 必须 有一个唯一的分词器。 分词器把字符串分解成单个词条或者词汇单元。 标准 分析器里使用的 标准 分词器 把一个字符串根据单词边界分解成单个词条，并且移除掉大部分的标点符号，然而还有其他不同行为的分词器存在。例如， 关键词 分词器 完整地输出 接收到的同样的字符串，并不做任何分词。 空格 分词器 只根据空格分割文本 。 正则 分词器 根据匹配正则表达式来分割文本 。 

- **词单元过滤器** 

经过分词，作为结果的 词单元流 会按照指定的顺序通过指定的词单元过滤器 。词单元过滤器可以修改、添加或者移除词单元。我们已经提到过 lowercase 和 stop 词过滤器 ，但是在 Elasticsearch 里面还有很多可供选择的词单元过滤器。词干过滤器 把单词 遏制 为 词干。 ascii_folding 过滤器移除变音符，把一个像 "très" 这样的词转换为 "tres" 。ngram 和 edge_ngram 词单元过滤器 可以产生 适合用于部分匹配或者自动补全的词单元。 

> **使用：**

```txt
# PUT http://localhost:9200/my_index
{
 "settings": {
 "analysis": {
 "char_filter": {
     "&_to_and": {
         "type": "mapping",
         "mappings": [ "&=> and "]
 }},
 "filter": {
     "my_stopwords": {
         "type": "stop",
         "stopwords": [ "the", "a" ]
 }},
 "analyzer": {
     "my_analyzer": {
         "type": "custom",
         "char_filter": [ "html_strip", "&_to_and" ],
         "tokenizer": "standard",
         "filter": [ "lowercase", "my_stopwords" ]
 }}
}}}
```

### 5.7 文档处理 

#### 5.7.1 文档冲突

当我们使用 index API 更新文档 ，可以一次性读取原始文档，做我们的修改，然后重新索引 整个文档 。 最近的索引请求将获胜：无论最后哪一个文档被索引，都将被唯一存储在 Elasticsearch 中。如果其他人同时更改这个文档，他们的更改将丢失。 （**比如库存超卖问题**）

**在数据库领域中，有两种方法通常被用来确保并发更新时变更不会丢失：** 

- **悲观并发控制** 

这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。 

- **乐观并发控制** 

Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何 解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。 

> **乐观并发控制** 

Elasticsearch 是分布式的。当文档创建、更新或删除时， 新版本的文档必须复制到集群中的其他节点。Elasticsearch 也是异步和并发的，这意味着这些复制请求被并行发送，并 且到达目的地时也许 顺序是乱的 。 Elasticsearch 需要一种方法确保文档的旧版本不会覆盖新的版本。 

当我们之前讨论 index ，GET 和 delete 请求时，我们指出每个文档都有一个 _version （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 version 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。我们可以利用 version 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 version 号来达到这个目的。 如果该版本不是当前版本号，我们的请求将会失败。 

**老的版本 es 使用 version，但是新版本不支持了，会报下面的错误，提示我们用 if_seq_no和 if_primary_term** 

> **外部系统版本控制** 

一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索， 这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch ，如果多个进程负责这一数据同步，你可能遇到类似于之前描述的并发问题。 

如果你的主数据库已经有了版本号 — 或一个能作为版本号的字段值比如 timestamp —那么你就可以在 Elasticsearch 中通过增加 version_type=external 到查询字符串的方式重用这些相同的版本号， 版本号必须是大于零的整数， 且小于 9.2E+18 — 一个 Java 中 long  类型的正值。 

外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同，Elasticsearch 不是检查当前 _version 和请求中指定的版本号是否相同， 而是检查当前 _version 是否 小于 指定的版本号。 如果请求成功，外部的版本号作为文档的新 _version 进行存储。

### 5.8 Kibana   

Kibana 是一个免费且开放的用户界面，能够让你对 Elasticsearch 数据进行可视化，并 让你在 Elastic Stack 中进行导航。你可以进行**各种操作，从跟踪查询负载，到理解请求如何流经你的整个应用**，都能轻松完成。 

下载地址：https://artifacts.elastic.co/downloads/kibana/kibana-7.8.0-windows-x86_64.zip 

## 6. Elasticsearch 优化

### 6.1 硬件选择 

**Elasticsearch 重度使用磁盘**，**你的磁盘能处理的吞吐量**越大，你的节点就越稳定。这里有一些优化磁盘 I/O 的技巧：

- **使用 SSD**。就像其他地方提过的， 他们比机械磁盘优秀多了。 

- 使用 RAID 0。条带化 RAID 会提高磁盘 I/O，代价显然就是当一块硬盘故障时整个就故障了。不要使用镜像或者奇偶校验 RAID 因为副本已经提供了这个功能。  

- 另外，使用多块硬盘，并允许 Elasticsearch 通过多个 path.data 目录配置把数据条带化分配到它们上面。 

- 不要使用远程挂载的存储，比如 NFS 或者 SMB/CIFS。这个引入的延迟对性能来说完全是背道而驰的。

### 6.2 分片策略

####  6.2.1 合理设置分片数

分片和副本的设计为 ES 提供了支持分布式和故障转移的特性，但并不意味着分片和 副本是可以无限分配的。而且**索引的分片完成分配后由于索引的路由机制，我们是不能重新修改分片数的。** 

> **多分区带来的问题(不是越多越好)**

- 一个分片的底层即为一个 Lucene 索引，会**消耗一定文件句柄、内存、以及 CPU 运转**。 
- 每一个搜索请求都需要命中索引中的每一个分片，如果每一个分片都处于不同的节点还好， 但如果多个分片都需要在同一个节点上**竞争使用相同的资源**就有些糟糕了。 
- **用于计算相关度的词项统计信息是基于分片的**。如果有许多分片，每一个都只有很少的数据会导致很低的相关度。

> **一些分片原则**

- 控制每个分片占用的硬盘容量不超过 ES 的最大 JVM 的堆空间设置（**一般设置不超过 32G**，参考下文 的 JVM 设置原则），因此，如果索引的总容量在 500G 左右，那分片大小在 16 个左右即可；当然， 最好同时考虑原则 2。 

- 考虑一下 node 数量，一般一个节点有时候就是一台物理机，如果分片数过多，大大超过了节点数，很可能会导致一个节点上存在多个分片，一旦该节点故障，即使保持了 1 个以上的副本，同样有可能 会导致数据丢失，集群无法恢复。所以， **一般都设置分片数不超过节点数的 3 倍。** 

- 主分片，副本和节点最大数之间数量，我们分配的时候可以参考以下关系： 

  **节点数<=主分片数*（副本数+1）** 

#### 6.2.2 推迟分片分配 

**对于节点瞬时中断的问题，默认情况，集群会等待一分钟来查看节点是否会重新加入，如果这个节点在此期间重新加入，重新加入的节点会保持其现有的分片数据，不会触发新的分片分配**。这样就可以减少 ES 在自动再平衡可用分片时所带来的极大开销。 

通过修改参数 **delayed_timeout ，可以延长再均衡的时间**，可以全局设置也可以在索引级别进行修改:

```txt
PUT /_all/_settings 
{
 "settings": {
 	"index.unassigned.node_left.delayed_timeout": "5m"  # 等待时间5分钟
 } }
```

### 6.3 路由选择

当我们查询文档的时候，一个文档应该存放到哪个分片

其实是通过下面这个公式来计算出来: 

**shard = hash(routing) % number_of_primary_shards** 

routing 默认值是文档的 id，也可以采用自定义值，比如用户 id。

- **不带 routing 查询(需要es计算)** 

  在查询的时候因为不知道要查询的数据具体在哪个分片上，所以整个过程分为 2 个步骤 

  - **分发**：请求到达协调节点后，协调节点将查询请求分发到每个分片上。 
  - **聚合**: 协调节点搜集到每个分片上查询结果，在将查询的结果进行排序，之后给用户返回结果。 

- **带 routing 查询(减少计算)** 

  查询的时候，可以直接根据 routing 信息定位到某个分配查询，不需要查询所有的分配，经过协调节点排序。 

  向上面自定义的用户查询，如果 routing 设置为 userid 的话，就可以直接查询出数据来，效率提升很多。

### 6.4  写入速度优化

ES 的默认配置，是综合了数据可靠性、写入速度、搜索实时性等因素。实际使用时，我们需要根据公司要求，进行偏向性的优化。 

针对于搜索性能要求不高，但是对写入要求较高的场景，我们需要尽可能的选择恰当写优化策略。综合来说，可以考虑以下几个方面来提升写索引的性能： 

- 加大 Translog Flush ，目的是降低 Iops、Writeblock。 

- 增加 Index Refresh 间隔，目的是减少 Segment Merge 的次数。 

- 调整 Bulk 线程池和队列。 

- 优化节点间的任务分布。 

- 优化 Lucene 层的索引建立，目的是降低 CPU 及 IO。 

#### 6.4.1 批量数据提交(批量减少频繁写入的开销) 

ES 提供了 Bulk API 支持批量操作，当我们有大量的写任务时，可以使用 Bulk 来进行批量写入。 

通用的策略如下：Bulk 默认设置批量提交的数据量不能超过 100M。数据条数一般是 根据文档的大小和服务器性能而定的，但是单次批处理的数据大小应从 5MB～15MB 逐渐增加，当性能没有提升时，把这个数据量作为最大值。 

#### 6.4.2 优化存储设备(硬件) 

ES 是一种密集使用磁盘的应用，在段合并的时候会频繁操作磁盘，所以对磁盘要求较高，当磁盘速度提升之后，集群的整体性能会大幅度提高。

#### 6.4.3 合理使用合并 

Lucene 以段的形式存储数据。当有新的数据写入索引时，Lucene 就会自动创建一个新的段。 

随着数据量的变化，段的数量会越来越多，消耗的多文件句柄数及 CPU 就越多，查询效率就会下降。 

由于 **Lucene 段合并的计算量庞大，会消耗大量的 I/O**，所以 ES 默认采用较保守的策略，让后台定期进行段合并

#### 6.4.4 减少 Refresh 的次数 

Lucene 在新增数据时，采用了延迟写入的策略，默认情况下索引的 refresh_interval 为1 秒。 

Lucene 将待写入的数据先写到内存中，超过 1 秒（默认）时就会触发一次 Refresh，然后 Refresh 会把内存中的的数据刷新到操作系统的文件缓存系统中。 

如果我们对搜索的实效性要求不高，可以将 Refresh 周期延长，例如 30 秒。 

这样还可以有效地**减少段刷新次数，但这同时意味着需要消耗更多的 Heap 内存。** 

#### 6.4.5 加大 Flush 设置 

Flush 的主要目的是把文件缓存系统中的段持久化到硬盘，当 **Translog 的数据量达到512MB 或者 30 分钟时，会触发一次 Flush。**

 **index.translog.flush_threshold_size 参数的默认值是 512MB**，我们进行修改。 增加参数值意味着文件缓存系统中可能需要存储更多的数据，所以我们需要为操作系统的文件缓存系统留下足够的空间。

####6.4.6 减少副本的数量 

ES 为了保证集群的可用性，提供了 Replicas（副本）支持，然而**每个副本也会执行分析、索引及可能的合并过程**，所以 Replicas 的数量会严重影响写索引的效率。 

当写索引时，需要把写入的数据都同步到副本节点，副本节点越多，写索引的效率就越慢。 

如 果 我 们 需 要 **大 批 量** 进 行 写 入 操 作 ， **可 以 先 禁 止 Replica 复 制 ， 设 置index.number_of_replicas: 0 关闭副本。在写入完成后，Replica 修改回正常的状态。**

### 6.5 内存设置 

ES 默认安装后设置的内存是 1GB，对于任何一个现实业务来说，这个设置都太小了。如果是通过解压安装的 ES，则在 ES 安装文件中包含一个 **jvm.option** 文件，添加如下命 令来设置 ES 的堆大小，Xms 表示堆的初始大小，Xmx 表示可分配的最大内存，都是 1GB。**确保 Xmx 和 Xms 的大小是相同的，其目的是为了能够在 Java 垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源，可以减轻伸缩堆大小带来的压力**。 假设你有一个 64G 内存的机器，按照正常思维思考，你可能会认为把 64G 内存都给ES 比较好，但现实是这样吗， 越大越好？虽然内存对 ES 来说是非常重要的，但是答案是否定的！ 

因为 ES 堆内存的分配需要满足以下两个原则： 

- **不要超过物理内存的 50%**：Lucene 的设计目的是把底层 OS 里的数据缓存到内存中。Lucene 的段是分别存储到单个文件中的，这些文件都是不会变化的，所以很利于缓存，同时操作系统也会把这些段文件缓存起来，以便更快的访问。如果我们设置的堆内存过大，Lucene 可用的内存将会减少，就会严重影响降低 Lucene 的全文本查询性能。 

- **堆内存的大小最好不要超过 32GB**：在 Java 中，所有对象都分配在堆上，然后有一个 Klass Pointer 指针指向它的类元数据。 

这个指针在 64 位的操作系统上为 64 位，64 位的操作系统可以使用更多的内存（2^64）。在 32 位的系统上为 32 位，32 位的操作系统的最大寻址空间为 4GB（2^32）。 

但是 64 位的指针意味着更大的浪费，因为你的指针本身大了。浪费内存不算，更糟糕的是，更大的指针在主内存和缓存器（例如 LLC, L1 等）之间移动数据的时候，会占用更多的带宽。 

**最终我们都会采用 31 G 设置** 

-Xms 31g 

**-Xmx 31g** 

假设你有个机器有 128 GB 的内存，你可以创建两个节点，每个节点内存分配不超过 32 GB。 也就是说不超过 64 GB 内存给 ES 的堆内存，剩下的超过 64 GB 的内存给 Lucene 

### 6.6 重要配置

|               参数名               |    参数值     |                             说明                             |
| :--------------------------------: | :-----------: | :----------------------------------------------------------: |
|            cluster.name            | elasticsearch | 配置 ES 的集群名称，默认值是 ES，建议改成与所  存数据相关的名称，**ES 会自动发现在同一网段下的  集群名称相同的节点** |
|             node.name              |    node-1     | 集群中的节点名，在同一个集群中不能重复。**节点的名称一旦设置，就不能再改变了**。当然，也可以设 置 成 服 务 器 的 主 机 名 称 ， 例 如node.name:${HOSTNAME} |
|            node.master             |     true      | 指定该节点**是否有资格被选举成为 Master 节点**，默认是 True，如果被设置为 True，则只是有资格成为Master 节点，具体能否成为 Master 节点，需要通过选举产生。 |
|             node.data              |     true      | 指定**该节点是否存储索引数据**，默认为 True。数据的增、删、改、查都是在 Data 节点完成的。 |
|       index.number_of_shards       |       1       | 设置都索引分片个数，默认是 1 片。也可以在创建索引时设置该值，具体设置为多大都值要根据数据量的大小来定。如果数据量不大，则设置成 1 时效率最高 |
|      index.number_of_replicas      |       1       | 设置默认的索引**副本个数，默认为 1 个**。副本数越多，集群的可用性越好，但是写索引时需要同步的数据越多。 |
|       transport.tcp.compress       |     true      |   设置在节点间**传输数据时是否压缩**，默认为 False，不压缩   |
| discovery.zen.minimum_master_nodes |       1       | 设置在选举 Master 节点时需要参与的最少的候选主节点数，默认为 1。如果使用默认值，则当网络不稳定时有可能会出现**脑裂**。合理的数值为 **(master_eligible_nodes/2)+1** ，其中master_eligible_nodes 表示集群中的候选主节点数 |
|     discovery.zen.ping.timeout     |      3s       | 设置在集群中自动**发现其他节点时 Ping 连接的超时时间**，默认为 3 秒。在较差的网络环境下需要设置得大一点，防止因误判该节点的存活状态而导致分片的转移 |

## 7. 面试题

## 8. 可视化工具kibana

