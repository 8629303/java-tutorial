# MySQL读写分离

[![1570776205802](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/mysql-proxy.jpg)](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/mysql-proxy.jpg)

## 1、读写分离的介绍

[![img](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB.jpg)](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/读写分离.jpg)

 MySQL读写分离基本原理是让master数据库处理写操作，slave数据库处理读操作。master将写操作的变更同步到各个slave节点。

 MySQL读写分离能提高系统性能的原因在于：

 1、物理服务器增加，机器处理能力提升。拿硬件换性能。

 2、主从只负责各自的读和写，极大程度缓解X锁和S锁争用。

 3、slave可以配置myiasm引擎，提升查询性能以及节约系统开销。

 4、master直接写是并发的，slave通过主库发送来的binlog恢复数据是异步。

 5、slave可以单独设置一些参数来提升其读的性能。

 6、增加冗余，提高可用性。

## 2、读写分离的配置

1、硬件配置

```
master 192.168.85.11
slave  192.168.85.12
proxy  192，168.85.14
```

2、首先在master和slave上配置主从复制

3、进行proxy的相关配置

```shell
#1、下载mysql-proxy
https://downloads.mysql.com/archives/proxy/#downloads
#2、上传软件到proxy的机器
直接通过xftp进行上传
#3、解压安装包
tar -zxvf mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
#4、修改解压后的目录
mv mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit mysql-proxy
#5、进入mysql-proxy的目录
cd mysql-proxy
#6、创建目录
mkdir conf
mkdir logs
#7、添加环境变量
#打开/etc/profile文件
vi /etc/profile
#在文件的最后面添加一下命令
export PATH=$PATH:/root/mysql-proxy/bin
#8、执行命令让环境变量生效
source /etc/profile
#9、进入conf目录，创建文件并添加一下内容
vi mysql-proxy.conf
添加内容
[mysql-proxy]
user=root
proxy-address=192.168.85.14:4040
proxy-backend-addresses=192.168.85.11:3306
proxy-read-only-backend-addresses=192.168.85.12:3306
proxy-lua-script=/root/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua
log-file=/root/mysql-proxy/logs/mysql-proxy.log
log-level=debug
daemon=true
#10、开启mysql-proxy
mysql-proxy --defaults-file=/root/mysql-proxy/conf/mysql-proxy.conf
#11、查看是否安装成功，打开日志文件
cd /root/mysql-proxy/logs
tail -100 mysql-proxy.log
#内容如下：表示安装成功
2019-10-11 21:49:41: (debug) max open file-descriptors = 1024
2019-10-11 21:49:41: (message) proxy listening on port 192.168.85.14:4040
2019-10-11 21:49:41: (message) added read/write backend: 192.168.85.11:3306
2019-10-11 21:49:41: (message) added read-only backend: 192.168.85.12:3306
2019-10-11 21:49:41: (debug) now running as user: root (0/0)
```

4、进行连接

```shell
#mysql的命令行会出现无法连接的情况，所以建议使用客户端
mysql -uroot -p123 -h192.168.85.14 -P 4040
```

















# MySQL的锁机制

## 1、MySQL锁的基本介绍

 **锁是计算机协调多个进程或线程并发访问某一资源的机制。**在数据库中，除传统的 计算资源（如CPU、RAM、I/O等）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一 个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

 相对其他数据库而言，MySQL的锁机制比较简单，其最 显著的特点是不同的**存储引擎**支持不同的锁机制。比如，MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking）；InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁。

 **表级锁：**开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
​ **行级锁：**开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

 从上述特点可见，很难笼统地说哪种锁更好，只能就具体应用的特点来说哪种锁更合适！仅从锁的角度 来说：表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有 并发查询的应用，如一些在线事务处理（OLTP）系统。

## 2、MyISAM表锁

MySQL的表级锁有两种模式：**表共享读锁（Table Read Lock）**和 **表独占写锁（Table Write Lock）**

对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM表的读操作与写操作之间，以及写操作之间是串行的！

建表语句：

```sql
CREATE TABLE `mylock` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `NAME` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

INSERT INTO `mylock` (`id`, `NAME`) VALUES ('1', 'a');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('2', 'b');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('3', 'c');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('4', 'd');
```

**MyISAM写锁阻塞读的案例：**

 当一个线程获得对一个表的写锁之后，只有持有锁的线程可以对表进行更新操作。其他线程的读写操作都会等待，直到锁释放为止。

