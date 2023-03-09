# ES-文档-基础入门

## 1.You Know，为了搜索

### 1.1适应新环境

为了让大家对 Elasticsearch 能实现什么及其上手难易程度有一个基本印象，让我们从一个简单的教程开始并介绍索引、搜索及聚合等基础概念。

#### 创建一个雇员目录

我们受雇于 *Megacorp* 公司，作为 HR 部门新的 *“热爱无人机”* （*"We love our drones!"*）激励项目的一部分，我们的任务是为此创建一个员工目录。该目录应当能培养员工认同感及支持实时、高效、动态协作，因此有一些业务需求：

- 支持包含多值标签、数值、以及全文本的数据
- 检索任一员工的完整信息
- 允许结构化搜索，比如查询 30 岁以上的员工
- 允许简单的全文搜索以及较复杂的短语搜索
- 支持在匹配文档内容中高亮显示搜索片段
- 支持基于数据创建和管理分析仪表盘

### 1.2索引员工文档

- 第一个业务需求是**存储员工数据**。 这将会以 *员工文档* 的形式存储：
  - 一个文档代表一个员工。==存储数据到 Elasticsearch 的行为叫做 ***索引*** ，==但在索引一个文档之前，需要确定将文档存储在哪里。
- 一个 Elasticsearch 集群可以 包含多个 *索引* ，相应的每个索引可以包含多个 *类型* 。 这些不同的类型存储着多个 *文档* ，每个文档又有 多个 *属性* 。

> **Index Versus Index Versus Index**
>
> 你也许已经注意到 *索引* 这个词在 Elasticsearch 语境中有多种含义， 这里有必要做一些说明：
>
> ==索引（名词）：==
>
> 如前所述，一个 *索引* 类似于传统关系数据库中的一个 ***数据库*** ，是一个存储关系型文档的地方。 *索引* (*index*) 的复数词为 ***indices*** 或 ***indexes*** 。
>
> ==索引（动词）：==
>
> *索引一个文档* 就是存储一个文档到一个 *索引* （名词）中以便被检索和查询。这非常类似于 SQL 语句中的 `INSERT` 关键词，除了文档已存在时，新文档会替换旧文档情况之外。
>
> ==倒排索引：==
>
> 关系型数据库通过增加一个 *索引* 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 *倒排索引* 的结构来达到相同的目的。
>
> \+ 默认的，一个文档中的每一个属性都是 *被索引* 的（有一个倒排索引）和可搜索的。一个没有倒排索引的属性是不能被搜索到的。我们将在 [倒排索引](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html) 讨论倒排索引的更多细节。

对于员工目录，我们将做如下操作：

- 每个员工索引一个文档，文档包含该员工的所有信息。
- 每个文档都将是 `employee` *类型* 。
- 该类型位于 *索引* `megacorp` 内。
- 该索引保存在我们的 Elasticsearch 集群中。

实践中这非常简单（尽管看起来有很多步骤），我们可以通过一条命令完成所有这些动作：

```json
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

注意，路径 `/megacorp/employee/1` 包含了三部分的信息：

- **`megacorp`**
  - 索引名称
- **`employee`**
  - 类型名称
- **`1`**
  - 特定雇员的ID

> 请求体 —— JSON 文档 —— 包含了这位员工的所有详细信息，他的名字叫 John Smith ，今年 25 岁，喜欢攀岩。

很简单！无需进行执行管理任务，如创建一个索引或指定每个属性的数据类型之类的，可以直接只索引一个文档。Elasticsearch 默认地完成其他一切，因此所有必需的管理任务都在后台使用默认设置完成。

进行下一步前，让我们增加更多的员工信息到目录中：

```json
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```



### 1.3检索文档

目前我们已经在 Elasticsearch 中存储了一些数据， 接下来就能专注于实现应用的业务需求了。第一个需求是可以检索到单个雇员的数据。

这在 Elasticsearch 中很简单。简单地执行 一个 HTTP `GET` 请求并指定文档的地址——索引库、类型和ID。 使用这三个信息可以返回原始的 JSON 文档：

```json
GET /megacorp/employee/1
```

返回结果包含了文档的一些元数据，以及 `_source` 属性，内容是 John Smith 雇员的原始 JSON 文档：

```json
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }

```



### 1.4轻量搜索Query-string

一个 `GET` 是相当简单的，可以直接得到指定的文档。 现在尝试点儿稍微高级的功能，比如一个简单的搜索！

第一个尝试的几乎是最简单的搜索了。我们使用下列请求来搜索所有雇员：

```json
GET /megacorp/employee/_search
```

但与指定一个文档 ID 不同，这次使用 `_search` 。返回结果包括了所有三个文档，放在数组 `hits` 中。一个搜索默认返回**十条**结果。

```js
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

> 注意：返回结果不仅告知匹配了哪些文档，还包含了整个文档本身：显示搜索结果给最终用户所需的全部信息。

接下来，尝试下搜索姓氏为 ``Smith`` 的雇员。为此，我们将使用一个==*高亮*搜索==，很容易通过命令行完成。这个方法一般涉及到一个 ==*查询字符串*==**（*query-string*）** 搜索，因为我们通过一个URL参数来传递查询信息给搜索接口：

```json
GET /megacorp/employee/_search?q=last_name:Smith
```

我们仍然在请求路径中使用 `_search` 端点，并将查询本身赋值给参数 `q=` 。返回结果给出了所有的 Smith：

```json
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```



### 1.5使用查询表达式搜索DSL

Query-string 搜索通过命令非常方便地进行临时性的即席搜索 ，但它有自身的局限性（参见 [*轻量* 搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-lite.html) ）。

Elasticsearch 提供一个丰富灵活的查询语言叫做 ==*查询表达式*== ， 它支持构建更加复杂和健壮的查询。

==*领域特定语言*== **（DSL）**， 使用 JSON 构造了一个请求。我们可以像这样重写之前的查询所有名为 Smith 的搜索 

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

返回结果与之前的查询一样，但还是可以看到有一些变化。其中之一是，不再使用 *query-string* 参数，而是一个请求体替代。这个请求使用 JSON 构造，并使用了一个 `match` 查询（属于查询类型之一，后面将继续介绍）。



### 1.6过滤Filter

现在尝试下更复杂的搜索。 同样搜索姓氏为 Smith 的员工，但这次我们只需要年龄大于 30 的。查询需要稍作调整，使用过滤器 *filter* ，它支持高效地执行一个结构化查询。

```json
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith"   //①
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } //②
                }
            }
        }
    }
}
```

![image-20230301003840155](E:/Development/Typora/images/image-20230301003840155.png)

目前无需太多担心语法问题，后续会更详细地介绍。只需明确我们添加了一个 *过滤器* 用于执行一个范围查询，并复用之前的 `match` 查询。现在结果只返回了一名员工，叫 Jane Smith，32 岁。

```js
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```



### 1.7 全文搜索

搜索下所有喜欢攀岩（rock climbing）的员工：

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
   }
}
```

显然我们依旧使用之前的 `match` 查询在`about` 属性上搜索 “rock climbing” 。得到两个匹配的文档：

```js
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327,  //①
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016,   //①
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

![image-20230301004050854](E:/Development/Typora/images/image-20230301004050854.png)

Elasticsearch **默认按照相关性得分排序**，即每个文档跟查询的匹配程度。第一个最高得分的结果很明显：John Smith 的 `about` 属性清楚地写着 “rock climbing” 。

但为什么 Jane Smith 也作为结果返回了呢？原因是她的 `about` 属性里提到了 “rock” 。因为只有 “rock” 而没有 “climbing” ，所以她的相关性得分低于 John 的。

