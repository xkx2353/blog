---

title: 攻克MySQL

date: 2020-09-05 13:11:07

---
##### 数据库基本介绍
> 先没有系统的记录，就是想起来哪些，然后记录下来，后续进行系统一点
> 全部测试于MySQL V5.6.23

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
7. 一些特殊的查询用法
```sql
-- GROUP BY age WITH ROLLUP 汇总之后的记录进行再次汇总
SELECT COALESCE(age, '总人数'),age,COUNT(*) FROM test GROUP BY age WITH ROLLUP;
```
8. 表连接
```sql
-- 交叉连接/笛卡尔积/cross join
SELECT * FROM test CROSS JOIN test2 CROSS JOIN test3;
SELECT * FROM test,test2,test3;
-- 内连接 选择出几个表中同时满足条件的记录
SELECT * FROM test
INNER JOIN test2 t ON test.cert_no = t.cert_no;
SELECT * FROM test
JOIN test2 t ON test.cert_no = t.cert_no;
SELECT * FROM test,test2 WHERE test.cert_no = test2.cert_no;
-- 自连接
SELECT * FROM test INNER JOIN test ON test.id = test.c_id;
-- 外连接 匹配的和不匹配的都有可能选出来
-- 左外连接 以左表为准，左表的记录都会展示出来，右表里面的匹配的会展示出来
SELECT * from test LEFT OUTER JOIN test2 t ON test.cert_no = t.cert_no;
SELECT * from test LEFT JOIN test2 t ON test.cert_no = t.cert_no
-- 查询在test中，不在test2中
SELECT * from test LEFT JOIN test2 t ON test.name = t.name WHERE t.name IS NULL;

-- 右外连接 以右表为准，右表的记录都会展示出来，左表里面的匹配的会展示出来

-- 全连接/查询在test中和在test2中的所有
SELECT * from test LEFT JOIN  test2 t ON test.name = t.name 
UNION
SELECT * FROM test2 LEFT JOIN test t2 ON test2.name = t2.name;

-- 查询只在test中和只在test2中
SELECT * from test LEFT JOIN  test2 t ON test.name = t.name WHERE t.name IS NULL
UNION 
SELECT * FROM test2 LEFT JOIN test t2 ON test2.name = t2.name WHERE t2.name IS NULL;




------------------------
-- UNION 和 UNION ALL 的区别

-- UNION:only keeps unique records
-- UNION first performs a sorting operation and eliminates of the records that are duplicated across all columns before finally returning the combined data set
-- UNION ALL:keeps all records,including duplicates

```

##### 一些函数使用
```sql
-- Return the first non-null value in a list:
SELECT COALESCE(NULL, NULL, NULL, 'W3Schools.com', NULL, 'Example.com');
-- W3Schools.com
-------------------------------------------------------------------------
-- 关于 CHAR_LENGTH() LENGTH();
-- MySQL CHAR_LENGTH() returns the length (how many characters are there) of a given string. The function simply counts the number characters and ignore whether the character(s) are single-byte or multi-byte. Therefore a string containing three 2-byte characters, LENGTH() function will return 6, whereas CHAR_LENGTH() function will returns 3

SELECT CHAR_LENGTH('test string'); -- 11
SELECT LENGTH('test string'); -- 11

SELECT CHAR_LENGTH('你好'); -- 2
SELECT LENGTH('你好'); -- 6
-------------------------------------------------------------------------
SELECT TIMESTAMPDIFF(DAY,'2012-10-01','2013-01-13');
-------------------------------------------------------------------------
SELECT CHAR(67,72,65,82);  -- CHAR
```

##### 数据类型
```sql
-- 分为三大类：数值类型；日期时间类型；字符串类型；
-- decimal在MySQL内部中以字符串的形式存在，比浮点数更为准确，适合用来表示精度特别高的数据。用（M,D）来表示，M表示整数和小数一共的位数，D表示小数位数，如果不写，会按照decimal(10,0)来处理，超过会报错。
-- 位类型 BIT(M),表示存放多多位二进制数，M的范围是1-64，默认1位，插入的时候会首先把数据转换成为二进制数
```
<center><img src="https://azou.tech/blog/static/image/char_varchar_difference.png" ></center>
<center>From here <a> https://www.tutorialspoint.com/What-is-the-difference-between-CHAR-and-VARCHAR-in-MySQL</a> </center>

