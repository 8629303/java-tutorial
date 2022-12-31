

# 一、SpringBoot 启动流程及原理（自动配置）

> SpringBoot 启动过程:
>
> 1. 作者：南宫煌_慧；来源：https://www.cnblogs.com/nangonghui/p/15663125.html
> 2. 作者：涛姐涛哥；来源：https://www.cnblogs.com/taojietaoge/p/16075820.html
> 3. 作者：Fly丶X；来源：https://blog.csdn.net/weixin_44947701/article/details/124055713
> 4. 作者：业余草；来源：https://mp.weixin.qq.com/s/viDJc489EjuCPLPWNAnIzA

SpringBoot 的启动经过了一系列的处理（装配、加载、监听...），我们先看看整体过程的流程图：

```
          ┌───────────────────────────────────────┐
          │           SpringBoot启动过程           │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │  1、运行 SpringApplication.run() 方法  │
          └───────────────────────────────────────┘
                            ⬇     
          ┌───────────────────────────────────────┐
          │          2、判断应用项目的类型          │ »» new SpringApplication() «« 开始
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │        3、初始化所有根引导启动器        │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │         4、初始化所有容器上下文         │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │          5、初始化所有的监听器          │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │          6、设置程序运行的主类          │ »» new SpringApplication() «« 结束
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │             7、开启计时器              │ »» springApplication.run() «« 开始
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │ 8、创建启动上下文对象即bootstrapContext │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │        9、开启headless无头模式         │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │         10、获取并启动所有监听器        │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │         11、设置默认应用程序参数        │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │            12、准备环境变量            │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │          13、配置忽略Bean信息          │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │            14、打印Banner信息          │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │       15、创建IOC容器并设置启动器       │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │         16、准备IOC容器基本信息         │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │             17、刷新上下文             │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │          18、刷新上下文后置处理         │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │         19、结束计时器并打印日志        │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │         20、发布监听应用启动事件        │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │        21、执行自定义的run()方法        │
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │        22、执行监听器的running()       │ »» springApplication.run() «« 结束
          └───────────────────────────────────────┘
                            ⬇
          ┌───────────────────────────────────────┐
          │           SpringBoot启动完成           │
          └───────────────────────────────────────┘
```

SpringBoot 启动流程分析使用版本SpringBoot VERSION：版本 **2.5.0**。



## 1、XxApplication.main() 方法

SpringBoot项目最简单的Application启动类。所有的标准的SpringBoot的应用程序都是从run方法开始的

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        // 启动SpringBoot的入口
        SpringApplication.run(TestApplication.class, args);
    }
}
```

可以看出Application启动类中，包含了@SpringBootApplication 注解和 SpringApplication.run() 启动方法，所以SpringBoot的启动可以分解为 **注解** 和 **启动方法** 两大过程，而仔细看启动类中还引入了一个【org.springframework.boot.SpringApplication】包，所以启动方法中又可以分为两个阶段即 创建 **SpringApplication实例** 和 **执行run方法**。

简单总结一下 SpringBoot 启动主要分为两个阶段：

1. 构造SpringApplection的实例
2. 调用实例的run方法



## 2、构造SpringApplication实例

从启动类中的run方法跟进去：SpringApplication.run » return run » return new SpringApplication(primarySources).run(args) » this(null, primarySources) » SpringApplication 构造方法

```java
    /**
     * Create a new {@link SpringApplication} instance. The application context will load
     * beans from the specified primary sources (see {@link SpringApplication class-level}
     * documentation for details. The instance can be customized before calling
     * {@link #run(String...)}.
     * @param resourceLoader the resource loader to use
     * @param primarySources the primary bean sources
     * @see #run(Class, String[])
     * @see #setSources(Set)
     */
    @SuppressWarnings({ "unchecked", "rawtypes" })
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        // 0.初始化类加载器
        this.resourceLoader = resourceLoader;
        // Assert 断言非空，若传入的class参数为null则打印异常并退出初始化
        Assert.notNull(primarySources, "PrimarySources must not be null");
        // 0.获取main方法中的args，初始化启动时配置的额外参数集合
        this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        // 1.判断项目启动类型：NONE/SERVLET/REACTIVE (默认是Servlet容器)
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
        // 2.设置Bootstrapper: 从Spring工厂获取 BootstrapRegistryInitializers
        this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
        // 3.设置所有初始化器: 获取Spring工厂实例 -> 容器上下文相关的初始化
        setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        // 4.设置所有监听器: 获取Spring工厂实例 -> 设置应用程序监听器
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        // 5.推导出主应用程序类，即从当前的栈信息中寻找main所在主类：com.iot.SpringBootLoveApplication
        this.mainApplicationClass = deduceMainApplicationClass();
	}
```

总结一下构造方法都做了什么：

1. 把参数sources设置到SpringApplection属性中，这个sources可以是任何类型的参数

2. 推断Web应用类型（WebApplicationType），本例中是SERVLET

3. 设置 Bootstrapper、ApplicationContextInitializer 和 ApplicationListener

4. 初始化主类mainApplectionClass，定位main方法

5. 注意到：在设置 Bootstrapper、ApplicationContextInitializer 和 ApplicationListener 时，都调用了同一个方法getSpringFactoriesInstances，从所有 META-INF/spring.factories 文件中加载，并查找指定名称的的值

   回到之前的构造方法那里，现在我们知道设置的 Bootstrapper、ApplicationContextInitializer、ApplicationListener 是从哪儿来的了，原来是在META-INF/spring.factories中配置的。查找指定key对应的值，然后实例化它们，并返回一个List。

总结一句话概括：构造函数的功能主要是为成员变量赋值，包括首要类、确定web应用类型，从资源文件中加载的类资源，为注册初始类、监听器类、上下文初始化类赋值，确定主类



### 2.1、判断应用项目类型

在SpringApplication的构造方法内，首先会通过 WebApplicationType.deduceFromClasspath()； 方法判断当前应用程序的容器，默认使用的是Servlet 容器，除了Servlet之外，还有NONE 和 REACTIVE （响应式编程）。WebApplicationType.java 枚举类方法如下：

```java
/**
 * deduceFromClasspath
 * 依次循环遍历当前应用中是否存在相关的类来判断最终应用的启动类型
 */
static WebApplicationType deduceFromClasspath() {
    /**
     * REACTIVE：响应式WEB项目
     * 若启动类型为REACTIVE,
     * 则类路径下存在 org.springframework.web.reactive.DispatcherHandler 类
     * 并且不存在 org.springframework.web.servlet.DispatcherServlet 和 org.glassfish.jersey.servlet.ServletContainer
     * 两者指的是SpringMVC/Tomcat和jersey容器
     */
    if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
            && !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
        return WebApplicationType.REACTIVE;
    }
    /**
     * NONE：非WEB项目,就是一个最简单的Springboot应用
     * 若启动类型为NONE
     * 则类路径下 javax.servlet.Servlet 和org.springframework.web.context.ConfigurableWebApplicationContext都不存在
     */
    for (String className : SERVLET_INDICATOR_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            return WebApplicationType.NONE;
        }
    }
    /**
     * SERVLET：SERVLET WEB 项目
     * 若启动类型为Servlet，则必须有SERVLET_INDICATOR_CLASSES中的javax.servlet.Servlet
     * 和org.springframework.web.context.ConfigurableWebApplicationContext
     */
    return WebApplicationType.SERVLET;
}
```



### 2.2、初始化所有根引导启动器

> 初始化根引导程序：Bootstrapper

getBootstrapRegistryInitializersFromSpringFactories() 方法从spring.factories 中获取 BootstrapRegistryInitializer

```java
// 2.设置Bootstrapper: 从Spring工厂获取 BootstrapRegistryInitializers
this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
```

```java
private List<BootstrapRegistryInitializer> getBootstrapRegistryInitializersFromSpringFactories() {
    ArrayList<BootstrapRegistryInitializer> initializers = new ArrayList<>();
    /**
     * 从spring.factories 中获取Bootstrapper集合，
     * 然后遍历转化为BootstrapRegistryInitializer，再存入 initializers
     */
    getSpringFactoriesInstances(Bootstrapper.class).stream()
            .map((bootstrapper) -> ((BootstrapRegistryInitializer) bootstrapper::initialize))
            .forEach(initializers::add);
    /**
     * 从spring.factories 中获取BootstrapRegistryInitializer集合，再存入 initializers
     * getSpringFactoriesInstances 该方法在整个启动流程中会频繁出现,下面集中介绍
     */
    initializers.addAll(getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
    return initializers;
}
```

设定初始引导启动器，保存在List中：从classpath的spring.factories文件中去找org.springframework.boot.Bootstrapper（在此项目中，没有找到）



### 2.3、初始化所有容器上下文

setInitializers 是容器上下文初始化，容器上下文初始化调用了getSpringFactoriesInstances() 方法，从spring.factories中获取配置

```java
// 3.设置所有初始化器: 获取Spring工厂实例 -> 容器上下文相关的初始化
setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
```

```java
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    ClassLoader classLoaderToUse = classLoader;
    if (classLoaderToUse == null) {
        classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
    }
    String factoryTypeName = factoryType.getName();
    return loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
}


/**
 * Springboot自动配置的秘密
 * Springboot在启动时读取了所有starter jar包里的META-INF/spring.factories配置文件，实现了所谓的自动化配置
 * 这里jar包里的都是默认配置，后续Springboot也会从xml、yaml文件中的用户配置去覆盖同名的配置。
 * 另外，这里的缓存配置是保存在一个map类型的cache中，其中的key键对应上面提到的各种Type类型，value就是Type的各种初始jar包里的同类型Java类。
 */
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
    // 获取相应类加载器中内容
    Map<String, List<String>> result = cache.get(classLoader);
    // 存在则直接返回类加载器中内容
    if (result != null) {
        return result;
    }
    // 不存在则初始化类加载器中内容
    result = new HashMap<>();
    try {
        /**
         * 获取资源 -> META-INF/spring.factories 列表
         * 其中FACTORIES_RESOURCE_LOCATION的值：META-INF/spring.factories
         */
        Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);
        // 可能存在多个META-INF/spring.factories 文件,循环加载
        while (urls.hasMoreElements()) {
            // 获取 META-INF/spring.factories 文件URL地址
            URL url = urls.nextElement();
            // 加载资源
            UrlResource resource = new UrlResource(url);
            // 加载资源配置
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            // key:value形式循环配置
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryTypeName = ((String) entry.getKey()).trim();
                // 逗号分隔列表到字符串数组
                String[] factoryImplementationNames =
                        StringUtils.commaDelimitedListToStringArray((String) entry.getValue());
                // 循环value中子项到列表中
                for (String factoryImplementationName : factoryImplementationNames) {
                    result.computeIfAbsent(factoryTypeName, key -> new ArrayList<>())
                            .add(factoryImplementationName.trim());
                }
            }
        }

        // Replace all lists with unmodifiable lists containing unique elements
        // 列表去重
        result.replaceAll((factoryType, implementations) -> implementations.stream().distinct()
                .collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList)));
        // 列表保存
        cache.put(classLoader, result);
    } catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
                FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
    return result;
}


/**
 * 反射创建实现类
 */
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
                                                   ClassLoader classLoader, Object[] args, Set<String> names) {
    List<T> instances = new ArrayList<>(names.size());
    for (String name : names) {
        try {
            Class<?> instanceClass = ClassUtils.forName(name, classLoader);
            Assert.isAssignable(type, instanceClass);
            Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
            T instance = (T) BeanUtils.instantiateClass(constructor, args);
            instances.add(instance);
        } catch (Throwable ex) {
            throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
        }
    }
    return instances;
}
```

```java
// 调用链如下：
org.springframework.boot.SpringApplication.getSpringFactoriesInstances() ->
    org.springframework.core.io.support.SpringFactoriesLoader.loadFactoryNames() ->
        org.springframework.core.io.support.SpringFactoriesLoader.loadSpringFactories() ->
            org.springframework.boot.SpringApplication.createSpringFactoriesInstances();
```



### 2.4、初始化所有监听器

初始化监听器与初始化容器上下文及引导启动器基本一样。调用getSpringFactoriesInstances() 方法，从spring.factories中获取配置

```java
// 4.设置所有监听器: 获取Spring工厂实例 -> 设置应用程序监听器
setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
```

getBootstrapRegistryInitializersFromSpringFactories && setInitializers && setListeners 分别是：**根引导程序初始化** & **容器上下文初始化** & **监听器初始化**。根引导程序初始化、容器上下文初始化 和 监听器初始化 都是调用了getSpringFactoriesInstances() 方法，从spring.factories中获取配置。不同的是传给它的type参数，主要有一下几种类型。

- ApplicationContextInitializer.class：上下文相关
- ApplicationListener.class：监听器相关
- SpringApplicationRunListener.class：运行时监听器
- SpringBootExceptionReporter.class：异常类相关
- Bootstrapper.class：根引导程序

***

***

> 模拟初始化自定义类：

在初始化 Bootstrapper、ApplicationContextInitializer 和 ApplicationListener 时，都调用了同一个方法getSpringFactoriesInstances，从所有 META-INF/spring.factories 文件中加载，并查找指定名称的的值，那么这个文件在哪呢？SpringBoot 自己定义的都放在源码的jar包的 spring-boot-autoconfigure 项目里，而其他第三方start都定义在自己的 xxx-spring-boot-autoconfigure 项目里。实际上我们也可以定义在自己的SpringBoot项目中的 classpath 路径下（只是这样不太规范）

下面我们做个示例根据配置文件（META-INF/spring.factories），创建ApplicationContextInitializer.class & ApplicationListener.class两个接口的实现类：

1、自定义初始化器：实现 ApplicationContextInitializer 接口

```java
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;

/**
 * 自定义的初始化器
 */
public class MyApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        System.out.println("我是初始化的 MyApplicationContextInitializer...");
    }
}
```

2、自定义监听器：实现 ApplicationListener 接口

```java
import org.springframework.boot.context.event.ApplicationStartedEvent;
import org.springframework.context.ApplicationListener;

public class MyApplicationListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("我是初始化的 MyApplicationListener...");
    }
}
```

3、在resources目录下添加 META-INF/spring.factories 配置文件，内容如下，将自定义的初始化器和监听器注册进去；

```properties
org.springframework.context.ApplicationContextInitializer=\
com.example.test.config.MyApplicationContextInitializer

org.springframework.context.ApplicationListener=\
com.example.test.config.MyApplicationListener
```

4、输出结果

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)
 
我是初始化的 MyApplicationContextInitializer...
### 省略了中间的日志 ###
我是初始化的 MyApplicationListener...
```



### 2.3、设置程序运行的主类

deduceMainApplicationClass() 这个方法仅仅是找到main方法所在的类，为后面的扫包作准备，deduce是推断的意思，所以准确地说，这个方法作用是推断出主方法所在的类（其实遍历当前虚拟机栈获取main方法所在的类并且返回）：

```java
// 5.推导出主应用程序类，即从当前的栈信息中寻找main所在主类：com.iot.SpringBootLoveApplication
this.mainApplicationClass = deduceMainApplicationClass();
```

```java
/**
 * 推导主应用程序类
 */
private Class<?> deduceMainApplicationClass() {
    try {
        // 获取当前的栈信息
        StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
        for (StackTraceElement stackTraceElement : stackTrace) {
            // 获取main方法所在的类class，此处即com.iot.SpringBootLoveApplication
            if ("main".equals(stackTraceElement.getMethodName())) {
                return Class.forName(stackTraceElement.getClassName());
            }
        }
    } catch (ClassNotFoundException ex) {
        // Swallow and continue
    }
    return null;
}
```



