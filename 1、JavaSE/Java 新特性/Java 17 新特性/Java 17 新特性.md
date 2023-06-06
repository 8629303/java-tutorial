# Java 17 新特性

Java 17 在 2021 年 9 月 14 日正式发布，Java 17 是一个长期支持（LTS）版本。

本次更新共带来 14 个新特性：

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

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 17 新特性：https://openjdk.org/projects/jdk/17/
- OracleJDK Java 17 新特性：https://www.oracle.com/java/technologies/javase/17-relnote-issues.html#NewFeature
- Oracle JDK 17 文档：https://docs.oracle.com/en/java/javase/17/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## 1、JEP 409：Sealed 密封类【正式版】

Sealed Classes 密封类允许描述哪个类或接口可以扩展或实现这个类或接口。在 Java 15 新特性中首次引入（使用详情请参考 Java 15 新特性教程），在 Java 16 中进行二次预览（更加严格的引用检查和密封类的继承关系），在 Java 17 这个版本中终于正式转正了。

发展历史：

1. 密封类是由 JEP 360 首次提出的，并在 Java 15 中作为 预览功能提供。
2. 然后由 JEP 397 再次提出并进行了改进，并作为预览功能在 Java 16 中提供。
3. 最后在 JEP 409 建议在 Java 17 中完成密封类，与 Java 16 没有任何变化。

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



## 2、JEP 406：Switch 模式匹配【预览】

在 Java 14 中新增 instanceof 模式匹配是一个非常赞的一个新特性，这次 Java 17 为 Switch 也增加了类型匹配自动转换功能。

使用 Switch 表达式和语句的模式匹配以及对模式语言的扩展来增强 Java 编程语言。扩展 instanceof 模式匹配以 Switch 允许针对多个模式测试表达式，每个模式都有特定的操作，以便可以简洁安全地表达复杂的面向数据的查询。

此提案的目标如下：

- Switch 通过允许模式出现在 case 标签中来扩展表达式和语句的表现力和适用性。
- 允许在 Switch 需要时放松对历史的零敌意。
- 引入两种新的模式：保护模式，允许使用任意布尔表达式细化模式匹配逻辑，以及带 括号的模式，以解决一些解析歧义。
- 确保所有现有的 Switch 表达式和语句继续编译而不做任何更改并以相同的语义执行。
- 不要引入 Switch 与传统 Switch 构造分离的具有模式匹配语义的新式表达式或语句。
- Switch 当 case 标签是模式与 case 标签是传统常量时，不要使表达式或语句的行为不同。
- instanceof 模式匹配还增加了 Record 类的匹配（实际上看起来跟 Switch 关系不大，不过也可以在 Switch 中配置 Record 类）。

1、新老 instanceof 代码示例：

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

2、不支持模式匹配的 Switch（需要提前考虑 null 判断）

```java
static String formatter(Object o) {
    if (o == null) {
        System.out.println("oops!");
        return;
    }
    String formatted = "unknown";
    if (o instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (o instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (o instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (o instanceof String s) {
        formatted = String.format("String %s", s);
    }
    return formatted;
}
```

3、支持模式匹配的 Switch（需要提前考虑 null 判断）

```java
static String formatterPatternSwitch(Object o) {
    return switch (o) {
        case null      -> System.out.println("Oops");
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> o.toString();
    };
}
```

直接在 Switch 上支持 Object 类型，这就等于同时支持多种类型，使用模式匹配得到具体类型，大大简化了语法量，这个功能还是挺实用的， 目前看转正只是一个时间上的问题而已。



## 3、JEP 414：Vector API【二次孵化】

Java 17 中我们在此建议结合改进以响应反馈以及性能改进和其他重要的实施改进。我们包括以下显着变化：

- 增强 API 以支持字符操作，例如 UTF-8 字符解码。具体来说，我们添加了在 short向量和 char数组之间复制字符的方法，以及用于与整数向量进行无符号比较的新向量比较运算符。
- 用于将 byte向量与 boolean 数组相互转换的 API 的增强功能。
- 使用英特尔的短向量数学库 (SVML)对 x64 上的超越和三角泳道运算的内在支持。
- Intel x64 和 ARM NEON 实现的一般性能增强。

