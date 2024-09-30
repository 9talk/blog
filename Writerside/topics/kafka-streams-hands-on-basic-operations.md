# Kafka Streams 动手实践

<a href="https://developer.confluent.io/courses/kafka-streams/hands-on-basic-operations/">hands-on-basic-operations</a>

基本操作练习演示了如何使用 Kafka Streams 无状态操作，例如 filter 和 mapValues。

请注意，接下来的几个步骤适用于每个练习，但只会在本练习中显示。

## 创建 KStream

创建一个订单号变量（您很快就会看到它的作用），然后创建 KStream 实例（请注意 inputTopic 变量的使用）：

```Java
KStream<String, String> firstStream = 
  builder.stream("inputTopic", Consumed.with(Serdes.String(), Serdes.String())); 
```

Spring Kafka:
```Java
KStream<String, String> stream = builder.stream("beats", Consumed.with(Serdes.String(), Serdes.String()));
```

## peek

peek 运算符不会修改键和值。在这里，它在记录进入拓扑时打印记录：

```Java
firstStream.peek((key, value) -> System.out.println("Incoming record - key " +key +" value " + value))
```

Spring Kafka:
```Java
stream.peek((key, value) -> System.out.println("Incoming record - key: " + key + " value length: " + value.length()),
        Named.as("peek-key-value-length"));
```

## filter

`filter` 添加筛选条件, 过滤出`key`含有 'a' 字符的数据

```Java
filter((key, value) -> key.contains("a"), Named.as("filter-key-contains-a"))
```

Spring Kafka:
```Java
stream
.filter((key, value) -> key.contains("a"), Named.as("filter-key-contains-a"))
.peek((key, value) -> System.out.println("Incoming record - key: " + key + " value length: " + value.length()),
  Named.as("peek-key-value-length"));
```

## mapValues

`mapValues` 用于更改值

```Java
.mapValues(value -> value.substring(value.indexOf("-") + 1))
```

## to

`to` 用于将更改后的数据发送到kafka的topic中

```Java
.to(outputTopic, Produced.with(Serdes.String(), Serdes.String()));
```