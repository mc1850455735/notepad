
# 初识Spring

- Spring发展到现在已经形成了一种开发的生态圈, Spring提供了若干个项目, 每个项目用于完成特定的功能
- 官网 : https://spring.io/

**使用Spring全家桶完成想要的功能, 简化开发**

**重要的技术应用**
- 微服务
- web应用
- 分布式服务

## 常用的框架

- Spring Framework : 设计型框架, 其他框架基于其开发
- Spring Boot : 在简化开发的基础上加速开发
- Spring Cloud : 分布式开发相关技术

## Spring开发历程

![[后端/SSM/01-Spring/Inbox/Pasted image 20240205211958.png]]

- Spring 1.0 : 纯配置方式开发
- Spring 2.0 : 加入使用注解的配置方式
- Spring 3.0 : 演化成了可以不写配置的开发方式
- Spring 4.0/5.0 : 做了调整和升级

# Spring Framework介绍

- Spring Framework是Spring生态圈中最基本的项目, 是其他项目的根基

**`Spring Framework` 组成**
- `Core Container` : 核心容器
- `AOP` : 面向切面编程 - 在不改变原程序的基础上增强功能
- `Aspects` : AOP思想的实现 - 不由Spring实现
- `Data Access / Integration` : 数据访问 / 集成
	- `Transactions` : 事务, Spring提供了开发效率很高的事务处理方案
- `Web` : Web开发
- `Test` : 单元测试与集成测试

**`Spring 4.x` 架构图**
![[后端/SSM/01-Spring/Inbox/Pasted image 20240205212842.png]]

**学习路线**
1. 核心容器 : 
	- IoC/DI
	- 容器基本操作
2. 整合 : Spring整合MyBatis
3. AOP
	- 核心概念
	- AOP基础操作
	- AOP实用开发
4. 事务 : 事务实用开发
...

# 核心概念

## IoC与DI

- 代码书写现状
	- 耦合度较高
- 解决方案
	- 使用对象时, 不主动使用new产生对象, 转换为由外部提供对象

- **IoC(Inversion of Control)** : 控制反转
	- 使用对象时, 由主动new产生对象转换为由**外部**提供对象, 此过程对象的创建控制权由程序转移到外部, 这种思想称为**控制反转**
	- 通过控制反转IoC, 实现了解耦
- Spring技术对IoC思想进行了实现
	- Spring提供了一个容器, 称为**IoC容器**, 用来充当IoC思想中的**外部**
	- IoC容器负责对象的**创建, 初始化**等一系列工作, 被创建或被管理的对象在IoC容器中统称为**Bean**
- **DI(Dependency Injection)** 依赖注入
	- 在容器中建立bean与bean之间的**依赖关系**的整个过程, 称为**依赖注入**
	- 当欲使用一个bean时, 其依赖的bean也会一并被提供

- 目标 : 充分解耦
	- 使用IoC容器管理bean
	- 在IoC容器内将有依赖关系的bean进行关系绑定, 即依赖注入(DI)
- 最终效果
	- 使用对象时不仅可以直接从IoC容器中获取, 而且获取到的bean已经绑定了所有的依赖关系

## IoC入门案例

**思路分析**
1. 管理什么 (Service与Dao)
2. 如何设置IoC管理对应的对象 (配置方式)
3. 如何得到IoC容器 (接口)
4. IoC得到后, 如何从容器中获取bean (接口方法)

**使用Spring流程**
1. 使用Spring导入的坐标
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```

2. 书写对应配置文件(XML版)
	- 配置时不配接口, 而是配实践类
```xml
<bean id="bookDao" 
	  class="top.majinliang.dao.impl.BookDaoImpl" />  
<bean id="bookService" 
	  class="top.majinliang.service.impl.BookServiceImpl" />
```
- *`<bean>` - 配置bean*
- *id属性标识bean的名字* - id不能重复
- *class属性给bean定义类型*

3. 获取IoC容器
```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

4. 获取Bean
```java
BookDao bookDao = (BookDao) context.getBean("bookDao");
```

## DI入门案例
- 作用 : 减少普通IoC的耦合

1. 基于IoC管理bean
2. Service中使用new形式创建的Dao对象是否保留 (否)
3. Service中需要的Dao对象如何进入到Service中 (提供方法)
4. Service与Dao之间的关系如何描述 (通过配置文件)

**步骤**
1. 删除业务层中使用new方式创建的dao对象
2. 提供对应的**set方法**
	- set方法在容器DI时使用
