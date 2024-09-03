> Mason技术记录：https://github.com/mianshenglee/my-example
>
> JdbcTemplate：
>
> - YourBatman博客：https://blog.csdn.net/f641385712/article/details/92404435
>
> Mybatis：
>
> - SpringBoot配置多数据源（动态切换，主从复制，读写分离）：https://blog.csdn.net/qq_39380737/article/details/103899713
> - SpringBoot+Mybatis-多数据源动态切换＋动态加载：https://blog.csdn.net/mengo1234/article/details/103167619
>
> 无头骑士：
>
> - SpringBoot+Hirika 实现动态数据源（AOP）物理千：https://mp.weixin.qq.com/s/n1dhtYBY-Mis8gV6N4Ck9Q
> - 动态数据源配置&运行时新增数据源：https://blog.95id.com/dynamic-datasource-in-springboot.html
>
> ***

> - Springboot 从数据库读取数据库配置信息，动态切换多数据源 最详细实战教程：https://blog.csdn.net/qq_35387940/article/details/102699765
> - Springboot 整合druid+mybatis+jta分布式事务+多数据源aop注解动态切换 （一篇到位）：https://blog.csdn.net/qq_35387940/article/details/103474353





# 1、SpringBoot 多数据源前言

## 1、使用场景

在开发过程中，避免不了需要同时操作多个数据库的情况，通常的应用场景如下 ：

- 同构或异构数据的业务处理：需要处理的数据存储在不同的数据库中，包括同构（如都是 MySQL ），异构（如一个MySQL ，另外是 PG 或者 Oracle ）。
- 数据库高性能场景：主从，包括一主一从，一主多从等，在主库进行增删改操作，在从库进行读操作。
- 数据库高可用场景：主备，包括一往一备，多主多备等，在数据库无法访问时可以切换。

使用 Spring Boot 该如何处理多个数据库的读写，一般有以下几种策略：

- 多套数据源：即针对一个数据库建立一套数据处理逻辑，每套数据库都包括数据源配置、会话工厂（ sessionFactory ）、连接、SQL 操作、实体。各套数据库相互独立。
- 动态数据源：确定数量的多个数据源共用一个会话工厂，根据条件动态选取数据源进行连接、SQL 操作。
- 参数化变更数据源：根据参数添加数据源，并进行数据源切换，数据源数量不确定。通常用于对多个数据库的管理工作





## 2、思路分析

- 要想配置多个数据源，必须新增配置文件。（你不写配置，系统怎么知道）
- 要有地方能够配置多数据源。（读出配置，告诉系统）
- 要能够选择用什么数据源。（系统知道有哪些，但是要我们自己指定用哪个）



## 3、实现方式

- 一种是静态数据源配置，配置好确定的数据源，想用哪个切换到哪个
- 另一种是动态数据源配置，我们可以运行时新增数据源以及切换数据源



## 4、事务分析

1. 事务的使用必须要指明是哪个数据源的事务管理器，否则事务失效。 例如：@Transactional(value = "userTransactionManager")
2. 如果一个方法涉及多个数据源操作，此时无法做到同时回滚，尽量避免这种操作，这里涉及到分布式事务处理，可自行研究。



# 2、SpringBoot 静态多数据源

多套数据源，顾名思义每一个数据库有一套独立的操作。从下往上，数据库、会话工厂、DAO操作，服务层都是独立的一套，如下所示：

```

```

本示例中，以一主一从两个数据库，两数据库的分别有一个表 `test_user`，表结构一致，为便于说明，两个表中的数据是不一样的。



## 1、JdbcTemplate + H2 静态多数据源配置

### 1、搭建项目工程

1、创建 SpringBoot 项目工程

2、添加项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>h2-jdbctemplate-multi-datasource-static</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>h2-jdbctemplate-multi-datasource-static</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```



### 2、多数据源配置

1、独立数据库连接信息配置 application.yml

```yaml
server:
  port: 8090
spring:
  #################### H2 数据库配置 ####################
  # 文件数据库
  # spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
  # 内存数据库，platform #表明使用的数据库平台是 h2
  datasource:
    master:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:master
      password: sa
      username: sa
      platform: h2
    slave:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:slave
      password: sa
      username: sa
      platform: h2
  #################### H2 Web Console设置 ####################
  ### enabled：程序开启时就会启动 H2 Web Console, 默认就是启动的
  ### path：配置访问地址为：/h2，访问 H2 Web Console，默认:/h2-console
  ### settings.web-allow-others：开启H2 Web Console远程访问，默认为false不开启只能在本机访问
  h2:
    console:
      enabled: true
      path: /h2
      settings:
        web-allow-others: true
```

2、SpringBoot 多套数据源配置

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {
    @Primary
    @Bean(name = "dataSourceMaster")
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource dataSourceMaster() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean(name = "dataSourceSlave")
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource dataSourceSlave() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Primary
    @Bean(name = "jdbcTemplateMaster")
    public JdbcTemplate jdbcTemplateMaster(@Qualifier("dataSourceMaster")DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean(name = "jdbcTemplateSlave")
    public JdbcTemplate jdbcTemplateSlave(@Qualifier("dataSourceSlave") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Primary
    @Bean(name = "platformTransactionManagerMaster")
    public PlatformTransactionManager platformTransactionManagerMaster() {
        return new DataSourceTransactionManager(dataSourceMaster());
    }

    @Bean(name = "platformTransactionManagerSlave")
    public PlatformTransactionManager platformTransactionManagerSlave() {
        return new DataSourceTransactionManager(dataSourceSlave());
    }
}
```



### 3、多数据源使用

1、经过上面的多套数据源配置，可知道，若需要操作哪个数据库，直接使用对应的 JdbcTemplate 进行 CRUD 操作即可。

```java
package com.example.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import java.util.List;
import java.util.Map;
import java.util.UUID;

@RestController
public class MultiController {

    @Autowired
    @Qualifier("jdbcTemplateMaster")
    JdbcTemplate jdbcTemplateMaster;
    @Resource(name = "jdbcTemplateSlave")
    JdbcTemplate jdbcTemplateSlave;

    /**
     * 数据Master: 创建表 USER_INFO, 并初始化数据
     */
    @PostConstruct
    public void init1() {
        jdbcTemplateMaster.execute("DROP TABLE IF EXISTS USER_INFO");
        jdbcTemplateMaster.execute("CREATE TABLE USER_INFO ("
                                   + "id VARCHAR(50) NOT NULL,"
                                   + "email VARCHAR(50) NOT NULL,"
                                   + "name VARCHAR(50) NOT NULL,"
                                   + "gender BOOLEAN NOT NULL,"
                                   + "createdAt BIGINT NOT NULL,"
                                   + "updatedAt BIGINT NOT NULL,"
                                   + "PRIMARY KEY (id))");
        String id = randomString();
        String email = randomString() + "@test.io";
        String name = "Mr " + randomString();
        boolean gender = randomString().hashCode() % 2 == 0;
        Long now = System.currentTimeMillis();
        String sql = "INSERT INTO USER_INFO (id, email, name, gender, createdAt, updatedAt) VALUES (?, ?, ?, ?, ?, ?)";
        jdbcTemplateMaster.update(sql, id, email, name, gender, now, now);
    }

    /**
     * 数据Slave: 创建表 USER_INFO, 并初始化数据
     */
    @PostConstruct
    public void init() {
        jdbcTemplateSlave.execute("DROP TABLE IF EXISTS BOOK_INFO");
        jdbcTemplateSlave.execute("CREATE TABLE BOOK_INFO ("
                                  + "id VARCHAR(50) NOT NULL,"
                                  + "name VARCHAR(50) NOT NULL,"
                                  + "isbn VARCHAR(50) NOT NULL,"
                                  + "createdAt BIGINT NOT NULL,"
                                  + "updatedAt BIGINT NOT NULL,"
                                  + "PRIMARY KEY (id))");
        String id = randomString();
        String name = "Java-" + randomString();
        String isbn = "ISBN-" + randomString().hashCode() % 2017;
        Long now = System.currentTimeMillis();
        String sql = "INSERT INTO BOOK_INFO (id, name, isbn, createdAt, updatedAt) VALUES (?, ?, ?, ?, ?)";
        jdbcTemplateSlave.update(sql, id, name, isbn, now, now);
    }

    private String randomString() {
        return UUID.randomUUID().toString();
    }

    @GetMapping("/api/users")
    public List<Map<String, Object>> getUsers2() {
        return jdbcTemplateMaster.queryForList("SELECT * FROM USER_INFO ORDER BY createdAt");
    }

    @GetMapping("/api/books")
    public List<Map<String, Object>> getBooks() {
        return jdbcTemplateSlave.queryForList("SELECT * FROM BOOK_INFO ORDER BY createdAt");
    }
}
```

2、测试主数据源

```bash
curl -X GET "localhost:8090/api/users" 
[
  {
    "ID": "9415439a-24a2-4563-8e7d-c9c08ea7794c",
    "EMAIL": "0425cbac-93a7-4872-a03b-2807d096c826@test.io",
    "NAME": "Mr d4a004a7-e673-4dbc-9c60-00ecc813966d",
    "GENDER": false,
    "CREATEDAT": 1724902823690,
    "UPDATEDAT": 1724902823690
  }
]
```

3、测试从数据源

```bash
curl -X GET "localhost:8090/api/books"
[
  {
    "ID": "77cd3fad-6642-4798-aca7-b5f755d0e072",
    "NAME": "Java-1ccf73bf-8d92-4abf-ab40-feb37897796c",
    "ISBN": "ISBN-1131",
    "CREATEDAT": 1724902823703,
    "UPDATEDAT": 1724902823703
  }
]
```





## 2、Mybatis + H2 静态多数据源配置

### 1、搭建项目工程

1、创建 SpringBoot 项目工程

2、添加项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>h2-mybaits-multi-datasource-static</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>h2-mybaits-multi-datasource-static</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```



### 2、多数据源配置

1、独立数据库连接信息配置 application.yml

```yaml
server:
  port: 8090
spring:
  #################### H2 数据库配置 ####################
  # 文件数据库
  # spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
  # 内存数据库，platform #表明使用的数据库平台是 h2
  datasource:
    master:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:master
      username: sa
      password:
    slave:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:slave
      username: sa
      password:
  #################### H2 Web Console设置 ####################
  ### enabled：程序开启时就会启动 H2 Web Console, 默认就是启动的
  ### path：配置访问地址为：/h2，访问 H2 Web Console，默认:/h2-console
  ### settings.web-allow-others：开启H2 Web Console远程访问，默认为false不开启只能在本机访问
  h2:
    console:
      enabled: true
      path: /h2
      settings:
        web-allow-others: true
```

2、SpringBoot 主数据源配置

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;

/**
 * @MapperScan.basePackages：扫描packages使用master数据源
 * @MapperScan.sqlSessionTemplateRef：注入sqlSessionTemplate
 */
@Configuration
@MapperScan(
        basePackages = "com.example.mapper.master",
        sqlSessionTemplateRef = "masterSqlSessionTemplate"
)
public class MasterDataSourceConfig {

    /**
     * 创建master主DataSource
     */
    @Primary
    @Bean(name = "masterDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    /**
     * 创建sqlSessionFactory
     */
    @Primary
    @Bean(name = "masterSqlSessionFactory")
    public SqlSessionFactory masterSqlSessionFactory(@Qualifier("masterDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        // 如果需要使用xml配置文件的形式的话，那么需要扫描XML文件的存放路径
        // String locationPattern = "classpath:mapper/master/*.xml";
        // bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(locationPattern));
        return bean.getObject();
    }

    /**
     * 创建sqlSession模板
     */
    @Primary
    @Bean(name = "masterSqlSessionTemplate")
    public SqlSessionTemplate masterSqlSessionTemplate(@Qualifier("masterSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    /**
     * 创建事务管理器：需要使用事务的时候，加上@Transactional("masterTransactionManager")
     */
    @Primary
    @Bean(name = "masterTransactionManager")
    public PlatformTransactionManager masterTransactionManager(@Qualifier("masterDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

}
```

3、SpringBoot 从数据源配置

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;

@Configuration
@MapperScan(
        basePackages = "com.example.mapper.slave",
        sqlSessionTemplateRef = "slaveSqlSessionTemplate"
)
public class SlaveDataSourceConfig {
    /**
     * 设置XML文件存放位置，扫描路径
     */
    private final String locationPattern = "classpath:mapper/slave/*.xml";

