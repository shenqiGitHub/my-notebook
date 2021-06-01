## ElasticSearch系列05：倒排序索引与分词Analysis

原创 TeHero [方才编程](javascript:void(0);) *2020-05-17*

收录于话题

\#系统学ElasticSearch

18个

> 引言：上一节我们学习ES索引和文档的CURD，本来计划这节就开始介绍ES的Query DSL，但考虑再三，还是应该**先学习了解“倒排序索引”和“Analysis”**，这样，对于检索才会有一个更好的理解，才能更好的应用。【关注公众号：**ZeroTeHero，系统学习ES**】

## 一、 倒排索引是什么？

> 倒排索引是 Elasticsearch 中非常**重要的索引结构**，是从**文档单词到文档 ID** 的映射过程

**1.1 通过示例，简单理解下**

>   就拿专栏文章来说，我们平时在各大平台根据关键词检索时，使用到的技术就有“倒排序索引”。

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/05-1.png)

数据结构

假设我们的文章的储存结果如上，对于关系型数据库mysql来说，**普通的索引结构就是“id->题目->内容”，**在我们搜索的时候，如果我们知道id或者题目**，那么检索效率是很高效的，因为“id”、“题目”是很方便创建索引的。**

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/05-2.png)

正向索引

但是当我们只有一个检索关键词，比如需求是**搜索到与“倒排序索引”相关的文章**时，在索引结构是“id->题目->内容”时，就只能对“题目”和“内容”进行**全文扫描了，当数量级上去后，效率是没办法接受的！**对于这类的搜索，关系型数据库的索引就很难应付了，**适合使用全文搜索的倒排索引。**

那么**倒排序索引**的结构是怎样的呢？简单来讲**就是“以内容的关键词”建立索引，**映射关系为**“内容的关键词->ID”。**这样的话，我们只需要在“关键词”中进行检索，效率肯定更快。

![图片](/Users/qishen/Documents/my-notebook/Middleware/elasticsearch/ES系列/image.assets/05-3.png)

倒排序索引

**1.2 核心组成**

- 倒排序索引包含两个部分：

- **》单词词典：**记录所有文档单词，记录单词到倒排列表的关联关系

- 》**倒排列表：**记录单词与对应文档结合，由倒排索引项组成

- 倒排索引项：

- 》**文档**

- 》**词频 TF** - 单词在文档中出现的次数，用于相关性评分

- 》**位置（Position）-** 单词在文档中分词的位置，用于phrase query

- 》**偏移（Offset）**- 记录单词开始结束的位置，实现高亮显示

  

  

  

- 》**偏移（Offset）**- 记录单词开始结束的位置，实现高亮显示



> 举个简单例子，理解下“倒排索引项”：以 Token“学习”为例：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowKOWCgl0CiaQ13Tg22jKtcibABDdUOGEWz2cqsjspl7kGuhqDFtSPNNxbTnSRVHp1AeGnhiamicwjC9MQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

倒排序索引项List

## 二、倒排索引是怎么工作的？

> 主要包括2个过程：1、创建倒排索引；2、倒排索引搜索

**2.1 创建倒排索引**

> 还是使用上面的例子。先对**文档的内容**进行分词，形成一个个的 **token**，也就是 **单词**，然后保存这些 token 与文档的对应关系。结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowKOWCgl0CiaQ13Tg22jKtcibAVicTKlB6v0vdL9GvvWMOWYjNibg2E7dt6XJ8PazpZ9xucKboRWRIXzxg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**2.2 倒排索引搜索**

> 搜索示例1：“学习索引”

- 先分词，得到两个Token：“学习”、“索引”
- 然后去倒排索引中进行匹配

这2个Token在2个文档中都匹配，所以2个文档都会返回，而且分数相同。

> 搜索示例2：“学习es”

同样，2个文档都匹配，都会返回。但是**文档1的相关性评分会高于文档2，**因为文档1匹配了两个Token，而文档2只匹配了一个Token【学习】。

> 通过上面的讲解，我们学习了解了：**倒排序索引是什么及其工作流程**。其中有一个非常重要的环节——**对文档进行分词**，得到Token。**那么这个分词过程，是怎样进行的呢？**



## 三、Analysis 进行分词

> Analysis：即文本分析，是把**全文本转化为一系列单词（term/token）的过程**，也叫分词；在Elasticsearch 中可通**过内置分词器实现分词，也可以按需定制分词器**。

**3.1 Analyzer 由三部分组成**

> • Character Filters：原始文本处理，如去除 html • Tokenizer：按照规则切分为单词 • Token Filters：对切分单词加工、小写、删除 stopwords，增加同义词

**3.2 Analyzer 分词过程简介**

- **1）字符过滤器  character filter**

首先，字符串按顺序通过每个字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 and。

- **2）分词器 tokenizer**

其次，字符串被 分词器 分为单个的词条。一个 whitespace的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WO9qeUgIowKU4z8gsQ3dYlAic2h0hbicr0ySvx54h0Jgqz9pfqoRgxYyzGnwD41SRArCtPkJ8JUCGT3gn8dgKdPA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ES分词器汇总

- **3）令牌过滤器token filter**

最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条，例如，lowercase token filter  小写化（将ES转为es）、stop token filter 删除词条（例如， 像 a， and， the 等无用词），或者synonym token filter 增加词条（例如，像 jump 和 leap 这种同义词）。



**3.3 自定义分析器**

```
#1、定义名为“custom_analyzer”的自定义分析器：大写转为小写
PUT tehero_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
# 2、该字段my_text使用custom_analyzer分析器
  "mappings": {
    "_doc": {
      "properties": {
        "my_text": {
          "type": "text",
          "analyzer": "custom_analyzer"
        }
      }
    }
  }
}
```

**3.4 测试分词**

```
# 要引用此分析器，analyzeAPI必须指定索引名称。
# 直接使用分析器
GET tehero_index/_analyze 
{
  "analyzer": "custom_analyzer",
  "text":"关注TeHero，系统学习ES"
}
# 通过字段使用
GET tehero_index/_analyze
{
  "field": "my_text",
  "text": "关注TeHero，系统学习ES"
}

```

效果：

```
{
  "tokens": [
    {
      "token": "关",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "注",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "tehero",
      "start_offset": 2,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 2
    }
…………
]
```

也可以直接使用analyzer

```
POST _analyze

{

  "analyzer": "whitespace",

  "text":"关注TeHero 系统学习ES"

}

# 也可直接设置tokenizer和filter

POST _analyze

{

  "tokenizer": "standard",

  "filter":  [ "lowercase" ],

  "text":"关注TeHero 系统学习ES"

}
```

效果：

```
{

  "tokens": [

    {

      "token": "关注TeHero",

      "start_offset": 0,

      "end_offset": 8,

      "type": "word",

      "position": 0

    },

    {

      "token": "系统学习ES",

      "start_offset": 9,

      "end_offset": 15,

      "type": "word",

      "position": 1

    }

  ]

}
```

> 从analyzeAPI 的输出可以看出，分析器不仅将搜索词转换为Token，而且还记录 每个Token的顺序或相对位置（用于短语查询或单词接近性查询），以及每个Token的开始和结束字符偏移量原始文字中的字词（用于突出显示搜索摘要）。