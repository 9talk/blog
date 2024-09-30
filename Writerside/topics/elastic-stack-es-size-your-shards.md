# 调整分片大小

原文地址: https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html

Elasticsearch 中的每个索引都分为一个或多个分片，每个分片都可以跨多个节点复制，以防止硬件故障。

如果使用的是 Data Streams，则每个 Data Stream 都由一系列索引提供支持。

可以在单个节点上存储的数据量是有限制的，因此可以通过添加节点并增加要匹配的索引和分片数量来增加集群的容量。

但是，每个索引和分片都有一些开销，如果将数据划分为太多的分片，则开销可能会变得不堪重负。

具有太多索引或分片的集群被称为 oversharding (过度分片)。oversharding 的集群在响应搜索时效率会降低，在极端情况下，它甚至可能变得不稳定。

## 创建分片策略

防止 oversharding 和其他分片相关问题的最佳方法是创建分片策略。

分片策略可帮助我们确定和维护集群的最佳分片数量，同时限制这些分片的大小。

<warning>没有万能的分片策略。在一个环境中有效的策略可能无法在另一个环境中扩展。一个好的分片策略必须考虑基础设施、使用案例和性能预期。</warning>

<compare>
<code-block collapsible="false">
The best way to create a 
sharding strategy is to 
benchmark your production data 
on production hardware using 
the same queries and indexing
loads you’d see in production.
For our recommended methodology,
watch the quantitative cluster
sizing video. 
As you testdifferent shard 
configurations,
use Kibana’s Elasticsearch 
monitoring tools to track your
cluster’s stability 
and performance.
</code-block>
<code-block>
创建分片策略的最佳方法是使用您在生产中看到
的相同查询和索引负载，在生产硬件上对生产
数据进行基准测试。
有关我们推荐的方法，请观看定量集群大小
调整视频。
在测试不同的分片配置时，请使用 Kibana 的 
Elasticsearch 监控工具来跟踪集群的
稳定性和性能。
</code-block>
</compare>

Elasticsearch 节点的性能通常受到底层存储性能的限制。 查看我们的建议，以优化您的存储以进行索引和搜索。
