> 作者：Java学习之道，来源：https://mp.weixin.qq.com/s/xQFjjHNqDV_XuOIYvKXGsQ
>
> 作者：Java知音，来源：https://mp.weixin.qq.com/s/9PQVeJCl86JIatCGIzyXtQ

## 一、Spring 依赖注入对比前言

本章的内容主要是想探讨我们在进行Spring 开发过程当中，关于依赖注入的几个知识点。感兴趣的读者可以先看下以下问题：

- @Autowired、@Resource、@Inject 三个注解的区别
- 当你在使用@Autowired时，是否有出现过`Field injection is not recommended`的警告？你知道这是为什么吗？
- Spring 依赖注入有哪几种方式？官方是怎么建议使用的呢？

如果你对上述问题都了解，那我个人觉得你的开发经验应该是不错的。

下面我们就依次对上述问题进行解答，并且总结知识点。



## 二、Spring 依赖注入三个注解的区别

Spring 支持使用 @Autowired、@Resource,、@Inject 三个注解进行依赖注入。下面来介绍一下这三个注解有什么区别。

### 1、@Autowired

@Autowired 为 Spring 框架提供的注解，需要导入包：**org.springframework.beans.factory.annotation.Autowired**

这里先给出一个示例代码，方便讲解说明：

```java
public interface Svc {

    void sayHello();
}

@Service
public class SvcA implements Svc {

    @Override
    public void sayHello() {
        System.out.println("hello, this is service A");
    }

}

@Service
public class SvcB implements Svc {

    @Override
    public void sayHello() {
        System.out.println("hello, this is service B");
    }

}

@Service
public class SvcC implements Svc {

    @Override
    public void sayHello() {
        System.out.println("hello, this is service C");
    }
}
```

测试类：

```java
@SpringBootTest
public class SimpleTest {

    @Autowired
    // @Qualifier("svcA")
    Svc svc;

    @Test
    void rc() {
        Assertions.assertNotNull(svc);
        svc.sayHello();
    }
}
```

**装配顺序：**

1. 按照type在上下文中查找匹配的bean：**查找type为Svc的bean**
2. 如果有多个Bean，则按照name进行匹配
   1. 如果有`@Qualifier`注解，则按照`@Qualifier`指定的`name`进行匹配：**查找name为svcA的bean**
   2. 如果没有，则按照变量名进行匹配：**查找name为svc的bean**
3. 匹配不到，则报错。设置@Autowired(required=false)，如果设置required为false（默认为true），则注入失败时不会抛出异常



### 2、@Inject

在Spring 的环境下，@Inject 和 @Autowired 是相同的，因为它们的依赖注入都是使用AutowiredAnnotationBeanPostProcessor来处理的。

```java
	/**
	 * Create a new {@code AutowiredAnnotationBeanPostProcessor} for Spring's
	 * standard {@link Autowired @Autowired} and {@link Value @Value} annotations.
	 * <p>Also supports JSR-330's {@link javax.inject.Inject @Inject} annotation,
	 * if available.
	 */
	@SuppressWarnings("unchecked")
	public AutowiredAnnotationBeanPostProcessor() {
		this.autowiredAnnotationTypes.add(Autowired.class);
		this.autowiredAnnotationTypes.add(Value.class);
		try {
			this.autowiredAnnotationTypes.add((Class<? extends Annotation>)
					ClassUtils.forName("javax.inject.Inject", AutowiredAnnotationBeanPostProcessor.class.getClassLoader()));
			logger.trace("JSR-330 'javax.inject.Inject' annotation found and supported for autowiring");
		}
		catch (ClassNotFoundException ex) {
			// JSR-330 API not available - simply skip.
		}
	}
```

@Inject是 JSR-330 定义的规范，如果使用这种方式，切换到Guice也是可以的。

> Guice 是 google 开源的轻量级 DI 框架

如果硬要说两者的区别，首先@Inject是JavaEE包的，在SE环境需要单独引入。另外区别在于@Autowired可以设置required=false而@Inject并没有这个属性。



### 3、@Resource

@Resource是JSR-250定义的注解。Spring 在CommonAnnotationBeanPostProcessor实现了对JSR-250的注解的处理，其中就包括@Resource

```java
	@Override
	public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
		InjectionMetadata metadata = findResourceMetadata(beanName, bean.getClass(), pvs);
		try {
			metadata.inject(bean, beanName, pvs);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Injection of resource dependencies failed", ex);
		}
		return pvs;
	}
```

@Resource有两个重要的属性：name和type，而Spring 将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。

**装配顺序：**

1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常
4. 如果既没有指定name，又没有指定type，则默认按照byName方式进行装配；如果没有匹配，按照byType进行装配



