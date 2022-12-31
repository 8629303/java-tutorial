# MySQL 数据库介绍与安装

## 1、MySQL各个版本介绍

各个版本的区别官网：https://dev.mysql.com/downloads/mysql/

- **MySQL Community Server** 社区版本，开源免费，但不提供官方技术支持。
- **MySQL Enterprise Edition** 企业版本，需付费，购买了之后可以电话支持
- **MySQL Cluster** 集群版，开源免费。可将几个MySQL Server封装成一个Server。



## 2、MySQL安装—win版

Mysql5.7-64位下载地址：https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.19-winx64.zip

1、下载后得到zip压缩包.

2、解压到自己想要安装到的目录，本人解压到的是D:\Environment\mysql-5.7.19

3、添加环境变量：我的电脑->属性->高级->环境变量->选择PATH,在其后面添加：mysql 安装文件下的bin文件夹

4、新建编辑 my.ini 文件 ,注意替换路径位置

```ini
[mysqld]
basedir=D:\Program Files\mysql-5.7\
datadir=D:\Program Files\mysql-5.7\data\
port=3306
skip-grant-tables
```

5、启动管理员模式下的CMD，并切换至mysql下的bin目录，将mysql 注册成服务

```shell
# mysqld --install 服务名称
mysqld --install mysql5.7.19
```

6、初始化数据文件输入(使用其中一个即可)：

```shell
# 方式一：初始化用户密码为空，同时生成Data文件夹
mysqld --initialize-insecure --user=mysql
# 方式二：会打印root用户的初始默认密码，同时并生成Data文件夹
mysqld --initialize --console
```

7、然后再次启动mysql 然后用命令 mysql –u root –p 进入mysql管理界面（密码可为空）

8、进入界面后更改root密码，然后刷新权限

```sql
-- 修改密码
update mysql.user set authentication_string=password('password') where user='root' and Host='localhost';

-- 刷新权限
flush privileges;
```

10、修改 my.ini文件删除最后一句skip-grant-tables

11、重启mysql即可正常使用

```shell
net stop mysql
net start mysql
```

12、删除服务命令

```shell
# mysqld --remove 服务名称
mysqld --remove mysql
```





## 3、MySQL安装—Linux版

> Linux Centos7安装mysql5.7详细教程(tar包安装)：https://blog.csdn.net/qq_37306041/article/details/89704783

历史版本下载地址：http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-5.7/

1、安装开发工具包和依赖包

```shell
# 安装开发工具包
$ yum groups mark install 'Development Tools'
$ yum -y install wget bzip2 bzip2-devel gcc gcc-c++

# 安装依赖包
$ yum -y install ncurses-devel openssl-devel openssl cmake mariadb-devel
```

2、为mysql创建用户和组

```shell
# 创建用户和组
$ groupadd -r -g 306 mysql
$ useradd -r -M -s /sbin/nologin -g mysql mysql
```

3、下载二进制格式的mysql软件包

```shell
# 下载二进制格式的mysql软件包
$ cd /usr/src/
$ wget https://downloads.mysql.com/archives/get/file/mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz
```

4、解压软件至/usr/local/

```shell
# 解压软件至/usr/local/
$ tar xf mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
$ cd /usr/local/

$ ln -s mysql-5.7.23-linux-glibc2.12-x86_64/ mysql
注：
1.ln软连接，-s是代号（symbolic）的意思；
2.使用方法：ln -s 源文件 目标文件；
3.我们需要在不同的目录，用到相同的文件时，我们只要在其它目录下用ln命令链接（link）它就可以，不必重复的占用磁盘空间。
```

5、修改目录/usr/local/mysql的属主属组,添加环境变量

```shell
# 修改目录/usr/local/mysql的属主属组,添加环境变量
# 注：-R处理指定目录以及其子目录下的所有文件
$ chown -R mysql.mysql /usr/local/mysql
$ echo 'export PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
$ ./etc/profile.d/mysql.sh 
```

6、建立数据存放目录

```shell
$ mkdir /opt/data
$ chown -R mysql.mysql /opt/data/
```

7、初始化数据库

```shell
$ /usr/local/mysql/bin/mysqld --initialize --user=mysql --datadir=/opt/data/
```

8、配置mysql

```shell
$ ln -sv /usr/local/mysql/include/ /usr/local/include/mysql
‘/usr/local/include/mysql’ -> ‘/usr/local/mysql/include/’

$ echo '/usr/local/mysql/lib' > /etc/ld.so.conf.d/mysql.conf
$ ldconfig -v
```

9、生成配置文件

```shell
[root@zzg local]# cat > /etc/my.cnf <<EOF
> [mysqld]
> basedir = /usr/local/mysql
> datadir = /opt/data
> socket = /tmp/mysql.sock
> port = 3306
> pid-file = /opt/data/mysql.pid
> user = mysql
> skip-name-resolve
> EOF
```

10、配置服务启动脚本

```shell
$ cp -a /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
$ sed -ri 's#^(basedir=).*#\1/usr/local/mysql#g' /etc/init.d/mysqld
$ sed -ri 's#^(datadir=).*#\1/opt/data#g' /etc/init.d/mysqld
```

11、启动

```shell
$ service mysqld start
```

12、修改密码

```shell
mysql> set password = password(123456');
```



> mysql的核心知识之服务管理

简介：mysql的service服务管理与登录管理

```
查看mysql服务进程：ps -ef | grep mysql service
服务管理：cp -a mysql.server /etc/rc.d/init.d/mysql
```

```
启动命令：service mysql start
关闭命令：service mysql stop
重新启动命令：service mysql restart
查看状态命令：service mysql status
```

```
登录管理：ln -s /usr/local/mysql/bin/*  /bin 
登录命令：mysql -uroot -p
```

```
默认端口号：3306 
配置文件：/etc/my.cnf
```





## 4、MySQL安装—Docker版

1、Docker 运行 mysql5.7

```shell
docker run -itd -p 3306:3306 --name mysql5.7 -e MYSQL_ROOT_PASSWORD=password mysql:5.7

docker run -itd -p 3306:3306 
 --name mysqltest1 \
 -e MYSQL_USER="dev" \
 -e MYSQL_PASSWORD="password" \
 -e MYSQL_ROOT_PASSWORD="password" 
 -v /mydata/mysql/log/:/var/log/mysql \
 -v /mydata/mysql/data:/var/lib/mysql \
 -v /mydata/mysql/conf:/etc/mysql \
 --character-set-server=utf8 \
 --collation-server=utf8_general_ci \
 mysql:5.7 

# 对上述参数进行解释：
-e MYSQL_USER=""  ：设置用户名
-e MYSQL_PASSWORD=""：设置该用户的密码
-e MYSQL_ROOT_PASSWORD=""：设置root用户密码
--character-set-server=utf8：设置字符集为utf8
--collation-server=utf8_general_cli：设置字符规则为utf8_general_cli
--privileged=true：容器内的root拥有真正root权限，否则容器内root只是外部普通用户权限
```

2、Docker 运行 mysql8

```shell
# Docker 运行 mysql8
################# 方式一 #####################
docker run -p 3306:3306 --name mysql8 \
 -v ~/mysql/conf:/etc/mysql/conf.d \
 -v ~/mysql/logs:/logs \
 -v ~/mysql/data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=root \
 -d mysql:8
################# 方式二 #####################
docker run -d -p 3306:3306 
  --restart always 
  --privileged=true
  --name mysql
  -e MYSQL_USER="test" 
  -e MYSQL_PASSWORD="test" 
  -e MYSQL_ROOT_PASSWORD="root"
  -v /data/docker/mysql/data/conf:/etc/my.cnf 
  -v /data/docker/mysql/data/db:/var/lib/mysql 
  mysql:8
  --default-authentication-plugin=mysql_native_password

## 参数说明
 –restart always  开机启动
 -e MYSQL_USER=”test”  添加用户test
 -e MYSQL_PASSWORD=”test”  设置test的密码为test
 -e MYSQL_ROOT_PASSWORD=”root” 设置root的密码为test
 –privileged=true  提升容器内权限(如果没有添加--privileged=true参数，容器创建后可能不能正常启动)
 -v /data/docker/mysql/data/conf:/etc/my.cnf  映射配置文件
 -v /data/docker/mysql/data/db:/var/lib/mysql  映射数据目录
 --default-authentication-plugin=mysql_native_password  更改默认的身份认证插件

######################################

# 进入mysql镜像
docker exec -it mysql8 /bin/bash
# 进入mysql命令行
mysql -uroot -p
# 设置远程访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
# Navicat 连接MySQL 8.0.11 出现2059错误,如下设置可以解决此错误
# 更改加密规则(mysql8需要设置)
ALTER USER 'root'@'%' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
# 更新root用户密码
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root'; 
# 刷新权限
FLUSH PRIVILEGES;
```

3、Docker之mysql容器中文输入：

```shell
# docker之mysql容器中文输入：
docker run -itd mysql env LANG=C.UTF-8
docker exec -it mysql env LANG=C.UTF-8 /bin/bash
```

4、docker-compose方式安装mysql8.0

创建 docker-compose.yml文件

```yaml
version: '3'
services:
  db:
    #构建mysql镜像
    image: mysql
    container_name: mysql # 容器名
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root #root管理员用户密码
      MYSQL_USER: test   #创建test用户
      MYSQL_PASSWORD: test  #设置test用户的密码
    ports:
    - '3306:3306'  #host物理直接映射端口为6606
    volumes:
    #mysql数据库挂载到host物理机目录
    - "/data/docker/mysql/data/db:/var/lib/mysql"
    #容器的配置目录挂载到host物理机目录
    - "/data/docker/mysql/data/conf:/etc/mysql/conf.d"
```

在 docker-compose.yml文件下的目录下，执行下面命令

```sh
docker-compose up
```



## 5、MySQL卸载—Linux版

> 参考：https://www.cnblogs.com/pengcode/p/13902639.html

1、查看mysql的安装情况

```shell
rpm -qa | grep -i mysql
```

[![1570605325400](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/1570605325400.png)](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/1570605325400.png)

2、删除上图安装的软件

```shell
rpm -ev mysql-community-libs-5.7.27-1.el6.x86_64 --nodeps
```

