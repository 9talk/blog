# Flink 核心概念

电子书: https://developer.aliyun.com/ebook/421?source=5176.11533457&userCode=b3pdrgck

Flink 的核心概念主要有四个：Event Streams、State、（Event）Time和Snapshots。

## Event Streams

即事件流，事件流可以是实时的也可以是历史的。Flink 是基于流的，但它不止能处理流，也能处理批，而流和批的输入都是事件流，差别在于实时与批量。

## State
Flink 擅长处理有状态的计算。通常的复杂业务逻辑都是有状态的，它不仅要处理单一的事件，而且需要记录一系列历史的信息，然后进行计算或者判断。

## （Event）Time
最主要处理的问题是数据乱序的时候，一致性如何保证。

## Snapshots
实现了数据的快照、故障的恢复，保证数据一致性和作业的升级迁移等。