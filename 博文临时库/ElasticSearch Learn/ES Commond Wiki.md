``` sense
GET my_store/products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "price": 20
        }
      }
    }
  }
}
```

- Constant_score
- term



``` sense
GET my_store/products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "productID": "XHDK-A-1293-#fJ3"
        }
      }
    }
  }
}
```

- productID 需要精确查询，not_analyzed，未分析的



``` sense
GET my_store/_analyze
{
  "field": "productID",
  "text": "XHDK-A-1293-#fJ3"
}
```

- analyze



``` sense
PUT /my_store 
{
    "mappings" : {
        "products" : {
            "properties" : {
                "productID" : {
                    "type" : "keyword"
                }
            }
        }
    }
}
```

- 5.X以上版本没有string类型了，换成了text和keyword
- 现在版本index只能用true或者false，如想不被分词就把数据类型设置为keyword



``` sense
GET /my_store/products/_search
{
   "query" : {
      "bool" : { 
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

"bool" : {
    "should" : [
    { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, 
    { "bool" : { 
        "must" : [
        { "term" : {"productID" : "JODL-X-1937-#pV7"}}, 
        { "term" : {"price" : 30}} 
        ]
    }}
    ]
}
```

- 需要一个 bool 包裹
- filtered 过滤查询已被弃用，并在ES 5.0中删除。现在应该使用bool / must / filter查询
- bool 可以嵌套 bool



``` sense
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "QUICK!"
        }
    }
}

GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
            "minimum_should_match": "75%"
        }
    }
}

GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": {      
                "query":    "BROWN DOG!",
                "operator": "and"
            }
        }
    }
}
```

- 不去匹配 `brown OR dog` ，而通过匹配 `brown AND dog` 找到所有文档
- 三词项的示例中， `75%` 会自动被截断成 `66.6%` ，即三个里面两个词。无论这个值设置成什么，至少包含一个词项的文档才会被认为是匹配的



``` sense
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "quick" }},
      "must_not": { "match": { "title": "lazy"  }},
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 
    }
  }
}
```

- 返回 `title` 字段包含词项 `quick` 但不包含 `lazy` 的任意文档,一个文档不必包含 `brown` 或 `dog` 这两个词项，但如果一旦包含，我们就认为它们 *更相关*
- 注意和 bool 过滤的区别
- minimum_should_match 控制精度



``` sense
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 
                    }
                }}
            ]
        }
    }
}
```

- boost



以上例子都是单个字段的（只有一个是多字段的）（一条查询语句对应一个字段？）

---

``` sense
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```

- 每条语句贡献三分之一评分
- 多字段搜索
- [用 boost 来提升优先级](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-query-strings.html#prioritising-clauses)



``` 
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}

{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

- 这个查询是简单将每个字段的评分结果加在一起，由于bool的计算方式使得2反而分数低，但是在2中两个单词是连着的，是一个词组

``` SENSE
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

- 将 *最佳匹配* 字段的评分作为查询的整体评分，即如果是2，则不去加和后除而是直接使用body字段的分数



muti_match 还没有写过来例子

``` sense
GET my_index/my_type/_search
{
  "query": {
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", 
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" 
    }
  }
}
```

- type 字段
- 外面包裹了 query



``` sense
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] 
        }
    }
}
```

