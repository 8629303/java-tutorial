# Java 12 新特性

Java 12 早在 2019 年 3 月 19 日发布，它不是一个长久支持（LTS）版本。

这次更新共带来 8 个新特性：

- JEP 189: 新增一个名为 Shenandoah 的垃圾回收器，它通过在 Java 线程运行的同时进行疏散 (evacuation) 工作来减少停顿时间（实验）

- JEP 230: 新增一套微基准测试，使开发者能够基于现有的 Java Microbenchmark Harness（JMH）轻松测试 JDK 的性能，并创建新的基准测试。

- JEP 325: 对 Switch 语句进行扩展，使其可以用作语句或表达式，简化日常代码。
- JEP 334: 引入一个 API 来对关键类文件 (key class-file) 和运行时工件的名义描述（nominal descriptions）进行建模，特别是那些可从常量池加载的常量。

- JEP 340: 删除与 arm64 端口相关的所有源码，保留 32 位 ARM 移植和 64 位 aarch64 移植。

- JEP 341: 默认生成类数据共享（CDS）存档。

- JEP 344: 当 G1 垃圾回收器的回收超过暂停目标，则能中止垃圾回收过程。

- JEP 346: 改进 G1 垃圾回收器，以便在空闲时自动将 Java 堆内存返回给操作系统。

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 12 新特性：https://openjdk.org/projects/jdk/12/
- OracleJDK Java 12 新特性：https://www.oracle.com/java/technologies/javase/12-relnote-issues.html#NewFeature
- Oracle JDK 12 文档：https://docs.oracle.com/en/java/javase/12/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## 1、预览新特性概述

1、作为预览特性加入，需要在 javac 编译和 java 运行时增加参数：`--enable-preview`、`--source`

```bash
# 编译时(推荐使用--source)
./bin/javac --enable-preview --source 12 ./Xxx.java
./bin/javac --enable-preview --release 12 ./Xxx.java
# 运行时
./bin/java --enable-preview Xxx
```

2、由于 Java 11 新特性中新增：单命令运行 Java，所以我们可以直接使用 java 命令运行：

```bash
# 编译 + 运行时
./bin/java --enable-preview --source 12 ./Xxx.java
```

3、如果是使用 IDEA 编码设置如下：

- File =》 Settings =》 Build, Execution, Deplloyment =》 Compiler => Java Compiler
  - 全局配置：Additional command line parameters =》`–enable-preview --add-modules jdk.incubator.concurrent`
  - 模块配置：Override compiler parameters per-module =》`–enable-preview --add-modules jdk.incubator.concurrent`
- 注意：最后还需要在：Edit Run/Debug Configuration =》Modify options =》Add VM options，中添加`–enable-preview`



## 2、Switch 表达式增强【预览】

预览功能，如果该 JDK12 的 Switch 效果不好的话，会被下一版本 JDK13 直接移除该功能，并不是之后每个版本必须的。Java 12 中，对 Switch 表达式的写法进行了改进，虽然是一个语法糖的改进，也让 Switch 的代码编写变得更加优雅。

【Java 12 之前的 Switch 的写法】：

```java
// 通过传入月份，输出月份所属的季节
public static void switchJava12Before(String day) {
    switch (day) {
        case "march":
        case "april":
        case "may":
            System.out.println("春天");
            break;
        case "june":
        case "july":
        case "august":
            System.out.println("夏天");
            break;
        case "september":
        case "october":
        case "november":
            System.out.println("秋天");
            break;
        case "december":
        case "january":
        case "february":
            System.out.println("冬天");
            break;
    }
}

// 通过传入月份，返回月份所属的季节
public static void switchJava12Before(String day) {
    String season;
    switch (day) {
        case "march":
        case "april":
        case "may":
            season = "春天";
            break;
        case "june":
        case "july":
        case "august":
            season = "夏天";
            break;
        case "september":
        case "october":
        case "november":
            season = "秋天";
            break;
        case "december":
        case "january":
        case "february":
            season = "冬天";
            break;
    }
}
```

> 太多 case 和 break 使它冗长，且漏 break，会产生意料之外的结果或者异常，所以对此 JDK12 在这里进行了优化升级。

【Java 12 Switch 的写法】：

1. 由于 Switch 存在的上述问题，所以在 Java 12 中对 Switch 进行了改进，让其可以使用 `case L ->` 的方式进行操作

   ```java
   public static void switchJava12(String day) {
       switch (day) {
           case "march", "april", "may"            -> System.out.println("春天");
           case "june", "july", "august"           -> System.out.println("夏天");
           case "september", "october", "november" -> System.out.println("秋天");
           case "december", "january", "february"  -> System.out.println("冬天");
           default                                 -> System.out.println("错误");
       }
   }
   ```

2. 这还不够，在 Switch 的改进中，还支持了使用 Switch 的返回值进行赋值。像下面这样：

   ```java
   public static void switchJava12(String day) {
       String season = switch (day) {
           case "march", "april", "may"            -> "春天";
           case "june", "july", "august"           -> "夏天";
           case "september", "october", "november" -> "秋天";
           case "december", "january", "february"  -> "冬天";
           default -> {
               // throw new RuntimeException("day error")
               System.out.println("day error");
               break "day error";
           }
       };
       System.out.println("当前季节是:" + season);
       return season;
   }
   
   // 如上的方式是其实就是使用在类上，我想到的这个 switch 这个功能可以用在抽象工厂类方面
   T result = switch (arg) {
       case L1 -> e1;
       case L2 -> e2;
       default -> e3;
   };
   ```

   > 虽然编写更加简单了，其实这些只不过是语法糖式的更新，编译后和之前并没有太大区别。



## 3、String 新增方法

