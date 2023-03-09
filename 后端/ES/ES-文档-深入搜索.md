# ES-文档-深入搜索

## 结构化搜索*（Structured search）*

### 简介

*结构化搜索（Structured search）* 是指有关探询那些具有内在结构数据的过程。比如日期、时间和数字都是结构化的：它们有精确的格式，我们可以对这些格式进行逻辑操作。比较常见的操作包括比较数字或时间的范围，或判定两个值的大小。

文本也可以是结构化的。如彩色笔可以有离散的颜色集合： `红（red）` 、 `绿（green）` 、 `蓝（blue）` 。一个博客可能被标记了关键词 `分布式（distributed）` 和 `搜索（search）` 。电商网站上的商品都有 UPCs（通用产品码 Universal Product Codes）或其他的唯一标识，它们都需要遵从严格规定的、结构化的格式。

在结构化查询中，我们得到的结果 *总是* 非是即否，要么存于集合之中，要么存在集合之外。结构化查询不关心文件的相关度或评分；它简单的对文档包括或排除处理。

这在逻辑上是能说通的，因为一个数字不能比其他数字 *更* 适合存于某个相同范围。结果只能是：存于范围之中，抑或反之。==同样，对于结构化文本来说，一个值要么相等，要么不等。没有 *更似* 这种概念。==



### 精确值查找

当进行精确值查找时， 我们会使用过滤器（filters）。过滤器很重要，因为它们执行速度非常快，不会计算相关度（直接跳过了整个评分阶段）而且很容易被缓存。我们会在本章后面的 [过滤器缓存](https://www.elastic.co/guide/cn/elasticsearch/guide/current/filter-caching.html) 中讨论过滤器的性能优势，==不过现在只要记住：请尽可能多的使用过滤式查询。==

#### term 查询数字

我们首先来看最为常用的 `term` 查询， 可以用它处理数字（numbers）、布尔值（Booleans）、日期（dates）以及文本（text）。】

让我们以下面的例子开始介绍，创建并索引一些表示产品的文档，文档里有字段 `price` 和 `productID` （ `价格` 和 `产品ID` ）：

```json
POST /my_store/products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }
```

我们想要做的是查找具有某个价格的所有产品，有关系数据库背景的人肯定熟悉 SQL，如果我们将其用 SQL 形式表达，会是下面这样：

```sql
SELECT document
FROM   products
WHERE  price = 20
```

在 Elasticsearch 的查询表达式（query DSL）中，我们可以使用 `term` 查询达到相同的目的。 `term` 查询会查找我们指定的精确值。作为其本身， `term` 查询是简单的。它接受一个字段名以及我们希望查找的数值：

```js
{
    "term" : {
        "price" : 20
    }
}
```

通常当查找一个精确值的时候，我们不希望对查询进行评分计算。只希望对文档进行包括或排除的计算，所以我们会使用 `constant_score` 查询以非评分模式来执行 `term` 查询并以一作为统一评分。

最终组合的结果是一个 `constant_score` 查询，它包含一个 `term` 查询：

```json
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : { ①
            "filter" : {
                "term" : {  ②
                    "price" : 20
                }
            }
        }
    }
}
```

![image-20230301115014972](E:/Development/Typora/images/image-20230301115014972.png)

执行后，这个查询所搜索到的结果与我们期望的一致：只有文档 2 命中并作为结果返回（因为只有 `2` 的价格是 `20` ):

```json
"hits" : [
    {
        "_index" : "my_store",
        "_type" :  "products",
        "_id" :    "2",
        "_score" : 1.0, ①
        "_source" : {
          "price" :     20,
          "productID" : "KDKE-B-9947-#kL5"
        }
    }
]
```

![image-20230301115103482](E:/Development/Typora/images/image-20230301115103482.png)

#### term 查询文本

如本部分开始处提到过的一样 ，使用 `term` 查询匹配字符串和匹配数字一样容易。如果我们想要查询某个具体 UPC ID 的产品，使用 SQL 表达式会是如下这样：

```sql
SELECT product
FROM   products
WHERE  productID = "XHDK-A-1293-#fJ3"
```

转换成查询表达式（query DSL），同样使用 `term` 查询，形式如下：

```json
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "productID" : "XHDK-A-1293-#fJ3"
                }
            }
        }
    }
}
```

