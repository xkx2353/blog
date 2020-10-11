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
- 关于日期时间类型：DATE，DATESTAMP，TIMESTAMP 的区别？

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

- TRUNCATE DROP DELETE区别

```latex
[DELETE] is a DML;It is comparatively slower than TRUNCATE cmd ;We can use the “ROLLBACK” command to restore the tuple.
[DROP] is a DDL;It is use to drop the whole table,drop the whole structure .Removes the named elements of the schema;Can’t restore the table by using the “ROLLBACK” command
[TRUNCATE] is a DDL; It is use to delete all the rows of a relation (table) ;It is comparatively faster than delete command as it deletes all the rows fastly. Can’t rollback the data after using the this command
------------
The TRUNCATE command is faster than both the DROP and the DELETE command.
```

##### 语句的执行顺序



##### 常用的语句

- 对于表的各种修改 ALTER COLUMN CHANGE COLUMN MODIFY COLUMN

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
- 对于表的查看

```sql
-- 下面两个执行结果一样一样的
SHOW COLUMNS FROM test;
DESC test;
```

- 对于表数据的插入

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

- 索引维护

> 生活中随处可见索引的例子，如火车站的车次表、图书的目录等。它们的原理都是一样的，通过不断的缩小想要获得数据的范围来筛选出最终想要的结果，同时把随机的事件变成顺序的事件，也就是我们总是通过同一种查找方式来锁定数据。

```sql
-- 表索引区分度，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录
SELECT count(DISTINCT col)/count(*);

-- 下面两个执行结果一样一样的
SHOW COLUMNS FROM test;
DESC test;
```

- 一些特殊的查询用法

```sql
-- GROUP BY age WITH ROLLUP 汇总之后的记录进行再次汇总
SELECT COALESCE(age, '总人数'),age,COUNT(*) FROM test GROUP BY age WITH ROLLUP;
```

- 表连接
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

-- JOIN的时候ON和USING的使用 Specifically, any columns mentioned in the USING list will appear only once
-- 两个表不同的字段名字，两个表列都会显示出来
SELECT * FROM world.City JOIN world.Country ON (City.CountryCode = Country.Code) WHERE ...
-- 两个表相同的字段名字,相同的列只会出现一次
SELECT ... FROM film JOIN film_actor USING (film_id) WHERE ...
```

- 函数使用
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
```mysql
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
> MyISAM 不支持事务，InnoDB支持事务，所以所有关于事务， 隔离级别，排它锁， 共享锁， MVCC (当前读 && 快照读)， select .. for update (排它锁)， select .. lock in share mode(共享锁) 都是针对InnoDB， 也是Innodb拥有事务特性的所有描述，也是Innodb比myisam好的地方。

- 事务拥有四个重要的特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability），习惯称之为 ACID 特性
- 原子性 事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。
- 一致性 指事务将数据库从一种状态转变为另一种一致的的状态。
- 隔离性 在并发环境中，当不同的事务同时操纵相同的数据时，每个事务都有各自的完整数据空间。由并发事务所做的修改必须与任何其他并发事务所做的修改隔离。锁机制来保证。
- 持久性 事务一旦提交，则其结果就是永久性的。即使发生宕机的故障，数据库也能将数据恢复。

> SavePoint
> 

> 如何避免长事务对业务的影响
> 

##### 隔离级别和锁
> 不同的隔离级别对事务的处理能力会有不同程度的影响
> 在 InnoDB 中，默认为 REPEATABLE 级别，但是我们生产上一般设置READ COMMITTED
> READ COMMITTED 能够避免脏读取，而且具有较好的并发性能。尽管它会导致不可重复读、幻读和第二类丢失更新这些并发问题，在可能出现这类问题的个别场合，可以由应用程序采用悲观锁或乐观锁来控制。

- READ UNCOMMITTED
	- 带来问题：脏读 ，幻读，不可重复读取
- READ COMMITTED 
	- 带来问题：幻读，不可重复读取
- REPEATABLE READ
	- 带来问题：幻读，BUT InnoDB 中使用一种被称为 next-key locking 的策略来避免幻读（phantom）现象的产生，但是不能完全避免，如果在事务在执行的过程中进行语句需要当前读，就会读到另一个事务提交的最新的数据。
- SERIALIZABLE
	- 带来问题：事务只能一个接着一个地执行，不能并发执行。

>脏读、幻读、不可重复读的概念

- 脏读 指一个事务中访问到了另外一个事务未提交的数据
- 幻读 一个事务读取2次，得到的记录条数不一致。
- 不可重复读 一个事务读取同一条记录2次，得到的结果不一致

>隔离级别的实现

