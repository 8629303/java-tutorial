# Java 14 新特性

Java 14 在 2020 年 03 月 17 日发布，它不是一个长久支持（LTS）版本。

这次更新共带来 16 个新特性：

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

具体新增特性的介绍：

1. 语言特性7项目：
   - Switch表达式（标准）
   - 友好的空指针异常
   - 非易失性字节缓冲区
   - record 关键字
   - instanceof 模式匹配
   - 文本块改进 二次预览
   - 外部存储API
2. 垃圾回收调整：
   - G1的NUMA内存分配优化
3. 新增工具：
   - JAVA打包工具 孵化
   - JFR事件流
4. 增加废弃和移除：
   - MacOS系统上的ZGC试验
   - windows系统上的ZGC实验
   - 弃用Parallel Scavenge 和Serial Old垃圾收集算法
   - 弃用Solaris和SPCRC端口
   - 移除CMS垃圾收集器
   - 删除Pack200工具和API

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 14 新特性：https://openjdk.org/projects/jdk/14/
- OracleJDK Java 14 新特性：https://www.oracle.com/java/technologies/javase/14-relnote-issues.html#NewFeature
- Oracle JDK 14 文档：https://docs.oracle.com/en/java/javase/14/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/





## 1、Switch【正式版】

与之前的 Java 12、13 功能一样，只不过确定下来为最终版。

Java Switch 语句是一个一直在变化的语法，可能是因为之前的不够强大，简单总结 Switch 语句在各个版本中的特点：

- Java 5 Switch 变量类型可以使用枚举了
- Java 7 Switch 变量类型中可以使用 String 字符串
- Java 12 Switch 语句可以自动省略 break 导致的贯穿提示警告 case L ->，可以作为表达式，用变量接收结果，可以省略 break
- Java 13 Switch 中可以使用 yield 关键字停止 switch 语句块


Java 14 JEP361：Switch 表达式（标准）是独立的，不依赖于 JEP 325 和 JEP 354，也就是说这里开始，之前学习的 Switch 语句的语法成为一个正式的标准。未来是否有更多的改进，我们可以拭目以待。

1、Java 12 对缺省 break 的贯穿弱点进行了改进，case: 改成 case L ->，这样即使不写也不会贯穿了，而且可以作为表达式返回结果。

```java
var grade ="a";
var res = switch(grade){
    case "a" -> "优秀";
    case "b" -> "良好";
    case "c" -> "一般";
    case "d" -> "及格";
    default -> "no such grade";
}
```

2、Java 12 也可以进行多值匹配的支持，无需冗余写很多个 case：

```java
var grade ="a";
var res = switch(grade){
    case "a","b" -> "优秀";
    case "c"     -> "一般";
    case "d"     -> "及格";
    default      -> "no such grade";
}
```

3、Java 13 开始可以使用 yield 关键字返回结果，这里的 case 后面仍然是表达式，不过可以是复杂的表达式了：

```java
String x = "3";
int i = switch (x) {
    case "1":
        yield 1;
    case "2":{
        System.out.println("");
        yield 2;
    }
    default:
        yield 3;
};
```



## 2、多行文本块【第二次预览】

文本块是 Java 13 引入的语法，在 Java14 中，文本块依然是预览特性，不过，其引入了两个新的转义字符：

- `\` : 表示行尾，不引入换行符
- `\s` ：表示单个空格

```java
String content = 
        """
        {
            "upperSummary": null,\
            "sensitiveTypeList": null,
            "gmtModified": "2011-08-05\s10:50:09",
        }
         """;
