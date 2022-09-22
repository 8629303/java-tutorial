# SpringData JPA 快速入门

## 1、SpringData JPA 简介

1、Spring Data JPA 认识，官网：https://spring.io/projects/spring-data-jpa

SpringData：其实SpringData就是Spring提供了一个操作数据的框架。而SpringData JPA 只是 SpringData 框架下的一个基于JPA标准操作数据的模块，简化操作持久层的代码。只需要编写接口就可以。

Spring Data JPA：是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据库的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！

Spring Data JPA 让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现,在实际的工作工程中，推荐使用Spring Data JPA + ORM（如：Hibernate）完成操作，这样在切换不同的ORM框架时提供了极大的方便，同时也使数据库层操作更加简单，方便解耦。



2、Spring Data JPA 特性

Features：

- Sophisticated support to build repositories based on Spring and JPA
- Support for Querydsl predicates and thus type-safe JPA queries
- Transparent auditing of domain class
- Pagination support, dynamic query execution, ability to integrate custom data access code
- Validation of @Query annotated queries at bootstrap time
- Support for XML based entity mapping
- JavaConfig based repository configuration by introducing @EnableJpaRepositories.

SpringDataJPA 极大简化了数据库访问层代码。 使用SpringDataJpa，我们的DAO层中只需要写接口，就自动具有了增删改查、分页查询等方法。



3、Hibernate 与 JPA 和  Spring Data JPA 的关系

```
Java代码-----》Spring Data JPA-----》JPA 规范-----》Hibernate库-----》封装JDBC操作-----》数据库
```

1. JPA 是一套规范，内部是有接口和抽象类组成的
2. Hibernate 是一套成熟的ORM框架，而且 Hibernate 实现了JPA规范，所以也可以称 hibernate 为 JPA 的一种实现方式，我们使用 JPA 的 API 编程，意味着站在更高的角度上看待问题（面向接口编程）
3. Spring Data JPA 是 Spring 提供的一套对JPA操作更加高级的封装，是在JPA规范下的专门用来进行数据持久化的解决方案



## 2、SpringBoot 集成 JPA

1、pom.xml导入依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

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
```

2、`application.properties`配置数据源信息

```properties
# 数据库配置信息
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/springdata_jpa?useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# 控制台打印SQL
spring.jpa.show-sql=true
# 格式化SQL
spring.jpa.properties.hibernate.format_sql=true
# 自动建表模式：create-drop,create,update,validate,none
spring.jpa.hibernate.ddl-auto=update
# 建表使用MyISAM引擎，默认是InnoDB
# spring.jpa.database-platform=org.hibernate.dialect.MySQL5Dialect
```

3、新建`User`实体类

```java
@Data
@Entity
@Table(name = "sys_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
}
```

4、编写`Dao`接口

```java
// 接口内可以什么也不写
public interface UserRepository extends JpaRepository<User, Long> {
}
```

5、测试

```java
@SpringBootTest
public class JpaApplicationTests {
    @Autowired
    private UserRepository userRepository;

    /**
     * @Transactional: 开启事物（事务提交方式默认的是回滚）
     * @Rollback(false): 取消自动回滚
     * 为什么加@Rollback(false)，可以参考SpringData JPA 之疑难杂症=》SpringBoot Junit事物自动回滚
     */
    @Test
    @Transactional
    @Rollback(false)
    public void save() {
        User user = new User();
        user.setUsername("admin");
        user.setPassword("password");
        userRepository.save(user);
    }

    @Test
    public void find(){
        System.out.println(userRepository.findAll());
        System.out.println(userRepository.findById(1L));
    }
}
```

查看日志：

```verilog
Hibernate: 
    insert 
    into
        sys_user
        (age, password, username) 
    values
        (?, ?, ?)
```

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_
[User(id=1, username=admin, password=password, age=18)]
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.age as age2_1_0_,
        user0_.password as password3_1_0_,
        user0_.username as username4_1_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
User(id=1, username=admin, password=password, age=18)
```

PS：SpringBoot启动类默认可以不加@EnableJpaRepositories(basePackages="com.xxx")扫描JPA包，与SpringBoot扫描包类似。



## 3、SpringDataJPA 整合 H2

1、使用IDEA创建SpringBoot项目

2、导入依赖 pom.xml

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <!-- 可以不配置版本号，默认就是最新版本 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
            <!--<version>1.4.200</version>-->
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
```

2、`application.properties`配置

```properties
#################### H2 数据库配置 ####################
# 文件数据库
# spring.datasource.url = jdbc:h2:file:./dbh2/dbc2m;AUTO_SERVER=TRUE
# 内存数据库，platform #表明使用的数据库平台是 h2
spring.datasource.url = jdbc:h2:mem:test_jpa
spring.datasource.driverClassName = org.h2.Driver
spring.datasource.username = sa
spring.datasource.password = sa
spring.datasource.platform = h2

#################### H2 Web Console设置 ####################
# enabled：程序开启时就会启动 H2 Web Console, 默认就是启动的
# path：配置访问地址为：/h2，访问 H2 Web Console，默认:/h2-console
# settings.web-allow-others：开启H2 Web Console远程访问，默认为false不开启只能在本机访问
spring.h2.console.enabled = true
spring.h2.console.path = /h2
spring.h2.console.settings.web-allow-others = true

#################### JPA 配置 ####################
# 控制台打印SQL、格式化SQL、设置ddl模式(如果数据库是内存模式，ddl-auto配置没有什么作用)、设置方言
spring.jpa.show-sql = true
spring.jpa.properties.hibernate.format_sql = true
spring.jpa.hibernate.ddl-auto = update
spring.jpa.database-platform = org.hibernate.dialect.H2Dialect
```

3、其他启动和测试一样的





# SpringData JPA 接口查询

## 1、SpringData JPA 核心接口

1. Repository：最顶层的接口，是一个空的接口，目的是为了统一所有Repository的类型，且能让组件扫描的时候自动识别

2. CrudRepository：是Repository的子接口，提供CRUD的功能

3. PagingAndSortingRepository：是CrudRepository的子接口，添加分页和排序的功能

4. JpaRepository：是PagingAndSortingRepository的子接口，增加了一些实用的功能，比如：批量操作等

5. JpaSpecificationExecutor：用来做负责查询的接口

   Specification：是 SpringData JPA提供的一个查询规范，要做复杂的查询，只需围绕这个规范来设置查询条件

SpringData JPA接口继承结构：

```
                Repository
                    ⬇
              CrudRepository
                    ⬇
        PagingAndSortingRepository             QueryByExampleExecutor
                    ⬇                                   ⬇
               JpaRepository                       JpaRepository                JpaSpecificationExecutor
```



## 2、Repository 接口

> Repository 接口自带了如下两种查询方式：1）提供了方法名称命名查询方式、2）提供了@Query注解查询与更新

### 2.1、方法名称命名规则查询

SpringData 里面有一种很不错的操作，那就是在接口中定义方法而不用实现也不用注解就可以实现其查询操作，但是写方法名查询是有一套规范的，这里我在官方文档里面整理出来了分享给大家。【PS：方法名命名规则不能对其增加（insert）和修改（update）】

```
规则: findBy(关键字) + 属性名称(属性名称的首字母大写) + 查询条件(首字母大写)
```

常用规则速查：

| 关键字            | SQL符号           | 方法命名                    | sql where 子句             |
| ----------------- | ----------------- | --------------------------- | -------------------------- |
| And               | and               | findByNameAndPwd            | where name= ? and pwd =?   |
| Or                | or                | findByNameOrSex             | where name= ? or sex=?     |
| Is、Equals        | =                 | findById、findByIdEquals    | where id = ?               |
| Between           | between xx and xx | findByIdBetween             | where id between ? and ?   |
| LessThan          | <                 | findByIdLessThan            | where id < ?               |
| LessThanEqual     | <=                | findByIdLessThanEqual       | where id <= ?              |
| GreaterThan       | >                 | findByIdGreaterThan         | where id > ?               |
| GreaterThanEqual  | >=                | findByIdGreaterThanEqual    | where id > = ?             |
| After             | >                 | findByIdAfter               | where id > ?               |
| Before            | <                 | findByIdBefore              | where id < ?               |
| IsNull            | is null           | findByNameIsNull            | where name is null         |
| isNotNull,NotNull | is not null       | findByNameNotNull           | where name is not null     |
| Like              | like              | findByNameLike              | where name like ?          |
| NotLike           | not like          | findByNameNotLike           | where name not like ?      |
| StartingWith      | like 'xx%'        | findByNameStartingWith      | where name like '?%'       |
| EndingWith        | like '%xx'        | findByNameEndingWith        | where name like '%?'       |
| Containing        | like '%xx%'       | findByNameContaining        | where name like '%?%'      |
| OrderBy           | order by          | findByIdOrderByXDesc        | where id=? order by x desc |
| Not               | <>、!=            | findByNameNot               | where name <> ?            |
| In                | in()              | findByIdIn(Collection<?> c) | where id in (?)            |
| NotIn             | not in()          | findByNameNot               | where name <> ?            |
| True              | =true             | findByAaaTue                | where aaa = true           |
| False             | =false            | findByAaaFalse              | where aaa = false          |
| IgnoreCase        | upper(x)=upper(y) | findByNameIgnoreCase        | where UPPER(name)=UPPER(?) |
| top               | top、rownum<=x    | findTop10                   | top 10 / where ROWNUM <=10 |



### 2.2、方法名查询的具体规则

1、规则描述：

按照 Spring Data 定义的规则，查询方法以 find | read | get 开头（比如 find、findBy、read、readBy、get、getBy），涉及条件查询时，条件的属性用条件关键字连接，要注意的是：条件属性首字母需大写。框架在进行方法名解析时，会先把方法名多余的前缀截取掉，然后对剩下部分进行解析。

如果方法的最后一个参数是 Sort 或者 Pageable 类型，也会提取相关的信息，以便按规则进行排序或者分页查询。



2、举例说明：

例如：findByUserAddressZip()。框架在解析该方法时，首先剔除 findBy，然后对剩下的属性进行解析，详细规则如下（此处假设该方法针对的域对象为 AccountInfo 类型）：

1. 先判断 userAddressZip (根据POJO规范，首字母变为小写)是否为 AccountInfo 的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，继续第二步
2. 从右往左截取第一个大写字母开头的字符串（此处为 Zip），然后检查剩下的字符串是否为 AccountInfo 的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，则重复第二步，继续从右往左截取；最后假设 user 为 AccountInfo 的一个属性；
3. 接着处理剩下部分（AddressZip ）先判断 user 所对应的类型是否有 addressZip 属性，如果有，则表示该方法最终是根据 "AccountInfo.user.addressZip" 的取值进行查询；否则继续按照步骤 2 的规则从右往左截取，最终表示根据 "AccountInfo.user.address.zip" 的值进行查询。

可能会存在一种特殊情况，比如 AccountInfo 包含一个 user 的属性，也有一个 userAddress 属性，此时会存在混淆。读者可以明确在属性之间加上 "\_" 以显式表达意图，比如 "findByUser_AddressZip()" 或者 "findByUserAddress_Zip()"（**强烈建议：无论是否存在混淆，都要在不同类层级之间加上"_" ，增加代码可读性**）



### 2.3、方法名查询实战案例

> 1、当查询条件为 null 时

  举例说明如下：

1. 实体定义：对于一个客户实体 User，包含字段：username、password，均是 String 类型。
2. 查询方法定义：`List<User> findByUsernameAndPassword(String username,String password);`
3. 使用时：`dao.findByUsernameAndPassword(null, "123456");`
4. 后台生成SQL片断：`where (user0_.username is null) and user0_.password=?`
5. 结论：当查询时传值是 null 时，数据库中只有该字段是 null 的记录才符合条件，并不是说忽略这个条件。也就是说，这种查询方式，只适合于明确查询条件必须传的业务，对于动态查询，这种简单查询是不能满足要求的。

---

> 2、排序功能

```java
    /**
     * 假如你需要所有的数据而不需要按照条件查询,就要按照这样来写：
     * findBy/getBy/queryBy.. + OrderBy + 排序字段 + 排序方式
     * 按照 username 默认正序排序
     * ..order by username
     */
    List<User> findByOrderByUsername();

    /**
     * 查询全部数据：先按age字段倒序再按照id字段正序排序
     * ..order by age desc, id asc
     */
    List<User> findByOrderByAgeDescIdAsc();

    /**
     * 查询名称结果集按正序排序：
     * ...where username = ? order by age asc
     */
    List<User> findByUsernameOrderByAgeAsc(String username);

    /**
     * 查询名称结果集按倒序排序：
     * ...where username = ? order by id desc
     */
    List<User> findByUsernameOrderByIdDesc(String username);
```

---

> 3、限定查询结果集大小

```java
    /**
     * 说明：对表的全部数据根据age进行Asc（升序）排序后再选择第一条数据返回
     * SQL：...order by age asc limit 1
     * 注意：findFirst3ByOrderByAgeAsc()/findTop3ByOrderByAgeAsc()则每次返回3条 limit 3
     */
    User findFirstByOrderByAgeAsc();
    User findTopByOrderByAgeAsc();

    /**
     * 说明：首先进行数据查询并通过Sort进行排序后 再筛选数据列表中最前面的2行数据返回 limit 2
     * SQL：where username = ? order by field asc/desc
     */
    List<User> findFirst2ByUsername(String username, Sort sort);
    List<User> findTop2ByUsername(String username,Sort sort);

    /**
     * 说明：首先进行数据查询,查询全部指定的username后进行分页，分页完后进行数据控制
     * 控制说明：
     *     关于带分页的控制是，假设分页过后查询的数据id为3,4,5 查询出这三条数据后进行数据控制，
     *     本案例是控制为2条，那返回的id数据就是3,4两条记录，id为5的就舍弃，
     *     那么如果数据控制是5条(Top5)，那么就会打印3,4,5另外再去添加6,7补充数据长度
     */
    Page<User> queryFirst2ByUsername(String username, Pageable pageable);
    List<User> queryTop2ByUsername(String username, Pageable pageable);
```

总结：这里的 `First` 和 `Top` 意思相同都是最前面，`query` 也和 `find` 一样

---

> 4、计数

```java
    /**
     * 计数 返回总数
     * countBy()等同于countAllBy()
     * SQL：
     *     select count(id) from sys_user where username = ?
     *     select count(id) from sys_user
     */
    Long countByUsername(String username);
    Long countAllBy();
```

---

> 5、删除

```java
    /**
     * 说明：必须添加事务和回滚，这样根据条件找到几条就删几条
     * 对应的SQL语句：
     *  Hibernate: select * from sys_user where username = ?
     *  Hibernate: delete from sys_user where id=?
     *  Hibernate: delete from sys_user where id=?
     *  ....
     *  关于removeBy和deleteBy方法区别：
     *  他们是一样的 .选择哪一个取决于您的个人偏好.
     */
    void deleteByUsername(String username);
    void removeByUsername(String username);
```



### 2.4、使用@Query查询与更新

> @Query：通过 JPQL 语句查询与更新（JPQL：通过 Hibernate 的 HQL 演变过来的。他和 HQL 语法及其相似。）

1、创建Repository接口：

```java
/**
 * Repository接口：使用@Query注解查询JPQL
 * JPQL查询 和原生 SQL 查询，区别在于nativeQuery属性。
 * nativeQuery：默认的是false，false为JPQL查询。true为开启SQL查询。
 * 更新删除操作需要加上 @Modifying 注解（注意：不支持新增）
 */
public interface UserRepositoryJPQL extends Repository<User, Long> {

    /**
     * ? 展示位置参数绑定：?1
     * 必须使用?1形式，如果有多个参数以此类推，如: ?1,?2..
     * JPQL如果只输入？查询会报错，SQL查询不会
     */
    @Query("from User where username = ?1")
    List<User> queryUserByUsernameUseJPQL(String username);

    /**
     * : 展示名字参数绑定：:username
     * 注意：使用 :username 绑定，必须在参数前加@Param("username")
     */
    @Query("from User where username like :username")
    List<User> queryUserByLikeUsernameUseJPQL(@Param("username")String username);

    @Query("from User where username = :username and age > :age")
    List<User> queryUserByUsernameAndAgeUserJPQL(@Param("username")String username, @Param("age")Integer age);

    @Query("update User set username = ?1 where id = ?2")
    @Modifying // 需要执行一个更新操作
    Integer updateUsernameById(String username,Long id);
}
```

2、测试类：

```java
@SpringBootTest
public class UserRepositoryJPQLTests {

    @Autowired
    private UserRepositoryJPQL userRepositoryJPQL;

    /**
     * 测试@Query查询 JPQL
     */
    @Test
    public void test1(){
        List<User> listA = userRepositoryJPQL.queryUserByUsernameUseJPQL("王五");
        listA.forEach(System.out::println);
    }

    @Test
    public void test2(){
        List<User> list = userRepositoryJPQL.queryUserByLikeUsernameUseJPQL("%王%");
        list.forEach(System.out::println);
    }

    @Test
    public void test3(){
        List<User> list = userRepositoryJPQL.queryUserByUsernameAndAgeUserJPQL("隔壁小王", 15);
        list.forEach(System.out::println);
    }

    /**
     * 更新删除操作记得加上事务操作，不然会报错：
     * Caused by: javax.persistence.TransactionRequiredException: Executing an update/delete query
     */
    @Test
    @Transactional // @Transactional与@Test 一起使用时 事务是自动回滚的。
    @Rollback(false) // 取消自动回滚
    public void test4(){
        Integer integer = userRepositoryJPQL.updateUsernameById("隔壁小王", 5L);
        System.out.println("更新数据行数：" + integer);
    }

}
```

3、查看日志（更新操作）

```verilog
Hibernate: 
    update
        sys_user 
    set
        username=? 
    where
        id=?
