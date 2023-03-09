*   原文地址: [Processors](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-processors.html)

*   版本:[6.0.0](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)

**说明**

本文是建立在有一些Elasticsearch基础和了解相关Pipeline概念的人

Ingest Node
===========

## 简介Ingest Node

**Ingest Node(预处理节点)**是ES用于功能上命名的一种节点类型,可以通过在elasticsearch.xml进行如下配置来标识出集群中的某个节点是否是Ingest Node.

```xml
node.ingest: false
```

> 上述将node.ingest设置成false,则表明当前节点不是Ingest Node,不具有预处理能力,当然Elasticsearch默认所有节点都是Ingest Node,即集群中所有的节点都具有预处理能力



### **何为预处理呢?**

用过Logstash对日志进行处理的用户都知道,**一般情况下我们并不会直接将原始的日志进行加载到Elasticsearch集群,而是对原始日志信息进行(深)加工后保存到Elasticsearch集群中.**比如Logstash支持多种解析器比如json,kv,date等,比较经典的是grok.这里我们不会对Logstash的解析器进行详细说明,只是为了描述一个问题.

有些时候我们需要Logstash来对加载到Elasticsearch中的数据进行处理,这个处理,从概念上而言,我们也能称之为"预处理".而这里我们所说的预处理也其实就是类似的概念.

可以这么说,在Elasticsearch没有提供IngestNode这一概念时,我们想对存储在Elasticsearch里的数据在存储之前进行加工处理的话,我们只能依赖Logstash或自定义插件来完成这一功能,但是在Elasticsearch 5.x版本中,官方在内部集成了部分Logstash的功能,这就是Ingest,而具有Ingest能力的节点称之为**Ingest Node.**

请查看[Filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html) 来了解更多关于Logstash解析器的内容.

如果要脱离Logstash来对在Elasticsearch写入文档之前对文档进行加工处理,比如为文档某个字段设置默认值,重命名某个字段,设置通过脚本来完成更加复杂的加工逻辑,我们则必须要了解两个基本概念: `Pipeline` 和 `Processors`.

