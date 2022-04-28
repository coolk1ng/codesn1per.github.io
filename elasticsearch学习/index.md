# ElasticSearch学习


### 1. 什么是ElasticSearch

ElasticSearch简称es,是一个开源的分布式全文检索引擎,他可以近乎实时的存储,检索数据;本身拓展性很好,可以拓展到上百台服务器,处理PB级别的数据.es也使用java开发并使用lucene作为核心来实现所有的所有和搜索的功能,但是他的目的是通过简单的Restful API来隐藏Lucene的复杂性,从而让全文搜索变的简单.

### 2. ElasticSearch概念

##### 2.1 概述

ElasticSearch是面向文档(document oriented)的,这意味着他可以存储整个对象或文档.然而他不仅是存储,还会索引(index)每个文档的内容使之可以被搜索.在ElasticSearch中,你可以对文档(而非成行成列的数据)进行索引,搜索,排序,过滤.与关系型数据库对比如下:

```
Relational DB -> Databases -> Tables - > Rows -> COlumns
ElasticSearch - > Indices -> Types -> Documents -> Fields
```

##### 2.2 索引-index

一个索引就是一个拥有积分相似特征的文档的集合.比如说,你可以有一个客户数据的索引,另一个产品目录的索引,还有一个订单数据的索引.一个索引由一个名字来标识(必须全部是小写字母的),并且当我们要对对应于这个索引中的文档进行索引,搜索,更新和删除的时候,都要使用到这个名字.在一个集群中,可以定义人一多的索引.

##### 2.3 类型-type

在一个索引中,你可以定义一种或多种类型,一个类型是你的索引的一个逻辑上的分类/分区,其寓意完全由你来定.通常,会为具有一组共同字段的的文档定义一个类型.比如,我们假设你运营一个博客并且将你所有的数据存储到一个索引中.在这个索引中,你可以为用户数据定义一个类型,为博客数据定义另一个类型,当然,也可以为评论数据定义另一个类型.

##### 2.4 字段-Field

相当于是数据表的字段.对文档数据根据不同属性进行的分类标识

##### 2.5 映射-mapping

mapping是处理数据的方式和规则方面做一些限制,如某个字段的数据类型,默认值,分析器,是否被索引等,这些都是映射里面可以设置的,其他就是处理es里面的数据一些使用葛泽设置也叫做映射.俺这最优规则处理数据对性能提升很大.

##### 2.6 文档-doucument

一个文档是一个可被索引的基础信息单元,在一个index/type里面,你可以存储人一多的文档.注意,尽管一个文档,物理上存在于一个索引之中,文档必须被索引/赋予一个索引的type.

##### 2.7 接近实时-NRT

es是一个接近实时的搜索平台.这意味着,从索引一个文档直到这个文档能够被搜索到有一个轻微的延迟.

### 3. 索引操作

##### 3.1 创建索引

postman发送put请求 127.0.0.1:9200/shopping

![image-20220427094453914](/image-20220427094453914.png)

##### 3.2 查看索引

postman发送get请求 127.0.0.1:9200/shopping

查看所有索引: 127.0.0.1:9200/_cat/indices?v

### 4. 文档操作

##### 4.1 创建文档

postman发送post请求 127.0.0.1:9200/shopping/_doc

```json
{
    "title":"小米手机",
    "category":"手机",
    "price":4999
}
```

##### 4.2 查询文档

根据主键查询: 127.0.0.1:9200/shoping/_doc/id

查询所有文档数据: 127.0.0.1:9200/shoping/_search

##### 4.3 修改文档数据

1. 全量数据更新: 127.0.0.1:9200/shoping/_doc/id (put请求)

```json
{
    "title":"小米手机",
    "category":"手机",
    "price":1999
}
```

2. 局部更新: 127.0.0.1:9200/shoping/_update/id (post请求)

   ```json
   {
       "doc":{
           "title":"苹果手机"
       }
   }
   ```

   

##### 4.4 删除文档数据

127.0.0.1:9200/shoping/_doc/id (delete请求)

### 5. 查询文档数据

##### 5.1 条件查询

127.0.0.1:9200/shoping/_search (get请求)

条件:

```json
{
  "query":{
    "match":{ //match_all 所有
      "category":"手机"
    }
  }
}
```

结果:

![image-20220427101621974](/image-20220427101621974.png)

##### 5.2 分页查询

条件:

```json
{
  "query":{
    "match":{ //match_all 所有
      "category":"手机"
    }
  },
  "from":0
  "size":1
}
```

结果:

![image-20220427101840883](/image-20220427101840883.png)

##### 5.3 查询指定字段

条件:

```json
{
    "query":{
        "match":{
            "category":"手机"
        }
    },
    "from":0,
    "size":1,
    "_source":["title"]
}
```



结果:

![image-20220427102042367](/image-20220427102042367.png)

##### 5.4 查询排序

条件:

```json
{
    "query":{
        "match":{
            "category":"手机"
        }
    },
    "_source":["title","price"],
    "sort":{
        "price":{
            "order":"desc"
        }
    }
}	
```

结果:

![image-20220427102407199](/image-20220427102407199.png)

##### 5.5 多条件查询

条件:

```json
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "category":"手机"
                    }
                },
                {
                    "match":{
                        "price":1999
                    }
                }
            ]
        }
    }
}
```

结果:

![image-20220427103020809](/image-20220427103020809.png)

##### 5.6 范围查询

条件:

```json
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "category":"手机"
                    }
                }
            ],
            "filter":{
            "range":{
                "price":{
                    "gt":3000
                }
            }
        }
        }
    }
}
```

结果:

![image-20220427103501202](/image-20220427103501202.png)

##### 5.7 完全匹配

条件:

```json
{
    "query":{
      "match_phrase":{
          "title":"苹果手机"
      }
    }
}
```

结果:

![image-20220428011736585](/image-20220428011736585.png)

##### 5.8 高亮显示

条件:

```json
{
    "query":{
      "match_phrase":{
          "title":"苹果手机"
      }
    },
    "highlight":{
        "fields":{
            "title":{}
        }
    }
}
```

结果:

![image-20220428012438049](/image-20220428012438049.png)

##### 5.9 聚合查询

**分组:**

条件:

```json
{
    "aggs":{ //聚合操作
        "price_group":{ //随意起名
            "terms":{ //分组
                "field":"price" //分组字段
            }
        }
    }
}
```

结果:

![image-20220428013120474](/image-20220428013120474.png)



(无原始数据)条件:

```json
{
    "aggs":{ //聚合操作
        "price_group":{ //随意起名
            "terms":{ //分组
                "field":"price" //分组字段
            }
        }
    },
    "size":0
}
```

结果:

![image-20220428013258717](/image-20220428013258717.png)

**平均值:**

条件:

```json
{
    "aggs":{ //聚合操作
        "price_avg":{ //随意起名
            "avg":{ //分组
                "field":"price" //分组字段
            }
        }
    },
    "size":0
}
```

结果:

![image-20220428014118889](/image-20220428014118889.png)

### 6. 映射关系

##### 6.1 创建mapping

1. 创建index: user

2. 创建mapping: 127.0.0.1:9200/user/_mapping (put请求)

   条件:

   ```json
   {
       "properties":{
           "name":{
               "type":"text",
               "index":true
           },
           "sex":{
               "type":"keyword",
               "index":true
           },
           "age":{
               "type":"keyword",
               "index":false
           }
       }
   }
   ```

##### 6.2 查看mapping

127.0.0.1:9200/user/_mapping (get请求)


