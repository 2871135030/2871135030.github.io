---
layout: post
title: Windows环境下MariaDB 10.3开启慢查询
date:   2021-11-11 14:11:58
categories: [mysql]
---

## Windows环境下MariaDB 10.3开启慢查询


1、在MairaDB的配置文件my.ini的配置标识[mysqld]下，增加以下配置
```markdown
[mysqld]
slow_query_log = ON
slow_query_log_file = C:/Program Files/MariaDB 10.3/data/slow.log
long_query_time = 0.2
```

其中 long_query_time表示慢查询阈值，设置为大于200ms即为慢查询；设置成功后可使用命令查看mysql变量，可以看到slow_query_log标识已为ON（默认为OFF），

```
MariaDB [mysql]> show variables like '%query%';
+------------------------------+---------------------------------------------+
| Variable_name                | Value                                       |
+------------------------------+---------------------------------------------+
| expensive_subquery_limit     | 100                                         |
| ft_query_expansion_limit     | 20                                          |
| have_query_cache             | YES                                         |
| long_query_time              | 0.200000                                    |
| query_alloc_block_size       | 16384                                       |
| query_cache_limit            | 1048576                                     |
| query_cache_min_res_unit     | 4096                                        |
| query_cache_size             | 1048576                                     |
| query_cache_strip_comments   | OFF                                         |
| query_cache_type             | OFF                                         |
| query_cache_wlock_invalidate | OFF                                         |
| query_prealloc_size          | 24576                                       |
| slow_query_log               | ON                                          |
| slow_query_log_file          | C:/Program Files/MariaDB 10.3/data/slow.log |
+------------------------------+---------------------------------------------+
14 rows in set (0.003 sec)

MariaDB [mysql]>
```

2、下面使用一些慢查询看看设置效果，为了方便测试直接使用sleep语句完成模拟，多次执行以下语句，其中rand()函数表示生成0~1之间的随机数，sleep()函数表示休眠时间，
上面设置的慢查为0.2秒(200毫秒)，注意观察执行的结果
```markdown
select sleep(rand());

```

3、部分结果如下

```markdown
 select sleep(rand());
# Time: 191126 23:41:36
# User@Host: root[root] @ localhost [127.0.0.1]
# Thread_id: 9  Schema: user  QC_hit: No
# Query_time: 0.811652  Lock_time: 0.000000  Rows_sent: 1  Rows_examined: 0
# Rows_affected: 0  Bytes_sent: 61
SET timestamp=1574782896;
select sleep(rand());
# Time: 191126 23:41:39
# User@Host: root[root] @ localhost [127.0.0.1]
# Thread_id: 9  Schema: user  QC_hit: No
# Query_time: 0.838039  Lock_time: 0.000000  Rows_sent: 1  Rows_examined: 0
# Rows_affected: 0  Bytes_sent: 61
SET timestamp=1574782899;
select sleep(rand());
# Time: 191126 23:41:40
# User@Host: root[root] @ localhost [127.0.0.1]
# Thread_id: 9  Schema: user  QC_hit: No
# Query_time: 0.369917  Lock_time: 0.000000  Rows_sent: 1  Rows_examined: 0
# Rows_affected: 0  Bytes_sent: 61
SET timestamp=1574782900;
select sleep(rand());
# Time: 191126 23:41:41
# User@Host: root[root] @ localhost [127.0.0.1]
# Thread_id: 9  Schema: user  QC_hit: No
# Query_time: 0.323326  Lock_time: 0.000000  Rows_sent: 1  Rows_examined: 0
# Rows_affected: 0  Bytes_sent: 61
SET timestamp=1574782901;
select sleep(rand());

```