System.out.println(content);
```

```json
{
    "upperSummary": null,    "sensitiveTypeList": null,
    "gmtModified": "2011-08-05 10:50:09",
}
```

> 多行文本块功能在 Java 15 中正式发布。



## 3、instanceof 增强【首次预览】

instanceof 的模式匹配：以往我们使用 instanceof 运算符都是先判断，然后在进行强转，可以查看 String 的 equals 方法源码。

在 Java 14 之前，使用 instanceof 进行类型判断之后，需要进行对象类型转换后才能使用。

```java
public class Java14BeaforInstanceof {
    public static void main(String[] args) {
        Object obj = "Hello Java";
        if (obj instanceof String) {
            String str = (String) obj;
            System.out.println(str);
        }
    }
}
```

而在 Java 14 中，可以在判断类型时指定变量名称进行类型转换，方便了使用。

```java
public class Java14Instanceof {
    public static void main(String[] args) {
        Object obj = "Hello Java";
        if (obj instanceof String str) {
            System.out.println(obj);
        }
    }
}
```



## 4、Record 关键字【首次预览】

Record 类型是 Java 14 中的预览函数引入的一种新型的类型声明，并且应作为普通的不可变数据类，用于在类和应用程序之间进行数据传输。像枚举 Enum  一样，Record 也是一个特殊的类输入 Java。它旨在用于仅创建类以充当普通数据载体的地方。Record 的使用获得了很大程度的简洁性。使用它可以替代构造器、getter 方法（类似）、equal 方法、toString 方法，hashCode 方法

类（Class）与记录（Record）之间的重要区别是，Record 旨在消除设置和从实例获取数据所需的所有代码，Record 将这种责任转移给生成编译器。我们可以在 record 定义中覆盖上面提供的任何默认方法，以实现自定义行为。



### 1、Record 语法

使用 record 关键字创建此类 Record 类。就像我们在构造函数中所做的一样，我们需要在 Record 中提及属性及其类型。

```java
public record UserRecord(Long id, String name, int age) {}
```



### 2、Record 使用

要创建一条Record，请调用其构造函数并在其中传递所有字段信息。然后，我们可以使用 JVM 生成的 getter 方法获取记录信息，并调用任何生成的方法。

```java
/**
 * 1. 使用record声明的类会自动拥有上面类中的三个方法
 * 2. 在这基础上还附赠了equals()，hashCode()方法以及toString()方法
 * 3. toString方法中包括所有成员属性的字符串表示形式及其名称
 */
record UserRecord(Long id, String name, int age) {}
public class Main {
    public static void main(String[] args) {
        UserRecord record = new UserRecord(1L,  "Sam", 18);
        System.out.println(record.id() + ", " + record.name() + ", " + record.age()); // 1, Sam, 18
        System.out.println(record);        // UserRecord[id=1, name=Sam, age=18]
    }
}
```



### 3、Record 查看

当我们创建 UserRecord 时，编译器会创建字节码，并在生成的类文件中包含以下内容：

1. 自动生成的 private final field（注意：当 Record 作为内部类的时候可以直接 record.field 获取值，这属于内部类的知识）
2. 自动生成的 public getter 方法（这种 getter 并不是 getxxx()，而是直接用变量名命名）
3. 自动生成的全属性构造器
4. 自动生成的 hashCode()，equals()，toString() 方法：
   1. 从字节码可以看出，这三个方法的底层实现是 invokeDynamic 另一个方法
   2. invokeDynamic 实际调用的是 ObjectMethods.java 这个类中的 bootstrap 方法
5. 类扩展了 java.lang.Record ，它是所有 Record 的基类。这意味着 Record 不能扩展其他类。
6. 该类被标记为 final ，这意味着我们无法创建它的子类。
7. 它没有任何 setter 方法，这意味着 Record 实例被设计为不可变的。

如果在生成的类文件上运行 javap 工具，我们将看到该类文件。

```bash
$ javac UserRecord.java && javap UserRecord
```

```java
/**
 * 如果一个类具有两个特征
 * 1. 所有成员属性都是final
 * 2. 全部方法由构造方法，和两个成员属性访问器组成（共三个）
 * 那么这种类就很适合使用record来声明
 */
public final class UserRecord extends java.lang.Record {
    
    public UserRecord(java.lang.Long, java.lang.String, int);

    public java.lang.String toString();

    public final int hashCode();
    public final boolean equals(java.lang.Object);

    public java.lang.Long id();
    public java.lang.String name();
    public int age();
}
```



### 4、Record 不能被继承

尽管所有 Record 都扩展了java.lang.Record 类，但是我们仍然不能显式创建 java.lang.Record 的子类。编译器不会通过。

```java
final class Data extends Record {
    private final int unit;
}

// Compiler error : The type Data may not subclass Record explicitly
```

这意味着获取 Record 的唯一方法是显式声明一个Record并让 javac 创建类文件。



### 5、Record 添加注解

我们可以为 Record 的组件添加注释，这些注释适用于它们。例如，我们可以将 @Transient 批注应用于 id 字段。

```java
public record UserRecord(@Transient Long id, String firstName, String lastName, int age) {}
```



### 6、Record 序列

Record 类的 serialVersionUID 是 **0 L**，除非它是明确声明。

Record 对象序列化的过程无法自定义；Record 类定义的任何特定于类的代码 writeObject，readObject，readObjectNoData，readResolve，writeExternal和readExternal 方法在序列化和反序列化期间都将被忽略。但是，writeReplace方法可用于返回要序列化的替代对象。

在执行任何 序列化或反序列化之前，我们必须确保Record必须可序列化或可外部化。

```java
import java.io.*;

