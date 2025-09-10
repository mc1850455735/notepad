# 逻辑存储结构

- 表空间 : 在InnoDB中, 每个表空间对应一个ibd文件 , 一个MySQL实例可以对应多个表空间, 用于存储**记录**, **索引**等数据.
- 段, 分为数据段(Leaf node segment) , 索引段(Non-leaf node segment) , 回滚段(Rollback Segement) , InnoDB是索引组织表, 数据段就是B+树的叶子节点, 索引段为B+树的非叶子节点. 段中管理了多个区(Extent)
- 区 : 表空间的单元结构, 每个区大小为1M, 默认情况下, InnoDB存储引擎页大小为16KB, 即一个区中一共有64个连续的页. 申请时, 每次通常申请4-5个区, 以保证页的连续性
- 页 : 是InnoDB管理磁盘的最小单位, 每个页默认大小为16KB
- 行 : InnoDB的数据是按行为单位进行存放的
	- Trx_id : 每次对某条记录进行改动时, 都会把对应的事务id赋值给trx_id隐藏列
	- Roll_poiner : 每次对某条记录进行改动时, 都会把旧的版本写入undo日志中, 然后这个隐藏列就相当于是一个指针, 可以通过它找到该条记录修改前的信息
![[数据库/MySQL/Inbox/Pasted image 20250910082805.png]]

# 架构

- MySQL5.5版本开始, MySQL就使用InnoDB作为默认引擎
- InnoDB擅长事务处理, 具有崩溃恢复特征, 在日常开发中使用非常广泛
- InnoDB架构图如下所示, 左侧为内存结构, 右侧为磁盘结构
![[数据库/MySQL/Inbox/Pasted image 20250910085502.png]]

### 内存结构
- `Buffer Pool` : 缓冲池
	- 缓冲池是**主内存**中的一个区域, 里面可以缓存磁盘上经常操作的真实数据, 在执行增删改查操作时, 先操作缓冲池中的数据(如果缓冲池中没有数据, 则从磁盘中加载并缓存) , 然后再以一定频率刷新到磁盘, 从而减少磁盘IO, 加快处理速度
	- 缓冲区以Page为单位, 底层采取链表数据结构管理Page. 根据Page的状态, 将Page分为三种类型
		- free page : 空闲Page, 未被使用
		- clean page : 被使用过Page, 但数据没有被修改过
		- dirty page : 脏Page, 数据被修改过, 与磁盘中数据产生了不一致
- `Change Buffer` : 更改缓冲区(MySQL5.5后引入)
	- 针对于**非唯一二级索引**, 在执行DML语句时, 如果这些数据Page没有在Buffer Pool中 , 不会直接操作磁盘, 而是将相关更改操作保存在Change Buffer中, 在未来数据被读取时, 再将该修改合并到Buffer Pool中, 最后将合并后的数据刷新到磁盘中
	- 由于二级索引通常非唯一且以相对随机的顺序插入二级索引, 且删除和更新可能会影响索引树中不相邻的二级索引页. 如果每一次都操作磁盘, 会造成大量磁盘IO, 故通过Change Buffer在缓冲区内进行合并处理, 减少磁盘IO
- `Log Buffer` :  日志缓冲区
	- 参数 : `innodb_log_buffer_size` : 日志缓冲区大小
	- 参数 : `innodb_flush_log_at_trx_commit` : 日志刷新到磁盘时机
		- 0 : 每秒将日志写入并刷新到磁盘一次
		- 1 (默认) : 日志在每次事务提交时写入并刷新到磁盘
		- 2 : 日志在每次事务提交后写入, 并每秒刷新到磁盘一次
	- 用来保存要写入磁盘的log日志数据 (redo log, undo log) , 默认大小为16MB, 日志缓冲区中的日志会定期刷新到磁盘中. 如果需要更新, 插入或删除许多行的事务, 增加日志缓冲区大小可以减少磁盘IO
- `Adaptive Hash Index` : 自适应哈希
	- 参数 : `adaptive_hash_index` (默认开启)
	- 用于优化对Buffer Pool数据的查询. InnoDB引擎会监控对表上各个索引页的查询, 如果观察到使用hash索引可以提升速度, 则建立hash索引, 故称为自适应hash索引
	- 自适应hash索引无需人工干预, 系统根据情况自动完成索引构建

### 磁盘结构
- System Tablespace : 系统表空间
	- 参数 : `innodb_data_file_path` (系统表空间位置, 该位置位于mysql文件夹下)
	- 更改缓冲区的存储区域 (在MySQL5.x版本中还包含着InnoDB数据字典, undolog等内容). 如果表是在系统表空间而不是每个表文件或通用表空间创建的(独立表空间关闭的情况下), 它也可能包含表及索引数据
