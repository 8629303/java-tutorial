# SpringBoot Cache 整合 Redis 缓存框架

> 作者：[次时代小羊](https://herenpeng.blog.csdn.net/)、来源：SpringBoot Cache 整合 Redis 缓存框架：https://blog.csdn.net/qq_45193304/article/details/131727266
>
> 作者：[扛麻袋的少年](https://blog.csdn.net/lzb348110175)、来源：@Cacheable注解属性介绍：https://blog.csdn.net/lzb348110175/article/details/105349109
>
> SpringCache 官方文档：https://docs.spring.io/spring-framework/reference/integration/cache.html

SpringBoot 提供了 SpringBootCache 框架作为 SpringBoot 的缓存统一框架，使用 SpringBootCache ，我们可以使用注解的方式使用缓存，从而避免了写一些重复代码的问题。

SpringBoot 缓存，默认使用的是 **ConcurrentMap** 的方式来实现的，然而我们在项目中并不会这么使用。我们经常会引入第三方缓存框架，来完成对数据的缓存操作。比如说：Redis 。本文就来介绍 SpringBoot 整合 Redis 来实现数据的缓存。

## 1、引入依赖

首先我们需要引入相关的依赖，`Cache` 是缓存框架的核心依赖，不过该框架只提供了各种操作的接口集合，并没有具体的实现。

而接口之外，SpringBoot 提供了各种具体的实现，比如：`RedisCache`,`EhCache`,`ConcurrentMapCache`等等。

Redis 依赖是为了使用 Redis 作为缓存而引入的，如果不使用 Redis 作为缓存，也可以将 Redis 的依赖替换为其他实现。

Jackson 是作为序列化工具。

*   Maven

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.7.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.7.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.7.3</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

*   Gradle

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-cache:2.7.3'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis:2.7.3'
    implementation 'org.springframework.boot:spring-boot-starter-test:2.7.3'
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.13.3'
}
```



## 2、配置 Redis

因为我们引入了 SpringBoot Redis 的依赖，Spring 会默认使用 Redis 作为我们的缓存，所以需要配置 Redis 的信息。

*   application.yaml

```yaml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
```

*   RedisCacheManager

```java
@Configuration
public class RedisCacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        // 使用缓存的默认配置
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        // 使用 GenericJackson2JsonRedisSerializer 作为序列化器
        config = config.serializeValuesWith(
            RedisSerializationContext.SerializationPair.fromSerializer(
                new GenericJackson2JsonRedisSerializer()));
        RedisCacheManager.RedisCacheManagerBuilder builder =
            RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(config);
        return builder.build();
    }
}
```



## 3、启动类注解

使用 SpringBootCache 作为缓存框架，我们需要在 SpringBoot 的启动类上添加 `@EnableCaching` 注解，这样才能够启用缓存框架。

```java
@EnableCaching
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }
}
```



## 4、注解开发

我们只需要在方法上面添加相关的注解，即可使用缓存逻辑，从而提升服务性能。

### 1、@Cacheable（添加缓存）

该注解可以注解在方法和类上，当被注解的类下的方法或者被注解的方法被调用时，会先去缓存中查看是否有缓存数据，有缓存数据则直接返回缓存数据，无缓存数据则调用该方法，获取该方法的返回值，并将该返回值存入缓存中。

```java
@Cacheable(value = "users", key = "#id")
public User getUserById(Long id) {
    // 从数据库中读取用户信息
}
```



### 2、@CachePut（更新缓存）

该注解可以注解在方法和类上，当被注解的类下的方法或者被注解的方法被调用时，不管缓存中是否有对应的缓存数据，都会将该方法的返回值更新（插入或覆盖）到缓存中。

```java
@CachePut(value = "users", key = "#user.id")
public User updateUser(User user) {
    // 更新数据库中的用户信息
}
```



### 3、@CacheEvict（删除缓存）

该注解可以注解在方法和类上，当被注解的类下的方法或者被注解的方法被调用时，会直接删除缓存中的缓存数据。

```java
@CacheEvict(value = "users", key = "#id")
public void deleteUserById(Long id) {
    // 删除数据库中的用户信息
}
```



### 4、@Caching（组合注解）

该注解可以将多个缓存注解组合在一起使用。例如：

```java
@Caching(
    cacheable = @Cacheable(value = "users", key = "#id"),
    put = @CachePut(value = "users", key = "#result.id"),
    evict = @CacheEvict(value = "allUsers", allEntries = true)
)
public User getUserById(Long id) {
    // 从数据库中读取用户信息
}
```



## 5、注解属性

`@Cacheable`,`@CachePut`,`@CacheEvict`三个注解都有一些属性。

### 1、cacheNames/value

用来指定缓存组件的名字。

```java
@Cacheable(cacheNames = "user")
public User selectById(Integer id) {
    return new User(id);
}
```



### 2、key

缓存数据时使用的 key，可以用它来指定。默认是使用方法参数的值。（这个 key 你必须使用 spEL 表达式来编写）

```java
@Cacheable(cacheNames = "user", key = "#id")
public User selectById(Integer id) {
    return new User(id);
}
```

使用以上的配置，如果入参的 `id` 值为 `1`，`Redis` 会存入一个 key 为 `user::1` 数据。

```java
@Cacheable(cacheNames = "user", key = "#root.methodName")
public User selectById(Integer id) {
    return new User(id);
}
```

使用以上的配置，如果入参的 `id` 值为 `1`，`Redis` 会存入一个 key 为 `user::selectById` 数据。



### 3、keyGenerator

key 的生成器。 key 和 keyGenerator 二选一使用。

```java
@Bean
public KeyGenerator keyGenerator() {
    return (target, method, params) -> Arrays.asList(params).toString();
}
```

使用以上的配置，如果入参的 `id` 值为 `1`，`Redis` 会存入一个 key 为 `user::[1]` 数据。



### 4、condition

可以用来指定符合条件的情况下才缓存。

```java
@Cacheable(cacheNames = "user", key = "#id", condition = "#id > 1")
public User selectById(Integer id) {
    return new User(id);
}

