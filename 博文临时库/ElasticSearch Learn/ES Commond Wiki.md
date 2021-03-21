# install

```
docker pull elasticsearch:6.8.2
docker pull kibana:6.8.2

docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d elasticsearch:6.8.2

docker exec -it elasticsearch /bin/bash
cd /usr/share/elasticsearch/config/
vi elasticsearch.yml

# 添加 已解决跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"

docker restart elasticsearch

# 插件

cd /usr/share/elasticsearch/plugins/
# ik
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.2/elasticsearch-analysis-ik-6.8.2.zip
# ik pinyin
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v6.8.2/elasticsearch-analysis-pinyin-6.8.2.zip
exit
docker restart elasticsearch 
```



# commond 

https://www.cnblogs.com/Rawls/p/10300639.html

https://segmentfault.com/a/1190000020140461

- 搜索单词

```
{
  "from": 0,
  "size": 5,
  "query": {
    "match": {
      "content": "源链接"
    }
  }
}
```

- 更改索引(向已有文档添加一个新的字段)

```
PUT is_docs_join_new/_mapping/cosmo
{
    "properties" : {
        "content_length" : {
            "type" : "long"
        }
    }
}

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

- minimum_should_match 控制精度

- 查找所有类型

```
{
  "aggs": {
    "all_type_search": {
      "terms": {
        "field": "type"
      }
    }
  }
}
```

