> 1. 1万七千字精讲，JDK8 的 Lambda、Stream、LocalDate 骚技能：https://mp.weixin.qq.com/s/xuvifX3Rh9aq4Z-vQuCEEQ
> 4. Java8 Stream 详情总结：https://blog.csdn.net/winterking3/article/details/87163993

> - Java Developers：https://dev.java/
> - JDK Releases：https://www.java.com/releases/
> - Java 中文网：https://java.p2hp.com/

# Java 版本发行历史

- 1995年5月23日，Java语言诞生
- 1996年1月，第一个JDK-JDK1.0诞生
- 1996年4月，10个最主要的操作系统供应商申明将在其产品中嵌入JAVA技术
- 1996年9月，约8.3万个网页应用了JAVA技术来制作
- 1997年2月18日，JDK1.1发布
- 1997年4月2日，JavaOne会议召开，参与者逾一万人，创当时全球同类会议规模之纪录
- 1997年9月，JavaDeveloperConnection社区成员超过十万
- 1998年2月，JDK1.1被下载超过2,000,000次
- 1998年12月8日，JAVA2企业平台J2EE发布
- 1999年6月，SUN公司发布Java的三个版本：标准版、企业版和微型版（J2SE、J2EE、J2ME）
- 2000年5月8日，JDK1.3发布
- 2000年5月29日，JDK1.4发布
- 2001年6月5日，NOKIA宣布，到2003年将出售1亿部支持Java的手机
- 2001年9月24日，J2EE1.3发布
- 2002年2月13日，J2SE1.4发布，自此Java的计算能力有了大幅提升。
- 2004年9月30日18:00PM，J2SE1.5发布，是Java语言的发展史上的又一里程碑事件。为了表示这个版本的重要性，J2SE1.5更名为J2SE5.0。
- 2005年6月，JavaOne大会召开，SUN公司公开Java SE 6（没有后缀.0）。此时，Java的各种版本已经更名以取消其中的数字“2”：J2EE更名为Java EE, J2SE更名为Java SE，J2ME更名为Java ME。
- 2006年11月13日，SUN公司宣布Java全线采纳GNU General Public License Version 2，从而公开了Java的源代码。
- 2011年Oracle发布了Java的一个新版本，Java 7，其中只做了一些简单的改进。
- 2014年，Java8 发布，Java8 版本有了很大的改变。Java8 提供了一种“函数式”编程方式，可以很容易地表述并发执行的计算。
- 2017 年 9 月开始（Java 9 开始），Java 的功能更新周期从之前的每两年一个新版本缩减到每六个月一个新版本。并且每6个版本会出来一个长期支持的版本，支持3年，普通版本只支持6个月。
- .......



# Java 各版本发行时间表

| 版本              | 名称                                | 发行日期   |
| ----------------- | ----------------------------------- | ---------- |
| JDK Beta          | none（无）                          | 1995-05-23 |
| JDK 1.0           | Oak（橡树）                         | 1996-01-23 |
| JDK 1.1           | none（无）                          | 1997-02-19 |
| JDK 1.1.4         | Sparkler（宝石）                    | 1997-09-12 |
| JDK 1.1.5         | Pumpkin（南瓜）                     | 1997-12-13 |
| JDK 1.1.6         | Abigail（阿比盖尔–女子名）          | 1998-04-24 |
| JDK 1.1.7         | Brutus（布鲁图–古罗马政治家和将军） | 1998-09-28 |
| JDK 1.1.8         | Chelsea（切尔西–城市名）            | 1999-04-08 |
| J2SE 1.2          | Playground（运动场）                | 1998-12-04 |
| J2SE 1.2.1        | none（无）                          | 1999-03-30 |
| J2SE 1.2.2        | Cricket（蟋蟀）                     | 1999-07-08 |
| J2SE 1.3          | Kestrel（美洲红隼）                 | 2000-05-08 |
| J2SE 1.3.1        | Ladybird（瓢虫）                    | 2001-05-17 |
| J2SE 1.4.0        | Merlin（灰背隼）                    | 2002-02-13 |
| J2SE 1.4.1        | grasshopper（蚱蜢）                 | 2002-09-16 |
| J2SE 1.4.2        | Mantis（螳螂）                      | 2003-06-26 |
| J2SE 5.0          | Tiger（老虎）                       | 2004-09-30 |
| Java SE 6         | Mustang（野马）                     | 2006-12-11 |
| Java SE 7         | Dolphin（海豚）                     | 2011-07-28 |
| Java SE 8（LTS）  | Spider（蜘蛛）                      | 2014-03-18 |
| Java SE 9         | 从 Java SE 9 开始已经没有版本号了   | 2017-09-21 |
| Java SE 10        |                                     | 2018-03-21 |
| Java SE 11（LTS） |                                     | 2018-09-25 |
| Java SE 12        |                                     | 2019-03-19 |
| Java SE 13        |                                     | 2019-09-17 |
| Java SE 14        |                                     | 2020-03-17 |
| Java SE 15        |                                     | 2020-09-15 |
| Java SE 16        |                                     | 2021-03-16 |
| Java SE 17（LTS） |                                     | 2021-09-14 |
| Java SE 18        |                                     | 2022-03-20 |
| Java SE 19        |                                     | 2022-09-20 |
| Java SE 20        |                                     | 2023-03-21 |
| Java SE 21（LTS） |                                     |            |

