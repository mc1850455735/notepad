## 通用语法

1. SQL语句可以单行或多行书写 , 以分号结尾
2. SQL语句可以使用 空格 / 缩进 来增强语句的可读性
3. MySQL数据库的SQL语句不区分大小写 , 关键字**建议使用大写**
4. 注释
	- 单行注释 : -- 注释内容 或者 # 注释内容(MySQL特有)
	- 多行注释 : \/\* \*\/注释内容

## SQL分类

![[Pasted image 20221217233952.png]]

- `DDL` - 数据定义语言, 用来定义数据库对象
- `DML` - 数据操作语言, 用来对数据库表中的数据进行增删改
- `DQL` - 数据查询语言, 用来查询数据库中表的记录
- `DCL` - 数据控制语言, 用来创建数据库用户、控制数据库的访问权限

## DDL


#### DDL数据库操作

##### 查询
**查询所有数据库**
```MySql 
SHOW DATABASES;
```

**查询当前数据库**
``` MySQl
SELECT DATABASE();
```

##### 创建
```mysql
CREATE DATABASE [IF NOT EXISTS] 数据库名 [DEFAULT CHARSET 字符集] [COLLATE 排序规则];
```

##### 删除
```mysql
DROP DATABASE [IF EXISTS] 数据库名;
```

##### 使用
```mysql
USE 数据库名;
```

#### DDL表操作

##### 查询
**查询当前数据库所有表**
```mysql
SHOW TABLES;
```

**查询表结构**
```mysql
DESC 表名;
```

**查询指定表的建表语句**
```mysql
SHOW CREATE TABLE 表名;
```

##### 创建
```Mysql
CREATE TABLE 表名(
	字段1 字段1类型[COMMENT 字段1解释],
	字段2 字段2类型[COMMENT 字段2解释],
	...
	字段n 字段n类型[COMMENT 字段n解释]
)[COMMENT 表注释]
```

##### 修改
**添加字段**
```Mysql
ALTER TABLE 表名 ADD 字段名 类型(长度) [COMMENT 注释] [约束];
```

**修改数据类型**
```Mysql
ALTER TABLE 表名 MODIFY 字段名 新数据类型(长度);
```

**修改字段名和字段类型**
```MySql
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型(长度) [COMMENT 注释] [约束]
```

**修改表名**
```Mysql
ALTER TABLE 表名 RENAME TO 新表名;
```

##### 删除
**删除字段**
```mysql
ALTER TABLE 表名 DROP 字段名;
```

**删除表**
```mysql
DROP TABLE [IF EXISTS] 表名;
```

**删除指定表并重新创建该表(格式化)**
```mysql
TRUNCATE TABLE 表名;
```

##### 数据类型
- 数值类型
	- `类型 unsigned` - 转换为无符号型
	- `tinyint` - 1字节
	- `smallint` - 2字节
	- `mediumint` - 3字节
	- `int` - 4字节
	- `bigint` - 8字节
	- `float` - 4字节
	- `double` - 8字节 - 两个参数 最长长度 和 小数位数
	- `decimal` - 取决于精度和标度的值
- 字符串类型
	- `char` - 定长字符串(性能高) - 参数 当前能存储的最长长度
	- `varchar` - 变长字符串 - 参数 最大能存储的空间
	- `tinyblob` - 不超过255字符的二进制数据
	- `tinytext` - 短文本字符串
	- `blob` - 二进制形式的长文本数据
	- `text` - 长文本数据
	- `mediumblob` - 二进制形式的中等长度文本数据
	- `mediumtext` - 中等长度文本数据
	- `longblob` - 二进制形式的极大长度文本数据
	- `longtext` - 中等极大文本数据
- 日期类型
	- `date` - 日期值 格式`YYYY-MM-DD`
	- `time` - 时间值 格式`HH:MM:SS`
	- `year` - 年份值 格式`YYYY`
	- `datatime` - 混合日期和时间值
	- `timestamp` - 混合日期和时间值, 时间戳


## DML

数据操作语言, 用来进行增删改表中数据

#### 添加数据
**给指定字段添加数据**
```mysql
INSERT INTO 表名 (字段名1, 字段名2 ...) VALUES (值1, 值2);
```

**给全部字段添加数据**
```mysql
# 添加数据
INSERT INTO 表名 VALUES (值1, 值2)
```

