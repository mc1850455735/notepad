
![[面试/Java/并发编程/Inbox/3PnMSzFO_qnghbthpgU_mianshiya.webp]]


# AQS

`AQS`，`Abstract Queued Synchronizer`, 抽象队列同步器，是从 `JDK1.5` 开始提供的 Java 并发核心组件, 位于 `java.util.concurrent.lock` 包下。

`AQS` 是一个抽象类, 用于为同步器提供通用执行框架。
他定义了资源的获取和释放的**通用流程**，而具体实现逻辑由同步器类通过重写方法来实现。

通过底层封装，`AQS` 隐藏了线程同步机制和线程管理，使开发者专注于具体的同步逻辑。

通过 `AQS`, 可以简单且高效地实现各种同步器, 如 `ReentrantLock`, `Semaphore`, `CountDownLatch` 等。

## AQS 执行流程

AQS 封装了一个 volatile int 类型的变量，名为 state；同时维护一个 FIFO 的双向同步队列。对于不同的 AQS 实现，state 表示的含义都可能存在不同，具体依据实现来决定。

当一个线程试图获取锁时，先通过 CAS 修改 state，如果成功则拿到锁，如果失败则被封装为一个 Node 加入到双向队列中，并 park 挂起等待；前方线程释放资源后，会 unpark 唤醒队列中的头部节点。

## AQS中CLH锁

### CLH锁

- 自旋锁通过线程不断对原子变量执行 CAS 操作来尝试获取锁. 在高并发场景下, 多个线程同时竞争一个原子变量, 容易造成某个线程长时间 CAS 获取锁失败的情况, 从而导致饥饿问题.
- CLH 锁引入单链表队列组织并发竞争的线程, 对自旋锁进行了改进. 在多线程场景下, 将每个线程作为节点组织成一个单链表, 并通过自旋操作监控前一个线程节点的状态, 而不是直接竞争共享变量. 
- 线程按顺序排队, 前一个线程释放锁后, 当前节点才可以获取锁. 保障了线程之间获取锁的公平性, 避免了饥饿问题.
![[面试/Java/并发编程/Inbox/Pasted image 20251124174321.png]]

### 变体CLH
- AQS 中借鉴了 CLH 锁的思想, 使用双链表作为阻塞队列, 形成 CLH 的变体
- 改进1: CLH 中节点通过自旋操作监控前一个节点, 导致大量自旋操作占用 CPU 资源. AQS 引入**自旋+阻塞**混合机制, 线程节点首先会通过短暂**自旋** CAS 操作尝试获取锁; 如果获取锁失败, 则将线程**阻塞**, 等待前驱节点唤醒, 从而减少了 CPU 的浪费.
- 改进2: CLH 中使用单链表队列, 节点只能获取前驱节点状态, 而 AQS 希望可以在该节点获取到锁后, 唤醒后继节点. 因此换用双向链表队列, 使节点不仅可以知道前驱信息, 也可以知道后继信息, 提高了唤醒节点效率.
![[面试/Java/并发编程/Inbox/Pasted image 20251124181243.png]]

## AQS高性能原因

1. 使用基于 CLH 变体的队列, 线程只与自己的前驱节点交互, 避免所有线程同时竞争锁, 显著减少了 CAS 冲突次数和总线争用现象.
2. 使用自旋+阻塞的混合策略。线程获取锁失败后不会立即阻塞, 而是短暂自旋尝试后再阻塞. 即避免了长期阻塞导致的 CPU 浪费, 也避免了直接挂起导致的频繁上下文切换.
3. 采用 `LockSupport.park/unpark` . 相较于传统的 `wait/notify` 效率更高, 无需依赖对象监视器, 同时实现了精确唤醒, 避免了不必要的唤醒操作.
4. 大量使用 `CAS` 操作实现对队列的操作. `CAS` 是用户态的原子操作, 基于 `CAS` 可以在避免内核切换的情况下实现队列的同步访问. 吞吐量更高, 延迟更低；
5. AQS 只唤醒队列中的下一个线程节点, 而不是广播唤醒多个线程, 避免竞争风暴, 提高整体吞吐量.

## AQS Node状态

AQS 中的 waitStatus 状态类似于**状态机**, 通过不同状态表明 Node 节点的不同含义, 并使用不同的操作控制 Node 状态之间的流转。

**Node 状态**
`0`：新节点加入队列的状态；

`SIGNAL`：当有新节点入队时，其前驱节点状态就会由 `0` 转为 `SIGNAL`, 表示前驱节点释放锁后，需要对新节点进行唤醒操作；
如果 `SIGANL` 状态的节点对后继节点进行了唤醒操作，则该节点状态由 `SIGNAL` 转为 `0`，通过清除 `SIGNAL` 状态表示唤醒操作执行完毕。

