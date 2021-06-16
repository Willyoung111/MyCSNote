[TOC]



# 数据库基本概念

**数据库（Database）指长期存储在计算机内的、有组织的、可共享的数据集合**

数据的重要性远远大于静态的程序

**数据库管理系统** **（****DBMS Database Manager System****）**是数据库系统的核心软件之一，在用户与操作系统之间的管理软件。我们常说的MYSQL，其实是MYSQL数据库管理系统。

## 关系型数据库 Relation Database Management System

建立在关系模型基础上的数据库，借助集合代数等数学概念和方法来处理数据库中的数据。

关系型数据库是由**多张能互相连接的表**组成的数据库。

数据分类存放，数据之间也有互相关联关系

**优点：**

- 都是使用表结构，格式一致，易于维护。
- 使用通用的SQL语言操作，使用方便，可用于复杂查询
- 数据存储在磁盘中，安全

**缺点**

- 读写性能较差
- 不节省空间，为了遵循规则，一些字段为空时也要分配空间
- 固定的表结构，灵活度较低

 

常见的关系型数据库：Oracle、、PostgreSQL、Microsoft SQL Server、Microsoft Access 和 **MySQL** 

DB2：商业数据库，性能好，价格贵

Oracle：商业数据库，根据性能决定价格

MySQL：开源免费，性能一般，灵活，可二次开发

SQL Server：只支持Windows，图形界面优秀

 

## 非关系型数据库

**NoSQL（Not Only SQL）**，数据以对象的形式存储在数据库中，对象之间的关系通过每个对象自身的属性来决定，数据之间没有关联关系。

它与关系型数据库之间是互补关系而不是竞争关系。

常用于：**秒杀库存、新闻信息、热点信息**

**优点：**

- 非关系型数据库存储数据的格式更灵活，如key-value，文档，图片等各种形式，没有严格的表信息，而关系型数据库只支持基础类型
- 速度快、效率高。NoSQL可以用硬盘或随机存储器为载体，而关系型数据库只能使用硬盘
- 海量数据的维护和处理轻松
- 扩展简单、高并发、高稳定性、低成本
- 可实现数据分布式处理

**缺点**

- 不提供SQL支持，学习使用成本高
- 没有事务处理，没有保证数据完整性和安全性
- 功能不够完善

常见的非关系型数据库：Neo4j、MongoDB、**Redis**、Memcached、MemcacheDB 和 HBase

Redis：单线程数据库，内存保持数据，数据读写速度快

MongoDB：硬盘保存，常用于保存低价值数据



# MySQL

## 安装

## 用户管理

### 覆盖ROOT密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456'

net stop mysql80

mysqld --defaults-file="C:\ProgramData\MySQL\MySQL Server 8.0\my.ini" --init-file="temp.txt" --console

net start mysql80
```

## 配置文件参数

```ini
[client]
port=3306 #服务端口号

[mysql]
no-beep #出现错误不开启蜂鸣器

[mysqld] #数据库设置
#默认端口
port=3306 

#数据目录
datadir=C:/ProgramData/MySQL/MySQL Server 8.0\Data 

#密码认证插件
default_authentication_plugin=mysql_native_password 

#默认存储引擎（INNODB、MYISAM）
default-storage-engine=INNODB sql-

#严格模式，字段校验
mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION" 

# 用文件记录日志
log-output=FILE

#关闭日志输出
general-log=0

#日志文件名称
general_log_file="DESKTOP-TQRNGI3.log"

#开启慢查询日志
slow-query-log=1

#慢查询日志文件名称
slow_query_log_file="DESKTOP-TQRNGI3-slow.log"

#慢查询SQL阈值，单位秒
long_query_time=10

#错误日志名称
log-error="DESKTOP-TQRNGI3.err"

#数据库ID，一般用于数据库集群
server-id=1

#把表名转换成小写
lower_case_table_names=1

#导入导出数据的目录地址
secure-file-priv="C:/ProgramData/MySQL/MySQL Server 8.0/Uploads"

#最大连接数
max_connections=151

table_open_cache=2000
tmp_table_size=94M

#线程数量
thread_cache_size=10
```

## 存储引擎

### InnoDB

MySQl中第一个提供**外键约束**的存储引擎

MySQL从5.5版本开始将默认存储引擎设置为InnoDB

**InnoDB的优势**

- **支持事务安装**

四种隔离级别：READ UNCOMMITTED, READ COMMITED, REPEATABLE READ， SERIALIZABLE

- **灾难恢复性好**

InnoDB通过commit、rollback、crash-recovery来保障数据的安全。

- **使用行级锁**

InnoDB 改变了 MyISAM 的锁机制，实现了行锁。虽然 InnoDB 的行锁机制是通过索引来完成的，但毕竟在数据库中 99%的 SQL 语句都要使用索引来检索数据。行锁定机制也为 InnoDB 在承受高并发压力的环境下增强了不小的竞争力。

- **实现缓冲处理**

提供专门的缓存池，能缓冲索引和数据。对比MyISAM只是缓存了索引。

- **支持外键**

InnoDB 支持外键约束，检查外键、插入、更新和删除，以确保数据的完整性。在存储表中数据时每张表的存储都按主键顺序存放，如果没有显式地在定义表时指定主键，InnoDB 会为每一行生成一个 6 字节的 ROWID ，并以此作为主键。

- **适合需要大型数据库的网站**

InnoDB 被用在众多需要高性能的大型数据库网站上。

**物理存储方式**

使用 InnoDB 时，MySQL 会在数据目录（Data）下创建一个名为 **ibdata1** 的 10MB 大小的**自动扩展数据文件**，以及**两个名为 ib_logfile0 和 ib_logfile1** 的 5MB 大小的日志文件。

1. **数据文件（表数据和索引数据）**

存放数据**表中的数据**和所有的**索引数据**，包括主键和其他普通索引

InnoDB 存储的数据采用**表空间（Tablepace）**进行存放设计。表空间是用来存放 MySQL 系统相关信息的一个**特殊共享表空间**。

 

InnoDB表空间的两种形式：

- **共享表空间**：表数据和索引都放在同一个表空间。默认的表空间文件就是ibdata1。
- **独立表空间**：每个表的数据和索引被存放在一个单独的.ibd文件中

 

查看是否使用独立表空间的指令

```
mysql> SHOW VARIABLES LIKE 'innodb_file_per_table%';
+-----------------------+-------+
| Variable_name     | Value |
+-----------------------+-------+
| innodb_file_per_table | ON  |
+-----------------------+-------+
1 row in set, 1 warning (0.01 sec)
 