- READ UNCOMMITTED 完全不加锁
- SERIALIZABLE 读的时候加共享锁，其他事务可以并发读，但是不能写，写的时候加排他锁，其他事务既不能读也不能写
- REPEATABLE READ 采用MVCC(Multi Versioning Concurrency Control)方式。一条记录会有多个版本，也就是数据会有多个快照。可重复读会在事务开始「第一条语句执行的时候」的时候生成一个全局数据的当前快照
- READ COMMITTED 每次执行语句都重新生成一次快照

> 当前读和快照读
> 

- 当前读 读取的是最新版本，update、delete、insert、select…lock in share mode、select…for update都是当前读。通过加record lock和gap lock间隙锁来实现，也就是next-key lock。使用next-key lock优势是获取实时数据，但是需要加锁，防止幻读，但是不能彻底解决幻读。

  当一个事务t1开始，在另一个事务t2开始执行插入数据column1之后，t1才开始进行操作，那么column1对于t1可见。

- 快照读  基于MVCC 通过undo log存储数据快照
	- READ COMMITTED 每次SELECT都会生成一个快照读，每个快照都是最新的，所以在当前事务中每次都可以看到其他事务提交的更改。
	
	- REPEATABLE READ 开启事务后第一个SELECT语句才是快照读的地方，而不是一开启事务就快照读，只有当前事务对数据进行更新的时候才会更新快照，在第一次SELECT之前其他事务提交的数据是可以看到的，在SELECT之后其他事务提交的数据在当前事务是看不到的。
	
> FOR UPDATE
> 

`SELECT * FROM test FOR UPDATE;` 可以锁表/锁行。应尽量使用锁行

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
> 参考：https://blog.csdn.net/usagoole/article/details/104761617

1. InnoDB 行锁是通过给索引上的索引项加锁来实现的，InnoDB 这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB 才使用行级锁，否则，InnoDB 将使用表锁！
2. Mysql 为了满足事务的隔离性，在 COMMIT后才释放锁。
3. 在普通索引列上，不管是何种查询，只要加锁，都会产生间隙锁，和唯一索引不一样。
4. 在普通索引跟唯一索引中，数据间隙的分析，数据行是优先根据普通索引排序，再根据唯一索引排序。
5. 间隙锁会封锁该条记录相邻两个键之间的空白区域，防止其它事务在这个区域内插入、修改、删除数据，这是为了防止出现幻读现象。
6. 记录锁、间隙锁、临键锁，都属于排它锁
7. 共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到最新数据。
8. 根据非唯一索引 对记录行进行 UPDATE \ FOR UPDATE \ LOCK IN SHARE MODE 操作时，InnoDB 会获取该记录行的临键锁 ，并同时获取该记录行下一个区间的间隙锁
9. MySql只有在RR的隔离级别下才有GAP Lock和Next-Key Lock
10. 加锁规则
    - 规则1:加锁的基本单位是Next-Key Lock(前开后闭区间)
    - 规则2:查找过程中访问到的对象才会加锁，如果查询使用覆盖索引，并不需要访问主键索引，所以主键索引上没有加任何锁；
    - 优化1:索引上的等值查询，给唯一索引加锁的时候，Next-Key Lock退化为Record Lock（行锁）
    - 优化2:索引上的等值查询，向右遍历时且最后 一个值不满足等值条件的时候，next-key lock退化为Gap Lock（间隙锁）
    - 在读提交隔离级别下还有一个优化，即：语句执行过程中加上的行锁，在语句执行完成后，就要把“不满足条件的行”上的行锁直接释放了，不需要等到事务提交。也就是说，读提交隔离级别下，锁的范围更小，锁的时间更短，这也是不少业务都默认使用读提交隔离级别的原因。
11. 默认情况下，InnoDB工作在RR级别下，并且会以Next-Key Lock的方式对数据行进行加锁，这样可以有效防止幻读的发生。
12. 要分清：表级别的意向共享锁和意向排它锁「意向锁之间是互相兼容的」，表级别的共享锁和排它锁「除了 IS 与 S 兼容外，意向锁会与表级共享锁 /表级排他锁 互斥」，行级别的共享锁和排他锁「意向锁不会与行级的共享 / 排他锁互斥」。
13. InnoDB 支持`多粒度锁（multiple granularity locking）`，它允许`行级锁`与`表级锁`共存，而**意向锁**就是其中的一种`表锁`。**意向共享锁**（intention shared lock, IS）：事务有意向对表中的某些行加**共享锁**（S锁）；**意向排他锁**（intention exclusive lock, IX）：事务有意向对表中的某些行加**排他锁**（X锁）；`意向锁是有数据引擎自己维护的，用户无法手动操作意向锁`，在为数据行加共享 / 排他锁之前，InooDB 会先获取该数据行所在在数据表的对应意向锁。
14. 在 InnoDB 事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释放，而是要等到事务结束时才释放。这个就是两阶段锁协议。所以如果事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。
15. **死锁** 是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。处理策略：一种是，直接进入等待，直到超时。这个超时时间可以通过参数 innodb_lock_wait_timeout ；另一种是，发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数 innodb_deadlock_detect 设置为 on「默认」，表示开启这个逻辑。InnoDB 引擎采取的是 `wait-for graph` 等待图的方法来自动检测死锁，如果发现死锁会自动回滚一个事务。


