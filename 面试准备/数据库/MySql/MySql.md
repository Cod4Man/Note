# MySql

## 1.  常用命令

### 1.1 常用系统命令

![1621666552007](E:\SoftwareNote\面试准备\数据库\MySql\img\常用系统命令.png)

### 1.2 常用命令

#### 1.2.1 show profile ：查看 sql 的执行周期  

- 查看是否开启

  ```shell
  查看 profile 是否开启：show variables like '%profiling%
  ```

- 开启

  ```shell
  set profiling=1(关闭为0)
  ```

- 查看最近sql执行周期：

```shell
mysql> show profiles;
+----------+------------+-----------------------------------+
| Query_ID | Duration   | Query                             |
+----------+------------+-----------------------------------+
|        1 | 0.00379350 | show variables like '%profiling%' |
|        2 | 0.00012975 | show database                     |
|        3 | 0.00108000 | show databases                    |
|        4 | 0.00054500 | use testl
use test                 |
|        5 | 0.00020575 | use database test                 |
|        6 | 0.00013375 | use database test
select           |
|        7 | 0.00021575 | select test                       |
|        8 | 0.00045325 | use test                          |
|        9 | 0.00035475 | show tables                       |
|       10 | 0.00102425 | select * from tbl_emp             |
+----------+------------+-----------------------------------+
10 rows in set (0.14 sec)
```

#### 1.2.2  reset query cache 清除查询缓存

### 1.3 常用关键字

#### 1.3.1 建表时的四种key

mysql的key和index多少有点令人迷惑，这实际上考察对数据库体系结构的了解的。 

```sql
-- 父表：
CREATE TABLE teacher (
	course varchar(16) NOT NULL,
    teacher char(16) NOT NULL,
	PRIMARY KEY (course)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
-- 子表
CREATE TABLE course (
	id tinyint(3) unsigned NOT NULL AUTO_INCREMENT,
    course varchar(16) NOT NULL,
    `type` varchar(100) NOT NULL, 
    name varchar(29),
    KEY name_name (`name`),
	PRIMARY KEY (id),
    UNIQUE KEY `type` (`type`) ，
    FOREIGN KEY (course) REFERENCES teacher(course)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

 **key** 是数据库的物理结构，它包含两层意义，一是约束（偏重于约束和规范数据库的结构完整性），二是索引（辅助查询用的）。包括primary key, unique key, foreign key 等。    

**primary key** 有两个作用，一是约束作用（constraint），用来规范一个存储主键和唯一性，但同时也在此key上建立了一个index；  

**unique key** 也有两个作用，一是约束作用（constraint），规范数据的唯一性，但同时也在这个key上建立了一个index；    

**foreign key**也有两个作用，一是约束作用（constraint），规范数据的引用完整性，但同时也在这个key上建立了一个index；    

可见，mysql的key是同时具有constraint和index的意义，这点和其他数据库表现的可能有区别。（至少在oracle上建立外键，不会自动建立index），因此创建key也有如下几种方式：    （1）在字段级以key方式建立， 如 create table t (id int not null primary key);    （2）在表级以constraint方式建立，如create table t(id int, CONSTRAINT pk_t_id PRIMARY key (id));    （3）在表级以key方式建立，如create table t(id int, primary key (id));     其它key创建类似，但不管那种方式，既建立了constraint，又建立了index，只不过index使用的就是这个constraint或key。 

### 1.3.2 union和union all

- union 自带去重
- union all 不带去重

```shell
mysql> select * from tbl_dept
    -> union
    -> select * from tbl_dept;
+----+----------+--------+
| id | deptName | locAdd |
+----+----------+--------+
|  1 | RD       | 11     |
|  2 | HR       | 12     |
|  3 | MK       | 13     |
|  4 | MIS      | 14     |
|  5 | FD       | 15     |
+----+----------+--------+
5 rows in set (0.14 sec)

mysql> select * from tbl_dept
    -> union all
    -> select * from tbl_dept;
+----+----------+--------+
| id | deptName | locAdd |
+----+----------+--------+
|  1 | RD       | 11     |
|  2 | HR       | 12     |
|  3 | MK       | 13     |
|  4 | MIS      | 14     |
|  5 | FD       | 15     |
|  1 | RD       | 11     |
|  2 | HR       | 12     |
|  3 | MK       | 13     |
|  4 | MIS      | 14     |
|  5 | FD       | 15     |
+----+----------+--------+
```

## 2. 常见问题

### 2.1 数据表乱码或者无法插入中文

- 产生原因：如果在建库建表的时候，没有明确指定字符集，则采用默认的字符集 latin1,其中是不包含中文字符的 
- 查看数据库字符集命令

```shell
mysql> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | latin1                     | <----
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

- 永久修改： 已经创建的数据库的设定不会发生变化，参数修改只对新建的数据库有效！ 

  修改配置文件

  ```shelll
  [client] default-character-set=utf8 
  [mysqld] character_set_server=utf8 
  character_set_client=utf8 
  collation-server=utf8_general_ci 
  [mysql] default-character-set=utf8
  ```

- 临时修改

  - 修改数据库

  ```shell
  mysql> alter database mydb character set 'utf8';
  ```

  - 修改数据表

  ```shell
  mysql> alter table mytbl convert to character set 'utf8';
  ```

  - 修改已经乱码的数据

    无法修复。

### 2.2 大小写敏感

- 查看命令： 查看大小写是否敏感

  ```shell
  mysql> show variables like '%lower_case_table_names%';
  +------------------------+-------+
  | Variable_name          | Value |
  +------------------------+-------+
  | lower_case_table_names | 0     |
  +------------------------+-------+
  ```

- 默认规则：

  **windows 系统默认大小写不敏感，但是 linux 系统是大小写敏感的**  

  0： 大小写敏感 

  1 ：大小写不敏感。创建的表，数据库都是以小写形式存放在磁盘上，对于 sql 语句都是转换为 

  小写对表和 DB 进行查找 

  2 ：创建的表和 DB 依据语句上格式存放，凡是查找都是转换为小写进行 

- 设置敏感：

  **注意：如果要设置属性为大小写不敏感，要在重启数据库实例之前就需要将原来的数据库和表转换为小写，否则将 找不到数据库名。在进行数据库参数设置之前，需要掌握这个参数带来的影响，切不可盲目设置。**

  在 my.cnf 这个配置文件 [mysqld] 中加入 lower_case_table_names = 1 ，然后重启服务器 

### 2.3 sql_mode 语句语法的校验 

sql_mode 定义了对 Mysql 中 sql 语句语法的校验规则！ 

sql_mode 是个很容易被忽视的变量，默认值是空值，在这种设置下是可以允许一些非法操作的，比如允许一些 

非法数据的插入。在生产环境必须将这个值设置为严格模式，所以开发、测试环境的数据库也必须要设置，这样在 

开发测试阶段就可以发现问题。

- 查看命令

```shell
mysql> select @@sql_mode;
+-------------------------------------------------------------------------------------------------------------------------------------------+
| @@sql_mode                                                                                                                                |
+-------------------------------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+-------------------------------------------------------------------------------------------------------------------------------------------+
```

- 修改配置：

  - 临时修改 sql_mode: 

  ```shell
  set @@sql_mode=’’; 
  ```

  - 永久修改，需要在配置文件中修改： 

  ```shell
  [mysqld] 下添加 sql_mode='' 然后重启 mysql 即可
  ```

### 2.4 MySql权限设置

## 3. Mysql 逻辑架构 

![1621668010275](E:\SoftwareNote\面试准备\数据库\MySql\img\Mysql 逻辑架构.png)

和其它数据库相比，MySQL 有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在 

存储引擎的架构上，插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。这种架构可 

以根据业务的需求和实际需要选择合适的存储引擎。

**四层： 连接层 -> 服务层 -> 引擎层 -> 存储层** 

## 4. Mysql大致查询流程

mysql 的查询流程大致是： 

1. mysql 客户端通过协议与 mysql 服务器建连接，发送查询语句，**先检查查询缓存，如果命中，直接返回结果，** **否则进行语句解析**,也就是说，在解析查询之前，服务器会先访问查询缓存(query cache)——它存储 SELECT 语句以及 相应的查询结果集。如果某个查询结果已经位于缓存中，服务器就不会再对查询进行解析、优化、以及执行。它仅 仅将缓存中的结果返回给用户即可，这将大大提高系统的性能。 
2. **语法解析器和预处理**：首先 mysql 通过关键字将 SQL 语句进行解析，并生成一颗对应的“解析树”。mysql 解析 器将使用 mysql **语法规则验证和解析查询**；预处理器则根据一些 mysql 规则进一步检查**解析数是否合法**。 
3. 查询优化器当解析树被认为是合法的了，并且由优化器将其转化成执行计划。一条查询可以有很多种执行方式， 最后都返回相同的结果。**优化器的作用就是找到这其中最好的执行计划**。
4. 然后，mysql 默认使用的 **BTREE 索引**，并且一个大致方向是:无论怎么折腾 sql，至少在目前来说，**mysql 最多只 用到表中的一个索引。** 

## 5. MySQL执行顺序

- 手写

```sql
select distinct <select_list> 
from <left_table> 
join <right_table> on <join_condition>
where <where_condition>
group by <groupby_list>
having <having_condition>
order by <orderby_list>
limit <limit_number>
```

- 优化器优化过后

  随着 Mysql 版本的更新换代，其优化器也在不断的升级，优化器会分析不同执行顺序产生的性能消耗不同而动 态调整执行顺序。下面是经常出现的查询顺序：

```sql
from <left_table>  
on <join_condition> join <right_table> 
where <where_condition>
group by <groupby_list>
having <having_condition>
select 
distinct <select_list> 
order by <orderby_list>
limit <limit_number>
```

![1621669214184](E:\SoftwareNote\面试准备\数据库\MySql\img\MySQL执行顺序.png)

## 6. MySql引擎： MyISAM 和 InnoDB 

|     对比项     |                   MyISAM                   |                            InnoDB                            |
| :------------: | :----------------------------------------: | :----------------------------------------------------------: |
|      外键      |                   不支持                   |                             支持                             |
|      事务      |                   不支持                   |                             支持                             |
|     行表锁     | **表锁**，即使操作一条记录也会锁住整个表， | **行锁,**操作时只锁某一行，不对其它行有影响，  适合**高并发**的操作 |
|      缓存      |       **只缓存索引**，不缓存真实数据       | **不仅缓存索引还要缓存真实数据，对内存要求较高，而且内 存大小对性能有决定性的影响** |
|     关注点     |                   读性能                   |                      并发写、事务、资源                      |
|    默认安装    |                     √                      |                              √                               |
|    默认使用    |                     ×                      |                              √                               |
| 自带系统表使用 |                     √                      |                              ×                               |

- 查看所有的数据库引擎 

```sql
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.08 sec)
```

- 查看默认的数据库引擎

```sql
mysql> show variables like '%storage_engine%' ;
+----------------------------------+--------+
| Variable_name                    | Value  |
+----------------------------------+--------+
| default_storage_engine           | InnoDB |
| default_tmp_storage_engine       | InnoDB |
| disabled_storage_engines         |        |
| internal_tmp_disk_storage_engine | InnoDB |
+----------------------------------+--------+
4 rows in set (0.10 sec)
```

## 7. 关联查询 Join（mysql不支持full outer join）

场景6：full outer join 合并+去重（Oracle支持）

![1621670121498](C:\Users\ASUS\AppData\Local\Temp\1621670121498.png)

## ☢ 8. 索引优化分析  

### 8.1 索引的概念

- MySQL官方对索引的定义：

		索引（Index）是帮助MySQL高效获取数据的**数据结构**。