```

ON表示打开独立表空间。

 

2. **日志文件**

InnoDB存储引擎的数据目录下默认会有两个名为ib_logfile0和ib_logfile1的文件。被称为重做日志文件(redo log file)

InnoDB可以通过重做日志将数据库宕机时已经完成但还没有写入磁盘的事务回复，也可以将所有部门完成并已经写入磁盘的事务回滚，并且将数据还原，以此来保证数据的完整性。

###  MyISAM

**优点**：

- 占用空间小
- 访问速度块，对事务完整性没有要求，或者使用SELECT、INSERT为主的应用可以使用此引擎
- 配合锁实现操作系统下的备份
- 支持全文检索（InnoDB在5.6后也支持）
- 数据紧凑存储，有更小的索引和更快的全表扫描性能

**缺点**：

- 不支持事务的完整性和并发性
- 不支持行级锁，并发性差
- 主机宕机后，MyISAM表易损坏，灾难恢复行不佳
- 数据库崩溃后无法安全恢复
- 只缓存索引，数据的缓存利用操作系统缓冲区实现，可能引发过多的系统调用



## MySQL数据表基本操作

### 创建数据表

```mysql
CREATE TABLE <表名> （[表定义选项]）[表选项][分区选项];
```

[表定义选项]的格式

```
<列名1><类型1>[,…]<列名n><类型n>
```

```mysql
CREATE TABLE tb_emp1
(
id INT(11),
deptId INT(11),
salary FLOAT
);
```

### 修改数据表

通过ALTER TABLE语句可以改变原有的表结构，如增加、删减列、更改原有列类型、重新命名列或表等。

修改选项的语法格式如下：

```mysql
{ ADD COLUMN <列名> <类型>
| CHANGE COLUMN <旧列名> <新列名> <新列类型>
| ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT }
| MODIFY COLUMN <列名> <类型>
| DROP COLUMN <列名>
| RENAME TO <新表名>
| CHARACTER SET <字符集名>
| COLLATE <校对规则名> }
```

**修改表名**

```mysql
ALTER TABLE <表名> [修改选项]
ALTER TABLE student RENAME TO tb_students_ifo;
```

**修改字段名称**

新数据类型可以设置为原来一样，但是不能为空
修改数据类型可能会映像数据表中原有的数据，需要谨慎修改。

```mysql
ALTER TABLE <表名> CHANGE <旧字段名> <新字段名> <新数据类型>；
ALTER TABLE tb_emp1 CHANGE col1 col3 CHAR(3);
```

**修改字段数据类型**

```mysql
ALTER TABLE <表名> MODIFY <字段名> <数据类型>
ALTER TABLE tb_emp1 MODIFY name VARCHAR(30)；
```

**删除字段**

```mysql
ALTER TABLE <表名> DROP <字段名>；
ALTER TABLE tb_emp1 DROP col2;
```

### 删除数据表

```mysql
DROP TABLE [IF EXISTS] 表名1 [ ,表名2, 表名3 ...]
DROP TABLE tb_emp3;
```

[IF EXISTS]用于判断，不加，当表不存在则会报错，加了之后可能顺利执行，发出warning

**MySQL删除被其它表关联的主表**

数据表之间经常存在**外键关联**的情况，这时如果直接删除父表，会破坏数据表的完整性，也会删除失败。

删除父表有以下两种方法：

1. **先删除与它关联的子表，再删除父表；但是这样会同时删除两个表中的数据。**
2. **将关联表的外键约束取消，再删除父表；适用于需要保留子表的数据，只删除父表的情况。**

 

介绍第二种方法：

 创建两个表

```mysql
CREATE TABLE tb_emp4
(
id INT(11) PRIMARY KEY,
name VARCHAR(22),
location VARCHAR (50)
);
 
CREATE TABLE tb_emp5
(
id INT(11) PRIMARY KEY,
name VARCHAR(25),
deptId INT(11),
salary FLOAT,
CONSTRAINT fk_emp4_emp5 FOREIGN KEY (deptId) REFERENCES tb_emp4(id)
);
```

其中tb_emp5 表为子表，具有名称为 fk_emp4_emp5 的外键约束；tb_emp4 为父表，其主键 id 被子表 tb_ emp5 所关联。

若直接删除父表tb_emp4，会报错，当存在外键约束时，不能被直接删除

```mysql
DROP TABLE tb_emp4;
ERROR 1217 (23000): Cannot delete or update a parent row: a foreign key constraint fails
 
