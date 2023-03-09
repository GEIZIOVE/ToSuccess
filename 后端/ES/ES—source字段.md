# ES— _source、doc_values and store字段

## 简介 

Elasticsearch（以下简称ES）和solr所基于的lucene提供了两种方式去**存储和检索**字段: stored fields 和 doc values. 此外,es使用默认的`_source`字段,==这是一个包含文档的所有字段的**json**==,它在编索引期间(index time)被作为输入.

es里的`source`字段用来存储文档的原始信息，默认是开启的。

比如我们写入两篇文档，

```json
PUT student/_doc/1
{
  "name":"Jack",
  "age": 15,
  "like": "hiking,basketball"
}

PUT student/_doc/2
{
  "name":"Tom",
  "age": 16,
  "like": "football,swiming,basketball"
}
```


ES首先会建立倒排索引，这个是为了方便我们搜索。

<img src="E:/Development/Typora/images/image-20230227183657341.png" alt="image-20230227183657341" style="zoom: 33%;" />

同时，默认情况下，`source`字段是会存储数据的原始信息的。而存储的结构一般称为正排索引。 如下图所示：

<img src="E:/Development/Typora/images/image-20230227183722094.png" alt="image-20230227183722094" style="zoom:67%;" />



我们查询的时候，符合条件的文档会通过`_source`字段返回原始的信息：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.6931471,
    "hits" : [
      {
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.6931471,
        "_source" : {
          "name" : "Jack",
          "age" : 15,
          "like" : "hiking,basketball"
        }
      }
    ]
  }
}
```



## lucene中的Stored fields 和 doc values



在lucene中将一个文档编入索引时,字段的原始值会丢失. 字段被分析(analyze),转换(transform)以至编入索引. 在没有其他额外添加的数据结构的情况下,当我们检索文档时,我们只能得到被检索文档的id,而没有文档的原始字段. 为了去获得这些信息,我们需要额外的数据结构. lucene提供了两种可能性来实现这个,stored fields 和 doc values.

### store fields

stored fields用于存储没有经过任何分析(without any analysis)的字段值,以实现在查询时能得到这些原始值.

### doc values

doc values用于加速一些诸如聚合(aggregation),排序(sorting),分组(grouping)的操作. doc values 也可以被用来在查询时返回字段值. 例外是doc values不能用来存储text类型的字段.

## Elasticsearch中的字段检索

我们可以在mapping中通过以下配置,指定某个字段是否启用 Stored Fields 和 Doc Values

```json
"properties" : {
    "field": {
      "type": "keyword",
      "store": true, //是否存储原始值在store fields中
      "doc_values" true //是否存储原始值在doc values中
    }
  }
```

默认情况下,`store`为false;而除了text类型的字段外,`doc_values`都为true.

除了stored fields 和 doc values之外,es还使用`_source`字段来检获得字段值,以使得在查询期间命中的文档(doc)的所有字段值都可以被返回.



## source

source 用于存储在索引期间(index time)传给es的原始json. 可通过以下配置决定是否启用:

```json
"mappings": {
    "_source": {
      "enabled": false //是否启用source
    }
  }
```

默认会存储所有字段,可以通过以下方式指定原始json中要排除的字段,这可以减少存到source中的数据大小,从而减少磁盘空间占用.但是在查询时,将无法得到被排除字段的原始值.

```json
"mappings": {
    "_source": {
      "excludes": [
        "meta.description",
        "meta.other.*"
      ]
    }
  }
```

在查询期,默认被命中文档的所有字段都会被返回,你也可以指定只返回source中的一部分字段,这可以使网络传输数据量减少.

如果禁用es的source(即不将文档json存储到source),将使得每次更新文档时都不得不重新索引(reindex)这个文档. 概念上讲,更新一个文档的字段值时,我们需要获得旧文档的该字段值,然后讲新值写入.在没有source的情况下,旧文档的某个字段值可以通过doc values或stored fields来获得.(solr中的原子更新就是通过这种方式来取旧值的) 然而,由于设计上的原因,es中这不被允许.如果你需要更新你的文档,就必须要在配置中启用source.





在实际的项目中，有时候我们并不需要这个source字段，只是希望拿到查询的结果，在一些业务场景下我们往往会用数据库的主键作为文档id。然后用这个id再进一步去其它的存储介质拿数据。因为source字段默认是打开的，如果你不了解这个机制，就白白浪费了ES的存储空间。

ES还有一种机制，可以在查询阶段控制返回source里面的部分内容，比如我只返回文档里的name字段，可以这样：

```json
GET student/_search?_source=name
{
  "query": {
    "match": {
      "like": "hiking"
    }
  }
}
```

我们来看下source字段不存储的例子。

设置mapping，source字段关闭。

```json
PUT student_new
{
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
      "age": {
        "type": "long"
      },
      "like": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```


写入跟上面一样的数据，用新的索引查询结果示例如下：

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.7549127,
    "hits" : [
      {
        "_index" : "student_new",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.7549127
      }
    ]
  }
}
```

可以看到source字段没有了。但是我们依然能够拿到文档的id。如果一条文档节省10KB，几亿文档的话节省的存储空间还是相当可观的。

source字段不存储确实可以节省不少存储空间，但是有几种情况是必须保留这个字段的。我这里列举下：

- 文档需要使用update或者update_by_query更新

- 会用到reindex
- 会用到文档高亮



### 检索字段

在配置索引时你可以为一个字段启用或禁用`_source`,以及`store`和`doc_values`. 与之相应,查询时,你也可选择从那里取这个字段,这可以通过以下方式实现.

```json
"_source": { //从source中取
    "includes": [
        "address"
    ],
    "excludes": [
        "!address"
    ],
},
"stored_fields":["activated"],  //从store中取
"docvalue_fields":["address.keyword"] //从docvalue中取
```

如果一个字段被同时开启了source,store,docvalues,那么你可以选择从3个源中取值. 从功能的角度看,得到的结果是一样的,但是不同的选择将对查询的时间产生影响.



## Stored fields & Doc values & source 的内部结构

这一部分,我只给这几种存储方式一个基本的概述,为了去理解为什么它们在性能上表现的差异.

### Stored fields 内部结构

Stored fields在磁盘上以行的方式放置: 每个文档对应一个行,这个行包含该文档所有的stored fields.

![img](E:/Development/Typora/images/v2-6565bbe8a6ceff07801b40cbc2ca5de5_720w.webp)

以上图作为例子. 要访问文档X的fields3,我们需要访问这个文档X的行,并且跳过field3前面的字段. 这个过程需要计算偏移地址, 虽然对性能影响并不特别大,但也不是一点都没有.

### Doc values 内部结构

Doc values 以列的方式存储. 不同文档的相同字段被连续地存储在一起. 这种存储方式,可以直接访问一个特定的文档的特定字段. 计算想要字段值的偏移地址有一定成本.但是想象一下,假如我们只想要文档某一个字段,这种存储方式显然更有效率.

### Source 内部结构

如我们上面提到,source是一个大字段,包含索引期的文档json. 那么source又是如何存储的呢? 不出意料, es利用了已经被lucene实现的机制: stored fields. 具体地讲,source 字段是一行stored fields中的第一个字段.

![img](E:/Development/Typora/images/v2-82fcd0090e1f77e7450d8736a43b2fab_720w.webp)

需要注意的是,当要读source中的部分字段时,需要把整个source中的json读取出来. 如果本就希望查出所有字段,那么直接读source显然是最快的. 而如果只是想读个别字段,读source则会带来不必要的性能影响.

