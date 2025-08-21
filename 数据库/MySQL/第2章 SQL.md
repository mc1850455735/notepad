# 通用语法

1. SQL语句可以单行或多行书写 , 以分号结尾
2. SQL语句可以使用 空格 / 缩进 来增强语句的**可读性**, 空格/缩进个数不作限制
3. MySQL数据库的SQL语句不区分大小写 , 关键字**建议使用大写**
4. 注释
	- 单行注释 : `--` 注释内容 或者 `#` 注释内容 (MySQL特有)
	- 多行注释 : `/* 注释内容 */`

# SQL分类

![[Pasted image 20221217233952.png]]
- `DDL` (Data Definition Language) - 数据定义语言, 用来定义数据库对象
- `DML` (Data Manipulation Language) - 数据操作语言, 用来对数据库表中的数据进行增删改
- `DQL` (Data Query Language) - 数据查询语言, 用来查询数据库中表的记录
- `DCL` (Data Control Language) - 数据控制语言, 用来创建数据库用户、控制数据库的访问权限

# DDL 数据定义

### 数据库操作

##### 查询

**查询所有数据库**
```MySql 
SHOW DATABASES;
```

**查询当前数据库**
- `DATABASE()` : 一个可以显示当前所用数据库名称的函数
``` MySQl
SELECT DATABASE();
```

##### 创建
- `IF NOT EXISTS` : 不存在则创建, 否则不执行任何操作
- `DEFAULT CHARSET` : 指定当前数据库所用字符集
	- 对于数据库字符集, 不推荐设置为utf8, 而是设置为utf8mb4
	- 因为部分字符可能占4字节, utf8默认占3字节
- `COLLATE` : 排序规则
- 使用 `DATABASE` 关键字或使用 `SCHEMA` 关键字效果相同
```mysql
CREATE DATABASE [IF NOT EXISTS] 数据库名 [DEFAULT CHARSET 字符集] [COLLATE 排序规则];
```

##### 删除
```mysql
DROP DATABASE [IF EXISTS] 数据库名;
```

##### 使用
- 切换到该数据库
```mysql
USE 数据库名;
```

### 表操作

##### 查询

**查询当前数据库所有表**
```mysql
SHOW TABLES;
```

**查询对应表的表结构**
```mysql
DESC 表名;
```

**查询指定表的建表语句**
- 相比 `desc` 更详细
```mysql
SHOW CREATE TABLE 表名;
```

##### 创建
- 注意, 最后一个字段末尾没有逗号
- 表结构创建语句的末尾需要有一个分号
- `[...]` 内为可选参数
```Mysql
CREATE TABLE 表名(
	字段1 字段1类型[COMMENT 字段1解释],
	字段2 字段2类型[COMMENT 字段2解释],
	...
	字段n 字段n类型[COMMENT 字段n解释]
)[COMMENT 表注释];
```

**示例**
```mysql
CREATE TABLE emp(
    id int comment '编号',
    workno varchar(10) comment '工号',
    name varchar(10) comment '姓名',
    gender char(1) comment '性别',
    age tinyint unsigned comment '年龄',
    idcard char(18) comment '身份证号',
    entrydate date comment '入职时间'
) comment '员工表';
```

##### 修改

**添加字段**
```Mysql
ALTER TABLE 表名 ADD 字段名 类型(长度) [COMMENT 注释] [约束];
```
- 示例 - 为emp表添加字段nickname, 类型为varchar
- `alter table emp add nickname varchar(20) comment '昵称';`

**修改数据类型**
```Mysql
ALTER TABLE 表名 MODIFY 字段名 新数据类型(长度);
```
- 示例 - 将emp表中nickname类型修改为varchar(30)
- `alter table emp modify nickname varchar(30);`

**修改字段名和字段类型**
- 注意 : `类型(长度)` 不能忽略
```MySql
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型(长度) [COMMENT 注释] [约束]
```
- 示例 - 将emp表中nickname字段名改为username, 类型为varchar(20)
- `alter table emp change nickname username varchar(20);`

