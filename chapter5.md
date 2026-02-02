## MapReduce分布式计算—shuffle阶段(面试高频)

MapReduce思想在生活中处处可见，简单来说核心就是“先分再合，分而治之”。所谓“分而治之”就是把一个复杂的问题，按照一定的“分解”方法分为等价的规模较小的若干部分，然后逐个解决，分别找出各部分的结果，把各部分的结果组成整个问题的结果。

- **Map**负责==“分”==，即把复杂的任务分解为若干个“简单的任务”来并行处理。可以进行拆分的前提是这些小任务可以并行计算，彼此间几乎没有依赖关系。

- **Reduce**负责==“合”==，即对map阶段的结果进行全局汇总。

分布式计算是一种计算方法，和集中计算是相对的。

随着计算技术的发展，有些应用需要非常巨大的计算能力才能完成。分布式计算将任务分解成许多小的部分，分配给多态计算机进行处理。

> 定义(中国科学院)：在两个或多个软件互相共享信息，这些软件既可以在同一台计算机上运行，也可以在通过网络连接起来的多台计算机上运行。分布式计算比起其它算法具有以下几个优点：
>
> 1.稀有资源共享
>
> 2.通过分布式计算可以在多台计算机上平衡计算负载。
>
> 3.可以把程序放在最适合它的计算机上

MapReduce借鉴了函数式语言中的思想，用Map和Reduce两个函数提供了高层的并行编程抽象模型。

Map: 对一组数据元素进行某种重复式的处理；==(k1,v1)->[(k2,v2)]==

Reduce: 对Map的中间结果进行某种进一步的结果整理。==(k2,[v2])->[(k3,v3)]==

Map和Reduce为程序员提供了一个清晰的操作接口抽象描述。通过以上两个编程接口，大家可以看出MapReduce处理的数据类型是**<key,value>键值对**。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/mrpro.png" style="zoom:35%;" />

## MapReduce架构体系

一个完整的MapReduce程序在分布式运行时有三类实例进程：

1. MRAppMaster：负责整个程序的过程调度及状态协调
2. MapTask：负责map阶段的整个数据处理流程
3. ReduceTask：负责reduce阶段的整个数据处理流程

<img src="/Users/mungeryang/Desktop/Bigdata/pic/mr架构.png" style="zoom:45%;" />

用户编写的程序分成三个部分：**Mapper**，**Reducer**，**Driver**。用户自定义的Mapper和Reducer都要继承各自的父类。Mapper中的业务逻辑写在map()方法中，Reducer的业务逻辑写在reduce()方法中。整个程序需要一个Driver来进行提交，提交的是一个描述了各种必要信息的job对象。整个MapReduce程序中，数据都是以k-v键值对的形式流转的。因此在实际编程解决各种业务问题中，需要考虑每个阶段的输入输出k-v分别是什么。并且在MapReduce中数据会因为某些默认的机制进行排序进行分组。所以说k-v的类型数据确定及其重要。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/map.png" style="zoom:43%;" />

整个Map阶段流程大体如上图所示。简单概述：input File通过split被`逻辑切分`为多个split文件，通过Record按行读取内容给map（用户自己实现的）进行处理，数据被map处理结束之后交给OutputCollector收集器，对其结果key进行分区（默认使用hash分区），然后写入buffer，每个map task都有一个内存缓冲区，存储着map的输出结果，当缓冲区快满的时候需要将缓冲区的数据以一个临时文件的方式存放到磁盘，当整个map task结束后再对磁盘中这个map task产生的所有临时文件做合并，生成最终的正式输出文件，然后等待reduce task来拉数据。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/reduce.png" style="zoom:43%;" />

Reduce大致分为==copy、sort、reduce==三个阶段，重点在前两个阶段。copy阶段包含一个eventFetcher来获取已完成的map列表，由Fetcher线程去copy数据，在此过程中会启动两个merge线程，分别为inMemoryMerger和onDiskMerger，分别将内存中的数据merge到磁盘和将磁盘中的数据进行merge。待数据copy完成之后，copy阶段就完成了，开始进行sort阶段，sort阶段主要是执行finalMerge操作，纯粹的sort阶段，完成之后就是reduce阶段，调用用户定义的reduce函数进行处理。