Java 12 String 类增加了两个方法：transform()、indent()

- `indent()` 方法可以实现字符串缩进。换行符 \n 后向前缩进 n 个空格，为 0 或负数不缩进。
- `transform()` 方法可以用来转变指定字符串。

【操作示例 1】indent() 使用

```java
String text = "Java";
// 缩进 4 格
text = text.indent(4);
System.out.println(text);
// 为 0 或负数不缩进。
text = text.indent(-10);
System.out.println(text);
```

```
    Java

Java
```

【操作示例 2】transform() 使用

```java
String result = "foo".transform(input -> input + " bar");
System.out.println(result); // foo bar
```



## 4、Files 新增文件对比方法

新增文件对比方法 **Files.mismatch**：在 Java 中对于文件的操作已经在 Java 7 中进行了一次增强，这次 Java 12 又带来了文件对比功能。对比两个文件，如果内容一致，会返回 -1 ，如果内容不同，会返回不同的字节开始位置。

```java
// 创建两个文件
Path pathA = Files.createFile(Paths.get("a.txt"));
Path pathB = Files.createFile(Paths.get("b.txt"));

// 写入相同内容
Files.write(pathA, "abc".getBytes(), StandardOpenOption.WRITE);
Files.write(pathB, "abc".getBytes(), StandardOpenOption.WRITE);
long mismatch = Files.mismatch(pathA, pathB);
System.out.println(mismatch);  // 输出: -1

// 追加不同内容
Files.write(pathA, "123".getBytes(), StandardOpenOption.APPEND);
Files.write(pathB, "321".getBytes(), StandardOpenOption.APPEND);
mismatch = Files.mismatch(pathA, pathB);
System.out.println(mismatch);  // 输出: 3

// 删除创建的文件
pathA.toFile().deleteOnExit();
pathB.toFile().deleteOnExit();
```



## 5、核心库  java.text 支持压缩数字格式

核心库 java.text 支持压缩数字格式：NumberFormat 新增了对复杂的数字进行格式化的支持，简化的数字格式可以直接转换数字显示格式，比如 1000 -> 1K，1000000 -> 1M 。

```java
NumberFormat upvotes = NumberFormat.getCompactNumberInstance(new Locale("en", "US"), NumberFormat.Style.SHORT);

System.out.println(upvotes.format(100));       // 100
System.out.println(upvotes.format(1000));      // 1K
System.out.println(upvotes.format(10000));     // 10K
System.out.println(upvotes.format(100000));    // 100K
System.out.println(upvotes.format(1000000));   // 1M

// 设置小数位数
upvotes.setMaximumFractionDigits(1);
System.out.println(upvotes.format(1234));      // 1.2K
System.out.println(upvotes.format(123456));    // 123.5K
System.out.println(upvotes.format(12345678));  // 12.3M

var cnf = NumberFormat.getCompactNumberInstance(Locale.CHINA, NumberFormat.Style.SHORT);
System.out.println(cnf.format(1_0000));     // 1万
System.out.println(cnf.format(1_9200));     // 2万
System.out.println(cnf.format(1_000_000));  // 100万
System.out.println(cnf.format(1L << 30));   // 11亿
System.out.println(cnf.format(1L << 40));   // 1兆
System.out.println(cnf.format(1L << 50));   // 1126兆
```



## 6、新增 Collectors.teeing 方法

Collectors.teeing：将 downstream1 和 downstream2 的流入合并，然后从 merger 流出

【操作示例 1】一次性过滤2种规则

```java
var result1 = Stream.of("Devoxx", "Voxxed Days", "Code One", "Basel One")
    .collect(Collectors.teeing(
        Collectors.filtering(n -> n.contains("xx"), Collectors.toList()),
        Collectors.filtering(n -> n.endsWith("One"), Collectors.toList()),
        (List<String> list1, List<String> list2) -> List.of(list1, list2)
    ));
System.out.println(result1.get(0));  // [Devoxx, Voxxed Days]
System.out.println(result1.get(1));  // [Code One, Basel One]
```

【操作示例 2】一次性请求出流中的个数和总和

```java
var result2 = Stream.of(2, 11, 1, 5, 7, 8, 12)
    .collect(Collectors.teeing(
        Collectors.counting(),
        Collectors.summingLong(e -> e),
        (Long i1, Long i2) -> Arrays.asList(i1, i2)
    ));
System.out.println(result2.get(0));  // 7
System.out.println(result2.get(1));  // 46
```



## 7、CompletionStage 新增方法

CompletionStage 新增 exceptionallyAsync、exceptionallyCompose、exceptionallyComposeAsync 方法。

JDK12 之前 CompletionStage 只有一个exceptionally，该方法体在主线程执行，JDK12 新增了exceptionallyAsync、exceptionallyComposeAsync 方法允许方法体在异步线程执行，同时新增了 exceptionallyCompose 方法支持在 exceptionally 的时候构建新的 CompletionStage。

【操作示例 1】exceptionallyAsync（对应 exceptionally）

```java
System.out.println("begin");
int result = CompletableFuture.supplyAsync(() -> {
    System.out.println("calculate");
    int i = 1 / 0;
    return 100;
}).exceptionallyAsync(ex -> {
    System.out.println("error error: " + ex.getMessage());
    return 0;
}).get();
System.out.println("result: " + result);
```

```java
begin
calculate
error error: java.lang.ArithmeticException: / by zero
result: 0
```

【操作示例 2】exceptionallyCompose （对应 exceptionallyComposeAsync）

```java
System.out.println("begin");
CompletableFuture<Integer> calculate = CompletableFuture.supplyAsync(() -> {
    System.out.println("calculate");
    int i = 1 / 0;
    return 100;
});
calculate.exceptionallyComposeAsync(ex -> {
    System.out.println("error error:" + ex.getMessage());
    return calculate;
});
int result = calculate.get();
System.out.println("result:" + result);
```