|                           session1                           |                       session2                       |
| :----------------------------------------------------------: | :--------------------------------------------------: |
|          获取表的write锁定 lock table mylock write;          |                                                      |
| 当前session对表的查询，插入，更新操作都可以执行 select * from mylock; insert into mylock values(5,'e'); | 当前session对表的查询会被阻塞 select * from mylock； |
|                   释放锁： unlock tables；                   |       当前session能够立刻执行，并返回对应结果        |

**MyISAM读阻塞写的案例：**

 一个session使用lock table给表加读锁，这个session可以锁定表中的记录，但更新和访问其他表都会提示错误，同时，另一个session可以查询表中的记录，但更新就会出现锁等待。

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|           获得表的read锁定 lock table mylock read;           |                                                              |
|     当前session可以查询该表记录： select * from mylock;      |     当前session可以查询该表记录： select * from mylock;      |
| 当前session不能查询没有锁定的表 select * from person Table 'person' was not locked with LOCK TABLES | 当前session可以查询或者更新未锁定的表 select * from mylock insert into person values(1,'zhangsan'); |
| 当前session插入或者更新表会提示错误 insert into mylock values(6,'f') Table 'mylock' was locked with a READ lock and can't be updated update mylock set name='aa' where id = 1; Table 'mylock' was locked with a READ lock and can't be updated | 当前session插入数据会等待获得锁 insert into mylock values(6,'f'); |
|                    释放锁 unlock tables;                     |                       获得锁，更新成功                       |

## 3、注意

**MyISAM在执行查询语句之前，会自动给涉及的所有表加读锁，在执行更新操作前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此用户一般不需要使用命令来显式加锁，上例中的加锁时为了演示效果。**

**MyISAM的并发插入问题**

MyISAM表的读和写是串行的，这是就总体而言的，在一定条件下，MyISAM也支持查询和插入操作的并发执行

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|     获取表的read local锁定 lock table mylock read local      |                                                              |
| 当前session不能对表进行更新或者插入操作 insert into mylock values(6,'f') Table 'mylock' was locked with a READ lock and can't be updated update mylock set name='aa' where id = 1; Table 'mylock' was locked with a READ lock and can't be updated |      其他session可以查询该表的记录 select* from mylock       |
| 当前session不能查询没有锁定的表 select * from person Table 'person' was not locked with LOCK TABLES | 其他session可以进行插入操作，但是更新会阻塞 update mylock set name = 'aa' where id = 1; |
|          当前session不能访问其他session插入的记录；          |                                                              |
|                  释放锁资源：unlock tables                   |               当前session获取锁，更新操作完成                |
|           当前session可以查看其他session插入的记录           |                                                              |

可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺：

```sql
mysql> show status like 'table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 352   |
| Table_locks_waited    | 2     |
+-----------------------+-------+
--如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。
```

**InnoDB锁**

**1、事务及其ACID属性**

事务是由一组SQL语句组成的逻辑处理单元，事务具有4属性，通常称为事务的ACID属性。

原子性（Actomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。
隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。
持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

**2、并发事务带来的问题**

相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多用户的并发操作，但与此同时，会带来一下问题：

**脏读**： 一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读”

**不可重复读**：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。

**幻读**： 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读”

上述出现的问题都是数据库读一致性的问题，可以通过事务的隔离机制来进行保证。

数据库的事务隔离越严格，并发副作用就越小，但付出的代价也就越大，因为事务隔离本质上就是使事务在一定程度上串行化，需要根据具体的业务需求来决定使用哪种隔离级别

|                  | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| read uncommitted |  √   |     √      |  √   |
|  read committed  |      |     √      |  √   |
| repeatable read  |      |            |  √   |
|   serializable   |      |            |      |

可以通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况：

```sql
mysql> show status like 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 18702 |
| Innodb_row_lock_time_avg      | 18702 |
| Innodb_row_lock_time_max      | 18702 |
| Innodb_row_lock_waits         | 1     |
+-------------------------------+-------+
--如果发现锁争用比较严重，如InnoDB_row_lock_waits和InnoDB_row_lock_time_avg的值比较高
```

**3、InnoDB的行锁模式及加锁方法**

 **共享锁（s）**：又称读锁。允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
​ **排他锁（x）**：又称写锁。允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。

 mysql InnoDB引擎默认的修改数据语句：**update,delete,insert都会自动给涉及到的数据加上排他锁，select语句默认不会加任何锁类型**，如果加排他锁可以使用select …for update语句，加共享锁可以使用select … lock in share mode语句。**所以加过排他锁的数据行在其他事务种是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但可以直接通过select …from…查询数据，因为普通查询没有任何锁机制。**

