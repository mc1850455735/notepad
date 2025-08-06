
# 简介

**概述**
- SpringMVC与Servlet技术等同, 均属于web层开发技术
	- SpringMVC属表现层
- SpringMVC是一种基于Java实现的MVC模型的轻量级Web框架
- 优点
	- 使用简单, 开发便捷 (相比于Servlet)
	- 灵活性强

![[后端/SSM/02-SpringMVC/Inbox/Pasted image 20250731132749.png]]

### 入门案例

1. 导入对应的坐标 : 其会**自动包含Spring**的基础坐标
	- 需要导入SpringMVC坐标和Servlet坐标
	- Servlet会和tomcat插件冲突, 所以将scope设置为provided
	- 导入SpringMVC, 其依赖坐标为spring-aop, spring-context等, 故不必再手动导入其依赖坐标
```xml
<dependency>  
    <groupId>javax.servlet</groupId>  
    <artifactId>javax.servlet-api</artifactId>  
    <version>3.1.0</version>  
    <scope>provided</scope>  
</dependency>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```
2. 创建SpringMVC控制器类
	1. 在类上添加bean定义注解 `@Controller`
	2. 在类中定义对应的操作方法
	3. 设置当前操作的访问路径 `@RequestMapping('路径')`
	4. 设置当前操作的返回值类型
		- `@ResponseBody` : 将返回值整体作为响应内容返回
```java
@Controller  
public class UserController {  
    @RequestMapping("/save")  
    @ResponseBody  
    public String save(){  
        System.out.println("user save");  
        return "{'module':'springmvc'}";  
    }  
}
```
3. 初始化SpringMVC配置文件, 加载对应的类
	- 新建SpringMVC配置类
	- 像Spring的配置类一样定义与扫描
```java
@Configuration  
@ComponentScan({"top.majinliang"})  
public class SpringMvcConfig {  
}
```
4. 初始化Servlet容器, 加载SpringMVC环境
	- 新建Servlet容器启动配置类`ServletContainersInitConfig`, **继承**`AbstractDispatcherServletInitializer`
	- 覆盖原有的方法, 对其进行重写
```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {  
    // 加载SpringMVC容器配置  
    protected WebApplicationContext createServletApplicationContext() {  
        // 创建Web配置类对象  
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();  
        // 加载Web配置  
        ctx.register(SpringMvcConfig.class);  
        return ctx;  
    }  
    // 设置哪些请求归属SpringMVC处理  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
    // 加载Spring容器配置  
    protected WebApplicationContext createRootApplicationContext() {  
        return null;  
    }  
}
```

**Servlet配置类方法** 配置完成后直接**删掉web.xml**即可
- `createServletApplicationContext` : 加载SpringMVC容器配置
```java
protected WebApplicationContext createServletApplicationContext() {  
    // 创建Web配置类对象  
    AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();  
    // 加载Web配置  
    ctx.register(SpringMvcConfig.class);  
    return ctx;
```
- `getServletMappings` : 设置哪些请求归属SpringMVC处理
	- 一般都是所有请求都由SpringMVC控制
```java
protected String[] getServletMappings() {  
    return new String[]{"/"};  
}
```
- `createRootApplicationContext` : 加载Spring容器配置

**Tomcat如何查找到相对应的配置**
利用了Servlet 规范 3.0+ 的机制, 自动发现实现了某些接口的类并调用它们的 `onStartup()` 方法, 从而注册其初始化配置