```java
begin
calculate
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
	at java.base/java.util.concurrent.CompletableFuture.reportGet(CompletableFuture.java:395)
	at java.base/java.util.concurrent.CompletableFuture.get(CompletableFuture.java:2070)
	at org.example.Main.main(Main.java:11)
Caused by: java.lang.ArithmeticException: / by zero
	at org.example.Main.lambda$main$0(Main.java:4)
	at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run$$$capture(CompletableFuture.java:1771)
	at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java)
	at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.exec(CompletableFuture.java:1763)
	at java.base/java.util.concurrent.ForkJoinTask.doExec$$$capture(ForkJoinTask.java:290)
	at java.base/java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java)
	at java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(ForkJoinPool.java:1020)
	at java.base/java.util.concurrent.ForkJoinPool.scan(ForkJoinPool.java:1656)
	at java.base/java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1594)
	at java.base/java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:177)
```



## 8、核心库 java.lang 支持 Unicode 11

在 Java 11 支持了 Unicode 10 之后， Java 12 支持了 Unicode 11，支持操作更多的表情、符号。JDK 12 中包含以下新功能：

```
1、684个新角色(684个新字符)
    1.1、66个表情符号字符
    1.2、Copyleft符号
    1.3、评级系统的半星
    1.4、额外的占星符号
    1.5、象棋中国象棋符号
2、11个新区块
    2.1、格鲁吉亚扩展（Georgian Extended）
    2.2、玛雅数字（Mayan Numerals）
    2.3、印度Siyaq数字（Mayan Numerals）
    2.4、国际象棋符号（Mayan Numerals）
3、7个新脚本
    3.1、Hanifi Rohingya
    3.2、Old Sogdian
    3.3、Sogdian
    3.4、Dogra
    3.5、Gunjala Gondi
    3.6、Makasar
    3.7、Medefaidrin
```



## 9、关于 GC 特性

### 1、Shenandoah GC

Shenandoah GC：低停顿时间的GC（实验性）：Shenandoah 垃圾回收器是 Red Hat 在 2014 年宣布进行的一项垃圾收集器研究项目 Pauseless GC 的实现，旨在针对 JVM 上的内存收回实现低停顿的需求。该设计将与应用程序线程并发，通过交换 CPU 并发周期和空间以改善停顿时间，使得垃圾回收器执行线程能够在 Java 线程运行时进行堆压缩，并且标记和整理能够同时进行，因此避免了在大多数 JVM 垃圾收集器中所遇到的问题。据 Red Hat 研发 Shenandoah 团队对外宣称，Shenandoah 垃圾回收器的暂停时间与堆大小无关，这意味着无论将堆设置为 200 MB 还是 200 GB，都将拥有一致的系统暂停时间，不过实际使用性能将取决于实际工作堆的大小和工作负载。与其他 Pauseless GC 类似，Shenandoah GC 主要目标是 99.9% 的暂停小于 10ms，暂停与堆大小无关等。这是一个实验性功能，不包含在默认（Oracle）的OpenJDK版本中。

**STW：Stop-the-World，简称STW ，指的是GC 事件发生过程中，停止所有的应用程序线程的执行。**

  垃圾回收器的任务是识别和回收垃圾对象进行内存清理。垃圾回收要求系统进入一个停顿的状态。停顿的目的是终止所有应用程序的执行，这样才不会有新的垃圾产生，同时保证了系统状态在某一个瞬间的一致性，并且有益于垃圾回收器更好地标记垃圾对象。停顿产生时整个应用程序会被暂停，没有任何响应，有点像卡死的感觉，这个停顿称为STW 。

  如果Stop-the-World 出现在新生代的 Minor GC 中时， 由于新生代的内存空间通常都比较小，所以暂停的时间较短,在可接受的范围内，老年代的Full GC 中时，程序的工作线程被暂停的时间将会更久。内存空间越大，执行Full GC 的时间就会越久，工作线程被暂停的时间也就会更长。到目前为止，哪怕是G1 也不能完全避免Stop-the-world 情况发生，只能说垃圾回收器越来越优秀，尽可能地缩短了暂停时间。

Java 垃圾收集器的分类 常见的：Serial（串行）、ParNEW（并行）、Parallel（吞吐优先并行）、CMS（低延迟）、G1（区域化分代），不同的垃圾收集器都有自己的特征，简单进行一个分类。

**按线程数分类：**

1. 串行垃圾回收器：串行回收指的是在同一时间段内只允许一件事情发生，简单来说，当多个CPU 可用时，也只能有一个CPU用于执行垃圾回收操作，井且在执行垃圾回收时，程序中的工作线程将会被暂停，当垃圾收集工作完成后才会恢复之前被暂停的工作线程，这就是串行回收。
2. 并行垃圾回收器：和串行回收相反，并行收集可以运用多个CPU 同时执行垃圾回收，因此提升了应用的吞吐量，不过并行回
   收仍然与串行回收一样，采用独占式，使用了 “Stop-the-world” 机制和复制算法。

**按照工作模式分类：**

1. 发式垃圾收集器：并发式垃圾回收器与应用程序线程交替工作，以尽可能减少应用程序的停顿时间。
2. 独占式垃圾收集器：独占式垃圾回收器（ Stop the world)一旦运行，就停止应用程序中的其他所有线程，直到垃圾回收过程完全结束。

**按照碎片处理方式：**

1. 压缩式垃圾回收器;:压缩式垃圾回收器会在回收完成后，对存活对象进行压缩整理，消除回收后的碎片。
2. 非压缩式垃圾回收器

