# ES 常用操作
## 分词器
查看分词器
```
POST /_analyze
{
    "analyzer":"standard/ik_smart/ik_max_word",
    "text":This is test doc
}
```

## 索引相关
* 查看所有索引
```
   GET /_cat/indices
   GET /_cat/indices\?v 
```

* 查看指定索引
```
    GET _cat/indices/{index}
```

* 增加索引
```
    PUT test_index
    {
        "settings":{
            "number_of_shards":5, // 分片数量
            "number_of_replicas":1// 备份数量
        }
    }
```

* 删除索引
```
    DELETE {index}
```
*Note*
ES 修改字段类型，注意事项：  
1. 一旦创建了一个索引并且开始往里面添加数据，就不能够修改一个字段的类型
2. 如果想要修改一个字段的类型，必须创建一个新的索引，并且将数据从旧索引中迁移到新索引中
3. 更改字段类型的过程中需要考虑到索引中现有的字段类型，是否能够兼容新的数据类型，否则可能导致数据不一致或者查询失败的情况
总结:
在设计索引时，需要慎重考虑字段类型，并确保其能够满足所有需要的查询，如果需要更改字段类型，建议在索引中添加新的字段，并逐步迁移数据，以确保数据完整性和正确性

## 文档CRUD

* 新增文档
```
    PUT {index}/{type}/{id} 
    {
        JSON格式，文档原始数据
    }
```

* 获取文档
```
GET {index}/{type}/{id}
```

* 返回指定的列
```
GET {index}/{type}/{id}?_source=字段1，字段2...
```

* 只要内容（只想显示字段）不要元数据
```
GET {index}/{type}/{id}/_source
```

* 修改文档
1. 整体修改（全量修改的语法跟添加文档的语法一样，如果文档已经存在就是添加，否则就是修改）
   ```
    PUT {index}/{type}/{id}
    {
        "id":1,
        "name":"zs"
    }
   ```
   上面的修改会把ES中的数据全部覆盖，比如，如果之前的文档中有`age`字段，即，`age`字段会消失  
2. 局部修改  
   ```
   POST {index}/{type}/{id}/_update
   {
        "doc":{
            "id":1,
            "name":"zs"
        }
   }
   ```
    只会修改传入的字段，其他的文档中原来的字段不做任何改变

* 删除文档
    ```
    DELETE {index}/{type}/{id}
    ```

## 文档查询
1. 查询所有
   ```
    GET _search
   ```
2. 查询指定索引库
   ```
   GET {index}/_search
   ```
3. 查询指定类型
   ```
   GET {index}/{type}/_search
   ```
4. 查询指定文档
   ```
   GET {index}/{type}/{id}
   ```
5. 分页查询
   ```
   GET _search
   {
    "from":0,
    "size":2
   }
   ```
6. 字符串查询
   ```
   GET _search?q=age:17&size=2&from2&sort=id:desc&_source=id,name
   ```
7. 批量查询
   1. 不同索引库查询
        ```
            GET _mget
            {
                "docs" : [
                    {
                        "_index":"shopping",
                        "type":"goods",
                        "_id":2
                    },
                    {
                        "_index":"animal",
                        "type":"dog",
                        "_id":1,
                        "_source":"name,age"
                    }
                ]
            }
        ```
   2. 同索引库同类型-推荐
        ```
        GET {index}/{type}/_mget
        {
            "ids":["{id}","{id1}"]
        }
        ```

## DSL查询、DSL过滤
### DSL查询
1. 简单查询，可以使用字符串查询
2. 复杂查询，建议通过DSL使用JSON内容格式的请求体代替
   > **Note：**
   > 1. 字符串查询可以应付简单查询，面对复杂查询有点吃力，因此ES提供了DSL。  
   > 2. DSL查询（Query DSL），允许构建更加复杂，强大的查询，以JSON请求体的形式出现。由两部分组成DSL查询和DSL过滤。
   > 3. DSL(ES提供的 Domain Specific Language)，特定领域语言，以JSON的请求体形式出现。
   1. 查询语法
    ```
    // 一个相对完整的查询语法
    GET shopping/goods/_search
    {
        "query":{
            "match_all":{}
        },
        "from":20,
        "size":10,
        "_source":["username", "age", "id"],
        "sort":[{"join_date":"desc"},{"age","asc"}]
    }
    // match_all 表示查询所有数据， 分页处理，想要的列排序
    ```
    举例：
    ```
    GET shopping/goods/_search
    {
        "query":{
            "match":{// match 指的是标准查询，该查询方式会对查询的内容进行分词
                "name":"李子"
            }
        }
    }
    ```
### DSL过滤
1. 什么是DSL过滤，和DSL查询非常相似，但是他们的使用目的却不同
   1. DSL过滤查询文档的方式更像是对于我的条件‘有’或者‘没有’（等于，不等于）
   2. 而DSL查询语句则像是“有多像”（模糊查询）
