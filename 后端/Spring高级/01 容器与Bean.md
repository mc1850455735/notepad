
# 容器接口

**源码解析**
- `SpringApplication` 的 `run` 方法会返回一个 `ConfigurableApplicationContext` 接口, 通过 `Ctrl + Alt + U` 可以看到类图
- `ConfigurableApplicationContext` 父接口为 `ApplicationContext` , 其顶层接口为 `BeanFactory`, 也就是说, `ApplicationContext` 实际上间接继承 `BeanFactory`
- 意味着 `ApplicationContext` 主要是实现了对 `BeanFactory` 的功能扩展

### BeanFactory

- 他是 `ApplicationContext` 的父接口
- 是 `Spring` 的**核心容器**, 主要的 `ApplicationContext` 实现都**组合**了他的功能
- `getBean` 功能是基于 `BeanFactory` 实现的, `BeanFactory` 变量取决于各自的具体实现
- 表面上只有 getBean, 实际上控制反转, 基本的依赖注入, 以及 Bean 的生命周期的各种功能, 都由其实现类 `DefaultListableBeanFactory` 所提供, 因为该实现类不止实现了 `BeanFactory`, 还实现了其他接口

**功能解析**
- `containsBean` : 判断容器中是否拥有该 Bean
- `getAliases` : 获取容器中 Bean 的别名
- `getBean` : 获取所需类型的 Bean

**单例对象管理**
- 通过类 `DefaultSingletonBeanRegistry` 实现了对单例对象的注册和管理
- Map类型的 `singletonObjects` 中保存了所有的单例对象

### ApplicationContext

`ApplicationContext` 共实现了4个接口
- `MessageSource`: 处理国际化资源能力
	- 通过 `getMessage()` , 读取 key 翻译后的结果
	- 翻译信息需要提前准备好, 存储在 `messages_` 开头的 `properties` 文件中
- `ResourcePatternResolver`: 通配符匹配资源的能力
	- 对应 `getResources()` 和 `getResource()`
	- `Resource` 类是 `Spring` 对资源的一个抽象
	- `"classpath*:META-INF/spring.factories"` : 对类路径和 jar 包中所有相关配置文件路径的访问, 如果不加 `*` 则只包含类路径
- `ApplicationEventPublisher`: 事件对象发布器功能
	- 使用时, 要发布的事件继承 `ApplicationEvent` , 参数为事件发送者
	- `Spring` 任何一个组件都可以作为事件的监听器, 用来接收事件
	- 使用 `@EventListener` 将方法标记为事件监听器, 并将事件作为参数传递
	- 事件发布器和监听器的功能可以实现业务的解耦合, 分布式架构下, 通常使用消息队列作为事件发送接收的方式
- `EnvironmentCapable`: 环境信息处理能力
	- 用于获取配置信息, 有的来自系统配置, 有的来自用户配置
	- 对于 `JAVA_HOME` , 在 `Environment` 中不区分大小写

# 容器实现

### BeanFactory 实现

- `DefaultListableBeanFactory` 是 `BeanFactory` 的最重要实现
- 使用 `BeanDefinitionBuilder` 中的 `genericBeanDefinition()` , 可以创建 `BeanDefinition` 供 `BeanFactory` 管理和使用
- 通过 `setScope()` , 可以设置 `Bean` 的生命周期
- 使用 `registerBeanDefinition()`, 在 `BeanFactory` 中注册 `Bean`

**配置注解解析**
- 默认的注册 `Bean` 功能并不能解析注解, 其注解是由其他类实现的
- `AnnotationConfigUtils.registerAnnotationConfigProcessors` 方法可以做到为 `BeanFactory` 添加一些常用的后处理器
- 通过调用添加的后处理器, 可以解析配置文件中的注解, 其类型为 `BeanFactoryPostProcessor`, 使用 `getBeansOfType` 进行获取并遍历执行
- 后处理器主要功能就是补充了一些 `bean` 定义
- 将每一个处理器都对 `beanFactory` 作用一遍, 以解析每一个注解
```java
AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).values().stream().forEach(  
    processor -> {  
        processor.postProcessBeanFactory(beanFactory);  
    }  
);
```

