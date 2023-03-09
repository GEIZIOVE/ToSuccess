# ES-数据建模

## 1.关联关系处理

### 1.1应用层联接

我们通过在我们的应用程序中实现联接可以（部分）模拟关系数据库。 例如，比方说我们正在对用户和他们的博客文章进行索引。在关系世界中，我们会这样来操作：

```json
PUT /my_index/user/1    ①
{
  "name":     "John Smith",
  "email":    "john@smith.com",
  "dob":      "1970/10/24"
}
PUT /my_index/blogpost/2 ①
{
  "title":    "Relationships",
  "body":     "It's complicated...",
  "user":     1  ②
}
```

> 每个文档的 index, type, 和 id 一起构造成主键。
>
>
> ②blogpost 通过用户的 id 链接到用户。index 和 type 并不需要因为在我们的应用程序中已经硬编码。



通过用户的 ID `1` 可以很容易的找到博客帖子。

```json
GET /my_index/blogpost/_search
{
  "query": {
    "filtered": {
      "filter": {
        "term": { "user": 1 }
      }
    }
  }
}
```

为了找到用户叫做 John 的博客帖子，我们需要运行两次查询： 

- 第一次会查找所有叫做 John 的用户从而获取他们的 ID 集合
- 第二次会将这些 ID 集合放到类似于前面一个例子的查询：

```json
GET /my_index/user/_search
{
  "query": {
    "match": {
      "name": "John"
    }
  }
}
GET /my_index/blogpost/_search
{
  "query": {
    "filtered": {
      "filter": {
        "terms": { "user": [1] }   ①
      }
    }
  }
}
```

> ① 执行第一个查询得到的结果将填充到 `terms` 过滤器中。 

- 应用层联接的主要优点是**可以对数据进行标准化处理**。只能在 `user` 文档中修改用户的名称。缺点是，为了在搜索时联接文档，必须运行额外的查询。
- 在这个例子中，只有一个用户匹配我们的第一个查询，但在现实世界中，我们可以很轻易的遇到数以百万计的叫做 John 的用户。 包含所有这些用户的 IDs 会产生一个非常大的查询，这是一个数百万词项的查找。
- 这种方法适用于第一个实体（例如，在这个例子中 `user` ）只有少量的文档记录的情况，并且最好它们很少改变。这将允许应用程序对结果进行缓存，并避免经常运行第一次查询。



### 1.2非规范化数据