这是一个很好的案例，阐明了 Elasticsearch 如何 *在* 全文属性上搜索并返回相关性最强的结果。Elasticsearch中的 *相关性* 概念非常重要，也是完全区别于传统关系型数据库的一个概念，数据库中的一条记录要么匹配要么不匹配。

### 1.8短语搜索 match_phrase

找出一个属性中的独立单词是没有问题的，但有时候想要精确匹配一系列单词或者_短语_ 。 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” *和* “climbing” ，*并且* 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。

为此对 `match` 查询稍作调整，使用一个叫做 `match_phrase` 的查询：

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

毫无悬念，返回结果仅有 John Smith 的文档。

```json
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



### 1.9高亮搜索highlight

许多应用都倾向于在每个搜索结果中 *高亮* 部分文本片段，以便让用户知道为何该文档符合查询条件。在 Elasticsearch 中检索出高亮片段也很容易。

再次执行前面的查询，并增加一个新的 `highlight` 参数：

```json
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
```

当执行该查询时，返回结果与之前一样，与此同时结果中还多了一个叫做 `highlight` 的部分。==这个部分包含了 `about` 属性匹配的文本片段，并以 HTML 标签 `<em></em>` 封装：==

```json
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



### 1.10聚合

终于到了最后一个业务需求：支持管理者对员工目录做分析。 Elasticsearch 有一个功能叫**聚合**（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 `GROUP BY` 类似但更强大。

举个例子，挖掘出员工中最受欢迎的兴趣爱好：

```json
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

暂时忽略掉语法，直接看看结果：

```json
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

可以看到，两位员工对音乐感兴趣，一位对林业感兴趣，一位对运动感兴趣。这些聚合的结果数据并非预先统计，而是根据匹配当前查询的文档即时生成的。如果想知道叫 Smith 的员工中最受欢迎的兴趣爱好，可以直接构造一个组合查询：

```json
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```

`all_interests` 聚合已经变为只包含匹配查询的文档：

```js
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
```

聚合还支持分级汇总 。比如，查询特定兴趣爱好员工的平均年龄：

```json
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

得到的聚合结果有点儿复杂，但理解起来还是很简单的：

```json
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```

输出基本是第一次聚合的加强版。依然有一个兴趣及数量的列表，只不过每个兴趣都有了一个附加的 `avg_age` 属性，代表有这个兴趣爱好的所有员工的平均年龄。

即使现在不太理解这些语法也没有关系，依然很容易了解到复杂聚合及分组通过 Elasticsearch 特性实现得很完美，能够提取的数据类型也没有任何限制。



## 搜索

*搜索（search）* 可以做到：

- 在类似于 `gender` 或者 `age` 这样的字段上使用结构化查询，`join_date` 这样的字段上使用排序，就像SQL的结构化查询一样。
- 全文检索，找出所有匹配关键字的文档并按照_相关性（relevance）_ 排序后返回结果。
- 以上二者兼而有之。

很多搜索都是开箱即用的，为了充分挖掘 Elasticsearch 的潜力，你需要理解以下三个概念：

- ***映射（Mapping）\***

  描述数据在每个字段内如何存储

- ***分析（Analysis）\***

  全文是如何处理使之可以被搜索的

- ***领域特定查询语言（Query DSL）\***

  Elasticsearch 中强大灵活的查询语言

 

### 空搜索

搜索API的最基础的形式是没有指定任何查询的空搜索，它简单地返回集群中所有索引下的所有文档：

```json
GET /_search
```

返回的结果（为了界面简洁编辑过的）像这样：

```json
{
   "hits" : {
      "total" :     14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       },
        ... 9 RESULTS REMOVED ...
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}
```

#### hits

返回结果中最重要的部分是 `hits` ，它包含 

1. `total` 字段来表示匹配到的文档总数，并且一个 hits 数组包含所查询结果的前十个文档。
2. 在 hits 数组中每个结果包含文档的 `_index` 、 `_type` 、 `_id` ，加上 `_source` 字段。这意味着我们可以直接从返回的搜索结果中使用整个文档。这不像其他的搜索引擎，仅仅返回文档的ID，需要你单独去获取文档。

3. 每个结果还有一个 `_score` ，它衡量了文档与查询的匹配程度。默认情况下，**首先返回最相关的文档结果**，就是说，返回的文档是按照 `_score` 降序排列的。在这个例子中，我们没有指定任何查询，故所有的文档具有相同的相关性，因此对所有的结果而言 `1` 是中性的 `_score` 。

4. `max_score` 值是与查询所匹配文档的 `_score` 的最大值。

#### took

`took` 值告诉我们执行整个搜索请求耗费了多少毫秒。

#### shards

`_shards` 部分告诉我们在查询中参与分片的总数，以及这些分片成功了多少个失败了多少个。正常情况下我们不希望分片失败，但是分片失败是可能发生的。如果我们遭遇到一种灾难级别的故障，在这个故障中丢失了相同分片的原始数据和副本，那么对这个分片将没有可用副本来对搜索请求作出响应。假若这样，Elasticsearch 将报告这个分片是失败的，但是会继续返回剩余分片的结果。

#### timeout

`timed_out` 值告诉我们查询是否超时。默认情况下，搜索请求不会超时。如果低响应时间比完成结果更重要，S你可以指定 `timeout` 为 10 或者 10ms（10毫秒），或者 1s（1秒）：

```js
GET /_search?timeout=10ms
```

在请求超时之前，Elasticsearch 将会返回已经成功从每个分片获取的结果。

> 应当注意的是 `timeout` 不是停止执行查询，它仅仅是告知正在协调的节点返回到目前为止收集的结果并且关闭连接。在后台，其他的分片可能仍在执行查询即使是结果已经被发送了。
>
> 使用超时是因为 SLA(服务等级协议)对你是很重要的，而不是因为想去中止长时间运行的查询。



### 多索引，多类型

如果不对某一特殊的索引或者类型做限制，**就会搜索集群中的所有文档。**Elasticsearch 转发搜索请求到每一个主分片或者副本分片，汇集查询出的前10个结果，并且返回给我们。

然而，经常的情况下，你想在一个或多个特殊的索引并且在一个或者多个特殊的类型中进行搜索。==我们可以通过在URL中指定特殊的索引和类型达到这种效果==，如下所示：

- **`/_search`**
  - 在所有的索引中搜索所有的类型
- **`/gb/_search`**
  - 在 `gb` 索引中搜索所有的类型
- **`/gb,us/_search`**
  - 在 `gb` 和 `us` 索引中搜索所有的文档
- **`/g\*,u\*/_search`**
  - 在任何以 `g` 或者 `u` 开头的索引中搜索所有的类型
- **`/gb/user/_search`**
  - 在 `gb` 索引中搜索 `user` 类型
- **`/gb,us/user,tweet/_search`**
  - 在 `gb` 和 `us` 索引中搜索 `user` 和 `tweet` 类型
- **`/_all/user,tweet/_search`**
  - 在所有的索引中搜索 `user` 和 `tweet` 类型

当在单一的索引下进行搜索的时候，Elasticsearch 转发请求到索引的每个分片中，可以是主分片也可以是副本分片，然后从每个分片中收集结果。多索引搜索恰好也是用相同的方式工作的—只是会涉及到更多的分片。

> 搜索一个索引有五个主分片和搜索五个索引各有一个分片准确来所说是等价的。
>
> 现在不用类型了！



### 分页

![image-20230301093328958](E:/Development/Typora/images/image-20230301093328958.png)

在之前的 [空搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/empty-search.html) 中说明了集群中有 14 个文档匹配了（empty）query 。 但是在 `hits` 数组中只有 10 个文档。如何才能看到其他的文档？

和 SQL 使用 `LIMIT` 关键字返回单个 `page` 结果的方法相同，Elasticsearch 接受 `from` 和 `size` 参数：

- **`size`**
  - 显示应该返回的结果数量，默认是 `10`
- **`from`**
  - 显示应该跳过的初始结果数量，默认是 `0`

如果每页展示 5 条结果，可以用下面方式请求得到 1 到 3 页的结果：

```json
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