从上方的 JDK 各个版本发布时间和版本名称表可以看到，Java 大体有三大类命名方式：JDK、J2SE、Java SE。

从上表还可以发现，就是JDK的每一个版本号都使用一个开发代号表示（就是表中的中文名）。而且从JDK1.2.2 开始,主要版本（如1.3，1.4，5.0）都是以鸟类或哺乳动物来命名的。而它们的BUG修正版本（如1.2.2，1.3.1，1.4.2）都是以昆虫命名的。

***

从2017年9月开始（Java SE 9 开始），Java的功能更新周期从之前的每两年一个新版本缩减到每六个月一个新版本。

| 年份 | 3月  | 9月        |
| ---- | ---- | ---------- |
| 2017 |      | 9          |
| 2018 | 10   | 11（LTS）  |
| 2019 | 12   | 13         |
| 2020 | 14   | 15         |
| 2021 | 16   | 17（LTS）  |
| 2022 | 18   | 19         |
| 2023 | 20   | 21         |
| 2024 | 22   | 23 （LTS） |

并且每6个版本会出来一个长期支持的版本，支持3年，普通版本只支持6个月。

以上说的都是 OracleJDK 的许可标准，OracleJDK 之后开始收费。

但是每一个版本发布的时候会发布 OpenJDK 跟 OracleJDK 几乎一样。OpenJDK 商用免费，Oralce 只维护半年，之后给 RedHat 和 IBM 来长期维护，这两家想扩大在Java圈的影响力。

所以 Java 收费算是个标题党。Oracle 的目的是想让商业用户跟随版本进行升级。Oracle 可以将精力投入到新的版本研发上。

关于一场丧心病狂的收购案：

- Oracle 于 2009年 在市值 2000 亿的 Sun 公司中用 74 亿进行了收购。
- 收购之后就开始告谷歌的安卓 Java 侵权。
- 并于 2018 年 3 月 27 日，法庭宣布谷歌赔偿 Oracle 合计 88 亿美元。

***

详细可以参考 OracleJDK 及 OpenJDK 官方文档：

- JDK Project：https://openjdk.org/projects/jdk/
- JDK Release Notes：https://www.oracle.com/java/technologies/javase/jdk-relnotes-index.html

OracleJDK 及 OpenJDK 官方下载地址：

- OpenJDK ：https://jdk.java.net/archive/



# Java 1.0【1996-01-23】

> 初代版本，伟大的一个里程碑，但是是纯解释运行，使用外挂JIT，性能比较差，运行速度慢。

1996-01-23 发布，版本代号为 Oak（橡树）

- 【新增】JVM；
- 【新增】Applet: 已被淘汰；
- 【新增】AWT(Abstract Window ToolKit): 已被淘汰；
- 【新增】File 和 IO(InputStream/OutputStream) 相关操作 API；



# Java 1.1【1997-02-19】

1997-02-19 发布, 无版本代号, major.minor 版本为 45

- 【新增】JDBC(Java Database Connectivity)， 注意不是 Connection；
- 【新增】内部类；
- 【新增】Java Bean；
- 【新增】RMI(Remote Method Invocation)；
- 【新增】反射(仅用于内省)；
- 【提升】IO 引入 Reader/Writer 及其子类；



