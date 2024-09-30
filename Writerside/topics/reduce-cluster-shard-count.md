# 减少集群的分片计数

如果您的集群已经过度分片，您可以使用以下一种或多种方法来减少其分片计数。

### 创建涵盖较长时间段的索引

如果您使用 ILM 并且您的保留策略允许，请避免对滚动更新操作使用 `max_age` 阈值。相反，请使用 `max_primary_shard_size` 以避免创建空索引或许多小分片。

如果您的保留策略需要 `max_age` 阈值，请增加该阈值以创建涵盖较长时间间隔的索引。例如，您可以每周或每月创建索引，而不是创建每日索引。

### 删除空索引或不需要的索引

如果您使用 ILM 并根据 max_age 阈值滚动索引，则可能会无意中创建没有文档的索引。这些空索引没有任何好处，但仍会消耗资源。

您可以使用 [cat count API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-count.html) 找到这些空索引。

```Console
GET _cat/count/my-index-000001?v=true
```

获得空索引列表后，您可以使用 [delete index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-delete-index.html) 删除它们。您还可以删除任何其他不需要的索引。

```Console
DELETE my-index-000001
```

### 在非高峰时段强制合并

如果不再写入索引，则可以使用[force merge API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html) 将较小的分段 [merge](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-merge.html) 为较大的分段。这可以减少分片开销并提高搜索速度。但是，强制合并会占用大量资源。如果可能，请在非高峰时段运行强制合并。

```Console
POST my-index-000001/_forcemerge
```

### 将现有索引缩减为更少的分片

如果您不再写入索引，则可以使用[shrink index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-shrink-index.html)来减少其分片计数。

ILM 还对处于热阶段的索引执行[shrink action](https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-shrink.html)

### 合并较小的索引

还可以使用 [reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) 将具有类似映射的索引合并到单个大型索引中。

对于时间序列数据，您可以将短时间段的索引重新索引为涵盖较长时间段的新索引。

例如，您可以使用共享索引模式（如 my-index-2099.10.11）将 10 月的每日索引重新索引为每月 my-index-2099.10 索引。重新编制索引后，删除较小的索引。