# 参考资料 & 鸣谢

1. https://blog.csdn.net/weixin_33913332/article/details/92007241（H2数据库连接 URL说明）
2. https://ld246.com/article/1493196785376（H2datebase 数据库连接 URL 说明）
3. https://www.cnblogs.com/V1haoge/p/9959855.html
4. https://www.cnblogs.com/jin-zhe/archive/2018/01/05/8203347.html
5. https://blog.csdn.net/sanpic/article/details/80816379
6. https://www.cnblogs.com/xuyatao/p/7080095.html
7. https://www.jianshu.com/p/3663ef3fc93d



# SpringBoot 整合 H2

一、背景：

```
1、H2数据库是一个开源的关系型数据库。H2是一个嵌入式数据库引擎，采用java语言编写，不受
平台的限制，同时支持网络版和嵌入式版本，有比较好的兼容性，支持相当标准的sql标准，支持集群。

2、提供JDBC、ODBC访问接口，提供了非常友好的基于web的数据库管理界面
```

H2是一个很小的嵌入式数据库引擎，它提供JDBC、ODBC访问接口，支持三种连接方式：

- 内嵌模式（通过JDBC进行本地连接，应用和数据库在同一个JVM中）
- 服务器模式（通过JDBC或ODBC或TCP/IP进行远程连接）
- 混合模式（同时支持本地和远程连接）

内嵌模式

内嵌模式下，应用和数据库同在一个*JVM*中，通过*JDBC*进行连接。内嵌模式是最快和最容易的连接模式。它的缺点是任何时候数据库只能在一台虚拟机（和加载类）。像所有的模式，持久数据库和内存数据库都被支持。没有打开连接数和打开数据库数量方面的限制。



服务器模式

使用服务器模式（有时被称为远程模式或是*C/S*模式）时，应用可以通过*JDBC*或*ODBC*打开一个远程的数据库。服务器可以启动在同一个虚拟机或是不同的虚拟机上，也可以启动在不同的计算机上。大量的应用可以同时连接到同一个数据库上。服务器模式相比内嵌模式性能慢一些，因为所有的数据都需要通过*TCP/IP*进行传输。像所有的模式一样，支持持久数据库和内存数据库。没有打开连接数和打开数据库数量方面的限制。

 

混合模式

混合模式是内嵌模式和服务器模式的组合。第一个应用通过内嵌模式与数据库建立连接，同时也作为一个服务器启动，于是另外的应用（运行在不同的进程或是虚拟机上）可以同时访问同样的数据。第一个应用的本地连接与嵌入式模式的连接性能一样的快，而远程连接有点慢。

服务器能通过应用来进行启动和停止（使用服务*API*），或是是自动的方式（自动混合模式）。当使用[**自动混合模式**](https://blog.csdn.net/admin/blogs/new#auto_mixed_mode)，所有客户端都需要使用同样的*URL*进行连接（不管它是一个本地还是一个远程连接



二、引入h2的依赖包

```xml
<!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

三、常用配置

```properties
#h2配置
#启用SQL语句的日志记录
spring.jpa.show-sql = true
#设置ddl模式
spring.jpa.hibernate.ddl-auto = update
##数据库连接设置
spring.datasource.driverClassName =org.h2.Driver
#spring.datasource.url = jdbc:h2:mem:dbc2m
#Windows当前用户路径
#spring.datasource.url = jdbc:h2:file:~/.h2/dbc2m;AUTO_SERVER=TRUE
#可执行程序的当前路径
spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
#指定的静态配置路径
#spring.datasource.url = jdbc:h2:file:D:/db/.h2/dbc2m;AUTO_SERVER=TRUE
spring.datasource.username = sa
spring.datasource.password =

##数据初始化设置
#进行该配置后，每次启动程序，程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。
spring.datasource.schema=classpath:db/schema.sql
#进行该配置后，每次启动程序，程序都会运行resources/db/data.sql文件，对数据库的数据操作。
spring.datasource.data=classpath:db/data.sql

##h2 web console设置
#表明使用的数据库平台是h2
spring.datasource.platform=h2
# 进行该配置后，h2 web consloe就可以在远程访问了。否则只能在本机访问。
spring.h2.console.settings.web-allow-others=true
#进行该配置，你就可以通过YOUR_URL/h2访问h2 web consloe。YOUR_URL是你程序的访问URl。
spring.h2.console.path=/h2
#进行该配置，程序开启时就会启动h2 web consloe。当然这是默认的，如果你不想在启动程序时启动h2 web consloe，那么就设置为false。
spring.h2.console.enabled=true
```



