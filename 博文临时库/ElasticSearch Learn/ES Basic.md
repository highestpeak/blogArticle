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