向量（Vector） API 孵化历史：

1. 向量（Vector） API 最初由 JEP 338 提出，并作为孵化 API 集成到 Java 16 中。它可以在运行时可靠的编译为支持的 CPU 架构，从而实现更优的计算能力。
2. 第二轮孵化由 JEP 414 提出并集成到 Java 17 中，改进了 Vector API 性能，增强了例如对字符的操作、字节向量与布尔数组之间的相互转换等功能。
3. 第三轮孵化由 JEP 417 提出并集成到 Java 18 中，第四轮由 JEP 426 提出并集成到了 Java 19 中。

该孵化器 API 提供了一个 API 的初始迭代以表达一些向量计算，这些计算在运行时可靠地编译为支持的 CPU 架构上的最佳向量硬件指令，从而获得优于同等标量计算的性能，充分利用单指令多数据（SIMD）技术（大多数现代 CPU 上都可以使用的一种指令）。尽管 HotSpot 支持自动向量化，但是可转换的标量操作集有限且易受代码更改的影响。该 API 将使开发人员能够轻松地用 Java 编写可移植的高性能向量算法。



## 4、JEP 415：指定上下文的反序列化过滤器

Java 对象序列化是一个非常重要的功能，如果没有序列化功能，Java 可能都不会占据开发语言的主导地位，序列化让远程处理变得容易和透明，也促进了 JavaEE 的成功。

序列化过程没有问题（把对象转换为可以在 JVM 之间自由传输），但是反序列化过程可能存在危险：

- 许多情况下传入的数据流内容是通过未知或未经身份验证的客户端获取的；
- 序列化数据流中可能携带带有攻击者精心构造的恶意代码。

终于在 Java17（JEP 415）中增加了反序列化过滤器，允许应用程序使用 JVM 范围的过滤器工厂，配置特定于上下文和动态选择的反序列化过滤器，该工厂用于为每个反序列化操作选择一个过滤器。

简单说：就是通过一个过滤配置，来告知本次反序列化允许或者禁止操作的类，反序列化时碰到被禁止的类，则会反序列化失败。

过滤器工厂可以为整个应用程序返回一个固定的过滤器。例如，这里有一个过滤器，它允许示例类，允许 java.base模块中的类，并拒绝所有其他类：

```java
var filter = ObjectInputFilter.Config.createFilter("example.*;java.base/*;!*")
```

【操作示例 1：反序列化】假设 Dog 类中的 Poc 是恶意构造的类，但是正常反序列化是可以成功的。

```java
package com.java17;
import java.io.*;

class Poc implements Serializable {}
class Dog implements Serializable {
    private String name;
    private Poc    poc;

    public Dog(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Poc getPoc() {
        return poc;
    }

    public void setPoc(Poc poc) {
        this.poc = poc;
    }

    @Override
    public String toString() {
        return "Dog {" + "name = '" + name + '\'' + '}';
    }

}

public class JEP415 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Dog dog = new Dog("哈士奇");
        dog.setPoc(new Poc());
        // 序列化 - 对象转字节数组
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream)) {
            objectOutputStream.writeObject(dog);
        }
        byte[] bytes = byteArrayOutputStream.toByteArray();
        // 反序列化 - 字节数组转对象
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        ObjectInputStream    objectInputStream    = new ObjectInputStream(byteArrayInputStream);
        Object               object               = objectInputStream.readObject();
        System.out.println(object.toString());
    }
}
```

```
Dog {name = '哈士奇'}
```

【操作示例 2：反序列化过滤器】在 Java 17 中可以自定义反序列化过滤器，拦截不允许的类。