考虑到分页过深以及一次请求太多结果的情况，结果集在返回之前先进行排序。 ==但请记住一个请求经常跨越多个分片，每个分片都产生自己的排序结果，这些结果需要进行集中排序以保证整体顺序是正确的。==

### *轻量* 搜索

**代码中不咋用**

有两种形式的 `搜索` API：

- 一种是 “轻量的” *查询字符串* 版本，要求在查询字符串中传递所有的参数
- 另一种是更完整的 *请求体* 版本，要求使用 JSON 格式和更丰富的查询表达式作为搜索语言。

查询字符串搜索非常适用于通过命令行做即席查询。例如，查询在 `tweet` 类型中 `tweet` 字段包含 `elasticsearch` 单词的所有文档：

```json
GET /_all/tweet/_search?q=tweet:elasticsearch
```

下一个查询在 `name` 字段中包含 `john` 并且在 `tweet` 字段中包含 `mary` 的文档。实际的查询就是这样

```
+name:john +tweet:mary
```

但是查询字符串参数所需要的 *百分比编码* （译者注：URL编码）实际上更加难懂：

```
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```

> `+` 前缀表示必须与查询条件匹配。
>
> 类似地， `-` 前缀表示一定不与查询条件匹配。没有 `+` 或者 `-` 的所有其他条件都是可选的——匹配的越多，文档就越相关。

#### _all 字段

这个简单搜索返回包含 `mary` 的所有文档：

```sense
GET /_search?q=mary
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/050_Search/20_All_field.json) 

之前的例子中，我们在 `tweet` 和 `name` 字段中搜索内容。然而，这个查询的结果在三个地方提到了 `mary` ：

- 有一个用户叫做 Mary
- 6条微博发自 Mary
- 一条微博直接 @mary

Elasticsearch 是如何在三个不同的字段中查找到结果的呢？

当索引一个文档的时候，Elasticsearch 取出所有字段的值拼接成一个大的字符串，作为 `_all` 字段进行索引。例如，当索引这个文档时：

```js
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```

这就好似增加了一个名叫 `_all` 的额外字段：

```js
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```

除非设置特定字段，否则查询字符串就使用 `_all` 字段进行搜索。

在刚开始开发一个应用时，`_all` 字段是一个很实用的特性。之后，你会发现如果搜索时用指定字段来代替 `_all` 字段，将会更好控制搜索结果。当 `_all` 字段不再有用的时候，可以将它置为失效，正如在 [元数据: _all 字段](https://www.elastic.co/guide/cn/elasticsearch/guide/current/root-object.html#all-field) 中所解释的。

#### 更复杂的查询

下面的查询针对tweents类型，并使用以下的条件：

- `name` 字段中包含 `mary` 或者 `john`
- `date` 值大于 `2014-09-10`
- `_all` 字段包含 `aggregations` 或者 `geo`

```sense
+name:(mary john) +date:>2014-09-10 +(aggregations geo)
```

查询字符串在做了适当的编码后，可读性很差：

```js
?q=%2Bname%3A(mary+john)+%2Bdate%3A%3E2014-09-10+%2B(aggregations+geo)
```

从之前的例子中可以看出，这种 *轻量* 的查询字符串搜索效果还是挺让人惊喜的。 它的查询语法在相关参考文档中有详细解释，以便简洁的表达很复杂的查询。对于通过命令做一次性查询，或者是在开发阶段，都非常方便。

但同时也可以看到，这种精简让调试更加晦涩和困难。而且很脆弱，一些查询字符串中很小的语法错误，像 `-` ， `:` ， `/` 或者 `"` 不匹配等，将会返回错误而不是搜索结果。

最后，查询字符串搜索允许任何用户在索引的任意字段上执行可能较慢且重量级的查询，这可能会暴露隐私信息，甚至将集群拖垮。

因为这些原因，不推荐直接向用户暴露查询字符串搜索功能，除非对于集群和数据来说非常信任他们。

相反，我们经常在生产环境中更多地使用功能全面的 *request body* 查询API，除了能完成以上所有功能，还有一些附加功能。但在到达那个阶段之前，首先需要了解数据在 Elasticsearch 中是如何被索引的。



## 映射和分析

### 简介

当摆弄索引里面的数据时，我们发现一些奇怪的事情。一些事情看起来被打乱了：在我们的索引中有12条推文，其中只有一条包含日期 `2014-09-15` ，但是看一看下面查询命中的 `总数` （total）：

```js
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results !
GET /_search?q=date:2014-09-15   # 1  result
GET /_search?q=date:2014         # 0  results !
```