3、都删除成功之后，查找相关的mysql的文件[#](https://www.cnblogs.com/pengcode/p/13902639.html#3、都删除成功之后，查找相关的mysql的文件)

```shell
find / -name mysql
```

[![1570605553095](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/1570605553095.png)](https://images-1300356155.cos.ap-nanjing.myqcloud.com/img/1570605553095.png)

4、删除全部文件

```shell
rm -rf /var/lib/mysql
rm -rf /var/lib/mysql/mysql
rm -rf /etc/logrotate.d/mysql
rm -rf /usr/share/mysql
rm -rf /usr/bin/mysql
rm -rf /usr/lib64/mysql
```

5、再次执行命令

```shell
# 如果没有显式则表示卸载完成
rpm -qa | grep -i mysql
```





## 6、数据库三大设计范式

第一范式（1NF）：

1. 不可分割性，只要字段值还可以继续拆分，就不满足第一范式。
2. 不可分割的意思就按字面理解就是最小单位，不能再分成更小单位了。
3. 字段只能是一个值，不能被拆分成多个字段，否则的话，它就是可分割的，就不符合一范式。

第二范式（2NF）：

1. 第二范式就是要有主键，要求其他字段都依赖于主键。
2. 为什么要有主键？没有主键就没有唯一性，没有唯一性在集合中就定位不到这行记录，所以要主键。
3. 在满足第一范式的前提下，主键外的每一列都必须完全依赖于主键。如果出现不完全依赖，只可能发生在联合主键的情况下。

第三范式（3NF）：

1. 在满足第二范式的前提下，除了主键列之外，其他列之间不能有传递依赖关系，即“消除冗余”。
2. 消除冗余，就是各种信息只在一个地方存储，不出现在多张表中。
3. 范式总结：范式，其实是用来学习参考的，设计的时候根据情况，未必一定要遵守，要灵活结合业务实际情况决定。



## 7、MySQL操作语句分类

| 名称                                           | 解释                 | 命令                    |
| ---------------------------------------------- | -------------------- | ----------------------- |
| DDL 数据定义语言 (Data Definition Language)    | 建库、建表           | CREATE、DROP、ALTER     |
| DML 数据操纵语言(Data Manipulation Language)   | 表中数据的增删改操作 | INSERT、UPDATE、DELETE  |
| DQL 数据查询语言(Data Query Language)          | 对数据进行查询       | SELECT                  |
| DCL 数据控制语言(Data Control Language)        | 对用户的权限进行设置 | GRANT、COMMIT、ROLLBACK |
| TCL 事务控制语句(Transaction Control Language) | 对数据一致性的处理   | BEGIN、COMMIT、ROLLBACK |

# DDL 数据定义语言

## 1、数据库基本操作

> 数据库的创建使用：创建/查看/使用/切换
>

1、创建数据库语法：

```sql
CREATE DATABASE [IF NOT EXISTS] <数据库名> 
[ [ DEFAULT ] CHARACTER SET <字符集名> ] 
[ [ DEFAULT ] COLLATE <校对规则名> ];
```

```sql
-- 直接创建数据库 db1，如该库已存在报错：ERROR 1007 (HY000): Can't create database 'test_db'; database exists
create database db1;

-- 判断是否存在，如果不存在则创建数据库 db2
create database if not exists db2;

-- 创建数据库并指定字符集为gbk
create database db3 default character set gbk;

-- 创建数据库并指定字符集为utf8，校对规则为utf8_chinese_ci(简体中文，不区分大小写)
create database db4 default character set utf8 default collate utf8_chinese_ci;
```



2、修改数据库语法：

```sql
ALTER DATABASE [数据库名] {
    [ DEFAULT ] CHARACTER SET <字符集名> |
    [ DEFAULT ] COLLATE <校对规则名>
}
```

```sql
-- 修改数据库db1的指定字符集修改为 gb2312，默认校对规则修改为：gb2312_unicode_ci
alter database db1 default character set gb2312 default collate gb2312_chinese_ci;
```



3、删除数据库语法：

```sql
DROP DATABASE [ IF EXISTS ] <数据库名>
```

```sql
-- 删除数据库，如何该库不存在会报错：ERROR 1008 (HY000): Can't drop database 'test_db_del'; database doesn't exist
drop database db1;

-- 删除数据库，如果存在在则删除，不存在也不会报错
drop database if exists db1;
```



4、选择切换数据库语法：

```sql
-- 选择（进入）数据库
USE <数据库名>
```

```sql
-- 选择或进入db1数据库操作
use db1;
```



5、查看数据库的语法：

```sql
-- 显示数据库
SHOW DATABASES [LIKE '数据库名'];

-- 查看当前在哪个数据库内
select database();
```

```sql
-- 显示所有数据
show databases;

-- 查询显示某个数据库
show databases like 'mysql';
```

查看表可以发现系统默认会有如下4个库：

1. information_schema：主要存储了系统中的一些数据库对象信息。比如用户表信息、列信息、权限信息、字符集信息、分区信息等
2. cluster：存储了系统的集群信息
3. mysql：存储了系统的用户权限信息
4. test：系统自动创建的测试数据库，任何用户都可以使用





6、其他数据库命令操作：

```sql
-- 查看某个库是什么字符集；
show create database db1; 

-- 查看当前mysql使用的字符集
show variables like 'character%';

# 显示MYSQL的版本
mysql> select version(); 
+-----------+
| version() |
+-----------+
| 5.7.29    |
+-----------+

# 显示当前时间
mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2021-03-21 15:06:11 |
+---------------------+

# 显示年月日
mysql> SELECT DAYOFMONTH(CURRENT_DATE), MONTH(CURRENT_DATE), YEAR(CURRENT_DATE); 
+--------------------------+---------------------+--------------------+
| DAYOFMONTH(CURRENT_DATE) | MONTH(CURRENT_DATE) | YEAR(CURRENT_DATE) |
+--------------------------+---------------------+--------------------+
|                       21 |                   3 |               2021 |
+--------------------------+---------------------+--------------------+
```



## 2、常用数据类型

> https://www.cnblogs.com/xiaoming521/p/12622746.html

```sql
<1>整数型
     类型      大小      范围（有符号）               范围（无符号unsigned）    用途
     TINYINT   1 字节    (-128，127)                (0，255)                 小整数值
     SMALLINT  2 字节    (-32768，32767)            (0，65535)               大整数值
     MEDIUMINT 3 字节    (-8388608，8388607)        (0，16777215)            大整数值
     INT       4 字节    (-2147483648，2147483647)  (0，4294967295)          大整数值
     BIGINT    8 字节    (-2的63次方，-2的63次方减1)  (0，2的64次方减1)          极大整数值

<2>浮点型
 FLOAT(m,d）  4 字节    单精度浮点型  备注：m代表总个数，d代表小数位个数
 DOUBLE(m,d） 8 字节    双精度浮点型  备注：m代表总个数，d代表小数位个数
 
 <3>定点型
 DECIMAL(m,d）    依赖于M和D的值,如果M>D，为M+2否则为D+2字节   备注：m代表总个数，d代表小数位个数
 
 <4>字符串类型 
 类型          大小              用途
 CHAR          0-255字节         定长字符串
 VARCHAR       0-65535字节       变长字符串
 TINYTEXT      0-255字节         短文本字符串
 TEXT          0-65535字节       长文本数据
 MEDIUMTEXT    0-16777215字节    中等长度文本数据
 LONGTEXT      0-4294967295字节  极大文本数据
 
 char的优缺点：存取速度比varchar更快，但是比varchar更占用空间
 varchar的优缺点：比char省空间。但是存取速度没有char快
 
 <5>时间型
 数据类型    字节数          格式                    备注
 date        3             yyyy-MM-dd             存储日期值
 time        3             HH:mm:ss               存储时分秒
 year        1             yyyy                   存储年
 datetime    8             yyyy-MM-dd HH:mm:ss    存储日期+时间
 timestamp   4             yyyy-MM-dd HH:mm:ss    存储日期+时间，可作时间戳(根据时区来显示)
```

- 时间类型：

```sql
create table test_time (
    date_value date,
    time_value time,
    year_value year,
    datetime_value datetime,
    timestamp_value timestamp
) engine=innodb charset=utf8;
-- 插入时间数据
insert into test_time values(now(), now(), now(), now(), now());

mysql> select * from test_time;
+------------+------------+------------+---------------------+---------------------+
| date_value | time_value | year_value | datetime_value      | timestamp_value     |
+------------+------------+------------+---------------------+---------------------+
| 2020-07-26 | 14:19:44   |       2020 | 2020-07-26 14:19:44 | 2020-07-26 14:19:44 |
+------------+------------+------------+---------------------+---------------------+
```

- **Decima（定点型）和 Float/Double（浮点型）区别：**

其区别在于：float，double等非标准类型，在DB中保存的是近似值，而Decimal则以字符串的形式保存数值。

float有个坏处，当你给定的数据是整数的时候，那么它就以整数给你处理。这样我们在存取货币值的时候自然遇到问题，我的default值为：0.00而实际存储是0，同样我存取货币为12.00，实际存储是12。

幸好mysql提供了两个数据类型：decimal，这种数据类型可以轻松解决上面的问题：decimal类型被 MySQL 以同样的类型实现，这在 SQL92 标准中是允许的。他们用于保存对准确精度有重要要求的值，例如与金钱有关的数据。

```sql
create table t1(
    field_float float(10,2) default null, 
    field_decimal decimal(10,2) default null
);
insert into t1 values
(1234567.21, 1234567.21),
(9876543.21, 9876543.12),
(98765430.21, 98765430.12);

mysql> select * from t1;
+-------------+---------------+
| field_float | field_decimal |
+-------------+---------------+
|  1234567.25 | 1234567.21    |
|  9876543.00 | 9876543.12    |
| 98765432.00 | 98765430.12   |
+-------------+---------------+
浮点型float：
1234567.21 出现偏差：1234567.25
9876543.21 出现偏差：9876543.00
98765430.21出现偏差：98765432.00


create table t2(
    field_float float(5,2) default null,
    field_double double(5,2) default null,
    field_decimal decimal(5,2) default null
);
insert into t2 values (1.2345,1.2345,1.2345);
insert into t2 values (12.345,12.345,12.345);
insert into t2 values (123.45,123.45,123.45);
insert into t2 values (1234.5,1234.5,1234.5);
insert into t2 values (123.4,123.4,123.4);

mysql> select * from t2;
+-------------+--------------+---------------+
| field_float | field_double | field_decimal |
+-------------+--------------+---------------+
|        1.23 |         1.23 | 1.23          |
|       12.35 |        12.35 | 12.35         |
|      123.45 |       123.45 | 123.45        |
|      999.99 |       999.99 | 999.99        |
|      123.40 |       123.40 | 123.40        |
+-------------+--------------+---------------+

1.2345 --- 小数点后最多2位，所以保存可以，自动四舍五入数据截断,输出: 1.23
12.345 --- OK,小数位超1为，由于超出位为5，自动四舍五入，最后得出为 12.35
123.45 --- 正好符合3个整数位和2个小数位
1234.5 --- 因为小数位未满2位,要补0.所以应该为1234.50,但是整个位数超出了5,保存报错,输出999.99
123.4 ---  小数未满部分补0。按照123.40保存
```

**默认状态比较：**

浮点数如果不写经度和标度，会按照实际精度值保存，如果有精度和标度，则会自动将四舍五入后的结果插入，系统不会报错；定点数如果不写精度和标度，则按照默认值decimal(10,0) 来操作，如果数据超过了精度和标度值，系统会报错。



## 3、创建数据表

简介：讲解表是怎么来创建的，以及常见约束条件举例说明

建表语法：

```sql
-- 直接创建新表
CREATE TABLE 表名 (
    字段名1 字段类型1 约束条件1 说明1,
    字段名2 字段类型2 约束条件2 说明2,
    ....
);
```

复制表方式建表：

```sql
-- 1.1.只复制表结构到新表,注意：这种方式用于日常测试，因为可能索引等会复制不过来
create table 新表名 as select * from 旧表名 where 1=2;

-- 1.2.复制表结构及数据到新表
create table 新表名 as select * from 旧表名

-- 2.复制表结构及数据到新表
create table 新表名 like 旧表名;
```

- 约束条件

```sql
comment         ----说明解释
not null        ----不为空
default         ----默认值
unsigned        ----无符号（即正数）
auto_increment  ----自增
zerofill        ----自动填充,不足位数的用0来填充,如 int(3)，5则为005
unique key      ----唯一值
```

- 测试

```sql
create table student (
    id tinyint(5)  auto_increment  default null comment '学生学号',
    name varchar(20) default null comment '学生姓名',
    age  tinyint  default null comment '学生年龄',
    class varchar(20) default null comment '学生班级',
    sex char(5) not null comment '学生性别',
    unique key (id)
)engine=innodb charset=utf8;

-- 某个公司的员工表
CREATE TABLE employee(
    empno       INT PRIMARY KEY comment '雇员编号',
    ename       VARCHAR(20) comment '雇员姓名',
    job         VARCHAR(20) comment '雇员职位',
    mgr         INT comment '雇员上级编号',
    hiredate    DATE comment '雇佣日期',
    sal         DECIMAL(7,2) comment '薪资',
    deptnu      INT comment '部门编号'
);
```



## 4、查看表结构

查看表的基本结构信息的语句：

1. 查看数据库中所有表：show tables；
2. 查看建表的SQL语句：show create table 表名;
3. 查看数据表结构：desc 表名;
4. 查看表信息：show table status
5. \G ：有结束SQL语句的作用，还有把显示的数据纵向旋转90度，替换“ ; ”
6. \g ：有结束SQL语句的作用，等同 “ ; ”

```sql
-- 查看数据库中所有表：
show tables;

-- 查看建表的SQL语句：
show create table 表名;

-- 查看数据表结构：
desc 表名;

-- 查看表信息：db_name表示查询哪个数据库下面的表信息。[LIKE 'pattern'] 表示查询哪些具体的表名
show table status [FROM db_name] [LIKE 'pattern']
```

查看表结构和定义操作示例：

```sql
-- 查看数据库中的所有表
mysql> show tables;
+---------------------+
| Tables_in_jdbcstudy |
+---------------------+
| t1                  |
| t2                  |
| test_time           |
| student             |
+---------------------+
4 rows in set (0.04 sec)

-- 查看建表的SQL语句：
mysql> show create table student;
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                                                                                                                                                                                                                                                                                                           |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| student | CREATE TABLE `student` (
  `id` tinyint(5) NOT NULL AUTO_INCREMENT COMMENT '学生学号',
  `name` varchar(20) DEFAULT NULL COMMENT '学生姓名',
  `age` tinyint(4) DEFAULT NULL COMMENT '学生年龄',
  `class` varchar(20) DEFAULT NULL COMMENT '学生班级',
  `sex` char(5) NOT NULL COMMENT '学生性别',
  UNIQUE KEY `id` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8                     |
+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- \G：有结束sql语句的作用，把显示的数据纵向旋转90度，替换;
mysql> show create table student\G
*************************** 1. row ***************************
       Table: student
Create Table: CREATE TABLE `student` (
  `id` tinyint(5) NOT NULL AUTO_INCREMENT COMMENT '学生学号',
  `name` varchar(20) DEFAULT NULL COMMENT '学生姓名',
  `age` tinyint(4) DEFAULT NULL COMMENT '学生年龄',
  `class` varchar(20) DEFAULT NULL COMMENT '学生班级',
  `sex` char(5) NOT NULL COMMENT '学生性别',
  UNIQUE KEY `id` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

-- 查看表结构
mysql> desc student;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | tinyint(5)  | NO   | PRI | NULL    | auto_increment |
| name  | varchar(20) | YES  |     | NULL    |                |
| age   | tinyint(4)  | YES  |     | NULL    |                |
| class | varchar(20) | YES  |     | NULL    |                |
| sex   | char(5)     | NO   |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)

-- \g ：有结束sql语句的作用，等同 ;
desc test_time\g
-- 等同于
desc test_time;
```

查看表信息操作示例：

```sql
-- 查看当前库所有表的信息
show table status\G

-- 查询db_name数据库里所有表的信息
show table status from db_name\G

-- 查询db_name里table_name表的信息
show table status from db_name like 'table_name '\G

-- 查询db_name数据库里表名以tb_开头的表的信息
show table status from db_name LIKE 'tb_%'\G

-- 查询当前库student表的信息
mysql> show table status like 'student' \G
*************************** 1. row ***************************
           Name: student
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 1
    Create_time: 2021-09-16 12:24:45
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)
```

输出字段说明：

```
Name：表名称
Engine：表的存储引擎
Version：版本
Row_format：行格式。对于MyISAM引擎，这可能是Dynamic，Fixed或Compressed。动态行的行长度可变，例如Varchar或Blob类型字段。固定行是指行长度不变，例如Char和Integer类型字段
Rows：表中的行数。对于MyISAM和其他存储引擎，这个值是精确的，对于innoDB存储引擎，这个值通常是估算的
Avg_row_length：平均每行包括的字节数 
Data_length：整个表的数据量(以字节为单位)
Max_data_length：表可以容纳的最大数据量，该值和存储引擎相关
Index_length：索引占用磁盘的空间大小(以字节为单位)
Data_free：对于MyISAM引擎，表示已经分配，但目前没有使用的空间。这部分空间包含之前被删除的行，以及后续可以被insert利用到的空间
Auto_increment：下一个Auto_increment的值
Create_time：表的创建时间
Update_time：表的最近更新时间
Check_time：使用 check table 或myisamchk工具最后一次检查表的时间
Collation：表的默认字符集和字符排序规则
Checksum：如果启用，保存的是整个表的实时校验和
Create_options：创建表时指定的其他选项
Comment：包含了其他额外信息，对于MyISAM引擎，保存的是表在创建时带的注释。如果表使用的是innodb引擎 ，保存的是InnoDB表空间的剩余空间。如果是一个视图，注释里面包含了VIEW字样。
```



## 5、删除数据表

```sql
-- 删除表语法：
drop table [IF EXISTS] 表名1 [, 表名2, 表名3...]

-- 如删除不存在的数据表会抛出错误
drop table user;
-- 看表是否存在，若存在则删除表：
drop table if exists teacher;
```



## 6、修改数据表

对于已经创建好的表，尤其是已经有大量数据的表，如果需要对表做一些结构上的改变，我们可以先将表删除（drop），然后再按照新的表定义重建表。这样做没有问题，但是必然要做一些额外的工作，比如数据的重新加载。而且，如果有服务在访问表，也会对服务产生影响。因此，在大多数情况下，表结构的更改一般都使用：alter table 语句，以下是一些常用的命令。

1、修改表类型语法

```sql
-- 简单常用语法
ALTER TABLE 表名 MODIFY [COLUMN] column_definition [FIRST | AFTER col_name]

-- 完整语法
ALTER TABLE <表名> { 
    ADD COLUMN <列名> <类型> [FIRST | AFTER col_name] | 
    CHANGE COLUMN <旧列名> <新列名> <新列类型> | 
    ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT } | 
    MODIFY COLUMN <列名> <类型> | 
    DROP COLUMN <列名> | 
    RENAME TO <新表名> | 
    CHARACTER SET <字符集名> | 
    COLLATE <校对规则名> 
}
```

2、添加表字段（增加列）

```sql
-- 给表添加一列语法：（注意：没有给表某个字段前添加一列的说法）
alter table 表名 ADD [COLUMN] [FIRST | AFTER col_name];

-- 给student表增加一个字段（列），默认增加在最后
alter table 表名 add 列名 类型 comment '说明';
alter table student add famliy varchar(50) comment '学生父母';

-- 给表最前面添加一列：
alter table 表名 add 列名 类型 first;
alter table student add job varchar(10) first;

-- 给表某个字段后添加一列：
alter table 表名 add 列名 类型 after 字段名;
alter table student add servnumber int(11) after id;

-- 注意：没有给表某个字段前添加一列的说法
```

3、删除表字段（删除列）

```sql
-- 删除表字段语法：
alter table 表名 drop 列名;
-- 删除famliy字段
alter table student drop famliy;
```

4、修改表字段类型

```sql
-- 语法
alter table 表名 modify [COLUMN] column_definition [FIRST | AFTER col_name]
-- 操作示例：
alter table 表名 modify 列名 新类型 ;
alter table student modify servnumber varchar(20);
```

5、修改表字段名称（或类型）

```sql
-- 语法
alter table 表名 change [COLUMN] old_col_name column_definition [FIRST|AFTER col_name]

-- 修改列明
alter table 表名 change 旧列名 新列名 类型;
alter table user change servnumber telephone varchar(20);
```

PS：change 和 modify 都可以修改表的定义，不同的是 change 后面需要写两次列名，不方便。但是 change 的优点是可以修改列名称，modify 则不能

6、修改字符集

```sql
-- 语法
alter table 表名 character set 字符集名;
-- 示例
alter table user character set GBK;
```

7、表名修改

```sql
-- 方式一语法：
rename table 旧表名 to 新表名;
-- 方式二语法：
alter table 表名 rename [TO | AS] 新表名;

-- 操作示例：
mysql> rename table student to tb_student;
Query OK, 0 rows affected (0.01 sec)

mysql> alter table tb_student rename to student;
Query OK, 0 rows affected (0.01 sec)
```



# DML 数据操纵语言

## 1、数据新增

数据插入语法：

```sql
-- 完整版插入，可以选择对于的列插入数据
insert into 表名 (列名1,列名2...) [value | values] (值1,值2...);

-- 所有字段都插入时可以省略字段名
insert into 表名 [value | values] (所有列对应值);

 -- 插入单条数据
insert into 表名 set 列名 = 值，列名 = 值

-- 插入多条数据
insert into 表名 (列名1,列名2) values (值1,值2),(值3.值4),(值5,值6);
insert into 表名 values (所有列对应值),(所有列对应值),(所有列对应值);

-- 蠕虫复制（将一张表的数据复制到另一张表中，简称：表数据复制）PS：两个表的字段需要一直，并尽量保证要新增的表中没有数据

insert into 表名1 select * from 表名2;
insert into 表名1 (列名1,列名2) select 列名1,列名2 from 表名2;

-- 建表复制（新建表的时候直接复制另一张表的结构和数据，可以说处理表明不一样，其他都一样）
create table 表名1 as select 字段名1，字段名2 from 表名2;
```

操作示例1：普通的插入表数据

```sql
-- 选择字段插入数据
insert into student (id, name, age, class, sex) value (1, "Sam", 18, "Java", 1);

-- 所有字段都插入时可以省略字段名
insert into values (1, "Sam", 18, "Java", 1); 
```

操作示2：例蠕虫复制（将一张表的数据复制到另一张表中）

```sql
-- 将student表的数据的id, name字段的数据复制到student_01表中（PS：前提student_01表必须存在）
insert into student_01 (id, name) select id, name from student;

-- 将student表的数据复制到student_01表中（PS：前提student_01表必须存在）
insert into student_01 select * from student;
```

操作示例3：建表复制

```sql
mysql> create table student02 as select * from student;
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0
```

操作示例4：一次性插入多个数据

```sql
insert into student (id, name, age, class, sex) values (1, "Sam", 18, "Java", 1), (2, "Tom", 18, "Go", 1);
insert into student values (1, "Sam", 18, "Java", 1), (2, "Tom", 18, "Go", 1), (3, "mier", 16 "Python", 0);
```



## 2、数据更新

Update 语句一共有两种语法，分别用于更新单表数据和多表数据

修改更新数据语法：

```sql
-- 修改一条数据或多条数据的某个或某几个字段。PS：没有 where 条件的 update 会更新所有值
update 表名 set 字段名 = 值 where 字段名 = 值
update 表名 set 字段名1 = 值1, 字段名2 = 值2 where 字段名 = 值;

-- 修改多条数据的字段为同一的值
update 表名 set 字段名= 值 where 字段名 in ('值1','值2','值3');

-- 使用case when实现批量更新
update 表名 set 字段名1 = 
  case 字段名2 
    when 值1 then '值' 
    when 值2 then '值' 
    when 值3 then '值' 
  end 
  where s_id in (值1,值2,值3)
```

操作示例：

```sql
-- 修改一条数据的某个字段
update student set name ='Sam_liu' where id = 1

-- 修改3条数据的age为18
update student set age = 18 where id in (1, 2, 3);

-- 修改name字段，如果id的值为1,则name的值为:小王，id = 2 则 name = 小周，id = 3 则 name = 老周
update student set name = case id when 1 then '小王' when 2 then '小周' when 3 then '老周' end where id in (1,2,3)

-- PS：这里where不影响代码的执行，但是会提高sql执行的效率。确保sql语句仅执行需要修改的行数，这里只有3条数据进行更新，而where子句确保只有3行数据执行
```





## 3、数据删除

数据库一旦删除数据，它就会永远消失。 因此，在执行DELETE语句之前，应该先备份数据库，以防万一要找回删除过的数据。

1、删除指定数据。语法：

```sql
--  删除指定数据
delete from 表名 where 字段名=值;
```

操作示例：

```sql
delete from student where id = 3;
```

PS：删除的时候如果不指定where条件，则保留数据表结构，删除全部数据行，有主外键关系的都删不了

与 SELECT 语句不同的是，DELETE 语句中不能使用 GROUP BY、 HAVING 和 ORDER BY 三类子句，而只能使用WHERE 子句。原因很简单， GROUP BY 和 HAVING 是从表中选取数据时用来改变抽取数据形式的， 而 ORDER BY 是用来指定取得结果显示顺序的。因此，在删除表中数据 时它们都起不到什么作用。



2、删除表中全部数据。语法：

```sql
-- 删除表中全部数据
truncate 表名;
```

PS：全部删除，内存无痕迹，如果有自增会重新开始编号。

与 DELETE 不同的是，TRUNCATE 只能删除表中的全部数据，而不能通过 WHERE 子句指定条件来删除部分数据。也正是因为它不能具体地控制删除对象， 所以其处理速度比 DELETE 要快得多。实际上，DELETE 语句在 DML 语句中也 属于处理时间比较长的，因此需要删除全部数据行时，使用 TRUNCATE 可以缩短 执行时间。




3、对比 truncate、delete、drop 三种数据删除方式。

```sql
-- 删除表数据的三种方式，truncate、delete 只是单纯删除数据，而drop不仅删除数据还删除了数据表
truncate table 表名;
delete from 表名;
drop table 表名;

-- 删除速度
drop > truncate > delete
```

PS：注意事项：

面试时：面试官问在删改数据之前，你会怎么做？

1. 答案：会对数据进行备份操作，以防万一，可以进行数据回退

面试时：delete与truncate与drop 这三种删除数据的共同点都是删除数据，他们的不同点是什么?

1. delele：会把删除的操作记录给记录起来，以便数据回退，不会释放空间，而且不会删除定义。
2. truncate：不会记录删除操作，会把表占用的空间恢复到最初，不会删除定义
3. drop：会删除整张表，释放表占用的空间。









## 4、中文乱码

查看当前MySQL使用的字符集：show variables like 'character%';

```shell
mysql> show variables like 'character%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8                             |
| character_set_connection | utf8                             |
| character_set_database   | utf8                             |
| character_set_filesystem | binary                           |
| character_set_results    | utf8                             |
| character_set_server     | utf8                             |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/mysql/share/charsets/ |
+--------------------------+----------------------------------+

- character_set_client：客户端请求数据的字符集
- character_set_connection：客户端与服务器连接的字符集
- character_set_database：数据库服务器中某个库使用的字符集设定，如果建库时没有指明，将默认使用配置上的字符集
- character_set_results：返回给客户端的字符集(从数据库读取到的数据是什么编码的)
- character_set_server：为服务器安装时指定的默认字符集设定。
- character_set_system：系统字符集(修改不了的，就是utf8)
- character_sets_dir：mysql字符集文件的保存路径
```
临时修改：

```shell
mysql> set character_set_xxx gbk;
```

永久修改：修改配置文件my.cnf里边的

```ini
[client]
# 作用于外部的显示
default-character-set=gbk

[mysqld]
# 作用于内部，会作用于创建库表时默认字符集
character_set_server=gbk
```

修改库的字符集编码

```sql
alter database db_name default character set gbk;
```

修改表的字符集编码

```sql
alter table employee default character set utf8;
```





# DQL 数据查询语言

细讲对数据表中的数据进行各种查询，以及项目实战查询

```sql
-- 创建部门表
CREATE TABLE dept(
    deptnu      INT  PRIMARY KEY comment '部门编号',
    dname       VARCHAR(50) comment '部门名称',
    addr        VARCHAR(50) comment '部门地址'
);

-- 某个公司的员工表
CREATE TABLE employee(
    empno       INT  PRIMARY KEY comment '雇员编号',
    ename       VARCHAR(50) comment '雇员姓名',
    job         VARCHAR(50) comment '雇员职位',
    mgr         INT comment '雇员上级编号',
    hiredate    DATE comment '雇佣日期',
    sal         DECIMAL(7,2) comment '薪资',
    deptnu      INT comment '部门编号'
)ENGINE=MyISAM DEFAULT CHARSET=utf8;

-- 创建工资等级表
CREATE TABLE salgrade(
    grade       INT  PRIMARY KEY comment '等级',
    lowsal      INT comment '最低薪资',
    higsal      INT comment '最高薪资'
);

-- 插入dept表数据
INSERT INTO dept VALUES (10, '研发部', '北京');
INSERT INTO dept VALUES (20, '工程部', '上海');
INSERT INTO dept VALUES (30, '销售部', '广州');
INSERT INTO dept VALUES (40, '财务部', '深圳');

-- 插入emp表数据
INSERT INTO employee VALUES (1009, '唐僧', '董事长', NULL, '2010-11-17', 50000,  10);
INSERT INTO employee VALUES (1004, '猪八戒', '经理', 1009, '2001-04-02', 29750, 20);
INSERT INTO employee VALUES (1006, '猴子', '经理', 1009, '2011-05-01', 28500, 30);
INSERT INTO employee VALUES (1007, '张飞', '经理', 1009, '2011-09-01', 24500,10);
INSERT INTO employee VALUES (1008, '诸葛亮', '分析师', 1004, '2017-04-19', 30000, 20);
INSERT INTO employee VALUES (1013, '林俊杰', '分析师', 1004, '2011-12-03', 30000, 20);
INSERT INTO employee VALUES (1002, '牛魔王', '销售员', 1006, '2018-02-20', 16000, 30);
INSERT INTO employee VALUES (1003, '程咬金', '销售员', 1006, '2017-02-22', 12500, 30);
INSERT INTO employee VALUES (1005, '后裔', '销售员', 1006, '2011-09-28', 12500, 30);
INSERT INTO employee VALUES (1010, '韩信', '销售员', 1006, '2018-09-08', 15000,30);
INSERT INTO employee VALUES (1012, '安琪拉', '文员', 1006, '2011-12-03', 9500,  30);
INSERT INTO employee VALUES (1014, '甄姬', '文员', 1007, '2019-01-23', 7500, 10);
INSERT INTO employee VALUES (1011, '妲己', '文员', 1008, '2018-05-23', 11000, 20);
INSERT INTO employee VALUES (1001, '小乔', '文员', 1013, '2018-12-17', 8000, 20);

-- 插入salgrade表数据
INSERT INTO salgrade VALUES (1, 7000, 12000);
INSERT INTO salgrade VALUES (2, 12010, 14000);
INSERT INTO salgrade VALUES (3, 14010, 20000);
INSERT INTO salgrade VALUES (4, 20010, 30000);
INSERT INTO salgrade VALUES (5, 30010, 99990);
```



## 1、简单查询

1、指定字段查询和所有字段查询

```sql
select * from employee;
select empno,ename,job as ename_job from employee;
```

2、清除重复值

```sql
select distinct(job) from employee;
```

3、as 取别名

```sql
select tb_employee.ename as name from employee as tb_employee
```

使用as也可以为表取别名 （作用：单表查询意义不大，但是当多个表的时候取别名就好操作，当不同的表里有相同名字的列的时候区分就会好区分）



## 2、where 条件查询

where 的作用：

1. 用于检索数据表中符合条件的记录
2. 搜索条件可由一个或多个逻辑表达式组成，结果一般为真或假

where 子语句操作符：

| 操 作 符 | 说明     | 操作符      | 说明                 |
| -------- | -------- | ----------- | -------------------- |
| =        | 等于     | >=          | 大于等于             |
| <>、!=   | 不等于   | !>          | 不大于               |
| <        | 小于     | between and | 在两值之间(包含边界) |
| <=       | 小于等于 | is null     | 是null值             |
| !<       | 不小于   | is not null | 不是null值           |
| >        | 大于     | in          | 查询in内的值         |



1、精确条件查询

```sql
select * from employee where ename='后裔';
select * from employee where sal != 50000;
select * from employee where sal <> 50000;
select * from employee where sal > 10000;
```

2、模糊条件查询

```sql
select * from employee  where ename like '林%';
```

3、范围查询

```sql
select * from employee where sal between 10000 and 30000;
select * from employee where hiredate between '2011-01-01' and '2017-12-1';
```

5、离散查询

```sql
select * from employee where ename in ('猴子','林俊杰','小红','小胡');  
```

6、统计查询（聚合函数）:

```sql
-- count(code)或者count(*)
select count(*) from employee;
select count(ename) from employee;
        
-- sum()  计算总和 
select sum(sal) from employee;
        
-- max()    计算最大值
select * from employee where sal= (select  max(sal) from employee);
        
-- avg()   计算平均值
select avg(sal) from employee;
        
-- min()   计算最低值
select * from employee where sal= (select  min(sal) from employee);
        
-- concat函数： 起到连接作用
select concat(ename,' 是 ',job) as aaaa from employee;
```



## 3、group by 分组查询（分组）

1. 作用：把行 按 字段 分组
2. 语法：group by 列1，列2...，列N
3. 场景：常用于统计场合，一般和聚合函数连用

```sql
select deptnu,count(*) from employee group by deptnu;
select deptnu,job,count(*) from employee group by deptnu,job;
select job,count(*) from employee group by job;
```



## 4、having 条件查询（筛选）

1. 作用：对查询的结果进行筛选操作
2. 语法：having 条件 或者 having 聚合函数 条件
3. 场景：一般跟在group by之后

```sql
select job,count(*) from employee group by job having job ='文员';
select deptnu,job,count(*) from employee group by deptnu,job having count(*) >= 2;
select deptnu,job,count(*) as 总数 from employee group by deptnu,job having 总数 >= 2;
```



## 5、order by 排序查询（排序）

1. 作用：对查询的结果进行排序操作（默认升序 asc，从小到大）
2. 语法：order by 字段1,字段2... [asc | desc] 
3. 场景：一般用在查询结果的排序

```sql
select * from employee order by sal;
select * from employee order by hiredate;

select  deptnu,job,count(*) as 总数 from employee group by deptnu,job having 总数>=2 order by deptnu desc;
select  deptnu,job,count(*) as 总数 from employee group by deptnu,job having 总数>=2 order by deptnu asc;
select  deptnu,job,count(*) as 总数 from employee group by deptnu,job having 总数>=2 order by deptnu;

-- 顺序：where ---- group by ----- having ------ order by 
```



## 6、limit 限制查询（限制）

1. 作用：对查询结果起到限制条数的作用
2. 语法：limit n，m。n：代表起始条数值，不写默认为0；m：代表取出的条数
3. 场景：数据量过多时，可以起到限制作用

```sql
-- 从第4条数据开始取，取5条数据
select * from XD.employee limit 4,5;
```



## 7、In & Exists  子查询

1. exists：子查询后面是一个受限的select查询语句
2. exists：子查询，如果exists后的内层查询能查出数据，则返回 TRUE 表示存在；为空则返回 FLASE则不存在。
3. 分为俩种：exists 跟 not exists​
4. in：当内层查询 (括号内的) 结果会有多个结果时, 不能使用 = 必须是in ,另外子查询必须只能包含一列数据

```sql
select 1 from employee where 1=1;
select * from 表名 a where exists (select 1 from 表名2 where 条件);

-- 查询出公司有员工的部门的详细信息
select * from dept a where exists (select 1 from employee b where a.deptnu=b.deptnu);
select * from dept a where not exists (select 1 from employee b where a.deptnu=b.deptnu);
```



## 8、左连接与右连接查询

1. 左连接称之为左外连接，右连接称之为右外连接，这俩个连接都是属于外连接
2. 左连接关键字：left join 表名 on 条件 / left outer 表名 join on 条件 
3. 右连接关键字：right join 表名 on 条件/ right outer 表名 join on 条件
4. 左连接说明： left join 是left outer join的简写，左(外)连接，左表(a_table)的记录将会全部表示出来， 而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL
5. 右连接说明：right join是right outer join的简写，与左(外)连接相反，右(外)连接，左表(a_table)只会显示符合搜索条件的记录，而右表(b_table)的记录将会全部表示出来。左表记录不足的地方均为NULL

```sql
-- 列出部门名称和这些部门的员工信息，同时列出那些没有的员工的部门
-- dept，employee
select a.dname,b.* from dept a  left join employee b on a.deptnu=b.deptnu;
select b.dname,a.* from employee a  right join  dept b on b.deptnu=a.deptnu;
```



## 9、内连接查询与联合查询

1. 内连接：获取两个表中字段匹配关系的记录，本质上就是笛卡尔积查询
2. 主要语法：INNER JOIN 表名 ON 条件;

```sql
-- 想查出员工张飞的所在部门的地址
select a.addr from dept a inner join employee b on a.deptnu=b.deptnu and b.ename='张飞';
select a.addr from dept a,employee b where a.deptnu=b.deptnu and b.ename='张飞';
```

联合查询：就是把多个查询语句的查询结果结合在一起
1. 主要语法1：... UNION ... （去除重复） 
2. 主要语法2：... UNION ALL ...（不去重复）

UNION 查询的注意事项：
1. 1.两个select语句的查询结果的“字段数”必须一致；​
2. 通常，也应该让两个查询语句的字段类型具有一致性；​
3. 也可以联合更多的查询结果；​
4. 用到order by排序时，需要加上limit（加上最大条数就行），需要对子句用括号括起来

```sql
-- 对销售员的工资从低到高排序，而文员的工资从高到低排序
(select * from employee a where a.job = '销售员'  order by a.sal limit 999999) 
    union  
(select * from employee b where b.job = '文员' order by b.sal desc limit 999999);
```



## 10、项目高级查询实战

1、查出至少有一个员工的部门。显示部门编号、部门名称、部门位置、部门人数。

```sql
-- 涉及表： employee dept
-- 语句：
select deptnu,count(*) from employee group by deptnu
-- 语句：
select a.deptnu,a.dname,a.addr, b.zongshu from dept a,
	(select deptnu,count(*) as zongshu from employee group by deptnu) b 
	where a.deptnu=b.deptnu;
```

2、列出薪金比安琪拉高的所有员工

```sql
--涉及表：employee
--语句：
select * from  employee where sal > (select sal from employee where ename='安琪拉');
```

3、列出所有员工的姓名及其直接上级的姓名

```sql
-- 涉及表：employee
-- 语句：
select a.ename,ifnull(b.ename,'BOSS') as leader from employee a left join employee b on a.mgr=b.empno;
```

4、列出受雇日期早于直接上级的所有员工的编号、姓名、部门名称

```sql
-- 涉及表：employee dept
-- 条件：a.hiredate < b.hiredate
-- 语句：
select a.empno,a.ename,c.dname from employee a 
    left join employee b on a.mgr=b.empno 
    left join dept c on a.deptnu=c.deptnu 
    where a.hiredate < b.hiredate;
```

5、列出部门名称和这些部门的员工信息，同时列出那些没有员工的部门

```sql
-- 涉及表：dept employee
-- 语句：
select a.dname,b.* from dept a left join employee b on a.deptnu=b.deptnu;
```

6、列出所有文员的姓名及其部门名称，所在部门的总人数

```sql
-- 涉及表：employee dept
-- 条件：job='文员'
-- 语句：
select deptnu,count(*) as zongshu from employee group by deptnu;
--语句：
select b.ename,a.dname,b.job,c.zongshu from dept a ,employee b ,
	(select deptnu,count(*) as zongshu from employee group by deptnu) c 
	where a.deptnu=b.deptnu and b.job='文员' and b.deptnu=c.deptnu;
```

7、列出最低薪金大于15000的各种工作及从事此工作的员工人数

```sql
-- 涉及表：employee
-- 条件：min(sal) > 15000 
-- 语句：
select job,count(*) from employee group by job having   min(sal) > 15000;
```

8、列出在销售部工作的员工的姓名，假定不知道销售部的部门编号

```sql
-- 涉及表：employee dept
select  ename  from employee where deptnu=(select deptnu from dept where dname='销售部');
```

9、列出与诸葛亮从事相同工作的所有员工及部门名称

```sql
-- 涉及表：employee dept
-- 语句：
select a.ename,b.dname from employee a,dept b 
	where a.deptnu = b.deptnu 
	and a.job = (select job from employee where ename='诸葛亮');
--语句：
select a.ename,b.dname from employee a 
	left join dept b on a.deptnu = b.deptnu 
	where a.job = (select job from employee where ename='诸葛亮');
```

10、列出薪金比 在部门30工作的员工的薪金 还高的员工姓名和薪金、部门名称

```sql
-- 涉及表：employee dept
-- 语句：
select a.ename,a.sal,b.dname from employee a ,dept b 
	where a.deptnu=b.deptnu 
	and sal > (select max(sal) from employee where deptnu=30);
```

11、列出每个部门的员工数量、平均工资

```sql
-- 涉及表：employee
-- 语句：
select deptnu , count(*) ,avg (sal) from employee  group by deptnu;
```

12、列出薪金高于公司平均薪金的所有员工信息，所在部门名称，上级领导，工资等级

```sql
-- 涉及表：employee dept salgrade
-- 条件：select avg(sal) from employee
-- 语句：
select a.*,c.dname,b.ename,d.grade from employee a,employee b,dept c ,salgrade d 
    where a.mgr=b.empno and a.deptnu =c.deptnu 
    and a.sal > (select avg(sal) from employee) 
    and a.sal between d.lowsal and d.higsal;
```


# DCL 数据控制语言（用户权限）

数据控制语言（DCL：Data Control Language）是用来设置或者更改数据库用户或角色权限的语句，这些语句包括GRANT、REVOKE、DENY等语句

## 1、创建用户

> MySQL 数据库提供 3 种创建用户的方式：
>
> 1. 使用 create user 语句创建用户
> 2. 在 mysql.user 表中添加用户
> 3. 使用 GRANT 语句创建用户

### 1、CREATE USER 语句创建用户

```sql
-- 简约版
-- user_name 表示用户名
-- host 表示允许用户从哪个主机连接 MySQL 服务器，如果省略（等价于 %）表示任何主机，如果是本地用户可用localhost
-- identified by 用于指定用户的密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器
create user [IF NOT EXISTS] user_name@host identified by 'auth_string';

-- 完整版
CREATE USER <用户> [ IDENTIFIED BY [ PASSWORD ] 'password' ] [ ,用户 [ IDENTIFIED BY [ PASSWORD ] 'password' ]]
```

参数说明如下：

1. 用户：指定创建用户账号，格式为 `user_name'@'host_name`。这里的`user_name`是用户名，`host_name`为主机名，即用户连接 MySQL 时所用主机的名字。如果在创建的过程中，只给出了用户名，而没指定主机名，那么主机名默认为“%”，表示一组主机，即对所有主机开放权限。设置为`localhost`代表本机
2. IDENTIFIED BY 子句：用于指定用户密码。新用户可以没有初始密码，若该用户不设密码，可省略此子句。
3. PASSWORD 'password'：PASSWORD 表示使用哈希值设置密码，该参数可选。如果密码是一个普通的字符串，则不需要使用 PASSWORD 关键字。'password' 表示用户登录时使用的密码，需要用单引号括起来。

使用 CREATE USER 语句时应注意以下几点：

1. CREATE USER 语句可以不指定初始密码。但是从安全的角度来说，不推荐这种做法。
2. 使用 CREATE USER 语句必须拥有 mysql 数据库的 INSERT 权限或全局 CREATE USER 权限。
3. 使用 CREATE USER 语句创建一个用户后，MySQL 会在 mysql 数据库的 user 表中添加一条新记录。
4. CREATE USER 语句可以同时创建多个用户，多个用户用逗号隔开。

新创建的用户拥有的权限很少，它们只能执行不需要权限的操作。如登录 MySQL、使用 SHOW 语句查询所有存储引擎和字符集的列表等。如果两个用户的用户名相同，但主机名不同，MySQL 会将它们视为两个用户，并允许为这两个用户分配不同的权限集合。



操作示例1：使用 CREATE USER 创建一个用户，用户名是 test1，密码是 password，主机名是 localhost。

```sql
create user 'test1'@'localhost' identified by 'password';
```

操作示例2：实际应用中我们为了避免明文指定密码，可以通过 PASSWORD 关键字使用密码的哈希值设置密码，可以使用 password() 函数获取密码的哈希值

```sql
mysql> select password('password');
+-------------------------------------------+
| password('password')                      |
+-------------------------------------------+
| *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19 |
+-------------------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> create user 'test2'@'localhost' identified by password '*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19';
Query OK, 0 rows affected, 1 warning (0.00 sec)

-- 刷新权限，不然创建了也无法登录
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

PS：实际上不管是通过hash创建还是明文创建，最终存入到数据库表中的密码都是哈希转换过了。

---

### 2、INSERT 语句创建用户

> MySQL 5.7 的 user 表中的密码字段从 Password 变成了 authentication_string，如果你使用的是 MySQL 5.7 之前的版本，将 authentication_string 字段替换成 Password 即可。

可以使用 INSERT 语句将用户的信息添加到 mysql.user 表中，但必须拥有对 mysql.user 表的 INSERT 权限。通常 INSERT 语句只添加 Host、User 和 authentication_string 这 3 个字段的值。

使用 INSERT 语句创建用户的代码如下：

```sql
insert into mysql.user (Host, User, authentication_string, ssl_cipher, x509_issuer, x509_subject) values ('hostname', 'username', PASSWORD('password'), '', '', '');
```

PS：由于 mysql 数据库的 user 表中，ssl_cipher、x509_issuer 和 x509_subject 这 3 个字段没有默认值，所以向 user 表插入新记录时，一定要设置这 3 个字段的值，否则 INSERT 语句将不能执行。



操作示例1：使用 INSERT 语句创建名为 test3 的用户，主机名是 localhost，密码也是 password

```sql
mysql> insert into mysql.user(Host, User, authentication_string, ssl_cipher, x509_issuer, x509_subject) VALUES ('localhost', 'test3', PASSWORD('password'), '', '', '');
Query OK, 1 row affected, 1 warning (0.00 sec)

-- 刷新权限，不然创建了也无法登录
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

PS 注意事项：

1. 刷新权限可以让 MySQL 刷新系统权限相关表。执行 FLUSH 命令需要 RELOAD 权限。
2. user 表中的 User 和 Host 字段区分大小写，创建用户时要指定正确的用户名称或主机名。

---

### 3、GRANT 语句创建用户

虽然 CREATE USER 和 INSERT INTO 语句都可以创建普通用户，但是这两种方式不便授予用户权限。于是 MySQL 提供了 GRANT 语句。

使用 GRANT 语句创建用户的基本语法形式如下：

```sql
GRANT priv_type ON database.table TO user [IDENTIFIED BY [PASSWORD] 'password']
```

参数说明如下：

1. priv_type：参数表示新用户的权限；
2. database.table：参数表示新用户的权限范围，即只能在指定的数据库和表上使用自己的权限。
3. user：参数指定新用户的账号，由用户名和主机名构成；
4. IDENTIFIED BY 关键字用来设置密码；
5. password 参数表示新用户的密码。

操作示例1：使用 GRANT 语句创建名为 test4 的用户，主机名为 localhost，密码为 password。该用户对所有数据库的所有表都有 SELECT 权限

```sql
-- *.* 表示所有数据库下的所有表
mysql> grant select on *.* TO 'test4'@localhost IDENTIFIED BY 'password';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

PS：GRANT 是 MySQL 中一个非常重要的语句，它可以用来创建用户、修改用户密码和设置用户权限



### 4、创建用户并限制IP登录

1、创建用户并限制IP网段登录

```sql
-- user_name 表示用户名
-- host 表示允许用户从哪个主机连接 MySQL 服务器，如果省略（等价于 %）表示任何主机，如果是本地用户可用localhost
-- identified by 用于指定用户的密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器
create user [IF NOT EXISTS] user_name@host identified by 'auth_string';

-- 1.创建一个pig用户，并指定登录密码：123456，可以在任何一台远程主机都可以登录
create user 'sam'@'%' identified by '123456';
-- 刷新权限
flush privileges;

-- 2.创建一个pig用户，并指定登录密码：为空，指定在120网段的机器登录
create user 'pig'@'120.%.%.%' identified by '';
-- 刷新权限
flush privileges;
```

2、限制已存在用户指定IP登录

```sql
-- 查看root用户可以在哪台机器登录
select user,host from mysql.user where user='root';

-- 修改mysql库里边的user表。host='localhost'表示：只允许root用户在本地环境登录。host='%'表示：允许root用户通过任意IP访问登录
update mysql.user set host='localhost' where user='root';
update mysql.user set host='%' where user='root';

-- 修改后必须刷新权限才能生效
flush privileges;
```

3、查看权限

```sql
-- 查询
select * from mysql.user where user='pig'\G

mysql> show grants for 'pig'@'%';
+---------------------------------+
| Grants for pig@%                |
+---------------------------------+
| GRANT USAGE ON *.* TO 'pig'@'%' |
+---------------------------------+
1 row in set (0.00 sec)
-- USAGE：无权限的意思 

mysql> show grants for 'root'@'localhost';
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+
2 rows in set (0.00 sec)
-- WITH GRANT OPTION：表示这个用户拥有grant权限，即可以对其他用户授权
```







## 2、查看用户

1、MySQL 中的用户信息存储在 mysql.user 系统表中：

```sql
mysql> select user,host from mysql.user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
4 rows in set (0.00 sec)
```

2、使用 user() 或者 current_user() 函数查看当前用户：

```sql
mysql> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)

mysql> select current_user();
+----------------+
| current_user() |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```



## 3、修改用户名

我们可以使用 RENAME USER 语句修改一个或多个已经存在的用户账号。语法格式如下：

```sql
-- <旧用户>：系统中已经存在的 MySQL 用户账号。<新用户>：新的 MySQL 用户账号
RENAME USER <旧用户> TO <新用户>
```

使用 RENAME USER 语句时应注意以下几点：

1. RENAME USER 语句用于对原有的 MySQL 用户进行重命名
2. 若系统中旧账户不存在或者新账户已存在，该语句执行时会出现错误
3. 使用 RENAME USER 语句，必须拥有 mysql 数据库的 UPDATE 权限或全局 CREATE USER 权限

操作示例：修改test1用户名为 testUser1

```sql
mysql> rename user 'test1'@'localhost' to 'testUser1'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```



## 4、删除用户

在 MySQL 数据库中，可以使用 DROP USER 语句删除用户，也可以直接在 mysql.user 表中删除用户以及相关权限。

用户删除方式一：使用 DROP USER 语句删除普通用户，语法格式如下：

```sql
DROP USER <用户1> [, <用户2>...]
```

使用 DROP USER 语句应注意以下几点：

1. DROP USER 语句可用于删除一个或多个用户，并撤销其权限
2. DROP USER 语句的使用必须拥有 mysql 数据库的 DELETE 权限或全局 CREATE USER 权限
3. DROP USER 语句的使用中，若没有明确地给出账户的主机名，则该主机名默认为“%”
4. PS 注意：用户的删除不会影响他们之前所创建的表、索引或其他数据库对象，因为 MySQL 并不会记录是谁创建了这些对象

用户删除方式二：使用 DELETE 语句删除普通用户，直接删除 mysql.user 表中相应的用户信息，但必须拥有 mysql.user 表的 DELETE 权限。语法：

```sql
-- Host 和 User 这两个字段都是 mysql.user 表的主键。因此，需要两个字段的值才能确定一条记录
delete from mysql.user where Host='hostname' and User='username';
```



---

操作示例 1：

```sql
mysql> drop user 'test2'@'localhost', 'test3'@'localhost', 'test4'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

操作示例 2：

```sql
mysql> delete from mysql.user where Host='localhost'AND User='testUser1';
Query OK, 1 row affected (0.00 sec)

mysql> select user,host from mysql.user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
4 rows in set (0.00 sec)
```



## 5、修改密码

修改用户密码分四种方法：

### 1、SET 语句修改密码

第一种：SET语句修改密码（MySQL客户端界面内）数据库中只有 root 用户可以通过更新 MySQL 数据库来更改密码。语法：

```sql
-- username：是普通用户的用户名。hostname：参数是普通用户的主机名。newpwd：是要更改的新密码
-- PS：新密码必须使用 PASSWORD() 函数来加密，如果不使用 PASSWORD() 加密，也会执行成功，但是用户会无法登录
set password for 'username'@'hostname' = password('newpwd');

-- 如果是普通用户修改密码，可省略 FOR 子句来更改自己的密码。语法格式如下：
set password = password('newpwd');
```

操作示例：

```sql
-- 为指定用户设置密码
set password for 'root'@'%' = password('newpassword');
set password for 'root'@'localhost' = password('newpassword');

-- 为当前登录用户设置密码
set password = password('newpassword');

-- 刷新权限
flush privileges;
```

---

### 2、UPDATE 语句修改密码

第二种：UPDATE语句修改普通用户的密码（MySQL客户端界面内）Root 用户可以使用 UPDATE 语句修改 MySQL 数据库的 user 表的 authentication_string 字段，从而修改普通用户的密码。语法：

```sql
-- authentication_string：密码字段。newpwd：新密码。（username 和 hostname：这2个字段必须都添值）
update mysql.user set authentication_string = PASSWORD("newpwd") where user = "username" AND Host = "hostname";
```

操作示例：

```sql
-- % 代表可以从任意远程主机登陆，localhost 代表只有本地用户可以登录
update mysql.user set authentication_string=password('root') where user='root' and host='%';
update mysql.user set authentication_string=password('root') where user='root' and host='localhost';

-- 刷新权限
flush privileges;
```

---

### 3、mysqladmin 命令修改密码

第三种：使用mysqladmin命令在命令行指定新密码（在Linux界面中）Root 用户可以使用 mysqladmin 命令来修改密码，mysqladmin 语法：

```sql
mysqladmin -u username -h hostname -p password "newpwd";
```

语法参数说明如下：

1. usermame：指需要修改密码的用户名称，在这里指定为 root 用户
2. hostname：指需要修改密码的用户主机名，该参数可以不写，默认是 localhost
3. password：为关键字，而不是指旧密码
4. newpwd：为新设置的密码，必须用双引号括起来。如果使用单引号会引发错误，可能会造成修改后的密码不是你想要的

操作示例：

```shell
mysqladmin -uroot -p password 123456;
mysqladmin -uroot -proot password 123456;
```

---

### 4、GRANT 语句修改密码

第四种：使用 GRANT 语句修改普通用户密码。除了前面介绍的方法，还可以在全局级别使用 GRANT USAGE 语句指定某个账户的密码而不影响账户当前的权限。需要注意的是，使用 GRANT 语句修改密码，必须拥有 GRANT 权限。一般情况下最好使用该方法来指定或修改密码。语法：

```sql
-- username 参数是普通用户的用户名，hostname 参数是普通用户的主机名，newpwd 是要更改的新密码。
GRANT USAGE ON *.* TO 'user'@'hostname' IDENTIFIED BY 'newpwd';
```

操作示例：Root 用户登录，再使用 GRANT 语句将 test1 用户的密码修改为“newpwd3”，SQL 语句和运行结果如下：

```sql
mysql> grant usage on *.* TO 'test1'@'localhost' IDENTIFIED BY 'newpassword';
Query OK, 0 rows affected, 1 warning (0.05 sec)
```



## 6、忘记密码

1. 第一步：修改配置文件my.cnf (默认在/etc/my.cnf)，在[mysqld]下面加上 skip-grant-tables （跳过权限的意思）
2. 第二步：重启MySQL服务
3. 第三步：登录服务端（无需密码登录进入）：mysql -uroot -p 
4. 第四步：修改密码



## 7、用户权限表详解

MySQL 在安装时会自动创建一个名为 mysql 的数据库，mysql 数据库中存储的都是用户权限表。用户登录以后，MySQL 会根据这些权限表的内容为每个用户赋予相应的权限。

user 表是 MySQL 中最重要的一个权限表，用来记录允许连接到服务器的账号信息。需要注意的是，在 user 表里启用的所有权限都是全局级的，适用于所有数据库。user 表中的字段大致可以分为 4 类，分别是用户列、权限列、安全列和资源控制列。

> 用户列

用户列存储了用户连接数据库时需要输入的信息。需要注意 MySQL 5.7 版本不再使用 Password 来作为密码的字段，而改成：authentication_string

MySQL 5.7 版本的用户列：

| 字段名                | 字段类型 | 是否为空 | 默认值 | 说明   |
| --------------------- | -------- | -------- | ------ | ------ |
| Host                  | char(60) | NO       | 无     | 主机名 |
| User                  | char(32) | NO       | 无     | 用户名 |
| authentication_string | text     | YES      | 无     | 密码   |

用户登录时，如果这 3 个字段同时匹配，MySQL 数据库系统才会允许其登录。创建新用户时，也是设置这 3 个字段的值。修改用户密码时，实际就是修改 user 表的 authentication_string 字段的值。因此，这 3 个字段决定了用户能否登录。

---

> 权限列

权限列的字段决定了用户的权限，用来描述在全局范围内允许对数据和数据库进行的操作。权限大致分为两大类，分别是高级管理权限和普通权限：

1. 高级管理权限主要对数据库进行管理，例如：关闭服务的权限、超级权限和加载用户等
2. 普通权限主要操作数据库，例如：查询权限、修改权限等


user 表的权限列包括 Select_priv、Insert_ priv 等以 priv 结尾的字段，这些字段值的数据类型为 ENUM，可取的值只有 Y 和 N：Y 表示该用户有对应的权限，N 表示该用户没有对应的权限。从安全角度考虑，这些字段的默认值都为 N。

MySQL 5.7 版本 user 表的权限列

| 字段名                 | 字段类型      | 是否为空 | 默认值 | 说明                                                         |
| ---------------------- | ------------- | -------- | ------ | ------------------------------------------------------------ |
| Select_priv            | enum('N','Y') | NO       | N      | 是否可以通过SELECT 命令查询数据                              |
| Insert_priv            | enum('N','Y') | NO       | N      | 是否可以通过 INSERT 命令插入数据                             |
| Update_priv            | enum('N','Y') | NO       | N      | 是否可以通过UPDATE 命令修改现有数据                          |
| Delete_priv            | enum('N','Y') | NO       | N      | 是否可以通过DELETE 命令删除现有数据                          |
| Create_priv            | enum('N','Y') | NO       | N      | 是否可以创建新的数据库和表                                   |
| Drop_priv              | enum('N','Y') | NO       | N      | 是否可以删除现有数据库和表                                   |
| Reload_priv            | enum('N','Y') | NO       | N      | 是否可以执行刷新和重新加载MySQL所用的各种内部缓存的特定命令，<br />包括日志、权限、主机、查询和表 |
| Shutdown_priv          | enum('N','Y') | NO       | N      | 是否可以关闭MySQL服务器。将此权限提供给root账户之外的任何用户时，都应当非常谨慎 |
| Process_priv           | enum('N','Y') | NO       | N      | 是否可以通过SHOW PROCESSLIST命令查看其他用户的进程           |
| File_priv              | enum('N','Y') | NO       | N      | 是否可以执行SELECT INTO OUTFILE和LOAD DATA INFILE命令        |
| Grant_priv             | enum('N','Y') | NO       | N      | 是否可以将自己的权限再授予其他用户                           |
| References_priv        | enum('N','Y') | NO       | N      | 是否可以创建外键约束                                         |
| Index_priv             | enum('N','Y') | NO       | N      | 是否可以对索引进行增删查                                     |
| Alter_priv             | enum('N','Y') | NO       | N      | 是否可以重命名和修改表结构                                   |
| Show_db_priv           | enum('N','Y') | NO       | N      | 是否可以查看服务器上所有数据库的名字，包括用户拥有足够访问权限的数据库 |
| Super_priv             | enum('N','Y') | NO       | N      | 是否可以执行某些强大的管理功能，例如通过KILL命令删除用户进程；<br />使用SET GLOBAL命令修改全局MySQL变量，执行关于复制和日志的各种命令（超级权限） |
| Create_tmp_table_priv  | enum('N','Y') | NO       | N      | 是否可以创建临时表                                           |
| Lock_tables_priv       | enum('N','Y') | NO       | N      | 是否可以使用LOCK TABLES命令阻止对表的访问/修改               |
| Execute_priv           | enum('N','Y') | NO       | N      | 是否可以执行存储过程                                         |
| Repl_slave_priv        | enum('N','Y') | NO       | N      | 是否可以读取用于维护复制数据库环境的二进制日志文件           |
| Repl_client_priv       | enum('N','Y') | NO       | N      | 是否可以确定复制从服务器和主服务器的位置                     |
| Create_view_priv       | enum('N','Y') | NO       | N      | 是否可以创建视图                                             |
| Show_view_priv         | enum('N','Y') | NO       | N      | 是否可以查看视图                                             |
| Create_routine_priv    | enum('N','Y') | NO       | N      | 是否可以更改或放弃存储过程和函数                             |
| Alter_routine_priv     | enum('N','Y') | NO       | N      | 是否可以修改或删除存储函数及函数                             |
| Create_user_priv       | enum('N','Y') | NO       | N      | 是否可以执行CREATE USER命令，这个命令用于创建新的MySQL账户   |
| Event_priv             | enum('N','Y') | NO       | N      | 是否可以创建、修改和删除事件                                 |
| Trigger_priv           | enum('N','Y') | NO       | N      | 是否可以创建和删除触发器                                     |
| Create_tablespace_priv | enum('N','Y') | NO       | N      | 是否可以创建表空间                                           |


 如果要修改权限，可以使用 GRANT 语句为用户赋予一些权限，也可以通过 UPDATE 语句更新 user 表的方式来设置权限

---

> 安全列

安全列主要用来判断用户是否能够登录成功，user 表中的安全列如表所示：

| 字段名                | 字段类型                          | 是否为空 | 默认值                | 说明                                                         |
| --------------------- | --------------------------------- | -------- | --------------------- | ------------------------------------------------------------ |
| ssl_type              | enum('','ANY','X509','SPECIFIED') | NO       |                       | 支持ssl标准加密安全字段                                      |
| ssl_cipher            | blob                              | NO       |                       | 支持ssl标准加密安全字段                                      |
| x509_issuer           | blob                              | NO       |                       | 支持x509标准字段                                             |
| x509_subject          | blob                              | NO       |                       | 支持x509标准字段                                             |
| plugin                | char(64)                          | NO       | mysql_native_password | 引入plugins以进行用户连接时的密码验证，<br />plugin创建外部/代理用户 |
| password_expired      | enum('N','Y')                     | NO       | N                     | 密码是否过期 (N 未过期，y 已过期)                            |
| password_last_changed | timestamp                         | YES      |                       | 记录密码最近修改的时间                                       |
| password_lifetime     | smallint(5) unsigned              | YES      |                       | 设置密码的有效时间，单位为天数                               |
| account_locked        | enum('N','Y')                     | NO       | N                     | 用户是否被锁定（Y 锁定，N 未锁定）                           |

注意 PS：即使 password_expired 为 “Y”，用户也可以使用密码登录 MySQL，但是不允许做任何操作。

通常标准的发行版不支持 ssl，读者可以使用 SHOW VARIABLES LIKE "have_openssl" 语句来查看是否具有 ssl 功能。如果 have_openssl 的值为 DISABLED，那么则不支持 ssl 加密功能。

---

> 资源控制列

资源控制列的字段用来限制用户使用的资源，user 表中的资源控制列如表所示

| 字段名               | 字段类型         | 是否为空 | 默认值 | 说明                             |
| -------------------- | ---------------- | -------- | ------ | -------------------------------- |
| max_questions        | int(11) unsigned | NO       | 0      | 规定每小时允许执行查询的操作次数 |
| max_updates          | int(11) unsigned | NO       | 0      | 规定每小时允许执行更新的操作次数 |
| max_connections      | int(11) unsigned | NO       | 0      | 规定每小时允许执行的连接操作次数 |
| max_user_connections | int(11) unsigned | NO       | 0      | 规定允许同时建立的连接次数       |


以上字段的默认值为 0，表示没有限制。一个小时内用户查询或者连接数量超过资源控制限制，用户将被锁定，直到下一个小时才可以在此执行对应的操作。可以使用 GRANT 语句更新这些字段的值。



## 8、其他权限表详解

在 MySQL 数据库中，权限表除了 user 表外，还有 db 表、tables_priv 表、columns_priv 表和 procs_priv 表。

> db表

db 表比较常用，是 MySQL 数据库中非常重要的权限表，表中存储了用户对某个数据库的操作权限。表中的字段大致可以分为两类，分别是用户列和权限列。

1、用户列

db 表用户列有 3 个字段，分别 Host、User、Db，标识从某个主机连接某个用户对某个数据库的操作权限，这 3 个字段的组合构成了 db 表的主键。如表所示

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明     |
| ------ | -------- | -------- | ------ | -------- |
| Host   | char(60) | NO       | 无     | 主机名   |
| Db     | char(64) | NO       | 无     | 数据库名 |
| User   | char(32) | NO       | 无     | 用户名   |

2、权限列

db 表中的权限列和 user 表中的权限列大致相同，只是 user 表中的权限是针对所有数据库的，而 db 表中的权限只针对指定的数据库。如果希望用户只对某个数据库有操作权限，可以先将 user 表中对应的权限设置为 N，然后在 db 表中设置对应数据库的操作权限。

---

> tables_priv表 和 columns_priv表

tables_priv 表用来对单个表进行权限设置，columns_priv 表用来对单个数据列进行权限设置。

tables_priv 表结构如下表所示：

| 字段名      | 字段类型                                                     | 是否为空 | 默认值            | 说明                                                         |
| ----------- | ------------------------------------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| Host        | char(60)                                                     | NO       | 无                | 主机                                                         |
| Db          | char(64)                                                     | NO       | 无                | 数据库名                                                     |
| User        | char(32)                                                     | NO       | 无                | 用户名                                                       |
| Table_name  | char(64)                                                     | NO       | 无                | 表名                                                         |
| Grantor     | char(93)                                                     | NO       | 无                | 修改该记录的用户                                             |
| Timestamp   | timestamp                                                    | NO       | CURRENT_TIMESTAMP | 修改该记录的时间                                             |
| Table_priv  | set('Select','Insert','Update','Delete',' Create','Drop','Grant','References', 'Index','Alter','Create View','Show view','Trigger') | NO       | 无                | 表示对表的操作权限，包括 Select、Insert、Update、Delete、Create、Drop、Grant、References、Index 和 Alter 等 |
| Column_priv | set('Select','Insert','Update','References')                 | NO       | 无                | 表示对表中的列的操作权限，包括 Select、Insert、Update 和 References |

columns_priv 表结构如下表所示：

| 字段名      | 字段类型                                     | 是否为空 | 默认值            | 说明                                                         |
| ----------- | -------------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| Host        | char(60)                                     | NO       | 无                | 主机                                                         |
| Db          | char(64)                                     | NO       | 无                | 数据库名                                                     |
| User        | char(32)                                     | NO       | 无                | 用户名                                                       |
| Table_name  | char(64)                                     | NO       | 无                | 表名                                                         |
| Column_name | char(64)                                     | NO       | 无                | 数据列名称，用来指定对哪些数据列具有操作权限                 |
| Timestamp   | timestamp                                    | NO       | CURRENT_TIMESTAMP | 修改该记录的时间                                             |
| Column_priv | set('Select','Insert','Update','References') | NO       | 无                | 表示对表中的列的操作权限，包括 Select、Insert、Update 和 References |

---

> procs_priv表

procs_priv 表可以对存储过程和存储函数进行权限设置，procs_priv 的表结构如表所示：

| 字段名       | 字段类型                               | 是否为空 | 默认值            | 说明                                                         |
| ------------ | -------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| Host         | char(60)                               | NO       | 无                | 主机名                                                       |
| Db           | char(64)                               | NO       | 无                | 数据库名                                                     |
| User         | char(32)                               | NO       | 无                | 用户名                                                       |
| Routine_name | char(64)                               | NO       | 无                | 表示存储过程或函数的名称                                     |
| Routine_type | enum('FUNCTION','PROCEDURE')           | NO       | 无                | 表示存储过程或函数的类型，Routine_type 字段有两个值，分别是 FUNCTION 和 PROCEDURE。FUNCTION 表示这是一个函数；PROCEDURE 表示这是一个 存储过程。 |
| Grantor      | char(93)                               | NO       | 无                | 插入或修改该记录的用户                                       |
| Proc_priv    | set('Execute','Alter Routine','Grant') | NO       | 无                | 表示拥有的权限，包括 Execute、Alter Routine、Grant 3种       |
| Timestamp    | timestamp                              | NO       | CURRENT_TIMESTAMP | 表示记录更新时间                                             |



## 9、查看用户权限

在 MySQL 中，可以通过查看 mysql.user 表中的数据记录来查看相应的用户权限，也可以使用 SHOW GRANTS 语句查询用户的权限。

方式一：数据库中的 user 表中存储着用户的基本权限，可以使用 SELECT 语句查看，语句如下（要执行该语句，必须拥有对 user 表的查询权限）

```sql
-- 查询全部用户的全部字段
select * from mysql.user;

-- 由于mysql.user表中字段过多，一般我们就查看两三个字段
mysql> select user,host from mysql.user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
4 rows in set (0.00 sec)
```

注意 PS：新创建的用户只有登录 MySQL 服务器的权限，没有任何其它权限，不能查询 user 表

---

方式二：使用 SHOW GRANTS FOR 语句查看权限。其语法格式如下：

```sql
-- 查询权限语法：username 表示用户名，hostname 表示主机名或主机 IP。
SHOW GRANTS FOR 'username'@'hostname';
```

操作示例1：创建 testuser1 用户并查询权限，SQL 语句和执行过程如下：

```sql
mysql> create user 'testuser1'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for 'testuser1'@'localhost';
+-----------------------------------------------+
| Grants for testuser1@localhost                |
+-----------------------------------------------+
| GRANT USAGE ON *.* TO 'testuser1'@'localhost' |
+-----------------------------------------------+
1 row in set (0.00 sec)
```

注意 PS：其中，`USAGE ON *.*` 表示该用户对任何数据库和任何表都没有权限，只能登录。

操作示例2：查询 root 用户的权限

```sql
mysql> show grants for 'root'@'localhost';
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+
2 rows in set (0.00 sec)
```



## 10、授予用户权限

授权就是为某个用户赋予某些权限。例如，可以为新建的用户赋予查询所有数据库和表的权限。MySQL 提供了 GRANT 语句来为用户设置权限。

在 MySQL 中必须拥有 GRANT 权限的用户才可以执行 GRANT 语句，其语法格式如下：

```sql
GRANT priv_type [(column_list)] [, priv_type [(column_list)]]...
ON database.table
TO user [IDENTIFIED BY [PASSWORD] 'password'] [, user[IDENTIFIED BY [PASSWORD] 'password']]...
[WITH with_option [with_option]...]
```

参数详细说明：

1. priv_type：表示权限类型；
2. columns_list：表示权限作用于哪些列上，省略该参数时，表示作用于整个表；
3. database.table：用于指定权限的级别
4. user：表示用户账户，由用户名和主机名构成，格式：'username'@'hostname'
5. IDENTIFIED BY：该参数用来为用户设置密码
6. password：是用户的新密码

WITH 关键字后面带有一个或多个 with_option 参数。这个参数有 5 个选项：

1. GRANT OPTION：被授权的用户可以将这些权限赋予给别的用户
2. MAX_QUERIES_PER_HOUR count：设置每个小时可以允许执行 count 次查询
3. MAX_UPDATES_PER_HOUR count：设置每个小时可以允许执行 count 次更新
4. MAX_CONNECTIONS_PER_HOUR count：设置每小时可以建立 count 个连接
5. MAX_USER_CONNECTIONS count：设置单个用户可以同时具有的 count 个连接

---

MySQL中可以授予的权限有如下几组：

1. 列权限：和表中的一个具体列相关。例如，可以使用 UPDATE 语句更新表 students 中 name 列的值的权限
2. 表权限：和一个具体表中的所有数据相关。例如，可以使用 SELECT 语句查询表 students 的所有数据的权限
3. 数据库权限：和一个具体的数据库中的所有表相关。例如，可以在已有的数据库 mytest 中创建新表的权限
4. 用户权限：和 MySQL 中所有的数据库相关。例如，可以删除已有的数据库或者创建一个新的数据库的权限

对应地，在 GRANT 语句中可用于指定权限级别的值有以下几类格式：

1. *：表示当前数据库中的所有表。
2. *.*：表示所有数据库中的所有表。
3. db_name.*：表示某个数据库中的所有表，db_name 指定数据库名。
4. db_name.tbl_name：表示某个数据库中的某个表或视图，db_name 指定数据库名，tbl_name 指定表名或视图名。
5. db_name.routine_name：表示某个数据库中的某个存储过程或函数，routine_name 指定存储过程名或函数名。
6. TO 子句：如果权限被授予给一个不存在的用户，MySQL 会自动执行一条 CREATE USER 语句来创建这个用户，但同时必须为该用户设置密码。

---

> 权限类型说明

下面讲解 GRANT 语句中的权限类型

1、授予数据库权限时，<权限类型>priv_type可以指定为以下值：

| 权限名称                       | 对应user表中的字段    | 说明                                                         |
| ------------------------------ | --------------------- | ------------------------------------------------------------ |
| SELECT                         | Select_priv           | 表示授予用户可以使用 SELECT 语句访问特定数据库中所有表和视图的权限 |
| INSERT                         | Insert_priv           | 表示授予用户可以使用 INSERT 语句向特定数据库中所有表添加数据行的权限 |
| DELETE                         | Delete_priv           | 表示授予用户可以使用 DELETE 语句删除特定数据库中所有表的数据行的权限 |
| UPDATE                         | Update_priv           | 表示授予用户可以使用 UPDATE 语句更新特定数据库中所有数据表的值的权限 |
| REFERENCES                     | References_priv       | 表示授予用户可以创建指向特定的数据库中的表外键的权限         |
| CREATE                         | Create_priv           | 表示授权用户可以使用 CREATE TABLE 语句在特定数据库中创建新表的权限 |
| ALTER                          | Alter_priv            | 表示授予用户可以使用 ALTER TABLE 语句修改特定数据库中所有数据表的权限 |
| SHOW VIEW                      | Show_view_priv        | 表示授予用户可以查看特定数据库中已有视图的视图定义的权限     |
| CREATE ROUTINE                 | Create_routine_priv   | 表示授予用户可以为特定的数据库创建存储过程和存储函数的权限   |
| ALTER ROUTINE                  | Alter_routine_priv    | 表示授予用户可以更新和删除数据库中已有的存储过程和存储函数的权限 |
| INDEX                          | Index_priv            | 表示授予用户可以在特定数据库中的所有数据表上定义和删除索引的权限 |
| DROP                           | Drop_priv             | 表示授予用户可以删除特定数据库中所有表和视图的权限           |
| CREATE TEMPORARY TABLES        | Create_tmp_table_priv | 表示授予用户可以在特定数据库中创建临时表的权限               |
| CREATE VIEW                    | Create_view_priv      | 表示授予用户可以在特定数据库中创建新的视图的权限             |
| EXECUTE ROUTINE                | Execute_priv          | 表示授予用户可以调用特定数据库的存储过程和存储函数的权限     |
| LOCK TABLES                    | Lock_tables_priv      | 表示授予用户可以锁定特定数据库的已有数据表的权限             |
| SHOW DATABASES                 | Show_db_priv          | 表示授权可以使用SHOW DATABASES语句查看所有已有的数据库的定义的权限 |
| ALL 或 ALL PRIVILEGES 或 SUPER | Super_priv            | 表示以上所有权限/超级权限                                    |

2、授予表权限时，<权限类型>priv_type可以指定为以下值：

| 权限名称                       | 对应user表中的字段 | 说明                                                       |
| ------------------------------ | ------------------ | ---------------------------------------------------------- |
| SELECT                         | Select_priv        | 授予用户可以使用 SELECT 语句进行访问特定表的权限           |
| INSERT                         | Insert_priv        | 授予用户可以使用 INSERT 语句向一个特定表中添加数据行的权限 |
| DELETE                         | Delete_priv        | 授予用户可以使用 DELETE 语句从一个特定表中删除数据行的权限 |
| DROP                           | Drop_priv          | 授予用户可以删除数据表的权限                               |
| UPDATE                         | Update_priv        | 授予用户可以使用 UPDATE 语句更新特定数据表的权限           |
| ALTER                          | Alter_priv         | 授予用户可以使用 ALTER TABLE 语句修改数据表的权限          |
| REFERENCES                     | References_priv    | 授予用户可以创建一个外键来参照特定数据表的权限             |
| CREATE                         | Create_priv        | 授予用户可以使用特定的名字创建一个数据表的权限             |
| INDEX                          | Index_priv         | 授予用户可以在表上定义索引的权限                           |
| ALL 或 ALL PRIVILEGES 或 SUPER | Super_priv         | 所有的权限名                                               |

3、授予列权限时，<权限类型>priv_type的值只能指定为：SELECT、INSERT 和 UPDATE，同时权限的后面需要加上列名列表 column-list

4、授予用户权限时，<权限类型>priv_type除了可以指定为授予数据库权限时的所有值之外，还可以是下面这些值（最有效率的权限是用户权限）：

1. CREATE USER：表示授予用户可以创建和删除新用户的权限。
2. SHOW DATABASES：表示授予用户可以使用 SHOW DATABASES 语句查看所有已有的数据库的定义的权限。

---

操作示例 1：使用 GRANT 语句创建一个新的用户 testUser，密码为 password。用户 testUser 对所有的数据有查询、插入权限，并授予 GRANT 权限：

```sql
mysql> grant select,insert on *.*
    -> to 'testUser'@'localhost'
    -> IDENTIFIED BY 'password'
    -> WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.05 sec)
```

操作示例 2：使用 SHOW GRANTS 语句查询用户 testUser 的权限

```sql
mysql> show grants for 'testUser'@'localhost';
+-------------------------------------------------------------------------+
| Grants for testUser@localhost                                           |
+-------------------------------------------------------------------------+
| GRANT SELECT, INSERT ON *.* TO 'testUser'@'localhost' WITH GRANT OPTION |
+-------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

结果显示，testUser 对所有数据库的所有表有查询、插入权限，并可以将这些权限赋予给别的用户



## 11、撤销用户权限

在 MySQL 中，可以使用 REVOKE 语句删除某个用户的某些权限（此用户不会被删除），在一定程度上可以保证系统的安全性

使用 REVOKE 语句删除权限的语法格式有两种形式，如下所示：

第一种：删除用户某些特定的权限，语法格式如下：

```sql
REVOKE priv_type [(column_list)]...
ON database.table
FROM user [, user]...
```

REVOKE 语句中的参数与 GRANT 语句的参数意思相同：

1. priv_type：表示权限的类型
2. column_list：表示权限作用于哪些列上，没有该参数时作用于整个表上
3. user：由用户名和主机名构成，格式为“username'@'hostname'”

第二种：删除特定用户的所有权限，语法格式如下：

```sql
REVOKE ALL PRIVILEGES, GRANT OPTION FROM user [, user] ...
```

删除用户权限需要注意以下几点：

1. REVOKE 语法和 GRANT 语句的语法格式相似，但具有相反的效果
2. 要使用 REVOKE 语句，必须拥有 MySQL 数据库的全局 CREATE USER 权限或 UPDATE 权限

---

操作示例：使用 REVOKE 语句取消用户 testUser 的插入权限，SQL 语句和执行过程如下。

```sql
mysql> REVOKE INSERT ON *.*
    -> FROM 'testUser'@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW GRANTS FOR 'testUser'@'localhost';
+-----------------------------------------------------------------+
| Grants for testUser@localhost                                   |
+-----------------------------------------------------------------+
| GRANT SELECT ON *.* TO 'testUser'@'localhost' WITH GRANT OPTION |
+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```



## 12、刷新用户权限

当创建和删除用户、修改用户名、修改密码、修改权限和撤销权限等等操作之后，都需要刷新用户权限才能生效，或者重启。

```sql
-- 刷新授权
FLUSH PRIVILEGES;
```



## 13、简版权限授权与回收

```sql
-- 授权语法（grant）
-- all privileges：代表所有权限
-- *.*：代表所有库所有表
-- with grant option：这个选项表示该用户可以将自己拥有的权限授权给别人
-- PS：经常有人在创建操作用户的时候不指定WITH GRANT OPTION选项导致后来该用户不能使用GRANT命令创建用户或者给其它用户授权
grant 权限1,权限2..... on 数据库对象 to '用户'  [with grant option]
grant 权限1,权限2..... on 数据库对象 to '用户'@'host' identified by 'password' [with grant option];

-- 权限授权 操作示例：
-- 对现有用户进行授权：对现有用户pig授予所有库所有表所有权限。 
grant all privileges on *.* to 'pig';

-- 对没有的用户进行授权：创建一个新用户dog授予XD库的所有权限，登录密码123456，任何一台主机登录 
grant all privileges on XD.* to 'dog'@'%' identified by '123456';

-- 对没有的用户进行授权：创建一个新用户cat授予XD库的employee表 查与修改权限，登录密码123456，任何一台主机登录
grant select,update on XD.employee to 'cat'@'%' identified by '123456';

-- 对没有的用户进行授权：对用户cat授予XD库的employee表insert 权限，登录密码123456，任何一台主机登录 
grant insert on XD.employee to 'cat'@'%' identified by '123456';

-- 刷新权限
flush privileges;
```

```sql
-- 权限回收（revoke）语法：
revoke 权限1,权限2..... on 数据库对象 from '用户'@'host';

-- 回收pig用户的所有权限（注意：并没有回收它的登录权限） 
revoke all privileges on *.* from 'pig' @ '%'; 
flush privileges;

-- 回收pig用户的所有权限（并回收它的登录权限） 
delete from mysql.user where user='pig'; 
flush privileges;

-- 回收cat用户对XD库的employee的查与修改权限 
revoke select,update on XD.employee from 'cat'@'%'; 

-- 刷新权限
flush privileges;
```

权限列表解释

```sql
-- 权限列表解释
ALL [PRIVILEGES]           -- 设置除GRANT OPTION之外的所有简单权限
ALTER                      -- 允许使用ALTER TABLE
ALTER ROUTINE              -- 更改或取消已存储的子程序
CREATE                     -- 允许使用CREATE TABLE
CREATE ROUTINE             -- 创建已存储的子程序
CREATE TEMPORARY TABLES    -- 允许使用CREATE TEMPORARY TABLE
CREATE USER                -- 允许使用CREATE USER, DROP USER, RENAME USER和REVOKE ALL PRIVILEGES。
CREATE VIEW                -- 允许使用CREATE VIEW
DELETE                     -- 允许使用DELETE
DROP                       -- 允许使用DROP TABLE
EXECUTE                    -- 允许用户运行已存储的子程序
FILE                       -- 允许使用SELECT...INTO OUTFILE和LOAD DATA INFILE
INDEX                      -- 允许使用CREATE INDEX和DROP INDEX
INSERT                     -- 允许使用INSERT
LOCK TABLES                -- 允许对您拥有SELECT权限的表使用LOCK TABLES
PROCESS                    -- 允许使用SHOW FULL PROCESSLIST
REFERENCES                 -- 未被实施
RELOAD                     -- 允许使用FLUSH
REPLICATION CLIENT         -- 允许用户询问从属服务器或主服务器的地址
REPLICATION SLAVE          -- 用于复制型从属服务器（从主服务器中读取二进制日志事件）
SELECT                     -- 允许使用SELECT
SHOW DATABASES             -- 显示所有数据库
SHOW VIEW                  -- 允许使用SHOW CREATE VIEW
SHUTDOWN                   -- 允许使用mysqladmin shutdown
SUPER     -- 允许使用CHANGE MASTER, KILL, PURGE MASTER LOGS和SET GLOBAL语句，mysqladmin debug命令,允许您连接(1次)，即使已达到max_connections
UPDATE                     -- 允许使用UPDATE
USAGE                      -- “无权限”的同义词,USAGE权限只能允许用户登录
GRANT OPTION               -- 允许授予权限


/* 表维护 */
-- 分析和存储表的关键字分布
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE 表名 ...
-- 检查一个或多个表是否有错误
CHECK TABLE tbl_name [, tbl_name] ... [option] ...
option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}
-- 整理数据文件的碎片
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```



# TCL 事务控制语言

## 1、事务处理介绍

什么是事务？

1. 一个或一组SQL语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行（除非明确指示）
2. 当多个应用程序在并发访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰
3. 如果发生错误，则进行回退（撤销），将数据库恢复到某个已知且安全的状态，以此来维护数据库的完整性
4. MySQL事务处理只支持InnoDB和BDB数据表类型，MyISAM 存储引擎不支持事务

事务处理术语：

1. 事务（transaction）：指一组 SQL 语句
2. 回退（rollback）：指撤销指定 SQL 语句的过程
3. 提交（commit）：指将未存储的 SQL 语句结果写入数据库表
4. 保留点（savepoint）：指事务处理中设置的临时占位符（placeholder），可以对它发布回退（与回退整个事务处理不同）

事务处理用来管理 `INSERT`、`UPDATE` 和 `DELETE` 语句。不能回退 `SELECT` 语句（回退 `SELECT` 语句也没有必要），也不能回退 `CREATE` 或 `DROP` 操 作。事务处理中可以使用这些语句，但进行回退时，这些操作也不撤销。

事务的特性（ACID）：

1. 原子性（Atomicity）：事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行
2. 一致性（Consistency）：事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束
3. 隔离性（Isolation）：多个事务并发执行时，但每个事务就像各自独立执行一样，互不影响
4. 持久性（Durability）：已被提交的事务对数据库的修改应该永久保存在数据库中

事务的分类：

1. 隐式事务：事务没有明显的开启和结束的标记（例如：insert，update，delete 语句）
2. 显式事务：事务具有明显的开启和结束的标记（配置：autocommit 变量设置为 0）

> 在 MySQL 命令行的默认设置下，事务都是自动提交的，即执行 SQL 语句后就会马上执行 COMMIT 操作（隐式事务）。因此要显式地开启一个事务务须使用命令 BEGIN 或 START TRANSACTION，或者执行命令 SET AUTOCOMMIT=0，用来禁止使用当前会话的自动提交（显式事务）



## 2、控制事务处理

1. 事务的开启：begin;（5.7之后）、start transaction;（5.7之前）
2. 事务的提交：commit;
3. 事务的回滚：rollback;
4. 查询autocommit是否开启：show variables like 'autocommit';
5. 开启autocommit（临时生效）：
   1. OFF（0）：表示关闭：**set autocommit=0;**
   2. ON  （1）：表示开启：**set autocommit=1;**
6. 开启autocommit（永久生效）：
   1. 修改配置文件：**vi /etc/my.cnf**
   2. 在[mysqld]下面加上：**autocommit=1**
   3. 记得重启服务才会生效

```sql
 # 关闭事务自动提交
 mysql> set autocommit=0;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> show variables like 'autocommit';
  +---------------+-------+
  | Variable_name | Value |
  +---------------+-------+
  | autocommit    | OFF   |
  +---------------+-------+
  
  # 开启事务自动提交
  mysql> set autocommit=1;
  Query OK, 0 rows affected (0.00 sec)
  mysql> 
  mysql> show variables like 'autocommit';
  +---------------+-------+
  | Variable_name | Value |
  +---------------+-------+
  | autocommit    | ON    |
  +---------------+-------+
