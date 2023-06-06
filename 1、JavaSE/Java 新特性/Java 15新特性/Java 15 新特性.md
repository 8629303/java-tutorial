# Java 15 新特性

Java 15 在 2020 年 09 月 15 日发布，它不是一个长久支持（LTS）版本。

这次更新共带来 14 个新特性：

- 339: Edwards-Curve Digital Signature Algorithm (EdDSA)：蒙哥马利与扭曲爱德华曲线签名算法
- 360: Sealed Classes (Preview)：密封类（预览）
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

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 15 新特性：https://openjdk.org/projects/jdk/15/
- OracleJDK Java 15 新特性：https://www.oracle.com/java/technologies/javase/15-relnote-issues.html#NewFeature
- Oracle JDK 15 文档：https://docs.oracle.com/en/java/javase/15/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## 1、密封类【预览】

> JEP 360：Sealed Classes（密封类）预览

在 Java 中如果想让一个类不能被继承和修改，我们应该使用 final 关键字对类进行修饰。不过这种要么可以继承，要么不能继承的机制不够灵活，有些时候我们可能想让某个类可以被某些类型继承，但是又不能随意继承，是做不到的。Java 15 尝试解决这个问题，引入了 sealed 类，被 sealed 修饰的类可以指定子类。这样这个类就只能被指定的类继承。**并且 sealed 修饰的类的机制具有传递性，它的子类必须使用指定的关键字进行修饰，且只能是 final、sealed、non-sealed 三者之一**。

- 密闭类修饰符：sealed、non-sealed、final
- 指定可扩展修饰符：permits

【操作示例 1】使用 final、sealed、non-sealed、permits，模拟各种继承及各种限制继承

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
```

> 再次强调：密封类的子类必须需要  final、sealed、non-sealed 三者之一修饰。

【操作示例 2】密封列与接口和模式匹配（instanceof）问题

```java
public class TestSealedClass {
    public static void main(String[] args) {
        test(new C());
    }
    public static void test(A a){
        if(a instanceof Base){
            System.out.println( "it is an Base");
        }else{
            System.out.println("it is not Base");
        }
    }

}

interface Base {}
sealed class A implements Base permits B, C {}
non-sealed class B extends A {}
final class C extends A {}
// 密封类仅仅是控制类的继承和实现关系,不会影响我们的模式匹配

/**
 * 犬类（Dog）只能被牧羊犬（Collie）和田园犬（TuGou）继承，使用 sealed 关键字。
 */
sealed interface Dog permits Collie, TuGou {}
```

【操作示例 3】密封接口和关键字 records，record 类本身就是隐匿式的 final，可以直接实现密封接口。

```java
public class Main {
    public static void main(String[] args) {
        Base base = new Person(10, "旋涡刘能");
    }
}

sealed interface Base {
    void eat();
}

/**
 * record 默认继承的 java.lang.Record
 * record 可以直接实现密封接口,不需要用 sealed、non-sealed 修饰
 * record 本身是隐式的 final 修饰
 */
record Person(Integer pid, String pname) implements Base {
    @Override
    public void eat() {
    }
}

record Student(Integer pid, String pname) implements Base {
    @Override
    public void eat() {
    }
}

record Cook(Integer pid, String pname) implements Base {
    @Override
    public void eat() {
    }
}

record Worker(Integer pid, String pname) implements Base {
    @Override
    public void eat() {
    }
}
```

【操作示例 4】密封类的使用总结

```java
/**
 * sealed 对类进行密封
 * permits 指明哪些类可以继承
 * 子类必须是final修饰的或者也是密封类(sealed)
 * 如果子类不想被密封,可以使用non-sealed修饰
 * <p>
 * final不能修饰接口:
 * - 密封接口在被继承时,子接口要么使用 sealed、non-sealed 修饰
 * - 密封类在被继承时, 子类要么使用 final、sealed、non-sealed 修饰
 * <p>
 * 密封接口不可以使用匿名内部类进行实现
 */
```



## 2、隐藏类【预览】

> JEP 371：Hidden Classes（隐藏类）

隐藏类是为框架所设计的，在运行时生成内部的 class，隐藏类不能直接被其他类的字节码使用，只能在运行时生成类并通过反射间接使用它们。可以独立于其他类加载、卸载，这样可以减少框架的内存占用。

这个特性让开发者可以引入一个无法被其他地方发现使用，且类的生命周期有限的类。这对运行时动态生成类的使用方式十分有利，可以减少内存占用，下面是一个使用示例。

```java
public class JEP371Test {
    public static String lookup() {
        return "hello java";
    }
}
```

把类 JEP371Test 编译后的 Class 转换成 Base64，然后使用 Java 15 新特性加载调用类中的 lookup 方法。

```java
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;
import java.util.Base64;

