# SpringBoot 整合 H2 数据库

## 1、SpringBoot  JPA H2 整合

一、H2的依赖包及版本

```xml
<!-- https://mvnrepository.com/artifact/com.h2database/h2 -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.220</version>
</dependency>
```

二、添加H2以及相关依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.220</version>
</dependency>
```

三、配置H2相关参数

```properties
## 数据库连接设置
spring.datasource.driverClassName = org.h2.Driver
spring.datasource.url = jdbc:h2:mem:test_mem
spring.datasource.username = sa
spring.datasource.password =

## H2 Web Console设置
# spring.h2.console.path: 进行该配置，你就可以通过YOUR_URL/h2访问h2 web consloe。YOUR_URL是你程序的访问URl。
# spring.h2.console.settings.web-allow-others: 配置后，Web控制台就可以在远程访问了。否则只能在本机访问。
# spring.h2.console.enabled: 开启时就会启动h2 Web控制台。默认就是true。
spring.h2.console.path = /h2
spring.h2.console.settings.web-allow-others = true
spring.h2.console.enabled = true

## 数据初始化设置
# spring.sql.init.platform: 表明使用的数据库平台是h2
# spring.sql.init.schema-locations: 每次启动程序, 程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。
# spring.sql.init.data-locations: 每次启动程序, 程序都会运行resources/db/data.sql文件，对数据库的数据操作。
spring.sql.init.platform = h2
spring.sql.init.schema-locations = classpath:db/schema.sql
spring.sql.init.data-locations = classpath:db/data.sql

## JPA 配置
# spring.jpa.show-sql: 启用SQL语句的日志记录
# spring.jpa.hibernate.ddl-auto: 设置ddl模式
spring.jpa.show-sql = true
spring.jpa.hibernate.ddl-auto = update
```

数据库支持多种连接模式和连接设置，不同的连接模式和连接设置是通过不同的URL来区分的，URL中的设置是不区分大小写。

四、其中在`resources/db`下创建表结构schema.sql：resources/db/schema.sql

```sql
DROP TABLE IF EXISTS USER_INFO;
CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255));
```

五、以及数据文件 data.sql，作为初始化数据：resourses/db/data.sql

```sql
-- 字段都是大写的
INSERT INTO USER_INFO VALUES(1, 'Hello');
INSERT INTO USER_INFO VALUES(2, 'World');
INSERT INTO USER_INFO VALUES(3, '乔丹');
INSERT INTO USER_INFO VALUES(4, '科比');
INSERT INTO USER_INFO VALUES(5, '艾佛森');
INSERT INTO USER_INFO VALUES(6, '麦迪');
```

六、实体关联表

```java
import lombok.Data;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Data
@Entity
@Table(name = "USER_INFO")
public class UserInfo {
    @Id
    @Column(name = "USER_ID")
    private int userId;
    @Column(name = "USER_NAME")
    private String userName;
}
```

七、创建UserInfoRepository继承JpaRepository

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
@Repository
public interface UserInfoRepository extends JpaRepository<UserInfo, Integer> {
}
```

八、测试查询数据

```java
package com.example;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import javax.annotation.PostConstruct;

@SpringBootApplication
public class SpringBootH2MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootH2MainApplication.class, args);
    }

    @Autowired
    UserInfoRepository userInfoRepository;

    @PostConstruct
    public void init() {
        userInfoRepository.findAll().forEach(System.out::println);
    }
}
```

```java
Hibernate: select userinfo0_.user_id as user_id1_0_, userinfo0_.user_name as user_nam2_0_ from user_info userinfo0_
UserInfo(userId=1, userName=Hello)
UserInfo(userId=2, userName=World)
UserInfo(userId=3, userName=乔丹)
UserInfo(userId=4, userName=科比)
UserInfo(userId=5, userName=艾佛森)
UserInfo(userId=6, userName=麦迪)
```



