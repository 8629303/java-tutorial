# Java 16 新特性

Java 16 在 2021 年 3 月 16 日正式发布，它不是一个长久支持（LTS）版本。

这次更新共带来 17 个新特性（此次更新没有很多语法改动，也带来了不少新的实用功能）：

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
- 394: Pattern Matching for instanceof：instanceof 匹配模式转正
- 395: Records：记录转正
- 396: Strongly Encapsulate JDK Internals by Default：默认情况下对JDK内部进行强封装
- 397: Sealed Classes (Second Preview)：密封类(第二次预览)

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 16 新特性：https://openjdk.org/projects/jdk/16/
- OracleJDK Java 16 新特性：https://www.oracle.com/java/technologies/javase/16-relnote-issues.html#NewFeature
- Oracle JDK 16 文档：https://docs.oracle.com/en/java/javase/16/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## X、Stream 新增API toList & mapMulti

### 1、Stream .toList()

Java 16 中对 Stream 类新增了一个 toList 方法，其实就是 collect(Collectors.toList()) 替换方案。如下是 Stream.toList() 源码：

```java
/**
 * @since 16
 */
default List<T> toList() {
    return (List<T>) Collections.unmodifiableList(new ArrayList<>(Arrays.asList(this.toArray())));
}
```

使用方式如下：

```java
List<Integer> list = List.of(10, 20, 30, 30);
// Java 16 新增转List方法
List<Integer> toList = list.stream().toList();
// Java 8 转换List方法
List<Integer> collect = list.stream().collect(Collectors.toList());
```



### 2、Stream.toList() 和 Collectors.toList() 性能对比

Stream 转 List 番外篇：Stream.toList() 和 Collectors.toList() 哪个性能更好

主要涉及下面这几种转换方式：

- Stream.toList()：返回的List是不可变List，不能增删改
- Collectors.toList()：返回的是个普通的List，可以增删改
- Collectors.toUnmodifiableList()：返回的List是不可变List，不能增删改

```java
list.stream().toList();
list.stream().collect(Collectors.toList());
list.stream().collect(Collectors.toUnmodifiableList());
```

而至于性能的话，今天我们就来测试一下，看看哪个性能更好。

```java
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.results.format.ResultFormatType;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

@BenchmarkMode(Mode.All)
@State(Scope.Thread)
@Fork(1)
@Warmup(iterations = 20, time = 1, batchSize = 10000)
@Measurement(iterations = 20, time = 1, batchSize = 10000)
public class BenchmarkTest {
 
    @Benchmark
    public List<Integer> streamToList() {
        return IntStream.range(1, 1000).boxed().toList();
    }
 
    @Benchmark
    public List<Integer> collectorsToList() {
        return IntStream.range(1, 1000).boxed().collect(Collectors.toList());
    }
 
    @Benchmark
    public List<Integer> streamToUnmodifiableList() {
        return IntStream.range(1, 1000).boxed().collect(Collectors.toUnmodifiableList());
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(BenchmarkTest.class.getSimpleName())
                .resultFormat(ResultFormatType.CSV)
                .result("report.csv")
                .build();
        new Runner(opt).run();
    }
}
```

结果报告：

```
Benchmark                                                                    Mode  Cnt   Score    Error  Units
BenchmarkTest.collectorsToList                                              thrpt   20  16.888 ±  0.071  ops/s
BenchmarkTest.streamToList                                                  thrpt   20  28.563 ±  0.322  ops/s
BenchmarkTest.streamTooUnmodifiableList                                     thrpt   20  16.078 ±  0.038  ops/s
BenchmarkTest.collectorsToList                                               avgt   20   0.057 ±  0.001   s/op
BenchmarkTest.streamToList                                                   avgt   20   0.035 ±  0.001   s/op
BenchmarkTest.streamTooUnmodifiableList                                      avgt   20   0.067 ±  0.001   s/op
BenchmarkTest.collectorsToList                                             sample  340   0.061 ±  0.001   s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.00                      sample        0.056            s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.50                      sample        0.060            s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.90                      sample        0.064            s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.95                      sample        0.066            s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.99                      sample        0.073            s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.999                     sample        0.076            s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.9999                    sample        0.076            s/op
BenchmarkTest.collectorsToList:collectorsToList·p1.00                      sample        0.076            s/op
BenchmarkTest.streamToList                                                 sample  577   0.035 ±  0.001   s/op
BenchmarkTest.streamToList:streamToList·p0.00                              sample        0.033            s/op
BenchmarkTest.streamToList:streamToList·p0.50                              sample        0.035            s/op
BenchmarkTest.streamToList:streamToList·p0.90                              sample        0.037            s/op
BenchmarkTest.streamToList:streamToList·p0.95                              sample        0.038            s/op
BenchmarkTest.streamToList:streamToList·p0.99                              sample        0.041            s/op
BenchmarkTest.streamToList:streamToList·p0.999                             sample        0.046            s/op
BenchmarkTest.streamToList:streamToList·p0.9999                            sample        0.046            s/op
BenchmarkTest.streamToList:streamToList·p1.00                              sample        0.046            s/op
BenchmarkTest.streamToUnmodifiableList                                     sample  320   0.065 ±  0.001   s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p0.00      sample        0.061            s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p0.50      sample        0.065            s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p0.90      sample        0.068            s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p0.95      sample        0.069            s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p0.99      sample        0.071            s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p0.999     sample        0.076            s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p0.9999    sample        0.076            s/op
BenchmarkTest.streamToUnmodifiableList:streamToUnmodifiableList·p1.00      sample        0.076            s/op
BenchmarkTest.collectorsToList                                                 ss   20   0.061 ±  0.001   s/op
BenchmarkTest.streamToList                                                     ss   20   0.038 ±  0.008   s/op
BenchmarkTest.streamToUnmodifiableList                                         ss   20   0.065 ±  0.001   s/op
```

从报告中我们可以看到：

- 吞吐量：streamToList > collectorsToList > collectorsToUnmodifiableList

- 平均耗时：streamToList > collectorsToList > collectorsToUnmodifiableList

- p0.9999耗时：streamToList > collectorsToUnmodifiableList > collectorsToList

- 冷启动耗时：streamToList > collectorsToList > collectorsToUnmodifiableList

所以，Stream.toList() 的性能要各方面都要好于Collectors.toList() 和 Collectors.toUnmodifiableList()。

我们再放大一些数据量，试试：

```java
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.results.format.ResultFormatType;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

@BenchmarkMode(Mode.All)
@State(Scope.Thread)
@Fork(1)
@Warmup(iterations = 20, time = 1, batchSize = 10000)
@Measurement(iterations = 20, time = 1, batchSize = 10000)
public class BenchmarkStreamToList {
 
    @Benchmark
    public List<Integer> streamToList() {
        return IntStream.range(1, 10000).boxed().toList();
    }
 
    @Benchmark
    public List<Integer> collectorsToList() {
        return IntStream.range(1, 10000).boxed().collect(Collectors.toList());
    }
 
    @Benchmark
    public List<Integer> streamTooUnmodifiableList() {
        return IntStream.range(1, 10000).boxed().collect(Collectors.toUnmodifiableList());
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(BenchmarkStreamToList.class.getSimpleName())
                .build();
        new Runner(opt).run();
    }
}
```

结果报告：