```

---

1. `BEGIN / START TRANSACTION`：显式地开启一个事务（begin 是5.7之后，transaction 是5.7之前）
2. `COMMIT / COMMIT WORK`：提交事务，使已对数据库进行的所有修改成为永久性的
3. `ROLLBACK / ROLLBACK WORK`：回滚会结束用户的事务，并撤销正在进行的所有**未提交**的修改
4. `SAVEPOINT`：`SAVEPOINT` 允许在事务中创建一个保存点，一个事务中可以有多个 `SAVEPOINT`
5. `RELEASE SAVEPOINT`：删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常
6. `ROLLBACK TO`：把事务回滚到标记点
7. `SET TRANSACTIO`：用来设置事务的隔离级别。InnoDB 存储引擎提供事务的隔离级别有 READ UNCOMMITTED（读未提交）、READ COMMITTED（读已提交，项目中常用）、REPEATABLE READ（可重复读，Mysql 默认隔离级别） 和 SERIALIZABLE（串行化）

操作示例 1：

```sql
mysql> create table transaction_test(id int(5)) engine = innodb;       # 创建数据表
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT * FROM transaction_test;                                 # 查询数据表
Empty set (0.00 sec)

mysql> begin;                                                          # 开启事务
Query OK, 0 rows affected (0.00 sec)