# Java 1.2【1998-12-04】

1998-12-04 发布, 版本代号为 Playground（操场）， major.minor 版本为 46

- 【新增】集合框架；
- 【新增】对字符串常量做内存映射；
- 【新增】JIT(Just In Time) 编译器；
- 【新增】对打包的 Java 文件进行数字签名；
- 【新增】控制授权访问系统资源的策略工具；
- 【新增】JFC(Java Foundation Classes)， 包括 Swing 1.0， 拖放和 Java2D 类库，很少使用， 主要的使用场景是用在后端生成图片的业务场景， 如二维码生成；
- 【新增】Java 插件；
- 【新增】strictfp 关键字；
- 【提升】在 JDBC 中引入可滚动结果集， BLOB， CLOB， 批量更新和用户自定义类型；
- 【提升】在 Applet 中添加声音支持， 已被淘汰；

J2SE 1.2 版本到 J2SE 5.0 的后续版本被更名为 Java 2，而版本名称“J2SE”（Java 2平台，标准版）取代JDK以区别J2EE（Java 2平台，企业版） 和J2ME（Java 2 Platform，Micro Edition）基础平台。



# Java 1.3【2000-05-08】

2000-05-08 发布, 版本代号为 Kestrel（红隼）， major.minor 版本为 47

- 【新增】Java Sound API， 已被淘汰；
- 【新增】jar文件索引；
- 【新增】JVM 配备 HotSpot JVM；
- 【新增】代理类；
- 【新增】Java 命名与目录接口；
- 【新增】Java 平台调试体系；



# Java 1.4【2002-02-13】

2002-02-13 发布（首次在 JCP 下发行）, 版本代号为 Merlin（隼）， major.minor 版本为 48

- 【新增】XML处理;
- 【新增】Java打印服务;
- 【新增】Logging API;
- 【新增】Java Web Start;
- 【新增】断言;
- 【新增】引入Preferences API;
- 【新增】链式异常处理;
- 【新增】支持IPV6;
- 【新增】正则表达式;
- 【新增】Image I/O API；
- 【新增】NIO API；
- 【新增】集成 JCE、JSSE、JAAS；
- 【提升】引入JDBC 3.0 API;



# Java 5【2004-09-30】

2004-09-30 发布, 版本代号为 Tiger（老虎）， major.minor 版本为 49。从 JDK5 开始， JDK 的版本不再以 1.x 的方式来命名了， 而是直接用 x 来命名。

- 【新增】泛型
- 【新增】增强 for 循环， 可以使用迭代方式；
- 【新增】自动装箱与自动拆箱；
- 【新增】类型安全的枚举；
- 【新增】支持可变参数；
- 【新增】静态导入；
- 【新增】注解： 动态注解、元数据；
- 【新增】Instrumentation；
- 【新增】内省(Introspector)
- 【新增】JUC 包
- 【新增】Scanner 类

详细可以参考 JDK 5：

- https://www.oracle.com/java/technologies/javase/j2se-v50.html

J2SE 1.2 版本到 J2SE 5.0 的后续版本被更名为 Java 2，而版本名称“J2SE”（Java 2平台，标准版）取代JDK以区别J2EE（Java 2平台，企业版） 和J2ME（Java 2 Platform，Micro Edition）基础平台。

这个在2004年9月30日发布的版本原本以1.5编号，也就是仍然使用内部版本编号。这个数字辈改变是为了“更好地反映成熟度、稳定性、可扩展性和 J2SE 的安全水准”。

此版本推出了JAVA语言的一个新的版本控制系统，而旧版本控制系统仍然可以在开发者资源库中继续使用。即Java 5 = JDK1.5，一次类推。



# Java 6【2006-12-11】

2006-12-11 发布, 版本代号为 Mustang（野马）， major.minor 版本为 50

- 【新增】Web 服务元数据；
- 【新增】脚本语言支持；
- 【新增】JTable 的排序和过滤；
- 【新增】轻量级的 Http Server；
- 【新增】插入式注解处理 API(Pluggable Annotation Processing API)；
- 【新增】支持嵌入式数据库 Derby；
- 【新增】Console API；
- 【新增】Compile API；
- 【新增】StAX(Streaming API for XML) 处理 XML；
- 【提升】引入 JAXB2 来处理对象和 XML 之间的映射；
- 【提升】AWT 中新增了两个类 Desktop 和 SystemTray， 极不常用；
- 【提升】Common Annotations；
- 【提升】JAX-WS2.0；
- 【提升】JDBC4.0；
- 【提升】引入新的 GC 算法；

