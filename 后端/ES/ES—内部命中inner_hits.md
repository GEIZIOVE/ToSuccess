# ES内部命中—inner_hits

## 灵魂三问

### 问题1，什么是inner_hits？

`inner_hits`是ElasticSearch进行nested，has_parent，has_child搜索时的一个选项，用来标记命中文档位置的。以官方文档中的例子为例。

```yaml
PUT blog
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested"
      }
    }
  }
}


PUT blog/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "number": 1
    },
    {
      "author": "nik9000",
      "number": 2
    }
  ]
}
```

索引”blog“有一个类型为”nested“的字段”comments“，我们写入了一个文档，其中包含了2个”comments“。下面我们对文档进行搜索，我们先不添加”`inner_hits`“看一下结果是怎么样：

```yaml
POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {
          "comments.number": "2"
        }
      }
    }
  }
}

.....

    "hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "title" : "Test title",
          "comments" : [
            {
              "author" : "kimchy",
              "number" : 1
            },
            {
              "author" : "nik9000",
              "number" : 2
            }
          ]
        }
      }
    ]
```

可以看到，这个结果和正常搜索没有太多区别。

现在我们在搜索时加入”`inner_hits`“看一下效果，为了简单起见，”`inner_hits`“不使用任何选项。



```yaml
POST test/_search
{

  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {
          "comments.number": 2
          }
      },
      "inner_hits": {
      }
    }
  }
}
```

我们可以看到结果比刚才多了一个”`inner_hits`“块：

```yaml
        "inner_hits" : {
          "comments" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 1.0,
              "hits" : [
                {
                  "_index" : "test",
                  "_type" : "_doc",
                  "_id" : "1",
                  "_nested" : {
                    "field" : "comments",
                    "offset" : 1
                  },
                  "_score" : 1.0,
                  "_source" : {
                    "author" : "nik9000",
                    "number" : 2
                  }
                }
              ]
            }
          }
        }
```

可以看到，”inner_hits“中包含了关于此处文档的一些匹配信息，其中比较重要的有两个：

”nested“，告诉我们此次命中是文档中的那个nested字段，我们的例子里只有一个”nested“类型的字段，而实际上一个索引中，默认最多可以有50个”nested“类型的字段，这个值由索引的配置项”**index.mapping.nested_fields.limit**“控制。

> ”`_source`“，告诉我们当前命中的是哪个文档。我们的例子里，查询的条件是”number=2“，因此在 ”`_source`“部分返回了对应的那个文档。“inner_hits”有一个选项“`_source`”，默认值是true，如果将其置为false可以在返回结果中不显示”`_source`“的内容。

### 问题2，为什么要使用inner_hits？

在ElasticSearch中，nested对象是以独立的隐藏文档的方式进行存储的，以上面的例子为例，id为1的文档，有2个comments类型的nested对象，最终存储在ES中的其实是三个文档。而relation类型的对象，父子文档的结构可以完全不同，却是存储在同一个索引中。在进行nested search或者has_child，has_parent search的时候我们可能需要知道，我们的搜索到底是匹配了哪些更细粒度的文档。因此需要inner_hits。

![image-20230227104137934](E:/Development/Typora/images/image-20230227104137934.png)

内部命中还支持以下每个文档功能：

- [Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-highlighting.html)
- [Explain](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-explain.html)
- [Source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-source-filtering.html)
- [Script fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-script-fields.html)
- [Doc value fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-docvalue-fields.html)
- [Include versions](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-version.html)



### 问题3，怎样使用inner_hits

最简单的用法，如上文例子中使用的一样。直接加入一个空的“inner_hits”块即可。除此之外还有一些更细粒度的控制选项：

可查看下面：

更多可参考官方文档的内容

[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-inner-hits.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.elastic.co%2Fguide%2Fen%2Felasticsearch%2Freference%2F7.2%2Fsearch-request-inner-hits.html)







## 详解

### 基本使用

```json
"<query>" : {
  "inner_hits" : {
    <inner_hits_options>
  }
}
 
```



如果在支持它的查询上定义了 `inner_hits` ，那么每个搜索命中都将包含一个具有以下结构的 `inner_hits` json 对象：

```json
"hits": [
  {
    "_index": ...,
    "_type": ...,
    "_id": ...,
    "inner_hits": {
      "<inner_hits_name>": {
        "hits": {
          "total": ...,
          "hits": [
            {
              "_type": ...,
              "_id": ...,
               ...
            },
            ...
          ]
        }
      }
    },
    ...
  },
  ...
 
```



### Nested inner hits 嵌套内部命中

嵌套的 `inner_hits` 可用于将嵌套的内部对象包含为搜索命中的内部命中。

**再次引用官文代码**（上面也有）

```js
POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.number" : 2}
      },
      "inner_hits": {} //嵌套查询中的内部命中定义。无需定义其他选项。
    }
  }
}
```



可以从上述搜索请求生成的响应片段示例：

```js
{
  ...,
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": ...,
        "inner_hits": {
          "comments": {  //在搜索请求的内部命中定义中使用的名称。可以通过 name 选项使用自定义键。
            "hits": {
              "total" : {
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 1.0,
              "hits": [
                {
                  "_index": "test",
                  "_type": "_doc",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1
                  },
                  "_score": 1.0,
                  "_source": {
                    "author": "nik9000",
                    "number": 2
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
```

