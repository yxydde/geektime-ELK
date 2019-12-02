# Request Body 与 Query DSL
## 课程Demo
- 需要通过 Kibana 导入Sample Data的电商数据。具体参考“2.2节-Kibana的安装与界面快速浏览”
- 需导入Movie测试数据，具体参考“2.4-Logstash安装与导入数据”
- Both HTTP GET and HTTP POST can be used to execute search with body. Since not all clients support GET with body, POST is allowed as well.

```
# Fast check for any matching docs
# 如果我们只是想知道满足查询条件的文档数量,我们可以设置大小为0,表明我们的搜索结果不感兴趣。
# 我们也可以设置 terminate_after=1 表明每个shard有第一个文档被匹配到时，查询就可以终止了。
# 如果我设置了 terminate_after ，则响应结果中的 terminated_early 会被置为true
GET /kibana_sample_data_ecommerce/_search?q=currency:EUR&size=0&terminate_after=1


#ignore_unavailable=true，可以忽略尝试访问不存在的索引“404_idx”导致的报错
#查询movies分页
POST /movies,404_idx/_search?ignore_unavailable=true
{
  "profile": true,
	"query": {
		"match_all": {}
	}
}

POST /kibana_sample_data_ecommerce/_search
{
  "from":10,
  "size":20,
  "query":{
    "match_all": {}
  }
}


#对日期排序
POST kibana_sample_data_ecommerce/_search
{
  "sort":[{"order_date":"desc"}],
  "query":{
    "match_all": {}
  }

}

#source filtering
POST kibana_sample_data_ecommerce/_search
{
  "_source":["order_date"],
  "query":{
    "match_all": {}
  }
}


#脚本字段
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_field": {
      "script": {
        "lang": "painless",
        "source": "doc['order_date'].value+'hello'"
      }
    }
  },
  "query": {
    "match_all": {}
  }
}


POST movies/_search
{
  "query": {
    "match": {
      "title": "last christmas"
    }
  }
}

POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "last christmas",
        "operator": "and"
      }
    }
  }
}

POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love"

      }
    }
  }
}

POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love",
        "slop": 1

      }
    }
  }
}

```


## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/7.4/search-request-body.html
- https://www.elastic.co/guide/en/elasticsearch/reference/7.4/query-dsl.html
