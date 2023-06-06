# Java 11 新特性

Java 11 在 2018 年 9 月 25 日正式发布，Java 11 是一个长期支持（LTS）版本，并且是 Java 8 之后的第一个 LTS 版本。

但是也自从 Java 11 开始， Oracle JDK 不再可以免费的用于商业用途，当然如果你是个人使用，或者是使用 OpenJDK ，那么还是可以免费使用的。Oracle 表示除非你在生产中使用，否则可以不收费。即便收费也可使用免费的 OpenJDK 。

这次更新共带来 17 个新特性：

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
- 【新增】333: ZGC: A Scalable Low-Latency Garbage Collector (Experimental)（ZGC：可伸缩的低延迟垃圾回收器，实验性阶段）
- 【废弃】335: Deprecate the Nashorn JavaScript Engine（弃用 Nashorn JavaScript 引擎）
- 【废弃】336: Deprecate the Pack200 Tools and API（弃用 Pack200 工具及其 API）

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 11 新特性：https://openjdk.org/projects/jdk/11/
- OracleJDK Java 11 新特性：https://www.oracle.com/java/technologies/javase/11-relnote-issues.html#NewFeature
- Oracle JDK 11 文档：https://docs.oracle.com/en/java/javase/11/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## 1、String API 增强

自 JDK9 和 JDK10 都为 Java 增加了许多的 API，如今 JDK11 又增加了许多字符串自带方法。

1、isBlank () 判空白、isEmpty () 判空。

```java
// 判空，blank里我放入了全角空格，半角空格，TAB，empty里放了空字符串
String blank = "　　  ";
String empty = "";
System.out.println(blank.isBlank()); // true
System.out.println(blank.isEmpty()); // false
System.out.println(empty.isBlank()); // true
System.out.println(empty.isEmpty()); // true
```

2、lines () 分割获取字符串流。

```java
// lines 返回一个 Stream
String line = "a\nb\nc";
Stream<String> lines = line.lines();
// 使用 lambda 遍历
lines.forEach(System.out::println);
// 统计有多少行
System.out.println(line.lines().count());

// 输出
// a
// b
// c
// 3
```

3、repeat () 复制字符串

```java
// 复制字符串
String repeat = "我是:Sam,";
String repeat3 = repeat.repeat(3);
System.out.println(repeat3);

// 输出
// 我是:Sam,我是:Sam,我是:Sam,
```

4、trim ()、strip ()、stripLeading ()、 stripTrailing () 去除前后空白字符。

```java
String strip = "   　 https://www.baidu.com 　";
// 去除前后空白字符，trim 只能去除半角空格
System.out.println("==" + strip.trim() + "==");
// 去除前后空白字符，如全角空格，TAB
System.out.println("==" + strip.strip() + "==");
// 去前面空白字符，如全角空格，TAB
System.out.println("==" + strip.stripLeading() + "==");
// 去后面空白字符，如全角空格，TAB
System.out.println("==" + strip.stripTrailing() + "==");

// 输出
// ==　 https://www.baidu.com 　==
// ==https://www.baidu.com==
// ==https://www.baidu.com 　==
// ==   　 https://www.baidu.com==
```

> 这里注意：trim 只能去除半角空格，而 strip 是**去除各种空白符**。
>

5、String API 增强方法总结：

```java
String  str       = "    Hello-Java     ";
boolean isblank   = str.isBlank();        // 判断字符串是空白
boolean isempty   = str.isEmpty();        // 判断字符串是否为空
String  result1   = str.strip();          // 首位空白
String  result2   = str.stripTrailing();  // 去除尾部空白
String  result3   = str.stripLeading();   // 去除首部空白
String  copyStr   = str.repeat(2);        // 复制几遍字符串
long    lineCount = str.lines().count();  // 行数统计

System.out.println(isblank);   // false
System.out.println(isempty);   // false
System.out.println(result1);   // Hello-Java
System.out.println(result2);   //     Hello-Java
System.out.println(result3);   // Hello-Java     
System.out.println(copyStr);   //     Hello-Java         Hello-Java     
System.out.println(lineCount); // 1
```



## 2、File API  增强

使得读写文件变得更加方便。

```java
// 创建临时文件
Path path = Files.writeString(Files.createTempFile("test", ".txt"),  "https://www.baidu.com");
System.out.println(path);

// 读取文件
// String ss = Files.readString(Path.of("file.json"));
String s = Files.readString(path);
System.out.println(s);

// 结果
// C:\Users\lsx\AppData\Local\Temp\test16049718326070900232.txt
// https://www.baidu.com
```



