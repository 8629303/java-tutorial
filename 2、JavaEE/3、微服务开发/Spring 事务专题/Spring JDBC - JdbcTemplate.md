# 一、JdbcTemplate 类介绍

Spring 对数据库的操作在 JDBC 上面做了深层次的封装，使用 Spring 的注入功能，可以把 DataSource 注册到 JdbcTemplate 中。

JdbcTemplate 是 Spring 框架中提供的一个对象，是对原始繁琐的 Jdbc API 对象的简单封装。Spring 框架为我们提供了很多的操作模板类。例如：操作关系型数据的 JdbcTemplate 和 HibernateTemplate，操作 NoSQL 数据库的 RedisTemplate，操作消息队列的JmsTemplate等等。

JdbcTemplate 位于 spring-jdbc-x.x.x.jar 中，其全限定命名为 org.springframework.jdbc.core.JdbcTemplate。要使用 JdbcTemplate 还需要一个  spring-tx-x.x.x.jar 包，这个包含了事务和异常控制。

- spring-jdbc-x.x.x.jar
- spring-tx-x.x.x.jar
- spring-core-x.x.x.jar
- spring-beans-x.x.x.jar

> 环境搭建：

- 需要导入Spring的 tx 和 jdbc 模块
- 需要相关数据库的驱动
- 数据库连接池（仅就本文非必需）
- 需要有相应的数据库进行操作

1、导入依赖

```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <spring-version>5.3.29</spring-version>
    <mysql-connector-version>5.1.49</mysql-connector-version>
</properties>

<dependencies>
    <!-- Spring的 context 和 jdbc 模块 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring-version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring-version}</version>
    </dependency>

    <!-- H2数据库驱动 -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>2.2.220</version>
    </dependency>

    <!-- Mysql数据库驱动 -->
    <!--        <dependency>-->
    <!--            <groupId>mysql</groupId>-->
    <!--            <artifactId>mysql-connector-java</artifactId>-->
    <!--            <version>${mysql-connector-version}</version>-->
    <!--        </dependency>-->
    <!-- 阿里的德鲁伊连接池 -->
    <!--        <dependency>-->
    <!--            <groupId>com.alibaba</groupId>-->
    <!--            <artifactId>druid</artifactId>-->
    <!--            <version>${druid-version}</version>-->
    <!--        </dependency>-->
</dependencies>
<repositories>
    <repository>
        <id>nexus-aliyun</id>
        <name>nexus-aliyun</name>
        <url>https://maven.aliyun.com/nexus/content/groups/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

2、简单使用

```java
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

public class Main {
    public static void main(String[] args) {
        // 获取数据源, 这里使用H数据库, 不知道为什么使用mem内存模式一直报错, 所以使用file文件模式
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver"); // 驱动
        dataSource.setUrl("jdbc:h2:file:~/test;;DATABASE_TO_UPPER=false"); // 数据库连接
        dataSource.setUsername("sa"); // 用户
        dataSource.setPassword(""); // 密码
        // 如果要使用其他数据源只需要修改上面部分, 然后把dataSource注入JdbcTemplate即可

        // 获取 JdbcTemplate
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

        // 先初始化表和初始化数据
        jdbcTemplate.execute("drop table if exists user_info");
        jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
        jdbcTemplate.update("insert into user_info values(1, 'Java')");
        jdbcTemplate.update("insert into user_info values(2, 'Python')");
        jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

        // 查询数据
        System.out.println(jdbcTemplate.queryForList("select * from user_info"));
    }
}
```

```、
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
```

3、配置文件定义，将数据源连接池以及 jdbcTemplate 改为配置文件定义。或者直接@Bean定义也行。

```xml
<!-- 德鲁伊连接池 -->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
</bean>

<!-- jdbcTemplate -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <!-- 设置数据源 -->
    <property name="dataSource" ref="dataSource"/>
</bean>
```



# 二、JdbcTemplate 方法使用概述

JdbcTemplate 主要提供以下五类或四类方法：

- execute 方法：可以用于执行任何SQL语句，一般用于执行DDL语句；
- update 方法及 batchUpdate 方法：用于执行新增、修改、删除等DML语句；batchUpdate 方法用于执行批量处理相关语句。

- query 方法及 queryForXXX 方法：用于执行查询相关DQL语句；

- call 方法：用于执行存储过程、函数相关语句。

注意：所有执行查询的语句，都要注意查询结果为空的情况，有些方法对于空的查询结果会报异常。

其中主要三类操作如下：

- execute：可以执行所有SQL语句，一般用于执行DDL语句。
- update：用于执行INSERT、UPDATE、DELETE等DML语句。
- queryXxx：用于DQL数据查询语句。

query 方法及 queryForXXX 方法 重点概述：

- queryForObject：查询结果，将结果封装为对象，一般用于聚合函数的查询。一般用户单行单列或单行多列
- queryForMap()：查询结果将结果集封装为map集合，列名作为key，值作为value。 该方法只能查一行数据。一般用于单行多列
- queryForList()：查询结果将结果集封装为list集合。注意：每一条记录为一个Map集合，再将Map集合装载到List集合中。一般用于多行多列。
- queryForRowSet：查询结果将结果集封装为SqlRowSet对象，该对象是一个集合，可以理解为ResultSet对象，一般用于多行多列。
- queryForStream：查询结果将结果集封装为Stream对象，该对象可以转换成绩集合也可以是单个对象。一般用于多行多列。
- query()：查询结果，将结果封装为JavaBean对象。
  1. query 的参数：RowCallbackHandler，这个回调方式，一条记录回调一次，不返回结果。
  2. query 的参数：ResultSetExtractor，这个回调方式，接收的是批量的结果，一次对所有的结果进行转换。返回类型为泛型ResultSetExtractor< T >，T 可以是单个JavaBean也可以是一个List。
  3. query 的参数：RowMapper，一般我们使用BeanPropertyRowMapper实现类。可以完成数据到JavaBean的自动封装new BeanPropertyRowMapper<类型>(类型.class)。RowMapper 返回结果是List。

JdbcTemplate 类支持的回调类：

- 预编译语句及存储过程创建回调：用于根据JdbcTemplate提供的连接创建相应的语句；
  - PreparedStatementCreator：通过回调获取JdbcTemplate提供的Connection，由用户使用该Conncetion创建相关的PreparedStatement；
  - CallableStatementCreator：通过回调获取JdbcTemplate提供的Connection，由用户使用该Conncetion创建相关的CallableStatement；
- 预编译语句设值回调：用于给预编译语句相应参数设值；
  - PreparedStatementSetter：通过回调获取JdbcTemplate提供的PreparedStatement，由用户来对相应的预编译语句相应参数设值；
  - BatchPreparedStatementSetter：类似于PreparedStatementSetter，但用于批处理，需要指定批处理大小；
- 自定义功能回调：提供给用户一个扩展点，用户可以在指定类型的扩展点执行任何数量需要的操作；
  -  StatementCallback：通过回调获取JdbcTemplate提供的Statement，用户可以在该Statement执行任何数量的操作；
  - PreparedStatementCallback：通过回调获取JdbcTemplate提供的PreparedStatement，用户可以在该PreparedStatement执行任何数量的操作；
  - CallableStatementCallback：通过回调获取JdbcTemplate提供的CallableStatement，用户可以在该CallableStatement执行任何数量的操作；
- 结果集处理回调：通过回调处理ResultSet或将ResultSet转换为需要的形式；
  - RowMapper：用于将结果集每行数据转换为需要的类型，用户需实现方法mapRow(ResultSet rs, int rowNum)来完成将每行数据转换为相应的类型。
  - RowCallbackHandler：用于处理ResultSet的每一行结果，用户需实现方法processRow(ResultSet rs)来完成处理，在该回调方法中无需执行rs.next()，该操作由JdbcTemplate来执行，用户只需按行获取数据然后处理即可。
  - ResultSetExtractor：用于结果集数据提取，用户需实现方法extractData(ResultSet rs)来处理结果集，用户必须处理整个结果集；



## 1、execute 系列

### 1、execute(final String sql)

```java
public void execute(final String sql) throws DataAccessException {}
```

该方法可以执行任意SQL，参数是一个SQL，没有返回值。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute1() {
    jdbcTemplate.execute("drop table if exists user_info"); // 删除表
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))"); // 创建表
    jdbcTemplate.execute("insert into user_info values(1, 'Java')"); // 插入数据
    jdbcTemplate.execute("update user_info set user_name = 'Python' where user_id = 1"); // 更新数据
    jdbcTemplate.execute("delete from user_info where user_id = 1"); // 删除数据
    jdbcTemplate.execute("truncate table user_info"); // 清空表数据
    jdbcTemplate.execute("begin add_data_to_account; end;"); // 执行存储过程
}
```



### 2、execute(StatementCallback< T > action)

```JAVA
public <T> T execute(StatementCallback<T> action) throws DataAccessException {}
```

此方法是传入一个 StatementCallback 对象。StatementCallback 接口只有一个方法。

```java
@FunctionalInterface
public interface StatementCallback<T> {
    @Nullable
    T doInStatement(Statement stmt) throws SQLException, DataAccessException;
}
```

通过回调获取 JdbcTemplate 提供的 Statement，用户可以在该 Statement 进行 SQL 操作。

其实 execute(final String sql) 该方法就是调用 execute(StatementCallback< T> action) 这个方法实现的。可以执行任意 SQL。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute2() {
    // 删除表、创建表、新增数据
    Integer num = jdbcTemplate.execute(new StatementCallback<Integer>() {
        @Override
        public Integer doInStatement(Statement st) throws SQLException, DataAccessException {
            st.execute("drop table if exists user_info"); // 删除表
            st.execute("create table user_info(user_id int primary key, user_name varchar(255))"); // 创建表
            return st.executeUpdate("insert into user_info values(1, 'Java')"); // 插入数据
        }
    });
    System.out.println("插入数据: " + num + " 条");

    // 更新数据、删除数据。采用Lambda表达式
    num = jdbcTemplate.execute((StatementCallback<Integer>) st -> {
        // 更新数据
        int numByUpdate = st.executeUpdate("update user_info set user_name = 'Python' where user_id = 1");
        System.out.println("更新数据: " + numByUpdate + " 条");
        return st.executeUpdate("delete from user_info where user_id = 1"); // 删除数据
    });
    System.out.println("删除数据: " + num + " 条");
}
```

```
插入数据: 1 条
更新数据: 1 条
删除数据: 1 条
```

实际上一些 query()、update()、batchUpdate() 内部都是调用这个方法。



### 3、execute(ConnectionCallback< T > action)

```java
public <T> T execute(ConnectionCallback<T> action) throws DataAccessException {}
```

通过回调获取 JdbcTemplate 提供的 Connection，用户可在该 Connection 执行一些操作。例如：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute3() {
    // 删除表、创建表、新增数据
    Integer num = jdbcTemplate.execute(new ConnectionCallback<Integer>() {
        @Override
        public Integer doInConnection(Connection con) throws SQLException, DataAccessException {
            Statement st = con.createStatement();
            st.execute("drop table if exists user_info"); // 删除表
            st.execute("create table user_info(user_id int primary key, user_name varchar(255))"); // 创建表
            return st.executeUpdate("insert into user_info values(1, 'Java')"); // 插入数据
        }
    });
    System.out.println("插入数据: " + num + " 条");

    // 更新数据、删除数据。采用Lambda表达式
    num = jdbcTemplate.execute((ConnectionCallback<Integer>) con -> {
        Statement st = con.createStatement();
        // 更新数据
        int numByUpdate = st.executeUpdate("update user_info set user_name = 'Python' where user_id = 1");
        System.out.println("更新数据: " + numByUpdate + " 条");
        return st.executeUpdate("delete from user_info where user_id = 1"); // 删除数据
    });
    System.out.println("删除数据: " + num + " 条");
}
```