    /**
     * 创建salve主DataSource
     */
    @Bean(name = "slaveDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    /**
     * 创建sqlSessionFactory
     */
    @Bean(name = "slaveSqlSessionFactory")
    public SqlSessionFactory slaveSqlSessionFactory(@Qualifier("slaveDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        // 如果需要使用xml配置文件的形式的话，那么需要扫描XML文件的存放路径
        // bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(locationPattern));
        return bean.getObject();
    }

    /**
     * 创建sqlSession模板
     */
    @Bean(name = "slaveSqlSessionTemplate")
    public SqlSessionTemplate slaveSqlSessionTemplate(@Qualifier("slaveSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    /**
     * 创建事务管理器：需要使用事务的时候，加上@Transactional("slaveTransactionManager")
     */
    @Bean(name = "slaveTransactionManager")
    public PlatformTransactionManager slaveTransactionManager(@Qualifier("slaveDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

}
```

4、数据表对应实体类的创建

```java
package com.example.model;

import lombok.Data;
import java.io.Serializable;

@Data
public class UserInfo implements Serializable {
    private Integer Id;
    private String name;
    private String type;
}
```

5、主数据源Mapper的创建

```java
package com.example.mapper.master;

import com.example.model.UserInfo;
import org.apache.ibatis.annotations.*;

import java.util.List;

public interface MasterUserMapper {
    @Select("select * from user_info")
    List<UserInfo> masterFindAllUser();

    @Insert("insert into user_info (name,type) values (#{name},'master');")
    int masterSaveUser(@Param("name") String name);

    @Update("drop table if exists user_info;")
    void dropUserExistTable();

    @Update("create TABLE user_info (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))")
    void createUserTable();

    @Insert("insert into user_info (id, name, type) values (1, 'Jone', 'master'),(2, 'Kath', 'master'),(3, 'Tom', 'master'),(4, 'Sandy', 'master'),(5, 'Oliver', 'master');")
    int initTableData();
}
```

5、从数据源Mapper的创建

```java
package com.example.mapper.slave;

import com.example.model.UserInfo;
import org.apache.ibatis.annotations.*;

import java.util.List;

public interface SlaveUserMapper {
    @Select("select * from user_info")
    List<UserInfo> slaveFindAllUser();

    @Insert("insert into user_info (name,type) values (#{name},'master');")
    int slaveSaveUser(@Param("name") String name);

    @Update("drop table if exists user_info;")
    void dropUserExistTable();

    @Update("create table user_info (id BIGINT  NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))")
    void createUserTable();

    @Insert("insert into user_info (id, name, type) values (1, 'Jone', 'slave'),(2, 'Kath', 'slave'),(3, 'Tom', 'slave'),(4, 'Sandy', 'slave'),(5, 'Oliver', 'slave');")
    int initTableData();
}
```



### 3、多数据源使用

1、经过上面的多套数据源配置，可知道，若需要操作哪个数据库，直接使用对应的 mapper 进行 CRUD 操作即可。

```java
package com.example.controller;

import com.example.mapper.master.MasterUserMapper;
import com.example.mapper.slave.SlaveUserMapper;
import com.example.model.UserInfo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import java.util.List;
import java.util.UUID;

@Slf4j
@RestController
public class UserController {
    @Resource
    MasterUserMapper masterUserMapper;
    @Resource
    SlaveUserMapper slaveUserMapper;

    @PostConstruct
    public void init() {
        log.info("-----------------初始化数据开始--------------------");
        // 初始化master表和数据
        masterUserMapper.dropUserExistTable();
        masterUserMapper.createUserTable();
        masterUserMapper.initTableData();
        // 初始化slave表和数据
        slaveUserMapper.dropUserExistTable();
        slaveUserMapper.createUserTable();
        slaveUserMapper.initTableData();
        log.info("-----------------初始化数据结束--------------------");

        log.info("============查询主数据源数据===============");
        masterUserMapper.masterFindAllUser().forEach(System.out::println);
        log.info("============查询从数据源数据===============");
        slaveUserMapper.slaveFindAllUser().forEach(System.out::println);
    }

    @GetMapping("/masterFindAllUser")
    public ResponseEntity<List<UserInfo>> masterFindAllUser() {
        return ResponseEntity.ok(masterUserMapper.masterFindAllUser());
    }

    @GetMapping("/slaveFindAllUser")
    public ResponseEntity<List<UserInfo>> slaveFindAllUser() {
        return ResponseEntity.ok(slaveUserMapper.slaveFindAllUser());
    }

    @Transactional(rollbackFor = Exception.class, transactionManager = "masterTransactionManager")
    @GetMapping("/saveMasterUser")
    public ResponseEntity<String> saveMasterUser(Integer number) {
        String response = masterUserMapper.masterSaveUser(UUID.randomUUID().toString()) > 0 ? "success" : "fail";
        // 如果number=0，则会报错，插入数据回滚
        System.out.println(10 / number);
        return ResponseEntity.ok(response);
    }

    @Transactional(rollbackFor = Exception.class, transactionManager = "slaveTransactionManager")
    @GetMapping("/saveSalveUser")
    public ResponseEntity<String> saveSalveUser(Integer number) {
        String response = slaveUserMapper.slaveSaveUser(UUID.randomUUID().toString()) > 0 ? "success" : "fail";
        // 如果number=0，则会报错，插入数据回滚
        System.out.println(10 / number);
        return ResponseEntity.ok(response);
    }
}
```

2、项目启动就会执行 @PostConstruct 中的代码，进行数据初始化，我们可以查看日志

```
2024-08-29 14:20:09.428  INFO 85224 --- [           main] com.example.controller.UserController    : -----------------初始化数据开始--------------------
2024-08-29 14:20:09.449  INFO 85224 --- [           main] com.example.controller.UserController    : -----------------初始化数据结束--------------------
2024-08-29 14:20:09.449  INFO 85224 --- [           main] com.example.controller.UserController    : ============查询主数据源数据===============
UserInfo(Id=1, name=Jone, type=master)
UserInfo(Id=2, name=Kath, type=master)
UserInfo(Id=3, name=Tom, type=master)
UserInfo(Id=4, name=Sandy, type=master)
UserInfo(Id=5, name=Oliver, type=master)
2024-08-29 14:20:09.476  INFO 85224 --- [           main] com.example.controller.UserController    : ============查询从数据源数据===============
UserInfo(Id=1, name=Jone, type=slave)
UserInfo(Id=2, name=Kath, type=slave)
UserInfo(Id=3, name=Tom, type=slave)
UserInfo(Id=4, name=Sandy, type=slave)
UserInfo(Id=5, name=Oliver, type=slave)
```

3、测试查询主数据源的 API

```bash
curl -X GET "localhost:8090/masterFindAllUser"
[
  {
    "name": "Jone",
    "type": "master",
    "id": 1
  },
  {
    "name": "Kath",
    "type": "master",
    "id": 2
  },
  {
    "name": "Tom",
    "type": "master",
    "id": 3
  },
  {
    "name": "Sandy",
    "type": "master",
    "id": 4
  },
  {
    "name": "Oliver",
    "type": "master",
    "id": 5
  }
]
```

4、测试查询从数据源的 API

```bash
curl -X GET "localhost:8090/slaveFindAllUser"
[
  {
    "name": "Jone",
    "type": "slave",
    "id": 1
  },
  {
    "name": "Kath",
    "type": "slave",
    "id": 2
  },
  {
    "name": "Tom",
    "type": "slave",
    "id": 3
  },
  {
    "name": "Sandy",
    "type": "slave",
    "id": 4
  },
  {
    "name": "Oliver",
    "type": "slave",
    "id": 5
  }
]
```



## 3、Spring Data JPA + H2 静态多数据源配置

### 1、搭建项目工程

1、创建 SpringBoot 项目工程

2、添加项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>h2-jpa-multi-datasource-static</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>h2-jpa-multi-datasource-static</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```



### 2、多数据源配置

1、独立数据库连接信息配置 application.yml

```yaml
server:
  port: 8090
spring:
  #################### H2 数据库配置 ####################
  # 文件数据库
  # spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
  # 内存数据库，platform #表明使用的数据库平台是 h2
  datasource:
    master:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:master
      password: sa
      username: sa
      platform: h2
    slave:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:slave
      password: sa
      username: sa
      platform: h2
  #################### H2 Web Console设置 ####################
  ### enabled：程序开启时就会启动 H2 Web Console, 默认就是启动的
  ### path：配置访问地址为：/h2，访问 H2 Web Console，默认:/h2-console
  ### settings.web-allow-others：开启H2 Web Console远程访问，默认为false不开启只能在本机访问
  h2:
    console:
      enabled: true
      path: /h2
      settings:
        web-allow-others: true
  #################### JPA 的配置 ####################
  # database-platform: 数据库方言的配置
  # hibernate.ddl-auto = none, 不启动 jpa 的自动通过实体类自动建表功能
  # show-sql = true 开启 SQL 打印功能
  # properties.hibernate.format_sql = true 开启SQL格式化功能
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        format_sql: true
    show-sql: true
```



2、SpringBoot 主数据配置。步骤：创建DataSource—》创建EntityManagerFactory—》创建TransactionManager—》开启JPA

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateProperties;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateSettings;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.annotation.Resource;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
import java.util.Map;

/**
 * 配置顺序如下：
 *   创建DataSource—》创建EntityManagerFactory—》创建TransactionManager—》@EnableJpaRepositories开启JPA
 *
 * 在多数据源配置中还是需要手动开启事务 @EnableTransactionManagement，并受用 @Primary 注解指定了主要的事务管理器
 * 利⽤ EnableJpaRepositories 配置哪些包下⾯的 Repositories，采⽤哪个 EntityManagerFactory 和哪个 TransactionManagement
 */
@Configuration
@EnableTransactionManagement  // 开启事务
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactoryMaster",  // 数据源的 repository 的包路径
        transactionManagerRef="transactionManagerMaster",  // 实体管理器工厂
        basePackages= { "com.example.repository.master"})  // 扫描Repository所在位置
public class JPAConfigMaster {
    @Resource
    private JpaProperties jpaProperties;

    @Resource
    private HibernateProperties hibernateProperties;

    @Primary
    @Bean(name = "masterDataSource")
    @ConfigurationProperties(prefix="spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    /**
     * primary实体工厂
     */
    @Primary
    @Bean(name = "entityManagerFactoryMaster")
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryMaster(EntityManagerFactoryBuilder builder) {
        // 可以使用注入 EntityManagerFactoryBuilder 或者直接 new 来创建 LocalContainerEntityManagerFactoryBean
        // LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
        Map<String,Object> properties = hibernateProperties.determineHibernateProperties(jpaProperties.getProperties(), new HibernateSettings());

        return builder.dataSource(masterDataSource())
                .properties(properties)
                // 扫描的实体类所在位置
                .packages("com.example.model")
                // jpa定义的名称
                .persistenceUnit("masterPersistenceUnit")
                .build();
    }

    /**
     * master实体管理器
     */
    @Primary
    @Bean(name = "entityManagerMaster")
    public EntityManager entityManagerMaster(EntityManagerFactoryBuilder builder) {
        return entityManagerFactoryMaster(builder).getObject().createEntityManager();
    }

    /**
     * master事务管理器
     */
    @Primary
    @Bean(name = "transactionManagerMaster")
    public PlatformTransactionManager transactionManagerMaster(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactoryMaster(builder).getObject());
    }
}
```

3、SpringBoot 从数据配置。步骤：创建DataSource—》创建EntityManagerFactory—》创建TransactionManager—》开启JPA

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateProperties;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateSettings;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.annotation.Resource;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
import java.util.Map;

/**
 * 配置顺序如下：
 *  创建DataSource—》创建EntityManagerFactory—》创建TransactionManager—》@EnableJpaRepositories开启JPA
 * 利⽤ EnableJpaRepositories 配置哪些包下⾯的 Repositories，采⽤哪个 EntityManagerFactory 和哪个 TransactionManagement
 */
@Configuration
@EnableTransactionManagement  // 开启事务
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactorySlave",  // 数据源的 repository 的包路径
        transactionManagerRef="transactionManagerSlave",  // 实体管理器工厂
        basePackages= { "com.example.repository.slave"})  // 扫描Repository所在位置
public class JPAConfigSlave {
    @Resource
    private JpaProperties jpaProperties;

    @Resource
    private HibernateProperties hibernateProperties;

    @Bean(name = "slaveDataSource")
    @ConfigurationProperties(prefix="spring.datasource.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    /**
     * Slave实体工厂
     */
    @Bean(name = "entityManagerFactorySlave")
    public LocalContainerEntityManagerFactoryBean entityManagerFactorySlave(EntityManagerFactoryBuilder builder) {

        Map<String,Object> properties = hibernateProperties.determineHibernateProperties(jpaProperties.getProperties(), new HibernateSettings());

        return builder.dataSource(slaveDataSource())
                .properties(properties)
                .packages("com.example.model")
                .persistenceUnit("slavePersistenceUnit")
                .build();
    }

    /**
     * Slave实体管理器
     */
    @Bean(name = "entityManagerSlave")
    public EntityManager entityManagerSlave(EntityManagerFactoryBuilder builder) {
        return entityManagerFactorySlave(builder).getObject().createEntityManager();
    }

    /**
     * Slave事务管理器
     */
    @Bean(name = "transactionManagerSlave")
    public PlatformTransactionManager transactionManagerSlave(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactorySlave(builder).getObject());
    }
}
```

4、数据表对应实体类的创建

- 创建主数据源对应的实体类

  ```java
  package com.example.model.master;
  
  import lombok.Data;
  import javax.persistence.Entity;
  import javax.persistence.GeneratedValue;
  import javax.persistence.Id;
  import javax.persistence.Table;
  
  @Data
  @Entity
  @Table(name = "USER_INFO")
  public class UserInfoMaster {
      @Id
      @GeneratedValue
      private Integer Id;
      private String name;
      private String type;
  }
  ```

- 创建从数据源对应的实体类

  ```java
  package com.example.model.slave;
  
  import lombok.Data;
  import javax.persistence.Entity;
  import javax.persistence.GeneratedValue;
  import javax.persistence.Id;
  import javax.persistence.Table;
  
  @Data
  @Entity
  @Table(name = "USER_INFO")
  public class UserInfoSlave {
      @Id
      @GeneratedValue
      private Integer Id;
      private String name;
      private String type;
  }
  ```

- 如果主从数据源中的表子段和类型一模一样可以使用同一个实体类

  ```java
  package com.example.model;
  
  import lombok.AllArgsConstructor;
  import lombok.Builder;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  import javax.persistence.Entity;
  import javax.persistence.GeneratedValue;
  import javax.persistence.Id;
  import javax.persistence.Table;
  
  @Data
  @Builder
  @AllArgsConstructor
  @NoArgsConstructor
  @Entity
  @Table(name = "USER_INFO")
  public class UserInfo {
      @Id
      @GeneratedValue
      private Integer Id;
      private String name;
      private String type;
  }
  ```



5、主数据源Repository的创建

```java
package com.example.repository.master;

import com.example.model.UserInfo;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.transaction.annotation.Transactional;

public interface UserRepositoryMaster extends JpaRepository<UserInfo, Integer> {
    @Transactional
    @Query(value = "drop table if exists USER_INFO;",nativeQuery=true)
    @Modifying
    void dropUserExistTable();

    @Transactional
    @Query( value= "create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))",nativeQuery=true)
    @Modifying
    void createUserTable();

    @Transactional
    @Query(value="insert into USER_INFO (id, name, type) values (1, 'Jone', 'master'),(2, 'Kath', 'master'),(3, 'Tom', 'master'),(4, 'Sandy', 'master'),(5, 'Oliver', 'master');",nativeQuery=true)
    @Modifying
    int initTableData();
}
```

6、从数据源Repository的创建

```java
package com.example.repository.slave;

import com.example.model.UserInfo;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.transaction.annotation.Transactional;

public interface UserRepositorySlave extends JpaRepository<UserInfo, Integer> {
    @Transactional
    @Query(value = "drop table if exists USER_INFO;",nativeQuery=true)
    @Modifying
    void dropUserExistTable();

    @Transactional
    @Query( value= "create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))",nativeQuery=true)
    @Modifying
    void createUserTable();

    @Transactional
    @Query(value="insert into USER_INFO (id, name, type) values (1, 'Jone', 'slave'),(2, 'Kath', 'slave'),(3, 'Tom', 'slave'),(4, 'Sandy', 'slave'),(5, 'Oliver', 'slave');",nativeQuery=true)
    @Modifying
    int initTableData();
}
```



### 3、多数据源使用

1、经过上面的多套数据源配置，可知道，若需要操作哪个数据库，直接使用对应的 repository 进行 CRUD 操作即可。

```java
package com.example.controller;

import com.example.repository.master.UserRepositoryMaster;
import com.example.repository.slave.UserRepositorySlave;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;

@Slf4j
@RestController
public class UserController {
    @Resource
    UserRepositoryMaster master;
    @Resource
    UserRepositorySlave slave;
    @PostConstruct
    public void init(){
        log.info("-----------------初始化数据开始--------------------");
        // 初始化master数据源
        master.dropUserExistTable();
        master.createUserTable();
        master.initTableData();
        // 初始化slave数据源，
        slave.dropUserExistTable();
        slave.createUserTable();
        slave.initTableData();
        log.info("-----------------初始化数据结束--------------------");

        // 查询master初始化的数据
        log.info("============查询主数据源数据===============");
        master.findAll().forEach(System.out::println);
        // 查询slave初始化的数据
        log.info("============查询从数据源数据===============");
        slave.findAll().forEach(System.out::println);
    }

    @GetMapping("/masterFindAllUser")
    public ResponseEntity masterFindAllUser(){
        return ResponseEntity.ok(master.findAll());
    }

    @GetMapping("/slaveFindAllUser")
    public ResponseEntity slaveFindAllUser(){
        return ResponseEntity.ok(slave.findAll());
    }
}
```

2、项目启动就会执行 @PostConstruct 中的代码，进行数据初始化，我们可以查看日志

```
2024-08-29 15:09:05.556  INFO 88210 --- [  restartedMain] com.example.controller.UserController    : -----------------初始化数据开始--------------------
2024-08-29 15:09:05.643  INFO 88210 --- [  restartedMain] com.example.controller.UserController    : -----------------初始化数据结束--------------------
2024-08-29 15:09:05.643  INFO 88210 --- [  restartedMain] com.example.controller.UserController    : ============查询主数据源数据===============
UserInfo(Id=1, name=Jone, type=master)
UserInfo(Id=2, name=Kath, type=master)
UserInfo(Id=3, name=Tom, type=master)
UserInfo(Id=4, name=Sandy, type=master)
UserInfo(Id=5, name=Oliver, type=master)
2024-08-29 15:09:05.693  INFO 88210 --- [  restartedMain] com.example.controller.UserController    : ============查询从数据源数据===============
UserInfo(Id=1, name=Jone, type=slave)
UserInfo(Id=2, name=Kath, type=slave)
UserInfo(Id=3, name=Tom, type=slave)
UserInfo(Id=4, name=Sandy, type=slave)
UserInfo(Id=5, name=Oliver, type=slave)
```

3、测试查询主数据源的 API

```bash
curl -X GET "localhost:8090/masterFindAllUser"
[
  {
    "name": "Jone",
    "type": "master",
    "id": 1
  },
  {
    "name": "Kath",
    "type": "master",
    "id": 2
  },
  {
    "name": "Tom",
    "type": "master",
    "id": 3
  },
  {
    "name": "Sandy",
    "type": "master",
    "id": 4
  },
  {
    "name": "Oliver",
    "type": "master",
    "id": 5
  }
]
```

4、测试查询从数据源的 API

```bash
curl -X GET "localhost:8090/slaveFindAllUser" 
[
  {
    "name": "Jone",
    "type": "slave",
    "id": 1
  },
  {
    "name": "Kath",
    "type": "slave",
    "id": 2
  },
  {
    "name": "Tom",
    "type": "slave",
    "id": 3
  },
  {
    "name": "Sandy",
    "type": "slave",
    "id": 4
  },
  {
    "name": "Oliver",
    "type": "slave",
    "id": 5
  }
]
```



## 4、静态多数据源优缺点

1、优点

- 简单、直接：一个库对应一套处理方式，很好理解。
- 符合开闭原则（ OCP ）：开发的设计模式告诉我们，对扩展开放，对修改关闭，添加多一个数据库，原来的那一套不需要改动，只添加即可。

2、缺点

- 资源浪费：针对每一个数据源写一套操作，连接数据库的资源也是独立的，分别占用同样多的资源。`SqlSessionFactory` 是一个工厂，建议是使用单例，完全可以重用，不需要建立多个，只需要更改数据源即可，跟多线程，使用线程池减少资源消耗是同一道理。
- 代码冗余：在前面的多数据源配置中可以看出，其实 master 和 slave 的很多操作是一样的，只是改个名称而已，因此会造成代码冗余。
- 缺乏灵活：所有需要使用的地方都需要引入对应的mapper，对于很多操作，只是选择数据源的不一样，代码逻辑是一致的。另外，对于一主多从的情况，若需要对多个从库进行负载均衡，相对比较麻烦。

正因为有上述的缺点，所以还有改进的空间。于是就有了动态数据源，至于动态数据源如何实现，下回分解。



# 3、SpringBoot 动态多数据源

动态数据源可以解决多套数据源的处理不够灵活、占用资源多等问题。用户可以根据实际的业务需要，统一操作逻辑，只要在需要切换数据源的进行处理即可。何为动态，其实是批切换数据源的时机可以动态选择，在需要的地方进行切换即可。

Spring Boot 的动态数据源，本质上是把多个数据源存储在一个 Map 中，当需要使用某个数据源时，从 Map 中获取此数据源进行处理。而在 Spring 中，已提供了抽象类 `AbstractRoutingDataSource` 来实现此功能。因此，我们在实现动态数据源的，只需要继承它，实现自己的获取数据源逻辑即可。动态数据源流程如下所示：

```

```

用户访问应用，在需要访问不同的数据源时，根据自己的数据源路由逻辑，访问不同的数据源，实现对应数据源的操作。



## 1、JdbcTemplate + H2 静态多数据源配置



### 1、动态数据源配置

1、添加项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>h2-jdbctemplate-multi-datasource-dynamic</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>h2-jdbctemplate-multi-datasource-dynamic</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>   
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

2、添加以下自定义的主从数据库配置 application.yml，这里暂时值定义了2个数据源，后面可以通过代码逻辑手动添加新的数据源

```yaml
server:
  port: 8090
spring:
  #################### H2 数据库配置 ####################
  # 文件数据库
  # spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
  # 内存数据库，platform #表明使用的数据库平台是 h2
  datasource:
    master:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:master
      password: sa
      username: sa
      platform: h2
    slave:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:slave
      password: sa
      username: sa
      platform: h2
  #################### H2 Web Console设置 ####################
  ### enabled：程序开启时就会启动 H2 Web Console, 默认就是启动的
  ### path：配置访问地址为：/h2，访问 H2 Web Console，默认:/h2-console
  ### settings.web-allow-others：开启H2 Web Console远程访问，默认为false不开启只能在本机访问
  h2:
    console:
      enabled: true
      path: /h2
      settings:
        web-allow-others: true
```



### 2、动态数据源设置

1、 多数据源配置类：DataSourceConfig

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

/**
 * @EnableAutoConfiguration 就是去掉springboot默认自动配置数据源
 */
@Configuration
@EnableAutoConfiguration(exclude = { DataSourceAutoConfiguration.class })
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource dataSourceMaster() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource dataSourceSlave() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    /**
     * 使用 Map 保存多个数据源，并设置到动态数据源对象中。
     * 设置默认的数据源是 master 数据源
     * 使用注解 Primary 优先从动态数据源中获取
     * 注意: 这个才是主数据源
     */
    @Primary
    @Bean
    public DynamicDataSource dynamicDataSource(){
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put("default",dataSourceMaster());
        targetDataSources.put("slave_1", dataSourceSlave());
        // 设置默认数据源
        dynamicDataSource.setDefaultTargetDataSource(dataSourceMaster());
        // 添加数据源集合
        dynamicDataSource.setTargetDataSources(targetDataSources);
        return dynamicDataSource;
    }

    /**
     * 事务管理
     */
    @Bean(name = "sqlTransactionManager")
    public PlatformTransactionManager platformTransactionManager() {
        return new DataSourceTransactionManager(dynamicDataSource());
    }

    /**
     * 初始化JdbcTemplate
     */
    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dynamicDataSource());
    }
}
```

2、动态数据设置以及获取类：实现 AbstractRoutingDataSource 的方法，使用 DynamicDataSourceContextHolder 统一数据源管理

```java
package com.example.config;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import java.util.HashMap;
import java.util.Map;

public class DynamicDataSource extends AbstractRoutingDataSource {
    /**
     * 添加一个DataSources Map，用来获取数据源集合
     */
    private final Map<Object, Object> targetDataSources = new HashMap<>();
    public Map<Object, Object> getTargetDataSources() {
        return targetDataSources;
    }

    /**
     * 重写setTargetDataSources方法，为了可以让自己实现动态添加数据源
     * @param targetDataSources
     */
    @Override
    public void setTargetDataSources(Map<Object, Object> targetDataSources) {
        this.targetDataSources.putAll(targetDataSources);
        super.setTargetDataSources(this.targetDataSources);
        super.afterPropertiesSet(); // 必须添加该句，否则新添加数据源无法识别到
    }

    /**
     * 必须实现该接口。用来获取当前数据源
     */
    @Override
    protected Object determineCurrentLookupKey() {
        // 或者return null.细节看源码AbstractRoutingDataSource.determineTargetDataSource
        return DynamicDataSourceContextHolder.getDataSource();
    }
}
```

3、DynamicDataSourceContextHolder 类的实现，通过 ThreadLocal 获取和设置线程安全的数据源 key

```java
package com.example.config;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class DynamicDataSourceContextHolder {
    /**
     * 动态数据源名称上下文
     */
    private static final ThreadLocal<String> contextLocal = ThreadLocal.withInitial(() -> "default");
    // private static final ThreadLocal<String> contextLocal = new ThreadLocal<>();

    /**
     * 设置数据源
     */
    public static void setDataSource(String key) {
        log.info("-----------切换数据源：" + key + "---------------");
        contextLocal.set(key);
    }

    /**
     * 获取数据源
     */
    public static String getDataSource() {
        String key = contextLocal.get();
        log.info("-----------当前数据源：" + key + "---------------");
        return key;
    }

    /**
     * 重置/释放数据源
     */
    public static void clearDataSource() {
        contextLocal.remove();
        String key = contextLocal.get();
        log.info("-----------重置数据源：" + key + "---------------");
    }
}
```



### 3、动态数据的使用【手动切换】

1、动态数据源使用，这里编写一个 Controller，里面有一个动态初始化数据源接口，一个初始化每个数据源的表和数据的接口，另外还有3个查询不同数据源中数据的接口

```java
package com.example.controller;

import com.example.config.DynamicDataSource;
import com.example.config.DynamicDataSourceContextHolder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;
import javax.sql.DataSource;
import java.util.List;
import java.util.Map;
import java.util.stream.Stream;

@Slf4j
@RestController
public class MultiController {

    @Resource
    JdbcTemplate jdbcTemplate;
    @Resource
    DynamicDataSource dynamicDataSource;

    /**
     * 动态数据源配置 & 运行时新增数据源
     * 可以从数据库中加载, 也可以通过API新增
     */
    @RequestMapping("initDataSource")
    public String initDataSource() {
        log.info("===============初始化动态数据源===============");
        Map<Object, Object> dataSourceMap = dynamicDataSource.getTargetDataSources();
        Stream.of(1, 2, 3).forEach(i -> {
            DataSource build = DataSourceBuilder.create()
                    .username("sa")
                    .password("sa")
                    .url("jdbc:h2:mem:test_" + i)
                    .build();
            dataSourceMap.put("test_" + i, build);
        });
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        log.info("===============动态数据源初始完毕===============");
        return "init dataSource success";
    }

    /**
     * 给新增的数据源创建表和初始化数据
     */
    @RequestMapping("initData")
    public String initData() {
        Stream.of(1, 2, 3).forEach(i -> {
            // 切换数据源
            DynamicDataSourceContextHolder.setDataSource("test_" + i);
            // 初始化表和数据
            jdbcTemplate.execute("DROP TABLE IF EXISTS USER_INFO");
            jdbcTemplate.execute("CREATE TABLE USER_INFO ("
                    + "id BIGINT NOT NULL,"
                    + "name VARCHAR(50) DEFAULT NULL,"
                    + "type VARCHAR(50) DEFAULT NULL,"
                    + "PRIMARY KEY (id))");
            String sql = "INSERT INTO USER_INFO (id, name, type) VALUES (?, ?, ?)";
            jdbcTemplate.update(sql, 1, "Jone", "db_test_" + i);
            jdbcTemplate.update(sql, 2, "Kath", "db_test_" + i);
            jdbcTemplate.update(sql, 3, "Tom", "db_test_" + i);
            jdbcTemplate.update(sql, 4, "Sandy", "db_test_" + i);
            jdbcTemplate.update(sql, 5, "Oliver", "db_test_" + i);
            // 还原数据源
            DynamicDataSourceContextHolder.clearDataSource();
        });
        return "init data success";
    }

    @RequestMapping("/test1")
    public List<Map<String, Object>> test1() {
        DynamicDataSourceContextHolder.setDataSource("test_1");
        List<Map<String, Object>> list = jdbcTemplate.queryForList("select * from USER_INFO");
        DynamicDataSourceContextHolder.clearDataSource();
        return list;
    }

    @RequestMapping("/test2")
    public List<Map<String, Object>> test2() {
        DynamicDataSourceContextHolder.setDataSource("test_2");
        List<Map<String, Object>> list = jdbcTemplate.queryForList("select * from USER_INFO");
        DynamicDataSourceContextHolder.clearDataSource();
        return list;
    }

    @RequestMapping("/test3")
    public List<Map<String, Object>> test3() {
        DynamicDataSourceContextHolder.setDataSource("test_3");
        List<Map<String, Object>> list = jdbcTemplate.queryForList("select * from USER_INFO");
        DynamicDataSourceContextHolder.clearDataSource();
        return list;
    }
}
```

2、调用新增动态数据源 & 运行时新增数据源的接口

```bash
curl -X GET "localhost:8090/initDataSource"
init dataSource success
```

3、调用初始化每个数据源表和数据的接口

```bash
curl -X GET "localhost:8090/initData"      
init data succes
```

4、查询不同数据源中数据的接口

```bash
curl -X GET "localhost:8090/test1"   
[{"ID":1,"NAME":"Jone","TYPE":"db_test_1"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_1"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_1"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_1"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_1"}]

curl -X GET "localhost:8090/test2"
[{"ID":1,"NAME":"Jone","TYPE":"db_test_2"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_2"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_2"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_2"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_2"}]

curl -X GET "localhost:8090/test3" 
[{"ID":1,"NAME":"Jone","TYPE":"db_test_3"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_3"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_3"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_3"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_3"}]
```



### 3、动态数据源使用【AOP切换】

1、增加 AOP 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2、定义数据源注解

```java
package com.example.annotation;

import java.lang.annotation.*;

/**
 * 数据源切换扫描注解
 */
@Documented
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface DS {
    String value() default "master";
}
```

3、定义数据源切面

```java
package com.example.aop;

import com.example.annotation.DS;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

@Slf4j
@Aspect
@Component
public class DataSourceAspect {

    @Pointcut("@annotation(com.example.annotation.DS)")
    public void dataSourcePointCut() {
    }

    @Around("dataSourcePointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        DS DS = method.getAnnotation(DS.class);
        log.info("注解的拦截方法名注解内容前: {}", DS.value());
        DynamicDataSourceContextHolder.setDataSource(DS.value());
        try {
            return point.proceed();
        } finally {
            DynamicDataSourceContextHolder.clearDataSource();
        }
    }
}
```

4、使用 AOP 进行数据源切换

```java
package com.example.controller;

import com.example.annotation.DS;
import com.example.config.DynamicDataSource;
import com.example.config.DynamicDataSourceContextHolder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;
import javax.sql.DataSource;
import java.util.List;
import java.util.Map;
import java.util.stream.Stream;

@Slf4j
@RestController
public class MultiController {

    @Resource
    JdbcTemplate jdbcTemplate;
    @Resource
    DynamicDataSource dynamicDataSource;

    /**
     * 动态数据源配置 & 运行时新增数据源
     * 可以从数据库中加载, 也可以通过API新增
     */
    @RequestMapping("initDataSource")
    public String initDataSource() {
        log.info("===============初始化动态数据源===============");
        Map<Object, Object> dataSourceMap = dynamicDataSource.getTargetDataSources();
        Stream.of(1, 2, 3).forEach(i -> {
            DataSource build = DataSourceBuilder.create()
                    .username("sa")
                    .password("sa")
                    .url("jdbc:h2:mem:test_" + i)
                    .build();
            dataSourceMap.put("test_" + i, build);
        });
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        log.info("===============动态数据源初始完毕===============");
        return "init dataSource success";
    }

    /**
     * 给新增的数据源创建表和初始化数据
     */
    @RequestMapping("initData")
    public String initData() {
        Stream.of(1, 2, 3).forEach(i -> {
            // 切换数据源
            DynamicDataSourceContextHolder.setDataSource("test_" + i);
            // 初始化表和数据
            jdbcTemplate.execute("DROP TABLE IF EXISTS USER_INFO");
            jdbcTemplate.execute("CREATE TABLE USER_INFO ("
                    + "id BIGINT NOT NULL,"
                    + "name VARCHAR(50) DEFAULT NULL,"
                    + "type VARCHAR(50) DEFAULT NULL,"
                    + "PRIMARY KEY (id))");
            String sql = "INSERT INTO USER_INFO (id, name, type) VALUES (?, ?, ?)";
            jdbcTemplate.update(sql, 1, "Jone", "db_test_" + i);
            jdbcTemplate.update(sql, 2, "Kath", "db_test_" + i);
            jdbcTemplate.update(sql, 3, "Tom", "db_test_" + i);
            jdbcTemplate.update(sql, 4, "Sandy", "db_test_" + i);
            jdbcTemplate.update(sql, 5, "Oliver", "db_test_" + i);
            // 还原数据源
            DynamicDataSourceContextHolder.clearDataSource();
        });
        return "init data success";
    }

    @DS("test_1")
    @RequestMapping("/test11")
    public List<Map<String, Object>> test11() {
        return jdbcTemplate.queryForList("select * from USER_INFO");
    }

    @DS("test_2")
    @RequestMapping("/test22")
    public List<Map<String, Object>> test22() {
        return jdbcTemplate.queryForList("select * from USER_INFO");
    }

    @DS("test_3")
    @RequestMapping("/test33")
    public List<Map<String, Object>> test33() {
        return jdbcTemplate.queryForList("select * from USER_INFO");
    }
}
```

5、调用新增动态数据源 & 运行时新增数据源的接口

```bash
curl -X GET "localhost:8090/initDataSource"
init dataSource success
```

6、调用初始化每个数据源表和数据的接口

```bash
curl -X GET "localhost:8090/initData"      
init data succes
```

7、查询不同数据源中数据的接口

```bash
curl -X GET "localhost:8090/test11"   
[{"ID":1,"NAME":"Jone","TYPE":"db_test_1"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_1"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_1"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_1"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_1"}]

curl -X GET "localhost:8090/test22"
[{"ID":1,"NAME":"Jone","TYPE":"db_test_2"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_2"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_2"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_2"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_2"}]

curl -X GET "localhost:8090/test33" 
[{"ID":1,"NAME":"Jone","TYPE":"db_test_3"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_3"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_3"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_3"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_3"}]
```





## 2、Mybatis + H2 静态多数据源配置

### 1、动态数据源配置

1、添加项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>h2-mybaits-multi-datasource-dynamic</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>h2-mybaits-multi-datasource-dynamic</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.3.2</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

2、添加以下自定义的主从数据库配置 application.yml，这里暂时值定义了2个数据源，后面可以通过代码逻辑手动添加新的数据源

```yaml
server:
  port: 8090
spring:
  #################### H2 数据库配置 ####################
  # 文件数据库
  # spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
  # 内存数据库，platform #表明使用的数据库平台是 h2
  datasource:
    master:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:master
      username: sa
      password:
    slave:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:slave
      username: sa
      password:
  #################### H2 Web Console设置 ####################
  ### enabled：程序开启时就会启动 H2 Web Console, 默认就是启动的
  ### path：配置访问地址为：/h2，访问 H2 Web Console，默认:/h2-console
  ### settings.web-allow-others：开启H2 Web Console远程访问，默认为false不开启只能在本机访问
  h2:
    console:
      enabled: true
      path: /h2
      settings:
        web-allow-others: true

# 开启控制台打印sql
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```



### 2、动态数据源设置

1、 多数据源配置类：DataSourceConfig

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.logging.stdout.StdOutImpl;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

/**
 * @EnableAutoConfiguration 就是去掉springboot默认自动配置数据源
 */
@Configuration
@MapperScan(basePackages = "com.example.mapper")
@EnableAutoConfiguration(exclude = { DataSourceAutoConfiguration.class })
public class DataSourceConfig {

    @Bean(name = "masterDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean(name = "slave1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource slave1DataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    /**
     * 使用 Map 保存多个数据源，并设置到动态数据源对象中。
     * 设置默认的数据源是 master 数据源
     * 使用注解 Primary 优先从动态数据源中获取
     * 注意: 这个才是主数据源
     */
    @Primary
    @Bean
    public DynamicDataSource dynamicDataSource(){
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put("master",masterDataSource());
        targetDataSources.put("slave", slave1DataSource());
        // 设置默认数据源
        dynamicDataSource.setDefaultTargetDataSource(masterDataSource());
        // 添加数据源集合
        dynamicDataSource.setTargetDataSources(targetDataSources);
        return dynamicDataSource;
    }

    /**
     * Spring-Mybatis 配置第一步：先配置SqlSessionFactory
     */
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dynamicDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        // 传入数据源
        bean.setDataSource(dataSource);
        // 指定扫描的xml文件所在位置，在配置文件里面配置，会报Invalid bound statement
        // bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
        org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
        // 开启驼峰转换
        configuration.setMapUnderscoreToCamelCase(true);
        //配置sql日志
        configuration.setLogImpl(StdOutImpl.class);
        bean.setConfiguration(configuration);
        return bean.getObject();
    }

    /**
     * Spring-Mybatis 配置第一步：先配置SqlSessionTemplate
     */
    @Bean(name = "sqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("sqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    /**
     * 事务管理
     * @param dataSource 数据源
     * @return 事务管理
     */
    @Bean(name = "sqlTransactionManager")
    public PlatformTransactionManager platformTransactionManager(@Qualifier("dynamicDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

2、动态数据设置以及获取类：实现 AbstractRoutingDataSource 的方法，使用 DynamicDataSourceContextHolder 统一数据源管理

```java
package com.example.config;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import java.util.HashMap;
import java.util.Map;

public class DynamicDataSource extends AbstractRoutingDataSource {
    /**
     * 添加一个DataSources Map，用来获取数据源集合
     */
    private final Map<Object, Object> targetDataSources = new HashMap<>();
    public Map<Object, Object> getTargetDataSources() {
        return targetDataSources;
    }

    /**
     * 重写setTargetDataSources方法，为了可以让自己实现动态添加数据源
     * @param targetDataSources
     */
    @Override
    public void setTargetDataSources(Map<Object, Object> targetDataSources) {
        this.targetDataSources.putAll(targetDataSources);
        super.setTargetDataSources(this.targetDataSources);
        super.afterPropertiesSet(); // 必须添加该句，否则新添加数据源无法识别到
    }

    /**
     * 必须实现该接口。用来获取当前数据源
     */
    @Override
    protected Object determineCurrentLookupKey() {
        // 或者return null.细节看源码AbstractRoutingDataSource.determineTargetDataSource
        return DynamicDataSourceContextHolder.getDataSource();
    }
}
```

3、DynamicDataSourceContextHolder 类的实现，通过 ThreadLocal 获取和设置线程安全的数据源 key

```java
package com.example.config;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class DynamicDataSourceContextHolder {
    /**
     * 动态数据源名称上下文
     */
    private static final ThreadLocal<String> contextLocal = ThreadLocal.withInitial(() -> "master");
    // private static final ThreadLocal<String> contextLocal = new ThreadLocal<>();

    /**
     * 设置数据源
     */
    public static void setDataSource(String key) {
        log.info("-----------切换数据源：" + key + "---------------");
        contextLocal.set(key);
    }

    /**
     * 获取数据源
     */
    public static String getDataSource() {
        String key = contextLocal.get();
        log.info("-----------当前数据源：" + key + "---------------");
        return key;
    }

    /**
     * 重置/释放数据源
     */
    public static void clearDataSource() {
        contextLocal.remove();
        String key = contextLocal.get();
        log.info("-----------重置数据源：" + key + "---------------");
    }
}
```



### 3、动态数据源使用【手动切换】

1、数据表对应实体类的创建

```java
package com.example.model;
import lombok.Data;
import java.io.Serializable;

@Data
public class UserInfo implements Serializable {
    private Integer Id;
    private String name;
    private String type;
}
```

2、数据源需要的 Mapper 接口创建

```java
package com.example.mapper;

import com.example.model.UserInfo;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
import java.util.List;

public interface UserMapper {
    @Select("select * from USER_INFO")
    List<UserInfo> findAll();

    @Update("drop table if exists USER_INFO;")
    void dropUserExistTable();

    @Update("create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))")
    void createUserTable();

    @Insert("insert into USER_INFO (id, name, type) values (1, 'Jone', #{type}),(2, 'Kath', #{type}),(3, 'Tom', #{type}),(4, 'Sandy', #{type}),(5, 'Oliver', #{type});")
    int initTableData(@Param("type") String type);
}
```

3、动态数据源使用，这里编写一个 Controller，里面有一个动态初始化数据源接口，一个初始化每个数据源的表和数据的接口，另外还有3个查询不同数据源中数据的接口

```java
package com.example.controller;

import com.example.config.DynamicDataSource;
import com.example.config.DynamicDataSourceContextHolder;
import com.example.mapper.UserMapper;
import com.example.model.UserInfo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import javax.sql.DataSource;
import java.util.List;
import java.util.Map;
import java.util.stream.Stream;

@Slf4j
@RestController
public class MultiController {

    @Resource
    UserMapper userMapper;
    @Resource
    DynamicDataSource dynamicDataSource;

    /**
     * 动态数据源配置 & 运行时新增数据源
     * 可以从数据库中加载, 也可以通过API新增
     */
    @RequestMapping("initDataSource")
    public String initDataSource() {
        log.info("===============初始化动态数据源===============");
        Map<Object, Object> dataSourceMap = dynamicDataSource.getTargetDataSources();
        Stream.of(1, 2, 3).forEach(i -> {
            DataSource build = DataSourceBuilder.create()
                    .username("sa")
                    .password("")
                    .url("jdbc:h2:mem:test" + i)
                    .build();
            dataSourceMap.put("test" + i, build);
        });
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        log.info("===============动态数据源初始完毕===============");
        return "init dataSource success";
    }

    /**
     * 给新增的数据源创建表和初始化数据
     */
    @RequestMapping("initData")
    public String initData() {
        Map<Object, Object> targetDataSources = dynamicDataSource.getTargetDataSources();
        targetDataSources.forEach((k, v) -> {
            DynamicDataSourceContextHolder.setDataSource(k.toString());
            userMapper.dropUserExistTable();
            userMapper.createUserTable();
            userMapper.initTableData(k.toString());
            DynamicDataSourceContextHolder.clearDataSource();
        });
        return "init data success";
    }

    @RequestMapping("/test1")
    public List<UserInfo> test1() {
        DynamicDataSourceContextHolder.setDataSource("test1");
        List<UserInfo> all = userMapper.findAll();
        DynamicDataSourceContextHolder.clearDataSource();
        return all;
    }

    @RequestMapping("/test2")
    public List<UserInfo> test2() {
        DynamicDataSourceContextHolder.setDataSource("test2");
        List<UserInfo> all = userMapper.findAll();
        DynamicDataSourceContextHolder.clearDataSource();
        return all;
    }

    @RequestMapping("/test3")
    public List<UserInfo> test3() {
        DynamicDataSourceContextHolder.setDataSource("test3");
        List<UserInfo> all = userMapper.findAll();
        DynamicDataSourceContextHolder.clearDataSource();
        return all;
    }
}
```

4、调用新增动态数据源 & 运行时新增数据源的接口

```bash
curl -X GET "localhost:8090/initDataSource"
init dataSource success
```

5、调用初始化每个数据源表和数据的接口

```bash
curl -X GET "localhost:8090/initData"      
init data succes
```

6、查询不同数据源中数据的接口

```bash
curl -X GET "localhost:8090/test1"   
[{"name":"Jone","type":"test1","id":1},
{"name":"Kath","type":"test1","id":2},
{"name":"Tom","type":"test1","id":3},
{"name":"Sandy","type":"test1","id":4},
{"name":"Oliver","type":"test1","id":5}]

curl -X GET "localhost:8090/test2"
[{"name":"Jone","type":"test2","id":1},
{"name":"Kath","type":"test2","id":2},
{"name":"Tom","type":"test2","id":3},
{"name":"Sandy","type":"test2","id":4},
{"name":"Oliver","type":"test2","id":5}]

curl -X GET "localhost:8090/test3" 
[{"name":"Jone","type":"test3","id":1},
{"name":"Kath","type":"test3","id":2},
{"name":"Tom","type":"test3","id":3},
{"name":"Sandy","type":"test3","id":4},
{"name":"Oliver","type":"test3","id":5}]
```



### 4、动态数据源使用【AOP切换】

1、增加 AOP 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2、定义数据源注解

```java
package com.example.annotation;

import java.lang.annotation.*;

/**
 * 数据源切换扫描注解
 */
@Documented
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface DS {
    String value() default "master";
}
```

3、定义数据源切面

```java
package com.example.aop;

import com.example.annotation.DS;
import com.example.config.DynamicDataSourceContextHolder;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import java.lang.reflect.Method;

@Slf4j
@Aspect
@Component
public class DataSourceAspect {
    @Pointcut("@annotation(com.example.annotation.DS)")
    public void dataSourcePointCut() {
    }

    @Around("dataSourcePointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        DS DS = method.getAnnotation(DS.class);
        log.info("注解的拦截方法名注解内容前：{}", DS.value());
        DynamicDataSourceContextHolder.setDataSource(DS.value());
        try {
            return point.proceed();
        } finally {
            DynamicDataSourceContextHolder.clearDataSource();
        }
    }
}
```

4、使用 AOP 进行数据源切换

```java
package com.example.controller;

import com.example.annotation.DS;
import com.example.config.DynamicDataSource;
import com.example.config.DynamicDataSourceContextHolder;
import com.example.mapper.UserMapper;
import com.example.model.UserInfo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;
import javax.sql.DataSource;
import java.util.List;
import java.util.Map;
import java.util.stream.Stream;

@Slf4j
@RestController
public class MultiController {

    @Resource
    UserMapper userMapper;
    @Resource
    DynamicDataSource dynamicDataSource;

    /**
     * 动态数据源配置 & 运行时新增数据源
     * 可以从数据库中加载, 也可以通过API新增
     */
    @RequestMapping("initDataSource")
    public String initDataSource() {
        log.info("===============初始化动态数据源===============");
        Map<Object, Object> dataSourceMap = dynamicDataSource.getTargetDataSources();
        Stream.of(1, 2, 3).forEach(i -> {
            DataSource build = DataSourceBuilder.create()
                    .username("sa")
                    .password("")
                    .url("jdbc:h2:mem:test" + i)
                    .build();
            dataSourceMap.put("test" + i, build);
        });
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        log.info("===============动态数据源初始完毕===============");
        return "init dataSource success";
    }

    /**
     * 给新增的数据源创建表和初始化数据
     */
    @RequestMapping("initData")
    public String initData() {
        Map<Object, Object> targetDataSources = dynamicDataSource.getTargetDataSources();
        targetDataSources.forEach((k, v) -> {
            DynamicDataSourceContextHolder.setDataSource(k.toString());
            userMapper.dropUserExistTable();
            userMapper.createUserTable();
            userMapper.initTableData(k.toString());
            DynamicDataSourceContextHolder.clearDataSource();
        });
        return "init data success";
    }

    @DS("test1")
    @RequestMapping("/test11")
    public List<UserInfo> test11() {
        return userMapper.findAll();
    }

    @DS("test2")
    @RequestMapping("/test22")
    public List<UserInfo> test22() {
        return userMapper.findAll();
    }

    @DS("test3")
    @RequestMapping("/test33")
    public List<UserInfo> test33() {
        return userMapper.findAll();
    }
}
```

5、调用新增动态数据源 & 运行时新增数据源的接口

```bash
curl -X GET "localhost:8090/initDataSource"
init dataSource success
```

6、调用初始化每个数据源表和数据的接口

```bash
curl -X GET "localhost:8090/initData"      
init data succes
```

7、查询不同数据源中数据的接口

```bash
curl -X GET "localhost:8090/test1"   
[{"ID":1,"NAME":"Jone","TYPE":"db_test_1"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_1"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_1"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_1"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_1"}]

curl -X GET "localhost:8090/test2"
[{"ID":1,"NAME":"Jone","TYPE":"db_test_2"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_2"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_2"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_2"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_2"}]

curl -X GET "localhost:8090/test3" 
[{"ID":1,"NAME":"Jone","TYPE":"db_test_3"},
{"ID":2,"NAME":"Kath","TYPE":"db_test_3"},
{"ID":3,"NAME":"Tom","TYPE":"db_test_3"},
{"ID":4,"NAME":"Sandy","TYPE":"db_test_3"},
{"ID":5,"NAME":"Oliver","TYPE":"db_test_3"}]
```





## 3、Spring Data JPA + H2 静态多数据源配置



```java
package com.example.config;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

import java.util.HashMap;
import java.util.Map;

public class DynamicDataSource extends AbstractRoutingDataSource {
    /**
     * 添加一个DataSources Map，用来获取数据源集合
     */
    private final Map<Object, Object> targetDataSources = new HashMap<>();
    public Map<Object, Object> getTargetDataSources() {
        return targetDataSources;
    }

    /**
     * 重写setTargetDataSources方法，为了可以让自己得到
     * @param targetDataSources
     */
    @Override
    public void setTargetDataSources(Map<Object, Object> targetDataSources) {
        this.targetDataSources.putAll(targetDataSources);
        super.setTargetDataSources(this.targetDataSources);
        super.afterPropertiesSet(); // 必须添加该句，否则新添加数据源无法识别到
    }

    /**
     * 必须实现该接口。用来获取当前数据源
     */
    @Override
    protected Object determineCurrentLookupKey() {
        // 或者return null.细节看源码AbstractRoutingDataSource.determineTargetDataSource
        return DynamicDataSourceContextHolder.getDataSource();
    }
}
```

```java
package com.example.config;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class DynamicDataSourceContextHolder {
    /**
     * 动态数据源名称上下文
     */
    private static final ThreadLocal<String> contextLocal = ThreadLocal.withInitial(() -> "master");
    // private static final ThreadLocal<String> contextLocal = new ThreadLocal<>();

    /**
     * 设置数据源
     */
    public static void setDataSource(String key) {
        log.info("-----------切换数据源：" + key + "---------------");
        contextLocal.set(key);
    }

    /**
     * 获取数据源
     */
    public static String getDataSource() {
        String key = contextLocal.get();
        log.info("-----------当前数据源：" + key + "---------------");
        return key;
    }

    /**
     * 重置/释放数据源
     */
    public static void clearDataSource() {
        contextLocal.remove();
        String key = contextLocal.get();
        log.info("-----------重置数据源：" + key + "---------------");
    }
}
```





### 1、动态数据源配置

1、添加项目依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>h2-jpa-multi-datasource-dynamic</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>h2-jpa-multi-datasource-dynamic</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

2、添加以下自定义的主从数据库配置 application.yml，这里暂时值定义了2个数据源，后面可以通过代码逻辑手动添加新的数据源

```yaml
server:
  port: 8090
spring:
  #################### H2 数据库配置 ####################
  # 文件数据库
  # spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
  # 内存数据库，platform #表明使用的数据库平台是 h2
  datasource:
    master:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:mem:master
      username: sa
      password:
      platform: h2
  #################### H2 Web Console设置 ####################
  ### enabled：程序开启时就会启动 H2 Web Console, 默认就是启动的
  ### path：配置访问地址为：/h2，访问 H2 Web Console，默认:/h2-console
  ### settings.web-allow-others：开启H2 Web Console远程访问，默认为false不开启只能在本机访问
  h2:
    console:
      enabled: true
      path: /h2
      settings:
        web-allow-others: true
  #################### JPA 的配置 ####################
  # database-platform: 数据库方言的配置
  # hibernate.ddl-auto = none, 不启动 jpa 的自动通过实体类自动建表功能
  # show-sql = true 开启 SQL 打印功能
  # properties.hibernate.format_sql = true 开启SQL格式化功能
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        format_sql: false
    show-sql: true
    #open-in-view: false

### 此配置是为了调试, 每个HTTP请求进来会打印 Open Hibernate 和 Open Session, 还有从当前线程中取出EntityManager ###
#################### 修改日志级别 ####################
#logging:
#  level:
#    org.hibernate.engine.transaction.internal: trace
#    org.springframework.orm.jpa.JpaTransactionManager: trace
#    org.hibernate.internal: trace

```



### 2、动态数据源设置

1、 多数据源配置类：DataSourceConfig

```java
package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateProperties;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateSettings;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.annotation.Resource;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

/**
 * 配置顺序如下：
 * 创建DataSource—》创建EntityManagerFactory—》创建TransactionManager—》@EnableJpaRepositories开启JPA
 * <p>
 * 在多数据源配置中还是需要手动开启事务 @EnableTransactionManagement，并受用 @Primary 注解指定了主要的事务管理器
 * 利⽤ EnableJpaRepositories 配置哪些包下⾯的 Repositories，采⽤哪个 EntityManagerFactory 和哪个 TransactionManagement
 */
@Configuration
@EnableTransactionManagement // 开启事务
@EnableJpaRepositories(
        entityManagerFactoryRef = "entityManagerFactoryMaster", // 数据源的 repository 的包路径
        transactionManagerRef = "transactionManagerMaster",  // 实体管理器工厂
        basePackages = {"com.example.repository"})  // 事务管理器
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
public class DataSourceConfig {
    @Resource
    private JpaProperties jpaProperties;

    @Resource
    private HibernateProperties hibernateProperties;

    @Bean(name = "masterDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Primary
    @Bean
    public DynamicDataSource dynamicDataSource() {
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        Map<Object, Object> targetDataSources = new HashMap<>(16);
        targetDataSources.put("master", masterDataSource());
        // 设置默认数据源
        dynamicDataSource.setDefaultTargetDataSource(masterDataSource());
        // 添加数据源集合
        dynamicDataSource.setTargetDataSources(targetDataSources);
        return dynamicDataSource;
    }

    /**
     * primary实体工厂
     */
    @Primary
    @Bean(name = "entityManagerFactoryMaster")
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryMaster(EntityManagerFactoryBuilder builder) {
        /**
         * LocalContainerEntityManagerFactoryBean entityManagerFactoryMaster(
         *          HibernateProperties hibernateProperties, JpaProperties jpaProperties) {}
         * 还可以使用 new 来创建 LocalContainerEntityManagerFactoryBean
         * LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
         * factoryBean.setDataSource(dataSource);
         * factoryBean.setPackagesToScan("com.example.model"); // 实体类包路径
         * factoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter()); // 设置Hibernate适配器
         * factoryBean.setJpaPropertyMap(jpaProperties.getProperties()); // 只设置了JpaProperties
         */

        // 配置 JPA 和 Hibernate 的属性, 相当于合并2个配置
        Map<String, Object> properties = hibernateProperties.
                determineHibernateProperties(jpaProperties.getProperties(), new HibernateSettings());
        // 使用 EntityManagerFactoryBuilder 构建 EntityManagerFactory
        // 调用 build() 方法时，会自动应用配置中的 JpaVendorAdapter, 这里默认构建 HibernateJpaVendorAdapter
        return builder.dataSource(dynamicDataSource())
                .properties(properties)
                // 扫描的实体类所在位置
                .packages("com.example.model")
                // jpa定义的名称, persistenceUnit 的名字采⽤ masterPersistenceUnit
                .persistenceUnit("masterPersistenceUnit")
                .build();
    }

    /**
     * master事务管理器
     */
    @Primary
    @Bean(name = "transactionManagerMaster")
    public PlatformTransactionManager transactionManagerMaster(EntityManagerFactoryBuilder builder) {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = entityManagerFactoryMaster(builder);
        return new JpaTransactionManager(entityManagerFactoryBean.getObject());
    }

    /**
     * master实体管理器
     */
    @Primary
    @Bean(name = "entityManagerMaster")
    public EntityManager entityManagerMaster(EntityManagerFactoryBuilder builder) {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = entityManagerFactoryMaster(builder);
        return entityManagerFactoryBean.getObject().createEntityManager();
    }
}
```

2、动态数据设置以及获取类：实现 AbstractRoutingDataSource 的方法，使用 DynamicDataSourceContextHolder 统一数据源管理

```java
package com.example.config;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import java.util.HashMap;
import java.util.Map;

public class DynamicDataSource extends AbstractRoutingDataSource {
    /**
     * 添加一个DataSources Map，用来获取数据源集合
     */
    private final Map<Object, Object> targetDataSources = new HashMap<>();
    public Map<Object, Object> getTargetDataSources() {
        return targetDataSources;
    }

    /**
     * 重写setTargetDataSources方法，为了可以让自己实现动态添加数据源
     * @param targetDataSources
     */
    @Override
    public void setTargetDataSources(Map<Object, Object> targetDataSources) {
        this.targetDataSources.putAll(targetDataSources);
        super.setTargetDataSources(this.targetDataSources);
        super.afterPropertiesSet(); // 必须添加该句，否则新添加数据源无法识别到
    }

    /**
     * 必须实现该接口。用来获取当前数据源
     */
    @Override
    protected Object determineCurrentLookupKey() {
        // 或者return null.细节看源码AbstractRoutingDataSource.determineTargetDataSource
        return DynamicDataSourceContextHolder.getDataSource();
    }
}
```

3、DynamicDataSourceContextHolder 类的实现，通过 ThreadLocal 获取和设置线程安全的数据源 key

```java
package com.example.config;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class DynamicDataSourceContextHolder {
    /**
     * 动态数据源名称上下文
     */
    private static final ThreadLocal<String> contextLocal = ThreadLocal.withInitial(() -> "master");
    // private static final ThreadLocal<String> contextLocal = new ThreadLocal<>();

    /**
     * 设置数据源
     */
    public static void setDataSource(String key) {
        log.info("-----------切换数据源：" + key + "---------------");
        contextLocal.set(key);
    }

    /**
     * 获取数据源
     */
    public static String getDataSource() {
        String key = contextLocal.get();
        log.info("-----------当前数据源：" + key + "---------------");
        return key;
    }

    /**
     * 重置/释放数据源
     */
    public static void clearDataSource() {
        contextLocal.remove();
        String key = contextLocal.get();
        log.info("-----------重置数据源：" + key + "---------------");
    }
}
```

4、由于 JPA 的事务与数据源处理机制特殊性，会导致在一个请求中，Spring会管理这个EntityManager的生命周期。如果在请求中尝试多次切换数据源，可能会导致EntityManager的状态不一致。但是在@PostConstruct方法中，由于它是在请求上下文之外执行的，因此可以自由地切换数据源，而不会受到当前事务的限制。

```java
package com.example.config;

import com.example.repository.UserRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Configuration;
import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import javax.sql.DataSource;
import java.util.Map;
import java.util.stream.Stream;

/**
 * 数据源初始化（从数据库中加载）
 * <p>
 * '@PostConstruct' 可以多次切换数据源
 * 没有事务绑定: @PostConstruct 方法中，Spring 并没有开启事务，也不会进行事务管理。由于没有事务的约束，数据源切换不会受到事务管理器的限制，每次切换都会生效。
 * 单线程执行: @PostConstruct 方法是在单线程中执行的，且不会与其他请求并发执行。数据源的上下文是通过 ThreadLocal 存储的，因此在方法内的每次切换都是独立的，不会受到其他线程的干扰。
 */
@Slf4j
@Configuration
public class DynamicDataSourceInit {
    @Resource
    DynamicDataSource dynamicDataSource;
    @Resource
    UserRepository userRepository;

    @PostConstruct
    public void InitDataSource() {
        log.info("===============初始化动态数据源===============");
        Map<Object, Object> dataSourceMap = dynamicDataSource.getTargetDataSources();
        Stream.of(1, 2, 3).forEach(i -> {
            DataSource dataSource = DataSourceBuilder.create()
                    .username("sa")
                    .password("")
                    .url("jdbc:h2:mem:test" + i)
                    .build();
            dataSourceMap.put("test" + i, dataSource);
        });
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        log.info("===============动态数据源初始完毕===============");

        log.info("===============初始化每个数据源数据开始===============");
        dynamicDataSource.getTargetDataSources().forEach((k, y) -> {
            DynamicDataSourceContextHolder.setDataSource(k.toString());
            userRepository.dropUserExistTable();
            userRepository.createUserTable();
            userRepository.initTableData(k.toString());
            userRepository.findAll().forEach(System.out::println);
            DynamicDataSourceContextHolder.clearDataSource();
        });
        log.info("===============初始化动态数据源结束===============");
    }
}
```



### 3、动态数据源使用【手动切换】

#### 1、数据表对应实体类的创建

```java
package com.example.model;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "USER_INFO")
public class UserInfo {
    @Id
    @GeneratedValue
    private Integer Id;
    private String name;
    private String type;
}
```

#### 2、数据源需要的 Repository 接口创建

```java
package com.example.repository;

import com.example.model.UserInfo;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.transaction.annotation.Transactional;

public interface UserInfoRepository extends JpaRepository<UserInfo, Integer> {
    @Modifying
    @Transactional
    @Query(value = "drop table if exists USER_INFO;",nativeQuery=true)
    void dropUserExistTable();

    @Modifying
    @Transactional
    @Query( value= "create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))",nativeQuery=true)
    void createUserTable();

    @Modifying
    @Transactional
    @Query(value="insert into USER_INFO (id, name, type) values (1, 'Jone', :type),(2, 'Kath', :type),(3, 'Tom', :type),(4, 'Sandy', :type),(5, 'Oliver', :type);",nativeQuery=true)
    int initTableData(@Param("type")String type);
}
```



#### 3、编写一个 Controller使用动态数据源

使用里面有一个动态初始化每个数据源的表和数据的接口，另外还有3个查询不同数据源中数据的接口。

通过此 Controller 可以发现如下几个问题【**重点**】：

1. SpringBoot整合JPA多数据源时，一个请求中无法无法多次切换数据源。可是使用@PostConstruct注释的方法却可以多次切换数据源，这是为什么？
2. SpringBoot整合JPA多数据源时，一个请求中无法无法多次切换数据源。可是整合Mybatis多数据源时，一个请求中却可以切换多次数据源，这是为什么？
3. spring.jpa.open-in-view = false 配置的解释，为什么加上此配置后，一个请求中就可以多次切换数据源

```java
package com.example.controller;

import com.example.annotation.DS;
import com.example.config.DynamicDataSource;
import com.example.config.DynamicDataSourceContextHolder;
import com.example.model.UserInfo;
import com.example.repository.UserInfoRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jpa.repository.support.JpaRepositoryFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;
import javax.persistence.*;
import java.util.List;

/**
 * 由于SpringMVC默认启用 “Open Session in View” 模式。
 * 这意味着在处理一个请求时，Hibernate 会在整个请求的生命周期内保持一个持久化的Session或EntityManager.
 * 虽然我们手动切换了数据源，但是在JPA事务中依旧是去获取当前请求线程中已经创建的EntityManager.
 * 所以会导致一个请求进来, 只有第一次数据源切换成功, 之后的切换都是无效的.
 * <p>
 * 详解可以参考：
 * 问题 1：SpringBoot整合JPA多数据源时，一个请求中无法无法多次切换数据源。可是使用@PostConstruct注释的方法却可以多次切换数据源，这是为什么？
 * 问题 2：SpringBoot整合JPA多数据源时，一个请求中无法无法多次切换数据源。可是整合Mybatis多数据源时，一个请求中却可以切换多次数据源，这是为什么？
 * 问题 3：spring.jpa.open-in-view = false 配置的解释
 */
@Slf4j
@RestController
public class MultiController {
    @Resource
    DynamicDataSource dynamicDataSource;
    @Resource
    UserInfoRepository userInfoRepository;
    @PersistenceContext
    EntityManager entityManager;
    @PersistenceUnit
    EntityManagerFactory entityManagerFactory;

    /**
     * 给新增的数据源创建表和初始化数据
     */
    @RequestMapping("/initData")
    public String initData() {
        dynamicDataSource.getTargetDataSources().forEach((k, y) -> {
            // 循环切换数据源, 因为手动多次切换数据源, 所以使用到entityManagerFactory+手动事务
            DynamicDataSourceContextHolder.setDataSource(k.toString());
            EntityManager manager = entityManagerFactory.createEntityManager();
            EntityTransaction transaction = manager.getTransaction();
            transaction.begin();
            UserInfoRepository repository = new JpaRepositoryFactory(manager).getRepository(UserInfoRepository.class);
            repository.dropUserExistTable();
            repository.createUserTable();
            repository.initTableData(k.toString());
            repository.findAll().forEach(System.out::println);
            transaction.commit();
            manager.close();
            DynamicDataSourceContextHolder.clearDataSource();
        });
        return "init data success";
    }

    /**
     * 【失效】多数据源切换案例一: repository 接口失效
     * 可以发现使用 repository 接口时，多次切换数据源会失效。只有第一次切换是成功的
     * 【解决】
     * 方案一：参考test3(),直接使用entityManagerFactory来创建EntityManager
     * 方案二：配置 spring.jpa.open-in-view = false
     * 【注意】使用此方式需要手动事务处理，声明式事务会失效
     */
    @RequestMapping("/test1")
    public List<UserInfo> test1() {
        DynamicDataSourceContextHolder.setDataSource("test1");
        List<UserInfo> userInfos1 = userInfoRepository.findAll();
        userInfos1.forEach(System.out::println);
        DynamicDataSourceContextHolder.clearDataSource();

        DynamicDataSourceContextHolder.setDataSource("test2");
        List<UserInfo> userInfos2 = userInfoRepository.findAll();
        userInfos2.forEach(System.out::println);
        DynamicDataSourceContextHolder.clearDataSource();
        return userInfos1;
    }

    /**
     * 【失效】多数据源切换案例二: 注入使用线程安全的 EntityManager 接口失效
     * 可以发现使用 EntityManager 接口时，多次切换数据源会失效。只有第一次切换是成功的
     * 【解决】
     * 方案一：参考test3(),直接使用entityManagerFactory来创建EntityManager
     * 方案二：配置 spring.jpa.open-in-view = false
     * 【注意】使用此方式需要手动事务处理，声明式事务会失效
     */
    @RequestMapping("/test2")
    public List<UserInfo> test2() {
        DynamicDataSourceContextHolder.setDataSource("test1");
        List<UserInfo> userInfos1 = entityManager.createQuery("from UserInfo", UserInfo.class).getResultList();
        userInfos1.forEach(System.out::println);
        DynamicDataSourceContextHolder.clearDataSource();

        DynamicDataSourceContextHolder.setDataSource("test2");
        List<UserInfo> userInfos2 = entityManager.createQuery("from UserInfo", UserInfo.class).getResultList();
        userInfos2.forEach(System.out::println);
        DynamicDataSourceContextHolder.clearDataSource();
        return userInfos2;
    }

    /**
     * 【成功】多数据源切换案例三: 使用注入的 EntityManagerFactory，重新手动创建EntityManager/Session
     * 或者使用 JpaRepositoryFactory + EntityManagerFactory 创建 repository
     * 【结论】最后发现都能正确的切换数据源，并且查询数据正确。
     * 【注意】使用此方式需要手动事务处理，声明式事务会失效
     */
    @RequestMapping("/test3")
    public List<UserInfo> test3() {
        DynamicDataSourceContextHolder.setDataSource("test1");
        EntityManager entityManager1 = entityManagerFactory.createEntityManager();
        entityManager1.createQuery("from UserInfo", UserInfo.class).getResultList().forEach(System.out::println);
        JpaRepositoryFactory jpaRepositoryFactory1 = new JpaRepositoryFactory(entityManager1);
        jpaRepositoryFactory1.getRepository(UserInfoRepository.class).findAll().forEach(System.out::println);
        entityManager1.close();
        DynamicDataSourceContextHolder.clearDataSource();

        DynamicDataSourceContextHolder.setDataSource("test2");
        EntityManager entityManager2 = entityManagerFactory.createEntityManager();
        entityManager2.createQuery("from UserInfo", UserInfo.class).getResultList().forEach(System.out::println);
        JpaRepositoryFactory jpaRepositoryFactory2 = new JpaRepositoryFactory(entityManager2);
        jpaRepositoryFactory2.getRepository(UserInfoRepository.class).findAll().forEach(System.out::println);
        entityManager1.close();
        DynamicDataSourceContextHolder.clearDataSource();

        DynamicDataSourceContextHolder.setDataSource("test3");
        EntityManager entityManager3 = entityManagerFactory.createEntityManager();
        entityManager3.createQuery("from UserInfo", UserInfo.class).getResultList().forEach(System.out::println);
        JpaRepositoryFactory jpaRepositoryFactory3 = new JpaRepositoryFactory(entityManager3);
        UserInfoRepository userInfoRepository3 = jpaRepositoryFactory3.getRepository(UserInfoRepository.class);
        List<UserInfo> userInfos3 = userInfoRepository3.findAll();
        userInfos3.forEach(System.out::println);
        entityManager3.close();
        DynamicDataSourceContextHolder.clearDataSource();

        return userInfos3;
    }

    @DS("test1")
    @RequestMapping("/test11")
    public List<UserInfo> slave11() {
        return userInfoRepository.findAll();
    }

    @DS("test2")
    @RequestMapping("/test22")
    public List<UserInfo> slave22() {
        return userInfoRepository.findAll();
    }

    @DS("test3")
    @RequestMapping("/test33")
    public List<UserInfo> slave33() {
        return userInfoRepository.findAll();
    }

}
```



#### 4、测试 @PostConstruct 内动态切换数据源

项目启动后会进行一系列初始化，由于配置了@PostConstruct方法，所以会动态初始化数据源和数据表以及数据。

```java
@PostConstruct
public void InitDataSource() {
    log.info("===============初始化动态数据源===============");
    Map<Object, Object> dataSourceMap = dynamicDataSource.getTargetDataSources();
    Stream.of(1, 2, 3).forEach(i -> {
        DataSource dataSource = DataSourceBuilder.create()
            .username("sa")
            .password("")
            .url("jdbc:h2:mem:test" + i)
            .build();
        dataSourceMap.put("test" + i, dataSource);
    });
    dynamicDataSource.setTargetDataSources(dataSourceMap);
    log.info("===============动态数据源初始完毕===============");

    log.info("===============初始化每个数据源数据开始===============");
    dynamicDataSource.getTargetDataSources().forEach((k, y) -> {
        DynamicDataSourceContextHolder.setDataSource(k.toString());
        userInfoRepository.dropUserExistTable();
        userInfoRepository.createUserTable();
        userInfoRepository.initTableData(k.toString());
        userInfoRepository.findAll().forEach(System.out::println);
        DynamicDataSourceContextHolder.clearDataSource();
    });
    log.info("===============初始化动态数据源结束===============");
}
```

```shell
2024-09-03 22:50:08.666  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2024-09-03 22:50:08.736  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2024-09-03 22:50:08.745  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：master---------------
2024-09-03 22:50:08.745  INFO 80264 --- [main] o.s.b.a.h2.H2ConsoleAutoConfiguration    : H2 console available at '/h2'. Databases available at 'jdbc:h2:mem:master', 'jdbc:h2:mem:master'
2024-09-03 22:50:08.854  INFO 80264 --- [main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: masterPersistenceUnit]
2024-09-03 22:50:08.873  INFO 80264 --- [main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.6.15.Final
2024-09-03 22:50:08.947  INFO 80264 --- [main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
2024-09-03 22:50:08.989  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：master---------------
2024-09-03 22:50:08.993  INFO 80264 --- [main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2024-09-03 22:50:09.249  INFO 80264 --- [main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2024-09-03 22:50:09.254  INFO 80264 --- [main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'masterPersistenceUnit'
2024-09-03 22:50:09.535  INFO 80264 --- [main] c.example.config.DynamicDataSourceInit   : ===============初始化动态数据源===============
2024-09-03 22:50:09.538  INFO 80264 --- [main] c.example.config.DynamicDataSourceInit   : ===============动态数据源初始完毕===============
2024-09-03 22:50:09.539  INFO 80264 --- [main] c.example.config.DynamicDataSourceInit   : ===============初始化每个数据源数据开始===============
2024-09-03 22:50:09.539  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test2---------------
2024-09-03 22:50:09.574  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test2---------------
2024-09-03 22:50:09.574  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Starting...
2024-09-03 22:50:09.577  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Start completed.
Hibernate: drop table if exists USER_INFO;
2024-09-03 22:50:09.608  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test2---------------
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
2024-09-03 22:50:09.616  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test2---------------
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
2024-09-03 22:50:09.623  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test2---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test2)
UserInfo(Id=2, name=Kath, type=test2)
UserInfo(Id=3, name=Tom, type=test2)
UserInfo(Id=4, name=Sandy, type=test2)
UserInfo(Id=5, name=Oliver, type=test2)
2024-09-03 22:50:09.774  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 22:50:09.774  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test3---------------
2024-09-03 22:50:09.775  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test3---------------
2024-09-03 22:50:09.775  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-3 - Starting...
2024-09-03 22:50:09.778  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-3 - Start completed.
Hibernate: drop table if exists USER_INFO;
2024-09-03 22:50:09.783  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test3---------------
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
2024-09-03 22:50:09.786  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test3---------------
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
2024-09-03 22:50:09.790  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test3---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test3)
UserInfo(Id=2, name=Kath, type=test3)
UserInfo(Id=3, name=Tom, type=test3)
UserInfo(Id=4, name=Sandy, type=test3)
UserInfo(Id=5, name=Oliver, type=test3)
2024-09-03 22:50:09.794  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 22:50:09.794  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test1---------------
2024-09-03 22:50:09.794  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
2024-09-03 22:50:09.794  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-4 - Starting...
2024-09-03 22:50:09.799  INFO 80264 --- [main] com.zaxxer.hikari.HikariDataSource       : HikariPool-4 - Start completed.
Hibernate: drop table if exists USER_INFO;
2024-09-03 22:50:09.802  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
2024-09-03 22:50:09.806  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
2024-09-03 22:50:09.807  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
2024-09-03 22:50:09.809  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 22:50:09.812  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：master---------------
2024-09-03 22:50:09.812  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：master---------------
Hibernate: drop table if exists USER_INFO;
2024-09-03 22:50:09.815  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：master---------------
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
2024-09-03 22:50:09.817  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：master---------------
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
2024-09-03 22:50:09.818  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：master---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=master)
UserInfo(Id=2, name=Kath, type=master)
UserInfo(Id=3, name=Tom, type=master)
UserInfo(Id=4, name=Sandy, type=master)
UserInfo(Id=5, name=Oliver, type=master)
2024-09-03 22:50:09.820  INFO 80264 --- [main] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 22:50:09.821  INFO 80264 --- [main] c.example.config.DynamicDataSourceInit   : ===============初始化动态数据源结束===============
2024-09-03 22:50:09.920  WARN 80264 --- [main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2024-09-03 22:50:10.221  INFO 80264 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8090 (http) with context path ''
2024-09-03 22:50:10.251  INFO 80264 --- [main] e.H2JpaMultiDatasourceDynamicApplication : Started H2JpaMultiDatasourceDynamicApplication in 3.705 seconds (JVM running for 4.359)
```

- 可以发现初始化数据源，动态新增了3个数据源
- 然后循环动态切换数据源初始化每个数据源的表和数据
- 最后打印了spring.jpa.open-in-view is enabled by default.可以发现默认是开启“Open Session in View” 模式的



#### 5、调用初始化表和数据的接口 /initData

```java
/**
 * 给新增的数据源创建表和初始化数据
    */
@RequestMapping("/initData")
public String initData() {
    dynamicDataSource.getTargetDataSources().forEach((k, y) -> {
        // 循环切换数据源, 因为手动多次切换数据源, 所以使用到entityManagerFactory+手动事务
        DynamicDataSourceContextHolder.setDataSource(k.toString());
        EntityManager manager = entityManagerFactory.createEntityManager();
        EntityTransaction transaction = manager.getTransaction();
        transaction.begin();
        UserInfoRepository repository = new JpaRepositoryFactory(manager).getRepository(UserInfoRepository.class);
        repository.dropUserExistTable();
        repository.createUserTable();
        repository.initTableData(k.toString());
        repository.findAll().forEach(System.out::println);
        transaction.commit();
        manager.close();
        DynamicDataSourceContextHolder.clearDataSource();
    });
    return "init data success";
}
```

```shell
curl -X GET "localhost:8090/initData"
init data success

###### 控制台输出的内容 ######
2024-09-03 22:59:36.916  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test2---------------
2024-09-03 22:59:36.919  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test2---------------
Hibernate: drop table if exists USER_INFO;
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test2)
UserInfo(Id=2, name=Kath, type=test2)
UserInfo(Id=3, name=Tom, type=test2)
UserInfo(Id=4, name=Sandy, type=test2)
UserInfo(Id=5, name=Oliver, type=test2)
2024-09-03 22:59:36.967  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 22:59:36.967  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test3---------------
2024-09-03 22:59:36.967  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test3---------------
Hibernate: drop table if exists USER_INFO;
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test3)
UserInfo(Id=2, name=Kath, type=test3)
UserInfo(Id=3, name=Tom, type=test3)
UserInfo(Id=4, name=Sandy, type=test3)
UserInfo(Id=5, name=Oliver, type=test3)
2024-09-03 22:59:36.981  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 22:59:36.982  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test1---------------
2024-09-03 22:59:36.982  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
Hibernate: drop table if exists USER_INFO;
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
2024-09-03 22:59:36.995  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 22:59:36.995  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：master---------------
2024-09-03 22:59:36.995  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：master---------------
Hibernate: drop table if exists USER_INFO;
Hibernate: create TABLE USER_INFO (id BIGINT NOT NULL AUTO_INCREMENT,name VARCHAR(50) DEFAULT NULL,type VARCHAR(50) DEFAULT NULL,PRIMARY KEY (id))
Hibernate: insert into USER_INFO (id, name, type) values (1, 'Jone', ?),(2, 'Kath', ?),(3, 'Tom', ?),(4, 'Sandy', ?),(5, 'Oliver', ?);
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=master)
UserInfo(Id=2, name=Kath, type=master)
UserInfo(Id=3, name=Tom, type=master)
UserInfo(Id=4, name=Sandy, type=master)
UserInfo(Id=5, name=Oliver, type=master)
2024-09-03 22:59:37.010  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
```



#### 6、调用 test1 数据源接口 /test1

这里使用的是 repository 接口查询的【**注意**：test1接口中切换了2次数据源，第1次切换到test1数据源，第2次切换到test2数据源并打印】，**从下面的日志可以发现第2次查询已经差出了test1数据源的数据，这表示第2次切换数据源失败了。**

```java
/**
 * 【失效】多数据源切换案例一: repository 接口失效
 * 可以发现使用 repository 接口时，多次切换数据源会失效。只有第一次切换是成功的
 * 【解决】
 * 方案一：参考test3(),直接使用entityManagerFactory来创建EntityManager
 * 方案二：配置 spring.jpa.open-in-view = false
 * 【注意】使用此方式需要手动事务处理，声明式事务会失效
 */
@RequestMapping("/test1")
public List<UserInfo> test1() {
    DynamicDataSourceContextHolder.setDataSource("test1");
    List<UserInfo> userInfos1 = userInfoRepository.findAll();
    userInfos1.forEach(System.out::println);
    DynamicDataSourceContextHolder.clearDataSource();

    DynamicDataSourceContextHolder.setDataSource("test2");
    List<UserInfo> userInfos2 = userInfoRepository.findAll();
    userInfos2.forEach(System.out::println);
    DynamicDataSourceContextHolder.clearDataSource();
    return userInfos1;
}
```

```shell
curl -X GET "localhost:8090/test1"   
[{"name":"Jone","type":"test1","id":1},
{"name":"Kath","type":"test1","id":2},
{"name":"Tom","type":"test1","id":3},
{"name":"Sandy","type":"test1","id":4},
{"name":"Oliver","type":"test1","id":5}]

###### 控制台输出的内容 ######
2024-09-03 23:01:13.017  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test1---------------
2024-09-03 23:01:13.019  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
2024-09-03 23:01:13.036  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 23:01:13.037  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test2---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
2024-09-03 23:01:13.041  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
```



#### 7、调用 test2 数据源接口 /test2

这里使用的是 EntityManager 接口查询的【注意：test2接口中切换了2次数据源，第1次切换到test1数据源，第2次切换到test2数据源并打印】，**从下面的日志可以发现第2次查询已经差出了test1数据源的数据，这表示第2次切换数据源失败了。**6、查询test1数据源中数据的接口/test1【注意：test1接口中切换了2次数据源，第1次切换到test1数据源，第2次切换到test2数据源并打印】，**从下面的日志可以发现第2次查询已经差出了test1数据源的数据，这表示第2次切换数据源失败了。**

```java
/**
 * 【失效】多数据源切换案例二: 注入使用线程安全的 EntityManager 接口失效
 * 可以发现使用 EntityManager 接口时，多次切换数据源会失效。只有第一次切换是成功的
 * 【解决】
 * 方案一：参考test3(),直接使用entityManagerFactory来创建EntityManager
 * 方案二：配置 spring.jpa.open-in-view = false
 * 【注意】使用此方式需要手动事务处理，声明式事务会失效
 */
@RequestMapping("/test2")
public List<UserInfo> test2() {
    DynamicDataSourceContextHolder.setDataSource("test1");
    List<UserInfo> userInfos1 = entityManager.createQuery("from UserInfo", UserInfo.class).getResultList();
    userInfos1.forEach(System.out::println);
    DynamicDataSourceContextHolder.clearDataSource();

    DynamicDataSourceContextHolder.setDataSource("test2");
    List<UserInfo> userInfos2 = entityManager.createQuery("from UserInfo", UserInfo.class).getResultList();
    userInfos2.forEach(System.out::println);
    DynamicDataSourceContextHolder.clearDataSource();
    return userInfos2;
}
```

```shell
curl -X GET "localhost:8090/test2"
[{"name":"Jone","type":"test1","id":1},
{"name":"Kath","type":"test1","id":2},
{"name":"Tom","type":"test1","id":3},
{"name":"Sandy","type":"test1","id":4},
{"name":"Oliver","type":"test1","id":5}]

###### 控制台输出的内容 ######
2024-09-03 23:06:30.837  INFO 80264 --- [nio-8090-exec-6] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test1---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
2024-09-03 23:06:30.852  INFO 80264 --- [nio-8090-exec-6] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
2024-09-03 23:06:30.856  INFO 80264 --- [nio-8090-exec-6] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 23:06:30.856  INFO 80264 --- [nio-8090-exec-6] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test2---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
2024-09-03 23:06:30.858  INFO 80264 --- [nio-8090-exec-6] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
```



#### 8、调用 test3 数据源接口 /test3

这里使用的是 EntityManagerFactory 动态创建 Entitymanager/Repository 接口查询的。【可以动态切换数据源】

```java
/**
 * 【成功】多数据源切换案例三: 使用注入的 EntityManagerFactory，重新手动创建EntityManager/Session
 * 或者使用 JpaRepositoryFactory + EntityManagerFactory 创建 repository
 * 【结论】最后发现都能正确的切换数据源，并且查询数据正确。
 * 【注意】使用此方式需要手动事务处理，声明式事务会失效
 */
@RequestMapping("/test3")
public List<UserInfo> test3() {
    DynamicDataSourceContextHolder.setDataSource("test1");
    EntityManager entityManager1 = entityManagerFactory.createEntityManager();
    entityManager1.createQuery("from UserInfo", UserInfo.class).getResultList().forEach(System.out::println);
    JpaRepositoryFactory jpaRepositoryFactory1 = new JpaRepositoryFactory(entityManager1);
    jpaRepositoryFactory1.getRepository(UserInfoRepository.class).findAll().forEach(System.out::println);
    entityManager1.close();
    DynamicDataSourceContextHolder.clearDataSource();

    DynamicDataSourceContextHolder.setDataSource("test2");
    EntityManager entityManager2 = entityManagerFactory.createEntityManager();
    entityManager2.createQuery("from UserInfo", UserInfo.class).getResultList().forEach(System.out::println);
    JpaRepositoryFactory jpaRepositoryFactory2 = new JpaRepositoryFactory(entityManager2);
    jpaRepositoryFactory2.getRepository(UserInfoRepository.class).findAll().forEach(System.out::println);
    entityManager1.close();
    DynamicDataSourceContextHolder.clearDataSource();

    DynamicDataSourceContextHolder.setDataSource("test3");
    EntityManager entityManager3 = entityManagerFactory.createEntityManager();
    entityManager3.createQuery("from UserInfo", UserInfo.class).getResultList().forEach(System.out::println);
    JpaRepositoryFactory jpaRepositoryFactory3 = new JpaRepositoryFactory(entityManager3);
    UserInfoRepository userInfoRepository3 = jpaRepositoryFactory3.getRepository(UserInfoRepository.class);
    List<UserInfo> userInfos3 = userInfoRepository3.findAll();
    userInfos3.forEach(System.out::println);
    entityManager3.close();
    DynamicDataSourceContextHolder.clearDataSource();

    return userInfos3;
}
```

```shell
curl -X GET "localhost:8090/test3"
[{"name":"Jone","type":"test3","id":1},
{"name":"Kath","type":"test3","id":2},
{"name":"Tom","type":"test3","id":3},
{"name":"Sandy","type":"test3","id":4},
{"name":"Oliver","type":"test3","id":5}]

###### 控制台输出的内容 ######
2024-09-03 23:34:59.669  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test1---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
2024-09-03 23:34:59.673  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test1)
UserInfo(Id=2, name=Kath, type=test1)
UserInfo(Id=3, name=Tom, type=test1)
UserInfo(Id=4, name=Sandy, type=test1)
UserInfo(Id=5, name=Oliver, type=test1)
2024-09-03 23:34:59.724  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 23:34:59.724  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test2---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
2024-09-03 23:34:59.724  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test2---------------
UserInfo(Id=1, name=Jone, type=test2)
UserInfo(Id=2, name=Kath, type=test2)
UserInfo(Id=3, name=Tom, type=test2)
UserInfo(Id=4, name=Sandy, type=test2)
UserInfo(Id=5, name=Oliver, type=test2)
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test2)
UserInfo(Id=2, name=Kath, type=test2)
UserInfo(Id=3, name=Tom, type=test2)
UserInfo(Id=4, name=Sandy, type=test2)
UserInfo(Id=5, name=Oliver, type=test2)
2024-09-03 23:34:59.742  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
2024-09-03 23:34:59.742  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test3---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
2024-09-03 23:34:59.742  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test3---------------
UserInfo(Id=1, name=Jone, type=test3)
UserInfo(Id=2, name=Kath, type=test3)
UserInfo(Id=3, name=Tom, type=test3)
UserInfo(Id=4, name=Sandy, type=test3)
UserInfo(Id=5, name=Oliver, type=test3)
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
UserInfo(Id=1, name=Jone, type=test3)
UserInfo(Id=2, name=Kath, type=test3)
UserInfo(Id=3, name=Tom, type=test3)
UserInfo(Id=4, name=Sandy, type=test3)
UserInfo(Id=5, name=Oliver, type=test3)
2024-09-03 23:34:59.755  INFO 80264 --- [nio-8090-exec-7] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
```



### 4、动态数据源使用【AOP切换】

1、增加 AOP 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2、定义数据源注解

```java
package com.example.annotation;

