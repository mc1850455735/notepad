
## 什么是ThreadLocal

ThreadLocal 用于存储线程私有数据，通过 set()/get() 在当前线程内共享变量。每个Thread内部维护 ThreadLocalMap，以ThreadLocal实例为键存储独立副本。数据隔离通过线程专属的Map实现，避免多线程竞争。需注意使用后及时remove() 防止内存泄漏，弱引用键设计可辅助垃圾回收。

## ThreadLocal的作用

ThreadLocal 允许每个线程绑定自己的值，用于存储私有数据，确保各线程之间互不干扰。

ThreadLocal 变量创建时， 每个访问该变量的线程都会有一个独立的副本，可以通过 `set()` 设置副本中的值，`get()` 获取副本中的值。

其核心价值在于**避免共享数据竞争**，同时**简化跨方法参数传递**。

## ThreadLocal实现原理

`ThreadLocal` 类中，存在一个名为 `ThreadLocal` 的子类，每个线程类中都含有该子类对象；对于一个特定的 `ThreadLocal` 实例，线程调用 set() 时，就是以该 `ThreadLocal` 实例为键，向当前线程的 `ThreadLocalMap` 子类对象中插入值；读取时，就是以该实例为键，获取 `ThreadLocalMap` 中的值。

如果在一个线程中定义了两个 ThreadLocal 对象，线程内部用的都是存在于线程对象内部的 `ThreadLocalMap` 存放数据，其 key 就是 ThreadLocal 对象，其 `value` 是调用 `set()` 方法设置的值。

所以，最终变量是存放在了当前线程的 `ThreadLocalMap` 中，而不是存在于 `ThreadLocal` 类上，`ThreadLocal` 可以被看作是对 `ThreadLocalMap` 的封装。

## 内存泄漏问题

在线程使用 ThreadLocal 存储值时，实际上是将值存储到当前线程中的 `ThreadLocalMap` 中，该对象使用 `ThreadLocal` 实例作为 key，通过哈希算法计算索引，并最终存储于 `Entry extends WeakReference<ThreadLocal<?>>` 类型的数组中。

也就是说，ThreadLocalMap 的 key 是 ThreadLocal 的弱引用。如果 ThreadLocal 不被任何值指向，则该对象会在 GC 的下一次回收时被直接回收，但是 value 是强引用存在的，当 ThreadLocal 弱引用被回收时，Map 中仍然强引用存在 value，导致无法被垃圾回收。当 ThreadLocal 不被强引用而回收时，若线程长期存活导致其中的 ThreadLocalMap 持续存在，就会造成内存泄漏。

为避免内存泄漏, 在不再使用某个 ThreadLocal 后, 务必调用 remove() 方法移除其中的 value, 再对 ThreadLocal 进行释放; 同时, 在线程池等线程复用的场景下, 尽量使用 `try-finally` 块, 确保即使出现异常也可以执行 remove() 方法

## 跨线程传递ThreadLocal值

为实现异步场景下, 父子线程之间传递 `ThreadLocal` 值, 有两种解决方案: 
- `InheritableThreadLocal` : 该类为 JDK1.2开始提供的工具类, 继承自 `ThreadLocal` 类, 使用该工具类时, 会在创建子线程时令子线程继承父线程当中的 `ThreadLocal` 值, 但是不支持线程池场景下的 `ThreadLocal` 值传递
- `TransmittableThreadLocal` : 阿里巴巴开源的工具类, 继承并加强了 `InheritableThreadLocal` 工具类, 允许在线程池场景进行 `ThreadLocal` 值传递

#### InheritableThreadLocal原理
- 在线程中, 同时存在两个 ThreadLocalMap, 一个 map 用来进行存储线程中数据, 另外一个 map 名为 InheritableThreadLocal , 用来进行跨线程传递
- inheritableThreadLocals 就表示可以传输给子线程的线程持有的数据或由父线程传输而来的线程持有的数据
- 子线程由父线程创建, 创建时父线程调用 init() 方法指定 ThreadLocal 的继承逻辑. 方法调用时, 传入参数 inheritThreadLocals, 如果为 true, 说明需要继承, 则将父线程的 inheritableThreadLocals 直接传给子线程即可

#### TransmittableThreadLocal原理
- JDK默认不支持线程池场景下的 ThreadLocal 传递, TTL 实现了该功能. 由于无法改动 JDK 源码, 该类通过装饰器模式在原有功能上实现了增强.
- TTL 共改造两处: 1. 实现了自定义的 Thread, 在 run 方法内部做 ThreadLocal 的赋值操作 2. 基于线程池进行装饰, 向 `execute()` 方法提交时, 不提交 JDK 内部的 Thread, 而是提交自定义的 Thread

## 常见用途

- 数据库连接管理：每个线程独立持有连接，避免多线程竞争。
- 用户会话信息：存储当前线程的登录用户 ID 或请求 ID。
- 线程上下文传递：在多层方法调用中传递参数（如日志追踪的 TraceID）。
- 线程安全对象池：如SimpleDateFormat的线程隔离实例。
