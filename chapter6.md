

## Hive

### 数据仓库

#### 定义

数据仓库，英文名称为==Data Warehouse==，可简写为DW或DWH。数据仓库顾名思义，是一个很大的数据存储集合，出于企业的分析性报告和决策支持目的而创建，对多样的业务数据进行筛选与整合。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/dw1.png" style="zoom:55%;" />

数据仓库的输入方是各种各样的数据源，最终的输出用于企业的数据分析、数据挖掘、数据报表等方向。

HDFS文件系统就是一个数仓，一般情况下需要写MapReduce程序对数仓中的程序进行分析。但是有了hive以后就可以将HDFS中的文件转化为数据表，通过hiveSQL进行数据分析。

#### 数仓的主要特点

1. 主题性：数仓中的数据不是泛泛而存，而是面向某一个主题，某一个方向
2. 集成性：数据来源于多方面，集成后的格式不统一需要ETL处理
3. 稳定性：在下一个采集周期未到来时，数据很少发生变化，没有增删改只有查询
4. 时变性：在下一个采集周期到来时，数据会发生变化

Hive是将数据(所有的数据文件全部存放在hadoop中的hdfs分布式文件系统之上)映射成数据库和一张张的表，库和表的==元数据==信息一般存在关系型数据库上（比如MySQL）。

#### 数据仓库的最大特点

既不生产数据也不消耗数据，数据来源于各个数据源。

#### 数仓与数据库的区别

- 数据库中的数据是为了业务开展而存在的(生存问题)，主要用于捕获信息
- 数仓中的数据是为了进一步分析而存在的(优化问题)，主要用于分析信息
- 数仓中的数据可以来自于数据库

#### 数据仓库的分层

数据仓库可以分为三层：**源数据、数据仓库、数据应用**

数仓的分层其实就是将不同阶段的数据存放在不同的数据库中

- 数据应用层(app)-DA层

- 数据仓库层(dw)-DW层

- 源数据(odfs)-ODS层


<img src="/Users/mungeryang/Desktop/Bigdata/pic/dw.png" style="zoom:33%;" />



### 数据仓库ETL

#### ETL的概念与设计

定义：ETL是将数据从来源端经过抽取(Extract)、转换(Transform)、加载(Load)至目的端的过程。

一般用于数据仓库领域，将数据进行采集并做转换处理，最终将转换好的数据加载至数据仓库中去。

ETL，是英文Extract-Transform-Load的缩写，用来描述将数据从来源端经过抽取（extract）、转换（transform）、加载（load）至目的端的过程。ETL是将业务系统的数据经过抽取、清洗、转换之后加载到数据仓库的过程，目的是将企业中分散、零乱、标准不统一的数据整合到一起。

ETL是数据仓库的流水线，也可以认为是数据仓库的血液，它维系着数据仓库中数据的新陈代谢，而数据仓库日常的管理和维护工作的大部分精力就是保持ETL的正常和稳定。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/ETL.png" style="zoom:50%;" />

ETL工程师的工作就是在数据装载到数据仓库之前对数据进行前期处理的过程，包括数据抽取、数据转换、数据装载。

#### ETL功能

– 抽取/采集：Extract

将不同数据来源的数据进行抽取

数据来源：RDNMS、文件系统、数据流端口

抽取方式：

​	全量：每次都抽取所有数据，覆盖之前的数据

​	增量：每次只抽取最新的数据，追加到之前的数据中



– 转换/处理

对抽取的数据进行转换处理，数据清洗

过滤：将非法数据进行剔除过滤  字段缺失、非法

转换：将数据格式进行转换或者数据类型进行转换

补全：利用已有的数据，将需要的但当前缺少的数据进行补全



– 加载：Load

将转换后的数据加载到目的地中

加载方式：全量、增量



### 数据集市

在数据仓库基础之上，基于主题对数据进行抽取处理分析工作，形成最终的结果。数据仓库包含数据集市，数据集市是小型的数据仓库。

<img src="/Users/mungeryang/Desktop/Bigdata/scrapy/数据仓库分层.png" style="zoom:43%;" />



### Hive安装与配置