可以看出，索引的本质也是数据结构，可以理解为“**排好序供快速查找的(有特定高级查找算法的)数据结构**”。

- 在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些**数据结构以某种方式引用（指向）数据(索引和数据有联系)**，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。下图就是一种可能的索引方式示例：

  ![1621674011325](E:\SoftwareNote\面试准备\数据库\MySql\img\MySQL索引数据结构简略图.png)

  左边是数据表，一共有两列七条记录，最左边的是数据记录的物理地址。为了加快 Col2 的查找，可以维护一个 右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指 针，这样就可以运用 二叉查找在一定的复杂度内获取到相应数据，从而快速的检索出符合条件的记录。 

  一般来说索引本身也很大，不可能全部存储在内存中，因此**索引往往以索引文件的形式存储的磁盘上**。

- 我们平时所说的索引，**如果没有特别指明，都是指B树（多路搜索树，并不一定是二叉的）结构组织的索引**。其中聚集索引，次要索引，覆盖索引，复合索引，前缀索引，**唯一索引默认都是使用B+树索引**，统称索引。当然，除了B+树这种数据结构的索引之外，<u>还有哈希索引（Hash Index）等</u>。

### 8.2 索引的优点和缺点

索引能提高**查询**和**排序(order by )**的效率

- 优势：
  - **查询**：提高数据**检索**的效率，降低数据库的IO成本。 
  - **排序**：通过索引对数据进行排序，降低数据**排序**的成本，降低CPU的消耗。 
- 劣势：
  - 虽然索引大大提高了查询速度，**同时却会降低更新表的速度**，如对表进行INSERT、UPDATE和DELETE。因为更新表时，**MySQL不仅要保存数据，还要保存一下索引文件**每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。 
  - 实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要**占用空间**的。

### 8.3 索引的分类

- 单值索引：一个索引只包含单个列，一个表可以有多个单列索引。
- 唯一索引：索引列的值必须唯一，**但是允许空值**。
- 复合索引：一个索引包含多个字段。

### 8.4 索引的CRUD

- 创建
  - 单语句创建

```sql
/* A.创建索引 [UNIQUE]可以省略*/
/* 如果只写一个字段就是单值索引，写多个字段就是复合索引 */
CREATE [UNIQUE] INDEX indexName ON tabName(columnName(length));

-- B. 使用alter
/* 1、该语句添加一个主键，这意味着索引值必须是唯一的，并且不能为NULL */
ALTER TABLE tabName ADD PRIMARY KEY(column_list);

/* 2、该语句创建索引的键值必须是唯一的(除了NULL之外，NULL可能会出现多次) */
ALTER TABLE tabName ADD UNIQUE indexName(column_list);

/* 3、该语句创建普通索引，索引值可以出现多次 */
ALTER TABLE tabName ADD INDEX indexName(column_list);

/* 4、该语句指定了索引为FULLTEXT，用于全文检索 */
ALTER TABLE tabName ADD FULLTEXT indexName(column_list);
```

​	- 建表时创建

```sql
-- 子表
CREATE TABLE course (
	id tinyint(3) unsigned NOT NULL AUTO_INCREMENT,
    course varchar(16) NOT NULL,
    `type` varchar(100) NOT NULL, 
    name varchar(29),
    KEY name_name (`name`),
	PRIMARY KEY (id),
    UNIQUE KEY `type` (`type`) ，
    KEY (`name`,`type`),
    FOREIGN KEY (course) REFERENCES teacher(course)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- 删除

```sql
/* 删除索引 */
DROP INDEX [indexName] ON tabName;
```

- 查看

```sql
/* 查看索引 */
/* 加上\G就可以以列的形式查看了 不加\G就是以表的形式查看 */
SHOW INDEX FROM tabName\G

