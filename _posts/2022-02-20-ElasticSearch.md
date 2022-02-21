---
layout: post
title: "ElasticSearch"
date: 2022-02-21 09:00:00 +0800 
categories: ElasticSearch
tag: Blog
---
* content
{:toc}
# ElasticSearch

## 1 概述

Elaticsearch，简称为es，es是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据;本身扩展性很好，可以扩展到上百台服务器，处理PB级别(大数据时代）的数据。es也使用java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是<mark>通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单</mark>。

据国际权威的数据库产品评测机构DB Engines的统计，在2016年1月，ElasticSearch已超过Solr等，成为排名第一的搜索引擎类应用。

## 2 ES和Solr

### 2.1 ElasticSearch简介

- Elasticsearch是一个**实时分布式搜索和分析引擎**。 它让你以前所未有的速度处理大数据成为可能。
- 它用于<mark>**全文搜索、结构化搜索、分析**</mark>以及将这三者混合使用:
- `维基百科`使用Elasticsearch提供**全文搜索**并**高亮关键字**,以及输入**实时搜索**(search-asyou-type)和**搜索纠错**(did-you-mean)等搜索建议功能。
- `英国卫报`使用Elasticsearch结合用户日志和社交网络数据提供给他们的编辑以实时的反馈,以便及时了解公众对新发表的文章的回应。
- `StackOverflow`结合全文搜索与地理位置查询,以及more-like-this功能来找到相关的问题和答案。
- `Github`使用Elasticsearch检索1300亿行的代码。
- 但是Elasticsearch不仅用于大型企业，它还让像`DataDog`以及`Klout`这样的创业公司将最初的想法变成可扩展的解决方案。
- Elasticsearch可以在你的笔记本上运行,也可以在数以百计的服务器上处理PB级别的数据。
- Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎。无论在开源还是专有领域, Lucene可被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。
  - 但是, **Lucene只是一个库**。 想要使用它,你必须使用Java来作为开发语言并将其直接集成到你的应用中,更糟糕的是, Lucene非常复杂,你需要深入了解检索的相关知识来理解它是如何工作的。
- Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能,但是它的**目的**是<mark>通过简单的**RESTful API**来隐藏Lucene的复杂性,从而让全文搜索变得简单。</mark>

### 2.2 Solr简介

- Solr是Apache下的一个顶级开源项目,采用Java开发,它是**基于Lucene的全文搜索服务器**。Solr提供了比Lucene更为**丰富的查询语言**,同时实现了**可配置**、**可扩展**，并**对索引、搜索性能进行了优化**
- Solr可以**独立运行**,运行在letty. Tomcat等这些Selrvlet容器中 , Solr 索引的实现方法很简单,<mark>用POST方法向Solr服务器发送一个描述Field及其内容的XML文档, Solr根据xml文档**添加、删除、更新**索引</mark>。Solr 搜索只需要发送HTTP GET请求,然后对Solr返回xml、json等格式的查询结果进行解析,组织页面布局。
- Solr不提供构建UI的功能, **Solr提供了一个管理界面,通过管理界面可以查询Solr的配置和运行情况。**
- Solr是基于lucene开发企业级搜索服务器,实际上就是封装了lucene.
- Solr是一个独立的企业级搜索应用服务器,它**对外提供类似于Web-service的API接口**。用户可以通过http请求,向搜索引擎服务器提交-定格式的文件,生成索引;也可以通过提出查找请求,并得到返回结果。

### 2.3 ElasticSearch与Solr比较

- 当单纯的对已有数据进行搜索时，Solr更快
  - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201124201341.png)
- 当实时建立索引时，Solr会产生io阻塞，查询性能较差，ElasticSearch具有明显的优势
  - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201124201508.png)
- 随着数据量的增加，Solr的搜索效率会变得更低，而ElasticSearch却没有明显的变化
  - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201124201629.png)
- 转变我们的搜索基础设施后从Solr ElasticSearch，我们看见一个即时~ 50x提高搜索性能！
  - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201124201712.png)

### 2.4 总结

1、**es**基本是**开箱即用**(解压就可以用!) ,非常简单。Solr安装略微复杂一丢丢!
2、**Solr 利用Zookeeper进行分布式管理**,而**Elasticsearch<mark>自身带有分布式协调管理功能</mark>。**
3、Solr 支持更多格式的数据,比如JSON、XML、 CSV , 而**Elasticsearch仅支持json文件格式**。
4、Solr 官方提供的功能更多,而Elasticsearch本身更注重于核心功能，高级功能多有第三方插件提供，例如图形化界面需要kibana友好支撑
5、**Solr 查询快,但更新索引时慢(即插入删除慢)** ，用于电商等查询多的应用;

