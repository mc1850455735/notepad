# 前言
在 Spring 的注解驱动开发模式中，`@Component` 是整个组件扫描与 Bean 注册流程的基础入口。尽管日常使用只需在类上标注该注解，但其背后涉及的元注解结构、`ClassPathBeanDefinitionScanner` 的扫描机制、`AnnotatedBeanDefinitionReader` 对注解元数据的解析过程、BeanDefinition 的构建与注册逻辑等一系列核心步骤，构成了 Spring IoC 容器自动发现与管理组件的核心能力。本文将从源码出发，解析 `@Component` 从被加入类到成功初始化为 Bean 到 Spring 容器内的整体流程。

> 本文会出现大量源码，部分源码可能会进行删减，只保留涉及 `@Component` 组件扫描与 Bean 注册流程的部分。部分流程调用链过长，可能不会解析深层方法的实现，只将注意力集中在方法的功能。本文基于 Spring Framework 5.3.x 版本分析。

本文演示所使用的代码如下：

组件类：

````java
@Component
public class MyComponent {
    public void sayHello() {
        System.out.println("Hello from MyComponent!");
    }
}
````

扫描与注册：

````java
public class B01Application {
    public static void main(String[] args) throws IOException {
        // 1. 创建一个空 Spring 容器
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();

        // 2. 创建一个 ClassPathBeanDefinitionScanner，用于扫描 @Component
        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(context);

        // 3. 手动触发扫描（自己配置时注意修改扫描位置）
        scanner.scan("com.example.b01");

        // 4. 刷新容器，使注册的 Bean 生效
        context.refresh();

        // 5. 测试获取 Bean
        MyComponent component = context.getBean(MyComponent.class);
        component.sayHello();

        context.close();
    }
}
````

# 元注解结构

`@Component` 本身只是一个极简的元注解结构，其最核心作用是通过 RUNTIME 级别的元信息指示 Spring：此类应被视为组件候选。复杂行为均发生在扫描器与 BeanDefinition 注册阶段，而非 `@Component` 注解内部。

除 `@Component` 自身，`@Controller`/`@Service`/`@Repository` 都是 `@Component` 的派生注解，其元注解中包含 `@Component`，因此能被扫描器识别为候选组件，可以用作 Bean 的标识。

