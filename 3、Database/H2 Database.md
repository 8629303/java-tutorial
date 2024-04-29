# H2 数据库 - 入门

## H2 数据库 - 简介

### 1、H2 数据库介绍

> **`官方文档地址：`**http://www.h2database.com/html/main.html

H2的开发始于2004年5月， 但它在2005年12月14日首次发表。 H2的原作者Thomas Mueller也是Hypersonic SQL的原始开发者。 2001年，他加入PointBase公司。在那里他编写了商业JavaSQL数据库PointBaseMicro。 在这一点上，他不得不停止Hypersonic SQL。HSQLDB集团成立 继续从事Hypersonic SQL代码库的工作。 名称H2代表Hypersonic 2号，但H2不与Hypersonic SQL或HSQLDB共享代码 。H2是从头开始构建的。

H2 是一个开放源码的轻量级 Java 数据库。它可以嵌入到 Java 应用程序中或以客户端 - 服务器模式运行。主要是 H2 数据库可以配置为作为内存数据库运行，这意味着数据不会在磁盘上持久存储。由于嵌入式数据库不适用于生产开发，而是主要用于开发和测试。

**1、该数据库可以在嵌入式模式或服务器模式下使用。以下是 H2 数据库的主要特点：**

- 极快，开源的 JDBC API
- 可用于嵌入式和服务器模式; 内存数据库
- 基于浏览器的控制台应用程序
- 占地面积小 - 约1.5MB的jar文件大小

**2、H2数据库的主要特点如下：**

- 它是一个非常快的数据库引擎。
- H2是开源的，用Java编写。
- 它支持标准的SQL和JDBC API。它也可以使用PostgreSQL ODBC驱动。
- 它具有嵌入式和服务器模式。
- H2支持**集群**和**多版本并发**。
- 它具有很强的安全特性。

**3、以下是H2数据库的一些附加功能：**

- H2是基于磁盘或内存的数据库和表，只读数据库支持，临时表。
- H2提供事务支持（读提交），2阶段提交多个连接，表级锁定。
- H2是基于成本的优化器，使用遗传算法进行复杂查询，零管理。
- H2包含可滚动和可更新的结果集支持，大结果集，外部结果排序，函数可以返回结果集。
- H2支持加密数据库（AES），SHA-256密码加密，加密功能和SSL。

**4、H2数据库中的组件：**

- 网页浏览器
- H2控制台服务器

这是一个客户端/服务器应用程序，因此服务器和客户端（浏览器）都需要运行它。



### 2、H2 数据库对比

H2 是一个内存数据库，提供基于浏览器的控制台管理程序，用法和MySQL数据库没有什么区别。

H2 比 HSQLDB 的最大的优势就是 H2 提供了一个十分方便的 Web 控制台用于操作和管理数据库内容，这点比起 HSQLDB 的 Swing 和 awt 控制台实在好用多了，并且支持集群。

1. 



## H2数据库 - 安装

H2 数据库下载：http://www.h2database.com/html/download.html

本人这里使用 zip 免安装版。解压即可使用。本人教程使用的版本号：h2-2023-07-04、h2-2.2.220.jar

- https://github.com/h2database/h2database/releases/download/version-2.2.220/h2-2023-07-04.zip

H2 数据库目录如下：

```java
h2
　|—bin
　| |—h2-2.2.220.jar　　// H2数据库的jar包（驱动也在里面）
　| |—h2.bat　　        // Windows控制台启动脚本
　| |—h2.sh　　         // Linux控制台启动脚本
　| |—h2w.bat　　       // Windows控制台启动脚本（不带黑屏窗口）
　|—docs　　            // H2数据库的帮助文档（内有H2数据库的使用手册）
　|—service　　         // 通过wrapper包装成服务, 以服务的方式运行
　|—src　　             // H2数据库的源代码
　|—build.bat　　       // windows构建脚本
　|—build.sh　　        // linux构建脚本
```

进入Ｈ2 的bin目录，运行 h2.bat（h2w.bat是后台静默运行），系统会进入H2 数据库的Web Console，`http://<你的ip>:8082`。

2、可选配置

在用户目录下新建 .h2.server.properties，支持如下属性配置：

- webAllowOthers: 是否允许远程连接，默认 false。
- webPort: h2 端口，默认为 8082。
- webSSL: 是否启用 SSL 加密连接，默认 false。
- webAdminPassword: 超级管理员密码。

Wen 控制台的 Preferences & Tools 按钮需要超级管理员密码才能进入。

> 如果没有手动配置此文件，以 web-server 方式首次启动 H2 后，点击打开的浏览器页面的 Save 按钮后就会自动创建一个。

`.h2.server.properties` 文件范例：

```properties
#H2 Server Properties
#Fri Aug 04 11:09:44 CST 2023
webSSL=false
webAllowOthers=false
webPort=8082
10=Generic DB2|com.ibm.db2.jcc.DB2Driver|jdbc\:db2\://localhost/test|
11=Generic Oracle|oracle.jdbc.driver.OracleDriver|jdbc\:oracle\:thin\:@localhost\:1521\:XE|sa
12=Generic MS SQL Server 2000|com.microsoft.jdbc.sqlserver.SQLServerDriver|jdbc\:microsoft\:sqlserver\://localhost\:1433;DatabaseName\=sqlexpress|sa
13=Generic MS SQL Server 2005|com.microsoft.sqlserver.jdbc.SQLServerDriver|jdbc\:sqlserver\://localhost;DatabaseName\=test|sa
14=Generic PostgreSQL|org.postgresql.Driver|jdbc\:postgresql\:test|
15=Generic MySQL|com.mysql.cj.jdbc.Driver|jdbc\:mysql\://localhost\:3306/test|
16=Generic MariaDB|org.mariadb.jdbc.Driver|jdbc\:mariadb\://localhost\:3306/test|
17=Generic HSQLDB|org.hsqldb.jdbcDriver|jdbc\:hsqldb\:test;hsqldb.default_table_type\=cached|sa
18=Generic Derby (Server)|org.apache.derby.client.ClientAutoloadedDriver|jdbc\:derby\://localhost\:1527/test;create\=true|sa
19=Generic Derby (Embedded)|org.apache.derby.iapi.jdbc.AutoloadedDriver|jdbc\:derby\:test;create\=true|sa
0=Generic JNDI Data Source|javax.naming.InitialContext|java\:comp/env/jdbc/Test|sa
1=Generic Teradata|com.teradata.jdbc.TeraDriver|jdbc\:teradata\://whomooz/|
2=Generic Snowflake|com.snowflake.client.jdbc.SnowflakeDriver|jdbc\:snowflake\://accountName.snowflakecomputing.com|
3=Generic Redshift|com.amazon.redshift.jdbc42.Driver|jdbc\:redshift\://endpoint\:5439/database|
4=Generic Impala|org.cloudera.impala.jdbc41.Driver|jdbc\:impala\://clustername\:21050/default|
5=Generic Hive 2|org.apache.hive.jdbc.HiveDriver|jdbc\:hive2\://clustername\:10000/default|
6=Generic Hive|org.apache.hadoop.hive.jdbc.HiveDriver|jdbc\:hive\://clustername\:10000/default|
7=Generic Azure SQL|com.microsoft.sqlserver.jdbc.SQLServerDriver|jdbc\:sqlserver\://name.database.windows.net\:1433|
8=Generic Firebird Server|org.firebirdsql.jdbc.FBDriver|jdbc\:firebirdsql\:localhost\:c\:/temp/firebird/test|sysdba
9=Generic SQLite|org.sqlite.JDBC|jdbc\:sqlite\:test|sa
20=Generic H2 (Server)|org.h2.Driver|jdbc\:h2\:tcp\://localhost/~/test|sa
21=Generic H2 (Embedded)|org.h2.Driver|jdbc\:h2\:mem\:test|sa
```

注：在 H2 Control 每次构建的数据库连接保存后都会作为历史配置项自动保存到 .h2.server.properties 文件内，一行一个链接配置，格式为：

```
<number>=<name>|<driver>|<url>|<user>
```

上述编号 0～21 的连接配置是 H2 默认提供的，也可以自定义，连接的密码因安全性不会保存下来。



## H2 数据库 - 连接

### 1、运行模式

H2 数据库有三种运行模式：

1. 内嵌模式：应用和数据库同在一个JVM中，通过JDBC进行连接，可以同应用程序打包在一起发布。
2. 服务器模式：和普通数据库一样。支持持久数据库和内存数据库。没有打开连接数和打开数据库数量方面的限制。
   1. TCP/IP Server：支持客户端/服务器端的连接方式
   2. Web Server：此种运行方式支持使用浏览器访问H2 Console
   3. PG Server：支持PostgreSQL客户端
3. 混合模式：混合模式是内嵌模式和服务器模式的组合。第一个应用通过内嵌模式与数据库建立连接。同一时候也作为一个服务器启动。于是另外的应用（运行在不同的进程或是虚拟机上）能够同一时候訪问相同的数据。

注意事项：

1. 第一次登陆可以用sa进行，密码可以不填。使用其他用户名登陆会自动创建用户，如果指定密码则下次登陆需要输入指定后的密码
2. 在file模式下只支持一个进程连接，如果有多个进程需要连接同一个url可以用Automatic Mixed Mode模式，关键字AUTO_SERVER=TRUE进行处理通过AUTO_SERVER_PORT=9090进行指定端口。当然此方法不适用于内存数据库即：jdbc:h2:mem:test_mem此类连接模式。



### 2、运行方式

#### 1、嵌入式

数据库持久化存储为单个文件。

连接字符串：`~/.h2/DBName` 表示数据库文件的存储位置，如果第一次连接则会自动创建数据库。

- `jdbc:h2:~/test` - 'test' 在用户根目录下
- `jdbc:h2:/data/test` - 'test' 在 /data 目录下
- `jdbc:h2:test` - 'test' 在当前工作目录

会在指定路径下创建一个名为 `test.mv.db` 的数据库，你可能还会看到一个 `test.trace.db` 的文件，这个是h2的错误信息，可以直接打开看。



#### 2、内存式

数据库只在内存中运行，关闭连接后数据库将被清空，适合测试环境.

连接字符串：`jdbc:h2:mem:DBName;DB_CLOSE_DELAY=-1`

如果不指定 DBName，则以私有方式启动，只允许一个连接。

- `jdbc:h2:mem:test` - 一个进程中有多个连接
- `jdbc:h2:mem:` - 未命名的私有库，一个连接



#### 3、服务模式

H2 支持三种服务模式：

- Web server：此种运行方式支持使用浏览器访问 H2 Console
- TCP server：支持客户端/服务器端的连接方式
- PG server：支持 PostgreSQL 客户端

启动 Tcp 服务连接字符串示例：

- `jdbc:h2:tcp://localhost/~/test`：用户根目录
- `jdbc:h2:tcp://localhost/data/test`：绝对路径
- `jdbc:h2:tcp://localhost/mem:test`：用户内存模式



#### 4、启动服务

Win10 / Win11 下启动：

- 方式一：进入Ｈ2 的bin目录，运行 h2.bat 或 h2w.bat（区别为h2w.bat是后台静默运行）

- 方式二：同理可以通过cmd直接使用命令进行启动：`java -cp h2-2.2.220.jar org.h2.tools.Server`

- 执行如下命令，获取选项列表及默认值：

  ```bash
  java -cp h2-2.2.220.jar org.h2.tools.Server -?
  ```

  常见的选项如下：

  - -web：启动支持H2 Console的服务
  - -webPort < port >：服务启动端口，默认为8082
  - -browser：启动H2 Console web管理页面
  - -tcp：使用TCP server模式启动
  - -pg：使用PG server模式启动

  ```bash
  # 浏览器Web服务方式, 使用 org.h2.tools.Console 默认可以不给后面的参数
  java -cp h2-2.2.220.jar org.h2.tools.Console
  java -cp h2-2.2.220.jar org.h2.tools.Server -web -webPort 8082 -browser -webAllowOthers
  
  # TCP服务方式
  java -cp h2-2.2.220.jar org.h2.tools.Server -tcp -tcpPort 9092 -tcpSSL -webAllowOthers
  ```

如上操作后系统会进入H2 数据库的Web Console，`http://<你的ip>:8082`。

**注意**：这里发现个小问题，也不确定是不是BUG，使用 org.h2.tools.Console 启动Web控制台时，是可以直接连接内存模式的，使用 org.h2.tools.Server 开启Web控制台是无法连接访问内存模式。



