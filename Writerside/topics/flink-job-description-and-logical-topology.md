# Flink 作业描述和逻辑拓扑

电子书: https://developer.aliyun.com/ebook/421?source=5176.11533457&userCode=b3pdrgck


接下来我们来具体的去看一下 Flink 的作业描述和逻辑拓扑。

## 作业描述


![flink-Assignment-Description.png](flink-Assignment-Description.png)

如上方所示，代码是一个简单的 Flink 作业描述。


1. 它首先定义了一个KafkaSource，说明数据源是来自于 Kafka 消息队列，然后解析 Kafka 里每一条数据。
```java
// 定义数据源
DataStream<String> lines = env.addSource(new FlinkKafkaConsumer<>());

// 解析kafka 消息
DataStream<Event> events = lines.map((line) -> parse(line));

```
2. 解析完成后，下发的数据我们会按照事件的 ID 进行 KeyBy，每个分组每10 秒钟进行一次窗口的聚合。
```Java
DataStream<Event> windowedEvents = events
        // ID 作为 Key
        .keyBy(Event::getId)
        // 每 10 秒一个窗口
        .timeWindow(Time.seconds(10))
        // 聚合
        .apply(new MyWindowAggregationFunction());
```
3. 聚合处理完之后，消息会写到自定义的 Sink。以上是一个简单的作业描述，这个作业描述会映射到一个直观的逻辑拓扑。
```Java
// 输出结果
stats.addSink(new MySink()); 
```

可以看到逻辑拓扑里面有 4 个称为算子或者是运算的单元，分别是 Source、Map、KeyBy/Window/Apply、Sink ， 我 们 把 逻 辑 拓 扑 称 为 Streaming Dataflow。

## Flink 物理拓扑