import java.lang.annotation.*;

/**
 * 数据源切换扫描注解
 */
@Documented
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface DS {
    String value() default "default";
}
```

3、定义数据源切面

```java
package com.example.aop;

import com.example.annotation.DS;
import com.example.config.DynamicDataSourceContextHolder;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import java.lang.reflect.Method;

@Slf4j
@Aspect
@Component
public class DataSourceAspect {
    @Pointcut("@annotation(com.example.annotation.DS)")
    public void dataSourcePointCut() {
    }

    @Around("dataSourcePointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        DS DS = method.getAnnotation(DS.class);
        log.info("注解的拦截方法名注解内容前：{}", DS.value());
        DynamicDataSourceContextHolder.setDataSource(DS.value());
        try {
            return point.proceed();
        } finally {
            DynamicDataSourceContextHolder.clearDataSource();
        }
    }
}
```

4、使用 AOP 进行数据源切换

```java
package com.example.controller;

import com.example.annotation.DS;
import com.example.model.UserInfo;
import com.example.repository.UserInfoRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;
import javax.persistence.EntityManagerFactory;
import javax.persistence.PersistenceUnit;
import java.util.List;

/**
 * 由于SpringMVC默认启用 “Open Session in View” 模式。
 * 这意味着在处理一个请求时，Hibernate 会在整个请求的生命周期内保持一个持久化的Session或EntityManager.
 * 虽然我们手动切换了数据源，但是在JPA事务中依旧是去获取当前请求线程中已经创建的EntityManager.
 * 所以会导致一个请求进来, 只有第一次数据源切换成功, 之后的切换都是无效的.
 * <p>
 * 详解可以参考：
 * 问题 1：SpringBoot整合JPA多数据源时，一个请求中无法无法多次切换数据源。可是使用@PostConstruct注释的方法却可以多次切换数据源，这是为什么？
 * 问题 2：SpringBoot整合JPA多数据源时，一个请求中无法无法多次切换数据源。可是整合Mybatis多数据源时，一个请求中却可以切换多次数据源，这是为什么？
 * 问题 3：spring.jpa.open-in-view = false 配置的解释
 */
