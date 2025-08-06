
# Filter

## 概念

- Filter表示过滤器, 是JavaWeb三大组件之一(Servlet, Filter, Listener)
- 过滤器可以拦截请求, 以实现一些特殊的功能
- 过滤器一般完成一些通用的操作
	- 权限控制, 统一编码处理, 敏感字符处理等

## Filter快速入门

1. 定义类, 实现Filter接口, 重写其所有方法
2. 配置Filter拦截资源路径 : 使用`@WebFilter("拦截路径")`注解
3. 在`doFilter`方法中输出一句话, 并放行

传统的Filter接口在**javax.servlet**中

## Filter执行流程

- 放行前逻辑 : 对request数据进行处理
- 放行
- 放行后逻辑 : 对response数据进行处理

**问题**
1. 放行后访问资源, 资源访问完成, 会返回Filter中
2. 回到Filter后, 会从放行后开始执行

**流程**
放行前逻辑 -> 放行 -> 访问资源 -> 访问放行后逻辑

## Filter拦截路径

**配置方式**
1. 拦截具体的资源 : `index.jsp` : 只拦截index.jsp
2. 目录拦截 : `user/*` : 拦截目录下所有资源
3. 后缀名拦截 : `*.jsp` : 访问后缀名为jsp的资源, 都会被拦截
4. 拦截所有(最常使用) : `/*` : 拦截所有访问


## 过滤器链

- Web应用中可以配置多个过滤器, 这些过滤器被成为过滤器链

![[后端/JavaWeb/Inbox/Pasted image 20240204164806.png]]

- 注解配置的过滤器Filter, 优先级按照过滤器类名的字符串自然排序

## 使用Filter检验登录

- 登录前应该注意放行登录和注册相关的资源

- 方法: 
	1. 判断访问的是否是登陆相关资源
	2. 判断Session中是否有Username对象
- 由于有很大可能没有对象, 所以要先用Object接收, 确认有对象后再进行对象类型的转换

**检验是否是登录相关资源**
```java
// 判断访问的资源路径是否和登陆注册相关  
String[] urls = {  
        "/imgs/",  
        "/css/",  
        "/login.jsp",  
        "/loginServlet",  
        "checkCodeServlet",  
        "register.jsp",  
        "registerServlet"  
};  
// 获取访问资源路径  
String url = req.getRequestURL().toString();  
  
// 循环遍历  
for (String u : urls) {  
    if(url.contains(u)){  
        // 放行  
        chain.doFilter(req, response);  
        // 结束检验  
        return;  
    }  
}
```

**检验是否登录**
```java
// 1.判断Session中是否有user  
HttpSession session = req.getSession();  
Object username = session.getAttribute("username");  
  
// 2.判断username是否为null  
if(null != username){  
    // 2.1用户登陆过  
    chain.doFilter(req, response);  
}else{  
    // 2.2用户没有登录  
    // 跳转到登录  
    req.setAttribute("login_msg", "请登录");  
    req.getRequestDispatcher("/login.jsp").forward(request,response);  
}
```


# Listener
**已经用的不多了**

- Listener表示监听器, 是JavaWeb三大组件之一
- 监听器可以监听就是在application, session, request三个对象创建, 销毁或者往其中添加修改删除属性时自动执行代码的功能

- ServletContext监听
	- ServletContextListener : 监听服务器是否启动成功
- Session监听
- Request监听

## Listener使用

1. 定义类, 实现 `ServletContextListener` 接口
2. 在类上添加 `@WebListener` 接口

**两个方法**
- `contextInitialized` : 加载资源
- `contextDestroyed` : 释放资源







