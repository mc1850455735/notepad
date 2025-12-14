# redo log
- redo log 是 InnoDB 独属的日志, 使得 MySQL 拥有了崩溃恢复的能力, 当 MySQL 因意外宕机等, 通过 redo log 日志可以恢复数据, 保证数据的持久性与完整性

### 刷盘时机
- MySQL 中, 数据在磁盘中的存取是以页为单位, 会从硬盘中加载一页的数据, 并将其放入 Buffer Pool 中, 后续如果再有数据的查询请求, 会优先从 Buffer Pool 中查询数据, 数据不存在再去硬盘中加载, 从而减少硬盘 IO 开销; 更新时, 同样先在 Buffer Pool 中进行数据更新, 并将修改相关数据存储到 redo log buffer 中, 接着刷盘到 redo log 中. 
- 理想情况下, 事务提交后就会立即将修改进行刷盘操作, 但是实际上是根据刷盘策略进行的. 在 InnoDB 中, 将 redo log buffer 中的数据刷盘到 redo log 中共有6种不同的时机.
1. 事务提交时: 当事务提交时, buffer 中的数据会被刷新到磁盘中. 该刷盘时机可以通过 `innodb_flush_log_at_trx` 参数控制.
2. redo log 空间不足时. InnoDB 主动进行空间管理, 避免因缓存区写满导致用户线程阻塞. 当 redo log buffer 已用空间超过总量50%时, 后台线程主动将缓存刷新到磁盘, 为后续日志写入留足空间; 当 redo log buffer 因达大事务或 IO 繁忙导致被完全写满时, 则所有试图写入新日志的用户都会被阻塞, 并强制进行一次同步刷盘, 直到缓存有可用空间为止.
3. 触发检查点(Checkpoint): Checkpoint 是 InnoDB 为缩短崩溃恢复时间设计的核心机制, Checkpoint 触发时, 必须将 Checkpoint 检查点之前的所有脏页都写入磁盘中. 而根据 WAL (Write-Ahead Logging) 原则, 数据页写入磁盘前必须先将对应的 redo log 写入磁盘.
4. 后台线程周期性刷新: InnoDB 存在一个后台线程 master thread, 其会以大约1秒的周期执行例行任务, 其中就包括将 buffer 中的记录刷盘到磁盘中. 这个机制是 `innodb_flush_log_at_trx` 设为0或2时的主要持久化保障.
5. 正常关闭服务器: 当MySQL服务器被正常关闭时, 为确保所有已提交的日志都被保存, InnoDB 会进行一次最终的刷盘操作, 确保 buffer 中的日志全部都情况并存储到了磁盘文件中.
6. binlog切换时: 开启 binlog 后, 在 MySQL 使用 `innodb_flush_log_at_trx=1` 和 `sync_binlog=1` 的双配置下, 为保证 redo log 与 binlog 之间的一致性, 在 binlog 写满或手动执行 flush logs 进行切换时, 会触发 redo log 刷盘操作.

### 刷盘策略
- 刷盘策略通过 `innodb_flush_log_at_trx` 进行刷盘策略的设置, `MySQL` 刷盘策略的不同导致了 `MySQL` 宕机后可能存在轻微的数据丢失问题.
- `0` : 设为0时, 表示每次事务提交后不进行刷盘操作. 该方法性能最高, 但是也最不安全, 因为如果 `MySQL` 挂或服务器宕机, 可能丢失最近1s数据.
- `1` : 设为1时, 表示事务每次提交都会进行刷盘操作. 该方法性能最低, 但是最安全, 只要事务成功提交, redo log 就一定存储在磁盘中, 不会有任何数据丢失.
- `2` : 设为2时, 表示每次提交都只把 `buffer` 中的内容写入 `page cache` 中. `page cache` 属于文件系统缓存, 专门用于缓存文件, 在这里被缓存的文件就是 `redo log` 文件. 如果 `MySQL` 挂了则不会出现数据丢失, 服务器若宕机则可能出现1s的数据丢失. 其性能与安全性介于 `0` 到 `1` 之间.

### 日志文件组
- 在 InnoDB 中, redo log 日志文件不由一个文件组成, 而是以一组文件的形式出现的, 每个 redo log 文件的大小都是相同的, 可以配置每个文件的大小以控制 redo log 日志整体大小. 文件之间头尾相连, 组成环形数组的形式.
- 在日志文件组中, 存在两个属性, 分别是 write pos 和 checkpoint. write pos 负责存储当前记录的位置, checkpoint 负责记录已写入的位置. 当 buffer 记录对应 redo log 到磁盘中时, write pos 后移; 当服务器空闲时将数据从 redo log 中写入数据库, 此时 checkpoint 后移.
- MySQL 8.0.30 开始, `innodb_log_files_in_group` 和 `innodb_log_file_size` 两个原本用来配置 redo log 文件数量和大小的配置已被废弃. 日志文件组的文件数被固定为 32, 而文件大小则由 `innodb_redo_log_capacity` 确定, 每个文件的大小为 `innodb_redo_log_capacity/32`

# binlog

### 概述
- redo log 是物理日志, 记录数据页中进行了什么修改, 属于 InnoDB 存储引擎; 
- binlog 是逻辑日志, 记录内容是语句的原始逻辑, 属于 MySQL Server 层, 不论使用什么存储引擎, 只要产生了数据表的更新, 都会产生 binlog 日志. MySQL 数据库的 数据备份、主备、主主、主从 都需要依赖 binlog 同步数据, 保证数据的一致性.
- binlog 会记录所有涉及更新数据的逻辑操作, 并且顺序写.

