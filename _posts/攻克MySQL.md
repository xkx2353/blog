---
title: 攻克MySQL
date: 2020-09-05 13:11:07
---
##### 数据库基本介绍
> 先没有系统的记录，就是想起来哪些，然后记录下来，后续进行系统一点

1. 范式。
2. 关系型数据库和非关系型数据库。
3. SQL：Strcuted Query Language，分为：
	DDL：Data Definition Language ，定义数据库的对象，比如，database，table，column；
	DML：Data Manipulation Language，对数据库的表记录进行操作；
	DCL：Data Control Lauguage，定义数据库访问权限和安全问题；
	DQL：Data Query Language，用来查询表中的记录。
4. 按操作对象进行分类，分为三类：库操作，表操作，数据操作；

##### 一些问题记录
1. 关于日期时间类型：DATE，DATESTAMP，TIMESTAMP 的区别？

```
DATE保存精度到天，格式为：YYYY-MM-DD，如 2020-09-05;
DATETIME，TIMESTAMP 保存精度保存到秒，格式为：YYYY-MM-DD HH:MM:SS,如：2020-09-05 14:13:59;can have fractional seconds part up to 6 digit microsecond precision，可以到微妙	
--------------------------------------
TIMESTAMP会跟随设置的时区变化而变化，而DATETIME保存的是绝对值不会变化，
--------------------------------------
关于日期不同类型占用的存储看下图
--------------------------------------
Supported range for DATETIME is '1000-01-01 00:00:00' to '9999-12-31 23:59:59' while for TIMESTAMP, it is '1970-01-01 00:00:01' UTC to '2038-01-09 03:14:07' UTC
UTC 是 Coordinated Universal Time 的缩写，译为中文为“世界标准时间”
UTC + 时区差 ＝ 本地时间；比如SHANGHAI，就是UTC +8
---------------------------------------
TIMESTAMP data can be indexed while the DATETIME data cannot
---------------------------------------
Queries with DATETIME will not be cached but queries with TIMESTAMP will be cached
---------------------------------------
Automatic initialization can happen for both;
change the data while updating the record with current data time as per the constraint.
`DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6)`
```
<center><img src="https://azou.tech/blog/static/image/mysql_date_datetime_timestamp.png" ></center>
<center>From here <a> https://dev.mysql.com/doc/refman/5.7/en/storage-requirements.html</a> </center>

##### 好玩的

```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍

##### 参考
- https://www.tutorialspoint.com/lua/index.htm


