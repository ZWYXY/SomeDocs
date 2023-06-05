ES基本操作
# 索引类
创建索引
```
PUT shopping
{
    "settings":{
        "number_of_shards":5,
        "number_of_replicas":1
    }
}
```

5个master shards分片 每个 master shards分片 有一个Replica Shard

查询所有索引
```
GET _cat/indices?v
```

查询指定索引
```
GET  _cat/indices/{index}
```

删除索引
```
DELETE {index}
```

修改索引
么得



# 文档类
指定ID创建索引文档
```
PUT shopping/goods/1
{
    "id":1,
    "name":"xixi"
}
```
## 文档查询类
_id查询文档

```
GET {index}/_doc/{id}
GET oss-call-phone-record-2023-05-22/_doc/wjuMVYgB61Mq4quA-iAn
```
```
GET oss-call-phone-record-2023-05-22/_search
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
```
GET oss-call-phone-record-2023-05-22/_search
{
  "query": {
    "terms": {
      "_id": ["wjuMVYgB61Mq4quA-iAn","wTueU4gB61Mq4quA8yCB"]
    }
  }
}
```
```
GET oss-call-phone-record-2023-05-22/_search
{
  "query": {
    "match": {
      "_id": "wjuMVYgB61Mq4quA-iAn"
    }
  }
}
```
### 

指定返回的列
```
GET {index}/{type}/{id}?_source=字段1，字段2
```

只要内容（只想显示字段，不要元数据）
GET {index}/{type}/{id}?_source

修改文档
整体修改
PUT {index}/{type}/{id}
```
{
    "id":1,
    "name":"xx"
}
```
修改过程，1.标记删除旧的文档 2添加新文档

局部修改
POST {index}/{type}/{id}
```
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







