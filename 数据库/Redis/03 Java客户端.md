
#  概述

- Redis官网提供了各种语言的客户端 https://redis.io/clients
- 建议使用的客户端使用星标记
- Java中, Redis官方推荐以下几种
- Jedis : 
	- 以Redis命令作为方法名, 学习成本低, 简单实用
	- Jedis实例是线程不安全的, 多线程环境下需要基于连接池使用
- Lettuce :
	- 基于Netty实现, 性能较高
	- 支持同步, 异步和响应式编程方式, 并且是线程安全的
	- 支持Redis哨兵模式, 集群模式和管道模式
	- Spring官方默认
- Redisson :
	- 是一个基于Redis实现的分布式, 可伸缩的Java数据结构集合
	- 包含了诸如Map, Queue, Lock, Semaphore, AtomicLong等功能
- Spring Data Redis : 
	- 底层兼容Jedis和Lettuce , 对二者实现了整合

# Jedis

**使用**
- Jedis官方网址 : https://github.com/redis/jedis
- 引入依赖
```xml
<dependency>  
  <groupId>redis.clients</groupId>  
  <artifactId>jedis</artifactId>  
  <version>3.7.0</version>  
</dependency>
```
- 建立连接
```java
@BeforeEach  
void setUp() {  
    // 1. 建立连接  
    jedis = new Jedis("192.168.199.199", 6379);  
    // 2. 设置密码  
    jedis.auth("123321");  
    // 3. 选择要使用的库  
    jedis.select(0);  
}
```
- 测试string
```java
@Test  
void testString() {  
    // 存入数据  
    String result = jedis.set("name", "虎哥");  
    System.out.println(result);  
    // 获取数据  
    String name = jedis.get("name");  
    System.out.println(name);  
}
```
- 释放资源
```java
@AfterEach  
void tearDown() {  
    if(jedis != null)  
        jedis.close();  
}
```

# Jedis连接池

- Jedis本身是线程不安全的, 所以并发环境下, Jedis要给每一个线程创建单独的Jedis对象
- 由于频繁的创建和销毁Jedis对象会有性能损耗, 所以推荐使用连接池替代Jedis的直接连接
- 创建连接池
```java
public class JedisConnectionFactory {  
    private static final JedisPool jedisPool;  
    static {  
        // 配置连接池  
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();  
        // 设置最大连接数  
        jedisPoolConfig.setMaxTotal(8);  
        // 设置最大空闲连接数  
        jedisPoolConfig.setMaxIdle(8);  
        // 设置最小空闲连接数  
        jedisPoolConfig.setMinIdle(0);  
        // 当连接池没有连接可用时的等待时间, 默认-1, 无限等待  
        jedisPoolConfig.setMaxWaitMillis(1000);  
        // 创建连接池对象  
        jedisPool = new JedisPool(jedisPoolConfig,  
                "192.168.199.199", 6379, 1000, "123321");  
    }  
    public static Jedis getJedis() {  
        return jedisPool.getResource();  
    }  
}
```
- 修改获取连接方法
- 使用连接池时, 释放Jedis连接实际上就是将该连接归还连接池
```java
@BeforeEach  
void setUp() {  
    // 1. 建立连接  
    jedis = JedisConnectionFactory.getJedis();  
    // 2. 设置密码  
    jedis.auth("123321");  
    // 3. 选择要使用的库  
    jedis.select(0);  
}
```

# SpringDataRedis

