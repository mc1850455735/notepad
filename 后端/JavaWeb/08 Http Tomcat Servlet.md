# Http

## 概念
`Hyper Text Transfer Protocol`, 超文本传输协议, 规定了浏览器和服务器之间数据传输的规则

## 特点
1. 基于TCP协议 : 面向连接, 安全
2. 基于请求-响应模型 : 一次请求对应一次响应
3. HTTP协议时无状态的协议: 对于事务处理没有记忆能力. 每次请求-响应都是独立的
	- 缺点: 多次请求间不能共享数据
		- 解决: 使用会话技术(Cookie, Session)解决
	- 优点: 速度快

## 请求数据格式

```
GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Cache-Control: max-age=0 ...
```

1. 请求行: 请求数据的第一行
	- `GET`表示请求方式, `/` 表示请求资源, `HTTP/1.1`表示协议版本
2. 请求头: 第二行开始, 格式为key : value形式
3. 请求体: `POST`请求的最后一部分, 存放请求参数; 只有`POST`请求有

**常见Http请求头**
- Host: 请求的主机名
- User-Agent: 浏览器版本
- Accept: 浏览器能接收的资源类型, 如`text/*`, `image/*`或`*/*`表示所有类型
- Accept-Language: 浏览器偏好的语言, 可以据此返回不同语言的网页
- Accept-Encoding: 浏览器支持的压缩类型

**GET请求和POST请求区别**
1. GET请求请求参数在请求行中, 没有请求体; POST请求参数在请求体中
2. GET请求请求参数大小有限制, POST没有

## 响应数据格式

```
HTTP/1.1 200 OK
Server: Tengine
Content-Type: text/html
Transfer-Encoding: chunked
...

<html>
<head>
	<title></title>
</head>
<body></body>
</html>
```

1. 响应行: 响应数据第一行. `HTTP/1.1`表示协议版本, `200`表示响应状态码, `OK`表示状态码描述
2. 响应头: 第二行开始, 格式为`key` : `value`形式
3. 响应体: 最后一部分. 存放响应数据

**常见Http响应头**
- Content-Type : 响应内容的类型
- Content-Length : 响应内容长度(字节数)
- Content-Encoding : 响应压缩算法
- Cache-Control : 指示客户端应该如何缓存, 如max-age=300, 最多缓存300s

**状态码分类**
 - 1xx - 响应中: 临时状态码, 表示请求已接收, 告诉客户端应该继续请求或已完成则忽略(不常见)
 - 2xx - 成功: 表示请求已经被成功接收, 处理已完成
 - 3xx - 重定向 : 重定向到其他地方, 让客户端再发起一个请求以完成整个处理
 - 4xx - 客户端错误 : 处理发生错误, 责任在客户端, 如: 客户端请求不存在资源
 - 5xx - 服务端错误 : 处理发生错误, 责任在服务端, 如: 服务端抛出异常等