**指定字段批量添加数据**
```mysql
INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);
````

**批量添加数据**
```mysql
INSERT INTO 表名 VALUES (值1, 值2, ...), (值1, 值2, ...);
```

*注意*
1. 插入数据时, 指定字段书写需要与值的顺序是一一对应的
2. 字符串和日期型数据应该包括在引号中
3. 插入的数据大小 , 应该在字段的规定范围内

#### 修改数据
```mysql
UPDATE 表名 SET 字段名1 = 值1, 字段名2 = 值2 , ... [WHERE 条件];
```

#### 删除数据
```mysql
DELETE FROM 表名 [WHERE 条件];
```

## DQL
Data Query Language

查询关键字 : `SELECT`

![[Pasted image 20221218152547.png]]

- 基本查询
- 条件查询(WHERE)
- 聚合函数(count, max, min, avg, sum)
- 分组查询(GROUP BY)
- 排序查询(ORDER BY)
- 分页查询(LIMIT)

#### 基本查询

1. 查询多个字段
```Mysql
SELECT 字段1, 字段2, ... FROM 表名;
```
```mysql
# 全部查询(实际开发尽量不写*)
SELECT * FROM 表名;
```

2. 设置别名 - 非必须, AS可以省略
```mysql
SELECT 字段1 [AS 别名1], 字段2 [AS 别名2] ... FROM 表名;
```

3. 去除重复记录
```mysql
SELECT DISTINCT 字段列表 FROM 表名;
```

#### 条件查询

语法
```mysql
SELECT 字段列表 FROM 表名 WHERE 条件列表;
```

**比较运算符**
![[Pasted image 20221218132251.png]]

**关于`between and`**
```mysql
SELECT * FROM emp WHERE age BETWEEN 15 AND 20;
```
`between`后跟的是最小值, `and`后跟的是最大值
相反的话就会什么都查找不出来

**`IN`的使用**
```mysql
SELECT * FROM emp WHERE age IN(18,20,40);
```

**`LIKE`模糊匹配**
`%代表字数和内容都模糊的匹配`
`_代表内容模糊，字数对应的匹配`
```mysql
SELECT * FROM emp WHERE name LIKE '__';
SELECT * FROM emp WHERE name LIKE '马%';
```

**条件运算符**
![[Pasted image 20221218132403.png]]

#### 聚合函数
将**一列**数据作为一个整体 , 进行纵向计算

**常见聚合函数**
![[Pasted image 20221218140825.png]]

**语法**
```mysql
SELECT 聚合函数(字段列表) FROM 表名;
```

**null值不参与**聚合函数计算


#### 分组查询
```mysql
SELECT 字段列表 FROM 表名 [WHERE 条件] GROUP BY 分组字段名 [HAVING 分组后过滤条件]
```

###### `where` 与 `having` 的区别
- 执行时机不同
	`where`是**分组之前**进行过滤, 不满足`where`条件, 不参与分组, 而`having`是对**分组之后**结果进行过滤
- 判断条件不同
	`where`**不能对聚合函数**进行判断, 而`having`可以

**注意**
- 执行顺序
	`where` > 聚合函数 > `having`
- 分组之后, **查询的字段一般为聚合函数和分组字段**, 查询其他字段并无意义


#### 排序查询
order by

SELECT 字段列表 FROM 表名 ORDER BY 字段1 排序方式1, 字段2 排序方式2;

- asc - 升序排序(默认)
- desc - 降序

#### 分页查询

SELECT 字段列表 FROM 表名 LIMIT 起始索引, 查询记录数;

注意
- 起始索引从0开始, 起始索引 = (查询页码 - 1) * 每页显示记录数
- 分页查询是**数据库的方言**, 不同数据库有不同的实现, mysql中是LIMIT
- 如果查询的是第一页的数据, 起始索引可以省略, 直接简写为limit 10

#### 执行顺序

![[Pasted image 20221218151523.png]]
可以使用 字段名 \[AS\] 别名 来设置别名


## DCL
Data Control Language
用来管理数据库用户, 控制数据库的访问权限

#### 管理用户

- 查询用户
	- USE mysql;
	- SELECT * FROM user;
- 创建用户
	- CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
- 修改用户密码
	- ALTER USER '用户名'@'主机名' IDENTIFIED WITH mysql_native_password BY '新密码';
- 删除用户
	- DROP USER '用户名'@'主机名';

注意
- 主机名可以通过%通配
- 这类SQL开发人员操作较少, 主要是DBA(Database Administrator)使用

#### 权限控制

![[Pasted image 20221218154315.png]]

- 查询权限
	- SHOW GRANTS FOR '用户名'@'主机名';
- 授予权限
	- GRANT 权限列表 ON 数据库名.表名 TO '用户名'@'主机名';
- 撤销权限
	- REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@'主机名';

多个权限之间使用逗号分隔
授权时, **数据库名**和**表名**可以使用 * 进行通配, 代表所有.


