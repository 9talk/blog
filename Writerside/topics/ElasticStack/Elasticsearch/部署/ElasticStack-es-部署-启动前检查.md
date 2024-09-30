# 启动前检查

## 端口检查

默认情况下，Elasticsearch绑定到回环地址以进行HTTP端口和传输端口（内部）通信。
通过`http.host`和`transport.host`可以独立配置HTTP端口和传输端口. 默认分别为9200, 9300

## 堆内存检查

默认情况下，Elasticsearch 会根据节点的角色和总内存自动调整JVM堆的大小。
如果手动覆盖默认大小并使用不同的初始堆大小和最大堆大小启动JVM，则在系统使用期间调整堆大小时，JVM可能会暂停。
如果启用`bootstrap.memory_lock: tue`，JVM将在启动时锁定初始堆大小。
如果`Xms`不等于`Xmx`，则某些JVM堆在调整大小后可能不会被锁定。要避免这些问题，应设置`Xms`等于`Xmx`。

## 文件描述符检查(最大打开文件数量)

Lucene 使用了 **大量的** 文件。 同时，Elasticsearch 在节点和 HTTP 客户端之间进行通信也使用了大量的套接字（注：sockets）。
所有这一切都需要足够的**文件描述符**。

可悲的是，许多现代的 Linux 发行版本，每个进程默认允许一个微不足道的 1024 文件描述符。这对一个 Elasticsearch 节点来说实在是太
**低** 了，更不用说一个处理数以百计索引的节点。

可以使用[Nodes stats](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/cluster-nodes-stats.html)
API检查为每个节点配置的`max_file_descriptors`，具体如下:

``` cURL
GET _nodes/stats/process?filter_path=**.max_file_descriptors 
```
