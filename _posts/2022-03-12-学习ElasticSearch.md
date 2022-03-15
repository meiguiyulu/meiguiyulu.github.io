# ElasticSearch

> 参考链接：
>
> - [视频](https://www.bilibili.com/video/BV1hh411D7sb)
> - [笔记](https://blog.csdn.net/u011863024/article/details/115721328)

## 1. 倒排索引

`Elasticsearch` 是**面向文档型数据库**，一条数据在这里就是一个文档。 为了方便大家理解，我们将 `Elasticsearch` 里存储文档数据和关系型数据库 `MySQL` 存储数据的概念进行一个类比：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/146a779da01f53e7f7a8d53132d3c7cf.png)

> 倒排索引

简单说就是按 `(文章关键字，对应的文档id<0个或多个>)` 形式建立索引，根据关键字就可直接查询对应的文档（含关键字的），无需查询每一个文档，如下图

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20201125003531.png)

## 2. 入门

### 2.1 索引创建

对比关系型数据库，创建索引就等同于创建数据库。

在 Postman 中，向 ES 服务器发 PUT 请求 ： `http://127.0.0.1:9200/shopping`

```json
{
    "acknowledged": true,//响应结果
    "shards_acknowledged": true,//分片结果
    "index": "shopping"//索引名称
}
```

如果重复发 PUT 请求 ： `http://127.0.0.1:9200/shopping` 添加索引，会返回错误信息。

- `PUT` 具有幂等性，每一次请求结果相同；
- `POST` 不具有幂等性，每一次请求结果不同。

### 2.2 索引删除

- 查看所有索引

  - `http://127.0.0.1:9200/_cat/indices?v`
  - 这里请求路径中的 `_cat` 表示查看的意思， `indices` 表示索引，所以整体含义就是查看当前 `ES` 服务器中的所有索引，就好像 `MySQL` 中的 `show tables` 的感觉

- 查看单个索引

  - `http://127.0.0.1:9200/shopping`

  - ```json
    {
        "shopping": {//索引名
            "aliases": {},//别名
            "mappings": {},//映射
            "settings": {//设置
                "index": {//设置 - 索引
                    "creation_date": "1617861426847",//设置 - 索引 - 创建时间
                    "number_of_shards": "1",//设置 - 索引 - 主分片数量
                    "number_of_replicas": "1",//设置 - 索引 - 主分片数量
                    "uuid": "J0WlEhh4R7aDrfIc3AkwWQ",//设置 - 索引 - 主分片数量
                    "version": {//设置 - 索引 - 主分片数量
                        "created": "7080099"
                    },
                    "provided_name": "shopping"//设置 - 索引 - 主分片数量
                }
            }
        }
    }
    ```

- 删除索引

  - 向 ES 服务器发 DELETE 请求 ： `http://127.0.0.1:9200/shopping`

### 2.3 文档创建

假设索引已经创建好了，接下来我们来创建文档，并添加数据。这里的文档可以类比为关系型数据库中的表数据，添加的数据格式为 JSON 格式

在 Postman 中，向 ES 服务器发 POST 请求 ：`http://127.0.0.1:9200/shopping/_doc`，请求体JSON内容为：

```json
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/20d54cba223bd9d70ea356d3e40a8161.png)

> 注意，此处发送请求的方式必须为 `POST`，不能是 `PUT`，否则会发生错误 。

上面的数据创建后，由于没有指定数据唯一性标识（ID），默认情况下， ES 服务器会随机生成一个。

如果想要自定义唯一性标识，需要在创建时指定： `http://127.0.0.1:9200/shopping/_doc/1`，请求体JSON内容为

```json
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}
```

返回结果如下

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",//<------------------自定义唯一性标识
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```

### 2.4 主键查询 & 全查询

查看文档时，需要指明文档的唯一性标识，类似于 MySQL 中数据的主键查询

在 Postman 中，向 ES 服务器发 GET 请求 ： `http://127.0.0.1:9200/shopping/_doc/1` 。

返回结果如下：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "_seq_no": 1,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title": "小米手机",
        "category": "小米",
        "images": "http://www.gulixueyuan.com/xm.jpg",
        "price": 3999
    }
}
```

查找不存在的内容，向 ES 服务器发 GET 请求 ： `http://127.0.0.1:9200/shopping/_doc/1001`。

返回结果如下：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1001",
    "found": false
}
```

查看索引下所有数据，向 ES 服务器发 GET 请求 ：` http://127.0.0.1:9200/shopping/_search`。

返回结果如下

```json
{
    "took": 133,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "ANQqsHgBaKNfVnMbhZYU",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "1",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                }
            }
        ]
    }
}
```

### 2.5 全量修改 & 局部修改 & 删除

#### 2.5.1 全量修改

和新增文档一样，输入相同的 URL 地址请求，如果请求体变化，会将原有的数据内容覆盖

在 Postman 中，向 ES 服务器发 POST 请求 ： `http://127.0.0.1:9200/shopping/_doc/1`

请求体JSON内容为:

```json
{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":1999.00
}
```

修改成功后，服务器响应结果：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 2,
    "result": "updated",//<-----------updated 表示数据被更新
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```

#### 2.5.2 局部修改

修改数据时，也可以只修改某一给条数据的局部信息

在 Postman 中，向 ES 服务器发 POST 请求 ： `http://127.0.0.1:9200/shopping/_update/1`。

请求体数据如下：

```json
{
	"doc": {
		"title":"小米手机",
		"category":"小米"
	}
}
```

返回结果如下：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 3,
    "result": "updated",//<-----------updated 表示数据被更新
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```

在 Postman 中，向 ES 服务器发 GET 请求 ： `http://127.0.0.1:9200/shopping/_doc/1`，查看修改内容：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 3,
    "_seq_no": 3,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title": "小米手机",
        "category": "小米",
        "images": "http://www.gulixueyuan.com/hw.jpg",
        "price": 1999
    }
}
```



#### 2.5.3 删除

删除一个文档不会立即从磁盘上移除，它只是被标记成已删除（逻辑删除）。

在 Postman 中，向 ES 服务器发 DELETE 请求 ： `http://127.0.0.1:9200/shopping/_doc/1`

返回结果：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version": 4,
    "result": "deleted",//<---删除成功
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 4,
    "_primary_term": 1
}
```

在 Postman 中，向 ES 服务器发 GET 请求 ： `http://127.0.0.1:9200/shopping/_doc/1`，查看是否删除成功：

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "found": false
}
```

### 2.6 条件查询 & 分页查询 & 查询排序

#### 2.6.1 条件查询

假设有以下文档内容，（在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`）：

```json
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
        "max_score": 1,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "ANQqsHgBaKNfVnMbhZYU",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "A9R5sHgBaKNfVnMb25Ya",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BNR5sHgBaKNfVnMb7pal",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BtR6sHgBaKNfVnMbX5Y5",
                "_score": 1,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "B9R6sHgBaKNfVnMbZpZ6",
                "_score": 1,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "CdR7sHgBaKNfVnMbsJb9",
                "_score": 1,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            }
        ]
    }
}
```

