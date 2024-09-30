# Kafka Streams 入门 

<a href="https://developer.confluent.io/courses/kafka-streams/get-started/">get-started</a>

要了解 Kafka Streams，需要从 Apache Kafka 开始，这是一个分布式、可扩展、弹性和容错的事件流平台。

## Logs, Brokers, and Topics

Kafka 的核心是日志，它只是一个附加记录的文件。

日志是不可变的，但通常无法存储无限量的数据，因此可以配置记录的生存时间。

## Producers and Consumers

可以通过生产者将数据放入 Kafka，然后通过消费者将其传出：生产者生产数据，每条数据在到达时都会被赋予一个称为偏移量的特殊编号，该编号只是该记录在日志中的逻辑位置。

消费者发送 fetch 请求来读取记录，并使用偏移量来添加书签，就像占位符一样。

例如，消费者将读取到偏移量 5，当它返回时，它将从偏移量 6 开始读取。

消费者被组织消费组，分区数据分布在消费组的成员之间。

## 处理数据对比

处理用 Kafka Streams 编写的代码比使用低级 Kafka 客户端编写的相同代码要简洁得多。

在下面的代码中，将创建创建者和使用者，然后订阅单个topic 'widgets'。然后，poll（），并返回 ConsumerRecords 集合。

遍历记录并提取值，筛选出'red'  records。
然后获取 “red” 记录，

为每个记录创建一个新的 ProducerRecord，并将它们写出到 'widgets-red' topic。

```Java
public static void main(String[] args) {
 // 创建消费者
 try(Consumer<String, Widget> consumer = new KafkaConsumer<>(consumerProperties());
    // 创建生产者
    Producer<String, Widget> producer = new KafkaProducer<>(producerProperties())) {
        // 消费者订阅主题
        consumer.subscribe(Collections.singletonList("widgets"));
        while (true) {
            // pull
            ConsumerRecords<String, Widget> records = consumer.poll(Duration.ofSeconds(5));
                for (ConsumerRecord<String, Widget> record : records) {
                    Widget widget = record.value();
                    if (widget.getColour().equals("red") {
                        // 创建记录并发送
                        ProducerRecord<String, Widget> producerRecord = new ProducerRecord<>("widgets-red", record.key(), widget);
                        producer.send(producerRecord, (metadata, exception)-> {…….} );
               …
```

以下是在 Kafka Streams 中执行相同操作的代码：

```Java
final StreamsBuilder builder = new StreamsBuilder();

builder
    // 创建stream流, 消费数据
    .stream(“widgets”, Consumed.with(stringSerde, widgetsSerde))
    // 过滤记录
    .filter((key, widget) -> widget.getColour.equals("red"))
    // 发送到新主题
    .to("widgets-red", Produced.with(stringSerde, widgetsSerde));
```