@Slf4j
@RestController
public class MultiControllerAop {
    /**
     * 如果想要多次切换数据源可以使用EntityManagerFactory创建 EntityManager/Repository
     */
    @PersistenceUnit
    EntityManagerFactory entityManagerFactory;
    @Resource
    UserInfoRepository userInfoRepository;

    @DS("test1")
    @RequestMapping("/test11")
    public List<UserInfo> slave11() {
        return userInfoRepository.findAll();
    }

    @DS("test2")
    @RequestMapping("/test22")
    public List<UserInfo> slave22() {
        return userInfoRepository.findAll();
    }

    @DS("test3")
    @RequestMapping("/test33")
    public List<UserInfo> slave33() {
        return userInfoRepository.findAll();
    }
}
```

5、查询不同数据源中数据的接口【注意：AOP 注解这种方式只能切换一次数据源，也没办法多次切换数据源，如果想多次切换数据源，需要把方法分割到多个小方法中去，然后每个方法上加上切换数据源的注解，并且还需要改造Repository/EntityManager使用方式，必须要使用 EntityManagerFactory】

```shell
curl -X GET "localhost:8090/test11"
[{"name":"Jone","type":"test1","id":1},
{"name":"Kath","type":"test1","id":2},
{"name":"Tom","type":"test1","id":3},
{"name":"Sandy","type":"test1","id":4},
{"name":"Oliver","type":"test1","id":5}]
###### test1接口 控制台输出的内容 ######
2024-09-03 23:57:14.037  INFO 80264 --- [nio-8090-exec-1] com.example.aop.DataSourceAspect         : 注解的拦截方法名注解内容前：test1
2024-09-03 23:57:14.038  INFO 80264 --- [nio-8090-exec-1] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test1---------------
2024-09-03 23:57:14.039  INFO 80264 --- [nio-8090-exec-1] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test1---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
2024-09-03 23:57:14.052  INFO 80264 --- [nio-8090-exec-1] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------


