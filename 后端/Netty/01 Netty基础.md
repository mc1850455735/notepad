# 基本概念

### 阻塞与非阻塞

- 阻塞与非阻塞是进程访问数据时, 对未就绪数据的方式
- **阻塞**: 如果缓冲区内数据未准备好, 则等待, 直到数据准备完毕
- **非阻塞**: 如果缓冲区内未准备好, 则直接返回, 并在一定时间后重新查询, 直到数据准备完毕, 返回数据给对应线程

### 同步与异步

- 同步与异步都是操作系统处理IO事件的方式
- 通过异步的方式, 可以将同步IO阻塞等待时消耗的资源用于处理其他事务, 提高服务自身的稳定性

**同步**
- 应用程序直接参与数据的IO读写
- 处理IO事件时, 必须阻塞在某个方法上, 等待IO事件完成

**异步**
- IO读写操作直接交由操作系统处理, 应用程序只需要等待通知
- 在进行IO时, 由操作系统完成相关操作, 应用程序可以执行其他操作, IO完毕后, 操作系统给应用程序一个通知

# BIO与NIO

### BIO

**概述**
- `BIO` 是一个**同步, 阻塞**的IO方式, 对应 `java.io` 包, 基于流模型进行实现, 提供了最常见的IO操作, 如 File抽象, 输入输出流 等功能.
- 在输入/输出完成前, 线程会一直阻塞, BIO之间的调用是**线性顺序**, 比较可靠

**通信模型**
![[后端/Netty/Inbox/Pasted image 20251024155526.jpg]]

### NIO

**概述**
- NIO是一种同步非阻塞的IO模型, 于 Java1.4 引入, 对应 `java.nio` 包, 提供了如 `Channel`, `Selector`, `Buffer` 等抽象
- 支持面向缓冲的, 基于通道的IO方法
- NIO提供了对应 `Socket` 和 `ServerSocket` 的 `SocketChannel` 和 `ServerSocketChannel` 两种套接字通道实现, 都支持阻塞和非阻塞两种模式.
- 对于高负载高并发的网络应用, 应用NIO的**非阻塞模式**进行开发
- **特点**: 
	- 一个线程可以处理多个通道, 减少线程创建数量
	- 读写非阻塞, 节约资源; 没有可读写数据时不会因阻塞而浪费线程资源

**通信模型**
- Client 通过 `SocketChannel.open()` 创建一个非阻塞的TCP通道, 并通过 `clientChannel.configureBlocking(false)` 设置其为非阻塞模式
- `clientChannel` 通过 `connect` 函数进行连接, 根据返回值判断连接是否完成
- 若连接未立即完成, 则将 `clientChannel` 注册到 `Selector`, 通过 Selector 监听 IO 操作是否完成
![[后端/Netty/Inbox/Pasted image 20251024155531.jpg]]

### Reactor模型


# Netty

**简介**
- Netty 是一个 NIO 客户端服务器框架, 可以快速开发网络应用程序, 如协议服务器和客户端, 简化了网络编程的过程

### 执行流程
1. 服务端中, Server启动, Netty 从 ParentGroup 中选出一个 NioEventLoop 对指定 port 进行监听操作
2. 客户端中, Client启动, Netty 从 EventLoopGroup 中选出一个 NioEventLoop对 连接Server并处理 Server来的数据
3. Client连接Server的port, 创建 Channel
4. 服务端中, Netty 从 ChildGroup 中选出一个 NioEventLoop 与该 channel 绑定, 用于处理该 Channel 中所有的操作
5. Client通过Channel向Server发送数据包
6. 服务端中, Pipeline中的处理器依次对Channel中的数据包进行处理
7. 服务端中, Server如需向Client发送数据, 则需要将数据经过pipeline中的处理器处理形成 ByteBuff 数据包进行传输
8. Server 将数据包通过 channel 发送给 Client
9. 客户端中, Pipeline中的处理器依次对channel中的数据包进行处理
![[后端/Netty/Inbox/Pasted image 20251024202626.jpg]]

### 核心组件

##### Channel
- Channel 是 Java NIO 的一个基本构造, 可以看作是传入或者传出数据的载体, 可以被打开或者关闭, 连接或者断开连接

##### EventLoop与EventLoopGroup

**EventLoop**
- EventLoop 定义了 Netty 的核心抽象, 用来处理连接生命周期中发生的事情, 内部会为每一个 Channel 分配一个 EventLoop

**EventLoopGroup**
- EventLoopGroup 是一个 EventLoop 池, 其中包含很多 EventLoop

**总结**
- EventLoop 本身是一个线程驱动, 在其生命周期内只会**绑定一个线程**, 使得该线程处理一个 Channel 的所有IO事件
- Channel 一旦与一个 EventLoop 绑定, 在其整个生命周期内是**不能改变**的, 一个 EventLoop 可以与多个不同的 Channel 绑定, 但是 Channel 只能绑定一个 EventLoop

##### ServerBootstrap与Bootstrap

**概述**
- ServerBootstrap与Bootstrap被称为引导类, 即对应用程序进行配置, 并使其运行起来的过程
- Netty处理引导类的方法是使应用程序与网络层隔离

**Bootstrap**
- Bootstrap是客户端的引导类, 在调用 `bind()` (连接UDP) 和 `connect()` (连接 TCP) 时, 会新创建一个 Channel, 仅创建一个单独的, 没有父 Channel 的 Channel 实现所有网络交换

**ServerBootstrap**
- ServerBootstrap是服务端的引导类, ServerBootstrap 在调用 bind() 方法时会创建一个 ServerChannel 来接受来自客户端的连接, 且该 ServerChannel 管理了多个子 Channel 用来与客户端之间进行通信

##### ChannelHandler与ChannelPipeline

- ChannelHandler是对Channel中数据的处理器, 这些处理器可以是系统本身定义好的编解码器等, 也可以是用户自定义的处理器.
- 这些处理器会被统一添加到一个 ChannelPipeline 对象中, 并按照添加的顺序对 Channel 中的数据进行依次处理

##### ChannelFuture

- Netty 中的所有 IO 操作都是异步的, 即操作不会理解返回结果
- 因此, Netty 中定义了一个 ChannelFuture 对象, 用于表示异步操作本身
- 如果向获取到该异步操作的返回值, 可以通过该异步操作的 addListener() 方法为该异步对象添加监听器, 为其注册回调; 结果出来后, 立即执行回调函数
- Netty的异步编程模型都是建立在 Future 与 回调 的概念之上的













