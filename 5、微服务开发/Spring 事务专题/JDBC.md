



## 一、JDBC 快速认识

### 1、JDBC 产生背景

1、如何操作数据库？

- 使⽤客户端⼯具访问数据库，需要⼿⼯建⽴连接，输⼊⽤户名和密码登录，编写SQL语句，点击执⾏，查看操作结果（结果集或受影响⾏数）。

2、实际开发中，会采用客户端操作数据库吗？

- 在实际开发中，当⽤户的数据发⽣改变时，不可能通过客户端操作执⾏SQL语句，因为操作量过⼤，⽆法保证效率和正确性。

3、所以就产生了 JDBC，那么 JDBC 是什么呢？

- JDBC（Java Database Connectivity）Java连接数据库的规范（标准），可以使⽤Java语⾔连接数据库完成 CRUD（CREATE READ UPDATE DELETE）操作。



### 2、JDBC 核心思想

Java 中定义了访问数据库的接口，可以为多种关系型数据库提供统一的访问方式。由数据库厂商提供驱动实现类（Driver 数据库驱动）。

```
                           ============应用程序============
                                         ↕↕↕
                           ==============JDBC=============
                               ↕↕↕                ↕↕↕
                             MySQL驱动          Oracle驱动
                                ↕                  ↕
                              MySQL              Oracle
```



### 3、JDBC 操作步骤

Java程序使用`JDBC`接口访问关系数据库的时候，需要以下几步：

- 创建全局`DataSource`实例，表示数据库连接池；
- 在需要读写数据库的方法内部，按如下步骤访问数据库：
- 从全局`DataSource`实例获取`Connection`实例；
- 通过`Connection`实例创建`PreparedStatement`实例；
- 执行`SQL`语句，如果是查询，则通过`ResultSet`读取结果集，如果是修改，则获得`int`结果。

正确编写`JDBC`代码的关键是使用`try ... finally`释放资源，涉及到事务的代码需要正确提交或回滚事务。



### 4、JDBC 环境搭建

方式一：没有使用 Maven 构建项目的情况下

- 在项目下新建 lib 文件夹，用于存放 jar 文件。
  - 如果使用 mysql 数据库，将 mysql 驱动 mysql-connector-java-5.1.X 复制到项目的 lib 文件夹中。
  - 如果使用 h2 数据库，将 h2 驱动 h2-2.2.220.jar 复制到项目 lib 文件夹中。
- 选中 lib 文件夹右键 Add as Libraay，点击 OK。

方式二：使用 Maven 构建项目的情况下

- 导入 H2 数据库驱动的依赖

  ```xml
  <!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
  <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <version>2.2.220</version>
  </dependency>
  ```


上面我们使用的是 H2 数据库驱动，想用 MySQL 数据库驱动如下：

- mysql-connector-java-5.1.X  适用于 5.X 版本
- mysql-connector-java-8.0.X  适用于 8.X 版本



## 二、JDBC API 对象说明

JDBC 是由多个接口和类进行功能实现。

|          权限定名          | 类型      |                             描述                             |
| :------------------------: | --------- | :----------------------------------------------------------: |
|   java.sql.DriverManager   | class     |       管理多个数据库驱动类，提供了获取数据库连接的方法       |
|    java.sql.Connection     | interface | 代表一个数据库连接（当connection不是null时，表示已连接数据库） |
|     java.sql.Statement     | interface |                   发送SQL语句到数据库工具                    |
| java.sql.PreparedStatement | interface | SQL 语句被预编译并存储在此对象中，可以使用此对象多次高效地执行该语句 |
| java.sql.CallableStatement | interface |                    用于执行 SQL 存储过程                     |
|     java.sql.ResultSet     | interface |             保存SQL查询语句的结果数据（结果集）              |
| java.sql.DatabaseMetaData  | interface |            定义数据库的信息，名称、类型、版本等等            |
| java.sql.ParameterMetaData | interface | 定义预编译对象的信息，包含了预编译的SQL中，参数的个数、类型等等 |
| java.sql.ResultSetMetaData | interface | 定义结果集对象的信息，包含查询结果集里，列的个数、名称、类型等等 |
|   java.sql.SQLException    | class     |               处理数据库应用程序时所发生的异常               |



### 1、DriverManager 类

JDBC 程序中的 DriverManager 用于加载驱动，并创建与数据库的链接，这个API的常用方法：

| 方法                                                         | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| public static void registerDriver(Driver driver)             | 用于在DriverManager中注册给定的驱动程序                      |
| public static void deregisterDriver(Driver driver)           | 用于通过DriverManager注销给定的驱动程序（从列表中删除该驱动程序） |
| public static Connection getConnection(String url)           | 用于建立与指定URL的连接                                      |
| public static Connection getConnection(String url,String userName,String password) | 用于使用指定的URL，数据库用户名和密码建立连接                |
| public static Connection getConnection(String url, Properties prop) | 用于使用指定的URL，数据库用户名和密码建立连接                |

**注意：在实际开发中并不推荐采用registerDriver方法注册驱动**。原因有二：

1. 查看Driver的源代码可以看到，如果采用此种方式，会导致驱动程序注册两次，也就是在内存中会有两个Driver对象。

2. 程序依赖mysql的api，脱离mysql的jar包，程序将无法编译，将来程序切换底层数据库将会非常麻烦。

- 推荐方式：**Class.forName("com.mysql.jdbc.Driver");**

采用此种方式不会导致驱动对象在内存中重复出现，并且采用此种方式，程序仅仅只需要一个字符串，不需要依赖具体的驱动，使程序的灵活性更高。



### 2、Connection 接口

JDBC 程序中的 Connection 接口，它用于代表数据库的连接，Collection 是数据库编程中最重要的一个对象，客户端与数据库所有交互都是通过 Connection 对象完成的，这个对象的常用方法：

| 方法                                                         | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| public Statement createStatement()                           | 创建一个可用于执行SQL查询的Statement对象，无参               |
| public Statement createStatement(int resultSetType,int resultSetConcurrency) | 创建一个Statement对象，参数一：结果集滚动类型，参数二：是否为可以更新的结果集 |
| public PreparedStatement prepareStatement(String sql)        | 创建向数据库发送预编译sql的PrepareSatement对象。             |
| public CallableStatement prepareCall(String sql)             | 创建执行 SQL 存储过程的CallableStatement对象。               |
| public void setAutoCommit(boolean status)                    | 用于设置事务是否自动提交，默认为true                         |
| public void commit()                                         | 提交事务                                                     |
| public void rollback()                                       | 回滚事务                                                     |
| public void close()                                          | 关闭连接                                                     |



### 3、Statement 接口

JDBC 程序中的 Statement 对象用于向数据库发送SQL语句， Statement 对象常用方法：

| 方法                                      | 说明                                                         |
| :---------------------------------------- | :----------------------------------------------------------- |
| public boolean execute(String sql)        | 用于向数据库发送任意SQL语句（不管是查询还是更新）            |
| public ResultSet executeQuery(String sql) | 用于执行Select查询。它返回ResultSet的对象                    |
| public int executeUpdate(String sql)      | 用于执行更新类操作，它可以用于执行Insert，Update、Delete等SQL语句 |
| public void addBatch( String sql )        | 把多条SQL语句放到一个批处理中。                              |
| public int[] executeBatch()               | 向数据库发送一批SQL语句执行                                  |
| public void clearBatch()                  | 清空缓存的数据，实际上就是清空批处理中的SQL语句。            |



