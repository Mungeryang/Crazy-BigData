## Hadoop技术栈

> 麦肯锡最早提出大数据概念
>
> 大数据的5V特点：
>
> ​	Volume-数据体量大
>
> ​	Variety-数据种类多
>
> ​	Value-数据价值低
>
> ​	Velocity-数据增长块
>
> ​	Veracity-数据质量高

### 大数据概述

**数据:**数据是可以获取和存储的信息,直观而言,表达客观事实的数值最容易被人们识别的数据。实际上，一切语言文字、图像、音频视频，只要感官可以察觉到，只要能被记录下来的并且可以查询到的，都是数据。==一切能被计算机存储的东西都是数据==

**数据的体量：**

~~~shell
#常见的数据存储单位：
1Byte = 8 bit	1KB(千) = 1024Byte	1MB(兆) = 1024KB	1GB(吉) = 1024MB
1TB(太) = 1024GB
#-------------------------------------------------------------
1PB(拍) = 1024TB	1EB(艾) = 1024PB	1ZB(泽) = 1024EB
1YB(尧) = 1024ZB	1BB(布) = 1024YB	1NB(诺) = 1024BB	1DB(刀) = 1024NB
~~~

大数据的体量一般在：T、P、E

大数据技术定义：

1. 传统常规的处理手段无法处理目前的数据，需要使用新的处理技术
2. 大数据主要解决==海量数据存储==和==海量数据计算==的问题	

~~~shell
#用于存储的框架
HDFS、HBase、Redis、Kafka
#用于计算的框架
MapReduce、Spark、Flink
#其他辅助框架
Zookeeper、Flume、Azkaban
~~~



### 分布式技术

为了解决算的慢的问题，采用计算并行化。一台计算机不够就多使用几台计算机，多线程/进程进化到计算并行化从而进化到分布式计算。

分布式强调的是任务的拆分，将原来的一个系统拆分到不同的主机上，每台主机负责一个部分模块。–多个主机做**不同**的事情

集群：将同一个任务分散到不同主机上进行–多个主机做**相同**的事情。

分布式+集群

#### 负载均衡

一台服务器相当于“包工头”，将任务分发给不同的服务器主机(“分活”)，专门负责拆分任务。

大数据集群=负载均衡集群+并行计算集群



### Hadoop框架

#### Hadoop介绍

<img src="/Users/mungeryang/Desktop/Bigdata/pic/hdfs.png" style="zoom:70%;" />

Hadoop是一个软件，它包含三个模块：

1. HDFS：Hadoop分布式文件系统
2. MapReduce：分布式计算系统
3. Yarn：分布式资源调度系统

Hadoop是一个生态圈，后期很多大数据框架都是基于Hadoop平台开发，版本使用最新的`3.x`版本。

频繁去磁盘进行交互，可以处理大规模数据集。



#### Hadoop集群搭建

<img src="/Users/mungeryang/Desktop/Bigdata/pic/Hadoop集群架构.png" style="zoom:45%;" />

Hadoop集群具体来说包括两个集群：HDFS集群和YARN集群，两者逻辑上分离，但在物理上是一起的。

HDFS负责海量数据的存储，YARN集群主要负责海量数据运算时的实时调度，MapReduce主要是一个分布式编程框架包括代码和软件包。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/had架构.png" style="zoom:43%;" />

<center>HDFS架构
<img src="pic/yarn%E6%9E%B6%E6%9E%84.webp" style="zoom:55%;" />


<center>YARN架构


~~~shell
#Hadoop配置文件的主要目录
#解压与安装
[root@node1 hadoop]# /export/server/hadoop-3.3.0/etc/hadoop
#----------------------------------
[root@node1 software]# tar -zxvf hadoop-3.3.0-Centos7-64-with-snappy.tar.gz -C /export/server/
[root@node1 hadoop-3.3.0]# mkdir -p /export/server/hadoop-3.3.0/data




~~~

#### 配置core-site.xml

~~~shell
[root@node1 hadoop]# vim core-site.xml
<configuration>
<!-- 指定HDFS 中NameNode 的地址-->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://node1:8020</value>
</property>
<!-- 指定Hadoop 运行时产生文件的存储目录-->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/export/server/hadoop-3.3.0/data</value>
</property>
<!-- 在web UI访问HDFS使用的用户名 -->
<property>
    <name>hadoop.http.staticuser.user</name>
    <value>root</value>
</property>
~~~

配置hdfs-site.xml

~~~shell
[root@node1 hadoop]# vim hdfs-site.xml
<!-- 指定Hadoop 辅助名称节点主机配置-->
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>node2:9868</value>
</property>
~~~

#### 配置mapred-site.xml

~~~shell
[root@node1 hadoop]# vim mapred-site.xml
<configuration>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
<!-- 指定MR 运行在YARN 上-->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>

<!-- MR App Master环境变量 -->
<property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>

<!-- MR MapTask环境变量 -->
<property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<!-- MR ReduceTask环境变量 -->
<property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
</configuration> 
~~~

#### 配置yarn-site.xml

~~~shell
[root@node1 hadoop]# vim yarn-site.xml
<configuration>
<!-- Reducer 获取数据的方式-->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN 的ResourceManager 的地址-->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>node1</value>
</property>

<!-- 服务器请求最小内存 -->
<property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>512</value>
</property>

<!-- 服务器请求最大内存 -->
<property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>2048</value>
</property>

<!-- 容器虚拟内存与物理内存之间的比率 -->
<property>
    <name>yarn.nodemanager.vmem-pmem-ratio</name>
    <value>4</value>
</property>
</configuration>
~~~

#### 配置workers

~~~shell
[root@node1 hadoop]# vim workers 
node1
node2
node3
~~~

#### 配置hadoop-env.sh

~~~shell
[root@node1 hadoop]# vim hadoop-env.sh
export JAVA_HOME=/export/server/jdk1.8.0_241

export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
~~~

#### 配置环境变量

~~~shell
[root@node1 hadoop]# vim /etc/profile
export HADOOP_HOME=/export/server/hadoop-3.3.0
export PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
~~~



#### 配置文件传输到其他节点

~~~shell
[root@node1 server]# scp -r /etc/profile node2:$PWD  
[root@node1 server]# scp -r /etc/profile node3:$PWD
[root@node1 server]# scp /etc/profile node2:/etc 
[root@node1 server]# scp /etc/profile node3:/etc
~~~

格式化与一键启动

~~~shell
#只能格式化一次
[root@node1 hadoop-3.3.0]# bin/hdfs namenode -format

