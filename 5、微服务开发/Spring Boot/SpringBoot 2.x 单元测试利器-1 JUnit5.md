> 作者：macrozheng，来源：https://mp.weixin.qq.com/s/dnDuZZvqrXbhqW1B9VRzAg

## 一、JUnit5 前言

> JUnit 5官方文档：https://junit.org/junit5/docs/current/user-guide/

JUnit是一款Java语言的单元测试框架，目前大多数Java开发环境都已经支持它了。JUnit测试也就是所谓的白盒测试，在程序员知道程序内部逻辑的基础上进行的测试，使用JUnit能让我们快速地完成单元测试。Spring Boot Test将JUnit和其他测试框架结合起来，提供了便捷高效的测试手段，从 SpringBoot 2.2.0 之后，Junit5 已经成为了默认的 Junit 版本。有了 JUnit Vintage，从 Junit4 迁移到 Junit5 的成本极低。所以本文就直接针对 Junit5 开始了。

今天聊聊如何在 SpringBoot 中集成 Junit5、MockMvc、Mocktio。Junit5 是现在 Java 栈中应用最广的测试框架，Junit4 升级到 Junit5 之后，除了增加 Java8 的很多特性，做了很多功能增强，在结构上做了优化调整，拆分了很多不同的模块，可以按需引入。

JUnit5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

- JUnit Platform：在 JVM 上启动测试框架
- JUnit Jupiter：在 JUnit5 中编写测试和扩展
- JUnit Vintage：提供运行基于 JUnit3 和 JUnit4 的测试引擎（为了兼容老项目）

注意：SpringBoot 2.4 以上版本移除了默认对 Vintage 的依赖。如果需要兼容 JUnit4.x 版本,需要自行引入（不能使用junit4的功能 @Test）



### 1、JUnit5 的新特性

新特性主要有以下几点：

- 模块化结构集合（ JUnit Platform + JUnit Jupiter + JUnit Vintage） 与 JUnit 4 不同，JUnit 5不再是单个库，而是模块化结构的集合，整个 API 分成了：自己的模块、引擎、Launcher、针对 Gradle 和 Surefire 的集成模块。
- 分组断言（Grouped Assertions） 分组断言允许执行一组断言，且会一起报告。要记得在 JUnit 4中，我们被告诫不要在一个测试中放入多个断言，以避免某些断言没有得到执行。现在，在 JUnit 5 中使用分组断言就无需再顾虑这个避讳了。
- 断言预期（assertThrows、equalsThrows） 对 JUnit 4的另一个改进是断言预期的异常。不再是以前那种把预期的异常类型放入 `@Test` 注释，或者是用 `try-catch`包裹代码，JUnit 5使用 assertThrows 和 equalsThrows 断言。
- 禁用标签（Disabled） 假设、标签和禁止测试是 JUnit 4 的特性，在 JUnit 5中仍然得以保留。不同的是假设中也支持 Lambda 表达式，假设的思想是如果假设条件没有得到满足，那么跳过测试执行。标签 Tags 等同于JUnit 4的测试分类的概念，可以对测试类和方法进行分类。JUnit 4禁止测试使用了 `@Ignore` 注释，而在 JUnit 5 中则使用 `@Disabled` 注释。
- 扩展 API JUnit 5提供了一套新的扩展 API，取代了以前的 `@RunWith` 和 `@Rule` 扩展机制。JUnit 4的测试类被限制到仅有一个 Runner 上，而新的扩展模型则允许一个类或方法 keyii 注册到多种扩展。
- Hamcrest 风格 JUnit 5 支持 Hamcrest 匹配和 AssertJ 断言库，



### 2、JUnit4 or JUnit5 对比

JUnit5与JUnit4 注解有所变化，具体可以参考官方文档：https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations

JUnit5 对比 JUnit4，升级改造了不少东西，从架构，到包、类等，看下官方，比如@Before和@After不再支持，使用@BeforeEach和@AfterEach进行替代，其他的都类似。而JUnit4常用的@RunWith，在JUnit5中由@ExtendWith替代，并且是在有个性化扩展需要的时候再添加就可以，一般只需要加一个@SpringBootTest就可以哈皮的开启单元测试了。

**JUnit 4 和 JUnit 5 常用注解说明表，如下所示：**

