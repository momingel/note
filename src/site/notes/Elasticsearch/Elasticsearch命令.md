---
{"dg-publish":true,"permalink":"/elasticsearch/elasticsearch/","title":"Elasticsearch","created":"2024-03-03T20:31:38.269+08:00","updated":"2024-03-07T11:07:19.467+08:00"}
---

# 集群健康状态

> 获取集群健康状态

``` shell
GET /_cat/health?v
```

> 获取节点信息

``` shell
GET /_cat/nodes?v
```

> 获取分片信息

``` shell
GET /_cat/shards?v
```

# 索引操作

> 创建索引

``` shell
PUT /index_name
```

> 获取索引信息

``` shell
GET /index_name
```

> 查看所有索引

``` shell
GET /_cat/indices?v
```

> 删除索引

``` shell
DELETE /index_name
```

# 文档操作

> 添加文档

``` shell
POST /index_name/_doc
{
  "field1": "value1",
  "field2": "value2"
}
```

> 获取文档

``` shell
GET /index_name/_doc/document_id
```

> 更新文档

``` shell
POST /index_name/_update/document_id
{
  "doc": {
    "field1": "new_value"
  }
}
```

> 删除文档

``` shell
DELETE /index_name/_doc/document_id
```

# 检索和搜索

> 执行简单查询

``` shell
GET /index_name/_search?q=query_string
```

> 执行查询DSL

``` shell
POST /index_name/_search
{
  "query": {
    "match": {
      "field": "value"
    }
  }
}
```

> 聚合查询

``` shell
POST /index_name/_search
{
  "aggs": {
    "agg_name": {
      "aggregation_type": {
        "field": "field_name"
      }
    }
  }
}
```

# 映射和设置

> 获取索引映射

``` shell
GET /index_name/_mapping
```

> 更新索引映射

``` shell
PUT /index_name/_mapping
{
  "properties": {
    "new_field": {
      "type": "text"
    }
  }
}
```

> 获取索引设置

``` shell
GET /index_name/_settings
```

> 更新索引设置

``` shell
PUT /index_name/_settings
{
  "number_of_replicas": 2
}
```

# 集群维护

> 关闭和打开索引

``` shell
POST /index_name/_close
POST /index_name/_open
```

> 强制合并索引分片

``` shell
POST /index_name/_forcemerge?max_num_segments=1
```

> 刷新索引

``` shell
POST /index_name/_refresh
```