mysql> insert into transaction_test values (1),(2);                    # 插入数据
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> commit;                                                         # 提交事务
Query OK, 0 rows affected (0.00 sec)

mysql> select * from transaction_test;                                 # 查询数据
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
2 rows in set (0.00 sec)
```

操作示例 2：

```sql
mysql> begin;                                         # 开启事务
Query OK, 0 rows affected (0.00 sec)

mysql> insert into transaction_test values (3);       # 插入数据
Query OK, 1 row affected (0.00 sec)

mysql> savepoint first_insert;                        # 声明一个保存点
Query OK, 0 rows affected (0.00 sec)

mysql> insert into transaction_test values (4);       # 插入数据
Query OK, 1 row affected (0.00 sec)

mysql> savepoint second_insert;                       # 声明一个保存点
Query OK, 0 rows affected (0.00 sec)

mysql> insert into transaction_test values (5);       # 插入数据
Query OK, 1 row affected (0.00 sec)

mysql> rollback to second_insert;                     # 回滚到第二个保存点
Query OK, 0 rows affected (0.00 sec)

mysql> select * from transaction_test;                # 发现数据数据回到第二个保存点
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
+------+
4 rows in set (0.00 sec)

mysql> rollback to first_insert;                      # 回滚到第一个保存点
Query OK, 0 rows affected (0.00 sec)