Hive是一个构建在Hadoop上的数据仓库框架。Hive是基于Hadoop的一个数据仓库工具，可以==将结构化的数据文件映射为一张数据库表，并提供类SQL查询功能。==

Hive其本质是将`SQL`==转换为==`MapReduce`的任务进行运算，底层由`HDFS`来提供数据的==存储==，说白了Hive可以理解为一个==将SQL转换为MapReduce的任务的工具==，甚至更进一步可以说hive就是一个MapReduce的客户端。

Hive具有SQL数据库的外表，但应用场景完全不同，Hive只适合用来做批量数据统计分析.

#### 安装方式

hive的安装一共有三种方式**:内嵌模式、本地模式、远程模式**

**内嵌模式**使用的是内嵌的Derby数据库来存储元数据，也不需要额外起Metastore服务。数据库和Metastore服务都嵌入在主Hive Server进程中。

解压hive安装包 bin/hive 启动即可使用

缺点：不同路径启动hive，每一个hive拥有一套自己的元数据，无法共享。

**本地模式**采用外部数据库来存储元数据，目前支持的数据库有：MySQL、Postgres、Oracle、MS SQL Server.

hive根据hive.metastore.uris 参数值来判断，如果为空，则为本地模式。

缺点是：每启动一次hive服务，都内置启动了一个metastore。

**远程模式**下，需要单独起metastore服务，然后每个客户端都在配置文件里配置连接到该metastore服务。远程模式的metastore服务和hive运行在不同的进程里。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/beenline.png" style="zoom:35%;" />

**元数据:**Metastore:本质上只是用来存储hive中有哪些数据库，哪些表，表的字段，,表所属数据库(默认是default) ，分区，表的数据所在目录等，元数据默认存储在自带的derby数据库中,推荐使用MySQL存储Metastore。

远程模式下，需要配置hive.metastore.uris 参数来指定metastore服务运行的机器ip和端口，并且需要单独手动启动metastore服务。

hiveserver2是Hive启动了一个server，客户端可以使用JDBC协议，通过IP+ Port的方式对其进行访问，达到并发访问的目的。

hiveserver2的作用就是提供jdbc/odb接口，为用户提供远程访问hive数据的功能；例如用户期望在自己的个人电脑上远程访问hive数据就必须用到hiveserver2。

#### 安装配置

~~~shell
# 上传压缩包到/export/software目录里，并解压安装包
cd /export/software/
tar -zxvf apache-hive-3.1.2-bin.tar.gz -C /export/server
cd /export/server
mv apache-hive-3.1.2-bin hive

#解决hadoop、hive之间guava版本差异
cd /export/server/hive
rm -rf lib/guava-19.0.jar
cp /export/server/hadoop-3.1.4/share/hadoop/common/lib/guava-27.0-jre.jar ./lib/

#添加mysql jdbc驱动到hive安装包lib/文件下
cd lib/
mysql-connector-java-5.1.41-bin.jar

#修改hive环境变量文件 添加Hadoop_HOME
cd /export/server/hive/conf/
mv hive-env.sh.template hive-env.sh

vim hive-env.sh
#查找HADOOP_HOME /HADOOP_HOME
HADOOP_HOME=/export/server/hadoop-3.1.4
export HIVE_CONF_DIR=/export/server/hive/conf
export HIVE_AUX_JARS_PATH=/export/server/hive/lib
#新增hive-site.xml 配置mysql等相关信息
vim hive-site.xml

