# Java 多线程事务处理最佳实践

首先需要确认的是，使用多线程处理事务的情况下，最好是使用编程式事务，这样可以更加灵活管理事务的提交与回滚。如果是使用声明式事务`@Transactional`的话非常容易出现事务**失效**的情况。【因为`@Transactional`使用的方法粒度需要控制非常好才行】

一般在多线程下使用事务管理器，我们会在每一个线程中管理自己的事务【也就是提交和回滚事务】，与其他线程中的事务互不干扰。不过这里还是会分两种情况：

1. 所有线程一起提交事务，要么全部成功，要么全部回滚。【相当于本次操作为一个事务，类似于分布式式事务提交】
2. 每个线程提交或回滚自己的事务、互不干扰。【相当于把本次操作切分为几个小模块，每个模块中管理自己的事务】

本次只演示第二种情况，每个线程管理自己的事务，线程A的提交，不会影响到线程B的回滚。这也是相对常见的情况。



## 1、环境搭建

1、导入 Maven 依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
</properties>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
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
```

2、properties 的配置

```properties
# H2 数据库的配置
spring.datasource.driverClassName = org.h2.Driver
spring.datasource.url = jdbc:h2:mem:test_mem
spring.datasource.username = sa
spring.datasource.password = sa

# H2 数据库的访问配置
spring.h2.console.path = /h2
spring.h2.console.settings.web-allow-others = true
spring.h2.console.enabled = true

# 数据库的初始化脚本
spring.sql.init.platform = h2
spring.sql.init.schema-locations = classpath:db/schema.sql
spring.sql.init.data-locations = classpath:db/data.sql

# 日志的打印配置
#logging.level.org.springframework.jdbc.core.JdbcTemplate=DEBUG
#logging.level.org.springframework.jdbc.core.StatementCreatorUtils=TRACE
#logging.level.org.springframework.data.jdbc.repository.query=DEBUG
#logging.level.oorg.springframework.data.jdbc.core.JdbcAggregateTemplate=DEBUG
```

3、需要初始化的数据库脚本

- resources/db/schema.sql

  ```sql
  DROP TABLE IF EXISTS USER_INFO;
  CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY AUTO_INCREMENT, USER_NAME VARCHAR(255));
  ```

- resources/db/data.sql

  ```sql
  INSERT INTO USER_INFO VALUES(100000001, 'Hello');
  INSERT INTO USER_INFO VALUES(100000002, 'World');
  INSERT INTO USER_INFO VALUES(100000003, '乔丹');
  INSERT INTO USER_INFO VALUES(100000004, '科比');
  INSERT INTO USER_INFO VALUES(100000005, '艾佛森');
  INSERT INTO USER_INFO VALUES(100000006, '麦迪');
  ```

4、Java 实体类 以及一些工具方法

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Table;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;
import java.util.stream.Stream;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Table("USER_INFO")
public class UserInfo {
    @Id
    private Integer userId;
    private String userName;

    public static List<UserInfo> getUserInfos(int size) {
        return Stream.iterate(1, i -> i + 1)
                .limit(size)
                .map(x -> new UserInfo(x, UUID.randomUUID().toString()))
                .collect(Collectors.toList());
    }

    /**
     * Splits a list into smaller sublists of a specified size.
     *
     * @param source        the original list to be partitioned
     * @param partitionSize the maximum size of each sublist
     * @param <T>           the type of elements in the list
     * @return a list of sublists
     */
    public static <T> List<List<T>> partition(List<T> source, int partitionSize) {
        if (source == null || source.isEmpty() || partitionSize <= 0) {
            throw new IllegalArgumentException("Invalid input parameters.");
        }

        List<List<T>> partitions = new ArrayList<>();
        int sourceSize = source.size();
        for (int i = 0; i < sourceSize; i += partitionSize) {
            partitions.add(new ArrayList<>(
                    source.subList(i, Math.min(i + partitionSize, sourceSize))
            ));
        }
        return partitions;
    }
}
```



## 2、CompletableFuture + TransactionManager 实现

操作示例一：每个线程提交或回滚自己的事务、互不干扰。这里使用 CompletableFuture + TransactionManager 演示无返回值的情况。

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.stream.StreamSupport;

@Slf4j
@Component
public class ThreadTxTransactionManager1 {
    // private static final int availableProcessors = Runtime.getRuntime().availableProcessors();
    private static final int availableProcessors = 5;
    private static final ThreadPoolExecutor executor = new ThreadPoolExecutor(
            availableProcessors,
            availableProcessors * 2,
            60L,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(1024),
            new ThreadPoolExecutor.CallerRunsPolicy());

    @Resource
    private PlatformTransactionManager transactionManager;
    @Resource
    private JdbcAggregateTemplate jdbcAggregateTemplate;

    public void tx() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);
        // 切分集合, 相当于每个线程处理个集合大小
        List<List<UserInfo>> partitions = UserInfo.partition(userInfos, 20_0000);

        // 记录多线程的耗时
        long startTime = System.currentTimeMillis();

        CompletableFuture.allOf(partitions.stream().map(partition ->
                CompletableFuture.runAsync(() -> {
                    // Spring事务内部由ThreadLocal存储事务绑定信息，因此需要每个线程新开一个事务
                    TransactionStatus transactionStatus = beginNewTransaction();
                    try {
                        log.info("线程开始启动，线程名 = {}", Thread.currentThread().getName());

                        // TODO: DB代码 或者 业务代码 的操作, 这里只是简单插入数据
                        partition.forEach(jdbcAggregateTemplate::insert);
                        // 无任何异常提交事务
                        transactionManager.commit(transactionStatus);
                    } catch (Exception e) {
                        log.error("线程出现异常，线程名 = {}", Thread.currentThread().getName(), e);
                        // 出现异常回滚事务
                        transactionManager.rollback(transactionStatus);
                    }
                }, executor)).toArray(CompletableFuture[]::new)).join();

        // 记录插入数据总耗时
        log.info("插入数据总耗时: {}", System.currentTimeMillis() - startTime);

        // 验证数据是否插入成功
        Iterable<UserInfo> all = jdbcAggregateTemplate.findAll(UserInfo.class);
        long count = StreamSupport.stream(all.spliterator(), false).count();
        log.info("查询插入数据总数量: {}", count);
    }

    private TransactionStatus beginNewTransaction() {
        DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
        definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        return transactionManager.getTransaction(definition);
    }

}
```

```java
2024-08-25 14:30:01.647  INFO 22645 --- [pool-6-thread-5] c.e.s.t.t.ThreadTxTransactionManager1    : 线程开始启动，线程名 = pool-6-thread-5
2024-08-25 14:30:01.648  INFO 22645 --- [pool-6-thread-4] c.e.s.t.t.ThreadTxTransactionManager1    : 线程开始启动，线程名 = pool-6-thread-4
2024-08-25 14:30:01.647  INFO 22645 --- [pool-6-thread-2] c.e.s.t.t.ThreadTxTransactionManager1    : 线程开始启动，线程名 = pool-6-thread-2
2024-08-25 14:30:01.647  INFO 22645 --- [pool-6-thread-3] c.e.s.t.t.ThreadTxTransactionManager1    : 线程开始启动，线程名 = pool-6-thread-3
2024-08-25 14:30:01.648  INFO 22645 --- [pool-6-thread-1] c.e.s.t.t.ThreadTxTransactionManager1    : 线程开始启动，线程名 = pool-6-thread-1
2024-08-25 14:30:22.771  INFO 22645 --- [           main] c.e.s.t.t.ThreadTxTransactionManager1    : 插入数据总耗时: 21206
2024-08-25 14:30:28.387  INFO 22645 --- [           main] c.e.s.t.t.ThreadTxTransactionManager1    : 查询插入数据总数量: 1000006
```



操作示例二：每个线程提交或回滚自己的事务、互不干扰。这里使用 CompletableFuture + TransactionManager 演示有返回值的情况。

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import javax.annotation.Resource;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;
import java.util.stream.StreamSupport;

@Slf4j
@Component
public class ThreadTxTransactionManager2 {
    // private static final int availableProcessors = Runtime.getRuntime().availableProcessors();
    private static final int availableProcessors = 5;
    private static final ThreadPoolExecutor executor = new ThreadPoolExecutor(
            availableProcessors,
            availableProcessors * 2,
            60L,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(1024),
            new ThreadPoolExecutor.CallerRunsPolicy());

    @Resource
    private PlatformTransactionManager transactionManager;
    @Resource
    private JdbcAggregateTemplate jdbcAggregateTemplate;

    public void tx() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);
        // 切分集合, 相当于每个线程处理个集合大小
        List<List<UserInfo>> partitions = UserInfo.partition(userInfos, 20_0000);

        // 记录多线程的耗时
        long startTime = System.currentTimeMillis();

        List<CompletableFuture<List<UserInfo>>> futures = partitions.stream().map(partition ->
                CompletableFuture.supplyAsync(() -> {
                    // Spring事务内部由ThreadLocal存储事务绑定信息，因此需要每个线程新开一个事务
                    TransactionStatus transactionStatus = beginNewTransaction();
                    try {
                        log.info("线程开始启动，线程名 = {}", Thread.currentThread().getName());

                        // TODO: DB代码 或者 业务代码 的操作, 这里只是简单插入数据
                        List<UserInfo> infos = partition.stream()
                                .map(jdbcAggregateTemplate::insert)
                                .collect(Collectors.toList());
                        // 无任何异常提交事务
                        transactionManager.commit(transactionStatus);
                        return infos;
                    } catch (Exception e) {
                        log.error("线程出现异常，线程名 = {}", Thread.currentThread().getName(), e);
                        // 出现异常回滚事务
                        transactionManager.rollback(transactionStatus);
                        throw new RuntimeException(e);
                    }
                }, executor)).collect(Collectors.toList());

        List<UserInfo> results = futures.stream()
                .map(CompletableFuture::join)
                .flatMap(Collection::stream)
                .collect(Collectors.toList());
        // 插入数据返回值数量
        log.info("插入数据总数量: {}", results.size());

        // 记录插入数据总耗时
        log.info("插入数据总耗时: {}", System.currentTimeMillis() - startTime);

        // 验证数据是否插入成功
        Iterable<UserInfo> all = jdbcAggregateTemplate.findAll(UserInfo.class);
        long count = StreamSupport.stream(all.spliterator(), false).count();
        log.info("查询插入数据总数量: {}", count);
    }

    private TransactionStatus beginNewTransaction() {
        DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
        definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        return transactionManager.getTransaction(definition);
    }

}
```