非压缩式的垃圾回收器不进行这步操作。

**按照工作的内存区间：**

1. 年轻代垃圾回收器
2. 老年代垃圾回收器

**如何判断垃圾回收器的性能：**

1. 吞吐量：程序的运行时间（程序的运行时间＋内存回收的时间）。
2. 垃圾收集开销：吞吐量的补数，垃圾收集器所占时间与总时间的比例。
3. 暂停时间：执行垃圾收集时，程序的工作线程被暂停的时间,就是用户功能延迟的时间。
4. 收集频率：相对于应用程序的执行，收集操作发生的频率。
5. 堆空间： Java 堆区所占的内存大小。
6. 快速： 一个对象从诞生到被回收所经历的时间。

  垃圾收集器中吞吐量和低延迟这两个目标本身是相互矛盾的，因为如果选择以吞吐量优先(单位时间希望运行更多的应用程序)，那么必然需要降低内存回收的执行频率，但是这样会导致GC 需要更长的暂停时间来执行内存回收。相反的，如果选择以低延迟优先为原则，那么为了降低每次执行内存回收时的暂停时间，也只能频繁地执行内存回收，但这又引起了年轻代内存的缩减和导致程序吞吐量的下降。

**Shenandoah 工作原理：**

  从原理的角度，我们可以参考该项目官方的示意图，其内存结构与 G1 非常相似，都是将内存划分为类似棋盘的region。整体流程与 G1 也是比较相似的，最大的区别在于实现了并发的 疏散(Evacuation) 环节，引入的 Brooks Forwarding Pointer 技术使得 GC 在移动对象时，对象引用仍然可以访问。

了解 Shenandoah GC 的人比较少，提及比较多的是 Oracle 在 JDK11 中开源出来的 ZGC，或者商业版本的 Azul C4（Continuously Concurrent Compacting Collector）。也有人认为Shenandoah 其实际意义大于后两者,原因有一下几点：

1. 使用 ZGC 的最低门槛是升级到 JDK11，版本的更新不是一件容易的事情,而且 ZGC 实际表现如何也是尚不清楚。
2. C4 成本较高,很多企业甚至斤斤计较几百元的软件成本。
3. Shenandoah GC 可是有稳定的 JDK8u 版本发布的。甚至已经有公司在 HBase 等高实时性产品中有较多的实践。
4. ZGC也是面向low-pause-time的垃圾收集器，不过ZGC是基于colored pointers来实现，而Shenandoah GC是
5. 基于 brooks pointers 来实现。

  不是唯有 GC 停顿可能导致常规应用程序响应时间比较长。具有较长的 GC 停顿时间会导致系统响应慢的问题，但响应时间慢并非一定是 GC 停顿时间长导致的，队列延迟、网络延迟、其他依赖服务延迟和操作提供调度程序抖动等都可能导致响应变慢。使用 Shenandoah 时需要全面了解系统运行情况，综合分析系统响应时间。下面是jbb15benchmark 中，Shenandoah GC 相对于其他主流 GC 的表现。

  GC 暂停相比于 CMS 等选择有数量级程度的提高，对于 GC 暂停非常敏感的场景，价值还是很明显的，能够在 SLA层面有显著提高。当然，这种对于低延迟的保证，也是以消耗 CPU 等计算资源为代价的，实际吞吐量表现也不是非常明朗，需要看企业的实际场景需求，并不是一个一劳永逸的解决方案。

```bash
-XX:+AlwaysPreTouch：使用所有可用的内存分页，减少系统运行停顿，为避免运行时性能损失。
  
-Xmx == -Xmsv：设置初始堆大小与最大值一致，可以减轻伸缩堆大小带来的压力，与 AlwaysPreTouch 参数配
合使用，在启动时提交所有内存，避免在最终使用中出现系统停顿。
  
-XX:+ UseTransparentHugePages：能够大大提高大堆的性能，同时建议在 Linux 上使用时将
/sys/kernel/mm/transparent_hugepage/enabled 和
/sys/kernel/mm/transparent_hugepage/defragv 设置为：madvise，同时与 AlwaysPreTouch 一起使
用时，init 和 shutdownv 速度会更快，因为它将使用更大的页面进行预处理。
  
-XX:+UseNUMA：虽然 Shenandoah 尚未明确支持 NUMA（Non-Uniform Memory Access），但最好启用此功
能以在多插槽主机上启用 NUMA 交错。与 AlwaysPreTouch 相结合，它提供了比默认配置更好的性能。
  
-XX:+DisableExplicitGC：忽略代码中的 System.gc() 调用。当用户在代码中调用 System.gc() 时会强制
Shenandoah 执行 STW Full GC ，应禁用它以防止执行此操作，另外还可以使用 
  
-XX:+ExplicitGCInvokesConcurrent，在 调用 System.gc() 时执行 CMS GC 而不是 Full GC，建议在有
System.gc() 调用的情况下使用。
不过目前 Shenandoah 垃圾回收器还被标记为实验项目，如果要使用Shenandoah GC需要编译时--with-jvmfeatures
选项带有shenandoahgc，然后启动时使用参数

# 要启用/使用Shenandoah GC，需要以下JVM选项：
-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
```



### 2、可中断的 G1 Mixed GC

  当 G1 垃圾回收器的回收超过暂停时间的目标，则能中止垃圾回收过程。

  G1是一个垃圾收集器，设计用于具有大量内存的多处理器机器。由于它提高了性能效率，G1垃圾收集器最终将取代
