# Java 10 新特性

Java 10 在 2018 年 03 月 20 日发布，它不是一个长久支持（LTS）版本。

本次更新共带来 12 个新特性：

- JEP 286: 局部变量类型推断。
- JEP 296: 整合 JDK 代码仓库；
- JEP 304: 统一的垃圾回收接口。
- JEP 307: G1 垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟。
- JEP 310: 应用程序类数据共享。
- JEP 312: 线程-局部管控，线程本地握手。
- JEP 313: 移除 Native-Header 自动生成工具
- JEP 314: 额外的 Unicode 语言标签扩展
- JEP 316: 备用存储装置上的堆分配
- JEP 317: 基于 Java 的 实验性 JIT 编译器
- JEP 319: 根证书认证
- JEP 322: 基于时间的版本发布模式

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 10 新特性：https://openjdk.org/projects/jdk/10/
- OracleJDK Java 10 新特性：https://www.oracle.com/java/technologies/javase/10-relnote-issues.html#NewFeature
- Oracle JDK 10 文档：https://docs.oracle.com/javase/10/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## 1、基于时间发布版本控制

从 Java 10 开始，Oracle 为 Java 版本引入了严格的基于时间的版本控制模型。现在 Java 将在每六个月后发布一个主要版本。Java 10 于 2018 年 3 月发布，以后所有主要版本都计划在未来几年的 3 月和 9 月发布。发布进一步分为三大类。

- 功能发布： 功能发布包含特定于语言的功能、JVM 功能、新/改进的 API、删除/弃用 API。这些功能发布的时间是固定的，并且对要包含在特定版本中的功能没有限制。如果正在开发的功能不是最新版本的一部分，那么它将在下一个版本中计划。
- 更新版本： 更新版本包括错误修复、安全问题修复、回归修复等。每个更新版本计划在 1 月、4 月、7 月和 10 月的每个季度发布。在宣布下一个功能版本之前，每个功能版本将收到两个更新版本。
- 长期支持 (LTS) 版本：从 2018 年 9 月开始，每三年发布一次长期支持版本。Oracle 将在未来三年内为此版本提供支持和更新。此版本主要面向在生产部署中使用 Java 的企业。

查看自己的 Java 10 版本，CMD 中打印 Java 10 版本信息。

```bash
D:\JDK\jdk-10\bin>java --version
openjdk 10 2018-03-20
OpenJDK Runtime Environment 18.3 (build 10+46)
OpenJDK 64-Bit Server VM 18.3 (build 10+46, mixed mode)

C:\Users\jdk-10>java -version
java version "10.0.1" 2018-04-17
Java(TM) SE Runtime Environment 18.3 (build 10.0.1+10)
Java HotSpot(TM) 64-Bit Server VM 18.3 (build 10.0.1+10, mixed mode)
```

JDK 版本格式，现在的版本遵循以下格式。

```shell
$FEATURE.$INTERIM.$UPDATE.$PATCH
```

| 名称     | Remark                                                       |
| -------- | ------------------------------------------------------------ |
| $FEATURE | 此数字表示主要功能发布，每次功能发布后（每6个月）都会增加 1。对于 Java 10，它是 10。(以前为 $MAJOR) |
| $INTERIM | 此数字表示包含错误修复和增强功能的任何非功能、非更新版本。此版本没有任何不兼容的更改、任何 API 删除或对标准 API 的更改。功能发布，将此计数器设为 0。（以前为 $MINOR） |
| $UPDATE  | 此数字表示在功能发布后完成的更新发布。例如，2018 年 4 月的 Java 更新版本是 JDK 10.0.1，2018 年 7 月的更新版本是 JDK 10.0.2 等等。（以前为 $SECURITY） |
| $PATCH   | 只有在需要紧急发布以解决关键问题时，它才会递增。             |

版本号中的数字序列与其他此类序列以数字逐点方式进行比较；例如：10.0.4小于10.1.2。如果一个序列比另一个序列短，则认为较短序列的缺失元素少于较长序列的相应元素；例如：10.0.2小于10.0.2.1。

【操作示例 1】Runtime.version() 可用于以编程方式获取版本计数器值

```java
public class Tester {
    public static void main(String[] args) {
        Runtime.Version version = Runtime.version();
        System.out.printf(" feature: %s%n interim: %s%n update: %s%n patch: %s%n",
                          version.feature(), 
                          version.interim(), 
                          version.update(), 
                          version.patch());
    }
}
```