@Cacheable(cacheNames = "user", key = "#id", condition = "#id != null")
public User selectById(Integer id) {
    return new User(id);
}
```



### 5、unless

该属性相当于 `!condition` 属性。否定缓存。当 unless 指定的条件为 true ，方法的返回值就不会被缓存。当然你也可以获取到结果进行判断。（通过 #result 获取方法结果）

```java
@Cacheable(cacheNames = "user", key = "#id", unless = "#id > 1")
public User selectById(Integer id) {
    return new User(id);
}
```



### 6、cacheManager

可以用来指定缓存管理器。从哪个缓存管理器里面获取缓存。针对不同的缓存技术，需要实现不同的 cacheManager，Spring 也为我们定义了如下的一些 cacheManger 实现（）

| CacheManger               | 描述                                                   |
| ------------------------- | ------------------------------------------------------ |
| SimpleCacheManager        | 使用简单的Collection来存储缓存，主要用于测试           |
| ConcurrentMapCacheManager | 使用ConcurrentMap作为缓存技术（默认）                  |
| NoOpCacheManager          | 测试用                                                 |
| EhCacheCacheManager       | 使用EhCache作为缓存技术，以前在hibernate的时候经常用   |
| GuavaCacheManager         | 使用google guava的GuavaCache作为缓存技术               |
| HazelcastCacheManager     | 使用Hazelcast作为缓存技术                              |
| JCacheCacheManager        | 使用JCache标准的实现作为缓存技术，如Apache Commons JCS |
| RedisCacheManager         | 使用Redis作为缓存技术                                  |

### 7、sync

该属性用来指定`是否使用异步模式`，该属性默认值为 false，默认为同步模式。`异步模式指定 sync = true 即可，异步模式下 unless 属性不可用。`

当 sync 设置为 true 时，多个线程同时访问同一个未缓存的值时，会进行同步控制，以防止并发地加载相同的数据。

- 通常在缓存值的生成非常耗时时使用，以避免多个线程重复加载同一个值。

```java
@Cacheable(value = "booksCache", sync = true)
public Book findBookByIsbn(String isbn) {
    return bookRepository.findByIsbn(isbn);
}
```



## 6、SpEL 表达式

| 名字          | 位置               | 描述                                                         | 示例                   |
| ------------- | ------------------ | ------------------------------------------------------------ | ---------------------- |
| methodName    | root object        | 当前被调用的方法名                                           | #root.methodName       |
| method        | root object        | 当前被调用的方法                                             | #root.method.name      |
| target        | root object        | 当前被调用的目标对象                                         | #root.target           |
| targetClass   | root object        | 当前被调用的目标对象类                                       | #root.targetClass      |
| args          | root object        | 当前被调用的方法的参数列表                                   | #root.args\[0\]        |
| caches        | root object        | 当前方法调用使用的缓存列表(如 @Cacheable(value=\[“cache1”,"cache2J)，则有两个 cache) | #root.caches\[0\].name |
| argument name | evaluation context | 方法参数的名字，可以直接使用 #参数名，也可以使用 #p0 或 #a0 的形式，0代表参数的索引。 | #id、#p0、#a0          |
| result        | evaluation context | 方法执行后的返回值 (仅当方法执行后的判断有效)                | #result                |