CMS垃圾收集器。该垃圾收集器设计的主要目标之一是满足用户设置的预期的 JVM 停顿时间。

  G1 采用一个高级分析引擎来选择在收集期间要处理的工作量，此选择过程的结果是一组称为 GC 回收集（collection set( CSet )）的区域。一旦收集器确定了 GC 回收集 并且 GC 回收、整理工作已经开始，这个过程是without stopping的，即 G1 收集器必须完成收集集合的所有区域中的所有活动对象之后才能停止；但是如果收集器选择过大的 GC 回收集，此时的STW时间会过长超出目标pause time。

  这种情况在mixed collections时候比较明显。这个特性启动了一个机制，当选择了一个比较大的collection set，Java 12 中将把 GC 回收集（混合收集集合）拆分为mandatory（必需或强制）及optional两部分( 当完成mandatory的部分，如果还有剩余时间则会去处理optional部分)来将mixed collections从without stopping变为abortable，以更好满足指定pause time的目标。

- 其中必需处理的部分包括 G1 垃圾收集器不能递增处理的 GC 回收集的部分（如年轻代），同时也可以包含老年代以提高处理效率。
- 将 GC 回收集拆分为必需和可选部分时，垃圾收集过程优先处理必需部分。同时，需要为可选 GC 回收集部分维护一些其他数据，这会产生轻微的 CPU 开销，但小于 1 ％的变化，同时在 G1 回收器处理 GC 回收集期间，本机内存使用率也可能会增加，使用上述情况只适用于包含可选 GC 回收部分的 GC 混合回收集合。
- 在 G1 垃圾回收器完成收集需要必需回收的部分之后，如果还有时间的话，便开始收集可选的部分。但是粗粒度的处理，可选部分的处理粒度取决于剩余的时间，一次只能处理可选部分的一个子集区域。在完成可选收集部分的收集后，G1 垃圾回收器可以根据剩余时间决定是否停止收集。如果在处理完必需处理的部分后，剩余时间不足，总时间花销接近预期时间，G1 垃圾回收器也可以中止可选部分的回收以达到满足预期停顿时间的目标。



### 3、增强 G1 垃圾回收器

> **G1 概述**

  上面介绍了 Java 12 中增强了 G1 垃圾收集器关于混合收集集合的处理策略，这节主要介绍在 Java 12 中同时也对 G1垃圾回收器进行了改进，使其能够在空闲时自动将 Java 堆内存返还给操作系统，这也是 Java 12 中的另外一项重大改进。

  目前 Java 11 版本中包含的 G1 垃圾收集器暂时无法及时将已提交的 Java 堆内存返回给操作系统。为什么呢？ G1目前只有在full GC或者concurrent cycle（并发处理周期）的时候才会归还内存，由于这两个场景都是G1极力避免的，

  因此在大多数场景下可能不会及时归还committed Java heap memory给操作系统。除非有外部强制执行。

  在使用云平台的容器环境中，这种不利之处特别明显。即使在虚拟机不活动，但如果仍然使用其分配的内存资源，哪怕是其中的一小部分，G1 回收器也仍将保留所有已分配的 Java 堆内存。而这将导致用户需要始终为所有资源付费，哪怕是实际并未用到，而云提供商也无法充分利用其硬件。如果在此期间虚拟机能够检测到 Java 堆内存的实际使用情况，并在利用空闲时间自动将 Java 堆内存返还，则两者都将受益。



> **具体操作**

  为了尽可能的向操作系统返回空闲内存，G1 垃圾收集器将在应用程序不活动期间定期生成或持续循环检查整体 Java堆使用情况，以便 G1 垃圾收集器能够更及时的将 Java 堆中不使用内存部分返还给操作系统。对于长时间处于空闲状态的应用程序，此项改进将使 JVM 的内存利用率更加高效。而在用户控制下，可以可选地执行 Full GC，以使返回的内存量最大化。

  JDK12 的这个特性新增了两个参数分别是 G1 PeriodicGCInterval 及 G1 PeriodicGCSystemLoadThreshold，设置为0的话，表示禁用。如果应用程序为非活动状态，在下面两种情况任何一个描述下，G1 回收器会触发定期垃圾收集：

- 自上次垃圾回收完成以来已超过 G1PeriodicGCInterval ( milliseconds )， 并且此时没有正在进行的垃圾回收任务。如果 G1PeriodicGCInterval 值为零表示禁用快速回收内存的定期垃圾收集。
- 应用所在主机系统上执行方法 getloadavg()，默认一分钟内系统返回的平均负载值低于G1PeriodicGCSystemLoadThreshold指定的阈值，则触发full GC或者concurrent GC( 如果开启G1PeriodicGCInvokesConcurrent )，GC 之后 Java heap size会被重写调整，然后多余的内存将会归还给操作系统。如果 G1PeriodicGCSystemLoadThreshold 值为零，则此条件不生效。

  如果不满足上述条件中的任何一个，则取消当期的定期垃圾回收。等一个 G1PeriodicGCInterval 时间周期后，将重新考虑是否执行定期垃圾回收。

  G1 定期垃圾收集的类型根据 G1PeriodicGCInvokesConcurrent 参数的值确定：如果设置值了，G1 垃圾回收器将继续上一个或者启动一个新并发周期；如果没有设置值，则 G1 回收器将执行一个Full GC。在每次一次 GC 回收末尾，G1 回收器将调整当前的 Java 堆大小，此时便有可能会将未使用内存返还给操作系统。新的 Java 堆内存大小根据现有配置确定，具体包括下列配置：

```bash
-XX:MinHeapFreeRatio
-XX:MaxHeapFreeRatio
-Xms
-Xmx
```

  默认情况下，G1 回收器在定期垃圾回收期间新启动或继续上一轮并发周期，将最大限度地减少应用程序的中断。如果定期垃圾收集严重影响程序执行，则需要考虑整个系统 CPU 负载，或让用户禁用定期垃圾收集。