##### 索引指南

> 生活中随处可见索引的例子，如火车站的车次表、图书的目录等。它们的原理都是一样的，通过不断的缩小想要获得数据的范围来筛选出最终想要的结果，同时把随机的事件变成顺序的事件，也就是我们总是通过同一种查找方式来锁定数据。

N叉树「我习惯叫多叉树」。以 InnoDB 的一个整数字段索引为例，这个 N 差不多是 1200。这棵树高是 4 的时候，就可以存 1200 的 3 次方个值，这已经 17 亿了。考虑到树根的数据块总是在内存中的，一个 10 亿行的表上一个整数字段的索引，查找一个值最多只需要访问 3 次磁盘。其实，树的第二层也有很大概率在内存中，那么访问磁盘的平均次数就更少了。

主键索引的叶子节点存的是整行数据。在 InnoDB 里，主键索引也被称为聚簇索引（clustered index）；非主键索引的叶子节点内容是主键的值。在 InnoDB 里，非主键索引也被称为二级索引（secondary index）

由于每个非主键索引的叶子节点上都是主键的值。如果用身份证号做主键，那么每个二级索引的叶子节点占用约 20 个字节，而如果用整型做主键，则只要 4 个字节，如果是长整型（bigint）则是 8 个字节。**显然，主键长度越小，普通索引的叶子节点就越小，普通索引占用的空间也就越小。**所以，从性能和存储空间方面考量，自增主键往往是更合理的选择。

> N的大小可以调整吗？
> N是由页大小和索引大小决定的

> 索引重建

1. 索引可能因为删除，或者页分裂等原因，导致数据页有空洞，重建索引的过程会创建一个新的索引，把数据按顺序插入，这样页面的利用率最高，也就是索引更紧凑、更省空间。
2. `alter table T engine=InnoDB`
3. 索引的统计信息不对，通过`analyze table t`重新统计索引信息

> 索引下推优化

1. 可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。

> 索引选择异常和处理

1. 使用force index 强制选择一个索引，虽然不是很优雅；
2. 修改语句引导MySQL使用索引，但是最好不要轻易修改语句的语义；
3. 可以新建一个索引或者删除掉误用的索引；

##### 关于Join

> 在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表。

- `select * from t1 straight_join t2 on (t1.a=t2.a);`，先遍历表 t1，然后根据从表 t1 中取出的每行数据中的 a 值，去表 t2 中查找满足条件的记录。在形式上，这个过程就跟我们写程序时的嵌套查询类似，并且可以用上被驱动表的索引，所以我们称之为“Index Nested-Loop Join”，简称 NLJ。驱动表是走全表扫描，而被驱动表是走树搜索。 
- 如果可以使用被驱动表的索引，join 语句还是有其优势的；不能使用被驱动表的索引，只能使用 Block Nested-Loop Join 算法，可能会因为 join_buffer 不够大，需要对被驱动表做多次全表扫描。这样的语句就尽量不要使用；在使用 join 的时候，应该让小表做驱动表。
- 如果你的 join 语句很慢，就把 join_buffer_size 改大。
- 在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表。
- Multi-Range Read 优化 (MRR)。这个优化的主要目的是尽量使用顺序读盘。MRR 能够提升性能的核心在于，这条查询语句在索引 a 「普通索引」上做的是一个范围查询（也就是说，这是一个多值查询），可以得到足够多的主键 id。这样通过排序以后，再去主键索引查数据，才能体现出“顺序性”的优势。
- BKA「Batched Key Access」 优化是 MySQL 已经内置支持的，建议你默认使用；
BNL 算法效率低，建议你都尽量转成 BKA 算法。优化的方向就是给被驱动表的关联字段加上索引；
基于临时表的改进方案，对于能够提前过滤出小数据的 join 语句来说，效果还是很好的；

- 多表关联，整体的思路就是，尽量让每一次参与 join 的驱动表的数据集，越小越好，因为这样我们的驱动表就会越小，要在相应的关联字段建立索引。

