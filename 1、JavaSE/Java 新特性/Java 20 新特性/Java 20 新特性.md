# Java 20 新特性

Java 20 于 2023 年 3 月 21 日正式发布，Java 20 不是一个长期支持版本。根据开发计划，下一个 LTS 版本就是将于 2023 年 9 月发布的 JDK 21。

JDK 20 只有 7 个新特性：

- JEP 429: Scoped Values：作用域值【第一次孵化】
- JEP 432: Record Patterns：Record 模式匹配【第二次预览】
- JEP 433: Switch 模式匹配【第四次预览】
- JEP 434: Foreign Function & Memory API：外部函数和内存 API【第二次预览】
- JEP 436: Virtual Threads：虚拟线程【第二次预览】
- JEP 437: Structured Concurrency：结构化并发【第二次孵化】
- JEP 432: Vector API：向量 API【第五次孵化】

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 20 新特性：https://openjdk.org/projects/jdk/20/
- OracleJDK Java 20 新特性：https://www.oracle.com/java/technologies/javase/20-relnote-issues.html#NewFeature
- OracleJDK 20 文档：https://docs.oracle.com/en/java/javase/20/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## 1、JEP 429: 作用域值【第一次孵化】

作用域值（Scoped Values）它可以在线程内和线程间共享不可变的数据，优于线程局部变量（比线程本地变量更好用），尤其是在使用大量虚拟线程时。作用域值允许在大型程序中的组件之间安全有效地共享数据，而无需求助于方法参数。

### 1、ThreadLocal【线程本地变量】

自Java 1.2 以来，我们可以使用 ThreadLocal 变量将某个值限制在创建它的线程中。在某些情况下，这可能是实现线程安全的简单方法。

但是，线程本地变量也有一些注意事项。每个线程本地变量都是可变的，这使得很难区分哪个组件更新了共享状态以及以何种顺序。还存在内存泄漏的风险，因为除非在 ThreadLocal 上调用 remove()，否则数据将保留直到垃圾回收（仅在线程终止后才会发生）。最后，父线程的线程本地变量可以被子线程继承，这导致子线程必须为在父线程中先前编写的每个线程本地变量分配存储。

现在虚拟线程被引入后，这些缺点变得更加明显，因为可能会有数百万个虚拟线程处于活动状态，即每个线程都有自己的线程本地变量，这将导致显着的内存占用。



### 2、ScopedValue【作用域值】

与线程本地变量类似，作用域值有多个实例，每个线程一个。不同之处在于，作用域值只写入一次，然后是不可变的，并且仅在线程执行期间的有界时间内可用。

以下是 JEP 429 用伪代码示例说明如何使用作用域值：

```java
class Server {
    final static ScopedValue<User> LOGGED_IN_USER = ScopedValue.newInstance();

    private void serve(Request request) {
        // ...
        User loggedInUser = authenticateUser(request);
        ScopedValue.where(LOGGED_IN_USER, loggedInUser)
            .run(() -> {
                LOGGED_IN_USER.get();
                restAdapter.processRequest(request);
            });
        // ...
        // In a method called directly or indirectly from the lambda expression
        LOGGED_IN_USER.get(); // 绑定已经销毁, 无法获取到值
        // ...
    }
}
```

可以看到，调用 ScopedValue.where(...) 方法会返回一个作用域范围的值和一个要绑定到该值的对象。调用 run(...) 方法会绑定该作用域的值，提供一个特定于当前线程的实例，然后执行作为参数传递的 lambda 表达式。在调用 run(...) 方法的生命周期内，lambda 表达式或任何直接或间接从该表达式调用的方法都可以通过值的 get() 方法读取作用域范围的值。在 run(...) 方法结束后，绑定会被销毁。

**典型应用场景**：作用域范围的值在所有当前使用单向传输不变数据的线程本地变量的地方都很有用。

**对比 Java 19 不同处**：Java 19 中还没有任何与作用域范围的值相关的内容，因此 Java 20 是我们第一次尝试使用它们。



### 3、简单总结作用域值

