# 概述

### 概述
- 在上述代码中, 调用服务时, 对目标的ip地址和端口port都是写死的, 默认为本机ip地址和9999端口
- 实际场景中, 服务地址和端口号应该记录到注册中心; 服务端上线时, 在注册中心注册自己的服务以及相应的地址和端口号; 客户端调用时, 直接在注册中心中根据服务名称寻找对应的服务端地址

### Zookeeper
- `zookeeper` 是一个经典的分布式数据一致性解决方案, 致力于为微服务应用提供高性能, 高可用, 且具有严格顺序访问控制能力的分布式协调存储服务, 通常被用作 注册中心和配置中心
- **高性能**
	- `zooKeeper` 将全量数据存储在内存中, 直接服务于客户端的所有非事务请求, 尤其是用于以读为主的应用场景
- **高可用**
	- `zooKeeper` 一般以集群的方式对外提供服务, 以奇数台机器构成一个集群, 每台机器都在内存中维护当前的服务器状态, 且机器之间保持互相通信
	- 只要集群中有超过一般的机器正常工作, 那么整个集群就可以正常对外提供服务
- **严格顺序访问**
	- 对于来自客户端的每一个更新请求, zookeeper 都会为其分配一个全局唯一且递增的编号, 该编号反应了事务操作的先后顺序

### 数据模型
- `zookeeper` 是一个**树形目录服务**, 其数据模型和 Unix文件系统目录树很类似. 拥有一个层次化的结构, 最顶级的节点为 `/` , 同时可以使用 path 定位 `ZNode`
- 每一个节点都被称为 `ZNode`, 每个节点上都会保存自己的**数据**和**节点信息**, 也就是说, 一个 `ZNode` 兼具文件和目录两个特点, 既可以像文件一样存储信息等数据, 也可以像目录一样作为路径标识的一部分
- 节点可以拥有多个子节点, 同时允许少量(1MB)数据存储在该节点下
- `ZNode` 大体可以分为3部分
	- 节点数据: `znode data`, 节点路径与数据的关系类似于(key, value)
	- 节点子节点: `children`
	- 节点状态: `stat`, 描述节点的创建, 修改记录
- `Zookeeper` 中节点可以分为四大类
	- `PERSISTENT` : 持久化节点
	- `EPHEMERAL` : 临时节点 `-e`
	- `PERSISTENT_SEQUENTIAL` : 持久化顺序节点 `-s`
	- `EPHEMERAL_SEQUENTIAL` : 临时顺序节点 `-es`
	![[项目/RPC/02 版本1/Inbox/Pasted image 20251025204355.png]]

### 环境配置
- 引入 Curator 客户端
```xml
<dependency>  
    <groupId>org.apache.curator</groupId>  
    <artifactId>curator-recipes</artifactId>  
    <version>5.1.0</version>  
</dependency>
```
- 导入log4j
```xml
<dependency>  
    <groupId>org.slf4j</groupId>  
    <artifactId>slf4j-log4j12</artifactId>  
    <version>1.7.25</version>  
</dependency>  
<dependency>  
    <groupId>org.apache.logging.log4j</groupId>  
    <artifactId>log4j-1.2-api</artifactId>  
    <version>2.8.2</version>  
</dependency>
```

### 启动
- 下载完成 Zookeeper 后, 在其根目录下新建 `data` 文件夹
- 在 `conf` 目录中, 创建 `zoo_sample.cfg` 的副本, 并改名为 `zoo.cfg`
- 在 `zoo.cfg` 中, 将 `dataDir` 的值改为新建的data文件夹的绝对路径
- 在 `Zookeeper` 的 `bin` 目录下
	- 先启动 `zkServer.cmd`
	- 再启动 `zkCli.cmd`

### 使用流程

**服务端**
- 服务端使用时, 向 ServiceProvider 的构造函数中传入自己的 host 和 port, 服务端需要注册方法时, 直接使用传入的host和port作为服务地址即可
- 注册方法时, 同时维护两个数据结构: `本机映射表` 和 `注册中心服务表`, 本机映射表用于当服务端被客户端访问时根据服务名和方法名找到找到方法; 注册中心服务表用于客户端发送请求时查询对应的方法所在服务器的地址
- `ZKServiceRegister` 负责建立于 zookeeper 的连接并进行操作. 建立连接时, 使用命名空间进行隔离, 并配置退避策略, 超时时间等属性.
- 注册服务时, 首先查询该服务在 zookeeper 中是否存在对应节点, 若不存在则先创建一个. 随后作为子节点插入具体服务实现者的地址

