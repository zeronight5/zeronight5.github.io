---
title: ElasticSearch 基本操作
date: 2021-03-27 14:58:46
tags:
---

# Elasticsearch介绍

全文搜索属于最常见的需求，开源的 Elasticsearch 是目前全文搜索引擎的首选。它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github 都采用它。

Elasticsearch 的底层是开源库 Lucene。Elasticsearch 是 Lucene 的封装，提供了 REST API 的操作接口，开箱即用。

原理简介参考[终于有人把Elasticsearch原理讲透了！](https://zhuanlan.zhihu.com/p/62892586)

1. ## 基础概念

   1. 节点 Node、集群 Cluster 和分片 Shards

      ElasticSearch 是分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个实例。单个实例称为一个节点（node），一组节点构成一个集群（cluster）。

      分片是底层的工作单元，文档保存在分片内，分片又被分配到集群内的各个节点里，每个分片仅保存全部数据的一部分。

   2. 索引 Index、类型 Type 和文档 Document

      方便理解，对比我们比较熟悉的 MySQL 数据库：

      index → db

      type → table

      document → row

      这是一个不恰当的比喻。在数据库中，table之间是相互独立的，两个table中相同名字的column之间无关联。

      但在Elasticsearch中，Index下不同Type中Document的同名field是同一个，必须要有相同的映射定义。具体参考[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)。

2. ## 使用 RESTful API 与 Elasticsearch 进行交互

   所有其他语言可以使用 RESTful API 通过默认端口 9200 和 Elasticsearch 进行通信。一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：

   `curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`

   被 < > 标记的部件：

   部件名作用

   VERB：适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。

   PROTOCOL：http 或者 https

   HOST：Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。

   PORT：运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。

   PATH：API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。

   QUERY_STRING：任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)

   BODY：一个 JSON 格式的请求体 (如果请求需要的话)

   示例

   ```
   curl -XGET 'http://localhost:9200/_count?pretty' -d '
   {
       "query": {
           "match_all": {}
       }
   }'
   ```

3. ## 文档管理（CRUD）

   1. 增加：

      ```
      POST /db/user/1
      {
        "username": "wmyskxz1",
        "password": "123456",
        "age": "22"
      }
      
      POST /db/user/2
      {
        "username": "wmyskxz2",
        "password": "123456",
        "age": "22"
      }
      ```

      这一段代码稍微解释一下，这其实就往索引为 db 类型为 user 的数据库中插入一条 id 为 1 的一条数据，这条数据其实就相当于一个拥有 username/password/age 三个属性的一个实体，就是 JSON 数据

      执行命令后，Elasticsearch 返回如下数据：

      ```
      # POST /db/user/1
      {
        "_index": "db",
        "_type": "user",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1
      }
      
      # POST /db/user/2
      {
        "_index": "db",
        "_type": "user",
        "_id": "2",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1
      }
      ```

      version 是版本号的意思，当我们执行操作会自动加 1

   2. 删除：

      ```
      DELETE /db/user/1
      ```

      Elasticsearch 返回数据如下：

      ```
      {
        "_index": "db",
        "_type": "user",
        "_id": "1",
        "_version": 2,
        "result": "deleted",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1
      }
      ```

      这里就可以看到 version 变成了 2

   3. 修改：

      ```
      PUT /db/user/2
      {
        "username": "wmyskxz3",
        "password": "123456",
        "age": "22"
      }
      ```

      Elasticsearch 返回数据如下：

      ```
      {
        "_index": "db",
        "_type": "user",
        "_id": "2",
        "_version": 2,
        "result": "updated",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1
      }
      ```

   4. 查询：

      ```
      GET /db/user/2
      ```

      返回数据如下：

      ```
      {
        "_index": "db",
        "_type": "user",
        "_id": "2",
        "_version": 2,
        "found": true,
        "_source": {
          "username": "wmyskxz3",
          "password": "123456",
          "age": "22"
        }
      }
      ```

