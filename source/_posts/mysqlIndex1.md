title: mysql索引命中分析
date: 2017/11/30 14:00:00
thumbnail: /img/context/mysql.jpeg
tags:
    - mysql
    - java
    - SQL
categories:
    - 数据库
---

## 索引

在关系型数据库中，索引的存在可以极大的提升关系型数据的查询效率。在mysql中，索引分为聚簇索引和非聚簇索引。

### 聚集规则

* 聚集规则是：有主键则定义主键索引为聚集索引；
* 没有主键则选第一个不允许为NULL的唯一索引；
* 还没有就使用innodb的内置rowid为聚集索引。

### 索引高度

 mysql的索引无论是聚集索引还是非聚集索引，都是B+树结构。聚集索引的叶子节点存放的是数据，非聚集索引的叶子节点存放的是非聚集索引的key和主键值。B+树的高度为索引的高度。 

* 聚集索引的高度决定了根据主键取数据的理论IO次数。根据非聚集索引读取数据的理论IO次数还要加上访问聚集索引的IO次数总和。实际上可能要不了这么多IO。因为索引的分支节点所在的Page因为多次读取会在mysql内存里cache住。 
* mysql的一个block大小默认是16K，可以根据索引列的长度粗略估算索引的高度。

### sql优化依据 

SQL语句中的where条件，使用以上的提取规则，最终都会被提取到Index Key (First Key & Last Key)，Index Filter与Table Filter之中。

* Index First Key，只是用来定位索引的起始范围，因此只在索引第一次Search Path(沿着索引B+树的根节点一直遍历，到索引正确的叶节点位置)时使用，一次判断即可；
* Index Last Key，用来定位索引的终止范围，因此对于起始范围之后读到的每一条索引记录，均需要判断是否已经超过了Index Last Key的范围，若超过，则当前查询结束；
* Index Filter，用于过滤索引查询范围中不满足查询条件的记录，因此对于索引范围中的每一条记录，均需要与Index Filter进行对比，若不满足Index Filter则直接丢弃，继续读取索引下一条记录；
* Table Filter，则是最后一道where条件的防线，用于过滤通过前面索引的层层考验的记录，此时的记录已经满足了Index First Key与Index Last Key构成的范围，并且满足Index Filter的条件，回表读取了完整的记录，判断完整记录是否满足Table Filter中的查询条件，同样的，若不满足，跳过当前记录，继续读取索引的下一条记录，若满足，则返回记录，此记录满足了where的所有条件，可以返回给前端用户

## 索引分析

一条sql语句要执行完成需要经历什么样的过程？
当一条sql语句提交给mysql数据库进行查询的时候需要经历以下几步 ：

* 先在where解析这一步把当前的查询语句中的查询条件分解成每一个独立的条件单元 
* mysql会自动将sql拆分重组 
* 然后where条件会在B-tree index这部分进行索引匹配，如果命中索引，就会定位到指定的table records位置。如果没有命中，则只能采用全部扫描的方式
* 根据当前查询字段返回对应的数据值 

## 命中判断

MySql中是通过 **Explain** 命令来分析低效SQL的执行计划。命令的使用很简单.

首先，我们定义表结构并录入数据，测试表结构如下：

```sql
/*Table structure for table `history_record` */

DROP TABLE IF EXISTS `history_record`;

CREATE TABLE `history_record` (
  `id` bigint(18) NOT NULL COMMENT '历史记录表',
  `imei` varchar(15) COLLATE utf8_unicode_ci NOT NULL COMMENT '设备唯一标识',
  `longitude` varchar(200) COLLATE utf8_unicode_ci NOT NULL COMMENT '经度',
  `lat` varchar(200) COLLATE utf8_unicode_ci NOT NULL COMMENT '纬度',
  `peroid` int(2) DEFAULT '50' COMMENT '显示历史记录时间段',
  `status` tinyint(1) DEFAULT '1' COMMENT '状态',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

```

