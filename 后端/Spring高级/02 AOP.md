
# AOP的其他实现

### AOP的AJC实现

- aop的AJC实现不只是代理, 而是在编译期间直接对相关类字节码进行改写
- 这个过程与Spring的执行不存在关联, 只要制定了创建流程, 即使手动创建, 也会存在相关注入, 可以通过相关插件实现该过程
- 通过这种方式, 可以实现不通过Spring容器注入AOP
- 由于需要加入插件编译, 进行编译时建议使用maven的complie命令确保插件参与了编译过程 (1.14.4版本最高支持Java16, 建议使用Java8)
- 如果想要独立实现注入效果, 需要加入相关插件及其配置如下
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.14.0</version>
    <configuration>
        <complianceLevel>1.8</complianceLevel>
        <source>8</source>
        <target>8</target>
        <showWeaveInfo>true</showWeaveInfo>
        <verbose>true</verbose>
        <Xlint>ignore</Xlint>
        <encoding>UTF-8</encoding>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>test-compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### AOP的agent类加载实现

- agent类加载不在编译期间进行增强, 而是在类加载阶段完成对类的拓展
- 在VM options中加入 `-javaagent: %maven_repository% /org/aspectj/aspectjweaver/1.9.7/aspectjweaver-1.9.7.jar`
- 同理, 最高支持Java16, 建议使用Java8
- 可以通过 `arthas` 的 jad工具查看运行中的字节码反编译后的Java代码, 可以看到使用agent方式注入后的Java代码

# AOP实现之proxy

- proxy代理方式有两种实现, 一种是使用 Java自带的JDK实现, 另一种是使用第三方的cglib代理技术

### JDK动态代理

##### 使用
- 限制: JDK只能针对接口进行代理
- 代理类和普通类的区别在于, 代理类不存在源码, 在运行期间直接生成源码, 并使用合适的 ClassLoader 加载动态生成的源码
- 代理类需要现获得一个目标对象, 然后针对接口生成代理对象, 并通过反射调用目标对象中欲使用的 method
- 由于代理对象和目标对象都实现同一个接口, 所以可以基于该接口进行转换
- JDK代理允许代理目标标记为 final, 因为实际修改的是接口, 而不是代理目标类本身, 不需要继承该类

```java
Target target = new Target();
ClassLoader loader = JdkProxyDemo.class.getClassLoader();
Foo proxy = (Foo) Proxy.newProxyInstance(
    loader, new Class[]{Foo.class}, (p, method, params) -> {
        System.out.println("before...");
        method.setAccessible(true);
        Object result = method.invoke(target, args);
        System.out.println("after...");
        return result;
    }
);
proxy.foo();
```

##### 原理
- JDK进行的动态代理实际上是直接生成了**字节码**, 这种直接生成字节码的技术被称为 **ASM**, 最终会直接返回一个byte数组描述字节码
- 使用方法接口接收对应的方法, 接收的参数即为要调用的方法及其参数列表
- 在类接口中定义需要的方法, 以便于代理对象创建对应的代理方法
- 代理对象使用方法接口接收用户传入的方法, 并使用反射获取到对应类的成员方法, 将获取到的成员方法通过invoke进行调用
- 可以使代理类继承Proxy类, 其中已经定义了对应的 `InvocationHandler`, 并且为 `Proxy` 提供了以 `InvocationHandler` 为参数的有参构造

**反射优化**
- 方法进行反射调用时, 底层依赖了一个名为MethodAccessor的实现类
- 当检测到对某个方法的反射调用次数超过16次时, 会建立一个反射包中的 `GeneratedMethodAccessor2` 类对象, 通过该对象直接实现对方法的直接调用, 不在通过反射进行, 从而优化了频繁调用的反射方法的调用时间
- 这种方法的代价是需要生成 `GeneratedMethodAccessor2` 代理类对象

**注意事项**
- 使用Object接收可能的返回值, 并在实际返回时进行类型转换处理
- 使用 try-catch 合理获取与抛出运行时异常以及可抛出异常
- 由于处理方法可能需要使用**代理对象**进行操作, 所以方法接口中应该还需要加入Object Proxy, 并在代理对象方法中直接传入this
- 由于目标类的方法对象只需要一次获取, 不需要每次调用都重新反射获得, 所以可以通过声明静态类和静态代码块进行初始化, 之后直接使用即可

```java
public class $Proxy0 extends Proxy implements Foo {
    private static Method foo;
    private static Method bar;
    static {
        try {
            foo = Foo.class.getMethod("foo");
            bar = Foo.class.getMethod("bar");
        } catch (NoSuchMethodException e) {
            throw new NoSuchMethodError(e.getMessage());
        }
    }
    
    public $Proxy0(InvocationHandler handler) {
        super(handler);
    }
    
    @Override
    public void foo() {
        try {
            Method foo = Foo.class.getDeclaredMethod("foo");
            h.invoke(foo, new Object[0]);
        } catch (Throwable e) {
            throw new RuntimeException(e);
        }
    }
    
    @Override
    public int bar() {
        try {
            Method bar = Foo.class.getDeclaredMethod("bar");
            return (int) h.invoke(this, bar, new Object[0]);
        } catch (RuntimeException | Error e) {
            throw e;
        } catch (Throwable e) {
            throw new UndeclaredThrowableException(e);
        }
    }
}

public static void main(String[] args) {
    Foo proxy = new $Proxy0((method, params)->{
        System.out.println("before");
        method.invoke(new Target(), params);
        System.out.println("after");
    });
    proxy.foo();
    proxy.bar();
}
```


### CGLIB代理

##### 使用
- `cglib` 代理通过 `Enhancer.create()` 进行创建, 通过**继承关系**实现代理
- 传入函数的一般不是 Callback 接口, 而是其子接口 `MethodInterceptor`
- 代理类是目标类的子类型, 与其是一个继承的关系
- 由于需要使用继承实现代理, 所以要求目标类**不能是final类型**的, 同时代理类需要实现对目标类的方法重写, 所以目标类中同样也不能使用final方法
- cglib中除了方法**反射调用**目标的method, 还提供了methodProxy, 使得在**避免反射**的前提下实现方法的代理, 效率更高
- methodProxy 既可以使用invoke方法传入**目标对象**以调用方法, 也可以使用invokeSuper方法直接传入**代理对象**调用对应方法

**method调用**
```java
Target target = new Target();
Target proxy = (Target) Enhancer.create(Target.class, (MethodInterceptor) (o, method, args, methodProxy) -> {
    System.out.println("before...");
    Object result = method.invoke(target, args);
    System.out.println("after...");
    return result;
});
proxy.foo();
```

**methodProxy.invoke调用** (Spring使用)
```java
Target target = new Target();  
Target proxy = (Target) Enhancer.create(Target.class, (MethodInterceptor) (o, method, args, methodProxy) -> {  
    System.out.println("before...");  
    Object result = methodProxy.invoke(target, args);
    System.out.println("after...");  
    return result;  
});
```

**methodProxy.invokeSuper调用**
```java
Target proxy = (Target) Enhancer.create(Target.class, (MethodInterceptor) (o, method, args, methodProxy) -> {  
    System.out.println("before...");  
    Object result = methodProxy.invokeSuper(o, args);
    System.out.println("after...");  
    return result;  
});  
proxy.foo();
```

##### 原理