[状态码大全](https://cloud.tencent.com/developer/chapter/13553)

**常见响应状态码**
- <span style="color:red">200</span> - `OK`, 客户端请求**成功**且服务端处理成功
- 302 - `Found`, 请求资源已移动到Location响应头给定的url, 浏览器会自动重新访问
- 304 - `Not Modified` : 请求的资源自上次成功取得后, 服务端未更改, 直接使用本地缓存. 隐式重定向
- 400 - `Bad Request` : 客户端**请求有语法错误**. 不能被服务器理解
- 403 - `Forbidden` : 服务器收到请求, 但是**拒绝提供服务**, 如:没有权限访问相关资源
- <span style="color:red">404</span> - `Not Found` : 请求**资源不存在**, url输入错误或请求资源已被删除
- 428 - `Precondition Required` : **服务器要求有条件的请求**, 想要访问该资源必须携带特定的请求头
- 429 - `Too Many Request` : **太多请求**, 可以限制客户端请求某个资源的数量, 配合`Retry-After(多长时间后重新尝试)`响应头一起使用
- 431 - `Request Header Fields Too Large` : **请求头太大**, 服务器拒绝处理该请求, 可在减少请求头大小后重新提交
- 405 - `Method Not Allowed` : **请求方式有误**, 比如改用GET时用POST
- <span style="color:red">500</span> - `Internal Server Error` : 服务器**发生不可预期的错误.** 去看服务器**日志**
- 503 - `Service Unavailable` : **服务器尚未准备好处理请求**
- 511 - `Network Authentication Required` : 客户端需要进行**身份验证**才能获得网络访问权限

# Tomcat

- web服务器是一个应用程序, 对Http协议的操作进行封装, 使程序员不必直接对协议进行操作, 使web开发更加便捷.

## 简介

**概念**
- Tomcat是Apache基金会的核心项目, 是一个开源免费的**轻量级**Web服务器, 支持Servlet/JSP少量JavaEE规范.
- JavaEE: Java Enterprise Edition, Java企业版. 指Java企业级开发的技术规范总和. 包含13项技术规范:
	- JDBC, JNDI, EJB, RMI, JSP, Servlet, XML, JMS, Java IDL, JTS, JavaMail, JAF
- Tomcat也被成为Web容器, Servlet容器. Servlet需要依赖于Tomcat才能运行

## 基本使用

### 下载
[Apache Tomcat® - Welcome!](https://tomcat.apache.org/)

> 建议使用`tomcat 8.5.x`

### 启动
双击`bin`目录下`startup.bat`

### 解决乱码
在`conf/logging.properties`中修改java.util.logging.ConsoleHandler.encoding = GBK

### 关闭
1. 直接关闭运行窗口 - 强制关闭
2. `bin\shutdown.bat` - 正常关闭
3. 窗口中Ctrl + C - 正常关闭

### 配置
1. 修改启动端口号 - conf/server.xml: ` <Connector port="8080" protocol="HTTP/1.1"`
	- 注: Http默认端口号为80, 如果设置为80则以后启动时不需要输入端口号
	- Https协议端口号为443

### 启动时的问题
1. 端口号冲突: 找到对应应用程序, 关掉
2. 启动窗口一闪而过: 检查JAVA_HOME配置

###  部署项目
- 将项目放置到webapps目录下, 即部署完成
- 一般Javaweb项目会被打成war包, 然后将war包放到webapps目录下, Tomcat会自动解压缩war文件

## IDEA中创建Maven Web

### Web项目结构

- Maven Web项目结构: 开发中的项目
	- src - main
		- java: 源代码
		- resources: 资源文件
		- webapp: web项目特有目录
			- html: html目录文件, 可自定义
			- WEB-INF: 固定名称, 放置web项目配置文件web.xml
	- pom.xml: packaging改为war(默认为jar)

- JavaWeb可部署项目
	- 项目名: 项目路径
		- html: html目录文件, 可自定义
		- WEB-INF: web项目核心目录
			- classes: 字节码文件
			- lib: 库文件
			- web.xml: web项目配置文件

- 编译后的字节码文件和resources的资源文件, 放到WEB-INF的classes目录下
- pom.xml中依赖坐标对应的jar包放到WEB-INF的lib目录下

### Maven创建Web项目

- 使用骨架 - 项目模板
1. 选择web项目骨架, 创建项目
2. 删除pom.xml中多余项目
3. 补齐缺失的目录结构

- 不使用骨架
1. 创建无骨架的maven项目
2. 在pom.xml中将`<packaging>`修改为war并更新
3. 在Project Structure中的Facets中, 添加webapp
4. 同样在Facets中, 添加web.xml并拖动到webapp中

## IDEA中使用Tomcat

- 将**本地Tomcat集成到Idea**, 然后进行项目部署
1. Add Configuration -> + -> Tomcat Server
2. 修改Application server找到tomcat路径, 设置jre版本
3. 在Deployment中 -> + -> Artifact -> 选择对应war包

- **使用Tomcat Maven插件**
1. 在pom.xml中添加tomcat插件
```xml
<build>  
    <plugins>  
        <!-- tomcat插件 -->  
        <plugin>  
            <groupId>org.apache.tomcat.maven</groupId>  
            <artifactId>tomcat7-maven-plugin</artifactId>  
            <version>2.2</version>  
            <configuration>  
			    <port>80</port>  
			    <path>/</path>  
			</configuration>
        </plugin>  
    </plugins>  
</build>
```
2. 使用Maven Helper中打开tomcat
3. 可以使用`<configuration>`设置tomcat启动配置
	- `<port>` - 端口
	- `<path>` - 启动路径

# Servlet

- Servlet是Java提供的一门**动态**web资源开发技术
- Servlet是JavaEE提供的规范之一, 其实就是一个接口, 将来需要定义Servlet类实现Servlet接口, 并由web服务器运行Servlet

## 快速入门

1. 创建web项目, 导入Servlet依赖坐标
	- 依赖范围必须修改成`provided` , web项目中不需要这个jar包, 因为Tomcat中自带这个jar包
```xml
<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.1.0</version>  
    <scope>provided</scope>  
</dependency>
```
2. 创建: 定义一个类, 实现**Servlet接口**, 并重写所有接口方法, 并在service方法中输入一段话
3. 配置: 在类上使用@WebServlet注解并配置该Servlet访问路径
4. 访问: 启动tomcat以访问Servlet


## 执行流程

对于网址
localhost:8080/web-demo1/demo1
分别对应
web服务器/web项目/Servlet对象

在访问该Servlet时, Tomcat会自动创建该Servlet的对象, 并且会自动调用service方法. 

## 生命周期

- 对象的生命周期指一个对象从创建到销毁的整个过程
- Servlet运行在Servlet容器(web服务器)中, 其生命周期由容器管理, 分为4个阶段
	1. 加载和实例化: 默认情况下, Servlet第一次被访问时, 容器创建Servlet对象
		1. 在@WebServlet注解中的loadOnStartup调整优先级
			- 负整数: 第一次被访问时创建
			- 0 / 正整数: 服务器启动时创建Servlet对象, 数字越小优先级越高
			- `@WebServlet(urlPatterns = "/demo2", loadOnStartup = 1)`
	2. 初始化: Servlet实例化后, 会自动调用init方法初始化对象, 该方法**只调用一次**.
	3. 请求处理: **每次**请求Servlet时, 容器就会调用service方法对请求进行处理.
	4. 服务终止: 当释放内存或容器关闭时, 容器会调用destroy方法完成资源释放. 容器会释放Servlet实例, 该实例随后会被Java垃圾收集器回收


## 方法介绍

- 初始化方法
`void init(ServletConfig config)`
- 提供服务方法
`void service(ServletRequest req, ServletResponse res)`
- 销毁方法
`void destory()`
- 获取ServletConfig对象
`ServletConfig getServletConfig()`
- 获取Servlet信息
`String getServletInfo()`

## 体系结构

Servlet <- GenericServlet <- HttpServlet
HttpServlet : 对HTTP协议封装的Servlet实现类
所以将来开发的Servlet一般会继承自HttpServlet

- Servlet中service方法在http协议中需要根据请求方式不同进行分别的处理
- HttpServlet的get和post方法相当于进行了集成.

## Servlet urlPattern配置
- Servlet想要被访问, 必须配置访问路径(urlPattern)

**特点**
1. 一个Servlet可以配置多个urlPattern
	- `@WebServlet(urlPatterns = {"/demo7", "/demo8"})`
2. urlPattern匹配规则
	1. 精确匹配
	2. 目录匹配
	3. 扩展名匹配
	4. 任意匹配

**优先级**
`精确路径 > 目录路径 > 扩展名路径 > /* > /`

- 精确匹配
	- 配置路径: `@WebServlet("/user/select")`
	- 访问路径: xxx/user/select
- 目录匹配
	- 配置路径: `@WebServlet("/user/*")`
	- 访问路径: xxx/user/不限
- 扩展名匹配
	- 配置路径: `@WebServlet("*.do")`
	- 访问路径: `xxx/任意/任意.do`
	- **配置路径不能以 `/` 开头**
- 任意匹配
	- 配置路径: `@WebServlet("/") / @WebServlet("/*")`
	- 访问路径: `xxx/任意`
	- 优先级: `/*` 优先级高于 `/`
	- `/` 和 `/*` 的区别
		- 当项目中的Servlet配置了`/`, 会覆盖掉tomcat中的**DefaultServlet**, 当其他的url-pattern匹配不上时会走这个Servlet
			- DefaultServlet被覆盖后, 会无法访问到静态资源
		- 当项目中配置了`/*`, 意味着匹配任意访问路径
	- 不建议使用 `/` 或者 `/*`

## XML配置方式编写Servlet

- Servlet3.0后支持使用注解配置, 3.0前只支持XML配置文件的配置方式
- 步骤
	1. 编写Servlet类
	2. 在web.xml配置该Servlet

```xml
<!-- 使用xml配置Servlet -->  
<!-- Servlet全类名 -->  
<servlet>  
    <servlet-name>demo13</servlet-name>  
    <servlet-class>top.majinliang.ServletDemo13</servlet-class>  
</servlet>  
<!-- Servlet访问路径 -->  
<servlet-mapping>  
    <servlet-name>demo13</servlet-name>  
    <url-pattern>/demo13</url-pattern>  
</servlet-mapping>
```