#初始化metadata
cd /export/server/hive
bin/schematool -initSchema -dbType mysql -verbos
#初始化成功会在mysql中创建74张表
+-------------------------------+
| Tables_in_hive                |
+-------------------------------+
| AUX_TABLE                     |
| BUCKETING_COLS                |
| CDS                           |
| COLUMNS_V2                    |
| COMPACTION_QUEUE              |
| COMPLETED_COMPACTIONS         |
| COMPLETED_TXN_COMPONENTS      |
| CTLGS                         |
| DATABASE_PARAMS               |
| DBS                           |
| DB_PRIVS                      |
| DELEGATION_TOKENS             |
| FUNCS                         |
| FUNC_RU                       |
| GLOBAL_PRIVS                  |
| HIVE_LOCKS                    |
| IDXS                          |
| INDEX_PARAMS                  |
| I_SCHEMA                      |
| KEY_CONSTRAINTS               |
| MASTER_KEYS                   |
| MATERIALIZATION_REBUILD_LOCKS |
| METASTORE_DB_PROPERTIES       |
| MIN_HISTORY_LEVEL             |
| MV_CREATION_METADATA          |
| MV_TABLES_USED                |
| NEXT_COMPACTION_QUEUE_ID      |
| NEXT_LOCK_ID                  |
| NEXT_TXN_ID                   |
| NEXT_WRITE_ID                 |
| NOTIFICATION_LOG              |
| NOTIFICATION_SEQUENCE         |
| NUCLEUS_TABLES                |
| PARTITIONS                    |
| PARTITION_EVENTS              |
| PARTITION_KEYS                |
| PARTITION_KEY_VALS            |
| PARTITION_PARAMS              |
| PART_COL_PRIVS                |
| PART_COL_STATS                |
| PART_PRIVS                    |
| REPL_TXN_MAP                  |
| ROLES                         |
| ROLE_MAP                      |
| RUNTIME_STATS                 |
| SCHEMA_VERSION                |
| SDS                           |
| SD_PARAMS                     |
| SEQUENCE_TABLE                |
| SERDES                        |
| SERDE_PARAMS                  |
| SKEWED_COL_NAMES              |
| SKEWED_COL_VALUE_LOC_MAP      |
| SKEWED_STRING_LIST            |
| SKEWED_STRING_LIST_VALUES     |
| SKEWED_VALUES                 |
| SORT_COLS                     |
| TABLE_PARAMS                  |
| TAB_COL_STATS                 |
| TBLS                          |
| TBL_COL_PRIVS                 |
| TBL_PRIVS                     |
| TXNS                          |
| TXN_COMPONENTS                |
| TXN_TO_WRITE_ID               |
| TYPES                         |
| TYPE_FIELDS                   |
| VERSION                       |
| WM_MAPPING                    |
| WM_POOL                       |
| WM_POOL_TO_TRIGGER            |
| WM_RESOURCEPLAN               |
| WM_TRIGGER                    |
| WRITE_SET                     |
+-------------------------------+


#添加环境变量
vim /etc/profile

export HIVE_HOME=/export/server/hive
export PATH=:$HIVE_HOME/bin:$PATH


#让环境变量生效
source /etc/profile

#-----------------Metastore 和 Hiveserver2启动----
nohup /export/server/hive/bin/hive --service metastore &
nohup /export/server/hive/bin/hive --service hiveserver2 &


#验证是否安装成功!
在Linux中输入hive命令，直接回车,出现一个终端，在该终端中可以输入sql命令:
show databases;
~~~



#### Hive拒绝连接的原因

1. hive-site.xml配置

   ~~~xml
   <?xml version="1.0" encoding="UTF-8" standalone="no"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
   <property>
         <name>javax.jdo.option.ConnectionUserName</name>
         <value>root</value>
     </property>
     <property>
         <name>javax.jdo.option.ConnectionPassword</name>
         <value>123456</value>
     </property>
     <property>
         <name>javax.jdo.option.ConnectionURL</name>
         <value>jdbc:mysql://node3:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
     </property>
     <property>
         <name>javax.jdo.option.ConnectionDriverName</name>
         <value>com.mysql.jdbc.Driver</value>
     </property>
     <property>
         <name>hive.metastore.schema.verification</name>
         <value>false</value>
     </property>
     <property>
       <name>datanucleus.schema.autoCreateAll</name>
       <value>true</value>
    </property>
    <property>
   	<name>hive.server2.thrift.bind.host</name>
   	<value>node3</value>
      </property>
   </configuration>
   ~~~

   

2. hadoop中core-site.xml配置

   ~~~xml
   <property>
       <name>hadoop.proxyuser.root.hosts</name>
       <value>*</value>
   </property>
   <property>
       <name>hadoop.proxyuser.root.groups</name>
       <value>*</value>
   </property>
   ~~~

   

3. mysql配置与connector是否正确

4. hadoop安全模式是否关闭

   ~~~shell
   hdfs dfsadmin -safemode leave
   ~~~

