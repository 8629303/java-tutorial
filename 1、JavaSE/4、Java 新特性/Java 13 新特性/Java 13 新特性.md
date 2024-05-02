# Java 13 新特性

Java 13 在 2019 年 9 月 17 日发布，它不是一个长久支持（LTS）版本。

这次更新共带来 5 个新特性：

1. 350: Dynamic CDS Archives：动态CDS档案
2. 351: ZGC: Uncommit Unused Memory：ZGC:取消使用未使用的内存
3. 353: Reimplement the Legacy Socket API：重新实现旧版套接字API
4. 354: Switch Expressions (Preview)：switch表达式（预览）
5. 355: Text Blocks (Preview)：文本块

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 13 新特性：https://openjdk.org/projects/jdk/13/
- OracleJDK Java 13 新特性：https://www.oracle.com/java/technologies/javase/13-relnote-issues.html#NewFeature
- Oracle JDK 13 文档：https://docs.oracle.com/en/java/javase/13/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## 1、Switch 表达式 【二次预览】

在 Java 12 中引入了 Switch 表达式作为预览特性，虽然增强了 Swtich 语法，但并不能在 `->` 之后写复杂的逻辑。Java 13 提出了第二个 Switch 表达式预览。JEP 354 修改了这个特性，它引入了 yield 语句，用于返回值。这意味着，Switch 表达式（返回值）应该使用 yield，Switch语句（不返回值）应该使用 break。Java 13 的 Switch 就可以像 Lambda一样，可以写逻辑，然后再返回。

【Java 11 以及之前的版本】：Switch 表达式的代码不够美观优雅，甚至有些啰嗦。

```java
// 通过传入月份，输出月份所属的季节
public static String switchJava12Before(String month) {
    String reuslt = null;
    switch (month) {
        case "march":
        case "april":
        case "may":
            reuslt = "春天";
            break;
        case "june":
        case "july":
        case "august":
            reuslt = "夏天";
            break;
        case "september":
        case "october":
        case "november":
            reuslt = "秋天";
            break;
        case "december":
        case "january":
        case "february":
            reuslt = "冬天";
            break;
    }
    return reuslt;
}
```

【Java 12 预览版本】：可以使用 `cast L ->` 表达式进行操作，且可以具有返回值，但并不能在 `->` 之后写复杂的逻辑。

```java
// 通过传入月份，输出月份所属的季节
public static String switchJava12(String month) {
    return switch (month) {
        case "march", "april", "may"            -> "春天";
        case "june", "july", "august"           -> "夏天";
        case "september", "october", "november" -> "秋天";
        case "december", "january", "february"  -> "冬天";
        default -> "month erro";
    };
}
```

【Java 13 预览版本】：在 JDK12 基础上，增加了 yield 关键词用于返回值（可写逻辑再返回），相比 break ，语义更加明确了。

```java
// 通过传入月份，输出月份所属的季节
public static String switchJava13(String month) {
    return switch (month) {
        case "march", "april", "may":
            yield "春天";
        case "june", "july", "august":
            yield "夏天";
        case "september", "october", "november":
            yield "秋天";
        case "december", "january", "february":
            yield "冬天";
        default:
            yield "month error";
    };
}
```

【**关键字 yield 与 return 的区别**】：

1. return 会直接跳出当前循环或者方法，而 yield 只会跳出当前 Switch 块
2. 注意：在使用 yield 时，需要有 default 条件。



## 2、多行文本块【预览】

### 1、概念

在 JDK 12 中引入了Raw String Literals特性，但在发布之前就放弃了。这个JEP与引入多行字符串文字（text block）在意义上是类似的。这条新特性跟 Kotlin 里的文本块或者 Python 多行文本是类似的。



### 2、问题

在 Java 中，通常需要使用 String 类型表达 HTML、XML、SQL 或 JSON 等格式的字符串，在进行字符串赋值时需要进行转义和连接操作，然后才能编译该代码，这种表达方式难以阅读并且难以维护。文本块就是指多行字符串，例如一段格式化后的 xml、jso n等。而有了文本块以后，用户不需要转义，Java 能自动搞定。因此，文本块将提高 Java 程序的可读性和可写性。