```
插入数据: 1 条
更新数据: 1 条
删除数据: 1 条
```



### 4、execute(String sql, PreparedStatementCallback< T > action)

```java
public <T> T execute(String sql, PreparedStatementCallback<T> action) throws DataAccessException {}
```

由传入的 SQL 生成一个预编译语句，PreparedStatementCallback 回调传回，由用户决定如何执行该 PreparedStatement。

```java
@Override
@Nullable
public <T> T execute(String sql, PreparedStatementCallback<T> action) throws DataAccessException {
    return execute(new SimplePreparedStatementCreator(sql), action);
}
```

execute(String sql, PreparedStatementCallback< T > action) 与 execute(PreparedStatementCreator psc, PreparedStatementCallback< T > action) 内部都是调用同一个方法。

操作示例：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute4() {
    // 先初始化表
    jdbcTemplate.execute("drop table if exists user_info"); // 删除表
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))"); // 创建表

    // 插入数据
    String insertSql = "insert into user_info values(?, ?)";
    Integer num = jdbcTemplate.execute(insertSql, new PreparedStatementCallback<Integer>() {
        @Override
        public Integer doInPreparedStatement(PreparedStatement ps)
            throws SQLException, DataAccessException {
            ps.setInt(1, 1);
            ps.setString(2, "Java");
            return ps.executeUpdate();
        }
    });
    System.out.println("插入数据: " + num + " 条");

    // 查询数据, 采用Lambda表达式。返回对象为自定义的JavaBean
    String querySql = "select * from user_info where user_id = ?";
    UserInfo userInfo = jdbcTemplate.execute(querySql, (PreparedStatementCallback<UserInfo>) ps -> {
        ps.setInt(1, 1);
        ResultSet rs = ps.executeQuery();
        rs.next();
        return new UserInfo(rs.getInt(1), rs.getString(2));
    });
    System.out.println(userInfo);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
插入数据: 1 条
UserInfo(userId=1, userName=Java)
```



### 5、execute(PreparedStatementCreator psc, PreparedStatementCallback< T > action)

```java
public <T> T execute(PreparedStatementCreator psc, PreparedStatementCallback<T> action) throws DataAccessException {}
```

第一个参数是创建一个预编译语句，第二个是预编译的回调函数。由用户决定如何创建如何 PreparedStatementCreator，以及如何执行该 PreparedStatement。

```java
@Override
@Nullable
public <T> T execute(String sql, PreparedStatementCallback<T> action) throws DataAccessException {
    return execute(new SimplePreparedStatementCreator(sql), action);
}
```

execute(String sql, PreparedStatementCallback< T > action) 与 execute(PreparedStatementCreator psc, PreparedStatementCallback< T > action) 内部都是调用同一个方法。

操作示例：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute5() {
    // 先初始化表
    jdbcTemplate.execute("drop table if exists user_info"); // 删除表
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))"); // 创建表

    // 插入数据
    Integer num = jdbcTemplate.execute(new PreparedStatementCreator(){
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            return con.prepareStatement("insert into user_info values(?, ?)");
        }
    }, new PreparedStatementCallback<Integer>() {
        @Override
        public Integer doInPreparedStatement(PreparedStatement ps)
                throws SQLException, DataAccessException {
            ps.setInt(1, 1);
            ps.setString(2, "Java");
            return ps.executeUpdate();
        }
    });
    System.out.println("插入数据: " + num + " 条");

    // 查询数据, 采用Lambda表达式。返回对象为自定义的JavaBean
    String querySql = "select * from user_info where user_id = ?";
    UserInfo userInfo = jdbcTemplate.execute(con -> con.prepareStatement(querySql),
            (PreparedStatementCallback<UserInfo>) ps -> {
                ps.setInt(1, 1);
                ResultSet rs = ps.executeQuery();
                rs.next();
                return new UserInfo(rs.getInt(1), rs.getString(2));
            });
    System.out.println(userInfo);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
插入数据: 1 条
UserInfo(userId=1, userName=Java)
```



### 6、execute(String callString, CallableStatementCallback< T > action)

```java
public <T> T execute(String callString, CallableStatementCallback<T> action) throws DataAccessException {}
```

此方法主要是处理存储过程和函数使用。由传入的 SQL 生成一个预编译语句，CallableStatementCallback 回调传回，由用户决定如何执行该 CallableStatement。

```java
@Override
@Nullable
public <T> T execute(String callString, CallableStatementCallback<T> action) throws DataAccessException {
    return execute(new SimpleCallableStatementCreator(callString), action);
}
```

操作示例：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute6() {
    // 创建存储过程
    jdbcTemplate.execute("CREATE ALIAS getDate FOR \"java.lang.System.currentTimeMillis\"");
    jdbcTemplate.execute("CREATE ALIAS getProperty FOR \"java.lang.System.getProperty\"");

    // 调用存储过程
    String call = jdbcTemplate.execute("call getDate()", new CallableStatementCallback<String>() {
        @Override
        public String doInCallableStatement(CallableStatement cs) throws SQLException, DataAccessException {
            ResultSet resultSet = cs.executeQuery();
            resultSet.next();
            return String.valueOf(resultSet.getLong(1));
        }
    });
    System.out.println(call);

    // 调用存储过程
    call = jdbcTemplate.execute("call getProperty(?)", (CallableStatementCallback<String>) cs -> {
        cs.setString(1, "user.home");
        ResultSet resultSet = cs.executeQuery();
        resultSet.next();
        return resultSet.getString(1);
    });
    System.out.println(call);
}
```

```
1692181535524
C:\Users\lsx
```



### 7、execute(CallableStatementCreator csc, CallableStatementCallback< T > action)

```java
public <T> T execute(CallableStatementCreator csc, CallableStatementCallback<T> action) {}
```

通过 CallableStatementCreator 创建存储过程或者函数的预编译语句，然后由 CallableStatementCallback 回调函数执行，获取结果。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute7() {
    // 创建存储过程
    jdbcTemplate.execute("CREATE ALIAS getDate FOR \"java.lang.System.currentTimeMillis\"");
    jdbcTemplate.execute("CREATE ALIAS getProperty FOR \"java.lang.System.getProperty\"");

    // 调用存储过程
    String call = jdbcTemplate.execute(new CallableStatementCreator() {
        @Override
        public CallableStatement createCallableStatement(Connection con) throws SQLException {
            return con.prepareCall("call getDate()");
        }
    }, new CallableStatementCallback<String>() {
        @Override
        public String doInCallableStatement(CallableStatement cs) throws SQLException, DataAccessException {
            ResultSet resultSet = cs.executeQuery();
            resultSet.next();
            return resultSet.getString(1);
        }
    });
    System.out.println(call);

    // 调用存储过程
    call = jdbcTemplate.execute(con -> {
        CallableStatement cs = con.prepareCall("call getProperty(?)");
        cs.setString(1, "user.home");
        return cs;
    }, (CallableStatementCallback<String>) cs -> {
        // cs.setString(1, "user.home");
        ResultSet resultSet = cs.executeQuery();
        resultSet.next();
        return resultSet.getString(1);
    });
    System.out.println(call);
}
```

```
1692182057557
C:\Users\lsx
```



## 2、update 系列

update()方法可以执行所有execute()能执行的SQL，几乎涵盖了所有常用的SQL。

### 1、update(final String sql)

```java
public int update(final String sql) throws DataAccessException {}
```

可以自行查看源码，源码中可以看到这个方法其实就是调用execute(StatementCallback< T > action)他的。故此 update() 方法也能执行很多操作类的SQL，不过SQL中不能处理动态参数。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute1() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");
    // 创建存储过程
    jdbcTemplate.update("CREATE ALIAS getDate FOR \"java.lang.System.currentTimeMillis\"");

    // 插入、更新、删除数据
    int insert = jdbcTemplate.update("insert into user_info values(1, 'Java')");
    int update = jdbcTemplate.update("update user_info set user_name = 'Python' where user_id = 1");
    int delete = jdbcTemplate.update("delete from user_info where user_id = 1");
    System.out.println("插入数据: " + insert + " 条");
    System.out.println("更新数据: " + update + " 条");
    System.out.println("删除数据: " + delete + " 条");
}
```

```
插入数据: 1 条
更新数据: 1 条
删除数据: 1 条
```



### 2、update(String sql, @Nullable Object... args)

```java
public int update(String sql, @Nullable Object... args) throws DataAccessException {}
```

推荐使用。方法比较简单。其中第二个多参参数形式为SQL中动态参数的值数组。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute2() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    // 插入、更新、删除数据
    int insert = jdbcTemplate.update("insert into user_info values(?, ?)", 1, "Java");
    int update = jdbcTemplate.update("update user_info set user_name = ? where user_id = ?", "Python", 1);
    int delete = jdbcTemplate.update("delete from user_info where user_id = ?", 1);
    System.out.println("插入数据: " + insert + " 条");
    System.out.println("更新数据: " + update + " 条");
    System.out.println("删除数据: " + delete + " 条");
}
```

```
插入数据: 1 条
更新数据: 1 条
删除数据: 1 条
```



### 3、update(String sql, Object[] args, int[] argTypes)

```java
public int update(String sql, Object[] args, int[] argTypes) throws DataAccessException {}
```

执行带有动态参数的sql，通过两个数组入参。对于此方法中：