##### 关于执行计划

> 执行计划的字段意义，优化语句的时候怎么优化
> 

- rows 预计扫描行数，注意如果使用普通索引，需要算上回表的查询的行数的代价
- Extra 
  - Using where
  - Using index condition
  - Using MRR
  - Using temporary 使用了临时表
  - Using filesort 表示需要排序
  - Using index 使用了覆盖索引
  - Using join buffer（Block Nested Loop）


##### 千万级表优化

执行计划很好，但是就是查询很慢，是不是应该强制使用索引呢？

```mysql
-- 分析是否是取值limit太大，超过count的1/3，Innodb引擎不使用索引
SELECT *
FROM database.table
FORCE index(create_time)
WHERE create_time >= 1508360400
  AND create_time <= 1508444806
ORDER BY create_time ASC
LIMIT 4000,
      1000;
```

相同的sql在不同的从库执行，一个未使用索引，分析是索引文件或者表的碎片导致，是表的碎片问题导致产生的执行计划不正常

```sql
-- 方案1：执行OPTIMIZE TABLE修复碎片或者执行ALTER TABLE foo ENGINE=InnoDB，以上两种操作都会锁表，对于数据量大，且业务高峰期执行需要慎重
-- 方案2：强制索引，也就是FORCE index create_time，强制mysql 引擎使用索引，这里需要注意一下，当使用强制索引时，存储引擎会检查强制索引是否可用，如果不可用，还需要扫描表来判断那种执行计划，
```

关于分页优化

```mysql
-- 日常分页SQL语句,扫描满足条件的10020行，扔掉前面的10000行，返回最后的20行
SELECT id,name,content FROM users ORDER BY id ASC LIMIT 100000,20;
-- 如果记录了上次的最大ID，扫描20行
SELECT id,NAME,content FROM users WHERE id>100073 ORDER BY id ASC LIMIT 20;
```

慢语句

在 MySQL 中，会引发性能问题的慢查询，大体有以下三种可能：索引没有设计好；SQL 语句没写好；MySQL 选错了索引。

   


##### 一些技巧

-  查询语句的执行顺序：
from:需要从哪个数据表检索数据 
where:过滤表中数据的条件 
group by:如何将上面过滤出的数据分组 
having:对上面已经分组的数据进行过滤的条件 
select:查看结果集中的哪个列，或列的计算结果 
order by :按照什么样的顺序来查看返回的数据 

- 行转列如何转「参考平凡希：https://www.cnblogs.com/xiaoxi/p/7151433.html」
```mysql
-- case...when....then
SELECT userid,
SUM(CASE `subject` WHEN '语文' THEN score ELSE 0 END) as '语文',
SUM(CASE `subject` WHEN '数学' THEN score ELSE 0 END) as '数学',
SUM(CASE `subject` WHEN '英语' THEN score ELSE 0 END) as '英语',
SUM(CASE `subject` WHEN '政治' THEN score ELSE 0 END) as '政治' 
FROM tb_score 
GROUP BY userid
-- if
SELECT userid,
SUM(IF(`subject`='语文',score,0)) as '语文',
SUM(IF(`subject`='数学',score,0)) as '数学',
SUM(IF(`subject`='英语',score,0)) as '英语',
SUM(IF(`subject`='政治',score,0)) as '政治' 
FROM tb_score 
GROUP BY userid
-- 行转列后进行汇总
SELECT IFNULL(userid,'total') AS userid,
SUM(IF(`subject`='语文',score,0)) AS 语文,
SUM(IF(`subject`='数学',score,0)) AS 数学,
SUM(IF(`subject`='英语',score,0)) AS 英语,
SUM(IF(`subject`='政治',score,0)) AS 政治,
SUM(IF(`subject`='total',score,0)) AS total
FROM(
    SELECT userid,IFNULL(`subject`,'total') AS `subject`,SUM(score) AS score
    FROM tb_score
    GROUP BY userid,`subject`
    WITH ROLLUP
    HAVING userid IS NOT NULL
)AS A 
GROUP BY userid
WITH ROLLUP;
-- way2
SELECT userid,
SUM(IF(`subject`='语文',score,0)) AS 语文,
SUM(IF(`subject`='数学',score,0)) AS 数学,
SUM(IF(`subject`='英语',score,0)) AS 英语,
SUM(IF(`subject`='政治',score,0)) AS 政治,
SUM(score) AS TOTAL 
FROM tb_score
GROUP BY userid
UNION
SELECT 'TOTAL',SUM(IF(`subject`='语文',score,0)) AS 语文,
SUM(IF(`subject`='数学',score,0)) AS 数学,
SUM(IF(`subject`='英语',score,0)) AS 英语,
SUM(IF(`subject`='政治',score,0)) AS 政治,
SUM(score) FROM tb_score
-- way3
SELECT IFNULL(userid,'TOTAL') AS userid,
SUM(IF(`subject`='语文',score,0)) AS 语文,
SUM(IF(`subject`='数学',score,0)) AS 数学,
SUM(IF(`subject`='英语',score,0)) AS 英语,
SUM(IF(`subject`='政治',score,0)) AS 政治,
SUM(score) AS TOTAL 
FROM tb_score
GROUP BY userid WITH ROLLUP;
-- 很好的建属于同一分组的多个行转化为一个列
SELECT userid,GROUP_CONCAT(`subject`,":",score)AS 成绩 FROM tb_score
GROUP BY userid
```

