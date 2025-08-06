
# Spring整合Mybatis

**思路分析**
- 分析可知, mybatis中最核心的对象是SqlSessionFactory
- mybatis中最核心的配置是初始化类型别名和初始化dataSource的部分

**需要额外添加的包**
*注意 :mybatis-spring 的版本和mybatis 的版本是对应的, 应该注意*
[mybatis-spring 与 mybatis 版本对照](https://mybatis.org/spring/zh/index.html)
```xml
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>1.3.0</version>  
</dependency>
```

**MyBatis-Spring的作用**
- Mybatis提供了与Spring整合的快速创建Bean的类, 叫做`SqlSessionFactoryBean`
- 其实现了FactoryBean接口, 可以通过他的方法快速获得bean对象
	- `SqlSessionFactoryBean`实现了FactoryBean的getObject方法

对于mapper自动扫描, Mybatis-Spring同样提供了对应的Bean

**配置示例**
```java
public class MybatisConfig {  
    //  整合mybatis配置
    @Bean  
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){  
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();  
        ssfb.setTypeAliasesPackage("top.majinliang.domain");        
        ssfb.setDataSource(dataSource);                             
        return ssfb;  
    }  

		// 设置扫描包的位置
    @Bean  
    public MapperScannerConfigurer mapperScannerConfigurer(){  
        MapperScannerConfigurer msc = new MapperScannerConfigurer();  
        msc.setBasePackage("top.majinliang.dao");  
        return msc;  
    }  
}
```

**使用实例**
```java
public static void main(String[] args) throws FileNotFoundException {  
    ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);  
    AccountService accountService = context.getBean(AccountService.class);  
    Account account = accountService.findById(1);  
    System.out.println(account);  
}
```

**注意**
- 使用Mybatis通过注解自动代理方式实现bean , 可以由此不扫描dao包就使用其中的bean, 但是这样不利于程序通用性
- 一旦不使用Mybatis , 就不能自动实现bean , 如此就不能不扫描dao包

整体需要添加的与数据库操作有关的包
```xml
<!-- 数据源相关jar包 -->  
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>1.1.16</version>  
</dependency>  
<!-- mysql依赖包 -->  
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <version>8.2.0</version>  
</dependency>  
<!-- mybatis相关包 -->  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis</artifactId>  
    <version>3.5.3</version>  
</dependency>  
<!-- spring操作数据库相关包 -->  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-jdbc</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>  
<!-- spring整合mybatis相关jar包 -->  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis-spring</artifactId>  
    <version>1.3.0</version>  
</dependency>
```

# Spring整合JUnit

1. 导入对应坐标(注意JUnit版本)
```xml
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>4.12</version>  
    <scope>test</scope>  
</dependency>
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>5.2.10.RELEASE</version>  
</dependency>
```
2. 在Test中设置类运行器, 设置上下文环境
	- `@RunWith(SpringJUnit4ClassRunner.class)` : 设置运行器环境
	- `@ContextConfiguration(classes = SpringConfig.class)` : 设置上下文
3. 书写代码, 进行测试, 测试方法需要添加`@Test`注解
4. 对要测试的bean进行注入

**示例**
```java
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(classes = SpringConfig.class)  
public class AccountServiceTest {  
    @Autowired  
    private AccountService accountService;  
  
    @Test  
    public void test(){  
        Account account = accountService.findById(1);  
        System.out.println(account);  
    }  
}
```