为什么在 [`_all` ](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-lite.html#all-field-intro)字段查询日期返回所有推文，而在 `date` 字段只查询年份却没有返回结果？为什么我们在 `_all` 字段和 `date` 字段的查询结果有差别？

推测起来，这是因为数据在 `_all` 字段与 `date` 字段的索引方式不同。所以，通过请求 `gb` 索引中 `tweet` 类型的*映射*（或模式定义），让我们看一看 Elasticsearch 是如何解释我们文档结构的：

```js
GET /gb/_mapping/tweet
```

这将得到如下结果：

```json
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

基于对字段类型的猜测， Elasticsearch 动态为我们产生了一个映射。这个响应告诉我们 `date` 字段被认为是 `date` 类型的。由于 `_all` 是默认字段，所以没有提及它。但是我们知道 `_all` 字段是 `string` 类型的。

所以 `date` 字段和 `string` 字段索引方式不同，因此搜索结果也不一样。这完全不令人吃惊。你可能会认为核心数据类型 strings、numbers、Booleans 和 dates 的索引方式有稍许不同。没错，他们确实稍有不同。

但是，到目前为止，最大的差异在于代表 *精确值* （它包括 `string` 字段）的字段和代表 *全文* 的字段。这个区别非常重要——它将搜索引擎和所有其他数据库区别开来。

### 精确值 VS 全文

Elasticsearch 中的数据可以概括的分为两类：**精确值和全文**。

- *精确值* 如它们听起来那样精确。例如日期或者用户 ID，但字符串也可以表示精确值，例如用户名或邮箱地址。对于精确值来讲，`Foo` 和 `foo` 是不同的，`2014` 和 `2014-09-15` 也是不同的。
- *全文* 是指文本数据（通常以人类容易识别的语言书写），例如一个推文的内容或一封邮件的内容。
  - 全文通常是指非结构化的数据，但这里有一个误解：自然语言是高度结构化的。问题在于自然语言的规则是复杂的，导致计算机难以正确解析。例如，考虑这条语句：

```
May is fun but June bores me.
```

它指的是月份还是人？

精确值很容易查询。结果是二进制的：要么匹配查询，要么不匹配。这种查询很容易用 SQL 表示：

```js
WHERE name    = "John Smith"
  AND user_id = 2
  AND date    > "2014-09-15"
```

查询全文数据要微妙的多。我们问的不只是“这个文档匹配查询吗”，而是“该文档匹配查询的程度有多大？”换句话说，该文档与给定查询的相关性如何？

我们很少对全文类型的域做精确匹配。相反，我们希望在文本类型的域中搜索。不仅如此，我们还希望搜索能够理解我们的 *意图* ：

- 搜索 `UK` ，会返回包含 `United Kindom` 的文档。
- 搜索 `jump` ，会匹配 `jumped` ， `jumps` ， `jumping` ，甚至是 `leap` 。
- 搜索 `johnny walker` 会匹配 `Johnnie Walker` ， `johnnie depp` 应该匹配 `Johnny Depp` 。
- `fox news hunting` 应该返回福克斯新闻（ Foxs News ）中关于狩猎的故事，同时， `fox hunting news` 应该返回关于猎狐的故事。

为了促进这类在全文域中的查询，Elasticsearch 首先 *分析* 文档，之后根据结果创建 *倒排索引* 。

### 倒排索引

Elasticsearch 使用一种称为 ***倒排索引*** 的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

例如，假设我们有两个文档，每个文档的 `content` 域包含如下内容：

1. The quick brown fox jumped over the lazy dog
2. Quick brown foxes leap over lazy dogs in summer

为了创建倒排索引，我们首先将每个文档的 `content` 域拆分成单独的 词（我们称它为 `词条` 或 `tokens` ），创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。结果如下所示：

```js
Term      Doc_1  Doc_2
-------------------------
Quick   |       |  X
The     |   X   |
brown   |   X   |  X
dog     |   X   |
dogs    |       |  X
fox     |   X   |
foxes   |       |  X
in      |       |  X
jumped  |   X   |
lazy    |   X   |  X
leap    |       |  X
over    |   X   |  X
quick   |   X   |
summer  |       |  X
the     |   X   |
------------------------
```

现在，如果我们想搜索 `quick brown` ，我们只需要查找包含每个词条的文档：

```js
Term      Doc_1  Doc_2
-------------------------
brown   |   X   |  X
quick   |   X   |
------------------------
Total   |   2   |  1
```

两个文档都匹配，但是第一个文档比第二个匹配度更高。如果我们使用仅计算匹配词条数量的简单 *相似性算法* ，那么，我们可以说，对于我们查询的相关性来讲，第一个文档比第二个文档更佳。

但是，我们目前的倒排索引有一些问题：

- `Quick` 和 `quick` 以独立的词条出现，然而用户可能认为它们是相同的词。
- `fox` 和 `foxes` 非常相似, 就像 `dog` 和 `dogs` ；他们有相同的词根。
- `jumped` 和 `leap`, 尽管没有相同的词根，但他们的意思很相近。他们是同义词。

使用前面的索引搜索 `+Quick +fox` 不会得到任何匹配文档。（记住，`+` 前缀表明这个词必须存在。）只有同时出现 `Quick` 和 `fox` 的文档才满足这个查询条件，但是第一个文档包含 `quick fox` ，第二个文档包含 `Quick foxes` 。

我们的用户可以合理的期望两个文档与查询匹配。我们可以做的更好。

如果我们将词条规范为标准模式，那么我们可以找到与用户搜索的词条不完全一致，但具有足够相关性的文档。例如：

- `Quick` 可以小写化为 `quick` 。
- `foxes` 可以 *词干提取* --变为词根的格式-- 为 `fox` 。类似的， `dogs` 可以为提取为 `dog` 。
- `jumped` 和 `leap` 是同义词，可以索引为相同的单词 `jump` 。

现在索引看上去像这样：

```js
Term      Doc_1  Doc_2
-------------------------
brown   |   X   |  X
dog     |   X   |  X
fox     |   X   |  X
in      |       |  X
jump    |   X   |  X
lazy    |   X   |  X
over    |   X   |  X
quick   |   X   |  X
summer  |       |  X
the     |   X   |  X
------------------------
```

这还远远不够。我们搜索 `+Quick +fox` *仍然* 会失败，因为在我们的索引中，已经没有 `Quick` 了。但是，如果我们对搜索的字符串使用与 `content` 域相同的标准化规则，会变成查询 `+quick +fox` ，这样两个文档都会匹配！

> 这非常重要。你只能搜索在索引中出现的词条，所以索引文本和查询字符串必须标准化为相同的格式。

分词和标准化的过程称为 *分析* 

### 分析与分析器

*分析* 包含下面的过程：

- 首先，将一块文本分成适合于倒排索引的独立的 *词条* ，
- 之后，将这些词条**统一化为标准格式**以提高它们的“可搜索性”，或者 *recall*

分析器执行上面的工作。 *分析器* 实际上是将三个功能封装到了一个包里：

- **字符过滤器**

  首先，字符串按顺序通过每个 *字符过滤器* 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 `&` 转化成 `and`。

- **分词器**

  其次，字符串被 *分词器* 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

- **Token 过滤器**

  最后，词条按顺序通过每个 *token 过滤器* 。这个过程可能会改变词条（例如，小写化 `Quick` ），删除词条（例如， 像 `a`， `and`， `the` 等无用词），或者增加词条（例如，像 `jump` 和 `leap` 这种同义词）。

Elasticsearch提供了开箱即用的字符过滤器、分词器和token 过滤器。 这些可以组合起来形成自定义的分析器以用于不同的目的。我们会在 [自定义分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html) 章节详细讨论。

#### 内置分析器

但是， Elasticsearch还附带了可以直接使用的预包装的分析器。接下来我们会列出最重要的分析器。为了证明它们的差异，我们看看每个分析器会从下面的字符串得到哪些词条：

```js
"Set the shape to semi-transparent by calling set_trans(5)"
```

- **标准分析器**

  标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 [Unicode 联盟](http://www.unicode.org/reports/tr29/) 定义的 *单词边界* 划分文本。删除绝大部分标点。最后，将词条小写。它会产生`set, the, shape, to, semi, transparent, by, calling, set_trans, 5`

- **简单分析器**

  简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生`set, the, shape, to, semi, transparent, by, calling, set, trans`

- **空格分析器**

  空格分析器在空格的地方划分文本。它会产生`Set, the, shape, to, semi-transparent, by, calling, set_trans(5)`

- **语言分析器**

  特定语言分析器可用于 [很多语言](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-lang-analyzer.html)。它们可以考虑指定语言的特点。例如， `英语` 分析器附带了一组英语无用词（常用单词，例如 `and` 或者 `the` ，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 *词干* 。`英语` 分词器会产生下面的词条：`set, shape, semi, transpar, call, set_tran, 5`注意看 `transparent`、 `calling` 和 `set_trans` 已经变为词根格式。

#### 什么时候使用分析器

当我们 *索引* 一个文档，它的全文域被分析成词条以用来创建倒排索引。 但是，当我们在全文域 *搜索* 的时候，我们需要将查询字符串通过 *相同的分析过程* ，以保证我们搜索的词条格式与索引中的词条格式一致。

全文查询，理解每个域是如何定义的，因此它们可以做正确的事：

- **当你查询一个 *全文* 域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。**
- **当你查询一个 *精确值* 域时，不会分析查询字符串，而是搜索你指定的精确值。**

现在你可以理解在 [开始章节](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-analysis.html) 的查询为什么返回那样的结果：

- `date` 域包含一个**精确值**：单独的词条 `2014-09-15`。
- `_all` 域是一个**全文域**，所以分词进程将日期转化为三个词条：`2014`， `09`， 和 `15`。

当我们在 `_all` 域查询 `2014`，它匹配所有的12条推文，因为它们都含有 `2014` ：

```js
GET /_search?q=2014              # 12 results
```

当我们在 `_all` 域查询 `2014-09-15`，它首先分析查询字符串，产生匹配 `2014`， `09`， 或 `15` 中 *任意* 词条的查询。这也会匹配所有12条推文，因为它们都含有 `2014` ：

```js
GET /_search?q=2014-09-15        # 12 results !
```

当我们在 `date` 域查询 `2014-09-15`，它寻找 *精确* 日期，只找到一个推文：

```js
GET /_search?q=date:2014-09-15   # 1  resul
```

当我们在 `date` 域查询 `2014`，它找不到任何文档，因为没有文档含有这个精确日志：

```js
GET /_search?q=date:2014         # 0  results !
```

有些时候很难理解分词的过程和实际被存储到索引中的词条，特别是你刚接触Elasticsearch。为了理解发生了什么，你可以使用 `analyze` API 来看文本是如何被分析的。在消息体里，指定分析器和要分析的文本：

```js
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
```

结果中每个元素代表一个单独的词条：

```js
{
   "tokens": [
      {
         "token":        "text",
         "start_offset": 0,
         "end_offset":   4,
         "type":         "<ALPHANUM>",
         "position":     1
      },
      {
         "token":        "to",
         "start_offset": 5,
         "end_offset":   7,
         "type":         "<ALPHANUM>",
         "position":     2
      },
      {
         "token":        "analyze",
         "start_offset": 8,
         "end_offset":   15,
         "type":         "<ALPHANUM>",
         "position":     3
      }
   ]
}
```

`token` 是实际存储到索引中的词条。 `position` 指明词条在原始文本中出现的位置。 `start_offset` 和 `end_offset` 指明字符在原始字符串中的位置。

每个分析器的 `type` 值都不一样，可以忽略它们。它们在Elasticsearch中的唯一作用在于[`keep_types` token 过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-keep-types-tokenfilter.html)。

`analyze` API 是一个有用的工具，它有助于我们理解Elasticsearch索引内部发生了什么，随着深入，我们会进一步讨论它。

#### 指定分析器

当Elasticsearch在你的文档中检测到一个新的字符串域，它会自动设置其为一个全文 `字符串` 域，使用 `标准` 分析器对它进行分析。

你不希望总是这样。可能你想使用一个不同的分析器，适用于你的数据使用的语言。有时候你想要一个字符串域就是一个字符串域—不使用分析，直接索引你传入的精确值，例如用户ID或者一个内部的状态域或标签。

要做到这一点，我们必须手动指定这些域的映射。

### 映射

为了能够将时间域视为时间，数字域视为数字，字符串域视为全文或精确值字符串， Elasticsearch 需要知道每个域中数据的类型。这个信息包含在映射中。

如 [*数据输入和输出*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/data-in-data-out.html) 中解释的，索引中每个文档都有 *类型* 。每种类型都有它自己的 *映射* ，或者 *模式定义* 。映射定义了类型中的域，每个域的数据类型，以及Elasticsearch如何处理这些域。映射也用于配置与类型有关的元数据。

我们会在 [类型和映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping.html) 详细讨论映射。本节，我们只讨论足够让你入门的内容。

#### 核心简单域类型

Elasticsearch 支持如下简单域类型：

- 字符串: `string`
- 整数 : `byte`, `short`, `integer`, `long`
- 浮点数: `float`, `double`
- 布尔型: `boolean`
- 日期: `date`

当你索引一个包含新域的文档—之前未曾出现-- Elasticsearch 会使用 [*动态映射*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html) ，通过JSON中基本数据类型，尝试猜测域类型，使用如下规则：

| **JSON type**                  | **域 type** |
| ------------------------------ | ----------- |
| 布尔型: `true` 或者 `false`    | `boolean`   |
| 整数: `123`                    | `long`      |
| 浮点数: `123.45`               | `double`    |
| 字符串，有效日期: `2014-09-15` | `date`      |
| 字符串: `foo bar`              | `string`    |

这意味着如果你通过引号( `"123"` )索引一个数字，它会被映射为 `string` 类型，而不是 `long` 。但是，如果这个域已经映射为 `long` ，那么 Elasticsearch 会尝试将这个字符串转化为 long ，如果无法转化，则抛出一个异常。

#### 查看映射

通过 `/_mapping` ，我们可以查看 Elasticsearch 在一个或多个索引中的一个或多个类型的映射。在 [开始章节](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-analysis.html) ，我们已经取得索引 `gb` 中类型 `tweet` 的映射：

```js
GET /gb/_mapping/tweet
```

Elasticsearch 根据我们索引的文档，为域(称为 *属性* )动态生成的映射。

```json
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

错误的映射，例如 将 `age` 域映射为 `string` 类型，而不是 `integer` ，会导致查询出现令人困惑的结果。

检查一下！而不是假设你的映射是正确的。

#### 自定义域映射

尽管在很多情况下基本域数据类型已经够用，但你经常需要为单独域自定义映射，特别是字符串域。自定义映射允许你执行下面的操作：

- 全文字符串域和精确值字符串域的区别
- **使用特定语言分析器**
- 优化域以适应部分匹配
- 指定自定义数据格式
- 还有更多

域最重要的属性是 `type` 。==对于不是 `string` 的域，你一般只需要设置 `type` ：==

```js
{
    "number_of_clicks": {
        "type": "integer"
    }
}
```

默认， `string` 类型域会被认为包含全文。就是说，它们的值在索引前，会通过一个分析器，针对于这个域的查询在搜索前也会经过一个分析器。

==`string` 域映射的两个最重要属性是 `index` 和 `analyzer` 。==

##### index

`index` 属性控制怎样索引字符串。它可以是下面三个值：

- **`analyzed`**
  - 首先分析字符串，然后索引它。换句话说，以全文索引这个域。
  
- **`not_analyzed`**
   - 索引这个域，所以它能够被搜索，但索引的是精确值。**不会对它进行分析。**

- **`no`**
  - 不索引这个域。这个域不会被搜索到。


`string` 域 `index` 属性默认是 `analyzed` 。如果我们想映射这个字段为一个精确值，我们需要设置它为 `not_analyzed` ：

```json
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}
```

其他简单类型（例如 `long` ， `double` ， `date` 等）也接受 `index` 参数，但有意义的值只有 `no` 和 `not_analyzed` ， 因为它们永远不会被分析。

##### analyzer

对于 `analyzed` 字符域，用 `analyzer` 属性指定在搜索和索引时使用的分析器。默认， Elasticsearch 使用 `standard` 分析器， 但你可以指定一个内置的分析器替代它，例如 `whitespace` 、 `simple` 和 `english`：

```js
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```

在 [自定义分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html) ，我们会展示怎样定义和使用自定义分析器。

#### 更新映射

当你首次创建一个索引的时候，可以指定类型的映射。你也可以使用 `/_mapping` 为新类型（或者为存在的类型更新映射）增加映射。

> 尽管你可以 *增加* 一个存在的映射，你不能 *修改* 存在的域映射。如果一个域的映射已经存在，那么该域的数据可能已经被索引。如果你意图修改这个域的映射，索引的数据可能会出错，不能被正常的搜索。
>

我们可以更新一个映射来添加一个新域，但不能将一个存在的域从 `analyzed` 改为 `not_analyzed` 。

为了描述指定映射的两种方式，我们先删除 `gd` 索引：

```js
DELETE /gb
```

然后创建一个新索引，指定 `tweet` 域使用 `english` 分析器：

```js
PUT /gb //①
{
  "mappings": {
    "tweet" : {
      "properties" : {
        "tweet" : {
          "type" :    "string",
          "analyzer": "english"
        },
        "date" : {
          "type" :   "date"
        },
        "name" : {
          "type" :   "string"
        },
        "user_id" : {
          "type" :   "long"
        }
      }
    }
  }
}
```

  ①通过消息体中指定的 `mappings` 创建了索引。

稍后，我们决定在 `tweet` 映射增加一个新的名为 `tag` 的 `not_analyzed` 的文本域，使用 `_mapping` ：

```json
PUT /gb/_mapping/tweet
{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}
```

注意，我们不需要再次列出所有已存在的域，因为无论如何我们都无法改变它们。新域已经被合并到存在的映射中。

#### 测试映射

你可以使用 `analyze` API 测试字符串域的映射。比较下面两个请求的输出：

```json
GET /gb/_analyze
{
  "field": "tweet",
  "text": "Black-cats" //①
}

GET /gb/_analyze
{
  "field": "tag",
  "text": "Black-cats" //①
}
```

①消息体里面传输我们想要分析的文本。

`tweet` 域产生两个词条 `black` 和 `cat` ， `tag` 域产生单独的词条 `Black-cats` 。换句话说，我们的映射正常工作。



### 复杂核心域类型

除了我们提到的简单标量数据类型， JSON 还有 `null` 值，数组，和对象，这些 Elasticsearch 都是支持的。

多值域

很有可能，我们希望 `tag` 域包含多个标签。我们可以以数组的形式索引标签：

```js
{ "tag": [ "search", "nosql" ]}
```

对于数组，没有特殊的映射需求。任何域都可以包含0、1或者多个值，就像全文域分析得到多个词条。

==这暗示 *数组中所有的值必须是相同数据类型的*==。你不能将日期和字符串混在一起。如果你通过索引数组来创建新的域，Elasticsearch 会用数组中第一个值S的数据类型作为这个域的 `类型` 。

> 当你从 Elasticsearch 得到一个文档，每个数组的顺序和你当初索引文档时一样。你得到的 `_source` 域，包含与你索引的一模一样的 JSON 文档。
>
> 但是，数组是以多值域 *索引的*—可以搜索，但是无序的。 在搜索的时候，你不能指定 “第一个” 或者 “最后一个”。 更确切的说，把数组想象成 *装在袋子里的值* 。

#### 空域

**当然，数组可以为空。这相当于存在零值。**

事实上，在 Lucene 中是不能存储 `null` 值的，所以我们认为存在 `null` 值的域为空域。

下面三种域被认为是空的，它们将不会被索引：

```js
"null_value":               null,
"empty_array":              [],
"array_with_null_value":    [ null ]
```

#### 多层级对象

我们讨论的最后一个 JSON 原生数据类是 *对象* -- **在其他语言中称为哈希，哈希 map，字典或者关联数组。**

*内部对象* 经常用于嵌入一个实体或对象到其它对象中。例如，与其在 `tweet` 文档中包含 `user_name` 和 `user_id` 域，我们也可以这样写：

```js
{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}
```

#### 内部对象的映射

Elasticsearch 会动态监测新的对象域并映射它们为 `对象` ，在 `properties` 属性下列出内部域：

```js
{
  "gb": {
    "tweet": { //①
      "properties": {
        "tweet":            { "type": "string" },
        "user": {  //②
          "type":             "object",
          "properties": {
            "id":           { "type": "string" },
            "gender":       { "type": "string" },
            "age":          { "type": "long"   },
            "name":   {  //②
              "type":         "object",
              "properties": {
                "full":     { "type": "string" },
                "first":    { "type": "string" },
                "last":     { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
```

![image-20230301104744616](E:/Development/Typora/images/image-20230301104744616.png)

`user` 和 `name` 域的映射结构与 `tweet` 类型的相同。事实上， `type` 映射只是一种特殊的 `对象` 映射，我们称之为 *根对象* 。除了它有一些文档元数据的特殊顶级域，例如 `_source` 和 `_all` 域，它和其他对象一样。

#### 内部对象是如何索引的

Lucene 不理解内部对象。 Lucene 文档是由一组键值对列表组成的。为了能让 Elasticsearch 有效地索引内部类，它把我们的文档转化成这样：

```js
{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
```

*内部域* 可以通过名称引用（例如， `first` ）。为了区分同名的两个域，我们可以使用全 *路径* （例如， `user.name.first` ） 或 `type` 名加路径（ `tweet.user.name.first` ）。

在前面简单扁平的文档中，没有 `user` 和 `user.name` 域。Lucene 索引只有标量和简单值，没有复杂数据结构。

#### 内部对象数组

最后，考虑包含内部对象的数组是如何被索引的。 假设我们有个 `followers` 数组：

```js
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
```

这个文档会像我们之前描述的那样被==扁平化处理==，结果如下所示：

```js
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
```

`{age: 35}` 和 `{name: Mary White}` 之间的相关性已经丢失了，因为每个多值域只是一包无序的值，而不是有序数组。这足以让我们问，“有一个26岁的追随者？”

但是我们不能得到一个准确的答案：“是否有一个26岁 *名字叫 Alex Jones* 的追随者？”

相关内部对象被称为 *nested* 对象，可以回答上面的查询，我们稍后会在[*嵌套对象*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/nested-objects.html)中介绍它。



## 请求体查询

### 空查询

让我们以最简单的 `search` API 的形式开启我们的旅程，空查询将返回所有索引库（indices)中的所有文档：