作用域值 ScopedValue 是一种类似 ThreadLocal 的线程内/父子线程传递变量的更优方案。ThreadLocal 提供了一种无需在方法参数上传递通用变量的方法，InheritableThreadLocal 使得子线程可以拷贝继承父线程的变量。但是 ThreadLocal 提供了set 方法，变量是可变的，另外 remove 方法很容易被忽略，导致在线程池场景下很容易造成内存泄露。ScopedValue 则提供了一种不可变、不拷贝的方案，即不提供 set 方法，子线程不需要拷贝就可以访问父线程的变量。

使用总结：通 过ScopedValue.where可以绑定ScopedValue的值，然后在run方法里可以使用，方法执行完毕自行释放，可以被垃圾收集器回收。

**注意**：JEP 429 处于孵化器阶段，因此需要在命令行中添加 --enable-preview --add-modules jdk.incubator.concurrent 才能试用该功能。



## 2、JEP 432: Record 模式匹配【第二次预览】

记录模式（Record Patterns） 可对 record 的值进行解构，可以嵌套记录模式和类型模式，实现强大的、声明性的和可组合的数据导航和处理形式。记录模式可以与 instanceof 或 switch 模式匹配一同使用。

记录模式在 Java 19 进行了第一次预览， 由 JEP 405 提出。JDK 20 中是第二次预览，由 JEP 432 提出。对比 JDK 19 这次改进包括：

- 添加对泛型记录模式类型参数推断的支持
- 添加对记录模式的支持以出现在增强语句的标题中`for`
- 删除对命名记录模式的支持

1、针对嵌套 record 的推断，可以这样：

```java
record Point(int x, int y) {}
enum Color { RED, GREEN, BLUE }
record ColoredPoint(Point p, Color c) {}
record Rectangle(ColoredPoint upperLeft, ColoredPoint lowerRight) {}

static void printColorOfUpperLeftPoint(Rectangle r) {
    if (r instanceof Rectangle(ColoredPoint(Point p, Color c), ColoredPoint lr)) {
        System.out.println(c);
    }
}
```

2、针对泛型推断

```java
record Box<T>(T t) {}

// 针对泛型推断
static void test1(Box<String> bo) {
    if (bo instanceof Box<String>(var s)) {
        System.out.println("String " + s);
    }
}

// 针对泛型推断当然也支持嵌套
static void test3(Box<Box<String>> bo) {
    if (bo instanceof Box<Box<String>>(Box(var s))) {    
        System.out.println("String " + s);
    }
}
```

3、增强的 for 语句

```java
record Delay(int timeInMs) {}
class DelayPrinter {
    static void printDelays(List<Delay> delays) {
        for (Delay(var timeInMs) : delays) {
            System.out.println("Delay found with timeInMs=" + timeInMs);
        }
    }
}
```



## 3、JEP 433: Switch 模式匹配【第四次预览】

发展历史：

- 在 Java 14 JEP 305: Pattern Matching for instanceof (Preview) 作为 Preview
- 在 Java 15 JEP 375: Pattern Matching for instanceof (Second Preview) 作为第二轮的 Preview
- 在 Java 16 JEP 394: Pattern Matching for instanceof 转正
- 在 Java 17 的 JEP 406: Pattern Matching for switch (Preview) 作为 Preview
- 在 Java 18 的 JEP 420: Pattern Matching for switch (Second Preview) 作为第二轮 Preview
- 在 Java 19 的 JEP 427: Pattern Matching for switch (Third Preview) 作为第三轮 Preview
- 在 Java 20 的 JEP 433: Pattern Matching for switch (Fourth Preview)  作为第四轮 Preview

Java 20 与 Java 19 相比，这个功能做了一些改变：

- 当一个模式匹配突然失败时，现在会抛出一个 MatchException（而不再是IncompatibleClassChangeError）
- 在 Switch 表达式和语句中支持推断 record 模式的类型参数。这意味着现在可以在要匹配的模式中使用 var

1、这里先回顾一下 Switch 模式匹配的用法