record UserRecord(Long id, String name, int age) implements Serializable {}
public class RecordTest {
    public static void main(String[] args) throws Exception {
        UserRecord userRecord = new UserRecord(1L, "Sam", 18);
        // 序列化 - 对象转字节数组
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream)) {
            objectOutputStream.writeObject(userRecord);
        }

        byte[] bytes = byteArrayOutputStream.toByteArray();
        // 反序列化 - 字节数组转对象
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);) {
            Object object = objectInputStream.readObject();
            System.out.println(object.toString());
        }
    }
}
```

```
UserRecord[id=1, name=Sam, age=18]
```



### 7、Record 字段与方法

添加到 Record 中的新字段必须为 static 静态。方法可以是成员方法也可以是静态方法，且该方法可以访问 Record 字段的内部状态（注意：**不能声明 abstract 或者 native 方法**）。

编译器不会在隐式生成的字节码中使用添加的字段和方法，因此它们不属于任何方法实现，例如：equals()， hashCode() 或 toString()。我们必须根据需要明确使用它们。

```java
record UserRecord(Long id, String name, int age) implements Serializable {
    // additional field
    static boolean minor;

    // additional method
    public String test() {
        return name + " " + age;
    }
    
    // additional static method
    public static String staticTest() {
        return name + " " + age;
    }
    
    // additional private method
    private String privateTest() {
        return name + " " + age;
    }
}
```



### 8、Record 构造函数

1、我们可以添加构造器特定代码，以在紧凑的构造器中进行数据验证。我们不需要为字段指定构造函数参数的分配，因为它以通常的方式在规范构造函数中发生。

```java
public record UserRecord(Long id, String name, int age) implements Serializable {
    public UserRecord {
        if (age < 18) {
            throw new IllegalArgumentException("You belong to a minor.");
        }
    }
}
```

2、也可以明确声明以所有属性为参数并且给每个属性赋值的构造器，这个构造器需要满足：

1. 构造器参数需要包括所有属性（同名同类型），并按照 Record 类头的声明顺序。
2. 不能声明抛出任何异常（不能使用 throws）
3. 不能调用其他构造器（即不能使用 this(xxx)）
4. 构造器需要对于每个属性进行赋值。

```java
public record UserRecord(Long id, String name, int age) implements Serializable {
    public UserRecord(Long id, int age) {
        this(id, "name", age);
    }
    // 此构造器可以声明, 也可以省略不声明, 因为默认会创建一个所以属性的构造器
    public UserRecord(Long id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}
```



### 9、Record 使用注意

1. 由于用 record 声明的类，已经继承了 java.lang.Record，故 record 不能在显示的继承其它类。
2. record 类在 Java14 暂时还不能声明为 private
3. 可以声明在类级别、属性及方法级别、方法体内部

这个特性在 Java 14 中还是预览功能，在 Java 15 中再次增强，在 Java 16 中正式转正。



## 5、 NullPointerException 精准提示

NullPointerException 一直都是一个比较常见的异常，但是在 Java 14 之前，如果一行有多个表达式时，这时报了空指针后，单纯的从报错信息来看，可能并不知道是哪个对象为 NULL ，下面是一个演示。

> 首先，通过 JVM 参数中添加`-XX:+ShowCodeDetailsInExceptionMessages`，可以在空指针异常中获取更为详细的调用信息，更快的定位和解决问题。

```java
public class Main {
    public static void main(String[] args) {
        String content1 = "Hello Java";
        String content2 = null;
        int length = content1.length() + content2.length();
        System.out.println(length);
    }
}
```

在 Java 14 之前，从下面的报错中我们只能得到错误出现的行数，但是并不能确定是 conteng1 还是 content2 为 null。

```java
Exception in thread "main" java.lang.NullPointerException
	at com.alibaba.security.astralnet.console.controller.ApiChartsTest.main(Java14NullPointerExceptions.java:5)
```

但是在 Java 14 中，会清晰的告诉你 because "content2" is null 。

```java
Exception in thread "main" java.lang.NullPointerException: 
	Cannot invoke "String.length()" because "content2" is null
	at com.wdbyte.Java14NullPointerExceptions.main(Java14NullPointerExceptions.java:5)
```



## 6、打包工具【首次孵化】

在 Java 14 中，引入了打包工具，命令是 jpackage。在 Java 16 中，成为了正式功能。使用 jpackage 命令可以把 JAR 包打包成不同操作系统支持的软件格式（安装包），常见平台格式如下：

1. Linux：deb 与 rpm
2. MacOS：pkg 与 dmg
3. Windows：msi 与 exe

需要注意的是：jpackage 不支持交叉编译，也就是说在 Windows 平台上是不能打包成 MacOS 或者 Linux 系统的软件格式的。（相对于第三方工具，jpackage 只适用于比较简单的场景，不过对很多应用来说已经足够好了）

> **项目打包：非模块化项目**

1、如果有一个包含 jar 文件的应用程序，所有的应用程序都位于一个名为 lib 的目录中，并且 lib/main.jar 包含主类，可以通过如下命令打包

```bash
$ jpackage --name myapp --input lib --main-jar main.jar
```

2、将以本地系统的默认格式打包应用程序，将生成的打包文件保留到当前目录中。如果 MANIFEST.MF 文件中没有 main.jar。没有 Main-Class 属性，则必须显式指定主类：

```bash
$ jpackage --name myapp --input lib --main-jar main.jar --main-class myapp.Main
```

软件包的名称将为没有app，尽管软件包文件本身的名称将更长，并以软件包类型皆为，该软件包将包括该应用程序的启动器，也称为myapp。要启动应用程序，启动程序将会从输入目录复制的每个 JAR 文件放在 JVM 的类路径上。

3、如果您希望默认格式以外的其他格式制作软件包，请使用 --type选项。例如：要在 MacOS 上生成 pkg 文件而不是 dmg 文件

```bash
$ jpackage --name myapp --input lib --main-jar main.jar --type pkg
```

> **项目打包：模块化项目**

1、如果您有一个模块化应用程序，该程序有目录中的模块化 jar 文件或 JMOD 文件组成，并且模块中 lib 包含主类 myapp，则命令为

```bash
$ jpackage -name myapp --moudule-path lib -m myapp
```

2、如果 myapp 模块未标识主类，则必须再次明确

```bash
$ jpackage -name myapp --moudule-path lib -m myapp/myapp.Main
```

> **JPackage 指令含义**：

查看帮助：jpackage -h （以下内容出于 Windows 平台下 Java 17 版本）

```bash
用法: jpackage <options>

简单示例:
--------------
    生成适合主机系统的应用程序包:
        对于模块化应用程序:
            jpackage -n name -p modulePath -m moduleName/className
        对于非模块化应用程序:
            jpackage -i inputDir -n name --main-class className --main-jar myJar.jar
        来自预构建的应用程序映像:
            jpackage -n name --app-image appImageDir
    