```
feature: 10
interim: 0
update: 2
patch: 0
```

【操作示例 2】解析现有版本

```java
public class Tester {
    public static void main(String[] args) {
        Runtime.Version version = Runtime.Version.parse("10.0.1");
        System.out.println(version.feature());
        System.out.println(version.interim());
        System.out.println(version.update());
        System.out.println(version.patch());
    }
}
```

```
10
0
1
0
```



## 2、局部变量的类型推断 var

将前端思想 var 关键字引入 Java 后端，自动检测所属于类型，一种情况除外，不能为 null，因为不能判断具体类型，会报异常。

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;

public class Tester {
    public static void main(String[] args) {
        var number = 10;
        var str = "i like java";
        var list = new ArrayList<>();
        var map = new HashMap<>();
        var set = new HashSet<>();

        list.add("test var is list");
        map.put("1", "test var is map");
        set.add("test var is set");
        System.out.println(number);
        System.out.println(str);
        System.out.println(list.toString());
        System.out.println(map.toString());
        System.out.println(set.toString());
    }
}
```

```
10
i like java
[test var is list]
{1=test var is map}
[test var is set]
```

var 可以用于任何类型，基本类型，集合，以及实体类等等都可以，省略了我们在前面写入具体的类型，我们尽管 var，让它自己去判断就好了。

**不能使用类型推断的场景如下**：

1、变量的声明不能使用类型推断，因为无法推断，必须要赋值

```java
// 根据右边的数据推断类型,不赋值压根没给推断的机会,这是错的
var userName;
```

2、初始值为null的情况也是不行的。因为同样没有办法推断

```java
// null值无法推断数据类型,这是不能使用类型推断
var userName = null;
```

3、lambda 表达式某些情况不行

```java
// 这个是可以的
Supplier<Double> supplier = () -> Math.random();
// 这个是不行的 lambda表达式需要显式数据类型,不然无法明确是哪个接口
var supplier2 = () -> Math.random();
```

4、方法引用 某些情况不行

```java
// 这是可以的进行方法引用的
Consumer<String> consumer = System.out::println;
// 这是不可以的,无法明确接口类型
var consumer2 = System.out::println;
```

5、为数组静态初始化

```java
// 动态初始化数据可以进行类型推断
var arr1 = new String[10];
// 静态初始化数组不可以使用类型推断
var arr2 = {"a12","222","333","444","555"};
```

6、类型推断仅仅是局部变量，成员变量不可以使用类型推断

```java
public class Demo07 {
    // 成员变量不能使用类型推断
    var name = "小明";
    public void testMethod1(){
        // 局部变量可以使用类型推断
        var localname = "小明";
    }
}
```

7、其他不可用场景总结

```java
// 情况1 没有初始化的局部变量声明
var s; var x=null;
// 情况2 方法的返回值类型
public var test1()
    // 情况3 方法的参数类型
    public void test2(var a,var b)
    // 情况4 构造器的参数类型
    public Person(var name,var age)
    // 情况5 属性
    class Person{
        var name;
    }
// 情况6 catch块
try{
}catch(var e){
}
```

> **注意**：var 不是一个关键字，我们无需担心变量名或者方法名会与var发生冲突，因为 var 实际上不是一个关键字，而是一个类型名，只有在编译器需要知道类型的地方法才会用到它。除此之外，他就是一个普通的合法标识符。也就是说，除了不能用它做类名，其他都可以。



## 3、集合新增 copyOf() 方法

在 java.util.List、java.util.Set、java.util.Map 新增加了一个静态方法 copyOf，这些方法按照其迭代顺序返回一个不可修改的列表、集合或映射包含了给定的元素的集合。但是如果将返回后的集合继续修改，那么报`java.lang.UnsupportedOperationException`异常。

【操作示例 1】List.copyOf 源码

```java
static <E> List<E> copyOf(Collection<? extends E> coll) {
    return ImmutableCollections.listCopy(coll);
}
```

【操作示例 2】测试 List.copyOf 方法

```java
import java.util.ArrayList;
import java.util.List;