curl -X GET "localhost:8090/test22"
[{"name":"Jone","type":"test2","id":1},
{"name":"Kath","type":"test2","id":2},
{"name":"Tom","type":"test2","id":3},
{"name":"Sandy","type":"test2","id":4},{
"name":"Oliver","type":"test2","id":5}]
###### test2接口 控制台输出的内容 ######
2024-09-04 00:04:48.723  INFO 80264 --- [nio-8090-exec-4] com.example.aop.DataSourceAspect         : 注解的拦截方法名注解内容前：test2
2024-09-04 00:04:48.725  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test2---------------
2024-09-04 00:04:48.727  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test2---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
2024-09-04 00:04:48.741  INFO 80264 --- [nio-8090-exec-4] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------


curl -X GET "localhost:8090/test33"
[{"name":"Jone","type":"test3","id":1},
{"name":"Kath","type":"test3","id":2},
{"name":"Tom","type":"test3","id":3},
{"name":"Sandy","type":"test3","id":4},
{"name":"Oliver","type":"test3","id":5}]
###### test3接口 控制台输出的内容 ######
2024-09-04 00:05:47.451  INFO 80264 --- [nio-8090-exec-2] com.example.aop.DataSourceAspect         : 注解的拦截方法名注解内容前：test3
2024-09-04 00:05:47.451  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------切换数据源：test3---------------
2024-09-04 00:05:47.453  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------当前数据源：test3---------------
Hibernate: select userinfo0_.id as id1_1_, userinfo0_.name as name2_1_, userinfo0_.type as type3_1_ from user_info userinfo0_
2024-09-04 00:05:47.462  INFO 80264 --- [nio-8090-exec-2] c.e.c.DynamicDataSourceContextHolder     : -----------重置数据源：master---------------
```



### 5、多次切换数据源失效原因分析及解决方案

#### 1、两种解决方案

问题：配置动态数据源之后，同一个请求，不同事务无法多次切换数据源【同一个事务中就更加不可能多次切换数据源了，这个自己分析理解】。

***

解决方案一：spring.jpa.open-in-view=false

原因：spring.jpa.open-in-view（默认为true），JPA 方法执行完成之后，并没有释放数据库连接（需要等到 Controlle r方法执行完成才会释放，具体源码可以查看拦截器 OpenEntityManagerInViewInterceptor，这个主要是整合 SpringMVC 默认配置的）

这里还有一点要注意数据源的切换是通过 AbstractRoutingDataSource 的 determineCurrentLookupKey() 实现，数据库建立连接时 getConnection 才会触发此方法，所以同一个事务中应该是触发不了多次的。

***

解决方案二：默认 spring.jpa.open-in-view=true 时，我们可以使用 EntityManagerFactory 创建 EntityManager/Repository 接口来使用，需要手动管理事务。

```java
// 先切换数据源
DynamicDataSourceContextHolder.setDataSource("test1");
// 使用entityManagerFactory创建EntityManager/Repository【如果update/save需要手动开启事务】
EntityManager em1 = entityManagerFactory.createEntityManager();
// 使用entityManagerFactory创建XxxRepository
JpaRepositoryFactory jrf1 = new JpaRepositoryFactory(em1);
XxxRepository repository1 = jrf1.getRepository(XxxRepository.class);
// 执行操作...