## 2、SpringBoot  Mybatis H2 整合

一、添加H2以及相关依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.220</version>
</dependency>
```

二、配置H2相关参数

```properties
## 数据库连接设置
spring.datasource.driverClassName = org.h2.Driver
spring.datasource.url = jdbc:h2:mem:test_mem
spring.datasource.username = sa
spring.datasource.password =

## H2 Web Console设置
# spring.h2.console.path: 进行该配置，你就可以通过YOUR_URL/h2访问h2 web consloe。YOUR_URL是你程序的访问URl。
# spring.h2.console.settings.web-allow-others: 配置后，Web控制台就可以在远程访问了。否则只能在本机访问。
# spring.h2.console.enabled: 开启时就会启动h2 Web控制台。默认就是true。
spring.h2.console.path = /h2
spring.h2.console.settings.web-allow-others = true
spring.h2.console.enabled = true

## 数据初始化设置
# spring.sql.init.platform: 表明使用的数据库平台是h2
# spring.sql.init.schema-locations: 每次启动程序, 程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。
# spring.sql.init.data-locations: 每次启动程序, 程序都会运行resources/db/data.sql文件，对数据库的数据操作。
spring.sql.init.platform = h2
spring.sql.init.schema-locations = classpath:db/schema.sql
spring.sql.init.data-locations = classpath:db/data.sql

## Mybatis配置, 开启驼峰转换和SQL打印
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

数据库支持多种连接模式和连接设置，不同的连接模式和连接设置是通过不同的URL来区分的，URL中的设置是不区分大小写。

三、其中在`resources/db`下创建表结构schema.sql：resources/db/schema.sql

```sql
DROP TABLE IF EXISTS USER_INFO;
CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255));
```

四、以及数据文件 data.sql，作为初始化数据：resourses/db/data.sql

```sql
-- 字段都是大写的
INSERT INTO USER_INFO VALUES(1, 'Hello');
INSERT INTO USER_INFO VALUES(2, 'World');
INSERT INTO USER_INFO VALUES(3, '乔丹');
INSERT INTO USER_INFO VALUES(4, '科比');
INSERT INTO USER_INFO VALUES(5, '艾佛森');
INSERT INTO USER_INFO VALUES(6, '麦迪');
```

五、实体关联表

```java
import lombok.Data;
@Data
public class UserInfo {
    private int userId;
    private String userName;
}
```

六、创建UserInfoMapper接口

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import java.util.List;
@Mapper
public interface UserInfoMapper {
    @Select("SELECT * FROM USER_INFO")
    List<UserInfo> findAll();
}
```

七、测试查询数据

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import javax.annotation.PostConstruct;
import java.util.List;

@SpringBootApplication
public class SpringBootH2MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootH2MainApplication.class, args);
    }

    @Autowired
    UserInfoMapper userInfoMapper;

    @PostConstruct
    public void init() {
        List<UserInfo> userInfos = userInfoMapper.findAll();
        userInfos.forEach(System.out::println);
    }
}
```

```java
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@12fcc71f] was not registered for synchronization because synchronization is not active
JDBC Connection [HikariProxyConnection@1846307450 wrapping conn0: url=jdbc:h2:mem:test_mem user=SA] will not be managed by Spring
==>  Preparing: SELECT * FROM USER_INFO
==> Parameters: 
<==    Columns: USER_ID, USER_NAME
<==        Row: 1, Hello
<==        Row: 2, World
<==        Row: 3, 乔丹
<==        Row: 4, 科比
<==        Row: 5, 艾佛森
<==        Row: 6, 麦迪
<==      Total: 6
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@12fcc71f]
UserInfo(userId=1, userName=Hello)
UserInfo(userId=2, userName=World)
UserInfo(userId=3, userName=乔丹)
UserInfo(userId=4, userName=科比)
UserInfo(userId=5, userName=艾佛森)
UserInfo(userId=6, userName=麦迪)
```