    生成预构建的应用程序映像:
        对于模块化应用程序:
            jpackage --type app-image -n name -p modulePath -m moduleName/className
        对于非模块化应用程序:
            jpackage --type app-image -i inputDir -n name --main-class className --main-jar myJar.jar
        要为jlink提供自己的选项，请单独运行jlink:
            jlink --output appRuntimeImage -p modulePath --add-modules moduleName --no-header-files [<additional jlink options>...]
            jpackage --type app-image -n name -m moduleName/className --runtime-image appRuntimeImage

    生成Java运行时包:
        jpackage -n name --runtime-image <runtime-image>

通用选项:
  @<filename> 
          从文件中读取选项和(或)模式;
          此选项可多次使用.
  --type -t <type> 
          要创建的包的类型;
          有效值为: {"app-image", "exe", "msi"};
          如果未指定此选项，将创建一个依赖于平台的默认类型.
  --app-version <version>
          应用程序和(或)包的版本
  --copyright <copyright string>
          应用程序的版权
  --description <description string>
          应用描述
  --help -h 
          将包含当前平台每个有效选项的列表和描述的使用文本打印到输出流中，然后退出.
  --icon <file path>
          应用程序包图标的路径(绝对路径或相对于当前目录).
  --name -n <name>
          应用程序和(或)包的名称
  --dest -d <destination path>
          生成的输出文件所在的路径(绝对路径或相对于当前目录);
          默认为当前工作目录.
  --temp <directory path>
          用于创建临时文件的新目录或空目录的路径(绝对路径或相对于当前目录);
          如果指定，临时目录将不会在任务完成时删除，必须手动删除;
          如果没有指定，将在任务完成时创建并删除一个临时目录.
  --vendor <vendor string>
          应用程序的供应商.
  --verbose
          启用详细输出
  --version
          将产品版本打印到输出流并退出.

用于创建运行时映像的选项:
  --add-modules <module name>[,<module name>...]
          要添加的模块列表，以英文逗号(",")分隔
          这个模块列表连同主模块(如果指定)将作为--add-module参数传递给jlink.
          如果没有指定，则只使用主模块(如果指定了--module)，或者使用默认的模块集(如果指定了--main-jar).
          此选项可多次使用.
  --module-path -p <module path>...
          一个英文分号(";")分隔的路径列表
          每个路径必须是模块的目录，或者是模块jar文件的路径(每个路径都是绝对或相对于当前目录).
          此选项可多次使用.
  --jlink-options <jlink options> 
          一个以空格分隔的传递给jlink的选项列表 
          If not specified, defaults to "--strip-native-commands --strip-debug --no-man-pages --no-header-files". 
          此选项可多次使用.
  --runtime-image <directory path>
          将复制到应用程序映像中的预定义运行时映像的路径(绝对路径或相对于当前目录)
          如果没有指定--runtime-image，jpackage将运行jlink来使用选项创建运行时映像：--strip-debug、--no-header-files、--no-man-pages和--strip-native-commands.

用于创建应用程序映像的选项:
  --input -i <directory path>
          包含要打包的jar文件的输入目录的路径(绝对路径或相对于当前目录)
          输入目录中的所有文件都将打包到应用程序映像中.

用于创建应用程序启动器的选项:
  --add-launcher <launcher name>=<file path>
          启动器的名称，以及包含键、值对列表的Properties文件的路径(绝对路径或相对于当前目录)
          可用的键：“module”、“main-jar”、“main-class”、“arguments”、“java-options”、“app-version”、“icon”和“win-console”.
          这些选项被添加到或用于覆盖原始命令行选项，以构建额外的替代启动程序.
          主应用程序启动器将从命令行选项构建。使用这个选项可以建造额外的替代启动器，并且这个选项可以多次使用来建造多个额外的启动器. 
  --arguments <main class arguments>
          如果没有给启动程序提供命令行参数，则要传递给主类的命令行参数
          此选项可多次使用.
  --java-options <java options>
          要传递给Java运行时的选项
          此选项可多次使用.
  --main-class <class name>
          要执行的应用程序主类的限定名称。
          这个选项只能在指定--main-jar时使用.
  --main-jar <main jar file>
          应用程序的主JAR;
          包含主类(指定为相对于输入路径的路径);
          --module或--main-jar选项可以指定，但不能同时指定.
  --module -m <module name>[/<main class>]
          应用程序的主模块(可选的主类);
          此模块必须位于模块路径上;
          指定此选项时，将在Java运行时映像中链接主模块。--module或--main-jar选项可以指定，但不能同时指定.

用来创建应用程序启动程序的与平台相关的选项：
  --win-console
          为应用程序创建控制台启动程序，应当为
          需要控制台交互的应用程序指定

用于创建应用程序包的选项:
  --about-url <url>
          应用程序主页的URL
  --app-image <directory path>
          用于构建可安装包的预定义应用程序映像的位置(绝对路径或相对于当前目录)。
  --file-associations <file path>
          包含键值对的列表的属性文件的路径(绝对路径或相对于当前目录)
          “extension”、“mime-type”、“icon”和“description”可用于描述该关联。
          此选项可多次使用。.
  --install-dir <directory path>
          默认安装位置下面的相对子路径
  --license-file <file path>
          license文件的路径(绝对路径或相对于当前目录)
  --resource-dir <directory path>
          覆盖jpackage资源的路径.
          通过向这个目录添加替换资源，可以覆盖jpackage的图标、模板文件和其他资源(绝对路径或相对于当前目录).
  --runtime-image <directory path>
          要安装的预定义运行时映像的路径(绝对路径或相对于当前目录)
          本选项在创建运行时包时是必需的！

Platform dependent options for creating the application package:
  --win-dir-chooser
          添加一个对话框，使用户能够选择产品的安装位置.
  --win-help-url <url>
          用户获取进一步信息或技术支持的网址.
  --win-menu
          请求为此应用程序添加开始菜单快捷方式
  --win-menu-group <menu group name>
          应用程序所在的开始菜单组
  --win-per-user-install
          请求在每个用户的基础上执行安装
  --win-shortcut
          请求为此应用程序添加桌面快捷方式
  --win-shortcut-prompt
          添加一个对话框，允许用户选择安装程序是否创建快捷方式.
  --win-update-url <url>
          可用的应用程序更新信息的URL
  --win-upgrade-uuid <id string>
          与此包的升级相关联的UUID
```







## 7、G1 的 NUMA 内存分配优化

G1 收集器现在可以感知 `NUMA` 内存分配方式，以提高 G1 的性能，可以使用 `+XX:+UseNUMA` 启用这项功能。

NUMA 就是非统一内存访问架构（英语：non-uniform memory access，简称：NUMA），是一种为多处理器的电脑设计的内存架构，内存访问时间取决于内存相对于处理器的位置。在NUMA下，处理器访问它自己的本地内存的速度比非本地内存（内存位于另一个处理器，或者是处理器之间共享的内存）快一些。如下图所示，Node0中的CPU如果访问Node0中的内存，那就是访问本地内存，如果它访问了Node1中的内存，那就是远程访问，性能较差。

非统一内存访问架构的特点是：

  被共享的内存物理上是分布式的，所有这些内存的集合就是全局地址空间。所以处理器访问这些内存的时间是不一样的，显然访问本地内存的速度要比访问全局共享内存或远程访问外地内存要快些。另外，NUMA中内存可能是分层的：本地内存，群内共享内存，全局共享内存。

此次更新的目标：JEP345 希望通过实现 NUMA-aware 的内存分配，改进 G1 在大型机上的性能。

  现代的 multi-socket 服务器越来越多都有 NUMA，意思是，内存到每个 socket 的距离是不相等的，内存到不同的 socket 之间的访问是有性能差异的，这个距离越长，延迟就会越大，性能就会越差。（https://openjdk.java.net/jeps/345）

  G1 的堆组织为固定大小区域的集合。一个区域通常是一组物理页面，尽管使用大页面（通过 -XX:+UseLargePages）时，多个区域可能组成一个物理页面。

  如果指定了+XX:+UseNUMA 选项，则在初始化JVM时，区域将平均分布在可用 NUMA 节点的总数上。

  在开始时固定每个区域的 NUMA 节点有些不灵活，但是可以通过以下增强来缓解。为了为 mutator 线程分配新的对象，G1 可能需要分配一个新的区域。它将通过从 NUMA 节点中优先选择一个与当前线程绑定的空闲区域来执行此操作，以便将对象保留在新生代的同一 NUMA 节点上。如果在为变量分配区域的过程中，同一 NUMA 节点上没有空闲区域，则 G1 将触发垃圾回收。要评估的另一种想法是，从距离最近的 NUMA 节点开始，按距离顺序在其他NUMA节点中搜索自由区域。

  该特性不会尝试将对象保留在老年代的同一NUMA节点上。

  JEP 345 专门用于实现 G1 垃圾收集器的NUMA支持，仅用于内存管理（内存分配），并且仅在 Linux 下。对于NUMA体系结构的这种支持是否也适用于其他垃圾回收器或其他部分（例如任务队列窃取），尚不清楚。



## 8、移除 CMS 垃圾收集器

自从 G1 出现后，CMS在 Java9 中就被标记为 Deprecate 了，CMS 的弊端如下：

- 会产生内存碎片,导致并发清除后,用户线程可用空间不足(标记清除算法产生,需要整理算法解决)
- 既然强调了并发(Concurrent) CMS收集器对于CPU资源非常敏感,导致吞吐量降低
- CMS收集器无法处理浮动垃圾(用户线程和垃圾回收线程并发执行,回收时用户线程产生新的垃圾)

当 CMS 停止工作时，会把 Serial Old GC 作为备选方案，而它是 JVM 中性能最差的垃圾收集方式，停顿几秒甚至十秒都有可能。移除了CMS 垃圾收集器，如果继续在 JDK14 中使用 -XX:+UseConcMarkSweepGC 不会报错，仅仅给出一个warning警告。

```bash
warning: Ignoring option UseConcMarkSweepGC; support was removed in 14.0
```

> 其他垃圾收集器

G1 回收器 hotSpot 已经默认使用有几年，我们还看到两个新的 GC，Java 11 中的 ZGC 和 OpenJDK12 中的 Shenandoah，后两者主要特点是：低停顿时间短。Shenandoah 非Oracle 官方发布的，是 OpenJDK 于Java 12发布的：

| 收集器名称       | 运行时间 | 总停顿时间 | 最大停顿时间 | 平均停顿时间 |
| ---------------- | -------- | ---------- | ------------ | ------------ |
| Shenandoah       | 387.602s | 320ms      | 89.79ms      | 53.01ms      |
| G1               | 312.052s | 11.7s      | 1.24s        | 450.12ms     |
| CMS              | 286.264s | 12.78s     | 4.39s        | 852.26ms     |
| ParallelScavenge | 260.092s | 6.59s      | 3.04s        | 823.75ms     |



## 10、外部存储器 API【孵化阶段】

通过一个API，以允许 Java 程序安全有效的访问 Java 堆之外的外部存储（堆以外的外部存储空间）

目的：JEP 370 旨在实现一种提供“通用性”，“安全性”和“确定性”的“外部存储器API”JEP还指出，此外部内存API旨在替代当前使用的方法（ java.nio.ByteBuffer和sun.misc.Unsafe ）。

许多 Java 的库都能访问外部存储，例如：ignite、mapDB、memcached 以及 netty 的 ByteBuffer API。这样可以：

- 避免垃圾回收相关成本和不可预测性
- 跨多个进程共享内存
- 通过将文件映射到内存中来序列化和反序列化内容

但是 Java API 本身没有提供一个令人满意的访问外部内存的解决方案，当 Java 程序需要访问堆内存之外的外部存储时，通常有两种方式

- java.nio.ByteBuffer：ByteBuffer 允许使用 allcateDirect() 方法在堆内存之外分配内存空间
- sum.misc.Unsafe：Unsafe 中的方法可以直接对内存地址进行操作

  ByteBuffer有自己的限制。首先是 ByteBuffer 的大小不能超过 2G，其次是内存的释放依靠垃圾回收器，Unsafe的API 在使用是不安全的，风险很高，可能会造成 JVM 崩溃。另外 Unsafe 本身是不被支持的API，并不推荐。

  JEP 370 的“描述”部分引入了安全高效的 API 来访问外部外部内存地址，目前该API还属于孵化阶段，相关API在jdk.incubator.foreign模块的jdk.incubator.foreign包中, 三个API分别是： MemorySegment ， MemoryAddress 和 MemoryLayout 。 MemorySegment 用于对具有给定空间和时间范围的连续内存区域进行建模。 可以将 MemoryAddress 视为段内的偏移量。 最后， MemoryLayout 是内存段内容的程序化描述。



## 11、非易失性映射字节缓冲区

Java 14 增加了一种文件映射模式，用于访问非易失性内存，非易失性内存能够持久保持数据，因此可以利用该特性来改进性能。

JEP352 可以使用 FileChannelAPI 创建引用非易失性内存，（non-volatile memory）的 MappedByteBuffer 实例，该 JEP 建议升级 MappedByteBuffer 以支持对非易失性存储器的访问，唯一需要的 API 更改是 FileChannel 客户端，以请求映射位于 NVM 的支持的文件系统，而不是常规的文件存储系统上的文件，对 MappedByteBuffer API 最新的更改意味着他支持允许直接内存更新所需要的所有行为，并提供更高级别的JAVA客户端库所需要的持久性保证，以实现持久性的数据类型。

> 目标

  NVM为引用程序程序员提供了在程序运行过程中创建和更新程序转台的机会，而减少了输出到持久性介质或者从持久性介质输入是的成本。对于事务程序特别重要，在事务程序中，需要定期保持不确定状态以启用崩溃恢复。

  现有的C库（例如Intel的libpmen），为 C 程序员提供了对集成 NVM 的高效访问，它们还一次基础来支持对各种持久性数据类型的简单管理。当前，由于频繁需要进行系统调用或者JNI来调用原始操作，从而确保内存更改是持久的，因此即使禁用 Java 的基础类库也很昂贵。同样的问题限制了高级库的使用。并且，由于 C 中提供的持久数据类型分配在无法从 Java 直接访问的内存中这一事实而加剧了这一问题。

  该特性试图通过允许映射到 ByteBuffer 的 NVM 的有效写回来解决第一个问题。由于 Java 可以直接访问 ByteBuffer 映射内存，因此可以通过实现与 C 语言中提供的客户端库等效的客户端库来解决第二个问题，以管理不同持久数据类型存储。

> 初步变更

该 JE P使用了JavaSE API 的两个增强功能：

- 支持 Implementation-defined 的映射模式
- MappedByteBuffer::force 方法指定范围

> 特定于 JDK 的 API 更改

通过新模块中的公共 API 公开新的 MApMode 枚举值，一个公共扩展枚举 ExtendedMapMode 将添加到 jdk.nio.mapmode 程序包。

```java
package jdk.nio.mapmode;
public class ExtendedMapMode{
    private ExtendedMapMode(){

    }
    public static final MapMode READ_ONLY_SYNC=  ... ...;
}
```

  在调用 FileChannel::map 方法创建映射到 NVM 设备文件上的只读或者写 MappedByteBuffer 时，可以使用上述的枚举值，如果这些标志在不支持 NVM 设备文件平台上传递，程序会抛出 UnsupportedOperationException 异常，在受支持的平台上，及当目标 FileChannel 实例是通过 NVM 设备打开的派生文件是，才能传递这些参数，在任何情况下，都会抛出IOException。



## 12、JFR 事件流

### 1、简介

Java Flight Recorder（JFR）是 JVM 的诊断和性能分析工具。Java 14 之前只能做离线的分析，现在可以做实时的持续监视。

它可以收集有关JVM以及在其上运行的Java应用程序的数据。JFR 是集成到 JVM 中的，所以 JFR 对 JVM 的性能影响非常小，我们可以放心的使用它。

一般来说，在使用默认配置的时候，性能影响要小于1%。

JFR的历史很久远了。早在Oracle2008年收购BEA的时候就有了。JFR一般和JMC（Java Mission Control）协同工作。

JFR是一个基于事件的低开销的分析引擎，具有高性能的后端，可以以二进制格式编写事件，而JMC是一个GUI工具，用于检查JFR创建的数据文件。

这些工具最早是在BEA的JRockit JVM中出现的，最后被移植到了Oracle JDK。最开始JFR是商用版本，但是在JDK11的时候，JFR和JMC完全开源了，这意味着我们在非商用的情况下也可以使用了。

而在今天的JDK 14中，引入了一个新的JFR特性叫做JFR Event Streaming，我们将在本文中简要介绍。

先介绍一下 JFR 和 JMC。



### 2、JFR

上面我们简单的介绍了一下JFR。JFR 是 JVM 的调优工具，通过不停的收集 JVM 和 Java 应用程序中的各种事件，从而为后续的JMC分析提供数据。

Event是由三部分组成的：时间戳，事件名和数据。同时JFR也会处理三种类型的Event：持续一段时间的Event，立刻触发的Event和抽样的Event。

为了保证性能的最新影响，在使用JFR的时候，请选择你需要的事件类型。

JFR 从J VM中 搜集到 Event 之后，会将其写入一个小的 thread-local 缓存中，然后刷新到一个全局的内存缓存中，最后将缓存中的数据写到磁盘中去。

或者你可以配置JFR不写到磁盘中去，但是这样缓存中只会保存部分events的信息。这也是为什么会有JDK14 JEP 349的原因。

开启 JFR 有很多种方式，这里我们关注下面两种：

1、添加命令行参数开启

```bash
-XX:StartFlightRecording:<options>
```

JFR 可以获取超过一百种不同类型的元数据。如果要我们一个个来指定这些元数据，将会是一个非常大的功能。所以 JDK 已经为我们提供了两个默认的 profile：default.jfc and profile.jfc。

其中 default.jfc 是默认的记录等级，对 JVM 性能影响不大，适合普通的，大部分应用程序。而 profile.jfc 包含了更多的细节，对性能影响会更多一些。

如果你不想使用默认的两个 jfc 文件，也可以按照你自己的需要来创建。

```bash
# 该命令会创建一个最大age是5h的profile信息文件。
-XX:StartFlightRecording:disk=true,filename=/tmp/customer.jfr,maxage=5h,settings=profile
```



2、使用 jcmd 命令开启

命令行添加参数还是太麻烦了，如果我们想动态添加 JFR，则可以使用 jcmd 命令。

```bash
jcmd <pid> JFR.start name=custProfile settings=default
jcmd <pid> JFR.dump filename=custProfile.jfr
jcmd <pid> JFR.stop
```

- 上面的命令在一个运行中的 JVM 中启动了 JFR，并将统计结果 dump 到了文件中。
- 上面的 custProfile.jfr 是一个二进制文件，为了对其进行分析，我们需要和 JFR 配套的工具 JMC。



### 3、JMC

JDK Mission Control：是一个用于对 Java 应用程序进行管理、监视、概要分析和故障排除的工具套件。

在 JDK14 中，JMC 是独立于 JDK 单独发行的。我们可以下载之后进行安装。

我们先启动一个程序，用于做 JFR 的测试。

```java
@Slf4j
public class ThreadTest {
    public static void main(String[] args) {
        ExecutorService executorService= Executors.newFixedThreadPool(10);
        Runnable runnable= ()->{
            while(true){
                log.info(Thread.currentThread().getName());
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    log.error(e.getMessage(),e);
                }
            }
        };

