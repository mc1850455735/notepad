
# 虚拟线程

### 背景问题

传统线程（Platform Thread）：
- 创建成本高
- 占用系统线程
- 高并发场景（如Web服务）资源消耗大

### 虚拟线程特点
- 轻量级线程（由JVM调度，不直接绑定OS线程）
- 可以创建百万级线程
- 编程模型仍然是“同步阻塞风格”

### 优势
- 简化高并发编程
- 不需要复杂的异步回调
- 更适合 Web 服务、IO 密集型系统

# 结构化并发

让多个线程作为一个“任务整体”来管理。

**优势**
- 子任务自动管理生命周期
- 失败自动传播
- 避免“线程泄露”

# switch 模式匹配

增强了 Switch 功能。

# Record 模式匹配

可以直接解构 Record。

# Sequenced Collections

为集合增加顺序接口 Sequenced Collections。新增 SequencedCollection、SequencedSet、SequencedMap 接口及其相关方法。统一了 List / LinkedHashSet / LinkedHashMap 的顺序操作能力。

# 字符串与模板增强

通过使用字符串模板，比传统字符串拼接更安全。

**优势**
- 避免SQL注入
- 避免拼接错误
- 可扩展模板处理器

# 性能优化

**ZGC / G1 改进**
- ZGC 支持更好
- 更低延迟
- 更好内存管理
