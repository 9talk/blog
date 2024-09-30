# 最佳实践

## 删除索引，而不是文档

已删除的文档不会立即从 Elasticsearch 的文件系统中删除。

相反，Elasticsearch 会在每个相关分片上将文档标记为已删除。

标记的文档将继续使用资源，直到在定期分段合并期间将其删除。

如果可能，请改为删除整个索引。Elasticsearch 可以立即直接从文件系统中删除已删除的索引并释放资源。

## 将 Data Streams 和 ILM 用于时间序列数据

Data Streams 允许跨多个基于时间的支持索引存储时间序列数据。

可以使用索引生命周期管理 （ILM） 自动管理这些支持索引。

Data Streams的一个优点是自动翻转，当当前写入索引满足定义的 <control>max_primary_shard_size</control>、**max_age**、**max_docs** 或 **max_size** 阈值时，它会创建一个新的**索引**。

当不再需要某个索引时，可以使用 **ILM** 自动删除它并释放资源

## 单 shard 最多2亿文档或10GB~50GB 之间

1. 每个分片都会产生一些相关的开销，包括集群管理和搜索性能。
2. 搜索 1000 个 50MB 分片的成本要比搜索包含相同数据的单个 50GB 分片要大。
3. 非常大的分片也可能导致搜索速度变慢，并且在失败后需要更长的时间才能恢复。
4. shard的物理大小没有硬性限制，理论上每个分片最多可以包含超过 20 亿个文档。
5. 但是，经验表明，每个分片的文档计数保持在 **2 亿**个以下，**10GB** 到 **50GB** 之间的分片通常更多场景。
6. 如果使用 ILM，请将滚动更新操作的 **max_primary_shard_size** 阈值设置为 50GB，以避免分片大于 50GB

<note>每个分片的文档计数保持在2亿个以下，10GB到50GB之间</note>

## 主节点JVM堆内存, 每 3000 个索引应至少有 1GB 的堆

1. 主节点可以管理的索引数量与其堆大小成正比。每个索引所需的确切堆内存量取决于各种因素，例如映射的大小和每个索引的分片数
2. 作为一般经验法则，主节点上每 GB 堆的索引数应少于 3000 个。
3. 如果集群具有专用主节点，每个节点具有 4GB 的堆，则索引应少于 12000 个。
4. 如果您的主节点不是专用主节点，则适用相同的大小调整：对于集群中的每 3000 个索引，您应该在每个符合主节点条件的节点上至少保留 1GB 的堆。

<note>主节点应预留出足够的内存空间来管理索引, 通常是每 3000 个索引应至少有 1GB 的堆, 即使不是专用主节点也应该预留出足够的堆内存</note>

## 添加足够的节点以保持在集群分片限制范围内

默认情况下, 集群限制了每个节点上最多1000个非冻结分片, 以及每个专用冻结节点创建 3000 个冻结分片。确保集群中有足够的每种类型的节点来处理所需的分片数量。

所有开放索引的主分片和副本分片都计入限制，包括未分配的分片。例如，具有 5 个主分片和 2 个副本的开放索引计为 15 个分片。已关闭的索引不会计入分片计数。

查看详情: https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#cluster-shard-limit

## 为字段映射和开销提供足够的堆

映射字段在每个节点上消耗一些堆内存，并且需要在数据节点上使用额外的堆。确保每个节点都有足够的堆用于映射，并为与其工作负载关联的开销留出额外的空间。以下部分说明如何确定这些堆要求。

### 在集群状态中映射元数据