+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table    | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| tbl_dept |          0 | PRIMARY  |            1 | id          | A         |           5 | NULL     | NULL   |      | BTREE      |         |               |
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
1 row in set (0.07 sec)
```

### 8.5 索引的数据结构

索引数据结构：

- `B(+)Tree`索引。
- `Hash`索引。
- `Full-text`全文索引。
- `R-Tree`索引。

#### 8.5.1 BTree

![1621680981423](E:\SoftwareNote\面试准备\数据库\MySql\img\BTree的结构图.png)

如磁盘块 1 包含数据项 17 和 35，包含指针 P1、P2、P3， P1 表示小于 17 的磁盘块，P2 表示在 17 和 35 之间的磁盘块，P3 表示大于 35 的磁盘块。 真实的数据存在于叶子节点即 3、5、9、10、13、15、28、29、36、60、75、79、90、99。 

**非叶子节点只不存储真实的数据，只存储指引搜索方向的数据项**，如 17、35 并不真实存在于数据表中。

【查找过程】 

如果要查找数据项 29，那么首先会把磁盘块 1 由磁盘加载到内存，此时发生一次 IO，在内存中用二分查找确定 29 在 17 和 35 之间，锁定磁盘块 1 的 P2 指针，内存时间因为非常短（相比磁盘的 IO）可以忽略不计，通过磁盘块 1 的 P2 指针的磁盘地址把磁盘块 3 由磁盘加载到内存，发生第二次 IO，29 在 26 和 30 之间，锁定磁盘块 3 的 P2 指 针，通过指针加载磁盘块 8 到内存，发生第三次 IO，同时内存中做二分查找找到 29，结束查询，总计三次 IO。 真实的情况是，3 **层的 b+树可以表示上百万的数据，如果上百万的数据查找只需要三次 IO，性能提高将是巨大的**， <u>如果没有索引，每个数据项都要发生一次 IO，那么总共需要百万次的 IO，显然成本非常非常高</u>。

#### 8.5.2 B+Tree

![1621681292055](E:\SoftwareNote\面试准备\数据库\MySql\img\B+Tree数据结构图.png)



- B+Tree 与 B-Tree 的区别 

1）B-树的关键字和记录是放在一起的，**叶子节点可以看作外部节点，不包含任何信息**；B+树的非叶子节点中只有关键字和指向下一个节点的索引，**记录*只*放在叶子节点中**

2）在 B-树中，越靠近根节点的记录查找时间越快，只要找到关键字即可确定记录的存在；而 B+树中每个记录的查找时间基本是一样的，都需要从根节点走到叶子节点，而且在叶子节点中还要再比较关键字。从这个角度看 B- 树的性能好像要比 B+树好，而在实际应用中却是 B+树的性能要好些。因为 B+树的非叶子节点不存放实际的数据，这样每个节点可容纳的元素个数比 B-树多，**树高比 B-树小，这样带来的好处是减少磁盘访问次数**。尽管 B+树找到一个记录所需的比较次数要比 B-树多，但是一次磁盘访问的时间相当于成百上千次内存比较的时间，因此实际中B+树的性能可能还会好些，而且 **B+树的*叶子节点使用指针连接*在一起，方便顺序遍历**（例如查看一个目录下的所有 文件，一个表中的所有记录等），这也是很多数据库和文件系统使用 B+树的缘故。

- 为什么说 B+树比 B-树更适合实际应用中操作系统的文件索引和数据库索引？ 

1) **B+树的磁盘读写代价更低** 

B+树的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对 B-树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就 越多。相对来说 IO 读写次数也就降低了。 

2) **B+树的查询效率更加稳定** 

由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

### 8.6 聚簇索引和非聚簇索引 

聚簇索引并不是一种单独的索引类型，而是一种数据存储方式。术语‘聚簇’表示**数据行和相邻的键值聚簇的存储在一起（有规律有序）**。如下图，左侧的索引就是聚簇索引，因为数据行在磁盘的排列和索引排序保持一致。

![1621681819781](E:\SoftwareNote\面试准备\数据库\MySql\img\聚簇索引和非聚簇索引.png)

- **聚簇索引的好处**： 

按照聚簇索引排列顺序，查询显示一定范围数据的时候，由于**数据都是紧密相连**，数据库不不用从多个数据块中提取数据，所以**节省了大量的 io 操作**。 

- **聚簇索引的限制**： 

对于 mysql 数据库目前**只有 innodb 数据引擎支持聚簇索引**，而 **Myisam 并不支持聚簇索引**。 由于数据物理存储排序方式只能有一种，所以**每个 Mysql 的表只能有一个聚簇索引。一般情况下就是该表的主键**。 

**为了充分利用聚簇索引的聚簇的特性，所以 innodb 表的<u>主键列尽量选用有序的顺序 id</u>，而不建议用 无序的 id，比如 uuid 这种**。

### 8.7 哪些情况需要建索引

- 主键自动建立主键索引（唯一 + **非空**）。
- **频繁作为查询条件**的字段应该创建索引。
- 查询中**与其他表关联的字段**，外键关系建立索引。
- 查询中**排序的字段**，<u>排序字段若通过索引去访问将大大提高排序速度</u>。
- 查询中统计或者分组字段（**group by也和索引有关**）。

### 8.8 那些情况不要建索引

- 记录太少的表。
- **经常增删改的表**。
- **频繁更新的字段**不适合创建索引。
- Where**条件里用不到的字段**不创建索引。
- **字段包含的值区间分布广(尽量不重复)**：假如一个表有10万行记录，有一个字段A只有true和false两种值，并且每个值的分布概率大约为50%，那么对A字段建索引一般不会提高数据库的查询速度。索引的选择性是指索引列中不同值的数目与表中记录数的比。如果一个表中有2000条记录，表索引列有1980个不同的值，那么这个索引的选择性就是1980/2000=0.99。一个索引的选择性越接近于1，这个索引的效率就越高。

### 8.9 强制走索引

**像一些查询范围占比全表10%左右的，索引就失效了，走的全表扫描，因为Mysql有优化，自认为全表扫描更快** 

**mysql强制使用索引:force index(索引名或者主键PRI)**

****例如:

select * from table force index(PRI) limit 2;(强制使用主键)

select * from table force index(ziduan1_index) limit 2;(强制使用索引"ziduan1_index")

select * from table force index(PRI,ziduan1_index) limit 2;(强制使用索引"PRI和ziduan1_index")

### 8.10 禁用索引

**mysql禁止某个索引：ignore index(索引名或者主键PRI)**

****例如:

select * from table ignore index(PRI) limit 2;(禁止使用主键)

select * from table ignore index(ziduan1_index) limit 2;(禁止使用索引"ziduan1_index")

select * from table ignore index(PRI,ziduan1_index) limit 2;(禁止使用索引"PRI,ziduan1_index")

## 9. Explain 性能分析 

### 9.1 概念

使用 EXPLAIN 关键字可以模拟优化器执行 SQL 查询语句，从而知道 MySQL 是如何处理你的 SQL 语句的。分 

析你的查询语句或是表结构的性能瓶颈。 

用法： Explain+SQL 语句。 

Explain 执行后返回的信息：

| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra

### 9.2 Explain 各属性

#### 9.2.1 id

select 查询的序列号,包含一组数字，表示查询中执行 select 子句或操作表的顺序。 

**id 号每个号码，表示一趟独立的查询。一个 sql 的查询趟数越少越好 。** 

1. id 相同，执行顺序由上至下 

```shell
+----+-------------+-------+------------+--------+---------------+---------+---------+------------+------+----------+-------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len |          ref        | rows | filtered | Extra |
+----+-------------+-------+------------+--------+---------------+---------+---------+------------+------+----------+-------+
|  1 | SIMPLE      | t1    | NULL       | ALL    | PRIMARY       | NULL    | NULL    |        NULL       |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | t2    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | test.t1.id |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | t3    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | test.t1.id |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+--------+---------------+---------+---------+------------+------+----------+-------+
```

2. id 不同，id 不同，如果是子查询，id 的序号会递增，**id 值越大优先级越高，越先被执行**

```shell
+----+--------------------+-------+-----------------+---------------+---------+---------+------+------+--------------------------+
| id | select_type        | table | type            | possible_keys | key     | key_len | ref  | rows | Extra                    |
+----+--------------------+-------+-----------------+---------------+---------+---------+------+------+--------------------------+
|  1 | PRIMARY            | t1    | index           | NULL          | PRIMARY | 4       | NULL |    1 | Using where; Using index |
|  2 | DEPENDENT SUBQUERY | t2    | unique_subquery | PRIMARY       | PRIMARY | 4       | func |    1 | Using index; Using where |
|  3 | DEPENDENT SUBQUERY | t3    | unique_subquery | PRIMARY       | PRIMARY | 4       | func |    1 | Using where              |
+----+--------------------+-------+-----------------+---------------+---------+---------+------+------+--------------------------+
```

3. 有相同也有不同。id 如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id 值越大，优先级越高，**越先执行衍生 = DERIVED**(如果有)

```shell
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                                               |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
|  1 | PRIMARY     | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE noticed after reading const tables |
|  2 | DERIVED     | t3    | ALL  | NULL          | NULL | NULL    | NULL |    1 | Using where                                         |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------------------------------------------+
```

#### 9.2.2 select_type

select_type 代表查询的类型，主要是用于区别**普通查询、联合查询、子查询**等的复杂查询。

- select_type 属性含义

  - **SIMPLE** ：简单的 select 查询,查询中**不包含子查询或者 UNION** 

  - **PRIMARY** ：查询中若包含任何复杂的子部分，**<u>最外层</u>查询则被标记为 Primary** 

  - **DERIVED** ：在 FROM 列表中包含的**子查询被标记为 DERIVED(衍生**) MySQL 会**递归(就像方法栈，从底层开始return)**执行这些子查询, 把***结果放在临时表里***。 

  - **SUBQUERY** ：在**SELECT或WHERE列表中包含了子查询** 

    `explain select t2.id from t2 where t2.id = (select t3.id from t3 where t3.id=2);`

  - **DEPEDENT SUBQUERY** ：<u>在SELECT或WHERE列表中包含了子查询</u>,**子查询基于外层** （**区别于SUBQUERY，SUBQUERY的子查询返回单个值“=“； DEPEDENT SUBQUERY子查询返回一组值”in“**）

    `explain select t2.id from t2 where t2.id in (select t3.id from t3 where t3.id=2);`

  - **UNCACHEABLE SUBQUERY** ：无法使用缓存的子查询 

    当使用了**@@来引用系统变量的时候**，不会使用缓存。 **(注意也是”=“，即返回单个值，用”in“就是上面的DEPEDENT SUBQUERY)**

    `explain select * from t3 where t3.id = (select id from t2 where t2.id=@@sort_buffer_size);`

  - **UNION** ：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中,外层SELECT将被标记为：DERIVED 

    ```shell
    mysql> explain select * from (select * from t2 union select * from t3) s3 where s3.id=2;
    +------+--------------+------------+------+---------------+------+---------+------+------+-------------+
    | id   | select_type  | table      | type | possible_keys | key  | key_len | ref  | rows | Extra       |
    +------+--------------+------------+------+---------------+------+---------+------+------+-------------+
    |    1 | PRIMARY      | <derived2> | ALL  | NULL          | NULL | NULL    | NULL |    2 | Using where |
    |    2 | DERIVED      | t2         | ALL  | NULL          | NULL | NULL    | NULL |    1 |             |
    |    3 | UNION        | t3         | ALL  | NULL          | NULL | NULL    | NULL |    1 |             |
    | NULL | UNION RESULT | <union2,3> | ALL  | NULL          | NULL | NULL    | NULL | NULL |             |
    +------+--------------+------------+------+---------------+------+---------+------+------+-------------+
    ```

  - **UNION RESULT** ：从UNION表获取结果的SELECT

#### 9.2.3 table

这个数据是基于哪张表的。

```shell
mysql> explain select * from (select * from t2 union select * from t3) s3 where s3.id=2;
+------+--------------+------------+------+---------------+------+---------+------+------+-------------+
| id   | select_type  | table      | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+--------------+------------+------+---------------+------+---------+------+------+-------------+
|    1 | PRIMARY      | <derived2> | ALL  | NULL          | NULL | NULL    | NULL |    2 | Using where |
|    2 | DERIVED      | t2         | ALL  | NULL          | NULL | NULL    | NULL |    1 |             |
|    3 | UNION        | t3         | ALL  | NULL          | NULL | NULL    | NULL |    1 |             |
| NULL | UNION RESULT | <union2,3> | ALL  | NULL          | NULL | NULL    | NULL | NULL |             |
+------+--------------+------------+------+---------------+------+---------+------+------+-------------+
```

其中,

\<derived2> 的“2”表示该primary外层查询的数据源来自id=2的衍生查询（子查询）

<union2,3>表示该union result为id=2和id=3的结果

#### 9.2.4 type

type 是查询的**访问类型**。**是较为重要的一个指标**，结果值从最好到最坏依次是： 

**system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index >ALL** ，一般来说，**得保证查询至少达到 range 级别，最好能达到 ref**。

1. system 

   表只有一行记录（等于系统表），这是 const 类型的特列，平时不会出现，这个也可以忽略不计

2. const 

   表示**通过索引一次(唯一)就找到了**,const 用于比较 primary key 或者 unique 索引。因为只匹配一行数据，所以很快 

   如将主键置于 where 列表中，MySQL 就能将该查询转换为一个常量。

   `explain select * from t2 where t2.id=1;`

3. eq_ref  （全表扫描唯一的(主键和唯一)索引）

   **唯一性索引扫描**，对于每个索引键，表中只有一条记录与之匹配。常见于**主键或唯一索引扫描**。

   ```shell
   explain select * from t1,t2 where t1.id=t2.id;
   +----+-------------+-------+--------+---------------+---------+---------+------------+------+-------+
   | id | select_type | table | type   | possible_keys | key     | key_len | ref        | rows | Extra |
   +----+-------------+-------+--------+---------------+---------+---------+------------+------+-------+
   |  1 | SIMPLE      | t1    | ALL    | PRIMARY       | NULL    | NULL    | NULL       |    1 |       |
   |  1 | SIMPLE      | t2    | eq_ref | PRIMARY       | PRIMARY | 4       | test.t1.id |    1 |       |
   +----+-------------+-------+--------+---------------+---------+---------+------------+------+-------+
   ```

4. ref  （全表扫描非唯一的索引）

   ***非*唯一性索引扫描**，返回匹配某个单独值的所有行.本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。

   ```shell
   mysql> explain select * from t1,t2 where t1.content=t2.content; -- content为index；
   +----+-------------+-------+------+---------------+----------+---------+-----------------+------+--------------------------+
   | id | select_type | table | type | possible_keys | key      | key_len | ref             | rows | Extra                    |
   +----+-------------+-------+------+---------------+----------+---------+-----------------+------+--------------------------+
   |  1 | SIMPLE      | t1    | ALL  | NULL          | NULL     | NULL    | NULL            |    1 |                          |
   |  1 | SIMPLE      | t2    | ref  | ind_ctnt      | ind_ctnt | 303     | test.t1.content |    1 | Using where; Using index |
   +----+-------------+-------+------+---------------+----------+---------+-----------------+------+--------------------------+
   ```

   5. **range** 

      只检索给定范围的行,使用一个索引来选择行。key 列显示使用了哪个索引一般就是在你的 where 语句中出现 了 **between、<、>、in 等**的查询这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引。

   6. index 

      出现index是sql使用了索引但是没用通过索引进行过滤，一般是使用了覆盖索引(select_list为索引列？)或者是利用索引进行了排序分组。

   ```shell
   mysql> explain select * from t3 ;
   +----+-------------+-------+------+---------------+------+---------+------+------+-------+
   | id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
   +----+-------------+-------+------+---------------+------+---------+------+------+-------+
   |  1 | SIMPLE      | t3    | ALL  | NULL          | NULL | NULL    | NULL |    1 |       |
   +----+-------------+-------+------+---------------+------+---------+------+------+-------+
   1 row in set (0.06 sec)
   
   mysql> explain select id from t3 ;
   +----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
   | id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
   +----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
   |  1 | SIMPLE      | t3    | index | NULL          | PRIMARY | 4       | NULL |    1 | Using index |
   +----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
   ```

   7. all 

      Full Table Scan，将遍历全表以找到匹配的行。

   8. index_merge 

      在查询过程中需要多个索引组合使用，通常出现在有 or 的关键字的 sql 中

   9. ref_or_null 

      对于某个字段既需要关联条件，也需要 null 值得情况下。查询优化器会选择用 ref_or_null 连接查询。

   ```shell
   mysql> explain select * from t2 where t2.content is null or t2.content='123'; -- content为 																		  -- index
   +----+-------------+-------+-------------+---------------+----------+---------+-------+------+--------------------------+
   | id | select_type | table | type        | possible_keys | key      | key_len | ref   | rows | Extra                    |
   +----+-------------+-------+-------------+---------------+----------+---------+-------+------+--------------------------+
   |  1 | SIMPLE      | t2    | ref_or_null | ind_ctnt      | ind_ctnt | 303     | const |    2 | Using where; Using index |
   +----+-------------+-------+-------------+---------------+----------+---------+-------+------+--------------------------+
   ```

   10. index_subquery 

       利用索引来关联子查询，不再全表扫描。

   ```shell
   mysql> explain select * from t3 where t3.content in (select t2.content from t2); -- content
   +----+--------------------+-------+----------------+---------------+----------+---------+------+------+--------------------------+
   | id | select_type        | table | type           | possible_keys | key      | key_len | ref  | rows | Extra                    |
   +----+--------------------+-------+----------------+---------------+----------+---------+------+------+--------------------------+
   |  1 | PRIMARY            | t3    | ALL            | NULL          | NULL     | NULL    | NULL |    1 | Using where              |
   |  2 | DEPENDENT SUBQUERY | t2    | index_subquery | ind_ctnt      | ind_ctnt | 303     | func |    1 | Using index; Using where |
   +----+--------------------+-------+----------------+---------------+----------+---------+------+------+--------------------------+
   ```

   11. unique_subquery 

       该联接类型类似于 index_subquery。 子查询中的唯一索引。

   ```shell
   mysql> explain select * from t3 where t3.id in (select t2.id from t2);
   +----+--------------------+-------+-----------------+---------------+---------+---------+------+------+-------------+
   | id | select_type        | table | type            | possible_keys | key     | key_len | ref  | rows | Extra       |
   +----+--------------------+-------+-----------------+---------------+---------+---------+------+------+-------------+
   |  1 | PRIMARY            | t3    | ALL             | NULL          | NULL    | NULL    | NULL |    1 | Using where |
   |  2 | DEPENDENT SUBQUERY | t2    | unique_subquery | PRIMARY       | PRIMARY | 4       | func |    1 | Using index |
   +----+--------------------+-------+-----------------+---------------+---------+---------+------+------+-------------+
   ```

### 9.3 possible_keys 

显示可能应用在这张表中的索引，一个或多个。**查询涉及到的字段**上若存在索引，则该索引将被列出，但不一定被查询实际使用。

### 9.4 key ：可用来判断是否经过索引 

**实际使用的索引**。如果为NULL，则没有使用索引。

### 9.5 key_len 

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。 key_len 字段能够帮你检查是否充分的利用上了索引。**ken_len 越长，说明索引使用的越充分**。

**如何计算：** 

①先看索引上字段的类型+长度比如 int=4 ; varchar(20) =20 ; char(20) =20 

②如果是 varchar 或者 char 这种字符串字段，视字符集要乘不同的值，比如 utf-8 要乘 3,GBK 要乘 2， 

③varchar 这种动态字符串要加 2 个字节 

④允许为空的字段要加 1 个字节

- Key_len计算规则

![1621840429660](E:\SoftwareNote\面试准备\数据库\MySql\img\ExplainKeyLen计算规则.png)

### 9.6 ref 

显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。

### 9.7 rows 

rows 列显示 MySQL 认为它执行**查询时必须检查的行数。越少越好**！

### 9.8 Extra 

其他的额外重要的信息。

#### 9.8.1 Using filesort

说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。**MySQL中无法利用索引完成的排序操作成为"文件内排序"。**

就是排序(order by)的字段不是索引字段，因此MySQL需要整理一套全新的临时排序方案，比较耗时。

因此：**查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度**  

```shell