```java
2024-08-25 14:31:23.352  INFO 22714 --- [pool-7-thread-1] c.e.s.t.t.ThreadTxTransactionManager2    : 线程开始启动，线程名 = pool-7-thread-1
2024-08-25 14:31:23.353  INFO 22714 --- [pool-7-thread-5] c.e.s.t.t.ThreadTxTransactionManager2    : 线程开始启动，线程名 = pool-7-thread-5
2024-08-25 14:31:23.353  INFO 22714 --- [pool-7-thread-3] c.e.s.t.t.ThreadTxTransactionManager2    : 线程开始启动，线程名 = pool-7-thread-3
2024-08-25 14:31:23.353  INFO 22714 --- [pool-7-thread-4] c.e.s.t.t.ThreadTxTransactionManager2    : 线程开始启动，线程名 = pool-7-thread-4
2024-08-25 14:31:23.353  INFO 22714 --- [pool-7-thread-2] c.e.s.t.t.ThreadTxTransactionManager2    : 线程开始启动，线程名 = pool-7-thread-2
2024-08-25 14:31:46.975  INFO 22714 --- [           main] c.e.s.t.t.ThreadTxTransactionManager2    : 插入数据总数量: 1000000
2024-08-25 14:31:46.979  INFO 22714 --- [           main] c.e.s.t.t.ThreadTxTransactionManager2    : 插入数据总耗时: 23736
2024-08-25 14:31:52.430  INFO 22714 --- [           main] c.e.s.t.t.ThreadTxTransactionManager2    : 查询插入数据总数量: 1000006
```



## 3、CompletableFuture + TransactionTemplate 实现

操作示例一：每个线程提交或回滚自己的事务、互不干扰。这里使用 CompletableFuture + TransactionTemplate 演示无返回值的情况。

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.support.TransactionTemplate;

import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.stream.StreamSupport;

@Slf4j
@Component
public class ThreadTxTransactionTemplate1 {
    // private static final int availableProcessors = Runtime.getRuntime().availableProcessors();
    private static final int availableProcessors = 5;
    private static final ThreadPoolExecutor executor = new ThreadPoolExecutor(
            availableProcessors,
            availableProcessors * 2,
            60L,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(1024),
            new ThreadPoolExecutor.CallerRunsPolicy());

    @Resource
    private TransactionTemplate transactionTemplate;
    @Resource
    private JdbcAggregateTemplate jdbcAggregateTemplate;

    public void tx() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);
        // 切分集合, 相当于每个线程处理个集合大小
        List<List<UserInfo>> partitions = UserInfo.partition(userInfos, 20_0000);

        // 记录多线程的耗时
        long startTime = System.currentTimeMillis();

        CompletableFuture.allOf(partitions.stream().map(partition ->
                CompletableFuture.runAsync(() -> transactionTemplate.executeWithoutResult(transactionStatus -> {
                    try {
                        log.info("线程开始启动，线程名 = {}", Thread.currentThread().getName());

                        // TODO: DB代码 或者 业务代码 的操作, 这里只是简单插入数据
                        partition.forEach(jdbcAggregateTemplate::insert);
                        // 无任何异常会自动提交事务
                    } catch (Exception e) {
                        log.error("线程出现异常，线程名 = {}", Thread.currentThread().getName(), e);
                        // 出现异常回滚事务
                        transactionStatus.setRollbackOnly();
                    }
                }), executor)).toArray(CompletableFuture[]::new)).join();

        // 记录插入数据总耗时
        log.info("插入数据总耗时: {}", System.currentTimeMillis() - startTime);

        // 验证数据是否插入成功
        Iterable<UserInfo> all = jdbcAggregateTemplate.findAll(UserInfo.class);
        long count = StreamSupport.stream(all.spliterator(), false).count();
        log.info("查询插入数据总数量: {}", count);
    }

}
```

```java
2024-08-25 14:32:26.978  INFO 22778 --- [pool-8-thread-2] c.e.s.t.t.ThreadTxTransactionTemplate1   : 线程开始启动，线程名 = pool-8-thread-2
2024-08-25 14:32:26.978  INFO 22778 --- [pool-8-thread-3] c.e.s.t.t.ThreadTxTransactionTemplate1   : 线程开始启动，线程名 = pool-8-thread-3
2024-08-25 14:32:26.977  INFO 22778 --- [pool-8-thread-5] c.e.s.t.t.ThreadTxTransactionTemplate1   : 线程开始启动，线程名 = pool-8-thread-5
2024-08-25 14:32:26.978  INFO 22778 --- [pool-8-thread-1] c.e.s.t.t.ThreadTxTransactionTemplate1   : 线程开始启动，线程名 = pool-8-thread-1
2024-08-25 14:32:26.977  INFO 22778 --- [pool-8-thread-4] c.e.s.t.t.ThreadTxTransactionTemplate1   : 线程开始启动，线程名 = pool-8-thread-4
2024-08-25 14:32:51.011  INFO 22778 --- [           main] c.e.s.t.t.ThreadTxTransactionTemplate1   : 插入数据总耗时: 24118
2024-08-25 14:32:56.696  INFO 22778 --- [           main] c.e.s.t.t.ThreadTxTransactionTemplate1   : 查询插入数据总数量: 1000006
```



操作示例二：每个线程提交或回滚自己的事务、互不干扰。这里使用 CompletableFuture + TransactionTemplate 演示有返回值的情况。

```java
import com.example.springboot.tech.UserInfo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
import org.springframework.transaction.support.TransactionTemplate;

import javax.annotation.Resource;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;
import java.util.stream.StreamSupport;

@Slf4j
@Component
public class ThreadTxTransactionTemplate2 {
    // private static final int availableProcessors = Runtime.getRuntime().availableProcessors();
    private static final int availableProcessors = 5;
    private static final ThreadPoolExecutor executor = new ThreadPoolExecutor(
            availableProcessors,
            availableProcessors * 2,
            60L,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(1024),
            new ThreadPoolExecutor.CallerRunsPolicy());

    @Resource
    private TransactionTemplate transactionTemplate;
    @Resource
    private JdbcAggregateTemplate jdbcAggregateTemplate;

    public void tx() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);
        // 切分集合, 相当于每个线程处理个集合大小
        List<List<UserInfo>> partitions = UserInfo.partition(userInfos, 20_0000);

        // 记录多线程的耗时
        long startTime = System.currentTimeMillis();

        CompletableFuture.allOf(partitions.stream().map(partition ->
                CompletableFuture.runAsync(() -> transactionTemplate.execute(transactionStatus -> {
                    try {
                        log.info("线程开始启动，线程名 = {}", Thread.currentThread().getName());

                        // TODO: DB代码 或者 业务代码 的操作, 这里只是简单插入数据
                        List<UserInfo> infos = partition.stream()
                                .map(jdbcAggregateTemplate::insert)
                                .collect(Collectors.toList());
                        return infos;
                        // 无任何异常会自动提交事务
                    } catch (Exception e) {
                        log.error("线程出现异常，线程名 = {}", Thread.currentThread().getName(), e);
                        // 出现异常回滚事务
                        transactionStatus.setRollbackOnly();
                        throw new RuntimeException(e);
                    }
                }), executor)).toArray(CompletableFuture[]::new)).join();

        // 记录插入数据总耗时
        log.info("插入数据总耗时: {}", System.currentTimeMillis() - startTime);

        // 验证数据是否插入成功
        Iterable<UserInfo> all = jdbcAggregateTemplate.findAll(UserInfo.class);
        long count = StreamSupport.stream(all.spliterator(), false).count();
        log.info("查询插入数据总数量: {}", count);
    }

}
```

```java
2024-08-25 14:33:50.508  INFO 22842 --- [pool-9-thread-3] c.e.s.t.t.ThreadTxTransactionTemplate2   : 线程开始启动，线程名 = pool-9-thread-3
2024-08-25 14:33:50.507  INFO 22842 --- [pool-9-thread-4] c.e.s.t.t.ThreadTxTransactionTemplate2   : 线程开始启动，线程名 = pool-9-thread-4
2024-08-25 14:33:50.507  INFO 22842 --- [pool-9-thread-1] c.e.s.t.t.ThreadTxTransactionTemplate2   : 线程开始启动，线程名 = pool-9-thread-1
2024-08-25 14:33:50.508  INFO 22842 --- [pool-9-thread-5] c.e.s.t.t.ThreadTxTransactionTemplate2   : 线程开始启动，线程名 = pool-9-thread-5
2024-08-25 14:33:50.508  INFO 22842 --- [pool-9-thread-2] c.e.s.t.t.ThreadTxTransactionTemplate2   : 线程开始启动，线程名 = pool-9-thread-2
2024-08-25 14:34:15.878  INFO 22842 --- [           main] c.e.s.t.t.ThreadTxTransactionTemplate2   : 插入数据总耗时: 25451
2024-08-25 14:34:21.349  INFO 22842 --- [           main] c.e.s.t.t.ThreadTxTransactionTemplate2   : 查询插入数据总数量: 1000006
```

注意：因为这里使用的是 CompletableFuture，所有无需使用 CountDownLatch 来控制，如果想使用 CountDownLatch 来控制主线程等待子线程，可以使用 executor.execute() 或者 executor.submit() 配合 CountDownLatch 来实现。这里就不演示了。



# Java 多线程数据库事务提交与回滚控制

> 作者：[圣心](https://blog.csdn.net/qq273766764)、来源：Java多线程数据库事务提交控制：https://blog.csdn.net/qq273766764/article/details/119972911

项目代码基于：H2 数据，开发框架为：SpringBoot、Spring-Data-JDBC。开发语言为：Java 11

项目代码：https://gitee.com/john273766764/springboot-mybatis-threads

公司业务中遇到一个需求，需要同时修改最多约100万条数据，而且还不支持批量或异步修改操作。于是只能写个for循环操作，但操作耗时太长，只能一步一步寻找其他解决方案。具体操作如下：

## 1、项目环境的搭建

1、导入 Maven 依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
</properties>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
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
```