mysql> select * from transaction_test;                # 发现数据数据回到第一个保存点
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

mysql> commit;                                       # 提交事务
Query OK, 0 rows affected (0.00 sec)

mysql> select * from transaction_test;               # 查询数据
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)
```



## 3、事务隔离级别

查看事务隔离级别语句：

```sql
# 查看默认事务隔离级别（session）
select @@transaction_isolation;
select @@tx_isolation;
# 查看当前会话的隔离级别
select @@session.transaction_isolation;
select @@session.tx_isolation;
# 查看全局的事务隔离级别
select @@global.transaction_isolation;
select @@global.tx_isolation;
```

在 MyQL下事务的隔离级别有四种，由低到高依次为 `Read uncommitted`（读未提交） 、`Read committed`（读已提交）、`Repeatable read` (可重复读。默认)、`Serializable`（串行化），这四个级别中的后三个级别可以逐个解决**脏读** 、**不可重复读** 、**幻读**的问题。

1. 未提交读（Read Uncommitted）：允许脏读，也就是可能读取到其他会话中未提交事务修改的数据
2. 提交读（Read Committed）：只能读取到已经提交的数据。Oracle等多数数据库默认都是该级别 （不重复读）
3. 可重复读（Repeated Read）：可重复读。在同一个事务内的查询都是事务开始时刻一致的，InnoDB默认级别。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻象读
4. 串行读（Serializable）：完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞（会把查询出的内容加锁：select * from user for update）

脏读 、不可重复读 、幻读 介绍：

1. 脏读：就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
2. 不可重复读：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读
3. 幻读：第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样



### 1、脏读

```
（一）脏读测试：将A的隔离级别设置为read uncommitted(未提交读)
A：启动事务，此时数据为初始状态
B：启动事务，更新数据，但不提交
A：再次读取数据，发现数据已经被修改了，这就是所谓的“脏读”
B：回滚事务
A：再次读数据，发现数据变回初始状态

经过上面的实验可以得出结论，事务B更新了一条记录，但是没有提交，此时事务A可以查询出未提交记录。造成脏读现象。未提交读是最低的隔离级别。
编者按：脏读，读取到未提交事务的数据。
```

对于两个事务T1与T2，T1读取了已经被T2更新但是还没有提交的字段之后，若此时T2回滚，T1读取的内容就是临时并且无效的。

T1 操作窗口：

```sql
# 修改当前会话隔离级别为：read uncommitted（读未提交）
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

# 步骤1：查询表数据（步骤2查看T2窗口）
mysql> select * from transaction_test;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

# 步骤4：查看T2事务中修改的数据（步骤5查看T2窗口）
mysql> select * from transaction_test;
+------+
| id   |
+------+
|   11 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

# 步骤6：查看数据又恢复原样了
mysql> select * from transaction_test;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)
```

T2 操作窗口：

```sql
# 步骤2：开启事务
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

# 步骤3：修改数据，此时并没有commit和rollback（步骤4查看T1窗口）
mysql> update transaction_test set id = 11 where id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

# 步骤5：rollback事务中的修改（步骤6查看T1窗口）
mysql> rollback;
Query OK, 0 rows affected (0.00 sec)
```

由于T2窗口的步骤5回滚，T1窗口的客户端中步骤4中读取的数据就是临时并且无效的。即脏读。



### 2、不可重复读

```
（二）不可重复读测试：将客户端A的事务隔离级别设置为read committed(已提交读)
A：启动事务，此时数据为初始状态
B：启动事务，更新数据，但不提交
A：再次读数据，发现数据未被修改
B：提交事务
A：再次读取数据，发现数据已发生变化，说明B提交的修改被事务中的A读到了，这就是所谓的“不可重复读”

经过上面的实验可以得出结论，已提交读隔离级别解决了脏读的问题，但是出现了不可重复读的问题，即事务A在两次查询的数据不一致，因为在两次查询之间事务B更新了一条数据。已提交读只允许读取已提交的记录，但不要求可重复读。
编者按：本次事务中不会读取到其他事务对相关数据做的改变，除非其他事务提交。不可重复读的意思就是说，如果另一事务B有提交，则A事务中两次读会出现不一致。
```

对于两个事务T1和T2，T1读取了一个字段，然后T2更新了该字段并提交之后，当T1再次读取的时候，结果不一致的情况发生。

T1 操作窗口：

```sql
# 步骤1：开启事务，然后查询数据（步骤2在T2窗口）
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from transaction_test;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

# 步骤3：查询修改后的数据
mysql> select * from transaction_test;
+------+
| id   |
+------+
|   11 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)
```

T2 操作窗口：

```sql
# 步骤2：开启事务、修改数据、并提交事务commit（步骤3在T1窗口）
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update transaction_test set id = 11 where id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

由于T2的修改操作，导致T1客户端2次查询结果不一致



### 3、幻读

```
（三）幻读测试：将A的隔离级别设置为repeatable read(可重复读)
A：启动事务，此时数据为初始状态
B：启动事务，更新数据，但不提交
A：再次读取数据，发现数据未被修改
B：提交事务
A：再次读取数据，发现数据依然未发生变化，这说明这次可以重复读了
B：插入一条新的数据，并提交
A：再次读取数据，发现数据依然未发生变化，虽然可以重复读了，但是却发现读的不是最新数据，这就是所谓的“幻读”
A：提交本次事务，再次读取数据，发现读取正常了

由以上的实验可以得出结论，可重复读隔离级别只允许读取已提交记录，而且在一个事务两次读取一个记录期间，其他事务不得更新该记录。但该事务不要求与其他事务可串行化。例如，当一个事务可以找到由一个已提交事务更新的记录，但是可能产生幻读问题(注意是可能，因为数据库对隔离级别的实现有所差别)。像以上的实验，就没有出现数据幻读的问题。
编者按：本次事务中不会读取到其他提交的事务数据，当前事务结束后再次读取会读到最新数据。
```



对于两个事务T1、T2，T1从表中读取数据，然后T2进行了INSERT操作并提交，当T1再次读取的时候，结果不一致的情况发生。

T1 操作窗口：

```sql
# 步骤1：开启事务、查询数据（步骤2在T2窗口）
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from transaction_test;
+------+
| id   |
+------+
|   11 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

# 步骤3：查看数据
mysql> select * from transaction_test;
+------+
| id   |
+------+
|   11 |
|    2 |
|    3 |
|    4 |
+------+
4 rows in set (0.00 sec)
```

T2 操作窗口：

```sql
# 步骤2：开启事务、插入数据、提交事务commit（步骤3在T1窗口）
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into transaction_test values(4);
Query OK, 1 row affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

由于T2的插入操作，导致T1客户端2次查询结果不一致



### 4、隔离级别导致的问题

| 隔离级别                     | 脏读（Dirty Read） | 不可重复读（NonRepeatable Read） | 幻读（Phantom Read） |
| ---------------------------- | ------------------ | -------------------------------- | -------------------- |
| 未提交读（Read uncommitted） | 可能               | 可能                             | 可能                 |
| 已提交读（Read committed）   | 不可能             | 可能                             | 可能                 |
| 可重复读（Repeatable read）  | 不可能             | 不可能                           | 可能                 |
| 可串行化（Serializable ）    | 不可能             | 不可能                           | 不可能               |

> 可参考：
>
> 1. https://blog.csdn.net/weisong530624687/article/details/90075063





# 视图

## 1、视图介绍

MySQL 视图（View）是一种虚拟存在的表，同真实表一样，视图也由列和行构成，但视图并不实际存在于数据库中。行和列的数据来自于定义视图的查询中所使用的表，并且还是在使用视图时动态生成的。

什么是视图？视图的作用是什么？
1. 视图（view）是一种虚拟存在的表，是一个逻辑表，它本身是不包含数据的。作为一个select语句保存在数据字典中的。
2. 通过视图，可以展现基表（用来创建视图的表叫做基表base table）的部分数据，说白了视图的数据就是来自于基表

视图的优点：

1. 简单：使用视图的用户完全不需要关心后面对应的表的结构、关联条件和筛选条件，对用户来说已经是过滤好的复合条件的结果集。
2. 安全：使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但是通过视图就可以简单的实现
3. 数据独立：一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列对视图没有影响。源表修改列名，则可以通过修改视图来解决，不会造成对访问者的影响
4. 不占用空间：视图是逻辑上的表，不占用内存空间
5. 总而言之，使用视图的大部分情况是为了保障数据安全性，提高查询效率。

视图的缺点：

1. 性能差：sql server必须把视图查询转化成对基本表的查询，如果这个视图是由一个复杂的多表查询所定义，那么，即使是视图的一个简单查询，sql server也要把它变成一个复杂的结合体，需要花费一定的时间
2. 修改限制：当用户试图修改试图的某些信息时，数据库必须把它转化为对基本表的某些信息的修改，对于简单的试图来说，这是很方便的，但是，对于比较复杂的试图，可能是不可修改的。



## 2、创建视图

视图创建语法：

```sql
CREATE [OR REPLACE] VIEW [field_name1,...] view_name AS select-statement WITH CHECK OPTION;
create [or replace] view [field_name1,...] view_name AS select-statement with check option;
```

参数说明：

1. or replace：视图存在时，可以对视图进行修改；视图不存在时，可以创建视图
2. field_name：默认情况下，创建的视图和基本表的字段是一样的，也可以通过指定视图字段的名称来创建视图
3. view_name：视图的名称。该名称在数据库中必须是唯一的，不能与其他表或视图同名
4. select-statement：指定创建视图的 SELECT 语句，可用于查询多个基础表或源视图
5. with check option：该选项可以阻止通过视图修改或者插入视图范围之外的基础表数据（修改视图时，检查插入的数据是否符合 WHERE 设置的条件）

对于创建视图中的 SELECT 语句的指定存在以下限制：

1. 用户除了拥有 CREATE VIEW 权限外，还具有操作中涉及的基础表和其他视图的相关权限。
2. SELECT 语句不能引用系统或用户变量。
3. SELECT 语句不能包含 FROM 子句中的子查询。
4. SELECT 语句不能引用预处理语句参数。

视图使用注意事项：

1. 
   视图定义中引用的表或视图必须存在。但创建完视图后，可以删除定义引用的表或视图。可使用 CHECK TABLE 语句检查视图定义是否存在这类问题

2. 视图定义中允许使用 ORDER BY 语句，但是若从特定视图进行选择，而该视图使用了自己的 ORDER BY，则视图定义中的 ORDER BY 将被忽略

3. 视图定义中不能引用 TEMPORARY 表（临时表），不能创建 TEMPORARY 视图




## 3、查看视图

查看视图的字段信息与查看视图的详细定义，具体语法如下：

```sql
-- 查看视图的字段信息（与查看数据表一样）
desc 视图名;
describe 视图名;

-- 查看视图的详细信息(定义语句)
show create view 视图名;
```

操作示例：

```sql
CREATE TABLE studentinfo(
    ID INT(11) PRIMARY KEY,
    NAME VARCHAR(20),
    SCORE DECIMAL(4,2),
    SUBJECT VARCHAR(20),
    TEACHER VARCHAR(20)
);
```

```sql
Database changed
mysql> CREATE TABLE studentinfo(
    ->     ID INT(11) PRIMARY KEY,
    ->     NAME VARCHAR(20),
    ->     SCORE DECIMAL(4,2),
    ->     SUBJECT VARCHAR(20),
    ->     TEACHER VARCHAR(20)
    -> );
Query OK, 0 rows affected (0.02 sec)

-- 创建查询学生姓名和分数的视图语句如下：
mysql> CREATE VIEW v_studentinfo AS SELECT name,score FROM studentinfo;
Query OK, 0 rows affected (0.01 sec)

-- 查看视图 v_studentsinfo 中的字段信息，SQL 语句和运行结果如下所示。
mysql> desc v_studentinfo;
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| name  | varchar(20)  | YES  |     | NULL    |       |
| score | decimal(4,2) | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

-- 查看视图定义的详细信息
mysql> show create view v_studentinfo \G
*************************** 1. row ***************************
                View: v_studentinfo
         Create View: CREATE ALGORITHM=UNDEFINED DEFINER=`root`@`localhost` SQL SECURITY DEFINER VIEW `v_studentinfo` AS select `studentinfo`.`NAME` AS `name`,`studentinfo`.`SCORE` AS `score` from `studentinfo`
character_set_client: utf8
collation_connection: utf8_general_ci
1 row in set (0.00 sec)
```

> 拓展阅读
>
> 所有视图的定义都是存储在 information_schema 数据库下的 views 表中，也可以在这个表中查看所有视图的详细信息，SQL 语句如下：
>
> ```sql
> select * from information_schema.views;
> ```



## 4、修改视图

```sql
-- 修改的语法是：
alter view <视图名称> as select 语句;
```

PS：对于 ALTER VIEW 使用，需要用户具有针对视图的 CREATE VIEW 和 DROP 权限，以及由 SELECT 语句选择的每一列上的某些权限。修改视图的定义，除了可以通过 ALTER VIEW 外，也可以使用 DROP VIEW 语句先删除视图，再使用 CREATE VIEW 语句来实现。

> 1、修改视图定义

操作示例 1：使用 ALTER 语句修改视图 view_students_info，输入的 SQL 语句和执行结果如下所示

```sql
mysql> alter view view_students_info AS select id,name,age from tb_students_info;
Query OK, 0 rows affected (0.07 sec)

mysql> desc view_students_info;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   |     | 0       |       |
| name  | varchar(45) | YES  |     | NULL    |       |
| age   | int(11)     | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.03 sec)
```



> 2、修改视图内容

视图是一个虚拟表，实际的数据来自于基本表，所以通过插入、修改和删除操作更新视图中的数据，实质上是在更新视图所引用的基本表的数据。

注意：对视图的修改就是对基本表的修改，因此在修改时，要满足基本表的数据定义。某些视图是可更新的。也就是说，可以使用 UPDATE、DELETE 或 INSERT 等语句更新基本表的内容。对于可更新的视图，视图中的行和基本表的行之间必须具有一对一的关系。还有一些特定结构的视图不可更新，不可更新的视图结构如下：

1. 聚合函数 SUM()、MIN()、MAX()、COUNT() 等
2. DISTINCT 关键字
3. GROUP BY 子句
4. HAVING 子句
5. UNION 或 UNION ALL 运算符
6. 位于选择列表中的子查询
7. FROM 子句中的不可更新视图或包含多个表
8. WHERE 子句中的子查询，引用 FROM 子句中的表
9. ALGORITHM 选项为 TEMPTABLE（使用临时表总会使视图成为不可更新的）的时候

---

操作示例 2：使用 UPDATE 语句更新视图 view_students_info（PS：修改完视图数据后，基表的数据也被修改了）

```sql
mysql> select * from view_students_info;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | Dany   |   24 |
+----+--------+------+
1 rows in set (0.00 sec)

mysql> update view_students_info set age=25 where id=1;
Query OK, 0 rows affected (0.24 sec)
Rows matched: 1  Changed: 0  Warnings: 0

mysql> select * from view_students_info where id = 1;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | Dany   |   25 |
+----+--------+------+
1 rows in set (0.00 sec)
```

> 3、修改视图名称：修改视图的名称可以先将视图删除，然后按照相同的定义语句进行视图的创建，并命名为新的视图名称





## 5、删除视图

```sql
-- 视图删除语法：
drop view [IF EXISTS] <视图名称> [ ,<视图名2>...];
```

操作示例：

```sql
mysql> drop view if exists v_students_info;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW CREATE VIEW v_students_info;
ERROR 1146 (42S02): Table 'test.v_students_info' doesn't exist
```



# 索引

## 1、索引介绍

> 参考资料：https://blog.csdn.net/qq_41573234/article/details/80250279

什么是索引：索引是一个单独的，存储在磁盘中上的数据库结构，它们包含着对数据表里的所有记录的引用指针。使用索引可以快速的找出在某列或多列中有特定值的行。

索引的优点：

1. 通过创建唯一索引，来保证数据库表中的每一行数据的唯一性
2. 可以加快数据的检索速度
3. 可以保证表数据的完整性与准确性

索引的缺点：

1. 索引需要占用物理空间
2. 对表中的数据进行改动时，索引也需要跟着动态维护，降低了数据的维护速度

索引的常见类型：

1. 普通索引：index
2. 唯一索引：unique
3. 主键索引：primary key
4. 外键索引：foreign key
5. 全文索引：fulltext
6. 组合索引：由上面的2和或2个以上组成

---

下面我们先准备大量数据来测试索引效果。创建test表的语句：

```sql
create table test (
    id int(7) zerofill auto_increment not null,
    username varchar(20),
    servnumber varchar(30),
    password varchar(20),
    createtime datetime,
    primary key (id)
)default charset=utf8;
```

生成百万甚至千万级别表的SQL语句SHELL脚本：

```shell
#!/bin/bash

echo "请输入字段servnumber的值："
read serber
echo "请输入创建sql语句的数量："
read number

# char=`head /dev/urandom | tr -dc 0-9 | head -c 11`

for (( i=0;i<$number;i++ ))
        do
        pass=`head /dev/urandom | tr -dc a-z | head -c 8`
        let serber=serber+1
        echo "insert into test(id,username,servnumber,password,createtime) values('$i','user${i}','${serber}','$pass',now());" >>sql.txt

done
```

执行步骤：vim test.sh ==》执行shell脚本：sh test.sh ==》进行插数操作：source /home/dazhu/sql.txt



## 2、创建索引

MySQL 提供了三种创建索引的方法：

### 1、创建表时创建索引

```sql
CREATE TABLE table_name(
    属性名  数据类型[约束条件],
    属性名  数据类型
    [UNIQUE | FULLTEXT | SPATIAL]  INDEX | KEY [别名] (属性名1  [( 长度 )] [ASC|DESC])
);
```

参数说明如下：

1. UNIQUE：可选参数，表示索引为唯一索引
2. FULLTEXT：可选参数，表示索引为全文索引
3. SPATIAL：可选参数，表示索引为空间索引
4. INDEX 与 KEY：参数用于指定字段为索引的，用户在选择时，只需要选择其中的一种即可
5.  别名：为可选参数，其作用是给创建的索引取新名称
6. 属性名1：指索引对应的字段名称，该字段必须被预先定义
7. 长度：可选参数，其指索引的长度，必须是字符串类型才可以使用
8. ASC|DESC：可选参数，ASC 表示升序排列，DESC 表示降序排列



### 2、直接为表添加索引

```sql
-- 注意：假如没有指定索引名称时，会以默认的字段名为索引名称
alter table 表名 add index 索引名称 (字段名称);
alter table test add unique unique_username (username);

-- 修改表的同时为该表添加索引
alter table 表名 add index [<索引名称>] [<索引类型>] (<列名>,…)
alter table 表名 add PRIMARY KEY [<索引类型>] (<列名>,…)
alter table 表名 add FOREIGN KEY [<索引名称>] (<列名>,…)
alter table 表名 add UNIQUE [ INDEX | KEY] [<索引名>] [<索引类型>] (<列名>,…)
```



### 3、直接创建索引

可以使用 CREATE INDEX 语句在一个已有的表上创建索引，但该语句不能创建主键（PS：CREATE INDEX 可对表增加普通索引或UNIQUE索引）

```sql
-- 索引名：指定索引名。一个表可以创建多个索引，但每个索引在该表中的名称是唯一的。
-- 表名：指定要创建索引的表名。
-- 列名：指定要创建索引的列名。通常可以考虑将查询语句中在 JOIN 子句和 WHERE 子句里经常出现的列作为索引列。
-- 长度：可选参数，其指索引的长度，必须是字符串类型才可以使用。
-- ASC|DESC：可选项。ASC指定索引按照升序来排列，DESC指定索引按照降序来排列，默认为ASC
create <索引名> on <表名> ( <列名> [<长度>] [ASC|DESC] )

-- 创建普通索引
create index <索引名> on <表名> ( <列名> [<长度>] [ASC|DESC] )
-- 创建唯一索引
create unique index <索引名> on <表名> ( <列名> [<长度>] [ASC|DESC] )
```

参数说明如下：

1. `<索引名>`：指定索引名。一个表可以创建多个索引，但每个索引在该表中的名称是唯一的
2. `<表名>`：指定要创建索引的表名
3. `<列名>`：指定要创建索引的列名。通常可以考虑将查询语句中在 JOIN 子句和 WHERE 子句里经常出现的列作为索引列
4. `<长度>`：可选参数，其指索引的长度，必须是字符串类型才可以使用
5. `ASC|DESC`：可选项。ASC指定索引按照升序来排列，DESC指定索引按照降序来排列，默认为ASC



## 3、普通索引

普通索引（index）：顾名思义就是各类索引中最为普通的索引，主要任务就是提高查询速度。其特点是允许出现相同的索引内容，允许空值（null）

1、创建表的时候创建

```sql
-- 普通索引index
create table test (
    id int(7) zerofill auto_increment not null,
    username varchar(20),
    servnumber varchar(30),
    password varchar(20),
    createtime datetime,
    index (id)
)default charset=utf8;
```

2、直接为表添加索引

```sql
alter table test add index index_name (username);
```

3、直接创建索引

```sql
create index index_createtime on test (createtime);
```



## 4、唯一索引

唯一索引：（unique）顾名思义就是不可以出现相同的索引内容，但是可以为空值（null）

1、创建表的时候创建

```sql
-- 唯一索引unique
create table test (
    id int(7) zerofill auto_increment not null,
    username varchar(20),
    servnumber varchar(30),
    password varchar(20),
    createtime datetime,
    unique index (id)
    -- unique index unique_id(id)  // unique_id为索引名称
)default charset=utf8;
```

2、直接为表添加索引

```sql
alter table test add unique unique_username (username);
```

3、直接创建索引

```sql
create unique index index_createtime on test (createtime);
```



## 5、主键索引

主键：主键是表的某一列，这一列的值是用来标志表中的每一行数据的。注意：每一张表只能拥有一个主键

主键索引（PRIMARY KEY）：把主键添加索引就是主键索引，它是一种特殊的唯一索引，不允许有空值，而唯一索引（unique是允许为空值的）

1、创建表的时候创建主键索引

```sql
-- 创建主键索引方式一
create table test (
    id int(7) auto_increment,-- 创建主键索引
    username varchar(20),
    password varchar(20)
)DEFAULT CHARSET=utf8;