参考: [(译) Ingest Node (预处理节点)](https://www.felayman.com/articles/2017/11/24/1511515806564.html) 或[Ingest Node](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html) 来了解更多关于上述两个概念.下文只是简单说明两者.

## 简介Pipeline、Processors

管道(Pipeline)是众所周知的一个概念,Elasticsearch引入这一概念,是为了让那些有过工作经验的人来说更直白,更轻松的理解这一概念.

本人是主要用Java进行开发,**这里就以Pipeline和java中的Stream进行类比,两者从功能和概念上很类似**,我们经常会对Stream中的数据进行处理,比如map操作,peek操作,reduce操作,count操作等,这些操作从行为上说,就是对数据的加工,而Pipeline也是如此,Pipeline也会对通过该Pipeline的数据(一般来说是文档)进行加工,比如上面说到的,修改文档的某个字段值,修改文档某个字段的类型等等.

而Elasticsearch对该加工行为进行抽象包装,并称之为`Processors`.Elasticsearch命名了多种类型的Processors来规范对文档的操作,比如==set,append,date,join,json,kv等等==.这些不同类型的Processors,我们会在后文进行说明

## Pipeline定义

定义一个Pipeline是件很简单的事情,官方给出了参考:

```json
PUT _ingest/pipeline/my-pipeline-id
{
  "description" : "describe pipeline",
  "processors" : [
    {
      "set" : {
        "field": "foo",
        "value": "bar"
      }
    }
  ]
}
```

上面的例子,表明通过指定的URL请求"`\_ingest/pipeline`"定义了一个ID为"my-pipeline-id"的pipeline,其中请求体中的存在两个必须要的元素:

*   **description** 描述该pipeline是做什么的
*   **processors** 定义了一系列的processors,这里只是简单的定义了一个赋值操作,即将字段名为"foo"的字段值都设置为"bar"

如果需要了解更多关于Pipeline定义的信息,可以参考: [Ingest APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/put-pipeline-api.html)

## 简介 Simulate Pipeline API

既然Elasticsearch提供了预处理的能力,总不能是黑盒处理吧,**为了让开发者更好的了解和使用预处理的方式和原理,官方也提供了相关的接口,来让我们对这些预处理操作进行测试**,这些接口,官方称之为: `Simulate Pipeline API.`想要深入了解pipeline以及processors的使用,我们基本离不开Simulate Pipeline API来辅助帮我们完成许多工作.

下面是一个比较简单的Simulate Pipeline API的例子:

```json
POST _ingest/pipeline/_simulate
{
  "pipeline" : {
    // pipeline definition here
  },
  "docs" : [
    { "_source": {/** first document **/} },
    { "_source": {/** second document **/} },
    // ...
  ]
}


```

上面的例子中,在请求URL中并没有明确指定使用哪个pipeline,这种情况下需要我们在请求体中即时定义一个,当然我们也可以在请求URL中指定一个pipeline的ID,如下面例子:

```json
POST _ingest/pipeline/my-pipeline-id/_simulate
{
  "docs" : [
    { "_source": {/** first document **/} },
    { "_source": {/** second document **/} },
    // ...
  ]
}


```

这样一来,在请求体中我们定义的"docs"中的内容就能够使用该pipeline和该pipeline种的processors,下面是一个比较具体的例子:

```json
POST _ingest/pipeline/_simulate
{
  "pipeline" :
  {
    "description": "_description",
    "processors": [
      {
        "set" : {
          "field" : "field2",
          "value" : "_value"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "foo": "bar"
      }
    },
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "foo": "rab"
      }
    }
  ]
}


```

在上面具体的例子中,我们并没有在请求URL中指定使用哪个pipeline,因此我们不得不在请求体中即时定义一个pipeline和对应的processors,例子很简单,只是让通过该pipeline的的文档中的字段名称为"field2"的字段值为"\_value",文档我们也指定了,就是"docs"中定义的文档,该请求的响应信息如下:

```json
{
   "docs": [
      {
         "doc": {
            "_id": "id",
            "_index": "index",
            "_type": "type",
            "_source": {
               "field2": "_value",
               "foo": "bar"
            },
            "_ingest": {
               "timestamp": "2017-05-04T22:30:03.187Z"
            }
         }
      },
      {
         "doc": {
            "_id": "id",
            "_index": "index",
            "_type": "type",
            "_source": {
               "field2": "_value",
               "foo": "rab"
            },
            "_ingest": {
               "timestamp": "2017-05-04T22:30:03.188Z"
            }
         }
      }
   ]
}


```

从具体的响应结果中看到,在文档通过pipeline时(或理解为被pipeline中的processors加工后),新的文档与原有的文档产生了差异,这些差异体现为:

> 1.  文档都**新增**了field2字段,这点我们可以通过对比响应前后的定义在"docs"中文档的\_source内容中看出
> 2.  **额外增加了一些临时(官方称之为瞬态)的字段**,比如timestamp,他们都在\_ingest节点下,(这些)字段都是临时与源文档存在一起,在被pipeline中的processors加工后后返回给对应的批量操作或索引操作之后,这些信息就不会携带返回.
>

## 访问Pipeline中的内容

如果我们只是定义一个Pipeline而不能访问该Pipeline的上下文信息,那么设计Pipeline就显然是多此一举.与logstash不同的时候,logstash环境与Elasticsearch环境是隔离的,两者无法形成上下文环境,而Pipeline则不同,天然的集成在Elasticsearch中,从而很好的与其形成上线文环境,这个环境让Pipeline能够更充分的利用起来,比如与Pipeline能够形成最直接的上线文关系的是文档信息,由此可见,我们可以在Pipeline(应该是processors中)能够直接访问通过pipeline的文档信息,如文档的字段,元数据信息.

下面的例子说明了我们如何在processors中访问这些上下文信息.

### **访问源文档字段**

```json
{
  "set": {
    "field": "my_field"
    "value": 582.1
  }
}


```

这个例子直接引用了通过该Pipeline内的文档字为"my\_field"的字段,并将所有文档该字段的值设置为582.1.

或者我们也可以通过\_source字段来访问源文档的字段,如下

```json
{
  "set": {
    "field": "_source.my_field"
    "value": 582.1
  }
}


```

### **访问文档的元数据字段**

每个文档都会有一些元数据字段信息(metadata filed),比如\_id，\_index,\_type等,我们在processors中也可以直接访问这些信息的,比如下面的例子:

```json
{
  "set": {
    "field": "_id"
    "value": "1"
  }
}


```

直接访问文档的\_id字段,并设置该值为1

### **访问瞬态元字段(ingest metadata field)**

这个地方并没有直接翻译官方名称,因为不好理解,我使用了我自己的理解意图去翻译该名称.其实含义是一致的,官方称之为**ingest metadata field**,翻译过来且为"摄取元数据字段?“(请高人翻译,我实在无法很好的翻译出来),从官方对该词的释义中可以理解出来,这些字段是临时保存的,在这些文档被处理完成之后返回给对应的批量请求和索引请求的时候,这些字段不会一并返回,因此这里称之为"瞬态元字段”，下面是一个如何访问这些字段的例子:

```json
{
  "set": {
    "field": "received"
    "value": "{{_ingest.timestamp}}"
  }
}
```

该例子会在文档中临时加入一个名称为"`received`"的字段,值为瞬态元字段中的timestamp,以`{{\_ingest.timestamp}}`的格式来进行访问,看到{{}}是不是有种很熟悉的感觉,因为只有在涉及上下文的环境中一般才会使用这种表达式.

## Processors类型详解

*   Append Processor 追加处理器
*   Convert Processor 转换处理器
*   Date Processor 日期转换器
*   Date Index Name Processor 日期索引名称处理器
*   Fail Processor 失败处理器
*   Foreach Processor 循环处理器
*   Grok Processor Grok 处理器
*   Gsub Processor
*   Join Processor
*   JSON Processor
*   KV Processor
*   Lowercase Processor
*   Remove Processor
*   Rename Processor
*   Script Processor
*   Split Processor
*   Sort Processor
*   Trim Processor
*   Uppercase Processor
*   Dot Expander Processor

### Append Processor

顾名思义,追加处理器.就是当我们加载原始文档到Elasticsearch的时候,某些字段的描述信息可能需要我们定制性的增加一些额外说明.

使用方法如下:

```json
{
  "append": {
    "field": "field1",
    "value": ["item2", "item3", "item4"]
  }
}
```

比如我们需要从第三方新导入一批商品,因为某种原因这批新商品必须要打上一个"家居"的标签来表示该批商品是"家居分类",当然一个商品也可能不止一个分类,这个时候要求我们在导入这批文档数据的时候,需要在原有的标签字段中新增一个"家居"标签,这个时候我们就通过追加处理器来完成.

下面的例子我们通过Simulate Pipeline API来帮助我们完成

```json
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "新增一个家居标签",
    "processors": [
      {
        "append": {
          "field": "tag",
          "value": [
            "家居"
          ]
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "code":"000001",
        "tag": "衣柜"
      }
    },
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "code":"000002",
        "tag": "沙发"
      }
    },
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "code":"000003"
      }
    }
  ]
}


```

返回结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "code": "000001",
          "tag": [
            "衣柜",
            "家居"
          ]
        },
        "_ingest": {
          "timestamp": "2017-11-24T13:19:55.555Z"
        }
      }
    },
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "code": "000002",
          "tag": [
            "沙发",
            "家居"
          ]
        },
        "_ingest": {
          "timestamp": "2017-11-24T13:19:55.555Z"
        }
      }
    },
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "code": "000003",
          "tag": [
            "家居"
          ]
        },
        "_ingest": {
          "timestamp": "2017-11-24T13:19:55.555Z"
        }
      }
    }
  ]
}


