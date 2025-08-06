**什么是AOP**
- AOP(Aspect Oriented Programming) 面向切面编程, 一种**编程范式**, 指导开发者如何组织程序结构
- 作用 : 在**不惊动原始设计**的基础上为其进行功能增强
- Spring理念 : 无入侵/侵入式编程

# 核心概念

![[后端/SSM/01-Spring/Inbox/Pasted image 20240208152233.png]]

- 连接点(Join Point) : 程序执行过程中**任意位置**, 粒度为执行方法, 抛出异常, 设置变量等
	- 在`SpringAOP中`, 理解为方法的执行
- 切入点(PointCut) : 匹配连接点的式子
	- 在SpringAOP中, 一个切入点可以只描述一个具体方法, 也可以匹配多个方法
		- 一个具体方法 : dao包下BookDao接口中的无形参无返回值的save方法
		- 匹配**多个方法** : 所有的save方法, 所有get开头的方法, 所有以Dao结尾的接口中的任意方法, 所有带有一个参数的方法 等
- 通知(Advice) : 在切入点执行的操作, 也就是**共性功能**
	- 在SpringAOP中, 功能最终以方法的形式呈现
- 通知类 : 定义通知的类
- 切面(Aspect) : 描述通知与切入点的对应关系

# AOP入门案例

**案例设定**
- 要求接口执行前输出当前系统时间
- 开发模式 : **注解**
**思路分析**
1. 在pom中导入坐标
2. 制作连接点方法(原始操作, Dao接口与实现类)
3. 制作共性功能(通知与通知类)
4. 定义切入点
5. 绑定切入点与通知关系(切面)

## 步骤
1. 导入依赖 : context导入, aop**默认**自动导入
   - 导入aspect包
```xml
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
    <version>1.9.4</version>  
</dependency>
```
2. 抽出共性的功能 (一个公有的方法)
3. 定义切入点
	 - 定义私有的方法, 切入点依托这个不具有实际意义的方法进行, 该方法为一个私有的, 无参数和返回值的, 无实际逻辑的空方法
	 - 添加切入点注解`@Pointcut` , 为注解添加对应的方法
		 - `@Pointcut("execution(void top.majinliang.dao.BookDao.update())")`
4. 绑定共性功能和切入点的关系
   - 在共性方法上添加注解`@Before("私有方法名")`
5. 为Advice添加注解
   - `@Component`
   - `@Aspect`
```java
@Component  
@Aspect  
public class MyAdvice {  
    @Pointcut("execution(void top.majinliang.dao.BookDao.update())")  
    private void pt(){}  
    
    @Before("pt()")  
    public void method() {  
        System.out.println(System.currentTimeMillis());  
    }  
}
```
6. 在配置类中添加注解 `@EnableAspectJAutoProxy` , 告诉Spring项目中有注解开发的Aspect
	 - 同时要注意配置类也要扫描aop包

# AOP工作流程

**概念**
1. 目标对象(Target) : **原始功能去掉共性功能**对应的类产生的对象, 这种对象是无法直接完成最终工作的
2. **代理(Proxy)** : 目标对象无法直接完成工作 , 需要**对其进行功能回填** , 通过原始对象的**代理对象**实现
	- 使用代理对象来代替对真实对象的访问，这样就可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能
	- 使用了**CGLIB 动态代理机制**

**流程**
1. Spring容器启动
2. 读取所有切面**配置中的**切入点
	 - 只有定义后使用的切入点才会读取
3. 初始化bean, 判定bean对应的类中的**方法**是否**匹配到**任意切入点(bean中的方法有没有和切入点匹配的)
	 - 匹配失败, 创建对象
	 - 匹配成功, 创建原始对象(**目标对象**)的**代理**对象
4.  获取bean执行方法
	 - 获取bean, 调用方法并执行, 完成操作
	 - 如果bean是代理对象, 根据代理对象的运行模式运行原始方法与增强过的内容
		 - 代理对象的类型是`class com.sun.proxy.$Proxy19`
		 - 之所以代理对象打印出来和普通对象类型一样, 是因为代理对象对toString函数进行了重写

# AOP切入点表达式

- 切入点 : 要进行增强的方法
- 切入点表达式 : 要进行增强的方法的描述方式

**两种描述方式**
- 精准匹配
- 通配匹配

### 精准匹配

`动作表达式 (访问修饰符 返回值 包名.类/接口名.方法名(参数)异常名)`
`execution(public User com.itheima.service.UserService.findById(int))`