## 3、执行实例的 run() 方法

接下来，重头戏来了，调用run()方法（重点看try里面的内容）

1. 首先是启动监听器来监听启动过程，这里再一次调用了熟悉的getSpringFactoriesInstances
2. 然后将main方法中传的参数包装成ApplicationArguments对象
3. 紧接着，创建并配置环境，返回一个ConfigurableEnvironment对象，由于本例中是SERVLET，所以创建的是StandardServletEnvironment
4. 然后打印Banner
5. 创建ApplicationContext，所有基于Spring的项目启动过程都是创建ApplicationContext，这里创建的ConfigurableApplicationContext是AnnotationConfigServletWebServerApplicationContext
6. 有了ApplicationContext，紧接着要配置ApplicationContext
7. 加载所有的source（其实就是class），然后实例化
8. 这里就是一个，就是启动类
9. 接下来，刷新容器
10. 最终是调用Spring的applicationContext.refresh()，又看到了熟悉的一幕
11. 最后还留了一个入口，回调子容器的onRefresh()
12. Spring MVC的启动是创建父子两个WebApplicationContext
13. 在刷新容器之后，所有的Bean都已经实例化完成
14. afterRefresh什么也没做
15. 最后是回调Runner，回调所有的ApplicationRunner和CommandLineRunner
16. 至此，Spring Boot启动完成

回顾一下：

1. 推断Web应用类型
2. 设置Bootstrapper、ApplicationContextInitializer、ApplicationListener
3. 推断main方法
4. 创建并配置环境Environment
5. 打印Banner
6. 创建并配置ApplicationContext
7. 刷新ApplicationContext（此处交由Spring处理，调用Spring的refresh()方法）实例化所有的bean
8. 回调所有的ApplicationRunner和CommandLineRunner

> 初始化完SpringApplication对象就可以运行他的run方法了，也就是启动方法中的第二阶段。

### 3.01、View Code

首先查看run方法的代码，返回值为：ConfigurableApplicationContext

```java
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 *
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 */
public ConfigurableApplicationContext run(String... args) {
    // 启动一个秒表计时器，用于统计项目启动时间
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // 创建启动上下文对象即Spring根容器
    DefaultBootstrapContext bootstrapContext = createBootstrapContext();
    // 定义可配置的应用程序上下文变量
    ConfigurableApplicationContext context = null;
    /**
     * 设置JDK系统属性
     * headless直译就是无头模式，
     * headless模式的意思是明确Springboot要在无鼠键支持的环境中运行，一般也都跑在Linux上，无鼠键支持，默认值true
     */
    configureHeadlessProperty();
    /**
     * 获取运行监听器 getRunListeners, 其中也是调用了上面说到的getSpringFactoriesInstances 方法
     * 从spring.factories中获取配置
     */
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 启动监听器
    listeners.starting(bootstrapContext, this.mainApplicationClass);
    try {
        // 包装默认应用程序参数，也就是在命令行下启动应用带的参数，如--server.port=9000
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        /**
         * 准备环境 prepareEnvironment 是个硬茬，里面主要涉及到
         * getOrCreateEnvironment、configureEnvironment、configurePropertySources、configureProfiles
         * environmentPrepared、bindToSpringApplication、attach诸多方法可以在下面的例子中查看
         */
        ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        // 配置忽略的 bean
        configureIgnoreBeanInfo(environment);
        // 打印 SpringBoot 标志，即启动的时候在控制台的图案logo，可以在src/main/resources下放入名字是banner的自定义文件
        Banner printedBanner = printBanner(environment);
        // 创建 IOC 容器
        context = createApplicationContext();
        // 设置一个启动器，设置应用程序启动
        context.setApplicationStartup(this.applicationStartup);
        // 配置 IOC 容器的基本信息 (spring容器前置处理)
        prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        /**
         * 刷新IOC容器
         * 这里会涉及Spring容器启动、自动装配、创建 WebServer启动Web服务即SpringBoot启动内嵌的 Tomcat
         */
        refreshContext(context);
        /**
         * 留给用户自定义容器刷新完成后的处理逻辑
         * 刷新容器后的扩展接口(spring容器后置处理)
         */
        afterRefresh(context, applicationArguments);
        // 结束计时器并打印，这就是我们启动后console的显示的时间
        stopWatch.stop();
        if (this.logStartupInfo) {
            // 打印启动完毕的那行日志
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        // 发布监听应用上下文启动完成（发出启动结束事件），所有的运行监听器调用 started() 方法
        listeners.started(context);
        // 执行runner，遍历所有的 runner，调用 run 方法
        callRunners(context, applicationArguments);
    } catch (Throwable ex) {
        // 异常处理，如果run过程发生异常
        handleRunFailure(context, ex, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        // 所有的运行监听器调用 running() 方法,监听应用上下文
        listeners.running(context);
    } catch (Throwable ex) {
        // 异常处理
        handleRunFailure(context, ex, null);
        throw new IllegalStateException(ex);
    }
    // 返回最终构建的容器对象
    return context;
}
```



### 3.02、开启计时器

程序运行到这里，就已经进入了run方法的主体了，第一步调用的run方法是静态方法，那个时候还没实例化SpringApplication对象，现在调用的run方法是非静态的，是需要实例化后才可以调用的，进来后首先会开启计时器，这个计时器有什么作用呢？顾名思义就使用来计时的嘛，计算SpringBoot启动花了多长时间；关键代码如下：

```java
// 实例化计时器，然后开始计时
StopWatch stopWatch = new StopWatch();
stopWatch.start();
```



### 3.03、createBootstrapContext

调用createBootstrapContext方法创建引导上下文（即：创建启动上下文对象即Spring根容器）：对之前保存的bootstrappers挨个遍历执行intitialize方法来完成对引导启动器上下文环境的设置。

```java
public class SpringApplication {
    private DefaultBootstrapContext createBootstrapContext() {
        DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
        // 此时的bootstrapRegistryInitializers为空，没有任何处理
        this.bootstrapRegistryInitializers.forEach((initializer) -> initializer.initialize(bootstrapContext));
        return bootstrapContext;
    }
}
```

```java
/**
 * 一个简单的引导上下文，在启动和环境后处理期间可用，直到准备好ApplicationContext。
 */
public class DefaultBootstrapContext implements ConfigurableBootstrapContext {
}
```



### 3.04、configureHeadlessProperty

configureHeadlessProperty 设置当前应用进入headless无头模式（java.awt.headless）

```java
private static final String SYSTEM_PROPERTY_JAVA_AWT_HEADLESS = "java.awt.headless";
/**
 * headless直译就是无头模式，
 * headless模式的意思是明确Springboot要在无鼠键支持的环境中运行，一般也都跑在Linux上，无鼠键支持，默认值true
 */
private void configureHeadlessProperty() {
    // SYSTEM_PROPERTY_JAVA_AWT_HEADLESS = "java.awt.headless";
    System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS,
            System.getProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
}
```

通过方法可以看到，setProperty()方法里面又有个getProperty()；这不多此一举吗？其实getProperty()方法里面有2个参数， 第一个key值，第二个是默认值，意思是通过key值查找属性值，如果属性值为空，则返回默认值 true；保证了一定有值的情况；



### 3.05、获取并启用监听器

获取所有RunListener运行监听器：从spring.factories中找SpringApplicationRunListener.class，遍历所有的RunListener，调用starting方法。相当于通知所有感兴趣系统正在启动的人，项目正在starting，这一部分涉及到自定义SpringBoot启动过程。

通过监听器来实现初始化的的基本操作：1.创建所有 Spring 运行监听器并发布应用启动事件。2.启用监听器

```java
/**
 * 获取运行监听器 getRunListeners, 其中也是调用了上面说到的getSpringFactoriesInstances方法
 * 从spring.factories中获取配置, 启动监听器
 */
SpringApplicationRunListeners listeners = getRunListeners(args);
listeners.starting(bootstrapContext,this.mainApplicationClass);
```



### 3.06、设置应用程序参数

将执行run方法时传入的参数封装成一个对象，**保存命令行参数**，保存到ApplicationArguments实例中

```java
// 包装默认应用程序参数，也就是在命令行下启动应用带的参数，如--server.port=9000
ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
```

```java
public DefaultApplicationArguments(String... args) {
    Assert.notNull(args, "Args must not be null");
    this.source = new Source(args);
    this.args = args;
}
```



### 3.07、准备加载环境变量

执行prepareEnvironment方法准备环境变量，包含系统属性和用户配置的属性。

```java
/**
 * 准备环境 prepareEnvironment 是个硬茬，里面主要涉及到
 * getOrCreateEnvironment、configureEnvironment、configurePropertySources、configureProfiles
 * environmentPrepared、bindToSpringApplication、attach诸多方法可以在下面的例子中查看
 */
ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
```

prepareEnvironment方法里面主要涉及到：getOrCreateEnvironment、configureEnvironment、configurePropertySources、configureProfilesenvironmentPrepared、bindToSpringApplication、attach诸多方法。

- getOrCreateEnvironment：获取ApplicationServletEnvironment，主要从类路径下面加载配置文件资源，包括加载激活文件等
- configureEnvironment：配置属性源和添加命令行参数；
- ConfigurationPropertySources.attach：添加附件配置属性
- listeners.environmentPrepared：发布环境已经准备好事件通知
- DefaultPropertiesPropertySource.moveToEnd：将默认配置文件属性移至最后
- bindToSpringApplication：将环境信息绑定至Spring应用

```java
/**
 * 准备环境
 *
 * @param listeners
 * @param bootstrapContext
 * @param applicationArguments
 * @return
 */
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
                                                   DefaultBootstrapContext bootstrapContext,
                                                   ApplicationArguments applicationArguments) {
    // Create and configure the environment 创建和配置环境
    // 根据项目类型建环境ConfigurableEnvironment
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    // 从环境中获取并设置 PropertySources 和 activeProfiles
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    // 把 PropertySources 设置在自己PropertySources的第一个位置
    ConfigurationPropertySources.attach(environment);
    /**
     * 运行监听器调用
     * 广播事件，listeners环境准备(就是广播ApplicationEnvironmentPreparedEvent事件)
     * 发布事件通知所有的监听器当前环境准备完成
     */
    listeners.environmentPrepared(bootstrapContext, environment);
    // 移动 defaultProperties 属性源到环境中的最后一个源
    DefaultPropertiesPropertySource.moveToEnd(environment);
    // 断言 抛异常
    Assert.state(!environment.containsProperty("spring.main.environment-prefix"),
            "Environment prefix cannot be set via properties.");
    // 与容器绑定当前环境
    bindToSpringApplication(environment);
    // 若非web环境，将环境转换成StandardEnvironment
    if (!this.isCustomEnvironment) {
        environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
                deduceEnvironmentClass());
    }
    // 配置PropertySources对它自己的递归依赖
    ConfigurationPropertySources.attach(environment);
    return environment;
}


/**
 * 获取或创建环境Environment
 *
 * @return
 */
private ConfigurableEnvironment getOrCreateEnvironment() {
    // 存在则直接返回
    if (this.environment != null) {
        return this.environment;
    }
    /**
     * 根据webApplicationType创建对应的Environment
     */
    switch (this.webApplicationType) {
        // SERVLET WEB 项目
        case SERVLET:
            return new ApplicationServletEnvironment();
        // REACTIVE：响应式WEB项目
        case REACTIVE:
            return new ApplicationReactiveWebEnvironment();
        // 非WEB项目,就是一个最简单的Springboot应用
        default:
            return new ApplicationEnvironment();
    }
}

/**
 * 从环境中获取并设置 PropertySources 和 activeProfiles
 * 将配置任务按顺序委托给configurePropertySources和configureProfiles
 * Template method delegating to
 * {@link #configurePropertySources(ConfigurableEnvironment, String[])} and
 * {@link #configureProfiles(ConfigurableEnvironment, String[])} in that order.
 * Override this method for complete control over Environment customization, or one of
 * the above for fine-grained control over property sources or profiles, respectively.
 *
 * @param environment this application's environment
 * @param args        arguments passed to the {@code run} method
 * @see #configureProfiles(ConfigurableEnvironment, String[])
 * @see #configurePropertySources(ConfigurableEnvironment, String[])
 */
protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
    if (this.addConversionService) {
        environment.setConversionService(new ApplicationConversionService());
    }
    // 配置PropertySources
    configurePropertySources(environment, args);
    // 配置Profiles
    configureProfiles(environment, args);
}

/**
 * 配置PropertySources
 * Add, remove or re-order any {@link PropertySource}s in this application's
 * environment.
 *
 * @param environment this application's environment
 * @param args        arguments passed to the {@code run} method
 * @see #configureEnvironment(ConfigurableEnvironment, String[])
 */
protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
    MutablePropertySources sources = environment.getPropertySources();
    // 初始化 defaultProperties
    if (!CollectionUtils.isEmpty(this.defaultProperties)) {
        // 存在的话将其放到最后位置
        DefaultPropertiesPropertySource.addOrMerge(this.defaultProperties, sources);
    }
    /**
     * 存在命令行参数，则解析它并封装进SimpleCommandLinePropertySource对象
     * 同时将此对象放到sources的第一位置（优先级最高）
     */
    if (this.addCommandLineProperties && args.length > 0) {
        String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
        if (sources.contains(name)) {
            PropertySource<?> source = sources.get(name);
            CompositePropertySource composite = new CompositePropertySource(name);
            composite.addPropertySource(
                    new SimpleCommandLinePropertySource("springApplicationCommandLineArgs", args));
            composite.addPropertySource(source);
            sources.replace(name, composite);
        } else {
            // 放到首位
            sources.addFirst(new SimpleCommandLinePropertySource(args));
        }
    }
}

/**
 * 配置Profiles
 *
 * @param environment
 * @param args
 */
protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
    /**
     * 保证environment的activeProfiles属性被初始化了。从PropertySources中查找spring.profiles.active属性
     * 存在则将其值添加activeProfiles集合中。
     * 配置应用环境中的哪些配置文件处于激活状态(或默认激活)
     * 可以通过spring.profiles.active属性在配置文件处理期间激活其他配置文件
     * 就是我们项目中通常配置的dev、sit、prod等环境配置信息设置哪些Profiles是激活的。
     */
    environment.getActiveProfiles(); // ensure they are initialized
    // But these ones should go first (last wins in a property key clash)
    // 如果存在其他的Profiles，则将这些Profiles放到第一的位置
    Set<String> profiles = new LinkedHashSet<>(this.additionalProfiles);
    profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
    environment.setActiveProfiles(StringUtils.toStringArray(profiles));
}

/**
 * 运行监听器调用
 *
 * @param bootstrapContext
 * @param environment
 */
void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
    doWithListeners("spring.boot.application.environment-prepared",
            (listener) -> listener.environmentPrepared(bootstrapContext, environment));
}

/**
 * 运行监听器调用
 * Called once the environment has been prepared, but before the
 * {@link ApplicationContext} has been created.
 *
 * @param environment the environment
 * @deprecated since 2.4.0 for removal in 2.6.0 in favor of
 * {@link #environmentPrepared(ConfigurableBootstrapContext, ConfigurableEnvironment)}
 */
@Deprecated
default void environmentPrepared(ConfigurableEnvironment environment) {
    for (SpringApplicationRunListener listener : this.listeners) {
        // 广播ApplicationEnvironmentPreparedEvent事件,后面再看
        listener.environmentPrepared(environment);
    }
}

/**
 * 与容器绑定当前环境
 * Bind the environment to the {@link SpringApplication}.
 *
 * @param environment the environment to bind
 */
protected void bindToSpringApplication(ConfigurableEnvironment environment) {
    try {
        // 将environment绑定到SpringApplication
        Binder.get(environment).bind("spring.main", Bindable.ofInstance(this));
    } catch (Exception ex) {
        throw new IllegalStateException("Cannot bind to SpringApplication", ex);
    }
}

/**
 * 配置PropertySources对它自己的递归依赖
 * Attach a {@link ConfigurationPropertySource} support to the specified
 * {@link Environment}. Adapts each {@link PropertySource} managed by the environment
 * to a {@link ConfigurationPropertySource} and allows classic
 * {@link PropertySourcesPropertyResolver} calls to resolve using
 * {@link ConfigurationPropertyName configuration property names}.
 * <p>
 * The attached resolver will dynamically track any additions or removals from the
 * underlying {@link Environment} property sources.
 *
 * @param environment the source environment (must be an instance of
 *                    {@link ConfigurableEnvironment})
 * @see #get(Environment)
 */
public static void attach(Environment environment) {
    // 判断environment是否是ConfigurableEnvironment的实例
    Assert.isInstanceOf(ConfigurableEnvironment.class, environment);
    // 从environment获取PropertySources
    MutablePropertySources sources = ((ConfigurableEnvironment) environment)
            .getPropertySources();
    PropertySource<?> attached = sources.get(ATTACHED_PROPERTY_SOURCE_NAME);
    if (attached != null && attached.getSource() != sources) {
        sources.remove(ATTACHED_PROPERTY_SOURCE_NAME);
        attached = null;
    }
    if (attached == null) {
        // 将sources封装成ConfigurationPropertySourcesPropertySource对象，并把这个对象放到sources的第一位置
        sources.addFirst(new ConfigurationPropertySourcesPropertySource(
                ATTACHED_PROPERTY_SOURCE_NAME,
                new SpringConfigurationPropertySources(sources)));
    }
}
```