详细可以参考 JDK 6：

- https://openjdk.org/projects/jdk6/
- https://www.oracle.com/java/technologies/javase/6-relnotes.html

2006年12月11日，Sun把原本的名称“J2SE”改为“Java SE”，然后再从版本号中去掉“.0”，而开发者内部编号仍然是1.6.0，并且一直持续用到1.8.0。在Java SE 9 得到废弃。



# Java 7【2011-07-28】

2011-07-28 发布, 版本代号为 Dolphin（海豚）， major.minor 版本为 51

- 【新增】Fork and Join
- 【提升】创建泛型对象时应用类型推导， 钻石表达式；
- 【提升】自动资源管理（try-with-resources）；
- 【提升】异常捕获的处理方式（通过 | 捕获多个异常），
- 【提升】Java NIO2 API(working with path 和 file change notification)；
- 【提升】switch 的分支条件支持字符串；
- 【提升】集合中新增 TransferQueue 接口， 是 BlockingQueue 的改进版，实现类为 LinkedTransferQueue；
- 【提升】JDBC4.1: try-with-resources 和 RowSet1.1
- 【提升】网络、Swing、XML 处理、国际化等 API 的提升
- 【提升】JVM方面， 支持非 Java 语言， Garbage-First-Collector 和提升了 Java HotSpot 虚拟机的性能；
- 【提升】数值类型可以用二进制字符串表示,并且可以在字符串表示中添加下划线;
- 【提升】null 值得自动处理;
- 【提升】支持动态语言;

详细可以参考 JDK 7：

- https://openjdk.org/projects/jdk7/
- https://www.oracle.com/java/technologies/javase/7u-relnotes.html

# Java 8【2014-03-18】【LTS】

2014-03-18 发布, 版本代号为 Spider（蜘蛛）， major.minor 版本为 52

- 【新增】Lambda 表达式；
- 【新增】管道和 Stream；
- 【新增】新的日期和时间 API；
- 【新增】函数式接口；
- 【新增】接口的默认方法，又称为扩展方法；
- 【新增】方法与构造函数的引用；
- 【新增】Optional API；
- 【新增】并行操作；
- 【新增】新工具，如 Nashorn 引擎 jjs、类依赖分析器 jdeps，它允许我们在JVM上运行特定的javascript应用；
- 【提升】支持多重注解，并新增了部分注解；