## 三、IDEA 提示 Field injection is not recommended

在使用IDEA 进行Spring 开发的时候，当你在字段上面使用@Autowired注解的时候，你会发现IDEA 会有警告提示：

> Field injection is not recommended
>
> Inspection info: Spring Team Recommends: "Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies".

翻译过来就是这个意思：

> 不建议使用基于 field 的注入方式。
>
> Spring 开发团队建议：在你的Spring Bean 永远使用基于constructor 的方式进行依赖注入。对于必须的依赖，永远使用断言来确认。

比如如下代码：

```java
public interface Svc {
    void sayHello();
}

@Service
public class SvcB implements Svc {
    @Override
    public void sayHello() {
        System.out.println("hello, this is service B");
    }
}

@Service
public class HelpService {
    @Autowired
    @Qualifier("svcB")
    private Svc svc;

    public void sayHello() {
        svc.sayHello();
    }
}
```

将光标放到@Autowired处，使用Alt + Enter 快捷进行修改之后，代码就会变成基于Constructor的注入方式，**修改之后**：

```java
@Service
public class HelpService {
    private final Svc svc;

    @Autowired
    public HelpService(@Qualifier("svcB") Svc svc) {
        // Assert.notNull(svc, "svc must not be null");
        this.svc = svc;
    }

    public void sayHello() {
        svc.sayHello();
    }
}
```

如果按照Spring 团队的建议，如果svc是必须的依赖，应该使用`Assert.notNull(svc, "svc must not be null")`来确认。

修正这个警告提示固然简单，但是我觉得更重要是去理解为什么Spring 团队会提出这样的建议？直接使用这种基于 field 的注入方式有什么问题？



## 四、Spring 常见的三种依赖注入方式

首先我们需要知道，Spring 中有这么3种依赖注入的方式：

- 基于 field 注入（属性注入）
- 基于 setter 注入（方法注入）
- 基于 constructor 注入（构造器注入）



### 1、基于 field 注入

所谓基于 field 注入，就是在bean的变量上使用注解进行依赖注入。本质上是通过反射的方式直接注入到field。这是我平常开发中看的最多也是最熟悉的一种方式，同时，也正是 Spring 团队所不推荐的方式。比如：

```java
@Autowired
private Svc svc;
```



### 2、基于 setter 方法注入

通过对应变量的`setXXX()`方法以及在方法上面使用注解，来完成依赖注入。比如：

```java
private Helper helper;

@Autowired
public void setHelper(Helper helper) {
    this.helper = helper;
}
```

> 注：在 Spring 4.3 及以后的版本中，setter 上面的 @Autowired 注解是可以不写的。



### 3、基于 constructor 注入

将各个必需的依赖全部放在带有注解构造方法的参数中，并在构造方法中完成对应变量的初始化，这种方式，就是基于构造方法的注入。比如：

```java
private final Svc svc;
    
@Autowired
public HelpService(@Qualifier("svcB") Svc svc) {
    this.svc = svc;
}
```

> 在 Spring 4.3 及以后的版本中，如果这个类只有一个构造方法，那么这个构造方法上面也可以不写 @Autowired 注解。
>
> **为什么加上构造器，不用@Autowired也能把对象注入Spring容器？**
>
> 1. 在Java语言中，每个类至少有一个构造方法。如果程序中没有显式定义任何构造方法，那么将自动提供一个隐含的默认无参构造方法。
> 2. 只要程序中已经显式定义了构造方法，那么将不再提供隐含的默认构造方法。
> 3. 所以这里定义了一个带参数的构造器，在初始化这个类的时候需要传入一个对象，这个Spring容器就把它创建了，所有内部不用加@Autowired



## 五、Spring 不同依赖注入的优缺点

### 1、基于 field 注入的好处

正如你所见，这种方式非常的简洁，代码看起来很简单，通俗易懂。你的类可以专注于业务而不被依赖注入所污染。你只需要把@Autowired扔到变量之上就好了，不需要特殊的构造器或者set方法，依赖注入容器会提供你所需的依赖。



### 2、基于 field 注入的坏处

基于 field 注入虽然简单，但是却会引发很多的问题。这些问题在我平常开发阅读项目代码的时候就经常遇见。

- **容易违背了单一职责原则** 使用这种基于 field 注入的方式，添加依赖是很简单的，就算你的类中有十几个依赖你可能都觉得没有什么问题，普通的开发者很可能会无意识地给一个类添加很多的依赖。但是当使用构造器方式注入，到了某个特定的点，构造器中的参数变得太多以至于很明显地发现something is wrong。拥有太多的依赖通常意味着你的类要承担更多的责任，明显违背了单一职责原则（SRP：Single responsibility principle）。

