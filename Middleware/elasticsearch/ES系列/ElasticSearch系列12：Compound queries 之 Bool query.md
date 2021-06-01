1、Bool query 的子句有哪些类型？

2、如何应用 Bool query？结合实际场景分析

3、minimum_should_match 参数如何配置？

ps：本文设计到的相关性评分，近期TeHero会专门讲解！[本文基于ES6.8]



![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/12-1.png)

本文知识导航图









01 查询和过滤上下文

在学习 Bool query  之前，我们应该先了解ES的两种上下文：



**1）Query context**

在查询上下文中，查询子句关注“ *此文档与该查询子句的**匹配程度如何**？*”，除了确定文档是否匹配之外，查询子句还计算_score元字段中的相关性得分 。



**2）Filter context**

在过滤器上下文中，查询子句关注“*此文档**是否与此查询子句匹配**？" ，* 答案很简单，是或否，不计算分数。过滤器上下文主要用于过滤结构化数据。



**常用过滤器将由Elasticsearch自动缓存，以提高性能。**



每当将查询子句传递到filter 参数（例如 bool查询中的filter或must_not参数，constant_score查询中的filter参数或filter聚合）时， 过滤器上下文即有效。







**02 Bool query  简介**

布尔查询映射到LuceneBooleanQuery。它是使用一个或多个布尔子句构建的，每个子句都有固定的类型。 **Bool query 的子句的类型有4种：**



**1）filter**

必须匹配，子句在过滤器上下文中执行，这意味着计分被忽略，并且子句被视为用于缓存。



**2）must**

子句（查询）必须出现在匹配的文档中，并将有助于得分。



**3）must_not**

子句（查询）不得出现在匹配的文档中。子句在过滤器上下文中执行，这意味着计分被忽略，并且子句被视为用于缓存。



**4）should**

子句（查询）应出现在匹配的文档中。【注意should的最小匹配数】



**5）Bool query 注意事项：**

- 1、Bool query 只支持以上4种查询的子句；



- 2、以上4种查询的子句，只支持 Full text queries 和 Term-level queries  和 Bool query ；【在学习boolQuery之前，建议先学习——[ES系列06：ik分词+Full text queries](http://mp.weixin.qq.com/s?__biz=MzIxMjE3NjYwOQ==&mid=2247483825&idx=1&sn=0b294bd614ed4504577b5d155706fd5f&chksm=974b5a3fa03cd329c9393f184299b430728118a7005e236cce049e1af9d31f8c75037b80a1f9&scene=21#wechat_redirect)和[ES系列09：Term-level queries](http://mp.weixin.qq.com/s?__biz=MzIxMjE3NjYwOQ==&mid=2247483903&idx=1&sn=caf90a146527927b01c329e371c39af7&chksm=974b5a71a03cd367fb64a99284bed011b0829bff6b9107362a58035eefdc60a8774ed229d07f&scene=21#wechat_redirect)】



- 3、简单而言就是：bool -》filter/must等-》bool -》filter/must等-》 queries 或者 bool -》filter/must等-》 queries ；



- 4、只有must 和 should 子句会计算相关性评分；filter 和 must_not 子句都是在过滤器上下文中执行，计分被忽略，并且子句被考虑用于缓存。









***\*03 通过实例学习\** Bool query**



**3.1 数据准备**

**1）创建index**

```
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
```

**2）导入数据**

```
POST _bulk{"index":{"_index":"blogs_index","_type":"_doc","_id":"1"}}{"id":1,"author":"方才兄","title":"关注我，系统学习es","content":"这是关于es的文章","tag":[1,2,3],"influence":{"gte":10,"lte":12},"createAt":"2020-05-24 10:56"}{"index":{"_index":"blogs_index","_type":"_doc","_id":"2"}}{"id":2,"author":"方才兄","title":"系统学习编程","content":"这是关于编程的文章","tag":[2,3,4],"influence":{"gte":12,"lte":15},"createAt":"2020-05-23 10:56"}{"index":{"_index":"blogs_index","_type":"_doc","_id":"3"}}{"id":3,"author":"方才兄","title":"关注我，必看文章","content":"这是关于关于es和编程的必看文章","tag":[2,3,4],"influence":{"gte":12,"lte":15},"createAt":"2020-05-22 10:56"}{"index":{"_index":"blogs_index","_type":"_doc","_id":"4"}}{"id":4,"author":"方才","title":"关注我，系统学习es","content":"这是关于es的文章","tag":[1,2,3],"influence":{"gte":10,"lte":15},"createAt":"2020-05-24 10:56"}{"index":{"_index":"blogs_index","_type":"_doc","_id":"5"}}{"id":5,"author":"方才","title":"系统学习编程","content":"这是关于编程的文章","tag":[2,3,4],"influence":{"gte":12,"lte":18},"createAt":"2020-05-25 10:56"}{"index":{"_index":"blogs_index","_type":"_doc","_id":"6"}}{"id":6,"author":"方才","title":"关注我，必看文章","content":"这是关于关于es和编程的必看文章","tag":[2,3,4],"influence":{"gte":15,"lte":18},"createAt":"2020-05-20 10:56"}
```





**3.2 场景与DSL**

**ps：为了更完整的学习boolQuery，以下内容，不分先后。**



**1）filter 的使用**

【语句1】：filter 子句内可包含多个 Full text queries 和 Term-level queries 的子句：

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "author": "方才兄"
          }
        },
        {
          "term": {
            "tag": "2"
          }
        },
        {
          "match": {
            "title": "es"
          }
        }
      ]
    }
  }
}
```

> **该DSL语句可以检索到文档1**，检索逻辑是：author = “方才兄” and tag 包含 2  and title的PostingLIst包含“es”。



【语句2】：filter 子句类可包含 bool query，实现更复杂的逻辑：

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "filter": {
        "bool": {
          "must": [
            {
              "term": {
                "author": "方才兄"
              }
            },
            {
              "term": {
                "tag": "2"
              }
            }
          ],
          "should": [
            {
              "match": {
                "title": "es"
              }
            },
            {
              "match": {
                "content": "es"
              }
            }
          ]
        }
      }
    }
  }
}
```