```
Benchmark                                                                          Mode  Cnt  Score   Error  Units
BenchmarkTest.collectorsToList                                                    thrpt   20  2.186 ± 0.162  ops/s
BenchmarkTest.collectorsToUnmodifiableList                                        thrpt   20  2.184 ± 0.042  ops/s
BenchmarkTest.streamToList                                                        thrpt   20  3.538 ± 0.058  ops/s
BenchmarkTest.collectorsToList                                                     avgt   20  0.426 ± 0.004   s/op
BenchmarkTest.collectorsToUnmodifiableList                                         avgt   20  0.469 ± 0.016   s/op
BenchmarkTest.streamToList                                                         avgt   20  0.293 ± 0.008   s/op
BenchmarkTest.collectorsToList                                                   sample   58  0.448 ± 0.049   s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.00                            sample       0.414           s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.50                            sample       0.422           s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.90                            sample       0.458           s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.95                            sample       0.560           s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.99                            sample       1.160           s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.999                           sample       1.160           s/op
BenchmarkTest.collectorsToList:collectorsToList·p0.9999                          sample       1.160           s/op
BenchmarkTest.collectorsToList:collectorsToList·p1.00                            sample       1.160           s/op
BenchmarkTest.collectorsToUnmodifiableList                                       sample   60  0.458 ± 0.004   s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p0.00    sample       0.447           s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p0.50    sample       0.455           s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p0.90    sample       0.471           s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p0.95    sample       0.482           s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p0.99    sample       0.492           s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p0.999   sample       0.492           s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p0.9999  sample       0.492           s/op
BenchmarkTest.collectorsToUnmodifiableList:collectorsToUnmodifiableList·p1.00    sample       0.492           s/op
BenchmarkTest.streamToList                                                       sample   78  0.293 ± 0.012   s/op
BenchmarkTest.streamToList:streamToList·p0.00                                    sample       0.277           s/op
BenchmarkTest.streamToList:streamToList·p0.50                                    sample       0.284           s/op
BenchmarkTest.streamToList:streamToList·p0.90                                    sample       0.309           s/op
BenchmarkTest.streamToList:streamToList·p0.95                                    sample       0.377           s/op
BenchmarkTest.streamToList:streamToList·p0.99                                    sample       0.459           s/op
BenchmarkTest.streamToList:streamToList·p0.999                                   sample       0.459           s/op
BenchmarkTest.streamToList:streamToList·p0.9999                                  sample       0.459           s/op
BenchmarkTest.streamToList:streamToList·p1.00                                    sample       0.459           s/op
BenchmarkTest.collectorsToList                                                       ss   20  0.474 ± 0.133   s/op
BenchmarkTest.collectorsToUnmodifiableList                                           ss   20  0.493 ± 0.099   s/op
BenchmarkTest.streamToList                                                           ss   20  0.325 ± 0.056   s/op
```

从报告中我们可以看到

- 吞吐量：streamToList > collectorsToList > collectorsToUnmodifiableList

- 平均耗时：streamToList > collectorsToList > collectorsToUnmodifiableList

- p0.9999耗时：streamToList > collectorsToUnmodifiableList > collectorsToList

- 冷启动耗时：streamToList > collectorsToList > collectorsToUnmodifiableList


所以，即使集合内的元素增大，Stream.toList() 的性能在各方面依然都要好于 Collectors 下的方法。



### 3、Stream.mapMulti()

> 作者：张哈希，来源：https://segmentfault.com/a/1190000041226351

mapMulti 其实主要是对现有的 flatMap 在某些场景使用起来不够合适的补充。flatMap 是将一个对象映射为多个对象之后继续 Stream，例如将 `List<List<Integer>>` 里面的每一个数字取出，转换成一个新的 `List<Integer>`：

```java
integerLists.stream().flatMap(integers -> integers.stream()).collect(Collectors.toList());
```

这对于每个元素本身就是集合类型的场景来说，非常适用。我们再来看一个例子，假设有邮件这个 Record 类，包含 id，以及发送到的邮箱和抄送到的邮箱：

```java
record Mail(int id, Set<String> sendTo, Set<String> cc) {}
```

我们想找到一批邮件的所有不同的联系人，最后放到一个 List 中，可能会这么写：

```java
Set<String> collect = mails.stream().flatMap(mail -> {
    Set<String> result = new HashSet<>();
    result.addAll(mail.sendTo());
    result.addAll(mail.cc());
    return result.stream();
}).collect(Collectors.toSet());
```

但是，这样写显然很不优雅，首先是对于**每一个 Mail 都创建了额外的 Set 和对应的 Stream**，并且，对于**每个 mail 的 sendTo 还有 cc 都遍历了两遍**（addAll 一遍，后续 Stream 又一遍）。其实我们的目前只是将 mail 中的 cc 以及 sendTo 取出来，用于参与后续的 Stream。在这种场景下，就非常适合用 mapMulti：

```java
Set<String> collect = mails.stream().<String>mapMulti((mail, consumer) -> {
    mail.cc().forEach(consumer::accept);
    mail.sendTo().forEach(consumer::accept);
}).collect(Collectors.toSet());
```

可以看出：

- mapMulti 的入参是一个 BiConsumer，其实就是**使用其参数中的 consumer 接收参与 Stream 后续的对象**
- mapMulti 的思路就是将参数中的**需要参与后续 Stream 的对象传入 consumer 来继续 Stream**
- consumer **没有限制对象类型**，想要限制必须加上形参 `<String>` 否则最后返回的是 `Set<Object>` 而不是 `Set<String>`

假设 mail 的 sendTo 还有 cc 都需要去其他地方获取，使用 mapMulti 还可以实现：

```java
Set<String> collect = mailIds.stream().<String>mapMulti((mailId, consumer) -> {
    mailService.getCCById(mailId).forEach(consumer::accept);
    mailService.getSendToById(mailId).forEach(consumer::accept);
}).collect(Collectors.toSet());
```

还有一些比较有意思的用法，例如混合类型的 List 转换成统一类型：

```java
class C {
    static void expandIterable(Object e, Consumer<Object> c) {
        if (e instanceof Iterable<?> elements) {
            for (Object ie : elements) {
                expandIterable(ie, c);
            }
        } else if (e != null) {
            c.accept(e);
        }
    }

    public static void main(String[] args) {
        var nestedList = List.of(1, List.of(2, List.of(3, 4)), 5);
        Stream<Object> expandedStream = nestedList.stream().mapMulti(C::expandIterable);
    }
}
```

活用 `Optional.ifPresent(Consumer<? super T> action)` 方法：

```java
Stream.of(Optional.of("0"), Optional.of("1"), Optional.empty())
    .mapMulti(Optional::ifPresent)
    .forEach(System.out::print);
```



## Y、java.time 新增格式化模板

java.time 新增可以根据时段获取时间

```java
/**
 * B 格式化模式 为 Java 16 新增, 具体可以参考 DateTimeFormatter.java
 */
String date1 = DateTimeFormatter.ofPattern("a").format(LocalTime.of(19, 20));
String date2 = DateTimeFormatter.ofPattern("B").format(LocalTime.of(19, 20));
String date3 = DateTimeFormatter.ofPattern("k").format(LocalTime.of(19, 20));
System.out.println(date1);  // 下午
System.out.println(date2);  // 晚上
System.out.println(date3);  // 19
String date4 = DateTimeFormatter.ofPattern("a").format(LocalTime.of(12, 20));
String date5 = DateTimeFormatter.ofPattern("B").format(LocalTime.of(12, 20));
String date6 = DateTimeFormatter.ofPattern("k").format(LocalTime.of(12, 20));
System.out.println(date4);  // 下午
System.out.println(date5);  // 中午
System.out.println(date6);  // 12
String date7 = DateTimeFormatter.ofPattern("a").format(LocalTime.of(11, 20));
String date8 = DateTimeFormatter.ofPattern("B").format(LocalTime.of(11, 20));
String date9 = DateTimeFormatter.ofPattern("k").format(LocalTime.of(11, 20));
System.out.println(date7);  // 上午
System.out.println(date8);  // 上午
System.out.println(date9);  // 11
```