> 从MySQL小册学习到的
> 

- 一条语句的执行流程：「连接器--->查询缓存--->分析器--->优化器--->执行器」（Server层）---->引擎层。Server 层包括连接器、查询缓存、分析器、优化器、执行器等，涵盖 MySQL 的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

- 关于binlog和redolog，redolog是InnoDB 引擎特有的日志，所以InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为crash-safe。binlog是Server 层的日志，所有引擎都可以使用。redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。Binlog有两种模式，statement 格式的话是记sql语句， row格式会记录行的内容，记两条，更新前和更新后都有。

- 执行器先找引擎取 ID=2 这一行。ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。执行器生成这个操作的 binlog，并把 binlog 写入磁盘。执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。

- redo log 用于保证 crash-safe 能力。innodb_flush_log_at_trx_commit 这个参数设置成 1 的时候，表示每次事务的 redo log 都直接持久化到磁盘。这个参数建议设置成 1，这样可以保证 MySQL 异常重启之后数据不丢失。sync_binlog 这个参数设置成 1 的时候，表示每次事务的 binlog 都持久化到磁盘。这个参数建议你设置成 1，这样可以保证 MySQL 异常重启之后 binlog 不丢失。

- 回滚记录：实际上每条记录在更新的时候都会同时记录一条回滚操作。记录上的最新值，通过回滚操作，都可以得到前一个状态的值。

- 尽量不要使用长事务：长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间；长事务还占用锁资源，也可能拖垮整个库。

- InnoDB 是索引组织表，主键索引树的叶子节点是数据，而普通索引树的叶子节点是主键值。所以，普通索引树比主键索引树小很多。对于 count(`*`) 这样的操作，遍历哪个索引树得到的结果逻辑上都是一样的。因此，MySQL 优化器会找到最小的那棵树来遍历。在保证逻辑正确的前提下，尽量减少扫描的数据量，是数据库系统设计的通用法则之一。

- 关于排序，如果 MySQL 实在是担心排序内存太小，会影响排序效率，才会采用 rowid 排序算法，这样排序过程中一次可以排序更多行，但是需要再回到原表去取数据。如果 MySQL 认为内存足够大，会优先选择全字段排序，把需要的字段都放到 sort_buffer 中，这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据。这也就体现了 MySQL 的一个设计思想：如果内存够，就要多利用内存，尽量减少磁盘访问。其实，并不是所有的 order by 语句，都需要排序操作的。从上面分析的执行过程，我们可以看到，MySQL 之所以需要生成临时表，并且在临时表上做排序操作，**其原因是原来的数据都是无序的。** 所以可以通过建立联合索引来提高效率「尽可能使用覆盖索引」。

- MySQL 里经常说到的 WAL 技术，WAL 的全称是 Write-Ahead Logging，它的关键点就是先写日志，再写磁盘，具体来说，当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log里面，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面。

- SELECT * FROM tb ORDER BY rand() LIMIT 1; 随机获取第一条。但是数据量大了后就可能会是有临时表和文件排序导致效率下降，order by rand() 使用了内存临时表，内存临时表排序的时候使用了 rowid 排序方法。`取得整个表的行数，并记为 C。取得 Y = floor(C * rand())。 floor 函数在这里的作用，就是取整数部分。再用 limit Y,1 取得一行。`实际应用的过程中，比较规范的用法就是：尽量将业务逻辑写在业务代码中，让数据库只做“读写数据”的事情。因此，这类方法的应用还是比较广泛的。

- 对于有主键的 InnoDB 表来说，这个 rowid 就是主键 ID；对于没有主键的 InnoDB 表来说，这个 rowid 就是由系统生成的；