**依赖注入注解解析**
- 上述对配置相关注解的后处理器是 `BeanFactory` 后处理器, 如果想要对 `Bean` 的相关注解进行处理, 如依赖注入, 就需要使用 `Bean` 后处理器
- `Bean` 后处理器针对 `Bean` 的生命周期各个阶段提供扩展, 如 `@Autowired` 等
- `Bean` 后处理器类似 `BeanFactory` 后处理器, 其类型为 `BeanPostProcessor`
- `BeanFactory` 中添加 `BeanDefinition` 后并不直接创建, 而是使用 `Bean` 时才进行创建
- 使用 `preInstantiateSingletons` 可以在单例 `Bean` 使用前创建
- 使用以下代码创建 `bean` 后处理器的实例并绑定到 `BeanFactory`
```java
beanFactory.getBeansOfType(BeanPostProcessor.class).values().forEach(  beanFactory::addBeanPostProcessor  );
```

**后处理器排序逻辑**
- 默认先加入 `Autowired` 的后处理器, 再加入 `Resource` 的后处理器
- 先加入 `beanFactory` 的后处理器优先级更高, 通常 `@Autowired` 先于 `@Resoure`
- `Bean` 后处理内置排序逻辑, 通过返回 `order` 来定义排序的顺序, `order` 值越小的优先级越高, `Resource` 后处理器排序优先级高于 `Autowired` 后处理器
- 所以如果不排序, `@Autowired` 优先级更高, 排序后, `@Resource` 优先级更高

**总结**
- `BeanFactory` 不会主动调用后处理器解析注解
- `BeanFactory` 不会主动初始化单例对象
- `BeanFactory` 不会解析 `${}` 和 `#{}`
- `BeanFactory` 后处理器会有排序的逻辑

### ApplicationContext 实现

**XmlApplicationContext**
- `ClassPathXmlApplicationContext` : 基于 `classpath` 下 `xml` 格式配置文件创建
- `FileSystemXmlApplicationContext` : 基于文件路径的 `xml` 格式配置文件创建
- 内部会创建一个 `DefaultListenableBeanFactory` 对象, 用来加载 `Bean` 配置
- 通过 `XmlBeanDefinitionReader` 读取 `Xml` 中定义的 `BeanDefinition`
- `XmlApplicationContext` 默认不会添加后处理器, 如果需要后处理器, 需要在 `xml` 中添加 `context` 命名空间, 并配置 `<context:annotation-config />`
```java
DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();   
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);  
reader.loadBeanDefinitions(new ClassPathResource("b01.xml"));  
```

**AnnotationApplicationContext**
- `AnnotationConfigApplicationContext` : 使用配置类完成容器创建
- 与 `Xml` 类型容器不同的是, 该容器内会包含相关配置类的 `BeanDefinition` , 且该容器默认会自动添加 `BeanFactory` 和 `Bean` 后处理器
- `AnnotationConfigServletWebServerApplicationContext` : 支持 `WebServlet` 的容器, 基于 `java` 配置类来创建, 用于 `web` 环境
- 该容器配置类需要声明多个不同的 `Bean` 来保证 `Web` 环境的正确运行, 包括 `ServletWebServerFactory Web服务器` , `DispatcherServlet 请求分发器` , `DispatcherServletRegistrationBean 注册器`
- `ServletWebServerFactory` : **内嵌** Web 服务器
- `DispatcherServlet` : 请求分发器, 所有的请求通过分发器到达 `Controller`
- `DispatcherServletRegistrationBean` : 将分发器与服务器绑定在一起
- 在配置类中可以通过声明 `Controller Bean` 来定义简单的 `Controller`, `Bean` 的**斜线开头**名称即为请求响应的路径
- 在 `SpringBoot` 中, 使用注解简化了各个部分的配置过程, 但是实际流程还是相同的
```java
@Bean  
public ServletWebServerFactory servletWebServerFactory() {  
    return new TomcatServletWebServerFactory();  
}  
@Bean  
public DispatcherServlet dispatcherServlet() {  
    return new DispatcherServlet();  
}  
@Bean  
public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet) {  
    return new DispatcherServletRegistrationBean(dispatcherServlet, "/");  
}
@Bean("/hello")  
public Controller controller1() {  
    return (request, response) -> {  
        response.getWriter().print("hello");  
        return null;    
    };  
}
```


