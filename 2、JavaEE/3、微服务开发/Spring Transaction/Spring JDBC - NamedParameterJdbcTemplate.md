# ·一、什么是 NamedParameterJdbcTemplate

> NamedParameterJdbcTemplate 类拓展了 JdbcTemplate 类，对 JdbcTemplate 类进行了封装从而支持**具名参数特性**。
>

什么是具名参数？SQL 按名称(以冒号开头)⽽不是按位置进⾏指定。

例如：

- 传统的jdbcTemplate的SQL语句定义，通过问号进行占位，如下：

```java
private final String QUERY_SQL = "INSERT INTO T_USER(username,password) VALUES(?,?)";
```

- NamedParameterJdbcTemplate的吗具名参数定义SQL语句：

```java
private final String QUERY_SQL = "INSERT INTO T_USER(username,password) VALUES(:username,:password)";
```

> NamedParameterJdbcTemplate 主要提供以下三类方法：execute 方法、query 及 queryForXXX 方法、update 及 batchUpdate 方法。

开发建议：

1. 开发中尽量使用NamedParameterJdbcTemplate代替JdbcTemplate，如果想使用JdbcTemplate，也可以通过NamedParameterJdbcTemplate#getJdbcOperations()获取。
2. 不建议使用查询结构为Map的API。

# 二、环境搭建

## 1、创建工程

使用 IDEA 创建项目工程。

## 2、添加Maven依赖

```xml
<!-- jdbc依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<!-- 连接数据库依赖 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- lombok依赖 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

**另外：JDBC 依赖也可以换成**

```xml
<!-- jdbcTemplate 依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.9.RELEASE</version>
</dependency>
```



## 3、yaml 配置

```yaml
#MySql8.0
spring:
  datasource:
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/数据库名?useSSL=false&characterEncoding=utf-8&useUnicode=true&serverTimezone=Asia/Shanghai
    username: 数据库用户名
    password: 数据库密码
```



## 4、数据库结构

注意：数据库的`id主键设置成自动递增`，所以添加的时候不用考虑id

| id   | username | password |
| ---- | -------- | -------- |
| 1    | admin    | admin    |
| 2    | abc      | 123      |
| 3    | abcd     | 1234     |
| 4    | abdce    | 12345    |
| 5    | 刘德华   | 111111   |
| 6    | 郭富城   | 222222   |
| 7    | 张学友   | 333333   |
| 8    | 黎明     | 444444   |



## 5、JavaBean 实体

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class TUser implements Serializable {
    private Integer id;
    private String username;
    private String password;
}
```



# 三、实际应用

## 1、支持的类

### 1、SqlParameterSource

可以使用`SqlParameterSource`实现具名参数，默认实现有 ：

1. **MapSqlParameterSource**：只是封装了`java.util.Map`
2. **BeanPropertySqlParameterSource**：封装了一个JavaBean对象，通过JavaBean对象属性设置具名参数值
3. **EmptySqlParameterSource**：一个空的SqlParameterSource ，占位使用。



### 2、RowMapper

这个接口为了实现SQL查询结果和对象间的转换，可以自己实现，也可以使用系统实现，主要实现类有：

1. **SingleColumnRowMapper**：SQL结果为一个单列的数据，如List 等
2. **BeanPropertyRowMapper**：SQL结果匹配到对象 List< XxxVO > , XxxV



## 2、增删改方法 update

### 1、Map 作为参数

API 如下：

```java
int update(String sql, Map<String, ?> paramMap);
```

```java
@Autowired
private NamedParameterJdbcTemplate template;

// 添加数据
Map<String,Object> map = new HashMap<>();
map.put("username", "刘亦菲");
map.put("password", "4983ghh");
template.update("INSERT INTO T_USER(username,password) VALUES(:username,:password)",map);

// 修改数据
Map<String,Object> map = new HashMap<>();
map.put("username", "刘诗诗");
map.put("password"," ewgg");
map.put("id",4);
template.update("UPDATE T_USER SET USERNAME = :username,PASSWORD = :password WHERE ID = :id",map);

// 删除数据
Map<String,Object> map = new HashMap<>();
map.put("id", 4);
template.update("DELETE FROM T_USER WHERE ID = :id",map);
```