**记得添加tomcat插件**
```xml
<build>  
	<plugins>        
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


### 入门案例分析

**启动服务器初始化过程**
1. 启动服务器, 执行 ServletContainersInitConfig (Servlet容器初始化类) , 初始化web容器
2. 执行 createServletApplicationContext , 创建了WebApplication对象, 加载到web容器中的ServletContext中
3. 加载SpringMvcConfig, 扫描包
4. 加载对应的bean
5. 加载Controller, 每个`@RequestMapping`对应一个具体的方法
	- Controller容器放在WebApplocationContext容器中
	- 地址与方法之间的关联书写在Controller下方法的@RequestMapping注解中, 但是二者之间关系的建立并不由Controller管理, 而是由其他模块统一管理
6. 执行**getServletMapping方法**, 设置**每一个**请求都通过SpringMvc, **拦截**每一个试图由Tomcat Servlet处理的方法

**单次请求过程**
1. 发送请求 `localhost/save`
2. 由于所有请求都转交SpringMvc处理, 所以web容器将发现的所有请求都转给SpringMvc
3. 解析请求路径`/save`
4. 由解析到的路径匹配对应的方法
5. 执行方法
6. 如果方法有`@ResponseBody`注解, 直接将返回值作为**响应体**返回给请求方

# bean加载控制

**要求**
- SpringMVC控制表现层bean
- Spring控制业务bean和功能bean

**方法**
- 设置SpringMVC配置类加载的bean均在controller包内
- 设置Spring
	1. 设置Spring加载范围为精确范围, 如只扫描service, dao等
	2. 设置Spring加载范围为全部, 并排除掉controller包
- 也可以**直接不区分**SpringMVC和Spring, 写到同一个配置类中
- 使用Spring扫描时, 对于MyBatis自动实现的dao包, 可以不设置扫描路径, 但是如果不设置扫描路径, 在其他不自动实现的情况时, Spring无法检测到Bean, 所以建议使用标准的添加扫描路径的标准情况

### 排除使用

- 使用属性`excludeFilters`设置需要排除的包
	- 属性`includeFilters`可以用来加载指定的bean, 使用与`excludeFilters`相似
- 为该属性创建其过滤器并设定规则
- 设定好过滤器的类型type为按注解过滤(还存在其他规则)
- 设定classes为想要过滤的注解类型, 对于Controller, 应写Controller.class
	- 既所有写 `@Controller` 的bean都将被排除
- Spring中排除Controller后, 由于SpringMvc配置类中又扫描了Controller, 所以Spring扫描完SpringMvc后, Controller在Spring中还能被访问, 如果设法使Spring扫描不到SpringMvc, 那么这种Spring与SpringMvc扫描的不同将被体现


**使用举例**
```java
@ComponentScan(value = {"top.majinliang"},  
        excludeFilters = @ComponentScan.Filter(  
                type= FilterType.ANNOTATION,  
                classes = Controller.class  
        )  
)
```

### 同时加载Spring容器

```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {  
    // 加载SpringMVC容器配置  
    protected WebApplicationContext createServletApplicationContext() {  
        // 创建Web配置类对象  
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();  
        // 加载Web配置  
        ctx.register(SpringMvcConfig.class);  
        return ctx;  
    }  
  
    // 设置哪些请求归属SpringMVC处理  
    protected String[] getServletMappings() {  
        return new String[]{"/"};  
    }  
  
    // 加载Spring容器配置  
    protected WebApplicationContext createRootApplicationContext() {  
        // 创建Web配置类对象  
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();  
        // 加载Web配置  
        ctx.register(SpringConfig.class);  
        return ctx;  
    }  
}
```

### 简化开发

- 将ServletContainerInitConfig类中继承的类修改为 `AbstractAnnotationConfigDispatcherServletInitializer`
- 重写该抽象类中的方法
	- getRootConfigClasses : 返回Spring的配置文件类
	- getServletConfigClasses : 返回SpringMVC的配置文件类
	- getServletMappings : 返回拦截路径字符串
- 优点 : **不需要手动创建配置类的实现对象**并返回, 简化了代码

**示例**
```java
protected Class<?>[] getRootConfigClasses() {  
    return new Class[]{SpringConfig.class};  
}  
  
protected Class<?>[] getServletConfigClasses() {  
    return new Class[]{SpringMvcConfig.class};  
}  
  
protected String[] getServletMappings() {  
    return new String[]{"/"};  
}
```

# Postman使用

**简介**
- 用于网页调试与发送网页HTTP请求的chrome插件
- 常用于进行接口测试
- 特征(优点) : 简单 , 实用 , 美观 , 大方

**步骤**
1. 进入WorkSpace, 创建新Workspace
2. 点击 + , 创建一个新的request
3. 输入路径, 发送请求, 得到响应
	1. pretty为响应数据
	2. preview为预览页面
4. ctrl + s , 保存设置好的request, 使用collection进行管理