# Bean 生命周期

- 依赖注入: 对方法中的参数值进行依赖注入
```java
@Autowired  
public void autowire(@Value("${JAVA_HOME}") String home) {  
    log.debug("依赖注入: {}", home);  
}
```
- 通过实现对应接口, 即可创建自己的 Bean 后处理器, 在 Bean 导入时对其进行注入和拓展, 通过实现函数来声明对应扩展点要执行的动作
```java
MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {}
```

**Bean生命周期**
- 构造函数
- 依赖注入函数
- `PostConstruct` : 初始化
- `PreDestroy` : 销毁

**Bean扩展点**
- `postProcessBeforeInstantiation` : 实例化之前执行, 返回的对象会替换原本的 `bean` , 返回 `null` 保持原有对象不变
- `postProcessAfterInstantiation` : 实例化之后执行, 返回 `false` 会跳过依赖注入环节
- `postProcessProperties` : 依赖注入环节执行, 解析注解, 为 `Bean` 提供扩展功能, 如 `@Autowired` , `@Value` 等
- `postProcessBeforeInitialization` : 初始化之前执行, 返回的 `bean` 会替换原本的 `bean` , 如 `@PostConstruct` , `@ConfigurationProperties` 等
- `postProcessAfterInitialization` : 初始化之后执行, 这里返回的对象会替换原本的 bean, 如代理增强
- `postProcessBeforeDestruction` : 销毁 Bean 之前执行

**模板方法**

- 作用: 提高现有代码的扩展能力
- 对固定不变的部分, 将其写为主干类, 调用少量功能; 对于变化的部分, 将其功能抽象成为接口, 该接口可能会存在多个实现, 用以完成不同功能
- 使用 `List` 保存该接口, 并在相关调用部分循环遍历该 `List`
- 创建 `addBeanPostProcessor` , 使 `BeanFactory` 可以向 `List` 添加后处理器
- 这种设计理念即为 模板方法, 对修改关闭, 对扩展开放

# Bean 后处理器

### 常见后处理器

**概述**
- `GenericApplicationContext` 是一个干净的容器, 没有添加 `Bean` 后处理器等
- 通过指定 `registerBean(name, class)` , 可以添加 `Bean` 到容器中, 但是由于没有添加后处理器, 所以无法对 `Bean` 中的注解进行解析
- 通过 `registerBean(name, class)` 添加后处理器
- `GenericApplicationContext` 存在 `refresh()` 方法, 执行此方法, 会执行 `BeanFactory` 后处理器, 添加 `Bean` 后处理器, 初始化所有单例 `Bean`
- 后处理器的执行通过 `order` 字段进行排序, 决定了解析注解的顺序

**AutowiredAnnotationBeanPostProcessor**
- 通过 `AutowiredAnnotationBeanPostProcessor` 后处理器可以解析 `@Autowired` , `@Value`
-  将 `ContextAnnotationAutowireCandidateResolver` 通过 `BeanFactory` 的 `setAutowireCandidateResolver` 方法加入, 添加解析 `String` 的功能

**CommonAnnotationBeanPostProcessor**
- 通过 `CommonAnnotationBeanPostProcessor` 后处理器可以解析 `@Resource`, `@PostConstructor`, `@PreDestroy`