```json
GET /_search
{} //① 
```

>   ① 这是一个空的请求体。

只用一个查询字符串，你就可以在一个、多个或者 `_all` 索引库（indices）和一个、多个或者所有types中查询：

```js
GET /index_2014*/type1,type2/_search
{}
```

同时你可以使用 `from` 和 `size` 参数来分页：

```js
GET /_search
{
  "from": 30,
  "size": 10
}
```



**一个带请求体的 GET 请求？**

某些特定语言（特别是 JavaScript）的 HTTP 库是不允许 `GET` 请求带有请求体的。事实上，一些使用者对于 `GET` 请求可以带请求体感到非常的吃惊。

而事实是这个RFC文档 [RFC 7231](http://tools.ietf.org/html/rfc7231#page-24)— 一个专门负责处理 HTTP 语义和内容的文档 — 并没有规定一个带有请求体的 `GET` 请求应该如何处理！结果是，一些 HTTP 服务器允许这样子，而有一些 — 特别是一些用于缓存和代理的服务器 — 则不允许。

对于一个查询请求，Elasticsearch 的工程师偏向于使用 `GET` 方式，因为他们觉得它比 `POST` 能更好的描述信息检索（retrieving information）的行为。然而，因为带请求体的 `GET` 请求并不被广泛支持，所以 `search` API同时支持 `POST` 请求：

```js
POST /_search
{
  "from": 30,
  "size": 10
}
```

类似的规则可以应用于任何需要带请求体的 `GET` API。

相对于使用晦涩难懂的查询字符串的方式，一个带请求体的查询允许我们使用 *查询领域特定语言（query domain-specific language）* 或者 Query DSL 来写查询语句。



### 查询表达式

查询表达式(Query DSL)是一种非常灵活又富有表现力的 查询语言。 Elasticsearch 使用它可以以简单的 JSON 接口来展现 Lucene 功能的绝大部分。

要使用这种查询表达式，只需将查询语句传递给 `query` 参数：

```js
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```

*空查询（empty search）* —`{}`— 在功能上等价于使用 `match_all` 查询， **正如其名字一样，匹配所有文档：**

```json
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

#### 查询语句的结构

一个查询语句的典型结构：

```json
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```

如果是针对某个字段，那么它的结构如下：

```json
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

举个例子，你可以使用 `match` 查询语句 来查询 `tweet` 字段中包含 `elasticsearch` 的 tweet：

```js
{
    "match": {
        "tweet": "elasticsearch"
    }
}
```

完整的查询请求如下：

```json
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

#### 合并查询语句

*查询语句(Query clauses)* 就像一些简单的组合块，这些组合块可以彼此之间合并组成更复杂的查询。这些语句可以是如下形式：

- *叶子语句（Leaf clauses）* (就像 `match` 语句) 被用于将查询字符串和一个字段（或者多个字段）对比。
- *复合(Compound)* 语句 主要用于 合并其它查询语句。 比如，一个 `bool` 语句 允许在你需要的时候组合其它语句，无论是 `must` 匹配、 `must_not` 匹配还是 `should` 匹配，同时它可以包含不评分的过滤器（filters）：

```json
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }},
        "filter":   { "range": { "age" : { "gt" : 30 }} }
    }
}
```

一条复合语句可以合并 *任何* 其它查询语句，包括复合语句，了解这一点是很重要的。这就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

例如，以下查询是为了找出信件正文包含 `business opportunity` 的星标邮件，或者在收件箱正文包含 `business opportunity` 的非垃圾邮件：

```js
{
    "bool": {
        "must": { "match":   { "email": "business opportunity" }},
        "should": [
            { "match":       { "starred": true }},
            { "bool": {
                "must":      { "match": { "folder": "inbox" }},
                "must_not":  { "match": { "spam": true }}
            }}
        ],
        "minimum_should_match": 1
    }
}
```

**一条复合语句可以将多条语句 — 叶子语句和其它复合语句 — 合并成一个单一的查询语句。**



### 常用查询

#### match_all 查询

`match_all` 查询简单的匹配所有文档。在没有指定查询方式时，它是默认的查询：

```sense
{ "match_all": {}}
```

它经常与 filter 结合使用—例如，检索收件箱里的所有邮件。所有邮件被认为具有相同的相关性，所以都将获得分值为 `1` 的中性 `_score`。

#### match 查询

无论你在任何字段上进行的是全文搜索还是精确查询，`match` 查询是你可用的标准查询。

如果你在一个全文字段上使用 `match` 查询，在执行查询前，它将用正确的分析器去分析查询字符串：

```sense
{ "match": { "tweet": "About Search" }}
```

如果在一个精确值的字段上使用它，例如数字、日期、布尔或者一个 `not_analyzed` 字符串字段，那么它将会精确匹配给定的值：

```sense
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```

对于精确值的查询，你可能需要使用 filter 语句来取代 query，因为 filter 将会被缓存。接下来，我们将看到一些关于 filter 的例子。

不像我们在 [*轻量* 搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-lite.html) 章节介绍的字符串查询（query-string search）， `match` 查询不使用类似 `+user_id:2 +tweet:search` 的查询语法。它只是去查找给定的单词。这就意味着将查询字段暴露给你的用户是安全的；你需要控制那些允许被查询字段，不易于抛出语法异常。

#### multi_match 查询

`multi_match` 查询可以在多个字段上执行相同的 `match` 查询：

```sense
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```

#### range 查询

`range` 查询找出那些落在指定区间内的数字或者时间：

```sense
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```

被允许的操作符如下：

- **`gt`**

  大于

- **`gte`**

  大于等于

- **`lt`**

  小于

- **`lte`**

  小于等于

#### term 查询

`term` 查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 `not_analyzed` 的字符串：

```sense
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```

`term` 查询对于输入的文本不 *分析* ，所以它将给定的值进行精确查询。

#### terms 查询

`terms` 查询和 `term` 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件：

```sense
{ "terms": { "tag": [ "search", "full_text", "nosql" ] }}
```

和 `term` 查询一样，`terms` 查询对于输入的文本不分析。它查询那些精确匹配的值（包括在大小写、重音、空格等方面的差异）。

#### exists 查询和 missing 查询

`exists` 查询和 `missing` 查询被用于查找那些指定字段中有值 (`exists`) 或无值 (`missing`) 的文档。这与SQL中的 `IS_NULL` (`missing`) 和 `NOT IS_NULL` (`exists`) 在本质上具有共性：

```sense
{
    "exists":   {
        "field":    "title"
    }
}
```

这些查询经常用于某个字段有值的情况和某个字段缺值的情况。

### 组合多查询

现实的查询需求从来都没有那么简单；它们需要在多个字段上查询多种多样的文本，并且根据一系列的标准来过滤。为了构建类似的高级查询，你需要一种能够将多查询组合成单一查询的查询方法。

你可以用 `bool` 查询来实现你的需求。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。它接收以下参数：

- **`must`**

  文档 *必须* 匹配这些条件才能被包含进来。

- **`must_not`**

  文档 *必须不* 匹配这些条件才能被包含进来。

- **`should`**

  如果满足这些语句中的任意语句，将增加 `_score` ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。

- **`filter`**

  *必须* 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

由于这是我们看到的第一个包含多个查询的查询，所以有必要讨论一下相关性得分是如何组合的。每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， `bool` 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

下面的查询用于查找 `title` 字段匹配 `how to make millions` 并且不被标识为 `spam` 的文档。那些被标识为 `starred` 或在2014之后的文档，将比另外那些文档拥有更高的排名。如果 *两者* 都满足，那么它排名将更高：

```sense
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