### 4、ResultSet 接口

JDBC 程序中的 ResultSet 用于代表SQL语句的执行结果。Resultset 封装执行结果时，采用的类似于表格的方式。ResultSet 对象维护了一个指向表格数据行的游标，初始的时候，游标在第一行之前，调用 ResultSet.next() 方法，可以使游标指向具体的数据行，进行调用方法获取该行的数据。

ResultSet 既然用于封装执行结果的，所以该对象提供的都是用于获取数据的get方法：

| 方法                                       | 说明                                                         |
| :----------------------------------------- | :----------------------------------------------------------- |
| public boolean next()                      | 用于将光标从当前位置移到下一行。                             |
| public boolean previous()                  | 用于将光标从当前位置移到上一行。                             |
| public boolean absolute(int row)           | 用于将光标移动到ResultSet对象中的指定行号。                  |
| public boolean relative(int row)           | 用于将光标移动到ResultSet对象中的相对行号，它可以是正数或负数。 |
| public boolean first()                     | 用于将光标移动到结果集对象的第一行。                         |
| public boolean last()                      | 用于将光标移动到结果集对象的最后一行。                       |
| public void beforeFirst()                  | 用于将光标移动到结果集对象的前面，就在第一行之前。           |
| public void afterLast()                    | 用于将光标移动到结果集对象的末尾，在最后一行之后。           |
| public boolean isFirst()                   | 检索光标是否位于此结果集对象的第一行。                       |
| public boolean isLast()                    | 检索光标是否在此结果集对象的最后一行。                       |
| public boolean isBeforeFirst()             | 检索光标是否在此结果集对象的第一行之前。                     |
| public boolean isAfterLast()               | 检索光标是否在此结果集对象的最后一行之后。                   |
| public String getObject(int columnIndex)   | 用于根据列索引查询该列数据。（Object类型）                   |
| public String getObject(String columnName) | 用于根据列名称查询该列数据。（Object类型）                   |
| public int getInt(int columnIndex)         | 用于根据列索引查询该列数据。（int类型）                      |
| public int getInt(String columnName)       | 用于根据列名称查询该列数据。（int类型）                      |
| public String getString(int columnIndex)   | 用于根据列索引查询该列数据。（String类型）                   |
| public String getString(String columnName) | 用于根据列名称查询该列数据。（String类型）                   |



### 5、释放资源

JDBC 程序运行完后，切记要释放程序在运行过程中，创建的那些与数据库进行交互的对象，这些对象通常是ResultSet, Statement和Connection对象，特别是Connection对象，它是非常稀有的资源，用完后必须马上释放，如果Connection不能及时、正确的关闭，极易导致系统宕机。Connection的使用原则是尽量晚创建，尽量早的释放。

为确保资源释放代码能运行，资源释放代码也一定要放在finally语句中。



## 三、JDBC 开发步骤【重点】

### 1、注册驱动

使用 Class.forName("org.h2.Driver");手动加载字节码文件到 JVM 中。

```java
Class.forName("org.h2.Driver"); // H2加载驱动
```

其他常用数据库驱动如下：

| RDBMS      | JDBC驱动程序名称                | URL格式                                            |
| ---------- | ------------------------------- | -------------------------------------------------- |
| MySQL      | com.mysql.jdbc.Driver           | jdbc:mysql://hostname/databaseName                 |
| Postgresql | org.postgresql.Driver           | jdbc:postgresql://hostname:port/dbname             |
| Oracle     | oracle.jdbc.driver.OracleDriver | jdbc:oracle:thin:@hostname:portNumber:databaseName |
| H2Database | org.h2.Driver                   | jdbc:h2:dbType:dbname                              |
| DB2        | com.ibm.db2.jdbc.net.DB2Driver  | jdbc:db2:hostname:port Number/databaseName         |
|            |                                 |                                                    |







### 2、连接数据库

1、通过 DriverManager.getConnection(url, user, password) 获取数据库连接对象

```java
Connection conn = DriverManager.getConnection("jdbc:h2:mem:test_mem", "h2", "1234");
```

2、URL（Uniform Resource Locator） 统一资源定位符：由协议、IP、端口、SID（程序实例名称）组成。

```
jdbc : mysql:[] //localhost:3306/ test ?参数名:参数值
 ⬇       ⬇ ⬇ ⬇          ⬇⬇⬇⬇        ⬇ 
协议     子协议       主机号:端口   数据库
```

3、常用数据库URL地址的写法：

- Oracle：`jdbc:oracle:thin:@localhost:1521:sid`
- Postgresql：`jdbc:postgresql://localhost/test`
- MySql：`jdbc:mysql://localhost:3306/sid`

如果连接的是本地的Mysql数据库，并且连接使用的端口是3306，那么的url地址可以简写为：`jdbc:mysql:///数据库`



### 3、获取发送SQL的对象

1、通过Connection对象获得Statement对象，用于对数据库进行通用访问。

```java
Statement statement = Conn.createStatement();
```

2、也可以通过 Connection 对象获得 PreparedStatement 对象，PreparedStatement 会对SQL预编译，会更安全，效率会更高。

```java
PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM USER_INFO WHERE USER_ID = ?")
```

3、简单介绍PreparedStatement 和 Statement 的区别：

- PreparedStatement在使用时只需要编译一次，就可以运行多次，Statement每运行一次就编译一次，所以PreparedStatement的效率更高

- PreparedStatement需要的sql语句为用?(占位符)来替换值，Statement所需要的sql语句为字符串拼接

- PreparedStatement解决了sql注入的问题，Statement没有解决，因为PreparedStatement有一个预编译的过程，就算传入占位符的数据中有sql关键字也都被认为是值。Statement所需要的是字符串拼接，传入的整个字符串被默认为sql语句，如果用户手动拼接了字符串，那么会导致语句的改变

- 从区别中看出，preparedStatement 的效率会更高点，就使用它了。



### 4、执行SQL语句

执行SQL语句并接收执行结果

```java
// 执行SQL语句并接收结果
boolean result1 = statement.execute("DROP TABLE IF EXISTS USER_INFO");
boolean result2 = statement.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
int result3 = statement.executeUpdate("INSERT INTO USER_INFO VALUES(1, 'Hello')");
```

- 注意：在编写DML语句时，一定要注意字符串参数的符号是单引号 '值' ；
- DML语句：增删改时，返回受影响行数（int 类型）。
- DQL语句：查询时，返回结果数据（ResultSet 结果集）。



### 5、处理结果

> 接受处理操作结果。

```java
if (result == 1) {
    System.out.println("Success");
}
```

- 受影响行数：逻辑判断、方法返回。
- 查询结果集：迭代、依次获取。结果集 ResultSet 下章节讲解。



### 6、释放资源

> 遵循先开后关原则，释放所使用到资源对象。

```java
statement.close();
conn.close();
```



### 7、综合案例

> 整合以上核心六步，实现向数据库表中插入一条数据。