```java
package com.java17;
import java.io.*;

public class JEP415 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Dog dog = new Dog("哈士奇");
        dog.setPoc(new Poc());
        // 序列化 - 对象转字节数组
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream)) {
            objectOutputStream.writeObject(dog);
        }
        byte[] bytes = byteArrayOutputStream.toByteArray();
        // 反序列化 - 字节数组转对象
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        ObjectInputStream    objectInputStream    = new ObjectInputStream(byteArrayInputStream);
        // 允许 com.java17.Dog 类，允许 java.base 中的所有类，拒绝其他任何类 ;等于and的意思
        ObjectInputFilter filter = ObjectInputFilter.Config.createFilter("com.java17.Dog;java.base/*;!*");
        objectInputStream.setObjectInputFilter(filter);
        // 从输入流中读取数据, 会在此行报错
        Object object = objectInputStream.readObject();
        System.out.println(object.toString());
    }
}

class Poc implements Serializable {}
class Dog implements Serializable {
    private String name;
    private Poc    poc;

    public Dog(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Poc getPoc() {
        return poc;
    }
    public void setPoc(Poc poc) {
        this.poc = poc;
    }
    @Override
    public String toString() {
        return "Dog {" + "name = '" + name + '\'' + '}';
    }
}
```

```java
Exception in thread "main" java.io.InvalidClassException: filter status: REJECTED
	at java.base/java.io.ObjectInputStream.filterCheck(ObjectInputStream.java:1412)
	at java.base/java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:2053)
	at java.base/java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1907)
	at java.base/java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2209)
	at java.base/java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1742)
	at java.base/java.io.ObjectInputStream$FieldValues.<init>(ObjectInputStream.java:2584)
	at java.base/java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2442)
	at java.base/java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2242)
	at java.base/java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1742)
	at java.base/java.io.ObjectInputStream.readObject(ObjectInputStream.java:514)
	at java.base/java.io.ObjectInputStream.readObject(ObjectInputStream.java:472)
	at com.java17.JEP415.main(JEP415.java:21)
```

如果想不报错则这样修改即可：`ObjectInputFilter filter = ObjectInputFilter.Config.createFilter("com.**;java.base/*;!*");`

【操作示例 3】使用 Record 类来测试 操作示例 2 的操作。

```java
package com.java17;
import java.io.*;

public class JEP415 {
    public static void main(String[] args) throws Exception {
        // 如下Record类可以定义在方法体,也可以定义在类体中
        record PocRecord() implements Serializable {}
        record DogRecord(String name, PocRecord pocRecord) implements Serializable {}
        
        // 初始化数据
        DogRecord dog = new DogRecord("哈士奇", new PocRecord());
        
        // 序列化 - 对象转字节数组
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream)) {
            objectOutputStream.writeObject(dog);
        }
        
        // 反序列化 - 字节数组转对象
        byte[] bytes = byteArrayOutputStream.toByteArray();
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
             ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
            // 允许 com.java17.Dog 类，允许 java.base 中的所有类，拒绝其他任何类 ;等于and的意思
            ObjectInputFilter filter = ObjectInputFilter.Config.createFilter("com.java17.*;java.base/*;!*");
            objectInputStream.setObjectInputFilter(filter);
            // 从输入流中读取数据, 会在此行报错
            Object object = objectInputStream.readObject();
            System.out.println(object.toString());
        }
    }
}
```

```
DogRecord[name=哈士奇, pocRecord=PocRecord[]]
```



## 5、JEP 306：恢复始终严格的浮点语义

既然是恢复严格的浮点语义，那么说明在某个时间点之前，是始终严格的浮点语义的。其实在 Java SE 1.2 之前，所有的浮点计算都是严格的，但是以当初的情况来看，过于严格的浮点计算在当初流行的 x86 架构和 x87 浮点协议处理器上运行，需要大量的额外的指令开销，所以在 Java SE 1.2 开始，需要手动使用关键字 **strictfp**（strict float point） 才能启用严格的浮点计算。

但是在 202x 年的今天，硬件早已发生巨变，当初的问题已经不存在了，所以从 Java 17 开始，恢复了始终严格的浮点语义这一特性。目标：简化数字敏感库开发，包括java.lang.Math和java.lang.StrictMath

**扩展**：***strictfp*** 是 Java 中的一个关键字，大多数人可能没有注意过它，它可以用在类、接口或者方法上，被 strictfp 修饰的部分中的 float 和 double 表达式会进行严格浮点计算。

【操作示例】testStrictfp() 方法被 strictfp 关键字修饰。