#### 5、Maven 方式

此外，使用 maven 也可以启动 H2 服务。添加以下插件

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>java</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <mainClass>org.h2.tools.Server</mainClass>
        <arguments>
            <argument>-web</argument>
            <argument>-webPort</argument>
            <argument>8090</argument>
            <argument>-browser</argument>
        </arguments>
    </configuration>
</plugin>
```

在命令行中执行如下命令启动 H2 Console

```bash
mvn exec:java
```

或者建立一个 bat 文件

```bash
@echo off
call mvn exec:java
pause
```

此操作相当于执行了如下命令：

```bash
java -jar h2-*.jar -web -webPort 8090 -browser
```



### 3、连接方式

> H2数据库启动方式：https://www.52dianzi.com/bangong/?read-1688066033a175130.html

1、以嵌入式(本地)连接方式连接H2数据库：这种连接方式默认情况下只允许有一个客户端连接到H2数据库，有客户端连接到H2数据库之后，此时数据库文件就会被锁定，那么其他客户端就无法再连接了。

- 连接语法：`jdbc:h2:[file:][<path>]<databaseName>`

2、H2数据库的内存模式

- H2数据库被称为内存数据库，因为它支持在内存中创建数据库和表。注意：如果使用H2数据库的内存模式，那么我们创建的数据库和表都只是保存在内存中，一旦服务器重启，那么内存中的数据库和表就不存在了。

3、使用TCP/IP的服务器模式(远程连接)方式连接H2数据库（推荐）：这种连接方式就和其他数据库类似了，是基于Service的形式进行连接的，因此允许多个客户端同时连接到H2数据库。

- 连接语法：`jdbc:h2:tcp://<server>[:<port>]/[<path>]<databaseName>`



#### 1、文件方式启动

严格来说应该叫文件方式连接，因为文件方式启动不需要先启动数据库服务器，直接在连接URL中填写绝对或相对路径即可，由于创建数据库连接时，H2引擎发现指定的路径下如果不存在数据库，则会自动创建，如：

```java
Class.forName("org.h2.Driver");
Connection conn2 = DriverManager.getConnection("jdbc:h2:~/test", "sa", "");
Connection conn3 = DriverManager.getConnection("jdbc:h2:file:~/test", "sa", "");
Connection conn4 = DriverManager.getConnection("jdbc:h2:file:C:/Users/lsx/test", "sa", "");
```

这段代码可以正常执行，其中file表示以文件方式启动数据库连接，但如果是第一次使用，可能不知道H2把数据库创建在了什么位置，熟悉linux的会知道~符号表示系统当前用户的主文件夹，所以(以win10/11为例)，上面这段代码会在C:\Users\lsx目录下创建名为test的数据库，用户名为sa，密码为空(用户名密码在首次创建时可以随意设置)。由于H2默认连接方式就是文件方式，所以file关键字可以省略，也就是说，在win下，如上三种写法是等价的。

以上的三种写法使用的是绝对路径，在实际应用中，你可能会希望程序在哪运行，就是哪创建数据库，所以会用到相对路径，如：

```java
Connection conn = DriverManager.getConnection("jdbc:h2:file:h2db/test", "sa", "");
```

当执行以上代码时，H2会在JAVA程序运行的根目录下创建h2db文件夹，然后在h2db下创建test数据库。

如果数据库已经创建，下次再进行连接时，就必须提供与第一次创建时一致的用户名密码，否则将连接不成功。

另外需要强调的是，以文件方式连接数据库时，H2引擎会新建两个守护线程，一个用来锁文件，一个用来写文件：

所以这种方式是独占的，即当以文件方式连接数据库所创建的Connection如果没有关闭，不允许再有其他的进程以文件方式连接此数据库。如果尝试连接，将会报以下错误：

```java
Exception in thread "main" org.h2.jdbc.JdbcSQLException: Database may be already in use: Locked by another process. Possible solutions: close all other connection(s); use the server mode [90020-72]
    at org.h2.message.Message.getSQLException(Message.java:92)
    at org.h2.message.Message.getSQLException(Message.java:96)
    at org.h2.message.Message.getSQLException(Message.java:74)
```

所以文件方式不适合做类似数据库连接池的应用。



#### 2、内存方式启动

严格来说应该叫内存方式启动也是文件方式连接的一种，因为内存方式启动也不需要先启动数据库服务器，直接在连接URL中填mem:即可，数据库只在内存中运行，关闭连接后数据库将被清空，适合测试环境，如：

```java
Class.forName("org.h2.Driver");
Connection conn1 = DriverManager.getConnection("jdbc:h2:mem:", "sa", "");
Connection conn2 = DriverManager.getConnection("jdbc:h2:mem:tete_mem", "sa", "");
```

这段代码可以正常执行，其中mem表示以内存方式启动数据库连接，当连接关闭后，数据空会被清空。可以注意到，上面代码中有2中写法，一种不带数据库名的可以理解为匿名内存数据库，默认情况下只允许自己访问。另一种是给定数据库名字的内存数据库。



#### 3、Web控制台启用

在讲完一种启动方式时需要先讲一下H2的web控制台，然后再讲更高级的数据库启动方式。因为无论以哪一种数据库方式启动数据库，你都可以使用web控制台来连接，并执行CRUD操作，非常方便，要启动Web控制台，可以执行如下命令登陆界面：

1. 方式一：命令行启动

   ```bash
   # 使用 org.h2.tools.Console 默认可以不给后面的参数
   java -cp h2-2.2.220.jar org.h2.tools.Console
   # 或者
   java -cp h2-2.2.220.jar org.h2.tools.Server -web -webPort 8082 -webAllowOthers -browser
   ```

2. 方式二：使用代码

   ```java
   Server.createWebServer("-webAllowOthers", "-webPort", "8082").start();
   Class.forName("org.h2.Driver");
   Connection conn1 = DriverManager.getConnection("jdbc:h2:mem:", "sa", "");
   Connection conn2 = DriverManager.getConnection("jdbc:h2:mem:test", "sa", "");
   Connection conn3 = DriverManager.getConnection("jdbc:h2:~/test", "sa", "");
   ```

3. 注意：使用代码创建后，想在Web控制台访问内存或文件数据时，需要先创建好数据库，如果访问一个从未创建过的数据库会报错

   ```java
   Database "mem:t" not found, either pre-create it or allow remote database creation (not recommended in secure environments) [90149-220] 90149/90149 (帮助)
   ```



#### 4、Tcp 方式启动

Tcp方式启动时，H2将在本地启动一个tcp服务器，你可以用客户端连接至此端口访问不同的数据库，这种方式支持多客户端。在启动数据库时，你不需要指定具体数据库（因为在连接时才需要指定），只需指定需要监听的端口即可，如：

```bash
java -cp h2-*.jar org.h2.tools.Server -tcp
```

这样即可启动数据库服务器了，由于你没有指定端口，所以H2将监听默认端口9092，如果你想指定端口，需要使用tcpPort参数，如：

```bash
java -cp h2-*.jar org.h2.tools.Server -tcp -tcpPort 9092
```

注意：此种方式启动后，你将可以在本地通过9092访问H2数据库，但不允许其他机器访问，如果你希望从其他机器也能通过此端口访问数据库，就需要用到tcpAllowOthers参数了，如：

```bash
java -cp h2-*.jar org.h2.tools.Server -tcp -tcpPort 9092 -tcpAllowOthers
```

数据库启动后，通过Web console或JDBC连接时，需要把URL也更改为tcp方式，如：

```bash
jdbc:h2:tcp://localhost:9092/~/test
```

可以看到，在端口号后面追加绝对或相对路径和前面讲的文件方式启动时规则是一样的。

如下是JAVA代码方式启动：

```java
Server.createWebServer("-webAllowOthers", "-webPort", "8082").start();
Server.createTcpServer("-tcpAllowOthers", "-tcpPort", "9092").start();
Class.forName("org.h2.Driver");
// 混合模式：使用TCP + file 模式, 需要提前创建一个文件模式的数据库, 不然通过TCP无法访问连接
Connection conn1 = DriverManager.getConnection("jdbc:h2:~/test", "sa", "");
Connection conn2 = DriverManager.getConnection("jdbc:h2:tcp://localhost:9092/~/test", "sa", "");
// 混合模式: 使用TCP + mem 模式, 需要提前创建一个内存模式的数据库, 不然通过TCP无法访问连接
Connection conn3 = DriverManager.getConnection("jdbc:h2:mem:test", "sa", "");
Connection conn4 = DriverManager.getConnection("jdbc:h2:tcp://localhost:9092/mem:test", "sa", "");
```



#### 5、Tcp 方式启动（匿名SSL）

当使用Tcp方式启动数据库时，如果需要从外网访问，就必须考虑安全问题，H2官网对SSL方式启动数据库做了非常精要的说明，如下：

> Remote SSL/TLS connections are supported using the Java Secure Socket Extension (SSLServerSocket, SSLSocket). By default, anonymous SSL is enabled. The default cipher suite is SSL_DH_anon_WITH_RC4_128_MD5.
> To use your own keystore, set the system properties javax.net.ssl.keyStore and javax.net.ssl.keyStorePassword before starting the H2 server and client. See also Customizing the Default Key and Trust Stores, Store Types, and Store Passwords for more information.
> To disable anonymous SSL, set the system property h2.enableAnonymousSSL to false.

要使用ssl方式启动数据库，需要使用`h2.enableAnonymousSSL`和`tcpSSL`参数，如：

```bash
java -Dh2.enableAnonymousSSL=true -cp h2-*.jar org.h2.tools.Server -tcp -tcpPort 9092 -tcpAllowOthers -tcpSSL
```

注意：前面的h2.enableAnonymousSSL需要设置到JAVA系统属性里，所以要在前面追加-D。由于默认使用的就是匿名SSL，所以h2.enableAnonymousSSL参数可以省略，也就是说，下面两个命令是等价的：

```bash
java -Dh2.enableAnonymousSSL=true -cp h2.jar org.h2.tools.Server -tcp -tcpPort 9092 -tcpAllowOthers -tcpSSL
java -classpath h2.jar org.h2.tools.Server -tcp -tcpPort 9092 -tcpAllowOthers -tcpSSL
```

启动完毕，客户端连接的方式还和原来一样，不过要把url更改为ssl方式，如：

```bash
jdbc:h2:ssl://localhost:9092/~/test
```



#### 6、Tcp方式启动（双向认证SSL）

尽管匿名ssl方式启动数据库保证了数据传输的可靠与安全性，但仍然没有解决客户端及服务器身份问题。也就是说，通过上面的方式启动数据库后，任何一个人都可以通过其他机器连接至9092端口，只需要破解数据库密码即可随意操作你的数据库了，这无疑非常危险，所以我们需要更安全的方式。SSL双向认证的原理及细节这里就不再阐述了，我们主要讲解如何使用SSL双向认证的方式启动H2数据库。步骤如下：

##### 1、制作证书

我们将使用JDK的keytool制作安全证书，请确保你的JDK已正确安装并且已经设置好环境变量。我是直接再JDK目录下执行：

- keytool -genkey -alias serverkey -keystore kserver.keystore
- keytool -export -alias serverkey -keystore kserver.keystore -file server.crt
- keytool -import -alias serverkey -file server.crt -keystore tclient.keystore
- keytool -genkey -alias clientkey -keystore kclient.keystore
- keytool -export -alias clientkey -keystore kclient.keystore -file client.crt
- keytool -import -alias clientkey -file client.crt -keystore tserver.keystore

为简单其见，凡是需要输入密码的地方都输入123456，凡是需要输入y和n的地方都输入y，这样命令执行成功后会生成以下文件

- kserver.keystore
- tserver.keystore
- server.crt
- kclient.keystore
- tclient.keystore
- client.crt

我们只需要其中的四个就行了，服务器需要使用kserver.keystore、tserver.keystore，客户端使用kclient.keystore、tclient.keystore。



##### 2、启动数据库

如果成功制作了安全证书，那么启动数据库就很简单了，只需要把上一步制作的kserver.keystore、tserver.keystore放在h2.jar同目录下（当然放在别的目录也可以，只要在参数上指定对的路径即可），启动命令如下：

```bash
java -Dh2.enableAnonymousSSL=false -Djavax.net.ssl.keyStore=kserver.keystore -Djavax.net.ssl.keyStorePassword=123456 -Djavax.net.ssl.trustStore=tserver.keystore -Djavax.net.ssl.trustStorePassword=123456 -classpath h2-*.jar org.h2.tools.Server -tcp -tcpPort 9092 -tcpAllowOthers -tcpSSL
```



