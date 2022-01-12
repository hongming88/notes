# 逻辑架构

<img src="..\..\pics\database\mysql\逻辑架构2.png" style="zoom:45%;" />

<img src="..\..\pics\database\mysql\逻辑架构.png" style="zoom:75%;" />



## 连接层

<font color="red">**每一个客户端发起一个新的请求都由服务器端的连接/线程处理工具负责接收客户端的请求并开辟一个新的内存空间，在服务器端的内存中生成一个新的线程，当每一个用户连接到服务器端的时候就会在进程地址空间里生成一个新的线程用于响应客户端请求，用户发起的查询请求都在线程空间内运行， 结果也在这里面缓存并返回给服务器端**</font>。线程的重用和销毁都是由连接/线程处理管理器实现的。

　　综上所述：用户发起请求，连接/线程处理器开辟内存空间，开始提供查询的机制。


最上层是一些客户端和连接服务，包含本地sock 通信和大多数基于客户端/服务端工具实现的类似于tcp/ip 的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在**该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程**。同样在该层上可以实现基于SSL 的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。



可以通过如下命令查看连接配置信息：`SHOW VARIABLES LIKE '%connect%';`可以看到最大连接和每个连接占用的内存等相关配置。



## 服务层

| Management Serveices & Utilities | 系统管理和控制工具                                           |
| -------------------------------- | ------------------------------------------------------------ |
| SQL Interface:                   | 接口。接受用户的SQL 命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface |
| Parser                           | 解析器。SQL 命令传递到解析器的时候会被解析器验证和解析       |
| Optimizer                        | 查询优化器。SQL 语句在查询之前会使用查询优化器对查询进行优化，比如有where 条件时，优化器来决定先投影还是先过滤。 |
| Cache 和Buffer                   | 查询缓存。如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key 缓存，权限缓存等 |

## 引擎层
存储引擎层，存储引擎真正的负责了MySQL 中数据的存储和提取，服务器通过API 与存储引擎进行通信。不同
的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。

可以通过下面两个命令查看MySQL当前版本，和对存储引擎的支持情况。

```sql
SELECT VERSION() ; SHOW ENGINES ;
```



## 存储层
数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互。



## **MySQL查询过程**

<img src="..\..\pics\database\mysql\查询过程.png" style="zoom:44%;" />



参考连接：https://blog.csdn.net/fuzhongmin05/article/details/70904190



## MyISAM 和InnoDB



| 对比项         |                          MyISAM                          |                            InnoDB                            |
| -------------- | :------------------------------------------------------: | :----------------------------------------------------------: |
| 外键           |                          不支持                          |                             支持                             |
| 事务           |                          不支持                          |                             支持                             |
| 行表锁         | 表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作 | 行锁,操作时只锁某一行，不对其它行有影响，**适合高并发**的操作 |
| 缓存           |                只缓存索引，不缓存真实数据                | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 关注点         |                          读性能                          |                      并发写、事务、资源                      |
| 默认安装       |                            Y                             |                              Y                               |
| 默认使用       |                            N                             |                              Y                               |
| 自带系统表使用 |                            Y                             |                              N                               |

查看命令：

- 查看mysql以提供什么存储引擎
  - `show engines;`

- 查看mysql当前默认的存储引擎
  - `show variables like '%storage_engine%';`

# 索引

## 索引是什么

**MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。可以得到索引的本质：索引是数据结构。**

索引的目的在于提高查询效率，可以类比字典。

如果要查“mysql”这个单词，我们肯定需要定位到m字母，然后从下往下找到y字母，再找到剩下的sql。

如果没有索引，那么你可能需要逐个逐个寻找，如果我想找到Java开头的单词呢？或者Oracle开头的单词呢？

是不是觉得如果没有索引，这个事情根本无法完成？

你可以简单理解为“排好序的快速查找数据结构”。



详解：

