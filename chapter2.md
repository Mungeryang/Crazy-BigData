## JDBC



### 概念

JDBC(Java database connectivity, java 数据库连接)，是一种执行SQL语句的Java API，是java访问数据库的一种标准规范，可以为不同的关系型数据库提供统一的访问，由一组用java语言编写的类和接口组成。 

JDBC需要连接驱动，驱动是两个设备之间进行通信，满足的一定的通信格式，数据格式有设备提供商提供，设备提供商为设备提供软件，通过软件可以与该设备进行通信。

今天我们使用的mysql驱动就是mysql-connector-java-5.1.37-bin-jar



### Java的安装配置



### JDBC-操作数据库朴素方式

`Ctrl+Alt+t`，代入代码块.

JDBC连接mysql数据库的操作流程

1. 注册驱动
2. 获得连接
3. 获取SQL
4. 执行SQL
5. 打印结果
6. 释放资源

```xml
pom.xml注册驱动
<?xml version="1.0" encoding="UTF-8"?>
<dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version>
        </dependency>
        <dependency>
            <groupId>org.dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>2.1.3</version>
        </dependency>
    </dependencies>
```

注册驱动

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```



获得连接

```java
String url = "jdbc:mysql://localhost:3306/mysql?useSSL=false";
Connection conn = DriverManager.getConnection(url, "root", "yanggm123");
```



获取SQL

```java
Statement stmt = conn.createStatement();
```



执行SQL

```java
ResultSet resultSet = stmt.executeQuery("select * from employee");
```



打印结果

```java
ResultSetMetaData metadata = resultSet.getMetaData();
int cols = metaData.getColumnCount();//获取数据表全部列的数量
while(resultSet.next()){
    for(int i = 1;i <= cols;i++){
        Object object = resultSet.getObject(i);
        System.out.print(object+"\t");
    }
    System.out.println();
}
```



释放资源

```java
conn.close();
stmt.close();
```

### 编写工具类操作数据库

为实现低耦合与软编码，我们不把参数写死(DriverClass, url, username, password),而是通过读取xml文件来自动提取相关信息，实现不同的业务需求。

xml配置文档

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<jdbc_config>
    <mysql_config>
        <DriverClass>com.mysql.cj.jdbc.Driver</DriverClass>
        <Url>jdbc:mysql://localhost:3306/mysql?useSSL=false</Url>
        <Username>root</Username>
        <Password>yanggm123</Password>
        <Sql>select * from employee</Sql>
    </mysql_config>
</jdbc_config>
```

工具类编写

```java
public class JDBCUtils{
    
    private JDBCUtils(){};
    
    private static String DriverClass;
    private static String Url;
    private static String Username;
    private static String Password;
    private static String Sql;
    
    //静态代码块-封装静态驱动并解析参数
    static{
        //创建Reader对象
        SAXReader reader = new SAXReader();
        //加载xml
        InputStream inputStream =
                JDBCUtils.class.getClassLoader().getResourceAsStream("jdbc.xml");
        Document document;

        {
            try {
                document = reader.read(inputStream);
                //获取根节点
                Element rootElement = document.getRootElement();
                //使通过根节点获取根节点的子元素
                Iterator<Element> iterator = rootElement.elementIterator();
                //遍历集合
                while (iterator.hasNext()){
                    //获取子元素
                    Element info = iterator.next();
                    Iterator<Element> iterator1 = info.elementIterator();
                    while(iterator1.hasNext()){
                        //获取子元素
                        Element infochild = iterator1.next();
                        String name = infochild.getName();
                        if("DriverClass".equals(name)){
                            DriverClass = infochild.getStringValue();
                        } else if ("Url".equals(name)) {
                            Url = infochild.getStringValue();
                        } else if ("Username".equals(name)) {
                            Username = infochild.getStringValue();
                        } else if ("Password".equals(name)) {
                            Password = infochild.getStringValue();
                        } else if ("Sql".equals(name)) {
                            Sql = infochild.getStringValue();
                        }

                    }
                }
                Class.forName(DriverClass);

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
    //2.封装获得连接
    public static Connection getConnection(){
        try {
            Connection conn = DriverManager.getConnection(Url,Username,Password);
            return conn;
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }

    //3.封装获得执行SQL语句的对象
    //4.封装执行SQL语句并返回结果

    public static ResultSet resultSet(Statement stmt,Connection conn){
        try {
            ResultSet resultSet = stmt.executeQuery(Sql);
            return resultSet;
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;

    }

    public static void printTable(ResultSet resultSet){

        try {
            //获取resultSet元数据得到列数
            ResultSetMetaData metaData = resultSet.getMetaData();
            int cols = metaData.getColumnCount();
            //打印SQL-table
            while(resultSet.next()){
                for (int i = 1; i <= cols; i++) {
                    Object object = resultSet.getObject(i);
                    System.out.print(object+"\t");
                }
                System.out.println();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static void closeAll(Statement stmt,Connection conn){
        try {
            stmt.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
        
```