- **依赖注入与容器本身耦合**

  依赖注入框架的核心思想之一就是受容器管理的类不应该去依赖容器所使用的依赖。换句话说，这个类应该是一个简单的POJO（Plain Ordinary Java Object）能够被单独实例化并且你也能为它提供它所需的依赖。

  这个问题具体可以表现在：

  - 你的类和依赖容器强耦合，不能在容器外使用
  - 你的类不能绕过反射（例如单元测试的时候）进行实例化，必须通过依赖容器才能实例化，这更像是集成测试

  ```java
  @Controller
  public class FooController {
      @Autowired
      private FooService fooService;
      public List<Foo> listFoo() {
          return fooService.list();
      }
  }
  
  // 承接上面field注入的代码，假如客户端代码使用下面的调用(或者再Junit测试中使用)
  // 这里只是模拟一下，正常来说我们只会暴露接口给客户端，不会暴露实现。
  FooController fooController = new FooController();
  fooController.listFoo(); // -> NullPointerException

- **不能使用属性注入的方式构建不可变对象（final 修饰的变量）**

- 还值得一提另外一点是：使用field注入可能会导致循环依赖，即A里面注入B，B里面又注入A：

  ```java
  public class A {
      @Autowired
      private B b;
  }
  
  public class B {
      @Autowired
      private A a;
  }
  ```

  > 如果使用构造器注入能够避免上述循环依赖这种情况



### 3、Constructor 注入的好处

造器注入的方式，能够保证注入的**组件不可变**，并且确保需要的**依赖不为空**。此外，构造器注入的依赖总是能够在返回客户端（组件）代码的时候保证**完全初始化的状态**。

- 依赖不可变：其实说的就是final关键字，这里不再多解释了。不明白的园友可以回去看看Java语法。
- 依赖不为空（省去了我们对其检查）：当要实例化FooController的时候，由于自己实现了有参数的构造函数，所以不会调用默认构造函数，那么就需要Spring容器传入所需要的参数，所以就两种情况：1、有该类型的参数->传入，OK 。2：无该类型的参数->报错。所以保证不会为空，Spring总不至于传一个null进去吧
- 完全初始化的状态：这个可以跟上面的依赖不为空结合起来，向构造器传参之前，要确保注入的内容不为空，那么肯定要调用依赖组件的构造方法完成实例化。而在Java类加载实例化的过程中，构造方法是最后一步（之前如果有父类先初始化父类，然后自己的成员变量，最后才是构造方法，这里不详细展开。）。所以返回来的都是初始化之后的状态。



### 4、Spring 开发团队的建议

> Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for mandatory dependencies and setter methods or configuration methods for optional dependencies.

简单来说，就是

- 强制依赖就用构造器方式

- 可选、可变的依赖就用setter 注入

  当然你可以在同一个类中使用这两种方法。构造器注入更适合强制性的注入旨在不变性，Setter注入更适合可变性的注入。

让我们看看Spring 这样推荐的理由，首先是基于构造方法注入，

> The Spring team generally advocates constructor injection as it enables one to implement application components as immutable objects and to ensure that required dependencies are not null. Furthermore constructor-injected components are always returned to client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.

Spring 团队提倡使用基于构造方法的注入，因为这样一方面可以**将依赖注入到一个不可变的变量中（注：final 修饰的变量）**，另一方面也可以**保证这些变量的值不会是 null**。此外，经过构造方法完成依赖注入的组件 （注：比如各个Service），在被调用时可以**保证它们都完全准备好了**。与此同时，从代码质量的角度来看，**一个巨大的构造方法通常代表着出现了代码异味，这个类可能承担了过多的责任**。

而对于基于 setter 的注入，他们是这么说的：

> Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later.

基于 setter 的注入，则只应该被用于注入非必需的依赖，同时在类中应该对这个依赖提供一个合理的默认值。如果使用 setter 注入必需的依赖，那么将会有过多的 null 检查充斥在代码中。**使用 setter 注入的一个优点是，这个依赖可以很方便的被改变或者重新注入**。



## 六、使用 @RequiredArgsConstructor 构造注入

通过上述学习，构造器的方法就成了我们的首选。代码如下：

```java
@Service
public class GoodsServiceImpl implements GoodsSrv {
    private GoodsRepo goodsRepo;
    private TagRepo tagRepo;
    private TagRefRepo tagRefRepo;
    private BrandRepo brandRepo;
    private UnitRepo unitRepo;

