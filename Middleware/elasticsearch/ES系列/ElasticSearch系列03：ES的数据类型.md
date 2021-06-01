>    
>
> ​    引言：上一节，我们学习了ES的基本概念和ES的数据架构。今天，TeHero将为大家讲解ES的数据类型。

> ​    数据的存储，都是需要预先确定好数据的类型的，不管是关系型数据库mysql还是非关系型数据库MongoDB，都有一套数据类型系统（两者很类似，但也有区别）。**那么ES的数据类型有哪些呢？TeHero为你倾情讲解^~^。**

## ES的数据类型汇总

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/03-1.png)



ES数据类型汇总图（注意标红的类型）



从上图可以看到ES的数据类型和mysql或MongoDB的是很相似的，所以对于有数据结构基础的伙伴，这个知识点是非常轻松的。

TeHero将详细为大家介绍上图中标红的**4种数据类型**（数值类型就很一目了然）【ps：如果你还想了解其他的类型，可以直接进ES的官网阅读】，**让大家在以后的工作中能熟练使用，知道什么时候该用哪种类型，该怎么用。**

## 一、String 类型

> String类型可以和java的string、mysql的varchar等同，但是为何会分为text、keyword呢？这两者又有什么区别？

> ES作为全文检索引擎，它强大的地方就在于**分词**和倒排序索引。而 text 和 keyword 的区别就**在于是否分词**（ps：什么叫分词？举个简单例子，“中国我爱你”这句话，如果使用了分词，那么这句话在底层的储存可能就是“中国”、“我爱你”，**被拆分成了两个关键字**），分词后面TeHero会专门写文章进行讲解，敬请期待哟。

**1）text——会分词**

就拿刚才的例子来说，“中国我爱你”这句话，如果使用text类型储存，我们不去特殊定义它的分词器，**那么ES就会使用默认的分词器 standard 。**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



ES的分词器（可先有个概念）



下图就是“中国我爱你”的ES分词效果：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这意味着什么呢？如果你使用text类型去储存你**本不想分词的string类型**，你在查询的时候，查询结果**将违背你的预期。**

简单看个示例：

```
# 创建索引
PUT /toherotest
{
  "mappings": {
    "_doc":{
      "properties" : {
                "field1" : { "type" : "text" }
            }
    }
  }
}
# 存入数据
POST /toherotest/_doc/1
{
  "field1":"中国我爱你"
}
```

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/03-3.png)

```
# 查询 索引下的所有数据
GET /toherotest/_doc/_search
```

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/03-4.png)

条件查询，等价于mysql的 **where field1 = "中国我爱你"** 。**发现居然查询不到**

```
GET /toherotest/_doc/_search
{
  "query": {
    "term": {
      "field1": {
        "value": "中国我爱你"
      }
    }
  }
}
```

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

再根据刚才的ES分词效果，我们**检索其中一个字**，**居然神奇的检索到了**

```
GET /toherotest/_doc/_search
{
  "query": {
    "term": {
      "field1": {
        "value": "中"
      }
    }
  }
}
```

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这是为什么呢？我们发现在使用term查询（等价于mysql的=）时却查不到结果，其实就是因为**text类型会分词，简单理解就是“中国我爱你”这句话在ES的倒排序索引中存储的是单个字**，所以无法检索。



**2）keywor——不会分词**

我们新增一个keyword类型的字段field2，再来看看检索效果：

```
# 测试分词效果
GET /_analyze
{
  "text": ["中国我爱你"],
  "analyzer": "keyword"
}
# 结果
{
  "tokens": [
    {
      "token": "中国我爱你",
      "start_offset": 0,
      "end_offset": 5,
      "type": "word",
      "position": 0
    }
  ]
}

# 新增 字段类型 keyword
PUT toherotest/_mapping/_doc
{
  "properties": {
    "field2": {
      "type": "keyword"
    }
  }
}
# 新增数据
PUT /toherotest/_doc/12
{
  "field2":"中国我爱你"
}
# 查询
GET /toherotest/_doc/_search
{
  "query": {
    "term": {
      "field2": {
        "value": "中国我爱你"
      }
    }
  }
}
```

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以发现，类型为keyword，**通过term是可以查询到，说明ES对keyword是没有分词的。**

## 二、date 时间类型 —— 可规定格式

对于date类型，和mysql的几乎一样，唯一的注意点就是，**储存的格式，ES是可以控制的**。

```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "date": {
          "type": "date"
        }
      }
    }
  }
}

PUT my_index/_doc/1
{ "date": "2015-01-01" }

PUT my_index/_doc/2
{ "date": "2015-01-01T12:10:30Z" }

PUT my_index/_doc/3
{ "date": 1420070400001 }

GET my_index/_search
{
  "sort": { "date": "asc"}
}
```

大家可以用kibana试试，**3种格式都可以的。**

同时ES的date类型**允许我们规定格式**，可以使用的格式有：

> yyyy-MM-dd HH:mm:ss

> yyyy-MM-dd

> epoch_millis（毫秒值）

```
# 规定格式如下：|| 表示或者
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "date": {
          "type":   "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}
```

**注意：一旦我们规定了格式，如果新增数据不符合这个格式，ES将会报错mapper_parsing_exception。**