2. DSL过滤与查询的区别
   1. 过滤结果可以缓存并应用到后续请求 ——>精确过滤后拿去模糊查询性能高
   2. 查询语句同时匹配文档，计算相关性，所以更耗时，且不缓存
   3. 过滤语句可以有效地配合查询语句完成文档过滤     

总结：需要模糊查询的使用DSL查询，需要精确查询的使用DSL过滤，在开发中组合使用（组合查询），关键字查询使用DSL查询，其他的都是用DSL过滤

### 查询方式
DSL查询 + 过滤语法
```json
GET animal/dog/_search
{
    "query": {
        "bool": {
            "must":[{ // 与（must）\或（should）\非（must not）
                "match":{
                    "name":"zs"
                }
            }],
            "filter":{
                "term":{
                    "name":"zs"
                }
            }
        }
    },
    "from":20,
    "size":10,
    "_source":["name","age","sex"],
    "sort":[{
        "join_date":"desc"
    },{
        "age":"asc"
    }]
}
```
> query:查询，所有查询条件在query里面  
> bool: 组合搜索bool可以组合查询条件为一个查询对象，这里包含了DSL查询和DSL过滤的条件  
> must:必须匹配，与（must）\或（should）\非（must not）  
> match:分词匹配查询，会对查询条件分词，multi_match:多字段匹配  
> filter:过滤条件  
> term:词元查询，不会对查询条件分词  
> from,size:分页  
> _source:查询结果中需要哪些列  
> sort:排序  

1. 全匹配（match_all）
> 匹配所有文档
```
GET _search
{
    "query":{
        "bool":{
            "must":[{
                "match_all":{}
            }],
            "filter":{
                "term":{
                    "name":"zs1"
                }
            }
        }
    }
}
```

1. 标准查询（match和multi_match）
标准查询，可以理解为分词查询，有点像模糊匹配`like`，会对查询的内容进行分词后，得到多个单词，分别带着多个单词去检索ES库，只要有一个单词能查出结果，整个查询就有结果。不管你需要全文本查询还是精确查询，都需要它
如下面的搜索对`斥候`分词，并找到包含`斥`或`候`的文档，然后给出排序分值  
```
GET _search
{
    "query":{
        "match":{
            "fieldName":"斥候"
        }
    }
}
// 上面的查询效果如同 `where filedName like %斥% or filedName like %候%`
// tips: match一般只用于全文字段的匹配和查询，一般不用于过滤
```
multi_match 查询允许做match查询基础上同时搜索多个字段：
```
GET _search
{
    "query":{
        "multi_match":{
            "query":"斥候",
            "fields":["filedName", "title"]
        }
    }
}
// 这个搜索同时在filedName和title字段中匹配
```

1. 单词搜索和过滤（Term和Terms）
单词/词元查询，可以理解为等值查询，字符串，数字等都可以使用它，把查询的内容看作一个整体去检索ES库
```
GET _search
{
    "query":{
        "bool":{
            "must":{
                "match_all":{}
            },
            "filter":{
                "term":{
                    "fieldName":"斥候"
                }
            }
        }
    }
}
```
”斥候“会被当做一个整体去`term`中匹配，它跟`match`不同的地方在于`match`会把”斥候“拆分成”斥“和”候“分别去`fieldName`中查询

Terms支持多个字段查询
```
GET _search
{
    "query":{
        "bool":{
            "must":{
               "match_all":{} 
            },
            "filter":{
                "terms":{
                    "fieldName":"斥候"
                }
            }
        }
    }
} 
```

4. 组合条件搜索过滤（Bool）
```json
GET _search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "hobby": "睡"
                    }
                }
            ],
            "should": [
                {
                    "term": {
                        "hobby": "游戏"
                    }
                }，
				{
                    "term": {
                        "hobby": "运动"
                    }
                }
            ],
            "must_not": [
                {
                    "range": {
                        "birth_date": {
                            "lt": "1990-06-30"
                        }
                    }
                }
            ],
            "filter": [ 
				......
            ]
        }
    }
}
// 上面案例如同：hobby=睡 and （hobby=游戏 or hobby=运动) and birth_date >= 1990-06-30
// 提示： 如果 bool 查询下没有must子句，那至少应该有一个should子句。但是 如果有must子句，那么没有should子句也可以进行查询。
//
```
5. 范围查询与过滤
```json
GET _search
{
    "query":{
        "range":{
            "age":{
                "gte":20,
                "lt":30
            }
        }
    }
}
// 查询年龄>=20并且<30
```
> `gt > gte >= lt < lte <=` 

6. 存在和缺失过滤器(exists和missing)
```json
GET _search
{
    "query":{
        "bool":{
            "must":[{
                "match_all":{}
            }],
            "filter":{
                "exists"{
                    "field":"gps"
                }
            }
        }
    }
}
// 通过exits/missing，判断某字段是否存在/缺失
```
7. 前匹配搜索与过滤（prefix）
> 和term查询相似，前匹配搜索不是精确匹配，而是类似于SQL中的like 'key%'
```json
GET _search
{
    "query":{
        "perfix":{
            "name":"王"
        }
    }
}
// 查询所有姓王的用户
```