改写main方法

```java
public class TestDemo04 {

    public static void main(String[] args) throws Exception {

        //2.获得连接
        Connection conn = JDBCPackage.getConnection();
        Statement stmt = conn.createStatement();

        //3.获得执行SQL语句的对象
        //4.执行SQL语句并返回结果
        ResultSet resultSet = JDBCPackage.resultSet(stmt, conn);

        //5.展示处理结果
        JDBCPackage.printTable(resultSet);

        //6.释放资源
        JDBCPackage.closeAll(stmt,conn);

    }
}
```

### 报错汇总

#### 时区错误

> The server time zone value ‘ÖÐ¹ú±ê×¼Ê±¼ä‘ is unrecognized or represents more than one time zone

服务器时区值“ÖÐ¹ú±ê×¼Ê±¼ä”无法识别或表示多个时区。如果希望利用时区支持，必须配置服务器或JDBC驱动程序(通过serverTimezone配置属性)以使用更具体的时区值。

#### 解决办法1-修改配置文件(my.ini)设置时区

- 打开并添加默认时区参数



#### 解决办法2-终端修改session时区

**set global time_zone = ‘+8:00’**



#### 解决办法3-在JDBC连接的URL后加上时区参数

```java
jdbc:mysql://localhost:3306/demo?serverTimezone=UTC
```

> MESSAGE: closing inbound before receiving peer's close_notify

在配置文件中数据库连接的**url属性**中加入`useSSL=false`即可解决

```java
String url = "jdbc:mysql://localhost:3306/mysql";
String url = "jdbc:mysql://localhost:3306/mysql?useSSL=false";
```

#### enabledtlsprotocols

连接本地虚拟机上mysql报错

服务器已终止握手。设置了协议列表选项(`enabledtlsprotocols`),此选项可能会导致某些版本的 mysql 出现连接问题。

~~~mysql
# url后添加参数?createDatabaseIfNotExist=true&useSSL=false即可解决
jdbc:mysql://192.168.52.150:3306?createDatabaseIfNotExist=true&useSSL=false
~~~







## SQL 必知必会

在SQL中，单引号用于表示字符串。在查询时，将字符串值放在单引号之间，以便数据库引擎正确地识别它们。

### 基本概念

**事务**(Transaction):是用户定义的一个数据库操作序列，这些操作要么全做，要么不做，是一个不可分割的整体工作单位。

事务的ACID四大特性：原子性(Atomicity)、一致性(Consistency)、隔离性(Isolation)、持续性(Durability)。

定义事务一般有三条语句：

BEGIN TRANSACTION-事务以此开始 begin

COMMIT-提交事务的所有操作，对数据库进行更新写回到磁盘上物理数据库中 commit

ROLLBACK-对数据库中已完成的操作全部撤销回滚到事务开始执行的状态 rollback

事务操作只对增删改有效，对查询无效



### MySQL安装配置





### 基本查询与函数

