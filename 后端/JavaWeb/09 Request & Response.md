- Request(请求): **获取**请求数据
- Response(响应): **设置**响应数据

# Request

## Request继承体系

ServletRequest --> Java提供的请求对象根接口
↑
HttpServletRequest --> Java提供的对Http协议封装的请求对象接口
↑
RequestFacade --> Tomcat定义的HttpServletRequest实现类

1. Tomcat想要解析请求数据, 封装成为request对象, 并且创建request对象传递到service方法中
2. 使用request, 只需要查阅JavaEE提供的API文档中的HttpServletRequest接口

## Request获取请求数据

### 获取请求数据
- 请求对象分为三个部分
1. 请求行:
	- `getMethod` - 获取请求方式
	- `getContextPath` - 获取虚拟目录(项目访问路径) 
	- `getRequestURL` - 获取URL(统一资源定位符)(`?`前为URL, 后为请求参数)
	- `getRequestURI` - 获取URI(统一资源标识符)
	- `getQueryString` - 获取GET方式请求参数
2. 请求头: 
	- `getHeader(String name)` - 根据请求头名称获取值
3. 请求体:
	- `getInputStream` - 获取字节输入流
	- `getReader` - 获取字符输入流

### 通用方式获取请求参数

请求参数获取方式
- GET方式
	- `getQueryString`
- POST方式
	- `getReader`

可以在POST方法中调用GET方法来简化操作

Request解析参数采用`Map<String, String[]>`进行存储
- `getParameterMap()` : 获取所有参数Map集合
- `getParameterValues(String name)` : 根据名称获取参数值(数组)
- `getParameter(String name)` : 根据名称获取参数值(单个值)

### Servlet模板创建Servlet
- 可以在Setting中修改Servlet的模板使其更加好用

### 请求参数中文乱码

**只在Tomcat 7及以下版本才存在该问题**

#### POST乱码
- POST底层通过getReader获取数据, 读取到数据流的解码方式默认是ISO-8859-1, 所以乱码发生

**解决**
- `setCharacterEncoding("UTF-8")` : 设置字符输入流的编码为UTF-8
- 应该写在最上面

#### GET乱码
- GET获取参数方式: `getQueryString`
- 不支持中文的浏览器会将中文转换为URL编码, 发送给Tomcat后对其进行URL解码, 但是浏览器使用UTF-8编码, Tomcat使用ISO-8859-1解码, 导致了乱码的发生. 但是Tomcat没有提供设置为UTF-8的解码方式
- **乱码原因:** Tomcat默认字符集为ISO-8859-1

**URL编码**
1. 将字符串按编码方式转为2进制
2. 每个字节转为2个16进制数并在前面加上`%`

- 编码工具类: `java.net.URLEncoder` - `encode(str, "utf-8")`
- 解码工具类: `java.net.URLDecoder` - `decode(str, "utf-8")`

**解码过程**
1. 编码: 转换为字节数据 - `str.getBytes("ISO-8859-1")[可以使用联想]`
2. 解码: 用字节数组构建字符串 - `new String(bytes, "utf-8");`
3. 合在一起: `username = new String(username.getBytes(StandardCharsets.ISO_8859_1),"utf-8")`
**这个代码也可以解决POST乱码问题**

#### Tomcat8
**Tomcat8以后已经解决了中文乱码问题**

## Request请求转发

- 请求转发(forward) : 一种在服务器内部的资源跳转方式
- 实现方式 : 
	- `req.getRequestDispatcher("资源b路径").forward(req, res);`

![[后端/JavaWeb/Inbox/Pasted image 20240201213001.png]]

### 请求转发间共享数据
- `setAttribute(name, obj)` - 存储数据到request域
- `getAttribute(name)` - 根据key获取值
- `removeAttribut(name)` - 根据key删除键值对

### 请求转发特点
- 浏览器地址栏路径不发生变化
- 只能转发到服务器的内部资源
- 一次请求可以在转发的资源间使用request共享数据

# Response

## Response继承体系

ServletResponse --> Java提供的请求对象根接口
↑
HttpServletResponse --> Java提供的对Http协议封装的请求对象接口
↑
ResponseFacade --> Tomcat定义的HttpServletResponse实现类

## Response设置响应数据功能

- 响应数据分为3部分
1. 响应行
	- `setStatus(int sc)`: 设置响应状态码
2. 响应头
	- `setHeader(String name, String value)`: 设置响应头键值对
		- `content-type` - 文本类型 : `text/html` - 以html格式解析文本
1. 响应体
	- `getWriter`: 获取字符输出流
	- `getOutpetStream`: 获取字节输出流

## Response完成重定向

![[后端/JavaWeb/Inbox/Pasted image 20240201212943.png]]

### 重定向实现
- 重定向(Redirect) : 一种资源跳转方式
	- 状态码 - 302
	- 响应头 - location xxx
- 实现方式
	- `setStatus(302)` - 设置状态码
	- `setHeader("location", "资源b路径")` - 设置响应头
- 简化方式实现重定向
	- `sendRedirect("资源路径")`

### 重定向特点

- 浏览器地址栏路径发生变化
- 可以重定向到任意位置资源
- 两次请求, 不能在多个资源使用Request共享数据

### 路径问题

**路径谁使用**
- 浏览器使用: 需要加虚拟目录
- 服务端使用: 不需要加虚拟目录
- 动态虚拟路径: `getContextPath()`

## Response响应字符数据

**使用**
1. 使用Response对象获取字符输出流
2. 写数据
3. 使用完毕自动销毁, 不需要手动关闭

- **解决中文乱码问题**
- `response.setContentType("text/html;charset=utf-8");`
	- 设置`context-type`头
	- 设置流的编码
	- 应该放到流的前面

## Response响应字节数据

**使用**
1. 使用Response获取字节输出流
2. 写数据

- 在浏览器中的img中使用Servlet作为url路径时, 浏览器会自动将Servlet的输出流作为图片进行解析
- 如果需要传递多张图片, 需要进行打包处理.

**IOUtils工具类的使用**
1. 导入坐标
2. 使用

**依赖**
```xml
<dependency>  
    <groupId>commons-io</groupId>  
    <artifactId>commons-io</artifactId>  
    <version>2.6</version>  
</dependency>
```

**Java代码**
```java
// 1.读取文件  
FileInputStream fileInputStream = new FileInputStream("src/main/resources/princess.jpg");  
  
// 2.获取response字节输出流  
ServletOutputStream outputStream = response.getOutputStream();  
  
// 使用工具类完成流的copy  
IOUtils.copy(fileInputStream, outputStream);  
  
fileInputStream.close();
```