- **ES建立索引快(即查询慢)** ，即**实时性查询快**，用于facebook新浪等搜索。
- Solr是传统搜索应用的有力解决方案，但Elasticsearch更适用于新兴的实时搜索应用。

6、Solr比较成熟，有一个更大，更成熟的用户、开发和贡献者社区，而Elasticsearch相对开发维护者较少,更新太快,学习使用成本较高。

## 3 ES核心概念理解

1、索引（ElasticSearch）

- 包多个分片

2、字段类型（映射）

- 字段类型映射（字段是整型，还是字符型…）

3、文档

4、分片（Lucene索引，倒排索引）

> elasticsearch是**面向文档**，关系型数据库和elasticsearch客观的对比！一切都是json

| Relational DB      | Elasticsearch   |
| ------------------ | --------------- |
| 数据库（database） | 索引（indices） |
| 表（tables）       | types           |
| 行（rows）         | documents       |
| 字段（columns）    | fields          |

**elasticsearch（集群）**中可以包含多个**索引（数据库）** , 每个索引中可以包含多个**类型（表）** , 每个类型下又包含多个**文档（行）** ,  每个文档中又包含多个**字段（列）**。

### 3.1 物理设计

elasticsearch在后台把**每个索引划分成多个分片**，每分分片可以在集群中的不同服务器间迁移

一个人就是一个集群! ，即**启动的ElasticSearch服务，默认就是一个集群，且默认集群名为elasticsearch**。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201124232258.png)

### 3.2 逻辑设计

一个索引类型中，包含多个文档，比如说文档1，文档2。当我们索引一篇文档时，可以通过这样的顺序找到它：索引 => 类型 => 文档ID ，通过这个组合我们就能索引到某个具体的文档。 注意：ID不必是整数，实际上它是个字符串。

### 3.3 文档(行)

之前说elasticsearch是面向文档的，那么就意味着**索引和搜索数据的最小单位是文档**，elasticsearch中，文档有几个重要属性:

- 自我包含，一篇文档同时包含字段和对应的值，也就是同时包含key:value !
- 可以是层次型的，一个文档中包含自文档，复杂的逻辑实体就是这么来的! {就是一个json对象 ! fastjson进行自动转换 !}
- 灵活的结构，文档不依赖预先定义的模式，我们知道关系型数据库中，要提前定义字段才能使用，在elasticsearch中，对于字段是非常灵活的，有时候,我们可以忽略该字段，或者动态的添加一个新的字段。

尽管我们可以随意的新增或者忽略某个字段，但是，每个字段的类型非常重要，比如一个年龄字段类型，可以是字符串也可以是整形。因为elasticsearch会保存字段和类型之间的映射及其他的设置。这种映射具体到每个映射的每种类型，这也是为什么在elasticsearch中，类型有时候也称为映射类型。

### 3.4 类型(表)

类型是文档的逻辑容器，就像关系型数据库一样，表格是行的容器。类型中对于字段的定义称为映射，比如name映射为字符串类型。我们说文档是无模式的，它们不需要拥有映射中所定义的所有字段，比如新增一个字段，那么elasticsearch是怎么做的呢?

- elasticsearch会自动的将新字段加入映射，但是这个字段的不确定它是什么类型，elasticsearch就开始猜，如果这个值是18，那么elasticsearch会认为它是整形。但是elasticsearch也可能猜不对，所以最安全的方式就是提前定义好所需要的映射，这点跟关系型数据库殊途同归了，先定义好字段，然后再使用，别整什么幺蛾子。

### 3.5 索引(库)

索引是映射类型的容器， elasticsearch中的索引是一个非常大的文档集合。 索引存储了映射类型的字段和其他设置。然后它们被存储到了各个分片上了。我们来研究下分片是如何工作的。

#### 3.5.1 物理设计：节点和分片 如何工作

创建新索引：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201125003116.png)

一个集群至少有一个节点，而一个节点就是一个elasricsearch进程，节点可以有多个索引默认的，如果你创建索引，那么索引将会有个5个分片(primary shard ,又称主分片)构成的，每一个主分片会有一个副本(replica shard，又称复制分片)。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201124234946.png)