### Hive数据库与表操作

#### 数据库创建

hive-site.xml：hive.metastore.warehouse.dir

```hive
在hive中每创建一个数据库，则会在HDFS的/user/hive/warehouse下创建对应文件夹
create databases if not exists mydatabase;
```

```xml
#hive的表存放位置模式是由hive-site.xml当中的一个属性指定的
<name>hive.metastore.warehouse.dir</name>
<value>/user/hive/warehouse</value>

```

#### 数据库创建并指定存储位置

```hive
create database myhive location '/myhive';
-- 查看数据库的基本信息
desc database myhive;
-- 删除数据库
drop database myhive;
-- 强制删除数据库和下面的表
drop database myhive cascade;
```

EXTERNAL关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION），Hive 创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。

#### 数据表类型

hive数据库创建表时有内部表和外部表之分。

内部表没有被`external`修饰，是私有表。存储位置有最初的配置文件hive-site.xml决定，删除内部表就是删除了元数据和存储数据，内部表不适合和其他工具共享数据。

外部表必须指定`external`修饰，外部表对应的文件存储在location下指定的HDFS目录下，删除外部表时，数据依然存放在HDFS文件系统中。向HDFS文件系统中添加新的文件时，该表也会读取该文件。删除hive外部表的时候，数据仍然存放在hdfs当中，不会删掉。

#### Hive建表的字段类型

| **分类** | **类型**                                       | **描述**                                    | **字面量示例**                              |
| -------- | ---------------------------------------------- | ------------------------------------------- | ------------------------------------------- |
| 原始类型 | BOOLEAN                                        | true/false                                  | TRUE                                        |
|          | TINYINT                                        | 1字节的有符号整数 -128~127                  | 1Y                                          |
|          | SMALLINT                                       | 2个字节的有符号整数，-32768~32767           | 1S                                          |
|          | INT                                            | 4个字节的带符号整数(-2147483648~2147483647) | 1                                           |
|          | BIGINT                                         | 8字节带符号整数                             | 1L                                          |
|          | FLOAT                                          | 4字节单精度浮点数1.0                        |                                             |
|          | DOUBLE                                         | 8字节双精度浮点数                           | 1.0                                         |
|          | DEICIMAL                                       | 任意精度的带符号小数                        | 1.0                                         |
|          | STRING                                         | 字符串，变长                                | “a”,’b’                                     |
|          | VARCHAR                                        | 变长字符串                                  | “a”,’b’                                     |
|          | CHAR                                           | 固定长度字符串                              | “a”,’b’                                     |
|          | BINARY                                         | 字节数组                                    | 无法表示                                    |
|          | TIMESTAMP                                      | 时间戳，毫秒值精度                          | 122327493795                                |
|          | DATE                                           | 日期                                        | ‘2016-03-29’                                |
|          | Time                                           | 时分秒                                      | ‘12:35:46’                                  |
|          | DateTime                                       | 年月日 时分秒                               |                                             |
| 复杂类型 | ARRAY                                          | 有序的的同类型的集合                        | ["beijing","shanghai","tianjin","hangzhou"] |
| MAP      | key-value,key必须为原始类型，value可以任意类型 | {"数学":80,"语文":89,"英语":95}             |                                             |
| STRUCT   | 字段集合,类型可以不同                          | struct(‘1’,1,1.0)                           |                                             |

#### 数据加载load

~~~hive
-- 从本地Linux系统加载数据文件
load data local inpath '/export/data/hivedatas/score.txt' into table score;
-- 从HDFS文件系统加载数据文件
load data inpath '/usr/data/warehouse/score.txt' into table score;
~~~



#### 分区与分桶

> Hive中的分桶表就是MapReduce中的分区，将数据在同一个文件夹下进行分文件存储
>
> Hive中的分区表和MapReduce中的分区没有关系，将数据分到==不同文件夹==下存储

分区表实际就是对应hdfs文件系统上的的独立的文件夹，该文件夹下是该分区所有数据文件。

分区可以理解为分类，通过分类把不同类型的数据放到不同的目录下。

分区表的意义在于优化查询。查询时尽量利用分区字段。如果不使用分区字段，就会全部扫描。

在hive中，==分区就是分文件夹==。