**客户端**
- 客户端使用时, 主要在 sendRequest 函数中查找实现位置部分.
- serviceDiscovery 会查找对应服务名对应的所有服务实例, 并返回一个 String 类型的 List, 并采用一定的措施进行负载均衡 (或者直接使用第一个实例)
- 将 String 类型的地址拆分为 host 和 port 部分, 创建 `InetSocketAddress` 类型的对象对二者进行组装, 并将该对象作为返回值返回, 供给 `sendRequest` 用作要发送请求的服务地址

# 客户端

### 简单类

**ServiceCenter**
- 服务管理中心接口, 负责根据服务名查找地址

### ZKServiceCenter

**客户端的初始化**
- 首先, 定义了一个指数回退重试策略, 用于在连接失败时进行自动重试
- 默认退避时间为1s, 最多尝试3次, 每次退避时间为前一次的2倍
```java
RetryPolicy policy = new ExponentialBackoffRetry(1000, 3);
```
- 使用 `CuratorFrameworkFactory.builder()` 连接到指定的 zookeeper 地址, 端口号默认为 2181. 无论是服务者还是消费者都需要与其建立连接
- `sessionTimeoutMs(40000)` 设置客户端会话超时时间为 40s, 单位为毫秒. 表示如果 40s 内客户端没有任何活动, 则 zookeeper 认为客户端失联
- `retryPolicy(policy)` 指定使用定义的重试策略
- `namespace(ROOT_PATH)` : 设置命名空间, 通过命名空间, 可以将不同服务的节点进行隔离, 避免了混淆
```java
this.client = CuratorFrameworkFactory.builder()
    .connectString("127.0.0.1:2181")
    .sessionTimeoutMs(40000)
    .retryPolicy(policy)
    .namespace(ROOT_PATH).build();
this.client.start();
```

**serviceDiscovery方法**
- 使用 `client.getChildren().forPath(xxx)` 获取 client 指定命名空间下的, path 路径(即 serviceName )下的所有子节点. 每个子节点表示一个服务提供者实例地址, 格式为 `ip:port`
```java
List<String> strings = client.getChildren().forPath("/" + serviceName);
```
- 在获得的 strings 中, 通过指定算法进行负载均衡处理, 获得本次需要调用的服务端口, 通过指定函数转为 `InetSocketAddress`, 返回给 `sendRequest` 函数
```java
String string = strings.get(0);
return parseAddress(string);
```

# 服务端

### 简单类

**ServiceRegister**
- 服务注册接口, 负责描述服务的实现

**ServiceProvider**
- 初始化时, 需要传入当前服务的地址和端口号, 以在 zookeeper 中注册服务
- 同时, 注册服务时, 会将服务同时注册到本机和 zookeeper 注册中心

### ZKServiceRegister

**InetSocketAddress介绍**
- `InetSocketAddress` 是 Java 中的一个类, 属于 `java.net` 包, 表示了一个网络地址, 通常用于在网络中表示计算机的端口, 内部包含 `ip地址` 和 `端口` 属性

**服务注册**
- 首先, 使用 `checkExists` 检查服务中心中是否已经存在服务名路径, 避免重复创建相同服务名的节点
- 若不存在, 使用 `create()` 在服务中心中创建对应节点, 同时, 为确保父路径存在, 使用 `creatingParentsIfNeeded` 一并创建父路径; 正常情况下不会出现根路径 `/` 不存在的情况, 但是可能存在误删除或者后续业务调整
- 使用 `withMode(CreateMode.PERSISTENT)` 表示使用持久化模式创建服务名节点. 持久化节点意味着该节点在 `zookeeper` 中将长期存在, 除非被手动删除
- `serverName` 对应的节点应该被配置为永久节点, 其下管理着具体服务提供者的网络地址. 当某一服务提供者下线时, 只删除服务提供者的节点, 不删除服务名对应的节点
```java
if (client.checkExists().forPath("/" + serviceName) == null) {
    client.create()
        .creatingParentsIfNeeded()
        .withMode(CreateMode.PERSISTENT)
        .forPath("/" + serviceName);
}
```
- 检测完或创建完服务名路径后, 根据传入的 `serviceAddress`, 使用自定义函数`getServiceAddress` 生成一个具体服务提供者的 `path`, `path` 路径的路径格式为 `/serverName/ip:port`
- 调用 `creatingParentsIfNeeded` 确保父路径已存在
- 使用 `withMode(CreateMode.EPHEMERAL)` 将该节点设置为一个临时节点. 当客户端断开与 zookeeper 的连接时, zookeeper 会自动删除对应的临时节点. 即临时节点会随着服务提供者的下线而自动删除, 适合用于注册服务实例
- `forPath` 指定节点路径, 即事先生成好的 `path`
```java
String path = "/" + serviceName + "/" + getServiceAddress(serviceAddress);
client.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath(path);
```