### 2、BeanPropertySqlParameterSource 作为参数

API 如下：

```java
int update(String sql, SqlParameterSource paramSource);
```

1、创建参数 DTO

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ConditionDTO {
    private String password;
    private String username;
}
```

2、给DTO属性传入数据并作为参数传入

```java
ConditionDTO conditionDTO = new ConditionDTO();
conditionDTO.setUsername("成龙");
conditionDTO.setPassword("2432tgh");

BeanPropertySqlParameterSource beanParam = new BeanPropertySqlParameterSource(conditionDTO);
template.update("INSERT INTO T_USER(username,password) VALUES(:username,:password)",beanParam);
```

***注意：用BeanPropertySqlParameterSource作为参数的时候，可以将参数DTO属性存值单独出来一个方法，这样可以解耦，代码维护相对轻松。如下：***

```java
private ConditionDTO getConditionDTO() {
    ConditionDTO conditionDTO = new ConditionDTO();
    conditionDTO.setUsername("成龙");
    conditionDTO.setPassword("2432tgh");
    return conditionDTO;
}

BeanPropertySqlParameterSource beanParam = new BeanPropertySqlParameterSource(getConditionDTO());
template.update("INSERT INTO T_USER(username,password) VALUES(:username,:password)",beanParam);
```



### 3、MapSqlParameterSource 作为参数

API 如下：

```java
int update(String sql, SqlParameterSource paramSource);
```

```java
MapSqlParameterSource mapSql = new MapSqlParameterSource();
mapSql.addValue("username","李连杰")
    .addValue("password","huewrgowrei");
template.update("INSERT INTO T_USER(username,password) VALUES(:username,:password)",mapSql);
```



## 3、查询方法

### 1、返回单行单列数据

注意：单行单列不是单独的一行或者单独的一列数据，单列单行是只有一个数据。如下：

| username |
| -------- |
| admin    |

API 如下：

```java
public <T> T queryForObject(String sql, Map<String, ?> paramMap, Class<T> requiredType);
public <T> T queryForObject(String sql, SqlParameterSource paramSource, Class<T> requiredType);
```

```java
Integer count = template.queryForObject("SELECT COUNT(*) FROM T_USER", new HashMap<>(), Integer.class);
```

使用`EmptySqlParameterSource`：

```java
String username = template.queryForObject( "SELECT USERNAME FROM T_USER WHERE ID = 4", 
                                          EmptySqlParameterSource.INSTANCE, String.class); 
```



### 2、返回多行单列数据

多行单列形式：

| username |
| -------- |
| admin    |
| abc      |
| abcd     |
| abdce    |
| 刘德华   |
| 郭富城   |
| 张学友   |
| 黎明     |

API 如下：

```java
public <T> List<T> queryForList(String sql, Map<String, ?> paramMap, Class<T> elementType);
public <T> List< T> queryForList(String sql, SqlParameterSource paramSource, Class<T> elementType);
```

```java
List<String> names = template.queryForList("SELECT USERNAME FROM T_USER", new HashMap<>(), String.class);
names.forEach(System.out::println);
```

控制台打印结果：

```
admin
abc
abcd
abcde
刘德华
郭富城
张学友
黎明
```



### 3、返回单行数据

解释：数据库中的一行数据对应的就是Java中的一个Bean实体

```java
public <T> T queryForObject(String sql, Map<String, ?> paramMap, RowMapper<T> rowMapper);
public <T> T queryForObject(String sql, SqlParameterSource paramSource, RowMapper<T> rowMapper);
```

```java
// BeanPropertyRowMapper会把下划线转化为驼峰属性
TUser user = template.queryForObject("SELECT * FROM T_USER LIMIT 1", 
                                     new HashMap<>(), 
                                     new BeanPropertyRowMapper<TUser>(TUser.class));