```java
public class Main {
    public static void main(String[] args) {
        testStrictfp(); // sum: 1.5555555754237704
    }

    // 在 Java 17 中会提示如下信息, 因为 Java 17 中已经默认开启严格模式
    // Modifier 'strictfp' is redundant on Java 17 and later 
    public strictfp static void testStrictfp() {
        float  aFloat  = 0.6666666666666666666f;
        double aDouble = 0.88888888888888888d;
        double sum     = aFloat + aDouble;
        System.out.println("sum: " + sum);
    }
}
```



## 6、JEP 356：增强型伪随机数生成器

为伪随机数生成器 RPNG（pseudorandom number generator）增加了新的接口类型和实现，让在代码中使用各种 PRNG 算法变得容易许多。

这次增加了 RandomGenerator 接口，为所有的 PRNG 算法提供统一的 API，并且可以获取不同类型的 PRNG 对象流。同时也提供了一个新类 RandomGeneratorFactory 用于构造各种 RandomGenerator 实例，在 RandomGeneratorFactory 中使用 ServiceLoader.provider 来加载各种 PRNG 实现。

【操作示例】随便选择一个 PRNG 算法生成 5 个 10 以内的随机数。

```java
import java.util.random.RandomGenerator;
import java.util.random.RandomGeneratorFactory;

public class JEP356 {
    public static void main(String[] args) {
        RandomGeneratorFactory<RandomGenerator> randomFactory = RandomGeneratorFactory.of("L128X256MixRandom");
        // 使用时间戳作为随机数种子
        RandomGenerator randomGenerator = randomFactory.create(System.currentTimeMillis());
        for (int i = 0; i < 5; i++) {
            System.out.println(randomGenerator.nextInt(10));
        }
    }
}
```

```
8
0
5
5
0
```

你也可以遍历出所有的 PRNG 算法。

```java
import java.util.random.RandomGeneratorFactory;

public class JEP356 {
    public static void main(String[] args) {
        RandomGeneratorFactory.all().forEach(factory -> 
                System.out.println(factory.group() + ":" + factory.name()));
    }
}
```

```
LXM:L32X64MixRandom
LXM:L128X128MixRandom
LXM:L64X128MixRandom
Legacy:SecureRandom
LXM:L128X1024MixRandom
LXM:L64X128StarStarRandom
Xoshiro:Xoshiro256PlusPlus
LXM:L64X256MixRandom
Legacy:Random
Xoroshiro:Xoroshiro128PlusPlus
LXM:L128X256MixRandom
Legacy:SplittableRandom
LXM:L64X1024MixRandom
```

可以看到 Legacy:Random 也在其中，新的 API 兼容了老的 Random 方式，所以你也可以使用新的 API 调用 Random 类生成随机数。

```java
import java.util.random.RandomGenerator;
import java.util.random.RandomGeneratorFactory;

public class JEP356 {
    public static void main(String[] args) {
        // 使用 Random
        RandomGeneratorFactory<RandomGenerator> l128X256MixRandom = RandomGeneratorFactory.of("Random");
        // 使用时间戳作为随机数种子
        RandomGenerator randomGenerator = l128X256MixRandom.create(System.currentTimeMillis());
        for (int i = 0; i < 5; i++) {
            System.out.println(randomGenerator.nextInt(10));
        }
    }
}
```

```
8
9
5
3
7
```

> 扩展阅读：增强的伪随机数生成器：https://openjdk.java.net/jeps/356



## 7、JEP 382：使用新的 macOS 渲染管线

macOS 为了提高图形的渲染性能，在 2018 年 9 月抛弃了之前的 OpenGL 渲染库 ，而使用了 Apple Metal 进行代替。Java 17 这次更新开始支持 Apple Metal，不过对于 API 没有任何改变，这一些都是内部修改。

> 扩展阅读：macOS Mojave 10.14 Release Notes：https://developer.apple.com/documentation/macos-release-notes/macos-mojave-10_14-release-notes



## 8、JEP 391：macOS/AArch64 端口

起因是 Apple 在 2020 年 6 月的 WWDC 演讲中宣布，将开启一项长期的将 Macintosh 计算机系列从 x64 过度到 AArch64 的长期计划，因此需要尽快的让 JDK 支持 macOS/AArch64 ，将 JDK 移植到 macOS/AArch64。

