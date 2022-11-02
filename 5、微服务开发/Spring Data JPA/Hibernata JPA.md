# Hibernate JPA 快速入门

## 1、ORM 的介绍

1、什么是 ORM？

ORM（Object-Relational Mapping） 表示对象关系映射。在面向对象的软件开发中，通过ORM，就可以把对象映射到关系型数据库中。只要有一套程序能够做到建立对象与数据库的关联，操作对象就可以直接操作数据库数据，就可以说这套程序实现了ORM对象关系映射。简单的说：ORM 就是建立实体类和数据库表之间的关系，从而达到操作实体类就相当于操作数据库表的目的（就是说ORM会把数据表映射成一个Java对象，使开发人员可以关注Java程序）



2、为什么要使用 ORM？

当实现一个应用程序时（不使用O/R Mapping），我们可能会写特别多数据访问层的代码（各种各样的DAO类），从数据库增删改查等操作，而这些代码都是重复的。但使用ORM框架则会大大减少重复性代码。对象关系映射（Object Relational Mapping，简称ORM），主要实现程序对象到关系数据库数据的映射。



3、ORM 的优缺点

优点：

1. 提高开发效率，降低开发成本  （减少了DAO类的操作）
2. 使开发更加对象化 （直接在实体类domain中来映射关系）
3. 可移植
4. 可以很方便地引入数据缓存之类的附加功能 

缺点： 

1. 自动化进行关系数据库的映射需要消耗系统性能（消耗的性能可以忽略不记）
2. 在处理多表联查、where条件复杂之类的查询时，ORM的语法会变得复杂（这才是最致命的缺点）



4、常见ORM思想的框架：JDBC、Hibernate、MyBatis、TopLink、JPA（JPA对ORM框架的再一次封装，这里是一套规范）

1. JDBC：其实JDBC是最原生的API，支持连接并操作各种关系型数据库，也就是说可以用JDBC完成ORM思想的程序编写，如一些有ORM思想的框架底层都调用JDBC，所有这个JDBC我对其理解为ORM思想
2. Hibernate：这个框架就不用多说了，完全使用ORM思想，不过现在直接使用Hibernate的少了，大多都是在JPA的封装上调用此框架
3. MyBatis：这个框架可以手动写SQL语句，也可以完成对象关系映射，其实严格上说Mybatis不完全是一个ORM框架（JPA的供应商就不支持Mybatis）



5、总结

1. JDBC：快，代码冗余、频繁的开关连接消耗性能、SQL不够灵活
2. Mybatis：小巧、方便、高效、简单、直接、半自动
3. Hibernate：强大、方便、高效、复杂、绕弯子、全自动



## 2、Hibernate JPA简介

1、认识 hibernate

Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装，它将 POJO与数据库表建立映射关系，是一个全自动的 orm 框架，hibernate 可以自动生成 SQL 语句，自动执行，使得 Java 程序员可以随心所欲的使用对象编程思维来操纵数据库。



2、认识 JPA

JPA（Java Persistence API）即Java 持久化API，是SUN公司推出的一套基于ORM的规范，内部是由一系列的接口和抽象类构成。JPA 通过JDK 5.0注解描述 对象－关系表 的映射关系，并将运行期的实体对象持久化到数据库中。



3、JPA 的优势：

- 标准化：
  JPA 是 JCP 组织发布的 Java EE 标准之一，因此任何声称符合 JPA 标准的框架都遵循同样的架构，提供相同的访问API，这保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行。
- 容器级特性的支持：
  JPA框架中支持大数据集、事务、并发等容器级事务，这使得 JPA 超越了简单持久化框架的局限，在企业应用发挥更大的作用。
- 简单方便：
  JPA的主要目标之一就是提供更加简单的编程模型：在JPA框架下创建实体和创建Java 类一样简单，没有任何的约束和限制，只需要使用 javax.persistence.Entity 进行注释，JPA的框架和接口也都非常简单，没有太多特别的规则和设计模式的要求，开发者可以很容易的掌握。JPA基于非侵入式原则设计，因此可以很容易的和其它框架或者容器集成
- 查询能力：
  JPA的查询语言是面向对象而非面向数据库的，它以面向对象的自然语法构造查询语句，可以看成是Hibernate HQL的等价物。JPA定义了独特的JPQL（Java Persistence Query Language），JPQL是EJB QL的一种扩展，它是针对实体的一种查询语言，操作对象是实体，而不是关系数据库的表，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能够提供的高级查询特性，甚至还能够支持子查询。
- 高级特性：
  JPA 中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系，这样的支持能够让开发者最大限度的使用面向对象的模型设计企业应用，而不需要自行处理这些特性在关系数据库的持久化。



4、JPA 与 Hibernate 的关系

JPA 规范本质上就是**一种ORM规范，注意不是ORM框架**，因为JPA并未提供ORM实现，它只是定义了一些规范，提供了一些编程的API接口，但具体实现则由服务厂商来提供实现。JPA示意图：

```
                                               Java代码
                                                  ⬇
                        JPA规范（SUM公司定义的，Java持久化规范，内部由一系列接口和抽象类组成）
                                                  ⬇
                             Hibernate/TopLink...其他ORM框架（它们都是实现了JPA规范）
                                                  ⬇
                                           JDBC规范（SUM公司制定）
                                                  ⬇
                                           MySQL/Oracle驱动
                                                  ⬇
                                           MySQL/Oracle数据库
```

JPA 和 Hibernate 的关系就像 JDBC 和 JDBC 驱动的关系，JPA是规范，Hibernate 除了作为ORM框架之外，它也是一种JPA实现。

PS：JPA 能取代 Hibernate 吗？正如同问 JDBC 规范可以驱动底层数据库吗？
答案是否定的，如果使用JPA规范进行数据库操作，底层需要 Hibernate 作为其实现类完成数据持久化工作。





## 3、搭建开发环境（IDEA）

> 直接使用IDEA创建JPA项目（使用Maven方式创建）：https://blog.csdn.net/jellily12/article/details/89304345
>

1、使用 IDEA 创建 maven 工程（如果缺少src/test/resources目录，手动创建和设置即可）

```cmd
├─JPA
  │  JPA.iml
  │  pom.xml
  └─src
      ├─main
      │  ├─java
      │  └─resources
      └─test
          ├─java
          └─resources
```

2、导入pom依赖

```xml
    <properties>
        <!--锁定 jdk 版本为 1.8-->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- hibernate对jpa的支持包 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.4.1.Final</version>
        </dependency>
        <!-- Mysql驱动 -->
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>
        <!-- junit4单元测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.16</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

3、建立配置文件：/resources/META-INF/persistence.xml

- 路径：配置到类路径（resources）下的 META-INF 的文件夹下，文件名：persistence.xml
- PS：由于主要在 src/test/ 下测试，所以建议在 src/test/resources/ 也建立 META-INF/persistence.xml

- IDEA创建 persistence 模板：`setting=》Editor=》file and code Template=》JPA==》Deployment descriptors=》persistenceXX.xml`

persistence.xml 文件内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
 
    <!-- name：持久化单元名称，transaction-type：持久化单元事务类型(JTA:分布式事务管理,RESOURCE_LOCAL:本地事务管理) -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!--jpa的实现方式,配置JPA服务提供商 -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <!-- 可配可不配，如果配置了顺序不能错，必须在provider之后-->
        <!--<class>com.caochenlei.hibernate.jpa.Customer</class>-->

        <!--可选配置：配置jpa实现方的配置信息-->
        <properties>
            <!-- 数据库信息配置：数据库驱动、数据库地址、数据库账户、数据库密码 -->
            <property name="hibernate.connection.driver_class" value="com.mysql.jdbc.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:mysql://127.0.0.1:3306/hibernate_jpa"/>
            <property name="hibernate.connection.username" value="root"/>
            <property name="hibernate.connection.password" value="password"/>

            <!-- 配置JPA服务提供商可选参数 -->
            <property name="hibernate.show_sql" value="true" /><!-- 自动显示sql -->
            <property name="hibernate.format_sql" value="true"/><!-- 格式化sql -->
            <!-- 自动创建数据库表：
                none        ：不会创建表
                create      : 程序运行时创建数据库表（如果有表，先删除表再创建）
                update      ：程序运行时创建表（如果有表，不会创建表）
                create-drop : 每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
                validate    : 每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。
            -->
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>
```

4、编写实体类和数据库表的映射配置（目的是达到操作实体类，就相当于操作数据库表）

```java
/**
 * 客户的实体类
 * 配置映射关系
 *   1.实体类和表的映射关系
 *      @Entity：声明实体类
 *      @Table：配置实体类和表的映射关系
 *          name : 配置数据库表的名称
 *   2.实体类中属性和表中字段的映射关系
 */
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {

    /**
     * @Id：声明主键的配置
     * @GeneratedValue:配置主键的生成策略
     *      strategy:
     *          GenerationType.IDENTITY ：自增，mysql。底层数据库必须支持自动增长（底层数据库支持的自动增长方式，对id自增）
     *          GenerationType.SEQUENCE ：序列，oracle（底层数据库必须支持序列）
     *          GenerationType.TABLE    ：jpa提供的一种机制，通过一张数据库表的形式帮助我们完成主键自增
     *          GenerationType.AUTO     ：由程序自动的帮助我们选择主键生成策略
     * @Column：配置属性和字段的映射关系
     *      name：数据库表中字段的名称
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long Id; // 客户的主键

    @Column(name = "customer_name")
    private String name; // 客户名称

    @Column(name="customer_age")
    private int age; // 客户年龄

    @Column(name="customer_sex")
    private boolean sex; // 客户性别

    @Column(name="customer_phone")
    private String phone; // 客户的联系方式

    @Column(name="customer_address")
    private String address; // 客户地址
}
```

5、对应的数据库表信息：

|      Field       |              Type              |       Comment        |
| :--------------: | :----------------------------: | :------------------: |
|   customer_id    | bigint not null auto_increment |   客户编号（主键）   |
|  customer_name   |          varchar(255)          | 客户名称（公司名称） |
|   customer_age   |            integer             |       客户年龄       |
|   customer_sex   |              bit               |       客户性别       |
|  customer_phone  |          varchar(255)          |     客户联系电话     |
| customer_address |          varchar(255)          |     客户联系地址     |

6、测试保存操作的执行

```java
    /**
     * 测试jpa的保存
     * 案例：保存一个客户到数据库中
     * Jpa的操作步骤
     * 1.加载配置文件创建工厂（实体管理器工厂）对象
     * 2.通过实体管理器工厂获取实体管理器
     * 3.获取事务对象，开启事务
     * 4.完成增删改查操作
     * 5.提交事务（回滚事务）
     * 6.释放资源
     */
    @Test
    public void testSave() {
        // 1.加载配置文件创建工厂（实体管理器工厂）对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        // 2.通过实体管理器工厂获取实体管理器
        EntityManager em = factory.createEntityManager();
        // 3.获取事务对象，开启事务
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        //4.完成增删改查操作：保存一个客户到数据库中
        Customer customer = new Customer();
        customer.setName("Sam");
        customer.setAddress("Guangzhou");
        // 保存操作
        em.persist(customer);
        // 5.提交事务
        tx.commit();
        // 6.释放资源
        em.close();
        factory.close();
    }
```

7、查看日志

```verilog
Hibernate: 
    create table tb_customer (
       customer_id bigint not null auto_increment,
        customer_address varchar(255),
        customer_age integer,
        customer_name varchar(255),
        customer_phone varchar(255),
        customer_sex bit,
        primary key (customer_id)
    ) engine=InnoDB

Hibernate: 
    insert 
    into
        tb_customer
        (customer_address, customer_age, customer_name, customer_phone, customer_sex) 
    values
        (?, ?, ?, ?, ?)
```

8、这里可以看到为我们自动生成了SQL语句，因为 xml 里这里设置的是 update，所以有表的前提下，不会再生成表

```xml
<!-- 自动创建数据库表：
    none        ：不会创建表
    create      : 程序运行时创建数据库表（如果有表，先删除表再创建）
    update      ：程序运行时创建表（如果有表，不会创建表）
    create-drop : 每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
    validate    : 每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。
-->
<property name="hibernate.hbm2ddl.auto" value="update" />
```

9、错误处理（暂无）



## 4、JPA 完整配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">

    <!--
        持久化单元：
            name：持久化单元名称
            transaction-type：持久化单元事务类型
              - JTA：分布式事务管理
              - RESOURCE_LOCAL：本地事务管理
     -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!--jpa的实现方式,配置JPA服务提供商 -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <!-- 可配可不配，如果配置了顺序不能错，必须在provider之后-->
        <!--<class>com.caochenlei.hibernate.jpa.Customer</class>-->
        <!--
            配置二级缓存时候使用的模式，可配置值有：
            - ALL：所有的实体类都被缓存
            - NONE：所有的实体类都不被缓存
            - ENABLE_SELECTIVE：标识@Cacheable(true)注解的实体类将被缓存
            - DISABLE_SELECTIVE；缓存除标识@Cacheable(false)以外的所有实体类
            - UNSPECIFIED：默认值，JPA 产品默认值将被使用
         -->
        <shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>

        <!--可选配置：配置jpa实现方的配置信息-->
        <properties>
            <!-- 数据库信息配置：数据库驱动、数据库地址、数据库账户、数据库密码 -->
            <property name="hibernate.connection.driver_class" value="com.mysql.jdbc.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:mysql://127.0.0.1:3306/hibernate_jpa"/>
            <property name="hibernate.connection.username" value="root"/>
            <property name="hibernate.connection.password" value="password"/>
            <!-- 数据库信息配置：数据库驱动、数据库地址、数据库账户、数据库密码 -->
            <!--<property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://127.0.0.1:3306/hibernate_jpa"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="password"/>-->

            <!-- 配置JPA服务提供商可选参数 -->
            <!-- 自动显示sql -->
            <property name="hibernate.show_sql" value="true" />
            <!-- 格式化sql -->
            <property name="hibernate.format_sql" value="true"/>
            <!-- 自动创建数据库表：
                none        ：不会创建表
                create      : 程序运行时创建数据库表（如果有表，先删除表再创建）
                update      ：程序运行时创建表（如果有表，不会创建表）
                create-drop : 每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
                validate    : 每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。
            -->
            <property name="hibernate.hbm2ddl.auto" value="update" />
            
            <!-- 建表使用MyISAM，默认是InnoDB，下列是MySQL5 和 MySQL8 两种方言引擎设置-->
            <!-- InnoDB引擎：org.hibernate.dialect.MySQL5InnoDBDialect、org.hibernate.dialect.MySQL8InnoDBDialect -->
            <!-- MyISAM引擎：org.hibernate.dialect.MySQL5Dialect、org.hibernate.dialect.MySQL5Dialect -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>

            <!-- Scan for annotated classes and Hibernate mapping XML files (可不配置)-->
            <!--<property name="hibernate.archive.autodetection" value="class, hbm"/>-->

            <!-- 二级缓存相关 -->
            <!-- 开启二级缓存 -->
            <property name="hibernate.cache.use_second_level_cache" value="true"/>
            <!-- 配置二级缓存处理类 -->
            <property name="hibernate.cache.region.factory_class" value="org.hibernate.cache.ehcache.EhCacheRegionFactory"/>
            <!-- 开启查询缓存，entityManager.find查询可以不配置，如果使用JPQL或SQL查询需要开启该配置 -->
            <property name="hibernate.cache.use_query_cache" value="true"/>
            <!-- 指定缓存配置文件位置,如果默认在resources下可不配置 -->
            <property name="hibernate.cache.provider_configuration" value="classpath:ehcache.xml"/>

            
            <!-- *****************************连接池的配置***************************** -->
            
            <!-- Hibernate JPA整合C3P0数据库连接池 -->
			<property name="hibernate.connection.provider_class" 
                      value="org.hibernate.connection.C3P0ConnectionProvider" />
			<!-- 数据库连接池的最小连接数 -->
			<property name="c3p0.min_size" value="5" />
			<!-- 数据库连接池的最大连接数 -->
			<property name="c3p0.max_size" value="30" />
			<!-- 最大空闲时间，60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
			<property name="c3p0.maxIdleTime" value="60" />
			<!-- 获得连接的超时时间,如果超过这个时间,会抛出异常，单位毫秒 -->
			<property name="c3p0.timeout" value="1800" />
			<!-- 最大的PreparedStatement的数量 -->
			<property name="c3p0.max_statements" value="50" />
			<!-- 每隔120秒检查连接池里的空闲连接，单位是秒 -->
			<property name="c3p0.idle_test_period" value="120" />
			<!-- 当连接池里面的连接用完的时候，C3P0一下获取的新的连接数 -->
			<property name="c3p0.acquire_increment" value="1" />
			<!-- 是否每次都验证连接是否可用 -->
			<property name="c3p0.validate" value="false" />
            

            <!-- Druid连接池配置 -->
			<property name="hibernate.connection.provider_class" 
                      value="com.alibaba.druid.support.hibernate.DruidConnectionProvider" />
			<property name="url"
                      value="jdbc:mysql://192.168.1.200:3306/SSH_Data?useSSL=false&amp;allowPublicKeyRetrieval=true" />
			<property name="username" value="root" />
			<property name="password" value="123456" />
			<property name="driverClassName"
				value="com.mysql.cj.jdbc.Driver" />
			<property name="filters" value="stat" />
			<property name="maxActive" value="20" />
			<property name="initialSize" value="1" />
			<property name="maxWait" value="60000" />
			<property name="minIdle" value="1" />
			<property name="timeBetweenEvictionRunsMillis" value="60000" />
			<property name="minEvictableIdleTimeMillis" value="300000" />
			<property name="testWhileIdle" value="true" />
			<property name="testOnBorrow" value="false" />
			<property name="testOnReturn" value="false" />
			<property name="poolPreparedStatements" value="true" />
			<property name="maxOpenPreparedStatements" value="20" />
			<property name="asyncInit" value="true" />
        </properties>
    </persistence-unit>
</persistence>
```



## 5、JPA 核心API对象

JPA类层次结构的显示单元：

| 单元                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| Persistence          | 这个类包含静态方法来获取EntityManagerFactory实例             |
| EntityManagerFactory | 一个EntityManager的工厂类，创建并管理多个EntityManager实例   |
| EntityManager        | 一个接口，管理持久化操作的对象，工厂原理类似工厂的查询实例   |
| EntityTransaction    | 与EntityManager是一对一的关系，对于每一个EntityManager的操作由EntityTransaction类维护 |
| Entity               | 实体是持久性对象，是存储在数据库中的记录（实际上就是Java Bean） |
| Query                | 该接口由每个JPA供应商实现，能够获得符合标准的关系对象        |

JPA四个核心API对象：Persistence、EntityManagerFactory、EntityManager、EntityTransaction

Hibernate JPA 的操作步骤：

1. 加载配置文件创建实体管理器工厂
2. 根据实体管理器工厂，创建实体管理器
3. 创建事务对象，开启事务
4. 完成增删改查操作
5. 提交事务（回滚事务）
6. 释放资源

> 1、Persistence对象（创建实体管理器工厂）

```java
/**
 * Persisitence.createEntityMnagerFactory（持久化单元名称）
 * 静态方法（根据持久化单元名称创建实体管理器工厂）
 **/
// 1.加载配置文件创建工厂（实体管理器工厂）对象
EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
```

Persistence对象主要作用是用于获取EntityManagerFactory对象的 。通过调用该类的`createEntityManagerFactory()`静态方法，根据配置文件中持久化单元名称创建EntityManagerFactory。

> 2、EntityManagerFactory（EntityManagerFactory 接口主要用来创建 EntityManager 实例）

```java
/**
 * entityManagerFactory.createEntityManager()：获取EntityManager对象
 *     内部维护的很多的内容:
 *         1.内部维护了数据库信息
 *         2.维护了缓存信息
 *         3.维护了所有的实体管理器对象
 *         4.在创建EntityManagerFactory的过程中会根据配置创建数据库表
 *     EntityManagerFactory的创建过程比较浪费资源
 *         特点：线程安全的对象，多个线程访问同一个EntityManagerFactory不会有线程安全问题
 *     如何解决EntityManagerFactory的创建过程浪费资源（耗时）的问题？
 *         思路：创建一个公共的EntityManagerFactory的对象
 *         静态代码块的形式创建EntityManagerFactory
 **/
// 2.通过实体管理器工厂获取实体管理器
EntityManager em = factory.createEntityManager();
```

> 3、EntityManager：实体类管理器

```java
/**
 * EntityManager：实体类管理器
 *     获取事务对象: getTransaction()
 *     保存数据: presist()
 *     更新数据: merge()
 *     删除数据: remove()
 *     根据id查询: find()/getRefrence()
 *     清空缓存: clear()
 *     强制立即写入数据库: flush()
 *     重新加载缓存的对象: refresh()
 *     ......(还有挺多方法可以自己慢慢琢磨)
 **/
// 3.获取事务对象，然后可以开启事务、提交事务、回滚事务
EntityTransaction tx = em.getTransaction();
```

在 JPA 规范中, `EntityManager`是完成持久化操作的核心对象。实体类作为普通 java对象，只有在调用 `EntityManager`将其持久化后才会变成持久化对象。`EntityManager`对象在一组实体类与底层数据源之间进行 O/R 映射的管理。它可以用来管理和更新 Entity Bean, 根椐主键查找 `Entity Bean`, 还可以通过JPQL语句查询实体。我们可以通过调用`EntityManager`的方法完成获取事务，以及持久化数据库的操作

> 4、EntityTransaction：事务对象

```java
/**
 * EntityTransaction: 事务对象
 *     开启事务: begin()
 *     提交事务: commit()
 *     回滚事务: rollback()
 **/
// 开启事务
tx.begin();
```

在 JPA 规范中, `EntityTransaction`是完成事务操作的核心对象，对于`EntityTransaction`在我们的 java 代码中承接的功能比较简单



## 6、JPA 实现CRUD操作

### 1、保存操作：persist

```java
    /**
     * 运行之前，修改hibernate.hbm2ddl.auto=create
     * 保存操作
     */
    @Test
    public void testSave() {
        // 获取实体管理器工厂
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myJpa");
        // 获取实体管理器
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        // 获取事务
        EntityTransaction transaction = entityManager.getTransaction();
        // 开启事务
        transaction.begin();
        // 创建实体对象并保存
        Customer customer1 = new Customer();
        customer1.setName("张三");
        customer1.setAge(20);
        customer1.setSex(true);
        customer1.setPhone("13018882888");
        customer1.setAddress("北京");
        entityManager.persist(customer1);

        Customer customer2 = new Customer();
        customer2.setName("李四");
        customer2.setAge(18);
        customer2.setSex(false);
        customer2.setPhone("13533333555");
        customer2.setAddress("广州");
        entityManager.persist(customer2);

        Customer customer3 = new Customer();
        customer3.setName("王五");
        customer3.setAge(28);
        customer3.setSex(true);
        customer3.setPhone("13012345678");
        customer3.setAddress("深圳");
        entityManager.persist(customer3);
        // 提交事务
        transaction.commit();
        // 释放资源
        entityManager.close();
        entityManagerFactory.close();
    }