        for(int i=0; i<10; i++){
            executorService.submit(runnable);
        }
    }
}
```

很简单的一个程序，启动了10个线程，我们启动这个程序。然后再去看看JMC的界面。JMC 非常强大，也有很多功能，具体的细节大家可以自己运行去体会。



## 13、新增、移除与废弃项

### 1、新增项

- JEP 364：MacOS 上的 ZGC（实验性）：Java 11 在 Linux 上引入了 ZGC 垃圾收集器，现在它可以移植到 macOS。
- JEP 365：Windows 上的 ZGC（实验性）：Java 11 在 Linux 上引入了 ZGC 垃圾收集器，现在它可以移植到 Windows 上（版本大于 1803）。



### 2、移除项

- JEP 363： 移除 CMS 垃圾收集器：移除对 CMS（Concurrent Mark Sweep） 垃圾收集器的支持，其实早在 Java 9 就开始移除 CMS 垃圾收集器了，只不过在 Java 14 中被正式删除。
- JEP 367：删除 Pack200 工具和 API。



### 3、废弃项

- 废弃了 Thread 的如下方法：
  - Thread.suspend()
  - Thread.resume()
  - ThreadGroup.suspend()
  - ThreadGroup.resume()
  - ThreadGroup.allowThreadSuspension(boolean)
- JEP 362：废弃对 Solaris 和 SPARC 端口支持：从 Java 14 开始，放弃对 Solaris/SPARC, Solaris/x64, 和 Linux/SPARC 端口的支持，放弃一部分开发这势必会加快 Java 整体的开发节奏（相关阅读：https://openjdk.java.net/jeps/362）
- JEP 366：弃用 ParallelScavenge + SerialOld GC 组合：由于使用场景不多，维护工作太大，废弃之（相关阅读：https://openjdk.java.net/jeps/366）



## 14、参考文献 & 鸣谢

1. Java 新特性文档：https://www.wdbyte.com/java-feature/
2. Java New Feature：https://javaguide.cn/java/new-features/
3. Java 的新特性：https://segmentfault.com/a/1190000004417288/
4. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
5. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/