1. **`URL` 带参查询**

   - **查找category为小米的文档**，在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search?q=category:小米`，返回结果如下：

   - ```json
     {
         "took": 94,
         "timed_out": false,
         "_shards": {
             "total": 1,
             "successful": 1,
             "skipped": 0,
             "failed": 0
         },
         "hits": {
             "total": {
                 "value": 3,
                 "relation": "eq"
             },
             "max_score": 1.3862942,
             "hits": [
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "ANQqsHgBaKNfVnMbhZYU",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 3999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "A9R5sHgBaKNfVnMb25Ya",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BNR5sHgBaKNfVnMb7pal",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 }
             ]
         }
     }
     ```

   - 上述为 `URL` 带参数形式查询，这很容易让不善者心怀恶意，或者参数值出现中文会出现乱码情况。为了避免这些情况，我们可用使用带 `JSON` 请求体请求进行查询。

2. **请求体带参查询**

   - 接下带JSON请求体，还是**查找category为小米的文档**，在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下

   - ```json
     {
     	"query":{
     		"match":{
     			"category":"小米"
     		}
     	}
     }
     ```

   - 返回结果如下

   - ```json
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
                 "value": 3,
                 "relation": "eq"
             },
             "max_score": 1.3862942,
             "hits": [
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "ANQqsHgBaKNfVnMbhZYU",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 3999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "A9R5sHgBaKNfVnMb25Ya",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BNR5sHgBaKNfVnMb7pal",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 }
             ]
         }
     }
     ```

3. **带请求体方式的查找所有内容**

   - **查找所有文档内容**，也可以这样，在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

   - ```json
     {
     	"query":{
     		"match_all":{}
     	}
     }
     ```

   - 返回结果如下：

   - ```json
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
                 "value": 6,
                 "relation": "eq"
             },
             "max_score": 1,
             "hits": [
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "ANQqsHgBaKNfVnMbhZYU",
                     "_score": 1,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 3999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "A9R5sHgBaKNfVnMb25Ya",
                     "_score": 1,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BNR5sHgBaKNfVnMb7pal",
                     "_score": 1,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BtR6sHgBaKNfVnMbX5Y5",
                     "_score": 1,
                     "_source": {
                         "title": "华为手机",
                         "category": "华为",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "B9R6sHgBaKNfVnMbZpZ6",
                     "_score": 1,
                     "_source": {
                         "title": "华为手机",
                         "category": "华为",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "CdR7sHgBaKNfVnMbsJb9",
                     "_score": 1,
                     "_source": {
                         "title": "华为手机",
                         "category": "华为",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 }
             ]
         }
     }
     ```

4. 查询指定字段

   - **如果你想查询指定字段**，在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

   - ```json
     {
     	"query":{
     		"match_all":{}
     	},
     	"_source":["title"]
     }
     ```

   - 返回结果如下：

   - ```json
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
             "max_score": 1,
             "hits": [
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "ANQqsHgBaKNfVnMbhZYU",
                     "_score": 1,
                     "_source": {
                         "title": "小米手机"
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "A9R5sHgBaKNfVnMb25Ya",
                     "_score": 1,
                     "_source": {
                         "title": "小米手机"
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BNR5sHgBaKNfVnMb7pal",
                     "_score": 1,
                     "_source": {
                         "title": "小米手机"
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BtR6sHgBaKNfVnMbX5Y5",
                     "_score": 1,
                     "_source": {
                         "title": "华为手机"
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "B9R6sHgBaKNfVnMbZpZ6",
                     "_score": 1,
                     "_source": {
                         "title": "华为手机"
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "CdR7sHgBaKNfVnMbsJb9",
                     "_score": 1,
                     "_source": {
                         "title": "华为手机"
                     }
                 }
             ]
         }
     }
     ```

#### 2.6.2 分页查询

在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"query":{
		"match_all":{}
	},
	"from":0,
	"size":2
}
```

返回结果如下：

```json
{
    "took": 1,
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
        "max_score": 1,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "ANQqsHgBaKNfVnMbhZYU",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "A9R5sHgBaKNfVnMb25Ya",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            }
        ]
    }
}
```

#### 2.6.3 查询排序

如果你想通过排序查出价格最高的手机，在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"query":{
		"match_all":{}
	},
	"sort":{
		"price":{
			"order":"desc"
		}
	}
}
```

返回结果如下：

```json
{
    "took": 96,
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
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "ANQqsHgBaKNfVnMbhZYU",
                "_score": null,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                },
                "sort": [
                    3999
                ]
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "A9R5sHgBaKNfVnMb25Ya",
                "_score": null,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "sort": [
                    1999
                ]
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BNR5sHgBaKNfVnMb7pal",
                "_score": null,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "sort": [
                    1999
                ]
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BtR6sHgBaKNfVnMbX5Y5",
                "_score": null,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "sort": [
                    1999
                ]
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "B9R6sHgBaKNfVnMbZpZ6",
                "_score": null,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "sort": [
                    1999
                ]
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "CdR7sHgBaKNfVnMbsJb9",
                "_score": null,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "sort": [
                    1999
                ]
            }
        ]
    }
}
```

### 2.7 多条件查询 & 范围查询

#### 2.7.1 多条件查询

1. 与（`must` 相当于数据库的 `&&` ）

   - 假设想找出小米牌子，价格为3999元的。

   - 在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

   - ````json
     {
     	"query":{
     		"bool":{
     			"must":[{
     				"match":{
     					"category":"小米"
     				}
     			},{
     				"match":{
     					"price":3999.00
     				}
     			}]
     		}
     	}
     }
     ````

   - 返回结果如下：

   - ```json
     {
         "took": 134,
         "timed_out": false,
         "_shards": {
             "total": 1,
             "successful": 1,
             "skipped": 0,
             "failed": 0
         },
         "hits": {
             "total": {
                 "value": 1,
                 "relation": "eq"
             },
             "max_score": 2.3862944,
             "hits": [
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "ANQqsHgBaKNfVnMbhZYU",
                     "_score": 2.3862944,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 3999
                     }
                 }
             ]
         }
     }
     ```

2. 或（ `should` 相当于数据库的 `||` ）

   - 在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

   - ```json
     {
     	"query":{
     		"bool":{
     			"should":[{
     				"match":{
     					"category":"小米"
     				}
     			},{
     				"match":{
     					"category":"华为"
     				}
     			}]
     		},
             "filter":{
                 "range":{
                     "price":{
                         "gt":2000
                     }
                 }
             }
     	}
     }
     ```

   - 返回结果如下：

   - ```json
     {
         "took": 8,
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
             "max_score": 1.3862942,
             "hits": [
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "ANQqsHgBaKNfVnMbhZYU",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 3999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "A9R5sHgBaKNfVnMb25Ya",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BNR5sHgBaKNfVnMb7pal",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "小米手机",
                         "category": "小米",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "BtR6sHgBaKNfVnMbX5Y5",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "华为手机",
                         "category": "华为",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "B9R6sHgBaKNfVnMbZpZ6",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "华为手机",
                         "category": "华为",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 },
                 {
                     "_index": "shopping",
                     "_type": "_doc",
                     "_id": "CdR7sHgBaKNfVnMbsJb9",
                     "_score": 1.3862942,
                     "_source": {
                         "title": "华为手机",
                         "category": "华为",
                         "images": "http://www.gulixueyuan.com/xm.jpg",
                         "price": 1999
                     }
                 }
             ]
         }
     }
     ```

#### 2.7.2 范围查询

假设想找出小米和华为的牌子，价格大于2000元的手机。

在 Postman 中，向 ES 服务器发 GET请求 ：`http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"query":{
		"bool":{
			"should":[{
				"match":{
					"category":"小米"
				}
			},{
				"match":{
					"category":"华为"
				}
			}],
            "filter":{
            	"range":{
                	"price":{
                    	"gt":2000
                	}
	            }
    	    }
		}
	}
}
```

输出结果如下：

```json
{
    "took": 72,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.3862942,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "ANQqsHgBaKNfVnMbhZYU",
                "_score": 1.3862942,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                }
            }
        ]
    }
}
```

### 2.8 全文检索 & 完全匹配 & 高亮查询

#### 2.8.1 全文检索

功能像搜索引擎那样，如品牌输入“小华”，返回结果带回品牌有“小米”和华为的。

在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"query":{
		"match":{
			"category" : "小华"
		}
	}
}
```

返回结果如下：

```json
{
    "took": 7,
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
        "max_score": 0.6931471,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "ANQqsHgBaKNfVnMbhZYU",
                "_score": 0.6931471,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "A9R5sHgBaKNfVnMb25Ya",
                "_score": 0.6931471,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BNR5sHgBaKNfVnMb7pal",
                "_score": 0.6931471,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BtR6sHgBaKNfVnMbX5Y5",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "B9R6sHgBaKNfVnMbZpZ6",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "CdR7sHgBaKNfVnMbsJb9",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            }
        ]
    }
}
```

#### 2.8.2 完全匹配

在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"query":{
		"match_phrase":{
			"category" : "为"
		}
	}
}
```

返回结果如下：

```json
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
            "value": 3,
            "relation": "eq"
        },
        "max_score": 0.6931471,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BtR6sHgBaKNfVnMbX5Y5",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "B9R6sHgBaKNfVnMbZpZ6",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "CdR7sHgBaKNfVnMbsJb9",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            }
        ]
    }
}
```

#### 2.8.3 高亮查询

在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"query":{
		"match_phrase":{
			"category" : "为"
		}
	},
    "highlight":{
        "fields":{
            "category":{}//<----高亮这字段
        }
    }
}
```

返回结果：

```json
{
    "took": 100,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 0.6931471,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BtR6sHgBaKNfVnMbX5Y5",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "highlight": {
                    "category": [
                        "华<em>为</em>"//<------高亮一个为字。
                    ]
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "B9R6sHgBaKNfVnMbZpZ6",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "highlight": {
                    "category": [
                        "华<em>为</em>"
                    ]
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "CdR7sHgBaKNfVnMbsJb9",
                "_score": 0.6931471,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                },
                "highlight": {
                    "category": [
                        "华<em>为</em>"
                    ]
                }
            }
        ]
    }
}
```

### 2.9 聚合查询

聚合允许使用者对 `es` 文档进行统计分析，类似与关系型数据库中的 `group by`，当然还有很多其他的聚合，例如取最大值 `max` 、平均值 `avg` 等等。

接下来按 `price` 字段进行分组：

在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"aggs":{//聚合操作
		"price_group":{//名称，随意起名
			"terms":{//分组
				"field":"price"//分组字段
			}
		}
	}
}
```

返回结果如下：

```json
{
    "took": 63,
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
        "max_score": 1,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "ANQqsHgBaKNfVnMbhZYU",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "A9R5sHgBaKNfVnMb25Ya",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BNR5sHgBaKNfVnMb7pal",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "BtR6sHgBaKNfVnMbX5Y5",
                "_score": 1,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "B9R6sHgBaKNfVnMbZpZ6",
                "_score": 1,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "CdR7sHgBaKNfVnMbsJb9",
                "_score": 1,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 1999
                }
            }
        ]
    },
    "aggregations": {
        "price_group": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 1999,
                    "doc_count": 5
                },
                {
                    "key": 3999,
                    "doc_count": 1
                }
            ]
        }
    }
}
```

上面返回结果会附带原始数据的。若不想要不附带原始数据的结果，在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"aggs":{
		"price_group":{
			"terms":{
				"field":"price"
			}
		}
	},
    "size":0
}
```

返回结果如下：

```json
{
    "took": 60,
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
        "price_group": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 1999,
                    "doc_count": 5
                },
                {
                    "key": 3999,
                    "doc_count": 1
                }
            ]
        }
    }
}
```

若想对所有手机价格求**平均值**。

在 Postman 中，向 ES 服务器发 GET请求 ： `http://127.0.0.1:9200/shopping/_search`，附带JSON体如下：

```json
{
	"aggs":{
		"price_avg":{//名称，随意起名
			"avg":{//求平均
				"field":"price"
			}
		}
	},
    "size":0
}
```

返回结果如下：

```json
{
    "took": 14,
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
        "price_avg": {
            "value": 2332.3333333333335
        }
    }
}
```

### 2.10 映射关系

有了索引库，等于有了数据库中的 `database`。

接下来就需要建索引库 `(index)` 中的映射了，类似于数据库 `(database)` 中的表结构 `(table)` 。

- 创建数据库表需要设置字段名称，类型，长度，约束等；
- 索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射 `(mapping)`。

1. 建一个索引 `PUT http://127.0.0.1:9200/user`

2. 返回结果

   - ```json
     {
         "acknowledged": true,
         "shards_acknowledged": true,
         "index": "user"
     }
     ```

3. 新建映射

   - ```json
     # PUT http://127.0.0.1:9200/user/_mapping
     
     {
         "properties": {
             "name":{
             	"type": "text",
             	"index": true
             },
             "sex":{
             	"type": "keyword", // 不能被分词
             	"index": true 
             },
             "tel":{
             	"type": "keyword",
             	"index": false // 不能被索引
             }
         }
     }
     ```

4. 返回结果

   - ```json
     {
         "acknowledged": true
     }
     ```

5. 查看映射

   - ```json
     #GET http://127.0.0.1:9200/user/_mapping
     ```

