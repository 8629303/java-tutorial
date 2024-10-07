# SpringBoot 集成 HirakiCP 数据库连接池

在 Spring Boot 2.7.18 中，HikariCP 是默认的数据库连接池，你可以通过 application.yml 或 application.properties 文件进行配置。如果需要更详细的自定义配置，可以根据项目需求对 HikariCP 的参数进行微调。下面是最全的 HikariCP 数据库连接池集成配置详解。

## 1、引入依赖
首先，确保在你的 SpringBoot 项目中使用了合适的数据库和 JDBC 驱动。Maven 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```



## 2、配置 application.yml 文件

### 1、基本配置
默认情况下，Spring Boot 使用 HikariCP 作为连接池。如果你不需要特别的自定义，下面是基本的数据库连接配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/your_database
    username: your_username
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      minimum-idle: 5
      maximum-pool-size: 20
      idle-timeout: 600000  # 10 minutes
      pool-name: HikariPool
      max-lifetime: 1800000  # 30 minutes
      connection-timeout: 30000  # 30 seconds
      connection-test-query: SELECT 1
      auto-commit: true
```

- **minimum-idle**：最小空闲连接数，空闲连接池中保持的最小连接数。
- **maximum-pool-size**：最大连接数，连接池中维护的最大数据库连接数。
- **idle-timeout**：连接空闲的最长时间（毫秒）。超过这个时间的连接将会被释放。
- **pool-name**：为连接池设置一个自定义的名称。
- **max-lifetime**：每个连接在池中存活的最长时间（毫秒）。超过这个时间的连接将被关闭，新的连接将被创建。
- **connection-timeout**：等待连接的最长时间（毫秒）。超过这个时间还没有可用连接则抛出异常。
- **connection-test-query**：当连接需要验证时执行的 SQL 语句，确保连接有效。



### 2、高级配置

可以通过设置一些更高级的属性，来微调数据库连接池的性能：

```yaml
spring:
  datasource:
    hikari:
      leak-detection-threshold: 2000  # 连接泄露检测的阈值（毫秒），检测连接是否泄露
      allow-pool-suspension: true     # 允许池暂停，暂停时不会创建或返回新连接
      validation-timeout: 3000        # 验证连接的超时时间（毫秒）
      initialization-fail-timeout: 1  # 连接池初始化失败的超时时间
```

- **leak-detection-threshold**：设置连接泄露检测的阈值（以毫秒为单位）。当连接被借用时间超过这个值时，连接泄露警告将被记录。
- **allow-pool-suspension**：如果设置为 `true`，池可以被暂停和恢复。在暂停期间，调用 `getConnection()` 的线程会被阻塞，直到恢复。
- **validation-timeout**：验证连接的超时时间，超过这个时间连接的验证会失败。
- **initialization-fail-timeout**：初始化连接池时，如果连接失败，等待多长时间再抛出异常。如果设置为 `1`，将会立即抛出异常。



## 3、配置 application.properties 文件

如果使用 application.properties 文件，配置方法如下：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/your_database
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# HikariCP 配置
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.pool-name=HikariPool
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.connection-test-query=SELECT 1
spring.datasource.hikari.leak-detection-threshold=2000
spring.datasource.hikari.allow-pool-suspension=true
spring.datasource.hikari.validation-timeout=3000
spring.datasource.hikari.initialization-fail-timeout=1
```



## 4、自定义 HikariCP 配置类

如果你需要在 Java 配置中进一步自定义 HikariCP，可以通过自定义 DataSource Bean 来实现。

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/your_database");
        config.setUsername("your_username");
        config.setPassword("your_password");
        config.setDriverClassName("com.mysql.cj.jdbc.Driver");
        config.setMinimumIdle(5);
        config.setMaximumPoolSize(20);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        config.setConnectionTimeout(30000);
        config.setConnectionTestQuery("SELECT 1");
        config.setLeakDetectionThreshold(2000);
        return new HikariDataSource(config);
    }
}
```



## 5、监控和调优

为了监控数据库连接池的性能，Spring Boot 提供了对 HikariCP 的 Actuator 集成，可以通过 Actuator 监控连接池的状态。