**InnoDB行锁实现方式**

 InnoDB行锁是通过给**索引**上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，**否则，InnoDB将使用表锁！**

1、在不通过索引条件查询的时候，innodb使用的是表锁而不是行锁

```sql
create table tab_no_index(id int,name varchar(10)) engine=innodb;
insert into tab_no_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                         session1                          |                        session2                         |
| :-------------------------------------------------------: | :-----------------------------------------------------: |
| set autocommit=0 select * from tab_no_index where id = 1; | set autocommit=0 select * from tab_no_index where id =2 |
|    select * from tab_no_index where id = 1 for update     |                                                         |
|                                                           |   select * from tab_no_index where id = 2 for update;   |

session1只给一行加了排他锁，但是session2在请求其他行的排他锁的时候，会出现锁等待。原因是在没有索引的情况下，innodb只能使用表锁。

2、创建带索引的表进行条件查询，innodb使用的是行锁

```sql
create table tab_with_index(id int,name varchar(10)) engine=innodb;
alter table tab_with_index add index id(id);
insert into tab_with_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                          session1                          |                         session2                         |
| :--------------------------------------------------------: | :------------------------------------------------------: |
| set autocommit=0 select * from tab_with_indexwhere id = 1; | set autocommit=0 select * from tab_with_indexwhere id =2 |
|    select * from tab_with_indexwhere id = 1 for update     |                                                          |
|                                                            |   select * from tab_with_indexwhere id = 2 for update;   |

3、由于mysql的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现冲突的。

```sql
alter table tab_with_index drop index id;
insert into tab_with_index  values(1,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                       set autocommit=0                       |                       set autocommit=0                       |
| select * from tab_with_index where id = 1 and name='1' for update |                                                              |
|                                                              | select * from tab_with_index where id = 1 and name='4' for update 虽然session2访问的是和session1不同的记录，但是因为使用了相同的索引，所以需要等待锁 |

## 4、总结

**对于MyISAM的表锁，主要讨论了以下几点：**
（1）共享读锁（S）之间是兼容的，但共享读锁（S）与排他写锁（X）之间，以及排他写锁（X）之间是互斥的，也就是说读和写是串行的。
（2）在一定条件下，MyISAM允许查询和插入并发执行，我们可以利用这一点来解决应用中对同一表查询和插入的锁争用问题。
（3）MyISAM默认的锁调度机制是写优先，这并不一定适合所有应用，用户可以通过设置LOW_PRIORITY_UPDATES参数，或在INSERT、UPDATE、DELETE语句中指定LOW_PRIORITY选项来调节读写锁的争用。
（4）由于表锁的锁定粒度大，读写之间又是串行的，因此，如果更新操作较多，MyISAM表可能会出现严重的锁等待，可以考虑采用InnoDB表来减少锁冲突。

**对于InnoDB表，本文主要讨论了以下几项内容：**
（1）InnoDB的行锁是基于索引实现的，如果不通过索引访问数据，InnoDB会使用表锁。
（2）在不同的隔离级别下，InnoDB的锁机制和一致性读策略不同。

在了解InnoDB锁特性后，用户可以通过设计和SQL调整等措施减少锁冲突和死锁，包括：

- 尽量使用较低的隔离级别； 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会；
- 选择合理的事务大小，小事务发生锁冲突的几率也更小；
- 给记录集显式加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁；
- 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大大减少死锁的机会；
- 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响； 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁；
- 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能。





# MySQL执行计划

 在企业的应用场景中，为了知道优化SQL语句的执行，需要查看SQL语句的具体执行过程，以加快SQL语句的执行效率。

 可以使用explain+SQL语句来模拟优化器执行SQL查询语句，从而知道mysql是如何处理sql语句的。

 官网地址： https://dev.mysql.com/doc/refman/5.5/en/explain-output.html

## 执行计划中包含的信息

|    Column     |                    Meaning                     |
| :-----------: | :--------------------------------------------: |
|      id       |            The `SELECT` identifier             |
|  select_type  |               The `SELECT` type                |
|     table     |          The table for the output row          |
|  partitions   |            The matching partitions             |
|     type      |                 The join type                  |
| possible_keys |         The possible indexes to choose         |
|      key      |           The index actually chosen            |
|    key_len    |          The length of the chosen key          |
|      ref      |       The columns compared to the index        |
|     rows      |        Estimate of rows to be examined         |
|   filtered    | Percentage of rows filtered by table condition |
|     extra     |             Additional information             |

## id

select查询的序列号，包含一组数字，表示查询中执行select子句或者操作表的顺序

id号分为三种情况：

 1、如果id相同，那么执行顺序从上到下

```sql
explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal;
```

 2、如果id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

```sql
explain select * from emp e where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
```

 3、id相同和不同的，同时存在：相同的可以认为是一组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行

```sql
explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
```

## select_type

主要用来分辨查询的类型，是普通查询还是联合查询还是子查询

| `select_type` Value  |                           Meaning                            |
| :------------------: | :----------------------------------------------------------: |
|        SIMPLE        |        Simple SELECT (not using UNION or subqueries)         |
|       PRIMARY        |                       Outermost SELECT                       |
|        UNION         |         Second or later SELECT statement in a UNION          |
|   DEPENDENT UNION    | Second or later SELECT statement in a UNION, dependent on outer query |
|     UNION RESULT     |                      Result of a UNION.                      |
|       SUBQUERY       |                   First SELECT in subquery                   |
|  DEPENDENT SUBQUERY  |      First SELECT in subquery, dependent on outer query      |
|       DERIVED        |                        Derived table                         |
| UNCACHEABLE SUBQUERY | A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query |
|  UNCACHEABLE UNION   | The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY) |

```sql
--sample:简单的查询，不包含子查询和union
explain select * from emp;