| Unit4        | JUnit5             |                             说明                             |
| :----------- | :----------------- | :----------------------------------------------------------: |
| @Test        | @Test              |                      定义测试方法即用例                      |
| @BeforeClass | @BeforeAll         |               在当前类中的所有测试方法之前执行               |
| @AfterClass  | @AfterAll          |               在当前类中的所有测试方法之后执行               |
| @Before      | @BeforeEach        |                     在每个测试用例前执行                     |
| @After       | @AfterEach         |                     在每个测试用例后执行                     |
| @Ignore      | @Disabled          |                用于禁用一个测试类或测试方法。                |
| @Category    | @Tag               | Tagging 和 filtering，用于声明过滤测试的 tags，该注解可以用在方法或类上 |
| @Parameters  | @ParameterizedTest |                 表示该方法是一个参数化测试。                 |
| @RunWith     | @ExtendWith        | @Runwith、@ExtendWith 就是放在测试类名之前，用来确定这个类怎么运行的 |
| @Rule        | @ExtendWith        | @Rule 是一组实现了 TestRule 接口的共享类，<br />提供了验证、监视 TestCase 和外部资源管理等能力 |
| @ClassRule   | @ExtendWith        | @ClassRule 用于测试类中的静态变量，<br />必须是 TestRule 接口的实例，且访问修饰符必须为 public |

断言：

1. Junit 4 ：org.junit.Assert
2. Junit 5：org.junit.jupiter.Assertions

### 3、JUnit 4 单元测试举例

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

```java
/**
 * 在JUnit 4中，使用SprinBootg框架构建测试看起来是这样的：
 * @SpringBootTest + @RunWith(SpringJUnit4ClassRunner.class)
 * @SpringBootTest + @RunWith(SpringTest.class)
 */
public class JUnit4DemonTest {
    @BeforeClass
    public static void initAll() {
        // TODO：测试类运行前初始化代码
    }
    @AfterClass
    public static void tearDownAll() {
        // TODO：测试类运行后清理代码
    }
    @Before
    public void init() {
        // TODO：测试方法运行前初始化代码
    }
    @After
    public void tearDown() {
        // TODO：测试方法运行后清理码
    }
    @Test
    public void succeedingTest() {
        // TODO：正面测试用例代码
    }
    @Test
    public void failingTest() {
        // TODO：负面测试用例代码
    }
}
```



### 4、JUnit 5单元测试举例

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.2.0</version>
    <scope>test</scope>
</dependency>
```

```java
/**
 * 在JUnit 5中，你可以用如下注解启动SpringBoot：
 * @SpringBootTest
 */