### 3、目标

Java 13 支持两个 `"""` 符号中间的任何内容都会被解释为字符串的一部分，包括换行符。简化跨越多行的字符串，避免对换行等特殊字符进行转义，简化编写 Java 程序。增强 Java 程序中字符串的可读性。

【Java 13 之前 JSON 写法】

```java
String json ="{\n" +
              "   \"name\":\"mkyong\",\n" +
              "   \"age\":38\n" +
              "}\n";
```

【Java 13 JSON 写法】

```java
 String json = """
                {
                    "name":"mkyong",
                    "age":38
                }
                """;
```

【Java 13 之前 SQL 写法】

```java
String sql = "SELECT `EMP_ID`, `LAST_NAME` FROM `EMPLOYEE_TB`\n" +
             "WHERE `CITY` = 'INDIANAPOLIS'\n" +
             "ORDER BY `EMP_ID`, `LAST_NAME`;\n";
```

【Java 13 SQL 写法】

```java
String sql = """
             SELECT `EMP_ID`, `LAST_NAME` FROM `EMPLOYEE_TB`
             WHERE `CITY` = 'INDIANAPOLIS'
             ORDER BY `EMP_ID`, `LAST_NAME`;
             """;
```



### 4、语法细节1：基本使用

- 文本块是 Java 语言中的一种新文字。它可以用来表示任何字符串，并且提供更大的表现力和更少的复杂性。
- 文本块由零个或多个字符组成，由开始和结束分隔符括起来。
  - 开始分隔符是由三个双引号字符（`"""`），后面可以跟零个或多个空格，最终以行终止符结束。文本块内容以开始分隔符的行终止符后的第一个字符开始。
  - 结束分隔符也是由三个双引号字符（`"""`）表示，文本块内容以结束分隔符的第一个双引号之前的最后一个字符结束。
- 文本块中的内容可以直接使用"，"但不是必需的。
- 文本块中的内容可以直接包括行终止符。允许在文本块中使用 \n，但不是必需的。例如，文本块：

```java
"""
line1
line2
line3
"""
```

相当于：

```java
"line1\nline2\nline3\n"
```

或者相当于一个字符串用+拼接：

```java
"line1\n" + "line2\n" + "line3\n"
```

文本块可以表示空字符串，但不建议这样做，因为它需要两行源代码：

```java
String empty = """
""";
```

以下是**错误格式**的文本块：

```java
String a = """""";  // 开始分隔符后没有行终止符
String b = """ """; // 开始分隔符后没有行终止符
String c = """
"; // 没有结束分隔符
String d = """
abc \ def
"""; // 含有未转义的反斜线（请参阅下面的转义处理）
```

在运行时，文本块将被实例化为 String 的实例，就像字符串一样。从文本块派生的String实例与从字符串派生的实例是无法区分的。具有相同内容的两个文本块将引用相同的 String 实例，就像字符串一样。



### 5、语法细节2：编译会删除多余的空格

下面这段代码中，我们用.来表示我们代码中的的空格，而这些位置的空格就是多余的。

```java
String html = 
........"""
........<html>
........     <body>
........         <p>Hello, world</p>
........     </body>
........</html>
........""";

// 正常格式
String html =
        """
        <html>
             <body>
                 <p>Hello, world</p>
             </body>
        </html>
        """;
```

多余的空格还会出现在每一行的结尾，特别是当你从其他地方复制过来时，更容易出现这种情况，比如下面的代码：

```java
String html = 
........"""
........<html>
........     <body>
........         <p>Hello, world</p>...
........     </body>.
........</html>>...
........""";
    
// 正常格式
String html =
        """
        <html>
             <body>
                 <p>Hello, world</p>   
             </body> 
        </html>   
        """;
```

PS 注意：每行文字后面的空格，编译器会自动帮助我们去掉，但是开头部分的空格和结束的`"""`；与前面的空格数有关。`"""`；前面有几个空格，就会自动帮助我们去掉每一行前面的几个空格

