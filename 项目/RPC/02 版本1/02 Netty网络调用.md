# 概述

### 概述
- 在客户端与服务端通信的过程中, 只使用 Java 原生 Socket 的方式效率低
- 引入 Netty 高性能网络框架进行了优化
- Netty的引入就是对原有的网络传输部分的重构

**Netty优势**
- IO方式由 BIO 改为 NIO, 底层使用了池化技术复用资源
- 可以自主编写 Encoder和Decoder, Serializator等技术, 可扩展性和灵活性高
- 支持多种传输协议, 支持阻塞返回和异步返回
- 等等

### 使用
- 引入 netty 包
```xml
<dependency>  
    <groupId>io.netty</groupId>  
    <artifactId>netty-all</artifactId>  
    <version>4.1.51.Final</version>  
    <scope>compile</scope>  
</dependency>
```

### 流程

**服务端**
- 在服务端, 实现了基于 Netty 的 RpcServer, 该 Server 同样接收一个 service Provicer 本地服务注册中心, 用来查询用户传来请求中的服务
- 在 Server 中, 通过接口 start 启动服务器, 分别创建 `bossGroup` 和 `workGroup` 两个不同的线程组, 前者负责连接的建立, 后者负责请求的处理
- 使用 ServerBootstrap 声明一个服务器并配置. 这里使用的channel类型为 NioServerSocketChannel, 即需要建立的 Socket 为NIO方式的ServerSocket, 处理链为用户自定义处理链
- 配置完成后, 使用 bind 函数指定监听的端口, 并使用 sync 函数, 使服务端线程进入同步阻塞, 直到服务器启动完成; 启动完毕后, 使用 closeFuture 监听ServerSocket 的关闭, 并使用 sync 阻塞等待, 避免线程执行完毕
- `NettyServerInitializer` 中, 使用 `initChannel` 定义 Channel 的 pipeline, 包括包的拆分方式, 长度字段的插入, 编解码方式等, 在处理链的最后加入用户自定义的 Handler, 该 Handler 负责处理用户传入的请求并关闭 Channel
- 在 `NettyRPCServerHandler` 用户自定义 Handler 中, 使用 `channelRead0` 自定义 Channel 处理方式, 该流程中, 会通过 getResponse 解析 Request, 获取 Response 后, 将其通过输出流传给客户端, 并主动关闭本次 channel

**客户端**
- 在客户端, 用户通过 ClientProxy 传入服务器地址并选择使用的代理方式
- ClientProxy中, 将 NettyRpcClient 对象 作为 rpcClient 字段的值, 表示使用该实现进行用户请求的动态代理, rpcClient 接口表示发送远程调用的过程
- 在 NettyRpcClient 中, Bootstrap负责启动客户端, EventLoopGroup 负责与服务端建立连接并处理 IO 操作
- 类加载时, 先初始化 bootstrap 和 eventLoopGroup, eventLoopGroup 的类型为 NioEventLoopGroup, 进行NIO异步操作; 同时配置客户端的 handler 处理器链 NettyClientInitializer
- `sendRequest` 实现中, 先使用 `bootstrap.connect()` 函数创建与服务端之间的 `channel` 连接, 并使用 sync 同步等待连接建立完成, 返回 `ChannelFuture`
- 使用 ChannelFuture 得到建立连接完成的 channel, 通过 channel 向服务端发送 request 请求, 随后等待连接关闭.
- 一旦服务端响应发出, 会通过客户端提前配置好的 `NettyClientInitializer` 中的处理器链进行处理. 经一系列预处理后到达 `NettyClientHandler`
- `NettyClientHandler` 中, 接收到 response 后, 将其存入 channel 的属性中, 并手动关闭当前 channel 连接
- `sendRequest` 中发现 channel 连接关闭后, 从 channel 属性中取出响应信息, 并将 response 作为返回值返回

# Netty服务端

### NettyServerHandler

- 作用: 继承自`SimpleChannelInboundHandler<RpcRequest>`, 作为 Netty 中用于处理客户端请求的处理器, 主要功能是接收来自客户端的 RpcRequest 请求, 并在处理过程中管理连接的生命周期

**为什么需要继承`SimpleChannelInboundHandler<RpcRequest>`**
- 该类是一个通用的 Netty 处理器积累, 为每个事件提供了默认的实现, 避免每次都需要实现 channelRead() 等方法
- `RpcRequest` 表示客户端传入服务端消息的类型, 表示 `NettyRPCServerHandler` 主要用于处理该类型的消息

**channelRead0() 函数**
- `channelRead0()` 是 `SimpleChannelInboundHandler` 抽象类的核心方法, 用于处理接收到的消息, 该方法中用于接收和处理 `RpcRequest` 客户端请求
- 接收到 `RpcRequest` 后, 该函数内部会首先调用一个 `getResponse()` 方法进行处理, 该方法没有变化, 返回值为 Response
- 拿到响应后, 调用 writeAndFlush 将响应结果写入 `Channel`, 并立即发送回客户端, `flush` 的使用意味着消息会立即推送到网络层, 不会被缓存在本地
- 操作完成后, 使用 close 关闭连接