上图是一个有3个节点的集群，可以看到主分片和对应的复制分片都不会在同一个节点内，这样有利于某个节点挂掉了，数据也不至于失。实际上，**一个分片是一个Lucene索引（<mark>一个ElasticSearch索引包含多个Lucene索引</mark>）** ，**一个包含倒排索引的文件目录，倒排索引的结构使得elasticsearch在不扫描全部文档的情况下，就能告诉你哪些文档包含特定的关键字**。不过，等等，倒排索引是什么鬼?

#### ==3.5.2 倒排索引==

简单说就是 按（文章关键字，对应的文档\<0个或多个\>）形式建立索引，根据关键字就可直接查询对应的文档（含关键字的），无需查询每一个文档，如下图

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201125003531.png)



## 4 IK分词器

分词：即把一段中文或者别的划分成一个个的关键字，我们在搜索时候会把自己的信息进行分词，会把数据库中或者索引库中的数据进行分词，然后进行一一个匹配操作，**默认的中文分词是将每个字看成一个词**（<mark>不使用用IK分词器的情况下</mark>），比如“我爱狂神”会被分为”我”，”爱”，”狂”，”神” ，这显然是不符合要求的，所以我们需要安装中文分词器ik来解决这个问题。

**IK提供了两个分词算法**: `ik_smart`和`ik_max_word` ,其中`ik_smart`为**最少切分**, `ik_max_word`为**最细粒度划分**!

![image-20220219195115140](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220219195115140.png)

`ik_max_word`：最细粒度划分（穷尽词库的可能）

![image-20220219195134983](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220219195134983.png)

### 添加自定义的词添加到扩展字典中

`elasticsearch目录/plugins/ik/config/IKAnalyzer.cfg.xml`

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201125020139.png)

打开 `IKAnalyzer.cfg.xml` 文件，扩展字典

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201125020519.png)

创建字典文件，添加字典内容

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201125020802.png)

## 5 Rest风格说明

**Rest风格是一种软件架构风格**，而不是标准,只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以**更简洁**，**更有层次**，**更易于实现缓存**等机制。

> **基本Rest命令**
>
> |      method      |                     url地址                     |          描述          |
> | :--------------: | :---------------------------------------------: | :--------------------: |
> | PUT（创建,修改） |     localhost:9200/索引名称/类型名称/文档id     | 创建文档（指定文档id） |
> |   POST（创建）   |        localhost:9200/索引名称/类型名称         | 创建文档（随机文档id） |
> |   POST（修改）   | localhost:9200/索引名称/类型名称/文档id/_update |        修改文档        |
> |  DELETE（删除）  |     localhost:9200/索引名称/类型名称/文档id     |        删除文档        |
> |   GET（查询）    |     localhost:9200/索引名称/类型名称/文档id     |   查询文档通过文档ID   |
> |   POST（查询）   | localhost:9200/索引名称/类型名称/文档id/_search |      查询所有数据      |

> 基础语法测试

1. 创建一个索引！

   ```
   PUT /test1/type1/1
   {
     "name" : "流柚",
     "age" : 18
   }
   ```

   ![image-20220219203016729](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220219203016729.png)

   ![image-20220219203052618](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220219203052618.png)

2. **字段数据类型**

   - 字符串类型

     - text、

       keyword

       - text：支持分词，全文检索,支持模糊、精确查询,不支持聚合,排序操作;text类型的最大支持的字符长度无限制,适合大字段存储；
       - keyword：不进行分词，直接索引、支持模糊、支持精确匹配，支持聚合、排序操作。keyword类型的最大支持的长度为——32766个UTF-8类型的字符,可以通过设置ignore_above指定自持字符长度，超过给定长度后的数据将不被索引，无法通过term精确匹配检索返回结果。

   - 数值型

     - long、Integer、short、byte、double、float、**half float**、**scaled float**

   - 日期类型

     - date

   - te布尔类型

     - boolean

   - 二进制类型

     - binary

   - 等等…

3. > 创建索引的规则

   ```
   PUT /test2
   {
     "mappings": {
       "properties": {
         "name": {
           "type": "text"
         },
         "age":{
           "type": "long"
         },
         "birthday":{
           "type": "date"
         }
       }
     }
   }
   ```

   ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201201144947.png)

4. > **获取建立的规则**

   ```
   GET test2
   ```

   ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201201144955.png)

