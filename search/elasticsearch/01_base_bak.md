
参考地址：  
https://es.xiaoleilu.com/054_Query_DSL/55_Request_body_search.html. 
https://www.elastic.co/guide/cn/elasticsearch/guide/current/bool-query.html  


什么是elasticsearch
----------------
+ 实时搜索和分析引擎
+ 可以用来做全文检索、结构化数据检索、聚合分析
+ 分布式，可以支技到数百节点，pb级别数据


核心概念
----------------
+ 索引(index)：一堆文档的集合，文档的物理组织方式
+ 分片(shard)：一个索引按照 id哈希分成多个shard，分配到多个物理机上
+ 文档(doc)：用json表示文档
+ 字段(field)：文档中的每个字段叫一个field，类似db中的column
+ 类型(type)：索引的逻辑分区



elk
----------------
是非常流行的一个用来收集和分析日志的架构
+ logstash：etl组件，把数据抽取并写入es
+ elasticsearch：分布式的搜索引擎
+ kibana：查询es并做数据可视化



使用场景
----------------
全文检索
日志分析（elk）



```
关系数据库       ⇒ 数据库         ⇒ 表          ⇒ 行             ⇒ 列(Columns)
Elasticsearch   ⇒ 索引(Index)   ⇒ 类型(type)  ⇒ 文档(Docments)  ⇒ 字段(Fields)  
```


向Elasticsearch发出的请求的组成部分与其它普通的HTTP请求是一样的：
------------
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```
举例说明，为了计算集群中的文档数量，我们可以这样做：
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```


3.使用Query DSL搜索
============
DSL (Domain Specific Language 领域特定语言) 需要使用 JSON 作为主体


服务是否正常
------------
```
curl localhost:9200

返回
{
  "name" : "atntrTf",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "tf9250XhQ6ee4h7YI11anA",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```


查看当前节点的所有 Index
------------
```
curl -X GET 'http://localhost:9200/_cat/indices?v'

返回
health status index                     pri rep docs.count docs.deleted store.size pri.store.size
green  open   ccccccccccccc               5   1          0            0      1.2kb           710b
green  open   xxxxxxxxxxxxxxxxxxxxxxxxx   5   1         10            0    417.1kb        206.1kb
 17gb            8gb
```




列出每个 Index 所包含的 Type
--------------
注：根据规划，Elastic 6.x 版只允许每个 Index 包含一个 Type， 7.x 版将会彻底移除Type。
```
curl 'localhost:9200/_mapping?pretty=true'

返回
"professionaldoc" : {
    "mappings" : {
      "doc" : {
        "dynamic" : "strict",
        "properties" : {
          "content" : {
            "type" : "string",
            "store" : true,
            "analyzer" : "ik_max_word",
            "similarity" : "BM25"
          },
          "create_time" : {
            "type" : "long",
            "store" : true
          },
          "doc_id" : {
            "type" : "long",
            "store" : true
          },
          "doc_type" : {
            "type" : "integer",
            "store" : true
          },
          "download_count" : {
            "type" : "long",
            "store" : true
          },
          "page_num" : {
            "type" : "long",
            "store" : true
          },
          "quality_score" : {
            "type" : "double",
            "store" : true
          },
          "status" : {
            "type" : "integer",
            "store" : true
          },
          "title" : {
            "type" : "string",
            "store" : true,
            "analyzer" : "ik_max_word",
            "similarity" : "BM25",
            "fields" : {
              "raw" : {
                "type" : "string",
                "index" : "not_analyzed",
                "store" : true,
                "similarity" : "BM25"
              }
            }
          },
          "version" : {
            "type" : "integer",
            "store" : true
          }
        }
      }
    }
}
```


以易读的格式返回结果
------------
```
地址后面加 ?pretty=true

curl 'localhost:9200/accounts/person/1?pretty=true'
```