6. 返回结果

   - ```json
     {
         "user": {
             "mappings": {
                 "properties": {
                     "name": {
                         "type": "text"
                     },
                     "sex": {
                         "type": "keyword"
                     },
                     "tel": {
                         "type": "keyword",
                         "index": false
                     }
                 }
             }
         }
     }
     ```

7. 增加数据

   - ```json
     #PUT http://127.0.0.1:9200/user/_create/1001
     {
     	"name":"小米",
     	"sex":"男的",
     	"tel":"1111"
     }
     ```

8. 返回结果

   - ```json
     {
         "_index": "user",
         "_type": "_doc",
         "_id": "1001",
         "_version": 1,
         "result": "created",
         "_shards": {
             "total": 2,
             "successful": 1,
             "failed": 0
         },
         "_seq_no": 0,
         "_primary_term": 1
     }
     ```

9. 查找name含有”小“数据：

   - ```json
     #GET http://127.0.0.1:9200/user/_search
     {
     	"query":{
     		"match":{
     			"name":"小"
     		}
     	}
     }
     ```

10. 返回结果

    - ```json
      {
          "took": 495,
          "timed_out": false,
          "_shards": {
              "total": 1,
              "successful": 1,
              "skipped": 0,
              "failed": 0
          },
          "hits": {
              "total": {
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 0.2876821,
              "hits": [
                  {
                      "_index": "user",
                      "_type": "_doc",
                      "_id": "1001",
                      "_score": 0.2876821,
                      "_source": {
                          "name": "小米",
                          "sex": "男的",
                          "tel": "1111"
                      }
                  }
              ]
          }
      }
      ```

11. 查找sex含有”男“数据：

    - ```json
      #GET http://127.0.0.1:9200/user/_search
      {
      	"query":{
      		"match":{
      			"sex":"男"
      		}
      	}
      }
      ```

12. 返回结果

    - ```json
      {
          "took": 1,
          "timed_out": false,
          "_shards": {
              "total": 1,
              "successful": 1,
              "skipped": 0,
              "failed": 0
          },
          "hits": {
              "total": {
                  "value": 0,
                  "relation": "eq"
              },
              "max_score": null,
              "hits": []
          }
      }
      ```

    - > 找不想要的结果，只因创建映射时"sex"的类型为"keyword"。
      >
      > "sex"只能完全为”男的“，才能得出原数据。

    - ```json
      #GET http://127.0.0.1:9200/user/_search
      {
      	"query":{
      		"match":{
      			"sex":"男的"
      		}
      	}
      }
      ```

    - ```json
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
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 0.2876821,
              "hits": [
                  {
                      "_index": "user",
                      "_type": "_doc",
                      "_id": "1001",
                      "_score": 0.2876821,
                      "_source": {
                          "name": "小米",
                          "sex": "男的",
                          "tel": "1111"
                      }
                  }
              ]
          }
      }
      ```

### 2.11 JavaAPI

#### 2.11.1 环境准备

添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch 的客户端 -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch 依赖 2.x 的 log4j -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.9</version>
    </dependency>
    <!-- junit 单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

测试连接

```java
public class HelloElasticSearch {
    public static void main(String[] args) throws IOException {

        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("127.0.0.1", 9200, "http")
                )
        );
        System.out.println(client);
        client.close();
    }
}
```

#### 2.11.2 创建索引

```java
public class CreateIndex {

    public static void main(String[] args) throws IOException {
        // 创建客户端对象
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        // 创建索引 - 请求对象
        CreateIndexRequest request = new CreateIndexRequest("user2");
        // 发送请求，获取响应
        CreateIndexResponse response = client.indices().create(request,
                RequestOptions.DEFAULT);
        boolean acknowledged = response.isAcknowledged();
        // 响应状态
        System.out.println("操作状态 = " + acknowledged);

        // 关闭客户端连接
        client.close();
    }
}
```

```java
四月 09, 2021 2:12:08 下午 org.elasticsearch.client.RestClient logResponse
警告: request [PUT http://localhost:9200/user2?master_timeout=30s&include_type_name=true&timeout=30s] returned 1 warnings: [299 Elasticsearch-7.8.0-757314695644ea9a1dc2fecd26d1a43856725e65 "[types removal] Using include_type_name in create index requests is deprecated. The parameter will be removed in the next major version."]
操作状态 = true

Process finished with exit code 0
```

#### 2.11.3 查询索引

```java
public class SearchIndex {
    public static void main(String[] args) throws IOException {
        // 创建客户端对象
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        // 查询索引 - 请求对象
        GetIndexRequest request = new GetIndexRequest("user2");
        // 发送请求，获取响应
        GetIndexResponse response = client.indices().get(request,
                RequestOptions.DEFAULT);
        // 响应状态
        System.out.println("aliases:"+response.getAliases());
        System.out.println("mappings:"+response.getMappings());
        System.out.println("settings:"+response.getSettings());

        client.close();
    }
}
```

```java
aliases:{user2=[]}
mappings:{user2=org.elasticsearch.cluster.metadata.MappingMetadata@ad700514}
settings:{user2={"index.creation_date":"1617948726976","index.number_of_replicas":"1","index.number_of_shards":"1","index.provided_name":"user2","index.uuid":"UGZ1ntcySnK6hWyP2qoVpQ","index.version.created":"7080099"}}

Process finished with exit code 0
```

#### 2.11.4 删除索引

```java
public class DeleteIndex {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));
        // 删除索引 - 请求对象
        DeleteIndexRequest request = new DeleteIndexRequest("user2");
        // 发送请求，获取响应
        AcknowledgedResponse response = client.indices().delete(request,RequestOptions.DEFAULT);
        // 操作结果
        System.out.println("操作结果 ： " + response.isAcknowledged());
        client.close();
    }
}
```

```java
操作结果 ： true

Process finished with exit code 0
```

#### 2.11.5 文档-新增 & 修改



- 新增

  - ```java
    public class InsertDoc {
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                // 新增文档 - 请求对象
                IndexRequest request = new IndexRequest();
                // 设置索引及唯一性标识
                request.index("user").id("1001");
    
                // 创建数据对象
                User user = new User();
                user.setName("zhangsan");
                user.setAge(30);
                user.setSex("男");
    
                ObjectMapper objectMapper = new ObjectMapper();
                String productJson = objectMapper.writeValueAsString(user);
                // 添加文档数据，数据格式为 JSON 格式
                request.source(productJson, XContentType.JSON);
                // 客户端发送请求，获取响应对象
                IndexResponse response = client.index(request, RequestOptions.DEFAULT);
                3.打印结果信息
                System.out.println("_index:" + response.getIndex());
                System.out.println("_id:" + response.getId());
                System.out.println("_result:" + response.getResult());
            });
        }
    }
    ```

  - ```java
    _index:user
    _id:1001
    _result:UPDATED
    
    Process finished with exit code 0
    ```

- 修改

  - ```java
    public class UpdateDoc {
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                // 修改文档 - 请求对象
                UpdateRequest request = new UpdateRequest();
                // 配置修改参数
                request.index("user").id("1001");
                // 设置请求体，对数据进行修改
                request.doc(XContentType.JSON, "sex", "女");
                // 客户端发送请求，获取响应对象
                UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
                System.out.println("_index:" + response.getIndex());
                System.out.println("_id:" + response.getId());
                System.out.println("_result:" + response.getResult());
            });
        }
    }
    ```

  - ```java
    _index:user
    _id:1001
    _result:UPDATED
    
    Process finished with exit code 0
    ```

#### 2.11.6 文档-查询 & 删除

- 插叙

  - ```java
    public class GetDoc {
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                // 1.创建请求对象
                GetRequest request = new GetRequest().index("user").id("1001");
                // 2.客户端发送请求，获取响应对象
                GetResponse response = client.get(request, RequestOptions.DEFAULT);
                // 3.打印结果信息
                System.out.println("_index:" + response.getIndex());
                System.out.println("_type:" + response.getType());
                System.out.println("_id:" + response.getId());
                System.out.println("source:" + response.getSourceAsString());
            });
        }
    }
    ```

  - ```java
    _index:user
    _type:_doc
    _id:1001
    source:{"name":"zhangsan","age":30,"sex":"男"}
    
    Process finished with exit code 0
    ```

- 删除

  - ```java
    public class DeleteDoc {
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                //创建请求对象
                DeleteRequest request = new DeleteRequest().index("user").id("1001");
                //客户端发送请求，获取响应对象
                DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
                //打印信息
                System.out.println(response.toString());
            });
        }
    }
    ```

  - ```java
    DeleteResponse[index=user,type=_doc,id=1001,version=16,result=deleted,shards=ShardInfo{total=2, successful=1, failures=[]}]
    
    Process finished with exit code 0
    ```

#### 2.11.7 文档-批量新增 & 批量删除

- 批量新增

  - ```java
    public class BatchInsertDoc {
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                //创建批量新增请求对象
                BulkRequest request = new BulkRequest();
                request.add(new
                        IndexRequest().index("user").id("1001").source(XContentType.JSON, "name",
                        "zhangsan"));
                request.add(new
                        IndexRequest().index("user").id("1002").source(XContentType.JSON, "name",
                                "lisi"));
                request.add(new
                        IndexRequest().index("user").id("1003").source(XContentType.JSON, "name",
                        "wangwu"));
                //客户端发送请求，获取响应对象
                BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
                //打印结果信息
                System.out.println("took:" + responses.getTook());
                System.out.println("items:" + responses.getItems());
            });
        }
    }
    ```

  - ```java
    took:294ms
    items:[Lorg.elasticsearch.action.bulk.BulkItemResponse;@2beee7ff
    
    Process finished with exit code 0
    ```

- 批量删除

  - ```java
    public class BatchDeleteDoc {
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                //创建批量删除请求对象
                BulkRequest request = new BulkRequest();
                request.add(new DeleteRequest().index("user").id("1001"));
                request.add(new DeleteRequest().index("user").id("1002"));
                request.add(new DeleteRequest().index("user").id("1003"));
                //客户端发送请求，获取响应对象
                BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
                //打印结果信息
                System.out.println("took:" + responses.getTook());
                System.out.println("items:" + responses.getItems());
            });
        }
    }
    ```

  - ```java
    took:108ms
    items:[Lorg.elasticsearch.action.bulk.BulkItemResponse;@7b02881e
    
    Process finished with exit code 0
    ```

