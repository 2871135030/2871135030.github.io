---
layout: post
title: 关于mysql的innodb 800w+数据不带条件count性能优化的思考
date:   2021-11-12 14:15:18
categories: [mysql]
---

##### 关于mysql的innodb 800w+数据不带条件count性能优化的思考

##### 1、查看mysql版本

```
mysql> show variables like '%version%';
+-------------------------+------------------------------+
| Variable_name           | Value                        |
+-------------------------+------------------------------+
| innodb_version          | 5.7.32                       |
| protocol_version        | 10                           |
| slave_type_conversions  |                              |
| tls_version             | TLSv1,TLSv1.1,TLSv1.2        |
| version                 | 5.7.32                       |
| version_comment         | MySQL Community Server (GPL) |
| version_compile_machine | x86_64                       |
| version_compile_os      | Win64                        |
+-------------------------+------------------------------+
```

##### 2、建表并初始化数据

###### 2.1、建表语句如下

```
mysql> desc user;
mysql> show create table user;

CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL,
  `birthday` date NOT NULL,
  `no` int(11) NOT NULL DEFAULT '0',
  `remark` varchar(255) NOT NULL,
  `modify_date` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  `create_date` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

###### 2.2、初始化500w+数据

```
insert into user(name,birthday,no,remark,modify_date,create_date) values("test-user-name",'2000-01-01',0,"测试count性能",'2019-01-01','2019-01-01');

反复执行以下语句直至数据量超过500w+
insert into user(name,birthday,no,remark,modify_date,create_date) select name,birthday,no,remark,modify_date,create_date from user ;

执行一些更新操作
update user set no=id+1000000;
```

##### 3、开启profiling参数并查看耗时
###### 3.1、开启profiling参数

```
mysql> show variables like '%profiling%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| have_profiling         | YES   |
| profiling              | OFF   |
| profiling_history_size | 15    |
+------------------------+-------+

mysql> set profiling=1;
Query OK, 0 rows affected

```

###### 3.2、查看count语句执行耗时

> 使用count(*)或count(1)耗时均超过4秒

```
mysql> select count(*) from user;
+----------+
| count(*) |
+----------+
|  8388608 |
+----------+

mysql> show profiles;
+----------+------------+-----------------------------------+
| Query_ID | Duration   | Query                             |
+----------+------------+-----------------------------------+
|        1 | 0.00172175 | show variables like '%profiling%' |
|        2 | 4.09369675 | select count(*) from user         |
+----------+------------+-----------------------------------+

mysql> show profile for query 2;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 4.7E-5   |
| checking permissions | 4E-6     |
| Opening tables       | 1.4E-5   |
| init                 | 8E-6     |
| System lock          | 5E-6     |
| optimizing           | 3E-6     |
| statistics           | 9E-6     |
| preparing            | 7E-6     |
| executing            | 2E-6     |
| Sending data         | 4.093506 |
| end                  | 8E-6     |
| query end            | 2E-5     |
| closing tables       | 7E-6     |
| freeing items        | 4.8E-5   |
| cleaning up          | 9E-6     |
+----------------------+----------+
```

> 在user表没有辅助索引，仅有一个主键索引的情况下，可以看到耗时超过4秒

###### 3.3、创建no字段的辅助索引再进行count统计

```
create index index_no on user(no);
```

###### 3.4、查看当前索引

```
mysql> show index from user;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| user  |          0 | PRIMARY  |            1 | id          | A         |     7482923 | NULL     | NULL   |      | BTREE      |         |               |
| user  |          1 | index_no |            1 | no          | A         |     8159175 | NULL     | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
```

###### 3.5、增加辅助索引后可以看到count速度有明显的提升

> 可通过`show profiles`查看最新的`queryid`，下面的23为最新count查询id

```
mysql> show profile for query 23;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 5.1E-5   |
| checking permissions | 4E-6     |
| Opening tables       | 1.4E-5   |
| init                 | 8E-6     |
| System lock          | 5E-6     |
| optimizing           | 3E-6     |
| statistics           | 1E-5     |
| preparing            | 7E-6     |
| executing            | 1E-6     |
| Sending data         | 1.344094 |
| end                  | 8E-6     |
| query end            | 7E-6     |
| closing tables       | 6E-6     |
| freeing items        | 3.1E-5   |
| cleaning up          | 1.1E-5   |
+----------------------+----------+
```

##### 4、通过explain执行计划分析

```
mysql> explain select count(*) from user;
+----+-------------+-------+------------+-------+---------------+----------+---------+------+---------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | user  | NULL       | index | NULL          | index_no | 4       | NULL | 8159175 |      100 | Using index |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+---------+----------+-------------+