## MapReduce工作执行流程

整个MapReduce工作流程可以分为3个阶段：map、shuffle、reduce。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/shuffle.png" style="zoom:55%;" />

**map阶段:**

负责把从数据源读取来到数据进行处理，默认情况下读取数据返回的是k-v键值对类型，经过自定义map方法处理之后，输出的也应该是k-v键值对类型。

maptask并行度个数机制：逻辑切片机制

影响maptask个数的因素：

- 文件的个数
- 文件的大小
- split size(block size)大小

**shuffle阶段:**

一般**把从Map产生输出开始到Reduce取得数据作为输入之前的过程**称作shuffle。

map输出的数据会经过``分区、排序、分组``等自带动作进行重组，相当于洗牌的逆过程。==这是MapReduce的核心所在，也是难点所在。也是值得我们深入探究的所在。==

默认分区规则：key相同的分在同一个分区，同一个分区被同一个reduce处理。    

默认排序规则：根据key字典序排序

默认分组规则：key相同的分为一组，一组调用reduce处理一次。

默认情况下，MapReduce程序不会涉及到partition分区。但是如果用户需要，可以设置reduce阶段有多个task执行。

==当reducetask>=2时，map阶段的task就需要针对自己的输出数据进行分区，所谓的分区就是根据什么规则交给哪一个reducetask来处理。==

**reduce阶段：**

负责针对shuffle好的数据进行聚合处理。输出的结果也应该是k-v键值对。

设置reducetask个数(输出结果文件个数为n)：

~~~java
//Driver类中设置reducetask个数-处理得到的文件个数
job.setNumReduceTasks(n);
~~~

maptask输出的结果<key，value>

key.hashcode % reducetask个数 = 余数(分区编号)

只要输出的key一样，数据就会到同一个分区。

底层是reducetask主动拉取数据而不是maptask主动分配。



## hadoop数据类型与序列化

Hadoop提供了如下内容的数据类型，这些数据类型都实现了==WritableComparable==接口，以便用这些类型定义的数据可以被序列化进行网络传输和文件存储，以及进行大小比较。

| **Hadoop**  **数据类型** | **Java数据类型** | **备注**                              |
| ------------------------ | ---------------- | ------------------------------------- |
| BooleanWritable          | boolean          | 标准布尔型数值                        |
| ByteWritable             | byte             | 单字节数值                            |
| IntWritable              | int              | 整型数                                |
| FloatWritable            | float            | 浮点数                                |
| LongWritable             | long             | 长整型数                              |
| DoubleWritable           | double           | 双字节数值                            |
| Text                     | String           | 使用UTF8格式存储的文本                |
| MapWritable              | map              | 映射                                  |
| ArrayWritable            | array            | 数组                                  |
| NullWritable             | null             | 当<key,value>中的key或value为空时使用 |

注意：如果需要将自定义的类放在key中传输，则还需要实现Comparable接口，因为MapReduce框中的Shuffle过程要求对key必须能排序。

序列化 (Serialization)是将结构化对象转换成字节流以便于进行网络传输或写入持久存储的过程。

反序列化（Deserialization）是将字节流转换为一系列结构化对象的过程，重新创建该对象。





## MapReduce

map阶段：

遍历大量数据记录

从记录中提取

reduce阶段：

对中间结果进行重新组合和排序

汇总结果

map: (k1,v1) —> list[<k2,v2>]

shuffle: list[<k2,v2>] —> (k2,list[v2])

reduce: (k2,list[v2]) —>  [<k3,v3>]



> 在整个MapReduce程序的开发过程中，我们最大的工作量是`Override map函数`和`Override reduce函数`。
>
> 图解出MapReduce过程中map阶段、shuffle阶段和reduce阶段==数据结构==的变化过程，根据分析过程编写合适的代码。

