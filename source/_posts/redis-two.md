---
title: redis_two
date: 2017-12-31 16:13:05
categories:
	- Redis
tags:
	- C
	- cach
	- db
---
摘要：redis
![](/images/redis1.jpg)
<!-- more -->
正文：
--------------------------------------------------------------------------------------------------------------------------------
## 解析Redis配置文件（不吝赐教了），常用的配置。
### 有些配置，我并没有亲自尝试，是从别人的博客中摘抄过来的，用之前请再次查阅官方文档。
```
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```
## redis的过期内存回收策略 五种 volatile-lru,volatile-random等策略
```
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# 最近最少使用算法（过期的数据）
# volatile-lru -> remove the key with an expire set using an LRU algorithm
# 最近最少使用 （所有的数据）
# allkeys-lru -> remove any key according to the LRU algorithm
# 随机去除过期数据
# volatile-random -> remove a random key with an expire set
# 随机去除所有数据 
# allkeys-random -> remove a random key, any key
#移除那些 TTL值最小的key 就是那些最近要过期的key
# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# 永不过期 一般不会使用
# noeviction -> don't expire at all, just return an error on write operations
#
```
## 常用的redis的配置 
1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
  daemonize yes
2.  当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
  pidfile /var/run/redis.pid
3. 指定Redis监听端口，默认端口为6379
4. bind 127.0.0.1
5. 当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
  timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
  loglevel verbose
7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
  logfile stdout
8. database 16 默认是0 可以使用 select<dbid>命令连接指定数据库id  
	database 16 
9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
## RDB的持久化配置, 
  save <seconds> <changes>
  Redis默认配置文件中提供了三个条件：  
  save 900 1  
  save 300 10  
  save 60 10000  
  分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。  
10.  指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大  
  rdbcompression yes
11.  指定本地数据库文件名，默认值是dump.rdb  
    dbfilename dump.rdb
12.  指定本地数据库存放目录  
   dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
    slaveof<masterip> <masterport> 
14. 当master服务设置了密码保护时，slav服务连接master的密码  
  masterauth <master-password>
15. requirepass foobared  默认是关闭的 
16. masclients 128 最大客户端连接数
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
  maxmemory <bytes>
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
  appendonly no
## AOF持久化配置
在Redis的配置文件中存在三种同步方式，它们分别是：
appendfsync always     #每次有数据修改发生时都会写入AOF文件。
appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。
appendfsync no          #从不同步。高效但是数据不会被持久化。

19. 指定更新日志文件名，默认为appendonly.aof
   appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值：   
  no：表示等操作系统进行数据缓存同步到磁盘（快）   
  always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）   
  everysec：表示每秒同步一次（折衷，默认值）  
  appendfsync everysec
21. 指定是否启用虚拟内存机制，默认值为no，
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
   vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
   vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
   vm-page-size 32
25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
   vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
   vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
  glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
  hash-max-zipmap-entries 64
  hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
  activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
  include /path/to/local.conf

Redis还可以用作消息队列，但是在这方面，阿里很多消息中间件做得更好
## 使用消息队列的必要性
1. 解耦：
允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。
2. 冗余：
消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。
3. 扩展性：
因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。
4. 灵活性&峰值处理能力：
在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。
5. 可恢复性：
系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
6. 顺序保证：
在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka保证一个Partition内的消息的有序性）
7. 缓冲：
有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。
8. 异步通信：
很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。



## 附上守护进程的概念，玩Linux的同学一定都知道拉。。。。
守护进程是在后台运行不受终端控制的进程（如输入、输出等），一般的网络服务都是以守护进程的方式运行。守护进程脱离终端的主要原因有两点：
（1）用来启动守护进程的终端在启动守护进程之后，需要执行其他任务。
（2）（如其他用户登录该终端后，以前的守护进程的错误信息不应出现）由终端上的一些键所产生的信号（如中断信号），不应对以前从该终端上启动的任何守护进程造成影响。要注意守护进程与后台运行程序（即加＆启动的程序）的区别。

