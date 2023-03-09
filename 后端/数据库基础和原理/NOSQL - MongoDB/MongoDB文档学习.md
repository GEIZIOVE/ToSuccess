# 1.MongoDB简介

## 1.1简介

### 1.1.1文档数据库

​		MongoDB中的记录是一个文档，它是由字段和值对组成的数据结构。MongoDB文档类似于JSON对象。字段的值可以包括其他文档，数组和文档数组。

![image-20230213142505597](E:/Development/Typora/images/image-20230213142505597.png)

### 1.1.2 集合/视图/按需实例化视图

> MongoDB将文档存储在[集合中](https://docs.mongodb.com/v4.2/core/databases-and-collections/#collections)。集合类似于关系数据库中的表。

除集合外，MongoDB还支持：

- 只读视图（从MongoDB 3.4开始）
- 按需实例化视图（从MongoDB 4.2开始



### 1.1.3主要特性

#### 高性能

MongoDB提供高性能的数据持久化。特别是，

- 对嵌入式数据模型的支持减少了数据库系统上的I / O操作。
- 索引支持更快的查询，并且可以包含来自嵌入式文档和数组的键。

#### 丰富的查询语言

MongoDB支持丰富的查询语言以支持[读写操作（CRUD）](https://docs.mongodb.com/v4.2/crud/)以及：

- [数据聚合](https://docs.mongodb.com/v4.2/core/aggregation-pipeline/)
- [文本搜索](https://docs.mongodb.com/v4.2/text-search/)和[地理空间查询](https://docs.mongodb.com/v4.2/tutorial/geospatial-tutorial/)。

也可以看看

- [SQL到MongoDB的映射图](https://docs.mongodb.com/v4.2/reference/sql-comparison/)
- [SQL到聚合的映射图](https://docs.mongodb.com/v4.2/reference/sql-aggregation-comparison/)

#### 水平拓展

MongoDB提供水平可伸缩性作为其_核心_ 功能的一部分：

- [分片](https://docs.mongodb.com/v4.2/sharding/#sharding-introduction)将数据分布在一个集群的机器上。
- 从3.4开始，MongoDB支持基于[分片键](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard-key)创建数据[区域](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding)。在平衡群集中，MongoDB仅将区域覆盖的读写定向到区域内的那些分片。有关 更多信息，请参见[区域](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding)章节。

#### 支持多种存储引擎

MongoDB支持[多个存储引擎](https://docs.mongodb.com/v4.2/core/storage-engines/)：

- [WiredTiger存储引擎](https://docs.mongodb.com/v4.2/core/wiredtiger/)（包括对[静态](https://docs.mongodb.com/v4.2/core/wiredtiger/)[加密的](https://docs.mongodb.com/v4.2/core/security-encryption-at-rest/)支持 ）
- [内存存储引擎](https://docs.mongodb.com/v4.2/core/inmemory/)

另外，MongoDB提供可插拔的存储引擎API，允许第三方为MongoDB开发存储引擎。



## 1.2数据库和集合

MongoDB将[BSON文档](https://docs.mongodb.com/v4.2/core/document/#bson-document-format)（即数据记录）存储在[集合中](https://docs.mongodb.com/v4.2/reference/glossary/#term-collection)；数据库中的集合。

![image-20230213143153990](E:/Development/Typora/images/image-20230213143153990.png)



### 1.2.1数据库

在MongoDB中，文档集合存在数据库中。

要选择使用的数据库，请在[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)shell程序中发出 `use <db>` 语句，如下方示例：

```sql
use myDB
```

![image-20230213143833371](E:/Development/Typora/images/image-20230213143833371.png)

### 1.2.2创建数据库

如果数据库不存在，**则在您第一次为该数据库存储数据时，MongoDB会创建该数据库**。这样，您可以切换到不存在的数据库并在[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)shell中执行以下操作 ：

```sql
use myNewDB
db.myNewCollection1.insertOne( { x: 1 } )
```

该[`insertOne()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.insertOne/#db.collection.insertOne)操作将同时创建数据库`myNewDB`和集合`myNewCollection1`（如果它们尚不存在）。确保数据库名称和集合名称均遵循MongoDB [命名限制](https://docs.mongodb.com/v4.2/reference/limits/#restrictions-on-db-names)。

![image-20230213143916817](E:/Development/Typora/images/image-20230213143916817.png)

### 1.2.3集合

MongoDB将文档存储在集合中。集合类似于关系数据库中的表。

#### 创建集合

​		如果不存在集合，则在您第一次为该集合存储数据时，MongoDB会创建该集合。

```sql
db.myNewCollection2.insertOne( { x: 1 } )
db.myNewCollection3.createIndex( { y: 1 } )
```

如果[`insertOne()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.insertOne/#db.collection.insertOne)和 [`createIndex()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.createIndex/#db.collection.createIndex)操作都还不存在，则会创建它们各自的集合。确保集合名称遵循MongoDB [命名限制](https://docs.mongodb.com/v4.2/reference/limits/#restrictions-on-db-names)。

![image-20230213144017680](E:/Development/Typora/images/image-20230213144017680.png)

#### 显示创建

MongoDB提供了[`db.createCollection()`](https://docs.mongodb.com/v4.2/reference/method/db.createCollection/#db.createCollection)使用各种选项显式创建集合的方法，例如设置最大大小或文档验证规则。如果未指定这些选项，则无需显式创建集合，因为在首次存储集合数据时，MongoDB会创建新集合。

> 要修改这些收集选项，请参见[`collMod`](https://docs.mongodb.com/v4.2/reference/command/collMod/#dbcmd.collMod)。

#### 文档验证

​		默认情况下，集合不要求其文档具有相同的模式。也就是说，单个集合中的文档不需要具有相同的字段集，并且字段的数据类型可以在集合中的不同文档之间有所不同。

​		但是，从MongoDB 3.2开始，您可以在更新和插入操作期间对集合强制执行[文档验证规则](https://docs.mongodb.com/v4.2/core/schema-validation/)。有关详细信息，请参见[模式验证](https://docs.mongodb.com/v4.2/core/schema-validation/)。

#### 修改文档结构

​		 要更改集合中文档的结构，例如添加新字段，删除现有字段或将字段值更改为新类型，请将文档更新为新结构。

## 1.3 视图

> 从 3.4 开始, MongoDB 添加了基于已存在的集合或者 View (视图) 创建只读的 View 支持.



### 创建视图

创建或者定义一个视图

```sql
db.createView(<view>, <source>, <pipeline>, <collation> )
```





xxx 暂时用不上

















