### hadoop开发需要的maven依赖

==org.apache.hadoop==中的`hadoop-common`、`hadoop-hdfs`、`hadoop-client`、`hadoop-mapreduce-client-core`

~~~xml
 <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.32</version>
        </dependency>
    </dependencies>
~~~

### 单词统计案例

map函数-WordCountMapper

~~~java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class WordCountMapper extends Mapper<Longwritable,Text,Text,LongWritable>{
    
    private final static LongWritable valueout = new LongWritable(1);
    
    @Override
    protected void map(LongWritable key,Text value,Content content) throws IOException, InterruptedException{
        //将文本数据以","分割并存入数组中
        String[] words = value.toString().split(",");
        for(word : words){
            context.write(new Text(word),valueout);
        }
        
    }
}
~~~

reduce函数-WordCountReduceor

~~~java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordCountReduceor extends Reducer<Text,LongWritable,Text,LongWritable>{
     @Override
    protected void reduce(Text key,Iterable<LongWritable> values,Context context) throws IOException, InterruptedException {
        long count = 0;
        for (LongWritable value : values) {
            count += value.get();//get方法将LongWritable转换成long
        }
        context.write(key,new LongWritable(count));
    }
}
~~~

Driver函数-WordCountDriver

~~~java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

public class WordCountDriver {

    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException, URISyntaxException {
        //配置文件对象
        Configuration configuration = new Configuration();
        //创建作业实例
        Job job = Job.getInstance(configuration, "wordcount");
        //设置作业驱动类
        job.setJarByClass(WordCountDriver.class);
        //设置作业mapper reducer类
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReduceor.class);
        //设置作业mapper阶段输出key(k2) value(v2)数据类型 也就是程序最终的输出数据类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);
        //设置作业reducer阶段输出key(k3) value(v3)数据类型 也就是程序最终的输出数据类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);
        //配置作业的输入数据路径
        //FileInputFormat.addInputPath(job,new Path("file:///D:\\input\\wordcount"));
        //FileInputFormat.addInputPath(job,new Path("hdfs://node1:8020/input/wordcount"));
        FileInputFormat.addInputPath(job,new Path(args[0]));
        //配置作业的输出数据路径
        //Path outpath = new Path("file:///D:\\output\\wordcount");
        //Path outpath = new Path("hdfs://node2:8020/output/wordcount");
        Path outpath = new Path(args[1]);
        FileOutputFormat.setOutputPath(job,outpath);

        //job.setNumReduceTasks(n);
        //判断路径是否存在 如果存在删除路径
        //FileSystem fs = FileSystem.get(configuration);
        FileSystem fs = FileSystem.get(new URI("hdfs://node1:8020"),configuration);
        boolean bl = fs.exists(outpath);
        if(bl){
            fs.delete(outpath,true);
        }
        boolean res = job.waitForCompletion(true);

        //提交作业等待执行完成
        System.exit(res ? 0 : 1);

    }
}
~~~

### 分类汇总统计案例

自定义java bean类

~~~java
public class CovidCountBean implements Writable{

    private long cases;//确诊病例数
    private long deaths;//死亡病例数

    public CovidCountBean() {
    }

    public CovidCountBean(long cases, long deaths) {
        this.cases = cases;
        this.deaths = deaths;
    }

    public void set(long cases, long deaths) {
        this.cases = cases;
        this.deaths = deaths;
    }

    public long getCases() {
        return cases;
    }

    public void setCases(long cases) {
        this.cases = cases;
    }

    public long getDeaths() {
        return deaths;
    }

    public void setDeaths(long deaths) {
        this.deaths = deaths;
    }

    /**
     *  序列化方法
     */
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(cases);
        out.writeLong(deaths);
    }

    /**
     * 反序列化方法 注意顺序
     */
    @Override
    public void readFields(DataInput in) throws IOException {
        this.cases = in.readLong();
        this.deaths =in.readLong();
    }

    @Override
    public String toString() {
        return  cases +"\t"+ deaths;
    }

}
~~~