##### 3、客户端连接

客户端连接时需要对应的kclient.keystore、tclient.keystore，放在h2.jar同一目录下即可，连接命令如下：

```bash
java -Djavax.net.ssl.keyStore=kclient.keystore -Djavax.net.ssl.keyStorePassword=123456 -Djavax.net.ssl.trustStore=tclient.keystore -Djavax.net.ssl.trustStorePassword=123456 -classpath h2-*.jar org.h2.tools.Console -web
```

如果你连接时出现Connection is broken [90067-72] 90067/90067 (Help)的错误，可能的原因有：

1. 未在启动参数中指明安全证书路径或指明的证书路径错误，或者忘了把证书放在指定的路径下
2. 启动参数中指明的密码错误
3. 连接的URL有误，比如忘了加ssl或者端口号指定错误等等
4. 连接的命令中忘了加-web参数，如果没有加-web，在本机测试时客户端也会占用9092端口，但它发现此端口已经被服务器占用了，就会启动失败，结果程序退出，这样的话IE虽然能打开，但是也会连接不上。



### 4、URL 配置

| Topic                               | URL Format and Examples                                      |
| :---------------------------------- | :----------------------------------------------------------- |
| 嵌入式（本地）连接                  | jdbc:h2:file:< databaseName > <br />jdbc:h2:~/test <br />jdbc:h2:file:/data/sample <br />jdbc:h2:file:C:/data/sample (Windows only) |
| 内存数据库（私有）                  | jdbc:h2:mem:                                                 |
| 内存数据库（被命名）                | jdbc:h2:mem:< databaseName > <br />jdbc:h2:mem:test_mem      |
| 使用TCP/IP的服务器模式（远程连接）  | jdbc:h2:tcp://< server >[:< port >]/[< path >]< databaseName > <br />jdbc:h2:tcp://localhost/~/test <br />jdbc:h2:tcp://dbserv:8084/~/sample |
| 使用SSL/TLS的服务器模式（远程连接） | jdbc:h2:ssl://< server >[:< port >]/< databaseName > <br />jdbc:h2:ssl://secureserv:8085/~/sample; |
| 使用加密文件                        | jdbc:h2:< url >;CIPHER=[AES\|XTEA] <br />jdbc:h2:ssl://secureserv/~/testdb;CIPHER=AES <br />jdbc:h2:file:~/secure;CIPHER=XTEA |
| 文件锁                              | jdbc:h2:< url >;FILE_LOCK={NO\|FILE\|SOCKET} <br />jdbc:h2:file:~/quickAndDirty;FILE_LOCK=NO<br />jdbc:h2:file:~/private;CIPHER=XTEA;FILE_LOCK=SOCKET |
| 仅打开存在的数据库                  | jdbc:h2:< url >;IFEXISTS=TRUE<br />jdbc:h2:file:~/sample;IFEXISTS=TRUE |
| 当虚拟机退出时并不关闭数据库        | jdbc:h2:< url >;DB_CLOSE_ON_EXIT=FALSE                       |
| 用户名和密码                        | jdbc:h2:< url >[;USER=< username >]\[;PASSWORD=] jdbc:h2:file:~/sample;USER=sa;PASSWORD=123 |
| 更新记入索引                        | jdbc:h2:< url >;LOG=2<br />jdbc:h2:file:~/sample;LOG=2       |
| 调试跟踪项设置                      | jdbc:h2:< url >;TRACE_LEVEL_FILE=<level 0..3><br />jdbc:h2:file:~/sample;TRACE_LEVEL_FILE=3 |
| 忽略位置参数设置                    | jdbc:h2:< url >;IGNORE_UNKNOWN_SETTINGS=TRUE                 |
| 指定文件读写模式                    | jdbc:h2:< url >;ACCESS_MODE_LOG=rws;ACCESS_MODE_DATA=rws     |
| 在Zip文件中的数据库                 | jdbc:h2:zip:< zipFileName >!/< databaseName ><br />jdbc:h2:zip:~/db.zip!/test |
| 兼容模式                            | jdbc:h2:< url >;MODE=< databaseType ><br />jdbc:h2:~/test;MODE=MYSQL;DATABASE_TO_LOWER=TRUE |
| 自动重连接                          | jdbc:h2:< url >;AUTO_RECONNECT=TRUE<br />jdbc:h2:tcp://localhost/~/test;AUTO_RECONNECT=TRUE |
| 自动混合模式                        | jdbc:h2:< url >;AUTO_SERVER=TRUE<br />jdbc:h2:~/test;AUTO_SERVER=TRUE |
| 页面大小                            | jdbc:h2:< url >;PAGE_SIZE=512<br />jdbc:h2:~/test;PAGE_SIZE=512 |
| 更改其他设置                        | jdbc:h2:< url >;< setting >=< value >[;< setting >=< value >…]<br />jdbc:h2:file:~/sample;TRACE_LEVEL_SYSTEM_OUT=3 |

连接字符串参数：

- DB_CLOSE_DELAY：要求最后一个正在连接的连接断开后，不要关闭数据库。
- MODE=MySQL：兼容模式，H2兼容多种数据库，该值可以为：DB2、Derby、HSQLDB、MSSQLServer、MySQL、Oracle、PostgreSQL
- AUTO_RECONNECT=TRUE：连接丢失后自动重新连接。
- AUTO_SERVER=TRUE：启动自动混合模式，允许开启多个连接，该参数不支持在内存中运行模式。
- TRACE_LEVEL_SYSTEM_OUT、TRACE_LEVEL_FILE：输出跟踪日志到控制台或文件， 取值0为OFF，1为ERROR（默认值），2为INFO，3为DEBUG。 记录到 *.trace.db 文件中。
- SET TRACE_MAX_FILE_SIZE mb：设置跟踪日志文件的大小，默认为16M。
- DATABASE_TO_LOWER：true 或 false。这个参数是用来指定H2在创建数据库与表时，false 表示不会默认转换为大写，因为H2对大小写明感，会自动将英文转换为大写。
- NON_KEYWORDS：忽略H2数据库关键字。NON_KEYWORDS=USER,account,value,key,month



## H2 数据库 - Select

选择命令用于从表或多个表中提取记录数据。如果我们设计一个选择查询，那么它将以结果表的形式返回数据，称为**结果集**。

> 语法

SELECT语句的基本语法如下：

```sql
SELECT [ TOP term ] [ DISTINCT | ALL ] selectExpression [,...] 
FROM tableExpression [,...] [ WHERE expression ] 
[ GROUP BY expression [,...] ] [ HAVING expression ] 
[ { UNION [ ALL ] | MINUS | EXCEPT | INTERSECT } select ] [ ORDER BY order [,...] ] 
[ [ LIMIT expression ] [ OFFSET expression ] [ SAMPLE_SIZE rowCountInt ] ] 
[ FOR UPDATE ]
```

要获取所有可用字段，请使用以下语法。

```sql
SELECT * FROM table_name;
```

> 示例

考虑具有以下记录的CUSTOMER表：

```sql
+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  6 | Komal    |  22 | MP        |  4500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
+----+----------+-----+-----------+----------+ 
```

要将客户表与给定的数据一起获取，请执行以下查询。

```sql
CREATE TABLE CUSTOMER (id number, name varchar(20), age number, address varchar(20), salary number);

INSERT into CUSTOMER values (1, 'Ramesh', 32, 'Ahmedabad', 2000); 
INSERT into CUSTOMER values (2, 'Khilan', 25, 'Delhi', 1500); 
INSERT into CUSTOMER values (3, 'kaushik', 23, 'Kota', 2000); 
INSERT into CUSTOMER values (4, 'Chaitali', 25, 'Mumbai', 6500); 
INSERT into CUSTOMER values (5, 'Hardik', 27, 'Bhopal', 8500); 
INSERT into CUSTOMER values (6, 'Komal', 22, 'MP', 4500); 
INSERT into CUSTOMER values (7, 'Muffy', 24, 'Indore', 10000);
```

以下命令是一个示例，它将获取CUSTOMER表中可用的客户的ID，Name和Salary字段。

```sql
SQL> SELECT ID, NAME, SALARY FROM CUSTOMERS;

+----+----------+----------+ 
| ID | NAME     | SALARY   | 
+----+----------+----------+ 
|  1 | Ramesh   |  2000.00 | 
|  2 | Khilan   |  1500.00 | 
|  3 | kaushik  |  2000.00 | 
|  4 | Chaitali |  6500.00 | 
|  5 | Hardik   |  8500.00 | 
|  6 | Komal    |  4500.00 | 
|  7 | Muffy    | 10000.00 | 
+----+----------+----------+ 
```

使用以下查询来获取CUSTOMERS表的所有字段。

```sql
SQL> SELECT * FROM CUSTOMERS;

+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  6 | Komal    |  22 | MP        |  4500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
+----+----------+-----+-----------+----------+ 
```



## H2 数据库 - Insert

SQL INSERT 语句用于向数据库中的表添加新的数据行.

> 语法

以下是INSERT INTO语句的基本语法。

```sql
INSERT INTO tableName 
{ [ ( columnName [,...] ) ] 
{ VALUES 
{ ( { DEFAULT | expression } [,...] ) } [,...] | [ DIRECT ] [ SORTED ] select } } | 
{ SET { columnName = { DEFAULT | expression } } [,...] }
```

使用此INSERT语句，我们可以在表中插入新记录或新行。使用DIRECT子句时，结果将直接影响目标表，而无需任何中间步骤。但是，在为表的所有列添加值时，请确保值的顺序与表中的列的顺序相同。

> 示例

让我们以一个示例为例，尝试将以下给定记录插入到Customer表中。

|  ID  |   Name   | Age  |  Address  | Salary |
| :--: | :------: | :--: | :-------: | :----: |
|  1   |  Ramesh  |  32  | Ahmedabad |  2000  |
|  2   |  Khilan  |  25  |   Delhi   |  1500  |
|  3   | Kaushik  |  23  |   Kota    |  2000  |
|  4   | Chaitail |  25  |  Mumbai   |  6500  |
|  5   |  Hardik  |  27  |  Bhopal   |  8500  |
|  6   |  Komal   |  22  |    MP     |  4500  |
|  7   |  Muffy   |  24  |  Indore   | 10000  |

通过执行以下命令，我们可以将所有给定的记录获取到客户表中。

```sql
INSERT INTO CUSTOMER VALUES (1, 'Ramesh', 32, 'Ahmedabad', 2000); 
INSERT INTO CUSTOMER VALUES (2, 'Khilan', 25, 'Delhi', 1500); 
INSERT INTO CUSTOMER VALUES (3, 'kaushik', 23, 'Kota', 2000); 
INSERT INTO CUSTOMER VALUES (4, 'Chaitali', 25, 'Mumbai', 6500); 
INSERT INTO CUSTOMER VALUES (5, 'Hardik', 27, 'Bhopal', 8500); 
INSERT INTO CUSTOMER VALUES (6, 'Komal', 22, 'MP', 4500); 
INSERT INTO CUSTOMER VALUES (7, 'Muffy', 24, 'Indore', 10000);
```



## H2 数据库 - Update

UPDATE 查询用于更新或修改表中的现有记录。我们可以将 WHERE 子句与 UPDATE 查询一起使用来更新选定的行，否则所有行都会受到影响。

> 语法

下面是 UPDATE 查询的基本语法。

```sql
UPDATE tableName [ [ AS ] newTableAlias ] SET 
{ { columnName = { DEFAULT | expression } } [,...] } | 
{ ( columnName [,...] ) = ( select ) } 
[ WHERE expression ] [ ORDER BY order [,...] ] [ LIMIT expression ]
```

在这个 UPDATE 语法中，我们可以通过使用 AND 或 OR 子句组合多个条件。

> 示例

考虑具有以下记录的 CUSTOMER 表。

```sql
+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  6 | Komal    |  22 | MP        |  4500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
+----+----------+-----+-----------+----------+ 
```

如果要获取客户表以及给定数据，请执行以下查询。

