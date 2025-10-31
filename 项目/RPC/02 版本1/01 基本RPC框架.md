### 概述

##### 服务端执行流程
- 服务端启动, 获取用户服务的实例, 并将服务对象加入 `serviceProvider` 中
- 使用 `SimpleRPCRPCServer` 进行监听, 监听某端口处的请求信息
- 接收到请求后, 创建新的线程, 传入 serviceProvider 和套接字对象
- 从套接字中获取对象输入流和对象输出流
	- 对象输入流用于获取传入的对象
	- 对象输出流用于将处理好的响应信息输出
- 获取传入对象请求的服务接口名, 在 `serviceProvider` 中根据接口名获取对应的服务实现类
- 获取传入对象调用的方法名以及参数类型列表, 根据二者信息, 得出调用者欲调用的实例对象方法
- 获取传入对象的参数值列表, 使用反射调用方法, 返回方法的执行结果
- 通过对象输出流输出返回的响应结果信息

##### 客户端执行流程
- 创建 `ClientProxy` 并使用其中的 `getProxy` 方法创建动态代理类
- `getProxy` 方法中使用了 `Proxy.newProxyInstance` , 为传入的clazz创建了代理类对象, 并设置了其 `invoke`
- `invoke` 中, 通过 `RpcRequest` 的 `build` 方法构建对象, 传入远程调用的各类信息, 并通过 `IOClient.sendRequest` 发送请求, 获取响应并返回
- `IOClient` 负责与服务端的通信, 发送 `RpcRequest` , 返回 `RpcResponse`
- 在 `IOClient.sendRequest` 中, 传入地址 `Host` , 端口号 `Port` 和要发送的 `RpcRequest`. 使用 host 和 port 创建一个 socket, 获取socket的输入流和输出流并将其包装为对应的对象流.
- 向对象输出流中写入对应的 `RpcRequest`, 并从对象输入流中获取远程发送来的响应对象 `RpcResponse`

### 公共部分

##### User

**注解作用**
- `@Builder` : Lombok提供, 为类自动提供一个 Builder模式的实现, 允许使用链式编程构建 User 对象, 从而避免了构造函数过多时的麻烦, 提高代码可读性
- `@Data`: 略
- `@NoArgsConstructor`: 略
- `@AllArgsConstructor`: 略

**Serializable接口**
- User类是一个普通的POJO类
- 通过实现 Serializable 接口, 允许对象转换流识别该类并将其转换为字节流以便于保存和传输, 并允许将其从字节流中恢复
- 序列化是分布式应用中不可或缺的组成部分

##### UserService

**组成**
- `UserService`: 定义远程调用和本地调用时所需要的服务接口
- `UserServiceImpl`: 根据 `UserService` 接口创建对应的实现类

##### RpcRequest
- **字段组成**: 服务接口名, 方法名, 参数列表, 参数类型列表
- 由于需要使用动态代理, 所以服务最好是以接口的方式提供和呈现

##### RpcResponse
略

### 客户端

##### IOClient

**概述**
- 建立连接, 发送请求, 接收并返回响应, 异常处理

**解释**
- 首先, 根据传入的 host 和 port 建立 socket 套接字连接
- 使用 `ObjectOutputStream` 包装 socket 的输入流, 使客户端传入的请求对象可以通过对象转换流转为二进制流输出
- 使用 `ObjectInputStream` 包装 socket 的输出流, 使服务端返回的响应可以通过对象转换流转为响应对象
```java
Socket socket = new Socket(host, port);  
ObjectOutputStream oos = new ObjectOutputStream(socket.getOutputStream());  
ObjectInputStream ois = new ObjectInputStream(socket.getInputStream());
```

##### ClientProxy

**InvocationHandler接口**
- JDK动态代理的核心就是通过反射生成一个代理对象, 该对象可以代理一个或者多个接口的方法调用
- 当代理对象的对应方法被调用时, 实际上的调用方法为该接口中的 invoke 方法, 而只有添加了这个接口, 才能进行 JDK 动态代理的实现
- 使用这种方式最终实现类的逻辑增强, 隐藏远程调用的过程
- **核心功能**:
	- 封装请求: 将传递的参数封装成 `RpcRequest` 对象
	- 网络通信: 调用 `IOClient.sendRequest` 进行网络通信
	- 返回结果: 将远程服务数据返回给调用者

**invoke方法的参数与作用**
- 每次调用接口中的方法时, 实际上是将方法名和方法参数等信息传入了 invoke, 由 invoke 接收方法的调用
- `proxy`: 代理对象本身; `method`: 被调用的方法; `args`: 传入的参数