调用getOrCreateEnvironment() 方法得到环境对象：如果环境信息已经存在，直接返回使用；如果环境信息没有，则创建环境信息对象，不同的应用创建不同的环境信息：

1. SERVLET => StandardServletEnvironment
2. REACTIVE => StandardReactiveWebEnvironment
3. NONE or Default => StandardEnvironment



### 3.08、配置忽略Bean信息

configureIgnoreBeanInfo() 这个方法是将 spring.beaninfo.ignore 的默认值值设为 true，意思是跳过beanInfo的搜索，其设置默认值的原理和configureHeadlessProperty设置headless无头模式一样；

```java
// 配置忽略的 bean
configureIgnoreBeanInfo(environment);
```

```java
private void configureIgnoreBeanInfo(ConfigurableEnvironment environment) {
    if (System.getProperty(
            CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME) == null) {
        Boolean ignore = environment.getProperty("spring.beaninfo.ignore",
                Boolean.class, Boolean.TRUE);
        System.setProperty(CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME,
                ignore.toString());
    }
}
```

如上是设置环境变量来配置，当然也可以在配置文件中添加以下配置来设为false。目前还不知道这个配置的具体作用。

```properties
spring.beaninfo.ignore=false
```



### 3.09、打印Banner信息

printBanner 打印SpringBoot标志。printBanner(environment)方法就是打印Banner，Banner就是项目启动时看到的那个logo，默认标准信息在SpringBootBanner.java类中。如果想自定义Banner可以在工程项目src/main/resources路径下下放入名字是banner的文件，后缀后可以是SpringApplicationBannerPrinter.java类里的{ "gif", "jpg", "png" }，或者是txt、图片也可以的，但是图片打印时会字符化，而不是打印图片本身。[自定义banner链接](https://www.bootschool.net/ascii)

```java
// 打印 SpringBoot 标志，即启动的时候在控制台的图案logo，可以在src/main/resources下放入名字是banner的自定义文件
Banner printedBanner = printBanner(environment);
```

```java
/**
 * 打印SpringBoot标志
 * banner的输出默认有三种种模式，LOG、CONSOLE、OFF。
 * 1. LOG：将banner信息输出到日志文件。
 * 2. CONSOLE：将banner信息输出到控制台。
 * 3. OFF：禁用banner的信息输出。
 *
 * @param environment
 * @return
 */
private Banner printBanner(ConfigurableEnvironment environment) {
    // 判断Banner的模式是否关闭，如果关闭直接返回。
    if (this.bannerMode == Banner.Mode.OFF) {
        return null;
    }
    ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
            : new DefaultResourceLoader(null);
    // 创建SpringApplicationBannerPrinter 打印类
    SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
    // LOG：将banner信息输出到日志文件
    if (this.bannerMode == Mode.LOG) {
        return bannerPrinter.print(environment, this.mainApplicationClass, logger);
    }
    //banner没有关闭且没有指定是写到log文件中 将banner信息输出到控制台
    return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}

/**
 * 打印
 *
 * @param environment
 * @param sourceClass
 * @param logger
 * @return
 */
Banner print(Environment environment, Class<?> sourceClass, Log logger) {
    // 获取banner内容
    Banner banner = getBanner(environment);
    try {
        logger.info(createStringFromBanner(banner, environment, sourceClass));
    } catch (UnsupportedEncodingException ex) {
        logger.warn("Failed to create String for banner", ex);
    }
    return new PrintedBanner(banner, sourceClass);
}

/**
 * 获取banner内容
 *
 * @param environment
 * @return
 */
private Banner getBanner(Environment environment) {
    Banners banners = new Banners();
    // 图片类型的banner内容
    banners.addIfNotNull(getImageBanner(environment));
    // 文本类型的banner内容
    banners.addIfNotNull(getTextBanner(environment));
    if (banners.hasAtLeastOneBanner()) {
        return banners;
    }
    if (this.fallbackBanner != null) {
        return this.fallbackBanner;
    }
    return DEFAULT_BANNER;
}

static final String BANNER_LOCATION_PROPERTY = "spring.banner.location";
static final String DEFAULT_BANNER_LOCATION = "banner.txt";

/**
 * 文本类型的banner内容获取
 *
 * @param environment
 * @return
 */
private Banner getTextBanner(Environment environment) {
    /**
     * 拿到自定义配置的banner文件地址
     * BANNER_LOCATION_PROPERTY = "spring.banner.location"
     * DEFAULT_BANNER_LOCATION = "banner.txt";
     */
    String location = environment.getProperty(BANNER_LOCATION_PROPERTY, DEFAULT_BANNER_LOCATION);
    Resource resource = this.resourceLoader.getResource(location);
    try {
        if (resource.exists() && !resource.getURL().toExternalForm().contains("liquibase-core")) {
            return new ResourceBanner(resource);
        }
    } catch (IOException ex) {
        // Ignore
    }
    return null;
}
```



### 3.10、创建IOC容器并设置启动器

createApplicationContext 创建IOC容器（应用程序上下文）。并设置应用启动器。

```java
// 创建 IOC 容器
context = createApplicationContext();
// 设置一个启动器，设置应用程序启动
context.setApplicationStartup(this.applicationStartup);
```

1.使用applicationContextFactory上下文工厂创建一个应用上下文，其中applicationContextFactory是一个成员变量

```java
public class SpringApplication {
    private ApplicationContextFactory applicationContextFactory = ApplicationContextFactory.DEFAULT;
    protected ConfigurableApplicationContext createApplicationContext() {
        // 进入default的lambda表达式，返回AnnotationConfigServletWebServerApplicationContext
        return this.applicationContextFactory.create(this.webApplicationType);
    }
}
```

2.ApplicationContextFactory根据webApplicationType返回应用上下文，返回AnnotationConfigServletWebServerApplicationContext

```java
@FunctionalInterface
public interface ApplicationContextFactory {
    /**
     * 创建 IOC 容器
     * A default {@link ApplicationContextFactory} implementation that will create an
     * appropriate context for the {@link WebApplicationType}.
     */
    ApplicationContextFactory DEFAULT = (webApplicationType) -> {
        try {
            // 根据当前应用的类型创建 IOC 容器
            switch (webApplicationType) {
                // Web 应用环境对应 AnnotationConfigServletWebServerApplicationContext
                case SERVLET:
                    return new AnnotationConfigServletWebServerApplicationContext();
                // 响应式编程对应 AnnotationConfigReactiveWebServerApplicationContext
                case REACTIVE:
                    return new AnnotationConfigReactiveWebServerApplicationContext();
                // 默认为 Spring 环境 AnnotationConfigApplicationContext
                default:
                    return new AnnotationConfigApplicationContext();
            }
        } catch (Exception ex) {
            throw new IllegalStateException("Unable create a default ApplicationContext instance, "
                    + "you may need a custom ApplicationContextFactory", ex);
        }
    };
    ConfigurableApplicationContext create(WebApplicationType webApplicationType);
}
```



### 3.11、准备IOC容器基本信息

prepareContext 配置 IOC 容器的基本信息（准备应用程序上下文环境）

```java
// 配置 IOC 容器的基本信息 (Spring容器前置处理)
prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
```

虽然已经得到了ApplicationContext对象，但此时的对象还只是一个空白对象，需要准备和处理后，ApplicationContext才能被使用。在准备过程中主要做了做了几件事：

1. 为ApplicationContext设置之前准备好的Environment对象
2. 通过对ApplicationContext后置处理或是BeanDefinitionLoader等方式往容器中添加一些初始的Bean
3. 应用默认的初始化器初始化应用程序上下文（责任链模式的应用，多个初始化器形成一个List，应用程序需要被每个初始化器应用一次，每个初始化器有自己的职责）
4. 准备过程中ApplicationRunListener发出两个消息，分别是contextPrepared和contextLoaded

```java
/**
 * 准备IOC容器基本信息
 *
 * @param bootstrapContext
 * @param context
 * @param environment
 * @param listeners
 * @param applicationArguments
 * @param printedBanner
 */
private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context,
                            ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
                            ApplicationArguments applicationArguments, Banner printedBanner) {
    // 设置容器环境，包括各种变量
    context.setEnvironment(environment);
    // 后置处理流程，设置IOC容器的 bean 生成器和资源加载器
    postProcessApplicationContext(context);
    /**
     * 获取所有的初始化器调用 initialize() 方法进行初始化
     * 执行容器中的ApplicationContextInitializer（包括从 spring.factories和自定义的实例）初始化
     */
    applyInitializers(context);
    /**
     * 触发所有 SpringApplicationRunListener 监听器的 contextPrepared 事件方法
     * 所有的运行监听器调用 environmentPrepared() 方法，EventPublishingRunListener 发布事件通知 IOC 容器准备完成
     */
    listeners.contextPrepared(context);
    bootstrapContext.close(context);
    // 打印启动日志
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }
    // Add boot specific singleton beans
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    // 注册添加特定的单例bean
    beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
    if (printedBanner != null) {
        beanFactory.registerSingleton("springBootBanner", printedBanner);
    }
    if (beanFactory instanceof DefaultListableBeanFactory) {
        ((DefaultListableBeanFactory) beanFactory)
                .setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    }
    if (this.lazyInitialization) {
        context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
    }
    // 加载所有资源 (Load the sources)
    Set<Object> sources = getAllSources();
    // 断言资源费控
    Assert.notEmpty(sources, "Sources must not be empty");
    // 创建BeanDefinitionLoader，加载启动类，将启动类注入容器
    load(context, sources.toArray(new Object[0]));
    // 触发所有 SpringApplicationRunListener 监听器的 contextLoaded 事件方法
    listeners.contextLoaded(context);
}
```



### 3.12、刷新上下文

refresh 刷新应用上下文，即刷新Spring上下文信息refreshContext。这里会涉及Spring容器启动（创建出容器中的所有组件）、SpringBoot 自动装配（starter 原理）、创建 WebServer启动Web服务即SpringBoot启动内嵌的 Tomcat。还有其他的Spring自带的机制在这里就不一一细说了。

```java
public class SpringApplication {
    /**
     * 刷新应用上下文
     *
     * @param context
     */
    private void refreshContext(ConfigurableApplicationContext context) {
        if (this.registerShutdownHook) {
            // 判断是否注册关闭的钩子，是则注册钩子
            shutdownHook.registerApplicationContext(context);
        }
        refresh(context);
    }

    /**
     * Refresh the underlying {@link ApplicationContext}.
     *
     * @param applicationContext the application context to refresh
     */
    protected void refresh(ConfigurableApplicationContext applicationContext) {
        applicationContext.refresh();
    }
}

/******************************************分割线******************************************/

public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {
    /**
     * 刷新IOC容器
     *
     * @throws BeansException
     * @throws IllegalStateException
     */
    @Override
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

            // Prepare this context for refreshing. 准备刷新上下文
            prepareRefresh();

            // Tell the subclass to refresh the internal bean factory. 通知子类刷新内部工厂
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // Prepare the bean factory for use in this context. 准备Bean工厂
            prepareBeanFactory(beanFactory);

            try {
                // Allows post-processing of the bean factory in context subclasses.
                // 允许在上下文子类中对bean工厂进行后处理,这部分涉及Web服务器的启动，如servlet
                postProcessBeanFactory(beanFactory);

                StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
                // Invoke factory processors registered as beans in the context.
                // 调用在上下文中注册为 bean 的工厂处理器
                invokeBeanFactoryPostProcessors(beanFactory);

                // Register bean processors that intercept bean creation. 注册拦截 bean 创建的 bean 处理器
                registerBeanPostProcessors(beanFactory);
                beanPostProcess.end();

                // Initialize message source for this context. 初始化此上下文的消息源
                initMessageSource();

                // Initialize event multicaster for this context. 为该上下文初始化事件多播器
                initApplicationEventMulticaster();

                // Initialize other special beans in specific context subclasses. 初始化特定上下文子类中的其他特殊 bean
                /**
                 * SpringBoot 一键启动web工程的关键方法
                 * 创建 WebServer启动Web服务
                 * SpringBoot启动内嵌的 Tomcat 首先要在pom文件配置内嵌容器为tomcat
                 * SpringBoot 嵌入式 Servlet 容器，默认支持的 webServe：Tomcat、Jetty、Undertow
                 *          <exclusion>
                 *             <groupId>org.springframework.boot</groupId>
                 *             <artifactId>spring-boot-starter-tomcat</artifactId>
                 *         </exclusion>
                 */
                onRefresh();

                // Check for listener beans and register them. 检查侦听器 bean 并注册
                registerListeners();

                // Instantiate all remaining (non-lazy-init) singletons. 实例化所有剩余的（非延迟初始化）单例
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event. 发布事件
                finishRefresh();
            } catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // Destroy already created singletons to avoid dangling resources.  销毁bean
                destroyBeans();

                // Reset 'active' flag.
                cancelRefresh(ex);

                // Propagate exception to caller.
                throw ex;
            } finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
                resetCommonCaches();
                contextRefresh.end();
            }
        }
    }
}
```

**onRefresh** 方法中创建WebServer、创建Tomcat对象，是SpringBoot一键启动web工程的关键。SpringBoot 嵌入式 Servlet 容器，默认支持的 webServe：Tomcat、Jetty、Undertow，但要在POM文件加入tomcat相关配置。

```java
/**
 * 创建 WebServer启动Web服务
 */
@Override
protected void onRefresh() {
    // 初始化给定应用程序上下文的主题资源
    super.onRefresh();
    try {
        // 创建Web 服务
        createWebServer();
    } catch (Throwable ex) {
        throw new ApplicationContextException("Unable to start web server", ex);
    }
}

/**
 * super.onRefresh();
 * Initialize the theme capability.
 */
@Override
protected void onRefresh() {
    /**
     * 初始化给定应用程序上下文的主题资源，自动检测一个名为“themeSource”的bean。
     * 如果没有这样的，将使用默认的(空的)ThemeSource。
     */
    this.themeSource = UiApplicationContextUtils.initThemeSource(this);
}

/**
 * 创建Web 服务
 */
private void createWebServer() {
    WebServer webServer = this.webServer;
    ServletContext servletContext = getServletContext();
    if (webServer == null && servletContext == null) {
        // 获取web server
        StartupStep createWebServer = this.getApplicationStartup().start("spring.boot.webserver.create");
        // 获取创建容器的工厂
        ServletWebServerFactory factory = getWebServerFactory();
        createWebServer.tag("factory", factory.getClass().toString());
        /**
         * 获取 tomcat 、Jetty 或 Undertow 容器
         * 从 getWebServer 方法点进去，找到 TomcatServletWebServerFactory 的实现方法，
         * 与之对应的还有 Jetty 和 Undertow。这里配置了基本的连接器、引擎、虚拟站点等配置。
         * 自动配置类 ServletWebServerFactoryAutoConfiguration 导入了 ServletWebServerFactoryConfiguration（配置类）,
         * 根据条件装配判断系统中到底导入了哪个 Web 服务器的包，创建出服务器并启动
         * 默认是 web-starter 导入 tomcat 包，容器中就有 TomcatServletWebServerFactory，创建出 Tomcat 服务器并启动
         */
        this.webServer = factory.getWebServer(getSelfInitializer());
        createWebServer.end();
        getBeanFactory().registerSingleton("webServerGracefulShutdown",
                new WebServerGracefulShutdownLifecycle(this.webServer));
        getBeanFactory().registerSingleton("webServerStartStop",
                new WebServerStartStopLifecycle(this, this.webServer));
    } else if (servletContext != null) {
        try {
            // 启动web server
            getSelfInitializer().onStartup(servletContext);
        } catch (ServletException ex) {
            throw new ApplicationContextException("Cannot initialize servlet context", ex);
        }
    }
    initPropertySources();
}

/**
 * 获取tomcat 容器
 * 配置了基本的连接器、引擎、虚拟站点等配置
 *
 * @param initializers
 * @return
 */
@Override
public WebServer getWebServer(ServletContextInitializer... initializers) {
    if (this.disableMBeanRegistry) {
        Registry.disableRegistry();
    }
    /**
     * 创建了Tomcat对象，并设置参数
     */
    Tomcat tomcat = new Tomcat();
    // 设置工作忙碌
    File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
    tomcat.setBaseDir(baseDir.getAbsolutePath());
    // 初始化tomcat 连接，默认NIO
    Connector connector = new Connector(this.protocol);
    connector.setThrowOnFailure(true);
    tomcat.getService().addConnector(connector);
    customizeConnector(connector);
    // 配置基本的连接器、引擎、虚拟站点
    tomcat.setConnector(connector);
    // 设置自动部署为false
    tomcat.getHost().setAutoDeploy(false);
    configureEngine(tomcat.getEngine());
    for (Connector additionalConnector : this.additionalTomcatConnectors) {
        tomcat.getService().addConnector(additionalConnector);
    }
    // 准备上下文
    prepareContext(tomcat.getHost(), initializers);
    // 返回TomcatWebServer服务
    return getTomcatWebServer(tomcat);
}

/**
 * Create a new {@link TomcatWebServer} instance.
 *
 * @param tomcat    the underlying Tomcat server
 * @param autoStart if the server should be started
 * @param shutdown  type of shutdown supported by the server
 * @since 2.3.0
 */
public TomcatWebServer(Tomcat tomcat, boolean autoStart, Shutdown shutdown) {
    Assert.notNull(tomcat, "Tomcat Server must not be null");
    this.tomcat = tomcat;
    this.autoStart = autoStart;
    this.gracefulShutdown = (shutdown == Shutdown.GRACEFUL) ? new GracefulShutdown(tomcat) : null;
    // 初始化Tomcat
    initialize();
}

/**
 * 初始化Tomcat
 *
 * @throws WebServerException
 */
private void initialize() throws WebServerException {
    logger.info("Tomcat initialized with port(s): " + getPortsDescription(false));
    synchronized (this.monitor) {
        try {
            addInstanceIdToEngineName();

            Context context = findContext();
            context.addLifecycleListener((event) -> {
                if (context.equals(event.getSource()) && Lifecycle.START_EVENT.equals(event.getType())) {
                    // Remove service connectors so that protocol binding doesn't
                    // happen when the service is started.
                    removeServiceConnectors();
                }
            });

            // Start the server to trigger initialization listeners
            this.tomcat.start();

            // We can re-throw failure exception directly in the main thread
            rethrowDeferredStartupExceptions();

            try {
                ContextBindings.bindClassLoader(context, context.getNamingToken(), getClass().getClassLoader());
            } catch (NamingException ex) {
                // Naming is not enabled. Continue
            }

            // Unlike Jetty, all Tomcat threads are daemon threads. We create a
            // blocking non-daemon to stop immediate shutdown
            startDaemonAwaitThread();
        } catch (Exception ex) {
            stopSilently();
            destroySilently();
            throw new WebServerException("Unable to start embedded Tomcat", ex);
        }
    }
}
```



### 3.13、刷新上下文后置处理

afterRefresh(context, applicationArguments) 方法是启动后的一些处理，是一个空实现，留给用户扩展使。

```java
/**
 * Called after the context has been refreshed.
 * @param context the application context
 * @param args the application arguments
  */
protected void afterRefresh(ConfigurableApplicationContext context, ApplicationArguments args) {
}
```



### 3.14、结束计时器并打印日志

```java
// 结束计时器并打印，这就是我们启动后控制台的显示的时间
stopWatch.stop();
if (this.logStartupInfo) {
    // 打印启动完毕的那行日志
    new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
}
```



### 3.15、发布监听应用启动事件

```java
// 通知所有监听器，可以started开始工作了
// 发布监听应用上下文启动完成（发出启动结束事件），所有的运行监听器调用 started() 方法
listeners.started(context);
```



### 3.16、执行自定义的run方法

这是一个扩展功能，callRunners(context, applicationArguments) 可以在启动完成后执行自定义的run方法；有2中方式可以实现：

1. 实现 ApplicationRunner 接口
2. 实现 CommandLineRunner 接口

```java
// 执行runner，遍历所有的 runner，调用 run 方法
callRunners(context, applicationArguments);
```

接下来我们验证一把，为了方便代码可读性，我把这2种方式都放在同一个类里面

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class MyRunner implements ApplicationRunner, CommandLineRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("我是自定义的run方法1，实现 ApplicationRunner 接口既可运行");
    }
    
    @Override
    public void run(String... args) throws Exception {
        System.out.println("我是自定义的run方法2，实现 CommandLineRunner 接口既可运行");
    }
}
```

> 如果以上步骤出现了异常，SpringBoot会执行`listeners.failed(context, exception);`，通知所有监听器应用failed



### 3.17、执行监听器的running()

通知所有监听器应用正在running，running 发布上下文完成准备事件，listeners.running() 发布上下文完成准备事件同前面的listeners.started() 方法一样，都是发布了一个running事件，代码也相同。

```java
/**
 * 发布上下文完成准备事件
 * 与上面的 listeners.started() 方法一样
 *
 * @param context
 */