Linux 上的的 AArch64 端口（JEP 237）已经存在，Linux 上的 AArch64 支持以及在 Java 16 时已经支持（可以查看之前的文章了解。Java 16 新特性 - JEP 386），Windows 的 AArch64 端口（JEP 388）的工作正在进行中。



## 9、JEP 398：删除已弃用的 Applet API

Applet API 在 Java 9 的 JEP 289 时已经标记了废弃，现在 Java 17 中将彻底删除。

Applet 是使用 Java 编写的可以嵌入到 HTML 中的小应用程序，嵌入方式是通过普通的 HTML 标记语法，由于早已过时，几乎没有场景在使用了。它基本上无关紧要，因为所有 Web 浏览器供应商都已取消对 Java 浏览器插件的支持或宣布了这样做的计划。 

操作示例：嵌入 Hello.class

```html
<applet code="Hello.class" height=200 width=200></applet>
```

弃用或移除标准 Java API 的这些类和接口：

- java.applet.Applet
- java.applet.AppletStub
- java.applet.AppletContext
- java.applet.AudioClip
- javax.swing.JApplet
- java.beans.AppletInitializer

弃用（删除）引用上述类和接口的任何 API 元素，包括以下中的方法和字段：

- java.beans.Beans
- javax.swing.RepaintManager
- javax.naming.Context



## 10、JEP 403：更强的 JDK 内部封装