- 关于binlog磁盘写入。 write，指的就是指把日志写入到文件系统的 page cache，并没有把数据持久化到磁盘，所以速度比较快。 fsync，才是将数据持久化到磁盘的操作。一般情况下，我们认为 fsync 才占磁盘的 IOPS。
  write 和 fsync 的时机，是由参数 sync_binlog 控制的：
  sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync；
  sync_binlog=1 的时候，表示每次提交事务都会执行 fsync；
  sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。在出现 IO 瓶颈的场景里，将 sync_binlog 设置成一个比较大的值，可以提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0，比较常见的是将其设置为 100~1000 中的某个数值。将 sync_binlog 设置为 N，对应的风险是：如果主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。

- 关于redolog磁盘写入。为了控制 redo log 的写入策略，InnoDB 提供了innodb_flush_log_at_trx_commit 参数，它有三种可能取值：
  设置为 0 的时候，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中 ;
  设置为 1 的时候，表示每次事务提交时都将 redo log 直接持久化到磁盘；
  设置为 2 的时候，表示每次事务提交时都只是把 redo log 写到 page cache。
  InnoDB 有一个后台线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache，然后调用 fsync 持久化到磁盘。
  redo log buffer 占用的空间即将达到 innodb_log_buffer_size 一半的时候，后台线程会主动写盘。注意，由于这个事务并没有提交，所以这个写盘动作只是 write，而没有调用 fsync，也就是只留在了文件系统的 page cache。
  并行的事务提交的时候，顺带将这个事务的 redo log buffer 持久化到磁盘。假设一个事务 A 执行到一半，已经写了一些 redo log 到 buffer 中，这时候有另外一个线程的事务 B 提交，如果 innodb_flush_log_at_trx_commit 设置的是 1，那么按照这个参数的逻辑，事务 B 要把 redo log buffer 里的日志全部持久化到磁盘。这时候，就会带上事务 A 在 redo log buffer 里的日志一起持久化到磁盘。
  把 innodb_flush_log_at_trx_commit 设置成 1，那么 redo log 在 prepare 阶段就要持久化一次，因为有一个崩溃恢复逻辑是要依赖于 prepare 的 redo log，再加上 binlog 来恢复的。每秒一次后台轮询刷盘，再加上崩溃恢复这个逻辑，InnoDB 就认为 redo log 在 commit 的时候就不需要 fsync 了，只会 write 到文件系统的 page cache 中就够了。

- 通常我们说 MySQL 的“双 1”配置，指的就是 sync_binlog 和 innodb_flush_log_at_trx_commit 都设置成 1。也就是说，一个事务完整提交前，需要等待两次刷盘，一次是 redo log（prepare 阶段），一次是 binlog。

- 什么时候会把线上生产库设置成“非双 1”？业务高峰期。一般如果有预知的高峰期，DBA 会有预案，把主库设置成“非双 1”。备库延迟，为了让备库尽快赶上主库。@永恒记忆和 @Second Sight 提到了这个场景。用备份恢复主库的副本，应用 binlog 的过程，这个跟上一种场景类似。批量导入数据的时候。一般情况下，把生产库改成“非双 1”配置，是设置 innodb_flush_logs_at_trx_commit=2、sync_binlog=1000。

- 提升IO性能：设置 binlog_group_commit_sync_delay 和 binlog_group_commit_sync_no_delay_count 参数，减少 binlog 的写盘次数。这个方法是基于“额外的故意等待”来实现的，因此可能会增加语句的响应时间，但没有丢失数据的风险。
  将 sync_binlog 设置为大于 1 的值（比较常见是 100~1000）。这样做的风险是，主机掉电时会丢 binlog 日志。
  将 innodb_flush_log_at_trx_commit 设置为 2。这样做的风险是，主机掉电的时候会丢数据。

- 实际上数据库的 crash-safe 保证的是：
  如果客户端收到事务成功的消息，事务就一定持久化了；
  如果客户端收到事务失败（比如主键冲突、回滚等）的消息，事务就一定失败了；
  如果客户端收到“执行异常”的消息，应用需要重连后通过查询当前状态来继续后续的逻辑。此时数据库只需要保证内部（数据和日志之间，主库和备库之间）一致就可以了。
  
- 当 binlog_format=statement 时，binlog 里面记录的就是 SQL 语句的原文，由于 statement 格式下，记录到 binlog 里的是语句原文，因此可能会出现这样一种情况：在主库执行这条 SQL 语句的时候，用的是索引 a；而在备库执行这条 SQL 语句的时候，却使用了索引 t_modified。因此，MySQL 认为这样写是有风险的。当 binlog_format 使用 row 格式的时候，binlog 里面记录了真实删除行的主键 id，这样 binlog 传到备库去的时候，就肯定会删除 id=4 的行，不会有主备删除不同行的问题。 mixed 格式的 binlog意思是，MySQL 自己会判断这条 SQL 语句是否可能引起主备不一致，如果有可能，就用 row 格式，否则就用 statement 格式。现在越来越多的场景要求把 MySQL 的 binlog 格式设置成 row。这么做的理由有很多，直接好处：**恢复数据**。