```

#### 9.8.2 Using temporary 

使了用临时表保存中间结果,MySQL 在对查询结果排序时使用临时表。常见于**排序 order by 和分组查询 group by**。 **临时表対系统性能损耗很大。**

```shell
mysql> explain select content from t3 group by content ;
+----+-------------+-------+------+---------------+------+---------+------+------+---------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                           |
+----+-------------+-------+------+---------------+------+---------+------+------+---------------------------------+
|  1 | SIMPLE      | t3    | ALL  | NULL          | NULL | NULL    | NULL |    1 | Using temporary; Using filesort |
+----+-------------+-------+------+---------------+------+---------+------+------+---------------------------------+
```

#### 9.8.3 Using index 

Using index 代表表示相应的 select 操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错！如果同时出现 using where，表明索引被用来执行索引键值的查找;如果没有同时出现 using where，表明索引只是用来读取数据而非利用索引执行查找。 

**利用索引进行了排序或分组**。

#### 9.8.4 Using where 

表明使用了 where 过滤。

#### 9.8.5 Using join buffer 

使用了连接缓存 

#### 9.8.6 impossible where 

where 子句的值总是 false，不能用来获取任何元组。

```shell
mysql> explain select * from t2 where id=2 and id=3;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra            |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
```

#### 9.8.7 select tables optimized away:  

在没有 GROUPBY 子句的情况下，基于索引优化 MIN/MAX 操作或者对于 **MyISAM 存储引擎优化 COUNT(*)操** 

**作**，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

## 10. 索引优化 

### 10.1 三个小案例

1. **单表**

   - 需求： 查询`category_id`为1且`comments`大于1的情况下，`views`最多的`article_id`。
   - explain :`EXPLAIN SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1`

   ```shell
   mysql> explain SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
   +----+-------------+---------+------+---------------+------+---------+------+------+-----------------------------+
   | id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows | Extra                       |
   +----+-------------+---------+------+---------------+------+---------+------+------+-----------------------------+
   |  1 | SIMPLE      | article | ALL  | NULL          | NULL | NULL    | NULL |    3 | Using where; Using filesort |
   +----+-------------+---------+------+---------------+------+---------+------+------+-----------------------------+
   ```

   **无索引： type=ALL && Extra=Using filesort**

   - 分析：comments > 1有个范围查询，如果创建三复合索引 index1(category_id ,comments ,views ) 可以改善where的查询效率，但是index1会因为comments **范围查询而导致联合复合失去索引效果** ，即 index1(category_id ,comments ,views )相当于 index1(category_id ,comments )，order by还是没有用到索引排序。 **extra：Using filesort**

   ```shell
   mysql> explain SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
   +----+-------------+---------+-------+---------------+--------+---------+------+------+-----------------------------+
   | id | select_type | table   | type  | possible_keys | key    | key_len | ref  | rows | Extra                       |
   +----+-------------+---------+-------+---------------+--------+---------+------+------+-----------------------------+
   |  1 | SIMPLE      | article | range | index1        | index1 | 8       | NULL |    1 | Using where; Using filesort |
   +----+-------------+---------+-------+---------------+--------+---------+------+------+-----------------------------+
   ```

   - 方案：index2(category_id ,views) 这样查询用的是index2，而排序也适用index2

   ```shell
   mysql> explain SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
   +----+-------------+---------+------+---------------+--------+---------+-------+------+-------------+
   | id | select_type | table   | type | possible_keys | key    | key_len | ref   | rows | Extra       |
   +----+-------------+---------+------+---------------+--------+---------+-------+------+-------------+
   |  1 | SIMPLE      | article | ref  | index2        | index2 | 4       | const |    1 | Using where |
   +----+-------------+---------+------+---------------+--------+---------+-------+------+-------------+
   ```

   **索引index2(category_id ,views)： type=ref && Extra=using where **

2. **双表**

   - explain :

   ```shell
   mysql> explain select * from class c left join book b on b.card=c.card;
   +----+-------------+-------+------+---------------+------+---------+------+------+-------+
   | id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
   +----+-------------+-------+------+---------------+------+---------+------+------+-------+
   |  1 | SIMPLE      | c     | ALL  | NULL          | NULL | NULL    | NULL |   20 |       |
   |  1 | SIMPLE      | b     | ALL  | NULL          | NULL | NULL    | NULL |   20 |       |
   +----+-------------+-------+------+---------------+------+---------+------+------+-------+
   ```

   **无索引： type=ALL && type=ALL  && rows=20  && rows=20**  

   - 分析：左连接，**左表是全表查询，增加索引意义不大**，而右边是通过左边的外键去查找的，增加索引可以提高查询效率

   - 方案： `create index book_card on book(card)`

     mysql> explain select * from class c left join book b on b.card=c.card;
     +----+-------------+-------+------+---------------+-----------+---------+-------------+------+-------------+
     | id | select_type | table | type | possible_keys | key       | key_len | ref         | rows | Extra       |
     +----+-------------+-------+------+---------------+-----------+---------+-------------+------+-------------+
     |  1 | SIMPLE      | c     | ALL  | NULL          | NULL      | NULL    | NULL        |   20 |             |
     |  1 | SIMPLE      | b     | ref  | book_card     | book_card | 4       | test.c.card |    1 | Using index |
     +----+-------------+-------+------+---------------+-----------+---------+-------------+------+-------------+

   **索引book(card)`： type=ALL && type=ref  && rows=20  && rows=1**  

3. **三表** : 其实三表=二表结果left join第三表，效果是一样的。

### 10.2 索引失效的情况

- 全值匹配我最爱。

- **最佳左前缀法则**。

  最佳左前缀法则：如果索引是多字段的复合索引，要遵守最佳左前缀法则。指的是查询从索引的最左前列开始并且**不跳过索引中的字段**。

  ***注意：但是，如果复合索引的字段全部作为where条件，MySQL查询优化会自动帮我们调整顺序（注意调整的是查询where顺序，而不调整排序order by顺序），因此，符合索引全字段不要求有序。*** 

  -- 复合索引 (c1,c2,c3,c4)

  -- 复合索引只用到3个字段，MySQL查询会优化，按复合索引重新排序。c1->c2-><u>c3</u>->c4

  EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c3 > 'a3' AND `c4` = 'a4';

  -- 复合索引只用到4个字段，MySQL查询会优化，按复合索引重新排序。c1->c2->c3-><u>c4</u>

  EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c4` > 'a4' AND `c3` = 'a3';

  **口诀：带头大哥不能死，中间兄弟不能断(断了变成range而已)**。

- 不在索引列上做任何操作（**计算、函数、(自动or手动)类型转换(如varchar传参为数值)**），会导致索引失效而转向全表扫描。

  **口诀：索引列上不计算。**

- 索引中**范围条件(> ，< ，betwwen, %like[like%不会])右边的字段会全部失效**。 <u>可用覆盖索引</u>

  **口诀：范围之后全失效。**

- in(元素>1个)  ，<u>可用覆盖索引</u>

- 尽量使用**覆盖索引（只访问索引的查询，索引列和查询列(少于等于，少用多于)一致）**，减少`SELECT *`。

  **口诀：查询一定不用`*`**。

- MySQL在使**用`!=`或者`<>`**的时候无法使用索引会导致全表扫描。 <u>可用覆盖索引</u>

- **`is null`、`is not null`也无法使用索引**。

- `like`以通配符开头`%abc`索引失效会变成全表扫描。<u>可用覆盖索引</u>

  **口诀：`like`百分加右边。**

  **口诀：覆盖索引保两边。**

- **字符串不加单引号索引失效**。

  **口诀：字符要加单引号。**

- **少用`or`**，用它来连接时会索引失效。

- **更新无索引字段，条件也是非索引字段，会导致表锁。**

  `update user set password='2' where password='1'` 

### 10.3 小案例分析 

**假设index(a,b,c)**

| Where语句                                               | 索引是否被使用                             |
| ------------------------------------------------------- | ------------------------------------------ |
| where a = 3                                             | Y，使用到a                                 |
| where a = 3 and b = 5                                   | Y，使用到a，b                              |
| where a = 3 and b = 5                                   | Y，使用到a，b，c                           |
| where b = 3 或者 where b = 3 and c = 4 或者 where c = 4 | N，没有用到a字段                           |
| where a = 3 and c = 5                                   | 使用到a，但是没有用到c，因为b断了          |
| where a = 3 and b > 4 and c = 5                         | 使用到a，b，但是没有用到c，因为c在范围之后 |
| where a = 3 and b like 'kk%' and c = 4                  | Y，a，b，c都用到                           |
| where a = 3 and b like '%kk' and c = 4                  | 只用到a                                    |
| where a = 3 and b like '%kk%' and c = 4                 | 只用到a                                    |
| where a = 3 and b like 'k%kk%' and c = 4                | Y，a，b，c都用到                           |

```sql
/* 最好索引怎么创建的，就怎么用，按照顺序使用，避免让MySQL再自己去翻译一次 */

/* 1.全值匹配 用到索引c1 c2 c3 c4全字段 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c3` = 'a3' AND `c4` = 'a4';

/* 2.用到索引c1 c2 c3 c4全字段 MySQL的查询优化器会优化SQL语句的顺序*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c4` = 'a4' AND `c3` = 'a3';

/* 3.用到索引c1 c2 c3 c4全字段 MySQL的查询优化器会优化SQL语句的顺序*/
EXPLAIN SELECT * FROM `test03` WHERE `c4` = 'a4' AND `c3` = 'a3' AND `c2` = 'a2' AND `c1` = 'a1';

/* 4.用到索引c1 c2 c3字段，c4字段失效，范围之后全失效 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c3` > 'a3' AND `c4` = 'a4';

/* 5.用到索引c1 c2 c3 c4全字段 MySQL的查询优化器会优化SQL语句的顺序*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c4` > 'a4' AND `c3` = 'a3';

/* 
   6.用到了索引c1 c2 c3三个字段, c1和c2两个字段用于查找,  c3字段用于排序了但是没有统计到key_len中，c4字段失效
*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c4` = 'a4' ORDER BY `c3`;

/* 7.用到了索引c1 c2 c3三个字段，c1和c2两个字段用于查找, c3字段用于排序了但是没有统计到key_len中*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' ORDER BY `c3`;

/* 
   8.用到了索引c1 c2两个字段，c4失效，c1和c2两个字段用于查找，c4字段排序产生了Using filesort说明排序没有用到c4字段 
*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' ORDER BY `c4`;

/* 9.用到了索引c1 c2 c3三个字段，c1用于查找，c2和c3用于排序 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c5` = 'a5' ORDER BY `c2`, `c3`;

/* 10.用到了c1一个字段，c1用于查找，c3和c2两个字段索引失效，产生了Using filesort */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c5` = 'a5' ORDER BY `c3`, `c2`;

/* 11.用到了c1 c2 c3三个字段，c1 c2用于查找，c2 c3用于排序 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND  `c2` = 'a2' ORDER BY c2, c3;

/* 12.用到了c1 c2 c3三个字段，c1 c2用于查找，c2 c3用于排序 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND  `c2` = 'a2' AND `c5` = 'a5' ORDER BY c2, c3;

/* 
   13.用到了c1 c2 c3三个字段，c1 c2用于查找，c2 c3用于排序 没有产生Using filesort 
      因为之前c2这个字段已经确定了是'a2'了，这是一个常量，再去ORDER BY c3,c2 这时候c2已经不用排序了！
      所以没有产生Using filesort 和(10)进行对比学习！
*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c5` = 'a5' ORDER BY c3, c2;