~~~hive
-- 创建分区表
create table score(sid string,cid string, sscore int) partitioned by (month string) row format delimited fields terminated by '\t';
-- 创建一个表带多个分区
create table score2 (sid string,cid string, sscore int) partitioned by (year string,month string,day string) row format delimited fields terminated by '\t';
-- 数据加载到分区
load data local inpath '/export/data/hivedatas/score.txt' into table score partition(month='202404');
-- 数据加载到多个分区
load data local inpath '/export/data/hivedatas/score.txt' into table score2 partition(year='2020',month='06',day='01');

-- 添加分区之后就可以在hdfs文件系统当中看到表下面多了一个文件夹
alter table score add partition(month='202009') partition(month = '202010');
~~~



分桶就是将数据划分到不同的文件，其实就是MapReduce的分区.

桶表的数据加载，由于桶表的数据加载通过`hdfs dfs -put`文件或者通过`load data`均不好使，只能通过`insert overwrite`。

先创建普通表，再将普通表的数据插入到分桶表中。

~~~hive
-- 开启hive的桶表功能 如果报错说明已经开启此功能
set hive.enforce.bucketing=true;
-- 设置reduce个数
set mapreduce.job.reduces=3;   #该参数在Hive2.x版本之后不起作用
-- 创建分桶表
create table course (cid string,c_name string,tid string) clustered by(cid) into 3 buckets row format delimited fields terminated by '\t';
-- 创建普通表
create table course_common (cid string,c_name string,tid string) row format delimited fields terminated by '\t';
-- 普通表加载数据
load data local inpath '/export/data/hivedatas/course.txt' into table course_common;
-- 通过insert  overwrite给桶表中加载数据
insert overwrite table course select * from course_common cluster by(cid);
~~~



#### hive的函数

~~~hive
/*
Hive的函数分为三类： 聚合函数、内置函数，表生成函数
*/
select round(3.1415); --3
select round(3.1415926, 4); --3.1415
select `floor`(3.1415926); --下取整3
select ceil(3.1415926); --上取整4
select rand(100); -- 随机数，seed=100
select pow(2, 4); -- 幂运算16
select abs(-3); --绝对值3
select length('sdfsaf'); --字符串长度6
select reverse("hello"); -- 反转olleh
select concat("ygm", "love"); --字符串连接concat(string A, string B…)
select concat_ws(",", "abc", "sd", "sdfw"); --6.1.2.4.	字符串连接函数-带分隔符:abc,sd,sdfw
select substr("abcde", 3); --cde
select substring('sfder', 3); --der
select substr('abcde', -1); --e
select substr('abcde', 3, 2); --cd
select substring('abcde', 3, 2); --cd
select substring('abcde', -2, 2); --de:从-2开始向后截取2个字符串
select lower('ABCDE'); --大写转换为小写
select ucase('ABCDE'); --大写转换为小写
select upper('abcde'); -- 小写转化为大写
select lcase('abcde'); -- 小写转化为大写
select trim(' sb '); --去空格
select ltrim(' db'); -- 左端去空格
select rtrim(' sb '); -- 右端去空格
select regexp_replace('foobar', 'oo|ar', ''); --字符串匹配替换，将oo和ar用空替换原来的字符串
select split('abtcdtef', 't');
--分割字符串["ab","cd","ef"]
/*
解析url：
parse_url(string urlString, string partToExtract [, stringkeyToExtract])
partToExtract的有效值为：HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, and USERINFO.
*/
select parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1', 'HOST');
-- facebook.com
select parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1', 'PATH');
-- /path1/p.php
select parse_url('http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1', 'QUERY', 'k1');
-- v1

/*
日期函数
*/
-- 日期转UNIX时间戳函数unix_timestamp
select unix_timestamp('2024-04-02 13:14:52');
-- 1323234063
select unix_timestamp('20240402 13:14:52', 'yyyyMMdd HH:mm:ss');
-- 1323234063
select date_format('2020-1-1 1:1:1', 'yyyy-MM-dd HH:mm:ss');
-- 时间格式转换 2020-01-01 01:01:01

-- 日期时间转日期函数:to_date
select to_date('2020-01-01 01:01:01');
-- 2020-01-01