**注意**
- 访问修饰符和异常名 **可以省略**
- 可以用接口名, 也可以用实现类名(如下)
	- `execution(void com.itheima.dao.BookDao.update())`
	- `execution(void com.itheima.dao.impl.BookDao.update())`

### 通配匹配
- 可以使用通配符描述切入点
	- `*` : 单个独立的任意符号, 可以独立出现, 也可以作为前缀或者后缀的范围匹配符出现
		- `*` 在参数时, 表示带有且只带有一个任意类型参数, 否则无法匹配
	- `..` : 多个连续任意符号, 独立出现 ,通常用于简化包名与参数的书写
		- 用于路径时, 表示该路径下可能有一个或多个包或类
		- 用于参数时, 表示任意参数, 可以带参, 也可以不带参
	- `+` : 专用于匹配子类类型, 表示匹配这个类的子类或接口的实现类
		- `execution(* *..*Service+.*(..))`
		- 表示任意返回值的, 任意包下的任意子路径的任何一个以Service结尾的类的子类的任意方法
- 正着不好识别时, 可以**反向观察**通配字符串描述的是哪个方法

```java
execution(public * com.itheima.*.UserService.find* ( * ) )
execution(public User com..UserService.findById (..) )
execution(* com..*Service+.find* (..) )
```

## 书写技巧

1. 所有代码必须**按照规范**开发
2. 描述切入点**通常描述接口**而不是类
3. 访问控制修饰符针对接口开发均采用public (访问控制修饰符**可省略**)
4. **返回值类型**对于**增删改类使用精准**类型加速匹配, **查询类使用 `*`** 快速匹配
5. 包名尽量不使用 .. 匹配, 效率过低, 通常只对单个包使用 * 描述或者直接精准匹配
6. **接口名**/类名书写与**模块**相关的使用 * 匹配, 如`UserService`写成`*Service`
7. **方法名**书写以**动词**进行精确匹配, **名词**采用 * 匹配, 如`selectById`写成`selectBy*`
8. 参数规则应该根据业务灵活调整
9. 通常**不使用异常**作为匹配规则

# AOP通知类型

- 根据共性功能抽取位置的不同, 运行代码时要加入合理的位置
- AOP通知分为5种类型
	- 前置通知 : `@Before`
	- 后置通知 : `@After`
	- **环绕通知** : `@Around` (重点)
		- 其中应该有一个表示对原始操作的调用
		- 在函数中添加参数`ProceedingJoinPoint point`
			- ProceedingJoinPoint对象中有getSignature方法, 可以获取这次**执行的签名信息**
			- **签名信息中**包含该对象的对象类型和方法
				- `getDeclaringTypeName()` : 获取调用者类型名
				- `getName()` : 获取被使用的方法名
		- 使用`point.proceed()`进行调用原始方法
			- 最后面应该接收原始方法的返回值并返回
			- 由于around方法是从方法内部手动调用原始方法而不是像其他类型那样自动调用并返回
	- 返回后通知(了解) : `@AfterReturning`
		- 原始方法成功运行以后调用
		- 方法**抛异常**就不会继续运行, 而after仍然会运行
	- 抛出异常后通知(了解) : `@AfterThrowing`
		- 只有方法抛异常才会运行

```java
@Around("pt2()")  
public Object around(ProceedingJoinPoint point) throws Throwable {  
    System.out.println("around before advice ...");  
    Object result = point.proceed();  
    System.out.println("around after advice ...");  
    return result;  
}
```

**@Around注意事项**
1. 环绕通知必须依赖形参ProceedingJoinPoint才能实现对原始方法的调用, 从而实现原始方法调用前后通知添加通知
2. 通知中如未使用ProceedingJoinPoint, 则将**跳过**原始方法的执行
3. 对原始方法的调用可以不接收返回值, 通知设为void即可 , 如果需要接收, 则**需要设置为Object**
4. 原始方法返回值如果是void, 通知方法返回值类型可以是void, 也可以是Object
5. 由于无法预知原始方法是否会抛出异常, 所以环绕通知方法**必须抛出**Throwable对象

**示例**
```java
@Around("pt2()")  
public Object aroundSelect(ProceedingJoinPoint point) throws Throwable {  
    System.out.println("before around advice");  
    Object ret = point.proceed();  
    System.out.println("after around advice");  
    return ret;  
}
```


# AOP获取数据

- 获取切入点方法的相关信息
	- `getSignature()` : 获取方法签名信息
		- `signature.getDeclaringType()` : 获取调用该方法的类型
		- `signature.getDeclaringTypeName()` : 获取调用该方法的类型名
		- `signature.getName()` : 获取该方法的名称