更新数据行数：1
```

---

> @Query：通过原生 SQL 语句查询与更新

1、创建Repository接口：

```java
/**
 * Repository接口：使用@Query注解查询SQL
 * JPQL查询 和 SQL查询，区别在于nativeQuery属性。
 * nativeQuery:默认的是false.表示不开启sql查询。是否对value中的语句做转义
 * 更新删除操作需要加上 @Modifying 注解（注意：不支持新增）
 */
public interface UserRepositorySQL extends Repository<User, Long> {

    /**
     * ? 展示位置参数绑定：? 或者 ?1
     */
    @Query(value = "select * from sys_user where username = ?1",nativeQuery=true)
    List<User> queryUserByUsernameUseSQL(String username);

    /**
     * 如果有多个参数以此类推，如: ?1,?2..或者 ? ?....
     * 如果只用？的话，方法中的参数必须按顺序填写，如果 ?1 则不用
     * JPQL查询如果只输入？查询会报错，SQL查询不会
     */
    @Query(value = "select * from sys_user where username = ?1 and age >= ?2",nativeQuery=true)
    List<User> queryUserByUsernameAndAgeUseSQL(String name, Integer age);

    /**
     * : 展示名字参数绑定：:username
     * 注意：使用 :username 绑定，必须在参数前加@Param("username")
     */
    @Query(value = "select * from sys_user where username like :username",nativeQuery=true)
    List<User> queryUserByLikeUsernameUseSQL(@Param("username") String username);

    @Query(value = "update sys_user set username = ? where id = ?",nativeQuery=true)
    @Modifying // 需要执行一个更新操作
    Integer updateUsernameById(String username,Long id);
}
```

2、测试代码

```java
@SpringBootTest
public class UserRepositorySQLTests {

    @Autowired
    private UserRepositorySQL userRepositorySQL;

    /**
     * 测试@Query查询 SQL
     */
    @Test
    public void test1(){
        List<User> list = this.userRepositorySQL.queryUserByUsernameUseSQL("隔壁小王");
        list.forEach(System.out::println);
    }
    
    @Test
    public void test2(){
        List<User> list = this.userRepositorySQL.queryUserByUsernameAndAgeUseSQL("隔壁小王", 15);
        list.forEach(System.out::println);
    }

    @Test
    public void test3(){
        List<User> list = this.userRepositorySQL.queryUserByLikeUsernameUseSQL("%王%");
        list.forEach(System.out::println);
    }

    /**
     * 更新删除操作记得加上事务操作，不然会报错：
     * Caused by: javax.persistence.TransactionRequiredException: Executing an update/delete query
     */
    @Test
    @Transactional //@Transactional与@Test 一起使用时 事务是自动回滚的。
    @Rollback(false) //取消自动回滚
    public void test4(){
        Integer integer = userRepositorySQL.updateUsernameById("隔壁小王", 5L);
        System.out.println("更新数据行数：" + integer);
    }

}
```

3、查看日志

```verilog
Hibernate: 
    update
        sys_user 
    set
        username = ? 
    where
        id = ?
更新数据行数：1
```

PS：从日志上看 JPQL 与 SQL 查询看不出什么区别，不过使用 SpringData 时推荐还是使用 JPQL 方式。

---

总结：

1. 可以通过自定义的 JPQL 完成 `UPDATE` 和 `DELETE` 操作。 注意： JPQL 不支持使用 `INSERT`（新增操作）
2. 在 `@Query` 注解中编写 JPQL 语句， 如果是 `UPDATE` 或 `DELETE` 操作，必须使用 `@Modifying` 修饰
3. `UPDATE` 或 `DELETE` 操作需要使用事务，此时需要的 Service 层的方法上添加事务操作
4. 默认情况下， SpringData 的每个方法上有事务， 但都是一个只读事务。 他们不能完成修改操作





### 2.5、使用@NamedQuery查询

@NamedQuery 与 @NamedNativeQuery 都是定义查询的一种形式，@NamedQuery 使用的是JPQL，而 @NamedNativeQuery 使用的是原生SQL。这两种不常用，所以简单介绍一下。

1、使用 @NamedQuery、@NamedQueries 注解在实体类中定义命名查询（单个和多个命名查询）

```java
@Data
@Entity
@Table(name = "sys_user")
@NamedQuery(name="User.findAllUserNamedQuery",query="from User") // name指定命名查询的名称，query属性指定命令查询的语句
@NamedQueries({                                                  // 定义多个
        @NamedQuery(name="User.findUserByUsernameNamedQuery",query="from User where username = ?1"),
        @NamedQuery(name="User.findUserByLikeUsernameNamedQuery",query="from User where username like :username")
})
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private Integer age;
}
```

2、使用 @NamedNativeQuery、@NamedNativeQueries 定义单个和多个命名查询（注意使用@NamedNativeQuery需要指定resultClass参数类型）

```java
@Data
@Entity
@Table(name = "sys_user")
@NamedNativeQuery(name="User.findAllUserNamedNativeQuery",query="select * from sys_user", resultClass = User.class)
@NamedNativeQueries({
        @NamedNativeQuery(name="User.findUserByUsernameNamedNativeQuery",query="select * from sys_user where username = ?1", resultClass = User.class),
        @NamedNativeQuery(name="User.findUserByLikeUsernameNamedNativeQuery",query="select * from sys_user where username like :username", resultClass = User.class)
})
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private Integer age;
}
```

3、在Repository接口中声明方法

```java
/**
 * Repository接口：
 * 在User.Java实体类使用@NamedQuery 与 @NamedNativeQuery 都是定义查询的一种形式，
 * @NamedQuery 使用的是JPQL，而 @NamedNativeQuery 使用的是原生SQL。
 * @NamedQuery 内的 name 就是本接口定义的方法名称
 */
public interface UserRepositoryNamedQuery extends Repository<User, Long> {
    /**
     * 使用@NamedQuery进行方法查询，方法名为@NamedQuery的name参数
     */
    List<User> findAllUserNamedQuery();
    List<User> findUserByUsernameNamedQuery(String name);
    List<User> findUserByLikeUsernameNamedQuery(String username);

    /**
     * 使用@NamedNativeQuery进行方法查询，方法名为@NamedQuery的name参数
     */
    List<User> findAllUserNamedNativeQuery();
    List<User> findUserByUsernameNamedNativeQuery(String name);
    List<User> findUserByLikeUsernameNamedNativeQuery(String username);
}
```

4、测试代码

```java
package com.example.jpa;

import com.example.jpa.entity.User;
import com.example.jpa.repository.UserJpaRepository;
import com.example.jpa.repository.UserRepositoryNamedQuery;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.util.Arrays;

@SpringBootTest
public class UserRepositoryNamedQueryTests {
    @Autowired
    private UserJpaRepository userJpaRepository;
    @BeforeEach
    public void initData(){
        // 保存多条数据
        User userA = new User();
        userA.setUsername("隔壁小王");
        userA.setPassword("password");
        userA.setAge(18);
        User userB = new User();
        userB.setUsername("赵小丽");
        userB.setPassword("666666");
        userB.setAge(21);
        User userC = new User();
        userC.setUsername("王小虎");
        userC.setPassword("123456");
        userC.setAge(25);
        this.userJpaRepository.saveAll(Arrays.asList(userA, userB, userC)).forEach(System.out::println);
    }

    @Autowired
    private UserRepositoryNamedQuery userRepositoryNamedQuery;

    @Test
    public void testNamedQuery(){
        System.out.println(userRepositoryNamedQuery.findAllUserNamedQuery());
        System.out.println(userRepositoryNamedQuery.findUserByUsernameNamedQuery("隔壁小王"));
        System.out.println(userRepositoryNamedQuery.findUserByLikeUsernameNamedQuery("%王%"));
    }

    @Test
    public void testNamedNativeQuery(){
        System.out.println(userRepositoryNamedQuery.findAllUserNamedNativeQuery());
        System.out.println(userRepositoryNamedQuery.findUserByUsernameNamedNativeQuery("隔壁小王"));
        System.out.println(userRepositoryNamedQuery.findUserByLikeUsernameNamedNativeQuery("%王%"));
    }
}
```

5、查看日志

```verilog
// 省略数据初始化日志
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_
[User(id=1, username=隔壁小王, password=password, age=18), User(id=2, username=赵小丽, password=666666, age=21), User(id=3, username=王小虎, password=123456, age=25)]
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.username=?
[User(id=1, username=隔壁小王, password=password, age=18)]
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.username like ?
[User(id=1, username=隔壁小王, password=password, age=18), User(id=3, username=王小虎, password=123456, age=25)]
```

```verilog
// 省略数据初始化日志
Hibernate: 
    select
        * 
    from
        sys_user
[User(id=1, username=隔壁小王, password=password, age=18), User(id=2, username=赵小丽, password=666666, age=21), User(id=3, username=王小虎, password=123456, age=25)]
Hibernate: 
    select
        * 
    from
        sys_user 
    where
        username = ?
[User(id=1, username=隔壁小王, password=password, age=18)]
Hibernate: 
    select
        * 
    from
        sys_user 
    where
        username like ?
[User(id=1, username=隔壁小王, password=password, age=18), User(id=3, username=王小虎, password=123456, age=25)]
```

> PS：
>
> 1. @NamedQuery、@NamedNativeQuery注解也可以使用`<named-query>、<named-native-query />`标签来替代写在orm.xml中。
> 2. @NamedNativeQuery还可以与@SqlResultSetMapping（@EntityResult、@ConstructorResult、@ColumnResult、@FieldResult）注解配置使用，指定映射。
> 3. @NamedQueries，@NamedNativeQueries、@SqlResultSetMappings用装多个@NamedQuery、@NamedNativeQuery、@SqlResultSetMapping
> 4. 我们一般不推荐使用@NamedQuery、@NamedNativeQuery，而使用下面的@Query注解。





## 3、CrudRepository 接口

CrudRepository 接口主要是完成一些增删改查的操作。CrudRepository 继承了 Repository 。查看 CrudRepository 源码，该接口提供了11个常用操作方法

```java
@NoRepositoryBean  
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> { // T 是要操作的实体类，ID 是实体类主键的类型

    <S extends T> S save(S entity); // 保存
    <S extends T> Iterable<S> saveAll(Iterable<S> entities); // 批量保存
    
    Optional<T> findById(ID id); // 根据id查询一个Optional
    boolean exists(ID id); // 根据id判断对象是否存在
    Iterable<T> findAll(); // 查询所有的对象  
    Iterable<T> findAllById(Iterable<ID> ids); // 根据id列表查询所有的对象  
 
    long count(); // 计算对象的总个数  

    void delete(ID id); // 根据id删除  
    void delete(T entity); // 删除一个对象 
    void delete(Iterable<? extends T> entities); // 批量删除，集合对象（后台执行时，一条一条删除）
    void deleteAll(); // 删除所有 （后台执行时，一条一条删除）
}
```

1、创建Repository层接口，并且继承 CrudRepository 接口

```java
/**
 * CrudRepository 接口
 * 主要是完成一些增删改查的操作。注意：CrudRepository接口继承了Repository接口
 */
public interface UserCrudRepository extends CrudRepository<User,Long> {
}
```

2、测试代码

```java
package com.example.jpa;

import com.example.jpa.entity.User;
import com.example.jpa.repository.UserCrudRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import javax.transaction.Transactional;
import java.util.Arrays;
import java.util.Optional;

@SpringBootTest
public class UserCrudRepositoryTests {

    @Autowired
    private UserCrudRepository userCrudRepository;

    /**
     * 数据保存：
     *  1.添加单条数据
     *  2.批量添加数据
     */
    @Test
    public void testSave(){
        // 保存单个数据
        User user = new User();
        user.setUsername("隔壁小王");
        user.setPassword("password");
        user.setAge(18);
        User save = this.userCrudRepository.save(user);
        System.out.println(save);

        // 保存多条数据
        User userA = new User();
        userA.setUsername("赵小丽");
        userA.setPassword("666666");
        userA.setAge(21);
        User userB = new User();
        userB.setUsername("王小虎");
        userB.setPassword("123456");
        userB.setAge(25);
        Iterable<User> saveAll = this.userCrudRepository.saveAll(Arrays.asList(userA, userB));
        saveAll.forEach(System.out::println);
    }


    /**
     * 数据查询：
     *  1.根据ID查询单条数据
     *  2.查询全部数据
     *  3.根据多个id查询多条数据
     *  4.根据ID判断数据是否存在
     *  5.查询该表（实体类）有多少数据
     */
    @Test
    public void testFind(){
        // 1.根据ID查询单条数据
        Optional<User> user = this.userCrudRepository.findById(1L);
        System.out.println(user.get());
        System.err.println("--------------华丽的分割线-------------------");
        // 2.查询全部数据
        Iterable<User> userIterableA = this.userCrudRepository.findAll();
        userIterableA.forEach(System.out::println);
        System.err.println("--------------华丽的分割线-------------------");
        // 3.根据多个id查询多条数据
        Iterable<User> userIterableB = this.userCrudRepository.findAllById(Arrays.asList(1L, 2L));
        userIterableB.forEach(System.out::println);
        System.err.println("--------------华丽的分割线-------------------");
        // 4.根据ID判断数据是否存在
        System.out.println(this.userCrudRepository.existsById(1L));
        System.err.println("--------------华丽的分割线-------------------");
        // 5.查询该表（实体类）有多少数据
        System.out.println(this.userCrudRepository.count());
    }

    /**
     * 更新数据方式一: save()方法
     */
    @Test
    public void testUpdate1(){
        // 持久化状态的
        User user = this.userCrudRepository.findById(1L).get();
        user.setUsername("王小红");
        this.userCrudRepository.save(user);
    }
    /**
     * 更新数据方式二: 设置对象属性自动更新数据库
     * 【持久态对象的属性被修改后，，在数据库事物提交的时候，会更新数据库】
     */
    @Test
    @Transactional
    @Rollback(false)
    public void testUpdate2(){
        // 持久化状态，当前方法加了事物，SpringDataJPA在事物完成的时候，会自动提交修改
        User user = this.userCrudRepository.findById(1L).get();
        user.setUsername("王小小");
    }
    
    /**
     * 删除数据
     *  1.根据id删除
     *  2.根据实体类删除单个数据
     *  3.根据实体类数组删除多调数据
     *  4.删除全部数据
     */
    @Test
    public void testDelete(){
        // 1.根据id删除
        this.userCrudRepository.deleteById(1L);
        // 2.根据实体类删除单个数据
        User user = this.userCrudRepository.findById(2L).get();
        this.userCrudRepository.delete(user);
        // 3.根据实体类数组删除多调数据
        this.userCrudRepository.deleteAll(this.userCrudRepository.findAllById(Arrays.asList(1L, 2L)));
        // 4.删除全部数据
        this.userCrudRepository.deleteAll();
    }
    
}
```

查看日志：数据保存 testSave()

```verilog
Hibernate: 
    insert 
    into
        sys_user
        (age, password, username) 
    values
        (?, ?, ?)
User(id=2, username=隔壁小王, password=password, age=18)
Hibernate: 
    insert 
    into
        sys_user
        (age, password, username) 
    values
        (?, ?, ?)
Hibernate: 
    insert 
    into
        sys_user
        (age, password, username) 
    values
        (?, ?, ?)
User(id=3, username=赵小丽, password=666666, age=21)
User(id=4, username=王小虎, password=123456, age=25)
```

查看日志：数据查询 testFind()

```verilog
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.age as age2_1_0_,
        user0_.password as password3_1_0_,
        user0_.username as username4_1_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
User(id=1, username=admin, password=password, age=18)
--------------华丽的分割线-------------------
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_
User(id=1, username=admin, password=password, age=18)
User(id=2, username=隔壁小王, password=password, age=18)
User(id=3, username=赵小丽, password=666666, age=21)
User(id=4, username=王小虎, password=123456, age=25)
--------------华丽的分割线-------------------
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.id in (
            ? , ?
        )
User(id=1, username=admin, password=password, age=18)
User(id=2, username=隔壁小王, password=password, age=18)
--------------华丽的分割线-------------------
Hibernate: 
    select
        count(*) as col_0_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
true
--------------华丽的分割线-------------------
Hibernate: 
    select
        count(*) as col_0_0_ 
    from
        sys_user user0_
4
```

查看日志：数据更新方式一 testUpdate1()，可以发现查询了2次Select（第二次select是save()方法查询的）

```verilog
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.age as age2_1_0_,
        user0_.password as password3_1_0_,
        user0_.username as username4_1_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.age as age2_1_0_,
        user0_.password as password3_1_0_,
        user0_.username as username4_1_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
Hibernate: 
    update
        sys_user 
    set
        age=?,
        password=?,
        username=? 
    where
        id=?
```

查看日志：数据更新方式二 testUpdate2()，可以发现只查询了1次Select

```verilog
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.age as age2_1_0_,
        user0_.password as password3_1_0_,
        user0_.username as username4_1_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
Hibernate: 
    update
        sys_user 
    set
        age=?,
        password=?,
        username=? 
    where
        id=?
