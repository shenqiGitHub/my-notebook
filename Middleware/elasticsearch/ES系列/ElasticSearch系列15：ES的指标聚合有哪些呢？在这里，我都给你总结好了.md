*本文目标*





通过本文，**你将系统掌握常用的指标聚合**，了解每种指标聚合的使用场景和语法。



ps：本文基于**ES 7.7.1**【文末附《指标聚合Metric Agg详解》xmind 获取方式】



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowJBLy6PQIZKJlia3ryHSqtlsiaOKTQ1B4FiblqPsm5cgrrpibeCRO53IDu2JxQ8jD7EdHaFrgwnPiaxUXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

本文知识导航



**ps：因为篇幅问题，TeHero在文章中就只通过示例进行简单讲解，涉及其他的注意事项，重要参数等，见xmind截图，毕竟一图胜千言！【\**文末有xmind源文件获取方式\**】。**

**
**

**
**

**01 写在前面**

**
**

通过上图我们可以看到，一共有18种类型的 Metrics Agg，而且这只是ES7.7.1，TeHero是在6月初安装的ES，**当时最新版就是7.7.1，但是半个月后，ES更新了！！！又！！\**又！！\**\**又！！\****新增了几种聚合方法。



**所以，对于每一种聚合类型，我们都去详细学习并掌握是比较费时间的，个人建议可以按如下方式学习：**

- 1）了解每种聚合类型的使用场景，简单而言，就是**知道每种聚合是干嘛的，能对数据做怎样的分析；**
- 2）掌握常用的聚合操作，了解其注意事项和重要参数；
- 3）完成以上2点，我觉得就差不多了，在实际工作中，**面对需求，我们知道可以用哪些聚合操作解决需求即可**，需要用到的时候再去详细学习具体的语法。







**02 Metrics Agg 详解**

**
**

Metrics Aggregations 指标分析类型，就是**一些数学运算，对文档字段进行统计分析，****类似于 sql 的 COUNT() 、 SUM() 、 MAX() 等统计方法**。





**2.1 4个基本统计聚合**

**
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowJBLy6PQIZKJlia3ryHSqtlsZaUdEMXFocdHEengzLngpedab0oNTzibz4xictB7MReiaMxFgDTdGbWOA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**
**

**这4个聚合我相信各位小伙伴一看就能明白，就简答看个示例，学习下语法即可：**

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
POST / exams / _search ? size = 0 {  "aggs": {    "avg_grade": {      "avg": {        "field": "grade"      }    }  }}
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

```
{  ...  "aggregations": {    "avg_grade": {      "value": 75.0    }  }}
```



**2.2 Value Count value计数聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



简单示例，了解语法：

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
POST / sales / _search ? size = 0 {  "aggs": {    "types_count": {      "value_count": {        "field": "type"      }    }  }}
```

结果：

- 
- 
- 
- 
- 
- 
- 

```
{ ..."aggregations": {    "types_count": {      "value": 7    }  }}
```





**2.3 Stats 统计聚合**

**
**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



就是一个聚合函数，包含了上述5种聚合，简单看个示例，学习语法：

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
POST / exams / _search ? size = 0 {  "aggs": {    "grades_stats": {      "stats": {        "field": "grade"      }    }  }}
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

```
{ ...  "aggregations": {    "grades_stats": {      "count": 2,      "min": 50.0,      "max": 100.0,      "avg": 75.0,      "sum": 150.0    }  }}
```





**2.4 Weighted Avg 加权平均聚合**



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowJBLy6PQIZKJlia3ryHSqtlsByhJpqNPQ4GWjFm6K5e07fYuyM0UofjUoSIHLpQkL2MgfnqDDlT0XA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



加权平均聚合和Avg Agg类似，**掌握其计算公式： ∑(value \* weight) / ∑(weight)。**



场景示例：博客文章指数计算：阅读量*作者影响力

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
POST / blogs_index / _search {  "size": 0,  "aggs": {    "weighted_grade": {      "weighted_avg": {        "value": {          "field": "views"        },        "weight": {          "field": "influence"        }      }    }  }}
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

```
{  ...  "aggregations": {    "weighted_grade": {      "value": 70.0    }  }}
```





**2.5 cardinality 基数聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



场景示例：统计已销售汽车的颜色一共有多少种

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
GET / cars / _search {  "size": 0,  "aggs": {    "distinct_colors": {      "cardinality": {        "field": "color"      }    }  }}
```

结果：

- 
- 
- 
- 
- 

```
"aggregations": {  "distinct_colors": {    "value": 3  }}
```





**2.6 Top Hits 热门匹配聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



场景示例：获取每种类型商品，其中价格最高的商品详情

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
POST / test / _search ? size = 0 {  "aggs": {    "top_tags": {      "terms": {        "field": "type",        "size": 3      },      "aggs": {        "top_sales_hits": {          "top_hits": {            "sort": [{              "price": {                "order": "desc"              }            }],            "_source": {              "includes": ["date", "price"]            },            "size": 1          }        }      }    }  }}
```

结果:

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
- 

```
"aggregations": {  "top_tags": {    "doc_count_error_upper_bound": 0,    "sum_other_doc_count": 0,    "buckets": [{      "key": 1,      "doc_count": 2,      "top_sales_hits": {        "hits": {          "total": {            "value": 2,            "relation": "eq"          },          "max_score": null,          "hits": [{            "_index": "test",            "_type": "_doc",            "_id": "ovDRLXMBqw5d_PggL2w6",            "_score": null,            "_source": {              "date": 6435,              "price": 3            },            "sort": [3]          }]        }      }    }, {      "key": 2,      "doc_count": 2,      "top_sales_hits": {        "hits": {          "total": {            "value": 2,            "relation": "eq"          },          "max_score": null,          "hits": [{            "_index": "test",            "_type": "_doc",            "_id": "o_DRLXMBqw5d_PggL2w6",            "_score": null,            "_source": {              "date": 5525,              "price": 5            },            "sort": [5]          }]        }      }    }]  }}
```





**2.7 Top Metrics 最高度量标准聚合**



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



看个示例，对比Top Hit Agg 的例子，如下DSL和上述例子效果一致，只是响应结构不同：

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
POST / sales / _search ? size = 0 {  "aggs": {    "top_tags": {      "terms": {        "field": "type",        "size": 3      },      "aggs": {        "top_sales_hits": {          "top_metrics": {            "metrics": [{              "field": "date"            }, {              "field": "price"            }],            "sort": {              "price": "desc"            }          }        }      }    }  }}
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
- 
- 
- 
- 
- 
- 
- 

```
"aggregations": {  "top_tags": {    "doc_count_error_upper_bound": 0,    "sum_other_doc_count": 0,    "buckets": [{      "key": 1,      "doc_count": 2,      "top_sales_hits": {        "top": [{          "sort": [3],          "metrics": {            "date": 6435,            "price": 3          }        }]      }    }, {      "key": 2,      "doc_count": 2,      "top_sales_hits": {        "top": [{          "sort": [5],          "metrics": {            "date": 5525,            "price": 5          }        }]      }    }]  }}
```



**2.8 剩余8种Metric聚合**



![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowJBLy6PQIZKJlia3ryHSqtls8cMHrGwQtqu9dDDaKrNLbKLFZ5ngQu6DUHP3Acicv3KWbPwiaAzAyTrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)