## 3、SpringBoot  JdbcTemplate H2 整合

一、添加H2以及相关依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.220</version>
</dependency>
```

二、配置H2相关参数

```properties
## 数据库连接设置
spring.datasource.driverClassName = org.h2.Driver
spring.datasource.url = jdbc:h2:mem:test_mem
spring.datasource.username = sa
spring.datasource.password =

## H2 Web Console设置
# spring.h2.console.path: 进行该配置，你就可以通过YOUR_URL/h2访问h2 web consloe。YOUR_URL是你程序的访问URl。
# spring.h2.console.settings.web-allow-others: 配置后，Web控制台就可以在远程访问了。否则只能在本机访问。
# spring.h2.console.enabled: 开启时就会启动h2 Web控制台。默认就是true。
spring.h2.console.path = /h2
spring.h2.console.settings.web-allow-others = true
spring.h2.console.enabled = true

## 数据初始化设置
# spring.sql.init.platform: 表明使用的数据库平台是h2
# spring.sql.init.schema-locations: 每次启动程序, 程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。
# spring.sql.init.data-locations: 每次启动程序, 程序都会运行resources/db/data.sql文件，对数据库的数据操作。
spring.sql.init.platform = h2
spring.sql.init.schema-locations = classpath:db/schema.sql
spring.sql.init.data-locations = classpath:db/data.sql

# 开启 JdbcTemplate SQL 打印
logging.level.org.springframework.jdbc.core.JdbcTemplate=DEBUG
```

数据库支持多种连接模式和连接设置，不同的连接模式和连接设置是通过不同的URL来区分的，URL中的设置是不区分大小写。

三、其中在`resources/db`下创建表结构schema.sql：resources/db/schema.sql

```sql
DROP TABLE IF EXISTS USER_INFO;
CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY, USER_NAME VARCHAR(255));
```

四、以及数据文件 data.sql，作为初始化数据：resourses/db/data.sql

```sql
-- 字段都是大写的
INSERT INTO USER_INFO VALUES(1, 'Hello');
INSERT INTO USER_INFO VALUES(2, 'World');
INSERT INTO USER_INFO VALUES(3, '乔丹');
INSERT INTO USER_INFO VALUES(4, '科比');
INSERT INTO USER_INFO VALUES(5, '艾佛森');
INSERT INTO USER_INFO VALUES(6, '麦迪');
```

五、测试查询数据

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import javax.annotation.PostConstruct;
import java.util.List;
import lombok.Data;

@SpringBootApplication
public class SpringBootH2DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootH2DemoApplication.class, args);
    }

    @Autowired
    JdbcTemplate jdbcTemplate;

    @PostConstruct
    public void init() {
        List<UserInfo> userInfos = jdbcTemplate.query("SELECT * FROM USER_INFO", 
                                                      new BeanPropertyRowMapper<>(UserInfo.class));
        userInfos.forEach(System.out::println);
    }
    
    @Data
    public static class UserInfo {
        private int userId;
        private String userName;
    }
}
```

```java
2023-08-07 12:01:42.355 DEBUG 29192 --- [main] o.s.jdbc.core.JdbcTemplate: Executing SQL query [SELECT * FROM USER_INFO]
UserInfo(userId=1, userName=Hello)
UserInfo(userId=2, userName=World)
UserInfo(userId=3, userName=乔丹)
UserInfo(userId=4, userName=科比)
UserInfo(userId=5, userName=艾佛森)
UserInfo(userId=6, userName=麦迪)
```



# 参考资料 & 鸣谢

1. H2数据库配置及相关使用方式一站式介绍：https://blog.csdn.net/weixin_44344089/article/details/129031552
2. Springboot 集成轻量级内存数据库H2：http://xiaoyuge.work/springboot-chapter3/index.html
4. SpringBoot集成h2：https://www.cnblogs.com/jin-zhe/archive/2018/01/05/8203347.html