> **该DSL语句可以检索到文档1和文档3**，检索逻辑是：author = “方才兄” and tag 包含 2  and  （ title的PostingLIst包含“es”or  content的PostingLIst包含“es”）



**注意：使用filter查询，是不会计算文档的相关性评分的，可以看一下结果：**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/12-2.png)



**2）must 的使用**

【语句3】：直接将【语句1】中的 must 换为 filter

```
# 检索到文档1
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "author": "方才兄"
          }
        },
        {
          "term": {
            "tag": "2"
          }
        },
        {
          "match": {
            "title": "es"
          }
        }
      ]
    }
  }
}
```

结果如下，检索逻辑与【语句1】完全一致，**唯一的区别就是must会计算相关性评分！**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/12-3.png)



**3）must_not 的使用**

【语句4】：直接将【语句1】中的 must 换为 must_not，同时删除对tag的检索

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "term": {
            "author": "方才兄"
          }
        },
        {
          "match": {
            "title": "es"
          }
        }
      ]
    }
  }
}
```

> **该DSL可检索到文档5和文档6**，检索逻辑为：author != 方才兄 and title的postingList 不包含“es”。



同时，**must_not 会将相关性评分处理为常数1：**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/12-4.png)



**4）should 的使用**

【语句5】：直接将【语句4】中的 must_not 换为 should

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "author": "方才兄"
          }
        },
      
        {
          "match": {
            "title": "es"
          }
        }
      ]
    }
  }
}
```

> 该DSL会检索到文档1、2、3、4，检索逻辑是：author = 方才兄 or title的PostingList包含“es”。





**3.3 should 的注意事项**

**1）should 仅影响得分的情况**

如果 **bool查询在Query context中**并且 **bool查询具有must或 filter子句**，那么bool的 should查询**即使没有匹配到**，文档也将与查询匹配。在这种情况下，**should的子句仅用于影响得分。**

看个示例理解下：

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "must": {
        "bool": {
          "must": [
            {
              "term": {
                "author": "方才兄"
              }
            },
            {
              "term": {
                "tag": "2"
              }
            }
          ],
          "should": [
            {
              "match": {
                "title": "es"
              }
            },
            {
              "match": {
                "content": "es"
              }
            }
          ]
        }
      }
    }
  }
}
```

> 该DSL语句会检索到文档1、2、3。正常理解should语句至少满足一个条件而言，检索到文档1和文档3是没问题，**但是文档2，也被检索出来了，那就证明此时的should子句仅用于影响得分。**

看下文档2的数据：

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/12-5.png)



**2）should 至少匹配一个的情况**



如果bool 查询**是 Filter context** **或 既没有must也没filter**，则文档**至少与一个should的查询相匹配。【注意ES版本】**



第一个例子：**bool 查询****是 Filter context。**直接将上述的DSL的第一个must替换为filter即可：

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "filter": {
        "bool": {
          "must": [
            {
              "term": {
                "author": "方才兄"
              }
            },
            {
              "term": {
                "tag": "2"
              }
            }
          ],
          "should": [
            {
              "match": {
                "title": "es"
              }
            },
            {
              "match": {
                "content": "es"
              }
            }
          ]
        }
      }
    }
  }
}
```

