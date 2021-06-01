1、wildcard query、prefix query、fuzzy query **这3种模糊查询**的异同点是什么？

2、如何使用 terms_set query **检索Array类型的字段？**

**ps：文末有关于Term-level queries所有查询的总结！**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/11-1.png)

本文导航



**01 wildcard query**

**
**

检索**包含通配符表达式**（**未分析**）字段的文档。【ps：**等价于mysql 的 like 查询**】

> 通配符 *：它**匹配任何字符**序列（包括空字符） 占位符 ?：它匹配任何**单个字符**。

**请注意**，此查询的速度可能很慢，因为它需要迭代许多项。为了防止极慢的通配符查询，**通配符术语不应以通配符\*或？之一开头。**

**
**

wildcard query是很好理解的，简单看两个示例，学会DSL语句的编写即可。



**1）通配符 \***

```
GET /blogs_index/_search
{
    "query": {
        "wildcard" : { "author": "方*" }
    }
}
```

> 上述DSL语句，可以检索到所有文档。**等价于sql【where author like "方%”】**



**2）占位符 ?**

```
GET /blogs_index/_search
{
    "query": {
        "wildcard" : { "author": "方?" }
    }
}
```

> 上述DSL语句，检索结果为空。**等价于sql【where author like "方_”】**







**02 prefix query**

**
**

查找指定字段包含**以指定确切前缀开头**的术语的文档。

```
GET /_search
{ "query": {
    "prefix" : { "author": "方" }
  }
}
```

> 该DSL等价于 wildcard query 的 **"wildcard"** : { "author": "方*" }，**等价于sql【where author like "方%”】**







**03 fuzzy query**

**
**

模糊查询使用基于**Levenshtein编辑距离的相似度**。是一种**误拼写时**的fuzzy**模糊搜索技术**，**用于搜索的时候可能输入的文本会****出现误拼写的情况****。**比如输入"**方财兄"**，这时候也要匹配到**“方才兄”**。



通过简单示例，理解 fuzzy query：

```
GET /blogs_index/_search
{
    "query": {
        "fuzzy" : {
            "author": {
                "value": "方财兄",
                "fuzziness": 1,
                "prefix_length": 1,
                "max_expansions": 100
            }
        }
    }
}
```

参数解释：

> **fuzziness**：最大编辑距离【**一个字符串要与另一个字符串相同必须更改的一个字符数**】。默认为AUTO。
>
> **prefix_length**：不会被“模糊化”的初始字符数。这有助于减少必须检查的术语数量。默认为0。
>
> **max_expansions**：fuzzy查询将扩展到的最大术语数。默认为50。
>
> **transpositions**：是否支持模糊转置（ab→ ba）。默认值为false。

> 上述DSL**等价于sql【where author like “方_兄”or  author like “方财_”\**or  author like “方\*\*_\*\*财兄”\*\*or  author like “方财\*\*_\*\*兄”\*\*or  author like “方财兄_”\*\*\*\*\****】（**会根据上述的4个参数穷尽所有可能组合**）

**注意****：**如果prefix_length将设置为0，并且max_expansions将设置为很高的数字，则此查询可能会很繁琐。这可能会导致索引中的每一项都受到检查！







**04 exists query**

**
**

1）查找指定字段**包含任何非空值【不是null 也不是[ ]】的文档。****【ps：等价于mysql 的 is null】**

**
**

注意：这些值不属于空值

> 1、空字符串，例如""或"-" 2、包含null和另一个值的数组，例如[null, "foo"] 3、自定义null-value，在字段映射中定义

简单看个示例，学会DSL语句的编写即可：

```
1、查询 title字段不为 null 的文档
GET /blogs_index/_search
{
    "query": {
        "exists" : { "field" : "title" }
    }
}
```



**2）查询为null的字段，**应该使用：**must_not + exists**【ps：关于bool语句，TeHero在明天将为大家分享】

```
GET /blogs_index/_search
{
    "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "title"
                }
            }
        }
    }
}
```







**05 terms_set query**

**
**

返回的文档**至少匹配一个或多个检索**的术语。这些术语**未进行分析**，因此必须完全匹配。每个文档中必须匹配的术语数会有所不同，并由“**最小匹配项”字段**控制，或者由“最小匹配项”脚本中的每个文档计算。

