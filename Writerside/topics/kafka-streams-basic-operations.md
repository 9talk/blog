# Kafka Streams 基础操作

<a href="https://developer.confluent.io/courses/kafka-streams/basic-operations/">basic-operations</a>

本模块涵盖 Kafka Streams 的基本操作（例如映射和筛选），但从其主要构建块开始：事件流和拓扑。

## 事件流

事件表示与操作 （如通知或状态传输） 对应的数据。事件流是事件记录的无限集合。

## 键值对 (Key-Value Pairs)

Apache Kafka® 以键值对形式工作，在事件流中，具有相同键的记录彼此之间没有任何关系。

例如，下图显示了四个独立的记录，即使其中两个键相同：

![kafka-streams-basic-operations-key-value-pairs.png](kafka-streams-basic-operations-key-value-pairs.png)

## 拓扑 (Topologies)

为了表示流处理流程，Kafka Streams 使用有向无环图 （`DAG`） 的拓扑。

![kafka-streams-basic-operations-topology.png](kafka-streams-basic-operations-topology.png)

每个 Kafka Streams 拓扑都有一个源处理器，其中记录从 Kafka 读入。

下面是执行某种操作的任意数量的子处理器。

一个子节点可以有多个父节点，一个父节点可以有多个子节点。这一切都发送到 sink 处理器，该处理器写入 Kafka。

## 流 (Streams)

可以使用 `StreamBuilder` 定义流，可以通过 `Consumed` 配置对象为其指定输入主题以及 SerDes 配置。
这些设置告诉 `Kafka Streams` 如何读取记录，在本例中，这些记录具有字符串类型的键和值：

```Java
StreamBuilder builder = new StreamBuilder();
KStream<String, String> firstStream = 
  builder.stream(inputTopic, Consumed.with(Serdes.String(), Serdes.String()));
```

`KStream` 是 Kafka Streams DSL 的一部分，它是将使用的主要结构之一。

## 流操作 (Stream Operations)

创建流后，可以对其执行基本操作，例如映射和筛选。

### 映射 (Mapping)

使用映射，您可以获取一种类型的输入对象，对其应用函数，然后将其输出为不同的对象，可能是另一种类型的对象。

![kafka-streams-basic-operations-mapping-stream-operation.png](kafka-streams-basic-operations-mapping-stream-operation.png)

例如，使用 `mapValues`，您可以将一个字符串转换为另一个截取前五个字符的字符串：

```Java
mapValues(value -> value.substring(5))
```

另一方面，Map 允许更改键和值：

```Java
map((key, value) -> ..)
```

### 筛选 (Filtering)

使用筛选条件，您可以发送键和值，并且只有与条件匹配的记录才能到达另一端。

```Java
filter((key, value) -> Long.parseLong(value) > 1000)
```

![kafka-streams-basic-operations-filter-operation.png](kafka-streams-basic-operations-filter-operation.png)