直接请求_search
------------
```
如果不对某一特殊的索引或者类型做限制，就会搜索集群中的所有文档。Elasticsearch 转发搜索请求到每一个主分片或者副本分片，汇集查询出的前10个结果。

curl 'xxx.xxx.xxx.xxx:9200/_search?pretty=true'
curl 'localhost:9200/_search'
curl 'localhost:9200/_search?pretty=true'
curl 'localhost:9200/accounts/_search?pretty=true'
curl 'localhost:9200/accounts/person/_search?pretty=true'


/_search
在所有的索引中搜索所有的类型

/gb/_search
在 gb 索引中搜索所有的类型

/gb,us/_search
在 gb 和 us 索引中搜索所有的文档

/g*,u*/_search
在任何以 g 或者 u 开头的索引中搜索所有的类型

/gb/user/_search
在 gb 索引中搜索 user 类型

/gb,us/user,tweet/_search
在 gb 和 us 索引中搜索 user 和 tweet 类型

/_all/user,tweet/_search
在所有的索引中搜索 user 和 tweet 类型


如果带参数
curl -XGET 'http://localhost:9200/megacorp/employee/_search?pretty=true' -d '{ 拼装json字串 }'
```



分页
------------
```
和 SQL 使用 LIMIT 关键字返回单个 page 结果的方法相同，Elasticsearch 接受 from 和 size 参数：

size
显示应该返回的结果数量，默认是 10
from
显示应该跳过的初始结果数量，默认是 0
如果每页展示 5 条结果，可以用下面方式请求得到 1 到 3 页的结果：

GET xxx/_search?size=5
GET xxx/_search?size=5&from=5
GET xxx/_search?size=5&from=10
```




结构化搜索（Structured search） 
=============

查询所有
------------
```
GET /ecommerce/product/_search
{
  "query": { "match_all": {} }
}
```



full-text search（全文检索）
------------
全文检索会将输入的搜索串拆解开来，去倒排索引里面去一一匹配，只要能匹配上任意一个拆解后的单词，就可以作为结果返回。  

我们可以这样查询姓 Smith 的员工
```
请求
curl -XGET  'http://localhost:9200/megacorp/employee/_search?pretty=true' -d '{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}'

返回
{
    "took": 3, 
    "timed_out": false, 
    "_shards": {
        "total": 2, 
        "successful": 2, 
        "failed": 0
    }, 
    "hits": {
        "total": 1, 
        "max_score": 1, 
        "hits": [
            {
                "_index": "megacorp", 
                "_type": "employee", 
                "_id": "2", 
                "_score": 1, 
                "_source": {
                    "first_name": "Jane", 
                    "last_name": "Smith", 
                    "age": 32, 
                    "about": "I like to collect rock albums", 
                    "interests": [
                        "music"
                    ], 
                    "join_time": "2014-11-24"
                }
            }
        ]
    }
}
```


phrase search（短语搜索）
-----------
找出一个属性中的独立单词是没有问题的，但有时候想要精确匹配一系列单词或者短语 。 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” 和 “climbing” ，并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。
```
GET /ecommerce/product/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}

返回
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         }
      ]
   }
}
```



highlight search（高亮搜索结果）
-----------
当执行该查询时，返回结果与之前一样，与此同时结果中还多了一个叫做 highlight 的部分。这个部分包含了 about 属性匹配的文本片段，并以 HTML 标签 <em></em> 封装：
```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}

返回
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" 
               ]
            }
         }
      ]
   }
}
```


分页
-----------
假设总共3条商品，每页就显示1条商品，现在显示第2页，所以就查出来第2个商品
```
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "from": 1,
  "size": 1
}
```


排序
----------
```
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "name" : "yagao"
        }
    },
    "sort": [
        { "price": "desc" }
    ]
}
```


指定返回的字段
----------
```
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "_source": ["name", "price"]
}
```



term 查询会查找我们指定的精确值
------------
```
curl 'xxx.xxx.xxx.xxx:8080/source/question_detail_info/_search?pretty=true' -d '{
  "query" :{
    "terms" : {
          "test_type" : ["4"]
      }
  }
}'


{
  "query" : {
    "term" : {
        "price" : 20
    }
}
}

{
	"query" :{
		"terms" : {
	        "_id" : [1,2,3]
	    }
	}
}
```


组合过滤器
--------------
布尔过滤器
```
一个 bool 过滤器由三部分组成：

{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
   }
}

must
所有的语句都 必须（must） 匹配，与 AND 等价。

must_not
所有的语句都 不能（must not） 匹配，与 NOT 等价。

should
至少有一个语句要匹配，与 OR 等价。

就这么简单！ 当我们需要多个过滤器时，只须将它们置入 bool 过滤器的不同部分即可。
```