**修改表名**
```Mysql
ALTER TABLE 表名 RENAME TO 新表名;
```
- 示例 - 将emp表名修改为tbl_emp
- `alter table emp rename to tbl_emp;`

##### 删除

**删除字段**
```mysql
ALTER TABLE 表名 DROP 字段名;
```
- 示例 - 将emp表中username字段删除
- `alter table emp drop username;`

**删除表**
- 删除表时, 表中所有数据也都会被删除
```mysql
DROP TABLE [IF EXISTS] 表名;
```
- 示例 - 删除表tb_user
- `drop table if exists tb_user;`

**删除指定表并重新创建该表(格式化)**
```mysql
TRUNCATE TABLE 表名;
```
- 示例 - 格式化emp
- `truncate table emp;`

##### 数据类型
- 数值类型
	- `类型 unsigned` - 转换为无符号型
	- `tinyint` - 1字节 => 对应java的byte
	- `smallint` - 2字节 => 对应java的short
	- `mediumint` - 3字节
	- `int/integer` - 4字节 => 对应java的int
	- `bigint` - 8字节 => 对应java的long
	- `float` - 4字节
	- `double` - 8字节 - 两个参数 最长长度 和 小数位数
		- `double(4, 1)` => 长度为4, 小数部分占1位
	- `decimal` - 取决于精度M和标度D的值
- 字符串类型
- 二进制文件实际使用时通常使用专门的文件服务器进行存储
	- `char` - 定长字符串 - 参数 当前能存储的最长长度 (0~255)
	- `varchar` - 变长字符串 - 参数 最大能存储的空间 (0~65535)
		- char不管使用与否, 都会占用对应的空间, 尽量用来管理长度相对固定的字段
		- varchar会根据存储内容更改对应占用的空间
		- 由于char无需管理, 故**性能较高**
	- `tinyblob` - 不超过255字符的二进制数据 (0~255)
	- `tinytext` - 短文本字符串 (0~255)
	- `blob` - 二进制形式的长文本数据 (0~65535)
	- `text` - 长文本数据 (0~65535)
	- `mediumblob` - 二进制形式的中等长度文本数据 (0~16777215)
	- `mediumtext` - 中等长度文本数据 (0~16777215)
	- `longblob` - 二进制形式的极大长度文本数据 (0~4294967295)
	- `longtext` - 极大文本数据 (0~4294967295)
- 日期类型
	- `date` - 日期值 格式`YYYY-MM-DD`
	- `time` - 时间值 格式`HH:MM:SS`
	- `year` - 年份值 格式`YYYY`
	- `datatime` - 混合日期和时间值 格式`YYYY-MM-DD HH-mm-SS`
	- `timestamp` - 混合日期和时间值, 时间戳 格式`YYYY-MM-DD HH-mm-SS`


# DML 数据操作

数据操作语言, 用来进行增删改表中数据

### 添加数据

**注意**
1. 插入数据时, 指定字段书写需要与值的顺序是一一对应的
2. 字符串和**日期型数据**应该包括在引号中
3. 插入的数据大小 , 应该在字段的规定范围内

**给指定字段添加数据**
```mysql
INSERT INTO 表名 (字段名1, 字段名2) VALUES (值1, 值2);
```
- 示例
```mysql
INSERT INTO emp(id, workno, `name`, gender, age, idcard, entrydate) 
VALUES (1, '1', 'itcast', '男', 10, '101010200101012010', '2020-01-01');
```

**给全部字段添加数据**
- 不指定字段时, 值需要和表中顺序一一对应
```mysql
# 添加数据
INSERT INTO 表名 VALUES (值1, 值2, ...)
```
- 示例
```mysql
INSERT INTO emp
VALUES (2, '2', 'majinliang', '男', 18, '371122200301012121', '2021-08-01');
```