public class Tester {
    public static void main(String[] args) {
        var list = new ArrayList<>();
        list.add("first");
        list.add("second");
        list.add("third");
        var result = List.copyOf(list);
        System.out.println(result.toString()); // [first, second, third]
    }
}
```



## 4、  Collectors 新增 API 收集器 

java.util.stream.Collectors 中新增了静态方法 toUnmodifiableList、toUnmodifiableSet 及 toUnmodifiableMap，用于将流中的元素收集为不可变的集合。

```java
var list = new ArrayList<>();
list.stream().collect(Collectors.toUnmodifiableList());
list.stream().collect(Collectors.toUnmodifiableSet());
```



## 5、新增 Optional.orElseThrow() 方法

Optional 增加了一个新的方法 orElseThrow。调用这个方法也可以获取到 optional 中的 value , 但是如果 value 为 null，就会抛出异常。它现在是get()方法的首选替代方法。

Optional 源码如下：

```java
/**
   * If a value is present, returns the value, otherwise throws
   * {@code NoSuchElementException}.
   *
   * @return the non-{@code null} value described by this {@code Optional}
   * @throws NoSuchElementException if no value is present
   * @since 10
   */
public T orElseThrow() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```

【操作示例 1】

```java
@Test
public void testOrElseThrow(){
    var data = List.of("a","b","c");
    Optional<String> optional = data.stream()
        .filter(s -> s.startsWith("z"))
        .findAny();
    String res = optional.orElseThrow();
    System.out.println(res);
}
```

```java
java.util.NoSuchElementException: No value present

    at java.base/java.util.Optional.orElseThrow(Optional.java:371)
    at com.example.FeatureTest.testOrElseThrow(FeatureTest.java:19)
    at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.base/java.lang.reflect.Method.invoke(Method.java:564)
    at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
    at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
    at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
    at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
    at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
    at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
    at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
    at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
    at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```



## 6、ByteArrayOutputStream 增强

java.io.ByteArrayOutputStream，重载 toString() 方法，通过使用指定的字符集编码字节，将缓冲区的内容转换为字符串，以前是默认没有参数，现在加了一个编码的字符方法。

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.util.Scanner;

public class Tester {
    public static void main(String[] args) throws Exception {
        String str = "我喜欢java";
        ByteArrayInputStream bis = new ByteArrayInputStream(str.getBytes("utf-8"));
        ByteArrayOutputStream bos = new ByteArrayOutputStream();

        int c;
        while ((c = bis.read()) != -1) {
            bos.write(c);
        }
        // bos.toString() 默认的使用的UTF-8编码
        System.out.println(bostoString());
    }
}
```

```
我喜欢java
```

这个是我们正常的格式，但是当我们修改代码，str.getBytes ("gbk") 的时候，创建的 txt 为乱码，那么我们使用 toString () 的新增方法，替换 gbk。

```
��ϲ��java
```



## 7、PrintStream、PrintWriter 新增 API

java.io.PrintStream,Java.io.PrintWriter，这两个类都有三个新的构造方法，他们需要而外的参数 charset。

```java
import java.io.PrintWriter;

public class Tester {
    public static void main(String[] args) throws Exception {
        String str = "我也特别喜欢java";
        var pri  = new PrintWriter("D:\\test_out.txt");
        pri.println(str);
        pri.flush();
        pri.close();
    }
}
```

文件中输出如下内容：

```
我也特别喜欢java
```

但是我们这个 txt 默认的都是 utf-8 格式的，如果突然改成 gbk 的话，格式会不匹配出现乱码，毕竟 txt 文件格式要和输出的格式一致才可以，这是我们可以使用一下。 我们直接在后面添加一个格式参数就可以直接解决啦

```java
var pri = new PrintWriter("D:\\test_out.txt","gbk");
```



## 8、Reader:transferTo 新增成员方法

java.io.Reader:transferTo 从这个 Reader 中读取所有字符串，并按照所读取的顺序将字符串写入给指定的 Writer。

```java
import java.io.*;

public class Tester {
    public static void main(String[] args) throws Exception {
        var reader = new BufferedReader(new InputStreamReader(new FileInputStream("D:\\test.txt"), "utf-8"));
        var writer = new PrintWriter("D:\\test_out.txt");
        reader.transferTo(writer);
        p.flush();
        p.close();
        reader.close();
    }
}
// 最后test.txt中的内容就输出到test_out.txt文件中了
```



## 9、Formatter、Scanner 新增构造方法

 java.util.Formatter、java.util.Scanner 新增三个构造方法，除了其他参数之外，都需要一个 charset 参数，将 11.txt 内容替换成这样

```
hellonword !!!
```