```java
static String apply(Effect effect, Guitar guitar) {
    return switch(effect) {
        case Delay de      -> String.format("Delay active of %d ms.", de.timeInMs());
        case Reverb re     -> String.format("Reverb active of type %s and roomSize %d.", re.name(), re.roomSize());
        case Overdrive ov  -> String.format("Overdrive active with gain %d.", ov.gain());
        case Tremolo tr    -> String.format("Tremolo active with depth %d and rate %d.", tr.depth(), tr.rate());
        case Tuner tu when !guitar.isInTune() -> String.format("Tuner active with pitch %d. Muting all signal!", tu.pitchInHz());
        case Tuner tu      -> "Guitar is already in tune.";
        case null, default -> String.format("Unknown or empty effect active: %s.", effect);
    };
}
```

2、针对 record 泛型的类型推断：

```java
record MyPair<S, T>(S fst, T snd) {}
static void recordInference(MyPair<String, Integer> pair){
    switch (pair) {
        case MyPair(var f, var s) -> 
            // Inferred record Pattern MyPair<String,Integer>(var f, var s)
            // ...
    }
}
```



## 4、JEP 434: **外部函数 & 内存 API**【第二次预览】

Java 程序一直可以选择与 Java 运行时外部的代码和数据进行交互，可以使用 Java 本地接口（JNI）调用外部函数（在 JVM 之外但在同一台机器上）。使用 ByteBuffer API 或 sun.misc.Unsafe API 可以访问外部内存（在 JVM 之外，因此是堆外内存）。

但是，这三种机制都具有自己的缺点，因此现在提出了更现代的 API 以更好地支持外部函数和外部内存。

性能关键的库（如 Tensorflow、Lucene 或 Netty）通常依赖使用外部内存，因为它们需要更多地控制它们使用的内存以防止垃圾回收带来的成本和不可预测性。

为了演示新 API，JEP 434 列出了一个代码示例，该示例获取 C 库函数 radixsort 的方法句柄，然后将其用于排序最初作为 Java 数组元素的四个字符串：

```java
// 1. Find foreign function on the C library path
Linker linker          = Linker.nativeLinker();
SymbolLookup stdlib    = linker.defaultLookup();
MethodHandle radixsort = linker.downcallHandle(stdlib.find("radixsort"), ...);
// 2. Allocate on-heap memory to store four strings
String[] javaStrings = { "mouse", "cat", "dog", "car" };
// 3. Use try-with-resources to manage the lifetime of off-heap memory
try (Arena offHeap = Arena.openConfined()) {
    // 4. Allocate a region of off-heap memory to store four pointers
    MemorySegment pointers = offHeap.allocateArray(ValueLayout.ADDRESS, javaStrings.length);
    // 5. Copy the strings from on-heap to off-heap
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = offHeap.allocateUtf8String(javaStrings[i]);
        pointers.setAtIndex(ValueLayout.ADDRESS, i, cString);
    }
    // 6. Sort the off-heap data by calling the foreign function
    radixsort.invoke(pointers, javaStrings.length, MemorySegment.NULL, '\0');
    // 7. Copy the (reordered) strings from off-heap to on-heap
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = pointers.getAtIndex(ValueLayout.ADDRESS, i);
        javaStrings[i] = cString.getUtf8String(0);
    }
} // 8. All off-heap memory is deallocated here
assert Arrays.equals(javaStrings, new String[] {"car", "cat", "dog", "mouse"});  // true
```

让我们仔细看看代码中用到的一些类型，了解它们在外部函数和内存 API 中的功能和目的：

- Linker: 提供了从Java代码访问外部函数和从外部函数访问Java代码的功能。它通过downcall方法句柄允许Java代码链接到外部函数。它还通过生成upcall stubs允许外部函数调用Java方法句柄。有关更多信息，请参见此类型的JavaDoc。
- SymbolLookup: 检索一个或多个库中符号的地址。有关更多信息，请参见此类型的JavaDoc。
- Arena: 控制内存段的生命周期。Arena具有称为竞技场范围的范围。当竞技场关闭时，竞技场范围不再存在。因此，与竞技场范围关联的所有段都无效，它们的支撑内存区域被释放（在适用的情况下），并且在竞技场关闭后不能再访问它们。有关更多信息，请参见此类型的JavaDoc。
- MemorySegment: 提供对连续内存区域的访问。有两种类型的内存段：heap segments（在Java内存堆中）和native segments（在Java内存堆之外）。有关更多信息，请参见此类型的JavaDoc。
- ValueLayout: 对基本数据类型的值进行建模，例如integral值、floating-point值和address值。除此之外，它还为Java原始类型和地址定义了有用的值布局常量。有关更多信息，请参见此类型的JavaDoc。

