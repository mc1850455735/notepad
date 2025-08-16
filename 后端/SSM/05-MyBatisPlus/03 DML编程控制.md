# 新增

### id生成策略控制

- 通常来讲, 不同的表拥有不同的id生成策略
	- 日志: 自增
	- 订单: 特殊规则生成的订单号 (JD23876SDxxxx)
	- 外卖单:  关联地区日期等信息 (SF202508163491)
	- 关系表: 可省略id ...
- 通过注解 `@TableId` , 将该注解放置在表示主键的属性定义上方, 即可设置当前类中主键属性的生成策略
- 参数: 
	- value => 用来设置数据库中主键的名称, 默认为字段名称
	- type => 设置主键属性的生成策略, 值按照IdType枚举值
- 常用type主键策略
	- `AUTO` : 默认方式, 使用自增主键
	- `NONE` : 无主键策略
	- `INPUT` : 用户输入id
		- 配置时如果不输入, 则报错
	- `ASSIGN_UUID` : 分配uuid默认实现类
	- `ASSIGN_ID` : 雪花算法生成id (默认)
		- 如果用户指定了ID, 使用用户指定的id, 否则通过雪花算法生成id
```java
@Data  
@TableName("tbl_user")  
public class User {  
    @TableId(type = IdType.AUTO)  
    private Long id;  
    ...
}
```

**雪花算法解析**
- 雪花算法对于Long, 会生成64位二进制数, 即一个Long类型, 其中的位数划分如下图所示
![[后端/SSM/05-MyBatisPlus/Inbox/Pasted image 20250816212337.png]]

### 全局策略配置

##### id分配策略
- 通过yaml配置, 可以使每个实体类都使用一个默认的相同id分配策略
```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: assign_id
```

##### 表名映射策略
- 通过设置表名前缀, 可以将所有的实体类名称前面都加入一串固定的字符串 (如`tbl_`) , 以省略配置
```yaml
mybatis-plus:
  global-config:
    db-config:
      table-prefix: tbl_
```


# 删除

### 多数据删除/查询

- 使用方法 `deleteBatchIds()` / `selectBatchIds` , 传入一个集合, 即可实现批量删除/查询
- 注意, 新版MyBatisPlus已经改为了 `deleteByIds` / `selectByIds`

**删除**
```java
@Test  
void testDelete() {  
    ArrayList<Long> list = new ArrayList<>();  
    list.add(666L);  
    list.add(1956714076948664321L);  
    list.add(1956714143814377473L);  
    userDao.deleteByIds(list);  
}
```

**查询**
```java
@Test
void testSelectByIds() {
    ArrayList<Long> list = new ArrayList<>();
    list.add(1L);
    list.add(2L);
    list.add(3L);
    List<User> users = userDao.selectByIds(list);
    System.out.println(users);
}
```

### 逻辑删除

- 删除操作业务问题: 业务数据从数据库中丢弃
- 逻辑删除 : 为数据设置是否可用状态字段, 删除时设置状态字段为不可用状态, 数据**保留**在数据库中

##### 操作步骤
- 在数据表中添加逻辑删除状态属性
- 在模型类中添加逻辑删除条件字段
- 在逻辑删除条件字段上添加`@TableLogic`注解, 标志为逻辑字段
- 在注解参数中, 设置默认值`value`和逻辑删除值`delval`
```java
@Data  
public class User {  
    ...
    // 逻辑删除字段, 标志当前记录是否被删除  
    @TableLogic(value = "0", delval = "1")  
    private Integer deleted;  
}
```

##### 逻辑操作变化
- 设置完逻辑删除字段后再执行删除操作, 就会让delete相关方法由原来的直接删除改为修改deleted字段, 即使用update语句执行逻辑删除操作
- 查询时, 只会查询没有被视为逻辑删除的字段, 对于已经删除的字段默认不会进行查询

##### 通用配置
- 同理, 可以通过通用配置, 直接配置逻辑删除时, 哪个字段为逻辑删除所用的字段, 字段中哪个值为未删除, 哪个值为已删除
```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-not-delete-value: 0
      logic-delete-value: 1
```

# 乐观锁

- 业务并发现象导致的问题 : 秒杀问题
- 通过乐观锁, 可以简单解决部分秒杀问题

**什么是乐观锁**
- 假设冲突很少发生, 所以在读取数据时不加锁, 只在更新时检查数据是否被改过, 如果被修改过, 那么更新失败, 重试该操作或放弃操作
- 修改时, 先获取要修改数据的version, 并以该version为条件进行修改操作, 并在修改后将version + 1
- 任一线程修改完成version后, 都会导致其他线程由于version条件不匹配而修改失败, 失败后重新获取version申请修改, 从而达成了加锁目的

**步骤**
- 在数据库表中加入一个字段`version`表示当前数据库中谁在操作这条数据, 数据类型为int, 长度为11, 设置默认值为1
- 在实体类中加入一个对应的Integer字段, 名为version, 并为其加入注解`@Version`
- 加入对应拦截器, 为拥有乐观锁的更新操作实现增强
```java
@Configuration  
public class MpConfig {  
    @Bean  
    public MybatisPlusInterceptor mybatisPlusInterceptor() {  
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();  
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());  
        return interceptor;  
    }  
}
```

**使用**
- 通过update语句返回值是否为0, 可以判断操作成功或失败
- 使用乐观锁时, 应手动提供version值, 通常使用如下方法
1. 先通过要修改数据的id将当前数据查询出来
2. 将要修改的属性逐一设置到查询出来的值
3. 执行带拦截器的update操作
```java
@Test
void testUpdate() {
    User user = userDao.selectById(5L);
    user.setName("Jack888");
    userDao.updateById(user)
}
```