[root@node1 sbin]# start-dfs.sh
[root@node1 sbin]# stop-dfs.sh
[root@node1 sbin]# start-yarn.sh
[root@node1 sbin]# stop-yarn.sh
#一键启动YARN、HDFS
start-all.sh
#一键关闭YARN、HDFS
start-all.sh

#单独启动或关闭集群
hdfs -daemon start namenode
hdfs -daemon stop namenode
hdfs -daemon start datanode
hdfs -daemon stop datanode
~~~



### 文件存储与分布式存储

单个硬盘的存储能力是有限的，如果需要存储更多的数据，可以通过某种技术，将若干个硬盘连接起来，提供能耗的储存能力。在我们服务器上插更多的磁盘来提高存储容量，而服务器的插槽是有限的，我们无法无限的增加硬盘。

文件系统非常抽象，是一种资源管理工具，方便用户管理文件存储，Linux是典型的`树形`文件系统。

**纵向扩展：**不停的加硬盘堆叠

**横向扩展：**不停的加主机–`分布式`

理论上，横向扩展可以无限制进行下去，因此就是海量数据存储下的解决问题的方式——分布式存储。大文件切片存储到不同的主机上进行存储计算。

**如何解决数据查询便捷问题呢？**

当文件被分布式存储到多台主机上时，后序获取文件依靠一台一台查询是不靠谱的，需要借助元数据记录来解决查询问题，把文件和其存储的机器的位置信息记录下来，类似于图书馆的查阅系统，这样就可以迅速定位文件存储到了那一台主机上了。

**如何解决大文件传输效率慢的问题呢？**

大数据场景下，TB、PB、GB级别的文件是常见的。当单个文件过大时，通常采用分块存储，把一个大文件分割拆分成若干个小块(block，简写为blk)分别存储到不同的机器上，并行操作提高效率。

**如何解决数据丢失问题？**

冗余存储采用副本机制，副本越多数据丢失风险越小。

**如何解决用户查询视角统一问题？**

随着存储的进行，数据文件越来越多，与之对应的元数据也越来越多。可以把分布式文件系统的元数据记录这一块也抽象成统一的目录树结构。

通常一个文件系统需要具备：`分布式特性`、`分块存储`、`副本机制`、`元数据记录`、`抽象目录树`、`统一的namespace命名空间`。



### 分布式文件系统-HDFS

**需求**是做系统设计的关键

HDFS(Hadoop Distributed File System)设计初衷是为了能够支持提高吞吐和超大文件的读写操作，非常适合操作存储大型数据。

超大数据集

流式数据访问

简单一致性模型

#### 主从架构

HDFS采用master/slave架构。一般一个集群是有一个Namenode和一定数目的Datanode组成。==Namenode是主节点==，==Datanode是从节点==，各司其职，协调完成分布式文件存储服务。

Namenode负责维护文件系统的名字空间，任何对文件系统名字空间或属性的修改都将被Namenode记录下来。应用程序可以设置HDFS保存的文件的副本数目。文件副本的数目称为文件的副本系数，这个信息也是由Namenode保存的。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/主从架构.png" style="zoom:55%;" />

NameNode：

- 保存HDFS整个集群的元数据
- NameNode需要知道每一个DataNode上的block信息
- 客户端在上传或下载文件时，需要从NameNode设置或者获取元数据信息
- NameNode的元数据是保存在内存中的，但是会定时保存到硬盘

DataNode：

- 保存具体文件数据
- 定时与NameNode之间发送心跳包
- 定时向NameNode汇报block信息
- 客户端下载或者上传文件时，具体的文件操作是和DataNode进行交互

Secondary NameNode

- 辅助NameNode进行元数据管理(元数据持久化存储，保存到硬盘)

Client：

- 负责上传文件和下载文件的发起工作
- 在上传文件时会对文件进行切片

<img src="/Users/mungeryang/Desktop/Bigdata/pic/HDFS archive.png" style="zoom:40%;" />



#### 分块机制

HDFS中的所有文件都是物理上分块操作的，blocksize的默认大小是128M，block只是一个单位。

BLOCK的大小可以通过hdfs-site.xml中的dfs.blocksize参数来设置

#### 副本机制

- HDFS默认保存3个副本
- 保存副本数量可以通过dfs.replication参数来设置

#### NameSpace

HDFS会给每一个存储的文件提供统一的访问路径

~~~shell
#格式
hdfs://namenode:port/dir-a/file.data
#使用-使用绝对前缀路径
hdfs://node1:8020/dir-a/a.txt
~~~

#### HDFS机架感知

- 第一个BLOCK会存储在离客户端最近的一台主机上
  - 如果客户端也是集群中的一台主机，则直接存在客户端所在主机上。
  - 如果Client不在集群范围内或者不在同一个子网，则会随机挑选机架将BLOCK存入
- 第二个BLOCK副本会随机选择健康主机进行数据存储
- 第三个BLOCK副本会在第二个BLOCK副本主机上随机选择另外一台健康主机，将BLOCK数据存入。

#### HDFS安全机制

在NameNode启动过程中，等待DataNode汇报可用的block信息。在此期间，NameNode保持在安全模式。随着NameNode的block汇报持续进行，当整个系统达到安全标准时，HDFS自动断开安全模式。在NameNode Web页面上还是会显示安全模式是打开还是关闭。

~~~shell
hdfs dfsadmin -safemode get #查看安全模式状态
hdfs dfsadmin -safemode enter #进入安全模式
hdfs dfsadmin -safemode leave #离开安全模式
~~~



### HDFS的shell命名

~~~shell
#格式
hadoop fs [-命令] [参数]
hdfs dfs [-命令] [参数]

#查看进程情况
jps

#文件的上传
hadoop fs -put a.txt /dir
hsfs dfs -put a.txt /dir

#文件的下载
hadoop fs -get /dira/a.txt /root
hdfs dfs -get /dira/a.txt /root

#文件夹的创建
hadoop fs -mkdir -p /aaa/bbb/ccc
hdfs dfs -mkdir /aaa

#文件的删除
hadoop fs -rm /c.txt
#递归删除目录及目录里面内容
hadoop fs -rm -r /c.txt

#文件的复制
hadoop fs -cp /dir/1.txt /
hdfs dfs -cp /dir/1.txt

#文件的删除
hadoop fs -mv /dir/1.txt
hdfs dfs -mv /dir/1.txt

#权限设置
hadoop fs -chmod 777 /dira/a.txt
hdfs dfs -chmod 777 /dira/a.txt

#统计文件夹或者文件的大小信息
hadoop fs -du /dira
hdfs dfs -du /dira

#修改文件
hadoop fs -vi /app1/test1.java
hdfs dfs -vi /app1/test1.java

#设置副本数量
hadoop fs -setrep 10 /test/test2.java
hdfs dfs -setrep 10 /test/test2.java