void running(ConfigurableApplicationContext context) {
    // listener.started(context) 中交由context.publishEvent（）方法处理
    // 实际上是发送了一个ApplicationStartedEvent的事件
    doWithListeners("spring.boot.application.running", (listener) -> listener.running(context));
}

/**
 * 发布上下文完成准备事件
 * Called immediately before the run method finishes, when the application context has
 * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
 * {@link ApplicationRunner ApplicationRunners} have been called.
 *
 * @param context the application context.
 * @since 2.0.0
 */
@Override
public void running(ConfigurableApplicationContext context) {
    context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context));
    AvailabilityChangeEvent.publish(context, ReadinessState.ACCEPTING_TRAFFIC);
}
```

> 如果running出现异常，则继续调用`listeners.failed(context, exception);`

这也是SpringBoot启动流程两大过程中的第二阶段的启动方法run中最后一个方法了，该方法执行完成后，SpringApplication的run(String... args)方法执行结束，至此Spring Boot的ApplicationContext 启动结束。SpringBoot启动流程比较复杂，期间涉及到许多的监听器和事件监听机制，需要深入学习



## 4、SpringBoot启动流程总结

SpringBoot启动流程总结就是下面代码块，一个创建SpringApplication实例，一个执行run方法，所有的猫腻都在其中。

1、创建SpringApplication实例

```java
@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    // 0.初始化类加载器
    this.resourceLoader = resourceLoader;
    // Assert 断言非空，若传入的class参数为null则打印异常并退出初始化
    Assert.notNull(primarySources, "PrimarySources must not be null");
    // 0.获取main方法中的args，初始化启动时配置的额外参数集合
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 1.判断项目启动类型：NONE/SERVLET/REACTIVE (默认是Servlet容器)
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 2.设置Bootstrapper: 从Spring工厂获取 BootstrapRegistryInitializers
    this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
    // 3.设置所有初始化器: 获取Spring工厂实例 -> 容器上下文相关的初始化
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    // 4.设置所有监听器: 获取Spring工厂实例 -> 设置应用程序监听器
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 5.推导出主应用程序类，即从当前的栈信息中寻找main所在主类：com.iot.SpringBootLoveApplication
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

2、执行实例的run()方法

```java
public ConfigurableApplicationContext run(String... args) {
    // 启动一个秒表计时器，用于统计项目启动时间
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // 创建启动上下文对象即Spring根容器
    DefaultBootstrapContext bootstrapContext = createBootstrapContext();
    // 定义可配置的应用程序上下文变量
    ConfigurableApplicationContext context = null;
    // headless直译就是无头模式，无鼠键支持
    configureHeadlessProperty();
    // 获取运行监听器 getRunListeners, 从spring.factories中获取配置, 并启动监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting(bootstrapContext, this.mainApplicationClass);
    try {
        // 包装默认应用程序参数，也就是在命令行下启动应用带的参数，如--server.port=9000
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 准备环境 prepareEnvironment
        ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        // 配置忽略的 bean
        configureIgnoreBeanInfo(environment);
        // 打印 SpringBoot 标志，即启动的时候在控制台的图案logo，可以在src/main/resources下放入名字是banner的自定义文件
        Banner printedBanner = printBanner(environment);
        // 创建 IOC 容器
        context = createApplicationContext();
        // 设置一个启动器，设置应用程序启动
        context.setApplicationStartup(this.applicationStartup);
        // 配置 IOC 容器的基本信息 (spring容器前置处理)
        prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        // 刷新IOC容器，会涉及Spring容器启动、自动装配、创建 WebServer启动Web服务即SpringBoot启动内嵌的 Tomcat
        refreshContext(context);
        // 留给用户自定义容器刷新完成后的处理逻辑, 刷新容器后的扩展接口(spring容器后置处理)
        afterRefresh(context, applicationArguments);
        // 结束计时器并打印，这就是我们启动后console的显示的时间
        stopWatch.stop();
        if (this.logStartupInfo) {
            // 打印启动完毕的那行日志
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        // 发布监听应用上下文启动完成（发出启动结束事件），所有的运行监听器调用 started() 方法
        listeners.started(context);
        // 执行runner，遍历所有的 runner，调用 run 方法
        callRunners(context, applicationArguments);
    } catch (Throwable ex) {
        // 异常处理，如果run过程发生异常
        handleRunFailure(context, ex, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        // 所有的运行监听器调用 running() 方法,监听应用上下文
        listeners.running(context);
    } catch (Throwable ex) {
        // 异常处理
        handleRunFailure(context, ex, null);
        throw new IllegalStateException(ex);
    }
    // 返回最终构建的容器对象
    return context;
}
```





# 二、SpringBoot 启动类上的注解（自动配置）

> 作者：Jimoer；来源：https://mp.weixin.qq.com/s/ibJnLu4vplGadWcWBqdPpQ

上面说到在SpringApplication的run()方法中，通过调用自己的prepareContext()方法，在prepareContext()方法中又调用getAllSources()方法，然后去获取启动类，然后通过SpringApplication的load()方法，去加载启动类，然后在刷新容器的时候就会去将启动类在容器中进行实例化。在刷新ApplicationContext容器时，就开始解析启动类上的注解了。

启动类XxoApplication就只有一个注解@SpringBootApplication，那么下面来看一下这个注解：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

可以看到这个注解是一个复合注解，有三个关键注解需要说明一下。



## 1、@SpringBootConfiguration

@SpringBootConfiguration这个注解说明再点进去查看详情发现就是一个@Configuration注解，这说明启动类就是一个配置类。支持Spring以JavaConfig的形式启动。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
public @interface SpringBootConfiguration {
}
```



## 2、@ComponentScan

这个注解，从字面的意思上也能看出来，就是组件扫描的意思，即默认扫描当前package以及其子包下面的Spring的注解，例如：@Controller、@Service、@Component等等注解。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
}
```



## 3、@EnableAutoConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```

@EnableAutoConfiguration这个注解也是一个复合注解：这个注解是比较核心的一个注解，SpringBoot的主要自动配置原理基本上都来自@EnableAutoConfiguration这个注解的配置，那么我们通过看这个注解的源码可以发现有两个注解比较重要的。

- 一个是@AutoConfigurationPackage，自动配置包。
- 另一个是@Import(AutoConfigurationImportSelector.class)，自动引入组件。



## 4、@AutoConfigurationPackage

@AutoConfigurationPackage这个注解字面的意思是**自动配置包**，那么我们点进去看看里面是什么样的。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
}
```

还是一个复合注解，但是最终依赖的确实@Import这个注解，这个注解后面我们会介绍，现在先明白它就是给Spring容器引入组件的功能的一个注解。

那么我们接着来看看AutoConfigurationPackages.Registrar.class这个类里面的代码。

```java
/**
 * {@link ImportBeanDefinitionRegistrar} to store the base package from the importing
 * configuration.
 */
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
    }

    @Override
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new PackageImports(metadata));
    }
}
```

```java
public static void register(BeanDefinitionRegistry registry, String... packageNames) {
    if (registry.containsBeanDefinition(BEAN)) {
        BasePackagesBeanDefinition beanDefinition = (BasePackagesBeanDefinition) registry.getBeanDefinition(BEAN);
        beanDefinition.addBasePackages(packageNames);
    }
    else {
        registry.registerBeanDefinition(BEAN, new BasePackagesBeanDefinition(packageNames));
    }
}
```

这两段代码就是这个AutoConfigurationPackages.Registrar这个类的关键部分，说实话，我是没看出来什么东西。但是网上搜到的是这个register()方法的作用是，用来自动注册一些组件中的配置，例如JPA的@Entity这个注解，这里就是会开启自动扫描这类注解的功能。



## 5、@Import(AutoConfigurationImportSelector.class)

我们接着回来看@EnableAutoConfiguration下的@Import(AutoConfigurationImportSelector.class)这个注解的功能。进入到AutoConfigurationImportSelector这个类里面后源码如下：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        // 关键代码
        AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

然后我们进入getAutoConfigurationEntry()方法来看看：

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 关键代码
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

我们继续进入getCandidateConfigurations()方法：

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    // 关键代码
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                                                                         getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

看来最核心的方法是SpringFactroiesLoader.loadFactoryNames()方法了，我们再进入看看：

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    ClassLoader classLoaderToUse = classLoader;
    if (classLoaderToUse == null) {
        classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
    }
    String factoryTypeName = factoryType.getName();
    // 关键代码
    return loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
}
```

