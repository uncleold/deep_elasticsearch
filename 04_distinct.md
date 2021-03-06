# 1、新建库表
```
PUT books
{
  "mappings": {
    "_doc": {
      "properties": {
        "title": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        }
      }
    }
  }
}
```
# 2、导入数据
```
PUT books/_doc/2
{
  "title":"Hadoop权 指南(第4版)+数据算法"
}

PUT books/_doc/3
{
  "title":"Spark与Hadoop大数据分析"
}

PUT books/_doc/4
{
  "title":"Hadoop + Spark 大数据巨量分析与机器"
}

PUT books/_doc/5
{
  "title":"Hadoop大数据开发基础 "
}

PUT books/_doc/1
{
  "title":"hadoop高级数据分析"
}

PUT books/_doc/6
{
  "title":"hadoop高级数据分析"
}
```

# 3、全量检索
```
GET books/_search
{
  "query": {
    "match_all": {}
  },
  "sort":{
    "_doc":"desc"
  }
}
```

# 4、折叠检索（类似去重效果）
```
GET books/_search
{
  "query": {
   "match_all":{}
  },
  "collapse": {
    "field": "title.keyword"
  }
}
```

# 5、统计去重后的计数
```
GET books/_search
{
   "size":0,
    "aggs" : {
    "books_count" : {
        "cardinality" : {
          "field" : "title.keyword"
          }
        }
    }
}
```

# 6、全量检索结果
```
GET books/_count
{
  "query":{
    "match_all":{}
  }
}
```

# 7、去重后的检索结果（比collapse重叠复杂）
```
GET books/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "type": {
      "terms": {
        "field": "title.keyword",
        "size": 10
      },
      "aggs": {
        "title_top": {
          "top_hits": {
            "_source": {
              "includes": ["title"]
            },
            "sort": [
              {
                "title.keyword": {
                  "order": "desc"
                }
              }
            ],
            "size":1
          }
        }
      }
    }
  },
  "size": 0
}
```

# 7、报错提示
```
{
  "error": {
    "root_cause": [
      {
        "type": "search_context_exception",
        "reason": "unknown type for collapse field `title`, only keywords and numbers are accepted"
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "books",
        "node": "ikKuXkFvRc-qFCqG99smGg",
        "reason": {
          "type": "search_context_exception",
          "reason": "unknown type for collapse field `title`, only keywords and numbers are accepted"
        }
      }
    ]
  },
  "status": 500
}
```