:heart:自学网站：SQL之母-http://sqlmother.yupi.icu/ by 程序员鱼皮

> 学习技术当然也要与时俱进啦，所以在这一小节中，我们首先先把SQL的基础查询语句搞定，然后我们再使用当下流行的Python第三方库==Pandas==复现一遍。

#### 模糊查询

模糊查询是一种特殊的条件查询，它允许我们根据模式匹配来查找符合特定条件的数据，可以使用 LIKE 关键字实现模糊查询。

在 LIKE 模糊查询中，我们使用通配符来代表零个或多个字符，从而能够快速地找到匹配的数据。

有如下 2 种通配符：

- 百分号（%）：表示任意长度的任意字符序列。
- 下划线（_）：表示任意单个字符。

~~~sql
# 请编写一条 SQL 查询语句，从名为 student 的数据表中选择出所有学生的姓名（name）和成绩（score），要求姓名（name）不包含 "杨" 这个字。
select name,score from student where name not like '%杨%';
~~~

#### 逻辑运算

逻辑运算是一种在条件查询中使用的运算符，它允许我们结合多个条件来过滤出符合特定条件的数据。

在逻辑运算中，常用的运算符有：

- AND：表示逻辑与，要求同时满足多个条件，才返回 true。
- OR：表示逻辑或，要求满足其中任意一个条件，就返回 true。
- NOT：表示逻辑非，用于否定一个条件（本来是 true，用了 not 后转为 false）

~~~mysql
# 请编写一条 SQL 查询语句，从名为 student 的数据表中选择出所有学生的姓名（name）、成绩（score），要求学生的姓名包含 "李"，或者成绩（score）大于 500。
select name,score from student where sore > 500 and name like "%李%"；

~~~



#### 去重

在数据表中，可能存在重复的数据记录，但如果我们想要过滤掉重复的记录，只保留不同的记录，就要使用 SQL 的去重功能。

在 SQL 中，我们可以使用 `DISTINCT` 关键字来实现去重操作。

除了按照单字段去重外，`DISTINCT` 关键字还支持根据多个字段的组合来进行去重操作，确保多个字段的组合是唯一的。

举个应用场景：假设你是班长，要统计班级中有哪些不同的学生，而不关心他们重复出现的次数，就可以使用去重。

~~~mysql
# 请编写一条 SQL 查询语句，从名为 student 的数据表中选择出所有不重复的班级 ID（class_id）和考试编号（exam_num）的组合。
select distinct class_id,exam_num from student;
~~~

#### 排序

在查询数据时，我们有时希望对结果按照某个字段的值进行排序，以便更好地查看数据。

在 SQL 中，我们可以使用 `ORDER BY` 关键字来实现排序操作。`ORDER BY` 后面跟上需要排序的字段，可以选择升序（ASC）或降序（DESC）排列。

在排序的基础上，我们还可以根据多个字段的值进行排序。当第一个字段的值相同时，再按照第二个字段的值进行排序，以此类推。

~~~mysql
order by 字段1 [升序/降序], 字段2 [升序/降序], ...
# 请编写一条 SQL 查询语句，从名为 student 的数据表中选择出学生姓名（name）、年龄（age）和成绩（score），首先按照成绩从大到小排序，如果成绩相同，则按照年龄从小到大排序。
select name,age,score from student order by score desc,age asc;
~~~



#### 截断和偏移

在 SQL 中，我们使用 `LIMIT` 关键字来实现数据的截断和偏移。

截断和偏移的一个典型的应用场景是分页，即网站内容很多时，用户可以根据页号每次只看部分数据。

```sql
-- LIMIT 后只跟一个整数，表示要截断的数据条数（一次获取几条）
select task_name, due_date from tasks limit 2;

-- LIMIT 后跟 2 个整数，依次表示从第几条数据开始、一次获取几条
select task_name, due_date from tasks limit 2, 2;

-- 请编写一条 SQL 查询语句，从名为 student 的数据表中选择学生姓名（name）和年龄（age），按照年龄从小到大排序，从第 2 条数据开始、截取 3 个学生的信息。
select name,age from student order by age asc limit 1,3;
```