包的好深，居然还有一层，那么继续进入loadSpringFactories()方法。

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
    Map<String, List<String>> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }

    result = new HashMap<>();
    try {
        /** 关键代码1 **/
        Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            UrlResource resource = new UrlResource(url);
            /** 关键代码2 **/
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryTypeName = ((String) entry.getKey()).trim();
                String[] factoryImplementationNames =
                    StringUtils.commaDelimitedListToStringArray((String) entry.getValue());
                for (String factoryImplementationName : factoryImplementationNames) {
                    /** 关键代码3 **/
                    result.computeIfAbsent(factoryTypeName, key -> new ArrayList<>())
                        .add(factoryImplementationName.trim());
                }
            }
        }

        // Replace all lists with unmodifiable lists containing unique elements
        result.replaceAll((factoryType, implementations) -> implementations.stream().distinct()
                          .collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList)));
        /** 关键代码4 **/
        cache.put(classLoader, result);
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
                                           FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
    return result;
}
```

> 注意：上面代码标注了四处关键代码

终于到最后一层了，算是“拨开云雾见天日,守得云开见月明”，下面就来梳理一下loadSpringFactories()方法。

首先**FACTORIES_RESOURCE_LOCATION**这个常量的值是："**META-INF/spring.factories**"

```java
/**
 * The location to look for factories.
 * <p>Can be present in multiple JAR files.
 */
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```

所以第一个关键核心代码的意思是：启动的时候会扫描所有jar包下**META-INF/spring.factories**这个文件。第二段代码的意思是将这些扫描到的文件转成Properties对象，后面两个核心代码的意思就是说将加载到的Properties对象放入到缓存中。

然后**getCandidateConfigurations()**方法，是只获取了key是**EnableAutoConfiguration.class**的配置。

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                                                                         getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}

// 上面的方法调用了当前方法
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;
}
```

我们可以打断点看到getCandidateConfigurations()方法，通过SpringFactoriesLoader.loadFactoryNames()获取到了131个配置（注意：不同SpringBoot版本看到的值是不一样的，本人使用的是SpringBoot-2.5.0版本）。请查看如下代码注意打上断点DEBUG测试查看值：

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 断点打在如下一行，然后查看List值
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

那么我们来看一个spring.factories文件中的内容是什么样子的呢？请查看spring-boot-autoconfigure.jar包下的文件：

```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\

### 省略...
```

原来是这种形式的，看来这和上一篇文章中讲解的Java中的SPI机制加载接口实现很像啊，其实通过查阅资料发现，这就是一种自定义SPI的实现方式的功能。

那么我们以第一个配置类：org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration来看一下，这些类都是如果实现的。打开org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration的源码：

```java
@Configuration(proxyBeanMethods = false)
@AutoConfigureAfter(JmxAutoConfiguration.class)
/** 关键代码 **/
@ConditionalOnProperty(prefix = "spring.application.admin", value = "enabled", havingValue = "true",
		matchIfMissing = false)
public class SpringApplicationAdminJmxAutoConfiguration {
	private static final String JMX_NAME_PROPERTY = "spring.application.admin.jmx-name";
	private static final String DEFAULT_JMX_NAME = "org.springframework.boot:type=Admin,name=SpringApplication";

	@Bean
	@ConditionalOnMissingBean
	public SpringApplicationAdminMXBeanRegistrar springApplicationAdminRegistrar(
			ObjectProvider<MBeanExporter> mbeanExporters, Environment environment) throws MalformedObjectNameException {
		String jmxName = environment.getProperty(JMX_NAME_PROPERTY, DEFAULT_JMX_NAME);
		if (mbeanExporters != null) { // Make sure to not register that MBean twice
			for (MBeanExporter mbeanExporter : mbeanExporters) {
				mbeanExporter.addExcludedBean(jmxName);
			}
		}
		return new SpringApplicationAdminMXBeanRegistrar(jmxName);
	}
}
```

我们看到这个类有三个注解@Configuration、@AutoConfigureAfter、@ConditionalOnProperty、因为有@Configuration注解所以它也是一个配置类，然后第二注解中的参数类JmxAutoConfiguration.class进入之后是这样的：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ MBeanExporter.class })
@ConditionalOnProperty(prefix = "spring.jmx", name = "enabled", havingValue = "true")
public class JmxAutoConfiguration {
}
```

也是存在@ConditionalOnProperty注解的。那看来关键点就是@ConditionalOnProperty这个注解了。这个注解其实是一个条件判断注解，这个条件注解后面的参数的意思是当存在系统属性前缀为**spring.application.admin**，并且属性名称为**enabled**，并且值为**true**时，才加载当前这个Bean并进行实例化。



## 6、@Conditional 常用条件注解

这种Spring4.0后面出现的的条件注解，可以极大的增加了框架的灵活性和扩展性，可以保证很多组件可以通过后期配置，而且阅读源码的人，通过这些注解就能明白在什么情况下才会实例化当前Bean。后面还有不少这种条件注解呢：

| 名称                            | 描述                                                     |
| ------------------------------- | -------------------------------------------------------- |
| @ConditionalOnBean              | 当容器里有指定Bean的条件下才生效                         |
| @ConditionalOnMissingBean       | 当容器里没有指定Bean的情况下才生效                       |
| @ConditionalOnClass             | 当类路径下有指定的类的条件下才生效                       |
| @ConditionalOnMissingClass      | 当容器里没有指定类的情况下才生效                         |
| @ConditionalOnProperty          | 指定的属性是否有指定的值才生效                           |
| @ConditionalOnWebApplication    | 当前项目时Web项目的条件下才生效                          |
| @ConditionalOnNotWebApplication | 当前项目不是Web项目的条件下才生效                        |
| @ConditionalOnExpression        | 基于SpEL表达式为true的时候作为判断条件才去实例化         |
| @ConditionalOnJava              | 基于JVM版本作为判断条件                                  |
| @ConditionalOnJndi              | 在JNDI存在的条件下查找指定的位置                         |
| @ConditionalOnResource          | 类路径是否有指定的值                                     |
| @ConditionalOnOnSingleCandidate | 当指定Bean在容器中只有一个，或者有多个但是指定首选的Bean |

这些注解其实都是通过@Conditional注解扩展而来的，只是使用了不同的组合条件来判断是否需要加载和初始化当前Bean。



## 7、SpringBoot 自动装配原理总结（面试）

这就是SpringBoot 自动配置的过程：

SpringBoot 启动时，是依靠启动类的main方法来进行启动的，而main方法中执行的是SpringApplication.run()方法，而SpringApplication.run()方法中会创建Spring的容器，并且刷新容器。而在刷新容器的时候就会去解析启动类，然后就会去解析启动类上的@SpringBootApplication注解，而这个注解是个复合注解，这个注解中有一个@EnableAutoConfiguration注解，这个注解就是开启自动配置，这个注解中又有@Import注解引入了一个AutoConfigurationImportSelector这个类，这个类会进过一些核心方法，然后去扫描我们所有jar包下的META-INF下的spring.factories文件，而从这个配置文件中取找key为EnableAutoConfiguration类的全路径的值下面的所有配置都加载，这些配置里面都是有条件注解的，然后这些条件注解会根据你当前的项目依赖的jar包以及是否配置了符合这些条件注解的配置来进行装载的。

***

**其实上面这些内容还是有点多，而且还有好多注解的单词也不好记，那换成大白话，再精炼一下：**

> SpringBoot在启动的时候会调用run()方法，run()方法会刷新容器，刷新容器的时候，会扫描classpath下面的的包中**META-INF/spring.factories**文件，在这个文件中记录了好多的自动配置类，在刷新容器的时候会将这些自动配置类加载到容器中，然后在根据这些配置类中的条件注解，来判断是否将这些配置类在容器中进行实例化，这些条件主要是判断项目是否有相关jar包或是否引入了相关的bean。这样SpringBoot就帮助我们完成了自动装配。



# 三、Spring-Boot-Starter 自定义及原理

> 1. 作者：业余草；来源：https://mp.weixin.qq.com/s/fcCz1V2asnLGmdYGo-uILg
> 2. 作者：Java笔记虾；来源：https://mp.weixin.qq.com/s/42HzC1AFbDb7tOWb_tdezQ
> 3. 作者：苏三说技术；来源：https://mp.weixin.qq.com/s/_dCZpoFGzkSSLXdz9rzI8g

## 1、Starter 场景启动器前言

### 1、什么是 Starter

Starter是Spring Boot中的一个非常重要的概念，Starter相当于模块，它能将模块所需的依赖整合起来并对模块内的Bean根据环境（ 条件）进行自动配置。

**「使用者只需要依赖相应功能的Starter，无需做过多的配置和依赖，Spring Boot就能自动扫描并加载相应的模块并设置默认值，做到开箱即用」**



### 2、为什么要用 Starter 

在SpringBoot还没有出来之前，我们使用Spring开发项目。如果程序需要连接数据库，我们一般会使用Hibernate或Mybatis等ORM框架，这里我以Mybatis为例，具体的操作步骤如下：

1. 到maven仓库去找需要引入的mybatis jar包，选取合适的版本
2. 到maven仓库去找mybatis-spring整合的jar包，选取合适的版本
3. 根据具体使用的数据库引入数据库驱动包（mysql、oracle、postgresql）
4. 在spring的applicationContext.xml文件中配置dataSource和mybatis相关信息

如上只是配置好了连接数据库这一个功能，后面还需要配置SpringMVC，需要引入更多的功能，比如：连接redis、连接mongodb、使用rocketmq、使用excel功能等等。引入这些功能的话，需要再把上面的步骤再重复一次，工作量无形当中增加了不少，而且有很多重复的工作。另外，还是有个问题，每次到要到maven中找合适的版本，如果哪次找的mybatis.jar包 和 mybatis-spring.jar包版本不兼容，程序不是会出现问题？

这就是使用Spring框架开发项目带来的一些的问题：

- **依赖导入问题：** 每个项目都需要来单独维护自己所依赖的jar包，在项目中使用到什么功能就需要引入什么样的依赖。手动导入依赖容易出错，且无法统一集中管理
- **配置繁琐：** 在引入依赖之后需要做繁杂的配置，并且这些配置是每个项目来说都是必要的，例如web.xml配置（Listener配置、Filter配置、Servlet配置）、log4j配置、数据库连接池配置等等。这些配置重复且繁杂，在不同的项目中需要进行多次重复开发，这在很大程度上降低了我们的开发效率

SpringBoot出现之后，它为我们提供了一个强大的功能来解决上述的两个痛点，这就是SpringBoot的starters机制（场景启动器）。

SpringBoot通过将我们常用的功能场景抽取出来，做成的一系列场景启动器，这些启动器帮我们导入了实现各个功能所需要依赖的全部组件，我们只需要在项目中引入这些starters，相关场景的所有依赖就会全部被导入进来，并且我们可以抛弃繁杂的配置，仅需要通过配置文件来进行少量的配置就可以使用相应的功能。



## 2、Starter 自动配置原理

在导入的starter之后，SpringBoot主要帮我们完成了两件事情：

- 相关组件的自动导入（实际上就是导入spring-boot-starter依赖，依赖中已经编写好自动配置类，等待SpringBoot启动扫描注入）
- 相关组件的自动配置（实际上就是启动SpringBoot应用，SpringBoot启动过程中会获取和注入自动配置类）

这两件事情统一称为SpringBoot的自动配置，如果想了解SpringBoot自动配置原理的可以参考另外两篇文章

1. SpringBoot 启动流程及原理
2. SpringBoot 启动类上的注解



## 3、Starter 官方与非官方模式分析

### 1、Starter 官方模式（SpringBoot开发）

选择一个官方的自动配置进行分析，这里就选择spring-boot-starter-web中常见的配置端口号配置。

#### 1、引入依赖

使用端口号之前我们需要先引入 web 依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

如果你观察 starter 多的话，也许你发已经发现了一个**模式**，SpringBoot 官方的 starter 的名字都是 spring-boot-starter-xxxx 命名的。

查看 spring-boot-starter-web 会发现，其实这个依赖只是一个空盒子，除了依赖其他 pom 之外，没有一行代码。

```bash
├─Maven: org.springframework.boot:spring-boot-starter-web:2.5.0
    ├─spring-boot-starter-web-2.5.0.jar
        ├─META-INF
            ├─LICENSE.txt
            ├─MANIFEST.MF
            ├─NOTICE.TXT
```

这时，发现了另外一个模式：starter 只依赖其他 pom，不做代码实现。那么 spring-boot-starter-web 到底依赖了哪些内容？

```java
├─Dependencies
    ├─org.springframework.boot:spring-boot-starter-web:2.5.0
        ├─org.springframework.boot:spring-boot-starter:2.5.0
        ├─org.springframework.boot:spring-boot-starter-json:2.5.0
        ├─org.springframework.boot:spring-boot-starter-tomcat:2.5.0
        ├─org.springframework:spring-web:5.3.7
        ├─org.springframework:spring-webmvc:5.3.7
```

观察这个依赖信息，然后再参照其他的官方 starter ，可以找到几个固定的引入，可以被称之为模式的依赖引入。

1. 依赖：spring-boot-starter
2. 依赖：spring-boot-autoconfigure



#### 2、自动配置

1、引入依赖让然后配置端口号

```properties
server.port=8090
```

IDEA 中可以通过点击 server.port 找到这个配置绑定的类文件。可以看到配置最终会注入到类 ServerProperties 类的 port 属性上。

```java
// 1.绑定的配置
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
    // 2.注入的属性
    /**
	 * Server HTTP port.
	 */
    private Integer port;
    // .....
}
```

那么这个 ServerProperties 到底是哪里使用的呢？继续查找，找到一个和 Servlet 的有关的调用。

发现是被 ServletWebServerFactoryCustomizer 类进行了调用，这个类里面定义了

```java
private final ServerProperties serverProperties;
```

用来使用配置的属性。继续查看这个类的调用，发现只有一个类使用这个类，这个类是 ServletWebServerFactoryAutoConfiguration。

```java
@Configuration(proxyBeanMethods = false)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
         ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
         ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
         ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {
}
```

根据我们对注解的理解，这个类就是自动配置主要类了。同时自动配置类都是以 AutoConfiguration 结尾。看这个类的几个注解的意思。

1. 优先级别较高

   ```java
   @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
   ```

2. 只有在 ServletRequest 类存在和是 Web 应用时生效

   ```java
   @ConditionalOnClass(ServletRequest.class)
   @ConditionalOnWebApplication(type = Type.SERVLET)
   ```

3. 开启了 ServerProperties 的配置绑定

   ```java
   @EnableConfigurationProperties(ServerProperties.class)
   ```

4. 导入了几个类

   ```java
   @Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
            ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
            ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
            ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
   ```

5. 同时注入配置到 Bean 工厂以供其他地方调用

   ```java
   @Bean
   public ServletWebServerFactoryCustomizer servletWebServerFactoryCustomizer(ServerProperties serverProperties) {
       return new ServletWebServerFactoryCustomizer(serverProperties);
   }
   ```

自动配置仅仅是这些东西吗？根据之前文章里的分析，我们知道不止代码，至少还有一个指定自动配置类的配置文件需要读取。也就是 spring.factories 文件。具体路径存在spring-boot-autoconfigure-2.5.0.jar/META-INF/spring.factories。

根据上面的分析，可以发现 SpringBoot 官方 starter 的几个**模式**。

1. 使用 XXXProperties 自动绑定 XXX 开头的配置信息，如：ServerProperties。
2. 把 XXXProperties 定义到要使用的类中，如：ServletWebServerFactoryCustomizer。
3. 编写一个 XXXAutoConfiguration ，开启 XXXProperties 的自动配置，限定生效场景，创建需要的类到 Bean 工厂。如：ServletWebServerFactoryAutoConfiguration。



### 2、Starter 非官方模式（第三方开发）

Springboot 官方如果把所有的框架都编写成 starter 是不现实的。因此很多第三方框架需要主动集成到 SpringBoot，所以我们选择一个常用的框架分析它的 starter 实现。因为已经看过了 SpringBoot 官方 starter 是如何配置的， 第三方框架也是类似，所以在下面观察的过程中会直接指出相同点，而不再做对比详细对比。

这里选择 mybatis-spring-boot-starter 进行学习分析。

#### 1、引入依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```