```

我们可以看到,通过该处理器处理之后,新的文档中的tag字段都新增了一个"家居"标签,同时如果原文档没有该字段,则会新建该字段.

#### Convert Processor

该类型的Processor是用于在写入某些文档之前对该文档的字段类型进行转换,比如文档中有个字符串类型的"价格"字段,我们希望将其转换成float类型,则我们可以使用该处理器来完成这项操作,实现如下:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "修改字符串类型为double类型",
    "processors": [
      {
       "convert": {
         "field": "price",
         "type": "float"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "code":"000001",
        "tag": "衣柜",
        "price":"999.8"
      }
    },
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "code":"000002",
        "tag": "沙发",
        "price":"899.8"
      }
    },
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "code":"000003",
        "price":"799.8"
      }
    }
  ]
}


```

测试结果如下：

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "code": "000001",
          "tag": "衣柜",
          "price": 999.8
        },
        "_ingest": {
          "timestamp": "2017-11-26T10:56:47.663Z"
        }
      }
    },
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "code": "000002",
          "tag": "沙发",
          "price": 899.8
        },
        "_ingest": {
          "timestamp": "2017-11-26T10:56:47.664Z"
        }
      }
    },
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "code": "000003",
          "price": 799.8
        },
        "_ingest": {
          "timestamp": "2017-11-26T10:56:47.664Z"
        }
      }
    }
  ]
}