public class JEP371 {

    private static String CLASS_INFO = "yv66vgAAADQAFAoAAgADBwAEDAAFAAYBABBqYXZhL2xhbmcvT2JqZWN0AQAGPGluaXQ+AQADKClWCAAIAQAOd3d3LndkYnl0ZS5jb20HAAoBABVjb20vd2RieXRlL0pFUDM3MVRlc3QBAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRhYmxlAQAEdGhpcwEAF0xjb20vd2RieXRlL0pFUDM3MVRlc3Q7AQAGbG9va3VwAQAUKClMamF2YS9sYW5nL1N0cmluZzsBAApTb3VyY2VGaWxlAQAPSkVQMzcxVGVzdC5qYXZhACEACQACAAAAAAACAAEABQAGAAEACwAAAC8AAQABAAAABSq3AAGxAAAAAgAMAAAABgABAAAAAwANAAAADAABAAAABQAOAA8AAAAJABAAEQABAAsAAAAbAAEAAAAAAAMSB7AAAAABAAwAAAAGAAEAAAAEAAEAEgAAAAIAEw==";

    public static void main(String[] args) throws Throwable {
        byte[] classInBytes = Base64.getDecoder().decode(CLASS_INFO);
        Class<?> proxy = MethodHandles.lookup()
            .defineHiddenClass(classInBytes, true, MethodHandles.Lookup.ClassOption.NESTMATE)
            .lookupClass();

        System.out.println(proxy.getName());
        MethodHandle mh = MethodHandles.lookup().findStatic(proxy, "lookup", MethodType.methodType(String.class));
        String result = (String) mh.invokeExact();
        System.out.println(result);
    }
}
```

输出内容

```java
JEP371Test/0x0000000800c01800
hello java
```

> 隐藏类番外详解：
>
> 1、什么是 Hidden Class：
>
> - 就是不能直接被其他class的二进制代码使用的class. 主要被一些框架用来生成运行时类,但是这些类不能被用来直接使用的,是通过反射来调用的
> - 比如JDK8中引入的lambda表达式,编译时不会将lambda表达式转换为专门的类,而是在运行时将相应的字节码动态生成相应的类对象
> - 另外使用动态代理也可以为某些类生成新的动态类
>
> 2、隐藏类的特性：
>
> - 不可发现性：因为我们是为某些静态的类动态生成的动态类,所以我们希望这个动态生成的类看作是静态类的一部分,所以我们不希望除了该静态类以外的其他机制发现
> - 访问控制：我们希望在访问控制静态类的同时,也能控制到动态生成的类
> - 生命周期：动态生成类的声明周期一般都比较短. 我们不需要将其保存和静态类的生命周期一致
>
> 3、隐藏类 API 支持：
>
> - java.lang.reflect.Proxy 可以定义隐藏类作为实现代理接口的代理类
>
> - java.lang.invoke.StringConcatFactory 可以生成隐藏类来保存常量连接方法
>
> - java.lang.invoke.LambdaMetaFactory 可以生成隐藏的 nestmate 类，以容纳访问封闭变量的 lambda 主体
>
> - 普通类是通过调用 ClassLoader::defineClass 创建的，而隐藏类是通过调用 Lookup::defineHiddenClass 创建的，这使 JVM 提供的字节派生一个隐藏类，链接该隐藏类，并返回提供对隐藏类的反射访问的查找对象，调用程序可以通过返回的查找对象来获取隐藏类的Class对象



## 3、instanceof 增强【二次预览】

Java 14 中作为预览语言功能引入 instanceof 模式匹配，在 Java 15 中出于第二次预览，而没有任何更改，用于接受更多的使用反馈。回顾 Java 14 即可。这个特性在 Java 16 中成为正式特性。



## 4、record 关键字【二次预览】

Java 14 中引入了 Record 类，Java 15 中对 Record 进行了增强。

1. 可以支持密封（sealed）类型
2. 支持 Record 注解以及相关的反射 API 等
3. record 类在 Java14 暂时还不能声明为 private，在 Java 15 支持 声明为 private

【操作示例】Record 支持密封（sealed）类型

```java
public sealed interface DataBase permits DataBaseSelect, DataBaseUpdate {}