### NettyServerInitializer
- Netty服务端初始化, 相较于 NettyClientInitializer 多出本地注册中心字段

**作用**
- 实现了一个名为 NettyServerInitializer 的类, 该类继承自名为 `ChannelInitializer<SocketChannel>` 的类, 其作用是用来初始化 Channel 和 ChannelPipeline. 
- Netty中, Channel是网络通信的基本单元, `ChannelPipeline` 是用于处理消息的责任链, 包含一系列 `ChannelHandler`, 每个 `ChannelHandler` 负责不同的操作, 如解码, 编码, 获取异常等

**为什么继承 `ChannelInitializer<SocketChannel>`**
- `ChannelInitializer<SocketChannel>` 是一个抽象类, 用于初始化 Channel
- 该类中可以初始化Channel, 配置用于处理网络数据的 ChannelPipeline
- `SocketChannel` 是 Netty 的实现, 通常表示一个 TCP 连接

**initChannel**
- initChannel 方法用于初始化每一个新的 SocketChannel, 每一个 SocketChannel 都会有一个独立的 ChannelPipeline, 用于定义该连接上所有数据的处理流程

**代码详解**
- `LengthFieldBasedFrameDecoder` : 用于处理基于长度字段的帧解码, 用于解决粘包和拆包的问题
- 根据消息头中包含的长度字段解析消息体, 确保读取消息的完整
- `Integer.MAX_VALUE`: 运行最大帧长度
- 第一个 `0,4`: 表示消息中长度字段的位置, 以及长度字段的字节长度
- 第二个 `0,4`: 表示从消息头向后4个字节作为消息体实际的数据部分
```java
pipeline.addLast(  
    new LengthFieldBasedFrameDecoder(Integer.MAX_VALUE,0,4,0,4));
```
- `LengthFieldPrepender` : 发送数据前, 自动计算消息体长度并将其添加到消息头的最前端
- `4` : 设置消息长度字段占前4个字节
```java
pipeline.addLast(new LengthFieldPrepender(4));
```
- `ObjectEncoder` : 将 Java 对象编码成为字节流进行传输
- 使用 Netty 自带序列化机制, 对欲发送对象进行编码
```java
pipeline.addLast(new ObjectEncoder());
```
- `ObjectDecoder` : 解码器, 将字节流解码转换为Java对象
- `ClassResolver`: 接口, 用于提供根据**类名加载类**的策略, 在本段函数中, 使用 `Class.forName(classname)` 的方式动态加载类
```java
pipeline.addLast(new ObjectDecoder(new ClassResolver() {
    @Override
    public Class<?> resolve(String className) throws ClassNotFoundException {
        return Class.forName(className);
    }
}));
```
- `NettyRPCServerHandler` : 用户自定义处理器
- 传入本地服务注册信息, 该Handler根据Channel上下文信息和传入的request参数获取响应并返回
```java
pipeline.addLast(new NettyRPCServerHandler(serviceProvider));
```

### NettyRpcServer

**请求监听**
- 在 Netty 中, 监听请求是通过 ServerBootstrap 和 NioServerSocketChannel 实现的
- ServerBootstrap 是 Netty 中用于启动服务端的核心类, 用于配置和启动 Netty 服务端, `serverBootstrap.group(bossGroup, workGroup)` 将两个线程组配置到 ServerBootstrap 中, 分别用于处理连接请求和处理IO操作
- `channel(NioServerSocketChannel.class)` : 指定服务端使用的 NIO 服务器套接字通道类型, `NioServerSocketChannel` 是 Netty 中专门用于接收**TCP连接**的通道类, 会自动使用 NIO 方式, 处理并发连接时效率高
	- channel代表了一个服务器套接字, 在后台依赖于NIO监听客户端连接
- `bind(port)` 将服务端绑定到指定的端口, 调用该方法后, Netty会启动一个监听套接字, 开始监听客户端的连接请求; 
	- 调用`bind`时, 会创建一个 ServerSocketChannel, 开始监听指定的端口, 等待客户端发起连接, 此时服务器进入监听状态
- `sync()` : 同步方法, 阻塞当前线程直到绑定操作完成, 确保服务器已经开始监听端口

**两个线程组**
- 在Netty中, NioEventLoopGroup用来管理所有的IO操作, 而将线程组分为 bossGroup 和 workGroup, 可以更好的分离**连接接收**和**数据处理**的职责, 提升系统的性能和可扩展性
	- **bossGroup** 负责监听指定的接口, 处理客户端的连接, 并为每一个连接创建一个新的 Channel. 连接的创建是一个轻量级的操作, 通常只涉及到网络 IO 和连接注册; 当客户端连接到服务器时, 该线程组会接管与客户端建立连接的过程, 并将建立好的连接交由 workGroup 进行后续IO处理
	- **workGroup** 负责处理客户端数据的读写操作. 获得了 bossGroup 建立的连接后, workGroup会负责处理网络数据读写和事件的处理, 包括网络读写, 消息编解码, 业务逻辑处理等等