#创建空文件夹
hadoop fs -touchz /name.txt
hdfs dfs -touchz /name.txt
~~~

### HDFS读写流程

因为NameNode维护管理文件系统的元数据信息，这就造成了不管是读还是写数据都是基于NameNode开始的，也就是说，==NameNode成了HDFS访问的唯一入口==。入口地址:node1:8080

**读数据**

<img src="/Users/mungeryang/Desktop/Bigdata/pic/write-data.png" style="zoom:53%;" />

- HDFS客户端通过对DistributedFileSystem 对象调用**create()**请求创建文件。
- DistributedFileSystem对namenode进行**RPC调用**，请求上传文件。namenode执行各种检查判断：目标文件是否存在、父目录是否存在、客户端是否具有创建该文件的权限。检查通过，namenode就会为创建新文件记录一条记录。否则，文件创建失败并向客户端抛出一个IOException。
- DistributedFileSystem为客户端返回FSDataOutputStream输出流对象。由此客户端可以开始写入数据。FSDataOutputStream是一个包装类，所包装的是DFSOutputStream。
- 在客户端写入数据时，DFSOutputStream将它分成一个个数据包（**packet** 默认64kb）,并写入一个称之为数据队列（data queue）的内部队列。DFSOutputStream有一个内部类做DataStreamer，用于请求NameNode挑选出适合存储数据副本的一组DataNode。这一组DataNode采用**pipeline机制**做数据的发送。默认是3副本存储。
- DataStreamer将数据包流式传输到pipeline的第一个datanode,该DataNode存储数据包并将它发送到pipeline的第二个DataNode。同样，第二个DataNode存储数据包并且发送给第三个（也是最后一个）DataNode。
- DFSOutputStream也维护着一个内部数据包队列来等待DataNode的收到确认回执，称之为确认队列（ack queue）,收到pipeline中所有DataNode确认信息后，该数据包才会从确认队列删除。
- 客户端完成数据写入后，将在流上调用close()方法关闭。该操作将剩余的所有数据包写入DataNode pipeline，并在联系到NameNode告知其文件写入完成之前，等待确认。
- 因为namenode已经知道文件由哪些块组成（DataStream请求分配数据块），因此它仅需等待最小复制块即可成功返回。
- 数据块最小复制是由参数dfs.namenode.replication.min指定，默认是1.

**写数据**

<img src="/Users/mungeryang/Desktop/Bigdata/pic/read-data.png" style="zoom:53%;" />

- 客户端通过调用DistributedFileSystem对象上的open()来打开希望读取的文件。
- DistributedFileSystem使用**RPC调用**namenode来确定文件中**前几个块**的块位置。对于每个块，namenode返回具有该块副本的datanode的地址，并且datanode根据块与客户端的距离进行排序。注意此距离指的是**网络拓扑中的距离**。比如客户端的本身就是一个DataNode，那么从本地读取数据明显比跨网络读取数据效率要高。
- DistributedFileSystem将FSDataInputStream（支持文件seek定位读的输入流）返回到客户端以供其读取数据。FSDataInputStream类转而封装为DFSInputStream类，DFSInputStream管理着datanode和namenode之间的IO。
- 客户端在流上调用read()方法。然后，已存储着文件前几个块DataNode地址的DFSInputStream随即连接到文件中第一个块的最近的DataNode节点。通过对数据流反复调用read()方法，可以将数据从DataNode传输到客户端。
- 当该块快要读取结束时，DFSInputStream将关闭与该DataNode的连接，然后寻找下一个块的最佳datanode。这些操作对用户来说是透明的。所以用户感觉起来它一直在读取一个连续的流。
- 客户端从流中读取数据时，块是按照打开DFSInputStream与DataNode新建连接的顺序读取的。它也会根据需要询问NameNode来检索下一批数据块的DataNode位置信息。一旦客户端完成读取，就对FSDataInputStream调用close()方法。
- 如果DFSInputStream与DataNode通信时遇到错误，它将尝试该块的下一个最接近的DataNode读取数据。并将记住发生故障的DataNode，保证以后不会反复读取该DataNode后续的块。此外，DFSInputStream也会通过校验和（checksum）确认从DataNode发来的数据是否完整。如果发现有损坏的块，DFSInputStream会尝试从其他DataNode读取该块的副本，也会将被损坏的块报告给namenode 。



### SecondaryNamenode checkpoint机制

**SecondaryNameNod**e的职责是**合并NameNode的edit logs到fsimage文件中**。

**fsimage**文件是Hadoop文件系统元数据的一个永久性的检查点，包含Hadoop文件系统中的所有目录和文件idnode的序列化信息。**edits log**文件存放的是Hadoop文件系统的所有更新操作记录日志，文件系统客户端执行的所有写操作首先会被记录到edits文件中。

**Checkpoint**核心是把fsimage与edits log合并以生成新的fsimage的过程。此过程有两个好处：fsimage版本不断更新不会太旧、edits log文件不会太大。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/checkpoint.png" style="zoom:43%;" />



当触发checkpoint操作条件时，SNN发生请求给NN滚动edits log。然后NN会生成一个新的编辑日志文件：edits new，便于记录后续操作记录。同时SNN会将edits文件和fsimage复制到本地（使用HTTP GET方式）

SNN首先将fsimage载入到内存，然后一条一条地执行edits文件中的操作，使得内存中的fsimage不断更新，这个过程就是edits和fsimage文件合并。合并结束，SNN将内存中的数据dump生成一个新的fsimage文件。SNN将新生成的Fsimage new文件复制到NN节点。

至此刚好是一个轮回，等待下一次checkpoint触发SecondaryNameNode进行工作，一直这样循环操作。

**触发机制**

Checkpoint触发条件受两个参数控制，可以通过**core-site.xml**进行配置：

~~~java
dfs.namenode.checkpoint.period=3600  //两次连续的checkpoint之间的时间间隔。默认1小时
dfs.namenode.checkpoint.txns=1000000 //最大没有执行checkpoint事务的数量，满足将强制执行紧急checkpoint，即使尚未达到检查点周期。默认100万事务数量。
~~~

从上面的描述我们可以看出，**SecondaryNamenode根本就不是Namenode的一个设备，只是将fsimage和edits合并**。





### HDFS的Java API代码实现

Java的注解和反射

**注解**并不是程序本身，可以对程序做出一定的解释。可以被其他程序(如：编译器)读取。

注解命名格式一般都是以`@注释名`在代码中存在。

~~~java
//重写的注解
@Override
public static void method(){
    method.method1 = ...；
}
~~~

