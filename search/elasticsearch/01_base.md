
Esasticsearch
==========
esasticsearch 基于lucene，隐藏复杂性，提供简单易用的restful api接口。


核心概念
--------
NRT (Near Realtime)：近实时。两个意思：
	从写入数据到数据可以被搜索到有一个小延迟（大概1秒）。
	基于es执行搜索和分析可以达到秒级。

Cluster
Node

Shard 其实叫primary shard，一般简称为shard。
Replica 其实叫replica shard，一般简称replica。

Index
Type
Document: es的document采用json数据格式
Field



节点 Node、集群 Cluster 和分片 Shards
----------
ElasticSearch 是分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个实例。单个实例称为一个节点（node），一组节点构成一个集群（cluster）。分片是底层的工作单元，文档保存在分片内，分片又被分配到集群内的各个节点里，每个分片仅保存全部数据的一部分。


索引 Index、类型 Type 和文档 Document
----------
对比我们比较熟悉的 MySQL 数据库：
```
index → db
type → table
document → row
```
如果我们要访问一个文档元数据应该包括囊括 index/type/id 这三种类型，很好理解。



es 对应 数据库
----------
```
Index 数据库
Type  表
Document 行
```

下载与安装
----------
1 需要先安装java环境

2 然后下载es包，解压后即可使用：进入es的bin目录，执行即可

2-1 检查es是否启动成功，访问 http://localhost:9200/?pretty

3 下载Kibana包，解压后即可使用：进入Kibana的bin目录，执行即可

3-1 进入Kibana的Dev Tools界面，访问 http://localhost:5601


查看es版本
----------
http://localhost:9200/?pretty
//http://xxx.xxx.xxx.xxx:8080/?pretty
```
{
	status: 200,
	name: "xxx.xxx.xxx.xxx", //node名称
	cluster_name: "aaaaaa", //集群名称
	version: {
		number: "1.6.0",
		build_hash: "bbbbbbccccccc",
		build_timestamp: "2018-01-05T18:10:31Z",
		build_snapshot: false,
		lucene_version: "4.10.4"
	},
	tagline: "You Know, for Search"
}

name: node名称
cluster_name: 集群名称，可在配置文件中调整(elasticsearch.yml文件中的cluster_name)
version: es 版本信息
```


Kibana
----------
下载Kibana包，解压后即可使用：进入Kibana的bin目录，执行即可

进入Kibana的Dev Tools界面，访问 http://localhost:5601

查看es健康状态
```
GET _cluster/health
```


curl方式请求
===========
> curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'

```
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```




es简单的集群管理
============
(1) 快速检查集群的健康状部
------------
es提供了一套api，叫做cat api，可以查看es中各种各样的数据

```
GET /_cat/health?v
加参数v 可以显示列头

epoch      timestamp cluster      status node.total node.data shards pri relo init unassign pending_tasks 
1565845665 13:07:45  abccccccccc  green          10        10    126  63    0    0        0             0 
```

如何快速了解集群的健康状况？查看status列的值即可：green yellow red
```
green: 
每个索引的primary shard和replica shard都是active状态

yellow: 
每个索引的primary shard都是active状态，但是部分replica shard不是active状态，处于不可用状态

red:
不是所有索引的primary shard都是active状态，部分索引有数据丢失了
```

个人搭建的单节点es，健康状态是yellow，原因是？  
我们用一台机子，就启动了一个es进程，相当于只有一个node。当我们在es中创建了一个index。由于默认的配置是给每个index分配5个primary shard和5个replica shard，而且rimary shard和replica shard不能在同一台机器上(为了容错)。当前就一个node，所以只有5个primary shard被分配了和启动了，但是一个replica shard没有第二台机器去启动。

我们可以再启动一个es，这时就有两个node了，replica shard将被分配过去，状态即可变为green。



