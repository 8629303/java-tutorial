# 日志的概念

## 1、日志简介

### 1、日志是什么

> 日志：记录程序的运行轨迹，方便查找关键信息，也方便快速定位解决问题

通常，Java程序员在开发项目时都是依赖Eclipse/IDEA等集成开发工具的Debug 调试功能来跟踪解决Bug，但项目发布到了测试、生产环境怎么办？你有可能会说可以使用远程调试，但实际并不能允许让你这么做。

所以，日志的作用就是在测试、生产环境没有 Debug 调试工具时开发和测试人员定位问题的手段。日志打得好，就能根据日志的轨迹快速定位并解决线上问题，反之，日志输出不好，不仅无法辅助定位问题反而可能会影响到程序的运行性能和稳定性。

很多介绍 AOP 的地方都采用日志来作为介绍，实际上日志要采用切面的话是极其不科学的！对于日志来说，只是在方法开始、结束、异常时输出一些什么，那是绝对不够的，这样的日志对于日志分析没有任何意义。如果在方法的开始和结束整个日志，那方法中呢？如果方法中没有日志的话，那就完全失去了日志的意义！如果应用出现问题要查找由什么原因造成的，也没有什么作用。这样的日志还不如不用！



### 2、日志的作用

不管是使用何种编程语言，日志输出几乎无处不再。总结起来，日志大致有以下几种用途：

- **「问题追踪」**：辅助排查和定位线上问题，优化程序运行性能
- **「状态监控」**：通过日志分析，可以监控系统的运行状态
- **「安全审计」**：审计主要体现在安全上，可以发现非授权的操作

**总结**：日志在应用程序中是非常非常重要的，好的日志信息能有助于我们在程序出现BUG时能快速进行定位，并能找出其中的原因。作为一个有修养的程序猿，对日志这个东西应当引起足够的重视。



## 2、常见日志框架

1、日志门面

- JCL（Apache Commons Logging，原名 Jakarta Commons Logging）：commons-logging 是日志的门面接口，它也是Apache 最早提供的日志门面接口，用户可以根据喜好选择不同的日志实现框架，而不必改动日志定义，这就是日志门面的好处，符合面对接口抽象编程。现在已经不太流行了，了解一下就行
- SLF4j（ Simple Logging Facade for Java）：为Java提供的简单日志Facade。Facade门面，更底层一点说就是接口。它允许用户以自己的喜好，在工程中通过slf4j接入不同的日志系统。因此slf4j入口就是众多接口的集合，它不负责具体的日志实现，只在编译时负责寻找合适的日志系统进行绑定。具体有哪些接口，全部都定义在slf4j-api中。查看slf4j-api源码就可以发现，里面除了public final class LoggerFactory类之外，都是接口定义。因此slf4j-api本质就是一个接口定义。
- jboss-logging（诞生就不是为了服务大众）

2、日志实现：JUL（java.util.logging）、Log4j、Log4j2、Logback

- JUL：这是 Java 自带的日志工具类，在 JDK 1.5 开始就已经有了，在 java.util.logging 包下。通常情况下，这个基本没什么人用了，了解一下就行
- Log4j：是 Apache 的一个开源日志框架，也是市场占有率最多的一个框架。注意：log4j 在 2015.08.05 这一天被 Apache 宣布停止维护了，用户需要切换到 Log4j2上面去
- Log4j2：Apache Log4j 2是apache开发的一款Log4j的升级产品。Log4j2与Log4j1发生了很大的变化，log4j2不兼容log4j1
- Logback：是 Slf4j 的原生实现框架，同样也是出自 Log4j 一个人之手，但拥有比 log4j 更多的优点、特性和更做强的性能，现在基本都用来代替 log4j 成为主流

SLF4J、Log4j、Logback都是同一个人Ceki写的。为什么写三个呢？那是因为Ceki先写了Log4j，但后面觉得Log4j太烂了，他已经不想改了。所以后面写了logback。Log4j2呢，从设计上来说很优秀，但太优秀了，太先进了，很多开源框架对其支持有限（Log4j2性能优于logback，但为了有高性能，设计得过于复杂，没必要，用Logback就够了）

如下是日志结构框架：

```
┌───────────────────┐                  ┌────────────────────────────────────┐
│    日志适配器      │    »»»»»»»»»     │              日志门面               │
│                   │    »»»»»»»»»     │        slf4j     commons-logging   │
│                   │                  └────────────────────────────────────┘
│   jul-to-slf4j    │                  ┌────────────────────────────────────┐
│  log4j-over-slf4j │                  │            日志门面的适配器          │
│   jcl-to-slf4j    │                  │ slf4j-jdk  slf4j-jcl slf4j-log4j12 |
└───────────────────┘                  └────────────────────────────────────┘
┌───────────────────────────────────────────────────────────────────────────┐
│                           日志库的具体实现                                  │
│      log4j       log4j2       logback       log-jdk       otherLogging    │
└───────────────────────────────────────────────────────────────────────────┘
```



## 3、日志框架怎么选

选项太多了的后果就是选择困难症，我的看法是没有最好的，只有最合适的：

- commons-loggin、slf4j 只是一种日志抽象门面，不是具体的日志框架。log4j、logback 是具体的日志实现框架。
- 在关注性能的地方，选择Logback或自己实现高性能Logging API可能更合适。推荐：slf4j + logback.
- 在非常关注异步性能或已经使用了Log4j的项目中，如果没有发现问题，继续使用可能是更合适的方式：推荐组合为：slf4j + Log4j2
- 如果不想有依赖则使用java.util.logging或框架容器已经提供的日志接口

***

- SLF4J实现机制决定SLF4J限制较少，使用范围更广。由于SLF4J在编译期间，静态绑定本地的LOG库使得通用性要比Commons Logging要好
- Commons Logging开销更高，所以一般日志门面都会选择SLF4J
- Logback拥有更好的性能。Logback声称：某些关键操作，比如判定是否记录一条日志语句的操作，其性能得到了显著的提高。这个操作在Logback中需要3纳秒，而在Log4J中则需要30纳秒。LogBack创建记录器（logger）的速度也更快：13毫秒，而在Log4J中需要23毫秒。更重要的是，它获取已存在的记录器只需94纳秒，而Log4J需要2234纳秒，时间减少到了1/23。跟JUL相比的性能提高也是显著的（这只是在Log4j2出现之前）
- Logback文档免费。Logback的所有文档是全面免费提供的，不象Log4J那样只提供部分免费文档而需要用户去购买付费文档
- Log4j2的出现改变的Logback的地位，他重写了Log4j1.x并修复了Logback中的不足，在性能方面不管是同步还是异步功能都吊打所有的日志框架
- 总结：如果是使用SpringBoot开发，正常没有特殊性能要求的话，使用Logback或Log4j2都行，建议使用Logback，SpringBoot自带可以减少引入依赖，如果对性能有要求，并且对功能要求较高，可以使用Log4j2（在功能和性能上都比Logback丰富）



## 4、日志发展历史

### 1、阶段一（System）

2001年以前，Java是没有日志库的，打印日志全凭：System.out 和 System.err。缺点：

1. 产生大量的IO操作同时在生产环境中无法合理的控制是否需要输出
2. 输出的内容不能保存到文件
3. 只打印在控制台，打印完就过去了，也就是说除非你一直盯着程序跑
4. 无法定制化，且日志粒度不够细



### 2、阶段二（Log4j）

2001年，一个叫Ceki Gulcü的大佬搞了一个日志框架 log4j后来（Log4j成为Apache项目），Ceki加入Apache组织Apache还曾经建议Sun引入Log4j到Java的标准库中，但Sun拒绝了。



### 3、阶段三（JUL）

Sun有自己的小心思，2002年2月JDK1.4发布，Sun推出了自己的日志标准库JUL（Java Util Logging），其实是照着Log4j抄的，而且还没抄好，还是在JDK1.5以后性能和可用性才有所提升。由于Log4j比JUL好用，并且成熟，所以Log4j在选择上占据了一定的优势。



### 4、阶段四（JCL）

2002年8月Apache推出了JCL（Jakarta Commons Logging），也就是日志抽象层，支持运行时动态加载日志组件的实现，当然也提供一个默认实现Simple Log（在ClassLoader中进行查找，如果能找到Log4j则默认使用Log4j实现，如果没有则使用JUL实现，再没有则使用JCL内部提供的Simple Log实现）

```
                                      项目
                                       ⬇
                                      JCL
                                       ⬇
     ┌────────────────────┬───────────────────────────┬──────────────────────────────┐
Log4JLogger          JDK14Logger              Jdk13LumberjackLogger              SimpleLog
```

JUL有三个缺点：

1．效率较低
2．容易引发混乱
3．使用了自定义ClassLoader的程序中，使用JCL会引发内存泄露



### 5、阶段五（SLF4J）

> SLF4J的由来与背景：在Log4j开发出来出来之后，Log4j就受到了广大开发者的爱好，纷纷开始使用Log4j，但是后来Log4j的创始人跟Apache因为一些矛盾从Apache辞职自己去创业了，创始人为了给自己的公司打出一点名声，所以就基于Log4j又开发出了一个新的日志框架Logback，Logback不管是性能还是功能都比Log4j强，但是却很少人使用， 因为JCL门面不支持Logback，所以这个创世人又设计出了slf4j，slf4j支持市面上所有主流的日志框架；所以目前为止，使用最多的就是SLF4J了； 后来Apache又基于Logback的源码设计出了Log4j2日志，性能上Log4j2比Logback更胜一筹，并且Log4j2既是日志框架，也是门面技术，但是Log4j2的门面技术很少人使用，大多还是使用SLF4J。

2006年巨佬Ceki（Log4j的作者）因为一些原因离开了Apache组织，之后Ceki觉得JCL不好用，自己重写了一套新的日志标准接口规范Slf4j（Simple Logging Facacfor Java），也可以称为日志门面，很明显Slf4j是对标JCL，后面也证明了Slf4j比JCL更优秀。

大佬Ceki提供了一系列的桥接包来帮助Slf4j接口与其他日志库建立关系，这种方式称桥接设计模式。代码使用Slf4j接口，就可以实现日志的统一标准化，后续如果想要更换日志实现，只需引入Slf4j与相关的桥接包，再引入具体的日志标准库即可。



### 6、阶段六（Logback）

Ceki巨佬觉得市场上的日志标准库都是间接实现Slf4j接口，也就是说每次都需要配合桥接包，因此在2006年，Ceki巨佬基于Slf4j接口写出了Logback日志标准库，做为Slf4j接口的默认实现，Logback也十分给力，在功能完整度和性能上超越了所有已有的日志标准库。

- 根本原因还在于，随着用户体量的提升，Log4j无法满足高性能的要求，成为应用的性能瓶颈
- 通过SLF4j桥接到具体的日志框架实现。日志框架的绑定（Binding）：绑定其他日志的实现
- 通过其他日志框架桥接到SLF4j。日志框架的桥接（Bridging）：桥接旧的日志框架



### 7、阶段七（Log4j2）

> 产生背景：Logback算是JAVA 里一个老牌的日志框架，从06年开始第一个版本，迭代至今也十几年了。不过logback最近一个最稳定版本还停留在 2017 年，好几年都没有更新大版本了Llogback的兄弟SLF4J最近一个稳定版也是2017年。而且Logback的异步性能实在拉跨，功能简陋，配置又繁琐，远不及Apache的新一代日志框架：Log4j2。目前来看，Log4j2就是王者，其他日志框架都不是对手。

2012年，Apache直接推出新项目Log4j2（不兼容Log4j），Log4j2全面借鉴Slf4j+Logback 。Log4j2不仅仅具有Logback的所有特性，还做了分离设计，分为log4j-api和log4j-core，log4j-api是日志接口，log4j-core是日志标准库，并且Apache也为Log4j2提供了各种桥接包。而且log4j2 的性能提升很大，而且支持异步日志打印。增加很多新的特性



## 5、日志打印的建议

### 1、打日志建议简单版

1. 应用中不可直接使用日志系统中的API (Log4j，Logback)，而应依赖使用日志框架SLF4J中的API。使用门面模式有利于维护和各个类的日志处理方法统一
2. 日志文件推荐至少保存15天，因为有些异常具备以“周”为频次发生的特点
3. 应用中的扩展日志（如打点、临时监控、访问日志等）命名方式：appName_logType_logName.log。logType为日志类型，推荐分类有stats/monitor/visit 等
4. logName为日志描述。这种命名的好处：通过文件名就可以知道日志文件属于哪个应用，哪种类型，有什么目的，这也有利于归类查找
5. 对trace、debug、info级别的日志输出，必须使用条件输出形式或者占位符的方式
6. 避免重复打印日志，否则会浪费磁盘空间。务必在日志配置文件中设置additivity=false
7. 异常信息应该包括两类：案发现场信息和异常堆栈信息。如果不处理，那么通过关键字向上抛出
8. 谨慎地记录日志。生产环境禁止输出debug日志；有选择地输出info日志；如果使用warn记录刚上线时的业务行为信息，一定要注意日志输出量的问题，避免吧服务器磁盘撑爆，并及时删除这些观察日志
9. 可以使用warn日志级别记录用户输入参数错误的情况，避免当用户投诉时无所适从



### 2、打日志建议详情版

> [工作总结：日志打印的15个建议 (qq.com)](https://mp.weixin.qq.com/s/sdNazXZmuQkIZWkA5_rINw)

日志是快速定位问题的好帮手，是撕逼和甩锅的利器！打印好日志非常重要。今天我们来聊聊日志打印的15个好建议

1、选择恰当的日志级别

常见的日志级别有5种，分别是error、warn、info、debug、trace。日常开发中，我们需要选择恰当的日志级别，不要直接打印info

- error：错误日志，指比较严重的错误，对正常业务有影响，需要**运维配置监控的**；
- warn：警告日志，一般的错误，对业务影响不大，但是需要**开发关注**；
- info：信息日志，记录排查问题的关键信息，如调用时间、出参入参等等；
- debug：用于开发DEBUG的，关键逻辑里面的运行时数据；
- trace：最详细的信息，一般这些信息只记录到日志文件中。

2、日志要打印出方法的入参和出参

我们并不需要打印很多很多日志，只需要打印可以快速定位问题的有效日志。有效的日志，才能有效的查到问题！哪些算得的上有效关键的日志呢？比如说，方法进来的时候，打印入参。再然后呢，在方法返回的时候，就是打印出参，返回值。入参的话，一般就是userId或者bizSeq这些关键信息。正例如下：

```java
public String testLogMethod(Document doc, Mode mode){
    log.debug(“method enter param：{}”,userId);
    String id = "666";
    log.debug(“method exit param：{}”,id);
    return id;
}
```

3、选择合适的日志格式

理想的日志格式，应当包括这些最基本的信息：如当**前时间戳**（一般毫秒精确度）、**日志级别**，**线程名字**等等。在logback日志里可以这么配置：

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} %-5level [%thread][%logger{0}] %m%n</pattern>
    </encoder>
</appender> 
```

4、遇到if...else...时，每个分支首行都尽量打印日志

当你碰到if...else...或者switch这样的条件时，可以在分支的首行就打印日志，这样排查问题时，就可以通过日志，确定进入了哪个分支，代码逻辑更清晰，也更方便排查问题了。

```java
if(user.isVip()){
    log.info("该用户是会员,Id:{},开始处理会员逻辑",user,getUserId());
    // 会员逻辑
}else{
    log.info("该用户是非会员,Id:{},开始处理非会员逻辑",user,getUserId());
    // 非会员逻辑
}
```

5、日志级别比较低时，进行日志开关判断

对于trace/debug这些比较低的日志级别，必须进行日志级别的开关判断。

```java
User user = new User(666L, "公众号", "捡田螺的小男孩");
if (log.isDebugEnabled()) {
    log.debug("userId is: {}", user.getId());
}
```

因为当前有如下的日志代码：

```java
logger.debug("Processing trade with id: " + id + " and symbol: " + symbol);
```

如果配置的日志级别是warn的话，上述日志不会打印，但是会执行字符串拼接操作，如果symbol是对象， 还会执行toString()方法，浪费了系统资源，执行了上述操作，最终日志却没有打印，因此建议加日志开关判断。

6、不能直接使用日志系统（Log4j、Logback）中的 API，而是使用日志框架SLF4J中的API

SLF4J 是门面模式的日志框架，有利于维护和各个类的日志处理方式统一，并且可以在保证不修改代码的情况下，很方便的实现底层日志框架的更换。

```java
import org.slf4j.Logger; 
import org.slf4j.LoggerFactory;

private static final Logger logger = LoggerFactory.getLogger(TianLuoBoy.class);
```

7、建议使用参数占位{}，而不是用+拼接

反例：

```java
logger.info("Processing trade with id: " + id + " and symbol: " + symbol);
```

上面的例子中，使用`+`操作符进行字符串的拼接，有一定的**性能损耗**。

正例：

```java
logger.info("Processing trade with id: {} and symbol : {} ", id, symbol); 
```

我们使用了大括号`{}`来作为日志中的占位符，比于使用`+`操作符，更加优雅简洁。并且，**相对于反例**，使用占位符仅是替换动作，可以有效提升性能。

8、建议使用异步的方式来输出日志。

- 日志最终会输出到文件或者其它输出流中的，IO性能会有要求的。如果异步，就可以显著提升IO性能。
- 除非有特殊要求，要不然建议使用异步的方式来输出日志。以logback为例吧，要配置异步很简单，使用AsyncAppender就行

```xml
<appender name="FILE_ASYNC" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="ASYNC"/>
</appender>
```

9、不要使用e.printStackTrace()

反例：

```java
try {
    // 业务代码处理
} catch(Exception e) {
    e.printStackTrace();
}
```

正例：

```java
try {
    // 业务代码处理
} catch(Exception e) {
    log.error("你的程序有异常啦",e);
}
```

理由：

- e.printStackTrace()打印出的堆栈日志跟业务代码日志是交错混合在一起的，通常排查异常日志不太方便。
- e.printStackTrace()语句产生的字符串记录的是堆栈信息，如果信息太长太多，字符串常量池所在的内存块没有空间了,即内存满了，那么，用户的请求就卡住啦~



10、异常日志不要只打一半，要输出全部错误信息

反例1：

```java
try {
    //业务代码处理
} catch (Exception e) {
    // 错误
    LOG.error('你的程序有异常啦');
} 
```

> 异常e都没有打印出来，所以压根不知道出了什么类型的异常。

反例2：

```java
try {
    //业务代码处理
} catch (Exception e) {
    // 错误
    LOG.error('你的程序有异常啦', e.getMessage());
} 
```

> e.getMessage()不会记录详细的堆栈异常信息，只会记录错误基本描述信息，不利于排查问题。

正例：

```java
try {
    //业务代码处理
} catch (Exception e) {
    // 错误
    LOG.error('你的程序有异常啦', e);
} 
```



11、禁止在线上环境开启 debug

禁止在线上环境开启debug，这一点非常重要。因为一般系统的debug日志会很多，并且各种框架中也大量使用 debug的日志，线上开启debug不久可能会打满磁盘，影响业务系统的正常运行。



12、不要记录了异常，又抛出异常

反例如下：

```java
log.error("IO exception", e);
throw new MyException(e);
```

- 这样实现的话，通常会把栈信息打印两次。这是因为捕获了MyException异常的地方，还会再打印一次。
- 这样的日志记录，或者包装后再抛出去，不要同时使用！否则你的日志看起来会让人很迷惑。



13、避免重复打印日志

避免重复打印日志，酱紫会浪费磁盘空间。如果你已经有一行日志清楚表达了意思，**避免再冗余打印**，反例如下：

```java
if (user.isVip()) {
    log.info("该用户是会员,Id:{}",user,getUserId());
    // 冗余，可以跟前面的日志合并一起
    log.info("开始处理会员逻辑,id:{}",user,getUserId());
    // 会员逻辑
} else {
    // 非会员逻辑
}
```

如果你是使用log4j日志框架，务必在`log4j.xml`中设置 additivity=false，因为可以避免重复打印日志。正例：

```xml
<logger name="com.taobao.dubbo.config" additivity="false"> 
```



14、日志文件分离

- 我们可以把不同类型的日志分离出去，比如access.log，或者error级别error.log，都可以单独打印到一个文件里面。
- 当然，也可以根据不同的业务模块，打印到不同的日志文件里，这样我们排查问题和做数据统计的时候，都会比较方便啦。

15、核心功能模块，建议打印较完整的日志

- 我们日常开发中，如果核心或者逻辑复杂的代码，建议添加详细的注释，以及较详细的日志。
- 日志要多详细呢？脑洞一下，如果你的核心程序哪一步出错了，通过日志可以定位到，那就可以啦。



## 6、日志分析工具汇总

> 运维利器：9个Java日志分析工具：https://mp.weixin.qq.com/s/cvnA_HiglxAd5x6XtmnOIQ

1. Apache Log4j：Java领域资格最老，应用最广的日志工具
   - 官网：https://logging.apache.org/log4j/2.x/
   - Log4j是apache的一个开源项目，创始人Ceki Gulcu。Log4j 从诞生之日到现在一直广受业界欢迎。Log4j是高度可配置的，并可通过在运行时的外部文件配置。它根据记录的优先级别，并提供机制，以指示记录信息到许多的目的地，诸如：数据库，文件，控制台，UNIX系统日志等。
2. Graylog：适合扩展角色和权限管理的开源聚合器
   - 官网：https://www.graylog.org/
   - Graylog是强大的日志管理、分析工具。它基于 Elasticsearch, Java和MongoDB。Graylog可以收集监控多种不同应用的日志。但是为了示范说明，我只收集syslog。并且，我将会把用到的组件全部安装到一个单独的服务器上。对于大型、生产系统你可以把组件分开安装在不同的服务器上，这样可以提高效率
3. Kibana：分析和可视化日志文件。 某些功能需要付款
   - 官网：https://www.elastic.co/products/kibana
   - Kibana是一个开源的分析和可视化平台，设计用于和Elasticsearch一起工作。你用Kibana来搜索，查看，并和存储在Elasticsearch索引中的数据进行交互。你可以轻松地执行高级数据分析，并且以各种图标、表格和地图的形式可视化数据。可谓「一张图片胜过千万行日志」
4. Logback：通过Groovy有趣的配置选项的强大的日志库
   - 官网：https://logback.qos.ch/
   - Logback是由log4j创始人Ceki Gulcu设计的又一个开源日记组件，目标是替代log4j
5. Logbook：可扩展的开源库，用于HTTP请求和响应日志记录
   - 官网：https://github.com/zalando/logbook
   - Logbook是一个可扩展的Java库，可为不同的客户端和服务器端技术提供完整的请求和响应日志记录。 它满足了以下特殊需求：a）允许Web应用程序开发人员记录应用程序接收或发送的任何HTTP流量；b），以便以后易于持久化和分析。 这对于传统的日志分析，满足审计要求或调查各个历史流量问题非常有用
6. Logstash：用于管理日志文件的工具
   - 官网：https://www.elastic.co/products/logstash
7. SLF4J：与实现一起使用的抽象层
   - 官网：https://www.slf4j.org/
   - 作者又是 Ceki Gulcu！这位大神写了Log4j、Logback和slf4j，专注日志组件开发五百年，一直只能超越自己。用作各种日志框架（例如java.util.logging，logback，log4j）的简单外观或抽象，允许最终用户在部署时插入所需的日志记录框架
8. tinylog：具有静态记录器类的轻量级日志框架
   - 官网：https://tinylog.org/
   - tinylog是一个用于Java和Android的轻量级开源日志记录框架，针对易用性进行了优化
9. Tracer：在分布式系统中调用跟踪和日志关联
   - 官网：https://github.com/zalando/tracer
   - Tracer是一个管理自定义跟踪标识符并通过分布式系统进行管理的库



## 7、参考资料 & 鸣谢

- 全网最全、最细致的Java日志框架以及门面技术【CSDN Java学术趴】https://blog.csdn.net/chengbaobao520/article/details/120218849
- java日志框架JUL、JCL、Slf4j、Log4j、Log4j2、Logback 一网打尽【CSDN java叶新东老师】https://yexindong.blog.csdn.net/article/details/122010178



# JUL日志框架

## 1、JUL 简单介绍

JUL全称Java util logging是Java原生的日志框架，使用时不需要另外引用第三方类库，相对其他日志框 架使用方便，学习简单，能够在小型应用中灵活使用。

在JUL中有以下组件：

1. Loggers：被称为记录器，应用程序通过获取Logger对象，调用其API来来发布日志信息。Logger通常时应用程序访问日志系统的入口程序
2. Handlers：也被称为Appenders，每个Logger都会关联一组Handlers，Logger会将日志交给关联的Handler处理，由Handler负责将日志做记录。Handlers在此是一个抽象，其具体的实现决定了日志记录的位置可以是控制台、文件、网络上的其他日志服务或操作系统日志等
3. Formatters：也被称为Layouts，它负责对日志事件中的数据进行转换和格式化。它决定了数据在一条日志记录中的最终形式
4. Level：日志的输出级别，每条日志消息都有一个关联的级别。我们根据输出级别的设置，来展现最终所呈现的日志信息，根据不同的需求设置不同级别
5. Filters：过滤器，根据需要定制哪些信息会被记录，哪些信息会被放过

**总结一下就是：**

用户使用Logger来进行日志记录，Logger持有若干个Handler，日志的输出操作是由Handler完成的。 在Handler在输出日志前，会经过Filter的过滤，判断哪些日志级别过滤放行哪些拦截，Handler会将日志内容输出到指定位置（日志文件、控制台等）。Handler在输出日志时会使用Layout，将输出内容进行排版。

JUC 流程示意图：

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│             LogManager─┐     Level            Filter         Formatter               │
│                        ⬇       ⬇                 ⬇               ⬇                   │
│  Application-------->Logger-------->LogRecord-------->Handler-------->Outside World  │
└──────────────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────────────┐
│    应用程序    运行  日志记录器    记录日志   日志处理器     处理日志                      │
│  Application-------->Logger---------------->Handler------------------>Outside World  │
│                         ⬇                     ⬇                                      │
│   ┌──────────────────────────────────────────────────────────────┐                   │
│   │Filters(日志过滤器), Formatters(日志格式化组件), Level日志输出级别│                   │
│   └──────────────────────────────────────────────────────────────┘                   │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

JUC 总体流程：

1. 初始化LogManager，LogManager加载logging.properties配置，添加Logger到LogManager中
2. 从单例LogManager获取Logger
3. 设置级别Level，并指定日志记录LogRecord
4. Filter提供了日志级别之外更细粒度的控制
5. Handler是用来处理日志输出位置
6. Formatter是用来格式化LogRecord的

JUL 日志原理解析：

1. Logger：是用来记录日志的，负责调用其有的Handler进行日志输出
2. LogRecord：是用来传递日志的，一条具体的日志对象，比如这条日志所在线程、日志级别，日志名称，日志内容，时间等信息
3. Handler：是用来导出（处理）日志的，负责具体的日志输出，有不同的实现类，比如（ConsoleHandler 控制台输出，FileHandler 文件输出）
4. Level：是用来定义日志级别的（控制日志输出）
5. Filter：提供了日志级别之外更细粒度的控制
6. Formatter：是用来格式化LogRecord的，Handler在输出前会调用Formatter 对LogRecord 进行格式化，如简单字符串格式化，Html格式化，XML格式化等
7. 此外还有一个单一的全局的LogManager维护Logger和管理日志（LogManager 负责创建日志对象，并将创建的日志对象继承RootLogger的默认配置）。JUL主要由这七个核心类或接口组成，再有就是一些子类或者实现类。API中关于这几个类或接口的类和方法的详细介绍这里不再复述。现在我们已经对JUL有了一个整体的了解，并且对核心类或接口的功能职责也有了初步了解，接下就该结合代码看看JUL的真面目。

> 在任何日志框架中都会有继承性的概念，在获取Logger对象时，默认会继承RootLogger的配置，但自己的Logger也可以单独设置级别，Handler等。但是如果日志名称是包名进行命名的话，子包默认会继承父包的配置，例如 指定了com.a.b这个包的Handler是ConsoleHandler,Level是info,则com.a.b.c或者 com.a.b.c.d等属于这个包的Logger都会默认继承这个包的配置



## 2、JUL 入门案例

```java
import java.util.logging.Level;
import java.util.logging.Logger;

public class Test {
    //1、获取日志记录器对象
    public static Logger logger = Logger.getLogger(Test.class.getName());
    public static void main(String[] args) {
        /**
         * 对于日志输出有两种方式
         *   1.一种是指定日志级别方法输出(支持占位符输出)
         *   2.另一种是通用方法输出，需要参数指定日志级别(不支持占位符输出)
         */
        // 1.通用方法输出，需要参数指定日志级别
        logger.log(Level.SEVERE, "this is a severe");
        logger.log(Level.WARNING, "this is a warn");
        logger.log(Level.INFO, "this is a info"); // JUL默认日志级别,比info高的输出,比info低的过滤掉
        logger.log(Level.CONFIG, "this is a config");
        logger.log(Level.FINE, "this is a fine");
        logger.log(Level.FINER, "this is a finer");
        logger.log(Level.FINEST, "this is a finest");
        logger.log(Level.INFO, "this is a info", new Exception("My Exception")); // 带异常输出

        // 2.指定日志级别方法输出,有不同的级别,可带参数,其它类似
        logger.severe("severe log");
        logger.warning("warning log");
        logger.info("info log"); // JUL默认日志级别,比info高的输出,比info低的过滤掉
        logger.config("config log");
        logger.fine("fine log");
        logger.finer("finer log");
        logger.finest("finest log");

        // 3.占位符输出, 只有.log()方法是支持占位符输出的
        logger.log(Level.WARNING, "this is a warn, {0} ", "p1");
        logger.log(Level.INFO, "this is a info, {0} {1}", new Object[]{"p1","p2"});
    }
}

// 输出内容
3月 26, 2022 11:21:58 上午 Test main
严重: this is a severe
3月 26, 2022 11:21:58 上午 Test main
警告: this is a warn
3月 26, 2022 11:21:58 上午 Test main
信息: this is a info
3月 26, 2022 11:21:58 上午 Test main
信息: this is a info
java.lang.Exception: My Exception
	at Test.main(Test.java:21)

3月 26, 2022 11:21:58 上午 Test main
严重: severe log
3月 26, 2022 11:21:58 上午 Test main
警告: warning log
3月 26, 2022 11:21:58 上午 Test main
信息: info log
3月 26, 2022 11:21:58 上午 Test main
警告: this is a warn, p1 
3月 26, 2022 11:21:58 上午 Test main
信息: this is a info, p1 p2
```

因为默认的日志级别是INFO，所以比INFO级别低的 CONFIG、FINE、FINER、FINEST的日志记录消息没有记录。



## 3、JUL 组件介绍

JUL常用组件如下：

- Level：日志级别
- Handler：日志输出地
- Formatter：日志格式化



### 1、Level（日志级别）

日志级别由高到低：OFF/SEVERE/WARNIN/INFO/CONFIG/FINE/FINERG/FINEST/ALL，每个级别有自己的数值，在java.util.logging.Level类中源码如下：

```java
public static final Level OFF = new Level("OFF",Integer.MAX_VALUE, defaultBundle);
public static final Level SEVERE = new Level("SEVERE",1000, defaultBundle);
public static final Level WARNING = new Level("WARNING", 900, defaultBundle);
public static final Level INFO = new Level("INFO", 800, defaultBundle);
public static final Level CONFIG = new Level("CONFIG", 700, defaultBundle);
public static final Level FINE = new Level("FINE", 500, defaultBundle);
public static final Level FINER = new Level("FINER", 400, defaultBundle);
public static final Level FINEST = new Level("FINEST", 300, defaultBundle);
public static final Level ALL = new Level("ALL", Integer.MIN_VALUE, defaultBundle);
```

- Level.SEVERE：最高级别的日志，主要记录错误信息
- Level.WARNING：级别排行第二，记录警告信息
- Level.INFO：级别排行第三，最常用的日志级别，记录普通消息，比如请求信息、连接信息、参数信息等等
- Level.CONFIG：级别排行第四，记录配置信息，加载配置文件、读取配置参数都可以使用CONFIG记录
- Level.FINE ：debug 时记录的 日志消息，记录运行时的状态、传递的参数等等
- Level.FINER：debug 时记录的 日志消息，记录运行时的状态、传递的参数等等
- Level.FINEST：debug 时记录的 日志消息，记录运行时的状态、传递的参数等等
- Level.OFF： 关闭日志记录
- Level.ALL：所有记录都开启

可自定义日志级别：继承java.util.logging.Level类即可



### 2、Handler（日志输出）

日志输出的目的，java.util.logging包中有如下四种（ConsoleHandler、FileHandler、SocketHandler、MemoryHandler）：

1. StreamHandler（父类）：
   1. ConsoleHandler：输出到控制台，默认Level为INFO，流为System.err，Formatter为SimpleFormatter
   2. FileHandler：输出到文件，默认Level为INFO，Formatter为XMLFormatter
   3. SocketHandler：输出到socket
2. MemoryHandler：输出到内存

可自定义日志输出：继承java.util.logging.Handler类即可



### 3、Formatter（日志格式化）

定义日志输出的格式，目前有SimpleFormatter和XMLFormatter两种格式，分别对应简单格式和xml格式。

注意：默认情况下：ConsoleHandler对应SimpleFormatter，FileHandler对应XMLFormatter。

可自定输出格式：继承抽象类java.util.logging.Formatter即可



## 4、自定义日志输出

### 1、自定义输出日志级别（控制台）

Java默认的只能输出3个级别，并且默认的还是输出到控制台。我们现在想要输出其他多个级别，只能自定义输出级别了。自定义的时候，先要关闭默认的。

```java
import java.util.logging.ConsoleHandler;
import java.util.logging.Level;
import java.util.logging.Logger;
import java.util.logging.SimpleFormatter;

public class Test {
    public static void main(String[] args) {
        // 1.获取日志记录器对象
        Logger logger = Logger.getLogger(Test.class.getName());
        // 2.关闭系统默认配置（需要自定义日志配置必须关闭）
        logger.setUseParentHandlers(false);

        // 3.自定义配置日志级别
        // 创建ConsoleHandler控制台输出
        ConsoleHandler consoleHandler = new ConsoleHandler();
        // 创建简单格式转换对象
        SimpleFormatter simpleFormatter = new SimpleFormatter();
        // 进行关联
        consoleHandler.setFormatter(simpleFormatter);
        logger.addHandler(consoleHandler);
        // 配置日志具体级别,注意:Logger与Handler都需要配置
        logger.setLevel(Level.ALL);
        consoleHandler.setLevel(Level.ALL);

        // 4.日志记录输出,可以发现设置级别为全部输出后,可以在控制台看到所有输出了
        logger.severe("severe");
        logger.warning("warning");
        logger.info("info");
        logger.config("config");
        logger.fine("fine");
        logger.finer("finer");
        logger.finest("finest");
    }
}

// 输出内容
3月 26, 2022 11:56:15 上午 Test main
严重: severe
3月 26, 2022 11:56:15 上午 Test main
警告: warning
3月 26, 2022 11:56:15 上午 Test main
信息: info
3月 26, 2022 11:56:15 上午 Test main
配置: config
3月 26, 2022 11:56:15 上午 Test main
详细: fine
3月 26, 2022 11:56:15 上午 Test main
较详细: finer
3月 26, 2022 11:56:15 上午 Test main
非常详细: finest
```



### 2、自定义日志输出到文件（磁盘）

```java
import java.io.IOException;
import java.util.logging.*;

public class Test {
    public static void main(String[] args) throws IOException {
        // 1.获取日志记录器对象
        Logger logger = Logger.getLogger(Test.class.getName());
        // 2.关闭系统默认配置（需要自定义日志配置必须关闭）
        logger.setUseParentHandlers(false);

        // 自定义配置日志Level、Handler、Formatter
        /**
         * 创建ConsoleHandler控制台输出
         * 并且设置为SimpleFormatter简单格式转换对象(默认就是SimpleFormatter)
         * 日志输出级别为ALL全部级别
         */
        ConsoleHandler consoleHandler = new ConsoleHandler();
        consoleHandler.setFormatter(new SimpleFormatter());
        logger.addHandler(consoleHandler);
        // 配置Handler日志具体级别
        consoleHandler.setLevel(Level.ALL);

        /**
         * 创建FileHandler文件输出
         * 并且设置为SimpleFormatter简单格式转换对象(默认是XMLFormatter)
         * 日志输出级别为ALL全部级别
         */
        // 第二个参数是日志输出内容追加，注意:目录必须提前存在,否则会抛异常
        FileHandler fileHandler = new FileHandler("./logs/jul.log", true);
        fileHandler.setFormatter(new SimpleFormatter());
        logger.addHandler(fileHandler);
        // 配置Handler日志具体级别
        fileHandler.setLevel(Level.ALL);

        // 配置Logger日志输出级别
        logger.setLevel(Level.ALL);

        // 3.日志记录输出,以下级别从大到小
        logger.severe("severe");
        logger.warning("warning");
        logger.info("info");
        logger.config("config");
        logger.fine("fine");
        logger.finer("finer");
        logger.finest("finest");
    }
}

// 输出内容（文件磁盘中也是同样的输出）
3月 26, 2022 12:27:32 下午 Test main
严重: severe
3月 26, 2022 12:27:32 下午 Test main
警告: warning
3月 26, 2022 12:27:32 下午 Test main
信息: info
3月 26, 2022 12:27:32 下午 Test main
配置: config
3月 26, 2022 12:27:32 下午 Test main
详细: fine
3月 26, 2022 12:27:33 下午 Test main
较详细: finer
3月 26, 2022 12:27:33 下午 Test main
非常详细: finest
```

以上创建了两个Handler，一个是控制台的，一个是文件的，把这个Handler设置到日志记录器对象里面。以后的日志不仅可以输出控制台，还可以输出到文件

可以把文件输出Formatter改成默认的XMLFormatter，观察文件内容

```java
FileHandler fileHandler = new FileHandler("./logs/jul.log", true);
fileHandler.setFormatter(new XMLFormatter()); // 这行代码可写可不写,因为FileHandler默认就是XMLFormatter
```

输出到文件的内容：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE log SYSTEM "logger.dtd">
<log>
<record>
  <date>2022-03-26T04:33:02.178157900Z</date>
  <millis>1648269182178</millis>
  <nanos>157900</nanos>
  <sequence>0</sequence>
  <logger>Test</logger>
  <level>SEVERE</level>
  <class>Test</class>
  <method>main</method>
  <thread>1</thread>
  <message>severe</message>
</record>
<record>
  <date>2022-03-26T04:33:02.228169900Z</date>
  <millis>1648269182228</millis>
  <nanos>169900</nanos>
  <sequence>1</sequence>
  <logger>Test</logger>
  <level>WARNING</level>
  <class>Test</class>
  <method>main</method>
  <thread>1</thread>
  <message>warning</message>
</record>
<record>
  <date>2022-03-26T04:33:02.229171100Z</date>
  <millis>1648269182229</millis>
  <nanos>171100</nanos>
  <sequence>2</sequence>
  <logger>Test</logger>
  <level>INFO</level>
  <class>Test</class>
  <method>main</method>
  <thread>1</thread>
  <message>info</message>
</record>
<record>
  <date>2022-03-26T04:33:02.232172300Z</date>
  <millis>1648269182232</millis>
  <nanos>172300</nanos>
  <sequence>3</sequence>
  <logger>Test</logger>
  <level>CONFIG</level>
  <class>Test</class>
  <method>main</method>
  <thread>1</thread>
  <message>config</message>
</record>
<record>
  <date>2022-03-26T04:33:02.233171500Z</date>
  <millis>1648269182233</millis>
  <nanos>171500</nanos>
  <sequence>4</sequence>
  <logger>Test</logger>
  <level>FINE</level>
  <class>Test</class>
  <method>main</method>
  <thread>1</thread>
  <message>fine</message>
</record>
<record>
  <date>2022-03-26T04:33:02.233171500Z</date>
  <millis>1648269182233</millis>
  <nanos>171500</nanos>
  <sequence>5</sequence>
  <logger>Test</logger>
  <level>FINER</level>
  <class>Test</class>
  <method>main</method>
  <thread>1</thread>
  <message>finer</message>
</record>
<record>
  <date>2022-03-26T04:33:02.234172500Z</date>
  <millis>1648269182234</millis>
  <nanos>172500</nanos>
  <sequence>6</sequence>
  <logger>Test</logger>
  <level>FINEST</level>
  <class>Test</class>
  <method>main</method>
  <thread>1</thread>
  <message>finest</message>
</record>
</log>
```



## 5、Logger之间的父子关系

JUL中Logger之间存在父子关系，这种父子关系通过树状结构存储，JUL在初始化时会创建一个顶层RootLogger作为所有Logger父Logger，存储上作为树状结构的根节点。并父子关系通过路径来关联。默认子Logger会继承父Logger的属性。

```java
import java.io.IOException;
import java.util.logging.*;

public class Test {
    public static void main(String[] args) throws IOException {
        // 1.获取日志记录器对象, logger1 是 logger2 的父对象
        Logger logger1 = Logger.getLogger("com");
        Logger logger2 = Logger.getLogger("com.controller");

        // 2.测试
        System.out.println(logger1 == logger2.getParent()); // true
        // 所有日志记录器的顶级父元素 LogManager$RootLogger，name 为  ""
        System.out.println("logger1 Parent:"+logger1.getParent() + " ,name:" + logger1.getParent().getName());
        // 打印logger2的父类logger1的对象与名称
        System.out.println("logger2 Parent:"+logger2.getParent() + " ,name:" + logger2.getParent().getName());

        // 3.logger1父类：关闭默认配置, 并自定义配置日志级别和设置ConsoleHandler控制台输出
        logger1.setUseParentHandlers(false);
        ConsoleHandler consoleHandler = new ConsoleHandler();
        consoleHandler.setFormatter(new SimpleFormatter());
        consoleHandler.setEncoding("UTF-8");
        consoleHandler.setLevel(Level.ALL);
        logger1.addHandler(consoleHandler);
        logger1.setLevel(Level.ALL);

        // 4.使用logger2子类输出（查看是否继承父类的属性）
        logger2.severe("severe");
        logger2.warning("warning");
        logger2.info("info");
        logger2.config("config");
        logger2.fine("fine");
        logger2.finer("finer");
        logger2.finest("finest");
    }
}

// 输出内容
true
logger1 Parent:java.util.logging.LogManager$RootLogger@4b9e13df ,name:
logger2 Parent:java.util.logging.Logger@2b98378d ,name:com
3月 26, 2022 12:57:31 下午 Test main
严重: severe
3月 26, 2022 12:57:31 下午 Test main
警告: warning
3月 26, 2022 12:57:31 下午 Test main
信息: info
3月 26, 2022 12:57:31 下午 Test main
配置: config
3月 26, 2022 12:57:31 下午 Test main
详细: fine
3月 26, 2022 12:57:31 下午 Test main
较详细: finer
3月 26, 2022 12:57:31 下午 Test main
非常详细: finest
```

从执行结果可以看出子类默认继承了父类Logger的配置。

注意：如果子类不想或者不需要继承父类Logger的配置，只需要关闭子类Logger默认配置，然后重新自定配置即可。

```java
import java.io.IOException;
import java.util.logging.*;

public class Test {
    public static void main(String[] args) throws IOException {
        // 1.获取日志记录器对象, logger1 是 logger2 的父对象
        Logger logger1 = Logger.getLogger("com");
        Logger logger2 = Logger.getLogger("com.controller");

        // 2.logger1父类：关闭默认配置, 并自定义配置日志级别和设置ConsoleHandler控制台输出
        logger1.setUseParentHandlers(false);
        ConsoleHandler consoleHandler = new ConsoleHandler();
        consoleHandler.setFormatter(new SimpleFormatter());
        consoleHandler.setEncoding("UTF-8");
        consoleHandler.setLevel(Level.ALL);
        logger1.addHandler(consoleHandler);
        logger1.setLevel(Level.ALL);

        // 3.使用logger2子类输出（查看是否继承父类的属性）
        System.out.println("===================如下是继承父类配置的输出==========================");
        log(logger2);

        // 4.重写自定义logger2的日志配置（不想使用logger1父类继承的配置）
        // 注意: 如果没有取消默认配置,那么当前又设置了子类的属性,logs输出会输出2次重复的,因为继承父类输出1次,子类自己输出1次
        logger2.setUseParentHandlers(false);
        ConsoleHandler consoleHandler2 = new ConsoleHandler();
        consoleHandler.setLevel(Level.WARNING);
        logger2.addHandler(consoleHandler2);
        logger2.setLevel(Level.WARNING);

        // 4.使用logger2子类输出（查看是否继承父类的属性）
        System.out.println("===================如下是没有继承父类配置的输出==========================");
        log(logger2);
    }

    private static void log(Logger logger2) {
        logger2.severe("severe");
        logger2.warning("warning");
        logger2.info("info");
        logger2.config("config");
        logger2.fine("fine");
        logger2.finer("finer");
        logger2.finest("finest");
    }
}

// 输出内容
===================如下是继承父类配置的输出==========================
3月 26, 2022 1:06:48 下午 Test log
严重: severe
3月 26, 2022 1:06:48 下午 Test log
警告: warning
3月 26, 2022 1:06:48 下午 Test log
信息: info
3月 26, 2022 1:06:48 下午 Test log
配置: config
3月 26, 2022 1:06:48 下午 Test log
详细: fine
3月 26, 2022 1:06:48 下午 Test log
较详细: finer
3月 26, 2022 1:06:48 下午 Test log
非常详细: finest
===================如下是没有继承父类配置的输出==========================
3月 26, 2022 1:06:48 下午 Test log
严重: severe
3月 26, 2022 1:06:48 下午 Test log
警告: warning
```



## 6、JUL自定义配置文件

以上所有配置相关操作都是以Java硬编码的形式进行的的，我们可以使用更加清晰，更加专业的一种做法，就是使用配置文件，如果我们没有自己添加配置文件，则会使用系统（JDK）默认的配置文件。默认配置文件路径`$JAVAHOME\jre\lib\logging.properties`，默认配置文件内容如下

```properties
############################################################
#  	Default Logging Configuration File
#
# You can use a different file by specifying a filename
# with the java.util.logging.config.file system property.  
# For example java -Djava.util.logging.config.file=myfile
############################################################

############################################################
#  	Global properties
############################################################

# "handlers" specifies a comma separated list of log Handler 
# classes.  These handlers will be installed during VM startup.
# Note that these classes must be on the system classpath.
# By default we only configure a ConsoleHandler, which will only
# show messages at the INFO and above levels.
handlers= java.util.logging.ConsoleHandler

# To also add the FileHandler, use the following line instead.
#handlers= java.util.logging.FileHandler, java.util.logging.ConsoleHandler

# Default global logging level.
# This specifies which kinds of events are logged across
# all loggers.  For any given facility this global level
# can be overriden by a facility specific level
# Note that the ConsoleHandler also has a separate level
# setting to limit messages printed to the console.
.level= INFO

############################################################
# Handler specific properties.
# Describes specific configuration info for Handlers.
############################################################

# default file output is in user's home directory.
java.util.logging.FileHandler.pattern = %h/java%u.log
java.util.logging.FileHandler.limit = 50000
java.util.logging.FileHandler.count = 1
java.util.logging.FileHandler.formatter = java.util.logging.XMLFormatter

# Limit the message that are printed on the console to INFO and above.
java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

# Example to customize the SimpleFormatter output format 
# to print one-line log message like this:
#     <level>: <log message> [<date/time>]
#
# java.util.logging.SimpleFormatter.format=%4$s: %5$s [%1$tc]%n

############################################################
# Facility specific properties.
# Provides extra control for each logger.
############################################################

# For example, set the com.xyz.foo logger to only log SEVERE
# messages:
com.xyz.foo.level = SEVERE
```

自定义配置文件：logging.properties。将该文件保存至resources目录下

```properties
# RootLogger顶级父元素指定的默认处理器,多个用逗号分隔
# handlers = java.util.logging.FileHandler,java.util.logging.ConsoleHandler
handlers = java.util.logging.ConsoleHandler
# RootLogger顶级父元素默认的日志级别为：ALL
.level = ALL

# 文件处理器属性设置
# 输出日志文件的路径
java.util.logging.FileHandler.pattern = ./logs/java%u.log
# 输出日志文件的限制(50000字节)
java.util.logging.FileHandler.limit = 50000
# 设置日志文件的数量
java.util.logging.FileHandler.count = 1
# 设置输出日志的格式,默认是以XML方式输出的
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter
# 指定handler对象的日志级别,默认是INFO级别
java.util.logging.FileHandler.level = ALL
# 指定义追加的方式添加日志内容
java.util.logging.FileHandler.append = true

# 控制台处理器属性设置
# 控制台默认输出的格式
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter
# 指定handler对象的字符集编码方式
java.util.logging.ConsoleHandler.encoding = UTF-8
# 控制台输出默认的级别设置(默认是INFO)
java.util.logging.ConsoleHandler.level = ALL

# 指定日志消息格式
#设置日志格式
# java.util.logging.SimpleFormatter.format= %1$tc %2$s%n%4$s: %5$s%6$s%n
java.util.logging.SimpleFormatter.format = %4$s: %5$s [%1$tc]%n

# 自定义Logger使用：包名 + handlers/level
com.xyz.handlers = java.util.logging.ConsoleHandler,java.util.logging.FileHandler
com.xyz.level = SEVERE
# 必须关闭默认设置(只有关闭该配置,自定义设置才能生效,否则还会使用RootLogger的属性配置)
com.xyz.useParentHandlers = false
```

加载自定义配置文件代码：

```java
package com.xyz;

import java.io.InputStream;
import java.util.logging.LogManager;
import java.util.logging.Logger;

public class Test {
    public static void main(String[] args) throws Exception {
        // 读取配置文件，通过类加载器
        InputStream ins = Test.class.getClassLoader().getResourceAsStream("logging.properties");
        // 创建LogManager
        LogManager logManager = LogManager.getLogManager();
        // 通过LogManager加载配置文件
        logManager.readConfiguration(ins);

        // 创建日志记录器(当前Logger使用的配置是RootLogger的)
        Logger logger = Logger.getLogger("com");
        logger.severe("severe");
        logger.warning("warning");
        logger.info("info");
        logger.config("config");
        logger.fine("fine");
        logger.finer("finer");
        logger.finest("finest");

        // 创建日志记录器(当前Logger使用的是自己自动配置的, Level配置为severe)
        Logger logger2 = Logger.getLogger("com.xyz");
        logger2.severe("severe test");
        logger2.warning("warning test");
        logger2.info("info test");
        logger2.config("config test");
        logger2.fine("fine test");
        logger2.finer("finer test");
        logger2.finest("finest test");
    }
}

// 输出内容
严重: severe [周六 3月 26 14:57:57 CST 2022]
警告: warning [周六 3月 26 14:57:58 CST 2022]
信息: info [周六 3月 26 14:57:58 CST 2022]
配置: config [周六 3月 26 14:57:58 CST 2022]
详细: fine [周六 3月 26 14:57:58 CST 2022]
较详细: finer [周六 3月 26 14:57:58 CST 2022]
非常详细: finest [周六 3月 26 14:57:58 CST 2022]
严重: severe test [周六 3月 26 14:57:58 CST 2022]
```



## 7、JUL自定义组件方式

这里通过继承Level、Handler、Formatter自定义Logger，日志存在文件中，且每个不同的loggerName存不同日志文件中。示例如下，解释请查看注释：

```java
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.logging.*;

public class Test {
    /**
     * 自定义日志level
     */
    public static class SelfLevel extends Level {
        public static SelfLevel ERROR = new SelfLevel("ERROR", 910);

        /**
         * @param name  自定义级别名称
         * @param value 自定义级别的权重值
         */
        protected SelfLevel(String name, int value) {
            super(name, value);
        }
    }

    /**
     * 自定义日志格式formatter
     */
    public static class SelfFormatter extends Formatter {
        @Override
        public String format(LogRecord record) {
            return String.format("[%s] %s %s.%s: %s\r\n",
                    LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")),
                    record.getLevel().getName(),
                    record.getSourceClassName(),
                    record.getSourceMethodName(),
                    record.getMessage());
        }
    }

    /**
     * 自定义日志输出handler
     */
    public static class SelfHandle extends Handler {
        /**
         * 日志写入的位置
         */
        @Override
        public void publish(LogRecord record) {
            try {
                // 每一个不同的loggerName分别记在不同的日志文件中
                File file = new File(String.format("%s_%s.log", record.getLoggerName(),
                        LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd"))));
                if (!file.exists()) {
                    file.createNewFile();
                }
                FileWriter fw = new FileWriter(file, true);
                PrintWriter pw = new PrintWriter(fw);
                String log = String.format("[%s] %s %s.%s: %s\r\n",
                        LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")),
                        record.getLevel().getName(),
                        record.getSourceClassName(),
                        record.getSourceMethodName(),
                        record.getMessage());
                pw.write(log);
                pw.flush();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }

        /**
         * 刷新缓存区，保存数据
         */
        @Override
        public void flush() {
        }

        /**
         * 关闭
         */
        @Override
        public void close() throws SecurityException {
        }
    }

    public static void main(String[] args) {
        // 创建日志记录器
        Logger logger = Logger.getLogger(Test.class.getName());
        // 生成日志输出handler实例
        logger.addHandler(new SelfHandle());
        logger.info("info log");
        logger.severe("severe log");
        //使用自定义的logger level
        logger.log(SelfLevel.ERROR, "error log");
        
        System.out.println("------------------------");
        Logger log = Logger.getLogger("com.test");
        log.setUseParentHandlers(false);
        ConsoleHandler console = new ConsoleHandler();
        console.setFormatter(new SelfFormatter());
        log.addHandler(console);
        log.log(Level.INFO, "selfFormatter test");
    }
}
```

控制台输出内容如下：

```
3月 27, 2022 6:19:11 下午 Test main
信息: info log
3月 27, 2022 6:19:11 下午 Test main
严重: severe log
3月 27, 2022 6:19:11 下午 Test main
ERROR: error log
------------------------
[2022-03-27 18:19:11] INFO Test.main: selfFormatter test
```

在项目同级目录会生产文件Test_2022-03-27.log

```java
[2022-03-27 18:19:11] INFO Test.main: info log
[2022-03-27 18:19:11] SEVERE Test.main: severe log
[2022-03-27 18:19:11] ERROR Test.main: error log
```



## 8、Logger private static final 为啥？

为什么将 Logger 对象声明为 private static final：

1. private：是为了防止其他类使用当前类的日志对象
2. static：是为了让每个类中的日志对象只生成一份，是属于类的，不是属于具体的实例的
3. final：是为了避免日志对象在运行时被修改



## 9、参考文献 & 鸣谢

- java日志：一、JUL使用【CSDN：小徐也要努力鸭】https://blog.csdn.net/a232884c/article/details/121103424
- 日志技术之JUL入门【CSDN：StudyWinter】https://blog.csdn.net/Zhouzi_heng/article/details/107943477
- java日志(二)–java官方日志jul的使用【CSDN：panda-star】https://blog.csdn.net/chinabestchina/article/details/104730196



# Log4j日志框架

## 1、Log4j 简单入门

### 1、简单介绍

Log4j 官方网站： http://logging.apache.org/log4j/1.2/、Maven仓库：https://search.maven.org/search?q=g:log4j

1. Log4j =》Log for java，从时间上讲，Log4j 的产生时间要比JUL早
2. Log4j 很重要的一个缺陷是不支持占位符，但是用户可自己定义实现占位符的功能
3. 从官网或者Maven仓库可以发现 Log4j 最后的版本是1.2.17

Log4j是Apache下的一款开源日志框架，通过使用Log4j，可以控制日志信息输出到控制台、文件、甚至是数据库中。我们可以控制每条日志的输出格式，通过定义日志的输出级别，可以更灵活的控制日志的输出过程，方便项目调试。这些设置可以通过一个配置文件来灵活的进行配置，而不需要修改应用的代码。



### 2、入门案例

1、建立maven工程并添加依赖

```xml
<!--https://search.maven.org/search?q=g:log4j-->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

2、默认直接获取Logger对象打印日志，可以发现没有打印我们想要的内容 （是由于我们什么都没有配置，并且Log4j也没有默认配置）

```java
import org.apache.log4j.Logger;
public class Log4jTest {
    public static void main(String[] args) {
        /**
         * Log4j获取日志记录器对象的两个方法:
         *   public static Logger getLogger(String name);
         *   public static Logger getLogger(Class clazz);
         */
        Logger logger = Logger.getLogger(Log4jTest.class);
        logger.info("hello log4j");
    }
}

// 输出内容
log4j:WARN No appenders could be found for logger (Log4jTest).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

> 注意：如果既没有Java硬编码配置，又没有使用配置文件配置日志输出。默认是没有任何配置，不会输出任何内容

3、使用Log4j提供的初始化默认配置方式

```java
import org.apache.log4j.BasicConfigurator;
import org.apache.log4j.Logger;

public class Log4jTest {
    public static void main(String[] args) {
        // 初始化配置信息，在入门案例中暂不使用配置文件
        BasicConfigurator.configure();
        /**
         * Log4j获取日志记录器对象的两个方法:
         *   public static Logger getLogger(String name);
         *   public static Logger getLogger(Class clazz);
         */
        Logger logger = Logger.getLogger(Log4jTest.class);
        logger.fatal("fatal test"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
        logger.error("error test"); // 记录错误信息，不会影响系统运行
        logger.warn("warn test"); // 记录警告信息
        logger.info("info test"); // 记录运行信息
        logger.debug("debug test"); // （默认）记录调式信息，一般在开发中使用
        logger.trace("trace test"); // 记录追踪信息，记录程序的流程信息
    }
}

// 输出内容
0 [main] FATAL Log4jTest  - fatal test
0 [main] ERROR Log4jTest  - error test
0 [main] WARN Log4jTest  - warn test
0 [main] INFO Log4jTest  - info test
0 [main] DEBUG Log4jTest  - debug test
```

> 可以发现只打印出了TRACE没有打印出来，是由于默认日志级别为DEBUG，低于DEBUG级别的是不会打印出来





## 2、Log4j 组件介绍

Log4j主要由Loggers（日志记录器）、Appenders（输出端）和Layout（日志格式化器）组成，其中Loggers控制日志的输出级别与日志是否输出；Appenders指定日志的输出方式（输出到控制台、文件等）；Layout控制日志信息的输出格式。

### 1、Loggers

日志记录器，负责收集处理日志记录，实例的命名就是类"xx"的full quailied name（类的全限定名），Logger的名字大小写敏感，其命名有继承机制。例如：name为org.apache.commoms的logger会继承name为org.apache的logger。

Log4j中有一个特殊的logger叫做"root"，它是所有logger的根，也就意味着其他所有的logger都会直接或者间接的继承自root。root logger可以用Logger.getRootLogger()方法获取。JUL也有一个名为”.“的根。

PS：自Log4j 1.2版以来， Logger 类已经取代了 Category 类。对于熟悉早期版本的Log4j的人来说，Logger 类可以被视为 Category 类的别名。



### 2、Appenders

Appender 用来指定日志输出到哪个地方（和JUL的Handler很像），可以同时指定日志的输出目的地。Log4j 常用的输出目的地。有以下几种：

| 输出端类型               | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| ConsoleAppender          | 将日志输出到控制台                                           |
| FileAppender             | 将日志输出到文件中                                           |
| DailyRollingFileAppender | 将日志输出到一个日志文件，并且每天输出到一个新的文件         |
| RollingFileAppender      | 将日志信息输出到一个日志文件，并且指定文件的尺寸，当文件大小达到指定尺寸时，会自动把文件改名，同时产生一个新的文件 |
| JDBCAppender             | 把日志信息保存到数据库中                                     |



### 3、Layouts

Layouts 布局器，用于控制日志输出的格式，让我么可以自定义日志格式，Log4j 常用的 Layouts 有以下几种

| 格式化器      | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| HTMLLayout    | 格式化日志为html表格形式                                     |
| XMLLayout     | 格式化日志为xml文档形式                                      |
| SimpleLayout  | （默认）简单的日志输出格式化，打印的日志格式为（info - message） |
| PatternLayout | 最强大的格式化器，可以根据自定义格式输出日志，如果没有指定转换格式，默认的格式只显示消息内容； |



### 4、日志输出格式

使用PatternLayout可以自定义格式输出，是我们最常用的方式，这种格式化输出采用类似C语言的printf函数的打印格式化日志信息，具体占位符及含义如下：

| 参数 | 说明                                           | 举例                                                         |
| ---- | ---------------------------------------------- | ------------------------------------------------------------ |
| `%m` | 输出代码中指定的日志信息                       | `logger.info(日志信息)`                                      |
| `%p` | 输出日志级别                                   | INFO、DEBUG、ERROR `%-8p` 表示占用8个字符，若字符不够用空格代替 |
| `%n` | 换行符                                         | 必须在结尾加上换行符，否则日志将杂乱不堪                     |
| `%r` | 输出从应用启动到输出该log信息耗费的毫秒数      | log打印时间 - 应用启动时间                                   |
| `%c` | 输出打印语句所属的类全名                       | `%c` ：com.xd.App %c{1}：App                                 |
| `%C` | 列出调用logger的类的全名（包含包路径）         | `%C` ：com.xd.App `%C{1}`：App                               |
| `%t` | 输出产生该日志所在的线程名名称                 | main 、thread-0、thread-1                                    |
| `%d` | 输出服务器当前时间，精确到豪秒                 | 输出服务器当前时间, 默认为ISO8601, 也可指定格式：`%d{yyyy-MM-dd HH:mm:ss}` |
| `%l` | 输出日志发生的位置，包含包、类、方法、代码行数 | com.log.App.haveConfigFile(App.java:25)                      |
| `%L` | 输出代码中的行号                               | 12                                                           |
| `%F` | 显示调用logger的类的文件名                     | App.class                                                    |
| `%%` | 输出一个百分号 %                               | %                                                            |

```
%*        可以在 % 与字符之间加上修饰符来控制最小宽度、最大宽度和文本的对其方式。如：
%5c       输出category名称，最小宽度是5，category<5，默认的情况下右对齐
%-5c      输出category名称，最小宽度是5，category<5，"-"号指定左对齐,会有空格
%.5c      输出category名称，最大宽度是5，category>5，就会将左边多出的字符截掉，<5不会有空格
%20.30c   category名称<20补空格，并且右对齐，>30字符，就从左边交远销出的字符截掉
```



### 5、日志输出级别

关于日志级别信息，例如：DEBUG、INFO、WARN、ERROR...级别是分大小的，DEBUG < INFO < WARN < ERROR，分别用来指定这条日志信息的重要程度，Log4j 输出日志的规则是：只输出级别不低于设定级别的日志信息，假设 Loggers 级别设定为 INFO，则 INFO、WARN、ERROR 级别的日志信息都会输出，而级别低于 INFO 的 DEBUG 则不会输出。

Log4j 的默认日志级别是：DEBUG

1. OFF：最高等级的而级别，用户关闭所有的日志记录
2. FATAL：严重的错误信息，一般会造成系统崩溃并终止运行
3. ERROR：记录错误信息，不会影响系统运行。一般情况不想输出太多日志，则使用该级别即可
4. WARN：记录警告信息
5. INFO：记录运行信息
6. DEBUG：（默认）记录调式信息，一般在开发中使用
7. TRACE：记录追踪信息，记录程序的流程信息，一般情况不会使用
8. ALL：最低等级，用于打开所有级别的日志记录

注：一般只使用4个级别，优先级从高到低为 ERROR > WARN > INFO > DEBUG



## 3、硬编码与配置文件

### 1、Java硬编码配置

1、Java代码配置：Appenders使用ConsoleAppender，Layouts使用SimpleLayout。

```java
import org.apache.log4j.ConsoleAppender;
import org.apache.log4j.Level;
import org.apache.log4j.Logger;
import org.apache.log4j.SimpleLayout;
import java.io.PrintWriter;

public class Log4jTest {
    public static void main(String[] args) {
        Logger logger = Logger.getLogger(Log4jTest.class);
        // 配置一个控制台输出源
        ConsoleAppender consoleAppender = new ConsoleAppender(new SimpleLayout());
        consoleAppender.setName("console");
        consoleAppender.setWriter(new PrintWriter(System.out));
        logger.addAppender(consoleAppender);
        logger.setLevel(Level.ERROR);

        logger.fatal("fatal test"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
        logger.error("error test"); // 记录错误信息，不会影响系统运行
        logger.warn("warn test"); // 记录警告信息
        logger.info("info test"); // 记录运行信息
        logger.debug("debug test"); // （默认）记录调式信息，一般在开发中使用
        logger.trace("trace test"); // 记录追踪信息，记录程序的流程信息
    }
}

// 输出内容
FATAL - fatal test
ERROR - error test
```

2、Java代码配置，Appenders使用ConsoleAppender，Layouts使用自定义Layout。

```java
import org.apache.log4j.*;
import org.apache.log4j.spi.LoggingEvent;

public class Log4jTest {
    public static void main(String[] args) {
        Logger logger = Logger.getLogger(Log4jTest.class);

        // 配置一个自定义Layout(自定义格式化输出)
        Layout layout = new Layout() {
            @Override
            public String format(LoggingEvent loggingEvent) {
                // loggingEvent.getLoggerName()输出包名+类名
                return loggingEvent.getLoggerName() + " " + loggingEvent.getMessage() + "\r\n";
            }

            @Override
            public boolean ignoresThrowable() {
                return false;
            }

            @Override
            public void activateOptions() {
            }
        };
        // 配置一个控制台输出源
        ConsoleAppender consoleAppender = new ConsoleAppender(layout);
        consoleAppender.setName("customConsole");
        logger.addAppender(consoleAppender);
        logger.setLevel(Level.ERROR);

        logger.fatal("fatal test"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
        logger.error("error test"); // 记录错误信息，不会影响系统运行
        logger.warn("warn test"); //记录警告信息
        logger.info("info test"); // 记录运行信息
        logger.debug("debug test"); // （默认）记录调式信息，一般在开发中使用
        logger.trace("trace test"); // 记录追踪信息，记录程序的流程信息
    }
}

// 输出内容
Log4jTest fatal test
Log4jTest error test
```

3、注意如下两个配置，需要的时候可以配置

```java
// 初始化配置信息(没有使用使用配置文件情况下)
BasicConfigurator.configure();
// 开启Log4j内置日志记录（用来打印Log4j更加详细的debug信息）
LogLog.setInternalDebugging(true);
```



### 2、加载配置文件

通过查看LogManager源码可知log4j是通过静态代码块中，类加载器加载的配置文件，所以可以在resources下新建一个`log4j.properties`文件：

```properties
# 指定日志的输出级别与输出端
# WARN代表的是输出级别, console是我们自定义的一个名称appenderNam, 这个可以设置打印到多个地方, 中间用逗号隔开
log4j.rootLogger = WARN,console
# 配置appender输出方法及配置输出信息的格式
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.layout = org.apache.log4j.SimpleLayout
```

```java
import org.apache.log4j.Logger;
public class Log4jTest {
    public static void main(String[] args) {
        Logger logger = Logger.getLogger(Log4jTest.class);
        logger.fatal("fatal test"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
        logger.error("error test"); // 记录错误信息，不会影响系统运行
        logger.warn("warn test"); //记录警告信息
        logger.info("info test"); // 记录运行信息
        logger.debug("debug test"); // （默认）记录调式信息，一般在开发中使用
        logger.trace("trace test"); // 记录追踪信息，记录程序的流程信息
    }
}

// 输出内容
FATAL - fatal test
ERROR - error test
WARN - warn test
```



## 4、Appender输出详解

### 1、将日志输出到控制台

ConsoleAppender：将日志输出到控制台

```properties
# 指定日志的输出级别与输出端
# 第1个逗号前是设置的日志级别,后面是自定义名称，输出到什么地方.名称在下方使用
log4j.rootLogger=INFO,console

############################# 控制台输出配置 #############################
# 1、指定控制台日志输出的appender
log4j.appender.console = org.apache.log4j.ConsoleAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式xml.XMLLayout
log4j.appender.console.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容
#log4j.appender.console.layout.conversionPattern = %r [%t] %p %c %x - %m%n
log4j.appender.console.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
```

```java
import org.apache.log4j.Logger;
public class Log4jTest {
    public static void main(String[] args) {
        Logger logger = Logger.getLogger(Log4jTest.class);

        logger.fatal("fatal test"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
        logger.error("error test"); // 记录错误信息，不会影响系统运行
        logger.warn("warn test"); // 记录警告信息
        logger.info("info test"); // 记录运行信息
        logger.debug("debug test"); // （默认）记录调式信息，一般在开发中使用
        logger.trace("trace test"); // 记录追踪信息，记录程序的流程信息
    }
}
```

控制台中输出如下内容

```
[FATAL] 0 Log4jTest.main(Log4jTest.java:6) 2022-03-30 15:09:46:620 fatal test
[ERROR] 0 Log4jTest.main(Log4jTest.java:7) 2022-03-30 15:09:46:620 error test
[WARN] 0 Log4jTest.main(Log4jTest.java:8) 2022-03-30 15:09:46:620 warn test
[INFO] 0 Log4jTest.main(Log4jTest.java:9) 2022-03-30 15:09:46:620 info test
```



### 2、将日志输出到指定文件

FileAppender：将日志输出到文件中

```properties
# 指定日志的输出级别与输出端
# 第1个逗号前是设置的日志级别,后面是自定义名称，输出到什么地方.名称在下方使用
log4j.rootLogger=INFO,file

############################# 文件输出配置 #############################
# 2、指定文件日志输出的appender【默认是追加的】
log4j.appender.file = org.apache.log4j.FileAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式XMLLayout
log4j.appender.file.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容 (%r [%t] %p %c %x - %m%n)
log4j.appender.file.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
# 指定日志文件的保存路径
log4j.appender.file.file = ./logs/log4j.log
# 指定日志文件的字符集
log4j.appender.file.encoding = UTF-8
```

此时这个日志信息会输出到这个指定位置的文件中。项目同级目录下会自动创建目录与文件（logs/log4j.log）输出内容如下：

```
[FATAL] 0 Log4jTest.main(Log4jTest.java:6) 2022-03-30 15:09:46:620 fatal test
[ERROR] 0 Log4jTest.main(Log4jTest.java:7) 2022-03-30 15:09:46:620 error test
[WARN] 0 Log4jTest.main(Log4jTest.java:8) 2022-03-30 15:09:46:620 warn test
[INFO] 0 Log4jTest.main(Log4jTest.java:9) 2022-03-30 15:09:46:620 info test
```



### 3、根据时间来拆分日志文件

DailyRollingFileAppender：将日志输出到一个日志文件，并且每天输出到一个新的文件

这个会根据你输入的时间间隔来生成新的日志信息，这个时间可以是一天，也可以是一秒，需要注意的是这个并不是自动为我们生成新的日志文件，是我们手动生成的日志文件，比如：你设置的间隔是 yyyy-MM-dd ，这个时候如果你现在输出了一个日志文件，那么在这个时间开始的后24个小时内都不会生成新的日志文件，在这24小时内输出的日志文件都会存储到这个一个旧的日志文件中。即使过了24个小时，系统也不会为我们自动生成一个新的日志文件，需要程序员在输出日志的时候才会生成一个新的日志文件，加可以精确到秒，那么一秒就会为我们生成一个新的日志文件。

```properties
# 指定日志的输出级别与输出端
# 第1个逗号前是设置的日志级别,后面是自定义名称，输出到什么地方.名称在下方使用
log4j.rootLogger=INFO,dailyFile

############################# 按照时间规则输出到文件 #############################
# 3、按日期日志文件输出的appender【在这个类中没有提供覆盖的方法】
log4j.appender.dailyFile = org.apache.log4j.DailyRollingFileAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式XMLLayout
log4j.appender.dailyFile.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容
log4j.appender.dailyFile.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
# 指定日志文件的保存路径【当前文件默认是覆盖的】
log4j.appender.dailyFile.file = ./logs/log4j.log
# 指定日志文件的字符集
log4j.appender.dailyFile.encoding = UTF-8
# 指定日期拆分规则 默认【'.'yyyy-MM-dd】时间pattern只能以短横线-划分
log4j.appender.dailyFile.datePattern = '.'yyyy-MM-dd-HH-mm-ss
```

多次执行测试类，这时可以发现指定路径下会按照时间秒生产日志文件（log4j.log 中文件是覆盖的，log4j.log.xxx文件在同一个时间内是追加的）

```
log4j.log
log4j.log.2022-03-30-15-46-12
log4j.log.2022-03-30-15-46-14
```

在测试时可以根据秒来指定拆分策略，但实际生产环境中，我们可以根据按天或者周、月进行拆分。



### 4、根据文件大小拆分日志文件

RollingFileAppender：将日志信息输出到一个日志文件，并且指定文件的尺寸，当文件大小达到指定尺寸时，会自动把文件改名，同时产生一个新的文件

```properties
# 指定日志的输出级别与输出端
# 第1个逗号前是设置的日志级别,后面是自定义名称，输出到什么地方.名称在下方使用
log4j.rootLogger=INFO,rollingFile

############################# 按照文件大小拆分输出到文件 #############################
# 4、按照文件大小拆分的appender
log4j.appender.rollingFile = org.apache.log4j.RollingFileAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式XMLLayout
log4j.appender.rollingFile.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容
log4j.appender.rollingFile.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
# 指定日志文件的保存路径
log4j.appender.rollingFile.file = ./logs/log4j.log
# 指定日志文件的字符集
log4j.appender.rollingFile.encoding = UTF-8
# 指定日志文件内容的大小【这里为了测试所以1KB】
log4j.appender.rollingFile.maxFileSize = 1KB
# 指定日志文件的数量【默认是1个】 超过5个，按照时间覆盖(最旧的文件覆盖最新的文件)
log4j.appender.rollingFile.maxBackupIndex = 5
```

```java
import org.apache.log4j.Logger;
public class Log4jTest {
    public static void main(String[] args) {
        Logger logger = Logger.getLogger(Log4jTest.class);
        for (int i = 0; i < 100; i++) {
            logger.fatal("fatal test"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
            logger.error("error test"); // 记录错误信息，不会影响系统运行
            logger.warn("warn test"); // 记录警告信息
            logger.info("info test"); // 记录运行信息
            logger.debug("debug test"); // （默认）记录调式信息，一般在开发中使用
            logger.trace("trace test"); // 记录追踪信息，记录程序的流程信息
        }
    }
}
```

只要文件超过1KB，那么则生成另一个文件，文件的数量最多5个。如果5个文件啊不够怎么办，作为日志管理来讲，不可能让日志无休止的增长，所以覆盖策略是：按照时间来进行覆盖，原则是保留新的，覆盖旧的。这个时候日志6会覆盖掉日志1.

```cmd
log4j.log
log4j.log.1
log4j.log.2
log4j.log.3
log4j.log.4
log4j.log.5
```



### 5、将日志信息存储到数据库

JDBCAppender：把日志信息保存到数据库中

数据库这里需要新建数据库，加入数据库的依赖，然后在配置文件中配置连接数据库的信息并且设置插入语句。

```sql
-- create table
CREATE TABLE tab_log (
    log_id int(11) NOT NULL AUTO_INCREMENT,
    project_name varchar(255) DEFAULT NULL COMMENT '目项名',
    create_date  varchar(255) DEFAULT NULL COMMENT '创建时间',
    level varchar(255) DEFAULT NULL COMMENT '优先级',
    category  varchar(255) DEFAULT NULL COMMENT '所在类的全名',
    file_name varchar(255) DEFAULT NULL COMMENT '输出日志消息产生时所在的文件名称',
    thread_name varchar(255) DEFAULT NULL COMMENT '日志事件的线程名',
    line varchar(255) DEFAULT NULL COMMENT '号行',
    all_category varchar(255) DEFAULT NULL COMMENT '日志事件的发生位置',
    message  varchar (4000) DEFAULT NULL COMMENT '输出代码中指定的消息',
    PRIMARY KEY (log_id)
);
```

```xml
<!--mysql依赖-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.9</version>
</dependency>
```

```properties
# 指定日志的输出级别与输出端
# 第1个逗号前是设置的日志级别,后面是自定义名称，输出到什么地方.名称在下方使用
log4j.rootLogger=INFO,logDB

############################# 将日志信息存储到数据库 #############################
# 5、持久化日志信息 将日志信息存储到数据库的appender
log4j.appender.logDB = org.apache.log4j.jdbc.JDBCAppender
log4j.appender.logDB.layout = org.apache.log4j.PatternLayout
log4j.appender.logDB.Driver = com.mysql.jdbc.Driver
log4j.appender.logDB.URL = jdbc:mysql://localhost/tab_log
log4j.appender.logDB.User = root
log4j.appender.logDB.Password = password
# 此时要像数据库中插入数据，使用insert语句
log4j.appender.logDB.Sql =  INSERT INTO tab_log(name,create,level,category,fileName,message) values('project_log','%d{yyyy-MM-dd HH:mm:ss}','%p','%c','%F','%m')
```

然后执行测试类，可以查看到数据中成功插入日志。



### 6、将日志信息输出到多个位置

上面分别介绍了单个Appender的用法，实际上我们还可以将日志同时输出到多个Appender中。下面示例同时输出到控制台、文件与数据库中

```properties
# 指定日志的输出级别与输出端
# 第1个逗号前是设置的日志级别,后面是自定义名称，输出到什么地方.名称在下方使用
log4j.rootLogger=INFO,console,file,logDB

############################# 控制台输出配置 #############################
# 1、指定控制台日志输出的appender
log4j.appender.console = org.apache.log4j.ConsoleAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式xml.XMLLayout
log4j.appender.console.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容
#log4j.appender.console.layout.conversionPattern = %r [%t] %p %c %x - %m%n
log4j.appender.console.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd- HH:mm:ss:SSS} %m%n

############################# 文件输出配置 #############################
# 2、指定文件日志输出的appender【默认是追加的】
log4j.appender.file = org.apache.log4j.FileAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式XMLLayout
log4j.appender.file.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容 (%r [%t] %p %c %x - %m%n)
log4j.appender.file.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
# 指定日志文件的保存路径
log4j.appender.file.file = ./logs/log4j.log
# 指定日志文件的字符集
log4j.appender.file.encoding = UTF-8

############################# 将日志信息存储到数据库 #############################
# 5、持久化日志信息 将日志信息存储到数据库的appender
log4j.appender.logDB = org.apache.log4j.jdbc.JDBCAppender
log4j.appender.logDB.layout = org.apache.log4j.PatternLayout
log4j.appender.logDB.Driver = com.mysql.jdbc.Driver
log4j.appender.logDB.URL = jdbc:mysql://localhost/tab_log
log4j.appender.logDB.User = root
log4j.appender.logDB.Password = password
# 此时要像数据库中插入数据，使用insert语句
log4j.appender.logDB.Sql =  INSERT INTO tab_log(name,create,level,category,fileName,message) values('project_log','%d{yyyy-MM-dd HH:mm:ss}','%p','%c','%F','%m')
```

实际上主要是对这个配置的修改：

```properties
# 指定日志的输出级别与输出端, 第1个逗号前是设置的日志级别,后面是自定义名称，输出到什么地方.名称在下方使用
log4j.rootLogger = info,console,file
```



## 5、自定义配置Logger

```properties
# 使用最高父类rootLogger配置logger，这个时候继承的是父logger(根节点)
log4j.rootLogger = info,console

# 配置自定义loggerlogger对象,会默认继承rootLogger
# 级别会覆盖, INFO升级为WARN. Appender会继承, 既会向控制台输出console, 也会向文件file输出
# 注意：这里自定义的com.xyz是Log4jTest类所在的包名路径
log4j.logger.com.xyz = warn,file

# 第三方框架(org.apache.log4j.Logger)的错误日志打印，不指定appender，就是默认rootLogger的appender（console）
log4j.logger.org.apache = error


############################# 控制台输出配置 #############################
# 1、指定控制台日志输出的appender
log4j.appender.console = org.apache.log4j.ConsoleAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式xml.XMLLayout
log4j.appender.console.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容
#log4j.appender.console.layout.conversionPattern = %r [%t] %p %c %x - %m%n
log4j.appender.console.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd HH:mm:ss:SSS} %m%n

############################# 文件输出配置 #############################
# 2、指定文件日志输出的appender【默认是追加的】
log4j.appender.file = org.apache.log4j.FileAppender
# 指定消息格式layout,使用简单格式SimpleLayout ,HTML格式HTMLLayout,xml格式XMLLayout
log4j.appender.file.layout = org.apache.log4j.PatternLayout
# 指定消息格式的内容 (%r [%t] %p %c %x - %m%n)
log4j.appender.file.layout.conversionPattern =[%p] %r %l %d{yyyy-MM-dd HH:mm:ss:SSS} %m%n
# 指定日志文件的保存路径
log4j.appender.file.file = ./logs/log4j.log
# 指定日志文件的字符集
log4j.appender.file.encoding = UTF-8
```

```java
package com.xyz;

import org.apache.log4j.Logger;
public class Log4jTest {
    public static void main(String[] args) {
        Logger logger = Logger.getLogger(Log4jTest.class);
        logger.fatal("fatal test"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
        logger.error("error test"); // 记录错误信息，不会影响系统运行
        logger.warn("warn test"); // 记录警告信息
        logger.info("info test"); // 记录运行信息
        logger.debug("debug test"); // （默认）记录调式信息，一般在开发中使用
        logger.trace("trace test"); // 记录追踪信息，记录程序的流程信息


        // 三方框架的错误日志信息打印
        // 因为log4j.properties文件中appender名是类名的包路径，
        // 这里是org.apache.log4j.Logger，就可以继承log4j.properties的配置
        Logger logger2 = Logger.getLogger(Logger.class);
        logger2.fatal("fatal test2"); // 记录严重的错误信息，一般会造成系统崩溃并终止运行
        logger2.error("error test2"); // 记录错误信息，不会影响系统运行
        logger2.warn("warn test2"); // 记录警告信息
        logger2.info("info test2"); // 记录运行信息
        logger2.debug("debug test2"); // （默认）记录调式信息，一般在开发中使用
        logger2.trace("trace test2"); // 记录追踪信息，记录程序的流程信息
    }
}
```

控制台输出：

```
[FATAL] 0 com.xyz.Log4jTest.main(Log4jTest.java:7) 2022-03-30 17:58:48:215 fatal test
[ERROR] 10 com.xyz.Log4jTest.main(Log4jTest.java:8) 2022-03-30 17:58:48:225 error test
[WARN] 10 com.xyz.Log4jTest.main(Log4jTest.java:9) 2022-03-30 17:58:48:225 warn test
[FATAL] 10 com.xyz.Log4jTest.main(Log4jTest.java:19) 2022-03-30 17:58:48:225 fatal test2
[ERROR] 10 com.xyz.Log4jTest.main(Log4jTest.java:20) 2022-03-30 17:58:48:225 error test2
```

文件中输出：

```
[FATAL] 0 com.xyz.Log4jTest.main(Log4jTest.java:7) 2022-03-30 17:58:48:215 fatal test
[ERROR] 10 com.xyz.Log4jTest.main(Log4jTest.java:8) 2022-03-30 17:58:48:225 error test
[WARN] 10 com.xyz.Log4jTest.main(Log4jTest.java:9) 2022-03-30 17:58:48:225 warn test
```

可以看出：file中不会存入org.apache这个logger的日志，因为该logger没有指定appender，默认的rootLogger的appender只是输出到控制台的

> 自定义Logger结论：如果根节点的Logger和自定义父Logger配置的输出位置是不同的则取二者的并集，也就是配置的位置都会进行日志的输出。如果二者配置的日志级别不同，主要以按照我们自定义的父logger的级别输出为主。



## 6、参考文献 & 鸣谢

- 日志技术之Log4j学习【CSDN：StudyWinter】https://blog.csdn.net/Zhouzi_heng/article/details/107965987
- java日志：二、log4j使用【CSDN：小徐也要努力鸭】https://blog.csdn.net/a232884c/article/details/121102525

# 日志门面统一技术

## 1、日志门面介绍

### 1、什么是日志门面技术

日志门面技术借鉴了 JDBC 的思想，为日志系统也提供了一套门面，开发者就可以面向这些接口规范来开发，避免了直接依赖具体日志框架，因为不同的日志框架有不同的用法，所以导致在更换日志框架时就需要改动大量的代码，所以为了减少开发量，就有了日志门面技术，门面技术简单地理解就是说你的日志框架变了，但是不需要改代码，只修改依赖和配置文件即可，提高了开发效率

- 常见的日志门面： JCL、SLF4J
- 常见的日志实现： JUL、Log4j、Logback、Log4j2
- 日志框架出现的历史顺序：Log4j » JUL » JCL » SLF4J » Logback » Log4j2
- 常用门面及实现组合：slf4j + logback、slf4j + log4j2

因为JCL支持的日志框架比较少，只支持目前比较主流的日志框架，且不支持扩展，需要扩展其他的日志实现技术的话需要用户自己修改源代码，但是随着技术的发展，以后肯定会发展出更多更好的日志框架，但JCL却不支持这些新出的框架，因此JCL在2014年就已经被 Apache 给淘汰了，目前使用最广泛的是SLF4J



### 2、为什么要用门面技术

1. 面向接口开发，不在依赖具体的实现类，减少代码的耦合
2. 项目通过导入不同的日志实现类，可以灵活的切换日志框架
3. 统一API，方便开发者学习和使用
4. 统一配置便于项目日志的管理



## 2、日志门面SLF4J

官网：https://www.slf4j.org/、Maven 仓库地址：https://search.maven.org/artifact/org.slf4j/slf4j-api

简单日志门面（Simple Logging Facade For Java）主要是为了给Java日志访问提供一套标准、规范的API框架，其主要意义在于提供接口，具体的实现可以交由其它日志框架，例如log4j和logback等。当然slf4j自己也提供了功能较为简单的实现，但是一般很少用到。对于一般的java项目而言，日志框架会选择slf4j-api作为门面，配上具体的实现框架（log4j、logback等），中间使用桥接器完成桥接。并且类路径要绑定具体的日志实现框架（Logback、Log4j、Log4j2、JUL），否则无操作去实现，因为SLF4J只是接口。

SLF4J是目前市面上最流行的日志门面。现在的项目中基本上都是使用SLF4J。主要提供两大功能：

1. 日志框架的绑定（Binding）：绑定其他日志的实现
2. 日志框架的桥接（Bridging）：桥接旧的日志框架

```
                         ┌       日志桥接绑定                 日志实现组件
                         ├  loback-classic-version.jar   »»»   logback
           日志门面       ├  slf4j-log4j12-version.jar    »»»   log4j
client »»» slf4j-api »»» ├  slf4j-jdk14-version.jar      »»»   jul(jdk)
                         ├  slf4j-jcl-version.jar        »»»   jcl
                         ├  slf4j-simple-version.jar     »»»   slf4j simple
                         └  slf4j-nop-version.jar        »»»   slf4j nop
```





### 1、核心接口或类 & 日志级别 

核心接口或类：

1. org.slf4j.Logger：核心接口（包含了日志操作的方法）
2. org.slf4j.LoggerFactory：工厂类，获取日志实例

LoggerFactory：位于slf4j-api.jar包中，是一个用于生产各种Logger的工厂，有3个公共方法。

| 方法名称                  | 含义                                          |
| ------------------------- | --------------------------------------------- |
| getILoggerFactory         | 返回logger工厂，getLogger方法内部会调用该方法 |
| getLogger(Class<?> clazz) | 通过clazz查找具体的logger                     |
| getLogger(String name)    | 通过name查找具体的logger                      |

日志级别有5类，分别是ERROR、WARN、INFO、DEBUG、TRACE。源码如下：

```java
ERROR(ERROR_INT, "ERROR"), 
WARN(WARN_INT, "WARN"), 
INFO(INFO_INT, "INFO"), 
DEBUG(DEBUG_INT, "DEBUG"), 
TRACE(TRACE_INT, "TRACE");

final public int TRACE_INT = 00;
final public int DEBUG_INT = 10;
final public int INFO_INT = 20;
final public int WARN_INT = 30;
final public int ERROR_INT = 40;
```



### 2、绑定默认日志实现 Binding

#### 1、单独使用SLF4J

直接使用SLF4J日志门面，不加日志实现框架

```xml
<!--  slf4j 日志门面  -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SLF4JTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(SLF4JTest.class);
        logger.error("error test");
        logger.warn("warn test");
        logger.info("info {}", "test");
        logger.debug("debug test ");
        logger.trace("trace test");
    }
}
```

因为目前只有接口，还没有具体的实现，此时还不会打印内容，会打印提示您没有实现类

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```



#### 2、SLF4J日志开关

slf4j-nop是slf4j开关，加上该依赖，重新执行Test，既不会打印内容，也不会提示

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-nop</artifactId>
    <version>1.7.32</version>
</dependency>
```



#### 3、绑定默认日志框架

slf4j-simple是slf4j内置的简单日志实现框架，功能比较简单

```xml
<!--slf4j 日志门面-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
<!--slf4j 内置的简单实现-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.32</version>
</dependency>
```

运行上述代码，控制台打印的结果如下

```
[main] ERROR SLF4JTest - error test
[main] WARN SLF4JTest - warn test
[main] INFO SLF4JTest - info test
```



### 3、绑定其他日志实现 Binding

如前所述，SLF4J支持各种日志框架。SLF4J发行版附带了几个称为“SLF4J绑定”的jar文件，每个绑定对应一个受支持的框架。使用SLF4J的日志绑定流程**:** 

1. 添加slf4j-api的依赖
2. 使用slf4j的API在项目中进行统一的日志记录
3. 绑定具体的日志实现框架
   - 绑定已经实现了slf4j的日志框架，直接添加对应依赖
   - 绑定没有实现slf4j的日志框架，先添加日志的适配器，再添加实现类的依赖
4. slf4j有且仅有一个日志实现框架的绑定（如果出现多个默认使用第一个依赖日志实现）

打开官网的用户手册查看流程图：https://www.slf4j.org/manual.html

![img](https://www.slf4j.org/images/concrete-bindings.png)

#### 1、绑定JUL日志实现

```xml
<!--slf4j 日志门面-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
<!--JUL适配器依赖,因为JUL是JDK内置的，所以不需要额外导入JUL实现的依赖-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.7.32</version>
</dependency>
```

运行上述代码，控制台打印的结果如下

```
3月 31, 2022 10:07:14 上午 SLF4JTest main
严重: error test
3月 31, 2022 10:07:14 上午 SLF4JTest main
警告: warn test
3月 31, 2022 10:07:14 上午 SLF4JTest main
信息: info test
```



#### 2、绑定Log4j日志框架

```xml
<!--slf4j 日志门面-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
<!--导入log4j适配器依赖-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.32</version>
</dependency>
<!--导入log4j依赖-->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

除此之外，还需要加上log4j的配置文件（main/resources目录下）：`log4j.properties`

```properties
# 使用根节点的logger对象, 将日志信息输出到控制台
log4j.rootLogger = info,console
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.conversionPattern = [%p]%r %c%t%d{yyyy-mm-dd HH:mm:ss:SSS} %m%n
```

运行上述代码，控制台打印的结果如下

```
[ERROR]0 SLF4JTestmain2022-16-31 10:16:50:961 error test
[WARN]0 SLF4JTestmain2022-16-31 10:16:50:961 warn test
[INFO]10 SLF4JTestmain2022-16-31 10:16:50:971 info test
```



#### 3、日志实现框架切换

要切换日志框架，只需替换类路JUL切换到Log4j，只需将 slf4j-jdk14-1.7.32.jar 替换为 slf4j-log4j12-1.7.32.jar 即可。

SLF4J不依赖于任何特殊的类装载。实际上，每个SLF4J绑定在编译时都是硬连线的， 以使用一个且只有一个特定的日志记录框架。例如：slf4j-log4j12-1.7.32.jar绑定在编译时绑定以使用log4j。



### 4、桥接旧的日志框架 Bridging

通常，您依赖的某些组件依赖于SLF4J以外的日志记录API。您也可以假设这些组件在不久的将来不会切换到SLF4J。为了解决这种情况，SLF4J附带了几个桥接模块，这些模块将对log4j、JCL和java.util.loggingAPI的调用重定向，就好像它们是SLF4J的API一样。

桥接解决的是项目中日志的遗留问题，当系统中存在之前的日志API，可以通过桥接转换到slf4j的实现：

1. 先去除之前老的日志框架的依赖，必须去掉
2. 添加SLF4J提供的桥接组件，这个组件就是模仿之前老的日志写了一套相同的api，只不过这个api是在调用SLF4J的api
3. 为项目添加SLF4J的具体实现（如Logback）

#### 1、SLF4J桥接器介绍

旧日志迁移方式：

```xml
<!--slf4j 日志门面-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
<!--桥接的组件,加上这个依赖后，虽然import上是log4j，但是底层调用的其实是logback的日志实现；-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.32</version>
</dependency>
<!--Logback 日志实现依赖-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.5</version>
</dependency>
```

SLF4J提供的桥接器：

```xml
<!-- log4j-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.32</version>
</dependency>
<!-- jul -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jul-to-slf4j</artifactId>
    <version>1.7.32</version>
</dependency>
<!--jcl -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.32</version>
</dependency>
```

注意问题：

1. log4j-over-slf4j 桥接器 和 slf4j-jcl.jar 适配器 不能同时部署，否则会导致无限循环
   前一个jar文件将导致JCL将日志系统的选择委托给SLF4J，后一个jar文件将导致SLF4J将日志系统的选择委托给JCL，从而导致无限循环
2. log4j-over-slf4j 桥接器 和 slf4j-log4j 适配器 不能同时出现
3. jul-to-slf4j.jar 桥接器和 slf4j-jdk14.jar 适配器 不能同时出现
4. 所有的桥接器都只对Logger日志记录器对象有效，如果程序中调用了内部的配置类或者是Appender、Filter等对象，将无法产生效果

模拟同时导入log4j 适配器与桥接器 导致死循环 示例：

```xml
<!--slf4j日志门面-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
<!--配置log4j的桥接器-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.32</version>
</dependency>
<!--配置log4j的适配器-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.32</version>
</dependency>
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SLF4JTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(SLF4JTest.class);
        logger.error("error test");
        logger.warn("warn test");
        logger.info("info test");
        logger.debug("debug test ");
        logger.trace("trace test");
    }
}
```

```
SLF4J: Detected both log4j-over-slf4j.jar AND bound slf4j-log4j12.jar on the class path, preempting StackOverflowError. 
SLF4J: See also http://www.slf4j.org/codes.html#log4jDelegationLoop for more details.
Exception in thread "main" java.lang.ExceptionInInitializerError
	at org.slf4j.impl.StaticLoggerBinder.<init>(StaticLoggerBinder.java:72)
	at org.slf4j.impl.StaticLoggerBinder.<clinit>(StaticLoggerBinder.java:45)
	at org.slf4j.LoggerFactory.bind(LoggerFactory.java:150)
	at org.slf4j.LoggerFactory.performInitialization(LoggerFactory.java:124)
	at org.slf4j.LoggerFactory.getILoggerFactory(LoggerFactory.java:417)
	at org.slf4j.LoggerFactory.getLogger(LoggerFactory.java:362)
	at org.apache.log4j.Category.<init>(Category.java:57)
	at org.apache.log4j.Logger.<init>(Logger.java:37)
	at org.apache.log4j.Log4jLoggerFactory.getLogger(Log4jLoggerFactory.java:43)
	at org.apache.log4j.Logger.getLogger(Logger.java:41)
	at org.apache.log4j.Logger.getLogger(Logger.java:49)
	at SLF4JTest.main(SLF4JTest.java:6)
Caused by: java.lang.IllegalStateException: Detected both log4j-over-slf4j.jar AND bound slf4j-log4j12.jar on the class path, preempting StackOverflowError. See also http://www.slf4j.org/codes.html#log4jDelegationLoop for more details.
	at org.slf4j.impl.Log4jLoggerFactory.<clinit>(Log4jLoggerFactory.java:54)
	... 12 more
```



#### 2、日志框架桥接示例

使用场景：老项目使用的日志框架是Log4j，现在想要使用Logback， 但是一大堆的Java文件中都有Log4j的导入，并且代码还是Log4j的api代码。因为要使用Logback，就一定要将Log4j相关的maven依赖删除，依赖删除后，这个import导入就一定会报错，一个个去改把，太麻烦了，搞不好改错了，更麻烦，于是就有了日志桥接器，只需要加上桥接器的maven依赖即可，代码一行都不需要改。

1、先模拟老项目使用Log4j日志框架，导入Log4j依赖，编写测试代码

```xml
<!--导入log4j依赖-->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

```java
import org.apache.log4j.BasicConfigurator;
import org.apache.log4j.Logger;

public class SLF4JTest {
    public static void main(String[] args) {
        BasicConfigurator.configure();
        Logger logger = Logger.getLogger(SLF4JTest.class);
        logger.error("error test");
        logger.warn("warn test");
        logger.info("info test");
        logger.debug("debug test ");
        logger.trace("trace test");
    }
}

// 输出内容
0 [main] ERROR SLF4JTest  - error test
0 [main] WARN SLF4JTest  - warn test
0 [main] INFO SLF4JTest  - info test
0 [main] DEBUG SLF4JTest  - debug test
```

2、修改依赖为Logback的日志框架（先注释掉Log4j的依赖，增加Slf4j-api和Logback的依赖）：

```xml
<!--配置slf4j日志门面-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
<!--配置logback-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.5</version>
</dependency>
```

3、此时由于没有了log4j的依赖包，代码就会提示错误（如果整个项目各个地方用到，每个地方都去修改就会工作量很大）在原有的依赖配置上添加如下依赖：

```xml
<!--配置log4j的桥接器-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.32</version>
</dependency>
```

4、报错消失，运行上面的代码结果如下（表面上看，导入的是log4j的包，实际上执行效果却是logback的日志打印）

```
12:14:36.179 [main] ERROR SLF4JTest - error test
12:14:36.179 [main] WARN SLF4JTest - warn test
12:14:36.179 [main] INFO SLF4JTest - info test
12:14:36.179 [main] DEBUG SLF4JTest - debug test
```



### 5、SLF4J注意事项与原理解析

1、maven依赖中，尽量保证只有一个日志实现，如果由多个日志实现的话默认使用第一个（maven依赖靠前的那个），并且在控制台会显示以下警告信息

```cmd
# 提示有多个日志实现
SLF4J: Class path contains multiple SLF4J bindings.
# 第一个日志实现
SLF4J: Found binding in [jar:file:/D:/repository/org/slf4j/slf4j-simple/1.7.25/ slf4j-simple-1.7.25.jar!/org/slf4j/impl/StaticLoggerBin
# 第二个日志实现
SLF4J: Found binding in [jar:file:/D:/repository/org/slf4j/slf4j-log4j12/1.7.12/slf4j-log4j12-1.7.12.jar!/org/slf4j/impl/StaticLoggerBinder.class]
# 默认绑定第一个
SLF4J: Actual binding is of type [org.slf4j.impl.SimpleLoggerFactory]
```

2、SLF4J原理解析

1. SLF4J通过LoggerFactory加载日志具体的实现对象。

2. LoggerFactory在初始化的过程中，会通过performInitialization()方法绑定具体的日志实现。

3. 在绑定具体实现的时候，通过类加载器，加载org/slf4j/impl/StaticLoggerBinder.class

4. 所以只要是一个日志实现框架，在org.slf4j.impl包中提供一个自己的StaticLoggerBinder类，在其中提供具体日志实现的LoggerFactory就可以被SLF4J所加载



### 6、参考文献 & 鸣谢

- java日志：四、slf4j使用【CSDN：小徐也要努力鸭】https://blog.csdn.net/a232884c/article/details/121160183
- java日志(三)–slf4j和logback使用【CSDN：panda-star】https://blog.csdn.net/chinabestchina/article/details/104743907



## 3、日志门面JCL

### 1、JCL简单入门

#### 1、JCL简介

JCL全称为Jakarta Commons Logging，是Apache提供的一个通用日志API。 该日志门面的使用并不是很广泛。它是为 "所有的Java日志实现"提供一个统一的接口，它自身也提供一个日志的实现（SimpleLog）但是功能非常常弱 。所以一般不会单独使用它。它允许开发人员使用不同的具体日志实现工具：Log4j，JDK自带的日志（JUL）

JCL 有两个基本的抽象类：Log（基本记录器）和LogFactory（负责创建Log实例）

APP（Java应用）=> JCL => Log4j、JUL、SimpleLog

日志实现优先级排列：JCL门面技术在获取Log实现时会按照以下顺序依次获取Logger的实现，可以看到优先使用Log4j，如果匹配就不会继续找了

1. log4j： 优先使用log4j
2. jdk14Logger：jdk自带logger
3. jdk13LumberjackLogger：也是jdk自带logger
4. SimpleLog： jcl自带的日志框架，功能比较简单



#### 2、JCL示例

1、先添加apache门面的pom依赖

```xml
<!--配置JCL日志门面-->
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
```

2、测试代码打印日志

```java
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

public class JCLTest {
    public static void main(String[] args) {
        // 创建日志对象
        Log log = LogFactory.getLog(JCLTest.class);
        // 日志记录输出
        log.fatal("fatal");
        log.error("error");
        log.warn("warn");
        log.info("info");
        log.debug("debug");
    }
}
```

3、打印结果如下，发现默认情况下使用的是jdk（jdk14）自带的日志框架

```
3月 31, 2022 9:44:45 下午 JCLTest main
严重: fatal
3月 31, 2022 9:44:45 下午 JCLTest main
严重: error
3月 31, 2022 9:44:45 下午 JCLTest main
警告: warn
3月 31, 2022 9:44:45 下午 JCLTest main
信息: info
```

4、接下来我们加上Log4j的依赖

```xml
<!--配置JCL日志门面-->
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
<!--配置log4j日志实现-->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

5、再加上Log4j的配置文件 log4j.properties，放在resources目录下

```properties
# 使用根节点的logger对象, 将日志信息输出到控制台
log4j.rootLogger = info,console
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.conversionPattern = [%p]%r %c%t%d{yyyy-mm-dd HH:mm:ss:SSS} %m%n
```

6、其他代码不变，再次运行main方法后打印结果如下 ，会发现自动使用了log4j进行输出日志

```
[FATAL]0 JCLTestmain2022-46-31 22:46:47:699 fatal
[ERROR]0 JCLTestmain2022-46-31 22:46:47:699 error
[WARN]0 JCLTestmain2022-46-31 22:46:47:699 warn
[INFO]0 JCLTestmain2022-46-31 22:46:47:699 info
```



### 2、JCL原理

1、通过LogFactory动态加载Log实现类

```
                               Log
                                ⬆
     ┌───────────────┬──────────────────────┬──────────────────┐
Log4JLogger     JDK14Logger      Jdk13LumberjackLogger      SimpleLog
```

2、日志门面支持的日志实现数组（在LogFactoryImpl类中）

```java
private static final String[] classesToDiscover = {
    "org.apache.commons.logging.impl.Log4JLogger",
    "org.apache.commons.logging.impl.Jdk14Logger",
    "org.apache.commons.logging.impl.Jdk13LumberjackLogger",
    "org.apache.commons.logging.impl.SimpleLog"
};
```

3、获取具体的日志实现（也在LogFactoryImpl类中）

```java
for(int i = 0; i < classesToDiscover.length && result == null; ++i) {
    result = this.createLogFromClass(classesToDiscover[i], logCategory, true);
}
if (result == null) {
    throw new LogConfigurationException("No suitable Log implementation");
}
return result;
```

源码可知，读取到一个日志的结果，result不为空，就会跳出for循环，直接返回。

以之前代码为例。这里首先会在 classesToDiscover 属性中寻找，按数组下标顺序进行寻找，开始我们没有加入log4j依赖和配置文件，这里 "org.apache.commons.logging.impl.Log4JLogger",就找不到，会继续在数组中寻找，第二个"org.apache.commons.logging.impl.Jdk14Logger",是JDK自带的，此时就使用它。当我们加入log4j依赖和配置文件后，就直接找到org.apache.commons.logging.impl.Log4JLogger，并使用它。如果需要再加入其它的日志，需要修改原码，并且在log中实现它，扩展性很不好。JCL已经被apache淘汰了。



### 3、参考文献 & 鸣谢

- java日志：三、JCL使用【CSDN：小徐也要努力鸭】https://blog.csdn.net/a232884c/article/details/121157039
- 日志技术之JCL学习【CSDN：StudyWinter】https://blog.csdn.net/Zhouzi_heng/article/details/107971135

# Logback日志框架

## 1、Logback入门案例

### 1、Logback简介

官方网站：https://logback.qos.ch/index.html、Maven 仓库地址：https://search.maven.org/artifact/ch.qos.logback/logback-classic

Logback日志框架的优势：

- Logback 是 log4j 框架的作者开发的新一代日志框架（性能比Log4j要好），它效率更高、能够适应诸多的运行环境，同时天然支持SLF4J
- Logback的定制性更加灵活，同时也是SpringBoot的内置日志框架

Logback主要分为三个模块：

- logback-core：其它两个模块的基础模块，也是最核心的模块
- logback-classic：它是Log4j的一个改良版本，同时它完整实现了SLF4J API。使你可以方便的更换成其它日志框架（Log4j、JUL）
- logback-access：访问模块与Servlet容器集成提供通过HTTP来访问日志的功能



### 2、Logback示例

```xml
<!--使用slf4j日志门面,实际上logback默认就自带slf4j依赖,不引入当前依赖也可以-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>
<!--logback的基础模块,实际上也可以不导入,因为logback-classic中也包含了基础模块包-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
</dependency>

<!--实际可以只导入logback-classic包即可-->
<!--导入logback-classic依赖,实际上当前依赖已经包含了slf4j-api及logback-core,所以导入此依赖就够了-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogbackTest.class);
        logger.error("error test");
        logger.warn("warn test");
        logger.info("info {}", "test");
        logger.debug("debug test ");
        logger.trace("trace test");
    }
}
```

```
17:12:38.837 [main] ERROR LogbackTest - error test
17:12:38.840 [main] WARN LogbackTest - warn test
17:12:38.840 [main] INFO LogbackTest - info test
17:12:38.841 [main] DEBUG LogbackTest - debug test 
```

***

源码分析：其实我们发现即使项目中没有引入slf4j我们这里也是用的slf4j门面进行编程，下面我们查看源码及依赖传递：

1、可以查看到Logback中的Logger继承了org.slf4j.Logger

```java
package ch.qos.logback.classic;
public final class Logger implements org.slf4j.Logger, LocationAwareLogger, AppenderAttachable<ILoggingEvent>, Serializable {}
```

2、从logback的pom依赖中我们看到slf4-api及logback-core，依赖会进行传递

```xml
<!--只贴出了关键部分--><
parent>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-parent</artifactId>
    <version>1.2.3</version>
</parent>
<artifactId>logback-classic</artifactId>
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```



## 2、组件介绍及配置

### 1、几大组件介绍

1. Logger：日志的记录器，主要用于存放日志对象，也可以定义日志类型、级别
2. Appender：输出源，用于指定日志输出的目的地（可以是控制台、文件、数据库 等等）
3. Encoder：与Layout一样，用来格式化日志信息输出，把事件转换成字符串。注意：在logback 0.9.19 版之前没有 encoder
4. Layout：在Logback中Layout对象被封装在Encoder中，Encoder等于Layout。所以一般使用的都是encoder
5. Level：是用来定义日志级别的（默认是：DEBUG）
6. Filter：过滤器，根据需要定制哪些信息会被记录，哪些信息会被放过
7. LoggerContext：各个logger都被关联到一个LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用

**Appender** ：Logback 将写入日志事件的任务委托给一个名为 Appender 的组件。Appender 必须实现 ch.qos.logback.core.Appender 接口，Appender组件用来定义日志输出格式，日志如何过滤以及日志文件的处理。appender元素可以包含零个或一个layout元素，零个或多个encoder元素以及零个或多个filter元素。

**Encoder**：一个appender有一个encoder，负责将一个event事件转换成一组byte数组，并将转换后的字节数据输出到文件中。Encoder负责把事件转换为字节数组，并把字节数组写到合适的输出流。因此，encoder可以控制在什么时候、把什么样的字节数组写入到其拥有者维护的输出流中。Encoder接口有两个实现类，LayoutWrappingEncoder与PatternLayoutEncoder。注意：在logback 0.9.19 版之前没有 encoder。在之前的版本里，多数 appender 依靠 layout 来把事件转换成字符串并用java.io.Writer 把字符串输出。在之前的版本里，用户需要在FileAppender里嵌入一个 PatternLayout。

**Level**：Logback日志级别如下（默认级别：DEBUG）：

```java
// ERROR(错误信息) > WARN(警告信息) > INFO(重要信息) > DEBUG(普通信息) > TRACE(追踪信息) DEBUG为默认的打印级别
public static final Integer OFF_INTEGER = 2147483647;  // OFF：最高等级的而级别，用于关闭所有的日志记录
public static final Integer ERROR_INTEGER = 40000;     // ERROR：记录错误信息，不会影响系统运行。一般不想输出太多日志，则使用该级别即可
public static final Integer WARN_INTEGER = 30000;      // WARN：记录警告信息
public static final Integer INFO_INTEGER = 20000;      // INFO：记录运行信息
public static final Integer DEBUG_INTEGER = 10000;     // DEBUG：（默认）记录调式信息，一般在开发中使用
public static final Integer TRACE_INTEGER = 5000;      // TRACE：记录追踪信息，记录程序的流程信息，一般情况不会使用
public static final Integer ALL_INTEGER = -2147483648; // ALL：最低等级，用于打开所有级别的日志记录
```

**LoggerContext**：可以在代码中直接获取LoggerContext，通过LoggerContext可以动态修改Logback的配置

```java
import org.sflf4j.LoggerFactory;
import ch.qos.logback.classic.LoggerContext;

LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
// 根据包路径或类路径获取日志记录器，"ROOT" 表示根记录器: Logger getLogger(final String name);
loggerContext.getLogger(Logger.ROOT_LOGGER_NAME).setLevel(Level.toLevel("error"));
// 停止logback-classic,也就是停止Logback
loggerContext.stop();
```



### 2、配置文件加载

Logback提供了三种种配置文件：logback.groovy、logback-test.xml、logback.xml（位于src/main/resources目录下）如果都不存在则采用默认的配置

1. Logback首先会试着查找logback.groovy文件
2. 当没有找到时，继续试着查找logback-test.xml文件
3. 当没有找到时，继续试着查找logback.xml文件
4. 如果仍然没有找到，则使用默认配置（打印到控制台）
5. 其加载顺序为（优先级从高到低）：logback.groovy =》 logback-test.xml =》 logback.xml

对XML日志文件的配置，大多数人第一次接触时有一种望而生畏的感觉，其实如果仔细分析，会发现核心的部分只有三个元素：appender、logger、root

```
                              ┌── 日志输出格式
               ┌───appender───┼── 日志过滤
               │              └── 日志文件处理
               │              ┌── 获取哪些(包)的日志
               ├────logger────┤                     	──┐
               │              └── 日志的输出级别            ├ 日志组件(root,logger)
               │              ┌── 根节点                 ──┘
configuration──┼─────root─────┤
               │              └── 只有level属性
               │
               │─────property──── 定义变量(非必须)
               │
               └───contextName─── 应用上下文名称(非必须)
```



### 3、XML配置模板

模板 1：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- scan:当此属性设置为true时，配置文档如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文档是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。
                 当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<!--
总体说明：根节点下有2个属性，三个节点
属性： contextName 上下文名称； property 设置变量
节点： appender,  root， logger
-->
<configuration scan="true" scanPeriod="10 seconds">
    <!--
           contextName说明：
           每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用设置成其他名字，
           用于区分不同应用程序的记录。一旦设置，不能修改,可以通过%contextName来打印日志上下文名称。
         -->
    <contextName>logback-spring</contextName>
    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义后，可以使“${}”来使用变量。 -->
    <property name="logging.path" value="myLogs"/>

    <!--0. 日志格式和颜色渲染 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr"
                    converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <!--1. 输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>

        <!--日志文档输出格式-->
        <encoder>
            <!--指定日志格式-->
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <!--设置字符集-->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--输出到文档-->
    <!-- 时间滚动输出 level为 DEBUG 日志 -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名~~~~~file设置打印的文件的路径及文件名，建议绝对路径-->
        <file>${logging.path}/web_debug.log</file>

        <!--日志文档输出格式-->
        <encoder>
            <!--指定日志格式-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>

        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <!--
            日志记录器的滚动策略
            SizeAndTimeBasedRollingPolicy 按日期，大小记录日志
            另外一种方式：
                rollingPolicy的class设置为ch.qos.logback.core.rolling.TimeBasedRollingPolicy

        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志归档 -->
            <!--
                归档的日志文件的路径，例如今天是2018-08-23日志，当前写的日志文件路径为file节点指定，
                可以将此文件与file指定文件路径设置为不同路径，从而将当前日志文件或归档日志文件置不同的目录。
                而2018-08-23的日志文件在由fileNamePattern指定。%d{yyyy-MM-dd}指定日期格式，%i指定索引
             -->
            <fileNamePattern>${logging.path}/web-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>

            <!--
               配置日志文件不能超过100M，若超过100M，日志文件会以索引0开始，命名日志文件
               例如error.20180823.0.txt
               -->
            <timeBasedFileNamingAndTriggeringPolicy
                                                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>

            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>

        <!-- 此日志文档只记录debug级别的 -->
        <!-- 过滤策略：
            LevelFilter ： 只打印level标签设置的日志级别
            ThresholdFilter：打印大于等于level标签设置的级别，小的舍弃
         -->
        <!--<filter class="ch.qos.logback.classic.filter.ThresholdFilter">-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的日志级别 -->
            <level>debug</level>
            <!--匹配到就允许-->
            <onMatch>ACCEPT</onMatch>
            <!--没有匹配到就禁止-->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <!-- 2.2 level为 INFO 日志，时间滚动输出  -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${logging.path}/web_info.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${logging.path}/web-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                                                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.3 level为 WARN 日志，时间滚动输出  -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${logging.path}/web_warn.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logging.path}/web-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                                                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录warn级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.4 level为 ERROR 日志，时间滚动输出  -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${logging.path}/web_error.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logging.path}/web-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                                                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录ERROR级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--
      <logger>用来设置某一个包或者具体的某一个类的日志打印级别、
      以及指定<appender>。<logger>仅有一个name属性，
      一个可选的level和一个可选的addtivity属性。
      name:用来指定受此logger约束的某一个包或者具体的某一个类。
      level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
            还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。
            如果未设置此属性，那么当前logger将会继承上级的级别。
      addtivity:是否向上级logger传递打印信息。默认是true。
      <logger name="org.springframework.web" level="info"/>
      <logger name="org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor" level="INFO"/>
  -->

    <!--
        使用mybatis的时候，sql语句是debug下才会打印，而这里我们只配置了info，所以想要查看sql语句的话，有以下两种操作：
        第一种把<root level="info">改成<root level="DEBUG">这样就会打印sql，不过这样日志那边会出现很多其他消息
        第二种就是单独给dao下目录配置debug模式，代码如下，这样配置sql语句会打印，其他还是正常info级别：
        【logging.level.org.mybatis=debug logging.level.dao=debug】
     -->

    <!--
        root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
        不能设置为INHERITED或者同义词NULL。默认是DEBUG
        可以包含零个或多个元素，标识这个appender将会添加到这个logger。
    -->

    <!-- 4. 最终的策略 -->
    <!-- 4.1 开发环境:打印控制台-->
    <!-- <springProfile name="dev">
        <logger name="com.cic.analysis.business.dao" level="debug"/>&lt;!&ndash; 修改此处扫描包名 &ndash;&gt;
    </springProfile>-->

    <!--
        root指定最基础的日志输出级别，level属性指定
        appender-ref标识的appender将会添加到这个logger
    -->
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="DEBUG_FILE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="WARN_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

    <!-- 4.2 生产环境:输出到文档
    <springProfile name="pro">
        <root level="info">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
        </root>
    </springProfile> -->

</configuration>
```

模板 2：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 属性描述 scan：设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
    debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 定义日志文件 输入位置 -->
    <property name="log_dir" value="logs/" />
    <!-- 日志最大的历史 30天 -->
    <property name="maxHistory" value="30"/>

    <!-- ConsoleAppender 控制台输出日志 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 对日志进行格式化 -->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger -%msg%n</pattern>
        </encoder>
    </appender>

    <!-- ERROR级别日志 -->
    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤器，只记录ERROR级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <!-- 最常用的滚动策略，它根据时间来制定滚动策略.既负责滚动也负责出发滚动 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志输出位置  可相对、和绝对路径 -->
            <fileNamePattern>${log_dir}/%d{yyyy-MM-dd}/error-log.log</fileNamePattern>
            <!-- 最大保存30天-->
            <maxHistory>${maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- WARN级别日志 appender -->
    <appender name="WARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤器，只记录WARN级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 按天回滚 daily -->
            <fileNamePattern>${log_dir}/%d{yyyy-MM-dd}/warn-log.log
            </fileNamePattern>
            <!-- 日志最大的历史 30天 -->
            <maxHistory>${maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- INFO级别日志 appender -->
    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤器，只记录INFO级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 按天回滚 daily -->
            <fileNamePattern>${log_dir}/%d{yyyy-MM-dd}/info-log.log
            </fileNamePattern>
            <!-- 日志最大的历史 30天 -->
            <maxHistory>${maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="DruidFILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名 -->
            <FileNamePattern>${log_dir}/settle-query.log-druid-%d{yyyy-MM-dd}</FileNamePattern>
            <!--日志文件保留天数 -->
            <MaxHistory>90</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
            </pattern>
        </encoder>
    </appender>

    <!-- DEBUG级别日志 appender -->
    <appender name="DEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤器，只记录DEBUG级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 按天回滚 daily -->
            <fileNamePattern>${log_dir}/%d{yyyy-MM-dd}/debug-log.log
            </fileNamePattern>
            <!-- 日志最大的历史 30天 -->
            <maxHistory>${maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- TRACE级别日志 appender -->
    <appender name="TRACE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤器，只记录ERROR级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>TRACE</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 按天回滚 daily -->
            <fileNamePattern>${log_dir}/%d{yyyy-MM-dd}/trace-log.log
            </fileNamePattern>
            <!-- 日志最大的历史 30天 -->
            <maxHistory>${maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 指定其他logger的Appender -->
    <logger name="org.springframework" value="WARN" />
    <logger name="org.apache" value="WARN" />
    <logger name="com.netflix" value="WARN" />
    <logger name="me.chanjar" value="WARN" />
    <logger name="com.ulisesbocchio" value="WARN" />
    <logger name="org.hibernate" value="WARN" />
    <logger name="com.github.liuweijw" value="INFO" />
    <logger name="c.n.discovery" value="WARN" />
    <logger name="o.s.c.s" value="WARN" />
    <logger name="c.u.j.encryptor" value="WARN" />
    <logger name="o.s.boot" value="WARN" />
    <!-- 指定druid的日志级别为DEBUG，除了继承rootLogger的appender外，自己还加了DruidFILE -->
    <logger name="druid" level="DEBUG">
        <appender-ref ref="DruidFILE" />
    </logger>
    <!-- 指定rootLogger级别   INFO，appender为STDOUT，ERROR,INFO-->
    <root level="INFO">
        <!-- 控制台输出 -->
        <appender-ref ref="STDOUT" />
        <!-- 文件输出 -->
        <appender-ref ref="ERROR" />
        <appender-ref ref="INFO" />
    </root>
</configuration>
```



## 3、配置文件结构详解

Logback的配置，需要配置输出源Appender，打日志的Logger（子节点）和Root（根节点）实际上，它输出日志是从子节点开始，子节点如果有输出源直接输出，如果无，判断配置的additivity，是否向上级传递，即是否向root传递，传递则采用Root的输出源，否则不输出日志。

logback.xml配置文件根结点为configuration，内主要包含contextName、property、appender、filter、pattern、logger、root 等结点。基本结构可以描述为1个configuration元素，包含零个或多个appender元素，后跟零个或多个logger元素，后跟最多1个root元素（也可以没有）

### 01、configuration 根元素

根元素configuration有三个属性：debug、 scan、scanPeriod，及一个特殊的属性：packagingData，示例及解释如下：

- debug：默认为false，若设置为true，则打印出logback内部运行日志信息
- scan：默认值为true，若设置为true，配置文件如果发生改变，将会自动重新加载配置文件
- scanPeriod：与scan配合使用，当scan为true时，此属性生效，默认的时间间隔为1分钟，设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。可以设置为scanPeriod="30 seconds"每30秒检测一次。单位有：milliseconds、seconds、munites、hours，默认为milliseconds
  注意：如果是在本地 IDE 编辑器中测试，注意修改的是 classes 编译目录下的 logback.xml 文件，而不是 resources 目录下的源文件（一般不用）
- packagingData：在堆栈跟踪中启用包数据。注意从版本1.1.4开始，包装数据默认为禁用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- debug : 开启logback运行日志输出，true为始终输出,false为出错时才输出-->
<!-- scan ：开启"热更新"-->
<!-- scanPeriod："热更新"扫描周期，默认 60 seconds(60秒) 单位有:milliseconds、seconds、munites、hours，默认为milliseconds-->
<configuration debug="true" scan="true" scanPeriod="60 seconds">
</configuration>    
```

开启packagingDatas配置：可按如下配置开启包数据（配置文件或者代码形式取其一即可）：

```xml
<configuration packagingData="true">
</configuration>
```

```java
import org.sflf4j.LoggerFactory;
import ch.qos.logback.classic.LoggerContext;

LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
loggerContext.setPackagingDataEnabled(true);
```

如果开启了，logback 会在输出的堆栈行中显示它是属于哪个 jar 或者哪个类的。此信息由 jar 文件的名称和版本组成，表明堆栈信息来源于此。此机制对于识别软件版本问题非常有用。但是，计算成本相当昂贵，尤其是在经常引发异常的应用程序中。以下演示开启的结果，即多了 [] 括号内的信息。

```java
14:28:48.835 [btpool0-7] INFO  c.q.l.demo.prime.PrimeAction - 99 is not a valid value
java.lang.Exception: 99 is invalid
  at ch.qos.logback.demo.prime.PrimeAction.execute(PrimeAction.java:28) [classes/:na]
  at org.apache.struts.action.RequestProcessor.processActionPerform(RequestProcessor.java:431) [struts-1.2.9.jar:1.2.9]
  at org.apache.struts.action.RequestProcessor.process(RequestProcessor.java:236) [struts-1.2.9.jar:1.2.9]
  at org.apache.struts.action.ActionServlet.doPost(ActionServlet.java:432) [struts-1.2.9.jar:1.2.9]
  at javax.servlet.http.HttpServlet.service(HttpServlet.java:820) [servlet-api-2.5-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.servlet.ServletHolder.handle(ServletHolder.java:502) [jetty-6.1.12.jar:6.1.12]
  at ch.qos.logback.demo.UserServletFilter.doFilter(UserServletFilter.java:44) [classes/:na]
  at org.mortbay.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1115) [jetty-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.servlet.ServletHandler.handle(ServletHandler.java:361) [jetty-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.webapp.WebAppContext.handle(WebAppContext.java:417) [jetty-6.1.12.jar:6.1.12]
  at org.mortbay.jetty.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:230) [jetty-6.1.12.jar:6.1.12]
```





### 02、contextName 元素

日志上下文名称：contextName，每个日志组件（logger）都会关联到日志上下文，默认上下文名称是：default，用于标识应用，如果多个应用输出到同一个地方，就有必要使用`%contextName`来区别。contextName直接在configuration元素下：

```xml
<configuration>
    <!--定义后,在其他property属性或appender中便可通过”%contextName”来获取和使用该上下文名称了-->
    <contextName>HelloWorld-log</contextName>
</configuration>
```



### 03、property 属性元素

1、用来定义变量值的标签，通过property定义的值会被插入到logger上下文中。property标签中有: name、value 两个属性，属性含义如下：

- name：值是变量的名称
- value：值是变量定义的值
- 定义变量后，可以使`${name}`来使用变量

```xml
<configuration>
    <!--定义变量-->
    <property name="logName" value="logbackStudy" />
    <!--从src/main/resources加载配置文件(文件内key=val形式每行一个), 配置文件中的值可直接使用-->
    <property resource="logback.properties" />
    <!--file获取绝对路径-->
    <property file="D:\\logback.properties" />
</configuration>
```

2、如果是在Spring或SpringBoot项目当中，想让value值是通过配置文件获取，可使用springProperty来定义。这里是获取项目名称和运行的服务器IP

- scope：作用域
- name：名称，在下面引用的时候的名字
- source：在application.properties文件中的key
- defaultValue：默认值，该属性在application.properties中未设置时，会使用默认值

```xml
<!--其中source指定的spring.application.name便是在application.properties当中配置变量。此配置还是比较常用的，可以做到灵活区分环境-->
<springProperty scope="context" name="appName" source="spring.application.name" />
<springProperty scope="context" name="ip" source="spring.cloud.client.ipAddress" />
```

3、变量作用域（scope）

定义的变量是有作用域的，如本地作用域，上下文作用域，系统级作用域。默认是本地作用域。

1. Local Scope（本地作用域）：从配置文件中定义的本地变量即在本地配置文件使用。每次解析和执行配置文件时，都会重新定义本地作用域中的变量
2. Context Scope（上下文作用域）：一个拥有上下文作用域的变量存在于上下文中，于上下文共存，直到被清除。在所有记录事件中都可用到，包括那些通过序列化发送到远程主机的事件
3. System Scope（系统级作用域）：系统级作用域的变量被插入到JVM的系统属性中，生命周期和JVM一致，直到被清除
4. 在进行属性替换时，查找变量的顺序为：local，context，system 



### 04、timestamp 时间元素

获取时间戳字符串并格式化：timestamp。他有两个属性key和datePattern

1. key：标识timestamp的名字
2. datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式

```xml
<configuration>
    <!-- 实际上也是定义成变量给后面使用，与property类似 -->
    <timestamp key="nowDate" datePattern="yyyyMMdd" />
    <timestamp key="nowDateTime" datePattern="yyyyMMdd_HHmmss" />
    <contextName>${nowDateTime}</contextName>  
</configuration>
```



### 05、conversionRule 元素

主要用来配置转换器，可以自定义Pattern模板，也可以引入一些其他模板来解析Pattern，比如打印彩色日志，打印IP地址。conversionRule 元素中有两个参数，解释如下：

- conversionWord：在Pattern中的关键字
- converterClass：转换器类

```xml
<!-- 彩色日志,需要在idea中安装grep console插件 -->
<!-- 彩色日志依赖的渲染类 -->
<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />

<!-- 彩色日志格式 -->
<property name="COLOR_LOG_PATTERN" value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(%p) %clr(${PID}){magenta} %clr([%.10t]){yellow} %clr([%c{1}]){blue}%clr([%L]){faint}%X{traceId} %m%n"/>
```

上面`%clr`就是要用`ColorConverter`来解析，后面`{blue}`就是要展示的颜色。



### 06、appender 日志输出元素

#### Appender 的介绍

官方文档为：https://logback.qos.ch/manual/appenders.html

Appender 的组件有两个属性：name和class，并且需要强制赋值，解释如下：

- name：定义写日志组件的唯一名称，可以随便定义，后面使用该appender是也是通过名称来指定
- class：指定要实例化的appender类的完全限定名称

Appender种类有很多，常用的种类有：ConsoleAppender（输出到控制台）、FileAppender（输出到文件）、SMTPAppender（输出到邮件）、DBAppender（输出到数据库）、AsyncAppender（异步输出，包装其它具体的appender，不单独使用）等，如下介绍常用的

- ConsoleAppender：日志输出到控制台，类名：ch.qos.logback.core.ConsoleAppender
- FileAppender：日志输入到文件，类名：ch.qos.logback.core.FileAppender
- RollingFileAppender：滚动记录文件（FileAppender的子类）当符合条件（大小、时间）日志进行切分处理。类名：ch.qos.logback.core.rolling.RollingFileAppender
- 实战中ConsoleAppender及RollingFileAppender使用较多，若需要自定义如把日志输出到消息队列，可以自定义实现AppenderBase接口

除了上述Appender外，另外还有SocketAppender（输出到socket）、SMTPAppender（输出到邮件)、DBAppender（输出到数据库）等具体使用查文档



#### ConsoleAppender

ConsoleAppender 就跟名字显示的一样，是将日志事件附加到控制台，进一步说就是通过 System.out（默认） 或 System.err 来进行输出。ConsoleAppender 通过用户指定的 encoder，格式化日志事件。

| 属性名    | 类型    | 描述                                                         |
| :-------- | :------ | :----------------------------------------------------------- |
| target    | String  | System.out 或 System.err。默认为 System.out                  |
| encoder   | Encoder | 决定通过哪种方式将事件写入 OutputStreamAppender              |
| withJansi | boolean | 默认值为 false。设为 true 可以激活 Jansi 在 windows 使用 ANSI 彩色代码。<br />在 windows 上如果设置为 true，你应该将 `org.fusesource.jansi:jansi:1.9` 这个 jar 包放到 classpath 下。<br />基于 Unix 实现的操作系统，像 Linux、Max OS X 都默认支持 ANSI 才彩色代码。 |

```xml
<configuration>
    <!--日志输出器:输出到控制台-->
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <!--控制输出流对象, 默认为 System.out 是黑色字体, System.err 是红色字体 -->
        <target>System.out</target>
        <!-- 输出的日志消息格式配置, encoder 默认使用 ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d [%t] [%-5level] %c %M %L %m %n</pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <!-- 根记录器 -->
    <root level="INFO">
        <!--输出记录的日志-->
        <appender-ref ref="consoleAppender"/>
    </root>
</configuration>
```



#### FileAppender

FileAppender是OutputStreamAppender的子类，将日志事件输出到文件中。通过file来指定目标文件。如果该文件存在，根据append的值，要么将日志追加到文件中，要么该文件被截断。

| 属性名  | 类型    | 描述                                                         |
| :------ | :------ | :----------------------------------------------------------- |
| file    | String  | 要写入文件的名称。如果文件不存在，则新建                     |
| encoder | Encoder | 参见 ConsoleAppender 的属性                                  |
| append  | boolean | 如果为 true，日志事件会被追加到文件中，否则的话，清空现存文件。默认为 true |
| prudent | boolean | 在严格模式下，FileAppender会将日志安全的写入指定文件。即使在不同的 JVM 或者不同的主机上运行 FileAppender 实例。默认值为 false。严格模式可以与 RollingFileAppender 结合使用 |

1、将日志输出到文件：FileAppender。包含file（文件名）、encoder（格式化）元素

```xml
<configuration>
    <!--定义变量, 通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量-->
    <property name="pattern" value="%d [%t] [%-5level] %c %M %L %m %n"></property>
    <property name="log_file" value="./logs"></property>
    <!-- 通过 "bySecond" 将时间格式化成 "yyyyMMdd'T'HHmmss" 的形式插入到 logger 的上下文中这个值对后续的配置也适用-->
    <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss" />

    <!--日志输出器:输出到文件-->
    <appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
        <!-- 利用之前创建的 timestamp 来创建唯一的文件, 生成的文件名: log-20220402T122403.log -->
        <file>${log_file}/logback-${bySecond}.log</file>
        <!-- 将 immediateFlush 设置为 false 可以获得更高的日志吞吐量 -->
        <immediateFlush>true</immediateFlush>
        <!-- 输出的日志消息格式配置, encoder 默认使用 ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
        <encoder>
            <pattern>${pattern}</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="fileAppender"/>
    </root>
</configuration>
```

2、输出到html格式的日志文件：appender设置为FileAppender，encoder设置为LayoutWrappingEncoder，layout设置为LayoutWrappingEncoder

```xml
<configuration>
    <!--name是变量的名称，value是变量定义的值。通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量-->
    <property name="html_pattern" value="%d{yyyy-MM-dd HH:mm:ss}%t%-5level%c%M%L%m%n"></property>
    <property name="log_file" value="./logs"></property>

    <!--输出到html格式的日志文件, 这个也属于输出到指定文件, 还是使用FileAppender类-->
    <appender name="htmlFileAppender" class="ch.qos.logback.core.FileAppender">
        <!-- 日志文件保存路径-->
        <file>${log_file}/logback.html</file>
        <!-- html消息格式配置-->
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="ch.qos.logback.classic.html.HTMLLayout">
                <pattern>${html_pattern}</pattern>
            </layout>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="htmlFileAppender"/>
    </root>
</configuration>
```

注意：我们只能设置打印的日志信息内容，不能设置这个网页的打印格式以及样式。但是，当我们打印出logback.html文件后，我们可以人为的修改其中的样式以及格式这个html中包含 HTML+CSS。在实际的开发中，如果日志文件不是很大，我们可以考虑使用html进行日志打印，因为可读性强。




#### RollingFileAppender

RollingFileAppender是FileAppender的子类，扩展了FileAppender，具有翻转日志文件的功能。当满足某个特定的条件之后，将日志输出到另外一个文件。

| 属性名           | 类型             | 描述                                                         |
| ---------------- | ---------------- | ------------------------------------------------------------ |
| file             | String           | 被写入文件名，可以是相对目录或绝对目录，如果上级目录不存在会自动创建，无默认值 |
| rollingPolicy    | RollingPolicy    | 当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名 |
| triggeringPolicy | TriggeringPolicy | 告知 RollingFileAppender 合适激活滚动                        |
| encoder          | Encoder          | 参见 ConsoleAppender 的属性                                  |
| append           | boolean          | 如果为 true，日志事件会被追加到文件中，否则的话，清空现存文件。默认为 true |
| prudent          | boolean          | 当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，<br />1.不支持也不允许文件压缩，2.不能设置file属性，必须留空 |



##### 1、RollingPolicy滚动策略

RollingPolicy用于配置滚动策略，支持TimeBasedRollingPolicy（基于时间滚动）、SizeAndTimeBasedRollingPolicy（基于大小和时间滚动）、FixedWindowRollingPolicy（固定窗口滚动和大小触发）、其中FixedWindowRollingPolicy和SizeBasedTriggeringPolicy一般同时使用

RollingPolicy滚动策略包括以下几种：

- TimeBasedRollingPolicy：基于时间的滚动策略
- SizeAndTimeBasedRollingPolicy：基于文件大小和时间的滚动策略
- FixedWindowRollingPolicy：基于固定窗口大小的滚动策略，就是将归档日志文件到最大了就写到下一个文件里，而窗口大小就是最多允许日志文件数
- SizeBasedTriggeringPolicy：FixedWindowRollingPolicy和SizeBasedTriggeringPolicy一般同时使用，用于触发FixedWindowRollingPolicy



##### 2、TimeBasedRollingPolicy

TimeBasedRollingPolicy：它是基于时间来定义轮转策（最常用的轮转策略）例如按天或月。TimeBasedRollingPolicy 既负责轮转的行为，也负责触发轮转。实际上：TimeBasedRollingPolicy 同时实现了 RollingPolicy 与 TriggeringPolicy 接口。

TimeBasedRollingPolicy 的配置需要一个强制的属性 fileNamePattern 以及其它的可选属性：

| 属性名              | 类型    | 描述                                                         |
| :------------------ | :------ | :----------------------------------------------------------- |
| fileNamePattern     | String  | 该属性的值应该由文件名加上一个 `%d`的占位符。`%d` 应包含 `java.text.SimpleDateFormat` 中规定的日期格式。如果省略日期格式，那么就默认为 `yyyy-MM-dd`。如果 fileNamePattern 以 .gz 或者 .zip 结尾，将会启动这个压缩特性 |
| maxHistory          | int     | 这个可选的属性用来控制最多保留多少数量的归档文件，将会异步删除旧的文件。如单位按月轮转时maxHistory = 6，只会保留6个月的日志。单位根据时间格式维度来区分：按天(yyyy-MM-dd) 按月(yyyy-MM) |
| totalSizeCap        | int     | 在有 maxHistory 的限制下，进一步限制所有日志文件大小之和的上限，超过则从最旧的日志开始删除 |
| cleanHistoryOnStart | boolean | 如果设置为 true，那么在 appender 启动的时候，归档文件将会被删除。默认的值为 false。归档文件的删除通常在轮转期间执行。但是，有些应用的存活时间可能等不到轮转触发。对于这种短期应用，可以通过设置该属性为 true，在 appender 启动的时候执行删除操作 |

```xml
<configuration>
    <!--日志输出器:基于时间的滚动策略输出到文件-->
    <appender name="rollingFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/logback.log</file>
        <!-- 启用基于时间的滚动策略：TimeBasedRollingPolicy-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 设置轮转策略为:每秒钟轮转 后缀如果设置成.gz或.zip后缀会自动压缩-->
            <fileNamePattern>logs/logback_%d{yyyy-MM-dd_HH-mm-ss}.log</fileNamePattern>
            <!-- 只保留最近1分钟内的日志,超出后会删除旧日志-->
            <maxHistory>60</maxHistory>
            <!-- 所有日志文件大小之和的上限,超出后会删除旧日志-->
            <totalSizeCap>10MB</totalSizeCap>
            <!-- 在appender启动的时候执行删除历史日志操作-->
            <cleanHistoryOnStart>true</cleanHistoryOnStart>
        </rollingPolicy>
        <!-- 默认为 ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
        <encoder>
            <pattern>%date %-5level -[%thread] %class.%method/%line : %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="rollingFileAppender"/>
    </root>
</configuration>
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.Timer;
import java.util.TimerTask;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogbackTest.class);
        // 1秒后启动,每秒跑1次
        new Timer().scheduleAtFixedRate(new TimerTask(){
            @Override
            public void run() {
                logger.error("error test");
                logger.warn("warn test");
                logger.info("info {}", "test");
                logger.debug("debug test ");
                logger.trace("trace test");
            }
        }, 1000, 1000);
    }
}
```

执行后可以发现1分钟后生成了61个文件。1个logback.log，60个logback_2022-04-03_12-12-xx.log。这是因为每秒生产一个文件，而maxHistory设置了保留最近一分钟的日志（并且每个日志文件都比较小，60个加起来也没超过totalSizeCap总日志大小的上限）。此时还可以发生另外一种情况，就是把totalSizeCap总文件大小设置比较小，才生成几个文件就达到totalSizeCap总日志大小的上限。



##### 3、SizeAndTimeBasedRollingPolicy

SizeAndTimeBasedRollingPolicy：基于时间+大小进行滚动（既能按时轮转，又能限制每个日志文件的大小）是对于第一种的补充。避免单个日志文件过大。实际查看源码也可以看出SizeAndTimeBasedRollingPolicy继承了TimeBasedRollingPolicy，并增加了一个maxFileSize属性。

| 属性名              | 类型    | 描述                                                         |
| :------------------ | :------ | :----------------------------------------------------------- |
| fileNamePattern     | String  | 该属性的值应该由文件名加上一个 `%d`的占位符。`%d` 应包含 `java.text.SimpleDateFormat` 中规定的日期格式。如果省略日期格式，那么就默认为 `yyyy-MM-dd`。并且需要包含一个 `i%` 的占位符，当文件大小超过maxFileSize大小时，则用下标新建文件。如果 fileNamePattern 以 .gz 或者 .zip 结尾，将会启动这个压缩特性 |
| maxFileSize         | int     | 按照文件大小进行拆分，当文件大小达到1MB时就会将日志进行压缩  |
| maxHistory          | int     | 这个可选的属性用来控制最多保留多少数量的归档文件，将会异步删除旧的文件。如单位按月轮转时maxHistory = 6，只会保留6个月的日志。单位根据时间格式维度来区分：按天(yyyy-MM-dd) 按月(yyyy-MM) |
| totalSizeCap        | int     | 在有 maxHistory 的限制下，进一步限制所有日志文件大小之和的上限，超过则从最旧的日志开始删除 |
| cleanHistoryOnStart | boolean | 如果设置为 true，那么在 appender 启动的时候，归档文件将会被删除。默认的值为 false。归档文件的删除通常在轮转期间执行。但是，有些应用的存活时间可能等不到轮转触发。对于这种短期应用，可以通过设置该属性为 true，在 appender 启动的时候执行删除操作 |

```xml
<configuration>
    <!--日志输出器:基于文件大小和时间的滚动策略-->
    <appender name="rollingFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 活动中的日志文件名(支持绝对和相对路径) -->
        <file>logs/logback.log</file>
        <!-- 基于时间和文件大小的滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 设置轮转策略为: 每小时轮转, 按照时间和压缩格式来声明文件名-->
            <!-- %i是当文件大小超过maxFileSize大小时, 则用下标新建文件, .gz或.zip后缀会自动压缩-->
            <fileNamePattern>logs/logback_%d{yyyy-MM-dd_HH-mm-ss}_%i.log.zip</fileNamePattern>
            <!-- 每个文件最多1MB, 保留1分钟的历史记录, 但最多200MB-->
            <!-- 单个日志文件的最大大小-->
            <maxFileSize>1MB</maxFileSize>
            <!-- 根据当前轮转策略, 只保留最近1分钟内的日志,超出后会删除旧日志-->
            <maxHistory>60</maxHistory>
            <!-- 所有日志文件大小之和的上限,超出后会删除旧日志-->
            <totalSizeCap>200MB</totalSizeCap>
            <!-- 设置为true并且maxHistory不为零才能起效-->
            <cleanHistoryOnStart>true</cleanHistoryOnStart>
        </rollingPolicy>
        <!-- 日志输出格式, 默认为 ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
        <encoder>
            <pattern>%date %-5level -[%thread] %class.%method/%line : %msg%n</pattern>
        </encoder>
    </appender>
    <!-- 根记录器 -->
    <root level="INFO">
        <appender-ref ref="rollingFileAppender"/>
    </root>
</configuration>
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.Timer;
import java.util.TimerTask;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogbackTest.class);
        // 1秒后启动,每秒跑1次
        new Timer().scheduleAtFixedRate(new TimerTask(){
            @Override
            public void run() {
                logger.error("error test");
                logger.warn("warn test");
                logger.info("info {}", "test");
                logger.debug("debug test ");
                logger.trace("trace test");
            }
        }, 1000, 1000);
    }
}
```

执行后可以发现会生成文件logback.log与文件logback_2022-04-03_17-42-xx_0.log.zip若干个。若在同一个时间段内生成的单个文件超过了maxFileSize后，`%i`就会生效（默认是从0开始）



##### 4、FixedWindowRollingPolicy

可以理解成自定义滚动规则，避免使用出现单个文件过大或者日志文件过多的情况。需要同时配置triggeringPolicy用于指定滚动触发规则。对以上的补充。

FixedWindowRollingPolicy 根据固定窗口算法重命名文件，filaNamePattern 表示归档文件的名字。这个属性是必选，并必须包含一个表示整形的占位符 `i%`

| 属性名          | 类型   | 描述                                                         |
| :-------------- | :----- | :----------------------------------------------------------- |
| fileNamePattern | String | FixedWindowRollingPolicy 在重命名日志文件时将会根据这个属性来命名。它必须包含一个 `i%` 的占位符，该占位符指明了窗口索引的值应该插入的位置。 例如，当该属性的值为 `MyLogFile%i.log`，最小与最大的值分别为 `1` 和 `3`。将会产生的归档文件为 `MyLogFile1.log`，`MyLogFile2.log`，`MyLogFile3.log`。 文件压缩的方式也是通过该属性来指定。例如，设置该属性的值为 `MyLogFile%i.log.zip`，那么归档文件将会被压缩成 `zip` 格式。也可以选择压缩成 `gz` 格式。 |
| minIndex        | int    | 表示窗口索引的下界                                           |
| maxIndex        | int    | 表示窗口索引的上界                                           |

```xml
<configuration>
    <appender name="rollingFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 活动中的日志文件名(支持绝对和相对路径) -->
        <file>logs/logback.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>logs/logback_%i.log.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>5</maxIndex>
        </rollingPolicy>
        <!-- 观察当前活动文件的大小，如果已经大于了指定的值，它会给RollingFileAppender发送信号触发对当前活动文件的轮转-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>10KB</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>%date %-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="rollingFileAppender" />
    </root>
</configuration>
```

TriggeringPolicy 用于通知何时触发滚动。实现类：SizeBasedTriggeringPolicy

- SizeBasedTriggeringPolicy：观察当前活动文件的大小，如果大于了指定的值，它会给RollingFileAppender发一个信号触发对当前活动文件的轮转
- SizeBasedTriggeringPolicy：只接收 maxFileSize 这一个参数，它的默认值是 10MB
- maxFileSize：可以为字节 / 千字节 /兆字节 /千兆字节，数值 加 KB，MB 或 GB。例：5000000，5000KB，5MB（这3个大小一样） 及 2GB

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.Timer;
import java.util.TimerTask;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogbackTest.class);
        // 1秒后启动,每秒跑1次
        new Timer().scheduleAtFixedRate(new TimerTask(){
            @Override
            public void run() {
                logger.error("error test");
                logger.warn("warn test");
                logger.info("info {}", "test");
                logger.debug("debug test ");
                logger.trace("trace test");
            }
        }, 1000, 1000);
    }
}
```

执行代码可以发现：最后生成文件：logback.log、logback_1.log.zip - logback_5.log.zip（zip解压后可以发现文件名：logback_2022-04-03_1754，具体可以参考源码：FixedWindowRollingPolicy）如果fileNamePattern中没有设置以.zip结尾的话，默认名称就是设定名称+Index



##### 5、轮转周期示例 fileNamePattern

适用上面的 SizeAndTimeBasedRollingPolicy 和 TimeBasedRollingPolicy 滚动策略`<fileNamePattern></fileNamePattern>`

| fileNamePattern 格式              | 说明                                                         |
| :-------------------------------- | :----------------------------------------------------------- |
| app_%d.log                        | 每天轮转。不指定`%d`日期格式时默认为 yyyy-MM-dd              |
| app_%d{yyyy-MM}.log               | 每个月开始的时候轮转                                         |
| app._%d{yyyy-ww}.log              | 每周的第一天（取决于时区）                                   |
| app_%d{yyyy-MM-dd_HH}.log         | 每小时轮转。如：app_2020-10-24_10.log                        |
| app_%d{yyyy-MM-dd_HH-mm}.log      | 每分钟轮转;。如：app_2020-10-24_10-32.log                    |
| app_%d{yyyy-MM-dd_HH-mm, UTC}.log | 每分钟轮转（时间格式是 UTC）                                 |
| app/%d{yyyy-MM}/%d.log            | 每天轮转。第一个`%d`被辅助标记。第二个`%d`为主要标记。如：app/2020-10/2020-10-24.log |

> **重点注意**：fileNamePattern中设置的时间格式生产文件名不能使用空格已经冒号等特殊符号（不然不会生产新的滚动文件）



#### AsyncAppender 异步

上面的Appender日志输出到文件是同步输出的，即每次输出都会直接写IO到磁盘文件。对于高并发，会产生一定的阻塞，造成不必要的性能损耗。Logback提供了日志异步输出的AsyncAppender。处理方式较简单，添加一个基于异步写日志的appender，并指向原配置的appender即可。AsyncAppender 参数说明：

| 参数                | 默认值 | 说明                                                         |
| ------------------- | ------ | ------------------------------------------------------------ |
| discardingThreshold | 20     | 如果设置discardingThreshold＝0，表示 queue 满了，不丢弃，block 线程。默认情况下，当阻塞队列剩余 20% 的容量时，它将丢弃级别跟踪、调试和信息事件，只保留级别警告和错误事件。要保留所有事件，请将 discardingThreshold 设置为0 |
| queueSize           | 256    | 假设 IO 影响 30s，日志和 qps 比例是1:1，单容器压测值 1500 qps 则可以推算出 queueSize 的值，queueSize 的设置公式：30 ＊ 1500=45000 |
| neverBlock          | false  | 如果为 false（默认值），则追加程序将阻止追加到完整队列，而不是丢失消息。设置为 true 时，附加程序只会丢弃消息，不会阻止您的应用程序 |
| includeCallerData   | false  | 提取呼叫者数据可能相当昂贵。 为了提高性能，默认情况下，当事件添加到事件队列时，不会提取与事件关联的调用者数据。 默认情况下，仅复制线程名称和 MDC 等“廉价”数据。 您可以通过将 includeCallerData 属性设置为 true 来指示此附加程序包含调用方数据。 |

```xml
<configuration>
    <!--输出到文件-->
    <appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
        <file>logs/logback.log</file>
        <!--提高高的日志吞吐量-->
        <immediateFlush>false</immediateFlush>
        <encoder>
            <pattern>${pattern}</pattern>
        </encoder>
    </appender>
    
    <!--异步输出-->
    <appender name="asyncAppender" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 默认如果队列的80%已满,则会丢弃TRACT,DEBUG,INFO级别的日志,若要保留全部日志,设置为0 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能,默认值为256 -->
        <queueSize>1024</queueSize>
        <!-- 设置附加程序只会丢弃消息，不会阻止您的应用程序-->
        <neverBlock>true</neverBlock>
        <!--是否提取调用者数据-->
        <includeCallerData>true</includeCallerData>
        <!-- 添加附加的appender,最多只能添加1个 -->
        <appender-ref ref="fileAppender"/>
    </appender>
    
    <!-- 异步输出关联到root根logger -->
    <root level="INFO">
        <appender-ref ref="asyncAppender" />
    </root>
</configuration>
```

这里有个需要注意的地方，那就是 AsyncAppender 必须在其引用的 Appender 配置的后面，否则会使配置不生效。



#### SMTPAppender 邮件

> 使用SpringBoot + Logback 配置程序异常自动发送邮件：https://mp.weixin.qq.com/s/N4OrKlrtCDeEd95XWGi9YA

现在想更要发生异常自动发送邮件的功能，需要使用到 SMTPAppender 输出日志到邮件组件

1、加入依赖（JavaMail）

```xml
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.4.7</version>
</dependency>
```

2、修改logback的xml的配置文件

```xml
<configuration debug="true">
    <!--日志输出器:输出到控制台-->
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d [%t] [%-5level] %c %M %L %m %n</pattern>
        </encoder>
    </appender>

    <!--邮件设置-->
    <property name="smtpHost" value="smtp.qq.com"/>
    <!--QQ邮箱的SMTP(SLL)端口为465或587,其他邮箱自行去查看)-->
    <property name="smtpPort" value="587"/>
    <property name="mailFrom" value="xxx@qq.com"/>
    <property name="username" value="xxx@qq.com"/>
    <property name="mailTo" value="xxx@qq.com,xxx@qq.com"/>
    <property name="password" value="fxomimpmgpjgbhxx"/>
    <property name="SSL" value="false"/>
    <property name="mailSubject" value="Exception information reminder"/>
    <appender name="mailAppender" class="ch.qos.logback.classic.net.SMTPAppender">
        <smtpHost>${smtpHost}</smtpHost>
        <smtpPort>${smtpPort}</smtpPort>
        <username>${username}</username>
        <password>${password}</password>
        <SSL>${SSL}</SSL>
        <asynchronousSending>false</asynchronousSending>
        <to>${mailTo}</to>
        <from>${mailFrom}</from>
        <subject>${mailSubject}</subject>
        <layout class="ch.qos.logback.classic.html.HTMLLayout">
            <!-- 这里配置的是邮件表格的形式 跟截图的邮件格式是一样 其实不加这一块也是样式也是一样的  但是可以自己研究自定义样式-->
            <Pattern>%date%level%thread%logger{0}%line%message</Pattern>
        </layout>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>

    <root level="INFO">
        <appender-ref ref="consoleAppender"/>
        <appender-ref ref="mailAppender"/>
    </root>
</configuration>
```

3、测试代码，输出ERROR级别日志，会自动发送到邮件中

```java
package com.xyz;

import ch.qos.logback.classic.Logger;
import org.slf4j.LoggerFactory;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = (Logger)LoggerFactory.getLogger(LogbackTest.class);

        String s = null;
        try {
            s.equals("");
        } catch (Exception e) {
            logger.error("error message: {}",e);
        }
    }
}
```

4、邮件中收到内容如下：

<table>
	<thead>
		<tr>
			<th>Date</th>
            <th>Level</th>
            <th>Thread</th>
            <th>Logger</th>
            <th>LineOfCaller</th>
            <th>Message</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>2022-04-06 11:37:18,549</td>
			<td>ERROR</td>
			<td>main</td>
            <td>LogbackTest</td>
			<td>14</td>
			<td>error message: {}</td>
		</tr>
		<tr>
            <td colspan=6>java.lang.NullPointerException: null<br/>at com.xyz.LogbackTest.main(LogbackTest.java:14)</td>
		</tr>
	</tbody>
</table>



#### Logback + ELK 配置

GitHub地址：https://github.com/logfellow/logstash-logback-encoder/blob/main/README.md

如果项目中的日志采用的是基于ELK（Elasticsearch、Logstash、Kibana）来进行日志管理。则可以在pom文件中引入logstash-logback-encoder依赖

```xml
<!--由于配置中使用了json格式的日志输出，所以需要引入如下依赖-->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>5.1</version>
</dependency>
```

然后在logback-spring.xml中配置对应的appender：

```xml
<!--logstash 配置部分-->
<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <!-- 发送到Logstash提供的服务地址 -->
    <destination>192.168.0.11:5061</destination>
    <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"project": "springboot-logback-elk"}</customFields>
    </encoder>
</appender>
```

该appender的使用与其他appender的使用无异。主要使用了LogstashTcpSocketAppender类来完成与Logstash的通信。其中destination为Logstash提供的服务地址。customFields为自定义的参数，便于Logstash识别日志是从哪个业务系统传输过来的。

如下是参考一个详细（完整的logback配置示例ELK整合包含生成json日志：https://www.zhangshengrong.com/p/OgN5Dvvxan/）

```xml
<appender name="logstash" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logs/logback.json</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logs/logback-%d{yyyy-MM-dd}.json</fileNamePattern>
        <maxHistory>7</maxHistory>
    </rollingPolicy>
    <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
        <providers>
            <timestamp>
                <timeZone>UTC</timeZone>
            </timestamp>
            <pattern>
                <pattern>
                    {
                    "ip": "${ip}",
                    "app": "${appName}",
                    "level": "%level",
                    "trace": "%X{X-B3-TraceId:-}",
                    "span": "%X{X-B3-SpanId:-}",
                    "parent": "%X{X-B3-ParentSpanId:-}",
                    "thread": "%thread",
                    "class": "%logger{40}",
                    "message": "%message",
                    "stack_trace": "%exception{10}"
                    }
                </pattern>
            </pattern>
        </providers>
    </encoder>
</appender>
```





### 07、encoder & pattern元素

官网地址：

- https://logback.qos.ch/manual/encoders.html
- https://logback.qos.ch/manual/layouts.html
- https://logback.qos.ch/manual/layouts.html#conversionWord

encoder和layout一样，用来格式化输出。encoder是在logback 0.9.19 版本之后引入的。在这个版本之前，大部分的appender使用layout来格式化输出，但之后FileAppender和他的子类推荐使用encoder来代替layout。常用的是PatternLayoutEncoder（这是默认的encoder）所以我们不用指定，直接使用encoder标签就行。PatternLayoutEncoder的组成和layout一样。如下：

**Pattern表达式**：

| Conversation Word                             | Effect                                                       |
| --------------------------------------------- | ------------------------------------------------------------ |
| d{pattern}、date{pattern}                     | 日志打印的时间，语法与java.text.SimpleDateFormat兼容。Conversion Pattern Result |
| %d                                            | 2006-10-20 14:06:49,812                                      |
| %date                                         | 2006-10-20 14:06:49,812                                      |
| %date{ISO8601}                                | 2006-10-20 14:06:49,812                                      |
| %date{HH:mm:ss.SSS}                           | 14:06:49.812                                                 |
| %date{dd MMM yyyy ;HH:mm:ss.SSS}              | 20 oct. 2006;14:06:49.812                                    |
| %d{dd MMM yyyy ;HH:mm:ss.SSS,Australia/Perth} | 05 4月 2022 ;11:34:13.168                                    |

| Conversation Word            | Effect                                                       |
| ---------------------------- | ------------------------------------------------------------ |
| **变量名**                   | %date 时间打印详情请参考上方                                 |
| %date{pattern, timezone}、%d | 日志打印的时间。pattern与java.text.SimpleDateFormat兼容<br />pattern不填写默认值是：yyyy-MM-dd hh:mm:ss,SSS。<br />第二个参数是时区，例如：%date{HH:mm:ss.SSS, Australia/Perth} |
| %logger、%lo、%c{length}     | 输出当前日志名称。没有length则输出全名。如：com.xyz.LogbackTest |
| %class、%C{length}           | 输出日志调用所在类，没有length则打印全类名。如：com.xyz.LogbackTest（与%c一样） |
| %method、%M                  | 输出日志所在方法（没有length参数）如：main                   |
| %caller{length}              | 日志调用位置，length代表日志深度，如：Caller+0	 at com.xyz.LogbackTest.main(LogbackTest.java:9) |
| %thread、%t                  | 输出线程名（没有length参数）                                 |
| %level、%le、%p              | 输出日志级别（没有length参数）                               |
| %message、%msg、%m           | 输出日志记录内容，就是调用logger的方法的时候传入的log字符串  |
| %exception、%ex              | 输出异常信息。如：`logger.error("error...",new Exception("exception"));` |
| %line、%L                    | 输出日志的行数。如：9                                        |
| %file、%F                    | 输出调用logger的java源码文件名，速度不快，避免使用。如：LogbackTest.java |
| %n                           | 换行符                                                       |
| \%                           | 输出%号                                                      |
| **特殊占位符**               |                                                              |
| %X{user}                     | 表示可以获取外部自定义传入的值, 如：`%X{user}`=》`org.slf4j.MDC.put("user", "xx-yy-zz");` |
| **宽度设置**                 | 可以限制上面的ConversationWord的宽度和左右对齐               |
| %20logger                    | 最小宽度20，当小于20时，则左侧留空白。右对齐                 |
| %-20logger                   | 最小宽度20，当小于20时，则右侧留空白。左对齐                 |
| %.30logger                   | 最大宽度30，超出时从头部开始截断。如：%.2、test=》st         |
| %.-30logger                  | 最大宽度30，超出时从末尾开始截断。如：%.-2、test=》te        |
| %20.30logger                 | 最小20，最大30，小于20的时候右对齐，大于30的时候开始处截断   |
| %-20.30logger                | 最小20，最大30，小于20的时候左对齐，大于30的时候开始处截断   |
| **显示设置**                 |                                                              |
| %highlight()                 | 突出显示，如：%highlight(%-5level)                           |
| %green(%red、%blue、%white)  | 字体显示为指定颜色                                           |
| {length}                     | 可指定长度，如：%logger{36}                                  |
| **网络访问设置**             | 需要依赖logger-access模块                                    |
| %remoteIP、%a                | 远程IP                                                       |
| %localIP、%A                 | 本地IP                                                       |
| %clientHost、%h              | 远程主机名                                                   |
| %localPort                   | 本地端口                                                     |
| %requestMethod、%m           | http请求方法                                                 |
| %protocol、%H                | http请求协议                                                 |
| %statusCode、%s              | http请求status code                                          |
| %requestURL、%r              | http请求地址                                                 |
| %requestURI、%U              | http请求资源地址                                             |
| %queryString、%q             | http请求参数                                                 |
| %server、%v                  | 服务器地址                                                   |
| %elapsedTime、%D             | http请求处理的时间，单位是毫秒                               |
| %elapsedSeconds、%T          | http请求处理的时间，单位是秒                                 |
| %date、%t                    | 日志记录时间                                                 |
| %threadName、%I              | 处理请求的线程名                                             |
| %reqAttribute{attributeName} | http请求attribute值                                          |
| %reqCookie{cookie}           | http请求cookie值                                             |
| %reqContent                  | http请求体内容                                               |
| %fullRequest                 | http完整请求                                                 |
| %responseContent             | http响应                                                     |
| %fullResponse                | http完整响应                                                 |

**格式化修饰器**：可以限制上面的ConversationWord的宽度和左右对齐。比如：

- 左对齐修饰符（-）：接着是可选的最小宽度修饰符，用十进制数表示。如果字符小于最小宽度，则左填充或右填充，默认是左填充（即右对齐），填充符为空格。如果字符大于最小宽度，字符永远不会被截断
- 最大宽度修饰符（.）：后面加十进制数。如果字符大于最大宽度，则从前面截断。后面加减号“-”在加数字，表示从尾部截断

1、用来指定日志的输出格式及编码等其他配置（encoder元素）

```xml
<appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
    <encoder><!--日志文件输出格式, 全称形式 及 简写形式-->
        <!--2022-04-05 11:17:26,325 com.xyz.LogbackTest com.xyz.LogbackTest main INFO main INFO com.xyz.LogbackTest : test pattern...-->
        <pattern>%d %lo %C %M %le %t %le %lo : %m%n</pattern>
        <!--2022-04-05 11:18:00,832 com.xyz.LogbackTest com.xyz.LogbackTest main INFO main : test pattern...-->
        <pattern>%date %logger %class %method %highlight(%level) %thread : %msg%n</pattern>
        <!--2022-04-05 11:18:42.394 default [main] INFO  com.xyz.LogbackTest - test pattern...-->
        <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
        <!-- 设置字符编码 -->
        <charset class="java.nio.charset.Charset">UTF-8</charset>
        <!-- 将 immediateFlush 设置为 false 可以获得更高的日志吞吐量 -->
        <immediateFlush>false</immediateFlush>
    </encoder>
</appender>
```

2、使用特殊占位符`%X{}`示例：

```xml
<configuration debug="false">
    <!--日志输出器:输出到控制台-->
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d [%X{uuid}] [%X{x}] [%t] [%-5level] %c %M %L %m %n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="consoleAppender"/>
    </root>
</configuration>
```

```java
package com.xyz;
import ch.qos.logback.classic.Logger;
import org.slf4j.LoggerFactory;
import java.util.UUID;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = (Logger)LoggerFactory.getLogger(LogbackTest.class);
        org.slf4j.MDC.put("uuid", UUID.randomUUID().toString());
        logger.info("info {}", "test");
    }
}
```

```
2022-04-06 18:46:33,795 [4effd2ef-a4de-4a6c-82c1-c1d522e9aa76] [] [main] [INFO ] com.xyz.LogbackTest main 10 info test 
```





### 08、filters 过滤器元素

官网地址：https://logback.qos.ch/manual/filters.html

Filter是日志过滤器，是appender里面的子元素。执行一个过滤器会有返回DENY、NEUTRAL、ACCEPT三个枚举值中的一个。appender 有多个过滤器时，按照配置顺序执行。

- DENY：日志将立即被抛弃不再经过其他过滤器
- NEUTRAL：有序列表里的下个过滤器过接着处理日志
- ACCEPT：日志会被立即处理，不再经过剩余过滤器



#### 1、LevelFilter

LevelFilter 基于级别来过滤日志事件。如果事件的级别与配置的级别相等，过滤器会根据配置的 onMatch 与 onMismatch 属性，接受或者拒绝事件

```xml
<configuration>
    <!--日志输出器:输出到控制台,只打印INFO级别的日志-->
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志级别(针对具体指定级别)过滤器-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 只显示INFO级别日志-->
            <level>INFO</level>
            <!-- INFO级别的日志, 被立即执行不再经过其他过滤器-->
            <onMatch>ACCEPT</onMatch>
            <!-- 不是INFO级别的日志, 立即被抛弃不再经过其他过滤器-->
            <onMismatch>DENY</onMismatch>
        </filter>
        <encoder>
            <pattern>%date %-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="consoleAppender" />
    </root>
</configuration>
```

#### 2、ThresholdFilter

ThresholdFilter 基于给定的临界值来过滤事件。如果事件的级别等于或高于给定的临界值，当调用 decide() 时，ThresholdFilter 将会返回 NEUTRAL。但是事件的级别低于临界值将会被拒绝。简单来说：针对指定级别及以上级别进行日志过滤

```xml
<configuration>
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志级别(针对指定级别及以上级别)过滤器-->
        <!-- 会打印INFO及比INFO等级高的日志-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <encoder>
            <pattern>%date %-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="consoleAppender" />
    </root>
</configuration>
```



#### 3、EvaluatorFilter

EvaluatorFilter： 求值过滤器，评估、鉴别日志是否符合指定条件。有一下子节点：

- `<evaluator>`：鉴别器，常用的鉴别器是JaninoEventEvaluato，也是默认的鉴别器，它以任意的java布尔值表达式作为求值条件，求值条件在配置文件解释过成功被动态编译，布尔值表达式返回true就表示符合过滤条件。evaluator有个子标签`<expression>`，用于配置求值条件
- `<onMatch>`：用于配置符合过滤条件的操作
- `<onMismatch>`：用于配置不符合过滤条件的操作

求值表达式作用于当前日志，logback向求值表达式暴露日志的各种字段（evaluator标签中可以用的对象和值）：

| Name             | Type                | Description                                                  |
| :--------------- | :------------------ | :----------------------------------------------------------- |
| event            | LoggingEvent        | 与记录请求相关联的原始记录事件，下面所有变量都来自event，例如，event.getMessage()返回下面"message"相同的字符串 |
| message          | String              | 日志的原始消息，例如，设有logger mylogger，"name"的值是"AUB"，对于 mylogger.info("Hello {}",name); "Hello {}"就是原始消息。 |
| formattedMessage | String              | 日志被各式话的消息，例如，设有logger mylogger，"name"的值是"AUB"，对于 mylogger.info("Hello {}",name); "Hello Aub"就是格式化后的消息。 |
| logger           | String              | logger 名。                                                  |
| loggerContext    | LoggerContextVO     | 日志所属的logger上下文。                                     |
| level            | int                 | 级别对应的整数值，所以 level > INFO 是正确的表达式。         |
| timeStamp        | long                | 创建日志的时间戳。                                           |
| marker           | Marker              | 与日志请求相关联的Marker对象，注意“Marker”有可能为null，所以你要确保它不能是null。 |
| mdc              | Map                 | 包含创建日志期间的MDC所有值得map。访问方法是：mdc.get("myKey") 。mdc.get()返回的是Object不是String，要想调用String的方法就要强转，例如，((String) mdc.get("k")).contains("val") .MDC可能为null，调用时注意。 |
| throwable        | java.lang.Throwable | 如果没有异常与日志关联"throwable" 变量为 null. 不幸的是， "throwable" 不能被序列化。在远程系统上永远为null，对于与位置无关的表达式请使用下面的变量`throwableProxy` |
| throwableProxy   | IThrowableProxy     | 与日志事件关联的异常代理。如果没有异常与日志事件关联，则变量"throwableProxy" 为 null. 当异常被关联到日志事件时，"throwableProxy" 在远程系统上不会为null |

过滤器还可以来自自带的或表达式直接写在配置文件中。

```xml
<!-- Janino 是一个极小、极快的 开源Java 编译器-->
<dependency>
    <groupId>org.codehaus.janino</groupId>
    <artifactId>janino</artifactId>
    <version>3.1.6</version>
</dependency>
```

```xml
<configuration>
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 根据判断条件的过虑器-->
        <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
            <evaluator>
                <!--expression为java代码,需要添加groupId为org.codehaus.janino,artifactId为janino的maven包-->
                <expression><!-- 默认为 ch.qos.logback.classic.boolex.JaninoEventEvaluator -->
                    // 根据日志消息判断
                    if(event.getMessage().contains("url")){
                    return true;
                    }
                    if(formattedMessage.contains("url")){
                    return true;
                    }
                    return false;
                </expression>
            </evaluator>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <encoder>
            <pattern>%date %-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="ALL">
        <appender-ref ref="consoleAppender"/>
    </root>
</configuration>
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.Timer;
import java.util.TimerTask;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogbackTest.class);
        // 1秒后启动,每秒跑一次
        new Timer().scheduleAtFixedRate(new TimerTask(){
            @Override
            public void run() {
                logger.error("error test");
                logger.warn("warn test");
                logger.info("info {}", "url");
                logger.debug("debug test ");
                logger.trace("trace url");
            }
        }, 1000, 1000);
    }
}
```

```
2022-04-03 22:46:31,251 1421 [Timer-0] INFO  LogbackTest - info url
2022-04-03 22:46:31,251 1421 [Timer-0] TRACE LogbackTest - trace url
2022-04-03 22:46:32,254 2424 [Timer-0] INFO  LogbackTest - info url
2022-04-03 22:46:32,254 2424 [Timer-0] TRACE LogbackTest - trace url
```

***

EvaluatorFilter 中还有一个特殊的标签，基本很少用，了解即可。

- `<matcher>` ：匹配器，尽管可以使用String类的matches()方法进行模式匹配，但会导致每次调用过滤器时都会创建一个新的Pattern对象，为了消除这种开销，可以预定义一个或多个matcher对象，定以后就可以在求值表达式中重复引用。`<matcher>`是`<evaluator>`的子标签。
- `<matcher>`中包含两个子标签，一个`<name>`，用于定义matcher的名字，求值表达式中使用这个名字来引用matcher；另一个`<regex>`，用于配置匹配条件

```xml
<configuration debug="true">   
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
        <filter class="ch.qos.logback.core.filter.EvaluatorFilter">   
            <evaluator>           
                <matcher>   
                    <Name>odd</Name>   
                    <!-- filter out odd numbered statements -->   
                    <regex>statement [13579]</regex>   
                </matcher>   

                <expression>odd.matches(formattedMessage)</expression>   
            </evaluator>   
            <OnMismatch>NEUTRAL</OnMismatch>   
            <OnMatch>DENY</OnMatch>   
        </filter>   
        <encoder>   
            <pattern>%-4relative [%thread] %-5level %logger - %msg%n</pattern>   
        </encoder>   
    </appender>   

    <root level="DEBUG">   
        <appender-ref ref="STDOUT" />   
    </root>   
</configuration>  
```



#### 4、自定义过滤器

通过实现ch.qos.logback.core.filter.Filter接口可以自定义过滤器，自定义过滤器类LogbackUrlFilter.class，示例如下：

```java
package com.xyz;
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.filter.Filter;
import ch.qos.logback.core.spi.FilterReply;

public class LogbackUrlFilter extends Filter<ILoggingEvent> {
    @Override
    public FilterReply decide(ILoggingEvent event) {
        if(event.getMessage().contains("url")){
            return FilterReply.ACCEPT;
        }
        if(event.getFormattedMessage().contains("url")){
            return FilterReply.ACCEPT;
        }
        return FilterReply.DENY;
    }
}
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.Timer;
import java.util.TimerTask;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogbackTest.class);
        // 1秒后启动,每秒跑一次
        new Timer().scheduleAtFixedRate(new TimerTask(){
            @Override
            public void run() {
                logger.error("error test");
                logger.warn("warn test");
                logger.info("info {}", "url");
                logger.debug("debug test ");
                logger.trace("trace url");
            }
        }, 1000, 1000);
    }
}
```

```
2022-04-03 22:46:31,251 1421 [Timer-0] INFO  LogbackTest - info url
2022-04-03 22:46:31,251 1421 [Timer-0] TRACE LogbackTest - trace url
2022-04-03 22:46:32,254 2424 [Timer-0] INFO  LogbackTest - info url
2022-04-03 22:46:32,254 2424 [Timer-0] TRACE LogbackTest - trace url
```

通常情况下，日志输出会配置三个，一个控制台输出用于开发阶段；一个INFO及以上级别的日志输出，可追踪相应的生产日志；一个单独ERROR级别的日志输出，方便快速检查出异常日志。都可以通过Appender和Filter来控制。



### 09、logger 元素

logger用来设置某一个类或者某个包的日志输出级别、以及关联的appender，这样就可以控制非业务日志不写到对应的文件日志中。

logger标签中包含三个属性：

- name：要输出日志的包名或者类名，比如com.xyz。必选项

- level：设置日志的级别（不区分大小写：TRACE，DEBUG，INFO，WARN，ERROR，ALL或OFF）如果未设置，则logger会向上继承最近一个非空级别。可选项
- additivity：是否将日志向上级传递，默认为 true。可选项

logger通过1个或多个子节点appender-ref来控制日志的输出：

```xml
<!--表示org.springframework包下的日志以warn级别输出-->
<logger name="org.springframework" level="WARN"/>

<!--表示com包下的日志以debug级别输出-->
<logger name="com" level="debug" />

<!--对指定包指定appender进行日志控制，由于设置了info级别，additivity为true，而且关联consoleAppender，因此info以上级别的日志会输出到控制台-->
<!--同时会把日志上传到父级，即root。若root也有配置consoleAppender的输出的话，会在控制台输出两次。additivity为false，则不会。-->
<!-- 只打印包com.xyz下的日志，最低级别从info开始-->
<logger name="com.xyz" level="info" additivity="true">
    <appender-ref ref="consoleAppender" />
</logger>
```



### 10、root 元素

root元素配置根记录器。它是个特殊的logger，是所有logger的根节点，只能指定日志级别及level（默认DEBUG）和appender-ref（输出的appender）

```xml
<!--这是第一种打印到多个位置的配置(此时打印到控制台和文件的日志级别都是info级别)-->
<root level="info">
    <!--引入appender，日志记录器，使用name属性来获取指定的appender对象-->
    <appender-ref ref="consoleAppender"/>
    <appender-ref ref="fileAppender"/>
</root>
```

```xml
<!--这是第二种打印到多个位置的配置(此时虽然代码中写的控制台和文件的打印级别不同，但是此时打印出来的都是info级别，以最后一个级别为准)-->
<root level="ALL">
    <appender-ref ref="fileAppender"/>
</root>
<!--最后一个root会覆盖前面所有的root,只会生效一个root配置-->
<root level="info">
    <appender-ref ref="consoleAppender"/>
</root>
```

level属性的值可以是不区分大小写的字符串TRACE，DEBUG，INFO，WARN，ERROR，ALL或OFF之一。root元素可以包含零个或多个appender-ref元素；被引用的每个appender都被添加到根记录器中。

> **root元素和logger元素的区别？**
>
> root是根logger，是一种logger，root和根logger其实是一回事；只不过root中不能有 name 和 additivity 这两个属性



### 11、include 包含元素

文件包含元素：将配置文件的一部分包含在另一个文件中，可以通过标签include来引入另一个配置文件。如下所示：

```xml
<configuration>
    <include file="src/main/java/resources/includedConfig.xml"/>
    <root level="DEBUG">
        <appender-ref ref="includedConsole" />
    </root>
</configuration>
```

includedConfig.xml文件定义了被引用的内容：

```xml
<included>
    <appender name="includedConsole" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>"%d - %m%n"</pattern>
        </encoder>
    </appender>
</included>
```

include标签引入的文件可以为一个文件，一个类路径上的资源，或者一个URL。如下：

```xml
<include file="src/main/java/resources/includedConfig.xml"/>
<include resource="includedConfig.xml"/>
<include url="http://xxx.com/includedConfig.xml"/>
```

如果被引用的文件不存在，logback会打印内部的状态信息。如果包含的文件是可选的，可以通过optional属性设置为true来进制打印显示警告信息。

```xml
<include optional="true"/>
```



### 12、springProfile 元素

如果是基于SpringBoot项目，针对不同环境（profile）有不同的日志输出，比如开发（dev）环境只输出到控制台，生产环境（prod）只输入INFO和ERROR，那则可用到Spring支持的profile机制。对应的元素为springProfile。profile的值与springboot中配置文件的spring.profiles.active值进行对照。

```xml
<configuration>
    <springProfile name="dev">
        <!-- configuration to be enabled when the "dev" profile is active -->
        <logger name="com.example.demo.controller" level="DEBUG" additivity="false">
            <appender-ref ref="consoleLog"/>
        </logger>
    </springProfile>

    <springProfile name="dev | test">
        <!-- configuration to be enabled when the "dev" or "test" profiles are active -->
        <logger name="com.example.demo.controller" level="INFO" additivity="false">
            <appender-ref ref="consoleLog"/>
        </logger>
    </springProfile>

    <springProfile name="!production">
        <!-- configuration to be enabled when the "production" profile is not active -->
        <logger name="com.example.demo.controller" level="INFO" additivity="false">
            <appender-ref ref="consoleLog"/>
        </logger>
    </springProfile>
</configuration>
```



## 4、Logback其他操作

### 1、动态切换日志级别

Logback提供有代码方式直接修改日志级别，我们可以获取LoggerContext对象修改日志级别及其他配置。如下是Java代码实现（无配置文件）

```java
package com.xyz;

import ch.qos.logback.classic.Level;
import ch.qos.logback.classic.Logger;
import ch.qos.logback.classic.LoggerContext;
import org.slf4j.LoggerFactory;

public class LogbackTest {
    public static void main(String[] args) {
        Logger logger = (Logger)LoggerFactory.getLogger(LogbackTest.class);

        System.out.println("============使用默认的日志输出级别DEBUG==============");
        printLogs(logger);

        System.out.println("============动态设置日志输出级别为ERROR==============");
        // 返回正在使用的 ILoggerFactory 实例.
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        // 根据包路径或类路径获取日志记录器，"ROOT" 表示根记录器: Logger getLogger(final String name);
        loggerContext.getLogger(Logger.ROOT_LOGGER_NAME).setLevel(Level.toLevel("error"));
        printLogs(logger);

        System.out.println("============动态设置日志输出级别为WARN==============");
        loggerContext.getLogger("com.xyz").setLevel(Level.valueOf("warn"));
        printLogs(logger);
    }

    private static void printLogs(Logger logger) {
        logger.error("test error...");
        logger.warn("test warn...");
        logger.info("test info...");
        logger.debug("test debug...");
        logger.trace("test trace...");
    }
}
```

```
============使用默认的日志输出级别DEBUG==============
18:16:51.570 [main] ERROR com.xyz.LogbackTest - test error...
18:16:51.570 [main] WARN com.xyz.LogbackTest - test warn...
18:16:51.570 [main] INFO com.xyz.LogbackTest - test info...
18:16:51.570 [main] DEBUG com.xyz.LogbackTest - test debug...
============动态设置日志输出级别为ERROR==============
18:16:51.570 [main] ERROR com.xyz.LogbackTest - test error...
============动态设置日志输出级别为WARN==============
18:16:51.570 [main] ERROR com.xyz.LogbackTest - test error...
18:16:51.570 [main] WARN com.xyz.LogbackTest - test warn...
```



### 2、停止logback-classic

为了释放 logback-classic 资源，停止 logback context 是一个好主意。如果停止，会关闭所有 loggers 关联的 appenders，并有序的停止所有活动线程。

```java
import org.sflf4j.LoggerFactory;
import ch.qos.logback.classic.LoggerContext;

// 返回正在使用的 ILoggerFactory 实例.
LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
loggerContext.stop(); // stop logback-classic
```



### 3、模块logback-access

logback-access模块与Servlet容器（如Tomcat和Jetty）集成，以提供HTTP访问日志功能。我们可以使用logback-access模块来替换tomcat的访问日志

1. 将logback-access.jar与logback-core.jar复制到$TOMCAT_HOME/lib/目录下（可以从本地的maven库直接copy）

   ```xml
   <!--logback的基础模块,实际上也可以不导入,因为logback-access中也包含了基础模块包-->
   <dependency>
       <groupId>ch.qos.logback</groupId>
       <artifactId>logback-core</artifactId>
       <version>1.2.3</version>
   </dependency>
   <!--实际可以只导入logback-access包即可-->
   <dependency>
       <groupId>ch.qos.logback</groupId>
       <artifactId>logback-access</artifactId>
       <version>1.2.3</version>
   </dependency>
   ```

2. 修改$TOMCAT_HOME/conf/server.xml中的Host元素中添加：

   ```xml
   <Valve className="ch.qos.logback.access.tomcat.LogbackValve" />
   ```

3. logback默认会在$TOMCAT_HOME/conf下查找文件：logback-access.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?> 
   <configuration> 
       <!-- always a good activate OnConsoleStatusListener --> 
       <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener"/> 
       <property name="LOG_DIR" value="${catalina.base}/logs"/> 
       <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
           <file>${LOG_DIR}/access.log</file> 
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
               <fileNamePattern>access.%d{yyyy-MM-dd}.log.zip</fileNamePattern> 
           </rollingPolicy>
           <encoder> 
               <!-- 访问日志的格式,官网有固定的枚举值 对应的表达式类型,两种写法都可以--> 
               <!--<pattern>%h %l %u [%t] "%r" %s %b "%i{Referer}" "%i{User-Agent}"</pattern>-->
               <pattern>combined</pattern> 
           </encoder> </appender> 
       <appender-ref ref="FILE"/> 
   </configuration>
   ```

4. 结果验证：启动tomcat，刷新页面，查看日志：logs/access.log

   ```
   127.0.0.1 - - [06/四月/2022:10:43:40 +0800] "GET / HTTP/1.1" 200 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36 Edg/99.0.1150.55"
   127.0.0.1 - - [06/四月/2022:10:43:40 +0800] "GET / HTTP/1.1" 200 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36 Edg/99.0.1150.55"
   ```

5. 官方配置：https://logback.qos.ch/access.html#configuration



## 5、MDC分布式应用追踪请求

> 此方式可以用在任何日志实现框架中使用，如：Logback、log4j2 等

上面所讲的日志都是在单个应用系统下记录日志的。一旦进入分布式系统，很可能就会出现日志错乱，对日志追踪和排查造成难题。如果使用像ELK这类框架将日志进行归集统一处理，也需要一个标识，来记录日志的整个请求处理过程。

Slf4j提供了MDC（Mapped Diagnostic Contexts诊断上下文映射），可以让开发人员在诊断上下文中放置信息。通过ThreadLocal实现了线程与线程之间的数据隔离。在输出时可以通过标识符%X{key}来输出MDC中设置的内容。【此方式可以用在任何日志实现框架中使用，如：Logback、log4j2 ...】

分布式应用追踪请求实现思路如下：

```
Web拦截器增加唯一ID »» 增加ID到MDC中 »» 调用其他服务时ID作为Header参数 »» 输出日志添加ID »» 请求结束,清除ID »» 根据相同ID排查日志
```

下面来看一下具体的实现代码。这里使用的是SpringBoot + Filter，也可以使用HandlerInterceptor拦截器（效果一样）

```java
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.util.UUID;
import org.slf4j.MDC;

public class CorrelationIdLoggingFilter implements Filter {
    private static final String CORRELATION_ID = "correlation_id";
    private static final String UU_ID = "uu_id";

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;

        String correlationId = req.getHeader(CORRELATION_ID);
        MDC.put(CORRELATION_ID, correlationId);
        String uuId = UUID.randomUUID().toString();
        MDC.put(UU_ID, uuId);

        chain.doFilter(request, response);
        MDC.remove(CORRELATION_ID);
        MDC.remove(UU_ID);
    }
}
```

```java
@Configuration
public class FilterRegistration {
    @Bean
    public FilterRegistrationBean correlationIdLoggingFilterRegistrationBean() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new CorrelationIdLoggingFilter()); // 可以new也可以Bean注入
        registration.setName("correlationIdLoggingFilter"); // 设置过滤器名称
        registration.addUrlPatterns("/*"); // 拦截路径
        registration.setOrder(-1); // 设置执行顺序,数字越低优先级越高
        return registration;
    }
}
```

```xml
<!--如下是Logback.xml配置文件配置-->
<property name="log_pattern" value="%d{MM-dd HH:mm:ss.SSS} [%X{uu_id}] [%X{correlational_id}] [%5p] [%40.40c{1.}:%3L] - %m%n"/>
<!--如下是SpringBoot全局配置文件配置-->
logging.pattern.console=%d{MM-dd HH:mm:ss.SSS} [%X{uu_id}] [%X{correlation_id}] [%5p] [%40.40c{1.}:%3L] - %m%n
```

启动后测试可以看出：启动时候没有correlation_id与uu_id值，当有请求进来才会生成：

```bash
curl -H 'Content-Type:application/json' -H 'correlation_id:xxxxx-yyyyy-zzzzz'  http://localhost:8080/logback/logs
```

```
04-15 12:16:29.740 [] [] [ INFO] [             o.s.b.w.e.t.TomcatWebServer:220] - Tomcat started on port(s): 8080 (http) with context path ''
04-15 12:16:29.750 [] [] [ INFO] [                      c.x.XyzApplication: 61] - Started XyzApplication in 1.504 seconds (JVM running for 2.672)
04-15 12:16:51.659 [] [] [ INFO] [                       o.a.c.c.C.[.[.[/]:173] - Initializing Spring DispatcherServlet 'dispatcherServlet'
04-15 12:16:51.659 [] [] [ INFO] [               o.s.w.s.DispatcherServlet:525] - Initializing Servlet 'dispatcherServlet'
04-15 12:16:51.660 [] [] [ INFO] [               o.s.w.s.DispatcherServlet:547] - Completed initialization in 1 ms
04-15 12:16:51.678 [e91e1660-69d2-4fd8-bdcb-304c0cd89c70] [xxxxx-yyyyy-zzzzz] [ERROR] [                    c.x.Log4j2Controller: 24] - hello, I am error!
04-15 12:16:51.678 [e91e1660-69d2-4fd8-bdcb-304c0cd89c70] [xxxxx-yyyyy-zzzzz] [ WARN] [                    c.x.Log4j2Controller: 25] - hello, I am warn!
04-15 12:16:51.680 [e91e1660-69d2-4fd8-bdcb-304c0cd89c70] [xxxxx-yyyyy-zzzzz] [ INFO] [                    c.x.Log4j2Controller: 26] - hello, I am info!
```



## 6、SpringBoot集成Logback

> SpringBoot官网文档：https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/features.html#features.logging

### 1、SpringBoot日志依赖关系

1. spring-boot-starter是SpringBoot启动器，每一个SpringBoot应用都会依赖到它

2. sping-boot-starter依赖Sping-boot-starter-looging

3. SpringBoot 底层默认使用 SLF4J+ Logback 方式进行日志记录

4. SpringBoot 使用中间替换包把其的日志框架都替换成了SLF4J

5. SpringBoot 能自动适配所有的日志框架，且底层使用SLF4J + Logback方式记录日志，引入其他框架时，只需要把这个框架依赖的日志框架排除掉即可

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
       <!-- 去掉springboot默认配置 -->
       <exclusions>
           <exclusion>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-logging</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   <!-- 引入log4j2依赖 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-log4j2</artifactId>
   </dependency>



### 2、SpringBoot全局日志设置

SpringBoot官网配置文件详解：https://docs.spring.io/spring-boot/docs/2.5.0/reference/htmlsingle/#application-properties.core

1. 日志的级别由低到高分别为：trace(跟踪) < debug(调试) < info(信息) < warn(警告) < error(错误)

2. 在配置文件中调整输出的日志级别，日志就只会在这个级别及以后的高级别生效，SpringBoot 默认使用 INFO 级别（Logback 默认是 DEBUG）

3. SpringBoot 的全局配置文件 "application.properties" 或 "application.yml" 中可以修改日志配置项：

   ```properties
   # 默认名logback-spring.xml,如果要设置其他名称则需要如下配置
   # logging.config=classpath:logback-spring.xml
   # 设置根节点的日志级别输出,root表示整个项目
   logging.level.root=INFO
   # 指定特定包及类的日志输出级别,未指定就按root设置的级别输出,如果root也未指定则按SpringBoot的默认级别info输出
   logging.level.org.springframework.web=DEBUG
   logging.level.org.hibernate=ERROR
   logging.level.com.xyz=DEBUG
   
   # 配置日志输出的文件,这两个选一个配置就可以了,一起配置的话,name的生效.每次启动都是追加日志
   # .name=具体文件, 写入指定的日志文件.文件名可以是确切的位置,也可以是相对于当前目录的位置(相对路径为与pom.xml同级)
   # .path=具体目录, 写入spring.log文件到指定目录.目录名可以是确切的位置,也可以是相对于位置(文件名spring.log无法更改)
   logging.file.name=logs/spring-boot.log
   logging.file.path=logs/
   
   # 指定控制台输出的日志格式,例如: %d{yyyy-MM-dd HH:mm:ss} [%X{uu_id}] -- [%thread] %-5level %logger{50} %msg%n
   # 1、%d 表示日期时间，
   # 2、%thread 表示线程名，
   # 3、%‐5level 级别从左显示5个字符宽度
   # 4、%logger{50} 表示logger名字最长50个字符，否则按照句点分割。
   # 5、%msg 日志消息，
   # 6、%n 换行符
   # 7、%line 显示日志输出位置的行号，方便寻找位置
   # 8、%X 特殊占位符，%X{uu_id}是获取uu_id的值，代码中设置uu_id值:org.slf4j.MDC.put("uuid", "xx-yy-zz");
   # 配置日志输出格式, .file是配置输出到文件的日志格式, .console是配置输出到控制台的日志格式
   logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{50}:%line %msg%n
   logging.pattern.file=%d{yyyy-MM-dd HH:mm} -- [%thread] %-5level %logger{50} %msg%n
   
   # 设置日志记录器组,将相关的记录器组合在一起,然后设置日志记录器组的日志级别为TRACE
   logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
   logging.level.tomcat=TRACE
   # SpringBoot包括以下预定义的日志记录组，可以开箱即用
   # web: org.springframework.core.codec, org.springframework.http, org.springframework.web
   # sql: org.springframework.jdbc.core, org.hibernate.SQL
   logging.level.web=INFO
   logging.level.sql=DEBUG
   ```
   
4. SpringBoot全局配置中的日志配置优先级高于第三方日志框架配置文件（application.properties > logback-spring.xml）



### 3、SpringBoot官网文档翻译

#### 1、SpringBoot日志生成路径

| logging.file.name | logging.file.path | 示例         | 说明                                                         |
| :---------------- | :---------------- | :----------- | :----------------------------------------------------------- |
| (none)            | (none)            |              | 只在控制台输出                                               |
| 指定文件名        | (none)            | demo.log     | 输出到当前项目根路径下的 demo.log 文件中                     |
| (none)            | 指定目录          | logs/log_lzy | 输出到当前项目所在磁盘根路径下的/logs/log_lzy目录中的 spring.log 文件中 |
| 指定文件名        | 指定目录          |              | 当两个同时指定时, name会生效。推荐使用logging.file.name设置，因为它可自定义文件名 |

```properties
# 配置日志输出的文件,这两个选一个配置就可以了,一起配置的话,name的生效. 每次启动都是追加日志
# .name=具体文件, 写入指定的日志文件.文件名可以是确切的位置,也可以是相对于当前目录的位置(相对路径为与pom.xml同级)
# .path=具体目录, 写入spring.log文件到指定目录.目录名可以是确切的位置,也可以是相对于位置(文件名spring.log无法更改)
logging.file.name=logs/spring-boot.log
logging.file.path=logs/
```

> 1. 默认情况：日志文件超过10M时，会新建文件进行递增，如logback.log、logback1.log、logback2.log，使用logging.file.max-size更改大小限制
> 2. 默认情况：日志只记录到控制台，不写入日志文件，如果要在控制台输出之外写入到日志文件，则需要设置 .file 或 .path 属性
> 2. 默认情况，logging.file.* 等配置使用的都是RollingFileAppender

日志输出到文件是非常有用的，比如命令行启动一个 jar 包，但是它一执行就自动报错，然后退出了，还来不急看清错误信息，此时可以修改配置把日志输出到指定文件中，再次启动错误信息就会自动存放进去



#### 2、SpringBoot日志级别设置

SpringBoot Log Levels 日志级别官方文档：https://docs.spring.io/spring-boot/docs/2.5.0/reference/htmlsingle/#features.logging.log-levels

1. 通过配置`logging.level.<logger name>=<level>`可以设置所有受支持的日志系统的日志级别，其中 level 是 ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF 之一

2. 可以使用 logging.level.root 配置根日志记录器：

   ```properties
   # 未特别指定的仍然按 Spring Boot 的默认的 logging.level.root 设置输出.
   logging.level.root=WARN  # 根节点日志级别，即整个应用的日志级别设置，默认为 info
   logging.level.org.springframework.web=DEBUG   # 单独某个包的日志级别设置，spring web 包日志输出级别
   logging.level.org.hibernate=ERROR    # 单独某个包的日志级别设置，hibernate 日志输出级别
   logging.level.com.xzy=info  # 自己项目中指定包下的日志输出级别
   ```



#### 3、SpringBoot日志记录器组

SpringBoot Log Groups 日志记录器组官方文档：https://docs.spring.io/spring-boot/docs/2.5.0/reference/htmlsingle/#features.logging.log-groups

1. 将相关的记录器组合在一起，以便可以同时对它们进行配置，这通常是很有用的。Spring Boot 允许在 Spring 环境中定义日志组。例如下面通过将 “tomcat” 组添加到 application.properties 中来定义它，定义后，可以用一行更改组中所有记录器的级别

   ```properties
   logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
   logging.level.tomcat=TRACE
   ```

2. SpringBoot 包括以下预定义的日志记录组，可以开箱即用：

   | Name | Loggers                                                      |
   | ---- | ------------------------------------------------------------ |
   | web  | org.springframework.core.codec, org.springframework.http, org.springframework.web |
   | sql  | org.springframework.jdbc.core, org.hibernate.SQL             |

   ```properties
   logging.level.web=INFO
   logging.level.sql=DEBUG
   ```



#### 4、SpringBoot设置打印SQL

使用Mybatis的时候，SQL语句是DEBUG下才会打印,而这里我们只配置了INFO，所以想要查看SQL语句的话,有以下两种操作：

1、第一种把`<root level="info">`改成`<root level="DEBUG">`这样就会打印SQL，不过这样日志那边会出现很多其他消息

2、第二种就是单独给dao下目录配置debug模式，代码如下，这样配置sql语句会打印，其他还是正常INFO级别：

- application.properties 全局配置

  ```properties
  logging.level.sql=DEBUG
  logging.level.org.mybatis=debug
  logging.level.com.xyz.*.dao=debug
  ```

- logback-spring.xml 自定义配置文件配置

```xml
<configuration debug="true" scan="true" scanPeriod="60 seconds">
    <!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
    <logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />
    <logger name="org.hibernate.SQL" level="DEBUG" />
    <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
    <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

    <!--myibatis log configure-->
    <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>
</configuration>
```



#### 5、JAR启动设置日志输出级别

可以在启动的时候通过命令行参数指定日志输出级别（java -jar xxx.jar --level），相当于 logging.level.root 配置的根日志记录器。

```bash
java -jar .java-se-1.0-SNAPSHOT.jar --debug
```



### 4、Logback日志配置文件详解

1. 可以直接在 SpringBoot 的全局配置文件中修改 SLF4J 的默认配置，也可以使用 SLF4J 实现框架的自己的配置文件。直接放置再类路径下即可

2. 官网参考链接，根据日志记录系统的不同，各自的配置文件文件也不同：

   | Logging System（日志系统） | Customization（日志文件名称）                                |
   | -------------------------- | ------------------------------------------------------------ |
   | Logback                    | logback-spring.xml，logback-spring.groovy，logback.xml or logback.groovy |
   | Log4j2                     | log4j2-spring.xml，log4j2.xml                                |
   | JDK (Java Util Logging)    | logging.properties                                           |

3. 因为 SpringBoot 底层默认采用 SLF4J+Logback 的日志组合，也就是说如果在 src/main/resources 目录下放置其中任一类型的配置文件，SpringBoot便会自动进行使用。而SpringBoot官方推荐优先使用带有-spring的文件名配置（如有logback-spring.xml，则不会使用logback.xml）。若需要对配置文件名进行修改，或者希望把放到其它目录下，可以在application中通过logging.config属性来指定，如：

   ```properties
   logging.config=classpath:dev/logback-spring.xml
   ```

4. 具体配置文件设置详情可参考上面的教程：【配置文件结构详解】

5. 这是转载其他人整理的百分号属性参数说明大全：

   ```
       ********************************************************************************************************************
       参数        说明                                         举例                    输出显示媒介
       ********************************************************************************************************************
       %c          列出logger名字空间的全称，如果加上{<层数>}，   假设当前logger的命名空间是"a.b.c"
                   则表示列出从最内层算起的指定层数的名字空间
                                                               %c                  a.b.c
                                                               %c{2}               b.c
                                                               %20c                （若名字空间长度小于20，则左边用空格填充）
                                                               %-20c               （若名字空间长度小于20，则右边用空格填充）
                                                               %.30c               （若名字空间长度超过30，截去多余字符）
                                                               %20.30c             （若名字空间长度小于20，则左边用空格填充；
                                                                                       若名字空间长度超过30，截去多余字符）
                                                               %-20.30c            （若名字空间长度小于20，则右边用空格填充；
                                                                                       若名字空间长度超过30，截去多余字符）
       ********************************************************************************************************************
       %C          列出调用logger的类的全名（包含包路径）         假设当前类是"org.apache.xyz.SomeClass"
                                                               %C                  org.apache.xyz.SomeClass
                                                               %C{1}               SomeClass
       %class
       ********************************************************************************************************************
       %d          显示日志记录时间，{<日期格式>}使用ISO8601定义的日期格式
                                                               %d{yyyy/MM/dd HH:mm:ss,SSS}     2005/10/12 22:23:30,117
                                                               %d{ABSOLUTE}        22:23:30,117
                                                               %d{DATE}            12 Oct 2005 22:23:30,117
                                                               %d{ISO8601}         2005-10-12 22:23:30,117
       ********************************************************************************************************************
       %F          显示调用logger的源文件名                     %F                   MyClass.java
       ********************************************************************************************************************
       %l          显示日志事件的发生位置，包含包路径、方法名、
                   源文件名，以及在代码中的行数                  %l                   com.a.b.MyClass.main(MyClass.java:168)
       ********************************************************************************************************************
       %L          显示调用logger的代码行                       %L                   129
       %line                                                  %line                129
       ********************************************************************************************************************
       %level      显示该条日志的优先级                         %level               INFO
       %p                                                     %p                   INFO
       ********************************************************************************************************************
       %m          显示输出消息                                 %m                  This is a message for debug.
       %message                                                %message            This is a message for debug.
       ********************************************************************************************************************
       %M          显示调用logger的方法名                       %M                   main
       ********************************************************************************************************************
       %n          当前平台下的换行符                           %n                   Windows平台下表示rn，UNIX平台下表示n
       ********************************************************************************************************************
       %p          显示该条日志的优先级                         %p                   INFO
       %level                                                 %level               INFO
       ********************************************************************************************************************
       %r          显示从程序启动时到记录该条日志时已经经过的毫秒数  %r                 1215
       ********************************************************************************************************************
       %t          输出产生该日志事件的线程名                    %t                   http-nio-8080-exec-10
       %thread                                                 %thread              http-nio-8080-exec-10
       ********************************************************************************************************************
       %x          按NDC（Nested Diagnostic Context，线程堆栈）顺序输出日志           假设某程序调用顺序是MyApp调用com.foo.Bar
                                                               %c %x - %m%n        MyApp - Call com.foo.Bar.
                                                                                   com.foo.Bar - Log in Bar
                                                                                   MyApp - Return to MyApp.
       ********************************************************************************************************************
       %X          按MDC（Mapped Diagnostic Context，线程映射表）
                   输出日志。通常用于多个客户端连接同一台服务器，
                   方便服务器区分是那个客户端访问留下来的日志。     %X{5}             （记录代号为5的客户端的日志）
       ********************************************************************************************************************
       %%          显示一个百分号                               %%                  %
       ********************************************************************************************************************
   ```



### 5、动态切换日志级别的几种方式

#### 1、logback.xml 配置文件定时监控

修改 logback.xml 配置文件，定时监控配置变化情况。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- debug : 开启logback运行日志输出，true为始终输出,false为出错时才输出-->
<!-- scan ：开启"热更新"-->
<!-- scanPeriod："热更新"扫描周期，默认 60 seconds(60秒) 单位有:milliseconds、seconds、munites、hours，默认为milliseconds-->
<configuration scan="true" scanPeriod="60 seconds" debug="false"/>
```

1. 如果是在本地 IDE 编辑器中测试，注意修改的是 classes 编译目录下的 logback.xml 文件，而不是 resources 目录下的源文件
2. 鉴于现在都是微服务开发，分布式部署，而且为了方便管理，基本都是打包后部署在一些平台上面。直接修改日志配置文件不太现实



#### 2、LoggerContext 切换日志输出级别

使用Controller接口动态修改日志输出级别，想设置什么级别，只需传参调接口即可，无论是设置root级别，还是指定包或类，不用重启服务能立马生效

```java
package com.example.xyz;

import ch.qos.logback.classic.Level;
import ch.qos.logback.classic.Logger;
import ch.qos.logback.classic.LoggerContext;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LogbackController {
    private static final Logger log = (Logger) LoggerFactory.getLogger(LogbackController.class);
    /**
     * 打印日志测试: http://localhost:8080/logback/logs
     */
    @GetMapping("logback/logs")
    public void logs() {
        log.error("hello, I am error!");
        log.warn("hello, I am warn!");
        log.info("hello, I am {}!", "info");
        log.debug("hello, I am debug!");
        log.trace("hello, I am trace!");
    }

    /**
     * logback 动态切换指定包或者类的日志输出级别，立即生效.
     *   http:localhost:8080/logback/updateLevel
     *   http:localhost:8080/logback/updateLevel?level=DEBUG
     *   http:localhost:8080/logback/updateLevel?level=DEBUG&clazz=com.xyz
     *
     * @param level ：日志级别，可选值有：ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF，默认为 debug。不区分大小写.
     * @param clazz ：指定的包或者类路径，为空时默认设置全局(root)日志级别。路径不存在时，照样会设置这个路径，不会影响全局级别，这一点与 log4j2 不同.
     *              比如 org.springframework。
     * @return map  ：返回指定路径的当前日志级别，root 表示全局级别
     */
    @GetMapping(value = "/logback/updateLevel")
    public String setLevel(String level, String clazz) {
        try {
            // 返回正在使用的 ILoggerFactory 实例.
            LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
            /**
             * Logger getLogger(final String name)：根据包路径或类路径获取日志记录器,"ROOT" 表示根记录器
             * Level toLevel(String sArg)：转换为日志级别，如果转换失败，则默认为 debug
             * synchronized void setLevel(Level newLevel)：为日志记录器设置日志输出级别，立即生效.
             */
            if (clazz == null || "".equals(clazz.trim())) {
                // 设置根日志记录器对象并设置日志输出级别
                loggerContext.getLogger(Logger.ROOT_LOGGER_NAME).setLevel(Level.toLevel(level));
            } else {
                // 设置指定日志记录器并设置日志输出级别,路径不存在也没有关系,会返回它的日志记录器,然后设置输出级别
                loggerContext.getLogger(clazz).setLevel(Level.toLevel(level));
            }
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            return "fail";
        }
        return "success";
    }
}
```



#### 3、SpringBoot Actuator 监控管理日志

使用SpringBoot Actuator监控管理日志的优点：

- 无需编码，只需要引用Spring Boot Actuator监控依赖，然后开启访问端点 /loggers，即可轻松查看日志输出级别，并进行切换
- 解耦日志框架，无论使用的 Logback 还是 Log4j2 ，都能轻松切换

1、引入pring Boot Actuator依赖

```xml
<!--监控和管理应用程序-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2、开启日志访问端点： /loggers

```yaml
management:
  endpoint:
    health:
      show-details: ALWAYS # 展示节点的详细信息
  endpoints:
    web:
      exposure:
        include: info,health,logfile,loggers # 指定公开的访问端点
```

3、查看级别

- 发送 GET 请求，获取日志等级：http://localhost:8080/actuator/loggers 
- 返回的信息非常详细，包含了 ROOT，以及程序中各个包和类的日志级别
- 其中 configuredLevel 表示配置级别，effectiveLevel 表示有效级别，configuredLevel  可能为 null，因为没有配置。

4、修改日志级别

- 发送 POST 请求，设置指定包或者类（com.xyz）日志输出级别：http://localhost:8080/actuator/loggers/com.xyz
- 发送 POST 请求，设置 root 全局日志输出级别：http://localhost:8080/actuator/loggers/root
- 请求 Body 的内容格式：{"configuredLevel":"error"}



## 7、参考文献 & 鸣谢

- Spring Boot 2.x 日志配置 与集成 Logback 日志框架【CSDN 蚩尤后裔】https://blog.csdn.net/wangmx1993328/article/details/81044147
- 万字详解logback日志框架，再没这么全的了【CSDN 程序新视界】https://blog.csdn.net/wo541075754/article/details/109193354
- 学会这些Logback高级知识点，程序日志性能提高几十倍(上)【CSDN 陈皮的JavaLib】https://javalib.blog.csdn.net/article/details/113899664
- java日志(三)–slf4j和logback使用【CSDN：panda-star】https://blog.csdn.net/chinabestchina/article/details/104743907



# Log4j2日志框架

## 1、简介及入门示例

### 1、背景介绍

官网地址：https://logging.apache.org/log4j/2.x/、Maven 仓库地址：https://search.maven.org/artifact/org.apache.logging.log4j/log4j-api

Log4j2是对Log4j1x的升级版，同时修复与升级了Logback的不足，被誉为是目前最优秀的Java日志框架。Apache Log4j2日志框架的主要特征：

- 自动重新加载配置：参考了Logback的设计，提供自动刷新参数配置，最实用的就是我们在生产上可以动态的修改日志的级别而无需重启应用
- 异常处理：在logback中，Appender中的异常不会被应用感知到，但是在Log4j2中，提供了一些异常处理机制
- 性能提升：log4j2相较于log4j和logback都具有很明显的性能提升，据官方测试，异步记录器的吞吐量比Log4j 1.x 和 Logback高18倍，延迟低
- 无垃圾机制：（默认开启）log4j2在大部分情况下，都可以使用无垃圾机制【对象重用、内存缓冲】，避免频繁的日志收集导致的 jvm gc
- 插件架构：Log4j2使用插件模式配置组件。因为无需编写代码来创建和配置Appender、Layout、Pattern Converyer等。在配置了的情况下，Log4j2自动识别插件并使用他们
- 高级过滤：与Logback一样，Log4j2支持基于Log事件中的上下文数据，标记，正则表达式和其他组件进行过滤。此外，过滤器还可以与记录器关联。与Logback不同，Log4j2可以在任何这些情况下使用通用的Filter类

Log4j2 除了提供日志实现以外，也拥有一套自己的独立的门面。不过目前市面上最主流的日志门面是SLF4J，虽然Log4j2也是日志门面，因为它的日志实现功能非常强大，性能优越。所以大家一般还是将Log4j2看作是日志的实现，SLF4J + Log4j2 的组合是市场上最强大的日志功能实现方式，绝对是主流日志框架。



### 2、模块介绍

Log4j2中分为：门面（log4j-api.jar）和实现 （log4j-core.jar） 两个模块。门面与SLF4J是一个类型，属于日志抽象门面。而实现部分才是Log4j2的核心：

- 日志门面：org.apache.logging.log4j » log4j-api
- 日志实现：org.apache.logging.log4j » log4j-core

Log4j2 重要类的概念：

- org.apache.logging.log4j.LogManager：日志类工厂，日志管理器，提供静态方法getContext，getFactory，getLogger，可以方便的获取相关信息
- org.apache.logging.log4j.Logger：日志记录类，用来输出被记录的日志
- org.apache.logging.log4j.core.LoggerContext：日志上下文，包含已配置的Configuration、Appender，Logger，Filter等信息
- org.apache.logging.log4j.core.config.Configuration：每一个LoggerContext都有一个有效的Configuration。Configuration包含了所有的Appenders、上下文范围内的过滤器、LoggerConfigs。在重配置期间，新与旧的Configuration将同时存在。当所有的Logger对象都被重定向到新的Configuration对象后，旧的Configuration对象将被停用和丢弃



### 3、入门案例

1、使用Log4j做日志门面

```xml
<!-- log4j2日志门面 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.2</version>
</dependency>
<!-- log4j2日志实面,log4j-api在log4j-core中已经有依赖了,单独依赖core即可 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.2</version>
</dependency>
```

```java
package com.xyz;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class Log4j2Test {
    public static void main(String[] args) {
        // 定义日志记录器对象,如下没有使用SLF4J门面技术,完全使用的log4j中的类和方法
        Logger logger = LogManager.getLogger(Log4j2Test.class);
        // log4j2中存在6中日志输出级别
        logger.fatal("fatal");
        logger.error("error");
        logger.warn("warn");
        logger.info("info");
        logger.debug("debug");
        logger.trace("trace");
    }
}
```

```
12:10:00.727 [main] FATAL com.xyz.Log4j2Test - fatal
12:10:00.727 [main] ERROR com.xyz.Log4j2Test - error
```

> 此时可以发现只输出了error及以上级别的日志，因为Log4j2门面默认是error级别，就说明此时这个日志是使用Log4j2框架实现的。
>

2、使用SLF4J作为日志的门面

```xml
<!-- 导入slf4j日志门面依赖 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.36</version>
</dependency>
<!-- log4j适配器 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.17.2</version>
</dependency>
<!-- log4j2日志门面 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.2</version>
</dependency>
<!-- log4j2日志实面,log4j-api在log4j-core中已经有依赖了,单独依赖core即可 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.2</version>
</dependency>
```

```java
package com.xyz;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4j2Test {
    public static void main(String[] args) {
        // 注意：此时使用的slf4j的门面技术,没有适应到log4j2,输出的是 slf4j的日志级别（有5个级别）
        Logger logger = LoggerFactory.getLogger(Log4j2Test.class);
        // slf4j中存在5种日志输出级别，此时使用是slf4j的记录器,而不是log4j2的,所以只能输出slf4j中的五种级别
        logger.error("error信息");
        logger.warn("warn信息");
        logger.info("info信息");
        logger.debug("debug信息");
        logger.trace("trace信息");
    }
}
```

```
23:11:22.977 [main] ERROR com.xyz.Log4j2Test - error信息
```

> 执行结果看到不管是Log4j2自己的门面还是SLF4J门面搭配Log4j2实现默认日志级别都是是error。



## 2、最牛的功能及性能

### 1、最强的异步和同步性能

Log4j2在目前JAVA中的日志框架里，异步日志的性能是最高的，没有之一。先来看一下，几种日志框架benchmark对比结果（官方测试结果）

![Peak throughput comparison](https://ask.qcloudimg.com/http-save/yehe-4464177/k286bbw776.png)

从图上可以看出，Log4j2的异步（全异步，非混合模式）下的性能，远超Log4j1和Logback，简直吊打。压力越大的情况下，吞吐上的差距就越大。在64线程测试下，Log4j2的吞吐达到了180w+/s，Llogback/Log4j1只有不到20w，相差近十倍。

其实与各个日志框架对比而言，无论在同步或者异步情况下，log4j2表现更加优异，而其他日志就显得差强人意了。



### 2、零GC（Garbage-free）

从Log4j 2.6版本开始（2016年），Log4j2默认就以零GC模式运行了。什么叫零GC呢？就是不会由于Log4j2而导致GC。Log4j2中各种Message对象，字符串数组，字节数组等全部复用，不重复创建，大大减少了无用对象的创建，从而做到“零GC”。



### 3、更高性能 I/O 写入的支持

Log4j2还提供了一个MemoryMappedFileAppender，I/O 部分使用MemoryMappedFile来实现，可以得到极高的I/O性能。不过在使用MemoryMappedFileAppender之前，得确定你足够了解MemoryMappedFile的相关知识，否则不要轻易使用呦。



### 4、更强大的参数格式化

Log4j的API与SLF4J相比，提供了更丰富的参数格式化功能。使用 {} 占位符格式化参数。在SLF4J里，我们可以用 {} 的方式来实现“format”的功能（参数会直接toString替换占位符），像下面这样：

```java
org.apache.logging.log4j.Logger logger = LogManager.getLogger("com.xyz");
logger.debug("Logging in user {} with birthday {}", user.getName(), user.getBirthdayCalendar());
```



### 5、使用String.format形式格式化参数

Log4j2 中除了支持 {} 的参数占位符，还支持 String.format 的形式

```java
org.apache.logging.log4j.Logger logger = LogManager.getFormatterLogger("com.xyz");
logger.debug("Logging in user %s with birthday %s", user.getName(), user.getBirthdayCalendar());
logger.debug("Logging in user %1$s with birthday %2$tm %2$te,%2$tY", user.getName(), user.getBirthdayCalendar());
logger.debug("Integer.MAX_VALUE = %,d、Long.MAX_VALUE = %,d", Integer.MAX_VALUE, Long.MAX_VALUE);
```

> 注意：如果想使用 String.format 的形式，需要使用 LogManager.getFormatterLogger 而不是 LogManager.getLogger
>



### 6、使用logger.printf格式化参数

Log4j2的Logger接口中，还有一个logger.printf()方法，无需创建 LogManager.getFormatterLogger，就可以使用 String.format 的形式

```java
org.apache.logging.log4j.Logger logger = LogManager.getLogger("com.xyz");
logger.printf(Level.INFO, "Logging in user %1$s with birthday %2$tm %2$te,%2$tY", user.getName(), user.getBirthdayCalendar());
logger.debug("Opening connection to {}...", someDataSource);
```



### 7、“惰性”打日志（lazy logging）

这个功能虽然小，但非常实用。在某些业务流程里，为了留根或追溯问题，需要完整的打印入参，一般是把入参给用JSON序列化后用debug级别打印：

```java
org.apache.logging.log4j.Logger logger = LogManager.getLogger("com.xyz");
logger.debug("入参报文：{}", JSON.toJSONString(policyDTO));
```

如果需要追溯问题时，会将系统的日志级别调到debug，这样就可以打印。但是这里有个问题，虽然在info级别下debug不会输出内容，但JSON.toJSONString()这个序列化的代码一定会执行，严重影响正常流程下的执行效率。

我们期望的结果是info级别下，连序列化都不执行。这里可以通过logger.isDebugEnable()来判断当前配置下debug级别是否可以输出：

```java
org.apache.logging.log4j.Logger logger = LogManager.getLogger("com.xyz");
if(logger.isDebugEnabled()) {
    logger.debug("入参报文：{}", JSON.toJSONString(policyDTO));
}
```

这样虽然可以避免不必要的序列化，但每个地方都这么写还是有点难受的，一行变成了三行。

Log4j2的Logger对象，提供了一系列Lambda的支持，通过这些接口可以实现“惰性”打日志：

```java
void fatal(String message, Supplier... paramSuppliers);
void error(String message, Supplier<?>... paramSuppliers);
void warn(String message, Supplier... paramSuppliers);
void info(String message, Supplier<?>... paramSuppliers);
void debug(String message, Supplier<?>... paramSuppliers);
void trace(String message, Supplier<?>... paramSuppliers);

org.apache.logging.log4j.Logger logger = LogManager.getLogger("com.xyz");
// 等同于下面的先判断,后打印
logger.debug("入参报文：{}",() -> JSON.toJSONString(policyDTO));
// 与上面效果一样
if(logger.isDebugEnabled()) {
    logger.debug("入参报文：{}",JSON.toJSONString(policyDTO));
}
```

这种 Supplier + Lambda 的形式，等同于上面的先判断 logger.isDebugEnable() 然后打印，三行的代码变成了一行。



### 8、更简化的配置文件

Log4j2同时支持XML/JSON/YML/Properties四种形式的配置文件，不过最主流的还是XML的方式，最直观。来查看logback和log4j2的配置文件对比：

logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name = "File" class= "ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/archives/app-%d{yyyy-MM-dd}.log.gz</fileNamePattern>
            <!--一天内大于size就单独分隔-->
            <maxFileSize>1 GB</maxFileSize>
        </rollingPolicy>
    </appender>
    <root level="info">
        <appender-ref ref="File"/>
    </root>
</configuration>
```

log4j2.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" name="XInclude" xmlns:xi="http://www.w3.org/2001/XInclude">
    <Appenders>
        <RollingFile name="File" fileName="logs/app.log" filePattern="logs/archives/app-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %5p [%t] %-40.40c{1.} : %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy />
                <!--一天内大于size就单独分隔-->
                <SizeBasedTriggeringPolicy size="1 GB"/>
            </Policies>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="INFO">
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```

在log4j2中，appender的配置从使用 Appender 实现名即标签名的形式，语法上更简洁一些：

```xml
<RollingFile name="File"></RollingFile>
<!-- 等同于logback中的 -->
<appender name = "File" class= "ch.qos.logback.core.rolling.RollingFileAppender"></appender>
```



## 3、组件介绍与配置

### 1、几大组件的介绍

Log4j2由如下几部分构成：

1. Logger：负责捕获日志记录，并传递给 Appender，他是日志行为的发起者
2. Appender：负责将日志事件进行分类处理，将日志发往他应该去的目标去向，因此也可以称为 Handler
3. Layout：Layout 负责在日志输出前决定日志的格式，因此也可以称为 Fomatter
4. Filter：是可选的组件，每一个 Logger、Appender 甚至全局都可以配置若干个 Filter，来决定相应的组件对当前的日志时间是否关心
5. Level：日志级别有：OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL（默认为：ERROR）源码：org.apache.logging.log4j.Level



### 2、Log4j2默认配置

DefaultConfiguration类中提供的默认配置将设置，通过debug可以在LoggerContext类中发现

```java
package org.apache.logging.log4j.core;

public class LoggerContext extends AbstractLifeCycle 
    implements org.apache.logging.log4j.spi.LoggerContext, AutoCloseable, Terminable, ConfigurationListener, LoggerContextShutdownEnabled {
    /**
     * The Configuration is volatile to guarantee that initialization of the Configuration has completed before the
     * reference is updated.
     */
    private volatile Configuration configuration = new DefaultConfiguration();
}
```

启动时可以通过debug看到：

- 默认的root日志的layout：`%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n`
- 默认的日志级别：ERROR

实际上也能从默认配置类中看到一些默认的配置：

```java
public abstract class AbstractConfiguration extends AbstractFilterable implements Configuration {
    protected void setToDefault() {
        // LOG4J2-1176 facilitate memory leak investigation
        setName(DefaultConfiguration.DEFAULT_NAME + "@" + Integer.toHexString(hashCode()));
        final Layout<? extends Serializable> layout = PatternLayout.newBuilder()
            .withPattern(DefaultConfiguration.DEFAULT_PATTERN)
            .withConfiguration(this)
            .build();
        final Appender appender = ConsoleAppender.createDefaultAppenderForLayout(layout);
        appender.start();
        addAppender(appender);
        final LoggerConfig rootLoggerConfig = getRootLogger();
        rootLoggerConfig.addAppender(appender, null, null);

        rootLoggerConfig.setLevel(getDefaultLevel());
    }
}
```



### 3、自定义配置文件位置

1、Log4j2默认在classpath下查找配置文件，可以修改配置文件的位置。在非web项目中：

```java
ConfigurationSource source = new ConfigurationSource(new FileInputStream("D:/log4j2.xml"));
Configurator.initialize(null, source);
org.apache.logging.log4j.Logger logger = LogManager.getLogger(Log4j2Test.class);
```

2、如果是在SpringBoot项目，在appliaction.properties 或 application.yml 中配置即可：

```properties
logging.config=classpath:log4j2.xml
```

3、如果是web项目，在web.xml中添加（Listener使用的是Log4j2默认的）：

```xml
<listener>
    <listener-class>org.apache.logging.log4j.web.Log4jServletContextListener</listener-class>
</listener>
<context-param>
    <param-name>log4jConfiguration</param-name>
    <param-value>/WEB-INF/conf/log4j2.xml</param-value>
</context-param>
```

4、如果是web项目，在web.xml中添加（使用自定义Listener）：

```xml
<!-- 系统日志配置监听器 -->
<listener>
    <listener-class>com.xyz.Log4j2ConfigListener</listener-class>
</listener>
<context-param>
    <description>Logging Configuration File Path</description>
    <param-name>log4j.configurationFile</param-name>
    <param-value>log4j/log4j2.xml</param-value>
</context-param>
```

```java
package com.xyz;
import java.util.Enumeration;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import org.apache.logging.log4j.core.config.Configurator;
public class Log4j2ConfigListener implements ServletContextListener {
    private static final String KEY = "log4j.configurationFile";
    @Override
    public void contextDestroyed(ServletContextEvent arg0) {
    }
    @Override
    public void contextInitialized(ServletContextEvent arg0) {
        String fileName = getContextParam(arg0);
        Configurator.initialize("Log4j2", "classpath:" + fileName);
    }
    @SuppressWarnings("unchecked")
    private String getContextParam(ServletContextEvent event) {
        Enumeration<String> names = event.getServletContext().getInitParameterNames();
        while (names.hasMoreElements()) {
            String name = names.nextElement();
            String value = event.getServletContext().getInitParameter(name);
            if(name.trim().equals(KEY)) {
                return value;
            }
        }
        return null;
    }
}
```



### 4、XML配置文件模板

模板 1（简约版）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration monitorInterval="1" status="ERROR" strict="true" name="LogConfig">
    <Properties>
        <Property name="logbasedir">E:/logs</Property>
        <Property name="log.layout">%d %-5p %t (%c:%L) - %m%n</Property>
    </Properties>

    <!--此处使用了两种类型的appender,RollingFile为滚动类型,满足策略条件后会新建文件夹记录 -->
    <Appenders>
        <Appender type="Console" name="STDOUT">
            <Target>SYSTEM_OUT</Target>
            <Layout type="PatternLayout" pattern="${log.layout}"/>
        </Appender>
        <Appender type="RollingFile" name="FILE" 
                  fileName="${logbasedir}/jutap-${sys:APPNAME}.log"
                  filePattern = "${logbasedir}/jutap-${sys:APPNAME}-%d{yyyy-MM-dd}.%i.log">
            <Policies>
                <TimeBasedTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <Layout type="PatternLayout">
                <Charset>GBK</Charset>
                <Pattern>${log.layout}</Pattern>
            </Layout>
        </Appender>
        <Appender type="RollingFile" 
                  name="ExceptionLog" 
                  fileName="${logbasedir}/exception-${sys:APPNAME}.log"
                  filePattern="${logbasedir}/exception-${sys:APPNAME}-%d{yyyy-MM-dd}.%i.log">
            <Policies>
                <TimeBasedTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <Layout type="PatternLayout">
                <Charset>GBK</Charset>
                <Pattern>${log.layout}</Pattern>
            </Layout>
        </Appender>
    </Appenders>
    <Loggers>
        <Logger name="exception" level="error" additivity="false">
            <AppenderRef ref="ExceptionLog"/>
        </Logger>
        <Root level="info">
            <AppenderRef ref="STDOUT"/>
            <AppenderRef ref="FILE"/>
        </Root>
        <Logger name="com.garfield.learn" level="debug"/>
        <Logger name="com.garfield.learnp" level="info"/>
    </Loggers>
</Configuration>
```

模板 2（复杂版）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
    status : 这个用于设置log4j2自身内部的信息输出,可以不设置,当设置成trace时,会看到log4j2内部各种详细输出
    monitorInterval : Log4j能够自动检测修改配置文件和重新配置本身, 设置间隔秒数。
    注：本配置文件的目标是将不同级别的日志输出到不同文件，最大2MB一个文件，
    文件数据达到最大值时，旧数据会被压缩并放进指定文件夹
-->
<Configuration status="WARN" monitorInterval="600">
    <Properties>
        <!-- 配置日志文件输出目录，此配置将日志输出到tomcat根目录下的指定文件夹 -->
        <Property name="LOG_HOME">${sys:catalina.home}/WebAppLogs/SSHExample</Property>
    </Properties>
    <Appenders>
        <!--这个输出控制台的配置，这里输出除了warn和error级别的信息到System.out-->
        <Console name="console_out_appender" target="SYSTEM_OUT">
            <!-- 控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch) -->
            <ThresholdFilter level="warn" onMatch="DENY" onMismatch="ACCEPT"/>
            <!-- 输出日志的格式 -->
            <PatternLayout pattern="%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n"/>
        </Console>
        <!--这个输出控制台的配置，这里输出warn和error级别的信息到System.err，在eclipse控制台上看到的是红色文字-->
        <Console name="console_err_appender" target="SYSTEM_ERR">
            <!-- 控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch) -->
            <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 输出日志的格式 -->
            <PatternLayout pattern="%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n"/>
        </Console>
        <!-- TRACE级别日志 -->
        <!-- 设置日志格式并配置日志压缩格式，压缩文件独立放在一个文件夹内，
        日期格式不能为冒号，否则无法生成，因为文件名不允许有冒号，此appender只输出trace级别的数据到trace.log -->
        <RollingRandomAccessFile name="trace_appender"
                                 immediateFlush="true" 
                                 fileName="${LOG_HOME}/trace.log"
                                 filePattern="${LOG_HOME}/trace/trace - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies><!-- 两个配置任选其一 -->
                <!-- 每个日志文件最大2MB -->
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>
            <Filters><!-- 此Filter意思是，只输出debug级别的数据 -->
                <!-- DENY，日志将立即被抛弃不再经过其他过滤器；
                       NEUTRAL，有序列表里的下个过滤器过接着处理日志；
                       ACCEPT，日志会被立即处理，不再经过剩余过滤器。 -->
                <ThresholdFilter level="debug" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingRandomAccessFile>
        <!-- DEBUG级别日志 -->
        <!-- 设置日志格式并配置日志压缩格式，压缩文件独立放在一个文件夹内，
        日期格式不能为冒号，否则无法生成，因为文件名不允许有冒号，此appender只输出debug级别的数据到debug.log -->
        <RollingRandomAccessFile name="debug_appender"
                                 immediateFlush="true" 
                                 fileName="${LOG_HOME}/debug.log"
                                 filePattern="${LOG_HOME}/debug/debug - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies><!-- 两个配置任选其一 -->
                <!-- 每个日志文件最大2MB -->
                <SizeBasedTriggeringPolicy size="2MB"/>
                <!-- 如果启用此配置，则日志会按文件名生成新压缩文件，
                即如果filePattern配置的日期格式为 %d{yyyy-MM-dd HH} ，则每小时生成一个压缩文件，
                如果filePattern配置的日期格式为 %d{yyyy-MM-dd} ，则天生成一个压缩文件 -->
                <!--                 <TimeBasedTriggeringPolicy interval="1" modulate="true" /> -->
            </Policies>
            <Filters><!-- 此Filter意思是，只输出debug级别的数据 -->
                <!-- DENY，日志将立即被抛弃不再经过其他过滤器；
                       NEUTRAL，有序列表里的下个过滤器过接着处理日志；
                       ACCEPT，日志会被立即处理，不再经过剩余过滤器。 -->
                <ThresholdFilter level="info" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingRandomAccessFile>
        <!-- INFO级别日志 -->
        <RollingRandomAccessFile name="info_appender"
                                 immediateFlush="true" 
                                 fileName="${LOG_HOME}/info.log"
                                 filePattern="${LOG_HOME}/info/info - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies>
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>
            <Filters>
                <ThresholdFilter level="warn" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingRandomAccessFile>
        <!-- WARN级别日志 -->
        <RollingRandomAccessFile name="warn_appender"
                                 immediateFlush="true" 
                                 fileName="${LOG_HOME}/warn.log"
                                 filePattern="${LOG_HOME}/warn/warn - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies>
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>
            <Filters>
                <ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingRandomAccessFile>
        <!-- ERROR级别日志 -->
        <RollingRandomAccessFile name="error_appender"
                                 immediateFlush="true" 
                                 fileName="${LOG_HOME}/error.log"
                                 filePattern="${LOG_HOME}/error/error - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies>
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>
            <Filters>
                <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingRandomAccessFile>
    </Appenders>
    <Loggers>
        <!-- 配置日志的根节点 -->
        <root level="trace">
            <appender-ref ref="console_out_appender"/>
            <appender-ref ref="console_err_appender"/>
            <appender-ref ref="trace_appender"/>
            <appender-ref ref="debug_appender"/>
            <appender-ref ref="info_appender"/>
            <appender-ref ref="warn_appender"/>
            <appender-ref ref="error_appender"/>
        </root>
        <!-- 第三方日志系统 -->
        <logger name="org.springframework.core" level="info"/>
        <logger name="org.springframework.beans" level="info"/>
        <logger name="org.springframework.context" level="info"/>
        <logger name="org.springframework.web" level="info"/>
        <logger name="org.jboss.netty" level="warn"/>
        <logger name="org.apache.http" level="warn"/>
    </Loggers>
</Configuration>
```



### 5、动态切换日志级别

```java
package com.xyz;

import org.apache.logging.log4j.Level;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.core.LoggerContext;
import org.apache.logging.log4j.core.config.Configuration;
import org.apache.logging.log4j.core.config.LoggerConfig;

public class Log4j2Test {
    private static  final Logger logger = LogManager.getLogger(Log4j2Test.class);
    public static void main(String[] args) {
        logger.fatal("fatal...");
        logger.error("error...");

        // LoggerContext getContext(final boolean currentContext):获取log4j日志上下文,false表示返回合适调用方的上下文
        LoggerContext loggerContext = (LoggerContext) LogManager.getContext(false);
        // 返回当前配置，发生重新配置时，将替换该配置。
        Configuration configuration = loggerContext.getConfiguration();
        // 查找记录器名称的相应 LoggerConfig
        LoggerConfig loggerConfig = configuration.getLoggerConfig(LogManager.ROOT_LOGGER_NAME);
        // 设置日志级别，如果 level 值不属于 ALL、TRACE、DEBUG、INFO、WARN、ERROR、FATAL、OFF，则默认会设置为 DEBUG
        loggerConfig.setLevel(Level.toLevel("FATAL"));
        // 根据当前配置更新所有记录器
        loggerContext.updateLoggers();
        // 查询 root(根)日志输出级别结果返回
        System.out.println("设置后的Root日志级别: " + LogManager.getRootLogger().getLevel().name());

        logger.fatal("fatal...");
        logger.error("error...");
    }
}
```

```
15:27:25.768 [main] FATAL com.xyz.Log4j2Test - fatal...
15:27:25.768 [main] ERROR com.xyz.Log4j2Test - error...
设置后的Root日志级别: FATAL
15:27:25.778 [main] FATAL com.xyz.Log4j2Test - fatal...
```



## 4、自定义配置文件

Log4j2配置文件通常为：log4j2.xml和log4j2-test.xml，加载顺序为：log4j2-test.xml » log4j2.xml。

XML配置文件语法

```xml
<?xml version="1.0" encoding="UTF-8"?>;
<Configuration>
    <Properties>
        <Property name="name1">value</property>
        <Property name="name2" value="value2"/>
    </Properties>
    <filter  ... />
    <Appenders>
        <appender ... >
            <filter  ... />
        </appender>
        ...
    </Appenders>
    <Loggers>
        <Logger name="name1" level="level">
            <filter  ... />
        </Logger>
        ...
        <Root level="level">
            <AppenderRef ref="name"/>
        </Root>
    </Loggers>
</Configuration>
```

| 节点名称      | 含义                                                         | 配置项                                                       |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Configuration | 配置的根节点，有两个子节点:Appenders和Loggers(表明可以定义多个Appender和Logger) | 该节点有两个配置项status：用于指定log4j2本身的日志打印级别，日志级别从低到高分为TRACE、DEBUG、INFO、WARN、ERROR、FATAL，如果设置为WARN，则低于WARN的信息都不会输出。monitorinterval：用于指定log4j自动重新配置的监测间隔时间，单位是秒(s)，最小是5s |
| Appenders     | 负责将日志输出到目的地（这个目的地可以是控制台，文件，数据库，甚至是邮件），定义输出内容,输出格式,输出方式,日志保存策略等 | log4j2支持的appender种类非常多，完整的appender可通过官方网站进行查询。其中，比较常见的appender有：ConsoleAppender、FileAppender、RandomAccessFileAppender、RollingFileAppender、RollingRandomAccessFileAppender，另外，还提供异步的AsyncAppender，实现异步处理日志事件 |
| Loggers       | LogEvent生产的源头，只有定义了logger并引入的appender，appender才会生效 | 常见的有两种：Root和Logger                                   |



### 1、Configuration

#### 1、Configuration

根节点Configuration中有两个常用的属性：status、monitorterval、name。示例及解释如下：

Configuration元素的属性：

- status：用于指定Log4j2日志的输出级别，会打印Log4j2加载、运行等信息。 有如下值：trace, debug, info, warn, error, fatal
- monitorterval：用于指定自动重新检测读取配置内容的间隔时间，单位为秒（s），最小值为5秒
- name：配置的名称，用的也不太多

```xml
<!--status表示log4j2自身的日志打印级别，如WARN会打印出log4j2加载、运行等信息, monitorinterval监控间隔，单位为秒-->
<Configuration status="WARN" monitorInterval="30" name="xyzConfiguration">
</Configuration>
```



#### 2、Properties

子节点元素Properties是用来定义常量，以便在其他配置项中引用，该配置是可选的，例如定义日志的存放位置

```xml
<Properties>
    <!-- 配置全局自定义变量,使用时通过${name}引用 -->
    <Property name="nowDate">%date{yyyy-MM-dd}</Property>
    <Property name="logDir">${sys:catalina.home}/logs/</Property>
    <Property name="logPath">./logs</Property>
    <property name="pattern_format">[%d{HH:mm:ss:SSS}] [%-5p] - %l -- %m%n</property>
</Properties>
```

| Prefix     | Context                                                      |
| ---------- | ------------------------------------------------------------ |
| base64     | Base64编码数据，格式为${base64:Base64_encoded_data}，如：${base64:SGVsbG8gV29ybGQhCg==} 输出 Hello World! |
| bundle     | 资源绑定。格式为：${bundle:BundleName:BundleKey}，示例如：${bundle:com.domain.Messages:MyKey} |
| ctx        | 线程上下文映射（MDC），如：$${ctx:loginId}                   |
| date       | 插入指定的时间格式。如：$${date:MM-dd-yyyy}                  |
| env        | 系统环境变量，格式为${env:ENV_NAME} 或者 ${env:ENV_NAME:-default_value} ，如：$${env:USER}，$${env:USER:-jdoe} |
| jndi       | 设置JNDI上下文，如：$${jndi:logging/context-name}。注意：在Android下面不能使用 |
| sys        | 系统属性，格式为${sys:some.property}或者 ${sys:some.property:-default_value}，如：${sys:logPath} |
| jvmrunargs | 通过 JMX 访问的 JVM 输入参数，但不是主参数； 请参阅 RuntimeMXBean.getInputArguments()。 在Android上不可用 |
| log4j      | Log4j配置属性。 表达式 ${log4j:configLocation} 和 ${log4j:configParentLocation} 分别提供了log4j配置文件及其父文件夹的绝对路径 |
| main       | 使用 MapLookup.setMainArguments(String[]) 设置的值           |
| map        | 来自 MapMessage 的值                                         |



#### 3、Loggers

Loggers 定义日志输出位置，包含Logger和Root结点。Root用来指定项目的根日志，若没有单独指定Logger，则会默认使用该Root日志输出

**Root**：每个配置都必须有一个根记录器Root。如果未配置，则将使用默认根LoggerConfig，其级别为ERROR且附加了ConsoleAppender。根记录器和其他记录器之间的主要区别是：1.根记录器只有level属性。2.根记录器没有name属性。3.根记录器不支持additivity属性，因为它没有父级

- 属性：level：日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF
- 属性：includeLocation：如果你的layouts或custom过滤器需要location信息，你需要在所有相关日志记录器(包括根日志记录器)的配置中设置“includeLocation=true”，默认为false
- 子节点：AppenderRef：Root的子节点，用来指定该日志输出到哪个Appender

**Logger**：用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。Logger元素必须有一个name属性，每个Logger可以使用TRACE，DEBUG，INFO，WARN，ERROR，ALL或OFF之一配置级别。如果未指定级别，则默认为ERROR。可以为additivity属性分配值true或false。如果省略该属性，则将使用默认值true。子节点AppenderRef用于指定日志输出到哪个Appender，若没有指定，默认集成自Root

- 属性：name：用来指定该Logger所适用的类或包全路径，继承自Root节点。一般是项目或者框架的包名，如：com.xyz、org.springframework
- 属性：level：日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF
- 属性：additivity：表示需不需要打印此logger的父logger，如果是false则只打印当前logger；如果是true则继续打印上一层的logger，直到root
- 属性：includeLocation：如果你的layouts或custom过滤器需要location信息，需要在所有相关日志记录器（包括根）中设置includeLocation为true，默认false
- 子节点：AppenderRef：Logger的子节点，用来指定该日志输出到哪个Appender，如果没有指定，就会默认继承自Root。如果指定了，那么会在指定的这个Appender和Root的Appender中都会输出，此时我们可以设置Logger的additivity="false"只在自定义的Appender中进行输出

**AsyncLogger**：异步日志，Log4j2有三种模式：全异步日志，混合模式，同步日志，性能从高到底，线程越多效率越高，也可以避免日志卡死线程情况发生

- includeLocation：如果你的layouts或custom过滤器需要location信息，需要在所有相关日志记录器（包括根）中设置includeLocation为true，默认false

```xml
<Configuration>
    <!--定义日志输出配置-->
    <Loggers>
        <!---->
        <!--Logger节点控制指定包或类的日志输出(包括等级和目的地),如下是过滤掉spring和mybatis的一些无用的DEBUG信息-->
        <!--参数详解: name为包路径,level为日志级别,additivity表示日志信息是否向上传递,false为不传递(即不重复打印)-->
        <Logger name="org.springframework" level="INFO"></logger>
        <Logger name="org.mybatis" level="INFO"></logger>
        <Logger name="com.xyz"  level="false" additivity="false">
            <AppenderRef ref="consoleAppender" />
        </Logger>

        <!-- Root节点用来指定项目的根日志,如果没有单独指定Logger,那么就会默认使用该Root日志输出 -->
        <Root level="info">
            <!--日志输出到控制台和文件中-->
            <AppenderRef ref="consoleAppender" />
            <AppenderRef ref="fileAppender" />
        </Root>

        <!--AsyncLogger: 异步日志,LOG4J有三种日志模式,全异步日志,混合模式,同步日志,性能从高到底,线程越多效率越高,也可以避免日志卡死线程情况发生-->
        <!--additivity="false" : additivity设置事件是否在rootLogger输出,为了避免重复输出,可以在Logger标签下设置additivity为”false”-->
        <AsyncLogger name="AsyncLogger" level="trace" includeLocation="true" additivity="false">
            <appender-ref ref="RollingFileError"/>
        </AsyncLogger>
    </Loggers>
</Configuration>
```



#### 4、XInclude

可以使用这个标签来引入外部xml文件

```xml
<configuration status="warn" name="XIncludeDemo">
    <xi:include href="log4j-xinclude-appenders.xml" />
    <xi:include href="log4j-xinclude-loggers.xml" />
</configuration>
```



### 2、Appenders

参考官网：http://logging.apache.org/log4j/2.x/manual/appenders.html

Appenders是输出源，用于定义日志输出的地方。Log4j2支持的输出源有很多：ConsoleAppender、FileAppender、RandomAccessFileAppender、RollingFileAppender、RollingRandomAccessFileAppender、AsyncAppender 等，如下是 Appender 所有分类（参考官网）：

| 名称                            | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| AsyncAppender                   | 使用一个单独的线程记录日志，实现异步处理日志时间             |
| CassandraAppender               | 将日志信息输出到一个Apache的Cassandre数据库                  |
| ConsoleAppender                 | 将日志信息输出到控制台                                       |
| FailoverAppender                | 包含其他Appenders，按照顺序尝试，直至成功或结尾              |
| FileAppender                    | 一个OutputStreamAppenders，将日志输出到文件                  |
| FlumeAppender                   | 将日志输出到Apache Flume系统                                 |
| JDBCAppender                    | 将日志通过JDBC输出到关系型数据库                             |
| JMSAppender                     | 将日志输出到JMS（Java Message Service）                      |
| JPAAppender                     | 将日志输出到JPA框架                                          |
| HTTPAppender                    | 通过HTTP输出到日志                                           |
| KafkaAppender                   | 将日志输出到Apache Kafka                                     |
| MemoryMappedFileAppender        | 将日志输出到一块文件关联的内存                               |
| NoSQLAppender                   | 将日志输出到NoSQL数据库，如MongoDB、CouchDB                  |
| OutputStreamAppender            | 将日志输出到一个OutputStream                                 |
| RandomAccessFileAppender        | 性能比FileAppender高20%~200%的文件输出Appender               |
| RewriteAppender                 | 允许对日志信息进行加工掩码输出                               |
| RollingFileAppender             | 按日志文件最大长度限度生成新文件                             |
| RollingRandomAccessFileAppender | 添加了缓存的RollingFileAppender                              |
| RoutingAppender                 | 将日志事件分类，允许通过规则路由日志到不同的输出地（子Appender） |
| SMTPAppender                    | 将日志输出到邮件                                             |
| ScriptAppenderSelectorAppender  | 使用自定义脚本的形式来输出日志                               |
| SocketAppender                  | 将日志输出到Socket                                           |
| SyslogAppender                  | 是一个SocketAppender，将日志输出到远程系统日志               |
| ZeroMQ/JeroMQAppender           | 使用JeroMQ库将日志输出到ZeroMQ终端                           |



#### 1、Appender常用模板

```xml
<!--日志输出到控制台-->
<Console name="consoleAppender" target="SYSTEM_OUT">
    <PatternLayout pattern="%date %logger %processId %threadId %method %class %file %highlight{%level} : %green{%msg} %n" />
</Console>

<!--日志写入文件中-->
<File name="fileAppender" fileName="log4j2Study_${date:yyyy-MM}.log">
    <PatternLayout pattern="%date %logger %level : %msg%n" />
</File>

<!--日志写入文件中，和fileAppender(File)类似，只是RandomAccessFile加入了buffer缓存，且该缓存不可删除 -->
<RandomAccessFile  name="randomFileAppender" fileName="log4j2_randomFileAppender.log">
    <PatternLayout pattern="%date %logger %level : %msg%n" />
</RandomAccessFile >

<!--日志写入文件，根据自定义的滚动策略归档文件，filePattern定义日志文件归档格式-->
<RollingFile name="rollingFileAppender" fileName="log4j2_rollingFileAppender.log"
             filePattern="$${date:yyyy-MM}/%d{MM-dd-yyyy}-%i.log.gz">
    <PatternLayout pattern="%date %logger %level : %msg%n" />
    <!--定义滚动策略-->
    <Policies>
        <!--基于cron表达式触发归档-->
        <CronTriggeringPolicy schedule="0 0 * * * ?"/>
        <!--基于时间触发归档,与上文filePattern配合使用，当filePattern不符合时，就归档-->
        <TimeBasedTriggeringPolicy />
        <!--基于文件大小触发归档-->
        <SizeBasedTriggeringPolicy size="100 MB" />
    </Policies>

    <!--默认日志保留7天-->
    <DefaultRolloverStrategy max="7" />

    <!--log4j 2.5引入，可更精细地控制删除日志策略-->
    <DefaultRolloverStrategy>
        <!--同时满足下述条件，进行删除-->
        <Delete>
            <!--文件名符合.log.gz后缀-->
            <IfFileName glob="*.log.gz" />
            <!--超过60天-->
            <IfLastModified age="60d" />
        </Delete>
    </DefaultRolloverStrategy>
</RollingFile>

<!--和RollingFile类似，只是加入了buffer缓存，且该缓存不可删除-->
<RollingRandomAccessFile></RollingRandomAccessFile>

<!--异步记录日志，配合其它appender使用，不单独使用-->
<Async name="fileAsync" >
    <AppenderRef ref="fileAppender" />
</Async>

<!--日志写入cassandra库中，具体配置在此略过,详情请查看官网-->
<Cassandra></Cassandra>

<!--日志写入jdbc库中，具体配置在此略过,详情请查看官网-->
<JDBC></JDBC>

<!--日志写入消息队列中，具体配置在此略过,详情请查看官网-->
<JMS></JMS>

<!--日志写入kafka消息队列中，具体配置在此略过,详情请查看官网-->
<Kafka></Kafka>

<!--日志写入nosql库中，具体配置在此略过,详情请查看官网-->
<NoSql></NoSql>

<!--日志写入邮件中，具体配置在此略过,详情请查看官网-->
<SMTP></SMTP>

<!--日志写入socket中，具体配置在此略过,详情请查看官网-->
<Socket></Socket>

<!--日志写入http中，具体配置在此略过,详情请查看官网-->
<Http></Http>

<!--日志重写，具体配置在此略过,详情请查看官网-->
<Rewrite></Rewrite>
```



#### 2、ConsoleAppender

ConsoleAppender：使用该Appender可以将日志信息输出到控制台，参数与配置如下

Console 元素的属性：

- name：Appender的名字
- target：输出方法，SYSTEM_OUT 或 SYSTEM_ERR。默认值：SYSTEM_OUT

Console 元素的节点：

- PatternLayout：输出格式，有一个pattern属性，不设置默认为：%m%n
- Filter：设置过滤器，可以为单个Appenders设置，也可以全局设置，后面会详细介绍

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="10" name="myApp">
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <PatternLayout pattern="%m%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="consoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```



#### 3、FileAppender

FileAppender：用于将LogEvent写入到一个文件中，是文件输出源，用于将日志写入到指定的文件，其底层是一个OutputStreamAppender

File 元素的属性

- name：Appender的名字
- fileName：指定写入的log文件的名称
- append：指定是否是追加写入（append=true，默认情况），还是覆盖写入（append=false）
- bufferedIO：是否对数据进行缓冲到缓冲区满后再写入。测试显示，即使在启用immediateFlush的情况下，设置bufferedIO=true也能提高性能。
- bufferSize：当bufferedIO=true时，缓冲区的大小，默认是8192bytes
- locking：是否对文件上锁，当有多个线程可能同时写该文件时需要考虑上锁（在《异常处理反模式》中就提到要把在一起的日志输出语句写到一句，而不是拆成几句来避免并发线程导致的日志语句之间的错位）。但对文件上锁会影响系统的性能，所以需要谨慎使用。默认值是false
- immediateFlush：是否立即将数据刷入磁盘，能够最大程度保证数据的完整性，但是对性能会有一定的影响，默认值为true

File 元素的节点

- PatternLayout：用于指定输出格式，不设置的话，默认为：%m%n
- Filter：设置过滤器，可以为单个Appenders设置，也可以全局设置，后面会详细介绍

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="10" name="myApp">
    <Appenders>
        <File name="fileAppender" fileName="logs/app.log" append="true">
            <PatternLayout>
                <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
            </PatternLayout>
        </File>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="fileAppender"/>
        </Root>
    </Loggers>
</Configuration>
```



#### 4、RandomAccessFileAppender

> RandomAccessFile参考：https://www.cnblogs.com/jyy599/p/12076662.html

与FileAppender很相似，除了将BufferdOutputStream替换为了ByteBuffer + RandomAccessFile，还使用了缓存，它的性能比FileAppender高20%~200%

RandomAccessFile 元素的属性

- 配置基本上与FileAppender相同，见上文，主要区别在于bufferSize
- bufferSize：缓存空间大小，默认为256M

RandomAccessFile 元素的节点

- PatternLayout：用于指定输出格式，不设置的话，默认为：%m%n
- Filter：设置过滤器，可以为单个Appenders设置，也可以全局设置，后面会详细介绍

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="10" name="myApp">
    <Appenders>
        <RandomAccessFile name="randomAccessFile" fileName="logs/app.log">
            <PatternLayout>
                <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
            </PatternLayout>
        </RandomAccessFile>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="randomAccessFile"/>
        </Root>
    </Loggers>
</Configuration>
```



#### 5、RollingFileAppender（最常用）

RollingFile用于实现日志文件动态更新的Appender，当满足条件（日志大小、指定时间等）重命名或打包原日志文件进行归档生成新文件，比File更强大

RollingFileAppender是一个OutputStreamAppender，先写入指定文件，并根据TriggeringPolicy和RolloverPolicy滚动文件。RollingFile需要两个策略的支持，分别是TriggeringPolicy和RolloverPolicy。TriggeringPolicy定义何时应该生成新的日志文件而RolloverPolicy则决定如何生成新的日志文件。RollingFile不支持文件锁

RollingFile 元素的属性：

- name：用于指定Appender的名称
- fileName：用于指定日志文件的全路径
- filePattern：用于指定分割文件的日志全路径（命名规则）

RollingFile 元素的节点：

- PatternLayout：用于指定输出格式，不设置的话，默认为：%m%n`
- Policies ：设置日志文件切割参数
  - OnStartupTriggeringPolicy：Policies的子节点，每次JVM启动，都滚动到新的日志文件开始记录，一般不怎么使用
  - SizeBasedTriggeringPolicy：Policies的子节点，按照日志文件大小进行触发滚动策略，size属性表示分割文件大小，单位 KB、MB、GB 或 TB
  - TimeBasedTriggeringPolicy：Policies的子节点，根据日期时间间隔触发进行滚动，interval属性用于指定滚动时间间隔，默认是1小时，modulate属性是用于对interval进行偏移调节，默认为false。若为true，则第一次触发时是第一个小时触发，后续以interval间隔触发
  - CronTriggeringPolicy：Policies的子节点，用于设置基于Cron表达式触发的滚动策略
  - 等等还有许多，后面会单独介绍，也可以参考官网
- DefaultRolloverStrategy：设置默认策略设置
- Filter：设置过滤器，可以为单个Appenders设置，也可以全局设置，后面会详细介绍

如下配置：有三个Trigger，表示每秒生成一个文件或者每小时生成一个文件或者当文件大小达到250MB时生成一个新的文件；有两个Strategy，表示最多保持文件数量为10，达到最大值后，旧的文件将被删除，或者文件名符合.log.gz后缀 or 超过60天 也会删除旧文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="0" name="myApp">
    <Appenders>
        <RollingFile name="rollingFile"
                     fileName="logs/app.log"
                     filePattern="logs/app-%d{yyyy-MM-dd_HH-mm-ss}-%i.log.zip">
            <PatternLayout pattern="%d %p %c{1.} [%t] %m%n"/>
            <!--定义滚动策略,满足其中一个就会触发滚动策略-->
            <Policies>
                <!--由于filePattern是按秒配置的,TimeBasedTriggeringPolicy触发后会每秒生成文件,所以后面的策略都不会触发-->
                <!--基于时间触发归档,与上文filePattern配合使用,当filePattern不符合时,就归档-->
                <TimeBasedTriggeringPolicy />
                <!--基于cron表达式触发归档-->
                <CronTriggeringPolicy schedule="0 0 * * * ?"/>
                <!--基于文件大小触发归档-->
                <SizeBasedTriggeringPolicy size="250 MB"/>
            </Policies>
            <!--默认日志保留10天-->
            <DefaultRolloverStrategy max="10"/>
            <!--log4j 2.5引入，可更精细地控制删除日志策略-->
            <DefaultRolloverStrategy>
                <!--同时满足下述条件，进行删除,maxDepth表示要访问的目录的最大级别数-->
                <Delete basePath="logs/" maxDepth="2">
                    <!--文件名符合.log.gz后缀-->
                    <IfFileName glob="logs/*.log.zip"/>
                    <!--超过60天-->
                    <IfLastModified age="60d"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="rollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

如下单独使用三个示例进行详细讲解：

（1）基于大小的滚动策略

日志先写入logs/app.log中，每当文件大小达到100MB时或经过1天，按照在logs/2020-09/目录下以app-2020-09-09-1.log.gz格式对该日志进行压缩重命名并归档，并生成新的文件app.log进行日志写入。其中，filePattern属性的文件格式中`%i`就类似于一个整数计数器，受到`<DefaultRolloverStrategy max="10"/>`控制，要特别注意的是：当文件个数达到10个的时候会循环覆盖前面已归档的1-10个文件。若不设置该参数，默认为7。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" name="MyApp" packages="">
    <Appenders>
        <RollingFile name="RollingFile" fileName="logs/app.log"
                     filePattern="logs/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout>
                <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
            </PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="10"/>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="RollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

（2）基于时间间隔的滚动策略

日志先写入logs/app.log中，每当文件的时间间隔到达6小时（由`%d{yyyy-MM-dd-HH}`决定，也可以设置成`%d{yyyy-MM-dd-HH-mm}`，则间隔为分钟级别），触发rollover操作。如下配置设置好后，10点的日志开始重启服务，则从11点触发一次rollover操作，生成`2022-04-10-10.log.gz`对该日志进行压缩重命名并归档，并生成新的文件app.log进行日志写入；然后，每间隔6小时，则下一次是17点触发一次，生成2022-04-10-17.log.gz对该日志进行压缩重命名并归档，并生成新的文件app.log进行日志写入。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" name="MyApp" packages="">
    <Appenders>
        <RollingFile name="RollingFile" fileName="logs/app.log"
                     filePattern="logs/$${date:yyyy-MM}/app-%d{yyyy-MM-dd-HH}.log.gz">
            <PatternLayout>
                <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
            </PatternLayout>
            <Policies>
                <!--interval:时间间隔数,默认1,例如:filePattern配置是小时,如果是2小时间隔,则配置成2即可-->
                <!--modulate:翻转发生在间隔边界上,就是整点,例如:启动时间3:30,间隔时间2小时,开启配置后下次反转时间为6:00-->
                <TimeBasedTriggeringPolicy interval="6" modulate="true"/>
            </Policies>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="RollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

（3）基于时间间隔和文件大小的滚动策略

日志先写入logs/app.log中，每当文件大小达到100MB或者当时间间隔到达6小时（由`%d{yyyy-MM-dd-HH}`决定），触发rollover操作，按照在`logs/2022-04/`目录下以`app-2022-04-1-1.log.gz`格式对该日志进行压缩重命名并归档，并生成新的文件app.log进行日志写入。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" name="MyApp" packages="">
    <Appenders>
        <RollingFile name="RollingFile"
                     fileName="logs/app.log"
                     filePattern="logs/$${date:yyyy-MM}/app-%d{yyyy-MM-dd-HH}-%i.log.gz">
            <PatternLayout>
                <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
            </PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy interval="6" modulate="true"/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="RollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```



#### 6、RollingRandomAccessFileAppender

RollingRandomAccessFile类似于标准的RollingFile，除了它总是被缓冲（且不能被关闭），并且在内部它使用 ByteBuffer + RandomAccessFile 而不是BufferedOutputStream。与RollingFileAppender相比，当 bufferedIO = true 时，性能提升 20-200％。

RollingFile 元素的属性：

- name：指定Appender的名字。
- fileName 指定当前日志文件的位置和文件名称
- filePattern 指定当发生Rolling时，文件的转移和重命名规则
- immediateFlush 设置为true时 - 默认值，每次写入后都会进行刷新。这将保证数据写入磁盘，但可能会影响性能。
- bufferSize 缓冲区大小，默认为256M 或 262,144字节（256 * 1024）

RollingFile 元素的节点：

- PatternLayout：用于指定输出格式，不设置的话，默认为：%m%n`
- Policies：指定滚动日志的策略，就是什么时候进行新建日志文件输出日志
  - SizeBasedTriggeringPolicy：指定当文件大小大于size指定的值时，触发Rolling
  - TimeBasedTriggeringPolicy：这个配置需要和filePattern结合使用，日期格式精确到哪一位，interval也精确到哪一个单位。注意filePattern中配置的文件重命名规则是${FILE_NAME}-%d{yyyy-MM-dd HH-mm-ss}-%i，最小的时间粒度是ss，即秒钟。TimeBasedTriggeringPolicy默认的size是1，结合起来就是每1秒钟生成一个新文件。如果改成%d{yyyy-MM-dd HH}，最小粒度为小时，则每一个小时生成一个文件
- DefaultRolloverStrategy：指定最多保存的文件个数
- Filter：设置过滤器，可以为单个Appenders设置，也可以全局设置，后面会详细介绍

```xml
<!--和RollingFile类似，只是加入了buffer缓存，且该缓存不可删除,案例可以直接参考RollingFile,替换标签即可-->
<RollingRandomAccessFile></RollingRandomAccessFile>
```



#### 7、AsyncAppender（异步）

异步输出日志，默认使用 java.util.concurrent.ArrayBlockingQueue 来实现的，不需要其他的外部库，这个配置在多线程的情况下需要小心，可能会出现锁竞争，如果在多线程下，可以使用Disruptor库（一个无锁的线程间通信库，而不是队列，从而产生更高的吞吐量和更低的延迟），可以使用asyncRoot或者asyncLogger来进行配置。

```xml
<Async name="Async">
    <AppenderRef ref="RollingFileAppender" />
    <AppenderRef ref="Console" />
    <LinkedTransferQueue/>
</Async>
```



### 3、Filters

官网地址：https://logging.apache.org/log4j/2.x/manual/filters.html

Filter用于过滤日志（是可选的）Log4j2会在日志产生时自动调用预先配置的Filter的filter方法进行过滤，以便获得是否允许打印的标识。

是否允许打印的标识是一个 Result 类型的枚举，他的值有三种：ACCEPT、DENY、NEUTRAL

- ACCEPT：（直接接受）日志会被立即处理，不再经过剩余过滤器
- DENY：（直接拒绝）日志将立即被抛弃不再经过其他过滤器
- NEUTRAL：（中立，不做处理，交由后面代码处理）有序列表里的下个过滤器过接着处理日志

重点讲解 NEUTRAL，如果只有一个 Filter，那么 NEUTRAL 与 ACCEPT 没有任何区别，只有在多个 Filter 级联使用时，NEUTRAL 才有意义，他表示由下一个 filter 决定是否 ACCEPT。

通常 filter 并不直接决定最终的结果，因为不同的场景下，filter 命中后的行为并不一定相同，因此 filter 只返回命中或未命中，然后由业务具体需要决定是否允许打印相应的日志是更好的选择。Log4j2 的 Filter 就是基于上述原则创建的，他提供了 onMatch（命中）与 onMisMatch（未命中）两个参数供用户配置，filter 值返回当前场景命中或未命中

Log4j2 允许你将 Filter 配置为全局有效或对某个 Appender 生效。



#### 1、Filters常用模板

常用的过滤器如下几类，解析请查看注释

```xml
<!--每秒平均允许的日志事件数为16，最大等待处理日志数为100，onMatch和onMismatch的值可以为ACCEPT、DENY、NEUTRAL-->
<BurstFilter level="info" rate="16" maxBurst="100"  onMatch="NEUTRAL" onMismatch="DENY"  />

<!--基于特定值org.apache.logging.log4j.ThreadContext值，对特定key设置特定的日志级别,这里key为ThreadContext中的key名-->
<DynamicThresholdFilter key="tcKey" defaultThreshold="info" onMatch="ACCEPT" onMismatch="DENY" >
    <!--这里key为ThreadContext中的value值-->
    <KeyValuePair key="tcVal1" value="info" />
    <KeyValuePair key="tcVal2" value="error" />
</DynamicThresholdFilter>

<!--针对info(Message msg)类似方法过滤, 使用如：MapMessage mm = new MapMessage()-->
<MapFilter onMatch="ACCEPT" onMismatch="DENY" >
    <KeyValuePair key="mmKey1" value="mmVal1" />
    <KeyValuePair key="mmKey2" value="mmVal2" />
</MapFilter>

<!--针对info(Marker marker, String message)类似方法过滤，使用如：Marker mk = MarkerManager.getMarker("mk1")-->
<MarkerFilter marker="mk1" onMatch="ACCEPT" onMismatch="DENY" />

<!--正则过滤器，根据正则表达式对合要求的日志记录执行过滤-->
<RegexFilter regex=".*or.*" onMatch="ACCEPT" onMismatch="DENY" />

<!--基于特定值org.apache.logging.log4j.ThreadContext值过滤，operator为or时，表示可以匹配一项，否则为所有项都匹配-->
<ContextMapFilter onMatch="ACCEPT" onMismatch="DENY" operator="or">
    <!--这里key为ThreadContext中的key值,key为ThreadContext中的value值-->
    <KeyValuePair key="tcKey" value="tcVal1" />
</ContextMapFilter>

<!--基于日志级别的过滤器-->
<ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY" />

<!--基于时间的过滤器，时间格式为：HH:mm:ss-->
<TimeFilter start="00:00:00" end="01:00:00" onMatch="ACCEPT" onMismatch="DENY" />

<!--组合过滤器，内可包含多个过滤器-->
<Filters></Filters>
```



#### 2、BurstFilter

BurstFilter 可以控制每秒日志量，对于超过数量的日志进行丢弃。实际就是：控制日志打印速度

- level：筛选消息级别。如果超过maxBurst，则将过滤掉此级别或以下的任何内容。默认WARN，任何高于WARN的日志都被记录，无论Burst大小如何
- rate：参数表示每秒最大日志数
- maxBurst：参数则表示在开始过滤前允许多少条日志请求，默认是rate的10倍
- onMatch：过滤器匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是NEUTRAL
- onMismatch：当过滤器不匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是DENY

```xml
<RollingFile name="RollingFile" fileName="logs/app.log" filePattern="logs/app-%d{MM-dd-yyyy}.log.gz">
    <!--每秒平均允许的日志事件数为16，最大等待处理日志数为100-->
    <BurstFilter level="INFO" rate="16" maxBurst="100"/>
    <PatternLayout pattern="%d %p %c{1.} [%t] %m%n" />
    <TimeBasedTriggeringPolicy />
</RollingFile>
```



#### 3、TimeFilter

TimeFilter 是限制时间的Filter，允许只在一天中的指定时间进行日志记录：

- start：开始时间，格式为：HH🇲🇲ss
- end：结束时间，格式为：HH🇲🇲ss。如果结束时间小于开始时间，那么将没有日志输出
- onMatch：过滤器匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是NEUTRAL
- onMismatch：当过滤器不匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是DENY

```xml
<RollingFile name="RollingFile" fileName="logs/app.log" filePattern="logs/app-%d{MM-dd-yyyy}.log.gz">
    <TimeFilter start="05:00:00" end="05:30:00" onMatch="ACCEPT" onMismatch="DENY"/>
    <PatternLayout pattern="%d %p %c{1.} [%t] %m%n" />
    <TimeBasedTriggeringPolicy />
</RollingFile>
```



#### 4、MarkerFilter

MarkerFilter：过滤不同类型的日志。有的时候，我们希望根据日志中的标记来决定不同的日志输出到不同的位置。MarkerFilter 实现了这一功能：

- marker：要比较的标记名称
- onMatch：过滤器匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是NEUTRAL
- onMismatch：当过滤器不匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是DENY

```xml
<RollingFile name="RollingFile" fileName="logs/app.log" filePattern="logs/app-%d{MM-dd-yyyy}.log.gz">
    <!--针对info(Marker marker, String message)类似方法过滤，使用如：Marker mk = MarkerManager.getMarker("mk1")-->
    <MarkerFilter marker="FLOW" onMatch="ACCEPT" onMismatch="DENY"/>
    <PatternLayout pattern="%d %p %c{1.} [%t] %m%n" />
    <TimeBasedTriggeringPolicy />
</RollingFile>
```

```java
// MarkerFilter中使用
org.apache.logging.log4j.Logger log4j = org.apache.logging.log4j.LogManager.getLogger(Log4j2Test.class);
Marker mk = MarkerManager.getMarker("mk1");
logger.info(mk,"info log by markerFilter");
```





#### 5、MapFilter

MapFilter允许对MapMessage中的数据元素进行过滤

- operator：如果配置为or，那么只要有一个匹配就可以了。否则所有的key/value都要匹配
- onMatch：过滤器匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是NEUTRAL
- onMismatch：当过滤器不匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是DENY
- keyValuePair 子元素：可以定义一个或多个KeyValuePair元素，这些元素定义映射中的键和要匹配的值。如果同一个key被多次指定，那么对该key的检查将自动成为“or”，因为映射只能包含一个值

```xml
<!--针对logger.info(Message msg)类似方法过滤, 使用如：MapMessage mm = new MapMessage(),也可用MDC.set("k","v")设置-->
<MapFilter onMatch="ACCEPT" onMismatch="NEUTRAL" operator="or">
    <KeyValuePair key="eventId" value="Login"/>
    <KeyValuePair key="eventId" value="Logout"/>
</MapFilter>
```

```xml
<Root level="error">
    <MapFilter onMatch="ACCEPT" onMismatch="NEUTRAL" operator="or">
        <KeyValuePair key="eventId" value="Login"/>
        <KeyValuePair key="eventId" value="Logout"/>
    </MapFilter>
    <AppenderRef ref="RollingFile">
    </AppenderRef>
</Root>
```

```xml
<Root level="error">
    <AppenderRef ref="RollingFile">
        <MapFilter onMatch="ACCEPT" onMismatch="NEUTRAL" operator="or">
            <KeyValuePair key="eventId" value="Login"/>
            <KeyValuePair key="eventId" value="Logout"/>
        </MapFilter>
    </AppenderRef>
</Root>
```

```java
// MapFilter中使用
org.apache.logging.log4j.Logger log4j = org.apache.logging.log4j.LogManager.getLogger(Log4j2Test.class);
MapMessage mm = new MapMessage();
mm.put("mmKey1","mmVal1");
mm.put("val","this info msg");
logger.info(mm);
```



#### 6、DynamicThresholdFilter

DynamicThresholdFilter：动态日志级别设置。很多时候，我们需要借助更多的日志来进行问题排查，但过多的日志又势必会对线上服务的性能以及磁盘等资源造成压力，此时有一个好的选择，那就是打印丰富的 debug 级别的日志，而 logger 的 level 至少定义在 info 级别以上，这样实际上在生产环境中，这些 debug 级别的日志并不会被打印出来，而在测试环境中，只需要改变 logger 的 level 就可以打开这些 debug 日志的打印，方便我们排查问题。

有时我们更想要知道线上场景下究竟发生了什么，但现实情况我们又不能让所有人都打印出 debug 级别的日志，有什么办法只让符合条件的请求打印出 debug 级别的日志吗？log4j2 用 DynamicThresholdFilter 解决了这一问题。

- key：去ThreadContext中比较的key
- defaultThreshold：需要被过滤的消息等级。当指定的key不在ThreadContext中时，使用该配置。
- onMatch：过滤器匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是NEUTRAL
- onMismatch：当过滤器不匹配时要采取的操作。可以是ACCEPT、DENY、NEUTRAL，默认是DENY
- keyValuePair 子元素：可以定义多个KeyValuePair属性。通过该属性可以对指定用户设置日志级别。KeyValuePair的key为ThreadContext中获取的value值，KeyValuePair的value为日志的级别，如`<KeyValuePair key="User1" value="DEBUG"/>`

```xml
<!--基于特定值org.apache.logging.log4j.ThreadContext值，对特定key设置特定的日志级别,这里key为ThreadContext中的key名-->
<!---org.apache.logging.log4j.ThreadContext.put("loginId", "user1"); 或 org.slf4j.MDC.put("loginId", "user1")-->
<DynamicThresholdFilter key="loginId" defaultThreshold="ERROR" onMatch="ACCEPT" onMismatch="NEUTRAL">
    <KeyValuePair key="user1" value="DEBUG"/>
    <KeyValuePair key="user2" value="INFO"/>
</DynamicThresholdFilter>
```

```java
// 定义ThreadContext值，可用于在生产环境中，对指定用户或场景进行跟踪
org.apache.logging.log4j.Logger log4j = org.apache.logging.log4j.LogManager.getLogger(Log4j2Test.class);
ThreadContext.put("loginId", "user1");
logger.debug("{} log", "info1");
ThreadContext.put("loginId", "user2");
logger.info("{} log", "info2");
```

这个配置表示：默认日志级别为 ERROR 级别，但符合`ThreadContext.get("loginId")` 或 `MDC.get("loginId")` 为 user1 的请求日志级别为 DEBUG。这样，我们只需要在日志打印前执行`ThreadContext.put("loginId", "user1")`或`MDC.put("loginId", "user1")` 就可以实现动态改变本次请求的日志级别了，这对于线上 vip 用户问题的排查是十分方便的



#### 7、CompositeFilter

CompositeFilter 复合过滤器提供了一种指定多个过滤器的方法。他以Filters元素加入到配置中，元素里面可以配置多个过滤器。该元素不支持添加参数

```xml
<Filters>
    <Marker marker="EVENT" onMatch="ACCEPT" onMismatch="NEUTRAL"/>
    <DynamicThresholdFilter key="loginId" defaultThreshold="ERROR" onMatch="ACCEPT" onMismatch="NEUTRAL">
        <KeyValuePair key="user" value="DEBUG"/>
    </DynamicThresholdFilter>
</Filters>
```



#### 8、OtherFilter

- StringMatchFilter：如果格式化后(即：最终)的日志信息中包含${指定的字符串}，则onMatch，否则onMismatch
  即： `msg.contains(this.text) ? onMatch : onMismatch;`
- LevelRangeFilter：若${maxLevel} <= 日志级别 <= ${minLevel}， 则onMatch，否则onMismatch
  如： 即为只记录日志info及warn级别的日志
- RegexFilter：如果日志信息匹配${指定的正则表达式}，则onMatch，否则onMismatch
  注：可通过useRawMsg属性来控制这个日志信息是格式化处理后(即：最终)的日志信息，还是格式化处理前(即：代码中输入)的日志信息
- ThresholdFilter：若日志级别 >= ${指定的日志级别}， 则onMatch，否则onMismatch
- LevelMatchFilter：如果日志级别等于${指定的日志级别}，则onMatch，否则onMismatch
- ThreadContextMapFilter：通过context(可以理解为一个Map)中对应的key-value值进行过滤。注：上下文默认是ThreadContext，也可以自定义使用ContextDataInjectorFactory配置ContextDataInjector来指定
- DynamicThresholdFilter：若上下文中包含指定的key，则触发DynamicThresholdFilter生效；若该key对应的value值等于任意一个我们指定的值，那么针对本条日志，可记录日志级别的约束下限调整为指定的级别。注：上下文默认是ThreadContext，也可以自定义使用ContextDataInjectorFactory配置ContextDataInjector来指定
- CompositeFilter：组合过滤器,即：按照xml配置中的配置，一个过滤器一个过滤器的走，如果在这过程中，任意一个过滤器ACCEPT或DENY了，那么就不会往后走了，直接返回对应的结果
- TimeFilter：如果记录日志时的当前时间落在每天指定的时间范围[start, end]内，则onMatch，否则onMismatch
  如：`<TimeFilter start="05:00:00" end="05:30:00" onMatch="ACCEPT" onMismatch="DENY"/>`
- ScriptFilter：是否匹配取决于指定的脚本返回值是否为true
- DenyAllFilter：This filter causes all logging events to be dropped
- BurstFilter：对低于或等于${指定日志级别}的日志，进行限流控制
- NoMarkerFilter：如果从对应事件对象获取(LogEvent#getMarker)到的marker为null， 则onMatch，否则onMismatch
- MarkerFilter：如果从对应事件对象获取(LogEvent#getMarker)到的marker为null， 则onMatch，否则onMismatch
- MapFilter：如果从对应事件对象获取(LogEvent#getMarker)到的marker的name值为等于${指定的值}， 则onMatch，否则onMismatch
- The MapFilter allows filtering against data elements that are in a MapMessage.注：需要使用org.apache.logging.log4j.Logger进行记录，且记录org.apache.logging.log4j.message.MapMessage日志，才会生效。注：因为暂时不兼容Slf4j这里不多作说明
- StructuredDataFilter：The StructuredDataFilter is a MapFilter that also allows filtering on the event id, type and message.注：需要使用org.apache.logging.log4j.Logger进行记录，且记录
  org.apache.logging.log4j.core.filter.StructuredDataFilter日志，才会生效。注：因为暂时不兼容Slf4j这里不多作说明



### 4、Layout

官网地址：http://logging.apache.org/log4j/2.x/manual/layouts.html

#### 1、PatternLayout

PatternLayout：用于指定输出日志的格式，控制台或文件输出源（Console、File、RollingRandomAccessFile）都必须包含一个PatternLayout节点，用于指定输出的格式（如 日志输出的时间 文件 方法 行数 等格式），例如 pattern="%d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5level %logger{0} - %msg%n"

```xml
<!--配置方式一: 推荐使用-->
<PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5level %logger{0} - %msg%n" charset="UTF-8"/>
<!--配置方式二-->
<PatternLayout charset="UTF-8">
    <Pattern>%d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5level %logger{0} - %msg%n</Pattern>
</PatternLayout>
```

具体详情可以参考官网或本人的Logback中的记录：

| Conversation Word           | Effect                                                       |
| --------------------------- | ------------------------------------------------------------ |
| **变量名**                  |                                                              |
| %logger、%c                 | 当前日志名称，如: Slf4jAndLogbackMain                        |
| %class、%C                  | 日志调用所在类，如: com.dragon.study.log.Slf4jAndLogbackMain |
| %file、%F                   | 日志调用所在文件，如: com.dragon.study.log.Log4j2Main Log4j2Main.java |
| %method、%M                 | 日志所在方法，如: main                                       |
| %thread、%t                 | 日志调用所有线程序,如：main                                  |
| %level、%p                  | 日志级别，如：INFO                                           |
| %date、%d                   | 日期，如: 2018-12-15 21:40:12,890                            |
| %msg、%m                    | 日志记录内容                                                 |
| %processId、%pid            | 进程号                                                       |
| %threadId、%tid             | 线程号                                                       |
| %exception、%ex             | 异常记录                                                     |
| %n                          | 行分隔                                                       |
| 特殊占位符                  |                                                              |
| %X{user}                    | 表示可以获取外部自定义传入的值, 如：`%X{user}`=》`org.slf4j.MDC.put("user", "xx-yy-zz");` |
| **宽度设置**                | 可以限制上面的ConversationWord的宽度和左右对齐               |
| %20logger                   | 最小宽度20，当小于20时，则左侧留空白。右对齐                 |
| %-20logger                  | 最小宽度20，当小于20时，则右侧留空白。左对齐                 |
| %.30logger                  | 最大宽度30，超出时从头部开始截断。如：%.2、test=》st         |
| %.-30logger                 | 最大宽度30，超出时从末尾开始截断。如：%.-2、test=》te        |
| %20.30logger                | 最小20，最大30，小于20的时候右对齐，大于30的时候开始处截断   |
| %-20.30logger               | 最小20，最大30，小于20的时候左对齐，大于30的时候开始处截断   |
| **显示设置**                |                                                              |
| %highlight()                | 突出显示，如：%highlight(%-5level)                           |
| %green(%red、%blue、%white) | 字体显示为指定颜色                                           |
| {length}                    | 可指定长度，如：%logger{36}                                  |



#### 2、JSONLayout

JSONLayout 表示日志以JSON的格式输出，如下是常用参数，其他参数请参考官网

- complete：如果配置了complete="true"，那么JSON将以完整的形式输出，默认为false
- compact：是否对JSON进行压缩，如果为 true，则附加程序不使用行尾和缩进。 默认为false
- eventEol：如果为 true，则 appender 在每条记录之后附加一个行尾。 默认 false。与 eventEol=true 和 compact=true 一起使用以获取每行一条记录
- locationInfo：如果为 true，则附加程序在生成的 JSON 中包含位置信息。 默认 false

```xml
<RollingFile name="RollingFile" fileName="logs/app.log" filePattern="logs/app-%d{MM-dd-yyyy}.log.gz">
    <JsonLayout />
    <TimeBasedTriggeringPolicy />
</RollingFile>
```

```json
{
  "timeMillis" : 1550798980964,
  "thread" : "main",
  "level" : "INFO",
  "loggerName" : "com.cimu.Log4j2Test",
  "message" : "log4j2",
  "endOfBatch" : true,
  "loggerFqcn" : "org.apache.logging.log4j.spi.AbstractLogger"
}
```

```xml
<RollingFile name="RollingFile" fileName="logs/app.log" filePattern="logs/app-%d{MM-dd-yyyy}.log.gz">
    <JSONLayout complete="true" compact="true" eventEol="true" locationInfo="true" />
    <TimeBasedTriggeringPolicy />
</RollingFile>
```

```json
{"timeMillis":1550799247113,"thread":"main","level":"INFO","loggerName":"com.cimu.Log4j2Test","message":"log4j2","endOfBatch":true,"loggerFqcn":"org.apache.logging.log4j.spi.AbstractLogger"}
```

注意：提醒使用JsonLayout是需要依赖Jackson包的（如何使使用SpringBoot的话web中自带了Jackson包）



#### 3、OtherLayout

还有其他Lyaout就不一一介绍了，具体用到可以参考官网

- CSVLayouts：以逗号分隔值(CSV)记录日志，需要Apache Commons CSV 1.4及以上。可以使用CsvParameterLayout或CsvLogEventLayout来配置
- GELFLayout：以扩展的日志格式（GELF）显示日志
- HTMLLayout：以HTML的形式输出
- JSONLayout：以JSON格式输出
- PatternLayout：以自定义格式输出
- RFC5424Layout：以标准的RFC5424进行日志输出
- SerializedLayout：以JAVA序列进行日志输出
- SyslogLayout：以BSD Syslog进行日志输出
- XMLLayout：以XML格式输出
- YAMLLayout：以YAML格式输出



### 5、Policy & Strategy 

> Log4j2的Policy触发策略与Strategy滚动策略配置详解

#### 1、Policy触发策略

- SizeBasedTriggeringPolicy：基于日志文件大小的触发策略。单位有：KB，MB，GB
- CronTriggeringPolicy：基于`Cron`表达式的触发策略，很灵活
- TimeBasedTriggeringPolicy：基于时间的触发策略。该策略主要是完成周期性的log文件封存工作
  1. interval：integer型，指定两次封存动作之间的时间间隔。这个配置需要和filePattern结合使用，filePattern日期格式精确到哪一位，interval也精确到哪一个单位。注意filePattern中配置的文件重命名规则是%d{yyyy-MM-dd HH-mm-ss}-%i，最小的时间粒度是ss，即秒钟。
     TimeBasedTriggeringPolicy默认的size是1，结合起来就是每1秒钟生成一个新文件。如果改成%d{yyyy-MM-dd HH}，最小粒度为小时，则每一个小时生成一个文件
  2. modulate：boolean型，说明是否对封存时间进行调制。若modulate=true， 则封存时间将以0点为边界进行偏移计算。比如，modulate=true，interval=4hours， 那么假设上次封存日志的时间为03:00，则下次封存日志的时间为04:00， 之后的封存时间依次为08:00，12:00，16:00
- OnStartupTriggeringPolicy：每次JVM启动，都滚动到新的日志文件开始记录
- 等等其他Policy，可以参考官网

#### 2、Strategy滚动策略

- DefaultRolloverStrategy：默认滚动策略。常用参数：max，保存日志文件的最大个数，默认是7，大于此值会删除旧的日志文件
- DirectWriteRolloverStrategy：日志直接写入由文件模式表示的文件

这两个Strategy都是控制如何进行日志滚动的，平时大部分用DefaultRolloverStrategy就可以了



## 5、Log4j2异步日志

官网详细介绍：http://logging.apache.org/log4j/2.x/performance.html

Log4j2最大的特点就是异步日志，其性能的提升主要也是从异步日志中受益。上述学习日志默认都是同步的，同步就是当输出日志时，必须等待日志输出语句执行完毕后，才能执行后面的业务逻辑语句。异步日志就是分配单独的线程做日志记录。

- 同步日志：

  ```
  请求进入 »»» 全局filter进行Level过滤 »»» Logger自身filter进行Level过滤 »»» 生成日志输出Message
                                                                                 ↓              
  输出日志 ««« PatternLayout格式化LogEvent ««« Logger自身filter进行Level过滤 ««« 生成Logevent
  ```

- 异步日志：

  ```
  请求进入 »»» 全局filter进行Level过滤 »»» Logger自身filter进行Level过滤 »»» 生成日志输出Message
            输出日志   «««   Appender ──┐                                        ↓
            输出日志   «««   Appender ──┤   «««  BlockingQueue<LogEvent>  «««  AsyncAppender
            输出日志   «««   Appender ──┘



### 1、异步组件简介

Log4j2 异步提供了两种实现日志的方式，一个是通过AsyncAppender【几乎没人用】，一个是通过AsyncLogger【主要是这个】，分别对应前面我们说的Appender组件和Logger组件。【**注意**：AsyncLogger是基于Disruptor实现，所以必须引入com.lmax.disruptor依赖】

> 这是两种不同的实现方式，在设计和源码上都是不同的体现，可以将这两种实现日志的方式看做完全不一样。

Log4j2 异步日志在如上两种实现方式的基础上，又分为全局异步和混合异步2种方式

- 全局异步：所有日志都异步记录
- 混合异步：在应用种同时使用同步日志和异步日志，这样的日志配置更加灵活



1、AsyncAppender 方式（实际中用的少，因为性能低下）

AsyncAppender 是通过引用别的 Appender 来实现的，当有日志事件到达时，会开启另外一个线程来处理他们。AsyncAppender默认使用Java中自带的类库（util类库），不需要导入外部类库。AsyncAppender应该是在它引用Appender之后配置，因为如果在Appender的时候出现异常，对应用来说是无法感知的。当使用此 Appender的时候，在多线程的环境需要注意，阻塞队列容易受到锁争用的影响，这可能会对性能产生影响。这时我们需要使用无锁的异步记录器AsyncLogger

2、AsyncLogger 方式（实际中用的多，因为性能高）

- AsyncLogger才是log4j2实现异步最重要的功能体现，也是官方推荐的异步方式
- 它可以使调用Logger.log返回更快。其中包括全局异步和混合异步
- 全局异步：所有的日志都异步的记录，在配置文件上下不用做任何的改动，只需要在jvm启动的时候增加一个参数即可实现，实际开发中使用较少。
- 混合异步：你可以同时在应用中使用同步日志和异步日志，这使得日志的配置方式更加的灵活。混合日志需要修改配置文件来实现，使用AsyncLogger标记配置，实际开发中使用较多。



### 2、AsyncAppender

> AsyncAppender ：性能提升较小，不推荐使用

#### 1、全局异步实现

使用AsyncAppender方式实现的全局异步日志输出

1、在 log4j.xml配置文件的 Appenders标签中，对于异步进行配置，使用Async标签

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="10" name="myApp">
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <PatternLayout pattern="%m%n"/>
        </Console>
        <!-- 配置异步日志 AsyncAppender -->
        <Async name="asyncConsole">
            <!-- 将控制台输出做异步的操作 -->
            <AppenderRef ref="consoleAppender"/>
        </Async>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="asyncConsole"/>
        </Root>
    </Loggers>
</Configuration>
```

2、编写测试代码，测试全局异步输出日志信息

```java
package com.xyz;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4j2Test {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Log4j2Test.class);
        // 日志的记录
        for (int i = 0; i < 3; i++) {
            logger.error("error信息");
            logger.warn("warn信息");
            logger.info("info信息");
            logger.debug("debug信息");
            logger.trace("trace信息");
        }
        // 系统业务逻辑
        for (int i = 0; i < 3; i++) {
            // 这里必须使用System.out输出,如果单独使用logger.xx输出看不出来效果
            logger.error("-----start-----");
            System.out.println("-----sys-----"); // 实际上只有这一行是业务代码
            logger.error("-----end-----");
        }
    }
}
```

3、输出结果（观察结果可以发现，此时实现了异步日志的输出）

```
-----sys-----
-----sys-----
-----sys-----
error信息
error信息
error信息
-----start-----
-----end-----
-----start-----
-----end-----
-----start-----
-----end-----
```

4、实际上可以得出结论：Log4j2的异步只是把打印日志单独提出一个线程（日志的打印还是顺序的），而业务的代码在主线程上先执行了。



#### 2、混合异步实现

使用AsyncAppender方式实现同时在应用中使用同步日志和异步日志输出。

1、在上面的基础上修改log4j.xml配置文件，自定义Logger-com.xyz，让其引用同步Appender，rootLogger依旧为异步的AsyncAppender

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="10" name="myApp">
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <PatternLayout pattern="%m%n"/>
        </Console>
        <!-- 配置异步日志 AsyncAppender -->
        <Async name="asyncConsole">
            <AppenderRef ref="consoleAppender"/>
        </Async>
    </Appenders>
    <Loggers>
        <!--
            自定义logger,让自定义的logger引用同步Appender(也可以引用异步AsyncAppender)
            includeLocation="false" 关闭日志记录的行号信息,开启的话会严重影响异步输出的性能
            additivity="false" 表示不再继承rootLogger对象
        -->
        <Logger name="com.xyz" level="error" includeLocation="false" additivity="false">
            <!-- 将com.xyz下的日志设置为控制台输出,同步打印 -->
            <AppenderRef ref="consoleAppender"/>
        </Logger>
        <Root level="error">
            <!--root根logger为异步控制台输出-->
            <AppenderRef ref="asyncConsole"/>
        </Root>
    </Loggers>
</Configuration>
```

2、编写测试代码，测试混合异步（同步和异步同时存在）输出日志信息（实际只测试了同步，想测试异步修改Logger对象包即可）

```java
package com.xyz;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4j2Test {
    public static void main(String[] args) {
        // 当前测试的是自定义的com.xyz的同步logger,如果想测试异步只需修改成其他名称即可
        Logger logger = LoggerFactory.getLogger(Log4j2Test.class);
        // 日志的记录
        for (int i = 0; i < 3; i++) {
            logger.error("error信息");
            logger.warn("warn信息");
            logger.info("info信息");
            logger.debug("debug信息");
            logger.trace("trace信息");
        }
        // 系统业务逻辑
        for (int i = 0; i < 3; i++) {
            // 这里必须使用System.out输出,如果单独使用logger.xx输出看不出来效果
            logger.error("-----start-----");
            System.out.println("-----sys-----"); // 实际上只有这一行是业务代码
            logger.error("-----end-----");
        }
    }
}
```

3、输出结果（观察结果可以发现，此时打印结果是按顺序执行的，这就代表实现了混合异步输出，这里没有重复再次测试异步输出，可自行测试）

```
error信息
error信息
error信息
-----start-----
-----sys-----
-----end-----
-----start-----
-----sys-----
-----end-----
-----start-----
-----sys-----
-----end-----
```



### 3、AsyncLogger

> AsyncLogger：性能提升较大，推荐使用

#### 1、全局异步实现

1、全局异步 ：所有的日志都是异步的日志记录，在配置文件上不用做任何的改动。有如下三种配置方式。

1. 全局启用异步Logger方案一 ：JVM启动加上如下参数

   ```bash
   -DLog4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
   ```

2. 全局启用异步Logger方案二：classpath中添加文件`log4j2.component.properties`，文件增加以下内容：

   ```properties
   Log4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
   ```

3. 全局启用异步Logger方案三：如果是SpringBoot项目可以改为在启动类的地方修改

   ```java
   @SpringBootApplication
   public class Log4j2Application {
       public static void main(String[] args) {
           System.setProperty("log4j2.contextSelector", "org.apache.logging.log4j.core.async.AsyncLoggerContextSelector");
           SpringApplication.run(Log4j2Application.class, args);
       }
   }
   ```

2、使用AsyncLogger实现异步日志必须要要引入Disruptor依赖，不然开启全局异步或者使用到AsyncLogger的时候都会报错。

```xml
<!--Log4j-2.9及更高版本在类路径上需要disruptor-3.3.4.jar或更高版本。在Log4j-2.9之前，需要disruptor-3.0.0.jar或更高版本-->
<!-- 异步日志依赖 -->
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.3.7</version>
</dependency>
```

3、正常在log4j.xml配置文件增加日志输出配置（配置文件与同步配置无区别）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="10" name="myApp">
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <PatternLayout pattern="%m%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="consoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

4、编写测试代码，测试全局异步输出日志信息

```java
package com.xyz;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4j2Test {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Log4j2Test.class);
        // 日志的记录
        for (int i = 0; i < 3; i++) {
            logger.error("error信息");
            logger.warn("warn信息");
            logger.info("info信息");
            logger.debug("debug信息");
            logger.trace("trace信息");
        }
        // 系统业务逻辑
        for (int i = 0; i < 3; i++) {
            // 这里必须使用System.out输出,如果单独使用logger.xx输出看不出来效果
            logger.error("-----start-----");
            System.out.println("-----sys-----"); // 实际上只有这一行是业务代码
            logger.error("-----end-----");
        }
    }
}
```

5、输出结果（观察结果可以发现，此时实现了异步日志的输出）

```
-----sys-----
-----sys-----
-----sys-----
error信息
error信息
error信息
-----start-----
-----end-----
-----start-----
-----end-----
-----start-----
-----end-----
```



#### 2、混合异步实现

混合异步：就是同时在应用中使用同步日志和异步日志，这使得日志的配置方式更加灵活，

- 需求：自定义一个异步Logger–com.xyz（当前包下的日志输出都为异步的），设置rootLogger是同步的（其他都为同步输出）

- 注意：在使用混合异步模式时，一定要将全局的异步配置注释掉（resources下的properties）

  ```properties
  #Log4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
  ```

1、修改log4j2.xml文件中的配置信息，增加异步Logger（AsyncLogger），如下配置：com.xyz 日志是异步的， root 日志是同步的

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorinterval="10" name="myApp">
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <PatternLayout pattern="%m%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <!--
            自定义异步logger对象
            includeLocation="false" 关闭日志记录的行号信息,开启的话会严重影响异步输出的性能
            additivity="false" 表示不再继承rootLogger对象
         -->
        <AsyncLogger name="com.xyz" level="error" includeLocation="false" additivity="false">
            <AppenderRef ref="consoleAppender"/>
        </AsyncLogger>
        <!-- RootLogger设置为同步日志输出,Root元素实际上也有对于的AsyncRoot,一般很少用 -->
        <Root level="error">
            <AppenderRef ref="consoleAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

2、异步日志输出：编写测试代码，测试混合异步输出日志信息，这里先测试异步日志输出，使用对应设置的Logger即可

```java
package com.xyz;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4j2Test {
    public static void main(String[] args) {
        // 当前测试的是自定义的com.xyz的异步logger,如果想测试同步只需修改成其他名称即可
        Logger logger = LoggerFactory.getLogger("com.xyz");
        // 日志的记录
        for (int i = 0; i < 3; i++) {
            logger.error("error信息");
            logger.warn("warn信息");
            logger.info("info信息");
            logger.debug("debug信息");
            logger.trace("trace信息");
        }
        // 系统业务逻辑
        for (int i = 0; i < 3; i++) {
            // 这里必须使用System.out输出,如果单独使用logger.xx输出看不出来效果
            logger.error("-----start-----");
            System.out.println("-----sys-----");
            logger.error("-----end-----");
        }
    }
}
```

3、输出结果（观察结果可以发现，此时实现了异步日志的输出）

```
-----sys-----
-----sys-----
-----sys-----
error信息
error信息
error信息
-----start-----
-----end-----
-----start-----
-----end-----
-----start-----
-----end-----
```

4、同步日志输出：编写测试代码，测试混合异步输出日志信息，这里测试同步日志输出，使用RootLogger即可（实际就修改定义的Logger）

```java
// 当前测试使用的是RootLogger,RootLogger设置的是同步日志输出
Logger logger = LoggerFactory.getLogger("com");
```

5、输出结果（观察结果可以发现，此时实现了同步日志的输出）

```
error信息
error信息
error信息
-----start-----
-----sys-----
-----end-----
-----start-----
-----sys-----
-----end-----
-----start-----
-----sys-----
-----end-----
```



### 4、AsyncRoot

AsyncRoot 实际就是根 AsyncLogger，其用法及功能与 AsyncLogger 一摸一样（全局混合异步），参考上面的 AsyncLogger 即可。

总结一下Log4j2 异步化日志的三种方式：

1. 全局启用异步Logger方案一：JVM启动参数（boot.ini）加上如下参数

   ```bash
   -DLog4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector

2. 全局启用异步Logger方案二：classpath中添加文件“log4j2.component.properties”，文件增加以下内容：

   ```properties
   Log4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
   ```

3. 手工指定部分Logger采用异步方式：log4j2.xml配置文件中使用AsyncRoot 和 AsyncLogger 替代 Root 和 Logger

4. （上述3种方式任选其一即可，不要同时采用）



### 5、异步配置注意

使用异步日志需要注意的问题：

- 如果使用异步日志，AsyncAppender、AsyncLogger 和 全局日志，不要同时出现。性能会和AsyncAppender 一致，降至最低。
- 需要设置 includeLocation=false，否则打印位置信息会急剧降低异步日志的性能，比同步日志还要慢。
- 当配置AsyncLoggerContextSelector作为（全局）异步日志时，请确保在配置中使用普通的Root和Logger元素。AsyncLoggerContextSelector将确保所有记录器都是异步的，使用的机制与配置AsyncRoot> 或AsyncLogger时的机制不同

通过`log.info("是否为全局异步日志：{}", AsyncLoggerContextSelector.isSelected());`可以查看是否为全局异步日志

```java
import org.apache.logging.log4j.core.async.AsyncLoggerContextSelector;
import org.slf4j.LoggerFactory;

LoggerFactory.getLogger("com").error("com是否为异步日志：{}", AsyncLoggerContextSelector.isSelected());
LoggerFactory.getLogger("com.xyz").error("xyz是否为异步日志：{}", AsyncLoggerContextSelector.isSelected());
```

Log4j2 最牛的地方在于异步输出日志时的性能表现，Log4j2 在多线程的环境下吞吐量与 Log4j 和 Logback 的比较可查官网图片。三种模式对比：

1. 全局使用异步模式（Log4j2 的性能较之 Log4j 和Logback有很大的优势）
2. 部分Logger采用异步模式（Log4j2 的性能较之 Log4j 和Logback有很大的优势）
3. 异步Appenderf（Log4j2 对比 Logback性能上差不多）

| 日志输出类型  | 日志输出方式                                                 |
| ------------- | ------------------------------------------------------------ |
| sync          | 同步打印日志，日志输出与业务逻辑在同一线程内，当日志输出完毕，才能进行后续业务逻辑操作 |
| AsyncAppender | 异步打印日志，内部采用ArrayBlockingQueue，对每个AsyncAppender创建一个线程用于处理日志输出 |
| AsyncLogger   | 异步打印日志，采用了高性能并发框架Disruptor，创建一个线程用于处理日志输出（要引入com.lmax.disruptor） |



### 6、异步日志原理

- 应用程序的主线程调用打印记录日志的方法后，传递到logEvent对象
- event将对象加入到一个阻塞队列（ArrayBlockingQueue）中
- 此时，主线程的工作就已经完成了，其余工作交给log异步线程就可以了
- 阻塞队列（ArrayBlockingQueue）收到日志消息后，立马将消息发给Appender对象
- Appender收到消息后开始打输出消息



## 6、SprintBoot集成Log4j2

### 1、SpringBoot日志依赖关系

1. sping-boot-starter 与 spring-boot-starter-web 都依赖 sping-boot-starter-looging

2. 而SpringBoot底层默认使用 SLF4J+ Logback 方式进行日志记录，SpringBoot 使用中间替换包把其的日志框架都替换成了SLF4J

3. SpringBoot 能自动适配所有的日志框架，引入其他框架时，只需要把sping-boot-starter-looging依赖的日志框架排除掉即可

   ```xml
   <!-- 如果有引用spring-boot-starter也需要排除-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
       <exclusions>
           <exclusion> <!--移除掉 spring boot 默认的日志启动器-->
               <artifactId>spring-boot-starter-logging</artifactId>
               <groupId>org.springframework.boot</groupId>
           </exclusion>
       </exclusions>
   </dependency>
   
   <!-- web 启动器-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
       <exclusions>
           <!--移除掉 spring boot 默认的日志启动器-->
           <exclusion>
               <artifactId>spring-boot-starter-logging</artifactId>
               <groupId>org.springframework.boot</groupId>
           </exclusion>
       </exclusions>
   </dependency>
   
   <!--引用 log4j2 spring boot 启动器，内部依赖了 slf4j、log4j-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-log4j2</artifactId>
   </dependency>
   ```



### 2、SpringBoot全局日志设置

官网配置详解（适用Logback与Log4j2）https://docs.spring.io/spring-boot/docs/2.5.0/reference/htmlsingle/#application-properties.core

1. 日志的级别由低到高分别为：trace(跟踪) < debug(调试) < info(信息) < warn(警告) < error(错误)

2. 在配置文件中调整输出的日志级别，日志就只会在这个级别及以后的高级别生效，SpringBoot 默认使用 INFO 级别（Log4j2 默认 ERROR）

3. SpringBoot 的全局配置文件 "application.properties" 或 "application.yml" 中可以修改日志配置项：

   ```properties
   # 默认名log4j2-spring.xml,如果要设置其他名称则需要如下配置
   # logging.config=classpath:log4j2-spring.xml
   # 设置根节点的日志级别输出,root表示整个项目
   logging.level.root=INFO
   # 指定特定包及类的日志输出级别,未指定就按root设置的级别输出,如果root也未指定则按SpringBoot的默认级别info输出
   logging.level.org.springframework.web=DEBUG
   logging.level.org.hibernate=ERROR
   logging.level.com.xyz=DEBUG
   
   # 配置日志输出的文件,这两个选一个配置就可以了,一起配置的话,name的生效. 每次启动都是追加日志
   # .name=具体文件, 写入指定的日志文件.文件名可以是确切的位置,也可以是相对于当前目录的位置
   # .path=具体目录, 写入spring.log文件指定目录.目录名可以是确切的位置,也可以是相对于当前目录的位置
   logging.file.name=logs/log4j2.log
   logging.file.path=logs/
   
   # 指定控制台输出的日志格式,例如: %d{yyyy-MM-dd HH:mm:ss} -- [%thread] %-5level %logger{50} %msg%n
   # 1、%d 表示日期时间，
   # 2、%thread 表示线程名，
   # 3、%‐5level 级别从左显示5个字符宽度
   # 4、%logger{50} 表示logger名字最长50个字符，否则按照句点分割。
   # 5、%msg 日志消息，
   # 6、%n 换行符
   # 7、%line 显示日志输出位置的行号，方便寻找位置
   # 8、%X 特殊占位符，%X{uu_id}是获取uu_id的值，代码中设置uu_id值:org.slf4j.MDC.put("uuid", "xx-yy-zz");
   # 配置日志输出格式, .file是配置输出到文件的日志格式, .console是配置输出到控制台的日志格式
   logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{50}:%line %msg%n
   logging.pattern.file=%d{yyyy-MM-dd HH:mm} -- [%thread] %-5level %logger{50} %msg%n
   
   # 设置日志记录器组,将相关的记录器组合在一起,然后设置日志记录器组的日志级别为TRACE
   logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
   logging.level.tomcat=TRACE
   # SpringBoot包括以下预定义的日志记录组，可以开箱即用
   # web: org.springframework.core.codec, org.springframework.http, org.springframework.web
   # sql: org.springframework.jdbc.core, org.hibernate.SQL
   logging.level.web=INFO
   logging.level.sql=DEBUG
   ```
   
   > - 默认情况，日志文件超过10M时，会新建文件进行递增，如logback.log、logback1.log、logback2.log，使用logging.file.max-size更改大小限制
   > - 默认情况，日志只记录到控制台，不写入日志文件，如果要在控制台输出之外写入到日志文件，则需要设置 .file 或 .path 属性
   > - 默认情况，logging.file.* 等配置使用的都是RollingFileAppender



### 3、SpringBoot + Log4j2示例

#### 1、导入log4j2依赖

SpringBoot默认是用Logback日志框架，需要排除Logback，不然会出现jar依赖冲突报错（spring-boot 2.5.2 + log4j 2.14.1 + JDK 1.8）

```xml
<!-- web 启动器-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion><!--移除掉 spring boot 默认的日志启动器-->
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<!--引用 log4j2 spring boot 启动器，内部依赖了 slf4j、log4j-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### 2、SprintBoot配置文件

如果自定义了Log4j2文件名，需要在application.properties中配置（ 默认名log4j2-spring.xml，就省下了在application.yml中配置），如果只引入了 log4j2 依赖，没有提供 log4j2.xml 配置文件，则默认日志输出级别为 INFO，会打印在控制台，但是不会记录到文件中。实际上可以不使用Log4j2配置文件，只用SpringBoot配置，不过这样能使用的功能是非常有限的（如果是这样还不如使用Logback）

```properties
logging.config=classpath:log4j2-spring.xml
```

#### 3、Log4j2配置文件（log4j2.xml）

log4j2-spring.xml（SpringBoot会自动加载-spring的文件，所以可不配置logging.config），如果要自定义名称则需要配置logging.config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<configuration status="WARN" monitorInterval="30">
    <!-- 配置日志文件输出目录，此配置将日志输出到tomcat根目录下的指定文件夹 -->
    <properties>
        <property name="LOG_HOME">./WebAppLogs/logs</property>
    </properties>
    <!--先定义所有的appender-->
    <appenders>
        <!-- 优先级从高到低分别是 OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL -->
        <!-- 单词解释： Match：匹配 DENY：拒绝 Mismatch：不匹配 ACCEPT：接受 -->
        <!-- DENY，日志将立即被抛弃不再经过其他过滤器； NEUTRAL，有序列表里的下个过滤器过接着处理日志； ACCEPT，日志会被立即处理，不再经过剩余过滤器。 -->
        <!--输出日志的格式
         %d{yyyy-MM-dd HH:mm:ss, SSS} : 日志生产时间
         %p : 日志输出格式
         %c : logger的名称
         %m : 日志内容，即 logger.info("message")
         %n : 换行符
         %C : Java类名
         %L : 日志输出所在行数
         %M : 日志输出所在方法名
         hostName : 本地机器名
         hostAddress : 本地ip地址 -->
        <!--这个输出控制台的配置-->
        <console name="Console" target="SYSTEM_OUT">
            <!--输出日志的格式-->
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] - [%t] [%p] - %logger{1.} - %m%n"/>
            <!--<PatternLayout pattern="[%d{HH:mm:ss:SSS}] - (%F:%l) - %m%n"/>-->
            <!--<PatternLayout pattern="[%d{HH:mm:ss:SSS}] (%F:%L) %m%n" />-->
        </console>
        <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <!-- TRACE级别日志 ; 设置日志格式并配置日志压缩格式，压缩文件独立放在一个文件夹内， 日期格式不能为冒号，否则无法生成，因为文件名不允许有冒号，此appender只输出trace级别的数据到trace.log -->
        <RollingFile name="RollingFileTrace" immediateFlush="true" fileName="${LOG_HOME}/trace.log"
                     filePattern="${LOG_HOME}/trace_%d{yyyy-MM-dd-HH}-%i.log.zip">
            <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] - [%t] [%p] - %logger{36} - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20">
                <!--这里的age必须和filePattern协调, 后者是精确到HH, 这里就要写成xH, xd就不起作用
                    另外, 数字最好>2, 否则可能造成删除的时候, 最近的文件还处于被占用状态,导致删除不成功!-->
                <Delete basePath="${LOG_HOME}" maxDepth="2">
                    <IfFileName glob="trace_*.zip"/>
                    <!-- 保存时间与filePattern相同即可 -->
                    <!-- 如果filePattern为：yyyy-MM-dd-HH:mm:ss, age也可以为5s,表示日志存活时间为5s -->
                    <IfLastModified age="168H"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>

        <RollingFile name="RollingFileDebug" immediateFlush="true" fileName="${LOG_HOME}/debug.log"
                     filePattern="${LOG_HOME}/debug_%d{yyyy-MM-dd-HH}-%i.log.zip">
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] - [%t] [%p] - %logger{36} - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="20">
                <Delete basePath="${LOG_HOME}" maxDepth="2">
                    <IfFileName glob="debug_*.zip"/>
                    <IfLastModified age="168H"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>

        <!-- info日志配置 -->
        <RollingFile name="RollingFileInfo" immediateFlush="true" fileName="${LOG_HOME}/info.log"
                     filePattern="${LOG_HOME}/info_%d{yyyy-MM-dd-HH}-%i.log.zip">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] - [%t] [%p] - %logger{36} - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="20">
                <Delete basePath="${LOG_HOME}" maxDepth="2">
                    <IfFileName glob="info_*.zip"/>
                    <IfLastModified age="168H"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>

        <!-- warn日志配置 -->
        <RollingFile name="RollingFileWarn" immediateFlush="true" fileName="${LOG_HOME}/warn.log"
                     filePattern="${LOG_HOME}/warn_%d{yyyy-MM-dd-HH}-%i.log.zip">
            <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] - [%t] [%p] - %logger{36} - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="20">
                <Delete basePath="${LOG_HOME}" maxDepth="2">
                    <IfFileName glob="warn_*.zip"/>
                    <IfLastModified age="168H"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>

        <!-- error日志配置 -->
        <RollingFile name="RollingFileError" immediateFlush="true" fileName="${LOG_HOME}/error.log"
                     filePattern="${LOG_HOME}/error_%d{yyyy-MM-dd-HH}-%i.log.zip">
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] - [%t] [%p] - %logger{36} - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="20">
                <Delete basePath="${LOG_HOME}" maxDepth="2">
                    <IfFileName glob="error_*.zip"/>
                    <IfLastModified age="168H"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <loggers>
        <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
        <logger name="org.springframework" level="INFO"/>
        <logger name="org.mybatis" level="INFO"/>
        <root level="all">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileDebug"/>
            <appender-ref ref="RollingFileTrace"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
        </root>
    </loggers>
</configuration>
```

> Log4j2配置详解请参考前面的教程【自定义配置文件】

这是转载其他人整理的百分号属性参数说明大全：

```
    ********************************************************************************************************************
    参数        说明                                         举例                    输出显示媒介
    ********************************************************************************************************************
    %c          列出logger名字空间的全称，如果加上{<层数>}，   假设当前logger的命名空间是"a.b.c"
                则表示列出从最内层算起的指定层数的名字空间
                                                            %c                  a.b.c
                                                            %c{2}               b.c
                                                            %20c                （若名字空间长度小于20，则左边用空格填充）
                                                            %-20c               （若名字空间长度小于20，则右边用空格填充）
                                                            %.30c               （若名字空间长度超过30，截去多余字符）
                                                            %20.30c             （若名字空间长度小于20，则左边用空格填充；
                                                                                    若名字空间长度超过30，截去多余字符）
                                                            %-20.30c            （若名字空间长度小于20，则右边用空格填充；
                                                                                    若名字空间长度超过30，截去多余字符）
    ********************************************************************************************************************
    %C          列出调用logger的类的全名（包含包路径）         假设当前类是"org.apache.xyz.SomeClass"
                                                            %C                  org.apache.xyz.SomeClass
                                                            %C{1}               SomeClass
    %class
    ********************************************************************************************************************
    %d          显示日志记录时间，{<日期格式>}使用ISO8601定义的日期格式
                                                            %d{yyyy/MM/dd HH:mm:ss,SSS}     2005/10/12 22:23:30,117
                                                            %d{ABSOLUTE}        22:23:30,117
                                                            %d{DATE}            12 Oct 2005 22:23:30,117
                                                            %d{ISO8601}         2005-10-12 22:23:30,117
    ********************************************************************************************************************
    %F          显示调用logger的源文件名                     %F                   MyClass.java
    ********************************************************************************************************************
    %l          显示日志事件的发生位置，包含包路径、方法名、
                源文件名，以及在代码中的行数                  %l                   com.a.b.MyClass.main(MyClass.java:168)
    ********************************************************************************************************************
    %L          显示调用logger的代码行                       %L                   129
    %line                                                  %line                129
    ********************************************************************************************************************
    %level      显示该条日志的优先级                         %level               INFO
    %p                                                     %p                   INFO
    ********************************************************************************************************************
    %m          显示输出消息                                 %m                  This is a message for debug.
    %message                                                %message            This is a message for debug.
    ********************************************************************************************************************
    %M          显示调用logger的方法名                       %M                   main
    ********************************************************************************************************************
    %n          当前平台下的换行符                           %n                   Windows平台下表示rn，UNIX平台下表示n
    ********************************************************************************************************************
    %p          显示该条日志的优先级                         %p                   INFO
    %level                                                 %level               INFO
    ********************************************************************************************************************
    %r          显示从程序启动时到记录该条日志时已经经过的毫秒数  %r                 1215
    ********************************************************************************************************************
    %t          输出产生该日志事件的线程名                    %t                   http-nio-8080-exec-10
    %thread                                                 %thread              http-nio-8080-exec-10
    ********************************************************************************************************************
    %x          按NDC（Nested Diagnostic Context，线程堆栈）顺序输出日志           假设某程序调用顺序是MyApp调用com.foo.Bar
                                                            %c %x - %m%n        MyApp - Call com.foo.Bar.
                                                                                com.foo.Bar - Log in Bar
                                                                                MyApp - Return to MyApp.
    ********************************************************************************************************************
    %X          按MDC（Mapped Diagnostic Context，线程映射表）
                输出日志。通常用于多个客户端连接同一台服务器，
                方便服务器区分是那个客户端访问留下来的日志。     %X{5}             （记录代号为5的客户端的日志）
    ********************************************************************************************************************
    %%          显示一个百分号                               %%                  %
    ********************************************************************************************************************
```



#### 4、日志实现-Slf4j（LogFactory）

如下使用 slf4j + log4j2 配合打印日志，当前方式也是比较流行的，slf4j 能适配各种日志框架，迁移起来也会非常方便，功能上会log4j2稍逊。

```java
    @Test
    void slf4jTest() {
        org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger("slf4jTest");
        log.error("hello, I am error!");
        log.warn("hello, I am warn!");
        log.info("hello, I am {}!", "info");
        log.debug("hello, I am debug!");
        log.trace("hello, I am trace!");
    }
```

日志实现 Slf4j 自定义工具类：

```java
public class Slf4jLogUtil {

    /**
     * debug级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void debug(Class clazz, String msg, Object... params) {
        Logger logger = LoggerFactory.getLogger(clazz.getName());
        logger.debug(msg, params);
    }

    /**
     * trace日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void trace(Class clazz, String msg, Object... params) {
        Logger logger = LoggerFactory.getLogger(clazz.getName());
        logger.trace(msg, params);
    }

    /**
     * info级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void info(Class clazz, String msg, Object... params) {
        Logger logger = LoggerFactory.getLogger(clazz.getName());
        logger.info(msg, params);
    }

    /**
     * warn级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void warn(Class clazz, String msg, Object... params) {
        Logger logger = LoggerFactory.getLogger(clazz);
        logger.warn(msg, params);
    }

    /**
     * error级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void error(Class clazz, String msg, Object... params) {
        Logger logger = LoggerFactory.getLogger(clazz);
        logger.error(msg, params);
    }
}
```



#### 5、日志实现-Log4j（LogManager）

```java
    @Test
    void log4j2Test() {
        org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger("log4j2Test");
        log.error("hello, I am error!");
        log.warn("hello, I am warn!");
        log.info("hello, I am {}!", "info");
        log.debug("hello, I am debug!");
        log.trace("hello, I am trace!");
    }
```

日志实现 Log4j2 自定义工具类：

```java
public class Log4j2LogUtil {
    /**
     * debug级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void debug(Class clazz, String msg, Object... params) {
        Logger logger = LogManager.getLogger(clazz.getName());
        logger.debug(msg, params);
    }

    /**
     * trace级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void trace(Class clazz, String msg, Object... params) {
        Logger logger = LogManager.getLogger(clazz.getName());
        logger.trace(msg, params);
    }

    /**
     * info级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void info(Class clazz, String msg, Object... params) {
        Logger logger = LogManager.getLogger(clazz.getName());
        logger.info(msg, params);
    }

    /**
     * warn级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void warn(Class clazz, String msg, Object... params) {
        Logger logger = LogManager.getLogger(clazz);
        logger.warn(msg, params);
    }

    /**
     * error级别日志输出
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void error(Class clazz, String msg, Object... params) {
        Logger logger = LogManager.getLogger(clazz);
        logger.error(msg, params);
    }

    /**
     * fatal日志
     * @param clazz 类
     * @param msg 日志
     * @param params 其他参数
     */
    public static void fatal(Class clazz, String msg, Object... params) {
        Logger logger = LogManager.getLogger(clazz);
        logger.fatal(msg, params);
    }
}
```



#### 6、SpringBoot + Log4j2最佳实践

建议使用 SLF4J门面 + Log4j2门面及实现，搭配同时使用，一般情况打印日志的时候可以使用SLF4J之类的API，需要使用Log4j2更多强大功能时就使用自己的门面API即可（例如：参数格式化以及"惰性"打日志之类的功能）

```java
package com.xyz;

public class Log4j2Test {
    private static final org.slf4j.Logger slf4j = org.slf4j.LoggerFactory.getLogger(Log4j2Test.class);
    private static final org.apache.logging.log4j.Logger log4j = org.apache.logging.log4j.LogManager.getLogger(Log4j2Test.class);
    public static void main(String[] args) {
        // 使用slf4j门面 + log4j实现
        slf4j.info("test: {}", "xx");
        // 使用log4j门面 + log4j实现
        log4j.debug("test：{}",() -> "xx");
    }
}
```



### 4、动态切换日志级别的几种方式

#### 1、monitorInterval 定时读取配置信息

1. 热加载配置文件是很有用的，比如服务上线之后，当需要输出详细日志信息排查问题时，可以降低优先级，比如：TRACE 、DEBUG，当完事之后，可以再调高优先级，比如 WARN、ERROR，减少日志记录，提供性能

2. Log4j2 与 Logback 一样，可以在修改后自动重新加载其配置，与 Logback 不同的是，它在进行重新配置时不会丢失日志事件

3. 使用非常简单只需要在文件开始的 configuration 标签中指定 monitorInterval 属性即可，值大于 0 时会自动创建子线程进行定时扫描，更新配置；不配置或者值为0时，不会进行监控，运行中修改是无效的：

   ```xml
   <!--monitorInterval：监视配置文件变化间隔时间，单位秒，Log4j2 能够自动检测配置文件是否修改，同时更新配置-->
   <configuration status="WARN" monitorInterval="60">
   ```

4. 如果是在本地 IDE 编辑器中测试，注意修改的是 classes 编译目录下的 log4j2.xml 文件，而不是 resources 目录的源文件

5. 对于可以手动修改配置文件的时候，此种方式还是很方便的，但是如果应用是打成 .jar、.war 包发布部署，则这种方式还是不太方便，因为仍然得从生产下载 jar、war 包，然后修改配置，重启服务



#### 2、LoggerConfig 动态修改日志记录器

> 比较推荐的一种使用方式

1. 鉴于现在都是微服务开发，分布式部署，而且为了方便管理，基本都是打包后部署在一些平台上面。直接修改日志配置文件不太现实
2. 所以我们可以自己提供一个 Controller 请求接口，用于动态修改日志输出级别，想设置什么级别，只需要调用一下接口，传递一下参数即可，无论是设置整个 root 级别，还是对指定包或者类都能轻松设置，不用重启服务就能立马生效
3. 实现也是非常简单，在线演示源码如下：

```java
package com.xyz;

import org.apache.logging.log4j.Level;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.core.LoggerContext;
import org.apache.logging.log4j.core.config.Configuration;
import org.apache.logging.log4j.core.config.LoggerConfig;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
public class Log4j2Controller {
    private static final Logger log =  LogManager.getLogger(Log4j2Controller.class);
    /**
     * 打印日志测试: http://localhost:8080/logback/logs
     */
    @GetMapping("logback/logs")
    public void logs() {
        log.error("hello, I am error!");
        log.warn("hello, I am warn!");
        log.info("hello, I am {}!", "info");
        log.debug("hello, I am debug!");
        log.trace("hello, I am trace!");
    }

    /**
     * Log4j2 动态修改 root(根)日志级别，修改之后配置立刻生效.
     * http://localhost:8080/log4j/setRootLevel?level=TRACE
     *
     * @param level ：可选值有：ALL、TRACE、DEBUG、INFO、WARN、ERROR、FATAL、OFF，否则默认会设置为 DEBUG，不区分大小写。
     * @return
     */
    @GetMapping(value = "/log4j/setRootLevel")
    public Map<String, Object> setRootLevel(@RequestParam(value = "level") String level) {
        Map<String, Object> dataMap = new HashMap<>();
        dataMap.put("code", 200);
        dataMap.put("msg", "修改root日志级别成功");
        try {
            // LoggerContext getContext(final boolean currentContext)：获取 log4j 日志上下文，false 表示返回合适调用方的上下文
            LoggerContext loggerContext = (LoggerContext) LogManager.getContext(false);

            // 返回当前配置，发生重新配置时，将替换该配置。
            Configuration configuration = loggerContext.getConfiguration();
            // 查找记录器名称的相应 LoggerConfig
            LoggerConfig loggerConfig = configuration.getLoggerConfig(LogManager.ROOT_LOGGER_NAME);

            // 设置日志级别，如果 level 值不属于 ALL、TRACE、DEBUG、INFO、WARN、ERROR、FATAL、OFF，则默认会设置为 DEBUG
            Level toLevel = Level.toLevel(level);
            loggerConfig.setLevel(toLevel);

            // 根据当前配置更新所有记录器
            loggerContext.updateLoggers();

            // 查询 root(根)日志输出级别结果返回
            String root_logger_name = LogManager.getRootLogger().getLevel().name();
            dataMap.put("data", root_logger_name);
        } catch (Exception e) {
            dataMap.put("code", 500);
            dataMap.put("msg", e.getMessage());
            e.printStackTrace();
        }
        return dataMap;
    }
}
```





#### 3、SpringBootActuator 监控管理日志

使用SpringBoot Actuator监控管理日志的优点：

- 无需编码，只需要引用Spring Boot Actuator监控依赖，然后开启访问端点 /loggers，即可轻松查看日志输出级别，并进行切换
- 解耦日志框架，无论使用的 Logback 还是 Log4j2 ，都能轻松切换

1、引入pring Boot Actuator依赖

```xml
<!--监控和管理应用程序-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2、开启日志访问端点： /loggers

```yaml
management:
  endpoint:
    health:
      show-details: ALWAYS # 展示节点的详细信息
  endpoints:
    web:
      exposure:
        include: info,health,logfile,loggers # 指定公开的访问端点
```

3、查看级别

- 发送 GET 请求，获取日志等级：http://localhost:8080/actuator/loggers 
- 返回的信息非常详细，包含了 ROOT，以及程序中各个包和类的日志级别
- 其中 configuredLevel 表示配置级别，effectiveLevel 表示有效级别，configuredLevel  可能为 null，因为没有配置。

4、修改日志级别

- 发送 POST 请求，设置指定包或者类（com.xyz）日志输出级别：http://localhost:8080/actuator/loggers/com.xyz
- 发送 POST 请求，设置 root 全局日志输出级别：http://localhost:8080/actuator/loggers/root
- 请求 Body 的内容格式：{"configuredLevel":"error"}



### 5、整合过程中出现问题解决方法

错误1："Logging system failed to initialize using configuration from 'classpath:log4j2.xml"

错误原因：引用jar时引用了多个logback的框架，由于idea开发工具未根据pom.xml的配置自动配置依赖，未去除之前的log4j的依赖，导致无法解析节点信息

解决办法：在pom.xml文件中右击选择Diagrams，查看依赖图，找到spring.boot.starter-logging依赖，删除即可。

错误2："java.lang.ClassNotFoundException: com.lmax.disruptor.EventFactory"

解决方法：修改配置

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.1</version>
</dependency>
```



## 7、参考资料 & 鸣谢

- Spring Boot 2.x 集成 SLF4j + log4j2 日志框架【CSDN 蚩尤后裔】：https://wangmaoxiong.blog.csdn.net/article/details/117634140
- java日志(四)--jcl和log4j及log4j2使用【CSDN panda-star】：https://blog.csdn.net/chinabestchina/article/details/104744069
- log4j2 的使用【超详细图文】【蚩尤后裔 不埋雷的探长】：https://tanzhang.blog.csdn.net/article/details/110723441
- log4j2使用filter过滤日志【CSDN justry_deng】：https://blog.csdn.net/justry_deng/article/details/109413153
- log4j2四大组件【此木 IT 码农】：http://www.chaiguanxin.com/tags/log4j2

如下是参考微信公众号文章：

- 你知道 log4j2 各项配置的全部含义吗？带你了解 log4j2 的全部组件【小脑斧科技博客】https://mp.weixin.qq.com/s/2XIR4_2UdAMQk-BYVd--vw
- SpringBoot—整合log4j2入门和log4j2.xml配置详解【后端技术解忧铺】https://mp.weixin.qq.com/s/sLo_yMwTxzp52TLBKtdkqA
- Log4j2的正确使用姿势【话题内有log4j2其他教程】【一只小笨龟】https://mp.weixin.qq.com/s/lJ4WSgd2Dg912RW7IaZ-3g

如下是Log4j2自定义组件文章：

- 教你打印自己的日志 -- 如何自定义 log4j2 各组件【小脑斧科技博客】https://mp.weixin.qq.com/s/Xh13NZVoGalgfL1zZwKDWw
- log4j日志级别调整&日志限流【一只小笨龟】https://mp.weixin.qq.com/s/Jc3kBDKl6izuEJIZFKDhxA
- log4j2如何实现日志自脱敏？山人自有妙计！【看点代码再上班】https://mp.weixin.qq.com/s/TAjC0OERPWxyjoeYkb685A



# Logbook HTTP日志框架

1. GitHub 文档及代码地址：https://github.com/zalando/logbook
2. SpringBoot使用Logbook记录HTTP请求响应日志：https://mp.weixin.qq.com/s/9LITBfpGqTDTLbpfzp7tfA

***

> Spring Boot的httptrace端口能够记录每次访问的请求和响应信息，但是不能记录body，这样在出问题时就不方便排查，而且httptrace不方便在原有的基础上进行扩展，所以只能寻求其他方式进行记录。

- 允许web应用记录程序接收或发送的所有HTTP通信
- 易于保留和进行分析

Logbook在大部分情况下是开箱即用的，即使对于一些不常用的技术或者应用，实现它们也非常简单。



## 特性简介

- 日志记录：HTTP请求和响应，包含body；未授权的请求会记录部分日志（不包含body）
- 自定义：能够自定义记录格式、记录方式以及请求记录的条件
- 支持框架：Servlet容器、Apache’s HTTP client、Square's OkHttp等
- 混淆敏感数据
- Spring Boot自动配置
- 兼容 Scalyr
- 合理的默认值



## 快速开始

Logbook为SpringBoot用户提供了很方便的自动配置功能，即我们所熟悉的starter。它使用了合理的默认值自动配置了以下功能：

- Servlet filter
- 适用于未授权请求的Servlet filter（如果检测到项目中使用Spring Security）
- Header过滤器、Parameter过滤器、Body过滤器
- HTTP格式化器、JSON格式化器
- 日志写入方式

引入starter模块后SpringBooot会自动装配

```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>logbook-spring-boot-starter</artifactId>
    <version>2.14.0</version>
    <!--<version>${logbook.version}</version>--><!--当前的最新版是2.14.0-->
</dependency>
```

日志记录器必须配置为trace才能记录请求和响应。 SpringBoot可以通过将以下行添加到 application.properties 来实现

```properties
logging.level.org.zalando.logbook = trace
```

默认配置下，输出的日志为JSON格式：Request、Response

```json
{
    "origin":"remote",
    "type":"request",
    "correlation":"2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b",
    "protocol":"HTTP/1.1",
    "sender":"127.0.0.1",
    "method":"GET",
    "path":"http://example.org/test",
    "headers":{
        "Accept":[
            "application/json"
        ],
        "Content-Type":[
            "text/plain"
        ]
    },
    "body":"Hello world!"
}
```

```json
{
    "origin":"local",
    "type":"response",
    "correlation":"2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b",
    "duration":25,
    "protocol":"HTTP/1.1",
    "status":200,
    "headers":{
        "Content-Type":[
            "text/plain"
        ]
    },
    "body":"Hello world!"
}
```



## 配置选项

下面的展示了SpringBoot中可配置的选项：

| 配置项                           | 描述                                                         |     默认值      |
| :------------------------------- | :----------------------------------------------------------- | :-------------: |
| logbook.include                  | 仅包含某些URL（如果设置的话）                                |       []        |
| logbook.exclude                  | 排除某些URL（会覆盖logbook.include）设置了exclude的url是不会触发logbook |       []        |
| logbook.filter.enabled           | 是否启用`LogbookFilter`                                      |      true       |
| logbook.filter.form-request-mode | 如何处理表单请求                                             |      body       |
| logbook.secure-filter.enabled    | 是否启用SecureLogbookFilter（同时项目中使用Spring Security才会生效） |      true       |
| logbook.format.style             | 格式化样式（http，json，curl，splunk）                       |      json       |
| logbook.strategy                 | 策略（default，status-at-least， body-only-if-status-at-least，without-body） |     default     |
| logbook.minimum-status           | 启用日志记录的最小HTTP响应状态值，当策略值为status-at-least或body-only-if-status-at-least时设置 |       400       |
| logbook.obfuscate.headers        | 需要混淆的HTTP Header集合，默认脱敏符为X                     | [Authorization] |
| logbook.obfuscate.paths          | 需要混淆的path集合，默认脱敏符为X                            |       []        |
| logbook.obfuscate.parameters     | 需要混淆的parameter集合，默认脱敏符为X                       | [access_token]  |
| logbook.write.chunk-size         | 日志拆分块的大小，默认不拆分                                 |    0 (禁用)     |
| logbook.write.max-body-size      | 截取Body的最大长度，后面使用`...`拼接                        |                 |

```yaml
# 使用logbook需要注意的配置,设置org.zalando.logbook包的日志输出级别为trace,不然无法输出日志
logging:
  level:
    org.zalando.logbook: trace

# 如下为logbook的配置
logbook:
  include:
    - /api/**
  exclude:
    - /actuator/**
  filter:
    enabled: true
  secure-filter:
    enabled: true
  format:
    style: json
  minimum-status: 400
  obfuscate:
    headers:
      - Authorization
      - X-Secret
    parameters:
      - access_token
      - password
    paths:
      - user_id
  write:
    chunk-size: 1000
    max-body-size: 10000
```



## 详细用法

所有的功能集成都需要一个Logbook实例来完成，它保存了所有的配置并将所有需要的组件连接在一起。你可以使用所有的默认值创建一个实例：

```java
Logbook logbook = Logbook.create();
```

或使用以下命令创建自定义版本：LogbookBuilder

```java
Logbook logbook = Logbook.builder()
    .condition(new CustomCondition())
    .queryFilter(new CustomQueryFilter())
    .pathFilter(new CustomPathFilter())
    .headerFilter(new CustomHeaderFilter())
    .bodyFilter(new CustomBodyFilter())
    .requestFilter(new CustomRequestFilter())
    .responseFilter(new CustomResponseFilter())
    .sink(new DefaultSink(
            new CustomHttpLogFormatter(),
            new CustomHttpLogWriter()
    ))
    .build();
```



### 策略（Strategy）

Logbook使用一个非常硬性的策略来执行请求/响应日志记录：

- 请求/响应分开记录
- 请求/响应尽快记录
- 请求/响应一起记录或不记录（即没有部分流量记录）

其中一些限制可以通过自定义HttpLogWriter实现来缓解，但都不是理想。从2.0版本开始，Logbook引入了一个新的策略模式为核心，它内置了部分策略：

- BodyOnlyIfStatusAtLeastStrategy
- StatusAtLeastStrategy
- WithoutBodyStrategy



### 阶段（Phases）

Logbook工作在几个不同的阶段：

1. 条件（Conditional）
2. 过滤（Filtering）
3. 格式化（Formatting）
4. 记录（Writing）

每个阶段都由一个或多个可以自定义的接口完成。每个阶段都有一个合理的默认值。



#### 条件（Conditional）

记录HTTP消息并且包含其body代价是非常大的，所以禁用某些请求的日志记录非常有意义。常见情景就是忽略一些不必要的请求，比如SpringBoot的Actuator端点。定义一个条件非常简单，只需要编写一个Predicate来决定请求是否需要记录。当然，你也可以组合预定义的 Predicate：

> 注意关键类：org.zalando.logbook.Conditions、org.zalando.logbook.HttpRequest

```java
import static org.zalando.logbook.Conditions.*;

@Bean
public Logbook logbook() {
    return Logbook.builder()
        .condition(exclude(
            requestTo("/health"),
            requestTo("/admin/**"),
            contentType("application/octet-stream"),
            header("X-Secret", Stream.of("1", "X-Secret").collect(Collectors.toSet())::contains)))
        .build();
}
```

```java
package com.xyz.config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.zalando.logbook.HttpRequest;
import java.util.function.Predicate;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import static org.zalando.logbook.Conditions.*;

@Configuration
public class LogbookConfig {
    @Bean
    public Predicate<HttpRequest> requestCondition() {
        return exclude(
            requestTo("/health"),
            requestTo("/admin/**"),
            contentType("application/octet-stream"),
            header("X-Secret", Stream.of("1", "X-Secret").collect(Collectors.toSet())::contains));
    }
}
```

对于路径的包含和排除也可以通过设置 logbook.include和 logbook.exclude属性实现。并且能使用通配符：例如 /admin/** 松散地遵循 Ant 的路径模式风格，而不考虑 URL 的查询字符串。



#### 过滤（Filtering）

过滤的目的是防止记录HTTP请求和响应的某些敏感数据。通常包括Authorization请求头，但也可以用于某些明文查询或表单参数，如access_token和password

Logbook支持不同类型的过滤器：

| 类型           | 作用于                | 适用于           | 默认值                                                       |
| -------------- | --------------------- | ---------------- | ------------------------------------------------------------ |
| QueryFilter    | 请求参数              | request          | access_token                                                 |
| PathFilter     | 路径                  | request          |                                                              |
| HeaderFilter   | 请求头                | request/response | Authorization                                                |
| BodyFilter     | Content-Type and body | request/response | json格式：access_token和refresh_token，form表单：client_secret和password |
| RequestFilter  | HttpRequest           | request          | 替换二进制、文件上传和流                                     |
| ResponseFilter | HttpResponse          | response         | 替换二进制、文件上传和流                                     |

QueryFilter， PathFilter， HeaderFilter 和 BodyFilter能够满足绝大多数情况下的需求，对于更复杂的需求，可以使用 RequestFilter 和 ResponseFilter（与ForwardingHttpRequest、ForwardingHttpResponse结合使用）

```java
import static org.zalando.logbook.HeaderFilters.authorization;
import static org.zalando.logbook.HeaderFilters.eachHeader;
import static org.zalando.logbook.QueryFilters.accessToken;
import static org.zalando.logbook.QueryFilters.replaceQuery;

Logbook logbook = Logbook.builder()
    .requestFilter(RequestFilters.replaceBody(message -> contentType("audio/*").test(message) ? "mmh mmh mmh" : null))
    .responseFilter(ResponseFilters.replaceBody(message -> contentType("*/*-stream").test(message) ? "keeps going" : null))
    .queryFilter(accessToken())
    .queryFilter(replaceQuery("password", "<secret>"))
    .headerFilter(authorization()) 
    .headerFilter(eachHeader("X-Secret"::equalsIgnoreCase, "<secret>"))
    .build();
```

```java
package com.xyz.config;

import org.springframework.context.annotation.Bean;
import org.zalando.logbook.*;
import org.zalando.logbook.json.JacksonJsonFieldBodyFilter;
import org.zalando.logbook.json.JsonBodyFilters;
import java.util.Collections;

public class LogbookConfig {

    @Bean
    public QueryFilter queryFilter() {
        // 当前Query中的参数access_token字段被混淆成了***
        return QueryFilters.replaceQuery("access_token", "***");
    }

    @Bean
    public PathFilter pathFilter() {
        // 当前Path中的参数userId字段被混淆成了***
        return PathFilters.replace("userId", "***");
    }

    @Bean
    public HeaderFilter headerFilter() {
        // 替换单个字段
        HeaderFilters.replaceHeaders("Authorization", "***");
        HeaderFilter.merge(HeaderFilters.defaultValue(),HeaderFilters.replaceHeaders("filed","***"));
        // 替换多个字段,用Set集合
        HeaderFilter.merge(HeaderFilters.defaultValue(),HeaderFilters.replaceHeaders(Collections.EMPTY_SET,"***"));
        return HeaderFilters.replaceHeaders(Collections.singleton("secret"), "***");
    }

    @Bean
    public BodyFilter bodyFilter() {
        // 方式一
        BodyFilter.merge(BodyFilters.defaultValue(), new JacksonJsonFieldBodyFilter(Collections.EMPTY_SET, "***"));
        // 方式二
        return BodyFilter.merge(BodyFilters.defaultValue(),
                JsonBodyFilters.replaceJsonStringProperty(Collections.singleton("secret"), "***"));
    }

    @Bean
    public RequestFilter requestFilter () {
        return RequestFilters.replaceBody(
                BodyReplacers.replaceBody(Conditions.contentType("audio/*"), "<audio>"));
    }

    @Bean
    public ResponseFilter responseFilter () {
        return ResponseFilters.replaceBody(
                BodyReplacers.replaceBody(Conditions.contentType("*/*-stream"), "<stream>"));
    }
}
```

您可以根据需要配置任意数量的过滤器 - 它们将连续运行。

***

JsonPath body 过滤（**实验**），您可以将 JSONPath 过滤应用于 JSON 正文。 这里有些例子：

```java
import static org.zalando.logbook.json.JsonPathBodyFilters.jsonPath;
import static java.util.regex.Pattern.compile;

Logbook logbook = Logbook.builder()
    .bodyFilter(jsonPath("$.password").delete())
    .bodyFilter(jsonPath("$.active").replace("unknown"))
    .bodyFilter(jsonPath("$.address").replace("X"))
    .bodyFilter(jsonPath("$.name").replace(compile("^(\\w).+"), "$1."))
    .bodyFilter(jsonPath("$.friends.*.name").replace(compile("^(\\w).+"), "$1."))
    .bodyFilter(jsonPath("$.grades.*").replace(1.0))
    .build();
```

在应用过滤之前和之后：

```json
{
    "id": 1,
    "name": "Alice",
    "password": "s3cr3t",
    "active": true,
    "address": "Anhalter Straße 17 13, 67278 Bockenheim an der Weinstraße",
    "friends": [
        {
            "id": 2,
            "name": "Bob"
        },
        {
            "id": 3,
            "name": "Charlie"
        }
    ],
    "grades": {
        "Math": 1.0,
        "English": 2.2,
        "Science": 1.9,
        "PE": 4.0
    }
}
```

```json
{
    "id": 1,
    "name": "Alice",
    "active": "unknown",
    "address": "XXX",
    "friends": [
        {
            "id": 2,
            "name": "B."
        },
        {
            "id": 3,
            "name": "C."
        }
    ],
    "grades": {
        "Math": 1.0,
        "English": 1.0,
        "Science": 1.0,
        "PE": 1.0
    }
}
```



#### 格式化（Formatting）

格式化是把请求和响应转换为字符串。格式化不会指定请求和响应的记录位置，这是由Writer完成的。Logbook有两种默认格式化：HTTP 和 JSON

##### HTTP

HTTP 是默认的格式化样式，由DefaultHttpLogFormatter提供。 它主要用于本地开发和调试，而不是用于生产用途。 这是因为它不像JSON那样易于读取SpringBoot只需要配置：`logbook.format.style=http`，如下是 Request 和 Response

```bash
Incoming Request: 2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b
GET http://example.org/test HTTP/1.1
Accept: application/json
Host: localhost
Content-Type: text/plain

Hello world!
```

```bash
Outgoing Response: 2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b
Duration: 25 ms
HTTP/1.1 200
Content-Type: application/json

{"value":"Hello world!"}
```



##### JSON

JSON 是另一种格式样式，由 JsonHttpLogFormatter 提供。 与 HTTP 不同，它主要是为生产使用而设计的——解析器和日志消费者可以轻松地使用它。SpringBoot只需要配置：`logbook.format.style=json`（默认也是JSON），如下是 Request 和 Response

```json
{
    "origin":"remote",
    "type":"request",
    "correlation":"2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b",
    "protocol":"HTTP/1.1",
    "sender":"127.0.0.1",
    "method":"GET",
    "path":"http://example.org/test",
    "headers":{
        "Accept":[
            "application/json"
        ],
        "Content-Type":[
            "text/plain"
        ]
    },
    "body":"Hello world!"
}
```

```json
{
    "origin":"local",
    "type":"response",
    "correlation":"2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b",
    "duration":25,
    "protocol":"HTTP/1.1",
    "status":200,
    "headers":{
        "Content-Type":[
            "text/plain"
        ]
    },
    "body":"Hello world!"
}
```



##### Common Log Format

通用日志格式 (CLF) 是 Web 服务器在生成服务器日志文件时使用的标准化文本文件格式。 通过 CommonsLogFormatSink 支持该格式：

```http
185.85.220.253 - - [02/Aug/2019:08:16:41 0000] "GET /search?q=zalando HTTP/1.1" 200 -
```



##### cURL

cURL 是另一种格式样式，由 CurlHttpLogFormatter 提供，它将请求呈现为可执行的 cURL 命令。 与 JSON 不同，它主要是为人类设计的。SpringBoot只需要配置：`logbook.format.style=curl`，如下是 Request 和 Response

```bash
curl -v -X GET 'http://localhost/test' -H 'Accept: application/json'
```

```bash
Outgoing Response: 2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b
Duration: 25 ms
HTTP/1.1 200
Content-Type: application/json

{"value":"Hello world!"}
```



##### Splunk

Splunk 是另一种格式样式，由 SplunkHttpLogFormatter 提供，它将请求和响应呈现为键值对。SpringBoot只需要配置：`logbook.format.style=splunk`，如下是 Request 和 Response

```bash
origin=remote type=request correlation=2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b protocol=HTTP/1.1 sender=127.0.0.1 method=POST uri=http://example.org/test host=example.org scheme=http port=null path=/test headers={Accept=[application/json], Content-Type=[text/plain]} body=Hello world!
```

```bash
origin=local type=response correlation=2d66e4bc-9a0d-11e5-a84c-1f39510f0d6b duration=25 protocol=HTTP/1.1 status=200 headers={Content-Type=[text/plain]} body=Hello world!
```



#### 记录（Writer）

Writer定义了格式化后的请求和响应写入的位置。Logbook内置了三种实现：Logger、Stream、Chunking

##### Logger

默认情况下，使用 org.zalando.logbook.Logbook 类别和日志级别 trace 的 slf4j 记录器记录请求和响应。 也可以自定义：

默认情况下，请求和响应使用了slf4j来进行日志记录，日志的级别为 trace。也可以自定义：

```java
Logbook logbook = Logbook.builder()
    .sink(new DefaultSink(
            new DefaultHttpLogFormatter(),
            new DefaultHttpLogWriter()
    ))
    .build();
```



##### Stream

另一种实现是将请求和响应记录到 PrintStream，例如 System.out 或 System.err。 在生产环境中这是一个糟糕的选择，但有时对短期本地开发和/或调查很有用

```java
Logbook logbook = Logbook.builder()
    .sink(new DefaultSink(
            new DefaultHttpLogFormatter(),
            new StreamHttpLogWriter(System.err)
    ))
    .build();
```



##### Chunking

ChunkingSink 会把长的消息分割成较小的块，并且会委托给另一个sink将它们写入，只需要设置 `logbook.write.chunk-size`属性即可

```java
Logbook logbook = Logbook.builder()
    .sink(new ChunkingSink(sink, 1000))
    .build();
```



#### 关联ID（Correlation）

> 在SpringCloud应用中一般会集成Zipkin进行链路追踪，此时可以使用TraceId来关联请求和响应日志记录。

Logbook使用一个id来关联请求和响应，因为请求和响应通常位于日志文件中的不同位置（默认ID是16位随机字符组成）如默认不满足可以自定义实现：

```java
@Bean
public org.zalando.logbook.CorrelationId correlationId () {
    return request -> UUID.randomUUID().toString();
}

Logbook logbook = Logbook.builder().correlationId(new CustomCorrelationId()).build();
```



#### Sink

HttpLogFormatter 和 HttpLogWriter 的组合能够适用于大部分场合，但也有一些局限性。 实现 Sink 接口允许更复杂的用例，例如把请求和响应持久化到数据库。你可以使用 CompositeSink 将多个Sink合并为一个。



## 其他框架支持

### Servlet

在Servlet环境中，Logbook是通过 LogbookFilter来实现的。默认情况下，对于`application/x-www-form-urlencoded`请求会同等对待，即你会在日志中看到请求body。这种方法的缺点是下游代码将无法使用任何 HttpServletRequest.getParameter*(..)方法。

从Logbook 1.5.0开始，可使用 logbook.servlet.form-request系统属性（System Property）指定三种策略之一，这些策略定义Logbook如何处理这种情况（可以从源码中查看：（org.zalando.logbook.servlet.FormRequestMode）

| 属性值       | 优点                                             | 缺点                             |
| ------------ | ------------------------------------------------ | -------------------------------- |
| body（默认） | body会被记录                                     | 下游代码不能使用getParameter()   |
| parameter    | body会被记录                                     | 下游代码不能使用getInputStream() |
| off          | 下游代码可以使用getParameter()或getInputStream() | body不会被记录                   |

Logbook默认还提供了对：Servlet、HTTP Client、JAX-RS、Netty、OkHttp v2.x、OkHttp v3.x的支持，具体使用方法可以参考官方文档



# Java 实现数据脱敏

- Java注解实现数据脱敏-与主业务解耦且权限管控关联【小兀军】https://mp.weixin.qq.com/s/RujXqk0MWhmtQKpf_zaTnQ
- Springboot日志、配置文件、接口数据如何脱敏？老鸟们都是这样玩的【架构师社区】https://mp.weixin.qq.com/s/uKlit0Cu8ZhqM_1I1_N-9Q
