> 昨天为大家介绍了[**ES系列06：ik分词+Full text queries 之match query**](http://mp.weixin.qq.com/s?__biz=MzIxMjE3NjYwOQ==&mid=2247483825&idx=1&sn=0b294bd614ed4504577b5d155706fd5f&chksm=974b5a3fa03cd329c9393f184299b430728118a7005e236cce049e1af9d31f8c75037b80a1f9&scene=21#wechat_redirect)**。**今天TeHero为大家分享 **Full text queries 的 match_phrase query 和match_phrase_prefix query**，**同时从倒排序索引原理入手，将DSL语句转化为sql语句****，方便大家理解学习。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowLXVeVsapojLrmMs7ibtibLicJX6lN4lvic6WibH127DZg6aV7sVJeULZeibEC7H6icPRlndYbLeJliat5Tew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

本文结构【开局一张图】

> ps：上图的xmind文件，公众号后台**回复【es06】即可免费获取！**

## 一、在开始之前，完成数据准备：

```
# 创建映射
PUT /tehero_index
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
        "content": {
          "type": "keyword",
          "fields": {
            "ik_max_analyzer": {
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            },
            "ik_smart_analyzer": {
              "type": "text",
              "analyzer": "ik_smart"
            }
          }
        },
        "name":{
          "type":"text"
        },
        "createAt": {
          "type": "date"
        }
      }
    }
  }
}
# 导入测试数据
POST _bulk
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "1" } }
{ "id" : 1,"content":"关注我,系统学编程" }
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "2" } }
{ "id" : 2,"content":"系统学编程,关注我" }
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "3" } }
{ "id" : 3,"content":"系统编程,关注我" }
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "4" } }
{ "id" : 4,"content":"关注我,间隔系统学编程" }
```

## 二、根据ik_smart分词和content字段建立倒排序索引

原始数据：

> **{ "id" : 1,"content":"关注我,系统学编程" }** 

> **{ "id" : 2,"content":"系统学编程,关注我" }** 

> **{ "id" : 3,"content":"系统编程,关注我" }** 

> **{ "id" : 4,"content":"关注我,间隔系统学编程" }**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowLXVeVsapojLrmMs7ibtibLicJiaq10doVpKudBTOepQbT6MUJqm964sjmhgEibh8BicqpMEwstrtv9ia0ZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

数据的倒排序索引

> **ps：如果看不懂上图，请先阅读学习：****[ElasticSearch系列05：倒排序索引与分词Analysis](http://mp.weixin.qq.com/s?__biz=MzIxMjE3NjYwOQ==&mid=2247483805&idx=1&sn=d890e383339c2fe5bf265b2d53a8a270&chksm=974b5a13a03cd30558de75c891fc9d471f3477981f49b53c8bb26a84256d9e986e97f5d33808&scene=21#wechat_redirect)**

## 三、match query 对应到mysql

> **昨天有小伙伴反馈说，match query 的实例写得太枯燥，建议和mysql对比讲解，今天它来了！**

```
# DSL 语句
GET /tehero_index/_doc/_search
{
  "query":{
    "match":{
      "content.ik_smart_analyzer":"系统编程"
    }
  }
}
```

DSL执行步骤分析：

- 1）检索词“系统编程”被ik_smart分词器分词为**两个Token【系统】【编程】；**

  **
  **

- 2）将这两个Token在【倒排索引】中，针对Token字段进行检索，**等价于sql：****【where Token = 系统 or Token = 编程】；**

  

- 3）对照图【数据的倒排序索引】，可见，**该DSL能检索到所有文档，文档3的评分最高（因为它包含两个Token），其他3个文档评分相同。**

> 有了对应到mysql 的例子，我想大家对match query 这个查询语句，就应该有一个很好的理解。那么接下来，开始学习今天的新知识：**match_phrase query 和match_phrase_prefix query**

## 四、match_phrase query

> match_phrase查询分**析文本并根据分析的文本创建一个短语查询。**match_phrase **会将检索关键词分词**。match_phrase的分词结果必**须在被检索字段的分词中都包含**，而且**顺序必须相同，**而且**默认必须都是连续的。**

简单看个例子，与match query 对比下，就很好理解了：

**使用 match_phrase 查询：**

```
# 使用match_phrase查询，ik_smart分词
GET /tehero_index/_doc/_search
{
    "query": {
        "match_phrase": {
            "content.ik_smart_analyzer": {
            	"query": "关注我,系统学"
            }
        }
    }
}

# 结果：只有文档1
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
    "max_score": 0.7370664,
    "hits": [
      {
        "_index": "tehero_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.7370664,
        "_source": {
          "id": 1,
          "content": "关注我,系统学编程"
        }
      }
    ]
  }
}
```

**使用 match 查询：**

```
# 使用match查询，ik_smart分词
GET /tehero_index/_doc/_search
{
    "query": {
        "match": {
            "content.ik_smart_analyzer": {
            	"query": "关注我,系统学"
            }
        }
    }
}
# 可以查询出所有结果
```

> 分析：上面的例子使用的**分词器是ik_smart，**所以**检索词“关注我，系统学”会被分词为3个Token【关注、我、系统学】**；而文档1、文档2和文档4 的content被分词后**都包含这3个关键词**，但是**只有文档1的Token的顺序和检索词一致，且连续**。所以使用 match_phrase 查询**只能查询到文档1**（ps：文档2 Token顺序不一致；文档4 Token不连续；文档3 Token没有完全包含）。**使用 match查询**可以查询到所有文档，是因为所有文档**都有【关注、我】这两个Token**。

- **4.1 match_phrase 核心参数：slop 参数-Token之间的位置距离容差值**

```
# 将上面的 match_phrase 查询新增一个 slop参数
GET /tehero_index/_doc/_search
{
    "query": {
        "match_phrase": {
            "content.ik_smart_analyzer": {
            	"query": "关注我,系统学",
            	"slop":1
            }
        }
    }
}
# 结果：文档1和文档4都被检索出来
```

> **分析：使用 analyze 接口 分析下文档4的Token**

```
# 文档4 content 的分词
GET /_analyze
{
  "text": ["关注我,间隔系统学编程"],
  "analyzer": "ik_smart"
}
# 结果
{
  "tokens": [
    {
      "token": "关注",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "我",
      "start_offset": 2,
      "end_offset": 3,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "间隔",
      "start_offset": 4,
      "end_offset": 6,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "系统学",
      "start_offset": 6,
      "end_offset": 9,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "编程",
      "start_offset": 9,
      "end_offset": 11,
      "type": "CN_WORD",
      "position": 4
    }
  ]
}
```

> 通过分词测试，发现Token【我】与【系统学】的**position差值为1(**等于slop的值**)**，**所以文档4也被检索出来了。**

ps：如果没看明白**，那就来看下match_phrase query对应到mysql是怎样的吧！**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowLXVeVsapojLrmMs7ibtibLicJiaq10doVpKudBTOepQbT6MUJqm964sjmhgEibh8BicqpMEwstrtv9ia0ZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

数据的倒排序索引

- **4.2** **match_phrase query** **对应到mysql**

```
# DSL语句
GET /tehero_index/_doc/_search
{
  "query":{
    "match_phrase":{
      "content.ik_smart_analyzer":"系统编程"
    }
  }
}
```

DSL执行步骤分析：

- 1）检索词“系统编程”被分词为**两个Token【系统，Position=0】【编程，Position=1】；**
- 2）倒排索引检索时，**等价于sql：**【where Token = 系统  and 【该and删除】**系统_Position=0** and Token = 编程 and  **编程_Position=系统_Position+1**】；
- 3）对照图【数据的倒排序索引】，只有文档3满足条件，所以该DSL语句只能查询到文档3。

## 五、match_phrase_prefix query

> 与match_phrase查询类似，但是**会对最后一个Token在倒排序索引列表中进行通配符搜索**。Token的模糊匹配数控制：**max_expansions 默认值为50。**我们使用**content.ik_smart_analyzer** 这个字段中的**【系统学】**（文档1、2、4 包含）和**【系统】**（文档3包含）这**两个Token**来讲解match_phraseprefix 的用法：（因为**使用的是ik_smart分词器**，所以【系统学】就只能被分词为一个Token）

```
# 1、先使用match_phrase查询，没有结果
GET tehero_index/_doc/_search
{
  "query": {
    "match_phrase": {
      "content.ik_smart_analyzer": {
        "query": "系"
      }
    }
  }
}

# 2、使用match_phrase_prefix查询， "max_expansions": 1，得到文档3
GET tehero_index/_doc/_search
{
  "query": {
    "match_phrase_prefix": {
      "content.ik_smart_analyzer": {
        "query": "系",
        "max_expansions": 1
      }
    }
  }
}

# 3、使用match_phrase_prefix查询， "max_expansions": 2，得到所有文档
GET tehero_index/_doc/_search
{
  "query": {
    "match_phrase_prefix": {
      "content.ik_smart_analyzer": {
        "query": "系",
        "max_expansions": 2
      }
    }
  }
}
```

> 结果分析：【语句1】查不到结果，是因为**根据ik_smart分词器生成的倒排序索引中**，所有文档中都**不包含Token【系】**；【语句2】查询到文档3，是因为**文档3包含Token【系统】，同时** "max_expansions": 1，所以**检索关键词【系】+ 1个通配符匹配**，就可以匹配到**一个Token**【系统】；【语句3】查询到所有文档，是因为"max_expansions": 2，所以**检索关键词【系】+ 2个通配符匹配**，就可以匹配到**两个Token**【系统、系统学】，所以就可以查询到所有。回忆下，之前所讲的es倒排序索引原理：**先分词创建倒排序索引，再检索倒排序索引得到文档**，就很好理解了**。**【链接！！！！】

注意："max_expansions"的**值最小为1**，**哪怕你设置为0，依然会 + 1个通配符匹配；所以，**尽量不要用该语句，因为，**最后一个Token始终要去扫描大量的索引**，**性能可能会很差。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowLXVeVsapojLrmMs7ibtibLicJiaq10doVpKudBTOepQbT6MUJqm964sjmhgEibh8BicqpMEwstrtv9ia0ZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

数据的倒排序索引

- **5.1 match_phrase_prefix query 对应到mysql**

```
GET tehero_index/_doc/_search
{
  "query": {
    "match_phrase_prefix": {
      "content.ik_smart_analyzer": {
        "query": "系",
        "max_expansions": 1
      }
    }
  }
}
```

DSL执行步骤分析：

- 1）检索词“系”被分词为一个**个Token【系】+ 1个通配符；**
- 2）倒排索引检索时，等价于sql：【where Token = 系  or Token **like “系_”**】；
- 3）对照图【数据的倒排序索引】，只有**文档3满足条件包含Token【系统】**，所以该DSL语句只能查询到文档3。

## 六、总结

到此，我们已经学习了 Full text queries最常用的3种查询：

> **1）match query**：用于执行全文查询的标准查询，包括**模糊匹配和短语或接近查询。****重要参数：****控制Token之间的布尔关系：operator：or/and**

> **2）match_phrase query：**与match查询类似，**但用于匹配确切的短语或单词接近匹配。****重要参数：****Token之间的位置距离：slop 参数**

> **3）match_phrase_prefix query：**与match_phrase查询类似，但是会**对最后一个Token在倒排序索引列表中进行通配符搜索。****重要参数：****模糊匹配数控制：max_expansions 默认值50，最小值为1**