System.out.println("查询出来的user是："+user);
```



### 4、返回单行数据（Map）

```java
public Map<String, Object> queryForMap(String sql, Map<String, ?> paramMap);
public Map<String, Object> queryForMap(String sql, SqlParameterSource paramSource);
```

```java
@RequestMapping("/querySingleTwo")
public Map testQueryForObjectTwo() {
    Map< String, Object> userMap = template.queryForMap("SELECT * FROM T_USER LIMIT 1", new HashMap<>());
    System.out.println(userMap);
    return userMap;
}
```

API 返回结果：

```json
{
    "id": 1,
    "username": "admin",
    "password": "admin"
}
```



### 5、返回多行数据

```java
public <T> List<T> query(String sql, Map<String, ?> paramMap, RowMapper<T> rowMapper);
public <T> List<T> query(String sql, SqlParameterSource paramSource, RowMapper<T> rowMapper);
public <T> List<T> query(String sql, RowMapper<T> rowMapper);
```

```java
@RequestMapping("/querySingleTwo")
public List< TUser> testQueryForObjectTwo() {
    List< TUser> userList = template.query(
        "SELECT * FROM T_USER",
        new BeanPropertyRowMapper<>(TUser.class)
    );
    return userList;
}
```

API 返回结果：

```json
[
    {
        "id": 1,
        "username": "admin",
        "password": "admin"
    },
    {
        "id": 2,
        "username": "abc",
        "password": "123"
    },
    {
        "id": 3,
        "username": "abcd",
        "password": "1234"
    },
    {
        "id": 4,
        "username": "abcde",
        "password": "12345"
    },
    {
        "id": 5,
        "username": "刘德华",
        "password": "111111"
    },
    {
        "id": 6,
        "username": "郭富城",
        "password": "22222"
    },
    {
        "id": 7,
        "username": "张学友",
        "password": "333333"
    },
    {
        "id": 8,
        "username": "黎明",
        "password": "444444"
    }
]
```



### 6、返回多行数据（Map）

API 如下：

```java
public List<Map<String, Object>> queryForList(String sql, Map<String, ?> paramMap);
public List<Map<String, Object>> queryForList(String sql, SqlParameterSource paramSource);
```

```java
@RequestMapping("/querySingleTwo")
public  List<Map<String, Object>> testQueryForObjectTwo() {
    List<Map<String, Object>> mapList = template.queryForList(
        "SELECT * FROM T_USER", new HashMap<>());
    return mapList;
}
```

API 返回结果：

```json
[
    {
        "id": 1,
        "username": "admin",
        "password": "admin"
    },
    {
        "id": 2,
        "username": "abc",
        "password": "123"
    },
    {
        "id": 3,
        "username": "abcd",
        "password": "1234"
    },
    {
        "id": 4,
        "username": "abcde",
        "password": "12345"
    },
    {
        "id": 5,
        "username": "刘德华",
        "password": "111111"
    },
    {
        "id": 6,
        "username": "郭富城",
        "password": "22222"
    },
    {
        "id": 7,
        "username": "张学友",
        "password": "333333"
    },
    {
        "id": 8,
        "username": "黎明",
        "password": "444444"
    }
]
```



## 4、获取新增的主键

`NamedParameterJdbcTemplate`还新增了`KeyHolder`类，使⽤它我们可以获得主键，类似`Mybatis`中的`useGeneratedKeys`。

```java
@RequestMapping("/querySingleTwo")
public int testQueryForObjectTwo() {
    String sql = "INSERT INTO T_USER(username,password) VALUES(:username,:password)";
    ConditionDTO conditionDTO = new ConditionDTO("r43g", "呼呼");
    SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(conditionDTO);
    KeyHolder keyHolder = new GeneratedKeyHolder();
    template.update(sql, sqlParameterSource, keyHolder);
    int k = keyHolder.getKey().intValue();
    System.out.println("主键值是："+k);
    return k;
}
```

返回结果就是新增的主键。



## 5、批量操作 batchUpdate

**方法源码：**

```java
@Override
public int[] batchUpdate(String sql, Map<String, ?>[] batchValues) {
    return batchUpdate(sql, SqlParameterSourceUtils.createBatch(batchValues));
}