#### 2.11.8 文档-高级查询-全量查询

- 批量插入数据

  - ```java
    public class BatchInsertDoc {
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                //创建批量新增请求对象
                BulkRequest request = new BulkRequest();
                request.add(new IndexRequest().index("user").id("1001").source(XContentType.JSON, "name", "zhangsan", "age", "10", "sex","女"));
                request.add(new IndexRequest().index("user").id("1002").source(XContentType.JSON, "name", "lisi", "age", "30", "sex","女"));
                request.add(new IndexRequest().index("user").id("1003").source(XContentType.JSON, "name", "wangwu1", "age", "40", "sex","男"));
                request.add(new IndexRequest().index("user").id("1004").source(XContentType.JSON, "name", "wangwu2", "age", "20", "sex","女"));
                request.add(new IndexRequest().index("user").id("1005").source(XContentType.JSON, "name", "wangwu3", "age", "50", "sex","男"));
                request.add(new IndexRequest().index("user").id("1006").source(XContentType.JSON, "name", "wangwu4", "age", "20", "sex","男"));
                //客户端发送请求，获取响应对象
                BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
                //打印结果信息
                System.out.println("took:" + responses.getTook());
                System.out.println("items:" + responses.getItems());
            });
        }
    }
    ```

  - ```java
    took:168ms
    items:[Lorg.elasticsearch.action.bulk.BulkItemResponse;@2beee7ff
    
    Process finished with exit code 0
    ```

- **查询所有索引数据**

  - ```java
    public class QueryDoc {
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(client -> {
                // 创建搜索请求对象
                SearchRequest request = new SearchRequest();
                request.indices("user");
                // 构建查询的请求体
                SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
                // 查询所有数据
                sourceBuilder.query(QueryBuilders.matchAllQuery());
                request.source(sourceBuilder);
                SearchResponse response = client.search(request, RequestOptions.DEFAULT);
                // 查询匹配
                SearchHits hits = response.getHits();
                System.out.println("took:" + response.getTook());
                System.out.println("timeout:" + response.isTimedOut());
                System.out.println("total:" + hits.getTotalHits());
                System.out.println("MaxScore:" + hits.getMaxScore());
                System.out.println("hits========>>");
                for (SearchHit hit : hits) {
                //输出每条查询的结果信息
                    System.out.println(hit.getSourceAsString());
                }
                System.out.println("<<========");
            });
        }
    }
    ```

  - ```java
    took:2ms
    timeout:false
    total:6 hits
    MaxScore:1.0
    hits========>>
    {"name":"zhangsan","age":"10","sex":"女"}
    {"name":"lisi","age":"30","sex":"女"}
    {"name":"wangwu1","age":"40","sex":"男"}
    {"name":"wangwu2","age":"20","sex":"女"}
    {"name":"wangwu3","age":"50","sex":"男"}
    {"name":"wangwu4","age":"20","sex":"男"}
    <<========
    
    Process finished with exit code 0
    ```

#### 2.11.9 文档-高级查询-分页查询 & 条件查询 & 查询排序

- 条件查询

  - ```java
    public class QueryDoc {
        
    	public static final ElasticsearchTask SEARCH_BY_CONDITION = client -> {
            // 创建搜索请求对象
            SearchRequest request = new SearchRequest();
            request.indices("user");
            // 构建查询的请求体
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            sourceBuilder.query(QueryBuilders.termQuery("age", "30"));
            request.source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            // 查询匹配
            SearchHits hits = response.getHits();
            System.out.println("took:" + response.getTook());
            System.out.println("timeout:" + response.isTimedOut());
            System.out.println("total:" + hits.getTotalHits());
            System.out.println("MaxScore:" + hits.getMaxScore());
            System.out.println("hits========>>");
            for (SearchHit hit : hits) {
                //输出每条查询的结果信息
                System.out.println(hit.getSourceAsString());
            }
            System.out.println("<<========");
        };
        
        public static void main(String[] args) {
            ConnectElasticsearch.connect(SEARCH_BY_CONDITION);
        }
    }
    ```

  - ```java
    took:1ms
    timeout:false
    total:1 hits
    MaxScore:1.0
    hits========>>
    {"name":"lisi","age":"30","sex":"女"}
    <<========
    ```

- 分页查询

  - ```java
    public class QueryDoc {
        
    	public static final ElasticsearchTask SEARCH_BY_PAGING = client -> {
            // 创建搜索请求对象
            SearchRequest request = new SearchRequest();
            request.indices("user");
            // 构建查询的请求体
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            sourceBuilder.query(QueryBuilders.matchAllQuery());
            // 分页查询
            // 当前页其实索引(第一条数据的顺序号)， from
            sourceBuilder.from(0);
    
            // 每页显示多少条 size
            sourceBuilder.size(2);
            request.source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            // 查询匹配
            SearchHits hits = response.getHits();
            System.out.println("took:" + response.getTook());
            System.out.println("timeout:" + response.isTimedOut());
            System.out.println("total:" + hits.getTotalHits());
            System.out.println("MaxScore:" + hits.getMaxScore());
            System.out.println("hits========>>");
            for (SearchHit hit : hits) {
                //输出每条查询的结果信息
                System.out.println(hit.getSourceAsString());
            }
            System.out.println("<<========");
        };
        
        public static void main(String[] args) {
            ConnectElasticsearch.connect(SEARCH_BY_CONDITION);
        }
    }
    ```

  - ```java
    took:1ms
    timeout:false
    total:6 hits
    MaxScore:1.0
    hits========>>
    {"name":"zhangsan","age":"10","sex":"女"}
    {"name":"lisi","age":"30","sex":"女"}
    <<========
    ```

- 查询排序

  - ```java
    public class QueryDoc {
        
    	public static final ElasticsearchTask SEARCH_WITH_ORDER = client -> {
            // 创建搜索请求对象
            SearchRequest request = new SearchRequest();
            request.indices("user");
    
            // 构建查询的请求体
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            sourceBuilder.query(QueryBuilders.matchAllQuery());
            // 排序
            sourceBuilder.sort("age", SortOrder.ASC);
            request.source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            // 查询匹配
            SearchHits hits = response.getHits();
            System.out.println("took:" + response.getTook());
            System.out.println("timeout:" + response.isTimedOut());
            System.out.println("total:" + hits.getTotalHits());
            System.out.println("MaxScore:" + hits.getMaxScore());
            System.out.println("hits========>>");
            for (SearchHit hit : hits) {
            //输出每条查询的结果信息
                System.out.println(hit.getSourceAsString());
            }
            System.out.println("<<========");
        };
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(SEARCH_WITH_ORDER);
        }
    }
    ```

  - ```java
    took:1ms
    timeout:false
    total:6 hits
    MaxScore:NaN
    hits========>>
    {"name":"zhangsan","age":"10","sex":"女"}
    {"name":"wangwu2","age":"20","sex":"女"}
    {"name":"wangwu4","age":"20","sex":"男"}
    {"name":"lisi","age":"30","sex":"女"}
    {"name":"wangwu1","age":"40","sex":"男"}
    {"name":"wangwu3","age":"50","sex":"男"}
    <<========
    ```

#### 2.11.10 高级查询-组合查询 & 范围查询

- 组合查询

  - ```java
    public class QueryDoc {
        
    	public static final ElasticsearchTask SEARCH_BY_BOOL_CONDITION = client -> {
            // 创建搜索请求对象
            SearchRequest request = new SearchRequest();
            request.indices("user");
            // 构建查询的请求体
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
            // 必须包含
            boolQueryBuilder.must(QueryBuilders.matchQuery("age", "30"));
            // 一定不含
            boolQueryBuilder.mustNot(QueryBuilders.matchQuery("name", "zhangsan"));
            // 可能包含
            boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "男"));
            sourceBuilder.query(boolQueryBuilder);
            request.source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            // 查询匹配
            SearchHits hits = response.getHits();
            System.out.println("took:" + response.getTook());
            System.out.println("timeout:" + response.isTimedOut());
            System.out.println("total:" + hits.getTotalHits());
            System.out.println("MaxScore:" + hits.getMaxScore());
            System.out.println("hits========>>");
            for (SearchHit hit : hits) {
                //输出每条查询的结果信息
                System.out.println(hit.getSourceAsString());
            }
            System.out.println("<<========");
    
        };
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(SEARCH_BY_BOOL_CONDITION);
        }
    }
    ```

  - ```java
    took:28ms
    timeout:false
    total:1 hits
    MaxScore:1.0
    hits========>>
    {"name":"lisi","age":"30","sex":"女"}
    <<========
    
    Process finished with exit code 0
    ```

- 范围查询

  - ```java
    public class QueryDoc {
        
    	public static final ElasticsearchTask SEARCH_BY_RANGE = client -> {
            // 创建搜索请求对象
            SearchRequest request = new SearchRequest();
            request.indices("user");
            // 构建查询的请求体
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("age");
            // 大于等于
            //rangeQuery.gte("30");
            // 小于等于
            rangeQuery.lte("40");
            sourceBuilder.query(rangeQuery);
            request.source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            // 查询匹配
            SearchHits hits = response.getHits();
            System.out.println("took:" + response.getTook());
            System.out.println("timeout:" + response.isTimedOut());
            System.out.println("total:" + hits.getTotalHits());
            System.out.println("MaxScore:" + hits.getMaxScore());
            System.out.println("hits========>>");
            for (SearchHit hit : hits) {
            //输出每条查询的结果信息
                System.out.println(hit.getSourceAsString());
            }
            System.out.println("<<========");
        };
    
        public static void main(String[] args) {
            ConnectElasticsearch.connect(SEARCH_BY_RANGE);
        }
    }
    ```

  - ```java
    took:1ms
    timeout:false
    total:5 hits
    MaxScore:1.0
    hits========>>
    {"name":"zhangsan","age":"10","sex":"女"}
    {"name":"lisi","age":"30","sex":"女"}
    {"name":"wangwu1","age":"40","sex":"男"}
    {"name":"wangwu2","age":"20","sex":"女"}
    {"name":"wangwu4","age":"20","sex":"男"}
    <<========
    
    Process finished with exit code 0
    ```