## 10、其他的特性

### 1、JVM 常量 API

Java 12 中引入 JVM 常量 API，用来更容易地对关键类文件（key class-file）和运行时构件（artefact）的名义描述（nominal description） 进行建模，特别是对那些从常量池加载的常量，这是一项非常技术性的变化，能够以更简单、标准的方式处理可加载常量。

具体来说就是 java.base 模块新增了 java.lang.constant 包。包中定义了一系列基于值的符号引用（JVMS 5.1）类型，它们能够描述每种可加载常量。

官方API链接地址：http://cr.openjdk.java.net/~iris/se/12/latestSpec/api/java.base/java/lang/constant/package-summary.html

Java SE > Java SE Specifications > Java Virtual Machine Specification下的第5章：

- Chapter 5. Loading, Linking, and Initializing：https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-5.html

引入了ConstantDesc接口( ClassDesc、MethodTypeDesc、MethodHandleDesc这几个接口直接继承了ConstantDesc接口)以及Constable接口；ConstantDesc接口定义了resolveConstantDesc方法，Constable接口定义了describeConstable方法；String、Integer、Long、Float、Double均实现了这两个接口，而EnumDesc实现了ConstantDesc接口

| Interface              | Description                                                  |
| :--------------------- | :----------------------------------------------------------- |
| ClassDesc              | A nominal descriptor for a `Class` constant.                 |
| Constable              | Represents a type which is constable.                        |
| ConstantDesc           | A nominal descriptor for a loadable constant value, as defined in JVMS 4.4. |
| DirectMethodHandleDesc | A nominal descriptor for a direct MethodHandle.              |
| MethodHandleDesc       | A nominal descriptor for a MethodHandle constant.            |
| MethodTypeDesc         | A nominal descriptor for a MethodType constant.              |

| Class                    | Description                                                  |
| :----------------------- | :----------------------------------------------------------- |
| ConstantDescs            | Predefined values of nominal descriptor for common constants, including descriptors for primitive class types and other common platform types, and descriptors for method handles for standard bootstrap methods. |
| DynamicCallSiteDesc      | A nominal descriptor for an `invokedynamic` call site.       |
| `DynamicConstantDesc<T>` | A nominal descriptor for a dynamic constant (one described in the constant pool with Constant_Dynamic_info.) |

| Enum                        | Description                                                  |
| :-------------------------- | :----------------------------------------------------------- |
| DirectMethodHandleDesc.Kind | Kinds of method handles that can be described with DirectMethodHandleDesc. |

符号引用以纯 nominal 形式描述可加载常量，与类加载或可访问性上下文区分开。有些类可以作为自己的符号引用（例如 String）。而对于可链接常量，另外定义了一系列符号引用类型，具体包括： ClassDesc (Class 的可加载常量标称描述符) ，MethodTypeDesc(方法类型常量标称描述符) ，MethodHandleDesc (方法句柄常量标称描述符) 和DynamicConstantDesc (动态常量标称描述符) ，它们包含描述这些常量的 nominal 信息。此 API 对于操作类和方法的工具很有帮助。

ConstantDesc 源码：

```java
/**A nominal descriptor for a loadable constant value, as defined in JVMS 4.4. Such a descriptor can be resolved via resolveConstantDesc(MethodHandles.Lookup) to yield the constant value itself.
Class names in a nominal descriptor, like class names in the constant pool of a classfile, must be interpreted with respect to a particular class loader, which is not part of the nominal descriptor.
Static constants that are expressible natively in the constant pool (String, Integer, Long, Float, and Double) implement ConstantDesc, and serve as nominal descriptors for themselves. Native linkable constants (Class, MethodType, and MethodHandle) have counterpart ConstantDesc types: ClassDesc, MethodTypeDesc, and MethodHandleDesc. Other constants are represented by subtypes of DynamicConstantDesc.
APIs that perform generation or parsing of bytecode are encouraged to use ConstantDesc to describe the operand of an ldc instruction (including dynamic constants), the static bootstrap arguments of dynamic constants and invokedynamic instructions, and other bytecodes or classfile structures that make use of the constant pool.
Constants describing various common constants (such as ClassDesc instances for platform types) can be found in ConstantDescs.
Implementations of ConstantDesc should be immutable and their behavior should not rely on object identity.
Non-platform classes should not implement ConstantDesc directly. Instead, they should extend DynamicConstantDesc (as Enum.EnumDesc and invoke.VarHandle.VarHandleDesc do.)
Nominal descriptors should be compared using the Object.equals(Object) method. There is no guarantee that any particular entity will always be represented by the same descriptor instance.
API Note:
In the future, if the Java language permits, ConstantDesc may become a sealed interface, which would prohibit subclassing except by explicitly permitted types. Clients can assume that the following set of subtypes is exhaustive: String, Integer, Long, Float, Double, ClassDesc, MethodTypeDesc, MethodHandleDesc, and DynamicConstantDesc; this list may be extended to reflect future changes to the constant pool format as defined in JVMS 4.4.
Since:
12
See Also:
Constable, ConstantDescs
*/
public interface ConstantDesc {
    /**
     * Resolves this descriptor reflectively, emulating the resolution behavior
     * of JVMS 5.4.3 and the access control behavior of JVMS 5.4.4.  The resolution
     * and access control context is provided by the {@link MethodHandles.Lookup}
     * parameter.  No caching of the resulting value is performed.
     *
     * @param lookup The {@link MethodHandles.Lookup} to provide name resolution
     *               and access control context
     * @return the resolved constant value
     * @throws ReflectiveOperationException if a class, method, or field
     * could not be reflectively resolved in the course of resolution
     * @throws LinkageError if a linkage error occurs
     *
     * @apiNote {@linkplain MethodTypeDesc} can represent method type descriptors
     * that are not representable by {@linkplain MethodType}, such as methods with
     * more than 255 parameter slots, so attempts to resolve these may result in errors.
     *
     * @jvms 5.4.3 Resolution
     * @jvms 5.4.4 Access Control
     */
    Object resolveConstantDesc(MethodHandles.Lookup lookup) throws ReflectiveOperationException;
}
```

