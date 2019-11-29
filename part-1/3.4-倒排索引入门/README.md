# 倒排索引入门
##课程Demo
```
POST _analyze
{
  "analyzer": "standard",
  "text": "Mastering Elasticsearch"
}

POST _analyze
{
  "analyzer": "standard",
  "text": "Elasticsearch Server"
}

POST _analyze
{
  "analyzer": "standard",
  "text": "Elasticsearch Essentials"
}

# bin/elasticsearch-plugin install analysis-icu

POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "这个苹果不大，好吃"
}
```
## 相关阅读
- https://zh.wikipedia.org/wiki/%E5%80%92%E6%8E%92%E7%B4%A2%E5%BC%95
- https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html
- Analyze API https://www.elastic.co/guide/en/elasticsearch/reference/7.4/indices-analyze.html
- Analyzer 构成 https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analyzer-anatomy.html
- Built-in Analyzers https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-analyzers.html
- Built-in Tokenizers https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis-tokenizers.html
