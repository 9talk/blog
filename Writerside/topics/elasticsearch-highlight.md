# elasticsearch high

1. 高亮只有在分词字段有意义
2. 高亮可以用于子字段 (fields字段), 用在子字段时, highlight结构体中设置的高亮字段也应该是子字段
3. 高亮字段建议使用store单独设置存储, 使用 `"store": true`, 如未单独存储, 会使用`_source`的值进行高亮
4. 高亮 `nested` 类型的子字段时, 只会从`_source`中取值, 即使子字段设置`"store": true`, 也无效

## 设置索引

```
PUT article
PUT article/_mapping
{
  "properties": {
    "article_title": {
      "type": "keyword",
      "fields": {
        "text": {
          "type": "text"
        }
      }
    } 
  }
}
```

## 索引数据

```

POST _bulk
{"index":{"_index":"article","_id":0}}
{"article_type": "原创","article_title": "Elasticsearch查询——Profile API（性能分析）","content": "Profile API 性能分析 平时开发的过程中我们可能需要对一些查询操作进行优化，而优化之前的工作就是要对操作的性能进行分析，而ES提供了Profile API来帮助用户进行性能分析。它让用户了解如何在较低的级别执行搜索请求，这样用户就可以理解为什么某些请求比较慢，并采取措施改进它们。","date": "2019-12-22 22:22:29","read_num": 4,"comment_num": 0}
{"index":{"_index":"article","_id":1}}
{"article_type": "原创","article_title": "Elasticsearch查询——Search Template（模板查询）","content": "关于版本 内容 版本 Elasticsearch版本 7.2.0 ES模板搜索——Search Template 日常开发中我们可能需要频繁的使用一些类似的查询。为了减少重复的工作，我们可以将这些类似的查询写成模板，在后续只需要传递不同的参数来调用模板获取结果。 模板的保存和删","date": "2019-12-19 23:48:14","read_num": 2,"comment_num": 0}
{"index":{"_index":"article","_id":2}}
{"article_type": "原创","article_title": "Elasticsearch查询——URI Search（简单查询字符串）","content": "简单查询字符串——URI Search 实际业务中我们通常都是使用一个完整的请求体从elasticsearch获得数据。然而在有些时候我们为了调试系统或者数据的时候需要临时进行一些简单的查询时候，编写完整的请求体JSON可能会稍微麻烦。而elasticsearch提供了一种基于URI的简单的查询方","date": "2019-12-18 21:58:18","read_num": 10,"comment_num": 0}
{"index":{"_index":"article","_id":3}}
{"article_type": "原创","article_title": "Elasticsearch摄取节点（十）——GeoIP以及Grok处理器","content": "IP解析处理器(GeoIP Processor) 处理器作用 GeoIP Processor主要是根据IP地址解析出具体的地理位置信息的处理器。此处理器需要配合Maxmind数据库中的数据。 ingest-geoip模块附带的GeoLite2 City、GeoLite2 Country和GeoLi","date": "2019-12-18 20:52:10","read_num": 4,"comment_num": 0}
{"index":{"_index":"article","_id":4}}
{"article_type": "原创","article_title": "Elasticsearch脚本使用","content": "如何使用Elasticsearch脚本 Elasticsearch默认脚本语言为Painless。其他lang插件使您可以运行以其他语言编写的脚本。 语言 沙盒 是否需要插件 备注 painless 是 内置的 默认脚本语言 expression 是 内置的 快速的自定义排名和","date": "2019-12-14 15:10:07","read_num": 11,"comment_num": 0}

```

## 高亮查询语句

```
GET article/_search
{
  "size": 5,
  "query": {
    "match": {
      "article_title.text": "URI Search"
    }
  },
  "highlight": {
    "boundary_scanner_locale": "zh_CN",
    "require_field_match": "true",
    "fields": {
      "article_title.text": {
        "pre_tags": [
          "<em>"
        ],
        "post_tags": [
          "</em>"
        ]
      }
    }
  }
}
```

## 设置索引 (使用字段设置的store进行高亮)

```
DELETE article
PUT article
PUT article/_mapping
{
  "_source": {
    "enabled": false
  },
  "properties": {
    "article_title": {
      "type": "keyword",
      "fields": {
        "text": {
          "type": "text",
          "store": true
        }
      }
    }
  }
}
```

## 高亮查询语句

```
GET article/_search
{
  "size": 5,
  "query": {
    "match": {
      "article_title.text": "URI Search"
    }
  },
  "highlight": {
    "boundary_scanner_locale": "zh_CN",
    "require_field_match": "true",
    "fields": {
      "article_title.text": {
        "pre_tags": [
          "<em>"
        ],
        "post_tags": [
          "</em>"
        ]
      }
    }
  }
}
```

## 设置索引(测试 nested 字段)

```
DELETE article
 
PUT article
{"mappings":{"_source":{"includes":["article"],"excludes":[]},"properties":{"article":{"type":"nested","properties":{"text":{"type":"text"}}},"context":{"type":"nested","properties":{"text":{"type":"text","store":true}}}}}}
```

索引一篇文档

```
PUT article/_doc/1
{"article":[{"text":["1Elasticsearch查询——Profile API（性能分析）","3Elasticsearch查询——Profile API（性能分析）"]},{"text":["2Elasticsearch查询——Profile API（性能分析）","4Elasticsearch查询——Profile API（性能分析）"]}],"context":[{"text":["1Elasticsearch查询——Profile API（性能分析）","3Elasticsearch查询——Profile API（性能分析）"]},{"text":["2Elasticsearch查询——Profile API（性能分析）","4Elasticsearch查询——Profile API（性能分析）"]}]}
```

测试高亮
```
GET article/_search
{"size":5,"query":{"bool":{"should":[{"nested":{"path":"article","query":{"match":{"article.text":"1Elasticsearch"}}}},{"nested":{"path":"context","query":{"match":{"context.text":"1Elasticsearch"}}}}],"minimum_should_match":1}},"highlight":{"boundary_scanner_locale":"zh_CN","require_field_match":"true","number_of_fragments":-1,"fields":{"context.text":{"pre_tags":["<em>"],"post_tags":["</em>"]},"article.text":{"pre_tags":["<em>"],"post_tags":["</em>"]}}}}
```