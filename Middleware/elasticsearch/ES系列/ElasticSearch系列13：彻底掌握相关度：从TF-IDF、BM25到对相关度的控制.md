ES 5.0 之前，默认的相关性算分采用的是 TF-IDF，而之后则默认采用 BM25。



1、什么是相关性/相关度？Lucene 是如何计算相关度的？

2、TF-IDF 和 BM25 究竟是什么？

3、相关度控制的方式有哪些？各自都有什么特点？



**本文从相关性概念入手，到 TF-IDF 和 BM25 讲解和数学公式学习，再到详细介绍多种常用的相关度控制方式。相信对你一定有用！**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/13-1.png)

本文知识导航



ps：xmind源文件获取方式，见文末。







**01 什么是相关性**

相关性描述的是**⼀个⽂档和查询语句匹配的程度**。ES 会对每个匹配查询条件的结果进⾏算分\_score。\_score 的评分越高，相关度越高。



对于信息检索工具，衡量其性能有3大指标：

> 1）**查准率 Precision**：尽可能返回较少的无关文档；
>
> 2）**查全率 Recall**：尽可能返回较多的相关文档；
>
> 3）**排序 Ranking**：是否能按相关性排序。

前两者更多与分词匹配相关，而后者则与相关性的判断与算分相关。【本文将详细介绍相关性系列知识点，分词部分后续TeHero会单独讲解！】



**02 TF-IDF 和 BM25 是什么**



**2.1 词频 TF（Term Frequency）**

**检索词在文档中出现的频度是多少？出现频率越高，相关性也越高。**



关于TF的数学表达式，参考ES官网，如下：

> tf(t in d) = √frequency  
>
> 词 t 在文档 d 的词频（ tf ）是该词在文档中出现次数的平方根。

**概念理解**：比如说我们检索关键字“es”，“es”在文档A中出现了10次，在文档B中只出现了1次。我们不会认为文档B与“es”的相关性更高，而是文档A。





**2.2 逆向⽂档频率 IDF（Inverse Document Frequency）**



**每个检索词在索引中出现的频率，频率越高，相关性越低。**



关于 IDF 的数学表达式，参考ES官网，如下：

> idf(t) = 1 + log ( numDocs / (docFreq + 1))  
>
> 词 t 的逆向文档频率（ idf ）是：索引中文档数量除以所有包含该词的文档数，然后求其对数。
>
> 注意: 这里的log是**指以e为底的对数,不是以10为底的对数。**

**概念理解：**比如说检索词“学习ES”，按照Ik分词会得到两个Token【学习】【ES】，假设在当前索引下有100个文档包含Token“学习”，只有10个文档包含Token“ES”。那么对于【学习】【ES】这两个Token来说，**出现次数较少的 Token【ES】就可以帮助我们快速缩小范围找到我们想要的文档，**所以说此时“ES”的权重就比“学习”的权重要高。





**2.3 字段长度准则 field-length norm**



**字段的长度是多少？字段越短，字段的权重\*越高\*。**检索词出现在一个内容短的 title 要比同样的词出现在一个内容长的 content 字段权重更大。



关于 norm 的数学表达式，参考ES官网，如下：

> norm(d) = 1 / √numTerms  
>
> 字段长度归一值（ norm ）是字段中词数平方根的倒数。



以上三个因素——词频（term frequency）、逆向文档频率（inverse document frequency）和字段长度归一值（field-length norm）——**是在索引时计算并存储的。最后将它们结合在一起计算单个词在特定文档中的权重。**







**2.4 Lucene 中的 TF-IDF 评分公式**

**该公式参考自官网：**

```
score(q,d)  =
            queryNorm(q)
          · coord(q,d)
          · ∑ (
                tf(t in d)
              · idf(t)²
              · t.getBoost()
              · norm(t,d)
            ) (t in q)
```

> **score(q,d)**  文档d对查询q的相关性得分
>
> 
>
> **queryNorm(q)**  查询的规范化因子
>
> 
>
> **coord(q,d)**  协调因子
>
> 
>
> **∑** 文档d的查询q中每个词t的权重之和
>
> 
>
> **tf(t in d)**  文档d中t词的词频(出现次数)
>
> 
>
> **idf(t)**  t词的逆文档频率
>
> 
>
> **t.getBoost()** 已应用于查询的boost
>
> 
>
> **norm(t,d)**  是字段长度归一值，与检索时字段的Boost （如果存在）相结合。