```sql
CREATE TABLE CUSTOMER (id number, name varchar(20), age number, address varchar(20), salary number);

INSERT into CUSTOMER values (1, 'Ramesh', 32, 'Ahmedabad', 2000); 
INSERT into CUSTOMER values (2, 'Khilan', 25, 'Delhi', 1500); 
INSERT into CUSTOMER values (3, 'kaushik', 23, 'Kota', 2000); 
INSERT into CUSTOMER values (4, 'Chaitali', 25, 'Mumbai', 6500); 
INSERT into CUSTOMER values (5, 'Hardik', 27, 'Bhopal', 8500); 
INSERT into CUSTOMER values (6, 'Komal', 22, 'MP', 4500); 
INSERT into CUSTOMER values (7, 'Muffy', 24, 'Indore', 10000);
```

以下命令是一个示例，它将更新 ID 为 6 的客户的 ADDRESS：

```sql
UPDATE CUSTOMERS SET ADDRESS = 'Pune' WHERE ID = 6;
```

现在，客户表将具有以下记录。我们可以通过执行以下查询来检查客户表记录。

```sql
SELECT * FROM CUSTOMERS;

+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  6 | Komal    |  22 | Pune      |  4500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
+----+----------+-----+-----------+----------+ 
```

要修改CUSTOMERS表中的所有ADDRESS和SALARY列值，我们不需要使用WHERE子句。UPDATE查询如下：

```sql
UPDATE CUSTOMERS SET ADDRESS = 'Pune', SALARY = 1000.00; 
```

现在，CUSTOMERS表将具有以下记录。我们可以通过执行以下查询来检查客户表记录。

```sql
SELECT * FROM CUSTOMERS;

+----+----------+-----+---------+---------+ 
| ID | NAME     | AGE | ADDRESS | SALARY  | 
+----+----------+-----+---------+---------+ 
|  1 | Ramesh   |  32 | Pune    | 1000.00 | 
|  2 | Khilan   |  25 | Pune    | 1000.00 | 
|  3 | kaushik  |  23 | Pune    | 1000.00 | 
|  4 | Chaitali |  25 | Pune    | 1000.00 | 
|  5 | Hardik   |  27 | Pune    | 1000.00 | 
|  6 | Komal    |  22 | Pune    | 1000.00 | 
|  7 | Muffy    |  24 | Pune    | 1000.00 | 
+----+----------+-----+---------+---------+ 
```



## H2 数据库 - Delete

SQL DELETE 查询用于从表中删除现有记录。我们可以使用WHERE子句与DELETE查询来删除选定的记录，否则所有的记录将被删除。

> 语法

以下是通用查询语法删除命令.

```sql
DELETE [ TOP term ] FROM tableName [ WHERE expression ] [ LIMIT term ]
```

上述语法从表中删除行。如果指定了TOP或LIMIT，则最多删除指定的行数（如果为零或小于零，则无限制）。

> 示例

考虑具有以下记录的CUSTOMER表。

```sql
+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  6 | Komal    |  22 | MP        |  4500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
+----+----------+-----+-----------+----------+
```

以下命令将删除客户的ID，其ID为6。

```sql
DELETE FROM CUSTOMERS WHERE ID = 6;
```

执行上述命令后，通过执行以下命令检查客户表。

```sql
SELECT * FROM CUSTOMERS; 

+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
+----+----------+-----+-----------+----------+ 
```

如果要删除CUSTOMERS表中的所有记录，我们不使用WHERE子句。DELETE查询如下。

```sql
DELETE FROM CUSTOMER; 
```

执行上述命令后，客户表中将不会有任何记录。



## H2 数据库 - Backup

BACKUP是用于将数据库备份转换为单独的.zip文件的命令.对象未锁定，在进行备份时，也会复制事务日志.执行此命令需要管理员权限。

> 语法

以下是备份命令的通用语法。

```sql
BACKUP TO fileNameString;
```

> 示例

在这个例子中，我们将当前数据库备份到**backup.zip**文件中。使用相同的命令。

```sql
BACKUP TO 'backup.zip';
```

执行上述命令时，您将在本地文件系统中获取backup.zip文件。



## H2 数据库 - Merge

MERGE 命令用于更新现有行并将新行插入表中.使用此命令时，主键列起着重要作用;它用于查找行。

> 语法

以下是MERGE命令的通用语法。

```sql
MERGE INTO tableName [ ( columnName [,...] ) ] 
[ KEY ( columnName [,...] ) ] 
{ VALUES { ( { DEFAULT | expression } [,...] ) } [,...] | select } 
```

在上面的语法中，KEY子句用于指定主键列名.与VALUES子句一起，我们可以使用原始值来插入，或者我们可以使用select命令检索并将另一个表值存储到此表中。

> 示例

在此示例中，让我们尝试将新记录添加到Customers表中.以下是表中新记录的详细信息.

| Column Name |   Value   |
| :---------: | :-------: |
|     ID      |     8     |
|    NAME     |  Lokesh   |
|     AGE     |    32     |
|   ADDRESS   | Hyderabad |
|   SALARY    |   2500    |

使用以下查询，让我们将给定记录插入H2数据库查询。

```sql
MERGE INTO CUSTOMER KEY (ID) VALUES (8, 'Lokesh', 32, 'Hyderabad', 2500);
-- 上述查询产生以下输出。
Update count: 1 
```

我们通过执行以下查询来验证客户表的记录。

```sql
SELECT * FROM CUSTOMER;

+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  6 | Komal    |  22 | MP        |  4500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
|  8 | Lokesh   |  32 | Hyderabad |  2500.00 | 
+----+----------+-----+-----------+----------+
```

现在让我们尝试更新使用**合并**命令记录.以下是要更新的记录的详细信息.

|  列名   | Value     |
| :-----: | --------- |
|   ID    | 8         |
|  NAME   | Loki      |
|   AGE   | 32        |
| ADDRESS | Hyderabad |
| SALARY  | 3000      |

使用以下查询将给定的记录插入到H2数据库查询中。

```sql
MERGE INTO CUSTOMER KEY (ID) VALUES (8, 'Loki', 32, 'Hyderabad', 3000);
-- 上述查询产生以下输出。
Update count: 1 
```

我们通过执行以下查询来验证客户表的记录。

```sql
SELECT * FROM CUSTOMER;

+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 | 
|  2 | Khilan   |  25 | Delhi     |  1500.00 | 
|  3 | kaushik  |  23 | Kota      |  2000.00 | 
|  4 | Chaitali |  25 | Mumbai    |  6500.00 | 
|  5 | Hardik   |  27 | Bhopal    |  8500.00 | 
|  6 | Komal    |  22 | MP        |  4500.00 | 
|  7 | Muffy    |  24 | Indore    | 10000.00 | 
|  8 | Loki     |  32 | Hyderabad |  3000.00 | 
+----+----------+-----+-----------+----------+
```





## H2 数据库 - Call

CALL是属于H2数据库服务器的SQL命令。此命令用于计算一个简单的表达式。它将给定表达式的结果返回到单个列字段中。当它返回结果数组时，数组中的每个元素都将显示为列值。

> 语法

以下是CALL命令的通用语法。

```sql
CALL expression;
```

我们可以在这个语法中使用算术表达式。

> 示例

让我们举个例子，并使用call命令执行一个算术表达式（15 * 25）。

```sql
CALL 15*25; 
```

上述命令产生以下输出。

| 375  |
| :--: |
| 375  |



## H2 数据库 - Explain

EXPLAIN命令显示语句的执行计划。当我们使用EXPLAIN ANALYZE命令执行语句时，查询计划将包括每个表的实际行扫描计数。

> 语法

以下是EXPLAIN命令的通用语法。

```sql
EXPLAIN { [ PLAN FOR ] | ANALYZE } { select | insert | update | delete | merge} 
```

除了这种语法，我们可以使用select，insert，delete和merge。

> 示例

此示例说明ID 1的客户的查询计划详细信息。

```sql
EXPLAIN SELECT * FROM CUSTOMER WHERE ID = 1;
```

上述命令产生以下输出：

```sql
SELECT
    "PUBLIC"."CUSTOMER"."ID",
    "PUBLIC"."CUSTOMER"."NAME",
    "PUBLIC"."CUSTOMER"."AGE",
    "PUBLIC"."CUSTOMER"."ADDRESS",
    "PUBLIC"."CUSTOMER"."SALARY"
FROM "PUBLIC"."CUSTOMER"
    /* PUBLIC.CUSTOMER.tableScan */
WHERE "ID" = CAST(1 AS DECFLOAT)
```



## H2 数据库 - Show

SHOW是一个用于显示表格的模式，表格或列的列表的命令.

> 语法

以下是通用SHOW命令的语法.

```sql
SHOW { SCHEMAS | TABLES [ FROM schemaName ] | COLUMNS FROM tableName [ FROM schemaName ] }
```

> 示例

1、以下命令可用于获取当前数据库中的模式列表。

```sql
SHOW SCHEMAS;
```

| SCHEMA_NAME        |
| :----------------- |
| INFORMATION_SCHEMA |
| PUBLIC             |

2、以下命令可用于获取当前数据库中的表列表。

```sql
SHOW TABLES;
```

上述命令产生以下输出.

| TABLE_NAME | TABLE_SCHEMA |
| :--------: | :----------: |
|  CUSTOMER  |    PUBLIC    |



## H2数据库 - Create

CREATE 是一个通用 SQL 命令，用于在 H2 数据库服务器中创建表，模式，序列，视图和用户。

### 1、Create Table

Create Table 是一个用于在当前数据库中创建用户定义表的命令。

> 语法

以下是Create的通用语法表命令.

```sql
CREATE [ CACHED | MEMORY ] [ TEMP | [ GLOBAL | LOCAL ] TEMPORARY ] 
TABLE [ IF NOT EXISTS ] name 
[ ( { columnDefinition | constraint } [,...] ) ] 
[ ENGINE tableEngineName [ WITH tableEngineParamName [,...] ] ] 
[ NOT PERSISTENT ] [ TRANSACTIONAL ] 
[ AS select ] 
```

通过使用 Create Table 命令的通用语法，我们可以创建不同类型的表，例如缓存表，内存表和临时表.以下是用于描述给定语法中的不同子句的列表。

- **CACHED（缓存表）**：缓存表是常规表的默认类型，行数不受主内存限制。

- **MEMORY（内存表）**：内存表是临时表的默认类型。内存表不应太大，索引数据应保留在主内存中。

- **TEMPORARY**：闭或打开数据库时，将删除临时表。基本上，临时表有两种类型：

  - GLOBAL type：所有连接均可访问。

  - LOCAL type：可通过当前连接访问。

    临时表的默认类型是全局类型。临时表的索引保存在主内存中，除非使用CREATE CACHED TABLE创建临时表。

- **ENGINE**：仅在使用自定义表实现时才需要ENGINE选项。

- **NOT PERSISTENT**：它是一个修饰符，用于将完整的表数据保留在内存中，并且在关闭数据库时所有行都将丢失。

- **TRANSACTIONAL**：它是提交未完成事务的关键字，此命令仅支持临时表。

> 示例

在这个例子中，让我们使用以下给定数据创建一个名为tutorials_tbl的表。

| Sr.No | 列名            |  数据类型   |
| :---: | --------------- | :---------: |
|   1   | ID              |     Int     |
|   2   | Title           | Varchar(50) |
|   3   | Author          | Varchar(20) |
|   4   | Submission_date |    日期     |

以下查询用于创建表 **tutorials_tbl** 以及给定的列数据.

```sql
CREATE TABLE tutorials_tbl (
    id INT NOT NULL, 
    title VARCHAR(50) NOT NULL, 
    author VARCHAR(20) NOT NULL,
    submission_date DATE
);
```

上述查询产生以下输出。

```sql
(0) rows effected
```



### 2、Create Schema

创建架构是用于在特定授权下创建依赖于用户的架构的命令（在当前注册的用户下）。

> 语法

以下是Create Schema命令的通用语法。

```sql
CREATE SCHEMA [ IF NOT EXISTS ] name [ AUTHORIZATION ownerUserName ]
```

在以上通用语法中，AUTHORIZATION是用于提供相应用户名的关键字，该命令是可选的这意味着如果无涯教程不提供用户名，则它将考虑当前用户。执行命令的用户必须具有管理员权限以及所有者。

> 示例

在此示例中，让我们使用以下命令在SA用户下创建名为 **test_schema** 的模式。

```sql
CREATE SCHEMA test_schema AUTHORIZATION sa;
```

上述命令产生以下输出。

```sql
(0) rows effected
```



### 3、Create Sequence

> 语法

序列是一个概念，是用于通过跟随ID或序列生成数字的概念任何随机列值.。