#### 条件查询

条件分支 `case when` 是 SQL 中用于根据条件进行分支处理的语法。它类似于其他编程语言中的 if else 条件判断语句，允许我们根据不同的条件选择不同的结果返回。

使用 `case when` 可以在查询结果中根据特定的条件动态生成新的列或对现有的列进行转换。

```sql
CASE WHEN (条件1) THEN 结果1
	   WHEN (条件2) THEN 结果2
	   ...
	   ELSE 其他结果 END 
	   AS ..
-- 假设有一个学生表 student，包含以下字段：name（姓名）、age（年龄）。请你编写一个 SQL 查询，将学生按照年龄划分为三个年龄等级（age_level）：60 岁以上为 "老同学"，20 岁以上（不包括 60 岁以上）为 "年轻"，20 岁及以下、以及没有年龄信息为 "小同学"。返回结果应包含学生的姓名（name）和年龄等级（age_level），并按姓名升序排序。	   
select 
    name,
    case 
        when (age > 60) then '老同学'
        when (age > 20) then '年轻'
        else '小同学'
    end as age_level
from 
    student
order by 
    name asc;
```

#### 日期处理

在 SQL 中，时间函数是用于处理日期和时间的特殊函数。它们允许我们在查询中操作和处理日期、时间、日期时间数据，从而使得在数据库中进行时间相关的操作变得更加方便和灵活。

常用的时间函数有：

- `DATE`：获取当前日期
- `DATETIME`：获取当前日期时间
- `TIME`：获取当前时间

```sql
-- 获取当前日期
SELECT DATE() AS current_date;

-- 获取当前日期时间
SELECT DATETIME() AS current_datetime;

-- 获取当前时间
SELECT TIME() AS current_time;

-- 查询并展示展示当前日期
select
  name,
  date () as 当前日期
from
  student
```

#### 字符串处理

在 SQL 中，字符串处理是一类用于处理文本数据的函数。它们允许我们对字符串进行各种操作，如转换大小写、计算字符串长度以及搜索和替换子字符串等。字符串处理函数可以帮助我们在数据库中对字符串进行加工和转换，从而满足不同的需求。

```sql
-- 将姓名转换为大写
SELECT name, UPPER(name) AS upper_name
FROM employees;

-- 计算姓名长度
SELECT name, LENGTH(name) AS name_length
FROM employees;

-- 将姓名转换为小写并进行条件筛选
SELECT name, LOWER(name) AS lower_name
FROM employees;

select 
	id,name,upper(name) as upper_name 
from 
	student 
where 
	name = '热dog';
```

#### 聚合查询

在 SQL 中，聚合函数是一类用于对数据集进行 **汇总计算** 的特殊函数。它们可以对一组数据执行诸如计数、求和、平均值、最大值和最小值等操作。聚合函数通常在 SELECT 语句中配合 `GROUP BY` 子句使用，用于对分组后的数据进行汇总分析。

常见的聚合函数包括：

- COUNT：计算指定列的行数或非空值的数量。
- SUM：计算指定列的数值之和。
- AVG：计算指定列的数值平均值。
- MAX：找出指定列的最大值。
- MIN：找出指定列的最小值。

1）使用聚合函数 `COUNT` 计算订单表中的总订单数：

```sql
SELECT COUNT(*) AS order_num
FROM orders;
```

2）使用聚合函数 `COUNT(DISTINCT 列名)` 计算订单表中不同客户的数量：

```sql
SELECT COUNT(DISTINCT customer_id) AS customer_num
FROM orders;
```

3）使用聚合函数 `SUM` 计算总订单金额：

```sql
SELECT SUM(amount) AS total_amount
FROM orders;
```



在 SQL 中，分组聚合是一种对数据进行分类并对每个分类进行聚合计算的操作。它允许我们按照指定的列或字段对数据进行分组，然后对每个分组应用聚合函数，如 COUNT、SUM、AVG 等，以获得分组后的汇总结果。