这里 mybatis 框架的 starter 依赖符合一定的规则，即：xxx-spring-boot-starter。观察这个 starter，发现它也没有做任何的代码实现，这一点和 SpringBoot 官方一致。

```bash
├─Maven: org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.2
    └─mybatis-spring-boot-starter-2.2.2.jar
        └─META-INF
            └─maven.org.mybatis.spring.boot.mybatis-spring-boot-starter
                ├─pom.properties
                ├─pom.xml
            └─MANIFEST.MF
```

可以看到它的META-INF目录下只包含了：

- pom.protperties：配置maven所需的项目version、groupId和artifactId
- pom.xml：配置所依赖的jar包
- MANIFEST.MF：这个文件描述了该Jar文件的很多信息

注意一下，没有一行代码。我们重点看一下`pom.xml`的依赖项，因为这个jar包里面除了这个没有啥重要的信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot</artifactId>
        <version>2.2.2</version>
    </parent>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <name>mybatis-spring-boot-starter</name>
    <properties>
        <module.name>org.mybatis.spring.boot.starter</module.name>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
        </dependency>
    </dependencies>
</project>
```

可以看出pom.xml中引入一些jar包，其中除了`spring-boot-starter`，之外重点看一下：`mybatis-spring-boot-autoconfigure`



#### 2、自动配置

查看 mybatis-spring-boot-autoconfigure 的内容发现和 SpringBoot 官方的 autoconfigure 结构上是差不多的。

```bash
├─Maven: org.mybatis.spring.boot:mybatis-spring-boot-autoconfigure:2.2.2
    ├─mybatis-spring-boot-autoconfigure-2.2.2.jar
        ├─META-INF
            ├─maven.org.mybatis.spring.boot.mybatis-spring-boot-autoconfigure
                ├─pom.properties
                ├─pom.xml
            ├─additional-spring-configuration-metadata.json
            ├─MANIFEST.MF
            ├─spring.factories
            ├─spring-autoconfigure-metadata.properties
            ├─spring-configuration-metadata.json
        ├─org.mybatis.spring.boot.autoconfigure
            ├─ConfigurationCustomizer
            ├─MybatisAutoConfiguration
            ├─MybatisLanguageDriverAutoConfiguration
            ├─MybatisProperties
            ├─SpringBootVFS
            ├─SqlSessionFactoryBeanCustomizer
```

里面包含如下文件：

- pom.properties：配置maven所需的项目version、groupId和artifactId
- pom.xml：配置所依赖的jar包
- additional-spring-configuration-metadata.json：手动添加IDE提示功能
- MANIFEST.MF：这个文件描述了该Jar文件的很多信息
- spring.factories：SPI会读取的文件
- spring-configuration-metadata.json：系统自动生成的IDE提示功能
- spring-autoconfigure-metadata.properties：系统自动生成的IDE提示功能
- ConfigurationCustomizer：自定义Configuration回调接口
- MybatisAutoConfiguration：mybatis配置类
- MybatisLanguageDriverAutoConfiguration：MyBatis脚本语言驱动程序的自动配置
- MybatisProperties：mybatis属性类
- SpringBootVFS：扫描嵌套的jar包中的类
- SqlSessionFactoryBeanCustomizer：回调接口，可以定制自动配置时生成的SqlSessionFactoryBean对象

mybatis 的自动配置也是通过 spring.factories 来指明自动配置，然后通过 XxxAutoConfiguration 绑定 XxxProperties 来进行自动配置

```java
// 如下所有注解都是一些辅助功能，决定Configuration是否生效，当然这些注解不是必须的
@Configuration
// @ConditionalOnClass 配置了只有包含SqlSessionFactory.class和SqlSessionFactoryBean.class，该配置类才生效
@ConditionalOnClass({SqlSessionFactory.class, SqlSessionFactoryBean.class})
// @ConditionalOnBean 配置了只有包含dataSource实例时，该配置类才生效
@ConditionalOnSingleCandidate(DataSource.class)
// @EnableConfigurationProperties 该注解会自动填充MybatisProperties实例中的属性
@EnableConfigurationProperties({MybatisProperties.class})
// AutoConfigureAfter 该配置类在DataSourceAutoConfiguration与MybatisLanguageDriverAutoConfiguratio类之后自动配置
@AutoConfigureAfter({DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class})
public class MybatisAutoConfiguration implements InitializingBean {
    // ...省略
}
```

在原理上，和上面 SpringBoot 官方的 starter 是相同的，所以不做过多的介绍了。可以自己点开源码分析。



## 4、场景启动器开发常用注解

注解使用已经大大方便我们开发，再也不需要写xml配置文件了，SpringBoot经过查找spring.factories文件，加载自动配置类，而自动配置类中定义了各种运行时判断条件，如@ConditionalOnMissingBean(A.class)等，只要IOC容器中没有指定的A类型的Bean信息，该配置文件才会生效。

@Conditional是Spring4新提供的注解，它的作用是按照一定的条件进行判断，满足条件给容器注册Bean。

- 属性映射注解
  - @ConfigurationProperties ：配置文件属性值和实体类的映射
  - @EnableConfigurationProperties：与@ConfigurationProperties配合，把@ConfigurationProperties修饰的类加入IOC容器

- 配置Bean注解
  - @Configuration ：标识该类为配置类，并把该类注入IOC容器
  - @Bean ：一般在方法上使用，声明一个Bean，Bean名称默认是方法名称，类型为返回值

- 条件注解：
  - @Conditional：根据条件类创建特定的Bean，条件类需要实现Condition接口，并重写matches接口来构造判断条件
  - @ConditionalOnBean ：容器中存在指定Bean，才会实例化一个Bean
  - @ConditionalOnMissingBean：容器中不存在指定Bean，才会实例化一个Bean
  - @ConditionalOnClass：系统中有指定类，才会实例化一个Bean
  - @ConditionalOnMissingClass：系统中没有指定类，才会实例化一个Bean
  - @ConditionalOnExpression：当SpEL表达式为true的时候，才会实例化一个Bean
  - @ConditionalOnWebApplication：是web应用
  - @ConditionalOnNotWebApplication：不是web应用
  - @ConditionalOnJava ：系统中版本是否符合要求
  - @ConditionalOnSingleCandidate：当指定的Bean在容器中只有一个，或者有多个但是指定了首选的Bean时触发实例化
  - @ConditionalOnResource：类路径下是否存在指定资源文件
  - @ConditionalOnJndi：JNDI指定存在项
  - @AutoConfigureAfter ：在某个Bean完成自动配置后实例化这个Bean
  - @AutoConfigureBefore ：在某个Bean完成自动配置前实例化这个Bean
  - @ConditionalOnProperty：配置Configuration的加载规则
    - prefix ：配置属性名称的前缀
    - value ：数组，获取对应property名称的值，与name不可同时使用
    - name ：数组，可与prefix组合使用，组成完整的配置属性名称，与value不可同时使用
    - havingValue ：比较获取到的属性值与havingValue给定的值是否相同，相同才加载配置
    - matchIfMissing ：缺少该配置属性时是否可以加载。如果为true，没有该配置属性时也会正常加载；反之则不生效




## 5、Full全模式和Lite轻量级模式

- @Configuration注解的参数：proxyBeanMethods：

- - 同一配置类下，当直接调用@Bean修饰的`方法`注入的对象，则调用`该方法不会被代理`，相当于直接调用一个普通方法，会有构造方法，但是没有bean的生命周期，返回的是不同的实例。

- - 同一配置类下，当直接调用@Bean修饰的`方法`注入的对象，则调用`该方法会被代理`，从ioc容器中取bean实列，所以实列是一样的。即单实例对象，在该模式下SpringBoot每次启动都会判断检查容器中是否存在该组件

- - Full 全模式（默认）：`@Configuration(proxyBeanMethods = true)`
  - Lite 轻量级模式：`@Configuration(proxyBeanMethods = false)`

- 注：proxyBeanMethods 是为了让使用@Bean注解的`方法`被代理。而不是@Bean的单例多例的设置参数。

```java
@Configuration(proxyBeanMethods = false)
public class AppConfig {
    // 注册myBean到IOC容器
    @Bean
    public MyBean myBean() {
        return new Mybean();
    }

    // 注册yourBean到IOC容器
    @Bean
    public YourBean yourBean() {
        System.out.println("==========");
        // 注意：@Configuration(proxyBeanMethods = false): myBean()方法不代理, 直接调用
        // 注意：@Configuration(proxyBeanMethods = true):  myBean()方法代理, 从IOC容器拿
        return new YourBean(myBean());
    }
}
```

具体什么时候用Full全模式，什么时候用Lite轻量级模式呢？

- 当在你的同一个Configuration配置类中，注入到容器中的Bean实例之间有依赖关系时，建议使用Full全模式
- 当在你的同一个Configuration配置类中，注入到容器中的Bean实例之间没有依赖关系时，建议使用Lite轻量级模式，以提高SpringBoot的启动速度和性能



## 6、自定义场景启动器的规范

1、SpringBoot Starter 的命名规范

官方命名空间

- 前缀：spring-boot-starter-
- 模式：spring-boot-starter-模块名
- 举例：spring-boot-starter-web、spring-boot-starter-jdbc

自定义命名空间

- 后缀：-spring-boot-starter
- 模式：模块-spring-boot-starter
- 举例：mybatis-spring-boot-starter

***

2、SpringBoot Starter 模块整体结构

- xxAutoConfiguration：自动配置类，对某个场景下需要使用到的组件进行自动注入，并利用xxProperties类来进行组件相关配置
- xxProperties：属性自动绑定类，某个场景下所有可配置属性的集成，在配置文件中配置可以进行属性值的覆盖，按照SpringBoot官方的定义，Starer的作用就是依赖聚合，因此直接在Starer内部去进行代码实现是不符合规定的，Starer应该只起到依赖导入的作用，而具体的代码实现应该去交给其他模块来实现，然后在Starer中去引用该模块即可。
- Starer模块依赖了两部分：一部分是一些常用依赖，另一部分就是对自动配置模块的依赖，而xxAutoConfiguration与xxProperties的具体实现，都封装在自动配置模块中，Starer实际是通过该模块来对外提供相应的功能。
- 实际上还有第三部分：自动处理类：xxService，这是可选的，可能每个工具处理方式不一样，命名也不一致。



## 7、Starter 自定义编写示例1（ID自动生成）

### 1、创建启动器空项目-spring-boot-starter

```bash
├─id-generate-spring-boot-starter
   │  pom.xml
   └─src
       └─main
           ├─java
           └─resources
```

由于启动器不需要代码实现，只需要依赖其他项目，所以直接创建一个空的 maven 项目（记住不是SpringBoot项目）。所以如果使用SpringBoot方法创建项目记得首先删除main启动类，没有main入口，需要去除pom文件中maven打包插件spring-boot-maven-plugin。但是记住项目名字要规范。这里创建的 starter 是 id-generate-spring-boot-starter。如下是添加依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>id-generate-spring-boot-starter</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <!--starter 模块中只进行依赖导入，添加对autoconfigure模块的依赖，如有需要可以添加一些其他必要依赖项-->
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>id-generate-spring-boot-autoconfigure</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
</project>
```

它只引入了id-generate-spring-boot-autoconfigure。当然如果有需要这里还可以引入多个autoconfigure或者多个其他jar包或者。



### 2、创建自动配置项目-spring-boot-autoconfigure

结合上面的项目，这里我们创建创建id-generate-spring-boot-autoconfigure项目，可以使用SpringBoot Initializr（这种方式记得删除main启动类，还有pom文件中的maven打包插件spring-boot-maven-plugin），所以还是建议直接创建一个简单的maven项目即可。项目结构如下所示：

```bash
├─id-generate-spring-boot-autoconfigure
   │  pom.xml
   ├─src
      ├─main
         ├─java
         │  └─org
         │      └─example
         │              IdGenerateAutoConfiguration.java
         │              IdGenerateProperties.java
         │              IdGenerateService.java
         └─resources
             └─META-INF
                     spring.factories
```

该项目当中包含：pom.xml、spring.factories、IdGenerateAutoConfiguration、IdGenerateService 和 IdGenerateProperties 这5个关键文件，下面我们逐一看看。

1、pom.xml，引入必要的依赖，视情况设置optional=true（有些业务相关类，设置optional后，需要用户方重新依赖）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>id-generate-spring-boot-autoconfigure</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- springboot的相关jar包, 也可不引入(引入spring-boot-autoconfigure的情况下) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- 包含自动配置的代码, 包含很多与自动配置相关的注解的定义, 必须要引入 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <!-- 非必须的, 配置文件点击可以跳转实体, 引入后可以在配置文件中输入我们自定义配置的时候有相应的提示 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- 其他依赖的选择根据项目需要进行添加即可 -->
    </dependencies>
</project>
```

我们可以看到，这个文件比较简单就引入了：

- spring-boot-starter：springboot的相关jar包
- spring-boot-autoconfigure：springboot自动配置相关jar包
- spring-boot-configuration-processor：springboot生成IDE提示功能相关jar包

spring-boot-configuration-processor 依赖的作用：

- spring-boot-configuration-processor 其实是一个注解处理器，在编译阶段干活的，一般maven的optional设置为true
- 你在idea里面可以点击port，进到这个字段里面，还可以看到配置的提示信息
- 这是因为在你的资源文件里面有一个spring-configuration-metadata.json文件，这是spring配置的元数据，是json形式

***

2、定义配置文件对应类IdGenerateProperties，为了让配置更加灵活，可以让使用方去配置参数

```java
package org.example;
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "id-generate")
public class IdGenerateProperties {
    /** 是否开启 **/
    private Boolean enable;
    /** 生成ID类型, uuid or bigint **/
    private String type = "bigint";
    /** 相当于自己给一串密钥丢进ID中 **/
    public Boolean getEnable() {
        return enable;
    }
    public void setEnable(Boolean enable) {
        this.enable = enable;
    }
    public String getType() {
        return type;
    }
    public void setType(String type) {
        this.type = type;
    }
}
```

它是一个配置实体类，里面包含了相关的配置文件。使用@ConfigurationProperties注解，会自动把application.properties文件中以id开通的，参数名称跟IdGenerateProperties中一样的参数值，自动注入到IdGenerateProperties对象中。

***

3、定义业务工具类IdGenerateService，用来生成随机ID的代码

```java
package org.example;
import java.util.*;