**ConfigurationProperties**
- `ConfigurationProperties` 是 `SpringBoot` 的配置注解, 通过该注解可以实现从配置文件中获取对应的值信息等
- 通过 `ConfigurationPropertiesBindingPostProcessor.register(x)` 进行后处理器的绑定, 此处不直接注册 `bean` 是因为其中涉及到复杂操作需要额外处理
```java
ConfigurationPropertiesBindingPostProcessor.register(context.getDefaultListableBeanFactory());
```

### Autowired 解析分析

**概述**
- `registerSingleton()` 只需要 `Bean` 名称和 `Bean` 实例对象即可添加单例 `Bean`
- 但是该方法只能添加成品 `Bean`, 不会对 `Bean` 进行初始化, 依赖注入等操作
- 通过 `setBeanFactory` 将后处理器与 `BeanFactory` 进行绑定
- 通过 `addEmbeddedValueResolver` 添加内嵌值解析器
- 通过添加 `StandardEnvironment()::resolvePlaceholders` 完成占位符解析

**依赖注入**
- `AutowiredAnnotationBeanPostProcessor` : `@Autowired` 后处理器
- 通过 `postProcessProperties(xxx)` 进行对单个 Bean 的依赖注入, 进行解析 `@Autowired` 等注解的操作
- `InjectionMetadata` 的 `inject` 方法完成对 `Bean` 的注入

**按类型查找值**
- `DependencyDescriptor` : 描述所依赖的对象
- `doResolveDependency` : 根据成员变量类型解析依赖, 得到对应 `Bean`
- 对于方法的依赖解析, 使用的封装参数为 `MethodParameter`, 在其中声明方法对象和要注入的参数序号
- 方法中的值注入同理, 直接按方法注入即可, 有占位符需要加占位符解析器
- 这就是 `Spring` 按类型自动注入的底层实现

# BeanFactory 后处理器

**概述**
- 为 `BeanFactory` 提供扩展, 对于配置类中的注解以及包扫描注解等内容, 由 `BeanFactory` 后处理器进行解析
- 使用 `ConfigurationClassPostProcessor` 后处理器完成对 `Config` 的解析
	- 如 `@ComponentScan`, `@Bean`, `@Import`, 等
- 通过 `refresh` 不光可以加载后处理器, 还可以初始化每个单例
- 使用 `MapperScannerConfigurer` 可以进行 `@Mapper` 的扫描, 添加后处理器时, 需要添加对应的包路径
- 注册 `Bean` 时, 可以通过 `Lambda` 表达式修改 `BeanDefinition` 中的属性
```java
context.registerBean(ConfigurationClassPostProcessor.class);
// 注册 Bean 时, 可以修改 BeanDefinition 中的属性
context.registerBean(MapperScannerConfigurer.class, bd -> {  
    bd.getPropertyValues().add("basePackage", "top.majinliang.a05.mapper");  
});
```

### @Component的解析