```java
import java.io.FileInputStream;
import java.util.Scanner;

public class Tester {
    public static void main(String[] args) throws Exception{
        var scan = new Scanner(new FileInputStream("D:\\test.txt"), "utf-8");
        scan.useDelimiter(" |,");
        while (scan.hasNext()) {
            System.out.println(scan.next());
        }
    }
}
```

scan.useDelimiter ("|,"); 这是空格或者‘,’ 都要分割，如下结果为：

```
hello
word
!!!

```



## 10、java.util.Properties 新增 API 

增加了一个新的构造函数，它接受一个 int 参数。这将创建一个没有默认值的空属性列表，并且指定初始大小以容纳指定的元素数量，而无需动态调整大小。还有一个新的重载的 replace 方法，接受三个 Object 参数并返回一个布尔值。只有在当前映射到指定值时，才会替换指定键的条目。



## 11、整合 JDK 代码仓库

为了简化开发流程，Java 10 中会将多个代码库合并到一个代码仓库中。

在 Java 9 中，有八个基于模块的目录，称为 repos：root、corba、hotspot、jaxp、jaxws、jdk、langtools、nashorn。

代码的组织方式如下 ：

```bash
$ROOT/jdk/src/java.base
...
$ROOT/langtools/src/java.compiler
...
```

从 Java 10 开始，将 JDK 多个代码仓库合并到一个仓库。现在的结构为 ：

```bash
$ROOT/src/java.base
$ROOT/src/java.compiler
...
```



## 12、垃圾回收器接口

在当前的 Java 结构中，组成垃圾回收器（GC）实现的组件分散在代码库的各个部分。尽管这些惯例对于使用 GC 计划的 JDK 开发者来说比较熟悉，但对新的开发人员来说，对于在哪里查找特定 GC 的源代码，或者实现一个新的垃圾收集器常常会感到困惑。更重要的是，随着 Java modules 的出现，我们希望在构建过程中排除不需要的 GC，但是当前 GC 接口的横向结构会给排除、定位问题带来困难

为解决此问题，需要整合并清理 GC 接口，以便更容易地实现新的 GC，并更好地维护现有的 GC。Java 10 中，hotspot/gc 代码实现方面，引入一个干净的 GC 接口，改进不同 GC 源代码的隔离性，多个 GC 之间共享的实现细节代码应该存在于辅助类中。这种方式提供了足够的灵活性来实现全新 GC 接口，同时允许以混合搭配方式重复使用现有代码，并且能够保持代码更加干净、整洁，便于排查收集器问题。

这纯粹是重构。之前工作的一切都需要事后工作，性能不应该倒退。



## 13、G1 并行 Full GC

从 Java 9 开始 G1 就了默认的垃圾回收器，G1 是以一种低延时的垃圾回收器来设计的，旨在避免进行 Full GC,但是 Java9 的 G1 的 FullGC 依然是使用单线程去完成标记清除算法,这可能会导致垃圾回收期在无法回收内存的时候触发 Full GC。

为了最大限度地减少 Full GC 造成的应用停顿的影响，从 Java10 开始，G1 的 FullGC 改为并行的标记清除算法，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。

Java 10 中将采用并行化 mark-sweep-compact 算法，并使用与年轻代回收和混合回收相同数量的线程。具体并行 GC 线程数量可以通过： -XX：ParallelGCThreads 参数来调节，但这也会影响用于年轻代和混合收集的工作线程数



## 14、应用程序类数据共享

当 JVM 启动时，它会将类加载到内存中作为初步步骤。如果有多个具有多个类的 jar，则第一个请求会出现明显的滞后。在无服务器架构中，这种延迟会延迟启动时间，这是此类架构中的关键操作。应用程序类数据共享概念有助于减少应用程序的启动时间。Java 具有现有的 CDS（类数据共享）功能。通过应用程序类数据共享，Java 10 允许将应用程序类放在共享存档中。这通过跨多个 Java 进程共享公共类元数据来减少应用程序启动和占用空间。

应用程序类数据共享 一共 有 3 个步骤。

- 创建要存档的类列表：使用 Java 启动器创建位于welcome.jar 中的Greeting.java 类的列表welcome.lst。

  ```bash
  $ java -Xshare:off -XX:+UseAppCDS -XX:DumpLoadedClassList=welcome.lst -cp welcome.jar Greeting
  ```

- 创建 AppCDS 存档： 存档用于应用程序类数据共享的类列表。

  ```bash
  $ java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=welcome.lst -XX:SharedArchiveFile=welcome.jsa -cp welcome.jar
  ```

