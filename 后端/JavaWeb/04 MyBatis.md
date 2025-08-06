# 概述

## 简介

**什么是Mybatis**
- 持久性框架, 用于简化JDBC
- 原为apache的开源项目iBatis，后迁移至google code并改名为Mybatis，再后迁移至GitHub

**持久层**
- 负责将数据保存到数据库的那一层代码
- JavaEE三层架构：表现层、业务层、持久层

**框架**
- 框架就是一个半成品软件，是一套可重用的、通用的、软件基础代码模型
- 是软件编写更加高效、规范、通用、可扩展

## 优势

**JDBC缺点**
1. 硬编码 - 代码写死, 不易更改
	- 注册驱动，获取连接
	- SQL语句
2. 操作繁琐
	- 手动设置参数
	- 手动封装结果集

**MyBatis简化**
1. 连接信息、SQL语句等写到配置文件中
2. 手动设置参数和手动封装结果集等自动完成

##  Mybatis入门

mybatis使用的依赖
-   **mybatis**：这是mybatis框架的核心依赖，它提供了mybatis的基本功能。
-   **mysql-connector-java**：这是mysql数据库的驱动依赖，它使得mybatis能够连接和操作mysql数据库。
-   **junit**：这是一个单元测试的依赖，它可以用来测试mybatis的功能和性能。
-   **log4j**：这是一个日志工具的依赖，它可以用来打印mybatis的运行日志和调试信息。

**步骤**
- 添加MyBatis依赖
- 编写MyBatis核心配置文件
- 编写SQL映射文件
- 进行编码
	- 定义pojo类
	- 加载核心配置, 获取SqlSessionFactory
		- 设置配置文件地址
		- 定义配置文件为输入流
		- 创建工厂对象
	- 获取SqlSession对象,执行SQL语句
	- 释放资源

**使用资源文件**
```xml
<!-- 加载资源文件 -->  
<properties resource="jdbc.properties"></properties>
```

### 依赖信息(注意版本)

```xml
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis</artifactId>  
    <version>3.5.6</version>  
</dependency>  
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <version>8.2.0</version>  
</dependency>  
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>4.13.2</version>  
    <scope>test</scope>  
</dependency>  
<dependency>  
    <groupId>org.slf4j</groupId>  
    <artifactId>slf4j-api</artifactId>  
    <version>1.7.20</version>  
</dependency>  
<dependency>  
    <groupId>ch.qos.logback</groupId>  
    <artifactId>logback-core</artifactId>  
    <version>1.2.3</version>  
</dependency>  
<dependency>  
    <groupId>ch.qos.logback</groupId>  
    <artifactId>logback-classic</artifactId>  
    <version>1.2.3</version>  
</dependency>
</dependency>
```

### MyBatis配置文件示例
```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
	<!-- 别名 -->  
	<typeAliases>  
	    <package name="top.majinliang.web.pojo"/>  
	</typeAliases>
    <environments default="development">  
        <environment id="development">  
            <transactionManager type="JDBC"/>  
            <dataSource type="POOLED">  
                <!-- 数据库连接信息 -->  
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
                <property name="url" value="jdbc:mysql://localhost:3306/itcast?useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>  
                <property name="username" value="root"/>  
                <property name="password" value="123456"/>  
            </dataSource>  
        </environment>  
    </environments>  
    <mappers>  
        <!-- 扫描mapper -->  
        <package name="top.majinliang.web.mapper"/>  
    </mappers>  
</configuration>
```

### SQL映射文件示例
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
	<resultMap id="brandResultMap" type="brand">  
	    <result column="brand_name" property="brandName" />  
	    <result column="company_name" property="companyName" />  
	</resultMap>
</mapper>
```

## SQL警告信息

在IDEA中添加数据库相关信息
idea也可以作为一个数据库管理工具使用

`Database` -> `+` -> `Mysql` -> 添加相关信息
添加完数据库后, 要刷新一下来获取数据库相关信息
添加完数据库后, 还需要添加SQL方言信息

# Mapper代理开发

## 目的
- 解决SqlSession使用时的硬编码问题。
- 简化后期执行SQL

## 步骤
1. 定义与SQL映射文件同名的Mapper接口且Mapper接口和SQL映射文件放置在同一目录下
	- 可以在resources和java中使用相同的包名来保证
	- 编译后resources和java代码在同一目录下
	- 在resources中创建 Directory 时不能用 "`.`" , 因为这在Directory中被视为文件夹名
	- 可以将"`.`"换成"`\`"
2. 设置SQL映射文件的namespace属性为Mapper接口全限定名
3. 在Mapper接口中定义方法, 方法名就是SQL映射文件中sql语句的id, 并保持参数类型和返回值类型一致
4. 修改配置路径
5. 进行编码
	1. `UserMapper userMapper = sqlSession.getMapper(UserMapper.class);`  
	2. `List<User> users = userMapper.selectAll();`

## Mapper代理扫描方式

如果Mapper接口名称和SQL映射文件名称相同且在同一目录下, 则可以使用包扫描方法简化SQL映射文件的加载

```xml
<!-- Mapper代理方式 -->  
<package name="top.majinliang.mapper"/>
```

# Mybatis配置文件

- `configuration` - 顶层标签
	- `environments` - 配置数据库连接环境信息, 可以配置多个environment, 通过default属性切换不同的environment
		- `environment` - 环境信息
			- `transactionManager` - type : 事务管理方式
			- `datasource` 数据库连接池 - type : 数据库连接池
				- `property` : 属性信息
	- `mappers` - mapper配置信息
		- `mapper` - 添加mapper文件
		- `package` - mapper代理方式

***配置标签要遵守前后规则(规则见官网)***
*transactionManager和datasource将来会被spring接管, 不需要多做了解*

类型别名:
可以写类名, 也可以直接写包名
*(如果写包名 : 定义完成后可以不再加包名, 同时不区分大小写)*
```xml
<!-- 写类名 -->
<typeAliases alias="User" type="top.majinliang.User" />