## 3、Optional 增强

新增了empty() 方法来判断指定的 Optional 对象是否为空。

```java
var optional = Optional.empty();
// 判断指定的 Optional 对象是否为空
System.out.println(optional.isEmpty()); // true
```



## 4、HTTP Client 增强

> 321: HTTP Client (Standard)

在 Java 11 中 Http Client API 得到了标准化的支持。且支持 HTTP/1.1 和 HTTP/2 ，也支持 WebSockets。

```java
// 同步调用
HttpClient client             = HttpClient.newHttpClient();
HttpRequest request           = HttpRequest.newBuilder(URI.create("")).build();
BodyHandler<String>  handler  = HttpResponse.BodyHandlers.ofString();
HttpResponse<String> response = client.send(request,handler);
String body                   = response.body();
System.out.println(body);

// 异步调用
HttpClient client             = HttpClient.newHttpClient();
HttpRequest request           = HttpRequest.newBuilder(URI.create("")).build();
BodyHandler<String>  handler  = HttpResponse.BodyHandlers.ofString();
CompletableFuture<HttpResponse<String>> response = client.sendAsync(request,handler);
HttpResponse<String> result   = response.get();
String body                   = result.body();
System.out.println(body);
```

简单写法如下：

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://www.hao123.com"))
        .build();

// 同步调用
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());

// 异步调用
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
        .thenApply(HttpResponse::body)
        .thenAccept(System.out::println)
        .join();
```

更多同步异步请求，并发访问，设置代理等方式，可以参考官方文档。

- HTTP Client OpenJDK 官方文档：https://openjdk.org/groups/net/httpclient/recipes-incubating.html




## 5、增强 Lambda 局部变量推断

在 Java 10 中引入了 var 语法，可以自动推断变量类型。在 Java 11 中这个语法糖可以在 Lambda 表达式中使用了。

```java
Consumer<String> consumer1 = t -> System.out.println(t.toUpperCase());
Consumer<String> consumer2 = (var t) -> System.out.println(t.toUpperCase());
// 错误的形式：必须要有类型，可以加上var
Consumer<String> consumer3 = (@Deprecated t) -> System.out.println(t.toUpperCase());
// 正确的形式
Consumer<String> consumer4 = (@Deprecated var t) -> System.out.println(t.toUpperCase());
```

```java
var hashMap = new HashMap<String, Object>();
hashMap.put("id", "001");
hashMap.put("name", "wudi");
hashMap.forEach((var k, var v) -> {
    System.out.println(k + ": " + v);
});
```

这里需要注意的是，(var k, var v) 中，k 和 v 的类型要么都用 var ，要么都不写，要么都写正确的变量类型。而不能 var 和其他变量类型混用。否则会提示报错。



## 6、单命令运行 Java

> 330: Launch Single-File Source-Code Programs

自从学习 Java 的第一天，我们就知道运行一个 Java 文件，要先用 javac 命令编译，再用 java 命令运行，而现在只要一个 java 命令就可以运行了。

```java
$ cat Main.java

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}