@Override
public int[] batchUpdate(String sql, SqlParameterSource[] batchArgs) {
    if (batchArgs.length == 0) {
        return new int[0];
    }

    ParsedSql parsedSql = getParsedSql(sql);
    PreparedStatementCreatorFactory pscf = getPreparedStatementCreatorFactory(parsedSql, batchArgs[0]);

    return getJdbcOperations().batchUpdate(
        pscf.getSql(),
        new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                Object[] values = NamedParameterUtils.buildValueArray(parsedSql, batchArgs[i], null);
                pscf.newPreparedStatementSetter(values).setValues(ps);
            }
            @Override
            public int getBatchSize() {
                return batchArgs.length;
            }
        });
}
```



### 1、批量新增

这里以`SqlParameterSource[]`参数为例

```java
List<TUser> list = new ArrayList<>();
TUser tUser1 = new TUser();
tUser1.setPassword("t7493857vd");
tUser1.setUsername("给会儿");

TUser tUser2 = new TUser();
tUser2.setPassword("erwghrthey");
tUser2.setUsername("国瑞");

list.add(tUser1);
list.add(tUser2);

SqlParameterSource[] batch = SqlParameterSourceUtils.createBatch(list.toArray());
template.batchUpdate("INSERT INTO T_USER(username,password) VALUES(:username,:password)", batch);
```



### 2、批量更新

这里以`SqlParameterSource[]`参数为例

```java
List<TUser> list = new ArrayList<>();
TUser tUser1 = new TUser();
tUser1.setPassword("eryg");
tUser1.setUsername("个人谈话人");
tUser1.setId(109);

TUser tUser2 = new TUser();
tUser2.setPassword("我二哥");
tUser2.setUsername("个羊肉汤");
tUser1.setId(110);

TUser tUser3 = new TUser();
tUser3.setPassword("gerhr6");
tUser3.setUsername("反倒是规范");
tUser3.setId(111);

list.add(tUser1);
list.add(tUser2);
list.add(tUser3);

SqlParameterSource[] batch = SqlParameterSourceUtils.createBatch(list.toArray());
template.batchUpdate("UPDATE T_USER SET USERNAME = :username,PASSWORD = :password WHERE ID = :id", batch);
```



### 3、批量删除

这里以`SqlParameterSource[]`参数为例

```java
List<TUser> list = new ArrayList<>();
TUser tUser1 = new TUser();
tUser1.setId(109);

TUser tUser2 = new TUser();
tUser2.setId(112);

TUser tUser3 = new TUser();
tUser3.setId(113);

list.add(tUser1);
list.add(tUser2);
list.add(tUser3);