在 application.yml 中启用 Actuator 端点：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: 'metrics'
  metrics:
    enable:
      hikari: true
```

然后，访问 `http://localhost:8080/actuator/metrics/hikari` 可以查看 HikariCP 的连接池状态。



## 6、常见问题与解决方案

- **连接池过小**：如果应用在高并发场景下性能不佳，可以考虑增加 maximum-pool-size。
- **连接泄露**：可以使用 leak-detection-threshold 来监测连接泄露问题，设置较短的阈值（如 2000 毫秒），并查看日志中是否出现泄露警告。

**总结：**

- HikariCP 是 Spring Boot 默认的数据库连接池，你可以通过 application.yml 或 application.properties 来配置各种参数。
- 自定义配置可以通过 Java 代码实现，以便根据应用的具体需求灵活调整。
- 通过 Actuator 监控 HikariCP 的状态，可以更好地掌握连接池的使用情况并进行调优。

通过合理的配置和监控，你可以确保 HikariCP 以最佳性能运行。



# SpringBoot 集成 Druid 数据库连接池

在 Spring Boot 2.7.18 中，集成 Druid 数据库连接池可以通过配置文件来进行，Druid 提供了强大的监控、SQL 执行统计、慢查询监控等功能。下面是 Druid 连接池的详细配置步骤和配置解释。

## 1、引入依赖

首先需要在项目的 pom.xml 中添加 Druid 的依赖：

示例 Maven 依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.15</version> <!-- 请确保使用的是最新版本 -->
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```



## 2、配置 application.yml

你可以通过 application.yml 配置文件来配置 Druid 连接池的各项参数。

基础配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/your_database
    username: your_username
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource  # 指定 Druid 数据源类型
    druid:
      initial-size: 5  # 初始化时建立的物理连接数
      min-idle: 5      # 最小空闲连接数
      max-active: 20   # 最大连接数
      max-wait: 60000  # 获取连接的最大等待时间（毫秒）
      time-between-eviction-runs-millis: 60000  # 间隔多久进行一次检测，空闲连接是否需要关闭
      min-evictable-idle-time-millis: 300000    # 连接空闲多久后被驱逐（毫秒）
      validation-query: SELECT 1 FROM DUAL      # 检测连接是否有效的 SQL
      test-while-idle: true  # 是否在空闲时检测连接
      test-on-borrow: false  # 是否在从连接池获取连接时检测
      test-on-return: false  # 是否在归还连接时检测
      pool-prepared-statements: true  # 是否缓存预编译语句
      max-pool-prepared-statement-per-connection-size: 20  # 每个连接的缓存预编译语句数
      filters: stat,wall,slf4j  # 监控统计、SQL 防火墙和日志输出的过滤器
      connection-properties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000  # SQL 合并统计和慢查询时间阈值

# Druid 的监控配置
druid:
  stat-view-servlet:
    enabled: true
    url-pattern: /druid/*
    login-username: admin     # 访问监控页面的用户名
    login-password: admin123  # 访问监控页面的密码
    reset-enable: false       # 禁止 HTML 页面上的 “Reset All” 功能
  filter:
    stat:
      log-slow-sql: true      # 打印慢 SQL
      slow-sql-millis: 5000   # 慢 SQL 阈值
```

配置解释：

- **initial-size**：初始化时建立的物理连接数。
- **min-idle**：最小空闲连接数。
- **max-active**：最大并发连接数。
- **max-wait**：最大等待时间，如果连接池满了，最多等待多久才能获取到连接，超过时间则抛出异常。
- **validation-query**：用来检测连接是否有效的 SQL 语句，`SELECT 1 FROM DUAL` 是 MySQL 的健康检查语句。
- **test-while-idle**：空闲时检测连接是否有效。
- **pool-prepared-statements**：是否开启 PSCache，即是否缓存 PreparedStatement。
- **max-pool-prepared-statement-per-connection-size**：指定每个连接上 PSCache 的大小。
- **filters**：Druid 提供的扩展功能，包括 `stat`（监控统计）、`wall`（SQL 防火墙）和 `slf4j`（日志输出）。
- **stat-view-servlet**：配置 Druid 内置的监控页面，默认地址为 `/druid/*`，可以通过账号和密码访问监控界面。
- **slow-sql-millis**：配置慢 SQL 的时间阈值，单位是毫秒。



