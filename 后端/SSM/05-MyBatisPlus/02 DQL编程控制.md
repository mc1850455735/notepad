
# 条件查询基础

- MyBatisPlus将复杂的SQL查询条件进行了封装, 使用编程的形式完成了查询条件的组合
- 使用 `Wrapper<T>` 接口实现查询条件的封装, 使用 `QueryWrapper<T>` 进行基本查询条件的书写, 在泛型类型中填入自己要查询的类(非lambda查询不写也能运行)
- 使用它的方法设置对不同字段的不同条件, 有以下三种方法

### 按条件查询
- 条件字段中使用字符串表示要用来比较的列, 缺点是容易写错列名导致条件设置无效
- 使用字符串表示列的Wrapper不强制要求指定泛型类型
```java
@Test
void testGetAll01(){
    QueryWrapper wrapper = new QueryWrapper();
    wrapper.lt("age", 20);
    List<User> users = userDao.selectList(wrapper);
    System.out.println(users);
}
```

### Lambda格式条件查询
- 在原`QueryWrapper`对象的基础上, 在其后面加入`.lambda`方法使其成为一个lambda查询条件
- 使用lambda查询条件, 可以在设置条件时, 直接使用类的字段作为查询条件所在列, 避免输出错误导致条件无效
- 对于lambda查询, 必须在`QueryWrapper`中指定其泛型, 否则会报错
```java
@Test  
void testGetAll02(){  
    QueryWrapper<User> wrapper = new QueryWrapper<>();  
    wrapper.lambda().gt(User::getAge, 18);  
    List<User> users = userDao.selectList(wrapper);  
    System.out.println(users);  
}
```

### Lambda Wrapper格式条件查询(推荐)
- 直接使用`LambdaQueryWrapper`对象, 可以省去普通`QueryWrapper`转化到lambda的过程
```java
@Test
void testGetAll03(){
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();
    lqw.lt(User::getAge, 15);
    System.out.println(userDao.selectList(lqw));
}
```

### 多条件查询
- 使用多个条件设置方法的叠加, 即可实现多条件查询(大于xx而小于yy)
- 对于较多条件方法设置的情况, 可以直接以方法链的形式进行书写
```java
@Test  
void testGetAll04(){  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.gt(User::getAge, 15);  
    lqw.lt(User::getAge, 20);  
    System.out.println(userDao.selectList(lqw));  
}
/* 或 */
@Test
void testGetAll04(){
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();
    lqw.gt(User::getAge, 15)
        .lt(User::getAge, 20);
    System.out.println(userDao.selectList(lqw));
}
```

- 当需要实现或方式查询时 (如年龄小于7或大于19的所有用户), 则需要在条件配置方法链中加入`or()`方法
```java
@Test  
void testGetAll05() {  
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();  
    lqw.lt(User::getAge, 10).or().gt(User::getAge, 30);  
    System.out.println(userDao.selectList(lqw));  
}
```

### null值处理

- 对于查询上下限等情况, 应有一个专门的查询类与现有的类对应, 以方便定义相关的上下限, 用来接收相关的查询数据
- query包定义在domain包下, 表示其实际为domain的一部分
```java
package top.majinliang.domain.query;  
  
@Data  
public class UserQuery extends User {  
    private Integer age2;  
}
```

- 当存在为null的字段时, 如果直接设置查询条件, 会导致为null的字段作为查询条件永远无法被满足, 从而无法查出任何结果
- 如果需要做到忽略null相关字段, 可以在查询方法中加入condition参数进行判断
```java
@Test
void testGetAll06() {
    UserQuery query = new UserQuery();
    query.setAge(10);
    query.setAge2(20);
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper<>();
    lqw.gt(null != query.getAge(), User::getAge, query.getAge());
    lqw.lt(null != query.getAge2(), User::getAge, query.getAge2());
    System.out.println(userDao.selectList(lqw));
}
```

# 查询投影

- 通过设置查询投影, 可以设置在本次查询中需要查询哪些字段, 没有设置的字段会默认置为null
- 通过select方法, 进行投影字段的设置

### Lambda查询投影设置
- 同上, LambdaQueryWrapper进行投影设置时, select中的参数应该为类的字段, 如下代码所示
```java
@Test  
void testGetAll07() {  
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();  
    wrapper.select(User::getName, User::getAge, User::getTel);  
    List<User> userList = userDao.selectList(wrapper);  
}
```

### 普通查询投影设置
- 对于非Lambda的QueryWrapper进行投影设置时, select中的参数需要填入字符串形式的其名称
```java
@Test  
void testGetAll08() {  
    QueryWrapper<User> wrapper = new QueryWrapper<>();  
    wrapper.select("name", "age", "tel");  
    List<User> userList = userDao.selectList(wrapper);  
}
```

### MySQL函数查询
- 如果想要进行如 `count(*)` 等统计工作, 则同样可以使用select方法, 使用字符串输入对应函数
	- 对于某些不支持的函数, 直接恢复到MyBatis时使用SQL语句的方式