详细参考：[Java 8 新特性 | 菜鸟教程](http://www.runoob.com/java/java8-new-features.html)

详细可以参考 JDK 8：

- http://openjdk.java.net/projects/jdk8/
- https://www.oracle.com/java/technologies/javase/8u-relnotes.html

Java 8 是一个长期支持版本（LTS），增加了许多阶段性的新特性，改变了编程方式。可以说是JDK升级的分水岭。

# Java 9【2017-09-21】

2017-09-21 发布, 无版本代号（从Java 9 开始就没有版本号了，后续版本也不提示版本号）

- 【新增】模块化（模块是一个包的容器），进而使得 JDK 目录结构发生变化；
- 【新增】交互式编程环境 REPL(JShell)；
- 【新增】轻量级 JSON API；
- 【新增】响应式流 (Reactive Streams) API
- 【新增】HTTP 2.0 客户端，新的 HTTPClient API 支持 WebSocket 和 HTTP2 流以及服务器推送特性；
- 【新增】多版本兼容 jar 包；
- 【新增】货币相关的 API；
- 【新增】代码分段缓存；
- 【新增】智能 Java 编译，第二阶段；
- 【新增】多分辨率图像 API：定义多分辨率图像API，开发者可以很容易的操作和展示不同分辨率的图像了；
- 【提升】集合： 提供集合工厂方法，List，Set 和 Map 接口中，新的静态工厂方法可以创建这些集合的不可变实例；
- 【提升】接口： 私有接口方法，在接口中使用private私有方法。我们可以使用 private 访问修饰符在接口中编写私有方法；
- 【提升】String： 底层存储结构更换；
- 【提升】锁争用机制；
- 【提升】简化进程 API；
- 【提升】Javadoc 的提升，现在支持在 API 文档中的进行搜索。另外，Javadoc 的输出现在符合兼容 HTML5 标准。
- 【提升】改进 CompletableFuture API ： CompletableFuture 类的异步机制可以在 ProcessHandle.onExit 方法退出时执行操作。
- 【提升】改进 Optional 类：java.util.Optional 添加了很多新的有用方法，Optional 可以直接转为 stream。
- 【提升】改进钻石操作符(Diamond Operator) ：匿名类可以使用钻石操作符(Diamond Operator)；
- 【提升】API：进程、Stream、@Deprecated、 Optional、CompletableFuture 等；
- 【提升】改进 try-with-resources：如果你已经有一个资源是 final 或等效于 final 变量,您可以在 try-with-resources 语句中使用该变量，而无需在 try-with-resources 语句中声明一个新变量。

详细参考：[Java 9 新特性 | 菜鸟教程](http://www.runoob.com/java/java9-new-features.html)

详细可以参考 JDK 9：

- http://openjdk.java.net/projects/jdk9
- https://www.oracle.com/java/technologies/javase/9u-relnotes.html

从 2017 年 9 月开始（Java 9 开始），Java 的功能更新周期从之前的每两年一个新版本缩减到每六个月一个新版本。

# Java 10【2018-03-20】

 2018-03-20 发布。

- 【新增】286: 局部变量类型推断。
- 【提升】296: 整合 JDK 代码仓库；
- 【新增】304: 统一的垃圾回收接口。
- 【新增】307: G1 垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟。
- 【新增】310: 应用程序类数据共享。
- 【新增】312: 线程-局部管控。
- 【删除】313: 移除 Native-Header 自动生成工具
- 【提升】314: 额外的 Unicode 语言标签扩展
- 【提升】316: 备用存储装置上的堆分配
- 【新增】317: 基于 Java 的 实验性 JIT 编译器
- 【提升】319: 根证书认证
- 【提升】322: 基于时间的版本发布模式

详细可以参考 JDK 10：

- http://openjdk.java.net/projects/jdk/10/
- https://www.oracle.com/java/technologies/javase/10-relnote-issues.html



# Java 11【2018-09-25】【LTS】

2018-09-25 发布。

- 【新增】181: Nest-Based Access Control（基于嵌套的访问控制）
- 【提升】309: Dynamic Class-File Constants（动态的类文件常量）
- 【提升】315: Improve Aarch64 Intrinsics（改进 Aarch64 Intrinsics）
- 【新增】318: Epsilon: A No-Op Garbage Collector（Epsilon 垃圾回收器，又被称为"No-Op（无操作）"回收器）
- 【移除】320: Remove the Java EE and CORBA Modules（移除 Java EE 和 CORBA 模块，JavaFX 也已被移除）
- 【提升】321: HTTP Client (Standard)
- 【提升】323: Local-Variable Syntax for Lambda Parameters（用于 Lambda 参数的局部变量语法）
- 【提升】324: Key Agreement with Curve25519 and Curve448（采用 Curve25519 和 Curve448 算法实现的密钥协议）
- 【提升】327: Unicode 10
- 【提升】328: Flight Recorder（飞行记录仪）
- 【提升】329: ChaCha20 and Poly1305 Cryptographic Algorithms（实现 ChaCha20 和 Poly1305 加密算法）
- 【提升】330: Launch Single-File Source-Code Programs（启动单个 Java 源代码文件的程序）
- 【新增】331: Low-Overhead Heap Profiling（低开销的堆分配采样方法）
- 【提升】332: Transport Layer Security (TLS) 1.3（对 TLS 1.3 的支持）
- 【新增】333: ZGC: A Scalable Low-Latency Garbage Collector (Experimental)（ZGC：可伸缩的低延迟垃圾回收器，处于实验性阶段）
- 【废弃】335: Deprecate the Nashorn JavaScript Engine（弃用 Nashorn JavaScript 引擎）
- 【废弃】336: Deprecate the Pack200 Tools and API（弃用 Pack200 工具及其 API）

详细可以参考 JDK 11：

- http://openjdk.java.net/projects/jdk/11/
- https://www.oracle.com/java/technologies/javase/11-relnote-issues.html



# Java 12【2019-03-19】

2019-03-19 发布。

作为“功能性版本”，JDK 12 总共包含 8 个新的 JEP ，分别为：

- 【新增】189: 新增一个名为 Shenandoah 的垃圾回收器，它通过在 Java 线程运行的同时进行疏散 (evacuation) 工作来减少停顿时间（实验）

- 【新增】230: 新增一套微基准测试，使开发者能够基于现有的 Java Microbenchmark Harness（JMH）轻松测试 JDK 的性能，并创建新的基准测试。

- 【提升】325: 对 Switch 语句进行扩展，使其可以用作语句或表达式，简化日常代码。
- 【新增】334: 引入一个 API 来对关键类文件 (key class-file) 和运行时工件的名义描述（nominal descriptions）进行建模，特别是那些可从常量池加载的常量。

- 【提升】340: 删除与 arm64 端口相关的所有源码，保留 32 位 ARM 移植和 64 位 aarch64 移植。

- 【提升】341: 默认生成类数据共享（CDS）存档。

- 【提升】344: 当 G1 垃圾回收器的回收超过暂停目标，则能中止垃圾回收过程。

- 【提升】346: 改进 G1 垃圾回收器，以便在空闲时自动将 Java 堆内存返回给操作系统。

 详细可以参考 JDK 12：

- http://openjdk.java.net/projects/jdk/12/
- https://www.oracle.com/java/technologies/javase/12-relnote-issues.html

# Java 13【2019-09-17】

2019-09-17 发布。

- 【提升】350: Dynamic CDS Archives：对appCDS进行性了扩展，允许在Java应用执行结束时动态归档类。归档类包括包括默认的基础层CDS(class data-sharing) 存档中不存在的所有已加载的应用程序类和类库。通过此仿瓷提高了AppCDS的可用性；
- 【提升】351: ZGC: Uncommit Unused Memory：对ZGC进行了增强，在以前的版本中，java GC之后并不会将系统内存释放给OS，因为每次释放都意味着重新调整jvm的内存大小，存在一定的消耗；随着软件的发展，我们发现在很多时候内存是比较昂贵的资源，所以将不用的内存释放回去给OS是非常有必要的；此功能在默认情况下已开始，但可以通过-xx:-zuncommit参数禁用；注意：如果最新内存参数设置比最大内存参数大，那么此功能将隐式禁用。
- 【提升】353: Reimplement the Legacy Socket API：在这个版本中，将使用新的实现来代替java.net.socket和java.net.serversocket API的底层实现。新版本中旧的API还未删除，可以通过配置系统属性"jdk.net.useplansocketimpl"来使用他们。但默认实现是最新版本的。
- 【提升】354: Switch Expressions (Preview)：扩展开关，以便它可以用作语句或表达式，并且两种形式都可以使用传统的情况…：标签（带有贯穿线）或新案例…->标签（没有掉进去），还有一个新的语句，用于从开关表达式中产生值。这些变化将简化日常编码，并为在交换机中使用模式匹配做好准备。这是jdk 13中的一个预览语言特性。
- 【提升】355: Text Blocks (Preview)：向Java语言添加文本块。文本块是一个多行字符串文本，它避免了大多数转义序列的需要，自动以可预测的方式格式化字符串，并在需要时让开发人员控制格式。这是jdk 13中的一个预览语言特性。

此外，JDK8 的截止时间为2019年1月份，到期后，Oracle 将不再提供补丁及其它的更新服务。官网称可能会更久，JDK9 的截止时间是2018年3月，JDK10的截止版本是2018年9月。（详细请前往：Oracle Java SE Support Roadmap），JDK 9 和 JDK 10 都是一个短期版本，故稳定长期的版本是JAVA 11（LTS - Long Term Support）版本。

详细可以参考 JDK 13：

- https://openjdk.org/projects/jdk/13/
- https://www.oracle.com/java/technologies/javase/13-relnote-issues.html

# Java 14【2020-03-17】

2020-03-17 发布。

- 305: Pattern Matching for instanceof (Preview)：instanceof 的模式匹配（预览）
- 343: Packaging Tool (Incubator)：打包工具（孵化）
- 345: NUMA-Aware Memory Allocation for G1：G1 的NUMA 内存分配优化
- 349: JFR Event Streaming：JFR事件流
- 352: Non-Volatile Mapped Byte Buffers：非原子性的字节缓冲区映射
- 358: Helpful NullPointerExceptions：非常有帮助的空指针异常
- 359: Records (Preview)：Records（预览）
- 361: Switch Expressions (Standard)：Switch 表达式（标准）
- 362: Deprecate the Solaris and SPARC Ports：弃用 Solaris 和S PARC 端口
- 363: Remove the Concurrent Mark Sweep (CMS) Garbage Collector：移除 CMS（Concurrent Mark Sweep）垃圾收集器
- 364: ZGC on macOS：macOS 系统上的 ZGC
- 365: ZGC on Windows：Windows 系统上的 ZGC
- 366: Deprecate the ParallelScavenge + SerialOld GC Combination：弃用 ParallelScavenge + SerialOld GC 组合
- 367: Remove the Pack200 Tools and API：移除Pack200 Tools和API
- 368: Text Blocks (Second Preview)：文本块（第二个预览版）
- 370: Foreign-Memory Access API (Incubator)：外部存储器API（孵化）

详细可以参考 JDK 14：

- https://openjdk.org/projects/jdk/14/
- https://www.oracle.com/java/technologies/javase/14-relnote-issues.html

# Java 15【2020-09-15】

2020-09-15 发布。

- 339: Edwards-Curve Digital Signature Algorithm (EdDSA)：蒙哥马利与扭曲爱德华曲线签名算法
- 360: Sealed Classes (Preview)：密封类(预览)
- 371: Hidden Classes：隐藏类
- 372: Remove the Nashorn JavaScript Engine：移除nasorn JavaScript引擎
- 373: Reimplement the Legacy DatagramSocket API：重新实现旧的DatagramSocket API
- 374: Disable and Deprecate Biased Locking：禁用和弃用偏置锁
- 375: Pattern Matching for instanceof (Second Preview)：模式匹配的instanceof(第二次预览)
- 377: ZGC: A Scalable Low-Latency Garbage Collector：ZGC:一个可扩展的低延迟垃圾收集器
- 378: Text Blocks：文本块
- 379: Shenandoah: A Low-Pause-Time Garbage Collector：Shenandoah:低暂停时间垃圾收集器
- 381: Remove the Solaris and SPARC Ports：移除Solaris和SPARC端口
- 383: Foreign-Memory Access API (Second Incubator)：外部内存访问API(第二个孵化器)
- 384: Records (Second Preview)：记录(第二次预览)
- 385: Deprecate RMI Activation for Removal：建议移除RMI激活

详细可以参考 JDK 15：

- https://openjdk.org/projects/jdk/15/
- https://www.oracle.com/java/technologies/javase/15-relnote-issues.html

# Java 16【2021-03-16】

2021-03-16 发布。

- 338: Vector API (Incubator)：Vector API (孵化器)
- 347: Enable C++14 Language Features：启用C++ 14种语言特性
- 357: Migrate from Mercurial to Git：从Mercurial迁移到Git
- 369: Migrate to GitHub：迁移到GitHub
- 376: ZGC: Concurrent Thread-Stack Processing：ZGC:并发线程堆栈处理
- 380: Unix-Domain Socket Channels：Unix-Domain 套接字通道
- 386: Alpine Linux Port：Alpine Linux端口
- 387: Elastic Metaspace：弹性Metaspace
- 388: Windows/AArch64 Port：Windows / AArch64端口
- 389: Foreign Linker API (Incubator)：国外连接器API(孵化器)
- 390: Warnings for Value-Based Classes：对基于值的类的警告
- 392: Packaging Tool：包装工具
- 393: Foreign-Memory Access API (Third Incubator)：外存储器访问API(第三孵化器)
- 394: Pattern Matching for instanceof：instanceof匹配模式
- 395: Records：记录转正
- 396: Strongly Encapsulate JDK Internals by Default：默认情况下对JDK内部进行强封装
- 397: Sealed Classes (Second Preview)：密封类(第二次预览)

详细可以参考 JDK 16：

- https://openjdk.org/projects/jdk/16/
- https://www.oracle.com/java/technologies/javase/16-relnote-issues.html

# Java 17【2021-09-14】【LTS】

2021-09-14 发布。

- 306: Restore Always-Strict Floating-Point Semantics：恢复总是严格的浮点语义
- 356: Enhanced Pseudo-Random Number Generators：增强型伪随机数发生器
- 382: New macOS Rendering Pipeline：使用新的 macOS 渲染管道
- 391: macOS/AArch64 Port：将 JDK 移植到 macOS/AArch64
- 398: Deprecate the Applet API for Removal：删除已启动的 Applet API
- 403: Strongly Encapsulate JDK Internals：更强的封装 JDK 内部构件
- 406: Pattern Matching for switch (Preview)：Switch 模式匹配（预览）
- 407: Remove RMI Activation：删除 RMI 激活
- 409: Sealed Classes：密封类（Sealed Classes）
- 410: Remove the Experimental AOT and JIT Compiler：移除实验性的 AOT 和 JIT 编译器
- 411: Deprecate the Security Manager for Removal：弃用 Security Manager
- 412: Foreign Function & Memory API (Incubator)：外部函数与内存 API (孵化器)
- 414: Vector API (Second Incubator)：Vector API（第二孵化器）
- 415: Context-Specific Deserialization Filters：指定上下文的反序列化过滤器

Java 17 目前已经进入 Rampdown Phase One 阶段，所有的功能特性都已经被冻结。这说明 Java 17的新特性已经定了，不会再增加新的 JEP （JDK增强建议）！

详细可以参考 JDK 17：

- https://openjdk.org/projects/jdk/17/
- https://www.oracle.com/java/technologies/javase/17-relnote-issues.html

# Java 18【2022-03-22】

2022-03-22 发布。

- 400: UTF-8 by Default：默认 UTF-8
- 408: Simple Web Server：简单的网络服务器
- 413: Code Snippets in Java API Documentation：Java API 文档中的代码片段
- 416: Reimplement Core Reflection with Method Handles：重构 Reflection，使用方法句柄重新实现核心反射
- 417: Vector API (Third Incubator)：计算向量的 API（三次孵化）
- 418: Internet-Address Resolution SPI：互联网地址解析 SPI
- 419: Foreign Function & Memory API (Second Incubator)：外部函数和内存 API（二次孵化）
- 420: Pattern Matching for switch (Second Preview)：Switch 模式匹配（二次预览）
- 421: Deprecate Finalization for Removal：弃用 finalize

详细可以参考 JDK 18：

- https://openjdk.org/projects/jdk/18/
- https://www.oracle.com/java/technologies/javase/18-relnote-issues.html

# Java 19【2022-09-20】

2022-09-20 发布。

- 405: Record Patterns (Preview)：记录模式
- 422: Linux/RISC-V Port：Linux/RISC-V 移植
- 424: Foreign Function & Memory API (Preview)：外部函数和内存 API
- 425: Virtual Threads (Preview)：虚拟线程
- 426: Vector API (Fourth Incubator)：向量 API
- 427: Pattern Matching for switch (Third Preview)：Switch 模式匹配
- 428: Structured Concurrency (Incubator)：结构化并发

详细可以参考 JDK 19：

- https://openjdk.org/projects/jdk/19/
- https://www.oracle.com/java/technologies/javase/19-relnote-issues.html

# Java 20【2023-03-21】

2023-03-21 发布。

- 429: Scoped Values (Incubator)：作用域值 Scoped Value（孵化器）
- 432: Record Patterns (Second Preview)：Record 模式（第二预览版）
- 433: Pattern Matching for switch (Fourth Preview)：switch 模式匹配（第四预览版）
- 434: Foreign Function & Memory API (Second Preview)：外部函数内存 API（第2版）
- 436: Virtual Threads (Second Preview)：虚拟线程（第二个预览版）
- 437: Structured Concurrency (Second Incubator)：结构化并发（第二个孵化器版本）
- 438: Vector API (Fifth Incubator)：Vector API（第五个孵化器版本）

详细可以参考 JDK 20：

- https://openjdk.org/projects/jdk/20/
- https://www.oracle.com/java/technologies/javase/20-relnote-issues.html



# 参考文献 & 鸣谢

1. 全网最全的JAVA所有版本特性【JAVA 1.0 - JAVA 20】https://blog.csdn.net/qq934235475/article/details/82220076
2. Java–Java版本和JDK版本「建议收藏」：https://cloud.tencent.com/developer/article/2128820
3. Java主流发布版本下载地址及各版本支持时间：https://blog.csdn.net/nalw2012/article/details/124146584
4. JDK各版本新特性：https://shendi.blog.csdn.net/article/details/108647463