2、properties 的配置

```properties
# H2 数据库的配置
spring.datasource.driverClassName = org.h2.Driver
spring.datasource.url = jdbc:h2:mem:test_mem
spring.datasource.username = sa
spring.datasource.password = sa

# H2 数据库的访问配置
spring.h2.console.path = /h2
spring.h2.console.settings.web-allow-others = true
spring.h2.console.enabled = true

# 数据库的初始化脚本
spring.sql.init.platform = h2
spring.sql.init.schema-locations = classpath:db/schema.sql
spring.sql.init.data-locations = classpath:db/data.sql

# 日志的打印配置
#logging.level.org.springframework.jdbc.core.JdbcTemplate=DEBUG
#logging.level.org.springframework.jdbc.core.StatementCreatorUtils=TRACE
#logging.level.org.springframework.data.jdbc.repository.query=DEBUG
#logging.level.oorg.springframework.data.jdbc.core.JdbcAggregateTemplate=DEBUG
```

3、需要初始化的数据库脚本

- resources/db/schema.sql

  ```sql
  DROP TABLE IF EXISTS USER_INFO;
  CREATE TABLE USER_INFO(USER_ID INT PRIMARY KEY AUTO_INCREMENT, USER_NAME VARCHAR(255));
  ```

- resources/db/data.sql

  ```sql
  INSERT INTO USER_INFO VALUES(100000001, 'Hello');
  INSERT INTO USER_INFO VALUES(100000002, 'World');
  INSERT INTO USER_INFO VALUES(100000003, '乔丹');
  INSERT INTO USER_INFO VALUES(100000004, '科比');
  INSERT INTO USER_INFO VALUES(100000005, '艾佛森');
  INSERT INTO USER_INFO VALUES(100000006, '麦迪');
  ```

4、Java 实体类 以及一些工具方法

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Table;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;
import java.util.stream.Stream;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Table("USER_INFO")
public class UserInfo {
    @Id
    private Integer userId;
    private String userName;

    public static List<UserInfo> getUserInfos(int size) {
        return Stream.iterate(1, i -> i + 1)
                .limit(size)
                .map(x -> new UserInfo(x, UUID.randomUUID().toString()))
                .collect(Collectors.toList());
    }

    /**
     * Splits a list into smaller sublists of a specified size.
     *
     * @param source        the original list to be partitioned
     * @param partitionSize the maximum size of each sublist
     * @param <T>           the type of elements in the list
     * @return a list of sublists
     */
    public static <T> List<List<T>> partition(List<T> source, int partitionSize) {
        if (source == null || source.isEmpty() || partitionSize <= 0) {
            throw new IllegalArgumentException("Invalid input parameters.");
        }

        List<List<T>> partitions = new ArrayList<>();
        int sourceSize = source.size();
        for (int i = 0; i < sourceSize; i += partitionSize) {
            partitions.add(new ArrayList<>(
                    source.subList(i, Math.min(i + partitionSize, sourceSize))
            ));
        }
        return partitions;
    }
}
```





2、循环操作的代码
------------------------------------------------------------------------------------------------

先写一个最简单的for循环代码，看看耗时情况怎么样。

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Service;
import org.springframework.util.StopWatch;

import javax.annotation.Resource;
import java.util.List;

@Slf4j
@Service
public class UserInfoService_1 {

    @Resource
    JdbcAggregateTemplate jdbcAggregateTemplate;

    /***
     * 一条一条依次对 1000000 条数据进行更新操作
     * 耗时：39s, 40s, 42s
     */
    public void saves() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);

        // 记录耗时
        StopWatch watch = new StopWatch();
        watch.start("autoTransactional");

        // 一条一条插入数据
        userInfos.forEach(jdbcAggregateTemplate::insert);

        watch.stop();
        log.info(watch.prettyPrint());
    }
}

```

测试代码：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;

@SpringBootTest
class ApplicationTests {
    @Resource
    UserInfoService_1 userInfoService1;

    @Test
    void test1() {
        userInfoService1.saves();
    }
}
```

输出结果：

```java
2024-08-25 14:48:26.999  INFO 23545 --- [           main] c.e.springboot.tech.UserInfoService_1    : StopWatch '': running time = 42033113041 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
42033113041  100%  autoTransactional
```

循环修改整体耗时约 **42秒**，且代码中没有手动事务控制应该是自动事务提交，所以每次操作事务都会提交所以操作比较慢，我们先对代码中添加手动事务控制，看查询效率怎样。



3、使用手动事务的操作代码
----------------------------------------------------------------------------------------------------

修改后的代码如下：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.stereotype.Service;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.util.StopWatch;

import javax.annotation.Resource;
import java.util.List;

@Slf4j
@Service
public class UserInfoService_2 {

    @Resource
    JdbcAggregateTemplate jdbcAggregateTemplate;
    @Resource
    DataSourceTransactionManager dataSourceTransactionManager;
    @Resource
    TransactionDefinition transactionDefinition;

    /***
     * 由于希望更新操作 一次性完成，需要手动控制添加事务
     * 耗时：38s, 37s, 38s
     * 从测试结果可以看出，添加事务后插入数据的效率有明显的提升
     */
    public void saves() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);

        // 记录耗时
        StopWatch watch = new StopWatch();
        watch.start("manualTransactional");

        // 手动开启事务
        TransactionStatus transactionStatus = dataSourceTransactionManager.getTransaction(transactionDefinition);
        try {
            // 插入数据
            userInfos.forEach(jdbcAggregateTemplate::insert);
            // 确保事务只提交一次, 不能先提交事务, 然后再次回滚或者重提提交事务
            dataSourceTransactionManager.commit(transactionStatus);
            log.info("manual transaction committed");
        } catch (Exception e) {
            // 如果事务提交失败，这里才会进行回滚
            if (!transactionStatus.isCompleted()) {
                dataSourceTransactionManager.rollback(transactionStatus);
            }
            log.error("Transaction failed and rolled back", e);
        } finally {
            watch.stop();
            log.info(watch.prettyPrint());
        }
    }
}

```

测试代码

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import javax.annotation.Resource;

@SpringBootTest
class ApplicationTests {
    @Resource
    UserInfoService_2 userInfoService2;

    @Test
    void test2() {
        userInfoService2.saves();
    }
}
```

输出结果

```java
2024-08-25 14:50:23.698  INFO 23645 --- [           main] c.e.springboot.tech.UserInfoService_2    : manual transaction committed
2024-08-25 14:50:23.701  INFO 23645 --- [           main] c.e.springboot.tech.UserInfoService_2    : StopWatch '': running time = 39086357708 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
39086357708  100%  manualTransactional
```

添加手动事务操控制后，整体耗时约 **39秒**，这相对于自动事务提交的代码，快了约3-4秒，**对于大量循环数据库提交操作，添加手动事务可以有效提高操作效率。**



4、尝试多线程进行数据修改【两种方案】
-------------

添加数据库手动事务后操作效率有明细提高，但还是比较长，接下来尝试多线程提交看是不是能够再快一些。

先添加一个 Service 将批量修改操作整合一下，具体代码如下：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.stereotype.Service;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.util.StopWatch;

import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;

@Slf4j
@Service
public class UserInfoService_3 {

    @Resource
    JdbcAggregateTemplate jdbcAggregateTemplate;
    @Resource
    DataSourceTransactionManager dataSourceTransactionManager;
    @Resource
    TransactionDefinition transactionDefinition;

    /**
     * 线程数量
     */
    public static final int THREAD_COUNT = 5;
    /**
     * 线程池初初始化
     */
    public static final ExecutorService THREAD_POOL = Executors.newFixedThreadPool(THREAD_COUNT);

    /**
     * 使用多线程创建1000000条数据
     * 耗时统计
     *  1 线程数：1      耗时：37s
     *  2 线程数：2      耗时：25s
     *  3 线程数：5      耗时：21s
     *  4 线程数：10     耗时：21s
     *  5 线程数：100    耗时：29s
     */
    public void saves() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);

        // 每个线程处理的数据量
        int dataPortionLength = (userInfos.size() + THREAD_COUNT - 1) / THREAD_COUNT;

        // 记录耗时
        StopWatch watch = new StopWatch();
        watch.start("manualTransactional");

        CountDownLatch threadLatch = new CountDownLatch(THREAD_COUNT);

        for (int i = 0; i < THREAD_COUNT; i++) {
            // 每个线程处理的数据
            List<UserInfo> threadData = userInfos.stream()
                    .skip(i * dataPortionLength)
                    .limit(dataPortionLength)
                    .collect(Collectors.toList());
            THREAD_POOL.execute(() -> this.savaUserInfos(threadData, threadLatch));
            // CompletableFuture.runAsync(() -> this.savaUserInfos(threadData, threadLatch), THREAD_POOL);
        }


        try {
            // 倒计时锁设置超时时间 300s
            boolean await = threadLatch.await(300, TimeUnit.SECONDS);
            // 等待超时，事务回滚
            if (!await) {
               log.error("CountDownLatch await timeout");
            }
            log.info("manual transaction completed");
        } catch (Throwable e) {
            log.error(e.getMessage(), e);
        }

        log.info("============================== 主线程完成 ==============================");
        watch.stop();
        log.info(watch.prettyPrint());
    }

    public void savaUserInfos(List<UserInfo> userInfos, CountDownLatch threadLatch) {
        TransactionStatus transactionStatus = dataSourceTransactionManager.getTransaction(transactionDefinition);
        log.info("子线程：{}", Thread.currentThread().getName());
        try {
            userInfos.forEach(jdbcAggregateTemplate::insert);
            dataSourceTransactionManager.commit(transactionStatus);
            log.info("commited {}", Thread.currentThread().getName());
        } catch (Throwable e) {
            dataSourceTransactionManager.rollback(transactionStatus);
            log.info("rollback {}", Thread.currentThread().getName());
        } finally {
            threadLatch.countDown();
        }
    }
}

```

批量测试代码，我们采用了多线程进行提交，修改后测试代码如下：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import javax.annotation.Resource;

