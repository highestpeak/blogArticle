- 垂直扩容和水平扩容

- *cluster* 、 *node* 、 *shard* 集群、节点和分片 的配置

  节点

  - 主节点 任何节点
  - 一个 es 实例=一个节点。多个相同名字的节点（为什么是相同？）--集群
  - 一个 集群 是一组拥有相同 `cluster.name` 的节点，废话，
  - 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。？

  分片（给数据装一个标准大小的箱子？）

  - 副本分片
  - 一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。 我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互
  - 分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里
  - 一个分片可以是 *主* 分片或者 *副本* 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量. p-主分片 r-副本分片
    
    - 所有新近被索引的文档都将会保存在主分片上，然后被并行的复制到对应的副本分片上。这就保证了我们既可以从主分片又可以从副本分片上获得文档
  - 

  - ```
    PUT /blogs
    {
       "settings" : {
          "number_of_shards" : 3,
          "number_of_replicas" : 1
       }
    }
    
    
    ```

  - 分片是一个功能完整的搜索引擎，它拥有使用一个节点上的所有资源的能力

  - 有6个分片（3个主分片和3个副本分片）的索引可以最大扩容到6个节点，每个节点上存在一个分片，并且每个分片拥有所在节点的全部资源

  - 

- 文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互

- 所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈？？？为什么不会呢？

- 索引内任意一个文档都归属于一个主分片
  
  - ？所以主分片的数目决定着索引能够保存的最大数据量
  
- 集群健康 green yellow red

- 游标不太懂

- 







- 索引：动词-insert/名词 倒排索引（每个字段-字段值有一个索引，索引到各自的记录） https://zhuanlan.zhihu.com/p/33671444

- ``` 
  路径 /megacorp/employee/1 包含了三部分的信息：
  
  megacorp
  索引名称
  employee
  类型名称----(note: 为什么叫类型啊？有点怪)
  1
  特定雇员的ID
  
  ```

- GET /megacorp/employee/1 使用http命令 put、get、post、等等

- 搜索所有雇员

  ```sense
  GET /megacorp/employee/_search
  ```

- 搜索姓氏为 ``Smith`` 的雇员

  ```sense
  GET /megacorp/employee/_search?q=last_name:Smith 
  ```

- Query-string 搜索--轻量搜索

  ```
  GET /megacorp/employee/_search
  {
      "query" : {
          "match" : {
              "last_name" : "Smith"
          }
      }
  }
  ```

- 查询表达式搜索（DSL）（sql DSL?)

  ```
  GET /megacorp/employee/_search
  {
      "query" : {
          "bool": {
              "must": {
                  "match" : {
                      "last_name" : "smith" 
                  }
              },
              "filter": {
                  "range" : {
                      "age" : { "gt" : 30 } 
                  }
              }
          }
      }
  }
  ```

- 全文搜索

   Elasticsearch 如何 *在* 全文属性上搜索并返回**相关性**最强的结果。Elasticsearch中的 *相关性* 概念非常重要，也是完全区别于传统关系型数据库的一个概念，

  - match_phrase 搜索 （短语搜索）

- 高亮搜索 会把匹配地方加上 html 的标签 `<em>`

- 聚合搜索 结果会有hits和aggregations，是先对match的结果是hit再对这个结果进行聚合，和sql类似？

  - ```
    GET /megacorp/employee/_search
    {
        "aggs" : {
            "all_interests" : {
                "terms" : { "field" : "interests" },
                "aggs" : {
                    "avg_age" : {
                        "avg" : { "field" : "age" }
                    }
                }
            }
        }
    }
    
    GET /megacorp/employee/_search
    {
      "query": {
        "match": {
          "last_name": "smith"
        }
    },
      "aggs": {
        "all_interests": {
          "terms": {
            "field": "interests"
          }
        }
      }
    }
    ```

  - todo: Search api wiki list 



- ```sense
  POST /my_store/products/_bulk
  GET /my_store/products/_search
  GET /my_store/_analyze
  DELETE /my_store
  注意后面的API，即 _bulk、_search、_analyze、_my_store
  ```





> 深入搜索

> 结构化搜索

- 尽可能多的使用过滤式查询

- 过滤器很快，不计算相关度

- term、constant_score、term查询类似“KDKE-B-9947-#kL5”的文本、

  ```
  {
      "term" : {
          "price" : 20
      }
  }
  ```

  - terms

    ```
    {
        "terms" : {
            "price" : [20, 30]
        }
    }
    ```

  - `term` 和 `terms` 是 *包含（contains）* 操作，精确匹配需要额外字段

  - `term` 查询只对倒排索引的词项精确匹配，不会对词的多样性进行处理（近义词等）

