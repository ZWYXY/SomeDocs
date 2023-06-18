# ES基本操作
## 查询
### 查询所有
GET _search
```json
{
  "query":{
    "match_all":{}
  }
}
```
### 查询指定索引所有
GET /{index}/_search
```json
{
  "query":{
    "match_all":{}
  }
}
```

### match查询
GET /{index}/_search
```json
{
  "query":{
    "match":{
      "{fieldName}":"{value}"
    }
  }
}
```
1. `match`查询用于执行全文搜索，根据查询字符串与字段内容的匹配程度进行评分
2. 它会对搜索词进行分词处理，然后根据分词后的结果与字段内容进行匹配
3. 默认情况下，它会根据相关性对文档进行排序，将最佳匹配的文档排在前面
4. 适合于全文搜索场景，如文章内容，产品描述

### Term查询
GET /{indexName}/_search
```json
{
  "query":{
    "term":{
      "{fieldName}":"{value}"
    }
  }
}
```
1. `term`查询用于精确匹配某个字段的值
2. 它会查找字段中完全匹配指定值的文档
3. 不会对搜索词进行分词处理，而是直接将搜索词与字段值进行比较
4. 适合于关键字字段，如ID，标签等


### match_phrase
可以用于匹配短语，保留原始短语的顺序，更加精确的匹配短语，类似于match查询，但会考虑短语顺序和位置

### Range查询
GET /{index}/_search
```json
{
  "query":{
    "range":{
      "{fieldName}":{
        "gte": "{value}",
        "lt": "{value}"
      }
    }
  }
}
```
举例：
```json
{
  "query":{
    "range":{
      "call_id":{
        "gte": 1,
        "lt": 7
      }
    }
  }
}
```

### bool查询
GET /{index}/_search
```json
{
  "query":{
    "bool":{
      "must":[
        {"match": {"{fieldName}":"{value}"}},
        {"match": {"{fieldName}":"{value}"}}
      ],
      "must_not":[
        {"term": {"{fieldName}":"{value}"}}
      ]
    }
  }
}
```

### 聚合查询
GET /{index}/_search
```json
{
  "aggs":{
    "duration_stats":{
      "stats":{ // stats || min || max || avg || sum 常见聚合操作
        "field":"{filedName}"
      }
    }
  }
}
```

# 文档类
### 新增文档
POST /{index}/_doc/{docId}(可选)
```json
{
  "username":"123",
  "age":"3"
}
```

# 索引类

### 查看所有索引
GET /_cat/indices

### 查询指定索引
GET /_cat/indices/{index}

### 查看索引模板
GET /_template/{index}

### 索引映射关系查看
GET /{index}/_mapping

### 为索引添加别名
PUT /{index}/_alias/{aliasName}

### 查询某个别名映射的所有索引
GET /*/_alias/{indexAliasName}

### 查询索引的所有别名
GET /{index}/_alias/*

### 为指定索引移除指定的别名和添加别名
POST /_aliases
```json
{
  "actions": [
    {
      "remove": {
        "index": "{indexName}",
        "alias": "{aliasName}"
      }
    },
    {
      "add": {
        "index": "{indexName}",
        "alias": "{aliasName}"
      }
    }
  ]
}
```

### 索引reindex
索引在建立后有些映射属性无法修改，比如字段类型等，但是可以修改字段的分析器或索引选项等
索引在修改后需要reindex，实现数据迁移
POST _reindex
```json
{
  "source": {
    "index": "source_index"
  },
  "dest": {
    "index": "destination_index"
  }
}
```
上面的示例中，将 source_index 替换为源索引的名称，destination_index 替换为目标索引的名称。执行这个请求后，Elasticsearch 会将源索引中的数据复制到目标索引中。
因此索引修改，比较麻烦，所以一般不提供索引删除操作

### 创建索引
PUT shopping
```json
{
    "settings":{
        "number_of_shards":5,
        "number_of_replicas":1
    }
}
```
5个master shards分片 每个 master shards分片 有一个Replica Shard

### 删除索引
DELETE {index}

### 修改索引
么得

# 文档类
### 指定ID创建索引文档
```json
PUT shopping/goods/1
{
    "id":1,
    "name":"xixi"
}
```
## 文档查询类
### _id查询文档

GET {index}/_doc/{id}
GET oss-call-phone-record-2023-05-22/_doc/wjuMVYgB61Mq4quA-iAn
GET oss-call-phone-record-2023-05-22/_search
```json
{
  "query": {
    "ids": {
      "values": ["wjuMVYgB61Mq4quA-iAn","wTueU4gB61Mq4quA8yCB"]
    }
  }
}
```
```
GET oss-call-phone-record-2023-05-22/_search
{
  "query": {
    "term": {
      "_id": {
        "value": "wjuMVYgB61Mq4quA-iAn"
      }
    }
  }
}
```

GET {index}/_search
```json 
{
  "query": {
    "terms": {
      "_id": ["wjuMVYgB61Mq4quA-iAn","wTueU4gB61Mq4quA8yCB"]
    }
  }
}
```
GET {index}/_search
```json
{
  "query": {
    "match": {
      "_id": "wjuMVYgB61Mq4quA-iAn"
    }
  }
}
```
### 

### 指定返回的列
GET {index}/{type}/{id}?_source=字段1，字段2


### 查询文档只要内容（只想显示字段，不要元数据）
GET {index}/{type}/{id}?_source

## 修改文档
### 整体修改
PUT {index}/{type}/{id}
```json
{
    "id":1,
    "name":"xx"
}
```
修改过程，1.标记删除旧的文档 2添加新文档

### 局部修改
POST {index}/{type}/{id}
```json
{
    "doc":{
        "id":1,
        "name":"xx"
    }
}
```

修改过程，1.检索旧文档 2. 修改文档 3. 标记删除旧文档 4 添加新文档


# ES 安装
ES
`docker run -d --name es --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms256m -Xmx256m" elasticsearch:7.17.7`
Kibana
`docker run -d --name kibana --net elastic -p 5601:5601 -e ELASTICSEARCH_HOSTS=http://es:9200 kibana:7.17.7`
msyql
`docker run -d --name mysql -p 10086:3306 -e MYSQL_ROOT_PASSWORD=1142165668  mysql:latest`







