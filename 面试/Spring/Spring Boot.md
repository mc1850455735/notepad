
# Spring Boot 的启动流程是什么？

对于 Spring Boot 的启动，主要是以下流程：

首先，Spring Boot 的启动从包含 `@SpringBootApplication` 的类的 main() 方法开始。对于 @SpringBootApplication 注解，包含三个注解：
- `@SpringBootConfiguration`：表示该类是一个配置类
- `@EnableAutoConfiguration`：启动自动配置功能
- `@ComponentScan`：启动组件扫描，默认扫描当前包及其子包下的 Spring 组件。

对于 `main()`，其中会包含 `SpringApplication.run()` 方法，该方法会调用 Spring 容器的构造函数，创建一个 SpringApplication 对象，并执行该对象的 `run()` 方法。

首先是构造函数部分，Spring Boot 会传入包含 `@SpringBootApplication` 的类的字节码文件作为配置来源，并传入 main 方法的参数作为容器创建依据。
SpringBoot 会根据类路径中的依赖和配置进行初始化，并根据自动扫描的类路径进行应用类型的推断。
如果类路径中存在 `javax.servlet.Servlet`，说明这是一个 `Servlet` 应用；如果存在 `org.springframework.web.reactive.DispatcherHandler`，说明这是一个 `Reactive` 应用；如果都没有，说明不包含 Web 服务器，仅适用于普通的非 Web 应用。

对于 run() 方法，其中完成了 SpringBoot 的剩余启动逻辑，具体流程总结如下：
**基础准备**
- 记录应用的启动开始时间，用于统计整个启动过程的耗时；
- 创建一个用于启动过程中共享对象的上下文，提供基础的启动支持；
- 配置 `java.awt.headless`，用于没有显示器的服务器环境；
- 获取启动监听器，监听应用的各个生命周期事件；
- 通知所有监听器，应用开始启动；
**启动应用**
- 对传入的命令行参数进行解析，成为一个参数对象；
- 根据配置文件、环境变量和命令行参数对象，生成当前运行的环境；
- 根据环境配置决定是否忽略 BeanInfo 类以加快启动速度；
- 打印启动时的 Banner；
- 通过 **`createApplication()` 方法**创建 `ApplicationContext`，并根据先前解析的应用类型，使用 `context.setApplicationStartUp()` 方法对其进行设置；
- 将前期获得的环境信息、监听器、启动参数等注入上下文，并进行应用上下文的准备工作；
- 使用 **`refreshContext()` 方法**刷新 Spring 容器，完成 Bean 的创建和初始化；
- 使用 **`afterRefresh()` 方法**刷新上下文后的回调，执行自定义逻辑；
- 记录应用的启动耗时；
- 通知应用启动完成；
- 调用实现了 CommandLineRunner 或者 ApplicationRunner 接口的 Bean，执行启动后的初始化逻辑；
- 发布 ApplicationReadyEvent 事件，表示应用启动完成；

在应用启动过程中，会捕获异常并进行处理。