`CANCELLED`：如果节点在队列中等待获取锁时, 因为某种原因失败了, 该节点的状态就会标为 `CANCELLED`, 表示取消获取锁；
这种状态节点是异常的, 无法被唤醒, 也无法唤醒其后继节点.

## AQS核心思想

AQS 核心思想是, 如果被请求的共享资源空闲, 则将当前请求资源的线程设为有效工作线程, 并将共享资源设为锁定状态；
如果被请求的共享资源被锁定，则当前请求资源的线程应该通过一套阻塞等待及唤醒的锁分配机制进行阻塞、等待、唤醒操作；

在 AQS 中，这套锁机制是通过 CLH 锁的变体实现的。

### 状态信息

AQS 使用 int 类型的成员变量 state 表示同步状态，该变量使用 volatile 修饰，确保使用时在线程间的可见性。
同时, 通过内置的线程等待队列完成获取资源线程的排队工作。

对 state 的操作通过 protected 类型的方法实现，且这些方法是使用 final 修饰的，无法在子类中被重写。

# ReentrantLock

### 概述
- `ReentrantLock`, 可重入锁, 是一个可重入且独占式的锁, 实现了 Lock 接口. 其与 synchronized 关键词类似, 但是更灵活更强大, 增加了轮询, 超时, 中断, 公平锁, 非公平锁等高级功能.
- `ReentrantLock` 中定义了 Sync 内部类, 该内部类中实现了大部分添加锁和释放锁的操作, 该类有 FairSync 公平锁和 NonfairSync 非公平锁两个子类; 可重入锁 `ReentrantLock` 的底层是由 [[CAS 与 AQS|CAS 与 AQS]] 实现的.
- `ReentrantLock` 默认使用非公平锁, 也可以使用构造器参数指定使用公平锁.

### 公平锁与非公平锁的区别
- 公平锁: 锁被释放以后, 先申请的线程先得到锁. 为保证时间上的公平, 性能相对较差一些, 上下文切换更频繁.
- 非公平锁: 锁被释放以后, 后申请的线程也有可能先得到锁; 得到锁的顺序是随机或者按线程优先级排序的. 由于不需要考虑时间, 性能更好, 但是可能存在线程一直无法获取到锁的情况出现.

### 与synchronized区别
- 两者都是**可重入锁**. 可重入锁又称递归锁, 指**线程**获得了一个锁后, 如果该锁还没有释放, 可以再次获取自己的内部锁. JDK 提供的所有 Lock 实现类都是可重入的.
- synchronized 在 JVM 虚拟机的层次进行实现; 而 ReentrantLock 在 JDK API 的层次进行实现, 需要手动进行 `lock()` 和 `unlock()`, 并且可以查看源码
- ReentrantLock 比 synchronized 增加了一些高级功能: 
	1. 等待可中断: ReentrantLock 提供了一种中断等待锁的线程的机制, 通过提供的 `lockInterruptibly()` 方法实现该机制. 如果阻塞等待过程中其他线程中断了当前的线程, 会抛出线程中断异常 `InterruptedException`, 可以捕获该异常进行处理
	2. 可实现公平锁: ReentrantLock 提供了公平锁的实现, 而 synchronized 关键字提供的锁只能是非公平锁.
	3. 可实现选择性通知: synchronized 关键字通过 `wait()` / `notify()` 机制可以实现等待/通知机制, 而 ReentrantLock 的实现需要借助 Condition 接口与 `newCondition()` 方法
	4. 支持超时: ReentrantLock 允许设定超时时间, 锁等待最长时间后仍未获得, 则直接失败, 不会一直等待

### 补充

#### 等待可中断
Java 中的线程在执行 `interrupt` 中断操作时, 只有当特定情况下, 才会响应中断, 并抛出 `InterruptedException` 异常. 其中包括当线程通过执行 `lockInterruptibly()` 方法执行加锁情况. 当执行该方法时, 检查是否存在中断标志, 存在中断标志则抛出异常. 否则尝试加锁, 加锁成功继续向下执行, 否则阻塞等待; 如果等待过程中出现了对该线程的中断, 则抛出异常

#### Condition 接口
Condition 接口在JDK1.5加入, 可以实现多路通知功能, 即一个 Lock 中可以创建多个 Condition 实例. 线程对象可以注册到某个特定的 Contidion 实例中, 从而有选择性的进行线程通知. 对于 synchronized, 执行 notify() / notifyAll() 时, 只能通知随机一个等待线程或全部等待线程, 而通过 ReentrantLock 中持有的某个 Condition 的 signalAll() 方法, 可以有选择性的只唤醒注册在这个 Condition 中的线程.