```java
public void setBookDao(BookDao bookDao){  
    this.bookDao = bookDao;  
}
```
3. 修改配置文件, 建立service和dao之间的关系
	- name表示接口的名称, 即set中参数的名称
	- ref表示xml中配置的bean的id名称
```xml
<!-- 3. dao放到Service中, 修改Service -->  
<bean id="bookService" 
	  class="top.majinliang.service.impl.BookServiceImpl">  
    <!-- 配置service与dao的关系 -->  
    <!-- property表示配置当前bean属性 -->
    <!-- name属性表示配置哪一个具体的属性 -->
    <!-- ref属性表示参照哪一个bean -->
    <property name="bookDao" ref="bookDao"/>  
</bean>
```

**使用实例**
```java
public class BookServiceImpl implements BookService {  
    // 1. 删除业务层中使用new方式创建的dao对象  
    // 由DI自动提供
    private BookDao bookDao;  
    public void save(){  
        System.out.println("book service save...");  
        bookDao.save();  
    }  
    // 2.提供对应的set方法  
    public void setBookDao(BookDao bookDao){  
        this.bookDao = bookDao;  
    }  
}
```

# Spring Bean

## bean基础配置
- 之前案例的配置方法
```xml
<bean id="bookDao" 
	  class="top.majinliang.dao.impl.BookDaoImpl" 
	  />  
<bean id="bookService" 
	  class="top.majinliang.service.impl.BookServiceImpl">  
    <property name="bookDao" ref="bookDao"/>  
</bean>
```

## bean别名配置

- 可以在bean标签内使用`name属性`配置别名
- 多个别名之间使用`空格`, `逗号`或`分号`分隔
- name可以和id一样, 用来寻找和连接对应的bean
`<bean id="bookDao" name="dao bookDao2" class="top.majinliang.dao.impl.BookDaoImpl" />`

## bean作用范围

- Spring默认创建的Bean对象是单例的
- 用于控制IoC创建的某个Bean的多个对象是不是同一个实例
- 改变**作用范围** - 使用属性 `scope`
	- singleton : 单例
	- prototype : 非单例

**非单例的Bean对象, Spring不会再管理其销毁和回收, 所以不会再调用destroy方法**
```java
<bean class="top.majinliang.dao.impl.BookDaoImpl" id="bookDao" scope="prototype"/>
```

**问题**
- 为什么bean默认为单例
	- 对于可以复用的对象, 只创建一次对象, 即提升了速度, 又减小了内存压力
- **适合**交给Spring容器管理的bean
	- 表现层对象
	- 业务层对象
	- 数据层对象
	- 工具对象
- **不适合**交给Spring容器管理的bean
	- 封装实体的域对象 - 有状态的对象

## bean实例化

**不管使用什么方法实例化bean, 创建对象的方法都是相同的**
bean共有三种实例化方法:
- 构造方法实例化
- 静态工厂实例化
- 实例工厂实例化

### 构造方法

- bean本质上就是对象 , 创建bean使用**构造方法**完成
- Spring会**自动执行**bean对应的构造方法
	- 不管是**公共还是私有**都能用到, 使用了**反射**的原理
	- spring创建bean调用的是**无参**构造方法
- 使用构造方法构造需要提供**可访问的构造方法**
	- 如果无参构造方法不存在, 会抛出异常
- 配置文件与之前类似

**报错信息阅读**
- 首先看最后一个报错信息, 观察是否能解决
- 如果无法解决, 那么就看上一个报错, 上一个报错会包含他的底层报错的信息
- 错误越向上, 就对错误的表述越封装

### 静态工厂(了解)

- bean的class要指定工厂类
- 工厂的get方法是一个**静态方法**
- 配置时要指定工厂的get方法, 否则创建的就是一个工厂对象
- 执行时会顺带着执行工厂get方法中的其他语句以对对象做一些必要处理

**静态工厂**
```java
public class OrderDaoFactory {  
    public static OrderDao getOrderDao(){  
        System.out.println("factory setup....");  
        return new OrderDaoImpl();  
    }
}
```

**配置方法**
- 通过`factory-method`指定工厂的get方法
```xml
<bean id="orderDao" 
	  class="top.majinliang.factory.OrderDaoFactory" 
	  factory-method="getOrderDao"
	  />
```

**创建对象**
```java
public static void main(String[] args) {  
    ApplicationContext ctx = 
    new ClassPathXmlApplicationContext("applicationContext.xml");  
    BookDao bookDao = (BookDao) ctx.getBean("bookDao");  
    bookDao.save();  
}
```