5. > **获取默认信息**

   ```
   _doc 默认类型（default type），type 在未来的版本中会逐渐弃用，因此产生一个默认类型进行代替
   
   PUT /test3/_doc/1
   {
     "name": "流柚",
     "age": 18,
     "birth": "1999-10-10"
   }
   GET test3
   ```

   ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201201145002.png)

   > 扩展：通过`get _cat/` 可以获取ElasticSearch的当前的很多信息！
   >
   > ```
   > GET _cat/indices
   > GET _cat/aliases
   > GET _cat/allocation
   > GET _cat/count
   > GET _cat/fielddata
   > GET _cat/health
   > GET _cat/indices
   > GET _cat/master
   > GET _cat/nodeattrs
   > GET _cat/nodes
   > GET _cat/pending_tasks
   > GET _cat/plugins
   > GET _cat/recovery
   > GET _cat/repositories
   > GET _cat/segments
   > GET _cat/shards
   > GET _cat/snapshots
   > GET _cat/tasks
   > GET _cat/templates
   > GET _cat/thread_pool
   > ```

6. > **修改**

   - 方式一：覆盖

     - 使用put覆盖原来的值

       - 缺点：

       - > - 版本+1（_version）
         > - 但是如果漏掉某个字段没有写，那么更新是没有写的字段 ，会消失

     - ```
       PUT /test3/_doc/1
       {
         "name" : "流柚是我的大哥",
         "age" : 18,
         "birth" : "1999-10-10"
       }
       GET /test3/_doc/1
       // 修改会有字段丢失
       PUT /test3/_doc/1
       {
         "name" : "流柚"
       }
       GET /test3/_doc/1
       ```

     - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201201145010.png)

     - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201201160911.png)

   - 方式二：使用post的update

     - > - version不会改变
       > - 需要注意doc
       > - 不会丢失字段

     - ```
       POST /test3/_doc/1/_update
       {
         "doc":{
           "name" : "post修改，version不会加一",
           "age" : 2
         }
       }
       GET /test3/_doc/1
       ```

     - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201201163030.png)

7. 删除

   - ```
     GET /test1
     DELETE /test1
     ```

   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203150602.png)

   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203150557.png)

## 6 关于文档的操作

### 6.1 基础操作

```
GET /test3/_doc/_search?q=name:流柚
```

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201202204347.png)

### 6.2 复杂操作

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203150547.png)

1. **查询匹配**

   - `match`：匹配（会使用分词器解析（先分析文档，然后进行查询））

   - `_source`：过滤字段

   - `sort`：排序

   - `form`、`size` 分页

     ```java
       // 查询匹配
       GET /blog/user/_search
       {
         "query":{
           "match":{
             "name":"流"
           }
         }
         ,
         "_source": ["name","desc"]
         ,
         "sort": [
           {
             "age": {
               "order": "asc"
             }
           }
         ]
         ,
         "from": 0
         ,
         "size": 1
       }
     ```

     ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203002017.png)

2. ##### 多条件查询（bool）

   - `must` 相当于 `and`

   - `should` 相当于 `or`

   - `must_not` 相当于 `not (... and ...)`

   - `filter` 过滤

   - ```json
     /// bool 多条件查询
     //// must <==> and
     //// should <==> or
     //// must_not <==> not (... and ...)
     //// filter数据过滤
     //// boost
     //// minimum_should_match
     GET /blog/user/_search
     {
       "query":{
         "bool": {
           "must": [
             {
               "match":{
                 "age":3
               }
             },
             {
               "match": {
                 "name": "流"
               }
             }
           ],
           "filter": {
             "range": {
               "age": {
                 "gte": 1,
                 "lte": 3
               }
             }
           }
         }
       }
     }
     ```

   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203150628.png)

3. ##### 匹配数组

   - 貌似不能与其它字段一起使用

   - 可以多关键字查（空格隔开）— 匹配字段也是符合的

   - `match` 会使用分词器解析（先分析文档，然后进行查询）

   - 搜词

   - ```json
     // 匹配数组 貌似不能与其它字段一起使用
     // 可以多关键字查（空格隔开）
     // match 会使用分词器解析（先分析文档，然后进行查询）
     GET /blog/user/_search
     {
       "query":{
         "match":{
           "desc":"年龄 牛 大"
         }
       }
     }
     ```

   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203150635.png)

4. 精确查询

   - `term` 直接通过 倒排索引 指定**词条**查询

   - 适合查询 number、date、keyword ，不适合text

   - ```json
     // 精确查询（必须全部都有，而且不可分，即按一个完整的词查询）
     // term 直接通过 倒排索引 指定的词条 进行精确查找的
     GET /blog/user/_search
     {
       "query":{
         "term":{
           "desc":"年 "
         }
       }
     }
     ```

   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203150641.png)