$ java Main.java
Hello
```



## 7、支持 Unicode 10

Unicode 10 增加了 8518 个字符，总计达到了 136690 个字符。并且增加了 4 个脚本。同时还有 56 个新的 emoji 表情符号。

## 8、免费的飞行记录器

> 328: Flight Recorder

商业版 JDK 中一直有一款低开销的事件信息收集工具，也就是飞行记录器（Java Flight Recorder），它可以对 JVM 进行检查、分析、记录等。当出现未知异常时可以通过记录进行故障分析。这个好用的工具在 Java 11 中将开源免费。所有人都可以使用这个功能了。



## 9、新的 Epsilon 垃圾收集器

> 318: Epsilon: A No-Op Garbage Collector

A NoOp Garbage Collector JDK 上对这个特性的描述是：开发一个处理内存分配但不实现任何实际内存回收机制的 GC，一旦可用堆内存用完，JVM 就会退出。

如果有 System.gc () 调用，实际上什么也不会发生（这种场景下和 -XX:+DisableExplicitGC 效果一样）， 因为没有内存回收，这个实现可能会警告用户尝试强制 GC 是徒劳。

- 用法：-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC
- 选项：-XX:+UseEpsilonGC，程序很快就因为堆空间不足而退出
- 原因：提供完全被动的 GC 实现，具有有限的分配限制和尽可能低的延迟开销，但代价是内存占用和内存吞吐量，Java 实现可广泛选择高度可配置的 GC 实现。各种可用的收集器最终满足不同的需求，即使它们的可配置性使它们的功能相交。有时更容易维护单独的实现，而不是在现有 GC 实现上堆积另一个配置选项。

主要用途如下：

1. 能测试（它可以帮助过滤掉 GC 引起的性能假象）

2. 内存压力测试（例如：知道测试用例应该分配不超过 1GB 的内存, 我们可以使用：-Xmx1g –XX:+UseEpsilonGC，如果程序有问题，则程序会崩溃）

3. 非常短的JOB任务（对象这种任务，接受GC清理堆那都是浪费空间）

4. VM 接口测试

5. Last-Drop 延迟&吞吐改进



## 10、ZGC 可伸缩低延迟垃圾收集器

> 333: ZGC: A Scalable Low-Latency Garbage Collector (Experimental)

**ZGC 即 Z Garbage Collector**，是一个可伸缩的、低延迟的垃圾收集器。

ZGC 主要为了满足如下目标进行设计：

- GC 停顿时间不超过 10ms
- 即能处理几百 MB 的小堆，也能处理几个 TB 的大堆
- 应用吞吐能力不会下降超过 15%（与 G1 回收算法相比）
- 方便在此基础上引入新的 GC 特性和利用 colored 针以及 Load barriers 优化奠定基础
- 当前只支持 Linux/x64 位平台

ZGC 目前 **处在实验阶段**，只支持 Linux/x64 平台。与 CMS 中的 ParNew 和 G1 类似，ZGC 也采用标记-复制算法，不过 ZGC 对该算法做了重大改进。在 ZGC 中出现 Stop The World 的情况会更少！

详情可以看 ： [《新一代垃圾回收器 ZGC 的探索与实践》open in new window](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)

***

ZGC：这应该是 Java11 最为瞩目的特性，没有之一。但是后面带了 Experimental，说明这还不建议用到生产环境。

GC 暂停时间不会超过10ms，既能处理几百兆的小堆，也能处理几个T的大堆（OMG），和 G1 相比，应用吞吐能力不会下降超过15%，为未来的 GC 功能和利用 colord 指针以及 Load barriers 优化奠定基础，初始只支持64位系统。

ZGC 的设计目标是：支持 TB 级内存容量，暂停时间低（<10ms），对整个程序吞吐量的影响小于 15%。将来还可以扩展实现机制，以支持不少令人兴奋的功能，例如多层堆（即热对象置于 DRAM 和冷对象置于 NVMe 闪存），或压缩堆。

GC 是 Java 主要优势之一，当 GC 停顿太长, 就会开始影响应用的响应时间，消除或者减少 GC 停顿时长，Java 将对更广泛的应用场景是一个更有吸引力的平台。此外，现代系统中可用内存不断增长，用户和程序员希望 JVM 能够以高效的方式充分利用这些内存，并且无需长时间的 GC 暂停时间。

用法：-XX:+UnlockExperimentalVMOptions –XX:+UseZGC，因为 ZGC 还处于实验阶段，所以需要通过 JVM 参数来解锁这个特性。



## 11、支持 G1 上的并行完全垃圾收集

对于 G1 GC，相比于 JDK 8，升级到 JDK 11 即可免费享受到：并行的 Full GC，快速的 CardTable 扫描，自适应的堆占用比例调整（IHOP），在并发标记阶段的类型卸载等等。这些都是针对 G1 的不断增强，其中串行 Full GC 等甚至是曾经被广泛诟病的短板，你会发现 GC 配置和调优在 JDK11 中越来越方便。



## 12、完全支持 Linux 容器（包括 Docker）

许多运行在 Java 虚拟机中的应用程序（包括 Apache Spark 和 Kafka 等数据服务以及传统的企业应用程序）都可以在 Docker 容器中运行。但是在 Docker 容器中运行 Java 应用程序一直存在一个问题，那就是在容器中运行 JVM 程序在设置内存大小和 CPU 使用率后，会导致应用程序的性能下降。这是因为 Java 应用程序没有意识到它正在容器中运行。随着 Java 10 的发布，这个问题总算得以解决，JVM 现在可以识别由容器控制组（cgroups）设置的约束。可以在容器中使用内存和 CPU 约束来直接管理 Java 应用程序，其中包括：

1. 遵守容器中设置的内存限制

2. 在容器中设置可用的 CPU

3. 在容器中设置 CPU 约束

4. Java10 的这个改进在 Docker for Mac、Docker for Windows 以及 Docker Enterprise Edition 等环境均有效。

5. 容器的内存限制

6. 在 Java9 之前，JVM 无法识别容器使用标志设置的内存限制和 CPU 限制。而在 Java10 中，内存限制会自动被识别并强制执行。Java 将服务器类机定义为具有2个 CPU 和 2GB 内存，以及默认堆大小为物理内存的 1/4。



## 13、移除 JavaEE 和 CORBA 技术

在 Java11 中移除了不太使用的 Java EE 模块和 CORBA 技术，在 Java11 中将 Java9 标记废弃的 Java EE 及 CORBA 模块移除掉。

```
 1. java.xml.ws
 2. java.xml.bind
 3. java.xml.ws
 4. java.xml.ws.annotation
 5. jdk.xml.bind
 6. jdk.xml.ws 被移除，只剩下 java.xml，java.xml.crypto,jdk.xml.dom 这几个模块
 7. java.corba
 8. java.se.ee
 9. java.activation