### 格式
- `binlog` 共有三种格式, 分别是 `statement`, `row`, `mixed`.
- 当指定 `statement` 时, `binlog` 会记录 `SQL` 语句的原文. 但可能存在数据不一致的情况, 如使用函数进行操作时, 只会记录函数名而无法获取函数得到的值, 从而导致可能的数据不一致问题 (如 `now()` 函数)
- 当指定 `row` 时, `binlog` 会记录操作的具体数据而不是简单的 `SQL` 语句, 其会将数据字段的原始数据加入记录中, 同时也会获取函数的执行结果并存入日志. 通常情况下都指定为 `row` 格式, 可以带来良好的数据库恢复与同步的可靠性, 但 `row` 格式比较占用空间, 需要更大容量用来记录, 同时在恢复与同步时需要更多 `IO` 资源, 影响执行速度.
- `mixed` 方法是前两者的混合, 使用 `mixed` 方法时, `MySQL` 会先判断这条 `SQL` 语句是否有可能引起数据不一致, 如果会, 使用 `row`, 否则使用 `statement`

### 写入机制
- 在事务执行阶段, 先把 `binlog` 写到 `binlog cache` 中, 待事务提交时, 再把 `binlog cache` 写入到 `binlog` 文件中. 事务的 `binlog` 无法拆开, 必须确保一次性写入 `binlog cache` 中, 如果无法一次性写入, 就必须**暂存**到磁盘中. 为确保每个事务线程都有缓存可用, 系统会为每一个线程都分配 `binlog cache`.
- `binlog` 的写入分为两个阶段, `write` 会将日志写入文件系统的 `page cache`, 不将数据存储到磁盘, 速度相对较快; `fsync` 将数据持久化到磁盘.
- `write` 和 `fsync` 的时机可以由参数 `sync_binlog` 控制, 默认是 `1` .
- `sync_binlog` 为0时, 表示每次提交事务时只做 `write`, 由系统自动判断什么时候执行 `fsync` 操作. 该方法性能高, 但如遇机器宕机可能导致 page cache 中的 binlog 丢失.
- `sync_binlog` 为1时, 表示每次提交事务时同时进行 write 和 fsync. 类似 redo log 日志刷盘流程. 相对性能较差, 但可靠性较好.
- `sync_binlog` 为N(N >= 2)时, 每次事务提交时进行 write, 当累积 N 个事务提交后, 进行 fsync, 是一种折中的方案. 在出现 IO 瓶颈场景中, 将 N 设为一个较大值可以提升性能.

# 两阶段提交
- `redo log` 让 `InnoDB` 存储引擎有了崩溃恢复的能力. `binlog` 保证了 `MySQL` 集群架构的数据一致性.
- 在执行更新语句的过程中, 会记录 `redo log` 与 `binlog` 两个日志. `redo log` 会在事务执行过程中不断写入, 而 `binlog` 只有在事务提交时才写入, 二者写入时机不同.
- 主数据库使用 `redo log` 恢复数据, 而备份数据库或从数据库通过 binlog 同步数据. 如果 `redo log` 提交成功, 但在写入 binlog 时出现异常, 使 binlog 中不包含写入的信息, 就会导致主从数据一致性出现异常.
- 为解决两日志之间的逻辑一致性问题, InnoDB 引擎使用**两阶段提交方案**解决. 将 `redo log` 拆分为 `prepare` 和 `commit` 两个阶段. `InnoDB` 记录日志时, 先记录 `redo log` 并设置其为 `prepare` 阶段, 再将 `binlog` 写入, 最后将 `redo log` 的状态改为 `commit`. 当 `MySQL` 根据 `redo log` 恢复数据时, 发现 `redo log` 还处于 `prepare` 阶段且不存在对应 `binlog`, 则直接回滚该事务. 即使 `redo log` 并未处于 `commit` 状态, 只要对应 `binlog` 可以被找到, 同样认定为事务提交成功.

# undo log
- 每一个事务对数据的修改都会被同步记录到 undo log 中, 当事务执行过程中需要进行回滚操作时, 可以借助 undo log 进行回滚.
- `undo log` 属于逻辑日志, 其中记录与执行的 `SQL` 功能相反的 `SQL` 语句, 并且 `undo log` 的内容同样会被记录到 `redo log` 中, 因为 `undo log` 也需要有持久化的保证, 以便于 `MVCC` 进行版本管理等. `undo log` 本身会被清理, 对于某些操作, 事务提交后立即会被清理, 而 `update` 等操作不会立即删除, 而是加入 `history list`, 等待不再使用后由后台线程进行清理.
- `InnoDB` 使用段来组织 undo log. 事务开始时, 需要先为每个事务分配一个 `Rollback Segment`, 并为该事务分配一个 `undo log segment`. 每个 `undo log segment` 中含有多个 `page`, 每个 `page` 记录一个 `undo log record`. 一个 `Rollback Segment` 中包含1024个 `undo log Segment`, 有助于管理多个并发事务的回滚需求.
- 每个 `Rollback Segment` 中含有一个 `Rollback Segment Header`, 通常位于回滚段的第一页, 负责管理 `Rollback Segment`. `Header` 中含有 `history list`, 负责记录所有已提交但是还未清理的事务的 `undo log`, 该 `list` 可以使得后台线程可以找到并清理不再需要的 `undo log` 记录.