8. 通配符搜索
```json
GET _search
{
    "query":{
        "wildcard":{
            "name":"嘻*嘻"
        }
    }
}
// 使用 `*` 代表0~n个，使用?代表1个
```


## 文档映射（mapping）
### 基本概念
1. 什么是文档映射
   1. ES的文档映射（mapping）机制，用于字段类型确认，将每个字段匹配为一种确定的数据类型；就如MySQL表中为每个column设置类型。为了方便检索，我们会指定存储在ES中的字段是否进行分词，但是有些字段类型可以分词，有些不可以，多以对于字段的类型需要我们自己去指定。
   note: 
    类似于我们使用MySQL数据库的过程：  
    建库->建表（指定字段类型）->操作数据  
    ES中也有类似的流程：  
    ES创建索引库-文档类型映射->操作文档  

2. 默认字段类型
查看索引类型的映射配置
```
GET {index}/_mapping/{type}
```
ES基本字段类型  
* 字符串类型：`text keyword`(string 类型在es5之后不再支持)
* 整数类型：` integer long short byte `
* 浮点类型：` double float half_float scaled_float`
* 逻辑类型：`boolean`
* 日期类型：`date date_nanos`
* 范围类型：`range`
ES复杂字段类型(复合类型)
* 二进制类型：`binary`
* 数组类型：`array`
* 对象类型：`object`
* 嵌套类型：`nested`
* 地理坐标类型：`geo_point`
* 地理图标类型：`geo_shape`
* IP类型：`ip`
* 范围类型：`completion`
* 令牌计数类型：`token_count`
* 附件类型：`attachment`
* 抽取类型：`percolator`

默认映射  
ES在没有配置Mapping的情况下新增文档，ES会对字段类型进行猜测，并且动态生成字段和类型的映射关系  
|  内容                            | 默认映射类型|
|  :----                           | :----:     |
| JSON type                        | Field Type |
| Boolean: true or false           | "boolean"  |
| Whole number: 123                | "long"     |
| Floating point: 123.1            | "double"   |
| String, valid date: "2023-4-25"  | "date"     |
| String: "foo bar"                | "string"   |

3. 映射规则
[映射规则](https://blog.csdn.net/qq_45989156/article/details/107826752)

### 添加映射
1. 添加索引
```
PUT shopping
```
2. 单类型创建映射
给`shopping`索引库中的`goods`类型创建映射，`id`指定为`long`类型，`name`指定为`text`类型（要分词），analyzer分词使用`ik`，查询分词器也是用`ik`
```json
PUT shopping/goods/_mapping
{
    "goods":{
        "properties":{
            "id" :{
                "type":"long"
            },
            "name":{
                "type":"text",
                "analyzer":"ik_smart",
                "search_analyzer":"ik_smart"
            }
        }
    }
}
```
3. 多类型创建映射
```json
PUT shopping
{
   "goods":{
        "properties":{
            "id" :{
                "type":"long"
            },
            "name":{
                "type":"text",
                "analyzer":"ik_smart",
                "search_analyzer":"ik_smart"
            }
        }
    },
    "dept":{
        "properties":{
            "id" :{
                "type":"integer"
            },
            "name":{
                "type":"text",
                "analyzer":"ik_smart",
                "search_analyzer":"ik_smart"
            }
        }
    }
}
```
同时给user和dept创建文档映射  

4. 对象映射
```JSON
{
    "id":1,
    "girl"::{
        "name":"催化",
        "age":22
    }
}
```
文档映射  
```JSON
{
    "properties":{
        "id" :{
            "type":"long"
        },
        "girl":{
            "properties":{
                "name": {
                    "type":"keyword"
                },
                "age": {
                    "type":"integer"
                }
            }
        }
    }
}
```
5. 数组映射  
```json
{
    "id":1,
    "hobby":["女","篮球"]
}
```
文档映射
```json
{
    "properties":{
        "id":{
            "type":"long"
        },
        "hobby":{
            "type":"keyword"
        }
    }
}
```
// 数组中的映射只需要一个元素即可，因为数组中的元素类型是一样的

6. 对象数组
```json
{
    "id":1,
    "person":[{"name":"张三","age":18},{"name":"李四","age":8}]
}
```

文档映射
```json
{
    "properties"{
        "id":{
            "type":"long"
        },
        "girl":{
            "properties":{
                "age":{
                    "type":"long"
                },
                "name":{
                    "type":"text"
                }
            }
        }
    }
}
```

7. 全局映射
索引库中多个类型（type,类似MySQL中的表）的字段是有相同的映射，如所有的ID都可以指定为integer类型，基于这种思想，我们可以做全局映射，让所有的文档都使用全局文档映射。全局映射可以通过*动态模板*和*默认设置*两种方式实现





## version
ES迭代策略激进，这里记录一些比较重要的新增和删除feature
1. ES7开始，Type概念将逐渐被淘汰
2. ES5已经移除了string类型