```
SELECT product
FROM   products
WHERE  (price = 20 OR productID = "XHDK-A-1293-#fJ3")
  AND  (price != 30)


GET /my_store/products/_search
{
   "query" : {
      "filtered" : { 
         "filter" : {
            "bool" : {
              "should" : [
                 { "term" : {"price" : 20}}, 
                 { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} 
              ],
              "must_not" : {
                 "term" : {"price" : 30} 
              }
           }
         }
      }
   }
}

    
注意，我们仍然需要一个 filtered 查询将所有的东西包起来。 

在 should 语句块里面的两个 term 过滤器与 bool 过滤器是父子关系，两个 term 条件需要匹配其一。

如果一个产品的价格是 30 ，那么它会自动被排除，因为它处于 must_not 语句里面。
```



尽管 bool 是一个复合的过滤器，可以接受多个子过滤器，需要注意的是 bool 过滤器本身仍然还只是一个过滤器。 这意味着我们可以将一个 bool 过滤器置于其他 bool 过滤器内部，这为我们提供了对任意复杂布尔逻辑进行处理的能力。
```
SELECT document
FROM   products
WHERE  productID      = "KDKE-B-9947-#kL5"
  OR ( 
       productID = "JODL-X-1937-#pV7"
       AND 
       price     = 30 
     )


GET /my_store/products/_search
{
   "query" : {
      "filtered" : {
         "filter" : {
            "bool" : {
              "should" : [
                { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, 
                { "bool" : { 
                  "must" : [
                    { "term" : {"productID" : "JODL-X-1937-#pV7"}}, 
                    { "term" : {"price" : 30}} 
                  ]
                }}
              ]
           }
         }
      }
   }
}


因为 term 和 bool 过滤器是兄弟关系，他们都处于外层的布尔逻辑 should 的内部，返回的命中文档至少须匹配其中一个过滤器的条件。

这两个 term 语句作为兄弟关系，同时处于 must 语句之中，所以返回的命中文档要必须都能同时匹配这两个条件。
```







query filter，有很多组合
------------
```
GET /ecommerce/product/_search
{
    "query" : {
        "bool" : {
            "must" : {
                "match" : {
                    "name" : "yagao" 
                }
            },
            "filter" : {
                "range" : {
                    "price" : { 
                        "gt" : 25,
                        "lt" : 35
                    } 
                }
            }
        }
    }
}
```

```
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }}, 
        { "match": { "content": "Elasticsearch" }}  
      ],
      "filter": [ 
        { "term":  { "status": "published" }}, 
        { "range": { "publish_date": { "gte": "2015-01-01" }}} 
      ]
    }
  }
}
```

```
GET /_search
{
    "query" : {
        "match" : {
            "name" : "yagao"
        }
    },
    "filter": {
    	'and':[ 
	        { "term": { "uid": 1 }}, 
	        { "term": { "status": 2 }} 
	    ]
    },
    "from": 1,
  	"size": 1,
    "sort": [
        { "price": "desc" }
    ]
}
```


查询语句query 和 过滤语句filter
-------------
本质区别:  
query关注点：此文档与此查询子句的匹配程度如何？  
filter关注点：此文档和查询子句匹配吗？  


使用场景:  
全文检索以及任何使用相关性评分的场景使用query检索。   
除此之外的其他使用filter过滤器过滤。  

原则上来说，使用查询语句做全文本搜索或其他需要进行相关性评分的时候，剩下的全部用过滤语句


查询query和过滤器filter已合并（在ES1.X版本是分开的，存在filtered检索类型）。

ES高版本（2.X/5.X/6.x以后），任何查询子句都可以在“查询上下文query”中用作查询，并在“过滤器上下文filter”中用作过滤器。 

```
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }}, 
        { "match": { "content": "Elasticsearch" }}  
      ],
      "filter": [ 
        { "term":  { "status": "published" }}, 
        { "range": { "publish_date": { "gte": "2015-01-01" }}} 
      ]
    }
  }
}
```




nested
----------
嵌套对象 被索引在独立隐藏的文档中，我们无法直接查询它们。 相应地，我们必须使用 nested 查询 去获取它们：
```
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "eggs" 
          }
        },
        {
          "nested": {
            "path": "comments", 
            "query": {
              "bool": {
                "must": [ 
                  {
                    "match": {
                      "comments.name": "john"
                    }
                  },
                  {
                    "match": {
                      "comments.age": 28
                    }
                  }
                ]
              }
            }
          }
        }
      ]
}}}
```

























倒排索引

id term 文档号:位置
1 best 2:3,3:1


查询性能

缓存





