```





## 4、PagingAndSortingRepository 接口

该接口提供了两个方法，实现了分页和排序的功能了，该接口继承了 CrudRepository 接口。查看 PagingAndSortingRepository 接口源码：

```java
@NoRepositoryBean  
public interface PagingAndSortingRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {  
    Iterable<T> findAll(Sort sort); // 仅排序
    Page<T> findAll(Pageable pageable); // 分页和排序
} 
```

1、创建Repository层接口，并且继承 PagingAndSortingRepository 接口

```java
/**
 * PagingAndSortingRepository 接口：
 * 该接口提供了分页与排序的操作，注意：该接口继承了CrudRepository接口
 */
public interface UserPagingAndSortingRepository extends PagingAndSortingRepository<User,Integer> {
}
```

2、测试代码

```java
package com.example.jpa;

import com.example.jpa.entity.User;
import com.example.jpa.repository.UserPagingAndSortingRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

@SpringBootTest
public class UserPagingAndSortingTests {

    @Autowired
    private UserPagingAndSortingRepository userPagingAndSortingRepository;

    /**
     * 分页操作
     */
    @Test
    public void test1(){
        /**
         * Pageable：封装了分页的参数，当前页，每页显示的条数。注意：当前页是从0开始
         * PageRequest(page,size)：page表示当前页，size表示每页显示多少条
         */
        Pageable pageable = PageRequest.of(0, 2);
        Page<User> page = this.userPagingAndSortingRepository.findAll(pageable);
        System.out.println("总记录数: " + page.getTotalElements());
        System.out.println("当前第几页: " + (page.getNumber() + 1));
        System.out.println("总页数: " + page.getTotalPages());
        System.out.println("当前页面的 List: " + page.getContent());
        System.out.println("当前页面的记录数: " + page.getNumberOfElements());
        page.getContent().forEach(System.out::println);
    }

    /**
     * 排序操作：
     *  1.对单列做排序处理
     *  2.多列的排序处理
     */
    @Test
    public void test2(){
        /**
         * Sort：该对象封装了排序规则以及指定的排序字段(对象的属性来表示)
         *   direction：排序规则（Sort.Direction.DESC）
         *   properties：指定做排序的属性（id）
         */
        // 1.对单列做排序处理，对id做降序排序
        Sort sortOne = Sort.by(Sort.Direction.DESC,"id");
        Iterable iterable = this.userPagingAndSortingRepository.findAll(sortOne);
        iterable.forEach(System.out::println);

        System.err.println("--------------华丽的分割线-------------------");

        /**
         * 可以用如下两种方式得到Order
         *   1.new Sort.Order()
         *   2.Sort.Order.desc()/asc()/by()
         */
        // 2.多列的排序处理，先对age列做降序处理，然后对id字段做升序处理
        Sort.Order age = Sort.Order.desc("age");
        Sort.Order id = new Sort.Order(Sort.Direction.ASC,"id");
        Sort sortMany = Sort.by(age,id);
        this.userPagingAndSortingRepository.findAll(sortMany).forEach(System.out::println);
    }

    /**
     * 分页 + 排序 处理
     *
     */
    @Test
    public void test3(){
        // 排序操作对象
        Sort.Order age = Sort.Order.desc("age");
        Sort.Order id = Sort.Order.asc("id");
        Sort sort = Sort.by(age,id);
        // 排序操作还是没变化，只需要在PageRequest.of内多加一个Sort参数
        Pageable pageable = PageRequest.of(0, 2 ,sort);
        Page<User> page = this.userPagingAndSortingRepository.findAll(pageable);
        System.out.println("总记录数: " + page.getTotalElements());
        System.out.println("当前第几页: " + (page.getNumber() + 1));
        System.out.println("总页数: " + page.getTotalPages());
        System.out.println("当前页面的记录数: " + page.getNumberOfElements());
        System.out.println("当前页面的 List: " + page.getContent());
        System.err.println("--------------华丽的分割线-------------------");
        page.getContent().forEach(System.out::println);
    }
}
```

查看日志：分页操作 test1()

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ limit ?
Hibernate: 
    select
        count(user0_.id) as col_0_0_ 
    from
        sys_user user0_
总记录数: 4
当前第几页: 1
总页数: 2
当前页面的 List: [User(id=1, username=王小小, password=password, age=18), User(id=2, username=隔壁小王, password=password, age=18)]
当前页面的记录数: 2
User(id=1, username=王小小, password=password, age=18)
User(id=2, username=隔壁小王, password=password, age=18)
```

查看日志：排序操作 test2()

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    order by
        user0_.id desc
User(id=4, username=王小虎, password=123456, age=25)
User(id=3, username=赵小丽, password=666666, age=21)
User(id=2, username=隔壁小王, password=password, age=18)
User(id=1, username=王小小, password=password, age=18)
--------------华丽的分割线-------------------
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    order by
        user0_.age desc,
        user0_.id asc
User(id=4, username=王小虎, password=123456, age=25)
User(id=3, username=赵小丽, password=666666, age=21)
User(id=1, username=王小小, password=password, age=18)
User(id=2, username=隔壁小王, password=password, age=18)
```

查看日志：分页 + 排序操作 test3()

```verilog
Hibernate: 
    select
        user0_.id as id1_0_,
        user0_.age as age2_0_,
        user0_.password as password3_0_,
        user0_.username as username4_0_ 
    from
        sys_user user0_ 
    order by
        user0_.age desc,
        user0_.id asc limit ?
Hibernate: 
    select
        count(user0_.id) as col_0_0_ 
    from
        sys_user user0_
总记录数: 3
当前第几页: 1
总页数: 2
当前页面的记录数: 2
当前页面的 List: [User(id=6, username=王小虎, password=123456, age=25), User(id=5, username=赵小丽, password=666666, age=21)]
--------------华丽的分割线-------------------
User(id=6, username=王小虎, password=123456, age=25)
User(id=5, username=赵小丽, password=666666, age=21)
```



## 5、JpaRepository 接口

该接口继承了PagingAndSortingRepository。是我们开发时使用的最多的接口。其特点是可以帮助我们将其他接口的方法的返回值做适配处理。可以使得我们在开发时更方便的使用这些方法。同时也继承QueryByExampleExecutor接口，这是个用“实例”进行查询的接口，后续再写文章详细说明。

查看 JpaRepository  接口源码（这个接口的几个方法跟之前的接口内的方法虽然一样，但是最返回值做了自适应）：

```java
@NoRepositoryBean
public interface JpaRepository<T, ID extends Serializable>
        extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    
    @Override
    List<T> findAll(); // 查询所有对象，返回List
    @Override
    List<T> findAll(Sort sort); // 查询所有对象，并排序，返回List
    @Override
    List<T> findAllById(Iterable<ID> ids); // 根据id列表 查询所有的对象，返回List

    @Override
    <S extends T> List<S> saveAll(Iterable<S> entities); // 批量保存，并返回对象List
    <S extends T> S saveAndFlush(S entity); // 保存并强制同步数据库
    void flush(); // 强制缓存与数据库同步 

    void deleteInBatch(Iterable<T> entities); // 批量删除 集合对象（后台执行时，生成一条语句执行，用多个or条件）
    void deleteAllInBatch(); // 删除所有 （执行一条语句，如：delete from user）

    T getOne(ID id); // 根据id查询对象，返回对象的引用（区别于findOne）。当对象不存时，返回引用不是null，但各个属性值是null
    
    @Override
    <S extends T> List<S> findAll(Example<S> example); // 根据实例查询
    @Override
    <S extends T> List<S> findAll(Example<S> example, Sort sort);// 根据实例查询并排序
}
```

1、创建Repository层接口，并且继承 JpaRepository 接口

```java
/**
 * JpaRepository 接口
 * 该接口继承了PagingAndSortingRepository。是我们开发时使用的最多的接口。
 * 接口其特点是可以帮助我们将其他接口的方法的返回值做适配处理。可以使得我们在开发时更方便的使用这些方法。
 */
public interface UserJpaRepository extends JpaRepository<User,Long> {
}
```

2、测试代码

```java
package com.example.jpa;

import com.example.jpa.entity.User;
import com.example.jpa.repository.UserJpaRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.Sort;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

@SpringBootTest
public class UserJpaRepositoryTests {

    @Autowired
    private UserJpaRepository userJpaRepository;

    /**
     * 这几个方法跟之前的接口内的方法虽然一样，但是最返回值做了自适应
     * 返回值都由 Iterable 变成了 List
     */
    @Test
    public void test1(){
        List<User> usersA = this.userJpaRepository.findAll();
        usersA.forEach(System.out::println);
        System.err.println("--------------华丽的分割线-------------------");
        List<User> usersB = this.userJpaRepository.findAll(Sort.by("age"));
        usersB.forEach(System.out::println);
        System.err.println("--------------华丽的分割线-------------------");
        List<User> usersC = this.userJpaRepository.findAllById(Arrays.asList(1L, 2L));
        usersC.forEach(System.out::println);

        System.err.println("--------------华丽的分割线-------------------");
        User user = new User();
        user.setUsername("隔壁小王");
        user.setPassword("password");
        user.setAge(18);
        List<User> users = this.userJpaRepository.saveAll(Arrays.asList(user));
        System.out.println(users);
    }

    /**
     * JpaRepository 接口内还新增了一方法
     * flush()：强制写入数据库
     * saveAndFlush：保存并且立马写入数据库（不需要等事务走完就写入数据库）
     */
    @Test
    public void test2(){
        User user = new User();
        user.setUsername("隔壁小王");
        user.setPassword("password");
        user.setAge(18);
        // 立马写入数据库
        User saveAndFlush = this.userJpaRepository.saveAndFlush(user);
        System.out.println(saveAndFlush);
    }

    /**
     * 批量删除方法：deleteInBatch(Iterable<T> var1) 和 deleteAllInBatch()
     * 之前CrudRepository 接口内也有删除方法：delete、deleteById、deleteAll
     * 对比：deleteAll() 和 deleteAllInBatch() 区别：
     *     deleteAll()是删除全部,先findALL查找出来,再一条一条删除,最后提交事务
     *     deleteAllInBatch()是删除全部,一条sql。显然：deleteAllInBatch()方法效率更高
     *
     * 分别执行批量删除方法，查看日志对比.可以得出结论：
     *     在进行批量删除操作时尽量使用JpaRepository自带的批量删除方法deleteInBatch()及deleteAllInBatch()
     *
     * 可参考：https://jingyan.baidu.com/article/597a06431601fb312b52431c.html
     */
    @Test
    public void test3(){
        // 不推荐使用
        this.userJpaRepository.deleteAll();
    }
    @Test
    public void test4(){
        // 推荐使用
        this.userJpaRepository.deleteAllInBatch();
    }

    /**
     * getOne()与findById()方法总结与区别：
     *  1、getOne() 是延迟加载。findById() 是立马加载
     *  2、getOne()返回一个实体的引用，无结果会抛出异常。findById()返回一个Optional对象
     *  3、getOne()属于JpaRepository，底层调用getReference()方法，懒加载的模式，当调用对象时才会发送SQL
     *  4、findById()属于CrudRepository，底层调用了find()方法，当调用该方法的时就直接发送SQL
     */
    @Test
    public void test5(){
        // getOne()返回一个实体的引用，无结果会抛出异常
        User user = this.userJpaRepository.getOne(1L);
        System.out.println(user);
        // findById()返回一个Optional对象,如果Optional为空get()会抛异常
        Optional<User> optional = this.userJpaRepository.findById(1L);
        System.out.println(optional.orElse(null));
    }
}
```

查看日志：查询操作 test1() 方法

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_
User(id=1, username=王小小, password=password, age=18)
--------------华丽的分割线-------------------
User(id=2, username=隔壁小王, password=password, age=18)
User(id=3, username=赵小丽, password=666666, age=21)
User(id=4, username=王小虎, password=123456, age=25)
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    order by
        user0_.age asc
User(id=1, username=王小小, password=password, age=18)
User(id=2, username=隔壁小王, password=password, age=18)
User(id=3, username=赵小丽, password=666666, age=21)
User(id=4, username=王小虎, password=123456, age=25)
--------------华丽的分割线-------------------
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.id in (
            ? , ?
        )
User(id=1, username=王小小, password=password, age=18)
User(id=2, username=隔壁小王, password=password, age=18)
--------------华丽的分割线-------------------
Hibernate: 
    insert 
    into
        sys_user
        (age, password, username) 
    values
        (?, ?, ?)
[User(id=5, username=隔壁小王, password=password, age=18)]
```

查看日志：新增操作 test2() 方法

```verilog
Hibernate: 
    insert 
    into
        sys_user
        (age, password, username) 
    values
        (?, ?, ?)
User(id=6, username=隔壁小王, password=password, age=18)
```

查看日志：批量删除操作 test3() 方法

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_
Hibernate: 
    delete 
    from
        sys_user 
    where
        id=?
Hibernate: 
    delete 
    from
        sys_user 
    where
        id=?
Hibernate: 
    delete 
    from
        sys_user 
    where
        id=?
...
```

查看日志：批量删除操作 test4() 方法（推荐使用）

```verilog
Hibernate: 
    delete 
    from
        sys_user
```

总结对比：deleteAll() 和 deleteAllInBatch() 方法的区别：

1. deleteAll()：是删除全部，先findAll()查找出来，再一条一条删除，最后提交事务
2. deleteAllInBatch()：是删除全部，不过一条sql。显然：deleteAllInBatch() 方法效率更高
3. 在进行批量删除操作时尽量使用JpaRepository自带的批量删除方法：deleteInBatch() 及 deleteAllInBatch()
4. 可参考：https://jingyan.baidu.com/article/597a06431601fb312b52431c.html



查看日志：test5()，getOne() 可能会报错：org.hibernate.LazyInitializationException: could not initialize proxy XXX - no Session

```verilog
org.hibernate.LazyInitializationException: could not initialize proxy [com.example.jpa.entity.User#4] - no Session

	at org.hibernate.proxy.AbstractLazyInitializer.initialize(AbstractLazyInitializer.java:170)
	at org.hibernate.proxy.AbstractLazyInitializer.getImplementation(AbstractLazyInitializer.java:310)
	at org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor.intercept(ByteBuddyInterceptor.java:45)
	at org.hibernate.proxy.ProxyConfiguration$InterceptorDispatcher.intercept(ProxyConfiguration.java:95)
	at com.example.jpa.entity.User$HibernateProxy$7rZgySgi.toString(Unknown Source)
	at java.lang.String.valueOf(String.java:2994)
	at java.io.PrintStream.println(PrintStream.java:821)
	at com.example.jpa.UserJpaRepositoryTests.test5(UserJpaRepositoryTests.java:95)
```

**原因：没有开启懒加载。解决方案如下：问题解决参考：https://www.gaoyaxuan.net/blog/229.html**

方案一：在出问题的实体类上加：`@Proxy(lazy = false)`

```java
// 默认懒加载，false为急加载
@Proxy(lazy = false)
public class User {}
```

方案二：、application.properties 文件中开启延迟加载的配置

```properties
# 开启延迟加载，这个配置的意思就是在没有事务的情况下允许懒加载
spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true
```







## 6、JpaSpecificationExecutor 接口

> JpaSpecificationExecutor 接口不能单独使用，需要配合着 JPA 的其他接口一起使用

该接口主要是提供了多条件查询（动态查询），并且可以在查询中添加排序与分页。查看 JpaSpecificationExecutor 接口源码中的方法

```java
/**
 * JpaSpecificationExecutor：用来做动态查询的接口
 * Specification：是SpringDataJPA提供的一个查询规范，要做复杂的查询，只需围绕这个规范来设置查询条件。
 * JpaSpecificationExecutor接口下一共就5个接口方法
 */
public interface JpaSpecificationExecutor<T> {
	T findOne(Specification<T> spec);// 查询单个
	List<T> findAll(Specification<T> spec);// 查询全部
	Page<T> findAll(Specification<T> spec, Pageable pageable);// 查询全部【分页】
	List<T> findAll(Specification<T> spec, Sort sort);// 查询全部【排序】
	long count(Specification<T> spec);// 统计总数
}
```

Specification 是我们传入进去的查询参数，实际上它是一个接口，并且只有一个方法：

```java
public interface Specification<T> {
    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
}
```



1、创建Repository层接口，除了继承JpaSpecificationExecutor，还需要继承 JPA 其他接口

```java
/**
 * JpaSpecificationExecutor 接口
 * 该接口主要是提供了多条件查询的支持，并且可以在查询中添加排序与分页。
 * 注意：JpaSpecificationExecutor不能单独使用，需要配合着jpa中的其他接口一起使用
 */
public interface UserJpaSpecificationExecutor extends JpaRepository<User,Long>, JpaSpecificationExecutor<User> {
}
```

2、下面测试类实战操作，先准备测试数据

```java
    @Autowired
    private UserJpaRepository userJpaRepository;
    @BeforeEach
    public void initData(){
        // 保存多条数据
        User userA = new User();
        userA.setUsername("隔壁小王");
        userA.setPassword("password");
        userA.setAge(18);
        User userB = new User();
        userB.setUsername("赵小丽");
        userB.setPassword("666666");
        userB.setAge(21);
        User userC = new User();
        userC.setUsername("王小虎");
        userC.setPassword("123456");
        userC.setAge(25);
        this.userJpaRepository.saveAll(Arrays.asList(userA, userB, userC)).forEach(System.out::println);
    }
```



### 1、条件查询

```java
package com.example.jpa;

import com.example.jpa.entity.User;
import com.example.jpa.repository.UserJpaSpecificationExecutor;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.jpa.domain.Specification;
import javax.persistence.criteria.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

@SpringBootTest
public class UserSpecificationExecutorTests {

    @Autowired
    private UserJpaSpecificationExecutor userJpaSpecificationExecutor;

    /**
     * 条件查询——单条件
     * Predicate 过滤条件
     * jpql：from User where username = '隔壁小王'
     * sql：select * from sys_user where username = '隔壁小王'
     */
    @Test
    public void test1(){
        Specification<User> spec = new Specification<User>() {
            /**
             * @param root 根对象。封装了查询条件的对象
             * @param criteriaQuery 定义了一个基本的查询。一般使用较少
             * @param criteriaBuilder 创建一个查询条件
             * @return Predicate:定义了查询条件
             */
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                Path<Object> username = root.get("username");
                Predicate predicate = criteriaBuilder.equal(username, "隔壁小王");
                return predicate;
            }
        };
        List<User> userList = userJpaSpecificationExecutor.findAll(spec);
        userList.forEach(System.out::println);
    }

    /**
     * 多条件查询一
     * jpql：from User where username = '隔壁小王' and password = 'password' and age < 30
     * sql：select * from sys_user where username = '隔壁小王' and password = 'password' and age < 30
     */
    @Test
    public void test2(){
        Specification<User> spec = (root, criteriaQuery, criteriaBuilder) -> {
            Predicate username = criteriaBuilder.equal(root.get("username"), "隔壁小王");
            Predicate password = criteriaBuilder.equal(root.get("password"), "password");
            Predicate age = criteriaBuilder.lt(root.get("age"), 30);
            List<Predicate> list = Arrays.asList(username, password, age);
            return criteriaBuilder.and(list.toArray(new Predicate[]{}));
        };
        List<User> userList = userJpaSpecificationExecutor.findAll(spec);
        userList.forEach(System.out::println);
    }

    /**
     * 多条件查询二
     * jpql：from User where username like '%小王' and password = 'password' and age < 30
     * sql：select * from sys_user where username like '%小王' and password = 'password' and age < 30
     */
    @Test
    public void test3(){
        Specification<User> spec = (root, criteriaQuery, criteriaBuilder) -> {
            Predicate username = criteriaBuilder.like(root.get("username"), "%小王");
            Predicate password = criteriaBuilder.equal(root.get("password"), "password");
            Predicate age = criteriaBuilder.lt(root.get("age"), 30);
            Predicate predicate = criteriaBuilder.and(username, password);
            predicate = criteriaBuilder.and(predicate, age);
            return predicate;
        };
        List<User> userList = userJpaSpecificationExecutor.findAll(spec);
        userList.forEach(System.out::println);
    }
}
```

查看日志：test1()

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.username=?
User(id=1, username=隔壁小王, password=password, age=18)
```