```java
package org.example;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;

public class JdbcInsert {
    public static void main(String[] args) throws Exception {
        // 1.加载驱动
        Class.forName("org.h2.Driver");
        // 2.获取连接对象
        Connection connection = DriverManager.getConnection("jdbc:h2:mem:test_mem", "h2", "");
        // 3.获得执行SQL的对象
        Statement statement = connection.createStatement();
        // 4.执行SQL语句，并接收结果
        boolean result1 = statement.execute("DROP TABLE IF EXISTS USER_INFO");
        boolean result2 = statement.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
        int result3 = statement.executeUpdate("INSERT INTO USER_INFO VALUES(1, 'Hello')");
        // 5.处理结果
        if (result3 == 1) {
            System.out.println("插入数据成功！");
        } else {
            System.out.println("插入数据失败！");
        }
        // 6.释放资源
        statement.close();
        connection.close();
    }
}
```





## 四、ResultSet【结果集】

> 在执行查询SQL后，存放查询到的结果集数据。

### 1、接收结果集

> ResultSet rs = statement.executeQuery(sql);

```java
ResultSet rs = statement.executeQuery("SELECT * FROM USER_INFO;");
```



### 2、遍历 ResultSet 中的数据

> ResultSet 以表（table）结构进行临时结果的存储，需要通过 JDBC API 将其中数据进行依次获取。
>
> - 数据行指针：初始位置在第一行数据前，每调用一次 boolean next() 方法 ResultSet 的指针向下移动一行，结果为 true，表示当前行有数据。
> - rs.getXxx(整数);代表根据列的编号顺序获得，从 1 开始。
> - rs.getXxx("列名");代表根据列名获得。

判断是否还有数据方法如下：

```java
boolean next() throws SQLException // 判断rs结果集中下一行是否存在数据。
```

遍历方法如下：

```java
int getInt(int columnIndex) throws SQLException          // 获取当前行第N列的int值
int getInt(String columnLabel) throws SQLException       // 获取当前行columnLabel列的int值
double getDouble(int columnIndex) throws SQLException    // 获取当前行第N列的double值
double getDouble(String columnLabel) throws SQLException // 获取当前行columnLabel列的double值
String getString(int columnIndex) throws SQLException    // 获取当前行第N列的String值
String getString(String columnLabel) throws SQLException // 获取当前行columnLabel列的String值
...
```

> - 注意：列的编号从1开始。



### 3、遍历 ResultSet  综合案例

> 对USER_INFO表中的所有数据进行遍历



#### 1、根据列的名称获取

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class JdbcQuery {
    public static void main(String[] args) throws Exception {
        // 1.加载驱动
        Class.forName("org.h2.Driver");
        // 2.获取连接对象
        Connection connection = DriverManager.getConnection("jdbc:h2:mem:test_mem", "h2", "");
        // 3.获得执行SQL的对象
        Statement statement = connection.createStatement();
        // 4.执行SQL语句，并接收结果
        statement.execute("DROP TABLE IF EXISTS USER_INFO");
        statement.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
        statement.executeUpdate("INSERT INTO USER_INFO VALUES(1, 'Hello')");
        statement.executeUpdate("INSERT INTO USER_INFO VALUES(2, 'World')");
        statement.executeUpdate("INSERT INTO USER_INFO VALUES(3, '乔丹')");
        statement.executeUpdate("INSERT INTO USER_INFO VALUES(4, '科比')");
        statement.executeUpdate("INSERT INTO USER_INFO VALUES(5, '艾佛森')");
        statement.executeUpdate("INSERT INTO USER_INFO VALUES(6, '麦迪')");

        ResultSet resultSet = statement.executeQuery("SELECT * FROM USER_INFO");

        // 5.处理结果集
        while (resultSet.next()) {
            // 5.1有数据，依据列名获取数据
            String user_id = resultSet.getString("USER_ID");
            String user_name = resultSet.getString("USER_NAME");
            System.out.println(user_id + ": " + user_name);
        }

        // 6.释放资源
        resultSet.close();
        statement.close();
        connection.close();
    }
}
```

```
1: Hello
2: World
3: 乔丹
4: 科比
5: 艾佛森
6: 麦迪
```



#### 2、根据列的编号获取数据

```java
// ....与上无异
// 5.处理结果集
while (resultSet.next()) {
    // 5.1有数据，依据列名获取数据
    String user_id = resultSet.getString(1);
    String user_name = resultSet.getString(2);
    System.out.println(user_id + ": " + user_name);
}
// 释放资源
```

```
1: Hello
2: World
3: 乔丹
4: 科比
5: 艾佛森
6: 麦迪
```



## 五、JDBC 封装工具类

- 在实际JDBC的使用中，存在着大量的重复代码：例如连接数据库，关闭数据库等操作！
- 我们需要把传统的JDBC代码进行重构，抽取出通用的JDBC工具类，以后连接任何数据库，释放资源都可以使用这个工具类。

### 1、重用性方案

封装获取连接、释放资源两个方法。

- 提供public static Connection getConnection(){}方法。
- 提供public static void release(Connection conn,Statement sm,ResultSet rs){}方法。

```java
import java.sql.*;

public class JdbcUtils {
    private static String driver = "org.h2.Driver";
    private static String url = "jdbc:h2:mem:test_mem";
    private static String username = "root";
    private static String password = "passsword";

    static {
        try {
            // 加载数据库驱动
            Class.forName(driver);
        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    // 获取数据库连接对象
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, username, password);
    }

