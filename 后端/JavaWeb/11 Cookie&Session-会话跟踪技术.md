# 概述

**会话**
用户打开浏览器, 访问web浏览器资源, 会话建立. 直到有一方断开连接, 会话结束. 一次会话中可以包含**多次**请求和响应.

**会话跟踪**
一种维护浏览器状态的方法, 服务器需要识别多次请求是否来自同一浏览器, 以便在同一次会话的多次请求间**共享数据**

HTTP协议是**无状态**的, 每次浏览器项服务器请求时, 服务器会将该请求视为新的请求, 所以需要会话跟踪技术来实现会话内数据共享

**实现方法**
1. 客户端会话跟踪技术 : Cookie
2. 服务端会话跟踪技术 : Session

# Cookie

- 客户端会话技术, 将数据保存到客户端, 每次请求都携带Cookie数据进行访问

## Cookie使用

### 发送Cookie

1. 创建Cookie对象, 设置数据
	- `Cookie cookie = new Cookie("username", "zs");`
2. 发送Cookie到客户端 : 使用response对象
	- `response.addCookie(cookie);`

### 获取Cookie

1. 使用request对象获取客户端携带的所有Cookie
	- `Cookie[] cookies = request.getCookies();`
2. 遍历数组获取每一个Cookie对象
3. 使用cookie对象方法获取数据
	- `cookie.getName();`
	- `cookie.getValue();`

## Cookie原理
- Cookie的实现是基于HTTP协议的
	- 响应头 : set-cookie
	- 请求头 : cookie

- 发送时, tomcat会在响应中设置响应头set-cookie告诉浏览器设置cookie
![[后端/JavaWeb/Inbox/Pasted image 20240203145741.png]]

- 请求时, 浏览器会自动添加请求头cookie以供tomcat获取cookie
![[后端/JavaWeb/Inbox/Pasted image 20240203145811.png]]

## Cookie使用细节

### Cookie存活时间

- 默认情况下, Cookie在浏览器内存中, 浏览器关闭, 内存释放, 则Cookie被销毁
- `setMaxAge(int seconds)` : 设置Cookie存活时间
	1. 正数: 写入浏览器所在电脑硬盘, 持久化存储, 到时间自动关闭
	2. 负数: 默认值, 存入浏览器内存, 浏览器关闭, Cookie销毁
	3. 〇 : **删除**对应Cookie

### Cookie存储中文

- 默认情况下, cookie存储中文会报错
- 如果需要存储, 则需要进行编码(一般使用URL编码)
	- `value = URLEncoder.encode(value, "utf-8");` - 存储 - URL编码
	- `URLDecoder.decode(value, "utf-8");` - 解析 - URL解码

# Session

- 服务端会话技术, 将数据保存到服务端 
- JavaEE提供HttpSession接口, 实现一次会话多次请求间数据共享功能

## Session使用

1. 获取Session对象
	- `HttpSession session = request.getSession();`
2. Session对象功能
	- `session.setAttribute("username","zs");` - 设置键值对, 值为Object
	- `session.getAttribute("username");` - 获取对象
	- `session.removeAttribute("username")` - 根据key删除键值对

## Session原理

- Session的实现是基于Cookie的

- tomcat会将session的id给浏览器, 保存为cookie
![[后端/JavaWeb/Inbox/Pasted image 20240203161401.png]]

- 浏览器携带着session的id向tomcat请求, tomcat根据session id进行查找
- 不同浏览器的session id不同
![[后端/JavaWeb/Inbox/Pasted image 20240203161523.png]]

## Session使用细节

### Session钝化&活化

- 钝化 : 服务器正常关闭后, Tomcat自动将Session数据写入硬盘的文件中
- 活化 : 服务器重新启动后, 从文件中加载Session, 并删除原文件

### Session销毁

- 默认情况下, 无任何操作, Session在30分钟内自动销毁
- 在web.xml中可以这样设置, 单位为分钟
```xml
<session-config>
	<session-timeout>30</session-timeout>
</session-config>
```

- 调用Session对象的`invalidate()`方法销毁Session

# 结论

- Cookie和Session都是用来在一次会话内多次请求间数据共享的

**区别**
- 存储位置 : Cookie客户端, Session服务端
- 安全性: Cookie不安全, Session安全
- 数据大小 : Cookie 3KB, Session无限制
- 存储时间 : Cookie可以长期存储, Session默认30分钟
- 服务器性能 : Cookie不占用服务器资源, Session占用服务器资源