- 第一个参数：执行的SQL。
- 第二个参数：SQL中参数的值。
- 第三个数组：动态参数的各个类型。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute3() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    // 插入、更新、删除数据
    Object[] param1 = {1, "Java"};
    int[] index1 = {Types.INTEGER, Types.VARCHAR};
    int insert = jdbcTemplate.update("insert into user_info values(?, ?)", param1, index1);
    Object[] param2 = {"Python", 1};
    int[] index2 = {Types.VARCHAR, Types.INTEGER};
    int update = jdbcTemplate.update("update user_info set user_name = ? where user_id = ?", param2, index2);
    Object[] param3 = {1};
    int[] index3 = {Types.INTEGER};
    int delete = jdbcTemplate.update("delete from user_info where user_id = ?", param3, index3);
    System.out.println("插入数据: " + insert + " 条");
    System.out.println("更新数据: " + update + " 条");
    System.out.println("删除数据: " + delete + " 条");
}
```

```
插入数据: 1 条
更新数据: 1 条
删除数据: 1 条
```



### 4、update(String sql, @Nullable PreparedStatementSetter pss)

```java
public int update(String sql, @Nullable PreparedStatementSetter pss) throws DataAccessException {}
```

此方法可执行带有动态参数的SQL，参数由PreparedStatement对象操作，无需ps.execute();方法，否则SQL会被执行两次。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute4() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    // 插入、更新、删除数据
    int insert = jdbcTemplate.update("insert into user_info values(?, ?)", new PreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement pss) throws SQLException {
            pss.setInt(1, 1);
            pss.setString(2, "Java");
        }
    });
    int update = jdbcTemplate.update("update user_info set user_name = ? where user_id = ?", pss -> {
        pss.setString(1, "Python");
        pss.setInt(2, 1);
    });

    int delete = jdbcTemplate.update("delete from user_info where user_id = ?", pss -> {
        pss.setInt(1, 1);
    });
    System.out.println("插入数据: " + insert + " 条");
    System.out.println("更新数据: " + update + " 条");
    System.out.println("删除数据: " + delete + " 条");
}
```

```
插入数据: 1 条
更新数据: 1 条
删除数据: 1 条
```



### 5、update(PreparedStatementCreator psc)

```java
public int update(PreparedStatementCreator psc) throws DataAccessException {}
```

此方法执行SQL时可以传递动态的参数，使用 PreparedStatement 对象操作参数。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute5() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    // 插入、更新、删除数据
    int insert = jdbcTemplate.update(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            PreparedStatement ps = con.prepareStatement("insert into user_info values(?, ?)");
            ps.setInt(1, 1);
            ps.setString(2, "Java");
            return ps;
        }
    });
    int update = jdbcTemplate.update((PreparedStatementCreator) con -> {
        PreparedStatement ps = con.prepareStatement("update user_info set user_name = ? where user_id = ?");
        ps.setString(1, "Python");
        ps.setInt(2, 1);
        return ps;
    });
    int delete = jdbcTemplate.update(con -> {
        PreparedStatement ps = con.prepareStatement("delete from user_info where user_id = ?");
        ps.setInt(1, 1);
        return ps;
    });
    System.out.println("插入数据: " + insert + " 条");
    System.out.println("更新数据: " + update + " 条");
    System.out.println("删除数据: " + delete + " 条");
}
```

```
插入数据: 1 条
更新数据: 1 条
删除数据: 1 条
```



### 6、update(final PreparedStatementCreator psc, final KeyHolder generatedKeyHolder)

```java
public int update(final PreparedStatementCreator psc, final KeyHolder generatedKeyHolder) throws DataAccessException {}
```

此方法对于执行insert sql语句时，可查询执行的主键id的值。也就是最重要的一点根据KeyHolder获取插入记录的ID。

不过获取主键的时候需要注意数据库类别，Oracle和MySQL是不一样的。不过本人使用的是H2数据库，与MySQL类似。所以如果操作的是Mysql数据库插入动作，获取Id应该是：

```java
Number key = keyHolder.getKey();
int intValue = key.intValue();
```

操作示例：获取主键ID

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute6() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key auto_increment, user_name varchar(255))");

    // 插入数据返回主键
    GeneratedKeyHolder keyHolder = new GeneratedKeyHolder();
    int insert1 = jdbcTemplate.update(con -> {
        // 直接设置返回自增主键值, Statement.RETURN_GENERATED_KEYS 属性是返回自增主键
        PreparedStatement ps = con.prepareStatement("insert into user_info(user_name) values(?)", Statement.RETURN_GENERATED_KEYS);
        ps.setString(1, "Java");
        return ps;
    }, keyHolder);
    System.out.println("插入数据: " + insert1 + " 条");
    System.out.println(keyHolder.getKeys());
    System.out.println(keyHolder.getKey());
    int insert2 = jdbcTemplate.update(con -> {
        // 指定需要返回的列名
        PreparedStatement ps = con.prepareStatement("insert into user_info(user_name) values(?)", new String[]{"user_id"});
        ps.setString(1, "Java");
        return ps;
    }, keyHolder);
    System.out.println("插入数据: " + insert2 + " 条");
    System.out.println(keyHolder.getKeys());
    System.out.println(keyHolder.getKey());
}
```

```
插入数据: 1 条
{user_id=1}
1
插入数据: 1 条
{user_id=2}
2
```



## 3、batchUpdate 系列

同 update() 方法一样，batchUpdate() 也能处理很多批量操作的sql，比如：insert、update、delete 语句等等。

### 1、batchUpdate(final String... sql)

```java
public int[] batchUpdate(final String... sql) throws DataAccessException {}
```

使用方法如下，对于执行的SQL不能带有动态参数。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute1() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    String sql1 = "insert into user_info values(1, 'Java')";
    String sql2 = "insert into user_info values(2, 'Python')";
    String sql3 = "insert into user_info values(3, 'JavaScript')";
    int[] ints = jdbcTemplate.batchUpdate(sql1, sql2, sql3);
    System.out.println(Arrays.toString(ints));
}
```

```
[1, 1, 1]
```



### 2、batchUpdate(String sql, List<Object[]> batchArgs)

```java
public int[] batchUpdate(String sql, List<Object[]> batchArgs) throws DataAccessException {}
```

对于第二个参数 List<Object[]> batchArgs用于存放sql中的动态参数对应的值，如下：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute2() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    String sql = "insert into user_info values(?, ?)";
    Object[] objects1 = new Object[]{1, "Java"};
    Object[] objects2 = new Object[]{2, "Python"};
    Object[] objects3 = new Object[]{3, "JavaScript"};
    List<Object[]> objects = List.of(objects1, objects2, objects3);
    int[] ints = jdbcTemplate.batchUpdate(sql, objects);
    System.out.println(Arrays.toString(ints));
}
```

```
[1, 1, 1]
```



### 3、batchUpdate(String sql, List<Object[]> batchArgs, final int[] argTypes)

```java
public int[] batchUpdate(String sql, List<Object[]> batchArgs, final int[] argTypes) throws DataAccessException {}
```

同上面的方法一样，采用数组泛型的List存放参数值，第三个参数为数据库中对应字段的属性。比如：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute3() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    String sql = "insert into user_info values(?, ?)";
    Object[] objects1 = new Object[]{1, "Java"};
    Object[] objects2 = new Object[]{2, "Python"};
    Object[] objects3 = new Object[]{3, "JavaScript"};
    int[] types = new int[]{Types.INTEGER, Types.VARCHAR};
    List<Object[]> objects = List.of(objects1, objects2, objects3);
    int[] ints = jdbcTemplate.batchUpdate(sql, objects, types);
    System.out.println(Arrays.toString(ints));
}
```

```
[1, 1, 1]
```



### 4、batchUpdate(String sql, final BatchPreparedStatementSetter pss)

```java
public int[] batchUpdate(String sql, final BatchPreparedStatementSetter pss) throws DataAccessException {}
```

动态参数通过BatchPreparedStatementSetter接口的匿名类实现。比如：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute4() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    String sql = "insert into user_info values(?, ?)";
    Object[] objects1 = new Object[]{1, "Java"};
    Object[] objects2 = new Object[]{2, "Python"};
    Object[] objects3 = new Object[]{3, "JavaScript"};
    List<Object[]> objects = List.of(objects1, objects2, objects3);
    int[] ints = jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement ps, int i) throws SQLException {
            ps.setInt(1, (Integer) objects.get(i)[0]);
            ps.setString(2, (String) objects.get(i)[1]);
        }

        @Override
        public int getBatchSize() {
            return objects.size();
        }
    });
    System.out.println(Arrays.toString(ints));
}
```



### 5、batchUpdate(String sql, final Collection< T > batchArgs, final int batchSize, final ParameterizedPreparedStatementSetter< T > pss)

```java
public <T> int[][] batchUpdate(String sql, final Collection<T> batchArgs, final int batchSize,
			final ParameterizedPreparedStatementSetter<T> pss) throws DataAccessException {}
```

推荐使用。这个批量操作的方法比较好用。

- 第一个参数：要执行的SQL。
- 第二个参数：SQL参数的对象集合。
- 第三个参数：需要处理数据的数量。
- 第四个参数：根据第二个参数泛型取对应的值赋值给PreparedStatement对象，用于执行SQL。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute5() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    String sql = "insert into user_info values(?, ?)";
    List<UserInfo> datas = List.of(
            new UserInfo(1, "Java"),
            new UserInfo(2, "Python"),
            new UserInfo(3, "JavaScript"));
    int[][] ints = jdbcTemplate.batchUpdate(sql, datas, datas.size(), new ParameterizedPreparedStatementSetter<UserInfo>() {
        @Override
        public void setValues(PreparedStatement ps, UserInfo userInfo) throws SQLException {
            ps.setInt(1, userInfo.getUserId());
            ps.setString(2, userInfo.getUserName());
        }
    });
    System.out.println(Arrays.toString(ints[0]));
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
[{user_id=1}, {user_id=2}, {user_id=3}]
```



### 6、批量插入并返回主键 ID

上面还漏了一个批量插入时，也需要返回主键ID，改怎么办？

直接看JdbcTemplate的接口，并没有发现类似单个插入获取主键的方式，是不是意味着没法实现呢？

当然不是了，既然没有提供，我们完全可以依葫芦画瓢，自己实现一个 ExtendJdbcTemplate， 首先看先单个插入返回ID的实现如下：

```java
@Override
public int update(final PreparedStatementCreator psc, final KeyHolder generatedKeyHolder)
    throws DataAccessException {

    Assert.notNull(generatedKeyHolder, "KeyHolder must not be null");
    logger.debug("Executing SQL update and returning generated keys");

    return updateCount(execute(psc, ps -> {
        int rows = ps.executeUpdate();
        List<Map<String, Object>> generatedKeys = generatedKeyHolder.getKeyList();
        generatedKeys.clear();
        ResultSet keys = ps.getGeneratedKeys();
        if (keys != null) {
            try {
                RowMapperResultSetExtractor<Map<String, Object>> rse =
                    new RowMapperResultSetExtractor<>(getColumnMapRowMapper(), 1);
                generatedKeys.addAll(result(rse.extractData(keys)));
            }
            finally {
                JdbcUtils.closeResultSet(keys);
            }
        }
        if (logger.isTraceEnabled()) {
            logger.trace("SQL update affected " + rows + " rows and returned " + generatedKeys.size() + " keys");
        }
        return rows;
    }, true));
}
```

接下来，我们自己的实现可以如下：

```java
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.*;
import org.springframework.jdbc.support.JdbcUtils;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Component;