(2) 快速查看集群中有哪些索引
------------
```
GET /_cat/indices?v

health status index            pri rep docs.count docs.deleted store.size pri.store.size 
green  open   xxx              6   1    3550577       119251      1.6gb        819.1mb 
green  open   xxxx1            6   1     132709            0    334.1mb          167mb 
green  open   xxx2             6   1   91329052      2223765    145.5gb         74.3gb 
green  open   abc              5   1    6420486       635673      5.3gb          2.6gb 
green  open   cccc             6   1     221157            0    767.2mb        383.7mb 
green  open   ddd              6   1      14992           90     10.6mb          5.3mb 
green  open   eee              5   1    4533120        13803      8.8gb          4.4gb 
green  open   ffff             6   1   34671028       680905    152.5gb           75gb 
green  open   ggggggg          5   1     280412         1550    823.6mb        411.8mb 
green  open   uuu              6   1     181074            0    677.3mb        338.2mb 
green  open   haha             6   1     677105            2    214.1mb          107mb 
```

简单索引操作
============
创建索引
```
PUT /test_index?pretty

返回
{
	"acknowledged":true,
	"shards_acknowledged":true,
}
```

删除索引
```
DELETE /test_index?pretty

返回
{
	"acknowledged":true
}
```


CRUD实例
===========
新增商品
```
语法：PUT /index/type/id

PUT /ecommerce/product/1
{
	"name":"gaolujie yagao",
	"desc":"gaoxiao meibai",
	"price":30,
	"producer":"gaolujie producer",
	"tags":["meibai", "fangzhu"]
}

PUT /ecommerce/product/2
{
	"name":"jiajieshi yagao",
	"desc":"youxiao fangzhu",
	"price":25,
	"producer":"jiajieshi producer",
	"tags":["fangzhu"]
}

注：es新增数据时会自动根据index和type创建index与type，不需要提前创建，而且es默认会对document每个field都建立倒排索引，让其可以被搜索。
```

查询商品
```
GET /ecommerce/product/1

返回

```


修改商品 - 替换文档
```
PUT /ecommerce/product/1
{
	"name":"gaolujie yagao",
	"desc":"gaoxiao meibai",
	"price":30,
	"producer":"gaolujie producer",
	"tags":["meibai", "fangzhu"]
}

注意：替换后，原文档字段丢失，用的是新文档的字段。如果只想修改某个字段，可用用post与_update更新文档
```

修改商品 - 更新文档
```
POST /ecommerce/product/1/_update
{
	"name":"jiaqiangban gaolujie yagao"
}
```

删除商品
```
DELETE /ecommerce/product/1/?pretty

返回：


?pretty表示格式化显示返回结果
```



多种搜索方式
===========
搜索全部商品
```
GET /ecommerce/product/_search

took: 耗费了几毫秒
timed_out: 是否超时，false表示没有
_shards: 数据拆成了5个分片，所以对于搜索请求，会打到所有的primary shard（或者是它的某个replica shard也可以）

hits.total: 查询结果的数量，即多少个document
hits.max_score: document对于一个search的相关度的匹配分数，越相关，就越匹配，分数也越高
hits.hits: 包含了匹配搜索的document的详细数据
```


1 query string search
-----------
在url后url中带条件参数

生产环境中不用


2 query DSL
-----------
DSL: Domain Specified Language，特定领域的语言

在http request body 请求体中，用json格式来构建查询语法，比较方便，可构建各种复杂的语法

查询所有
```
GET /ecommerce/product/_search
{
	"query": {
		"match_all":{}
	}
}
```

查询名称包含yaogao的商品，同时按照降序排序
```
GET /ecommerce/product/_search
{
	"query": {
		"match":{
			"name":"yagao"
		}
	},
	"sort":[
		{"price":"desc"}
	]
}
```

分页查询商品，假设每页显示一条商品
```
GET /ecommerce/product/_search
{
	"query": {
		"match":{
			"name":"yagao"
		}
	},
	"sort":[
		{"price":"desc"}
	],
	"from":1,//从第几条开始
	"size":1 //每页条数
}
```

