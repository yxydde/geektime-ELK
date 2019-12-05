# Full text queries

The full text queries enable you to search [analyzed text fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/analysis.html) such as the body of an email. The query string is processed using the same analyzer that was applied to the field during indexing.

The queries in this group are:

## intervals query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-intervals-query.html

  A full text query that allows fine-grained control of the ordering and proximity of matching terms.

### Example request

The following `intervals` search returns documents containing `my favorite food` immediately followed by `hot water` or `cold porridge` in the `my_text` field.

This search would match a `my_text` value of `my favorite food is cold porridge` but not `when it's cold my favorite food is porridge`.

```
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "all_of" : {
          "ordered" : true,
          "intervals" : [
            {
              "match" : {
                "query" : "my favorite food",
                "max_gaps" : 0,
                "ordered" : true
              }
            },
            {
              "any_of" : {
                "intervals" : [
                  { "match" : { "query" : "hot water" } },
                  { "match" : { "query" : "cold porridge" } }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```

### Filter example

The following search includes a `filter` rule. It returns documents that have the words `hot` and `porridge` within 10 positions of each other, without the word `salty` in between:

```console
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "hot porridge",
          "max_gaps" : 10,
          "filter" : {
            "not_containing" : {
              "match" : {
                "query" : "salty"
              }
            }
          }
        }
      }
    }
  }
}
```



## match query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-match-query.html

The standard query for performing full text queries, including fuzzy matching and phrase or proximity queries.

执行标准的全文检索查询，包括模糊匹配和短语或近似匹配。

match query 会对查询语句进行分词，分词后查询语句中的任何一个词项被匹配，文档就
会被搜索到。如果想查询匹配所有关键词的文档，可以用and 操作符连接，命令如下：

```
GET /_search
{
    "query": {
        "match" : {
            "message" : {
                "query" : "this is a test"
            }
        }
    }
}

GET /_search
{
    "query": {
        "match" : {
            "message" : {
                "query" : "this is a test",
                "operator" : "and"
            }
        }
    }
}

GET /_search
{
    "query": {
        "match" : {
            "message" : "this is a test"
        }
    }
}

# Synonyms（同义词）
# The example above creates a boolean query:
# (ny OR (new AND york)) city
GET /_search
{
   "query": {
       "match" : {
           "message": {
               "query" : "ny city",
               "auto_generate_synonyms_phrase_query" : false
           }
       }
   }
}
```



## match_bool_prefix query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-match-bool-prefix-query.html

Creates a  bool query that matches each term as a  term query, except for the last term, which is matched as a  prefix query

match _phrase _prefix 和 match_phrase 类似，只不过 match_phrase _prefix 支持最后一个term 前缀匹配：

```console
GET /_search
{
    "query": {
        "match_bool_prefix" : {
            "message" : "quick brown f"
        }
    }
}
```

where analysis produces the terms `quick`, `brown`, and `f` is similar to the following `bool` query

```console
GET /_search
{
    "query": {
        "bool" : {
            "should": [
                { "term": { "message": "quick" }},
                { "term": { "message": "brown" }},
                { "prefix": { "message": "f"}}
            ]
        }
    }
}
```

## match_phrase query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-match-query-phrase.html

Like the  match query but used for matching exact phrases or word proximity matches.

match _phrase query 首先会把 query 内容分词，分词器可以自定义，同时文档还要满足以
下两个条件才会被搜索到：
(1). 分词后所有词项都要出现在该字段中。
(2). 字段中的词项顺序要一致。

```
GET /_search
{
    "query": {
        "match_phrase" : {
            "message" : {
                "query" : "this is a test",
                "analyzer" : "my_analyzer"
            }
        }
    }
}
```



## match_phrase_prefix query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-match-query-phrase-prefix.html

Like the  match_phrase query, but does a wildcard search on the final word.

The following search returns documents that contain phrases beginning with `quick brown f` in the `message` field.

This search would match a `message` value of `quick brown fox` or `two quick
brown ferrets` but not `the fox is quick and brown`.

```
GET /_search
{
    "query": {
        "match_phrase_prefix" : {
            "message" : {
                "query" : "quick brown f"
            }
        }
    }
}
```



## multi_match query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-multi-match-query.html

The `multi_match` query builds on the [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-match-query.html) to allow multi-field queries:

Types of `multi_match` query

The way the `multi_match` query is executed internally depends on the `type` parameter, which can be set to:

| `best_fields`   | (**default**) Finds documents which match any field, but uses the `_score` from the best field. See [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-multi-match-query.html#type-best-fields). |
| --------------- | ------------------------------------------------------------ |
| `most_fields`   | Finds documents which match any field and combines the `_score` from each field. See [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-multi-match-query.html#type-most-fields). |
| `cross_fields`  | Treats fields with the same `analyzer` as though they were one big field. Looks for each word in **any** field. See [`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-multi-match-query.html#type-cross-fields). |
| `phrase`        | Runs a `match_phrase` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-multi-match-query.html#type-phrase). |
| `phrase_prefix` | Runs a `match_phrase_prefix` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-multi-match-query.html#type-phrase). |
| `bool_prefix`   | Creates a `match_bool_prefix` query on each field and combines the `_score` from each field. See [`bool_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-multi-match-query.html#type-bool-prefix). |

```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}

# Query the title, first_name and last_name fields.
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] 
    }
  }
}

# The subject field is three times as important as the message field.
GET /_search
{
  "query": {
    "multi_match" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ] 
    }
  }
}

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

## query_string query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-query-string-query.html

Supports the compact Lucene [query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/query-dsl-query-string-query.html#query-string-syntax), allowing you to specify AND|OR|NOT conditions and multi-field search within a single query string. For expert users only.

query_string query 是与Lucene 查询语句的语法结合非常紧密的一种查询，允许在一个查询语句中使用多个特殊条件关键字（ 如： ANDIORINOT ）对多个字段进行查询，建议熟悉 Lucene 查询语法的用户去使用。

```
GET /_search
{
    "query": {
        "query_string" : {
            "query" : "(new york city) OR (big apple)",
            "default_field" : "content"
        }
    }
}
```



## simple_query_string query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-simple-query-string-query.html

A simpler, more robust version of the  query_string syntax suitable for exposing directly to users.

simple_ query_ string 是一种适合直接暴露给用户，并且具有非常完善的查询语法的查询语句，接受L ucene 查询语法，解析过程中发生错误不会抛出异常。

```
GET /_search
{
  "query": {
    "simple_query_string" : {
        "query": "\"fried eggs\" +(eggplant | potato) -frittata",
        "fields": ["title^5", "body"],
        "default_operator": "and"
    }
  }
}
```

