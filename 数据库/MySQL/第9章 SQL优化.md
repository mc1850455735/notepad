
# 插入数据

**批量插入**

- 对于多条数据插入, 建议使用批量插入而不是使用多个insert, 减少对数据库的访问
- 即使是批量插入, 一次插入的数据也不建议超过1000条
- 如果数据量过多, 可以将其分割为多条insert
```mysql
insert into tb_test values(1, 'Tom'),(2, 'Cat'),(3, 'Jerry');
```

**手动提交事务**

- 执行insert前开启事务, 并进行insert操作
- 执行完insert后, 提交事务, 将修改同步到数据库中
```mysql
start transaction;
insert into ...;
commit;
```

**主键顺序插入**
- 建议主键顺序插入, 顺序插入效率高于乱序插入

### 大批量数据插入

- 如果一次性需要插入大批量数据, 使用insert语句插入性能较低, 此时可以使用MySQL提供的load指令进行插入
- 通过load, 可以将本地文件中的数据一次性加载到数据库中
- load所用文件中, 各个字段之间使用逗号分割, 按字段顺序排列, 使用换行区分各个不同的数据, 如 : `1,马金良,123456,majinliang,男`

**步骤**
- 客户端连接时, 加上参数 `--local-infile`
- 设置全局参数local_infile为1, 开启从本地加载文件导入数据的开关
	- `set global local_infile = 1;`
- 执行load指令, 将准备好的数据加载到表结构当中
- load中, 主键顺序插入性能同样高于乱序插入
```mysql
load data local infile '/root/sql1.log'
into table `tb_user` 
fields terminated by ','
lines terminated by '\n';
```

# 主键优化

**数据组织方式**
- 在InnoDB存储引擎中, 表数据都是根据主键**顺序组织存放**的, 这种存储方式的表称为 **索引组织表** (index organized table, IOT)

**页分裂**
- 页可以为空, 也可以填充一半, 也可以填满. 每个页包含了2-N行数据, 如果一行数据过大, 会出现行溢出, 页内数据根据主键排列
- 如果根据主键顺序插入, 则只需要依次在每一页中插入行数据即可
- 如果乱序插入, 则需要插入时, 如果前方已满, 则会将前方数据页中数据拿出50%, 与本条数据一起作为一个新的数据页, 并将该数据页插入原数据页之后. 这个现象就叫 **页分裂**

**页合并**
- 在InnoDB中, 当删除一条记录时, 实际上记录并没有被物理删除, 只是记录被标记为已删除且它的空间变得允许其他记录声明使用
- 当页中删除的记录数量达到 `MERGE_THRESHOLD`(默认为50%) 时, InnoDB会开始寻找最靠近的页并观察是否可以将两个**页合并**以优化空间
- `MERGE_THRESHOLD` : 合并页的阈值, 可以自己设置, 在创建表或者创建索引时指定

**主键设计原则**
- 满足业务要求的情况下, 尽量降低主键的长度
	- 二级索引叶子节点中挂的是主键, 主键越长, 二级索引占空间越多, 且在搜索时将会耗费大量磁盘io
- 插入数据时, 尽量选择**顺序插入**, 选择使用`AUTO_INCREMENT`自增主键
- 尽量不要使用UUID做主键或其他**自然主键**, 如身份证号
- 业务操作时, 尽量避免对主键的修改

# order by优化

- `Using filesort` : 
	- 通过表的索引或者全表扫描, 读取满足条件的数据行, 然后在排序缓冲区sort buffer中完成排序操作.
	- 所有不是通过索引直接返回排序结果的排序都叫做 **FileSort排序**
	- 如果不可避免的出现了filesort , 那么再大数据量排序时, 可以适当增大排序缓冲区大小 `sort_buffer_size`(默认256K), 从而提升排序效率
- `Using index` : 
	- 通过有序索引顺序扫描直接返回有序数据, 这种情况即为 using index
	- 不需要额外排序, 操作效率高, 涉及到排序的字段尽量使用索引
	- 索引排序的前提是返回的数据要使用**覆盖索引**
- 在进行order by操作时, 尽量进行using index排序
- MySQL使用索引排序时, 要求多列排序的方向要完全一致, 否则无法直接从索引中得到结果

**示例** - 拥有 professsion, age, status 联合索引
- 01 : `select id, age, phone from tb_user order by age;`
	- Extra字段中, 提示 Using filesort, 效率相对较低
- 02 : `select id, age, phone from tb_user order by age, phone;`
	- Extra字段中, 提示 Using filesort
- 03 : 建立了 age, phone 的联合索引
	- `select id, age, phone from tb_user order by age, phone;`
	- Extra字段中, 提示 Using index, 说明创建完索引后, 使用排序的方式为索引排序, 效率相对更高
- 04 : 对 age, phone 进行倒序排序
	- `select id, age, phone from tb_user order by age desc, phone desc;`
	- Extra 字段中提示 Backward index scan; Using index , 说明此次排序的结果是通过反向扫描索引进行的
- 05 : 先根据 phone 排序, 再根据 age 排序
	- `select id, age, phone from tb_user order by phone, age;`
	- Extra 字段中提示 Using index; Using filesort
	- 联合索引顺序和要求排序顺序不一致, 所以必须通过filesort进行排序
	- 多字段排序时遵循最左前缀法则