```

可以看到,新的文档,price字段将不再是字符串类型了,而是float类型.

使用Convert Processor需要注意的是,目前官方支持转换的类型有如下几个:integer, float, string, boolean, and auto。

关于更多该处理器的使用说明,请参考: [https://www.elastic.co/guide/en/elasticsearch/reference/current/convert-processor.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/convert-processor.html)

#### Date Processor

日期处理器,顾名思义,就是将原文档中的某个日期字段转换成一个Elasticsearch识别的时间戳字段(一般默认为@timestamp),该时间戳字段将会新增到原文档中.(目前发现该功能比较鸡肋,因为转换后的时间格式只能是ISO 8601时间格式的,并不能转换成其他格式,因此很少用)

下面是一个使用Date Processor的例子:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "转换成指定格式的日期格式字段",
    "processors": [
      {
      "date": {
        "field": "date",
        "formats": ["UNIX"],
        "timezone": "Asia/Shanghai"
      }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "code":"000001",
        "tag": "衣柜",
        "price":"999.8",
        "date":"1511696379"
      }
    }
  ]
}


```

就是将原有的字符串表示的日期格式进行格式转换(最终转换成的是ISO时间格式),结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "date": "1511696379",
          "code": "000001",
          "@timestamp": "2017-11-26T19:39:39.000+08:00",
          "price": "999.8",
          "tag": "衣柜"
        },
        "_ingest": {
          "timestamp": "2017-11-26T12:01:16.787Z"
        }
      }
    }
  ]
}


```

我们可以看到多了一个默认字段[@timestamp](https://my.oschina.net/u/1049939),该字段名称可以通过target\_field字段来指定该字段名称.

为什么说只能生成ISO格式的时间呢？这方面的资料官方没有提供,只能自己去源码中看,这里

```
public void execute(IngestDocument ingestDocument) {
        String value = ingestDocument.getFieldValue(field, String.class);

        DateTime dateTime = null;
        Exception lastException = null;
        for (Function<String, DateTime> dateParser : dateParsers) {
            try {
                dateTime = dateParser.apply(value);
            } catch (Exception e) {
                //try the next parser and keep track of the exceptions
                lastException = ExceptionsHelper.useOrSuppress(lastException, e);
            }
        }

        if (dateTime == null) {
            throw new IllegalArgumentException("unable to parse date [" + value + "]", lastException);
        }

        ingestDocument.setFieldValue(targetField, ISODateTimeFormat.dateTime().print(dateTime));
    }


```

其中最后一行,在设置字段的时候,发现源码中已经写死为ISODateTimeFormat.个人感觉官方应该会自定义pattern.

#### Date Index Name Processor

Date Index Name Processor算得上是一个比较强大的处理了,它的目的是让通过该处理器的文档能够分配到符合指定时间格式的索引中,前提是按照官方提供的说明来进行使用:

我们先创建一个该类型的pipeline,如下:

```
curl -XPUT 'localhost:9200/_ingest/pipeline/monthlyindex?pretty' -H 'Content-Type: application/json' -d'
{
  "description": "monthly date-time index naming",
  "processors" : [
    {
      "date_index_name" : {
        "field" : "date1",
        "index_name_prefix" : "myindex-",
        "date_rounding" : "M"
      }
    }
  ]
}
'