- 使用 AppCDS 存档：使用 Java 启动器时使用 AppCDS 存档。

  ```bash
  $ java -Xshare:on -XX:+UseAppCDS -XX:SharedArchiveFile=welcome.jsa -cp welcome.jar Greeting
  ```



## 15、Thread-Local 握手

在已有的 Java 版本中，JVM 线程只能全部启用或者停止，没法做到对单独某个线程的操作。为了能够对单独的某个线程进行操作，Java 10 中线程管控引入 JVM 安全点的概念，将允许在不运行全局 JVM 安全点的情况下实现线程回调，由线程本身或者 JVM 线程来执行，同时保持线程处于阻塞状态，这种方式使得停止单个线程变成可能，而不是只能启用或停止所有线程。通过这种方式显著地提高了现有 JVM 功能的性能开销，并且改变了到达 JVM 全局安全点的现有时间语义

在 Java 10 中，为 JVM 引入了一个新选项 -XX:ThreadLocalHandshakes。此选项仅适用于基于 x64 和 SPARC 的机器。

此选项可用于提高 VM 性能。它允许在不创建全局 VM 安全点的情况下对应用程序线程进行回调。因此允许 JVM 停止单个线程而不停止所有线程。

由于此选项并非适用于所有平台，因此其他平台将回退到正常的安全点。



## 16、移除 Native-Header 自动生成工具

自 Java 9 以来便开始了一些对 JDK 的调整，用户每次调用 javah 工具时会被警告该工具在未来的版本中将会执行的删除操作。当编译 JNI 代码时，已不再需要单独的 Native-Header 工具来生成头文件，因为这可以通过 Java 8（JDK-7150368）中添加的 javac 来完成。在未来的某一时刻，JNI 将会被 Panama 项目的结果取代，但是何时发生还没有具体时间表。

这是 Java 10 的另一个专注于内务管理的特性。它将 javah 从 JDK 中删除该工具，这是一个在编译 JNI 代码时生成头文件的单独工具，因为这可以通过 javac。



## 17、额外的 Unicode 语言标签扩展

自 Java 7 开始支持 BCP 47 语言标记以来， JDK 中便增加了与日历和数字相关的 Unicode 区域设置扩展，在 Java 9 中，新增支持 ca 和 nu 两种语言标签扩展。而在 Java 10 中将继续增加 Unicode 语言标签扩展，具体为：增强 java.util.Locale 类及其相关的 API，以更方便的获得所需要的语言地域环境信息。同时在这次升级中还带来了如下扩展支持。

Unicode 扩展表

| 编码 | 注释         |
| :--- | :----------- |
| cu   | 货币类型     |
| fw   | 一周的第一天 |
| rg   | 区域覆盖     |
| tz   | 时区         |

如 Java 10 加入的一个方法，Unicode 语言标签扩展示例：`java.time.format.DateTimeFormatter::localizedBy`

通过这个方法，可以采用某种数字样式，区域定义或者时区来获得时间信息所需的语言地域本地环境信息

修改后的相关API有：

```java
java.text.DateFormat::get*Instance
java.text.DateFormatSymbols::getInstance
java.text.DecimalFormatSymbols::getInstance
java.text.NumberFormat::get*Instance
java.time.format.DateTimeFormatter::localizedBy
java.time.format.DateTimeFormatterBuilder::getLocalizedDateTimePattern
java.time.format.DecimalStyle::of
java.time.temporal.WeekFields::of
java.util.Calendar::{getFirstDayOfWeek,getMinimalDaysInWeek}
java.util.Currency::getInstance
java.util.Locale::getDisplayName
java.util.spi.LocaleNameProvider
```



## 18、备用存储装置上的堆分配

通过 Java 10 中的这一增强，现在用户可以指定替代内存设备，例如 NV-DIMM 到 HotSpot VM 来分配 Java 堆空间。用户需要使用新选项 -XX:AllocateHeapAt 将路径传递到文件系统。

```bash
-XX：AllocateHeapAt = <path> 
-XX:AllocateHeapAt=~/etc/heap
```

此选项采用文件路径并进行内存映射以达到所需的结果。其他堆标志如 -Xmx、-Xms 继续工作。



## 19、基于 Java 的 实验性 JIT 编译器