#首先解除子表的外键约束
ALTER TABLE tb_emp5 DROP FOREIGN KEY fk_emp4_emp5;
 
#接着就可以正常删除子表
DROP TABLE tb_emp4;
```

### 查看表结构

使用DESCRIBE 和 SHOW CREATE TABLE命令来查看数据表的结构。

```mysql
DESCRIBE table_name;
SHOW CREATE TABLE table_name;
```

### 数据表添加字段

MySQL 数据表是由行和列构成的，通常把表的“**列”称为字段（Field）**，把表的“**行”称为记录（Record）**

- **在末尾添加字段**

```mysql
ALTER TABLE <表名> ADD <新字段名><数据类型>[约束条件];
ALTER TABLE student ADD age INT(4);
```

- **在开头添加字段**

```mysql
ALTER TABLE <表名> ADD <新字段名> <数据类型> [约束条件] FIRST;
ALTER TABLE student ADD stuId INT(4) FIRST;
```

- **在中间位置添加字段**

```mysql
ALTER TABLE <表名> ADD <新字段名> <数据类型> [约束条件] AFTER <已经存在的字段名>;
ALTER TABLE student ADD stuno INT(11) AFTER name;
```

## 数据操作



## 约束

约束是指对表中**数据**的一种约束，用于保证数据库中数据的**正确性和有效性**。

MySQL主要支持以下6种约束：

- **主键约束**

表的一个特殊字段，该字段能唯一标识表中的每条信息。**一个表中只能有一个。**

- **外键约束**

通常与主键约束一起使用，保证数据的一致性。

例如，一个水果摊，只有苹果、桃子、李子、西瓜 4 种水果，那么，你来到水果摊要买水果只能选择苹果、桃子、李子和西瓜，不能购买其它的水果。

- **唯一约束**

唯一约束和主键约束相似的，能够确保列的唯一性。但是唯一约束在一个表中**可以有多个**，并且设置唯一约束的列是**允许有空值**的，这个空值最多**只能有一个**。

- **检查约束**

检查数据表中，**字段值是否有效**。如负数校验等

- **非空约束**

约束表中的字段不能为空

- **默认值约束**

默认值约束用来约束当数据表中某个字段不输入值时，自动为其添加一个已经设置好的值。

### 主键约束 PRIMARY KEY

主键分为**单字段主键**和**多字段联合主键**，本节将分别讲解这两种主键约束的创建、修改和删除。

使用主键应注意以下几点：

- 每个表只能定义**一个主键**。
- 主键值必须唯一标识表中的每一行，且**不能为 NULL**，即表中不可能存在有相同主键值的两行数据。这是**唯一性原则**。
- 一个字段名只能在联合主键字段表中出现一次。
- 联合主键**不能包含不必要的多余字段**。当把联合主键的某一字段删除后，如果剩下的字段构成的主键仍然满足唯一性原则，那么这个联合主键是不正确的。这是**最小化原则**。

**在创建表时设置主键约束**

**设置单字段主键**

在 CREATE TABLE 语句中，通过 PRIMARY KEY 关键字来指定主键。

```mysql
<字段名> <数据类型> PRIMARY KEY [默认值]
 
mysql> CREATE TABLE tb_emp3
  -> (
  -> id INT(11) PRIMARY KEY,
  -> name VARCHAR(25),
  -> deptId INT(11),
  -> salary FLOAT
  -> );
```

或者是在定义完所有字段之后指定主键，语法格式如下：

```mysql
[CONSTRAINT <约束名>] PRIMARY KEY [字段名]
 
mysql> CREATE TABLE tb_emp4
  -> (
  -> id INT(11),
  -> name VARCHAR(25),
  -> deptId INT(11),
  -> salary FLOAT,
  -> PRIMARY KEY(id)
  -> );
```

**设置联合主键**

比如，设置学生选课数据表时，使用学生编号做主键还是用课程编号做主键呢？如果用学生编号做主键，那么一个学生就只能选择一门课程。如果用课程编号做主键，那么一门课程只能有一个学生来选。显然，这两种情况都是不符合实际情况的。

实际上设计学生选课表，**要限定的是一个学生只能选择同一课程一次**。因此，学生编号和课程编号可以放在一起共同作为主键，这也就是联合主键了。

```mysql
PRIMARY KEY [字段1，字段2，…,字段n]
 
mysql> CREATE TABLE tb_emp5
  -> (
  -> name VARCHAR(25),
  -> deptId INT(11),
  -> salary FLOAT,
  -> PRIMARY KEY(name,deptId)
  -> );
```

**在修改表时添加主键约束**

```mysql
ALTER TABLE <数据表名> ADD PRIMARY KEY(<字段名>);
 
mysql> ALTER TABLE tb_emp2
  -> ADD PRIMARY KEY(id);
```

**删除主键约束**

```mysql
ALTER TABLE <数据表名> DROP PRIMARY KEY;
 
mysql> ALTER TABLE tb_emp2
  -> DROP PRIMARY KEY;
```

### 主键自增长 AUTO_INCREMENT

```mysql
字段名 数据类型 AUTO_INCREMENT
mysql> CREATE TABLE tb_student(
  -> id INT(4) PRIMARY KEY AUTO_INCREMENT,
  -> name VARCHAR(25) NOT NULL
  -> );