#### 2.11.11 文档-高级查询-模糊查询 & 高亮查询

- 模糊查询

  - ```java
    public class QueryDoc {
        
        public static final ElasticsearchTask SEARCH_BY_FUZZY_CONDITION = client -> {
            // 创建搜索请求对象
            SearchRequest request = new SearchRequest();
            request.indices("user");
            // 构建查询的请求体
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            sourceBuilder.query(QueryBuilders.fuzzyQuery("name","wangwu").fuzziness(Fuzziness.ONE));
            request.source(sourceBuilder);
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            // 查询匹配
            SearchHits hits = response.getHits();
            System.out.println("took:" + response.getTook());
            System.out.println("timeout:" + response.isTimedOut());
            System.out.println("total:" + hits.getTotalHits());
            System.out.println("MaxScore:" + hits.getMaxScore());
            System.out.println("hits========>>");
            for (SearchHit hit : hits) {
                //输出每条查询的结果信息
                System.out.println(hit.getSourceAsString());
            }
            System.out.println("<<========");
        };
    ```

  - ```java
    took:152ms
    timeout:false
    total:4 hits
    MaxScore:1.2837042
    hits========>>
    {"name":"wangwu1","age":"40","sex":"男"}
    {"name":"wangwu2","age":"20","sex":"女"}
    {"name":"wangwu3","age":"50","sex":"男"}
    {"name":"wangwu4","age":"20","sex":"男"}
    <<========
    
    Process finished with exit code 0
    ```

- 高亮查询

  - ```java
    public class QueryDoc {
        
        public static final ElasticsearchTask SEARCH_WITH_HIGHLIGHT = client -> {
            // 高亮查询
            SearchRequest request = new SearchRequest().indices("user");
            //2.创建查询请求体构建器
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            //构建查询方式：高亮查询
            TermsQueryBuilder termsQueryBuilder =
                    QueryBuilders.termsQuery("name","zhangsan");
            //设置查询方式
            sourceBuilder.query(termsQueryBuilder);
            //构建高亮字段
            HighlightBuilder highlightBuilder = new HighlightBuilder();
            highlightBuilder.preTags("<font color='red'>");//设置标签前缀
            highlightBuilder.postTags("</font>");//设置标签后缀
            highlightBuilder.field("name");//设置高亮字段
            //设置高亮构建对象
            sourceBuilder.highlighter(highlightBuilder);
            //设置请求体
            request.source(sourceBuilder);
            //3.客户端发送请求，获取响应对象
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            //4.打印响应结果
            SearchHits hits = response.getHits();
            System.out.println("took::"+response.getTook());
            System.out.println("time_out::"+response.isTimedOut());
            System.out.println("total::"+hits.getTotalHits());
            System.out.println("max_score::"+hits.getMaxScore());
            System.out.println("hits::::>>");
            for (SearchHit hit : hits) {
                String sourceAsString = hit.getSourceAsString();
                System.out.println(sourceAsString);
                //打印高亮结果
                Map<String, HighlightField> highlightFields = hit.getHighlightFields();
                System.out.println(highlightFields);
            }
            System.out.println("<<::::");
        };
    }
    ```

  - ```java
    took::672ms
    time_out::false
    total::1 hits
    max_score::1.0
    hits::::>>
    {"name":"zhangsan","age":"10","sex":"女"}
    {name=[name], fragments[[<font color='red'>zhangsan</font>]]}
    <<::::
    
    Process finished with exit code 0
    ```

#### 2.11.12 文档-高级查询-最大值查询 & 分组查询

- 最大值查询

  - ```java
    public class QueryDoc {
        
        public static final ElasticsearchTask SEARCH_WITH_MAX = client -> {
            // 高亮查询
            SearchRequest request = new SearchRequest().indices("user");
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            sourceBuilder.aggregation(AggregationBuilders.max("maxAge").field("age"));
            //设置请求体
            request.source(sourceBuilder);
            //3.客户端发送请求，获取响应对象
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            //4.打印响应结果
            SearchHits hits = response.getHits();
            System.out.println(response);
        };
    }
    ```

  - ```java
    {"took":16,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":6,"relation":"eq"},"max_score":1.0,"hits":[{"_index":"user","_type":"_doc","_id":"1001","_score":1.0,"_source":{"name":"zhangsan","age":"10","sex":"女"}},{"_index":"user","_type":"_doc","_id":"1002","_score":1.0,"_source":{"name":"lisi","age":"30","sex":"女"}},{"_index":"user","_type":"_doc","_id":"1003","_score":1.0,"_source":{"name":"wangwu1","age":"40","sex":"男"}},{"_index":"user","_type":"_doc","_id":"1004","_score":1.0,"_source":{"name":"wangwu2","age":"20","sex":"女"}},{"_index":"user","_type":"_doc","_id":"1005","_score":1.0,"_source":{"name":"wangwu3","age":"50","sex":"男"}},{"_index":"user","_type":"_doc","_id":"1006","_score":1.0,"_source":{"name":"wangwu4","age":"20","sex":"男"}}]},"aggregations":{"max#maxAge":{"value":50.0}}}
    
    Process finished with exit code 0
    ```

- 分组查询

  - ```java
    public class QueryDoc {
    
    	public static final ElasticsearchTask SEARCH_WITH_GROUP = client -> {
            SearchRequest request = new SearchRequest().indices("user");
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            sourceBuilder.aggregation(AggregationBuilders.terms("age_groupby").field("age"));
            //设置请求体
            request.source(sourceBuilder);
            //3.客户端发送请求，获取响应对象
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            //4.打印响应结果
            SearchHits hits = response.getHits();
            System.out.println(response);
        };
    }
    ```

  - ```java
    {"took":10,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":6,"relation":"eq"},"max_score":1.0,"hits":[{"_index":"user","_type":"_doc","_id":"1001","_score":1.0,"_source":{"name":"zhangsan","age":"10","sex":"女"}},{"_index":"user","_type":"_doc","_id":"1002","_score":1.0,"_source":{"name":"lisi","age":"30","sex":"女"}},{"_index":"user","_type":"_doc","_id":"1003","_score":1.0,"_source":{"name":"wangwu1","age":"40","sex":"男"}},{"_index":"user","_type":"_doc","_id":"1004","_score":1.0,"_source":{"name":"wangwu2","age":"20","sex":"女"}},{"_index":"user","_type":"_doc","_id":"1005","_score":1.0,"_source":{"name":"wangwu3","age":"50","sex":"男"}},{"_index":"user","_type":"_doc","_id":"1006","_score":1.0,"_source":{"name":"wangwu4","age":"20","sex":"男"}}]},"aggregations":{"lterms#age_groupby":{"doc_count_error_upper_bound":0,"sum_other_doc_count":0,"buckets":[{"key":20,"doc_count":2},{"key":10,"doc_count":1},{"key":30,"doc_count":1},{"key":40,"doc_count":1},{"key":50,"doc_count":1}]}}}
    
    Process finished with exit code 0
    ```

## 3. ElasticSearch环境

### 3.1 环境简介

#### 3.1.1 单机 & 集群

单台 `Elasticsearch` 服务器提供服务，往往都有最大的负载能力，超过这个阈值，服务器性能就会大大降低甚至不可用，所以生产环境中，一般都是运行在指定服务器集群中。除了负载能力，单点服务器也存在其他问题：

- 单台机器存储容量有限
- 单服务器容易出现单点故障，无法实现高可用
- 单服务的并发处理能力有限

配置服务器集群时，集群中节点数量没有限制，大于等于 2 个节点就可以看做是集群了。一般出于高性能及高可用方面来考虑集群中节点数量都是 3 个以上

总之，集群能提高性能，增加容错。

#### 3.1.2 集群 Cluster

**一个集群就是由一个或多个服务器节点组织在一起，共同持有整个的数据，并一起提供索引和搜索功能。**一个 `Elasticsearch` 集群有一个唯一的名字标识，这个名字默认就是 `“elasticsearch”` 。名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

#### 3.1.3 节点 Node

集群中包含很多服务器， 一个节点就是其中的一个服务器。 作为集群的一部分，它存储数据，参与集群的索引和搜索功能。

一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于 `Elasticsearch` 集群中的哪些节点。

一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做 `“elasticsearch”` 的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做 `“elasticsearch”` 的集群中。

在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何 `Elasticsearch` 节点，这时启动一个节点，会默认创建并加入一个叫做 `“elasticsearch”` 的集群。

### 3.2 Windows集群部署

1. 创建 elasticsearch-7.8.0-cluster 文件夹，在内部复制三个 elasticsearch 服务

   - ![image-20220315112633788](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220315112633788.png)

2. 修改集群文件目录中每个节点的 config/elasticsearch.yml 配置文件

   1. > 如果有必要，删除每个节点中的 data 目录中所有内容 。

   2. `node-1001` 节点

      - ```yaml
        #节点 1 的配置信息：
        #集群名称，节点之间要保持一致
        cluster.name: my-elasticsearch
        #节点名称，集群内要唯一
        node.name: node-1001
        node.master: true
        node.data: true
        #ip 地址
        network.host: localhost
        #http 端口
        http.port: 1001
        #tcp 监听端口
        transport.tcp.port: 9301
        #集群内的可以被选为主节点的节点列表
        #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
        #跨域配置
        #action.destructive_requires_name: true
        http.cors.enabled: true
        http.cors.allow-origin: "*"
        ```

   3. `node-1002 ` 节点

      - ```yaml
        #节点 2 的配置信息：
        #集群名称，节点之间要保持一致
        cluster.name: my-elasticsearch
        #节点名称，集群内要唯一
        node.name: node-1002
        node.master: true
        node.data: true
        #ip 地址
        network.host: localhost
        #http 端口
        http.port: 1002
        #tcp 监听端口
        transport.tcp.port: 9302
        discovery.seed_hosts: ["localhost:9301"]
        discovery.zen.fd.ping_timeout: 1m
        discovery.zen.fd.ping_retries: 5
        #集群内的可以被选为主节点的节点列表
        #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
        #跨域配置
        #action.destructive_requires_name: true
        http.cors.enabled: true
        http.cors.allow-origin: "*"
        ```

   4. `node-1003` 节点

      - ```yaml
        #节点 3 的配置信息：
        #集群名称，节点之间要保持一致
        cluster.name: my-elasticsearch
        #节点名称，集群内要唯一
        node.name: node-1003
        node.master: true
        node.data: true
        #ip 地址
        network.host: localhost
        #http 端口
        http.port: 1003
        #tcp 监听端口
        transport.tcp.port: 9303
        #候选主节点的地址，在开启服务后可以被选为主节点
        discovery.seed_hosts: ["localhost:9301", "localhost:9302"]
        discovery.zen.fd.ping_timeout: 1m
        discovery.zen.fd.ping_retries: 5
        #集群内的可以被选为主节点的节点列表
        #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
        #跨域配置
        #action.destructive_requires_name: true
        http.cors.enabled: true
        http.cors.allow-origin: "*"
        ```