## 三、复杂类型

> ES的复杂类型有3个，Array、object、nested。

1）Array：在Elasticsearch中，**数组不需要专用的字段数据类型**。默认情况下，**任何字段都可以包含零个或多个值**，但是，数组中的所有值都**必须具有相同的数据类型。**

> 举个简单例子理解下：比如上一个例子中的field1这个字段，可以只存储一个值“中国我爱你”，同时也可以存储一个数组：["这是","一个","数组"]

```
# 新增数据
POST /toherotest/_doc/2
{
  "field1":["这是","一个","数组"]
}
```

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/03-5.png)

2）object我相信大家都能理解；需要注意的是，**object类型的字段，也可以有多个值**，形成List<object>的数据结构。

> 重点：List<object>中的**object不允许彼此独立地索引查询**。这是什么意思呢？

举个简单例子：我们现在有2条数据：数据结构都是一个List<object>

> \# 第一条数据：[ { "name":"tohero1", "age":1 }, { "name":"tohero2", "age":2 } ]

> \# 第二条数据：[ { "name":"tohero1", "age":2 }, { "name":"tohero2", "age":1 } ]

如果此时我们的需求是，只要 **name = “tohero1”and “age”= 1** 的数据，根据我们常规的理解，只有第一条数据才能被检索出来，但是真的是这样么？我们写个例子看看：

```
# 添加 属性为object的字段 field3
PUT toherotest/_mapping/_doc
{
  "properties": {
    "field3": {
      "type": "object"
    }
  }
}
# 新增数据
POST /toherotest/_doc/3
{
  "field3":[ { "name":"tohero1", "age":1 }, { "name":"tohero2", "age":2 } ]
}
  
POST /toherotest/_doc/4
{
  "field3": [ { "name":"tohero1", "age":2 }, { "name":"tohero2", "age":1 } ]
}

#执行查询语句
GET /toherotest/_doc/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "field3.name": "tohero1"
          }
        },
        {
          "term": {
            "field3.age": 1
          }
        }
      ]
    }
  }
}
```

> （ps：现在看不懂查询语句，没关系，**主要是理解几个类型的差异，**可以后面学了查询语句再回头看查询语句）查询语句等价于mysql的 **where** **name = “tohero1”and “age”= 1**

**查询结果如下：**

```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1.287682,
    "hits": [
      {
        "_index": "toherotest",
        "_type": "_doc",
        "_id": "4",
        "_score": 1.287682,
        "_source": {
          "field3": [
            {
              "name": "tohero1",
              "age": 2
            },
            {
              "name": "tohero2",
              "age": 1
            }
          ]
        }
      },
      {
        "_index": "toherotest",
        "_type": "_doc",
        "_id": "3",
        "_score": 1.287682,
        "_source": {
          "field3": [
            {
              "name": "tohero1",
              "age": 1
            },
            {
              "name": "tohero2",
              "age": 2
            }
          ]
        }
      }
    ]
  }
}
```

**可以看到两条数据都被我们检索到了。**所以，现在理解什么叫做“**object不允许彼此独立地索引查询”**了吧。

但是，我们在日常的使用过程中，常规的需求就是，**希望object能被独立的索引，**难道es满足不了这个需求么？那是不可能。下面就来看下nested类型。



**3）nested 类型**

> **需要建立对象数组的索引并保持数组中每个对象的独立性，则应使用nested数据类型而不是 object数据类型。在内部，嵌套对象索引阵列作为一个单独的隐藏文档中的每个对象，这意味着每个嵌套的对象可以被独立的查询。**

备注：关于nested类型，TeHero在此就不写实例了，因为对于nested的应用本身属于ES的高级操作**，后面TeHero会单独出一期关于nested的使用教程。**

**对于复杂类型，目前先知道 object和nested类型的区别即可。**

**
**

**
**

***\*三、GEO 地理位置类型\****

> 对于 GEO 地理位置类型，分为 地图：Geo-point 和 形状 ：Geo-shape，两种数据类型。

> 对于web开发，一般常用的是 地图类型 Geo-point。

> 要求：先知道如何定义，如何查询即可

```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text": "Geo-point as an object",
  "location": {
    "lat": 41.12,
    "lon": -71.34
  }
}

PUT my_index/_doc/2
{
  "text": "Geo-point as a string",
  "location": "41.12,-71.34"
}

PUT my_index/_doc/3
{
  "text": "Geo-point as a geohash",
  "location": "drm3btev3e86"
}

PUT my_index/_doc/4
{
  "text": "Geo-point as an array",
  "location": [ -71.34, 41.12 ]
}
```

距离查询：距离某个点方圆200km

```
GET /my_locations/_search
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_distance" : {
                    "distance" : "200km",
                    "pin.location" : {
                        "lat" : 40,
                        "lon" : -70
                    }
                }
            }
        }
    }
}
```

> 到此，ES的数据类型就讲解完了。里面的DSL语句如果看不懂，没关系！通过本文章的学习，你知道了ES有哪些数据类型，这3个重点类型的注意点、使用区别和场景，就足够了。
>
> 下期预告：ES索引和文档的 CRUD 操作