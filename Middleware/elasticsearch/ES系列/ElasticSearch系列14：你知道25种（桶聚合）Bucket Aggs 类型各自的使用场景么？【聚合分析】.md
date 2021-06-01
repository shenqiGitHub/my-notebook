 *看了本文，你将掌握*





1、ES有哪些聚合类型？Bucket、Metric、Pipeline Aggregations 各自的特点是什么？？

2、Bucket Aggs 有哪些种类？各自的使用场景是什么？

3、Bucket Aggs 各种类型的重要参数有哪些？注意事项是什么？

**ps：本文基于****ES 7.7.1****【文末附\**《Bucket aggs 25种类型详解》xmind 获取方式\**】**





**01 ES聚合类型简介**



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3YOhFzUrtmsZ1JoSgH5JufKibznF3gQOcnJtfEbyjYcLAI6Gb4f7XdGQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一图胜千言



如上图，**E****S的聚合一共有4种类型，Bucket 、Metric、Pipeline 是经常使用的，掌握了这3种聚合**，就已经可以满足日常大部分的聚合分析场景了。



在学习之前，先掌握aggregations的语法结构：**【注意aggregations关键字可使用aggs代替】**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



简单示例，学会agg语法：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /cars/_search{  "size": 0,  "aggs": {    "first_agg_name": {      "terms": {        "field": "color"      },      "aggs": {        "sub_agg_name1": {          "avg": {            "field": "price"          }        },        "sub_agg_name2": {          "terms": {            "field": "make"          }        }      }    }  }}
```







**02 Bucket Aggregations**



Bucket 就是桶的意思，**即按照一定的规则将文档分配到不同的桶中，达到分类分析的目的。**



ES从 2.x 到 7.x，聚合功能已经日渐强大，到 7.7 版本， Bucket 聚合已经有**25种类型**了，今天我们就一起系统学习 **Bucket Aggregations，**全面掌握 Bucket 聚合**。
**

**
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF32Tp15uib1zkuANpg317KjRcAEBYTRsp6ztNLTRRWZPchHkZTwZkOsLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Bucket Aggs 概览





ps：因为篇幅问题，TeHero在文章中就只通过示例进行简单讲解，涉及其他的注意事项，重要参数等，见xmind截图，毕竟一图胜千言，哈哈，好吧，我承认，就是懒得写重复的内容【**文末有xmind源文件获取方式**】。





**2.0 写在前面**

通过上图《Bucket Aggs 概览》我们可以看到，一共有25种类型的 Bucket Aggs，**对于每一种聚合类型，我们都去详细学习并掌握是比较费时间的，个人建议可以按如下方式学习：**

- 1）了解每种聚合类型的使用场景，简单而言，就是**知道每种聚合是干嘛的，能对数据做怎样的分析；**
- 2）了解其注意事项和重要参数；
- 3）完成以上2点，我觉得就差不多了，在实际工作中，**面对需求，我们知道可以用哪些聚合操作解决需求即可**，需要用到的时候再去详细学习具体的语法。





**2.1 Terms 术语聚合**



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3PhelwB3DlGcHicXUwUKBJNnYOBvG7uPwbOn5585UuEApoTutDErx1Kg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**场景示例：**对于博客系统，按不同的作者分类聚合，得到每位作者的博文总数

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /blogs_index/_search{  "size": 0,  "aggs": {    "author": {      "terms": {        "field": "author"      }    }  }}
```

**结果：**

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  "aggregations" : {    "author" : {      "doc_count_error_upper_bound" : 0,      "sum_other_doc_count" : 0,      "buckets" : [        {          "key" : "方才兄",          "doc_count" : 3        },        {          "key" : "方才",          "doc_count" : 1        }      ]    }  }
```





 **2.2 Rare Terms 稀有术语聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



在 Terms Aggs 中，**聚合结果的排序是默认根据 doc_count 的值降序排列**，但在实际使用过程中，我们有时候希望**根据 doc_count 的值升序排列**，这个时候就应该使用 **Rare Terms【之所以不使用 Terms aggs再去改变排序规则，是因为聚合精度问题，后续专门讨论】**



**场景示例：**按不同的作者分类聚合，同时根据每位作者的文章总数进行升序排列

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /blogs_index/_search{  "size": 0,  "aggs": {    "author": {      "rare_terms": {        "field": "author",        "max_doc_count": 10      }    }  }}
```

**注意max_doc_count参数**：术语出现的最大文档数【**返回的bucket 的 doc_count <= 该值**】，默认值为1，最大值为100。