JIT 编译器是用 C++ 编写的，用于将 Java 转换为字节码。现在 Java 10 可以选择启用基于 Java 的实验性 JIT 编译器 Graal 来代替标准的 JIT 编译器。Graal 正在使用 Java 9 中引入的 JVMCI，即 JVM 编译器接口。 Graal 在 Java 9 中也可用。使用 Java 10，我们可以启用 Graal 来测试和调试实验性 JVM 编译器。

1、JIT编译器语法：

```bash
$ java -XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler
```

Graal 编译器完全重写了基于 C++ 的早期编译器，针对基于 Linux/x64 的平台。Graal 是在 Java 9 中引入的，作为目前使用的 JIT 编译器的替代品。Graal 是 JVM 的插件，可以动态插入。它也支持多语言解释。

2、JIT 编译器的风险和假设：

由于 Graal 是实验性的，并且考虑到各种 Hotspots 和带有各种标志选项的 jdk 测试，因此需要进行测试工作。与标准的 JIT Ahead of Time 编译器相比，它可能无法通过一些性能基准测试。



## 20、根证书认证

Cacerts 存储，在 Java 10 之前是一个空集。它需要包含一组证书，可用于在供应商的各种安全协议的证书链中建立信任。

OpenJDK 构建没有这样的证书，这就是 TLS 等关键安全组件在默认构建中不起作用的原因。

现在，由于 Oracle 已经使用 Oracle JAVA SE Root CA 程序开源了根证书，OpenJDK 构建现在可以拥有根证书，从而可以减少 OpenJDK 和 Oracle JDK 之间的差异。

Oracle JAVA SE Root CA 程序颁发根证书。签署协议的供应商包含在一组根证书中。未注册的供应商将包含在下一个版本中。



## 21、新增与删除 API

Java 10 中添加了 73 个新的 API。让我们来看看其中的几个：

1. Optional.orElseThrow()：Optional 类中添加了一个新方法 orElseThrow。它是现有方法的同义词，现在是现有方法的首选替代get方法。
2. ``  ``List.copyOf, Set.copyOf, 和Map.copyOf：这些方法从现有实例创建新的集合实例。
3. Collectors.toUnmodifiableList, Collectors.toUnmodifiableSet,Collectors.toUnmodifiableMap：这些方法允许将 a 的元素Stream收集到不可修改的集合中。
4. --jdk.disableLastUsageTracking：为正在运行的 VM 禁用 JRE 上次使用跟踪。
5. --add-stylesheet：支持在生成的文档中使用多个样式表。
6. --main-stylesheet：帮助区分主样式表和任何附加样式表。
7. `@summary`标签：添加以明确指定用作 API 描述摘要的文本。默认情况下，API 描述的摘要是从第一句话推断出来的。

 Java 10 中 删除的 API：

1. LookAndFeels
2. Runtime.getLocalizedInputStream`,`Runtime.getLocalizedOutputStream：过时的国际化机制的一部分，没有已知用途。
3. RMI 服务器端多路复用协议支持：它在 JDK 9 中被禁用，现在已被删除。
4. 常见的 DOM API：和API 已被删除`com.sun.java.browser.plugin2.DOM`。`sun.plugin.dom.DOMObject`应用程序可以`netscape.javascript.JSObject`用来操作 DOM。
5. 平面轮廓仪：在 JDK 9 中已弃用，已通过删除实现代码而过时。
6. -Xoss`, `-Xsqnopause`, `-Xoptimize`, `-Xboundthreads, 和-Xusealtsigs：已删除选项。
7. policytool：policytoo l安全工具已从 JDK 中删除。
8. 不推荐使用的类 com.sun.security.auth.**：com.sun.security.auth.PolicyFile``com.sun.security.auth.SolarisNumericGroupPrincipal``com.sun.security.auth.SolarisNumericUserPrincipal``com.sun.security.auth.SolarisPrincipal``com.sun.security.auth.X500Principal``com.sun.security.auth.module.SolarisLoginModule``com.sun.security.auth.module.SolarisSystem

总体而言，Java 10 有许多我们可能不会在日常编程中使用的特性，但它仍然有许多在幕后工作的特性使其成为重要的里程碑。



## 22、参考文献 & 鸣谢

1. Java 10 新特性概述：https://blog.csdn.net/vincent_wen0766/article/details/114434865
2. Java10教程：http://www.yiidian.com/java10/java10-root-certificate.html
3. Java 新特性文档：https://www.wdbyte.com/java-feature/
4. Java New Feature：https://javaguide.cn/java/new-features/
5. Java 的新特性：https://segmentfault.com/a/1190000004417288/
6. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
7. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/