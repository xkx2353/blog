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
2. TRUNCATE DROP DELETE区别
```
[DELETE] is a DML;It is comparatively slower than TRUNCATE cmd ;We can use the “ROLLBACK” command to restore the tuple.
[DROP] is a DDL;It is use to drop the whole table,drop the whole structure .Removes the named elements of the schema;Can’t restore the table by using the “ROLLBACK” command
[TRUNCATE] is a DDL; It is use to delete all the rows of a relation (table) ;It is comparatively faster than delete command as it deletes all the rows fastly. Can’t rollback the data after using the this command
------------
The TRUNCATE command is faster than both the DROP and the DELETE command.
```

3. 对于表的各种修改 ALTER COLUMN CHANGE COLUMN MODIFY COLUMN
```sql
-- 添加新的列
-- MySQL allows you to add the new column as the first column of the table by specifying the FIRST keyword. It also allows you to add the new column after an existing column using the AFTER existing_column clause. If you don’t explicitly specify the position of the new column, MySQL will add it as the last column
ALTER TABLE table
ADD [COLUMN] column_name_1 column_1_definition [FIRST|AFTER existing_column],
ADD [COLUMN] column_name_2 column_2_definition [FIRST|AFTER existing_column],
...;
-- 添加到列的最前面
ALTER TABLE test ADD COLUMN age TINYINT NOT NULL DEFAULT 0 COMMENT '年龄' FIRST;
-- 添加到指定列的后面
ALTER TABLE test ADD COLUMN `age` TINYINT NOT NULL DEFAULT 0 COMMENT '年龄' AFTER `name`;
-- 删除一列
ALTER TABLE test DROP COLUMN age;
---------------------------------------------------------------------
-- CHANGE Used to rename a column, change its datatype, or move it within the schema.
-- MODIFY Used to do everything CHANGE COLUMN can, but without renaming the column
-- 修改列名
ALTER TABLE test CHANGE `test_date` `test_date_for_change` DATE;
-- 修改属性BY CHANGE
ALTER TABLE test CHANGE `test_date` `test_date` DATE COMMENT '测试时间';
-- 修改属性BY MODIFY
ALTER TABLE test MODIFY `test_datetime` DATETIME(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '测试datetime';
-- set the default value for a column
ALTER TABLE test ALTER COLUMN `age` SET DEFAULT 18;
-- remove the default value for a column
ALTER TABLE test ALTER COLUMN `age` DROP DEFAULT;
-- rename table
ALTER TABLE `test_rename` RENAME TO `test`;
```
4. 对于表的查看
```sql
-- 下面两个执行结果一样一样的
SHOW COLUMNS FROM test;
DESC test;
```

5. 对于表数据的插入
```sql
-- insert multiple rows
INSERT INTO table(c1,c2,...)
VALUES 
   (v11,v12,...),
   (v21,v22,...),
    ...
   (vnn,vn2,...);
------------------------------
-- In theory, you can insert any number of rows using a single INSERT statement. However, when MySQL server receives the INSERT statement whose size is bigger than max_allowed_packet, it will issue a packet too large error and terminates the connection.
SHOW VARIABLES LIKE 'max_allowed_packet';-- 4194304
--  set a new value for the max_allowed_packet variable
SET GLOBAL max_allowed_packet=size;
-- where size is an integer that represents the number the maximum allowed packet size in bytes.
-- Note that the max_allowed_packet has no influence on the INSERT INTO .. SELECT statement. The INSERT INTO .. SELECT statement can insert as many rows as you want.
------------------------------
INSERT INTO table_name(column_list)
SELECT 
   select_list 
FROM 
   another_table
WHERE
   condition;
------------------------------
-- update data if a duplicate in the UNIQUE index or PRIMARY KEY error occurs when you insert a row into a table.
INSERT INTO table (column_list)
VALUES (value_list)
ON DUPLICATE KEY UPDATE
   c1 = v1, 
   c2 = v2,
   ...;
------------------------------
-- When you use the INSERT statement, if an error occurs during the processing,the rows with invalid data that cause the error are ignored and the rows with valid data are inserted into the table
INSERT IGNORE INTO table(column_list)
VALUES( value_list),
      ( value_list),
      ...
```

6. 对于表索引的添加
```sql
-- 下面两个执行结果一样一样的
SHOW COLUMNS FROM test;
DESC test;
```
##### 好玩的

```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍

##### 参考
- https://www.tutorialspoint.com/lua/index.htm