查看日志：test2()

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.username=? 
        and user0_.password=? 
        and user0_.age<30
User(id=1, username=隔壁小王, password=password, age=18)
```

查看日志：test3()

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        (
            user0_.username like ?
        ) 
        and user0_.password=? 
        and user0_.age<30
User(id=1, username=隔壁小王, password=password, age=18)
```



### 2、分页查询

```java
    /**
     * 分页操作
     */
    @Test
    public void testPageable(){
        Specification<User> spec = (root, criteriaQuery, criteriaBuilder) -> {
            Predicate username = criteriaBuilder.equal(root.get("username"), "隔壁小王");
            Predicate password = criteriaBuilder.equal(root.get("password"), "password");
            return criteriaBuilder.and(username, password);
        };
        //分页
        Pageable pageable = PageRequest.of(0, 2);
        Page<User> page = this.userJpaSpecificationExecutor.findAll(spec, pageable);
        System.out.println("总记录数: " + page.getTotalElements());
        System.out.println("当前第几页: " + (page.getNumber() + 1));
        System.out.println("总页数: " + page.getTotalPages());
        System.out.println("当前页面的 List: " + page.getContent());
        System.out.println("当前页面的记录数: " + page.getNumberOfElements());
        page.getContent().forEach(System.out::println);
    }
```

查看日志：

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.username=? 
        and user0_.password=? limit ?
总记录数: 1
当前第几页: 1
总页数: 1
当前页面的 List: [User(id=1, username=隔壁小王, password=password, age=18)]
当前页面的记录数: 1
User(id=1, username=隔壁小王, password=password, age=18)
```





### 3、排序查询

```java
    /**
     * 排序操作
     */
    @Test
    public void testSort(){
        Specification<User> spec = (root, criteriaQuery, criteriaBuilder) -> {
            Predicate username = criteriaBuilder.equal(root.get("username"), "隔壁小王");
            Predicate password = criteriaBuilder.equal(root.get("password"), "password");
            return criteriaBuilder.and(username, password);
        };
        // 排序
        Sort sort = Sort.by(Sort.Direction.DESC,"id");
        List<User> users = this.userJpaSpecificationExecutor.findAll(spec, sort);
        users.forEach(System.out::println);
    }
```

查看日志：

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.username=? 
        and user0_.password=? 
    order by
        user0_.id desc
User(id=1, username=隔壁小王, password=password, age=18)
```



### 4、分页 + 排序查询

```java
    /**
     * 分页+排序 操作
     */
    @Test
    public void testPageableAndSort(){
        Specification<User> spec = (root, criteriaQuery, criteriaBuilder) -> {
            Predicate username = criteriaBuilder.equal(root.get("username"), "隔壁小王");
            Predicate password = criteriaBuilder.equal(root.get("password"), "password");
            return criteriaBuilder.and(username, password);
        };
        // 排序
        Sort sort = Sort.by(Sort.Direction.DESC,"id");
        // 分页
        Pageable pageable = PageRequest.of(0, 2, sort);
        Page<User> page = this.userJpaSpecificationExecutor.findAll(spec, pageable);
        System.out.println("总记录数: " + page.getTotalElements());
        System.out.println("当前第几页: " + (page.getNumber() + 1));
        System.out.println("总页数: " + page.getTotalPages());
        System.out.println("当前页面的 List: " + page.getContent());
        System.out.println("当前页面的记录数: " + page.getNumberOfElements());
        page.getContent().forEach(System.out::println);
    }
```

查看日志：

```verilog
Hibernate: 
    select
        user0_.id as id1_1_,
        user0_.age as age2_1_,
        user0_.password as password3_1_,
        user0_.username as username4_1_ 
    from
        sys_user user0_ 
    where
        user0_.username=? 
        and user0_.password=? 
    order by
        user0_.id desc limit ?
总记录数: 1
当前第几页: 1
总页数: 1
当前页面的 List: [User(id=1, username=隔壁小王, password=password, age=18)]
当前页面的记录数: 1
User(id=1, username=隔壁小王, password=password, age=18)
```



> 5、多表查询（使用 单向多对一 模式）

```java
/**
 * 多方外键表
 * 单向多对一模式
 */
@Data
@Entity
@Table(name = "sys_address")
public class UserAddress {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "address_id")
    private Long addressId;
    private String country;
    private String city;

    @ManyToOne
    @JoinColumn(name = "user_id", referencedColumnName = "id")
    private User user;
}

/**
 * 一方主键表
 */
@Data
@Entity
@Table(name = "sys_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private Integer age;
}
```

测试代码：

```java
    /**
     * Join代表链接查询，通过root对象获取
     * 创建的过程中，第一个参数为关联对象的属性名称，第二个参数为连接查询的方式（left，inner，right）
     * JoinType.LEFT : 左外连接,JoinType.INNER：内连接,JoinType.RIGHT：右外连接
     * 使用多表关联后，Join 就相当有了 root 的功能，可以join.get("对象属性名称")
     * 注意！注意！注意：
     *     Root：代表的是UserAddress表，所以通过root.get()只能获取Many表的属性或字段.
     *     join：代表的是One表，虽然是关联查询，但它只代表关联的One表，只能获取One的属性
     *     如果root/join获取自己表以外的属性或字段会报如下错：
     *     Unable to locate Attribute  with the the given name [categoryType] on this ManagedType[XXX]
     *     如果有第三张表关联使用join.join()往下传递即可。
     */
    @Test
    public void testFindJoinTable(){
        // 1.初始化数据
        User user = new User();
        user.setUsername("老王");
        user.setPassword("000000");
        user.setAge(19);
        userJpaSpecificationExecutor.save(user);
        UserAddress userAddressA = new UserAddress();
        userAddressA.setCountry("中国");
        userAddressA.setCity("广州");
        userAddressA.setUser(user);
        UserAddress userAddressB = new UserAddress();
        userAddressB.setCountry("中国");
        userAddressB.setCity("深圳");
        userAddressB.setUser(user);
        userAddressJpaSpecificationExecutor.saveAll(Arrays.asList(userAddressA,userAddressB));

        // 2.联表复杂查询
        Specification<UserAddress> spec = (root, criteriaQuery, criteriaBuilder) -> {
            Join<UserAddress, User> join = root.join("user", JoinType.INNER);
            Path<String> username = join.get("username");
            Path<Object> city = root.get("city");
            // Path<Object> city 由于Path没有指定类型，所以criteriaBuilder.like(city.as(String.class)这里指定了as(String.class)
            return criteriaBuilder.like(city.as(String.class), "广州");
        };
        userAddressJpaSpecificationExecutor.findAll(spec).forEach(System.out::println);
    }
```

查看日志：

```verilog
Hibernate: 
    insert 
    into
        sys_user
        (age, password, username) 
    values
        (?, ?, ?)
Hibernate: 
    insert 
    into
        sys_address
        (city, country, user_id) 
    values
        (?, ?, ?)
Hibernate: 
    insert 
    into
        sys_address
        (city, country, user_id) 
    values
        (?, ?, ?)
Hibernate: 
    select
        useraddres0_.address_id as address_1_0_,
        useraddres0_.city as city2_0_,
        useraddres0_.country as country3_0_,
        useraddres0_.user_id as user_id4_0_ 
    from
        sys_address useraddres0_ 
    inner join
        sys_user user1_ 
            on useraddres0_.user_id=user1_.id 
    where
        useraddres0_.city like ?
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.age as age2_1_0_,
        user0_.password as password3_1_0_,
        user0_.username as username4_1_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
UserAddress(addressId=1, country=中国, city=广州, user=User(id=1, username=老王, password=000000, age=19))
```

> PS 注意（重点）：
>
> 可以发现 `Specification` 重写方法里面的`root`，`criteriaQuery`和`builder`都已经被JPA赋值好了，我们只需要关注`Predicate`的构建，也就是说在调用`JpaSpecificationExecutor`接口内的`findXXX`方法只能完成`where`条件的构建，而不能实现`select`后面属性的选择和`groupBy`的构建。



### 5、投影和分组查询

虽然JpaSpecificationExecutor无法实现`select`后面属性的选择和`groupBy`的构建，不过可以直接使用`Criteria`方式来完成。

1、创建接口实现代码类（为了简便没有写接口了，直接写了类）（主要关注`@PersistenceContext`注入`EntityManager`对象）

```java
package com.example.jpa.repository;

import com.example.jpa.entity.User;
import org.springframework.stereotype.Repository;
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Tuple;
import javax.persistence.TypedQuery;
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Root;
import java.util.List;

@Repository
public class UserCriteriaRepository {
    @PersistenceContext
    private EntityManager entityManager;

    /**
     * 投影查询 — criteriaBuilder.createTupleQuery()
     * sql：select username, age from sys_user
     * 注意：
     *   使用：返回元组(Tuple)的查询
     */
    public List<Tuple> testFindAllMultiSelect(){
        // 1.CriteriaBuilder 安全查询创建工厂
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        // 2.CriteriaQuery 安全查询主语句
        CriteriaQuery<Tuple> criteriaQuery = criteriaBuilder.createTupleQuery();
        // 3.Root 定义查询的From子句中能出现的类型
        Root<User> from = criteriaQuery.from(User.class);

        // 需要给字段取别名,否则无法通过tuple.get(field)获取数据
        criteriaQuery.multiselect(from.get("username").alias("username"), from.get("age").alias("age"));
        TypedQuery<Tuple> query = entityManager.createQuery(criteriaQuery);
        List<Tuple> list = query.getResultList();
        return list;
    }

    /**
     * 分组查询
     * sql：select username,count(username),max(age),min(age),sum(age),avg(age) from sys_user group by username
     */
    public List<Tuple> testFindGroupBy() {
        CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Tuple> criteriaQuery = criteriaBuilder.createTupleQuery();
        Root<User> from = criteriaQuery.from(User.class);

        // .alias("name") 取别名
        criteriaQuery.multiselect(
                from.get("username").alias("username"),
                criteriaBuilder.count(from.get("username")).alias("count"),
                criteriaBuilder.max(from.get("age")).alias("max"),
                criteriaBuilder.min(from.get("age")).alias("min"),
                criteriaBuilder.sum(from.get("age")).alias("sum"),
                criteriaBuilder.avg(from.get("age")).alias("avg"));
        criteriaQuery.groupBy(from.get("username"));
        // criteriaQuery.having(criteriaBuilder.disjunction());

        List<Tuple> tupleList = entityManager.createQuery(criteriaQuery).getResultList();
        return tupleList;
    }
}

```

2、测试代码：

```java
package com.example.jpa;

import com.example.jpa.entity.User;
import com.example.jpa.repository.UserCriteriaRepository;
import com.example.jpa.repository.UserJpaRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import javax.persistence.Tuple;
import java.util.Arrays;
import java.util.List;

@SpringBootTest
public class UserCriteriaRepositoryTests {

    @Autowired
    private UserJpaRepository userJpaRepository;
    @BeforeEach
    public void initData(){
        // 保存多条数据
        User userA = new User();
        userA.setUsername("隔壁小王");
        userA.setPassword("password");
        userA.setAge(18);
        User userB = new User();
        userB.setUsername("赵小丽");
        userB.setPassword("666666");
        userB.setAge(21);
        User userC = new User();
        userC.setUsername("王小虎");
        userC.setPassword("123456");
        userC.setAge(25);
        this.userJpaRepository.saveAll(Arrays.asList(userA, userB, userC)).forEach(System.out::println);
    }

    @Autowired
    private UserCriteriaRepository userCriteriaRepository;

    @Test
    public void testFindAllMultiSelect(){
        List<Tuple> tuples = userCriteriaRepository.testFindAllMultiSelect();
        // name = list.get(0).get(0); age = list.get(0).get(1)
        tuples.forEach(x-> System.out.println(x.get("username")+"、"+x.get("age")));
    }

    @Test
    public void testFindGroupBy(){
        List<Tuple> tuples = userCriteriaRepository.testFindGroupBy();
        tuples.forEach(x-> System.out.println(x.get("username") + "、" +
                        x.get("count") + "、" +
                        x.get("max") + "、" +
                        x.get("min") + "、" +
                        x.get("sum") + "、" +
                        x.get("avg"))
                );
    }
}

```

3、查看日志：投影查询

```verilog
Hibernate: 
    select
        user0_.username as col_0_0_,
        user0_.age as col_1_0_ 
    from
        sys_user user0_
隔壁小王、18
赵小丽、21
王小虎、25
```

4、查看日志：分组查询

```verilog
Hibernate: 
    select
        user0_.username as col_0_0_,
        count(user0_.username) as col_1_0_,
        max(user0_.age) as col_2_0_,
        min(user0_.age) as col_3_0_,
        sum(user0_.age) as col_4_0_,
        avg(user0_.age) as col_5_0_ 
    from
        sys_user user0_ 
    group by
        user0_.username
王小虎、1、25、25、25、25.0
赵小丽、1、21、21、21、21.0
隔壁小王、1、18、18、18、18.0
```

> 所以如果想投影查询和分组查询的话还得使用`Criteria`方式来查询。
>
> 一般的复杂或者连表复杂查询使用CriteriaBuilder、CriteriaQuery、Root 来完成查询。如果不需要构建Selection、groupBy时，也可以只构建`Predicate`，然后使用JPA的`findAll()`方法即可。



## 7、Repository 自定义接口

有时JPA接口提供的方法并不满足我们的需求，这时我们可以自定义Repository来扩展。主要通过`@PersistenceContext` 获取 `EntityManager` 对象来实现

1、定义一个普通接口

```java
/**
 * 自定义Repository接口
 */
public interface UserCustomRepository {
    User findUserById(Long id);
}
```

2、创建接口实现类（使用 `@PersistenceContext` 注入 `EntityManager` 对象）

```java
public class UserCustomRepositoryImpl implements UserCustomRepository {
    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public User findUserById(Long id) {
        return entityManager.find(User.class,id);
    }
}
```

3、使用接口

```java
/**
 * 用户自定义Repository接口讲解
 */
public interface UserDao extends JpaRepository<User, Integer>, JpaSpecificationExecutor<User>, UserCustomRepository{
}
```

4、测试代码

```java
@SpringBootTest
public class UserCustomRepositoryTests {
    @Autowired
    private UserDao userDao;
    @Test
    public void testFind(){
        System.out.println(userDao.findUserById(6L));
    }
}
```

5、查看日志

```verilog
Hibernate: 
    select
        user0_.id as id1_0_0_,
        user0_.age as age2_0_0_,
        user0_.password as password3_0_0_,
        user0_.username as username4_0_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
User(id=2, username=王小虎, password=123456, age=25)
```



# SpringData JPA 数据表映射

## 1、映射注解说明