在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用(指向）数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。下图就是**一种可能的索引方式**示例：


![](..\..\pics\database\mysql\索引1.jpg)



左边是**数据表**，一共有两列七条记录，最左边的是数据记录的**物理地址**。

为了加快Col2的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定的复杂度内获取到相应数据，从而快速的检索出符合条件的记录。

**数据本身之外，数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种数据结构就是索引。**

<font color="red">一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。</font>

==我们平常所说的索引，如果没有特别指明，都是指B树（多路搜索树，并不一定是二叉的）结构组织的索引。==其中聚集索引，次要索引，覆盖索引，复合索引，前缀索引，唯一索引默认都是使用B+树索引，统称索引。当然，除了B+树这种类型的索引之外，还有哈稀索引(hash index)等。

## 索引优劣势

**优势**

- 类似大学图书馆建书目索引，提高数据检索的效率，降低数据库的IO成本。

- 通过索引列对数据进行排序，降低数据排序的成本，降低了CPU的消耗。

**劣势**

- 实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要占用空间的（占空间）

- 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。**因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息**。

- 索引只是提高效率的一个因素，如果你的MysQL有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询。

**总结**

- 索引，空间换取时间。



## 索引分类和建索引命令语句

**MySQL索引分类：**

- 单值索引：即一个索引只包含单个列，一个表可以有多个单列索引。
- 唯一索引：索引列的值必须唯一，但允许有空值。
- 主键索引:设定为主键后数据库会自动建立索引，innodb为聚簇索引
- 复合索引：即一个索引包含多个列。
- 基本语法：

| 操作                                       | 命令                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| 创建                                       | - `CREATE [UNIQUE] INDEX indexName ON mytable(columnName(length));`<br/>- `ALTER mytable ADD [UNIQUE] INDEX [indexName] ON (columnName(length));` |
| 删除                                       | `DROP INDEX [indexName] ON mytable;`                         |
| 查看                                       | `SHOW INDEX FROM tableName;`                                 |
| 使用alter命令 有四种方式来添加数据表的索引 | `ALTER TABLE tbl_name ADD PRIMARY KEY (column_list);`：该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。 |
|                                            | `ALTER TABLE tbl name ADD UNIQUE index_name (column_list);`：这条语句创建索引的值必须是唯一的(除了NULL外，NULL可能会出现多次)。 |
|                                            | `ALTER TABLE tbl_name ADD INDEX index_name (column_list);`：添加普通索引，索引值可出现多次。 |
|                                            | `ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list);`：该语句指定了索引为FULLTEXT，用于全文索引。 |



## 索引的创建时机
 **适合创建索引的情况**

- 主键自动建立唯一索引；

-  频繁作为查询条件的字段应该创建索引

-  查询中与其它表关联的字段，外键关系建立索引

- 单键/组合索引的选择问题， 组合索引性价比更高

- 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度

- 查询中统计或者分组字段

  

**不适合创建索引的情况**

- 表记录太少
- 经常增删改的表或者字段

# Explain 性能分析

## 性能分析前提知识

**MySQL Query Optimizer**

- **Mysql中有专门负责优化SELECT语句的优化器模块**，主要功能:通过计算分析系统中收集到的统计信息，为客户端请求的Query提供他认为最优的执行计划（他认为最优的数据检索方式，但不见得是DBA认为是最优的,这部分最耗费时间）

- 当客户端向MySQL请求一条Query，命令解析器模块完成请求分类，区别出是SELECT并转发给MySQL Query Optimizer时，MySQL Query Optimizer首先会对整条Query进行优化，处理掉一些常量表达式的预算直接换算成常量值。并对Query中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等。然后分析Query 中的 Hint信息(如果有），看显示Hint信息是否可以完全确定该Query的执行计划。如果没有Hint 或Hint信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据Query进行写相应的计算分析，然后再得出最后的执行计划。

**MySQL常见瓶颈**

- `CPU`：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
- `IO`：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候
- 服务器硬件的性能瓶颈：top，free，iostat和vmstat来查看系统的性能状态

## **explain使用简介**
使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈。

[官网地址](https://dev.mysql.com/doc/refman/8.0/en/execution-plan-information.html)

**能干嘛**

- 表的读取顺序
- 数据读取操作的操作类型
- 哪些索引可以使用
- 哪些索引被实际使用
- 表之间的引用
- 每张表有多少行被优化器查询

**怎么玩**

`explain + sql语句`

执行计划包含的信息
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |

例如：

```sql
mysql> select * from tbl_dept;
+----+----------+--------+
| id | deptName | locAdd |
+----+----------+--------+
|  1 | RD       | 11     |
|  2 | HR       | 12     |
|  3 | MK       | 13     |
|  4 | MIS      | 14     |
|  5 | FD       | 15     |
+----+----------+--------+
5 rows in set (0.00 sec)

mysql> explain select * from tbl_dept;
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table    | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | tbl_dept | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |   100.00 | NULL  |
+----+-------------+----------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

```



## explain之id介绍

select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序

三种情况：

- id相同，执行顺序由上至下
- id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
- id相同不同，同时存在



①**id相同，执行顺序由上至下**

![](..\..\pics\database\mysql\explain_id1.jpg)

②**id 不同，id 不同，如果是子查询，id 的序号会递增，id 值越大优先级越高，越先被执行**

![](..\..\pics\database\mysql\explain_id2.jpg)

③id 如果相同，可以认为是一组，从上往下顺序执行；**在所有组中，id 值越大，优先级越高**，越先执行衍生= DERIVED

![](..\..\pics\database\mysql\explain_id3.jpg)

关注点：<font color="red">id 号每个号码，表示一趟独立的查询。一个sql 的查询趟数越少越好。</font>

## explain之select_type和table介绍

**select_type**：查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询。

**select_type有哪些？**

1. `SIMPLE` - 简单的select查询,查询中不包含子查询或者UNION。
2. `PRIMARY` - 查询中若包含任何复杂的子部分，最外层查询则被标记为。
3. `SUBQUERY` - 在SELECT或WHERE列表中包含了子查询。
4. `DERIUED` - 在FROM列表中包含的子查询被标记为DERIVED（衍生）MySQ会递归执行这些子查询，把结果放在临时表里。
5. `UNION` - 若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中外层SELECT将被标记为：DERIVED。
6. `UNION RESULT` - 从UNION表获取结果的SELECT。


**table**：显示这一行的数据是关于哪张表的。

## explain之type介绍
访问类型排列

type显示的是访问类型，是较为重要的一个指标，结果值从最好到最坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index >ALL



<font color="red">从最好到最差以此是</font>:

system>const>eq_ref>ref>range>index>ALL



**详细说明**

- `system`：表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计。
- `const`：表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快如将主键置于where列表中，MySQL就能将该查询转换为一个常量。
- `eq_ref`：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。
- `ref`：非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。
- `range`：只检索给定范围的行,使用一个索引来选择行。key列显示使用了哪个索引一般就是在你的where语句中出现了between、<、>、in等的查询。这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引。
- `index`：Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）。
- `all`：Full Table Scan，将遍历全表以找到匹配的行。



<font color="red">一般来说，得保证查询至少达到range级别，最好能达到ref。</font>

## explain之possible_keys和key介绍
**possible_keys**

显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段火若存在索引，则该索引将被列出，==但不一定被查询实际使用==。

**key**

**实际使用的索引**。如果为NULL，则没有使用索引

**查询中若使用了覆盖索引，则该索引仅出现在key列表中**

## explain之key_len介绍

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。key_len 字段能够帮你检查是否充分的 利用上了索引。ken_len 越长，说明索引使用的越充分。在不损失精确性的情况下，长度越短越好

key_len显示的值为索引字段的最大可能长度，**并非实际使用长度**，即key_len是根据表定义计算而得，不是通过表内检索出的



**如何计算**：
①先看索引上字段的类型+长度比如`int=4 ; varchar(20) =20 ; char(20) =20`
②如果是varchar 或者char 这种字符串字段，视字符集要乘不同的值，比如utf-8 要乘3,GBK 要乘2，
③varchar 这种动态字符串要加2 个字节
④允许为空的字段要加1 个字节

## explain之ref介绍

**显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。**



查询中与其它表关联的字段，外键关系建立索引。

## explain之rows介绍

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数。

rows 列显示MySQL 认为它执行查询时必须检查的行数。越少越好！

## explain之Extra介绍

包含不适合在其他列中显示但十分重要的额外信息。

**Using filesort**（九死一生）

说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。<font color="red">MySQL中无法利用索引完成的排序操作称为"文件排序"</font>



**Using temporary**（不好）

使了用临时表保存中间结果，MysQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by。

**Using index**

表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错！

如果同时出现using where，表明索引被用来执行索引键值的查找；

如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。



<font color="red"> **覆盖索引**（Covering Index）</font>,一说为索引覆盖。

- 理解方式一：就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件,换句话说**查询列要被所建的索引覆盖**。

- 理解方式二：索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据；当能通过读取索引就可以得到想要的数据，那就不需要读取行了。一个索引包含了（或覆盖了）满足查询结果的数据就叫做覆盖索引。

- 注意：如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select*，因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。



**Using where**

表明使用了where过滤。



**Using join buffer**

使用了连接缓存。



**impossible where**

where子句的值总是false，不能用来获取任何元组。



**select tables optimized away**

在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作，或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。



**distinct**

优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作。


## explain之热身Case

![](..\..\pics\database\mysql\explain_Case.png)

第一行（执行顺序4）：id列为1，表示是union里的第一个select，select_type列的primary表示该查询为外层查询，table列被标记为，表示查询结果来自一个衍生表，其中derived3中3代表该查询衍生自第三个select查询，即id为3的select。`【select d1.name… 】`

第二行（执行顺序2）：id为3，是整个查询中第三个select的一部分。因查询包含在from中，所以为derived。`【select id,namefrom t1 where other_column=’’】`

第三行（执行顺序3）：select列表中的子查询select_type为subquery，为整个查询中的第二个select。`【select id from t3】`

第四行（执行顺序1）：select_type为union，说明第四个select是union里的第二个select，最先执行`【select name,id from t2】`

第五行（执行顺序5）：代表从union的临时表中读取行的阶段，table列的<union1,4>表示用第一个和第四个select的结果进行union操作。`【两个结果union操作】`


## 总结

id,type,key,rows Extra最重要了



# 索引优化

## 索引单表优化案例

`mysql> explain SELECT id, author_id FROM article WHERE category_id = 1 AND comments = 1 ORDER BY views DESC LIMIT 1;`



type变成了range，这是可以忍受的。但是extra里使用Using filesort仍是无法接受的。

但是我们已经建立了索引，为啥没用呢？

这是因为按照BTree索引的工作原理，先排序category_id，如果遇到相同的category_id则再排序comments,如果遇到相同的comments 则再排序views。

当comments字段在联合索引里处于中间位置时，因comments > 1条件是一个范围值(所谓range)，MySQL无法利用索引再对后面的views部分进行检索，即range类型查询字段后面的索引无效。


## 索引两表优化案例

**小结**

索引两表优化，<font color="red">左连接右表建索引，右连接左表建索引。</font>

主表不管加不加都会全表扫描

## 索引三表优化案例



Join语句的优化

尽可能减少Join语句中的NestedLoop的循环总次数，不要join过多或者嵌套；<font color="red">“永远用小结果集驱动大的结果集”。</font>

优先优化NestedLoop的内层循环，保证Join语句中被驱动表上Join条件字段已经被索引。

当无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，不要太吝惜JoinBuffer的设置。



# 单表使用索引常见的索引失效



索引失效（应该避免）

1. 最佳左前缀法则 - <font color="red">如果索引了多列，要遵守最左前缀法则。**指的是查询从索引的最左前列开始并且不跳过复合索引中间列。**</font>

2. 不在**索引列上做任何操作**（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描。

   如：`SELECT SQL_NO_CACHE * FROM emp WHERE LEFT(age,3)=30;`

3. 存储引擎**不能使用索引中范围条件右边的列**。(范围之后全失效，所以范围放后面)

   如：`mysql> EXPLAIN SELECT * FROM staffs WHERE NAME='July' AND age>25 AND pos='dev'`

   - `这里我的理解是,age>25是需要遍历的,在遍历的过程中同时判断了pos条件,因此pos的索引就无效了`

4. 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *。

   如：

   ```sql
   explain SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30 and deptId=4 and name='XamgXt';()
   
   explain SELECT SQL_NO_CACHE age,deptId,name FROM emp WHERE emp.age=30 and deptId=4 and name='XamgXt';(这个有use index)
   ```

5. mysql在使用不等于（!=或者<>）的时候**无法使用索引会导致全表扫描。**

   例如：`mysql> EXPLAIN SELECT * FROM staffs WHERE NAME is not null;`

6. 字段的is null, is not null 也无法使用索引。

7. like以通配符开头（’%abc…’），mysql索引失效会变成全表扫描的操作。

8. 字符串不加单引号索引失效。

9. 少用or，用它来连接时会索引失效。
   





**问题：解决like '%字符串%'时索引不被使用的方法？**

用上索引（覆盖索引）

**小结**

解决like '%字符串%'时索引不被使用的方法？复合索引，然后覆盖索引。

## 索引失效10-小总结

小总结

假设index(a, b, c)

| where语句   | 索引是否被使用 |
| ----------- | -------------- |
| where a = 3 | Y，使用到a     |
|where a = 3 and b = 5	|Y，使用到a，b|
|where a = 3 and b = 5 and c = 4|Y，使用到a，b，c|
|where b = 3 或者 where b = 3 and c = 4 或者 where c = 4|Y，使用到|
|where a = 3 and c = 5|Y，使用到a但是c不可以，b中间断了|
|where a = 3 and b > 4 and c = 5|Y，使用到a使用到c不能用在范围之后，b断了|
|where a = 3 and b like ‘kk%’ and c = 4|Y，使用到a，b，c|
## 口诀
全职匹配我最爱，最左前缀要遵守；
带头大哥不能死，中间兄弟不能断；
索引列上少计算，范围之后全失效；
LIKE 百分写最右，覆盖索引不写*；
不等空值还有OR，索引影响要注意；
VAR 引号不可丢，SQL 优化有诀窍。







group by基本上都需要进行排序，会有临时表产生

## 一般性建议

- 对于单键索引，尽量选择针对当前query过滤性更好的索引。
- 在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。
- 在选择组合索引的时候，尽量选择可以能够包含当前query中的where字句中更多字段的索引。
- 尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的。
  

# 查询截取分析
## 小表驱动大表

通常SQL调优过程：

1. 观察，至少跑1天，看看生产的慢SQL情况。
2. 开启慢查询日志，设置阙值，比如超过5秒钟的就是慢SQL，并将它抓取出来。
3. explain + 慢SQL分析。
4. show profile。
5. 运维经理 or DBA，进行SQL数据库服务器的参数调优。



总结：

1. 慢查询的开启并捕获
2. explain + 慢SQL分析
3. show profile查询SQL在Mysql服务器里面的执行细节和生命周期情况
4. SQL数据库服务器的参数调优。


优化原则：==小表驱动大表，即小的数据集驱动大的数据集。==



**RBO原理**

```
select * from A where id in (select id from B)
等价于:
for select id from B
for select * from A where A.id = B.id
```

**当B表的数据集必须小于A表的数据集时，用in优于exists。**

```
select * from A where exists (select 1 from B where B.id = A.id)
等价于：
for select * from A
for select * from B where B.id = A.id
```


**当A表的数据集系小于B表的数据集时，用exists优于in。**

注意：A表与B表的ID字段应建立索引。

**EXISTS关键字**

```
SELECT ...FROM table WHERE EXISTS (subquery)
```


该语法可以理解为：将主查询的数据，放到子查询中做条件验证，根据验证结果（TRUE或FALSE）来决定主查询的数据结果是否得以保留。

提示

1. EXSTS(subquey)只返回TRUE或FALSE，因此子查询中的SELECT * 也可以是 SELECT 1 或select ‘X’，官方说法是实际执行时会忽略SELECT清单，因此没有区别。
2. EXISTS子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担忧效率问题，可进行实际检验以确定是否有效率问题。
3. EXISTS子查询往往也可以用条件表达式，其他子查询或者JOIN来替代，何种最优需要具体问题具体分析



参考：https://www.cnblogs.com/emilyyoucan/p/7833769.html



## 为排序使用索引OrderBy优化

ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序



MySQL支持二种方式的排序，`FileSort`和`Index`，Index效率高，它指MySQL扫描索引本身完成排序。FileSort方式效率较低。

ORDER BY满足两情况，会使用Index方式排序：

1. **ORDER BY语句使用索引最左前列**。
2. **使用where子句与Order BY子句条件列组合满足索引最左前列**。



```sql
mysql> EXPLAIN SELECT * FROM tblA where age > 20 order by age;

mysql> EXPLAIN SELECT * FROM tblA where age>20 order by age,birth;



#Using filesort:

mysql> EXPLAIN SELECT * FROM tblA where age>20 order by birth;

mysql> EXPLAIN SELECT * FROM tblA where age>20 order by birth,age;
```



如果不在索引列上，mysql的filesort有两种算法：

- 双路排序
- 单路排序



### **双路排序**

MySQL4.1之前是使用双路排序，字面意思就是两次扫描磁盘，最终得到数据，读取行指针和OrderBy列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读对应的数据输出。

从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。

取一批数据，要对磁盘进行了两次扫描，众所周知，I\O是很耗时的，所以在mysql4.1之后，出现了第二种改进的算法，就是单路排序。



### **单路排序**

从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序压的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO,但是它会使用更多的空间，因为它把每一行都保存在内存中了。

### **结论及引申出的问题**

由于单路是后出的，总体而言好过双路

但是用单路有问题

在sort_buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出,所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并)，排完再取取
sort_buffer容量大小，再排……从而多次I/O。

本来想省一次I/O操作，反而导致了大量的I/O操作，反而得不偿失。



单路排序会把所有需要查询的字段都放到 sort buffer 中，而双路排序只会把主键 和需要排序的字段放到 sort buffer 中进行排序，然后再通过主键回到原表查询需要的字段



### **优化策略**

- 增大sort_buffer_size参数的设置
- 增大max_length_for_sort_data参数的设置



**为什么设置sort_buffer_size、max_length_for_sort_data参数能优化排序？**

提高Order By的速度

1. Order by时select * 是一个大忌只Query需要的字段，这点非常重要。在这里的影响是;
   - 当Query的字段大小总和小于max_length_for_sort_data而且排序字段不是TEXT|BLOB类型时，会用改进后的算法——单路排序，否则用老算法——多路排序。
   - 两种算法的数据都有可能超出sort_buffer的容量，超出之后，会创建tmp文件进行合并排序，导致多次IO，但是用单路排序算法的风险会更大一些，所以要提高sort_buffer__size。
     

2. 尝试提高sort_buffer_size，不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的。
3. 尝试提高max_length_for_sort_data，提高这个参数，会增加用改进算法的概率。但是如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘I/O活动和低的处理器使用率。
   

### **小结**
为排序使用索引

- MySql两种排序方式∶文件排序 或 扫描有序索引排序
- MySql能为 排序 与 查询 使用相同的索引
  

### 案例

创建复合索引 a_b_c (a, b, c)

order by能使用索引最左前缀

- ORDER BY a
- ORDER BY a, b
- ORDER BY a, b, c
- ORDER BY a DESC, b DESC, c DESC
  

如果WHERE使用素引的最左前缀定义为常量，则order by能使用索引

- WHERE a = const ORDER BY b,c
- WHERE a = const AND b = const ORDER BY c
- WHERE a = const ORDER BY b, c
- WHERE a = const AND b > const ORDER BY b, c



不能使用索引进行排序

- ORDER BY a ASC, b DESC, c DESC //排序不—致
- WHERE g = const ORDER BY b, c //产丢失a索引
- WHERE a = const ORDER BY c //产丢失b索引
- WHERE a = const ORDER BY a, d //d不是素引的一部分
- WHERE a in (…) ORDER BY b, c //对于排序来说,多个相等条件也是范围查询
  

## GroupBy优化与慢查询日志
**GroupBy优化**

- group by实质是先排序后进行分组，遵照索引建的最佳左前缀。
- 当无法使用索引列，增大max_length_for_sort_data参数的设置 + 增大sort_buffer_size参数的设置。
- where高于having，能写在where限定的条件就不要去having限定了。

**慢查询日志**

- MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。
- 具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的默认值为10，意思是运行10秒以上的语句。
- 由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。



**如何操作**
默认情况下，MySQL数据库没有开启慢查询日速，需要我们手动来设置这个参数。

当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件。


**查看是否开启及如何开启**

- 默认 - `SHOW VARIABLES LIKE '%slow_query_log%';`
- 开启 - `set global slow_query_log=1;`，只对当前数据库生效，如果MySQL重启后则会失效。



如果要**永久生效**，就必须修改配置文件my.cnf(其它系统变量也是如此)

修改my.cnf文件，[mysqld]下增加或修改参数slow_query_log和slow_query_log_file后，然后重启MySQL服务器。也即将如下两行配置进my.cnf文件

```
slow_query_log =1
slow_query_log_file=/var/lib/mysqatguigu-slow.log
```

关于慢查询的参数slow_query_log_file，它指定慢查询日志文件的存放路径，系统默认会给一个缺省的文件host_name-slow.log（如果没有指定参数slow_query_log_file的话）

**开启了慢查询日志后，什么样的SQL才会记录到慢查询日志里面呢？**

这个是由参数long_query_time控制，默认情况下long_query_time的值为10秒，命令：`SHOW VARIABLES LIKE 'long_query_time%';`


可以使用命令修改，也可以在my.cnf参数里面修改。

假如运行时间正好等于long_query_time的情况，并不会被记录下来。也就是说，在mysql源码里是判断大于long_query_time，而非大于等于。

设置慢SQL阈值时间：`set global long_query_time=3;`


**为什么设置后看不出变化？**

需要重新连接或新开一个会话才能看到修改值。

**记录慢SQL并后续分析**

假设我们成功设置慢SQL阈值时间为3秒（`set global long_query_time=3;`）。

模拟超时SQL：

```
mysql> SELECT sleep(4);
+----------+
| sleep(4) |
+----------+
|        0 |
+----------+
1 row in set (4.00 sec)

```

日志记录：

![](..\..\pics\database\Mysql\slow.png)



**查询当前系统中有多少条慢查询记录**

`mysql> show global status like '%Slow_queries%';`

**在配置文件中设置慢SQL阈值时间**

```
#[mysqld]下配置:
slow_query_log=1;
slow_query_log_file=/var/lib/mysql/atguigu-slow.log
long_query_time=3;
log_output=FILE;

```



**日志分析工具mysqldumpslow**

在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具mysqldumpslow。

查看mysqldumpslow的帮助信息，`mysqldumpslow --help`。



- s：是表示按照何种方式排序
- c：访问次数
- l：锁定时间
- r：返回记录
- t：查询时间
- al：平均锁定时间
- ar：平均返回记录数
- at：平均查询时间
- t：即为返回前面多少条的数据
- g：后边搭配一个正则匹配模式，大小写不敏感的



**工作常用参考**



- 得到返回记录集最多的10个SQL：

  `mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log`

- 得到访问次数最多的10个SQL：

  `mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log`

- 得到按照时间排序的前10条里面含有左连接的查询语句：

  `mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log`

- 另外建议在使用这些命令时结合│和more 使用，否则有可能出现爆屏情况：

  `mysqldumpslow -s r-t 10 /ar/lib/mysql/atguigu-slow.log | more`


## 用Show Profile进行sql分析

Show Profile是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量。

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/show-profile.html)

默认情况下，参数处于关闭状态，并保存最近15次的运行结果。

**分析步骤**

1.**是否支持**，看看当前的mysql版本是否支持。

`mysql> show variables like 'profiling';`

默认是关闭，使用前需要开启。

2.开启功能，默认是关闭，使用前需要开启。

`set profiling=on;`

3.**运行SQL**

4.**查看结果**，`show profiles;`

```
mysql> show profiles;
+----------+------------+-----------------------------------------------+
| Query_ID | Duration   | Query                                         |
+----------+------------+-----------------------------------------------+
|        1 | 0.00204000 | show variables like 'profiling'               |
|        2 | 0.55134250 | select * from emp group by id%10 limit 150000 |
|        3 | 0.56902000 | select * from emp group by id%20 order by 5   |
+----------+------------+-----------------------------------------------+
3 rows in set, 1 warning (0.00 sec)

```



5.诊断SQL，`show profile cpu,block io for query 上一步前面的问题SQL数字号码;`

`mysql> show profile cpu,block io for query 3;`

参数备注

- ALL：显示所有的开销信息。
- BLOCK IO：显示块lO相关开销。
- CONTEXT SWITCHES ：上下文切换相关开销。
- CPU：显示CPU相关开销信息。
- IPC：显示发送和接收相关开销信息。
- MEMORY：显示内存相关开销信息。
- PAGE FAULTS：显示页面错误相关开销信息。
- SOURCE：显示和Source_function，Source_file，Source_line相关的开销信息。
- SWAPS：显示交换次数相关开销的信息。



6.日常开发需要注意的结论

- converting HEAP to MyISAM 查询结果太大，内存都不够用了往磁盘上搬了。
- Creating tmp table 创建临时表，拷贝数据到临时表，用完再删除
- Copying to tmp table on disk 把内存中临时表复制到磁盘，危险!
  locked


## 全局查询日志

永远不要在生产环境开启这个功能。

配置文件启用。在mysql的my.cnf中，设置如下

```
#开启
general_log=1
#记录日志文件的路径
general_log_file=/path/logfile
#输出格式
log_output=FILE

```

编码启用。命令如下：

- `set global general_log=1;`
- `set global log_output='TABLE';`

此后，你所编写的sql语句，将会记录到mysql库里的geneial_log表，可以用下面的命令查看

`mysql> select * from mysql.general_log;`

# MySQL锁机制
## 数据库锁理论概述
锁是计算机协调多个进程或线程并发访问某一资源的机制。

在数据库中，除传统的计算资源（如CPU、RAM、I/O等）的争用以外**，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素**。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

**类比：网上购物**

打个比方，我们到淘宝上买一件商品，商品只有一件库存，这个时候如果还有另一个人买，那么如何解决是你买到还是另一个人买到的问题？

这里肯定要用到事务，我们先从库存表中取出物品数量，然后插入订单，付款后插入付款表信息，然后更新商品数量。在这个过程中，使用锁可以对有限的资源进行保护，解决隔离和并发的矛盾。

**锁的分类**

从对数据操作的类型（读\写）分

- 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。
- 写锁（排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。
  

从对数据操作的粒度分

- 表锁
- 行锁
  

**表锁（偏读）**

特点：偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。

手动增加表锁

```
lock table 表名字 read(write), 表名字2 read(write), 其他;
```



```
mysql> lock table mylock read;
Query OK, 0 rows affected (0.00 sec)
```

查看表上加过的锁

`mysql> show open tables;`

释放锁

`mysql> unlock tables;`



## 结论

MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁。

MySQL的表级锁有两种模式：

1. 表共享读锁(Table Read Lock)
2. 表独占写锁(Table Write Lock)




| 锁类型 | 可否兼容 | 读锁 | 写锁 |
| ------ | -------- | ---- | ---- |
| 读锁   | 是       | 是   | 否   |
| 写锁   | 是       | 否   | 否   |

结合上表，所以对MyISAM表进行操作，会有以下情况：

1. 对MyISAM表的读操作（加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求。只有当读锁释放后，才会执行其它进程的写操作。
2. 对MyISAM表的写操作〈加写锁），会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其它进程的读写操作。
   

**简而言之，就是读锁会阻塞写，但是不会堵塞读。而写锁则会把读和写都堵塞。**

**表锁分析**

*看看哪些表被加锁了*

```
mysql> show open tables;
```

*如何分析表锁定*

可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定。

```
mysql>  show status like 'table_locks%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 170   |
| Table_locks_waited    | 0     |
+-----------------------+-------+
2 rows in set (0.00 sec)

```

这里有两个状态变量记录MySQL内部表级锁定的情况，两个变量说明如下：

- Table_locks_immediate：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1 ;
- Table_locks_waited：出现表级锁定争用而发生等待的次数(不能立即获取锁的次数，每等待一次锁值加1)，此值高则说明存在着较严重的表级锁争用情况；

此外，MyISAM的读写锁调度是写优先，这也是MyISAM不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。



## 行锁理论
偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

InnoDB与MyISAM的最大不同有两点：一是支持事务(TRANSACTION)；二是采用了行级锁。

**由于行锁支持事务，复习老知识**

- 事务(Transaction）及其ACID属性
- 并发事务处理带来的问题
- 事务隔离级别



事务是由一组SQL语句组成的逻辑处理单元，事务具有以下4个属性，通常简称为事务的ACID属性：**略**，参考：https://blog.csdn.net/weixin_46168350/article/details/117229761



常看当前数据库的事务隔离级别：`show variables like 'tx_isolation';`

```
mysql> show variables like 'tx_isolation';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
1 row in set, 1 warning (0.00 sec)
```



## 总结

读写锁：表锁

事务：行锁

## 索引失效行锁变表锁

无索引行锁升级为表锁

原因之网友：Mysql 的行锁是通过索引实现的，行锁锁的是索引，没有索引就不行了举报

## 间隙锁危害

![](..\..\pics\database\Mysql\间隙锁.png)

**什么是间隙锁**

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁，对于键值在条件范围内但**并不存在**的记录，叫做“间隙（GAP）”。

InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。

**危害**

因为Query执行过程中通过过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。

间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害。


## 如何锁定一行

```
begin;
select * from test_db where a=8 for update
commit;
```



## 行锁总结与页锁
Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，Innodb的整体性能和MylISAM相比就会有比较明显的优势了。

但是，Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让Innodb的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

**行锁分析**

如何分析行锁定
通过检查lnnoDB_row_lock状态变量来分析系统上的行锁的争夺情况

```
mysql> show status like 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 0     |
| Innodb_row_lock_time_avg      | 0     |
| Innodb_row_lock_time_max      | 0     |
| Innodb_row_lock_waits         | 0     |
+-------------------------------+-------+
5 rows in set (0.00 sec)
```


对各个状态量的说明如下:

- Innodb_row_lock_current_waits：当前正在等待锁定的数量；
- Innodb_row_lock_time：从系统启动到现在锁定总时间长度；
- Innodb_row_lock_time_avg：每次等待所花平均时间；
- Innodb_row_lock_time_max：从系统启动到现在等待最常的一次所花的时间；
- Innodb_row_lock_waits：系统启动后到现在总共等待的次数；



对于这5个状态变量，比较重要的主要是

- Innodb_row_lock_time_avg（等待平均时长）
- lnnodb_row_lock_waits（等待总次数）
- lnnodb_row_lock_time（等待总时长）这三项。

尤其是当等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手指定优化计划。

**优化建议**

1. 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。
2. 合理设计索引，尽量缩小锁的范围
3. 尽可能较少检索条件，避免间隙锁
4. 尽量控制事务大小，减少锁定资源量和时间长度
5. 尽可能低级别事务隔离

**页锁**

开销和加锁时间界于表锁和行锁之间;会出现死锁;锁定粒度界于表锁和行锁之间，并发度一般。（了解一下即可）


# 其他知识

```
#创建用户
create user "ming"@"%" identified by "xxxx";
#修改密码
alter user'root'@'%' IDENTIFIED BY '';

# 授予权限
grant all PRIVILEGES on *.*  to 'ming"@"%'  

grant all privileges on database_name.* to 'username'@'localhost';

GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'ming'@'%' WITH GRANT OPTION;

#启动： sudo service mysql start

#关闭： sudo service mysql stop


#步骤：

mysql -u root -p

SHOW DATABASES;

use mysql;

RENAME USER 'root'@'localhost' TO 'root'@'%';

1.update user set host='%' where user='root' limit 1(两种方法都可以，选一个)

2.grant all privileges on.to ‘root’@’%’ identified by ‘abc.123’ with grant option;

flush privileges;

select user,authentication_string,host from user;
```



## 其他配置

**查看字符集**

- `show variables like 'character%";`
- `show variables like ‘%char%";`

**数据文件**

两系统

- windows
  输入mysql后select @@database;

- linux
  默认路径：/var/lib/mysql



frm文件（form）

- 存放表结构



myd文件（my data）

- 存放表数据



myi文件（my index）

- 存放表索引
  
  



```
1、加入开机启动
systemctl enable mysqld

2、启动mysql服务进程
systemctl start mysqld

# 设置开机启动mysql
[root@VM-16-2-centos ~]# chkconfig mysql on

# 2,3,4,5 启动就可以
[root@VM-16-2-centos ~]# chkconfig --list|grep mysql

# 看到[*]mysql 表示开机会后会启动mysql
[root@VM-16-2-centos ~]# ntsysv

```

```
如何配置

Windows - my.ini文件
Linux - /etc/my.cnf文件
```





