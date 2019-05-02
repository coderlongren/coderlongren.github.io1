---
title: explain语句怎么用
date: 2018-4-3 10:18:40
categories:
	- DB
tags:
	- MySQL
---
摘要：MySQL优化之 explain
<!-- more -->
看完《高性能MySQL》这本书之后，终于敢在自己的简历上写上一行"熟悉MySQL数据库，能够进行基本的调优工作"，不过MySQL对Web开发人员来说的确的是很重要的。  
你说你会基本的调优，那么咱们先来分析一下最基本的SQL语句的调优工作吧？  
`explain`就这个时候闪亮登场了。
用法嘛，看一遍就记住了。  
`explain + select * from table_name where ........`  
![mysql](https://images2015.cnblogs.com/blog/763020/201604/763020-20160417142105895-121211600.png)

* id: SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符.
* select_type: SELECT 查询的类型.
* table: 查询的是哪个表
* partitions: 匹配的分区
* type: join 类型
* possible_keys: 此次查询中可能选用的索引
* key: 此次查询中确切使用到的索引.
* ref: 哪个字段或常数与 key 一起被使用
* rows: 显示此查询一共扫描了多少行. 这个是一个估计值.
* filtered: 表示此查询条件所过滤的数据的百分比
* extra: 额外的信息

## id
id列数字越大越先执行，如果说数字一样大，那么就从上往下依次执行.

## select_type字段,常见的就是 simple primary,  
A：simple：表示不需要union操作或者不包含子查询的简单select查询。有连接查询时，外层的查询为simple，且只有一个
B：primary：一个需要union操作或者含有子查询的select，位于最外层的单位查询的select_type即为primary。且只有一个
C：union：union连接的两个select查询，第一个查询是dervied派生表，除了第一个表外，第二个以后的表select_type都是union
D：dependent union：与union一样，出现在union 或union all语句中，但是这个查询要受到外部查询的影响
E：union result：包含union的结果集，在union和union all语句中,因为它不需要参与查询，所以id字段为null
F：subquery：除了from字句中包含的子查询外，其他地方出现的子查询都可能是subquery
G：dependent subquery：与dependent union类似，表示这个subquery的查询要受到外部表查询的影响
H：derived：from字句中出现的子查询，也叫做派生表，其他数据库中可能叫做内联视图或嵌套select
## table字段
大家一看就懂，从哪个表查的。
## type  重要字段`
指的联合查询使用的类型  
1. system：表中只有一行数据或者是空表，且只能用于myisam索引和memory索引表。如果是Innodb引擎表，type列在这个情况通常都是all或者index
2. const：使用唯一索引或者主键，返回记录一定是1行记录的等值where条件时，通常type是const。
3. eq_ref：出现在要连接过个表的查询计划中，驱动表只返回一行数据，且这行数据是第二个表的主键或者唯一索引，且必须为not null，唯一索引和主键是多列时，只有所有的列都用作比较时才会出现eq_ref
4. ref：不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现。
5. fulltext：全文索引检索，全文索引的优先级很高，若全文索引和普通索引同时存在时，mysql不管代价，优先选择使用全文索引
6. ref_or_null：与ref方法类似，只是增加了null值的比较。实际用的不多。
7. unique_subquery：用于where中的in形式子查询，子查询返回不重复值唯一值
8. index_subquery：用于in形式子查询使用到了辅助索引或者in常数列表，子查询可能返回重复值，可以使用索引将子查询去重。
9. range：索引范围扫描，常见于使用>,<,is null,between ,in ,like等运算符的查询中。
`-------------------->>>华丽的分界线，type到达这里就可以了`
10. index_merge：表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取所个索引，性能可能大部分时间都不如range
11. index：索引全表扫描，把索引从头到尾扫一遍，常见于使用索引列就可以处理不需要读取数据文件的查询、可以使用索引排序或者分组的查询。
12. all：这个就是全表扫描数据文件，然后再在server层进行过滤返回符合要求的记录。最坏的情况。

`system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
一般来说，得保证查询至少达到range级别，最好能达到ref。`
## possible_keys
可能用到的索引
## key
查询真正使用到的索引
## key_len
用于处理查询的索引长度，如果是单列索引，那就整个索引长度算进去，如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列，这里不会计算进去。

## ref
如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func
## rows
执行查询估计的扫描行数，非精确

## Extra
using where  
using idex  

再来一点优化方案的小例子。  
add index idx_ABC on (a,b,c);
where a = 3 | 只用到了 a   
where a = 3 and b = 5 | 只用到了 a,b   
where a = 3 and b = 5 and c = 6| 用到了 a,b,c   
where a = 3 od b = 5 od c = 6| 用不到索引
b line 'kk%' 可以用到  
b line '%kk' 用不到, 左值匹配  

### EXIT And In的比较使用
in 是把外表和内表作hash 连接，而exists是对外表作loop循环，每次loop循环再对内表进行查询。
如果两张表的大小是差不多的，那么in 和 exit的效率的确是差不多的，可是如果是一个小表和一个大表关联查询的话，
select * from A where id in (select id from B)
应该尽可能的让B表比A表小，这样使用小表驱动大表增加效率。  
相同的 
select * from A where exists(select id from B where cc=A.id)
应该把外围的查询使用小表，因为会用到exits()内的索引。  
总的来说，就是尽量小表驱动大表。

### 为使用排序而使用索引的规范
`KEY a_b_c (a,b,c)
order by 能使用索引最左前缀
order by a
order by a,b
order by a,b,c
order by a DESC,b DESC, c DESC # 全部升序，或者全部降序

# 如果where 使用索引最左前缀为const orderby可以使用索引
where a = const order by b,c
where a = const and b = const order by c
where a = const and b > const order by b,c
## 不可以的情况
排序不一致，不可以使用
where g = const orderby b,c 丢失了a索引
where a = const orderby c 丢失了b索引
where a = const order by a,d d不是索引
where a in(.....) order by b,c带头的a索引是范围值，不知道是否能用上索引
`
### Group by关键字的优化
其实group by实质就是先排序之后，在进行分组，同样遵循最左前缀  
能在where限定条件之下，尽量不要使用having在进行限定  

### profile的优化
慢查询无果后，可以使用show profile cpu, block io for query query_id
来诊断sql，一般出现数据库压力的也就几种情况。
`
converting HEAP to MyISAM 这是由于查询结果过大，内存会往次磁盘搬运  
Creating tmp table 显而易见的创建了临时表，copy->remove   
Coping to tmp table on dick 内存临时表复制到磁盘，磁盘IO影响较大  
locked 加锁了。
 `
Begin
select * from table_name where ... for update 锁定一行  


 
