

`@Autowired` 是 Spring 提供的**按类型注入**的注解，若存在多个同类型 Bean 需配合 `@Qualifier` 指定名称。
`@Resource` 是 Java 标准注解，默认按名称匹配，名称未匹配时按类型注入。

前者依赖 Spring 框架，后者更通用且支持名称/类型双策略。  


### 来源不同

- **@Autowired**：Spring 框架自定义的注解（属于 Spring Core 模块），与 Spring 深度绑定。
- **@Resource**：Java 标准注解（JSR-250 规范），属于 Java EE（现 Jakarta EE）的一部分，不依赖 Spring，可用于其他支持 JSR-250 的 DI 容器（如 Java EE 应用服务器）。

### 注入策略不同

#### @Autowired

默认**按类型（byType）匹配**：

- 首先根据字段/方法参数的类型在 Spring 容器中查找匹配的 Bean。
- 如果存在多个同类型的 Bean（例如同一个接口的多个实现类），会自动**按名称（byName）匹配**：
	- **字段名**或**方法参数名**需与 Bean 的名称（@Bean 或 @Component 指定的 name）一致；
	- 若仍无法匹配，需通过 @Qualifier("beanName") 显式指定要注入的 Bean 名称。

#### @Resource

默认**按名称（byName）匹配**：

- 优先根据name属性指定的名称查找 Bean（name需与 Bean 的名称完全一致）；
- 若未显式设置name，则默认使用**字段名或方法名**作为名称查找；
- 若名称匹配失败，才会**回退到按类型（byType）匹配**（仅当名称不存在时触发）。

### 支持的注入位置不同

- **@Autowired**：支持**更灵活**的注入位置，可标注在构造方法（推荐，显式依赖）、字段、方法、参数上。
- **@Resource**：通常用于字段和 setter 方法，**不支持构造方法和参数注入**。

### 依赖的强制性

- **@Autowired**：默认要求依赖必须存在 (`required = true`)，否则启动时抛出 `NoSuchBeanDefinitionException` 异常；
	- 可通过设置 required 属性为 false ( 即`@Autowired(required = false)` 允许依赖为 null )，此时注入失败时字段为 null。
- **@Resource**：没有 required 属性，依赖必须存在，否则启动时抛出 `NoSuchBeanDefinitionException` 异常 (与 `@Autowired` 类似)。

### 与 Spring 特性的集成

- **@Autowired**：与 Spring 的其他注解（如@Qualifier、@Primary、@Lazy）深度配合，支持更复杂的依赖注入逻辑。  
- **@Resource**：作为标准注解，功能相对基础，不支持与 @Primary、@Lazy 等 Spring 特有的注解直接配合，但是可以通过其他方式实现。