如 Java 16 的 JEP 396 中描述的一样，为了提高 JDK 的安全性，使 `--illegal-access` 选项的默认模式从允许更改为拒绝。通过此更改，JDK 的内部包和 API（[关键内部 API](https://openjdk.java.net/jeps/260#Description) 除外）将不再默认打开。

但是在 Java 17 中，使用 `--illegal-access` 命令也不能打开 JDK 内部的强封装模式了，除了 sun.misc.Unsafe API。

在 Java 17 中使用 `--illegal-access` 选项将会得到一个命令已经移除的警告。

```bash
➜  bin ./java -version
openjdk version "17" 2021-09-14
OpenJDK Runtime Environment (build 17+35-2724)
OpenJDK 64-Bit Server VM (build 17+35-2724, mixed mode, sharing)
➜  bin ./java --illegal-access=warn
OpenJDK 64-Bit Server VM warning: Ignoring option --illegal-access=warn; support was removed in 17.0
```

> 扩展阅读：Java 16 新特性 - JEP 403：更强的 JDK 内部封装：https://openjdk.java.net/jeps/403



## 11、JEP 407：删除 RMI 激活

RMI（Remote Method Invocation）Activation 在 Java 15 中的 JEP 385 已经被标记为过时废弃，至今没有收到不良反馈，因此决定在 Java 17 中正式移除。但是 RMI 其他部分不会受影响。

Java EE 平台包含一项称为 JavaBeans Activation Framework (JAF) 的技术。作为 Eclipse EE4J 计划的一部分，它后来更名为 Jakarta Activation。JavaBeans Activation 和 Jakarta Activation 技术与 RMI Activation 完全无关，它们不受从 Java SE 中删除 RMI Activation 的影响。

具体描述：

- java.rmi.activation 从 Java SE API 规范中删除包
- 更新 RMI 规范以删除提及 RMI 激活
- 去掉实现RMI激活机制的 JDK 库代码
- 删除 RMI 激活机制的 JDK 回归测试
- 删除 JDK 的 rmid 激活守护进程及其文档



## 12、JEP 410：删除实验性 AOT 和 JIT 编译器

在 Java 9 的 JEP 295 中，引入了实验性的提前编译 jaotc 工具，但是这个特性自从引入依赖用处都不太大，而且需要大量的维护工作，所以在 Java 17 中决定删除这个特性。

主要移除了三个 JDK 模块：

1. jdk.aot - jaotc 工具。
2. Jdk.internal.vm.compiler - Graal 编译器。
3. jdk.internal.vm.compiler.management - 格拉尔的 MBean。

保留这两个 Graal 相关的源文件，以便 JVMCI 模块 ( jdk.internal.vm.ci, JEP 243 ) 继续构建：

1. `src/jdk.internal.vm.compiler/share/classes/module-info.java`
2. `src/jdk.internal.vm.compiler.management/share/classes/module-info.java`

同时也移除了部分与 AOT 编译相关的 HotSpot 代码：

1. `src/hotspot/share/aot` — dumps and loads AOT code
2. Additional code guarded by `#if INCLUDE_AOT`

最后，删除测试以及与 Graal 和 AOT 编译相关的 makefile 中的代码。



## 13、JEP 411：弃用安全管理器以进行删除

**Security Manager** 在 JDK 1.0 时就已经引入，但是它一直都不是保护服务端以及客户端 Java 代码的主要手段，为了 Java 的继续发展，决定弃用 Security Manager，以便与旧 Applet API ( JEP 398 ) 在不久的未来进行删除。

```java
@Deprecated(since="17", forRemoval=true)
public class SecurityManager {
    // ...
}
```



## 14、JEP 412：外部函数和内存 API【首次孵化】

新的 API 允许 Java 开发者与 JVM 之外的代码和数据进行交互，通过调用外部函数，可以在不使用 JNI 的情况下调用本地库。

发展历史如下：

- Java 14 [JEP 370](https://openjdk.java.net/jeps/370) 引入了外部内存访问 API（孵化器）
- Java 15 [JEP 383](https://openjdk.java.net/jeps/383) 引入了外部内存访问 API（第二孵化器）
- Java 16 [JEP 389](https://openjdk.java.net/jeps/389) 引入了外部链接器 API（孵化器）
- Java 16 [JEP 393](https://openjdk.java.net/jeps/393) 引入了外部内存访问 API（第三孵化器）
- Java 17 [JEP 412](https://openjdk.java.net/jeps/412) 引入了外部函数和内存 API（孵化器）

这是一个孵化功能；需要添加 --add-modules jdk.incubator.foreign 来编译和运行 Java 代码。因为 FFM API （外部函数和内存 API）位于模块的 jdk.incubator.foreign 包中 jdk.incubator.foreign。

作为使用 FFM API 的简短示例，这里是 Java 代码，它获取 C 库函数的方法句柄 `radixsort`，然后使用它对 Java 数组中的四个字符串进行排序（省略了一些细节）：

```java
// 1. Find foreign function on the C library path
MethodHandle radixSort = CLinker.getInstance().downcallHandle(CLinker.systemLookup().lookup("radixsort"), ...);
// 2. Allocate on-heap memory to store four strings
String[] javaStrings   = { "mouse", "cat", "dog", "car" };
// 3. Allocate off-heap memory to store four pointers
MemorySegment offHeap  = MemorySegment.allocateNative(
                             MemoryLayout.ofSequence(javaStrings.length, CLinker.C_POINTER), ...);
// 4. Copy the strings from on-heap to off-heap
for (int i = 0; i < javaStrings.length; i++) {
    // Allocate a string off-heap, then store a pointer to it
    MemorySegment cString = CLinker.toCString(javaStrings[i], newImplicitScope());
    MemoryAccess.setAddressAtIndex(offHeap, i, cString.address());
}
// 5. Sort the off-heap data by calling the foreign function
radixSort.invoke(offHeap.address(), javaStrings.length, MemoryAddress.NULL, '\0');
// 6. Copy the (reordered) strings from off-heap to on-heap
for (int i = 0; i < javaStrings.length; i++) {
    MemoryAddress cStringPtr = MemoryAccess.getAddressAtIndex(offHeap, i);
    javaStrings[i] = CLinker.toJavaStringRestricted(cStringPtr);
}
assert Arrays.equals(javaStrings, new String[] {"car", "cat", "dog", "mouse"});  // true
```

这段代码比任何使用 JNI 的解决方案都清晰得多，因为隐藏在 native 方法调用后面的隐式转换和内存取消引用现在直接用 Java 表示。也可以使用现代 Java 习语；例如：流可以允许多个线程在堆上和堆外内存之间并行复制数据。



## 15、参考文献 & 鸣谢

1. Java 新特性文档：https://www.wdbyte.com/java-feature/
2. Java New Feature：https://javaguide.cn/java/new-features/
3. Java 的新特性：https://segmentfault.com/a/1190000004417288/
4. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
5. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/