**反射**是Java被视为“动态语言”的关键，反射机制允许程序在执行期间借助于Reflection API取得任何类内部的信息，并能注解操作任意对象内部属性及方法。

注解使用：

- 提供信息给编译器
- 编译阶段时的处理
- 运行时的处理

---------

为了让我们的win系统顺利运行hadoop程序代码，我们要为电脑安装hadoop winutils并配置好环境变量。

在Java中操作HDFS，主要涉及以下Class：

**Configuration:**该类的对象封转了客户端或者服务器的配置

**FileSystem:**该类的对象是一个文件系统对象，可以用该对象的一些方法来对文件进行操作，通过FileSystem的静态方法get获得该对象

~~~java
FileSystem fs = FileSystem.get(conf);
~~~

#### maven依赖

~~~xml
<repositories>
    <repository>
        <id>cental</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public//</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
    </dependency>

    <!-- Google Options -->
    <dependency>
        <groupId>com.github.pcj</groupId>
        <artifactId>google-options</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
</dependencies>
~~~

#### 遍历HDFS文件目录

~~~java
//遍历HDFS中的所有文件
    @Test
    public void getFileSystem() throws IOException, InterruptedException {
        FileSystem fileSystem = FileSystem.get(URI.create("hdfs://node1:8020"), new Configuration(), "root");
        RemoteIterator<LocatedFileStatus> locatedFileStatusRemoteIterator = fileSystem.listFiles(new Path("/"), true);
        while (locatedFileStatusRemoteIterator.hasNext()){
            LocatedFileStatus next = locatedFileStatusRemoteIterator.next();
            System.out.println(next.getPath().toString());
        }
    }
~~~

#### 创建文件夹

~~~java
//创建文件夹
    @Test
    public void mkdir() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://node1:8020"), new Configuration(), "root");
        boolean mkdirs = fileSystem.mkdirs(new Path("/test/java"));
        fileSystem.close();
    }
~~~

#### 下载文件

~~~java
//下载文件1--直接拷贝(hdfs地址->本机地址)
    @Test
    public void downloadFile() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://node1:8020"), new Configuration(), "root");
        fileSystem.copyFromLocalFile(new Path("/anaconda-ks.cfg"),new Path("D:\\hadoop\\test"));

    }

//下载文件2--文件流式处理(输入流->输出流)
    @Test
    public void getLoaclfile() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://node1:8020"), new Configuration(), "root");
        FSDataInputStream inputStream = fileSystem.open(new Path("/timer.txt"));
        FileOutputStream fileOutputStream = new FileOutputStream(new File("e:\\timer.txt"));
        IOUtils.copy(inputStream,fileOutputStream);
        IOUtils.closeQuietly(inputStream);
        IOUtils.closeQuietly(fileOutputStream);
        fileSystem.close();
    }
~~~

#### 上传文件

```java
//上传文件--直接拷贝(本机地址->hdfs地址)
@Test
public void uploadFile() throws URISyntaxException, IOException, InterruptedException {
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://node1:8020"), new Configuration(), "root");
    fileSystem.copyFromLocalFile(new Path("D:/hadoop/test.java"),new Path("/test/mydir"));
    fileSystem.close();
}
```

#### 小文件合并

~~~java
@Test
public void mergeFile() throws URISyntaxException, IOException, InterruptedException {
    //获取分布式文件系统
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://node1:8020"), new Configuration(), "root");     
    FSDataOutputStream outputStream = fileSystem.create(new Path("/bigfile.txt"));   
    //获取本地文件系统 
    LocalFileSystem local = fileSystem.getLocal(new Configuration());
    //通过本地文件系统获取文件列表
    FileStatus[] fileStatuses = local.listStatus(new Path("file:///E:\\input"));
    for (FileStatus fileStatus : fileStatuses) { 
        FSDataInputStream inputStream = local.open(fileStatus.getPath());
        IOUtils.copy(inputStream,outputStream);
        IOUtils.closeQuietly(inputStream);
    }
    IOUtils.closeQuietly(outputStream);  
    local.close();   
    fileSystem.close();
}
~~~



#### 集群内部文件传输

在Linux系统中，文件的传输统一使用scp指令操作。

本地文件复制到远程

~~~shell
scp -r local folder remote username@remote ip:remote folder
#注意，如果实现了ssh免密登录之后，则不需要输入密码即可拷贝。

#复制文件-将 /root/test.txt 拷贝到 192.168.88.161 的 /root/ 目录下，文件名还是 text.txt，使用 root 用户，此时会提示输入远程 root 用户的密码。
scp  /root/test.txt root@192.168.88.161:/root/

#复制文件并重命名-将 /root/test.txt 拷贝到 192.168.88.161 的 /root/ 目录下，文件名是 text1.txt，使用 root 用户，此时会提示输入远程 root 用户的密码。
scp /root/test.txt root@192.168.88.161:/root/text1.txt

#复制目录-将整个目录 /root/test/ 复制到 192.168.88.161 的 /root/ 下，即递归的复制，使用 root 用户，此时会提示输入远程 root 用户的密码。
scp -r /root/test root@192.168.88.161:/root/
~~~

远程文件复制到本地

远程复制到本地 与 从本地复制到远程命令类似，不同的是远程文件作为源文件在前，本地文件作为目标文件在后。

~~~shell
#复制文件-将192.168.88.162的/root目录下的test.txt拷贝到当前主机的/root/目录下，文件名不变
scp root@192.168.88.162:/root/test.txt /root/test.txt
~~~



### HDFS分布式拷贝工具-DistCp

DistCp是Apache Hadoop中的一种流行工具，在hadoop-tools工程下，作为独立子工程存在。其定位就是用于数据迁移的，定期在集群之间和集群内部备份数据。（在备份过程中，==每次运行DistCp都称为一个备份周期==。）尽管性能相对较慢，但它的普及程度已经越来越高。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/distCp.png" style="zoom:60%;" />

DistCp指令

~~~shell
$ hadoop distcp
usage: distcp OPTIONS [source_path...] <target_path>
append #拷贝文件时支持对现有文件进行追加写操作
async  #异步执行distcp拷贝任务
bandwidth #对每一个Map任务的带宽限速
delete #删除相对源端，目标端多出来的文件
diff <arg> #快照diff信息进行数据同步
overwrite	#覆盖方式拷贝，目标端已经存在则直接覆盖
p <arg>		#拷贝数据时，扩展属性信息的保留，包括权限信息、块大小信息
skipcrccheck	#拷贝数据时是否跳过checksum校验
update		#拷贝数据时只拷贝源端，目标端不存在文件数据
cd /export/server/hadoop-3.3.0/bin/hadoop distcp hdfs ://node1.8020/jdk-8u241-linux-x64.tar.gz hdfs://cluster2:8020/
~~~



