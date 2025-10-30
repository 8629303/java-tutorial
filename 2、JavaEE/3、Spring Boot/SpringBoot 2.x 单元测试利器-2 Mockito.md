# SpringBoot 整合 Mock 进行单元测试

> 作者：[YD_1989](https://blog.csdn.net/m0_58680865)、来源：超详细 springboot 整合 Mock 进行单元测试！本文带你搞清楚！https://blog.csdn.net/m0_58680865/article/details/138711160

## 一、Mock 是什么

### 1、Mock 的定义

Mockito 是 Java 单元测试开发框架。在写测试单元时它可以 Mock（Mock 的中文释义是模拟，所以 Mockito 从名字上可以看出是要模拟一种场景）。

它可以模拟任何 Spring 管理的 Bean、模拟方法的返回值、模拟抛出异常等，避免为了测试一个方法，却要自行构建整个 bean 的依赖链。

Mock 测试主要是用来进行开发中一些 未完成的接口 或者 网络断开、数据库连接错误 等方法调用。

举个例子：如下代码所示，list 集合需要从数据库查询出来。但是如果暂时数据库不能用，有需要测试，这个时候就可以进行 Mock 模拟出符合条件的 list 集合进行本地测试，无需连接数据库。

```java
@Override
public PageInfo<Department> getAll(String name, int pageNum, int pageSize) {
    Pagehelper.startPage(pageNum, pageSize);
    List<Department> list = departmentDao.getAll(name); // mock 此方法查询数据库
    Pageinfo<Department> pageinfo = new PageInfo<>(list);
    return. pageInfo;
}
```



### 2、为什么使用

在对代码进行单元测试过程中，经常会有以下的情况发生：

```java
class A 依赖 class B  
class B 依赖 class C和class D  
class C 依赖 ...  
class D 依赖 ...  

1.被测对象依赖的对象构造复杂
- 我们想对class A进行单元测试，需要构造大量的class B、C、D等依赖造步骤多、耗时较长的对象，
- 对于他们的构造我们可以利用mock去构造过程复杂的对象用于class A的测试，
- 因为我们只是想测试class A的行为是否符合预期，我们并不需要测试依赖对象。

2.被测单元依赖的模块尚未开发完成，而被测对象需要依赖模块的返回值进行测试：
- 比如service层的代码中，包含对dao层的调用，但dao层代码尚未实现
- 比如web的前端依赖后端接口获取数据进行联调测试，但后端接口并未开发完成
- 比如数据库还不能正常使用但是需要测试功能逻辑是否可行。
```



### 3、常用的 Mock 技术

- PowerMock
- EasyMock
- Mockito
- JMock

目前在 Java 中主流的 Mock 测试工具有 Mockito、JMock、EasyMock 等等，而 SpringBoot 目前内建的是 Mockito 框架。



### 4、Mokito 中文文档

Mokito 中文官网：https://github.com/hehonghui/mockito-doc-zh



### 5、集成测试和单元测试区别

1、集成测试：测试过程中，会启动整个Spring容器，调用DB 或者 依赖的外部接口等。只不过访问的环境是测试环境。这个过程最大程度还原生产环境过程，但是耗时长。

2、单元测试：不启动整个应用，只对单个接口/类进行测试。不调用DB 、外部接口，依赖的服务都Mock掉，只测试代码逻辑。这个过程，测试用例耗时短。



## 二、Mock API

### 1、Mockito API

- mock：构建一个我们需要的对象；可以mock具体的对象，也可以mock接口
- spy：构建监控对象
- verify：验证某种行为
- when：当执行什么操作的时候，一般配合thenXXX 一起使用。表示执行了一个操作之后产生什么效果
- doReturn：返回什么结果
- doThrow：抛出一个指定异常
- doAnswer：做一个什么相应，需要我们自定义Answer
- times：某个操作执行了多少次
- atLeastOnce：某个操作至少执行一次
- atLeast：某个操作至少执行指定次数
- atMost：某个操作至多执行指定次数
- atMostOnce：某个操作至多执行一次
- doNothing：不做任何处理
- doReturn：返回一个结果
- doThrow：抛出一个指定异常
- doAnswer：指定一个操作，传入Answer
- doCallRealMethod：返回真实业务执行的结果，只能用于监控对象



### 2、ArgumentMatchers 参数匹配

- anyInt：任何int类型的参数，类似的还有anyLong/anyByte等等。
- eq：等于某个值的时候，如果是对象类型的，则看toString方法
- isA：匹配某种类型
- matches：使用正则表达式进行匹配



### 3、OngoingStubbing 返回操作

- thenReturn：指定一个返回的值
- thenThrow：抛出一个指定异常
- then：指定一个操作，需要传入自定义Answer
- thenCallRealMethod：返回真实业务执行的结果，只能用于监控对象



## 三、Mockito 的使用

### 1、添加 Maven 依赖

- Java 环境依赖

  ```xml
   <!-- https://mvnrepository.com/search?q=Mockito -->
  <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-core</artifactId>
      <version>4.4.0</version>
  </dependency>
  ```

- SpringBoot 环境依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
  </dependency>
  ```

**注意**：SpringBoot 默认的 Mock 框架是 Mockito，和 junit 一样，只需要依赖 spring-boot-starter-test 就可以了。



### 2、@Mock、@InjectMocks、@Spy 的初始化 

**使用 Mockito 的注解，需要让注解生效，让注解生效的方法有两个**：

1. 给被测类添加 `@ExtendWith(MockitoExtension.class)` 或者 `@RunWith(MockitoJUnitRunner.class)` 注解

   ```java
   // @RunWith(MockitoJUnitRunner.class) JUnit 4 的产物, 不能与其他 Runner 同时使用, 了解即可.
   @ExtendWith(MockitoExtension.class) // JUnit 5
   public class MockitoAnnotationTest {
   }
   ```

2. 在初始化方法中使用`MockitoAnnotations.openMocks(this)`

   ```java
   @Before
   public void init() {
       MockitoAnnotations.openMocks(this);
   }
   ```

**注意：新版spring-boot-starter-test不再集成junit，而是junit-jupiter，找不到`@RunWith`注解**：

- **spring-boot-starter-test 2.5.5 版本只需要在类上加上@SpringBootTest即可，不需要再加@RunWith()注解了**。
- **spring-boot-starter-test 2.4.x 版本的也没有@RunWith()注解，至于从哪个版本开始没有@RunWith()注解的，请自行查阅相关文档**。
- **一些较低版本也没有 openMocks 方法，而是 initMocks**。

***

所以我们在使用 @SpringBootTest 时，通常不需要再使用 @RunWith(MockitoJUnitRunner.class)。这是因为 @SpringBootTest 已经提供了一个完整的 Spring 应用上下文，可以管理所有的 bean，包括通过 Mockito 创建的 mock 对象。

**详细解释**：

1. @SpringBootTest：
   - @SpringBootTest 会启动一个完整的 Spring 应用上下文。这意味着你的测试将运行在一个模拟的 Spring 环境中，类似于在生产环境中运行的应用程序。
   - 它通常用于集成测试（integration tests），其中涉及多个 bean 的交互，并且需要上下文中的完整配置。
2. @RunWith(MockitoJUnitRunner.class)：
   - @RunWith(MockitoJUnitRunner.class) 是 Mockito 的一个 JUnit 运行器，用于在测试中初始化 @Mock、@Spy、@InjectMocks 等注解。它主要用于单元测试（unit tests），在不启动 Spring 应用上下文的情况下模拟对象的行为。

**如何选择**：

- 如果你使用 @SpringBootTest，你不需要再使用 @RunWith(MockitoJUnitRunner.class)，因为 Spring Boot 自己可以管理所有的 bean，包括使用 @MockBean 注解创建的 mock 对象。如果你确实需要 mock 某些 bean，可以使用 @MockBean 注解，而不是 @Mock 注解。例如：

  ```java
  @SpringBootTest
  public class MyServiceTest {
  
      @MockBean
      private DependencyService dependencyService;
  
      @Autowired
      private MyService myService;
  
      @Test
      public void testMyService() {
          // 设置 mock 行为
          when(dependencyService.someMethod()).thenReturn("mocked response");
  
          // 调用被测试的方法
          String result = myService.callDependency();
  
          // 验证结果
          assertEquals("mocked response", result);
      }
  }
  ```

- 如果你的测试只涉及单个类或少数几个类，并且你希望避免启动整个 Spring 应用上下文（从而加快测试速度），可以使用 @RunWith(MockitoJUnitRunner.class)。

**典型的使用场景**：

- **单元测试**：使用 @RunWith(MockitoJUnitRunner.class) 或者 JUnit 5 的扩展机制（如 @ExtendWith(MockitoExtension.class)）来模拟依赖关系。
- **集成测试**：使用 @SpringBootTest 来测试组件之间的实际交互，包括使用真实的 Spring 应用上下文。



### 3、@Mock、@InjectMocks、@Spy  的使用

- @Mock：用于代替 Mockito.mock 创建 mock 对象，创建一个 Mock 实例，需要基于 JUnit5 环境。
- @InjectMocks：创建一个实例，其余用 @Mock 或 @Spy 注解创建的 mock 将**被注入到用该实例中**。
- @Spy：@Spy 会保留被测对象的真实行为。你可以选择性地 mock 一些方法，而不影响对象的整体行为。这对于测试需要与复杂对象交互的场景非常有用。

**直白的理解就是：**

实体类 TUserServiceImpl 通过注解` @Autowired`注入了三个实体：TUserMapper、JdbcTemplate 和 NamedParameterJdbcTemplate。

```java
@Service
public class TUserSeriveImpl implments TUserService {
    @Autowired
    private TUserMapper tUserMapper;
    @Autowired(required = false)
    private JdbcTemplate jdbcTemplate;
    @Autowired(required = false)
    private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
}
```

如果想要测试 TUserServiceImpl，那么test中，TUserMapper、JdbcTemplate 和 NamedParameterJdbcTemplate 就要用@Mock注解创建Mock实例，而要被测试TUserServiceImpl就要用@InjectMocks注解创建实例，这个时候被@Mock注解创建的TUserMapper、JdbcTemplate 和 NamedParameterJdbcTemplate 就被注入到通过 @InjectMocks 注解创建的 TUserServiceImpl 实例中。如下所示:

```java
class MockitoTest {
    @Mock
    TUser tUser;
    @Mock // jdbcTemplate 中所有的方法都是模拟的
    JdbcTemplate jdbcTemplate;
    @Spy // namedParameterJdbcTemplate 部分方法模拟，部分方法真实调用
    NamedParameterJdbcTemplate namedParameterJdbcTemplate;
    @InjectMocks // 依赖注入 TUserServiceImpl 对象，相当于 new TUserServiceImpl()
    TUserServiceImpl tUserServiceImpl;
}
```

你要测试哪个类（如TUserServiceImpl），那么就用 @InjectMocks注解；被测试的类中通过 @Autowired注解注入了几个，那么测试类里面就用@Mock注解创建几个实例！

**模拟对象有两种方式**：

> 前提我们已经通过注解或者静态方法已经初始化 Mock 模拟器，使得注解已经生效的情况下：

1. 对单个对象使用注解 @Mock 自动进行 mock。
2. 对单个对象手动 mock ：`Xxx xxx = Mockito.mock(Xxx.class)`;

@Mock

```java
@Mock private User user;  // 方式一
User user = Mockito.mock(User.class); // 方式二, 此方式一般在方法中使用
```

@Spy

```java
@Spy private user user; // 方式一
User user = Mockito.spy(User.class); // 方式二, 此方式一般在方法中使用
```

@captor 主要用来捕捉程序运行时调用 mock 或者 spy 的对象的方法时，传入的参数。它支持泛型，即要捕获的参数的类型。

```java
@Captor private ArgumentCaptor<User> userCaptor; // 方式一
ArgumentCaptor<User> arg = ArgumentCaptor.forClass(User.class); // 方式二, 此方式一般在方法中使用
```

@captor一般是与 given 或者 verify 等方法配合使用。

```java
@Mock
List<String> mockedList;

@Captor
ArgumentCaptor<String> argCaptor;

@Test
public void whenUseCaptorAnnotation_thenTheSam() {
    mockedList.add("one");
    Mockito.verify(mockedList).add(argCaptor.capture());
    assertEquals("one", argCaptor.getValue());
}
```



### 4、@MockBean 与 @Mack 注解的区别

@MockBean 和 @Mock 都是用于创建 mock 对象的注解，但它们的用途和适用场景有所不同。

**@Mock**

- **用途**: @Mock 是 Mockito 提供的注解，用于在测试类中创建一个 mock 对象。它通常用于单元测试（unit tests），尤其是当你不需要 Spring 应用上下文时。

- **作用范围**: @Mock 注解的对象仅在当前测试类中可用，通常与 @RunWith(MockitoJUnitRunner.class) 或 @ExtendWith(MockitoExtension.class) 一起使用来初始化这些 mock 对象。

  ```java
  // @RunWith(MockitoJUnitRunner.class)
  @ExtendWith(MockitoExtension.class)
  public class MyServiceTest {
  
      @Mock
      private DependencyService dependencyService;
  
      @InjectMocks
      private MyService myService;
  
      @Test
      public void testMyService() {
          when(dependencyService.someMethod()).thenReturn("mocked response");
  
          String result = myService.callDependency();
  
          assertEquals("mocked response", result);
      }
  }
  ```

**@MockBean**

- **用途**: @MockBean 是 Spring Boot 提供的注解，用于在 Spring 应用上下文中创建一个 mock 对象。它通常用于集成测试（integration tests）或在需要使用 Spring 上下文时替换某个 Bean 的实现。

- **作用范围**: 使用 @MockBean 标记的对象会被注入到 Spring 应用上下文中，并替换原有的 Bean。这样，你可以在测试 Spring Boot 应用时，对某个 Bean 进行 mock，而其他 Bean 可以正常工作。

  ```java
  @SpringBootTest
  public class MyServiceIntegrationTest {
  
      @MockBean
      private DependencyService dependencyService;
  
      @Autowired
      private MyService myService;
  
      @Test
      public void testMyService() {
          when(dependencyService.someMethod()).thenReturn("mocked response");
  
          String result = myService.callDependency();
  
          assertEquals("mocked response", result);
      }
  }
  ```

**区别总结**：

1. **应用上下文**：
   - @Mock 只是在测试类中创建 mock 对象，不与 Spring 上下文交互。
   - @MockBean 则将 mock 对象注入到 Spring 上下文中，替换原来的 Bean。
2. **使用场景**：
   - @Mock 更适用于单元测试，测试范围更小，不涉及 Spring 上下文。
   - @MockBean 更适用于集成测试，涉及多个组件的交互，并且你希望替换某些依赖的实现。
3. **依赖注入**：
   - @Mock 通常与 @InjectMocks 一起使用手动注入 mock 对象。
   - @MockBean 通过 Spring 的依赖注入机制自动注入到被测试的组件中。

**结论**：

- 使用 @Mock 进行单元测试，尤其是在你不需要整个 Spring 应用上下文的情况下。
- 使用 @MockBean 进行集成测试，或者当你需要在 Spring 应用上下文中替换某个 Bean 的实现时。



### 5、SpringBootTest 注解和 RunWith 注解在测试类的作用

1、**@SpringBootTest**：这个注解相当于启动类的作用，加了这个注解后，当使用加了 @Test 注解的方法时，会加载 Spring 上下文，跟 SpringBootApplication 这个启动类一样，把 Bean 加载进 IOC 容器。

其中参数 classes 需指明启动类.class，如果不指明，需要保证启动类所在的包和加了 SpringBootTest 注解的类 在同一个包或者是启动类的子包下，否则注入到（ @Autowired / @Resource）会报空指针异常。如下：

```java
@SpringBootTest(classes = MySpringbootApplication.class)
```

***

2、**@RunWith**：@RunWith（SpringRunner.class），作用是与 Spring 环境整合，因为在测试类中我们可以需要用 @Autowired 自动装配 IOC 容器中的 Bean，所以需要与 Spring 环境进行整合，才能实现自动装配，否则会装配失败，导致 Bean 为 null。

有时候会发现，有的测试类不添加 @RunWith 也能注入成功，这是因为，**如果导入 @Test 注解的包是org.junit.jupiter.api.Test，则不需要添加 @RunWith 注解，如果导入的是 org.junit.Test，则需要添加**，这点需要注意。



## 四、Mock 测试代码案例

### 1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

**注意：本案例用的 SpringBoot 版本是 2.6.4 版本**



### 2、编写业务代码

```java
@Service
public class PositionService {
    public String getStr(String str){
        return str;
    }
    public void getVoid(){
        System.out.println("没有返回值");
    }   
}
```



### 3、Mock 测试

#### 1）常规测试

先不使用 Mockito ，而是真的去调用一个正常的 Spring Bean ，测试类写法如下。其实就是很普通的注入 PositionService Bean，然后去调用他的方法。

测试代码：

```java
import com.cyd.applicationstartup.MySpringbootApplication;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(classes = MySpringbootApplication.class)
public class PositionServiceTest {

    @Autowired
    private PositionService positionService;

    @BeforeEach
    public void init() {

        // 对所有注解了@Mock的对象进行模拟
        MockitoAnnotations.openMocks(this);
        System.out.println("初始化方法");
    }

    @Test
    public void testGetStr() {

        String str = positionService.getStr("刘亦菲");
        System.out.println("测试结果：" + str);

    }
}
```

测试结果：

```java
初始化方法
测试结果：刘亦菲
```

> **特别需要注意**：
>
> - 可以看到测试类中引用的包是 org.junit.jupiter.api.Test 和 org.junit.jupiter.api.BeforeEach，是 jupiter.api 包下面的，此时测试类只用了 @SpringBootTest 这一个注解；
> - 但是，如果用的是 org.junit.Test 和 org.junit.Before，测试类上面必须同时用 @RunWith(SpringRunner.class) 和 @SpringBootTest(classes = MySpringbootApplication.class)。必须同时用！！！！！



#### 2）Mock 测试

Mock 测试需要自定返回结果，结果和方法返回结果类型一致。

**语法如下：**

```java
Mockito.when( 对象.方法名() ).thenReturn( 自定义结果 );
```

使用 Mockito 模拟 Bean 的单元测试代码示例如下：

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(classes = MySpringbootApplication.class)
public class PositionServiceTest {

    @Mock
    private PositionService positionService;

    @BeforeEach
    public void init() {
        System.out.println("初始化方法");
    }

    @Test
    public void testMockGetStr() {
        // 定义当调用mock positionService 的 getStr() 方法，并且任意字符串参数时，就返回字符串 "刘亦菲"
        Mockito.when(positionService.getStr(Mockito.anyString())).thenReturn("刘亦菲");

        // 定义当调用 mock positionService 的 getStr() 方法，并且参数是字符串 "美女" 时，就返回字符串 "刘亦菲"
        Mockito.when(positionService.getStr("美女")).thenReturn("刘亦菲");
        System.out.println(positionService.getStr("美女"));

        // 验证 positionService 的 getStr()这个方法是否被调用过
        Mockito.verify(positionService).getStr("刘亦菲");
    }
}
```

**注意**：代码中第一个 Mockito.when 的参数用的是`Mockito.anyString()`，表示 任意字符串参数调用 getStr() 方法，就会返回字符串 “刘亦菲”；第二个 Mockito.when 的参数用的是字符串"美女"，表示限制只有当参数是 "美女"时，才会返回 “刘亦菲”。因此，在日常 Mock 测试中，通常使用 Mockito.any 作为参数。



#### 3）Mock 测试常用方法

##### thenReturn 系列方法

1、定义当调用mock positionService 的 getStr() 方法，并且任意字符串参数时，就返回字符串 “哈哈哈哈”：

```java
Mockito.when(positionService.getStr(Mockito.anyString())).thenReturn("哈哈哈哈");
System.out.println(positionService.getStr("任意参数"));
```

2、② 定义当调用 mock positionService 的 getStr() 方法，并且限制参数只有是字符串 “美女” 时，才返回字符串 “刘亦菲”：

```
Mockito.when(positionService.getStr("美女")).thenReturn("刘亦菲");
System.out.println(positionService.getStr("美女"));
```



##### thenThrow 系列方法

1、当调用 mock positionService 的 getStr() 方法，输入的的参数是 字符串 “9” 时，抛出一个 RuntimeException：

```java
Mockito.when(positionService.getStr("9")).thenThrow(new RuntimeException("mock throw exception"));
String str = positionService.getStr("9"); // 会抛出一个RuntimeExceptio
```

测试结果：

```java
java.lang.RuntimeException: mock throw exception
	at com.example.springboot.tech.SpringBootTech1ApplicationTests.contextLoads(SpringBootTech1ApplicationTests.java:15)
```



2、如果方法没有返回值的话（即是方法定义为 public void myMethod() {…}），要改用 doThrow() 抛出 Exception：

```java
Mockito.doThrow(new RuntimeException("mock throw exception when method is void")).when(positionService).getVoid();
positionService.getVoid(); // 会抛出一个RuntimeException
```

测试结果：

```java
java.lang.RuntimeException: mock throw exception when method is void
	at com.example.springboot.tech.SpringBootTech1ApplicationTests.contextLoads(SpringBootTech1ApplicationTests.java:15)
```



##### verify 系列方法

1、检查调用 positionService 的 getStr() 方法，、且参数为 “3” 的次数是否为1次:

```java
Mockito.verify(positionService, Mockito.times(1)).getStr(Mockito.eq("3"));
```

2、验证调用顺序，验证 positionService 是否先调用 getStr() 两次，并且第一次的参数是 “3”、第二次的参数是 “5”，然后才调用 getVoid() 方法:

```java
InOrder inOrder = Mockito.inOrder(positionService);
inOrder.verify(positionService).getStr("3");
inOrder.verify(positionService).getStr("5");
inOrder.verify(positionService).getVoid();
```



##### 对 void 的方法设置模拟

使用 @Mock、@Spy 模拟 void 方法时，可以使用 Mockito 提供的 doNothing()、doThrow()、doAnswer() 等方法来控制 void 方法的行为。以下是几种常见的方法来模拟 void 方法：

- doNothing()：是 Mockito 默认的行为，用于模拟 void 方法时什么都不做。

  ```java
  doNothing().when(mockObject).voidMethod();
  ```

- doThrow()：用于指定在调用 void 方法时抛出异常。这在测试异常处理逻辑时很有用。

  ```java
  doThrow(new RuntimeException("Exception")).when(mockObject).voidMethod();
  ```

- doAnswer()：可以用来自定义 void 方法的行为，甚至可以根据传入的参数动态返回结果或执行逻辑。

  ```java
  doAnswer(invocation -> {
      System.out.println("Custom behavior in void method");
      return null;  // void 方法需要返回 null
  }).when(mockObject).voidMethod("param");
  ```

假设有一个类 MyService，其中有一个 void 方法 performAction()，我们可以使用 @Mock 来完全模拟这个方法的行为：

```java
public class MyService {
    public void performAction() {
        // 真实方法逻辑
        System.out.println("Performing action...");
    }
}
```

测试类中使用 @Mock 模拟 void 方法：

```java
import static org.mockito.Mockito.*;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(MockitoExtension.class)
public class MyServiceTest {

    @Mock
    private MyService myService;

    @Test
    public void testPerformActionWithDoNothing() {
        // 使用 doNothing 模拟 void 方法
        doNothing().when(myService).performAction();

        // 调用 performAction 方法
        myService.performAction();

        // 验证 performAction 被调用了一次
        verify(myService, times(1)).performAction();
    }

    @Test
    public void testPerformActionWithException() {
        // 使用 doThrow 模拟 void 方法抛出异常
        doThrow(new RuntimeException("Test exception")).when(myService).performAction();

        // 调用 performAction 方法并捕获异常
        try {
            myService.performAction();
        } catch (RuntimeException e) {
            assertEquals("Test exception", e.getMessage());
        }
    }

    @Test
    public void testPerformActionWithCustomBehavior() {
        // 使用 doAnswer 自定义 void 方法行为
        doAnswer(invocation -> {
            System.out.println("Custom behavior during test");
            return null;
        }).when(myService).performAction();

        // 调用 performAction 方法
        myService.performAction();

        // 验证 performAction 被调用了一次
        verify(myService, times(1)).performAction();
    }
}
```

**总结**：

- doNothing()：用于确保 void 方法的调用没有副作用。
- doThrow()：用于在调用 void 方法时抛出异常，便于测试异常处理逻辑。
- doAnswer()：用于自定义 void 方法的行为，提供最大灵活性。

@Mock 注解结合 doNothing()、doThrow() 和 doAnswer()，可以完全控制 void 方法的行为，适用于单元测试中对依赖的精确模拟。



##### 对 @Value 的属性模拟

如果测试的类中有如下配置

```java
@Value("${mock.mapping.name}")
private String  mockValue;
```

测试代码中需要如下设置配置值：

```java
ReflectionTestUtils.setField(mockService, "mockValue", "58699DFR-1456984524");
```



##### 对方法的参数进行模拟

在使用 Mockito 进行单元测试时，anyXxx() 方法用于匹配特定类型的任意参数。这些方法非常有用，特别是在编写测试时，你希望忽略某个参数的具体值，而只关注方法是否被调用，以及调用的其他参数。以下是常见的 anyXxx() 方法汇总：

**1. 基本类型**

- anyBoolean()：匹配任何 boolean 类型的参数。
- anyByte()：匹配任何 byte 类型的参数。
- anyChar()：匹配任何 char 类型的参数。
- anyDouble()：匹配任何 double 类型的参数。
- anyFloat()：匹配任何 float 类型的参数。
- anyInt()：匹配任何 int 类型的参数。
- anyLong()：匹配任何 long 类型的参数。
- anyShort()：匹配任何 short 类型的参数。

**2. 引用类型**

- any(Class< T > clazz)：匹配指定类型的任意对象。例如 any(String.class) 匹配任何 String 类型的对象。
- anyString()：匹配任何 String 类型的参数。
- anyList()：匹配任何 List 类型的参数。
- anySet()：匹配任何 Set 类型的参数。
- anyMap()：匹配任何 Map 类型的参数。
- anyCollection()：匹配任何 Collection 类型的参数。

**3. 其他常用方法**

- anyVararg()：匹配任意数量的变长参数。
- anyObject()：匹配任何 Object 类型的参数。这是一个泛型方法，可用于匹配任意类型的对象。
- anyIterable()：匹配任何 Iterable 类型的参数。



#### 4）Mock 测试常用注解

- 全部 Mock：这种方式，serviceA中的所有方法都会被mock，并不会真正被调用到

  ```java
  @Mock
  private ServiceA serviceA;
  ```

- 依赖注入：ServiceA 依赖了 ServiceC 和 DaoA，使用InjectMocks可以自动注入。

  ```java
  @InjectMocks
  private ServiceA serviceA;
  ```

- 真实调用 & 部分 Mock：这种方式，调用 serviceC 的方法，可以被真实调用。也可以 mock 调用。 

  ```java
  @Spy // 相当于实现了 @Mock + @InjectMocks 的功能
  private ServiceC serviceC;
  ```
  
  @Spy 的具体作用：
  
  - **部分模拟**：@Spy 使得你可以仅模拟对象的某些方法，而不是整个对象。这意味着你可以在测试中控制某些方法的返回值或行为，而其他未被模拟的方法仍然会执行真实的代码。
  - **与真实方法结合**：与 @Mock 不同，@Spy 会保留被测对象的真实行为。你可以选择性地 mock 一些方法，而不影响对象的整体行为。这对于测试需要与复杂对象交互的场景非常有用。
  - **继承自真实对象**：@Spy 通常用于那些你想要部分测试的真实对象。它会创建一个真实对象的 spy 实例，并允许你对这个实例进行部分 mock。
  
  @Spy 的使用场景：
  
  - **部分模拟**: 当你希望测试中的对象大部分行为保持真实，但需要控制或验证某些方法的调用。
  - **测试遗留代码**: 当你不能或不想重构现有的复杂对象，但希望在测试中控制其某些方法的行为。
  - **复杂的依赖关系**: 当一个对象依赖于外部服务或复杂的内部状态时，你可以使用 @Spy 来部分模拟该对象，而不是完全依赖于复杂的依赖关系。



### 4、@Mock、@InjectMocks、@Spy 案例

#### 1）@Mock 的使用

假设你有一个服务类 MyService，其中包含了调用其他服务的方法，你希望在测试中全部模拟这些调用：

```java
public class MyService {
    public String processData(String data) {
        return processInternal(data) + " processed";
    }

    public String processInternal(String data) {
        return data.toUpperCase();
    }
}
```

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
public class MyServiceTest {

    @Mock
    private MyService myService;

    @Test
    public void testProcessData() {
        // 使用 doReturn 来模拟 processInternal 和 processData 方法
        doReturn("mocked data processed").when(myService).processData(anyString());
        doReturn("mocked data").when(myService).processInternal(anyString());


        // 验证最终的结果
        assertEquals("mocked data processed", myService.processData(""));
        assertEquals("mocked data", myService.processInternal(""));

        // 验证 processInternal 方法被调用了0次
        verify(myService, times(0)).processInternal("input data");
    }
}
```



#### 2）@InjectMocks 的使用

假设你有一个服务类 MyService，其中包含了调用其他服务的方法，你希望在测试中全部真实的这些调用：

```java
public class MyService {
    public String processData(String data) {
        return processInternal(data) + " processed";
    }

    public String processInternal(String data) {
        return data.toUpperCase();
    }
}
```

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
public class MyServiceTest {

    @InjectMocks
    private MyService myService;

    @Test
    public void testProcessData() {

        // 验证最终的结果
        assertEquals("MOCKED DATA processed", myService.processData("mocked data"));
        assertEquals("MOCKED DATA", myService.processInternal("mocked data"));

        // 这里需要注意，由于上面为真实调用，所以这里会报错：
        // Argument passed to verify() is of type MyService and is not a mock!
        // 正确的做法应该是删除如下一行代码
        verify(myService, times(0)).processInternal("mocked data");
    }
}
```



#### 3）@Spy 的使用

@Spy 注解在 Mockito 中用于创建一个部分 mock 对象，即它可以让你部分模拟一个对象的方法，同时保留其他方法的真实行为。这在测试需要调用真实对象的一些方法，但希望控制或验证其他方法的行为时非常有用。

**@Spy 的使用示例-1**：假设你有一个服务类 MyService，其中包含了调用其他服务的方法，你希望在测试中部分模拟这些调用：

```java
public class MyService {
    public String processData(String data) {
        // 复杂的业务逻辑
        return processInternal(data) + " processed";
    }

    public String processInternal(String data) {
        // 模拟一个内部处理方法
        return data.toUpperCase();
    }
}
```

现在，你希望测试 processData 方法，但模拟 processInternal 方法的行为：

```java
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Spy;
import org.mockito.junit.jupiter.MockitoExtension;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(MockitoExtension.class)
public class MyServiceTest {

    @Spy
    // @InjectMocks 可以省略
    private MyService myService;

    @Test
    public void testProcessData() {
        // 使用 doReturn 来部分模拟 processInternal 方法
        doReturn("mocked data").when(myService).processInternal(anyString());

        // 调用 processData 方法，它会调用被部分模拟的 processInternal 方法
        String result = myService.processData("input data");

        // 验证最终的结果
        assertEquals("mocked data processed", result);

        // 验证 processInternal 方法被调用了一次
        verify(myService, times(1)).processInternal("input data");
    }
}
```

- @Spy: 用于创建 myService 的一个部分 mock 对象，其中 processInternal 方法的行为将被控制，但 processData 方法将保持原样。
- doReturn().when(): 用于模拟 processInternal 方法的行为，使其返回 mocked data，而不是实际调用 processInternal 方法。
- verify(): 用于验证 processInternal 方法是否被调用，并检查调用次数。

***

**@Spy 的使用示例-2**：假设有一个类 UserService，它有一个方法 findUserById 通过调用 userRepository 来查找用户。如果你想测试 findUserById 方法，但不希望依赖 userRepository 的实际实现，可以使用 @Spy。

```java
public class UserService {
    private UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findUserById(Long id) {
        User user = userRepository.findById(id);
        // 复杂的业务逻辑
        return user;
    }

    public void updateUser(User user) {
        // 另一个方法，不希望被 mock
        userRepository.save(user);
    }
}
```

```java
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Spy;
import org.mockito.junit.jupiter.MockitoExtension;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(MockitoExtension.class)
public class UserServiceTest {

    @Spy
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    public void testFindUserById() {
        // 模拟 userRepository.findById 方法
        User mockUser = new User(1L, "John Doe");
        doReturn(mockUser).when(userRepository).findById(1L);

        // 调用 userService.findUserById
        User result = userService.findUserById(1L);

        // 验证结果
        assertEquals("John Doe", result.getName());

        // 验证 findById 方法是否被调用
        verify(userRepository).findById(1L);

        // userService 的其他方法仍然可以调用真实的 userRepository 方法
        userService.updateUser(mockUser);
        verify(userRepository).save(mockUser);
    }
}
```

@Spy 使用总结：

- 保留真实行为: @Spy 会保留对象的大部分真实行为，并且可以有选择地模拟某些方法。
- 部分控制: 适用于你希望部分控制对象行为的场景，而不是完全替换对象。
- 测试灵活性: 提供了一种灵活的方式来测试复杂对象或依赖关系较多的场景，而不需要重构代码。



#### 4）@Mock、@InjectMocks、@Spy 区别

**@Mock**:

- 创建一个完全模拟的对象。
- 所有方法调用都会被拦截，可以返回预设的模拟值。
- 适用于单元测试中隔离依赖项。

**@InjectMocks**:

- 自动注入所有 @Mock 或 @Spy 对象到被测试的对象中。
- 适用于依赖注入较多的类，简化测试类的设置。
- 不会直接模拟任何对象，而是帮助将依赖注入到被测试对象中。

**@Spy**:

- 创建一个部分模拟的对象。
- 保留对象的真实行为，但允许选择性地模拟其中的方法。
- 适用于需要测试真实对象部分行为的场景。

**组合使用**：

通常，@Mock 和 @InjectMocks 会组合使用，以创建和注入依赖的 mock 对象。而 @Spy 则在需要保留部分真实行为时使用。以下是一个组合使用的示例：

```java
@ExtendWith(MockitoExtension.class)
public class MyServiceTest {
    @Mock
    private DependencyService dependencyService;
    @InjectMocks
    private MyService myService;
    @Spy
    private AnotherDependencyService anotherDependencyService;

    @Test
    public void testMyService() {
        when(dependencyService.someMethod()).thenReturn("mocked response");
        doReturn("spy response").when(anotherDependencyService).anotherMethod();

        String result = myService.processData("input");

        assertEquals("expected result", result);
    }
}
```

在这个示例中，@Mock 用于完全模拟 DependencyService，@Spy 用于部分模拟 AnotherDependencyService，而 @InjectMocks 则用于将它们注入到 MyService 中进行测试。



### 5、when..thenReturn.. 与 doReturn..when.. 的区别

在 Mockito 中，doReturn().when() 和 when().thenReturn() 都用于模拟方法的返回值，但它们在使用场景和工作原理上有一些细微的差别。

#### 1）when().thenReturn()

```java
when(mockObject.someMethod()).thenReturn(expectedValue);

when(mockList.get(0)).thenReturn("Mocked String");
```

**工作原理**

- when().thenReturn() 是 Mockito 模拟方法行为的标准方式。当你调用 when() 时，它实际上执行了传入的那个方法，并记录该方法调用的行为。
- 如果该方法调用在 when() 内部抛出异常（如空指针异常）或改变了状态，则会导致意外行为或错误。

**适用场景**

- 当模拟的方法不涉及副作用、不抛出异常，或者你明确知道调用该方法不会有问题时，使用 when().thenReturn() 是首选。



#### 2）doReturn().when()

```java
doReturn(expectedValue).when(mockObject).someMethod();

doReturn("Mocked String").when(mockList).get(0);
```

**工作原理**

- doReturn().when() 不会立即执行 someMethod()。相反，它在指定了行为之后再去模拟方法调用。这避免了在 when() 中直接调用方法可能导致的副作用或异常。
- doReturn() 的优势在于可以在需要避免方法实际执行时使用，例如当被模拟的方法调用可能抛出异常或有其他副作用时。

**适用场景**

- 当你模拟的方法可能抛出异常或你不希望实际调用该方法（因为它可能有副作用或未被完全设置）时，使用 doReturn().when() 会更加安全。
- 它也是模拟 void 方法行为的推荐方式。



#### 3）区别总结

**方法执行**：

- when().thenReturn() 会在 when() 内部执行方法调用并记录其行为。
- doReturn().when() 不会立即执行方法调用，而是先定义返回值，再指定哪个方法要模拟。

**异常处理**：

- 如果模拟的方法在调用时可能抛出异常，doReturn().when() 是更安全的选择，因为它不会在模拟过程中执行实际的方法调用。

**使用场景**：

- when().thenReturn() 是标准的、最常用的模拟方法的方式，适用于大多数返回非 void 类型的方法。
- doReturn().when() 更适合在模拟有副作用或抛出异常的方法时使用，尤其是那些你不希望在模拟过程中实际调用的方法。

**最终总结**：

- when().thenReturn() 是更直接的模拟方式，适用于大多数场景。
- doReturn().when() 提供了更大的灵活性，特别适合处理那些可能在 when() 内部引发问题的方法。



#### 4）示例对比

假设我们有一个可能抛出异常的方法：

```java
public String riskyMethod() {
    throw new RuntimeException("Error");
}
```

- 使用 when..thenReturn.. 模拟

  ```java
  // 这可能会抛出异常，导致测试失败
  when(mockObject.riskyMethod()).thenReturn("Safe Value");
  ```

- 使用 doReturn..when.. 模拟

  ```java
  // 这不会调用 riskyMethod()，因此不会抛出异常
  doReturn("Safe Value").when(mockObject).riskyMethod();
  ```



## 五、Mock 测试结合 Java 反射综合案例

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.math.BigDecimal;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;

import org.junit.Before;
import org.junit.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.test.util.ReflectionTestUtils;

// @RunWith(MockitoJUnitRunner.class)
@ExtendWith(MockitoExtension.class)
public class MyMockServiceTest {

    @Mock
    private CustomerDao customerDao;

    @Mock
    private MockDaoA mockDaoA;

    @Mock
    private MockDaoC mockDaoC;

    @Mock
    private MockDaoD mockDaoD;

    @Mock
    private MockDaoE mockDaoE;

    @InjectMocks
    MyMockService myMockService;

    MockTestDataDto mockTestDataDto;

    @Before
    public void init() {

        // Apollo 配置
        ReflectionTestUtils.setField(myMockService, "mockValue", "58699DFR-1456984524");

        MockitoAnnotations.initMocks(this);

        mockTestDataDto = new MockTestDataDto();
        mockTestDataDto.setCallback("callback");

        PolicyRelatedInfo policyRelatedInfo = new PolicyRelatedInfo();
        policyRelatedInfo.setRelationToAppnt("1");
        Mockito.when(mockDaoA.selectRelationByParams(Mockito.any()))
            .thenReturn(policyRelatedInfo);

        Customer customer = new Customer();
        insu.setPhone("4654");
        insu.setSex("1");
        insu.setIdType("1");
        insu.setIdNo("1");
        insu.setName("张三");
        insu.setBirthday(new Date());
        Mockito.when(customerDao.selectByPrimaryKey(Mockito.anyInt())).thenReturn(customer);

    }

    @Test
    public void test() throws NoSuchMethodException, SecurityException, IllegalAccessException,
    IllegalArgumentException, InvocationTargetException, InstantiationException {

        ArrayList<PolicyDanger> getPolicyDangerList = new ArrayList<>();


        PolicyDanger policyDanger1 = new PolicyDanger();
        policyDanger1.setIsPassFlag("M");
        policyDanger1.setDanderCode("595648FD");
        policyDanger1.setTotalAmnt(new BigDecimal(100.1223));



        PolicyDanger policyDanger2 = new PolicyDanger();
        policyDanger2.setIsPassFlag("M");
        policyDanger2.setDanderCode("595648FD");
        policyDanger2.setTotalAmnt(new BigDecimal(100.1223));

        getPolicyDangerList.add(policyDanger1);
        getPolicyDangerList.add(policyDanger2);
        Mockito.when(mockDaoC.selectPolicyDangerList(Mockito.any())).thenReturn(getPolicyDangerList);

        ArrayList<Province> provinceList = new ArrayList<>();

        Province province  = new Province();
        province.setProvinceCode("5894");
        province.setDutyCode("5928D2");
        provinceList.add(province);
        Mockito.when(mockDaoD.selectPolicyByQueryParam(Mockito.any())).thenReturn(provinceList);

        ArrayList<User> userList = new ArrayList<>();
        User user = new User();
        user.setBuyDate(new Date());
        userList.add(user);
        Mockito.when(mockDaoE.selectUserByQueryParam(Mockito.any())).thenReturn(userList);

        // 反射获得类
        MyMockService  hx = new MyMockService();
        Class<? extends MyMockService> cls1 = hx.getClass();

        // 通过指定方法名称和参数类型的方法来获取Method对象
        // (注意: 如果方法名称不存在或参数类型不正确的话,会报错,不会返回null)
        // 注意：这里测试的是 private 修饰的私有方法，需要用 getDeclaredMethod
        // setUserInfo 是需要测试的方法名，后面为该方法需要的参数类型
        Method method = cls1.getDeclaredMethod("setUserInfo", 
                                               MockTestDataDto.class, 
                                               Integer.class, 
                                               String.class,
                                               SimpleDateFormat.class);
        method.setAccessible(true);

        // 执行方法
        method.invoke(myMockService, mockTestDataDto, 1, "1", new SimpleDateFormat());

    }
}
```



# Mock & Mockito 详解 

> 1. 作者：武哥聊编程]；来源：https://mp.weixin.qq.com/s/MowhCzlMG48deW7NkVQkxQ
>
> 2. 作者：编了个程（Yasin x）；来源：https://mp.weixin.qq.com/s/pjT8D3iEYauSWJas_xGbDg

Mockito 是一种 Java Mock 框架，他主要是用来做 mock 测试的，他可以模拟任何 Spring 管理的 Bean、模拟方法的返回值、模拟抛出异常...等，在了解 Mockito 的具体用法之前，得先了解什麽是 mock 测试



## 一、  Mock 与 Mockito 前言

### 1、Mock 测试是什么

mock 测试就是在测试过程中，创建一个假的对象，避免你为了测试一个方法，却要自行构建整个 bean 的依赖链

像是以下这张图，类 A 需要调用类 B 和类 C，而类 B 和类 C 又需要调用其他类如 D、E、F 等，假设类 D 是一个外部服务，那就会很难测，因为你的返回结果会直接的受外部服务影响，导致你的单元测试可能今天会过、但明天就过不了了

```
Class A────>Class B────>Class D
   │          │
   ▼          ▼
Class C    Class E   
   │
   ▼
Class F
```

而当我们引入 mock 测试时，就可以创建一个假的对象，替换掉真实的 bean B 和 C，这样在调用B、C的方法时，实际上就会去调用这个假的 mock 对象的方法，而我们就可以自己设定这个 mock 对象的参数和期望结果，让我们可以专注在测试当前的类 A，而不会受到其他的外部服务影响，这样测试效率就能提高很多

```java
Class A────>Class B
   │
   ▼
Class C
```



### 2、Mockito 简介

说完了 mock 测试的概念，接下来我们进入到今天的主题，Mockito

Mockito 是一种 Java mock 框架，他主要就是用来做 mock 测试的，他可以模拟任何 Spring 管理的 bean、模拟方法的返回值、模拟抛出异常...等，他同时也会记录调用这些模拟方法的参数、调用顺序，从而可以校验出这个 mock 对象是否有被正确的顺序调用，以及按照期望的参数被调用

像是 Mockito 可以在单元测试中模拟一个 service 返回的数据，而不会真正去调用该 service，这就是上面提到的 mock 测试精神，也就是通过模拟一个假的 service 对象，来快速的测试当前我想要测试的类

目前在 Java 中主流的 mock 测试工具有 Mockito、JMock、EasyMock..等，而 SpringBoot 目前内建的是 Mockito 框架

题外话说一下，Mockito 是命名自一种调酒莫吉托（Mojito），外国人也爱玩谐音梗。。。



## 二、Mockito 的基本用法

Mockito是Java语言非常主流的一个框架，自己使用起来感觉也比较好用。所以这篇文章想汇总介绍一下Mockito的各种用法，这样大家在以后写单元测试和看单元测试的时候，就能够比较清晰为什么要这么写。

### 1、设置Mockito环境

要使用Mockito，首先得在测试类里面设置好Mockito环境。这是为了能够让单元测试框架能够识别和使用Mockito。

在JUnit 4， JUnit使用了@RunWith注解来声明一个“运行器”。这个运行期的作用是为单元测试提供**「mock的初始化工作」**（比如使用@Mock、@Spy等注解时，需要初始化），以及**「验证mock语法」**的功能。

比如我们可能会经常用到的：

```java
@RunWith(JUnit4.class)
@RunWith(SpringRunner.class)
@RunWith(SpringJUnit4ClassRunner.class)
```

Mockito 也有相应的启动器，JUnit 4，在@RunWith注解上面使用这个启动器就可以使用Mockito的环境了：

```java
@RunWith(MockitoJUnitRunner.class) // JUnit5 之前
```

在JUnit 5，使用了@ExtendWith注解来代替@RunWith注解，Mockito也支持JUnit 5，提供了MockitoExtension类。

```java
@ExtendWith(MockitoExtension.class) // JUnit5 之后
```

除了使用注解以外，也可以使用静态方法initMocks来实现这个功能：

```java
MockitoAnnotations.initMocks(this) 
```



### 2、mock

mock，即mock一个对象。也是也注解和代码两种方式可以实现。

```JAVA
// 方式一
@Mock
private User user;
// 方式二
Order order = Mockito.mock(Order.class);
```

mock对象后，就可以对它使用given等方法模拟它的输入和输出。

```
Mockito.given(order.getId()).willReturn(1L);
Mockito.when(order.getId()).willReturn(1L);
assertEquals(order.getId(), 1L);
```

 

### 3、spy

mock出来的对象是完全虚拟的，不会真正地调用本来的实现。如果不对它使用given等方法，会返回默认值(null, 0, false等)。而spy如果不使用given等方法，会调用这个对象本来的实现，返回实际运行后的值。

> 不是很推荐使用spy，因为它没有消除依赖

spy同样有注解和静态方法的方式：

```java
// 方式一
@Spy
private user user;
// 方式二
Order order = Mockito.spy(Order.class);
```



### 4、captor

captor翻译过来是“捕获”的意思，主要用来捕捉程序运行时调用mock或者spy的对象的方法时，传入的参数。它支持泛型，即要捕获的参数的类型。

```java
@Captor 
ArgumentCaptor<User> userCaptor;
ArgumentCaptor<String> arg = ArgumentCaptor.forClass(String.class);
```

captor一般是与given或者verify等方法配合使用。

```java
@Mock
List mockedList;

@Captor 
ArgumentCaptor argCaptor;

@Test
public void whenUseCaptorAnnotation_thenTheSam() {
    mockedList.add("one");
    Mockito.verify(mockedList).add(argCaptor.capture());
    assertEquals("one", argCaptor.getValue());
}
```



### 5、InjectMocks

使用@InjectMocks注解，可以将mock或spy的对象自动注入要测试的对象。这在Spring等使用自动注入的框架里用得非常广泛。

```java
@Mock
Map<String, String> wordMap;

@InjectMocks
MyDictionary dic = new MyDictionary();

// 类定义：
class MyDictionary {
    Map<String, String> wordMap;

    public MyDictionary() {
        wordMap = new HashMap<String, String>();
    }
    public void add(final String word, final String meaning) {
        wordMap.put(word, meaning);
    }
    public String getMeaning(final String word) {
        return wordMap.get(word);
    }
}
```



## 三、Mock 打桩（了解）

所谓打桩，其实就是mock的一个过程。我们给定期望的输入和输入，mock的对象就能够如我们期望的那样工作。

打桩有两种写法，一种是传统的写法，一种是BDD **Behavior-Driven Development (行为驱动开发)**的写法。

### 1、传统写法

我们先看看传统的写法，传统的写法主要用`Mockito`类的方法。基本是when-then-invoke-verify形式。

```java
when(phoneBookRepository.contains(momContactName)).thenReturn(false);

phoneBookService.register(momContactName, momPhoneNumber);

verify(phoneBookRepository).insert(momContactName, momPhoneNumber);
```



### 2、BDD写法

BDD写法主要是用`BDDMockito`类的方法，看起来是given–will-invoke-then的形式。

```java
given(phoneBookRepository.contains(momContactName)).willReturn(false);
 
phoneBookService.register(momContactName, momPhoneNumber);
 
then(phoneBookRepository).should().insert(momContactName, momPhoneNumber);
```



### 3、动态mock

有时候我们可能会遇到这个问题：在方法内部可能有循环或者递归，会多次调用其它对象的方法，但输入的参数不同，我们在mock的时候，期望它根据不同的输入参数返回不同的结果。这个时候如果一个一个写given，就需要写很多次。但其实可以用动态mock来做。它是基于`InvocationOnMock`来实现的。

```java
given(phoneBookRepository.contains(momContactName)).willReturn(true);

given(phoneBookRepository.getPhoneNumberByContactName(momContactName))
    .will((InvocationOnMock invocation) ->
          invocation.getArgument(0).equals(momContactName) 
          ? momPhoneNumber 
          : null);

phoneBookService.search(momContactName);

then(phoneBookRepository).should().getPhoneNumberByContactName(momContactName);
```



## 四、Mockito 代码实战

### 1、代码测试

首先在 pom.xml 下新增 spring-boot-starter-test 依赖，该依赖内就有包含了 JUnit、Mockito

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

先写好一个 UserService，他里面有两个方法 getUserById() 和 insertUser()，而他们会分别去再去调用 UserDao 这个 bean的 getUserById() 和 insertUser()方法

```java
@Component
publicclass UserService {

    @Autowired
    private UserDao userDao;

    public User getUserById(Integer id) {
        return userDao.getUserById(id);
    }

    public Integer insertUser(User user) {
        return userDao.insertUser(user);
    }
}
```

User model 的定义如下

```java
@Data
public class User {
    private Integer id;
    private String name;
}
```

如果这时候我们先不使用 Mockito 模拟一个假的 userDao bean，而是真的去调用一个正常的 Spring bean 的 userDao 的话，测试类写法如下。其实就是很普通的注入 userService bean，然后去调用他的方法，而他会再去调用 userDao 取得数据库的数据，然后我们再对返回结果做 assert 断言检查

```java
// 如果时Junit5,可以不加 @RunWith(SpringRunner.class)
@RunWith(SpringRunner.class)
@SpringBootTest
publicclass UserServiceTest {

    // 先普通的注入一个userService bean
    @Autowired
    private UserService userService;

    @Test
    public void getUserById() throws Exception {
        // 普通的使用userService，他里面会再去调用userDao取得数据库的数据
        User user = userService.getUserById(1);

        // 检查结果
        Assert.assertNotNull(user);
        Assert.assertEquals(user.getId(), new Integer(1));
        Assert.assertEquals(user.getName(), "John");
    }
}
```

但是如果 userDao 还没写好，又想先测 userService 的话，就需要使用 Mockito 去模拟一个假的 userDao 出来

使用方法是在 userDao 上加上一个 @MockBean 注解，当 userDao 被加上这个注解之后，表示 Mockito 会帮我们创建一个假的 mock 对象，替换掉 Spring 中已存在的那个真实的 userDao bean，也就是说，注入进 userService 的 userDao bean，已经被我们替换成假的 mock 对象了，所以当我们再次调用 userService 的方法时，会去调用的实际上是 mock userDao bean 的方法，而不是真实的 userDao bean

当我们创建了一个假的 userDao 后，我们需要为这个 mock userDao 自定义方法的返回值，这里有一个公式用法，下面这段代码的意思为，当调用了某个 mock 对象的方法时，就回传我们想要的自定义结果

```java
Mockito.when( 对象.方法名() ).thenReturn( 自定义结果 );
```

使用 Mockito 模拟 Bean 的单元测试具体实例如下：

```java
@SpringBootTest
publicclass UserServiceTest {
    
    @Autowired
    private UserService userService;
    /**
     * 使用@Mock注解对测试接口进行修饰。该注解位于org.mockito.Mock中
     *  使用@mock注解会创建 Mock 对象 userDao
     *  等同于之前代码：
     *  UserDao userDao = Mockito.mock(UserDao.class);
     *  注意：使用 @MockBean 或 @Mock 都可以
     */
    @MockBean
    private UserDao userDao;

    @Test
    public void getUserById() throws Exception {
        // UserDao userDao = Mockito.mock(UserDao.class);
        // 定义当调用mock userDao的getUserById()方法，并且参数为3时，就返回id为200、name为I'm mock3的user对象
        Mockito.when(userDao.getUserById(3)).thenReturn(new User(200, "I'm mock 3"));
        
        // 返回的会是名字为I'm mock 3的user对象
        User user = userService.getUserById(1);

        Assert.assertNotNull(user);
        Assert.assertEquals(user.getId(), new Integer(200));
        Assert.assertEquals(user.getName(), "I'm mock 3");
    }
}
```

说明：

1. 使用@Mock注解对测试接口进行修饰，使用@Mock 或 @MockBean注解会创建 Mock 对象，等同于之前的代码：

   ```java
   UserDao userDao = Mockito.mock(UserDao.class);
   ```

2. `Mockito.when(userDao.getUserById(3)).thenReturn(new User(200, "I'm mock 3"));`

   MOCK设置：当调用getUserById(3)的时候，返回mock User对象。

Mockito 除了最基本的 `Mockito.when( 对象.方法名() ).thenReturn( 自定义结果 )`，还提供了其他用法让我们使用



### 2、thenReturn 系列方法

当使用任何整数值调用 userService 的 getUserById() 方法时，就回传一个名字为 I'm mock3 的 user 对象

```java
Mockito.when(userService.getUserById(Mockito.anyInt())).thenReturn(new User(3, "I'm mock"));
User user1 = userService.getUserById(3); // 回传的user的名字为I'm mock
User user2 = userService.getUserById(200); // 回传的user的名字也为I'm mock
```

限制只有当参数的数字是 3 时，才会回传名字为 I'm mock 3 的 user 对象

```java
Mockito.when(userService.getUserById(3)).thenReturn(new User(3, "I'm mock"));
User user1 = userService.getUserById(3); // 回传的user的名字为I'm mock
User user2 = userService.getUserById(200); // 回传的user为null
```

当调用 userService 的 insertUser() 方法时，不管传进来的 user 是什麽，都回传 100

```java
Mockito.when(userService.insertUser(Mockito.any(User.class))).thenReturn(100);
Integer i = userService.insertUser(new User()); //会返回100
```



### 3、thenThrow 系列方法

当调用 userService 的 getUserById() 时的参数是 9 时，抛出一个 RuntimeException

```java
Mockito.when(userService.getUserById(9)).thenThrow(new RuntimeException("mock throw exception"));
User user = userService.getUserById(9); // 会抛出一个RuntimeException
```

如果方法没有返回值的话（即是方法定义为`public void myMethod() {...}`），要改用 doThrow() 抛出 Exception

```java
Mockito.doThrow(new RuntimeException("mock throw exception")).when(userService).print();
userService.print(); // 会抛出一个RuntimeException
```



### 4、verify 系列方法

检查调用 userService 的 getUserById()、且参数为3的次数是否为1次

```java
Mockito.verify(userService, Mockito.times(1)).getUserById(Mockito.eq(3));
```

验证调用顺序，验证 userService 是否先调用 getUserById() 两次，并且第一次的参数是 3、第二次的参数是 5，然后才调用insertUser() 方法

```java
InOrder inOrder = Mockito.inOrder(userService);
inOrder.verify(userService).getUserById(3);
inOrder.verify(userService).getUserById(5);
inOrder.verify(userService).insertUser(Mockito.any(User.class));
```



## 五、Mockito 的限制

上述就是 Mockito 的 mock 对象使用方法，不过当使用 Mockito 在 mock 对象时，有一些限制需要遵守

- 不能 mock 静态方法
- 不能 mock private 方法
- 不能 mock final class

因此在写代码时，需要做良好的功能拆分，才能够使用 Mockito 的 mock 技术，帮助我们降低测试时 bean 的耦合度



## 六、Mockito 总结

Mockito 是一个非常强大的框架，可以在执行单元测试时帮助我们模拟一个 bean，提高单元测试的稳定性

并且大家可以尝试在写代码时，从 Mock 测试的角度来写，更能够写出功能切分良好的代码架构，像是如果有把专门和外部服务沟通的代码抽出来成一个 Bean，在进行单元测试时，只要透过 Mockito 更换掉那个 Bean 就行了