**指定字段批量添加数据**
```mysql
INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);
````

**批量添加数据**
```mysql
INSERT INTO 表名 VALUES (值1, 值2, ...), (值1, 值2, ...);
```
- 示例
```mysql
INSERT INTO emp VALUES 
(3, '3', '韦一敏', '男', 18, '371122200301012121', '2021-08-01'),
(4, '4', '曾小贤', '女', 18, '379900200401012121', '2021-05-01');
```

### 修改数据
- 修改条件可以有, 也可以没有
- 如果没有where条件, 那么修改的就是表中所有数据
```mysql
UPDATE 表名 SET 字段名1 = 值1, 字段名2 = 值2 , ... [WHERE 条件];
```
- 示例01 - 修改id为1的数据name为itheima
	- `UPDATE emp SET name = 'itheima' WHERE id = 1;`
- 示例02 - 修改id为1的数据, name为小明, gender改为女
	- `UPDATE emp SET `name` = '小明', gender = '女' WHERE id = 1;`
- 示例03 - 将所有员工入职日期改为2008年1月1号
	- `UPDATE emp SET entrydate = '2008-01-01';`

### 删除数据
- 如果不添加删除条件, 则表示删除表内所有数据
```mysql
DELETE FROM 表名 [WHERE 条件];
```
- 示例01 - 删除gender为女的员工 (独立女性可以删gender为男数据)
  - `DELETE FROM emp WHERE gender = '女';`
- 示例02 - 删除所有员工(不添加任何条件)
	- `DELETE FROM emp;`


# DQL 数据查询

- Data Query Language
- 查询关键字 : `SELECT`
- 通常来说, 查询频次远高于增删改

**查询分类**
- 基本查询
- 条件查询 (WHERE)
- 聚合函数 (count, max, min, avg, sum)
- 分组查询 (GROUP BY)
- 排序查询 (ORDER BY)
- 分页查询 (LIMIT)

![[Pasted image 20221218152547.png]]

### 基本查询

1. 查询多个字段
```Mysql
SELECT 字段1, 字段2, ... FROM 表名;
```
- 示例 : 查询表中 name, workno, age 三个字段
- `SELECT name, workno, age FROM emp;`

```mysql
# 全部查询(实际开发尽量不写*)
SELECT * FROM 表名;
```
- 查询所有字段并返回, 可以选择写所有字段或者写通配符 `*`
	- 实际工作中尽量少些 `*` , 多使用全部字段形式
- `SELECT id, workno, name, gender, age, idcard, workaddress, entrydate FROM emp;`
- `SELECT * FROM emp;`

2. 设置别名 - AS非必须, 可以省略
```mysql
SELECT 字段1 [AS 别名1], 字段2 [AS 别名2] ... FROM 表名;
```
- 示例 : 查询所有员工工作地址, 并起别名为 `工作地址`
- `SELECT workaddress AS '工作地址' FROM emp;`
- `SELECT workaddress '工作地址' FROM emp;`

3. 去除重复记录
```mysql
SELECT DISTINCT 字段列表 FROM 表名;
```
- 示例 : 查询当前企业员工的上班地址, 不要重复
- `SELECT DISTINCT workaddress FROM emp;`

### 条件查询

语法
```mysql
SELECT 字段列表 FROM 表名 WHERE 条件列表;
```

##### 比较运算符

![[Pasted image 20221218132251.png]]
- 示例01 : 查询年龄等于88的员工
	- `SELECT * FROM emp WHERE age = 88;`
- 示例02 : 查询年龄小于20的员工
	- `SELECT * FROM emp WHERE age < 20;`
- 示例03 : 查询年龄小于等于20的员工
	- `SELECT * FROM emp WHERE age <= 20;`
- 示例04 : 查询没有身份证号的员工
	- `SELECT * FROM emp WHERE idcard IS NULL;`
- 示例05 : 查询有身份证号的员工
	- `SELECT * FROM emp WHERE idcard IS NOT NULL;`
- 示例06 : 查询年龄不等于88岁的员工
	- `SELECT * FROM emp WHERE age != 88`
	- `SELECT * FROM emp WHERE age <> 88`
- 示例08 : 查询性别为女且年龄小于25岁的员工
	- `SELECT * FROM emp WHERE gender = '女' AND age < 25;`

##### between ... and ...
- `between`后跟的是最小值, `and`后跟的是最大值
- 包含最小值和最大值
- 相反的话就会什么都查找不出来
```mysql
SELECT * FROM emp WHERE age BETWEEN 15 AND 20;
```
- 示例07 : 查询年龄在15岁(包含) 到 20岁(包含) 之间的员工信息
	- `SELECT * FROM emp WHERE 15 <= age and age <= 20;`
	- `SELECT * FROM emp WHERE age BETWEEN 15 AND 20;`

##### IN的使用
```mysql
SELECT * FROM emp WHERE age IN(18,20,40);
```
- 查询09 : 查询年龄为18岁或20岁或40岁的员工
	- `SELECT * FROM emp WHERE age = 18 OR age = 20 OR age = 40;`
	- `SELECT * FROM emp WHERE age IN(18, 20, 40);`

##### LIKE模糊匹配
- `%` 代表字数和内容都模糊的匹配
- `_` 代表内容模糊，字数对应的匹配
```mysql
SELECT * FROM emp WHERE name LIKE '__';
SELECT * FROM emp WHERE name LIKE '马%';
```
- 查询10 : 查询姓名为2个字的员工信息
	- `SELECT * FROM emp WHERE name LIKE '__';`
- 查询11 : 查询身份证号最后一位为x的员工信息
	- `SELECT * FROM emp WHERE idcard LIKE '%X';`

##### 条件运算符
![[Pasted image 20221218132403.png]]

### 聚合函数
- 将**一列**数据作为一个整体 , 进行纵向计算
- **null值不参与**聚合函数计算

**常见聚合函数**
- 这些函数都是作用于表中某一列的
![[Pasted image 20221218140825.png]]

**语法**
```mysql
SELECT 聚合函数(字段列表) FROM 表名;
```
- 案例01 : 统计该企业员工数量
	- `SELECT COUNT(*) FROM emp;`
	- `SELECT COUNT(id) FROM emp;`
- 案例02 : 统计该企业员工平均年龄
	- `SELECT AVG(age) FROM emp;`
- 案例03 : 统计该企业员工最大年龄
	- `SELECT MAX(age) FROM emp;`
- 案例04 : 统计该企业员工最小年龄
	- `SELECT MIN(age) FROM emp;`
- 案例05 : 统计西安地区员工年龄之和
	- `SELECT SUM(age) FROM emp WHERE workaddress = '西安';`

### 分组查询

```mysql
SELECT 字段列表 FROM 表名 [WHERE 条件] GROUP BY 分组字段名 [HAVING 分组后过滤条件]
```

**注意**
- 执行顺序 :  `where` > 聚合函数 > `having`
- 分组之后, 查询的字段一般为**聚合函数**和**分组字段**, 查询其他字段并无意义, 通常也无法通过

**`where` 与 `having` 的区别**
- 执行时机不同
	- `where`是**分组之前**进行过滤, 不满足`where`条件, 不参与分组, 而`having`是对**分组之后**结果进行过滤
- 判断条件不同
	- `where`**不能对聚合函数**进行判断, 而`having`可以

**案例**
- 案例01 : 根据性别分组, 统计男性员工与女性员工数量
	- `SELECT gender, COUNT(*) FROM emp GROUP BY gender;`
- 案例02 : 根据性别分组, 统计男性员工与女性员工的平均年龄
	- `SELECT gender, AVG(age) FROM emp GROUP BY gender;`
- 案例03 : 查询年龄小于45的员工, 并根据工作地址分组, 获取员工数量大于等于3的工作地址
	- `SELECT workaddress, COUNT(*) FROM emp WHERE age < 45 GROUP BY workaddress HAVING COUNT(*) > 3;`
	- 使用了**别名**的 : `SELECT workaddress, COUNT(*) AS address_count FROM emp WHERE age < 45 GROUP BY workaddress HAVING address_count > 3;`

### 排序查询

- 关键字 : order by
	- asc - 升序排序(默认)
	- desc - 降序
- sql中, 支持多字段排序, 排序时要同时指定字段名和排序方式
- 先按照第一个字段进行排序, **若相同**, 则按第二个字段排序, 依此类推

```mysql
SELECT 字段列表 FROM 表名 ORDER BY 字段1 排序方式1, 字段2 排序方式2;
```

- 案例01 : 根据年龄对公司员工进行升序排序
	- `SELECT * FROM emp ORDER BY age;`
- 案例02 : 根据入职时间对员工进行降序排序
	- `SELECT * FROM emp ORDER BY entrydate DESC;`
- 案例03 : 根据年龄对员工进行**升序**排序, 在按照入职时间进行**降序**排序
	- `SELECT * FROM emp ORDER BY age, entrydate DESC;`

### 分页查询

- MySQL中的分页, 实际上就是从第x+1条记录开始, 向后显示y条数据
```mysql
SELECT 字段列表 FROM 表名 LIMIT 起始索引x, 查询记录数y;
```

- 案例01 : 查询第1页数据, 每页展示10条数据
	- `SELECT * FROM emp LIMIT 10;`
- 案例02 : 查询第2页数据, 每页展示10条数据
	- `SELECT * FROM emp LIMIT 10, 10;`

**注意**
- 起始索引**从0开始**, 起始索引 = (查询页码 - 1) * 每页显示记录数
- 分页查询是数据库的**方言**, 不同数据库有不同的实现, mysql中是LIMIT
- 如果查询的是第一页的数据, 起始索引可以省略, 直接简写为limit 10

### 执行顺序
- 执行顺序不同于编写时的结构, 指的是运行时的先后顺序
	- `FROM` : 执行查询操作所在的表
	- `WHERE` : 查询操作数据项筛选条件
	- `GROUP BY` : 分组依据
	- `HAVING` : 根据分组内聚合函数值, 决定哪个分组可以被留下
	- `SELECT` : 决定查询使用哪些字段执行select
	- `ORDER BY` : 决定查询后字段列表排序方式
	- `LIMIT` : 决定查询后显示的数据条数
- 可以使用 `字段名 [AS] 别名` 来设置别名
![[Pasted image 20221218151523.png]]


# DCL 数据控制
- Data Control Language
- 用来管理数据库用户, 设置每个数据库可以由哪些用户访问, 控制数据库的访问权限
- 这类SQL**开发人员**操作较少, 主要是DBA(Database Administrator, 数据库管理员)使用

### 管理用户

**查询用户**
```mysql
USE mysql;
SELECT * FROM user;
```

**创建用户**
- 如果想要在任意主机上都能访问数据库, 则将 `主机名` 字段修改为 `%`
```sql
CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
```

**修改用户密码**
- `IDENTIFIED WITH` : 指定加密方式
```sql
ALTER USER '用户名'@'主机名' IDENTIFIED WITH mysql_native_password BY '新密码';
```

**删除用户**
```mysql
DROP USER '用户名'@'主机名';
```


### 权限控制

- 多个权限之间使用**逗号**分隔
- 授权时, **数据库名**和**表名**可以使用 * 进行通配, 代表所有.
![[Pasted image 20221218154315.png]]

**查询权限**
```sql
SHOW GRANTS FOR '用户名'@'主机名';
```

**授予权限**
```sql
GRANT 权限列表 ON 数据库名.表名 TO '用户名'@'主机名';
```

**撤销权限**
```sql
REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@'主机名';
```