SqlParameterSource[] batch = SqlParameterSourceUtils.createBatch(list.toArray());
template.batchUpdate("DELETE FROM T_USER WHERE ID = :id", batch);
```



# 四、深入 SqlParameterSource 实现具名参数查询

注意：使用`queryForList()与queryForObject()`这两个方法一般情况下只返回`单一列`的数据，不能返回复杂的数据对象。本文只介绍通过`SqlParameterSource`具名参数返回复杂数据对象。

## 1、数据库准备

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : my_project
 Source Server Type    : MySQL
 Source Server Version : 80027
 Source Host           : localhost:3306
 Source Schema         : order

 Target Server Type    : MySQL
 Target Server Version : 80027
 File Encoding         : 65001

 Date: 12/12/2022 23:00:57
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for ordertb
-- ----------------------------
DROP TABLE IF EXISTS `ordertb`;
CREATE TABLE `ordertb`  (
    `orderID` int NOT NULL AUTO_INCREMENT COMMENT '订单号',
    `webName` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '订购商平台',
    `productName` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '商品名称',
    `productType` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '商品类别',
    `orderCount` int NULL DEFAULT NULL COMMENT '订单数量',
    `orderDate` datetime NULL DEFAULT NULL COMMENT '订单日期',
    `agent` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '代理商',
    PRIMARY KEY (`orderID`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of ordertb
-- ----------------------------
INSERT INTO `ordertb` VALUES (1, '淘宝', '高露洁', '牙膏', 2000, '2015-03-14 09:02:28', '王丽');
INSERT INTO `ordertb` VALUES (2, '淘宝', '佳洁士', '牙膏', 700, '2015-03-14 09:02:28', '张涛');
INSERT INTO `ordertb` VALUES (3, '淘宝', '佳洁士', '牙膏', 700, '2015-03-14 09:02:28', '张涛');
INSERT INTO `ordertb` VALUES (4, '京东', 'IPAD Mini', '电子产品', 670, '2015-03-14 09:02:28', '赵晓菲');
INSERT INTO `ordertb` VALUES (5, '淘宝', '高露洁', '牙膏', 532, '2015-03-16 12:32:18', '张涛');
INSERT INTO `ordertb` VALUES (6, '一号店', '佳洁士', '牙膏', 988, '2015-03-16 12:32:18', '张丽');
INSERT INTO `ordertb` VALUES (7, '京东', 'iphone 6s', '电子产品', 2180, '2015-03-16 12:32:18', '张涛');
INSERT INTO `ordertb` VALUES (8, '京东', '佳洁士', '牙膏', 700, '2015-03-16 12:32:18', '朱晓宇');
INSERT INTO `ordertb` VALUES (9, '淘宝', '黑妹', '牙膏', 855, '2015-03-16 12:32:18', '张涛');
INSERT INTO `ordertb` VALUES (10, '淘宝', '佳洁士', '牙膏', 745, '2015-03-16 12:32:18', '赵晓菲');
INSERT INTO `ordertb` VALUES (11, '京东', '力士', '牙膏', 923, '2015-03-17 12:32:18', '赵晓菲');
INSERT INTO `ordertb` VALUES (12, '淘宝', '舒肤佳', '香皂', 866, '2015-03-17 12:32:18', '朱晓宇');
INSERT INTO `ordertb` VALUES (13, '淘宝', 'iphone 6s', '电子产品', 998, '2015-03-21 12:32:18', '赵晓菲');
INSERT INTO `ordertb` VALUES (14, '京东', '佳洁士', '牙膏', 2110, '2015-03-21 12:32:18', '张涛');
INSERT INTO `ordertb` VALUES (15, '一号店', '佳洁士', '牙膏', 1000, '2015-03-21 12:32:18', '张涛');
INSERT INTO `ordertb` VALUES (16, '京东', '佳洁士', '牙膏', 809, '2015-03-21 12:32:18', '朱晓宇');
INSERT INTO `ordertb` VALUES (17, '淘宝', '舒肤佳', '香皂', 319, '2015-03-24 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (18, '淘宝', '佳洁士', '牙膏', 3290, '2015-03-24 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (19, '京东', '华为888手机', '电子产品', 500, '2015-03-24 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (20, '京东', '力士', '香皂', 2188, '2015-03-24 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (21, '一号店', '佳洁士', '牙膏', 2000, '2015-03-24 15:31:48', '张丽');
INSERT INTO `ordertb` VALUES (22, '淘宝', '佳洁士', '牙膏', 1000, '2015-03-24 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (23, '京东', 'IPAD Mini', '电子产品', 1000, '2015-03-24 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (24, '淘宝', '高露洁', '牙膏', 1000, '2015-03-24 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (25, '一号店', '佳洁士', '牙膏', 1000, '2015-03-24 15:31:48', '张丽');
INSERT INTO `ordertb` VALUES (26, '京东', 'iphone 6s', '电子产品', 1000, '2015-03-24 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (27, '京东', '佳洁士', '牙膏', 1000, '2015-03-24 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (28, '淘宝', '黑妹', '牙膏', 1000, '2015-03-24 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (29, '淘宝', '佳洁士', '牙膏', 1000, '2015-04-02 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (30, '京东', '力士', '牙膏', 1000, '2015-04-02 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (31, '淘宝', '舒肤佳', '香皂', 1000, '2015-04-02 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (32, '淘宝', 'iphone 6s', '电子产品', 1000, '2015-04-02 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (33, '京东', '佳洁士', '牙膏', 1000, '2015-04-02 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (34, '一号店', '佳洁士', '牙膏', 1000, '2015-04-02 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (35, '京东', '佳洁士', '牙膏', 1000, '2015-04-02 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (36, '淘宝', '舒肤佳', '香皂', 1000, '2015-04-02 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (37, '淘宝', '佳洁士', '牙膏', 1000, '2015-04-02 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (38, '京东', '华为888手机', '电子产品', 1000, '2015-04-02 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (39, '京东', '力士', '香皂', 1000, '2015-04-02 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (40, '一号店', '佳洁士', '牙膏', 1000, '2015-04-02 15:31:48', '张丽');
INSERT INTO `ordertb` VALUES (41, '淘宝', '佳洁士', '牙膏', 1000, '2015-04-06 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (42, '京东', 'IPAD Mini', '电子产品', 1000, '2015-04-06 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (43, '淘宝', '高露洁', '牙膏', 1000, '2015-04-06 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (44, '一号店', '佳洁士', '牙膏', 1000, '2015-04-06 15:31:48', '张丽');
INSERT INTO `ordertb` VALUES (45, '京东', 'iphone 6s', '电子产品', 1000, '2015-04-06 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (46, '京东', '佳洁士', '牙膏', 1000, '2015-04-06 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (47, '淘宝', '黑妹', '牙膏', 1000, '2015-04-06 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (48, '淘宝', '佳洁士', '牙膏', 1000, '2015-04-10 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (49, '京东', '力士', '牙膏', 1000, '2015-04-10 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (50, '淘宝', '舒肤佳', '香皂', 1000, '2015-04-10 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (51, '淘宝', 'iphone 6s', '电子产品', 1000, '2015-04-10 15:31:48', '赵晓菲');
INSERT INTO `ordertb` VALUES (52, '京东', '佳洁士', '牙膏', 1000, '2015-04-10 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (53, '一号店', '佳洁士', '牙膏', 1000, '2015-04-10 15:31:48', '张涛');
INSERT INTO `ordertb` VALUES (54, '京东', '佳洁士', '牙膏', 1000, '2015-04-10 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (55, '淘宝', '舒肤佳', '香皂', 1000, '2015-04-10 15:31:48', '朱晓宇');
INSERT INTO `ordertb` VALUES (56, '淘宝', '佳洁士', '牙膏', 1000, '2016-05-03 14:39:24', '张涛');
INSERT INTO `ordertb` VALUES (57, '京东', '华为888手机', '电子产品', 1000, '2016-05-03 14:39:24', '赵晓菲');
INSERT INTO `ordertb` VALUES (58, '京东', '力士', '香皂', 1000, '2016-05-03 14:39:24', '张涛');
INSERT INTO `ordertb` VALUES (59, '一号店', '佳洁士', '牙膏', 1000, '2016-05-03 14:39:24', '张丽');
INSERT INTO `ordertb` VALUES (60, '淘宝', '佳洁士', '牙膏', 1000, '2016-05-03 14:39:25', '张涛');
INSERT INTO `ordertb` VALUES (61, '淘宝', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:49', '张涛');
INSERT INTO `ordertb` VALUES (62, '京东', 'IPAD Mini', '电子产品', 1000, '2016-04-05 14:40:49', '赵晓菲');
INSERT INTO `ordertb` VALUES (63, '淘宝', '高露洁', '牙膏', 1000, '2016-04-05 14:40:49', '张涛');
INSERT INTO `ordertb` VALUES (64, '一号店', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:49', '张丽');
INSERT INTO `ordertb` VALUES (65, '京东', 'iphone 6s', '电子产品', 1000, '2016-04-05 14:40:49', '张涛');
INSERT INTO `ordertb` VALUES (66, '京东', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:49', '朱晓宇');
INSERT INTO `ordertb` VALUES (67, '淘宝', '黑妹', '牙膏', 1000, '2016-04-05 14:40:49', '张涛');
INSERT INTO `ordertb` VALUES (68, '淘宝', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:49', '赵晓菲');
INSERT INTO `ordertb` VALUES (69, '京东', '力士', '牙膏', 1000, '2016-04-05 14:40:49', '赵晓菲');
INSERT INTO `ordertb` VALUES (70, '淘宝', '舒肤佳', '香皂', 1000, '2016-04-05 14:40:49', '朱晓宇');
INSERT INTO `ordertb` VALUES (71, '淘宝', 'iphone 6s', '电子产品', 1000, '2016-04-05 14:40:49', '赵晓菲');
INSERT INTO `ordertb` VALUES (72, '京东', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:49', '张涛');
INSERT INTO `ordertb` VALUES (73, '一号店', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:49', '张涛');
INSERT INTO `ordertb` VALUES (74, '京东', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:49', '朱晓宇');
INSERT INTO `ordertb` VALUES (75, '淘宝', '舒肤佳', '香皂', 1000, '2016-04-05 14:40:49', '朱晓宇');
INSERT INTO `ordertb` VALUES (76, '淘宝', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:50', '张涛');
INSERT INTO `ordertb` VALUES (77, '京东', '华为888手机', '电子产品', 1000, '2016-04-05 14:40:50', '赵晓菲');
INSERT INTO `ordertb` VALUES (78, '京东', '力士', '香皂', 1000, '2016-04-05 14:40:50', '张涛');
INSERT INTO `ordertb` VALUES (79, '一号店', '佳洁士', '牙膏', 1000, '2016-04-05 14:40:50', '张丽');

SET FOREIGN_KEY_CHECKS = 1;
```