import javax.sql.DataSource;
import java.sql.*;
import java.util.*;

@Component
public class ExtendJdbcTemplate extends JdbcTemplate {
    public ExtendJdbcTemplate(DataSource dataSource) {
        super(dataSource);
    }

    public int[] batchUpdate(final PreparedStatementCreator psc, final BatchPreparedStatementSetter pss,
                             final KeyHolder generatedKeyHolder) throws DataAccessException {
        return execute(psc, ps -> {
            try {
                int batchSize = pss.getBatchSize();
                int totalRowsAffected = 0;
                int[] rowsAffected = new int[batchSize];
                List<Map<String, Object>> generatedKeys = generatedKeyHolder.getKeyList();
                generatedKeys.clear();
                ResultSet keys = null;
                for (int i = 0; i < batchSize; i++) {
                    pss.setValues(ps, i);
                    rowsAffected[i] = ps.executeUpdate();
                    totalRowsAffected += rowsAffected[i];
                    try {
                        keys = ps.getGeneratedKeys();
                        if (keys != null) {
                            RowMapper<Map<String, Object>> rowMapper = new ColumnMapRowMapper();
                            RowMapperResultSetExtractor<Map<String, Object>> rse =
                                    new RowMapperResultSetExtractor<>(rowMapper, 1);
                            generatedKeys.addAll(Objects.requireNonNull(rse.extractData(keys)));
                        }
                    } finally {
                        JdbcUtils.closeResultSet(keys);
                    }
                }
                if (logger.isDebugEnabled()) {
                    logger.debug("SQL batch update affected " + totalRowsAffected +
                            " rows and returned " + generatedKeys.size() + " keys");
                }
                return rowsAffected;
            } finally {
                if (pss instanceof ParameterDisposer) {
                    ((ParameterDisposer) pss).cleanupParameters();
                }
            }
        });
    }
}
```

```java
@Autowired
private ExtendJdbcTemplate extendJdbcTemplate;

@PostConstruct
public void execute6() {
    // 先初始化表
    jdbcTemplate.update("drop table if exists user_info");
    jdbcTemplate.update("create table user_info(user_id int primary key, user_name varchar(255))");

    String sql = "insert into user_info values(?, ?)";
    List<UserInfo> userInfos = List.of(
            new UserInfo(1, "Java"),
            new UserInfo(2, "Python"),
            new UserInfo(3, "JavaScript"));

    GeneratedKeyHolder generatedKeyHolder = new GeneratedKeyHolder();
    extendJdbcTemplate.batchUpdate(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection conn) throws SQLException {
            return conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
        }
    }, new BatchPreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement ps, int i) throws SQLException {
            ps.setInt(1, userInfos.get(i).getUserId());
            ps.setString(2, userInfos.get(i).getUserName());
        }

        @Override
        public int getBatchSize() {
            return userInfos.size();
        }
    }, generatedKeyHolder);
    System.out.println(generatedKeyHolder.getKeyList());
}
```





## 4、call 系列

### 1、call(CallableStatementCreator csc, List< SqlParameter > declaredParameters)

```java
public Map<String, Object> call(CallableStatementCreator csc, List<SqlParameter> declaredParameters)
			throws DataAccessException {}
```

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 创建存储过程
    jdbcTemplate.execute("CREATE ALIAS getDate FOR \"java.lang.System.currentTimeMillis\"");
    jdbcTemplate.execute("CREATE ALIAS getProperty FOR \"java.lang.System.getProperty\"");

    // H2调用存储过程实际上也可以不用给SqlParameter,如果有Out参数就必须要要使用
    SqlParameter parameter = new SqlParameter(Types.VARCHAR);
    Map<String, Object> call = jdbcTemplate.call(new CallableStatementCreator() {
        @Override
        public CallableStatement createCallableStatement(Connection con) throws SQLException {
            CallableStatement cs = con.prepareCall("call getProperty(?)");
            cs.setString(1, "user.home");
            return cs;
        }
    }, Collections.singletonList(parameter));
    System.out.println(call);

    // Lambda 方式调用
    call = jdbcTemplate.call(con -> con.prepareCall("call getDate()"), Collections.emptyList());
    System.out.println(call);
}
```

```
{#result-set-1=[{PUBLIC.getProperty(?1)=C:\Users\lsx}]}
{#result-set-1=[{PUBLIC.getDate()=1692331016474}]}
```



## 5、queryForObject 系列

queryForObject() 方法只能查询单个列且结果只有一行。否则报错。这一点尤其注意。

如果查询结果为空也会报错：

```java
org.springframework.dao.EmptyResultDataAccessException: Incorrect result size: expected 1, actual 0 。
```

如果查询结果是多行，则报错：

```java
org.springframework.dao.IncorrectResultSizeDataAccessException: Incorrect result size: expected 1, actual 12
```

注意：可以在网上看到很人多：查询的结果必须有且只有一行一列的值。这种说法也不完全对，如果查询结果映射为 RowMapper< T > ，泛型中可以是 Java 对象，这样就可以使一行多列了。

queryForObject：查询结果，将结果封装为对象，一般用于聚合函数的查询。



### 1、queryForObject(String sql, RowMapper< T > rowMapper)

```java
public <T> T queryForObject(String sql, RowMapper<T> rowMapper) throws DataAccessException {}
```

返回一个泛型对象。且查询SQL不能动态传参，除非将参数在查询之前动态的拼接好，RowMapper 用于处理查询的结果。此种方式既可以返回单行单列，也可以返回单行多列。主要看 RowMapper 接口中泛型的设置。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单行单列
    String sql = "select user_name from user_info where user_id = 1";
    String str = jdbcTemplate.queryForObject(sql, new RowMapper<String>() {
        @Override
        public String mapRow(ResultSet rs, int rowNum) throws SQLException {
            return rs.getString(1);
        }
    });
    System.out.println(str);

    // 查询单行多列
    sql = "select user_id,user_name from user_info where user_id = 1";
    UserInfo userInfo = jdbcTemplate.queryForObject(sql, (rs, rowNum) ->
                                                    new UserInfo(rs.getInt(1), rs.getString(2)));
    System.out.println(userInfo);

    // 注意这里Spring自身提供了一个RowMapper接口实现类:ColumnMapRowMapper
    Map<String, Object> map = jdbcTemplate.queryForObject(sql, new ColumnMapRowMapper());
    System.out.println(map);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
Java
UserInfo(userId=1, userName=Java)
{user_id=1, user_name=Java}
```



### 2、queryForObject(String sql, RowMapper< T > rowMapper, @Nullable Object... args)

```java
public <T> T queryForObject(String sql, RowMapper<T> rowMapper, @Nullable Object... args) throws DataAccessException {}
```

和上面的方法一样，区别在于查询SQL能动态传参。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单行单列
    String sql = "select user_name from user_info where user_id = ?";
    String str = jdbcTemplate.queryForObject(sql, new RowMapper<String>() {
        @Override
        public String mapRow(ResultSet rs, int rowNum) throws SQLException {
            return rs.getString(1);
        }
    }, 1);
    System.out.println(str);

    // 查询单行多列
    sql = "select user_id,user_name from user_info where user_id = ?";
    UserInfo userInfo = jdbcTemplate.queryForObject(sql,
            (rs, rowNum) -> new UserInfo(rs.getInt(1), rs.getString(2)),
            1);
    System.out.println(userInfo);

    // 注意这里Spring自身提供了一个RowMapper接口实现类:ColumnMapRowMapper
    Map<String, Object> map = jdbcTemplate.queryForObject(sql, new ColumnMapRowMapper(), 1);
    System.out.println(map);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
Java
UserInfo(userId=1, userName=Java)
{user_id=1, user_name=Java}
```



### 3、queryForObject(String sql, Object[] args, int[] argTypes, RowMapper< T > rowMapper)

```java
public <T> T queryForObject(String sql, Object[] args, int[] argTypes, RowMapper<T> rowMapper) throws DataAccessException {}
```

查询带有参数的 SQL，并且指定参数在数据库中的类型，如果SQL中没有参数，则为null。

- 第一个参数：查询SQL。

- 第二个参数：如果SQL中有动态参数，则此为SQL中占位符参数的数组，注意顺序保持一致。如果没有则为null。

- 第三个参数：如果SQL有占位符需要处理参数，则此为参数在数据库中的类型，如果SQL中没有参数，则为null。

- 第四个参数：手动处理结果集。


```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单行单列
    String sql = "select user_name from user_info where user_id = ?";
    Object[] param = {1};
    int[] index = {Types.INTEGER};
    String str = jdbcTemplate.queryForObject(sql, param, index, new RowMapper<String>() {
        @Override
        public String mapRow(ResultSet rs, int rowNum) throws SQLException {
            return rs.getString(1);
        }
    });
    System.out.println(str);

    // 查询单行多列
    sql = "select user_id,user_name from user_info where user_id = ?";
    UserInfo userInfo = jdbcTemplate.queryForObject(sql, param, index,
            (rs, rowNum) -> new UserInfo(rs.getInt(1), rs.getString(2)));
    System.out.println(userInfo);

    // 注意这里Spring自身提供了一个RowMapper接口实现类:ColumnMapRowMapper
    Map<String, Object> map = jdbcTemplate.queryForObject(sql, param, index, new ColumnMapRowMapper());
    System.out.println(map);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
Java
UserInfo(userId=1, userName=Java)
{user_id=1, user_name=Java}
```



### 4、queryForObject(String sql, Class< T > requiredType)

```java
public <T> T queryForObject(String sql, Class<T> requiredType) throws DataAccessException {}
```

这个方法就是用上面的**queryForObject(String sql, RowMapper< T > rowMapper)**方法，只不过把第二个参数封装了。requiredType 必须是常用的8个基本类型，否则报错。且查询SQL不能动态传参。