10. java.transaction 被移除，但是Java11新增一个 java.transaction.xa 模块
```



## 14、废除 Nashorn javascript 引擎

> JEP : 335 : Deprecate the Nashorn JavaScript Engine

废除 Nashorn javascript 引擎，在后续版本准备移除掉，有需要的可以考虑使用 GraalVM。



## 15、废除 Pack200 工具和 API

> JEP : 336 : Deprecate the Pack200 Tools and API

Java5 中带了一个压缩工具：Pack200，这个工具能对普通的 jar 文件进行高效压缩。其 实现原理是根据 Java 类特有的结构，合并常数 池，去掉无用信息等来实现对 java 类的高效压缩。由于是专门对 Java 类进行压缩的，所以对普通文件的压缩和普通压缩软件没有什么两样，但是对于 Jar 文件却能轻易达到 10-40% 的压缩率。这在 Java 应用部署中很有用，尤其对于移动 Java 计算，能够大大减小代码下载量。

Java5 中还提供了这一技术的 API 接口，你可以将其嵌入到你的程序中使用。使用的方法很简单，下面的短短几行代码即可以实现 jar 的压缩和解压：

```java
// 压缩
Packer packer = Pack200.newPacker();
OutputStream output = new BufferedOutputStream(new FileOutputStream(outfile));
packer.pack(new JarFile(jarFile), output);
output.close();

// 解压
Unpacker unpacker = Pack200.newUnpacker();
output = new JarOutputStream(new FileOutputStream(jarFile));
unpacker.unpack(pack200File, output);
output.close();
```

Pack200 的压缩和解压缩速度是比较快的，而且压缩率也是很惊人的，在我是使用 的包 4.46MB 压缩后成了 1.44MB（0.322%），而且随着包的越大压缩率会根据明显，据说如果 jar 包都是 class 类可以压缩到 1/9 的大 小。其实 JavaWebStart 还有很多功能，例如可以按不同的 jar 包进行 lazy 下载和 单独更新，设置可以根据 jar 中的类变动进行 class 粒度的下载。但是在 Java11 中废除了 pack200 以及 unpack200 工具以及 java.util.jar 中的 Pack200 API。因为 Pack200 主要是用来压缩 jar 包的工具，由于网络下载速度的提升以及 Java9 引入模块化系统之后不再依赖 Pack200，因此这个版本将其移除掉。



## 16、移除和废弃的内容

移除项

1. 移除了`com.sun.awt.AWTUtilities`
2. 移除了`sun.misc.Unsafe.defineClass`，使用`java.lang.invoke.MethodHandles.Lookup.defineClass`来替代
3. 移除了`Thread.destroy()`以及 `Thread.stop(Throwable)`方法
4. 移除了`sun.nio.ch.disableSystemWideOverlappingFileLockCheck`、`sun.locale.formatasdefaul`t属性
5. 移除了`jdk.snmp`模块
6. 移除了`javafx`，OpenJDK估计从Java10就移除了，OracleJDK10还尚未移除，而Java11版本则OracleJDK版也移除了`javafx`
7. 移除了`Java Mission Control`，从JDK中移除之后，需要自己单独下载
8. 移除了这些`Root Certificates` ：`Baltimore Cybertrust Code Signing CA`，`SECOM` ，`AOL and Swisscom`

废弃项

- `-XX+AggressiveOpts`选项
- `-XX:+UnlockCommercialFeatures`
- `-XX:+LogCommercialFeatures`选项也不再需要



## 17、参考文献 & 鸣谢

1. Java 新特性文档：https://www.wdbyte.com/java-feature/
2. Java New Feature：https://javaguide.cn/java/new-features/
3. Java 的新特性：https://segmentfault.com/a/1190000004417288/
4. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
5. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/
