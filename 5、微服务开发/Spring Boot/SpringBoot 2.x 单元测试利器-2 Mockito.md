> 作者：武哥聊编程]；来源：https://mp.weixin.qq.com/s/MowhCzlMG48deW7NkVQkxQ
>
> 作者：编了个程（Yasin x）；来源：https://mp.weixin.qq.com/s/pjT8D3iEYauSWJas_xGbDg

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

Mockito也有相应的启动器，在@RunWith注解上面使用这个启动器就可以使用Mockito的环境了：

```java
@RunWith(MockitoJUnitRunner)
```

在JUnit 5，使用了@ExtendWith注解来代替@RunWith注解，Mockito也支持JUnit 5，提供了MockitoExtension类。

```java
@ExtendWith(MockitoExtension.class)
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
@Spy
private user user;
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
Mockito.when( 对象.方法名() ).thenReturn( 自定义结果 )
```

使用 Mockito 模拟 bean 的单元测试具体实例如下：

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

2.  `Mockito.when(userDao.getUserById(3)).thenReturn(new User(200, "I'm mock 3"));`

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
User user = userService.getUserById(9); //会抛出一个RuntimeException
```

如果方法没有返回值的话（即是方法定义为`public void myMethod() {...}`），要改用 doThrow() 抛出 Exception

```java
Mockito.doThrow(new RuntimeException("mock throw exception")).when(userService).print();
userService.print(); //会抛出一个RuntimeException
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