```

我们再创建一个文档,如下(该文档会使用该pipeline)

```
curl -XPUT 'localhost:9200/myindex/type/1?pipeline=monthlyindex&pretty' -H 'Content-Type: application/json' -d'
{
  "date1" : "2016-04-25T12:02:01.789Z"
}
'


```

结果如下:

```
{
  "_index" : "myindex-2016-04-01",
  "_type" : "type",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}


```

以上请求将不会将此 document（文档）放入 myindex 索引，而是放入到 myindex-2016-04-01 索引中.

这就是日期索引名称处理器(Date Index Name Processor)强大的地方,我们写入的文档可以根据其中的某个日期格式的字段来指定该文档将写入哪个索引中,该功能配上template,能够实现很强大的日志收集功能,比如按月按天来将日志写入Elasticsearch.

如果想了解Date Index Name Processor的使用,请参考:[Date Index Name Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-index-name-processor.html)

#### Fail Processor

该处理器比较简单,就是当文档通过该pipeline的时候,一旦出现异常,该pipeline指定的错误信息就会返回给请求者.

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "monthly date-time index naming",
  "processors" : [
    {
     "fail": {
       "message": "an error message"
     }
    }
  ]
  }, 
  "docs": [
    {
      "_index": "myindex",
      "_type": "type",
      "_id": "id",
      "_source": {
        "id":"1"
      }
    }
  ]
}


```

结果如下:

```
{
  "docs": [
    {
      "error": {
        "root_cause": [
          {
            "type": "exception",
            "reason": "java.lang.IllegalArgumentException: org.elasticsearch.ingest.common.FailProcessorException: an error message",
            "header": {
              "processor_type": "fail"
            }
          }
        ],
        "type": "exception",
        "reason": "java.lang.IllegalArgumentException: org.elasticsearch.ingest.common.FailProcessorException: an error message",
        "caused_by": {
          "type": "illegal_argument_exception",
          "reason": "org.elasticsearch.ingest.common.FailProcessorException: an error message",
          "caused_by": {
            "type": "fail_processor_exception",
            "reason": "an error message"
          }
        },
        "header": {
          "processor_type": "fail"
        }
      }
    }
  ]
}


```

#### Foreach Processor

一个Foreach Processor是用来处理一些数组字段,数组内的每个元素都会使用到一个相同的处理器,比如

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "foreach processor",
  "processors" : [
    {
      "foreach": {
        "field": "values",
        "processor": {
          "uppercase": {
            "field": "_ingest._value"
          }
        }
      }
    }
  ]
  }, 
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "id":"1",
        "values":["hello","world","felayman"]
      }
    }
  ]
}


```

上面的例子中,文档中的values字段是一个数组类型的字段,其中每个元素都需要使用Foreach Processor中的一个共同的uppercase Processor来保证该字段中的每个元素都能执行相同的操作

结果如下:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "id": "1",
          "values": [
            "HELLO",
            "WORLD",
            "FELAYMAN"
          ]
        },
        "_ingest": {
          "_value": null,
          "timestamp": "2017-11-27T08:55:17.496Z"
        }
      }
    }
  ]
}


```

关于每个Processor,深入下去都有许多需要说明的,这里没有深入,只是让大家了解其基本使用。

详情请参考: [Foreach Processore](https://www.elastic.co/guide/en/elasticsearch/reference/current/foreach-processor.html)

#### Grok Processor

Grok Processor可以算的上是一个比较实用的处理器了,会经常使用到日志格式切割上,有用过logstash的用户应该都知道Grok的强大.这里并不会涉及到到Grok详细的语法知识.这里我们只是略过的说明Elasticsearch中的ingest node中的pipeline也能提供logstash中的Grok的功能.

如下的使用例子:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "grok processor",
  "processors" : [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"]
      }
    }
  ]
  }, 
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
       "message": "55.3.244.1 GET /index.html 15824 0.043"
      }
    }
  ]
}


```

上面例子中的日志可能是nginx的日志,格式为:

```
55.3.244.1 GET /index.html 15824 0.043


```

对应的Grok语法为:

```
%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}