```java
@Override
@Nullable
public <T> T queryForObject(String sql, Class<T> requiredType) throws DataAccessException {
    return queryForObject(sql, getSingleColumnRowMapper(requiredType));
}
```

如果只是查询单个字段时，或者聚合查询时，推荐使用此方法。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    String userName = jdbcTemplate.queryForObject("select user_name from user_info where user_id = 1", String.class);
    System.out.println("userName: " + userName);

    Integer count = jdbcTemplate.queryForObject("select count(*) from user_info", Integer.class);
    System.out.println("count: " + count);

    // 下面这个会报错：
    //  org.springframework.dao.TypeMismatchDataAccessException: Type mismatch affecting row number 0 and column type 'CHARACTER VARYING':
    //  Value [Java] is of type [java.lang.String] and cannot be converted to required type [UserInfo]
    // UserInfo userInfo = jdbcTemplate.queryForObject("select user_name from user_info where user_id = 1", UserInfo.class);
}
```

```
userName: Java
count: 3
```



### 5、queryForObject(String sql, Class< T > requiredType, @Nullable Object... args)

```java
public <T> T queryForObject(String sql, Class<T> requiredType, @Nullable Object... args) throws DataAccessException {}
```

在上面的方法的基础上，增加了查询SQL可以动态传参。需要注意：requiredType 必须是常用的8个基本类型，否则报错。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    String sql = "select user_name from user_info where user_id = ? and user_name = ?";
    String userName = jdbcTemplate.queryForObject(sql, String.class, 1, "Java");
    System.out.println("userName: " + userName);
}
```

```
userName: Java
```



### 6、queryForObject(String sql, Object[] args, int[] argTypes, Class< T > requiredType)

```java
public <T> T queryForObject(String sql, Object[] args, int[] argTypes, Class<T> requiredType) throws DataAccessException {}
```

查询带有参数的 SQL，并且指定参数在数据库中的类型，如果SQL中没有参数，则为null。需要注意：requiredType 必须是常用的8个基本类型，否则报错。

- 第一个参数：查询SQL。
- 第二个参数：如果SQL中有动态参数，则此为SQL中占位符参数的数组，注意顺序保持一致。如果没有则为null。
- 第三个参数：如果SQL有占位符需要处理参数，则此为参数在数据库中的类型，如果SQL中没有参数，则为null。
- 第四个参数：手动处理结果集。结果集只能为常用的8个基本类型，否则报错。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    String sql = "select user_name from user_info where user_id = ? and user_name = ?";
    Object[] param = {1, "Java"};
    int[] index = {Types.INTEGER, Types.VARCHAR};
    String userName = jdbcTemplate.queryForObject(sql, param, index, String.class);
    System.out.println("userName: " + userName);
}
```

```
userName: Java
```



## 6、queryForMap 系列

queryForMap() 方法是 JdbcOperations 接口的方法，由 JdbcTemplate 子类进行了重写。queryForMap 方法查询结果将结果集封装为 Map 集合，将列名作为 key，将值作为 value 将这条记录封装为一个 Map 集合。 注意：这个方法查询的结果集长度只能是1，也就是一行。

### 1、queryForMap(String sql)

```java
public Map<String, Object> queryForMap(String sql) throws DataAccessException {}
```

返回一个Map对象。且查询SQL不能动态传参。

操作案例请参考：queryForMap(String sql, Object[] args, int[] argTypes)



### 2、queryForMap(String sql, @Nullable Object... args)

```java
public Map<String, Object> queryForMap(String sql, @Nullable Object... args) throws DataAccessException {}
```

与上面方法唯一的区别是：查询SQL能动态传参。

操作案例请参考：queryForMap(String sql, Object[] args, int[] argTypes)



### 3、queryForMap(String sql, Object[] args, int[] argTypes)

```java
public Map<String, Object> queryForMap(String sql, Object[] args, int[] argTypes) throws DataAccessException {}
```

查询带有参数的 SQL，并且指定参数在数据库中的类型，如果SQL中没有参数，则为null。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    Map<String, Object> map = jdbcTemplate.queryForMap("select * from user_info where user_id = 1");
    System.out.println(map);

    map = jdbcTemplate.queryForMap("select * from user_info where user_id = ?", 1);
    System.out.println(map);

    Object[] param = {1};
    int[] index = {Types.INTEGER};
    map = jdbcTemplate.queryForMap("select * from user_info where user_id = ?", param, index);
    System.out.println(map);

    // 如下会报错: Invocation of init method failed;
    // nested exception is org.springframework.dao.IncorrectResultSizeDataAccessException: 
    // Incorrect result size: expected 1, actual 3
    // System.out.println(jdbcTemplate.queryForMap("select * from user_info"));
}
```

```
{user_id=1, user_name=Java}
{user_id=1, user_name=Java}
{user_id=1, user_name=Java}
```



## 7、queryForRowSet 系列

queryForRowSet 方法也是 JdbcOperations 接口的方法，由 JdbcTemplate 子类进行了重写。queryForRowSet 方法返回是 SqlRowSet 对象，这是一个集合，也就是说，可以查询多条记录。【与 queryForMap 的区别：queryForMap 返回 Map 集合，并且只能放回一行数据。而 queryForRowSet 返回的是一个SqlRowSet对象，这是个集合对象，所以可以查询多条记录】

### 1、queryForRowSet(String sql)

```java
public SqlRowSet queryForRowSet(String sql) throws DataAccessException {}
```

返回一个SqlRowSet 对象。且查询SQL不能动态传参。

操作案例请参考：queryForRowSet(String sql, Object[] args, int[] argTypes)



### 2、queryForRowSet(String sql, @Nullable Object... args)

```java
public SqlRowSet queryForRowSet(String sql, @Nullable Object... args) throws DataAccessException {}
```

与上面方法唯一的区别是：查询SQL能动态传参。

操作案例请参考：queryForRowSet(String sql, Object[] args, int[] argTypes)



### 3、queryForRowSet(String sql, Object[] args, int[] argTypes)

```java
public SqlRowSet queryForRowSet(String sql, Object[] args, int[] argTypes) throws DataAccessException {}
```

查询带有参数的 SQL，并且指定参数在数据库中的类型，如果SQL中没有参数，则为null。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    String sql = "select * from user_info";
    SqlRowSet rowSet = jdbcTemplate.queryForRowSet(sql);
    List<UserInfo> userInfos = new ArrayList<>();
    while (rowSet.next()) {
        userInfos.add(new UserInfo(rowSet.getInt(1), rowSet.getString(2)));
    }
    System.out.println(userInfos);
    userInfos.clear();

    sql = "select * from user_info where user_id = ?";
    rowSet = jdbcTemplate.queryForRowSet(sql, 1);
    while (rowSet.next()) {
        userInfos.add(new UserInfo(rowSet.getInt(1), rowSet.getString(2)));
    }
    System.out.println(userInfos);
    userInfos.clear();

    Object[] param = {1};
    int[] index = {Types.INTEGER};
    rowSet = jdbcTemplate.queryForRowSet(sql, param, index);
    while (rowSet.next()) {
        userInfos.add(new UserInfo(rowSet.getInt(1), rowSet.getString(2)));
    }
    System.out.println(userInfos);
    userInfos.clear();
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
[UserInfo(userId=1, userName=Java)]
[UserInfo(userId=1, userName=Java)]
```



## 8、queryForList 系列

queryForList() 方法查询有两种形式：

1. 查询结果将结果集封装为List集合。注意：将每一条记录封装为一个Map集合，再将Map集合装载到List集合中。
2. 查询结果将结果集封装为List集合，如果指定elementType类型，则只能查询单列。再将单列这个类型装载到List集合中。

查询多行多列，以List的形式返回结果，如果指定elementType类型，则只能查询单列，否则报错：

```java
org.springframework.jdbc.IncorrectResultSetColumnCountException: Incorrect column count: expected 1, actual 4 
```

如果指定elementType类型，则只能查询单列，且类型只能时8中基本数据类型，否则报错：

```java
org.springframework.dao.TypeMismatchDataAccessException:

Type mismatch affecting row number 0 and column type 'xxxxxx':

Value [xxxxxx] is of type [java.lang.String] and cannot be converted to required type [xxx.xxx.xxx.xxx]
```

如果经常查询一些数据，建议使用queryForList()。需要注意的是：如果查询结果为空则返回空集合，也就是如下，不会报错。

```java
list.isEmpty() == true
```



### 1、queryForList(String sql)

```java
public List<Map<String, Object>> queryForList(String sql) throws DataAccessException {}
```

可以查询多行多列，返回一个List集合，List元素是Map，也就是查询结果中每一行的数据，只不过以Map的形式返回。 如果SQL中没有动态参数，**推荐使用。**

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from user_info");
    System.out.println(maps);
}
```

```
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
```



### 2、queryForList(String sql, @Nullable Object... args)

```java
public List<Map<String, Object>> queryForList(String sql, @Nullable Object... args) throws DataAccessException {}
```

与上面方法一样，如果SQL中有动态参数，推荐使用这个，比较方便。例如：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from user_info where user_id < ?", 4);
    System.out.println(maps);
}
```

```
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
```



### 3、queryForList(String sql, Object[] args, int[] argTypes)

```java
public List<Map<String, Object>> queryForList(String sql, Object[] args, int[] argTypes) throws DataAccessException {}
```

查询多行多列，SQL中可以传多个动态参数，并且指定参数在数据库中的类型，该参数可有可无，如：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    Object[] param = {4};
    int[] index = {Types.INTEGER};
    List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from user_info where user_id < ?", param, index);
    System.out.println(maps);
}
```

```
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
```



### 4、queryForList(String sql, Class< T > elementType)

```java
public <T> List<T> queryForList(String sql, Class<T> elementType) throws DataAccessException {}
```

此方法用于SQL中没有动态参数，且查询结果为单列多行，返回一个指定类型的List集合，指定的类型只能是8个基本数据类型。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    List<String> userNames = jdbcTemplate.queryForList("select user_name from user_info", String.class);
    System.out.println(userNames);
}
```

```
[Java, Python, JavaScript]
```



### 5、queryForList(String sql, Class< T > elementType, @Nullable Object... args)

```java
public <T> List<T> queryForList(String sql, Class<T> elementType, @Nullable Object... args) throws DataAccessException {}
```

查询多行单列，第二个参数的类型必须是八个基本数据类型之一，动态占位符参数采用多个参数传参的方式。如：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    String sql = "select user_name from user_info where user_id < ?";
    List<String> userNames = jdbcTemplate.queryForList(sql, String.class, 4);
    System.out.println(userNames);
}
```

```
[Java, Python, JavaScript]
```



### 6、queryForList(String sql, Object[] args, int[] argTypes, Class< T > elementType)

