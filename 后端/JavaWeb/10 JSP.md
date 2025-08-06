# 概念

- Java Server Page : Java服务端页面
- 一种动态网页技术, 其中既可以定义HTML, JS, CSS静态内容, 也可以定义Java代码的动态内容.
- JSP : HTML + Java

**作用**
简化开发, 避免了在Servlet中直接使用标签

## 快速入门

1. 导入JSP坐标
```xml
<dependency>  
  <groupId>javax.servlet.jsp</groupId>  
  <artifactId>jsp-api</artifactId>  
  <version>2.2</version>  
  <scope>provided</scope>  
</dependency>
```

2. 创建JSP文件
```jsp
<%@ page contentType="text/html;charset=utf-8" language="java" %>  
<html>  
<head>  
    <title>title</title>  
</head>  
<body>  
    <h1>JSP, Hello world</h1>  
    <%  
        System.out.println("hello jsp!");  
    %>  
</body>  
</html>
```

3. 编写代码

## 原理

- JSP本质上就是一个Servlet
- jsp会自动转换为java文件, 并编译成为class文件

编译后的java以及class可以在`\jsp-demo\target\tomcat\work\Tomcat\localhost\jsp-demo\org\apache\jsp`中查看
每次**重新访问后**Java代码刷新

# JSP脚本

- JSP脚本用于在JSP页面定义Java代码

## JSP脚本分类
1. `<% ... %>`  - 内容直接放到_jspService()方法中
2. `<%= ... %>` - 内容放到out.print中, 作为其参数
3. `<%! ... %>` - 内容放到_jspService()方法外, 被类直接包含, 作为类的成员

**技巧**
- JAVA代码在JSP脚本中是可以截断的

# JSP缺点

- 书写麻烦, 尤其是复杂页面
- 阅读麻烦
- 复杂度高, 运行需要依赖于各种环境: JRE, JSP容器, JavaEE...
- 占用内存和磁盘: JSP会自动生成.java和.class文件, 且运行.class占内存
- 调试困难: 出错时需要找到生成的.java文件调试
- 不利于团队合作
- ....

**JSP已经逐渐退出历史舞台**

## JavaWeb演化过程

`Servlet` -> `JSP` -> `Servlet+JSP` -> `Servlet+html+ajax`

**问题**
JSP中的Java代码难以维护
**解决**
- Servlet负责数据的处理
- JSP负责获取与展示遍历数据

# EL表达式

- Expression Language 表达式语言, 用于简化JSP页面内的Java代码
- 主要功能 : 获取数据
- 语法 : `${expression}`

**步骤**
1. 使用前配置(idea 2023以上)
	- `<%@ page isELIgnored="false" %>`
2. 获取数据
3. 存储到Request域
4. 转发到jsp

**特殊使用 - 获取Cookie**
`${cookie.key.value}` - key为要查找的键

## JavaWeb

1. page : 当前页面有效
2. request : 当前请求有效
3. session : 当前会话有效
4. application : 当前应用有效

<span style="color:red">el表达式获取数据会依此从这四个域中寻找 , 直到找到为止</span>

# JSTL标签

- JSP标准标签库(Jsp Strandarded Tag Library) , 使用标签取代JSP页面上的Java代码
- 注意EL表达式可能被IDEA禁用

**步骤**
1. 导入坐标
```xml
<dependency>  
  <groupId>jstl</groupId>  
  <artifactId>jstl</artifactId>  
  <version>1.2</version>  
</dependency>  
<dependency>  
  <groupId>taglibs</groupId>  
  <artifactId>standard</artifactId>  
  <version>1.1.2</version>  
</dependency>
```
2. 引入标签
`<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`
3. 编写代码

**常用**
- c:forEach
	- `<c:forEach>` - 相当于for循环
	- `items`: 被遍历的容器
	- `var`:遍历产生的临时变量
	- `vatStatus` : 遍历状态对象 - 遍历的第几个数据
		- `index` - 从0开始
		- `count` - 从1开始
```jsp
<c:forEach items="${brands}" var="brand">
	<tr>
		<td>${brand.id}</td>
		<td>${brand.brandName}</td>
		<td>${brand.companyName}</td>
		<td>${brand.description}</td>
	</tr>
</c:forEach>
```
**forEach中属性会被反射调用方法**

- 普通for循环
```jsp
<c:forEach begin="0" end="10" step="1" var="i">
	${i}
</c:foreach>
```
- c:if
	- `<c:if test="判断条件">`

# MVC模式和三次架构

## MVC模式

- M : Model - 业务模型, 处理业务 `[JavaBean]`
- V : View - 视图, 界面展示 `[JSP]`
- C : Controller - 控制器, 处理请求, 调用模型和视图 `[Servlet]`

**过程**
1. 浏览器访问控制器
2. 控制器从模型中获取数据
3. 控制器获得数据后使用数据创建视图
4. 视图返回给浏览器

**好处**
- 职责单一, 互不影响
- 有利于分工协作
- 有利于组件重用

## 三层架构

1. 表现层 : 接收请求, 封装数据, 调用业务逻辑层, 响应数据
	- 包名: `web / controller`
2. 业务逻辑层 : 对业务逻辑进行封装, 组合数据访问层中的基本功能, 形成复杂逻辑功能
	- 包名: `service`
3. 数据访问层(持久层) : 对数据库进行CRUD基本操作
	- 包名: `dao / mapper`

### 三大框架

**SSM三大框架**
- 表现层 : SpringMVC
- 数据访问层 : MyBatis
- 业务逻辑层 : Spring

**可以提高代码复用性**

![[后端/JavaWeb/Inbox/Pasted image 20240202204038.png]]