### 实例工厂

- 实例工厂和静态工厂是不同的
- 首先要分别构建工厂的bean和要被实例化的bean
	- 工厂bean的目的就是为了制造实际bean, 无其他意义
	- 所以后续以Factory Bean的方式进行了改良
- 实例bean中**不写class** , 而是分别写`factory-method`和`factory-bean`
	- factory-bean : 工厂bean名称
	- factory-method : 工厂bean的创建对象方法名称

**配置方法**
```xml
<!--方式三：使用实例工厂实例化bean-->  
<bean id="userFactory" 
	  class="top.majinliang.factory.UserDaoFactory"
	  />  
<bean id="userDao" 
	  factory-method="getUserDao" 
	  factory-bean="userFactory"
	  />
```

#### Factory Bean(重要)

**步骤**
1. 创建对应的类, 实现**FactoryBean接口**
2. 在泛型中填入工厂创建的Bean
3. 重写FactoryBean的方法
	- getObject : 实例工厂中创建对象的方法
	- getObjectType : 返回工厂创建对象的类型

**作用**
- 简化实例对象的配置过程

**配置文件**
- 配置文件中的class填FactoryBean的名称
```xml
<bean id="userDao" 
	  class="top.majinliang.factory.UserDaoFactoryBean"
	  />
```

**示例代码**
```java
public class UserDaoFactoryBean implements FactoryBean<UserDao> {  
    //代替原始实例工厂中创建对象的方法  
    public UserDao getObject() throws Exception {  
        return new UserDaoImpl();  
    }  

	// 返回工厂创建对象的类型
    public Class<?> getObjectType() {  
        return UserDao.class;  
    }  
}
```

**修改作用范围**
- FactoryBean创建的对象默认为单例模式
- 如果想要修改FactoryBean的作用范围为非单例, 则需要重新定义接口内函数
	- `isSingleton` - 设置单例或非单例
	- 返回true - 单例
	- 返回false - 非单例
```java
public boolean isSingleton() {
	return false;
}
```

## bean生命周期

- 生命周期 : 从创建到消亡的完整过程
- bean生命周期 : bean从创建到销毁的整体过程
- bean生命周期控制 : 在bean创建后到销毁前做一些事情

 - 初始化容器
	1. 创建对象(内存分配)
	2. 执行构造方法
	3. 执行属性注入(set操作)
	4. 执行bean初始化方法
		- 即`context.getBean`的使用
- 使用bean
	1. 执行业务操作
- 关闭/销毁容器
	1. 执行bean销毁方法

### bean销毁方法
- 由于虚拟机退出时没有主动关闭 `ApplicationContext`, 所以不会自动调用bean销毁方法
1. 主动调用 `ApplicationContext` 的`close`方法
	- `ApplicationContext` 中不存在`close`方法, 但是其实现类 `ClassPathXmlApplicationContext` 中存在`close`方法
	- 所以应该把原先获取bean的IoC容器类型由接口 `ApplicationContext` 改变为 `ClassPathXmlApplicationContext`
	- 相对来说比较暴力
	- 必须在容器使用完毕后才能关闭容器
2. 注册关闭钩子, 提醒虚拟机关闭前记得先关闭该容器
	- `context.registerShutdownHook();`
	- 注册关闭钩子的方法同样只有 `ClassPathXmlApplicationContext` 实现类中才存在
		- 其实从接口`ConfigurableApplicationContext`中就存在了
	- 注册关闭钩子在任何时间设置都可以, 不需要等待容器使用完毕
3. 这两种方法都**不需要深入了解**, 以后可以自动执行

### 生命周期相关配置

- 可以直接在bean对应的类中实现 `InitializingBean` 和 `DisposableBean` 接口 
- 实现其中的两个方法以做到初始化和销毁相关操作
	- afterPropertiesSet 方法
	- destroy 方法
- 如果xml中配置过后依然实现接口, 并不会报错, 而是同时执行这两个方法
- InitializingBean 中的 afterPropertiesSet 在属性设置完成后执行
	- 即如果该bean需要设置属性, 那么 afterPropertiesSet 会在该函数执行后执行
	- 对于使用xml定义的init, 执行顺序和 afterPropertiesSet 一样, 而且在 afterPropertiesSet 后执行

**建议使用了一个另一个就不使用**

**定义**
1. 在bean对应的类中添加对应的init和destory方法
	- init - 初始化
	- destory - 销毁
2. 在bean配置文件中添加对应的属性
	- init-method : 初始化函数
	- destroy-method : 销毁函数