`@Component`的元注解结构如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Component {
    String value() default "";
}
```

`@Target(ElementType.TYPE)`表示该注解只能用于类、接口、枚举等类型级别的声明，意味着该注解是一个类级别的组件标识。Spring扫描时，通过该class元数据判断此类是否是组件。

`@Retention(RetentionPolicy.RUNTIME)`表示该注解运行时可读取，为Spring组件扫描器在运行阶段的扫描提供了依据，如果不是 RUNTIME 的，就无法在运行时的扫描阶段被获取到。

`@Documented` 主要用于生成 JavaDoc 时包含该注解，对 Spring 行为无影响，主要为了提高 API 文档友好性。

`value()` 属性用于标识 BeanName 生成器的可选别名。如果不设置 value，Spring 会采取默认规则，使用类名首字母小写作为 BeanName。

# 组件扫描过程

## scan 方法整体流程

通过进行对`scanner.scan("com.example.demo")` 代码进行断点调试，观察代码执行流程。`scan()` 方法相关代码如下：

````java
public int scan(String... basePackages) {
   int beanCountAtScanStart = this.registry.getBeanDefinitionCount();

   doScan(basePackages);

   // Register annotation config processors, if necessary.
   if (this.includeAnnotationConfig) {
      AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
   }

   return (this.registry.getBeanDefinitionCount() - beanCountAtScanStart);
}
````

## doScan组件扫描核心流程

可以发现，在 `scan()` 方法中，方法的返回值是本次扫描获取到的 bean 的数量，而在该方法中，最主要用来实现扫描功能的方法是 `doScan()` 方法，其相关代码如下：

````java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
   Assert.notEmpty(basePackages, "At least one base package must be specified");
   Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
   for (String basePackage : basePackages) {
      Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
      for (BeanDefinition candidate : candidates) {
         ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
         candidate.setScope(scopeMetadata.getScopeName());
         String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
         if (candidate instanceof AbstractBeanDefinition) {
            postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
         }
         if (candidate instanceof AnnotatedBeanDefinition) {
            AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
         }
         if (checkCandidate(beanName, candidate)) {
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
            definitionHolder =
                  AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            beanDefinitions.add(definitionHolder);
            registerBeanDefinition(definitionHolder, this.registry);
         }
      }
   }
   return beanDefinitions;
}
````

## 轮询包名的候选组件查询

首先创建一个 Set 集合，用以存储扫描得出的 BeanDefinitionHolder。BeanDefinitionHolder 提供 BeanDefinition 与 beanName 的封装，而 BeanDefinition 代表描述 Bean 在容器中应如何被管理。

```java
Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
```

遍历传入的不定长包名集合，扫描包中的类，得到包路径中的候选组件类集合。`findCandidateComponents(basePackage)` 方法用于从指定包路径中扫描出候选组件类，该方法会读取 classpath 下的所有 `.class` 文件，并判断扫描到的 `.class` 文件是否满足候选组件类的条件，返回经判断后的 BeanDefinition 集合。

```java
Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
```

`findCandidateComponents()` 方法源码如下。传统方法在 `scanCandidateComponents()` 方法中实现组件的扫描功能。而在 Spring 5 中，引入了组件索引功能加速扫描，如果类路径中存在 Spring 5 引入的组件索引文件，并且当前的 `include-filters` 能够被索引支持，则优先使用索引文件快速找到对应组件。

```java
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
    if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
        return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
    }
    else {
        return scanCandidateComponents(basePackage);
    }
}
```

## 扫描获取BeanDefinition集合

对于 `scanCandidateComponents()` 方法，其返回值为扫描到的 BeanDefinition 集合，其实现如下。

```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
   Set<BeanDefinition> candidates = new LinkedHashSet<>();
   try {
      String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
      Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
      boolean traceEnabled = logger.isTraceEnabled();
      boolean debugEnabled = logger.isDebugEnabled();
      for (Resource resource : resources) {
         if (traceEnabled) {
            logger.trace("Scanning " + resource);
         }
         if (resource.isReadable()) {
            try {
               MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
               if (isCandidateComponent(metadataReader)) {
                  ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                  sbd.setSource(resource);
                  if (isCandidateComponent(sbd)) {
                     if (debugEnabled) {
                        logger.debug("Identified candidate component class: " + resource);
                     }
                     candidates.add(sbd);
                  }
                  else { ... }
               }
               else { ... }
            }
            catch (Throwable ex) {
               throw new BeanDefinitionStoreException("Failed to read candidate component class: " + resource, ex);
            }
         }
         else { ... }
      }
   }
   catch (IOException ex) {
      throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
   }
   return candidates;
}
```

### 加载资源文件

首先解析传入的 `basePackage` 包路径，将其通过字符串拼接构建为 `classpath*:包路径/**/*.class` 的路径格式。随后使用 `getResourcePatternResolver().getResources()` 解析对应路径，从 classpath 找到所有 class 文件资源，并逐个遍历得到的资源集合。

```java
String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
        resolveBasePackage(basePackage) + '/' + this.resourcePattern;
Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
```

### 基于 ASM 读取类元数据

遍历过程中，对所有可读的 `.class` 文件进行处理。首先通过 `MetadataReader` 读取类的元数据。该过程由 ASM 在字节码层面完成解析而非直接加载类，避免了类加载过程中触发静态代码块、减少内存占用，提升扫描效率。

```java
MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
```

### 两轮过滤与 BeanDefinition 构建

获取到对应资源的 MetadataReader 后，首先通过 `isCandidateComponent(MetadataReader)` 方法进行第一轮过滤，基于字节码元数据判断类是否带有符合条件的注解（通常是 `@Component`、`@Controller` 等）。整个过程通过读取 metadata 判断，不加载类，也不对类进行实例化判断。

若第一轮过滤通过，则根据 `MetadataReader` 构建对应 `ScannedGenericBeanDefinition`，并设置其资源路径。

随后进入第二轮过滤，使用 `isCandidateComponent(BeanDefinition)` 方法，针对 BeanDefinition 自身进行验证，判断该 BeanDefinition 是否真正能够作为 Spring Bean 进行注册。这一步主要判断类结构特征，即是否为独立类、是否为具体类、是否是带有 @Lookup 方法的抽象类（Spring 可通过 CGLIB 为其生成可实例化的子类）。

只有满足这些条件的类，才被视为可被容器管理的候选组件，加入候选组件集合。

```java
if (isCandidateComponent(metadataReader)) {
  ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
  sbd.setSource(resource);
  if (isCandidateComponent(sbd)) {
     if (debugEnabled) {
        logger.debug("Identified candidate component class: " + resource);
     }
     candidates.add(sbd);
  }
  else { ... }
}
```

## 组件处理流程

### 解析作用域信息

遍历扫描返回的 BeanDefinition 集合，处理其中每一个候选组件类，首先使用 `resolveScopeMetadata()` 方法确定 Bean 的创建作用域及代理模式。只有使用注解扫描方式得到的 BeanDefinition 才支持 `@Scope` 判断， 对于添加了 `@Component` 注解但未添加 `@Scope` 注解的 BeanDefinition，Spring 默认作用域为单例 `Singleton`，如果类上添加了 `@Scope` 注解并指定了作用域类型，则该方法会解析出对应作用域设置到候选类上。

```java
ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
candidate.setScope(scopeMetadata.getScopeName());
```

### 生成 BeanName

对于 BeanName，通过调用 `generateBeanName()` 方法进行获取。该方法会先判断该 BeanDefinition 是否由注解标识的，只有注解标识的 Bean 才能够用这种方法生成 BeanName。方法内部通过 `determineBeanNameFromAnnotation()` 方法解析 `@Component` 注解的 `value`，如果用户显式定义了值，则使用该值作为 BeanName，否则取类的简单类名，并将其首字母小写作为 BeanName。

```java
String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
```

### 结构补充及定制化处理

当 BeanDefinition 属于 `AbstractBeanDefinition` 类型时，Spring 会调用 `postProcessBeanDefinition()` 方法。这个方法本身是扫描器提供的扩展点，用于在必要时对扫描生成的 BeanDefinition 进行结构上的补充或定制化处理。默认实现只应用 BeanDefinitionDefaults 和自动注入候选规则，并不会自动设置 init-method、destroy-method 等属性。该方法主要用于扫描器扩展，而不是常规元数据处理的位置。

之所以要进行类型检查，是因为只有 `AbstractBeanDefinition` 才具备完整的可扩展属性模型；如果 BeanDefinition 不是由此类派生，说明它可能来自 XML、程序化注册等其他来源，不一定适合修改内部结构，因此在执行额外属性补充前必须保证类型安全。

```java
if (candidate instanceof AnnotatedBeanDefinition) {
   AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
}
```

在 `ClassPathBeanDefinitionScanner` 类中，`postProcessBeanDefinition()` 方法的实现主要是用来为通过注解得到的 BeanDefinition 设置默认配置，并根据 `ClassPathBeanDefinitionScanner` 中设置的可能的过滤 beanName 规则决定它是否是自动注入候选者。

```java
protected void postProcessBeanDefinition(AbstractBeanDefinition beanDefinition, String beanName) {
   beanDefinition.applyDefaults(this.beanDefinitionDefaults);
   if (this.autowireCandidatePatterns != null) {
      beanDefinition.setAutowireCandidate(PatternMatchUtils.simpleMatch(this.autowireCandidatePatterns, beanName));
   }
}
```

### 处理常见元注解

当该 BeanDefinition 是由注解标识的类声明时，即该 BeanDefinition 继承自 `AnnotatedBeanDefinition` 时，执行 `processCommonDefinitionAnnotations()` 方法，用来处理 Bean 类上常见的 Spring 元注解，如 `@Lazy`、`@Primary` 等。

```java
if (candidate instanceof AnnotatedBeanDefinition) {
   AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
}
```

### BeanDefinition 冲突校验与代理处理

`checkCandidate()` 方法用于检查 BeanName 是否与已有 BeanDefinition 冲突，确保 BeanDefinition 的注册不会产生重复、覆盖或不兼容的情况，是 Spring 保证容器一致性的关键步骤。如果符合安全注册条件，则根据候选组件类信息和生成的 BeanName ，创建一个 BeanDefinitionHolder 并通过 `applyScopedProxyMode()` 方法设置该 BeanDefinitionHolder 的代理模式，将其添加到 beanDefinition 集合中。

```java
if (checkCandidate(beanName, candidate)) {
   BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
   definitionHolder =
         AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
   beanDefinitions.add(definitionHolder);
   registerBeanDefinition(definitionHolder, this.registry);
}
```

# BeanDefinition 注册

## 调用流程

通过 `registerBeanDefinition()` 方法，将对应 BeanDefinitionHolder 中的 BeanDefinition 注册到指定容器类中。其中，`this.registry` 表示 BeanDefinition 注册表，是 Spring 容器用于保存 BeanDefinition 的核心接口，具体职责由 `DefaultListableBeanFactory` 等实现类进行实现。

```java
registerBeanDefinition(definitionHolder, this.registry);
```

`registerBeanDefinition` 方法中调用了 `BeanDefinitionReaderUtils` 中的 `registerBeanDefinition(definitionHolder, registry)` 方法，该方法通过调用 `BeanDefinitionRegister` 接口中的 `registerBeanDefinition` 方法注册 `BeanDefinition`，并为注册好的 `BeanDefinition` 添加别名。

```java
public static void registerBeanDefinition(
      BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
      throws BeanDefinitionStoreException {

   // Register bean definition under primary name.
   String beanName = definitionHolder.getBeanName();
   registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

   // Register aliases for bean name, if any.
   String[] aliases = definitionHolder.getAliases();
   if (aliases != null) {
      for (String alias : aliases) {
         registry.registerAlias(beanName, alias);
      }
   }
}
```

`registry.registerBeanDefinition` 方法即为 `BeanDefinitionRegister` 接口的 `registerBeanDefinition()` 方法，将 BeanDefinitionHolder 中包装的 BeanName 和 BeanDefinition 作为参数传入该方法。BeanDefinition 注册在 `DefaultListableBeanFactory` 实现类中进行实现。该方法实现如下：

## BeanDefinition 注册实现概览

```java
@Override
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
      throws BeanDefinitionStoreException {

   Assert.hasText(beanName, "Bean name must not be empty");
   Assert.notNull(beanDefinition, "BeanDefinition must not be null");

   if (beanDefinition instanceof AbstractBeanDefinition) {
      try {
         ((AbstractBeanDefinition) beanDefinition).validate();
      }
      catch (BeanDefinitionValidationException ex) {
         throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
               "Validation of bean definition failed", ex);
      }
   }

   BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
   if (existingDefinition != null) {
      if (!isAllowBeanDefinitionOverriding()) {
         throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
      }
      else if (existingDefinition.getRole() < beanDefinition.getRole()) {
         // e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
         if (logger.isInfoEnabled()) {
            logger.info("Overriding user-defined bean definition for bean '" + beanName +
                  "' with a framework-generated bean definition: replacing [" +
                  existingDefinition + "] with [" + beanDefinition + "]");
         }
      }
      else if (!beanDefinition.equals(existingDefinition)) {
         if (logger.isDebugEnabled()) {
            logger.debug("Overriding bean definition for bean '" + beanName +
                  "' with a different definition: replacing [" + existingDefinition +
                  "] with [" + beanDefinition + "]");
         }
      }
      else {
         if (logger.isTraceEnabled()) {
            logger.trace("Overriding bean definition for bean '" + beanName +
                  "' with an equivalent definition: replacing [" + existingDefinition +
                  "] with [" + beanDefinition + "]");
         }
      }
      this.beanDefinitionMap.put(beanName, beanDefinition);
   }
   else {
      if (hasBeanCreationStarted()) {
         // Cannot modify startup-time collection elements anymore (for stable iteration)
         synchronized (this.beanDefinitionMap) {
            this.beanDefinitionMap.put(beanName, beanDefinition);
            List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
            updatedDefinitions.addAll(this.beanDefinitionNames);
            updatedDefinitions.add(beanName);
            this.beanDefinitionNames = updatedDefinitions;
            removeManualSingletonName(beanName);
         }
      }
      else {
         // Still in startup registration phase
         this.beanDefinitionMap.put(beanName, beanDefinition);
         this.beanDefinitionNames.add(beanName);
         removeManualSingletonName(beanName);
      }
      this.frozenBeanDefinitionNames = null;
   }

   if (existingDefinition != null || containsSingleton(beanName)) {
      resetBeanDefinition(beanName);
   }
   else if (isConfigurationFrozen()) {
      clearByTypeCache();
   }
}
```

## BeanDefinition 校验

在 Spring 注册 BeanDefinition 之前，会对继承自 `AbstractBeanDefinition` 的 BeanDefinition 的合法性进行校验，因为它提供了更完整的元数据模型和校验规则。通过调用 `validate()` 方法，Spring会检查指定的 Bean Class 是否存在、构造方法或工厂方法等配置是否有效、作用域与生命周期配置是否冲突、是否存在无效的自动装配模式、Init / Destroy 方法是否可解析等。

```java
if (beanDefinition instanceof AbstractBeanDefinition) {
   try {
      ((AbstractBeanDefinition) beanDefinition).validate();
   }
   catch (BeanDefinitionValidationException ex) {
      throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
            "Validation of bean definition failed", ex);
   }
}
```

## BeanDefinition 注册/覆盖

通过 BeanName 在 `beanDefinitionMap` 中查找对应的 BeanDefinition 是否存在。存在返回对应的 BeanDefinition，不存在则返回 `NULL`。


```java
BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
```

### BeanDefinition 已存在

如果 BeanDefinition 已存在，此时应判断是否需要对 BeanDefinition 进行覆盖，共分为四种情况，分别是**不允许覆盖**、**框架级 Bean 覆盖用户级 Bean**、**新用户 Bean 覆盖旧用户 Bean**、**重复扫描导致的等价定义覆盖**，除不允许覆盖的情况外，每种情况都对应输出不同的日志等级。最后**统一写入** `beanDefinitionMap`。

是否允许覆盖基于 `isAllowBeanDefinitionOverriding()` 方法，该方法返回 BeanFactory 容器内的 `this.allowBeanDefinitionOverriding` 属性。在 Spring Framework 中，该属性默认为 `true`，即 BeanDefinition 允许被覆盖；而 Spring Boot 从 2.1 开始默认不允许覆盖。

当允许覆盖时，首先通过 `getRole()` 方法判断已有的 BeanDefinition 和传入的 BeanDefinition 之间的等级，当传入的 BeanDefinition 的 `getRole` 等级大于已有 BeanDefinition 时， 说明这种情况是框架级 Bean 覆盖用户级 Bean。

随后比较两个 BeanDefinition，当二者不相等时，说明该覆盖操作是新的 BeanDefinition 覆盖旧的 BeanDefinition。

最后，如果都不符合，说明新传入的 BeanDefinition 和现有的 BeanDefinition 实际上是同一个 `@Component` 类的重复扫描。

```java
if (existingDefinition != null) {
   if (!isAllowBeanDefinitionOverriding()) {
      throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
   }
   else if (existingDefinition.getRole() < beanDefinition.getRole()) {
      // e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
      if (logger.isInfoEnabled()) {
         logger.info("Overriding user-defined bean definition for bean '" + beanName +
               "' with a framework-generated bean definition: replacing [" +
               existingDefinition + "] with [" + beanDefinition + "]");
      }
   }
   else if (!beanDefinition.equals(existingDefinition)) {
      if (logger.isDebugEnabled()) {
         logger.debug("Overriding bean definition for bean '" + beanName +
               "' with a different definition: replacing [" + existingDefinition +
               "] with [" + beanDefinition + "]");
      }
   }
   else {
      if (logger.isTraceEnabled()) {
         logger.trace("Overriding bean definition for bean '" + beanName +
               "' with an equivalent definition: replacing [" + existingDefinition +
               "] with [" + beanDefinition + "]");
      }
   }
   this.beanDefinitionMap.put(beanName, beanDefinition);
}
```

### BeanDefinition 不存在

如果 BeanDefinition 目前不存在，则应创建对应的 BeanDefinition。`hasBeanCreationStarted()` 方法表明容器是否已经开始创建 Bean。根据该方法返回值，创建过程分为两种情况：

若 `hasBeanCreationStarted()` 方法返回值为 `true`，说明容器正在执行 `refresh()`，该过程中已经开始实例化 Bean，在实例化过程中，容器内部的数据结构可能被频繁遍历，在遍历过程中修改它们可能导致 `ConcurrentModificationException` 或 Bean 创建逻辑状态不一致。所以通过加 `synchronized` 锁的方式保障线程安全，并采用 `Copy-on-write` 思想对 `beanDefinitionNames` 执行修改。

最后，调用 `removeManualSingletonName()` 方法从手动注册单例名称集合中移除同名 beanName。Spring 中存在一个 Set 集合用于管理手动注册的单例 Bean 名称。在注册 BeanDefinition 时，如果对应 beanName 通过 `registerSingleton()` 手动注册过，避免 BeanDefinition 注册机制与手动注册机制发生冲突，导致后续流程中继续将该 beanName 视为手动单例，Spring 会在手动单例名称集合中移除该记录。但对应实际手动注册单例对象仍存储在 `singletonObjects` 中，不会因集合中 BeanName 的删除而移除。

```java
if (hasBeanCreationStarted()) {
   // Cannot modify startup-time collection elements anymore (for stable iteration)
   synchronized (this.beanDefinitionMap) {
      this.beanDefinitionMap.put(beanName, beanDefinition);
      List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
      updatedDefinitions.addAll(this.beanDefinitionNames);
      updatedDefinitions.add(beanName);
      this.beanDefinitionNames = updatedDefinitions;
      removeManualSingletonName(beanName);
   }
}
```

如果 `hasBeanCreationStarted()` 方法返回值为 `false`，说明容器尚未开始实例化 Bean，数据结构不需要锁保护，也不存在遍历冲突，可以直接修改 Map 和 List。同理，需要调用 `removeManualSingletonName()` 移除手动单例名称。

```java
else {
   // Still in startup registration phase
   this.beanDefinitionMap.put(beanName, beanDefinition);
   this.beanDefinitionNames.add(beanName);
   removeManualSingletonName(beanName);
}
```

最后，无论是否已经开始实例化 Bean，最后都要清空 `frozenBeanDefinitionNames`，打破冻结状态。这是因为 Spring 在某些阶段会“冻结配置”，以加速按类型查找。一旦新增 `BeanDefinition`，必须取消冻结，否则缓存失效。

```java
this.frozenBeanDefinitionNames = null;
```

## 缓存刷新

如果当前 BeanDefinition 覆盖已有定义或之前已经存在同名手动注册单例时，调用 `resetBeanDefinition()` 方法清理所有依赖于旧定义信息的缓存，确保容器之后基于最新定义行为执行。

若容器配置已被冻结（通常发生在 refresh 阶段之后），新增 BeanDefinition 可能导致按类型查找缓存失效，此时通过 `clearByTypeCache()` 方法清除类型缓存，保持检索结果的准确性。

```java
if (existingDefinition != null || containsSingleton(beanName)) {
   resetBeanDefinition(beanName);
}
else if (isConfigurationFrozen()) {
   clearByTypeCache();
}
```

# Bean实例化

## refresh() 实例化 Bean

通过调用 `context.refresh()` 刷新容器，使注册好的 BeanDefinition 能够被实例化为 Bean 并使用。在该方法中，通过调用 `finishBeanFactoryInitialization()` 方法完成非懒加载 Bean 的实例化。`refresh()` 的相关代码（缩略版）如下，主要关心 `finishBeanFactoryInitialization()` 方法：

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // ...（上下文刷新准备工作）
        // ...（创建 / 刷新 BeanFactory）
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // ...（为 BeanFactory 设置标准配置）
        try {
            // ...（子类可扩展 BeanFactory）
            // ...（执行 BeanFactoryPostProcessor）
            // ...（注册 BeanPostProcessor）
            // ...（初始化消息源）
            // ...（初始化事件广播器）
            // ...（子类额外的特殊初始化）
            // ...（注册监听器）

            // 实例化所有非懒加载单例 Bean
            finishBeanFactoryInitialization(beanFactory);

            // ...（发布 ContextRefreshedEvent）
        }
        catch (BeansException ex) {
            // ...（异常处理：销毁已创建单例、恢复容器状态）
            throw ex;
        }
        finally {
            // ...（清理缓存）
        }
    }
}
```