以下是Create Sequence命令的通用语法。

```sql
CREATE SEQUENCE [ IF NOT EXISTS ] newSequenceName [ START WITH long ] 
[ INCREMENT BY long ] 
[ MINVALUE long | NOMINVALUE | NO MINVALUE ] 
[ MAXVALUE long | NOMAXVALUE | NO MAXVALUE ] 
[ CYCLE long | NOCYCLE | NO CYCLE ] 
[ CACHE long | NOCACHE | NO CACHE ] 
```

此通用语法用于创建序列，序列的数据类型为 BIGINT ，按照这种顺序，即使回滚了事务，也永远不会重复使用值。

> 示例

在这个例子中，让我们创建一个名为 **SEQ_ID** 的序列，使用以下查询。

```sql
CREATE SEQUENCE SEQ_ID;
```

上述查询产生以下输出。

```sql
(0) rows effected
```



## H2数据库 - Alter

ALTER是一个用于通过向 **alter** 命令添加不同子句来更改表结构的命令。根据场景，我们需要在alter命令中添加相应的子句。在本章中，我们将讨论alter命令的各种场景。

### 1、Alter Table Add 字段

更改表添加是用于添加新列的命令表格以及相应的数据类型。此命令在此连接中提交事务。

> 语法

以下是"**Alter Table Add Column**"命令的通用语法。该命令用于将新列以及相应数据类型添加到表中的命令。

```sql
ALTER TABLE [ IF EXISTS ] tableName ADD [ COLUMN ] 
{ [ IF NOT EXISTS ] columnDefinition [ { BEFORE | AFTER } columnName ] 
   | ( { columnDefinition } [,...] ) }
```

> 示例

在此示例中，我们将在表 tutorials_tbl 中添加新列 start_date ， start_date 的数据类型为 Date。以下是添加新列的查询。

```sql
ALTER TABLE tutorials_tbl ADD start_date DATE;
```

上面的查询产生以下输出。

```sql
(6) rows effected 
```



### 2、Alter Table Add 约束

更改表添加约束是用于向表添加不同约束（例如主键，外键，not null等）的命令，如果所需的索引尚不存在，则会自动创建它们无法禁用唯一约束检查。此命令在此连接中提交一个打开的事务。

> 语法

以下是"**Alter Table add Constraint**"命令的通用语法。

```sql
ALTER TABLE [ IF EXISTS ] tableName ADD constraint [ CHECK | NOCHECK ] 
```

> 示例

在此示例中，我们使用以下查询，将一个主键约束**（tutorials_tbl_pk）**添加到**tutorials_tbl**表的列中。

```sql
ALTER TABLE tutorials_tbl ADD CONSTRAINT tutorials_tbl_pk PRIMARYKEY(id); 
```

上面的查询产生以下输出。

```sql
(6) row (s) effected
```



### 3、Alter Table Alter 约束

此命令用于更改特定表的列的结构和属性，更改属性意味着更改列的数据类型，重命名列，更改标识值或更改选择性。

> 语法

以下是"**Alter Table Alter Column**"命令的通用语法：

```sql
ALTER TABLE [ IF EXISTS ] tableName ALTER COLUMN columnName 
{ { dataType [ DEFAULT expression ] [ [ NOT ] NULL ] [ AUTO_INCREMENT | IDENTITY ] } 
| { RENAME TO name } 
| { RESTART WITH long } 
| { SELECTIVITY int } 
| { SET DEFAULT expression } 
| { SET NULL } 
| { SET NOT NULL } } 
```

在上面的语法：

- **RESTART**：命令更改自动增量列的下一个值。
- **SELECTIVITY**：命令设置列的选择性（1-100）。基于选择性值，我们可以对列的值进行成像。
- **SET DEFAULT**：更改列的默认值。
- **SET NULL**：将列设置为允许NULL。
- **SET NOT NULL**：设置列允许NOT NULL。

> 示例

在这个例子中，我们将使用以下查询将表**tutorials_tbl**的列重命名为**Title**到**Tutorial_Title**。

```sql
ALTER TABLE tutorials_tbl ALTER COLUMN title RENAME TO tutorial_title; 
```

上述查询产生以下输出。

```sql
(0) row(s) effected 
```

以类似的方式，我们可以使用ALTER命令执行不同的场景。



### 4、Alter Table Rename 约束

此命令用于重命名特定关系表的约束名称。此命令在此连接中提交一个打开的事务。

> 语法

以下是"**Alter Table Rename Constraint**"命令的通用语法。

```sql
ALTER TABLE [ IF EXISTS ] tableName RENAME oldConstraintName TO newConstraintName
```

使用此语法时，确保旧约束名称应与相应列一起存在。

> 示例

在此示例中，我们将更改主键约束名称表 **tutorials_tbl** 从 **tutorials_tbl_pk** 到 **tutorials_tbl_pk_constraint** 。以下是这样做的查询。

```sql
ALTER TABLE tutorials_tbl RENAME CONSTRAINT tutorials_tbl_pk TO tutorials_tbl_pk_constraint;
```

以上查询产生以下输出。

```sql
(1) row (s) effected
```



## H2数据库 - Drop

DROP是一个取自通用SQL语法的命令。此命令用于从内存中删除数据库组件及其结构。Drop命令有不同的场景我们将在本章讨论。

### 1、Drop Table

Drop Table是一个删除相应表的命令及其结构。

> 语法

以下是Drop Table命令的通用语法。

```sql
DROP TABLE [ IF EXISTS ] tableName [,...] [ RESTRICT | CASCADE ]
```

如果我们使用RESTRICT并且存在具有相关视图的表，则命令将失败。当我们使用CASCADE关键字时，将删除所有相关视图。

> 示例

在此示例中，我们将使用以下内容删除名为test的表查询。

```sql
DROP TABLE test;
```

上述查询产生以下输出。

```sql
(6) row (s) effected
```



### 2、Drop Schema

Drop Schema是一个从数据库服务器中删除相应架构的命令.它不适用于当前架构。

> 语法

```sql
DROP SCHEMA [ IF EXISTS ] schemaName
```

> 示例

在此示例中，我们将使用以下内容删除名为 **test_schema** 的模式查询。

```sql
DROP SCHEMA TEST_SCHEMA;
```

上述查询产生以下输出。

```sql
(0) row(s) effected
```



### 3、Drop Sequence

Drop Sequence是一个用于从表结构中删除序列的命令。

> 语法

以下是Drop Sequence命令的通用语法。此命令在此连接中提交一个打开的事务。

```sql
DROP SEQUENCE [ IF EXISTS ] sequenceName
```

> 示例

在此示例中，我们将删除名为 **sequence_id** 的序列.以下是命令。

```sql
DROP SEQUENCE sequence_id;
```

上述命令产生以下输出。

```sql
(0) row (s) effected
```



### 4、Drop View

> 语法

以下是Drop View命令的通用语法。

```sql
DROP VIEW [ IF EXISTS ] viewName [ RESTRICT | CASCADE ]
```

> 示例

在此示例中，我们将删除名为 **sample_view** 的视图使用以下查询。

```sql
DROP VIEW sample_view;
```

上述查询产生以下输出。

```sql
(0) row (s) effected
```



## H2数据库 - Truncate

TRUNCATE是用于从表中删除数据的命令。与DELETE FROM without WHERE子句不同，此命令无法回滚.此命令在此连接中提交一个打开的事务。

> 语法

以下是truncate命令的通用语法.

```sql
TRUNCATE TABLE tableName
```

> 示例

在此示例中，我们将使用以下查询截断名为 **test** 的表。

```sql
TRUNCATE TABLE test;
```

上述查询产生以下输出。

```sql
(6) row (s) effected
```



## H2数据库 - Commit

COMMIT是来自用于提交事务的SQL语法的命令。我们可以提交特定的事务，也可以提交当前执行的事务。

> 语法

COMMIT命令有两种不同的语法

- 以下是commit命令提交当前事务的通用语法

  ```sql
  COMMIT [WORK]
  ```

- 以下是commit命令提交特定事务的通用语法

  ```sql
  COMMIT TRANSACTION transactionName
  ```

> 示例1

在本例中，让我们使用以下内容提交当前事务命令。

```sql
COMMIT
```

上述命令产生以下输出.

```sql
Committed successfully
```

> 示例2

在这个例子中，我们将使用followin提交名为 **tx_test** 的事务g命令。

```sql
COMMIT TRANSACTION tx_test;
```

上述命令产生以下输出。

```sql
Committed successfully
```



## H2数据库 - Grant

Grant 是一个来自SQL语法的命令，用于向表，用户或角色授予权限.执行此命令需要管理员权限.此命令在此连接中提交一个打开的事务。在本章中，我们将讨论Grant命令的不同场景。

### 1、Grant Right

授予权限是用于向表，用户或角色提供管理员权限的命令。

> 语法

以下是Grant命令的通用语法。

```sql
GRANT { SELECT | INSERT | UPDATE | DELETE | ALL } [,...] ON 
{ { SCHEMA schemaName } | { tableName [,...] } } 
TO { PUBLIC | userName | roleName }
```

在此示例中，无涯教程将使用以下命令将测试表授予只读权限。

```sql
GRANT SELECT ON TEST TO READONLY
```

上面的命令产生以下输出。

```sql
Grant successfully
```



### 2、Grant Alter Any Schema

Grant Alter Any Schema 是一个向相应用户授予模式更改权限的命令。【授予更改任何架构】

> 语法

以下是Grant Alter Any Schema命令的通用语法。

```sql
GRANT ALTER ANY SCHEMA TO userName
```

> 示例

在此示例中，我们将授予模式的更改权限给名为 **test_user** 的用户。确保test_user存在.以下是授予更改权限的查询。

```sql
GRANT ALTER SCHY TO test_user;
```

以上查询产生以下输出。

```sql
Granted successfully to test_user
```



## H2数据库 - Savepoint

SAVEPOINT是用于临时保存事务的命令。最好在事务中维护保存点，因为在必要时将事务回滚到相应的Savepoint会很有帮助。

> 语法

以下是Savepoint命令的通用语法。

```sql
SAVEPOINT savepointName
```

> 示例

在这个例子中，我们将使用以下命令创建一个名为Half_Done的Savepoint。

```sql
SAVEPOINT Half_Done ;
```

上述命令产生以下输出。

```sql
Savepoint created
```



## H2数据库 - Rollback

ROLLBACK是一种来自SQL语法的命令，用于将事务回滚到Savepoint或前一个事务.通过使用此命令，我们可以回滚到特定 Savepoint，或者我们可以回滚到先前执行的事务。

> 语法

ROLLABCK命令的两种不同语法。

- 以下是rollback命令的通用语法。

  ```sql
  ROLLBACK [TO SAVEPOINT savepointName]
  ```

- 以下是特定事务的Rollback命令的通用语法。

  ```sql
  ROLLBACK TRANSACTION transactionName
  ```

> 示例1

在此示例中，我们将回滚使用以下命令将当前事务转换为名为 **sp1_test** 的保存点。

```sql
ROLLBACK sp1_test;
```

上述命令产生以下输出。

```sql
Rollback successfully
```

> 示例2

在以下示例中，我们将回滚名为 **tx_test 使用给定的命令**。

```sql
ROLLBACK TRANSACTION tx_test;
```

上述命令产生以下输出。

```sql
Rollback successfully
```



## H2 数据库 - 查看日志

H2 数据库在文件模式下会生成： **xxx.mv.db** 和 **xxx.trace.db** 文件。

- xxx.mv.db：是H2数据库的主文件，用于保存数据表的结构和数据。
- xxx.trace.db：是H2数据库的跟踪文件，用于记录数据库的操作和日志信息。

1、首先我们先故意执行一些错误操作：建表时字段使用到H2的关键字导致报错：

```sql
DROP TABLE IF EXISTS TEST;
CREATE TABLE TEST(ID INT PRIMARY KEY,VALUEVARCHAR(255));

Syntax error in SQL statement "CREATE TABLE TEST(ID INT PRIMARY KEY,VALUEVARCHAR[*](255))"; expected "IDENTITY, BIGSERIAL, SERIAL, data type"; SQL statement:
CREATE TABLE TEST(ID INT PRIMARY KEY,VALUEVARCHAR(255)) [42001-220] 42001/42001 (Help)
```

2、上面已经提到过，类似 `test.trace.db` 这样的文件，就是H2的错误信息，我们打开看看：

