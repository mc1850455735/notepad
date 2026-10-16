
# go 语言中 map 的扩容机制是什么

**旧版 Map**
在 1.23 之前，go 主要使用 Bucket + Overflow Bucket 的哈希表结构，使用渐进式迁移实现扩容。

旧版 map 通过 hmap 管理哈希表，每个 bucket 最多存储 8 个键值对，当一个 Bucket 存满时，该 Bucket 可以后接 Overflow Bucket，通过 Overflow Bucket 继续存储数据。

当 hmap 中负载因子超过阈值、或 Overflow Bucket 过多时，就会执行扩容。扩容分为**翻倍扩容和等量扩容**。

负载因子超过阈值时，就会触发**翻倍扩容**。平均每个主 Bucket 存储超过 **6.5 个元素**时就会扩容，旧的 Bucket i 中的元素，扩容后会重新分布到 Bucket i 或者 Bucket i+旧桶数。翻倍扩容的过程采用的是渐进式扩容的方法，每次分配原来容量 2 倍的 Bucket 数组。分配完新的 Bucket 后，将旧的 Bucket 数组保存到 oldbuckets 中，后续操作 map 的过程中，逐步迁移 oldbuckets 中的数据到新 Bucket 中，在操作过程中，根据对应的 Bucket 是否迁移，决定是从旧数组还是新数组中进行查找。全部迁移完成后，回收旧 Bucket 数组。

Overflow Bucket 过多时，就会触发**等量扩容**。即使整个哈希表的负载因子不高，也可能因为哈希冲突、反复插入删除等，产生很多溢出桶。此时执行等量扩容，不改变主 Bucket 数量，但是通过重新组织数据的方式，减少不必要的溢出桶。

**新版 Map**
在 1.24 及之后，默认采用基于 Swiss Table 思想的新实现，通过多层目录和多个哈希表实现动态增长。不再使用旧版的全局 Bucket 数组渐进式迁移。

新版 Map 采用：目录(Directory)、哈希表(Table)、组(Group)、槽位(Slot)的结构进行组织，每个 Table 可以独立扩容。当遇到一下情况时，会进行扩容：
- Table 达到负载阈值
- 如果 Table 未达到最大容量，则直接扩容为原来的两倍
- 如果 Table 已达到最大容量（1024 个 Slot），则将其拆分为两个 Table
- 如果存在较多删除后留下的 tombstone，也可能通过 Rehash 整理而不增加容量

新版默认使用 7/8 的负载因子阈值。

相比旧版，新版更适合大 map，因为新版扩容时只需要操作一个 Table，避免一次性迁移一整个 map。
