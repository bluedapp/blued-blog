---
uuid: ce532aa0-4c7d-11e8-b96e-ef2d752197a5
author: jiandandkl
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars1.githubusercontent.com/u/16009933?v=3
title: 面对有2亿条数据的mysql表
date: 2018-04-30 21:53:05
tags: mysql node
---

 > 看到这个2亿5千条数据的表，我的内心是拒绝的，各种条件筛选要取出相应的数据，被折磨了两天，现在记录下心路历程

 ** 先分享下mysql相关的知识点 **
 * 1 名词解释

    主键(PRIMARY KEY): 唯一索引，不能重复
    组合主键(UNIQUE): 组合索引，若干个字段组成一个主键

* 2 SELECT必备 -- EXPLAIN
> 使用EXPLAIN来查看sql语句是否能命中索引，量大的表必须要命中索引才能执行，也还得看以下条件

    (1) type
    ALL， index，  range， ref， eq_ref， const， system， NULL（从左到右，性能从差到好）(具体就不在这解释了)
    (2) possible_keys
    可能用到的索引，但不一定命中
    (3) key
    实际使用的key。想强制使用某个索引，可用FORCE INDEX。
    (4) key_len
    根据表定义计算而得的索引中使用的字节数，越短越好。
    (5) rows
    可能要读取的行数
    (6) Extra (划重点)
    Using Index: 表示使用索引，叫做覆盖索引，没有查询数据表，只查询了索引。如果同时出现Using Where代表可以用到索引，但是需要查询数据表。
    Using where: 表示条件查询，不读取所有数据或者通过索引获取所需的数据。
    Using filesort: 文件排序，无法利用索引的完成的排序操作
    Using temporary: 需要使用临时表来存储结果集，常见于多表联合查询和排序
> 如果出现Using filesort和Using temporary，尽量优化sql语句

** 开始操作这个2亿条数据的表 **

* 1 命中联合主键

表中有若干个联合主键，就尽量往这些联合主键上靠了。

```javascript
EXPLAIN 
SELECT *
FROM users_orders
WHERE
	class IN (1, 2, 3) AND
	score > 90 AND
	sex = 1
ORDER BY id DESC
LIMIT 0, 60
```

| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | student | range | index2， index3， index4， index5 | index2 | 44 | NULL | 642323 | Using index condition; Using filesort |

可以命中联合主键索引，但rows特别多，执行非常慢，这个方案就pass了。

* 2 命中联合主键进阶版

方案一中有些条件筛选命中联合主键速度还可以，但是加上`ORDER BY time`(time也是个索引)后就不行了。
主键id是时间加随机数，大体能满足排序，就改成了`ORDER BY id`，发现命中的索引还是联合主键。
对sql语句进行了如下修改:

```javascript
EXPLAIN
SELECT *
FROM student AS st1
LEFT JOIN student AS st2
ON st1.id = st2.id
WHERE
    st1.class IN (1, 2, 3) AND
    st1.score > 90 AND
    st1.sex = 1
ORDER BY st1.id DESC
LIMIT 0, 60
```
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | st1 | range | index2， index3， index4， index5 | index2 | 44 | NULL | 642323 | Using where; Using index; Using filesort |
| 1 | SIMPLE | st2 | eq_ref | PRIMARY | PRIMARY | 413800 | st.id | 1 | NULL |

这样命中了联合主键和唯一主键，执行速度也很快，但是如果量比较大，改为`st1.score > 60`，就又不行了，第二个方案也pass。

* 3 命中主键

方法总比困难多，最后只能往主键索引上靠了。先限定时间区间(可命中主键索引)，能有效控制要筛选的数据量，再对时间区间内的数据筛选。

```javascript
EXPLAIN
SELECT *
FROM student
WHERE
	(id LIKE '20180501%' OR id LIKE '20180502%' OR id LIKE '20180503%') AND
	class IN (1, 2, 3) AND
	score > 90 AND
	sex = 1
ORDER BY id DESC
LIMIT 0, 60
```

| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | student | range | PRIMARY， index2， index3， index4， index5 | PRIMARY | 98 | NULL | 413800 | Using where |

这样命中主键索引，Extra是Using where，执行速度也是相关快。
> 某个条件筛选时Extra会有Using filesort，但执行速度挺快，就没管了。

 ** 总结 **
 面对这样的mysql表只有命中主键索引啦。