public class JUnit5DemonTest {
    @BeforeAll
    static void initAll() {
        // TODO：测试类运行前初始化代码
    }
    @AfterAll
    static void tearDownAll() {
        // TODO：测试类运行后清理代码
    }
    @BeforeEach
    void init() {
        // TODO：测试方法运行前初始化代码
    }
    @AfterEach
    void tearDown() {
        // TODO：测试方法运行后清理代码
    }
    @Test
    public void succeedingTest() {
        // TODO：正面测试用例代码
    }
    @Test
    public void failingTest() {
        // TODO：负面测试用例代码
    }
}
```



## 二、常用注解

在使用SpringBoot Test之前，我们先来了解下JUnit5它常用的注解

1、如下主要是整合SpringBoot的注解，也是最常用的

| 注解                  | 作用                                                         |
| :-------------------- | :----------------------------------------------------------- |
| @SpringBootTest       | 用于指定测试类启用SpringBoot Test，默认会提供Mock环境        |
| @ExtendWith           | 如果只想启用Spring环境进行简单测试，不想启用SpringBoot环境，可以配置扩展为：SpringExtension |
| @SpringJunitConfig    | 可以只指定需要加载到Spring容器中的类，不需要的类不会加载到spring容器中，此属于Spring的注解 |
| @ContextConfiguration | 指定需要加载到Spring中的类，与@SpringJunitConfig类似，需要搭配@ExtendWith使用 |
| @Test                 | 指定方法为测试方法                                           |
| @TestMethodOrder      | 用于配置测试类中方法的执行顺序策略，配置为OrderAnnotation时，按@Order顺序执行 |
| @Order                | 用于配置方法的执行顺序，数字越低执行顺序越高                 |
| @DisplayName          | 用于指定测试类和测试方法的别名                               |
| @BeforeAll            | 在测试类的所有测试方法前执行一次，可用于全局初始化           |
| @AfterAll             | 在测试类的所有测试方法后执行一次，可用于全局销毁资源         |
| @BeforeEach           | 在测试类的每个测试方法前都执行一次                           |
| @AfterEach            | 在测试类的每个测试方法后都执行一次                           |
| @Disabled             | 禁用测试方法                                                 |
| @RepeatedTest         | 指定测试方法重复执行                                         |
| @ParameterizedTest    | 指定参数化测试方法，类似重复执行，从@ValueSource中获取参数   |
| @ValueSource          | 用于参数化测试指定参数                                       |
| @AutoConfigureMockMvc | 启用MockMvc的自动配置，可用于测试接口                        |

2、如下是JUnit5 自身的注解，与Spring及SpringBoot都无关

| 注解                   | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| @Test                  | 表示该方法是测试方法。与JUnit4的@Test注解不同的是，这个注解没有声明任何属性 |
| @ParameterizedTest     | 表示该方法是参数化测试方法                                   |
| @RepeatedTest          | 表示该方法是用于重复测试的测试模板                           |
| @TestFactory           | 表示该方法是用于动态测试的测试工厂                           |
| @TestTemplate          | 表示方法是测试用例的模板，设计为被调用多次，调用次数取决于注册提供者返回的调用上下文 |
| @TestMethodOrder       | 用于配置带注解测试类的测试方法执行顺序；类似于JUnit 4的@FixMethodOrder |
| @TestInstance          | 用于为带注解的测试类配置测试实例生命周期                     |
| @DisplayName           | 声明测试类或测试方法的自定义名称。                           |
| @DisplayNameGeneration | 声明测试类的自定义名称生成器                                 |
| @BeforeEach            | 表示被注解的方法应在当前类的每个 @Test ， @RepeatedTest ， @ParameterizedTest  或 @TestFactory 方法之前执行 ; 类似于 JUnit 4 的 @Before |
| @AfterEach             | 表示被注解的方法应在当前类的每个 @Test ， @RepeatedTest ， @ParameterizedTest  或 @TestFactory 方法之后执行 ; 类似于 JUnit 4 的 @After |
| @BeforeAll             | 表示被注解的方法应该在当前类的所有 @Test ， @RepeatedTest ， @ParameterizedTest 和  @TestFactory 方法之前执行 ; 类似于 JUnit 4 的 @BeforeClass 。必须是静态方法 |
| @AfterAll              | 表示被注解的方法应该在当前类的所有 @Test ， @RepeatedTest ， @ParameterizedTest 和  @TestFactory 方法之后执行 ; 类似于 JUnit 4 的 @AfterClass 。必须是静态方法 |
| @Nested                | 表示被注解的类是一个嵌套的非静态测试类                       |
| @Tag                   | 在类或方法级别声明标签，用于过滤测试 ; 类似于 TestNG 中的 test group 或 JUnit 4中的 Categories 。这个注释可以在类级别上继承，但不能在方法级别上继承 |
| @Disabled              | 用于禁用测试类或测试方法; 类似于JUnit4的@Ignore              |
| @Timeout               | 如果执行超过给定的时间，则使 test, test factory, test template, or lifecycle method 失败 |
| @ExtendWith            | 用于注册自定义扩展                                           |
| @RegisterExtension     | 用于通过字段以编程方式注册扩展名                             |
| @TempDir               | 用于通过 lifecycle method或test method中的字段注入或参数注入来提供临时目录 |

注意：方法被：@Test、@TestTemplate、@RepeatedTest、@BeforeAll、@AfterAll、@BeforeEach、@AfterEach 注解了都不能有返回值，都应该是 public void XXX(args...){} 这样的方法 



## 三、基本使用

### 1、集成SpringBoot

先说版本，是为了避免因为版本差异出现各种奇怪的问题：

- JDK：JDK8（小版本可以忽略）

- SpringBoot：2.5.0

- - 继承：spring-boot-starter-parent
  - 依赖：spring-boot-starter-web
  - 依赖：spring-boot-starter-test

- JUnit：5.7.2

- Mockito：3.9.0

- hamcrest：2.2

SpringBoot 的好处在于，只要继承spring-boot-starter-parent或引入spring-boot-pom-dependencies，然后添加spring-boot-starter-test依赖即可。定义的 POM 内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.2</version>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>springboot-junit5-mockito</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-junit5-mockio</name>

    <dependencies>
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



### 2、最简单的测试

我们先来一个最简单的单元测试，使用@SpringBootTest注解启用单元测试，使用@Test指定测试方法，使用Assertions类的方法来断言结果是否符合预期，具体代码如下。

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

/**
 * JUnit基本测试
 * Created by macro on 2022/10/11.
 */
@SpringBootTest
public class FirstTest {
    @Test
    public void test() {
        int a = 1;
        Assertions.assertEquals(1, a);
    }
}
```