```

- 默认情况下，AUTO_INCREMENT 的**初始值是** **1**，每新增一条记录，字段值自动加 1。
- 一个表中只能有一个字段使用 AUTO_INCREMENT 约束，且该字段**必须有唯一索引**，以避免序号重复（即为主键或主键的一部分）。
- AUTO_INCREMENT 约束的字段必须具备 **NOT NULL** 属性。
- AUTO_INCREMENT 约束的字段只能是**整数类型**（TINYINT、SMALLINT、INT、BIGINT 等）。
- AUTO_INCREMENT 约束字段的最大值受该字段的**数据类型约束**，如果达到上限，AUTO_INCREMENT 就会失效。

**指定自增字段初始值**

```mysql
mysql> CREATE TABLE tb_student2 (
  -> id INT NOT NULL AUTO_INCREMENT,
  -> name VARCHAR(20) NOT NULL,
  -> PRIMARY KEY(ID)
  -> )AUTO_INCREMENT=100;
```

### 外键约束 FOREIGN KEY

MySQL 外键约束（FOREIGN KEY）是表的一个特殊字段，经常与主键约束一起使用。对于两个具有关联关系的表而言，相关联字段中主键所在的表就是**主表（父表）**，外键所在的表就是**从表（子表）**。

主表删除某条记录时，从表中与之对应的记录也必须有相应的改变。一个表可以**有一个或多个外键**，外键可以为空值，若不为空值，则**每一个外键的值必须等于主表中主键的某个值。**

定义外键时，需要遵守下列规则：

- 主表必须**已经存在**于数据库中，或者是**当前正在创建的表**。如果是后一种情况，则主表与从表是**同一个表**，这样的表称为**自参照表**，这种结构称为**自参照完整性**。
- 必须**为主表定义主键**。
- **主键不能包含空值**，但允许在外键中出现空值。也就是说，只要外键的每个非空值出现在指定的主键中，这个外键的内容就是正确的。
- 在主表的表名后面指定列名或列名的组合。这个列或列的组合必须是主表的主键或候选键。
- 外键中**列的数目必须和主表的主键中列的数目**相同。
- 外键中**列的数据类型必须和主表主键中对应列的数据类型**相同。

**语法**

```mysql
[CONSTRAINT <外键名>] FOREIGN KEY 字段名 [，字段名2，…]
REFERENCES <主表名> 主键列1 [，主键列2，…]
```

```mysql
# 创建一个部门表tb_dept1，结构如下
CREATE TABLE tb_dept1
(
id INT(11) PRIMARY KEY,
name VARCHAR(22) NOT NULL,
location VARCHAR(50) 
);

# 创建数据表tb_emp6，并在表中创建外键约束。将它的键deptId作为外键关联到表tb_dept1的主键id

CREATE TABLE tb_emp6
(
id INT(11) PRIMARY KEY,
name VARCHAR(25),
deptId INT(11),
salary FLOAT,
CONSTRAINT fk_emp_dept1
FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
);
```



**在修改表时添加外键约束**

外键约束也可以在修改表时添加，但是添加外键约束的前提是：**从表中外键列中的数据必须与主表中主键列中的数据一致或者是没有数据**。

```mysql
ALTER TABLE <数据表名> ADD CONSTRAINT <外键名>
FOREIGN KEY(<列名>) REFERENCES <主表名> (<列名>);
 
ALTER TABLE tb_emp2 ADD CONSTRANT fk_tb_dept1 FOREIGN KEY(deptId) REFERENCES tb_dept1(id);
```

 

**删除外键约束**

```mysql
ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>;
 
ALTER TABLE tb_emp2 DROP FOREIGN KEY fk_tb_dept1;
```



### 唯一约束 UNIQUE

MySQL 唯一约束（Unique Key）是指所有记录中字段的值不能重复出现。

唯一约束在一个表中**可有多个**，并且设置唯一约束的列**允许有空值**，但是**只能有一个空值。**



**创建表时设置唯一约束**

```mysql
<字段名> <数据类型> UNIQUE
 
CREATE TABLE tb_dept2
(
id INT(11) PRIMARY KEY,
name VARCHAR(22) UNIQUE,
location VARCHAR(50)
);
```



**修改表时设置唯一约束**

```mysql
ALTER TABLE <table name> ADD CONSTRAINT <unique name> UNIQUE(<column name>)
 
ALTER TABLE tb_dept1 ADD CONSTRAINT unique_name UNIQUE(name);
```



**删除唯一约束**

```mysql
ALTER TABLE <表名> DROP INDEX <唯一约束名>;
 
ALTER TABLE tb_dept1 DROP INDEX unique_name;
```

### 检查约束 CHECK

MySQL 检查约束（CHECK）是用来**检查数据表中字段值有效性**的一种手段

包括**基于列**的CHECK约束和**基于表**的CHECK约束

注意：若将 CHECK 约束子句置于所有列的定义以及主键约束和外键定义之后，则这种约束也称为基于表的 CHECK 约束。该约束可以同时对表中多个列设置限定条件。

 

**在创建表时设置检查约束**

```mysql
mysql> CREATE TABLE tb_emp7
  -> (
  -> id INT(11) PRIMARY KEY,
  -> name VARCHAR(25),
  -> deptId INT(11),
  -> salary FLOAT,
  -> CHECK(salary>0 AND salary<100),
  -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
  -> );
Query OK, 0 rows affected (0.37 sec)
```

 

**在修改表时添加检查约束**

```mysql
ALTER TABLE tb_emp7 ADD CONSTRAINT <检查约束名> CHECK(<检查约束>)
 