如果没有 `must` 语句，那么至少需要能够匹配其中的一条 `should` 语句。但，如果存在至少一条 `must` 语句，则对 `should` 语句的匹配没有要求。

#### 增加带过滤器（filtering）的查询

如果我们不想因为文档的时间而影响得分，可以用 `filter` 语句来重写前面的例子：

```sense
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }} //①
        }
    }
}
```

> ①range 查询已经从 `should` 语句中移到 `filter` 语句

通过将 range 查询移到 `filter` 语句中，我们将它转成不评分的查询，将不再影响文档的相关性排名。由于它现在是一个不评分的查询，可以使用各种对 filter 查询有效的优化手段来提升性能。

所有查询都可以借鉴这种方式。将查询移到 `bool` 查询的 `filter` 语句中，这样它就自动的转成一个不评分的 filter 了。

如果你需要通过多个不同的标准来过滤你的文档，`bool` 查询本身也可以被用做不评分的查询。简单地将它放置到 `filter` 语句中并在内部构建布尔逻辑：

```sense
{
    "bool": { 
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": {  //①
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```

> ①将 `bool` 查询包裹在 `filter` 语句中，我们可以在过滤标准中增加布尔逻辑

通过混合布尔查询，我们可以在我们的查询请求中灵活地编写 scoring 和 filtering 查询逻辑。

