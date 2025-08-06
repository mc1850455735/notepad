# SSM整合

- 整合, 就是将以前学的 `Spring` , `SpringMvc` , `Mybatis` 整合在一起

### 流程

1. 创建工程
2. SSM整合
	- Spring
		- SpringConfig : 设置其他层路径 (除表现层)
	- SpringMVC
		- ServletConfig : 设置使用的配置文件, 要拦截的访问路径以及过滤器
		- SpringMvcConfig : 设置表现层扫描路径
	- MyBatis
		- MybatisConfig : 设置Mybatis扫描路径以及包路径
		- JdbcConfig : 设置使用的DataSource的bean
		- jdbc.properties : 设置数据库相关属性
3. 功能模块
	- 表与实体类
	- dao (接口 + Mybatis自动代理)
	- service (接口 + 实现类)
		- 业务层接口测试 ( 整合JUnit )
	- controller
		- 表现层接口测试 ( Postman )

### 父子容器

- 在ServletConfig中进行的设置, 将 SpringMvcConfig 设置到`getServletConfigClasses()`函数的返回值中, 而将 SpringConfig 设置到 `getRootConfigClasses()` 中
- SpringMvc可以访问到Spring中的内容, 但是Spring无法访问到SpringMvc中的内容
	- 这主要涉及到父子容器的相关概念

### 注意事项

##### 各层配置
- 通常情况下, 不推荐将dao层方法名直接作为service层方法名, 因为service层通常需要可以使用户见名知意
- 同时, service层与dao层不一致的是, 对于dao层返回值是void的某些操作, service层通常要返回boolean类型来提醒用户操作成功/失败
- 同时, 对于service层, 为了便于使用, 要添加注释说明每个方法的作用等

##### idea报错
- 在ssm整合过程中, 对于dao层, 可能有部分操作是通过自动代理接口的形式完成的, 于是该接口不存在对应的实现类. 在你尝试通过自动装配形式注入相关对象时, idea会报错不存在对应的bean, 但是实际上逻辑和行为都是正常的
- 解决 : 重新设置idea的错误配置, 使idea忽略该错误或降低该错误的报错等级

**不要忘记添加事务**
- 首先在`SpringConfig`中添加注解`@EnableTransactionManagement`
- 其次在`JdbcConfig`中添加bean用来切实管理事务
```java
@Bean  
public PlatformTransactionManager transactionManager(DataSource dataSource) {  
    DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();  
    transactionManager.setDataSource(dataSource);  
    return transactionManager;  
}
```
- 最后在业务层接口中加`@Transaction`
```java
@Transactional  
public interface BookService {
		...
}
```

# 表现层数据封装

### 前端接收数据格式 

- 创建结果模型类
	- 封装数据到data属性中
	- 封装操作以及操作结果到code中 - 假设尾数0为失败, 1为成功
	- 封装特殊消息到message(msg)中

```json
{
	"code": 20040,
	"data": null,
	"msg": "操作失败, 请重试"
}
{
	"code": 20031,
	"data": [
		{...},
		{...}
	]
}
```

### 设置统一返回结果类

##### Result
- 通常情况下, 由于Result类由Controller使用, 所以定义在Controller包下
- 为了方便使用, 通常为Result设置各式各样的构造函数来满足使用中的需求
```java
public class Result{
		private Object data;
		private Integer code;
		private String msg;
		
		public Result() {  
		}  
		public Result(Object data, Integer code) {  
		    this.data = data;  
		    this.code = code;  
		}  
		public Result(Object data, Integer code, String msg) {  
		    this.data = data;  
		    this.code = code;  
		    this.msg = msg;  
		}
}
```

**注意**
- Result类中的字段并不是固定的, 可以根据需要自行删减
- 提供若干个构造方法, 方便操作

##### Code
- 用来存储各种代码以及其代表的意义
```java  
public class Code {  
    public static Integer SAVE_OK = 20011;  
    public static Integer DELETE_OK = 20021;  
    public static Integer UPDATE_OK = 20031;  
    public static Integer GET_OK = 20041;  
    public static Integer SAVE_ERR = 20010;  
    public static Integer DELETE_ERR = 20020;  
    public static Integer UPDATE_ERR = 20030;  
    public static Integer GET_ERR = 20040;  
}
```

### 封装方式
- 对于一个使用Result和Code组织的Controller方法, 通常采取如下书写, 返回值为Result
- 根据情况设定合理的Result
```java
@GetMapping("/{id}")  
public Result getById(@PathVariable Integer id) {  
    Book book = bookService.getById(id);  
    Integer code = book != null ? Code.GET_OK : Code.GET_ERR;  
    String msg = book != null ? "" : "数据查询失败, 请重试";  
  
    return new Result(code, book, msg);  
}
```

# 项目异常处理方案

- 程序开发过程中不可避免会遇到异常现象
- 所有的异常均一级级往上抛, 抛到表现层中进行统一处理
	- 表现层处理异常, 如果每一层都单独书写异常处理代码, 代码量巨大, 相似度高且意义不强
	- 所以使用aop思想, 将高度集中的代码汇集到一起统一处理
	- 由此, 得到异常处理器