**示例**
```xml
<bean id="bookDao"  
      class="top.majinliang.dao.impl.BookDaoImpl"  
      init-method="init"  
      destroy-method="destory"  
/>
```

```java
public class BookDaoImpl implements BookDao {  
    public void save() {  
        System.out.println("book dao save...");  
    }  
    // bean初始化对应的操作  
    public void init(){  
    }  
    // bean销毁前对应的操作  
    public void destory(){  
    }  
}
```

# 依赖注入

## 依赖注入方式

- 向类中传递数据的方式
1. 普通方法 (set方法)
2. 构造方法

- bean中传递数据的类型
1. 引用类型
2. 简单类型(基本数据类型 / String)

### setter方式注入

**注入形式相对灵活, 不需要注意其先后顺序**

##### 引用类型

- 在bean中定义引用类型属性并**提供可访问的set方法**
- 配置中使用**property标签**的**ref属性**注入引用对象
- 一个bean内可以使用多个property标签依赖多个bean

##### 直接类型

- 不同于property中的ref, 如果给直接类型赋值, 那么需要使用value属性, 并设置其属性值
- value会根据对应的值进行封装与识别, 书写时要写字符串

```xml
<bean id="bookDao" class="top.majinliang.dao.impl.BookDaoImpl">  
    <property name="databaseName" value="mysql"/>  
    <property name="connectionNumber" value="10"/>  
</bean>
```

### 构造器方式注入

**其注入形式在xml中同样没有先后限制**

**标准使用**
- 和property差不多
1. 创建好对应属性的构造器
2. 使用`constrcutor-arg`标签
	- name : **构造器** (注意与配置文件中名称无关) 的形参 (耦合度相对较高)
	- ref : bean的name或id

- 创建多个bean作为依赖时, 和property类似, 只需要多写constructor-arg标签即可
- 简单类型也与property同理

```xml
<bean id="bookService" class="top.majinliang.service.impl.BookServiceImpl">  
    <constructor-arg name="bookDao" ref="bookDao" />  
    <constructor-arg name="userDao" ref="userDao" />  
</bean>
```

**解耦**
由于`name`与形参名必须相同的方式耦合度高, 所以还有解耦的方式
- 使用`type`属性确认标签与形参之间的对应关系, 解决了形参会变名的问题
	- 降低了与形参名称之间的耦合度
	- 但是只适合各类型只有一个的构造参数
```xml
<bean id="bookDao" class="top.majinliang.dao.impl.BookDaoImpl">  
    <constructor-arg type="int" value="12" />  
    <constructor-arg type="java.lang.String" value="majinliang" />  
</bean>
```

- 使用`index`属性, 将参数位置与标签对应起来
	- 使用位置解决参数匹配的耦合问题
```xml
<bean id="bookDao" class="top.majinliang.dao.impl.BookDaoImpl">  
    <constructor-arg index="0" value="12" />  
    <constructor-arg index="1" value="majinliang" />  
</bean>
```
- **以上两种都不是标准的用法**


### 依赖注入方式选择

1. **强制依赖**使用构造器进行, 使用setter注入有概率不进行注入导致null对象出现
2. **可选依赖**使用setter注入进行, 灵活性强
3. Spring框架**倡导使用构造器** , 第三方框架内部大多数采用构造器注入的形式进行数据初始化, **相对严谨**
4. 如有必要可以两者同时使用, 使用构造器完成强制注入, 使用setter注入完成可选注入
5. 实际开发过程中还要根据实际情况分析, 如果受控对象没有提供setter方法则必须使用构造器注入
6. **自己开发的模块推荐使用setter注入**

### 依赖自动装配

- IoC容器根据bean所依赖的资源在容器中自动查找并注入到bean的过程称为**自动装配**
- 只适用于引用类型, 不能对简单类型进行操作
- 优先级低于setter注入和构造器注入, 与前两者同时出现时自动装配失效

**自动装配方式**
1. 按类型(**常用**)
2. 按名称
3. 按构造方法
4. 不启用自动装配

**注意**
-  虽然是自动装配, 但是bean的类中依然需要set方法
- 使用时尽可能使用按类型自动装配(避免耦合度过高)
1. 按类型装配 - (保证容器中相同类型的bean唯一, 即不能同时出现bookDao和bookDao2)
	- 按类型装配的自动装配中, 要声明好要被装配的bean
	- 同一个类文件被两个bean声明, 同样会导致自动装配失败
	- 如果没有id但是类型正确, 自动装配可以顺利进行