### Hadoop Archive归档

HDFS并不擅长存储小文件，因为每个文件最少一个block，每个block的元数据都会在NameNode占用内存，如果存在大量的小文件，它们会吃掉NameNode节点的大量内存。

~~~shell
[root@node1 ~]# hadoop fs -mkdir /smallfile
[root@node1 ~]# echo 1 > 1.txt
[root@node1 ~]# echo 2 > 2.txt
[root@node1 ~]# echo 3 > 3.txt
[root@node1 ~]# hadoop fs -put 1.txt 2.txt 3.txt /smallfile
~~~

Hadoop Archives可以有效的处理以上问题，它可以把多个文件归档成为一个文件，归档成一个文件后还可以透明的访问每一个文件。

注意：Archive归档是通过MapReduce程序完成的，需要启动YARN集群。

~~~shell
#这样就会在/outputdir目录下创建一个名为test.har的存档文件。
[root@node1 ~]# hadoop archiveName test.bar -p /smallfile/outputdir

#查看归档后的文件
hadoop fs -ls /outputdir/test.bar
~~~

可以看到har文件包括：两个索引文件，多个part文件（本例只有一个）以及一个标识成功与否的文件。part文件是多个原文件的集合， 通过index文件可以去找到原文件。

在查看har文件的时候，如果没有指定访问协议，默认使用的就是hdfs://，此时所能看到的就是归档之后的样子。

Hadoop Archives的URI是：

har://scheme-hostname:port/archivepath/fileinarchive  

~~~shell
[root@node1 ~]# hadoop fs -ls har://hdfs-node1:8020/outputdir/test.har/
[root@node1 ~]# hadoop fs -ls har:///outputdir/tesr.har
[root@node1 ~]# hadoop fs -cat har:///outputdir/test.har/1.txt
~~~

提取Archive

~~~shell
[root@node1 ~]# hadoop fs -mkdir /smallfile1
[root@node1 ~]# hadoop fs -cp har:///outputdir/test.har/* /smallfile1
[root@node1 ~]# hadoop fs -ls /smallfile1
~~~

要并行解压存档，请使用DistCp,对应大的归档文件可以提高效率：

~~~shell
[root@node1 ~]# hadoop distcp har:///outputdir/test.har/* /smallfile2
~~~

- Hadoop archives是特殊的档案格式。一个Hadoop archive对应一个文件系统目录。Hadoop archive的扩展名是***.har**
- 创建archives本质是运行一个Map/Reduce任务，所以应该在Hadoop集群上运行创建档案的命令
- 创建archive文件要消耗和原文件一样多的硬盘空间
- archive文件不支持压缩，尽管archive文件看起来像已经被压缩过
- archive文件一旦创建就无法改变，要修改的话，需要创建新的archive文件。事实上，一般不会再对存档后的文件进行修改，因为它们是**定期存档的，比如每周或每日**
- 当创建archive时，源文件不会被更改或删除



### Trash垃圾回收

**HDFS中是没有回收站垃圾桶**概念的，删除操作的数据将会被直接删除，没有后悔药。

~~~shell
[root@node1 ~]# hadoop fs -rm /smallfile1/1.txt
~~~

执行操作后文件就删除了，后悔该怎么办呢？

**Trash**机制，叫做回收站或者垃圾桶。Trash就像Windows操作系统中的回收站一样。它的目的是防止你无意中删除某些东西。默认情况下是不开启的。

启用Trash功能后，从HDFS中删除某些内容时，文件或目录不会立即被清除，它们将被移动到回收站Current目录中(/user/${username}/.Trash/current)。

.Trash中的文件在用户==可配置的时间延迟后==被永久删除。也可以简单地将回收站里的文件移动到.Trash目录之外的位置来恢复回收站中的文件和目录。

#### Trash checkpoint

检查点仅仅是用户回收站下的一个目录，用于存储在创建检查点之前删除的所有文件或目录。

最近删除的文件被移动到回收站Current目录，并且在可配置的时间间隔内，HDFS会为在Current回收站目录下的文件创建检查点/user/${username}/.Trash/<日期>，并在过期时删除旧的检查点。

~~~shell
[root@node1 ~]# hdfs://node1:8020/user/root/.Trash
~~~



#### Trash功能开启

关闭HDFS集群：stop-dfs.sh

修改core-site.xml：

~~~shell
[root@node1 ~]# vim /export/server/hadoop-3.1.4/etc/hadoop/core-site.xml
~~~

~~~xml
<property>  
    <name>fs.trash.interval</name>  
    <value>1440</value>  
</property>  
<property>  
    <name>fs.trash.checkpoint.interval</name>  
    <value>0</value>  
</property>
~~~

**fs.trash.interval**：分钟数，当超过这个分钟数后检查点会被删除。如果为零，Trash回收站功能将被禁用。

**fs.trash.checkpoint.interval**：检查点创建的时间间隔(单位为分钟)。其值应该小于或等于fs.trash.interval。如果为零，则将该值设置为fs.trash.interval的值。每次运行检查点时，它都会从当前版本中创建一个新的检查点，并删除在数分钟之前创建的检查点。

同步集群配置文件

~~~shell
[root@node1 ~]# scp -r /export/server/hadoop-3.3.0/etc/hadoop/core-site.xml node2:/export/server/hadoop-3.3.0/etc/hadoop/
[root@node1 ~]# scp -r /export/server/hadoop-3.3.0/etc/hadoop/core-site.xml node3:/export/server/hadoop-3.3.0/etc/hadoop/
~~~

启动HDFS集群：start-dfs.sh



#### 功能使用

开启Trash功能后，正常执行删除操作，文件实际并不会被直接删除，而是被移动到了垃圾回收站。

虽然是删除操作但是底层是移动操作。

而有的时候，我们希望直接把文件删除，不需要再经过Trash回收站了，可以在执行删除操作的时候添加一个参数：**-skipTrash.**

~~~shell
[root@node1 ~]# hadoop fs -rm -skipTrash /smallfile1/3.txt
~~~

#### 恢复与清空文件

回收站里面的文件，在到期被自动删除之前，都可以通过命令恢复出来。使用mv、cp命令把数据文件从Trash目录下复制移动出来就可以了。

~~~shell
[root@node1 ~]# hadoop fs -mv /usr/root/.Trash/Current/smallfile1/* /smallfile1/
~~~

除了fs.trash.interval参数控制到期自动删除之外，用户还可以通过命令手动清空回收站，释放HDFS磁盘存储空间。

~~~shell
[root@node1 ~]# hadoop fs -expunge
#该命令立即从文件系统中删除过期的检查点。
~~~

