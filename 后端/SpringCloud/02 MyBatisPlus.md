
注意: 本笔记不同于SSM课程的MyBatisPlus, 有新内容的增加(基本没有旧内容)

# 基础使用

**常见注解**

- MP通过扫描实体类, 并基于反射获取实体类信息作为数据库表信息
- 实体类到数据库表信息转换关系
	- 类名驼峰转下划线作为表名
	- 名为id的字段作为主键
	- 变量名驼峰转下划线作为表的字段名
- MP提供了以下注解用来指定表中信息 (详见SSM课程)
	- `@TableName`: 用来指定表名
	- `@TableId`: 用来指定表中的主键字段信息, 可以通过type设置id增长策略, 默认使用雪花算法
	- `@TableField`: 用来指定表中的普通字段信息
- 对于`is`开头且是布尔值的变量, 需要使用`@TableField`注解进行声明, 否则在编译时, MP会将`is`删除并将大写字母变为小写来寻找对应字段
	- 如: `isMarried` => `married`
- 对于变量名与数据库关键字冲突的 (如`order`), 也要使用`@TableField`
- 对于表中不存在对应字段名的变量, 也需要使用`@TableField`
	- 如: `@TableField(exist = false)`

**配置**
- MP的配置项继承了MyBatis的原生配置和一些自己的特有配置
- MP可以在yml文件中进行一些常用的配置
```yml
mybatis-plus:
	type-aliases-package: com.itheima.mp.domain.po # 别名扫描包
	mapper-location: "classpath*:/mapper/**/*.xml" # Mapper.xml文件地址, 默认值
	configuration: 
		map-underscore-to-camel-case: true # 是否开启下划线和驼峰的映射
		cache-enabled: false # 是否开启二级缓存
	global-config:
		db-config:
			id-type: assign_id # id为雪花算法生成
			update-strategy: not_null # 更新策略: 只更新非空字段
```


其余略

