4. ## 搜索

   请求_search接口时，某些客户端不支持GET请求中有Body，需将GET改为POST。

   先往 Elasticsearch 中插入一些数据：

   ```
   PUT /movies/movie/1
   {
     "title": "The Godfather",
     "director": "Francis Ford Coppola",
     "year": 1972,
     "genres": [
       "Crime",
       "Drama"
     ]
   }
   
   PUT /movies/movie/2
   {
     "title": "Lawrence of Arabia",
     "director": "David Lean",
     "year": 1962,
     "genres": [
       "Adventure",
       "Biography",
       "Drama"
     ]
   }
   
   PUT /movies/movie/3
   {
     "title": "To Kill a Mockingbird",
     "director": "Robert Mulligan",
     "year": 1962,
     "genres": [
       "Crime",
       "Drama",
       "Mystery"
     ]
   }
   
   PUT /movies/movie/4
   {
     "title": "Apocalypse Now",
     "director": "Francis Ford Coppola",
     "year": 1979,
     "genres": [
       "Drama",
       "War"
     ]
   }
   
   PUT /movies/movie/5
   {
     "title": "Kill Bill: Vol. 1",
     "director": "Quentin Tarantino",
     "year": 2003,
     "genres": [
       "Action",
       "Crime",
       "Thriller"
     ]
   }
   
   PUT /movies/movie/6
   {
     "title": "The Assassination of Jesse James by the Coward Robert Ford",
     "director": "Andrew Dominik",
     "year": 2007,
     "genres": [
       "Biography",
       "Crime",
       "Drama"
     ]
   }
   ```

   现在已经把一些电影信息放入了索引，可以通过搜索看看是否可找到它们。 为了使用 ElasticSearch 进行搜索，我们使用 _search 端点，可选择使用索引和类型。

   也就是说，按照以下模式向URL发出请求：<index>/<type>/_search。其中，index 和 type 都是可选的。

   换句话说，为了搜索电影，可以对以下任一URL进行POST请求：

   > http://localhost:9200/_search - 搜索所有索引和所有类型。
   >
   > http://localhost:9200/movies/_search - 在电影索引中搜索所有类型
   >
   > http://localhost:9200/movies/movie/_search - 在电影索引中显式搜索电影类型的文档。

   ##### **搜索请求正文和ElasticSearch查询DSL**

   如果只是发送一个请求到上面的URL，我们会得到所有的电影信息。为了创建更有用的搜索请求，还需要向请求正文中提供查询。 

   请求正文是一个JSON对象，除了其它属性以外，它还要包含一个名称为 “query” 的属性，这就可使用ElasticSearch的查询DSL。

   ```
   { "query": { //Query DSL here } }
   ```

   DSL是ElasticSearch自己基于JSON的域特定语言，可以在其中表达查询和过滤器。可以把它简单同SQL对应起来，相当于条件语句。

   ##### **基本自由文本搜索**

   查询DSL具有一长列不同类型的查询可以使用。 对于“普通”自由文本搜索，最有可能想使用一个名称为“查询字符串查询”。

   [查询字符串查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-query-string-query.html)是一个高级查询，有很多不同的选项，ElasticSearch将解析和转换为更简单的查询树。如果忽略了所有的可选参数，并且只需要给它一个字符串用于搜索，它可以很容易使用。

   现在尝试在两部电影的标题中搜索有“kill”这个词的电影信息：

   ```
   GET /_search 
   { "query": { "query_string": { "query": "kill" } } }
   ```

   ##### **指定搜索的字段**

   在前面的例子中，使用了一个非常简单的查询，一个只有一个属性 “query” 的查询字符串查询。 如前所述，查询字符串查询有一些可以指定设置，如果不使用，它将会使用默认的设置值。

   这样的设置称为“fields”，可用于指定要搜索的字段列表。如果不使用“fields”字段，ElasticSearch查询将默认自动生成的名为 “_all” 的特殊字段，来基于所有文档中的各个字段匹配搜索。

   为了做到这一点，修改以前的搜索请求正文，以便查询字符串查询有一个 fields 属性用来要搜索的字段数组：

   ```
   GET /_search
   {
     "query": {
       "query_string": {
         "query": "ford",
         "fields": [
           "title"
         ]
       }
     }
   }
   ```

   ##### **条件匹配**

   ```
   GET /_search
   {
     "query": {
       "match": {
         "year": 2007
       }
     }
   }
   ```

   ##### **过滤返回结果字段**

   ```
   GET /_search
   {
     "query": {
       "query_string": {
         "query": "ford",
         "fields": [
           "title"
         ]
       }
     },
     "_source": ["title","director"]
   }
   ```

   ##### **返回结果包含指定字段**

   ```
   GET /_search
   {
     "query": {
       "query_string": {
         "query": "ford",
         "fields": [
           "title"
         ]
       }
     },
     "_source": {
       "includes": [
         "title",
         "director"
       ]
     }
   }
   ```

   ##### **返回结果排除指定字段**

   ```
   GET /_search
   {
     "query": {
       "query_string": {
         "query": "ford",
         "fields": [
           "title"
         ]
       }
     },
     "_source": {
       "excludes": [
         "title",
         "director"
       ]
     }
   }
   ```

   ##### **排序**

   ```
   GET /_search
   {
     "query": {
       "query_string": {
         "query": "ford"
       }
     },
     "sort": {
       "year": {
         "order": "desc"
       }
     }
   }
   ```

   ##### **分页**

   ```
   GET /_search
   {
     "query": {
       "query_string": {
         "query": "ford"
       }
     },
     "sort": {
       "year": {
         "order": "desc"
       }
     },
     "from": 0,
     "size": 1
   }
   ```

   ##### **多条件查询-布尔值查询**

   ```
   GET /_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "year": 2007
             }
           }
         ]
       }
     }
   }
   ```

   ```
   GET /_search
   {
     "query": {
       "bool": {
         "should": [
           {
             "match": {
               "year": 2007
             }
           }
         ]
       }
     }
   }
   ```

   ```
   GET /_search
   {
     "query": {
       "bool": {
         "must_not": [
           {
             "match": {
               "year": 2007
             }
           }
         ]
       }
     }
   }
   ```

   ##### **多条件查询-条件区间**

   ```
   GET /_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "genres": "Drama"
             }
           }
         ],
         "filter": [
           {
             "range": {
               "year": {
                 "gte": 2000
               }
             }
           }
         ]
       }
     }
   }
   ```

   ##### **精确查找**

   term是代表完全匹配，即不进行分词器分析，文档中必须包含整个搜索的词汇。

   match和term的区别是，match查询的时候，elasticsearch会根据给定的字段提供合适的分析器，而term查询不会有分析器分析的过程，match查询相当于模糊匹配，只包含其中一部分关键词就行。

   ```
   GET /_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "term": {
               "year": 2007
             }
           }
         ]
       }
     }
   }
   ```

   ##### **短语搜索**

   match_phrase 称为短语搜索，要求所有的分词必须同时出现在文档中，同时位置必须紧邻一致。

   ```
   GET /_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match_phrase": {
               "director": "Andrew Dominik"
             }
           }
         ]
       }
     }
   }
   ```