- 布尔过滤器（组合条件查询）、可以嵌套布尔过滤器、

  ```js
  {
     "bool" : {
        "must" :     [],
        "should" :   [],
        "must_not" : [],
     }
  }
    
  一个例子
  GET /my_store/products/_search
  {
     "query" : {
        "filtered" : { 
           "filter" : {
              "bool" : {
                "should" : [
                   { "term" : {"price" : 20}}, 
                   { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} 
                ],
                "must_not" : {
                   "term" : {"price" : 30} 
                }
             }
           }
        }
     }
  }
    
  一个例子
  GET /my_index/my_type/_search
  {
    "query": {
      "bool": {
        "must":     { "match": { "title": "quick" }},
        "must_not": { "match": { "title": "lazy"  }},
        "should": [
                    { "match": { "title": "brown" }},
                    { "match": { "title": "dog"   }}
        ]
      }
    }
  }
    
    bool 查询 和 bool 过滤有类似的功能，只有一个重要的区别，即 should
    bool 过滤 should 至少有一个语句要匹配
    bool 查询 should 中的语句不必需要被匹配
  ```

- range：数字、日期、字符串

- exists：is not null

> 全文搜索

- 相关性、分析
- 词项查询、全文查询
  - 全文查询：match` 或 `query_string
  - match 既能处理全文字段，又能处理精确字段，全文字段和精确字段？？
  - `match` 查询还可以接受 `operator` 操作符，不去匹配 `brown OR dog` ，而通过匹配 `brown AND dog`
    - minimum_should_match 设置匹配百分比
  - 多词match查询和有些bool查询等价
  - bool查询的should表示，如果满足，则分数将提高，boost 参数也可以来设置
  - 分析器不同会影响查询结果
- 评分的计算
- 多字段查询 有点晕

> 多字段搜索

- 多字符串？单字符串？
- 最佳字段、多数字段？most_fields？、混合字段
  - 最佳字段 bool、dis_max 、tie_breaker
- `multi_match` 查询为能在多个字段上反复执行相同查询提供了一种便捷方式
- 跨字段实体搜索和多字符串？
- 字段中心？词中心？
- Best_field、most_field、cross_fields
- cross_fields 和 自定义 _all 字段
- 



```sense
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

**`megacorp`**索引名称、**`employee`**类型名称、**`1`**特定雇员的ID



**什么是全文字段？**

*精确值* （它包括 `string` 字段）的字段和代表 *全文* 的字段

查询全文数据要微妙的多。我们问的不只是“这个文档匹配查询吗”，而是“该文档匹配查询的程度有多大？”换句话说，该文档与给定查询的相关性如何？

- 搜索 `UK` ，会返回包含 `United Kindom` 的文档。
- 搜索 `jump` ，会匹配 `jumped` ， `jumps` ， `jumping` ，甚至是 `leap` 。
- 搜索 `johnny walker` 会匹配 `Johnnie Walker` ， `johnnie depp` 应该匹配 `Johnny Depp` 。
- `fox news hunting` 应该返回福克斯新闻（ Foxs News ）中关于狩猎的故事，同时， `fox hunting news` 应该返回关于猎狐的故事。

为了促进这类在全文域中的查询，Elasticsearch 首先 *分析* 文档，之后根据结果创建 *倒排索引* 

分析和倒排索引是为了全文搜索的

还是不太懂全文字段，一个字符串即是精确字段也是全文字段？
全文通常是指非结构化的数据，但这里有一个误解：自然语言是高度结构化的。问题在于自然语言的规则是复杂的，导致计算机难以正确解析。例如，考虑这条语句：

```
May is fun but June bores me.
```

它指的是月份还是人？



分词和标准化的过程称为 *分析*

*分析* 包含下面的过程：

- 首先，将一块文本分成适合于倒排索引的独立的 *词条* ，
- 之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 *recall*

字符过滤器----》分词器-----〉token 过滤器

去掉没用的词（html、div）---》字符串分成词列表-----〉小/大写单词、删除a-the等、增加近义词等

索引和搜索全文域时都需要 分析

[analysis 分析器按照顺序执行](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html#custom-analyzers)



过滤和查询两个步骤



[重要的查询：](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_most_important_queries.html#_most_important_queries)

- match_all
- match
- multi_match
- range
- term
- terms
- exists missing

组合多查询：

- bool：must、must_not、should
- constant_score

`alidate-query` API 可以用来验证查询是否合法

```sense
GET /gb/tweet/_validate/query
{...}
```

- 告诉查询是否合法

```sense
GET /gb/tweet/_validate/query?explain
{...}
```

- 告诉查询不合法的原因

```sense
GET /_validate/query?explain
{...}
```

- 如何解析查询的



分析器、如何编写分析器

analysis 和 analyzer

[分析与分析器 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html)

[Ngrams 在部分匹配的应用 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_ngrams_for_partial_matching.html)

[自定义分析器 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html)

- es 节点 集群 
- 主节点 任何节点 
- 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。？
- 垂直扩容和水平扩容
- 副本分片
- *cluster* 、 *node* 、 *shard*
- 文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互
- 索引内任意一个文档都归属于一个主分片
  - ？所以主分片的数目决定着索引能够保存的最大数据量
- 一个 集群 是一组拥有相同 `cluster.name` 的节点？
- 



嵌套字段：

https://www.elastic.co/guide/cn/elasticsearch/guide/current/nested-objects.html

通过这个也了解es对字段的处理存储方式



