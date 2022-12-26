# Hive 基本概念

## 1.1 什么是Hive

1. hive 简介

**Hive：**由Facebook 开源用于解决海量==结构化==日志的数据统计工具。
Hive 是基于Hadoop 的一个==数据仓库工具==，可以将==结构化的数据文件映射为一张表==，并提供类SQL 查询功能。

2. Hive 本质：==将HQL 转化成MapReduce 程序==

（1）Hive 处理的**数据存储在HDFS**
（2）Hive 分析数据**底层的实现是MapReduce**
（3）执行**程序运行在Yarn 上**

## 1.2 Hive 的优缺点

**1.2.1 优点**
（1）==操作接口采用类SQL 语法==，提供快速开发的能力（简单、容易上手）。

（2）==避免了去写MapReduce==，减少开发人员的学习成本。
（3）Hive 的==执行延迟比较高==，因此Hive 常用于数据分析，对实时性要求不高的场合。
（4）Hive 优势在==于处理大数据==，对于处理小数据没有优势，因为Hive 的执行延迟比较高。
（5）Hive 支持==用户自定义函数==，用户可以根据自己的需求来实现自己的函数。

**1.2.2 缺点**
1）Hive 的HQL 表达能力有限
（1）==迭代式算法无法表达== for循环
（2）数据挖掘方面不擅长，由于MapReduce 数据处理流程的限制，==效率更高的算法却无法实现。==
2）Hive 的效率比较低
（1）Hive 自动生成的MapReduce 作业，==通常情况下不够智能化==
（2）Hive ==调优比较困难，粒度较粗==



## 1.3 Hive 架构原理

![](..\..\pics\bigdata\hive\hive架构.PNG)

1. **用户接口：Client**
   CLI（command-line interface）、JDBC/ODBC(jdbc 访问hive)、WEBUI（浏览器访问hive）

2. **元数据：Metastore**
   元数据包括：**表名**、表所属的**数据库**（默认是default）、表的拥有者、**列/分区字段**、**表的类型**（是否是外部表）、**表的数据所在目录等**；
   ==默认存储在自带的derby 数据库中，推荐使用MySQL 存储Metastore==
3. **Hadoop**
   ==使用HDFS 进行存储==，使用==MapReduce 进行计算==。
4. **驱动器：Driver**
   （1）**解析器（SQL Parser）**：将SQL 字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST 进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
   （2）**编译器（Physical Plan）**：将AST 编译生成逻辑执行计划。
   （3）**优化器（Query Optimizer）**：对逻辑执行计划进行优化。
   （4）**执行器（Execution）**：把逻辑执行计划转换成可以运行的物理计划。对于Hive 来说，就是MR/Spark。

## 1.4 hive和数据库的区别

（1）**数据存储位置不同**

Hive中处理的结构化数据**存储在HDFS中**，元数据存储在mysql的Meta store中；

数据库将数据**保存在块设备或本地文件系统中；**

（2）数据更新

由于Hive 是针对数据仓库应用设计的，而数据仓库的内容==是读多写少的==。因此，==Hive 中**不建议对数据的改写**，**所有的数据都是在加载的时候确定好的**==。而数据库中的数据通常是需要经常进行修改的，因此可以使用 INSERT INTO …  VALUES 添加数据，使用 UPDATE … SET 修改数据。



Hive是针对数据仓库设计的，**主要用于读，所有的数据在加载时已经确定好，适合处理静态数据；**

数据库通常是**实时进行修改的，增删改查，适合处理动态数据；**

（3）执行机制

Hive大多数查询的执行是**通过Hadoop提供的MapReduce实现的**；

数据库通常是用自**己的引擎innodb；**

（4）执行延迟

Hive因为没有索引、利用MapReduce框架执行查询，**所以Hive本身的延迟较高；**

**数据库的延迟较低，但是不太适合处理PB级别以上海量数据；**

处理海量数据时，Hive的优势就显出来了；



当然，这个低是有条件的，即数据规模较小，==当数据规模大到超过数据库的处理能力的时候，**Hive 的并行计算**显然能体现出优势。== 

（5）可扩展性

Hive是建立在Hadoop上的，所以Hive也具备可扩展性，并发运行；

数据库由于ACID语义的严格限制，扩展性非常有限，例如目前最先进的并行数据库oracle在理论上扩展能力也就只有100台左右。