ALTER TABLE tb_emp7 ADD CONSTRAINT check_id CHECK(id>0);
```

 

**删除检查约束**

```mysql
ALTER TABLE <数据表名> DROP CONSTRAINT <检查约束名>;
 
ALTER TABLE tb_emp7 DROP CONSTRAINT check_id;
```

### 默认值约束 DEFAULT

默认值约束通常用在已经设置了非空约束的列，这样能够防止数据表在录入数据时出现错误。

 

**在创建表时设置默认值约束**

```mysql
<字段名> <数据类型> DEFAULT <默认值>;
 
CREATE TABLE tb_dept3
(
id INT(11) PRIMARY KEY,
name VARCHAR(22),
location VARCHAR(50) DEFAULT 'Beijing'
);
```

 

**在修改表时添加默认值约束**

```mysql
ALTER TABLE <数据表名> CHANGE COLUMN <字段名> <数据类型> DEFAULT <默认值>;
 
ALTER TABLE tb_dept3 CHANGE COLUMN location VARCHAR(50) DEFAULT 'Shanghai';
```

 

**删除默认值约束**

```mysql
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> DEFAULT NULL;
ALTER TABLE tb_dept3 CHANGE COLUMN location VARCHAR(50) DEFAULT NULL;
```



### 非空约束 NOT NULL

MySQL 非空约束（NOT NULL）指字段的值不能为空。

**在创建表时设置非空约束**

```mysql
<字段名> <数据类型> NOT NULL;
CREATE TABLE tb_dept4
(
id INT(11) PRIMARY KEY,
name VARCHAR(22) NOT NULL,
location VARCHAR(50)
);
```

**在修改表时添加非空约束**

```mysql
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名>
<字段名> <数据类型> NOT NULL;
 
 ALTER TABLE tb_dept4 CHANGE COLUMN location VARCHAR(50) NOT NULL;
```

**删除非空约束**

```mysql
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> NULL;
 
ALTER TABLE tb_dept4 CHANGE COLUMN location VARCHAR(50) NULL;
```



## 存储过程

在数据库的实际操作中，经常会有需要多条 SQL 语句处理多个表才能完成的操作。

存储过程是一组为了完成特定功能的 **SQL 语句集合**。使用存储过程的目的是将常用或复杂的工作预先用 SQL 语句写好并用一个指定名称存储起来，这个过程经编译和优化后存储在数据库服务器中，因此称为存储过程。当以后需要数据库提供与已定义好的存储过程的功能相同的服务时，只需调用“CALL存储过程名字”即可自动完成。

**1) 封装性**

**2) 可增强 SQL 语句的功能和灵活性**

存储过程可以用流程控制语句编写，可以完成复杂的判断和较复杂的运算。

**3) 可减少网络流量**

由于存储过程是在服务器端运行的，且执行速度快，因此当客户计算机上调用该存储过程时，网络中传送的只是该调用语句，从而可降低网络负载。

**4) 高性能**

当存储过程被成功编译后，就存储在数据库服务器里了，以后客户端可以直接调用，这样所有的 SQL 语句将从服务器执行，从而提高性能。但需要说明的是，**存储过程不是越多越好，过多的使用存储过程反而影响系统性能。**

**5) 提高数据库的安全性和数据的完整性**

存储过程提高安全性的一个方案就是把它作为**中间组件**，存储过程里可以对某些表做相关操作，然后存储过程作为接口提供给外部程序。这样，**外部程序无法直接操作数据库表**，只能通过存储过程来操作对应的表，因此在一定程度上，安全性是可以得到提高的。

**6) 使数据独立**

数据的独立可以达到解耦的效果，也就是说，程序可以调用存储过程，来替代执行多条的 SQL 语句。这种情况下，存储过程把数据同用户隔离开来，优点就是当数据表的结构改变时，调用表不用修改程序，只需要数据库管理者重新编写存储过程即可。

**具体语法查询**

[创建存储过程](http://c.biancheng.net/view/2593.html)

[查看存储过程](http://c.biancheng.net/view/7238.html)

[修改存储过程](http://c.biancheng.net/view/2594.html)

[删除存储过程](http://c.biancheng.net/view/2596.html)

# 事务

## 概念

当多个用户访问同一数据时，一个用户在更改数据的过程中可能有其它用户同时发起更改请求，**为保证数据的一致性状态**，MySQL 引入了事务。

事务将一系列的数据操作捆绑成一个整体进行统一管理，事务执行成功后，所有数据更改均会提交。若事务执行遇到错误，则会进行取消或回滚，数据会全部恢复到操作前的状态。（要么全部成果，要么全部失败）

事务是一个不可分割的工作逻辑单元。具有四个特性（ACID）：

- 原子性 Atomicity
- 一致性 Consistency
- 隔离性 Isolation
- 持久性 Durability



**原子性**

事务各元素不可分割

**一致性**

事务完成时，数据必须处于一致状态。

以银行转账事务事务为例。在事务开始之前，所有账户余额的总额处于一致状态。在事务进行的过程中，一个账户余额减少了，而另一个账户余额尚未修改。因此，所有账户余额的总额处于不一致状态。事务完成以后，账户余额的总额再次恢复到一致状态。

**隔离性**

并发的事务是彼此隔离的，事务必须是独立的，不应以任何方式依赖于或影响其他事务。事务正在修改数据时，任何其他进程对数据的操作必须要等到事务成功提交之后才能生效。

**持久性**

一个事务成功完成之后，它对数据库所作的改变是永久性的，即使系统出现故障也是如此。也就是说，一旦事务被提交，事务对数据所做的任何变动都会被永久地保留在数据库中。

## 执行事务的语法和流程

InnoDB通过UNDO日志和REDO日志实现。MyISAM不支持事务。

UNDO日志：复制事务执行前的数据，用于事务异常时回滚数据

REDO日志：记录事务执行中，每条对数据进行更新的操作，当事物提交时，该内容将被刷新到磁盘。

```mysql
# 开始事务
BEGIN