```

因此该文本格式的日志信息在经过Grok Processor之后,能够解析成一个标准文档的格式,该文档可以使用Elasticsearch提供的检索和聚合功能充分使用到,而原始的文本格式的日志信息则无法做到这一点

返回结果如下:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "duration": "0.043",
          "request": "/index.html",
          "method": "GET",
          "bytes": "15824",
          "client": "55.3.244.1",
          "message": "55.3.244.1 GET /index.html 15824 0.043"
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:07:40.782Z"
        }
      }
    }
  ]
}


```

可以看到,对应的切割点都转换成文档中的一个字段.

关于Grok Processor的详细介绍,请参考:[Grok Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/grok-processor.html)

#### Gsub Processor

Gsub Processor能够解决一些字符串中才特有的问题,比如我想把字符串格式的日期格式如"yyyy-MM-dd HH:mm:ss"转换成"yyyy/MM/dd HH:mm:ss"的格式,我们可以借助于Gsub Processor来解决,而Gsub Processor也正是利用正则来完成这一任务的.

如:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "gsub processor",
  "processors" : [
    {
      "gsub": {
        "field": "message",
        "pattern": "-",
        "replacement": "/"
      }
    }
  ]
  }, 
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
       "message": "2017-11-27 00:00:00"
      }
    }
  ]
}


```

返回结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": "2017/11/27 00:00:00"
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:15:05.502Z"
        }
      }
    }
  ]
}


```

就将message字段中的日期格式修改成我们想要的格式了.

更多关于Gsub Processor的介绍,请参考[Gsub Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/gsub-processor.html)

#### Join Processor

Join Processor能够将原本一个数组类型的字段值,分解成以指定分隔符分割的一个字符串.

如下例子:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "join processor",
    "processors": [
      {
        "join": {
          "field": "message",
          "separator": "、、、、"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message": ["hello","world","felayman"]
      }
    }
  ]
}


```

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": "hello、、、、world、、、、felayman"
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:25:41.260Z"
        }
      }
    }
  ]
}


```

可以看到,原文档中数组格式的的message字段,经过Join Processor处理后,变成了字符串类型的字段"hello、、、、world、、、、felayman".

更多关于Join Processor的内容,请参考:[Join Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/join-processor.html)

#### JSON Processor

JSON Processor也是用来处理字符串类型的字段,可以将那些符合JSON格式(或被JSON串化)的文本,在经过JSON Processor加工之后,解析成对应的JSON格式,如下例子:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "join processor",
    "processors": [
      {
        "json": {
          "field": "message"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message":"{\"foo\": 2000}"
      }
    }
  ]
}


```

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": {
            "foo": 2000
          }
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:30:28.546Z"
        }
      }
    }
  ]
}


```

可以看到,原本是字符串格式的字段message,在处理之后变成了一个标准的JSON格式.

更多关于JSON Processor的内容,请参考:[JSON Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/json-processor.html)

#### KV Processor

KV Processor用来K,V字符串格式的处理器,比如K=V, K:V,K->V等格式的解析.

如下例:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "kv processor",
    "processors": [
      {
        "kv": {
          "field": "message",
          "field_split":" ",
          "value_split": ":"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message":"ip:127.0.0.1"
      }
    }
  ]
}


```

其中字段message的原始值为""ip:127.0.0.1"",我们想将该格式的内容新增一个独立的字段如ip:127.0.0.1这种格式.

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": "ip:107.0.0.1",
          "ip": "107.0.0.1"
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:46:48.715Z"
        }
      }
    }
  ]
}


```

更多关于KV Processor的内容,请参考:[KV Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/kv-processor.html)

#### Lowercase Processor

Lowercase Processor也是一个专用于字符串类型的字段处理器,顾名思义,是将字符串都转换成小写格式.

如下:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "lowercase processor",
    "processors": [
      {
        "lowercase": {
          "field": "message"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message":"HELLO,WORLD,FELAYMAN."
      }
    }
  ]
}


```

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": "hello,world,felayman."
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:49:58.564Z"
        }
      }
    }
  ]
}


```

可以看到原文档中的message字段的值,在使用Lowercase Processor之后,都变成大写的了.

更多关于Lowercase Processor的内容,请参考:[Lowercase Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/kv-processor.html)