## 1、JEP 397：密封类【二次预览】

Sealed Classes 再次预览，在 Java 15 新特性中引入，使用详情请参考 Java 15 新特性教程。，在 Java 16 功能没有任何更改（更加严格的引用检查和密封类的继承关系），用于接受更多的使用反馈，并且会在 Java 17 中成为正式版（且在 Java 17 中也没有任何更改）。

密封类的优点：虽然我们可以通过 final 来限定子类继承，但是这是绝对杜绝类子类，而类的密封是允许子类，但是限定是那个或者哪些。使其变得更加灵活。

简单回顾 Java 15 新特性之 Sealed Classes 密封类：

```java
/**
 * 犬类（Dog）只能被牧羊犬（Collie）和田园犬（TuGou）继承，使用 sealed 关键字。
 */
sealed interface Dog permits Collie, TuGou {}

/**
 * 牧羊犬（Collie）只能被边境牧羊犬（BorderCollie）继承。
 */
sealed class Collie implements Dog permits BorderCollie {}

/**
 * 边境牧羊犬（BorderCollie）不能被继承，使用 final 关键字。
 */
final class BorderCollie extends Collie{}

/**
 * 田园犬（ToGou）可以被任意继承，使用 non-sealed 关键字。
 */
non-sealed class TuGou implements Dog {}

/**
 * sealed 对类进行密封
 * permits 指明哪些类可以继承
 * 子类必须是final修饰的或者也是密封类(sealed)
 * 如果子类不想被密封,可以使用non-sealed修饰
 * <p>
 * final不能修饰接口:
 * - 密封接口在被继承时,子接口要么使用 sealed、non-sealed 修饰
 * - 密封类在被继承时,子类要么使用 final、sealed、non-sealed 修饰
 * <p>
 * 密封接口不可以使用匿名内部类进行实现
 */
```



## 2、JEP 394：instanceof 模式匹配【正式版】

instanceof 改进在 Java 14 中已经提出预览，在 Java 15 中继续预览，而现在，在 Java 16 中成为正式功能。