在 Java 19 中，此功能处于第一次预览状态（以 JEP 424 形式存在），因此语言功能已经完成并收集了开发者反馈。基于这些反馈，Java 20 发生了以下变化：

- 合并了 MemorySegment 和 MemoryAddress 抽象（内存地址现在由零长度的内存段建模）；
- 密封的 MemoryLayout 层次结构得到了增强，以便与 switch 模式匹配一起使用；
- MemorySession 已被分成 Arena 和 SegmentScope，以便跨维护边界共享段。



## 5、JEP 436: 虚拟线程【第二次预览】

自 Java 诞生以来，线程一直是 Java 的一部分，自从 Loom 项目开始，我们渐渐开始将其称为“平台线程”。平台线程在底层操作系统线程上运行 Java 代码，并在代码的整个生命周期中捕获操作系统线程。因此，平台线程的数量受限于可用的操作系统线程数。

然而，现代应用程序可能需要比这更多的线程。例如，在同时处理成千上万个请求时。这就是虚拟线程的作用。虚拟线程是 java.lang.Thread 的实例，它在底层操作系统线程上运行 Java 代码，但不会在代码的整个生命周期中捕获操作系统线程。这意味着许多虚拟线程可以在同一个操作系统线程上运行它们的 Java 代码，有效地共享它。因此，虚拟线程的数量可以比可用的操作系统线程数量多得多。

虚拟线程不仅数量众多，而且创建和处理开销也很低。这意味着一个 Web 框架可以将一个新的虚拟线程专门用于处理请求的任务，并且仍然能够处理成千上万甚至数百万个请求。

**典型用例**：使用虚拟线程不需要学习新概念，但可能需要放弃开发用于应付现代高成本线程的习惯。虚拟线程不仅将帮助应用程序开发人员，而且还将帮助框架设计人员提供易于使用的 API，这些 API 与平台的设计兼容，而不会影响可扩展性。

**创建虚拟线程**：与平台线程一样，虚拟线程也是 java.lang.Thread 的一个实例。因此，可以像使用平台线程一样使用虚拟线程。创建虚拟线程与创建平台线程有所不同，但同样容易：

```java
import java.util.concurrent.Executors;
import java.util.stream.IntStream;

public class ThreadTest {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            IntStream.range(0, 100_000).forEach(i -> {
                executor.submit(() -> {
                    Thread.sleep(1000);
                    return i;
                });
            });
        } // executor.close() 会被自动调用
        // 提交了 10 万个虚拟线程，每个线程休眠 1 秒钟，1秒左右完成
        System.out.println("耗时:" + (System.currentTimeMillis() - start)+"ms");
    }
}

// $ java --enable-preview --source 19 ThreadTest.java
// 注: ThreadTest.java 使用 Java SE 19 的预览功能。
// 注: 有关详细信息，请使用 -Xlint:preview 重新编译。
// 耗时:1309ms
```

**对比 Java 19 有何不同**：此功能处于“第二个预览版”阶段，以便获得更多反馈。除此之外，还有一些API更改已经成为常规功能，不再提供预览。这是因为它们涉及的功能通常很有用，不仅适用于虚拟线程，还包括：

- Thread 类中的新方法：

- - join(Duration);
  - sleep(Duration);
  - threadId();

- Future 中的新方法（用于检查任务状态和结果）

- ExecutorService 扩展 AutoCloseable，以便可以在 try-with-resources 块中使用

- 除此之外，ThreadGroup 的降级也已经成为常规功能。



## 6、JEP 437: 结构化并发【第二次孵化】

Java 19 引入了结构化并发，目的是为了通过结构化并发 API 来简化多线程编程，并不是为了取代java.util.concurrent，目前处于孵化器阶段。

结构化并发将不同线程中运行的多个任务视为单个工作单元，从而简化错误处理、提高可靠性并增强可观察性。也就是说，结构化并发保留了单线程代码的可读性、可维护性和可观察性。

结构化并发的基本 API 是 StructuredTaskScope。StructuredTaskScope 支持将任务拆分为多个并发子任务，在它们自己的线程中执行，并且子任务必须在主任务继续之前完成。