5. ## 聚合

   ### 聚合分析简介

   ##### 1. ES聚合分析是什么？

   聚合分析是数据库中重要的功能特性，完成对一个查询的数据集中数据的聚合计算，如：找出某字段（或计算表达式的结果）的最大值、最小值，计算和、平均值等。ES作为搜索引擎兼数据库，同样提供了强大的聚合分析能力。

   对一个数据集求最大、最小、和、平均值等指标的聚合，在ES中称为**指标聚合(metric)**

   而关系型数据库中除了有聚合函数外，还可以对查询出的数据进行分组group by，再在组上进行指标聚合。在 ES 中group by 称为**分桶**，**桶聚合(bucketing)**

   ES中还提供了矩阵聚合（matrix）、管道聚合（pipleline），但还在完善中。 

   ##### 2. ES聚合分析查询的写法

    在查询请求体中以aggregations节点按如下语法定义聚合分析：

   ```
   {
     "aggregations" : {
       "<aggregation_name>" : { <!--聚合的名字 -->
           "<aggregation_type>" : { <!--聚合的类型 -->
               <aggregation_body> <!--聚合体：对哪些字段进行聚合 -->
           }
           [,"meta" : {  [<meta_data_body>] } ]? <!--元 -->
           [,"aggregations" : { [<sub_aggregation>]+ } ]? <!--在聚合里面在定义子聚合 -->
       }
       [,"<aggregation_name_2>" : { ... } ]*<!--聚合的名字 -->
     }
   }
   ```

   **aggregations 也可简写为 aggs**

   ##### 3. 聚合分析的值来源

   聚合计算的值可以取**字段的值**，也可是**脚本计算的结果**。

   ### 指标聚合

   ##### 1. max min sum avg

   获取指定字段最大值

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "max_year": {
         "max": {
           "field": "year"
         }
       }
     }
   }
   ```

   获取查询结果指定字段最小值，按指定字段排序

   ```
   GET /_search
   {
     "size": 1,
     "query": {
       "match": {
         "genres": "Drama"
       }
     },
     "sort": [
       {
         "year": {
           "order": "asc"
         }
       }
     ],
     "aggs": {
       "max_year": {
         "min": {
           "field": "year"
         }
       }
     }
   }
   ```

   指定field，在脚本中用_value 取字段的值

   ```
   {
     "size": 0,
     "aggs": {
       "sum_year": {
         "sum": {
           "field": "year",
           "script": {
             "source": "_value * 2"
           }
         }
       }
     }
   }
   ```

   ##### 2. 文档计数

   注意接口地址为_count

   ```
   GET /_count
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   ##### 3. value_count 统计某字段有值的文档数

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "year_count": {
         "value_count": {
           "field": "year"
         }
       }
     }
   }
   ```

   ##### 4. cardinality 值去重计数

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "year_count": {
         "cardinality": {
           "field": "year"
         }
       }
     }
   }
   ```

   ##### 5. stats 统计 count max min avg sum 5个值

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "stats": {
         "stats": {
           "field": "year"
         }
       }
     }
   }
   ```

   ##### 6. extended_stats 高级统计，比stats多4个统计结果： 平方和、方差、标准差、平均值加/减两个标准差的区间

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "stats": {
         "extended_stats": {
           "field": "year"
         }
       }
     }
   }
   ```

   ##### 7. percentiles 占比百分位对应的值统计

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "percentiles": {
         "percentiles": {
           "field": "year"
         }
       }
     }
   }
   ```

   ##### 8. percentile_ranks 统计值小于等于指定值的文档占比

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "percentiles": {
         "percentile_ranks": {
           "field": "year",
           "values": [
             2000,
             1990
           ]
         }
       }
     }
   }
   ```

   ### 桶聚合

   ##### 1. Terms Aggregation 根据字段值项分组聚合 

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "year_terms": {
         "terms": {
           "field": "year"
         }
       }
     }
   }
   ```

   ##### 2. Filter Aggregation 对满足过滤查询的文档进行聚合计算

   ```
   GET /_search
   {
     "size": 0,
     "aggs": {
       "filtered_year_terms": {
         "filter": {
           "match": {
             "genres": "Drama"
           }
         },
         "aggs": {
           "year_terms": {
             "terms": {
               "field": "year"
             }
           }
         }
       }
     }
   }
   ```

# 使用提示

1. 新建文档索引若不指定字段类型映射，则默认用动态映射。string类型会映射成text。[Elasticsearch中text与keyword的区别](https://www.cnblogs.com/sanduzxcvbnm/p/12177377.html)
2. 新建文档索引时，如果有date字段，注意格式化结果和索引配置是否一致，若不一致新增索引文档会失败。
3. es的服务端和客户端需要使用相同版本号，不通版本之间不兼容。

# Elasticsearch Head插件

elasticsearch-head将是一款专门针对于elasticsearch的客户端工具，提供Chrome浏览器插件。仓库地址https://github.com/mobz/elasticsearch-head 。