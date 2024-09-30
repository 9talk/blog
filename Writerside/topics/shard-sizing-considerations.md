# 调整大小时需要注意的事项

在构建分片策略时，请记住以下事项。

## 每个分片在单个 CPU 线程上运行搜索

大多数 search 请求都会找到多个分片。

每个分片在单个 CPU 线程上运行搜索。

虽然一个分片可以运行多个并发搜索，但跨大量分片进行search可能会耗尽节点的搜索线程池。

这可能会导致吞吐量低和搜索速度慢。

线程池说明: https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html

## 每个索引、分片、分段和字段都有开销

1. 每个index和每个shard都需要一些内存和 CPU 资源。
2. 在大多数情况下，一小部分大shard使用的资源比许多小shard少。

<note>尽量使用符合规格的大shard, 而不是过度分片</note>

1. shard 由多个 Segment 组成, Segment 存储其索引数据, Elasticsearch 将一些 segment 的元数据保存在堆内存中，以便可以快速检索以进行搜索。
2. 随着分片的增长，其segment会合并为更少、更大的segment。这减少了 Segment 的数量，这意味着保存在堆内存中的元数据更少。

<note>定期合并segment, 但是需要注意单个segment不要大于5GiB!, 有关信息: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html#forcemerge-api-desc</note>

1. 每个映射的字段在内存使用和磁盘空间方面也会带来一些开销。
2. 默认情况下，Elasticsearch 会自动为其索引的每个文档中的每个字段创建一个映射，但可以关闭此行为以控制映射。
3. 此外，每个 segment 都需要每个 mapped 字段的少量堆内存。此每个字段每个分段的堆开销包括字段名称的副本，使用 ISO-8859-1 （如果适用）或 UTF-16 编码。通常这并不明显，但如果您的分片具有较高的分段计数，并且相应的映射包含较高的字段计数和/或非常长的字段名称，则可能需要考虑此开销。

<note>尽量使用 <control>显示映射</control> , 如果遇到没有映射的字段, 增加映射后更新即可, 映射配置不能修改但能新增</note>

集群的节点有数据层这个概念, 每个存放数据的节点都应该存在数据层这个配置, 在每个层中，Elasticsearch 会尝试将索引的分片分布到尽可能多的节点。

当您添加新节点或节点失败时，Elasticsearch 会自动在该层的其余节点之间重新平衡索引的分片。