**虽然现在es的相关性评分算法改为了BM25，但对于该公式，我们还是应该掌握，这有利于我们理解后续对相关度的控制。**



**2.5 BM25**



整体而言 **BM25 就是对 TF-IDF 算法的改进**，对于 TF-IDF 算法，**TF(t) 部分的值越大，整个公式返回的值就会越大。**



BM25 就针对这点进行来优化，**随着TF(t) 的逐步加大，该算法的返回值会趋于一个数值。**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/13-2.png)

该图来自ES官网

 BM25 有一个比较好的特性就是提供了**两个可调参数：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowJwOpr1WP1pWq96aPEOd5pUY2lsOYAuT6SMgJZqM00wwWvWmjZlN6TJtQoN7flbuictTRHPaAaCdnA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

该公式来源于阮一鸣老师的课程

> k1：这个参数控制着词频结果在**词频饱和度中的上升速度。**默认值为1.2。值越小饱和度变化越快，值越大饱和度变化越慢。
>
> 
>
> b：这个参数**控制着字段长归一值所起的作用，**0.0会禁用归一化，1.0会启用完全归一化。默认值为0.75。

该公式"."的前部分就是 IDF 的算法，后部分就是 TF+Norm 的算法。







**03 explain**



**使用 explain 查看搜索相关性分数的计算过程。**这非常有助于我们理解ES的相关度计算过程。下面通过示例来学习：



**1）导入测试数据**

```
#创建index
PUT /blogs_index
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    }
  },
  "mappings": {
    "_doc": {
      "dynamic": false,
      "properties": {
        "id": {
          "type": "integer"
        },
        "author": {
          "type": "keyword"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_smart"
        },
        "content": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_smart"
        },
        "tag": {
          "type": "keyword"
        },
        "influence": {
          "type": "integer_range"
        },
        "createAt": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm"
        }
      }
    }
  }
}

# 导入数据
POST _bulk
{"index":{"_index":"blogs_index","_type":"_doc","_id":"1"}}
{"id":1,"author":"方才兄","title":"es的相关度","content":"这是关于es的相关度的文章","tag":[1,2,3],"influence":{"gte":10,"lte":12},"createAt":"2020-05-24 10:56"}
{"index":{"_index":"blogs_index","_type":"_doc","_id":"2"}}
{"id":2,"author":"方才兄","title":"相关度","content":"这是关于相关度的文章","tag":[2,3,4],"influence":{"gte":12,"lte":15},"createAt":"2020-05-23 10:56"}
{"index":{"_index":"blogs_index","_type":"_doc","_id":"3"}}
{"id":3,"author":"方才兄","title":"es","content":"这是关于关于es和编程的必看文章","tag":[2,3,4],"influence":{"gte":12,"lte":15},"createAt":"2020-05-22 10:56"}
{"index":{"_index":"blogs_index","_type":"_doc","_id":"4"}}
{"id":4,"author":"方才","title":"关注我，系统学习es","content":"这是关于es的文章，介绍了一点相关度的知识","tag":[1,2,3],"influence":{"gte":10,"lte":15},"createAt":"2020-05-24 10:56"}
```



**2）使用explain**

```
GET /blogs_index/_search
{
  "query": {
    "match": {
      "title": "es的相关度"
    }
  },
  "explain": true
}
```