3. 用 `Postman`，查看集群状态

   - ```json
     GET http://127.0.0.1:1001/_cluster/health
     GET http://127.0.0.1:1002/_cluster/health
     GET http://127.0.0.1:1003/_cluster/health
     ```

   - 返回结果皆为

   - ```json
     {
         "cluster_name": "my-application",
         "status": "green",
     /*
         status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：
     	1. green： 所有的主分片和副本分片都正常运行。
     	2. yellow：所有的主分片都正常运行，但不是所有的副本分片都正常运行。
     	3. red：   有主分片没能正常运行。
     */
         "timed_out": false,
         "number_of_nodes": 3,
         "number_of_data_nodes": 3,
         "active_primary_shards": 0,
         "active_shards": 0,
         "relocating_shards": 0,
         "initializing_shards": 0,
         "unassigned_shards": 0,
         "delayed_unassigned_shards": 0,
         "number_of_pending_tasks": 0,
         "number_of_in_flight_fetch": 0,
         "task_max_waiting_in_queue_millis": 0,
         "active_shards_percent_as_number": 100.0
     }
     ```

4. 用 Postman，在一节点增加索引，另一节点获取索引

   - 向集群中的 `node-1001` 节点增加索引：

   - ```json
     #PUT http://127.0.0.1:1001/user
     ```

   - 返回结果

   - ```json
     {
         "acknowledged": true,
         "shards_acknowledged": true,
         "index": "user"
     }
     ```

   - 向集群中的 `node-1003` 节点获取索引：

   - ```json
     #GET http://127.0.0.1:1003/user
     ```

   - 获取结果

   - ```json
     {
         "user": {
             "aliases": {},
             "mappings": {},
             "settings": {
                 "index": {
                     "creation_date": "1617993035885",
                     "number_of_shards": "1",
                     "number_of_replicas": "1",
                     "uuid": "XJKERwQlSJ6aUxZEN2EV0w",
                     "version": {
                         "created": "7080099"
                     },
                     "provided_name": "user"
                 }
             }
         }
     }
     ```

### 3.3 Linux单节点部署

1. 下载软件  [下载Linux版的Elasticsearch](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-8-0)

2. 解压软件

   - ```bash
     # 解压缩
     tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/module
     # 改名
     mv elasticsearch-7.8.0 es
     ```

3. 创建用户

   1. 因为安全问题， `Elasticsearch` 不允许 `root` 用户直接运行，所以要创建新用户，在 `root` 用户中创建新用户。

   2. ```bash
      useradd es #新增 es 用户
      passwd es #为 es 用户设置密码
      userdel -r es #如果错了，可以删除再加
      chown -R es:es /opt/module/es #文件夹所有者
      ```

4. 修改配置文件

   - 修改 `/opt/module/es/config/elasticsearch.yml` 文件

     - ```yaml
       # 加入如下配置
       cluster.name: elasticsearch
       node.name: node-1
       network.host: 0.0.0.0
       http.port: 9200
       cluster.initial_master_nodes: ["node-1"]
       ```

   - 修改 `/etc/security/limits.conf`

     - ```bash
       # 在文件末尾中增加下面内容
       # 每个进程可以打开的文件数的限制
       es soft nofile 65536
       es hard nofile 65536
       ```

   - 修改 `/etc/security/limits.conf`

     - ```bash
       # 在文件末尾中增加下面内容
       # 每个进程可以打开的文件数的限制
       es soft nofile 65536
       es hard nofile 65536
       ```

   - 修改 `/etc/security/limits.d/20-nproc.conf`

     - ```bash
       # 在文件末尾中增加下面内容
       # 每个进程可以打开的文件数的限制
       es soft nofile 65536
       es hard nofile 65536
       # 操作系统级别对每个用户创建的进程数的限制
       * hard nproc 4096
       # 注： * 带表 Linux 所有用户名称
       ```

   -  修改 `/etc/sysctl.conf`

     - ```bash
       # 在文件中增加下面内容
       vm.max_map_count=655360
       ```

5. 直接加载 `sysctl -p`

6. 启动

   - ```bash
     cd /opt/module/es-cluster
     #启动
     bin/elasticsearch
     #后台启动
     bin/elasticsearch -d
     ```

   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/477065a2cb499a32871549c21e2fc487.png)

### 3.3 Linux集群部署