然后点击测试方法左侧按钮即可进行测试。执行完成后我们在IDEA的执行窗口中就可以看到方法测试通过了，由于使用@SpringBootTest启用了SpringBoot环境，日志中会输出SpringBoot的Banner。具体测试结果可以在IDEA左下侧框查看。



### 3、指定测试方法顺序

我们可以通过**@TestMethodOrder**注解和**@Order**注解来指定所有测试方法的执行顺序，具体代码如下。

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.test.context.junit.jupiter.SpringExtension;

/**
 * JUnit指定方法测试顺序
 * Created by macro on 2022/10/10.
 */
@Slf4j
@ExtendWith(SpringExtension.class)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class MethodOrderTest {

    @Test
    @Order(1)
    @DisplayName("order为1的方法")
    void lowOrder(){
        log.info("lowOrder method");
    }

    @Test
    @Order(10)
    @DisplayName("order为10的方法")
    void highOrder(){
        log.info("highOrder method");
    }
}
```

```
09:32:46.688 [main] INFO com.example.MethodOrderTest - lowOrder method
09:33:31.602 [main] INFO com.example.MethodOrderTest - highOrder method
```

这里我们使用了@DisplayName注解给测试方法取了个别名



### 4、快速运行单元测试

1、@ExtendWith指定了运行环境为Spring而不是SpringBoot，所以日志中不会出现SpringBoot的Banner，执行速度也更快。

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.env.Environment;
import org.springframework.test.context.junit.jupiter.SpringExtension;

@Slf4j
@ExtendWith(SpringExtension.class)
class SpringExtensionTest {
    /**
     * @Autowired
     * private TestService testService;
     * 如果使用如上注入自定开发的类, 会报错.
     * SpringExtension.class, 表示只启用Spring自己的环境进行简单测试
     */
    @Value("${test:'sam'}")
    private String test;
    @Autowired
    Environment environment;
    @Test
    void serviceTest() {
        log.info(test);
        log.info(environment.getProperty("test"));
    }
}
```

```
21:00:10.493 [main] INFO com.example.SpringExtensionTest - 'sam'
21:00:10.493 [main] INFO com.example.SpringExtensionTest - null
```

2、@SpringJunitConfig可以只指定需要加载到Spring容器中的类，单个测试中用不到的类，不会加载到Spring容器中。

```java
import com.example.springdynamiccrudbean.repository.TestRepository;
import com.example.springdynamiccrudbean.service.TestService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

/**
 * @SpringJUnitConfig中包含@ExtendWith({SpringExtension.class})
 * 如果@SpringJUnitConfig种单独之注入TestService,执行后会报错,
 * 因为TestService中有方法调用TestRepository,而TestRepository没有被注入
 */
// @SpringJUnitConfig(locations = "classpath:文件名.xml")
@Slf4j
@SpringJUnitConfig(classes = {TestRepository.class, TestService.class})
class SpringJUnitConfigTest {
    @Autowired
    private TestService testService;
    @Test
    void serviceTest() {
        log.info(testService.get());
    }
}
```

```
21:03:51.201 [main] INFO com.example.SpringJUnitConfigTest - test
```

3、@ContextConfiguration + @ExtendWith(SpringExtension.class) = @SpringJUnitConfig。指定需要加载到Spring容器中的类

```java
package com.example.jpa;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import static org.assertj.core.api.Assertions.assertThat;

@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = ConfigurationTest.Config.class)
public class ConfigurationTest {
    @Test
    void explicitContextTest() {
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(Config.class);
        assertThat(applicationContext.getBean(Skeleton.class)).isNotNull();
    }

    @Test
    void contextConfigInjectionTest(@Autowired Skeleton skeleton) {
        assertThat(skeleton).isNotNull();
    }

    @Configuration
    static class Config {
        @Bean
        Skeleton skeleton() {
            return new SkeletonImpl();
        }
    }

    interface Skeleton {
    }

    static class SkeletonImpl implements Skeleton {
    }
}
```