###  Snapshot快照

**HDFS snapshot**是HDFS整个文件系统，或者某个目录在某个时刻的镜像。该镜像并不会随着源目录的改变而进行动态的更新。

HDFS快照的核心功能包括：数据恢复、数据备份、数据测试。

数据恢复：可以通过滚动的方式来对重要的目录进行创建snapshot的操作，这样在系统中就存在针对某个目录的多个快照版本。当用户误删除掉某个文件时，可以通过最新的snapshot来进行相关的恢复操作。

数据备份：可以使用snapshot来进行整个集群，或者某些目录、文件的备份。管理员以某个时刻的snapshot作为备份的起始结点，然后通过比较不同备份之间差异性，来进行增量备份。

数据测试：在某些重要数据上进行测试或者实验，可能会直接将原始的数据破坏掉。可以临时的为用户针对要操作的数据来创建一个snapshot，然后让用户在对应的snapshot上进行相关的实验和测试，从而避免对原始数据的破坏。

**快照不是数据的简单拷贝，快照只做差异的记录**



#### 快照命令

HDFS中可以针对整个文件系统或者文件系统中某个目录创建快照，但是**创建快照的前提是相应的目录开启快照的功能**。

~~~shell
#启用快照功能
hdfs dfsadmin -allowSnapshot /allenwoon
#禁用快照功能
hdfs dfsadmin -allowSnapshot /allenwoon

#对指定目录创建快照
hdfs dfsadmin -createSnapshot /alloenwoon #系统自动生成快照名称
hdfs dfsadmin -createSnapshot /alloenwoon mysnapl  #//指定名称创建快照

#重命名快照
hdfs dfsadmin -renameSnapshot /alloenwoon mysnap1 nysnap2
#列出当前用户所有可以快照的目录
hdfs lsSnapshottableDir

#比较两个快照的不同之处
echo 222 > 2.txt
hadoop fs -appendToFile 2.txt /allenwoon/1.txt
hadoop fs -cat /allenwoon/1.txt
1
222
hdfs dfs -creatSnapshot /allenwoon mysnap3
hadoop fs -put zookeeper.out /allenwoon
hdfs dfs -creatSnapshot /allenwoon mysnap4

hdfs snapshotDiff /allenwoon mysnap2 mysnap3
hdfs snapshotDiff /allenwoon mysnap2 mysnap4
~~~

**+** The file/directory has been **created**.

**-** The file/directory has been **deleted**.

**M** The file/directory has been **modified**.

**R** The file/directory has been **renamed**.

~~~shell
#删除快照
dfs -deleteSnapshot /allenwoon mysnap4
#删除所有快照
hadoop fs -rm -r /allenwoon
~~~



### HDFS权限管理

作为分布式文件系统，HDFS也集成了一套兼容POSIX的权限管理系统。客户端在进行每次文件操时，系统会从**用户身份认证**和**数据访问授权**两个环节进行验证。

HDFS的文件权限与Linux/Unix系统的UGO模型类型类似，可以简单描述为：每个文件和目录都与一个所有者和一个组相关联。

该文件或目录对作为**所有者（USER）**的用户，作为该**组成员的其他用户（GROUP）**以及对所有**其他用户（OTHER）**具有单独的权限。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/hdfs权限.png" style="zoom:43%;" />

在HDFS中，对于文件，需要r权限才能读取文件，而w权限才能写入或追加到文件。没有x可执行文件的概念。Linux中**umask可用来设定权限掩码**。

权限掩码是由3个八进制的数字所组成，将现有的存取权限减掉权限掩码后，即可产生建立文件时预设的权限。

HDFS也提供了umask掩码，用于设置在HDFS中默认新建的文件和目录权限位。默认umask值有属性fs.permissions.umask-mode指定，默认值022。

创建文件和目录时使用的umask，默认的权限就是：777-022=755。也就是drwxr-xr-x。

#### UGO权限命令

~~~shell
hadoop fs -chmod 750 /user/itcast/foo	#变更目录或文件的权限位
hadoop fs -chown:portal /user/itcast/foo #变更目录或文件的属主或用户组
hadoop fs -chgrp itcast group1 /user/itcast/foo	#变更用户组,使用这个命令的必须是超级用户，或者是该文件的属主，勇士也是用户组的成员
~~~

Hadoop3.0之后，支持在HDFS **Web****页面上使用鼠标修改。



### HDFS High Availability (HA)高可用

**单点故障**（single point of failure，**SPOF**）是指系统中某一点一旦失效，就会让整个系统无法运作，换句话说，单点故障即会整体故障。

**高可用性**（high availability，**HA**），IT术语，指系统无中断地执行其功能的能力，代表系统的可用性程度。高可用性系统意味着系统服务可以更长时间运行，通常通过提高系统的容错能力来实现。

高可用性或者高可靠度的系统==不会希望有单点故障造成整体故障的情形==。一般可以透过冗余的方式`增加`**多个相同机能的部件**，只要这些部件没有同时失效，系统（或至少部分系统）仍可运作，这会让可靠度提高。

#### 主-备集群

当下企业中成熟的做法就是给单点故障的位置设置备份，形成主备架构。通俗描述就是**当主挂掉，备份顶上**，短暂的中断之后继续提供服务。

常见的是**一主一备**架构，当然也可以一主多备。备份越多，容错能力越强，与此同时，冗余也越大，浪费资源。

**Active**：主角色。活跃的角色，代表正在对外提供服务的角色服务。==任意时间有且只有一个active对外提供服务==。

**Standby**：备份角色。需要和主角色保持数据、状态同步，并且时刻准备切换成主角色（当主角色挂掉或者出现故障时），对外提供服务，保持服务的可用性。

#### HA系统设计

**脑裂**(split-brain)是指“大脑分裂”,本是医学名词。在HA集群中，脑裂指的是当联系主备节点的"心跳线"断开时(即两个节点断开联系时)，本来为一个整体、动作协调的HA系统，就分裂成为两个独立的节点。由于相互失去了联系，主备节点之间像"裂脑人"一样，使得整个集群处于混乱状态。

1）**集群无主**：都认为对方是状态好的，自己是备份角色，后果是无服务；

2）**集群多主**：都认为对方是故障的，自己是主角色。

避免脑裂问题的核心是：保持任意时刻系统有且只有一个主角色提供服务。

数据同步常见做法是：**通过日志重演操作记录**。主角色正常提供服务，发生的事务性操作通过日志记录，备用角色读取日志重演操作。



### HDFS NAMENODE单点故障