-- 创建主键索引方式二
create table test (
    id int(7) primary key auto_increment,
    username varchar(20),
    password varchar(20)
    PRIMARY KEY (id) -- 创建主键索引
)DEFAULT CHARSET=utf8;
```

2、直接为表添加主键索引

```sql
alter table 表名 add primary key (字段名);
alter table test add primary key (id);
```

3、删除之间索引

```sql
-- 注意：在有自增的情况下，必须先删除自增，才可以删除主键
alter table test drop primary key;

-- 删除自增：
alter table test change id id int(7) unsigned zerofill not null;
```



## 6、外键约束

什么是外键（foreign key）：外键就是作用于两个表数据之间的链接的一列或多列，用来保证表与表之间的数据的完整性和准确性

增加外键约束语法

```sql
-- 注意：主键跟外键的字段类型一定要相同
... foreign key (字段名) references 关联的表名(关联表的字段名)
```

1、创建表的时候创建外键约束

```sql
-- 1、create table的方法：
CREATE TABLE `employee` (
    `empno` int(11) NOT NULL COMMENT '雇员编号',
    `ename` varchar(50) DEFAULT NULL COMMENT '雇员姓名',
    `job` varchar(30) DEFAULT NULL,
    `mgr` int(11) DEFAULT NULL COMMENT '雇员上级编号',
    `hiredate` date DEFAULT NULL COMMENT '雇佣日期',
    `sal` decimal(7,2) DEFAULT NULL COMMENT '薪资',
    `deptnu` int(11) DEFAULT NULL COMMENT '部门编号',
    PRIMARY KEY (`empno`),
    foreign key (`deptnu`) references dept(deptnu)
) ENGINE=InnoDB DEFAULT CHARSET=utf8; 
```

2、直接为表添加主键索引

```sql
 -- 2、alter table的方法：
 alter table employee add foreign key (deptnu) references dept(deptnu);
```

3、删除外键约束：（注意：在干掉外键索引之前必须先把外键约束删除，才能删除索引）

```sql
mysql> alter table employee drop index deptnu;
ERROR 1553 (HY000): Cannot drop index 'deptnu': needed in a foreign key constraint
mysql> 
mysql> alter table employee drop foreign key employee_ibfk_1;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> 
mysql> alter table employee drop index deptnu;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

注意点总结：

- 俩个表，主键跟外键的字段类型一定要相同​
- 要使用外键约束表的引擎一定得是InnoDB引擎，MyISAM是不起作用的​
- 在干掉外键索引之前必须先把外键约束删除，才能删除索引



## 7、全文索引

全文索引（fulltex）：全文索引是将存储在数据库中的文章或者句子等任意内容信息查找出来的索引，单位是词。也是目前搜索引擎使用的一种关键技术

全文索引只能作用在 CHAR、VARCHAR、TEXT、类型的字段上。创建全文索引需要使用 FULLTEXT 参数进行约束

1、创建练习表的SQL并插入数据：

```sql
-- 建表
create table command (
    id int(5) unsigned primary key auto_increment,
    name varchar(10),
    instruction varchar(60)
)engine=MyISAM;

-- 数据插入
insert into command values('1','ls','list directory contents');
insert into command values('2','wc','print newline, word, and byte counts for each file');
insert into command values('3','cut','remove sections from each line of files');
insert into command values('4','sort','sort lines of text files');
insert into command values('5','find','search for files in a directory hierarchy');
insert into command values('6','cp','复制文件或者文件夹');
insert into command values('7','top','display Linux processes');
insert into command values('8','mv','修改文件名，移动');
insert into command values('9','停止词','is,not,me,yes,no ...');
```

2、添加全文索引：

```sql
-- 方式一：创建表的时候创建全文索引，索引名称为command_instruction
create table command (
    id int(5) unsigned primary key auto_increment,
    name varchar(10),
    instruction varchar(60),
    fulltext key command_instruction(instruction)
)engine=MyISAM;

-- 方式二：直接为表添加全文索引
alter table command add fulltext(instruction);
```

3、使用全文索引：

```sql
select * from 表名 where match (字段名) against ('检索内容');
select * from command where match(instruction) against ('sections');
```

4、查看匹配度：

```sql
select * from command where match(instruction) against ('directory');
```

1. 停止词：出现频率很高的词，将会使全文索引失效
2. in boolean mode 模式：

```sql
in boolean mode：意思是指定全文检索模式为布尔全文检索（简单可以理解为是检索方式）
select * from 表名 where match (字段名) against ('检索内容' in boolean mode);
```

PS：注意点：使用通配符*时，只能放在词的后边，不能放前边。

5、删除全文索引：

```sql
alter table command drop index instruction;
```

注意点总结：

1. 一般情况下创建全文索引的字段数据类型为 char、varchar、text 。其它字段类型不可以​
2. 全文索引不针对非常频繁的词做索引。比如 is，no，not，you，me，yes这些，我们称之为停止词​
3. 对英文检索时忽略大小写



## 8、联合索引

联合索引：联合索引又称组合索引或者复合索引，是建立在俩列或者多列以上的索引。

1、创建和删除联合索引

```sql
-- 方式一：建表时创建索引
create table test (
    id int(7) primary key auto_increment not null,
    username varchar(20),
    servnumber varchar(30),
    password varchar(20),
    createtime datetime,
    index (username,servnumber,password)
)DEFAULT CHARSET=utf8;

-- 方式二：增加表联合索引
alter table 表名 add index(字段1,字段2,字段3);
alter table test add index(username,servnumber,password);
```

2、删除联合索引

```sql
-- 删除联合索引
alter table test drop index username;
```

为什么要使用联合索引，而不使用多个单列索引？答：联合索引的效率远远高于单列索引

联合索引的最左原则：只有查询条件中使用了这些字段中第一个字段(即username)，索引才会被使用。

注意点总结：

1. 索引并非越多越好，过多的索引会增加数据的维护速度还有磁盘空间的浪费
2. 当表的数据量很大的时候，可以考虑建立索引
3. 表中经常查数据的字段，可以考虑建立索引
4. 想要保证表中数据的唯一性，可以考虑建立唯一索引
5. 想要保证俩张表中的数据的完整性跟准确性，可以考虑建立外键约束
6. 经常对多列数据进行查询时，可以考虑建立联合索引



## 9、空间索引

创建空间索引时，需要设置 SPATIAL 参数。同样，必须说明的是，只有 MyISAM 类型表支持该类型索引。而且，索引字段必须有非空约束。

创建一个名称为 list 的数据表，并创建一个名为 listinfo 的空间索引，SQL语句如下:

```sql
create table list(
    id int(11) primary  ey auto_increment not null,
    goods geometry not null,
    SPATIAL INDEX listinfo(goods)
)engine=MyISAM;
```

 goods  字段上已经建立名称为  listinfo 的空间索引，其中  goods  字段必须不能为空，且数据类型是  GEOMETRY，该类型是空间数据类型。空间类型不能用其他类型代替，否则在生成空间素引时会产生错误且不能正常创建该类型索引。

空间类型除了上述示例中提到的 GEOMETRY 类型外，还包括如  POINT、LINESTRING、POLYGON  等类型，这些空间教据类型在平常的操作中很少被用到



## 10、查看索引

索引创建完成后，可以利用 SQL 语句查看已经存在的索引，查看索引的语法格式如下：

```sql
SHOW INDEX FROM <表名> [ FROM <数据库名>]
```

操作示例：

```sql
mysql> SHOW INDEX FROM tb_stu_info2\G
*************************** 1. row ***************************
        Table: tb_stu_info2
   Non_unique: 0
     Key_name: height
 Seq_in_index: 1
  Column_name: height
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment:
Index_comment:
1 row in set (0.03 sec)
```

其中各主要参数说明如下：

| 参数         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| Table        | 表示创建索引的数据表名，这里是 tb_stu_info2 数据表           |
| Non_unique   | 表示该索引是否是唯一索引。若不是唯一索引，则该列的值为 1；若是唯一索引，则该列的值为 0 |
| Key_name     | 表示索引的名称                                               |
| Seq_in_index | 表示该列在索引中的位置，如果索引是单列的，则该列的值为 1；如果索引是组合索引，则该列的值为每列在索引定义中的顺序 |
| Column_name  | 表示定义索引的列字段                                         |
| Collation    | 表示列以何种顺序存储在索引中。在 MySQL 中，升序显示值“A”（升序），若显示为 NULL，则表示无分类 |
| Cardinality  | 索引中唯一值数目的估计值。基数根据被存储为整数的统计数据计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL 使用该索引的机会就越大 |
| Sub_part     | 表示列中被编入索引的字符的数量。若列只是部分被编入索引，则该列的值为被编入索引的字符的数目；若整列被编入索引，则该列的值为 NULL。 |
| Packed       | 指示关键字如何被压缩。若没有被压缩，值为 NULL                |
| Null         | 用于显示索引列中是否包含 NULL。若列含有 NULL，该列的值为 YES。若没有，则该列的值为 NO |
| Index_type   | 显示索引使用的类型和方法（BTREE、FULLTEXT、HASH、RTREE）     |
| Comment      | 显示评注                                                     |



## 11、删除索引

当不再需要索引时，可以使用 DROP INDEX 语句或 ALTER TABLE 语句来对索引进行删除。

1、使用 DROP INDEX 语句，删除索引语法格式：

```sql
-- 索引名：要删除的索引名。表名：指定该索引所在的表名。
drop index <索引名称> on 表名;
```



2、使用 ALTER TABLE 语句，该语句也可以用于删除索引。具体使用方法是将 ALTER TABLE 语句的语法中部分指定为以下子句中的某一项

```sql
alter table 表名 drop index 索引名;
alter table 表名 DROP PRIMARY KEY
alter table 表名 DROP FOREIGN KEY fk_symbol
```

1. DROP PRIMARY KEY：表示删除表中的主键。一个表只有一个主键，主键也是一个索引
2. DROP INDEX index_name：表示删除名称为 index_name 的索引
3. DROP FOREIGN KEY fk_symbol：表示删除外键

> 注意：如果删除的列是索引的组成部分，那么在删除该列时，也会将该列从索引中删除；如果组成索引的所有列都被删除，那么整个索引将被删除。

操作示例 1：

```sql
mysql> DROP INDEX height ON tb_stu_info;
Query OK, 0 rows affected (0.27 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE tb_stu_info\G
*************************** 1. row ***************************
       Table: tb_stu_info
Create Table: CREATE TABLE `tb_stu_info` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```

操作示例 2：

```sql
mysql> ALTER TABLE tb_stu_info2 DROP INDEX height;
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE tb_stu_info2\G
*************************** 1. row ***************************
       Table: tb_stu_info2
Create Table: CREATE TABLE `tb_stu_info2` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```

# 存储过程

## 1、存储过程介绍

我们前面所学习的 MySQL 语句都是针对一个表或几个表的单条 SQL 语句，但是在数据库的实际操作中，经常会有需要多条 SQL 语句处理多个表才能完成的操作。例如，为了确认学生能否毕业，需要同时查询学生档案表、成绩表和综合表，此时就需要使用多条 SQL 语句来针对这几个数据表完成处理要求。

存储过程是一组为了完成特定功能的 SQL 语句集合。使用存储过程的目的是将常用或复杂的工作预先用 SQL 语句写好并用一个指定名称存储起来，这个过程经编译和优化后存储在数据库服务器中，因此称为存储过程。当以后需要数据库提供与已定义好的存储过程的功能相同的服务时，只需调用“CALL存储过程名字”即可自动完成。

简单说什么是存储过程：存储过程就是把复杂的一系列操作，封装成一个过程。类似于shell，python脚本等。

常用操作数据库的 SQL 语句在执行的时候需要先编译，然后执行。存储过程则采用另一种方式来执行 SQL 语句。

一个存储过程是一个可编程的函数，它在数据库中创建并保存，一般由 SQL 语句和一些特殊的控制结构组成。当希望在不同的应用程序或平台上执行相同的特定功能时，存储过程尤为合适。

MySQL 5.0 版本以前并不支持存储过程，这使 MySQL 在应用上大打折扣。MySQL 从 5.0 版本开始支持存储过程，既提高了数据库的处理速度，同时也提高了数据库编程的灵活性

存储过程是数据库中的一个重要功能，存储过程可以用来转换数据、数据迁移、制作报表，它类似于编程语言，一次执行成功，就可以随时被调用，完成指定的功能操作。

使用存储过程不仅可以提高数据库的访问效率，同时也可以提高数据库使用的安全性。

对于调用者来说，存储过程封装了 SQL 语句，调用者无需考虑逻辑功能的具体实现过程。只是简单调用即可，它可以被 Java 和 C# 等编程语言调用。

编写存储过程对开发者要求稍微高一些，但这并不影响存储过程的普遍使用，因为存储过程有如下优点：

1. 封装性：通常完成一个逻辑功能需要多条 SQL 语句，而且各个语句之间很可能传递参数，所以，编写逻辑功能相对来说稍微复杂些，而存储过程可以把这些 SQL 语句包含到一个独立的单元中，使外界看不到复杂的 SQL 语句，只需要简单调用即可达到目的。并且数据库专业人员可以随时对存储过程进行修改，而不会影响到调用它的应用程序源代码
2. 可增强 SQL 语句的功能和灵活性：存储过程可以用流程控制语句编写，有很强的灵活性，可以完成复杂的判断和较复杂的运算
3. 可减少网络流量：由于存储过程是在服务器端运行的，且执行速度快，因此当客户计算机上调用该存储过程时，网络中传送的只是该调用语句，从而可降低网络负载
4. 高性能：当存储过程被成功编译后，就存储在数据库服务器里了，以后客户端可以直接调用，这样所有的 SQL 语句将从服务器执行，从而提高性能。但需要说明的是，存储过程不是越多越好，过多的使用存储过程反而影响系统性能
5. 提高数据库的安全性和数据的完整性：存储过程提高安全性的一个方案就是把它作为中间组件，存储过程里可以对某些表做相关操作，然后存储过程作为接口提供给外部程序。这样，外部程序无法直接操作数据库表，只能通过存储过程来操作对应的表，因此在一定程度上，安全性是可以得到提高的
6. 使数据独立：数据的独立可以达到解耦的效果，也就是说，程序可以调用存储过程，来替代执行多条的 SQL 语句。这种情况下，存储过程把数据同用户隔离开来，优点就是当数据表的结构改变时，调用表不用修改程序，只需要数据库管理者重新编写存储过程即可



## 2、创建存储过程

存储过程是一些 SQL 语句的集合，比如有时候我们可能需要一大串的 SQL 语句，或者说在编写 SQL 语句的过程中需要设置一些变量的值，这个时候我们就完全有必要编写一个存储过程。编写存储过程并不是件简单的事情，但是使用存储过程可以简化操作，且减少冗余的操作步骤，同时，还可以减少操作过程中的失误，提高效率，因此应该尽可能的学会使用存储过程。

创建存储过程语法：

```sql
CREATE PROCEDURE <过程名> ( [过程参数[,…] ] ) 
<过程体> [过程参数[,…] ] 格式
[ IN | OUT | INOUT ] <参数名> <类型>

-- 简约版
create procedure 过程名 (参数....)
    begin
        过程体;
        过程体;
    end
```

语法参数说明如下：

1、过程名：

存储过程的名称，默认在当前数据库中创建。若需要在特定数据库中创建存储过程，则要在名称前面加上数据库的名称，即 db_name.sp_name。
需要注意的是，名称应当尽量避免选取与 MySQL 内置函数相同的名称，否则会发生错误。

2、过程参数：

存储过程的参数列表。其中，`<参数名>`为参数名，`<类型>`为参数的类型（可以是任何有效的 MySQL 数据类型）。当有多个参数时，参数列表中彼此间用逗号分隔。存储过程可以没有参数（此时存储过程的名称后仍需加上一对括号），也可以有 1 个或多个参数。

MySQL 存储过程支持三种类型的参数，即输入参数、输出参数和输入/输出参数，分别用 IN、OUT 和 INOUT 三个关键字标识。其中，输入参数可以传递给一个存储过程，输出参数用于存储过程需要返回一个操作结果的情形，而输入/输出参数既可以充当输入参数也可以充当输出参数。

需要注意的是，参数的取名不要与数据表的列名相同，否则尽管不会返回出错信息，但是存储过程的 SQL 语句会将参数名看作列名，从而引发不可预知的结果。

3、过程体：

存储过程的主体部分，也称为存储过程体，包含在过程调用的时候必须执行的 SQL 语句。这个部分以关键字 `BEGIN` 开始，以关键字 `END` 结束。若存储过程体中只有一条 SQL 语句，则可以省略 BEGIN-END 标志。

---

在存储过程的创建中，经常会用到一个十分重要的 MySQL 命令，即 DELIMITER 命令，特别是对于通过命令行的方式来操作 MySQL 数据库的使用者，更是要学会使用该命令。

在 MySQL 中，服务器处理 SQL 语句默认是以分号作为语句结束标志的。然而，在创建存储过程时，存储过程体可能包含有多条 SQL 语句，这些 SQL 语句如果仍以分号作为语句结束符，那么 MySQL 服务器在处理时会以遇到的第一条 SQL 语句结尾处的分号作为整个程序的结束符，而不再去处理存储过程体中后面的 SQL 语句，这样显然不行。

为解决以上问题，通常使用 `DELIMITER` 命令将结束命令修改为其他字符。语法格式如下：

```sql
DELIMITER $$
```

语法说明如下：

1. $$ 是用户定义的结束符，通常这个符号可以是一些特殊的符号，如两个“?”或两个“￥”等。
2. 当使用 DELIMITER 命令时，应该避免使用反斜杠“\”字符，因为它是 MySQL 的转义字符

若希望换回默认的分号“;”作为结束标志，则在 MySQL 命令行客户端输入下列语句即可：

```sql
mysql > DELIMITER ;
```



操作示例 1：创建名称为 ShowStuScore 的存储过程，存储过程的作用是从学生成绩信息表中查询学生的成绩信息

```sql
mysql> DELIMITER //
mysql> CREATE PROCEDURE ShowStuScore()
    -> BEGIN
    -> SELECT * FROM tb_students_score;
    -> END //
Query OK， 0 rows affected (0.09 sec)
```

操作示例 2：创建名为 GetScoreByStu 的存储过程，输入参数是学生姓名。存储过程的作用是通过输入的学生姓名从学生成绩信息表查询指定学生的成绩信息

```sql
mysql> DELIMITER //
mysql> CREATE PROCEDURE GetScoreByStu (IN name VARCHAR(30))
    -> BEGIN
    -> SELECT student_score FROM tb_students_score
    -> WHERE student_name=name;
    -> END //
Query OK, 0 rows affected (0.01 sec)
```





## 3、查看存储过程

1、查看存储过程的状态，通过 SHOW STATUS 语句查看存储过程的状态，语法如下：

```sql
-- LIKE 存储过程名：用来匹配存储过程的名称，LIKE 不能省略
SHOW PROCEDURE STATUS LIKE 存储过程名;
```

操作示例 1：创建数据表 studentinfo，向数据表 studentinfo 中插入数据，创建存储过程 showstuscore，然后查询名为 showstuscore 的存储过程的状态

```sql
-- 1.创建数据表 studentinfo
CREATE TABLE `studentinfo` (
    `ID` int(11) NOT NULL,
    `NAME` varchar(20) DEFAULT NULL,
    `SCORE` decimal(4,2) DEFAULT NULL,
    `SUBJECT` varchar(20) DEFAULT NULL,
    `TEACHER` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`ID`)
);

-- 2.插入数据
mysql> INSERT INTO studentinfo(id,name,score) VALUES(1,"zhangsan",80),(2,"lisi","70");
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

-- 3.创建存储过程，名称为showstuscore
mysql> DELIMITER //
mysql> CREATE PROCEDURE showstuscore()
    -> BEGIN
    -> SELECT id,name,score FROM studentinfo;
    -> END //
Query OK, 0 rows affected (0.07 sec)

-- 4.查询名为 showstuscore 的存储过程的状态
mysql> SHOW PROCEDURE STATUS LIKE 'showstuscore' \G
*************************** 1. row ***************************
                  Db: test
                Name: showstuscore
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2020-02-20 13:34:50
             Created: 2020-02-20 13:34:50
       Security_type: DEFINER
             Comment:
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.01 sec)

mysql> SHOW PROCEDURE STATUS LIKE 'show%' \G
*************************** 1. row ***************************
                  Db: test
                Name: showstuscore
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2020-02-21 09:34:50
             Created: 2020-02-21 09:34:50
       Security_type: DEFINER
             Comment:
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.00 sec)
```

查询存储过程的状态，查询结果显示了存储过程的创建时间、修改时间和字符集等信息。



2、查看存储过程的定义，通过 SHOW CREATE 语句查看存储过程的状态，语法如下：

```sql
SHOW CREATE PROCEDURE 存储过程名;
```

操作示例 2： 使用 SHOW CREATE查询名为 showstuscore 的存储过程的状态

```sql
mysql> SHOW CREATE PROCEDURE showstuscore \G
*************************** 1. row ***************************
           Procedure: showstuscore
            sql_mode: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `showstuscore`()
BEGIN
SELECT id,name,score FROM studentinfo;
END
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.01 sec)
```

查询结果显示了存储过程的定义和字符集信息等。

SHOW STATUS 语句只能查看存储过程是操作的哪一个数据库、存储过程的名称、类型、谁定义的、创建和修改时间、字符编码等信息。但是，这个语句不能查询存储过程的集体定义，如果需要查看详细定义，需要使用 SHOW CREATE 语句。

> 拓展阅读：
>
> 存储过程的信息都存储在 information_schema 数据库下的 Routines 表中，可以查询该表的记录来查询存储过程的信息，一定要使用 routine_name 字段指定存储过程的名称，否则，将查询出所有的存储过程的定义：
>
> ```sql
> SELECT * FROM information_schema.Routines WHERE ROUTINE_NAME=存储过程名;
> ```





## 4、修改存储过程

修改存储过程的语法格式如下：

```sql
ALTER PROCEDURE 存储过程名 [ 特征 ... ]
```

`特征`指定了存储过程的特性，可能的取值有：

1. CONTAINS SQL：表示子程序包含 SQL 语句，但不包含读或写数据的语句。
2. NO SQL：表示子程序中不包含 SQL 语句。
3. READS SQL DATA：表示子程序中包含读数据的语句。
4. MODIFIES SQL DATA：表示子程序中包含写数据的语句。
5. SQL SECURITY { DEFINER |INVOKER }：指明谁有权限来执行。
6. DEFINER：表示只有定义者自己才能够执行。
7. INVOKER：表示调用者可以执行。
8. COMMENT 'string'：表示注释信息。

操作示例：修改存储过程 showstuscore 的定义，将读写权限改为 MODIFIES SQL DATA，并指明调用者可以执行，代码如下：

```sql
mysql> ALTER PROCEDURE showstuscore MODIFIES SQL DATA SQL SECURITY INVOKER;
Query OK, 0 rows affected (0.01 sec)
```

执行代码，并查看修改后的信息，运行结果如下：

```sql
mysql> SHOW CREATE PROCEDURE showstuscore \G
*************************** 1. row ***************************
           Procedure: showstuscore
            sql_mode: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `showstuscore`()
    MODIFIES SQL DATA
    SQL SECURITY INVOKER
BEGIN
SELECT id,name,score FROM studentinfo;
END
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.00 sec)
```

结果显示，存储过程修改成功。从运行结果可以看到，访问数据的权限已经变成了 MODIFIES SQL DATA，安全类型也变成了 INVOKE。

提示：ALTER PROCEDURE 语句用于修改存储过程的某些特征。如果要修改存储过程的内容，可以先删除原存储过程，再以相同的命名创建新的存储过程；如果要修改存储过程的名称，可以先删除原存储过程，再以不同的命名创建新的存储过程。



## 5、删除存储过程

使用 DROP PROCEDURE 语句来删除数据库中已经存在的存储过程。语法格式如下：

```sql
-- 过程名：指定要删除的存储过程的名称
-- IF EXISTS：指定这个关键字，用于防止因删除不存在的存储过程而引发的错误
DROP PROCEDURE [ IF EXISTS ] <过程名>
```

注意：存储过程名称后面没有参数列表，也没有括号，在删除之前，必须确认该存储过程没有任何依赖关系，否则会导致其他与之关联的存储过程无法运行。

操作示例：下面删除存储过程 ShowStuScore，SQL 语句和运行结果如下：