<!-- 写包名 -->
<typeAliases>
	<package name="top.majinliang.pojo" />
</typeAliases>
```

基本类型已经写好了别名, 不需要继续配置

# Mybatis操作

MybatisX插件(小鸟插件)
- xml跳转对应Mapper接口
- 根据接口自动生成statement

## 问题

`数据库表字段名称` 和 `实体类属性名称` 不一致, 则不能自动封装数据  
1. 起别名 - 对不一样的列名起别名使其一致  
	- 缺点: 每次查询都要定义一次别名  
	    - 解决: 使用SQL片段  
		    - 缺点: 不灵活  
2. resultMap
	- `id`: 主键字段  
	- `result`: 一般字段
		- `column` :表的列名
		- `property` :实体类属性名
	- 将select语句中的 `resultType` 修改为 `resultMap`

## 使用sql片段集成标签
```xml
<!-- sql片段 -->  
<!-- 使用if标签绕过idea -->  
<sql id="brand_column">  
    <if test="true">  
        id, brand_name as brandName, company_name as companyName, ordered, description, status    
    </if>  
</sql>
```

## 使用resultMap(推荐)
```xml
<!-- 
	id: 唯一标识
	type: 映射类型, 支持别名
-->
<resultMap id="brandResultMap" type="brand">  
	<!--
		column :表的列名
		property :实体类属性名
	-->
    <result column="brand_name" property="brandName"/>  
    <result column="company_name" property="companyName" />  
</resultMap>
```


## 配置文件开发

### 查询

#### 查询所有
- 编写接口 - Mapper
- 编写SQL - XML
- 执行
```xml
<!-- 使用sql片段 -->
<select id="selectAll" resultType="top.majinliang.pojo.Brand">
	select 
		<include refid="brand_column"></include>
	from tb_brand;
</select>

<!-- 使用resultMap -->
<select id="selectAll" resultMap="brandResultMap">  
    select *    
    from tb_brand;
</select>
```

#### 查询详情
- 参数id
- 返回Brand

语句中的`#{id}`在编译时会被替换成 `?` , 参数会被替换到 `?` 里  
- 参数占位符  
	1. #{} - 替换成 ? 再把参数传入, 可以防止sql注入  
	2. ${} - 拼sql, 直接替换成参数, 存在sql注入问题  
	3. 使用时机  
		- 参数传递的时候: #{}  
		- 表名/列名不固定情况下传递的时候: ${} 存在sql注入问题  
- 参数类型: `parameterType` - 可以省略, 到时候传什么值赋什么值  
- 特殊字符处理:  
	1. 转义字符: `<` 改为 `&lt` 等  
	2. CDATA区: 其中所有都会作为文本理解
		- <![CDATA[内容]]>: CD提示

```xml
<select id="selectById" resultMap="brandResultMap">  
    select *    
    from tb_brand    
    where id = #{id};
</select>
```

#### 条件查询

##### 多条件查询
- 参数: 所有查询条件
- 结果: `List<Brand>`

**方法**
- 多个散装参数需要使用`@Param()`注解进行对应
- 一个对象可以直接传入, MyBatis寻找对应的get方法
- 也可以Map对象进行键值的对应

**参数接收**  
1. 散装参数: 如果方法中有多个参数, 需要使用`@Param("SQL占位符名称")`  
2. 对象参数: 对象属性名称要和参数占位符名称一致  
3. map集合参数

```java
public List<Brand> selectByCondition(@Param("status")int status, @Param("brandName")String brandName, @Param("companyName")String companyName);  
public List<Brand> selectByCondition(Brand brand);  
public List<Brand> selectByCondition(Map map);
```

##### 动态条件查询

Mybatis对动态SQL有很强大的支撑
- if: 条件判断
- choose(when, otherwise): 多个条件选择一个
- trim(where, set)
- foreach