### 5、生命周期测试

我们还可以通过JUnit 5的生命周期注解来执行测试方法，比如在@BeforeAll注解指定的方法中做全局初始化，在@AfterAll注解指定的方法中做资源的销毁，具体代码如下。

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.test.context.junit.jupiter.SpringExtension;

/**
 * JUnit生命周期测试
 * Created by macro on 2022/10/10.
 */
@Slf4j
@ExtendWith(SpringExtension.class)
public class LifecycleTest {

    @BeforeAll
    static void allInit(){
        log.info("allInit()：在所有方法前执行，只执行一次");
    }

    @BeforeEach
    void eachInit(){
        log.info("eachInit()：在测试方法前执行，每个测试方法前都执行");
    }

    @Test
    void successTest() {
        log.info("successTest()：方法执行成功");
    }

    @AfterEach
    void eachDown(){
        log.info("eachDown()：在测试方法后执行，每个测试方法后都执行");
    }

    @AfterAll
    static void allDown(){
        log.info("allDown()：在测试方法后执行，每个测试方法后都执行");
    }
}
```

```
09:29:30.547 [main] INFO com.example.LifecycleTest - allInit()：在所有方法前执行，只执行一次
09:29:31.764 [main] INFO com.example.LifecycleTest - eachInit()：在测试方法前执行，每个测试方法前都执行
09:29:31.771 [main] INFO com.example.LifecycleTest - successTest()：方法执行成功
09:29:31.784 [main] INFO com.example..LifecycleTest - eachDown()：在测试方法后执行，每个测试方法后都执行
09:29:31.822 [main] INFO com.example..LifecycleTest - allDown()：在测试方法后执行，每个测试方法后都执行
```



### 6、断言 Assertions

我们可以通过Assertions类中提供的断言API来断言测试结果。

例如我们可以使用fail方法直接断言方法执行失败并输出提示信息。还可以通过assertTrue、assertNull、assertEquals这类方法来断言结果是否符合预期。

```java
import jodd.util.ThreadUtil;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.time.Duration;

/**
 * JUnit断言测试
 * Created by macro on 2022/10/11.
 */
@Slf4j
@ExtendWith(SpringExtension.class)
public class AssertTest {

    /**
     * 直接断言方法执行失败并输出提示信息
     */
    @Test
    void failTest() {
        Assertions.fail("failTest()：方法执行失败");
    }

    /**
     * 断言结果是否为 true
     */
    @Test
    void trueTest() {
        Assertions.assertTrue(1 == 1);
    }

    /**
     * 断言结果是否为 false
     */
    @Test
    void trueFalse() {
        Assertions.assertFalse(3 <= 2);
    }

    /**
     * 断言结果是否为 null
     */
    @Test
    void nullTest() {
        Assertions.assertNull(null);
    }

    /**
     * 断言结果是否不为 null
     */
    @Test
    void notNullTest() {
        String str = "test";
        Assertions.assertNotNull(str);
    }

    /**
     * 断言两个值是否 Equals
     */
    @Test
    void equalsTest() {
        String str1 = "test";
        String str2 = "test";
        Assertions.assertEquals(str1, str2);
    }

    /**
     * 断言两个值是否 NotEquals
     */
    @Test
    void notEqualsTest() {
        String str1 = "test";
        String str2 = "test";
        Assertions.assertNotEquals(str1, str2);
    }

    /**
     * 使用assertThrows方法来断言方法中抛出的异常
     */
    @Test
    void throwsTest() {
        Assertions.assertThrows(NullPointerException.class, () -> {
            String str = null;
            log.info(str.toLowerCase());
        });
    }

    /**
     * 使用assertTimeout方法断言方法的执行时间
     */
    @Test
    void timeoutTest() {
        Assertions.assertTimeout(Duration.ofMillis(1000), () -> {
            long sleepTime = 2000;
            ThreadUtil.sleep(sleepTime); // ThreadUtil 来之jodd-core 依赖
            log.info("timeoutTest():休眠{}毫秒", sleepTime);
        });
    }