mysql> drop index index_no on user;
Query OK, 0 rows affected
Records: 0  Duplicates: 0  Warnings: 0
mysql> explain select count(*) from user;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | user  | NULL       | index | NULL          | PRIMARY | 4       | NULL | 8159175 |      100 | Using index |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
```

> 通过explain执行计划可以看到`count`统计查询在有辅助索引时，选择了走辅助索引，没有时选择了`PRIMARY`主键索引，从结果可以看到主键索引效率不高反而比较慢。这是为什么呢？通常mysql普通的数据检索时主键索引会比普通索引快，原因是主键索引不需要回表，而`count`是什么原因会造成有如此大的差异呢？


##### 5、继续通过optimizer trace跟踪优化器过程

###### 5.1、查看optimizer_trace状态

```
mysql> show variables like '%optimizer_trace%';
+------------------------------+----------------------------------------------------------------------------+
| Variable_name                | Value                                                                      |
+------------------------------+----------------------------------------------------------------------------+
| optimizer_trace              | enabled=off,one_line=off                                                   |
| optimizer_trace_features     | greedy_search=on,range_optimizer=on,dynamic_range=on,repeated_subselect=on |
| optimizer_trace_limit        | 1                                                                          |
| optimizer_trace_max_mem_size | 16384                                                                      |
| optimizer_trace_offset       | -1                                                                         |
+------------------------------+----------------------------------------------------------------------------+
```

###### 5.2、开启并查看执行情况

* 开启optimizer_trace

```
mysql> show variables like '%optimizer_trace%';
+------------------------------+----------------------------------------------------------------------------+
| Variable_name                | Value                                                                      |
+------------------------------+----------------------------------------------------------------------------+
| optimizer_trace              | enabled=off,one_line=off                                                   |
| optimizer_trace_features     | greedy_search=on,range_optimizer=on,dynamic_range=on,repeated_subselect=on |
| optimizer_trace_limit        | 1                                                                          |
| optimizer_trace_max_mem_size | 16384                                                                      |
| optimizer_trace_offset       | -1                                                                         |
+------------------------------+----------------------------------------------------------------------------+

mysql> set optimizer_trace="enabled=on";
Query OK, 0 rows affected
mysql> set end_markers_in_json=on; 
Query OK, 0 rows affected

```

* 在有索引与无索引情况下执行count查询后均使用以下查询语句获取优化器优化过程

```
SELECT * FROM information_schema.OPTIMIZER_TRACE;
```

> 可以观察到在有辅助索引及无辅助索引`index_no`的两种情况下，优化器的执行过程一样，并没有差异，没有体现出性能的差异，不过在`optimizer_trace`结果中可以看到是`scan`；

```
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ],
                "table": "`user`",
                "best_access_path": {
                  "considered_access_paths": [
                    {
                      "rows_to_scan": 8159175,
                      "access_type": "scan",
                      "resulting_rows": 8.16e6,
                      "cost": 1.67e6,
                      "chosen": true
                    }
                  ]
                },
                "condition_filtering_pct": 100,
                "rows_for_plan": 8.16e6,
                "cost_for_plan": 1.67e6,
                "chosen": true
              }
            ]
```


##### 6、既然都是scan那应该就是和本身的结构有关了

###### 6.1、聚簇索引（clustered index）和非聚簇索引（secondary index）的区别

> innodb的clustered index是把primary key以及row data保存在一起的，而secondary index则是单独存放，然后有个指针指向primary key，所以二级索引树比主键索引树小。因此优化器基于成本的考虑，优先选择的是二级索引。


###### 6.2、验证

* 创建新表`user_no`仅保留`id`及`no`字段

```
CREATE TABLE `user_no` (
  `id` int(10) unsigned NOT NULL,
  `no` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `index_no` (`no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

* 插入所有数据

```
insert into user_no select id,no from user
```

> 基于上面的结论，如果表的主键索引树与二级索引树差别不大的话应该可以获得相近的结果，默认`count(*)`时会选择二级索引（辅助索引）`index_no`，可以通过`explain`检查

```
mysql> explain select count(*) from user_no;
+----+-------------+---------+------------+-------+---------------+----------+---------+------+---------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key      | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+----------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | user_no | NULL       | index | NULL          | index_no | 4       | NULL | 8374395 |      100 | Using index |
+----+-------------+---------+------------+-------+---------------+----------+---------+------+---------+----------+-------------+
1 row in set

mysql> explain select count(*) from user_no force index(primary);
+----+-------------+---------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | user_no | NULL       | index | NULL          | PRIMARY | 4       | NULL | 8374395 |      100 | Using index |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
1 row in set

mysql> 
```

* 开启`profiling`参数并执行统计，查看耗时的差异

> 将分别统计`user`表（此表无辅助索引）、`user_no`表的两种情况（默认走辅助索引`index_no`以及主键索引）

```
mysql> set profiling=1;
Query OK, 0 rows affected

mysql> select count(*) from user;
+----------+
| count(*) |
+----------+
|  8388608 |
+----------+
1 row in set

mysql> select count(*) from user_no;
+----------+
| count(*) |
+----------+
|  8388608 |
+----------+
1 row in set

mysql> select count(*) from user_no force index(primary);
+----------+
| count(*) |
+----------+
|  8388608 |
+----------+
1 row in set

mysql> show profiles;
+----------+------------+---------------------------------------------------+
| Query_ID | Duration   | Query                                             |
+----------+------------+---------------------------------------------------+
|        1 |   4.862296 | select count(*) from user                         |
|        2 |  1.8102375 | select count(*) from user_no                      |
|        3 | 1.98250375 | select count(*) from user_no force index(primary) |
+----------+------------+---------------------------------------------------+
3 rows in set

mysql> 
```

> 可以看到当表`user_no`只剩下2个字段时，使用主键索引或辅助索引时略有差异并不大，按照这个结论，尝试在大表（较多字段）中使用了`like`操作，同样能够得到不错的性能提升`select id from user force index(index_name) where name like '%33D0EB9%' ;`

> 不同的机器性能或不同的mysql版本可能表现不一样



#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