/* GROUP BY 表面上是叫做分组，但是分组之前必定排序。 */

/* 14.用到c1 c2 c3三个字段，c1用于查找，c2 c3用于排序，c4失效 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c4` = 'a4' GROUP BY `c2`,`c3`;

/* 15.用到c1这一个字段，c4失效，c2和c3排序失效产生了Using filesort */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c4` = 'a4' GROUP BY `c3`,`c2`;
```

**`GROUP BY`基本上都需要进行排序，索引优化几乎和`ORDER BY`一致，但是`GROUP BY`会有<u>临时表的产生</u>。**

**分组之前必排序，所以规则差不多**

### 10.4 查询优化：小表驱动大表

- 优化原则：对于MySQL数据库而言，永远都是小表驱动大表。

```java
/**
* 举个例子：可以使用嵌套的for循环来理解小表驱动大表。
* 以下两个循环结果都是一样的，但是对于MySQL来说不一样，
* 第一种可以理解为，和MySQL建立5次连接每次查询1000次。
* 第一种可以理解为，和MySQL建立1000次连接每次查询5次。
*/
for(int i = 1; i <= 5; i ++){
    for(int j = 1; j <= 1000; j++){
        
    }
}
// ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
for(int i = 1; i <= 1000; i ++){
    for(int j = 1; j <= 5; j++){
        
    }
}
```

- IN和EXISTS：可相互转换

```sql
/* 优化原则：小表驱动大表，即小的数据集驱动大的数据集 */

/* IN适合B表比A表数据小的情况*/
SELECT * FROM `A` WHERE `id` IN (SELECT `id` FROM `B`)

/* EXISTS适合B表比A表数据大的情况 */
SELECT * FROM `A` WHERE EXISTS (SELECT 1 FROM `B` WHERE `B`.id = `A`.id);
```

​	-  `EXISTS(subquery)`子查询只返回`true`或者`false`，因此子查询中的`SELECT *`可以是`SELECT 1 OR SELECT X`，它们并没有区别（常量即可）。

​	- `EXISTS(subquery)`**子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比**，如果担心效率问题，可进行实际检验以确定是否有效率问题。

​	- `EXISTS(subquery)`子查询往往也可以用条件表达式，其他子查询或者`JOIN`替代，何种最优需要具体问题具体分析。

### 10.5 order by 和 group by

#### 10.5.1 order by

```sql
-- CREATE INDEX idx_talA_age_birth ON `talA`(`age`, `birth`);

/* 1.使用索引进行排序了 不会产生Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `age`;

/* 2.使用索引进行排序了 不会产生Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `age`,`birth`;

/* 3.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `birth`;

/* 4.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `birth`,`age`;

/* 5.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` ORDER BY `birth`;

/* 6.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `birth` > '2020-08-04 07:42:21' ORDER BY `birth`;

/* 7.使用索引进行排序了 不会产生Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `birth` > '2020-08-04 07:42:21' ORDER BY `age`;

/* 8.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` ORDER BY `age` ASC, `birth` DESC;
```

`ORDER BY`子句，尽量使用索引排序，避免使用`Using filesort`排序。

MySQL支持两种方式的排序，`FileSort`和`Index`，`Index`的效率高，它指MySQL扫描索引本身完成排序。`FileSort`方式效率较低。

`ORDER BY`满足两情况，会使用`Index`方式排序：

- `ORDER BY`语句使用索引最左前列。
- 使用`WHERE`子句与`ORDER BY`子句条件列组合满足索引最左前列。

**结论：尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀原则。**



- **如果不在索引列上，File Sort有两种算法：MySQL就要启动双路排序算法和单路排序算法**

  1、双路排序算法：**MySQL4.1之前**使用双路排序，字面意思就是两次扫描磁盘，最终得到数据，读取行指针和`ORDER BY`列，対他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出。**一句话，从磁盘取排序字段，在`buffer`中进行排序，再从磁盘取其他字段。**

  取一批数据，要对磁盘进行两次扫描，众所周知，IO是很耗时的，所以在MySQL4.1之后，出现了改进的算法，就是单路排序算法。

  2、单路排序算法：从磁盘读取查询需要的所有列，按照`ORDER BY`列在`buffer`対它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。

  

  由于单路排序算法是后出的，总体而言效率好过双路排序算法。

  但是单路排序算法有问题：如果`SortBuffer`缓冲区太小，导致从磁盘中读取所有的列不能完全保存在`SortBuffer`缓冲区中，这时候单路复用算法就会出现问题，反而性能不如双路复用算法。

- **单路复用算法的优化策略：**

  - 增大**`sort_buffer_size`**参数的设置。
  - 增大**`max_length_for_sort_data`**参数的设置。

- **提高ORDER BY排序的速度：**

  - `ORDER BY`时使用`SELECT *`是大忌，查什么字段就写什么字段，这点非常重要。在这里的影响是：
    - 当查询的字段大小总和小于`max_length_for_sort_data`而且排序字段不是`TEXT|BLOB`类型时，会使用单路排序算法，否则使用多路排序算法。
    - 两种排序算法的数据都有可能超出`sort_buffer`缓冲区的容量，超出之后，会创建`tmp`临时文件进行合并排序，导致多次IO，但是单路排序算法的风险会更大一些，所以要增大`sort_buffer_size`参数的设置。
  - 尝试提高`sort_buffer_size`：不管使用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的。
  - 尝试提高`max_length_for_sort_data`：提高这个参数，会增加用单路排序算法的概率。但是如果设置的太高，数据总容量`sort_buffer_size`的概率就增大，明显症状是高的磁盘IO活动和低的处理器使用率。

#### 10.5.2 group by

- **`GROUP BY`实质是<u>先排序</u>后进行分组**，遵照索引建的最佳左前缀。
- 当无法使用索引列时，会使用`Using filesort`进行排序，增**大`max_length_for_sort_data`参数的设置和增大`sort_buffer_size`参数**的设置，会提高性能。
- **`WHERE`执行顺序高于`HAVING`，能写在`WHERE`限定条件里的就不要写在`HAVING`中了**。

### 10.6 总结

索引优化的一般性建议：

- 对于单值索引，尽量选择针对当前`query`过滤性更好的索引。
- 在选择复合索引的时候，当前`query`中过滤性最好的字段在索引字段顺序中，位置越靠前越好。
- 在选择复合索引的时候，尽量选择可以能够包含当前`query`中的`where`子句中更多字段的索引。
- 尽可能通过分析统计信息和调整`query`的写法来达到选择合适索引的目的。

**为排序使用索引**

- MySQL两种排序方式：`Using filesort`和`Index`扫描有序索引排序。
- MySQL能为排序与查询使用相同的索引，创建的索引既可以用于排序也可以用于查询。

```sql
/* 创建a b c三个字段的索引 */
idx_table_a_b_c(a, b, c)

/* 1.ORDER BY 能使用索引最左前缀 */
ORDER BY a;
ORDER BY a, b;
ORDER BY a, b, c;
ORDER BY a DESC, b DESC, c DESC;

/* 2.如果WHERE子句中使用索引的最左前缀定义为常量，则ORDER BY能使用索引 */
WHERE a = 'Ringo' ORDER BY b, c;
WHERE a = 'Ringo' AND b = 'Tangs' ORDER BY c;
WHERE a = 'Ringo' AND b > 2000 ORDER BY b, c;

/* 3.不能使用索引进行排序 */
ORDER BY a ASC, b DESC, c DESC;  /* 排序不一致 */
WHERE g = const ORDER BY b, c;   /* 丢失a字段索引 */
WHERE a = const ORDER BY c;      /* 丢失b字段索引 */
WHERE a = const ORDER BY a, d;   /* d字段不是索引的一部分 */
WHERE a IN (...) ORDER BY b, c;  /* 对于排序来说，多个相等条件(a=1 or a=2)也是范围查询 */
```



口诀：

- 带头大哥不能死。
- 中间兄弟不能断。
- 索引列上不计算。
- 范围之后全失效。
- 覆盖索引尽量用。
- 不等有时会失效。
- like百分加右边。
- 字符要加单引号。
- 一般SQL少用or。

## 11. 分析慢SQL的步骤

分析：

1、观察，至少跑1天，看看生产的慢SQL情况。

2、开启慢查询日志，设置阈值，比如超过5秒钟的就是慢SQL，并将它抓取出来。

3、explain + 慢SQL分析。

4、show Profile。

5、运维经理 OR DBA，进行MySQL数据库服务器的参数调优。



总结（大纲）：

1、慢查询的开启并捕获。

2、explain + 慢SQL分析。

3、show Profile查询SQL在MySQL数据库中的执行细节和生命周期情况。

4、MySQL数据库服务器的参数调优。

## 12.慢查询日志

### 12.1.基本介绍

> 慢查询日志是什么？

- MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阈值的语句，具体指运行时间超过`long_query_time`值的SQL，则会被记录到慢查询日志中。
- `long_query_time`的默认值为10，意思是运行10秒以上的语句。
- 由慢查询日志来查看哪些SQL超出了我们的最大忍耐时间值，比如一条SQL执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒钟的SQL，结合之前`explain`进行全面分析。

> 特别说明

**默认情况下，MySQL数据库没有开启慢查询日志，**需要我们手动来设置这个参数。

**当然，如果不是调优需要的话，一般不建议启动该参数**，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件。

> 查看慢查询日志是否开以及如何开启

- 查看慢查询日志是否开启：`SHOW VARIABLES LIKE '%slow_query_log%';`。
- 开启慢查询日志：`SET GLOBAL slow_query_log = 1;`。**使用该方法开启MySQL的慢查询日志只对当前数据库生效，如果MySQL重启后会失效。**

```shell
# 1、查看慢查询日志是否开启
mysql> SHOW VARIABLES LIKE '%slow_query_log%';
+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_query_log      | OFF                                  |
| slow_query_log_file | /var/lib/mysql/1dcb5644392c-slow.log |
+---------------------+--------------------------------------+
2 rows in set (0.01 sec)

# 2、开启慢查询日志
mysql> SET GLOBAL slow_query_log = 1;
Query OK, 0 rows affected (0.00 sec)
```



如果要使慢查询日志永久开启，需要修改`my.cnf`文件，在`[mysqld]`下增加修改参数。

```shell
# my.cnf
[mysqld]
# 1.这个是开启慢查询。注意ON需要大写
slow_query_log=ON  

