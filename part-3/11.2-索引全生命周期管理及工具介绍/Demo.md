```
# 设置 1秒刷新1次，生产环境10分种刷新一次
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval":"1s"
  }
}

# 设置 Policy
PUT /_ilm/policy/log_ilm_policy
{
  "policy": {                       
    "phases": {
      "hot": {                      
        "actions": {
          "rollover": {             
            "max_docs": "5"
          }
        }
      },
      "delete": {
        "min_age": "30s",           
        "actions": {
          "delete": {}              
        }
      }
    }
  }
}



# 设置索引模版
PUT /_template/log_ilm_template
{
  "index_patterns" : [
      "ilm_index-*"
    ],
    "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "log_ilm_policy",
          "rollover_alias" : "ilm_alias"
        },
        "number_of_shards" : "1",
        "number_of_replicas" : "0"
      }
    }
}



#创建索引
PUT ilm_index-000001
{
  "aliases": {
    "ilm_alias": {
      "is_write_index": true
    }
  }
}

# 对 Alias写入文档
POST  ilm_alias/_doc
{
  "dfd":"dfdsf"
}

GET ilm_index-*/_ilm/explain
```