**getProxy**
- 利用 JDK 动态代理机制, 动态生成一个接口的代理对象
- `Proxy.newProxyInstance(clazz.getClassLoader(), new Class[]{clazz}, this)`
- `clazz.getClassLoader()`: 获取当前类的类加载器
- `new Class[]{clazz}`: 创建当前类的实例数组, 用于生成代理对象
- `this`: 作为 `InvocationHandler` 的实现类对象传入, 处理所有方法调用

##### TestClient
略

### 服务端

##### RpcServer
- 接口, 定义了 `start` 和 `stop` 两个方法, 用来指明服务的开启和关闭

##### ServiceProvider
- 该类提供了一种简单的方式来注册和获取服务, 其本地维护一个 Map 类型的集合, 将服务类的全限定名和服务对象实例进行了一个映射, 以便于注册服务和快速获取服务
- 其内部存在两个方法:
	- `provideServiceInterface` : 传入一个对象, 该函数内部会获取该对象的所有实现接口, 并将其遍历注册到内部维护的 Map 中
	- `getService` : 根据接口全限定名, 获取其对应的实现类对象
- Map中, key为接口全限定名, value为实现了该接口的类对象引用

##### WorkThread

**概述**
- Thread类, 实现了 Runnable 接口, 用于处理客户端请求并返回对应响应
- 核心作用是是西安多线程环境中对客户端接收请求并返回服务结果

**为什么实现Runnable接口**
- 通过实现 Runnable 接口, 使该类允许作为线程类对象被创建和使用
- 在 `SimpleRPCRPCServer` 中, 直接将 WorkThread 作为线程类对象的参数传入
- 在 `ThreadPoolRPCRPCServer` 中, 则作为 `threadPool.execute` 的参数传入, 既为线程池中的线程分配一个任务对象
- 如果没有 Runnable 接口, 则无法将 WorkThread 交由线程池进行管理, 也无法将其作为一个 Thread 类构造函数参数传入

**Socket字段**
- 用于与客户端进行网络通信. Socket 是Java中用于进行网络通信的类, 表示一个TCP连接端点, 客户端和服务端都会通过 Socket进行数据的传输
- 在 WorkThread 中, socket 表示了服务端与某一特定客户端之间的连接
- **接收请求**: 使用 `socket.getInputStream()` 读取客户端发送过来的数据流
- **发送数据**: 使用 `socket.getOutputStream()` 获取输出流并以此写出响应数据

**serviceProvide**
- 一个简易的本地服务注册中心, 负责管理本地服务的注册和查找, 根据接口名称获取对应的服务实现对象
- 通过该变量, 在客户端请求服务时, 只需要根据请求的接口名称, 从注册中心中获取对应的实例即可
- 一旦获取到服务实例, 就可以通过反射根据实例方法名称调用实例方法, 处理请求

**getResponse**
- 该函数中, 首先会根据传入的 request 对象获得一系列字段, 如需要调用的接口名称, 接口方法名称, 传入方法的参数列表, 参数类型列表等信息
- 根据接口名称, 在 `serviceProvide` 简易服务注册中心中获取接口的实现实例
- 根据传入方法名和参数类型列表的, 通过反射获取到该实例方法 `Method` 对象
- 使用 Method 类的 invoke 函数, 传入接口实现实例和参数列表, 获取服务的返回值, 并将其封装在 `Response.success()` 中进行返回
- 最后使用 catch 捕获可能出现的异常, 并返回 `Response.fail()`

##### SimpleRPCServer

**作用**
- 实现了 RpcServer 接口, 用于启动一个简单的 RPC 服务器, 监听用户的连接, 处理客户端请求, 并通过创建线程的形式并发处理每一个连接

**ServerSocket**
- 该类用于监听指定端口上的客户端连接请求, 是一个 TCP 服务端的类, 负责接收客户端的连接并为每一个连接创建一个 Socket 对象
- 通过 `serverSocket.accept()` , 线程会阻塞等待一个连接的到来, 一旦监听到连接, 就返回一个 Socket 类型的对象用于处理请求和响应

##### ThreadPoolRPCRPCServer

**作用**
- 实现了 RpcServer 接口, 该实现通过线程池的方式管理和执行请求处理任务, 以此提高并发处理能力

**优势**
- 相比较于 SimpleRPCServer 直接创建线程执行的方式, 线程池实现方式减少了线程创建和销毁时的开销, 避免了性能问题

**threadPool**
- threadPool 是一个 ThreadPoolExecutor 类型的字段, 表示当前服务器管理的线程池, 通过该线程池管理和执行客户端的请求, 避免线程的频繁创建和销毁

**构造函数**
- 构造函数中, 既可以只设置 serviceProvider, 既服务注册中心, 也可以提供一系列线程池设置, 用来调节各类型线程池参数