指定只需要查询出商品的名称和价格
```
需要用到_source 

GET /ecommerce/product/_search
{
	"query": {
		"match":{
			"name":"yagao"
		}
	},
	"_source":[
		"name",
		"price"
	]
	"sort":[
		{"price":"desc"}
	],
	"from":1,//从第几条开始
	"size":1 //每页条数
}
```

更加适合在生产环境中使用，可以构建复杂的查询

3 query filter 对数据进行过滤
-----------
搜索商品名称含yaogao，而且售价大于25元的商品
```
GET /ecommerce/product/_search
{
	"query":{
		"bool":{
			"must":{
				"match":{
					"name":"yaogao"
				}
			},
			"filter":{
				"range":{
					"price":{
						"gt":25
					}
				}
			}
		}
	}
}

这种包含多种查询条件的
```


4 full-text search （全文检索）
-----------
将输入的搜索串拆解开来，去倒排索引里面一一匹配，只要能匹配上任意一个拆解后的单语，就可以作为结果返回。
```
GET /ecommerce/product/_search
{
	"query":{
		"match":{
			"producer":"yagao producer"
		}
	}
}
```


5 phrase search（短语搜索）
-----------
要求输入的搜索串，必须在指定的字段文本中，完全包含一模一样的，才可以算匹配，才能作为结果返回
```
GET /ecommerce/product/_search
{
	"query":{
		"match_phrase":{
			"producer":"yagao producer"
		}
	}
}
```

6 高亮搜索结果
-----------
```
GET /ecommerce/product/_search
{
	"query":{
		"match":{
			"producer":"producer"
		}
	},
	"highlight":{
		"fields":{
			"producer":{}
		}
	}
}
```


group by + avg + sort等聚合分析
================

使用aggs前，需要先将文本field的fielddata属性设置为true。
```
PUT /ecommerce/_mapping/product
{
	"properties":{
		"tags":{
			"type":"text",
			"fielddata":true
		}
	}
}
```

1 计算每个tag下的商品数量
-------------
```
GET /ecommerce/product/_search
{
	"size":0,//如果不加该参数，则会同时返回参与聚合的数据。size设为0，则不返回
	"aggs":{
		"group_tags 这个是起的任意名字":{
			"terms":{
				"field":"tags"
			}
		}
	}
}
```


2 对名称中包含yaogao的商品，计算每个tag下的商品数量
----------
```
GET /ecommerce/product/_search
{
	"size":0,//如果不加该参数，则会同时返回参与聚合的数据。size设为0，则不返回
	"query":{
		"match":{
			"name":"yagao"
		}
	},
	"aggs":{
		"group_tags 这个是起的任意名字":{
			"terms":{
				"field":"tags"
			}
		}
	}
}
```

3 计算每个tag下的商品的平均价格
-----------
思路：先分组，再算每组的平均值
```
GET /ecommerce/product/_search
{
	"size":0,//如果不加该参数，则会同时返回参与聚合的数据。size设为0，则不返回
	"query":{
		"match":{
			"name":"yagao"
		}
	},
	"aggs":{
		"group_tags 这个是起的任意名字":{
			"terms":{
				"field":"tags",
				"order":{"avg_price":"desc"} //
			},
			"aggs":{
				"avg_price 这个是起的任意名字":{
					"avg":{"field":"price"}
				}
			}
		}
	}
}
```

4 按照指定的价格范围区间进行分组，然后在每组内再按照tag进行分组，最后再计算每组的平均价格
-----------
思路：先分组，再算每组的平均值
```
GET /ecommerce/product/_search
{
	"size":0,
	"aggs":{
		"group_by_price 这个是起的任意名字":{
			"range":{
				"field":"price",
				"ranges":[
					{
						"from":0,
						"to":20
					},
					{
						"from":20,
						"to":40
					},
					{
						"from":40,
						"to":50
					},
				]
			},
			"aggs":{
				"group_by_tags 这个是起的任意名字":{
					"term":{
						"field":"tags"
					},
					"aggs":{
						"average_price 这个是起的任意名字":{
							"avg":{
								"field":"price"
							}
						}
					}
				}
			}
		}
	}
}
```