```
1.配置多表联系注解介绍
@OneToOne                           一对一映射
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
            
            
3：首先确定多表之间的关系：
    一对一和一对多：
        一的一方称为主表，而多的一方称为从表，外键就要建立在从表上，它们的取值的来源主要来自主键
    多对多：
        这个时候需要建立一个中间表，中间表中最少由2个字段组成，这2个字段作为外键指向2张表的主键又组成了联合主键

4、mappedBy 属性
mappedBy是OneToOne、OneToMany和ManyToMany这三种关联关系的属性。
用来标注拥有这种关系的字段。 除非关系是单向的，否则是必需的。
那么什么叫拥有关联关系呢，假设是双向一对一的话，那么拥有关系的这一方有建立、解除和更新与另一方关系的能力。而另一方没有，只能被动管理。
由于JoinTable和JoinColumn一般定义在拥有关系的这一端，而mappedBy一定是定义在关系的被拥有方（the owned side），也就是跟定义JoinTable和JoinColumn互斥的一方，它的值指向拥有方中关于被拥有方的字段，可能是一个对象（OneToMany），也可能是一个对象集合（ManyToMany）
```



## 2、一对一关系映射

### 2.1、准备一对一

1、表结构和需求

```sql
-- 需求:学生表与学生信息表的一对一的关联关系
-- 学生表（主键表）:一方
-- 学生信息表（外键表）:一方

-- 学生表结构
+--------------+--------------+------+-----+---------+----------------+
| Field        | Type         | Null | Key | Default | Extra          |
+--------------+--------------+------+-----+---------+----------------+
| student_id   | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| student_name | varchar(255) | YES  |     | NULL    |                |
| student_pwd  | varchar(255) | YES  |     | NULL    |                |
+--------------+--------------+------+-----+---------+----------------+

-- 学生信息表结构
+----------------+--------------+------+-----+---------+----------------+
| Field          | Type         | Null | Key | Default | Extra          |
+----------------+--------------+------+-----+---------+----------------+
| information_id | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| student_age    | int(11)      | YES  |     | NULL    |                |
| student_number | varchar(255) | YES  |     | NULL    |                |
| student_id     | bigint(20)   | YES  | MUL | NULL    |                |
+----------------+--------------+------+-----+---------+----------------+
```

2、创建2个实体类对应的DAO接口

```java
/**
 * 学生表repository接口
 */
public interface StudentRepository extends JpaRepository<Student,Long> {
}
```

```java
/**
 * 学生信息表repository接口
 */
public interface StudentInformationRepository extends JpaRepository<StudentInformation,Long> {
}
```



### 2.2、单向一对一

1、创建：学生信息实体类（拥有方，外键表）

```java
@Data
@Entity
@Table(name = "tb_student_information")
public class StudentInformation implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "information_id")
    private Long informationId;
    @Column(name = "student_number")
    private String studentNumber;
    @Column(name = "student_age")
    private Integer studentAge;

    // 单向的话配置可以只配置拥有方（外键方）
    @OneToOne
    @JoinColumn(name = "student_id")
    private Student student;
}
```

2、创建：学生实体类（被拥有方，主键表）

```java
@Data
@Entity
@Table(name = "tb_student")
public class Student implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long studentId;
    @Column(name = "student_name")
    private String studentName;
    @Column(name = "student_pwd")
    private String studentPwd;
}
```

3、测试代码：

```java
@SpringBootTest
public class AssociationTableTests {
    @Autowired
    StudentRepository studentRepository;
    @Autowired
    StudentInformationRepository informationRepository;

    /**
     * 单向一对一
     */
    @Test
    public void testOneToOneSave(){
        // 首先给学生表（主键表）增加一条数据
        Student student = new Student();
        student.setStudentName("小明");
        student.setStudentPwd("123456");
        studentRepository.saveAndFlush(student);

        // 然后再给学生信息表（外键表）增加一条数据
        StudentInformation information = new StudentInformation();
        information.setStudentNumber("001");
        information.setStudentAge(13);
        // 设置这个字段相当于给外键设置值，如果没有设置那么保存后外键字段将为null
        information.setStudent(student);
        informationRepository.saveAndFlush(information);

        // 查询信息
        System.out.println(teacherRepository.findById(student.getStudentId()));
        System.out.println(informationRepository.findById(information.getInformationId()));
    }
}
```

查看日志：

```verilog
Hibernate: 
    insert 
    into
        tb_student
        (student_name, student_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_information
        (student_id, student_age, student_number) 
    values
        (?, ?, ?)
Hibernate: 
    select
        teacher0_.teacher_id as teacher_1_5_0_,
        teacher0_.teacher_name as teacher_2_5_0_,
        teacher0_.teacher_pwd as teacher_3_5_0_ 
    from
        tb_teacher teacher0_ 
    where
        teacher0_.teacher_id=?
Optional.empty
Hibernate: 
    select
        studentinf0_.information_id as informat1_3_0_,
        studentinf0_.student_id as student_4_3_0_,
        studentinf0_.student_age as student_2_3_0_,
        studentinf0_.student_number as student_3_3_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_information studentinf0_ 
    left outer join
        tb_student student1_ 
            on studentinf0_.student_id=student1_.student_id 
    where
        studentinf0_.information_id=?
Optional[StudentInformation(informationId=8, studentNumber=001, studentAge=13, student=Student(studentId=8, studentName=小明, studentPwd=123456))]
```



### 2.3、双向一对一

1、学生信息实体类（拥有方，外键表）不变

```java
@Data
@Entity
@Table(name = "tb_student_information")
public class StudentInformation implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "information_id")
    private Long informationId;
    @Column(name = "student_number")
    private String studentNumber;
    @Column(name = "student_age")
    private Integer studentAge;

    // 单向的话配置可以只配置拥有方（外键方）
    @OneToOne
    @JoinColumn(name = "student_id")
    private Student student;
}
```

2、学生实体类（主键表）需要增加属性与注解

```java
@Data
@Entity
@Table(name = "tb_student")
public class Student implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long studentId;
    @Column(name = "student_name")
    private String studentName;
    @Column(name = "student_pwd")
    private String studentPwd;
    // 上面的属性与单向一对一是一样的，就增加了如下信息
    /**
     * mappedBy:放弃维护外键，值为另一方引用本类的属性名
     */
    @OneToOne(mappedBy = "student")
    private StudentInformation studentInformation;
}
```

3、测试代码与单向一对一也是一样的

```java
@SpringBootTest
public class AssociationTableTests {
    @Autowired
    StudentRepository studentRepository;
    @Autowired
    StudentInformationRepository informationRepository;

    /**
     * 双向一对一
     */
    @Test
    public void testOneToOneSave(){
        // 首先给学生表（主键表）增加一条数据
        Student student = new Student();
        student.setStudentName("小明");
        student.setStudentPwd("123456");
        studentRepository.saveAndFlush(student);

        // 然后再给学生信息表（外键表）增加一条数据
        StudentInformation information = new StudentInformation();
        information.setStudentNumber("001");
        information.setStudentAge(13);
        // 设置这个字段相当于给外键设置值，如果没有设置那么保存后外键字段将为null
        information.setStudent(student);
        informationRepository.saveAndFlush(information);

        // 查询信息
        System.out.println(teacherRepository.findById(student.getStudentId()));
        System.out.println(informationRepository.findById(information.getInformationId()));
    }
}
```

4、查看日志：

```verilog
// 插入语句与查询语句
Hibernate: 
    insert 
    into
        tb_student
        (student_name, student_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_information
        (student_id, student_age, student_number) 
    values
        (?, ?, ?)
Hibernate: 
    select
        teacher0_.teacher_id as teacher_1_5_0_,
        teacher0_.teacher_name as teacher_2_5_0_,
        teacher0_.teacher_pwd as teacher_3_5_0_ 
    from
        tb_teacher teacher0_ 
    where
        teacher0_.teacher_id=?
Optional.empty
Hibernate: 
    select
        studentinf0_.information_id as informat1_3_0_,
        studentinf0_.student_id as student_4_3_0_,
        studentinf0_.student_age as student_2_3_0_,
        studentinf0_.student_number as student_3_3_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_information studentinf0_ 
    left outer join
        tb_student student1_ 
            on studentinf0_.student_id=student1_.student_id 
    where
        studentinf0_.information_id=?
// 查询后开始报错
java.lang.StackOverflowError
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at com.example.jpa.entity.Student.toString(Student.java:13)
	at java.lang.String.valueOf(String.java:2994)
```

可以发现会报如上栈溢出错误。原因：双向查询后使用（System.out）输出时，toString() 方式时各自对象相互引用导致死循环。

解决方案：

1. 去除@Data注解，替换成@Getter和@Setter注解。使用@ToString(exclude = "引用类型字段")
2. 也可以重写toString()方法，把引用类型的输出删掉即可
3. 查询后可以不要输出打印该对象，这样就不会死循环输出对象中的对象了（治标不治本, 不推荐）
4. ~~在实体类上增加`@JsonIgnoreProperties(value = "对方类引用本类的属性名")`（暂时我试了时无效的，打印对象还是会报错）~~



## 3、一对多关系映射

### 3.1、准备一对多

1、表结构和需求

```sql
-- 需求:学生表与学生称即表的一对多的关联关系
-- 学生表（主键表）:一方
-- 学生成绩表（外键表）:多方

-- 学生表结构
+--------------+--------------+------+-----+---------+----------------+
| Field        | Type         | Null | Key | Default | Extra          |
+--------------+--------------+------+-----+---------+----------------+
| student_id   | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| student_name | varchar(255) | YES  |     | NULL    |                |
| student_pwd  | varchar(255) | YES  |     | NULL    |                |
+--------------+--------------+------+-----+---------+----------------+

-- 学生成绩表结构
+----------------+--------------+------+-----+---------+----------------+
| Field          | Type         | Null | Key | Default | Extra          |
+----------------+--------------+------+-----+---------+----------------+
| result_id      | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| result_score   | float        | YES  |     | NULL    |                |
| result_subject | varchar(255) | YES  |     | NULL    |                |
| student_id     | bigint(20)   | YES  | MUL | NULL    |                |
+----------------+--------------+------+-----+---------+----------------+
```

2、创建2个实体类对应的DAO接口

```java
/**
 * 学生表repository接口
 */
public interface StudentRepository extends JpaRepository<Student,Long> {
}
```

```java
/**
 * 学生成绩表repository接口
 */
public interface StudentResultRepository extends JpaRepository<StudentResult,Long> {
}
```



### 3.2、单向一对多

单向一对多的话，那么只能是：一方（主键表）为拥有方，多方（外键表）为被拥有端。（只配置一方即可）

1、创建：学生实体类（拥有方，主键表）只配置一方（主键方）即可

```java
@Data
@Entity
@Table(name = "tb_student")
public class Student implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long studentId;
    @Column(name = "student_name")
    private String studentName;
    @Column(name = "student_pwd")
    private String studentPwd;

    /**
     * 单向一对多：只需要配置一方（主键表）即可
     * 一般配置@OneToMany + @JoinColumn
     * 也可以只配置@OneToMany，@JoinColumn会按照默认规则生成外键
     * @OneToMany：
     *     targetEntity：指定对方类的字节码（可选）
     *     fetch：指定是懒加载还是立即加载（@OneToMany中默认是懒加载）（最好必填）
     * @JoinColumn：
     *     name：外键字段名称（必填，不然会默认生成外键字段名）
     *     referencedColumnName：指定引用主表的主键字段名称（注意是数据库字段不是实体类属性名）
     * 注意！注意！注意！
     * 一对多时一方默认时开启懒加载的，需要关闭，非则查询会报错
     */
    @OneToMany(targetEntity = StudentResult.class,fetch = FetchType.EAGER)
    @JoinColumn(name="student_id",referencedColumnName = "student_id")
    private List<StudentResult> studentResults = new ArrayList<>();
}
```

2、创建：学生成绩实体类（被拥有方，外键表），多方（外键方）无需配置

```java
@Data
@Entity
@Table(name = "tb_student_result")
public class StudentResult {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "result_id")
    private Long resultId;
    @Column(name = "result_subject")
    private String resultSubject;
    @Column(name = "result_score")
    private float resultScore;
}
```

3、测试代码：

```java
    /**
     * 单向 一对多
     * 数据插入的步骤为：
     * 先插入外键表数据，此时外键表中的外键为null，
     * 然后在插入主键表数据，最后更新外键表中的外键
     * 可以查看日志输入的顺序
     */
    @Test
    public void testOneToManySave(){
        // 首先给学生成绩表插入两条数据
        StudentResult studentResultA = new StudentResult();
        studentResultA.setResultSubject("语文");
        studentResultA.setResultScore(89.5f);
        resultRepository.saveAndFlush(studentResultA);
        StudentResult studentResultB = new StudentResult();
        studentResultB.setResultSubject("数学");
        studentResultB.setResultScore(92f);
        resultRepository.saveAndFlush(studentResultB);

        // 然后给学生表（主键表）增加一条数据
        Student student = new Student();
        student.setStudentName("小明");
        student.setStudentPwd("123456");
        student.setStudentResults(Arrays.asList(studentResultA, studentResultB));
        studentRepository.saveAndFlush(student);

        // 查询学生(主)表数据
        System.out.println(studentRepository.findById(student.getStudentId()));
        // 查询成绩（从）表数据
        System.out.println(resultRepository.findById(studentResultA.getResultId()));
        System.out.println(resultRepository.findById(studentResultB.getResultId()));
    }
```

4、查看日志

```verilog
// 初始化表的语句
Hibernate: 
    create table tb_student (
       student_id bigint not null auto_increment,
        student_name varchar(255),
        student_pwd varchar(255),
        primary key (student_id)
    ) engine=InnoDB
Hibernate: 
    create table tb_student_result (
       result_id bigint not null auto_increment,
        result_score float,
        result_subject varchar(255),
        student_id bigint,
        primary key (result_id)
    ) engine=InnoDB
Hibernate: 
    alter table tb_student_result 
       add constraint FK5101rgeygkbcenssvt342o3kg 
       foreign key (student_id) 
       references tb_student (student_id)
```

```verilog
Hibernate: // 插入数据语句
    insert 
    into
        tb_student_result
        (result_score, result_subject) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_result
        (result_score, result_subject) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student
        (student_name, student_pwd) 
    values
        (?, ?)
Hibernate: 
    update
        tb_student_result 
    set
        student_id=? 
    where
        result_id=?
Hibernate: 
    update
        tb_student_result 
    set
        student_id=? 
    where
        result_id=?
Hibernate: // 查询语句
    select
        student0_.student_id as student_1_2_0_,
        student0_.student_name as student_2_2_0_,
        student0_.student_pwd as student_3_2_0_,
        studentres1_.student_id as student_4_4_1_,
        studentres1_.result_id as result_i1_4_1_,
        studentres1_.result_id as result_i1_4_2_,
        studentres1_.result_score as result_s2_4_2_,
        studentres1_.result_subject as result_s3_4_2_ 
    from
        tb_student student0_ 
    left outer join
        tb_student_result studentres1_ 
            on student0_.student_id=studentres1_.student_id 
    where
        student0_.student_id=?
// 输出查询结果
Optional[Student(studentId=1, studentName=小明, studentPwd=123456, studentResults=[StudentResult(resultId=1, resultSubject=语文, resultScore=89.5), StudentResult(resultId=2, resultSubject=数学, resultScore=92.0)])]
Hibernate: // 查询语句
    select
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_score as result_s2_4_0_,
        studentres0_.result_subject as result_s3_4_0_ 
    from
        tb_student_result studentres0_ 
    where
        studentres0_.result_id=?
Optional[StudentResult(resultId=1, resultSubject=语文, resultScore=89.5)]
Hibernate: 
    select
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_score as result_s2_4_0_,
        studentres0_.result_subject as result_s3_4_0_ 
    from
        tb_student_result studentres0_ 
    where
        studentres0_.result_id=?
Optional[StudentResult(resultId=2, resultSubject=数学, resultScore=92.0)]
```

PS：可能会出现如下报错信息：

```
org.hibernate.LazyInitializationException: failed to lazily initialize XXXXX could not initialize proxy - no Session
```

问题解决：这个问题是由于实体中一对多或者多对多关联关系的加载方式配置不当引起的。一对多或者多对多关联关系的加载策略使用了懒加载，结果在加载子实体时就会报：org.hibernate.LazyInitializationException: failed to lazily initialize XXXXX could not initialize proxy - no Session错误，只需要将懒加载改为急加载即可

方式一：修改实体类

```java
// 修改学生实体类（主表），从一方获取多方的时候默认时懒加载.改成立即加载
@OneToMany(fetch = FetchType.EAGER)
```

方式二：配置文件修改

```properties
# 开启延迟加载
spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true
```

两者方式选择一种即可。



### 3.3、单向多对一

单向多对一的话：多方（外键表）为拥有方，一方（主键表）为被拥有端。（只配置多方即可）

1、创建：学生成绩实体类（拥有方，外键表）（只配置多方（外键方）即可）

```java
@Data
@Entity
@Table(name = "tb_student_result")
public class StudentResult {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "result_id")
    private Long resultId;
    @Column(name = "result_subject")
    private String resultSubject;
    @Column(name = "result_score")
    private float resultScore;

    /**
     * 单向多对一：只配置多方（外键方）即可。
     * @ManyToOne
     *     name：指定对方类的字节码（可选）
     * @JoinColumn
     *     name：外键字段名称（必填，不然会默认生成外键字段名）
     *     referencedColumnName：指定引用主表的主键字段名称（注意是数据库字段不是实体类属性名）
     */
    @ManyToOne(targetEntity = Student.class)
    @JoinColumn(name = "student_id", referencedColumnName = "student_id")
    private Student student;
}
```

2、创建：学生实体类（被拥有方，主键表）（一方（主键方）无需配置）

