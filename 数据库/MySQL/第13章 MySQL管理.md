
# 系统数据库

MySQL安装完成后自带4个数据库, 其具体作用如下
- information_schema
	- 提供了访问数据库元数据的各种表和视图, 包括数据库, 表, 字段类型, 访问权限, 触发器, 存储过程等
- mysql
	- 存储MySQL服务器正常运行所需的各种信息
	- 时区, 主从, 用户, 权限等
- performance_schema
	- 为MySQL服务器**运行时状态**提供了一个底层监控功能, 用于收集数据库服务器性能参数
- sys
	- 包含了一系列方便DBA和开发人员利用performance_schema性能数据库进行性能调优和诊断的视图

# 常用工具

### mysql
- 该mysql指的是mysql的客户端工具

**语法**
```shell
mysql [options] [database]
```

**参数**
- `-u` , `--user=name` : 指定用户名
- `-p` , `--password[=name]` : 指定密码
- `-h` , `--host=name` : 指定服务器IP或者域名
- `-P` , `--port=port` : 指定连接端口
- `-e` , `--execute=name` : 执行SQL语句并退出

**示例**
```shell 
mysql -uroot -p1234 db01 -e "select * from tb_user";
```

### mysqladmin
- 一个执行管理操作的客户端程序, 可以用它来检查服务器的配置和当前状态, 创建或删除数据库等
- 使用mysqladmin时, 不需要使用字符串, 直接输入指令及其对应参数即可

**示例**
```shell
mysqladmin -uroot -p1234 drop 'test01';
mysqladmin -uroot -p1234 version;
```

### mysqlbinlog
- 由于服务器生成的二进制日志文件以二进制形式保存, 所以如果想要检查这些文本的格式, 就需要使用mysqlbinlog日志管理工具

**语法**
```shell
mysqlbinlog [options] log-files1 log-files2 ...
```

**选项**
- `-d` , `--database=name`
	- 指定数据库名称, 只列出指定数据库相关操作
- `-o` , `--offset=#`
	- 忽略日志前n行命令
- `-r` , `--result-file=name`
	- 将输出的文本格式日志输出到指定文件
- `-s` , `--short-form`
	- 显示简单格式, 省略掉一些信息
- `--start-datetime=date1 --stop-datetime=date2`
	- 指定日期间隔内的所有日志
- `--start-position=pos1 --stop-position=pos2`
	- 指定位置间隔内的所有日志

### mysqlshow
- 客户端对象查找工具, 用来快速查找存在哪些数据库, 数据库中的表, 表中的列或者索引

**语法**
```shell
mysqlshow [options][db_name[table_name[colume_name]]]
```

**选项**
- `--count` : 显示数据库以及表的统计信息 (数据库, 表, 字段均可不指定)
- `-i` : 实现数据库或者指定表的状态

**示例** : 展示test库中的book表信息
```shell
mysqlshow -uroot -p1234 test book --count
```

### mysqldump
- 用来备份数据库或在不同的数据库之间进行数据迁移
- 备份内容包括 创建表, 以及插入表的SQL语句

**语法**
```shell
# 保存某个数据库(或者其中的表)
mysqldump [options] db_name [tables] > xxx.sql
# 保存多个数据库的两个选项
mysqldump [options] --database / -B db1 [db2 db3 ...] > xxx.sql
# 保存所有数据库的两个选项
mysqldump [options] --all-databases / -A > xxx.sql
```

**连接选项**
- `-u` , `--user=name` : 指定用户名
- `-p` , `--password[=name]` : 指定密码
- `-h` , `--host=name` : 指定服务器IP或者域名
- `-P` , `--port=port` : 指定连接端口

**输出选项**
- `--add-drop-database` : 在每个数据库创建语句前加上drop database语句
- `--add-drop-table` : 在每个表创建语句前加上drop table语句, 默认开启
	- 关闭 : `--skip-add-drop-table`
- `-n` , `--no-create-db` : 不包含数据库的创建语句
- `-t` , `--no-create-info` : 不包含数据表的创建语句
- `-d` , `--nodata` : 不包含数据
- `-T` , `--tab=name` : 自动生成两个文件: `.sql`创建表结构, `.txt`保存数据
	- `mysqldump -uroot -p1234 -T /var/lib/mysql-files/ db01 score`
	- 使用 `show variables like '%secure_file_priv%';`查看mysql信任的数据存储目录

### mysqlimport/source
- mysqlimport是客户端数据导入工具, 用来导入mysqldump加-T参数后导出的文本文件
- 如果需要导入sql文件, 可以使用mysql中的source指令
- 同样的, 导入时需要从mysql信任的目录出发

**语法**
- mysqlimport
```shell
mysqlimport [options] db_name textfile1 [textfile2...]
# 示例
mysqlimport -uroot -p1234 test /tmp/city.txt
```
- source
```mysql
source /root/xxx.sql
```