public class IdGenerateService {
    private final IdGenerateProperties idGenerateProperties;
    public IdGenerateService(IdGenerateProperties idGenerateProperties) {
        this.idGenerateProperties = idGenerateProperties;
    }

    public String generateId() {
        if (Objects.isNull(idGenerateProperties.getEnable()) || !idGenerateProperties.getEnable()) {
            throw new RuntimeException("id-generate function is not enabled");
        }
        if ("uuid".equals(idGenerateProperties.getType())) {
            return UUID.randomUUID().toString();
        } else if ("bigint".equals(idGenerateProperties.getType())) {
            return new Random().nextLong() + "";
        } else {
            throw new RuntimeException("id-generate type is error");
        }
    }
}
```

***

4、重点看一下 IdGenerateAutoConfiguration，核心自动配置代码

```java
package org.example;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Configuration: 会把此类加入Spring
 * @EnableConfigurationProperties: 导入我们自定义的配置类,供当前类使用
 * @ConditionalOnClass: 当存在某个类时，此自动配置类才会生效
 * @ConditionalOnProperty:
 *     判断id-generate.enable的值是否为“true”, 如果为true的化会加载此类
 *     matchIfMissing = true: 当前属性表示没有该配置属性时也会正常加载
 */
@Configuration
@EnableConfigurationProperties(IdGenerateProperties.class)
@ConditionalOnClass({IdGenerateProperties.class, IdGenerateService.class})
@ConditionalOnProperty(prefix = "id-generate", name = "enable", havingValue = "true", matchIfMissing = true)
public class IdGenerateAutoConfiguration {
    private final IdGenerateProperties properties;
    public IdGenerateAutoConfiguration(IdGenerateProperties properties) {
        this.properties = properties;
    }
    /**
     * @ConditionalOnMissingBean: 当容器里没有指定Bean的条件下此配置才会生效
     */
    @Bean
    @ConditionalOnMissingBean(IdGenerateService.class)
    public IdGenerateService idGenerateService() {
        System.out.println("create idGenerateService bean...");
        return new IdGenerateService(properties);
    }
}
```

该类是一个使用了@Configuration注解标记为了配置类，生效的条件是@ConditionalOnClass注解中检测到包含IdProperties.class。并且使用@EnableConfigurationProperties注解会自动注入IdProperties的实例。关键的点是该类里面创建了idGenerateService的bean实例，这是自动配置的精髓。

***

5、在resource/META-INF目录下新建spring.factories文件，SpringBoot会自动加载该文件并根据条件装配：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.example.IdGenerateAutoConfiguration
```

***

6、编写配置提示文件（非必须）：`resource/META-INF/additional-spring-configuration-metadata.json`

配置additional-spring-configuration-metadata.json文件后，在IDE工具使用编写的配置读取很有效的在application.properties 或 application.yml文件下完成提示。

配置详细格式参数可查看文档：https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/configuration-metadata.html#configuration-metadata.format

additional-spring-configuration-metadata.json 配置内容如下：

```json
{
    "properties": [
        {
            "name": "id-generate.enable",
            "type": "java.lang.Boolean",
            "description": "是否开启",
            "sourceType": "org.example.IdGenerateProperties"
        },
        {
            "name": "id-generate.type",
            "type": "java.lang.String",
            "description": "生成ID类型, uuid or bigint",
            "sourceType": "org.example.IdGenerateProperties",
            "defaultValue": "bigint"
        },
        {
            "name": "id-generate.staticId",
            "type": "java.lang.Integer",
            "description": "静态Id, 如果staticId有值则使用staticId当成初始Id值",
            "sourceType": "org.example.IdGenerateProperties",
            "defaultValue": "0",
            "deprecation": {
                "level": "error",
                "reason": "过时原因.",
                "replacement": "替代key是：id-generate.type"
            }
        }
    ]
}
```

参考下面properties表格进行配置上的理解。

| 名称         | 类型   | 目的                                                         |
| :----------- | :----- | :----------------------------------------------------------- |
| name         | String | 属性的全名。名称采用小写的周期分隔形式(例如server.address)。此属性是强制性的。 |
| type         | String | 属性的数据类型的完整签名（例如java.lang.String），但也是完整的泛型类型（例如java.util.Map<java.util.String,acme.MyEnum>）。您可以使用此属性来指导用户可以输入的值的类型。为了保持一致性，通过使用其包装对应项（例如，boolean变为java.lang.Boolean）来指定基元的类型。请注意，此类可能是一个复杂类型，它从Stringas绑定的值转换而来。`如果类型未知或基本类型，则可以省略。` |
| description  | String | 可以向用户显示的组的简短描述。如果没有可用的描述，则可以省略。建议描述为简短段落，第一行提供简明摘要。描述中的最后一行应以句点（.）结尾。 |
| sourceType   | String | 贡献此属性的源的类名称。例如，如果属性来自带注释的类@ConfigurationProperties，则此属性将包含该类的完全限定名称。如果源类型未知，则可以省略。 |
| defaultValue | Object | 默认值，如果未指定属性，则使用该值。如果属性的类型是数组，则它可以是值数组。如果默认值未知，则可以省略。 |
| deprecation  | 数组   | 过时的描述。                                                 |

`deprecation`每个`properties`元素的属性中包含的JSON对象可以包含以下属性：

| 名称        | 类型   | 目的                                                         |
| :---------- | :----- | :----------------------------------------------------------- |
| level       | String | 弃用级别，可以是warning（默认）或error。当属性具有warning弃用级别时，它仍应绑定在环境中。但是，当它具有error弃用级别时，该属性不再受管理且不受约束。 |
| reason      | String | 该属性被弃用的原因的简短描述。如果没有可用的原因，可以省略。建议描述为简短段落，第一行提供简明摘要。描述中的最后一行应以句点（.）结尾。 |
| replacement | String | 替换此不推荐使用的属性的属性的全名。如果此属性没有替换，则可以省略。 |

***

7、SpringBoot 配置文件提示及自动生成

spring-configuration-metadata.json 代码量挺大的，为了方便我们可以通过IDE来生成，这里使用的是idea。

**「在idea设置中搜索Annotation Processors，接下来勾住Enable annonation processing就完成了。在编译打包后的文件中看到自动生成的spring-configuration-metadata.json。这个文件不用我们编写」**

下面是自动生成的：

```json
{
    "groups": [
        {
            "name": "id-generate",
            "type": "org.example.IdGenerateProperties",
            "sourceType": "org.example.IdGenerateProperties"
        }
    ],
    "properties": [
        {
            "name": "id-generate.enable",
            "type": "java.lang.Boolean",
            "description": "是否开启",
            "sourceType": "org.example.IdGenerateProperties"
        },
        {
            "name": "id-generate.type",
            "type": "java.lang.String",
            "description": "生成ID类型, uuid or bigint",
            "sourceType": "org.example.IdGenerateProperties",
            "defaultValue": "bigint"
        }
    ],
    "hints": []
}
```

***

8、mvn install打包自定义starter与autoconfigure项目：id-generate-spring-boot-autoconfigure、id-generate-spring-boot-starter

然后新建一个SpringBoot项目：只需要引入id-generate-spring-boot-starter即可，测试代码如下：

```properties
id-generate.enable=true
id-generate.type=uuid
```

```java
@Autowired
private IdGenerateService idGenerateService;
@Test
void contextLoads() {
    String generateId = idGenerateService.generateId();
    System.out.println(generateId);
}
```

```
565886ae-6f0c-4207-a4ef-ff1948810cd1
```

***

9、**查看自动配置类生效的方法**：通过启用 debug=true 属性，开启SpringBoot的日志调试类，让控制台打印自动配置报告，这样就可以方便的知道哪些自动配置类生效。

```shell
 Positive matches:（自动配置类启用的：正匹配）
-----------------
 IdGenerateAutoConfiguration matched:
      - @ConditionalOnClass found required classes 'org.example.IdGenerateProperties', 'org.example.IdGenerateService' (OnClassCondition)
      - @ConditionalOnProperty (id-generate.enable=true) matched (OnPropertyCondition)

   IdGenerateAutoConfiguration#idGenerateService matched:
      - @ConditionalOnMissingBean (types: org.example.IdGenerateService; SearchStrategy: all) did not find any beans (OnBeanCondition)
      
Negative matches:（没有启动，没有匹配成功的自动配置类：负匹配）
-----------------
   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

Exclusions:
-----------

    org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration


Unconditional classes:
----------------------

    org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration
```

- Positive matches:（自动配置类启用的：正匹配）
- Negative matches:（没有启动，没有匹配成功的自动配置类：负匹配）
- Exclusions、Unconditional classes（排除的、没有限定条件的自动配置类）



## 8、Starter 自定义编写示例2（API日志打印）

本案例我们需要通过自动配置来创建一个拦截器对象，通过此拦截器对象来实现记录日志功能。

### 1、mylog-spring-boot-starter

```bash
├─mylog-spring-boot-starter
   │  pom.xml
   └─src
       └─main
           ├─java
           └─resources
```

创建一个空的 maven 项目，在mylog-spring-boot-starter的pom.xml文件中添加如下maven坐标（与上面案例一致）

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>mylog-spring-boot-starter</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>mylog-spring-boot-autoconfigure</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
</project>
```



### 2、mylog-spring-boot-autoconfigure

```bash
├─mylog-spring-boot-autoconfigure
   │  pom.xml
   └─src
       └─main
           ├─java
           │  └─org
           │      └─example
           │              MyLog.java
           │              MyLogAutoConfiguration.java
           │              MyLogInterceptor.java
           └─resources
               └─META-INF
                       spring.factories
```

1、pom.xml 文件引入依赖，这里增加了spring-boot-web依赖主要是为了使用拦截器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>mylog-spring-boot-autoconfigure</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- springboot的相关jar包, 也可不引入(引入spring-boot-autoconfigure的情况下) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- 包含自动配置的代码, 包含很多与自动配置相关的注解的定义, 必须要引入 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <!-- 非必须的, 配置文件点击可以跳转实体, 引入后可以在配置文件中输入我们自定义配置的时候有相应的提示 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- 其他依赖的选择根据项目需要进行添加即可 -->
        <!-- 由于使用了拦截器, 所以引入了spring-mvc, 只在编译阶段干活, 所以optional=true -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

2、自定义MyLog注解，用来记录那些接口需要打印日志

```java
package org.example;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyLog {
    /** 方法描述 **/
    String desc() default "";
}
```

3、自定义日志拦截器MyLogInterceptor，主要处理方法在这里编写，我这只打印日志，有需要也可以记录到数据库

```java
package org.example;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.lang.reflect.Method;

/**
 * 日志拦截器
 */
public class MyLogInterceptor implements HandlerInterceptor {
    private static final ThreadLocal<Long> startTimeThreadLocal = new ThreadLocal<>();

    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response, Object handler) {
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        // 获得被拦截的方法对象
        Method method = handlerMethod.getMethod();
        // 获得方法上的注解
        MyLog myLog = method.getAnnotation(MyLog.class);
        if (myLog != null) {
            // 方法上加了MyLog注解，需要进行日志记录
            long startTime = System.currentTimeMillis();
            startTimeThreadLocal.set(startTime);
        }
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) {
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        // 获得被拦截的方法对象
        Method method = handlerMethod.getMethod();
        // 获得方法上的注解
        MyLog myLog = method.getAnnotation(MyLog.class);
        if (myLog != null) {
            // 方法上加了MyLog注解，需要进行日志记录
            long endTime = System.currentTimeMillis();
            Long startTime = startTimeThreadLocal.get();
            long optTime = endTime - startTime;

            String requestUri = request.getRequestURI();
            String methodName = method.getDeclaringClass().getName() +
                    "." + method.getName();
            String methodDesc = myLog.desc();

            System.out.println("请求uri：" + requestUri);
            System.out.println("请求方法名：" + methodName);
            System.out.println("方法描述：" + methodDesc);
            System.out.println("方法执行时间：" + optTime + "ms");
        }
    }
}
```

4、创建自动配置类MyLogAutoConfiguration，用于自动配置拦截器、参数解析器等web组件

```java
package org.example;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 配置类，用于自动配置拦截器、参数解析器等web组件
 */
@Configuration
public class MyLogAutoConfiguration implements WebMvcConfigurer{
    // 注册自定义日志拦截器
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyLogInterceptor());
    }
}
```

5、在resource/META-INF/spring.factories中追加MyLogAutoConfiguration配置，让SpringBoot可以注入容器

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.example.MyLogAutoConfiguration
```

6、mvn intall 打包测试，在SpringBoot工程引入mylog-spring-boot-starter，在Controller方法上加入@MyLog注解

```java
@RestController
public class TestController {
    @MyLog(desc = "test api") //日志记录注解
    @GetMapping("/test")
    public String test() {
        return "ok";
    }
}
```

7、访问地址：localhost:8080/test，控制台打印结果如下：

```java
请求uri：/test
请求方法名：com.example.controller.TestController.test
方法描述：test api
方法执行时间：59ms
```



## 9、Starter 自定义编写示例3（AOP日志打印）

本案例我们需要通过自动配置来创建一个AOP，通过此AOP来实现记录日志功能，对比上面的日志打印区别是可以记录Service方法

### 1、aspectlog-spring-boot-starter

```bash
├─aspectlog-spring-boot-starter
   │  pom.xml
   └─src
       └─main
           ├─java
           └─resources
```

创建一个空的 maven 项目，在aspectlog-spring-boot-starter的pom.xml文件中添加如下maven坐标（与上面案例一致）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>aspectlog-spring-boot-starter</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>aspectlog-spring-boot-autoconfigure</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
</project>
```



### 2、aspectlog-spring-boot-autoconfigure

```bash
├─aspectlog-spring-boot-autoconfigure
   │  pom.xml
   └─src
       ├─main
          ├─java
          │  └─org
          │      └─example
          │              AspectLog.java
          │              AspectLogAutoConfiguration.java
          │              AspectLogProperties.java
          └─resources
              └─META-INF
                      spring.factories
```

1、pom.xml 依赖，这里增加了spring-boot-aop依赖，也可以使用spring-boot-web依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>aspectlog-spring-boot-autoconfigure</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- springboot的相关jar包, 也可不引入(引入spring-boot-autoconfigure的情况下) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- 包含自动配置的代码, 包含很多与自动配置相关的注解的定义, 必须要引入 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <!-- 非必须的, 配置文件点击可以跳转实体, 引入后可以在配置文件中输入我们自定义配置的时候有相应的提示 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- 其他依赖的选择根据项目需要进行添加即可 -->
        <!-- 这里使用到了AOP, 所以增加AOP的依赖, 由于增加optional=true, 所以使用者还需引入该依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

2、定义AspectLog注解，该注解用于标注需要打印执行时间的方法

```java
package org.example;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
/**
 * 用于控制定时任务的开启与关闭
 * 对应切面
 **/
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AspectLog {
}
```

3、定义配置文件对应类

```java
package org.example;
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "aspect-log")
public class AspectLogProperties {
    private boolean enable;
    public boolean isEnable() {
        return enable;
    }
    public void setEnable(boolean enable) {
        this.enable = enable;
    }
}
```

4、定义自动配置类

```java
package org.example;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.autoconfigure.condition.*;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.core.PriorityOrdered;