-- 日期转年、月、天、周
select year('2020-01-01 01:01:01'); --2020
select month('2020-01-01 01:01:01'); --01
select day('2020-01-28 01:01:01'); --28
select weekofyear('2020-01-28 01:01:01'); --4

-- 日期比较、增加、减少
select datediff('2020-01-01', '2021-01-01'); --返回结束日期减去开始日期的天数 365
select date_add('2020-01-01', 10); --2020-01-10
select date_sub('2020-01-10', 10);
--2020-01-01

/*
条件函数：if and case
if(boolean testCondition, T valueTrue, T valueFalseOrNull)
当条件testCondition为TRUE时，返回valueTrue；否则返回valueFalseOrNull
case a when b then c end
如果a等于b，那么返回c；如果a等于d，那么返回e；否则返回f

*/
select if(1 = 2, 100, 200); --200
select if(1 = 1, 100, 200); --100

select case 100 when 50 then 'torm' when 100 then 'mary' else 'tim' end; --mary
select case 200 when 50 then 'tom' when 100 then 'mary' else 'tim' end; --tim

select case when 1 = 3 then 'tom' when 2 = 2 then 'mary' else 'tim' end; -- mary
select case when 1 = 1 then 'tom' when 2 = 2 then 'mary' else 'tim' end; --tom
select score.sid, case when score.sscore >= 60 then '及格' when score.sscore < 60 then '不及格' else '其他' end
from score;

-- 数据类型强转
select cast(12.12 as int);
select cast('20200202' as date);

/*
行转列:多行数据转换为一个列的字段
行转列用到的函数：
    concat(STR1,STR2) --字段或字符串拼接
    concat_ws(sep,str1,str2) --以分隔符拼接每个字符串
    collect_set(col) --将某字段值进行去重汇总，产生array类型字段
测试数据：deptno_ename
20	 SMITH
30 	ALLEN
30 	WARD
20	 JONES
30 	MARTIN
30 	BLAKE
10 	CLARK

*/

create table emp
(
    deptno int,
    ename  string
)
    row format delimited fields terminated by '\t';

load data local inpath "/export/data/hivedatas/emp.txt" into table emp;

select deptno, concat_ws("|", collect_set(ename)) as ems
from emp
group by deptno;

/*
表生成函数-explode
explode(col)：将hive一列中复杂的array或者map结构拆分成多行。
explode(ARRAY)  数组的每个元素生成一行
explode(MAP)    map中每个key-value对，生成一行，key为一列，value为一列
测试数据：
10	 CLARK|KING|MILLER
20	 SMITH|JONES|SCOTT|ADAMS|FORD
30 	ALLEN|WARD|MARTIN|BLAKE|TURNER|JAMES

*/

create table emp2
(
    deotno int,
    names  array<string>
)
    row format delimited fields terminated by '\t'
        collection items terminated by '|';

load data local inpath '/export/data/hivedata/emp2.txt' into table emp2;

select *
from emp2;

select explode(names) as name
from emp2;

/*
开窗函数
窗口函数(一)--ROW_NUMBER,RANK,DENSE_RANK
Hive分析窗口函数(2)--SUM,AVG,MIN,MAX
数据用例：
cookie1,2018-04-10,1
cookie1,2018-04-11,5
cookie1,2018-04-12,7
cookie1,2018-04-13,3
cookie1,2018-04-14,2
cookie1,2018-04-15,4
cookie1,2018-04-16,4
cookie2,2018-04-10,2
···
cookie2,2018-04-16,7
*/

-- row_number:从1开始，按照顺序，生成分组内记录的序列
SELECT cookieid,
       createtime,
       pv,
       ROW_NUMBER() OVER (PARTITION BY cookieid ORDER BY pv desc) AS rn
FROM itcast_t1;

-- RANK() 生成数据项在分组中的排名，排名相等会在名次中留下空位
-- DENSE_RANK() 生成数据项在分组中的排名，排名相等会在名次中不会留下空位
select cookieid,
       createtime,
       pv,
       rank() over (partition by cookieid order by pv desc)       as rn2,
       dense_rank() over (partition by cookieid order by pv desc) as rn3
