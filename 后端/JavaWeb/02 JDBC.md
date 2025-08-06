
# 概述

## 简介

**JDBC就是使用Java语言操作关系型数据库的一套API。**
全称 Java DateBase Connectivity - Java数据库连接

## 本质

JDBC定义了操控数据库的一套接口，由数据库厂家来实现其接口，这种实现类就叫做驱动，叫做jar包。使用JDBC真正运行的就是驱动jar包

## 优点

- 各数据库厂商使用相同的接口，不需要针对不容的数据库分开开发
- 可随时替换底层数据库

## 模板

```java
// 1.注册驱动(mysql8.0以上使用cj包)  
Class.forName("com.mysql.cj.jdbc.Driver");  
// 2.连接到数据库  
String url = "jdbc:mysql://localhost:3306/itcast?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai";  
String username = "root";  
String password = "123456";  
Connection connection = DriverManager.getConnection(url, username, password);  
// 3.定义sql语句  
String sql = "update account set money=1000 where id = 1;";  
// 4.创建执行sql的对象  
Statement statement = connection.createStatement();  
// 5.执行sql, count为受影响语句数量  
int count = statement.executeUpdate(sql);  
// 6.打印结果  
System.out.println(count);  
// 7.关闭连接  
statement.close();  
connection.close();
```

连接url：
`"jdbc:mysql://localhost:3306/itcast?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai"`

jar包放到src的同级目录lib中
右键选择 `Add as Library`


# API详解

## DriverManager

1. **注册驱动 - `registerDriver()`**

registerDriver在Mysql5.0以后驱动中默认执行, 所以代码中可以省略
自动加载jar包中META-INF/service/java.sql.Driver文件中的驱动类

2. **获取数据库连接 - `getConnection(url, username, password)`**

连接url：
`"jdbc:mysql://localhost:3306/itcast?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai"`

连接本机mysql且本机端口为3306则可以简化掉url中的`localhost:3306`

useSSL为安全连接方式，8.0以前的Mysql版本不设置为false会出现警告

## Connection

1. 获取执行SQL的对象
	- 普通执行SQL对象 - `createStatement()`
	- 预编译执行SQL对象, 防止SQL注入 - `prepareStatement(sql)`
	- 执行存储过程对象(不重要) - `prepareCall(sql)`
2. 管理事务
	1. MySQL中
		- 开启事务：`BEGIN`;`START TRANSACTION`;
		- 提交事务：`COMMIT`;
		- 回滚事务：`ROLLBACK`;
	2. JDBC事务管理
		- 开启事务: `setAutoCommit()` - `true` 为自动提交, `false` 为手动提交事务, 即开启事务
		- 提交事务: `commit()`
		- 回滚事务: `rollback()`

**用完以后不要忘记释放资源**

## Statement

**作用:**
1. 执行SQL语句
	- `int executeUpdate()` - 执行DML和DDL; DML返回受影响行数, DDL成功也有可能返回0
	- `ResultSet executeQuery()` - 执行DQL语句

>可以执行DDL, DML, DQL三种SQL语句

**用完以后不要忘记释放资源**

**`@Test` - 单元测试方法**

## ResultSet

**作用 :**
1. 封装了DQL查询语句的结果
	- `ResultSet stmt.executeQuery()` - 执行DQL语句, 返回ResultSet对象
2. 获取查询结果
	- `next()` 
		1. 将光标从当前位置向前移动一行
		2. 判断当前行是否为有效行 (true - 有效, false - 无效)
	- `xxx getXxx( "列名"/列号 )` - 获取参数

游标行默认在表头, 一个next到达下一行

**示例代码 :**
```java
// 循环判断游标是否是最后一行末尾
while(rs.next()){
	// 获取数据
	rs.getXxx(参数);
}
```

**用完以后不要忘记释放资源**

## PreparedStatement

**作用 :**
1. 预编译SQL语句并执行以防止SQL注入问题 - 将敏感字符进行转义

SQL注入 : 通过输入实现定义好的SQL语句以达到执行代码对服务器进行**攻击**的方法
sql注入的一个可能代码
`"'or '1"`

步骤:
1. 获取PreparedStatement对象
	- 书写sql语句, 使用?代替其中的参数
	- connection.preparedStatement(sql);
2. 设置参数值
	- setXxx(参数1, 参数2) - 设置参数的值, 参数1为?的编号(从1开始), 参数2为?的值
3. 执行SQL
	- 不需要再传递sql

**原理 :**

1. 通过预编译sql模板, 提升了执行时的速度
开启预编译功能 - `useServerPrepStms=true`

配置mysql执行日志:
```
log-output=FILE
general-log=1
general_log_file="D:\mysql.log"
slow-query-log=1
slow_query_log_file="D:\mysql_slow.log"
long_query_time=2
```

# 数据库连接池

## 概述

**简介**
1. 数据库连接池是个容器，负责分配、管理数据库连接
2. 允许应用程序重复使用一个现有的数据库连接而不是重新建立
3. 释放`空闲时间`超过`最大空闲时间`的数据库连接来避免因为没有释放数据库连接引起的数据库连接泄露。

**优点**
1. 资源重用
2. 提升系统响应速度
3. 避免数据库连接泄露

## 实现

标准接口：`DataSource` - 由第三方组织提供此接口
功能：获取接口 - 不再通过`DriverManager`的`getConnection`
`Connection getConnection();`

常见的数据库连接池：
- DBCP
- C3P0
- Druid ( 德鲁伊 - 阿里巴巴提供 )

## Druid

数据源名称 : DruidDataSource

**使用步骤**
1. 导入jar包 - druid-1.1.12.jar
```xml
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid-spring-boot-starter</artifactId>  
    <version>1.2.6</version>  
</dependency>
```

2. 定义配置文件
```yml
spring:  
  datasource:  
    druid:  
      driver-class-name: com.mysql.cj.jdbc.Driver  
      url: jdbc:mysql://localhost:3306/itcast?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai  
      username: root  
      password: 123456  
      # 初始化连接数量  
      initial-size: 5  
      # 最大连接数  
      max-active: 10  
      # 最大等待时间(ms)  
      max-wait: 3000
```

3. 加载配置文件
```java
Properties properties = new Properties();
properties.load(new FileInputStream("文件路径"));
```

4. 获取连接池对象
```java
Datasource datasource = DruidDataSourceFactory.createDataSource(properties);
```

5. 获取连接
```java
Connection connection = datasource.createConnection();
```

# JDBC练习

(略)






