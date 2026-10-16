
# HashMap, ConcurrentHashMap, Hashtable 的 key 与 value 是否可以为 null？原因是什么？

| Map 类型            | Key 可以为 null | Value 可以为 null |
| ----------------- | ------------ | -------------- |
| HashMap           | ✅ 可以（最多一个）   | ✅ 可以（多个）       |
| Hashtable         | ❌ 不可以        | ❌ 不可以          |
| ConcurrentHashMap | ❌ 不可以        | ❌ 不可以          |

HashMap 可以设置 key 和 value 为 null。这是因为 HashMap 对 null key 做了特殊处理，当 key 为 null 时，会将 null key 设为 0，并根据这个 0 作为 hash code 插入到数组中。实际查询时，可以通过 containsKey(key) 去除二义性。

Hashtable 不可以设置 key 或者 value 为 null。这个原因与 ConcurrentHashMap 类似，如果有不存在的 key，可以直接通过 null 返回知道其不存在，如果支持 null value，会引入一些二义性，同时因为并发安全的限制，没有办法高效的消除这种二义性。

ConcurrentHashMap (简称 CHM) 不可以设置 key 或者 value 为 null。CHM 与 HashMap 类似，但是其不具备通过 containsKey 消除二义性的可能，因为即使通过 containsKey 完成相关操作后，再进行下一个操作，两个操作之间并不是原子的，无法保证在此期间不会有其他线程对 CHM 进行修改；如果使用 synchronized 修饰符，虽然可以保证两个操作在同一个线程内完成，但是违背了 CHM 设计的初衷。处于一致性考虑，key 和 value 一样，