5. ##### text和keyword

   - text：

     - **支持分词**，**全文检索**、支持模糊、精确查询,不支持聚合,排序操作;
     - text类型的最大支持的字符长度无限制,适合大字段存储；

   - keyword：

     - **不进行分词**，**直接索引**、支持模糊、支持精确匹配，支持聚合、排序操作。
     - keyword类型的最大支持的长度为——32766个UTF-8类型的字符,可以通过设置ignore_above指定自持字符长度，超过给定长度后的数据将不被索引，**无法通过term精确匹配检索返回结果**。

   - ```json
     // 测试keyword和text是否支持分词
     // 设置索引类型
     PUT /test
     {
       "mappings": {
         "properties": {
           "text":{
             "type":"text"
           },
           "keyword":{
             "type":"keyword"
           }
         }
       }
     }
     // 设置字段数据
     PUT /test/_doc/1
     {
       "text":"测试keyword和text是否支持分词",
       "keyword":"测试keyword和text是否支持分词"
     }
     // text 支持分词
     // keyword 不支持分词
     GET /test/_doc/_search
     {
       "query":{
        "match":{
           "text":"测试"
        }
       }
     }// 查的到
     GET /test/_doc/_search
     {
       "query":{
        "match":{
           "keyword":"测试"
        }
       }
     }// 查不到，必须是 "测试keyword和text是否支持分词" 才能查到
     GET _analyze
     {
       "analyzer": "keyword",
       "text": ["测试liu"]
     }// 不会分词，即 测试liu
     GET _analyze
     {
       "analyzer": "standard",
       "text": ["测试liu"]
     }// 分为 测 试 liu
     GET _analyze
     {
       "analyzer":"ik_max_word",
       "text": ["测试liu"]
     }// 分为 测试 liu
     ```

6. **高亮查询**

   - ```json
     /// 高亮查询
     GET blog/user/_search
     {
       "query": {
         "match": {
           "name":"流"
         }
       }
       ,
       "highlight": {
         "fields": {
           "name": {}
         }
       }
     }
     // 自定义前缀和后缀
     GET blog/user/_search
     {
       "query": {
         "match": {
           "name":"流"
         }
       }
       ,
       "highlight": {
         "pre_tags": "<p class='key' style='color:red'>",
         "post_tags": "</p>", 
         "fields": {
           "name": {}
         }
       }
     }
     ```

   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201203150652.png)

   

## 7 继承SpringBoot

官方文档：https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html。

> **注意：**
>
> ​	注意依赖版本需要和本地的版本一致
>
> ![image-20220219222454657](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220219222454657.png)



### 7.1 索引API

- 测试创建索引

  - ```java
        @Autowired
        RestHighLevelClient restHighLevelClient;
    
        // 测试索引的创建
        @Test
        void testCreateIndex() throws IOException {
            /*1. 创建索引请求*/
            CreateIndexRequest request = new CreateIndexRequest("test_index");
            /*2. 执行请求响应*/
            CreateIndexResponse createIndexResponse = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
            System.out.println(createIndexResponse);
        }
    ```

  - ![image-20220220105302822](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220105302822.png)

- 测试删除索引

  - ```java
        /*测试删除索引*/
        @Test
        void testDeleteIndex() throws IOException {
            DeleteIndexRequest request = new DeleteIndexRequest("test_index");
            AcknowledgedResponse delete = restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
            System.out.println(delete.isAcknowledged());
        }
    ```

  - ![image-20220220105352664](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220105352664.png)

### 7.2 文档API

1. 添加文档

   - ```java
         /*测试添加文档*/
         @Test
         public void testAddDocument() throws IOException {
             // 创建一个User对象
             User user = new User("刘云杰", 23);
             // 创建请求
             IndexRequest request = new IndexRequest("test_index");
             // 制定规则 PUT /liuyou_index/_doc/1
             request.id("1");// 设置文档ID
             request.timeout(TimeValue.timeValueMillis(1000));// request.timeout("1s")
             // 将我们的数据放入请求中
             request.source(JSON.toJSONString(user), XContentType.JSON);
             // 客户端发送请求，获取响应的结果
             IndexResponse response = restHighLevelClient.index(request, RequestOptions.DEFAULT);
             System.out.println(response.status());// 获取建立索引的状态信息
             System.out.println(response);// 查看返回内容
     
             /*
             * CREATED
             * IndexResponse[index=test_index,type=_doc,id=1,version=1,result=created,seqNo=0,primaryTerm=1,shards={"total":2,"successful":1,"failed":0}]
             * */
         }
     ```

   - ![image-20220220142503679](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220142503679.png)

