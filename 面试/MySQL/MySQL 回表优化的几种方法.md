
# 重要

## 用户层面

覆盖索引

索引条件设计优化

调整联合索引顺序

减少 SELECT 字段（业务层优化）

## 系统层面

### ICP（Index Condition Pushdown，索引下推）

将部分 WHERE 条件下推到存储引擎层，在二级索引扫描过程中提前过滤数据，减少不必要的回表操作，从而降低 IO 开销

### MRR（Multi-Range Read，多范围读取）

二级索引中存储了索引字段和主键字段。
通过 MRR，将扫描二级索引得到的主键字段，但先不直接返回，而是将主键索引排序，再按照主键顺序回表。

MRR 不减少回表次数。而是将原本随机访问页面的过程改为一定程度上的顺序访问。
通过优化回表顺序，减少了随机 IO，提高了数据库性能。

ICP 发生在**索引扫描阶段**，而 MRR 发生在**回表阶段**。
ICP 的目的是减少回表，而 MRR 的目的是加快回表速度。

### Covering Index Detection（覆盖索引判断）


# 一般

## 用户层面

延迟关联

分区裁剪（Partition Pruning）

使用主键查询代替二级索引查询

小表优化（避免索引回表）

## 系统层面

批量回表（Batch Key Access，BKA）

Index Merge（索引合并）

Semijoin 优化（子查询减少回表）

Adaptive Hash Index（AHI）( InnoDB 独有 )

Buffer Pool 优化（间接降低回表成本）( InnoDB 独有)

Change Buffer（写场景）