但这里有个小问题：我们无法获得期望的结果。为什么呢？问题不在 `term` 查询，而在于索引数据的方式。 如果我们使用 `analyze` API ([分析 API](https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html#analyze-api))，我们可以看到这里的 UPC 码被拆分成多个更小的 token ：

```json
GET /my_store/_analyze
{
  "field": "productID",
  "text": "XHDK-A-1293-#fJ3"
}
```

```json
{
  "tokens" : [ {
    "token" :        "xhdk",
    "start_offset" : 0,
    "end_offset" :   4,
    "type" :         "<ALPHANUM>",
    "position" :     1
  }, {
    "token" :        "a",
    "start_offset" : 5,
    "end_offset" :   6,
    "type" :         "<ALPHANUM>",
    "position" :     2
  }, {
    "token" :        "1293",
    "start_offset" : 7,
    "end_offset" :   11,
    "type" :         "<NUM>",
    "position" :     3
  }, {
    "token" :        "fj3",
    "start_offset" : 13,
    "end_offset" :   16,
    "type" :         "<ALPHANUM>",
    "position" :     4
  } ]
}
```

这里有几点需要注意：

- Elasticsearch 用 4 个不同的 token 而不是单个 token 来表示这个 UPC 。
- 所有字母都是小写的。
- 丢失了连字符和哈希符（ `#` ）。

所以当我们用 `term` 查询查找精确值 `XHDK-A-1293-#fJ3` 的时候，找不到任何文档，因为它并不在我们的倒排索引中，正如前面呈现出的分析结果，索引里有四个 token 。

显然这种对 ID 码或其他任何精确值的处理方式并不是我们想要的。

为了避免这种问题，我们需要告诉 Elasticsearch 该字段具有精确值，要将其设置成 `not_analyzed` 无需分析的。 我们可以在 [自定义字段映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-intro.html#custom-field-mappings) 中查看它的用法。==为了修正搜索结果，我们需要首先删除旧索引（因为它的映射不再正确）然后创建一个能正确映射的新索引：==

```json
DELETE /my_store  ①

PUT /my_store  ②
{
    "mappings" : {
        "products" : {
            "properties" : {
                "productID" : {
                    "type" : "string",
                    "index" : "not_analyzed"  ③
                }
            }
        }
    }

}
```

![image-20230301115349964](E:/Development/Typora/images/image-20230301115349964.png)

现在我们可以为文档重建索引：

```json
POST /my_store/products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }
```

此时， `term` 查询就能搜索到我们想要的结果，让我们再次搜索新索引过的数据（注意，查询和过滤并没有发生任何改变，改变的是数据映射的方式）：

```json
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "productID" : "XHDK-A-1293-#fJ3"
                }
            }
        }
    }
}
```

因为 `productID` 字段是未分析过的， `term` 查询不会对其做任何分析，查询会进行精确查找并返回文档 1 。成功！

#### 内部过滤器的操作

在内部，Elasticsearch 会在运行非评分查询的时执行多个操作：

1. *查找匹配文档*.

   `term` 查询在倒排索引中查找 `XHDK-A-1293-#fJ3` 然后获取包含该 term 的所有文档。本例中，只有文档 1 满足我们要求。

2. *创建 bitset*.

   过滤器会创建一个 *bitset* （一个包含 0 和 1 的数组），它描述了哪个文档会包含该 term 。匹配文档的标志位是 1 。本例中，bitset 的值为 `[1,0,0,0]` 。在内部，它表示成一个 ["roaring bitmap"](https://www.elastic.co/blog/frame-of-reference-and-roaring-bitmaps)，可以同时对稀疏或密集的集合进行高效编码。

3. *迭代 bitset(s)*

   一旦为每个查询生成了 bitsets ，Elasticsearch 就会循环迭代 bitsets 从而找到满足所有过滤条件的匹配文档的集合。执行顺序是启发式的，但一般来说先迭代稀疏的 bitset （因为它可以排除掉大量的文档）。

4. *增量使用计数*.

   Elasticsearch 能够缓存非评分查询从而获取更快的访问，但是它也会不太聪明地缓存一些使用极少的东西。非评分计算因为倒排索引已经足够快了，所以我们只想缓存那些我们 *知道* 在将来会被再次使用的查询，以避免资源的浪费。

   为了实现以上设想，Elasticsearch 会为每个索引跟踪保留查询使用的历史状态。如果查询在最近的 256 次查询中会被用到，那么它就会被缓存到内存中。当 bitset 被缓存后，缓存会在那些低于 10,000 个文档（或少于 3% 的总索引数）的段（segment）中被忽略。这些小的段即将会消失，所以为它们分配缓存是一种浪费。

实际情况并非如此（执行有它的复杂性，这取决于查询计划是如何重新规划的，有些启发式的算法是基于查询代价的），理论上非评分查询 *先于* 评分查询执行。非评分查询任务旨在降低那些将对评分查询计算带来更高成本的文档数量，从而达到快速搜索的目的。

从概念上记住非评分计算是首先执行的，这将有助于写出高效又快速的搜索请求。

### 组合过滤器

前面的两个例子都是单个过滤器（filter）的使用方式。 在实际应用中，我们很有可能会过滤多个值或字段。比方说，怎样用 Elasticsearch 来表达下面的 SQL ？

```sql
SELECT product
FROM   products
WHERE  (price = 20 OR productID = "XHDK-A-1293-#fJ3")
  AND  (price != 30)
```



这种情况下，我们需要 `bool` （布尔）过滤器。 这是个 *复合过滤器（compound filter）* ，它可以接受多个其他过滤器作为参数，并将这些过滤器结合成各式各样的布尔（逻辑）组合。

#### 布尔过滤器

一个 `bool` 过滤器由三部分组成：

```js
{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
   }
}
```

- **`must`**

  所有的语句都 *必须（must）* 匹配，与 `AND` 等价。

- **`must_not`**

  所有的语句都 *不能（must not）* 匹配，与 `NOT` 等价。

- **`should`**

  至少有一个语句要匹配，与 `OR` 等价。

就这么简单！ 当我们需要多个过滤器时，只须将它们置入 `bool` 过滤器的不同部分即可。

一个 `bool` 过滤器的每个部分都是可选的（例如，我们可以只有一个 `must` 语句），而且每个部分内部可以只有一个或一组过滤器。

用 Elasticsearch 来表示本部分开始处的 SQL 例子，将两个 `term` 过滤器置入 `bool` 过滤器的 `should` 语句内，再增加一个语句处理 `NOT` 非的条件：

```json
GET /my_store/products/_search
{
   "query" : {
      "filtered" : { ①
         "filter" : {
            "bool" : {
              "should" : [
                 { "term" : {"price" : 20}},  ②
                 { "term" : {"productID" : "XHDK-A-1293-#fJ3"}}  ②
              ],
              "must_not" : {
                 "term" : {"price" : 30}  ③
              }
           }
         }
      }
   }
}
```

![image-20230301134634235](E:/Development/Typora/images/image-20230301134634235.png)

> filtered是比较老的的版本的语法。现在目前已经被bool替代。推荐使用bool。



我们搜索的结果返回了 2 个命中结果，两个文档分别匹配了 `bool` 过滤器其中的一个条件：

```json
"hits" : [
    {
        "_id" :     "1",
        "_score" :  1.0,
        "_source" : {
          "price" :     10,
          "productID" : "XHDK-A-1293-#fJ3" ①
        }
    },
    {
        "_id" :     "2",
        "_score" :  1.0,
        "_source" : {
          "price" :     20, 
          "productID" : "KDKE-B-9947-#kL5" ②
        }
    }
]
```

![image-20230301134727731](E:/Development/Typora/images/image-20230301134727731.png)

#### 嵌套布尔过滤器

尽管 `bool` 是一个复合的过滤器，可以接受多个子过滤器，需要注意的是 `bool` 过滤器本身仍然还只是一个过滤器。 这意味着我们可以将一个 `bool` 过滤器置于其他 `bool` 过滤器内部，这为我们提供了对任意复杂布尔逻辑进行处理的能力。

对于以下这个 SQL 语句：

```sql
SELECT document
FROM   products
WHERE  productID      = "KDKE-B-9947-#kL5"
  OR (     productID = "JODL-X-1937-#pV7"
       AND price     = 30 )
```

我们将其转换成一组嵌套的 `bool` 过滤器：

```json
GET /my_store/products/_search
{
   "query" : {
      "filtered" : {
         "filter" : {
            "bool" : {
              "should" : [
                { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, ①
                { "bool" : {  ①
                  "must" : [
                    { "term" : {"productID" : "JODL-X-1937-#pV7"}}, ②
                    { "term" : {"price" : 30}}  ②
                  ]
                }}
              ]
           }
         }
      }
   }
}
```

![image-20230301134805034](E:/Development/Typora/images/image-20230301134805034.png)

得到的结果有两个文档，它们各匹配 `should` 语句中的一个条件：

```json
"hits" : [
    {
        "_id" :     "2",
        "_score" :  1.0,
        "_source" : {
          "price" :     20,
          "productID" : "KDKE-B-9947-#kL5" ①
        }
    },
    {
        "_id" :     "3",
        "_score" :  1.0,
        "_source" : {
          "price" :      30,  ②
          "productID" : "JODL-X-1937-#pV7"  ②
        }
    }
]
```

![image-20230301134823732](E:/Development/Typora/images/image-20230301134823732.png)

这只是个简单的例子，但足以展示布尔过滤器可以用来作为构造复杂逻辑条件的基本构建模块。



### 查找多个精确值

`term` 查询对于查找单个值非常有用，但通常我们可能想搜索多个值。 如果我们想要查找价格字段值为 \$20 或 $30 的文档该如何处理呢？

不需要使用多个 `term` 查询，我们只要用单个 `terms` 查询（注意末尾的 *s* ）， `terms` 查询好比是 `term` 查询的复数形式（以英语名词的单复数做比）。

它几乎与 `term` 的使用方式一模一样，与指定单个价格不同，我们只要将 `term` 字段的值改为数组即可：

```js
{
    "terms" : {
        "price" : [20, 30]
    }
}
```



与 `term` 查询一样，也需要将其置入 `filter` 语句的常量评分查询中使用：

```json
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "terms" : {  ①
                    "price" : [20, 30]
                }
            }
        }
    }
}
```

> ①这个 `terms` 查询被置于 `constant_score` 查询中

运行结果返回第二、第三和第四个文档：

```json
"hits" : [
    {
        "_id" :    "2",
        "_score" : 1.0,
        "_source" : {
          "price" :     20,
          "productID" : "KDKE-B-9947-#kL5"
        }
    },
    {
        "_id" :    "3",
        "_score" : 1.0,
        "_source" : {
          "price" :     30,
          "productID" : "JODL-X-1937-#pV7"
        }
    },
    {
        "_id":     "4",
        "_score":  1.0,
        "_source": {
           "price":     30,
           "productID": "QQPX-R-3956-#aD8"
        }
     }
]
```

#### 包含，而不是相等

一定要了解 `term` 和 `terms` 是 *包含（contains）* 操作，而非 *等值（equals）* （判断）。 如何理解这句话呢？

如果我们有一个 term（词项）过滤器 `{ "term" : { "tags" : "search" } }` ，它会与以下两个文档 *同时* 匹配：

```js
{ "tags" : ["search"] }
{ "tags" : ["search", "open_source"] } ①
```

> ①尽管第二个文档包含除 `search` 以外的其他词，它还是被匹配并作为结果返回。

回忆一下 `term` 查询是如何工作的？ Elasticsearch 会在倒排索引中查找包括某 term 的所有文档，然后构造一个 bitset 。在我们的例子中，倒排索引表如下：

| Token       | DocIDs |
| ----------- | ------ |
| open_source | 2      |
| search      | 1,2    |

当 `term` 查询匹配标记 `search` 时，它直接在倒排索引中找到记录并获取相关的文档 ID，如倒排索引所示，这里文档 1 和文档 2 均包含该标记，所以两个文档会同时作为结果返回。

> 由于倒排索引表自身的特性，整个字段是否相等会难以计算，如果确定某个特定文档是否 *只（only）* 包含我们想要查找的词呢？首先我们需要在倒排索引中找到相关的记录并获取文档 ID，然后再扫描 *倒排索引中的每行记录* ，查看它们是否包含其他的 terms 。
>
> 可以想象，这样不仅低效，而且代价高昂。正因如此， `term` 和 `terms` 是 *必须包含（must contain）* 操作，而不是 *必须精确相等（must equal exactly）* 。

#### 精确相等

如果一定期望得到我们前面说的那种行为（即整个字段完全相等），最好的方式是增加并索引另一个字段， 这个字段用以存储该字段包含词项的数量，同样以上面提到的两个文档为例，现在我们包括了一个维护标签数的新字段：

```json
{ "tags" : ["search"], "tag_count" : 1 }
{ "tags" : ["search", "open_source"], "tag_count" : 2 }
```

一旦增加这个用来索引项 term 数目信息的字段，我们就可以构造一个 `constant_score` 查询，来确保结果中的文档所包含的词项数量与要求是一致的：

```sense
GET /my_index/my_type/_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                 "bool" : {
                    "must" : [
                        { "term" : { "tags" : "search" } }, ①
                        { "term" : { "tag_count" : 1 } } ②
                    ]
                }
            }
        }
    }
}
```

![image-20230301135426367](E:/Development/Typora/images/image-20230301135426367.png)

这个查询现在只会匹配具有单个标签 `search` 的文档，而不是任意一个包含 `search` 的文档。

### 范围

本章到目前为止，对于数字，只介绍如何处理精确值查询。实际上，对数字范围进行过滤有时会更有用。例如，我们可能想要查找所有价格大于 $20 且小于 $40 美元的产品。

在 SQL 中，范围查询可以表示为：

```sql
SELECT document
FROM   products
WHERE  price BETWEEN 20 AND 40
```



Elasticsearch 有 `range` 查询，不出所料地，可以用它来查找处于某个范围内的文档：

```js
"range" : {
    "price" : {
        "gte" : 20,
        "lte" : 40
    }
}
```



`range` 查询可同时提供包含（inclusive）和不包含（exclusive）这两种范围表达式，可供组合的选项如下：

- `gt`: `>` 大于（greater than）
- `lt`: `<` 小于（less than）
- `gte`: `>=` 大于或等于（greater than or equal to）
- `lte`: `<=` 小于或等于（less than or equal to）

**下面是一个范围查询的例子：.**

```json
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "price" : {
                        "gte" : 20,
                        "lt"  : 40
                    }
                }
            }
        }
    }
}
```

如果想要范围无界（比方说 >20 ），只须省略其中一边的限制：

```json
"range" : {
    "price" : {
        "gt" : 20
    }
}
```

#### 日期范围

`range` 查询同样可以应用在日期字段上：

```js
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-07 00:00:00"
    }
}
```

当使用它处理日期字段时， `range` 查询支持对 *日期计算（date math）* 进行操作，比方说，如果我们想查找时间戳在过去一小时内的所有文档：

```js
"range" : {
    "timestamp" : {
        "gt" : "now-1h"
    }
}
```

这个过滤器会一直查找时间戳在过去一个小时内的所有文档，让过滤器作为一个时间 *滑动窗口（sliding window）* 来过滤文档。

日期计算还可以被应用到某个具体的时间，并非只能是一个像 now 这样的占位符。只要在某个日期后加上一个双管符号 (`||`) 并紧跟一个日期数学表达式就能做到：

```json
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-01 00:00:00||+1M" ①
    }
}
```

> ①早于 2014 年 1 月 1 日加 1 月（2014 年 2 月 1 日 零时）

日期计算是 *日历相关（calendar aware）* 的，所以它不仅知道每月的具体天数，还知道某年的总天数（闰年）等信息。更详细的内容可以参考： [时间格式参考文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/mapping-date-format.html) 。

#### 字符串范围

`range` 查询同样可以处理字符串字段，字符串范围可采用 *字典顺序（lexicographically）* 或字母顺序（alphabetically）。例如，下面这些字符串是采用字典序（lexicographically）排序的：

- 5, 50, 6, B, C, a, ab, abb, abc, b

在倒排索引中的词项就是采取字典顺序（lexicographically）排列的，这也是字符串范围可以使用这个顺序来确定的原因。

如果我们想查找从 `a` 到 `b` （不包含）的字符串，同样可以使用 `range` 查询语法：

```js
"range" : {
    "title" : {
        "gte" : "a",
        "lt" :  "b"
    }
}
```

> **注意基数**
>
> 数字和日期字段的索引方式使高效地范围计算成为可能。但字符串却并非如此，要想对其使用范围过滤，Elasticsearch 实际上是在为范围内的每个词项都执行 `term` 过滤器，这会比日期或数字的范围过滤慢许多。
>
> 字符串范围在过滤 *低基数（low cardinality）* 字段（即只有少量唯一词项）时可以正常工作，但是唯一词项越多，字符串范围的计算会越慢。



### 处理Null值

回想在之前例子中，有的文档有名为 `tags` （标签）的字段，它是个多值字段，一个文档可能有一个或多个标签，也可能根本就没有标签。如果一个字段没有值，那么如何将它存入倒排索引中的呢？

这是个有欺骗性的问题，因为答案是：什么都不存。让我们看看之前内容里提到过的倒排索引：

| Token       | DocIDs |
| ----------- | ------ |
| open_source | 2      |
| search      | 1,2    |

如何将某个不存在的字段存储在这个数据结构中呢？无法做到！简单的说，一个倒排索引只是一个 token 列表和与之相关的文档信息，如果字段不存在，那么它也不会持有任何 token，也就无法在倒排索引结构中表现。

> 最终，这也就意味着，`null`, `[]` （空数组）和 `[null]` 所有这些都是等价的，它们无法存于倒排索引中。

显然，世界并不简单，数据往往会有缺失字段，或有显式的空值或空数组。为了应对这些状况，Elasticsearch 提供了一些工具来处理空或缺失值。

#### 存在查询

第一件武器就是 `exists` 存在查询。这个查询会返回那些在指定字段有任何值的文档，让我们索引一些示例文档并用标签的例子来说明：

```json
POST /my_index/posts/_bulk
{ "index": { "_id": "1"              }}
{ "tags" : ["search"]                }  
{ "index": { "_id": "2"              }}
{ "tags" : ["search", "open_source"] }  
{ "index": { "_id": "3"              }}
{ "other_field" : "some data"        }  
{ "index": { "_id": "4"              }}
{ "tags" : null                      }  
{ "index": { "_id": "5"              }}
{ "tags" : ["search", null]          }  
```

|      | `tags` 字段有 1 个值。                |
| ---- | ------------------------------------- |
|      | `tags` 字段有 2 个值。                |
|      | `tags` 字段缺失。                     |
|      | `tags` 字段被置为 `null` 。           |
|      | `tags` 字段有 1 个值和 1 个 `null` 。 |

以上文档集合中 `tags` 字段对应的倒排索引如下：

| Token         | DocIDs      |
| ------------- | ----------- |
| `open_source` | `2`         |
| `search`      | `1`,`2`,`5` |

我们的目标是找到那些被设置过标签字段的文档，并不关心标签的具体内容。只要它存在于文档中即可，用 SQL 的话就是用 `IS NOT NULL` 非空进行查询：

```sql
SELECT tags
FROM   posts
WHERE  tags IS NOT NULL
```

在 Elasticsearch 中，使用 `exists` 查询的方式如下：

```json
GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "exists" : { "field" : "tags" }
            }
        }
    }
}
```

这个查询返回 3 个文档：

```json
"hits" : [
    {
      "_id" :     "1",
      "_score" :  1.0,
      "_source" : { "tags" : ["search"] }
    },
    {
      "_id" :     "5",
      "_score" :  1.0,
      "_source" : { "tags" : ["search", null] } //①
    },
    {
      "_id" :     "2",
      "_score" :  1.0,
      "_source" : { "tags" : ["search", "open source"] }
    }
]
```

![image-20230306102111945](E:/Development/Typora/images/image-20230306102111945.png)

显而易见，只要 `tags` 字段存在项（term）的文档都会命中并作为结果返回，只有 3 和 4 两个文档被排除。

#### 缺失查询

这个 `missing` 查询本质上与 `exists` 恰好相反：它返回某个特定 *无* 值字段的文档，与以下 SQL 表达的意思类似：

```sql
SELECT tags
FROM   posts
WHERE  tags IS NULL
```

我们将前面例子中 `exists` 查询换成 `missing` 查询：

```json
GET /my_index/posts/_search
{
    "query" : {
        "constant_score" : {
            "filter": {
                "missing" : { "field" : "tags" }
            }
        }
    }
}
```



按照期望的那样，我们得到 3 和 4 两个文档（这两个文档的 `tags` 字段没有实际值）：

```json
"hits" : [
    {
      "_id" :     "3",
      "_score" :  1.0,
      "_source" : { "other_field" : "some data" }
    },
    {
      "_id" :     "4",
      "_score" :  1.0,
      "_source" : { "tags" : null }
    }
]
```

**当 null 的意思是 null**

有时候我们需要区分一个字段是没有值，还是它已被显式的设置成了 `null` 。在之前例子中，我们看到的默认的行为是无法做到这点的；数据被丢失了。不过幸运的是，我们可以选择将显式的 `null` 值替换成我们指定 *占位符（placeholder）* 。

在为字符串（string）、数字（numeric）、布尔值（Boolean）或日期（date）字段指定映射时，同样可以为之设置 `null_value` 空值，用以处理显式 `null` 值的情况。**不过即使如此，还是会将一个没有值的字段从倒排索引中排除。**

当选择合适的 `null_value` 空值的时候，需要保证以下几点：

> - 它会匹配字段的类型，我们不能为一个 `date` 日期字段设置字符串类型的 `null_value` 。
> - 它必须与普通值不一样，这可以避免把实际值当成 `null` 空的情况。

#### 对象上的存在与缺失

不仅可以过滤核心类型， `exists` and `missing` 查询 还可以处理一个对象的内部字段。以下面文档为例：

```js
{
   "name" : {
      "first" : "John",
      "last" :  "Smith"
   }
}
```

我们不仅可以检查 `name.first` 和 `name.last` 的存在性，也可以检查 `name` ，不过在 [映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping.html) 中，如上对象的内部是个扁平的字段与值（field-value）的简单键值结构，类似下面这样：

```js
{
   "name.first" : "John",
   "name.last"  : "Smith"
}
```

那么我们如何用 `exists` 或 `missing` 查询 `name` 字段呢？ `name` 字段并不真实存在于倒排索引中。

原因是当我们执行下面这个过滤的时候：

```js
{
    "exists" : { "field" : "name" }
}
```

实际执行的是：

```js
{
    "bool": {
        "should": [
            { "exists": { "field": "name.first" }},
            { "exists": { "field": "name.last" }}
        ]
    }
}
```

这也就意味着，如果 `first` 和 `last` 都是空，那么 `name` 这个命名空间才会被认为不存在。





## 全文搜索*full-text search）*

### 基于词项与基于全文

所有查询会或多或少的执行相关度计算，但不是所有查询都有分析阶段。和一些特殊的完全不会对文本进行操作的查询（如 `bool` 或 `function_score` ）不同，文本查询可以划分成两大家族：

- **基于词项的查询**
  - 如 `term` 或 `fuzzy` 这样的底层查询不需要分析阶段，它们对单个词项进行操作。用 `term` 查询词项 `Foo` 只要在倒排索引中查找 *准确词项* ，并且用 TF/IDF 算法为每个包含该词项的文档计算相关度评分 `_score` 。
  - **记住 `term` 查询只对倒排索引的词项精确匹配**，这点很重要，它不会对词的多样性进行处理（如， `foo` 或 `FOO` ）。这里，无须考虑词项是如何存入索引的。如果是将 `["Foo","Bar"]` 索引存入一个不分析的（ `not_analyzed` ）包含精确值的字段，或者将 `Foo Bar` 索引到一个带有 `whitespace` 空格分析器的字段，两者的结果都会是在倒排索引中有 `Foo` 和 `Bar` 这两个词。
- **基于全文的查询**
  - 像 `match` 或 `query_string` 这样的查询是高层查询，它们了解字段映射的信息：如果查询 `日期（date）` 或 `整数（integer）` 字段，它们会将查询字符串分别作为日期或整数对待。
  - 如果查询一个（ `not_analyzed` ）未分析的精确值字符串字段，它们会将整个查询字符串作为单个词项对待。
  - 但如果要查询一个（ `analyzed` ）已分析的全文字段，它们会先将查询字符串传递到一个合适的分析器，然后生成一个供查询的词项列表。一旦组成了词项列表，这个查询会对每个词项逐一执行底层的查询，再将结果合并，然后为每个文档生成一个最终的相关度评分。

我们很少直接使用基于词项的搜索，通常情况下都是对全文进行查询，而非单个词项，这只需要简单的执行一个高层全文查询（==进而在高层查询内部会以基于词项的底层查询完成搜索==）。

当我们想要查询一个具有精确值的 `not_analyzed` 未分析字段之前，需要考虑，是否真的采用评分查询，或者非评分查询会更好。

单词项查询通常可以用是、非这种二元问题表示，所以更适合用过滤，而且这样做可以有效利用[缓存](https://www.elastic.co/guide/cn/elasticsearch/guide/current/filter-caching.html)：

```json
GET /_search
{
    "query": {
        "constant_score": {
            "filter": {
                "term": { "gender": "female" }
            }
        }
    }
}
```

### 匹配查询

匹配查询 `match` 是个 *核心* 查询。无论需要查询什么字段， `match` 查询都应该会是首选的查询方式。它是一个高级 *全文查询* ，这表示它既能处理全文字段，又能处理精确字段。

这就是说， `match` 查询主要的应用场景就是进行全文搜索，我们以下面一个简单例子来说明全文搜索是如何工作的：

#### 索引一些数据

首先，我们使用 [`bulk` API](https://www.elastic.co/guide/cn/elasticsearch/guide/current/bulk.html) 创建一些新的文档和索引：

```json
DELETE /my_index  //①

PUT /my_index
{ "settings": { "number_of_shards": 1 }}  //②

POST /my_index/my_type/_bulk
{ "index": { "_id": 1 }}
{ "title": "The quick brown fox" }
{ "index": { "_id": 2 }}
{ "title": "The quick brown fox jumps over the lazy dog" }
{ "index": { "_id": 3 }}
{ "title": "The quick brown fox jumps over the quick dog" }
{ "index": { "_id": 4 }}
{ "title": "Brown fox brown dog" }
```

![image-20230306104522418](E:/Development/Typora/images/image-20230306104522418.png)

#### 单个词查询

我们用第一个示例来解释使用 `match` 查询搜索全文字段中的单个词：

```json
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "QUICK!"
        }
    }
}
```

Elasticsearch 执行上面这个 `match` 查询的步骤是：

1. *检查字段类型* 。

   标题 `title` 字段是一个 `string` 类型（ `analyzed` ）已分析的全文字段，这意味着查询字符串本身也应该被分析。

2. *分析查询字符串* 。

   将查询的字符串 `QUICK!` 传入标准分析器中，输出的结果是单个项 `quick` 。==因为只有一个单词项，所以 `match` 查询执行的是单个底层 `term` 查询。==

3. *查找匹配文档* 。

   用 `term` 查询在倒排索引中查找 `quick` 然后获取一组包含该项的文档，本例的结果是文档：1、2 和 3 。

4. *为每个文档评分* 。

   用 `term` 查询计算每个文档相关度评分 `_score` ，这是种将词频（term frequency，即词 `quick` 在相关文档的 `title` 字段中出现的频率）和反向文档频率（inverse document frequency，即词 `quick` 在所有文档的 `title` 字段中出现的频率），以及字段的长度（即字段越短相关度越高）相结合的计算方式。参见 [相关性的介绍](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-intro.html) 。

这个过程给我们以下（经缩减）结果：

```js
"hits": [
 {
    "_id":      "1",
    "_score":   0.5, 
    "_source": {
       "title": "The quick brown fox"
    }
 },
 {
    "_id":      "3",
    "_score":   0.44194174, 
    "_source": {
       "title": "The quick brown fox jumps over the quick dog"
    }
 },
 {
    "_id":      "2",
    "_score":   0.3125, 
    "_source": {
       "title": "The quick brown fox jumps over the lazy dog"
    }
 }
]
```



### 多词查询

如果我们一次只能搜索一个词，那么全文搜索就会不太灵活，幸运的是 `match` 查询让多词查询变得简单：

```json
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
        }
    }
}
```

上面这个查询返回所有四个文档：

```json
{
  "hits": [
     {
        "_id":      "4",
        "_score":   0.73185337, 
        "_source": {
           "title": "Brown fox brown dog"
        }
     },
     {
        "_id":      "2",
        "_score":   0.47486103, 
        "_source": {
           "title": "The quick brown fox jumps over the lazy dog"
        }
     },
     {
        "_id":      "3",
        "_score":   0.47486103, 
        "_source": {
           "title": "The quick brown fox jumps over the quick dog"
        }
     },
     {
        "_id":      "1",
        "_score":   0.11914785, 
        "_source": {
           "title": "The quick brown fox"
        }
     }
  ]
}
```

因为 `match` 查询必须查找两个词（ `["brown","dog"]` ），==它在内部实际上先执行两次 `term` 查询，然后将两次查询的结果合并作为最终结果输出。==为了做到这点，它将两个 `term` 查询包入一个 `bool` 查询中，详细信息见 [布尔查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/bool-query.html)。

以上示例告诉我们一个重要信息：即任何文档只要 `title` 字段里包含 *指定词项中的至少一个词* 就能匹配，被匹配的词项越多，文档就越相关。

#### 提高精度

用 *任意* 查询词项匹配文档可能会导致结果中出现不相关的长尾。这是种散弹式搜索。可能我们只想搜索包含 *所有* 词项的文档，也就是说，不去匹配 `brown OR dog` ，而通过匹配 `brown AND dog` 找到所有文档。

`match` 查询还可以接受 `operator` 操作符作为输入参数，默认情况下该操作符是 `or` 。我们可以将它修改成 `and` 让所有指定词项都必须匹配：

```json
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": {      //①
                "query":    "BROWN DOG!",
                "operator": "and"
            }
        }
    }
}
```

> ①  `match` 查询的结构需要做稍许调整才能使用 `operator` 操作符参数。



这个查询可以把文档 1 排除在外，因为它只包含两个词项中的一个。

#### 控制精度

在 *所有* 与 *任意* 间二选一有点过于非黑即白。如果用户给定 5 个查询词项，想查找只包含其中 4 个的文档，该如何处理？将 `operator` 操作符参数设置成 `and` 只会将此文档排除。

有时候这正是我们期望的，但在全文搜索的大多数应用场景下，我们既想包含那些可能相关的文档，同时又排除那些不太相关的。换句话说，我们想要处于中间某种结果。

`match` 查询支持 `minimum_should_match` 最小匹配参数，这让我们可以指定必须匹配的词项数用来表示一个文档是否相关。我们可以将其设置为某个具体数字，更常用的做法是将其设置为一个百分数，因为我们无法控制用户搜索时输入的单词数量：

```json
GET /my_index/my_type/_search
{
  "query": {
    "match": {
      "title": {
        "query":   "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
```

当给定百分比的时候， `minimum_should_match` 会做合适的事情：在之前三词项的示例中， `75%` 会自动被截断成 `66.6%` ，即三个里面两个词。无论这个值设置成什么，至少包含一个词项的文档才会被认为是匹配的。

> 参数 `minimum_should_match` 的设置非常灵活，可以根据用户输入词项的数目应用不同的规则。完整的信息参考文档 https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-minimum-should-match.html#query-dsl-minimum-should-match



### 组合查询

在 [组合过滤器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/combining-filters.html) 中，我们讨论过如何使用 `bool` 过滤器通过 `and` 、 `or` 和 `not` 逻辑组合将多个过滤器进行组合。在查询中， `bool` 查询有类似的功能，只有一个重要的区别。

过滤器做二元判断：文档是否应该出现在结果中？但查询更精妙，它除了决定一个文档是否应该被包括在结果中，还会计算文档的 *相关程度* 。

与过滤器一样， `bool` 查询也可以接受 `must` 、 `must_not` 和 `should` 参数下的多个查询语句。比如：

```json
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "quick" }},
      "must_not": { "match": { "title": "lazy"  }},
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ]
    }
  }
}
```

以上的查询结果返回 `title` 字段包含词项 `quick` 但不包含 `lazy` 的任意文档。目前为止，这与 `bool` 过滤器的工作方式非常相似。

区别就在于两个 `should` 语句，也就是说：一个文档不必包含 `brown` 或 `dog` 这两个词项，但如果一旦包含，我们就认为它们 *更相关* ：

```js
{
  "hits": [
     {
        "_id":      "3",
        "_score":   0.70134366, 
        "_source": {
           "title": "The quick brown fox jumps over the quick dog"
        }
     },
     {
        "_id":      "1",
        "_score":   0.3312608,
        "_source": {
           "title": "The quick brown fox"
        }
     }
  ]
}
```

#### 评分计算

`bool` 查询会为每个文档计算相关度评分 `_score` ，再将所有匹配的 `must` 和 `should` 语句的分数 `_score` 求和，最后除以 `must` 和 `should` 语句的总数。

> `must_not` 语句不会影响评分；它的作用只是将不相关的文档排除。

#### 控制精度

所有 `must` 语句必须匹配，所有 `must_not` 语句都必须不匹配，但有多少 `should` 语句应该匹配呢？默认情况下，没有 `should` 语句是必须匹配的，只有一个例外：那就是当没有 `must` 语句的时候，至少有一个 `should` 语句必须匹配。

就像我们能控制 [`match` 查询的精度](https://www.elastic.co/guide/cn/elasticsearch/guide/current/match-multi-word.html#match-precision) 一样，我们可以通过 `minimum_should_match` 参数控制需要匹配的 `should` 语句的数量，它既可以是一个绝对的数字，又可以是个百分比：

```json
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "brown" }},
        { "match": { "title": "fox"   }},
        { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 //①
    }
  }
}
```

>  ① 这也可以用百分比表示。

这个查询结果会将所有满足以下条件的文档返回： `title` 字段包含 `"brown" AND "fox"` 、 `"brown" AND "dog"` 或 `"fox" AND "dog"` 。如果有文档包含所有三个条件，它会比只包含两个的文档更相关。



### 如何使用布尔匹配

目前为止，可能已经意识到[多词 `match` 查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/match-multi-word.html)只是简单地将生成的 `term` 查询包裹在一个 `bool` 查询中。如果使用默认的 `or` 操作符，每个 `term` 查询都被当作 `should` 语句，这样就要求必须至少匹配一条语句。以下两个查询是等价的：

```js
{
    "match": { "title": "brown fox"}
}
```

 

```js
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

如果使用 `and` 操作符，所有的 `term` 查询都被当作 `must` 语句，所以 *所有（all）* 语句都必须匹配。以下两个查询是等价的：

```json
{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
```

 

```js
{
  "bool": {
    "must": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```



如果指定参数 `minimum_should_match` ，它可以通过 `bool` 查询直接传递，使以下两个查询等价：

```json
{
    "match": {
        "title": {
            "query":  "quick brown fox",
            "minimum_should_match": "75%"
        }
    }
}
```

 

```json
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }},
      { "term": { "title": "quick" }}
    ],
    "minimum_should_match": 2  //①
  }
}
```

> ①  因为只有三条语句，`match` 查询的参数 `minimum_should_match` 值 75% 会被截断成 `2` 。即三条 `should` 语句中至少有两条必须匹配。



当然，我们通常将这些查询用 `match` 查询来表示，但是如果了解 `match` 内部的工作原理，我们就能根据自己的需要来控制查询过程。有些时候单个 `match` 查询无法满足需求，比如为某些查询条件分配更高的权重。我们会在下一小节中看到这个例子。



### 查询语句提升权重

当然 `bool` 查询不仅限于组合简单的单个词 `match` 查询，它可以组合任意其他的查询，以及其他 `bool` 查询。普遍的用法是通过汇总多个独立查询的分数，从而达到为每个文档微调其相关度评分 `_score` 的目的。

假设想要查询关于 “full-text search（全文搜索）” 的文档，但我们希望为提及 “Elasticsearch” 或 “Lucene” 的文档给予更高的 *权重* ，这里 *更高权重* 是指如果文档中出现 “Elasticsearch” 或 “Lucene” ，它们会比没有的出现这些词的文档获得更高的相关度评分 `_score` ，也就是说，它们会出现在结果集的更上面。

一个简单的 `bool` *查询* 允许我们写出如下这种非常复杂的逻辑：

```json
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {
                    "content": {  //①
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [  //er
                { "match": { "content": "Elasticsearch" }},
                { "match": { "content": "Lucene"        }}
            ]
        }
    }
}
```

![image-20230306111044597](E:/Development/Typora/images/image-20230306111044597.png)

`should` 语句匹配得越多表示文档的相关度越高。目前为止还挺好。

但是如果我们想让包含 `Lucene` 的有更高的权重，并且包含 `Elasticsearch` 的语句比 `Lucene` 的权重更高，该如何处理?

我们可以通过指定 `boost` 来控制任何查询语句的相对的权重， `boost` 的默认值为 `1` ，大于 `1` 会提升一个语句的相对权重。所以下面重写之前的查询：

```json
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  //①
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3  //②
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 //③
                    }
                }}
            ]
        }
    }
}
```

![image-20230306111220948](E:/Development/Typora/images/image-20230306111220948.png)

> `boost` 参数被用来提升一个语句的相对权重（ `boost` 值大于 `1` ）或降低相对权重（ `boost` 值处于 `0` 到 `1` 之间），==但是这种提升或降低并不是线性的，换句话说，如果一个 `boost` 值为 `2` ，并不能获得两倍的评分 `_score` 。==
>
> 相反，新的评分 `_score` 会在应用权重提升之后被 *归一化* ，每种类型的查询都有自己的归一算法，细节超出了本书的范围，所以不作介绍。简单的说，更高的 `boost` 值为我们带来更高的评分 `_score` 。
>
> 如果不基于 TF/IDF 要实现自己的评分模型，我们就需要对权重提升的过程能有更多控制，可以使用 [`function_score` 查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/function-score-query.html)操纵一个文档的权重提升方式而跳过归一化这一步骤。



### 控制分析

查询只能查找倒排索引表中真实存在的项，**所以保证文档在索引时与查询字符串在搜索时应用相同的分析过程非常重要**，这样查询的项才能够匹配倒排索引中的项。

尽管是在说 *文档* ，不过分析器可以由每个字段决定。每个字段都可以有不同的分析器，既可以通过配置为字段指定分析器，也可以使用更高层的类型（type）、索引（index）或节点（node）的默认配置。在索引时，一个字段值是根据配置或默认分析器分析的。

例如为 `my_index` 新增一个字段：

```json
PUT /my_index/_mapping/my_type
{
    "my_type": {
        "properties": {
            "english_title": {
                "type":     "string",
                "analyzer": "english"
            }
        }
    }
}
```

现在我们就可以通过使用 `analyze` API 来分析单词 `Foxes` ，进而比较 `english_title` 字段和 `title` 字段在索引时的分析结果：

```json
GET /my_index/_analyze
{
  "field": "my_type.title",   
  "text": "Foxes"
}