- 先通过注解获取要扫描的包, 取出包中所有字节码文件, 将其解析成为资源
- 读取 `Resource` 数据, 检查包中 `class` 是否添加了 `@Component` 或其派生注解
- 对于添加了注解的 class, 根据其类名生成它的类定义
- 通过类定义和类名生成器, 获取 `BeanName`, 并将 `BeanDefinition` 添加到 `BeanFactory` 中
- 将这段代码抽取为后处理器, 加入 `context`, 即可实现注解解析
- 使用 `AnnotationUtils.findAnnotation` 可以查看某个类中是否有相关注解
- `AnnotationBeanNameGenerator` : `BeanName` 生成器, 用于自动获取 `BeanName`
- `CachingMetadataReaderFactory` : 用来生成元数据读取工具类
- `reader.getAnnotationMetadata().hasAnnotation()` : 含有xx注解
- `reader.getAnnotationMetadata().hasMetaAnnotation` : 含有xx的派生注解
- 可以直接实现 `BeanDefinitionRegistryPostProcessor` 接口添加注g'n
- 相关解析流程如下
```java
ComponentScan componentScan = AnnotationUtils.findAnnotation(Config.class, ComponentScan.class);  
if (componentScan != null) {  
    // 得到 ComponentScan 的位置  
    String[] packages = componentScan.basePackages();  
    // 找到对应包名  
    for (String aPackage : packages) {  
        // System.out.println(aPackage);  
        // 将包名转换为路径  
        // com.itheima.a05.component -> classpath*:com/itheima/a05/component/**/*.class  
        String pathStr = 
        "classpath*:" + aPackage.replace(".", "/") + "/**/*.class";  
        // 获取包中资源, 通过 ResourcePatternResolver  
        Resource[] resources = context.getResources(pathStr);  
        // 读取类中元数据  
        CachingMetadataReaderFactory factory = 
            new CachingMetadataReaderFactory();  
        // BeanName 生成器  
        AnnotationBeanNameGenerator generator = 
            new AnnotationBeanNameGenerator();  
        // 解析资源  
        for (Resource resource : resources) {  
            // 读取 Resource           
             MetadataReader reader = 
                 factory.getMetadataReader(resource);  
            if(reader.getAnnotationMetadata()
               .hasAnnotation(Component.class.getName())
               || reader.getAnnotationMetadata()
                  .hasMetaAnnotation(Component.class.getName())) {  
                // 生成 BeanDefinition  
                AbstractBeanDefinition beanDefinition =  
                    BeanDefinitionBuilder.genericBeanDefinition(  
                    reader.getClassMetadata().getClassName()  
                        ).getBeanDefinition();  
                // 生成 BeanName                
                String beanName = 
                    generator.generateBeanName(beanDefinition, 
                    context.getDefaultListableBeanFactory());  
                // 注册 BeanDefinition             
                context.getDefaultListableBeanFactory()
                .registerBeanDefinition(beanName, beanDefinition);  
            }  
        }  
    }  
}
```

### @Bean的解析

- 使用 `getMetadataReader` 进行读取时, 不需要进行类加载, 效率高于反射方式
- 通过 `getAnnotatedMethods(xxx)` 获取被指定注解标注的类方法
- 在 `Bean` 中通常使用工厂方法的方式创建 `BeanDefinition`
- 使用 `BeanDefinitinoBuilder` 创建对应的构造类, 并通过 `setFactoryMethodOnBean` 设置工厂方法
- 工厂方法中 `factoryBean` 的名字设置为对应的工厂即可
-  对于构造方法和工厂方法参数, `BeanDefinitionBuilder` 装配模式选择 `AUTOWIRE_CONSTRUCTOR`
- 相关代码示例如下
```java
CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
MetadataReader reader = factory.getMetadataReader(
    new ClassPathResource("top/majinliang/a05/Config.class"));
Set<MethodMetadata> methods = 
    reader.getAnnotationMetadata().getAnnotatedMethods(Bean.class.getName());
for (MethodMetadata method : methods) {
    // 获取注解属性
    String initMethod = 
        (String) method.getAnnotationAttributes(Bean.class.getName()).get("initMethod");
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
    // 设置自动装配模式, 对于构造方法和工厂方法参数, 装配模式选择 Constructor
    builder.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR);
    if(initMethod.length() > 0) {
        builder.setInitMethodName(initMethod);
    }
    // 设置工厂方法
    AbstractBeanDefinition beanDefinition 
        = builder.setFactoryMethodOnBean(method.getMethodName(), "config").getBeanDefinition();
    context.getDefaultListableBeanFactory()
        .registerBeanDefinition(method.getMethodName(), beanDefinition);
}
```

### @Mapper的解析