这些多余的空格对于程序员来说是看不到的，但是他又是实际存在的，所以如果编译器不做处理，可能会导致程序员看到的两个文本块内容是一样的，但是这两个文本块却因为存在这种多余的空格而导致差异，比如哈希值不相等。



### 6、语法细节3：转义字符

允许开发人员使用 \n，\f 和 \r 来进行字符串的垂直格式化，使用 \b和 \t 进行水平格式化。比如下面的代码是合法的：

```java
String html = """
<html>\n
<body>\n
<p>Hello, world</p>\n
</body>\n
</html>\n
""";
```

请注意，在文本块内自由使用"是合法的。例如：

```java
String story = """
"When I use a word," Humpty Dumpty said,
in rather a scornful tone, "it means just what I
choose it to mean - neither more nor less."
"The question is," said Alice, "whether you
can make words mean so many different things."
"The question is," said Humpty Dumpty,
"which is to be master - that's all."
""";
```

但是，三个`"`字符的序列需要进行转义至少一个"以避免模仿结束分隔符：

```java
String code =
"""
String text = \"""
A text block inside a text block
\""";
""";
```



### 7、语法细节4：文本块连接

可以在任何可以使用字符串的地方使用文本块。例如，文本块和字符串可以相互连接：

```java
String code = "public void print(Object o) {" +
"""
System.out.println(Objects.toString(o));
}
""";
```

但是，涉及文本块的连接可能变得相当笨重。以下面文本块为基础：

```java
String code = """
public void print(Object o) {
System.out.println(Objects.toString(o));
}
""";
```

假设我们想把上面的 Object 改为来自某一变量，我们可能会这么写：

```java
String code = """
public void print(""" + type + """
o) {
System.out.println(Objects.toString(o));
}
""";
```

可以发现这种写法可读性是非常差的，更简洁的替代方法是使用 String :: replace 或 String :: format，比如：

```java
String code = """
public void print($type o) {
System.out.println(Objects.toString(o));
}
""".replace("$type", type);
```

```java
String code = String.format("""
public void print(%s o) {
System.out.println(Objects.toString(o));
}
""", type);
```

另一个方法是使用 String :: formatted，这是一个新方法，比如：

```java
String source = """
public void print(%s object) {
System.out.println(Objects.toString(object));
}
""".formatted(type);
```



## 3、String 增强【预览】

JEP 355 在 String 类中添加了三个新的实例方法：formatted，stripIndent 和 translateEscapes。 格式化是一种辅助方法，它使使用多行文字更加容易，而 stripIndent 和 translateEscapes 表示其解释的最后两个编译步骤。

String 类新增加了 3 个新的方法来操作文本块如下所示：

- `formatted(Object... args)` ：它类似于 `String` 的`format()`方法。添加它是为了支持文本块的格式设置。
- `stripIndent()` ：用于去除文本块中每一行开头和结尾的空格。
- `translateEscapes()` ：转义序列如`\t`转换为`\t`，使用该方法可以让转义符生效，否则按字符串输出。

由于文本块是一项预览功能，可以在未来版本中删除，因此这些新方法被标记为弃用。

```java
@Deprecated(forRemoval=true, since="13")
public String stripIndent() {
}
@Deprecated(forRemoval=true, since="13")
public String formatted(Object... args) {

}
@Deprecated(forRemoval=true, since="13")
public String translateEscapes() {
}
```

formatted 方法类似于静态方法 format，结果也是一个使用占位符格式化的字符串，但是与文本块结合使用时，其代码会更整齐。

```java
String xml =
        """
        <customer>
             <age>%s</age>
             <name>%s</name>
        </customer>
        """.formatted("18", "Franz Sam");
System.out.println(xml);
```

```xml
<customer>
     <age>18</age>
     <name>Franz Sam</name>
</customer>
```

stripIndent 方法删除所有行共有的多行字符串前面的空格，即，将整个文本向左移动而不更改格式。 translateEscapes 方法解释字符串中包含的所有转义序列。 

```java
// The left margin is marked by | for illustration purposes.
String example = " a\n  b\\n   c";

System.out.println(example);
//| a
//|  b\n   c

System.out.println(example.stripIndent());
//|a
//| b\n   c

System.out.println(example.translateEscapes());
//| a
//|  b
//|   c

System.out.println(example.stripIndent().translateEscapes());
//|a
//| b
//|  c
```