# 2.这个是存储慢查询的日志文件。这个文件不存在的话，需要自己创建
slow_query_log_file=/var/lib/mysql/slow.log
```



> 开启了慢查询日志后，什么样的SQL才会被记录到慢查询日志里面呢？

这个是由参数`long_query_time`控制的，默认情况下`long_query_time`的值为10秒。

MySQL中查看`long_query_time`的时间：`SHOW VARIABLES LIKE 'long_query_time%';`。

```shell
# 查看long_query_time 默认是10秒
# 只有SQL的执行时间>10才会被记录
mysql> SHOW VARIABLES LIKE 'long_query_time%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)
```



修改`long_query_time`的时间，需要在`my.cnf`修改配置文件

```shell
[mysqld]
# 这个是设置慢查询的时间，我设置的为1秒
long_query_time=1
```



查新慢查询日志的总记录条数：`SHOW GLOBAL STATUS LIKE '%Slow_queries%';`。

```shell
mysql> SHOW GLOBAL STATUS LIKE '%Slow_queries%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 3     |
+---------------+-------+
1 row in set (0.00 sec)
```

### 12.2.日志分析工具 

日志分析工具`mysqldumpslow`：在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具`mysqldumpslow`。

```shell
# 1、mysqldumpslow --help 来查看mysqldumpslow的帮助信息
root@1dcb5644392c:/usr/bin# mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default  # 按照何种方式排序
                al: average lock time # 平均锁定时间
                ar: average rows sent # 平均返回记录数
                at: average query time # 平均查询时间
                 c: count  # 访问次数
                 l: lock time  # 锁定时间
                 r: rows sent  # 返回记录
                 t: query time  # 查询时间 
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries  # 返回前面多少条记录
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string  
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
  
# 2、 案例
# 2.1、得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/slow.log
 
# 2.2、得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/slow.log
 
# 2.3、得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/slow.log

# 2.4、另外建议使用这些命令时结合|和more使用，否则出现爆屏的情况
mysqldumpslow -s r -t 10 /var/lib/mysql/slow.log | more
```

## 13.Show Profile

> Show Profile是什么？

`Show Profile`：MySQL提供可以用来**分析当前会话中语句执行的资源消耗情况**。可以用于SQL的调优的测量。**默认情况下，参数处于关闭状态，并保存最近15次的运行结果。**

> 分析步骤

1、是否支持，看看当前的MySQL版本是否支持。

```shell
# 查看Show Profile功能是否开启
mysql> SHOW VARIABLES LIKE 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```

2、开启`Show Profile`功能，默认是关闭的，使用前需要开启。

```shell
# 开启Show Profile功能
mysql> SET profiling=ON;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

3、运行SQL

```mysql
SELECT * FROM `emp` GROUP BY `id`%10 LIMIT 150000;

SELECT * FROM `emp` GROUP BY `id`%20 ORDER BY 5;
```

4、查看结果，执行`SHOW PROFILES;`

`Duration`：持续时间。

```shell
mysql> SHOW PROFILES;
+----------+------------+---------------------------------------------------+
| Query_ID | Duration   | Query                                             |
+----------+------------+---------------------------------------------------+
|        1 | 0.00156100 | SHOW VARIABLES LIKE 'profiling'                   |
|        2 | 0.56296725 | SELECT * FROM `emp` GROUP BY `id`%10 LIMIT 150000 |
|        3 | 0.52105825 | SELECT * FROM `emp` GROUP BY `id`%10 LIMIT 150000 |
|        4 | 0.51279775 | SELECT * FROM `emp` GROUP BY `id`%20 ORDER BY 5   |
+----------+------------+---------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

5、**诊断SQL，`SHOW PROFILE cpu,block io FOR QUERY Query_ID;`**

```shell
# 这里的3是第四步中的Query_ID。
# 可以在SHOW PROFILE中看到一条SQL中完整的生命周期。
mysql> SHOW PROFILE cpu,block io FOR QUERY 3;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.000097 | 0.000090 |   0.000002 |            0 |             0 |
| checking permissions | 0.000010 | 0.000009 |   0.000000 |            0 |             0 |
| Opening tables       | 0.000039 | 0.000058 |   0.000000 |            0 |             0 |
| init                 | 0.000046 | 0.000046 |   0.000000 |            0 |             0 |
| System lock          | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |
| optimizing           | 0.000005 | 0.000000 |   0.000000 |            0 |             0 |
| statistics           | 0.000023 | 0.000037 |   0.000000 |            0 |             0 |
| preparing            | 0.000014 | 0.000000 |   0.000000 |            0 |             0 |
| Creating tmp table   | 0.000041 | 0.000053 |   0.000000 |            0 |             0 |
| Sorting result       | 0.000005 | 0.000000 |   0.000000 |            0 |             0 |
| executing            | 0.000003 | 0.000000 |   0.000000 |            0 |             0 |
| Sending data         | 0.520620 | 0.516267 |   0.000000 |            0 |             0 |
| Creating sort index  | 0.000060 | 0.000051 |   0.000000 |            0 |             0 |
| end                  | 0.000006 | 0.000000 |   0.000000 |            0 |             0 |
| query end            | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |
| removing tmp table   | 0.000006 | 0.000000 |   0.000000 |            0 |             0 |
| query end            | 0.000004 | 0.000000 |   0.000000 |            0 |             0 |
| closing tables       | 0.000009 | 0.000000 |   0.000000 |            0 |             0 |
| freeing items        | 0.000032 | 0.000064 |   0.000000 |            0 |             0 |
| cleaning up          | 0.000019 | 0.000000 |   0.000000 |            0 |             0 |
+----------------------+----------+----------+------------+--------------+---------------+
20 rows in set, 1 warning (0.00 sec)
```

`Show Profile`查询参数备注：

- `ALL`：显示所有的开销信息。
- `BLOCK IO`：显示块IO相关开销（通用）。
- `CONTEXT SWITCHES`：上下文切换相关开销。
- `CPU`：显示CPU相关开销信息（通用）。
- `IPC`：显示发送和接收相关开销信息。
- `MEMORY`：显示内存相关开销信息。
- `PAGE FAULTS`：显示页面错误相关开销信息。
- `SOURCE`：显示和Source_function。
- `SWAPS`：显示交换次数相关开销的信息。



6、`Show Profile`查询列表，日常开发需要注意的结论：

- **`converting HEAP to MyISAM`：查询结果太大，内存都不够用了，往磁盘上搬了。**
- **`Creating tmp table`：创建临时表（拷贝数据到临时表，用完再删除），非常耗费数据库性能。**
- **`Copying to tmp table on disk`：把内存中的临时表复制到磁盘，危险！！！**
- **`locked`：死锁。**

## 14. MyISAM表锁(偏读) 

**表锁特点：**

- 表锁偏向`MyISAM`存储引擎，开销小，加锁快，无死锁，锁定粒度大，发生锁冲突的概率最高，并发度最低。

### 14.1 锁表相关命令

- 查看数据库表锁的命令。

```shell
# 查看数据库表锁的命令
mysql>SHOW OPEN TABLES;
+--------------------+----------------------------------------------+--------+-------------+
| Database           | Table                                        | In_use | Name_locked |
+--------------------+----------------------------------------------+--------+-------------+
| mysql              | time_zone_transition_type                    |      0 |           0 |
| performance_schema | events_waits_summary_global_by_event_name    |      0 |           0 |
| performance_schema | setup_timers                                 |      0 |           0 |
| performance_schema | events_waits_history_long                    |      0 |           0 |
| mysql              | db                                           |      0 |           0 |
| performance_schema | performance_timers                           |      0 |           0 |
| performance_schema | events_waits_summary_by_instance             |      0 |           0 |
| performance_schema | rwlock_instances                             |      0 |           0 |
| test               | mylock                                       |      0 |           0 |
```

- 锁表

```shell
# 给mylock表上读锁，给book表上写锁
LOCK TABLE `mylock` READ, `book` WRITE;

# 查询状态出现改变，In_Use=1
SHOW OPEN TABLES;
+--------------------+----------------------------------------------+--------+-------------+
| Database           | Table                                        | In_use | Name_locked |
+--------------------+----------------------------------------------+--------+-------------+
| mysql              | time_zone_transition_type                    |      0 |           0 |
| performance_schema | events_waits_summary_global_by_event_name    |      0 |           0 |
| test               | mylock                                       |      1 |           0 |
| mysql              | time_zone_transition                         |      0 |           0 |
| performance_schema | mutex_instances                              |      0 |           0 |
| test               | book                                         |      1 |           0 |
```

- 释放锁

```shell
# 释放给表添加的锁
UNLOCK TABLES;
```

### 14.2 MyISAM 表锁总结

**MYISAM`引擎在执行查询语句`SELECT`之前，会自动给涉及到的所有表加读锁，在执行增删改之前，会自动给涉及的表加写锁。**

MySQL的表级锁有两种模式：

- 表共享读锁（Table Read Lock）。
- 表独占写锁（Table Write Lock）。

対`MyISAM`表进行操作，会有以下情况：

- 対`MyISAM`表的读操作（加读锁READ），不会阻塞其他线程（会话）対同一表的读操作，但是会**阻塞**其他线程対同一表的写操作。只有当读锁释放之后，才会执行其他线程的写操作。(**阻塞写，阻塞*<u>本会话</u>*读其他表**)
- 対`MyISAM`表的写操作（加写锁WRITE），会阻塞其他线程対同一表的**读**和写操作，只有当写锁释放之后，才会执行其他线程的读写操作。(**阻塞读写(本会话的写不阻塞)，阻塞*<u>本会话</u>*读其他表** )

**总的来说：不管是不是其他会话，表锁只有读读可以共存(本会话的写写可以共存)，表锁时本会话不可读其他表**

### 14.3 表锁分析

```shell
mysql> SHOW STATUS LIKE 'table%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Table_locks_immediate      | 173   |
| Table_locks_waited         | 0     |
| Table_open_cache_hits      | 5     |
| Table_open_cache_misses    | 8     |
| Table_open_cache_overflows | 0     |
+----------------------------+-------+
5 rows in set (0.00 sec)
```

可以通过`Table_locks_immediate`和`Table_locks_waited`状态变量来分析系统上的表锁定。具体说明如下：

`Table_locks_immediate`：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1。

`Table_locks_waited`：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），此值高则说明存在较严重的表级锁争用情况。

**此外，`MyISAM`的读写锁调度是写优先，这也是`MyISAM`不适合作为主表的引擎。因为写锁后，其他线程不能进行任何操作，大量的写操作会使查询很难得到锁，从而造成永远阻塞。**

## 15. InnoDB行锁 

### 15.1 **行锁特点：**

- 偏向`InnoDB`存储引擎，**开销大，加锁慢；会出现死锁**；锁定粒度最小，发生锁冲突的概率最低，**并发度最高。**

**`InnoDB`存储引擎和`MyISAM`存储引擎最大不同有两点：<u>一是支持事务，二是采用行锁</u>。**

事务的ACID：

- `Atomicity [ˌætəˈmɪsəti] `。原子性。

  事务的原子性是指事务必须是一个原子的操作序列单元。事务中包含的各项操作在一次执行过程中，只允许出现以下两种状态之一。 **同时成功，同时失败。** 

- `Consistency [kənˈsɪstənsi] `。一致性。

  事务的一致性是指事务的执行不能破坏数据库数据的完整性和一致性，**一个事务在执行之前和执行之后，数据库都必须处于一致性状态**。也就是说，事务执行的结果必须是使数据库从一个一致性状态转变到另一个一致性状态，因此当数据库只包含成功事务提交 的结果时，就能说数据库处于一致性状态。而如果数据库系统在运行过程中发生故障，  有些事务尚未完成就被迫中断，这些未完成的事务对数据库所做的修改有一部分已写入物理数据库，这时数据库就处于一种不正确的状态，或者说是不一致的状态。 

- `Isolation [ˌaɪsəˈleɪʃn]`。隔离性。

  事务的隔离性是指在并发环境中，并发的事务是相互隔离的，**一个事务的执行不能被其他事务干扰**。也就是说，不同的事务并发操纵相同的数据时，每个事务都有各自完整的数据空间，即一个事务内部的操作及使用的数据对其他并发事务是隔离的，**并发执行的 各个事务之间不能互相干扰**。 