@SpringBootTest
class ApplicationTests {
    @Resource
    UserInfoService_3 userInfoService3;

    @Test
    void test3() {
        userInfoService3.saves();
    }
}
```

输出结果

```java
2024-08-25 14:56:43.019  INFO 24605 --- [pool-1-thread-1] c.e.springboot.tech.UserInfoService_3    : 子线程：pool-1-thread-1
2024-08-25 14:56:43.043  INFO 24605 --- [pool-1-thread-2] c.e.springboot.tech.UserInfoService_3    : 子线程：pool-1-thread-2
2024-08-25 14:56:43.063  INFO 24605 --- [pool-1-thread-3] c.e.springboot.tech.UserInfoService_3    : 子线程：pool-1-thread-3
2024-08-25 14:56:43.109  INFO 24605 --- [pool-1-thread-4] c.e.springboot.tech.UserInfoService_3    : 子线程：pool-1-thread-4
2024-08-25 14:56:43.179  INFO 24605 --- [pool-1-thread-5] c.e.springboot.tech.UserInfoService_3    : 子线程：pool-1-thread-5
2024-08-25 14:57:08.310  INFO 24605 --- [pool-1-thread-1] c.e.springboot.tech.UserInfoService_3    : commited pool-1-thread-1
2024-08-25 14:57:08.431  INFO 24605 --- [pool-1-thread-5] c.e.springboot.tech.UserInfoService_3    : commited pool-1-thread-5
2024-08-25 14:57:08.479  INFO 24605 --- [pool-1-thread-3] c.e.springboot.tech.UserInfoService_3    : commited pool-1-thread-3
2024-08-25 14:57:08.500  INFO 24605 --- [pool-1-thread-2] c.e.springboot.tech.UserInfoService_3    : commited pool-1-thread-2
2024-08-25 14:57:08.511  INFO 24605 --- [pool-1-thread-4] c.e.springboot.tech.UserInfoService_3    : commited pool-1-thread-4
2024-08-25 14:57:08.515  INFO 24605 --- [           main] c.e.springboot.tech.UserInfoService_3    : manual transaction completed
2024-08-25 14:57:08.515  INFO 24605 --- [           main] c.e.springboot.tech.UserInfoService_3    : ============================== 主线程完成 ==============================
2024-08-25 14:57:08.520  INFO 24605 --- [           main] c.e.springboot.tech.UserInfoService_3    : StopWatch '': running time = 21574228917 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
21574228917  100%  manualTransactional
```

我们尝试了不同线程数对提交速度的影响，具体可以看下面表格，多线程修改50000条数据时 不同线程数耗时对比（秒）

| 线程数 | 1    | 2    | 5    | 10   | 100  |
| ------ | ---- | ---- | ---- | ---- | ---- |
| 耗时   | 37s  | 25s  | 21s  | 21s  | 29s  |

**根据表格，我们线程数增大提交速度并非一直增大，在当前情况下约在2-5个线程数时，提交速度最快（实际线程数还是需要根据服务器配置实际测试）**。

***

> 如下为方案二：

不知道大家有没有注意到，如上的方式都是一次性使用完线程池中的所有线程，并没有让线程池出现等待的情况，也是是线程数大于线程池配置的核心数，下面我们来模拟此种情况。我们线程池配置5个核心数，需要跑的线程有10个。

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.util.StopWatch;

import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Slf4j
@Service
public class UserInfoService_33 {

    @Resource
    JdbcAggregateTemplate jdbcAggregateTemplate;
    @Resource
    PlatformTransactionManager transactionManager;
    @Resource
    TransactionDefinition transactionDefinition;

    /**
     * 线程数量
     */
    public static final int THREAD_COUNT = 5;
    /**
     * 线程池初初始化
     */
    public static final ExecutorService THREAD_POOL = Executors.newFixedThreadPool(THREAD_COUNT);

    /**
     * 使用多线程创建1000000条数据
     * 耗时统计
     * 1 线程数：1      耗时：38s
     * 2 线程数：2      耗时：27s
     * 3 线程数：5      耗时：21s
     * 4 线程数：10     耗时：25s
     * 5 线程数：100    耗时：29s
     */
    public void saves() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);

        // 切分集合, 相当于每个线程处理个集合大小
        List<List<UserInfo>> partitions = UserInfo.partition(userInfos, 10_0000);

        // 记录耗时
        StopWatch watch = new StopWatch();
        watch.start("manualTransactional");

        // 这里使用 Stream + CompletableFuture, join() 等待所有线程执行完毕
        CompletableFuture.allOf(partitions.stream().map(partition ->
                CompletableFuture.runAsync(() -> {
                    TransactionStatus transactionStatus = transactionManager.getTransaction(transactionDefinition);
                    log.info("子线程：{}", Thread.currentThread().getName());
                    try {
                        partition.forEach(jdbcAggregateTemplate::insert);
                        transactionManager.commit(transactionStatus);
                        log.info("commited {}", Thread.currentThread().getName());
                    } catch (Throwable e) {
                        transactionManager.rollback(transactionStatus);
                        log.info("rollback {}", Thread.currentThread().getName());
                    }
                }, THREAD_POOL)
        ).toArray(CompletableFuture[]::new)).join();

        log.info("============================== 主线程完成 ==============================");
        watch.stop();
        log.info(watch.prettyPrint());
        // 关闭线程池
        THREAD_POOL.shutdown();
    }

}
```

测试代码

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import javax.annotation.Resource;

@SpringBootTest
class ApplicationTests {
    @Resource
    UserInfoService_33 userInfoService33;

    @Test
    void test33() {
        userInfoService33.saves();
    }
}
```

输出结果

```java
2024-08-25 15:13:00.670  INFO 25479 --- [pool-2-thread-2] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-2
2024-08-25 15:13:00.669  INFO 25479 --- [pool-2-thread-5] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-5
2024-08-25 15:13:00.670  INFO 25479 --- [pool-2-thread-1] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-1
2024-08-25 15:13:00.669  INFO 25479 --- [pool-2-thread-4] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-4
2024-08-25 15:13:00.669  INFO 25479 --- [pool-2-thread-3] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-3
2024-08-25 15:13:15.550  INFO 25479 --- [pool-2-thread-5] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-5
2024-08-25 15:13:15.564  INFO 25479 --- [pool-2-thread-5] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-5
2024-08-25 15:13:15.715  INFO 25479 --- [pool-2-thread-1] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-1
2024-08-25 15:13:15.715  INFO 25479 --- [pool-2-thread-4] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-4
2024-08-25 15:13:15.716  INFO 25479 --- [pool-2-thread-3] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-3
2024-08-25 15:13:15.716  INFO 25479 --- [pool-2-thread-1] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-1
2024-08-25 15:13:15.716  INFO 25479 --- [pool-2-thread-4] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-4
2024-08-25 15:13:15.716  INFO 25479 --- [pool-2-thread-3] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-3
2024-08-25 15:13:16.057  INFO 25479 --- [pool-2-thread-2] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-2
2024-08-25 15:13:16.075  INFO 25479 --- [pool-2-thread-2] c.e.springboot.tech.UserInfoService_33   : 子线程：pool-2-thread-2
2024-08-25 15:13:24.485  INFO 25479 --- [pool-2-thread-1] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-1
2024-08-25 15:13:24.594  INFO 25479 --- [pool-2-thread-4] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-4
2024-08-25 15:13:24.613  INFO 25479 --- [pool-2-thread-5] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-5
2024-08-25 15:13:24.617  INFO 25479 --- [pool-2-thread-2] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-2
2024-08-25 15:13:24.624  INFO 25479 --- [pool-2-thread-3] c.e.springboot.tech.UserInfoService_33   : commited pool-2-thread-3
2024-08-25 15:13:24.625  INFO 25479 --- [           main] c.e.springboot.tech.UserInfoService_33   : ============================== 主线程完成 ==============================
2024-08-25 15:13:24.628  INFO 25479 --- [           main] c.e.springboot.tech.UserInfoService_33   : StopWatch '': running time = 24033524417 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
24033524417  100%  manualTransactional
```



5、两个 CountDownLatch 控制多线程事务提交
---------------------------------

由于多线程提交时，每个线程事务时单独的，无法保证一致性，我们尝试给多线程添加事务控制，来保证每个线程都是在插入数据完成后在提交事务，

**这里我们使用两个 CountDownLatch 来控制主线程与子线程事务提交，并设置了超时时间为 30 秒**。我们对代码进行了一点修改：

```java
package com.example.springboot.tech;

import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.util.StopWatch;

import javax.annotation.Resource;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.stream.Collectors;

@Slf4j
@Service
public class UserInfoService_4 {

    @Resource
    JdbcAggregateTemplate jdbcAggregateTemplate;
    @Resource
    PlatformTransactionManager transactionManager;
    @Resource
    TransactionDefinition transactionDefinition;

    /**
     * 线程数量
     */
    public static final int THREAD_COUNT = 5;
    /**
     * 线程池初初始化
     */
    public static final ExecutorService THREAD_POOL = Executors.newFixedThreadPool(THREAD_COUNT);

    /**
     * 使用多线程创建1000000条数据
     * 耗时统计
     *  1 线程数：1      耗时：36s
     *  2 线程数：2      耗时：22s
     *  3 线程数：5      耗时：18s
     *  4 线程数：10     耗时：21s
     *  5 线程数：100    耗时：33s
     *  6 线程数：200    耗时：29s
     *  7 线程数：500    耗时：44s
     */
    public void saves() {
        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);

        // 每个线程处理的数据量
        int dataPortionLength = (userInfos.size() + THREAD_COUNT - 1) / THREAD_COUNT;

        // 记录耗时
        StopWatch watch = new StopWatch();
        watch.start("manualTransactional");

        /**
         * 由于每个线程都是单独的事务，需要添加对线程事务的统一控制
         * 我们这边使用两个 CountDownLatch 对子线程的事务进行控制
         */
        // 用于计算子线程提交数量
        CountDownLatch threadLatch = new CountDownLatch(THREAD_COUNT);
        // 用于判断主线程是否提交
        CountDownLatch mainLatch = new CountDownLatch(1);
        // 用于判断子线程任务是否有错误
        AtomicBoolean isError = new AtomicBoolean();