结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
   "aggregations" : {    "author" : {      "buckets" : [        {          "key" : "方才",          "doc_count" : 1        },        {          "key" : "方才兄",          "doc_count" : 3        }      ]    }  }
```





**2.3 Histogram 直方图聚合**

**
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3jl2MInjrY8tKgkjapHxYOeG1JfSs9D3S8tuqSvo93uQicViaEQKHhIeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**场景示例：**按商品价格区间聚合，得到不同价格区间的商品总数

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /product/_search{  "size": 0,  "aggs": {    "price": {      "histogram": {        "field": "price",        "interval": 2000      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
{  "aggregations": {    "price": {      "buckets": [        {          "key": 0,          "doc_count": 3        },        {          "key": 20000,          "doc_count": 4        },        {          "key": 80000,          "doc_count": 1        }      ]    }  }}
```

简单解释下，**返回的 “key” 值**：0代表区间【0,2000），2000代表区间【2000,4000）。





**2.4 Date histogram 日期直方图聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



**场景示例：**查看每天博客系统的发文总数

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /blogs_index/_search{  "size": 0,  "aggs": {    "price": {      "date_histogram": {        "field": "createAt",        "calendar_interval": "day",        "format": "yyyy-MM-dd"      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  "aggregations": {    "price": {      "buckets": [        {          "key_as_string": "2020-05-22",          "key": 1590105600000,          "doc_count": 1        },        {          "key_as_string": "2020-05-23",          "key": 1590192000000,          "doc_count": 1        },        {          "key_as_string": "2020-05-24",          "key": 1590278400000,          "doc_count": 2        }      ]    }  }
```





**2.5 Auto-interval Date Histogram 自动间隔日期直方图聚合**



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3Q8OVSOVyHsiaDDib4VbOrGXkxu6Iibm1mF0aruSRzQCFeYAOAWibNnaYwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



该聚合的应用场景，更多的可能是，**页面强制需要多个点绘制图表。**



**场景示例：**还是通过博客的创建时间做聚合，这次我希望返回3个bucket

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /blogs_index/_search{  "size": 0,  "aggs": {    "createTime": {      "auto_date_histogram": {        "field": "createAt",        "format": "yyyy-MM-dd",        "buckets": 3      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  "aggregations": {    "createTime": {      "buckets": [        {          "key_as_string": "2020-05-22",          "key": 1590105600000,          "doc_count": 1        },        {          "key_as_string": "2020-05-23",          "key": 1590192000000,          "doc_count": 1        },        {          "key_as_string": "2020-05-24",          "key": 1590278400000,          "doc_count": 2        }      ],      "interval": "1d"    }  }
```





**2.6 Range 范围聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



**场景示例：**查看价格在100以内，100-200和200以上 这3个范围的商品数量

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /product/_search{  "aggs": {    "price_ranges": {      "range": {        "field": "price",        "ranges": [          {            "to": 100          },          {            "from": 100,            "to": 200          },          {            "from": 200          }        ]      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  "aggregations": {    "price_ranges": {      "buckets": [        {          "key": "*-100.0",          "to": 100,          "doc_count": 0        },        {          "key": "100.0-200.0",          "from": 100,          "to": 200,          "doc_count": 0        },        {          "key": "200.0-*",          "from": 200,          "doc_count": 0        }      ]    }  }
```





**2.7 Date Range 日期范围聚合**

**
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3nHYJPE7wRfYwffwN6O8sibtBqMwNYcJe2ubqJzt1KfLL4efZnibLhJbA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**场景示例：**获取过去到10个月之前的所有商品总数和10个月之前的商品总数：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /product/_search{  "aggs": {    "range": {      "date_range": {        "field": "date",        "format": "yyyy-MM",        "ranges": [          {            "to": "now-10M/M"          },          {            "from": "now-10M/M"          }        ]      }    }  }}
```

**注意：to：date < 现在减去10个月，向下舍入到月初；from：date > = 现在减去10个月，向下舍入到月初。**



结果：现在是 2020-07

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  "aggregations": {    "range": {      "buckets": [        {          "key": "*-2019-09",          "to": 1567296000000,          "to_as_string": "2019-09",          "doc_count": 20        },        {          "key": "2019-09-*",          "from": 1567296000000,          "from_as_string": "2019-09",          "doc_count": 50        }      ]    }  }
```





**2.8 IP Range IP范围聚合**

**
**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



看个示例即可：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /ip_addresses/_search{  "size": 10,  "aggs": {    "ip_ranges": {      "ip_range": {        "field": "ip",        "ranges": [          {            "to": "10.0.0.5"          },          {            "from": "10.0.0.5"          }        ]      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /product/_search{  "aggregations": {    "ip_ranges": {      "buckets": [        {          "key": "*-10.0.0.5",          "to": "10.0.0.5",          "doc_count": 10        },        {          "key": "10.0.0.5-*",          "from": "10.0.0.5",          "doc_count": 0        }      ]    }  }}
```





**2.9 Composite 复合聚合**

**
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3ic0q2JSOxmlSA78AqkmricC9R8h1yQU2WqOTZZ3hqyeG2zVHNqrJW01w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



1）对于**Composite Agg 需要看两个示例，一个是翻页的示例：**

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /teamwork_task/_search{  "size": 0,  "aggs": {    "my_buckets": {      "composite": {        "size": 2,        "sources": [          {            "customName": {              "terms": {                "field": "team_id"              }            }          }        ]      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  "aggregations": {    "my_buckets": {      "after_key": {        "customName": 135      },      "buckets": [        {          "key": {            "customName": 128          },          "doc_count": 2        },        {          "key": {            "customName": 135          },          "doc_count": 3        }      ]    }  }}
```

翻页查询：**查询 body不变，添加上次返回的 after即可**

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /teamwork_task/_search{  "size": 0,  "aggs": {    "my_buckets": {      "composite": {        "size": 2,        "sources": [          {            "customName": {              "terms": {                "field": "team_id"              }            }          }        ],        "after": {          "customName": 135        }      }    }  }}
```



2）**问卷结果统计示例：**假设有A、B两道题，每题都有2个答案，那么 Composite聚合就可以**得到所有可能组合**的答案的问卷数

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET /question_index/_search{  "size": 0,  "aggs": {    "my_buckets": {      "composite": {        "sources": [          {            "question_A": {              "terms": {                "field": "question_A"              }            }          },          {            "question_B": {              "terms": {                "field": "question_B"              }            }          }        ]      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  "aggregations": {    "my_buckets": {      "after_key": {        "question_A": "B",        "question_B": 1      },      "buckets": [        {          "key": {            "question_A": "A",            "question_B": 1          },          "doc_count": 1        },        {          "key": {            "question_A": "B",            "question_B": 1          },          "doc_count": 1        }      ]    }  }
```





**2.10 Filter 过滤器聚合**

**
**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



**场景示例：**只想查看商品类型是 t-shirt 的平均价格

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
POST /sales/_search?size=0{    "aggs" : {        "t_shirts" : {            "filter" : { "term": { "type": "t-shirt" } },            "aggs" : {                "avg_price" : { "avg" : { "field" : "price" } }            }        }    }}
```



**ps：考虑篇幅问题，后面的例子仅写DSL，省略结果。**





**2.11 Filters 过滤器集合聚合**

**
**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



**场景示例：**日志统计，统计 error 和 warning 各有多少条记录

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
GET logs/_search{  "size": 0,  "aggs": {    "messages": {      "filters": {        "filters": {          "errors": {            "match": {              "body": "error"            }          },          "warnings": {            "match": {              "body": "warning"            }          }        }      }    }  }}
```





**2.12 Global 全局聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



**场景示例：**查询商品类型为 t-shirt 的商品集合及其平均价格，同时得到所有商品的平均价格

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
POST /sales/_search?size=0{  "query": {    "match": {      "type": "t-shirt"    }  },  "aggs": {    "all_products": {      "global": {},      "aggs": {        "avg_price": {          "avg": {            "field": "price"          }        }      }    },    "t_shirts": {      "avg": {        "field": "price"      }    }  }}
```





**2.13 Missing 缺少聚合**

**
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3lNZOskbRW2DR3VDBsSFYnU9xpHtDyx1Jcntl9aj97nWbOM950t1wlw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**场景示例：**获取没有标价的商品的总数

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
POST /sales/_search?size=0{  "aggs": {    "products_without_a_price": {      "missing": {        "field": "price"      }    }  }}
```





**2.14 Adjacency Matrix 邻接矩阵聚合**

**
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3Vq1CDoxQL6bkONJLyhSjo3MVoQAYdwoOVP9KRrk742r6dyfTwRg5ibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**概念理解图示：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3vSzlqu5PaUtmRPfABBzkj03ej3nwyaUfvAujxFdOb5iasQhWwIEybLw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





**2.15 3种地理位置的bucket聚合**



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF3QTFjcUnATo9qDmU0Q6srfexRPcrican1ia0q62GKXWdgiazspGUKoZkUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





 **2.16 后续8种bucket聚合待深入学习**



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowIlaV7DUib9AdqrdwHCyjnF33icN6UalaPYqWswFUx6NRlDegljcQmibbxBIz6o9J3bqTCURMlNAiawpA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)