- File-Per-Table Tablespaces : 表独立表空间
	- 参数 : `innodb_file_per_table` (每张表是否使用独立表空间, 默认开启)
	- 每个表的文件表空间包含单个InnoDB表的数据和索引, 并存储在文件系统上的单个数据文件中
- General Tablespaces : 通用表空间
	- 需要使用`create tablespace`语法创建通用表空间, 创建表时, 可以指定存储在该表空间
	- 如果不手动进行创建, 不会出现该表空间文件
- Undo Tablespaces : 撤销表空间
	- MySQL实例在初始化时会自动创建两个默认的undo表空间, 初始大小为16M, 用于存储undo log日志
- Temporary Tablespaces : 临时表空间
	- InnoDB使用会话临时表空间和全局临时表空间, 用于存储用户创建的**临时表**等数据
- Doublewrite Buffer Files : 双写缓冲区
	- InnoDB引擎将数据页从Buffer Pool刷新到磁盘前, 先将数据页写入双写缓冲区文件中, 便于系统发生异常时**恢复数据**
- Redo Log : 重做日志
	- 用来实现事务的永久性. 该文件由两部分组成 : 重做日志缓冲区 (redo log buffer) 和重做日志文件 (redo log file) , 前者在内存中, 后者在磁盘中.
	- 当事务提交之后会把所有的修改信息都存储到该日志中, 用于在刷新脏页到磁盘发生错误时, 进行**数据恢复**使用
	- redo log不会永久保存, 每隔一段时间就会进行一次清理, 相关事务提交后, redo log存在的意义就不大了
	- 以循环方式写入重做日志文件, 涉及`ib_logfile0`和`ib_logfile1`

**通用表空间语法**
```mysql
-- 创建
create tablespace xxx
add datafile 'file_name'
engine='engine_name';
-- 使用
create table xxx ...
tablespace ts_name;
```

### 后台线程

- 将innoDB缓冲区内的数据在合适的时机刷新到磁盘文件当中
- 主要分为4类
	- Master Thread : 核心后台线程, 负责调度其他线程, 还负责将缓冲池中的数据异步刷新到磁盘中, 保持数据的一致性, 还包括脏页的刷新, 合并插入缓存, undo页的回收等操作
	- IO Thread : 在InnoDB中采用了大量AIO而非阻塞IO来处理IO请求, 大大提高了数据库的性能, 而IO Thread主要负责这些IO请求的回调
	- Purge Thread : 主要用于回收事务已经提交了的undo log , 在事务提交之后, undo log大概率不再使用, 由Purge Thread进行回收
	- Page Cleaner Thread : 协助Master Thread刷新脏页到磁盘的线程, 可以减轻Master Thread的压力, 减少阻塞
- 使用`show engine innodb status;`即可在innodb状态中查看各个线程信息

**IO线程及其数量**
![[数据库/MySQL/Inbox/Pasted image 20250910102527.png]]

# 事务原理

**事务4大特性**
- 原子性, 一致性, 隔离性, 持久性 (ACID)
- 原子性 : undo log
- 持久性 : redo log
- 一致性 : undo log + redo log
- 隔离性 : MVCC + 锁

### Redo Log
- 重做日志记录的是事务提交时数据页的物理修改, 用来实现事务的**持久性**
- 该日志文件由两部分组成, 重做日志缓冲区 (redo log buffer) 和重做日志文件 (redo log file) , 前者在内存中, 后者在磁盘中. 
- 当事务提交之后会把所有的修改信息都存储到该日志中, 用于在刷新脏页到磁盘发生错误时, 进行**数据恢复**使用
- 这种使用日志控制写的方法被称为 WAL (Write-Ahead Logging), 先写日志在进行磁盘IO, 可以将原本随机的磁盘IO改为顺序的根据日志写磁盘, 不仅实现了事务持久性, 更提高了磁盘IO效率
- 事务成功提交后, Redo Log的记录就不需要了, 所以由后台线程进行清理后再供其他事务使用. Redo Log采用循环使用的方法, 不需要持续创建log文件

### Undo Log
- Undo Log用来解决事务的**原子性**
- 回滚日志, 用来记录数据被修改前的信息, 作用包含两个: 提供回滚 和 MVCC (多版本并发控制)
- undo log不同于redo log记录物理日志, undo log记录逻辑日志.
	- 例如: 当进行delete操作时, undo log中会记录一条与delete相反的insert操作, 反之亦然. 
	- 当进行rollback时, 就可以从undo log的逻辑记录中读取到对应的内容并进行回滚
