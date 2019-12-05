# Term-level queries

https://www.elastic.co/guide/en/elasticsearch/reference/7.4/term-level-queries.html

## exists query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/term-level-queries.html

查询会返回字段中至少有一个非空值的文档.

```
# 返回包含 user 字段的文档
GET /_search
{
    "query": {
        "exists": {
            "field": "user"
        }
    }
}

GET /_search
{
    "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "user"
                }
            }
        }
    }
}
```

## fuzzy query 

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-fuzzy-query.html

Returns documents that contain terms similar to the search term. Elasticsearch measures similarity, or fuzziness, using a [Levenshtein edit distance](http://en.wikipedia.org/wiki/Levenshtein_distance).

```
# 返回 user 字段中与 ki 值相近似的文档（模糊匹配）
GET /_search
{
    "query": {
        "fuzzy": {
            "user": {
                "value": "ki"
            }
        }
    }
}

GET /_search
{
    "query": {
        "fuzzy": {
            "user": {
                "value": "ki",
                "fuzziness": "AUTO",
                "max_expansions": 50,
                "prefix_length": 0,
                "transpositions": true,
                "rewrite": "constant_score"
            }
        }
    }
}
```

## ids query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-ids-query.html

Returns documents based on their [document IDs](https://www.elastic.co/guide/en/elasticsearch/reference/7.4/mapping-id-field.html).

This query uses document IDs stored in the [`_id`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/mapping-id-field.html) field.

```
# 返回 _id 为 "1", "4", "100" 的文档
GET /_search
{
    "query": {
        "ids" : {
            "values" : ["1", "4", "100"]
        }
    }
}
```

## prefix query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-prefix-query.html

Returns documents that contain a specific prefix in a provided field.

```
# 返回 user 字段中以 ki 开头的文档
GET /_search
{
    "query": {
        "prefix": {
            "user": {
                "value": "ki"
            }
        }
    }
}
```

## range  query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-range-query.html

Returns documents that contain terms within a provided range.

```
# 返回 age >= 10 && age <=20 的文档
GET _search
{
    "query": {
        "range" : {
            "age" : {
                "gte" : 10,
                "lte" : 20,
                "boost" : 2.0
            }
        }
    }
}

GET _search
{
    "query": {
        "range" : {
            "timestamp" : {
                "gte" : "now-1d/d",
                "lt" :  "now/d"
            }
        }
    }
}

# With a UTC offset of +01:00, Elasticsearch converts this date to 2014-12-31T23:00:00
GET _search
{
    "query": {
        "range" : {
            "timestamp" : {
                "time_zone": "+01:00", 
                "gte": "2015-01-01 00:00:00", 
                "lte": "now" 
            }
        }
    }
}
```

## regexp  query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-regexp-query.html

Returns documents that contain terms matching a [regular expression](https://en.wikipedia.org/wiki/Regular_expression).

```
# 返回 user 字段的值 以 k 开头 y 结尾，中间包含0到多个任意字符的文档
GET /_search
{
    "query": {
        "regexp": {
            "user": {
                "value": "k.*y",
                "flags" : "ALL",
                "max_determinized_states": 10000,
                "rewrite": "constant_score"
            }
        }
    }
}
```

## term  query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-term-query.html

Returns documents that contain an exact term in a provided field.

By default, Elasticsearch changes the values of `text` fields during analysis. For example, the default [standard analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-standard-analyzer.html) changes `text` field values as follows:

- Removes most punctuation (标点符号)
- Divides the remaining content into individual words, called [tokens](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/analysis-tokenizers.html)
- Lowercases the tokens

```
GET /_search
{
    "query": {
        "term": {
            "user": {
                "value": "Kimchy",
                "boost": 1.0
            }
        }
    }
}
```

## terms  query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-terms-query.html

Returns documents that contain one or more exact terms in a provided field.

To return a document, one or more terms must exactly match a field value, including whitespace and capitalization.

```
GET /_search
{
    "query" : {
        "terms" : {
            "user" : ["kimchy", "elasticsearch"],
            "boost" : 1.0
        }
    }
}
```

## terms_set query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-terms-set-query.html

Returns documents that contain a minimum number of exact terms in a provided field. You can define the minimum number of matching terms using a field or script.

```
PUT /job-candidates
{
    "mappings": {
        "properties": {
            "name": {
                "type": "keyword"
            },
            "programming_languages": {
                "type": "keyword"
            },
            "required_matches": {
                "type": "long"
            }
        }
    }
}

PUT /job-candidates/_doc/1?refresh
{
    "name": "Jane Smith",
    "programming_languages": ["c++", "java"],
    "required_matches": 2
}

PUT /job-candidates/_doc/2?refresh
{
    "name": "Jason Response",
    "programming_languages": ["java", "php"],
    "required_matches": 2
}

# 返回 programming_languages 字段至少包含 c++、java、php 中至少两项的文档
GET /job-candidates/_search
{
    "query": {
        "terms_set": {
            "programming_languages": {
                "terms": ["c++", "java", "php"],
                "minimum_should_match_field": "required_matches"
            }
        }
    }
}
```

## type query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-type-query.html

Returns documents of the specified type.

```
GET /_search
{
    "query": {
        "type" : {
            "value" : "_doc"
        }
    }
}
```



## wildcard query

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-wildcard-query.html

Returns documents that contain terms matching a wildcard pattern.

```
GET /_search
{
    "query": {
        "wildcard": {
            "user": {
                "value": "ki*y",
                "boost": 1.0,
                "rewrite": "constant_score"
            }
        }
    }
}
```