--primary:查询中若包含任何复杂的子查询，最外层查询则被标记为Primary
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--union:若第二个select出现在union之后，则被标记为union
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--dependent union:跟union类似，此处的depentent表示union或union all联合而成的结果会受外部表影响
explain select * from emp e where e.empno  in ( select empno from emp where deptno = 10 union select empno from emp where sal >2000)

--union result:从union表获取结果的select
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--subquery:在select或者where列表中包含子查询
explain select * from emp where sal > (select avg(sal) from emp) ;

--dependent subquery:subquery的子查询要受到外部表查询的影响
explain select * from emp e where e.deptno in (select distinct deptno from dept);

--DERIVED: from子句中出现的子查询，也叫做派生类，
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--UNCACHEABLE SUBQUERY：表示使用子查询的结果不能被缓存
 explain select * from emp where empno = (select empno from emp where deptno=@@sort_buffer_size);
 
--uncacheable union:表示union的查询结果不能被缓存：sql语句未验证
```

## table

对应行正在访问哪一个表，表名或者别名，可能是临时表或者union合并结果集
1、如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名

 2、表名是derivedN的形式，表示使用了id为N的查询产生的衍生表

 3、当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

## type

type显示的是访问类型，访问类型表示我是以何种方式去访问我们的数据，最容易想的是全表扫描，直接暴力的遍历一张表去寻找需要的数据，效率非常低下，访问的类型有很多，效率从最好到最坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

一般情况下，得保证查询至少达到range级别，最好能达到ref

```sql
--all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。
explain select * from emp;

--index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序
explain  select empno from emp;

--range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN() 
explain select * from emp where empno between 7000 and 7500;

--index_subquery：利用索引来关联子查询，不再扫描全表
explain select * from emp where emp.job in (select job from t_job);

--unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引
 explain select * from emp e where e.deptno in (select distinct deptno from dept);
 
--index_merge：在查询过程中需要多个索引组合使用，没有模拟出来

--ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式
explain select * from emp e where  e.mgr is null or e.mgr=7369;

--ref：使用了非唯一性索引进行数据的查找
 create index idx_3 on emp(deptno);
 explain select * from emp e,dept d where e.deptno =d.deptno;

--eq_ref ：使用唯一性索引进行数据查找
explain select * from emp,emp2 where emp.empno = emp2.empno;

--const：这个表至多有一个匹配行，
explain select * from emp where empno = 7369;
 
--system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现
```

## possible_keys

 显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

## key

 实际使用的索引，如果为null，则没有使用索引，查询中若使用了覆盖索引，则该索引和查询的select字段重叠。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

## key_len

表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，在不损失精度的情况下长度越短越好。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

## ref

显示索引的哪一列被使用了，如果可能的话，是一个常数

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

## rows

根据表的统计信息及索引使用情况，大致估算出找出所需记录需要读取的行数，此参数很重要，直接反应的sql找了多少数据，在完成目的的情况下越少越好

```sql
explain select * from emp;
```

## extra

包含额外的信息。

```sql
--using filesort:说明mysql无法利用索引进行排序，只能利用排序算法进行排序，会消耗额外的位置
explain select * from emp order by sal;

