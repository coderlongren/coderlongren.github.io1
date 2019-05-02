---
title: redis_first
date: 2017-12-31 16:11:59
categories:
	- Redis
tags:
	- C
	- cach
	- db
	- redis
---
摘要：redis
摘要：没有摘要
![](/images/redis.jpg)
<!-- more -->
正文：
# Redis 使用命令总结

## key 
1. set key 该命令用于在key存在时删除该key
2. DUMP key 序列化该key,并且返回key被序列化之后的值
3. exists key 检查这个key 是否存在
4. expire key seconds 为key设置过期时间。单位为妙。
5. expireat key timestamp expireat和 expire 作用类似都是为key设置过期时间，不同于expireat接受Unix时间戳
6. PEXPIRE key milliseconds  设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
7. keys pattern 查找所有符合给定模式key
8. move key db 将当前数据库的key移动到数据库db中 
9. persist key 移除该key的过期时间，key将保持持久化 
10. ttl key 以秒为单位返回该key的剩余过期时间
11. pttl key 以秒为单位返回给定key的剩余过期时间 
12. randomkey 从当前数据库随机返回一个key
13. rename key newkey 修改key的名称 
14. type key 返回key所存储的值得类型 
## 字符串 
1. set key value 设置指定key的值
2. get key 获取指定key的值
3. getrange key start end  返回key中字符串值得子字符串
4. getset key value 将给定的key值设为value，并返回old key
5. getbit key offset 对 key 所储存的字符串值，获取指定偏移量上的位(bit)
 （用表格拉！！！）

|String命令           |  作用解释  |
| ------------- |:------------------:|
| mget key | 获取一个或者多个给定key的值
| SETEX key seconds value | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (
| setnx| 只有在key不存在时设置key的值
|  setlen | 返回key所存储的字符串长度
| mset key value[key value......] | 同时设置一个多个key-value对
| PSETEX key milliseconds value| 和setex 差不多，只不过十一毫秒为单位的
| incr key | 将key存储的值加一 
| incrby key value | 将key存储的指定值增加定值
| incrbyfloat key value | 将key存储的指定值增加浮点定值
|decr key | 将key中存储的值减一
| decrby key value | key 所储存的值减去给定的减量值（value）
| append key value | 将value追加到key的末尾
| ！阅读官方文档个人体会| redis数据结构是不是抄袭Python的？


## Hash 
|Hash命令           |  作用解释  |
| ------------- |:------------------:|
| hmset key value .....| 设置 key 的值
| hgetall key | 获取key的所有key-value|
| hdel key field | 删除一个或者多个哈希表字段
| hexists key field | 查看哈希表中，指定的字段是否存在
| hincrby key field increment | 为哈希表中指定key整数值增加increment
| hkeys key | 获取所有哈希表的字段
| hincrbyfloat key field increment | 为哈希表 key 中的指定字段的浮点数值加上增量 increment 
| hlen key | 获取哈希表中字段的数量
| hmset key field1,field2,,,| 获取所有给定字段的值
| hmset key field1 value1 [field2 value2]...| 同时将多个 field-value (域-值)对设置到哈希表 key 中
| hsetnx key field value | 只有在fiel字段不存在时，设置哈希表字段的值
| hvals key | 获取哈希表中所有值
| hscan key |
 
## List
|List命令           |  作用解释  |
| ------------- |:------------------:|
| lpush key  value1,value1...   | |将一个或多个值插入到列表头部
| lrange key start end | 获取列表范围内的元素
| rpop key | 移除并获取列表最后一个元素
| lpushx key value | 将一个值插入到已存在的列表头部
| llen key | 获取列表头部
| lindex key index | 通过索引获取列表中的元素
| lpop key | 移除获取列表的第一个元素
| linsert key before/after privo value | 在列表的元素前或者元素后，插入一个元素
| lpushx key value| 将一个值value插入到已存在的列表头部
| lrem key count value | 移除列表元素
| lset key index value | 通过索引设置元素的value
| LTRIM key start stop | 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
| rpushx key value | 为已存在的列表末尾添加值
| rpoplpush source destination | 将source rpop一个 并将这个元素 lpush进destination
| lset key before//after value1 value2 | 在一个元素前面或者后面插入元素


## Set
|Set命令           |  作用解释  |
| ------------- |:------------------:|
| sadd key value   |  向集合中set添加一个或多个成员
| scard key | 获得集合的成员数
| sdiff key1 key2 | 返回给定所有集合的差集
| sinter key1 key2 | 返回给定集合的交集
| SINTERSTORE destination key1 [key2] | 返回给定所有集合的交集并存储在 destination 中
|SISMEMBER key member | 判断 member 元素是否是集合 key 的成员
| smembers key | 返回集合中的所有元素 
| smove source destination member | 把 member元素从source集合移动到destination集合
| spop key | 随机返回集合中的一个元素
| srem key member1,member2 .... | 移除集合中一个或多个随机数 
| sunion key1 key2 返回给定集合的并集
| SUNIONSTORE destination key1 [key2] | 所有给定集合的并集存储在 destination 集合中


## sorted set 
what is set :集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。   
集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。 
## sorted Set
|sorted Set命令           |  作用解释  |
| ------------- |:------------------:|
| zadd key score1 member1[score2 member2] | 向有序集合中添加一个或者多个成员，或更新已存在的成员
| zcard key | 获取有序集合soredset的成员数
| zcount key min max | 计算有序集合中指定区间的成员数
| zrange key 0 -1 | 获取0~-1之间的 zset item
| zrem key member | 去掉Zset里面的一个 score-member
| ZRANK key member | 返回zset 指定成员的索引
| zrem key member1[member2] | 移除zset中的一个或者多个成员

未完待续。。。。。。
下一篇  解析Redis的配置文件