举个例子：某个学校可以按照班级将学生分组，并对每个班级进行统计。查看每个班级有多少学生、每个班级的平均成绩。这样我们就能够对学校各班的学生情况有一个整体的了解，而不是单纯看个别学生的信息。

在 SQL 中，通常使用 `GROUP BY` 关键字对数据进行分组。

1）使用分组聚合查询中每个客户的编号：

```sql
SELECT customer_id
FROM orders
GROUP BY customer_id;
```

2）使用分组聚合查询每个客户的下单数：

```sql
SELECT customer_id, COUNT(order_id) AS order_num
FROM orders
GROUP BY customer_id;
```

有时，单字段分组并不能满足我们的需求，比如想统计学校里每个班级每次考试的学生情况，这时就可以使用多字段分组。

多字段分组和单字段分组的实现方式几乎一致，使用 `GROUP BY` 语法即可。



在 SQL 中，`HAVING` 子句用于在分组聚合后对分组进行过滤。它允许我们对分组后的结果进行条件筛选，只保留满足特定条件的分组。

HAVING 子句与条件查询 WHERE 子句的区别在于，WHERE 子句用于在 **分组之前** 进行过滤，而 HAVING 子句用于在 **分组之后** 进行过滤。

说白了，having用在group by之后，where用在group by之前。where是对整个表进行选择过滤，而having是对group by分组之后的数据进行过滤。

1）使用 HAVING 子句查询订单数超过 1 的客户：

```sql
SELECT customer_id, COUNT(order_id) AS order_num
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) > 1;
```



#### 关联查询

在之前的教程中，我们所有的查询操作都是在单个数据表中进行的。但有时，我们可能希望在单张表的基础上，获取更多额外数据，比如获取学生表中学生所属的班级信息等。这时，就需要使用关联查询。

在 SQL 中，关联查询是一种用于联合多个数据表中的数据的查询方式。

其中，`CROSS JOIN` 是一种简单的关联查询，不需要任何条件来匹配行，它直接将左表的 **每一行** 与右表的 **每一行** 进行组合，返回的结果是两个表的笛卡尔积。

```sql
SELECT e.emp_name, e.salary, e.department, d.manager
FROM employees e
CROSS JOIN departments d;
```

在 SQL 中，`INNER JOIN `是一种常见的关联查询方式，它根据两个表之间的关联条件，将满足条件的行组合在一起。

注意，INNER JOIN 只返回两个表中满足关联条件的==交集部分==，即在两个表中都存在的匹配行。

```sql
SELECT e.emp_name, e.salary, e.department, d.manager
FROM employees e
JOIN departments d ON e.department = d.department;
```

在 SQL 中，`OUTER JOIN` 是一种关联查询方式，它根据指定的关联条件，将两个表中满足条件的行组合在一起，并 **包含没有匹配的行** 。

在 OUTER JOIN 中，包括 LEFT OUTER JOIN 和 RIGHT OUTER JOIN 两种类型，它们分别表示查询左表和右表的所有行（即使没有被匹配），再加上满足条件的交集部分。

```sql
SELECT e.emp_name, e.salary, e.department, d.manager
FROM employees e
LEFT JOIN departments d ON e.department = d.department;
```



#### 子查询

子查询是指在一个查询语句内部 **嵌套** 另一个完整的查询语句，内层查询被称为子查询。子查询可以用于获取更复杂的查询结果或者用于过滤数据。

当执行包含子查询的查询语句时，数据库引擎会==首先执行子查询==，然后将其结果作为条件或数据源来执行外层查询。

```sql
-- 主查询
SELECT name, total_amount
FROM customers
WHERE customer_id IN (
    -- 子查询
    SELECT DISTINCT customer_id
    FROM orders
    WHERE total_amount > 200
);
```