```sql
mysql> DROP PROCEDURE ShowStuScore;
Query OK, 0 rows affected (0.08 sec)
```

删除后，可以通过查询 information_schema 数据库下的 routines 表来确认上面的删除是否成功。SQL 语句和运行结果如下：

```sql
mysql> SELECT * FROM information_schema.routines WHERE routine_name='ShowStuScore';
Empty set (0.03 sec)
```

结果显示，没有查询出任何记录，说明存储过程 ShowStuScore 已经被删除了



## 6、调用存储过程

调用存储过程后，数据库系统将执行存储过程中的 SQL 语句，然后将结果返回给输出值。CALL 语句接收存储过程的名字以及需要传递给它的任意参数。

```sql
-- sp_name 表示存储过程的名称， parameter 表示存储过程的参数
CALL sp_name([parameter[...]]);
```

操作示例：

```sql
mysql> DELIMITER ;
mysql> call ShowStuScore();
+--------------+---------------+
| student_name | student_score |
+--------------+---------------+
| Dany         |            90 |
| Green        |            99 |
| Henry        |            95 |
| Jane         |            98 |
| Jim          |            88 |
| John         |            94 |
| Lily         |           100 |
| Susan        |            96 |
| Thomas       |            93 |
| Tom          |            89 |
+--------------+---------------+
10 rows in set (0.00 sec)
Query OK, 0 rows affected (0.02 sec)

mysql> call GetScoreByStu('Green');
+---------------+
| student_score |
+---------------+
|            99 |
+---------------+
1 row in set (0.03 sec)
Query OK, 0 rows affected (0.03 sec)
```

因为存储过程实际上也是一种函数，所以存储过程名后需要有`( )`符号，即使不传递参数也需要。

> 简洁版语法和示例：
>
> ```sql
> -- in|out|inout 参数名称 类型（长度）
> -- in：表示调用者向过程传入值（传入值可以是字面量或变量）
> -- out：表示过程向调用者传出值(可以返回多个值)（传出值只能是变量）
> -- inout：既表示调用者向过程传入值，又表示过程向调用者传出值（值只能是变量）
> create procedure 名称 (参数....) 过程体
> create procedure 名称 (参数....)
>     begin
>         过程体;
>         过程体;
>     end
> 
> -- 声明变量
> declare 变量名 类型(长度) default 默认值;
> -- 给变量赋值
> set @变量名=值;
> -- 调用存储命令
> call 名称(@变量名);
> -- 删除存储过程命令
> drop procedure 名称;
> -- 查看创建的存储过程命令
> show create procedure 名称\G
> ```
>
> ```sql
> # 创建一个带参数的存储过程：
> mysql> delimiter //
> mysql> create procedure name(in n int)
>     -> begin
>     -> select * from employee limit n;
>     -> end
>     -> //
> Query OK, 0 rows affected (0.00 sec)
> 
> # 调用存储过程
> mysql> set @n=5;
>     -> //
> Query OK, 0 rows affected (0.00 sec)
> mysql> call name(@n);
> 
> 
> 
> 
> # 创建一个无参数的存储过程：
> mysql> delimiter //
> mysql> create procedure name()
>     -> begin
>     -> declare  n int default 6;
>     -> select * from employee limit n;
>     -> end
>     -> //
> Query OK, 0 rows affected (0.00 sec)
> 
> # 调用存储过程
> mysql> call name();
> ```





## 7、存储函数实战

存储函数和存储过程一样，都是在数据库中定义一些 SQL 语句的集合。存储函数可以通过 return 语句返回函数值，主要用于计算并返回一个值。而存储过程没有直接返回值，主要用于执行操作。

1、使用 CREATE FUNCTION 语句来创建存储函数，其语法形式如下：

```sql
CREATE FUNCTION sp_name ([func_parameter[...]])
RETURNS type
[characteristic ...] routine_body
```

参数说明：

1. sp_name 参数：表示存储函数的名称；
2. func_parameter：表示存储函数的参数列表；
3. RETURNS type：指定返回值的类型；
4. characteristic 参数：指定存储函数的特性，该参数的取值与存储过程是一样的；
5. routine_body 参数：表示 SQL 代码的内容，可以用 BEGIN...END 来标示 SQL 代码的开始和结束。

注意：在具体创建函数时，函数名不能与已经存在的函数名重名。除了上述要求外，推荐函数名命名（标识符）为 function_xxx 或者 func_xxx。

func_parameter 可以由多个参数组成，其中每个参数由参数名称和参数类型组成，其形式如下：

```sql
[IN | OUT | INOUT] param_name type;
```

参数说明：

1. IN 表示输入参数，OUT 表示输出参数，INOUT 表示既可以输入也可以输出；
2. param_name 参数是存储函数的参数名称；
3. type 参数指定存储函数的参数类型，该类型可以是 MySQL 数据库的任意数据类型。

操作示例 1：使用 CREATE FUNCTION 创建查询 tb_student 表中某个学生姓名的函数，SQL 语句和执行过程如下：

```sql
mysql> USE test;
Database changed
mysql> DELIMITER //
mysql> CREATE FUNCTION func_student(id INT(11))
    -> RETURNS VARCHAR(20)
    -> COMMENT '查询某个学生的姓名'
    -> BEGIN
    -> RETURN(SELECT name FROM tb_student WHERE tb_student.id = id);
    -> END //
Query OK, 0 rows affected (0.10 sec)
mysql> DELIMITER ;
```

上述代码中，创建了 func_student 函数，该函数拥有一个类型为 INT(11) 的参数 id，返回值为 VARCHAR(20) 类型。SELECT 语句从 tb_student 表中查询 id 字段值等于所传入参数 id 值的记录，同时返回该条记录的 name 字段值。

创建函数与创建存储过程一样，需要通过命令 `DELIMITER //` 将 SQL 语句的结束符由“;”修改为“//”，最后通过命令 `DELIMITER ;` 将结束符号修改成 SQL 语句中默认的结束符号。

如果在存储函数中的 RETURN 语句返回一个类型不同于函数的 RETURNS 子句中指定类型的值，返回值将被强制为恰当的类型。比如，如果一个函数返回一个 ENUM 或 SET 值，但是 RETURN 语句返回一个整数，对于 SET 成员集的相应的 ENUM 成员，从函数返回的值是字符串。

---

2、查看存储函数的语法如下：（可以发现，操作存储函数和操作存储过程不同的是将 PROCEDURE 替换成了 FUNCTION）

```sql
SHOW FUNCTION STATUS LIKE 存储函数名;
SHOW CREATE FUNCTION 存储函数名;
SELECT * FROM information_schema.Routines WHERE ROUTINE_NAME = 存储函数名;
```

---

2、修改存储函数的语法如下：（存储函数的特征与存储过程的基本一样）

```sql
ALTER FUNCTION 存储函数名 [ 特征 ... ]
```

---

3、删除存储过程的语法如下：

```sql
DROP FUNCTION [ IF EXISTS ] <函数名>
```

---

4、调用存储函数

```sql
mysql> SELECT func_student(3);
+-----------------+
| func_student(3) |
+-----------------+
| 王五            |
+-----------------+
1 row in set (0.10 sec)
```

存储过程通过 CALL 语句来调用，存储函数的使用方法与 MySQL 内部函数的使用方法相同。执行存储过程和存储函数需要拥有 EXECUTE 权限（EXECUTE 权限的信息存储在 information_schema 数据库下的 USER_PRIVILEGES 表中）



# 触发器

## 1、触发器介绍

触发器和存储过程一样，都是嵌入到 MySQL 中的一段程序，是管理数据的有力工具。不同的是执行存储过程要使用 CALL 语句来调用，而触发器的执行不需要使用 CALL 语句来调用，也不需要手工启动，而是通过对数据表的相关操作来触发、激活从而实现执行。比如当对 student 表进行操作（INSERT，DELETE 或 UPDATE）时就会激活它执行。

触发器与数据表关系密切，主要用于保护表中的数据。特别是当有多个表具有一定的相互联系的时候，触发器能够让不同的表保持数据的一致性。

PS：只有执行 INSERT、UPDATE 和 DELETE 操作时才能激活触发器，其它 SQL 语句则不会激活触发器。

那么为什么要使用触发器呢？比如，在实际开发项目时，我们经常会遇到以下情况：

1. 在学生表中添加一条关于学生的记录时，学生的总数就必须同时改变
2. 增加一条学生记录时，需要检查年龄是否符合范围要求
3. 删除一条学生信息时，需要删除其成绩表上的对应记录
4. 删除一条数据时，需要在数据库存档表中保留一个备份副本

虽然上述情况实现的业务逻辑不同，但是它们都需要在数据表发生更改时，自动进行一些处理。这时就可以使用触发器处理。例如，对于第一种情况，可以创建一个触发器对象，每当添加一条学生记录时，就执行一次计算学生总数的操作，这样就可以保证每次添加一条学生记录后，学生总数和学生记录数是一致的。

> 触发器的优缺点

触发器的优点：

1. 触发器的执行是自动的，当对触发器相关表的数据做出相应的修改后立即执行
2. 触发器可以实施比 FOREIGN KEY 约束、CHECK 约束更为复杂的检查和操作
3. 触发器可以实现表数据的级联更改，在一定程度上保证了数据的完整性

触发器的缺点：

1. 使用触发器实现的业务逻辑在出现问题时很难进行定位，特别是涉及到多个触发器的情况下，会使后期维护变得困难
2. 大量使用触发器容易导致代码结构被打乱，增加了程序的复杂性，
3. 如果需要变动的数据量较大时，触发器的执行效率会非常低

---

> MySQL 支持的触发器

MySQL支持的触发器有三种：INSERT 触发器、UPDATE 触发器 和 DELETE 触发器

1、INSERT 触发器。在 INSERT 语句执行之前或之后响应的触发器。使用 INSERT 触发器需要注意以下几点：

1. 在 INSERT 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问被插入的行。
2. 在 BEFORE INSERT 触发器中，NEW 中的值也可以被更新，即允许更改被插入的值（只要具有对应的操作权限）。
3. 对于 AUTO_INCREMENT 列，NEW 在 INSERT 执行之前包含的值是 0，在 INSERT 执行之后将包含新的自动生成值。

---

2、UPDATE 触发器。在 UPDATE 语句执行之前或之后响应的触发器。使用 UPDATE 触发器需要注意以下几点：

1. 在 UPDATE 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问更新的值
2. 在 UPDATE 触发器代码内，可引用一个名为 OLD（不区分大小写）的虚拟表来访问 UPDATE 语句执行前的值
3. 在 BEFORE UPDATE 触发器中，NEW 中的值可能也被更新，即允许更改将要用于 UPDATE 语句中的值（只要具有对应的操作权限）
4. OLD 中的值全部是只读的，不能被更新

注意：当触发器设计对触发表自身的更新操作时，只能使用 BEFORE 类型的触发器，AFTER 类型的触发器将不被允许

---

3、DELETE 触发器。在 DELETE 语句执行之前或之后响应的触发器。使用 DELETE 触发器需要注意以下几点：

1. 在 DELETE 触发器代码内，可以引用一个名为 OLD（不区分大小写）的虚拟表来访问被删除的行
2. OLD 中的值全部是只读的，不能被更新

---

总体来说，触发器使用的过程中，MySQL 会按照以下方式来处理错误。

对于事务性表，如果触发程序失败，以及由此导致的整个语句失败，那么该语句所执行的所有更改将回滚；对于非事务性表，则不能执行此类回滚，即使语句失败，失败之前所做的任何更改依然有效。

若 BEFORE 触发程序失败，则 MySQL 将不执行相应行上的操作。

若在 BEFORE 或 AFTER 触发程序的执行过程中出现错误，则将导致调用触发程序的整个语句失败。

仅当 BEFORE 触发程序和行操作均已被成功执行，MySQL 才会执行 AFTER 触发程序。





## 2、创建触发器

使用 CREATE TRIGGER 语句创建触发器。语法格式如下：

```sql
-- before/after:可以设置为事件发生前或后
-- insert/update/delete:它们可以在执行insert、update或delete的过程中触发
-- for each row:每隔一行执行一次动作
CREATE <触发器名> 
< BEFORE | AFTER >
<INSERT | UPDATE | DELETE >
ON <表名> FOR EACH Row<触发器主体>
```

参数说明：

1. 触发器名：触发器的名称，触发器在当前数据库中必须具有唯一的名称。如果要在某个特定数据库中创建，名称前面应该加上数据库的名称。
2. INSERT | UPDATE | DELETE：触发事件，用于指定激活触发器的语句的种类
3. BEFORE | AFTER：BEFORE 和 AFTER，触发器被触发的时刻，表示触发器是在激活它的语句之前或之后触发
4. 触发器主体：触发器动作主体，包含触发器激活时将要执行的 MySQL 语句。如果要执行多个语句，可使用 BEGIN…END 复合语句结构
5. FOR EACH ROW：一般是指行级触发，对于受触发事件影响的每一行都要激活触发器的动作（每隔一行执行一次动作）
6. 表名：与触发器相关联的表名，此表必须是永久性表，不能将触发器与临时表或视图关联起来。在该表上触发事件发生时才会激活触发器。同一个表不能拥有两个具有相同触发时刻和事件的触发器。例如，对于一张数据表，不能同时有两个 BEFORE UPDATE 触发器，但可以有一个 BEFORE UPDATE 触发器和一个 BEFORE INSERT 触发器，或一个 BEFORE UPDATE 触发器和一个 AFTER UPDATE 触发器

注意：三种触发器的执行时间如下：

1. INSERT：将新行插入表时激活触发器。例如，INSERT 的 BEFORE 触发器不仅能被 MySQL 的 INSERT 语句激活，也能被 LOAD DATA 语句激活
2. DELETE： 从表中删除某一行数据时激活触发器，例如 DELETE 和 REPLACE 语句。
3. UPDATE：更改表中某一行数据时激活触发器，例如 UPDATE 语句。

> 注意：每个表都支持 INSERT、UPDATE 和 DELETE 的 BEFORE 与 AFTER，因此每个表最多支持 6 个触发器。每个表的每个事件每次只允许有一个触发器。单一触发器不能与多个事件或多个表关联。另外若需要查看数据库中已有的触发器，则可以使用 SHOW TRIGGERS 语句

---

1、创建 BEFORE 触发器

在 test_db 数据库中，数据表 tb_emp8 为员工信息表，包含 id、name、deptId 和 salary 字段，数据表 tb_emp8 的表结构如下所示

```sql
mysql> select * from tb_emp8;
Empty set (0.07 sec)
mysql> desc tb_emp8;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(22) | YES  | UNI | NULL    |       |
| deptId | int(11)     | NO   | MUL | NULL    |       |
| salary | float       | YES  |     | 0       |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.05 sec)
```

操作示例 1：创建一个名为 SumOfSalary 的触发器，触发的条件是向数据表 tb_emp8 中插入数据之前，对新插入的 salary 字段值进行求和计算。输入的 SQL 语句和执行过程如下所示。

```sql
-- new：指的是事件发生before或者after保存的新数据
mysql> create trigger SumOfSalary
    -> before insert on tb_emp8
    -> for each row
    -> set @sum = @sum + new.salary;
Query OK, 0 rows affected (0.35 sec)
```

触发器 SumOfSalary 创建完成之后，向表 tb_emp8 中插入记录时，定义的 sum 值由 0 变成了 1500，即插入值 1000 和 500 的和，如下所示。

```sql
SET @sum=0;
Query OK, 0 rows affected (0.05 sec)

mysql> insert into tb_emp8 values (1,'A',1,1000),(2,'B',1,500);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT @sum;
+------+
| @sum |
+------+
| 1500 |
+------+
1 row in set (0.03 sec)
```

---

2、创建 AFTER 类型触发器

在 test_db 数据库中，数据表 tb_emp6 和 tb_emp7 都为员工信息表，包含 id、name、deptId 和 salary 字段，数据表 tb_emp6 和 tb_emp7 的表结构如下所示

```sql
mysql> select * from tb_emp6;
Empty set (0.07 sec)

mysql> select * from tb_emp7;
Empty set (0.03 sec)

mysql> desc tb_emp6;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  | MUL | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

mysql> desc tb_emp7;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | 0       |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.04 sec)
```

操作示例 2：创建一个名为 double_salary 的触发器，触发的条件是向数据表 tb_emp6 中插入数据之后，再向数据表 tb_emp7 中插入相同的数据，并且 salary 为 tb_emp6 中新插入的 salary 字段值的 2 倍。输入的 SQL 语句和执行过程如下所示。

```sql
-- new：指的是事件发生before或者after保存的新数据
mysql> create trigger double_salary
    -> after insert on tb_emp6
    -> for each row
    -> insert into tb_emp7 values (new.id,new.name,new.deptId,2*new.salary);
Query OK, 0 rows affected (0.25 sec)
```

触发器 double_salary 创建完成之后，向表 tb_emp6 中插入记录时，同时向表 tb_emp7 中插入相同的记录，并且 salary 字段为 tb_emp6 中 salary 字段值的 2 倍，如下所示。

```sql
mysql> insert into tb_emp6 values (1,'A',1,1000),(2,'B',1,500);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from tb_emp6;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   1000 |
|  2 | B    |      1 |    500 |
+----+------+--------+--------+
3 rows in set (0.04 sec)

mysql> select * from tb_emp7;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   2000 |
|  2 | B    |      1 |   1000 |
+----+------+--------+--------+
2 rows in set (0.06 sec)
```



## 3、查看触发器

可以通过 SHOW TRIGGERS 语句来查看触发器的基本信息，语法格式如下：

```sql
-- 查看触发器信息（查看所有触发器）
SHOW TRIGGERS;
```

操作示例 1：创建名为 trigupdate 的触发器，每次 account 表更新数据之后都向 myevent 数据表中插入一条数据

```sql
-- 先创建数据表account，表中有两个字段，分别是 INT 类型的 accnum 和 DECIMAL 类型的 amount
create table account(
    accnum INT(4),
    amount DECIMAL(10,2)
);
-- 创建数据表myeven
create table myevent(
    id INT(11) DEFAULT NULL,
    evtname CHAR(20) DEFAULT NULL
);
```

```sql
-- 创建 trigupdate 触发器
mysql> create trigger trigupdate after update on account
    -> for each row
    -> insert into myevent values(1,'after update');
Query OK, 0 rows affected (0.15 sec)
```

使用 SHOW TRIGGERS 语句查看触发器（在 SHOW TRIGGERS 命令后添加`\G`，这样显示信息会比较有条理），SQL 语句和运行结果如下：

```sql
mysql> SHOW TRIGGERS \G
*************************** 1. row ***************************
             Trigger: trigupdate
               Event: UPDATE
               Table: account
           Statement: INSERT INTO myevent VALUES(1,'after update')
              Timing: AFTER
             Created: 2020-02-24 14:07:15.08
            sql_mode: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
             Definer: root@localhost
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.09 sec)
```

由运行结果可以看到触发器的基本信息。对以上显示信息的说明如下：

1. Trigger：表示触发器的名称，在这里触发器的名称为 trigupdate；
2. Event：表示激活触发器的事件，这里的触发事件为更新操作 UPDATE；
3. Table：表示激活触发器的操作对象表，这里为 account 表；
4. Statement：表示触发器执行的操作，这里是向 myevent 数据表中插入一条数据；
5. Timing：表示触发器触发的时间，这里为更新操作之后（AFTER）；
6. 还有一些其他信息，比如触发器的创建时间、SQL 的模式、触发器的定义账户和字符集等，这里不再一一介绍

PS：`SHOW TRIGGERS` 语句用来查看当前创建的所有触发器的信息。因为该语句无法查询指定的触发器，所以在触发器较少的情况下，使用该语句会很方便。如果要查看特定触发器的信息或者数据库中触发器较多时，可以直接从 information_schema 数据库中的 triggers 数据表中查找。

---

在 MySQL 中，所有触发器的信息都存在 information_schema 数据库的 triggers 表中，可以通过查询命令 SELECT 来查看，具体的语法如下：

```sql
select * from information_schema.triggers where trigger_name= '触发器名';
```

操作示例 2：使用 SELECT 命令查看 trigupdate 触发器，SQL 语句如下：

```sql
mysql> select * from information_schema.triggers where TRIGGER_NAME= 'trigupdate'\G
*************************** 1. row ***************************
           TRIGGER_CATALOG: def
            TRIGGER_SCHEMA: test
              TRIGGER_NAME: trigupdate
        EVENT_MANIPULATION: UPDATE
      EVENT_OBJECT_CATALOG: def
       EVENT_OBJECT_SCHEMA: test
        EVENT_OBJECT_TABLE: account
              ACTION_ORDER: 1
          ACTION_CONDITION: NULL
          ACTION_STATEMENT: INSERT INTO myevent VALUES(1,'after update')
        ACTION_ORIENTATION: ROW
             ACTION_TIMING: AFTER
ACTION_REFERENCE_OLD_TABLE: NULL
ACTION_REFERENCE_NEW_TABLE: NULL
  ACTION_REFERENCE_OLD_ROW: OLD
  ACTION_REFERENCE_NEW_ROW: NEW
                   CREATED: 2020-02-24 16:07:15.08
                  SQL_MODE: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
                   DEFINER: root@localhost
      CHARACTER_SET_CLIENT: gbk
      COLLATION_CONNECTION: gbk_chinese_ci
        DATABASE_COLLATION: latin1_swedish_ci
1 row in set (0.22 sec)
```

由运行结果可以看到触发器的详细信息。对以上显示信息的说明如下：

- TRIGGER_SCHEMA：表示触发器所在的数据库；
- TRIGGER_NAME：表示触发器的名称；
- EVENT_OBJECT_TABLE：表示在哪个数据表上触发；
- ACTION_STATEMENT：表示触发器触发的时候执行的具体操作；
- ACTION_ORIENTATION：的值为 ROW，表示在每条记录上都触发；
- ACTION_TIMING：表示触发的时刻是 AFTER；
- 还有一些其他信息，比如触发器的创建时间、SQL 的模式、触发器的定义账户和字符集等，这里不再一一介绍。



## 4、删除触发器

触发器删除的语法：

```sql
-- 触发器名：要删除的触发器名称。
-- 数据库名：可选项。指定触发器所在的数据库的名称。若没有指定，则为当前默认的数据库
-- IF EXISTS：可选项。避免在没有触发器的情况下删除触发器。
DROP TRIGGER [ IF EXISTS ] [数据库名] <触发器名>
```

注意：删除一个表的同时，也会自动删除该表上的触发器。另外，触发器不能更新或覆盖，为了修改一个触发器，必须先删除它，再重新创建。

操作示例：删除 double_salary 触发器，输入的 SQL 语句和执行过程如下所示。

```sql
mysql> DROP TRIGGER double_salary;
Query OK, 0 rows affected (0.03 sec)
```



> 简洁版语法和操作示例：
>
> ```sql
> -- before/after:可以设置为事件发生前或后
> -- insert/update/delete:它们可以在执行insert、update或delete的过程中触发
> -- for each row:每隔一行执行一次动作
> 
> -- 创建单个执行语句的触发器
> create trigger 触发器名称 before|after insert|update|delete
> on 表名 for each row sql语句;
> -- 创建有多个执行语句的触发器，不同语句用分号隔开  
> create trigger 触发器名称 before|after insert|update|delete
> on 表名 for each row
> begin
>     执行语句列表
> end
> 
> -- 查看触发器信息（查看所有触发器）
> SHOW TRIGGERS;
> -- 查看某个触发器，所有触发器的信息都存在 information_schema 数据库的 triggers 表中
> SELECT * FROM information_schema.triggers WHERE trigger_name= '触发器名';
> 
> -- 删除触发器
> drop trigger 触发器名称;
> ```
>
> ```sql
> -- 创建一个员工迟到表：
>  create table work_time_delay(
>     empno int not null comment '雇员编号',
>     ename varchar(50) comment '雇员姓名',
>     status int comment '状态'
> );
> 
> -- new：指的是事件发生before或者after保存的新数据
> mysql> delimiter //
> mysql> create trigger trig_work after insert on work_time_delay
>     -> for each row
>     -> begin
>     ->     update employee set sal=sal-100 where empno=new.empno;
>     -> end
>     -> //
> Query OK, 0 rows affected (0.01 sec)
> ```



# 存储引擎

主要来详细介绍MyISAM与InnoDB引擎

数据库存储引擎：是数据库底层软件组件，不同的存储引擎提供不同的存储机制，索引技巧，锁定水平等功能，使用不同的数据库引擎，可以获得特定的功能

1、如何查看引擎

```sql
-- 如何查看数据库支持的引擎
show engines;

-- 查看当前数据的引擎：
show create table 表名\G

-- 查看当前库所有表的引擎：
show table status\G
```

