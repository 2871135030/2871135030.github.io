---
layout: post
title: Mongodb聚合操作
date:   2017-10-07 14:10:58
categories: [spring]
---

## Mongodb聚合操作

1、
```markdown
sql语句
select code,count(*) from user group by code having count(*)>2
mongodb脚本
db.user.aggregate([{$group:{_id:'$code',count:{$sum:1}}},{$match:{count:{$gte:2}}}]);

```
2、
```markdown
sql语句
select count(distinct name) from user 对应如下
mongodb脚本
db.user.distinct("name").length

```