##### 比较运算符
```sql
-- 记录不太清楚的点 NULL=NULL的比较结果是NULL，也就是NULL不能通过=来进行比较
-- NULL<=>NULL的比较结果是1，<=> 称为 NULL-safe 的等于运算符

```
##### 不同的存储引擎
> 常见的存储引擎：MyISAM、InnoDB、MERGE、MEMORY(HEAP)、BDB(BerkeleyDB)等
> 每种引擎技术使用不同的存储机制、索引技巧、锁定水平并且最终提供广泛的不同的功能和能力。如何存储数据；如何为存储的数据建立索引；如何更新、查询数据 等技术的实现方法。然后不同的数据库或者表可以选择不同的引擎来存储数据
> 参考：https://juejin.im/post/6844903795281887245

- InnoDB 
	- 事务
	- 行级锁
	- 外键 不过不喜欢用
	- 适合处理多重并发的更新请求
	- 关于Hash索引的支持：https://blog.csdn.net/doctor_who2004/article/details/77414742
	
- MyISAM 独立于操作系统，我们建立一个MyISAM引擎的表时，就会在本地磁盘上建立三个文件，文件名就是表名，demo.frm，存储表定义；demo.MYD，存储数据；demo.MYI，存储索引。
	- 筛选大量数据时非常迅速，不是很清楚
	- 并发插入特性允许同时选择和插入数据，不是很清楚

- MEMORY 出发点是速度，为得到最快的响应时间，采用的逻辑存储介质是系统内存。
	- 支持散列索引和B树索引

- CSV 出发点是速度，为得到最快的响应时间，采用的逻辑存储介质是系统内存。
	- 所有列必须强制指定 NOT NULL

- ARCHIVE 归档， 仅仅支持最基本的插入和查询

- BLACKHOLE 任何写入到此引擎的数据均会被丢弃掉， 不做实际存储；Select语句的内容永远是空，和Linux中的 /dev/null 文件完成的作用完全一致
	- 不存储数据，但是会记录Binlog
	- 可以提供一些检测的场景使用 
	- 验证语法 验证dump file语法的正确性
	- 检测负载 以使用blackhole引擎来检测binlog功能所需要的额外负载
	- 检测性能 由于blackhole性能损耗极小，可以用来检测除了存储引擎这个功能点之外的其他MySQL功能点的性能。

##### 事务
> MyISAM 不支持事务，InnoDB支持事务，所以所有关于事务， 隔离级别，排它锁， 共享锁， MVCC (当前读 VS 快照读)， select .. for update (排它锁)， select .. lock in share mode(共享锁) 都是针对InnoDB， 也是Innodb拥有事务特性的所有描述，也是Innodb比myisam好的地方。

- 事务拥有四个重要的特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability），习惯称之为 ACID 特性
- 原子性 事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。
- 一致性 指事务将数据库从一种状态转变为另一种一致的的状态。
- 隔离性 在并发环境中，当不同的事务同时操纵相同的数据时，每个事务都有各自的完整数据空间。由并发事务所做的修改必须与任何其他并发事务所做的修改隔离。锁机制来保证。
- 持久性 事务一旦提交，则其结果就是永久性的。即使发生宕机的故障，数据库也能将数据恢复。

> SavePoint

##### 隔离级别和锁
> 不同的隔离级别对事务的处理能力会有不同程度的影响
> 在 InnoDB 中，默认为 REPEATABLE 级别，但是我们生产上一般设置READ COMMITTED
> READ COMMITTED 能够避免脏读取，而且具有较好的并发性能。尽管它会导致不可重复读、幻读和第二类丢失更新这些并发问题，在可能出现这类问题的个别场合，可以由应用程序采用悲观锁或乐观锁来控制。

