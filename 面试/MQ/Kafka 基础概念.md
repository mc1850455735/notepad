
# Kafka Rebalance 机制是什么？触发时机是什么？

Kafka Rebalance 发生在 Consumer Group 层面，是消费者组内部的 Partition 重新分配机制。

当 Consumer Group 中消费者数量或状态发生变化，或者 Topic 分区发生变化时，Kafka 会重新分配 Partition 与 Consumer 的对应关系。通过 Rebalance，实现了消费者之间的负载均衡，并在某个消费者出现故障时自动恢复其负责的 Partition 的消费。

Rebalance 在以下时机时会触发：
- 消费者加入 Group
- 消费者正常关闭，主动退出 Group
- **消费者异常宕机**，Broker 在超时时间内收不到心跳信号；
- **消费者处理超时**，长时间没有调用 poll()；
- Topic 分区变化，Topic 中 Partition 数量增加或减少
- 订阅关系变化，Consumer Group 订阅另一个 Topic 集合

Rebalance 的执行过程是：
1. Consumer Group 成员发生变化
2. Group Coordinator 发起重平衡
3. Consumer 重新加入消费者组
4. 根据 Partition 生成分配结果
5. Consumer 重新获得新的 Partition
6. 恢复消费

Group Coordinator 负责协调消费者组成员关系和 Rebalance 过程。
在经典协议中，Consumer Group Leader 负责生成分区分配方案，Coordinator 负责协调并下发结果；在较新的协议中，分区的分配可以由 Broker 端执行。

虽然 Rebalance 保证了高可用，但是其也存在一些问题。
首先，传统 Eager Rebalance 时通常会先撤销消费者所有原分区分配，再重新分配，这个过程可能会导致**消息消费暂停**；而 Cooperative Rebalance 支持增量调整，不必撤销所有分区，减少了影响。
Consumer 消费时，可能会在消费成功，但是还未提交 Offset 时发生 Rebalance，导致原有的 Offset 位置的消息被另一个 Consumer **重复执行**，因此需要通过业务幂等性消除这种影响。
如果消费者不断加入退出，Kafka 就需要不断调整分配，影响消费者正常工作。