2. 获取文档信息

   - ```java
     // 测试获得文档信息
     @Test
     public void testGetDocument() throws IOException {
         GetRequest request = new GetRequest("test_index","1");
         GetResponse response = restHighLevelClient.get(request, RequestOptions.DEFAULT);
         System.out.println(response.getSourceAsString());// 打印文档内容
         System.out.println(request);// 返回的全部内容和命令是一样的
         restHighLevelClient.close();
     }
     ```

   - ![image-20220220142723402](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220142723402.png)

3. 判断文档是否存在

   - ```java
         /*获取文档*/
         @Test
         void testIsExists() throws IOException {
             GetRequest request = new GetRequest("test_index", "1");
             // 不获取返回的 _source的上下文了
             request.fetchSourceContext(new FetchSourceContext(false));
             request.storedFields("_none_");
     
             boolean exists = restHighLevelClient.exists(request, RequestOptions.DEFAULT);
             System.out.println(exists);
         }
     ```

   - ![image-20220220142823082](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220142823082.png)

4. 更新文档

   - ```java
         // 测试更新文档内容
         @Test
         public void testUpdateDocument() throws IOException {
             UpdateRequest request = new UpdateRequest("test_index", "1");
             User user = new User("lmk",11);
             request.doc(JSON.toJSONString(user),XContentType.JSON);
             UpdateResponse response = restHighLevelClient.update(request, RequestOptions.DEFAULT);
             System.out.println(response.status()); // OK
             restHighLevelClient.close();
         }
     ```

   - ![image-20220220142937230](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220142937230.png)

   - ![image-20220220143157225](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220143157225.png)

5. 删除文档

   - ```JAVA
         // 测试删除文档
         @Test
         public void testDeleteDocument() throws IOException {
             DeleteRequest request = new DeleteRequest("test_index", "1");
             request.timeout("1s");
             DeleteResponse response = restHighLevelClient.delete(request, RequestOptions.DEFAULT);
             System.out.println(response.status());// OK
         }
     ```

   - ![image-20220220143316614](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220143316614.png)

   - ![image-20220220143334762](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220143334762.png)

6. 查询文档

   - ```java
     // 查询
     // SearchRequest 搜索请求
     // SearchSourceBuilder 条件构造
     // HighlightBuilder 高亮
     // TermQueryBuilder 精确查询
     // MatchAllQueryBuilder
     // xxxQueryBuilder ...
     @Test
     public void testSearch() throws IOException {
         // 1.创建查询请求对象
         SearchRequest searchRequest = new SearchRequest();
         // 2.构建搜索条件
         SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
         // (1)查询条件 使用QueryBuilders工具类创建
         // 精确查询
         TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name", "liuyou");
         //        // 匹配查询
         //        MatchAllQueryBuilder matchAllQueryBuilder = QueryBuilders.matchAllQuery();
         // (2)其他<可有可无>：（可以参考 SearchSourceBuilder 的字段部分）
         // 设置高亮
         searchSourceBuilder.highlighter(new HighlightBuilder());
         //        // 分页
         //        searchSourceBuilder.from();
         //        searchSourceBuilder.size();
         searchSourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
         // (3)条件投入
         searchSourceBuilder.query(termQueryBuilder);
         // 3.添加条件到请求
         searchRequest.source(searchSourceBuilder);
         // 4.客户端查询请求
         SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
         // 5.查看返回结果
         SearchHits hits = search.getHits();
         System.out.println(JSON.toJSONString(hits));
         System.out.println("=======================");
         for (SearchHit documentFields : hits.getHits()) {
             System.out.println(documentFields.getSourceAsMap());
         }
     }
     ```

