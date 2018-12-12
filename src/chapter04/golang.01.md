### 数据库

数据库这块我们先从在开发中常用的数据库Mysql开始，MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下公司。MySQL 最流行的关系型数据库管理系统，在 WEB 应用方面MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。

MySQL基础:

* [MySQL的多存储引擎架构](#MySQL的多存储引擎架构)
* [MySQL的数据类型](#MySQL的数据类型)
* [数据库的事务](#数据库的事务)
* [数据库ACID](#数据库ACID)
* [数据库中的范式](#数据库中的范式)
* [并发一致性问题](#并发一致性问题)
* [事务隔离级别](#事务隔离级别)
* [存储引擎](#存储引擎)
* [MySQL的索引](#MySQL的索引)

#### MySQL的多存储引擎架构
<p align="center">
<img width="500" align="center" src="../images/2.jpg" />
</p>

MySQL作为一个大型的网络程序、数据管理系统，架构非常复杂。

#### MySQL的数据类型

1. 整型

| 类型      | 存储 | 存储 | 最小值                        | 最大值                        |
| --------- | ---- | ---- | ----------------------------- | ----------------------------- |
|           | byte | bit  | signed                        | signed                        |
| TINYINT   | 1    | 8    | -2<sup>7</sup> = -128         | 2<sup>7</sup>-1 = 127         |
| SMALLINT  | 2    | 16   |                               |                               |
| MEDIUMINT | 3    | 24   |                               |                               |
| INT       | 4    | 32   | -2<sup>31</sup> = -2147483648 | 2<sup>31</sup>-1 = 2147483647 |
| BIGINT    | 8    | 64   |                               |                               |

TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT 分别使用 8, 16, 24, 32, 64 位存储空间，一般情况下越小的列越好。

INT(11) 中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。

2. 浮点数

FLOAT 和 DOUBLE 为浮点类型，DECIMAL 为高精度小数类型。CPU 原生支持浮点运算，但是不支持 DECIMAl 类型的计算，因此 DECIMAL 的计算比浮点类型需要更高的代价。

FLOAT、DOUBLE 和 DECIMAL 都可以指定列宽，例如 DECIMAL(18, 9) 表示总共 18 位，取 9 位存储小数部分，剩下 9 位存储整数部分。

3. 字符串

MySQL中主要有 CHAR 和 VARCHAR 两种字符串类型，一种是定长的，一种是变长的。

VARCHAR 这种变长类型能够节省空间，因为只需要存储必要的内容。但是在执行 UPDATE 时可能会使行变得比原来长，当超出一个页所能容纳的大小时，就要执行额外的操作。MyISAM 会将行拆成不同的片段存储，而 InnoDB 则需要分裂页来使行放进页内。

VARCHAR 会保留字符串末尾的空格，而 CHAR 会删除。

4. 时间和日期

MySQL中提供了两种相似的日期时间类型：DATATIME 和 TIMESTAMP。

* DATATIME

能够保存从 1001 年到 9999 年的日期和时间，精度为秒，使用 8 字节的存储空间。

它与时区无关。

默认情况下，MySQL 以一种可排序的、无歧义的格式显示 DATATIME 值，例如“2008-01-16 22:37:08”，这是 ANSI 标准定义的日期和时间表示方法。

* TIMESTAMP

和 UNIX 时间戳相同，保存从 1970 年 1 月 1 日午夜（格林威治时间）以来的秒数，使用 4 个字节，只能表示从 1970 年 到 2038 年。

它和时区有关，也就是说一个时间戳在不同的时区所代表的具体时间是不同的。

MySQL 提供了 FROM_UNIXTIME() 函数把 UNIX 时间戳转换为日期，并提供了 UNIX_TIMESTAMP() 函数把日期转换为 UNIX 时间戳。

默认情况下，如果插入时没有指定 TIMESTAMP 列的值，会将这个值设置为当前时间。

应该尽量使用 TIMESTAMP，因为它比 DATETIME 空间效率更高。


#### 数据库的事务
<p align="center">
<img width="500" align="center" src="../images/3.jpg" />
</p>

数据库的事务指的是满足数据库的 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。

MySQL 中默认采用自动提交(AUTOCOMMIT)模式。如果不显式使用 START TRANSACTION 语句来开始一个事务，那么每个查询都会被当做一个事务自动提交。

#### 数据库ACID

1. 原子性（Atomicity）

原子性是指事务是一个不可分割的工作单位，事务中的操作要么全部成功，要么全部失败。比如在同一个事务中的SQL语句，要么全部执行成功，要么全部执行失败。
回滚可以用日志来实现，日志记录着事务所执行的修改操作，在回滚时反向执行这些修改操作即可。

2. 一致性（Consistency）

事务必须使数据库从一个一致性状态变换到另外一个一致性状态。以转账为例子，A向B转账，假设转账之前这两个用户的钱加起来总共是100，那么A向B转账之后，不管这两个账户怎么转，A用户的钱和B用户的钱加起来的总额还是100，这个就是事务的一致性。

3. 隔离性（Isolation）

隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。

即要达到这么一种效果：对于任意两个并发的事务 T1 和 T2，在事务 T1 看来，T2 要么在 T1 开始之前就已经结束，要么在 T1 结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。

4. 持久性（Durability）

一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。

可以通过数据库备份和恢复来实现，在系统发生奔溃时，使用备份的数据库进行数据恢复。

事务的 ACID 特性概念简单，但不是很好理解，主要是因为这几个特性不是一种平级关系：
* 只有满足一致性，事务的执行结果才是正确的。
* 在无并发的情况下，事务串行执行，隔离性一定能够满足。此时要只要能满足原子性，就一定能满足一致性。
* 在并发的情况下，多个事务并发执行，事务不仅要满足原子性，还需要满足隔离性，才能满足一致性。
* 事务满足持久化是为了能应对数据库奔溃的情况。

<p align="center">
<img width="500" align="center" src="../images/4.jpg" />
</p>

#### 数据库中的范式
满足最低要求的范式是第一范式（1NF）。在第一范式的基础上进一步满足更多规范要求的称为第二范式（2NF），其余范式以次类推。一般说来，数据库只需满足第三范式 (3NF）就可以了。范式的包含关系。一个数据库设计如果符合第二范式，一定也符合第一范式。如果符合第三范式，一定也符合第二范式…

* 1NF：属性不可分
* 2NF：属性完全依赖于主键 [消除部分子函数依赖]
* 3NF：属性不依赖于其它非主属性 [消除传递依赖]
* BCNF（巴斯-科德范式）：在1NF基础上，任何非主属性不能对主键子集依赖[在3NF基础上消除对主码子集的依赖]
* 4NF：要求把同一表内的多对多关系删除。
* 5NF（完美范式）：从最终结构重新建立原始结构。

范式理论是为了解决以上提到四种异常。高级别范式的依赖于低级别的范式，1NF 是最低级别的范式。

<p align="center">
<img width="500" align="center" src="../images/5.jpg" />
</p>

1. 第一范式 (1NF)

属性不可分。

2. 第二范式 (2NF)

每个非主属性完全函数依赖于键码。

可以通过分解来满足。

**分解前** 

| Sno  | Sname  | Sdept  | Mname  | Cname  | Grade |
| ---- | ------ | ------ | ------ | ------ | ----- |
| 1    | 学生-1 | 学院-1 | 院长-1 | 课程-1 | 60    |
| 2    | 学生-2 | 学院-2 | 院长-2 | 课程-2 | 90    |
| 2    | 学生-2 | 学院-2 | 院长-2 | 课程-1 | 100   |
| 3    | 学生-3 | 学院-2 | 院长-2 | 课程-2 | 95    |

以上学生课程关系中，{Sno, Cname} 为键码，有如下函数依赖：

- Sno -> Sname, Sdept
- Sdept -> Mname
- Sno, Cname-> Grade

Grade 完全函数依赖于键码，它没有任何冗余数据，每个学生的每门课都有特定的成绩。

Sname, Sdept 和 Mname 都部分依赖于键码，当一个学生选修了多门课时，这些数据就会出现多次，造成大量冗余数据。

**分解后** 

关系-1

| Sno  | Sname  | Sdept  | Mname  |
| ---- | ------ | ------ | ------ |
| 1    | 学生-1 | 学院-1 | 院长-1 |
| 2    | 学生-2 | 学院-2 | 院长-2 |
| 3    | 学生-3 | 学院-2 | 院长-2 |

有以下函数依赖：

- Sno -> Sname, Sdept
- Sdept -> Mname

关系-2

| Sno  | Cname  | Grade |
| ---- | ------ | ----- |
| 1    | 课程-1 | 90    |
| 2    | 课程-2 | 80    |
| 2    | 课程-1 | 100   |
| 3    | 课程-2 | 95    |

有以下函数依赖：

- Sno, Cname -> Grade

3. 第三范式 (3NF)

非主属性不传递函数依赖于键码。

上面的 关系-1 中存在以下传递函数依赖：

- Sno -> Sdept -> Mname

可以进行以下分解：

关系-11

| Sno  | Sname  | Sdept  |
| ---- | ------ | ------ |
| 1    | 学生-1 | 学院-1 |
| 2    | 学生-2 | 学院-2 |
| 3    | 学生-3 | 学院-2 |

关系-12

| Sdept  | Mname  |
| ------ | ------ |
| 学院-1 | 院长-1 |
| 学院-2 | 院长-2 |

#### 并发一致性问题 

1. 丢失修改

T1 和 T2 两个事务都对一个数据进行修改，T1 先修改，T2 随后修改，T2 的修改覆盖了 T1 的修改。
<p align="center">
<img width="500" align="center" src="../images/6.jpg" />
</p>

2. 脏数据读取
（针对未提交数据）如果一个事务中对数据进行了更新，但事务还没有提交，另一个事务可以 “看到” 该事务没有提交的更新结果，这样造成的问题就是，如果第一个事务回滚，那么，第二个事务在此之前所 “看到” 的数据就是一笔脏数据。 （脏读又称无效数据读出。一个事务读取另外一个事务还没有提交的数据叫脏读。 ）

应用示例：
```markdown
Jame的原工资为 1000, 财务人员将Jame的工资改为了 8000 (但未提交事务)

Jame读取自己的工资，发现自己的工资变为了 8000，欢天喜地！

而财务发现操作有误，回滚了事务，Jame的工资又变为了1000

像这样的事件，Jame记取的工资数8000就是一个脏数据。
```

解决办法：
```markdown
把数据库的事务隔离级别调整到 READ_COMMITTED
```
图解：
```markdown
1 修改一个数据，T2 随后读取这个数据。如果 T1 撤销了这次修改，那么 T2 读取的数据是脏数据。
```
<p align="center">
<img width="500" align="center" src="../images/7.jpg" />
</p>

3. 不可重复读

是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。（同时操作，事务1分别读取事务2操作时和提交后的数据，读取的记录内容不一致。不可重复读是指在同一个事务内，两个相同的查询返回了不同的结果。 ）

应用示例：
```markdown
#在事务1中，Jame读取了自己的工资为1000，操作并没有完成

con1 = getConnection();  
select salary from employee empId ="Mary"; 

#在事务2中，这时财务人员修改了Jame的工资为 2000，并提交了事务.

con2 = getConnection();  
update employee set salary = 2000;  
con2.commit();  

#在事务1中，Jame再次读取自己的工资时，工资变为了2000

#con1  
select salary from employee empId ="Mary";  
```
在一个事务中前后两次读取的结果并不致，导致了不可重复读。

解决办法：
```markdown
如果只有在修改事务完全提交之后才可以读取数据，则可以避免该问题。把数据库的事务隔离级别调整到REPEATABLE_READ
```
T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。

<p align="center">
<img width="500" align="center" src="../images/8.jpg" />
</p>

4. 幻读

事务 T1 读取一条指定的 Where 子句所返回的结果集，然后 T2 事务新插入一行记录，这行记录恰好可以满足T1 所使用的查询条件。然后 T1 再次对表进行检索，但又看到了 T2 插入的数据。 （和可重复读类似，但是事务 T2 的数据操作仅仅是插入和删除，不是修改数据，读取的记录数量前后不一致）.

幻读的重点在于新增或者删除 (数据条数变化).

同样的条件，第1次和第2次读出来的记录数不一样.

应用示例:

目前工资为1000的员工有10人
```markdown
#事务1，读取所有工资为 1000 的员工（共读取 10 条记录 ）

con1 = getConnection();  
Select * from employee where salary =1000;  

#这时另一个事务向 employee 表插入了一条员工记录，工资也为 1000

con2 = getConnection();  
Insert into employee(empId,salary) values("Lili",1000);  
con2.commit();  

#事务1再次读取所有工资为 1000的 员工（共读取到了 11 条记录，这就像产生了幻读）

#con1  
select * from employee where salary =1000;  
```
解决办法：
```markdown
如果在操作事务完成数据处理之前，任何其他事务都不可以添加新数据，则可避免该问题。把数据库的事务隔离级别调整到 SERIALIZABLE_READ
```
图解：

T1 读取某个范围的数据，T2 在这个范围内插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。

<p align="center">
<img width="500" align="center" src="../images/9.jpg" />
</p>

#### 事务隔离级别

1. 串行化 (Serializable)

所有事务一个接着一个的执行，这样可以避免幻读 (phantom read)，对于基于锁来实现并发控制的数据库来说，串行化要求在执行范围查询的时候，需要获取范围锁，如果不是基于锁实现并发控制的数据库，则检查到有违反串行操作的事务时，需回滚该事务。 

2. 可重复读 (Repeated Read)

所有被 Select 获取的数据都不能被修改，这样就可以避免一个事务前后读取数据不一致的情况。但是却没有办法控制幻读，因为这个时候其他事务不能更改所选的数据，但是可以增加数据，即前一个事务有读锁但是没有范围锁，为什么叫做可重复读等级呢？那是因为该等级解决了下面的不可重复读问题。

注意：现在主流数据库都使用 MVCC 并发控制，使用之后RR（可重复读）隔离级别下是不会出现幻读的现象。

3. 读已提交 (Read Committed)

被读取的数据可以被其他事务修改，这样可能导致不可重复读。也就是说，事务读取的时候获取读锁，但是在读完之后立即释放(不需要等事务结束)，而写锁则是事务提交之后才释放，释放读锁之后，就可能被其他事务修改数据。该等级也是 SQL Server 默认的隔离等级。

4. 读未提交 (Read Uncommitted)

最低的隔离等级，允许其他事务看到没有提交的数据，会导致脏读。

**总结**

- 四个级别逐渐增强，每个级别解决一个问题，每个级别解决一个问题，事务级别遇到，性能越差，大多数环境(Read committed 就可以用了) 

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| -------- | ---- | ---------- | ------ |
| 未提交读 | √    | √          | √      |
| 提交读   | ×    | √          | √      |
| 可重复读 | ×    | ×          | √      |
| 可串行化 | ×    | ×          | ×      |

#### 存储引擎
MySQL中提供了多个存储引擎，包括处理事务安全表的引擎和处理非事务安全表的引擎。在 MySQL 中，不需要在整个服务器中使用同一种存储引擎，针对具体的要求，可以对每一个表使用不同的存储引擎。 

MySQL 中的数据用各种不同的技术存储在文件（或者内存）中。这些技术中的每一种技术都使用不同的存储机制、索引技巧、锁定水平并且最终提供广泛的不同的功能和能力。通过选择不同的技术，你能够获得额外的速度或者功能，从而改善你的应用的整体功能。存储引擎说白了就是如何存储数据、如何为存储的数据建立索引和如何更新、查询数据等技术的实现方法。

通常，如果你在研究大量的临时数据，你也许需要使用内存存储引擎。内存存储引擎能够在内存中存储所有的表格数据。又或者，你也许需要一个支持事务处理的数据库（以确保事务处理不成功时数据的回退能力）。

在MySQL中有很多存储引擎，每种存储引擎大相径庭，那么又改如何选择呢？

`MySQL 5.5` 以前的默认存储引擎是 `MyISAM`, `MySQL 5.5` 之后的默认存储引擎是 `InnoDB`

不同存储引起都有各自的特点，为适应不同的需求，需要选择不同的存储引擎，所以首先考虑这些存储引擎各自的功能和兼容。

##### 1. MyISAM

MySQL 5.5 版本之前的默认存储引擎，在 `5.0` 以前最大表存储空间最大 `4G`，`5.0` 以后最大 `256TB`。

Myisam 存储引擎由 `.myd`（数据）和 `.myi`（索引文件）组成，`.frm`文件存储表结构（所以存储引擎都有）

**特性**

- 并发性和锁级别 （对于读写混合的操作不好，为表级锁，写入和读互斥）
- 表损坏修复
- Myisam 表支持的索引类型（全文索引）
- Myisam 支持表压缩（压缩后，此表为只读，不可以写入。使用 myisampack 压缩）

**应用场景**

- 没有事务
- 只读类应用（插入不频繁，查询非常频繁）
- 空间类应用（唯一支持空间函数的引擎）
- 做很多 count 的计算

##### 2. InnoDB

MySQL 5.5 及之后版本的默认存储引擎

**特性**

- InnoDB为事务性存储引擎
- 完全支持事物的 ACID 特性
- Redo log （实现事务的持久性） 和 Undo log（为了实现事务的原子性，存储未完成事务log，用于回滚）
- InnoDB支持行级锁
- 行级锁可以最大程度的支持并发
- 行级锁是由存储引擎层实现的

**应用场景**

- 可靠性要求比较高，或者要求事务
- 表更新和查询都相当的频繁，并且行锁定的机会比较大的情况。

##### 3. CSV

**文件系统存储特点**

- 数据以文本方式存储在文件中
- `.csv`文件存储表内容
- `.csm`文件存储表的元数据，如表状态和数据量
- `.frm`存储表的结构

**CSV存储引擎特点**

- 以 CSV 格式进行数据存储
- 所有列必须都是不能为 NULL
- 不支持索引
- 可以对数据文件直接编辑（其他引擎是二进制存储，不可编辑）

**引用场景**

- 作为数据交换的中间表

##### 4. Archive

**特性**

- 以 zlib 对表数据进行压缩，磁盘 I/O 更少
- 数据存储在ARZ为后缀的文件中（表文件为 `a.arz`，`a.frm`）
- 只支持 insert 和 select 操作（不可以 delete 和 update，会提示没有这个功能）
- 只允许在自增ID列上加索引

**应用场景**

- 日志和数据采集类应用

##### 5. Memory

特性:

- 也称为 HEAP 存储引擎，所以数据保存在内存中（数据库重启后会导致数据丢失）
- 支持 HASH 索引（等值查找应选择 HASH）和 BTree 索引（范围查找应选择）
- 所有字段都为固定长度，varchar(10) == char(10)
- 不支持 BLOG 和 TEXT 等大字段
- Memory 存储使用表级锁（性能可能不如 innodb） 
- 最大大小由 `max_heap_table_size` 参数决定
- Memory存储引擎默认表大小只有 `16M`，可以通过调整 `max_heap_table_size` 参数

应用场景:

- 用于查找或是映射表，例如右边和地区的对应表
- 用于保存数据分析中产生的中间表
- 用于缓存周期性聚合数据的结果表

注意： Memory 数据易丢失，所以要求数据可再生

##### 6. Federated

**特性**

- 提供了访问远程 MySQL 服务器上表的方法
- 本地不存储数据，数据全部放在远程服务器上

使用 Federated,默认是禁止的。如果需要启用，需要在启动时增加Federated参数

* 问：独立表空间和系统表空间应该如何抉择

**两者比较**

- 系统表空间：无法简单的收缩大小（这很恐怖，会导致 ibdata1 一直增大，即使删除了数据也不会变小）
- 独立表空间：可以通过 optimize table 命令收缩系统文件
- 系统表空间：会产生I/O瓶颈（因为只有一个文件）
- 独立表空间：可以向多个文件刷新数据

**总结**
强烈建议：对Innodb引擎使用独立表空间（mysql5.6版本以后默认是独立表空间）

**系统表转移为独立表的步骤（非常繁琐）**

- 使用 mysqldump 导出所有数据库表数据
- 停止 mysql 服务，修改参数，并且删除Innodb相关文件
- 重启 mysql 服务，重建mysql系统表空间
- 重新导入数据

* 问：如何选择存储引擎

**参考条件：**  

- 是否需要事务
- 是否可以热备份
- 崩溃恢复
- 存储引擎的特有特性  


**重要一点：** 不要混合使用存储引擎
**强烈推荐：** Innodb

* 问：MyISAM和InnoDB引擎的区别

**区别：**

- MyISAM 不支持外键，而 InnoDB 支持
- MyISAM 是非事务安全型的，而 InnoDB 是事务安全型的。
- MyISAM 锁的粒度是表级，而 InnoDB 支持行级锁定。
- MyISAM 支持全文类型索引，而 InnoDB 不支持全文索引。
- MyISAM 相对简单，所以在效率上要优于 InnoDB，小型应用可以考虑使用 MyISAM。
- MyISAM 表是保存成文件的形式，在跨平台的数据转移中使用 MyISAM 存储会省去不少的麻烦。
- InnoDB 表比 MyISAM 表更安全，可以在保证数据不会丢失的情况下，切换非事务表到事务表（alter table tablename type=innodb）。

**应用场景：**

- MyISAM 管理非事务表。它提供高速存储和检索，以及全文搜索能力。如果应用中需要执行大量的 SELECT 查询，那么 MyISAM 是更好的选择。
- InnoDB 用于事务处理应用程序，具有众多特性，包括 ACID 事务支持。如果应用中需要执行大量的 INSERT
   或 UPDATE 操作，则应该使用 InnoDB，这样可以提高多用户并发操作的性能。

#### MySQL的索引

##### 索引使用的场景

索引能够轻易将查询性能提升几个数量级。

* 对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效。
* 对于中到大型的表，索引就非常有效。
* 但是对于特大型的表，建立和维护索引的代价将会随之增长。这种情况下，需要用到一种技术可以直接区分出需要查询的一组数据，而不是一条记录一条记录地匹配，例如可以使用分区技术。

索引是在存储引擎层实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。

##### 索引分类

| 特性                         | 说明                                                 | InnoDB | MyISAM | MEMORY |
| ---------------------------- | ---------------------------------------------------- | ------ | ------ | ------ |
| B树索引 (B-tree indexes)     | 自增ID物理连续性更高，<br />二叉树，红黑树高度不可控 | √      | √      | √      |
| R树索引 (R-tree indexes)     | 空间索引                                             |        | √      |        |
| 哈希索引 (Hash indexes)      | 无法做范围查询                                       | √      |        | √      |
| 全文索引 (Full-text indexes) |                                                      | √      | √      |        |


##### B Tree 原理

B-tree（多路搜索树，并不是二叉的）是一种常见的数据结构,也就是我们常说的B树。使用B-tree结构可以显著减少定位记录时所经历的中间过程，从而加快存取速度。按照翻译，B 通常认为是Balance的简称。这个数据结构一般用于数据库的索引，综合效率较高。

B-Tree:
<p align="center">
<img width="500" align="center" src="../images/10.jpg" />
</p>

定义一条数据记录为一个二元组 [key, data]，B-Tree 是满足下列条件的数据结构：

* 所有叶节点具有相同的深度，也就是说 B-Tree 是平衡的；
* 一个节点中的 key 从左到右非递减排列；
* 如果某个指针的左右相邻 key 分别是 keyi 和 keyi+1，且不为 null，则该指针指向节点的（所有 key ≥ keyi） 且（key ≤ keyi+1）。

查找算法：首先在根节点进行二分查找，如果找到则返回对应节点的 data，否则在相应区间的指针指向的节点递归进行查找。

由于插入删除新的数据记录会破坏 B-Tree 的性质，因此在插入删除时，需要对树进行一个分裂、合并、旋转等操作以保持 B-Tree 性质。

B+Tree:

<p align="center">
<img width="500" align="center" src="../images/11.jpg" />
</p>

与 B-Tree 相比，B+Tree 有以下不同点：

* 每个节点的指针上限为 2d 而不是 2d+1（d 为节点的出度）；
* 内节点不存储 data，只存储 key；
* 叶子节点不存储指针。

顺序访问指针:
<p align="center">
<img width="500" align="center" src="../images/12.jpg" />
</p>

一般在数据库系统或文件系统中使用的 B+Tree 结构都在经典 B+Tree 基础上进行了优化，在叶子节点增加了顺序访问指针，做这个优化的目的是为了提高区间访问的性能。

优势:

红黑树等平衡树也可以用来实现索引，但是文件系统及数据库系统普遍采用 B Tree 作为索引结构，主要有以下两个原因：

* 更少的检索次数

平衡树检索数据的时间复杂度等于树高 h，而树高大致为 O(h)=O(logdN)，其中 d 为每个节点的出度。

红黑树的出度为 2，而 B Tree 的出度一般都非常大。红黑树的树高 h 很明显比 B Tree 大非常多，因此检索的次数也就更多。

B+Tree 相比于 B-Tree 更适合外存索引，因为 B+Tree 内节点去掉了 data 域，因此可以拥有更大的出度，检索效率会更高。

* 利用计算机预读特性

为了减少磁盘 I/O，磁盘往往不是严格按需读取，而是每次都会预读。这样做的理论依据是计算机科学中著名的局部性原理：当一个数据被用到时，其附近的数据也通常会马上被使用。预读过程中，磁盘进行顺序读取，顺序读取不需要进行磁盘寻道，并且只需要很短的旋转时间，因此速度会非常快。

操作系统一般将内存和磁盘分割成固态大小的块，每一块称为一页，内存与磁盘以页为单位交换数据。数据库系统将索引的一个节点的大小设置为页的大小，使得一次 I/O 就能完全载入一个节点，并且可以利用预读特性，相邻的节点也能够被预先载入。

参考：[MySQL 索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

##### B+Tree 索引
B+Tree 索引是大多数 MySQL 存储引擎的默认索引类型。

因为不再需要进行全表扫描，只需要对树进行搜索即可，因此查找速度快很多。除了用于查找，还可以用于排序和分组。

可以指定多个列作为索引列，多个索引列共同组成键。

B+Tree 索引适用于全键值、键值范围和键前缀查找，其中键前缀查找只适用于最左前缀查找。

如果不是按照索引列的顺序进行查找，则无法使用索引。

InnoDB 的 B+Tree 索引分为**主索引**和**辅助索引**。

主索引的叶子节点 data 域记录着完整的数据记录，这种索引方式被称为聚簇索引。因为无法把数据行存放在两个不同的地方，所以一个表只能有一个聚簇索引。

<p align="center">
<img width="500" align="center" src="../images/13.jpg" />
</p>

辅助索引的叶子节点的 data域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后再到主索引中进行查找。
<p align="center">
<img width="500" align="center" src="../images/14.jpg" />
</p>

##### 哈希索引

InnoDB 引擎有一个特殊的功能叫 “自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。

哈希索引能以 O(1) 时间进行查找，但是失去了有序性，它具有以下限制：

* 无法用于排序与分组；
* 只支持精确查找，无法用于部分查找和范围查找；

##### 全文索引

MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。查找条件使用 MATCH AGAINST，而不是普通的 WHERE。

全文索引一般使用倒排索引实现，它记录着关键词到其所在文档的映射。

InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。

##### 空间数据索引（R-Tree）

MyISAM 存储引擎支持空间数据索引，可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用 GIS 相关的函数来维护数据。

##### 索引的特点

* 可以加快数据库的检索速度
* 降低数据库插入、修改、删除等维护的速度
* 只能创建在表上，不能创建到视图上
* 既可以直接创建又可以间接创建
* 可以在优化隐藏中使用索引
* 使用查询处理器执行SQL语句，在一个表上，一次只能使用一个索引

##### 索引的优点和缺点

索引的优点:
* 可以创建唯一性索引，保证数据库表中每一行数据的唯一性
* 大大加快数据的检索速度，这是创建索引的最主要的原因
* 加速数据库表之间的连接，特别是在实现数据的参考完整性方面特别有意义
* 在使用分组和排序子句进行数据检索时，同样可以显著减少查询中分组和排序的时间
* 通过使用索引，可以在查询中使用优化隐藏器，提高系统的性能

索引的缺点:
* 可以创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加
* 索引需要占用物理空间，除了数据表占用数据空间之外，每一个索引还要占一定的物理空间，如果建立聚簇索引，那么需要的空间就会更大
* 当对表中的数据进行增加、删除和修改的时候，索引也需要维护，降低数据维护的速度

##### 索引失效的情况

* 如果MySQL估计使用全表扫秒比使用索引快，则不适用索引。

如果列key均匀分布在1和100之间，下面的查询使用索引就不是很好：select * from table_name where key>1 and key<90;

* 如果条件中有or，即使其中有条件带索引也不会使用

select * from table_name where key1='a' or key2='b';如果在key1上有索引而在key2上没有索引，则该查询也不会走索引

* 复合索引，如果索引列不是复合索引的第一部分，则不使用索引（即不符合最左前缀）

复合索引为(key1,key2),则查询select * from table_name where key2='b';将不会使用索引

* 如果like是以 % 开始的，则该列上的索引不会被使用。

select * from table_name where key1 like '%a'；该查询即使key1上存在索引，也不会被使用如果列类型是字符串，那一定要在条件中使用引号引起来，否则不会使用索引

* 如果列为字符串，则where条件中必须将字符常量值加引号，否则即使该列上存在索引，也不会被使用。

select * from table_name where key1=1;如果key1列保存的是字符串，即使key1上有索引，也不会被使用。

* 如果使用MEMORY/HEAP表，并且where条件中不使用“=”进行索引列，那么不会用到索引，head表只有在“=”的条件下才会使用索引

#### 适合建立索引的情况

下面的查询适合建立索引：

* 如果SQL差选中出现在关键字order by、group by、distinct后面的字段，建立索引。
* 在union等集合操作的结果集字段上，建立索引。其建立索引的目的同上。
* 经常用作查询选择 where 后的字段，建立索引。
* 经常用作表连接 join 的属性上，建立索引。
* 考虑使用索引覆盖。对数据很少被更新的表，如果用户经常只查询其中的几个字段，可以考虑在这几个字段上建立索引，从而将表的扫描改变为索引的扫描。

##### 为何选择用B+树做索引而不用B-树或红黑树

B+ 树只有叶节点存放数据，其余节点用来索引，而 B- 树是每个索引节点都会有 Data 域。所以从 InooDB 的角度来看，B+ 树是用来充当索引的，一般来说索引非常大，尤其是关系性数据库这种数据量大的索引能达到亿级别，所以为了减少内存的占用，索引也会被存储在磁盘上。

MySQL如何衡量查询效率呢？

主要是通过磁盘 IO 次数。

* B- 树 / B+ 树 的特点就是每层节点数目非常多，层数很少，目的就是为了就少磁盘 IO 次数，但是 B- 树的每个节点都有 data 域（指针），这无疑增大了节点大小，说白了增加了磁盘 IO 次数（磁盘 IO 一次读出的数据量大小是固定的，单个数据变大，每次读出的就少，IO 次数增多，一次 IO 多耗时），而 B+ 树除了叶子节点其它节点并不存储数据，节点小，磁盘 IO 次数就少。

* B+ 树所有的 Data 域在叶子节点，一般来说都会进行一个优化，就是将所有的叶子节点用指针串起来。这样遍历叶子节点就能获得全部数据，这样就能进行区间访问啦。在数据库中基于范围的查询是非常频繁的，而 B 树不支持这样的遍历操作。

B 树和红黑树之间的区别:

* AVL 树和红黑树基本都是存储在内存中才会使用的数据结构。在大规模数据存储的时候，红黑树往往出现由于树的深度过大而造成磁盘 IO 读写过于频繁，进而导致效率低下的情况。之所以会出现这样的情况，是由于我们要获取磁盘上数据，需要先通过磁盘移动臂移动到数据所在的柱面，然后找到指定盘面，接着旋转盘面找到数据所在的磁道，最后对数据进行读写。磁盘IO代价主要花费在查找所需的柱面上，树的深度过大会造成磁盘IO频繁读写。根据磁盘查找存取的次数往往由树的高度所决定，所以，只要我们通过某种较好的树结构减少树的结构尽量减少树的高度，B树可以有多个子女，从几十到上千，可以降低树的高度。

* 数据库系统的设计者巧妙利用了磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次 I/O 就可以完全载入。为了达到这个目的，在实际实现 B-Tree 还需要在每次新建节点时，申请一个页的空间，这样就保证一个节点物理上也存储在一个页里，加之计算机存储分配都是按页对齐的，这样就实现了一个 node 只需一次 I/O。