- `Durability [ˌdjʊərəˈbɪlɪti] `。持久性。

  事务的持久性也被称为永久性，是指一个**事务一旦提交，它对数据库中对应数据的状态变更就应该是永久性的**。换句话说，一旦某个事务成功结束，那么它对数据库所做的更新就必须被永久保存下来——**即使发生系统崩溃或机器宕机等故障**，只要数据库能够重新启动，那么一定能够将其恢复到事务成功结束时的状态 

### 15.2 索引失效行锁变表锁

这个要特别小心，一不小心变成表锁。

**如varchar条件用数值，就会导致索引失效**。select * from students where name =5; (name varchar2(20))

### 15.3 间隙锁的危害

> 什么是间隙锁？

当我们用**范围条件**而不是相等条件检索数据，并请求共享或者排他锁时，`InnoDB`会给符合条件的已有数据记录的索引项加锁，**对于键值在条件范围内但并不存在的记录，叫做"间隙(GAP)"。**

`InnoDB`也会对这个"间隙"加锁，这种锁的机制就是所谓的"间隙锁"。

> 间隙锁的危害

因为`Query`执行过程中通过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值不存在。

间隙锁有一个比较致命的缺点，就是**当锁定一个范围的键值后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。**在某些场景下这可能会対性能造成很大的危害。

### 15.4 for update行锁（排他锁）

`SELECT .....FOR UPDATE`在锁定某一行后，**其他写操作会被阻塞，直到锁定的行被`COMMIT`。**

mysql InnoDB引擎默认的修改数据语句，**update,delete,insert都会<u>自动</u>给涉及到的数据加上排他锁**，select语句默认不会加任何锁类型，如果加排他锁可以使用select ...for update语句，加共享锁可以使用select ... lock in share mode语句。所以加过排他锁的数据行在其他事务种是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但**可以直接通过select ...from...查询数据，因为普通查询没有任何锁机制**。

### 15.5 结论

`InnoDB`存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体**并发处理能力**方面要远远优于`MyISAM`的表级锁定的。当系统并发量较高的时候，`InnoDB`的整体性能和`MyISAM`相比就会有比较明显的优势了。

但是，`InnoDB`的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让`InnoDB`的整体性能表现不仅不能比`MyISAM`高，甚至可能会更差。

### 15.6 行锁分析 

```shell
mysql> SHOW STATUS LIKE 'innodb_row_lock%';
+-------------------------------+--------+
| Variable_name                 | Value  |
+-------------------------------+--------+
| Innodb_row_lock_current_waits | 0      |
| Innodb_row_lock_time          | 124150 |
| Innodb_row_lock_time_avg      | 31037  |
| Innodb_row_lock_time_max      | 51004  |
| Innodb_row_lock_waits         | 4      |
+-------------------------------+--------+
5 rows in set (0.00 sec)
```

対各个状态量的说明如下：

- `Innodb_row_lock_current_waits`：当前正在等待锁定的数量。
- `Innodb_row_lock_time`：从系统启动到现在锁定总时间长度（重要）。
- `Innodb_row_lock_time_avg`：每次等待所花的平均时间（重要）。
- `Innodb_row_lock_time_max`：从系统启动到现在等待最长的一次所花的时间。
- `Innodb_row_lock_waits`：系统启动后到现在总共等待的次数（重要）。

尤其是当等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手制定优化策略。

## 16. MySQL主从复制

### 16.1 原理

![1621927773232](E:\SoftwareNote\面试准备\数据库\MySql\img\MySQL主从复制原理.png)

MySQL复制过程分为三步：

- **Master将改变记录到二进制日志(Binary Log)**。这些记录过程叫做二进制日志事件，`Binary Log Events`；
- **Slave将Master的`Binary Log Events`拷贝到它的中继日志**(Replay  Log);
- **Slave重做中继日志中的事件**，将改变应用到自己的数据库中。**MySQL复制是异步且串行化的**。

### 16.2 主从复制基本原则

- 每个Slave只有一个Master。
- **每个Slave只能有一个唯一的服务器ID**。
- 每个Master可以有多个Salve。

### 16.3 配置 

> 1、基本要求：Master和Slave的MySQL服务器版本一致且后台以服务运行。

```shell
# 创建mysql-slave1实例
docker run -p 3307:3306 --name mysql-slave1 \
-v /root/mysql-slave1/log:/var/log/mysql \
-v /root/mysql-slave1/data:/var/lib/mysql \
-v /root/mysql-slave1/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=333 \
-d mysql:5.7
```

> 2、主从配置都是配在[mysqld]节点下，都是小写

```shell
# Master配置
[mysqld]
server-id=1 # 必须
log-bin=/var/lib/mysql/mysql-bin # 必须
read-only=0
binlog-ignore-db=mysql
```

```shell
# Slave配置
[mysqld]
server-id=2 # 必须
log-bin=/var/lib/mysql/mysql-bin
```

> 3、Master配置

```shell
# 1、GRANT REPLICATION SLAVE ON *.* TO 'username'@'从机IP地址' IDENTIFIED BY 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'zhangsan'@'172.18.0.3' IDENTIFIED BY '123456';
Query OK, 0 rows affected, 1 warning (0.01 sec)

# 2、刷新命令
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

# 3、记录下File和Position
# 每次配从机的时候都要SHOW MASTER STATUS;查看最新的File和Position
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      602 |              | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

> 4、Slave从机配置

```shell
CHANGE MASTER TO MASTER_HOST='172.18.0.4',
MASTER_USER='zhangsan',
MASTER_PASSWORD='123456',
MASTER_LOG_FILE='mysql-bin.File的编号',
MASTER_LOG_POS=Position的最新值;
```



```shell
# 1、使用用户名密码登录进Master
mysql> CHANGE MASTER TO MASTER_HOST='172.18.0.4',
    -> MASTER_USER='zhangsan',
    -> MASTER_PASSWORD='123456',
    -> MASTER_LOG_FILE='mysql-bin.000001',
    -> MASTER_LOG_POS=602;
Query OK, 0 rows affected, 2 warnings (0.02 sec)

# 2、开启Slave从机的复制
mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)

# 3、查看Slave状态
# Slave_IO_Running 和 Slave_SQL_Running 必须同时为Yes 说明主从复制配置成功！
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event # Slave待命状态
                  Master_Host: 172.18.0.4
                  Master_User: zhangsan
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 602
               Relay_Log_File: b030ad25d5fe-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes  
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 602
              Relay_Log_Space: 534
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: bd047557-b20c-11ea-9961-0242ac120002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

> 5、测试主从复制

```shell
# Master创建数据库
mysql> create database test_replication;
Query OK, 1 row affected (0.01 sec)

# Slave查询数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_replication   |
+--------------------+
5 rows in set (0.00 sec)
```

> 6、停止主从复制功能

```shell
# 1、停止Slave
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.00 sec)

# 2、重新配置主从
# MASTER_LOG_FILE 和 MASTER_LOG_POS一定要根据最新的数据来配
mysql> CHANGE MASTER TO MASTER_HOST='172.18.0.4',
    -> MASTER_USER='zhangsan',
    -> MASTER_PASSWORD='123456',
    -> MASTER_LOG_FILE='mysql-bin.000001',
    -> MASTER_LOG_POS=797;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.18.0.4
                  Master_User: zhangsan
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 797
               Relay_Log_File: b030ad25d5fe-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 797
              Relay_Log_Space: 534
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: bd047557-b20c-11ea-9961-0242ac120002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

### 16.4 一些细节
#### 16.4.1 服务器配置
从服务器高于主服务器，免得出现同步受阻
#### 16.4.2 从同步的语句更新
语句更新会执行语句，因此一些函数,比如now()，on update等等，会导致从库数据不一致，
因此开发过程中，尽量将业务逻辑放在代码中而不是使用sql的语法。

## 17. Mysql与Oracle区别

1. **Oracle是大型数据库而Mysql是中小型数据库**，Oracle市场占有率达40%，Mysql只有20%左右，同时Mysql是开源的而Oracle价格非常高。 

2. **Oracle支持大并发，大访问量**，是OLTP最好的工具。 

3. **安装所用的空间差别**也是很大的，Mysql安装完后才152M而Oracle有3G左右，且使用的时候**Oracle占用特别大的内存空间和其他机器性能。** 

4. 4.Oracle也Mysql操作上的一些区别 

   ①**主键** 

   Mysql一般使用自动增长类型，在创建表时只要指定表的主键为auto increment,插入记录时，不需要再指定该记录的主键值，Mysql将自动增长；Oracle没有自动增长类型，**主键一般使用的序列**，插入记录时将序列号的下一个值付给该字段即可；只是ORM框架是只要是native主键生成策略即可。 

   ②单引号的处理 

   MYSQL里可以用双引号包起字符串，**ORACLE里只可以用单引号包起字符串**。在插入和修改字符串前必须做单引号的替换：把所有出现的一个单引号替换成两个单引号。 

   ③翻页的SQL语句的处理 

   MYSQL处理翻页的SQL语句比较简单，用LIMIT 开始位置, 记录个数；ORACLE处理翻页的SQL语句就比较繁琐了。**每个结果集只有一个ROWNUM字段标明它的位置,** 并且**只能用ROWNUM<100, 不能用ROWNUM>80** 

   ④ 长字符串的处理 

   长字符串的处理ORACLE也有它特殊的地方。INSERT和UPDATE时最大可操作的字符串长度小于等于**4000个单**字节, 如果要插入更长的字符串,  请考虑字段用**CLOB类型**，方法借用ORACLE里自带的DBMS_LOB程序包。插入修改记录前一定要做进行非空和长度判断，不能为空的字段值和超出长度字段值都应该提出警告,返回上次操作。 

   ⑤空字符的处理 

   **MYSQL的非空字段也有空的内容，ORACLE里定义了非空字段就不容许有空的内容**。按MYSQL的NOT NULL来定义ORACLE表结构, 导数据的时候会产生错误。因此导数据时要对空字符进行判断，如果为NULL或空字符，需要把它改成一个空格的字符串。 

   ⑥字符串的模糊比较 

   MYSQL里用 字段名 like '%字符串%',ORACLE里也可以用 字段名 like '%字符串%' 但这种方法不能使用索引, 速度不快。 

   ⑦Oracle实现了ANSII SQL中大部分功能，如，**事务的隔离级别、传播特性**等而Mysql在这方面还是比较的弱。 

   **⑧ group by ： Oracle是select字段必须在group by里面的；而MySQL不需要。**

## 18 大数据量测试

>  1000万数据，四个字段 id(PK) username   gender  password

- 硬盘存储：888MB

- 调用存储过程

  - call myproc() => 此时数据表的引擎为MyISAM ，无事务，可以快速插入

    Time: 569.570s   

  		插入效率： 1000w / 569.570s = 17,557 row/s 

  - alter table test_user ENGINE='INNODB'

    Time: 34.615s 再改成InnoDB节省时间

- 非主键无索引搜索

  `select * from test_user where username='15232';`  **4.404s**

- 主键搜搜

  `select * from test_user where id=15232`  **0.056s**

- 创建普通单列索引

  `alter table test_user add index index1(username)  ` **21.047s**

  硬盘大小变化： 888MB -> 1100MB , 相当于 一个1000w数据量的普通index占用222MB

- 有索引查询

  `select * from test_user where username='15232';`  **0.052s ** ，4.402/0.052=84.69 效率提升8400%

- 范围查询

  - `select * from test_user where username like '1%'; ` **4.892s** 

   查询结果count: 1111112 占比总数据为1111112/10000000=**11.11% 索引失效，type由range变成all**

  - `select * from test_user where username like '15%';` **0.368s**    **range**

   查询结果count: 111112 占比总数据为111112/10000000=**1.11% 索引未失效，type为range**

- **大数据量，mysql会优化不走索引，可以走强制索引**

  `select * from test_user  force index(index1) where username like '1%';` 

  5s -> 3.9s 时间少了1s， 查询条数rows由992w->222w， type 由 all -> range

- order by 和 group by

  `select * from test_user Group BY username;` -- 33s

  `select * from test_user order BY username;` -- 20.8s

- 索引覆盖

  select username from test_user   order BY username; -- 3.997s  索引 覆盖查询

  select username from test_user   group BY username; -- 4.892s  索引覆盖查询

- limit  1000w数据，limit页数越往后，速度越慢

explain SELECT * FROM `test_user` limit 9000000,100  -- ALL  9665713   3.917S

explain SELECT * FROM `test_user` limit 900000,100  -- ALL  9665713    0.412S

explain SELECT * FROM `test_user` limit 90000,100  -- ALL  9665713    0.093S

## 19. Limit原理及性能优化

### 19.1 limit原理以及慢查询分析

```sql
limit m -> limt 0, m