## 4、FileSystems 新增方法

java.nio.file.FileSystems 针对任何将文件内容视作文件系统的文件系统提供者，添加了三个扩展方法：

- `newFileSystem(Path)`
- `newFileSystem(Path, Map<String, ?>)`
- `newFileSystem(Path, Map<String, ?>, ClassLoader)`

需要说明的是 `newFileSystem(Path, Map<String, ?>)`方法是在现有 `newFileSystem(Path, ClassLoader)` 方法基础上，对参数class loader 作 null处理的调用。因此需要避免如下使用：
```java
FileSystem fs = FileSystems.newFileSystem(path, null);
```



## 5、Socket API 重构

重构 Socket 接口实现类。现有的 java.net.Socket 和 java.net.ServerSocket 以及它们的实现类，都可以回溯到 JDK 1.0 时代了。

- 它们的实现是混合了 Java 和 C 的代码的，维护和调试都很痛苦。
- 实现类还使用了线程栈作为 I/O 的缓冲，导致在某些情况下还需要增加线程栈的大小。
- 支持异步关闭，此操作是通过使用一个本地的数据结构来实现的，这种方式这些年也带来了潜在的不稳定性和跨平台移植问题。该实现还存在几个并发问题，需要彻底解决。

因此，在 Java 13 中引入了新的实现方式，使用了新的实现 NioSocketImpl 来代替老旧的 PlainSocketImpl 实现，它使用 java.util.concurrent 包下的锁而不是同步方法。虽然功能相同，但是老的方式在当前以及未来几个版本内不会删除，用户随时可以通过 `-Djdk.net.usePlainSocketImpl=true` 参数切换回老的实现方式，以兼容意外情况。

Java 13 中是默认使用新的 Socket 实现。

```java
public final class NioSocketImpl extends SocketImpl implements PlatformSocketImpl {
}
```

```java
public abstract class SocketImpl implements SocketOptions {
    private static final boolean USE_PLAINSOCKETIMPL = usePlainSocketImpl();

    private static boolean usePlainSocketImpl() {
        PrivilegedAction<String> pa = () -> NetProperties.get("jdk.net.usePlainSocketImpl");
        String s = AccessController.doPrivileged(pa);
        return (s != null) && !s.equalsIgnoreCase("false");
    }
}
```

SocketImpl 的 USE_PLAINSOCKETIMPL 取决于 usePlainSocketImpl 方法，而它会从 NetProperties 读取 jdk.net.usePlainSocketImpl 配置，如果不为 null 且不为 false，则 usePlainSocketImpl 方法返回 true；createPlatformSocketImpl 会根据 USE_PLAINSOCKETIMPL来创建 PlainSocketImpl 或者 NioSocketImpl。

【操作示例】编写一个测试类可以发现这个变化。

