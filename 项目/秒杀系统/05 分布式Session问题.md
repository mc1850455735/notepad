
**问题：**
部署多台系统配合**Nginx**时会出现用户登录问题

**原因：**
由于**Nginx**使用**轮询机制**进行负载均衡，请求会按时间顺序注意分发到后端应用。
Tomcat1中的Session和Tomcat2中的Session是不互通的。

**解决：**
- Session复制
	- 优点：
		- 无需修改Session代码，只需修改Tomcat配置。
	- 缺点：
		- Session同步传输占用内网宽带
		- 多台Tomcat同步性能迅速下降
		- Session占用内存，无法有效水平拓展
- 前端存储
	- 优点：
		- 不占用服务端内存
	- 缺点：
		- 存在安全风险
		- 数据大小受Cookie限制
		- 占用外网宽带（需要额外发送Session对象）
- Session粘滞
	- 优点：
		- 无需修改代码
		- 服务器可以水平拓展
	- 缺点：
		- 增加新机器会重新Hash，导致重新登录
		- 应用重启需要重新登录
- 后端集中存储
	- 优点：
		- 安全
		- 容易水平拓展
	- 缺点：
		- 增加复杂度
		- 需要修改代码

按实际业务需求进行选择
使用Redis进行分布式Session

## Redis 操作命令

**del key (删除任何类型的数据)**

- **String**
	- `set key value` (置 `key` 为 `value`)
	- `get key` (获取 `key` 值)
	- `mset key1 value1 key2 value2` (置多个值)
	- `mget key1 key2` (获取多个值)
- Hash
	- `hset key field value` (`key` 为 `redis` 键, `field` 为哈希键, `value` 为哈希值)
	- `hget key filed` (`key` 为 `redis` 键, `field` 为哈希键)
	- `hmset key field1 value1 filed2 value2`
	- `hmget key field1 field2`
	- `hgetall key` (获取所有 `key` 内所有哈希键和哈希值)
	- `hdel key field1 field2` (删除哈希数据)
- List (有序可重复)
	- `lpush key field1 field2`
	- `rpush key field1 field2`
	- `lrange key start stop` (查询 `key` 的 `list` 中从 `start` 到 `stop` 的项)
	- `llen key` (查询列表长度)
	- `lrem key count value` (删除列表, `count` 表示删除几个这样的值, `value` 为值)
(`lpush` 就是从左边开始 `push` , `rpush` 就是从右边开始 `push`)

- Set (内部排序不可重复集合)
	- `sadd key member1 member2` (向 `set` 中添加)
	- `smembers key` (查看 `set` 中所有元素)
	- `scard key` (查看 `set` 的长度)
	- `srem key member` (删除 `set` 中的 `member`)
- Sorted Set / Zset (自定义序列不可重复集合)
	- `zadd key score1 member1 score2 member2` (`score` 为分数, `member` 为成员, 在内部按分数进行排序)
	- `zrange key start stop` (从 `start` 到 `stop` 查看集合中的数)
	- `zcard key` (获取长度)
	- `zrem key member`(删除)

**失效时间 ( `EX` )**
`ttl` 为 `s`  , ` pttl` 为 `ms` 
*-1 为永不失效, -2为已失效*
不存在的键其对应的EX同样为 -2
- 设置 `key` 时直接设置失效时间
	- `set code test ex 10 ` (10s 后失效的名为 `code` 的 `set` `集合)
	- `ttl code` (查询 `code` 剩余的时间, 从 0 直接到 -2)
- 给已存在的 `key` 添加失效时间
	- `expire code second` (设置 code 的失效时间为 second 秒 )
	- `pexpire code millisecond` (设置 code 的失效时间为 millisecond 毫秒 )

- **之前不存在( `NX` )**
- **之前存在( `XX`)**


## Redis实现分布式Session

使用Redis存储Session实现分布式