- READ UNCOMMITTED
	- 带来问题：脏读 ，幻读，不可重复读取
- READ COMMITTED 
	- 带来问题：幻读，不可重复读取
- REPEATABLE READ
	- 带来问题：幻读，BUT InnoDB 中使用一种被称为 next-key locking 的策略来避免幻读（phantom）现象的产生
- SERIALIZABLE
	- 带来问题：事务只能一个接着一个地执行，不能并发执行。

>脏读、幻读、不可重复读的概念

- 脏读 指一个事务中访问到了另外一个事务未提交的数据
- 幻读 一个事务读取2次，得到的记录条数不一致。
- 不可重复读 一个事务读取同一条记录2次，得到的结果不一致

>隔离级别的实现

- READ UNCOMMITTED 完全不加锁
- SERIALIZABLE 读的时候加共享锁，其他事务可以并发读，但是不能写，写的时候加排他锁，其他事务既不能读也不能写
- REPEATABLE READ 采用MVCC(Multi Versioning Concurrency Control)方式。一条记录会有多个版本，也就是数据会有多个快照。可重复读会在事务开始的时候生成一个全局数据的当前快照
- READ COMMITTED 每次执行语句都重新生成一次快照

> 当前读和快照读
> 

- 当前读 读取的是最新版本，update、delete、insert、select…lock in share mode、select…for update都是锁定读。
通过加record lock和gap lock间隙锁来实现，也就是next-key lock。使用next-key lock优势是获取实时数据，但是需要加锁。

- 快照读  基于MVCC 通过undo log存储数据快照
	- READ COMMITTED 每次SELECT都会生成一个快照读，每个快照都是最新的，所以在当前事务中每次都可以看到其他事务提交的更改。
	
	- REPEATABLE READ 开启事务后第一个SELECTt语句才是快照读的地方，而不是一开启事务就快照读，只有当前事务对数据进行更新的时候才会更新快照，在第一次SELECT之前其他事务提交的数据是可以看到的，在SELECT之后其他事务提交的数据在当前事务是看不到的。
	
> FOR UPDATE
> 

`select * from test for update` 可以锁表/锁行。应尽量使用锁行

```sql

-- 明确指定主键，有数据，row lock
SELECT * FROM test WHERE id= 3 FOR UPDATE;
-- 明确指定主键，无此数据，会产生记录锁和间隙锁
SELECT * FROM test WHERE id= -1  FOR UPDATE;
-- 无主键，table lock
SELECT * FROM test WHERE name='xkx' FOR UPDATE;
-- 主键不明确，table lock
SELECT * FROM test WHERE id != 3 FOR UPDATE;
-- 主键不明确，table lock
SELECT * FROM test WHERE id LIKE '%3%' FOR UPDATE;

```
> 锁
> 

1. InnoDB 行锁是通过给索引上的索引项加锁来实现的，InnoDB 这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB 才使用行级锁，否则，InnoDB 将使用表锁！
2. Mysql 为了满足事务的隔离性，必须在 commit 才释放锁。
3. 在普通索引列上，不管是何种查询，只要加锁，都会产生间隙锁，这跟唯一索引不一样；
4. 在普通索引跟唯一索引中，数据间隙的分析，数据行是优先根据普通索引排序，再根据唯一索引排序。
5. 间隙锁会封锁该条记录相邻两个键之间的空白区域，防止其它事务在这个区域内插入、修改、删除数据，这是为了防止出现 幻读 现象
6. 记录锁、间隙锁、临键锁，都属于排它锁
7. 共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到最新数据。
8. 根据非唯一索引 对记录行进行 UPDATE \ FOR UPDATE \ LOCK IN SHARE MODE 操作时，InnoDB 会获取该记录行的 临键锁 ，并同时获取该记录行下一个区间的间隙锁

##### 好玩的

```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍

##### 参考
- https://www.tutorialspoint.com/lua/index.htm
- https://www.zsythink.net/archives/1105