> ps：terms_set query 在**对Array类型的字段**做检索时非常有用，特别是对于每个文档，需要**匹配的数量不一致时**。如果所有文档**需要匹配的数量一致**，可以使用**match query替代**。



**1) 数据准备**

```
PUT /term_set_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "codes": {
          "type": "keyword"
        },
        "required_matches": {
          "type": "integer"
        }
      }
    }
  }
}

PUT /term_set_index/_doc/1?refresh
{
    "codes": ["系统学习", "es","关注我"],
    "required_matches": 2
}
PUT /term_set_index/_doc/2?refresh
{
    "codes": ["系统", "学习"],
    "required_matches": 1
}
```

ps：控制**必须匹配术语的数量的字段**必须**是数字字段**。





**2) minimum_should_match_field**

```
GET /term_set_index/_search
{
  "query": {
    "terms_set": {
      "codes": {
        "terms": [
          "关注我",
          "学习"
        ],
        "minimum_should_match_field": "required_matches"
      }
    }
  }
}
```

> 分析：该DSL语句可以检索到文档2。
>
> **对于文档1，**需要**至少匹配2个term**，但是在检索terms里，只能匹配上【关注我】一个term，所以文档1不符合检索条件**；**
>
> **对于文档2，**只需要匹配一个term，刚好能匹配上检索terms里的【学习】。





**3) minimum_should_match_script**

```
GET /term_set_index/_search
{
  "query": {
    "terms_set": {
      "codes": {
        "terms": [
          "系统学习",
          "关注我"
        ],
        "minimum_should_match_script": {
          "source": " doc['required_matches'].value"
        }
      }
    }
  }
}
```

> 等价于上一句DSL。





**4) 与match query的比较**

**
**

**当每个文档的required_matches值都相同时，**上述两句DSL与下面的match query 语句检索效果**完全一致：**

```
GET /term_set_index/_search
{
    "query": {
        "match": {
            "codes" : {
                "query":  "系统学习 关注我",
                "analyzer": "whitespace",
                "minimum_should_match": 2
            }
        }
    }
}
```

> 分析：DSL语句使用 "analyzer": "whitespace", 所以 **query会被分词两个Token/term【系统学习】【关注我】**。"minimum_should_match": 2，所以可以检索到文档1。

> **ps：关于Term-level queries 与 Full Text queries 的对比分析，使用场景对比，后续TeHero将详细为大家讲解！敬请期待哟！**







**07 仅用于了解的term-level queries**



**1)  regexp query——使用正则表达式术语查询**

```
GET /_search
{
    "query": {
        "regexp":{
            "name.first": "s.*y"
        }
    }
}
```

> 注意：**regexp查询的性能在很大程度上取决于所选的正则表达式。**匹配所有类似的东西.*都很慢，而且使用环视正则表达式也很慢。如果可能，应在正则表达式开始之前尝试使用长前缀。【ps，正在表达式，在日志系统使用较多，后面在Logstash系列，TeHero再为大家讲解】



**2)  type query**

**
**

筛选与提供的文档/映射类型匹配的文档。**【几乎无用，因为 type 在7.x已被弃用，并将在8.x版本中被删除】**

```
GET /your_index/_search
{
    "query": {
        "type" : {
            "value" : "_doc"
        }
    }
}
```



**3) ids query**

**
**

根据index的 **_id 字段**检索文档

```
GET /_search
{
    "query": {
        "ids" : {
            "type" : "_doc",
            "values" : ["1", "4", "100"]
        }
    }
}
```





**08 总结**



到此我们已经学完 **Term-level queries 的11种查询**，下面我们进行一个简单的总结：

> 1、 所有的 Term-level queries 的**检索关键词都不会分词**；
>
> 2、**term query** 等价于sql【where Token = “检索词”】；
>
> 3、**terms query** 等价于sql【where Token in ( 检索词List )】；
>
> 4、**range query** 掌握Date Math 和对 range类型字段检索的 relation参数；
>
> 5、掌握 wildcard query、prefix query、fuzzy query 这3种模糊查询；
>
> 6、**terms_set query** 用于检索Array类型的字段，但文档中必须定义一个数字字段——表示最低匹配的term数量；
>
> 7、**exists query** 用于检索为null的字段，检索不为null的字段使用 must_not + exists。