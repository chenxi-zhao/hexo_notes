### Match All Query
最简单的查询，所有文档的评分_score都是1

```json
{
    "match_all": {
        "boost": 1.2 //The _score can be changed with the boost parameter:
    }
    // "match_none": {}
}
```

### Full Text Query
```json
{
    "match" : {
        "message" : "this is a test"
    }
}
```

#### Match Query
三种`match`查询: boolean, phrase, and phrase_prefix

1. boolean

```json
{
    "match" : {
        "message" : {
            "query" : "this is a test",
            "operator" : "and", // 可以设置成or或者and，默认or
            "analyzer" : "IK", // 指定分词器，未指定使用默认mapping中的
            "lenient" : true, // 是否忽略查询过程中的类型不匹配，比如用一个String的文本查数字，默认false
            "fuzziness": 3, // 分词过后的模糊匹配，值类型多样 https://www.elastic.co/guide/en/elasticsearch/reference/2.1/common-options.html#fuzziness
            "prefix_length" : 4, //不能被 “模糊化” 的初始字符数。
            "max_expansions" : 20, // 用来限制将产生的模糊选项的总数量
            "zero_terms_query": "all", // 收none(默认)和all
            "cutoff_frequency" : 0.001 //
        }
    }
}
```