# JUnit5 入门

> 参考：https://blog.csdn.net/qq_41573234/article/details/110456894

## 1、JUnit5 组成

JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

1. JUnit Platform：为在JVM上启动测试框架提供基础。它还定义了TestEngine API, 用来开发在平台上运行的测试框架。此外，平台提供了一个控制台启动器，用于从命令行启动平台，并为Gradle和Maven提供构建插件以及基于 JUnit 4的Runner，用于在平台上运行任意 TestEngine 。 

2. JUnit Jupiter：是在JUnit 5中编写测试和扩展的新型编程模型和扩展模型的组合.Jupiter子项目提供 了 TestEngine ，用于在平台上运行基于Jupiter的测试。

3. JUnit Vintage：提供 TestEngine ，用于在平台上运行基于JUnit 3和JUnit 4的测试。

```
                                        测试用例
                                           ⬇
                          ⬇                                  ⬇
                    JUnit 5 API                      JUnit 4.x,3.x API
                          ⬇                                  ⬇
                    JUnit Jupiter                      JUnit Vintage
                                           ⬇
                                     JUnit TestEngine
                                           ⬇
                                     JUnit Platform
                                           ⬇
                           InterllJ Idea, Eclipse, Maven, Gradle...
```

![img](https://img-blog.csdnimg.cn/img_convert/94a3af94d8471b9b5aefac6af1b26b5a.png)



## 2、JUnit5 注解

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

**PS**：方法被：@Test、@TestTemplate、@RepeatedTest、@BeforeAll、@AfterAll、@BeforeEach、@AfterEach 注解了都不能有返回值，都应该是 public void XXX(args...){} 这样的方法 



## 3、导入依赖

```xml
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>1.7.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
```

```java
@Test
public void testCase(){
    Assertions.assertEquals(2,1+1);
}
```





# SpringBoot 2.x + JUnit 5

> 参考：https://www.cnblogs.com/xiaomaomao/p/14307749.html

## 1、JUnit5 简介

Spring Boot 2.2.0 版本开始引入 JUnit5 作为单元测试默认库, JUnit5作为最新版本的 JUnit框架, 它与之前版本的 JUnit框架有很大的不同，由三个不同子项目的几个不同模块组成.

JUnit5 = JUnitPlatform + JUnitJupiter + JUnitVintage

1. JUnitPlatform：是在 JVM 上启动测试框架的基础,不仅支持 JUnit自制的测试引擎,其它测试引擎也都可以接入
2. JUnitJupiter：提供了JUnit5 的新的编程模型，是 JUnit5 新特性的核心.内部包含了一个测试引擎,用于在 JUnitPlatform 上运行
3. JUnitVintage：由于 JUnit已经发展多年,为了照顾老的项目,JUnitVintage 提供了兼容 JUnit 4.x,JUnit 3.x 的测试引擎

注意：SpringBoot 2.4 以上版本移除了默认对 Vintage 的依赖。如果需要兼容 JUnit4.x 版本,需要自行引入（不能使用junit4的功能 @Test）

![img](https://img2020.cnblogs.com/blog/1636535/202101/1636535-20210121113558850-584369660.jpg)

打开项目发布信息 ----> 选择 v 2.4 这个版本，该版本的变更中说明了已经移除了 JUnit5 了

![img](https://img2020.cnblogs.com/blog/1636535/202101/1636535-20210121113745451-904951035.jpg)





## 2、整合 JUnit5 

未整合 JUnit 之前：

```java
@SpringBootTest + @RunWith(SpringTest.class)
```

SpringBoot 整合 Junit 以后：

编写测试方法：@Test标注（注意需要使用 junit5 版本的注解）Junit类具有 Spring 的功能, 例如 @Autowired、@Transactional (标注测试方法，测试完成后自动回)

我们只需要引入 spring-boot-starter-test 依赖即可完成整合，其它的 SpringBoot 底层都已经帮我们自动配置好了

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```



## 3、Junit 5 常用注解

具体的使用规范可以参考官网 JUnit 5 官网：https://junit.org/junit5/

JUnit5与JUnit4 注解有所变化，具体可以参考官方文档：https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations

| **注解**      | **说明**                                                     |
| ------------- | ------------------------------------------------------------ |
| @Test         | 表示方法是测试方法,但是与 JUnit4 的 @Test 不同,它的职责非常单一不能声明任何属性,拓展的测试将会由 Jupiter 提供额外测试 |
| @DisplayName  | 为测试类或者测试方法设置展示名称                             |
| @BeforeAll    | 表示在所有单元测试之前执行                                   |
| @BeforeEach   | 表示在每个单元测试之前执行                                   |
| @Timeout      | 表示测试方法运行如果超过了指定时间将会返回错误               |
| @Disabled     | 表示测试类或测试方法不执行                                   |
| @RepeatedTest | 表示方法可重复执行                                           |
| @ExtendWith   | 为测试类或测试方法提供扩展类引用                             |
| @AfterEach    | 表示在每个单元测试之后执行                                   |
| @AfterAll     | 表示在所有单元测试之后执行                                   |



## 4、Junit 5 测试

```java
@DisplayName("测试类")
@SpringBootTest
class DatasourceApplicationTests {

    @Test
    @BeforeAll
    public static void beforeAll() {
        System.out.println("@BeforeAll: 表示在所有单元测试之前执行");
    }

    @BeforeEach
    public void beforeEach() {
        System.out.println("@BeforeEach: 表示在每个单元测试之前执行");
    }

    @Test
    @DisplayName("测试方法一")
    public void test01() {
        System.out.println("@DisplayName: 为测试类或者测试方法设置展示名称");
    }

    @Test
    @DisplayName("测试方法二")
    @Timeout(value=1,unit=TimeUnit.MICROSECONDS)
    public void test02() {
        System.out.println("@Timeout: 表示测试方法运行如果超过了指定时间将会返回错误");
    }
}
```

![img](https://img2020.cnblogs.com/blog/1636535/202101/1636535-20210121141617444-1304033619.jpg)

![img](https://img2020.cnblogs.com/blog/1636535/202101/1636535-20210121141626218-270653487.jpg)

当然 JUnit 5还有很多其它的使用，例如：断言、前置条件、嵌套测试、参数化测试 以及如何从 Junit 4 迁移到 JUnit 5 等等。在实际开发过程中，建议去参照官方文档的用户指南并且结合自身项目的实际情况选择合适的 JUnit 5 功能