```shell
2023-08-04 13:24:32 jdbc[3]: exception
org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "CREATE TABLE TEST(ID INT PRIMARY KEY,VALUEVARCHAR[*](255))"; expected "IDENTITY, BIGSERIAL, SERIAL, data type"; SQL statement:
CREATE TABLE TEST(ID INT PRIMARY KEY,VALUEVARCHAR(255)) [42001-220]
```

我们主要去找 `[*]` 这个符号和后面的错误码，然后对照着官方的错误码说明看，就知道哪里报错了。

我们看看`[42001-220]`这个错误码，前面的 42001 是错误码，后面的 220 的 h2 的版本。

大概意思就是：尝试执行无效的SQL语句时引发代码42001的错误，原因就是 H2 建表时如果列名或表名中出现了关键字，就会报错。我们这里避开关键字即可。或者和再URL后面加上参数：

```sql
jdbc:h2:mem:test;NON_KEYWORDS=USER,account,value,key,month
```





## H2数据库 - 数据类型

H2支持如下数据类型：

1. INT Type
2. BOOLEAN Type
3. TINYINT Type
4. SMALLINT Type
5. BIGINT Type
6. IDENTITY Type
7. DECIMAL Type
8. DOUBLE Type
9. REAL Type
10. TIME Type
11. DATE Type
12. TIMESTAMP Type
13. BINARY Type
14. OTHER Type
15. VARCHAR Type
16. VARCHAR_IGNORECASE Type
17. CHAR Type
18. BLOB Type
19. CLOB Type
20. UUID Type
21. ARRAY Type
22. GEOMETRY Type

1、INT Type

- 分别有 INT、INTEGER、MEDIUMINT、INT4、SIGNED
- 值范围：-2147483648 – 2147483647

2、BOOLEAN Type

- 可选值：TRUE和FALSE

3、TINYINT Type

- 值范围：-128 -127

4、SMALLINT Type

- 值范围： -32768 to 32767

5、BIGINT Type

- 值范围：-9223372036854775808 – 9223372036854775807

6、IDENTITY Type

- 自动增长值
- 值范围：-9223372036854775808 – 9223372036854775807

7、DECIMAL Type

- 示例：DECIMAL(20, 2)

8、DOUBLE Type

9、REAL Type

- 浮点类型

10、TIME Type

- 时间类型，格式为：hh:mm:ss

11 、DATE Type

- 日期类型，格式为：yyyy-MM-dd

12、TIMESTAMP Type

- 时间戳类型，格式为：yyyy-MM-dd hh:mm:ss[.nnnnnnnnn]
- 可映射为java对象：java.sql.Timestamp/java.util.Date

13、BINARY Type

- 存储字节流，大数据建议使用BLOB类型存储
- 示例：BINARY(1000)

14、OTHER Type

- 用于存储序列化的java对象

15、VARCHAR Type

- 存储字符串，大文本数据建议使用CLOB类型存储
- 示例：VARCHAR(255)

16、VARCHAR_IGNORECASE Type

- 与VARCHAR类似，但是不区分大小写

17、CHAR Type

- 存储unicode字符串，与VARCHAR的区别是，尾部的空格不会被存储

18、BLOB Type

- 存储大容量二进制流，最大存储2G数据
- 映射java对象类型为：java.sql.Blob /java.io.InputStream

19、CLOB Type

- 用于存储大容量的文本数据
- 映射java对象类型为：java.sql.Clob/java.io.Reader

20、UUID Type

- 存储通用唯一识别码，占用128bit

21、ARRAY Type

- 数组类型，使用一个值列表(1,2)或PreparedStatement.setObject(.., new Object[] {..})存储值
- 映射java对象类型为：java.lang.Object[]

22、GEOMETRY Type

- 存储空间几何对象，一般存储WKT格式的空间数据



## H2数据库 - 创建集群

H2支持两台服务器运行两个数据库成为集群，两个数据库互为备份，如果一个服务器失效，另一个服务器仍然可以工作。另外只有服务模式支持集群配置。H2可以通过CreateCluster工具创建集群，示例步骤如下（在在一台服务器上模拟两个数据库组成集群）：

1、创建目录，创建两个服务器工作的目录



```bash
# mkdir server1
# mkdir server2
```

2、启动tcp服务，执行如下命令分别在9101、9102端口启动两个使用tcp服务模式的数据库

```bash
java org.h2.tools.Server-tcp -tcpPort 9101-baseDir server1
java org.h2.tools.Server-tcp -tcpPort 9102 -baseDir server2
```

3、使用CreateCluster工具创建集群，如果两个数据库不存在，该命令将会自动创建数据库。如果一个数据库失效，可以先删除坏的数据库文件，重新启动数据库，然后重新运行CreateCluster工具

```bash
java org.h2.tools.CreateCluster
 -urlSource jdbc:h2:tcp://localhost:9101/~/test
 -urlTarget jdbc:h2:tcp://localhost:9102/~/test
 -user sa
 -serverList localhost:9101,localhost:9102
```

4、连接数据库，现在可以使用如下连接字符串连接集群数据库

```bash
jdbc:h2:tcp://localhost:9101,localhost:9102/~/test
```

5、监控集群运行状态，可以使用如下命令查看配置的集群服务器是否都在运行

```sql
SELECT VALUE FROM INFORMATION_SCHEMA.SETTINGS WHERE NAME='CLUSTER'
```

6、限制，H2的集群并不支持针对事务的负载均衡，所以很多操作会使两个数据库产生不一致的结果，执行如下操作时请小心：

```sql
RANDOM_UUID(), SECURE_RAND(), SESSION_ID(), MEMORY_FREE(), MEMORY_USED(), CSVREAD(), CSVWRITE(), RAND()
```

自动增长列和标识列不支持集群，当插入数据时，序列值需要手动创建。不支持SET AUTOCOMMIT FALSE语句，如果需要设置成为不自动提交，可以执行方法Connection.setAutoCommit(false);



## H2数据库 - JDBC 连接

H2是一个JAVA数据库，我们可以使用JDBC与此数据库进行交互。在本章中，我们将了解如何使用H2数据库创建JDBC连接以及如何使用H2数据库创建CRUD操作。

通常，创建JDBC连接有五个步骤：

**第1步** : 注册JDBC数据库驱动程序

```java
Class.forName("org.h2.Driver");
```

**第2步** : 打开连接

```java
Connection conn = DriverManager.getConnection ("jdbc:h2:~/test", "sa","");
```

**第3步** : 创建语句

```
Statement st = conn.createStatement();
```

**第4步** : 执行语句并接收Resultset

```sql
Stmt.executeUpdate("sql statement");
```

**第5步** : 关闭连接

```
conn.close();
```

在创建完整程序之前，我们需要将 **h2-*.jar文件** 添加到CLASSPATH.我们可以从文件夹 **C:\Program Files(x86)\H2 \bin** 中获取 **jar** .

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class H2Test {

    /**
     * 以嵌入式(本地)连接方式连接H2数据库
     */
    private static final String JDBC_URL = "jdbc:h2:E:/Java/H2Test/user";

    /**
     * 使用TCP/IP的服务器模式(远程连接)方式连接H2数据库(推荐)
     */
    // private static final String JDBC_URL = "jdbc:h2:tcp://10.35.14.122/C:/H2/user";

    private static final String USER = "root";
    private static final String PASSWORD = "root";
    private static final String DRIVER_CLASS = "org.h2.Driver";

    public static void main(String[] args) throws Exception {
        // TODO Auto-generated method stub
        Class.forName(DRIVER_CLASS);
        Connection conn = DriverManager.getConnection(JDBC_URL, USER, PASSWORD);
        Statement statement = conn.createStatement();
        statement.execute("DROP TABLE IF EXISTS USER_INF");
        statement.execute("CREATE TABLE USER_INF(id INTEGER PRIMARY KEY, name VARCHAR(100), sex VARCHAR(2))");

        statement.executeUpdate("INSERT INTO USER_INF VALUES(1, 'tom', '男') ");
        statement.executeUpdate("INSERT INTO USER_INF VALUES(2, 'jack', '女') ");
        statement.executeUpdate("INSERT INTO USER_INF VALUES(3, 'marry', '男') ");
        statement.executeUpdate("INSERT INTO USER_INF VALUES(4, 'lucy', '男') ");

        ResultSet resultSet = statement.executeQuery("select * from USER_INF");

        while (resultSet.next()) {
            System.out.println(
                resultSet.getInt("id") + ", " + resultSet.getString("name") + ", " + resultSet.getString("sex"));
        }

        statement.close();
        conn.close();
    }

}
```



### 1、创建表

在本例中，我们将为create table编写一个程序.考虑一个名为注册的表，其中包含以下字段.

| S.No | 列名 |   数据类型   | NOT NULL | 主键 |
| :--: | ---- | :----------: | :------: | :--: |
|  1   | ID   |     数字     |    是    |  是  |
|  2   | 首先 | Varchar(255) |    否    |  否  |
|  3   | 最后 | Varchar(255) |    否    |  否  |
|  4   | 年龄 |     数字     |    否    |  否  |

以下是名为 **H2jdbcCreateDemo** 的示例程序.

```java
import java.sql.Connection; 
import java.sql.DriverManager; 
import java.sql.SQLException; 
import java.sql.Statement;  

public class H2jdbcCreateDemo { 
    // JDBC driver name and database URL 
    static final String JDBC_DRIVER = "org.h2.Driver";   
    static final String DB_URL = "jdbc:h2:~/test";  

    // Database credentials 
    static final String USER = "sa"; 
    static final String PASS = ""; 

    public static void main(String[] args) { 
        Connection conn = null; 
        Statement stmt = null; 
        try { 
            // STEP 1: 注册 JDBC 驱动程序
            Class.forName(JDBC_DRIVER); 

            // STEP 2: 打开连接
            System.out.println("Connecting to database..."); 
            conn = DriverManager.getConnection(DB_URL,USER,PASS);  

            // STEP 3: 执行查询
            System.out.println("Creating table in given database..."); 
            stmt = conn.createStatement(); 
            String sql =  "CREATE TABLE   REGISTRATION " + 
                "(id INTEGER not NULL, " + 
                " first VARCHAR(255), " +  
                " last VARCHAR(255), " +  
                " age INTEGER, " +  
                " PRIMARY KEY ( id ))";  
            stmt.executeUpdate(sql);
            System.out.println("Created table in given database..."); 

            // STEP 4:关闭连接
            stmt.close(); 
            conn.close(); 
        } catch(SQLException se) { 
            // 处理 JDBC 错误
            se.printStackTrace(); 
        } catch(Exception e) { 
            // 处理 Class.forName 的错误
            e.printStackTrace(); 
        } finally { 
            // finally 块用于关闭资源
            try{ 
                if(stmt!=null) stmt.close(); 
            } catch(SQLException se2) { 
            } 
            try { 
                if(conn!=null) conn.close(); 
            } catch(SQLException se){ 
                se.printStackTrace(); 
            } // end finally try 
        } // end try 
        System.out.println("Goodbye!");
    } 
}
```

将上述程序保存到H2jdbcCreateDemo.java中.通过在命令提示符中执行以下命令来编译并执行上述程序.

```bash
> javac H2jdbcCreateDemo.java
> java H2jdbcCreateDemo
```

上述命令产生以下输出.

```bash
Connecting to database... 
Creating table in given database... 
Created table in given database... 
Goodbye!
```

执行此操作后，我们可以检查使用H2 SQL接口创建的表。



### 2、插入数据

在这个例子中，我们将编写一个用于插入记录的程序.让我们将以下记录插入表注册。

|  ID  | First  | Last  | 年龄 |
| :--: | :----: | :---: | :--: |
| 100  |  Zara  |  Ali  |  18  |
| 101  | Mahnaz | Fatma |  25  |
| 102  |  Zaid  | Khan  |  30  |
| 103  | Sumit  | Mital |  28  |

以下是名为 **H2jdbcInsertDemo** 的示例程序。

```java
import java.sql.Connection; 
import java.sql.DriverManager; 
import java.sql.SQLException; 
import java.sql.Statement;  

public class H2jdbcInsertDemo { 
    //JDBC 驱动程序名称和数据库 URL
    static final String JDBC_DRIVER = "org.h2.Driver";   
    static final String DB_URL = "jdbc:h2:~/test";  

    //数据库凭据
    static final String USER = "sa"; 
    static final String PASS = ""; 