        for (int i = 0; i < THREAD_COUNT; i++) {
            // 每个线程处理的数据
            List<UserInfo> threadData = userInfos.stream()
                    .skip(i * dataPortionLength)
                    .limit(dataPortionLength)
                    .collect(Collectors.toList());
            THREAD_POOL.execute(() -> this.savaUserInfos(threadData, threadLatch, mainLatch, isError));
            // CompletableFuture.runAsync(() -> this.savaUserInfos(threadData, threadLatch), THREAD_POOL);
        }

        try {
            // 倒计时锁设置超时时间 300s
            boolean await = threadLatch.await(300, TimeUnit.SECONDS);
            // 等待超时，事务回滚
            if (!await) {
                isError.set(true);
            }
        } catch (Throwable e) {
            isError.set(true);
            log.error(e.getMessage(), e);
        }
        // 切换到子线程执行
        mainLatch.countDown();
        // 关闭线程池
        THREAD_POOL.shutdown();

        log.info("============================== 主线程完成 ==============================");
        watch.stop();
        log.info(watch.prettyPrint());
    }

    public void savaUserInfos(List<UserInfo> userInfos, CountDownLatch threadLatch, CountDownLatch mainLatch, AtomicBoolean isError) {
        TransactionStatus transactionStatus = transactionManager.getTransaction(transactionDefinition);
        log.info("子线程：{}", Thread.currentThread().getName());
        try {
            userInfos.forEach(jdbcAggregateTemplate::insert);
        } catch (Throwable e) {
            isError.set(true);
            log.error("insert error: ", e);
        } finally {
            // 切换到主线程执行
            threadLatch.countDown();
        }
        try {
            mainLatch.await();
        } catch (InterruptedException e) {
            isError.set(true);
            log.error("mainLatch wait error: ", e);
        }
        // 判断是否有错误，如有错误 就回滚事务
        if (isError.get()) {
            transactionManager.rollback(transactionStatus);
            log.info("rollback {}", Thread.currentThread().getName());
        } else {
            transactionManager.commit(transactionStatus);
            log.info("commited {}", Thread.currentThread().getName());
        }
    }
}
```
测试代码

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import javax.annotation.Resource;

@SpringBootTest
class ApplicationTests {
    @Resource
    UserInfoService_4 userInfoService4;

    @Test
    void test4() {
        userInfoService4.saves();
    }
}
```

输出结果

```java
2024-08-25 15:00:31.305  INFO 24810 --- [pool-3-thread-1] c.e.springboot.tech.UserInfoService_4    : 子线程：pool-3-thread-1
2024-08-25 15:00:31.326  INFO 24810 --- [pool-3-thread-2] c.e.springboot.tech.UserInfoService_4    : 子线程：pool-3-thread-2
2024-08-25 15:00:31.342  INFO 24810 --- [pool-3-thread-3] c.e.springboot.tech.UserInfoService_4    : 子线程：pool-3-thread-3
2024-08-25 15:00:31.361  INFO 24810 --- [pool-3-thread-4] c.e.springboot.tech.UserInfoService_4    : 子线程：pool-3-thread-4
2024-08-25 15:00:31.379  INFO 24810 --- [pool-3-thread-5] c.e.springboot.tech.UserInfoService_4    : 子线程：pool-3-thread-5
2024-08-25 15:00:53.607  INFO 24810 --- [           main] c.e.springboot.tech.UserInfoService_4    : ============================== 主线程完成 ==============================
2024-08-25 15:00:53.632  INFO 24810 --- [           main] c.e.springboot.tech.UserInfoService_4    : StopWatch '': running time = 21372770250 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
21372770250  100%  manualTransactional
```

***

本想再次测试一下不同线程数对执行效率的影响时，发现当线程数超过10个时，执行时就报错。具体错误内容如下：

```java
Exception in thread "pool-1-thread-2" org.springframework.transaction.CannotCreateTransactionException: Could not open JDBC Connection for transaction; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30055ms.
	at org.springframework.jdbc.datasource.DataSourceTransactionManager.doBegin(DataSourceTransactionManager.java:309)
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.startTransaction(AbstractPlatformTransactionManager.java:400)
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.getTransaction(AbstractPlatformTransactionManager.java:373)
	at com.example.springbootmybatis.service.Impl.StudentServiceImpl.updateStudentsThread(StudentServiceImpl.java:58)
	at com.example.springbootmybatis.StudentTest.lambda$updateStudentWithThreadsAndTrans$3(StudentTest.java:164)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30055ms.
	at com.zaxxer.hikari.pool.HikariPool.createTimeoutException(HikariPool.java:696)
	at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:197)
	at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:162)
	at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:128)
	at org.springframework.jdbc.datasource.DataSourceTransactionManager.doBegin(DataSourceTransactionManager.java:265)
	... 7 more
```

错误的大致意思时，不能为数据库事务打开 jdbc Connection，连接在30s的时候超时了。由于前面启动的十个线程需要等待主线程完成后才能提交，所以一直占用连接未释放，造成后面的进程创建连接超时。

看错误日志中错误的来源是 **HikariPool** ，我们来重新配置一下这个连接池的参数，将最大连接数修改为100，具体配置如下：

```properties
# 连接池中允许的最小连接数。缺省值：10
spring.datasource.hikari.minimum-idle=10
# 连接池中允许的最大连接数。缺省值：10
spring.datasource.hikari.maximum-pool-size=100
# 自动提交
spring.datasource.hikari.auto-commit=true
# 一个连接idle状态的最大时长（毫秒），超时则被释放（retired），缺省:10分钟
spring.datasource.hikari.idle-timeout=30000
# 一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒
spring.datasource.hikari.max-lifetime=1800000
# 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 缺省:30秒
```

再次执行测试发现没有报错，修改线程数为20又执行了一下，同样执行成功了。



6、TransactionStatus 集合控制多线程事务【不推荐】
---------------------------------

在查询了需要教程后我们还可以使用事务集合来进行多线程事务控制，这里需要特别注意的是，事务的开启与提交必须在同一个线程内，如果在线程A开启事务，然后到线程B或者主线程中去提交线程A的事务则会报错：No value for key [HikariDataSource] bound to thread。为了避免这种情况我们只能增加代码的复杂度，使用TransactionResource来装载我们copy的事务资源。必须找到对应线程中的事务资源才能提交事务。主要代码如下：