limit m,n ->  m < count < m+n

select * from table_name  order by id limit 10000,10
```

这句 SQL 的执行逻辑是  

1.从数据表中读取第N条数据添加到数据集中  

2.**重复第一步直到 N = 10000 + 10**  

3.**根据 offset 抛弃前面 10000 条数**  

4.返回剩余的 10 条数据 

**limit会按顺序查询数据，直至查到你写的偏移量10000，因此，页数越往后查询越慢**

### 19.2 limit 慢查询优化

`select * from table_name order by id limit 10000,10`

1. 场景一(几乎没这种场景）：主键自增，且id不断

   `select * from table_name where id > 10001 order by id limit 10`

2.  先用索引列扫描，再子查询

   ```sql
   explain SELECT * FROM `test_user` order by id limit 9000000,10  -- 3.971
   
   explain SELECT id FROM `test_user` order by id limit 9000000,10  -- 2.998
   
   explain select * from test_user where id >= (select id from test_user order by id asc limit 9000000,1 ) order by id Limit 10;  -- 3.099
   ```

## 20. using(id) 语句

join查询时，单on条件可以用using(相同的字段)来代替on条件内容



## 21. 事务隔离级别

### 21.1 未隔离：脏读 / 不可重复读 / 幻读

- **脏读（Dirty Read）：**

  **A更新，B读取到更新，A更新失败回滚，B读到的是回滚前的脏数据**

- **不可重复读（Nonrepeatable Read）：**

  **B读取数据，A更新并提交，B再读取数据，发现两次数据不一致**

  不可重复读有一种特殊情况，两个事务更新同一条数据资源，**后完成的事务会造成先完成的事务更新丢失**。这种情况就是大名鼎鼎的**第二类丢失更新**。主流的数据库已经默认屏蔽了第一类丢失更新问题（即：后做的事务撤销，发生回滚造成已完成事务的更新丢失），但我们编程的时候仍需要特别注意第二类丢失更新。 

- **幻读（Phantom Read）：**

  **B读取数据，A更新并提交，B再读取数据，发现两次数据<u>集合</u>不一致(和不可重复读有点类似，但是幻读是指集合长度变化，不可重复读指数据的变化)**

### 21.2 数据库隔离级别

**为了解决上面提及的并发问题**，主流关系型数据库**都会提供四种事务隔离级别。** 

>  **一个事务与其他事务的隔离程度成为隔离级别。（即：隔离级别讨论的是多个事务同时开启的情况）**

SQL标准中定义不同隔离级别，**隔离级别越高，数据一致性越好，但并发性越弱**。

- **读未提交（READ UnCommitted）**

  事务1可读取事务2**未提交**的修改。 

- **读已提交（READ Committed）**（**Oracle**的默认隔离级别，**开始时常用**）

  事务1**只能**读事务2**已提交**的修改

- **可重复读（RePeatable READ）** （**MySQL**的默认隔离级别）

  **事务1执行期间，禁止其他事务修改这个字段。**

- **串行化（Serializable）**

  **（类似表锁）**在事务1执行期间，禁止其他事务修改这个表。这样可避免任何并发问题，然鹅性能十分底下。

> **隔离级别能力大赏**
>
> |                  | 脏读 | 不可重复读 | 幻读 |
> | :--------------: | :--: | :--------: | :--: |
> | READ uncommitted |  有  |     有     |  有  |
> |  READ committed  |  无  |     有     |  有  |
> | RePeatable READ  |  无  |     无     |  有  |
> |   Serializable   |  无  |     无     |  无  |

> **MySQL和Oracle对隔离级别的支持**
>
> |                  | Oracle  |  MySQL  |
> | :--------------: | :-----: | :-----: |
> | READ uncommitted |    ×    |    √    |
> |  READ committed  | √(默认) |    √    |
> | RePeatable READ  |    ×    | √(默认) |
> |   Serializable   |    √    |    √    |

## 22. 表设计三大范式

① 列不可再分：比如把学号性名等整在一个字段内

② 行唯一： 一般有主键就行

③ 非主键字段互不关联：

比如学生表：id / name / no / **teacherNo / teacherName / teacherPhoneNum**（老师名字电话依赖老师No，应该拆分新表）

## 23. offset 和 limit

\1. select* from article LIMIT 1,3

2.select * from article LIMIT 3 OFFSET 1

上面两种写法都表示取2,3,4三条条数据

## 24. 千万级别表优化

https://www.zhihu.com/question/19719997

**1.用小表驱动大表，把left join的条件先按按单表条件查询出来，然后按查出来的小数据集去left join大表，**
**达到小表驱动大表的效果**

**2.分表，按时间分历史表** 

EXPLAIN select * FROM `test_user` t1 
left join test_user t2 on t1.id=t2.id
where t1.username BETWEEN '100' and '200'   -- 9.9724s

-- 1	SIMPLE	t1		ALL	index1				9665713	23.58	Using where
-- 1	SIMPLE	t2		eq_ref	PRIMARY	PRIMARY	8	test.t1.id	1	100	

flush query cache

EXPLAIN select t2.* from 
(select id from test_user where username BETWEEN '100' and '200') t1
LEFT join test_user t2
on t1.id=t2.id   -- 4.140s

-- 1	SIMPLE	test_user		range	index1	index1	36		2279046	100	Using where; Using index
-- 1	SIMPLE	t2		eq_ref	PRIMARY	PRIMARY	8	test.test_user.id	1	100	

## 25.  MySQL高级语法

### 25.1 优雅的createOrUpdate （ON DUPLICATE KEY UPDATE）

1：ON DUPLICATE KEY UPDATE需要有在INSERT语句中有存在**主键或者唯一索引的列** ，并且对应的数据已经在表中才会执行更新操作。而且如果要更新的字段是主键或者唯一索引，不能和表中已有的数据重复，否则插入更新都失败。

2：不管是更新还是增加语句都不允许将主键或者唯一索引的对应字段的数据变成表中已经存在的数据

### 25.2 ignore 自动忽略重复插入

数据不存在则插入，数据已存在则忽略，可以使用 MySQL ignore 来实现。当 primary key 或者 unique key 重复时会自动忽略本次插入操作。 

`insert ignore into `webVisit`(`ip`) values(#{ip})`

### 25.3 replace into 自动替换重复数据

数据不存在则插入，数据已存在则替换，可以使用 MySQL replace into 来实现。当 primary key 或者 unique key 重复时会自动替换已存在的数据。**实际上是先删除原有数据，然后插入新数据。**

```
replace into `webVisit`(`date`,`ip`, uri, `count`)
values(#{date},#{ip},#{uri}, 1)
```

### 25.4 IP转换

SELECT INET_NTOA(3232235521)

SELECT INET_ATON('192.168.0.1');

## 26. 数值类型长度

MYSQL中数值类型()并不表示长度，字段可以存储的长度由下表所示。

- MySQL数值相关的范围及占用字节数

| 类型      | 占用字节数 | 有符号取值范围           | 无符号取值范围(关键字unsigned) |
| --------- | ---------- | ------------------------ | ------------------------------ |
| TINYINT   | 1          | -128 ~ 127               | 0 ~ 255                        |
| SMALLINT  | 2          | -32768 ~ 32767           | 0 ~ 65535                      |
| MEDIUMINT | 3          | -8388608 ~ 8388607       | 0 ~ 16777215                   |
| INT       | 4          | -2147483648 ~ 2147483647 | 0 ~ 4294967295                 |
| BIGINT    | 8          | -2^63 ~ 2^63-1           | 0 ~ 2^64-1                     |

- 作用：配合zerofill使用

**int(5) 其实是和另一个属性 zerofill 配合使用的，表示如果该字段值的宽度小于 5 时，会自动在前面补 0 ，如果宽度大于等于 5 ，那就不需要补 0** 

```sql
#int(5) length 字段添加了 zerofill 属性
mysql> select * from test;
+----+--------+
| id | length |
+----+--------+
|  1 |  00888 |
|  2 |  00012 |
|  3 |  12345 |
|  4 | 123456 |
+----+--------+
```

注意前提是给该字段添加了 zerofill 属性，不然 int(5) 不起作用。

## 27. MySQL数据表的字段信息详情 information_schema.COLUMNS
### 27.1 查询数据表的全字段
- **group_concat语法**
`**语法：group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )**`
- **带别名的数据表全字段**
select CONCAT('dt.',group_concat(COLUMN_NAME separator ',dt.')) from information_schema.COLUMNS where table_name = 'tb_event_dt';
## 28. MySQL 一些规则
### 28.1 varchar
varchar 在5.0后，中英文数字都只占一个字符，所以varchar(5) 可以存5个汉字
### 28.2 数据表行格式ROW_FORMAT
ENGINE=MyISAM DEFAULT CHARSET=utf8

ROW_FORMAT=COMPACT

;

ROW_FORMAT有这些存储格式选择：

fixed：**默认格式**，

当表不包含变长字段(varchar / varbinary / blob / text)时使用，

每行都是固定的，所以很容易获取行在页上的具体位置，存取效率比较高，

但是占用磁盘空间较大

dynamic：

每行都有一个行头部，包含bitmap，记录列为空的情况。（字符类型长度为0，或数字类型为0，而不是NULL值）

所有字符串列都是动态存储的，除非长度小于4;

fixed->dynamic : 会导致CHAR->VARCHAR，反之亦然， 

MYISAM引擎可以修改ROW_FORMAT，InnoDB不可以，默认Compact

REDUNDANT为固长，有冗余，COMPACT更灵活
## 28. sql特殊字符转义
> MySQL查询LIKE如何匹配下划线 通配符转义
MySQL查询时使用LIKE匹配下划线，您会发现连查询“%A_B%”时会出现“%A B%”和“%AB%”也查询出来了，这是因为下划线也被当作特殊字符，做了任意匹配转换了，所以，要想匹配下划线，那么就需要“转义”一下。转义的方法有如下（示例想查询A_B匹配字段）。

- 使用Escape转义
示例：

SELECT * FROM mytable WHERE col LIKE '%A#_B%' ESCAPE '#';

或，

SELECT * FROM mytable WHERE col LIKE '%A\_B%';

其中#符号随意写，只是告诉解析器把这个字符当特殊字符解析。“\”则默认按照转义字符解析。

- 使用终括号[]转义
示例：SELECT * FROM mytable WHERE col LIKE '%A[_]B%';

 
- 其它方法，还有使用instr函数查询等方式，不过，那样把简单问题搞复杂了，不推荐。