```java
@Data
@Entity
@Table(name = "tb_student")
public class Student implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long studentId;
    @Column(name = "student_name")
    private String studentName;
    @Column(name = "student_pwd")
    private String studentPwd;
}
```

3、测试代码

```java
    /**
     * 单向 多对一
     * 数据插入的步骤为：先插入插入主键表数据，然后在外键表数据
     */
    @Test
    public void testManyToOneSave(){

        // 首先给学生表（主键表）增加一条数据
        Student student = new Student();
        student.setStudentName("小明");
        student.setStudentPwd("123456");
        studentRepository.saveAndFlush(student);

        /**
         * 然后给学生成绩表插入两条数据
         * 与单向一对多不同的是：setStudent(student),这个步骤就是给外键字段设置值
         */
        StudentResult studentResultA = new StudentResult();
        studentResultA.setResultSubject("语文");
        studentResultA.setResultScore(89.5f);
        studentResultA.setStudent(student);
        resultRepository.saveAndFlush(studentResultA);
        StudentResult studentResultB = new StudentResult();
        studentResultB.setResultSubject("数学");
        studentResultB.setResultScore(92f);
        studentResultB.setStudent(student);
        resultRepository.saveAndFlush(studentResultB);

        // 查询学生(主)表数据
        System.out.println(studentRepository.findById(student.getStudentId()));
        // 查询成绩（从）表数据
        System.out.println(resultRepository.findById(studentResultA.getResultId()));
        System.out.println(resultRepository.findById(studentResultB.getResultId()));
    }
```

4、查看日志

```verilog
// 初始化表的语句
Hibernate: 
    create table tb_student (
       student_id bigint not null auto_increment,
        student_name varchar(255),
        student_pwd varchar(255),
        primary key (student_id)
    ) engine=InnoDB
Hibernate: 
    create table tb_student_result (
       result_id bigint not null auto_increment,
        result_score float,
        result_subject varchar(255),
        student_id bigint,
        primary key (result_id)
    ) engine=InnoDB
Hibernate: 
    alter table tb_student_result 
       add constraint FK5101rgeygkbcenssvt342o3kg 
       foreign key (student_id) 
       references tb_student (student_id)
```

```verilog
Hibernate: // 插入语句
    insert 
    into
        tb_student
        (student_name, student_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_result
        (result_score, result_subject, student_id) 
    values
        (?, ?, ?)
Hibernate: 
    insert 
    into
        tb_student_result
        (result_score, result_subject, student_id) 
    values
        (?, ?, ?)
Hibernate: // 查询学生表语句
    select
        student0_.student_id as student_1_2_0_,
        student0_.student_name as student_2_2_0_,
        student0_.student_pwd as student_3_2_0_ 
    from
        tb_student student0_ 
    where
        student0_.student_id=?
Optional[Student(studentId=1, studentName=小明, studentPwd=123456)]
Hibernate: // 关联查询语句
    select
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_score as result_s2_4_0_,
        studentres0_.result_subject as result_s3_4_0_,
        studentres0_.student_id as student_4_4_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_result studentres0_ 
    left outer join
        tb_student student1_ 
            on studentres0_.student_id=student1_.student_id 
    where
        studentres0_.result_id=?
Optional[StudentResult(resultId=1, resultSubject=语文, resultScore=89.5, student=Student(studentId=1, studentName=小明, studentPwd=123456))]
Hibernate: 
    select
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_score as result_s2_4_0_,
        studentres0_.result_subject as result_s3_4_0_,
        studentres0_.student_id as student_4_4_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_result studentres0_ 
    left outer join
        tb_student student1_ 
            on studentres0_.student_id=student1_.student_id 
    where
        studentres0_.result_id=?
Optional[StudentResult(resultId=2, resultSubject=数学, resultScore=92.0, student=Student(studentId=1, studentName=小明, studentPwd=123456))]
```



### 3.4、双向一对多

双向一对多的话：多方（外键表）为拥有方，一方（主键表）为被拥有端。

1、创建：学生成绩实体类（拥有方，多方、外键表）（多方 / 外键方 配置：@ManyToOne + @JoinColumn）
PS：实际跟单向多对一代码一样，多了一个重写toString方法，为了避免输出死循环

```java
@Data
@ToString(exclude = "student") // 为了避免输出死循环重写了toString
@Entity
@Table(name = "tb_student_result")
public class StudentResult {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "result_id")
    private Long resultId;
    @Column(name = "result_subject")
    private String resultSubject;
    @Column(name = "result_score")
    private float resultScore;

    /**
     * 单向多对一：只配置多方（外键方）即可。
     * @ManyToOne
     *     name：指定对方类的字节码（可选）
     * @JoinColumn
     *     name：外键字段名称（必填，不然会默认生成外键字段名）
     *     referencedColumnName：指定引用主表的主键字段名称（注意是数据库字段不是实体类属性名）
     */
    @ManyToOne(targetEntity = Student.class)
    @JoinColumn(name = "student_id", referencedColumnName = "student_id")
    private Student student;
}
```

2、创建：学生实体类（被拥有方，一方、主键表）（一方 / 主键方 配置：@OneToMany）
PS：实际上跟单向多对一相比：加了一个属性加上了@OneToMany，并且重写toString方法避免死循环

```java
@Data
@ToString(exclude = "studentResults") // 为了避免输出死循环重写了toString
@Entity
@Table(name = "tb_student")
public class Student implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long studentId;
    @Column(name = "student_name")
    private String studentName;
    @Column(name = "student_pwd")
    private String studentPwd;

    /**
     * 双向一对多：（实际就比单向多对一多了一个@OneToMany）
     * 必须在一方配置：@OneToMany：mappedBy + fetch 属性
     * @OneToMany
     *     mappedBy：放弃维护外键，值为另一方（拥有方 / 外键方 / 多方）引用本类的属性名
     *     fetch：指定是懒加载还是立即加载（@OneToMany中默认是懒加载）（最好必填）
     */
    @OneToMany(mappedBy = "student",fetch = FetchType.EAGER)
    private List<StudentResult> studentResults = new ArrayList<>();
}
```

3、测试代码

```java
    /**
     * 双向 一对多
     * 数据插入的步骤为：先插入插入主键表数据，然后在外键表数据
     */
    @Test
    public void testOneToManyTwoWaySave2(){

        // 首先给学生表（主键表）增加一条数据
        Student student = new Student();
        student.setStudentName("小明");
        student.setStudentPwd("123456");
        studentRepository.saveAndFlush(student);

        /**
         * 然后给学生成绩表插入两条数据
         * 与单向一对多不同的是：setStudent(student),这个步骤就是给外键字段设置值
         */
        StudentResult studentResultA = new StudentResult();
        studentResultA.setResultSubject("语文");
        studentResultA.setResultScore(89.5f);
        resultRepository.saveAndFlush(studentResultA);
        StudentResult studentResultB = new StudentResult();
        studentResultB.setResultSubject("数学");
        studentResultB.setResultScore(92f);
        resultRepository.saveAndFlush(studentResultB);

        // 查询学生(主)表数据
        System.out.println(studentRepository.findById(student.getStudentId()));
        // 查询成绩（从）表数据
        System.out.println(resultRepository.findById(studentResultA.getResultId()));
        System.out.println(resultRepository.findById(studentResultB.getResultId()));
    }
```

4、查看日志

```verilog
// 建表语句跟之前一样
Hibernate: // 插入语句
    insert 
    into
        tb_student
        (student_name, student_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_result
        (result_score, result_subject, student_id) 
    values
        (?, ?, ?)
Hibernate: 
    insert 
    into
        tb_student_result
        (result_score, result_subject, student_id) 
    values
        (?, ?, ?)
Hibernate: // 关联查询语句
    select
        student0_.student_id as student_1_2_0_,
        student0_.student_name as student_2_2_0_,
        student0_.student_pwd as student_3_2_0_,
        studentres1_.student_id as student_4_4_1_,
        studentres1_.result_id as result_i1_4_1_,
        studentres1_.result_id as result_i1_4_2_,
        studentres1_.result_score as result_s2_4_2_,
        studentres1_.result_subject as result_s3_4_2_,
        studentres1_.student_id as student_4_4_2_ 
    from
        tb_student student0_ 
    left outer join
        tb_student_result studentres1_ 
            on student0_.student_id=studentres1_.student_id 
    where
        student0_.student_id=?
Optional[Student{studentId=1, studentName='小明', studentPwd='123456'}]
Hibernate: // 关联查询语句
    select
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_score as result_s2_4_0_,
        studentres0_.result_subject as result_s3_4_0_,
        studentres0_.student_id as student_4_4_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_result studentres0_ 
    left outer join
        tb_student student1_ 
            on studentres0_.student_id=student1_.student_id 
    where
        studentres0_.result_id=?
Hibernate: 
    select
        studentres0_.student_id as student_4_4_0_,
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_id as result_i1_4_1_,
        studentres0_.result_score as result_s2_4_1_,
        studentres0_.result_subject as result_s3_4_1_,
        studentres0_.student_id as student_4_4_1_ 
    from
        tb_student_result studentres0_ 
    where
        studentres0_.student_id=?
Optional[StudentResult{resultId=1, resultSubject='语文', resultScore=89.5}]
Hibernate: // 关联查询语句
    select
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_score as result_s2_4_0_,
        studentres0_.result_subject as result_s3_4_0_,
        studentres0_.student_id as student_4_4_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_result studentres0_ 
    left outer join
        tb_student student1_ 
            on studentres0_.student_id=student1_.student_id 
    where
        studentres0_.result_id=?
Hibernate: 
    select
        studentres0_.student_id as student_4_4_0_,
        studentres0_.result_id as result_i1_4_0_,
        studentres0_.result_id as result_i1_4_1_,
        studentres0_.result_score as result_s2_4_1_,
        studentres0_.result_subject as result_s3_4_1_,
        studentres0_.student_id as student_4_4_1_ 
    from
        tb_student_result studentres0_ 
    where
        studentres0_.student_id=?
Optional[StudentResult{resultId=2, resultSubject='数学', resultScore=92.0}]
```

---

PS 注意1：

双向一对多中，一方/被拥有方/主键表 一般我们是放弃外键维护的，所以想使用单向一对多的方式插入数据，那么外键是会为空的（即：先插入多方/外键表数据，然后插入一方/主键表数据，最后在一方/主键表 内给引用的多方属性设置值。如果是在单向一对多的情况下，最后会有一个update语句修改外键null的值，当时在双向一对多中并不会，最后插入完数据，外键表会为空）

PS 注意2：

双向一对多中，一方/被拥有方/主键表 中我们**去掉放弃外键维护配置**，然后再次使用单向一对多的方式插入数据，会发现外键字段依旧为**null**，当时可以看到日志中显示：JPA给我多创建了一个中间表来维护两个表的主外键。还是把两张表关联起来了。

```java
// 去除放弃外键维护配置，然后再运行一对多中测试代码，我们再查看日志
@OneToMany(/*mappedBy = "student",*/fetch = FetchType.EAGER)
private List<StudentResult> studentResults = new ArrayList<>();
```

```verilog
Hibernate: 
    
    create table tb_student_student_results (
       student_student_id bigint not null,
        student_results_result_id bigint not null
    ) engine=MyISAM
Hibernate: 
    
    alter table tb_student_student_results 
       drop index UK_3ifcd0d9c34kmalbnsk94a3h2
Hibernate: 
    
    alter table tb_student_student_results 
       add constraint UK_3ifcd0d9c34kmalbnsk94a3h2 unique (student_results_result_id)
Hibernate: 
    
    alter table tb_student_result 
       add constraint FK5101rgeygkbcenssvt342o3kg 
       foreign key (student_id) 
       references tb_student (student_id)
Hibernate: 
    
    alter table tb_student_student_results 
       add constraint FKknyiphtyp1nnff3jm5c5ahe3a 
       foreign key (student_results_result_id) 
       references tb_student_result (result_id)
Hibernate: 
    
    alter table tb_student_student_results 
       add constraint FKfpfap07ekhngj6ya5e9sx9ngb 
       foreign key (student_student_id) 
       references tb_student (student_id)
```





## 4、多对多关系映射

### 4.1、准备多对多

1、表结构和需求

```sql
-- 需求:学生表与学生称即表的一对多的关联关系
-- 学生表（主键表）:多方
-- 老师表（主键表）:多方
-- 学生老师表（外键中间表）

-- 学生表结构
+--------------+--------------+------+-----+---------+----------------+
| Field        | Type         | Null | Key | Default | Extra          |
+--------------+--------------+------+-----+---------+----------------+
| student_id   | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| student_name | varchar(255) | YES  |     | NULL    |                |
| student_pwd  | varchar(255) | YES  |     | NULL    |                |
+--------------+--------------+------+-----+---------+----------------+

-- 老师表结构
+--------------+--------------+------+-----+---------+----------------+
| Field        | Type         | Null | Key | Default | Extra          |
+--------------+--------------+------+-----+---------+----------------+
| teacher_id   | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| teacher_name | varchar(255) | YES  |     | NULL    |                |
| teacher_pwd  | varchar(255) | YES  |     | NULL    |                |
+--------------+--------------+------+-----+---------+----------------+

-- 学生老师表结构
+------------+------------+------+-----+---------+-------+
| Field      | Type       | Null | Key | Default | Extra |
+------------+------------+------+-----+---------+-------+
| student_id | bigint(20) | NO   | MUL | NULL    |       |
| teacher_id | bigint(20) | NO   | MUL | NULL    |       |
+------------+------------+------+-----+---------+-------+
```

2、创建2个实体类对应的**Dao**接口

```java
/**
 * 学生表repository接口
 */
public interface StudentRepository extends JpaRepository<Student,Long> {
}
```

```java
/**
 * 老师表repository接口
 */
public interface TeacherRepository extends JpaRepository<Teacher,Long> {
}
```



### 4.2、单向多对多

单向多对多：两张表都为多方，谁都可以为拥有方和被拥有方。这里定义：学生表为拥有方，老师表为被拥有方

1、创建：学生实体类（拥有方）

```java
@Data
@Entity
@Table(name = "tb_student")
public class Student implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long studentId;
    @Column(name = "student_name")
    private String studentName;
    @Column(name = "student_pwd")
    private String studentPwd;

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
     *
     *  注意！注意！注意：
     *  不管是一对多或者多对多：
     *  @OneToMany、@ManyToMany 中的fetch属性默认都是懒加载.
     *  可以理解为查询本实体类中的多方（List/Set）属性时，默认是懒加载
     */
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name="tb_student_teacher",
            joinColumns=@JoinColumn(
                    name="student_id",
                    referencedColumnName = "student_id"),
            inverseJoinColumns=@JoinColumn(
                    name="teacher_id",
                    referencedColumnName = "teacher_id"))
    private List<Teacher> teachers = new ArrayList<>();
}
```

2、创建：老师实体类（被拥有方）（无需配置）

```java
@Data
@Entity
@Table(name = "tb_teacher")
public class Teacher {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "teacher_id")
    private Long teacherId;
    @Column(name = "teacher_name")
    private String teacherName;
    @Column(name = "teacher_pwd")
    private String teacherPwd;
}
```

3、测试代码

```java
    /**
     * 单向 多对多
     * 先插入被拥有端数据，然后插入拥有端数据
     */
    @Test
    public void testManyToMany(){
        // 首先给老师表（被拥有方）增加数据
        Teacher teacherA = new Teacher();
        teacherA.setTeacherName("语文老师");
        teacherA.setTeacherPwd("666666");
        teacherRepository.saveAndFlush(teacherA);
        Teacher teacherB = new Teacher();
        teacherB.setTeacherName("数学老师");
        teacherB.setTeacherPwd("888888");
        teacherRepository.saveAndFlush(teacherB);
        Teacher teacherC = new Teacher();
        teacherC.setTeacherName("英语老师");
        teacherC.setTeacherPwd("888888");
        teacherRepository.saveAndFlush(teacherC);

        // 然后给学生表数据（拥有方）增加数据
        Student studentA = new Student();
        studentA.setStudentName("小明");
        studentA.setStudentPwd("123456");
        // 这步骤设置值就是给中间表插入数据
        studentA.setTeachers(Arrays.asList(teacherA,teacherB));
        studentRepository.saveAndFlush(studentA);
        Student studentB = new Student();
        studentB.setStudentName("小明");
        studentB.setStudentPwd("123456");
        // 这步骤设置值就是给中间表插入数据
        studentB.setTeachers(Arrays.asList(teacherB,teacherC));
        studentRepository.saveAndFlush(studentB);

        // 查询学生表（拥有方）数据
        System.out.println(studentRepository.findById(studentA.getStudentId()));
        System.out.println(studentRepository.findById(studentB.getStudentId()));
    }
```

4、查看日志

```java
// 建立中间表日志
Hibernate: 
    
    alter table tb_student_teacher 
       add constraint FKf3vdcta2bcm9seh723ge2tel 
       foreign key (teacher_id) 
       references tb_teacher (teacher_id)
Hibernate: 
    
    alter table tb_student_teacher 
       add constraint FKkhrh8vhu4pnumefhvuwdiby8y 
       foreign key (student_id) 
       references tb_student (student_id)

// 省略了插入数据语句和查询语句...
Optional[Student(studentId=1, studentName=小明, studentPwd=123456, teachers=[Teacher(teacherId=1, teacherName=语文老师, teacherPwd=666666), Teacher(teacherId=2, teacherName=数学老师, teacherPwd=888888)])]
// 查询语句省略...可以参考上面查询语句，一样的
Optional[Student(studentId=2, studentName=小明, studentPwd=123456, teachers=[Teacher(teacherId=2, teacherName=数学老师, teacherPwd=888888), Teacher(teacherId=3, teacherName=英语老师, teacherPwd=888888)])]
```

