# 排查分片相关错误

## this action would add [x] total shards, but this cluster currently has [y]/[z] maximum shards open;

此操作将添加 [x] 个分片，但此集群当前打开的最大分片数为 [y]/[z];

[`cluster.max_shards_per_node`](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#cluster-max-shards-per-node) 集群设置限制集群的最大打开分片数。此错误表示操作将超过此限制。

如果您确信您的更改不会破坏集群的稳定性，则可以使用[集群更新设置 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html "Cluster update settings API") 临时增加限制，然后重试该操作。

```Console
PUT _cluster/settings
{
  "persistent" : {
    "cluster.max_shards_per_node": 1200
  }
}
```

这种增加应该只是暂时的。作为长期解决方案，我们建议您将节点添加到过度分片数据层或[减少集群的分片计数](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html#reduce-cluster-shard-count "Reduce a cluster’s shard count")。

要在进行更改后获取集群的当前分片计数，请使用[集群统计 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html "Cluster stats API")。

```Console
GET _cluster/stats?filter_path=indices.shards.total
```

当长期解决方案到位时，我们建议您重置 `cluster.max_shards_per_node` 限制

```Console
PUT _cluster/settings
{
  "persistent" : {
    "cluster.max_shards_per_node": null
  }
}
```


## Number of documents in the shard cannot exceed [2147483519]

分片中的文档数不能超过 [2147483519]

每个 Elasticsearch 分片都是一个单独的 Lucene 索引，因此它与 Lucene 的 [`MAX_DOC` 限制](https://github.com/apache/lucene/issues/5176)相同，即最多包含 2,147,483,519 （`（2^31）-129`） 个文档。此每个分片的限制适用于[索引统计信息 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-stats.html "Index stats API") 报告的 `docs.count` 加上 `docs.deleted` 的总和。超过此限制将导致如下错误：

```text
Elasticsearch exception [type=illegal_argument_exception, reason=Number of documents in the shard cannot exceed [2147483519]]
```

> 此计算可能与 [Count API 的计算](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-count.html "Count API")不同，因为 Count API 不包括嵌套文档，也不对已删除的文档进行计数。

此限制远高于[建议的最大文档计数](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html#shard-size-recommendation "Aim for shards of up to 200M documents, or with sizes between 10GB and 50GB")（每个分片约 200M 个文档）。

如果你遇到这个问题，可以尝试使用 [强制合并 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html "Force merge API") 来合并一些已删除的文档来缓解这个问题。例如：

这将启动一个异步任务，该任务可以通过 [Task Management API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html "Task management API") 进行监控。

[删除不需要的文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html "Delete by query API")，或者将索引[拆分](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-split-index.html "Split index API")或[重新索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html "Reindex API")为具有更多分片的索引也可能有所帮助。