```

查看日志：

```verilog
Hibernate: 
    insert 
    into
        tb_customer
        (customer_address, customer_age, customer_name, customer_phone, customer_sex) 
    values
        (?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        tb_customer
        (customer_address, customer_age, customer_name, customer_phone, customer_sex) 
    values
        (?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        tb_customer
        (customer_address, customer_age, customer_name, customer_phone, customer_sex) 
    values
        (?, ?, ?, ?, ?)
```



### 2、查询操作：find/getReference

根据 ID 查询操作

1、`find` 方法查询（立即查询）

```java
    /**
     * 实际发送的SQL语句就是：select * from customer where id = 2
     * 运行之前，修改hibernate.hbm2ddl.auto=update
     * 立即查询操作
     */
    @Test
    public void testQuery1() {
        // 获取实体管理器工厂
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myJpa");
        // 获取实体管理器
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        // 获取事务
        EntityTransaction transaction = entityManager.getTransaction();
        // 开启事务
        transaction.begin();
        // 查询实体并输出
        Customer customer = entityManager.find(Customer.class, 2L);
        System.out.println(customer);
        // 提交事务
        transaction.commit();
        // 释放资源
        entityManager.close();
        entityManagerFactory.close();
    }
```

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_0_0_,
        customer0_.customer_address as customer2_0_0_,
        customer0_.customer_age as customer3_0_0_,
        customer0_.customer_name as customer4_0_0_,
        customer0_.customer_phone as customer5_0_0_,
        customer0_.customer_sex as customer6_0_0_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_id=?
Customer(Id=2, name=李四, age=18, sex=false, phone=13533333555, address=广州)
```



2、`getReference` 方法实现（懒加载查询）（推荐）

```java
    /**
     * 实际发送的SQL语句就是：select * from customer where id = 2
     * 运行之前，修改hibernate.hbm2ddl.auto=update
     * 延迟查询操作(查询结果对象的时候，才会发送查询的sql语句)
     */
    @Test
    public void testQuery2() {
        // 获取实体管理器工厂
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myJpa");
        // 获取实体管理器
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        // 获取事务
        EntityTransaction transaction = entityManager.getTransaction();
        // 开启事务
        transaction.begin();
        // 查询实体并输出
        Customer customer = entityManager.getReference(Customer.class, 2L);
        System.out.println(customer);
        // 提交事务
        transaction.commit();
        // 释放资源
        entityManager.close();
        entityManagerFactory.close();
    }
```

日志发现与 find 查询没有区别：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_0_0_,
        customer0_.customer_address as customer2_0_0_,
        customer0_.customer_age as customer3_0_0_,
        customer0_.customer_name as customer4_0_0_,
        customer0_.customer_phone as customer5_0_0_,
        customer0_.customer_sex as customer6_0_0_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_id=?
Customer(Id=2, name=李四, age=18, sex=false, phone=13533333555, address=广州)
```

可以注释打印对象在对比试试：`//System.out.println(customer)`;（可以发现，如果没有使用对象的话，是没有调用查询SQL语句的）

```verilog
INFO: HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
```



3、`find` 与`getReference` 查询对比：

- `find` 实现的查询时，是会在调用 find 方法时，立即发送SQL语句查询数据库的操作
- `getReference` 是一种延迟加载策略的操作，调用getReference方法不会立即发送sql语句查询数据库，当调用查询结果对象的时候，才会发送查询的sql语句（实际就是动态代理）



### 3、更新操作：merge

```java
    /**
     * 运行之前，修改hibernate.hbm2ddl.auto=update
     * 更新操作
     */
    @Test
    public void testUpdate() {
        // 获取实体管理器工厂
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myJpa");
        // 获取实体管理器
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        // 获取事务
        EntityTransaction transaction = entityManager.getTransaction();
        // 开启事务
        transaction.begin();
        // 查询实体并更新
        Customer customer = entityManager.find(Customer.class, 2L);
        customer.setAddress("上海");
        entityManager.merge(customer);
        // 提交事务
        transaction.commit();
        // 释放资源
        entityManager.close();
        entityManagerFactory.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_0_0_,
        customer0_.customer_address as customer2_0_0_,
        customer0_.customer_age as customer3_0_0_,
        customer0_.customer_name as customer4_0_0_,
        customer0_.customer_phone as customer5_0_0_,
        customer0_.customer_sex as customer6_0_0_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_id=?
Hibernate: 
    update
        tb_customer 
    set
        customer_address=?,
        customer_age=?,
        customer_name=?,
        customer_phone=?,
        customer_sex=? 
    where
        customer_id=?
```



### 4、删除操作：remove

```java
    /**
     * 运行之前，修改hibernate.hbm2ddl.auto=update
     * 删除操作
     */
    @Test
    public void testDelete() {
        // 获取实体管理器工厂
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myJpa");
        // 获取实体管理器
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        // 获取事务
        EntityTransaction transaction = entityManager.getTransaction();
        // 开启事务
        transaction.begin();
        // 查询实体并删除
        Customer customer = entityManager.find(Customer.class, 1L);
        entityManager.remove(customer);
        // 提交事务
        transaction.commit();
        // 释放资源
        entityManager.close();
        entityManagerFactory.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_0_0_,
        customer0_.customer_address as customer2_0_0_,
        customer0_.customer_age as customer3_0_0_,
        customer0_.customer_name as customer4_0_0_,
        customer0_.customer_phone as customer5_0_0_,
        customer0_.customer_sex as customer6_0_0_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_id=?
Hibernate: 
    delete 
    from
        tb_customer 
    where
        customer_id=?
```





## 7、JPA 连接工厂工具类

为什么要抽取 JpaUtil 工具类？

由于`EntityManagerFactory` 是一个线程安全的对象（即多个线程访问同一个`EntityManagerFactory` 对象不会有线程安全问题），并且`EntityManagerFactory` 的创建极其浪费资源，所以在使用JPA编程时，我们可以对`EntityManagerFactory` 的创建进行优化，只需要做到一个工程只存在一个`EntityManagerFactory` 即可。

解决思路是通过静态代码的形式创建 `EntityManagerFactory`

`JpaUtil.java` 工具类

```java
/**
 * 解决实体管理器工厂的浪费资源和耗时问题
 *      通过静态代码块的形式，当程序第一次访问此工具类时，创建一个公共的实体管理器工厂对象
 *
 * 第一次访问getEntityManager方法：经过静态代码块创建一个factory对象，再调用方法创建一个EntityManager对象
 * 第二次方法getEntityManager方法：直接通过一个已经创建好的factory对象，创建EntityManager对象
 */
public class JpaUtils {
    private static EntityManagerFactory factory;
    static  {
        //1.加载配置文件，创建entityManagerFactory
        factory = Persistence.createEntityManagerFactory("myJpa");
    }

    /**
     * 获取EntityManager对象
     */
    public static EntityManager getEntityManager() {
        return factory.createEntityManager();
    }
}
```

测试工具类

```java
public class JpaTest {
    @Test
    public void testSave() {
        // 1.通过工具类获取实体类管理器
        EntityManager em = JpaUtils.getEntityManager();
        // 2.获取事务对象
        EntityTransaction tx = em.getTransaction();
        // 开启事务
        tx.begin();
        //3.完成增删改查操作：保存一个客户到数据库中
        Customer customer = new Customer();
        customer.setName("Yolo");
        customer.setAddress("BeiJing");
        //保存操作
        em.persist(customer);
        //4.提交事务
        tx.commit();
        //5.释放资源
        em.close();
        //因为工厂是公共的，不能关闭，不然其他线程将无法获取
        //factory.close();
    }
}
```



# Hibernate JPA 复杂查询

## 1、JPQL 查询与更新

> JPQL全称：Java Persistence Query Language

在 `EJB2.0` 中引入的 `EJB` 查询语言 (`EJB QL`)，`Java` 持久化查询语言 (`JPQL`) 是一种可移植的查询语言，旨在以面向对象表达式语言的表达式，将 `SQL` 语法和简单查询语义绑定在一起,使用这种语言编写的查询是可移植的，可以被编译成所有主流数据库服务器上的 `SQL`。

在使用 `JPQL` 的时候整体的处理效果与`SQL`的语法形式是几乎相同的，而后在`JPA`执行的时候会将 `JPQL` 编译为传统的关系型数据库支持的`SQL`语句。其特征与原生 `SQL` 语句类似，并且完全面向对象，**通过类名和属性访问，而不是表名和表的属性**。

> 环境准备

1、在实体类上加上一些Lombok的注解，方便准备测试数据

```java
import lombok.*;
import lombok.experimental.Tolerate;
import javax.persistence.*;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tb_customer")
public class Customer {

    // 这个构造是用来给 投影查询 准备的
    @Tolerate
    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long Id; // 客户的主键

    @Column(name = "customer_name")
    private String name; // 客户名称

    @Column(name="customer_age")
    private int age; // 客户年龄

    @Column(name="customer_sex")
    private boolean sex; // 客户性别

    @Column(name="customer_phone")
    private String phone; // 客户的联系方式

    @Column(name="customer_address")
    private String address; // 客户地址
}
```

2、准备测试数据

```java
    @Before
    public void initData() {
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();
        Arrays.asList(
                Customer.builder().name("Sam").age(18).sex(true).phone("135000000001").address("广州").build(),
                Customer.builder().name("Mike").age(20).sex(true).phone("135000000002").address("广州").build(),
                Customer.builder().name("Nick").age(25).sex(true).phone("135000000003").address("深圳").build(),
                Customer.builder().name("Hom").age(30).sex(true).phone("135000000004").address("西安").build(),
                Customer.builder().name("Rachel").age(25).sex(false).phone("135000000005").address("北京").build(),
                Customer.builder().name("Kath").age(30).sex(true).phone("135000000006").address("武汉").build(),
                Customer.builder().name("Vivi").age(28).sex(false).phone("135000000007").address("南京").build(),
                Customer.builder().name("Oliver").age(30).sex(true).phone("135000000008").address("深圳").build(),
                Customer.builder().name("Angus").age(10).sex(false).phone("135000000009").address("广州").build(),
                Customer.builder().name("Wendy").age(15).sex(false).phone("135000000000").address("西安").build()
        ).forEach(customer -> entityManager.persist(customer));
        entityManager.getTransaction().commit();
        entityManager.close();
    }
```



### 1.1、简单查询

```java
   /**
     * 查询全部
     * jqpl：from Customer
     * sql：select * from c_customer
     */
    @Test
    public void testFindAll() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 2.查询全部：创建Query查询对象，query对象才是执行jqpl的对象
        Query query = entityManager.createQuery("from Customer");

        // 3.发送查询，并封装结果集
        List<Customer> list = query.getResultList();
        list.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
```



### 1.2、别名查询

```java
    /**
     * 别名查询
     *     jqpl：from Customer c
     *     jqpl: select c from Customer c
     *     sql：select * from c_customer c
     */
    @Test
    public void testFindAll2() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 查询所有客户,采用链式调用
        List<Customer> list1 = entityManager.createQuery("from Customer c").getResultList();
        list1.forEach(System.out::println);

        // 查询所有客户,采用链式调用
        List<Customer> list2 = entityManager.createQuery("select c from Customer c").getResultList();
        list2.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
```



### 1.3、排序查询

```java
    /**
     * 排序查询
     *     jqpl：from Customer order by Id desc
     *     sql：select * from c_customer order by Id desc
     */
    @Test
    public void testOder() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 采用链式调用，默认情况（升序）
        List<Customer> list1 = entityManager.createQuery("from Customer order by Id").getResultList();
        list1.forEach(System.out::println);

        // 采用链式调用，升序情况
        List<Customer> list2 = entityManager.createQuery("from Customer order by Id asc").getResultList();
        list2.forEach(System.out::println);

        // 采用链式调用，降序情况
        List<Customer> list3 = entityManager.createQuery("from Customer order by Id desc").getResultList();
        list3.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    order by
        customer0_.customer_id
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    order by
        customer0_.customer_id asc
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    order by
        customer0_.customer_id desc
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
```



### 1.4、条件查询

```java
    /**
     * 条件查询
     *     jqpl：from Customer where sex = ?1 and name like ?2
     *     sql：SELECT * FROM c_customer where c_sex = ?1 and c_name like ?2
     *     jqpl：from Customer where sex = :aaa and name like :bbb
     *     sql：SELECT * FROM c_customer where c_sex = :aaa and c_name like :bbb
     */
    @Test
    public void testWhere() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 条件查询：按参数位置绑定
        Query query1 = entityManager.createQuery("from Customer where sex = ?1 and name like ?2");
        query1.setParameter(1, true);
        query1.setParameter(2, "S%");
        List<Customer> list1 = query1.getResultList();
        list1.forEach(System.out::println);

        // 条件查询：按参数名称绑定
        Query query2 = entityManager.createQuery("from Customer where sex = :aaa and name like :bbb");
        query2.setParameter("aaa", true);
        query2.setParameter("bbb", "S%");
        List<Customer> list2 = query2.getResultList();
        list2.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_sex=? 
        and (
            customer0_.customer_name like ?
        )
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_sex=? 
        and (
            customer0_.customer_name like ?
        )
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
```



### 1.5、投影查询

```java
    /**
     * 投影查询
     * 1.单个字段查询
     * jqpl：select c.name from Customer c
     * sql：SELECT * FROM c_customer order by Id desc
     * 2.多个字段查询
     * jpql：select c.name,c.age from Customer c
     * jpql：select new Customer(c.name,c.age) from Customer c
     * sql：SELECT * FROM c_customer order by Id desc
     */
    @Test
    public void testFieldName() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 查询所有客户名称：单个字段查询
        Query query1 = entityManager.createQuery("select c.name from Customer c");
        List<Object> list1 = query1.getResultList();
        list1.forEach(System.out::println);

        // 查询所有客户名称、客户年龄：多个字段查询，封装到数组中
        Query query2 = entityManager.createQuery("select c.name,c.age from Customer c");
        List<Object[]> list2 = query2.getResultList();
        list2.forEach(x-> System.out.println(Arrays.toString(x)));

        // 查询所有客户名称、客户来源：多个字段查询，封装到对象中
        // 请在Customer.java添加以下两个构造方法，否则会执行失败
        // public Customer() 和 public Customer(String name, int age)
        Query query3 = entityManager.createQuery("select new Customer(c.name,c.age) from Customer c");
        List<Customer> list3 = query3.getResultList();
        list3.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_name as col_0_0_ 
    from
        tb_customer customer0_
Sam
Mike
Nick
Hom
Rachel
Kath
Vivi
Oliver
Angus
Wendy
Hibernate: 
    select
        customer0_.customer_name as col_0_0_,
        customer0_.customer_age as col_1_0_ 
    from
        tb_customer customer0_
[Sam, 18]
[Mike, 20]
[Nick, 25]
[Hom, 30]
[Rachel, 25]
[Kath, 30]
[Vivi, 28]
[Oliver, 30]
[Angus, 10]
[Wendy, 15]
Hibernate: 
    select
        customer0_.customer_name as col_0_0_,
        customer0_.customer_age as col_1_0_ 
    from
        tb_customer customer0_
Customer(Id=null, name=Sam, age=18, sex=false, phone=null, address=null)
Customer(Id=null, name=Mike, age=20, sex=false, phone=null, address=null)
Customer(Id=null, name=Nick, age=25, sex=false, phone=null, address=null)
Customer(Id=null, name=Hom, age=30, sex=false, phone=null, address=null)
Customer(Id=null, name=Rachel, age=25, sex=false, phone=null, address=null)
Customer(Id=null, name=Kath, age=30, sex=false, phone=null, address=null)
Customer(Id=null, name=Vivi, age=28, sex=false, phone=null, address=null)
Customer(Id=null, name=Oliver, age=30, sex=false, phone=null, address=null)
Customer(Id=null, name=Angus, age=10, sex=false, phone=null, address=null)
Customer(Id=null, name=Wendy, age=15, sex=false, phone=null, address=null)
```



### 1.6、分页查询

```java
    /**
     * 分页查询
     *      jqpl : from Customer
     *      sql：select * from c_customer limit 2,5
     */
    @Test
    public void testLimit() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 分页查询
        Query query = entityManager.createQuery("from Customer");
        // 起始索引、每页查询的条数
        query.setFirstResult(2).setMaxResults(5);
        List<Customer> list = query.getResultList();
        list.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ limit ?,
        ?
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
```



### 1.7、分组查询

```java
    /**
     * 分组查询
     *      jqpl : select count(*) from Customer
     *      sql：select count(*) from c_customer
     *
     *      jqpl : select sex,count(*) from Customer group by sex
     *      sql：select c_sex,count(*) from c_customer group by c_sex
     */
    @Test
    public void testGroupBy() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 聚合函数：count(),max(),min(),avg(),sum()
        Object object2 = entityManager.createQuery("select count(*) from Customer").getSingleResult();
        System.out.println(object2);

        // 分组统计：
        List<Object[]> list = entityManager.createQuery("select sex,count(*) from Customer group by sex").getResultList();
        list.forEach(x->System.out.println(Arrays.toString(x)));

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        count(*) as col_0_0_ 
    from
        tb_customer customer0_
10
Hibernate: 
    select
        customer0_.customer_sex as col_0_0_,
        count(*) as col_1_0_ 
    from
        tb_customer customer0_ 
    group by
        customer0_.customer_sex
[false, 4]
[true, 6]
```



### 1.8、多表查询

```java
    @Test
    public void testXXJoin() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 内连接
        List<Object[]> list1 = entityManager.createQuery("from Customer c inner join c.linkMans").getResultList();
        list1.forEach(x->System.out.println(Arrays.toString(x)));

        // 迫切内连接（hibernate独有，将另一个对象的数据封装到该对象中）
        List<Customer> list2 = entityManager.createQuery("select distinct c from Customer c inner join fetch c.linkMans").getResultList();
        list2.forEach(System.out::println);

        // 左外连接
        List<Object[]> list3 = entityManager.createQuery("from Customer c left outer join c.linkMans").getResultList();
        list3.forEach(x->System.out.println(Arrays.toString(x)));

        // 迫切左外连接（hibernate独有，将另一个对象的数据封装到该对象中）
        List<Customer> list4 = entityManager.createQuery("select distinct c from Customer c left outer join fetch c.linkMans").getResultList();
        list4.forEach(System.out::println);

        // 右外连接
        List<Object[]> list5 = entityManager.createQuery("from Customer c right outer join c.linkMans").getResultList();
        list5.forEach(x->System.out.println(Arrays.toString(x)));

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```



### 1.9、更新删除

```java
    /**
     * 数据更新
     *     jpql：update Customer c set c.age = :age where c.name = :name
     *     sql：sql：update tb_customer set customer_age = 20 where customer_name = 'Sam'
     */
    @Test
    public void testUpdate() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 数据更新
        String jpql = "update Customer c set c.age = :age where c.name = :name";
        int updateSize = entityManager.createQuery(jpql)
                .setParameter("age", 20)
                .setParameter("name", "Sam")
                .executeUpdate();
        System.out.println("更新数据量：" + updateSize);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }

    /**
     * 数据删除
     *     jpql：delete Customer c where c.name = :name
     *     sql：delete tb_customer where customer_name = 'Sam'
     */
    @Test
    public void testDelete() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 数据删除
        String sql = "delete Customer c where c.name = :name";
        int updateSize = entityManager.createQuery(sql)
                .setParameter("name", "Sam")
                .executeUpdate();
        System.out.println("删除数据量：" + updateSize);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
# 更新日志
Hibernate: 
    update
        tb_customer 
    set
        customer_age=? 
    where
        customer_name=?
更新数据量：1
# 删除日志
Hibernate: 
    delete 
    from
        tb_customer 
    where
        customer_name=?
删除数据量：1
```

> 如果报错：java.lang.IllegalArgumentException: Update/delete queries cannot be typed
>
> 问题原因（错误写法）：Query query = em.createQuery(jpql, Entity.class);
>
> 解决办法（正确写法）：Query query = em.createQuery(jpql);



## 2、SQL 查询与更新

有些时候，JPQL使用不当会导致转化成的SQL并不如理想或者特定场合需要SQL优化，还是得用到原生SQL查询的。`EntityManager`对象的`createNativeQuery`方法，可以实现十分复杂的查询，但是需要对查询结果进行实体映射处理，并且不能跨数据库。

- 获得原生SQL查询对象：`Query createNativeQuery(String var1, Class var2);`

- SQL对比JPQL只需要把查询对象方法由 `createQuery()`替换成`createNativeQuery()` 即可（传入值变成原生SQL和**类对象**）

```java
package query;

import entity.Customer;
import org.junit.Before;
import org.junit.Test;
import util.JpaUtils;
import javax.persistence.*;
import java.util.Arrays;
import java.util.List;

public class JpaSqlTest {

    @Before
    public void initData() {
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();
        Arrays.asList(
                Customer.builder().name("Sam").age(18).sex(true).phone("135000000001").address("广州").build(),
                Customer.builder().name("Mike").age(20).sex(true).phone("135000000002").address("广州").build(),
                Customer.builder().name("Nick").age(25).sex(true).phone("135000000003").address("深圳").build(),
                Customer.builder().name("Hom").age(30).sex(true).phone("135000000004").address("西安").build(),
                Customer.builder().name("Rachel").age(25).sex(false).phone("135000000005").address("北京").build(),
                Customer.builder().name("Kath").age(30).sex(true).phone("135000000006").address("武汉").build(),
                Customer.builder().name("Vivi").age(28).sex(false).phone("135000000007").address("南京").build(),
                Customer.builder().name("Oliver").age(30).sex(true).phone("135000000008").address("深圳").build(),
                Customer.builder().name("Angus").age(10).sex(false).phone("135000000009").address("广州").build(),
                Customer.builder().name("Wendy").age(15).sex(false).phone("135000000000").address("西安").build()
        ).forEach(customer -> entityManager.persist(customer));
        entityManager.getTransaction().commit();
        entityManager.close();
    }

    /**
     * 查询全部 sql：SELECT * FROM tb_customer
     */
    @Test
    public void testFindAll() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 2.查询全部：创建Query查询对象，query对象才是执行jqpl的对象
        String sql = "select * from tb_customer";
        Query query = entityManager.createNativeQuery(sql, Customer.class);

        // 3.发送查询，并封装结果集
        List<Customer> list = query.getResultList();
        list.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }

    /**
     * 排序查询 sql：select * from tb_customer order by customer_id desc
     */
    @Test
    public void testOder() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 采用链式调用，默认情况（升序）
        String sql1 = "select * from tb_customer order by customer_id";
        List<Customer> list1 = entityManager.createNativeQuery(sql1, Customer.class).getResultList();
        list1.forEach(System.out::println);

        // 采用链式调用，升序情况
        String sql2 = "select * from tb_customer order by customer_id asc ";
        List<Customer> list2 = entityManager.createNativeQuery(sql2, Customer.class).getResultList();
        list2.forEach(System.out::println);

        // 采用链式调用，降序情况
        String sql3 = "select * from tb_customer order by customer_id desc";
        List<Customer> list3 = entityManager.createNativeQuery(sql3, Customer.class).getResultList();
        list3.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }

    /**
     * 条件查询
     *     sql：SELECT * FROM tb_customer where customer_sex = ?1 and customer_name like ?2
     *     sql：SELECT * FROM tb_customer where customer_sex = :aaa and customer_name like :bbb
     */
    @Test
    public void testWhere() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 条件查询：按参数位置绑定
        String sql1 = "SELECT * FROM tb_customer where customer_sex = ?1 and customer_name like ?2";
        Query query1 = entityManager.createNativeQuery(sql1, Customer.class);
        query1.setParameter(1, true);
        query1.setParameter(2, "S%");
        List<Customer> list1 = query1.getResultList();
        list1.forEach(System.out::println);

        // 条件查询：按参数名称绑定
        String sql2 = "SELECT * FROM tb_customer where customer_sex = :aaa and customer_name like :bbb";
        Query query2 = entityManager.createNativeQuery(sql2, Customer.class);
        query2.setParameter("aaa", true);
        query2.setParameter("bbb", "S%");
        List<Customer> list2 = query2.getResultList();
        list2.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }

    /**
     * 分页查询 sql：select * from tb_customer limit 2,5
     */
    @Test
    public void testLimit() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 分页查询
        String sql = "select * from tb_customer limit 2,5";
        Query query = entityManager.createNativeQuery(sql, Customer.class);
        // 起始索引、每页查询的条数
        //query.setFirstResult(2).setMaxResults(5);
        List<Customer> list = query.getResultList();
        list.forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }

    /**
     * 分组查询
     *      sql：select count(*) from tb_customer
     *      sql：select c_sex,count(*) from tb_customer group by c_sex
     */
    @Test
    public void testGroupBy() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 聚合函数：count(),max(),min(),avg(),sum()
        String sql1 = "select count(*) from tb_customer";
        Object object2 = entityManager.createNativeQuery(sql1).getSingleResult();
        System.out.println(object2);

        // 分组统计：
        String sql2 = "select customer_sex,count(*) from tb_customer group by customer_sex";
        List<Object[]> list = entityManager.createNativeQuery(sql2).getResultList();
        list.forEach(x->System.out.println(Arrays.toString(x)));

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
    
    /**
     * 数据更新 sql：update tb_customer set customer_age = 20 where customer_name = 'Sam'
     */
    @Test
    public void testUpdate() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 2.数据更新
        String sql = "update tb_customer set customer_age = :customer_age where customer_name = :customer_name";
        int updateSize = entityManager.createNativeQuery(sql, Customer.class)
                .setParameter("customer_age", 20)
                .setParameter("customer_name", "Sam")
                .executeUpdate();
        System.out.println("更新数据量：" + updateSize);

        // 3.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }

    /**
     * 数据删除 sql：delete tb_customer where customer_name = 'Sam'
     */
    @Test
    public void testDelete() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 2.数据删除
        String sql = "delete tb_customer where customer_name = :customer_name";
        int updateSize = entityManager.createNativeQuery(sql, Customer.class)
                .setParameter("customer_name", "Sam")
                .executeUpdate();
        System.out.println("删除数据量：" + updateSize);

        // 3.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
}
```

查看日志（随意抽取一条）：可以发现SQL语句与JPQL生成的还是有些不一样

```verilog
Hibernate: 
    select
        * 
    from
        tb_customer
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
```

查看数据更新与删除日志：

```verilog
# 数据更新
Hibernate: 
    update
        tb_customer 
    set
        customer_age = ? 
    where
        customer_name = ?
更新数据量：1
# 数据删除
Hibernate: 
    delete tb_customer 
    where
        customer_name = ?
删除数据量：1
```





## 3、Criteria 复杂查询

> JPA Criteria API：https://www.cnblogs.com/xingqi/p/3929386.html

1、通过JPA的 Criteria API 实现：

1. EntityManager 获取 CriteriaBuilder
2. CriteriaBuilder 创建 CriteriaQuery
3. CriteriaQuery 指定要查询的表，得到 Root（Root 代表要查询的表）
4. CriteriaBuilder 创建条件 Predicate，Predicate 相当于SQL的 where 条件，多个 Predicate 可以进行与、或操作
5. 通过 EntityManager 创建 TypedQuery
6. TypedQuery 执行查询，返回结果



2、基本对象的构建详细步骤：

1. 通过 EntityManager.getCriteriaBuilder() 或 EntityManagerFactory.getCriteriaBuilder() 方法可以得到 CriteriaBuilder 对象
2. 通过 CriteriaBuilder.createQuery() 或 CriteriaBuilder.createTupleQuery() 方法可以获得 CriteriaQuery 实例
3. 通过 CriteriaQuery.from() 方法可以获得 Root 实例
4. 子句总结：
   1. SELECT：CriteriaQuery select()
   2. FROM：AbstractQuery from()
   3. WHERE：AbstractQuery where()
   4. ORDER BY：CriteriaQuery orderBy()
   5. GROUP BY：AbstractQuery groupBy()
   6. HAVING：AbstractQuery having()



3、过滤条件：

1. 过滤条件会被应用到SQL语句的FROM子句中。在Criteria查询中，查询条件通过Predicate或Expression实例应用到CriteriaQuery对象上
2. 这些条件使用 CriteriaQuery.where()方 法应用到CriteriaQuery对象上
3. CriteriaBuilder也作为Predicate实例的工厂，通过调用CriteriaBuilder的条件方法（equal，notEqual， gt， ge，lt， le，between，like等）创建Predicate对象
4. 复合的 Predicate 语句可以使用CriteriaBuilder的 and, or, andnot 方法构建



> 准备测试数据

```java
import lombok.*;
import lombok.experimental.Tolerate;
import javax.persistence.*;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tb_customer")
public class Customer {

    // 这个构造是用来给 投影查询 准备的
    @Tolerate
    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long Id; // 客户的主键

    @Column(name = "customer_name")
    private String name; // 客户名称

    @Column(name="customer_age")
    private int age; // 客户年龄

    @Column(name="customer_sex")
    private boolean sex; // 客户性别

    @Column(name="customer_phone")
    private String phone; // 客户的联系方式

    @Column(name="customer_address")
    private String address; // 客户地址
}
```

```java
    @Before
    public void initData() {
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();
        Arrays.asList(
                Customer.builder().name("Sam").age(18).sex(true).phone("135000000001").address("广州").build(),
                Customer.builder().name("Mike").age(20).sex(true).phone("135000000002").address("广州").build(),
                Customer.builder().name("Nick").age(25).sex(true).phone("135000000003").address("深圳").build(),
                Customer.builder().name("Hom").age(30).sex(true).phone("135000000004").address("西安").build(),
                Customer.builder().name("Rachel").age(25).sex(false).phone("135000000005").address("北京").build(),
                Customer.builder().name("Kath").age(30).sex(true).phone("135000000006").address("武汉").build(),
                Customer.builder().name("Vivi").age(28).sex(false).phone("135000000007").address("南京").build(),
                Customer.builder().name("Oliver").age(30).sex(true).phone("135000000008").address("深圳").build(),
                Customer.builder().name("Angus").age(10).sex(false).phone("135000000009").address("广州").build(),
                Customer.builder().name("Wendy").age(15).sex(false).phone("135000000000").address("西安").build()
        ).forEach(customer -> entityManager.persist(customer));
        entityManager.getTransaction().commit();
        entityManager.close();
    }
```

### 3.1、查询全部

```java
    /**
     * 查询全部（简单写法）
     * sql：select * from tb_customer
     */
    @Test
    public void testFindAll() {
        // 获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        // 1.EntityManager获取CriteriaBuilder
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        // 2.CriteriaBuilder创建CriteriaQuery
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        // 3.CriteriaQuery指定要查询的表，得到Root，Root代表要查询的表
        criteriaQuery.from(Customer.class);
        TypedQuery<Customer> query = entityManager.createQuery(criteriaQuery);
        List<Customer> list = query.getResultList();
        list.forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }

    /**
     * 查询全部（完整写法）
     * sql：select * from tb_customer
     */
    @Test
    public void testFindAll2() {
        // 获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        // 1.EntityManager获取CriteriaBuilder
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        // 2.CriteriaBuilder创建CriteriaQuery
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        // 3.CriteriaQuery指定要查询的表，得到Root，Root代表要查询的表
        Root<Customer> from = criteriaQuery.from(Customer.class);
        // 这一步骤可以省略,返回的依旧是CriteriaQuery对象
        CriteriaQuery<Customer> select = criteriaQuery.select(from);
        // 如果上一步省略就传入criteriaQuery
        TypedQuery<Customer> query = entityManager.createQuery(select);
        List<Customer> list = query.getResultList();
        list.forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
```



### 3.2、投影查询

获取 `CriteriaQuery` 实例有两种方式：

1. `CriteriaBuilder.createQuery()`：返回 `CriteriaQuery<T>`
2. `CriteriaBuilder.createTupleQuery()`：返回`CriteriaQuery<Tuple>`

两种方式的区别：

1. `createQuery()`： 主要用来查询与实体类字段对应的SQL
2. `createTupleQuery()`：可以查询实体类字段以外的字段数据（count、sum、max、min、avg 等）group by用的较多，一般配合`criteriaQuery.multiselect()` 方法较多

---

方式一：CriteriaBuilder.createQuery()

```java
    /**
     * 投影查询 — criteriaBuilder.createQuery(Customer.class)
     * sql：select customer_name, customer_age from tb_customer
     * 注意：
     *   实体类中必须要有该构造方法：public Customer(String name, int age) 不然会报错
     * 也可以用另一个方式：
     *   criteriaBuilder.createTupleQuery() // 创建查询，返回元组类型，Tuple包含多个TupleElements
     */
    @Test
    public void testFindAllMultiSelect() {
        // 获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        // 1.CriteriaBuilder 安全查询创建工厂
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        // 2.CriteriaQuery 安全查询主语句
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        // 3.Root 定义查询的From子句中能出现的类型
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // 必须要有该构造方法才行：public Customer(String name, int age)，不然会报错
        criteriaQuery.multiselect(from.get("name"), from.get("age"));
        TypedQuery<Customer> query = entityManager.createQuery(criteriaQuery);
        List<Customer> list = query.getResultList();
        list.forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_name as col_0_0_,
        customer0_.customer_age as col_1_0_ 
    from
        tb_customer customer0_
Customer(Id=null, name=Sam, age=18, sex=false, phone=null, address=null)
Customer(Id=null, name=Mike, age=20, sex=false, phone=null, address=null)
Customer(Id=null, name=Nick, age=25, sex=false, phone=null, address=null)
Customer(Id=null, name=Hom, age=30, sex=false, phone=null, address=null)
Customer(Id=null, name=Rachel, age=25, sex=false, phone=null, address=null)
Customer(Id=null, name=Kath, age=30, sex=false, phone=null, address=null)
Customer(Id=null, name=Vivi, age=28, sex=false, phone=null, address=null)
Customer(Id=null, name=Oliver, age=30, sex=false, phone=null, address=null)
Customer(Id=null, name=Angus, age=10, sex=false, phone=null, address=null)
Customer(Id=null, name=Wendy, age=15, sex=false, phone=null, address=null)
```

方式二：CriteriaBuilder.createTupleQuery()

```java
    /**
     * 投影查询 — criteriaBuilder.createTupleQuery()
     * sql：select customer_name, customer_age from tb_customer
     * 注意：
     *   使用：返回元组(Tuple)的查询，不会报错
     */
    @Test
    public void testFindAllMultiSelect2() {
        // 获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        // 1.CriteriaBuilder 安全查询创建工厂
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        // 2.CriteriaQuery 安全查询主语句
        CriteriaQuery<Tuple> criteriaQuery = criteriaBuilder.createTupleQuery();
        // 3.Root 定义查询的From子句中能出现的类型
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // 需要给字段取别名,否则无法通过tuple.get(field)获取数据
        criteriaQuery.multiselect(from.get("name").alias("name"), from.get("age").alias("age"));
        TypedQuery<Tuple> query = entityManager.createQuery(criteriaQuery);
        List<Tuple> list = query.getResultList();
        // name = list.get(0).get(0); age = list.get(0).get(1)
        list.forEach(x-> System.out.println(x.get("name")+"、"+x.get("age")));

        // 释放资源
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_name as col_0_0_,
        customer0_.customer_age as col_1_0_ 
    from
        tb_customer customer0_
Sam、18
Mike、20
Nick、25
Hom、30
Rachel、25
Kath、30
Vivi、28
Oliver、30
Angus、10
Wendy、15
```





### 3.3、条件查询

1、单条件查询

```java
    /**
     * 条件查询——单条件
     * Predicate 过滤条件
     * sql：select * from tb_customer where customer_name = 'Sam'
     */
    @Test
    public void testFindWhereEQ() {
        // 获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        Root<Customer> from = criteriaQuery.from(Customer.class);
        criteriaQuery.select(from);// 定义查询，该行可以省略

        // 4.Predicate 或 Predicate[] 过滤条件
        Predicate predicate = criteriaBuilder.equal(from.get("name"), "Sam");
        criteriaQuery.where(predicate);
        // 采用链式调用
        entityManager.createQuery(criteriaQuery).getResultList().forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_name=?
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
```



2、多条件查询

Predicate[]、Conjunction和 Disjunction 对比：

1. Predicate[]：逻辑与（and）
2. criteriaBuilder.conjunction()：逻辑与（and）
3. criteriaBuilder.disjunction()：逻辑或（or）

Predicate[] 与 conjunction 区别：Predicate[] 正常把数组内所有条件用 and 拼接，而 conjunction 会在 where 后面自动加上 1=1 然后再拼接条件

操作示例：

```java
    /**
     * 条件查询——多条件（and，equals，lt）
     * Predicate[] 多个过滤条件（个人推荐）
     * sql：select * from tb_customer where customer_name = 'Sam' and c_age < 20
     */
    @Test
    public void testFindWhereEqGt() {
        // 1.获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // 4.Predicate[] 过滤条件，设置一个查询条件集合
        List<Predicate> predicates = new ArrayList<>();
        predicates.add(criteriaBuilder.equal(from.get("name"), "Sam"));
        predicates.add(criteriaBuilder.lt(from.get("age"), 20));
        criteriaQuery.where(predicates.toArray(new Predicate[]{}));// new Predicate[]{}=》new Predicate[predicates.size()]
        // 采用链式调用
        entityManager.createQuery(criteriaQuery).getResultList().forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }

    /**
     * 条件查询——多条件（like，and，equal，lt）
     * Predicate 多个过滤条件 —— criteriaBuilder.conjunction() // 逻辑与
     *                       —— criteriaBuilder.disjunction() // 逻辑或
     * sql：select * from tb_customer where 1=1 and customer_name like 'S%' and customer_name = 'Sam' and customer_age < 20
     */
    @Test
    public void testFindWhereEqGt2() {
        // 1.获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // 4.Predicate[] 过滤条件，设置一个查询条件集合
        Predicate predicate = criteriaBuilder.conjunction();
        // 过滤条件— like：and customer_name like S%
        predicate= criteriaBuilder.and(predicate, criteriaBuilder.like(from.get("name"), "S%"));
        // 过滤条件— equal：and customer_name like S% and customer_name = 'Sam'
        predicate= criteriaBuilder.and(predicate, criteriaBuilder.equal(from.get("name"), "Sam"));
        // 过滤条件— lt：and customer_name like S% and customer_name = 'Sam' and customer_age < 20
        predicate= criteriaBuilder.and(predicate, criteriaBuilder.lt(from.get("age"), 20));
        criteriaQuery.where(predicate);
        // 采用链式调用
        entityManager.createQuery(criteriaQuery).getResultList().forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }

    /**
     * 条件查询——多条件（not in，between）
     * Predicate 多个过滤条件 —— criteriaBuilder.conjunction() // 逻辑与
     *                        —— criteriaBuilder.disjunction() // 逻辑或
     * sql：select * from tb_customer where 1=1 and customer_age not in (18 , 20) and customer_id between 5 and 9
     */
    @Test
    public void testFindWhereNotInBetween() {
        // 1.获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // 4.Predicate 过滤多条件
        Predicate predicate = criteriaBuilder.conjunction();
        // 过滤条件— in：customer_age not in (18 , 20)
        predicate= criteriaBuilder.and(predicate, from.get("age").in(Arrays.asList(18, 20)).not());
        // 过滤条件— in：customer_age not in (18 , 20) and customer_id between 5 and 9
        predicate= criteriaBuilder.and(predicate, criteriaBuilder.between(from.get("Id"), 5,9));
        criteriaQuery.where(predicate);
        // 采用链式调用
        entityManager.createQuery(criteriaQuery).getResultList().forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }
```

查看日志：（条件查询——多条件（and，equals，lt））

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_name=? 
        and customer0_.customer_age<20
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
```

查看日志：（条件查询——多条件（like，and，equal，lt））

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        1=1 
        and (
            customer0_.customer_name like ?
        ) 
        and customer0_.customer_name=? 
        and customer0_.customer_age<20
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
```

查看日志：（条件查询——多条件（not in，between））

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        1=1 
        and (
            customer0_.customer_age not in  (
                18 , 20
            )
        ) 
        and (
            customer0_.customer_id between 5 and 9
        )
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
```



### 3.4、排序查询

可以设置单个order和多个order

```java
    /**
     * 排序查询
     * sql：select * from tb_customer where 1=1 and customer_age in (18 , 28) and customer_id between 2 and 4 order by customer_id desc
     */
    @Test
    public void testFindWhereInBetweenOderBy() {
        // 1.获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // 4.Predicate 或 Predicate[] 过滤条件
        Predicate age = from.get("age").in(18, 28);
        Predicate id = criteriaBuilder.between(from.get("Id"), 2, 10);
        criteriaQuery.where(Arrays.asList(age, id).toArray(new Predicate[]{}));
        // 排序（降序）
        Order idOrder = criteriaBuilder.desc(from.get("Id"));
        criteriaQuery.orderBy(idOrder);
        // 可以设置多个 order。先按Id降序，然后按age升序排序
        // criteriaQuery.orderBy(criteriaBuilder.desc(from.get("Id")),criteriaBuilder.asc(from.get("age")));
        // 采用链式调用
        entityManager.createQuery(criteriaQuery).getResultList().forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        (
            customer0_.customer_age in (
                18 , 28
            )
        ) 
        and (
            customer0_.customer_id between 2 and 10
        ) 
    order by
        customer0_.customer_id desc
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
```



### 3.5、分页查询

```java
    /**
     * 分页查询
     * sql：select * from tb_customer where 1=1 and customer_id between 2 and 10 limit 0,5
     */
    @Test
    public void testFindWhereBetween() {
        // 1.获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Customer> criteriaQuery = criteriaBuilder.createQuery(Customer.class);
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // 4.Predicate 或 Predicate[] 过滤条件
        Predicate id = criteriaBuilder.between(from.get("Id"), 2, 10);
        criteriaQuery.where(Arrays.asList(id).toArray(new Predicate[]{}));

        // 采用链式调用,分页查询
        entityManager.createQuery(criteriaQuery)
                .setFirstResult(0) // 起始索引
                .setMaxResults(5)  // 每页查询的条数
                .getResultList()
                .forEach(System.out::println);

        // 释放资源
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_2_,
        customer0_.customer_address as customer2_2_,
        customer0_.customer_age as customer3_2_,
        customer0_.customer_name as customer4_2_,
        customer0_.customer_phone as customer5_2_,
        customer0_.customer_sex as customer6_2_ 
    from
        tb_customer customer0_ 
    where
        customer0_.customer_id between 2 and 10 limit ?
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
```



### 3.6、分组查询

分组查询需要注意事项（`group by` ）：

1. 创建查询用元组类型：`criteriaBuilder.createTupleQuery()`
2. 设置多个选择结果：`criteriaQuery.multiselect()`
3. 给字段设置别名（这不是必须的，为了获取方便可以设置）

```java
    /**
     * 分组查询
     * sql：select c_name,count(c_name),max(c_age),min(c_age),sum(c_age),avg(c_age) from c_customer group by c_name
     */
    @Test
    public void testFindGroupBy() {
        // 1.获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Tuple> criteriaQuery = criteriaBuilder.createTupleQuery();
        Root<Customer> from = criteriaQuery.from(Customer.class);

        // .alias("name") 取别名
        criteriaQuery.multiselect(
                from.get("name").alias("name"),
                criteriaBuilder.count(from.get("name")).alias("count"),
                criteriaBuilder.max(from.get("age")).alias("max"),
                criteriaBuilder.min(from.get("age")).alias("min"),
                criteriaBuilder.sum(from.get("age")).alias("sum"),
                criteriaBuilder.avg(from.get("age")).alias("avg"));
        criteriaQuery.groupBy(from.get("name"));
        // criteriaQuery.having(criteriaBuilder.disjunction());

        // 采用链式调用
        entityManager.createQuery(criteriaQuery).getResultList()
                .forEach(x-> System.out.println(
                        x.get("name") + "、" +
                        x.get("count") + "、" +
                        x.get("max") + "、" +
                        x.get("min") + "、" +
                        x.get("sum") + "、" +
                        x.get("avg"))
                );

        // 释放资源
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_name as col_0_0_,
        count(customer0_.customer_name) as col_1_0_,
        max(customer0_.customer_age) as col_2_0_,
        min(customer0_.customer_age) as col_3_0_,
        sum(customer0_.customer_age) as col_4_0_,
        avg(customer0_.customer_age) as col_5_0_ 
    from
        tb_customer customer0_ 
    group by
        customer0_.customer_name
Angus、1、10、10、10、10.0
Hom、1、30、30、30、30.0
Kath、1、30、30、30、30.0
Mike、1、20、20、20、20.0
Nick、1、25、25、25、25.0
Oliver、1、30、30、30、30.0
Rachel、1、25、25、25、25.0
Sam、1、18、18、18、18.0
Vivi、1、28、28、28、28.0
Wendy、1、15、15、15、15.0
```



### 3.7、多表查询

建议先参考多对一教程的实体类测试，或者也可以直接使用下方给定的实体类

```java
/**
 * 多方/外键表
 * 本实例采用的是：单向多对一。所以只需要配置多方即可
 */
@Data
@Entity
@Table(name = "t_many")
public class Many {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "many_id")
    private Long manyId;
    @Column(name = "many_name")
    private String manyName;

    @ManyToOne
    @JoinColumn(name="one_id")
    private One one;
}


/**
 * 一方/主键表
 * 本实例采用的是：单向多对一。所以只需要配置多方即可
 */
@Data
@Entity
@Table(name = "t_one")
public class One {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "one_id")
    private Long oneId;
    @Column(name = "one_type")
    private String oneType;
}
```

测试代码：

```java
    /**
     * 多表关联：单向多对一
     * 数据插入
     */
    @Test
    public void testSaveJoinTable() {
        // 1.获取entityManager对象
        EntityManager entityManager = JpaUtils.getEntityManager();
        // 开启事务
        entityManager.getTransaction().begin();

        // 给One插入一条数据（主键表）
        One one = new One();
        one.setOneType("初中");
        entityManager.persist(one);

        // 给Many插入三条数据（外键表）
        Many manyA = new Many();
        manyA.setManyName("初一");
        manyA.setOne(one);
        entityManager.persist(manyA);
        Many manyB = new Many();
        manyB.setManyName("初二");
        manyB.setOne(one);
        entityManager.persist(manyB);
        Many manyC = new Many();
        manyC.setManyName("初三");
        manyC.setOne(one);
        entityManager.persist(manyC);

        // 提交事务
        entityManager.getTransaction().commit();
    }

    /**
     * Root代表需要查询的表（这里表示的是Many表）
     * Join代表连接查询(表)，通过root对象获取（Join代表的是关联表One表）
     * 创建的过程中，第一个参数为关联对象的属性名称，第二个参数为连接查询的方式（left，inner，right）
     * JoinType.INNER：内连接, JoinType.LEFT: 左外连接, JoinType.RIGHT：右外连接
     * 使用多表关联后，Join 就相当有了 root 的功能，可以join.get("对象属性名称")
     * 注意！注意！注意：
     *     Root：代表的是Many表，所以通过root.get()只能获取Many表的属性或字段.
     *     join：代表的是One表，虽然是关联查询，但它只代表关联的One表，只能获取One的属性
     *     如果root/join获取自己表以外的属性或字段会报如下错：
     *     Unable to locate Attribute  with the the given name [categoryType] on this ManagedType[XXX]
     *     如果有第三张表关联使用join.join()往下传递即可。
     */
    @Test
    public void testFindJoinTable() {
        // 1.获取entityManager对象 && 这里就省略事务了
        EntityManager entityManager = JpaUtils.getEntityManager();

        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Many> criteriaQuery = criteriaBuilder.createQuery(Many.class);

        // 重点：root只能代表Many表，join只能代表One表。
        Root<Many> root = criteriaQuery.from(Many.class);
        Join<Many, One> join = root.join("one", JoinType.LEFT);

        Path<Object> type = join.get("oneType");
        Path<Object> manyName = root.get("manyName");
        Predicate p1 = criteriaBuilder.equal(type, "初中");
        Predicate p2 = criteriaBuilder.equal(manyName, "初一");
        Predicate predicate = criteriaBuilder.and(p1, p2);
        criteriaQuery.where(predicate);
        TypedQuery<Many> query = entityManager.createQuery(criteriaQuery);
        List<Many> resultList = query.getResultList();
        resultList.forEach(System.out::println);
    }
```

查看日志：

```sql
Hibernate: 
    select
        many0_.many_id as many_id1_3_,
        many0_.many_name as many_nam2_3_,
        many0_.one_id as one_id3_3_ 
    from
        t_many many0_ 
    left outer join
        t_one one1_ 
            on many0_.one_id=one1_.one_id 
    where
        one1_.one_type=? 
        and many0_.many_name=?
Hibernate: 
    select
        one0_.one_id as one_id1_4_0_,
        one0_.one_type as one_type2_4_0_ 
    from
        t_one one0_ 
    where
        one0_.one_id=?
Many(manyId=1, manyName=初一, one=One(oneId=1, oneType=初中))
```



## 4、NamedQuery 别名查询

又叫别名查询。主要使用：@NamedQuery 或者 @NamedNativeQuery+  entityManager.createNamedQuery() 来查询

实体类：

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tb_customer")
// 主要加上该注解，name是取名，用来被引用的。query是JPQL语句
@NamedQuery(name = "selectAllNamedQuery", query = "from Customer")
@NamedNativeQuery(name = "selectAllNamedNativeQuery", query = "select * from tb_customer", resultClass = Customer.class)
public class Customer {

    @Tolerate
    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long Id; // 客户的主键

    @Column(name = "customer_name")
    private String name; // 客户名称

    @Column(name="customer_age")
    private int age; // 客户年龄

    @Column(name="customer_sex")
    private boolean sex; // 客户性别

    @Column(name="customer_phone")
    private String phone; // 客户的联系方式

    @Column(name="customer_address")
    private String address; // 客户地址

}
```

测试方法：

```java
    /**
     * 查询全部：createNamedQuery()
     *     jqpl：from Customer
     *     sql：SELECT * FROM c_customer
     */
    @Test
    public void testNamedQuery() {
        // 1.获取entityManager对象 && 开启事务
        EntityManager entityManager = JpaUtils.getEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        // 2.查询全部：创建Query查询对象，query对象才是执行jqpl和原生sql的对象。发送查询，并封装结果集
        // 使用@NamedQuery的name
        entityManager.createNamedQuery("selectAllNamedQuery").getResultList().forEach(System.out::println);
        // 使用@NamedNativeQuery的name
        entityManager.createNamedQuery("selectAllNamedNativeQuery", Customer.class).getResultList().forEach(System.out::println);

        // 4.提交事务 && 释放资源
        transaction.commit();
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.customer_id as customer1_3_,
        customer0_.customer_address as customer2_3_,
        customer0_.customer_age as customer3_3_,
        customer0_.customer_name as customer4_3_,
        customer0_.customer_phone as customer5_3_,
        customer0_.customer_sex as customer6_3_ 
    from
        tb_customer customer0_
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
Hibernate: 
    select
        * 
    from
        tb_customer
Customer(Id=1, name=Sam, age=18, sex=true, phone=135000000001, address=广州)
Customer(Id=2, name=Mike, age=20, sex=true, phone=135000000002, address=广州)
Customer(Id=3, name=Nick, age=25, sex=true, phone=135000000003, address=深圳)
Customer(Id=4, name=Hom, age=30, sex=true, phone=135000000004, address=西安)
Customer(Id=5, name=Rachel, age=25, sex=false, phone=135000000005, address=北京)
Customer(Id=6, name=Kath, age=30, sex=true, phone=135000000006, address=武汉)
Customer(Id=7, name=Vivi, age=28, sex=false, phone=135000000007, address=南京)
Customer(Id=8, name=Oliver, age=30, sex=true, phone=135000000008, address=深圳)
Customer(Id=9, name=Angus, age=10, sex=false, phone=135000000009, address=广州)
Customer(Id=10, name=Wendy, age=15, sex=false, phone=135000000000, address=西安)
```



## 5、Query 接口方法介绍

JPQL 语言和原生 SQL 语句可以是 select 语句、update 语句或 delete语句（注意：无法执行insert语句），它们都通过 Query 接口封装执行。Query 接口封装了执行数据库查询的相关方法。调用 EntityManager 的 createQuery、createNamedQuery 及 createNativeQuery 方法可以获得查询对象，进而调用 Query 接口的相关方法来执行查询操作。

```java
/**
 * JPQL 语言和原生 SQL 语句可以是 select 语句、update 语句或 delete语句（没有insert），它们都通过 Query 接口封装执行
 * Query 接口封装了执行数据库查询的相关方法。调用 EntityManager 的 createQuery、createNamedQuery 及 createNativeQuery 方法可以获得查询对象，进而调用 Query 接口的相关方法来执行查询操作
 *
 * int executeUpdate()
 * 用于执行update或delete语句
 *
 * List getResultList()
 * 用于执行select语句并返回结果集实体列表
 *
 * Object getSingleResult()
 * 用于执行只返回单个结果实体的select语句
 *
 * Query setFirstResult(int startPosition)
 * 用于设置从哪个实体记录开始返回查询结果
 *
 * Query setMaxResults(int maxResult)
 * 用于设置返回结果实体的最大数。与setFirstResult结合使用可实现分页查询
 *
 * Query setFlushMode(FlushModeType flushMode)
 * 设置查询对象的Flush模式。参数可以取2个枚举值：FlushModeType.AUTO 为自动更新数据库记录，FlushMode Type.COMMIT 为直到提交事务时才更新数据库记录
 *
 * setHint(String hintName, Object value)
 * 设置与查询对象相关的特定供应商参数或提示信息。参数名及其取值需要参考特定 JPA 实现库提供商的文档。如果第二个参数无效将抛出IllegalArgumentException异常
 *
 * setParameter(int position, Object value)
 * 为查询语句的指定位置参数赋值。Position 指定参数序号，value 为赋给参数的值
 *
 * setParameter(int position, Date d, TemporalType type)
 * 为查询语句的指定位置参数赋 Date 值。Position 指定参数序号，value 为赋给参数的值，temporalType 取 TemporalType 的枚举常量，包括 DATE、TIME 及 TIMESTAMP 三个，用于将 Java 的 Date 型值临时转换为数据库支持的日期时间类型（java.sql.Date、java.sql.Time及java.sql.Timestamp）
 *
 * setParameter(int position, Calendar c, TemporalType type)
 * 为查询语句的指定位置参数赋 Calenda r值。position 指定参数序号，value 为赋给参数的值，temporalType 的含义及取舍同前
 *
 * setParameter(String name, Object value)
 * 为查询语句的指定名称参数赋值
 *
 * setParameter(String name, Date d, TemporalType type)
 * 为查询语句的指定名称参数赋 Date 值。用法同前
 *
 * setParameter(String name, Calendar c, TemporalType type)
 * 为查询语句的指定名称参数设置Calendar值。name为参数名，其它同前。该方法调用时如果参数位置或参数名不正确，或者所赋的参数值类型不匹配，将抛出 IllegalArgumentException 异常
 */
```



# Hibernate JPA 注解分析

> 参考资料 & 鸣谢：
>
> 1. JPA 批注参考：https://www.oracle.com/cn/middleware/technologies/ias/toplink-jpa-annotations.html
> 1. JPA 批注参考：https://blog.51cto.com/u_9058648/3562214
> 1. https://blog.csdn.net/yswKnight/article/details/79257372
> 2. https://blog.csdn.net/yiyelanxin/article/details/100107335
> 3. https://www.cnblogs.com/banywl/articles/14004262.html
> 4. https://blog.csdn.net/justlpf/article/details/84956212

## 1、常用注解

### Entity

@Entity 应用于实体类，并且使用默认的ORM规则，即 Class 名对应数据库表中表名，Class 字段名即表中的字段名（如想改变这种默认的ORM规则，就要使用 @Table 来改变 Class 名与数据库中表名的映射规则，@Column 来改变 Class 字段名与DB中表的字段名的映射规则）

元数据属性说明： 

- name（可选）：表名

下面的代码说明，Customer类对应数据库中的Customer表，其中name为可选，缺省类名即表名！

```java
@Entity(name="Customer")
public class Customer {}
```



### Table

@Table用来定义Entity主表的name，catalog，schema等属性。

元数据属性说明： 

- name（可选）：指定表的名称
- catalog（可选）：指定数据库名称（对应关系数据库中的catalog）
- schema（可选）：指定数据库的用户名（对应关系数据库中的schema）
- UniqueConstraints（可选）：指定唯一性字段约束的列，字段值唯一不重复（定义一个UniqueConstraint数组）
- indexes（可选）：索引，只有创建表的时候有用，默认不需要（indexes是一个数组）

```java
@Entity(name="Customer")
@Table(name = "tb_customer", catalog = "db_jpa", schema = "root", 
       uniqueConstraints={@UniqueConstraint(columnNames={"name", "age"})
})
public class Customer {}
```



### Entity 与 Table 对比

> 比较：@Entity(name = "实体名") 、@Entity @Table(name = "表名")
> 参考：https://mp.weixin.qq.com/s/GYMJT5sNoJ8-KcdIVQAoNw

我们在使用JPA的时候，需要做数据表和实体类的映射，@Entity注解的实体类，默认的实体名是非限定类名，对实体名首字母小写后得到表名，例如：

```java
@Entity
public class Comment{}
```

实体名：Comment，表名：comment。但实际项目中，有的表名以 tb_ 开头，那就映射不到了，那么就需要把表名写出来，有两种方式可以写：

1. @Entity(name = "实体名")，一般把实体名写成跟表名相同，相当于写出了表名
2. @Entity + @Table(name = "表名")

那么哪种方式好呢？粗略一看，应该是第一种好，但是应用到JPQL语句时，未必。

例如只用@Entity定义：

```java
// 定义实体类
@Entity(name = "tb_comment")
public class Comment{}

// JPQL 语句
@Query("update tb_comment c set c.author = ?1 where c.id= ?2 ")
```

这是JPQL语句，JPA一不小心就把`tb_comment`认为是表名，后面的 c.xxx 就会去表中找字段，**如果字段名跟实体的属性名不一样**，就报错了。

如果写成下面这样的，使用@Entity + @Table(name = "表名")：

```java
// 定义实体类
@Entity
@Table(name = "tb_comment")
public class Comment{}

// JPQL 语句
@Query("update Comment c set c.author = ?1 where c.id = ?2")
```

可读性好，Comment一看就是实体类名，后面的c.xxx就会去类中找属性名，但实体名不能用JPQL的保留字。

**结论：**

1. @Entity(name = "实体名")：会降低JPQL语句的可读性，一般实体名不会跟JPQL保留字重复
2. @Entity + @Table(name = "表名")：可读性强，更容易看出来面向对象编程思想，但要避免实体名与JPQL保留字重复





### Column

> 基于Column注解的columnDefinition用法：https://www.jb51.net/article/226168.htm

@Column应用于实体类属性，可以指定数据库表字段的名字和其他属性。

```java
@Target({METHOD, FIELD}) 
@Retention(RUNTIME)
public @interface Column {
    String name() default "";
    boolean unique() default false;
    boolean nullable() default true;
    boolean insertable() default true;
    boolean updatable() default true;
    String columnDefinition() default "";
    String table() default "";
    int length() default 255;
    int precision() default 0;
    int scale() default 0;
}
```

元数据属性说明： 

1. name：表示数据库表中该字段的名称，默认情形属性名称一致
2. unique：表示该字段是否是唯一标识，默认：false
3. nullable：表示该字段是否允许为null，默认：true
4. insertable：表示在ORM框架执行插入操作时，该字段是否应出现INSETRT语句中，默认：true
5. updateable：表示在ORM框架执行更新操作时，该字段是否应该出现在UPDATE语句中，默认：true。对于一经创建就不能更改的字段，该属性非常有用，比如`email`属性。（insertable、updatable属性一般多用于只读的属性，例如主键和外键等。这些字段的值通常是自动生成的）
6. columnDefinition：表示该字段在数据库中的实际类型。通常ORM框架可以根据属性类型自动判断数据库中字段的类型，但是依然有些例外：Date类型无法确定数据库中字段类型究竟是DATE、TIME、TIMESTAMP。此外，String的默认映射类型为VARCHAR，如果希望将String类型映射到特定数据库的BLOB或TEXT字段类型，则需要进行设置
7. table：表示当映射多个表时，指定表的表中的字段。默认值为主表的表名
8. length：表示该字段的大小，仅对String类型的字段有效
9. precision：表示精度，当字段类型为double时，precision表示数值的总长度
10. scale：表示精度，当字段类型为double时，scale表示小数点所占的位数

> Column注解的columnDefinition用法：

columnDefinition属性表示创建表时，该字段创建的SQL语句，一般用于通过Entity生成表定义时使用，如果数据库中表已经建好，该属性没有必要使用

```java
/**
 * 1、指定字段类型、长度、是否允许null、是否唯一、默认值
 */
@Column(name = "code",columnDefinition = "Varchar(100) not null default'' unique")
private String code;

/**
 * 2、需要特殊指定字段类型的情况
 * String默认映射类型为VARCHAR, 如果要将String类型映射到特定数据库的BLOB或TEXT字段类型
 */
@Column(name = "text",columnDefinition="TEXT")
private String text;
@Column(name = "salary", columnDefinition = "decimal(5,2)")
private BigDecimal salary;
@Column(name="birthday",columnDefinition="date")
private Date birthday;
@Column(name="createTime",columnDefinition="datetime")
private Date createTime;
```



### Basic

应用于实体类属性，表明该字段是要映射到数据库表，@Entity标注的实体类，所有属性默认为@Basic（所以默认该注解可以不加）

元数据属性说明： 

- fetch：属性的读取策略，有两种策略：`FetchType.LAZY`（延迟加载）和`FetchType.EAGER`（立即加载），默认：`EAGER`
- optional：表示该属性是否可以为null，默认值：true

PS 注意：@Basic(fetch =FetchType.LAZY) 标注某属性时，表示只有调用Hibernate对象的该属性的get方法时，才会从数据库表中查找对应该属性的字段值



### Transient

@Transient作用在类属性上，与@Basic作用相反，表明该属性不需要持久化，JPA映射数据时忽略该属性



### Temporal

@Temporal作用在类属性上，用来设置Date类型的属性映射到数据库时的精度。当我们使用到 java.util 包中的时间日期类型，则需要此注释来说明转化成 java.util包中的类型。

1. @Temporal(TemporalType.DATE)：映射为日期 date （只有日期）等于Java的`java.sql.Date`
2. @Temporal(TemporalType.TIME)：映射为日期 time （是有时间）等于Java的`java.sql.Time`
3. @Temporal(TemporalType.TIMESTAMP)：映射为日期 date + time （日期 + 时间）等于Java的`java.sql.Timestamp`

**PS 注意**：@Temporal注解只能映射`java.util.Date or java.util.Calendar`，这两种时间类型，否则将报如下错误：

```verilog
Hibernate.AnnotationException: @Temporal should only be set on a java.util.Date or java.util.Calendar property: entity.User.createLocalDateTime
```

1、首先定义实体类，并给需要处理的时间字段加上@Temporal注解。示例代码：

```java
@Data
@Entity
@Table(name = "sys_user")
public class User implements Serializable {
    @Id
    private Long id;
    @Temporal(TemporalType.DATE)
    private java.util.Date createDate;
    @Temporal(TemporalType.TIME)
    private java.util.Date createTime;
    @Temporal(TemporalType.TIMESTAMP)
    private java.util.Date createTIMESTAMP;
    // @Temporal只能映射 ava.util.Date or java.util.Calendar 两种时间类型
    // @Temporal(TemporalType.TIMESTAMP)
    // private java.time.LocalDateTime createLocalDateTime;
}
```

2、启动项目或者测试类，查看建表语句

```sql
create table sys_user (
    id bigint not null,
    createDate date,
    createTIMESTAMP timestamp,
    createTime time,
    primary key (id)
)
```

3、测试代码

```java
import entity.User;
import org.junit.Test;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class TestDemo {

    @Test
    public void testSave() {
        // 1.加载配置文件创建工厂（实体管理器工厂）对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        // 2.通过实体管理器工厂获取实体管理器
        EntityManager em = factory.createEntityManager();
        // 3.获取事务对象，开启事务
        EntityTransaction tx = em.getTransaction();

        // 开启事物新增数据，手动设置时间
        tx.begin();
        User user = new User();
        user.setId(1L);
        user.setCreateDate(new java.util.Date());
        user.setCreateTime(new java.util.Date());
        user.setCreateTIMESTAMP(new java.util.Date());
        em.persist(user);
        tx.commit();

        em.refresh(user);
        User user_selelct = em.find(User.class, 1L);
        System.out.println(user_selelct);

        // 6.释放资源
        em.close();
        factory.close();
    }
}
```

4、查看输出日志

```verilog
User(id=1, createDate=2022-02-01, createTime=15:13:36, createTIMESTAMP=2022-02-01 15:13:36.325)
```



### CreationTimestamp

> @CreationTimestamp 与 @UpdateTimestamp 注解是JPA用来自动创建、更新时间戳

使用该注解可以让Hibernate JPA在插入时针对注解的属性对应的日期类型创建默认值。此批注没有属性。

```java
/**
 * Marks a property as the creation timestamp of the containing entity. The property value will be set to the current
 * VM date exactly once when saving the owning entity for the first time.
 * <p>
 * Supported property types:
 * <ul>
 * <li>{@link java.util.Date}</li>
 * <li>{@link java.util.Calendar}</li>
 * <li>{@link java.sql.Date}</li>
 * <li>{@link java.sql.Time}</li>
 * <li>{@link java.sql.Timestamp}</li>
 * <li>{@link java.time.Instant}</li>
 * <li>{@link java.time.LocalDate}</li>
 * <li>{@link java.time.LocalDateTime}</li>
 * <li>{@link java.time.LocalTime}</li>
 * <li>{@link java.time.MonthDay}</li>
 * <li>{@link java.time.OffsetDateTime}</li>
 * <li>{@link java.time.OffsetTime}</li>
 * <li>{@link java.time.Year}</li>
 * <li>{@link java.time.YearMonth}</li>
 * <li>{@link java.time.ZonedDateTime}</li>
 * </ul>
 *
 * @author Gunnar Morling
 */
@ValueGenerationType(generatedBy = CreationTimestampGeneration.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface CreationTimestamp {
}
```

示例代码参考@UpdateTimestamp



### UpdateTimestamp

使用该注解可以让Hibernate JPA在更新时时针对注解的属性对应的日期类型创建默认值。此批注没有属性。

```java
/**
 * Marks a property as the update timestamp of the containing entity. The property value will be set to the current VM
 * date whenever the owning entity is updated.
 * <p>
 * Supported property types:
 * <ul>
 * <li>{@link java.util.Date}</li>
 * <li>{@link java.util.Calendar}</li>
 * <li>{@link java.sql.Date}</li>
 * <li>{@link java.sql.Time}</li>
 * <li>{@link java.sql.Timestamp}</li>
 * <li>{@link java.time.Instant}</li>
 * <li>{@link java.time.LocalDate}</li>
 * <li>{@link java.time.LocalDateTime}</li>
 * <li>{@link java.time.LocalTime}</li>
 * <li>{@link java.time.MonthDay}</li>
 * <li>{@link java.time.OffsetDateTime}</li>
 * <li>{@link java.time.OffsetTime}</li>
 * <li>{@link java.time.Year}</li>
 * <li>{@link java.time.YearMonth}</li>
 * <li>{@link java.time.ZonedDateTime}</li>
 * </ul>
 *
 * @author Gunnar Morling
 */
@ValueGenerationType(generatedBy = UpdateTimestampGeneration.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface UpdateTimestamp {
}
```

1、一般情况下创建、更新时间戳注解 @CreationTimestamp、@UpdateTimestamp 会一起使用。下面是代码示例：

```java
@Data
@Entity
@Table(name = "sys_user")
public class User implements Serializable {
    @Id
    private Long id;
    @CreationTimestamp
    private java.util.Date createDate;
    @UpdateTimestamp
    private java.sql.Timestamp updateTimestamp;
    @CreationTimestamp
    private java.time.LocalDate createLocalDate;
    @UpdateTimestamp
    private java.time.LocalTime updateLocalTime;
    @UpdateTimestamp
    private java.time.LocalDateTime updateLocalDateTime;
}
```

2、启动项目或者测试类，查看JPA自动建表语句。**可以看到他们的对应关系，使用时需要特别注意。**

```sql
create table sys_user (
    id bigint not null,
    createDate timestamp,
    createLocalDate date,
    updateLocalDateTime timestamp,
    updateLocalTime time,
    updateTimestamp timestamp,
    primary key (id)
)
```

3、运行单元测试，查看手动设置时间与JPA自动设置时间的区别

```java
import entity.User;
import org.junit.Test;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.Date;

public class TestDemo {

    @Test
    public void testSave() {
        // 1.加载配置文件创建工厂（实体管理器工厂）对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        // 2.通过实体管理器工厂获取实体管理器
        EntityManager em = factory.createEntityManager();
        // 3.获取事务对象，开启事务
        EntityTransaction tx = em.getTransaction();

        // 开启事物新增数据，手动设置时间
        tx.begin();
        User user = new User();
        user.setId(1L);
        user.setCreateDate(new Date());
        user.setCreateLocalDate(LocalDate.now());
        user.setUpdateTimestamp(new Timestamp(System.currentTimeMillis()));
        user.setUpdateLocalTime(LocalTime.now());
        user.setUpdateLocalDateTime(LocalDateTime.now());
        em.persist(user);
        tx.commit();

        em.refresh(user);
        User user_selelct = em.find(User.class, 1L);
        System.out.println(user_selelct);

        // 开启事物新增数据，JPA自动设置时间
        tx.begin();
        User user2 = new User();
        user2.setId(2L);
        em.persist(user2);
        tx.commit();

        em.refresh(user2);
        User user_selelct2 = em.find(User.class, 2L);
        System.out.println(user_selelct2);

        // 6.释放资源
        em.close();
        factory.close();
    }
}
```

4、查看输出日志

```verilog
User(id=1, createDate=2022-01-31 21:59:16.181, updateTimestamp=2022-01-31 21:59:16.181, createLocalDate=2022-01-31, updateLocalTime=21:59:16, updateLocalDateTime=2022-01-31T21:59:16.181568)
User(id=2, createDate=2022-01-31 21:59:16.245, updateTimestamp=2022-01-31 21:59:16.245, createLocalDate=2022-01-31, updateLocalTime=21:59:16, updateLocalDateTime=2022-01-31T21:59:16.245574)
```



### 自定义默认值设置注解

> 参考文献 & 鸣谢：
>
> 1. JPA注解@CreationTimestamp和@UpdateTimestamp：https://blog.csdn.net/z185665096/article/details/106090223
> 2. JPA设置默认值、Timestamp设置、自动获取时间：https://blog.csdn.net/ctwy291314/article/details/88250205

比如，用户名，插入时可以根据当前线程直接获取，然后利用注解赋值，这样在业务处理时就可以不用关注该属性。类似还有更新时自动赋值操作人等。

- 新增注解@CreationUser

```java
/**
 * Hibernate新增插入时自动填充
 */
@ValueGenerationType(
    generatedBy = CreationUserGeneration.class
)
@Retention(RetentionPolicy.RUNTIME)
public @interface CreationUser {
}
```

- 新增注解值生成类。主要继承`org.hibernate.tuple.AnnotationValueGeneration<A extends Annotation>`

```java
import org.hibernate.tuple.AnnotationValueGeneration;
import org.hibernate.tuple.GenerationTiming;
import org.hibernate.tuple.ValueGenerator;

/**
 * 针对加了注解{@link @CreationUser}的属性在插入时，自动根据当前线程获取用户信息赋值
 */
public class CreationUserGeneration implements AnnotationValueGeneration<CreationUser> {
    private ValueGenerator<?> generator;

    public CreationUserGeneration() {
    }

    @Override
    public void initialize(CreationUser annotation, Class<?> propertyType) {
        // 主要逻辑在这里实现
        this.generator = (ValueGenerator<Object>) (session, o) -> UserContextHolder.getUserName();
    }

    @Override
    public GenerationTiming getGenerationTiming() {
        // 只有插入时，每次都修改使用GenerationTiming.ALWAYS
        return GenerationTiming.INSERT;
    }

    @Override
    public ValueGenerator<?> getValueGenerator() {
        return this.generator;
    }

    @Override
    public boolean referenceColumnInSql() {
        return false;
    }

    @Override
    public String getDatabaseGeneratedReferencedColumnValue() {
        return null;
    }
}
```

就这样就OK了，注解@CreationUser就可以直接在实体类使用了。



## 2、主键注解

### Id 

@Id 定义了映射到数据库表的主键的属性，一个实体只能有一个属性被映射为主键，可置于getXxxx()前。



### IdClass

当实体类使用复合主键时，需要定义一个类作为 ID 实体类。作为联合主键类，需要满足以下要求：

1. 必须实现Serializable
2. 必须有默认的public无参构造方法
3. 必须覆盖equals和hashCode方法（EntityManager通过find方法查找Entity时是根据equals来判断的）

元数据属性说明： 

- value: id class的类名

假设user_article表中的联合主键是 title 与 create_user_id，联合主键类代码如下：

```java
@Data
public class UserArticleKey implements Serializable {
    private String title;
    private Long createUserId;
    public UserArticleKey() {
    }
    public UserArticleKey(String title, String content, Long createUserId) {
        this.title = title;
        this.createUserId = createUserId;
    }
}
```

user_article表实体类如下：

```java
@Data
@Entity
@Table(name="user_article")
@IdClass(value = UserArticleKey.class)
public class UserArticle {
    private Integer id;
    @Id
    private String title;
    @Id
    private Long createUserId;
}
```



### EmbeddedId

使用 @EmbeddedId 批注指定一个由实体拥有的可嵌入**复合主键**类（通常由两个或更多基元类型或 JDK 对象类型组成）。从原有数据库映射时（此时数据库键由多列组成），通常将出现**复合主键**。（或者也可以使复合主键类（IdClass）成为非嵌入类）

复合主键类具有下列特征：

- 它是一个普通的旧式 Java 对象 (POJO) 类。

- 它必须为 public，并且必须有一个 public 无参数构造函数。

- 如果使用基于属性的访问，则主键类的属性必须为 public 或 protected。

- 它必须是可序列化的。

- 它必须定义 equals 和 hashCode 方法。

- 这些方法的值相等性的语义必须与键映射到的数据库类型的数据库相等性一致。
- 此批注没有属性，一般与 @Embeddable（注解在类上，表示此类是可以被其他类嵌套）配合使用。
- @Embeddable 还能多层嵌套使用

假设sys_user表中的联合主键是 id 与 username，联合主键类代码如下：

1、主键的实体类

```java
@Data
@Embeddable
public class UserPK implements Serializable {
    @Column(name = "id")
    private Long id;
    @Column(name = "username")
    private String username;
}
```

2、表映射的持久类（持久类里要有一个注解@EmbeddedId修饰主键的成员变量，其他的和普通的Entity类一样。它所对应复合主键类需要使用@Embeddable注解）

```java
@Data
@Entity
@Table(name = "sys_user")
public class User implements Serializable {
    @EmbeddedId
    private UserPK userPK;
    private String password;
}
```

3、启动项目或者测试类，查看JPA自动建表语句，可以发现把 id 与 usrname 创建成联合主键了

```sql
create table sys_user (
    id bigint not null,
    username varchar(255) not null,
    password varchar(255),
    primary key (id, username)
)
```

4、普通Java项目测试

```java
import entity.User;
import entity.UserPK;
import org.junit.Test;
import javax.persistence.*;

public class TestDemo {

    @Test
    public void testSave() {
        // 1.加载配置文件创建工厂（实体管理器工厂）对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        // 2.通过实体管理器工厂获取实体管理器
        EntityManager em = factory.createEntityManager();
        // 3.获取事务对象，开启事务
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        UserPK userPK = new UserPK();
        userPK.setId(1L);
        userPK.setUsername("admin");
        User user = new User();
        user.setUserPK(userPK);
        user.setPassword("password");
        em.persist(user);
        tx.commit();

        em.refresh(user);
        User user_selelct = em.find(User.class, userPK);
        System.out.println(user_selelct);

        // 6.释放资源
        em.close();
        factory.close();
    }
}
```

输出日志：

```verilog
Hibernate: 
    insert 
    into
        sys_user
        (password, id, username) 
    values
        (?, ?, ?)
Hibernate: 
    select
        user0_.id as id1_0_0_,
        user0_.username as username2_0_0_,
        user0_.password as password3_0_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=? 
        and user0_.username=?
User(userPK=UserPK(id=1, username=admin), password=password)
```

5、SpringBoot项目测试，查看UserRepository接口定义查询方法

```java
@Repository
public interface UserRepository extends JpaRepository<User, UserPK> {
    // 只查找一个条件（只查询复合主键中的一个字段）
    List<User> findByIdUsername(String username);

    // 查找两个条件（如果有三个复合主键可以这样使用，如果只有两个复合主键就直接findById就行了）
    List<User> findByIdIdAndUsername(String id,String username);
    
    // 查询所有复合主键就直接findById就行
    User findById(UserPK, userpk);
}
```



### GeneratedValue

> 请参考本人的其他篇章，Hibernate JPA 主键策略的所有文章

@GeneratedValue主要与@Id一同使用，定义主键的生成策略，通过`strategy`属性指定。

元数据属性说明： 

- strategy（表示主键生成策略）：
  1. GenerationType.AUTO：JPA自动选择合适的策略，是默认选项
  2. GenerationType.IDENTITY： 采用数据库ID自增长的方式生成主键值，一般用于MySQL数据库,Oracle不支持这种方式
  3. GenerationType.SEQUENCE：通过序列产生主键，通过@SequenceGenerator注解指定序列名,MySql不支持这种方式
  4. GenerationType.TABLE：通过表产生主键，会生成一张表模拟序列产生主键,该策略通用性强易于数据库的移植，效率低
- generator（表示生成器的名字）：这个属性通常和ORM框架相关。例如：Hibernate可以指定uuid等主键生成方式（要和@SequenceGenerator(name = "seq_tbl_user", sequenceName = "seq_tbl_user", allocationSize = 1）注解配合使用，其中name指定生成器的名字（与generator的值一样），sequenceName指定数据库中定义序列的名字，allocationSize指定序列每次增长1 )

```java
@javax.persistence.Id
@javax.persistence.GeneratedValue(generator="xxx",strategy=GenerationType.AUTO)

strategy:表示主键生成策略,有如下四种方式:
方式一：@GeneratedValue(strategy=GenerationType.AUTO) 默认策略，生成方式取决于底层的数据库。
方式二：@GeneratedValue(strategy = GenerationType.IDENTITY)指定“自动增长”策略，适用于MySQL。 
方式三：@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "seq_tbl_user")
       指定“序列”策略，适用于Oracle。其中generator表示生成器的名字，这个属性通常和ORM框架相关。
       例如,Hibernate可以指定uuid等主键生成方式(要和@SequenceGenerator(name = "seq_tbl_user", 
       sequenceName = "seq_tbl_user", allocationSize = 1)注解配合使用，其中name指定生成器的名 
       字（与generator的值一样），sequenceName指定数据库中定义序列的名字，allocationSize指定序
       列每次增长1 )
方式四：@GeneratedValue(strategy=GenerationType.TABLE)  使用一个特定的数据库表格来保存主键
```



### SequenceGenerator 

> 请参考本人的其他篇章，Hibernate JPA 主键策略：SEQUENCE 策略

### GenericGenerator

> 请参考本人的其他篇章，Hibernate JPA 主键策略：自定义主键生成器

### TableGenerator

> 请参考本人的其他篇章，Hibernate JPA 主键策略：TABLE 策略



## 3、嵌入注解

> @Embedded 与 @Embeddable：当一个实体类要在多个不同的实体类中进行使用，而其不需要生成数据库表
>
> 1. @Embeddable：注解在类上，表示此类是可以被其他类嵌套
> 2. @Embedded：注解在属性上，表示嵌套被@Embeddable注解的同类型类
>
> @AttributeOverride、@AttributeOverrides 注解重定义其父类或者嵌套类属性映射到数据库表中字段

### Embeddable

默认情况下，JPA 持续性提供程序假设每个实体均持久保存到它自己的数据库表。

使用 @Embeddable 批注指定一个类，该类的实例存储为拥有实体的固有部分并共享该实体的身份。嵌入对象的每个持久属性或字段都将映射到实体的数据库表。此批注没有属性。

下面使用此注解批注 实体类UserCommon.java，然后可以使用 @Embedded 或 @EmbeddedId 把UserCommon 注入另一个实体类中当做其属性。（可以理解为单独抽离几个字段出来独立到另一个类中）

```java
@Data
@Embeddable
public class UserCommon {
    @Column(name = "startDate")
    private java.util.Date startDate;
    private java.util.Date endDate;
}
```

PS：Embeddable一般与Embedded或EmbeddedId组合使用：@Embeddable + @Embedded、@Embeddable + @EmbeddedId



### Embedded

使用 @Embedded 批注指定一个持久字段，该字段必须为引用类型，且还被 @Embeddable 批注。默认情况下 @Embeddable 类中指定的列定义适用于 @Embedded 类。如果要覆盖这些列定义可以使用 @AttributeOverride。

此批注没有属性。下面使用@Embeddable + @Embedded组合定义代码示例：

1、使用@Embeddable定义公共实体类

```java
@Data
@Embeddable
public class UserCommon {
    @Column(name = "startDate")
    private java.util.Date startDate;
    private java.util.Date endDate;
}
```

2、表映射的持久类（持久类里要有一个注解@Embedded修饰主键的成员变量，其他的和普通的Entity类一样。它所对应复合主键类需要使用@Embeddable注解）

```java
@Data
@Entity
@Table(name = "sys_user")
public class User implements Serializable {
    @EmbeddedId
    private UserPK userPK;
    @Embedded
    private UserCommon userCommon;
    private String password;
}
```

3、启动项目或者测试类，查看JPA自动建表语句，可以发现把 startDate 与 endDate 也创建到实体类 User 映射的数据表中了

```sql
create table sys_user (
    id bigint not null,
    username varchar(255) not null,
    password varchar(255),
    endDate timestamp,
    startDate timestamp,
    primary key (id, username)
)
```

> **备注**：正常用法一般都是 @Embedded + @Embeddable 同时使用，实际上单独使用任意一个，效果也是一样（可读性差）



### MappedSuperclass

> @MappedSuperclass：（很重要）实现将实体类的多个属性分别封装到不同的非实体类中
>
> 1. 注解的类将不是完整的实体类，不会映射到数据库表，但其属性将映射到子类的数据库字段
> 2. 注解的类不能再标注@Entity或@Table注解，也无需实现序列化接口
> 3. 注解的类继承另一个实体类或标注@MappedSuperclass类，他可使用@AttributeOverride或@AttributeOverrides注解重定义其父类属性映射到数据库表中字段

当我们在定义多个实体类时，可能会遇到这几个实体类都有几个共同的属性，这时就会出现很多重复代码。 这时我们可以选择用注解@MappedSuperclass。编写一个父类（基类），将这些共同属性放到这个父类中，并且在父类上加上@MappedSuperclass注解。

**使用方法和注意事项：**

- @MappedSuperclass 注解使用在父类上面，是用来标识父类的
- @MappedSuperclass 标识的类不是一个完整的实体类，它不会映射到数据库表，但是它的属性都将映射到其子类的数据库字段中
- @MappedSuperclass 标识的类不能再有@Entity或@Table，也无需实现序列化接口，但其子类可以有@Entity或@Table

该批注没有属性。可以在子类中使用  @AttributeOverride 或  @AssociationOverride 批注来覆盖超类的映射配置。

```java
@Data
@Entity
@Table(name = "sys_user")
public class User extends BaseEntity implements Serializable {
    private String admin;
    private String password;
}

@Data
@MappedSuperclass
abstract class BaseEntity {
    @Id
    private Long id; // 数据库主键
    @Column(name = "creation_time")
    private java.util.Date creationTime; //创建时间
    @Column(name = "update_time")
    private java.util.Date updateTime; //修改时间
}
```

启动项目或者测试类，查看JPA自动建表语句，可以发现 id、creation_time、update_time 字段也新增了

```sql
create table sys_user (
    id bigint not null,
    creation_time timestamp,
    update_time timestamp,
    admin varchar(255),
    password varchar(255),
    primary key (id)
)
```



### AttributeOverride

@AttributeOverride 批注是用来覆盖 @Embeddable 类或者 @MappedSuperclas 类中字段的列属性 （包含要覆盖的`@Embeddable`类中字段名name和新增的`@Column`字段的属性；）

元数据属性说明：

- name：如果使用了基于属性的访问，则映射的为嵌入对象中的属性名称，如果使用了基于字段的访问，则映射的为字段名称
- column：映射到持久属性的 @Column。映射类型将与可嵌套类或映射超类中定义的类型相同

1、下面使用 @AttributeOverride 注解重写 @Embeddable 批注的类中的属性字段。为了方便查看使用了内部类。

```java
@Data
@Entity
@Table(name = "sys_user")
public class User implements Serializable {
    @EmbeddedId
    private UserPK userPK;
    @Embedded
    @AttributeOverrides({ //把属性映射的列名称进行重写，startDate 和 endDate 是UserCommon类中的字段
            @AttributeOverride(name = "startDate", column = @Column(name = "start_date", nullable = false)),
            @AttributeOverride(name = "endDate", column = @Column(name = "end_date", nullable = false))
    })
    private UserCommon userCommon;
    private String password;

    @Data
    @Embeddable
    public static class UserCommon implements Serializable {
        @Column(name = "startDate")
        private java.util.Date startDate;
        private java.util.Date endDate;
    }

    @Data
    @Embeddable
    public static class UserPK implements Serializable {
        @Column(name = "id")
        private Long id;
        @Column(name = "username")
        private String username;
    }
}
```

2、启动项目或者测试类，查看JPA自动建表语句，可以发现startDate与endDate字段都被重写了，并且加了不能为空的约束。

```sql
create table sys_user (
    id bigint not null,
    username varchar(255) not null,
    password varchar(255),
    end_date timestamp not null,
    start_date timestamp not null,
    primary key (id, username)
)
```

3、下面使用 @AttributeOverride 注解重写 @MappedSuperclass 批注的类中的属性字段。

```java
@Data
@Entity
@Table(name = "sys_user")
@AttributeOverrides({ // 把属性映射的列名称进行重写，creationTime 和 updateTime 是BaseEntity类中的字段
        @AttributeOverride(name = "creationTime", column = @Column(name = "create_date", nullable = false)),
        @AttributeOverride(name = "updateTime", column = @Column(name = "update_date", nullable = false))
})
public class User extends BaseEntity implements Serializable {
    private String admin;
    private String password;
}

@Data
@MappedSuperclass
abstract class BaseEntity {
    @Id
    private Long id; // 数据库主键
    @Column(name = "creation_time")
    private java.util.Date creationTime; //创建时间
    @Column(name = "update_time")
    private java.util.Date updateTime; //修改时间
}
```

4、启动项目或者测试类，查看JPA自动建表语句，可以发现creationTime与updateTime字段都被重写了，并且加了不能为空的约束。

```sql
create table sys_user (
    id bigint not null,
    create_date timestamp not null,
    update_date timestamp not null,
    admin varchar(255),
    password varchar(255),
    primary key (id)
)
```



### AttributeOverrides

@AttributeOverrides 里面只包含了@AttributeOverride 类型数组，用法直接参考@AttributeOverride的即可。



### AssociationOverride

默认情况下，JPA 持续性提供程序自动假设子类继承超类中定义的持久属性及其关联映射。

如果继承的列定义对实体不正确（例如：继承的列名与已经存在的数据模型不兼容或作为数据库中的列名无效），请使用 @AssociationOverride 批注自定义从 @MappedSuperclass 或 @Embeddable 继承的 @OneToOne 或 @ManyToOne 映射，以更改与字段或属性关联的 @JoinColumn。如果有多个要进行的 @AssociationOverride 更改，则必须使用 @AssociationOverrides。

要自定义基本映射以更改它的 @Column，请使用 @AttributeOverride。

元数据属性说明：

- name：如果使用了基于属性的访问，则映射的为嵌入对象中的属性名称，如果使用了基于字段的访问，则映射的为字段名称
- joinColumns：要指定映射到持久属性的连接列，请将 joinColums 设置为 JoinColumn 实例的数组，映射类型将与可嵌套类或映射的超类中定义的类型相同。

1、这里使用@MappedSuperclass示例，先测试没有使用@AssociationOverride注解的效果。

```java
import lombok.Data;
import javax.persistence.*;

@Data
@Entity
@Table(name = "sys_user")
public class User extends BaseEntity implements java.io.Serializable {
    @Id
    private Long id;
    private String admin;
    private String password;
}

@Data
@MappedSuperclass
abstract class BaseEntity {
    @ManyToOne
    @JoinColumn(name="address_id")
    private Address address;
}

@Data
@Entity
@Table(name = "sys_address")
class Address implements java.io.Serializable {
    @Id
    @Column(name = "address_id")
    private Long addressId;
    private String country;
}
```

查看输出日志：

```sql
create table sys_address (
    address_id bigint not null,
    country varchar(255),
    primary key (address_id)
);
create table sys_user (
    id bigint not null,
    admin varchar(255),
    password varchar(255),
    address_id bigint,
    primary key (id)
);
alter table sys_user 
       add constraint FK9abddsodne05nqpe2if8gjekl 
       foreign key (address_id) 
       references sys_address;
```

2、再使用@MappedSuperclass + AssociationOverride示例查看效果。

```java
import lombok.Data;
import javax.persistence.*;

@Data
@Entity
@Table(name = "sys_user")
@AssociationOverride(name="address", joinColumns=@JoinColumn(name="fk_address_id"))
public class User extends BaseEntity implements java.io.Serializable {
    @Id
    private Long id;
    private String admin;
    private String password;
}

@Data
@MappedSuperclass
abstract class BaseEntity {
    @ManyToOne
    @JoinColumn(name="address_id")
    private Address address;
}

@Data
@Entity
@Table(name = "sys_address")
class Address implements java.io.Serializable {
    @Id
    private Long id;
    private String country;
}
```

查看输出日志：

```sql
create table sys_address (
    id bigint not null,
    country varchar(255),
    primary key (id)
);
create table sys_user (
    id bigint not null,
    admin varchar(255),
    password varchar(255),
    fk_address_id bigint,
    primary key (id)
);
alter table sys_user 
       add constraint FKmas1faywy7oyb4vr78hhm6kp0 
       foreign key (fk_address_id) 
       references sys_address;
```

从日志中可以看出@AssociationOverride的作用主要是重写了@JoinColumn注解的属性。



### AssociationOverrides

@AssociationOverrides 里面只包含了@AssociationOverride 类型数组，用法直接参考@AssociationOverride的即可。

> 简单对比@AssociationOverride与AttributeOverride的作用与区别：
>
> 1. @AssociationOverride主要重写@MappedSuperclass 或 @Embeddable 继承的 @JoinColumn 注解字段
> 2. @AttributeOverride主要重写@MappedSuperclass 或 @Embeddable 继承的 @Column 注解字段



## 4、次用注解

### SecondaryTable

@SecondaryTable将一个实体映射到多个数据库表中。

元数据属性说明：

- name：表名 
- catalog：对应关系数据库中的catalog 
- schema：对应关系数据库中的schema 
- pkJoinColumns：定义一个PrimaryKeyJoinColumn数组，指定从表的主键列 
- UniqueConstraints：定义一个UniqueConstraint数组，指定需要建唯一约束的列

```java
// 如果只多映射一个数据表的话可以只用@SecondaryTable
@Entity
@Table(name = "tb_customer")
@SecondaryTables({
    @SecondaryTable(name = "tb_address", pkJoinColumns = {@PrimaryKeyJoinColumn(name="address_id",referencedColumnName="id")}),
    @SecondaryTable(name = "tb_comments", pkJoinColumns = {@PrimaryKeyJoinColumn(name="comments_id",referencedColumnName="id")})
})
public class Customer implements Serializable {
    @Id
    private Long id;
    private String username;
    private String password;
    @Column(table = "tb_address")
    private String street; 
    @Column(table = "tb_address")
    private String city; 
    @Column(table = "tb_address")
    private String conutry; 
    @Column(table = "tb_comments")
    private String title; 
    @Column(table = "tb_comments")
    private String comments;
}
```

@Column中的table属性的值指定属性存储的哪张数据库表。没有用@Column中table值注解改变的属性，将会存在于默认tb_forum主表中。

```sql
mysql> desc tb_customer;
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| id       | bigint(20)   | NO   | PRI | NULL    |       |
| password | varchar(255) | YES  |     | NULL    |       |
| username | varchar(255) | YES  |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
3 rows in set (0.02 sec)

mysql> desc tb_address;
+------------+--------------+------+-----+---------+-------+
| Field      | Type         | Null | Key | Default | Extra |
+------------+--------------+------+-----+---------+-------+
| city       | varchar(255) | YES  |     | NULL    |       |
| conutry    | varchar(255) | YES  |     | NULL    |       |
| street     | varchar(255) | YES  |     | NULL    |       |
| address_id | bigint(20)   | NO   | PRI | NULL    |       |
+------------+--------------+------+-----+---------+-------+
4 rows in set (0.02 sec)

mysql> desc tb_comments;
+-------------+--------------+------+-----+---------+-------+
| Field       | Type         | Null | Key | Default | Extra |
+-------------+--------------+------+-----+---------+-------+
| comments    | varchar(255) | YES  |     | NULL    |       |
| title       | varchar(255) | YES  |     | NULL    |       |
| comments_id | bigint(20)   | NO   | PRI | NULL    |       |
+-------------+--------------+------+-----+---------+-------+
3 rows in set (0.03 sec)
```



### SecondaryTables

当一个实体类映射到一个主表和多个从表时，用@SecondaryTables来定义各个从表的属性。

元数据属性说明：

- columnNames:定义一个字符串数组，指定要建唯一约束的列名

案例可以参考上面@SecondaryTable的示例代码。



### PrimaryKeyJoinColumn

@PrimaryKeyJoinColumn一般在三种情况下会用到（@PrimaryKeyJoinColumns是用来装@PrimaryKeyJoinColumn数组的）

1. 继承
2. 实体类映射到一个或多个从表。从表根据主表的主键列（列名为referencedColumnName值的列），建立一个类型一样的主键列，列名由name属性定义
3. one-to-one关系，关系维护端的主键作为外键指向关系被维护端的主键，不再新建一个外键列

元数据属性说明：

- name：列名
- referencedColumnName：该列引用列的列名
- columnDefinition：定义建表时创建此列的DDL

下面的代码说明Customer映射到两个表，主表tb_customer，从表tb_address，从表需要建立主键列address_id，该列和主表的主键列id除了列名不同，其他定义一样

```java
@Entity
@Table(name = "tb_customer")
@SecondaryTable(name = "tb_address", pkJoinColumns = {@PrimaryKeyJoinColumn(name="address_id",referencedColumnName="id")})
public class Customer implements Serializable {
    @Id
    private Long id;
    private String username;
    private String password;
    @Column(table = "tb_address")
    private String street; 
    @Column(table = "tb_address")
    private String city; 
    @Column(table = "tb_address")
    private String conutry; 
}
```

下面的代码说明Customer和Address是一对一关系，Customer的主键列id作为外键指向Address的主键列address_id（这个本人暂时还没研究清楚）

```java
@Entity
@Table(name = "tb_customer")
public class Customer {
    private Long customer_id;
    @OneToOne
    @PrimaryKeyJoinColumn(name = "customer_id", referencedColumnName="address_id")
    private Address address;
}
```



### PrimaryKeyJoinColumns

如果实体类使用了复合主键，指定单个PrimaryKeyJoinColumn不能满足要求时，可以用PrimaryKeyJoinColumns来定义多个PrimaryKeyJoinColumn

元数据属性说明：

- value: 一个PrimaryKeyJoinColumn数组，包含所有PrimaryKeyJoinColumn。

下面的代码说明了Employee和EmployeeInfo是一对一关系。他们都使用复合主键，建表时需要在Employee表建立一个外键，从Employee的主键列id,name指向EmployeeInfo的主键列INFO_ID和INFO_NAME

```java
@Data
@IdClass(EmpPK.class)
@Entity
@Table(name = "EMPLOYEE")
public class Employee {
    private int id;
    private String name;
    private String address;

    @OneToOne(cascade = CascadeType.ALL)
    @PrimaryKeyJoinColumns({
        @PrimaryKeyJoinColumn(name="id", referencedColumnName="INFO_ID"),
        @PrimaryKeyJoinColumn(name="name" , referencedColumnName="INFO_NAME")})
    EmployeeInfo info;
}
```

```java
@Data
@IdClass(EmpPK.class)
@Entity
@Table(name = "EMPLOYEE_INFO")
public class EmployeeInfo {
    @Id
    @Column(name = "INFO_ID")
    private int id;
    @Id
    @Column(name = "INFO_NAME")
    private String name;
}
```





### UniqueConstraint

@UniqueConstraint定义在@Table或@SecondaryTable元数据里，用来指定建表时需要建唯一约束的列。

元数据属性说明：

1. name：定义的名称
2. columnNames：定义一个字符串数组，指定要建唯一约束的列名

```java
@Entity
@Table(name="tb_person",uniqueConstraints={@UniqueConstraint(columnNames={"person_id", "person_name"})})
public class Person {
    @Id
    private Long person_id;
    private String person_name;
}
```



### OrderBy

@OrderBy 在加载数据的时候可以为其指定顺序。有时我们希望从数据库加载出来的集合对象是按一定方式排序的，这可以通过OrderBy来实现，默认是按对象的主键升序排列。

元数据属性说明： 

- value：字符串类型，指定排序方式
- 格式为：`fieldName1 [ASC|DESC]，fieldName2 [ASC|DESC]，...，`（排序类型可以不指定，默认是ASC）

下面的代码说明Person和Book之间是一对多关系。集合books按照Book的age升序，name降序排列。

```java
@Table(name="tb_person")
public class Person {
    @OneToMany(targetEntity = Book.class, cascade = CascadeType.ALL, mappedBy = "person")
    @OrderBy(name = "age asc, name desc")
    private List<Book> books = new ArrayList();
    
    @OneToMany(targetEntity = Detail.class, cascade = CascadeType.ALL, mappedBy = "person")
    @OrderBy // 默认使用主键排序
    private List<Detail> details = new ArrayList();
}
```



### Version

@Version 指定实体类在乐观事务中的 version 属性。在实体类重新由EntityManager管理并且加入到乐观事务中时，保证完整性。每一个类只能有一个属性被指定为 version，version 属性应该映射到实体类的主表上。version支持（int，Integer，short，Short, long，Long，java.sql.Timestamp）类型的属性或字段。

使用起来非常方便，我们只需要在实体中添加一个字段，并添加@Version注解就可以了。加了@Version后，insert和update的SQL语句都会带上version的操作。当乐观锁更新失败的时候，会抛出异常org.springframework.orm.ObjectOptimisticLockingFailureException。我们自己进行业务处理。

```java
@Version
@Column(name = "customer_version", columnDefinition="bigint default 0")
private Long version;
```



### Lob

@Lob指定一个属性作为数据库支持的大对象类型在数据库中存储。使用LobType这个枚举来定义Lob是二进制类型还是字符类型。

~~元数据属性说明（较老的版本）：~~

- ~~fetch： 定义这个字段是 lazy loaded 还是 eagerly fetched。数据类型是FetchType枚举，默认：LAZY（lazy loaded）~~ 
- ~~type： 定义这个字段在数据库中的JDBC数据类型。数据类型是LobType枚举，默认：BLOB~~
  - ~~LobType.BLOB：二进制大对象，byte[] 或者 Serializable 类型可以指定为 BLOB~~
  - ~~LobType.CLOB：字符型大对象，char[]、Character[]或String类型可以指定为 CLOB~~


新版本中该注解中没有任何参数了。下面的代码测试：

```java
@Lob
private byte[] picture; // 默认是立即加载

@Lob
@Basic(fetch = FetchType.LAZY) // 修改成懒加载
private String description;
```

PS：Clob、Blob占用内存空间较大，一般配合@Basic(fetch=FetchType.LAZY)将其设置为延迟加载



### Enumerated

@Enumerated虽然不常用却很重要，使用此注解映射枚举字段，以String类型或者Interger类型存入数据库

注入数据库的类型有两种：

1. EnumType.ORDINAL：如果当前实体类字段为枚举类会转换成Interger类型存入数据库（默认）
2. EnumType.STRING：如果当前实体类字段为枚举类会转换成String类型存入数据库

```java
// Sex为枚举类
@Column
@Enumerated(EnumType.STRING)
private Sex sex;
```



### ElementCollection

@ElementCollection集合映射，请参考：https://mp.weixin.qq.com/s/K8ori0frTguph18nrcVPAQ



### NoRepositoryBean

**@NoRepositoryBean**：一般用作父类的repository，有这个注解，Spring不会去实例化该repository。



## 5、审计功能（Spring Data）

> 参考文献 & 鸣谢：https://blog.csdn.net/qq_28804275/article/details/84801457

@CreatedDate、@CreatedBy、@LastModifiedDate、@LastModifiedBy：表示字段为：创建时间字段（insert自动设置）、创建用户字段（insert自动设置）、最后修改时间字段（update自定设置）、最后修改用户字段（update自定设置）

@CreatedDate、@CreatedBy、@LastModifiedDate、@LastModifiedBy 注解的用法：

1. 申明实体类并在头部加注解：@EntityListeners(AuditingEntityListener.class)
2. 在实体类中属性中加上面四种注解：@CreatedDate、@LastModifiedDate、@CreatedBy、@LastModifiedBy
3. 在SpringBoot启动类中加此注解：@EnableJpaAuditing
4. 如上操作默认只能实现@CreatedDate、@LastModifiedDate的自动赋值，@CreatedBy、@LastModifiedBy自定义添加修改用户字段，需要配置转换类实现AuditorAware接口才能使它们生效

如下是操作示例：

1、在实体类上加上注解 @EntityListeners(AuditingEntityListener.class)，在相应的字段上添加对应的时间和用户注解 @CreatedDate、@LastModifiedDate、@CreatedBy、@LastModifiedBy（**PS：注意：日期类型可以用 Date 也可以是 Long**）

```java
package com.example.jpa.entity;
import lombok.Data;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import javax.persistence.*;
import java.io.Serializable;

@Data
@Entity
@Table(name = "sys_user")
@EntityListeners(AuditingEntityListener.class)
public class User implements Serializable {
    @Id
    private Long id;

    @CreatedDate
    @Column(updatable = false, nullable = false)
    private java.util.Date createTime; // 创建时间

    @LastModifiedDate
    @Column(nullable = false)
    private Long updateTime; // 更新时间

    @CreatedBy
    private String createBy; // 创建人

    @LastModifiedBy
    private String lastModifiedBy; // 最后修改人
}
```

2、配置实现AuditorAware接口，以获取用户字段需要插入的信息：

```java
package com.example.jpa.config;
import org.springframework.data.domain.AuditorAware;
import org.springframework.stereotype.Component;
import java.util.Optional;

/**
 * @ClassName: JpaAuditorAware
 * @Description:根据你需要返回的类型修改这个T，比如我需要返回的是字符串，就是String。需要注意的是，类需要加上@Component以便Spring扫描到，否则不起作用。
 */
@Component
public class JpaAuditorAware implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        String userId = null; // SecurityUtils.getCurrentUserId(); 正常项目安装项目逻辑获取用户ID
        if (userId != null) {
            return Optional.of(userId);
        } else {
            return Optional.of("admin_user");
        }
    }
}
```

3、在启动类中添加注解@EnableJpaAuditing，注意如果有多个AuditorAware接口实现类，需要在@EnableJpaAuditing内指定实现类

```java
@SpringBootApplication
@EnableJpaAuditing
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

4、启动项目和单元测试查看效果

```java
@Test
@Transactional // 在测试类对于事务提交方式默认的是回滚
@Rollback(false) // 取消自动回滚
public void save() {
    User user = new User();
    user.setId(1L);
    User save = userRepository.save(user);
    System.out.println(save);
}
```

5、查看输出日志

```verilog
Hibernate: 
    select
        user0_.id as id1_0_0_,
        user0_.create_by as create_b2_0_0_,
        user0_.create_time as create_t3_0_0_,
        user0_.last_modified_by as last_mod4_0_0_,
        user0_.update_time as update_t5_0_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
User(id=1, createTime=Sat Feb 05 17:07:15 CST 2022, updateTime=1644052035337, createBy=admin_user, lastModifiedBy=admin_user)
Hibernate: 
    insert 
    into
        sys_user
        (create_by, create_time, last_modified_by, update_time, id) 
    values
        (?, ?, ?, ?, ?)
```



## 6、关联关系注解

```
1.配置多表联系注解介绍
@OneToOne（一对一）                  一对一映射
    targetEntityClass：             指定另一方类的字节码
    cascade：                       级联操作
        CascadeType.MERGE           级联更新
        CascadeType.PERSIST         级联保存
        CascadeType.REFRESH         级联刷新
        CascadeType.REMOVE          级联删除
        CascadeType.ALL             级联上述4种操作
    fetch：                         抓取策略
      FetchType.LAZY：              延迟加载（默认）
      FetchType.EAGER：             迫切查询（多表连接查询）
    mappedBy：                      放弃外键维护
    orphanRemoval：                 是否使用孤儿删除

@OneToMany（一对多）
    targetEntityClass：             指定多方类的字节码
    cascade：                       指定要使用的级联操作
    fetch：                         指定是否采用延迟加载
    mappedBy：                      指定从表实体类中引用主表对象的名称
    orphanRemoval：                 是否使用孤儿删除

@ManyToOne（多对一）
    targetEntityClass：             指定一的一方实体类字节码
    cascade：                       指定要使用的级联操作
    fetch：                         指定是否采用延迟加载
    optional：                      关联是否可选。如果设置为false，则必须始终存在非空关系

@ManyToMany（多对多）
    targetEntityClass：             指定另一方类的字节码
    cascade：                       配置级联操作
    fetch：                         配置延迟加载和立即加载
    mappedBy：                      放弃外键维护
    
    
2：配置外键关系的注解
@JoinColumn：用于定义主键字段和外键字段的对应关系。
    name：                          指定外键字段的名称
    referencedColumnName：          指定引用主表的主键字段名称（注意是数据库字段不是实体类属性名）
    unique：                        是否唯一。默认值不唯一
    nullable：                      是否允许为空。默认值允许
    insertable：                    是否允许插入。默认值允许
    updatable：                     是否允许更新。默认值允许
    columnDefinition：              列的定义信息
    table
    foreignKey

@JoinTable（针对中间表的设置）
    name：                          配置中间表的名称
    joinColumns：                   中间表的外键字段关联当前实体类所对应表的主键字段
        @JoinColumn：
            name：                  本类的外键
            referencedColumnName：  本类与外键(表)对应的主键
    inverseJoinColumn：             中间表的外键字段关联对方表的主键字段
        @JoinColumn：               
            name：                  对方类的外键
            referencedColumnName：  对方类与外键(表)对应的主键
            
@OrderBy（也属于关联关系注解）主要用来排序外键列集合对象

3：首先确定多表之间的关系：
    一对一和一对多：
        一的一方称为主表，而多的一方称为从表，外键就要建立在从表上，它们的取值的来源主要来自主键
    多对多：
        这个时候需要建立一个中间表，中间表中最少由2个字段组成，这2个字段作为外键指向2张表的主键又组成了联合主键
```





## 7、生命周期回调方法

> 1. [Java持久性API（JPA）第7讲——实体生命周期及生命周期回调方法_seloba-CSDN博客](https://blog.csdn.net/iteye_6832/article/details/81469467)
> 2. [【原创】JPA中@PrePersist和@PreUpdate的用法_DCTANT的博客-CSDN博客_jpa prepersist](https://blog.csdn.net/DCTANT/article/details/115583053)
> 3. [@PreUpdate 和@PrePersist_sunrainamazing的博客-CSDN博客_prepersist](https://blog.csdn.net/sunrainamazing/article/details/75041402)
> 4. [JPA @PostPersist 等注解的使用场景和使用方法 - 码农教程 (manongjc.com)](http://www.manongjc.com/article/44756.html)

我们在使用JPA对数据库进行操作的时候，我们时常会出现数据库字段设置未不能为空，而我们保存的字段为null导致程序报错。这个时候我们就可以使用：@PrePersist、@PostPersist 等注解回调方法来解决问题。该注释可以应用于实体类，映射超类或回调监听器类的方法。

回调方法是附加到实体生命周期事件的用户定义方法，并且在发生这些事件时由JPA自动调用。可以发现有很多类似的注解可以使用：

1. @PrePersist：在新实体持久化之前（添加到EntityManager之前，即：保存操作之前）
2. @PostPersist：在数据库中存储新实体（在commit或flush，即：保存操作之后）
3. @PreUpdate：当一个实体被识别为被修改时EntityManager（即：更新数据之前）
4. @PostUpdate：更新数据库中的实体（在commit或flush，即：更新数据之后）
5. @PreRemove：在EntityManager中标记要删除的实体时（即：删除数据之前）
6. @PostRemove：从数据库中删除实体（在commit或flush，即：删除数据之后）
7. @PostLoad：从数据库中检索实体后（即：查询完数据后）
8. @EntityListeners：指定外部生命周期事件实现类

> 在Entity实体类中进行代码测试

1、创建User实体类Entity，在Entity实体内加回调方法时，回调方法形参可有可无（外部类定义时必须要有）

```java
package entity;
import lombok.Data;
import javax.persistence.*;

@Data
@Entity
@Table(name = "sys_user")
public class User {
    @Id
    private Long id;
    private String username;
    private String password;

    @PostLoad
    public void postLoad(){
        System.out.println("PreLoad 生命周期方法被调用！ ");
    }
    @PrePersist
    public void prePersist(){
        System.out.println("PrePersist 生命周期方法被调用！ ");
    }
    @PreUpdate
    public void preUpdate(){
        System.out.println("PreUpdate 生命周期方法被调用！ ");
    }
    @PreRemove
    public void preRemove(){
        System.out.println("PreRemove 生命周期方法被调用！ ");
    }
    @PostPersist
    public void postPersist(){
        System.out.println("PostPersist 生命周期方法被调用！ ");
    }
    @PostUpdate
    public void postUpdate(){
        System.out.println("PostUpdate 生命周期方法被调用！ ");
    }
    @PostRemove
    public void postRemove(){
        System.out.println("PostRemove 生命周期方法被调用！ ");
    }
}
```

2、进行CRUD单元测试

```java
import entity.User;
import org.junit.Test;
import javax.persistence.*;

public class TestDemo {

    @Test
    public void testSave() {
        // 1.加载配置文件创建工厂（实体管理器工厂）对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        // 2.通过实体管理器工厂获取实体管理器
        EntityManager em = factory.createEntityManager();
        // 3.获取事务对象，开启事务
        EntityTransaction tx = em.getTransaction();

        // 开启事物，保存操作
        tx.begin();
        User user = new User();
        user.setId(1L);
        user.setUsername("admin");
        user.setPassword("password");
        em.persist(user);
        tx.commit();

        // 查询操作，查询同一个数据需要先清理一级缓存，不然不会发送查询SQL
        em.refresh(user);
        User user_select = em.find(User.class, 1L);

        // 开启事物，修改操作
        tx.begin();
        user_select.setUsername("root");
        em.merge(user_select);
        tx.commit();

        // 开启事物，删除操作
        tx.begin();
        em.remove(user_select);
        tx.commit();
        // 6.释放资源
        em.close();
        factory.close();
    }
}
```

3、输出日志

```
PrePersist 生命周期方法被调用！ 
Hibernate: 
    insert 
    into
        sys_user
        (password, username, id) 
    values
        (?, ?, ?)
PostPersist 生命周期方法被调用！ 
Hibernate: 
    select
        user0_.id as id1_0_0_,
        user0_.password as password2_0_0_,
        user0_.username as username3_0_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
PreLoad 生命周期方法被调用！ 
PreUpdate 生命周期方法被调用！ 
Hibernate: 
    update
        sys_user 
    set
        password=?,
        username=? 
    where
        id=?
PostUpdate 生命周期方法被调用！ 
PreRemove 生命周期方法被调用！ 
Hibernate: 
    delete 
    from
        sys_user 
    where
        id=?
PostRemove 生命周期方法被调用！ 
```

> @EntityListeners：指定外部生命周期事件实现类，在Entity实体类中进行代码测试

1、创建User实体类（这里为了方便查看，使用static建了一个内部类，实际上登录外部新建一个类）
PS：在外部类中使用@PrePersist等注解时，必须要注意：回调方法中要有Object entity参数，不然会报错。

```java
package entity;
import lombok.Data;
import javax.persistence.*;
import java.io.Serializable;

@Data
@Entity
@Table(name = "sys_user")
@EntityListeners(User.UserListener.class)
public class User implements Serializable {
    @Id
    private Long id;
    private String username;
    private String password;

    public static class UserListener{
        @PostLoad
        public void postLoad(Object user){
            System.out.println("PreLoad 生命周期方法被调用！ ");
        }
        @PrePersist
        public void prePersist(Object user){
            System.out.println("PrePersist 生命周期方法被调用！ ");
        }
        @PreUpdate
        public void preUpdate(Object user){
            System.out.println("PreUpdate 生命周期方法被调用！ ");
        }
        @PreRemove
        public void preRemove(Object user){
            System.out.println("PreRemove 生命周期方法被调用！ ");
        }
        @PostPersist
        public void postPersist(Object user){
            System.out.println("PostPersist 生命周期方法被调用！ ");
        }
        @PostUpdate
        public void postUpdate(Object user){
            System.out.println("PostUpdate 生命周期方法被调用！ ");
        }
        @PostRemove
        public void postRemove(Object user){
            System.out.println("PostRemove 生命周期方法被调用！ ");
        }
    }
}
```

2、进行CRUD单元测试与输出日志（单元测试可以复用上面的，日志输出与上面也是一摸一样）

两种方式的区别：内部Entity时回调方法形参可有可无，但是外部实现时回调函数形参必须要有









## 8、Convert 转换器（重点）

JPA实体属性类型转换器： @Convert + AttributeConverter，通过 @Convert 注解指定自定义转换器，可用于实体属性类型与数据库字段类型之间的相互转换，便于将数据存储至数据库或从数据库读取数据。

```java
@Repeatable(Converts.class)
@Target({METHOD, FIELD, TYPE}) @Retention(RUNTIME)
public @interface Convert {
    Class converter() default void.class;
    String attributeName() default "";
    boolean disableConversion() default false; // 用于禁用自动应用或继承的转换器。如果为true，则不应指定converter
}
```

自定义转换器详解：

1. 实体类的字段添加@Converter注解，标识为转换器
2. 编写一个转换类，实现 `AttributeConverter<X, Y>` 接口，X 为实体属性类型，Y 为数据库字段类型
3. 转换类实现接口后需要重写 convertToDatabaseColumn()和convertToEntityAttribute()方法
4. Y convertToDatabaseColumn(X) 的作用：将实体属性X转化为Y存储到数据库中，即插入和更新操作时执行
5. X convertToEntityAttribute(Y) 的作用：将数据库中的字段Y转化为实体属性X，即查询操作时执行

操作示例：

1、创建Convert，需要实现AttributeConverter接口

```java
/**
 * 该转换器主要处理遇到null转为空字符串
 */
@Converter
public class CustConvert implements AttributeConverter<String, String> {
    /**
     * 将实体属性x转化为y存储到数据库中，即插入和更新操作时执行
     */
    @Override
    public String convertToDatabaseColumn(String attribute) {
        return Objects.isNull(attribute) ? "" : attribute;
    }

    /**
     * 将数据库中的字段y转化为实体属性x，即查询操作时执行
     */
    @Override
    public String convertToEntityAttribute(String dbData) {
        return Objects.isNull(dbData) ? "" : dbData;
    }
}
```

2、实体类对象使用@Convert指定转换器

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tb_convert")
public class CustConvertVO {
    @Id
    private Long id;
    @Convert(converter = CustConvert.class)
    private String username;
}
```

3、测试操作

```java
public class JpaConvertTest {
    /**
     * 运行之前，修改hibernate.hbm2ddl.auto=create
     */
    @Test
    public void testSave() {
        // 获取实体管理器工厂
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myJpa");
        // 获取实体管理器
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        // 获取事务
        EntityTransaction transaction = entityManager.getTransaction();
        // 开启事务
        transaction.begin();
        // 创建实体对象并保存
        entityManager.persist(new CustConvertVO(1L,null));
        entityManager.persist(new CustConvertVO(2L, "sam"));
        // 提交事务
        transaction.commit();
        entityManager.clear();
        System.out.println(entityManager.find(CustConvertVO.class, 1L));
        System.out.println(entityManager.find(CustConvertVO.class, 2L));
        // 释放资源
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

4、查看日志：

```verilog
Hibernate: 
    insert 
    into
        tb_convert
        (username, id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_convert
        (username, id) 
    values
        (?, ?)
Hibernate: 
    select
        custconver0_.id as id1_3_0_,
        custconver0_.username as username2_3_0_ 
    from
        tb_convert custconver0_ 
    where
        custconver0_.id=?
CustConvertVO(id=1, username=)
Hibernate: 
    select
        custconver0_.id as id1_3_0_,
        custconver0_.username as username2_3_0_ 
    from
        tb_convert custconver0_ 
    where
        custconver0_.id=?
CustConvertVO(id=2, username=sam)
```



## 9、Hibernate-Validation 校验注解

> 由于Hibernate-Validation篇幅教程过于多，后面可能会单独教程笔记，暂时请参考如下教程：
>
> 1. SpringBoot使用Validation校验参数（CSDN）：https://blog.csdn.net/justry_deng/article/details/86571671
> 2. SpringBoot使用Hibernate-Validator校验（博客园）：https://www.cnblogs.com/mr-yang-localhost/p/7812038.html
> 3. SpringBoot中在除Controller层 使用Validation的方式（博客园）：https://www.cnblogs.com/gxc6/p/11407599.html
> 4. Spring Boot 参数校验（CSDN-废物大师兄）：https://www.cnblogs.com/cjsblog/p/8946768.html
> 5. 使用Spring Validation优雅地校验参数（博客园）：https://www.cnblogs.com/zhengxl5566/p/13398546.html
> 5. SpringBoot参数校验-Validator（博客园：你在谁的风景里a)：https://www.cnblogs.com/sanye613/p/15027448.html

# Hibernate JPA 主键策略

Hibernate JPA 生成主键主要通过：@Id 和 @GeneratedValue 注解实现，其生成规则由 @GeneratedValue 设定

@GeneratedValue的源码：

```java
@Target({METHOD,FIELD})
@Retention(RUNTIME)
public @interface GeneratedValue{  
    GenerationType strategy() default AUTO;  
    String generator() default "";  
} 
```

其中GenerationType枚举属性：

```java
public enum GenerationType{  
    TABLE,  
    SEQUENCE,  
    IDENTITY,  
    AUTO 
}
```

JPA 4种主键策略分别为：AUTO、IDENTITY、SEQUENCE、TABLE。剩下都是Hibernate自己的策略，包括我们常用的 native、uuid7n、assigned、sequence

1. HAUTO：JPA自动选择合适的策略，是默认选项
2. IDENTITY：采用数据库 ID自增长的方式来自增主键字段，Oracle 不支持这种方式
3. SEQUENCE：通过序列产生主键，通过`@SequenceGenerator`注解指定序列名，MySql 不支持这种方式

4. TABLE：通过一张数据库表的形式帮助我们完成主键自增



## 1、AUTO 策略

默认的配置。如果不指定主键生成策略，默认 AUTO。设置自动主键策略，在保存对象时可以自己设置主键值，也可以不填。示例（任选其一都可以）

```java
// 方式一：如果AUTO可以不用配置@GeneratedValue，默认就是AUTO设置
@Id
private Long Id;

// 方式二：默认配置@GeneratedValue，strategy属性default AUTO
@Id  
@GeneratedValue
private Long Id;

// 方式三：完整配置
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long Id;
```

**PS 注意（在使用AUTO策略时）：**

1. 如果是MySQL数据库，一定要将数据库的主键列设置成自增长，否则使用AUTO策略的时候，会报错：`org.hibernate.exception.GenericJDBCException: Field 'id' doesn't have a default value`
2. 如果是Oracle数据库，那么会使用`hibernate_sequence`，这个名称是固定的，不能更改。



## 2、IDENTITY 策略

主键则由数据库自动维护，底层数据库必须支持自动增长（对id自增，MySQL支持，Oracle不支持）示例：

```java
@Id  
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long Id;
```

PS：设置了主键自增的话建议在保存对象是就不要设置主键Id值了，会报错。如果想手动设置值可以先注释@GeneratedValue



## 3、SEQUENCE 策略

> @SequenceGenerator 注解的使。该策略一般会 @GeneratedValue 与 @SequenceGenerator 注解同时使用

GenerationType.SEQUENCE：在某些数据库中不支持主键自增长。如`Oracle` ，其提供了一种叫做**序列（sequence）**的机制生成主键。该策略不足之处正好与`TABLE`相反，由于只有部分数据库（Oracle、PostgreSQL、DB2）支持序列对象。所以该策略一般不应用其他数据库。该策略一般与 @SequenceGenerator 注解一起使用，该注解指定了生成主键的序列，然后 JPA 会根据注解内容创建一个序列（或使用一个现有序列）如果不指定序列，则使用厂商提供的默认序列生成器：`Hibernate`默认提供序列名称为`HIBERNATE_SEQUENCE`

```java
@Id  
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "id_sequence")  
@SequenceGenerator(name="id_sequence", initialValue=8, allocationSize=1, sequenceName="ID_SEQUENCE")  
private int id;
```

@SequenceGenerator 注解的定义（用来指定序列的相关信息）

```java
@Repeatable(SequenceGenerators.class)
@Target({TYPE, METHOD, FIELD}) 
@Retention(RUNTIME)
public @interface SequenceGenerator {
    String name();                    // 序列生成器的名称，会在@GeneratedValue中进行引用
    String sequenceName() default ""; // 表示生成策略用到的数据库序列名称
    String catalog() default "";      // 指定生成序列号的表的 schema
    String schema() default "";       // 指定生成序列号的表的 schema
    int initialValue() default 1;     // 主键的初始值，默认为0
    int allocationSize() default 50;  // 主键每次增长值的大小，默认为50
}
```

| 属性           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| name           | 该属性是必须设置的属性，序列生成器的名称，会在`@GeneratedValue`中进行引用 |
| sequenceName   | 实体标识所使用的数据库序列号的名称。该属性是可选的，如果我们没有为该属性设置值，OpenJPA 框架将自动创建名为 OPENJPA_SEQUENCE的序列号。如果一个 OpenJPA 容器中管理的多个实体都选择使用序列号机制生成实体标识，而且实体类中都没有指定标识字段的sequenceName属性，那么这些实体将会共享系统提供的默认名为 OPENJPA_SEQUENCE的序列号。这可能引起实体类编号的不连续。我们可以用下面的这个简单例子说明这种情况：假设 OpenJPA 容器中存在两个实体类 Dog 和 Fish，它们的实体标识字段都是数值型，并且都选择使用序列号生成实体标识，但是实体类中并没有提供sequenceName属性值。当我们首先持久化一个 Dog 对象时，它的实体标识将会是 1，紧接着我们持久化一个 Fish 对象，它的实体标识就是 2，依次类推。 |
| initialValue   | 该属性设置所使用序列号的起始值。默认为0                      |
| allocationSize | 一些数据库的序列化机制允许预先分配序列号，比如 Oracle，这种预先分配机制可以一次性生成多个序列号，然后放在 cache 中，数据库用户获取的序列号是从序列号 cache 中获取的，这样就避免了在每一次数据库用户获取序列号的时候都要重新生成序列号。allocationSize属性设置的就是一次预先分配序列号的数目，默认情况下allocationSize属性的值是 50。 |
| schema         | 该属性设置的是生成序列号的表的 schema。该属性并不是必须设置的属性，如果开发者没有为该属性设置值，OpenJPA 容器将会默认使用当前数据库用户对应的 schema。 |
| catalog        | 该属性设置的是生成序列号的表的 catalog。该属性并不是必须设置的属性，如果开发者没有为该属性设置值，OpenJPA 容器将会使用默认当前数据库用户对应的 catalog。 |

**PS**：如果底层数据库不执行序列，会报错：`org.hibernate.MappingException: org.hibernate.dialect.MySQLDialect does not support sequences`

> SQL创建序列：
>
> ```sql
> CREATE SEQUENCE seqTest
> INCREMENT BY 1 -- 每次加几个
> START WITH 1   -- 从1开始计数
> NOMAXvalue     -- 不设置最大值
> NOCYCLE        -- 一直累加，不循环
> CACHE 10;      -- 设置缓存cache个序列，如果系统down掉了或者其它情况将会导致序列不连续，也可以设置为：NOCACHE
> 
> 
> create sequence s_config_para
>   maxvalue 4294967295 -- 设置最大值为4294967295
>   cycle; -- 设置cycle属性，当达到最大值时，不是从start with设置的值开始循环。而是从1开始循环
> ```
>
> SQL创建序列链接：https://blog.csdn.net/jiejie5945/article/details/44198283



## 4、TABLE 策略

主要使用 @TableGenerator 注解，GenerationType.TABLE ：使用一张特殊的数据库表，保存插入记录的时，需要的主键值。

有时候为了不依赖于数据库的具体实现，在不同数据库之间更好的移植，可以在数据库中新建序列表来生成主键，序列表一般包含两个字段：第一个字段引用不同的关系表（表名），第二个字段是该关系表的最大序号。这样，只需要一张序列就可以用于多张表的主键生成。 

@TableGenerator 注解的定义

```java
@Target({TYPE, METHOD, FIELD}) 
@Retention(RUNTIME)
public @interface TableGenerator {
    String name();
    String table() default "";
    String catalog() default "";
    String schema() default "";
    String pkColumnName() default "";
    String valueColumnName() default "";
    String pkColumnValue() default "";
    int initialValue() default 0;
    int allocationSize() default 50;
    UniqueConstraint[] uniqueConstraints() default {};
}
```

其中属性说明：

| 属性名           | 解释                                                         |
| ---------------- | ------------------------------------------------------------ |
| name             | 对应GeneratedValue的generator属性值。通过俩者将其互相关联    |
| table            | 对应第三方主键生成表名[代表JPA将使用哪个第三方表来主键值得计算] |
| schema           | 指定生成序列号的表的 schema。如没设置，OpenJPA 容器将会默认使用当前数据库用户对应的 schema |
| catalog          | 指定生成序列号的表的 catalog。如没设值，OpenJPA 容器将会使用默认当前数据库用户对应的 catalog |
| pkColumnName     | 指定第三方表中对应的某个列名                                 |
| valueColumnName  | 指定生成的列名[对应第三方表的另外一个列值                    |
| pkColumnValue    | 指定第三方表中对应的某个列的值，某个列指 pkColumnName属性中的指定的列名 |
| initialValue     | 默认情况下，JPA 持续性提供程序将所有主键值的起始值设置为 0   |
| allocationSize   | 分配大小，指主键增长步长。这里指定为1，则意思是主键每次增长为1 |
| UniqueConstraint | 默认值：JPA 持续性提供程序假设主键生成器表中任何列均没有唯一约束。若设置需uniqueContraints配合 |



1、先创建保存主键的数据表，并插入初始数据（与JPA方式对比，使用一种即可）

```sql
-- 创建表，插入一条信息（也可以不插入，使用JPA建表和初始化数据）
drop table if exists tb_generator;
create table tb_generator (
    pk_name varchar(50) not null,
    pk_value int(50) not null
) engine = innodb;
INSERT INTO tb_generator (pk_name, pk_value) VALUES ('table_id', 10);

-- tb_generator表初始数据
mysql> select * from tb_generator;
+----------+----------+
| pk_name  | pk_value |
+----------+----------+
| table_id |       10 |
+----------+----------+
1 row in set (0.04 sec)
```

注意：这个表可以给无数的表作为主键表，现在只插入了一数据，这一条数据只是为一个表做主键而已。需要为其他表作为主键表著需要插入一行数据即可。需要保证 table、pkColumnName、valueColumnName 三个属性值相同就可以了

2、使用 GenerationType.TABLE 主键策略：

```java
@Data
@Entity
public class TablePrimarykey {
    @Id
    @TableGenerator(
            name = "id_generator",        // 定义一个主键生成器的名称，GeneratedValue会引用
            table = "tb_generator",       // 表示表生成策略所持久化的表名（如果DB没有tb_generator表会自动创建该表，那么可以设置initialValue）
            pkColumnName = "pk_name",     // 在持久化表中该主键生成策略所对应键值的名称(列)
            pkColumnValue = "table_id",   // 主键操作的内容字段(pkColumnName列的值)
            valueColumnName = "pk_value", // 表示在持久化表中该生成策略所对应的主键(列)
            allocationSize = 10)          // 每次增长的步长
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "id_generator")
    private Long id;
    private String name;
}
```

3、测试代码：

```java
public class TablePrimaryKeyTest {

    /**
     * 运行之前，修改hibernate.hbm2ddl.auto=update
     * 多执行几次然后查看数据库中的数据，
     * 如果是刚创建表那么第一条数据是不会按照规则生成，从第二条数据开始查看
     */
    @Test
    public void testTableId(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();
        entityManager.persist(new TablePrimaryKey());// 保存数据，按照TABLE主键策略生成主键
        entityManager.getTransaction().commit();
        entityManager.close();
    }
}
```

4、查看日志：

```verilog
Hibernate: 
    select
        tbl.pk_value 
    from
        tb_generator tbl 
    where
        tbl.pk_name=? for update
            
Hibernate: 
    update
        tb_generator 
    set
        pk_value=?  
    where
        pk_value=? 
        and pk_name=?
Hibernate: 
    insert 
    into
        tb_primary_key
        (name, id) 
    values
        (?, ?)
```

5、查看数据库表信息：

```sql
mysql> use hibernate_jpa;
Database changed
mysql> select * from tb_generator;
+----+----------+----------+
| id | pk_name  | pk_value |
+----+----------+----------+
|  1 | table_id |       40 |
+----+----------+----------+
1 row in set (0.01 sec)

mysql> select * from tb_primary_key;
+----+------+
| id | name |
+----+------+
|  2 | NULL |
| 12 | NULL |
| 22 | NULL |
+----+------+
3 rows in set (0.02 sec)
```



## 5、自定义主键生成器

@GenericGenerator 注解（是 Hibernate 自定义主键生成器，可以直接引用内置主键策略） 该策略一般会 @GeneratedValue 和 @GenericGenerator 同时使用，并且@GeneratedValue注解中的”generator”属性要与@GenericGenerator注解中name属性一致，strategy属性表示hibernate的主键生成策略。
（换种方式解释：Hibernate 同时对 JPA 进行了扩展，在 @GeneratedValue 中指定 generator，然后用 @GenericGenerator 指定策略来维护主键）

```java
@Id
@GeneratedValue(generator = "myGenerator") // 使用了generator默认可以不指定strategy
@GenericGenerator(name = "myGenerator", strategy = "uuid") // 使用uuid的生成策略
private String Id;
```

@GenericGenerator 注解的定义（用来指定序列的相关信息）

```java
@Target({ElementType.PACKAGE, ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(GenericGenerators.class)
public @interface GenericGenerator {
    String name(); // @GeneratedValue 中 generator 的值
    String strategy(); // 生成器名称
    Parameter[] parameters() default {}; // 生成器的参数
}
```

1. name：指定生成器名称。会在@GeneratedValue中进行引用
2. strategy：指定具体生成器的类名
3. parameters：得到strategy指定的具体生成器所用到的参数（可选参数）

---

**通过查看 Hibernate 的源码查看内置主键策略，可以发现`strategy` 有14个内置主键策略选项：**

```java
    public DefaultIdentifierGeneratorFactory() {
        this.register("uuid2", UUIDGenerator.class);
        this.register("guid", GUIDGenerator.class);
        this.register("uuid", UUIDHexGenerator.class);
        this.register("uuid.hex", UUIDHexGenerator.class);
        this.register("assigned", Assigned.class);
        this.register("identity", IdentityGenerator.class);
        this.register("select", SelectGenerator.class);
        this.register("sequence", SequenceStyleGenerator.class);
        this.register("seqhilo", SequenceHiLoGenerator.class);
        this.register("increment", IncrementGenerator.class);
        this.register("foreign", ForeignGenerator.class);
        this.register("sequence-identity", SequenceIdentityGenerator.class);
        this.register("enhanced-sequence", SequenceStyleGenerator.class);
        this.register("enhanced-table", TableGenerator.class);
    }
```

列出几个 Hibernate 比较常用的生成策略：

1. native：对于Oracle采用Sequence方式，对于MySQL和SQL Server采用identity（主键自增），native就是将主键的生成交由数据库完成，hibernate不管
2. uuid：采用128位的uuid算法生成主键，uuid被编码为一个32位16进制数字的字符串。占用空间大（字符串类型）
3. assigned：在插入数据的时候主键由程序处理（即程序员手动指定）这是元素没有指定时的默认生成策略。等同AUTO
4. identity：使用SQL Server和MySQL自增字段，Oracle不支持主键自增，要设定Sequence（MySQL 和 SQL Server中很常用） 等于 JPA 的 INDENTITY
5. increment：插入数据的时候 Hibernate 会给主键添加一个自增的主键，但是一个 Hibernate 实例就维护一个计数器，所以多实例运行时不能使用这个

```java
@Id
@GeneratedValue(generator = "IDGenerator")
@GenericGenerator(name = "IDGenerator", strategy = "identity")
// 等价于JPA中的IDENTITY策略
@Id
@GeneratedValue(strategy=GenerationType.IDENTITY)

@Id
@GeneratedValue(generator="paymentableGenerator")
@GenericGenerator(name="paymentableGenerator", strategy="assigned")
// 等价于JPA中的AUTO策略
@Id  
@GeneratedValue(GenerationType.AUTO) 
```

```java
// uuid策略的使用，用在Oracle数据库的主键生成上，它会根据内部程序计算出32位长度的唯一id，此时保存对象可以不设置id字段的值
// 如果不用如下2个注解生成，也可以用代码手动生成UUID：`xx.setId(UUID.randomUUID().toString());
@GeneratedValue(generator = "systemUUID")
@GenericGenerator(name = "systemUUID", strategy = "uuid")
@Id
private String id; // 注意类型必须为String
```

---

> Hibernate—14个 内置主键策略详解：
>

<details>
    <summary>内置主键策略</summary>

- uuid2：
  IdentifierGenerator 的实现类是 UUIDGenerator，具体由 UUIDGenerationStrategy 策略负责生成，它有两个实现 StandardRandomStrategy 和 CustomVersionOneStrategy，他们都是使用j ava.util.UUID 的 api 生成主键的。
  StandardRandomStrategy 最终由 UUID.randomUUID(); 生成；
  CustomVersionOneStrategy 则采用版本号与位运算通过构造函数 new UUID(mostSignificantBits,leastSignificantBits); 生成。
  **特点是：不需要和数据库交互，可根据RFC4122定义的5中变量控制具体的生成策略（因为符合RFC4122定义，所以避免了警告信息）**

- guid：
  IdentifierGenerator 的实现类是 GUIDGenerator，通过 session.getFactory().getDialect().getSelectGUIDString(); 获得各个数据库中的标示字符串.
  MySQL 用 select uuid();
  Oracle 用 return "select rawtohex(sys_guid()) from dual"; 
  **特点是：需要和数据库进行一次查询才能生成。数据库全局唯一**

- uuid，uuid.hex：
  uuid和uuid.hex 两个一个东西。IdentifierGenerator的实现类是UUIDHexGenerator，通过：StringBuffer(36).append(format(getIP())).append(sep).append(format(getJVM())).append(sep).append(format(getHiTime())).append(sep).append(format(getLoTime())).append(sep).append(format(getCount()))生成。
  **特点：不需要和数据库交互，全网唯一**

- hilo：
  IdentifierGenerator 的实现类 TableHiLoGenerator，逻辑较为复杂，通过高低位酸腐生成，但是需要给定表和列作为高值的源。加上本地的地位计算所得。
  **特点：需要和数据库交互，全数据库唯一，与guid不同的是，在标识符的单个源必须被多个插入访问时可以避免拥堵。**

- assigned：
  IdentifierGenerator 的实现类 Assigned，没有生成逻辑，如果为空就抛出异常。
  **特点：不需要和数据库交互，自己管理主键生成，显示的指定id**

- identity：
  IdentityGenerator 并没有直接实现 IdentifierGenerator，而是扩展了AbstractPostInsertGenerator，并实现PostInsertIdentifierGenerator。
  而 PostInsertIdentifierGenerator 实现了 IdentifierGenerator，通过IdentifierGeneratorHelper类生成。
  这个比较特殊，它返回是个常量 "POST_INSERT_INDICATOR"，指在数据库插入后时生成，然后返回数据库生成的id；
  还有个常量 "SHORT_CIRCUIT_INDICATOR"，是用外键ForeignGenerator时使用的。
  **特点：需要和数据库交互，数据插入后返回（反查）id，同一列唯一**

- select：
  SelectGenerator 扩展了 AbstractPostInsertGenerator 实现了 Configurable 接口，而 AbstractPostInsertGenerator 实现了 PostInsertIdentifierGenerator。所以具有和identity类似的行为，有数据库触发器生成。
  **特点：需要和数据库交互**

- sequence：
  SequenceGenerator 实现了 PersistentIdentifierGenerator 接口，和 Configurable 接口。
  PersistentIdentifierGenerator 接口扩展 IdentifierGenerator 接口，通过不同的数据库，获取不同的取值语句 dialect.getSequenceNextValString(sequenceName); 然后进行查询，缓存到IntegralDataTypeHolder中，通过 generateHolder( session ).makeValue(); 获得。
  **特点：需要和数据库交互（但不是每次都是）。sequence唯一**

- seqhilo：
  seqhilo，扩展了 SequenceGenerator， 处理逻辑和 hilo 相同，值不过是使用一个具名的数据库序列来生成高值部分。
  **特点：需要和数据库交互，全数据库唯一，与guid不同的是，在标识符的单个源必须被多个插入访问时可以避免拥堵**

- increment：
  IdentifierGenerator 的实现类 IncrementGenerator，并实现了 Configurable 接口。数据库启动时查询表的最大主键列支，并通过 IntegralDataTypeHolder 缓存。插入一条，它自加一。
  **特点：仅需要首次访问数据库**

- foreign：
  IdentifierGenerator 的实现类 ForeignGenerator，通过给定的 entityName 和 propertyName 查询获得值。
  **特点：需要和数据库访问**

</details>




## 6、主键对数据库的支持

| 数据库类型 | GenerationType.AUTO | GenerationType.IDENTITY | GenerationType.SEQUENCE | GenerationType.TABLE |
| ---------- | ------------------- | ----------------------- | ----------------------- | -------------------- |
| MySQL      | 支持                | 支持                    | 不支持                  | 支持                 |
| Oracle     | 支持                | 不支持                  | 支持                    | 支持                 |
| PostgreSQL | 支持                | 支持                    | 支持                    | 支持                 |
| Kingbase   | 支持                | 支持                    | 支持                    | 支持                 |



## 7、主键UUID与数字对比

自增主键：

- 优点
  1. 数据存储空间小
  2. 查询效率高
- 缺点：
  1. 如果数据量过大，会超出自增长的值范围
  2. 分布式存储的表操作，尤其是在合并的时候操作复杂
  3. 安全性低,因为是有规律的，如果恶意扒取用户信息会很容易,如果是单据编号使用，竞争对手会容易查询出货量

UUID主键：

- 优点：
  1. 出现重复的机会少
  2. 适合大量数据的插入和更新操作，尤其是在高并发和分布式环境下
  3. 安全性较高
- 缺点：
  1. 存储空间大（16 byte）,因此它将会占用更多的磁盘空间, MySQL官方有明确的建议主键要尽量越短越好，36个字符长度的UUID不符合要求
  2. 性能降低，对MySQL索引不利：如果作为数据库主键，在InnoDB引擎下，UUID的无序性可能会引起数据位置频繁变动，严重影响性能

适用场景：

1. 项目是单机版的，并且数据量比较大（百万级）时，用自增长的,此时最好能考虑下安全性,做些安全措施
2. 项目是单机版的，并且数据量没那么大，对速度和存储要求不高时，用UUID
3. 项目是分布式的，那么首选UUID，分布式一般对速度和存储要求不高
4. 项目是分布式的，并且数据量达到千万级别可更高时，对速度和存储有要求时,可以用自增长



## 8、参考文献 & 鸣谢

1. https://blog.csdn.net/weixin_38446891/article/details/109813272
2. https://www.cnblogs.com/badtree/articles/10189769.html
2. https://blog.csdn.net/weixin_37878255/article/details/102628997



# Hibernate JPA 缓存配置

## 1、一级缓存

一级缓存指的是 EntityManager 级的缓存，对于这样的缓存几乎是一直存在的，也就是说只要用户进行JPA的操作，那么就永远都会存在有一级缓存

> 新建数据库视图类和初始化数据

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tb_cache")
public class CacheEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;
    private String name;
    private int age;
}
```

```java
   @Before
    public void initData(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();
        Arrays.asList(
                CacheEntity.builder().name("Sam").age(18).build(),
                CacheEntity.builder().name("Mike").age(58).build(),
                CacheEntity.builder().name("Hom").age(48).build(),
                CacheEntity.builder().name("Nick").age(28).build(),
                CacheEntity.builder().name("Kath").age(28).build()
        ).forEach(cacheEntity -> entityManager.persist(cacheEntity));
        entityManager.getTransaction().commit();
    }
```



1、使用 fnd() 执行两次查询，并且查询的ID信息都一样：

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindCache(){
        EntityManager entityManager = JpaUtils.getEntityManager();

        CacheEntity cacheEntity1 = entityManager.find(CacheEntity.class, 1L);
        System.err.println(cacheEntity1);

        System.err.println("--------------华丽的分割线-------------------");

        CacheEntity cacheEntity2 = entityManager.find(CacheEntity.class, 1L);
        System.err.println(cacheEntity2);
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=Sam, age=18)
--------------华丽的分割线-------------------
CacheEntity(Id=1, name=Sam, age=18) // 可以发现第二次查询的时候没有发送SQL语句
```

PS：可以发现这个时候真正执行的是一次数据库查询，也就是说在同一个`EntityManager`对象操作之中如果使用 find() 查询，那么同一条数据默认情况下只会査询一次，查询完成之后会将这个数据进行缓存。



2、如果说现在在查询过程之中发生了内容的变更呢？如：第一次查询出来的对象现在要求进行一些修改。

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindCache2(){
        EntityManager entityManager = JpaUtils.getEntityManager();

        CacheEntity cacheEntity1 = entityManager.find(CacheEntity.class, 1L);
        cacheEntity1.setName("SamXX"); // 第一次查询的数据发生改变
        System.err.println(cacheEntity1);

        System.err.println("--------------华丽的分割线-------------------");

        CacheEntity cacheEntity2 = entityManager.find(CacheEntity.class, 1L);
        System.err.println(cacheEntity2);
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=SamXX, age=18)
--------------华丽的分割线-------------------
CacheEntity(Id=1, name=SamXX, age=18) // 可以发现第二次查询的时候可以发送SQL语句
```

PS：第一个数据读取进来之后由于发生了改变，肯定和原始数据库的内容不一样了，那么此时通过第二次查询获得的结果可以发现，依然只是查询了一次，并且在ID不改变的情况下，会发生相同ID的查询依然引用已有的缓存对象.



3、如果现在想内容进行更新，需要采用刷新的模式完成。`entityManager.refresh(object)`

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindRefresh(){
        EntityManager entityManager = JpaUtils.getEntityManager();

        CacheEntity cacheEntity1 = entityManager.find(CacheEntity.class, 1L);
        cacheEntity1.setName("SamXX"); // 第一次查询的数据发生改变
        System.err.println(cacheEntity1);
        entityManager.refresh(cacheEntity1); // 重新加载，会再次发送SQL

        System.err.println("--------------华丽的分割线-------------------");

        CacheEntity cacheEntity2 = entityManager.find(CacheEntity.class, 1L);
        System.err.println(cacheEntity2);
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=SamXX, age=18)
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
--------------华丽的分割线-------------------
CacheEntity(Id=1, name=Sam, age=18)
```

此时同样操作发现重复查询了两次，主要是因为使用了`refresh()`明确表示该缓存对象需要重新加载。在`JPA`中有一个非常重要的概念：`JPA` 的对象状态，在`JPA` 里面一共有四种对象状态.



## 2、对象状态（生命周期）

> 参考文献 & 鸣谢：JPA实体的四种状态：https://www.jianshu.com/p/636954880af8

JPA 对象四种状态（老版中只有三种，没有移除态）：

1. 瞬时态对象（瞬时态：New or Transient）：尚未有id，还未和 Persistence Context 建立关联的对象
2. 持久态对象（持久态：Managed or Persistent）：有id值，与 Persistence Context 建立了关联的对象
3. 游离态对象（游离态：Datached）：有id值，没有和 Persistence Context 建立关联的对象
4. 删除的对象（移除态：Removed）：有id值，尚未和 Persistence Context 有关联，但是已经准备好从数据库中删除（确切的说在事物提交前还与 Persistence Context 有关联，事物提交后就与 Persistence Context 没有关联了）

---

1. 临时状态/瞬时状态（transient）：没有与 entityManager 发生关系，没有被持久化，不处于 entityManager 中的对象
2. 持久化状态/托管状态（persistent）：与 entityManager 发生关系，已经被持久化，加入到 entityManager 的一级缓存中的对象
3. 删除状态（removed）：调用了 entityManager.remove(obj)，对象有关联的ID，并且在 entityManager 管理下。调用remove方法后已经计划删除，事物提交后才会被真正从数据库删除
4. 游离状态（detached）：脱管状态：对象和 entityManager 解除关系

```
                                            持久状态     
                                            |     ⬆     
                                         find() flush() 
                                            ⬇     |     
New POJO() ----> 瞬时状态 ----persist()----> 托管状态 ----remove()----> 销毁状态
                                            |     ⬆ 
                                        事物提交 meger/refresh
                                            ⬇     |
                                            游离状态
```

<img src="https://upload-images.jianshu.io/upload_images/448235-c149bf356c0aaf35.png" alt="img"  />

1、示例说明：

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindAdd(){
        CacheEntity cacheEntity = new CacheEntity(); // 临时状态
        cacheEntity.setName("lsx");
        cacheEntity.setAge(16);
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();
        entityManager.persist(cacheEntity); // 持久状态
        entityManager.remove(cacheEntity); // 删除状态
        entityManager.getTransaction().commit();// 提交更新事务
        entityManager.close();
        System.err.println(cacheEntity);// 游离状态
    }
```



2、观察在持久态下的数据更新（重点）

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindEdit(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        // 持久态
        CacheEntity cacheEntity = entityManager.find(CacheEntity.class, 1L);
        entityManager.getTransaction().begin(); // 开启事务
        cacheEntity.setName("SamXX"); // 持久化数据发生改变
        entityManager.getTransaction().commit(); // 提交更新事务（发现我们并没有提交更新操作）
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: // entityManager.find(CacheEntity.class, 1L);
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
Hibernate: // 更新了持久态中的属性内容，并且提交了事务
    update
        tb_cache 
    set
        age=?,
        name=? 
    where
        Id=?
```

```sql
-- 然后查看数据库这条数据：
mysql> select * from tb_cache where id = 1;
+----+-----+-------+
| Id | age | name  |
+----+-----+-------+
|  1 |  18 | SamXX |
+----+-----+-------+
1 row in set (0.02 sec)
```

**PS：发现这条数据name发生了改变，但是我们并没有执行更新方法，可以得出结论：持久态话下的数据发生改变的话，只要在事务中并且提交了事务更新，那么默认就相当于执行了更新数据库操作**



3、一个新的对象默认情况下属于瞬时态，瞬时态也可以持久化

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindAdd2(){
        CacheEntity cacheEntity = new CacheEntity(); // 临时状态
        cacheEntity.setName("lsx");
        cacheEntity.setAge(16);

        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务
        entityManager.persist(cacheEntity);// 数据持久化，自动可以获取ID
        entityManager.getTransaction().commit();// 提交更新事务
        System.err.println("-----新增数据的ID是：" + cacheEntity.getId());

        // 将刚刚保存对象信息根据ID查询出来
        CacheEntity selectCacheEntity = entityManager.find(CacheEntity.class, cacheEntity.getId());
        System.err.println(selectCacheEntity);
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    insert 
    into
        tb_cache
        (age, name) 
    values
        (?, ?)
-----新增数据的ID是：6
CacheEntity(Id=6, name=lsx, age=16) // 可以发现查询数据没有发送SQL，而是从缓存中获取
```

**PS：由于现在使用了`persisit()`方法将瞬时态对象变为了持久态，所以这个对象就会被缓存起来，那么再执行一次查询的时候就不会重复发出查询命令，而是直接使用缓存中的数据.**



4、如果每新增一个数据都被缓存起来，当批量新增时，缓存所占用的空间就会出现严重的不足。最好的做法是进行：**强制性的保存以及清空**

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindFlushAndClear(){
        CacheEntity cacheEntity = new CacheEntity();
        cacheEntity.setName("lsx");
        cacheEntity.setAge(16);

        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();

        entityManager.persist(cacheEntity);// 数据持久化，自动可以获取ID
        entityManager.flush(); // 强制立即写入数据库
        entityManager.clear(); // 清空缓存

        entityManager.getTransaction().commit();// 提交更新事务
        System.err.println("-----新增数据的ID是：" + cacheEntity.getId());

        // 将刚刚保存对象信息根据ID查询出来
        CacheEntity selectCacheEntity = entityManager.find(CacheEntity.class, cacheEntity.getId());
        System.err.println(selectCacheEntity);
        entityManager.close();
    }
```

查看日志：

```
Hibernate: 
    insert 
    into
        tb_cache
        (age, name) 
    values
        (?, ?)
-----新增数据的ID是：6
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=6, name=lsx, age=16)
```

**PS：由于清除了缓存，所以此时的数据再次查询的时候就需要重新发出查询指令，当然重新查询之后也就意味这一个对象重新被缓存了**

所以在真实进行数据的批量增加时，我们应该适当加上强制写入和清空缓存（例如每新增10条数据后就执行一次`flush`和`clear`方法）



5、删除数据（游离状态）

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testDelete(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        CacheEntity cacheEntity = entityManager.find(CacheEntity.class, 1L);
        entityManager.getTransaction().begin();// 开启事务
        entityManager.remove(cacheEntity); // 删除数据
        cacheEntity.setName("小明");
        entityManager.getTransaction().commit();// 提交事务
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
Hibernate: 
    delete 
    from
        tb_cache 
    where
        Id=?

```

**PS：删除的数据就属于游离态了，所以此时已无法实现试据的持久态管理了**



## 3、二级缓存

一级缓存时针对与`EntityManager`的缓存处理，并且永久存在，而二级缓存指的是针对于多个`EntityManager`实现的缓存处理，但是二级缓存默认是没有开启的

1、建立两个不同的 EntityManager 进行数据查询

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFind(){
        EntityManager entityManagerA = JpaUtils.getEntityManager();
        System.err.println(entityManagerA.find(CacheEntity.class, 1L));
        entityManagerA.close();

        System.err.println("--------------华丽的分割线-------------------");

        EntityManager entityManagerB = JpaUtils.getEntityManager();
        System.err.println(entityManagerB.find(CacheEntity.class, 1L));
        entityManagerB.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=Sam, age=18)
--------------华丽的分割线-------------------
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=Sam, age=18)
```

可以发现不同的Seeson（EntityManager）查询同一条数据的时候依然发出了两次查询指令，所以此时表示JPA中没有开启二级缓存



2、二级缓存一般用第三方组件：Redis、Ehache，这里用`Ehcache`为例，pox.xml引入`Ehcache`依赖

```xml
        <!--添加Hibernate-Ehcache包,版本号与hibernate一样 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-ehcache</artifactId>
            <version>5.4.1.Final</version>
        </dependency>
        <!--Ehcache-core 包 -->
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache-core</artifactId>
            <version>2.6.11</version>
        </dependency>
```



3、在`resources`目录下新建一个**`ehcache.xml`**文件。如果在加载时未找到`/ehcache.xml`资源或出现问题，则将使用默认配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
        updateCheck="false">
   <!--
      diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
      - user.home – 用户主目录
      - user.dir – 用户当前工作目录
      - java.io.tmpdir – 默认临时文件路径
      - 或自定义一个本地磁盘路径例如：/home、./tmpdir/Tmp_EhCache
    -->
   <!-- 磁盘缓存位置 --> 
   <diskStore path="./tmpdir/Tmp_EhCache"/>
   
   <defaultCache
           eternal="false"
           maxElementsInMemory="10000"
           overflowToDisk="false"
           diskPersistent="false"
           timeToIdleSeconds="1800"
           timeToLiveSeconds="259200"
           memoryStoreEvictionPolicy="LRU"/>

   <cache
           name="cloud_user"
           eternal="false"
           maxElementsInMemory="5000"
           overflowToDisk="false"
           diskPersistent="false"
           timeToIdleSeconds="1800"
           timeToLiveSeconds="1800"
           memoryStoreEvictionPolicy="LRU"/>
   <!--
      defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
    -->
   <!--
     name:缓存名称。
     maxElementsInMemory:缓存最大数目
     maxElementsOnDisk：硬盘最大缓存个数。
     eternal:对象是否永久有效，一但设置了，timeout将不起作用。
     overflowToDisk:是否保存到磁盘，当系统当机时
     timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
     timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
     diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
     diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
     diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
     memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
     clearOnFlush：内存数量最大时是否清除。
     memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
     FIFO，first in first out，这个是大家最熟的，先进先出。
     LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
     LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
  -->
</ehcache>
```



4、在JPA配置文件`persistence.xml`中增加二级缓存配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">

    <!-- 持久化单元：持久化单元事务类型,RESOURCE_LOCAL：本地事务管理 -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <!--
            配置二级缓存时候使用的模式，可配置值有：
            - ALL：所有的实体类都被缓存
            - NONE：所有的实体类都不被缓存
            - ENABLE_SELECTIVE：标识@Cacheable(true)注解的实体类将被缓存
            - DISABLE_SELECTIVE；缓存除标识@Cacheable(false)以外的所有实体类
            - UNSPECIFIED：默认值，JPA 产品默认值将被使用
         -->
        <shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>

        <!--可选配置：配置jpa实现方的配置信息-->
        <properties>
            <!-- 数据库信息配置：数据库驱动、数据库地址、数据库账户、数据库密码 -->
            <property name="hibernate.connection.driver_class" value="com.mysql.jdbc.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:mysql://127.0.0.1:3306/hibernate_jpa"/>
            <property name="hibernate.connection.username" value="root"/>
            <property name="hibernate.connection.password" value="password"/>
            
            <!-- 配置JPA服务提供商可选参数 -->
            <property name="hibernate.show_sql" value="true" /><!-- 自动显示sql -->
            <property name="hibernate.format_sql" value="true"/><!-- 格式化sql -->
            <property name="hibernate.hbm2ddl.auto" value="update" /><!-- 自动建表：none,create,update,create-drop,validate -->

            <!-- 二级缓存相关 -->
            <!-- 开启二级缓存 -->
            <property name="hibernate.cache.use_second_level_cache" value="true"/>
            <!-- 配置二级缓存处理类 -->
            <property name="hibernate.cache.region.factory_class" value="org.hibernate.cache.ehcache.EhCacheRegionFactory"/>
            <!-- 开启查询缓存，entityManager.find查询可以不配置，如果使用JPQL或SQL查询需要开启该配置 -->
            <property name="hibernate.cache.use_query_cache" value="true"/>
            <!-- 指定缓存配置文件位置,如果默认在resources下可不配置 -->
            <property name="hibernate.cache.provider_configuration" value="classpath:ehcache.xml"/>

        </properties>
    </persistence-unit>
</persistence>
```



5、其中`ENABLE_SELECTIVE`模式为实体类上配置 `@Cacheable(true)`  的才会进行生效

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tb_cache")
@Cacheable(value = true) // 增加此注解即可，value默认为true，直接配置@Cacheable也可以
public class CacheEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;
    private String name;
    private int age;
}
```

```java
/***************另一种用法（没有特殊需求只用上面配置即可）******************/
/**
 * 如果只配置@Cacheable的话是用ehcache.xml中默认defaultCache配置，
 * 想用自定义名称的缓存配置，需要增加@Cache注解，例如想使用ehcache.xml中的<cache name="cloud_user"../> 缓存配置
 * 增加注解 @Cache(usage = CacheConcurrencyStrategy.READ_WRITE,region="指定的cache name")
 **/
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE,region="cloud_user")
@Cacheable(value = true)
@Entity
@Table(name = "tb_cache")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CacheEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;
    private String name;
    private int age;
}
```

usage提供缓存对象的事务隔离机制有这几种：NONE、READ_ONLY、 NONSTRICT_READ_WRITE、READ_WRITE、TRANSACTIONAL

1. NONE：默认什么都不做
2. READ_ONLY：只读模式，在此模式下，如果对数据进行更新操作，会有异常
3. READ_WRITE：读写模式在更新缓存的时候会把缓存里面的数据换成一个锁，其它事务如果去取相应的缓存数据，发现被锁了，直接就去数据库查询
4. NONSTRICT_READ_WRITE：不严格的读写模式则不会的缓存数据加锁
5. TRANSACTIONAL：事务模式指缓存支持事务，当事务回滚时，缓存也能回滚，只支持JTA环境。

---

Ehcache不支持transaction事务机制，但其他三种可以使用：

 1. read-only：无需修改， 那么就可以对其进行只读 缓存，注意，在此策略下，如果直接修改数据库，即使能够看到前台显示效果，但是将对象修改至cache中会报error，cache不会发生作用。另：删除记录会报错，因为不能在read-only模式的对象从cache中删除
 2. read-write：需要更新数据，那么使用读/写缓存 比较合适，前提：数据库不可以为serializable transaction isolation level（序列化事务隔离级别）
 3. nonstrice-read-write：只偶尔需要更新数据（也就是说，两个事务同时更新同一记录的情况很不常见），也不需要十分严格的事务隔离，那么比较适合使用非严格读/写缓存策略。



6、再次测试Test代码：

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFind(){
        EntityManager entityManagerA = JpaUtils.getEntityManager();
        System.err.println(entityManagerA.find(CacheEntity.class, 1L));
        entityManagerA.close();

        System.err.println("--------------华丽的分割线-------------------");

        EntityManager entityManagerB = JpaUtils.getEntityManager();
        System.err.println(entityManagerB.find(CacheEntity.class, 1L));
        entityManagerB.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_0_,
        cacheentit0_.age as age2_2_0_,
        cacheentit0_.name as name3_2_0_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=Sam, age=18)
--------------华丽的分割线-------------------
CacheEntity(Id=1, name=Sam, age=18) // 可以发现第二次查询没有发送查询语句了
```



## 3、查询缓存

在进行数据查询都是直接`EntityManager`的方法直接查询，这种方式可以帮助用户直接进行缓存处理，如果现在使用`Query`查询，则对于缓存配置就不生效了

PS：而使用`Query`查询是没有缓存的，需要设置：`query.setHint(QueryHints.HINT_CACHEABLE, true);`（且每新创建一个`Query`都要设置）

由于在二级缓存中已经配置了如下配置，我们先注释掉下面的配置或者关闭配置开始测试：

```xml
<!-- 开启查询缓存，entityManager.find查询可以不配置，如果使用JPQL或SQL查询需要开启该配置，并设置query.setHint() -->
<!--<property name="hibernate.cache.use_query_cache" value="true"/>-->
<property name="hibernate.cache.use_query_cache" value="false"/>
```

1、观察默认情况下`Query`查询处理（分别测试开启查询缓存和关闭查询缓存）

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindQuery(){
        String jpql = "from CacheEntity where Id = :Id";
        EntityManager entityManagerA = JpaUtils.getEntityManager();
        TypedQuery<CacheEntity> queryA = entityManagerA.createQuery(jpql, CacheEntity.class);
        queryA.setParameter("Id", 1L);
        System.err.println(queryA.getSingleResult());
        entityManagerA.close();

        System.err.println("--------------华丽的分割线-------------------");

        EntityManager entityManagerB = JpaUtils.getEntityManager();
        TypedQuery<CacheEntity> queryB = entityManagerB.createQuery(jpql, CacheEntity.class);
        queryB.setParameter("Id", 1L);
        System.err.println(queryB.getSingleResult());
        entityManagerB.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_,
        cacheentit0_.age as age2_2_,
        cacheentit0_.name as name3_2_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=Sam, age=18)
--------------华丽的分割线-------------------
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_,
        cacheentit0_.age as age2_2_,
        cacheentit0_.name as name3_2_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=1, name=Sam, age=18)
```

**PS：发现默认情况下即便开启或者固安必二级查询缓存，对于Query查询也是无效的，所以还是需要开启查询缓存配置（后面需要加一个配置才能生效）**



2、开启查询缓存配置（还原注释掉的配置），使用`QueryHints`操作

```java
    // 运行之前，修改hibernate.hbm2ddl.auto=create
    @Test
    public void testFindQuery2(){
        String jpql = "from CacheEntity where Id = :Id";
        EntityManager entityManagerA = JpaUtils.getEntityManager();
        TypedQuery<CacheEntity> queryA = entityManagerA.createQuery(jpql, CacheEntity.class);
        queryA.setHint(QueryHints.HINT_CACHEABLE,true);
        queryA.setParameter("Id", 2L);
        System.err.println(queryA.getSingleResult());
        entityManagerA.close();

        System.err.println("--------------华丽的分割线-------------------");

        EntityManager entityManagerB = JpaUtils.getEntityManager();
        TypedQuery<CacheEntity> queryB = entityManagerB.createQuery(jpql, CacheEntity.class);
        queryB.setHint(QueryHints.HINT_CACHEABLE,true);
        queryB.setParameter("Id", 2L);
        System.err.println(queryB.getSingleResult());
        entityManagerB.close();
    }
```

查看日志：

```verilog
Hibernate: 
    select
        cacheentit0_.Id as Id1_2_,
        cacheentit0_.age as age2_2_,
        cacheentit0_.name as name3_2_ 
    from
        tb_cache cacheentit0_ 
    where
        cacheentit0_.Id=?
CacheEntity(Id=2, name=Mike, age=58)
--------------华丽的分割线-------------------
CacheEntity(Id=2, name=Mike, age=58) // 可以发现只发送了一次查询操作
```

**PS：使用`Query`查询是没有缓存的，需要设置：`query.setHint(QueryHints.HINT_CACHEABLE, true);`（且每新创建一个`Query`都要设置）**



# Hibernate JPA 锁机制

当数据库并发访问的时候为了保证操作的一致性，那么往往会对并发数据的访问做出限制，例如：只允许一个会话处理，这样的机制就称为锁机制，而在`JPA`之中也支持锁机制的处理，而`JPA`支持两类锁：

1. 悲观锁：假设数据的访问一直都存在有并发，所以悲观锁一直都会存在，主要依靠的是数据库的锁机制
2. 乐观锁：假设不会进行并发访问（不会出现同时的数据更新处理）主要是依靠算法来实现的，设置版本号，通过版本号来判断当前的`Session`能否进行更新

在`JPA`里面专门提供有一个锁的处理模式：`javax.persistence.LockModeType;`



## 1、悲观锁：Pessimistic

悲观锁认为用户的并发访问会一直发生，并且在整个的处理之中悲观锁一定会采用锁的机制，对一个事务内的操作数据进行锁定，这样其他的事务就无法进行该数据的更新操作了。在悲观锁中定义了如下几种处理模式：

- NONE：不适用锁

- PESSIMISTIEC_READ：只要事务读实体，实体管理器就锁定实体，直到事务完成锁才会解开，当你想使用重复语义查询数据时使用这种锁模式，换句话说，当你想确保数据在连续读取期间不被修改，这种锁模式不会阻碍其他事务读取数据

- PESSIMISTIC_WRITE：只要事务更新实体，实体管理器就会锁定实体，这种锁模式强制尝试修改实体数据的事务串行化，当多个并发更新事务出现更新失败几率较高时使用这种锁模式

- PESSIMISTIC_FORCE_INCREMENT：当事务读实体时，实体管理器就锁定实体，当事务结束时会增加实体的版本属性，即使实体没有修改

  

1、使用悲观锁编写测试类

```java
   /**
     * 悲观锁
     */
    @Test
    public void testFindPessimisticLock(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin(); // 开启事务
        // 加上写入悲观锁
        entityManager.find(Customer.class, 2L, LockModeType.PESSIMISTIC_WRITE);
        entityManager.getTransaction().rollback(); // 可以回滚或提交
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.id as id1_0_0_,
        customer0_.c_address as c_addres2_0_0_,
        customer0_.c_age as c_age3_0_0_,
        customer0_.c_name as c_name4_0_0_,
        customer0_.c_phone as c_phone5_0_0_,
        customer0_.c_sex as c_sex6_0_0_ 
    from
        tb_customer customer0_ 
    where
        customer0_.id=? for update
```

可以发现查询语句最后跟上了：`for update`（代表我对这条数据进行锁定了，在我没有提交或回滚事务之前，其他线程不能对该数据进行修改）



2、模仿两个线程来进行锁处理。模拟场景：

1. 线程A查询数据，并给这条数据加上写入悲观锁，然后对数据进行修改，在事务提交之前休眠20秒
2. 线程B查询同一条数据，并给这条数据加上写入悲观锁，然后对数据进行修改，不休眠直接提交事务

线程A：`TestPessimisticA.java`

```java
public class TestPessimisticA {
    /**
     * 悲观锁
     */
    public static void main(String[] args) throws Exception {
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务
        // 加上写入悲观锁
        Customer customer = entityManager.find(Customer.class, 14L, LockModeType.PESSIMISTIC_WRITE);
        customer.setName("悲观锁修改A");
        TimeUnit.SECONDS.sleep(20);// 休眠20秒
        entityManager.getTransaction().commit();// 可以提交或回滚
        entityManager.close();
    }
```

线程B：`TestPessimisticB.java`

```java
public class TestPessimisticB {
    /**
     * 悲观锁
     */
    public static void main(String[] args) {
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务
        // 加上写入悲观锁
        Customer customer = entityManager.find(Customer.class, 14L, LockModeType.PESSIMISTIC_WRITE);
        customer.setName("悲观锁修改B");
        entityManager.getTransaction().commit();// 可以提交或回滚
        entityManager.close();
    }
```

操作步骤和结论：

1. 先执行线程A，然后里面去执行线程B
2. 会出现线程A执行完数据修改后提交事务前，休眠20秒，控制台停住了
3. 线程B虽然没有休眠，可是也依旧停住了，因为在等待线程A操作完该数据
4. 等待线程A操作完数据提交了事务，线程B也立马提交了事务
5. 最终这条数据经历了先被A修改，然后立马又被线程B修改了

查看日志（从日志上什么也看不出来，线程A和B输出日志一摸一样，需要模拟时看控制台停滞与停止状态）

```verilog
Hibernate: 
    select
        customer0_.id as id1_0_0_,
        customer0_.c_address as c_addres2_0_0_,
        customer0_.c_age as c_age3_0_0_,
        customer0_.c_name as c_name4_0_0_,
        customer0_.c_phone as c_phone5_0_0_,
        customer0_.c_sex as c_sex6_0_0_ 
    from
        c_customer customer0_ 
    where
        customer0_.id=? for update
            
Hibernate: 
    update
        tb_customer 
    set
        customer_address=?,
        customer_age=?,
        customer_name=?,
        customer_phone=?,
        customer_sex=? 
    where
        customer_id=?
```

这个必须要亲自模拟多次才能理解清楚，可以在线程B加上控制台输出



## 2、乐观锁：Optimistic

`JPA`最早的时候所提供的锁机制就是乐观锁，乐观锁：假设没有多个事务修改同一条数据的情况，而且乐观锁最大的差别就是需要对数据表上增加一个表示数据版本的编号。对于乐观锁有如下几种锁的处理模式：

1. OPTIMISTIC：它和READ锁模式相同，`JPA 2.0`仍然支持`READ`模式，但明确指出在新应用程序中推荐使用`OPTIMISTIC`
2. OPTIMISTIC_FORCE_INCREMENT：它和WRITE锁模式相同，`JPA 2.0`仍然支持`WRITE`锁模式，但明确的指出在新应用程序中推荐使用`OPTIMISTIC_FORCE_INCREMENT`



1、修改数据库脚本：增加一个版本号字段：`c_version`（如果`JPA`设置自动检测修改表的话可以忽略这一步）

```sql
alter table tb_customer add column customer_version bigint default 0;
```



2、修改实体类，增加版本号字段

```java
@Version
@Column(name = "customer_version",columnDefinition="bigint default 0")
private Long version;
```



3、采用乐观锁操作，编写程序类

```java
    /**
     * 乐观锁
     */
    @Test
    public void testFindOptimisticLock(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务
        Customer customer = entityManager.find(Customer.class, 2L, LockModeType.OPTIMISTIC_FORCE_INCREMENT);// 查询并加上写入乐观锁
        customer.setName("李四四");
        entityManager.getTransaction().commit();// 可以回滚或提交
    }
```

查看日志：

```verilog
Hibernate: 
    select
        customer0_.id as id1_0_0_,
        customer0_.c_address as c_addres2_0_0_,
        customer0_.c_age as c_age3_0_0_,
        customer0_.c_name as c_name4_0_0_,
        customer0_.c_phone as c_phone5_0_0_,
        customer0_.c_sex as c_sex6_0_0_,
        customer0_.c_version as c_versio7_0_0_ 
    from
        c_customer customer0_ 
    where
        customer0_.id=?
Hibernate: // 会发现每一次在进行更新的时候会出现版本号的修改操作
    update
        tb_customer 
    set
        customer_version=? 
    where
        customer_id=? 
        and customer_version=?
```



4、模拟多用户并发访问。模拟场景（与悲观锁场景一样）：

1. 线程A查询数据，并给这条数据加上写入乐观锁，然后对数据进行修改，在事务提交之前休眠20秒
2. 线程B查询同一条数据，并给这条数据加上写入悲观锁，然后对数据进行修改，不休眠直接提交事务

线程A：`TestOptimisticA.java`

```java
public class TestOptimisticA {
    /**
     * 乐观锁
     * @param args
     */
    public static void main(String[] args) throws Exception {
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务
        Customer customer = entityManager.find(Customer.class, 14L, LockModeType.OPTIMISTIC_FORCE_INCREMENT);// 加上写入乐观锁
        customer.setName("乐观锁修改A");
        TimeUnit.SECONDS.sleep(20);// 休眠20秒
        entityManager.getTransaction().commit();// 可以提交或回滚
        entityManager.close();
    }
```

线程B：`TestOptimisticB.java`

```java
    /**
     * 乐观锁
     * @param args
     */
    public static void main(String[] args) {
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务
        System.err.println("查询数据--------");
        Customer customer = entityManager.find(Customer.class, 14L, LockModeType.OPTIMISTIC_FORCE_INCREMENT);// 加上写入乐观锁
        customer.setName("乐观锁修改B");
        entityManager.getTransaction().commit();// 可以提交或回滚
        System.err.println("修改线程B的数据完成");
        entityManager.close();
    }
```

操作步骤和结论：

1. 先执行线程A，然后里面去执行线程B
2. 由于乐观锁不是对一条数据的锁定，等于现在第二个事务（线程B）会先实现数据的更新
3. 但是由于第一个事务先启动，所以他读取到版本号和它更新时候的版本号肯定是不同的
4. 所以在线程A提交事务的时候会报如下错误：

```
Caused by: javax.persistence.OptimisticLockException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect) : [domain.Customer#14]
....
Caused by: org.hibernate.StaleObjectStateException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect) : [domain.Customer#14]
```

乐观锁是一种基于数据算法的锁的处理机制，乐观锁特点就是在于你的项目之中不存在多个用户更新同一数据的情况。如果一直存在并发更新同一数据的话，那么一定采用悲观锁



# Hibernate JPA 关联关系

关联关系从整体上分为单向关联和双向关联：

1. **单向关联**：只需从一端访问另一端，如教师Teacher可访问学生Student，则Teacher实体需要包含类型为Student的属性
2. **双向关联**：两端均可互相访问，如教师Teacher可访问学生Student，学生Student也可访问教师Teacher，两个实体均需要包含类型为对方的属性



## 1、一对一关联映射

### 1.1、单向一对一

单向1-1：需要在控制关系的一方实体中使用注解 @OneToOne 和 @JoinColumn 标注类型为对方的属性。如`Person`端为控制关系的一方，只需要在`Person`控制方加这两个注解即可。反端既不用配置属性字段也不用配置注解

> 单向一对一是关联关系映射中最简单的一种，简单地说就是可以从关联的一方去查询另一方，却不能反向查询。我们用下面的例子来举例说明，清单 1 中的 Person 实体类和清单 2 中的 Address 类就是这种单向的一对一关系，我们可以查询一个 Person 的对应的 Address 的内容，但是我们却不能由一个 Address 的值去查询这个值对应的 Person

1. tb_person_address（外键表）：`address_id`，street，city，country，`person_id`
2. tb_person（主键表）：`person_id`，username



清单 1：单向一对一关系的拥有端（正端）

```java
@Data
@Entity
@Table(name = "tb_person")
public class Person implements Serializable {
   private static final long serialVersionUID = 1L;
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   @Column(name = "person_id")
   private Long personId;
   private String username;
   /**
    * @JoinColumn
    *     name: person_address表外键字段名（数据库字段名）
    *     referencedColumnName: person表主键字段（数据库字段名）
    */
   @OneToOne
   @JoinColumn(name = "address_id",referencedColumnName = "address_id")
   private PersonAddress personAddress;
}
```

清单 2：单向一对一关系的被拥有端（反端）

```java
@Data
@Entity
@Table(name = "tb_person_address")
public class PersonAddress implements Serializable {
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   @Column(name = "address_id")
   private Long addressId;
   private String country;
   private String city;
}
```

执行代码测试：

```java
    /**
     * 运行之前，修改hibernate.hbm2ddl.auto=create
     * 单向一对一查询
     */
    @Test
    public void testOneToOne(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        // 开启事务
        entityManager.getTransaction().begin();

        // 保存地址数据
        PersonAddress address = new PersonAddress();
        address.setCountry("中国");
        address.setCity("广州");
        entityManager.persist(address);

        // 保存用户信息
        Person person = new Person();
        person.setUsername("Sam");
        person.setPersonAddress(address);
        entityManager.persist(person);

        // 提交更新事务
        entityManager.getTransaction().commit();

        // 查询拥有端（外键表端）,先清理缓存
        entityManager.clear();
        System.out.println(entityManager.find(Person.class,person.getPersonId()));
        entityManager.close();
    }
```

查看日志1（日志分为两段，1为建表）：

```verilog
Hibernate: 
    
    create table tb_person (
       person_id bigint not null auto_increment,
        username varchar(255),
        address_id bigint,
        primary key (person_id)
    ) engine=InnoDB
Hibernate: 
    
    create table tb_person_address (
       address_id bigint not null auto_increment,
        city varchar(255),
        country varchar(255),
        primary key (address_id)
    ) engine=InnoDB
Hibernate: 
    
    alter table tb_person 
       add constraint FKcep21ttdy3yuo1f56giyatasf 
       foreign key (address_id) 
       references t_person_address (address_id)
```

查看日志2（日志分为两段，2为数据插入与查询）：

```verilog
Hibernate: // 数据插入
    insert 
    into
        t_person_address
        (city, country) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        t_person
        (address_id, username) 
    values
        (?, ?)
Hibernate: // 数据查询
    select
        person0_.person_id as person_i1_5_0_,
        person0_.address_id as address_3_5_0_,
        person0_.username as username2_5_0_,
        personaddr1_.address_id as address_1_6_1_,
        personaddr1_.city as city2_6_1_,
        personaddr1_.country as country3_6_1_ 
    from
        tb_person person0_ 
    left outer join
        tb_person_address personaddr1_ 
            on person0_.address_id=personaddr1_.address_id 
    where
        person0_.person_id=?
Person(personId=1, username=Sam, address=PersonAddress(addressId=1, country=中国, city=广州))
```



### 1.2、双向一对一

清单 3：单向一对一关系的拥有端

```java
@Data
@Entity
@Table(name = "tb_person_address")
public class PersonAddress implements Serializable {
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   @Column(name = "address_id")
   private Long addressId;
   private String country;
   private String city;

   /**
    * @OneToOne
    *    mappedBy：放弃外键维护。mappedBy 只有在双向关联的时候设置。值为对方类引用本类的属性名
    */
   @OneToOne(mappedBy = "personAddress")
   private Person person;
}
```

清单 4：双向一对一关系中的接受端

```java
@Data
@Entity
@Table(name = "tb_person")
public class Person implements Serializable {
   private static final long serialVersionUID = 1L;
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   @Column(name = "person_id")
   private Long personId;
   private String username;
   /**
    * @JoinColumn
    *     name: person_address表外键字段名（数据库字段名）
    *     referencedColumnName: person表主键字段（数据库字段名）
    */
   @OneToOne
   @JoinColumn(name = "address_id",referencedColumnName = "address_id")
   private PersonAddress personAddress;
}
```

执行代码测试：

```java
    /**
     * 运行之前，修改hibernate.hbm2ddl.auto=create
     * 双向一对一查询
     */
    @Test
    public void testOneToOne2(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        // 开启事务
        entityManager.getTransaction().begin();

        PersonAddress personAddress = new PersonAddress();
        personAddress.setCountry("中国");
        personAddress.setCity("广州");
        entityManager.persist(personAddress);

        Person person = new Person();
        person.setUsername("Sam");
        person.setPersonAddress(personAddress);
        entityManager.persist(person);

        // 提交事务
        entityManager.getTransaction().commit();

        // 查询拥有端（外键表端）先清理缓存
        entityManager.clear();
        System.out.println(entityManager.find(PersonAddress.class,personAddress.getAddressId()));
        // 查询被拥有端（主键表端）
        System.out.println(entityManager.find(Person.class,person.getPersonId()));
        entityManager.close();
    }
```

查看日志（建表语句就不重复打印了，因为是一摸一样的）：

```
// ...省略插入语句和查询语句

java.lang.StackOverflowError
	at java.lang.Long.toString(Long.java:396)
	at java.lang.Long.toString(Long.java:1032)
	at java.lang.String.valueOf(String.java:2994)
	at java.lang.StringBuilder.append(StringBuilder.java:131)
	at OneToOne.Person.toString(Person.java:8)
	at java.lang.String.valueOf(String.java:2994)
	at java.lang.StringBuilder.append(StringBuilder.java:131)
	at OneToOne.PersonAddress.toString(PersonAddress.java:8)
	at java.lang.String.valueOf(String.java:2994)
	at java.lang.StringBuilder.append(StringBuilder.java:131)
	at OneToOne.Person.toString(Person.java:8)
	at java.lang.String.valueOf(String.java:2994)
...
```

> 现在在查询步骤中会出现死循环（后面解决）



## 2、一对多关联映射

### 2.1、单向一对多

单向1-N：需要在控制关系的一方实体中使用注解 @OneToMany 和 @JoinColumn 标注类型为对方的集合属性（有两种方式：一种是只加@OneToMany、另一种是 @OneToMany + @JoinColumn）

1. tb_people（外键表）：`people_id`，name，`people_id`
2. tb_people_phone（主键表）：`phone_id`，type，phone

清单 5：单向一对多关系的拥有端（一方、主键表方、主表）

```java
@Data
@Entity
@Table(name = "tb_people")
public class People {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "people_id")
    private Long peopleId;
    private String name;

    /**
     *  @OneToMany
     *    cascade = CascadeType.ALL：级联保存、更新、删除、刷新
     *    fetch = FetchType.LAZY   ：延迟加载
     *  @JoinColumn
     *    name 指定外键列，这里注意指定的是people_id,实际上是为了外键表定义的字段。该字段在PeoplePhone类必须定义
     */
    @OneToMany
    @JoinColumn(name="people_id")
    private List<PeoplePhone> peoplePhones = new ArrayList<>();
}
```

清单 6：单向一对多关系的接收端（多方、外键表方、从表）

```java
@Data
@Entity
@Table(name = "tb_people_phone")
public class PeoplePhone {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "phone_id")
    private Long phoneId;
    private String type;
    private String phone;
}
```

测试代码：

```java
    /**
     * 单向一对多查询
     */
    @Test
    public void testOneToMany(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin(); // 开启事务

        PeoplePhone peoplePhoneA = new PeoplePhone();
        peoplePhoneA.setType("date_time");
        peoplePhoneA.setPhone("13011113333");
        PeoplePhone peoplePhoneB = new PeoplePhone();
        peoplePhoneB.setType("mobile");
        peoplePhoneB.setPhone("0208514851");
        entityManager.persist(peoplePhoneA);
        entityManager.persist(peoplePhoneB);

        People people = new People();
        people.setName("Sam");
        people.getPeoplePhones().add(peoplePhoneA);
        people.getPeoplePhones().add(peoplePhoneB);
        entityManager.persist(people);
        entityManager.getTransaction().commit(); // 提交更新事务

        // 查询拥有端（主键表端，注意：单向一对多是配置在 一方/拥有端）
        System.out.println(entityManager.find(People.class,people.getPeopleId()));
        entityManager.close();
    }
```

查看日志1：（建表语句）

```verilog
Hibernate: 
    
    create table tb_people (
       people_id bigint not null auto_increment,
        name varchar(255),
        primary key (people_id)
    ) engine=InnoDB
Hibernate: 
    
    create table tb_people_phone (
       phone_id bigint not null auto_increment,
        phone varchar(255),
        type varchar(255),
        people_id bigint,
        primary key (phone_id)
    ) engine=InnoDB
Hibernate: 
    
    alter table tb_people_phone 
       add constraint FKnied6axrmqsyl5olnjywa7set 
       foreign key (people_id) 
       references t_people (people_id)
```

查看日志2：

```verilog
Hibernate: 
    insert 
    into
        tb_people_phone
        (phone, type) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_people_phone
        (phone, type) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_people
        (name) 
    values
        (?)
Hibernate: 
    update
        tb_people_phone 
    set
        people_id=? 
    where
        phone_id=?
Hibernate: 
    update
        tb_people_phone 
    set
        people_id=? 
    where
        phone_id=?
People(peopleId=1, name=Sam, peoplePhones=[PeoplePhone(phoneId=1, type=date_time, phone=13011113333), PeoplePhone(phoneId=2, type=mobile, phone=0208514851)])
```

```sql
mysql> select * from tb_people;
+-----------+------+
| people_id | name |
+-----------+------+
|         1 | Sam  |
+-----------+------+
1 row in set (0.01 sec)

mysql> select * from tb_people_phone;
+----------+-------------+-----------+-----------+
| phone_id | phone       | type      | people_id |
+----------+-------------+-----------+-----------+
|        1 | 13011113333 | date_time |         1 |
|        2 | 0208514851  | mobile    |         1 |
+----------+-------------+-----------+-----------+
2 rows in set (0.02 sec)
```



### 2.2、单向多对一

单向N-1：需要在控制关系的一方实体中使用注解 @ManyToOne 和 @JoinColumn 标注类型为对方的属性。

清单 7：单向多对一关系的拥有端（多方、外键表方、从表）

```java
@Data
@Entity
@Table(name = "tb_people_phone")
public class PeoplePhone {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "phone_id")
    private Long phoneId;
    private String type;
    private String phone;
    /**
     * @JoinColumn
     *     name 指定外键列
     *     referencedColumnName: people 表主键字段（数据库字段名）
     */
    @ManyToOne
    @JoinColumn(name="people_id", referencedColumnName = "people_id")
    private People people;
}
```

清单 8：单向多对一关系的被拥有端（一方、主键表方、主表）

```java
@Data
@Entity
@Table(name = "tb_people")
public class People {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "people_id")
    private Long peopleId;
    private String name;
}
```

测试代码：

```java
    /**
     * 单向多对一查询
     */
    @Test
    public void testManyToOne(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务

        // 先保存主键（被维护端）数据
        People people = new People();
        people.setName("Sam");
        entityManager.persist(people);

        // 然后保存外键（维护端）数据
        PeoplePhone peoplePhoneA = new PeoplePhone();
        peoplePhoneA.setType("date_time");
        peoplePhoneA.setPhone("13011113333");
        peoplePhoneA.setPeople(people);
        PeoplePhone peoplePhoneB = new PeoplePhone();
        peoplePhoneB.setType("mobile");
        peoplePhoneB.setPhone("0208514851");
        peoplePhoneB.setPeople(people);
        entityManager.persist(peoplePhoneA);
        entityManager.persist(peoplePhoneB);

        entityManager.getTransaction().commit();// 提交更新事务

        // 查询拥有端（外键表端，注意：单向多对一是配置在多方外键拥有端）
        entityManager.clear();
        System.out.println(entityManager.find(PeoplePhone.class,peoplePhoneA.getPhoneId()));
        System.out.println(entityManager.find(PeoplePhone.class,peoplePhoneB.getPhoneId()));
        entityManager.close();
    }
```

查看日志：

```verilog
Hibernate: // 插入数据
    insert 
    into
        tb_people
        (name) 
    values
        (?)
Hibernate: // 插入数据
    insert 
    into
        tb_people_phone
        (people_id, phone, type) 
    values
        (?, ?, ?)
Hibernate: // 插入数据
    insert 
    into
        tb_people_phone
        (people_id, phone, type) 
    values
        (?, ?, ?)
Hibernate: // 查询拥有端
    select
        peoplephon0_.phone_id as phone_id1_4_0_,
        peoplephon0_.people_id as people_i4_4_0_,
        peoplephon0_.phone as phone2_4_0_,
        peoplephon0_.type as type3_4_0_,
        people1_.people_id as people_i1_3_1_,
        people1_.name as name2_3_1_ 
    from
        tb_people_phone peoplephon0_ 
    left outer join
        tb_people people1_ 
            on peoplephon0_.people_id=people1_.people_id 
    where
        peoplephon0_.phone_id=?
PeoplePhone(phoneId=1, type=date_time, phone=13011113333, people=People(peopleId=1, name=Sam))
Hibernate: // 查询拥有端
    select
        peoplephon0_.phone_id as phone_id1_4_0_,
        peoplephon0_.people_id as people_i4_4_0_,
        peoplephon0_.phone as phone2_4_0_,
        peoplephon0_.type as type3_4_0_,
        people1_.people_id as people_i1_3_1_,
        people1_.name as name2_3_1_ 
    from
        tb_people_phone peoplephon0_ 
    left outer join
        tb_people people1_ 
            on peoplephon0_.people_id=people1_.people_id 
    where
        peoplephon0_.phone_id=?
PeoplePhone(phoneId=2, type=mobile, phone=0208514851, people=People(peopleId=1, name=Sam))
```

```sql
mysql> select * from tb_people;
+-----------+------+
| people_id | name |
+-----------+------+
|         1 | Sam  |
+-----------+------+
1 row in set (0.02 sec)

mysql> select * from tb_people_phone;
+----------+-------------+-----------+-----------+
| phone_id | phone       | type      | people_id |
+----------+-------------+-----------+-----------+
|        1 | 13011113333 | date_time |         1 |
|        2 | 0208514851  | mobile    |         1 |
+----------+-------------+-----------+-----------+
```



### 2.3、双向一对多

双向1-N（N-1）：1的一端需要使用注解`@OneToMany`标注类型为对方的集合属性，同时指定`mappedBy`属性表示1的一端不控制关系，N的一端则需要使用注解@ManyToOne 和 @JoinColumn 标注类型为对方的属性。

清单 9：双向一对多关系的接受端（一方、主键表方、主表）

```java
@Data
@Entity
@Table(name = "tb_people")
public class People {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "people_id")
    private Long peopleId;
    private String name;
    /**
     * mappedBy：指定从表实体类中引用主表对象的名称。指明这端不控制关系
     * targetEntity:指定多的一方的类的字节码
     * cascade ：指定要使用的级联操作
     * fetch   ：指定是否采用延迟加载
     * orphanRemoval：是否使用孤儿删除
     */
    @OneToMany(
            mappedBy = "people",
            targetEntity = PeoplePhone.class,
            cascade = CascadeType.ALL,
            fetch = FetchType.LAZY)
    private List<PeoplePhone> peoplePhones = new ArrayList<>();
}
```

清单 10：双向一对多关系的发出端（多方、外键表方、从表）

```java
@Data
@Entity
@Table(name = "tb_people_phone")
public class PeoplePhone {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "phone_id")
    private Long phoneId;
    private String type;
    private String phone;
    /**
     * @JoinColumn
     *     name 指定外键列
     *     referencedColumnName: people 表主键字段（数据库字段名）
     */
    @ManyToOne // cascade | fetch | targetEntity 都是可选
    @JoinColumn(name="people_id", referencedColumnName = "people_id")
    private People people;
}
```

测试代码：

```java
    /**
     * 双向一对多查询
     */
    @Test
    public void testManyToOne(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        entityManager.getTransaction().begin();// 开启事务

        // 先保存主键（被维护端）数据
        People people = new People();
        people.setName("Sam");
        entityManager.persist(people);

        // 然后保存外键（维护端）数据
        PeoplePhone peoplePhoneA = new PeoplePhone();
        peoplePhoneA.setType("date_time");
        peoplePhoneA.setPhone("13011113333");
        peoplePhoneA.setPeople(people);
        PeoplePhone peoplePhoneB = new PeoplePhone();
        peoplePhoneB.setType("mobile");
        peoplePhoneB.setPhone("0208514851");
        peoplePhoneB.setPeople(people);
        entityManager.persist(peoplePhoneA);
        entityManager.persist(peoplePhoneB);

        entityManager.getTransaction().commit();// 提交更新事务

        entityManager.clear();
        // 查询被拥有端（主键表端）
        System.out.println(entityManager.find(People.class,people.getPeopleId()));
        // 查询拥有端（外键表端，注意：单向多对一是配置在多方外键拥有端）
        System.out.println(entityManager.find(PeoplePhone.class,peoplePhoneA.getPhoneId()));
        System.out.println(entityManager.find(PeoplePhone.class,peoplePhoneB.getPhoneId()));
        entityManager.close();
    }
```

查看日志（建表语句就不重复打印了，因为是一摸一样的）：

```verilog
// ...省略插入语句和查询语句

java.lang.StackOverflowError
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at OneToMany.PeoplePhone.toString(PeoplePhone.java:7)
	at java.lang.String.valueOf(String.java:2994)
	at java.lang.StringBuilder.append(StringBuilder.java:131)
	at java.util.AbstractCollection.toString(AbstractCollection.java:462)
	at org.hibernate.collection.internal.PersistentBag.toString(PersistentBag.java:538)
	at java.lang.String.valueOf(String.java:2994)
	at java.lang.StringBuilder.append(StringBuilder.java:131)
	at OneToMany.People.toString(People.java:9)
	at java.lang.String.valueOf(String.java:2994)
	at java.lang.StringBuilder.append(StringBuilder.java:131)
	at OneToMany.PeoplePhone.toString(PeoplePhone.java:7)
...
```

> 现在在查询步骤中会出现死循环（后面解决）





## 3、多对多关联映射

### 3.1、单向多对多

单向N-N：需要在控制关系的一方实体中使用注解@ManyToMany 和 @JoinTable标注类型为对方的属性，这里应该是一个集合属性

清单 11：单向多对多关系的发出端（拥有端）

```java
@Data
@Entity
@Table(name = "tb_role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_id")
    private Long roleId;
    @Column(name = "role_name")
    private String roleName;

    /**
     * @JoinTable:
     *   name：中间表名称
     *   joinColumns          中间表对应本类的信息
     *     @JoinColumn；
     *       name：本类的外键字段（中间表的数据库字段）
     *       referencedColumnName：本类与外键(表)对应的主键（本类的主键字段）
     *   inverseJoinColumns    中间表对应对方类的信息
     *     @JoinColumn：
     *       name：对方类的外键（中间表的数据字段）
     *       referencedColumnName：对方类与外键(表)对应的主键（对方类的主键字段）
     */
    @ManyToMany
    @JoinTable(name="tb_role_permission", // 中间表明
            joinColumns=@JoinColumn(
                    name="role_id", // 本类的外键
                    referencedColumnName = "role_id"), // 本类与外键(表)对应的主键
            inverseJoinColumns=@JoinColumn(
                    name="permission_id", // 对方类的外键
                    referencedColumnName = "permission_id")) // 对方类与外键(表)对应的主键
    private Set<Permission> permissions = new HashSet<>();
}
```

清单 11：单向多对多关系的接收端（被拥有端）什么都不用配置

```java
@Data
@Entity
@Table(name = "tb_permission")
public class Permission {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "permission_id")
    private Long permissionId;
    @Column(name = "permission_name")
    private String permissionName;
}
```

测试代码：

```java
    /**
     * 单向多对多
     */
    @Test
    public void testManyToMany(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        // 开启事务
        entityManager.getTransaction().begin();

        // 增加权限数据
        Permission permissionA = new Permission();
        permissionA.setPermissionName("增加");
        Permission permissionB = new Permission();
        permissionB.setPermissionName("查询");
        entityManager.persist(permissionA);
        entityManager.persist(permissionB);
        entityManager.persist(permissionB);

        // 增加角色数据
        Role role = new Role();
        role.setRoleName("网络管理员");
        role.getPermissions().add(permissionA);
        role.getPermissions().add(permissionB);
        entityManager.persist(role);

        // 提交更新事务
        entityManager.getTransaction().commit();

        // 查询角色信息
        entityManager.clear();
        System.out.println(entityManager.find(Role.class,role.getRoleId()));
        entityManager.close();
    }
```

查看日志1：（建表语句）

```verilog
Hibernate: 
    
    create table tb_permission (
       permission_id bigint not null auto_increment,
        permission_name varchar(255),
        primary key (permission_id)
    ) engine=InnoDB
Hibernate: 
    
    create table tb_role (
       role_id bigint not null auto_increment,
        role_name varchar(255),
        primary key (role_id)
    ) engine=InnoDB
Hibernate: 
    
    create table tb_role_permission (
       role_id bigint not null,
        permission_id bigint not null,
        primary key (role_id, permission_id)
    ) engine=InnoDB
Hibernate: 
    
    alter table tb_role_permission 
       add constraint FKjobmrl6dorhlfite4u34hciik 
       foreign key (permission_id) 
       references tb_permission (permission_id)
Hibernate: 
    
    alter table tb_role_permission 
       add constraint FK90j038mnbnthgkc17mqnoilu9 
       foreign key (role_id) 
       references tb_role (role_id)
```

查看日志2：（数据插入与查询）

```verilog
Hibernate: 
    insert 
    into
        tb_permission
        (permission_name) 
    values
        (?)
Hibernate: 
    insert 
    into
        tb_permission
        (permission_name) 
    values
        (?)
Hibernate: 
    insert 
    into
        tb_role
        (role_name) 
    values
        (?)
Hibernate: 
    insert 
    into
        tb_role_permission
        (role_id, permission_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_role_permission
        (role_id, permission_id) 
    values
        (?, ?)
Hibernate: 
    select
        role0_.role_id as role_id1_6_0_,
        role0_.role_name as role_nam2_6_0_ 
    from
        tb_role role0_ 
    where
        role0_.role_id=?
Hibernate: 
    select
        permission0_.role_id as role_id1_0_0_,
        permission0_.permission_id as permissi2_0_0_,
        permission1_.permission_id as permissi1_3_1_,
        permission1_.permission_name as permissi2_3_1_ 
    from
        tb_role_permission permission0_ 
    inner join
        tb_permission permission1_ 
            on permission0_.permission_id=permission1_.permission_id 
    where
        permission0_.role_id=?
Role(roleId=1, roleName=网络管理员, permissions=[Permission(permissionId=2, permissionName=查询), Permission(permissionId=1, permissionName=增加)])
```

```sql
mysql> select * from tb_role;
+---------+------------+
| role_id | role_name  |
+---------+------------+
|       1 | 网络管理员 |
+---------+------------+
1 row in set (0.05 sec)
mysql> select * from tb_permission;
+---------------+-----------------+
| permission_id | permission_name |
+---------------+-----------------+
|             1 | 增加            |
|             2 | 查询            |
+---------------+-----------------+
2 rows in set (0.06 sec)
mysql> select * from tb_role_permission;
+---------+---------------+
| role_id | permission_id |
+---------+---------------+
|       1 |             3 |
|       1 |             4 |
+---------+---------------+
2 rows in set (0.05 sec)
```

PS：单向多对多反过来配置到另一个类也是一样。只需要把本类的外键和对方类外键调换一下即可。



### 3.2、双向多对多

发出端（拥有端）代码不变

```java
@Data
@Entity
@Table(name = "t_role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_id")
    private Long roleId;
    @Column(name = "role_name")
    private String roleName;

    /**
     * @JoinTable:
     *   name：中间表名称
     *   joinColumns          中间表对应本类的信息
     *     @JoinColumn；
     *       name：本类的外键字段（中间表的数据库字段）
     *       referencedColumnName：本类与外键(表)对应的主键（本类的主键字段）
     *   inverseJoinColumns    中间表对应对方类的信息
     *     @JoinColumn：
     *       name：对方类的外键（中间表的数据字段）
     *       referencedColumnName：对方类与外键(表)对应的主键（对方类的主键字段）
     */
    @ManyToMany
    @JoinTable(name="t_role_permission", // 中间表明
            joinColumns=@JoinColumn(
                    name="role_id", // 本类的外键
                    referencedColumnName = "role_id"), // 本类与外键(表)对应的主键
            inverseJoinColumns=@JoinColumn(
                    name="permission_id", // 对方类的外键
                    referencedColumnName = "permission_id")) // 对方类与外键(表)对应的主键
    private Set<Permission> permissions = new HashSet<>();
}
```

接收端（被拥有端）代码增加了@ManyToMany 注解和`mappedBy`属性

```java
@Data
@Entity
@Table(name = "t_permission")
public class Permission {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "permission_id")
    private Long permissionId;
    @Column(name = "permission_name")
    private String permissionName;

    /**
     * @ManyToMany
     *   mappedBy:对方类应用该类的属性名，指明这端不控制关系
     *   cascade | fetch | targetEntity 为可选属性
     */
    @ManyToMany(mappedBy = "permissions")
    private Set<Role> role = new HashSet<>();
}
```

测试代码：

```java
    /**
     * 单向/双向 多对多查询代码
     */
    @Test
    public void testManyToManyFind(){
        EntityManager entityManager = JpaUtils.getEntityManager();
        System.out.println(entityManager.find(Role.class,1L));
        System.err.println("--------------华丽的分割线-------------------");
        System.out.println(entityManager.find(Permission.class,1L));
        entityManager.close();
    }
```

可以发现单向查询依旧正常，而双向查询 **依旧会有死循环问题**（该问题在SpringDataJPA篇章中解决）