- 在默认情况下, 实际上是通过添加 `Mapper` 相关接口的 `MapperFactoryBean` 泛型的实现来对每一个 `Mapper` 进行 `Bean` 注入的
- 由于直接通过扫描 `mapper` 包的方式进行 `Mapper` 添加, 所以需要在 `BeanDefinition` 中在进行声明
- 使用 `MapperFactoryBean` 进行 `Bean` 实例化, 传入接口名作为 `Bean` 创建的依据
- 设置其参数为 `Mapper` 接口, 并将注入类型改为按类型注入, 方便实例化完成后使用 `setSqlSessionFactory` 进行初始化
```java
// 资源路径解析加载器
PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
// 获取资源
Resource[] resources = resolver.getResources("classpath:top/majinliang/a05/mapper/**/*.class");
CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
AnnotationBeanNameGenerator generator = new AnnotationBeanNameGenerator();
for (Resource resource : resources) {
    MetadataReader reader = factory.getMetadataReader(resource);
    // 判断是接口还是实现类
    ClassMetadata classMetadata = reader.getClassMetadata();
    if(classMetadata.isInterface()) {
        AbstractBeanDefinition beanDefinition = 
            BeanDefinitionBuilder.genericBeanDefinition(MapperFactoryBean.class)
            .addConstructorArgValue(classMetadata.getClassName())
            .setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE)
            .getBeanDefinition();
        // 用作 BeanName 生成
        AbstractBeanDefinition bdMapper = 
            BeanDefinitionBuilder
            .genericBeanDefinition(classMetadata.getClassName())
            .getBeanDefinition();
        String beanName = generator.generateBeanName(bdMapper, beanFactory);
        beanFactory.registerBeanDefinition(beanName, beanDefinition);
    }
}
```

# Aware与Initializing

### Aware
- `Aware` 接口用于注入一些与容器相关信息, 是一种内置的注入手段
- `BeanNameAware`: 注入 `bean` 名字
- `BeanFactoryAware`: 注入 `BeanFactory` 容器
- `ApplicationContextAware`: 注入 `ApplicationContext` 容器
- `EmbeddedValueResolverAware`: 注入解析器解析 `${}`

**流程**
- 在 `refresh` 中, 会在 `bean` 初始化之前调用 `Aware` 接口的相关方法
- 先进行 `Aware` 方法的回调, 再进行初始化方法的回调

**与@Autowired区别**
- `@Autowired` 的解析需要用到 `bean` 后处理器, 属于扩展功能
- 而 `Aware` 属于 `bean` 的**内置功能**, 不需要任何扩展, `Spring` 就可以进行识别和调用
- 某些情况下, 扩展功能会失效, 但是内置功能不会失效
- 比如, 无法使用 `@Autowired` 为 `bean` 注入 `Context` , 但是使用 `Aware` 就可以对其进行注入

### InitializingBean
- 为 `Bean` 添加初始化方法, 是一种内置的初始化手段
- 先进行 `Aware` 方法的回调, 再进行初始化方法的回调
- `Aware` 和 `InitializingBean` 都在依赖注入后的初始化阶段进行, 属于 `Bean` 的一部分, 即使没有配置相应的后处理器也可以被执行

**后处理器的失效情况**
- refresh 首先找到所有 BeanFactory 后处理器并执行
- refresh 添加所有 Bean 后处理器
- refresh 执行初始化所有单例

**Java配置类不含BeanFactoryPostProcessor初始化顺序**
- 由于 `refresh` 先执行所有的 `BeanFacotry` 后处理器, 再注册所有的 Bean 后处理器, 使用后处理器对每个单例进行创建, 创建时, 先解析依赖注入拓展, 如 `@Autowired`, 再解析初始化拓展, 如 `@PostConstructor`, 随后进行初始化阶段, 在其中执行 `Aware` 和 `InitializingBean`, 最后创建 `Bean` 完成
 ![[后端/Spring高级/Inbox/Pasted image 20251010140546.png]]

**Java配置类含有BeanFactoryPostProcessor初始化顺序**
- 要执行配置类中定义的后处理器bean, 就需要先初始化配置类的单例对象, 因此, 初始化时, 必须先创建和初始化 Java 配置类对象, 取出 BeanFactory 后处理器和 Bean 后处理器, 才能创建其他的类; 该配置类相当于被提前创建了, 从而导致其他后处理器无法生效, 使得@Autowired 等注解失效
![[后端/Spring高级/Inbox/Pasted image 20251010140916.png]]

# 初始化与销毁