from itcast_t1
where cookieid = 'cookie1';
~~~





#### hive压缩方式

~~~hive
/*
数据压缩
在实际工作当中，hive当中处理的数据，
一般都需要经过压缩，可以使用压缩来节省我们的MR处理的网络带宽
----------------------------------------------------
压缩格式	对应的编码/解码器                            |
DEFLATE	org.apache.hadoop.io.compress.DefaultCodec |
gzip	org.apache.hadoop.io.compress.GzipCodec    |
bzip2	org.apache.hadoop.io.compress.BZip2Codec   |
LZO	com.hadoop.compression.lzo.LzopCodec           |
LZ4	org.apache.hadoop.io.compress.Lz4Codec         |
Snappy	org.apache.hadoop.io.compress.SnappyCodec  |
----------------------------------------------------
*/


/*
数据存储
Hive支持的存储数的格式主要有：
TEXTFILE（行式存储） 、SEQUENCEFILE(行式存储)、ORC（列式存储）、PARQUET（列式存储）

存储文件的压缩比总结：
    ORC >  Parquet >  textFile
存储文件的查询速度总结：
	ORC > TextFile > Parquet

*/
create table log_orc_none
(
    track_time string,
    url        string,
    session_id string,
    ip         string,
    city_id    string
)
    row format delimited fields terminated by '\t'
    stored as orc tblproperties ("orc.compress" = "NONE");
    
-- 创建一个SNAPPY压缩的ORC存储方式
create table log_orc_snappy
(
    track_time string,
    url        string,
    session_id string,
    ip         string,
    city_id    string
)
    row format delimited fields terminated by '\t'
    stored as orc tblproperties ("orc.compress" = "SNAPPY");

--在实际的项目开发当中，hive表的数据存储格式一般选择：orc或parquet。压缩方式一般选择snappy
-- orc + snappy是最优选择

~~~



#### SQL优化

`key值过滤、hive去重和数据倾斜`

空key处理：

​    1.空key过滤

​    2.空key转换

空值发生的场景：有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够。

~~~hive
-- 不过滤
insert overwrite table jointable
select a.*
from nullidtable a
         join ori b on a.id = b.id;

--结果：No rows affected (152.135 seconds)

-- 过滤
insert overwrite table jointable
select a.*
from (select * from nullidtable where id is not null) a
         join ori b on a.id = b.id;
--结果：No rows affected (141.585 seconds)

-- 有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中
-- 此时我们可以表a中key为空的字段赋一个随机的值，使得数据随机均匀地分不到不同的reducer上。
set hive.exec.reducers.bytes.per.reducer=32123456;
set mapreduce.job.reduces=7;
insert overwrite table jointable
select a.*
from nullidtable a
         left join ori b on case when a.id is not null then 'hive' else a.id end = b.id;
--结果：这样的后果就是所有为null值的id全部都变成了相同的字符串，及其容易造成数据的倾斜
-- （所有的key相同，相同key的数据会到同一个reduce当中去）

insert overwrite table jointable
select a.*
from nullidtable a
         left join ori b on case when a.id is null then concat('hive', rand()) else a.id end = b.id;
~~~



 默认情况下map阶段同一个key数据分发给一个reduce，当一个key数据过大时就倾斜了。并不是所有的聚合都要在reduce端进行，很多聚合可以在map阶段完成

~~~hive
-- 是否在Map端进行聚合，默认为True
set hive.map.aggr = true;
-- 在Map端进行聚合操作的条目数目（阈值）
set hive.groupby.manager.checkinterval = 100000;
-- 有数据倾斜的时候进行负载均衡（默认是false）
set hive.groupby.skewindata = true;

-- 当选项设定为 true，生成的查询计划会有两个MR Job
-- 第一个MR Job中:map输出结果会随机分布到reduce中，每个reduce做聚合操作
    -- 这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的
-- 第二个MR Job中:根据预处理的数据结果按照Group By Key分布到Reduce中
~~~



`count distinct`

数据量小的时候无所谓，数据量大的情况下，由于COUNT DISTINCT操作需要用一个Reduce Task来完成。这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换。



`避免笛卡尔积`

尽量避免笛卡尔积，即避免join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积。

`并行执行`