final record DataBaseSelect(@Deprecated String table, String sql) implements DataBase {}

// 由于 records 类本身就是 final 修饰的, 所以可以省略开头的 final 关键字
record DataBaseUpdate() implements DataBase {}
```

在 java.lang.Class 增加了两个公共方法用于获取 Record 类信息：

1. RecordComponent[] getRecordComponents()：获取 record 中的属性数组 RecordComponent[]
2. boolean isRecord()：判断一个类是否用 record 进行声明的



## 5、多行文本块【最终版】

在 Java 15 中，文本块是正式的功能特性了。回顾 Java 13、Java 14 即可。



## 6、CharSequence 新增方法

CharSequence 接口添加了一个默认方法 isEmpty() 来判断字符序列为空，如果是则返回 true。

```java
public interface CharSequence {
    default boolean isEmpty() {
        return this.length() == 0;
    }
}
```



## 7、TreeMap 新增方法

> Specialized Implementations of TreeMap Methods ([JDK-8176894](https://bugs.openjdk.java.net/browse/JDK-8176894))

Java 15对 TreeMap 提供了putIfAbsent、computeIfAbsent、computeIfPresent、compute、merge 方法提供了overriding 实现

TreeMap 新引入了下面这些方法：

- putIfAbsent()：当 key 不存在时才执行 put 方法
- computeIfAbsent()：当 value 不存在时重新计算
- computeIfPresent()：当 value 存在时重新计算
- compute()：用于计算指定key的值，如果Map中不存在指定的key,则计算时值为null, 并将计算的结果作为key的值
- merge()：合并新旧两个值

> 具体用法可以参考Java基础集合篇中的Map教程



## 8、ZGC 功能【正式版】

> JEP377:ZGC：A Scalable Low_Latency Garbage Collector ZGC 功能成为正式标准

ZGC 垃圾收集器在 Java 11 中被引入（JDK9 以后默认的垃圾回收器是G1），经过了多个实验阶段，在 Java 15 中 ZGC 正式发布。

自 2018 年以来，ZGC 已增加了许多改进，从并发类卸载、取消使用未使用的内存、对类数据共享的支持到改进的 NUMA 感知。此外，最大堆大小从 4 TB 增加到 16 TB。支持的平台包括 Linux、Windows 和 MacOS。

ZGC 是一个重新设计的并发的垃圾回收器，通过减少 GC 停顿时间来提高性能。默认的 GC 仍然还是 G1。

Java 15 之前需要通过如下参数来启用 ZGC：

```bash
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC

java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC className
```

Java 15 开始只需要简单的如下参考开启 ZGC：

```bash
-XX:+UseZGC

java -XX:+UseZGC className
```



## 9、Shenandoah GC【正式版】

> JEP 379：Shenandoah: 低停顿时间的垃圾收集器

Shenandoah 一种低停顿的垃圾回收器，在 Java 12 中开始引入，Java 15 中成为了正式功能的一部分，可以使用下面的参数启用 Shenandoah 垃圾收集器（可以不需要添加参数 -XX:+UnlockExperimentalVMOptions）。

```shell
-XX:+UseShenandoahGC

java -XX:+UseShenandoahGC className
```

但是 OpenJDK 15 中默认是没有 Shenandoah 收集器，想要使用此功能可以下载：[AdoptOpenJDK](https://adoptopenjdk.net/)。

> **为什么 OpenJDK 中没有 Shenandoah 垃圾收集器？**
>
> Shenandoah 是一个高性能、低暂停时间的垃圾收集器，它是 Red Hat 主导的项目。当 Red Hat 第一次提议将 Shenandoah 贡献给 OpenJDK 时，Oracle 明确表示不想支持它，OpenJDK 作为一个免费软件，不想支持 Red Hat 的 Shenandoah 完全没有问题。
>
> 最后 Red Hat 选择和 Oracle 合作设计一个真正干净的可插拔垃圾收集器接口，允许任何人轻松选择垃圾收集器以包含在他们的构建中。最终 Shenandoah 进入了 JDK 12，但是没有构建进 OpenJDK。
>
> **Shenandoah 和 ZGC 对比：**
>
> - 相同：性能几乎认为是相同的
> - 不同：ZGC 是 OracleJDK 的，而 Shenandoah 只存在于 OpenJDK 中（需要手动下载），因此使用时需要注意JDK版本。



## 10、edDSA 数字签名算法

> JEP 339 爱德华曲线算法（EdDSA）

Java 15 中增加了一个新的密码学算法，爱德华曲线算法（EdDSA）签名算法。它是由 Schnorr 算法发展而来，在 RFC8032 中被定义实现。

```java
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.Signature;
import java.security.SignatureException;
import java.util.Base64;