// 在此切换数据源
DynamicDataSourceContextHolder.setDataSource("test2");
// 使用entityManagerFactory创建EntityManager/Repository【如果update/save需要手动开启事务】
EntityManager em2 = entityManagerFactory.createEntityManager();
// 使用entityManagerFactory创建XxxRepository
JpaRepositoryFactory jrf2 = new JpaRepositoryFactory(em2);
XxxRepository repository2 = jrf2.getRepository(XxxRepository.class);
// 执行操作...
```



#### 2、JPA 数据源切换失效分析及扩展

##### 问题 1：SpringBoot整合JPA多数据源时，一个请求中无法多次切换数据源。可是使用@PostConstruct注释的方法却可以多次切换数据源，这是为什么？

> ChatGPT 解答：

在 Spring Boot 中使用 JPA 多数据源时，通常一个请求中无法多次切换数据源，但如果在 @PostConstruct 注解的方法中，却可以多次切换数据源。这种现象与 Spring 的事务管理机制、EntityManager 的生命周期、@PostConstruct 方法的执行上下文有关。

**1、事务管理的影响**:

- 在一个 HTTP 请求中，Spring 会为该请求创建一个事务上下文，通常会绑定到一个特定的数据源上。在这个上下文中，切换数据源可能会导致事务不一致或异常。但是在 @PostConstruct 方法中，由于它是在请求上下文之外执行的，因此可以自由地切换数据源，而不会受到当前事务的限制。
- 在一个 HTTP 请求的生命周期中，Spring 通常会为请求内的操作开启事务（尤其是当使用 @Transactional 注解时）。一旦事务开始，Spring 的事务管理器会绑定当前线程的数据库连接，并且会一直使用这个连接，直到事务结束。这意味着在一个事务中，你无法切换数据源，因为事务管理器已经绑定了最初的数据源。
- 多次切换数据源的尝试会被忽略，导致整个请求期间只使用最初绑定的数据源。

**2、@PostConstruct** **方法的执行上下文**:

- @PostConstruct 注解的方法在 Spring 容器初始化完成后会被调用，此时并没有开启事务，也没有 HTTP 请求上下文的约束。由于没有事务管理的干扰，数据源切换完全取决于你在方法内的逻辑。
- 在 @PostConstruct 方法中，切换数据源的逻辑直接影响每次数据库操作的实际数据源，因此可以多次切换数据源并成功执行。

**3、EntityManager 生命周期的管理**

- 在 JPA 中，EntityManager 通常与特定的数据源绑定，并且在一个请求中，Spring 会管理这个 EntityManager 的生命周期。如果在请求中尝试切换数据源，可能会导致 EntityManager 的状态不一致。
- 在 @PostConstruct 方法中，你可以手动创建和管理不同的数据源的 EntityManager，这使得你可以在这个方法中自由切换数据源。

**4、为什么 @PostConstruct 可以多次切换数据源**

- **没有事务绑定**: 在 @PostConstruct 方法中，Spring 并没有开启事务，也不会进行事务管理。由于没有事务的约束，数据源切换不会受到事务管理器的限制，每次切换都会生效。
- **单线程执行**: @PostConstruct 方法是在单线程中执行的，且不会与其他请求并发执行。数据源的上下文是通过 ThreadLocal 存储的，因此在方法内的每次切换都是独立的，不会受到其他线程的干扰。

**5、结论：**

- 在一个请求中无法多次切换数据源的根本原因是 Spring 的事务管理机制，它在事务开始时锁定了一个数据源。@PostConstruct 方法没有事务管理的限制，因此可以自由地在方法内部多次切换数据源，并且每次切换都会生效。
- 在 SpringBoot 中，@PostConstruct 方法的执行不受当前请求的事务上下文限制，因此可以在这个方法中自由切换数据源。而在一个请求中切换数据源时，由于事务管理和 EntityManager 的绑定，可能会导致不一致性和异常。因此，理解这两者之间的区别是关键。
- 如果你需要在一个请求中多次切换数据源，可能需要考虑手动管理事务、分离操作到不同的事务中，或者结合使用 MyBatis 等其他技术来绕过事务管理器的限制。

> DuckDuckGo AI Chat 解答：

***

##### 问题 2：SpringBoot整合JPA多数据源时，一个请求中无法多次切换数据源。可整合Mybatis多数据源时，一个请求中却可以切换多次数据源，这是为什么？

> ChatGPT 解答：

在 Spring Boot 中，当整合 JPA 和 MyBatis 时，虽然它们都可以实现多数据源的配置和切换，但由于两者的工作机制不同，导致在同一个请求中切换数据源时，表现也有所不同。具体来说，JPA 和 MyBatis 在处理事务和数据源上下文的方式上存在差异，这就解释了为什么在一个请求中，JPA 无法多次切换数据源，而 MyBatis 可以。

JPA 的事务与数据源处理机制

1. 事务管理绑定：
   - Spring 的事务管理器（通常是 JpaTransactionManager）在事务开始时，会绑定当前线程的数据源（连接）到事务中。一旦事务开始，所有的数据库操作都会使用最初绑定的连接，事务管理器不会轻易切换数据源。
   - 即使你在同一个请求中多次切换数据源，JPA 依赖的 Hibernate 仍会使用第一次绑定的数据库连接，导致无法多次切换数据源。

2. Hibernate 的 Session 缓存：
   - Hibernate 的 Session 会缓存实体和查询结果，并且这个缓存通常是绑定到当前事务的。这意味着在同一个事务中，即使你切换了数据源，Hibernate 可能仍然会使用缓存中的数据，而不是从新切换的数据源中获取。

3. EntityManager 管理：
   - EntityManager 是由 JPA 提供的接口，用来管理持久化上下文。EntityManager 在 Spring 中通常和事务一起管理。一个事务开始后，EntityManager 就绑定到当前线程，而这个绑定的数据源在整个事务生命周期中是不变的，除非开启新事务。

MyBatis 的事务与数据源处理机制

1. MyBatis 与 Spring 的集成：
   - MyBatis 是一个更轻量级的 ORM 框架，事务管理相对更灵活。MyBatis 在执行 SQL 语句时，通过 SqlSession 获取当前的数据源，执行 SQL 语句，并返回结果。
   - 在 Spring 环境下，MyBatis 可以通过 Spring 的 DataSourceTransactionManager 来管理事务，但它并不会像 JPA 那样严格绑定数据源。每次 SqlSession 执行时，都会重新从当前的 ThreadLocal 获取数据源，因此 MyBatis 可以在同一个请求中多次切换数据源。

2. 数据源上下文的管理：
   - MyBatis 在执行每个 SQL 语句时，都是临时获取当前的数据源。这意味着，如果你在同一个请求中切换数据源，MyBatis 会从当前的 `ThreadLocal` 获取最新的数据源，并且每次操作都会使用最新的连接。
   - 这种机制使得 MyBatis 能够在一个请求中多次切换数据源，而不会受到事务绑定的影响。

总结

- **JPA**：由于 JPA 的事务管理和 EntityManager 绑定在事务开始时就确定了数据源，并且在同一个事务中会一直使用这个数据源，这就导致了无法在一个请求中多次切换数据源。
- **MyBatis**：MyBatis 更灵活，每次执行 SQL 时都会获取最新的数据源，不会受事务绑定的限制，因此在一个请求中可以多次切换数据源。

解决思路

如果你希望在 JPA 中实现类似于 MyBatis 的多次数据源切换，可以考虑以下几种方式：

1. **手动管理事务**：手动控制事务的开启和关闭，每次操作都切换数据源并开启新事务。
2. **分割业务逻辑**：将不同的数据源操作分离到不同的方法或服务中，并使用 `@Transactional(propagation = Propagation.REQUIRES_NEW)` 来强制开启新事务，从而允许在一个请求中切换数据源。
3. **混合使用 MyBatis 和 JPA**：如果业务逻辑允许，可以考虑在需要频繁切换数据源的场景下使用 MyBatis，而在需要强大的 ORM 支持时使用 JPA。这样可以结合两者的优点。

通过理解 JPA 和 MyBatis 的事务和数据源管理机制，选择适合你应用的方式进行实现。

> DuckDuckGo AI Chat 解答：

在Spring Boot中整合JPA和MyBatis时，处理多数据源的方式有所不同，这导致了在一个请求中切换数据源的能力差异。

JPA多数据源

1. **事务管理**：JPA通常与Spring的声明式事务管理紧密集成。每个事务通常会绑定到一个特定的数据源上，这意味着在一个事务中切换数据源会导致事务管理的复杂性和不一致性。因此，JPA在一个请求中切换数据源的支持较弱。

2. **EntityManager**：JPA使用EntityManager来管理实体的生命周期和数据库操作。每个`EntityManager`通常与一个特定的数据源绑定，因此在同一个请求中频繁切换数据源可能会导致EntityManager的状态不一致。

3. **配置复杂性**：JPA的多数据源配置相对复杂，通常需要为每个数据源配置不同的EntityManagerFactory和TransactionManager，这使得在同一请求中动态切换数据源变得更加困难。

MyBatis多数据源

1. **灵活性**：MyBatis的设计更为灵活，允许在同一请求中通过`SqlSession`动态选择数据源。你可以在代码中直接指定使用哪个数据源的`SqlSessionFactory`，这使得在一个请求中切换数据源变得简单。

2. **事务管理**：MyBatis的事务管理相对独立于Spring的事务管理，可以更容易地在不同的数据源之间切换，而不会影响到其他数据源的事务状态。

3. **配置简单**：MyBatis的多数据源配置相对简单，通常只需要在`SqlSessionFactory`中指定不同的数据源即可，这使得在同一请求中切换数据源的实现更加直接。

总结

因此，JPA在多数据源切换方面的限制主要是由于其与事务管理的紧密集成和`EntityManager`的绑定，而MyBatis则提供了更大的灵活性和简单的配置，使得在同一请求中切换数据源成为可能。如果需要在JPA中实现类似的功能，可能需要手动管理`EntityManager`的切换和事务的处理，这会增加代码的复杂性。

***

##### 问题 3：spring.jpa.open-in-view = false 配置的解释

spring.jpa.open-in-view=false 是 SpringBoot 中的一个配置项，主要与 JPA 的实体管理和事务处理有关。这个配置项控制了 “Open Session in View” 模式的启用与否。

**配置解释**

**1、spring.jpa.open-in-view=true（默认值）**：

- **作用**：启用 “Open Session in View” 模式。这意味着在处理一个请求时，Hibernate 会在整个请求的生命周期内保持一个持久化的Session或EntityManager（即打开的数据库连接）。因此，即使在视图层（例如，模板引擎或@ResponseBody生成的JSON响应）访问延迟加载的实体属性时，Hibernate 仍然可以从数据库中加载数据。
- **优点**：允许在视图层或控制器返回响应时访问延迟加载的关联数据，而不抛出 LazyInitializationException。
- **缺点**：由于Session或EntityManager在整个请求过程中都保持打开状态，可能会导致数据库连接的长期占用，增加了内存泄漏和性能问题的风险。此外，使用这种模式可能掩盖了设计中的潜在问题，如在视图层进行数据库访问，而这是不推荐的实践。

**2、spring.jpa.open-in-view=false**：

- **作用**：禁用 “Open Session in View” 模式。在这种情况下，Session或EntityManager只在事务范围内打开，并在事务结束后立即关闭。这意味着在事务之外（通常在视图层）访问延迟加载的实体属性将抛出LazyInitializationException。
- **优点**：更符合事务的生命周期原则，有助于更清晰的架构设计，避免在视图层进行数据库访问，减少了内存泄漏的风险和数据库连接的长期占用问题。
- **缺点**：如果在视图层或控制器中访问未完全加载的实体，会导致LazyInitializationException。因此，开发者需要提前加载实体的关联数据，通常通过使用JOIN FETCH或设置合适的加载策略（如 Eager Fetching）。

**3、什么时候使用 spring.jpa.open-in-view=false？**

1. **性能优化**：如果你发现数据库连接过多，或内存占用过高，可以考虑禁用 “Open Session in View”，以减少数据库连接的时间和内存消耗。
2. **架构设计**：在设计中，你希望确保所有数据库访问都发生在事务范围内，避免在视图层进行数据库操作。
3. **问题排查**：当遇到LazyInitializationException时，可以通过禁用 “Open Session in View” 来强制在事务中加载所有需要的数据，从而找出问题所在。

**4、总结**

- true（默认）：方便但可能有性能问题，适用于简单场景或开发阶段。
- false：推荐用于生产环境，有助于更清晰的代码结构和更高的性能。
- 根据你的项目需求，可以选择启用或禁用这个配置。