## finishBeanFactoryInitialization() 方法解析

`finishBeanFactoryInitialization()` 方法的相关代码如下：

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
   // Initialize conversion service for this context.
   if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
         beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
      beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
   }

   // Register a default embedded value resolver if no BeanFactoryPostProcessor
   // (such as a PropertySourcesPlaceholderConfigurer bean) registered any before:
   // at this point, primarily for resolution in annotation attribute values.
   if (!beanFactory.hasEmbeddedValueResolver()) {
      beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
   }

   // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
   String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
   for (String weaverAwareName : weaverAwareNames) {
      getBean(weaverAwareName);
   }

   // Stop using the temporary ClassLoader for type matching.
   beanFactory.setTempClassLoader(null);

   // Allow for caching all bean definition metadata, not expecting further changes.
   beanFactory.freezeConfiguration();

   // Instantiate all remaining (non-lazy-init) singletons.
   beanFactory.preInstantiateSingletons();
}
```

### 初始化全局 ConversionService

在 `finishBeanFactoryInitialization()` 方法中，首先通过 `containsBean()`方法检测容器中是否自定义了名为 `conversionService` 的 Bean，该 Bean 的作用是负责注入过程中的类型转换。如果存在，则将该 Bean 设置为容器的全局类型转换服务，确保后续 Bean 创建与属性注入过程能正确执行类型转换。

```java
// Initialize conversion service for this context.
if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) && beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
      beanFactory.setConversionService(beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
}
```

### 注册占位符解析器

其次，如果容器中还没有占位符解析器 `EmbeddedValueResolver`，则注册一个默认的解析器，用于解析字符串中的 `${...}` 占位符。

```java
// Register a default embedded value resolver if no BeanFactoryPostProcessor
// (such as a PropertySourcesPlaceholderConfigurer bean) registered any before:
// at this point, primarily for resolution in annotation attribute values.
if (!beanFactory.hasEmbeddedValueResolver()) {
   beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
}
```

### 提前实例化 LoadTimeWeaverAware

提前实例化所有实现了 `LoadTimeWeaverAware` 接口的 Bean，使它们能够尽早向 JVM 注册 `ClassFileTransformer`，用于类加载期增强（Load-Time Weaving，LTW）。

```java
// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
for (String weaverAwareName : weaverAwareNames) {
   getBean(weaverAwareName);
}
```

### 停止临时类加载器并冻结 BeanDefinition 配置

随后，停止使用临时类加载器并冻结 BeanDefinition 配置，禁止容器后续再修改定义。

```java
// Stop using the temporary ClassLoader for type matching.
beanFactory.setTempClassLoader(null);