1. 修改 `/opt/module/es/config/elasticsearch.yml` 文件，分发文件。

   - ```yaml
     # 加入如下配置
     #集群名称
     cluster.name: cluster-es
     #节点名称， 每个节点的名称不能重复
     node.name: node-1
     #ip 地址， 每个节点的地址不能重复
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

2. 修改 `/etc/security/limits.conf` ，分发文件

   - ```bash
     # 在文件末尾中增加下面内容
     es soft nofile 65536
     es hard nofile 65536
     ```

3. 修改 `/etc/security/limits.d/20-nproc.conf`，分发文件

   - ```bash
     # 在文件末尾中增加下面内容
     es soft nofile 65536
     es hard nofile 65536
     \* hard nproc 4096
     \# 注： * 带表 Linux 所有用户名称
     ```

4. 修改 `/etc/sysctl.conf`

   - ```bash
     # 在文件中增加下面内容
     vm.max_map_count=655360
     ```

5. 加载 `sysctl -p`

6. 启动软件

   - ```bash
     cd /opt/module/es-cluster
     #启动
     bin/elasticsearch
     #后台启动
     bin/elasticsearch -d
     ```

   - ![img](https://img-blog.csdnimg.cn/img_convert/0412e37cb5249d1ff0e813ee87f49a50.png)



## 4. Elasticsearch进阶

### 4.1 核心概念

![img](https://img-blog.csdnimg.cn/img_convert/146a779da01f53e7f7a8d53132d3c7cf.png)

#### 4.1.1 Index索引

​		一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（必须全部是小写字母），并且当我们要对这个索引中的文档进行索引、搜索、更新和删除（CRUD）的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。

​		能搜索的数据必须索引，这样的好处是可以提高查询速度，比如：新华字典前面的目录就是索引的意思，目录可以提高查询速度。

**`Elasticsearch` 索引的精髓：一切设计都是为了提高搜索的性能。**

#### 4.1.2 Type类型

在一个索引中，你可以定义一种或多种类型。

一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定。通常，会为具有一组共同字段的文档定义一个类型。不同的版本，类型发生了不同的变化。

| 版本 |                      Type                       |
| :--: | :---------------------------------------------: |
| 5.x  |                  支持多种 type                  |
| 6.x  |                 只能有一种 type                 |
| 7.x  | 默认不再支持自定义索引类型（默认类型为： _doc） |

#### 4.1.3 Document文档

**一个文档是一个可被索引的基础信息单元，也就是一条数据。**

比如：你可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。文档以 `JSON（Javascript Object Notation`）格式来表示，而 `JSON` 是一个到处存在的互联网数据交互格式。

在一个 `index/type` 里面，你可以存储任意多的文档。

#### 4.1.4 Field字段

相当于是数据表的字段，对文档数据根据不同属性进行的分类标识。

#### 4.1.5 Mapping映射

​		`mapping` 是处理数据的方式和规则方面做一些限制，如：某个字段的数据类型、默认值、分析器、是否被索引等等。这些都是映射里面可以设置的，其它就是处理 `ES` 里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。

#### 4.1.6 Shards分片

​		一个索引可以存储超出单个节点硬件限制的大量数据。比如，一个具有 10 亿文档数据的索引占据 1TB 的磁盘空间，而任一节点都可能没有这样大的磁盘空间。 或者单个节点处理搜索请求，响应太慢。为了解决这个问题，**Elasticsearch 提供了将索引划分成多份的能力，每一份就称之为分片。**当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。

分片很重要，主要有两方面的原因：

1. 允许你水平分割 / 扩展你的内容容量。
2. 允许你在分片之上进行分布式的、并行的操作，进而提高性能/吞吐量。

至于一个分片怎样分布，它的文档怎样聚合和搜索请求，是完全由 `Elasticsearch` 管理的，对于作为用户的你来说，这些都是透明的，无需过分关心。

被混淆的概念是，一个 `Lucene` 索引 我们在 `Elasticsearch` 称作 分片 。 一个 `Elasticsearch` 索引是分片的集合。 当 `Elasticsearch` 在索引中搜索的时候， 他发送查询到每一个属于索引的分片（`Lucene` 索引），然后合并每个分片的结果到一个全局的结果集。

Lucene 是 Apache 软件基金会 Jakarta 项目组的一个子项目，提供了一个简单却强大的应用程式接口，能够做全文索引和搜寻。在 Java 开发环境里 Lucene 是一个成熟的免费开源工具。就其本身而言， Lucene 是当前以及最近几年最受欢迎的免费 Java 信息检索程序库。但 Lucene 只是一个提供全文搜索功能类库的核心工具包，而真正使用它还需要一个完善的服务框架搭建起来进行应用。

目前市面上流行的搜索引擎软件，主流的就两款： Elasticsearch 和 Solr, 这两款都是基于 Lucene 搭建的，可以独立部署启动的搜索引擎服务软件。由于内核相同，所以两者除了服务器安装、部署、管理、集群以外，对于数据的操作 修改、添加、保存、查询等等都十分类似。

#### 4.1.7 Replicas副本

​		在一个网络 / 云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的。为此目的， `Elasticsearch` 允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片(副本)。

复制分片之所以重要，有两个主要原因：

- 在分片/节点失败的情况下，**提供了高可用性**。因为这个原因，注意到**复制分片从不与原/主要`(original/primary)`分片置于同一节点上**是非常重要的。
- 扩展你的搜索量/吞吐量，因为搜索可以在所有的副本上并行运行。

总之，每个索引可以被分成多个分片。一个索引也可以被复制 0 次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。

分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。

默认情况下，Elasticsearch 中的每个索引被分为 1 个主分片和 1 个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有 1 个主分片和另外 1 个复制分片（1 个完全拷贝），这样的话每个索引总共就有 2 个分片， 我们需要根据索引需要确定分片个数。

#### 4.1.8 Allocation分配

将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。这个过程是由 `master` 节点完成的。

### 4.2 系统架构-简介

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/e4d13427545dc174eb9ccface85c1f0c.png)

一个运行中的 `Elasticsearch` 实例称为一个节点，而集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为主节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 `Elasticsearch` 对这一切的管理都是透明的。

### 4.3 分布式集群

#### 4.3.1 单节点集群

我们在包含一个空节点的集群内创建名为 `users` 的索引，为了演示目的，我们将分配 `3` 个主分片和一份副本（每个主分片拥有一个副本分片）。

```json
#PUT http://127.0.0.1:1001/users
{
    "settings" : {
        "number_of_shards" : 3,
        "number_of_replicas" : 1
    }
}
```

集群现在是拥有一个索引的单节点集群。所有 `3` 个主分片都被分配在 `node-1` 。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/54ee6753be248cc7d345b38a0eae7d96.png)

通过 `elasticsearch-head` 插件（一个Chrome插件）查看集群情况 。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/e8b15d0b243d486e91f478a220da63bf.png)

- 集群健康值 `yellow( 3 of 6 )`：表示当前集群的全部主分片都正常运行，但是副本分片没有全部处在正常状态。
- ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/489b6de480112879a00067b793bde685.png)  三个主分片正常
- ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/3ce9a78d26ee762f0a7a8abf7817a58e.png)  3 个副本分片都是 Unassigned，它们都没有被分配到任何节点。 **在同 一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点 上的所有副本数据。**

#### 4.3.2 故障转移

当集群中只有一个节点在运行时，意味着会有一个单点故障问题——没有冗余。 幸运的是，我们只需再启动一个节点即可防止数据丢失。当你在同一台机器上启动了第二个节点时，只要它和第一个节点有同样的 `cluster.name` 配置，它就会自动发现集群并加入到其中。但是在不同机器上启动节点的时候，为了加入到同一集群，你需要配置一个可连接到的单播主机列表。之所以配置为使用单播发现，以防止节点无意中加入集群。只有在同一台机器上
运行的节点才会自动组成集群。

如果启动了第二个节点，集群将会拥有两个节点：所有主分片和副本分片都已被分配 。

![img](https://img-blog.csdnimg.cn/img_convert/bf76cb1bfbdf07555918d9055817ab44.png)

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/18db400822b83e727d6206f486b7b2ea.png)

- 集群健康值 `green( 3 of 6 )`：表示所有 `6` 个分片（包括 `3` 个主分片和 `3` 个副本分片）都在正常运行。
- ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/e485d8263a4aa3a94af0be951bd5a241.png)    三个主分片正常
- ![img](https://img-blog.csdnimg.cn/img_convert/e485d8263a4aa3a94af0be951bd5a241.png)   第二个节点加入到集群后， 3 个副本分片将会分配到这个节点上——每 个主分片对应一个副本分片。这意味着当集群内任何一个节点出现问题时，我们的数据都完好无损。所有新近被索引的文档都将会保存在主分片上，然后被并行的复制到对应的副本分片上。这就保证了我们既可以从主分片又可以从副本分片上获得文档。

#### 4.3.3 水平扩容

怎样为我们的正在增长中的应用程序按需扩容呢？当启动了第三个节点，我们的集群将会拥有三个节点的集群 : 为了分散负载而对分片进行重新分配 。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/d527e26aa2bccdf54b11410024eadc92.png)

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/6985fe14454c1269204478320d089bd7.png)

- 集群健康值 `green( 6 of 6 )`：表示所有 6 个分片（包括 3 个主分片和 3 个副本分片）都在正常运行。
- ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/9494419153adb44bedb395ac5d7bc488.png) Node 1 和 Node 2 上各有一个分片被迁移到了新的 Node 3 节点，现在每个节点上都拥有 2 个分片， 而不是之前的 3 个。 这表示每个节点的硬件资源（CPU, RAM, I/O）将被更少的分片所共享，每个分片 的性能将会得到提升。

分片是一个功能完整的搜索引擎，它拥有使用一个节点上的所有资源的能力。 我们这个拥有 6 个分片（3 个主分片和 3 个副本分片）的索引可以最大扩容到 6 个节点，每个节点上存在一个分片，并且每个分片拥有所在节点的全部资源。

**但是如果我们想要扩容超过 6 个节点怎么办呢？**

​		主分片的数目在索引创建时就已经确定了下来。实际上，这个数目定义了这个索引能够存储的最大数据量。（实际大小取决于你的数据、硬件和使用场景。） 但是，读操作——搜索和返回数据——可以同时被主分片或副本分片所处理，所以当你拥有越多的副本分片时，也将拥有越高的吞吐量。

在运行中的集群上是可以动态调整副本分片数目的，我们可以按需伸缩集群。让我们把副本数从默认的 1 增加到2。

```json
#PUT http://127.0.0.1:1001/users/_settings

