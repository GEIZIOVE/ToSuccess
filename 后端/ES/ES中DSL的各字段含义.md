# ES中DSL的各字段含义

```json
{
	"size": 10000,
	"query": {
		"nested": {
			"query": {
				"bool": {
					"must": [{
						"match": {
							"pageContents.content": {
								"query": "Java",
								"operator": "OR",
								"prefix_length": 0,
								"max_expansions": 50,
								"fuzzy_transpositions": true,
								"lenient": false,
								"zero_terms_query": "NONE",
								"auto_generate_synonyms_phrase_query": true,
								"boost": 1.0
							}
						}
					}],
					"adjust_pure_negative": true,
					"boost": 1.0
				}
			},
			"path": "pageContents",
			"ignore_unmapped": false,
			"score_mode": "avg",
			"boost": 1.0,
			"inner_hits": {
				"ignore_unmapped": false,
				"from": 0,
				"size": 1,
				"version": false,
				"seq_no_primary_term": false,
				"explain": false,
				"track_scores": false
			}
		}
	},
	"_source": {
		"includes": [],
		"excludes": ["pageContents"]
	},
	"track_total_hits": 2147483647
}
```

## track_total_hits

```json
	"track_total_hits": 2147483647
```

在Elasticsearch中，查询可以返回数百万甚至数十亿的文档。在这种情况下，`track_total_hits`可以控制匹配的文档总数是否被计算和返回。如果结果集非常大，而又需要知道总文档数，就会非常有用。如果您没有设置，Elasticsearch会默认返回前10,000个匹配的文档。如果您希望获取所有匹配的文档，则可以将track_total_hits设置为true，但这样会增加性能开销。如果您不需要知道匹配的文档总数，则将track_total_hits设置为false，这样就只返回匹配文档的实际数量，而不会计算匹配文档的总数。如果您想获取所有匹配的文档，则可以将track_total_hits设置为2147483647，这个数字是一个特殊值，表示获取所有匹配的文档。

**1.  ES 查询 hits 统计总数不准？**

当我们使用 ES 的时候，有时会比较关心匹配到的文档总数是多少，所以在查询得到结果后会使用 hits.total.value 这个值作为匹配的总数，如下

![1062096-20220306150421889-46080103.png](E:\Development\Typora\images\1062096-20220306150421889-46080103.png)

说明：这是因为，es官方默认限制索引查询最多只能查询10000条数据 

**2.track_total_hits**

平常数据量不大的情况下,这个数值没问题。但是当超出 10000 个数据量的时候，这个 value 将不会增长了，固定为 10000。这个时候很显然数量统计就不准了。

ES 为我们准备了这样一个参数来开启精确匹配`track_total_hits`

这个时候返回值将是精确的：

![1062096-20220306150549028-1521043984.png](E:\Development\Typora\images\1062096-20220306150549028-1521043984.png)

**3.track_total_hits的使用场景**

如果你的业务需求在超过 10000 这个阈值之后就不需要精准的计算的话，就不需要设置该值，毕竟匹配大量的文档是一个成本较高的操作，同样的如果你并不需要统计数量，那么将该值设置为 false `"track_total_hits": false` 也是一种优化检索效率的操作。

**4.hits.total.relation**

通过图一和图二，可以发现设置 track_total_hits 为 true 的时候返回 relation 的值是不一样的，gte 表示 hits.total.value 是查询匹配总命中数的上限。 eq 则表示hits.total.value 是准确计数。

因此，我们可以通过设置 track_total_hits 为整数，来自定义上限如：

![1062096-20220306150742825-894041677.png](E:\Development\Typora\images\1062096-20220306150742825-894041677.png)

## inner_hits

```json
"inner_hits": {
    "ignore_unmapped": false,
    "from": 0,
    "size": 1,
    "version": false,
    "seq_no_primary_term": false,
    "explain": false,
    "track_scores": false
}
```

以下是**`inner_hits`**参数的含义和用法：