- mysqlbinlog 解析出日志，然后把里面的 statement 语句直接拷贝出来执行，这个方法是有风险的。因为有些语句的执行结果是依赖于上下文命令的，直接执行的结果很可能是错误的。正确做法：`mysqlbinlog master.000001  --start-position=2738 --stop-position=2973 | mysql -h127.0.0.1 -P13000 -u$user -p$pwd;`，意思是将 master.000001 文件里面从第 2738 字节到第 2973 字节中间这段内容解析出来，放到 MySQL 去执行。

- 双 M 结构，即：节点 A 和 B 之间总是互为主备关系。这样在切换的时候就不用再修改主备关系。

- 如果节点 A 同时是节点 B 的备库，相当于又把节点 B 新生成的 binlog 拿过来执行了一次，然后节点 A 和 B 间，会不断地循环执行这个更新语句，也就是循环复制了。这个要怎么解决呢？MySQL 在 binlog 中记录了这个命令第一次执行时所在实例的 server id，规定两个库的 server id 必须不同，如果相同，则它们之间不能设定为主备关系；一个备库接到 binlog 并在重放的过程中，生成与原 binlog 的 server id 相同的新的 binlog；每个库在收到从自己的主库发过来的日志后，先判断 server id，如果跟自己的相同，表示这个日志是自己生成的，就直接丢弃这个日志。按照这个逻辑，如果我们设置了双 M 结构，日志的执行流就会变成这样：从节点 A 更新的事务，binlog 里面记的都是 A 的 server id；传到节点 B 执行一次以后，节点 B 生成的 binlog 的 server id 也是 A 的 server id；再传回给节点 A，A 判断到这个 server id 与自己的相同，就不会再处理这个日志。所以，死循环在这里就断掉了。

- 备库 B 跟主库 A 之间维持了一个长连接。主库 A 内部有一个线程，专门用于服务备库 B 的这个长连接。一个事务日志同步的完整过程是这样的：

  1. 在备库 B 上通过 change master 命令，设置主库 A 的 IP、端口、用户名、密码，以及要从哪个位置开始请求 binlog，这个位置包含文件名和日志偏移量。

  2. 在备库 B 上执行 start slave 命令，这时候备库会启动两个线程，就是图中的 io_thread 和 sql_thread。其中 io_thread 负责与主库建立连接。

  3. 主库 A 校验完用户名、密码后，开始按照备库 B 传过来的位置，从本地读取 binlog，发给 B。

  4. 备库 B 拿到 binlog 后，写到本地文件，称为中转日志（relay log）。relaylog 是应用后就丢弃了的，binlog会保存着。

  5. sql_thread 读取中转日志，解析出日志里的命令，并执行。
  
- 可以在备库上执行 show slave status 命令，它的返回结果里面会显示 seconds_behind_master，用于表示当前备库延迟了多少秒。网络正常情况下，主备延迟的主要来源是备库接收完 binlog 和执行完这个事务之间的时间差。其实就是备库消费中转日志（relay log）的速度，比主库生产 binlog 的速度要慢。

  - 由于主库直接影响业务，大家使用起来会比较克制，反而忽视了备库的压力控制。结果就是，备库上的查询耗费了大量的 CPU 资源，影响了同步速度，造成主备延迟。解决方案：一主多从。除了备库外，可以多接几个从库，让这些从库来分担读的压力；通过 binlog 输出到外部系统，比如 Hadoop 这类系统，让外部系统提供统计类查询的能力。
  - 大事务的影响，因为主库上必须等事务执行完成才会写入 binlog，再传给备库。所以，如果一个主库上的语句执行 10 分钟，那这个事务很可能就会导致从库延迟 10 分钟。所以删除大量数据的时候，要控制每个事务删除的数据量，分成多次删除。
  
- 因为主备可能发生切换，备库随时可能变成主库，所以主备库选用相同规格的机器，并且做对称部署，由于主备延迟的存在，所以在主备切换的时候，就相应的有不同的策略：

  - 可靠性优先策略
  - 判断备库 B 现在的 seconds_behind_master，如果小于某个值（比如 5 秒）继续下一步，否则持续重试这一步；
    - 把主库 A 改成只读状态，即把 readonly 设置为 true；
  - 判断备库 B 的 seconds_behind_master 的值，直到这个值变成 0 为止；
    - 把备库 B 改成可读写状态，也就是把 readonly 设置为 false；
    - 把业务请求切到备库 B。
  - 可用性优先策略，主库 A 和备库 B 可能会出现不一致的数据。
  