==由于**Hive 建立在集群上并可以利用 MapReduce 进行并行计算**==，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。 



==很明显，除了都用sql语句，Hive和数据库其实没啥太大关系==



# Hive数据类型

## 基本数据类型 



| Hive           | Java数据类型 | 长度                | 例子                                  |
| -------------- | ------------ | ------------------- | ------------------------------------- |
| TINYINT        | byte         | 1byte有符号整数     | 20                                    |
| SMALINT  short |              | 2byte有符号整数     | 20                                    |
| INT            | int          | 4byte有符号整数     | 20                                    |
| BIGINT         | long         | 8byte 有符号整数    | 20                                    |
| BOOLEAN        | boolean      | 布尔类型，true 或者 | TRUE、FALSE                           |
| FLOAT          | float        | 单精度浮点数        | 3.14159                               |
| DOUBLE         | double       | 双精度浮点数        | 3.14159                               |
| STRING         | string       | 字符系列。          | “now is the time ’ “for all good men” |
| TIMESTAMP      |              | 时间类型            |                                       |
| BINARY         |              | 字节数组            |                                       |

  ‘

对于==Hive 的 String 类型相当于数据库的varchar 类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB 的字符数==。

## 集合数据类型

| 数据类型 | 描述                                                         | 语法示例                                           |
| -------- | ------------------------------------------------------------ | -------------------------------------------------- |
| STRUCT   | 和c 语言中的 struct 类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是 STRUCT{first STRING, last STRING},**那么第 1 个元素可以通过字段.first 来引用。** | struct()例如：`struct<street:string, city:string>` |
| MAP      | MAP 是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是 MAP，其中键 ->值对是’first’->’John’和’last’->’Doe’，那么可以**通过字段名[‘last’]获取最后一个元素** | `map() `<br/>例如 `map<string, int>`               |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’, ‘Doe’]，**那么第 2 个元素可以通过数组名[1]进行引用。** | `Array()` <br/>例如 `array<string>`                |



hive 有三种复杂数据类型 ARRAY、MAP 和 STRUCT。ARRAY 和 MAP 与 Java 中的 Array和Map 类似，而 STRUCT 与C 语言中的Struct 类似，它封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套。 

## 实操

（1）假设某表有如下一行，我们用 我们用 JSON格式来表示其数据结构。在 Hive下访问的格式为 

```json
	{
    "name": "songsong",
    "friends": [
        "bingbing",
        "lili"
    ],
    "students": {
        "xiaohaihai": 18,
        "xiaoyangyang": 19
    },
    "address": {
        "street": "hui long guan",
        "city": "beijing"
    }
}
```



（2）**基于上述数据结构**  ，我们在Hive里创建对应的表 ，并导入数据  。
创建本地测试文件test.txt

```
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing 
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing 
```

注意：MAP，STRUCT 和 ARRAY 里的元素间关系都可以用同一个字符表示，这里用“_”。 

（3）**Hive 上创建测试表test**

```
create table test( 
name String,
friends array<String>,
children map<string, int>,
address struct<street:string, city:string> 
)
row format delimited fields terminated by ',' 
collection items terminated by '_' 
map keys terminated by ':' 
lines terminated by '\n'; 
```

字段解释： 

> `row format delimited fields terminated by` ','  -- 列分隔符 
>
> `collection items terminated by '_' `    --MAP STRUCT 和 ARRAY 的分隔符(数据分割符号) 
>
> `map keys terminated by ':'`     -- MAP 中的key 与value 的分隔符 
> lines terminated by '\n';  -- 行分隔符 

（4）	导入文本数据到测试表
`load data local inpath '/opt/module/hive/datas/test.txt' into table test;`
（5）	访问三种集合列里的数据，以下分别是 ARRAY，MAP，STRUCT 的访问方式

`hive (default)> select friends[1],children['xiao song'],address.city from test where name="songsong"; `

## 类型转化 

Hive 的原子数据类型是可以进行隐式转换的，类似于 Java 的类型转换，例如某表达式使用INT 类型，==TINYINT 会自动转换为INT 类型==，但是==Hive 不会进行反向转化==，例如，某表达式使用 TINYINT 类型，INT 不会自动转换为 TINYINT 类型，它会返回错误，除非使用 CAST操作。 

​	**隐式类型转换规则如下**
a. 任何整数类型都可以隐式地转换为一个范围更广的类型，如tinyint可以转换成int，int可以转换成bigint。

