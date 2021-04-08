相关度不只与全文查询有关，也需要将结构化的数据考虑其中。

- 一个度假屋，需要一些的详细特征（空调、海景、免费 WiFi ），匹配的特征越多相关度越高
- 结构化的数据匹配的子句越多，相关性评分越高。

多条查询子句被合并为一条复合查询语句，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中

[使用 _explain 语法来理解评分标准](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-intro.html#explain)



---

**<u>*ES 使用 bool 模型来查找匹配文档，使用”实用评分函数“来计算相关度。”实用评分函数“ 借鉴了 tf/idf 和 向量空间模型。同时加入了一些特性：协调因子、字段长度归一化、词或查询语句的权重提升*</u>**



ES的相似度算法：TF/IDF 检索词频率/反向文档频率

- [检索词频率/词频](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scoring-theory.html#tf)：

  出现频率越高，相关性也越高

  **字段**中出现过 5 次要比只出现过 1 次的相关性高。

- [反向文档频率](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scoring-theory.html#idf)

  频率越高，相关性越低。

  检索词出现在多数**文档**中会比出现在少数文档中的权重更低。

- [字段长度准则](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scoring-theory.html#field-norm)

  长度越长，相关性越低。

  在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大

注意：

- 禁用词频：

  将参数 `index_options` 设置为 `docs` 可以禁用词频统计及词频位置，这个映射的字段不会计算词的出现次数，对于短语或近似查询也不可用。要求精确查询的 `not_analyzed` 字符串字段会默认使用该设置

- 禁用归一化：

  对于有些应用场景如日志，归一值不是很有用，要关心的只是字段是否包含特殊的错误码或者特定的浏览器唯一标识符。



查询通常不止一个词，所以需要一种合并多词权重的方式——向量空间模型：

- 看向量的夹角来计算相关度（线性代数）



归一化因子：

- 为了能够将两个结果进行比较

协调因子

- 为那些查询词包含度高的文档提供奖励，文档里出现的查询词越多，它越有机会成为好的匹配结果
- 默认会对所有 `should` 语句使用协调功能
- [todo：何时关闭协调因子](https://www.elastic.co/guide/cn/elasticsearch/guide/current/practical-scoring-function.html#coord)

权重提升：

- 索引时 和 查询时的权重提升

- 任意类型的查询都能接受 `boost` 参数

  - 将 `boost` 设置为 `2` ，并不代表最终的评分 `_score` 是原值的两倍

- 当在多个索引中搜索时，可以使用参数 `indices_boost` 来提升整个索引的权重

  ```json
  GET /docs_2014_*/_search 
  {
    "indices_boost": { 
      "docs_2014_10": 3,
      "docs_2014_09": 2
    },
    "query": {
      "match": {
        "text": "quick brown fox"
      }
    }
  }
  ```

---



- Bool 查询可以调整查询结构中查询语句的层次来改变他的重要性

  ```json
  # quick OR brown OR red OR fox
  GET /_search
  {
    "query": {
      "bool": {
        "should": [
          { "term": { "text": "quick" }},
          { "term": { "text": "brown" }},
          { "term": { "text": "red"   }},
          { "term": { "text": "fox"   }}
        ]
      }
    }
  }
  
  # quick OR (brown OR red) OR fox
  # red 和 brown 处于相互竞争的层次
  GET /_search
  {
    "query": {
      "bool": {
        "should": [
          { "term": { "text": "quick" }},
          { "term": { "text": "fox"   }},
          {
            "bool": {
              "should": [
                { "term": { "text": "brown" }},
                { "term": { "text": "red"   }}
              ]
            }
          }
        ]
      }
    }
  }
  ```

- [negative_boost](https://www.elastic.co/guide/cn/elasticsearch/guide/current/not-quite-not.html#not-quite-not)
  - 接受 `positive` 和 `negative` 查询 
  - 仍然允许我们将关于水果或甜点的结果包括到结果中，但是使它们降级——即降低它们原来可能应有的排名
-  [constant_score 查询：](https://www.elastic.co/guide/cn/elasticsearch/guide/current/ignoring-tfidf.html#constant-score-query)
  - 有时候我们根本不关心 TF/IDF ，只想知道一个词是否在某个字段中出现过
  - 为任意一个匹配的文档指定评分 `1`，不匹配为0，并且也可以提升某些条件的权重
  - 和过滤器的区别：过滤器无法计算评分。这样就需要寻求一种方式将过滤器和查询间的差异抹平。 `function_score` 查询不仅正好可以扮演这个角色，而且有更强大的功能
- [function_score 查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/function-score-query.html#function-score-query)
  
  - ？？？？
  - modifier
  - [过滤集提升权重](https://www.elastic.co/guide/cn/elasticsearch/guide/current/function-score-filters.html) ???
  - 随机评分：相同分数的文档，均等相似的展现机率（即不是一个文档一直在前或者后）
  
- script_score





https://blog.csdn.net/weixin_40341116/article/details/80931573

https://www.elastic.co/guide/cn/elasticsearch/guide/current/not-quite-not.html

https://www.cnblogs.com/qdhxhz/p/11529107.html