START TRANSACTION

# 提交事务
COMMIT

#回滚事务
ROLLBACK
```

## 事务的隔离级别

事务的隔离性指当多个事务同时运行时，各个事务之间相互隔离，不可相互干扰。事务并发时可能会出现**脏读、不可重复读、幻读**等情况。

1、**脏读**

指一个事务正在访问数据，并对数据进行了修改，但是这个修改还没有提交到数据库中，这时另一个事务也访问这个数据，并且使用了这个数据。

2、**不可重复读**

指在一个事务内，多次读取同一个数据。

在这个事务还没有结束时，**另外一个事务也访问了该同一数据**。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。

3、**幻读**

幻读是指当事务不是独立执行时发生的一种现象，例如**第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行**。同时，第二个事务也修改这个表中的数据，这种修改是向表中**插入一行新数据**。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

标准SQL定义4种隔离级别：

- **读未提交（READ UNCOMMITED）**
- **读提交（READ COMMITED）**
- **可重复读（REPEATABLE READ）**
- **串行化（SERIALIZABLE）**

在MySQL中，事务的默认隔离级别是REPEATABLE READ隔离级别，事务未结束时，其他会话只能读取到未提交数据。

MySQL 事务隔离级别可能产生的问题如下表所示：



| 隔离级别        | 脏读 | 不可重复读 | 幻读 |
| --------------- | ---- | ---------- | ---- |
| READ UNCOMITTED | √    | √          | √    |
| READ COMMITTED  | ×    | √          | √    |
| REPEATABLE READ | ×    | ×          | √    |
| SERIALIZABLE    | ×    | ×          | ×    |

MySQL 的事务的隔离级别由低到高分别为 READ UNCOMITTED、READ COMMITTED、REPEATABLE READ、SERIALIZABLE。低级别的隔离级别可以支持更高的并发处理，同时占用的系统资源更少。

### 读未提交

可以读到未提交的内容。几乎不用。

### 读提交

一个事务只能读取到另一个已提交事务修改过的数据，并且其它事务每对该数据进行一次修改并提交后，该事务都能查询得到最新值，那么这种隔离级别就称之为读提交。

Oracle、SQL Server等大多数数据库的默认事务隔离级别是读提交，但MySQL不是。

### 可重复读

专门针对不可重复读这种情况而指定的隔离级别。

在一些场景中，一个事务只能读取到另一个已提交事务修改过的数据，但是第一次读过某条记录后，即使其它事务修改了该记录的值并且提交，之后该事务再读该条记录时，读到的仍是第一次读到的值，而不是每次都读到不同的数据。那么这种隔离级别就称之为可重复读。

能确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。在该隔离级别下，**如果有事务正在读取数据，就不允许有其它事务进行修改操作**，这样就解决了可重复读问题。

### 串行化

如果一个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次按照该条件查询时，能把另一个事务插入的记录也读出来。那么这种隔离级别就称之为串行化。

SERIALIZABLE 是最高的事务隔离级别，主要通过强制事务排序来解决幻读问题。简单来说，就是在每个读取的数据行上加上共享锁实现，这样就避免了脏读、不可重复读和幻读等问题。但是该事务隔离级别执行效率低下，且性能开销也最大，所以一般情况下不推荐使用。

## 查看和修改事务隔离级别

```mysql
# 查看
show variables like '%transaction_isolation%';

select @@transaction_isolation;
                         
#修改
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE};

set tx_isolation='READ-COMMITTED';
                         