#### Remove Processor

Remove Processor是用来处理在写入文档之前,删除原文档中的某些字段值的.

如下:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "remove processor",
    "processors": [
      {
       "remove": {
         "field": "extra_field"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "extra_field":"extra_field",
        "message":" hello,  felayman."
      }
    }
  ]
}


```

在经过Remove Processor处理后,extra\_field字段将不存在了.

结果为下:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": " hello,  felayman."
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:58:46.038Z"
        }
      }
    }
  ]
}


```

#### Rename Processor

Rename Processor是用来处理在文档写入Elasticsearch之前修改某个文档的字段的名称

如下:

```json
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "remove processor",
    "processors": [
      {
       "rename": {
         "field": "old_name",
         "target_field": "new_name"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
       "old_name":"old_name"
      }
    }
  ]
}


```

会发现原文档的字段old\_name被重新命名为new\_name字段

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "new_name": "old_name"
        },
        "_ingest": {
          "timestamp": "2017-11-27T10:00:48.694Z"
        }
      }
    }
  ]
}


```

#### Script Processor

Script Processor算的上是最强大的处理了,因为能充分利用Elasticsearch提供的脚本能力.

这里也不会详细介绍Elasticsearch中的脚本如何使用,有关信息,请参考:[Script](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)

我么看下在Script Processor中使用脚本的例子:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "script processor",
    "processors": [
      {
        "script": {
          "lang": "painless",
          "inline": "ctx.viewCount = (ctx.viewCount) *10"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "viewCount": 100
      }
    }
  ]
}


```

这里我们通过脚本让原文档中的viewCount字段的值扩大十倍,结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "viewCount": 1000
        },
        "_ingest": {
          "timestamp": "2017-11-27T10:07:56.112Z"
        }
      }
    }
  ]
}


```

#### Set Processor

Set Processor作用于两种不同情况:

1.  指定字段存在时,修改指定字段的值
2.  指定字段不存在时,新增该字段并设置该字段的值

例子如下:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "set processor",
    "processors": [
      {
       "set": {
         "field": "category",
         "value": "家居"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "id":"0001"
      }
    }
  ]
}


```

这里我们为每个使用Set Processor的文档新增一个分类"家居"

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "id": "0001",
          "category": "家居"
        },
        "_ingest": {
          "timestamp": "2017-11-27T10:12:05.306Z"
        }
      }
    }
  ]
}


```

#### Split Processor

Split Processor用于将一个以指定分隔分开的字符串转换成一个数组类型的字段

如下:

```json
 POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "Split processor",
    "processors": [
      {
       "split": {
         "field": "message",
         "separator": "-"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message":"hello-world"
      }
    }
  ]
}


```

其中原文档的message字段值将会有"hello-world"转换为\[“hello”,“world”\]

结果为：

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": [
            "hell",
            "world"
          ]
        },
        "_ingest": {
          "timestamp": "2017-11-27T10:13:35.982Z"
        }
      }
    }
  ]
}


```

#### Sort Processor

Sort Processor用于处理数组类型的字段,可以将存储在原文档中某个数组类型的字段中的元素按照升序或降序来对原元素进行排序

如下:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "sort processor",
    "processors": [
      {
       "sort": {
         "field": "category",
         "order": "asc"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "category":[2,3,4,1]
      }
    }
  ]
}


```

我们使用升序来修改原文档category字段的值的存储排序

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "category": [
            1,
            2,
            3,
            4
          ]
        },
        "_ingest": {
          "timestamp": "2017-11-27T10:16:16.409Z"
        }
      }
    }
  ]
}


```

### Trim Processor

哎,Elasticsearch开发者真的没谁了,基本上能把String类中的方法都拿过来搬一套过来使用.

Trim Processor是专门用于处理字符串两端的空格问题,如下

```json
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "trim processor",
    "processors": [
      {
       "trim": {
         "field": "message"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message":" hello,  felayman."
      }
    }
  ]
}


```

请注意,是去除字符串两端的空格.

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": "hello,  felayman."
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:56:17.946Z"
        }
      }
    }
  ]
}