    public GoodsServiceImpl(
            GoodsRepo goodsRepo,
            TagRepo tagRepo,
            TagRefRepo tagRefRepo,
            BrandRepo brandRepo,
            UnitRepo unitRepo) {
        this.goodsRepo = goodsRepo;
        this.tagRefRepo = tagRefRepo;
        this.tagRefRepo = tagRefRepo;
        this.brandRepo = brandRepo;
        this.unitRepo = unitRepo;
        this.tagRepo = tagRepo;
    }
}
```

Spring 不需要加入其他注解，就可以使用构造器完成注入。问题是，我们依然要写很多代码。这个时候，你可能想到了Lombok的#AllArgsConstructor注解。但它是针对于全部的属性的，如果类中有一些非Bean的属性，Spring就会晕菜。这个时候，就可以使用@RequiredArgsConstructor了。代码如下：

```java
@Service
@RequiredArgsConstructor
public class GoodsServiceImpl implements GoodsSrv {
    final GoodsRepo goodsRepo;
    final TagRepo tagRepo;
    final TagRefRepo tagRefRepo;
    final BrandRepo brandRepo;
    final UnitRepo unitRepo;
}
```

@RequiredArgsConstructor 需要变量是final，非final的会忽略，我们把需要注入的属性，修改成final类型的（或者使用@NotNull注解，此不推荐），这些属性将构成默认的构造器。Java要求final类型的属性必须要初始化，如果没有构造方法代码就会变红。

***

@RequiredArgsConstructor 源码看到有onConstructor参数。

试试看看什么效果：@RequiredArgsConstructor 与 @RequiredArgsConstructor(onConstructor_={@Autowired})

1、先查看@RequiredArgsConstructor的效果

```java
public interface TestService {
    void test();
}

@Service
public class TestServiceImpl implements TestService{
    @Override
    public void test() {
    }
}

@RestController
@RequiredArgsConstructor
public class TestController {
    private String enable;
    private final TestService testService;

    @GetMapping(value = "/save")
    public String get() {
        testService.test();
        return enable;
    }
}
```

查看编译后TestController的class

```java
@RestController
public class TestController {
    private String enable;
    private final TestService testService;

    @GetMapping({"/save"})
    public String get() {
        this.testService.test();
        return this.enable;
    }

    public TestController(final TestService testService) {
        this.testService = testService;
    }
}
```

2、再查看@RequiredArgsConstructor(onConstructor_={@Autowired})的效果

```java
public interface TestService {
    void test();
}

@Service
public class TestServiceImpl implements TestService{
    @Override
    public void test() {
    }
}

@RestController
@RequiredArgsConstructor
public class TestController {
    private String enable;
    private final TestService testService;

    @GetMapping(value = "/save")
    public String get() {
        testService.test();
        return enable;
    }
}
```

查看编译后TestController的class

```java
@RestController
public class TestController {
    private String enable;
    private final TestService testService;

    @GetMapping({"/save"})
    public String get() {
        this.testService.test();
        return this.enable;
    }

    @Autowired
    public TestController(final TestService testService) {
        this.testService = testService;
    }
}
```

> 在 Spring 4.3 及以后的版本中，如果这个类只有一个构造方法，那么这个构造方法上面也可以不写 @Autowired 注解。所以可以不用这个属性
>
> **为什么加上构造器，不用@Autowired也能把对象注入Spring容器？**
>
> 1. 在Java语言中，每个类至少有一个构造方法。如果程序中没有显式定义任何构造方法，那么将自动提供一个隐含的默认无参构造方法。
> 2. 只要程序中已经显式定义了构造方法，那么将不再提供隐含的默认构造方法。
> 3. 所以这里定义了一个带参数的构造器，在初始化这个类的时候需要传入一个对象，这个Spring容器就把它创建了，所有内部不用加@Autowired



## 七、Spring 依赖注入总结 

Spring 使用构造器注入的好处：

1. 保证依赖不可变（final关键字）
2. 保证依赖不为空（省去了我们对其检查）
3. 保证返回客户端（调用）的代码的时候是完全初始化的状态
4. 避免了循环依赖
5. 提升了代码的可复用性

另外，当有一个依赖有多个实现的使用，推荐使用field注入或者setter注入的方式来指定注入的类型。



## 八、参考文献 & 鸣谢

- [Setter-based dependency injection](https://link.juejin.cn?target=https%3A%2F%2Fdocs.spring.io%2Fspring%2Fdocs%2F4.2.x%2Fspring-framework-reference%2Fhtml%2Fbeans.html%23beans-setter-injection)
- [Field Dependency Injection Considered Harmful](https://link.juejin.cn?target=https%3A%2F%2Fwww.vojtechruzicka.com%2Ffield-dependency-injection-considered-harmful%2F)
- [IDEA 警告 Field injection is not recommended](https://link.juejin.cn?target=https%3A%2F%2Fboris1993.github.io%2Fprojects%2Fjava%2Fcoding-tips%2Fidea-warninig-field-injection-is-not-recommended.html)