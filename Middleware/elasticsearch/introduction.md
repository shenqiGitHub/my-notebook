**阅读目录**

[TOC]
------



## 推荐博客

阮一峰大神：http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html

ElasticSearch 权威指南（中文版）：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

Elasticsearch教程：https://www.sojson.com/blog/81.html

ik分词器下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases

[回到顶部](#推荐博客：)

##  与数据库的关系

![img](https://images2018.cnblogs.com/blog/1212358/201807/1212358-20180731214834847-795920663.png)

 

[回到顶部](#推荐博客：)

## 安装目录的介绍

![img](https://images2018.cnblogs.com/blog/1212358/201807/1212358-20180731220453242-350303152.png)

 

[回到顶部](#推荐博客：)

## 在单机环境下创建器群

一般我们要在单机状态下创建器群需要复制好几份 然后改端口啥的，es不需要，直接在后面加参数就ok，爽的一比

在windows平台下 ， 一个是端口  一个是数据的存储位置 ，成功之后会在es的根目录创建一个node2文件夹

.\elasticsearch.bat -E http.port=8200 -E path.data=node2

 

[回到顶部](#推荐博客)

## 安装ik分词器

 确认好自己的es版本

![img](https://img2018.cnblogs.com/blog/1212358/201811/1212358-20181107151300901-2037874711.png)

下载地址：https://github.com/medcl/elasticsearch-analysis-ik

然后把下载好的zip包解压到plugins文件夹下。

```
unzip elasticsearch-analysis-ik-6.3.0.zip -d ik-analyzer
```

然后重启elasticsearch：



### 测试

大家先不管语法，我们先测试一波。

在kibana控制台输入下面的请求：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

[回到顶部](#推荐博客：)

## 索引



### 增加索引



```
PUT heima
{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 5
  }
}
```



索引库demo 5个分片 1个备份



### 查看索引

```
GET /索引库名
```



### 删除索引

```
DELETE /索引库名
```

### 查看服务下的所有索引

```
GET _cat/indices

GET _cat/indices?v
```

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 表



### 创建表（映射配置）

索引有了，接下来肯定是添加数据。但是，在添加数据之前必须定义映射。

什么是映射？

映射是定义文档的过程，文档包含哪些字段，这些字段是否保存，是否索引，是否分词等



```
PUT /索引库名/_mapping/类型名称
{
  "properties": {
    "字段名": {
      "type": "类型",
      "index": true，
      "store": true，
      "analyzer": "分词器"
    }
  }
}

类型名称：就是前面将的type的概念，类似于数据库中的不同表字段名：任意填写 ，可以指定许多属性，例如： 
type：类型，可以是text、long、short、date、integer、object等 
index：是否索引，默认为true 
store：是否存储，默认为false 
analyzer：分词器，这里的ik_max_word即使用ik分词器
 
```

示例：

```
PUT heima/_mapping/goods
{
  "properties": {
    "title": {
      "type": "text",
      "analyzer": "ik_max_word"
    },
    "images": {
      "type": "keyword",
      "index": "false"
    },
    "price": {
      "type": "float"
    }
  }
}


响应结果：
{
  "acknowledged": true
}
 
```



### 查看表

```
GET /索引库名/_mapping
```

 

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 字段属性详解



### type

我们说几个关键的：

- String类型，又分两种：

  - text：可分词，不可参与聚合
  - keyword：不可分词，数据会作为完整字段进行匹配，可以参与聚合

- Numerical：数值类型，分两类

  - 基本数据类型：long、interger、short、byte、double、float、half_float
  - 浮点数的高精度类型：scaled_float
  - 需要指定一个精度因子，比如10或100。elasticsearch会把真实值乘以这个因子后存储，取出时再还原。

- Date：日期类型

  elasticsearch可以对日期格式化为字符串存储，但是建议我们存储为毫秒值，存储为long，节省空间。



### index

index影响字段的索引情况。

- true：字段会被索引，则可以用来进行搜索。默认值就是true
- false：字段不会被索引，不能用来搜索

index的默认值就是true，也就是说你不进行任何配置，所有字段都会被索引。

但是有些字段是我们不希望被索引的，比如商品的图片信息，就需要手动设置index为false。



### store

是否将数据进行额外存储。

在学习lucene和solr时，我们知道如果一个字段的store设置为false，那么在文档列表中就不会有这个字段的值，用户的搜索结果中不会显示出来。

但是在Elasticsearch中，即便store设置为false，也可以搜索到结果。

原因是Elasticsearch在创建文档索引时，会将文档中的原始数据备份，保存到一个叫做`_source`的属性中。而且我们可以通过过滤`_source`来选择哪些要显示，哪些不显示。

而如果设置store为true，就会在`_source`以外额外存储一份数据，多余，因此一般我们都会将store设置为false，事实上，**store的默认值就是false。**



### boost

激励因子，这个与lucene中一样 

其它的不再一一讲解，用的不多，大家参考官方文档：

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 数据增删改查



### 新增数据



```
POST /索引库名/类型名
{
    "key":"value"
}

---------------------------------------------------------------

POST /heima/goods/
{
    "title":"小米手机",
    "images":"http://image.leyou.com/12479122.jpg",
    "price":2699.00
}
```





### 通过kibana查看数据



```
get _search
{
    "query":{
        "match_all":{}
    }
}
---------------------------------------------------------------
{
  "_index": "heima",
  "_type": "goods",
  "_id": "r9c1KGMBIhaxtY5rlRKv",
  "_version": 1,
  "_score": 1,
  "_source": {
    "title": "小米手机",
    "images": "http://image.leyou.com/12479122.jpg",
    "price": 2699
  }
}
```



- `_source`：源文档信息，所有的数据都在里面。
- `_id`：这条文档的唯一标示，与文档自己的id字段没有关联 



### 自定义id

如果我们想要自己新增的时候指定id，可以这么做：



```
POST /索引库名/类型/id值
{
    ...
}
------------------------------------------------
POST /heima/goods/2
{
    "title":"大米手机",
    "images":"http://image.leyou.com/12479122.jpg",
    "price":2899.00
}
```





### 智能判断

在学习Solr时我们发现，我们在新增数据时，只能使用提前配置好映射属性的字段，否则就会报错。

不过在Elasticsearch中并没有这样的规定。

事实上Elasticsearch非常智能，你不需要给索引库设置任何mapping映射，它也可以根据你输入的数据来判断类型，动态添加数据映射。

测试一下：



```
POST /heima/goods/3
{
    "title":"超米手机",
    "images":"http://image.leyou.com/12479122.jpg",
    "price":2899.00,
    "stock": 200,
    "saleable":true
}

我们额外添加了stock库存，和saleable是否上架两个字段。

来看结果：

{
  "_index": "heima",
  "_type": "goods",
  "_id": "3",
  "_version": 1,
  "_score": 1,
  "_source": {
    "title": "超米手机",
    "images": "http://image.leyou.com/12479122.jpg",
    "price": 2899,
    "stock": 200,
    "saleable": true
  }
}
```



在看下索引库的映射关系:



```
{
  "heima": {
    "mappings": {
      "goods": {
        "properties": {
          "images": {
            "type": "keyword",
            "index": false
          },
          "price": {
            "type": "float"
          },
          "saleable": {
            "type": "boolean"
          },
          "stock": {
            "type": "long"
          },
          "title": {
            "type": "text",
            "analyzer": "ik_max_word"
          }
        }
      }
    }
  }
}
```



stock和saleable都被成功映射了。



### 修改数据

把刚才新增的请求方式改为PUT，就是修改了。不过修改必须指定id，

- id对应文档存在，则修改
- id对应文档不存在，则新增

比如，我们把id为3的数据进行修改：



```
PUT /heima/goods/3
{
    "title":"超大米手机",
    "images":"http://image.leyou.com/12479122.jpg",
    "price":3899.00,
    "stock": 100,
    "saleable":true
}

结果：

{
  "took": 17,
  "timed_out": false,
  "_shards": {
    "total": 9,
    "successful": 9,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "3",
        "_score": 1,
        "_source": {
          "title": "超大米手机",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 3899,
          "stock": 100,
          "saleable": true
        }
      }
    ]
  }
}
```





### 删除数据

删除使用DELETE请求，同样，需要根据id进行删除：

```
DELETE /索引库名/类型名/id值
```

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 查询

我们从4块来讲查询：

- 基本查询
- `_source`过滤
- 结果过滤
- 高级查询
- 排序 



### 基本查询：



```
GET /索引库名/_search
{
    "query":{
        "查询类型":{
            "查询条件":"查询条件值"
        }
    }
}
```



这里的query代表一个查询对象，里面可以有不同的查询属性

- 查询类型：
  - 例如：`match_all`，?`match`，`term`?，?`range` 等等
- 查询条件：查询条件会根据类型的不同，写法也有差异，后面详细讲解 



### 查询所有（match_all)

```
GET /heima/_search
{
    "query":{
        "match_all": {}
    }
}
```

- `query`：代表查询对象
- `match_all`：代表查询所有 

结果：



```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "大米手机",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 2899
        }
      },
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "r9c1KGMBIhaxtY5rlRKv",
        "_score": 1,
        "_source": {
          "title": "小米手机",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 2699
        }
      }
    ]
  }
}
```



- took：查询花费时间，单位是毫秒
- time_out：是否超时
- _shards：分片信息
- hits：搜索结果总览对象
  - total：搜索到的总条数
  - max_score：所有结果中文档得分的最高分
  - hits：搜索结果的文档对象数组，每个元素是一条搜索到的文档信息
  - _index：索引库
  - _type：文档类型
  - _id：文档id
  - _score：文档得分
  - _source：文档的源数据 



### 匹配查询（match）

```
PUT /heima/goods/3
{
    "title":"小米电视4A",
    "images":"http://image.leyou.com/12479122.jpg",
    "price":3899.00
}
```

现在，索引库中有2部手机，1台电视：

- or关系 

`match`类型查询，会把查询条件进行分词，然后进行查询,多个词条之间是or的关系



```
GET /heima/_search
{
    "query":{
        "match":{
            "title":"小米电视"
        }
    }
}

结果：

"hits": {
    "total": 2,
    "max_score": 0.6931472,
    "hits": [
        {
            "_index": "heima",
            "_type": "goods",
            "_id": "tmUBomQB_mwm6wH_EC1-",
            "_score": 0.6931472,
            "_source": {
                "title": "小米手机",
                "images": "http://image.leyou.com/12479122.jpg",
                "price": 2699
            }
        },
        {
            "_index": "heima",
            "_type": "goods",
            "_id": "3",
            "_score": 0.5753642,
            "_source": {
                "title": "小米电视4A",
                "images": "http://image.leyou.com/12479122.jpg",
                "price": 3899
            }
        }
    ]
}
```



在上面的案例中，不仅会查询到电视，而且与小米相关的都会查询到，多个词之间是`or`的关系。

- and关系

某些情况下，我们需要更精确查找，我们希望这个关系变成`and`，可以这样做：



```
GET /heima/_search
{
    "query":{
        "match": {
          "title": {
            "query": "小米电视",
            "operator": "and"
          }
        }
    }
}

结果：

{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "3",
        "_score": 0.5753642,
        "_source": {
          "title": "小米电视4A",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 3899
        }
      }
    ]
  }
}
```



本例中，只有同时包含`小米`和`电视`的词条才会被搜索到。

- or和and之间？ 

在 `or` 与 `and` 间二选一有点过于非黑即白。 如果用户给定的条件分词后有 5 个查询词项，想查找只包含其中 4 个词的文档，该如何处理？将 operator 操作符参数设置成 `and` 只会将此文档排除。

有时候这正是我们期望的，但在全文搜索的大多数应用场景下，我们既想包含那些可能相关的文档，同时又排除那些不太相关的。换句话说，我们想要处于中间某种结果。

`match` 查询支持 `minimum_should_match` 最小匹配参数， 这让我们可以指定必须匹配的词项数用来表示一个文档是否相关。我们可以将其设置为某个具体数字，更常用的做法是将其设置为一个`百分数`，因为我们无法控制用户搜索时输入的单词数量：



```
GET /heima/_search
{
    "query":{
        "match":{
            "title":{
                "query":"小米曲面电视",
                "minimum_should_match": "75%"
            }
        }
    }
}
```



本例中，搜索语句可以分为3个词，如果使用and关系，需要同时满足3个词才会被搜索到。这里我们采用最小品牌数：75%，那么也就是说只要匹配到总词条数量的75%即可，这里3*75% 约等于2。所以只要包含2个词条就算满足条件了。



### 多字段查询（multi_match）

`multi_match`与`match`类似，不同的是它可以在多个字段中查询



```
GET /heima/_search
{
    "query":{
        "multi_match": {
            "query":    "小米",
            "fields":   [ "title", "subTitle" ]
        }
    }
}
```



本例中，我们会在title字段和subtitle字段中查询`小米`这个词



### 词条匹配(term)

`term` 查询被用于精确值 匹配，这些精确值可能是数字、时间、布尔或者那些**未分词**的字符串



```
GET /heima/_search
{
    "query":{
        "term":{
            "price":2699.00
        }
    }
}
-----------------------------------------------------------
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "r9c1KGMBIhaxtY5rlRKv",
        "_score": 1,
        "_source": {
          "title": "小米手机",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 2699
        }
      }
    ]
  }
}
```





### 多词条精确匹配(terms)

`terms` 查询和 term 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件：



```
GET /heima/_search
{
    "query":{
        "terms":{
            "price":[2699.00,2899.00,3899.00]
        }
    }
}

结果：

{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "大米手机",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 2899
        }
      },
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "r9c1KGMBIhaxtY5rlRKv",
        "_score": 1,
        "_source": {
          "title": "小米手机",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 2699
        }
      },
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "3",
        "_score": 1,
        "_source": {
          "title": "小米电视4A",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 3899
        }
      }
    ]
  }
}
```





## 添加别名

```
PUT /索引名/_alias/别名
```





[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 结果过滤

默认情况下，elasticsearch在搜索的结果中，会把文档中保存在`_source`的所有字段都返回。

如果我们只想获取其中的部分字段，我们可以添加`_source`的过滤



### 直接指定字段



```
GET /heima/_search
{
  "_source": ["title","price"],
  "query": {
    "term": {
      "price": 2699
    }
  }
}

返回的结果：

{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "r9c1KGMBIhaxtY5rlRKv",
        "_score": 1,
        "_source": {
          "price": 2699,
          "title": "小米手机"
        }
      }
    ]
  }
}
```





### 指定includes和excludes

我们也可以通过：

- includes：来指定想要显示的字段
- excludes：来指定不想要显示的字段ter

二者都是可选的。



```
GET /heima/_search
{
  "_source": {
    "includes":["title","price"]
  },
  "query": {
    "term": {
      "price": 2699
    }
  }
}

与下面的结果将是一样的：


GET /heima/_search
{
  "_source": {
     "excludes": ["images"]
  },
  "query": {
    "term": {
      "price": 2699
    }
  }
}
```



[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 高级查询



### 布尔组合（bool)

`bool`把各种其它查询通过`must`（与）、`must_not`（非）、`should`（或）的方式进行组合

==注意:**should在与must或者filter同级时，默认是不需要满足should中的任何条件**==
解决办法:

(待验证)

1、增加“minimum_should_match”字段，使其必须满足must和filter中的条件，并且必须满足should中的一个条件

2、或者将must改为must_not逻辑

3、在语音上把must的部分跟should部分进行改造，must内嵌入should，或者should联级

```
GET /heima/_search
{
    "query":{
        "bool":{
            "must":     { "match": { "title": "大米" }},
            "must_not": { "match": { "title":  "电视" }},
            "should":   { "match": { "title": "手机" }}
        }
    }
}
```

结果：

```
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "heima",
        "_type": "goods",
        "_id": "2",
        "_score": 0.5753642,
        "_source": {
          "title": "大米手机",
          "images": "http://image.leyou.com/12479122.jpg",
          "price": 2899
        }
      }
    ]
  }
}
```





### 范围查询(range)

`range` 查询找出那些落在指定区间内的数字或者时间



```
GET /heima/_search
{
    "query":{
        "range": {
            "price": {
                "gte":  1000.0,
                "lt":   2800.00
            }
        }
    }
}
```



`range`查询允许以下字符：

| 操作符 | 说明     |
| ------ | -------- |
| gt     | 大于     |
| gte    | 大于等于 |
| lt     | 小于     |
| lte    | 小于等于 |



### 模糊查询(fuzzy)

我们新增一个商品：

```
POST /heima/goods/4
{
    "title":"apple手机",
    "images":"http://image.leyou.com/12479122.jpg",
    "price":6899.00
}
```

`fuzzy` 查询是 `term` 查询的模糊等价。它允许用户搜索词条与实际词条的拼写出现偏差，但是偏差的编辑距离不得超过2：

```
GET /heima/_search
{
  "query": {
    "fuzzy": {
      "title": "appla"
    }
  }
}
```

上面的查询，也能查询到apple手机

我们可以通过`fuzziness`来指定允许的编辑距离：

```
GET /heima/_search
{
  "query": {
    "fuzzy": {
        "title": {
            "value":"appla",
            "fuzziness":1
        }
    }
  }
}
```

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 过滤(filter)



### **条件查询中进行过滤**

所有的查询都会影响到文档的评分及排名。如果我们需要在查询结果中进行过滤，并且不希望过滤条件影响评分，那么就不要把过滤条件作为查询条件来用。而是使用`filter`方式：



```
GET /heima/_search
{
    "query":{
        "bool":{
            "must":{ "match": { "title": "小米手机" }},
            "filter":{
                "range":{"price":{"gt":2000.00,"lt":3800.00}}
            }
        }
    }
}
```



注意：`filter`中还可以再次进行`bool`组合条件过滤。



### **无查询条件，直接过滤**

如果一次查询只有过滤，没有查询条件，不希望进行评分，我们可以使用`constant_score`取代只有 filter 语句的 bool 查询。在性能上是完全相同的，但对于提高查询简洁性和清晰度有很大帮助。



```
GET /heima/_search
{
    "query":{
        "constant_score":   {
            "filter": {
                 "range":{"price":{"gt":2000.00,"lt":3000.00}}
            }
        }
}
```



[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

##  排序



### 单字段排序

`sort` 可以让我们按照不同的字段进行排序，并且通过`order`指定排序的方式



```
GET /heima/_search
{
  "query": {
    "match": {
      "title": "小米手机"
    }
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
```





### 多字段排序

假定我们想要结合使用 price和 _score（得分） 进行查询，并且匹配的结果首先按照价格排序，然后按照相关性得分排序：



```
GET /goods/_search
{
    "query":{
        "bool":{
            "must":{ "match": { "title": "小米手机" }},
            "filter":{
                "range":{"price":{"gt":200000,"lt":300000}}
            }
        }
    },
    "sort": [
      { "price": { "order": "desc" }},
      { "_score": { "order": "desc" }}
    ]
}
```



[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 聚合aggregations

聚合可以让我们极其方便的实现对数据的统计、分析。例如：

- 什么品牌的手机最受欢迎？
- 这些手机的平均价格、最高价格、最低价格？
- 这些手机每月的销售情况如何？

实现这些统计功能的比数据库的sql要方便的多，而且查询速度非常快，可以实现实时搜索效果。

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 基本概念

Elasticsearch中的聚合，包含多种类型，最常用的两种，一个叫`桶`，一个叫`度量`：



### **桶（bucket）**

桶的作用，是按照某种方式对数据进行分组，每一组数据在ES中称为一个`桶`，例如我们根据国籍对人划分，可以得到`中国桶`、`英国桶`，`日本桶`……或者我们按照年龄段对人进行划分：0~10,10~20,20~30,30~40等。

Elasticsearch中提供的划分桶的方式有很多：

- Date Histogram Aggregation：根据日期阶梯分组，例如给定阶梯为周，会自动每周分为一组
- Histogram Aggregation：根据数值阶梯分组，与日期类似
- Terms Aggregation：根据词条内容分组，词条内容完全匹配的为一组
- Range Aggregation：数值和日期的范围分组，指定开始和结束，然后按段分组
- …… 

综上所述，我们发现bucket aggregations 只负责对数据进行分组，并不进行计算，因此往往bucket中往往会嵌套另一种聚合：metrics aggregations即度量



### **度量（metrics）**

分组完成以后，我们一般会对组中的数据进行聚合运算，例如求平均值、最大、最小、求和等，这些在ES中称为`度量`

比较常用的一些度量聚合方式：

- Avg Aggregation：求平均值
- Max Aggregation：求最大值
- Min Aggregation：求最小值
- Percentiles Aggregation：求百分比
- Stats Aggregation：同时返回avg、max、min、sum、count等
- Sum Aggregation：求和
- Top hits Aggregation：求前几
- Value Count Aggregation：求总数
- cardinality 去重计数  （[优化](https://www.elastic.co/guide/cn/elasticsearch/guide/current/cardinality.html)）
- ......

为了测试聚合，我们先批量导入一些数据

创建索引：



```
PUT /cars
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "transactions": {
      "properties": {
        "color": {
          "type": "keyword"
        },
        "make": {
          "type": "keyword"
        }
      }
    }
  }
}
```



**注意**：在ES中，需要进行聚合、排序、过滤的字段其处理方式比较特殊，因此不能被分词。这里我们将color和make这两个文字类型的字段设置为keyword类型，这个类型不会被分词，将来就可以参与聚合

导入数据



```
POST /cars/transactions/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }
```



[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 聚合为桶

首先，我们按照 汽车的颜色`color`来划分`桶`



```
GET /cars/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color"
            }
        }
    }
}
```



- size： 查询条数，这里设置为0，因为我们不关心搜索到的数据，只关心聚合结果，提高效率
- aggs：声明这是一个聚合查询，是aggregations的缩写
  - popular_colors：给这次聚合起一个名字，任意。
  - terms：划分桶的方式，这里是根据词条划分
    - field：划分桶的字段 

结果：



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
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "popular_colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4
        },
        {
          "key": "blue",
          "doc_count": 2
        },
        {
          "key": "green",
          "doc_count": 2
        }
      ]
    }
  }
}
```



- hits：查询结果为空，因为我们设置了size为0
- aggregations：聚合的结果
- popular_colors：我们定义的聚合名称
- buckets：查找到的桶，每个不同的color字段值都会形成一个桶
  - key：这个桶对应的color字段的值
  - doc_count：这个桶中的文档数量

通过聚合的结果我们发现，目前红色的小车比较畅销！

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 桶内度量

前面的例子告诉我们每个桶里面的文档数量，这很有用。 但通常，我们的应用需要提供更复杂的文档度量。 例如，每种颜色汽车的平均价格是多少？

因此，我们需要告诉Elasticsearch`使用哪个字段`，`使用何种度量方式`进行运算，这些信息要嵌套在`桶`内，`度量`的运算会基于`桶`内的文档进行

现在，我们为刚刚的聚合结果添加 求价格平均值的度量：



```
GET /cars/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color"
            },
            "aggs":{
                "avg_price": { 
                   "avg": {
                      "field": "price" 
                   }
                }
            }
        }
    }
}
```



- aggs：我们在上一个aggs(popular_colors)中添加新的aggs。可见`度量`也是一个聚合,度量是在桶内的聚合
- avg_price：聚合的名称
- avg：度量的类型，这里是求平均值
- field：度量运算的字段 

结果：



```
...
  "aggregations": {
    "popular_colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4,
          "avg_price": {
            "value": 32500
          }
        },
        {
          "key": "blue",
          "doc_count": 2,
          "avg_price": {
            "value": 20000
          }
        },
        {
          "key": "green",
          "doc_count": 2,
          "avg_price": {
            "value": 21000
          }
        }
      ]
    }
  }
...
```



可以看到每个桶中都有自己的`avg_price`字段，这是度量聚合的结果

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 桶内嵌套桶

刚刚的案例中，我们在桶内嵌套度量运算。事实上桶不仅可以嵌套运算， 还可以再嵌套其它桶。也就是说在每个分组中，再分更多组。

比如：我们想统计每种颜色的汽车中，分别属于哪个制造商，按照`make`字段再进行分桶



```
GET /cars/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color"
            },
            "aggs":{
                "avg_price": { 
                   "avg": {
                      "field": "price" 
                   }
                },
                "maker":{
                    "terms":{
                        "field":"make"
                    }
                }
            }
        }
    }
}
```



- 原来的color桶和avg计算我们不变
- maker：在嵌套的aggs下新添一个桶，叫做maker
- terms：桶的划分类型依然是词条
- filed：这里根据make字段进行划分 

部分结果：



```
...
{"aggregations": {
    "popular_colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4,
          "maker": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "honda",
                "doc_count": 3
              },
              {
                "key": "bmw",
                "doc_count": 1
              }
            ]
          },
          "avg_price": {
            "value": 32500
          }
        },
        {
          "key": "blue",
          "doc_count": 2,
          "maker": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "ford",
                "doc_count": 1
              },
              {
                "key": "toyota",
                "doc_count": 1
              }
            ]
          },
          "avg_price": {
            "value": 20000
          }
        },
        {
          "key": "green",
          "doc_count": 2,
          "maker": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "ford",
                "doc_count": 1
              },
              {
                "key": "toyota",
                "doc_count": 1
              }
            ]
          },
          "avg_price": {
            "value": 21000
          }
        }
      ]
    }
  }
}
...
```



- 我们可以看到，新的聚合`maker`被嵌套在原来每一个`color`的桶中。
- 每个颜色下面都根据 `make`字段进行了分组
- 我们能读取到的信息：
  - 红色车共有4辆
  - 红色车的平均售价是 $32，500 美元。
  - 其中3辆是 Honda 本田制造，1辆是 BMW 宝马制造

[回到顶部](https://www.cnblogs.com/coder-lzh/p/9299247.html#_labelTop)

## 划分桶的其它方式

前面讲了，划分桶的方式有很多，例如：

- Date Histogram Aggregation：根据日期阶梯分组，例如给定阶梯为周，会自动每周分为一组
- Histogram Aggregation：根据数值阶梯分组，与日期类似
- Terms Aggregation：根据词条内容分组，词条内容完全匹配的为一组
- Range Aggregation：数值和日期的范围分组，指定开始和结束，然后按段分组

刚刚的案例中，我们采用的是Terms Aggregation，即根据词条划分桶。

接下来，我们再学习几个比较实用的：



### 阶梯分桶Histogram

原理： histogram是把数值类型的字段，按照一定的阶梯大小进行分组。你需要指定一个阶梯值（interval）来划分阶梯大小。

举例：比如你有价格字段，如果你设定interval的值为200，那么阶梯就会是这样的：

0，200，400，600，...

上面列出的是每个阶梯的key，也是区间的启点。

如果一件商品的价格是450，会落入哪个阶梯区间呢？计算公式如下：

```
bucket_key = Math.floor((value - offset) / interval) * interval + offset
```

value：就是当前数据的值，本例中是450

offset：起始偏移量，默认为0

interval：阶梯间隔，比如200

因此你得到的key = Math.floor((450 - 0) / 200) * 200 + 0 = 400

操作一下：

比如，我们对汽车的价格进行分组，指定间隔interval为5000：



```
GET /cars/_search
{
  "size":0,
  "aggs":{
    "price":{
      "histogram": {
        "field": "price",
        "interval": 5000
      }
    }
  }
}
```



结果：



```
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "price": {
      "buckets": [
        {
          "key": 10000,
          "doc_count": 2
        },
        {
          "key": 15000,
          "doc_count": 1
        },
        {
          "key": 20000,
          "doc_count": 2
        },
        {
          "key": 25000,
          "doc_count": 1
        },
        {
          "key": 30000,
          "doc_count": 1
        },
        {
          "key": 35000,
          "doc_count": 0
        },
        {
          "key": 40000,
          "doc_count": 0
        },
        {
          "key": 45000,
          "doc_count": 0
        },
        {
          "key": 50000,
          "doc_count": 0
        },
        {
          "key": 55000,
          "doc_count": 0
        },
        {
          "key": 60000,
          "doc_count": 0
        },
        {
          "key": 65000,
          "doc_count": 0
        },
        {
          "key": 70000,
          "doc_count": 0
        },
        {
          "key": 75000,
          "doc_count": 0
        },
        {
          "key": 80000,
          "doc_count": 1
        }
      ]
    }
  }
}
```



你会发现，中间有大量的文档数量为0 的桶，看起来很丑。

我们可以增加一个参数min_doc_count为1，来约束最少文档数量为1，这样文档数量为0的桶会被过滤

示例：



```
GET /cars/_search
{
  "size":0,
  "aggs":{
    "price":{
      "histogram": {
        "field": "price",
        "interval": 5000,
        "min_doc_count": 1
      }
    }
  }
}
```



结果：



```
{
  "took": 15,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "price": {
      "buckets": [
        {
          "key": 10000,
          "doc_count": 2
        },
        {
          "key": 15000,
          "doc_count": 1
        },
        {
          "key": 20000,
          "doc_count": 2
        },
        {
          "key": 25000,
          "doc_count": 1
        },
        {
          "key": 30000,
          "doc_count": 1
        },
        {
          "key": 80000,
          "doc_count": 1
        }
      ]
    }
  }
}
```



完美，！

如果你用kibana将结果变为柱形图，会更好看：



### 范围分桶range

范围分桶与阶梯分桶类似，也是把数字按照阶段进行分组，只不过range方式需要你自己指定每一组的起始和结束大小。