```xml
<bean id="bookService" 
	  class="top.majinliang.service.impl.BookServiceImpl" 
	  autowire="byType" 
/>
```
2. 按名称匹配 - (保证容器中有指定名称的bean)
	- 要保证被匹配的bean有一个名称和类中的依赖名称相同
	- 类中查找需要什么依赖, 使用的是类中的setXxx(), 即set方法进行匹配的, 如果依赖项名称与setter方法名称不匹配, 则会报错

**特征**
1. 自动装配用于**引用类型**依赖注入, 不能对简单类型进行操作
2. 使用按类型装配时, 要求容器中相同类型的bean唯一
3. 使用按名称装配时, 邀请容器中就有指定名称的bean(耦合较高, 不推荐使用)
4. 自动装配优先级低于setter注入和构造器注入, 同时出现时自动装配配置失效


### 集合注入
**很少用**

使用`<property>`标签标识

- 数组
	- `<array>`表示数组, `<value>`表示其中值
```xml
<property name="it_array">  
    <array>        
		    <value>100</value>  
        <value>200</value>  
        <value>300</value>  
    </array>
</property>
```
- List
	- `<list>`表示列表, `<value>`表示其中值
```xml
<property name="it_list">  
    <list>        
		    <value>itheima</value>  
        <value>itcast</value>  
        <value>majinliang</value>  
    </list>
</property>
```
- Set
	- `<set>`表示集合, `<value>`表示其中值
```xml
<property name="it_set">  
    <set>        
		    <value>majinliang01</value>  
        <value>majinliang02</value>  
        <value>majinliang02</value>  
        <value>majinliang03</value>  
    </set>
</property>
```
- Map
	- `<map>`表示map表, `<entry>`表示其中值
	- `<entry>`是一个封闭标签, `key`和`value`属性分别代表键和值
```xml
<property name="it_map">  
    <map>        
		    <entry key="name"   value="majinliang" />  
        <entry key="age"    value="18" />  
        <entry key="gender" value="male" />  
    </map>
</property>
```
- Properties
	- `<props>`表示属性集, `<prop>`表示其中项
	- `<prop>`中key表示键, 其值写在标签内部
```xml
<property name="it_properties">  
    <props>        
		    <prop key="database">mysql</prop>  
        <prop key="username">root</prop>  
        <prop key="password">123456</prop>  
    </props>
</property>
```
- 引用类型(一般用不到)
	- `<ref>`标签表示引用类型对象
	- 其中bean属性填写对象类型的bean
```xml
<property name="it_array">  
    <array>        
        <ref bean="beanId" /> 
    </array>
</property>
```

# 数据源对象管理

**Druid**
1. 添加Druid对应的依赖
2. 添加其对应的bean
```xml
<!-- 管理DruidDataSource对象 -->  
<bean id="datasource" 
	  class="com.alibaba.druid.pool.DruidDataSource">  
    <property name="driverClassName" 
		      value="com.mysql.cj.jdbc.Driver"/>  
    <property name="url" 
		      value="jdbc:mysql://localhost:3306/spring_db"/>  
    <property name="username" value="root"/>  
    <property name="password" value="123456"/>  
</bean>
```