> 上述DSL就只能检索文档1和3了。说明should 至少匹配了一个。【由于篇幅问题，结果就不贴出来】



第二个例子：**既没有must也没filter，should 至少匹配一个的情况，**可参考上面【语句5】，就不解释了。

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "author": "方才兄"
          }
        },
      
        {
          "match": {
            "title": "es"
          }
        }
      ]
    }
  }
}
```



**3）minimum_should_match 参数值说明**

**
**

如果不管哪种情况，我们都希望should子句应该被匹配，或者应该被匹配多个，那应该怎么做呢？**可以通过minimum_should_match 参数控制。**



看个示例理解下：

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "must": {
        "bool": {
          "must": [
            {
              "term": {
                "author": "方才兄"
              }
            }
          ],
          "should": [
            {
              "match": {
                "title": "es"
              }
            },
            {
              "match": {
                "content": "es"
              }
            },
            {
              "match": {
                "content": "编程"
              }
            }
          ],
          "minimum_should_match":2
        }
      }
    }
  }
}
```

> 该DSL可以检索到文档1和3，检索逻辑为：author = 方才兄 and  （title的PostingList 包含“es” 、content的PostingList 包含“es” 、content的PostingList 包含“编程”）**【这3个条件满足2个】**



**minimum_should_match 参数可以有以下形式的参数：**

> **标记：**N——**应该匹配的子句数，S——子句总数，X——用户给定的参数值**

**（1）正整数**



N = X，比如给定值为3，那么 N=3。



**（2）负整数**



**N = S - |X|，**比如给定值为-2，那么 N= S - 2。



**（3）正百分比**



**N = min 取整（S\*X），**比如 S=5，X = 25% ，那么 N= 1



**（4）负百分比**



**N =S - min 取整（|S\*X|），**比如 S=5，X = -25% ，那么 N= 5 - 1 = 4。



**（5）组合，比如 3<90%**



**当 S <= 3，则全部都是必需的；**当 S > 3**，则仅需要90％，按上面的正百分比计算。**



**（6）多种组合，比如：2<-25%  9<3**



多个条件规范可以用空格分隔。每个条件规范仅对大于其前一个的数字有效。



在此示例中：**如果有1或2个子句，则都需要；如果有3-9个子句，则需要-25％(按负百分数计算)；如果有9个以上的子句，则需要3个。**



**（7）注意：**



**当**minimum_should_match**的值\**大于\**子句数量数，DSL将检索不到值**；

当minimum_should_match**为0时，should子句失效。**





**3.4  一道Bool Query 练习题**



影响力 influence在范围12~20；文章标签tag包含3或者4，同时不能包含1；发布时间createAt一周内；标题title或内容content 包含“es”、“编程”、“必看”【3选2】且需要相关性评分。



参考答案如下：【ps：实现需求的DSL语句有多种可能，建议初学者自己练习下，以下仅供参考】

```
GET /blogs_index/_search
{
  "query": {
    "bool": {
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "influence": {
                  "gte": 12,
                  "lte": 20,
                  "relation": "WITHIN"
                }
              }
            },
            {
              "range": {
                "createAt": {
                  "gte": "now-1w/d"
                }
              }
            },
            {
              "terms": {
                "tag": [
                  3,
                  4
                ]
              }
            }
          ],
          "must_not": [
            {
              "term": {
                "tag": 1
              }
            }
          ]
        }
      },
      "should": [
        {
          "multi_match": {
            "query": "es",
            "fields": [
              "title",
              "content"
            ]
          }
        },
        {
          "multi_match": {
            "query": "编程",
            "fields": [
              "title",
              "content"
            ]
          }
        },
        {
          "multi_match": {
            "query": "必看",
            "fields": [
              "title",
              "content"
            ]
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
```