#### constant_score 查询

尽管没有 `bool` 查询使用这么频繁，`constant_score` 查询也是你工具箱里有用的查询工具。它将一个不变的常量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 filter 而没有其它查询（例如，评分查询）的情况下。

可以使用它来取代只有 filter 语句的 `bool` 查询。在性能上是完全相同的，但对于提高查询简洁性和清晰度有很大帮助。

```sense
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } //①
        }
    }
}
```

> ①`term` 查询被放置在 `constant_score` 中，转成不评分的 filter。这种方式可以用来取代只有 filter 语句的 `bool` 查询。

### 验证查询

查询可以变得非常的复杂，尤其和不同的分析器与不同的字段映射结合时，理解起来就有点困难了。不过 `validate-query` API 可以用来验证查询是否合法。

```sense
GET /gb/tweet/_validate/query
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```



拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/80_Validate_query.json) 

以上 `validate` 请求的应答告诉我们这个查询是不合法的：

```js
{
  "valid" :         false,
  "_shards" : {
    "total" :       1,
    "successful" :  1,
    "failed" :      0
  }
}
```



#### 理解错误信息

为了找出 查询不合法的原因，可以将 `explain` 参数 加到查询字符串中：

```json
GET /gb/tweet/_validate/query?explain  //①
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```