    /**
     * 使用assertAll方法将几个断言结合起来使用
     */
    @Test
    void assertAllTest() {
        Assertions.assertAll(() -> {
            trueTest();
        }, () -> {
            nullTest();
        }, () -> {
            equalsTest();
        });
    }
}
```

Assertions 类中提供的工具方法很多，具体可以参考它的源代码。



### 7、其他测试

SpringBoot Test除了上述测试功能，还可以使用@Disabled来禁用某个测试方法、使用@RepeatedTest来实现循环测试 等方法

```java
import jodd.util.ThreadUtil;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.Timeout;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import java.util.concurrent.TimeUnit;

/**
 * JUnit其他测试
 * Created by macro on 2022/10/10.
 */
@Slf4j
@ExtendWith(SpringExtension.class)
public class OtherTest {

    /**
     * 使用@Disabled来禁用某个测试方法
     */
    @Test
    @Disabled("用于测试@Disabled注解")
    void disabledTest() {
        log.info("disabledTest()：方法被执行");
    }

    private static int count = 0;

    /**
     * 使用@RepeatedTest来实现循环测试
     */
    @RepeatedTest(3)
    void repeatedTest() {
        count++;
        log.info("repeatedTest()：重复执行第{}次", count);
    }

    /**
     * 通过@ParameterizedTest来进行参数化测试
     */
    @ParameterizedTest
    @ValueSource(ints = {1, 2, 3})
    public void parameterizedTest(int a) {
        log.info("parameterizedTest()：a={}", a);
    }

    /**
     * 通过@Timeout来进行返回时间是否超时
     */
    @Test
    @Timeout(value = 1, unit = TimeUnit.MICROSECONDS)
    public void timeoutTest() {
        ThreadUtil.sleep(1001);
        log.info("@Timeout: 表示测试方法运行如果超过了指定时间将会返回错误");
    }
}
```



## 四、DB 测试

### 1、@DataJpaTest

@DataJpaTest 简介：为了测试 Spring Data JPA 存储库或任何其他与 JPA 相关的组件，Spring Boot 提供了 `@DataJpaTest` 注解。我们可以将它添加到单元测试中，它将设置一个 Spring 应用程序上下文。

```properties
#################### H2 ####################
spring.datasource.url = jdbc:h2:mem:test_jpa
spring.datasource.driver-class-name = org.h2.Driver
spring.datasource.username = sa
spring.datasource.password = sa

#################### H2 Web Console ####################
spring.h2.console.enabled = true
spring.h2.console.path = /h2
spring.h2.console.settings.web-allow-others = true

#################### JPA  ####################
spring.jpa.show-sql = true
spring.jpa.properties.hibernate.format_sql = true
spring.jpa.hibernate.ddl-auto = update
spring.jpa.database-platform = org.hibernate.dialect.H2Dialect
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Table
@Entity
public class User {
    @Id
    private Integer userId;
    private String username;
    private String password;
}
```

```java
public interface UserRepository extends JpaRepository<User, Integer>, JpaSpecificationExecutor<User> {
}
```

```java
import com.example.jpa.entity.User;
import com.example.jpa.repository.UserRepository;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.TestPropertySource;
import org.springframework.transaction.annotation.Transactional;
import javax.persistence.EntityManager;
import javax.sql.DataSource;

@TestPropertySource(properties = {
        "spring.jpa.hibernate.ddl-auto=create",
        "spring.jpa.properties.hibernate.format_sql=true"
})
@DataJpaTest
class JpaApplicationTests {
    @Autowired DataSource dataSource;
    @Autowired JdbcTemplate jdbcTemplate;
    @Autowired EntityManager entityManager;
    @Autowired UserRepository userRepository;

    @Test
    void injectedComponentsAreNotNull() {
        Assertions.assertNotNull(dataSource);
        Assertions.assertNotNull(jdbcTemplate);
        Assertions.assertNotNull(entityManager);
        Assertions.assertNotNull(userRepository);
    }

