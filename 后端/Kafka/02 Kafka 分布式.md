
# ISR

ISR，即 In-Sync Replicas，同步副本集合，是 Kafka 保证消息可靠性和高可用性的核心机制。

在 Kafka 中，一个 Partition 存在多个 Replica，分为一个 Leader 和多个 Follower，但是 Follower 并不一定和 Leader 是同步的。

所以，设计了 ISR，用于管理 Leader 和所有与 Leader 同步的 Follower。之所以需要 ISR，是因为当 Leader 出现宕机等问题时，可以在 ISR 维护的有效 Replica 列表中快速选择同步的 Follower，不会出现与 Leader 不一致的现象。

ISR 主要通过 Replica 是否落后太多来判断当前的 Follower 是否同步。在 Kafka 中，Follower 通过 Fetch 向 Leader 中主动获取同步数据。所以，会通过提前设置的 `replica.lag.time.max.ms` 判断某个 Replica 是否超过了 Fetch 间隔的阈值，超过该阈值，视为 Follower 不可靠，移出 ISR。

`acks` 参数表示，Producer 向 Topic 发送数据后，需要接收多少 ack，才被认为是发送成功。存在三个值，分别是 `0`、`1` 和 `all`。`0` 表示不等待任何响应，直接认为成功；`1` 表示 Leader 返回成功确认后，视为写入成功；`all` 表示 Leader 和所有的 Follower 全部返回确认后，才会视为写入成功。默认设置为 `1`。

`min.insync.replicas` 参数表示，Producer 进行写入时，ISR 中至少要存在多少个副本。通常与 acks 参数配合使用，确保写入的可靠性，使 Kafka 在节点返回不成功和 ISR 节点数量过少时都不认为写入成功。

与 ISR 相对的，还存在 AR（所有 Replicas） 和 OSR（非同步 Replicas）两个集合。