## yarn架构

<img src="/Users/mungeryang/Desktop/Bigdata/pic/yarn.jpg" style="zoom:63%;" />

Apache Hadoop YARN是一种新的 Hadoop 资源管理器，它是一个`通用`==资源管理==系统和==调度平台==，可为上层应用提供统一的资源管理和调度，它的引入为集群在利用率、资源统一管理和数据共享等方面带来了巨大好处。

> 1.如何理解通用资源管理系统？
>
> 资源管理是指内存和CPU。通用指的是yarn不关心程序是什么，理论上任何种类的程序比如MapReduce、spark、flink都可以申请资源。
>
> yarn只负责分配资源，使用完回收资源
>
> 2.如何理解调度平台？
>
> 所谓的调度指的是当集群资源繁忙时候，多个程序申请资源，涉及到的资源分配问题，也就是调度程序谁先谁后的问题。调度的关键是调度算法和策略。
>
> 优先级、先来先服务、权重
>
> 

可以把yarn理解为相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序，Yarn为这些程序提供运算所需的资源(内存、cpu)。

### yarn的三大组件-yarn执行流程(面试高频)

从集群的角度有两个：ResourcesManager、NodeManager

从程序的角度有1个：ApplicationManager

ResourceManager负责所有资源的监控、分配和管理；NodeManager以`心跳的方式`向ResourceManager汇报资源使用情况（目前主要是CPU和内存的使用情况）。ResourceManager只接受NodeManager的资源回报信息，对于具体的资源处理则交给NodeManager自己处理。

ApplicationMaster负责每一个具体应用程序的调度和协调；

NodeManager负责每一个节点的维护; NodeManager定时向ResourceManager汇报本节点资源（CPU、内存）的使用情况和Container的运行状态。当ResourceManager宕机时NodeManager自动连接ResourceManager备用节点。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/mr架构1.jpg" style="zoom:45%;" />

### Yarn的调度器Scheduler

理想情况下，我们应用对Yarn资源的请求应该立刻得到满足，但现实情况资源往往是有限的，特别是在一个很繁忙的集群，一个应用资源的请求经常需要等待一段时间才能的到相应的资源。在**Yarn中，负责给应用分配资源的就是Scheduler**。

在Yarn中有三种调度器可以选择：FIFO Scheduler ，Capacity Scheduler，Fair Scheduler。



### FIFO Scheduler

**FIFO** Scheduler把应用按提交的顺序排成一个队列，这是一个**先进先出**队列，在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。

FIFO Scheduler是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。

大的应用可能会占用所有集群资源，这就导致其它应用被阻塞。在共享集群中，更适合采用Capacity Scheduler或Fair Scheduler，这两个调度器都允许大任务和小任务在提交的同时获得一定的系统资源。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/FIFO Scheduler.jpg" style="zoom:55%;" />

### Capacity Scheduler

Capacity Scheduler调度器**以队列为单位划分资源**。简单通俗点来说，就是**一个个队列有独立的资源，队列的结构和资源是可以进行配置的**。

CapacityScheduler的配置项包括两部分，其中一部分在yarn-site.xml中，主要用于配置YARN集群使用的调度器；另一部分在capacity-scheduler.xml配置文件中，主要用于配置各个队列的资源量、权重等信息。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/Capacity Scheduler.png" style="zoom:50%;" />

### Fair Scheduler

在Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。

需要注意的是，在下图Fair调度器中，从第二个任务提交到获得资源会有一定的延迟，因为它需要等待第一个任务释放占用的Container。小任务执行完成之后也会释放自己占用的资源，大任务又获得了全部的系统资源。最终效果就是Fair调度器即得到了高的资源利用率又能保证小任务及时完成。

<img src="/Users/mungeryang/Desktop/Bigdata/pic/Fair Scheduler.jpg" style="zoom:53%;" />