7. 批量添加数据

   - ```java
         // 特殊的，真的项目一般会 批量插入数据
         @Test
         public void testBulk() throws IOException {
             BulkRequest bulkRequest = new BulkRequest();
             bulkRequest.timeout("10s");
             ArrayList<User> users = new ArrayList<>();
             users.add(new User("liuyou-1",1));
             users.add(new User("liuyou-2",2));
             users.add(new User("liuyou-3",3));
             users.add(new User("liuyou-4",4));
             users.add(new User("liuyou-5",5));
             users.add(new User("liuyou-6",6));
             // 批量请求处理
             for (int i = 0; i < users.size(); i++) {
                 bulkRequest.add(
                         // 这里是数据信息
                         new IndexRequest("bulk")
                                 .id(""+(i + 1)) // 没有设置id 会自定生成一个随机id
                                 .source(JSON.toJSONString(users.get(i)),XContentType.JSON)
                 );
             }
             BulkResponse bulk = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
             System.out.println(bulk.status());// ok
         }
     ```

   - ![image-20220220143745058](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220143745058.png)

## 8 ElasticSearch实战

> **防京东商城搜索（高亮）**



### 8.1 爬虫

```java
package com.lyj.esjd.utils;

import com.lyj.esjd.pojo.Content;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.io.IOException;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;

/**
 * 数据获取：数据库、消息队列、爬虫、…
 */
public class HtmlParseUtil {

    public static void main(String[] args) throws IOException {

        new HtmlParseUtil().parseJD("java").forEach(System.out::println);
    }

    private List<Content> parseJD(String keyword) throws IOException {
//        String url = "https://search.jd.com/Search?keyword=java&enc=utf-8&wq=java&pvid=3cf31181476a4799aa2025cf0080f71c";
        String url = "https://search.jd.com/Search?keyword=" + keyword + "&enc=utf-8";
        //  解析网页
        Document parse = Jsoup.parse(new URL(url), 30000);
        Element element = parse.getElementById("J_goodsList");

        System.out.println(element.html());

        List<Content> goodsList = new ArrayList<>();

        /*获取所有的li元素*/
        Elements li = element.getElementsByTag("li");
        for (Element element1 : li) {
            /**
             * 关于图片特别多的网站， 图片一般都是延迟加载的
             */
//            String img = element1.getElementsByTag("src").eq(0).attr("src");
            String img = element1.getElementsByTag("img").eq(0).attr("data-lazy-img");
            String price = element1.getElementsByClass("p-price").eq(0).text();
            String title = element1.getElementsByClass("p-name").eq(0).text();

            goodsList.add(new Content(img, price, title));

        }
        return goodsList;
    }

}

```



### 8.2 业务编写

#### 8.2.1 查询数据，存储到es

`ContentService`

```java
    @Autowired
    private RestHighLevelClient restHighLevelClient;

    /**
     * 查询数据存储到 es
     *
     * @param keywords
     * @return
     * @throws IOException
     */
    public Boolean parseContent(String keywords) throws IOException {
        List<Content> contents = new HtmlParseUtil().parseJD(keywords);
        /*把查询到的数据放入es中*/
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.timeout("2m");

        for (int i = 0; i < contents.size(); i++) {
            bulkRequest.add(new IndexRequest("jd_goods")
                    .source(JSON.toJSONString(contents.get(i)), XContentType.JSON));
        }
        ;
        BulkResponse bulk = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        return !bulk.hasFailures();
    }
```

`ContentController`

```java
    @Autowired
    private ContentService contentService;

    @GetMapping("/parse/{keyword}")
    public Boolean parse(@PathVariable("keyword") String keyword) throws IOException {
        return contentService.parseContent(keyword);
    }
```

结果：



![image-20220220230847168](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220230847168.png)

![image-20220220230912811](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220230912811.png)



#### 8.2.2 从es中获取数据

`ContentService`

```java
    @Autowired
    private RestHighLevelClient restHighLevelClient;

    /**
     * 从es中获取数据
     *
     * @param keyword
     * @return
     */
    public List<Map<String, Object>> searchPage(String keyword, int pageNo, int pageSize) throws IOException {
        if (pageNo <= 1) {
            pageNo = 1;
        }
        /*条件搜索*/
        SearchRequest searchRequest = new SearchRequest("jd_goods");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        /*分页*/
        sourceBuilder.from(pageNo);
        sourceBuilder.size(pageSize);

        /*精准匹配*/
        TermQueryBuilder termQuery = QueryBuilders.termQuery("title", keyword);
        sourceBuilder.query(termQuery);
        sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

        /*执行查询*/
        searchRequest.source(sourceBuilder);
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        /*解析结果*/
        List<Map<String, Object>> list = new ArrayList<>();
        for (SearchHit documentFields : search.getHits().getHits()) {
            list.add(documentFields.getSourceAsMap());
        }
        return list;
    }
```