- 在满足数据可靠性的前提下，MySQL 高可用系统的可用性，是依赖于主备延迟的。延迟的时间越小，在主库故障的时候，服务恢复需要的时间就越短，可用性就越高。

- 内存临时表「sort buffer；join bufffer等」，第一，内存临时表起到了暂存数据的作用，而且计算过程还用上了临时表主键 id 的唯一性约束，实现了 union 的语义。`(select 1000 as f) union (select id from t1 order by id desc limit 2);`会使用到临时表；把上面这个语句中的 union 改成 union all 的话，就没有了“去重”的语义。这样执行的时候，就依次执行子查询，得到的结果直接作为结果集的一部分，发给客户端。因此也就不需要临时表了。内存临时表的大小是有限制的，参数 tmp_table_size 就是控制这个内存大小的，默认是 16M。

- group by的时候，字段尽量有索引，这样就不需要临时表，也不需要排序。

- 如果你的需求并不需要对结果进行排序，那你可以在 SQL 语句末尾增加 order by null

- `select SQL_BIG_RESULT id%100 as m, count(*) as c from t1 group by m;`在 group by 语句中加入 SQL_BIG_RESULT 这个提示（hint），就可以告诉优化器：这个语句涉及的数据量很大，请直接用磁盘临时表。

- 内存表也是支 B-Tree 索引的，在 id 列上创建一个 B-Tree 索引`alter table t1 add index a_btree_index using btree (id);`内存表的优势是速度快，其中的一个原因就是 Memory 引擎支持 hash 索引。当然，更重要的原因是，内存表的所有数据都保存在内存，而内存的读写速度总是比磁盘快。

- 内存表的问题：
  - 锁粒度问题；只支持表锁。因此，一张表只要有更新，就会堵住其他所有在这个表上的读写操作。内存表的锁粒度问题，决定了它在处理并发事务的时候，性能也不会太好。
  - 数据持久化问题。
  
- 主键不连续：唯一键冲突是导致自增主键 id 不连续；事务回滚；批量申请自增 id 的策略「用不完的也会空着」。所以InnoDB只保证了自增 id 是递增的，但不保证是连续的。

- 自增 id 锁并不是一个事务锁，而是每次申请完就马上释放，以便允许别的事务再申请。

- insert … select 是很常见的在两个表之间拷贝数据的方法。你需要注意，在可重复读隔离级别下，这个语句会给 select 的表里扫描到的记录和间隙加读锁。insert 语句如果出现唯一键冲突，会在冲突的唯一值上加共享的 next-key lock(S 锁)

- 两张表中拷贝数据：为了避免对源表加读锁，更稳妥的方案是先将数据写到外部文本文件，然后再写回目标表。

  - 使用 mysqldump 命令将数据导出成一组 INSERT 语句：`mysqldump -h$host -P$port -u$user --add-locks=0 --no-create-info --single-transaction  --set-gtid-purged=OFF db1 t --where="a>900" --result-file=/client_tmp/t.sql`如果你希望生成的文件中一条 INSERT 语句只插入一行数据的话，可以在执行 mysqldump 命令时，加上参数–skip-extended-insert。数据写入到db2`mysql -h127.0.0.1 -P13000  -uroot db2 -e "source /client_tmp/t.sql"`
  - 直接将结果导出成.csv 文件：`select * from db1.t where a>900 into outfile '/server_tmp/t.csv';`，保存在服务端，执行：`load data infile '/server_tmp/t.csv' into table db2.t;`

- id到最大怎么办：
  - 表的自增 id 达到上限后，再申请时它的值就不会改变，进而导致继续插入数据时报主键冲突的错误。
  - row_id 达到上限后，则会归 0 再重新递增，如果出现相同的 row_id，后写的数据会覆盖之前的数据。
  - Xid 只需要不在同一个 binlog 文件中出现重复值即可。虽然理论上会出现重复值，但是概率极小，可以忽略不计。
  - InnoDB 的 max_trx_id 递增值每次 MySQL 重启都会被保存起来。
  - thread_id 是我们使用中最常见的，而且也是处理得最好的一个自增 id 逻辑了。

##### 好玩的

```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍

##### 参考
- https://www.tutorialspoint.com/lua/index.htm
- https://www.zsythink.net/archives/1105
- [MySQL索引原理及慢查询优化](https://www.mtyun.com/library/mysql-index)