HDFS高可用性解决方案：在同一群集中运行两个（从3.0.0起，超过两个）冗余NameNode。这样可以在机器崩溃的情况下快速故障转移到新的NameNode，或者出于计划维护的目的由管理员发起的正常故障转移。

**QJM解决方案**

QJM全称**Quorum Journal Manager**，由cloudera公司提出，是Hadoop官方推荐的HDFS HA解决方案之一。

QJM中，使用zookeeper中ZKFailoverController来实现主备切换；使用Journal Node（JN）集群实现edits log的共享以达到数据同步的目的。

Zookeeper的下列特性功能参与了HDFS的HA解决方案中：

- 临时Znode：客户端断开连接session结束，znode将会被自动删除。
- Path唯一路径：zookeeper中维持了一份类似目录树的数据结构。每个节点称之为Znode。Znode具有唯一性，不会重名。也可以理解为排他性。
- 监听机制：客户端可以针对znode上发生的事件设置监听，当事件发生触发条件，zk服务会把事件通知给设置监听的客户端。

运行NameNode的每台计算机也都运行**ZKFailoverController**，**ZKFailoverController**的主要职责：

- 监视和管理NameNode健康状态
- 维持和Zookeeper集群的联系

故障转移过程也就是俗称的主备角色切换的过程，切换过程中最怕的就是脑裂的发送。因此需要**Fencing机制**来避免，将先前的Active节点隔离，然后将本地NameNode转换为Active状态。

**Journal Node**（JN）集群是轻量级分布式系统，主要用于高速读写数据、存储数据。通常使用2N+1台JournalNode存储共享Edits Log（编辑日志）。

当发生故障Active NN挂掉后，Standby NN 会在它成为Active NN 前，读取所有的JN里面的修改日志，这样就能高可靠的保证与挂掉的NN的目录镜像树一致，然后无缝的接替它的职责，维护来自客户端请求，从而达到一个高可用的目的。



### 集群搭建的准备

#### 修改Linux主机名

~~~shell
vim /etc/hostname
node1
node2
node3
~~~

#### 修改IP地址

~~~shell
[root@localhost network-scripts]#vim /etc/sysconfig/network-scripts/ifcfg-ens33
IPADDR="192.168.88.162" #集群项目只修改ipaddr即可
GATEWAY="192.168.88.2"
NETMASK="255.255.255.0"
DNS1="8.8.8.8"
DNS2="114.114.114.114"
IPV6_PRIVACY="no"
[root@localhost network-scripts]# systemctl restart network
~~~

#### 修改主机名和ip的映射关系

~~~shell
vim /etc/hosts/
~~~

#### 关闭防火墙

~~~shell
systemctl status firewalled
systemctl stop firewalled
~~~

#### ssh免密登录

~~~shell
ssh-keygen
ssh-copy-id
~~~

#### jdk部署与时钟同步

~~~shell
crontab -e
*/1 * * * * /usr/sbin/ntpdate_ntp4.aliyun.com;
~~~



~~~shell
#JN节点关闭与启动
mapred --daemon start historyserver
mapred --daemon stop historyserver
~~~



### Zookeeper框架

<img src="/Users/mungeryang/Desktop/Bigdata/pic/zoo_logo.png" style="zoom:53%;" />

#### Zookeeper基本知识

**Apache ZooKeeper**是Apache软件基金会的一个软件项目，它为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。ZooKeeper曾经是Hadoop的一个子项目，但现在是一个独立的顶级项目。

- ZooKeeper本质上是==分布式小文件存储系统==
- ZooKeeper有自己的集群，横跨多台主机
- ZooKeeper协调服务主要为其他集群提供服务，单独使用意义不大
- 后期很多大数据框架都需要ZooKeeper来管理，比如Hadoop、Hbase、Kafka
- ZooKeeper也可以理解为小型数据库系统，本身也可以存储数据

ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them, which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

#### ZooKeeper特性

全局数据一致性：客户端在ZooKeeper集群上每存一份数据，则这份数据就在所有的主机之上都有备份数据，以后其他客户端不管访问哪台主机，看到的数据都是一样的。

可靠性：在ZooKeeper集群上只要一份数据发生变化，其他主机上的数据也会发生变化

顺序性：在ZooKeeper集群上，在一台主机上的操作顺序是A、B，则在其他主机上的操作顺序也是A、B，不会发生错乱。

数据更新的原子性：Client在ZooKeeper集群上的任何一个操作，都是事务操作，不可再分，这个操作过程要么全部成果才算成功，只要有一个环节失败，则整体全部失败，数据退回到操作之前的状态。

实时性：Client在ZooKeeper集群上进行操作的时间不会太长，会在有限的时间之内完成近乎实时的获取操作之后的数据。

#### ZooKeeper的角色

<img src="/Users/mungeryang/Desktop/Bigdata/pic/zoo_service.png" style="zoom:70%;" />

**Leader**

- Leader是整个集群的管理者
- Leader可以完成整个集群的事务操作即写操作(增删改)
- Leader也可以完成请求操作
- 一个集群只可以有一个Leader，如果当前Leader宕机，则有其他Follower选举新的Leader

**Follower**

- 只能完成客户端发来的读请求
- 如果Client发来写请求，则需要转发给Leader去进行处理
- 如果Leader挂掉，Follower会参与Leader的选举(政治权利)

**Observer**

- Observer除了不能选举Leader之外，其他功能和Follower一样
- 一个集群，Observer存在的唯一意义就是增加集群对外的读取能力

#### ZooKeeper集群部署

- 集群一般是有奇数台
- ZooKeeper基于java语言开发，所以必须安装jdk

部署ZooKeeper集群一键启动

```shell
[root@node1 onekey]# vim /export/onekey/zk.sh
#!/bin/bash

for num in 1 2 3
do
        ssh root@node${num} "source /etc/profile;zkServer.sh $1"
        echo "正在操作第${num}台主机"
done

[root@node1 onekey]# chmod +x zk.sh

```

#### ZooKeeper数据模型

ZooKeeper是一个树形结构文件系统，类似于Linux系统，以此系统来存储数据。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/zoo_ds.png" style="zoom:70%;" />

名称是由斜杠（/）分隔的一系列路径元素。ZooKeeper命名空间中的每个节点都由路径进行唯一标识。与标准文件系统不同，ZooKeeper命名空间中的每个节点都可以具有与其关联的数据以及子节点。每一个节点都可以存储数据，只是需要注意的是存储的容量是有限，一般不能超过 1MB。

ZooKeeper树中每一个节点被称为一个**Znode**。

- ZooKeeper中的Znode既具有文件的特性又具有文件夹的特性

  文件特性：每一个Znode节点可以存储数据

  文件夹特性：每一个Znode都可以有子节点		