```java
@Around("MyAdvice.servicePt()")  
public void testFunction(ProceedingJoinPoint point) throws Throwable {  
    Signature signature = point.getSignature();  
    String typeName = signature.getDeclaringTypeName();  
    String functionName = signature.getName();  
		...
}
```

### 能够获取的类型

- 获取切入点方法的**参数**
	- JoinPoint : 适用于前置, 后置, 返回后, 异常后通知
	- ProceedJoinPoint : 适用于环绕通知
	- JointPoint是ProceedingJoinPoint的父类/父接口
- 获取切入点方法返回值 : 必须保证原始操作正常执行才能拿到返回值
	- 返回后通知
	- 环绕通知
	- 通过proceed函数获取的返回值类型为Object
- 获取切入点方法运行异常信息
	- 抛出异常后通知
	- 环绕通知

### 获取参数(重要)
- JoinPoint是ProceedJoinPoint的父接口
- `getArgs()` : 获取参数 , 其返回值是一个Object数组
	- 可以将getArgs获得的参数传入 `@Around` 的 `proceed` 方法中
		- 即使不传入 , 默认也会将args传入proceed
	- 可以通过传入与原args不相同的值来修改其参数
		- 可以做到对参数的**检查**, 提高程序的**健壮性** , 而且只需要一个AOP

```java
@Before("pt()")
public void before(JoinPoint point) {  
		// 获取参数信息
    Object[] args = point.getArgs();  
    System.out.println("before advice ...");  
    System.out.println(Arrays.toString(args));  
}

// 获取参数, 修改后重新传回, 达到检查参数目的
@Around("pt()")  
public Object around(ProceedingJoinPoint point) {  
    Object[] args = point.getArgs();  
    System.out.println(Arrays.toString(args));  
    args[0] = 60;  
    Object proceed = point.proceed(args);
    return proceed;  
}
```

### 获取返回值
- `@Around`中直接接收proceed的返回值, 即可获取原始方法的返回值
- `@AfterReturning`中 
	- 首先定义一个接收返回值的形参ret
	- 在`@AfterReturning`中 , 设置参数 : `returning="接收返回值形参名"`
		- `@AfterReturning(value="pt()", returning="ret")`
	- 形参ret通常使用Object类型, 也可以使用其他类型, 比如String, 但是如果类型不匹配可能会报错, 且可能需要类型转换, 所以推荐使用Object
- 如果JoinPoint和返回值的形参同时存在, **必须JoinPoint在最前** , 否则会报异常
```java
@AfterReturning(value = "pt()", returning = "ret")  
public void afterReturning(JoinPoint point, Object ret) {  
    Object[] args = point.getArgs();  
    System.out.println(Arrays.toString(args));  
    System.out.println(ret);  
    System.out.println("after returning advice ...");  
}
```

### 获取异常
- 在`@Around`中
	- 直接通过try-catch接收即可
- 在`@AfterThrowing`中
	- 首先定义一个接收异常的形参t
	- 设置`@AfterThrowing`中 , 设置参数 : `throwing="接收返回值形参名"`

```java
@AfterThrowing(value = "pt()", throwing = "t")  
public void afterThrowing(Throwable t) {  
    System.out.println("after throwing advice ...");  
    System.out.println(t);  
}
```

# 总结

- 使用AOP可以简化共性功能的开发
	- 如去掉密码前后的多余空格

- **概念**
AOP(Aspect Oriented Programming)面向切面编程
- **作用**
不惊动原始设计的基础上为方法进行功能增强
- **核心概念**
	- 代理(Proxy) : SpringAOP的核心本质是通过代理模式实现的
	- 连接点(JoinPoint) : 理解为任意方法的执行
	- 切入点(Pointcut) : 匹配连接点的式子, 具有共性功能的方法的描述
	- 通知(Advice) : 弓干个方法的共性功能, 切入点处执行, 体现为一个方法
	- 切面(Aspect) : 描述通知与切入点的关系
	- 目标对象(Target) : 被代理的原始对象成为目标对象
- **切入点**
	- `execution(* com.itheima.service.*Service.*(..))`
	- 通配符
		- `* , .. , +`
	- 切入点表达式书写**技巧**
- **通知类型**
	- 前置
	- 后置
	- 环绕 : 可以**模拟**其他四类所有通知
		- 依赖形参ProceedingJoinPoint实现对原始方法的调用
		- 环绕通知可以**隔离**原始方法的调用执行
		- 环绕通知返回值设置为**Object类型**
		- 环绕通知中可以对原始方法调用中出现的异常进行处理
	- 返回后
	- 抛出异常后
- **获取数据**
	- 参数
	- 返回值
	- 异常


