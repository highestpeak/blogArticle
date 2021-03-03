

- match 查询
- 词相似度、部分匹配、模糊匹配、语言感知
- _score：最佳文档、去除长尾
  - 长尾数据比如说我们搜索4个关键词，但很多文档只匹配1个，也显示出来了，这些文档其实不是我们想要的，可以通过这几个参数的设置，将门槛提高，过滤掉长尾数据
- 必须先删除旧索引，才能创建新的正确的索引
- bool 既可以在filter也可以在查询，
- 需要例子来说明每个地方的情况，这些各种各样的语法就是为了不同的搜索情形设立的，所以依据不同的搜索情形来看也好？
  - todo： 例子
- 各个查询的评分公式

> 迷惑

- 精确值字段和带有分析器的字段？还是不太搞得懂这些字段都有什么？
- 整个存储结构，index、类型、字段？字段类型：精确、未分析和已分析？倒排索引的建立过程？
  - 字段和分析器？
- ***基于词项与基于全文？？？***
  - ***所以是说 词项和全文是指的字段？字段是全文还是词项的？***
- [为什么说多字段搜索要比单字段搜索要简单？](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_single_query_string.html#_single_query_string)
  - 因为多字段搜索已经指定了每个字段及其对应的值，而单字段搜索用户只输入了一串字符串，我需要去猜测这个字符串是命中哪个字段或者哪些字段的组合情况？
  - 关键在于用户输入了多个字符串还是一个字符串？
- ***单字符串查询的几个问题：最佳字段、多数字段、混合字段？？？***
  - 还是不太懂 单字符串的几种情形：最佳、多数、跨越？？？
- ***字段中心式 和 词中心式？***
- (+first_name:peter +first_name:smith) 这种结果怎么看？



搜索：结构化搜索、全文搜索

搜索：单字段搜索、多字段搜索

# 结构化搜索

- 精确的格式
- 数字-时间 范围、两值比较、时间日期数字、结构化的文本（枚举文本、通用产品码）
- 结果：是/否，无相关度和评分，包括和排除处理
- 过滤器查询：速度快、不计算相关度和评分、可缓存
  - Todo：[过滤器缓存](https://www.elastic.co/guide/cn/elasticsearch/guide/current/filter-caching.html)
  - Todo：[非评分查询的操作](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_finding_exact_values.html#_internal_filter_operation) 
  - 包括了：精确值、组合过滤器查询、范围
- 其他问题： 
  - Todo：null值和缓存

## 精确值

- 单个精确值

- 多个精确值

  - 只需要把 term 字段的值改为数组

    ``` sense
    {
        "terms" : {
            "price" : [20, 30]
        }
    }
    ```

  - Todo：[包含而不是相等（列表什么情况下算匹配）](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_finding_multiple_exact_values.html#_%E5%8C%85%E5%90%AB%E8%80%8C%E4%B8%8D%E6%98%AF%E7%9B%B8%E7%AD%89)

  - Todo：精确相等就需要增加并索引另一个字段（列表完全相等）（link 同上）

- term、match都可以精确查询

## 组合过滤器

```js
{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
   }
}
```

- must==and、must_not==not、should==or
  - or 至少有一个语句要匹配
- bool 就是一个条件表达式的在es中的形式

## 范围

- 数字范围、日期范围、字符串范围

# 全文搜索

- 相关性、分析
  - Todo：相关性算法、分析的目的和过程
  - [Todo：被破坏的相关度](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-is-broken.html)
- 基于词项的查询：不是所有查询都有分析阶段
  - term、fuzzy 查询没有分析阶段
  - term 只对倒排索引的词项精确匹配，不会对词的多样性进行处理（大小写）
- 基于全文的查询：
  - [todo：match 和 query_string 查询 的多种情况？](https://www.elastic.co/guide/cn/elasticsearch/guide/current/term-vs-full-text.html#term-vs-full-text)

## match 查询

- 既能全文查询也能精确查询

- 查询单个词

  - [Todo：match 查询全文字段的单个词](https://www.elastic.co/guide/cn/elasticsearch/guide/current/match-query.html#_%E5%8D%95%E4%B8%AA%E8%AF%8D%E6%9F%A5%E8%AF%A2)

- 查询多个词

  - 被匹配的词项越多，文档就越相关，这会影响分数

  - 可以提高精度，通过 operator 的方式

    eg：不去匹配 `brown OR dog` ，而通过匹配 `brown AND dog` 找到所有文档

  - 可以用 minimum_should_match 控制精度（百分比）

## bool 查询

- bool查询和bool过滤的区别？
  - 在于should的处理，如果包含我们认为它们更相关
  - must 和 must_not的处理是相同的
  - 可以通过 minimum_should_match 来控制需要匹配的 should 的语句的数量
- 多词 match 查询只是简单地将生成的 term 查询包裹在一个 bool 查询中，[等价的一些情况](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_how_match_uses_bool.html#_how_match_uses_bool)
- should 是用来微调分数的
  - [可以使用 boost 来更为精确的指定分数](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_boosting_query_clauses.html#_boosting_query_clauses)
- Todo：bool 是如何评分的

## 分析器

- 分析器会影响查询结果
- 可以在索引时和搜索时使用不同的分析器
- [Todo：分析器的配置、分析器的使用顺序、搜索的完整顺序](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_controlling_analysis.html)

# 多字段搜索

- 字面意思：查询语句涉及多个字段

- 用户输入了单个字符串查询、用户输入了多个字符串查询

- [用 boost 来提升优先级](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-query-strings.html#prioritising-clauses)

- 单字符串查询三种情形（type字段指定）（不同的处理方式/策略）

  - 最佳字段 best_fields：

    - 文档在 相同字段 中包含的词越多越好

  - 多数字段 most_fields

    - Todo：？？？？和最佳字段？
    - 和原词匹配程度越高越好？

  - 混合字段 cross_fields

    - Person： `first_name` 和 `last_name` （人：名和姓）

    - 如在一个大字段中进行搜索，这个大字段包括了所有列出的字段

    - 希望在 *任何* 这些列出的字段中找到尽可能多的词？？？？？

    - 与之前描述的 多字符串查询 很像，但这存在着巨大的区别。在 多字符串查询 中，我们为每个字段使用不同的字符串，在本例中，我们想使用 单个 字符串在多个字段中进行搜索

      即 相同的字符串在很多不同的字段搜索

    - `cross_fields` 类型首先分析查询字符串并生成一个词列表，然后它从所有字段中依次搜索每个词

    - 使用 cross_field 和 自定义 _all 字段

  - `cross_fields` 使用词中心式（term-centric）的查询方式，这与 `best_fields` 和 `most_fields` 使用字段中心式（field-centric）的查询方式非常不同

  - Todo： 还是不太懂？？？

  - [stackoverflow：elasticsearch what is the difference between best_field and most_field](https://stackoverflow.com/questions/35393793/elasticsearch-what-is-the-difference-between-best-field-and-most-field)

- [`multi_match` 查询为能在多个字段上反复执行相同查询提供了一种便捷方式](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-match-query.html#multi-match-query)

  - Fields 可以用模糊匹配（正则）、可以提升单个字段的权重
  - [可以参考这个来想下为什么会有 multi_match](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_cross_fields_entity_search.html#_%E7%AE%80%E5%8D%95%E7%9A%84%E6%96%B9%E5%BC%8F) 
  - 需要在 `multi_match` 查询中避免使用 `not_analyzed` 字段

## 最佳字段 best_fields

- 输入词组 “Brown fox” 然后点击搜索，有时候希望 两者同时出现

- 不使用 `bool` 查询，可以使用 `dis_max` 即分离 *最大化查询*

- 单的 `dis_max` 查询会采用单个最佳匹配字段，而忽略其他的匹配

  - 可能期望同时匹配 `title` 和 `body` 字段的文档比只与一个字段匹配的文档的相关度更高，但 dis_max 不能满足这样的情况

    [这时需要一个额外的参数 tie_breaker](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_tuning_best_fields_queries.html#_tie_breaker_%E5%8F%82%E6%95%B0)

- *将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回*

- dis_max 分值是最佳匹配字段产生的分值

- multi_match 默认情况下，查询以 best_fields 类型执行，它会为每个字段生成一个 match 查询，然后将这些查询包含在一个 dis_max 查询中。

```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "brown fox",
      "type":       "best_fields",
      "fields":     [ "subject", "message" ],
      "tie_breaker": 0.3
    }
  }
}
```

执行时就变成了：

```
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "subject": "brown fox" }},
        { "match": { "message": "brown fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

# 近似匹配

- Sue ate the alligator.
- The alligator ate Sue.
- Sue never goes anywhere without her alligator-skin purse.

用 `match` 搜索 `sue alligator` 上面的三个文档都会得到匹配，但它却不能确定这两个词是否只来自于一种语境，甚至都不能确定是否来自于同一个段落

---

- match_phrase 查询

  类似 `match` 查询， `match_phrase` 查询首先将查询字符串解析成一个词项列表，然后对这些词项进行搜索，但只保留那些包含 *全部* 搜索词项，且 *位置* 与搜索词项相同的文档

  - slop 参数

    也许我们想要包含 “quick brown fox” 的文档也能够匹配 “quick fox,”

    `slop` 参数告诉 `match_phrase` 查询词条相隔多远时仍然能将文档视为匹配

  - 多值字段使用 slop 可能因为相邻而被命中，如

    ```sense
    "names": [ "John Abraham", "Lincoln Smith"]
    ```

    运行一个对 `Abraham Lincoln` 的短语查询，结果会匹配到，虽然没有这个人

    - 可以在建立类型时，设置一个 position_increment_gap 参数

- 如果七个词条中有六个匹配， 那么这个文档对用户而言就已经足够相关了， 但是 `match_phrase` 查询可能会将它排除在外

  可以将 match_phrase 放入 bool 的should 中

- ```sense
  Sue ate the alligator
  多种索引方式
  ["sue", "ate", "the", "alligator"]
  ["sue ate", "ate the", "the alligator"]
  ["sue ate the", "ate the alligator"]
  ```

# 部分匹配

- todo：n-gram