- ZooKeeper具有原子性，当我们在一台主机上创建一个节点时，其他主机也会同时创建节点，这个过程具有原子性不可再分

- Znode节点存储数据有大小限制，由于ZooKeeper是用来管理集群的，每一个节点存储的都是配置信息，所以数据量不会太大，一般都是以KB为单位最多不超过1M

- Znode节点中访问节点数据必须使用**绝对路径**

~~~shell
正确的：/app1/p_3
错误的：p_3
~~~



Znode的类型分为三类：

- 持久节点（`persistent node`）节点会被持久
- 临时节点（`ephemeral node`），客户端断开连接后，ZooKeeper会自动删除临时节点
- 顺序节点（`sequential node`），每次创建顺序节点时，ZooKeeper都会在路径后面自动添加上10位的数字，从1开始，最大是2147483647 （2^32-1）。顺序节点又分为持久顺序节点和临时顺序节点。

1. PRESISTENT
2. EPHEMERAL
3. PRESISTENT_SEQUENTIAL
4. EPHEMERAL_SEQUENTIAL

#### ZooKeeper的shell操作

初始化客户端连接

~~~shell
#连接到本地ZooKeeper
[root@node1 onekey]# zkCli.sh
#连接到指定主机的服务器
[root@node1 onekey]# zkCli.sh -server node1:2181

#1.创建永久节点-不管终端是否关闭，节点永久存在
create /app1 hello
#2.创建永久顺序节点-在节点后面加上一串数字，数字越大表示节点创建的越晚
create -s /app2 world
#3.创建临时节点-节点随着客户端与服务器之间的会话存在而存在
create -e /tempnode world
#4.创建临时有序节点
create -s -e /tempnode2 aaa
#5.创建子节点-临时节点不能有子节点
create /app1/app11 hello11

#获取节点数据
get /app1
get /app1/app11

#修改节点数据
set /app1 hadoop

#删除节点
delete /app1(删除的节点不能有子节点)
rmr /app1


~~~

#### ZooKeeper中的Watch机制

ZooKeeper默认的算法是FastLeaderElection，采用投票数大于半数票则称为新的Leader。

节点状态：

1. LEADING
2. FOLLOWER

Leader选举有两个时机：

- 第一次启动zk选举Leader-编号越大，选举权重越大(myid:/export/server/zookeeper-3.4.6/zkdatas/myid)：票数过半时，谁的myid越大谁就是Leader。
- 在集群工作中Leader选举(前Leader宕机挂掉)-dataVersoin越大权重越高，dataVersion相同时再比较myid进行选举：所有主机都给自己投票，票数过半时，开始选举Leader；票数不足半数，放弃Leader选举。

~~~shell
[zk: node1:2181(CONNECTED) 4] get /app1      
hello
cZxid = 0x300000004	#Znode创建的事务id，一般不会发生变化
ctime = Sat Jan 13 10:34:49 CST 2024 #创建时间
mZxid = 0x300000004	#修改事务id，每次对znode修改都会更新mZxid
mtime = Sat Jan 13 10:34:49 CST 2024
pZxid = 0x300000004
cversion = 0
dataVersion = 0	#数据版本，每次对数据进行修改则值会加1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5	#数据长度
numChildren = 0	#子节点数量
~~~



maven配置jdk时，必须使用java_path才可以，否则虚拟机会报错，相应的项目jdk要和maven中使用的jdk版本保持一致！



### Zookeeper的Java API实现

核心思想是调用`CuratorFrameworkFactory`类实现client方法(==客户端==)

~~~java
public class zkUtils {

    private static CuratorFramework client = null;
    //执行静态代码块-自启动创建客户端和服务器之间的重试策略
    //获取客户端对象
    static{

        ExponentialBackoffRetry backoffRetry = new ExponentialBackoffRetry(1000, 3);
        String server_list = "192.168.88.161:2181,192.168.88.162:2181,192.168.88.163:2181";
        client = CuratorFrameworkFactory.newClient(server_list, backoffRetry);
        client.start();
    }

    //创建节点--withMode
    public static void CreateZnode(CreateMode mode,String path,String...data){

        try {
            if(data.length != 0){
                client.create().creatingParentsIfNeeded().
                        withMode(mode).forPath(path,data[0].getBytes());

            }else{
                client.create().creatingParentsIfNeeded().
                        withMode(mode).forPath(path);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            client.close();
        }

    }

    //删除节点
    public static void deleteZnode(String path){
        try {
            client.delete().deletingChildrenIfNeeded().forPath(path);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            client.close();
        }
    }
	//获取节点信息
    public static String getZnode(String path){
        try {
            byte[] bytes = client.getData().forPath(path);
            String str = new String(bytes);
            System.out.println(str);
        } catch (Exception e) {
            e.printStackTrace();
        }
        finally {
            client.close();
        }
        return null;

    }
    //更新节点信息
    public static void setZnode(String path,String newdata){
        try {
            client.setData().forPath(path,newdata.getBytes());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
~~~



### Zookeeper的集群搭建

~~~shell
[root@node1 ~]# tar -zxvf /export/software/zookeeper-3.4.6.tar.gz -C  /export/server/
[root@node1 ~]# cd /export/server/zookeeper-3.4.6/conf/
[root@node1 conf]# cp zoo_sample.cfg zoo.cfg
[root@node1 conf]# mkdir -p /export/server/zookeeper-3.4.6/zkdatas/
[root@node1 conf]# vim zoo.cfg
dataDir=/export/server/zookeeper-3.4.6/zkdatas/
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
server.1=node1:2888:3888
server.2=node2:2888:3888
server.3=node3:2888:3888
[root@node1 conf]# echo 1 > /export/server/zookeeper-3.4.6/zkdatas/myid
[root@node1 conf]# scp -r /export/server/zookeeper-3.4.6/ node2:/export/server/
[root@node1 conf]# scp -r /export/server/zookeeper-3.4.6/ node3:/export/server/
[root@node2 ~]# echo 2 > /export/server/zookeeper-3.4.6/zkdatas/myid
[root@node3 ~]# echo 3 > /export/server/zookeeper-3.4.6/zkdatas/myid
[root@node1 conf]# vim /etc/profile
export ZOOKEEPER_HOME=/export/server/zookeeper-3.4.6
export PATH=:$ZOOKEEPER_HOME/bin:$PATH
[root@node1 conf]# source /etc/profile
[root@node1 export]# cd onekey/
[root@node1 onekey]# vim zk.sh
for num in 1 2 3
do
        ssh root@node${num} "source /etc/profile;zkServer.sh $1"
        echo "正在操作第${num}台主机"
done
[root@node1 onekey]# chmod +x zk.sh
~~~