```java
public <T> List<T> queryForList(String sql, Object[] args, int[] argTypes, Class<T> elementType) throws DataAccessException {}
```

查询多行单列，SQL中可以有动态参数。

- 第一个参数：SQL语句。
- 第二个参数：如果SQL中有动态参数，则此为SQL中占位符参数的数组，注意顺序保持一致。如果没有则为null。
- 第三个参数：如果SQL有占位符需要处理参数，则此为参数的数据库类型，如果SQL中没有参数，则为null。该参数其实可有可无

- 第四个参数：必须是8个基本数据类型其一。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    Object[] param = {4};
    int[] index = {Types.INTEGER};
    String sql = "select user_name from user_info where user_id < ?";
    List<String> userNames = jdbcTemplate.queryForList(sql, param, index, String.class);
    System.out.println(userNames);
}
```

```
[Java, Python, JavaScript]
```



## 9、query 系列



### 1、query(String sql, RowCallbackHandler rch)

```java
public void query(String sql, RowCallbackHandler rch) throws DataAccessException {}
```

其实这个方式就是调用query(String sql, @Nullable PreparedStatementSetter pss, RowCallbackHandler rch)方法，需要用户自己取处理结果集，不过不需要判断ResultSet.next()，交由ResultSetExtractor的实现类的extractData(ResultSet rs)完成。用户只需要处理ResultSet的每一行数据。不支持SQL动态传入参数。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 一条记录回调一次, 不返回结果
    UserInfo userInfo = new UserInfo();
    jdbcTemplate.query("select * from user_info where user_id = 1", new RowCallbackHandler() {
        @Override
        public void processRow(ResultSet rs) throws SQLException {
            userInfo.setUserId(rs.getInt(1));
            userInfo.setUserName(rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 一条记录回调一次, 不返回结果, 我们可以用List去一条一条的接收数据
    List<UserInfo> userInfos = new ArrayList<>();
    jdbcTemplate.query("select * from user_info", rs -> {
        userInfos.add(new UserInfo(rs.getInt(1), rs.getString(2)));
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 2、query(String sql, RowCallbackHandler rch, @Nullable Object... args)

```java
public <T> T query(String sql, ResultSetExtractor<T> rse, @Nullable Object... args) throws DataAccessException {}
```

和上面的方法类似，只不过把参数改为动态可传参数的形式。如果没有参数就传个空的数组。主要是针对有可变参数的SQL。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 一条记录回调一次, 不返回结果
    UserInfo userInfo = new UserInfo();
    jdbcTemplate.query("select * from user_info where user_id = ?", new RowCallbackHandler() {
        @Override
        public void processRow(ResultSet rs) throws SQLException {
            userInfo.setUserId(rs.getInt(1));
            userInfo.setUserName(rs.getString(2));
        }
    }, 1);
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 一条记录回调一次, 不返回结果, 我们可以用List去一条一条的接收数据
    List<UserInfo> userInfos = new ArrayList<>();
    jdbcTemplate.query("select * from user_info where user_id < ?", rs -> {
        userInfos.add(new UserInfo(rs.getInt(1), rs.getString(2)));
    }, 4);
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 3、query(String sql, Object[] args, int[] argTypes, RowCallbackHandler rch)

```java
public void query(String sql, Object[] args, int[] argTypes, RowCallbackHandler rch) throws DataAccessException {}
```

在上面方法得基础多多了一个SQL参数类型，该参数可有可无。

- 第一个参数：要查询的SQL。

- 第二个参数：如果SQL中有动态参数，则此为sql中占位符参数的数组，注意顺序保持一致。如果没有则为null。

- 第三个参数：如果SQL有占位符需要处理参数，则此为参数的数据类型，如果sql中没有参数，则为null。该参数可有可无。

- 第四个参数：对结果集的每一行进行处理，转换成想要的对象。不需要做ResultSet.next()判断。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 一条记录回调一次, 不返回结果
    Object[] param1 = {1};
    int[] index1 = {Types.INTEGER};
    String sql = "select * from user_info where user_id = ?";
    UserInfo userInfo = new UserInfo();
    jdbcTemplate.query(sql, param1, index1, new RowCallbackHandler() {
        @Override
        public void processRow(ResultSet rs) throws SQLException {
            userInfo.setUserId(rs.getInt(1));
            userInfo.setUserName(rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 一条记录回调一次, 不返回结果, 我们可以用List去一条一条的接收数据
    Object[] param2 = {4};
    int[] index2 = {Types.INTEGER};
    sql = "select * from user_info where user_id < ?";
    List<UserInfo> userInfos = new ArrayList<>();
    jdbcTemplate.query(sql, param2, index2, rs -> {
        userInfos.add(new UserInfo(rs.getInt(1), rs.getString(2)));
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 4、query(String sql, @Nullable PreparedStatementSetter pss, RowCallbackHandler rch)

```java
public void query(String sql, @Nullable PreparedStatementSetter pss, RowCallbackHandler rch) throws DataAccessException {}
```

- 第一个参数：需要查询的SQL。
- 第二个参数：对SQL中的参数处理，如果没有则不处理。
- 第三个参数：对结果集的每一行进行处理，转换成想要的对象。不需要做ResultSet.next()判断。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 一条记录回调一次, 不返回结果
    UserInfo userInfo = new UserInfo();
    String sql = "select * from user_info where user_id = ?";
    jdbcTemplate.query(sql, new PreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement ps) throws SQLException {
            ps.setInt(1, 1);
        }
    }, new RowCallbackHandler() {
        @Override
        public void processRow(ResultSet rs) throws SQLException {
            userInfo.setUserId(rs.getInt(1));
            userInfo.setUserName(rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 一条记录回调一次, 不返回结果, 我们可以用List去一条一条的接收数据
    List<UserInfo> userInfos = new ArrayList<>();
    sql = "select * from user_info where user_id < ?";
    jdbcTemplate.query(sql, ps -> ps.setInt(1, 4), rs -> {
        userInfos.add(new UserInfo(rs.getInt(1), rs.getString(2)));
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 5、query(PreparedStatementCreator psc, RowCallbackHandler rch)

```java
public void query(PreparedStatementCreator psc, RowCallbackHandler rch) throws DataAccessException {}
```

- 参数一：构建一个预编译的SQL语句，如果有可变参数，给其赋值。
- 参数二：用户处理每一行的结果集，不需要判断ResultSet.next()。
- 注意：这是个没有返回值的方法。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 一条记录回调一次, 不返回结果
    UserInfo userInfo = new UserInfo();
    jdbcTemplate.query(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            String sql = "select * from user_info where user_id = ?";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setInt(1, 1);
            return ps;
        }
    }, new RowCallbackHandler() {
        @Override
        public void processRow(ResultSet rs) throws SQLException {
            userInfo.setUserId(rs.getInt(1));
            userInfo.setUserName(rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 一条记录回调一次, 不返回结果, 我们可以用List去一条一条的接收数据
    List<UserInfo> userInfos = new ArrayList<>();
    jdbcTemplate.query(con -> {
        String sql = "select * from user_info where user_id < ?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setInt(1, 4);
        return ps;
    }, rs -> {
        userInfos.add(new UserInfo(rs.getInt(1), rs.getString(2)));
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```



### 6、query(String sql, ResultSetExtractor rse)

```java
public <T> T query(final String sql, final ResultSetExtractor<T> rse) throws DataAccessException {}
```

