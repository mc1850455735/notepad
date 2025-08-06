# 注解开发基础

- spring2.5开始, 就提供了比较完善的注解开发
- 到了spring3.0, 就提供了纯注解开发的开发模式

## 注解开发定义bean

1. `@Component`代表了xml中的bean标签
	- 注解中默认的属性为value, 为bean的名称
	- 不写参数也是可以的
		- 忽略参数时**建议使用按类型**创建bean
		- 忽略参数时, 默认的bean名称为注解下的类名, 但首字母小写
```java
@Component("bookDao")  
public class BookDaoImpl implements BookDao {  
    @Override  
    public void save() {  
        System.out.println("Book Dao save ...");  
    }  
}
```
2. 在配置文件中**添加组件的路径**, 需要在context命名空间下
	- `xmlns:context="http://www.springframework.org/schema/context"`
	- `http://www.springframework.org/schema/context`
	- `http://www.springframework.org/schema/context/spring-context.xsd`
```xml
xmlns:context="http://www.springframework.org/schema/context"  
xsi:schemaLocation="
http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context.xsd
"
```
3. 添加扫描路径
	- 可以由多个标签以扫描多个包
	- **建议**使用自己的顶级域名如`top.majinliang`作为扫描路径
```xml
<context:component-scan base-package="top.majinliang"/>
```

### 三个衍生注解
和`@Component`**完全一样** , 但是可读性更强
- `@Controller` : 用于表现层bean定义
- `@Service` : 用于业务层bean定义
- `@Repository` : 用于数据层bean定义
一般来说工具类会用`@Component`

## 纯注解开发

- Spring 3.0升级了纯注解开发模式 , 使用Java类代替了配置文件, 开启了Spring快速开发赛道

**使用**
1. `@Configuration` : 将当前类定义成为配置类
	- 相当于xml配置文件的外壳
2. `@ComponentScan` : 指定配置类要扫描的路径
	- `@ComponentScan` 只能定义一次
	- 想要扫描多个包, 应传入**字符串数组**
		- `{"top.majinliang.dao", "top.majinliang.service"}`
```java
@Configuration  
@ComponentScan("top.majinliang")  
public class SpringConfig {  
}
```
3. 获取容器 
	- 将原本的获取容器的接口改为`AnnotationConfigApplicationContext`
	- 也可以直接使用`对象.register(类)`来注册对应的配置, 而不是在构造函数中书写
	- 使用register后要注意refresh
```java
ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
```
*如此就可以不写配置文件*

# bean管理

**设置bean为非单例对象**
- 设置`@Scope`注解 , 在属性中添加对应的字符串
	- singleton : 单例
	- prototype : 非单例
```java
@Repository  
@Scope("prototype")  
public class BookDaoImpl implements BookDao {  
    @Override  
    public void save() {  
        System.out.println("Book Dao save ...");  
    }  
}
```

**设置bean生命周期**
- 写出bean的两个生命周期方法
- 对于init方法, 添加注解 `@PostConstruct` : 构造方法后
- 对于destroy方法 , 添加注解 `@PreDestroy` : 销毁前
- 在当前版本, 欲使用该对注解需要添加依赖`javax-annotation`
```xml
<dependency>  
    <groupId>javax.annotation</groupId>  
    <artifactId>javax.annotation-api</artifactId>  
    <version>1.3.2</version>  
</dependency>
```


## 第三方bean管理

1. 在配置类中定义一个方法获得要管理的对象
2. 将方法返回值通过`@Bean`定义成bean
	- 添加@bean, 表示当前方法返回值是一个bean
	- bean可以添加名称, 也可以直接使用 类对象 进行创建
