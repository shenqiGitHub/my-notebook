> 引言：上一节我们学习了ES的数据类型，有同学反馈说，里面的语句看不懂。今天，TeHero就为大家讲解ES索引和文档的CURD的操作。掌握了基本操作才能更好的系统学习，让我们开始吧！【关注公众号：**ZeroTeHero，系统学习ES**】

## 1、索引的CURD

**1）新增**

```
# 创建索引名为 tehero_index 的索引
PUT /tehero_index?pretty
{
# 索引设置
  "settings": {
    "index": {
      "number_of_shards": 1, # 分片数量设置为1，默认为5
      "number_of_replicas": 1 # 副本数量设置为1，默认为1
    }
  },
# 映射配置
  "mappings": {
    "_doc": { # 类型名，强烈建议设置为 _doc
      "dynamic": false, # 动态映射配置
# 字段属性配置
      "properties": {
        "id": {
          "type": "integer"  # 表示字段id，类型为integer
        },
        "name": {
          "type": "text",
          "analyzer": "ik_max_word", # 存储时的分词器
          "search_analyzer": "ik_smart"  # 查询时的分词器
        },
        "createAt": {
          "type": "date"
        }
      }
    }
  }
}
```

> 注：**dynamic：是动态映射的开关**，有3种状态：true 动态添加新的字段--缺省；推荐使用）false 忽略新的字段,不会添加字段映射，但是会存在于_source中；（strict 如果遇到新字段抛出异常；

```
# 返回值如下：
{
  "acknowledged": true, # 是否在集群中成功创建了索引
  "shards_acknowledged": true,
  "index": "tehero_index"
}
```

**2）查询**

```
GET /tehero_index  # 索引名，可以同时检索多个索引或所有索引
如：GET /*    GET /tehero_index,other_index

GET /_cat/indices?v  #查看所有 index
```

结果：

```
{
  "tehero_index": {
    "aliases": {},
    "mappings": {
      "_doc": {
        "dynamic": "false",
        "properties": {
          "createAt": {
            "type": "date"
          },
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1589271136921",
        "number_of_shards": "1",
        "number_of_replicas": "1",
        "uuid": "xueDIxeUQnGBQTms65wA6Q",
        "version": {
          "created": "6050499"
        },
        "provided_name": "tehero_index"
      }
    }
  }
}
```

**3）修改**

> ES提供了一系列对index修改的语句，包括**副本数量的修改、新增字段、refresh_interval值的修改、索引分析器的修改（后面重点讲解）、别名的修改**（关于别名，TeHero后面会专门讲解，这是一个在实践中非常有用的操作）。【关注公众号：**ZeroTeHero，系统学习ES**】

先学习常用的语法：

```
# 修改副本数
PUT /tehero_index/_settings
{
    "index" : {
        "number_of_replicas" : 2
    }
}

# 修改分片刷新时间,默认为1s
PUT /tehero_index/_settings
{
    "index" : {
        "refresh_interval" : "2s"
    }
}

# 新增字段 age
PUT /tehero_index/_mapping/_doc
{
  "properties": {
    "age": {
      "type": "integer"
    }
  }
}
```

更新完后，我们再次查看索引配置：

```
GET /tehero_index
结果：
{
  "tehero_index": {
    "aliases": {},
    "mappings": {
      "_doc": {
        "dynamic": "false",
        "properties": {
          "age": {
            "type": "integer"
          },
          "createAt": {
            "type": "date"
          },
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
          }
        }
      }
    },
    "settings": {
      "index": {
        "refresh_interval": "2s",
        "number_of_shards": "1",
        "provided_name": "tehero_index",
        "creation_date": "1589271136921",
        "number_of_replicas": "2",
        "uuid": "xueDIxeUQnGBQTms65wA6Q",
        "version": {
          "created": "6050499"
        }
      }
    }
  }
}
已经修改成功
```

**4）删除**

```
# 删除索引
DELETE /tehero_index
# 验证索引是否存在
HEAD tehero_index
返回：404 - Not Found
```

## 2、文档的CURD

**1）新增**

```
# 新增单条数据，并指定es的id 为 1
PUT /tehero_index/_doc/1?pretty
{
  "name": "Te Hero"
}
# 新增单条数据，使用ES自动生成id
POST /tehero_index/_doc?pretty
{
  "name": "Te Hero2"
}

# 使用 op_type 属性，强制执行某种操作
PUT tehero_index/_doc/1?op_type=create
{
     "name": "Te Hero3"
}
注意：op_type=create强制执行时，若id已存在，ES会报“version_conflict_engine_exception”。
op_type 属性在实践中同步数据时是有用的，后面讲解数据库与ES的数据同步问题时，TeHero再为大家详细讲解。
```