## 2、JavaBean 实体类准备

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class OrderTB {
    private Integer orderID;
    private String webName;
    private String productName;
    private String productType;
    private Integer orderCount;
    private Date orderDate;
    private String agent;
}
```



## 3、要执行的SQL语句

```java
public interface Constants {
    String QUERY_ORDER_TB_LIST =
        "SELECT ORDERID,WEBNAME,PRODUCTNAME,PRODUCTTYPE,ORDERCOUNT,ORDERDATE,AGENT FROM ORDERTB " + 
        "WHERE WEBNAME = :webName AND PRODUCTNAME = :productName AND " +
        "PRODUCTTYPE = :productType AND ORDERCOUNT = :orderCount AND " + 
        "AGENT = :agent" ;
}
```

注意：可以看到SQL语句放在interface修饰的接口内，因为接口的成员变量默认被`public static final`修饰，所以可以使用接口文件定义全局静态变量。



## 4、定义业务接口 Service

```java
public interface OrderService {
    List<OrderTB> queryOrderTB(QueryCondition condition);
}
```



## 5、定义一个SqlParameterSource实现具名参数的实体类

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class QueryCondition {
    private String webName;
    private String productName;
    private String productType;
    private Integer orderCount;
    private String agent;
}
```



## 6、定义接口 Controller

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
public class OrderController {