    @Test
    @Rollback(false)
    @Transactional
    void saveTest() {
        userRepository.saveAndFlush(new User(1, "Sam", "123456"));
    }
}
```

@ExtendWith：本教程中的代码示例使用 @ExtendWith 注解告诉 JUnit 5 启用 Spring 支持。从 Spring Boot 2.1 开始，我们不再需要加载 SpringExtension，因为它作为元注解包含在 Spring Boot 测试注解中，例如 @DataJpaTest、@WebMvcTest 和 @SpringBootTest。本教程中的代码示例使用 @ExtendWith 注解告诉 JUnit 5 启用 Spring 支持。从 Spring Boot 2.1 开始，我们不再需要加载 SpringExtension，因为它作为元注解包含在 Spring Boot 测试注解中，例如 @DataJpaTest、@WebMvcTest 和 @SpringBootTest。

这样创建的应用程序上下文将不包含我们的 Spring Boot 应用程序所需的整个上下文，而只是它的一个“切片”，其中包含初始化任何 JPA 相关组件（如我们的 Spring Data 存储库）所需的组件。

例如，如果需要，我们可以将 DataSource、JdbcTemplate 或 EntityManage 注入我们的测试类。此外，我们可以从我们的应用程序中注入任何 Spring Data 存储库。上述所有组件将自动配置为指向嵌入式内存数据库，而不是我们可能在 application.properties 或 application.yml 文件中配置的“真实”数据库。请注意，默认情况下，包含所有这些组件（包括内存数据库）的应用程序上下文在所有 `@DataJpaTest` 注解的测试类中的所有测试方法之间共享。

这就是为什么在默认情况下**每个测试方法都在自己的事务中运行的原因，该事务在方法执行后回滚**。这样，数据库状态在测试之间保持原始状态，并且测试保持相互独立。



### 2、@Sql

解决测试时测试数据写入与销毁不方便，使用@Sql自动导入SQL脚本语句，结合@Transactional事务回滚实现测试后数据销毁，便捷测试。

@Sql注解可以执行SQL脚本，也可以执行SQL语句。它既可以加上类上面，也可以加在方法上面。 默认情况下，方法上的@Sql注解会覆盖类上的@Sql注解，但可以通过@SqlMergeMode注解来修改此默认行为。

@Sql有下面的属性：

-  config:配置@SqlConfig，它的作用域是其@Sql注释的本地范围。它用于配置commentPrefix、分隔符等。

-  executionPhase:决定何时执行SQL脚本和语句。默认是BEFORE_TEST_METHOD。

-  statements:配置执行内联SQL语句。

-  scripts:配置SQL脚本执行的路径。

-  value:脚本元素的别名。脚本和值不能一起使用，但可以与statement元素一起使用。

```sql
INSERT INTO USER (id,  NAME,  email) VALUES (1, 'Zaphod Beeblebrox', 'zaphod@galaxy.net');
```

```java
// @ExtendWith(SpringExtension.class)
@DataJpaTest
class SqlTest {
    @Autowired
    private UserRepository userRepository;
    @Test
    @Sql("createUser.sql")
    void whenInitializedByDbUnit_thenFindsByName() {
        UserEntity user = userRepository.findByName("Zaphod Beeblebrox");
        assertThat(user).isNotNull();
    }
}
```

如果我们需要多个脚本，我们可以使用 `@SqlGroup` 来组合它们。



### 3、@SqlConfig

用于配置如何去解释@Sql注解中指定的Sql脚本。 可以用于类上，也可以用于方法上。 @Sql注解也有一个config属性，作用与@SqlConfig相同，不同的是作用域只在对应的@Sql注解范围。它的优先级也大于类注解的

- blockCommentStartDelimiter：多行注释开始字符，默认是/*。

-  blockCommentEndDelimiter：多行注释结束字符，默认是*/。

-  commentPrefix：单行注释前缀，默认是–。

-  commentPrefixes：指定多个单行注释前缀，默认是["–"]。

-  dataSource：指定脚本执行的数据库的名字，只有在多个数据源时需要指定。

-  encoding：指定sql脚本文件的字符编码。

-  errorMode：配置错误模式，默认是SqlConfig.ErrorMode的DEFAULT。

-  separator：配置脚本语句分隔符，默认是’\n’。

-  transactionManager：指定transactionManager bean，只有有多个transactionManager时需要指定。

-  transactionMode：指定脚本执行的事务模式，默认是SqlConfig.TransactionMode的DEFAULT。



### 4、@SqlMergeMode

@SqlMergeMode可以加在类上，也可以加在方法上。用于指示方法上的@Sql和类上@Sql注解配置是否合并。方法上的@SqlMergeMode注解优先级更高。默认值是SqlMergeMode.MergeMode的OVERRIDE。



### 5、参考文献 & 鸣谢

1. 使用 Spring Boot 和 @DataJpaTest 测试 JPA 查询：https://blog.csdn.net/dancuowang/article/details/121034672
2. 测试之@Sql（自动插入sql脚本，测试后销毁）https://blog.csdn.net/system_cls/article/details/124270066



## 五、项目实战

上面介绍了Spring Boot Test的基本使用，下面我们结合项目来使用下它。

```java
@RestController
@RequiredArgsConstructor
public class TestController {
    final TestService testService;