```verilog
// 完整的日志
Hibernate: // 插入数据
    insert 
    into
        tb_teacher
        (teacher_name, teacher_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_teacher
        (teacher_name, teacher_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_teacher
        (teacher_name, teacher_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student
        (student_name, student_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_teacher
        (student_id, teacher_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_teacher
        (student_id, teacher_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student
        (student_name, student_pwd) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_teacher
        (student_id, teacher_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        tb_student_teacher
        (student_id, teacher_id) 
    values
        (?, ?)
Hibernate: // 查询语句
    select
        student0_.student_id as student_1_2_0_,
        student0_.student_name as student_2_2_0_,
        student0_.student_pwd as student_3_2_0_,
        teachers1_.student_id as student_1_5_1_,
        teacher2_.teacher_id as teacher_2_5_1_,
        teacher2_.teacher_id as teacher_1_6_2_,
        teacher2_.teacher_name as teacher_2_6_2_,
        teacher2_.teacher_pwd as teacher_3_6_2_ 
    from
        tb_student student0_ 
    left outer join
        tb_student_teacher teachers1_ 
            on student0_.student_id=teachers1_.student_id 
    left outer join
        tb_teacher teacher2_ 
            on teachers1_.teacher_id=teacher2_.teacher_id 
    where
        student0_.student_id=?
Student(studentId=1, studentName=小明, studentPwd=123456, teachers=[Teacher(teacherId=1, teacherName=语文老师, teacherPwd=666666), Teacher(teacherId=2, teacherName=数学老师, teacherPwd=888888)])
2
Hibernate: 
    select
        student0_.student_id as student_1_2_0_,
        student0_.student_name as student_2_2_0_,
        student0_.student_pwd as student_3_2_0_,
        teachers1_.student_id as student_1_5_1_,
        teacher2_.teacher_id as teacher_2_5_1_,
        teacher2_.teacher_id as teacher_1_6_2_,
        teacher2_.teacher_name as teacher_2_6_2_,
        teacher2_.teacher_pwd as teacher_3_6_2_ 
    from
        tb_student student0_ 
    left outer join
        tb_student_teacher teachers1_ 
            on student0_.student_id=teachers1_.student_id 
    left outer join
        tb_teacher teacher2_ 
            on teachers1_.teacher_id=teacher2_.teacher_id 
    where
        student0_.student_id=?
Student(studentId=2, studentName=小明, studentPwd=123456, teachers=[Teacher(teacherId=2, teacherName=数学老师, teacherPwd=888888), Teacher(teacherId=3, teacherName=英语老师, teacherPwd=888888)])
2
Hibernate: 
    select
        teacher0_.teacher_id as teacher_1_6_0_,
        teacher0_.teacher_name as teacher_2_6_0_,
        teacher0_.teacher_pwd as teacher_3_6_0_ 
    from
        tb_teacher teacher0_ 
    where
        teacher0_.teacher_id=?
Teacher(teacherId=2, teacherName=数学老师, teacherPwd=888888)
```



### 4.3、双向多对多

双向多对多：两张表都为多方，谁都可以为拥有方和被拥有方。这里定义：学生表为拥有方，老师表为被拥有方

1、创建：学生实体类（拥有方）（参考单向多对多代码，为了避免死循环重写了ToString方法）

```java
@Data
@ToString(exclude = "teachers")
@Entity
@Table(name = "tb_student")
public class Student implements Serializable {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long studentId;
    @Column(name = "student_name")
    private String studentName;
    @Column(name = "student_pwd")
    private String studentPwd;

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
     *
     *  注意！注意！注意：
     *  不管是一对多或者多对多：
     *  @OneToMany、@ManyToMany 中的fetch属性默认都是懒加载.
     *  可以理解为查询本实体类中的多方（List/Set）属性时，默认是懒加载
     */
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name="tb_student_teacher",
            joinColumns=@JoinColumn(
                    name="student_id",
                    referencedColumnName = "student_id"),
            inverseJoinColumns=@JoinColumn(
                    name="teacher_id",
                    referencedColumnName = "teacher_id"))
    private List<Teacher> teachers = new ArrayList<>();
}
```

2、创建：老师实体类（被拥有方）（增加@ManyToMany和重写ToString方法）

```java
@Data
@Entity
@Table(name = "tb_teacher")
public class Teacher {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "teacher_id")
    private Long teacherId;
    @Column(name = "teacher_name")
    private String teacherName;
    @Column(name = "teacher_pwd")
    private String teacherPwd;

    /**
     * 双向多对多：
     * 需要配置 @ManyToMany：mappedBy + fetch 属性
     * @ManyToMany
     *     mappedBy：放弃维护外键，值为另一方（拥有方）引用本类的属性名
     *     fetch：指定是懒加载还是立即加载（@OneToMany中默认是懒加载）（最好必填）
     */
    @ManyToMany(mappedBy = "teachers",fetch = FetchType.EAGER)
    private List<Student> students = new ArrayList<>();
}
```

3、测试代码

```java
    /**
     * 单向/双向 多对多
     * 先插入被拥有端数据，然后插入拥有端数据
     */
    @Test
    public void testManyToMany(){
        // 首先给老师表（被拥有方）增加数据
        Teacher teacherA = new Teacher();
        teacherA.setTeacherName("语文老师");
        teacherA.setTeacherPwd("666666");
        teacherRepository.saveAndFlush(teacherA);
        Teacher teacherB = new Teacher();
        teacherB.setTeacherName("数学老师");
        teacherB.setTeacherPwd("888888");
        teacherRepository.saveAndFlush(teacherB);
        Teacher teacherC = new Teacher();
        teacherC.setTeacherName("英语老师");
        teacherC.setTeacherPwd("888888");
        teacherRepository.saveAndFlush(teacherC);

        // 然后给学生表数据（拥有方）增加数据
        Student studentA = new Student();
        studentA.setStudentName("小明");
        studentA.setStudentPwd("123456");
        // 这步骤设置值就是给中间表插入数据
        studentA.setTeachers(Arrays.asList(teacherA,teacherB));
        studentRepository.saveAndFlush(studentA);
        Student studentB = new Student();
        studentB.setStudentName("小明");
        studentB.setStudentPwd("123456");
        // 这步骤设置值就是给中间表插入数据
        studentB.setTeachers(Arrays.asList(teacherB,teacherC));
        studentRepository.saveAndFlush(studentB);

        // 查询学生表（拥有方）数据
        Student student1 = studentRepository.findById(studentA.getStudentId()).get();
        System.out.println(student1.getTeachers().size());
        Student student2 = studentRepository.findById(studentB.getStudentId()).get();
        System.out.println(student2.getTeachers().size());

        // 查询老师表（被拥有方）数据
        Teacher teacher = teacherRepository.findById(teacherB.getTeacherId()).get();
        System.out.println(teacher);
        System.out.println(teacher.getStudents().size());
    }
```

4、查看日志

```verilog
Hibernate: 
    select
        student0_.student_id as student_1_2_0_,
        student0_.student_name as student_2_2_0_,
        student0_.student_pwd as student_3_2_0_,
        teachers1_.student_id as student_1_5_1_,
        teacher2_.teacher_id as teacher_2_5_1_,
        teacher2_.teacher_id as teacher_1_6_2_,
        teacher2_.teacher_name as teacher_2_6_2_,
        teacher2_.teacher_pwd as teacher_3_6_2_ 
    from
        tb_student student0_ 
    left outer join
        tb_student_teacher teachers1_ 
            on student0_.student_id=teachers1_.student_id 
    left outer join
        tb_teacher teacher2_ 
            on teachers1_.teacher_id=teacher2_.teacher_id 
    where
        student0_.student_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Hibernate: 
    select
        teachers0_.student_id as student_1_5_0_,
        teachers0_.teacher_id as teacher_2_5_0_,
        teacher1_.teacher_id as teacher_1_6_1_,
        teacher1_.teacher_name as teacher_2_6_1_,
        teacher1_.teacher_pwd as teacher_3_6_1_ 
    from
        tb_student_teacher teachers0_ 
    inner join
        tb_teacher teacher1_ 
            on teachers0_.teacher_id=teacher1_.teacher_id 
    where
        teachers0_.student_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Student{studentId=7, studentName='小明', studentPwd='123456'}
2
Hibernate: 
    select
        student0_.student_id as student_1_2_0_,
        student0_.student_name as student_2_2_0_,
        student0_.student_pwd as student_3_2_0_,
        teachers1_.student_id as student_1_5_1_,
        teacher2_.teacher_id as teacher_2_5_1_,
        teacher2_.teacher_id as teacher_1_6_2_,
        teacher2_.teacher_name as teacher_2_6_2_,
        teacher2_.teacher_pwd as teacher_3_6_2_ 
    from
        tb_student student0_ 
    left outer join
        tb_student_teacher teachers1_ 
            on student0_.student_id=teachers1_.student_id 
    left outer join
        tb_teacher teacher2_ 
            on teachers1_.teacher_id=teacher2_.teacher_id 
    where
        student0_.student_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Hibernate: 
    select
        teachers0_.student_id as student_1_5_0_,
        teachers0_.teacher_id as teacher_2_5_0_,
        teacher1_.teacher_id as teacher_1_6_1_,
        teacher1_.teacher_name as teacher_2_6_1_,
        teacher1_.teacher_pwd as teacher_3_6_1_ 
    from
        tb_student_teacher teachers0_ 
    inner join
        tb_teacher teacher1_ 
            on teachers0_.teacher_id=teacher1_.teacher_id 
    where
        teachers0_.student_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Student{studentId=8, studentName='小明', studentPwd='123456'}
2
Hibernate: 
    select
        teacher0_.teacher_id as teacher_1_6_0_,
        teacher0_.teacher_name as teacher_2_6_0_,
        teacher0_.teacher_pwd as teacher_3_6_0_,
        students1_.teacher_id as teacher_2_5_1_,
        student2_.student_id as student_1_5_1_,
        student2_.student_id as student_1_2_2_,
        student2_.student_name as student_2_2_2_,
        student2_.student_pwd as student_3_2_2_ 
    from
        tb_teacher teacher0_ 
    left outer join
        tb_student_teacher students1_ 
            on teacher0_.teacher_id=students1_.teacher_id 
    left outer join
        tb_student student2_ 
            on students1_.student_id=student2_.student_id 
    where
        teacher0_.teacher_id=?
Hibernate: 
    select
        teachers0_.student_id as student_1_5_0_,
        teachers0_.teacher_id as teacher_2_5_0_,
        teacher1_.teacher_id as teacher_1_6_1_,
        teacher1_.teacher_name as teacher_2_6_1_,
        teacher1_.teacher_pwd as teacher_3_6_1_ 
    from
        tb_student_teacher teachers0_ 
    inner join
        tb_teacher teacher1_ 
            on teachers0_.teacher_id=teacher1_.teacher_id 
    where
        teachers0_.student_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Hibernate: 
    select
        teachers0_.student_id as student_1_5_0_,
        teachers0_.teacher_id as teacher_2_5_0_,
        teacher1_.teacher_id as teacher_1_6_1_,
        teacher1_.teacher_name as teacher_2_6_1_,
        teacher1_.teacher_pwd as teacher_3_6_1_ 
    from
        tb_student_teacher teachers0_ 
    inner join
        tb_teacher teacher1_ 
            on teachers0_.teacher_id=teacher1_.teacher_id 
    where
        teachers0_.student_id=?
Hibernate: 
    select
        students0_.teacher_id as teacher_2_5_0_,
        students0_.student_id as student_1_5_0_,
        student1_.student_id as student_1_2_1_,
        student1_.student_name as student_2_2_1_,
        student1_.student_pwd as student_3_2_1_ 
    from
        tb_student_teacher students0_ 
    inner join
        tb_student student1_ 
            on students0_.student_id=student1_.student_id 
    where
        students0_.teacher_id=?
Teacher{teacherId=11, teacherName='数学老师', teacherPwd='888888'}
2
```



# SpringData JPA 之疑难杂症

## 1、JPA自动建表不生成外键

SpringBoot项目搭配的JPA使用时候，有一对多的关系注解，那么自动会生成外键。外键在有些时候，会导致代码不能走通，我们不想要怎么做。有两种解决方案.

### 1.1、多表关系映射：一对多

情况一：单向一对多/多对一，在@JoinColumn内加上@ForeignKey属性配置即可

```java
@OneToMany
@JoinColumn(name="cid",foreignKey = @ForeignKey(name = "none",value = ConstraintMode.NO_CONSTRAINT))

@ManyToOne
@JoinColumn(name="cid",foreignKey = @ForeignKey(name = "none",value = ConstraintMode.NO_CONSTRAINT))
```



情况二：双向一对多，除了在@JoinColumn加@ForeignKey配置，还需要再另一方（一方）加配置

1.解决方案一：

```java
// 一方加@org.hibernate.annotations.ForeignKey，注意：这个注解被废弃了，所以更新jar包的时候需要注意下
@org.hibernate.annotations.ForeignKey(name = “none”)
@OneToMany

// 多方配置不变
@ManyToOne
@JoinColumn(name="cid",foreignKey = @ForeignKey(name = "none",value = ConstraintMode.NO_CONSTRAINT))
```

2.解决方案二：

```java
// 一方加@Transient，意思是忽略该字段，我认为加上这个字段后在加@OneToMany也无意义了.
@Transient
@OneToMany

// 多方配置不变
@ManyToOne
@JoinColumn(name="cid",foreignKey = @ForeignKey(name = "none",value = ConstraintMode.NO_CONSTRAINT))
```

---

### 2.2、多表关系映射：多对多

多对多也是类似的处理方式：

单向多对多：只要在@JoinTable内的@JoinColumn内加上@ForeignKey属性配置即可，记得连个表外键都要加

```java
@ManyToMany(fetch = FetchType.EAGER)
@JoinTable(name="tb_student_teacher",
           joinColumns=@JoinColumn(
               name="student_id",
               referencedColumnName = "student_id",
               foreignKey = @ForeignKey(name = "none",value = ConstraintMode.NO_CONSTRAINT)),
           inverseJoinColumns=@JoinColumn(
               name="teacher_id",
               referencedColumnName = "teacher_id",
               foreignKey = @ForeignKey(name = "none",value = ConstraintMode.NO_CONSTRAINT)))
```

双向多对多：参照双向一对多/多对一，在另一方加注解即可。



## 2、关于实体对象查询异常

```verilog
WARNING: Please consider reporting this to the maintainers of org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
Hibernate: sql语句
java.lang.StackOverflowError
    at java.base/java.lang.AbstractStringBuilder.inflate(AbstractStringBuilder.java:202)
    at java.base/java.lang.AbstractStringBuilder.putStringAt(AbstractStringBuilder.java:1639)
    at java.base/java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:513)
　　 .......
```

关以这种错误大多都是编写的`toString`语句发生了错误，大家在写`toString`的时候一定要分清主表和从表，在编写主表的toString的时候一定要去除外键字段，而在编写从表的时候一定要加上外键字段，因为平时我们都是通过从表查询数据（因为从表有指向主表的外键），这样可以把主表的数据通过外键查询出来，但是主表上如果也有从表的字段的话就会一只循环，数据没完没了，所有抛异常也是对的，

**总结一句话：从表有主表的字段，主表有从表的字段，打印从表顺带打印主表，但是主表里面还有从表字段，然后继续打印从表字段.......**

解决方案：

1. 去除 @Data 注解，替换成 @Getter 和 @Setter 注解，使用@ToString(exclude = "引用类型字段")
2. 也可以重写`toString()`方法，把引用类型的输出删掉即可
3. 查询后可以不要输出打印该对象，这样就不会死循环输出对象中的对象了（治标不治本, 不推荐）
4. ~~在实体类上增加`@JsonIgnoreProperties(value = "对方类引用本类的属性名")`（我暂时试了是无效的，打印对象还是会报错）~~



## 3、关于延迟加载事务问题

```verver
org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: cn.xw.domain.Teacher.students, could not initialize proxy - no Session 
    at org.hibernate.collection.internal.AbstractPersistentCollection.throwLazyInitializationException(AbstractPersistentCollection.java:606)
    at org.hibernate.collection.internal.AbstractPersistentCollection.withTemporarySessionIfNeeded(AbstractPersistentCollection.java:218)
    at org.hibernate.collection.internal.AbstractPersistentCollection.initialize(AbstractPersistentCollection.java:585)
    at org.hibernate.collection.internal.AbstractPersistentCollection.read(AbstractPersistentCollection.java:149)
    at org.hibernate.collection.internal.PersistentSet.toString(PersistentSet.java:327)
    at java.base/java.lang.String.valueOf(String.java:2801)
    at java.base/java.lang.StringBuilder.append(StringBuilder.java:135)
```

关于在使用延迟加载的时候，在当前的方法上必须设置@Transactional，因为在使用延迟加载底层已经使用了事务的相关方法



## 4、关于懒加载no session异常

```console
org.hibernate.LazyInitializationException: failed to lazily initialize XXXXX could not initialize proxy - no Session
```

仔细看英文，懒加载存在，session 都关了，获取不到数据。

解决办法：lazy=“false”。若是用注解生成的。fetch = FetchType.EAGER

```properties
# 这个配置的意思就是在没有事务的情况下允许懒加载（可以理解为开启懒加载，实际上默认是开启的）
spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true
```