该方法返回一个泛型对象。SQL为查询的SQL语句，ResultSetExtractor是结果集数据提取用的，通过extractData(ResultSet rs)处理整个结果集。不支持传入参数。可以根据自己想要的类型返回结果，需要手工处理结果集。这种查询处理既可以查询单条数据，也可以查询多条数据。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回泛型<T>
    String sql = "select * from user_info where user_id = 1";
    UserInfo userInfo = jdbcTemplate.query(sql, new ResultSetExtractor<UserInfo>() {
        @Override
        public UserInfo extractData(ResultSet rs) throws SQLException, DataAccessException {
            rs.next();
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回List<T>
    List<UserInfo> userInfos = jdbcTemplate.query("select * from user_info", rs -> {
        List<UserInfo> list = new ArrayList<>();
        while (rs.next()) {
            list.add(new UserInfo(rs.getInt(1), rs.getString(2)));
        }
        return list;
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 7、query(String sql, ResultSetExtractor rse, @Nullable Object... args)

```java
public <T> T query(String sql, ResultSetExtractor<T> rse, @Nullable Object... args) throws DataAccessException {}
```

和上面的方法类似，只不过把参数改为动态可传参数的形式。如果没有参数就传个空的数组。主要是针对有可变参数的SQL。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回泛型<T>
    String sql = "select * from user_info where user_id = ?";
    UserInfo userInfo = jdbcTemplate.query(sql, new ResultSetExtractor<UserInfo>() {
        @Override
        public UserInfo extractData(ResultSet rs) throws SQLException, DataAccessException {
            rs.next();
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    }, 1);
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回List<T>
    sql = "select * from user_info where user_id < ?";
    List<UserInfo> userInfos = jdbcTemplate.query(sql, rs -> {
        List<UserInfo> list = new ArrayList<>();
        while (rs.next()) {
            list.add(new UserInfo(rs.getInt(1), rs.getString(2)));
        }
        return list;
    }, 4);
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 8、query(String sql, Object[] args, int[] argTypes, ResultSetExtractor rse)

```java
public <T> T query(String sql, Object[] args, int[] argTypes, ResultSetExtractor<T> rse) throws DataAccessException {}
```

这个方式和上面的几个没有什么差别，看一下方法内部的调用就知道了，也是调用的query(PreparedStatementCreator psc, @Nullable final PreparedStatementSetter pss, final ResultSetExtractor< T > rse)，只不过把Object[] args, int[] argTypes这两个参数分开了。主要是针对有可变参数的sql。这两个参数的意思是：

- Object[] args：占位符对应的参数值。
- int[] argTypes：占位符参数值的类型。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回泛型<T>
    Object[] param1 = {1};
    int[] index1 = {Types.INTEGER};
    String sql = "select * from user_info where user_id = ?";
    UserInfo userInfo = jdbcTemplate.query(sql, param1, index1, new ResultSetExtractor<UserInfo>() {
        @Override
        public UserInfo extractData(ResultSet rs) throws SQLException, DataAccessException {
            rs.next();
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回List<T>
    Object[] param2 = {4};
    int[] index2 = {Types.INTEGER};
    sql = "select * from user_info where user_id < ?";
    List<UserInfo> userInfos = jdbcTemplate.query(sql, param2, index2, rs -> {
        List<UserInfo> list = new ArrayList<>();
        while (rs.next()) {
            list.add(new UserInfo(rs.getInt(1), rs.getString(2)));
        }
        return list;
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 9、query(String sql, @Nullable PreparedStatementSetter pss, ResultSetExtractor< T > rse)

```java
public <T> T query(String sql, @Nullable PreparedStatementSetter pss, ResultSetExtractor<T> rse) throws DataAccessException {}
```

和上面的方法一样，使用的是PreparedStatementSetter来支持动态的参数，需要处理SQL中的占位符。具体的处理方式都一样。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回泛型<T>
    String sql = "select * from user_info where user_id = ?";
    UserInfo userInfo = jdbcTemplate.query(sql, new PreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement ps) throws SQLException {
            ps.setInt(1, 1);
        }
    }, new ResultSetExtractor<UserInfo>() {
        @Override
        public UserInfo extractData(ResultSet rs) throws SQLException, DataAccessException {
            rs.next();
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回List<T>
    sql = "select * from user_info where user_id < ?";
    List<UserInfo> userInfos = jdbcTemplate.query(sql, ps -> ps.setInt(1, 4), rs -> {
        List<UserInfo> list = new ArrayList<>();
        while (rs.next()) {
            list.add(new UserInfo(rs.getInt(1), rs.getString(2)));
        }
        return list;
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 10、query(PreparedStatementCreator psc, @Nullable final PreparedStatementSetter pss, final ResultSetExtractor< T > rse)

```java
public <T> T query(PreparedStatementCreator psc, 
                   @Nullable final PreparedStatementSetter pss, 
                   final ResultSetExtractor<T> rse) throws DataAccessException {}
```

***这是个很重要的方法，其它很多query()方法底层都是调用这个方法实现的。***该方法有三个参数：

- PreparedStatementCreator：创建一个预编译的SQL语句。

- PreparedStatementSetter：如果SQL中有动态的参数占位符，则给占位符赋值参数值。

- ResultSetExtractor：处理SQL执行的结果，将结果集中每一行转换为需要对象，需要先调用ResultSet.next()。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回泛型<T>
    UserInfo userInfo = jdbcTemplate.query(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            String sql = "select * from user_info where user_id = ?";
            return con.prepareStatement(sql);
        }
    }, new PreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement ps) throws SQLException {
            ps.setInt(1, 1);
        }
    }, new ResultSetExtractor<UserInfo>() {
        @Override
        public UserInfo extractData(ResultSet rs) throws SQLException, DataAccessException {
            rs.next();
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回List<T>
    List<UserInfo> userInfos = jdbcTemplate.query(con -> {
        String sql = "select * from user_info where user_id < ?";
        return con.prepareStatement(sql);
    }, ps -> ps.setInt(1, 4), rs -> {
        List<UserInfo> list = new ArrayList<>();
        while (rs.next()) {
            list.add(new UserInfo(rs.getInt(1), rs.getString(2)));
        }
        return list;
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 11、query(PreparedStatementCreator psc, final ResultSetExtractor< T > rse)

```java
public <T> T query(PreparedStatementCreator psc, ResultSetExtractor<T> rse) throws DataAccessException {}
```

该方法其实和上面的一样，从其方法内部的实现可以看出就是调用上面的方法（query(PreparedStatementCreator psc, @Nullable final PreparedStatementSetter pss, final ResultSetExtractor< T > rse)）。只是不支持传入动态的参数了，使用方式和上面类似：

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 查询单个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回泛型<T>
    UserInfo userInfo = jdbcTemplate.query(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            String sql = "select * from user_info where user_id = ?";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setInt(1, 1); // 实际上这种方式不算是动态传参
            return ps;
        }
    }, new ResultSetExtractor<UserInfo>() {
        @Override
        public UserInfo extractData(ResultSet rs) throws SQLException, DataAccessException {
            rs.next();
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });
    System.out.println(userInfo);

    // 查询多个对象, 这个回调方式, 接收的是批量的结果，一次对所有的结果进行转换, 返回List<T>
    List<UserInfo> userInfos = jdbcTemplate.query(con -> {
        String sql = "select * from user_info where user_id < ?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setInt(1, 4); // 实际上这种方式不算是动态传参
        return ps;
    }, rs -> {
        List<UserInfo> list = new ArrayList<>();
        while (rs.next()) {
            list.add(new UserInfo(rs.getInt(1), rs.getString(2)));
        }
        return list;
    });
    System.out.println(userInfos);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
UserInfo(userId=1, userName=Java)
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
```



### 12、query(String sql, RowMapper< T > rowMapper)

```java
public <T> List<T> query(String sql, RowMapper<T> rowMapper) throws DataAccessException {}
```

该方法返回一个RowMapper指定泛型对象的List集合，用户需要通过mapRow(ResultSet rs, int rowNum)实现方法手动将每一行转换为需要的类型，可以是Map也可以是自定义的对象。不支持动态传入参数。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 这个回调方式, 一行数据回调一次, 结果自动封装到List中, 可理解为 RowCallbackHandler + ResultSetExtractor
    String sql = "select * from user_info where user_id = 1";
    jdbcTemplate.query(sql, new RowMapper<UserInfo>() {
        @Override
        public UserInfo mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });

    // BeanPropertyRowMapper 为 RowMapper的子类, 可以直接转换为 Java Bean, 是 Spring 封装的.
    sql = "select * from user_info";
    userInfos = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(UserInfo.class));
    System.out.println(userInfos);

    // ColumnMapRowMapper 为 RowMapper的子类, 可以直接转换为 Map, 是 Spring 封装的.
    List<Map<String, Object>> userNames = jdbcTemplate.query(sql, new ColumnMapRowMapper());
    System.out.println(userNames);

    // 查询多行单列
    List<Integer> ids = jdbcTemplate.query(sql, (rs, rowNum) -> rs.getInt(1));
    System.out.println(ids);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
[UserInfo(userId=1, userName=Java)]
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
[1, 2, 3]
```



### 13、query(String sql, RowMapper< T > rowMapper, @Nullable Object... args)

```java
public <T> List<T> query(String sql, RowMapper<T> rowMapper, @Nullable Object... args) throws DataAccessException {}
```

- 第一个参数：需要执行的查询SQL。
- 第二个参数：查询结果的每一行结果集。
- 第三个参数：采用多参数组的形式，指SQL中占位符的参数值，没有则不写。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 这个回调方式, 一行数据回调一次, 结果自动封装到List中, 可理解为 RowCallbackHandler + ResultSetExtractor
    String sql = "select * from user_info where user_id = ?";
    List<UserInfo> userInfos = jdbcTemplate.query(sql, new RowMapper<UserInfo>() {
        @Override
        public UserInfo mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    }, 1);
    System.out.println(userInfos);

    // BeanPropertyRowMapper 为 RowMapper的子类, 可以直接转换为 Java Bean, 是 Spring 封装的.
    sql = "select * from user_info where user_id < ?";
    userInfos = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(UserInfo.class), 4);
    System.out.println(userInfos);

    // ColumnMapRowMapper 为 RowMapper的子类, 可以直接转换为 Map, 是 Spring 封装的.
    List<Map<String, Object>> userNames = jdbcTemplate.query(sql, new ColumnMapRowMapper(), 4);
    System.out.println(userNames);

    // 查询多行单列
    List<Integer> ids = jdbcTemplate.query(sql, (rs, rowNum) -> rs.getInt(1), 4);
    System.out.println(ids);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
[UserInfo(userId=1, userName=Java)]
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
[1, 2, 3]
```



### 14、query(String sql, Object[] args, int[] argTypes, RowMapper< T > rowMapper

```java
public <T> List<T> query(String sql, Object[] args, int[] argTypes, RowMapper<T> rowMapper) throws DataAccessException {}
```

- 第一个参数：需要执行的查询SQL。
- 第二个参数：有SQL中占位符参数构成的数组。如果没有则为传null。注意顺序保持一致。
- 第三个参数：参数值的数据库类型。如果没有则为传null。该参数可有可无。
- 第四个参数：查询结果的每一行结果集。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

        // 这个回调方式, 一行数据回调一次, 结果自动封装到List中, 可理解为 RowCallbackHandler + ResultSetExtractor
        Object[] param = {1};
        int[] index = {Types.INTEGER};
        String sql = "select * from user_info where user_id = ?";
        List<UserInfo> userInfos = jdbcTemplate.query(sql, param, index, new RowMapper<UserInfo>() {
            @Override
            public UserInfo mapRow(ResultSet rs, int rowNum) throws SQLException {
                return new UserInfo(rs.getInt(1), rs.getString(2));
            }
        });
        System.out.println(userInfos);

        // BeanPropertyRowMapper 为 RowMapper的子类, 可以直接转换为 Java Bean, 是 Spring 封装的.
        param[0] = 4;
        sql = "select * from user_info where user_id < ?";
        userInfos = jdbcTemplate.query(sql, param, index, new BeanPropertyRowMapper<>(UserInfo.class));
        System.out.println(userInfos);

        // ColumnMapRowMapper 为 RowMapper的子类, 可以直接转换为 Map, 是 Spring 封装的.
        List<Map<String, Object>> userNames = jdbcTemplate.query(sql, param, index, new ColumnMapRowMapper());
        System.out.println(userNames);

        // 查询多行单列
        List<Integer> ids = jdbcTemplate.query(sql, param, index, (rs, rowNum) -> rs.getInt(1));
        System.out.println(ids);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
[UserInfo(userId=1, userName=Java)]
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
[1, 2, 3]
```



### 15、query(String sql, @Nullable PreparedStatementSetter pss, RowMapper< T > rowMapper)

```java
public <T> List<T> query(String sql, @Nullable PreparedStatementSetter pss, RowMapper<T> rowMapper) throws DataAccessException {}
```

使用的是PreparedStatementSetter来支持动态的参数，需要处理SQL中的占位符。具体的处理方式都一样。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 这个回调方式, 一行数据回调一次, 结果自动封装到List中, 可理解为 RowCallbackHandler + ResultSetExtractor
    String sql = "select * from user_info where user_id = ?";
    List<UserInfo> userInfos = jdbcTemplate.query(sql, new PreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement ps) throws SQLException {
            ps.setInt(1, 1);
        }
    }, new RowMapper<UserInfo>() {
        @Override
        public UserInfo mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });
    System.out.println(userInfos);

    // BeanPropertyRowMapper 为 RowMapper的子类, 可以直接转换为 Java Bean, 是 Spring 封装的.
    sql = "select * from user_info where user_id < ?";
    userInfos = jdbcTemplate.query(sql,  ps -> ps.setInt(1, 4), new BeanPropertyRowMapper<>(UserInfo.class));
    System.out.println(userInfos);

    // ColumnMapRowMapper 为 RowMapper的子类, 可以直接转换为 Map, 是 Spring 封装的.
    List<Map<String, Object>> userNames = jdbcTemplate.query(sql, ps -> ps.setInt(1, 4), new ColumnMapRowMapper());
    System.out.println(userNames);

    // 查询多行单列
    List<Integer> ids = jdbcTemplate.query(sql, ps -> ps.setInt(1, 4), (rs, rowNum) -> rs.getInt(1));
    System.out.println(ids);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
[UserInfo(userId=1, userName=Java)]
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
[1, 2, 3]
```



### 16、query(PreparedStatementCreator psc, RowMapper< T > rowMapper)

```java
public <T> List<T> query(PreparedStatementCreator psc, RowMapper<T> rowMapper) throws DataAccessException {}
```

- 第一个参数：创建一个预编译的SQL，如果SQL中有占位符，则需要进行处理。
- 第二个参数：查询的每一行结果集，需要手动进行转换为想要的对象。
- 注意：这个方法和上面的几个query(rowMapper)方法返回的是List< T >对象。无论查询时单条记录还是多条记录，返回的都是List。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@PostConstruct
public void execute() {
    // 先初始化表和初始化数据
    jdbcTemplate.execute("drop table if exists user_info");
    jdbcTemplate.execute("create table user_info(user_id int primary key, user_name varchar(255))");
    jdbcTemplate.update("insert into user_info values(1, 'Java')");
    jdbcTemplate.update("insert into user_info values(2, 'Python')");
    jdbcTemplate.update("insert into user_info values(3, 'JavaScript')");

    // 这个回调方式, 一行数据回调一次, 结果自动封装到List中, 可理解为 RowCallbackHandler + ResultSetExtractor
    List<UserInfo> userInfos = jdbcTemplate.query(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            String sql = "select * from user_info where user_id = ?";
            PreparedStatement ps = con.prepareStatement(sql);
            ps.setInt(1, 1);
            return ps;
        }
    }, new RowMapper<UserInfo>() {
        @Override
        public UserInfo mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new UserInfo(rs.getInt(1), rs.getString(2));
        }
    });
    System.out.println(userInfos);

    // BeanPropertyRowMapper 为 RowMapper的子类, 可以直接转换为 Java Bean, 是 Spring 封装的.
    userInfos = jdbcTemplate.query(con -> {
        String sql = "select * from user_info where user_id < ?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setInt(1, 4);
        return ps;
    }, new BeanPropertyRowMapper<>(UserInfo.class));
    System.out.println(userInfos);

    // ColumnMapRowMapper 为 RowMapper的子类, 可以直接转换为 Map, 是 Spring 封装的.
    List<Map<String, Object>> userNames = jdbcTemplate.query(con -> {
        String sql = "select * from user_info where user_id < ?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setInt(1, 4);
        return ps;
    }, new ColumnMapRowMapper());
    System.out.println(userNames);

    // 查询多行单列
    List<Integer> ids = jdbcTemplate.query(con -> {
        String sql = "select * from user_info where user_id < ?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setInt(1, 4);
        return ps;
    }, (rs, rowNum) -> rs.getInt(1));
    System.out.println(ids);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public static class UserInfo {
    private int userId;
    private String userName;
}
```

```
[UserInfo(userId=1, userName=Java)]
[UserInfo(userId=1, userName=Java), UserInfo(userId=2, userName=Python), UserInfo(userId=3, userName=JavaScript)]
[{user_id=1, user_name=Java}, {user_id=2, user_name=Python}, {user_id=3, user_name=JavaScript}]
[1, 2, 3]
```



# 三、Junit 测试 ResultSetExtractor/RowMapper

> 作者：[YD_1989](https://blog.csdn.net/m0_58680865)、来源：Spring JdbcTemplate Junit 测试 - ResultSetExtractor/RowMapper：https://blog.csdn.net/m0_58680865/article/details/134429114

Spring JdbcTemplate Junit 测试覆盖率 - 以 ResultSetExtractor / RowMapper 为例

## 1、RowMapper Mockito 测试

1、创建实体类 User

```java
@Data
public class User {
    private Integer id;
    private String name;
    private String applicant;
    private String address;
    private Boolean flag;
}
```

2、JdbcTemplate 业务代码

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

@Repository
public class InsertGroup {

    @Autowired
    JdbcTemplate jdbcTemplate;

    public List<User> getUsers(String sql, String name, String address) {

        return jdbcTemplate.query(sql, new RowMapper<User>() {
            @Override
            public User mapRow(ResultSet rs, int rowNum) throws SQLException {

                User user = new User();
                user.setId(rs.getInt("ID"));
                user.setName(rs.getString("Name"));
                user.setApplicant(rs.getString("Applicant"));
                user.setAddress(rs.getString("Address"));
                user.setFlag(rs.getBoolean("Flag"));
                return user;
            }
        }, name, address);
    }
}
```

3、Junit Mockito 测试

```java
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.*;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class)
public class InsertGroupTest {

    @Mock
    JdbcTemplate jdbcTemplate;

    @InjectMocks
    InsertGroup insertGroup;

    @Before
    public void init() {
        MockitoAnnotations.openMocks(this);
    }


    @Test
    public void test() {

        /**
         * 需要注意的是：invocation.getArgument(1)
         * jdbcTemplate.query() 中 Mock 的参数索引是以0开始的，RowMapper 是第二个元素，因此索引是 1，如果是第三个位置，那么索引是 2，即 invocation.getArgument(2)
         * 同时需要注意的是，jdbcTemplate 中 query 的方法有很多，但是传的参数是不同的，因此 Mock 的参数数量要根据自己实际用到的 query 参数为准
         */
        Mockito.when(jdbcTemplate.query(ArgumentMatchers.anyString(), 
                                        ArgumentMatchers.any(RowMapper.class), 
                                        ArgumentMatchers.any()))
            .thenAnswer((invocation) -> {

                RowMapper<User> rowMapper = (RowMapper<User>) invocation.getArgument(1);
                ResultSet rs = Mockito.mock(ResultSet.class);

                // Mock ResultSet to return one rows.
                // Mockito.when(rs.getInt(ArgumentMatchers.eq("ID"))).thenReturn(506);

                // Mock ResultSet to return two rows.
                Mockito.when(rs.getInt(ArgumentMatchers.eq("ID"))).thenReturn(412, 300);
                Mockito.when(rs.getString(ArgumentMatchers.eq("Name"))).thenReturn("刘亦菲", "刘诗诗");
                Mockito.when(rs.getBoolean(ArgumentMatchers.eq("Flag"))).thenReturn(true, false);

                List<User> users = new ArrayList<>();
                users.add(rowMapper.mapRow(rs, 0));
                users.add(rowMapper.mapRow(rs, 1));

                return users;
            });

        List<User> userList = insertGroup.getUsers("sql", "1", "2");

        // Assert First Row
        assertFirstUser(userList.get(0));

        // Assert Second Row
        assertSecondUser(userList.get(1));
    }

    public void assertFirstUser(User user) {
        Assert.assertEquals(Integer.valueOf(412), user.getId());
        Assert.assertEquals("刘亦菲", user.getName());
        Assert.assertTrue(user.getFlag());
    }

    public void assertSecondUser(User user) {
        Assert.assertEquals(Integer.valueOf(300), user.getId());
        Assert.assertEquals("刘诗诗", user.getName());
        Assert.assertFalse(user.getFlag());
    }
}
```



## 2、ResultSetExtractor Mockito 测试

1、创建 User 实体

```java
@Data
public class User {
    private Integer id;
    private String name;
    private String applicant;
    private String address;
    private Boolean flag;
}
```



2、JdbcTemplate 业务代码

```java
public List<User> getUsers2(String sql, String name, String address) {

    return jdbcTemplate.query(sql, new ResultSetExtractor<List<User>>() {
        @Override
        public List<User> extractData(ResultSet rs) throws SQLException, DataAccessException {
            List<User> userList = new ArrayList<>();
            while (rs.next()) {
                User user = new User();
                user.setId(rs.getInt("ID"));
                user.setName(rs.getString("Name"));
                user.setApplicant(rs.getString("Applicant"));
                user.setAddress(rs.getString("Address"));
                user.setFlag(rs.getBoolean("Flag"));
                userList.add(user);
            }
            return userList;
        }
    }, name, address);
}
```



3、Junit Mockito 测试

```java
@Test
public void test2() {
    Mockito.when(jdbcTemplate.query(ArgumentMatchers.anyString(), 
                                    ArgumentMatchers.any(ResultSetExtractor.class), 
                                    ArgumentMatchers.any()))
        .thenAnswer((invocation) -> {
            ResultSetExtractor<List<User>> resultSetExtractor =
                (ResultSetExtractor<List<User>>) invocation.getArgument(1);

            ResultSet rs = Mockito.mock(ResultSet.class);

            // two times it returns true and third time returns false.
            Mockito.when(rs.next()).thenReturn(true, true, false);

            // Mock ResultSet to return two rows.
            Mockito.when(rs.getInt(ArgumentMatchers.eq("ID"))).thenReturn(412, 300);
            Mockito.when(rs.getString(ArgumentMatchers.eq("Name"))) .thenReturn("刘亦菲", "刘诗诗");
            Mockito.when(rs.getBoolean(ArgumentMatchers.eq("Flag"))).thenReturn(true, false);

            return resultSetExtractor.extractData(rs);
        });

    List<User> users = insertGroup.getUsers2("sql", "1", "2");

    Assert.assertEquals(Integer.valueOf(412), users.get(0).getId());
    Assert.assertEquals("刘亦菲", users.get(0).getName());
    Assert.assertTrue(users.get(0).getFlag());
}
```

# 四、JdbcTemplate 参考文献 & 鸣谢

1. Spring中JdbcTemplate各个方法的使用介绍（持续更新中....）：https://blog.csdn.net/whxjason/article/details/108949343
2. SpringBoot高级篇JdbcTemplate之数据查询上篇：https://mp.weixin.qq.com/s/SeN5q4g92LfHYOAVlQxytA
3. SpringBoot高级篇JdbcTemplate之数据查询下篇：https://mp.weixin.qq.com/s/laab3OyzTKL8mfR80Wy_QA
4. 【JavaWeb】73：JdbcTemplate竟然只能算是江南七怪级别的：https://mp.weixin.qq.com/s/qsVKW3wwbAYYi4zrrReL5A
5. Spring之JdbcTemplate使用：https://www.cnblogs.com/antLaddie/p/12859647.html
6. Spring JdbcTemplate Junit 测试 - ResultSetExtractor/RowMapper：https://blog.csdn.net/m0_58680865/article/details/134429114