public class JEP339 {
    public static void main(String[] args) 
        throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        KeyPairGenerator kpg = KeyPairGenerator.getInstance("Ed25519");
        KeyPair kp = kpg.generateKeyPair();
        byte[] msg = "hello java".getBytes(StandardCharsets.UTF_8);
        Signature sig = Signature.getInstance("Ed25519");
        sig.initSign(kp.getPrivate());
        sig.update(msg);
        byte[] s = sig.sign();
        System.out.println(Base64.getEncoder().encodeToString(s));
    }
}
```





## 11、禁用和废弃偏向锁（Biased Locking）

> JEP 374：禁用和废弃偏向锁（Biased Locking）

在之前，JVM 在处理同步操作，如使用 synchronized 同步时，有一套锁的升级机制，其中有一个锁机制就是偏向锁。然而通过目前的 Java 开发环境来看，使用这些被 synchronized 同步的类的机会并不多，如开发者更喜欢使用 HashMap 或者 ArrayList 而非 `HashTable` 和 Vector。

即使换个角度，当初使用偏向锁是为了提高性能，如今看来性能提升的程度和使用次数都不太有用。而偏向锁的引入增加了 JVM 的复杂性。

所以现在偏向锁被默认禁用，在不久的将来将会彻底删除，对于 Java 15，我们仍然可以使用 `-XX:+UseBiasedLocking` 启用偏向锁定，但它会提示 这是一个已弃用的 API。



## 12、重新实现 DatagramSocket API

> JEP 373:Reimplement the Legacy DatagramSocket API(重新实现 DatagramSocket API)

JEP 373 是 JEP 353 的后续，该方案重新实现了遗留的套接字API。Java 13 中重新实现了旧的 Socket API，在 Java 15 重新实现了遗留的 DatagramSocket。

更改 java.net.DatagramSocket 和 java.net.MulticastSocket 为更加简单、现代化的底层实现。提高了 JDK 的可维护性和稳定性。

通过将 java.net.datagram.Socket 和 java.net.MulticastSocket API 的底层实现替换为更简单、更现代的实现来重新实现遗留的DatagramSocket API。



## 13、外部存储器访问 API【二次孵化】

> JEP383 Foreign-Memory Access API (Second Incubator) 外部存储器访问API

Foreign-Memory Access API 在 Java 14 被作为 incubating API 引入，在 Java 15 处于 Second Incubator，提供了改进。

目的是引入一个 API。以允许 Java 程序安全。有效地访问 Java 对之外的外部存储器.如本机,持久和托管堆。

有许多 Java 程序访是访问外部内存的，比如 Ignite 和 MapDB。该 API 将有助于避免与垃圾收集相关的成本以及与跨进程共享内存以及通过将文件映射到内存来序列化和返序列化内存内容相关的不可预测性。该 Java API 目前没有为访问外部内存停工令人满意的解决方案。但是在新的提议中，API 不应该破坏 JVM 的安全性。



## 14、移除 Nashorn JavaScript 引擎

> JEP 372：移除 Nashorn JavaScript 引擎

Nashorn JavaScript 引擎在 Java 8 中被引入，在 Java 11 中被标记为废弃。由于 ECMAScript 语言发展很快，维护 Nashorn JavaScript 的成本过于高昂，在 Java 15 中被彻底删除。

Java 15 移除了Nashorn JavaScript Engine 及 jjs 命令行工具。具体就是jdk.scripting.nashorn及jdk.scripting.nashorn.shell这两个模块被移除了。

并且在 Java 11 中取以代之的是 GraalVM。GraalVM 是一个运行时平台，它支持 Java 和其他基于 Java 字节码的语言，但也支持其他语言，如 JavaScript，Ruby，Python 或 LLVM。性能是 Nashorn 的2倍以上。

> 扩展阅读：[Nashorn JavaScript Engine](https://openjdk.java.net/jeps/174)，[Deprecate the Nashorn JavaScript Engine](https://openjdk.java.net/jeps/335)



## 15、新增、移除与废弃项

### 1、新增及优化项

- 在 CharSequence 中添加了 isEmpty 默认方法。

- TreeMap 重新实现压倒性一切的如：putIfAbsent、computeIfAbsent、computeIfPresent、compute、merage。

- 支持 Unicode 13.0：此版本将Unicode支持升级到13.0，其中包括:java.lang.Character13.0水平，这增加了13.0 5930个字符，总共143859个字符类支持Unicode字符数据库。这些增加的内容包括4个新脚本，总共154个脚本，以及55个新表情符号字符。

- 优化空子字符串处理 String.substring：在某些情况下，String.substring返回""，但在子字符串长度为零时，在所有情况下都可以进行改进。

- 支持货币分组分隔符：DecimalFormat / DecimalFormatSymbols类现在可以处理货币值的分组分隔符。例如，在奥地利（de-AT语言环境）中使用的德语语言的货币分组分隔符为“。”，而在其他德语语言环境中的货币分组分隔符为“。”。

- time 用默认值覆盖本地化值：java.time.format.DateTimeFormatter.localizedBy(Locale)方法现在采用默认的语言环境值，例如Chronology和/或DecimalStyle指定的语言环境参数。

  ```bash
  # 在先前的 JDK 版本中：
  jshell> DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL)
      .localizedBy(Locale.forLanguageTag("fa"))
      .format(LocalDate.now())
  $3 ==> "جمعه 1 مهٔ 2020"
  
  # 在 JDK 15 中，数字是阿拉伯文（西文）数字：
  jshell> DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL)
      .localizedBy(Locale.forLanguageTag("fa"))
      .format(LocalDate.now())
  $3 ==> "جمعه ۱ مهٔ ۲۰۲۰"
  ```

  > 这些数字使用扩展阿拉伯语 - 印度数字，因为它是波斯语区域设置的默认编号系统。

- time ValueRange.of：`ValueRange.of（long min，long maxSmallest，long maxLargest）`，如果最小值大于最小最大值，则将引发异常。但是，仅当最小值大于最大最大值时才会发生例外。

- 集合性能提升（Better Listing of Arrays）：复制集合的首选方法是使用“复制构造函数”。例如，要将集合复制到新的ArrayList中，可以编写new ArrayList<>(collection)。在某些情况下，可能会制作其他临时副本，如果要复制的集合非常大，则应用程序应（意识到/监视）制作副本所需的大量资源。

- 性能改进 InflaterOutputStream.write：

- 从JDK 15开始，通过`InflaterOutputStream(OutputStream out, Inflater infl,int bufLen)`所指定的缓冲区大小将在对的调用中使用`InflaterOutputStream.write(byte[] b,int off, int len)`。如果在调用 InflaterOutputStream 构造函数时未指定缓冲区大小，则默认为512字节。



### 2、移除项

- Java 14 JEP 362 弃用了 Solaris/SPARC、Solaris/x64 和 Linux/SPARC 端口，JEP 381：现在它在 Java 15 中被正式删除。

  近年来，Solaris 和 SPARC 都已被 Linux 操作系统和英特尔处理器取代。放弃对 Solaris 和 SPARC 端口的支持，将使 OpenJDK 社区的贡献者们能够加速开发新功能，从而推动平台向前发展。

- Obsolete -XX:UseAdaptiveGCBoundary，淘汰了 -XX:UseAdativeGCBoundary

- 移除 Nashorn JS 引擎



### 3、废弃项

- Deprecated -XX:ForceMUMA Option ,废弃了ForceNUMA选项

- Disable Native SunEC Implementation by Default 默认禁用了Native SunEC Implementation

- JEP 385：废弃 RMI 激活机制，只是废弃 RMI 激活机制，不影响 RMI 其他功能。

  RMI Activation（延迟激活：延迟激活对象，推迟到客户第一次使用之前）被标记为删除，在未来的版本中将会删除，自 Java 8 依赖一直是可选的，而不是必选项目。RMI激活机制增加了持续的维护负担，RMI的其他部分暂时不会被弃用。

  对于现在应用程序来说。分布式系统大部都是基于Web的，Web 服务器已经解决了穿越防火墙，过滤请求，身份验证和安全性问题，并且也提供了很多延迟加载的技术。所以在现代引用程序中，RMIActivation 已经很少用了，并且在各种开源代码库中，也基本上找不见了。





## 16、参考文献 & 鸣谢

1. Java 新特性文档：https://www.wdbyte.com/java-feature/
2. Java New Feature：https://javaguide.cn/java/new-features/
3. Java 的新特性：https://segmentfault.com/a/1190000004417288/
4. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
5. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/