当然了，这样的话，你的缓存就不起作用了。另外解决办法，获取数据时，使用用 left join fetch 或 inner join fetch 语法。



## 5、关于数据库方言问题（Dialect）

```console
6月 03, 2020 4:57:00 下午 org.hibernate.dialect.Dialect <init>
INFO: HHH000400: Using dialect: org.hibernate.dialect.MySQLDialect
省去部分...
Hibernate: create table family (...) type=MyISAM
//上面一局为我们创建的是一张表并设置MyISAM引擎  错误就在这 无法运行了
6月 03, 2020 4:57:01 下午 org.hibernate.tool.schema.internal.ExceptionHandlerLoggedImpl handleException
WARN: GenerationTarget encountered exception accepting command : Error executing DDL "create table family (...) type=MyISAM" via JDBC Statement
org.hibernate.tool.schema.spi.CommandAcceptanceException: Error executing DDL "create table family (...) type=MyISAM" via JDBC Statement
//从上面错误可以看出 程序运行的时候默认的数据库方言设置了 org.hibernate.dialect.MySQLDialect  而这个默认是MyISAM引擎
```

问题所在：因为我导入的hibernate坐标是5.4.10.Final，在导入这类高版本的坐标往往要为数据库方言设置MySQL5InnoDBDialect的配置，在我前面也测试了，关于坐标版本问题，发现5.0.x.Final左右的版本不用设置数据库方言，默认即可。

```properties
# 设置数据库方言,建表使用MyISAM，默认是InnoDB
spring.jpa.database-platform=org.hibernate.dialect.MySQL5Dialect
```

具体的版本在创建数据库表的时候会抛各种异常，这里我整理了一下数据库方言，方便大家参考

```
数据库方言
DB2                         org.hibernate.dialect.DB2Dialect
DB2 AS/400                  org.hibernate.dialect.DB2400Dialect
DB2 OS390                   org.hibernate.dialect.DB2390Dialect
PostgreSQL                  org.hibernate.dialect.PostgreSQLDialect
MySQL                       org.hibernate.dialect.MySQLDialect
MySQL with InnoDB           org.hibernate.dialect.MySQLInnoDBDialect
MySQL with MyISAM           org.hibernate.dialect.MySQLMyISAMDialect
Oracle (any version)        org.hibernate.dialect.OracleDialect
Oracle 9i/10g               org.hibernate.dialect.Oracle9Dialect
Sybase                      org.hibernate.dialect.SybaseDialect
Sybase Anywhere             org.hibernate.dialect.SybaseAnywhereDialect
Microsoft SQL Server        org.hibernate.dialect.SQLServerDialect
SAP DB                      org.hibernate.dialect.SAPDBDialect
Informix                    org.hibernate.dialect.InformixDialect
HypersonicSQL               org.hibernate.dialect.HSQLDialect
Ingres                      org.hibernate.dialect.IngresDialect
Progress                    org.hibernate.dialect.ProgressDialect
Mckoi SQL                   org.hibernate.dialect.MckoiDialect
Interbase                   org.hibernate.dialect.InterbaseDialect
Pointbase                   org.hibernate.dialect.PointbaseDialect
FrontBase                   org.hibernate.dialect.FrontbaseDialect
Firebird                    org.hibernate.dialect.FirebirdDialect
```



## 6、Executing an update/delete query

```verilog
Caused by: javax.persistence.TransactionRequiredException: Executing an update/delete query
```

如果代码执行报如上错误，是由于在执行了更新删除操作没有加上事物，我们只需要加上@Transactional即可。不加会报错。

注意：有一种情况除外，就是在做单元测试的时候，加了@Transactiona也不起作用，必须加上@Transactiona + @Rollback(false)，具体缘由参考下篇文章（SpringBoot Junit事物自动回滚）



## 7、SpringBoot Junit事物自动回滚

**1、问题**：在Spring单元测试类中的更新方法测试成功通过并没有报错，但数据库没有插入数据。

测试代码如下（执行测试方法后，数据库没有插入数据）：

```java
@Test
@Transactional
public void save() {
    User user = new User();
    user.setUsername("admin");
    user.setPassword("password");
    userRepository.save(user); // userRepository继承了JpaRepository
}
```

输出日志：

```v
2022-01-27 15:39:29.022  INFO 6964 --- [           main] o.s.t.c.transaction.TransactionContext   : Began transaction (1) for test context [DefaultTestContext@301ec38b testClass = DemoApplicationTests, testInstance = com.example.demo.DemoApplicationTests@57ea113a, testMethod = save@DemoApplicationTests, testException = [null], mergedContextConfiguration = [MergedContextConfiguration@17a1e4ca testClass = DemoApplicationTests, locations = '{}', classes = '{class com.example.demo.DemoApplication}', contextInitializerClasses = '[]', activeProfiles = '{}', propertySourceLocations = '{}', propertySourceProperties = '{org.springframework.boot.test.context.SpringBootTestContextBootstrapper=true}', contextCustomizers = set[org.springframework.boot.test.context.filter.ExcludeFilterContextCustomizer@194fad1, org.springframework.boot.test.json.DuplicateJsonObjectContextCustomizerFactory$DuplicateJsonObjectContextCustomizer@65f8f5ae, org.springframework.boot.test.mock.mockito.MockitoContextCustomizer@0, org.springframework.boot.test.web.client.TestRestTemplateContextCustomizer@4d15107f, org.springframework.boot.test.autoconfigure.actuate.metrics.MetricsExportContextCustomizerFactory$DisableMetricExportContextCustomizer@5a5a729f, org.springframework.boot.test.autoconfigure.properties.PropertyMappingContextCustomizer@0, org.springframework.boot.test.autoconfigure.web.servlet.WebDriverContextCustomizerFactory$Customizer@71687585, org.springframework.boot.test.context.SpringBootTestArgs@1, org.springframework.boot.test.context.SpringBootTestWebEnvironment@1e7c7811], contextLoader = 'org.springframework.boot.test.context.SpringBootContextLoader', parent = [null]], attributes = map['org.springframework.test.context.event.ApplicationEventsTestExecutionListener.recordApplicationEvents' -> false]]; transaction manager [org.springframework.orm.jpa.JpaTransactionManager@4348fa35]; rollback [true]
Hibernate: 
    insert 
    into
        sys_user
        (id, password, username) 
    values
        (null, ?, ?)
2022-01-27 15:39:29.261  INFO 6964 --- [           main] o.s.t.c.transaction.TransactionContext   : Rolled back transaction for test: [DefaultTestContext@301ec38b testClass = DemoApplicationTests, testInstance = com.example.demo.DemoApplicationTests@57ea113a, testMethod = save@DemoApplicationTests, testException = [null], mergedContextConfiguration = [MergedContextConfiguration@17a1e4ca testClass = DemoApplicationTests, locations = '{}', classes = '{class com.example.demo.DemoApplication}', contextInitializerClasses = '[]', activeProfiles = '{}', propertySourceLocations = '{}', propertySourceProperties = '{org.springframework.boot.test.context.SpringBootTestContextBootstrapper=true}', contextCustomizers = set[org.springframework.boot.test.context.filter.ExcludeFilterContextCustomizer@194fad1, org.springframework.boot.test.json.DuplicateJsonObjectContextCustomizerFactory$DuplicateJsonObjectContextCustomizer@65f8f5ae, org.springframework.boot.test.mock.mockito.MockitoContextCustomizer@0, org.springframework.boot.test.web.client.TestRestTemplateContextCustomizer@4d15107f, org.springframework.boot.test.autoconfigure.actuate.metrics.MetricsExportContextCustomizerFactory$DisableMetricExportContextCustomizer@5a5a729f, org.springframework.boot.test.autoconfigure.properties.PropertyMappingContextCustomizer@0, org.springframework.boot.test.autoconfigure.web.servlet.WebDriverContextCustomizerFactory$Customizer@71687585, org.springframework.boot.test.context.SpringBootTestArgs@1, org.springframework.boot.test.context.SpringBootTestWebEnvironment@1e7c7811], contextLoader = 'org.springframework.boot.test.context.SpringBootContextLoader', parent = [null]], attributes = map['org.springframework.test.context.event.ApplicationEventsTestExecutionListener.recordApplicationEvents' -> false]]
```



**2、原因：**SpringBoot使用Junit编写单元测试，@Transactional默认是事物回滚的，这样测试的脏数据不影响数据库。具体看控制台输出关键日志部分：

```verilog
Began transaction (1) for test context ....
Rolled back transaction for test: ....
```



**3、解决方法：**测试方法加上注解 @Rollback(true)，关闭自动事物回滚。

```java
@Test
@Transactional
@Rollback(false)
public void save() {
    User user = new User();
    user.setUsername("admin");
    user.setPassword("password");
    userRepository.save(user); // userRepository继承了JpaRepository
}
```

查看关键日志部分：

```verilog
Began transaction (1) for test context ...
Committed transaction for test: ...
```



> **Spring Boot Junit事物自动回滚总结：**
>
> 因为用了JPA配合Hibernate ，采用注解默认是开启了LayzLoad也就是懒加载，所以不得不在Junit的单元测试上加上@Transactional注解，这样Spring会自动为当前线程开启Session，这样在单元测试里面懒加载才不会因为访问完Repo之后，出现：`session not found.`，但是单元测试里面如果加上了@Transactional会自动回滚事务，需要在单元测试上面加上@Rollback(false)



## 8、对象属性设置值会自动更新数据库

> 使用Spring Data JPA获取的对象，事物开启时，其属性变更后自动更新数据库问题排查与解决方案
>
> 参考文献 & 鸣谢：转-Spring Data JPA中对象属性自动更新数据库：https://www.cnblogs.com/east7/p/14453910.html

**问题描述**：

使用继承了JpaRepository的接口从数据库中获取到某个对象，然后操作这个对象的set属性值时，新值会直接更新到数据库。例如，UserRepository继承了JpaRepository，从数据库查询出User类实例user，当对user执行set属性修改时，新值会直接更新到数据库。

问题代码：（PS：出现这种情况有个前提条件，就是当前方法开启了事物）

```java
@Test
@Transactional
@Rollback(false) // 由于是在junit中测试，所以必须取消自动回滚
public void testUpdate(){
    // 持久化状态，当前方法加了事物，SpringDataJPA在事物完成的时候，会自动提交修改
    User user = this.userRepository.findById(1L).get();
    user.setUsername("王小小");
}
```

输出日志：

```verilog
Hibernate: 
    select
        user0_.id as id1_0_0_,
        user0_.password as password2_0_0_,
        user0_.username as username3_0_0_ 
    from
        sys_user user0_ 
    where
        user0_.id=?
Hibernate: 
    update
        sys_user 
    set
        password=?,
        username=? 
    where
        id=?
```

可以发现查询完对象后，在给对象设置值后，就自动更新到数据库了



**问题分析**：

由于方法上开启了事物，发现事务结束时会把属性发生改变的user对象更新到数据库，在控制台打印的日志中也可以看见相关update sql语句。百思不得其解，为什么从数据库中获取到的对象属性发生变更的时候，数据库就自动执行更新操作呢？原来是JPA的自带特性在作怪：Hibernate对象有四种状态，【**持久态对象的属性被修改后，在数据库事务提交的时候，会更新数据库**。】



**回顾实体对象的四种状态以及转换关系**：

Spring Data JPA 可以理解为 是对 JPA 规范的二次封装和抽象，底层还是使用了 Hibernate 的 JPA 技术实现。因此，还是要不辞劳苦地翻阅Hibernate文档。最新的Hibernate文档中为Hibernate对象定义了四种状态（原来是三种状态，面试的时候基本上问的也是三种状态），分别是：瞬时态（new, or transient）、持久态（managed, or persistent）、游状态（detached）和移除态（removed，以前Hibernate文档中定义的三种状态中没有移除态），如下图所示，就以前的Hibernate文档中移除态被视为是瞬时态。

![img](https://images2015.cnblogs.com/blog/884874/201602/884874-20160225171547411-369659608.png)

- 瞬时态：当new一个实体对象后，这个对象处于瞬时态，即这个对象只是一个保存临时数据的内存区域，如果没有变量引用这个对象，则会被JVM的垃圾回收机制回收。这个对象所保存的数据与数据库没有任何关系，除非通过Session的save()、saveOrUpdate()、persist()、merge()方法把瞬时态对象与数据库关联，并把数据插入或者更新到数据库，这个对象才转换为持久态对象。
- 持久态：持久态对象的实例在数据库中有对应的记录，并拥有一个持久化标识（ID）。对持久态对象进行delete操作后，数据库中对应的记录将被删除，那么持久态对象与数据库记录不再存在对应关系，持久态对象变成移除态（可以视为瞬时态）。持久态对象被修改变更后，不会马上同步到数据库，直到数据库事务提交。
- 游离态：当Session进行了close()、clear()、evict()或flush()后，实体对象从持久态变成游离态，对象虽然拥有持久和与数据库对应记录一致的标识值，但是因为对象已经从会话中清除掉，对象不在持久化管理之内，所以处于游离态（也叫脱管态）。游离态的对象与临时状态对象是十分相似的，只是它还含有持久化标识。
- 移除态：当调用EntityManger对实体进行delete后，该实体对象就处于删除状态。其本质也就是一个瞬时状态的对象



**问题解决**：

1、更新的方法不开启事物处理，然后使用save()等方法更新操作即可（缺点：没有事物无法回滚，不能保证数据一致性）

```java
@Test
public void testUpdate(){
    // 持久化状态的，不过没有加事物控制
    User user = this.userCrudRepository.findById(1L).get();
    user.setUsername("王小红");
    this.userCrudRepository.save(user);
}
```

2、因持久层框架JPA自身的机制，会在事务提交后将持久状态的对象自动更新到数据库，为了避免自动更新，我们可以创建一个对象，设置好属性后再调用save()方法更新数据（缺点：比较繁琐，代码也会变得不够简洁）

```java
@Test
@Transactional
public void testUpdate(){
    // 持久化状态的，不过没有加事物控制
    User user = this.userCrudRepository.findById(1L).get();
    User userNew = new User();
    userNew.setId(user.getId)
    userNew.setUsername("admin");
    userNew.setPassword(user.getPassword);
    this.userCrudRepository.save(userNew);
}
```

3、把持久态的对象变成游离态即可，有如下三种方法（缺点也是需要注入EntityManager）

1. close() 方法：关闭session可以，但是若后面还要用session这个方法就不太好了
2. clear() 方法：将session中所有的对象全部清除缓存
3. evict() 方法：将某一个对象清楚缓存session（推荐）

```java
@PersistenceContext
private EntityManager entityManager;

@Test
@Transactional
public void testUpdate(){
    // 持久化状态的，不过没有加事物控制
    User user = this.userCrudRepository.findById(1L).get();
    // 检查对象是否是持久化态
    if (entityManager.contains(user)) {
        // 获取session
        Session session = entityManager.unwrap(org.hibernate.Session.class);
        // 转换成游离态
        session.evict(entry);
    }
    user.setUsername("王小红");
    this.userCrudRepository.save(user);
}
```



# 参考文献 & 鸣谢

1. CSDN（C语言中文社区）十分钟入门 JPA：https://blog.csdn.net/csdn_wangchong/article/details/103095115
2. CSDN（詠聖wK）JPA注解大全详解参考手册：https://blog.csdn.net/yswKnight/article/details/79257372
3. CSDN（入秋枫叶）JPA注释详解参考手册：https://blog.csdn.net/yiyelanxin/article/details/100107335
4. CSDN（风情客家）JPA常用注解：https://blog.csdn.net/justlpf/article/details/84956212
5. CSDN（星仔说）SpringBoot集成JPA一篇就够了 超详细：https://blog.csdn.net/cmx1060220219/article/details/106259423
6. 博客园（如莲家园）Spring Data JPA Tag：https://www.cnblogs.com/rulian/tag/jpa/
7. 博客园（蚂蚁小哥，其他博文也挺好）Spring Data JPA入门及深入（重点关注）：https://www.cnblogs.com/antLaddie/p/12996320.html
8. 博客园（路迢迢）Spring Data JPA 基本使用（重点关注）：https://www.cnblogs.com/chenglc/p/11226693.html
9. CSDN（布道）JPA之Spring Data JPA（重点关注）：https://blog.csdn.net/alex_xfboy/article/details/82907517

JPA之Criteria查询：第一种：通过JPA的Criteria API实现。第二种：DAO层实现 JpaSpecificationExecutor 接口

1. 博客园（My Sunshine）JPA 使用 Specification 复杂查询和 Criteria 查询：https://blog.wuwii.com/jpa-specification.html
2. CSDN（天涯泪小武）JPA 复杂条件查询Group By等等：https://tianyalei.blog.csdn.net/article/details/90704969
3. CNDN（流烟默）JpaSpecificationExecutor复杂动态查询实例：https://blog.csdn.net/J080624/article/details/84581231
4. 博客园，JPA criteria 查询:类型安全与面向对象：https://www.cnblogs.com/JAYIT/p/6972144.html

JPA关系映射详情：

1. https://blog.csdn.net/qq_42315782/article/details/103280330
2. https://blog.csdn.net/u012523359/article/details/76732771
3. https://blog.csdn.net/petercnmei/article/details/54893622
4. https://blog.csdn.net/qq_39256374/article/details/107949709
5. https://blog.csdn.net/qq_36135928/article/details/82081588

Chapter 4 - JPA Queries (JPQL / Criteria) (objectdb.com)：https://www.objectdb.com/java/jpa/query