```

#### Uppercase Processor

该处理器类似于Lowercase Processor,将字符串文本统一转换成大写.

如下:

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "uppercase processor",
    "processors": [
      {
       "uppercase": {
         "field": "message"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message":"hello,world,felayman."
      }
    }
  ]
}


```

结果为:

```
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
          "message": "HELLO,WORLD,FELAYMAN."
        },
        "_ingest": {
          "timestamp": "2017-11-27T09:53:33.672Z"
        }
      }
    }
  ]
}


```

更多关于Uppercase Processor的内容,请参考:[Uppercase Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/uppercase-processor.html)

#### Dot Expander Processor

## 自定义Processor

如果发现官方提供的Processor不满足我们的加工逻辑怎么办？不用担心,官方提供了很好的插件机制来帮助我们对其进行扩展,想实现一个自定义的Processor需要两个过程:

*   实现IngestPlugin接口,并实现IngestPlugin中的getProcessors方法
*   实现自定义的Processor

下面我们给一个具体的例子,我们将任何传入的一个字段的值都变成大写(最简化模型,不指定字段,使用\_ingest/pipeline/\_simulate api 模拟该操作)

新建一个项目,名称为elasticsearch-help,新建FirstUpperPlugin类,如下:

```java
package org.elasticsearch.help;
import org.elasticsearch.help.processor.FirstUpperProcessor;
import org.elasticsearch.ingest.Processor;
import org.elasticsearch.plugins.IngestPlugin;
import org.elasticsearch.plugins.Plugin;
import java.util.Collections;
import java.util.Map;

/**
 * @auhthor lei.fang@shijue.me
 * @since . 2017-11-26
 */
public class FirstUpperPlugin extends Plugin implements IngestPlugin{

    @Override
    public Map<String, Processor.Factory> getProcessors(Processor.Parameters parameters) {
        return Collections.singletonMap(FirstUpperProcessor.TYPE,new FirstUpperProcessor.Factory());
    }
}



```

其中我们实现了getProcessors方法,该方法主要用来提供我们自定义的额Processor,其中FirstUpperProcessor源码如下:

```java
package org.elasticsearch.help.processor;
import org.elasticsearch.ingest.AbstractProcessor;
import org.elasticsearch.ingest.ConfigurationUtils;
import org.elasticsearch.ingest.IngestDocument;
import org.elasticsearch.ingest.Processor;
import java.util.Map;

/**
 * @auhthor lei.fang@shijue.me
 * @since . 2017-11-26
 */
public class FirstUpperProcessor extends AbstractProcessor {

    public static final String TYPE = "firstUpper";

    private final String field;

    public FirstUpperProcessor(String tag,String field) {
        super(tag);
        this.field = field;
    }

    @Override
    public void execute(IngestDocument ingestDocument) throws Exception {
        ingestDocument.setFieldValue(field,field.toUpperCase());
    }

    @Override
    public String getType() {
        return TYPE;
    }

    public static final class Factory implements Processor.Factory {

        public FirstUpperProcessor create(Map<String, Processor.Factory> registry, String processorTag,
                                    Map<String, Object> config) throws Exception {
            String field = ConfigurationUtils.readStringProperty(TYPE, processorTag, config, "field");
            return new FirstUpperProcessor(processorTag, field);
        }
    }

}



```

可以看到,其中核心的方法为execute,主要是将传入的字段的值转换成大写,这里只是调用了简单的toUpperCase()方法.

我们使用\_ingest/pipeline/\_simulate api 来进行测试:

```java
curl -XGET 'http://localhost:9200/_ingest/pipeline/_simulate?pretty ' -d '{
  "pipeline": {
    "description": "字符串首字母转换成大写",
    "processors": [
      {
       "firstUpper":{
         "field":"message"
       }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "message":"hello,world."
      }
    }
  ]
}'


```

结果为:

```java
{
  "docs": [
    {
      "doc": {
        "_index": "index",
        "_type": "type",
        "_id": "id",
        "_source": {
            "message":"Hello,world."
        },
        "_ingest": {
          "timestamp": "2017-11-27T10:16:16.409Z"
        }
      }
    }
  ]
}


```

可以看到,自定义的Ingest node 插件成功了.