- 06 : 先根据 age 正序, 再根据 phone 倒序
	- `select id, age, phone from tb_user order by age asc, phone desc;`
	- Extra 字段中提示 Using index; Using filesort
	- 若age使用正序排序, phone的顺序就必须也是正序, 所以混合方式排序的情况下只能使用filesort方式排序

**解决**
- 对于经常出现某一行正序而另一行倒序的联合查询, 在建立联合索引时可以设置, 使某一列正序而另一列倒序
- 此时, 索引中正序列的Collation为A, 而降序列的Collation为D
- 示例如下
```mysql
create index idx_user_age_phone_ad on tb_user(age asc, phone desc);
```
- 建立完索引后, 再使用如 `select id, age, phone from tb_user order by age asc, phone desc;` 的语句, 则Extra提示 Using index, 不再使用filesort排序

# group by优化

- 直接使用默认方式进行分组时, Extra提示Using Temporary, 即用到了临时表, 性能相对较低
- 对欲分组字段创建索引, 可以提高分组的效率
- 创建联合索引时, 如果使用联合索引进行分组时符合最左前缀法则, 则可以正常用作分组; 否则, 由于不符合最左前缀法则, 索引在分组中不生效, 仍要使用临时表
- 如果先根据联合索引左侧字段进行筛选, 再根据筛选后的数据进行分组, 则该种查询方法仍然符合最左前缀法则, 分组效率较高

**示例**
```mysql
-- 创建联合索引前, 查看分组操作执行计划
-- Using temporary, 分组效率较低
explain select profession, count(*) from tb_user group by profession;
-- 对字段profession, age, status创建联合索引
create index idx_user_pro_age_sta on tb_user(profession, age, status);
-- 再次执行分组操作, 根据profession字段进行分组
-- 显示Using index, 分组效率较高
explain select profession, count(*) from tb_user group by profession;
-- 根据profession和age进行分组, 符合联合索引的最左前缀法则
explain select profession, age, count(*) from tb_user group by profession, age;
-- 不符合最左前缀法则, 查询效率较低
-- 执行计划Extra字段为Using index; Using temporary
explain select age, count(*) from tb_user group by age;
-- 先通过profession进行where筛选, 再通过group by进行分组
-- 同样符合最左前缀法则, 可以使用索引分组, 效率高
explain select age, count(*) from tb_user where profession = '软件工程' group by age;
```

# limit优化

- 对于MySQL来说, 不加索引的情况下, 越是进行靠后的分页, 所需要花费的时间就越长
- 这是由于MySQL在不加索引的情况下使用的是顺序扫描的方式, 如果想要的分页结果相对靠后, MySQL必须依次扫描直到最后, 导致越靠后的记录分页的代价越大
- 官方对此给出的优化方案为覆盖索引+子查询的优化方式

**示例**
```mysql
select s.* 
from 
    tb_sku s, 
    (select id from tb_sku order by id limit 9000000, 10) a
where 
    s.id = a.id;
```

# count优化

- 对于数据量比较大时, count的处理速度相对较慢, 这是由InnoDB的处理方式决定的
- 在MyISAM存储引擎中, 会把一个表的总行数存储在磁盘中, 执行`count(*)`时, 会直接返回磁盘中的数, 效率较高
- 在InnoDB存储引擎中, 执行`count(*)`时, 需要将数据从磁盘中一行行读出计数, 效率低
	- 优化思路 : 在InnoDB引擎中, 自己计数, 自行维护一个key:value记录数据库中条目的数量

**count的几种用法**
- count() 是一个聚合函数, 对于返回的结果集, 一行行地判断, 如果count函数的参数**不是NULL**, 累计值就加1, 否则不加, 最后返回累计值
- 用法 : 
- `count(*)` : 读取表中总记录数
- `count(主键)` : 
	- InnoDB遍历整张表, 把每一行的主键ID值都取出来, 返回给服务层
	- 服务层拿到主键后, 直接按行进行累加(主键不可能为NULL)
- `count(字段)` 
	- 没有not null约束 : InnoDB会遍历整张表, 取出每一行的对应字段值, 返回给服务层, 服务层判断不为null, 则计数增加
	- 有not null约束 : InnoDB引擎会遍历整张表, 去除每一行的字段值, 返回给服务层, 直接按行进行累加
- `count(1)` 
	- InnoDB引擎遍历整张表, 但不取值. 服务层对于返回的每一个行, 放一个数字1进去, 直接按行进行累加 (数字的大小对于累加的过程没有影响)
- `count(*)` 
	- InnoDB引擎并不会把全部字段取出来, 而是专门进行了优化, 不取值, 服务层直接按行进行累加

**效率排序**
- `count(*) ≈ count(1) > count(主键) > count(字段)`
- 所以尽量使用`count(*)`

# update优化

- 对于有索引的列, 在根据索引对某行中的数据进行修改时, 在该事务提交前, 会对该行上**行锁** , 此时试图修改该行将会被阻塞, 但是表中其他行可以被正常修改
- 对于无索引的列, 根据该列中数据对某行内数据进行修改时, 在该事务提交前, 对该表上**表锁** , 此时, 任何试图修改该表的尝试都将被阻塞
- InnoDB的行锁是针对索引加的锁, 并且该索引不能失效, 否则会从行锁升级成为表锁
- 通过索引, 可以尽量避免在update时, 对数据上表锁, 从而提高了并发性能, 所以在更新时, 尽量根据主键/索引字段进行数据更新