**3）结果与分析**

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

    "total": 4,

    "max_score": 2.5933092,

    "hits": [

      {

        "_shard": "[blogs_index][0]",

        "_node": "VAAU48LMQ_iQfscqT2gjLA",

        "_index": "blogs_index",

        "_type": "_doc",

        "_id": "1",

        "_score": 2.5933092,

        "_source": {

          "id": 1,

          "author": "方才兄",

          "title": "es的相关度",

          "content": "这是关于es的相关度的文章",

          "tag": [

            1,

            2,

            3

          ],

          "influence": {

            "gte": 10,

            "lte": 12

          },

          "createAt": "2020-05-24 10:56"

        },

        "_explanation": {
          "value": 2.593309,
          "description": "sum of:",
          "details": [
            {
              "value": 0.31387395,
             "description": "weight(title:es in 0) [PerFieldSimilarity], result of:",
              "details": [
                {
                  "value": 0.31387395,
                  "description": "score(doc=0,freq=1.0 = termFreq=1.0\n), product of:",
                  "details": [
                    {
                      "value": 0.35667494,
                      "description": "idf, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:",
                      "details": [
                        {
                          "value": 3,
                          "description": "docFreq",
                          "details": []
                        },
                        {
                          "value": 4,
                          "description": "docCount",
                          "details": []
                        }
                      ]
                    },
                    {
                      "value": 0.88,
                      "description": "tfNorm, computed as (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * fieldLength / avgFieldLength)) from:",
                      "details": [
                        {
                          "value": 1,
                          "description": "termFreq=1.0",
                          "details": []
                        },
                        {
                          "value": 1.2,
                          "description": "parameter k1",
                          "details": []
                        },
                        {
                          "value": 0.75,
                          "description": "parameter b",
                          "details": []
                        },
                        {
                          "value": 3,
                          "description": "avgFieldLength",
                          "details": []
                        },
                        {
                          "value": 4,
                          "description": "fieldLength",
                          "details": []
                        }
                      ]
                    }
                  ]
                }
              ]
            },
            {
              "value": 1.059496,
              "description": "weight(title:的 in 0) [PerFieldSimilarity], result of:",
              "details": [
……………………………………………………
```



我们简单分析下文档1的相关性算分过程，去理解ES的相关性算分：

> 1）"description": "**idf**, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:"，根据该公式，docCount  = 4，docFreq = 3，**计算出 value = log10/7 = ln 10/7 = 0.3566749440**
>
> **2）**"description": "**tfNorm**, computed as (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * fieldLength / avgFieldLength)) from:", 根据details的信息，计算出 value = （1*（1.2+1）/（1+1.2 *（1-0.75+0.75*4/3）））=  0.88
> 
>
> 
>3）**BM25（es）= idf \* tfNorm = 0.3566749440 \* 0.88 = 0.3138739947**
> 
>
> 
>4）同理得到 BM25（的）= 1.059496，BM25（相关）= 0.6099695，BM25（度）= 0.6099695；
> 
>
> 
>5）根据"description": "**sum** of:",当检索【es的相关度】，**文档1的_score = BM（es）+ BM25（的）+ BM25（相关）+ BM25（度）**= 2.5933092



上述算分过程，建议自己使用 explain 实践一波。毕竟纸上得来终觉浅！









**04 相关度控制**



通过上面的学习，我们已经知道了什么是TF-IDF，什么是BM25，同时通过explain大致了解了ES的相关性算分过程。



**那么如果ES默认的相关性算分不符合我们的使用需求，我们可以通过哪些方式去改变或控制相关度评分呢？**



TeHero通过官网和网课的学习，并结合自身实践**，目前总结了以下4种常用的相关度控制方式，供大家参考：**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/13-3.png)



**4.1 boost  参数【常用】**



我们检索博客时，我们一般会认为标题 title 的权重应该比内容 content 的权重大，那么这个时候我们就可以使用 boost 参数进行控制：

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": {
              "query": "es",
              "boost": 2
            }
          }
        },
        {
          "match": {
            "content": "es"
          }
        }
      ]
    }
  },
  "explain": true
}
```

通过 explain 查看下算分过程：

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/13-4.png)

> 根据结果，我们可以看到：对应文档的_score = BM25（es in title） + BM25（es in content）；其中**BM25（es in title）= boost * idf  *  tfNorm**

**boost 参数值范围：**

> boost>1 相关度相对性提升
>
> 
>
> 0<boost<1，相对性降低
>
> 
>
> boost<0，贡献负分



**注意：**1）boost 可用于任何查询语句；**2）这种提升或降低并不一定是线性的**，新的评分 _score 会在应用权重提升之后被归一化 ，每种类型的查询都有自己的归一算法。





**4.2 查询方式改变**



**我们也可以通过使用不同的组合查询来实现对相关度的控制，**关于组合查询，TeHero之前就只讲解了布尔查询【Bool Query[ES系列12：Compound queries 之  Bool query](http://mp.weixin.qq.com/s?__biz=MzIxMjE3NjYwOQ==&mid=2247483976&idx=1&sn=f9fc58f7f38ef79d4a652a9578ce1181&chksm=974b59c6a03cd0d036f9e1cc9d211b999c9d3acdd664f4a250a1573089fdfe747c7784191066&scene=21#wechat_redirect)】，**是因为剩余的4种组合查询涉及到相关度。**



今天，我们先简单了解下**剩余的4种组合查询，具体深入的使用，TeHero后面会结合实践详细和大家一起交流分享。**



1）constant_score

嵌套一个 filter 查询，**为任意一个匹配的文档<font color=#FF0000>指定一个常量评分</font>，常量值<span style='color:red;background:背景颜色;font-size:文字大小;'>为 boost 的参数值</span>【默认值为1】 ，忽略 TF-IDF 信息。**

```
GET /blogs_index/_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "title": "es"}
            },
            "boost" : 1.2
        }
    }
}
```

结果展示：

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/13-5.png)



**2）function_score query**

**Function Score Query 允许我们修改通过 query 检索出来的文档的分数。**

在使用时，我们必须**定义一个查询**和**一个或多个函数**，这些函数为查询返回的每个文档**计算一个新分数。**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/13-6.png)

从上图就可以看到，Function Score Query 涉及的参数很多。此处先简单看个DSL示例，**具体的分析后续专门讲解：**

```
GET /blogs_index/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_all": {}
      },
      "boost": "5",
      "functions": [
        {
          "filter": {
            "match": {
              "title": "es"
            }
          },
          "random_score": {},
          "weight": 23
        },
        {
          "filter": {
            "match": {
              "title": "相关度"
            }
          },
          "weight": 42
        }
      ],
      "max_boost": 42,
      "score_mode": "max",
      "boost_mode": "multiply",
      "min_score": 10
    }
  },
  "explain": true
}
```

备注：**function_score query** **的用法非常多，适用场景也比较广，比如说：1）通过文档中的字段值影响相关度，比如可以让博客的点赞数越多，相关度越高；2）随机分数【可应用于千人千面】；3）根据距离参考值的衰减函数计算相关度，比如说地理位置查询，距离参考点越远的，相关性越低；4）更复杂的场景也可以用自定义脚本完全控制评分计算，实现所需逻辑。**

**
**

**关于对** **function_score query** **的详细讲解，TeHero后续会和大家分享的。**

***\*
\****



**3）dis_max query**

dis_max query **使用单个最佳匹配查询子句的分数**。同时，也可以通过参数 tie_breaker 【默认值为0】 **控制其他查询子句的分数对 _score 的影响**。



相关性得分计算公式：**_score = max(BM25) + ∑ other(BM25)\*tie_breaker**

```
GET /blogs_index/_search
{
  "query": {
    "dis_max": {
      "tie_breaker": 0.5,
      "boost": 1.2,
      "queries": [
        {
          "term": {
            "content": "es"
          }
        },
        {
          "match": {
            "content": "相关度"
          }
        }
      ]
    }
  },
  "explain": true
}
```

> 注意：**queries 下的查询子句间的布尔关系是OR。**

dis_max query 有一个非常好的使用场景就是，利用参数 tie_breaker 能够**确保满足多个条件的文档的相关性得分一定比只满足单个条件的文档的得分要高。**





**4）boosting query**【常用】

boosting query 可用于**有效降级与给定查询匹配的结果**。与布尔查询中的“ NOT”子句不同的是，**它仍会选择包含不良词的文档，但会降低其总体得分。**



**参数解释：**

> positive：用于获取返回结果
>
> 
>
> negative：对上述结果的相关性打分进行调整
>
> 
>
> negative_boost：调整参数：升权(>1), 降权(>0 and <1)

来看一个DSL示例，我们希望检索title 包含“es”“相关性”的文章，同时认为如果content包含“编程”，那我们认为这个文档的相关性应该被降低：

```
GET /blogs_index/_search
{
  "query": {
    "boosting": {
      "positive": {
        "bool": {
          "should": [
            {
              "term": {
                "title": "es"
              }
            },
            {
              "term": {
                "title": "相关性"
              }
            }
          ]
        }
      },
      "negative": {
        "term": {
          "content": "编程"
        }
      },
      "negative_boost": 0.2
    }
  },
  "explain": true
}
```

DSL分析：

> 1）根据 positive 下的查询语句检索，得到结果集；
>
> 
>
> 2）**在上述的结果集中**，对于那些同时还匹配 negative 查询的文档，**将通过文档的原始 _score 与 negative_boost 相乘的方式重新计算相关性得分。**

注意：

> **negative_boost 的值>1，是正向评分，增加匹配 negative 查询的文档的权重。**







**4.3 rescore 结果集重新评分**

**
**

**先query，再在结果集基础上 rescore。query 目前唯一支持的重新打分算法。参数** **window_size 是每一分片进行重新评分的顶部文档数量。**

**
**

看个示例，先检索content包含“es的相关度”或者 title 包含“es”的文档，再此基础上，对于前3个文档，利用match_phrase 重新计算相关度。

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "content": {
              "query": "es的相关度",
              "minimum_should_match": "30%"
            }
          }
        },
        {
          "match": {
            "title": {
              "query": "es"
            }
          }
        }
      ]
    }
  },
  "rescore": {
    "window_size": 3,
    "query": {
      "rescore_query": {
        "match_phrase": {
          "content": {
            "query": "es的相关度",
            "slop": 50
          }
        }
      }
    }
  }
}
```

rescore 和 上面的 Boosting Query 是比较相似的，都是在 query 结果集的基础上重新修改相关性得分。但是修改的算法是不一样的，根据场景需求，选择即可。



**同时 rescore  可以利用 window_size 参数控制重新计算得分的文档数量，在数据量较大的情况，适当控制 window_size 参数，性能上会比 Boosting Query好。**

**
**



**4.4 更改BM25 参数 k1 和 b 的值**

**
**

在介绍BM25算法时，我们知道 k1 参数【默认值1.2】控制着词频结果在词频饱和度中的上升速度。b 参数【默认值0.75】控制着字段长归一值所起的作用。



那么我们就可以通过**手动定义这两个参数的值**，**从而去改变相关性算分。**

**
**

**只能在创建index的时候定义字段的similarity** ，在后续，可以通过关闭索引，更新索引设置，开启索引这个过程进行更新 my_bm25 的 参数值。这样可以无须重建索引又能试验不同的相似度算法配置。

```
PUT /my_index
{
  "settings": {
    "similarity": {
      "my_bm25": {
        "type": "BM25",
        "b": 0.8,
        "k1": 1.5
      }
    }
  },
  "mappings": {
    "doc": {
      "properties": {
        "title": {
          "type": "text",
          "similarity": "my_bm25"
        }
      }
    }
  }
}
```

注意：一般情况不建议更改这两个参数值。





05 被破坏的相关度



每个分片都会根据该分片内的所有文档计算一个**本地 IDF**。这会导致打分偏离，特别是数据量很少时。



相关性算分的IDF 在分⽚之间是相互独⽴。当⽂档总数很少的情况下，如果主分⽚⼤于 1，主分⽚数越多 ，相关性算分会越不准。



**ps：了解该现象，主要是为了解决很多小伙伴在做测试时的疑惑。****简单浏览即可。**



**5.1 现象示例：**

```
PUT /blogs_index1
{
  "settings": {
    "index": {
      "number_of_shards": 10,
      "number_of_replicas": 0
    }
  },
  "mappings": {
    "_doc": {
      "dynamic": false,
      "properties": {
        "id": {
          "type": "integer"
        },
        "author": {
          "type": "keyword"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_smart"
        },
        "content": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_smart"
        },
        "tag": {
          "type": "keyword"
        },
        "influence": {
          "type": "integer_range"
        },
        "createAt": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm"
        }
      }
    }
  }
}
POST _bulk
{"index":{"_index":"blogs_index1","_type":"_doc","_id":"1"}}
{"id":1,"author":"方才兄","title":"es的相关度","content":"这是关于es的相关度的文章","tag":[1,2,3],"influence":{"gte":10,"lte":12},"createAt":"2020-05-24 10:56"}
{"index":{"_index":"blogs_index1","_type":"_doc","_id":"2"}}
{"id":2,"author":"方才兄","title":"相关度","content":"这是关于相关度的文章","tag":[2,3,4],"influence":{"gte":12,"lte":15},"createAt":"2020-05-23 10:56"}
{"index":{"_index":"blogs_index1","_type":"_doc","_id":"3"}}
{"id":3,"author":"方才兄","title":"es","content":"这是关于关于es和编程的必看文章","tag":[2,3,4],"influence":{"gte":12,"lte":15},"createAt":"2020-05-22 10:56"}
{"index":{"_index":"blogs_index1","_type":"_doc","_id":"4"}}
{"id":4,"author":"方才","title":"关注我，系统学习es","content":"这是关于es的文章，介绍了一点相关度的知识","tag":[1,2,3],"influence":{"gte":10,"lte":15},"createAt":"2020-05-24 10:56"}
```

查询：

```
GET /blogs_index1/_search
{
  "_source": "title",
  "query": {
    "match": {
      "title": {
        "query": "es"
       
      }
    }
  }
}
```

结果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowJwOpr1WP1pWq96aPEOd5pUUtA4AqVx3W8CyLVCicLxctwk75ic7SHP2kX0icUJzkdZx5OF1Gj3tcLuQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

根据我们前面学的TF-IDF和BM25 算法，很明显，该结果违背了预期。





**5.2 两种方式解决**

**
**

1）当数据量不大时，将主分片数设置为1。**【学习过程建议设置为1】**

```
PUT /tehero_index
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    }
}
```

2）搜索的URL 中指定参数 “_search?search_type=dfs_query_then_fetch”

```
GET /blogs_index1/_search?search_type=dfs_query_then_fetch
{
  "query": {
    "match": {
      "title": {
        "query": "es"
       
      }
    }
  }
}
```

使用该参数查询时，es会到每个分⽚把各分⽚的词频和⽂档频率进⾏搜集，然后完整的进⾏⼀次相关性算分，耗费更加多的 CPU 和内存，执⾏性能低下，⼀般不建议使⽤。



**5.3 该现象不用深究**

**
**

在实际应用中，这并不是一个问题，本地和全局的 IDF 的差异会随着**索引里文档数的增多渐渐消失**，在真实世界的数据量下，局部的 IDF 会被迅速均化，所以上述问题并不是相关度被破坏所导致的**，而是由于数据太少。**







**06 相关度控制最后要做的事情**



1、**理解评分过程是非常重要的，**这样就可以根据具体的业务对评分结果进行调试、调节、减弱和定制。

**2、本文介绍的4种相关度控制方案，建议结合实践，根据自己的业务需求，多动手调试练习。**

3、*最相关* 这个概念是一个难以触及的模糊目标，通常不同人对文档排序又有着不同的想法，这很容易使人陷入持续反复调整而没有明显进展的怪圈。**强烈建议不要去追求最相关，而要监控测量搜索结果。**

4、**评价搜索结果与用户之间相关程度的指标。**如果查询能返回高相关的文档，用户会选择前五中的一个，得到想要的结果，然后离开。不相关的结果会让用户来回点击并尝试新的搜索条件。

5、要想物尽其用并将搜索结果提高到 *极高的* 水平，**唯一途径就是需要具备能评价度量用户行为的强大能力。**



**最后，如果你有更好的相关度控制方式，或者在es的学习过程中有疑问，****欢迎加入[es交流群](http://mp.weixin.qq.com/s?__biz=MzIxMjE3NjYwOQ==&mid=2247483875&idx=1&sn=89b29a1d9a508d02f6422cf73c98f1f4&chksm=974b5a6da03cd37bc924a15bd170f6f61fc45d96773f6cbe300e904b7977c12fda3caa95c0a1&scene=21#wechat_redirect)，和大家一起交流学习。**