    public static void main(String[] args) { 
        Connection conn = null; 
        Statement stmt = null; 
        try{
            // STEP 1: 注册 JDBC 驱动程序
            Class.forName(JDBC_DRIVER);  

            // STEP 2: 打开连接
            System.out.println("Connecting to a selected database..."); 
            conn = DriverManager.getConnection(DB_URL,USER,PASS); 
            System.out.println("Connected database successfully..."); 

            // STEP 3: 执行查询
            stmt = conn.createStatement();  
            
            String sql = "INSERT INTO Registration " + "VALUES (100, 'Zara', 'Ali', 18)"; 
            stmt.executeUpdate(sql); 
            
            sql = "INSERT INTO Registration " + "VALUES (101, 'Mahnaz', 'Fatma', 25)";  
            stmt.executeUpdate(sql); 
            
            sql = "INSERT INTO Registration " + "VALUES (102, 'Zaid', 'Khan', 30)"; 
            stmt.executeUpdate(sql); 
            
            sql = "INSERT INTO Registration " + "VALUES(103, 'Sumit', 'Mittal', 28)"; 
            stmt.executeUpdate(sql); 
            System.out.println("Inserted records into the table..."); 

            // STEP 4: 关闭链接
            stmt.close(); 
            conn.close(); 
        } catch(SQLException se) { 
            // 处理 JDBC 错误
            se.printStackTrace(); 
        } catch(Exception e) { 
            // 处理 Class.forName 的错误
            e.printStackTrace(); 
        } finally { 
            // finally 块用于关闭资源
            try {
                if(stmt!=null) stmt.close();  
            } catch(SQLException se2) { 
            } // nothing we can do 
            try { 
                if(conn!=null) conn.close(); 
            } catch(SQLException se) { 
                se.printStackTrace(); 
            } // end finally try 
        } // end try 
        System.out.println("Goodbye!"); 
    } 
}
```

将上述程序保存到H2jdbcInsertDemo.java中.通过在命令提示符中执行以下命令来编译并执行上述程序。

```bash
> javac H2jdbcInsertDemo.java 
> java H2jdbcInsertDemo
```

上述命令产生以下输出。

```bash
Connecting to a selected database... 
Connected database successfully... 
Inserted records into the table... 
Goodbye!
```



### 3、读取数据

在本例中，我们将编写一个用于读取记录的程序.让我们尝试从表**注册**中读取所有记录。

以下是一个名为 **H2jdbcRecordDemo** 的示例程序。

```java
import java.sql.Connection; 
import java.sql.DriverManager; 
import java.sql.ResultSet; 
import java.sql.SQLException; 
import java.sql.Statement;  

public class H2jdbcReadDemo { 
    // JDBC driver name and database URL 
    static final String JDBC_DRIVER = "org.h2.Driver";   
    static final String DB_URL = "jdbc:h2:~/test";  

    // Database credentials 
    static final String USER = "sa"; 
    static final String PASS = ""; 

    public static void main(String[] args) { 
        Connection conn = null; 
        Statement stmt = null; 
        try { 
            // STEP 1: Register JDBC driver 
            Class.forName(JDBC_DRIVER); 

            // STEP 2: Open a connection 
            System.out.println("Connecting to database..."); 
            conn = DriverManager.getConnection(DB_URL,USER,PASS);  

            // STEP 3: Execute a query 
            System.out.println("Connected database successfully..."); 
            stmt = conn.createStatement(); 
            String sql = "SELECT id, first, last, age FROM Registration"; 
            ResultSet rs = stmt.executeQuery(sql); 

            // STEP 4: Extract data from result set 
            while(rs.next()) { 
                // Retrieve by column name 
                int id  = rs.getInt("id"); 
                int age = rs.getInt("age"); 
                String first = rs.getString("first"); 
                String last = rs.getString("last");  

                // Display values 
                System.out.print("ID: " + id); 
                System.out.print(", Age: " + age); 
                System.out.print(", First: " + first); 
                System.out.println(", Last: " + last); 
            } 
            // STEP 5: Clean-up environment 
            rs.close(); 
        } catch(SQLException se) { 
            // Handle errors for JDBC 
            se.printStackTrace(); 
        } catch(Exception e) { 
            // Handle errors for Class.forName 
            e.printStackTrace(); 
        } finally { 
            // finally block used to close resources 
            try { 
                if(stmt!=null) stmt.close();  
            } catch(SQLException se2) { 
            } // nothing we can do 
            try { 
                if(conn!=null) conn.close(); 
            } catch(SQLException se) { 
                se.printStackTrace(); 
            } // end finally try 
        } // end try 
        System.out.println("Goodbye!"); 
    } 
}
```

将上述程序保存到H2jdbcReadDemo.java中，通过在命令提示符中执行以下命令来编译并执行上述程序。

```bash
> javac H2jdbcReadDemo.java 
> java H2jdbcReadDemo
```

上述命令产生以下输出。

```bash
Connecting to a selected database... 
Connected database successfully... 
ID: 100, Age: 18, First: Zara, Last: Ali 
ID: 101, Age: 25, First: Mahnaz, Last: Fatma 
ID: 102, Age: 30, First: Zaid, Last: Khan 
ID: 103, Age: 28, First: Sumit, Last: Mittal 
Goodbye!
```



### 4、更新数据

在本例中，我们将编写一个程序来更新记录.让我们尝试阅读表格中的所有记录**注册**。

以下是一个名为 **H2jdbcUpdateDemo** 的示例程序。

```java
import java.sql.Connection; 
import java.sql.DriverManager; 
import java.sql.ResultSet; 
import java.sql.SQLException; 
import java.sql.Statement;  

public class H2jdbcUpdateDemo { 
    // JDBC driver name and database URL 
    static final String JDBC_DRIVER = "org.h2.Driver";   
    static final String DB_URL = "jdbc:h2:~/test";  

    // Database credentials 
    static final String USER = "sa"; 
    static final String PASS = ""; 

    public static void main(String[] args) { 
        Connection conn = null; 
        Statement stmt = null; 
        try { 
            // STEP 1: Register JDBC driver 
            Class.forName(JDBC_DRIVER); 

            // STEP 2: Open a connection 
            System.out.println("Connecting to a database..."); 
            conn = DriverManager.getConnection(DB_URL,USER,PASS);  

            // STEP 3: Execute a query 
            System.out.println("Connected database successfully..."); 
            stmt = conn.createStatement(); 
            String sql = "UPDATE Registration " + "SET age = 30 WHERE id in (100, 101)"; 
            stmt.executeUpdate(sql);  

            // Now you can extract all the records 
            // to see the updated records 
            sql = "SELECT id, first, last, age FROM Registration"; 
            ResultSet rs = stmt.executeQuery(sql);  

            while(rs.next()){ 
                // Retrieve by column name 
                int id  = rs.getInt("id"); 
                int age = rs.getInt("age"); 
                String first = rs.getString("first"); 
                String last = rs.getString("last");  

                // Display values 
                System.out.print("ID: " + id); 
                System.out.print(", Age: " + age); 
                System.out.print(", First: " + first); 
                System.out.println(", Last: " + last); 
            } 
            rs.close();    
        } catch(SQLException se) { 
            // Handle errors for JDBC 
            se.printStackTrace(); 
        } catch(Exception e) { 
            // Handle errors for Class.forName 
            e.printStackTrace(); 
        } finally { 
            // finally block used to close resources  
            try { 
                if(stmt!=null) stmt.close(); 
            } catch(SQLException se2) { 
            } // nothing we can do 
            try { 
                if(conn!=null) conn.close(); 
            } catch(SQLException se) { 
                se.printStackTrace(); 
            } // end finally try 
        } // end try 
        System.out.println("Goodbye!"); 
    } 
}
```

将上述程序保存到H2jdbcUpdateDemo.java中.通过在命令提示符中执行以下命令来编译并执行上述程序。

```bash
> javac H2jdbcUpdateDemo.java 
> java H2jdbcUpdateDemo
```

上述命令产生以下输出。

```bash
Connecting to a selected database... 
Connected database successfully... 
ID: 100, Age: 30, First: Zara, Last: Ali 
ID: 101, Age: 30, First: Mahnaz, Last: Fatma 
ID: 102, Age: 30, First: Zaid, Last: Khan 
ID: 103, Age: 28, First: Sumit, Last: Mittal 
Goodbye!
```



### 5、删除数据

在本例中，我们将编写一个程序来删除记录.让我们尝试从表**注册**中读取所有记录。

以下是一个名为 **H2jdbcDeleteDemo** 的示例程序。

```java
import java.sql.Connection; 
import java.sql.DriverManager; 
import java.sql.ResultSet; 
import java.sql.SQLException; 
import java.sql.Statement;  

public class H2jdbcDeleteDemo { 
    // JDBC driver name and database URL 
    static final String JDBC_DRIVER = "org.h2.Driver";   
    static final String DB_URL = "jdbc:h2:~/test";  

    // Database credentials 
    static final String USER = "sa"; 
    static final String PASS = ""; 

    public static void main(String[] args) { 
        Connection conn = null; 
        Statement stmt = null; 
        try { 
            // STEP 1: Register JDBC driver 
            Class.forName(JDBC_DRIVER);  

            // STEP 2: Open a connection 
            System.out.println("Connecting to database..."); 
            conn = DriverManager.getConnection(DB_URL,USER,PASS);  

            // STEP 3: Execute a query
            System.out.println("Creating table in given database..."); 
            stmt = conn.createStatement();  
            String sql = "DELETE FROM Registration " + "WHERE id = 101"; 
            stmt.executeUpdate(sql);  

            // Now you can extract all the records 
            // to see the remaining records 
            sql = "SELECT id, first, last, age FROM Registration"; 
            ResultSet rs = stmt.executeQuery(sql);  

            while(rs.next()){ 
                // Retrieve by column name 
                int id  = rs.getInt("id"); 
                int age = rs.getInt("age"); 
                String first = rs.getString("first"); 
                String last = rs.getString("last");  

                // Display values 
                System.out.print("ID: " + id); 
                System.out.print(", Age: " + age); 
                System.out.print(", First: " + first); 
                System.out.println(", Last: " + last); 
            } 
            rs.close(); 
        } catch(SQLException se) { 
            // Handle errors for JDBC 
            se.printStackTrace();  
        } catch(Exception e) { 
            // Handle errors for Class.forName 
            e.printStackTrace(); 
        } finally { 
            // finally block used to close resources 
            try { 
                if(stmt!=null) stmt.close(); 
            } catch(SQLException se2) { 
            } // nothing we can do 
            try { 
                if(conn!=null) conn.close(); 
            } catch(SQLException se) { 
                se.printStackTrace(); 
            } // end finally try
        } // end try 
        System.out.println("Goodbye!"); 
    } 
}
```

将上述程序保存到H2jdbcDeleteDemo.java中.通过在命令提示符中执行以下命令来编译并执行上述程序。

```bash
>javac H2jdbcDeleteDemo.java 
>java H2jdbcDeleteDemo
```

上述命令产生以下输出。

```bash
Connecting to a selected database... 
Connected database successfully... 
ID: 100, Age: 30, First: Zara, Last: Ali 
ID: 102, Age: 30, First: Zaid, Last: Khan 
ID: 103, Age: 28, First: Sumit, Last: Mittal 
Goodbye!
```



# H2 数据库 - 进阶

## 1、数据导入导出

1、数据导出

```sql
CALL CSVWRITE('D:\Hadoop\H2\hello_world.csv', 'SELECT * FROM hello_world');
```

2、数据导入 

```sql
CALL CSVREAD('D:\Hadoop\H2\test.csv','id|name', 'charset=UTF-8 fieldSeparator=|');
CREATE TABLE hello_world  AS SELECT * FROM CSVREAD('D:\Hadoop\H2\hello_world.csv');
```

3、数据插入

```sql
INSERT INTO hello_world SELECT * FROM CSVREAD('D:\Hadoop\H2\hello_world.csv');
```



## 2、中文乱码解决

H2数据库导出CSV数据默认字符编码虽然是UTF-8，但是分隔符是逗号。在导入的时候会中文乱码，解决办法是导出和导入时指定分隔符为 | ，字符编码为UTF-8。

### 1、H2数据导入导出乱码处理

1、新建测试表

```sql
DROP TABLE IF EXISTS TEST;
CREATE TABLE TEST(ID INT PRIMARY KEY,NAME VARCHAR(255));
INSERT INTO TEST VALUES(1, 'Hello');
INSERT INTO TEST VALUES(2, 'World');
INSERT INTO TEST VALUES(3, '乔丹');
INSERT INTO TEST VALUES(4, '科比');
INSERT INTO TEST VALUES(5, '艾佛森');
INSERT INTO TEST VALUES(6, '麦迪');
```

```sql
SELECT * FROM TEST;

