
## 概念

- ZooKeeper是Apache Hadoop项目下的一个子项目, 是一个树形目录服务
- ZooKeeper是用来管理各个组件的管理者, 简称 zk
- Zookeeper是一个分布式的, 开源的分布式应用程序的协调服务
- Zookeeper主要提供如下功能
	- 配置管理
	- 分布式锁
	- 集群管理

## Broker注册

Kafka 的 Broker 的分布式部署且相互之间独立的，但是需要有一个注册系统管理整个集群中的 Broker，此时就需要 Zookeeper。Zookeeper 上会有一个专门用于记录 Broker 服务器列表记录的节点。记录位置位于 `/brokers/ids`。

Kafka 依赖全局唯一的数字指代每一个 Broker，不同的 Broker 必须使用不同的数字进行表示。每个 Broker 在启动时都会在 Zookeeper 中的 `/brokers/ids` 位置创建自己的节点，节点中存储 Broker 自身的 IP 地址和端口信息。Broker 创建的节点类型是临时节点，Broker 下线后该临时节点也会被自动删除。

Kafka 集群中的一个 Broker 会被选举成为 Controller，负责集群中 Broker 的上下线、所有 Topic 的分区副本分配和 Leader 选举等工作。Controller 的管理工作依赖于 Zookeeper。

## Topic 注册

在 Kafka 中，同一个 Topic 的消息会被分成多个分区并将其分布在多个 Broker 上，这些分区信息以及与 Broker 的对应关系也存储在 Zookeeper 中，由专门的节点记录，如 `/brokers/topics`。

Kafka 中的每一个 Topic 都会以 `/brokers/topics/[name]` 的格式记录在 Zookeeper 中。Broker 启动后，会到对应的 Topic 节点中注册自己的 Broker ID 节点，并在节点中记录针对该 Topic 的分区数量。该节点同样是临时节点。

## 负载均衡

由于同一个 Topic 消息会被分区并将其分布到多个 Broker 上。因此，生产者需要通过负载均衡，将消息合理地发布到分布式的 Broker 上；同时，消费者也需要进行负载均衡，实现多个消费者合理的从对应的 Broker 服务器上接受消息。

Kafka 支持传统的四层负载均衡或 Zookeeper 方式实现负载均衡。