> 我们查询数据，看下效果：GET /tehero_index/_doc/_search

```
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
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "tehero_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Te Hero"
        }
      },
      {
        "_index": "tehero_index",
        "_type": "_doc",
        "_id": "P7-FCHIBJxE1TMY0WNGN",
        "_score": 1,
        "_source": {
          "name": "Te Hero2"
        }
      }
    ]
  }
}
```

**2）修改**

```
# 根据id，修改单条数据
（ps：修改语句和新增语句相同，可以理解为根据ID，存在则更新；不存在则新增）
PUT /tehero_index/_doc/1?pretty
{
  "name": "Te Hero-update"
}# 根据查询条件id=10，修改name="更新后的name"（版本冲突而不会导致_update_by_query 中止）
POST tehero_index/_update_by_query
{
  "script": {
    "source": "ctx._source.name = params.name",
    "lang": "painless",
    "params":{
      "name":"更新后的name"
    }
  },
  "query": {
    "term": {
      "id": "10"
    }
  }
}
```

> 关于文档的更新，Update By Query API，对于该API的使用，TeHero将其归类为进阶知识，后续章节将为大家更深入的讲解。【关注公众号：**ZeroTeHero，系统学习ES**】

**3）查询**

```
# 1、根据id，获取单个数据
GET /tehero_index/_doc/1
结果：
{
  "_index": "tehero_index",
  "_type": "_doc",
  "_id": "1",
  "_version": 5,
  "found": true,
  "_source": {
    "name": "Te Hero-update",
    "age": 18
  }
}

# 2、获取索引下的所有数据
GET /tehero_index/_doc/_search
结果：
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
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "tehero_index",
        "_type": "_doc",
        "_id": "P7-FCHIBJxE1TMY0WNGN",
        "_score": 1,
        "_source": {
          "name": "Te Hero2"
        }
      },
      {
        "_index": "tehero_index",
        "_type": "_doc",
        "_id": "_update",
        "_score": 1,
        "_source": {
          "name": "Te Hero3"
        }
      },
      {
        "_index": "tehero_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Te Hero-update",
          "age": 18
        }
      }
    ]
  }
}

# 3、条件查询（下一节详细介绍）
GET /tehero_index/_doc/_search
{
  "query": {
    "match": {
      "name": "2"
    }
  }
}
结果：
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
    "total": 1,
    "max_score": 0.9808292,
    "hits": [
      {
        "_index": "tehero_index",
        "_type": "_doc",
        "_id": "P7-FCHIBJxE1TMY0WNGN",
        "_score": 0.9808292,
        "_source": {
          "name": "Te Hero2"
        }
      }
    ]
  }
}
```

**4）删除**

```
# 1、根据id，删除单个数据
DELETE /tehero_index/_doc/1

# 2、delete by query
POST tehero_index/_delete_by_query
{
  "query": {
    "match": {
     "name": "2"
    }
  }
}
```

## 3、批量操作 Bulk API

```
# 批量操作
POST _bulk
{ "index" : { "_index" : "tehero_test1", "_type" : "_doc", "_id" : "1" } }
{ "this_is_field1" : "this_is_index_value" }
{ "delete" : { "_index" : "tehero_test1", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "tehero_test1", "_type" : "_doc", "_id" : "3" } }
{ "this_is_field3" : "this_is_create_value" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "tehero_test1"} }
{ "doc" : {"this_is_field2" : "this_is_update_value"} }

# 查询所有数据
GET /tehero_test1/_doc/_search
结果：
{
  "took": 33,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "tehero_test1",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "this_is_field1": "this_is_index_value",
          "this_is_field2": "this_is_update_value"
        }
      },
      {
        "_index": "tehero_test1",
        "_type": "_doc",
        "_id": "3",
        "_score": 1,
        "_source": {
          "this_is_field3": "this_is_create_value"
        }
      }
    ]
  }
}
```

> 注：POST _bulk 都做了哪些操作呢？

> 1、若索引“tehero_test1”不存在，则创建一个名为“tehero_test1”的 index，同时若id = 1 的文档存在，则更新；不存在则插入一条 id=1 的文档；

> 2、删除 id=2 的文档；

> 3、插入 id=3 的文档；若文档已存在，则报异常；

> 4、更新 id = 1 的文档。

ps：**批量操作在实践中使用是比较多的，因为减少了IO，提高了效率！**

> 下节预告：倒排序索引是什么？【欢迎关注公众号：**ZeroTeHero，系统学习ES**】



最后附上ES的知识脑图【ps：如有需要，公众号后台回复ES，即可免费获取】

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowKOWCgl0CiaQ13Tg22jKtcibAacOMzictM92RzIF0IxwdbOSvEgolSCFQZSVX5oKMvwxViaoK8gC5Heuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)