使用 StructuredTaskScope API 确保了许多有价值的属性：

- 短路错误处理：如果子任务失败，则另一个子任务将被取消（如果它尚未完成）。这由 ShutdownOnFailure 实现的取消策略管理；其他策略，如 ShutdownOnSuccess 也是可用的。
- 取消传播：如果在调用 join() 之前或期间中断运行 handle() 的线程，则当该线程退出作用域时，两个分叉将自动取消。
- 清晰可见：上面的代码具有明确的结构：设置子任务，等待它们完成或被取消，然后决定是成功（并处理子任务的结果，这些结果已经完成）还是失败（子任务已经完成，因此没有更多的清理工作）。

顺便说一下，结构化并发正好与虚拟线程同时出现在 Java 中，这绝非偶然。现代 Java 程序可能会使用大量的线程，并且需要正确而稳健地协调它们。结构化并发正好可以提供这种功能，同时还可以使观察工具按照开发人员的理解显示线程。

**对比 Java 19 有何不同**：情况与 Java 19 中的情况大致相同（请参见 JEP 428）。唯一的变化是更新了 StructuredTaskScope，以使其支持在任务作用域中创建的线程继承作用域值。这简化了在线程之间共享不可变数据，详见 JEP 429。

**注意**：JEP 处于孵化器阶段，因此需要将 --enable-preview --add-modules jdk.incubator.concurrent 添加到命令行中，以便能够使用该功能。



## 7、JEP 432: Vector API【第五次孵化】

发展历史：

- Java 16 JEP 338: Vector API (Incubator ) 提供了jdk.incubator.vector来用于矢量计算，作为第一轮的 incubator
- Java 17 JEP 414: Vector API (Second Incubator) 进行改进并作为第二轮的 incubator
- Java 18 JEP 417: Vector API (Third Incubator) 进行改进并作为第三轮的 incubator
- Java 19 JEP 426:Vector API (Fourth Incubator) 作为第四轮的 incubator
- Java 20 JEP 432: Vector API 作为第五轮的 incubator

Vector API 将使得能够表达可靠地在运行时编译为最优向量指令的向量计算。这意味着这些计算将在支持的 CPU 体系结构（x64 和 AArch64）上显着优于等价的标量计算。

向量（Vector）计算是对一个或多个任意长度的一维矩阵进行的数学运算。将向量视为具有动态长度的数组。此外，可以通过索引在常数时间内访问向量中的元素，就像数组一样。

过去 Java 程序员只能在汇编代码级别上编写此类计算。但现在，现代 CPU 支持高级单指令，多数据（SIMD）功能，因此重要性更大，需要利用 SIMD 指令和多个并行操作的性能增益。Vector API 让 Java 程序员更容易实现这一点。

以下是一段代码示例（取自 JEP），它比较了使用数组元素的简单标量计算以及使用 Vector API 的等效计算：

```java
void scalarComputation(float[] a, float[] b, float[] c) {
    for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}

static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_PREFERRED;

void vectorComputation(float[] a, float[] b, float[] c) {
    int i = 0;
    int upperBound = SPECIES.loopBound(a.length);
    for (; i < upperBound; i += SPECIES.length()) {
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i);
        var vb = FloatVector.fromArray(SPECIES, b, i);
        var vc = va.mul(va)
            .add(vb.mul(vb))
            .neg();
        vc.intoArray(c, i);
    }
    for (; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}
```

从 Java 开发者的角度来看，这只是表达标量计算的另一种方式。它可能会显得略微冗长，但另一方面它可以带来惊人的性能提升。

**典型用例**：

- Vector API 提供了一种在 Java 中编写复杂向量算法的方法，例如向量化的 hashCode 实现或专门的数组比较。许多领域可以从中受益，包括机器学习、线性代数、加密、文本处理、金融和 JDK 本身的代码。

**与 Java 19 有何不同**：