集群中的每个节点都有[集群状态](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-state.html#cluster-state-api-desc "Description")的副本。

cluster state （集群状态） 包括有关每个索引的字段映射的信息。此信息具有堆开销。

您可以使用[集群统计 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html "Cluster stats API") 来获取去重和压缩后所有映射总大小的堆开销。

```Console
GET _cluster/stats?human&filter_path=indices.mappings.total_deduplicated_mapping_size*
```

这将向您显示如下例输出中的信息：

```JSON
{
  "indices": {
    "mappings": {
      "total_deduplicated_mapping_size": "1gb",
      "total_deduplicated_mapping_size_in_bytes": 1073741824
    }
  }
}
```

### 检索堆大小和字段映射器开销

您可以使用 [Nodes stats API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-stats.html "Nodes stats API") 获取每个节点的两个相关指标：
1. 每个节点上的堆大小。
2. 每个节点的字段的任何其他估计堆开销。这是特定于数据节点的，其中除了上面提到的 cluster state 字段信息外，数据节点持有的索引的每个 mapped 字段都有额外的堆开销。对于不是数据节点的节点，此字段可能为零。

```Console
GET _nodes/stats?human&filter_path=nodes.*.name,nodes.*.indices.mappings.total_estimated_overhead*,nodes.*.jvm.mem.heap_max*
```


### 考虑额外的堆开销

除了上述两个字段开销指标外，您还必须为 Elasticsearch 的基准使用情况以及您的工作负载（如索引、搜索和聚合）留出足够的堆。

0.5GB 的额外堆足以处理许多合理的工作负载，如果您的工作负载非常轻，则可能需要更少，而繁重的工作负载可能需要更多。

### Example

例如，考虑上述数据节点的输出。节点的堆至少需要：
* 1 GB 用于集群状态字段信息。
* 1 GB 用于数据节点字段的额外估计堆开销。
* 0.5 GB 的额外堆用于其他开销。

由于示例中节点的最大堆大小为 4GB，因此它足以满足所需的 2.5GB 总堆。

如果节点的堆最大大小不足，请考虑[避免不必要的字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html#avoid-unnecessary-fields "Avoid unnecessary mapped fields")，或纵向扩展集群，或重新分配索引分片。

请注意，上述规则不一定保证涉及大量索引的搜索或索引的性能。您还必须确保您的数据节点有足够的资源来支持您的工作负载，并且您的整体分片策略满足您的所有性能要求。另请参阅 [搜索在每个分片的单个线程上运行](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html#single-thread-per-shard "Searches run on a single thread per shard") 和 [每个索引、分片、分段和字段都有开销](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html#each-shard-has-overhead "Each index, shard, segment and field has overhead")。

## 避免节点热点

如果为特定节点分配的分片过多，则该节点可能会成为热点。例如，如果单个节点包含的分片过多，而索引量较高，则该节点可能会出现问题。

要防止出现热点，请使用 [`index.routing.allocation.total_shards_per_node`](https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-total-shards.html#total-shards-per-node) index 设置显式限制单个节点上的分片数。您可以使用[更新索引设置 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html "Update index settings API") 进行配置 `index.routing.allocation.total_shards_per_node` 。。

```Console
PUT my-index-000001/_settings
{
  "index" : {
    "routing.allocation.total_shards_per_node" : 5
  }
}
```

## 避免不必要的映射字段

默认情况下，Elasticsearch 会自动为其索引的每个文档中的每个字段[创建一个映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-mapping.html "Dynamic mapping")。每个映射的字段都对应于磁盘上的一些数据结构，这些数据结构是在此字段上进行高效搜索、检索和聚合所必需的。有关每个映射字段的详细信息也保存在内存中。在许多情况下，此开销是不必要的，因为字段未用于任何搜索或聚合。使用 [_Explicit mapping 而不是 dynamic_](https://www.elastic.co/guide/en/elasticsearch/reference/current/explicit-mapping.html "Explicit mapping") mapping 以避免创建从未使用的字段。如果字段集合通常一起使用，请考虑在索引时使用 [`copy_to`](https://www.elastic.co/guide/en/elasticsearch/reference/current/copy-to.html "copy_to") 来合并它们。如果某个字段很少使用，则最好将其设为 [Runtime 字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/runtime.html "Runtime fields")。

您可以通过 [Field usage stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/field-usage-stats.html "Field usage stats API") API 获取有关正在使用哪些字段的信息，并且可以使用 [Analyze index disk usage](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-disk-usage.html "Analyze index disk usage API") API 分析映射字段的磁盘使用情况。但请注意，不必要的映射字段也会带来一些内存开销及其磁盘使用率。