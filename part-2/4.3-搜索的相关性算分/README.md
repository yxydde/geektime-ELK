# 搜索的相关性算分

## 课程demo
```


PUT testscore
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text"
      }
    }
  }
}


PUT testscore/_bulk
{ "index": { "_id": 1 }}
{ "content":"we use Elasticsearch to power the search" }
{ "index": { "_id": 2 }}
{ "content":"we like elasticsearch" }
{ "index": { "_id": 3 }}
{ "content":"The scoring of documents is caculated by the scoring formula" }
{ "index": { "_id": 4 }}
{ "content":"you know, for search" }



POST /testscore/_search
{
  //"explain": true,
  "query": {
    "match": {
      "content":"you"
      //"content": "elasticsearch"
      //"content":"the"
      //"content": "the elasticsearch"
    }
  }
}

## boosting 查询包括 positive 、negative 和 negative_ boost 三个部分， positive 中的查询评分保持不变，
## negative 中的查询会降低文档评分， negative_boost 指明 negative 中降低的权值。
POST testscore/_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "content" : "elasticsearch"
                }
            },
            "negative" : {
                 "term" : {
                     "content" : "like"
                }
            },
            "negative_boost" : 0.2
        }
    }
}


POST tmdb/_search
{
  "_source": ["title","overview"],
  "query": {
    "more_like_this": {
      "fields": [
        "title^10","overview"
      ],
      "like": [{"_id":"14191"}],
      "min_term_freq": 1,
      "max_query_terms": 12
    }
  }
}



```

# 相关阅读
https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-boosting-query.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.5/index-modules-similarity.html
https://www.elastic.co/cn/blog/practical-bm25-part-1-how-shards-affect-relevance-scoring-in-elasticsearch
https://www.elastic.co/cn/blog/practical-bm25-part-2-the-bm25-algorithm-and-its-variables
https://www.elastic.co/cn/blog/practical-bm25-part-3-considerations-for-picking-b-and-k1-in-elasticsearch