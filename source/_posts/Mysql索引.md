---
title: Mysql索引以及优化笔记
date: 2018-3-15 21:18:40
categories:
	- MySQL
tags:
	- MySQL
---

## MySQL索引
主要有BTree索引、Hash索引、full-text全文索引、R-Tree索引。  

BTree性质
	索引字段要尽量的小
    索引的最左匹配特性
BTree是Innodb默认的索引
B+树是一个平衡的多叉树，从根节点到每个叶子节点的高度差值不超过1，而且同层级的节点间有指针相互链接。

B+树是一个平衡的多叉树，从根节点到每个叶子节点的高度差值不超过1，而且同层级的节点间有指针相互链接。
　　在B+树上的常规检索，从根节点到叶子节点的搜索效率基本相当，不会出现大幅波动，而且基于索引的顺序扫描时，也可以利用双向指针快速左右移动，效率非常高。

## hash索引
hash就是一种（key=>value）形式的键值对,允许多个key对应相同的value，但不允许一个key对应多个value,为某一列或几列建立hash索引，就会利用这一列或几列的值通过一定的算法计算出一个hash值，对应一行或几行数据. hash索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率.
HASH与BTREE比较:  
    hash类型的索引：查找单条快，查询范围慢
    bree类型的索引：范围查询和随机查询快（innodb默认索引类型）


## butong的索引引擎区别
* InnoDB 支持事务，支持行级别锁定，支持 Btree、Hash 等索引，不支持Full-text 索引；
* MyISAM 不支持事务，支持表级别锁定，支持 Btree、Full-text 等索引，不支持 Hash 索引；
* Memory 不支持事务，支持表级别锁定，支持 Btree、Hash 等索引，不支持 Full-text 索引
* NDB 支持事务，支持行级别锁定，支持 Hash 索引，不支持 Btree、Full-text 等索引；

## 聚促索引
InnoDB存储引擎表示索引组织表，即表中数据按照主键顺序存放。而聚集索引（clustered index）就是按照每张表的主键构造一棵B+树，同时叶子结点存放的即为整张表的行记录数据，也将聚集索引的叶子结点称为数据页。聚集索引的这个特性决定了索引组织表中数据也是索引的一部分。同B+树数据结构一样，每个数据页都通过一个双向链表来进行链接。
如果未定义主键，MySQL取第一个唯一索引（unique）而且只含非空列（NOT NULL）作为主键，InnoDB使用它作为聚簇索引。
如果没有这样的列，InnoDB就自己产生一个这样的ID值，它有六个字节，而且是隐藏的，使其作为聚簇索引。

# 由于实际的数据页只能按照一棵B+树进行排序，因此每张表只能拥有一个聚集索引。在多少情况下，查询优化器倾向于采用聚集索引。因为聚集索引能够在B+树索引的叶子节点上直接找到数据。此外由于定义了数据的逻辑顺序，聚集索引能够特别快地访问针对范围值得查询。

## Explain查询优化
1. id id列数字越大越先执行，如果说数字一样大，那么就从上往下依次执行  
2. select_type   
A：simple：表示不需要union操作或者不包含子查询的简单select查询。有连接查询时，外层的查询为simple，且只有一个
B：primary
C：union：union连接的两个select查询，第一个查询是dervied派生表，除了第一个表外，第二个以后的表select_type都是union
D：dependent union：与union一样，出现在union 或union all语句中，但是这个查询要受到外部查询的影响
E：union result：包含union的结果集，在union和union all语句中,因为它不需要参与查询，所以id字段为null
F：subquery：除了from字句中包含的子查询外，其他地方出现的子查询都可能是subquery
G：dependent subquery：与dependent union类似，表示这个subquery的查询要受到外部表查询的影响
H：derived：from字句中出现的子查询，也叫做派生表，其他数据库中可能叫做内联视图或嵌套select 
3. table 显示的查询表的名字
4. type 
const 表示一次通过索引就找到了  
primary 使用主键  
all 全表扫描，性能极差  
null 未使用索引  
ref 联合查询
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
5. possible_keys
指出MySQL能使用哪个索引在该表中找到行。如果是空的，没有相关的索引。这时要提高性能，可通过检验where子句，看是否引用某些字段，或者检查字段不是适合索引。
6. key
显示MySQL实际决定使用的键。如果没有索引被选择，键是null。
7. key_len
显示MySQL决定使用的键长度。如果键是null，长度就是null。文档提示特别注意这个值可以得出一个多重主键里MySQL实际使用了哪一部分。
8. ref
显示哪个字段或常数与key一起被使用。

9. rows
这个数表示MySQL要遍历多少数据才能找到，在InnoDB上是不准确的。

10. Extra
 如果是only index，这意味着信息只用索引树中的信息检索出的，这比扫描整个表要快。
    如果是where used，就是使用上了where限制。
    如果是impossible where 表示用不着where，一般就是没查出来啥。
    如果此信息显示using filesort或者using temporary的话会很吃力，where和order by的索引经常无法兼顾，如果按照where来确定索引，那么在order by时，就必然会引起using filesort，这就要看是先过滤再排序划算，还是先排序再过滤划算。

数据库的优化（一条sql中能使用一个索引，多个索引会自动选择最优的索引,从sql语句优化和索引两个部分回答）
原则
1. sql尽量使用索引
2. 对sql语句优化
子查询变成left join
limit 分布优化，先利用ID定位，再分页
or条件优化，多个or条件可以用union all对结果进行合并（union all结果可能重复）
不必要的排序
where代替having,having 检索完所有记录，才进行过滤
避免嵌套查询
对多个字段进行等值查询时，联合索引

## Sql 面试题
* 库表结构  *
    Student(S#,Sname,Sage,Ssex)学生表
    S#：学号
    Sname：学生姓名
    Sage：学生年龄
    Ssex：学生性别
    Course(C#,Cname,T#)课程表
    C#：课程编号
    Cname：课程名称
    T#：教师编号
    SC(S#,C#,score)成绩表
    S#：学号
    C#：课程编号
    score：成绩
    Teacher(T#,Tname)教师表
    T#：教师编号：
    Tname：教师名字
1. 查询 001课程比002课程成绩高的所有学生的学号
select a.S# from (select S#,score from SC where C#='001')a, (select s#,score from SC where c#='002')b Where a.score>b.score and a.s# = b.s#;

2. 查询平均成绩大于60分的同学的学号和平均成绩
select s#, avg(score) from sc group by s# having avg(score)>60;


```
public void say(String paramString) {
        String str = paramString;
        int i = -1;
        switch (str.hashCode()) {
        case 3254818:
            if (str.equals("java"))
                i = 0;
            break;
        case 109250886:
            if (str.equals("scala"))
                i = 1;
        }
        switch (i) {
        case 0:
            System.out.println("hello java!");
            break;
        case 1:
            System.out.println("hello scala!");
            break;
        default:
            System.out.println("no match!");
        }
    }
```