**概述**
- SpringData是Spring中数据操作的模块, 包含对各种数据库的集成, 其中对Redis做集成的模块就叫做SpringDataRedis
- 官网地址 : [Spring Data Redis](https://spring.io/projects/spring-data-redis)
- 优势
	- 提供了对不同Redis客户端的整合 (Jedis和Lettuce)
	- 提供了RedisTemplate统一API来操作Redis
	- 支持Redis的发布订阅模型
	- 支持Redis哨兵和Redis集群
	- 支持基于Lettuce的响应式编程
	- 支持基于JDK, JSON, 字符串, Spring对象的数据序列化和反序列化
	- 支持基于Redis的JDKCollection实现


### RedisTemplate

- SpringDataRedis中提供了RedisTemplate工具类, 其中封装了各种对Redis的操作, 并且将不同数据类型的操作API封装到不同类型中
![[数据库/Redis/Inbox/Pasted image 20250913100112.png]]

##### 快速入门

- 引入依赖
```xml
<!-- redis template依赖 -->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>  
<!-- 连接池依赖 -->  
<dependency>  
    <groupId>org.apache.commons</groupId>  
    <artifactId>commons-pool2</artifactId>  
</dependency>
```
- 配置yaml文件
	- Spring默认使用lettuce连接池, 配置连接池后生效
	- 如果使用jedis连接池, 需要引入jedis相关依赖
```yaml
spring:  
  data:  
    redis:  
      host: 192.168.199.199  
      port: 6379  
      password: 123321  
      lettuce:  
        pool:  
          max-active: 8  
          max-idle: 8  
          min-idle: 0  
          max-wait: 100
```
- 注入RedisTemplate
```java
@Autowired  
private RedisTemplate redisTemplate;
```
- 编写测试代码
```java
@Test  
void testString() {  
    // 插入一条string类型的数据  
    redisTemplate.opsForValue().set("name", "李四");  
    // 读取一条string类型的数据  
    Object name = redisTemplate.opsForValue().get("name");  
    System.out.println("name= " + name);  
}
```

##### 序列化

**默认序列化器**
- 当使用RedisTemplate进行数据写入和读出时, 会首先经过序列化器进行序列化和反序列化
- 如果用户不指定, 默认使用JDK序列化工具, 其会将Java对象转为字节以后再传入Redis中
- 缺点
	- 可读性差, 在Redis中存储的键值不具备直接阅读性, 且可能引起误解
	- 内存占用大, 序列化后比较原值占用空间明显增大

**序列化器**
- StringRedisSerializer : 
	- 字符串序列化器, 将键值作为字符串处理
	- 如果传入的key和hash key都是字符串情况, 使用此序列化器
	- 一般适合在key使用, 因为key通常是String
- GenericJackson2JsonRedisSerializer
	- 将对象等转换为JSON字符串形式进行处理, 同时默认会将对应的类信息存入JSON中, 如 `"@class": "top.majinliang.pojo.User"`
	- 适用于传入的value是对象的形式
	- 使用时, 注意需要有 `Jackson` 依赖

**自定义序列化方法**
- `RedisSerializer.string()` : 默认提供的UTF-8字符串序列化器 
```java
@Bean  
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {  
    RedisTemplate<String, Object> template = new RedisTemplate<>();  
    template.setConnectionFactory(redisConnectionFactory);  
  
    template.setKeySerializer(RedisSerializer.string());  
    template.setHashKeySerializer(RedisSerializer.string());  
    GenericJackson2JsonRedisSerializer jsonRedisSerializer = 
        new GenericJackson2JsonRedisSerializer();  
    template.setValueSerializer(jsonRedisSerializer);  
    template.setHashValueSerializer(jsonRedisSerializer);  
  
    return template;
}
```

**使用示例**
```java
@Test  
void testSaveUser() {  
    User user = new User(1, "majinliang");  
    // 存储信息  
    redisTemplate.opsForValue().set("user:1", user);  
    // 获取信息  
    User u1 = (User) redisTemplate.opsForValue().get("user:1");  
    System.out.println(u1);  
}
```

##### StringRedisTemplate
- 为了节省内存空间, 并不会使用JSON序列化器来处理value, 而是使用统一的String序列化器, 要求只能存储String类型的key和value
- 需要存储Java对象时, 手动完成对象的序列化和反序列化
- Spring默认提供了一个StringRedisTemplate类, 它的key和value的序列化方法默认就是String方式
- 通过这种方式, 省去了定义RedisTemplate的过程
```java
private static final ObjectMapper mapper = new ObjectMapper();
@Test  
void testSaveUser() throws JsonProcessingException {  
    // 创建对象  
    User user = new User(2, "zhuangxiaohan");  
    // JSON序列化, 并进行写入  
    String value = mapper.writeValueAsString(user);  
    template.opsForValue().set("user:200", value);  
    // 读取, 并进行JSON反序列化  
    String result = template.opsForValue().get("user:200");  
    User readValue = mapper.readValue(result, User.class);  
    System.out.println(readValue);  
}
```


##### Hash基础

- SpringDataRedis中, 很多方法与Redis命令不同
- 如Hash , 其方法名更接近于Java的HashMap

**代码示例**
```java
@Test  
void testHash() {  
    template.opsForHash().put("user:998", "name", "majinliang");  
    template.opsForHash().put("user:998", "age", "98");  
    Map<Object, Object> entries = 
        template.opsForHash().entries("user:998");  
    System.out.println(entries);  
}
```

