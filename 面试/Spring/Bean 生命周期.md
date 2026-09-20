
在 Spring 中，Bean 的生命周期指一个 Bean 从创建到销毁的完整过程，Spring 容器会在各个阶段提供扩展点，允许开发者插入自定义逻辑。其核心流程可分为 4 个阶段，容器通过回调机制在各阶段执行特定扩展点。。

1. 实例化（构造函数）
2. 属性注入（依赖注入）
3. 初始化
4. 使用阶段
5. 销毁

### 1. 实例化（Instantiation）

- **目标**：通过**反射**创建 Bean 的原始对象（未初始化状态）。
- **触发时机**：容器启动时（单例 Bean）或首次获取 Bean 时（原型 Bean）。
- **说明**：此时对象仅分配了内存，属性未填充，是最原始的状态。

### 2. 属性注入（Population）

- **目标**：填充 Bean 的依赖属性（如通过 @Autowired、@Resource 或 XML 配置的 property）。
- **触发时机**：实例化后，初始化前。
- **说明**：依赖注入完成后，Bean 对象的属性已被赋值，但尚未执行自定义初始化逻辑。

### 3. 初始化（Initialization）

完成 Bean 的最终初始化，使其达到可使用状态

执行Aware接口、BeanPostProcessor前置处理、@PostConstruct、InitializingBean、自定义init方法

**关键步骤（按执行顺序）**：
- **Aware 接口回调**：若 Bean 实现了Aware系列接口（如 ApplicationContextAware、BeanFactoryAware、BeanNameAware），Spring 会回调这些接口的方法，注入容器相关资源。例如： setApplicationContext(ApplicationContext ctx) 会注入当前容器实例。
- **BeanPostProcessor 前置处理**：若存在 BeanPostProcessor 实现类，会调用其 postProcessBeforeInitialization(Object bean, String beanName) 方法，允许在初始化前修改 Bean（如动态代理、属性校验）。
- **@PostConstruct 注解方法**：使用 JSR-250 规范的 @PostConstruct 注解标记的方法会被执行 (需配合 `CommonAnnotationBeanPostProcessor` )。
- **InitializingBean 接口**：若 Bean 实现了 InitializingBean 接口，会调用其 `afterPropertiesSet()` 方法。
- **自定义 init-method**：通过 XML 配置 (init-method) 或 `@Bean(initMethod = "xxx")` 指定的初始化方法会被执行。
- **BeanPostProcessor 后置处理**：BeanPostProcessor 的 `postProcessAfterInitialization(Object bean, String beanName)` 方法会被调用，常用于生成代理对象（如 AOP）或最终修饰 Bean。

- 总结：初始化阶段是**开发者自定义逻辑**的核心扩展点，允许在 Bean 可用前完成资源初始化（如连接池、配置加载）。

### 4. 使用

略

### 5. 销毁（Destruction）

@PreDestroy、DisposableBean接口或自定义destroy方法

- **目标**：容器关闭时释放 Bean 的资源（如关闭连接、清理临时文件）。
- **触发时机**：仅对单例 Bean 有效（原型 Bean 由用户管理生命周期，容器不负责销毁）。