Constable 接口源码

```java
/**
 * Represents a type which is <em>constable</em>.  A constable type is one whose
 * values are constants that can be represented in the constant pool of a Java
 * classfile as described in JVMS 4.4, and whose instances can describe themselves
 * nominally as a {@link ConstantDesc}.
 *
 * <p>Some constable types have a native representation in the constant pool:
 * {@link String}, {@link Integer}, {@link Long}, {@link Float},
 * {@link Double}, {@link Class}, {@link MethodType}, and {@link MethodHandle}.
 * The types {@link String}, {@link Integer}, {@link Long}, {@link Float},
 * and {@link Double} serve as their own nominal descriptors; {@link Class},
 * {@link MethodType}, and {@link MethodHandle} have corresponding nominal
 * descriptors {@link ClassDesc}, {@link MethodTypeDesc}, and {@link MethodHandleDesc}.
 *
 * <p>Other reference types can be constable if their instances can describe
 * themselves in nominal form as a {@link ConstantDesc}. Examples in the Java SE
 * Platform API are types that support Java language features such as {@link Enum},
 * and runtime support classes such as {@link VarHandle}.  These are typically
 * described with a {@link DynamicConstantDesc}, which describes dynamically
 * generated constants (JVMS 4.4.10).
 *
 * <p>The nominal form of an instance of a constable type is obtained via
 * {@link #describeConstable()}. A {@linkplain Constable} need
 * not be able to (or may choose not to) describe all its instances in the form of
 * a {@link ConstantDesc}; this method returns an {@link Optional} that can be
 * empty to indicate that a nominal descriptor could not be created for an instance.
 * (For example, {@link MethodHandle} will produce nominal descriptors for direct
 * method handles, but not necessarily those produced by method handle
 * combinators.)
 * @jvms 4.4 The Constant Pool
 * @jvms 4.4.10 The {@code CONSTANT_Dynamic_info} and {@code CONSTANT_InvokeDynamic_info} Structures
 *
 * @since 12
 */
public interface Constable {
    /**
     * Returns an {@link Optional} containing the nominal descriptor for this
     * instance, if one can be constructed, or an empty {@link Optional}
     * if one cannot be constructed.
     *
     * @return An {@link Optional} containing the resulting nominal descriptor,
     * or an empty {@link Optional} if one cannot be constructed.
     */
    Optional<? extends ConstantDesc> describeConstable();
}
```

String 源码

```java
/**
* Returns an {@link Optional} containing the nominal descriptor for this
* instance, which is the instance itself.
*
* @return an {@link Optional} describing the {@linkplain String} instance
* @since 12
*/
@Override
public Optional<String> describeConstable() {
    return Optional.of(this);
}
/**
* Resolves this instance as a {@link ConstantDesc}, the result of which is
* the instance itself.
*
* @param lookup ignored
* @return the {@linkplain String} instance
* @since 12
*/
@Override
public String resolveConstantDesc(MethodHandles.Lookup lookup) {
    return this;
}
```

【操作示例】

```java
String name = "Hello Java";
Optional<String> optional = name.describeConstable();
System.out.println(optional.get()); // Hello Java
```



### 2、微基准测试套件

> 什么是 JMH(java微基准测试)

JMH，即Java Microbenchmark Harness，是专门用于代码微基准测试的工具套件。何谓Micro Benchmark呢？简单的来说就是基于方法层面的基准测试，精度可以达到微秒级。当你定位到热点方法，希望进一步优化方法性能的时候，就可以使用JMH对优化的结果进行量化的分析。

> 应用场景

1. 想准确的知道某个方法需要执行多长时间，以及执行时间和输入之间的相关性；
2. 对比接口不同实现在给定条件下的吞吐量；
3. 查看多少百分比的请求在多长时间内完成；

> JMH 使用

如果要在现有Maven项目中使用 JMH，只需要把生成出来的两个依赖以及shade插件拷贝到项目的pom中即可。

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>0.7.1</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>0.7.1</version>
    <scope>provided</scope>
</dependency>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <finalName>microbenchmarks</finalName>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>org.openjdk.jmh.Main</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

> 新特性说明
>

Java 12 中添加一套新的基本的微基准测试套件（microbenchmarks suite），此功能为JDK源代码添加了一套微基准测试（大约100个），简化了现有微基准测试的运行和新基准测试的创建过程.使开发人员可以轻松运行现有的微基准测试并创建新的基准测试，其目标在于提供一个稳定且优化过的基准。 它基于Java Microbenchmark
Harness（JMH），可以轻松测试JDK性能，支持JMH更新。

微基准套件与 JDK 源代码位于同一个目录中，并且在构建后将生成单个 jar 文件。但它是一个单独的项目，在支持构建期间不会执行，以方便开发人员和其他对构建微基准套件不感兴趣的人在构建时花费比较少的构建时间。

要构建微基准套件，用户需要运行命令：make build-microbenchmark， 类似的命令还有：make test TEST=“micro:java.lang.invoke” 将使用默认设置运行java.lang.invoke 相关的微基准测试。



### 3、只保留一个AArch64实现

> 现状
>

