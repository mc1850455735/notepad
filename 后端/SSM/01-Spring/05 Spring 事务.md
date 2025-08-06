# 简介

- **事务作用**
	- 在数据层保障一系列的数据库操作同成功同失败
- Spring事务作用
	- 在数据层**或者业务层**保证一系列的数据库操作同成功同失败
	- Spring中使用`PlatformTransactionManager`类(平台事务管理器)实现了事务中的**commit**和**rollback**
	- Spring中使用`DataSourceTransactionManager`类(数据源食物管理器)实现了对事务的管理
- 不使用事务时, 如果处理数据中出现异常导致失败, 可能造成数据的不正确, 所以需要**引入事务**来保证数据的一致性
- Spring和Mybatis使用的都是JDBC的事务

**使用流程**
1. 在**业务层方法接口**上添加`@Transactional`注解 : 降低耦合
	- 也可以直接将事务接到接口或类上(大多数情况下, 应该写到接口上, 降低耦合), 表示接口内所有方法开启事务
```java
@Transactional  
void transfer(String out, String in, Double money);
```
2. 在配置文件中添加事务管理器
   - 在**Jdbc的配置类**中添加对应的bean定义
```java
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {  
    DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();  
    transactionManager.setDataSource(dataSource);  
    return transactionManager;  
}
```
3. 在SpringConfig中添加注解`@EnableTransactionManagement`
```java
@Configuration  
@ComponentScan("top.majinliang")  
@PropertySource("classpath:jdbc.properties")  
@Import({MybatisConfig.class, JdbcConfig.class})  
@EnableTransactionManagement  
public class SpringConfig {  
}
```

# 实现原理

- 原本的数据层操作应该有两个或多个事务(事务协调员)
- 在Spring对应方法加入`@Transactional`注解后, 相当于为该方法添加了一个事务(事务管理员), 囊括该方法所有的数据层操作
- 于是让原本数据层操作的事务改变为Spring该方法的事务
- 所以现在只有一个事务, 可以实现出错后的回滚
- 加入的要求是具有相同的数据源(dataSource)

**概念和注意事项**
- Spring开启的事务叫做**事务管理员**
	- 发起事务方 , 通常代指业务层**开启事务的方法**
- 加入Spring的事务称为**事务协调员**
	- 加入事务方 , 通常代指数据层方法 , 也可以是业务层方法
- Mybatis中的datasource要和**事务**中使用相同的datasource
- 事务不会受try-catch结构的影响, 一旦出现问题, 就会将事务回滚, 也并不会执行finally(就算真的执行了也会回滚掉)

# 相关配置
这些配置使Spring事务可以进行精细化管理, 在注解中进行开启

- readOnly : 设置是否为只读事务 - true为只读事务
- timeout : 设置事务超时时间 -  `-1` 为永不超时
- rollbackFor : 设置**事务回滚异常** , 指定遇到时需要回滚的异常
	- 默认只回滚Exception(运行时异常) , 而不会回滚Error
- rollbackForClassName : 根据类名设置事务回滚异常
- noRollbackFor : 设置事务不回滚异常
- noRollbackForClassName : 根据类名设置事务不回滚异常

```java
@Transactional(rollbackFor = {IOException.class})  
void transfer(String out, String in, Double money) throws IOException;
```

![[后端/SSM/01-Spring/Inbox/Pasted image 20250731010711.png]]

`try-finally`结构虽然可以保证finally内行为一定被实施, 但是由于事务未成功, 所以即使finally中的行为被实施, 也会被回滚

**事务传播行为**
- 事务协调员对事务管理员所携带事务的**处理态度**
- propagation : 设置事务传播行为

```java
public interface LogService {  
    @Transactional(propagation = Propagation.REQUIRES_NEW)  
    void log(String out, String in, Double money);  
}
```

**行为**
- Required : 事务管理员有事务, 则加入事务管理员, 否则新开一个事务
- Required_New : 无论事务管理员是否有事务, 都新开一个事务
- Supports : 如果事务管理员有事务, 则加入事务管理员, 否则不进行事务
- Not_Supported : 无论如何, 都不加入事务
- Mandatory : 事务管理员有事务, 则加入事务管理员, 否则报错
- Never : 如果事务管理员有事务, 则报错, 否则正常运行且没有事务
- Nested : 设置savePoint, 事务回滚时, 将回滚到savePoint处, 有客户响应提交或回滚

![[后端/SSM/01-Spring/Inbox/Pasted image 20240216001120.png]]