我选取最近项目中使用的设备地理位置经纬度历史记录表进行测试分析。

使用 Explain命令进行分析操作，

```
EXPLAIN SELECT * from test_watch.history_record；
```
执行结果如图所示：

![EXPLAIN](/img/context/EXPLAIN.png)

执行的每列结果说明：

| 名称        | 含义    | 备注 |
|:------- |:----------:|: ---:|
| select_type  | 查询类型 | 查询类型，常见的值SIMPLE：简单表，不使用表连接或子查询。PRIMARY : 主查询，外层的查询。UNION 第二个或者后面的查询语句。SUBQUERY : 子查询中的第一个select |
| table  | 输出结果的表      | --- |
| type  | 表示MySql在表中找到所需行的方式，或者叫访问类型。      | 常见的类型：ALL index rangeEXPLAIN.png ref eq_ref const system NULL 从左到右，性能由最差到最好。 下见详细解读 |
| possible_keys  | 可能使用的索引列表      | --- |
| key  | 实现执行使用索引列表      |--- |
| key_len  | 索引的长度      | --- |
| ref  | 显示使用哪个列或常数与key一起从表中选择行。      | --- |
| row  | 执行查询的行数，简单且重要，数值越大越不好，说明没有用好索引      | ---|
| filtered  | 过滤      | --- |
| Extra  | 该列包含MySQL解决查询的详细信息     | 见下文详细解读|

###  type 详细解读

1. type＝ALL　全表扫描 
2. type＝index 索引全扫描，遍历整个索引来查询匹配的行
3. type=range 索引范围扫描，常见于　<,<=,>,>=,between,in等操作符。例如
	* explain select * from adminlog where id>0 , 
	* explain select * from adminlog where id>0 and id<=100
	* explain select * from adminlog where id in (1,2) 
4. type=ref　使用非唯一索引或唯一索引的前缀扫描，返回匹配某个单独值的记录行。ref还经常出现在JOIN操作中
5. type=eq_ref 类似于ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中有一条记录匹配；简单来说，说是多表连接中使用　主建或唯一健作为关联条件
6. type=const/system 单表中最多有一个匹配行。主要用于比较primary key [主键索引]或者unique[唯一]索引,因为数据都是唯一的，所以性能最优。条件使用=。 
7. type=NULL　不用访问表或者索引，直接就能够得到结果 例如 EXPLAIN SELECT 1 from history_record;,类型type 还有其他值
	* ref_or_null : 与ref 类似，区别在于条件中包含对NULL的查询
	* index_merge : 索引合并优化
	* unique_subquery : in的后面是一个主键字段的子查询
	* ndex_subquery : 与unique_subquery 类似,区别在于in的后面是查询非唯一索引字段的子查询

### Extra 详细解读

1. Not exists 没有找到合适的索引 
2. using index 只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的信息。就是建议取索引列。这样就可以不要通过索引去实际表中找数据了。直接返回索引列的数据。一次查询。否则就是索引表查一次，实际表中查一次。
3. using temporary 为了解决查询，MySQL需要创建一个临时表来容纳结果。典型情况如查询包含可以按不同情况列出列的GROUP BY和ORDER BY子句时。


## 无效索引

数据变化不大的列。如某类型，是否有效，项目ID等列的索引都是无效的。这些无效索引还是影响Insert 、Update、Delete 语句的性能。因为这些语包的执行都要对索引表进行更新。又因为这些表的值变化不大，数据库很难为他们合理分配索引。所以影响语句的性能。

### IN,OR 是否会走索引？

一条SQL会不会走索引一个看条件使用的运算符，另一个看有没有索引。所以SQL会不会走索引和IN.OR,group by 没有关系。

但是 对于OR 来说， OR前后两个条件都要有索引整个SQL才会使用索引！只要有一个条件没索引那么整个SQL都不使用索引。如果出现OR的一个条件没有索引时，建议使用 union 


### 不走索引的运算符

什么运算符不走索引 为  <>,!= 运算符