```java
import lombok.Builder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
import org.springframework.transaction.support.TransactionSynchronization;
import org.springframework.transaction.support.TransactionSynchronizationManager;
import org.springframework.util.StopWatch;

import javax.annotation.Resource;
import javax.sql.DataSource;
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.stream.Collectors;

@Slf4j
@Service
public class UserInfoService_5 {

    @Resource
    JdbcAggregateTemplate jdbcAggregateTemplate;
    @Resource
    PlatformTransactionManager transactionManager;

    /**
     * 使用TransactionStatus集合保存所有线程中的TransactionStatus，最后在主线程中一并提交
     * 这里需要特别注意：
     * 此方式如果按照正常处理会报错：No value for key [HikariDataSource] bound to thread
     * 因为在开启新事务时，事务相关资源都被绑定到了子线程thread-cache-pool-1对应的threadLocalMap内部，
     * 而当执行事务提交代码时，commit内部需要从TransactionSynchronizationManager中获取当前事务的资源，
     * 显然我们无法从main线程对应的threadLocalMap中获取到对应的事务资源，这也就是异常抛出的原因。
     * 
     * 解决方案参考: https://www.cnblogs.com/fix200/p/18066537
     */
    List<TransactionStatus> transactionStatuses = new CopyOnWriteArrayList<>();
    /**
     * 此集合用来装载上面提到的当前事务的资源。提交或者回滚必须先找到对应的资源，不然会报错
     */
    List<TransactionResource> transactionResources = new CopyOnWriteArrayList<>();
    /**
     * 线程数量
     */
    public static final int THREAD_COUNT = 5;
    /**
     * 线程池初初始化
     */
    public static final ExecutorService THREAD_POOL = Executors.newFixedThreadPool(THREAD_COUNT);

    /**
     * 使用多线程创建1000000条数据
     * 耗时统计
     *  1 线程数：1      耗时：36s
     *  2 线程数：2      耗时：22s
     *  3 线程数：5      耗时：18s
     *  4 线程数：10     耗时：21s
     *  5 线程数：100    耗时：33s
     *  6 线程数：200    耗时：29s
     *  7 线程数：500    耗时：44s
     */
    public void saves() {

        // 模拟需要新增的用户
        List<UserInfo> userInfos = UserInfo.getUserInfos(100_0000);

        // 每个线程处理的数据量
        int dataPortionLength = (userInfos.size() + THREAD_COUNT - 1) / THREAD_COUNT;

        // 记录耗时
        StopWatch watch = new StopWatch();
        watch.start("manualTransactional");

        // 用于计算子线程提交数量
        CountDownLatch threadLatch = new CountDownLatch(THREAD_COUNT);
        // 用于判断子线程任务是否有错误
        AtomicBoolean isError = new AtomicBoolean(false);

        for (int i = 0; i < THREAD_COUNT; i++) {
            // 每个线程处理的数据
            List<UserInfo> threadData = userInfos.stream()
                    .skip(i * dataPortionLength)
                    .limit(dataPortionLength)
                    .collect(Collectors.toList());
            THREAD_POOL.execute(() -> this.savaUserInfos(threadData, threadLatch, isError));
            // CompletableFuture.runAsync(() -> this.savaUserInfos(threadData, threadLatch), THREAD_POOL);
        }


        try {
            // 倒计时锁设置超时时间 300s
            boolean await = threadLatch.await(300, TimeUnit.SECONDS);
            // 等待超时，事务回滚
            if (!await) {
                isError.set(true);
            }
        } catch (Throwable e) {
            isError.set(true);
            log.error(e.getMessage(), e);
        }

        // 发生了异常则进行回滚操作,否则提交
        if (isError.get()) {
            log.error("发生异常,全部事务回滚");
            for (int i = 0; i < transactionStatuses.size(); i++) {
                transactionResources.get(i).autoWiredTransactionResource();
                transactionManager.rollback(transactionStatuses.get(i));
                transactionResources.get(i).removeTransactionResource();
            }
        } else {
            log.info("全部事务正常提交");
            for (int i = 0; i < transactionStatuses.size(); i++) {
                transactionResources.get(i).autoWiredTransactionResource();
                transactionManager.commit(transactionStatuses.get(i));
                transactionResources.get(i).removeTransactionResource();
            }
        }

        // 关闭线程池
        THREAD_POOL.shutdown();

        log.info("============================== 主线程完成 ==============================");
        watch.stop();
        log.info(watch.prettyPrint());
    }

    public void savaUserInfos(List<UserInfo> userInfos, CountDownLatch threadLatch, AtomicBoolean taskStatus) {
        DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
        // 事物隔离级别，开启新事务，这样会比较安全些。
        definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        // 开启事务 并获得事务状态
        TransactionStatus transactionStatus = transactionManager.getTransaction(definition);
        // 使用这种方式将事务状态都放在同一个事务里面
        transactionStatuses.add(transactionStatus);
        // 继续事务资源复制,因为在sql执行是会产生新的资源对象
        transactionResources.add(TransactionResource.copyTransactionResource());
        log.info("子线程：{}", Thread.currentThread().getName());

        try {
            userInfos.forEach(jdbcAggregateTemplate::insert);
        } catch (Throwable e) {
            taskStatus.set(true);
            log.error("insert error: ", e);
        } finally {
            // 切换到主线程执行
            threadLatch.countDown();
        }
    }


    /**
     * 保存当前事务资源,用于线程间的事务资源COPY操作
     */
    @Builder
    private static class TransactionResource {
        // 事务结束后默认会移除集合中的DataSource作为key关联的资源记录
        private Map<Object, Object> resources = new HashMap<>();

        // 下面五个属性会在事务结束后被自动清理,无需我们手动清理
        private Set<TransactionSynchronization> synchronizations = new HashSet<>();

        private String currentTransactionName;

        private Boolean currentTransactionReadOnly;

        private Integer currentTransactionIsolationLevel;

        private Boolean actualTransactionActive;

        public static TransactionResource copyTransactionResource() {
            return TransactionResource.builder()
                    // 返回的是不可变集合
                    .resources(TransactionSynchronizationManager.getResourceMap())
                    // 如果需要注册事务监听者,这里记得修改--我们这里不需要,就采用默认负责--spring事务内部默认也是这个值
                    .synchronizations(new LinkedHashSet<>())
                    .currentTransactionName(TransactionSynchronizationManager.getCurrentTransactionName())
                    .currentTransactionReadOnly(TransactionSynchronizationManager.isCurrentTransactionReadOnly())
                    .currentTransactionIsolationLevel(TransactionSynchronizationManager.getCurrentTransactionIsolationLevel())
                    .actualTransactionActive(TransactionSynchronizationManager.isActualTransactionActive())
                    .build();
        }

        /**
         * 装配事务资源，为提交 or 回滚做储备
         */
        public void autoWiredTransactionResource() {
            // 绑定事务资源，注意 绑定是绑定到当前主线程上，记得最后释放交换主线程，再由主线程收回原有事务自选
            resources.forEach(TransactionSynchronizationManager::bindResource);
            // 如果需要注册事务监听者,这里记得修改--我们这里不需要,就采用默认负责--Spring事务内部默认也是这个值
            TransactionSynchronizationManager.initSynchronization();
            TransactionSynchronizationManager.setActualTransactionActive(actualTransactionActive);
            TransactionSynchronizationManager.setCurrentTransactionName(currentTransactionName);
            TransactionSynchronizationManager.setCurrentTransactionIsolationLevel(currentTransactionIsolationLevel);
            TransactionSynchronizationManager.setCurrentTransactionReadOnly(currentTransactionReadOnly);
        }

        public void removeTransactionResource() {
            Map<Object, Object> resourceMap = new HashMap<>(TransactionSynchronizationManager.getResourceMap());
            // 事务结束后默认会移除集合中的DataSource作为key关联的资源记录
            // DataSource如果重复移除,unbindResource时会因为不存在此key关联的事务资源而报错
            resources.keySet().forEach(key -> {
                if (resourceMap.containsKey(key) && !(key instanceof DataSource)) {
                    TransactionSynchronizationManager.unbindResource(key);
                }
            });
        }
    }
}
```
测试代码：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import javax.annotation.Resource;

@SpringBootTest
class ApplicationTests {
    @Resource
    UserInfoService_5 userInfoService5;

    @Test
    void test5() {
        userInfoService5.saves();
    }
}
```

输出结果：

```java
2024-08-25 15:07:21.902  INFO 25185 --- [pool-4-thread-1] c.e.springboot.tech.UserInfoService_5    : 子线程：pool-4-thread-1
2024-08-25 15:07:21.910  INFO 25185 --- [pool-4-thread-2] c.e.springboot.tech.UserInfoService_5    : 子线程：pool-4-thread-2
2024-08-25 15:07:21.926  INFO 25185 --- [pool-4-thread-3] c.e.springboot.tech.UserInfoService_5    : 子线程：pool-4-thread-3
2024-08-25 15:07:21.957  INFO 25185 --- [pool-4-thread-4] c.e.springboot.tech.UserInfoService_5    : 子线程：pool-4-thread-4
2024-08-25 15:07:22.008  INFO 25185 --- [pool-4-thread-5] c.e.springboot.tech.UserInfoService_5    : 子线程：pool-4-thread-5
2024-08-25 15:07:45.443  INFO 25185 --- [           main] c.e.springboot.tech.UserInfoService_5    : 全部事务正常提交
2024-08-25 15:07:46.424  INFO 25185 --- [           main] c.e.springboot.tech.UserInfoService_5    : ============================== 主线程完成 ==============================
2024-08-25 15:07:46.430  INFO 25185 --- [           main] c.e.springboot.tech.UserInfoService_5    : StopWatch '': running time = 23609632500 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
23609632500  100%  manualTransactional
```

由于这个中方式去前面方式相同，需要等待线程执行完成后才会提交事务，所有任会占用JDBC连接池，如果线程数量超过连接池最大数量会产生连接超时。所以在使用过程中任要控制线程数量。

***

这里需要特别需要提到为什么不推荐使用此种方式呢？因为一般我们正常业务代码使用线程池的时候，创建的线程数肯定是会大于核心线程数的。这道理第一轮的线程还没有提交，第二轮的线程又再次手动开启了事务【重复开启事务与重复提交事务也是不允许的】。所以这种情况下我们也无法使用到 TransactionStatus 即可控制多线程事务提交



六、使用 union 连接多个 select 实现批量 update
-----------------------------

有些情况写不支持，批量 update，但支持 insert 多条数据，这个时候可尝试将需要更新的数据拼接成多条 select 语句，然后使用 union 连接起来，再使用 update 关联这个数据进行 update，具体代码演示如下：

```sql
update student,(
    (select  1 as id,'teacher_A' as teacher) union
    (select  2 as id,'teacher_A' as teacher) union
    (select  3 as id,'teacher_A' as teacher) union
    (select  4 as id,'teacher_A' as teacher)
    /* ....more data ... */
) as new_teacher
set
	student.teacher=new_teacher.teacher
where
	student.id=new_teacher.id
```

这种方式在 Mysql 数据库没有配置 **allowMultiQueries=true** 也可以实现批量更新**。**



七、总结
-----------------------------------------------------------------------------------------

*   对于大批量数据库操作，使用手动事务提交可以很多程度上提高操作效率
*   多线程对数据库进行操作时，并非线程数越多操作时间越快，按上述示例大约在2-5个线程时操作时间最快。
*   对于多线程阻塞事务提交时，线程数量不能过多。
*   如果能有办法实现批量更新那是最好



# Spring 多线程事务控制各种案例汇总

> 作者：[qq_35987023](https://blog.csdn.net/qq_35987023)、来源：Spring 多线程事务控制：https://blog.csdn.net/qq_35987023/article/details/132908628

在Java多线程事务控制中，有一些注意事项和实例可以帮助你更好地理解和应用。

- 确保线程安全：在多线程环境下，确保代码是线程安全的。这可以通过使用synchronized关键字、Lock接口或Atomic类来实现。

- 事务的隔离级别：根据需要选择适当的事务隔离级别，以避免并发问题，例如脏读、不可重复读和幻读。
- 事务的传播行为：了解事务的传播行为，例如事务的提交和回滚如何影响其他事务。
- 异常处理：在多线程环境下处理异常时，需要特别小心。确保捕获和处理所有异常，并正确地处理事务回滚。

***

Spring 实现事务通过 ThreadLocal 把事务和当前线程进行了绑定。

ThreadLocal 作为本地线程变量载体，保存了当前线程的变量，并确保所有变量是线程安全的。

这些封闭隔离的变量中就包含了数据库连接，Session管理的对象以及当前事务运行的其他必要信息，而开启的新线程是获取不到这些变量和对象的。

也就是说：主线程事务与子线程事务是相互独立的。该怎么验证？

操作示例：验证事务 以及 多线程事务控制编码

```java
import cn.cjf.tt.dao.UserMapper;
import cn.cjf.tt.po.User;
import cn.cjf.tt.service.UserService;
import lombok.SneakyThrows;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;


// 使用Spring整合Junit专用的类加载器
@RunWith(SpringJUnit4ClassRunner.class)
// 加载配置文件或者配置类
@ContextConfiguration(locations = {"classpath:spring.xml"}) // 加载配置文件
public class UserTest {
    @Autowired
    private UserService userService;
    @Autowired
    private DataSourceTransactionManager transactionManager;
    @Autowired
    private UserMapper userMapper;

    /**
     * 验证数据库连接是否正常
     */
    @Test
    public void selectAllUser() {
        List<User> users = userService.selectAllUser();
        for (User i : users) {
            System.out.println(i);
        }
    }

    /**
     * 验证：能否正常插入数据
     */
    public void test() {
        final User user = new User() {{
            this.setUsername("test_" + UUID.randomUUID().toString());
            this.setPassword("123456");
            this.setCreateTime(new Date());
        }};
        userService.addUser(user);
    }