3. 通过加载配置类获取想要的对象
```java
@Configuration  
public class JdbcConfig {  
    @Bean  
    public DataSource dataSource() {  
        DruidDataSource dataSource = new DruidDataSource();  
        ... 
        return dataSource;  
    }  
}
```
**Jdbc配置类一般由专门的类进行管理**
- 问题 : 类无法被识别
	- 解决1 : **不推荐** , 阅读性差
		1. 将管理的配置类也添加`@Configuration`注解
		2. 在SpringConfig中**扫描**对应的包
	- 解决2 : 可以精准看出导入了什么类
		- 使用`@Import`注解导入对应的配置类, 多个配置类使用**数组**
		- 数组形式: `{xx.class, yy.class, ...}`
```java
@Configuration  
@Import({JdbcConfig.class})  
public class SpringConfig {}
```

### 第三方bean的依赖

需要使用资源文件的配置类时, propertySource应该配置在SpringConfig上

**简单类型**
- 将需要使用的值提取到函数外, 再直接通过`@Value`注入值
```java
@Value("com.mysql.cj.jdbc.Driver")  
private String driverName;  
@Value("jdbc:mysql://localhost:3306/spring_db")  
private String url;  
@Value("root")  
private String username;  
@Value("123456")  
private String password;
```
**引用类型**
1. 扫描获取需要被引用类型的bean
2. 在第三方依赖bean的方法中添加对应的依赖bean参数即可
	- 通过自动装配进行
	- 必须保证依赖的bean被定义
- 只需要在函数参数中加入对应bean参数, 即可使用自动装配完成
```java
@Bean  
public DataSource dataSource(BookDao bookDao) {...}

```

# 依赖注入

- 注解开发中, 为了加速开发速度, 只提供了自动注入的依赖注入

### 引用类型

- 使用`@Autowired`注解在对应的bean中添加依赖关系
	- 默认为**按类型装配**
	- 当同时有多个相同类型的实现类时(如同时实现接口BookDao的BookDaoImpl1和BookDaoImpl2), 默认的类型装配会失效, 这时应该使用按名称装配
	- 不需要在`@Autowired`注解上额外添加名称, 只需要显式声明bookDao中`@Repository`的名称, 在发生类型相同的冲突时 , 其会根据**变量名**查找对应的命名相符的bean
		- 不推荐使用`@Autowired`的按名称注入
		- 可以使用`@Qualifier`按名称注入, 但是`@Autowired`是必须的
可以通过给对应bean命名的方式使注解根据变量名找到两相同类型bean中对应的那个bean
```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {}
@Repository("bookDao2")
public class BookDaoImpl2 implements BookDao {}
```
也可以使用Qulifier注解按名称为autowired选择要注入的bean
```java
@Autowired
@Qualifier("bookDao2")
private BookDao bookDao;
```

- 既可以放在对应的变量上, 也可以放在变量对应的setter上
- 不需要setter, 直接放在对应的变量上, 也可以正确运行
```java
@Autowired  
private BookDao bookDao;
```

**注意**
- 自动装配基于反射设置创建对象并**暴力反射**对应属性为私有属性初始化数据, 因此**无需**提供**setter方法**
- 自动装配默认使用**无参构造方法**创建对象, 如果不提供对应构造方法, 请提供唯一的构造方法

**Spring推荐将注解放到setter上的方式**
**实际开发中更推荐不写setter方法的形式**

### 简单类型

- 使用`@Value`进行简单类型的注入
	- 其值有可能来自外部properties, 方便了属性的注入
```java
@Value("majinliang")  
private String name;
```

##### 加载外部properties文件
- 注意: 该注解中**不支持通配符**
- 在配置类中添加注解`@PropertySource("配置文件名")`
	- 注解中**不支持通配符**, 每个位置都必须准确的写入
		- 但是classpath是允许的
	- 同样可以通过String数组实现导入多个配置文件
- 在`@Value`中使用`${}`
```java
@Configuration  
@ComponentScan("top.majinliang")  
@PropertySource("{jdbc.properties, jdbc2.properties}")  
public class SpringConfig {  
}
```

# 注解开发总结

**XML配置与注解配置比较**

![[后端/SSM/01-Spring/Inbox/Pasted image 20240207152054.png]]