当前 Java 11 及之前版本JDK中存在两个64位ARM端口。这些文件的主要来源位于src/hotspot/cpu/arm 和
open/src/hotspot/cpu/aarch64 目录中。尽管两个端口都产生了aarch64 实现，我们将前者（由Oracle贡献）称为arm64 ，将后者称为aarch64 。

> 新特征
>

Java 12 中将删除由 Oracle 提供的 arm64端口相关的所有源码，即删除目录 open/src/hotspot/cpu/arm 中关于64-bit 的这套实现，只保留其中有关 32-bit ARM端口的实现，余下目录的 open/src/hotspot/cpu/aarch64 代码部分就成了 AArch64 的默认实现。

> 目的
>

这将使开发贡献者将他们的精力集中在单个 64 位 ARM 实现上，并消除维护两套实现所需的重复工作。



### 4、默认生成类的数据共享 CDS 归档文件

> 概述
>

  我们知道在同一个物理机／虚拟机上启动多个JVM时，如果每个虚拟机都单独装载自己需要的所有类，启动成本和内存占用是比较高的。所以Java团队引入了类数据共享机制 (Class Data Sharing ，简称 CDS) 的概念，通过把一些核心类在每个JVM间共享，每个JVM只需要装载自己的应用类即可。好处是：启动时间减少了，另外核心类是共享的，所以JVM的内存占用也减少了。

> 历史版本

1. JDK5 引入了Class-Data Sharing 可以用于多个 JVM 共享 Class，提升启动速度，最早只支持 system classes 及 serial GC。
2. JDK9 对其进行扩展以支持 application classes（自定义Class）及其他GC算法。
3. Java10 的新特性 JEP 310: Application Class-Data Sharing 扩展了 JDK5 引入的 Class-Data Sharing，支持 application的Class-Data Sharing 并开源出来（以前是commercial feature）CDS 只能作用于 BootClassLoader 加载的类，不能作用于 AppClassLoader 或者自定义的 ClassLoader加载的类。在 Java 10 中，则将 CDS 扩展为 AppCDS，顾名思义，AppCDS 不止能够作用于BootClassLoader了，AppClassLoader 和自定义的 ClassLoader 也都能够起作用，大大加大了 CDS 的适用范围。也就说开发自定义的类也可以装载给多个JVM共享了。
4. JDK11 将`-Xshare:off` 改为默认`-Xshare:auto`，以更加方便使用CDS特性

> 迭代效果

  可以说，自 Java 8 以来，在基本 CDS 功能上进行了许多增强、改进，启用 CDS 后应用的启动时间和内存占用量显着减少。使用 Java 11 早期版本在 64 位 Linux 平台上运行 HelloWorld 进行测试，测试结果显示启动时间缩短有 32％，同时在其他 64 位平台上，也有类似或更高的启动性能提升。

> JAVA12 中的新特性

  JDK 12之前，想要利用CDS的用户，即使仅使用JDK中提供的默认类列表，也必须java -Xshare:dump 作为额外的步骤来运行。

  Java 12 针对 64 位平台下的 JDK 构建过程进行了增强改进，使其默认生成类数据共享（CDS）归档，以进一步达到改进应用程序的启动时间的目的，同时也避免了需要手动运行：java -Xshare:dump 的需要，修改后的 JDK 将在${JAVA_HOME}/lib/server 目录中生成一份名为classes.jsa的默认archive文件(大概有18M)方便大家使用。

  当然如果需要，也可以添加其他 GC 参数，来调整堆大小等，以获得更优的内存分布情况，同时用户也可以像之前一样创建自定义的 CDS 存档文件。



### 5、新增项

其他新增项如下：

- Collectors新增teeing方法用于聚合两个downstream的结果

- CompletionStage新增exceptionallyAsync、exceptionallyComposeAsync方法，允许方法体在异步线程执行，同时新增了exceptionallyCompose方法支持在exceptionally的时候构建新的CompletionStage。

- ZGC: Concurrent Class Unloading
  - ZGC在JDK11的时候还不支持class unloading，JDK12对ZGC支持了Concurrent Class Unloading，默认是开启，使用`-XX:-ClassUnloading`可以禁用
- 新增-XX:+ExtensiveErrorReports
  - -XX:+ExtensiveErrorReports可以用于在jvm crash的时候收集更多的报告信息到hs_err.log文件中，product builds中默认是关闭的，要开启的话，需要自己添加-XX:+ExtensiveErrorReports参数
- 新增安全相关的改进
  - 支持java.security.manager系统属性，当设置为disallow的时候，则不使用SecurityManager以提升性能，如果此时调用System.setSecurityManager则会抛出UnsupportedOperationExceptionkeytool新增-groupname选项允许在生成key pair的时候指定一个named group新增PKCS12 KeyStore配置属性用于自定义PKCS12 keystores的生成Java Flight Recorder新增了security-related的event支持ChaCha20 and Poly1305 TLS Cipher Suites
    

### 6、移除项

- 移除com.sun.awt.SecurityWarnin
- 移除FileInputStream、FileOutputStream、Java.util.ZipFile/Inflator/Deflator的finalize方法
- 移除GTE CyberTrust Global Root
- 移除javac的-source, -target对6及1.6的支持，同时移除--release选项



### 7、废弃项

- 废弃的API列表见deprecated-list：https://docs.oracle.com/en/java/javase/12/docs/api/deprecated-list.html
- 废弃-XX:+/-MonitorInUseLists选项
- 废弃Default Keytool的-keyalg值





## 11、参考文献 & 鸣谢

1. Java 新特性文档：https://www.wdbyte.com/java-feature/
2. Java New Feature：https://javaguide.cn/java/new-features/
3. Java 的新特性：https://segmentfault.com/a/1190000004417288/
4. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
5. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/