{
    "number_of_replicas" : 2
}
```

`users` 索引现在拥有 9 个分片： 3 个主分片和 6 个副本分片。 这意味着我们可以将集群扩容到 9 个节点，每个节点上一个分片。相比原来 3 个节点时，集群搜索性能可以提升 3 倍。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/97fd01e34e5d8df23d226c4fef157801.png)

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/8bf9dbf0cec5b7875bf8aa9d17a9a67c.png)

当然，如果只是在相同节点数目的集群上增加更多的副本分片并不能提高性能，因为每个分片从节点上获得的资源会变少，你需要增加更多的硬件资源来提升吞吐量。

但是更多的副本分片数提高了数据冗余量：按照上面的节点配置，我们可以在失去 2 个节点的情况下不丢失任何数据。

#### 4.3.4 应对故障

如果第一个节点 `node-1` 宕机了，

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/44e841004be934e6bce08187ca3852bb.png)

因为宕机的是主节点。而集群必须拥有一个主节点来保证正常工作，所以发生的第一件事情就是选举一个新的主节点： `Node 2` 。在  `Node-1` 宕机的同时也失去了主分片 1 和 2 ，并且在缺失主分片的时候索引也不能正常工作。 如果此时来检查集群的状况，我们看到的状态将会为 red ：不是所有主分片都在正常工作。

幸运的是，在其它节点上存在着这两个主分片的完整副本， 所以新的主节点立即将这些分片在 `Node 2` 和 `Node 3` 上对应的副本分片提升为主分片， 此时集群的状态将会为 `yellow`。这个提升主分片的过程是瞬间发生的。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/e956bda7e0d005699e27760d4193d101.png)

**为什么我们集群状态是 yellow 而不是 green 呢？**

虽然我们拥有所有的三个主分片，但是同时设置了每个主分片需要对应 2 份副本分片，而此时只存在一份副本分片。 所以集群不能为 green 的状态，不过我们不必过于担心：如果我们同样关闭了 Node 2 ，我们的程序 依然 可以保持在不丢任何数据的情况下运行，因为 Node 3 为每一个分片都保留着一份副本。

**节点 `node-1` 恢复正常**

集群可以将缺失的副本分片再次进行分配，那么集群的状态也将恢复成之前的状态。 如果 `Node 1` 依然拥有着之前的分片，它将尝试去重用它们，同时仅从主分片复制发生了修改的数据文件。和**之前的集群相比，只是 `Master` 节点切换了。**

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/37eea6a8dae7ba908312f2ebf0eced11.png)

### 4.4 路由计算 & 分片控制

#### 4.4.1 路由计算

​		当索引一个文档的时候，文档会被存储到一个主分片中。 `Elasticsearch` 如何知道一个文档应该存放到哪个分片中呢？当我们创建文档时，它如何决定这个文档应当被存储在分片 1 还是分片 2 中呢？

​	首先这肯定不会是随机的，否则将来要获取文档的时候我们就不知道从何处寻找了。实际上，这个过程是根据下面这个公式决定的：

```java
shard = hash(routing) % number_of_primary_shards
```

`routing` 是一个可变值，默认是文档的  `_id` ，也可以设置成一个自定义的值。 `routing` 通过 `hash` 函数生成一个数字，然后这个数字再除以 `number_of_primary_shards` （主分片的数量）后得到余数 。这个分布在 `0 `到 `number_of_primary_shards-1` 之间的余数，就是我们所寻求的文档所在分片的位置。
![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/9c34e8603887c2bed475416e3b67cd9a.png)

这就解释了为什么我们要在创建索引的时候就确定好主分片的数量并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。

所有的文档 `API(get,index,delete,bulk,update以及mget)`都接受一个叫做 `routing` 的路由参数，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档----例如所有属于同一个用户的文档都被存储到同一个分片中。

#### 4.4.2 分片控制

​		我们可以发送请求到集群中的任一节点。每个节点都有能力处理任意请求。每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上。在下面的例子中，如果将所有的请求发送到 `Node 1001`，我们将其称为协调节点 **`coordinating node`**。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/3940d6cdb197259368542b86384911a4.png)

当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点。

### 4.5 数据写流程

新建、索引和删除请求都是写操作， 必须在主分片上面完成之后才能被复制到相关的副本分片。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/418356a32516c222a8d366df021276c2.png)

在客户端收到成功响应时，文档变更已经在主分片和所有副本分片执行完成，变更是安全的。有一些可选的**请求参数**允许您影响这个过程，可能以数据安全为代价提升性能。这些选项很少使用，因为 `Elasticsearch` 已经很快，但是为了完整起见， 请参考下文：

1. `consistency`
   - 即一致性。在默认设置下，即使仅仅是在试图执行一个写操作之前，主分片都会要求必须要有规定数量`quorum`（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态，才会去执行写操作（其中分片副本可以是主分片或者副本分片）。这是为了避免在发生网络分区故障 `(network partition)` 的时候进行写操作，进而导致数据不一致。 规定数量即： `int((primary + number_of_replicas) / 2 ) + 1`
   - `consistency` 参数的值可以设为：
     - `one` ：只要主分片状态 ok 就允许执行写操作。
     - `all`：必须要主分片和所有副本分片的状态没问题才允许执行写操作。
     - `quorum`：默认值为 `quorum` , 即大多数的分片副本状态没问题就允许执行写操作。
   - 注意，规定数量的计算公式中 `number_of_replicas` 指的是在索引设置中的设定副本分片数，而不是指当前处理活动状态的副本分片数。如果你的索引设置中指定了当前索引拥有3个副本分片，那规定数量的计算结果即：`int((1 primary + 3 replicas) / 2) + 1 = 3`，如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，也因此您将无法索引和删除任何文档。
2. `timeout`
   - 如果没有足够的副本分片会发生什么？
     - `Elasticsearch` 会等待，希望更多的分片出现。默认情况下，它最多等待 1 分钟。 如果你需要，你可以使用 `timeout` 参数使它更早终止：100是100 毫秒，30s是30秒。

新索引默认有 `1` 个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当 `number_of_replicas` 大于 `1` 的时候，规定数量才会执行。

### 4.6 数据读流程

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/7139df83ee6f7a59c5d3252d34cc8762.png)

​		在处理读取请求时，协调结点在每次请求的时候都会通过**轮询所有的副本分片来达到负载均衡**。在文档被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报告文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的。

### 4.7 更新流程 & 批量操作流程

#### 4.7.1 更新流程

部分更新一个文档结合了先前说明的读取和写入流程：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/ca41993144aee98311671278725437cd.png)

部分更新一个文档的步骤如下：

1. 客户端向 `Node 1` 发送更新请求。
2. 它将请求转发到主分片所在的 `Node 3` 。
3. `Node 3` 从主分片检索文档，修改 `_source` 字段中的 `JSON`，并且尝试重新索引主分片的文档。如果文档已经被另一个进程修改，它会重试步骤 `3`，超过 `retry_on_conflict` 次后放弃。
4. 如果 `Node 3` 成功地更新文档，它将新版本的文档并行转发到 `Node 1` 和 `Node 2`上的副本分片，重新建立索引。一旦所有副本分片都返回成功，Node 3向协调节点也返回成功，协调节点向客户端返回成功。

当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。请记住，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。 如果 `Elasticsearch` 仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档。

#### 4.7.2 批量操作

**mget和 bulk API的模式类似于单文档模式。**区别在于协调节点知道每个文档存在于哪个分片中。它将整个多文档请求分解成每个分片的多文档请求，并且将这些请求并行转发到每个参与节点。

协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/b90ea9c79138d8361ca339cff205fdb0.png)

**用单个 mget 请求取回多个文档所需的步骤顺序:**

1. 客户端向 Node 1 发送 mget 请求。
2. Node 1为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复，Node 1 构建响应并将其返回给客户端。

可以对 `docs` 数组中每个文档设置 `routing` 参数。

`bulk API`， 允许在单个批量请求中执行多个创建、索引、删除和更新请求。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/83499315a7b8ab81471a88f3e142f0a8.png)

**bulk API 按如下步骤顺序执行：**

1. 客户端向 `Node 1` 发送 `bulk` 请求。
2. `Node 1` 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。
3. 主分片一个接一个按顺序执行每个操作。当每个操作成功时,主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

### 4.8 倒排索引

分片是Elasticsearch最小的工作单元。但是究竟什么是一个分片，它是如何工作的？

传统的数据库每个字段存储单个值，但这对全文检索并不够。文本字段中的每个单词需要被搜索，对数据库意味着需要单个字段有索引多值的能力。最好的支持是一个字段多个值需求的数据结构是**倒排索引**。

#### 4.8.1 原理

`Elasticsearch` 使用一种称为 **倒排索引** 的结构，它适用于快速的全文搜索。

见其名，知其意，有倒排索引，肯定会对应有正向索引。正向索引（forward index），反向索引（inverted index）更熟悉的名字是**倒排索引**。

所谓的**正向索引**，就是搜索引擎会将待搜索的文件都对应一个文件ID，搜索时将这个ID和搜索关键字进行对应，形成 `K-V` 对，然后对关键字进行统计计数。

但是互联网上收录在搜索引擎中的文档的数目是个天文数字，这样的索引结构根本无法满足实时返回排名结果的要求。所以，搜索引擎会将正向索引重新构建为倒排索引，即把文件 `ID` 对应到关键词的映射转换为关键词到文件 `ID` 的映射，每个关键词都对应着一系列的文件，这些文件中都出现这个关键词。

#### 4.8.2 例子

一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。例如，假设我们有两个文档，每个文档的 `content` 域包含如下内容：

- The quick brown fox jumped over the lazy dog
- Quick brown foxes leap over lazy dogs in summer

为了创建倒排索引，我们首先将每个文档的 `content` 域拆分成单独的词（我们称它为词条或 `tokens` )，创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。结果如下所示：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/3cc642e9bae776c3e617f9d117d41e21.png)

现在，如果我们想搜索 `quick` `brown` ，我们只需要查找包含每个词条的文档：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/f26aaa01e011edfa68736956b2f1ddea.png)

两个文档都匹配，但是第一个文档比第二个匹配度更高。如果我们使用仅计算匹配词条数量的简单相似性算法，那么我们可以说，对于我们查询的相关性来讲，第一个文档比第二个文档更佳。

但是，我们目前的倒排索引有一些问题：

- `Quick`和`quick`以独立的词条出现，然而用户可能认为它们是相同的词。
- `fox`和`foxes`非常相似，就像`dog`和`dogs`；他们有相同的词根。
- `jumped`和`leap`，尽管没有相同的词根，但他们的意思很相近。他们是同义词。

使用前面的索引搜索`+Quick` `+fox`不会得到任何匹配文档。(记住，＋前缀表明这个词必须存在）。

只有同时出现`Quick`和`fox` 的文档才满足这个查询条件，但是第一个文档包含`quick` `fox` ，第二个文档包含`Quick` `foxes` 。

我们的用户可以合理的期望两个文档与查询匹配。我们可以做的更好。

如果我们将词条规范为标准模式，那么我们可以找到与用户搜索的词条不完全一致，但具有足够相关性的文档。例如：

- `Quick`可以小写化为`quick`。
- `foxes`可以词干提取变为词根的格式为`fox`。类似的，`dogs`可以为提取为`dog`。
- `jumped`和`leap`是同义词，可以索引为相同的单词`jump` 

现在索引看上去像这样：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/19813d1918c89461303377444cf85c8c.png)

这还远远不够。我们搜索 `+Quick +fox` 仍然会失败，因为在我们的索引中，已经没有 `Quick` 了。但是，如果我们对搜索的字符串使用与 `content` 域相同的标准化规则，会变成查询 `+quick +fox`，这样两个文档都会匹配！分词和标准化的过程称为分析，这非常重要。你只能搜索在索引中出现的词条，所以索引文本和查询字符串必须标准化为相同的格式。

### 4.9 文档搜索

#### 4.9.1 不可改变的倒排索引

早期的全文检索会为整个文档集合建立一个很大的倒排索引并将其写入到磁盘。 一旦新的索引就绪，旧的就会被其替换，这样最近的变化便可以被检索到。

倒排索引被写入磁盘后是不可改变的，即它永远不会修改，但是可以被删除。

- 不需要锁。如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。

- 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升。

- 其它缓存(像 `filter` 缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化。

- 写入单个大的倒排索引允许数据被压缩，减少磁盘 `IO` 和需要被缓存到内存的索引的使用量。

当然，一个不变的索引也有不好的地方。主要事实是它是不可变的! 你不能修改它。如果你需要让一个新的文档可被搜索，你需要重建整个索引。这要么对一个索引所能包含的数据量造成了很大的限制，要么对索引可被更新的频率造成了很大的限制。

#### 4.9.2 动态更新索引

如何在保留不变性的前提下实现倒排索引的更新？

答案是：用更多的索引。通过增加新的补充索引来反映新近的修改，而不是直接重写整个倒排索引。每一个倒排索引都会被轮流查询到，从最早的开始查询完后再对结果进行合并。

Elasticsearch基于Lucene，这个java库引入了**按段搜索**的概念。每一段本身都是一个倒排索引，但索引在 Lucene 中除表示所有段的集合外，还增加了提交点的概念—一个列出了所有已知段的文件。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/9ee1adbb2d55e710257e01b812a6d8cf.png)

按段搜索会以如下流程执行：

1. 新文档被收集到内存索引缓存。
   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/9d499fde966ee9825fa5a424d8357489.png)
2. 不时地, 缓存被提交。
   1. 一个新的段，一个追加的倒排索引，被写入磁盘。
   2. 一个新的包含新段名字的提交点被写入磁盘。
   3. 磁盘进行同步，所有在文件系统缓存中等待的写入都刷新到磁盘，以确保它们被写入物理文件
3. 新的段被开启，让它包含的文档可见以被搜索。
4. 内存缓存被清空，等待接收新的文档。
   - ![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/f74828ff58cc4635a97e88706a221e50.png)

当一个查询被触发，所有已知的段按顺序被查询。词项统计会对所有段的结果进行聚合，以保证每个词和每个文档的关联都被准确计算。这种方式可以用相对较低的成本将新文档添加到索引。

段是不可改变的，所以既不能从把文档从旧的段中移除，也不能修改旧的段来进行反映文档的更新。取而代之的是，每个提交点会包含一个 `.del ` 文件，文件中会列出这些被删除文档的段信息。

当一个**文档被“删除”**时，它实际上只是在 `.del` 文件中被标记删除。一个被标记删除的文档仍然可以被查询匹配到，但它会在最终结果被返回前从结果集中移除。

**文档更新**也是类似的操作方式：当一个文档被更新时，旧版本文档被标记删除，文档的新版本被索引到一个新的段中。可能两个版本的文档都会被一个查询匹配到，但被删除的那个旧版本文档在结果集返回前就已经被移除。

### 4.10 文档刷新&文档刷新