| JDK 版本   | 更新类型          | JEP                                     | 更新内容                               |
| ---------- | ----------------- | --------------------------------------- | -------------------------------------- |
| Java SE 14 | preview           | [JEP 305](https://openjdk.org/jeps/305) | 首次引入 instanceof 模式匹配           |
| Java SE 15 | Second Preview    | [JEP 375](https://openjdk.org/jeps/375) | 相比较上个版本无变化，继续收集更多反馈 |
| Java SE 16 | Permanent Release | [JEP 394](https://openjdk.org/jeps/394) | 模式变量不在隐式为 final               |

从 Java 16 开始，你可以正式使用对 instanceof 中的变量值进行修改。

```java
// Old code
if (o instanceof String) {
    String s = (String)o;
    // ... use s ...
}

// New code
if (o instanceof String s) {
    // ... use s ...
}
```



## 3、JEP 395：record 关键字【正式版】

### 1、Record 发展历史

Record 成为 Java 16 的正式功能，Record 是 Java 14 时提出的，record 是一种全新的类型，它本质上是一个 final 类，同时所有的属性都是 final 修饰，它会自动编译出 hashcode 、equals、toString 等方法，减少了代码编写量。

如下是 Record 的发展历史：

| JDK 版本   | 更新类型          | JEP                                          | 更新内容                                                     |
| ---------- | ----------------- | -------------------------------------------- | ------------------------------------------------------------ |
| Java SE 14 | Preview           | [JEP 359](https://openjdk.java.net/jeps/359) | 引入 record 关键字，record 提供一种紧凑的语法来定义类中的不可变数据 |
| Java SE 15 | Second Preview    | [JEP 384](https://openjdk.org/jeps/384)      | 支持在局部方法和接口中使用 record                            |
| Java SE 16 | Permanent Release | [JEP 395](https://openjdk.org/jeps/395)      | 非静态内部类可以定义非常量的静态成员                         |

1. 记录由 JEP 359 提出 并在 Java 14 中作为预览功能提供。
2. 作为对反馈的回应，JEP 384 对该设计进行了改进， 并在 Java 15 中作为第二次预览功能交付 。第二次预览的改进如下：
   1. 在第一个预览版中，规范构造函数必须是 public。在第二个预览中，如果隐式声明了规范构造函数，则其访问修饰符与记录类相同；如果显式声明了规范构造函数，则其访问修饰符必须提供至少与记录类一样多的访问权限。
   2. @Override注释的含义被扩展为包括注释方法是记录组件的显式声明的访问器方法的情况。
   3. 为了强制使用紧凑构造函数，分配给构造函数主体中的任何实例字段会导致编译时错误。
   4. 引入了声明本地记录类、本地枚举类和本地接口的能力。
3. 该 JEP 建议在 JDK 16 中完成该功能，并进行以下改进：
   1. 放宽长期存在的限制，即内部类不能声明显式或隐式静态成员。这将变得合法，特别是将允许内部类声明作为记录类的成员。



### 2、Record 类的声明

1、Record 可以**单独作为一个文件的顶级类**，即：User.java 文件：

```java
public record UserRecord(long id, String name, int age) {}
```

2、也可以**作为一个成员类**，即：

```java
public class RecordTest {
    public record UserRecord(long id, String name, int age) {}
}
```

3、也可以**作为一个本地类**，即：

```java
public class RecordTest {
    public void test() {
        record Mail (long id, String content) {}
        Mail mail = new Mail(10, "content");
    }
}
```

**不能用 abstract 修饰 Record 类**，会有编译错误。可以用 final 修饰 Record 类，但是没有必要的，因为 **Record 类本身就是 final 的**。

也可以用 static 修饰，但是没有必要。**成员 Record 类，还有本地 Record 类，本身就是 static 的**。

和普通类一样，Record 类可以被 public、protected、private 修饰，也可以不带这些修饰，这样就是 package-private 的。

和一般类不同的是，Record 类的直接父类不是 java.lang.Object 而是 **java.lang.Record**。但是，**Record 类不能使用 extends**，因为 Record 类不能继承任何类。Record 类允许你实现接口，无论是单个接口还是多个接口。



### 3、Record 类的属性

1、正常情况，**在 Record 类声明头部指定这个 Record 类有哪些属性**：

```java
public record UserRecord(long id, String name, int age) {}
```

2、Record **类属性**必须在头部声明，**在 Record 类体只能声明静态属性**：

```java
public record UserRecord(long id, String name, int age) {
    static long anotherId;
}
```

3、另外 Record 类也可以在头部的属性列表中运用注解：

```java
@Target({ ElementType.RECORD_COMPONENT})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidA {}

@Target({ ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidB {}

public record User(@ValidA @ValidB long id, String name, int age) {}
```

但是，需要注意一点，这里通过反射获取 id 的注解的时候，需要通过对应的方式进行获取，否则获取不到，即 ElementType.FIELD 通过 Field 获取，ElementType.RECORD_COMPONENT 通过 RecordComponent 获取：

```java
Field[] fields = User.class.getDeclaredFields();
Annotation[] annotations = fields[0].getAnnotations(); // 获取到注解 @ValidB

RecordComponent[] recordComponents = User.class.getRecordComponents();
annotations = recordComponents[0].getAnnotations(); // 获取到注解 @ValidA
```

完整代码如下：

```java
@Target({ ElementType.RECORD_COMPONENT})
@Retention(RetentionPolicy.RUNTIME)
@interface ValidA {}

@Target({ ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@interface ValidB {}

record User(@ValidA @ValidB long id, String name, int age) {}

public class Main {
    public static void main(String[] args) {
        Field[]      fields      = User.class.getDeclaredFields();
        Annotation[] annotations = fields[0].getAnnotations(); // 获取到注解 @ValidB
        System.out.println(annotations[0]); // @org.example.ValidB()

        RecordComponent[] recordComponents = User.class.getRecordComponents();
        annotations = recordComponents[0].getAnnotations(); // 获取到注解 @ValidA
        System.out.println(annotations[0]); // @org.example.ValidA()
    }
}
```



### 4、Record 类的方法

> **普通方法和静态方法**

1、Record 类体可以声明成员方法和静态方法，和一般类一样。但是**不能声明 abstract 或者 native 方法**：

```java
public record User(long id, String name, int age) {
    public String test1() {
        return "test1";
    }

    public void test2() {
        System.out.println("test2");
    }

    public static void test3() {
        System.out.println("test3");
    }

    public static User of() {
        return new User(1L, "Sam", 18);
    }

    public static User of(String name) {
        return new User(2L, name, 16);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(User.of().test1()); // test1
        User.of().test2(); // test2
        User.test3(); // test3
        System.out.println(new User(1L, "Sam", 18)); // User[id=1, name=Sam, age=18]
        System.out.println(User.of()); // User[id=1, name=Sam, age=18]
        System.out.println(User.of("Sam")); // User[id=2, name=Sam, age=16]
    }
}
```

> **代码块与静态代码块**

1、Record 类体也不能包含实例初始化块代码块：

```java
public record UserRecord(long id, String name, int age) {
    {
        System.out.println(); // 编译异常
    }
}
// Instance initializer is not allowed in record
```

2、Record 类体可以包含静态代码块：

```java
public record UserRecord(long id, String name, int age) {
    static {
        System.out.println(); // 编译与执行都正常
    }
}
```

> **成员方法**（获取属性值的方法）

Record 的所有成员方法，都是 **public final 非 static 的**，对于每一个属性，**都有一个对应的无参数返回类型为属性类型方法名称为属性名称的方法，即这个属性的 accessor**。前面说这个方法是 getter 方法其实不太准确，因为方法名称中并没有 get 或者 is 而是只是纯属性名称作为方法名。

1、这个方法如果我们自己指定了，就不会自动生成：

```java
public record User(long id) {
    @Override
    public long id() {
        return id;
    }
}
```

2、如果没有自己指定，则会自动生成这样一个方法：

1. 方法名就是属性名称

2. 返回类型就是对应的属性类型

3. 是一个 public 方法，并且没有声明抛出任何异常

4. 方法体就是返回对应属性

5. 如果属性上面有任何注解，那么这个注解如果能加到方法上那么也会自动加到这个方法上。例如：

   ```java
   @Target({ElementType.RECORD_COMPONENT, ElementType.METHOD})
   @Retention(RetentionPolicy.RUNTIME)
   public @interface A {}
   
   @Target({ ElementType.FIELD})
   @Retention(RetentionPolicy.RUNTIME)
   public @interface B {}
   
   public record User(@A @B long id, String name, int age) {}
   ```

6. 下面获取 id() 这个方法的注解则会获取到注解 `@A`：

   ```java
   record User(@A @B long id, String name, int age) {}
   Method id = User.class.getDeclaredMethod("id");
   Annotation[] idAnnotations = id.getAnnotations(); // @A
   ```

3、由于会自动生成这些方法，所以 **Record 成员的名称不能和 Object 的某些不符合上述条件（即上面提到的 6 条）的方法的名称一样**，例如：

```java
public record User(
    int wait,     // 编译错误
    int hashcode, // 这个不会有错误，因为 hashcode() 方法符合自动生成的 accessor 的限制条件。
    int toString, // 编译错误
    int finalize  // 编译错误
)}
```

4、Record 类如果没有指定，则默认会生成实现 java.lang.Record 的抽象方法，即：hashcode()、equals()、toStrng() 这三个方法。这**三个方法的实现方式，在之前 Java 14 已经详细分析过**，这里简单回顾下要点：

1. hashcode()：在编译的时候自动生成字节码实现，核心逻辑基于 ObjectMethods 的 makeHashCode 方法，里面的逻辑是对于每一个属性的哈希值移位组合起来。注意这里的所有调用（包括对于 ObjectMethods 的方法调用以及获取每个属性）都是利用 MethodHandle 实现的近似于直接调用的方式调用的。
2. equals()：在编译的时候自动生成字节码实现，核心逻辑基于 ObjectMethods 的 makeEquals 方法，里面的逻辑是对于两个 Record 对象每一个属性判断是否相等（对于引用类型用Objects.equals()，原始类型使用 ==），注意这里的所有调用（包括对于 ObjectMethods 的方法调用以及获取每个属性）都是利用 MethodHandle 实现的近似于直接调用的方式调用的。
3. toString()：在编译的时候自动生成字节码实现，核心逻辑基于 ObjectMethods 的 makeToString 方法，里面的逻辑是对于每一个属性的值组合起来构成字符串。注意这里的所有调用（包括对于 ObjectMethods 的方法调用以及获取每个属性）都是利用 MethodHandle 实现的近似于直接调用的方式调用的。



### 5、Record 类构造器

如果没有指定构造器，Record 类会自动生成一个以所有属性为参数并且给每个属性赋值的构造器。我们可以通过两种方式自己声明构造器：

**第一种是明确声明以所有属性为参数并且给每个属性赋值的构造器**，这个构造器需要满足：

1. 构造器参数需要包括所有属性（同名同类型），并按照 Record 类头的声明顺序。
2. 不能声明抛出任何异常（不能使用 throws）
3. 不能调用其他构造器（即不能使用 this(xxx)）
4. 构造器需要对于每个属性进行赋值。

**对于其他构造器，需要明确调用这个包含所有属性的构造器**：

```java
public record User(long id, String name, int age) {
    public User(int age, long id) {
        this(id, "name", age);
    }
    // 此构造器可以声明, 也可以省略不声明, 因为默认会创建一个所以属性的构造器
    public User(long id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}
```

**第二种是简略方式声明**，又称：紧凑的方式进行参数校验，例如：

```java
public record User(long id, String name, int age) {
    public User {
        if (null == name || id == 0) {
            throw new IllegalArgumentException();
        }
        System.out.println("initialized");
        id = 1000 + id;
        name = "prefix_" + name;
        age = 1 + age;
        // 在这之后，对每个属性赋值
    }
}
```



### 6、Record 特性总结

1. Record 定义的类是 final 不可变类
2. Record 定义的类需要将所有成员变量通过参数的形式定义
3. Record 定义的类默认会生成全部参数的构造方法，除了自动生成的构造方法，还可以自定义构造方法。
4. Record 定义的类只能声明静态属性，不能声明普通属性，普通属性只能在头部声明
5. Record 定义的类所有成员属性的方法（类似 getter 方法），都是 public final 非 static 的
6. Record 定义的类中可以定义静态方法、普通方法（可以是public、private等）
7. Record 定义的类中会自动生成 hashcode()、equals()、toStrng() 这三个方法
8. Record 定义的类可以提供紧凑的方式进行参数校验（其实就是全部参数的构造方法中做校验）



## 4、JEP 390：基于值的类的警告【正式版】

> 以下介绍摘自：[实操 | 剖析 Java16 新语法特性](https://xie.infoq.cn/article/8304c894c4e38318d38ceb116)，原文写的很不错，推荐阅读。

早在 Java 9 版本时，Java 的设计者们就对 @Deprecated 注解进行了一次升级，增加了 since 和 forRemoval 等 2 个新元素。其中，since 元素用于指定标记了 @Deprecated 注解的 API 被弃用时的版本，而 forRemoval 则进一步明确了 API 标记 @Deprecated 注解时的语义，如果`forRemoval=true`时，则表示该 API 在未来版本中肯定会被删除，开发人员应该使用新的 API 进行替代，不再容易产生歧义（Java 9 之前，标记 @Deprecated 注解的 API，语义上存在多种可能性，比如：存在使用风险、可能在未来存在兼容性错误、可能在未来版本中被删除，以及应该使用更好的替代方案等）。

仔细观察原始类型的包装类（比如：java.lang.Integer、java.lang.Double），不难发现，其构造函数上都已经标记有注解`@Deprecated(since="9", forRemoval = true)`，这就意味着其构造函数在将来会被删除，不应该在程序中继续使用诸如`new Integer();`这样的编码方式（建议使用`Integer a = 10;`或者`Integer.valueOf()`函数），如果继续使用，编译期将会产生'Integer(int)' is deprecated and marked for removal 告警。并且，值得注意的是，这些包装类型已经被指定为同 `java.util.Optional` 和 `java.time.LocalDateTime` 一样的值类型。

1、在 Java 16 中添加了一个注解，用于标识当前是是基于值的类，比如 Java 8 引入的预防空指针的 Optional 类，现在已经添加了注解标识。

```java
@jdk.internal.ValueBased
public final class Optional<T> {
    // ...
}
```

2、其次，如果继续在 synchronized 同步块中使用值类型，将会在编译期和运行期产生警告，甚至是异常。在此大家需要注意，就算编译期和运行期没有产生警告和异常，也不建议在 synchronized 同步块中使用值类型，举个自增的例子。示例：

```java
public void inc(Integer count) {
    for (int i = 0; i < 10; i++) {
        new Thread(() -> {
            synchronized (count) {
                count++;
            }
        }).start();
    }
}
```

当执行上述程序示例时，最终的输出结果一定会与你的期望产生差异，这是许多新人经常犯错的一个点，因为在并发环境下，Integer 对象根本无法通过 synchronized 来保证线程安全，这是因为每次的 count++ 操作，所产生的 hashcode 均不同，简而言之，每次加锁都锁在了不同的对象上。因此，如果希望在实际的开发过程中保证其原子性，应该使用 **AtomicInteger**。

3、目前 JDK 中的 **Value-based Classes** 都带有 jdk.internal.ValueBased 注解，或者他们的实现接口，父类带有这个注解，包括：

- java.lang 包：
  - 原始类型的封装类，例如 java.lang.Integer
  - java.lang.Runtime.Version 类
  - 操作系统进程的句柄 java.lang.ProcessHandle 和他的实现类 java.lang.ProcessHandleImpl
- java.time 包下的一些时间封装类
- java.util 包：
  - Optional 相关，例如：java.util.Optional, java.util.OptionalInt, java.util.OptionalLong, java.util.OptionalDouble
  - 所有不可变集合以及底层实现的不可变元素，例如：Set.of 的返回 java.util.ImmutableCollections.AbstractImmutableSet



## 5、JEP 338：Vector API【首次孵化】

向量（Vector） API 最初由 [JEP 338](https://openjdk.java.net/jeps/338) 提出，并作为[孵化 API](http://openjdk.java.net/jeps/11)集成到 Java 16 中。第二轮孵化由 [JEP 414](https://openjdk.java.net/jeps/414) 提出并集成到 Java 17 中，第三轮孵化由 [JEP 417 提出并集成到 Java 18 中，第四轮由 [JEP 426](https://openjdk.java.net/jeps/426) 提出并集成到了 Java 19 中。

该孵化器 API 提供了一个 API 的初始迭代以表达一些向量计算，这些计算在运行时可靠地编译为支持的 CPU 架构上的最佳向量硬件指令，从而获得优于同等标量计算的性能，充分利用单指令多数据（SIMD）技术（大多数现代 CPU 上都可以使用的一种指令）。尽管 HotSpot 支持自动向量化，但是可转换的标量操作集有限且易受代码更改的影响。该 API 将使开发人员能够轻松地用 Java 编写可移植的高性能向量算法。

Vector API 旨在通过提供一种在 Java 中编写复杂矢量算法的机制来解决这些问题，使用 HotSpot 中预先存在的矢量化支持，但使用用户模型使矢量化更加可预测和健壮。手工编码的向量循环可以表达 hashCode自动向量化器可能永远不会优化的高性能算法（例如向量化或专门的数组比较）。这种显式矢量化 API 可能适用于许多领域，例如机器学习、线性代数、密码学、金融和 JDK 本身的用法。

主要是提供了 jdk.incubator.vector 来用于矢量计算，实例如下

```java
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_256;

void vectorComputation(float[] a, float[] b, float[] c) {

    for (int i = 0; i < a.length; i += SPECIES.length()) {
        var m = SPECIES.indexInRange(i, a.length);
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i, m);
        var vb = FloatVector.fromArray(SPECIES, b, i, m);
        var vc = va.mul(va).add(vb.mul(vb)).neg();
        vc.intoArray(c, i, m);
    }
}
```



## 6、JEP 347：启用 C++14 语言功能

这项更新和 Java 开发者关系不太密切，JEP 347 允许 在 JDK 的 C++ 源码中使用 C++ 14 的语言特性，并且给出了哪些特性可以在 HotSpot 代码中使用的具体说明或指导。

在 Java 15 中，JDK 中 C++ 代码使用的语言特性仅限于 C++98/03 语言标准。它要求更新各种平台编译器的最低可接受版本。

要利用 C++14 语言该功能，需要在构建时进行一些更改，具体取决于平台编译器。还需要指定各种平台编译器的最低可接受版本。应明确指定所需的语言标准；较新的编译器版本可能会默认使用较新的且可能不兼容的语言标准。

- Windows：JDK 11 需要 Visual Studio 2017。（早期版本会生成配置时警告，可能会也可能不起作用。）对于 Visual Studio 2017，默认的 C++ 标准是 C++14。/std:c++14 应添加该选项。将完全放弃对旧版本的支持。
- Linux：将 -std=gnu++98编译器选项替换为 -std=c++14. gcc 的最低支持版本是 5.0。
- macOS：将 -std=gnu++98编译器选项替换为 -std=c++14. clang 的最低支持版本是 3.5。
- AIX/PowerPC：将 -std=gnu++98编译器选项 替换为 -std=c++14并要求使用 xlclang++ 作为编译器。xlclang++ 的最低支持版本是 16.1。



## 7、JEP 357：从 Mercurial 迁移到 Git

在此之前，OpenJDK 源代码是使用版本管理工具 Mercurial 进行管理的，我们可以在：http://hg.openjdk.java.net/ 中查看 OpenJDK 的源代码历史版本。

但是现在迁移到了 GIt ，主要原因如下：

1. Mercurial 生成的版本控制元数据过大。
2. Mercurial 相关的开发工具比较少，而 Git 几乎在所有的主流 IDE 中已经无缝集成。
3. Mercurial 相关的服务比较少，无论是自建托管，还是服务托管。

为了优雅的迁移到 Git，OpenJDK 做了如下操作。

1. 将所有的单存储库 OpenJDK 项目从 Mercurial 迁移到 Git。
2. 保留所有的版本控制历史，也包括 Tag。
3. 根据 Git 的最佳实践重新格式化提交的消息。
4. 创建了一个工具用来在 Mercurial 和 Git 哈希之间进行转换。

转换后的存储库的初始原型显示版本控制元数据的大小显着减少。例如，存储库的 .git目录对于 jdk/jdkGit 大约为 300 MB，.hg对于 Mercurial，该目录大约为 1.2 GB，具体取决于所使用的 Mercurial 版本。元数据的减少保留了本地磁盘空间并减少了克隆时间，因为需要通过线路的位更少。Git 还具有 仅克隆部分历史记录的浅层克隆，从而为不需要整个历史记录的用户提供更少的元数据。



## 8、JEP 369：迁移到 GitHub

和 JEP 357 从 Mercurial 迁移到 Git 的改变一致，在把版本管理迁移到 Git 之后，选择了在 GitHub 上托管 OpenJDK 社区的 Git 仓库。不过只对 JDK 11 以及更高版本 JDK 进行了迁移。

可以去 GitHub 上查看：https://github.com/openjdk/jdk/



## 9、JEP 376：ZGC 并发线程堆栈处理

Java16 将 ZGC 线程堆栈处理从安全点（Safepoints）移动到并发阶段。甚至在大堆上也允许在毫秒内暂停 GC 安全点（Safepoints）。消除 ZGC 垃圾收集器中最后一个延迟源可以极大地提高应用程序的性能和效率。

- 从 ZGC 安全点中删除线程堆栈处理。
- 使堆栈处理变得懒惰、协作、并发和增量。
- 从 ZGC 安全点中删除所有其他每线程根处理。
- 提供一种机制，其他 HotSpot 子系统可以通过该机制延迟处理堆栈。

> 如果忘记了什么是 Safepoints，可以先回顾一下：
>
> 1. 在之前需要 GC 的时候，为了进行垃圾回收，需要所有的线程都暂停下来，这个暂停的时间我们成为 **Stop The World**。
> 2. 而为了实现 STW 这个操作， JVM 需要为每个线程选择一个点停止运行，这个点就叫做**安全点（Safepoints）**。

为什么要这么优化呢？

- ZGC 垃圾收集器 (GC) 旨在使 HotSpot 中的 GC 暂停和可扩展性问题成为过去。到目前为止，我们已经将所有随堆大小和元空间大小扩展的 GC 操作从安全点操作移到并发阶段。这些包括标记、重定位、引用处理、类卸载和大多数根处理。

- 仍然在 GC 安全点中完成的唯一活动是根处理的子集和有时间限制的标记终止操作。根包括 Java 线程堆栈和各种其他线程根。这些根是有问题的，因为它们会随着线程的数量而扩展。由于大型机器上有许多线程，根处理成为一个问题。

- 为了超越我们今天所拥有的，并满足在 GC 安全点内花费的时间不超过一毫秒的期望，即使在大型机器上，我们也必须将这种每线程处理，包括堆栈扫描，移出并发阶段。

- 在这项工作之后，在 ZGC 安全点操作中基本上不会做任何重要的事情。

- 作为该项目的一部分构建的基础设施最终可能会被其他项目使用，例如 Loom 和 JFR，以统一延迟堆栈处理。



## 10、JEP 380：Unix 域套接字通道

Unix-Domain 套接字长期以来一直是大多数 Unix 平台的一个功能，现在在 Windows 10 和 Windows Server 2019 也提供了支持。此特性为 java.nio.channels 包的套接字通道和服务器套接字通道 API 添加了 Unix-Domain（AF_UNIX）套接字支持。它扩展了继承的通道机制以支持 Unix-Domain 套接字通道和服务器套接字通道。Unix-Domain 套接字用于同一主机上的进程间通信（IPC）。它们在很大程度上类似于 TCP/IP，区别在于套接字是通过文件系统路径名而不是 Internet 协议（IP）地址和端口号寻址的。对于本地进程间通信，Unix-Domain 套接字比 TCP/IP 环回连接更安全、更有效。

为了支持 Unix 域套接字通道，我们将添加以下 API 元素：

- 添加 java.net.UnixDomainSocketAddress.java 类用于支持 Unix 域套接字通道。
- 添加 Unix-Domain Socket 到 SocketChannel 和 ServerSocketChannel API 中。
- 添加枚举信息 java.net.StandardProtocolFamily.UNIX。



## 11、JEP 386：移植 Alpine Linux

Apine Linux 是一个独立的、非商业的 Linux 发行版，它十分的小，一个容器需要不超过 8MB 的空间，最小安装到磁盘只需要大约 130MB 存储空间，并且十分的简单，同时兼顾了安全性。

此提案将 JDK 移植到了 Apline Linux，由于 Apline Linux 是基于 musl lib 的轻量级 Linux 发行版，因此其他 x64 和 AArch64 架构上使用 musl lib 的 Linux 发行版也适用。

Alpine Linux 发行版由于其较小的镜像大小，被广泛用于云部署、微服务和容器环境。例如，用于 Alpine Linux 的 Docker 基础映像小于 6 MB。使 Java 在此类设置中开箱即用将允许 Tomcat、Jetty、Spring 和其他流行框架在此类环境中本地工作。

通过使用 jlink (JEP 282) 来减少 Java 运行时的大小，用户将能够创建一个更小的图像来运行特定的应用程序。应用程序所需的模块集可以通过 jdeps 命令确定。例如，如果目标应用程序仅依赖于 java.base 模块，则带有 Alpine Linux 的 Docker 映像和仅带有该模块的 Java 运行时和服务器 VM 大小为 38 MB。同样的动机也适用于嵌入式部署，它们也有大小限制。

具体操作如下：

- 该 JEP 旨在整合上游的Portola 项目。

- 此端口将不支持 HotSpot Serviceability Agent 的附加机制。

- 要在 Alpine Linux 上构建 JDK 的 musl 变体，需要以下软件包：

  ```bash
  alpine-sdk alsa-lib alsa-lib-dev autoconf bash cups-dev cups-libs fontconfig fontconfig-dev freetype freetype-dev grep libx11 libx11-dev libxext libxext-dev libxrandr libxrandr-dev libxrender libxrender-dev libxt libxt-dev libxtst -dev linux-headers zip
  ```

- 安装这些软件包后，JDK 构建过程将照常工作。

- 如果有需求，可以在后续增强中实现其他架构的 Musl 端口。



## 12、JEP 387：弹性元空间

自从 JEP 122 引入了 Metaspace 以来，根据反馈，Metaspace 经常占用过多的堆外内存，从而导致内存浪费。弹性元空间这个特性可将未使用的 HotSpot 类元数据（即元空间：metaspace）内存更快速地返回到操作系统，从而减少元空间的占用空间。此 JEP 387 提案简化元空间代码以降低维护成本。

> 具体操作：

我们建议用基于好友的分配方案替换现有的元空间内存分配器。这是一种古老且经过验证的算法，已成功用于例如 Linux 内核。这种方案将使以更小的块分配元空间内存变得可行，这将减少类加载器的开销。它还将减少碎片，这将使我们能够通过将未使用的元空间内存返回给操作系统来提高弹性。

我们还将根据需要将操作系统中的内存延迟提交到 arenas。这将减少从大型竞技场开始但不立即使用它们或可能永远不会使用它们的全部范围的加载器的占用空间，例如引导类加载器。

最后，为了充分利用伙伴分配提供的弹性，我们将元空间内存安排成大小均匀的颗粒，这些颗粒可以相互独立地提交和取消提交。这些颗粒的大小可以通过一个新的命令行选项来控制，它提供了一种控制虚拟内存碎片的简单方法。

可以在 [此处](https://cr.openjdk.java.net/~stuefe/JEP-Improve-Metaspace-Allocator/review-guide/review-guide-1.0.html) 找到详细描述新算法的文档。工作原型作为 [JDK 沙箱存储库中的一个分支](https://hg.openjdk.java.net/jdk/sandbox/shortlog/38a706be96d4) 存在。



## 13、JEP 388：Windows/AArch64 端口

将 JDK 移植到 Windows/AArch64 架构上，Windows/AArch64 已经是终端用户市场的热门需求。

> 具体操作：

通过扩展之前为 Linux/AArch64 移植 ([JEP 237](https://openjdk.org/jeps/237)) 所做的工作，我们已将 JDK 移植到 Windows/AArch64 。此端口包括模板解释器、C1 和 C2 JIT 编译器以及垃圾收集器（串行、并行、G1、Z 和 Shenandoah）。它支持 Windows 10 和 Windows Server 2016 操作系统。

这个 JEP 的重点不是移植工作本身，它大部分是完整的，而是将移植集成到 JDK 主线存储库中。

目前，我们有十几个变更集。我们将对共享代码的更改保持在最低限度。我们的更改将 AArch64 内存模型的支持扩展到 Windows，解决了一些 MSVC 问题，将 LLP64 支持添加到 AArch64 端口，并在 Windows 上执行 CPU 功能检测。我们还修改了构建脚本以更好地支持交叉编译和 Windows 工具链。

新平台代码本身仅限于 15 (+4) 个文件和 1222 行 (+322)。

可在 [此处](https://openjdk.org/jeps/237) 获得抢先体验的二进制文件。



## 14、JEP 389：外部链接器 API【孵化中】

该孵化器 API 提供了静态类型、纯 Java 访问原生代码的特性，该 API 将大大简化绑定原生库的原本复杂且容易出错的过程。Java 1.1 就已通过 Java 原生接口（JNI）支持了原生方法调用，但并不好用。Java 开发人员应该能够为特定任务绑定特定的原生库。它还提供了外来函数支持，而无需任何中间的 JNI 粘合代码。

这项提案让 Java 代码可以调用由其他语言（比如 C ，C++）编写的编译后的机器代码，替换了之前的 JNI 形式。由于这是一个孵化中的功能，运行时需要添加 `--add-modules jdk.incubator.foreign` 参数来编译和运行 Java 代码。下面是一个调用 C 语言函数方法，然后输出运行结果的例子。

1、编写一个 C 函数打印一个 "hello world"，文件名：hello.c

```c
#include <stdio.h>

void printHello(){
	printf("hello world\n");
}
```

2、将上面的代码编译，然后输出到共享库 hello.so

```bash
$ gcc -c -fPIC hello.c
$ gcc -shared -o hello.so hello.o
$ ll
total 128
-rw-r--r--  1 darcy  staff    76B 10 28 19:46 hello.c
-rw-r--r--  1 darcy  staff   776B 10 28 19:46 hello.o
-rwxr-xr-x  1 darcy  staff    48K 10 28 19:47 hello.so
```

3、编写一个 Java 代码，调用 hello.so 的 printHello 方法。

```java
import jdk.incubator.foreign.CLinker;
import jdk.incubator.foreign.FunctionDescriptor;
import jdk.incubator.foreign.LibraryLookup;
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodType;
import java.nio.file.Path;
import java.util.Optional;

public class JEP389 {
    public static void main(String[] args) throws Throwable {
        Path path = Path.of("/Users/darcy/git/java-core/java-16/src/com/wdbyte/hello.so");
        LibraryLookup libraryLookup = LibraryLookup.ofPath(path);
        Optional<LibraryLookup.Symbol> optionalSymbol = libraryLookup.lookup("printHello");
        
        if (optionalSymbol.isPresent()) {
            LibraryLookup.Symbol symbol = optionalSymbol.get();
            FunctionDescriptor functionDescriptor = FunctionDescriptor.ofVoid();
            MethodType methodType = MethodType.methodType(Void.TYPE);
            MethodHandle methodHandle = CLinker.getInstance().downcallHandle(
                symbol.address(),
                methodType,
                functionDescriptor);
            methodHandle.invokeExact();
        }
    }
}
```

4、Java 代码编译

```bash
$ javac --add-modules jdk.incubator.foreign JEP389.java
警告: 使用 incubating 模块: jdk.incubator.foreign
1 个警告
```

5、Java 代码执行

```bash
$ java --add-modules  jdk.incubator.foreign -Dforeign.restricted=permit JEP389.java
WARNING: Using incubator modules: jdk.incubator.foreign
警告: 使用 incubating 模块: jdk.incubator.foreign
1 个警告
hello world
```



## 15、JEP 392：打包工具【正式版】

在 Java 14 中，JEP 343 引入了打包工具，命令是 jpackage。在 Java 15 中，继续孵化，现在 Java 16 中，终于成为了正式功能。

下面是一个例子，把一个简单的 Java Swing 程序打包成当前操作系统支持的软件格式，然后安装到当前电脑。

1、编写 Java 代码

```java
import javax.swing.*;
import java.awt.*;

public class JEP392 {
    public static void main(String[] args) {
        JFrame frame = new JFrame("Hello World Java Swing");
        frame.setMinimumSize(new Dimension(800, 600));
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        JLabel lblText = new JLabel("Hello World!", SwingConstants.CENTER);
        frame.getContentPane().add(lblText);
        frame.pack();
        frame.setVisible(true);
    }
}
```

2、编译后，创建一个 JAR 文件

```bash
$ javac JEP392.java
$ java JEP392.java
$ jar cvf JEP392.jar JEP392.class
```

3、将生成的 JEP392.jar 打包到符合当前平台的软件包中（可以查看到目录中显示 JEP392-1.0.exe（本人 Windows，所以格式 exe）

```bash
# -i 等于 --input
# -n 等于 --name 
$ jpackage -i . -n JEP392 --main-jar JEP392.jar --main-class JEP392
$ ll
-r-xr-xr-x 1 lsx 197609 47338496 Apr 20 17:14 JEP392-1.0.exe*
-rw-r--r-- 1 lsx 197609      864 Apr 20 16:10 JEP392.class
-rw-r--r-- 1 lsx 197609     1000 Apr 20 16:11 JEP392.jar
-rw-r--r-- 1 lsx 197609      484 Apr 20 16:04 JEP392.java
```

4、双击如上的 JEP392-1.0.exe 文件即可安装，查找安装好的 exe 程序目录，双击如下的 JEP392.exe 文件即可启动。

```bash
$ ll C:\Program Files\JEP392
-rwxr-xr-x 1 lsx 197609 427520 Apr 20 17:14 JEP392.exe*
-rw-r--r-- 1 lsx 197609  25214 Apr 20 17:14 JEP392.ico
drwxr-xr-x 1 lsx 197609      0 Apr 20 17:15 app/
drwxr-xr-x 1 lsx 197609      0 Apr 20 17:15 runtime/
```

5、不同的系统安装位置不同：

- Linux：`/opt`
- MacOS：`/Applications`
- Windows：`C:\Program Files\`



## 16、JEP 393：外部内存访问 API【三次孵化】

此提案旨在引入新的 API 以允许 Java 程序安全有效的访问 Java 堆之外的内存。Foreign-Memory Access API 早在 Java 14 的时候就已经提出了 JEP 370，在 Java 15 中重新孵化 JEP 370，现在在 Java 16 中再次孵化 JEP 393。此 API 更新中包含以下更改：

- MemorySegment和 MemoryAddress接口之间的角色分离更清晰；
- 一个新的接口，MemoryAccess它提供了通用的静态内存访问器，以便 VarHandle在简单的情况下最大限度地减少对API的需求
- 支持*共享*段，和使用 Cleaner

此提案的目标如下：

1. 通用：单个 API 应该能够对各种外部内存（如本机内存、持久内存、堆内存等）进行操作。
2. 安全：无论操作何种内存，API 都不应该破坏 JVM 的安全性。
3. 控制：可以自由的选择如何释放内存（显式、隐式等）。
4. 可用：如果需要访问外部内存，API 应该是：sun.misc.Unsafa.

> 具体操作：

Foreign-Memory Access API 作为一个名为的孵化器模块提供 jdk.incubator.foreign，位于同名的包中。它引入了三个主要抽象：MemorySegment，MemoryAddress和 MemoryLayout：

- MemorySegment对具有给定空间和时间界限的连续内存区域进行建模，
- MemoryAddress对一个地址建模，该地址可以驻留在堆上或堆外，并且
- MemoryLayout是对内存段内容的编程描述。

可以从各种来源创建内存段，例如本机内存缓冲区、内存映射文件、Java 数组和字节缓冲区（直接或基于堆）。例如，可以按如下方式创建本机内存段：

```java
try (MemorySegment segment = MemorySegment.allocateNative(100)) {
   //...
}
```

这将创建一个与大小为 100 字节的本机内存缓冲区相关联的内存段。

内存段在空间上是有界的，这意味着它们有下限和上限。任何尝试使用该段访问这些边界之外的内存都将导致异常。

正如 try上面使用-with-resource 构造所证明的那样，内存段也是有时间限制的，这意味着它们必须被创建、使用，然后在不再使用时关闭。关闭一个段会导致额外的副作用，例如与该段相关联的内存的释放。任何访问已关闭内存段的尝试都会导致异常。空间和时间边界共同保证了外部内存访问 API 的安全性，从而保证它的使用不会使 JVM 崩溃。



## 17、JEP 396：默认强封装 JDK 内部

此特性会默认强封装 JDK 的所有内部元素，但关键内部 API（例如 `sun.misc.Unsafe`）除外。默认情况下，使用早期版本成功编译的访问 JDK 内部 API 的代码可能不再起作用。鼓励开发人员从使用内部元素迁移到使用标准 API 的方法上，以便他们及其用户都可以无缝升级到将来的 Java 版本。强封装由 JDK 9 的启动器选项`–illegal-access`控制，到 JDK 15 默认改为 warning，从 JDK 16 开始默认为 deny。（目前）仍然可以使用单个命令行选项放宽对所有软件包的封装，将来只有使用`–add-opens`打开特定的软件包才行。

> 此提案的目标：

- 继续提高 JDK 的安全性和可维护性，这是 [Project Jigsaw](https://openjdk.java.net/projects/jigsaw) 的主要目标之一。
- 鼓励开发人员从使用内部元素迁移到使用标准 API，以便他们和他们的用户可以轻松升级到未来的 Java 版本。

> 此提案的动机：

多年来，各种库、框架、工具和应用程序的开发人员以损害安全性和可维护性的方式使用 JDK 的内部元素。特别是：

包的一些非 public类、方法和字段 `java.*`定义了特权操作，例如在特定类加载器中定义新类的能力，而其他则传送敏感数据，例如加密密钥。尽管在 `java.*`包中，但这些元素是 JDK 的内部元素。外部代码通过反射使用这些内部元素会使平台的安全性处于危险之中。
包的所有类、方法和字段 `sun.*`都是 JDK 的内部 API。一些类，方法和属性 `com.sun.*`， `jdk.*`以及 `org.*`包也是内部API。这些 API 从来都不是标准的，从来没有得到支持，也从来没有打算供外部使用。外部代码使用这些内部元素是一种持续的维护负担。为了不破坏现有代码，保留这些 API 所花费的时间和精力可以更好地用于推动平台向前发展。

在 Java 9 中，我们通过利用模块来限制对其内部元素的访问，提高了 JDK 的安全性和可维护性。模块提供强封装，这意味着

模块外的代码只能访问该 模块导出的包的 public和 protected元素，并且protected 此外，元素只能从定义它们的类的子类中访问。强封装适用于编译时和运行时，包括编译代码尝试在运行时通过反射访问元素时。public导出包的非元素和未导出包的所有元素都被称为强装。

在 JDK 9 及更高版本中，我们强烈封装了所有新的内部元素，从而限制了对它们的访问。然而，为了帮助迁移，我们故意选择不在运行时强封装 JDK 8 中存在的包的内容。因此类路径上的库和应用程序代码可以继续使用反射来访问非 public元素的 `java.*`包，以及所有元素 `sun.*`和其他内部包，用于封装在JDK 8存在这种安排被称为放松强大的封装性。

我们于 2017 年 9 月发布了 JDK 9。 JDK 的大多数常用内部元素现在都有标准替代品。开发人员必须在三年中，从标准的API，如JDK的内部元素迁移走 java.lang.invoke.MethodHandles.Lookup::defineClass，java.util.Base64和 java.lang.ref.Cleaner。许多库、框架和工具维护者已经完成了迁移并发布了其组件的更新版本。我们现在准备迈出下一步，按照 sun.misc.UnsafeProject Jigsaw 最初计划的那样，对 JDK 的所有内部元素进行强封装——除了关键的内部 API，例如。

> 此提案的具体操作：

松弛强封装由启动器选项控制 --illegal-access。这个选项由JEP 261引入，被挑衅地命名以阻止它的使用。它目前的工作原理如下：

- --illegal-access=permit安排 JDK 8 中存在的每个包都对未命名模块中的代码开放。因此，类路径上的代码可以继续使用反射来访问包的非公共元素 java.*，以及 sun.* JDK 8 中存在的包和其他内部包的所有元素。对任何此类元素的第一次反射访问操作会导致发出警告，但在那之后不会发出警告。

自 JDK 9 以来，此模式一直是默认模式。

- --illegal-access=warn``permit除了针对每个非法反射访问操作发出警告消息之外，其他都相同。

- --illegal-access=debug``warn除了为每个非法反射访问操作发出警告消息和堆栈跟踪之外，其他都相同。

- --illegal-access=deny禁用所有非法访问操作，但由其他命令行选项启用的操作除外，例如， --add-opens。

作为对 JDK 的所有内部元素进行强封装的下一步，我们建议将 --illegal-access 选项的默认模式从 permit 更改为 deny。通过此更改，默认情况下将不再打开 JDK 8 中存在且不包含关键内部 API 的包；此处提供完整列表 。 该 sun.misc 包仍将由 jdk.unsupported 模块导出，并且仍可通过反射访问。

我们还将修改Java 平台规范中的相关文本，以禁止在任何 Java 平台实现中默认打开任何包，除非该包明确声明 open在其包含模块的声明中的 permit，warn以及 debug该模式 --illegal-access选项将继续工作。这些模式允许最终用户根据需要选择宽松的强封装。

我们预计未来的 JEP 会 --illegal-access 完全取消该选项。那时将无法通过单个命令行选项打开所有 JDK 8 包。仍然可以使用 --add-opens命令行选项或 Add-OpensJAR-file 属性来打开特定的包。

为了准备最终删除该 --illegal-access 选项，我们将弃用它作为本 JEP 的一部分进行删除。因此，为 java 启动器指定该选项将导致发出弃用警告。



## 18、参考文献 & 鸣谢

1. Java 新特性文档：https://www.wdbyte.com/java-feature/
2. Java New Feature：https://javaguide.cn/java/new-features/
3. Java 的新特性：https://segmentfault.com/a/1190000004417288/
4. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
5. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/