**出现异常的常见位置和常见诱因**
- 框架内部抛出的异常 : 因使用不合规导致
- 数据层抛出的异常 : 因外部服务器故障导致 (例如: 服务器访问超时)
- 业务层抛出的异常 : 因业务逻辑书写错误导致 (例如: 遍历业务书写操作, 导致索引异常等)
- 表现层抛出的异常 : 因数据收集, 校验等规则导致 (例如: 不匹配的数据类型间导致异常)
- 工具类抛出的异常 : 因工具类书写不严谨不够健壮导致 (例如: 必要释放的连接长期未释放)

### 异常处理器
- 异常处理器类位于controller包内, 会拦截异常, 运行用户对拦截的异常进行处理, 并返回对应的数据
- 使用rest风格开发时, 在异常处理器类上加注释 `@RestControllerAdvice`
	- 此注解自带`@ResponseBody`和`@Component`
- 在对应的异常处理函数上加注解`@ExceptionHandler(异常类型)`
- 可以根据异常的不同制作多个方法并处理对应的异常
```java
@RestControllerAdvice  
public class ProjectExceptionAdvice {  
    @ExceptionHandler(Exception.class)  
    public Result doException(Exception ex) {  
        System.out.println("我捉到, 神奇异常了!");  
        return new Result(666, null, "出异常了");  
    }  
}
```

### 异常产生原因

##### 项目异常分类
- 业务异常
	- 不规范的用户操作产生的异常
	- 规范的用户行为产生的异常
- 系统异常
	- 项目运行过程中**可预计**且无法避免的异常 (如 数据库服务器宕机等)
- 其他异常
	- 编程人员未预计到的其他异常

##### 项目异常处理方案
- 业务异常
	- 发送对应消息传递给用户, 提醒规范操作
- 系统异常
	- 发送固定消息传递给用户, 安抚用户
	- 发送特定消息给运维人员, 提醒维护
	- 记录日志
- 其他日常
	- 发送固定消息传递给用户, 安抚用户
	- 发送特定消息给编程人员, 提醒维护 (纳入预期范围内 作为业务异常或系统异常)
	- 记录日志

### 异常处理

##### 创建对应异常类
- 创建一个名为exception的包, 在其中定义 `SystemException` 类, 并使该类继承 `RuntimeException`
	- 运行时异常 (RuntimeException) 可以出现后不处理, 使其自动向上抛. 避免书写大串的 throw
- 在类中添加code, 方便表明是哪一种异常, 同时覆写RuntimeException所有的构造方法 (也可以用哪个写哪个, 把不需要的删除)
- BusinessException同理
```java
public class SystemException extends RuntimeException{  
    private Integer code;  

		public Integer getCode() {  
		    return code;  
		}  
		public void setCode(Integer code) {  
		    this.code = code;  
		}
    public SystemException(Integer code, String message) {  
        super(message);  
        this.code = code;  
    }  
    public SystemException(Integer code, String message, Throwable cause) {  
        super(message, cause);  
        this.code = code;  
    }  
}
```

##### 包装可能出现的异常
- 为Code添加对应的错误类型, 替代直接书写错误码的方式, 增强程序的可读性和可靠性
```java
public static Integer SYSTEM_ERR = 50001;  
public static Integer SYSTEM_TIMEOUT_ERR = 50002;
public static Integer SYSTEM_UNKNOWN_ERR = 59999;
public static Integer BUSINESS_ERR = 60002;
```
- 使用try-catch模拟包装 系统异常
```java
try {  
    int i = 1 / 0;  
} catch (Exception ex) {  
    throw new SystemException(Code.SYSTEM_TIMEOUT_ERR, "服务器访问超时, 请重试!", ex);  
}
```
- 使用if模拟包装 业务异常
```java
if (id == 1) {  
    throw new BusinessException(Code.BUSINESS_ERR, "请不要使用你的技术挑战我的极限");  
}
```

##### 添加对应的处理方式
- 处理系统异常
```java
@ExceptionHandler(SystemException.class)  
public Result doSystemException(SystemException ex) {  
    // 记录日志  
    // 发送消息给运维  
    // 发送邮件给开发人员  
    // 返回消息  
    return new Result(ex.getCode(), null, ex.getMessage());  
}
```
- 处理业务异常
```java
@ExceptionHandler(BusinessException.class)  
public Result doBusinessException(BusinessException ex) {  
    // 返回消息  
    return new Result(ex.getCode(), null, ex.getMessage());  
}
```
- 处理没有被预料到的其他异常 ( 安抚用户, 加急处理 )
```java
@ExceptionHandler(Exception.class)  
public Result doException(Exception ex) {  
    // 记录日志  
    // 发送消息给运维  
    // 发送邮件给开发人员  
    // 返回消息  
    return new Result(Code.SYSTEM_UNKNOWN_ERR, null, "系统繁忙, 请稍后再试(安抚用户)");  
}
```

# 案例：SSM整合标准开发

- 为判断dao操作是否成功, 需要直接返回 行计数 , 用来判断是否操作成功
- service层通过判断受影响的行, 返回true或false
```java
public boolean delete(Integer id) {  
    return bookDao.delete(id) > 0;  
}
```