```java
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
public class Test {
    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(8000)){
            boolean running = true;
            while(running){
                Socket clientSocket = serverSocket.accept();
                //do something with clientSocket
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

使用 Java 13 运行，通过参数 `-XX:+TraceClassLoading` 追踪加载的类，日志中可以看到 NioSocketImpl。

```java
➜  develop ./jdk-13.0.2.jdk/Contents/Home/bin/java -XX:+TraceClassLoading Test.java | grep SocketImpl
[0.699s][info   ][class,load] java.net.SocketImpl source: jrt:/java.base
[0.699s][info   ][class,load] java.net.SocketImpl$$Lambda$173/0x0000000800c37440 source: java.net.SocketImpl
[0.702s][info   ][class,load] sun.net.PlatformSocketImpl source: jrt:/java.base
[0.702s][info   ][class,load] sun.nio.ch.NioSocketImpl source: jrt:/java.base
[0.713s][info   ][class,load] sun.nio.ch.NioSocketImpl$FileDescriptorCloser source: jrt:/java.base
```

但在使用 Java 12 及之前并不是 NioSocketImpl。

```java
➜  develop ./jdk-12.0.2.jdk/Contents/Home/bin/java -XX:+TraceClassLoading Test.java | grep SocketImpl
[0.665s][info   ][class,load] java.net.SocketImpl source: jrt:/java.base
[0.665s][info   ][class,load] java.net.AbstractPlainSocketImpl source: jrt:/java.base
[0.665s][info   ][class,load] java.net.PlainSocketImpl source: jrt:/java.base
[0.665s][info   ][class,load] java.net.SocksSocketImpl source: jrt:/java.base
[0.666s][info   ][class,load] java.net.AbstractPlainSocketImpl$1 source: jrt:/java.base
```





## 6、增强 ZGC 释放未使用内存

> **G1 与 Shenandoah**

JVM 的 GC 释放的内存会还给操作系统吗？

GC 后的内存如何处置，其实是取决于不同的垃圾回收器。因为把内存还给OS，意味着要调整JVM的堆大小，这个过程是比较耗费资源的。

- Java 12 的 346: Promptly Return Unused Committed Memory from G1新增了两个参数分别是：G1PeriodicGCInterval 及G1PeriodicGCSystemLoadThreshold 用于 GC之 后重新调整 Java heap size，然后将多余的内存归还给操作系统。
- Java 12 的189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)拥有参数：-XX:ShenandoahUncommitDelay=来指定 ZPage的page cache 的失效时间，然后归还内存。

HotSpot 的 G1 和Shenandoah这两个 GC 已经提供了这种能力，并且对某些用户来说，非常有用。因此，Java13 则给 ZGC 新增归还unused heap memory 给操作系统的特性。

> **ZGC 的使用背景**

  在 JDK 11 中，Java 引入了 ZGC，这是一款可伸缩的低延迟垃圾收集器，但是当时只是实验性的。号称不管你开了多大的堆内存，它都能保证在 10 毫秒内释放 JVM ，不让它停顿在那。但是，当时的设计是它不能把内存归还给操作系统。对于比较关心内存占用的应用来说，肯定希望进程不要占用过多的内存空间了，所以这次增加了这个特性。

  在Java 13中，JEP 351再次对ZGC做了增强，将没有使用的堆内存归还给操作系统。ZGC当前不能把内存归还给操作系统，即使是那些很久都没有使用的内存，也只进不出。这种行为并不是对任何应用和环境都是友好的，尤其是那些内存占用敏感的服务，例如：

- 按需付费使用的容器环境；
- 应用程序可能长时间闲置，并且和很多其他应用共享和竞争资源的环境；
- 应用程序在执行期间有非常不同的堆空间需求，例如，可能在启动的时候所需的堆比稳定运行的时候需要更多的堆内存。

> **使用细节**

  ZGC的堆由若干个Region组成，每个Region被称之为ZPage。每个Zpage与数量可变的已提交内存相关联。当ZGC压缩堆的时候，ZPage就会释放，然后进入page cache，即ZPageCache。这些在page cache中的ZPage集合就表示没有使用部分的堆，这部分内存应该被归还给操作系统。回收内存可以简单的通过从page cache中逐出若干个选好的。

  ZPage来实现，由于page cache是以LRU（Least recently used，最近最少使用）顺序保存ZPage的，并且按照尺寸（小，中，大）进行隔离，因此逐出ZPage机制和回收内存相对简单了很多，主要挑战是设计关于何时从page cache中逐出ZPage的策略。

  一个简单的策略就是设定一个超时或者延迟值，表示ZPage被驱逐前，能在page cache中驻留多长时间。这个超时时间会有一个合理的默认值，也可以通过JVM参数覆盖它。Shenandoah GC用了一个类型的策略，默认超时时间是5分钟，可以通过参数-XX:ShenandoahUncommitDelay = milliseconds覆盖默认值。

  像上面这样的策略可能会运作得相当好。但是，用户还可以设想更复杂的策略：不需要添加任何新的命令行选项。例如，基于GC频率或某些其他数据找到合适超时值的启发式算法。JDK13将使用哪种具体策略目前尚未确定。可能最初只提供一个简单的超时策略，使用-XX:ZUncommitDelay = seconds选项，以后的版本会添加更复杂、更智能的策略（如果可以的话）。

  uncommit能力默认是开启的，但是无论指定何种策略，ZGC都不能把堆内存降到低于Xms。这就意味着，如果Xmx和Xms相等的话，这个能力就失效了。-XX:-ZUncommit这个参数也能让这个内存管理能力失效。




## 7、动态  CDS  存档

Java 13 中对 Java 10 中引入的应用程序类数据共享（AppCDS）进行了进一步的简化、改进和扩展，即：**允许在 Java 应用程序执行结束时动态进行类归档**，具体能够被归档的类包括所有已被加载，但不属于默认基层 CDS 的应用程序类和引用类库中的类。

这提高了应用程序类数据共享（[JEP 310: Application Class-Data Sharing](https://openjdk.java.net/jeps/310)）的可用性。无需用户进行试运行来为每个应用程序创建类列表。

```bash
$ java -XX:ArchiveClassesAtExit=my_app_cds.jsa -cp my_app.jar
$ java -XX:SharedArchiveFile=my_app_cds.jsa -cp my_app.jar
```

> 作用

  在 Java 应用程序在程序执行结束时动态归档类。归档的类将包括默认基层 CDS 归档中不存在的所有已加载应用程序类和类库。

  CDS，是 Java 12 的特性了，可以让不同 Java 进程之间共享一份类元数据，减少内存占用，它还能加快应用的启动速度。而 JDK13 的这个特性支持在 Java application 执行之后进行动态 archive。存档类将包括默认的基础层CDS存档中不存在的所有已加载的应用程序和库类。也就是说，在 Java 13 中再使用 AppCDS 的时候，就不再需要这么复杂了。

  该提案处于目标阶段，旨在提高 AppCDS 的可用性，并消除用户进行运行时创建每个应用程序的类列表的需要。

```bash
# JVM退出时动态创建共享归档文件：导出jsa
$ java -XX:ArchiveClassesAtExit=hello.jsa -cp hello.jar Hello
# 用动态创建的共享归档文件运行应用:使用jsa
$ java -XX:SharedArchiveFile=hello.jsa -cp hello.jar Hello
```

> 目的

Java 13 这次对CDS增强的目的

- 改善APPCDS的可用性，减少用户每次都要创建一个类列表的需要
- 通过开启 -Xshare:dump 选项来开启静态归档，使用类列表仍然行得通，包含内置的类加载信息和用户定义的类加载信息

> 意义

在 Java 13 中做的增强,可以只开启命令行选项完成上述过程，在程序运行的时候,动态评估哪些类需要归档，同时支持内置的类加载器和用户定义的类加载器。

在第一次程序执行完成后，会自动的将类进行归档，后续启动项目的时候也无需指定要使用哪些归档，整个过程看起来更加透明。



## 8、增加废弃和移除

### 1、增加项

- 添加 FileSystems.newFileSystem(Path, Map) 等三个方法

- 新的 java.nio.ByteBuffer Bulk get/put Methods Transfer Bytes Without Regard to Buffer Position

  java.nio.ByteBufferjava.nio 现在，其他缓冲区类型定义绝对批量get和put传输连续字节序列的方法，而不考虑或影响缓冲区位置。

- 支持 Unicode 12.1（java.util 中 I18N 支持 Unicode 12.1）

- 添加 -XX:SoftMaxHeapSize Flag，目前仅仅对ZGC起作用

- ZGC 的最大 heap 大小增大到 16TB



### 2、移除项

- 移除 awt.toolkit System Property
- 移除 Runtime Trace Methods
- 移除 -XX:+AggressiveOpts
- 移除 Two Comodo Root CA Certificates、Two DocuSign Root CA Certificates
- 移除内部的 com.sun.net.ssl 包



### 3、废弃项

- 废弃 -Xverify:none 及 -noverify
- 废弃 rmic Tool 并准备移除
- 废弃 javax.security.cert 并准备移除



## 9、参考文献 & 鸣谢

1. Java 新特性文档：https://www.wdbyte.com/java-feature/
2. Java New Feature：https://javaguide.cn/java/new-features/
3. Java 的新特性：https://segmentfault.com/a/1190000004417288/
4. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
5. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/