# ReentrantReadWriteLock



# StampedLock


# Semaphore

#### 作用
- `Semaphore` 可以用来控制同时访问某个资源的线程数量. 创建时, 在构造函数中指定数量, 每次执行 `acquire()` 方法时数量减1, 执行 `release()` 方法时数量加1. 只有当 `Semaphore` 中数量不为0时线程才能够获取资源. 当 `Semaphore` 中的资源数量设置为1时, `Semaphore` 退化为排他锁
- `Semaphore` 中存在公平和非公平两种模式, 其设置在构造函数中进行. 公平锁时 FIFO 的, 而非公平锁是抢占式的. 无论采用什么类型的锁, 都必须提供许可的数量.
- `Semaphore` 通常用于资源有明确访问数量限制的场景, 但通常来说仅限于单体模式应用程序, 实际项目中更推荐使用 `Redis` + `Lua`

#### 原理
- Semaphore 是共享锁的一种实现, 其默认设置 state 为构造函数传入的资源数量. 当调用 `semaphore.acquire()` 时, 线程获取许可证, 判断是否 state >= 0, 使用 CAS 更新state 的值; 如果 state < 0, 则将当前任务加入阻塞队列, 挂起当前线程. 当调用 `semaphore.release()` 时, 线程释放许可证, 使用 CAS 操作修改 state 的值, 使 state = state + 1, 释放许可证成功后, 同步唤醒等待队列中下一个线程, 该线程继续尝试获取许可证, 若不成功则继续进入阻塞队列

# CountDownLatch

#### 作用
- `CountDownLatch` 允许 `count` 个进程阻塞在调用处, 直到所有线程任务都执行完毕. 
- `CountDownLatch` 是一次性的, 计数器中的值只能在构造函数中初始化一次, 之后就无法继续设置. `CountDownLatch` 使用完毕后无法再次使用.

#### 原理
- `CountDownLatch` 是共享锁的一种实现, 其默认设置 state 为构造函数传入的 `count` 数量. 线程使用 `countDown()` 时, 实际上就是以 CAS 操作减少了 state, 直到 state 为0.
- 当调用 `CountDownLatch` 中的 `await()` 时, 如果 state 不为 0, 则该方法会一直阻塞等待, 直到 `count` 个线程调用了 `countDown()` 使 `state` 为0, 或者调用 `await()` 方法的线程被中断, 线程才能在阻塞中被唤醒, 继续向下执行.

#### 应用场景
- 对于需要等待所有线程执行完毕的情况, 可以通过 `CountDownLatch` 类对象进行. 循环插入线程池所有的任务, 并使用 try-finally 包裹任务目标, 使任务执行完毕后进行 `countDownLatch.countDown()`.
- 在需要获取数据之前, 使用 `countDownLatch.await()`, 阻塞等待确保所有需要执行的任务都已经执行完毕.
- 在 Java8 之后提供了 `CompletableFuture` 类, 通过类中的 allOf() 方法, 可以将需要并行执行的任务都加入对象中, 并指定对返回结果的处理方式.

# CyclicBarrier

#### 作用
- `CyclicBarrier` (可循环使用屏障), 其作用是使一组线程到达屏障 (同步点) 后被阻塞, 直到最后一个线程到达屏障时, 屏障才会开门, 所有被拦截的线程继续开始工作.
- 与 `CountDownLatch` 类似, 也用来实现线程间的计数等待, 但其功能相较于 `CountDownLatch` 更加强大, 应用场景与 `CountDownLatch` 相似.
- `CountDownLatch` 是基于 `AQS` 的, 而 `CyclicBarrier` 是基于 `ReentrantLock`  和 `Condition` 的 (`ReentrantLock` 也基于 `AQS`).

#### 原理
- `CyclicBarrier` 内部通过 `count` 变量作为计数器标记当前等待到达的线程数量, 使用 `final` 修饰的 `parties` 变量标记屏障总计要拦截的线程数量. 当 `count` 值为0时, 打开栅栏放出所有线程. 通过构造方法指定 `CyclicBarrier` 的 `parties`, 令 `count` 初始值等于 `parties`
- 当调用任务内部调用 `CyclicBarrier` 的 `await()` 时, 实际上就是调用 `dowait` 方法. `await()` 就像是栅栏挡住了线程, 只有当拦截线程的数量达到 `parties` 时, 栅栏才会打开, 线程得以通过执行.