@Aspect
@EnableAspectJAutoProxy(exposeProxy = true, proxyTargetClass = true)
@Configuration
@EnableConfigurationProperties(AspectLogProperties.class)
@ConditionalOnProperty(prefix = "aspectLog", name = "enable", havingValue = "true", matchIfMissing = true)
public class AspectLogAutoConfiguration implements PriorityOrdered {
    protected Logger logger = LoggerFactory.getLogger(getClass());

    @Around("@annotation(org.example.AspectLog)")
    public Object isOpen(ProceedingJoinPoint thisJoinPoint) throws Throwable {
        // 执行方法名称
        String taskName = thisJoinPoint.getSignature()
                .toString().substring(
                        thisJoinPoint.getSignature()
                                .toString().indexOf(" "),
                        thisJoinPoint.getSignature().toString().indexOf("("));
        taskName = taskName.trim();
        long time = System.currentTimeMillis();
        Object result = thisJoinPoint.proceed();
        logger.info("method:{} run :{} ms", taskName, (System.currentTimeMillis() - time));
        return result;
    }

    @Override
    public int getOrder() {
        // 保证事务等切面先执行
        return Integer.MAX_VALUE;
    }
}
```

配置类简要说明：

```java
// 1.当配置文件有aspectLog.enable=true时开启，如果配置文件没有设置aspectLog.enable也开启
@ConditionalOnProperty(prefix = "aspectLog", name = "enable",havingValue = "true", matchIfMissing = true)
// 2.此注解作用是把当前类标识为一个切面供容器读取
@Aspect
/**
 * 3.@EnbleAspectJAutoProxy类有两个成员属性:
 * - proxyTargetClass: 表明该类采用CGLIB代理还是使用JDK的动态代理，如果选择为true则是cglib，
 *                     false则尽可能选择JDK，如果bean没有实现任何接口，就算是false也会选择cglib
 * - exposeProxy: 解决内部调用不能使用代理的场景，默认为false表示不处理，
 *                true则表示这个代理对象的副本可以通过AopContext.currentProxy()获得(ThreadLocal里面),
 *                从而我们可以很方便得在Spring框架上下文中拿到当前代理对象（处理事务时很方便）
 *
 * 注意: Spring中如果不在配置类中添加@EnableAspectJAutoProxy，那么所有切面注解是不生效的
 *      SpringBoot因为有自动配置，所以不需要开发人员手工配置@EnableAspectJAutoProxy　　　
 */
@EnableAspectJAutoProxy(exposeProxy = true, proxyTargetClass = true)
```

5、META-INF/spring.factories是Spring的工厂机制，文件中定义的类，都会被自动加载。多个配置使用逗号分割，换行用`\`

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.example.AspectLogAutoConfiguration
```

6、mvn intall 打包测试，在SpringBoot工程引入aspectlog-spring-boot-starter，在任意方法上加入@AspectLog注解测试

```properties
aspect-log.enable=true
```

```java
@Service
public class TestService {
    @SneakyThrows
    @AspectLog
    public void sleep () {
        TimeUnit.SECONDS.sleep(1);
    }
}
```

```java
@Autowired
TestService testService;
@Test
void testAspectLog() {
    testService.sleep();
}
```

7、执行测试方法，控制台打印结果如下：

```
method:com.swagger3.service.TestService.sleep run :1019 ms
```



## 10、SpringBoot 自定义多数据源 starter 组件

本案例我们使用多数据源封装成一个starter组件，以方便使用多数据源访问数据库的操作。

```bash
├─dynamic-multi-datasource-spring-boot-starter
   │  pom.xml
   └─src
       ├─main
          ├─java
          │  └─org
          │      └─example
          │              DS.java
          │              DynamicDataSourceAspect.java
          │              DynamicDataSourceContextHolder.java
          │              DynamicMultiDataSourceAutoConfiguration.java
          │              DynamicMultiDataSourceProperties.java
          └─resources
              └─META-INF
                      spring.factories
```



### 1、创建普通项目，引入相关的依赖

创建一个普通Java项目，引入SpringBoot相关的依赖。pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>org.example</groupId>
    <artifactId>dynamic-multi-datasource-spring-boot-starter</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- 包含了springboot与spring的相关jar包, 也包括spring-boot-autoconfigure -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- 非必须的, 配置文件点击可以跳转实体, 引入后可以在配置文件中输入我们自定义配置的时候有相应的提示 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- 其他依赖的选择根据项目需要进行添加即可 -->
        <!-- 导入jdbc相关依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!-- 这里使用到了AOP, 所以增加AOP的依赖, 这里不能optional=true, 因为配置中使用了new -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```



### 2、创建常量类和注解

```java
package org.example;
import java.lang.annotation.*;

/**
 * 数据源切换扫描注解及常量
 */
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DS {
    String value() default DS.DEFAULT;
    /**
     * 如下为常量
     */
    String DEFAULT = "default";
    String DY_MULTI_DS = "dynamicMultiDataSource";
    String DS_TYPE = "datasourceType";
    String CONFIG_PREFIX = "spring.datasource.multi";
}
```



### 3、创建多数据源属性类

```java
package org.example;
import org.springframework.boot.context.properties.ConfigurationProperties;
import java.util.HashMap;
import java.util.Map;

/**
 * 主要用于存储SpringBoot配置文件中配置的数据源属性
 */
@ConfigurationProperties(prefix = DS.CONFIG_PREFIX)
public class DynamicMultiDataSourceProperties {
    private Map<String, DataSourceProp> dataSourcePropMap;

    public Map<String, DataSourceProp> getDataSourcePropMap() {
        return dataSourcePropMap;
    }

    public void setDataSourcePropMap(Map<String, DataSourceProp> dataSourcePropMap) {
        this.dataSourcePropMap = dataSourcePropMap;
    }

    public static class DataSourceProp extends HashMap<String, String> {
    }
}
```



### 4、创建数据源key的切换工具

```java
package org.example;

/**
 * 主要用于设置当前线程下数据源切换时的数据源唯一标识key, 以便获取指定的数据源
 */
public class DynamicDataSourceContextHolder {

    /**
     * 动态数据源名称上下文
     */
    private static final ThreadLocal<String> DATA_SOURCE_THEAD_LOCAL =
            ThreadLocal.withInitial(() -> DS.DEFAULT);

    /**
     * 返回当前数据源
     */
    public static String getDataSource() {
        return DATA_SOURCE_THEAD_LOCAL.get();
    }

    /**
     * 设置当前数据源
     */
    public static void setDataSource(String dataSource) {
        DATA_SOURCE_THEAD_LOCAL.set(dataSource);
    }

    /**
     * 重置释放数据源
     */
    public static void remove() {
        DATA_SOURCE_THEAD_LOCAL.remove();
    }
}
```



### 5、创建多数据源的切面类

切面类主要用于获取被数据与注解指定的方法，拿到其注解中的属性值，再设置到数据源key设置组件中，方便数据源类获取该key，需使用@Order设置切面优先级，或者继承PriorityOrdered重写getOrder()也可以，否则设置无效。

```java
package org.example;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.Order;
import java.lang.reflect.Method;

@Order
@Aspect
public class DynamicDataSourceAspect {
    private static final Logger log = LoggerFactory.getLogger(DynamicDataSourceAspect.class);

    @Around("@annotation(org.example.DS)")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        try {
            // 获取被代理的方法对象、数据源注解
            MethodSignature signature = (MethodSignature) point.getSignature();
            Method method = signature.getMethod();
            DS DS = method.getAnnotation(DS.class);
            log.debug("切换前数据源为: {}" + DS.value());
            DynamicDataSourceContextHolder.setDataSource(DS.value());
            return point.proceed();
        } finally {
            DynamicDataSourceContextHolder.remove();
        }
    }
}
```



### 6、创建数据源配置类及多数据源类

创建多数据源类继承AbstractRoutingDataSource类，重写determineCurrentLookupKey()方法，用于获取当前线程中的指定的数据源key，通过该key拿到对应的数据源对象。此类DynamicMultiDataSource我偷懒了写成静态内部类了。

```java
package org.example;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import org.springframework.util.ReflectionUtils;
import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

@Configuration
@EnableConfigurationProperties(DynamicMultiDataSourceProperties.class)
public class DynamicMultiDataSourceAutoConfiguration {
    private final DynamicMultiDataSourceProperties dynamicMultiDataSourceProperties;

    public DynamicMultiDataSourceAutoConfiguration(DynamicMultiDataSourceProperties dynamicMultiDataSourceProperties) {
        this.dynamicMultiDataSourceProperties = dynamicMultiDataSourceProperties;
    }

    /**
     * 必须实现该接口。用来获取当前数据源
     */
    static class DynamicMultiDataSource extends AbstractRoutingDataSource {
        @Override
        protected Object determineCurrentLookupKey() {
            return DynamicDataSourceContextHolder.getDataSource();
        }
    }

    @Primary
    @Bean(DS.DY_MULTI_DS)
    public DataSource dynamicMultiDataSource() {
        DynamicMultiDataSource dynamicMultiDataSource = new DynamicMultiDataSource();
        Map<String, DynamicMultiDataSourceProperties.DataSourceProp> dataSourcePropMap = dynamicMultiDataSourceProperties.getDataSourcePropMap();

        Map<Object, Object> dataSourceMap = new HashMap<>(dataSourcePropMap.size());
        dataSourcePropMap.forEach((lookupKey, dsProp) -> dataSourceMap.put(lookupKey, createDs(dsProp)));

        // 添加数据源集合, 然后设置默认数据源
        dynamicMultiDataSource.setTargetDataSources(dataSourceMap);
        dynamicMultiDataSource.setDefaultTargetDataSource(dataSourceMap.get(DS.DEFAULT));
        return dynamicMultiDataSource;
    }

    @Bean
    public DataSourceTransactionManager dataSourceTransactionManager(
            @Qualifier(DS.DY_MULTI_DS) DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean
    public DynamicDataSourceAspect dynamicDataSourceAspect() {
        return new DynamicDataSourceAspect();
    }

    /**
     * 这里是定义了Map所以才这样处理了, 实际上也可以直接定义一个JavaBean，
     * 然后设置设定的字段, 如果有需要还能把连接池配置一起带上
     */
    private DataSource createDs(DynamicMultiDataSourceProperties.DataSourceProp dataSourceProp) {
        DataSource dataSource = null;
        try {
            Class<?> dsClass = Class.forName(dataSourceProp.get(DS.DS_TYPE));
            if (DataSource.class.isAssignableFrom(dsClass)) {
                dataSource = (DataSource) dsClass.getConstructor().newInstance();

                DataSource finalDataSource = dataSource;
                // 反射获取指定类中的属性
                ReflectionUtils.doWithFields(dsClass,
                        field -> {
                            field.setAccessible(true);
                            field.set(finalDataSource, dataSourceProp.get(field.getName()));
                        },
                        field -> {
                            if (Objects.equals(dataSourceProp.get(DS.DS_TYPE), field.getName())) {
                                return false;
                            }
                            return Objects.nonNull(dataSourceProp.get(field.getName()));
                        });

            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        return dataSource;
    }
}
```



### 7、配置spring.factories文件

在resources目录下创建META-INF目录，在该目录创建spring.factories文件，内容如下：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.example.DynamicMultiDataSourceAutoConfiguration
```

设置key为开启自动配置的注解全路径名，后面的value值为配置类全路径名，本starter组件中为数据源配置类，如有多个配置类，则以逗号分隔，以反斜杆表示忽略换行



### 9、新建SpringBoot项目测试示例

我们封装的一个简单的多数据源starter组件就完成了，只需进行maven打包即可在本地使用。为了方便测试使用H2内存数据库。

1、maven 命令：`mvn clean install`。引入打包后的依赖：

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>dynamic-multi-datasource-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

2、修改SpringBoot全局配置文件：default为默认数据源，必须配置， master为可选数据源，名称可自定义。数据源的属性名称为对应的dsType数据源类型的属性字段。

```yaml
spring:
  datasource:
    multi:
      data-source-prop-map:
        default:
          datasourceType: com.zaxxer.hikari.HikariDataSource
          driverClassName: org.h2.Driver
          jdbcUrl: jdbc:h2:mem:default
          username: sa
          password: sa
        master:
          datasourceType: com.zaxxer.hikari.HikariDataSource
          driverClassName: org.h2.Driver
          jdbcUrl: jdbc:h2:mem:master
          username: sa
          password: sa
```

3、使用数据源：直接在指定的方法上添加@DataSource注解即可，注解的默认值为default,数据源的切换通过注解的值进行切换。值为application.yml中配置的default，master等。

4、查看完整pom.xml与代码，yml配置文件参考上面即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>test-spring-boot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>test-spring-boot</name>
    <description>test-spring-boot</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
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
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>dynamic-multi-datasource-spring-boot-starter</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

```java
package com.example.testspringboot;
import org.example.DS;
import org.example.DynamicDataSourceContextHolder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.PostConstruct;
import javax.sql.DataSource;
import java.util.Arrays;
import java.util.List;
import java.util.Map;

@RestController
@SpringBootApplication
public class TestSpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestSpringBootApplication.class, args);
    }

    @Autowired
    DataSource dynamicMultiDataSource;
    JdbcTemplate jdbcTemplate;
    @PostConstruct
    public void initData() {
        jdbcTemplate = new JdbcTemplate(dynamicMultiDataSource);
        Arrays.asList("default", "master").forEach(type -> {
            // 切换数据源
            DynamicDataSourceContextHolder.setDataSource(type);
            // 初始化表和数据
            jdbcTemplate.execute("DROP TABLE IF EXISTS user");
            jdbcTemplate.execute("CREATE TABLE user ("
                    + "id BIGINT NOT NULL,"
                    + "name VARCHAR(50) DEFAULT NULL,"
                    + "type VARCHAR(50) DEFAULT NULL,"
                    + "PRIMARY KEY (id))");
            String sql = "INSERT INTO user (id, name, type) VALUES (?, ?, ?)";
            jdbcTemplate.update(sql, 1, "Sam", type);
            jdbcTemplate.update(sql, 2, "Kath", type);
            jdbcTemplate.update(sql, 3, "Tom", type);
            jdbcTemplate.update(sql, 4, "Sandy", type);
            jdbcTemplate.update(sql, 5, "Oliver", type);
        });
        // 还原数据源
        DynamicDataSourceContextHolder.remove();
    }

    @DS
    @GetMapping("/default")
    public List<Map<String, Object>> test1(){
        return jdbcTemplate.queryForList("select * from user");
    }

    @DS("master")
    @GetMapping("/master")
    public List<Map<String, Object>> test2(){
        return jdbcTemplate.queryForList("select * from user");
    }
}
```

5、动态切换多数据测试，我这里直接 CURL 请求

```shell
➜  ~ curl localhost:8080/default
[{"ID":1,"NAME":"Sam","TYPE":"default"},
{"ID":2,"NAME":"Kath","TYPE":"default"},
{"ID":3,"NAME":"Tom","TYPE":"default"},
{"ID":4,"NAME":"Sandy","TYPE":"default"},
{"ID":5,"NAME":"Oliver","TYPE":"default"}]

➜  ~ curl localhost:8080/master
[{"ID":1,"NAME":"Sam","TYPE":"master"},
{"ID":2,"NAME":"Kath","TYPE":"master"},
{"ID":3,"NAME":"Tom","TYPE":"master"},
{"ID":4,"NAME":"Sandy","TYPE":"master"},
{"ID":5,"NAME":"Oliver","TYPE":"master"}]
```