**多条件动态查询**
- if: 条件判断  
	- test: 逻辑表达式  
	- 问题: 第一个不存在, `and` 就直接在最前面, sql语句报错  
		- 解决: 添加恒等式 `1 = 1` 在最前, 所有条件都添加 `and`  
		- 解决: `<where>`替换 `where` 关键字, 所有条件前都加上 `and`

**单条件动态查询**
- choose(when, otherwise): 多个条件选择一个
	- `<choose>` : 相当于switch
	- `<when>` : 相当于case
	- `<otherwise>` : 相当于default
	- 也可以用 `<where>` 将 `<choose>` 包裹起来, where会将语法自动调节正确

### 添加

- 参数: 除id外所有数据
- 结果: void - 通过异常来控制添加成功与否

mybatis事务不是自动提交的, 所以修改完数据库后要提交
也可以在设置sqlSession时设置成自动提交事务

**主键返回**
- useGeneratedKeys="true";
- keyProperty="id";
执行完sql后, 类中的与主键相关的属性会被自动赋值

```xml
<insert id="add" useGeneratedKeys="true" keyProperty="id">  
    insert into 
    tb_brand 
	    (brand_name, company_name, ordered, description, status)
	values 
		(#{brandName}, #{companyName}, #{ordered},
		#{description}, #{status});
</insert>
```

### 修改

- 参数 - 所有数据
- 结果 - void

修改方法可以返回int类型, 该返回值即为受影响的行数。

#### 修改全部字段

```java
public void add(Brand brand);
```

```xml
<update id="update">  
    update 
	    tb_brand    
    set        
	    brand_name = #{brandName},        
	    company_name = #{companyName},        
	    ordered = #{ordered},        
	    description = #{description},        
	    status = #{status}    
    where        
	    id=#{id};
</update>
```
   
#### 修改动态字段

- 只使用if判断原有的传入
	- 问题
		1. 有逗号的项有可能在最后一个
		2. 如果没有任何参数, 可能set直接接where语句
	- 方法:通过`<set>`标签代替原有的set , 在`<set>`内使用`<if>`, 可以动态提供update语句.

### 删除

#### 删除单个
- 参数: id
- 结果: void

```xml
<delete id="deleteById">  
    delete    
    from tb_brand    
    where id = #{id};
</delete>
```

#### 批量删除
- 参数: id数组
- 结果: void

Mybatis会将数组参数封装成为一个Map集合  
- 默认: array = 数组  
- 可以使用`@Param`注解改变map集合默认key名称  
- 使用`separator`来定义分隔符防止各个id之间没有逗号

可以设置foreach标签的open和close分别为左右括号, 可以省略代码中的括号
`<foreach collection="ids" item="id" separator="," open="(" close=")">`
参数数量随着id数组长度产生变化

```java
void deleteByIds(@Param("ids")int[] ids);
```

```xml
<delete id="deleteByIds">  
    delete    
    from tb_brand    
    where id in (        
	    <foreach collection="ids" item="id" separator=",">  
            #{id}        
		</foreach>  
        );
</delete>
```

## MyBatis注解传递

- 单个参数:
	- pojo类: 直接使用, 属性名和参数占位符名称一致
	- Map集合: 直接使用, 键名和参数占位符名称一致
	- Collection: 其他Collection封装成Map集合, 有两个键分别对应collection, 名为arg0和collection
		- List: 多出来一个名为list的key
		- Array: 没有名为collection的key, 多出来一个名为arrray的key
	- 其他: 直接使用
- 多个参数
	- 使用`@Param`注解

**MyBatis提供了ParamNameResolver类进行参数封装**
- 如果有多个参数, 会将参数封装成为Map集合
	- Map中对于同一个参数值有两个默认key(不推荐使用)
		- 一个为`arg0`, 从0开始 ( 可以使用`@Param`替换 )
		- 一个为`param1`, 从1开始
	- 可以使用`@Param`注解替换Map集合中默认的arg键名

**结论**
Collection, Array, List, 多个参数都使用`@Param`进行替换

## 注解开发
**通过注释实现了自动代理**

将SQL语句直接写到注解中
- 比使用配置文件更加方便
- 只适合写一些简单的sql语句

查询: `@Select`
添加: `@Insert`
修改: `@Update`
删除: `@Delete`

**使用ResultMap**
`@ResultMap("Map名")`

SQL语句直接写在注解的字符串参数中

```java
@Select("select * from tb_user where id=#{id}")  
User selectById(int id);
```

# SqlSessionFactory优化

**问题**
- 代码重复
	- 使用工具类
- 多个工厂导致多个线程池, 资源浪费严重
	- 使用静态代码块保证只创建一次

```java
private static SqlSessionFactory sqlSessionFactory;  
  
// 使用静态代码块  
static {  
    try {  
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(new FileInputStream("src/main/resources/mybatis-config.xml"));  
    } catch (FileNotFoundException e) {  
        throw new RuntimeException(e);  
    }  
}

public static SqlSessionFactory getSqlSessionFactory(){  
    return sqlSessionFactory;  
}
```