- 书写MySQL函数统计的列名时, 不应使用selectList调用, 因为该函数其会返回一个`List<User>`, 不能接收统计结果, 而是应该使用 `selectMaps` 方法, 将对应wrapper传入
- `selectMaps` 方法的返回值为 `List<Map<String, Object>>` , 即一个由String映射到Object的Map对象数组
  - String代表key, Object代表value
- 可以通过 `count(*) as count` 来设置统计列的名称
```java
@Test
void testGetAll09() {
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.select("count(*) as count");
    List<Map<String, Object>> maps = userDao.selectMaps(wrapper);
}
```

### MyBatisPlus分组
- 使用 `groupBy()` 方法并在参数中使用字符串表示用于分组的字段
- 如下, 是统计分组后按组计数的配置
```java
@Test
void testGetAll09() {
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.select("count(*) as count, tel");
wrapper.groupBy("tel");
List<Map<String, Object>> maps = userDao.selectMaps(wrapper);
System.out.println(maps);
}
```

# 查询应用
- 条件合集 : https://baomidou.com/guides/wrapper/

### 字段等匹配
- 使用 `eq(字段名, 字段值)` 来实现等价判断
- 如果查询只有一条数据, 那么没有必要使用 `selectList` , 直接使用 `selectOne` 查询单独一条即可
```java
@Test
void testLogin() {
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(User::getName, "majinliang").eq(User::getPassword, "mjl111");
    User user = userDao.selectOne(wrapper);
}
```

### 范围查询
- `lt` , `gt` => 小于, 大于
- `le` , `ge` => 小于等于, 大于等于
- `eq` , `ne` => 等于, 不等于
- `between` => 范围查询
	- `val1` , `val2` : 左为小, 右为大
```java
@Test
void testRange() {  
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();  
    wrapper.between(User::getAge, 15, 20);  
    List<User> users = userDao.selectList(wrapper);  
}
```

### 模糊匹配(非全文检索)

- 使用 `like` 方法来设定模糊匹配的条件
```java
@Test  
void testLike() {  
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();  
    wrapper.like(User::getName, "J");  
    List<User> users = userDao.selectList(wrapper);  
}
```
- 同时, 可以使用 `likeLeft` 和 `likeRight` 来设定模糊匹配的位置
- `likeLeft` : 字符串位置在最右侧, 查看左边部分是否符合like
- `likeRight` : 字符串位置在最左侧, 查看右边部分是否符合like
```java
@Test  
void testLike02() {  
    LambdaQueryWrapper<User> wrapper01 = new LambdaQueryWrapper<>();  
    LambdaQueryWrapper<User> wrapper02 = new LambdaQueryWrapper<>();  
    wrapper01.likeLeft(User::getName, "J");  
    wrapper01.likeRight(User::getName, "J");  
    List<User> users01 = userDao.selectList(wrapper01);  
    List<User> users02 = userDao.selectList(wrapper02);  
}
```

# 字段映射与表名映射

- 默认情况下, 实体类名称与表名一致, 且实体类中字段与表中字段名一致, 所以可以进行自动关联
- 当表字段与编码属性设置不同步时, 后端应进行修改

### 字段映射

##### 字段名称不一致
- 使用 `@TableField` 注解进行注释, 将其置于模型类属性定义上方
- 设置后, 可以用于定义当前字段与数据库表中的字段关系
- 属性为value, 用于设置映射到表中哪个属性, 如 `@TableField(value = "password")`
```java
@Data  
public class User {  
    ...
    @TableField(value = "password")  
    private String pwd;  
}
```

##### 模型存在额外字段
- 在某些情况下, 模型类中可能存在额外字段用以表示当前用户的状态等, 虽不作为永久数据保存, 但是应在类中声明
	- 此时, 该额外字段与表中属性存在不对应的关系. 
- 应该使用 `@TableField(exist = false)`, 表示该字段在表中不存在, 使SQL忽略该字段
- 注意, 在`@TableField`注解中, `value`属性和`exist`属性不可同时出现
```java
@Data  
public class User {  
    ...
    @TableField(exist = false)  
    private boolean isOnline;  
}
```

##### 设置参与查询
- 对于某些比较私密的表属性, 通常情况下不建议返回给后台相应结果, 为此, 可以指定对其不进行查询
- 在模型类中, 使用 `@TableField(select = false)` , 来设置该字段是否参与查询, 该参数可以和value同时存在
- 要注意的是, 即使配置了该字段, 如果在 `wrapper` 中使用`select`函数进行设置, 则select函数优先级更高
```java
@Data  
public class User {  
    ...
    @TableField(value = "password", select = false)  
    private String pwd;  
}
```

### 表名映射
- 如果模型类名和表名不一致, 则可以通过字段 `@TableName` 使表名和模型类名之间建立映射关系
```java
@Data  
@TableName("tbl_user")  
public class User {  
    ... 
}
```

# 隐藏日志

**隐藏info和debug**
- 在配置文件中新建一个名为`logback.xml`的xml文件, 并进行基本配置如下, 表示不进行日志输出
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<configuration></configuration>
```

**隐藏banner**
- SpringBoot和MyBatisPlus的banner都可以在yml中控制隐藏或显示, 具体配置如下
```yaml
spring:
  main:
    banner-mode: off
mybatis-plus:
  global-config:
  	banner: false
```