--using temporary:建立临时表来保存中间结果，查询完成之后把临时表删除
explain select ename,count(*) from emp where deptno = 10 group by ename;

--using index:这个表示当前的查询时覆盖索引的，直接从索引中读取数据，而不用访问数据表。如果同时出现using where 表名索引被用来执行索引键值的查找，如果没有，表面索引被用来读取数据，而不是真的查找
explain select deptno,count(*) from emp group by deptno limit 10;

--using where:使用where进行条件过滤
explain select * from t_user where id = 1;

--using join buffer:使用连接缓存，情况没有模拟出来

--impossible where：where语句的结果总是false
explain select * from emp where empno = 7469;
```





# MySQL事务测试

1、打开mysql的命令行，将自动提交事务给关闭

```sql
--查看是否是自动提交 1表示开启，0表示关闭
select @@autocommit;
--设置关闭
set autocommit = 0;
```

2、数据准备

```sql
--创建数据库
create database tran;
--切换数据库 两个窗口都执行
use tran;
--准备数据
create table psn(id int primary key,name varchar(10)) engine=innodb;
--插入数据
insert into psn values(1,'zhangsan');
insert into psn values(2,'lisi');
insert into psn values(3,'wangwu');
commit;
```

3、测试事务

```sql
--事务包含四个隔离级别：从上往下，隔离级别越来越高，意味着数据越来越安全
read uncommitted; 	--读未提交
read commited;		--读已提交
repeatable read;	--可重复读
(seariable)			--序列化执行，串行执行
--产生数据不一致的情况：
脏读
不可重复读
幻读
```

| 隔离级别 | 异常情况 |            | 异常情况 |
| -------- | -------- | ---------- | -------- |
| 读未提交 | 脏读     | 不可重复读 | 幻读     |
| 读已提交 |          | 不可重复读 | 幻读     |
| 可重复读 |          |            | 幻读     |
| 序列化   |          |            |          |

4、测试1：脏读 read uncommitted

```sql
set session transaction isolation level read uncommitted;
A:start transaction;
A:select * from psn;
B:start transaction;
B:select * from psn;
A:update psn set name='msb';
A:selecet * from psn
B:select * from psn;  --读取的结果msb。产生脏读，因为A事务并没有commit，读取到了不存在的数据
A:commit;
B:select * from psn; --读取的数据是msb,因为A事务已经commit，数据永久的被修改
```

5、测试2：当使用 read committed 的时候，就不会出现脏读的情况了，当时会出现不可重复读的问题

```sql
set session transaction isolation level read committed;
A:start transaction;
A:select * from psn;
B:start transaction;
B:select * from psn;
--执行到此处的时候发现，两个窗口读取的数据是一致的
A:update psn set name ='zhangsan' where id = 1;
A:select * from psn;
B:select * from psn;
--执行到此处发现两个窗口读取的数据不一致，B窗口中读取不到更新的数据
A:commit;
A:select * from psn;--读取到更新的数据
B:select * from psn;--也读取到更新的数据
--发现同一个事务中多次读取数据出现不一致的情况
```

6、测试3：当使用 repeatable read 的时候(按照上面的步骤操作)，就不会出现不可重复读的问题，但是会出现幻读的问题

```sql
set session transaction isolation level repeatable read;
A:start transaction;
A:select * from psn;
B:start transaction;
B:select * from psn;
--此时两个窗口读取的数据是一致的
A:insert into psn values(4,'sisi');
A:commit;
A:select * from psn;--读取到添加的数据
B:select * from psn;--读取不到添加的数据
B:insert into psn values(4,'sisi');--报错，无法插入数据
--此时发现读取不到数据，但是在插入的时候不允许插入，出现了幻读，设置更高级别的隔离级别即可解决
```



# 参考文献 & 鸣谢

1. [MySQL 高级-数据库优化](https://blog.csdn.net/qq_36903261/category_10394968.html)
1. [MySQL之SQL优化详解](https://www.cnblogs.com/dwlovelife/category/1486258.html)
2. [MySQL5.7版本数据库多实例安装部署](https://blog.csdn.net/u011208222/article/details/109217474)