- Undo log销毁 : undo log在事务执行时产生, 事务提交时, 不会立即删除undo log, 因为这些日志还有可能用于MVCC
- Undo log存储 : undo log采用段的方式进行管理和记录, 存放在 rollback segment 回滚段中, 内部包含1024个undo log segment

# MVCC

### 概念

- **当前读**
	- 读取的是记录的**最新版本**, 读取时要保证其他并发事务不能修改当前记录, 会对读取的记录进行加锁
	- 日常操作, 如 `select ... lock in share mode` (加共享锁的select) , `select ... for undate` (加排他锁的select), `update`, `insert`, `delete` 都是一种当前读
- 快照读
	- 简单的select(不加锁)就是快照读, 其读取的是记录数据的可见版本, 有可能是历史数据, 读取过程中不加锁, 是一种**非阻塞读**
	- `Read Committed` : 每次select都生成一个快照读
	- `Repeatable Read` : 开启事务后第一个select语句才快照读, 后续select读取这个快照
	- `Serializable` : 快照读会退化成为当前读
- **MVCC**
	- 全称 Multi-Version Concurrency Control, 多版本并发控制
	- 指维护一个数据的多个版本, 使得读写操作没有冲突, 快照读为MySQL实现MVCC提供了一个非阻塞读功能
	- MVCC的具体实现还需要依赖数据库中的 三个隐式字段 , undo log日志, readView
	- MVCC和锁共同实现了事务的**隔离性**

### 隐藏字段

- 对于每一张表, 都会对应三个隐式的字段, 分别是 `DB_TRX_ID` , `DB_ROLL_PTR` , `DB_ROW_ID`
- `DB_TRX_ID` : 最近修改事务id, 记录插入这条记录或最后一次修改该记录的事务id
- `DB_ROLL_PTR` : 回滚指针, 指向这条记录的上一个版本, 用于配合undo log, 指向上个版本
- `DB_ROW_ID` : 隐藏主键, 如果表结构没有指定主键, 将会生成该隐藏字段

**查看ibd文件**
- 使用指令 `ibd2sdi xxx.ibd` 即可看到表空间文件的结构, 可以看到隐藏字段的存在

### undo log版本链

**undo log日志**
- 回滚日志, 在insert, update, delete时产生的便于数据回滚的日志
- 当insert时, 产生的undo log日志只在**回滚**时需要, 在事务提交后即可被立即删除
- 而update, delete时, 产生的undo log日志不仅在回滚时需要, 在**快照读**时也需要, 不会被立即删除

**undo log版本链**
- 不同事务或相同事务对同一条记录进行修改, 会导致该记录的undo log生成一条记录版本链表, 链表头部是最新的旧记录, 链表尾部是最早的古记录
![[数据库/MySQL/Inbox/Pasted image 20250910134119.png]]

### readView
- ReadView(读视图)是**快照读** 相关SQL语句执行时MVCC提取数据的依据, 记录并维护系统当前活跃事务(未提交事务)的id
- ReadView中包含四个核心字段
	- m_ids : 当前活跃的事务ID集合
	- min_trx_id : 最小活跃事务id
	- max_trx_id : **预分配**事务id. 当前最大事务id+1 (因为事务id是自增的)
	- creator_trx_id : ReadView创建者的事务id

**生成ReadView**
- 不同隔离级别生成ReadView的时机不同
- `Read Committed` : 在事务中每一次执行快照读时生成ReadView
- `Repeatable Read` : 仅在事务中第一次执行快照读时生成ReadView, 后续复用该ReadView

**版本链事务访问规则**
- trx_id : 代表版本链中该记录当前事务id
1. trx_id == creator_trx_id -> 可以访问该版本
	- 成立, 说明这个数据是当前这个事务更改的
2. trx_id < min_trx_id -> 可以访问该版本
	- 成立, 说明更改该数据的事务已经提交了
3. trx_id > max_trx_id -> 不可以访问该版本
	- 成立, 说明数据的事务在当前事务ReadView创建以后才开启
4. min_trx_id <= trx_id <= max_trx_id -> 如果trx_id不在m_ids中, 则可以访问该版本
	- trx_id不在m_ids中, 说明该数据的事务已经提交

**事务使用数据**
- 当事务试图使用记录时, 使用记录的哪个快照版本由上述事务链访问规则决定
- 沿版本链由新到老根据规则判断, 当版本链中某一记录满足任一允许规则条件, 则使用该记录作为最终使用的快照版本; 若满足拒绝规则条件, 则说明该条记录一定不能作为快照使用
![[数据库/MySQL/Inbox/Pasted image 20250910151542.png]]