    @Autowired
    OrderService service;

    @GetMapping("/query/order/by/bean")
    public List<OrderTB> queryOrderList(@RequestBody QueryCondition condition){
        return service.queryOrderTB(condition);
    }
}
```



## 7、业务实现

使用`SqlParameterSource`实现具名参数，默认实现有 ：

- **MapSqlParameterSource**：只是封装了java.util.Map。
- **BeanPropertySqlParameterSource**：封装了一个JavaBean对象，通过JavaBean对象属性设置具名参数值。
- **EmptySqlParameterSource**：一个空的SqlParameterSource ，占位使用。

这里只对前两个的使用进行介绍。



### 1、使用 MapSqlParameterSource

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Service;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

@Service
@Slf4j
public class OrderServiceImpl implements OrderService {

    @Autowired(required = false)
    NamedParameterJdbcTemplate template;

    @Override
    public List<OrderTB> queryOrderTB(QueryCondition condition) {

        MapSqlParameterSource mapCondition = this.setCondition(condition);

        List<OrderTB> list = template.query(Constants.QUERY_ORDER_TB_LIST, mapCondition, new RowMapper<OrderTB>() {
            @Override
            public OrderTB mapRow(ResultSet rs, int rowNum) throws SQLException {
                OrderTB orderTB = new OrderTB();
                orderTB.setOrderID(rs.getInt("orderID"));
                orderTB.setWebName(rs.getString("webName"));
                orderTB.setProductName(rs.getString("productName"));
                orderTB.setProductType(rs.getString("productType"));
                orderTB.setOrderCount(rs.getInt("orderCount"));
                orderTB.setOrderDate(rs.getDate("orderDate"));
                orderTB.setAgent(rs.getString("agent"));
                return orderTB;
            }
        });
        return list;
    }

    private MapSqlParameterSource setCondition(QueryCondition condition) {
        MapSqlParameterSource mapCondition = new MapSqlParameterSource();
        mapCondition.addValue("webName", condition.getWebName())
            .addValue("productName", condition.getProductName())
            .addValue("productType", condition.getProductType())
            .addValue("orderCount", condition.getOrderCount())
            .addValue("agent", condition.getAgent());
        return mapCondition;
    }
}
```