b. 所有整数类型、**float和string类型**都可以隐式地转换成double。

c. tinyint、smallint、int都可以转换为float。

d. boolean类型不可以转换为任何其它的类型。

详情可参考Hive官方说明：[Allowed Implicit Conversions](https://cwiki.apache.org/confluence/display/hive/languagemanual+types#LanguageManualTypes-AllowedImplicitConversions)

**显示转换**

可以借助cast函数完成显示的类型转换

a.语法

`cast(expr as <type>) `

b.案例

```
hive (default)> select '1' + 2, cast('1' as int) + 2;

_c0   _c1
3.0   3
```

# DDL（Data Definition Language）数据定义

## 数据库（database）

### 创建数据库

**1）语法**



> ==CREATE DATABASE== [IF NOT EXISTS] database_name
>
> ==[COMMENT== database_comment]
>
> [==LOCATION== hdfs_path]
>
> [==WITH DBPROPERTIES== (property_name=property_value, ...)];

**2）案例**

**（1）创建一个数据库，不指定路径**

`hive (default)> create database db_hive1;`

注：若不指定路径，其默认路径为`${hive.metastore.warehouse.dir}/database_name.db`

**（2）创建一个数据库，指定路径**

`hive (default)> create database db_hive2 location '/db_hive2';`

**（2）创建一个数据库，带有dbproperties**

`hive (default)> create database db_hive3 with dbproperties('create_date'='2022-11-18');`

###  查询数据库

**1）展示所有数据库**

**（1）语法**

`SHOW DATABASES [LIKE 'identifier_with_wildcards'];`

注：like通配表达式说明：*表示任意个任意字符，|表示或的关系。

**（2）案例**

```
hive> show databases like 'db_hive*';
OK
db_hive_1
db_hive_2
```

**2）查看数据库信息**

**（1）语法**

`DESCRIBE DATABASE [EXTENDED] db_name;`

**（2）案例**

**1查看基本信息**

```
hive> desc database db_hive3;
OK
db_hive   hdfs://hadoop102:8020/user/hive/warehouse/db_hive.db  atguigu  USER
```

**2查看更多信息**

```
hive> desc database extended db_hive3;
OK
db_name  comment  location owner_name  owner_type  parameters
db_hive3   hdfs://hadoop102:8020/user/hive/warehouse/db_hive3.db atguigu  USER  {create_date=2022-11-18}
```

### 修改数据库

用户可以使用`alter database`命令修改数据库某些信息，其中能够修改的信息包括`dbproperties`、`location`、`owner user`。需要注意的是：==修改数据库location，不会改变当前已有表的路径信息，而只是改变后续创建的新表的默认的父目录。==

**1）语法**

```sql
--修改dbproperties
ALTER DATABASE database_name SET DBPROPERTIES (property_name=property_value, ...);

--修改location
ALTER DATABASE database_name SET LOCATION hdfs_path;

--修改owner user
ALTER DATABASE database_name SET OWNER USER user_name;
```



**2）案例**

```sql
（1）修改dbproperties
hive> ALTER DATABASE db_hive3 SET DBPROPERTIES ('create_date'='2022-11-20');
```



### 删除数据库

1）语法

DROP DATABASE [IF EXISTS] database_name [RESTRICT|CASCADE];

注：

==RESTRICT：严格模式==，若数据库不为空，则**会删除失败，默认为该模式。**

  ==CASCADE：级联模式==，若数据库不为空，**则会将库中的表一并删除。**

2）案例

（1）删除空数据库

`hive> drop database db_hive2;`

（2）删除非空数据库

`hive> drop database db_hive3 cascade;`

###  切换当前数据库

1）语法

`USE database_name;`

## 表（table）

### ==创建表==

 语法

1）普通建表

（1）完整语法

> ==CREATE== [==TEMPORARY==] [==EXTERNAL==] ==TABLE==[IF NOT EXISTS] [db_name.]table_name 
>
> [(col_name ==data_type== [==COMMENT== col_comment], ...)]
>
> [==COMMENT== table_comment]
>
> [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
>
> [CLUSTERED BY (col_name, col_name, ...) 
>
> [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
>
> [==ROW FORMAT== row_format] 
>
> [==STORED AS== file_format]
>
> [==LOCATION== hdfs_path]
>
> [==TBLPROPERTIES== (property_name=property_value, ...)]

（2）关键字说明：

1. **TEMPORARY**

临时表，该表只在当前会话可见，会话结束，表会被删除。

2. **EXTERNAL（重点）**

外部表，与之相对应的是内部表（管理表）。==管理表意味着Hive会完全接管该表，包括元数据和HDFS中的数据==。而外部表则意味着==**Hive只接管元数据，而不完全接管HDFS中的数据**。==

3. **data_type（重点） 数据类型（略）**



4. **PARTITIONED BY（重点）**

创建分区表

5. **CLUSTERED BY ... SORTED BY...INTO ... BUCKETS（重点）**

创建分桶表

6. **ROW FORMAT（重点）**

指定SERDE，SERDE是Serializer and Deserializer的简写。Hive使用SERDE序列化和反序列化每行数据。详情可参考 [Hive-Serde](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)。语法说明如下：

语法一：DELIMITED关键字表示对文件中的每个字段按照特定分割符进行分割，其会使用**默认的SERDE**对每行数据进行序列化和反序列化。

> ROW FORAMT ==DELIMITED== 
>
> [==FIELDS TERMINATED BY== char]
>
> [==COLLECTION ITEMS TERMINATED BY== char]
>
> [==MAP KEYS TERMINATED BY== char]
>
> [==LINES TERMINATED BY== char]
>
> [==NULL DEFINED AS== char]

注：

- `fields terminated by` ：列分隔符
-  `collection items terminated by` ： map、struct和array中每个元素之间的分隔符
-  `map keys terminated by` ：map中的key与value的分隔符
- `lines terminated by `：行分隔符

语法二：SERDE关键字可用于==指定其他内置的SERDE==或者==用户自定义的SERDE==。例如==JSON SERDE==，可用于处理JSON字符串。

> ROW FORMAT ==SERDE== serde_name [==WITH SERDEPROPERTIES==(property_name=property_value,property_name=property_value, ...)] 

7. **STORED AS（重点）**

指定文件格式，常用的文件格式有，textfile（默认值），sequence file，orc file、parquet file等等。

8. **LOCATION**

指定表所对应的HDFS路径，若不指定路径，其默认值为

${hive.metastore.warehouse.dir}/db_name.db/table_name

9. **TBLPROPERTIES**

用于配置表的一些KV键值对参数

**2）==Create Table As Select==（CTAS）建表**

该语法允许用户利用select查询语句返回的结果，直接建表，表的结构和查询语句的结构保持一致，且保证包含select查询语句放回的内容。

**CREATE [TEMPORARY] TABLE [IF NOT EXISTS] table_name**

**[COMMENT** table_comment] 

**[ROW FORMAT** row_format] 

**[STORED AS** file_format] 

**[LOCATION** hdfs_path]

**[TBLPROPERTIES** (property_name=property_value, ...)]

**[AS** select_statement]

**3）==Create Table Like==语法**

该语法允许用户复刻一张已经存在的表结构，与上述的CTAS语法不同，==该语法创建出来的表中不包含数据。==

**CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS]** [db_name.]table_name

**[LIKE** exist_table_name]

**[ROW FORMAT** row_format] 

**[STORED AS** file_format] 

**[LOCATION** hdfs_path]

**[TBLPROPERTIES** (property_name=property_value, ...)]

 **案例**

**1）内部表与外部表**

**（1）内部表**

Hive中默认创建的表都是的==内部表==，有时也被称为==管理表==。对于内部表，==Hive会完全管理表的元数据和数据文件。==

**（2）外部表**

​     外部表通常可用于处理其他工具上传的数据文件，对于外部表，==Hive只负责管理元数据，不负责管理HDFS中的数据文件。==

**2）SERDE和复杂数据类型**

本案例重点练习SERDE和复杂数据类型的使用。

若现有如下格式的JSON文件需要由Hive进行分析处理，请考虑如何设计表？

注：以下内容为格式化之后的结果，文件中每行数据为一个完整的JSON字符串。

```json
{
  "name": "dasongsong",
  "friends": [
    "bingbing",
    "lili"
  ],
  "students": {
    "xiaohaihai": 18,
    "xiaoyangyang": 16
  },
  "address": {
    "street": "hui long guan",
    "city": "beijing",
    "postal_code": 10010
  }
}
```

我们可以考虑使用专门负责JSON文件的JSON Serde，设计表字段时，表的字段与JSON字符串中的一级字段保持一致，对于具有嵌套结构的JSON字符串，考虑使用合适复杂数据类型保存其内容。最终设计出的表结构如下：

```sql
hive>
create table teacher
 (
   name   string,
   friends array<string>,
   students map<string,int>,
   address struct<city:string,street:string,postal_code:int>
 )
 row format serde 'org.apache.hadoop.hive.serde2.JsonSerDe'
location '/user/hive/warehouse/teacher';
```

创建该表，并准备以下文件。注意，需要确保文件中==每行数据都是一个完整的JSON字符串，`JSON SERDE`才能正确的处理。==

```
[atguigu@hadoop102 datas]$ vim /opt/module/datas/teacher.txt

{"name":"dasongsong","friends":["bingbing","lili"],"students":{"xiaohaihai":18,"xiaoyangyang":16},"address":{"street":"hui long guan","city":"beijing","postal_code":10010}}
```



### 查看表

**1）展示所有表**

（1）语法

`SHOW TABLES== [IN database_name] LIKE ['identifier_with_wildcards'];`

注：like通配表达式说明：*表示任意个任意字符，|表示或的关系。

（2）案例

`hive> show tables like 'stu*';`

**2）查看表信息**

（1）语法

`DESCRIBE [EXTENDED | FORMATTED] [db_name.]table_name`

==**注：**==

- **EXTENDED：**展示详细信息

-  **FORMATTED**：对详细信息进行格式化的展示

**（2）案例**

**1查看基本信息**

hive> desc stu;

**2查看更多信息**

hive> desc formatted stu;

### ==修改表==

**1）重命名表**

（1）语法

ALTER TABLE table_name RENAME TO new_table_name

（2）案例

hive (default)> alter table stu rename to stu1;

**2）修改列信息**

（1）语法

1. **==增加列==**

该语句允许用户增加新的列，新增列的位置位于末尾。

`ALTER TABLE `table_name `ADD COLUMNS` (col_name data_type [`COMMENT` col_comment], ...)

2. ==更新列==

该语句允许用户修改指定列的列名、数据类型、注释信息以及在表中的位置。

`ALTER TABLE` table_name `CHANGE` [`COLUMN`] col_old_name col_new_name column_type [`COMMENT` col_comment] [`FIRST|AFTER` column_name]

3. **==替换列==**

该语句允许用户用新的列集替换表中原有的全部列。

`ALTER TABLE` table_name `REPLACE COLUMNS` (col_name data_type [COMMENT col_comment], ...)

**2）案例**

**（1）查询表结构**

`hive (default)> desc stu;`

**（2）添加列**

`hive (default)> alter table stu add columns(age int);`

**（3）查询表结构**

`hive (default)> desc stu;`

**（4）更新列**

`hive (default)> alter table stu change column age ages double;`

**（6）替换列**

`hive (default)> alter table stu replace columns(id int, name string);`

###  删除表

1）语法

DROP TABLE [IF EXISTS] table_name;

2）案例

hive (default)> drop table stu;

注意如果库里有表会报错

解决这个错误有两种方法：

一、就是很简单的将所有表先删除完，再删除库。

另外一种就是使用下述的方法：使用cascade关键字执行强制删库。drop database if exists 库名 cascade;



### 清空表

1）语法

`TRUNCATE [TABLE] table_name`

注意：truncate只能清空管理表，不能删除外部表中数据。

2）案例

hive (default)> truncate table student;

# DML（Data Manipulation Language）数据操作

##  ==Load==

**Load语句可将文件导入到Hive表中。**

1）语法

hive> 

LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)];

关键字说明：

**（1）local：**表示==从本地加载数据到Hive表；否则从HDFS加载数据到Hive表。==

**（2）overwrite：**表示==覆盖表中已有数据，否则表示追加。==

**（3）partition：**表示==上传到指定分区，若目标是分区表，需指定分区。==

例如：

`hive (default)> load data local inpath '/opt/module/datas/student.txt' into table student;`

## Insert

### 将==查询结果==插入表中

**1**）语法

**INSERT** (**INTO** | **OVERWRITE**) **TABLE** tablename [**PARTITION** (partcol1=val1, partcol2=val2 ...)] select_statement;	

**关键字说明：**

（1）INTO：将结果追加到目标表

（2）OVERWRITE：用结果覆盖原有数据

例如

```
hive (default)> insert overwrite table student3 
select 
    id, 
    name 
from student;

```

### 将==给定Values==插入表中

**1**）语法

**INSERT** (**INTO | OVERWRITE**) **TABLE** tablename [**PARTITION** (partcol1[=val1], partcol2[=val2] ...)] **VALUES** values_row [, values_row ...]

**2**）案例

`hive (default)> insert into table student1 values(1,'wangwu'),(2,'zhaoliu');`

###  将查询结果写入目标路径

**1**）语法

**INSERT** **OVERWRITE** [**LOCAL**] **DIRECTORY** directory

 [**ROW FORMAT** row_format] [**STORED AS** file_format] select_statement;

**2**）案例

```sql
insert overwrite local directory '/opt/module/datas/student' ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'

select id,name from student;
```

## Export&Import

==Export导出语句可将表的数据和元数据信息一**并到处的HDFS路径**==，Import可将Export导出的内容导入Hive，表的数据和元数据信息都会恢复。==Export和Import可用于两个Hive实例之间的数据迁移==。

**1**）语法**

--导出

**EXPORT TABLE** tablename **TO** 'export_target_path'

--导入

**IMPORT** [**EXTERNAL**] **TABLE** new_or_original_tablename **FROM** 'source_path' [**LOCATION** 'import_target_path']

**2**）案例

```sql
--导出
hive>
export table default.student to '/user/hive/warehouse/export/student';


--导入
hive>
import table student2 from '/user/hive/warehouse/export/student';
```

# 查询

## 排序

### 分区（Distribute By）

Distribute By：在有些情况下，我们需要控制某个特定行应该到哪个Reducer，通常是为了进行后续的聚集操作。**distribute by**子句可以做这件事。**distribute by**类似MapReduce中partition（自定义分区），进行分区，结合sort by使用。 

对于distribute by进行测试，一定要分配多reduce进行处理，否则无法看到distribute by的效果。

**1**）案例实操：

（1）先按照部门编号分区，再按照员工编号薪资排序

```sql
hive (default)> set mapreduce.job.reduces=3;
hive (default)> 
insert overwrite local directory 
'/opt/module/hive/datas/distribute-result'

select 
  * 
from emp 
distribute by deptno 
sort by sal desc;
```

注意：

-  distribute by的分区规则是==根据分区字段的hash码与reduce的个数进行相除后==，余数相同的分到一个区。
-  Hive要求**distribute by**语句要写在sort by语句之前。

演示完以后mapreduce.job.reduces的值要设置回-1，否则下面分区or分桶表load跑MapReduce的时候会报错

### 分区排序（Cluster By）

当distribute by和sort by字段相同时，可以使用cluster by方式。

cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是升序排序，不能指定排序规则为asc或者desc。

（1）以下两种写法等价

```
hive (default)> 
select 
  * 
from emp 
cluster by deptno;

 
hive (default)>
select 
* 
from emp 
distribute by deptno 
sort by deptno;
```

注意：按照部门编号分区，不一定就是固定死的数值，可以是20号和30号部门分到一个分区里面去。

## 分区表

Hive中的分区就是把一张大表的数据按照业务需要分散的存储到多个目录，每个目录就称为该表的一个分区。在查询时通过where子句中的表达式选择查询所需要的分区，这样的查询效率会提高很多。

### 分区表基本语法

**1.** **创建分区表**

```sql
hive (default)> 
create table dept_partition
(
  deptno int,  --部门编号
  dname string, --部门名称
  loc  string --部门位置
)
  partitioned by (day string)
  row format delimited fields terminated by '\t';
```



**2.** **分区表基本操作**

**1**）查看所有分区信息

`hive> show partitions dept_partition;`

**2）增加分区**

**（1）创建单个分区**

```
hive (default)> 
alter table dept_partition 
add partition(day='20220403');
```

**（**2**）同时创建多个分区（分区之间不能有逗号）**

```sql
hive (default)> 
alter table dept_partition 
add partition(day='20220404') partition(day='20220405');
```

**3）删除分区**

**（**1）删除单个分区**

```sql
hive (default)> 
alter table dept_partition 
drop partition (day='20220403');
```



**（**2**）同时删除多个分区（分区之间必须有逗号）**

```sql
hive (default)> 

alter table dept_partition 

drop partition (day='20220404'), partition(day='20220405');
```



**4）修复分区**

Hive将分区表的所有分区信息都保存在了元数据中，==只有元数据与HDFS上的**分区路径一致时**==，分区表才能正常读写数据。若==用户手动创建/删除分区路径，Hive都是感知不到的==，这样就会导致Hive的元数据和HDFS的分区路径不一致。再比如，若分区表为外部表，用户执行drop partition命令后，分区元数据会被删除，而HDFS的分区路径不会被删除，同样会导致Hive的元数据和HDFS的分区路径不一致。

若出现元数据和HDFS路径不一致的情况，可通过如下几种手段进行修复。

（**1**）**add partition**

若手动创建HDFS的分区路径，Hive无法识别，可通过==add partition==命令增加分区元数据信息，从而使元数据和分区路径保持一致。

（**2**）**drop partition**

若手动删除HDFS的分区路径，Hive无法识别，可通过drop partition命令删除分区元数据信息，从而使元数据和分区路径保持一致。

（**3**）**msck**

若==分区元数据和HDFS的分区路径不一致==，还可使用msck命令进行修复，以下是该命令的用法说明。

hive (default)> 

**msck repair** **table** table_name [**add**/**drop**/**sync** **partitions**];

说明：

msck repair table table_name add partitions：该命令会==增加HDFS路径存在但元数据缺失的分区信息==。

msck repair table table_name drop partitions：该命令会==删除HDFS路径已经删除但元数据仍然存在的分区信息==。

msck repair table table_name sync partitions：该命令会==同步HDFS路径和元数据分区信息，相当于同时执行上述的两个命令。==

msck repair table table_name：等价于msck repair table table_name **add** partitions命令。

### 二级分区表

思考：如果一天内的日志数据量也很大，如何再将数据拆分?答案是二级分区表，例如可以在按天分区的基础上，再对每天的数据按小时进行分区。

**1**）**二级分区表建表语句**

```sql
hive (default)>
create table dept_partition2(
  deptno int,  -- 部门编号
  dname string, -- 部门名称
  loc string   -- 部门位置
)
partitioned by (day string, hour string)
row format delimited fields terminated by '\t';
```



### 动态分区

动态分区是指向分区表insert数据时，==被写往的分区不由用户指定，而是由每行数据的最后一个字段的值来动态的决定==。使用动态分区，可只用一个insert语句将数据写入多个分区。

1**）动态分区相关参数**

（1）动态分区功能总开关（默认true，开启）

`set hive.exec.dynamic.partition=true`

**（2）严格模式和非严格模式**

动态分区的模式，默认strict（严格模式），要求必须指定至少一个分区为静态分区，nonstrict（非严格模式）允许所有的分区字段都使用动态分区。

`set hive.exec.dynamic.partition.mode=nonstrict`

（3）一条insert语句可同时创建的最大的分区个数，默认为1000。

`set hive.exec.max.dynamic.partitions=1000`

（4）单个Mapper或者Reducer可同时创建的最大的分区个数，默认为100。

`set hive.exec.max.dynamic.partitions.pernode=100`

（5）一条insert语句可以创建的最大的文件个数，默认100000。

`hive.exec.max.created.files=100000`

（6）当查询结果为空时且进行动态分区时，是否抛出异常，默认false。

`hive.error.on.empty.partition=false`

## 分桶表

分区提供一个隔离数据和优化查询的便利方式。不过，==并非所有的数据集都可形成合理的分区==。对于一张表或者分区，==Hive 可以进一步组织成桶==，也就是更为细粒度的数据范围划分，==分区针对的是**数据的存储路径**，分桶**针对的是数据文件**==。

分桶表的基本原理是，首先为每行数据计算一个指定字段的数据的hash值，然后模以一个指定的分桶数，最后将取模运算结果相同的行，写入同一个文件中，这个文件就称为一个分桶（bucket）。

### 分桶表基本语法

1**）建表语句**

```sql
hive (default)> 
create table stu_buck(
  id int, 
  name string
)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';
```



###  分桶排序表

1**）建表语句**

```sql
hive (default)> 
create table stu_buck_sort(
  id int, 
  name string
)
clustered by(id) sorted by(id)
into 4 buckets
row format delimited fields terminated by '\t';
```





# 其他

## hiveserver2服务

Hive的hiveserver2服务的作用==是提供jdbc/odbc接口==，为用户提供远程访问Hive数据的功能，例如用户期望在个人电脑中访问远程服务中的Hive数据，==就需要用到Hiveserver2==。

1）用户说明

在远程访问Hive数据时，客户端并未直接访问Hadoop集群，而是由Hivesever2代理访问。由于Hadoop集群中的数据具备访问权限控制，所以此时需考虑一个问题：那就是访问Hadoop集群的用户身份是谁？是Hiveserver2的启动用户？还是客户端的登录用户？

答案是都有可能，具体是谁，由Hiveserver2的hive.server2.enable.doAs参数决定，该参数的含义是是否启用Hiveserver2用户模拟的功能。==若启用，则Hiveserver2会**模拟成客户端的登录用户去访问Hadoop集群的数据**==，不启用，则Hivesever2会直接使用==**启动用户访问Hadoop集群数据**==。



==生产环境，推荐开启用户模拟功能，因为开启后才能保证各用户之间的权限隔离。==

## metastore服务

Hive的metastore服务的作用是==为`Hive CLI`或者`Hiveserver2`提供元数据访问接口==。

**1）metastore运行模式**

metastore有两种运行模式，分别为嵌入式模式和独立服务模式。下面分别对两种模式进行说明：

生产环境中，不推荐使用嵌入式模式。因为其存在以下两个问题：

（1）嵌入式模式下，每个Hive CLI都需要直接连接元数据库，当Hive CLI较多时，数据库压力会比较大。

（2）每个客户端都需要用户元数据库的读写权限，元数据库的安全得不到很好的保证。

## 编写Hive服务启动脚本（了解）

**1****）前台启动的方式导致需要打开多个**Xshell**窗口，可以使用如下方式后台方式启动**

- `nohup`：放在命令开头，表示不挂起，也就是关闭终端进程也继续保持运行状态
- `/dev/null`：是Linux文件系统中的一个文件，被称为黑洞，所有写入该文件的内容都会被自动丢弃
- ` 2>&1`：表示将错误重定向到标准输出上
- ` &`：放在命令结尾，==表示后台运行==

一般会组合使用：nohup [xxx命令操作]> file 2>&1 &，表示将xxx命令运行的结果输出到file中，并保持命令启动的进程在后台运行。

如上命令不要求掌握。

```
nohup hive --service metastore 2>&1 &

nohup hive --service hiveserver2 2>&1 &
```



## Hive常用交互命令

```shell
[atguigu@hadoop102 hive]$ bin/hive -help
usage: hive
 -d,--define <key=value>          Variable subsitution to apply to hive
                                  commands. e.g. -d A=B or --define A=B
    --database <databasename>     Specify the database to use
 -e <quoted-query-string>         SQL from command line
 -f <filename>                      SQL from files
 -H,--help                        Print help information
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable subsitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the console)

```

**2****）`“-e”不进入hive的交互窗口执行hql语句`

例子：`[atguigu@hadoop102 hive]$ bin/hive -e "select id from student;"`

**3****）`“-f”执行脚本中的hql语句**`

```shell
[atguigu@hadoop102 hive]$ mkdir datas
[atguigu@hadoop102 datas]$ vim hivef.sql
```

## Hive参数配置方式

1）**查看当前所有的配置信息**

`hive>set;`

2）**参数的配置三种方式**

（1）**配置文件方式**

Ø 默认配置文件：`hive-default.xml`

Ø 用户自定义配置文件：`hive-site.xml`

注意：用户自定义配置会覆盖默认配置。另外，Hive也会读入Hadoop的配置，因为Hive是作为Hadoop的客户端启动的，==Hive的配置会覆盖Hadoop的配置==。配置文件的设定对本机启动的所有Hive进程都有效。

（2）**命令行参数方式**

①启动Hive时，可以在命令行添加`-hiveconf param=value`来设定参数。例如：

```
[atguigu@hadoop103 hive]$ bin/hive -hiveconf 

mapreduce.job.reduces=10;
```

注意：仅对本次Hive启动有效。

②查看参数设置

`hive (default)> set mapreduce.job.reduces;`

（3）**参数声明方式**

可以在HQL中使用SET关键字设定参数，例如：

`hive(default)> set mapreduce.job.reduces=10;`

注意：仅对本次Hive启动有效。

查看参数设置：

`hive(default)> set mapreduce.job.reduces;`

上述三种设定方式的优先级依次递增。即配置文件 < 命令行参数 < 参数声明。注意某些系统级的参数，例如log4j相关的设定，必须用前两种方式设定，因为那些参数的读取在会话建立以前已经完成了。

# 