-- Hive会将一个查询转化成一个或者多个阶段

-- MapReduce阶段、抽样阶段、合并阶段、limit阶段

-- 更多的阶段并行执行，job就会很快的完成



`严格模式`

-- Hive提供了一个严格模式，可以防止用户执行那些可能意想不到的不好的影响的查询。

~~~hive
set hive.mapred.mode = strict;  --开启严格模式
set hive.mapred.mode = nostrict; --开启非严格模式
~~~



1. 对于分区表，==在where语句中必须含有分区字段作为过滤条件来限制范围==，否则不允许执行。换句话说，就是用户不允许扫描所有分区。进行这个限制的原因是，通常分区表都拥有非常大的数据集，而且数据增加迅速。没有进行分区限制的查询可能会消耗令人不可接受的巨大资源来处理这个表。
2. 对于使用了==order by==语句的查询，==要求必须使用limit语句==。因为order by为了执行排序过程会将所有的结果数据分发到同一个Reducer中进行处理，强制要求用户增加这个LIMIT语句可以防止Reducer额外执行很长一段时间。
3. ==限制笛卡尔积的查询==。对关系型数据库非常了解的用户可能期望在执行JOIN查询的时候不使用ON语句而是使用where语句，这样关系数据库的执行优化器就可以高效地将WHERE语句转化成那个ON语句。不幸的是，Hive并不会执行这种优化，因此，如果表足够大，那么这个查询就会出现不可控的情况。



## 维度分析

针对某一个主题，可以从不同的角度进行数据分析，从而得出各种指标的过程。

### 什么是维度？

维度就是指的分析的角度，看待一个问题可以从多个角度，不同的角度就是维度。

维度也是事务的特征

比如一个订单数据，可以从时间、地域、商品、来源、用户……多角度分析

维度分类–定性维度：计算每天、每月各个维度，一般来说定性维度的字段都是放置在==group by==中，定量维度：指的是某一个具体的维度或者范围下信息–一般来说定量维度的字段都是放置在==where==中。

### 什么是指标？

指标就是度量值，衡量事物发展的标准

常见的度量值：count()	sum()	max()	min()	avg()

还有一些比例指标：转换率、流失率，同比等

指标的分类–绝对指标：反映出具体的大小和多少，如价格、销量、分数等；相对指标反应一定的程度，如及格率、购买率、涨幅等。



### 维度的分层和分级与上钻和下卷

本质上相当于将维度进行细分的过程。比如按照年、月、日、季度分层统计；按照省、市、县分层统计。在实际的统计中，统计的层级越多，意味着统计越细化，设置维度内容越多。

以某一个维度为基准，往细化统计的过程是下钻，往粗粒度方向统计称为上卷。从实际分析中，下钻和上卷意味着统计的维度增加了。



### 数仓建模

数仓建模指的是规定如何在hive中构建表，数仓建模主要提供了两种建模理论：三范式建模和维度建模。

三范式建模：主要存在于关系型数据库建模方案上，规定了每个表都应该有一个主键，数据要避免冗余发生。

SQL必须需要三表关联。

维度建模(dimensional modeling)：主要存在于分析型数据库建模方案上，主要一切以分析为目标，只要利于分析建模的都是可以的，允许存在一些冗余。维度建模两个核心：==事实表和维度表==

SQL只需要操作一个表即可完成。

### 数仓建模的三种模型

- 星型模型
- 雪花模型
- 星座模型

## 事实表与维度表

`事实表`一般是指的就是==分析主题==所对应的表，每一条数据用来描述一个具体的事实信息，这些表一般是一坨主键(外键)的聚集。事实表是主题所对应的表，一般也是计算指标所对应的表。 

事实表可以分为：

- 事务事实表：保存的是最原子的数据，也称为“原子事实表”和“交易事实表”。
- 周期快照事实表：周期快照事实表是具有规律性、可预见的时间间隔来记录事实，由事务表加工而成
- 累计快照事实表：完全覆盖一个事务或产品的生命周期时间跨度。

`维度表`指的是对事实表进行统计分析的时候，基于==某一个维度==(这个维度可能存在于其他表中)，这些表就是维度表。维度表不一定存在但是维度一定存在。