    @GetMapping("/get")
    public String get() {
        return testService.get();
    }
}

@Service
@RequiredArgsConstructor
public class TestService {
    final TestRepository testRepository;

    public String get() {
        return testRepository.get();
    }
}

@Component
public class TestRepository {
    public String get() {
        return "test";
    }
}
```



### 1、非 Controller 层测试

1、SpringBoot JUnit5 单元测试

```java
import com.example.springdynamiccrudbean.repository.TestRepository;
import com.example.springdynamiccrudbean.service.TestService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@Slf4j
@SpringBootTest
public class ServiceDaoTest {
    @Autowired
    private TestRepository testRepository;
    @Autowired
    private TestService testService;

    @Test
    void daoTest(){
        Assertions.assertEquals("test", testRepository.get());
    }

    @Test
    void serviceTest(){
        Assertions.assertEquals("test", testService.get());
    }
}
```

2、使用Mockito来Mock一下testService和testRepository对象

```java
import com.example.springdynamiccrudbean.repository.TestRepository;
import com.example.springdynamiccrudbean.service.TestService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.springframework.boot.test.context.SpringBootTest;

@Slf4j
@SpringBootTest
public class ServiceDaoTest {
    /**
     * 使用@Mock注解对测试接口进行修饰。该注解位于org.mockito.Mock中
     *  使用@mock注解会创建 Mock 对象 testRepository
     *  等同于之前代码：
     *  TestRepository testRepository = Mockito.mock(TestRepository.class);
     *  注意：使用 @MockBean 或 @Mock 都可以
     */
    @Mock
    private TestRepository testRepository;

    @Test
    void daoTest(){
        // MOCK设置：当调用get的时候，返回test mock
        Mockito.when(testRepository.get()).thenReturn("test mock");
        Assertions.assertEquals("test mock", testRepository.get());
    }

    @Test
    void serviceTest(){
        // 除了注解还可以使用Mockito.mock()方法获取Mock对象
        TestService testService = Mockito.mock(TestService.class);
        // MOCK设置：当调用get的时候，返回test mock
        Mockito.when(testService.get()).thenReturn("test mock");
        Assertions.assertEquals("test mock", testService.get());
    }
}
```



### 2、Controller 层测试

对于Controller层方法进行测试，有时我们需要模拟请求，使用MockMvc即可。

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultHandlers;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

@SpringBootTest
@AutoConfigureMockMvc
public class ControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void mvcTest() throws Exception{
        // 模拟发送一个请求访问分页查询品牌列表的接口
        mockMvc.perform(MockMvcRequestBuilders.get("/get") // 设置请求地址
                .param("pageNum","1") // 设置请求参数
                .param("pageSize","5"))
                .andExpect(MockMvcResultMatchers.status().isOk()) // 断言返回状态码为200
                .andDo(MockMvcResultHandlers.print()) // 在控制台打印日志
                .andReturn(); // 返回请求结果
    }
}
```





## 五、参考文献 & 鸣谢

作者：西魏陶渊明，来源如下：

1. 《SpringBoot单元测试》第一篇认知篇：https://mp.weixin.qq.com/s/be3Ol4iT4Pl8K7Ob5f2V3A
2. 《SpringBoot单元测试》第二篇框架篇：https://mp.weixin.qq.com/s/g8VpeRn7Qe1G6I7jGF7fFw
3. 《SpringBoot单元测试》第三篇源码篇：https://mp.weixin.qq.com/s/2uWe7ZLSfQt9FyK9BNsDsA



1. SpringBoot 单元测试：https://blog.csdn.net/dancuowang/article/details/120969535
2. SpringBoot 与 @WebMvcTest 测试 Controller：https://blog.csdn.net/dancuowang/article/details/120999938
3. SpringBoot 与  @DataJpaTest 测试 JPA 查询：https://blog.csdn.net/dancuowang/article/details/121034672
4. SpringBoot 与  @SpringBootTest 测试：https://blog.csdn.net/dancuowang/article/details/121115049