**初始化**
- 共有三种初始化方法
- `@PostConstruct` 初始化: 最先执行
- `InitializingBean` 接口初始化: 第二执行
- `@Bean(initMethod)` 初始化: 最后执行
- `Aware` 初始化顺序介于 `@Autowired` 和  `@PostConstructor` 之间
- 即初始化顺序: `@Autowired` -> `Aware` -> `@PostConstructor` -> `InitializingBean` -> `@Bean(initMethod)`

**销毁**
- 共有三种销毁方法
- `@PreDestroy` 销毁 : 最先执行
- `DisposableBean` 接口销毁: 第二执行
- `@Bean(destroyMethod)` 销毁: 最后执行

# Scope

### scope类型
- 在 `Spring5` 中, 共有 5 种 `Scope` 类型
- `singleton`: 每次获取 `Bean`, 获取的都是同一个对象
	- `Spring` 容器初始化时创建, 容器关闭时销毁
- `prototype`: 每次获取 `Bean`, 获取的是新的对象
	- 每次使用时创建, 由用户自行管理多例对象的销毁
- `request`: `Bean` 的生命周期与 `request` 域相当, 请求结束时销毁
	- 每次刷新都是一个新的请求, 该次访问结束 request 域结束
- `session`: 同一个会话内, `Bean` 保存相同, 会话结束时销毁
	- 不同的浏览器窗口相当于不同的会话, 一段时间不发送请求后, `session` 请求域对象自动销毁
	- 通过 `server.servlet.session.timeout=10s` 设置超时时间, 默认 `30min`
- `application`: 应用程序域, 应用程序 ( `ServletContext` ) 启动时创建, 结束时销毁
	- 在 `idea` 中不走销毁流程, 直接强制结束

### 注意事项

- 单例使用其他不为单例的域都需要加 `@Lazy`, 否则可能存在问题
- `JDK >= 9` 时, 如果反射调用 `JDK` 中方法或成员变量, 就会报错 `IllegalAccessException` , 即方法访问异常
- 使用 `@Lazy` 时, 相当于为对象创建了一个代理对象, 使用时会通过代理对象进行调用, 会通过反射调用 `JDK` 中 `Object` 的方法, 从而可能出现上述错误
- 通过更换 `JDK`, 或者重写 `toString()` 避免调用 `Object` 中方法, 或者添加参数 `--add-opens java.base/java.lang=ALL-UNNAMED` 允许反射调用 `JDK`

### scope失效

**概述**
- 单例对象中含有多例时, 对多例对象只注入一次 (因为单例对象只拥有一次创建机会), 所以会使单例 `Bean` 中的多例失效, 只使用第一次注入的多例
- 能不使用代理的情况下尽量少使用代理, 因为代理存在更多的性能损耗, 且代码复杂性相对更高
- 解决方法虽然不同, 但是理念殊途同归, 都是**推迟**其他 `scope bean` 的获取
![[后端/Spring高级/Inbox/Pasted image 20251011085858.png]]

**解决方法1**
- 使用 `@Lazy` 生成代理
- 虽然依然只存在一个代理对象, 但是每次调用代理时创建新的多例对象
**解决方法2**
- 在 `@Scope` 中添加 `proxyMode = ScopeProxyMode.TARGET_CLASS`
- 原理同样是生成一个代理用于控制多例对象的生成
![[后端/Spring高级/Inbox/Pasted image 20251011091025.png]]

**解决方法3**
- 在注入多例 Bean 的位置声明 `ObjectFactory` 对象工厂, 指定泛型类型后注入对象工厂, 并将原本的 `get` 方法返回值替换成工厂的 `getObject` 方法
```java
@Autowired  
private ObjectFactory<F3> f3;
public F3 getF3() {  
    return f3.getObject();  
}
```

**解决方法4**
- 注入 `ApplicationContext`, 直接使用 `context` 的 `getBean` 方法返回一个多例
```java
@Autowired  
private ApplicationContext context;  
public F4 getF4() {  
    return context.getBean(F4.class);  
}
```