// Allow for caching all bean definition metadata, not expecting further changes.
beanFactory.freezeConfiguration();
```

最后使用 `preInstantiateSingleton()` 实例化所有非延迟加载的单例 Bean。

```java
// Instantiate all remaining (non-lazy-init) singletons.
beanFactory.preInstantiateSingletons();
```

## 非懒加载单例实例化流程

`beanFactory.preInstantiateSingletons()` 的调用完成了 bean 的实例化，其实现代码如下：

```java
@Override
public void preInstantiateSingletons() throws BeansException {
   if (logger.isTraceEnabled()) {
      logger.trace("Pre-instantiating singletons in " + this);
   }

   // Iterate over a copy to allow for init methods which in turn register new bean definitions.
   // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
   List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

   // Trigger initialization of all non-lazy singleton beans...
   for (String beanName : beanNames) {
      RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
      if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
         if (isFactoryBean(beanName)) {
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
               FactoryBean<?> factory = (FactoryBean<?>) bean;
               boolean isEagerInit;
               if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                  isEagerInit = AccessController.doPrivileged(
                        (PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
                        getAccessControlContext());
               }
               else {
                  isEagerInit = (factory instanceof SmartFactoryBean &&
                        ((SmartFactoryBean<?>) factory).isEagerInit());
               }
               if (isEagerInit) {
                  getBean(beanName);
               }
            }
         }
         else {
            getBean(beanName);
         }
      }
   }

   // Trigger post-initialization callback for all applicable beans...
   for (String beanName : beanNames) {
      Object singletonInstance = getSingleton(beanName);
      if (singletonInstance instanceof SmartInitializingSingleton) {
         StartupStep smartInitialize = this.getApplicationStartup().start("spring.beans.smart-initialize")
               .tag("beanName", beanName);
         SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
         if (System.getSecurityManager() != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
               smartSingleton.afterSingletonsInstantiated();
               return null;
            }, getAccessControlContext());
         }
         else {
            smartSingleton.afterSingletonsInstantiated();
         }
         smartInitialize.end();
      }
   }
}
```

### 获取并遍历 beanDefinitionNames

首先获取当前存在的 `beanDefinitionNames`, 并遍历集合内容。在集合中，根据 beanName 获取对应的 BeanDefinition 并对其进行处理。

```java
List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
for (String beanName : beanNames) {
    RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
```

判断该 BeanDefinition 是否允许在非懒加载环节作为单例 Bean 被创建，能够被创建的条件是类不为抽象类且为单例模式且不是懒加载。如果符合条件，则实例化该 Bean，否则直接跳过该 BeanDefinition。

当确定该 BeanDefinition 符合实例化条件后，还需要进行是否 FactoryBean 的判断。FactoryBean 是 Spring 中一个特殊类型的 Bean，其本身就是一个 Bean，同时执行其 `getObject()` 方法还可以获取对应产品 Bean。如果是 FactoryBean，则通过添加前缀后执行 `getBean()` 方法获取 FactoryBean 本身（而非产品 Bean），最后使用 `bean instanceof FactoryBean` 进行是否为 FactoryBean 的确认。

```java
if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
    if (isFactoryBean(beanName)) {
        Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
        if (bean instanceof FactoryBean) {
```

### FactoryBean 的特殊初始化

在 Spring 的预实例化过程中，FactoryBean 的处理方式不同于普通 Bean。Spring 会首先通过 `&beanName` 获取 FactoryBean 本体，随后定义 `isEagerInit` 变量，用于根据 FactoryBean 类型决定是否在容器启动阶段提前创建 FactoryBean 的产品对象。

如果 JVM 开启了 `SecurityManager`，则 Spring 需要在受控权限下执行 `SmartFactoryBean` 的 `isEagerInit()` 方法，所以使用 `AccessController.doPrivileged()` 在受限权限环境中以安全方式执行。

否则，对于普通环境下的判断，直接判断是否是 `SmartFactoryBean` 并调用 `isEagerInit()`。

```java
FactoryBean<?> factory = (FactoryBean<?>) bean;
boolean isEagerInit;
if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
    isEagerInit = AccessController.doPrivileged(
        (PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
        getAccessControlContext());
}
else {
    isEagerInit = (factory instanceof SmartFactoryBean &&
                   ((SmartFactoryBean<?>) factory).isEagerInit());
}
if (isEagerInit) {
    getBean(beanName);
}
```

### 普通初始化

通过 `getBean()` 方法进行初始化。Spring 中，`getBean()` 方法不仅负责返回 Bean 实例，当 BeanDefinition 已存在、目标 Bean 尚未实例化，它也会触发 Bean 的实例化流程。

### 回调SmartInitializingSingleton

在容器中所有非懒加载单例 Bean 都已经完成创建之后，再次统一回调实现了 `SmartInitializingSingleton` 接口的 Bean，让它们执行接口中声明的 `afterSingletonsInstantiated()` 逻辑。由于与 Bean 实例化关系有限，在此不作详细说明。

```java
// Trigger post-initialization callback for all applicable beans...
for (String beanName : beanNames) {
    Object singletonInstance = getSingleton(beanName);
    if (singletonInstance instanceof SmartInitializingSingleton) {
        StartupStep smartInitialize = this.getApplicationStartup().start("spring.beans.smart-initialize")
            .tag("beanName", beanName);
        SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
        if (System.getSecurityManager() != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                smartSingleton.afterSingletonsInstantiated();
                return null;
            }, getAccessControlContext());
        }
        else {
            smartSingleton.afterSingletonsInstantiated();
        }
        smartInitialize.end();
    }
}
```

# Bean 创建核心流程

`getBean()` 方法是 Bean 创建核心流程的核心，而该方法的关键步骤是 `doGetBean()` 方法。

## doGetBean 方法概览

对于 `doGetBean()` 方法，各部分实现如下所示，方法体部分简化：

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
    规范化 BeanName，处理别名和 FactoryBean 前缀;
    如果单例缓存命中且无构造参数，直接以缓存 Bean 作为最终实例（含 FactoryBean 产品处理）;
    否则 {
        原型 Bean 循环依赖检测;
        如果该容器内不含对应 BeanDefinition 且存在父容器，则尝试从父容器中获取 Bean 并返回;
        如果不是类型检查模式，则标记该 Bean 已进入创建阶段;
        获取合并后的 BeanDefinition;
        若存在依赖 dependsOn 依赖，先初始化依赖的 Bean;
        判断作用域 {
            如果是单例作用域，则从单例池中创建或获取 Bean;
            如果是原型作用域，则每次创建新的 Bean 实例;
            如果是自定义作用域，则根据 Scope 规则创建或获取 Bean;
        }
    }
    将最终实例转换为用户需要的对象（处理 FactoryBean 产品与类型适配）并返回;
}
```

## BeanName 规范化

在 Spring 中，一个 Bean 可能同时拥有别名、规范名称以及特殊前缀，Spring 内部的数据结构大多都基于规范 BeanName 作为 key。通过 `transformedBeanName(name)` 方法，将传入的名称统一转为容器内部使用的标准 BeanName，确保后续操作都基于规范 BeanName 进行。

```java
String beanName = transformedBeanName(name);
```

## 单例缓存快速返回

在 Spring 中，单例 Bean 会在创建后被缓存到一级缓存 `singletonObjects`，以确保整个应用中只存在一个实例。因此，当调用 `getBean()` 时，Spring 会首先通过 `getSingleton(beanName)` 方法尝试从单例缓存中获取实例。如果 Bean 已存在于单例缓存中且本次获取不需要传入构造参数，Spring 会直接返回缓存对象，避免重复实例化。

Spring 对 `singleton Bean` 只会创建并缓存一个实例。但当调用 `getBean(beanName, args)` 且显式传入构造参数时，Spring 无法保证这些参数与当前单例实例的构造参数一致，因此不能安全复用已有单例实例，必须强制进入完整创建流程。因此即使缓存中存在 Bean，也不会走单例缓存快速返回分支。

为确保不同类型 Bean 都能拿到符合语义的最终 Bean，即使缓存命中，也必须对得到的共享实例 `sharedInstance` 调用 `getObjectForBeanInstance()` 方法。若共享实例是普通 Bean，则直接返回；若共享实例是 FactoryBean，则返回其产品对象。

```java
// 最终返回的 Bean 实例
Object beanInstance;
// Eagerly check singleton cache for manually registered singletons.
Object sharedInstance = getSingleton(beanName);
if (sharedInstance != null && args == null) {
    ... 日志输出 ...
    beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
}
```

## 创建流程前置检查

### 原型作用域Bean循环依赖检测

对于单例 Bean 的循环依赖情况，Spring 基于三层缓存进行了支持，通过提前暴露对象部分构造的方式破除单例 Bean 的循环依赖。然而，`prototype Bean` 每次调用都必须返回全新实例，其语义与暴露早期对象并缓存的三级缓存模型存在根本冲突，因此 Spring 不会也无法为 prototype 提供类似单例的循环依赖解决机制。

所以，使用 `isPrototypeCurrentlyInCreation(beanName)` 方法检测当前的原型 Bean 是否正在创建流程，如果正在创建，则认为当前的原型 Bean 出现了循环依赖，抛出 `BeanCurrentlyInCreationException` 终止创建，避免无限递归。

```java
if (isPrototypeCurrentlyInCreation(beanName)) {
    throw new BeanCurrentlyInCreationException(beanName);
}
```

### 父容器委派加载

若当前 BeanFactory 中没有定义该 BeanDefinition 时，则委派其父 BeanFactory 继续查找和实例化 Bean，以解决 Spring 项目容器嵌套过程中的 Bean 单向可见性，即子容器可访问父容器中的 Bean，但父容器不可访问子容器中的 Bean。

`originalBeanName()` 方法内部会将传入的名称转为要查找的规范名称，而不再使用别名。这一步与 BeanName 规范化的区别是：只会转换别名，不会去除 FactoryBean 的特殊前缀。

如果父容器是 `AbstractBeanFactory`，则直接递归调用 `doGetBean()` 传入当前的所有参数进行强委派；如果父容器不是 `AbstractBeanFactory`，则必须根据参数情况（对应参数是否存在）选择合适的 `getBean()` 重载版本。

```java
// Check if bean definition exists in this factory.
BeanFactory parentBeanFactory = getParentBeanFactory();
if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
    // Not found -> check parent.
    String nameToLookup = originalBeanName(name);
    if (parentBeanFactory instanceof AbstractBeanFactory) {
        return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
            nameToLookup, requiredType, args, typeCheckOnly);
    }
    else if (args != null) {
        // Delegation to parent with explicit args.
        return (T) parentBeanFactory.getBean(nameToLookup, args);
    }
    else if (requiredType != null) {
        // No args -> delegate to standard getBean method.
        return parentBeanFactory.getBean(nameToLookup, requiredType);
    }
    else {
        return (T) parentBeanFactory.getBean(nameToLookup);
    }
}
```

### 标记Bean创建状态

`typeCheckOnly` 表示当前 `doGetBean()` 的调用用来在解析依赖时进行类型匹配，并未真正打算创建 Bean。只有当本次调用的目的不是检查类型时，才会将当前 Bean 标记为**已开始创建**，以便 Spring 在后续流程中进行循环依赖检测、生命周期管理等。

```java
if (!typeCheckOnly) {
    markBeanAsCreated(beanName);
}
```

## BeanDefinition 合并与验证

Spring 中存在三类 BeanDefinition，分别是原始 BeanDefinition、父子 BeanDefinition、最终的 RootBeanDefinition。通过 `getMergedLocalBeanDefinition()` 方法将父 BeanDefinition 与子 BeanDefinition 合并为最终的 RootBeanDefinition，即 Spring 创建 Bean 时实际使用的完整 BeanDefinition。

通过 `checkMergedBeanDefinition` 校验合并后的 BeanDefinition 是否满足实例化要求。在创建流程中，主要用于判断当前 BeanDefinition 是否是抽象类，只有在非抽象类的情况下才能继续实例化，否则抛出 `BeanIsAbstractException` 异常。该方法只检查 BeanDefinition 是否为抽象定义，因为抽象 Bean 在语义上明确禁止被实例化，其余配置合法性校验在注册阶段或实例化过程中完成。

```java
RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
checkMergedBeanDefinition(mbd, beanName, args);
```

## 依赖预初始化

在真正初始化 Spring Bean 之前，需要提前初始化当前 Bean 的 `dependsOn` 依赖，确保它们在当前 Bean 之前完成创建，且不会出现循环依赖。`dependsOn` 是一种初始化顺序依赖，用于显式声明当前 Bean 在实例化之前，必须先完成 `dependsOn Bean` 的初始化。这与 `@Autowired` 的依赖不同，`@Autowired` 依赖只负责在 Bean 创建后自动填充引用，不影响创建顺序。

获取到对应的依赖项后，对依赖项列表进行遍历。首先通过 `isDependent(beanName, dep)` 方法进行循环依赖判断，对循环依赖的情况，直接抛出异常，避免死循环发生。随后通过 `registerDependentBean()` 注册依赖关系，用于控制销毁顺序、管理依赖关系、辅助检测循环依赖。当预处理完成后，调用 `getBean()` 方法预先创建 `dependsOn` 的 Bean。如果依赖 Bean 不存在，则抛出对应异常。

```java
String[] dependsOn = mbd.getDependsOn();
if (dependsOn != null) {
   for (String dep : dependsOn) {
      if (isDependent(beanName, dep)) {
         throw new BeanCreationException(mbd.getResourceDescription(), beanName,
               "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
      }
      registerDependentBean(dep, beanName);
      try {
         getBean(dep);
      }
      catch (NoSuchBeanDefinitionException ex) {
         throw new BeanCreationException(mbd.getResourceDescription(), beanName,
               "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
      }
   }
}
```

## 根据作用域创建Bean实例

### 单例作用域

如果当前 Bean 作用域为 singleton，则 Spring 需要按照单例池 + 三级缓存策略创建对象。在 `getSingleton()` 方法中，首先会查一级缓存判断是否存在 Bean。如果该单例 Bean 存在，则直接返回单例 Bean；如果单例 Bean 正在创建，则从二级缓存或三级缓存返回早期暴露对象来解决循环依赖问题；如果不存在，则执行 `createBean()` 回调方法创建对应单例。如果创建失败，则清理缓存。

最终，使用 `getObjectForBeanInstance()` 方法获取最终 Bean 实例。该方法主要是为了实现普通 Bean 和 FactoryBean 的统一处理，保证 FactoryBean 返回其产品 Bean，后续的使用都是这个目的。

```java
// Create bean instance.
if (mbd.isSingleton()) {
   sharedInstance = getSingleton(beanName, () -> {
      try {
         return createBean(beanName, mbd, args);
      }
      catch (BeansException ex) {
         // Explicitly remove instance from singleton cache: It might have been put there
         // eagerly by the creation process, to allow for circular reference resolution.
         // Also remove any beans that received a temporary reference to the bean.
         destroySingleton(beanName);
         throw ex;
      }
   });
   beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```

### 原型作用域

对于原型作用域 Bean，每次调用 `doGetBean()` 都创建一个新的 Bean 实例。在 Bean 创建前和创建后，会分别进行 before 和 after 方法回调管理 prototype 创建状态，并使用 createBean() 方法完成 Bean 的创建。Spring 中通过 ThreadLocal 保存 `prototype Bean` 的创建状态，其中维护一个 Set 集合，用于存储正在创建的 prototype Bean 的 BeanName，之所以使用 ThreadLocal ，原因在于 `prototype Bean` 的创建状态只对当前线程有效，跨线程共享既没有意义，也会引入错误的循环依赖判断。

`beforePrototypeCreation()` 方法用于标记 Bean 状态为正在创建。由于 prototype 不能通过三级缓存解决循环依赖问题，因此必须通过状态回调方法防止递归创建时进入无限循环。`afterPrototypeCreation()` 用于从 ThreadLocal 中移除正在创建的 prototype 标记。

```java
else if (mbd.isPrototype()) {
   // It's a prototype -> create a new instance.
   Object prototypeInstance = null;
   try {
      beforePrototypeCreation(beanName);
      prototypeInstance = createBean(beanName, mbd, args);
   }
   finally {
      afterPrototypeCreation(beanName);
   }
   beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
}
```

### 自定义作用域

对于非单例作用域或原型作用域的 Bean，最终会进入到自定义作用域 Bean 的判定阶段。内部会首先判断 scopeName 的合法性，包括当前 scopeName 是否为空、是否定义了对应 Scope 的实现。例如，如果在非 Web 环境下尝试使用 session 作用域，则抛出异常，说明当前作用域未激活或不存在，无法获取该 Bean。

判定如果通过，则沿用 prototype 的创建状态管理模型，以检测循环依赖并维护创建状态。同时，Spring 会将 Bean 的创建与缓存策略完全委托给对应的 `Scope` 实现。

```java
else {
   String scopeName = mbd.getScope();
   if (!StringUtils.hasLength(scopeName)) {
      throw new IllegalStateException("No scope name defined for bean ´" + beanName + "'");
   }
   Scope scope = this.scopes.get(scopeName);
   if (scope == null) {
      throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
   }
   try {
      Object scopedInstance = scope.get(beanName, () -> {
         beforePrototypeCreation(beanName);
         try {
            return createBean(beanName, mbd, args);
         }
         finally {
            afterPrototypeCreation(beanName);
         }
      });
      beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
   }
   catch (IllegalStateException ex) {
      throw new ScopeNotActiveException(beanName, scopeName, ex);
   }
}
```

## FactoryBean 产品对象解析与类型适配

获取到最终的 Bean 实例后，对已获取的 Bean 实例进行最终适配，使其满足调用方对类型和语义的要求，然后安全返回。它主要负责校验 Bean 实例是否满足 `requiredType`，并在必要时执行类型转换，当类型不匹配时，确保不返回错误 Bean 实例而是直接抛出异常。

之所以需要确认和适配 Bean 实例，是因为 Bean 实例的创建流程是依据 BeanName 实现的。在实现过程中不关心是否匹配用户需要的结果，因此在最终返回阶段需要对 Bean 实例进行处理，确保返回对象确实可以赋值给接收方，必要时进行类型转换或抛出异常。

```java
return adaptBeanInstance(name, beanInstance, requiredType);
```

# 测试

分别在 `scanner.scan("com.example.b01");` 和 `context.refresh();` 打断点，观察容器状态。

![image-20251212110450922](D:\Majinliang\Documents\笔记\博文\Inbox\image-20251212110450922.png)

Debug 执行。在 `scanner.scan("com.example.b01");` 执行前，可以看到 `beanDefinitonMap` 中只有5个 BeanDefinition。

![image-20251212104247355](D:\Majinliang\Documents\笔记\博文\Inbox\image-20251212104247355.png)

执行后，对 `com.example.b01` 包进行扫描，`beanDefinitonMap` 中加入对 `MyComponent` 的 BeanDefinition。

![image-20251212110241282](D:\Majinliang\Documents\笔记\博文\Inbox\image-20251212110241282.png)

在 `scanner.scan()` 方法执行后，`refresh()` 执行前，对应 beanFactory 中的 `singletonObject` 为空。

![image-20251212104817354](D:\Majinliang\Documents\笔记\博文\Inbox\image-20251212104817354.png)

`refresh()` 执行后，一共有 14 个单例 Bean 被实例化，这 14 个单例 Bean 既包括之前 BeanDefinition 的实例化，也有 `refresh()` 方法执行过程中额外创建的基础设施 Bean。

![image-20251212105914334](D:\Majinliang\Documents\笔记\博文\Inbox\image-20251212105914334.png)

最终，执行：

```java
MyComponent component = context.getBean(MyComponent.class);
component.sayHello();
```

得到最终 Bean 输出。

![image-20251212110025189](D:\Majinliang\Documents\笔记\博文\Inbox\image-20251212110025189.png)