### 2、使用 BeanPropertySqlParameterSource

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Service;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

@Service
@Slf4j
public class OrderServiceImpl implements OrderService {

    @Autowired(required = false)
    NamedParameterJdbcTemplate template;

    @Override
    public List<OrderTB> queryOrderTB(QueryCondition condition) {

        BeanPropertySqlParameterSource beanParam = this.setCondition(condition);

        List<OrderTB> list = template.query(Constants.QUERY_ORDER_TB_LIST, beanParam, new RowMapper<OrderTB>() {
            @Override
            public OrderTB mapRow(ResultSet rs, int rowNum) throws SQLException {
                OrderTB orderTB = new OrderTB();
                orderTB.setOrderID(rs.getInt("orderID"));
                orderTB.setWebName(rs.getString("webName"));
                orderTB.setProductName(rs.getString("productName"));
                orderTB.setProductType(rs.getString("productType"));
                orderTB.setOrderCount(rs.getInt("orderCount"));
                orderTB.setOrderDate(rs.getDate("orderDate"));
                orderTB.setAgent(rs.getString("agent"));
                return orderTB;
            }
        });
        return list;
    }

    private BeanPropertySqlParameterSource setCondition(QueryCondition condition) {
        BeanPropertySqlParameterSource beanParam = new BeanPropertySqlParameterSource(condition);
        return beanParam;
    }
}
```

API：localhost:8080/query/order/by/bean、Method: POST

RequestBody:

```json
{
    "webName": "淘宝",
    "productName": "佳洁士",
    "productType": "牙膏",
    "orderCount": "700",
    "agent": "张涛"
}
```

ResponseBody

```json
[
    {
        "orderID": "2",
        "webName": "淘宝",
        "productName": "佳洁士",
        "productType": "牙膏",
        "orderCount": "700",
        "orderDate": "2015-03-14",
        "agent": "张涛"
    },
    {
        "orderID": "3",
        "webName": "淘宝",
        "productName": "佳洁士",
        "productType": "牙膏",
        "orderCount": "700",
        "orderDate": "2015-03-14",
        "agent": "张涛"
    }
]
```



# 五、参考文献 & 鸣谢

- 作者：[YD_1989](https://blog.csdn.net/m0_58680865)、来源：
  - NamedParameterJdbcTemplate使用详解：https://blog.csdn.net/m0_58680865/article/details/126901687
  - jdbcTemplate使用：https://blog.csdn.net/m0_58680865/article/details/126907528
  - NamedParameterJdbcTemplate —— SqlParameterSource实现具名参数查询：https://blog.csdn.net/m0_58680865/article/details/128294786