使用 Elasticsearch 得到最好的搜索性能的方法是有目的的通过在索引时进行非规范化 [denormalizing](http://en.wikipedia.org/wiki/Denormalization)。对每个文档保持一定数量的冗余副本可以在需要访问时避免进行关联。

如果我们希望能够通过某个用户姓名找到他写的博客文章，可以在博客文档中包含这个用户的姓名：

```json
PUT /my_index/user/1
{
  "name":     "John Smith",
  "email":    "john@smith.com",
  "dob":      "1970/10/24"
}

PUT /my_index/blogpost/2
{
  "title":    "Relationships",
  "body":     "It's complicated...",
  "user":     {
    "id":       1,
    "name":     "John Smith"  ①
  }
}
```

> ①这部分用户的字段数据已被冗余到 `blogpost` 文档中。

现在，我们通过单次查询就能够通过 `relationships` 找到用户 `John` 的博客文章。

```json
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title":     "relationships" }},
        { "match": { "user.name": "John"          }}
      ]
    }
  }
}
```



数据非规范化的优点是速度快。因为每个文档都包含了所需的所有信息，当这些信息需要在查询进行匹配时，并不需要进行昂贵的联接操作。



### 1.3字段折叠

一个普遍的需求是需要通过特定字段进行分组。例如我们需要按照用户名称 *分组* 返回最相关的博客文章。 按照用户名分组意味着进行 `terms` 聚合。 为能够按照用户 *整体* 名称进行分组，名称字段应保持 `not_analyzed` 的形式， 具体说明参考 [聚合与分析](https://www.elastic.co/guide/cn/elasticsearch/guide/current/aggregations-and-analysis.html)：

```json
PUT /my_index/_mapping/blogpost
{
  "properties": {
    "user": {
      "properties": {
        "name": {  ①
          "type": "string",
          "fields": {
            "raw": {  ②
              "type":  "string",
              "index": "not_analyzed"
            }
          }
        }
      }
    }
  }
}
```

> ①`user.name` 字段将用来进行全文检索。
>
> ②`user.name.raw` 字段将用来通过 `terms` 聚合进行分组。

然后添加一些数据:

```json
PUT /my_index/user/1
{
  "name": "John Smith",
  "email": "john@smith.com",
  "dob": "1970/10/24"
}

PUT /my_index/blogpost/2
{
  "title": "Relationships",
  "body": "It's complicated...",
  "user": {
    "id": 1,
    "name": "John Smith"
  }
}

PUT /my_index/user/3
{
  "name": "Alice John",
  "email": "alice@john.com",
  "dob": "1979/01/04"
}

PUT /my_index/blogpost/4
{
  "title": "Relationships are cool",
  "body": "It's not complicated at all...",
  "user": {
    "id": 3,
    "name": "Alice John"
  }
}
```

现在我们来查询标题包含 `relationships` 并且作者名包含 `John` 的博客，查询结果再按作者名分组，感谢 [`top_hits` aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/search-aggregations-metrics-top-hits-aggregation.html) 提供了按照用户进行分组的功能：

```json
GET /my_index/blogpost/_search
{
  "size" : 0,  ①
  "query": {  ②
    "bool": {
      "must": [
        { "match": { "title":     "relationships" }},
        { "match": { "user.name": "John"          }}
      ]
    }
  },
  "aggs": {
    "users": {
      "terms": {
        "field":   "user.name.raw",     ③
        "order": { "top_score": "desc" }  ④
      },
      "aggs": {
        "top_score": { "max":      { "script":  "_score"           }}, 
        "blogposts": { "top_hits": { "_source": "title", "size": 5 }}  
      }
    }
  }
}
```

![image-20230228231444049](E:/Development/Typora/images/image-20230228231444049.png)



这里显示简短响应结果：

```json
...
"hits": {
  "total":     2,
  "max_score": 0,
  "hits":      []  ①
},
"aggregations": {
  "users": {
     "buckets": [
        {
           "key":       "John Smith",  ②
           "doc_count": 1,
           "blogposts": {
              "hits": {  ③
                 "total":     1,
                 "max_score": 0.35258877,
                 "hits": [
                    {
                       "_index": "my_index",
                       "_type":  "blogpost",
                       "_id":    "2",
                       "_score": 0.35258877,
                       "_source": {
                          "title": "Relationships"
                       }
                    }
                 ]
              }
           },
           "top_score": { ④
              "value": 0.3525887727737427
           }
        },
...
```



![image-20230228231551881](E:/Development/Typora/images/image-20230228231551881.png)使用 

`top_hits` 聚合等效执行一个查询返回这些用户的名字和他们最相关的博客文章，然后为每一个用户执行相同的查询，以获得最好的博客。但前者的效率要好很多。

每一个桶返回的顶层查询命中结果是基于最初主查询进行的一个轻量 ***迷你查询*** 结果集。这个迷你查询提供了一些你期望的常用特性，例如高亮显示以及分页功能。



## 2.嵌套对象

### 2.1简介

由于在 Elasticsearch 中单个文档的增删改都是原子性操作,那么将相关实体数据都存储在同一文档中也就理所当然。 比如说,我们可以将订单及其明细数据存储在一个文档中。又比如,我们可以将一篇博客文章的评论以一个 `comments` 数组的形式和博客文章放在一起：

```json
PUT /my_index/blogpost/1
{
  "title": "Nest eggs",
  "body":  "Making your money work...",
  "tags":  [ "cash", "shares" ],
  "comments": [ //①
    {
      "name":    "John Smith",
      "comment": "Great article",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "More like this please",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}
```

> ①如果我们依赖[字段自动映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html),那么 `comments` 字段会自动映射为 `object` 类型

由于所有的信息都在一个文档中,当我们查询时就没有必要去联合文章和评论文档,查询效率就很高。

但是当我们使用如下查询时,上面的文档也会被当做是符合条件的结果：

```json
GET /_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "Alice" }},
        { "match": { "age":  28      }}  //①
      ]
    }
  }
}
```

> ①Alice实际是31岁,不是28!

正如我们在 [对象数组](https://www.elastic.co/guide/cn/elasticsearch/guide/current/complex-core-fields.html#object-arrays) 中讨论的一样,出现上面这种问题的原因是 JSON 格式的文档被处理成如下的扁平式键值对的结构。

```json
{
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ],
  "comments.name":    [ alice, john, smith, white ],
  "comments.comment": [ article, great, like, more, please, this ],
  "comments.age":     [ 28, 31 ],
  "comments.stars":   [ 4, 5 ],
  "comments.date":    [ 2014-09-01, 2014-10-22 ]
}
```



`Alice` 和 31 、 `John` 和 `2014-09-01` 之间的相关性信息不再存在。虽然 `object` 类型 (参见 [内部对象](https://www.elastic.co/guide/cn/elasticsearch/guide/current/complex-core-fields.html#inner-objects)) 在存储 *单一对象* 时非常有用,==但对于对象数组的搜索而言,毫无用处。==

*嵌套对象* 就是来解决这个问题的。将 `comments` 字段类型设置为 `nested` 而不是 `object` 后,每一个嵌套对象都会被索引为一个 *隐藏的独立文档* ,举例如下:

```json
{ ①
  "comments.name":    [ john, smith ],
  "comments.comment": [ article, great ],
  "comments.age":     [ 28 ],
  "comments.stars":   [ 4 ],
  "comments.date":    [ 2014-09-01 ]
}
{ ②
  "comments.name":    [ alice, white ],
  "comments.comment": [ like, more, please, this ],
  "comments.age":     [ 31 ],
  "comments.stars":   [ 5 ],
  "comments.date":    [ 2014-10-22 ]
}
{ ③
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ]
}
```

![image-20230228232043078](E:/Development/Typora/images/image-20230228232043078.png)



在独立索引每一个嵌套对象后,对象中每个字段的相关性得以保留。我们查询时,也仅仅返回那些真正符合条件的文档。

不仅如此,由于嵌套文档直接存储在文档内部,查询时嵌套文档和根文档联合成本很低,速度和单独存储几乎一样。

嵌套文档是隐藏存储的,我们不能直接获取。如果要增删改一个嵌套对象,我们必须把整个文档重新索引才可以。值得注意的是,查询的时候返回的是整个文档,而不是嵌套文档本身。

### 2.2 嵌套对象的映射

设置一个字段为 `nested` 很简单 —  你只需要将字段类型 `object` 替换为 `nested` 即可：

```json
PUT /my_index
{
  "mappings": {
    "blogpost": {
      "properties": {
        "comments": {
          "type": "nested",  ①
          "properties": {
            "name":    { "type": "string"  },
            "comment": { "type": "string"  },
            "age":     { "type": "short"   },
            "stars":   { "type": "short"   },
            "date":    { "type": "date"    }
          }
        }
      }
    }
  }
}
```

> ①`nested` 字段类型的设置参数与 `object` 相同。

这就是需要设置的一切。至此，所有 `comments` 对象会被索引在独立的嵌套文档中。可以查看 [`nested` 类型参考文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/nested.html) 获取更多详细信息。

### 2.3嵌套对象查询

由于嵌套对象 被索引在独立隐藏的文档中，我们无法直接查询它们。 相应地，我们必须使用 [`nested` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-nested-query.html) 去获取它们：

```json
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "eggs" // ①
          }
        },
        {
          "nested": {
            "path": "comments",  //②
            "query": {
              "bool": {
                "must": [  //③
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

![image-20230228233245650](E:/Development/Typora/images/image-20230228233245650.png)

> `nested` 字段可以包含其他的 `nested` 字段。同样地，`nested` 查询也可以包含其他的 `nested` 查询。而嵌套的层次会按照你所期待的被应用。

`nested` 查询肯定可以匹配到多个嵌套的文档。每一个匹配的嵌套文档都有自己的相关度得分，但是这众多的分数最终需要汇聚为可供根文档使用的一个分数。

==默认情况下，根文档的分数是这些嵌套文档分数的平均值==。可以通过设置 `score_mode` 参数来控制这个得分策略，相关策略有 `avg` (平均值), `max` (最大值), `sum` (加和) 和 `none` (直接返回 **1.0** 常数值分数)。

```json
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
            "score_mode": "max", //①
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
    }
  }
}
```



>  ①返回最优匹配嵌套文档的 `_score` 给根文档使用。



> 如果 `nested` 查询放在一个布尔查询的 `filter` 子句中，其表现就像一个 `nested` 查询，只是 `score_mode` 参数不再生效。因为它被用于不打分的查询中 — 只是符合或不符合条件，不必打分 — 那么 `score_mode` 就没有任何意义，因为根本就没有要打分的地方。
>
> 



### 2.4 使用嵌套字段排序

尽管嵌套字段的值存储于独立的嵌套文档中，但依然有方法按照嵌套字段的值排序。 让我们添加另一个记录，以使得结果更有意思：

```json
PUT /my_index/blogpost/2
{
  "title": "Investment secrets",
  "body":  "What they don't tell you ...",
  "tags":  [ "shares", "equities" ],
  "comments": [
    {
      "name":    "Mary Brown",
      "comment": "Lies, lies, lies",
      "age":     42,
      "stars":   1,
      "date":    "2014-10-18"
    },
    {
      "name":    "John Smith",
      "comment": "You're making it up!",
      "age":     28,
      "stars":   2,
      "date":    "2014-10-16"
    }
  ]
}
```



假如我们想要查询在10月份收到评论的博客文章，并且按照 `stars` 数的最小值来由小到大排序，那么查询语句如下：

```json
GET /_search
{
  "query": {
    "nested": {  //① 
      "path": "comments",
      "filter": {
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  },
  "sort": {
    "comments.stars": { //②
      "order": "asc",   //②
      "mode":  "min",   //②
      "nested_path": "comments", //③
      "nested_filter": {
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  }
}
```



![image-20230301000027768](E:/Development/Typora/images/image-20230301000027768.png)

> 我们为什么要用 nested_path 和 nested_filter 重复查询条件呢？
>
> 原因在于，排序发生在查询执行之后。 查询条件限定了在10月份收到评论的博客文档，但返回的是博客文档。如果我们不在排序子句中加入 `nested_filter` ， 那么我们对博客文档的排序将基于博客文档的所有评论，而不是仅仅在10月份接收到的评论。

### 2.5嵌套聚合

在查询的时候，我们使用 `nested` 查询 就可以获取嵌套对象的信息。同理， `nested` 聚合允许我们对嵌套对象里的字段进行聚合操作。

```json
GET /my_index/blogpost/_search
{
  "size" : 0,
  "aggs": {
    "comments": { //①
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "by_month": {
          "date_histogram": {   //②
            "field":    "comments.date",
            "interval": "month",
            "format":   "yyyy-MM"
          },
          "aggs": {
            "avg_stars": {
              "avg": {  //③
                "field": "comments.stars"
              }
            }
          }
        }
      }
    }
  }
}
```

![image-20230301000710887](E:/Development/Typora/images/image-20230301000710887.png)

从下面的结果可以看出聚合是在嵌套文档层面进行的：

```json
...
"aggregations": {
  "comments": {
     "doc_count": 4,  //①
     "by_month": {
        "buckets": [
           {
              "key_as_string": "2014-09",
              "key": 1409529600000,
              "doc_count": 1,  //②
              "avg_stars": {
                 "value": 4
              }
           },
           {
              "key_as_string": "2014-10",
              "key": 1412121600000,
              "doc_count": 3,  //①
              "avg_stars": {
                 "value": 2.6666666666666665
              }
           }
        ]
     }
  }
}
...
```

![image-20230301000858338](E:/Development/Typora/images/image-20230301000858338.png)

### 2.6逆向嵌套聚合

`nested` 聚合 只能对嵌套文档的字段进行操作。 根文档或者其他嵌套文档的字段对它是不可见的。 然而，通过 `reverse_nested` 聚合，我们可以 *走出* 嵌套层级，回到父级文档进行操作。

例如，我们要基于评论者的年龄找出评论者感兴趣 `tags` 的分布。 `comment.age` 是一个嵌套字段，但 `tags` 在根文档中：

```json
GET /my_index/blogpost/_search
{
  "size" : 0,
  "aggs": {
    "comments": {
      "nested": {  //①
        "path": "comments"
      },
      "aggs": {
        "age_group": {
          "histogram": {  //②
            "field":    "comments.age",
            "interval": 10
          },
          "aggs": {
            "blogposts": {
              "reverse_nested": {},  //③
              "aggs": {
                "tags": {
                  "terms": {  //④
                    "field": "tags"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

![image-20230301001025614](E:/Development/Typora/images/image-20230301001025614.png)

简略结果如下所示：

```json
..
"aggregations": {
  "comments": {
     "doc_count": 4,  //①
     "age_group": {
        "buckets": [
           {
              "key": 20,  //②
              "doc_count": 2,  //②
              "blogposts": {
                 "doc_count": 2,  //③
                 "tags": {
                    "doc_count_error_upper_bound": 0,
                    "buckets": [ //④
                       { "key": "shares",   "doc_count": 2 },
                       { "key": "cash",     "doc_count": 1 },
                       { "key": "equities", "doc_count": 1 }
                    ]
                 }
              }
           },
...
```

![image-20230301001107717](E:/Development/Typora/images/image-20230301001107717.png)



嵌套模型的缺点如下：

- 当对嵌套文档做增加、修改或者删除时，整个文档都要重新被索引。嵌套文档越多，这带来的成本就越大。

- 查询结果返回的是整个文档，而不仅仅是匹配的嵌套文档。尽管目前有计划支持只返回根文档中最佳匹配的嵌套文档。（现在我们可以在父级元素中排出嵌套对象，然后使用inner_hint只返回评分最高的那个元素来实现）

  - 下面是一个例子，假设您有一个名为“my_index”的索引，其中包含名为“my_type”的类型，其中每个文档都有一个名为“nested_doc”的嵌套文档：

  - ```json
    PUT my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "nested_doc": {
              "type": "nested",
              "properties": {
                "name": {"type": "text"},
                "age": {"type": "integer"}
              }
            }
          }
        }
      }
    }
    
    PUT my_index/my_type/1
    {
      "nested_doc": [
        {"name": "John", "age": 25},
        {"name": "Mary", "age": 30}
      ]
    }
    
    PUT my_index/my_type/2
    {
      "nested_doc": [
        {"name": "Peter", "age": 35},
        {"name": "Mary", "age": 30}
      ]
    }
    
    ```

  - 现在，假设您想要查询“my_index”索引中“my_type”类型中的所有文档，并且只返回每个文档中与特定名称匹配的最佳匹配的嵌套文档。您可以使用如下查询来实现：

  - ```json
    GET my_index/my_type/_search
    {
      "query": {
        "nested": {
          "path": "nested_doc",
          "query": {
            "bool": {
              "must": [
                {"match": {"nested_doc.name": "Mary"}}
              ],
              "should": [
                {"term": {"nested_doc.name": "Mary"}},
                {"range": {"nested_doc.age": {"gte": 30}}}
              ],
              "minimum_should_match": 1,
              "boost": 1.0
            }
          },
          "inner_hits": {
            "size": 1,
            "sort": [{"nested_doc.age": {"order": "desc"}}]
          }
        }
      }
    }
    
    ```

在这个查询中，我们使用了一个嵌套查询来查询名为“Mary”的嵌套文档，并使用了一些bool查询来确保我们只返回与特定名称匹配的最佳匹配的文档。我们还使用了内部命中（inner_hits）来确保我们只返回每个文档中与特定名称匹配的最佳匹配的嵌套文档。

请注意，这个查询中的“minimum_should_match”参数被设置为1，这意味着我们只需要满足bool查询中的一个条件即可。在这个例子中，我们使用了两个should查询，其中一个是匹配特定名称，另一个是匹配年龄大于等于30的文档。这样，如果没有与特定名称匹配的文档，我们仍然可以返回匹配年龄的文档。

另外，我们使用了“sort”参数来按照年龄





### 2.7 查询Elasticsearch嵌套类型数据，且只返回嵌套数据中命中的元素

#### 新建一个index作为测试

以下是一个存储博客文章及其评论的数据结构，评论(comments)是nested类型：

```css
PUT /es_blog
{
    "mappings": {
        "blogpost": {
            "properties": {
                "title": {
                    "type": "text"
                },
                "summary": {
                    "type": "text"
                },
                "content": {
                    "type": "text"
                },
                "comments": {
                    "type": "nested",
                    "properties": {
                        "name": {
                            "type": "text"
                        },
                        "comment": {
                            "type": "text"
                        },
                        "age": {
                            "type": "short"
                        },
                        "stars": {
                            "type": "short"
                        },
                        "date": {
                            "type": "date"
                        }
                    }
                }
            }
        }
    }
}
```

#### 写入一些测试数据

```bash
PUT /es_blog/blogpost/1
{
  "title": "无标题",
  "summary": "全栈工程师、JAVA、HTML5",
  "content": "全栈工程师需要掌握：JAVA、HTML5、JavaScript、常用缓存、大数据等等",
  "comments": [
    {
      "name":    "John Smith",
      "comment": "Great article",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "More like this please",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}

PUT /es_blog/blogpost/2
{
  "title": "Java后端开发工程师",
  "summary": "JAVA、Oracle、Hibernate、Spring",
  "content": "Java后端开发工程师需要掌握：JAVA、Oracle、Hibernate、Spring、常用缓存等等",
  "comments": [
    {
      "name":    "John Smith",
      "comment": "工程师真牛",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "Java工程师真牛",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}

PUT /es_blog/blogpost/3
{
  "title": "大数据工程师",
  "summary": "Hadoop、Hive、Hdfs、JAVA",
  "content": "大数据工程师需要掌握：Hadoop、Hive、Hdfs、JAVA、Spark、流式计算等等",
  "comments": [
    {
      "name":    "John Smith",
      "comment": "大数据工程师真牛",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "我不会啊",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}

PUT /es_blog/blogpost/4
{
  "title": "机器学习工程师",
  "summary": "Python、回归算法、分类算法、神经网络、数据基础",
  "content": "机器学习工程师需要掌握：Python、回归算法、分类算法、神经网络、有扎实的数据基础等等",
  "comments": [
    {
      "name":    "John Smith",
      "comment": "机器学习NX",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "Python好学么？",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}
```



#### 查询文本字段中出现了[工程师]的数据

```css
GET es_blog/blogpost/_search
{
  "_source": {
    "includes": [
      "*"
    ],
    "excludes": [
      "comments"   //去掉返回结果中父级中的comments信息
    ]
  },
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": "工程师"
          }
        },
        {
          "match": {
            "summary": "工程师"
          }
        },
        {
          "match": {
            "content": "工程师"
          }
        },
        {
          "nested": {
            "path": "comments",
            "query": {
              "bool": {
                "should": [
                  {
                    "match": {
                      "comments.comment": "工程师"
                    }
                  }
                ]
              }
            },
            "inner_hits": {}
          }
        }
      ]
    }
  }
}
```

##### 返回结果：

```json
{
  "took": 9,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 3.5560012,
    "hits": [
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "3",
        "_score": 3.5560012,
        "_source": {
          "summary": "Hadoop、Hive、Hdfs、JAVA",
          "title": "大数据工程师",
          "content": "大数据工程师需要掌握：Hadoop、Hive、Hdfs、JAVA、Spark、流式计算等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 1,
              "max_score": 1.8299085,
              "hits": [
                {
                  "_index": "es_blog",
                  "_type": "blogpost",
                  "_id": "3",
                  "_nested": {
                    "field": "comments",
                    "offset": 0
                  },
                  "_score": 1.8299085,
                  "_source": {
                    "name": "John Smith",
                    "comment": "大数据工程师真牛",
                    "age": 28,
                    "stars": 4,
                    "date": "2014-09-01"
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "2",
        "_score": 3.1327708,
        "_source": {
          "summary": "JAVA、Oracle、Hibernate、Spring",
          "title": "Java后端开发工程师",
          "content": "Java后端开发工程师需要掌握：JAVA、Oracle、Hibernate、Spring、常用缓存等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 2,
              "max_score": 2.0794415,
              "hits": [
                {
                  "_index": "es_blog",
                  "_type": "blogpost",
                  "_id": "2",
                  "_nested": {
                    "field": "comments",
                    "offset": 0
                  },
                  "_score": 2.0794415,
                  "_source": {
                    "name": "John Smith",
                    "comment": "工程师真牛",
                    "age": 28,
                    "stars": 4,
                    "date": "2014-09-01"
                  }
                },
                {
                  "_index": "es_blog",
                  "_type": "blogpost",
                  "_id": "2",
                  "_nested": {
                    "field": "comments",
                    "offset": 1
                  },
                  "_score": 1.9221728,
                  "_source": {
                    "name": "Alice White",
                    "comment": "Java工程师真牛",
                    "age": 31,
                    "stars": 5,
                    "date": "2014-10-22"
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "1",
        "_score": 1.7260926,
        "_source": {
          "summary": "全栈工程师、JAVA、HTML5",
          "title": "无标题",
          "content": "全栈工程师需要掌握：JAVA、HTML5、JavaScript、常用缓存、大数据等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 0,
              "max_score": null,
              "hits": []
            }
          }
        }
      },
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "4",
        "_score": 1.0651813,
        "_source": {
          "summary": "Python、回归算法、分类算法、神经网络、数据基础",
          "title": "机器学习工程师",
          "content": "机器学习工程师需要掌握：Python、回归算法、分类算法、神经网络、有扎实的数据基础等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 0,
              "max_score": null,
              "hits": []
            }
          }
        }
      }
    ]
  }
}
```



#### 查询content中出现[java]或评论中出现[python]的数据

```json
GET es_blog/blogpost/_search
{
  "_source": {
    "includes": [
      "*"
    ],
    "excludes": [
      "comments"
    ]
  },
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "content": "java"
          }
        },
        {
          "nested": {
            "path": "comments",
            "query": {
              "bool": {
                "should": [
                  {
                    "match": {
                      "comments.comment": "python"
                    }
                  }
                ]
              }
            },
            "inner_hits": {}
          }
        }
      ]
    }
  }
}
```



##### 返回结果

```json
{
  "took": 9,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 1.3112576,
    "hits": [
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "4",
        "_score": 1.3112576,
        "_source": {
          "summary": "Python、回归算法、分类算法、神经网络、数据基础",
          "title": "机器学习工程师",
          "content": "机器学习工程师需要掌握：Python、回归算法、分类算法、神经网络、有扎实的数据基础等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 1,
              "max_score": 1.3112576,
              "hits": [
                {
                  "_index": "es_blog",
                  "_type": "blogpost",
                  "_id": "4",
                  "_nested": {
                    "field": "comments",
                    "offset": 1
                  },
                  "_score": 1.3112576,
                  "_source": {
                    "name": "Alice White",
                    "comment": "Python好学么？",
                    "age": 31,
                    "stars": 5,
                    "date": "2014-10-22"
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "2",
        "_score": 0.75974846,
        "_source": {
          "summary": "JAVA、Oracle、Hibernate、Spring",
          "title": "Java后端开发工程师",
          "content": "Java后端开发工程师需要掌握：JAVA、Oracle、Hibernate、Spring、常用缓存等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 0,
              "max_score": null,
              "hits": []
            }
          }
        }
      },
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "summary": "全栈工程师、JAVA、HTML5",
          "title": "无标题",
          "content": "全栈工程师需要掌握：JAVA、HTML5、JavaScript、常用缓存、大数据等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 0,
              "max_score": null,
              "hits": []
            }
          }
        }
      },
      {
        "_index": "es_blog",
        "_type": "blogpost",
        "_id": "3",
        "_score": 0.2876821,
        "_source": {
          "summary": "Hadoop、Hive、Hdfs、JAVA",
          "title": "大数据工程师",
          "content": "大数据工程师需要掌握：Hadoop、Hive、Hdfs、JAVA、Spark、流式计算等等"
        },
        "inner_hits": {
          "comments": {
            "hits": {
              "total": 0,
              "max_score": null,
              "hits": []
            }
          }
        }
      }
    ]
  }
}
```

