## 3、配置 application.properties

如果使用 application.properties，你可以按以下方式配置：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/your_database
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

# Druid 连接池配置
spring.datasource.druid.initial-size=5
spring.datasource.druid.min-idle=5
spring.datasource.druid.max-active=20
spring.datasource.druid.max-wait=60000
spring.datasource.druid.time-between-eviction-runs-millis=60000
spring.datasource.druid.min-evictable-idle-time-millis=300000
spring.datasource.druid.validation-query=SELECT 1 FROM DUAL
spring.datasource.druid.test-while-idle=true
spring.datasource.druid.test-on-borrow=false
spring.datasource.druid.test-on-return=false
spring.datasource.druid.pool-prepared-statements=true
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
spring.datasource.druid.filters=stat,wall,slf4j
spring.datasource.druid.connection-properties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

# Druid 监控配置
druid.stat-view-servlet.enabled=true
druid.stat-view-servlet.url-pattern=/druid/*
druid.stat-view-servlet.login-username=admin
druid.stat-view-servlet.login-password=admin123
druid.stat-view-servlet.reset-enable=false
druid.filter.stat.log-slow-sql=true
druid.filter.stat.slow-sql-millis=5000
```



## 4、Druid 监控页面

Druid 提供了一个内置的监控页面，地址为 http://localhost:8080/druid/，你可以通过 application.yml 或 application.properties 中配置的用户名和密码进行访问。

监控页面可以显示：
- 连接池的状态信息。
- SQL 执行统计信息。
- 慢 SQL 查询日志。



## 5、自定义 Druid DataSource

如果你需要通过 Java 配置类来进一步自定义 Druid 连接池，可以使用以下配置方式：

```java
import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.sql.SQLException;

@Configuration
public class DruidConfig {

    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource datasource = new DruidDataSource();
        datasource.setUrl("jdbc:mysql://localhost:3306/your_database");
        datasource.setUsername("your_username");
        datasource.setPassword("your_password");
        datasource.setDriverClassName("com.mysql.cj.jdbc.Driver");

        // 配置初始化大小、最小、最大
        datasource.setInitialSize(5);
        datasource.setMinIdle(5);
        datasource.setMaxActive(20);
        datasource.setMaxWait(60000);

        // 配置检测空闲连接的线程和超时时间
        datasource.setTimeBetweenEvictionRunsMillis(60000);
        datasource.setMinEvictableIdleTimeMillis(300000);

        // 配置检验连接的 SQL
        datasource.setValidationQuery("SELECT 1 FROM DUAL");
        datasource.setTestWhileIdle(true);
        datasource.setTestOnBorrow(false);
        datasource.setTestOnReturn(false);

        // 配置监控统计功能
        datasource.setFilters("stat,wall,slf4j");

        return datasource;
    }
}
```



## 6、监控和调优

- **连接池监控**：通过 Druid 的监控页面实时查看连接池的运行情况，可以帮助你发现慢 SQL 和连接泄露等问题。
- **性能调优**：Druid 提供了详细的 SQL 统计和慢查询日志，可以根据这些信息优化你的 SQL 查询和数据库配置。



## 7、常见问题与解决方案

- **SQL 运行慢**：通过 Druid 的监控页面查看慢查询日志，并优化慢 SQL。
- **连接数过多或过少**：根据项目需求调整连接池的最大连接数 max-active 和最小空闲连接数 min-idle，以优化性能。

**总结：**

通过集成 Druid 数据库连接池，Spring Boot 项目可以获得强大的数据库连接管理和监控能力。你可以通过 application.yml 或 application.properties 文件配置，也可以通过 Java 配置类来自定义 Druid 的行为。同时，Druid 提供的内置监控页面有助于实时监控连接