
# select，from，where，group by，having，join，order by，limit 执行顺序是什么？

1. from：确定要执行的表
2. join：执行表连接，合并多表数据
3. where：行级过滤，筛选掉不符合条件的表
4. group by：按指定条件对数据分组
5. having：组级过滤，筛选掉不符合条件的组
6. select：选择要返回的列，计算表达式
7. order by：对列进行排序
8. limit：限制返回行数

 