其中，子查询中的一种特殊类型是 "exists" 子查询，用于检查主查询的结果集是否存在满足条件的记录，它返回布尔值（True 或 False），而不返回实际的数据。

```sql
-- 主查询
SELECT name, total_amount
FROM customers
WHERE EXISTS (
    -- 子查询
    SELECT 1
    FROM orders
    WHERE orders.customer_id = customers.customer_id
);
```

#### 组合查询

在 SQL 中，组合查询是一种将多个 SELECT 查询结果合并在一起的查询操作。

包括两种常见的组合查询操作：UNION 和 UNION ALL。

1. UNION 操作：它用于将两个或多个查询的结果集合并， **并去除重复的行** 。即如果两个查询的结果有相同的行，则只保留一行。
2. UNION ALL 操作：它也用于将两个或多个查询的结果集合并， **但不去除重复的行** 。即如果两个查询的结果有相同的行，则全部保留。

```sql
-- UNION 操作
SELECT name, age, department
FROM table1
UNION
SELECT name, age, department
FROM table2;

-- UNION ALL操作
SELECT name, age, department
FROM table1
UNION ALL
SELECT name, age, department
FROM table2;
```

### 开窗函数

在 SQL 中，开窗函数是一种强大的查询工具，它允许我们在查询中进行对分组数据进行计算、 **同时保留原始行的详细信息** 。

开窗函数可以与聚合函数（如 SUM、AVG、COUNT 等）结合使用，但与普通聚合函数不同，开窗函数不会导致结果集的行数减少。

打个比方，可以将开窗函数想象成一种 "透视镜"，它能够将我们聚焦在某个特定的分组，同时还能看到整体的全景。

```sql
SUM(计算字段名) OVER (PARTITION BY 分组字段名)
-- sum over order by，可以实现同组内数据的 累加求和 。
SUM(计算字段名) OVER (PARTITION BY 分组字段名 ORDER BY 排序字段 排序规则)
-- Rank 开窗函数是 SQL 中一种用于对查询结果集中的行进行 排名 的开窗函数
RANK() OVER (
  PARTITION BY 列名1, 列名2, ... -- 可选，用于指定分组列
  ORDER BY 列名3 [ASC|DESC], 列名4 [ASC|DESC], ... -- 用于指定排序列及排序方式
) AS rank_column

-- Row_Number 开窗函数是 SQL 中的一种用于为查询结果集中的每一行 分配唯一连续排名 的开窗函数。
-- 它与之前讲到的 Rank 函数，Row_Number 函数为每一行都分配一个唯一的整数值，不管是否存在并列（相同排序值）的情况。每一行都有一个唯一的行号，从 1 开始连续递增。
ROW_NUMBER() OVER (
  PARTITION BY column1, column2, ... -- 可选，用于指定分组列
  ORDER BY column3 [ASC|DESC], column4 [ASC|DESC], ... -- 用于指定排序列及排序方式
) AS row_number_column

-- 开窗函数 Lag 和 Lead 的作用是获取在当前行之前或之后的行的值，这两个函数通常在需要比较相邻行数据或进行时间序列分析时非常有用。
LAG(column_name, offset, default_value) OVER (PARTITION BY partition_column ORDER BY sort_column)
LEAD(column_name, offset, default_value) OVER (PARTITION BY partition_column ORDER BY sort_column)

```

lead：

- `column_name`：要获取值的列名。
- `offset`：表示要向下偏移的行数。例如，offset为1表示获取下一行的值，offset为2表示获取下两行的值，以此类推。
- `default_value`：可选参数，用于指定当没有后一行时的默认值。
- `PARTITION BY`和`ORDER BY`子句可选，用于分组和排序数据。

lag：

- `column_name`：要获取值的列名。
- `offset`：表示要向上偏移的行数。例如，offset为1表示获取上一行的值，offset为2表示获取上两行的值，以此类推。
- `default_value`：可选参数，用于指定当没有前一行时的默认值。
- `PARTITION BY`和`ORDER BY`子句可选，用于分组和排序数据。2）Lead 函数