- **并行处理**: 通过分开 workGroup 和 bossGroup, Netty 可以实现更好的**负载均衡**和**并行处理**. bossGroup专注于连接的接收, workGroup专注于连接处理. 由此, bossGroup 和 workGroup 可以同时工作, 提高服务器并发性能
- **线程分配**: 每个 `NioEventLoopGroup` 实际上是由多个线程组成的. 由于建立连接的频率相对较低且建立连接速度较快, bossGroup的线程相对较少. 由此, 可以将 workGroup 中分配较多线程, 以处理每个连接的大量数据传输操作, 从而避免了某一组处理过程成为整个系统的性能瓶颈

# Netty客户端

### 简单类

##### RpcClient
- 通过 Request 获取 Response 的共性方法的抽象

##### SimpleSocketRpcClient
- 简单的客户端实现, 为Part1中的 RpcClient 实现

**ClientProxy**
- 添加名为 choose 的参数, 表示使用的请求发送方式
- socket为BIO发送方式, netty为NIO发送方式

### NettyClientHandler
- 作用: 继承自 `SimpleChannelInboundHandler` 类, 用于处理服务端返回的 RpcResponse 类型数据, 是服务器响应的处理器

**channelRead0()作用**
- 是 SimpleChannelInboundHandler 的核心方法, 负责处理接收到的消息
- `ChannelHandlerContext` 表示每一个 Netty 处理器的上下文对象, 代表了当前执行 IO 操作的环境
- `AttributeKey.valueOf("RPCResponse")` : 创建一个 AttributeKey, 用来存储和检索 Channel 上的属性, 相当于为 Channel 设置别名
- `ctx.channel()` 获取当前上下文环境中的 channel
- `ctx.channel().attr(key).set(response)` : 将接收到的响应信息作为 channel 的一个属性值存入 channel, 确保在后续对 channel 的处理过程中可以根据用户指定的属性名获取到对应的 Response 信息
- `ctx.channel().close()` : 接收到响应后, 关闭当前 channel 的网络连接

### NettyClientInitializer
- 继承自 `ChannelInitializer<SocketChannel>` 抽象类, 用于初始化客户端 Channel 和 ChannelPipeline. 
- ChannelPipeline 是一个用于处理 Channel内部消息的责任链, 内部包含了一系列用于处理消息的 ChannelHandler

**initChannel**
- 略, 同 `NettyServerInitializer` 中的 `initChannel`

### NettyRpcClient

**bootstrap**
- Netty 的客户端启动类, 用来表示客户端的开启
- 初始化时, 通过传入的参数指定用到的线程组以及处理 Channel 的类型, 并通过 handler() 设置 ChannelPipeline

**eventLoopGroup**
- 事件处理线程组, 负责处理网络通信 IO 操作
- 类型为 NioEventLoopGroup, 表示使用 NIO 进行异步非阻塞通讯, 提高了网络通信过程中的 IO 效率和系统利用率, 适合客户端与服务端的网络通信

**netty客户端初始化**
- 客户端初始化主要是为了配置 Netty 进行网络通信中需要的各类资源, 使客户端能够正确的向服务器建立连接, 发送请求, 处理响应
- **NioEventLoopGroup** : NIO方式的Netty事件循环组, 负责管理所有的IO线程, 每个线程负责一个或者多个 Channel 的IO操作
- **Bootstrap**: Netty客户端的启动类, 用于设置与服务端连接的相关配置, 如所用IO线程池, 连接类型, 消息处理器等, 并以此建立与服务端的连接

**sendRequest**
- 试图建立一个 Channel 连接, 使用 sync() 使操作阻塞直至操作完成或失败
- 若建立连接失败, 直接抛出异常, 不再继续向下执行
```java
ChannelFuture channelFuture  = bootstrap.connect(host, port).sync();
```
- 获取 channelFuture 中存储的 channel 类型变量, 并发送请求对象
```java
Channel channel = channelFuture.channel();  
channel.writeAndFlush(request);
```
- 使用 sync 阻塞等待 channel 关闭
- 服务端获取到 Request 后, 返回处理得到的 Response
- 客户端 channel 读取到 response 后, 经过 pipeline 的层层channel, 最终到达用户自定义的 NettyClientHandler
- NettyClientHandler 中将 response 存入 channel 的属性中, 并关闭 channel, 释放 channel 建立的连接等, 但不会销毁 channel
```java
channel.closeFuture().sync();
```
- 获取名称为 `RPCResponse` 的Channel属性, 即服务端响应, 并返回该响应
```java
AttributeKey<RpcResponse> key = AttributeKey.valueOf("RPCResponse");  
RpcResponse response = channel.attr(key).get();
return response;
```