- **`ignore_unmapped`**：这个参数是用来判断是否忽略掉那些未被映射的字段。如果设置为**`false`**，在嵌套查询时如果内嵌的字段没有被映射，则会抛出异常。如果设置为**`true`**，则会忽略那些未被映射的字段。
- **`from`**和**`size`**：这两个参数用来控制返回的内部文档的数量，和从哪个位置开始返回。例如，如果**`from`**设置为2，**`size`**设置为5，则会从匹配的内部文档的第三项开始返回，最多返回5个内部文档的结果。
- **`version`**：这个参数用来控制是否返回每个匹配的内部文档的版本号。如果设置为**`true`**，则每个匹配的内部文档的版本号将会被返回。
- **`seq_no_primary_term`**：这个参数用来控制是否返回每个匹配的内部文档的序列号和主要版本号。如果设置为**`true`**，则每个匹配的内部文档的序列号和主要版本号将会被返回。
- **`explain`**：这个参数用来控制是否返回匹配内部文档的解释性说明。如果设置为**`true`**，则对于每个匹配的内部文档，将返回一个解释性说明。
- **`track_scores`**：这个参数用来控制是否返回每个匹配的内部文档的分数。如果设置为**`true`**，则每个匹配的内部文档的分数将会被返回。

内部文档的匹配结果将被嵌套到父文档的**`inner_hits`**字段中，可以通过**`inner_hits`**字段进行访问。例如，可以使用以下方式来访问内部文档匹配的第一项：**`hits['inner_hits']['<inner_hits_name>']['hits']['hits'][0]['_source']`**。其中，**`<inner_hits_name>`**是内部文档匹配查询的名称，在此查询中没有指定名称，因此使用默认名称**`inner_hits`**。

需要注意的是，**`inner_hits`**参数只有在使用嵌套查询时才能使用。



## 嵌套查询pageContents.content参数：

```json
"pageContents.content": {
    "query": "Java",
    "operator": "OR",
    "prefix_length": 0,
    "max_expansions": 50,
    "fuzzy_transpositions": true,
    "lenient": false,
    "zero_terms_query": "NONE",
    "auto_generate_synonyms_phrase_query": true,
    "boost": 1.0
}
```



1. `query`：这个参数是用来设置要搜索的文本内容。在这个例子中，我们要搜索`pageContents.content`字段中包含`Java`的文本。
2. `operator`：这个参数用来设置多个搜索词之间的逻辑运算符。在这个例子中，我们使用的是`OR`，表示只要包含`Java`中的任意一个单词，就会被搜索到。
3. `prefix_length`：这个参数用来设置在搜索时匹配搜索词的前缀长度。默认值为0，表示完全匹配。
4. `max_expansions`：这个参数用来控制模糊搜索时最多扩展多少个搜索词。默认值为50。
5. `fuzzy_transpositions`：这个参数用来控制模糊搜索时是否允许字符转置。如果设置为`true`，则表示允许字符转置，否则不允许。默认值为`true`。
6. `lenient`：这个参数用来控制是否使用宽松模式进行搜索。如果设置为`true`，则会尝试解析错误的查询，否则会抛出异常。默认值为`false`。
7. `zero_terms_query`：这个参数用来控制搜索词为空时的行为。如果设置为`NONE`，则不会匹配任何文档；如果设置为`ALL`，则会匹配所有文档；如果设置为`NOT_ZERO`，则只匹配那些包含非空搜索词的文档。默认值为`NONE`。
8. `auto_generate_synonyms_phrase_query`：这个参数用来控制是否自动生成同义词短语查询。如果设置为`true`，则会尝试自动生成同义词短语查询，否则不会。默认值为`false`。
9. `boost`：这个参数用来控制搜索结果的排序。它可以为搜索词的匹配打分。默认值为1.0。

`pageContents.content`是一个嵌套字段，指的是`pageContents`对象中的`content`字段。在这个例子中，我们使用了一个嵌套查询，来搜索`pageContents.content`字段中包含`Java`的文本。因此，在嵌套查询中，我们使用了上述的参数来设置搜索的行为和匹配规则。



> - **`prefix_length`**
>   - 不能被 “模糊化” 的初始字符数。 大部分的拼写错误发生在词的结尾，而不是词的开始。 例如通过将 `prefix_length` 设置为 `3` ，你可能够显著降低匹配的词项数量。
> - **`max_expansions`**
>   - 如果一个模糊查询扩展了三个或四个模糊选项， 这些新的模糊选项也许是有意义的。如 果它产生 1000 个模糊选项，那么就基本没有意义了。 设置 `max_expansions` 用来限制将产生的模糊选项的总数量。模糊查询将收集匹配词项直到达到 `max_expansions` 的限制。