- `_nested` 元数据在上面的示例中至关重要，因为它定义了此内部命中来自哪个内部嵌套对象。 `field` 定义嵌套命中来自的对象数组字段， `offset` 定义相对于它在 `_source` 中的位置。由于排序和评分，命中对象在 `inner_hits` 中的实际位置通常不同于定义嵌套内部对象的位置。
- 默认情况下， `inner_hits` 中的命中对象也会返回 `_source` ，但这可以更改。通过 `_source` 过滤功能可以返回或禁用源的一部分。如果存储字段是在嵌套级别上定义的，则这些字段也可以通过 `fields` 功能返回。
- 一个重要的默认值是 `inner_hits` 内的命中返回的 `_source` 是相对于 `_nested` 元数据的。因此，在上面的示例中，每个嵌套命中仅返回评论部分，而不是包含评论的顶级文档的整个来源。



### 嵌套内部命中和 `_source`

嵌套文档没有 `_source` 字SSSS段，因为整个文档源与根文档一起存储在其 `_source` 字段下。为了仅包括嵌套文档的源，解析根文档的源并且仅将嵌套文档的相关位作为源包含在内部命中中。

为每个匹配的嵌套文档执行此操作会影响执行整个搜索请求所需的时间，尤其是当 `size` 和内部匹配项的 `size` 设置为高于默认值时。为了避免嵌套内部命中的相对昂贵的源提取，可以禁用包括源并仅依赖文档值字段。像这样：

```js
POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.text" : "words"}
      },
      "inner_hits": {
        "_source" : false,
        "docvalue_fields" : [
          "comments.text.keyword"
        ]
      }
    }
  }
}
```

### 嵌套对象字段和内部命中的层次结构。

如果一个映射有多个层级的层次嵌套对象字段，每个层级都可以通过点符号路径访问。例如，如果有一个包含 `votes` 嵌套字段的 `comments` 嵌套字段，并且投票应该直接与根命中一起返回，则可以定义以下路径：

```js
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested",
        "properties": {
          "votes": {
            "type": "nested"
          }
        }
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text",
      "votes": []
    },
    {
      "author": "nik9000",
      "text": "words words words",
      "votes": [
        {"value": 1 , "voter": "kimchy"},
        {"value": -1, "voter": "other"}
      ]
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments.votes",
        "query": {
          "match": {
            "comments.votes.voter": "kimchy"
          }
        },
        "inner_hits" : {}
    }
  }
}
```

这看起来像：

```js
{
  ...,
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.6931472,
        "_source": ...,
        "inner_hits": {
          "comments.votes": { 
            "hits": {
              "total" : {
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 0.6931472,
              "hits": [
                {
                  "_index": "test",
                  "_type": "_doc",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1,
                    "_nested": {
                      "field": "votes",
                      "offset": 0
                    }
                  },
                  "_score": 0.6931472,
                  "_source": {
                    "value": 1,
                    "voter": "kimchy"
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
```

这种间接引用仅支持嵌套的内部命中。





## InnerHitBuilder

### 字段

`InnerHitBuilder` 类是 Elasticsearch Java API 中的一个类，用于构建查询内部嵌套查询结果的特定字段。

以下是 `InnerHitBuilder` 类中常用字段的含义：

- `name`：内部嵌套查询的名称，用于在响应结果中标识该查询的结果。
- `size`：内部嵌套查询返回的文档数量，默认为 3，表示返回所有文档。
- `from`：内部嵌套查询结果的起始位置，默认为 0。
- `explain`：是否解释内部嵌套查询结果的评分计算方式，默认为 false。
- `trackScores`：是否跟踪内部嵌套查询结果的评分，默认为 false。
- `version`：是否在内部嵌套查询结果中包含文档版本信息，默认为 false。
- `sort`：内部嵌套查询结果的排序方式。
- `fetchSource`：是否在内部嵌套查询结果中包含文档的源数据，默认为 false。

这些字段可以通过 `InnerHitBuilder` 类的相应方法进行设置，例如：

```java
InnerHitBuilder innerHitBuilder = new InnerHitBuilder();
innerHitBuilder.setName("my_inner_hit");
innerHitBuilder.setSize(10);
innerHitBuilder.setFrom(5);
innerHitBuilder.setExplain(true);
innerHitBuilder.setTrackScores(true);
innerHitBuilder.setVersion(true);
innerHitBuilder.setSort(new FieldSortBuilder("my_field").order(SortOrder.DESC));
innerHitBuilder.setFetchSource(true);
```

以上代码将创建一个名为 `my_inner_hit` 的内部嵌套查询，返回从第 5 条文档开始的 10 条文档，同时包含文档的评分计算方式、评分、版本信息、以及在源数据中包含的字段，并按照 `my_field` 字段进行降序排序。





### 方法

当然，常用方法包括对上面常用字段的`set`和`get`，比如

`setName(String name)`：

设置内部嵌套查询的名称，用于在响应结果中标识该查询的结果。

```java
InnerHitBuilder innerHitBuilder = new InnerHitBuilder();
innerHitBuilder.setName("my_inner_hit");
```

- `setHighlightBuilder(HighlightBuilder highlightBuilder)`：设置内部嵌套查询结果的高亮显示方式。
- `setQuery(QueryBuilder queryBuilder)`：设置内部嵌套查询的查询条件。
- `setCollapseBuilder(CollapseBuilder collapseBuilder)`：设置内部嵌套查询结果的折叠方式。
- `setStoredField(String field)`：设置内部嵌套查询结果中要返回的存储字段。