GET /my_index/_analyze
{
  "field": "my_type.english_title",   
  "text": "Foxes"
}
```

这意味着，如果使用底层 `term` 查询精确项 `fox` 时， `english_title` 字段会匹配但 `title` 字段不会。

如同 `match` 查询这样的高层查询知道字段映射的关系，能为每个被查询的字段应用正确的分析器。可以使用 `validate-query` API 查看这个行为：

```json
GET /my_index/my_type/_validate/query?explain
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title":         "Foxes"}},
                { "match": { "english_title": "Foxes"}}
            ]
        }
    }
}
```

返回语句的 `explanation` 结果：

```json
(title:foxes english_title:fox)
```

`match` 查询为每个字段使用合适的分析器，以保证它在寻找每个项时都为该字段使用正确的格式。

#### 默认分析器

虽然我们可以在字段层级指定分析器，但是如果该层级没有指定任何的分析器，那么我们如何能确定这个字段使用的是哪个分析器呢？

分析器可以从三个层面进行定义：按字段（per-field）、按索引（per-index）或全局缺省（global default）。Elasticsearch 会按照以下顺序依次处理，直到它找到能够使用的分析器。索引时的顺序如下：

- 字段映射里定义的 `analyzer` ，否则
- 索引设置中名为 `default` 的分析器，默认为
- `standard` 标准分析器

在搜索时，顺序有些许不同：

- 查询自己定义的 `analyzer` ，否则
- 字段映射里定义的 `analyzer` ，否则
- 索引设置中名为 `default` 的分析器，默认为
- `standard` 标准分析器

==有时，在索引时和搜索时使用不同的分析器是合理的==。我们可能要想为同义词建索引（例如，所有 `quick` 出现的地方，同时也为 `fast` 、 `rapid` 和 `speedy` 创建索引）。但在搜索时，我们不需要搜索所有的同义词，取而代之的是寻找用户输入的单词是否是 `quick` 、 `fast` 、 `rapid` 或 `speedy` 。

为了区分，Elasticsearch 也支持一个可选的 `search_analyzer` 映射，它仅会应用于搜索时（ `analyzer` 还用于索引时）。还有一个等价的 `default_search` 映射，用以指定索引层的默认配置。

如果考虑到这些额外参数，一个搜索时的 *完整* 顺序会是下面这样：

- 查询自己定义的 `analyzer` ，否则
- 字段映射里定义的 `search_analyzer` ，否则
- 字段映射里定义的 `analyzer` ，否则
- 索引设置中名为 `default_search` 的分析器，默认为
- 索引设置中名为 `default` 的分析器，默认为
- `standard` 标准分析器

### 分析器配置实践

就可以配置分析器地方的数量而言是十分惊人的，但是实际非常简单。

#### 保持简单

多数情况下，会提前知道文档会包括哪些字段。最简单的途径就是在创建索引或者增加类型映射时，为每个全文字段设置分析器。这种方式尽管有点麻烦，但是它让我们可以清楚的看到每个字段每个分析器是如何设置的。

通常，多数字符串字段都是 `not_analyzed` 精确值字段，比如标签（tag）或枚举（enum），而且更多的全文字段会使用默认的 `standard` 分析器或 `english` 或其他某种语言的分析器。这样只需要为少数一两个字段指定自定义分析：或许标题 `title` 字段需要以支持 *输入即查找（find-as-you-type）* 的方式进行索引。

可以在索引级别设置中，为绝大部分的字段设置你想指定的 `default` 默认分析器。然后在字段级别设置中，对某一两个字段配置需要指定的分析器。

对于和时间相关的日志数据，通常的做法是每天自行创建索引，由于这种方式不是从头创建的索引，仍然可以用 [索引模板（Index Template）](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/indices-templates.html) 为新建的索引指定配置和映射。