2、建表时指定引擎

```sql
create table yingqin (
    id int,
    name varchar(20)
) engine='InnoDB';-- (or MyISAM)
```

3、修改表的引擎

```sql
alter table 表名 engine='MyiSAm';
```

4、修改默认引擎

```shell
# 修改默认引擎，记得保存后重启服务
vi /etc/my.cnf
[mysqld] #增加下面配置
default-storage-engine=MyIsAM
```





# 查询语句的执行顺序

> mysql 查询语句的执行顺序(重重点)：https://www.cnblogs.com/Xanderzyl/p/11000354.html



# SQL 查询语句优化

## 1、慢查询日志开启与问题定位

简介：介绍如何开启慢查询日志与问题定位

- 第一步：查看是否已经开启了慢查询日志

```shell
mysql> show variables like 'slow%';
+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_launch_time    | 2                                    |
| slow_query_log      | OFF                                  |
| slow_query_log_file | /data/mydata/xdclass-public-slow.log |
+---------------------+--------------------------------------+
```

- 第二步：开启慢查询日志

```sql
set global slow_query_log = on;

-- 日志路径也可以自定义：
set global slow_query_log_file = '路径';
```

- 第三步：查看慢查询的时间临界值（查询慢的日志才会存入日志）

```shell
mysql> show variables like '%long%';
mysql> 
+----------------------------------------------------------+-----------+
| Variable_name                                            | Value     |
+----------------------------------------------------------+-----------+
| long_query_time                                          | 10.000000 |
| performance_schema_events_stages_history_long_size       | 10000     |
| performance_schema_events_statements_history_long_size   | 10000     |
| performance_schema_events_transactions_history_long_size | 10000     |
| performance_schema_events_waits_history_long_size        | 10000     |
+----------------------------------------------------------+-----------+
```

- 第四步：设置慢查询的时间标准(临界值)

```shell
set long_query_time=0.4;
```

- 注意：重启mysql服务会让在交互界面设置的慢查询恢复到默认
- 永久生效的设置方法：

```shell
修改配置文件 vi /etc/my.cnf
[mysqld]
slow_query_log = 1
long_query_time = 0.1
slow_query_log_file =/usr/local/mysql/mysql_slow.log

最后必须重启服务才能生效！
```



## 2、SQL语句执行过程解析

简介：介绍如何开启性能详情

- 第一步：查看性能详情是否开启

```shell
mysql> show variables like '%profiling%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| have_profiling         | YES   |
| profiling              | OFF   |
| profiling_history_size | 15    |
+------------------------+-------+
```

- 第二步：开启性能记录功能

```shell
set profiling = on ;
```

- 第三步：查看性能的记录

```shell
mysql> show profiles;
+----------+------------+---------------------------------------------------+
| Query_ID | Duration   | Query                                             |
+----------+------------+---------------------------------------------------+
|        1 | 0.00177775 | show variables like '%profiling%'                 |
|        2 | 0.00037900 | select * from test where id='087878'              |
|        3 | 0.34618025 | select * from test where servnumber='1367008787'  |
|        4 | 0.31986825 | select * from test where servnumber='13670087879' |
+----------+------------+---------------------------------------------------+
```

- 第四步：查看语句的执行性能详情

```shell
mysql> show profile for query 4;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000100 |
| checking permissions | 0.000010 |
| Opening tables       | 0.000023 |
| init                 | 0.000045 |
| System lock          | 0.000015 |
| optimizing           | 0.000016 |
| statistics           | 0.000028 |
| preparing            | 0.000020 |
| executing            | 0.000006 |
| Sending data         | 0.319489 |
| end                  | 0.000037 |
| query end            | 0.000012 |
| closing tables       | 0.000012 |
| freeing items        | 0.000040 |
| cleaning up          | 0.000017 |
+----------------------+----------+
```
性能线程的详细解释官方文档链接：https://dev.mysql.com/doc/refman/5.7/en/general-thread-states.html



## 3、SQL语句优化建议(面试)

日常工作应该尽量避免的SQL语句：

1. 第一个注意点：尽量避免使用select * from ，尽量精确到想要的结果字段
2. 第二个注意点：尽量避免条件使用or
3. 第三个注意点：记得加上limit 限制行数，避免数据量过大消耗性能
4. 第四个注意点：使用模糊查询时，%放在前面是会使索引失效

```sql
mysql> explain select * from test where servnumber like '%1367000%'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: test
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 996303
     filtered: 11.11
        Extra: Using where
```
1. 第五个注意点：要小心条件字段类型的转换（varchar类型少了引号会使索引失效）
2. https://www.cnblogs.com/leeego-123/p/11846613.html



# 数据安全与备份

## 1、备份的背景意义

1、数据备份的意义：

- 保护数据的安全；
- 在出现意外的时候（硬盘的损坏，断电，黑客的攻击），以便数据的恢复；
- 导出生产的数据以便研发人员或者测试人员测试学习；
- 高权限的人员操作失误导致数据丢失，以便恢复；

2、数据库的备份类型：

- 完全备份：对整个数据库的数据进行备份 
- 部分备份：对部分数据进行备份（可以是一张表也可以是多张表） 
- 增量备份：是以上一次备份为基础来备份变更数据的，节约空间 
- 差异备份：是以第一次完全备份的基础来备份变更备份的，浪费空间

3、数据库备份的方式：

- 逻辑备份：直接生成sql语句保存起来，在恢复数据的时候执行备份的sql语句来实现数据的恢复
- 物理备份：直接拷贝相关的物理数据

4、区别：

- 逻辑备份效率低，恢复数据效率低，但是逻辑备份节约空间；
- 物理备份浪费空间，但是相对逻辑备份而言效率比较高

5、数据库备份的场景：

- 热备份：备份时，数据库的读写操作不会受到影响
- 温备份：备份时，数据库的读操作可以进行，但是写操作不能执行
- 冷备份：备份时，不能进行任何操作



## 2、备份与恢复(mysqldump)

简介：如何利用mysql自带命令mysqldump来备份单库或者多库

1、mysqldump 使用语法：

```shell
mysqldump -u用户名 -p密码 dbname table > 路径/xxx.sql
mysqldump -u用户名 -p密码 --databases dbname1 dbname2 > xxx.sql

-- 常见选项：
-u: 用户名
-p: 密码
-P: 端口号，不写默认3306
--host，-h：服务器IP地址，不写默认本地localhost
--all-databases, -A：备份所有数据库
--databases, -B: 用于备份多个数据库。如果没有该选项，mysqldump把第一个名字参数作为数据库名，后面的作为表名。使用该选项，mysqldum把每个名字都当作为数据库名。
--no-data, -d：不导出任何数据，只导出数据库表结构。
--no-create-info, -t：只导出数据库的数据，而不添加CREATE TABLE 语句
--no-create-db, -n：只导出数据库的数据，而不添加CREATE DATABASE 语句

--quick, -q：快速导出
--xml, -X：导出为xml文件
```

2、本地备份示例：

```shell
# 1. 备份全部数据库的数据和结构（-A）
mysqldump -uroot -p123456 -A > 0101.sql

# 2. 备份全部数据库的结构（-d）
mysqldump -uroot -p123456 -A -d > 0102.sql

# 3. 备份全部数据库的数据（-t）（-n）
mysqldump -uroot -p123456 -A -t > 0103.sql
mysqldump -uroot -p123456 -A -n > 0113.sql

# 4. 备份单个数据库的数据和结构（dnname 为数据库名）
mysqldump -uroot -p123456 dnname > 0104.sql

# 5. 备份单个数据库结构（dbname 为数据库名，-d）
mysqldump -uroot -p123456 dbname -d > 0105.sql

# 6. 备份单个数据库数据（sakila 为数据库名，-t）
mysqldump -uroot -p123456 sakila -t > 0106.sql

# 7. 备份多个表的结构和数据 (table1,table2为表名)
mysqldump -uroot -p123456 dbname table1 table2 > 0107.sql

# 8. 一次备份多个数据库
mysqldump -uroot -p123456 --databases db1 db2 > 0108.sql
```

3、本机恢复数据：

```shell
# 数据恢复方式一：
# Linux系统命令行：1.先建库  2.导入数据
mysqladmin -uroot -p123456 create db_name 
mysql -uroot -p123456  db_name < d:\bak\0101.sql

# 注：在导入备份数据库前，db_name如果没有，是需要创建的；而且与backup20110527.sql中数据库名是一样的才可以导入。

# 数据恢复方式二：将导出的本地文件导入到指定数据库
# 2. soure 方法： 
mysql > use db
mysql > source d:\bak\0101.sql
```

4、远程备份示例：

```shell
# 1.远程备份单库：
mysqldump -uroot -p123456 -h120.25.93.69 zabbix | gzip > /mysql_data_back/zabbix_users.sql.gz

# 2.远程备份单库并保留创建库语句：
mysqldump -uroot -p123456 -h120.25.93.69 --databases zabbix | gzip > /mysql_data_back/zabbix_bak.sql.gz

# 3.远程备份单库单表：
mysqldump -uroot -p123456 -h120.25.93.69 zabbix users | gzip > /mysql_data_back/zabbix_users.sql.gz

# 4.远程备份多库：
mysqldump -uroot -p123456 -h120.25.93.69 --databases zabbix XD | gzip > /mysql_data_back/zabbix_XD.sql.gz

# 5.远程备份整库：
mysqldump -uroot -p123456 -h120.25.93.69 --all-databases | gzip > /mysql_data_back/all.sql.gz
```

5、远程恢复数据

```shell
# 远程恢复数据（备份的数据文件里有创建库的语句）：
mysql -uroot -p123456 -h120.25.93.69 < zabbix_bak.sql

# 远程恢复数据（备份的数据文件里没有创建库的语句）：
mysql -uroot -p123456 -h120.25.93.69 zabbix < zabbix_bak.sql
```

6、注意：

1. 还原单个数据库、单个数据库多个表需要指定数据库，而还原多个数据库时不用指定数据库。
2. 区别：备份用的命令是mysqldump， 还原用的是mysql。



## 3、物理备份与恢复

简介：详解数据库源文件以及如何物理备份

1、查找数据库源文件路径（2种方式）：

```shell
# 方式一:mysql内查询
mysql> show variables like 'datadir%';
+---------------+---------------+
| Variable_name | Value         |
+---------------+---------------+
| datadir       | /data/mydata/ |
+---------------+---------------+

# 方式二:配置文件查询
cat /etc/my.cnf
```

2、MyISAM表源文件：

```
└── data
    └── dbname
        ├── db.opt     -- 创建库的时候生成，主要存储着当前库的默认字符集和字符校验规则
        └── table_name
            ├── .frm   --记录着表结构信息的文件
            ├── .MYI   --记录着索引的文件
            └── .MYD   --记录着表的数据
            
            
db.opt：创建库的时候生成，主要存储着当前库的默认字符集和字符校验规则
.frm ：记录着表结构信息的文件
.MYI：记录着索引的文件
.MYD：记录着表的数据
```

3、nnoDB表源文件：InnoDB有着共享表空间跟独立表空间的概念。

```
└── data
    ├── ibdata1        -- 共享表空间，里边记录表的数据和索引
    └── dbname
        ├── db.opt     -- 创建库的时候生成，主要存储着当前库的默认字符集和字符校验规则
        └── table_name
            ├── .frm  --记录着表结构信息的文件
            └── .ibd  --独立表空间，里边记录这个表的数据和索引
 

db.opt：创建库的时候生成，主要存储着当前库的默认字符集和字符校验规则
.frm ：记录着表结构信息的文件
.ibd ：独立表空间，里边记录这个表的数据和索引
ibdata1：共享表空间，里边记录表的数据和索引
```

4、备份单库时分两种情况：

```


1.包含InnoDB表的库：
1.1.直接拷贝/data下的数据库名目录
2.2.拷贝/data 目录下三个文件
    -ib_logfile0：事务日志
    -ib_logfile1：事务日志
    -ibdata1   ：共享表空间

2.包含MyISAM表的库
2.1.直接拷贝/data下的数据库目录即可
```

5、备份数据库的时候遇到数据库正在写入数据报错：

```
# 备份dbname数据库时该数据库正在写入数据会显示报错
tar -czvf dbname.tar.gz dbname/
# 如果有数据写入时会报错，所以我们开源开启读锁

# 请求全局读锁
flush tables with read lock;
# 解锁
unlock tables;
```





## 4、二进制日志备份(mysqlbinlog)

简介：讲解如何利用二进制日志来备份数据

- 什么是二进制日志：二进制日志就是记录着mysql数据库中的一些写入性操作，比如一些增删改，但是，不包括查询！
- 二进制日志有哪些功能：一般情况下，二进制日志有着数据复制和数据恢复的功能
- 注意：开启二进制日志会有1%的性能消耗！
- 查看二进制日志是否开启：

```shell
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
```

- 开启二进制日志 :（记得重启服务） 

```shell
# 修改my.cnf配置文件
vi /etc/my.cnf

# 加入log-bin配置
[mysqld]
# 开启二进制日志log-bin
log-bin=/data/mydata/log_bin/mysql-bin
server-id=1
```

- 查看所有的binlog日志列表：

```shell
mysql> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |     23638 |
+------------------+-----------+
```

- 刷新二进制日志：

```shell
# 重置(清空）二进制日志文件
mysql> flush logs;

mysql> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |     23638 |
| mysql-bin.000002 |      1091 |
+------------------+-----------+
```

- 使用mysqldump备份数据时，加上-F选项可以重新生成一个新的二进制日志文件

```shell
mysqldump -uroot -p test user -F  > test_bak.sql
```





## 5、二进制日志恢复(mysqlbinlog)

简介：讲解如何利用二进制日志来恢复数据

- 查看二进制日志文件的内容报错：

```shell
$ mysqlbinlog mysql-bin.000002
mysqlbinlog: [ERROR] unknown variable 'default-character-set=utf8'
```

- 解决

```shell
# 第一种：在mysqlbinlog 后边加上 --no-defaults 
$ mysqlbinlog --no-defaults mysql-bin.000002
# 第二种：注释掉配置文件里边的：default-character-set=utf8
```

- 把二进制日志文件导出成普通文件：

```shell
# 第一种：在mysqlbinlog 后边加上 --no-defaults 
$ mysqlbinlog --no-defaults --base64-output=DECODE-ROWS -v mysql-bin.000002 > mysqlbin.sql

# 第二种：先注释掉配置文件里边的：default-character-set=utf8，在执行下命令
$ mysqlbinlog --base64-output=DECODE-ROWS -v mysql-bin.000002 > mysqlbin.sql
```

- 找出要恢复的位置：

```shell
# 1.找出关键字的行数：
$ mysqlbinlog --no-defaults mysql-bin.000002 | cat -n  | grep -iw 'drop'
  4180  DROP TABLE `user` /* generated by server */

# 2.打印出相关内容：
$ mysqlbinlog --no-defaults mysql-bin.000002 | cat -n | sed -n '4170,4180p'​
  4170  # at 59578
  4171  #190419  0:41:48 server id 1  end_log_pos 59609 CRC32 0x36cda2b7        Xid = 6380
  4172  COMMIT/*!*/;
  4173  # at 59609
  4174  #190419  0:41:48 server id 1  end_log_pos 59674 CRC32 0x8de2f06a        Anonymous_GTID  last_committed=145      sequence_number=146
  4175  SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
  4176  # at 59674
  4177  #190419  0:41:48 server id 1  end_log_pos 59787 CRC32 0x6b2edd2b        Query   thread_id=14    exec_time=0     error_code=0
  4178  use `XD`/*!*/;
  4179  SET TIMESTAMP=1555605708/*!*/;
  4180  DROP TABLE `user` /* generated by server */
```

恢复数据：

- 第一步：把备份的数据表user恢复到数据库中：mysql -uroot -p test < /mysql_data_back/user_bak.sql
- 第二步：利用上面找到的删除的位置进行恢复数据

```shell
$ mysqlbinlog --no-defaults --set-charset=utf8  --stop-position="59674" /data/mydata/log_bin/mysql-bin.000002 | mysql -uroot -p 
```



## 6、实操MySQL误删日志恢复

> 参考：https://mp.weixin.qq.com/s/nA8VmV83PuZNiBxvWNcdvg

**第一步**：保证mysql已经开启binlog，查看命令：

查看binklog是否开启

```mysql
show variables like '%log_bin%';

+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
```

查看binlog存放日志文件目录（如下图，博主binlog目录为/data/mysql）：

```mysql
show variables like '%datadir%';

+---------------+---------------+
| Variable_name | Value         |
+---------------+---------------+
| datadir       | /data/mydata/ |
+---------------+---------------+
```



**第二步**：进入binlog文件目录，找出日志文件

**第三步**：切换到mysqlbinlog目录（当线上数据出现错误的时候首先可以询问具体操作人记录时间点，这个时候可以借助mysql自带的binlog解析工具mysqlbinlog，具体位置在mysql安装目录**/mysql/bin/下）

**第四步**：通过mysqlbinlog工具命令查看数据库增删改查记录（必须切换到mysqlbinlog目录才有效）



## 7、Linux定时备份数据库

> MySQL 定时备份数据库（非常全）https://mp.weixin.qq.com/s/rglMxD-pRtlZZ8adqEKaxQ

在实际开发中，备份数据库是一件非常重要的事情，我们以Linux、MySql为例：

```shell
# 首先就是要找到MySql的位置，使用以下命令查看：
whereis mysql

# 若Linux没有安装crontab则可以执行以下命令进行安装:
yum install vixie-cron
yum install crontab
```

crontab常用定时任务命令：

```shell
systemctl start crond.service		# 启动
systemctl stop crond.service		# 停止
systemctl restart crond.service		# 重新启动
systemctl status crond.service		# 查看状态 
systemctl reload crond.service		# 重新载入配置

# 语法：crontab 【选项】 
crontab -e # 编辑crontab任务（是针对某个用户的，而编辑/etc/crontab是针对系统的任务）
crontab -r # 删除crontab里的所有任务
crontab -l # 列出crontab有哪些任务 
crontab -l -u jp # 列出用户jp的所有调度任务

# 内容格式： 
* * * * * 级别 命令 
分 时 日 月 周
```



先创建一个脚本（此脚本将备份的sql进行了压缩）：

```shell
cd /opt/db_back/
vim mysql_backup.sh
```

脚本内容如下：

```shell
DB_USER="root"
DB_PASS="passpword"
DB_HOST="xx.xx.xx.xx"
DB_NAME="BD_NAME"
BIN_DIR="/usr/bin/" 
BCK_DIR="/root/db_back/"
DATE=`date +%s`
$BIN_DIR/mysqldump --opt -u$DB_USER -p$DB_PASS -h$DB_HOST -P3306  $DB_NAME | gzip > $BCK_DIR/$DATE.sql.gz
```

```
DB_USER：用户名
DB_PASS：数据库密码
DB_HOST：数据库IP
DB_NAME：数据库名名称
DIN_DIR：MySql地址，使用准备工作当中所述的命令获取
BCK_DIR：存储位置
DATE：以时间戳为文件名
```

**注:-P:表示端口**

编辑完脚本可以测试一下脚本是否正确，输入sh mysql_backup.sh，然后去对应存储位置看下是否备份成功，在执行的时候会提示如下信息：

```
Warning: Using a password on the command line interface can be insecure.
```

意思就是直接在脚本里明文显示数据库的用户名、IP、密码、名称等相关内容是不安全的，所以**可以根据实际需求修改脚本命令，例如可以在脚本中将数据库用户名、IP、密码、名称等相关内容使用read命令输入。**

脚本制作完成即可添加定时任务，输入以下命令进行编辑

```shell
# 编辑crontab任务
crontab -e
# 每分钟执行一次备份
*/1 * * * * sh /root/db_back/mysql_backup.sh
```



# MySQL字符串拆分法（可用于统计）

参考：[MySQL逗号分割字段的行列转换技巧](https://www.cnblogs.com/cenalulu/archive/2012/08/20/2647463.html)

> 前言：由于很多业务表因为历史原因或者性能原因，都使用了违反第一范式的设计模式。即同一个列中存储了多个属性值（具体结构见下表）。这种模式下，应用常常需要将这个列依据分隔符进行分割，并得到列转行的结果。

表数据：

| ID   | Value          |
| ---- | -------------- |
| 1    | tiny,small,big |
| 2    | small,medium   |
| 3    | tiny,big       |

期望得到结果：

| ID   | Value  |
| ---- | ------ |
| 1    | tiny   |
| 1    | small  |
| 1    | big    |
| 2    | small  |
| 2    | medium |
| 3    | tiny   |
| 3    | big    |

```sql
# 需要处理的表
create table tbl_name (ID int ,mSize varchar(100));
insert into tbl_name values (1,'tiny,small,big');
insert into tbl_name values (2,'small,medium');
insert into tbl_name values (3,'tiny,big');

# 用于循环的自增表
create table incre_table (AutoIncreID int);
insert into incre_table values (1);
insert into incre_table values (2);
insert into incre_table values (3);
```

```sql
SELECT
	a.ID,
	substring_index( substring_index( a.mSize, ',', b.AutoIncreID ), ',',- 1 ) 
FROM
	tbl_name a
	JOIN incre_table b ON b.AutoIncreID <= ( length( a.mSize ) - length( REPLACE ( a.mSize, ',', '' ))+ 1 ) 
ORDER BY
	a.ID;
```

原理分析：这个join最基本原理是笛卡尔积。通过这个方式来实现循环。

总结：

1. 这种方法的缺点在于，我们需要一个拥有连续数列的独立表（这里是incre_table)。并且连续数列的最大值一定要大于符合分割的值的个数
2. 例如有一行的mSize 有100个逗号分割的值，那么我们的incre_table 就需要有至少100个连续行
3. 当然，mysql内部也有现成的连续数列表可用。如mysql.help_topic： help_topic_id 共有504个数值，一般能满足于大部分需求了

```sql
SELECT
	a.ID,
	substring_index( substring_index( a.mSize, ',', b.help_topic_id + 1 ), ',',- 1 ) 
FROM
	tbl_name a
	JOIN mysql.help_topic b ON b.help_topic_id < ( length( a.mSize ) - length( REPLACE ( a.mSize, ',', '' ))+ 1 ) 
ORDER BY
	a.ID;
```





# 参考资料 & 鸣谢

1. SQL语法速成手册：https://mp.weixin.qq.com/s/_ch-mwQfHRkJJ7N59S7jpQ
2. MySQL官方压测神器：https://mp.weixin.qq.com/s/E_C86nx871ef7rCsOHebQA
3. 必须了解的MySQL三大日志：https://mp.weixin.qq.com/s/SfHImi2WVXTKFtlxiwztEA
4. 必须了解的MySQL三大日志：https://mp.weixin.qq.com/s/L9xqW-975eEY3JnYtjEUHw
5. 面试官：不会看SQL执行计划，简历也敢写精通SQL优化？：https://mp.weixin.qq.com/s/5xvN29sCJImYClQWrRNvCA
6. MySQL执行计划Explain详解：https://mp.weixin.qq.com/s/km5jPQaPVn1n8m8y5vGl4g
7. 不要再问我 in，exists 走不走索引了：https://mp.weixin.qq.com/s/_-mZZUAXs01JcPiJO5C-lw
8. MySQL专题：[看清所苡看轻](https://blog.csdn.net/weixin_44907813/category_9894528.html)
9. MySQL之my.conf详解：https://www.cnblogs.com/panwenbin-logs/p/8360703.html
10. MySQL经典面试题：https://www.cnblogs.com/panwenbin-logs/p/8366940.html
11. 博客园（Linux系统运维之路）（MySQL入门专栏）：https://www.cnblogs.com/linuxk/category/1248281.html
12. CSDN（服务端开发）（MySQL专栏）：https://blog.csdn.net/u010013573/category_8739812.html
13. [centos 7 linux系统安装 mysql5.7.17（glibc版）](https://www.cnblogs.com/mujingyu/p/7689116.html)
14. 【MySQL】之MySQL5.7版本数据库多实例安装部署：https://blog.csdn.net/u011208222/article/details/109217474
15. https://www.cnblogs.com/captain_jack/archive/2010/10/12/1848496.html（my.conf）
16. MySQL8.0 密码管理与密码策略：https://blog.csdn.net/qq_18948359/article/details/105837540
17. 20000 字干货笔记，一天搞定 MySQL ！：https://blog.csdn.net/csdnnews/article/details/106798808

***

1. MySQL统计一列中不同值的数量方法示例：https://www.cnblogs.com/cxxjohnson/p/7966766.html
2. Mysql中字段分割字符串（ 一行转多行 ）：https://www.cnblogs.com/JMrLi/p/11320273.html





















