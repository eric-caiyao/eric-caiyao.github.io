---
layout: post
title: "MySQL Explain 学习"
description: ""
category: 技术分享
tags: [blog]
---
{% include JB/setup %}

### MySQL Explain 学习记录
> MySQL 提供的explain命令是查看查询优化器如何决定执行查询的主要方法，只支持select语句的分析，对于非select语句可以进行近似转换再使用explain进行分析，分析结果只能用于参考，因为该命令不会真正执行整条SQL语句，而是使用静态统计信息(比如表数据行数、索引列表等)对执行计划进行近似分析。

###### EXPLAIN 分类
1. EXPLAIN EXTENDED 使用该命令和 `show warnings` 可以查看MySQL优化器将原始SQL优化后的结果。
2. EXPLAIN PARTITIONS 该命令查询将要访问的分区信息。

###### EXPLAIN 执行结果分析

*测试数据库：sakila*
*测试SQL：*

``` sql
explain select actor_id ,
  (select 1 from film_actor where film_actor.actor_id = der_1.actor_id limit 1)
from (
  select actor_id
  from actor limit 5
) as der_1
union all
select film_id ,
  (select @var1 from rental limit 1)
  from (
    select film_id,
    (select 1 from store limit 1)
    from film limit 5
  ) as der_2
```

执行结果：

| id | select_type          | table      | type  | possible_keys | key                 | key_len | ref            | rows  | Extra           |
|----|----------------------|------------|-------|---------------|---------------------|---------|----------------|-------|-----------------|
|  1 | PRIMARY              | <derived3> | ALL   | NULL          | NULL                | NULL    | NULL           |     5 | NULL            |
|  3 | DERIVED              | actor      | index | NULL          | idx_actor_last_name | 137     | NULL           |   200 | Using index     |
|  2 | DEPENDENT SUBQUERY   | film_actor | ref   | PRIMARY       | PRIMARY             | 2       | der_1.actor_id |    13 | Using index     |
|  4 | UNION                | <derived6> | ALL   | NULL          | NULL                | NULL    | NULL           |     5 | NULL            |
|  6 | DERIVED              | film       | index | NULL          | idx_fk_language_id  | 1       | NULL           |  1000 | Using index     |
|  7 | SUBQUERY             | store      | index | NULL          | idx_unique_manager  | 1       | NULL           |     2 | Using index     |
|  5 | UNCACHEABLE SUBQUERY | rental     | index | NULL          | idx_fk_staff_id     | 1       | NULL           | 16005 | Using index     |
| NULL | UNION RESULT         | <union1,4> | ALL   | NULL          | NULL                | NULL    | NULL           |  NULL | Using temporary |

1. ID:
> 表示当前条目select在整条SQL查询中的顺序。当结果是UNION RESULT时，该字段为Null，因为该结果不存在查询SQL中。

2. select_type
> 当查询SQL不存在子查询，只是简单的select查询时，该字段为SIMPLE。如果存在子查询，可能存在PRIMARY、DERIVED、DEPENDENT SUBQUERY、UNION、DERIVED、SUBQUERY、UNCACHEABLE SUBQUERY、UNION RESULT，以下为各字段代表含义：
  *  PRIMARY： 最完成表select查询
  *  DERIVED： from后的select查询
  *  UNION： 整条SQL UNION链条第二个（第一个可能为PRIMARY或者DERIVED）开始以后的所有select查询。比如 A UNION B UNION ALL C 。那么B 、 C的type就是UNION
  * SUBQUERY： select 结果列表里的select查询
  * UNION： RESULT 用来存储UNION结果列表的匿名临时表
3. table
> 用于表示当前查询所操作的表名，对于子查询情况可能存在<deriveN>表示当前查询是引用id=N的查询结果，对于UNION的情况该字段显示<union1。。N>。表示该结果是1.。。。N这些查询合并所得。table字段中显示的字段就是执行计划真正关联的顺序，
对于上面的测试执行联接表执行顺序为： <derived3> -> actor -> film_actor -> <derived6> -> film -> store -> rental -> <union1,4>
4. type
> 该字段表示执行查询的类型，性能从好到坏依次为：
  * NULL： 表示在SQL优化后可以不访问表或者索引的数据直接获得查询结果，具体什么时候不需要访问表数据、什么时候不需要访问索引数据具体情况具体分析。例如查询索引键最小值可以直接通过访问B-Tree索引最左值即可，不需要访问表数据。
  * const,system： 表示可以将整条SQL查询转换成一个常亮，例如 `select primary_key_column from table_name where primary_key_column = input_primary_key_column`。 这条SQL直接可以转换成input_primary_key_column
  这个常亮。
  * eq_ref： 使用这种索引，MySQL直到最多只返回一条符合条件的记录，这种情况可能存在于使用主键或者唯一索引。
  * ref： 返回符合单个值的所有行，这种情况存在于使用非唯一索引和唯一索引前缀时。
  * range： 这种类型可能存在于两种情况，
    a. 在索引列上使用between或者>、<符号。 b. 使用in关键字列表。*（注意对于最左前缀原则：当使用between或者>、<符号筛选数据时前缀只能到当前范围参数，也就是如果该范围字段以后还有索引条件字段则无法使用索引。
    当使用in关键字列表时后面索引字段还可以索引。）*
  * index： 该类型表示会扫瞄全索引，根据索引顺序扫描全表数据。如果按照随机顺序访问行，开销将会非常大
  * ALL： 这是最坏的情况，按照行的顺序扫描全表。
3. possible_keys
> 表示在执行计划时可能会使用的索引。揭示哪一个索引能有助于高效的执行查找
4. key
> MySQL决定采用哪个索引来执行查询，该字段显示的值为MySQL选择的可以最小化执行成本的索引。
5. key_len
> 使用的索引字节数，显示使用的索引字段的最大长度，不是实际存储的数据长度。换言之，就是表字段定义的长度。
在计算该长度时要考虑到字符集的影响。
6. ref
> 对应与key字段表示具体使用的是哪个字段值，如果是固定值则表示为const。
7. row
> 表示为了找到符合查询的数据MySQL可能查找的数据行数，这个值是估计数据。可以简单将explain出来的多行rows相乘得到可能总的扫描行数，因为MySQL默认的表连接算法是内嵌循环关联，但是如果右连接字段如果建有索引，可以大大减少内层循环的次数，rows字段会显示为1。如果右连接的是子查询则必须全表扫描右查询的结果了，因为子查询会默认放到内存临时表里，临时表是没法用索引的。
8. Extra
> 该列显示不适合在其他列显示的数据，具体如下：
  * Using index：  使用覆盖索引
  * Using where：  MySQL服务器对于存储引擎返回的数据可以通过where字段过滤，出现这样的语句表示where字句中的字段可以通过添加索引优化，让数据
  过滤直接在存储引擎层完成
  * Using temporary： 使用临时表，可能是内存或者磁盘，没有字段可以区分这两种情况
  * Using filesort： 使用文件排序 可能是内存或者磁盘，没有字段可以区分这两种情况
  * Range checked for each record (index map:N)：