**c3p0**
1. 在Maven Respository中查看C3P0的坐标
	- [Maven Repository: Search/Browse/Explore (mvnrepository.com)](https://mvnrepository.com/)
2.  添加其对应的bean
**注: C3P0对象创建时就需要访问数据库, 而Druid只是设置了驱动, 所以C3P0会在创建时就会报错没有mysql connector j**

# 案例:加载properties文件

1. 在applicationContext.xml中开启context命名空间
`xmlns:context="http://www.springframework.org/schema/context"`
2. 在`xsi:schemaLocation`中添加以下位置
`http://www.springframework.org/schema/context`
`http://www.springframework.org/schema/context/spring-context.xsd`
3. 使用context空间添加properties文件
`<context:property-placeholder location="jdbc.properties"/>`
4. 在bean中使用properties中的属性
	- 使用`${}`占位符

#### 同时加载多个properties文件

- 在`location`属性中, 使用逗号`,`分隔两个`properties`文件
```xml
<context:property-placeholder location="jdbc.properties, jdbc2.properties" system-properties-mode="NEVER"/>
```
- 可以使用`classpath:*.properties`加载所有的properties文件
	- 只使用`classpath:*.properties` : 只能读取当前工程的所有properties文件(**标准格式**)
	- 使用`classpath*:*.properties` : 读取该工程以及其依赖的jar包中的properties文件

```xml
<!-- 只加载当前项目配置文件(标准格式) -->
<context:property-placeholder location="classpath:*.properties" system-properties-mode="NEVER"/>
<!-- 也加载当前项目以及依赖jar包配置文件 -->
<context:property-placeholder location="classpath*:*.properties" system-properties-mode="NEVER"/>
```

#### 不加载系统属性

- 如果直接在properties中书写username, 会和系统环境变量中的username, 即本机用户名冲突, 从而被系统用户名覆盖
- 如果想要自己定义的属性不被系统用户名覆盖, 可以在标签`context:property-placeholder`中定义属性`system-properties-mode="NEVER"`

# 容器

## 创建容器

**加载容器的方法**
```java
// 1. 加载类路径下配置文件  
ApplicationContext context = new ClassPathXmlApplicationContext("Context.xml");  
// 2. 通过文件系统下加载配置文件  
ApplicationContext context = 
new FileSystemXmlApplicationContext("D:\\program\\softwareProgram\\JavaProject\\JavaWebItHeima\\SSM\\spring-10-container\\src\\main\\resources\\Context.xml");
```

**加载多个配置对象(了解即可)**
`ApplicationContext context = ClassPathXmlApplicationContext("Context1.xml", "Context2.xml");`

## 获取Bean

**使用Bean名获取后强转**
`BookDao bookDao = (BookDao) context.getBean("bookDao");`

**利用反射的不需要强转的Bean获取方法**
`BookDao bookDao = context.getBean("bookDao", BookDao.class);`

**利用自动装配的获取Bean方法**
- 要求这种情况容器中的同一类型的bean只能有一个
`BookDao bookDao = context.getBean(BookDao.class);`

## BeanFactory接口

- 使用 Ctrl + H 查看该类的继承关系图
- 为ApplicationContext接口的最顶级接口, 同样也可以用来创建容器
- ApplicationContext的功能比BeanFactory更多

**区别**
- BeanFactory是延迟加载类 , 在调用获取Bean方法时才会进行构造
- ApplicationContext是立即加载类
	- 如果想要延迟加载, 那么需要在bean的标签中添加lazy-init属性
```xml
<bean id="bookDao" class="top.majinliang.dao.impl.BookDaoImpl" lazy-init="true">  
    <property name="name" value="mysql"/>  
</bean>
```

**示例**
```java
Resource resources = new ClassPathResource("Context.xml");  
BeanFactory beanFactory = new XmlBeanFactory(resources);
```

### 容器类结构层次图

![[后端/SSM/01-Spring/Inbox/Pasted image 20240206221128.png]]

# 总结

## 容器相关

- **BeanFactory**时IoC容器顶级接口, 初始化BeanFactory对象时, 加载的bean延迟加载
- **ApplicationContext**接口是Spring容器的核心接口, 初始化时bean立即加载
- ApplicationContext接口提供基础的**bean操作相关方法**, 通过其他接口**扩展**其功能
- ApplicationContext接口常用初始化类
	- ClassPathXmlApplicationContext : 根据类名进行初始化
	- FileSystemXmlApplicationContext : 根据文件路径进行初始化

## bean相关

```xml
<bean 
	id="bookDao"
	name="dao bookDaoImpl daoImpl"
	class="com.itcast.dao.impl.BookDao.Impl"
	scope="singleton"
	init-method="init"
	destroy-method="destroy"
	autoWire="byType"
	factory-method="getInstance"
	factory-bean="com.itcast.factory.BookDaoFactory"
	lazy-init="true"
/>
```

**解释**
- id -> bean的id
- name -> bean的别名
- class -> bean类型, 静态工厂类, FactoryBean类
- scope -> 控制bean的实例数量
- init-method -> 生命周期初始化方法
- destroy-method -> 生命周期销毁方法
- autowire -> 自动装配类型
- factory-method -> bean工厂方法, 应用于静态工厂或实例工厂
- factory-bean -> 实例工厂bean
- lazy-init -> 控制bean延迟加载

## 依赖注入相关

- 构造器注入引用类型 : `<constructor-arg name="" ref="" >`
- 构造器注入简单类型 : `<constructor-arg name="" value="">`
- 类型与索引装配 : 
	- `<constructor-arg name="" autoware="byType">`
	- `<constructor-arg name="" autoware="byIndex">`
- setter注入引用类型
- setter注入简单类型
- setter注入集合类型
	- list集合
		- 集合注入简单类型
		- 集合注入引用类型