> ①`explain` 参数可以提供更多关于查询不合法的信息。

很明显，我们将查询类型(`match`)与字段名称 (`tweet`)搞混了：

```js
{
  "valid" :     false,
  "_shards" :   { ... },
  "explanations" : [ {
    "index" :   "gb",
    "valid" :   false,
    "error" :   "org.elasticsearch.index.query.QueryParsingException:
                 [gb] No query registered for [tweet]"
  } ]
}
```

#### 理查询语句

对于合法查询，使用 `explain` 参数将返回可读的描述，这对准确理解 Elasticsearch 是如何解析你的 query 是非常有用的：

```sense
GET /_validate/query?explain
{
   "query": {
      "match" : {
         "tweet" : "really powerful"
      }
   }
}
```

我们查询的每一个 index 都会返回对应的 `explanation` ，因为每一个 index 都有自己的映射和分析器：

```js
{
  "valid" :         true,
  "_shards" :       { ... },
  "explanations" : [ {
    "index" :       "us",
    "valid" :       true,
    "explanation" : "tweet:really tweet:powerful"
  }, {
    "index" :       "gb",
    "valid" :       true,
    "explanation" : "tweet:realli tweet:power"
  } ]
}
```

从 `explanation` 中可以看出，匹配 `really powerful` 的 `match` 查询被重写为两个针对 `tweet` 字段的 single-term 查询，一个single-term查询对应查询字符串分出来的一个term。

当然，对于索引 `us` ，这两个 term 分别是 `really` 和 `powerful` ，而对于索引 `gb` ，term 则分别是 `realli` 和 `power` 。之所以出现这个情况，是由于我们将索引 `gb` 中 `tweet` 字段的分析器修改为 `english` 分析器。





## 排序与相关性

### 排序

为了按照相关性来排序，需要将相关性表示为一个数值。在 Elasticsearch 中， *相关性得分* 由一个浮点数进行表示，并在搜索结果中通过 `_score` 参数返回， 默认排序是 `_score` 降序。

有时，相关性评分对你来说并没有意义。例如，下面的查询返回所有 `user_id` 字段包含 `1` 的结果：

```js
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```

这里没有一个有意义的分数：因为我们使用的是 filter （过滤），这表明我们只希望获取匹配 `user_id: 1` 的文档，并没有试图确定这些文档的相关性。 实际上文档将按照随机顺序返回，并且每个文档都会评为零分。

如果评分为零对你造成了困扰，你可以使用 `constant_score` 查询进行替代：

```js
GET /_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```

这将让所有文档应用一个恒定分数（默认为 `1` ）。它将执行与前述查询相同的查询，并且所有的文档将像之前一样随机返回，这些文档只是有了一个分数而不是零分。

#### 按照字段的值排序

在这个案例中，通过时间来对 tweets 进行排序是有意义的，最新的 tweets 排在最前。 我们可以使用 `sort` 参数进行实现：

```sense
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "user_id" : 1 }}
        }
    },
    "sort": { "date": { "order": "desc" }}
}
```

你会注意到结果中的两个不同点：

```json
"hits" : {
    "total" :           6,
    "max_score" :       null,  //①
    "hits" : [ {
        "_index" :      "us",
        "_type" :       "tweet",
        "_id" :         "14",
        "_score" :      null,  //①
        "_source" :     {
             "date":    "2014-09-24",
             ...
        },
        "sort" :        [ 1411516800000 ] //②
    },
    ...
}
```

![image-20230301113046129](E:/Development/Typora/images/image-20230301113046129.png)

首先我们在每个结果中有一个新的名为 `sort` 的元素，它包含了我们用于排序的值。 在这个案例中，我们按照 `date` 进行排序，在内部被索引为 *自 epoch 以来的毫秒数* 。 long 类型数 `1411516800000` 等价于日期字符串 `2014-09-24 00:00:00 UTC` 。

其次 `_score` 和 `max_score` 字段都是 `null` 。计算 `_score` 的花销巨大，通常仅用于排序； 我们并不根据相关性排序，所以记录 `_score` 是没有意义的。如果无论如何你都要计算 `_score` ， 你可以将 `track_scores` 参数设置为 `true` 。

> 一个简便方法是, 你可以指定一个字段用来排序：

```js
    "sort": "number_of_children"
```

字段将会默认升序排序，而按照 `_score` 的值进行降序排序。

#### 多级排序

假定我们想要结合使用 `date` 和 `_score` 进行查询，并且匹配的结果首先按照日期排序，然后按照相关性排序：

```json
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }}
    ]
}
```

排序条件的顺序是很重要的。结果首先按第一个条件排序，仅当结果集的第一个 `sort` 值完全相同时才会按照第二个条件进行排序，以此类推。

多级排序并不一定包含 `_score` 。你可以根据一些不同的字段进行排序，如地理距离或是脚本计算的特定值。

> Query-string 搜索 也支持自定义排序，可以在查询字符串中使用 `sort` 参数：

```js
GET /_search?sort=date:desc&sort=_score&q=search
```

#### 多值字段的排序

一种情形是字段有多个值的排序， 需要记住这些值并没有固有的顺序；一个多值的字段仅仅是多个值的包装，这时应该选择哪个进行排序呢？

对于数字或日期，你可以将多值字段减为单值，这可以通过使用 `min` 、 `max` 、 `avg` 或是 `sum` *排序模式* 。 例如你可以按照每个 `date` 字段中的最早日期进行排序，通过以下方法：

```json
"sort": {
    "dates": {
        "order": "asc",
        "mode":  "min"
    }
}
```



### 字符串排序与多字段

被解析的字符串字段也是多值字段， 但是很少会按照你想要的方式进行排序。如果你想分析一个字符串，如 `fine old art` ， 这包含 3 项。我们很可能想要按第一项的字母排序，然后按第二项的字母排序，诸如此类，但是 Elasticsearch 在排序过程中没有这样的信息。

你可以使用 `min` 和 `max` 排序模式（默认是 `min` ），但是这会导致排序以 `art` 或是 `old` ，任何一个都不是所希望的。

为了以字符串字段进行排序，这个字段应仅包含一项： 整个 `not_analyzed` 字符串。 但是我们仍需要 `analyzed` 字段，这样才能以全文进行查询

==一个简单的方法是用两种方式对同一个字符串进行索引，这将在文档中包括两个字段： `analyzed` 用于搜索， `not_analyzed` 用于排序==

但是保存相同的字符串两次在 `_source` 字段是浪费空间的。 我们真正想要做的是传递一个*单字段* 但是却用两种方式索引它。所有的 **_core_field 类型 (strings, numbers, Booleans, dates)** 接收一个 `fields` 参数

该参数允许你转化一个简单的映射如：

```json
"tweet": {
    "type":     "string",
    "analyzer": "english"
}
```

为一个多字段映射如：

```json
"tweet": {  //①
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": {  //②
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}
```

![image-20230301113543559](E:/Development/Typora/images/image-20230301113543559.png)

现在，至少只要我们重新索引了我们的数据，使用 `tweet` 字段用于搜索，`tweet.raw` 字段用于排序：

```json
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    },
    "sort": "tweet.raw"
}
```

以全文 `analyzed` 字段排序会消耗大量的内存。获取更多信息请看 [聚合与分析](https://www.elastic.co/guide/cn/elasticsearch/guide/current/aggregations-and-analysis.html) 。



## 索引管理——待看