ID | NAME
1  | Hello
2  | World
3  | 乔丹
4  | 科比
5  | 艾佛森
6  | 麦迪
```

2、指定编码导出表数据

```sql
CALL CSVWRITE('D:\test.csv', 'SELECT * FROM TEST', 'charset=UTF-8 fieldSeparator=|');
```

| CSVWRITE('D:\test.csv', 'SELECT * FROM TEST', 'charset=UTF-8 fieldSeparator=\|') |
| :----------------------------------------------------------- |
| 6                                                            |

3、删除测试表重新导入，测试读取文件是否会发生乱码。注意：这里只是读取csv数据到控制台，并未真正导入。

```sql
DROP TABLE test;
CALL CSVREAD('D:\test.csv','id|name', 'charset=UTF-8 fieldSeparator=|');

ID | NAME
1  | Hello
2  | World
3  | 乔丹
4  | 科比
5  | 艾佛森
6  | 麦迪
```

4、重新指定字符编码和文件分隔符导入表，

```sql
CREATE TABLE test AS SELECT * FROM CSVREAD('D:\test.csv','id|name', 'charset=UTF-8 fieldSeparator=|');
```

5、验证查询

```sql
SELECT * FROM TEST;

ID | NAME
1  | Hello
2  | World
3  | 乔丹
4  | 科比
5  | 艾佛森
6  | 麦迪
```



### 2、H2函数处理中文乱码

使用H2自带的UTF-8函数处理

```sql
CALL UTF8TOSTRING(STRINGTOUTF8('中文乱码测试！'))
```



## 3、空间索引表的使用

### 1、创建空间索引表

```sql
CREATE TABLE GEO_TABLE(GID SERIAL, THE_GEOM GEOMETRY);
INSERT INTO GEO_TABLE(THE_GEOM) VALUES
 ('POINT(500 505)'),
 ('LINESTRING(550 551, 525 512, 565 566)'),
 ('POLYGON ((550 521, 580 540, 570 564, 512 566, 550 521))');
CREATE SPATIAL INDEX GEO_TABLE_SPATIAL_INDEX ON GEO_TABLE(THE_GEOM);
```

```
- GEO_TABLE
  + GID
  + THE_GEOM
  + Indexes
  - GEO_TABLE_SPATIAL_INDEX
      Non unique
      THE_GEOM
```



### 2、几何空间相交查询

```sql
SET @POLY='POLYGON ((490 490, 536 490, 536 515, 490 515, 490 490))';
SELECT * FROM GEO_TABLE WHERE THE_GEOM && @POLY;
```

执行结果如下：

```sql
SET @POLY='POLYGON ((490 490, 536 490, 536 515, 490 515, 490 490))';
Update count: 0
(1 ms)

SELECT * FROM GEO_TABLE WHERE THE_GEOM && @POLY;
```

| GID  | THE_GEOM                               |
| :--- | :------------------------------------- |
| 1    | POINT (500 505)                        |
| 2    | LINESTRING (550 551, 525 512, 565 566) |



## 4、H2GIS 空间扩展

### 1、H2GIS 下载

H2GIS 官网：http://www.h2gis.org/

H2GIS 文档：http://javadoc.orbisgis.org/

打开官方，点击 Download 下载即可。



### 2、H2GIS 安装运行

1、Linux

```bash
wget https://github.com/orbisgis/h2gis/releases/download/v1.5.0/h2gis-dist-1.5.0-bin.zip -O h2gis.zip
unzip h2gis.zip
cd h2gis-standalone
nohup java -cp h2gis-dist-1.5.0.jar org.h2.tools.Server -tcpAllowOthers -tcpPort 9101 -webAllowOthers -webPort 8081 2>&1 &
## java -jar h2gis-dist-1.5.0.jar
```

2、Windows

```bash
## 将压缩包解压，cmd打开进入解压目录
cd h2gis-standalone
java -cp h2gis-dist-2.0.0-SNAPSHOT.jar org.h2.tools.Console
```



### 3、H2GIS空间扩展

初始化之后就有了H2扩展

```sql
CREATE ALIAS IF NOT EXISTS H2GIS_SPATIAL FOR "org.h2gis.functions.factory.H2GISFunctions.load";
CALL H2GIS_SPATIAL();
```

| PUBLIC.H2GIS_SPATIAL() |
| :--------------------- |
| null                   |



## 5、H2 数据库备份

### 1、启动 h2gis

```bash
cd h2gis-standalone
java -cp h2gis-dist-2.0.0-SNAPSHOT.jar org.h2.tools.Server -tcpAllowOthers -tcpPort 9101 -webAllowOthers -webPort 8081
```

URL链接：jdbc:h2:tcp://127.0.0.1:9101/~/testdb



### 2、数据库备份

```bash
java -cp h2gis-standalone/bin/h2-2.1.210.jar org.h2.tools.Script -url jdbc:h2:tcp://127.0.0.1:8081/~/testdb -user sa -script h2gis-standalone/table/testdb.zip -options compression zip
```



### 3、H2GIS空间扩展

```bash
CREATE ALIAS IF NOT EXISTS H2GIS_SPATIAL FOR "org.h2gis.functions.factory.H2GISFunctions.load";
CALL H2GIS_SPATIAL();
```



### 4、删除H2GIS空间扩展的表

```sql
DROP TABLE SPATIAL_REF_SYS ;
DROP TABLE GEOMETRY_COLUMNS ;
```



### 5、注释压缩文件重新打包

讲zip压缩文件内容全部注释。



### 6、导入数据

```sql
java -cp /usr/local/hadoop/h2gis-standalone/bin/h2-2.1.210.jar org.h2.tools.RunScript -url jdbc:h2:tcp://127.0.0.1:9101/~/testdb -user sa -script /usr/local/hadoop/h2gis-standalone/table/testdb.zip -options compression zip
```



# H2 数据库 - 报错

## 1、关键字冲突：Error executing DDL

> H2数据库关键字报错 executing DDL：https://blog.csdn.net/qq_43041751/article/details/125198166

报错：H2数据库关键字导致 org.hibernate.tool.schema.spi.CommandAcceptanceException: Error executing DDL

h2加载时报错：

```sql
GenerationTarget encountered exception accepting command : Error executing DDL “create table *” via JDBC Statement
org.hibernate.tool.schema.spi.CommandAcceptanceException: Error executing DDL"create table *" via JDBC Statement
Caused by: org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement “create table *”; expected “identifier”; SQL 
statement:
```

原因：经排查创建表时如果列名或表名中出现了关键字。具体关键字可去h2官网查询：http://www.h2database.com/html/advanced.html
There is a list of keywords that can’t be used as identifiers (table names, column names and so on), unless they are quoted (surrounded with double quotes). The following tokens are keywords in H2:

```sql
Keyword H2 SQL Standard
2016 2011 2008 2003 1999 92
ALL + + + + + + +
AND + + + + + + +
ANY + + + + + + +
ARRAY + + + + + +
AS + + + + + + +
ASYMMETRIC + + + + + NR
AUTHORIZATION + + + + + + +
BETWEEN + + + + + NR +
BOTH CS + + + + + +
CASE + + + + + + +
CAST + + + + + + +
CHECK + + + + + + +
CONSTRAINT + + + + + + +
CROSS + + + + + + +
CURRENT_CATALOG + + + +
CURRENT_DATE + + + + + + +
CURRENT_PATH + + + + + +
CURRENT_ROLE + + + + + +
CURRENT_SCHEMA + + + +
CURRENT_TIME + + + + + + +
CURRENT_TIMESTAMP + + + + + + +
CURRENT_USER + + + + + + +
DAY + + + + + + +
DEFAULT + + + + + + +
DISTINCT + + + + + + +
ELSE + + + + + + +
END + + + + + + +
EXCEPT + + + + + + +
EXISTS + + + + + NR +
FALSE + + + + + + +
FETCH + + + + + + +
FILTER CS + + + +
FOR + + + + + + +
FOREIGN + + + + + + +
FROM + + + + + + +
FULL + + + + + + +
GROUP + + + + + + +
GROUPS CS + +
HAVING + + + + + + +
HOUR + + + + + + +
IF +
ILIKE CS
IN + + + + + + +
INNER + + + + + + +
INTERSECT + + + + + + +
INTERVAL + + + + + + +
IS + + + + + + +
JOIN + + + + + + +
KEY + NR NR NR NR + +
LEADING CS + + + + + +
LEFT + + + + + + +
LIKE + + + + + + +
LIMIT MS +
LOCALTIME + + + + + +
LOCALTIMESTAMP + + + + + +
MINUS MS
MINUTE + + + + + + +
MONTH + + + + + + +
NATURAL + + + + + + +
NOT + + + + + + +
NULL + + + + + + +
OFFSET + + + +
ON + + + + + + +
OR + + + + + + +
ORDER + + + + + + +
OVER CS + + + +
PARTITION CS + + + +
PRIMARY + + + + + + +
QUALIFY +
RANGE CS + + + +
REGEXP CS
RIGHT + + + + + + +
ROW + + + + + +
ROWNUM +
ROWS CS + + + + + +
SECOND + + + + + + +
SELECT + + + + + + +
SESSION_USER + + + + + +
SET + + + + + + +
SOME + + + + + + +
SYMMETRIC + + + + + NR
SYSTEM_USER + + + + + + +
TABLE + + + + + + +
TO + + + + + + +
TOP MS
CS
TRAILING CS + + + + + +
TRUE + + + + + + +
UESCAPE + + + + +
UNION + + + + + + +
UNIQUE + + + + + + +
UNKNOWN + + + + + + +
USER + + + + + + +
USING + + + + + + +
VALUE + + + + + + +
VALUES + + + + + + +
WHEN + + + + + + +
WHERE + + + + + + +
WINDOW + + + + +
WITH + + + + + + +
YEAR + + + + + + +
ROWID +
Mode-sensitive keywords (MS) are keywords only in some compatibility modes.
```

LIMIT is a keywords only in Regular, Legacy, DB2, HSQLDB, MariaDB, MySQL, and PostgreSQL compatibility modes. It is an identifier in Strict, Derby, MSSQLServer, and Oracle compatibility modes.
MINUS is a keyword only in Regular, Legacy, DB2, HSQLDB, and Oracle compatibility modes. It is an identifier in Strict, Derby, MSSQLServer, MariaDB, MySQL, and PostgreSQL compatibility modes.
TOP is a context-sensitive keyword (can be either keyword or identifier) only in Regular, Legacy, HSQLDB, and MSSQLServer compatibility modes. It is an identifier unconditionally in Strict, Derby, DB2, MariaDB, MySQL, Oracle, and PostgreSQL compatibility modes.
Context-sensitive keywords (CS) can be used as identifiers in some places, but cannot be used as identifiers in others. Normal keywords (+) are always treated as keywords.

Most keywords in H2 are also reserved (+) or non-reserved (NR) words in the SQL Standard. Newer versions of H2 may have more keywords than older ones. Reserved words from the SQL Standard are potential candidates for keywords in future versions.

There is a compatibility setting SET NON_KEYWORDS that can be used as a temporary workaround for applications that use keywords as unquoted identifiers.

解决办法：在h2连接串后添加NON_KEYWORDS=value,key,month（表中出现的关键字用逗号隔开）例：

```properties
spring.datasource.url=jdbc:h2:mem:baoxian;MODE=MySQL;LOCK_MODE=0;DB_CLOSE_ON_EXIT=FALSE;DATABASE_TO_UPPER=true;NON_KEYWORDS=USER,account,value,key,month
```



# 参考文献 & 鸣谢

1. vue5 H2数据库教程：http://www.vue5.com/h2_database/h2_database.html
2. it1352 H2数据库教程：https://www.it1352.com/OnLineTutorial/h2_database/index.html
3. 无涯教程 H2数据库教程：https://www.learnfk.com/h2/h2-tutorial.html
4. H2 入门指南：https://www.wenjiangs.com/doc/rilfkxazj
5. H2 数据库入门和基本使用：https://blog.csdn.net/qq262593421/article/details/108651119
6. Java嵌入式数据库之Hsqldb、Derby、H2：https://blog.csdn.net/sumengnan/article/details/105003172