- 除了一小部分错误修复和性能增强外，这个特性与 Valhalla 项目的对齐是与 Java 19 最大的不同之处。它的意义是非常明确的，因为向量 API 和 Valhalla 项目都专注于性能提升。
- 回想一下，Valhalla 项目的目标是通过值对象和用户自定义原语增强 Java 对象模型，将面向对象编程的抽象与简单原语的性能特性相结合。
- 一旦 Valhalla 项目的功能可用，将会适应 Vector API 以利用值对象，届时它将被提升为预览功能。



## 8、细项解读

上面列出的是大方面的特性，除此之外还有一些api的更新及废弃，主要见：[JDK 20 Release Notes](https://jdk.java.net/20/release-notes)，这里举几个例子。

### 1、添加项

- Support Unicode 15.0 [Update Unicode Data Files to 15.0.0](https://bugs.openjdk.org/browse/JDK-8284842)
- Add GarbageCollectorMXBean for Remark and Cleanup Pause Time in G1 [JDK-8297247](https://bugs.openjdk.org/browse/JDK-8284842)



### 2、移除项

- Thread.suspend/resume Changed to Throw UnsupportedOperationException [JDK-8249627](https://bugs.openjdk.org/browse/JDK-8284842)

- Thread.Stop Changed to Throw UnsupportedOperationException [JDK-8289610](https://bugs.openjdk.org/browse/JDK-8289610)

- Improved Control of G1 Concurrent Refinement Threads [JDK-8137022](https://bugs.openjdk.org/browse/JDK-8137022)

  > 以下这些参数未来版本移除

  ```ruby
  -XX:-G1UseAdaptiveConcRefinement
  
  -XX:G1ConcRefinementGreenZone=buffer-count
  
  -XX:G1ConcRefinementYellowZone=buffer-count
  
  -XX:G1ConcRefinementRedZone=buffer-count
  
  -XX:G1ConcRefinementThresholdStep=buffer-count
  
  -XX:G1ConcRefinementServiceIntervalMillis=msec
  ```



### 3、废弃项

完整列表见[Java SE 20 deprecated-list](https://cr.openjdk.java.net/~iris/se/20/latestSpec/api/deprecated-list.html)

- java.net.URL Constructors Are Deprecated [JDK-8294241](https://cr.openjdk.java.net/~iris/se/20/latestSpec/api/deprecated-list.html)

  > URL的构造器被废弃，可以使用URL::of(URI, URLStreamHandler)工厂方法替代



### 4、已知问题

- java.lang.Float.floatToFloat16 and java.lang.Float.float16ToFloat May Return Different NaN Results when Optimized by the JIT Compiler ([JDK-8302976](https://bugs.openjdk.org/browse/JDK-8302976), [JDK-8289551](https://bugs.openjdk.org/browse/JDK-8289551), [JDK-8289552](https://bugs.openjdk.org/browse/JDK-8289552))

  > JDK20引入了java.lang.Float.floatToFloat16及java.lang.Float.float16ToFloat方法，在JIT编译优化时可能会返回不同的Nan结果，可以使用如下参数禁用JIT对此的优化

  ```ruby
  -XX:+UnlockDiagnosticVMOptions -XX:DisableIntrinsic=_floatToFloat16,_float16ToFloat
  ```



### 5、其他事项

- Disabled TLS_ECDH_* Cipher Suites ([JDK-8279164](https://bugs.openjdk.org/browse/JDK-8279164))

  > TLS_ECDH_* cipher suites默认被禁用了

- HTTP Response Input Streams Will Throw an IOException on Interrupt ([JDK-8294047](https://bugs.openjdk.org/browse/JDK-8294047))

- HttpClient Default Keep Alive Time is 30 Seconds ([JDK-8297030](https://bugs.openjdk.org/browse/JDK-8297030))

  > http1.1及http2的空闲连接的超时时间从1200秒改为30秒



## 9、参考文献 & 鸣谢

- 一起来学习下 Java 20 的新功能：https://mp.weixin.qq.com/s/Oa2lxQZIw9qyWmnPj1ohHg
- Java 20 新特性概览：https://javaguide.cn/java/new-features/java20.html
- Java 20 新特性：https://segmentfault.com/a/1190000043584234
- Java 新特性文档：https://www.wdbyte.com/java-feature/
- Java New Feature：https://javaguide.cn/java/new-features/
- Java 的新特性：https://segmentfault.com/a/1190000004417288/
- Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
- Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/