```

## MySQL锁机制

锁机制用于解决数据库的并发控制问题，用于保证数据的一致性，并为隔离级别提供保证。锁机制的优劣直接影响数据库并发处理能力和系统性能。

按锁级别分类：共享锁、排他锁、意向锁

按锁粒度分类：行级锁、表级锁、页级锁

1、**共享锁**

Share，代号S，也称为读锁。一种可以查看但无法修改和删除的数据锁。

共享锁的锁粒度是行或者元组。一个事务获取了共享锁之后，可以对锁定范围内的数据执行读操作。会阻止其它事务获得相同数据集的排他锁。

2、**排他锁**

eXclusive，代号X，也称为写锁，是基本的锁类型。

排他锁的代号是 X，是 eXclusive 的缩写，也可称为写锁，是基本的锁类型。

排他锁的粒度与共享锁相同，也是行或者元组。一个事务获取了排他锁之后，可以对锁定范围内的数据执行写操作。允许获得排他锁的事务更新数据，阻止其它事务取得相同数据集的共享锁和排他锁。

如有两个事务 A 和 B，如果事务 A 获取了一个元组的共享锁，事务 B 还可以立即获取这个元组的共享锁，但不能立即获取这个元组的排他锁，必须等到事务 A 释放共享锁之后才可以。

如果事务 A 获取了一个元组的排他锁，事务 B 不能立即获取这个元组的共享锁，也不能立即获取这个元组的排他锁，必须等到 A 释放排他锁之后才可以。

3、**意向锁**

为了允许**行锁和表锁共存**，实现多粒度锁机制，InnoDB 还有两种内部使用的意向锁。

意向锁是一种**表锁**，锁定的粒度是整张表，分为**意向共享锁（IS）**和**意向排他锁（IX）**两类。

意向共享锁表示一个事务**有意对数据上共享锁或者排他锁。**“有意”表示事务**想执行操作但还没有真正执行。**

锁的兼容模式：Y表示相容、N表示互斥

| 参数             | X    | S    | IX   | IS   |
| ---------------- | ---- | ---- | ---- | ---- |
| X（排他锁）      | N    | N    | N    | N    |
| S（共享锁）      | N    | Y    | N    | Y    |
| IX（意向排他锁） | N    | N    | Y    | Y    |
| IS（意向共享锁） | N    | Y    | Y    | Y    |

如果一个事务请求的锁模式与当前的锁兼容，InnoDB 就将请求的锁授予该事务；反之，如果两者不兼容，该事务就要等待锁释放。

1、**表级锁 table lock**

表级锁为表级别的锁定，会锁定整张表，可以很好的避免死锁，是 MySQL 中最大颗粒度的锁定机制。

实现逻辑非常简单，带来的系统负面影响最小。

出现锁定资源争用的概率会很高，致使并发度大打折扣。

服务器会为诸如 ALTER TABLE 之类的语句使用表级锁，而忽略存储引擎的锁机制。

2、**页级锁 page lock**

页级锁是 MySQL 中比较独特的一种锁定级别。

页级锁的颗粒度介于行级锁与表级锁之间，所以获取锁定所需要的资源开销，以及所能提供的并发处理能力同样也是介于上面二者之间。另外，页级锁和行级锁一样，**会发生死锁**。

页级锁主要应用于 **BDB 存储引擎**。

3、**行级锁 row lock**

粒度最小，**只针对操作的当前行**进行加锁，所以行级锁发生锁定资源争用的概率也最小。

由于锁定资源的颗粒度很小，所以每次获取锁和释放锁需要做的事情也就更多，带来的消耗自然也就更大。此外，行级锁也**最容易发生死锁**。所以说行级锁最大程度地支持并发处理的同时，也带来了最大的锁开销。

随着锁定资源颗粒度的减小，锁定相同数据量的数据所需要消耗的内存数量也越来越多，实现算法也会越来越复杂。不过，随着锁定资源颗粒度的减小，应用程序的访问请求遇到锁等待的可能性也会随之降低，系统整体并发度也会随之提升。

行级锁主要应用于 InnoDB 存储引擎。

|        | 表级锁       | 行级锁     | 页级锁                 |
| ------ | ------------ | ---------- | ---------------------- |
| 开销   | 小           | 大         | 介于表级锁和行级锁之间 |
| 加锁   | 快           | 慢         | 介于表级锁和行级锁之间 |
| 死锁   | 不会出现死锁 | 会出现死锁 | 会出现死锁             |
| 锁粒度 | 大           | 小         | 介于表级锁和行级锁之间 |
| 并发度 | 低           | 高         | 一般                   |

从锁的角度来说，**表级锁**适合以查询为主，只有少量按索引条件更新数据的应用，如 Web 应用。而**行级锁**更适合于有大量按索引条件，同时又有并发查询的应用，如一些**在线事务处理（OLTP）系统。**

### MySQL InnoDB 3种行锁定方式

InnoDB 支持 3 种行锁定方式：

- 行锁（Record Lock）：直接对索引项加锁。
- 间隙锁（Gap Lock）：锁加在索引项之间的间隙，也可以是第一条记录前的“间隙”或最后一条记录后的“间隙”。
- Next-Key Lock：行锁与间隙锁组合起来用就叫做 Next-Key Lock。 前两种的组合，对记录及其前面的间隙加锁

默认情况下，InnoDB 工作在**可重复读**（默认隔离级别）下，并且以 **Next-Key Lock** 的方式对数据行进行加锁，这样可以**有效防止幻读**的发生。

Next-Key Lock 是行锁与间隙锁的组合，这样，

1. 当 InnoDB 扫描索引项的时候，会首先**对选中的索引项加上行锁**（Record Lock）
2. 再**对索引项两边的间隙**（向左扫描扫到第一个比给定参数小的值， 向右扫描扫到第一个比给定参数大的值， 然后以此为界，构建一个区间）**加上间隙锁**（Gap Lock）。如果一个间隙被事务 T1 加了锁，其它事务不能在这个间隙插入记录。

### MySQL并发时常见的死锁和解决方法

**死锁是指两个或两个以上的事务在执行过程中，因争夺资源而造成的一种互相等待的现象。**

```
“Deadlock found when trying to get lock...”
```



```mysql
#查看死锁信息
SHOW ENGINE INNODB STATUS;
```

**死锁检测**

InnoDB 的并发写操作会触发死锁，同时 InnoDB 也提供了死锁检测机制。通过设置 innodb_deadlock_detect 参数的值来控制是否打开死锁检测。

- innodb_deadlock_detect = ON ：默认值，打开死锁检测。数据库发生死锁时，系统会**自动回滚其中的某一个事务**，让其它事务可以继续执行。
- innodb_deadlock_detect = OFF：关闭死锁检测。发生死锁时，系统会用锁等待来处理。

> 锁等待是指在事务过程中产生的锁，其它事务需要等待上一个事务释放锁，才能占用该资源。如果该事务一直不释放，就需要持续等待下去，直到超过了锁等待时间。当超过锁等待允许的最大时间，就会出现死锁，然后当前事务执行失败，自动执行回滚操作。

在实际应用中，我们要尽量防止锁等待现象的发生，下面介绍几种避免死锁的方法：

1. 如果不同程序会并发存取多个表，或者涉及多行记录时，尽量**约定以相同的顺序**访问表，这样可以大大降低死锁的发生。
2. 业务中要**及时提交或者回滚事务**，可减少死锁产生的概率。
3. 在同一个事务中，尽可能做到**一次锁定所需要的所有资源**，减少死锁产生概率。
4. 对于非常容易产生死锁的业务部分，可以尝试使用**升级锁粒度**，通过表锁定来减少死锁产生的概率（**表级锁不会产生死锁**）。

### 锁监控

http://c.biancheng.net/view/vip_8314.html

## MySQL用户管理

http://c.biancheng.net/mysql/100/





# MYSQL基本架构示意图

MYSQL由Server层和存储引擎层两部分

<img src="DataBase.assets/0d2070e8f84c4801adbfa03bda1f98d9.png" alt="img" style="zoom: 33%;" />

- Server层：涵盖MYSQL大多数核心服务功能，以及所有内置函数。所有**跨存储引擎功能**在这一层实现，如存储过程、触发器、视图。
- 存储引擎层：负责数据的存储和提取。其架构模式是**插件式**的，支持InnoDB、MyISAM、Memory等多个存储引擎。**InnoDB从MySQL 5.5.5版本开始成为了默认存储引擎**

# 一条MYSQL语句的执行过程

**连接器**：连接到数据库。连接器负责跟客户端建立连接，获取权限，维持和管理连接。

**查询缓存**：MYSQL拿到查询请求后，会首先通过查询缓存看看之前是不是执行过这条语句。执行过则直接从缓存返回结果。

查询缓存往往**弊大于利**，查询缓存失效非常频繁，只要有表更新，这个表上所有的查询缓存都会被清空。**MYSQL8.0删除了查询缓存的功能。**

**分析器**：对输入的SQL语句做**词法分析**和**语法分析**。

**优化器**：在表里有**多个索引**时，用于决定使用哪个索引；或者在一个语句有**多表关联(join)**时，决定各个表的连接顺序。（优化器选择索引的方法）

**执行器**：执行器首先判断对表T是否有执行权限。有权限则调用引擎提供的接口。（取下一行、取满足条件的下一行）

> Q：如果表T中没有字段k，而你执行了这个语句 select * from T where k=1, 那肯定是会报“不存在这个列”的错误： “Unknown column ‘k’ in ‘where clause’”。你觉得这个错误是在我们上面提到的哪个阶段报出来的呢？
>
> A：分析器，分析器阶段会判断语句是否正确，表、列是否存在。不是执行器，因为表、列的字段是事先定义好的，不是存在存储引擎中的数据，不需要打开引擎去读取。
>
> Q：如果是一个对表T没有权限的用户执行此语句，报错是“select command denied”，而不是“unknown column”
>
> A：出于安全性考虑，对没有权限的用户，其也没有权限知道列k是否存在。

# 一条MYSQL更新语句的执行过程



# 面试相关

## 极高频，不会很危险

知道哪些存储引擎，InnoDB和MyISAM区别和特点，适用于什么场景

索引使用的数据结构，为什么使用b+树，和b树、红黑树、hash表的比较

什么是聚集索引、辅助索引，如何避免回表，覆盖索引

联合索引及其要求（最左匹配等，通常考察方式是给一个表和sql问是否走索引及为什么）

事务的ACID四大特性、事务的隔离级别、脏读、不可重复读、幻读

## 高频，应当熟练

多表查询中内连接、外连接、笛卡尔积等

select * from t where * group by * having * order by * limit * 标准查询语句（字节常考SQL题）

三大范式

一条SQL语句查询很慢有哪些解决方法

SQL注入攻击及预编译

你知道MySQL中有哪些日志（重点是binary log、redo log、undo log），他们的区别、作用是什么？

MVCC的作用、原理、MySQL是怎么实现的

索引优化，选取怎样的列作索引列，怎么安排索引列的顺序，如果有几个查询任务你怎样建立索引，索引失效的场景有哪些

读锁、写锁、（后面几个是我觉得完整的知识体系必须具备的，面试中能主动提到或者答上来绝对是极大的加分点：意向锁、next-key lock、gap lock）

MySQL两个账户相互转账会发生什么，如何解决出现的死锁问题

MySQL是如何解决脏读、不可重复读、幻读的？前两个是mvcc、后一个是next-key lock

事务的持久性是如何保证的？（可以想想隔离性、原子性、一致性又是如何保证的，这三个不常问，但是勤思考总是没错的）

## 一般，拓宽知识，加分项

MySQL支持的基本数据类型（问的较少，但是不会的话场面会很难看）

主键、外键、唯一约束、视图等知识

分区分表（根据功能、业务、访问频率等按列分，根据时间、用户、hash等按行分）

查询语句优化技巧，如覆盖索引的延迟关联、limit 10000000， 100语句的优化、explain的运用

一条sql语句的执行过程（包括查询和修改两大类操作）

什么是快照读和当前读

MySQL复制是如何实现的？主从复制的好处是什么？