    /**
     * 验证：线程池子线程中能否正常插入数据
     */
    @Test
    public void testForBatch() throws InterruptedException {
        final ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            service.submit(new Runnable() {
                @SneakyThrows
                @Override
                public void run() {
                    test1();
                }
            });
        }
        Thread.sleep(5000);
    }

    /**
     * 验证：正常插入数据，抛出异常后，注解事务是否回滚
     */
    @Transactional(rollbackFor = Exception.class)
    @Test
    public void test1() throws Exception {
        final User user = new User() {{
            this.setUsername("test_" + UUID.randomUUID().toString());
            this.setPassword("123456");
            this.setCreateTime(new Date());
        }};
        userService.addUser(user);
        if (true) {
            throw new Exception();
        }
    }

    /**
     * 验证：正常插入数据，抛出异常后，手动事务是否回滚
     */
    @Test
    public void test11() {
        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
        final TransactionStatus transaction = transactionManager.getTransaction(dd);
        try {
            final User user = new User() {{
                this.setUsername("test_" + UUID.randomUUID().toString());
                this.setPassword("123456");
                this.setCreateTime(new Date());
            }};
            userService.addUser(user);
            System.out.println(user);
            if (true) {
                throw new Exception();
            }
            // User(id=13, username=test1694675733277, password=123456, salt=null, token=null, isEnabled=null, createTime=Thu Sep 14 15:15:33 CST 2023, modifiedTime=null)
            transactionManager.commit(transaction);
            System.out.println("---------------------" + Thread.currentThread().getName() + "事务提交");
        } catch (Exception e) {
            e.printStackTrace();
            transactionManager.rollback(transaction);
            System.out.println("---------------------" + Thread.currentThread().getName() + "事务回滚");
        }
    }


    /**
     * 验证：主线程事务，是否能影响到子线程事务
     */
    @Transactional(rollbackFor = Exception.class)
    @Test
    public void test2() throws Exception {
        final User user = new User() {{
            this.setUsername("test_" + UUID.randomUUID().toString());
            this.setPassword("123456");
            this.setCreateTime(new Date());
        }};
        userService.addUser(user);
        System.out.println("---------------------" + Thread.currentThread().getName() + "：" + user);
        final ExecutorService service = Executors.newFixedThreadPool(5);
        List<Integer> idList = new ArrayList<>();
        idList.add(user.getId());
        for (int i = 0; i < 5; i++) {
            service.submit(new Runnable() {
                @Override
                public void run() {
                    final User user = new User() {{
                        this.setUsername("test_" + UUID.randomUUID().toString());
                        this.setPassword("123456");
                        this.setCreateTime(new Date());
                    }};
                    userService.addUserForTransaction(user);
                    System.out.println("---------------------" + Thread.currentThread().getName() + "：" + user);
                    final Integer id = user.getId();
                    idList.add(id);
                }
            });
        }

        Thread.sleep(5000);
        try {
            throw new Exception();
        } finally {
            for (int i = 0; i < idList.size(); i++) {
                final Integer id = idList.get(i);
                final User po = userMapper.selectByPrimaryKey(id);
                if (po == null) {
                    System.out.println("---------------------id:" + id + "事务回滚");
                } else {
                    System.out.println("---------------------id:" + id + "事务提交");
                }
                // 主线程事务未结束
                // 实际主线程事务回滚了，但子线程事务未回滚
            }
        }
    }

    /**
     * 验证：主线程事务，未能影响到子线程事务，是因为子线程的事务传播行为影响
     */
    @Transactional(rollbackFor = Exception.class)
    @Test
    public void test21() throws Exception {
        final User user = new User() {{
            this.setUsername("test_" + UUID.randomUUID().toString());
            this.setPassword("123456");
            this.setCreateTime(new Date());
        }};
        userService.addUser(user);
        System.out.println("---------------------" + Thread.currentThread().getName() + "：" + user);
        final ExecutorService service = Executors.newFixedThreadPool(5);
        List<Integer> idList = new ArrayList<>();
        idList.add(user.getId());
        for (int i = 0; i < 5; i++) {
            service.submit(new Runnable() {
                @Override
                public void run() {
                    final User user = new User() {{
                        this.setUsername("test_" + UUID.randomUUID().toString());
                        this.setPassword("123456");
                        this.setCreateTime(new Date());
                    }};
                    userService.addUserForNestedTransaction(user);
                    System.out.println("---------------------" + Thread.currentThread().getName() + "：" + user);
                    final Integer id = user.getId();
                    idList.add(id);
                }
            });
        }

        Thread.sleep(5000);
        try {
            throw new Exception();
        } finally {
            for (int i = 0; i < idList.size(); i++) {
                final Integer id = idList.get(i);
                final User po = userMapper.selectByPrimaryKey(id);
                if (po == null) {
                    System.out.println("---------------------id:" + id + "事务回滚");
                } else {
                    System.out.println("---------------------id:" + id + "事务提交");
                }
                // 主线程事务未结束
                // 实际主线程事务回滚了，但子线程事务未回滚
            }
        }
    }

    /**
     * 验证：主线程事务，未能影响到子线程事务
     * 主线程手动控制事务，与注解自动控制事务，结果是否依然是，主线程事务不能影响到子线程事务
     */
    @Test
    public void test22() throws Exception {
        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
        final TransactionStatus transaction = transactionManager.getTransaction(dd);
        List<Integer> idList = new ArrayList<>();
        try {
            final User user = new User() {{
                this.setUsername("test_" + UUID.randomUUID().toString());
                this.setPassword("123456");
                this.setCreateTime(new Date());
            }};
            userService.addUser(user);
            System.out.println("---------------------" + Thread.currentThread().getName() + "：" + user);
            final ExecutorService service = Executors.newFixedThreadPool(5);
            idList.add(user.getId());
            for (int i = 0; i < 5; i++) {
                service.submit(new Runnable() {
                    @Override
                    public void run() {
                        final User user = new User() {{
                            this.setUsername("test_" + UUID.randomUUID().toString());
                            this.setPassword("123456");
                            this.setCreateTime(new Date());
                        }};
                        userService.addUserForNestedTransaction(user);
                        System.out.println("---------------------" + Thread.currentThread().getName() + "：" + user);
                        final Integer id = user.getId();
                        idList.add(id);
                    }
                });
            }

            Thread.sleep(5000);
            if (true) {
                throw new Exception();
            }

            // User(id=13, username=test1694675733277, password=123456, salt=null, token=null, isEnabled=null, createTime=Thu Sep 14 15:15:33 CST 2023, modifiedTime=null)
            System.out.println("---------------------" + Thread.currentThread().getName() + "事务提交");
        } catch (Exception e) {
            e.printStackTrace();
            transactionManager.rollback(transaction);
            System.out.println("---------------------" + Thread.currentThread().getName() + "事务回滚");
        }

        for (int i = 0; i < idList.size(); i++) {
            final Integer id = idList.get(i);
            final User po = userMapper.selectByPrimaryKey(id);
            if (po == null) {
                System.out.println("---------------------id:" + id + "事务回滚");
            } else {
                System.out.println("---------------------id:" + id + "事务提交");
            }
            // 主线程事务已结束
            // 主线程事务回滚了，但子线程事务未回滚
        }
    }

    /**
     * 验证：主线程事务，未能影响到子线程事务
     * 主线程手动控制事务，子线程也手动控制事务，结果是否依然是，主线程事务不能影响到子线程事务
     */
    @Test
    public void test23() throws InterruptedException {
        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
        final TransactionStatus transaction = transactionManager.getTransaction(dd);
        List<Integer> idList = new ArrayList<>();
        try {
            final User user = new User() {{
                this.setUsername("test_" + UUID.randomUUID().toString());
                this.setPassword("123456");
                this.setCreateTime(new Date());
            }};
            userService.addUser(user);
            idList.add(user.getId());
            final ExecutorService service = Executors.newFixedThreadPool(5);
            for (int i = 0; i < 5; i++) {
                service.submit(new Runnable() {
                    @Override
                    public void run() {
                        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
                        final TransactionStatus transaction = transactionManager.getTransaction(dd);
                        try {
                            final User user = new User() {{
                                this.setUsername("test_" + UUID.randomUUID().toString());
                                this.setPassword("123456");
                                this.setCreateTime(new Date());
                            }};
                            userService.addUser(user);
                            System.out.println(user);
                            final Integer id = user.getId();
                            idList.add(id);
                            // User(id=13, username=test1694675733277, password=123456, salt=null, token=null, isEnabled=null, createTime=Thu Sep 14 15:15:33 CST 2023, modifiedTime=null)
                            transactionManager.commit(transaction);
                            System.out.println("---------------------" + Thread.currentThread().getName() + "事务提交");
                        } catch (Exception e) {
                            e.printStackTrace();
                            transactionManager.rollback(transaction);
                            System.out.println("---------------------" + Thread.currentThread().getName() + "事务回滚");
                        }
                    }
                });
            }

            Thread.sleep(5000);
            if (true) {
                throw new Exception();
            }

            // User(id=13, username=test1694675733277, password=123456, salt=null, token=null, isEnabled=null, createTime=Thu Sep 14 15:15:33 CST 2023, modifiedTime=null)
            System.out.println("---------------------" + Thread.currentThread().getName() + "事务提交");
        } catch (Exception e) {
            e.printStackTrace();
            transactionManager.rollback(transaction);
            System.out.println("---------------------" + Thread.currentThread().getName() + "事务回滚");
        }

        for (int i = 0; i < idList.size(); i++) {
            final Integer id = idList.get(i);
            final User po = userMapper.selectByPrimaryKey(id);
            if (po == null) {
                System.out.println("---------------------id:" + id + "事务回滚");
            } else {
                System.out.println("---------------------id:" + id + "事务提交");
            }
            // 主线程事务已结束
            // 主线程事务回滚了，但子线程事务未回滚
        }
    }

    /**
     * 验证结果：主线程事务不能影响到子线程事务
     * <p>
     * 主线程，子线程控制各自事务，等待一起提交
     */
    @Test
    public void test3() throws InterruptedException {
        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
        final TransactionStatus transaction = transactionManager.getTransaction(dd);
        List<Integer> idList = new ArrayList<>();
        int time = 5;
        CountDownLatch cdl = new CountDownLatch(time);
        AtomicBoolean flag = new AtomicBoolean(true);
        try {
            final User user = new User() {{
                this.setUsername("test_" + UUID.randomUUID().toString());
                this.setPassword("123456");
                this.setCreateTime(new Date());
            }};
            userService.addUser(user);
            idList.add(user.getId());
            final ExecutorService service = Executors.newFixedThreadPool(time);
            for (int i = 0; i < time; i++) {
                service.submit(new Runnable() {
                    @SneakyThrows
                    @Override
                    public void run() {
                        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
                        final TransactionStatus transaction = transactionManager.getTransaction(dd);
                        try {
                            final User user = new User() {{
                                this.setUsername("test_" + UUID.randomUUID().toString());
                                this.setPassword("123456");
                                this.setCreateTime(new Date());
                            }};
                            userMapper.insertSelective(user);
                            idList.add(user.getId());
                            System.out.println("---------------" + Thread.currentThread().getName() + "--执行成功");
                        } catch (Exception e) {
                            e.printStackTrace();
                            System.out.println("---------------" + Thread.currentThread().getName() + "--执行失败，等待事务回滚");
                            flag.set(false);
                        } finally {
                            System.out.println("---------------" + Thread.currentThread().getName() + "--等待");
                            cdl.countDown();
                            cdl.await();
                            if (flag.get()) {
                                System.out.println("---------------" + Thread.currentThread().getName() + "--提交事务");
                                transactionManager.commit(transaction);
                            } else {
                                System.out.println("---------------" + Thread.currentThread().getName() + "--回滚事务");
                                transactionManager.rollback(transaction);
                            }

                            System.out.println("---------------" + Thread.currentThread().getName() + "--End");
                        }
                    }
                });
            }

        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("---------------" + Thread.currentThread().getName() + "--执行失败，等待事务回滚");
            flag.set(false);
        } finally {
            System.out.println("---------------" + Thread.currentThread().getName() + "--等待");
            cdl.await();
            if (flag.get()) {
                System.out.println("---------------" + Thread.currentThread().getName() + "--提交事务");
                transactionManager.commit(transaction);
            } else {
                System.out.println("---------------" + Thread.currentThread().getName() + "--回滚事务");
                transactionManager.rollback(transaction);
            }
        }

        System.out.println("---------------" + Thread.currentThread().getName() + "--End");

        for (int i = 0; i < idList.size(); i++) {
            final Integer id = idList.get(i);
            final User po = userMapper.selectByPrimaryKey(id);
            if (po == null) {
                System.out.println("---------------------id:" + id + "事务回滚");
            } else {
                System.out.println("---------------------id:" + id + "事务提交");
            }
            // 主线程事务已结束
            // 主线程事务回滚了，子线程事务也回滚
        }
    }

    /**
     * 验证结果：主线程事务不能影响到子线程事务
     * <p>
     * 主线程，子线程控制各自事务，等待一起提交
     * 验证，主线程异常，子线程未异常，事务都回滚了
     */
    @Test
    public void test31() throws InterruptedException {
        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
        final TransactionStatus transaction = transactionManager.getTransaction(dd);
        List<Integer> idList = new ArrayList<>();
        int time = 5;
        CountDownLatch cdl = new CountDownLatch(time);
        AtomicBoolean flag = new AtomicBoolean(true);
        try {
            final User user = new User() {{
                this.setUsername("test_" + UUID.randomUUID().toString());
                this.setPassword("123456");
                this.setCreateTime(new Date());
            }};
            userService.addUser(user);
            idList.add(user.getId());
            final ExecutorService service = Executors.newFixedThreadPool(time);
            for (int i = 0; i < time; i++) {
                //                int finalI = i;
                service.submit(new Runnable() {
                    @SneakyThrows
                    @Override
                    public void run() {
                        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
                        final TransactionStatus transaction = transactionManager.getTransaction(dd);
                        try {
                            final User user = new User() {{
                                this.setUsername("test_" + UUID.randomUUID().toString());
                                this.setPassword("123456");
                                this.setCreateTime(new Date());
                            }};
                            userMapper.insertSelective(user);
                            idList.add(user.getId());
                            // 最后一个提交的任务，抛出异常；注释掉会全部完成，否则全部回滚
                            //                            if (finalI == time - 1) {
                            //                                throw new RuntimeException();
                            //                            }
                            // User(id=13, username=test1694675733277, password=123456, salt=null, token=null, isEnabled=null, createTime=Thu Sep 14 15:15:33 CST 2023, modifiedTime=null)
                            System.out.println("---------------" + Thread.currentThread().getName() + "--执行成功");
                        } catch (Exception e) {
                            e.printStackTrace();
                            System.out.println("---------------" + Thread.currentThread().getName() + "--执行失败，等待事务回滚");
                            flag.set(false);
                        } finally {
                            System.out.println("---------------" + Thread.currentThread().getName() + "--等待");
                            cdl.countDown();
                            cdl.await();
                            if (flag.get()) {
                                System.out.println("---------------" + Thread.currentThread().getName() + "--提交事务");
                                transactionManager.commit(transaction);
                            } else {
                                System.out.println("---------------" + Thread.currentThread().getName() + "--回滚事务");
                                transactionManager.rollback(transaction);
                            }

                            System.out.println("---------------" + Thread.currentThread().getName() + "--End");
                        }
                    }
                });
            }

            if (true) {
                throw new Exception();
            }

        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("---------------" + Thread.currentThread().getName() + "--执行失败，等待事务回滚");
            flag.set(false);
        } finally {
            System.out.println("---------------" + Thread.currentThread().getName() + "--等待");
            cdl.await();
            if (flag.get()) {
                System.out.println("---------------" + Thread.currentThread().getName() + "--提交事务");
                transactionManager.commit(transaction);
            } else {
                System.out.println("---------------" + Thread.currentThread().getName() + "--回滚事务");
                transactionManager.rollback(transaction);
            }
        }

        System.out.println("---------------" + Thread.currentThread().getName() + "--End");

        for (int i = 0; i < idList.size(); i++) {
            final Integer id = idList.get(i);
            final User po = userMapper.selectByPrimaryKey(id);
            if (po == null) {
                System.out.println("---------------------id:" + id + "事务回滚");
            } else {
                System.out.println("---------------------id:" + id + "事务提交");
            }
            // 主线程事务已结束
            // 主线程事务回滚了，子线程事务也回滚
        }
    }

    /**
     * 验证结果：主线程事务不能影响到子线程事务
     * <p>
     * 主线程，子线程控制各自事务，等待一起提交
     * 验证，主线程未异常，子线程异常，事务都回滚了
     */
    @Test
    public void test32() throws InterruptedException {
        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
        final TransactionStatus transaction = transactionManager.getTransaction(dd);
        List<Integer> idList = new ArrayList<>();
        int time = 5;
        CountDownLatch cdl = new CountDownLatch(time);
        AtomicBoolean flag = new AtomicBoolean(true);
        try {
            final User user = new User() {{
                this.setUsername("test_" + UUID.randomUUID().toString());
                this.setPassword("123456");
                this.setCreateTime(new Date());
            }};
            userService.addUser(user);
            idList.add(user.getId());
            final ExecutorService service = Executors.newFixedThreadPool(time);
            for (int i = 0; i < time; i++) {
                int finalI = i;
                service.submit(new Runnable() {
                    @SneakyThrows
                    @Override
                    public void run() {
                        DefaultTransactionDefinition dd = new DefaultTransactionDefinition();
                        final TransactionStatus transaction = transactionManager.getTransaction(dd);
                        try {
                            final User user = new User() {{
                                this.setUsername("test_" + UUID.randomUUID().toString());
                                this.setPassword("123456");
                                this.setCreateTime(new Date());
                            }};
                            userMapper.insertSelective(user);
                            idList.add(user.getId());
                            // 最后一个提交的任务，抛出异常；注释掉会全部完成，否则全部回滚
                            if (finalI == time - 1) {
                                throw new RuntimeException();
                            }
                            // User(id=13, username=test1694675733277, password=123456, salt=null, token=null, isEnabled=null, createTime=Thu Sep 14 15:15:33 CST 2023, modifiedTime=null)
                            System.out.println("---------------" + Thread.currentThread().getName() + "--执行成功");
                        } catch (Exception e) {
                            e.printStackTrace();
                            System.out.println("---------------" + Thread.currentThread().getName() + "--执行失败，等待事务回滚");
                            flag.set(false);
                        } finally {
                            System.out.println("---------------" + Thread.currentThread().getName() + "--等待");
                            cdl.countDown();
                            cdl.await();
                            if (flag.get()) {
                                System.out.println("---------------" + Thread.currentThread().getName() + "--提交事务");
                                transactionManager.commit(transaction);
                            } else {
                                System.out.println("---------------" + Thread.currentThread().getName() + "--回滚事务");
                                transactionManager.rollback(transaction);
                            }

                            System.out.println("---------------" + Thread.currentThread().getName() + "--End");
                        }
                    }
                });
            }

            //            if (true) {
            //                throw new Exception();
            //            }

        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("---------------" + Thread.currentThread().getName() + "--执行失败，等待事务回滚");
            flag.set(false);
        } finally {
            System.out.println("---------------" + Thread.currentThread().getName() + "--等待");
            cdl.await();
            if (flag.get()) {
                System.out.println("---------------" + Thread.currentThread().getName() + "--提交事务");
                transactionManager.commit(transaction);
            } else {
                System.out.println("---------------" + Thread.currentThread().getName() + "--回滚事务");
                transactionManager.rollback(transaction);
            }
        }

        System.out.println("---------------" + Thread.currentThread().getName() + "--End");

        for (int i = 0; i < idList.size(); i++) {
            final Integer id = idList.get(i);
            final User po = userMapper.selectByPrimaryKey(id);
            if (po == null) {
                System.out.println("---------------------id:" + id + "事务回滚");
            } else {
                System.out.println("---------------------id:" + id + "事务提交");
            }
            // 主线程事务已结束
            // 主线程事务回滚了，子线程事务也回滚
        }
    }
}
```