    /**
     * 释放资源，要释放的资源包括Connection数据库连接对象，
     * 负责执行SQL命令的Statement对象，存储查询结果的ResultSet对象
     */
    public static void release(Connection connection, Statement statement, ResultSet resultSet) {
        // 先开后关
        try {
            if (resultSet != null) {
                resultSet.close();
            }
            if (statement != null) {
                statement.close();
            }
            if (connection != null) {
                connection.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```



### 2、跨平台工具类实现

1、在src目录下新建db.properties文件。

```properties
driver=org.h2.Driver
url=jdbc:h2:mem:test_mem
username=h2
password=
```

2、工具类的封装

```java
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class JdbcUtils {
    private static String url = null;
    private static String username = null;
    private static String password = null;

    static {
        try {
            // 读取db.properties文件中的数据库连接信息
            InputStream in = JdbcUtils.class.getClassLoader().getResourceAsStream("db.properties");
            Properties prop = new Properties();
            prop.load(in);
            // 获取数据库连接驱动
            String driver = prop.getProperty("driver");
            // 获取数据库连接URL地址
            url = prop.getProperty("url");
            // 获取数据库连接用户名
            username = prop.getProperty("username");
            // 获取数据库连接密码
            password = prop.getProperty("password");
            // 加载数据库驱动
            Class.forName(driver);
        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    // 获取数据库连接对象
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, username, password);
    }

    /**
     * 释放资源，要释放的资源包括Connection数据库连接对象，
     * 负责执行SQL命令的Statement对象，存储查询结果的ResultSet对象
     */
    public static void release(Connection connection, Statement statement, ResultSet resultSet) {
        // 先开后关
        try {
            if (resultSet != null) {
                resultSet.close();
            }
            if (statement != null) {
                statement.close();
            }
            if (connection != null) {
                connection.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```





## 六、Statement【完整案例】

使用 Statement 对象建表、插入数据、删除数据、更新数据及查询数据

### 1、插入数据

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

public class JDBCTest {

    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            // 获取数据库连接
            conn = JdbcUtils.getConnection();
            // 通过conn对象获取负责执行SQL命令的Statement对象
            st = conn.createStatement();
            // 先创建表, 然后插入数据
            st.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
            // 执行插入操作，executeUpdate方法返回成功的条数
            int num = st.executeUpdate("INSERT INTO USER_INFO VALUES(1, 'Hello')");
            if (num > 0) {
                System.out.println("插入成功！！");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // SQL执行完成之后释放相关资源
            JdbcUtils.release(conn, st, rs);
        }
    }
}
```

### 2、删除数据

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

public class JDBCTest {

    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            // 获取数据库连接
            conn = JdbcUtils.getConnection();
            // 通过conn对象获取负责执行SQL命令的Statement对象
            st = conn.createStatement();
            // 先创建表, 然后插入数据
            st.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
            // 执行插入操作，executeUpdate方法返回成功的条数
            int num = st.executeUpdate("INSERT INTO USER_INFO VALUES(1, 'Hello')");
            if (num > 0) {
                System.out.println("插入成功！！");
            }
            // 执行删除操作，executeUpdate方法返回成功的条数
            num = st.executeUpdate("DELETE FROM USER_INFO WHERE USER_ID = 1");
            if (num > 0) {
                System.out.println("删除成功！！");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // SQL执行完成之后释放相关资源
            JdbcUtils.release(conn, st, rs);
        }
    }
}
```

```
插入成功！！
删除成功！！
```



### 3、更新数据

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

public class JDBCTest {

    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            // 获取数据库连接
            conn = JdbcUtils.getConnection();
            // 通过conn对象获取负责执行SQL命令的Statement对象
            st = conn.createStatement();
            // 先创建表, 然后插入数据
            st.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
            // 执行插入操作，executeUpdate方法返回成功的条数
            int num = st.executeUpdate("INSERT INTO USER_INFO VALUES(1, 'Hello')");
            if (num > 0) {
                System.out.println("插入成功！！");
            }
            // 执行更新操作，executeUpdate方法返回成功的条数
            num = st.executeUpdate("UPDATE USER_INFO SET USER_NAME = 'World' WHERE USER_ID = 1");
            if (num > 0) {
                System.out.println("更新成功！！");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // SQL执行完成之后释放相关资源
            JdbcUtils.release(conn, st, rs);
        }
    }
}
```

```
插入成功！！
更新成功！！
```



### 4、查询数据

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

public class JDBCTest {

    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            // 获取数据库连接
            conn = JdbcUtils.getConnection();
            // 通过conn对象获取负责执行SQL命令的Statement对象
            st = conn.createStatement();
            // 先创建表, 然后插入数据
            st.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
            // 执行插入操作，executeUpdate方法返回成功的条数
            int num = st.executeUpdate("INSERT INTO USER_INFO VALUES(1, 'Hello')");
            if (num > 0) {
                System.out.println("插入成功！！");
            }
            rs = st.executeQuery("SELECT * FROM USER_INFO");
            System.out.println("查询成功！！");
            while (rs.next()) {
                System.out.println(rs.getString(1) + ": " + rs.getString(2));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // SQL执行完成之后释放相关资源
            JdbcUtils.release(conn, st, rs);
        }
    }
}
```

```
插入成功！！
查询成功！！
1: Hello
```





## 七、PreparedStatement【重点】

### 1、Statement SQL 注入问题

> 什么是 SQL 注入

用户输入的数据中有 SQL 关键字或语法并且参与了 SQL 语句的编译，导致 SQL 语句编译后的条件含义为 true，一直得到正确的结果。这种现象称为 SQL 注入。

需求：实现登录功能，需要通过传入的登录名和密码查询数据库表中的用户是否存在。

1、如果使用的是Statement语句来执行sql语句。

```java
String sql = "select * from account where username='" + loginName + "' and password='" + loginPwd + "'";
```

2、如果应用中传入：

```bash
> 请输入用户名：
test1
> 请输入密码：
abc' or '1'='1
username = "test1"
password ="abc' or '1'='1"
```

3、那么当上面的sql拼接变量值之后的sql语句为

```sql
select * from account where username = 'test1 and password = 'abc' or '1' = '1';
```

4、那么将登录成功。【这样非常危险，错误的密码也被成功登录】

> 如何避免SQL注入

5、由于编写的 SQL 语句是在用户输入数据，整合后再进行编译。所以为了避免 SQL 注入的问题，我们要使 SQL 语句在用户输入数据前就已进行编译成完整的 SQL 语句，再进行填充数据。所以就出现了 PreparedStatement 接口，它是 Statement 的子接口，可以理解为加强版的 PreparedStatement。PreparedStatement 继承了 Statement 接口，执行 SQL 语句的方法无异。





### 2、PreparedStatement 优点

- 预编译SQL 语句，效率高。

  PS：什么是预编译？预编译语句PreparedStatement是java.sql中的一个接口，它是Statement的子接口。通过Statement对象执行sql语句时，需要将sql语句发送给DBMS,由DBMS首先进行编译再执行（在创建通道的时候并不进行sql的编译工作，事实上也无法进行编译）。而通过PreparedStatement不同，在创建PreparedStatement对象时就指定了sql语句，该语句立即发送给DBMS进行编译，当该语句被执行时，DBMS直接运行编译后的sql语句，而不需要像其他sql语句那样首先将被编译。一般在考虑反复使用一个sql语句时才使用预编译，预编译语句常常放在一个循环中使用（在这种情况下预编译的优势就很明显了），通过反复设置参数从而达到多次使用该语句；

- 安全，避免SQL注入。

- 可以动态的填充数据，执行多个同构的 SQL 语句。



### 3、PreparedStatement 应用

1、参数标记

```java
// 1.预编译 SQL 语句
PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM USER_INFO WHERE USER_ID=? and USER_NAME=?");
```

- 注意：JDBC中的所有参数都由？符号占位，这被称为参数标记。在执行SQL语句之前，必须为每个参数提供值。
- 简化SQL语句的编写，提高SQL语句执行效率，提高安全性。

2、动态参数绑定：pstmt.setXxx(下标，值); 参数下标从1开始，为指定参数下标绑定值。

```java
// 1.预编译SQL语句
PreparedStatement pstmt = conn.preparedStatement("SELECT * FROM USER_INFO WHERE USER_ID=? and USER_NAME=?");
// 2.为参数下标赋值
pstmt.setInt(1, userId);
pstmt.setString(2, userName);
```

3、PreparedStatement 完整案例

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.List;

public class JdbcQuery {
    public static void main(String[] args) throws Exception {
        // 1.加载驱动
        Class.forName("org.h2.Driver");
        // 2.获取连接对象
        Connection connection = DriverManager.getConnection("jdbc:h2:mem:test_mem", "h2", "");
        // 3.获得执行SQL的对象PreparedStatement, 创建表和插入数据
        connection.prepareStatement("DROP TABLE IF EXISTS USER_INFO").execute();
        connection.prepareStatement("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))").execute();
        PreparedStatement pstmtInsert = connection.prepareStatement("INSERT INTO USER_INFO VALUES(?, ?)");
        List<String> list = List.of("Hello", "World", "乔丹", "科比", "艾佛森", "麦迪");
        for (int i = 0; i < list.size(); i++) {
            pstmtInsert.setInt(1, i + 1);
            pstmtInsert.setString(2, list.get(i));
            pstmtInsert.addBatch();
        }
        int[] ints = pstmtInsert.executeBatch();
        if (ints.length == 6) {
            System.out.println("数据插入成功");
        }

        // 4、获取 PreparedStatement 对象, 预编译SQL执行查询
        String selectSql = "SELECT * FROM USER_INFO WHERE USER_ID = ? and USER_NAME = ?";
        PreparedStatement pstmtSelect = connection.prepareStatement("SELECT * FROM USER_INFO WHERE USER_ID = ? and USER_NAME = ?");
        pstmtSelect.setInt(1, 1);
        pstmtSelect.setString(2, "Hello");
        ResultSet resultSet = pstmtSelect.executeQuery();

        // 5.处理结果集
        while (resultSet.next()) {
            // 6.1有数据，依据列名获取数据
            String user_id = resultSet.getString(1);
            String user_name = resultSet.getString(2);
            System.out.println(user_id + ": " + user_name);
        }

        // 6.释放资源
        resultSet.close();
        pstmtSelect.close();
        connection.close();
    }
}
```

```
数据插入成功
1: Hello
```



## 八、JDBC 控制事务

### 1、数据库事务概述

> 概念：事务指逻辑上的一组操作，组成这组操作的各个单元，要不全部成功，要不全部不成功。 
>

ACID 原则：

- 原子性（Atomic）：整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚（ROLLBACK）到事务开始前的状态，就像这个事务从来没有执行过一样
- 一致性（Consist）：一个事务可以封装状态改变（除非它是一个只读的）。事务必须始终保持系统处于一致的状态，不管在任何给定的时间并发事务有多少。也就是说：如果事务是并发多个，系统也必须如同串行事务一样操作。其主要特征是保护性和不变性(Preserving an Invariant)，以转账案例为例，假设有五个账户，每个账户余额是100元，那么五个账户总额是500元，如果在这个5个账户之间同时发生多个转账，无论并发多少个，比如在A与B账户之间转账5元，在C与D账户之间转账10元，在B与E之间转账15元，五个账户总额也应该还是500元，这就是保护性和不变性。
- 隔离性（Isolated）：隔离状态执行事务，使它们好像是系统在给定时间内执行的唯一操作。如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，使得在同一时间仅有一个请求用于同一数据。
- 持久性（Durable）：在事务完成以后，该事务对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

隔离性问题：

1. 脏读：脏读指一个事务读取了另外一个事务未提交的数据。
2. 不可重复读：不可重复读指在一个事务内读取表中的某一行数据，多次读取结果不同。
3. 虚读(幻读) : 虚读(幻读)是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。



### 2、JDBC 编程事务

> 1. 开启事务：setAutoCommit(boolean autoCommit):调用该方法设置参数为false，即开启事务(在执行sql之前开启事务)
> 2. 提交事务：commit()。(当所有sql都执行完提交事务)
> 3. 回滚事务:rollback()。在catch中回滚事务

1、模拟转账成功时的业务场景

```java
package org.example;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

// 模拟转账成功时的业务场景
public class TestTransaction1 {

    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement st = null;
        ResultSet rs = null;
        try {
            conn = JdbcUtils.getConnection();
            // 准备测试数据, 所以放在事务开始前
            conn.prepareStatement("CREATE TABLE account(id INT PRIMARY KEY,NAME VARCHAR(40),money FLOAT)").execute();
            conn.prepareStatement("insert into account values(1, 'A',1000)").execute();
            conn.prepareStatement("insert into account values(2, 'B',1000)").execute();
            conn.prepareStatement("insert into account values(3, 'C',1000)").execute();
            // 通知数据库开启事务(start transaction)
            conn.setAutoCommit(false);
            String sql1 = "update account set money = money-100 where name='A'";
            st = conn.prepareStatement(sql1);
            st.executeUpdate();
            String sql2 = "update account set money = money+100 where name='B'";
            st = conn.prepareStatement(sql2);
            st.executeUpdate();
            // 上面的两条SQL执行Update语句成功之后就通知数据库提交事务(commit)
            conn.commit();
            System.out.println("成功！！！");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(conn, st, rs);
        }
    }
}
```

2、模拟转账过程中出现异常导致有一部分SQL执行失败后让数据库自动回滚事务

```java
package org.example;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

// 模拟转账过程中出现异常导致有一部分SQL执行失败后让数据库自动回滚事务
public class TestTransaction2 {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement st = null;
        ResultSet rs = null;
        try {
            conn = JdbcUtils.getConnection();
            // 准备测试数据, 所以放在事务开始前
            conn.prepareStatement("CREATE TABLE account(id INT PRIMARY KEY,NAME VARCHAR(40),money FLOAT)").execute();
            conn.prepareStatement("insert into account values(1, 'A',1000)").execute();
            conn.prepareStatement("insert into account values(2, 'B',1000)").execute();
            conn.prepareStatement("insert into account values(3, 'C',1000)").execute();
            // 通知数据库开启事务(start transaction)
            conn.setAutoCommit(false);
            String sql1 = "update account set money=money-100 where name='A'";
            st = conn.prepareStatement(sql1);
            st.executeUpdate();
            // 用这句代码模拟执行完SQL1之后程序出现了异常而导致后面的SQL无法正常执行，事务也无法正常提交，此时数据库会自动执行回滚操作 
            int x = 1 / 0;
            String sql2 = "update account set money=money+100 where name='B'";
            st = conn.prepareStatement(sql2);
            st.executeUpdate();
            // 上面的两条SQL执行Update语句成功之后就通知数据库提交事务(commit)
            conn.commit();
            System.out.println("成功！！！");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(conn, st, rs);
        }
    }
}
```

3、模拟转账过程中出现异常导致有一部分SQL执行失败时手动通知数据库回滚事务

```java
package org.example;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

// 模拟转账过程中出现异常导致有一部分SQL执行失败时手动通知数据库回滚事务
public class TestTransaction3 {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement st = null;
        ResultSet rs = null;
        try {
            conn = JdbcUtils.getConnection();
            // 准备测试数据, 所以放在事务开始前
            conn.prepareStatement("CREATE TABLE account(id INT PRIMARY KEY,NAME VARCHAR(40),money FLOAT)").execute();
            conn.prepareStatement("insert into account values(1, 'A',1000)").execute();
            conn.prepareStatement("insert into account values(2, 'B',1000)").execute();
            conn.prepareStatement("insert into account values(3, 'C',1000)").execute();
            // 通知数据库开启事务(start transaction)
            conn.setAutoCommit(false);
            String sql1 = "update account set money=money-100 where name='A'";
            st = conn.prepareStatement(sql1);
            st.executeUpdate();
            // 用这句代码模拟执行完SQL1之后程序出现了异常而导致后面的SQL无法正常执行，事务也无法正常提交
            int x = 1 / 0;
            String sql2 = "update account set money=money+100 where name='B'";
            st = conn.prepareStatement(sql2);
            st.executeUpdate();
            // 上面的两条SQL执行Update语句成功之后就通知数据库提交事务(commit)
            conn.commit();
            System.out.println("成功！！！");
        } catch (Exception e) {
            try {
                // 捕获到异常之后手动通知数据库执行回滚事务的操作
                conn.rollback();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            e.printStackTrace();
        } finally {
            JdbcUtils.release(conn, st, rs);
        }
    }
}
```



## 九、获取自增长键值、批处理

### 1、获取自增长键值

当我们通过JDBC往数据库的表格中添加一条记录，其中有一个字段是自增的，那么在JDBC这边怎么在添加之后直接获取到这个自增的值。JDBC中默认不允许获取自增的主键，需要在connection.prepareStatement()中设置参Statement.RETURN_GENERATED_KEYS表示允许获取自增的主键。

实现步骤如下：

```java
statement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
statement.executeUpdate();
resultSet = statement.getGeneratedKeys();
resultSet.next();
int id = resultSet.getInt(1);
```

操作示例如下：

```java
import java.sql.*;

public class JdbcQuery {
    public static void main(String[] args) throws Exception {
        // 1.加载驱动
        Class.forName("org.h2.Driver");
        // 2.获取连接对象
        Connection connection = DriverManager.getConnection("jdbc:h2:mem:test_mem", "h2", "");
        // 3.获得执行SQL的对象
        Statement statement = connection.createStatement();
        // 4.执行SQL语句，并接收结果
        statement.execute("DROP TABLE IF EXISTS USER_INFO");
        statement.execute("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY AUTO_INCREMENT, USER_NAME VARCHAR(255))");

        // 方式一: 新增数据返回自动增长主键ID的值
        statement.executeUpdate("INSERT INTO USER_INFO(USER_NAME) VALUES('Java')", Statement.RETURN_GENERATED_KEYS);
        // 以结果集的方式获取自增的主键
        ResultSet resultSet = statement.getGeneratedKeys();
        resultSet.next();
        System.out.println(resultSet.getInt(1));
        statement.executeUpdate("INSERT INTO USER_INFO(USER_NAME) VALUES('Python')", Statement.RETURN_GENERATED_KEYS);
        // 以结果集的方式获取自增的主键
        resultSet = statement.getGeneratedKeys();
        resultSet.next();
        int id = resultSet.getInt(1);
        System.out.println(id);

        // 方式二: 新增数据返回指定字段, 这里我们指定返回主键字段
        PreparedStatement ps = connection.prepareStatement("INSERT INTO USER_INFO(USER_NAME) VALUES(?)", new int[]{1});
        ps.setString(1, "Javascript");
        ps.executeUpdate();
        resultSet = ps.getGeneratedKeys();
        resultSet.next();
        System.out.println(resultSet.getInt(1));

        // 方式二: 新增数据返回指定字段, 这里我们指定返回主键字段
        ps = connection.prepareStatement("INSERT INTO USER_INFO(USER_NAME) VALUES(?)", new String[]{"USER_ID"});
        ps.setString(1, "SpringBoot");
        ps.executeUpdate();
        resultSet = ps.getGeneratedKeys();
        resultSet.next();
        System.out.println(resultSet.getInt(1));

        resultSet = statement.executeQuery("SELECT * FROM USER_INFO");

        // 5.处理结果集
        while (resultSet.next()) {
            // 5.1有数据，依据列名获取数据
            String user_id = resultSet.getString("USER_ID");
            String user_name = resultSet.getString("USER_NAME");
            System.out.println(user_id + ": " + user_name);
        }

        // 6.释放资源
        resultSet.close();
        statement.close();
        connection.close();
    }
}
```

```
1
2
3
4
1: Java
2: Python
3: Javascript
4: SpringBoot
```



### 2、批处理

- statement.addBatch();将SQL语句攒为一批
- statement.executeBatch();批处理
- **注意：要实现批处理，必须在连接地址后添加一个参数rewriteBatchedStatements=true**【本人测试不是所有数据库都需要】

```java
package org.example;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.List;

public class JdbcQuery {
    public static void main(String[] args) throws Exception {
        // 1.加载驱动
        Class.forName("org.h2.Driver");
        // 2.获取连接对象
        Connection connection = DriverManager.getConnection("jdbc:h2:mem:test_mem", "h2", "");
        // 3.获得执行SQL的对象PreparedStatement, 创建表和插入数据
        connection.prepareStatement("DROP TABLE IF EXISTS USER_INFO").execute();
        connection.prepareStatement("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))").execute();
        PreparedStatement pstmtInsert = connection.prepareStatement("INSERT INTO USER_INFO VALUES(?, ?)");
        for (int i = 0; i < 100000; i++) {
            pstmtInsert.setInt(1, i + 1);
            pstmtInsert.setString(2, "模拟批处理数据" + i);
            // 添加到批处理一组操作中，攒一块处理
            pstmtInsert.addBatch();
            // 有时候也攒一部分，执行一部分
            if (i % 10000 == 0) {
                // 执行
                pstmtInsert.executeBatch();
                // 清空
                pstmtInsert.clearBatch();
            }
        }
        // 最后执行批处理操作
        pstmtInsert.executeBatch();
        System.out.println("批处理数据插入成功");
        
        // 4.释放资源
        pstmtInsert.close();
        connection.close();
    }
}
```



## 十、数据库连接池

用户每次请求都需要向数据库获得链接，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。假设网站一天10万访问量，数据库服务器就需要创建10万次连接，极大的浪费数据库的资源，并且极易造成数据库服务器内存溢出、拓机。

### 1、数据库连接池的基本概念

数据库连接是一种关键的有限的昂贵的资源，这一点在多用户的网页应用程序中体现的尤为突出.对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标.数据库连接池正式针对这个问题提出来的。**数据库连接池负责分配,管理和释放数据库连接,它允许应用程序重复使用一个现有的数据库连接,而不是重新建立一个。**

数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中, 这些数据库连接的数量是由最小数据库连接数来设定的.无论这些数据库连接是否被使用,连接池都将一直保证至少拥有这么多的连接数量.连接池的最大数据库连接数量限定了这个连接池能占有的最大连接数,当应用程序向连接池请求的连接数超过最大连接数量时,这些请求将被加入到等待队列中.

数据库连接池的最小连接数和最大连接数的设置要考虑到以下几个因素：

1. 最小连接数：是连接池一直保持的数据库连接,所以如果应用程序对数据库连接的使用量不大,将会有大量的数据库连接资源被浪费. 
2. 最大连接数：是连接池能申请的最大连接数,如果数据库连接请求超过次数,后面的数据库连接请求将被加入到等待队列中,这会影响以后的数据库操作
3. 如果最小连接数与最大连接数相差很大：那么最先连接请求将会获利,之后超过最小连接数量的连接请求等价于建立一个新的数据库连接.不过,这些大于最小连接数的数据库连接在使用完不会马上被释放,他将被放到连接池中等待重复使用或是空间超时后被释放

**编写连接池需实现java.sql.DataSource接口。**

**数据源（DataSource）**：是一种升级版的 DriverManager。之所以需要这种升级版，原因是 DriverManager 过于底层。数据源可以由连接池作为实现，用以减少连接数据库时所带来的消耗。DataSource 位于 java**x**.sql.DataSource （不是 `java.sql.DataSource`）。从代码逻辑来讲，DataSource 与 DriverManage 几乎是等同的，DataSource 同样有一个类似的方法 `Connection getConnection(...)`。



### 2、开源数据库连接池

现在很多WEB服务器(Weblogic, WebSphere, Tomcat)都提供了DataSoruce的实现，即连接池的实现。通常我们把DataSource的实现，按其英文含义称之为数据源，数据源中都包含了数据库连接池的实现。

也有一些开源组织提供了数据源的独立实现：

- DBCP 数据库连接池
- C3P0 数据库连接池
- Druid 数据库连接池
- HikariCP 数据库连接池

在使用了数据库连接池之后，在项目的实际开发中就不需要编写连接数据库的代码了，直接从数据源获得数据库的连接。



### 3、DBCP 数据源

DBCP 是 Apache 软件基金组织下的开源连接池实现，要使用DBCP数据源，需要应用程序应在系统中增加如下两个 jar 文件：

- Commons-dbcp.jar：连接池的实现
- Commons-pool.jar：连接池实现的依赖库

Tomcat 的连接池正是采用该连接池来实现的。该数据库连接池既可以与应用服务器整合使用，也可由应用程序独立使用。

1、导入相关jar包

```xml
<!-- https://mvnrepository.com/artifact/commons-dbcp/commons-dbcp -->
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
<!-- https://mvnrepository.com/artifact/commons-pool/commons-pool -->
<dependency>
    <groupId>commons-pool</groupId>
    <artifactId>commons-pool</artifactId>
    <version>1.6</version>
</dependency>
```

2、在类目录下加入dbcp的配置文件：dbcpconfig.properties

```properties
# 连接设置
driverClassName=org.h2.Driver
url=jdbc:h2:mem:test_mem
username=h2
password=

# 初始化连接
initialSize=10
# 最大连接数量
maxActive=50
# 最大空闲连接
maxIdle=20
# 最小空闲连接
minIdle=5
# 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒
maxWait=60000
# JDBC驱动建立连接时附带的连接属性属性的格式必须为这样：[属性名=property;]
# 注意："user" 与 "password" 两个属性会被明确地传递，因此这里不需要包含他们
connectionProperties=useUnicode=true;characterEncoding=UTF8
# 指定由连接池所创建的连接的自动提交（auto-commit）状态。
defaultAutoCommit=true
# driver default 指定由连接池所创建的连接的只读（read-only）状态。
# 如果没有设置该值，则“setReadOnly”方法将不被调用.（某些驱动并不支持只读模式，如：Informix）
defaultReadOnly=

# driver default 指定由连接池所创建的连接的事务级别（TransactionIsolation）。
# 可用值为下列之一：（详情可见javadoc。）NONE,READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE
defaultTransactionIsolation=READ_UNCOMMITTED
```

3、编写工具类 JdbcUtils_DBCP.java

```java
import java.io.InputStream;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;
import javax.sql.DataSource;
import org.apache.commons.dbcp.BasicDataSourceFactory;

// 数据库连接工具类
public class JdbcUtils_DBCP {

    /**
     * 在java中，编写数据库连接池需实现java.sql.DataSource接口，
     * 每一种数据库连接池都是DataSource接口的实现
     * DBCP连接池就是java.sql.DataSource接口的一个具体实现
     */
    private static DataSource ds = null;

    // 在静态代码块中创建数据库连接池
    static {
        try {
            // 加载dbcpconfig.properties配置文件
            InputStream in = JdbcUtils_DBCP.class.getClassLoader().getResourceAsStream("dbcpconfig.properties");
            Properties prop = new Properties();
            prop.load(in);
            // 创建数据源
            ds = BasicDataSourceFactory.createDataSource(prop);
        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    // 从数据源中获取数据库连接
    public static Connection getConnection() throws SQLException {
        // 从数据源中获取数据库连接
        return ds.getConnection();
    }

    // 释放资源
    public static void release(Connection conn, Statement st, ResultSet rs) {
        if (rs != null) {
            try {
                // 关闭存储查询结果的ResultSet对象
                rs.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
            rs = null;
        }
        if (st != null) {
            try {
                // 关闭负责执行SQL命令的Statement对象
                st.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                // 将Connection连接对象还给数据库连接池 
                conn.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

4、测试类

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class DBCPTest {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement st = null;
        ResultSet rs = null;
        try {
            // 获取数据库连接
            conn = JdbcUtils_DBCP.getConnection();
            conn.prepareStatement("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))").execute();
            st = conn.prepareStatement("INSERT INTO USER_INFO VALUES(?, ?)");
            st.setInt(1, 1);
            st.setString(2, "Hello");
            int i = st.executeUpdate();
            if (i > 0) {
                System.out.println("插入成功");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //释放资源
            JdbcUtils_DBCP.release(conn, st, rs);
        }
    }
}
```



### 4、C3P0 数据源

C3P0是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。目前使用它的开源项目有**Hibernate**，**Spring**等。C3P0数据源在项目开发中使用得比较多。

**C3P0与DBCP区别**

- DBCP没有自动回收空闲连接的功能
- C3P0有自动回收空闲连接功能

1、导入相关jar包

```xml
<!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.5</version>
</dependency>
```

2、在类目录下加入C3P0的配置文件：c3p0-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<c3p0-config>
    <!--
    C3P0的缺省(默认)配置，
    如果在代码中“ComboPooledDataSource ds = new ComboPooledDataSource();”
	这样写就表示使用的是C3P0的缺省(默认)配置信息来创建数据源
	-->
    <default-config>
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">
            jdbc:mysql://localhost:3306/jdbcStudy?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false
        </property>
        <property name="user">root</property>
        <property name="password">password</property>
        <property name="acquireIncrement">5</property>
        <property name="initialPoolSize">10</property>
        <property name="minPoolSize">5</property>
        <property name="maxPoolSize">20</property>
    </default-config>

    <!--
    C3P0的命名配置，
    如果在代码中“ComboPooledDataSource ds = new ComboPooledDataSource("H2");”
	这样写就表示使用的是name是MySQL的配置信息来创建数据源
    -->
    <named-config name="MySQL">
        <property name="driverClass">org.h2.Driver</property>
        <property name="jdbcUrl">jdbc:h2:mem:test_mem</property>
        <property name="user">h2</property>
        <property name="password"></property>
        <property name="acquireIncrement">5</property>
        <property name="initialPoolSize">10</property>
        <property name="minPoolSize">5</property>
        <property name="maxPoolSize">20</property>
    </named-config>
</c3p0-config>
```

3、创建工具类

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import com.mchange.v2.c3p0.ComboPooledDataSource;

// 数据库连接工具类
public class JdbcUtils_C3P0 {
    private static ComboPooledDataSource ds = null;

    // 在静态代码块中创建数据库连接池
    static {
        try {
            // 通过代码创建C3P0数据库连接池 
            /*  ds = new ComboPooledDataSource(); 
                ds.setDriverClass("com.mysql.jdbc.Driver"); 
                ds.setJdbcUrl("jdbc:mysql://localhost:3306/jdbcstudy"); 
                ds.setUser("root"); 
                ds.setPassword("123456"); 
                ds.setInitialPoolSize(10);
                ds.setMinPoolSize(5); 
                ds.setMaxPoolSize(20);
              */
            // 通过读取C3P0的xml配置文件创建数据源，C3P0的xml配置文件c3p0-config.xml必须放在src目录下
            // ds = new ComboPooledDataSource();     // 使用C3P0的默认配置来创建数据源
            ds = new ComboPooledDataSource("H2"); // 使用C3P0的命名配置来创建数据源
        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    // 从数据源中获取数据库连接
    public static Connection getConnection() throws SQLException {
        // 从数据源中获取数据库连接
        return ds.getConnection();
    }

    // 释放资源
    public static void release(Connection conn, Statement st, ResultSet rs) {
        if (rs != null) {
            try {
                // 关闭存储查询结果的ResultSet对象
                rs.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
            rs = null;
        }
        if (st != null) {
            try {
                // 关闭负责执行SQL命令的Statement对象
                st.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                // 将Connection连接对象还给数据库连接池
                conn.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

4、测试类

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class C3P0Test {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement st = null;
        ResultSet rs = null;
        try {
            // 获取数据库连接
            conn = JdbcUtils_C3P0.getConnection();
            conn.prepareStatement("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))").execute();
            st = conn.prepareStatement("INSERT INTO USER_INFO VALUES(?, ?)");
            st.setInt(1, 1);
            st.setString(2, "Hello");
            int i = st.executeUpdate();
            if (i > 0) {
                System.out.println("插入成功");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 释放资源
            JdbcUtils_C3P0.release(conn, st, rs);
        }
    }
}
```



### 5、Druid 数据源

> 步骤：
>
> 1. 导入jar包 (druid-1.0.9.jar)
> 2. 定义配置文件：是properties形式的，直接放在src目录下
> 3. 加载配置文件。Properties
> 4. 获取数据库连接池对象：通过工厂来来获取  DruidDataSourceFactory
> 5. 获取连接：getConnection

1、导入pom.xml依赖

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.18</version>
</dependency>
```

2、druid.properties

```properties
driverClassName=org.h2.Driver
url=jdbc:h2:mem:test_mem
username=h2
password=
initialSize=5
maxActive=10
maxWait=3000
```

3、DruidUtil

```java
package org.example;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Properties;

/**
 * druid连接池的工具类需要具备哪些内容
 * 1.加载配置文件:要想优先加载，可以放在静态代码块中
 * 2.提供方法 获取数据库连接对象
 * 3.提供方法  获取连接池对象
 * 4.释放获取归还相关资源   rest  pst  con
 */
public class DruidUtil {
    static DataSource ds;

    static {
        try {
            InputStream is = DruidUtil.class.getClassLoader().getResourceAsStream("druid.properties");
            Properties properties = new Properties();
            properties.load(is);
            ds = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 2.提供方法 获取数据库连接对象
    public static Connection getConnection() throws Exception {
        return ds.getConnection();
    }

    public static void close(PreparedStatement pst, Connection con) {
        close(null, pst, con);
    }

    // 3.释放获取归还相关资源   rest  pst  con
    public static void close(ResultSet rest, PreparedStatement pst, Connection con) {
        if (rest != null) {
            try {
                rest.close();
                rest = null;//加快释放的速度
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }

        if (pst != null) {
            try {
                pst.close();
                pst = null;//加快释放的速度
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }

        if (con != null) {
            try {
                con.close();
                con = null;//加快释放的速度
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
```

4、DruidDemo.java

```java
import java.sql.Connection;
import java.sql.PreparedStatement;

public class DruidDemo {
    public static void main(String[] args) throws Exception {
        // 创建druid连接池对象（阿里巴巴）
        // DruidDataSource ds = new DruidDataSource();

        // 使用工厂类获取druid对象, 需要手动加载配置文件, druid.properties建议放在src
        // String path = DruidDemo1.class.getClassLoader().getResource("druid.properties").getPath();
        // Properties pt = new Properties();
        // pt.load(new FileReader(path));

        // InputStream is = DruidDemo1.class.getClassLoader().getResourceAsStream("druid.properties");
        // Properties pt = new Properties();
        // pt.load(is);

        // DataSource ds = DruidDataSourceFactory.createDataSource(pt);

        Connection conn = DruidUtil.getConnection();
        PreparedStatement pst = conn.prepareStatement("CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255))");
        pst.executeUpdate();
        pst = conn.prepareStatement("INSERT INTO USER_INFO VALUES(?, ?)");
        pst.setInt(1, 1);
        pst.setString(2, "Hello");
        int i = pst.executeUpdate();
        if (i > 0) {
            System.out.println("插入成功");
        }
        conn.close();
    }
}
```



## 十一、JDBC 元数据

### 1、什么是元数据

- 元数据：MetaData，定义数据的数据。在数据库里，指数据库、表、字段等等的定义信息
- JDBC里有哪些常见的元数据对象：
  - `DatabaseMetaData`：定义数据库的信息，名称、类型、版本等等
  - `ParameterMetaData`：定义预编译对象的信息，包含了预编译的SQL中，参数的个数、类型等等
  - `ResultSetMetaData`：定义结果集对象的信息，包含查询结果集里，列的个数、名称、类型等等



### 2、ParameterMetaData

- 怎样获取：`preparedStatement.getParameterMetaData()`

- 使用的API：

  - 获取预编译SQL中，参数个数：`getParameterCount()`
  - 获取预编译SQL中，参数类型：`getParamteType(int 参数序号)`

  > 注意：并非所有数据库都可以获取SQL中参数类型，比如：MySql



### 3、ResultSetMetaData

- 怎样获取：`resultSet.getMetadata()`
- 使用的API：
  - 获取结果集中，列的个数：`getColumnCount()`
  - 获取结果集中，列的名称：`getColumnName(int 列序号)`
  - 获取结果集中，列的类型：`getColumnType(int 列序号)`



## 十二、常见异常信息

- java.lang.ClassNotFoundException：找不到类（类名书写错误、没有导入jar包）

- java.sql.SQLException：与sql语句相关的错误（约束错误、表名列名书写错误）

  建议：在客户端工具中测试SQL语句之后再粘贴在代码中

- com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Unknown column

  原因：列值String类型没有加单引号

- Duplicate entry '1' for key 'PRIMARY'

  原因，主键值已存在或混乱，更改主键值或清空表

- com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Unknown column 'password' in

  原因：可能输入的值的类型不对，确定是否插入的元素时对应的值的类型正确



## 十二、参考文献 & 鸣谢

- JDBC基础：https://mp.weixin.qq.com/s/gFbVoPxuLlOmJFK-i74hCg
- 连接池&JDBCTemplate：https://mp.weixin.qq.com/s/c4AwjX-IwWmFP2qdN3yAWw
- JDBC核心技术笔记—— 目录：https://blog.csdn.net/kuaixiao0217/article/details/124578876
- JDBC 系列篇：https://blog.csdn.net/lizhiqiang1217/category_8979965.html
- JDBC、DriverManager(驱动管理器) 与DataSource(数据源) 的区别：https://blog.csdn.net/qq_39326472/article/details/123389429