`ContnetController`

```java
    @Autowired
    private ContentService contentService;

    @GetMapping("/search/{keyword}/{pageNo}/{pageSize}")
    public List<Map<String, Object>> search(@PathVariable("keyword") String keyword,
                                            @PathVariable("pageNo") int pageNo,
                                            @PathVariable("pageSize") int pageSize) throws IOException {
        List<Map<String, Object>> lists = contentService.searchPage(keyword, pageNo, pageSize);
        return lists;
    }
```

结果：

![image-20220220233655544](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220220233655544.png)

### 8.3 前后端交互

- 引入 `vue.min.js` 与 `axios.min.js`

  - ```html
    <!--前端使用Vue 实现前后端分离-->
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    ```

- 实现绑定

  - ```html
    <div class="page" id="app">
        ...
        ...
        ...
        <button type="submit" @click.prevent="searchKey()" id="searchbtn">搜索</button>
        ...
        ...
        ...
        <div class="product" v-for="result in results">
            <div class="product-iWrap">
                <!--商品封面-->
                <div class="productImg-wrap">
                    <a class="productImg">
                        <img :src="result.img">
                    </a>
                </div>
                <!--价格-->
                <p class="productPrice">
                    <em>{{result.price}}</em>
                </p>
                <!--标题-->
                <p class="productTitle">
                    <a> {{result.title}} </a>
                </p>
                <!-- 店铺名 -->
                <div class="productShop">
                    <span>店铺： 狂神说Java </span>
                </div>
                <!-- 成交信息 -->
                <p class="productStatus">
                    <span>月成交<em>999笔</em></span>
                    <span>评价 <a>3</a></span>
                </p>
            </div>
        </div>
        ...
        ...
    
    <script>
        new Vue({
            el: '#app',
            data: {
                keyword: '', // 搜索的关键字
                results: []  // 搜索的结果
            },
            methods: {
                searchKey() {
                    let keyword = this.keyword;
                    console.log(keyword);
                    axios.get('search/' + keyword + "/1/10").then(response=>{
                        console.log(response);
                        this.results = response.data;
                    })
                }
            }
        })
    </script>
    ```

结果：

![image-20220221154519037](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221154519037.png)

![image-20220221154558833](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221154558833.png)

### 8.4 关键字高亮

`ContentService`

```java
    /**
     * 从es中获取数据
     *
     * @param keyword
     * @return
     */
    public List<Map<String, Object>> searchPageHighlight(String keyword, int pageNo, int pageSize) throws IOException {
        if (pageNo <= 1) {
            pageNo = 1;
        }
        /*条件搜索*/
        SearchRequest searchRequest = new SearchRequest("jd_goods");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        /*分页*/
        sourceBuilder.from(pageNo);
        sourceBuilder.size(pageSize);

        /*精准匹配*/
        TermQueryBuilder termQuery = QueryBuilders.termQuery("title", keyword);
        sourceBuilder.query(termQuery);
        sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

        /**
         * 高亮
         */
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("title");
        highlightBuilder.requireFieldMatch(false);
        highlightBuilder.preTags("<span style='color: red>'");
        highlightBuilder.postTags("</span>");
        sourceBuilder.highlighter(highlightBuilder);

        /*执行查询*/
        searchRequest.source(sourceBuilder);
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        /*解析结果*/
        List<Map<String, Object>> list = new ArrayList<>();
        for (SearchHit documentFields : search.getHits().getHits()) {
            /**
             * 解析高亮的字段
             */
            Map<String, HighlightField> highlightFields = documentFields.getHighlightFields();
            HighlightField title = highlightFields.get("title");
            Map<String, Object> sourceAsMap = documentFields.getSourceAsMap(); /*原来的结果*/
            /*解析高亮的字段 将原来的字段换为我们高亮的字段*/
            if (title!=null) {
                Text[] texts = title.fragments();
                StringBuilder newTitle = new StringBuilder();
                for (Text text : texts) {
                    newTitle.append(text);
                }
                sourceAsMap.put("title", newTitle.toString());/*高亮字段替换原来的内容*/
            }
            list.add(documentFields.getSourceAsMap());
        }
        return list;
    }
```

> 出现问题：高亮未解析



![image-20220221160420101](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221160420101.png)

> 解决方法：
>
> ```html
> <!--标题-->
> <p class="productTitle">
>     <a v-html="result.title"></a>
> </p>
> ```

![image-20220221163434071](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221163434071.png)