
### ConcurrentHashMap 的实现

#### JDK1.8之前

JDK1.8 之前，采用 `Segment数组 + HashEntry数组 + 链表` 的形式实现，`Segment` 数组默认元素数量为16个，一旦确定就不能变化。`Segment` 是一种可重入锁，`HashEntry` 用于存储键值对。

`ConcurrentHashMap` 将加入的元素**按照哈希值**分配到不同 `Segment` 中，当试图对对应的 `HashEntry` 修改时，必须先获得对应的 `Segment`。不同 `Segment` 可以并发写入，同一 `Segment` 并发写入会导致阻塞。

![[面试/Java/集合/Inbox/Pasted image 20251127115727.png]]

#### JDK1.8及之后

JDK1.8 及之后，对 `ConcurrentHashMap` 进行了彻底重写。

不再使用 `Segment` 分段锁，改为 `Node + CAS + synchronized` 保证并发安全；同时采用类似于新版 `HashMap` 的数据结构，使用 `数组 + 链表 + 红黑树` 的形式，在**链表长度大于8**时进行红黑树转换。这种实现方式锁粒度更细，`synchronized` 只锁当前链表或者红黑树的首节点，**只要不发生 hash 冲突**的并发访问就不会锁节点，不影响其他 Node 的读写，**大大提升**读写效率。

![[面试/Java/集合/Inbox/Pasted image 20251127121213.png]]

### 和HashTable区别

#### ConcurrentHashMap

**JDK1.7 及之前**, 底层采用**分段的**数组+链表实现. `ConcurrentHashMap` 对整个桶数组进行分割分段, 分为多个 `Segment` 可重入锁, 每个锁只锁容器内部分数据. 多线程访问容器内不同数据段的数据时就不存在锁竞争, 提高了并发访问效率.

**JDK1.8 之后**, 底层数据结构与 HashMap 一致. 摒弃 `Segment` 概念, 改用 Node 数组 + 链表 + 红黑树, 并配合使用 `synchronized` 和 `CAS` 进行并发操作. 类似优化后且线程安全的 HashMap, 虽然 JDK1.8 中还可以看到 Segment, 但已简化属性, 只是为了兼容旧版本.

`ConcurrentHashMap` 中使用 `TreeNode` 存储红黑树节点, 并使用 `TreeBin` 对节点进行包装, TreeBin 使用 waiter 维护红黑树根节点, 阻止其他线程进入, 预防红黑树旋转过程中由于根节点变化导致其他线程读写红黑树异常.

#### HashTable

底层采用 **数组 + 链表** 的形式实现。类中所有方法都使用 `synchronized` 关键字进行修饰，以确保线程安全，效率非常低下。当多个线程同时访问时，只能有一个线程进行操作，导致越来越多线程阻塞，效率越来越低。

### key和value不能为null

在 `ConcurrentHashMap` 中，无论 `key` 还是 `value` 都不能置为空，这是因为 `null` 作为 `key` 或者 `value` 时存在二义性，单线程下可以该容忍歧义，而多线程下无法容忍。

以 `null` 作为 `k/v` 的二义性主要体现在 `get(k)` 返回 `null` 时，无法确认实际情况：
1. `key=null` 不存在
2. `key=null` 存在但是 `value=null`

在 HashMap 中，这种二义性可以通过 get(k) 后的 `containsKey(k)` 方法进行二义性的消除。但在 `ConcurrentHashMap` 中，两次操作之间可能插入其他线程对 `ConcurrentHashMap` 的修改，使 `containsKey(k)` 方法结果不可信。

如果确实需要在 `ConcurrentHashMap` 中使用 null，可以由使用者手动定义特殊的静态空对象 (如 `NULL = new Object()` ) 替代 null。

### 复合操作原子性

`ConcurrentHashMap` 是线程安全的, 意味着多线程情况下同时进行读写操作时不会出现数据不一致情况, 也不会导致如 JDK1.7 及之前 HashMap 的多线程操作链表死循环问题. 但只能确保单个操作的安全, 不意味着复合操作也是原子性的. 在两个基本操作之间, 其他线程可能执行, 从而导致结果改变.

为保证部分复合操作的原子性, `ConcurrentHashMap` 提供了一些原子性的复合操作, 如 `putIfAbsent`, `compute`, `computeIfAbsent` , `computeIfPresent`、`merge`等. 通过单个方法调用, 根据给定的 `key` 和 `value` 在内部进行一系列操作

虽然复合操作也可以通过加锁的方式实现同步, 但这种方式效率相对低, 且违背了使用 `ConcurrentHashMap` 的初衷, 尽量使用提供的原子性的复合方法.




