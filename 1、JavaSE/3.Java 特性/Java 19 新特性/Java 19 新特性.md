# Java 19 新特性

Java 19 在 2022 年 9 月 20 日正式发布，Java 19 不是一个长期支持版本。

JDK 19 只有 7 个新特性：

- JEP 405: Record Patterns：Record 模式匹配【第一次预览】
- JEP 422: Linux/RISC-V Port
- JEP 424: Foreign Function & Memory API：外部函数和内存【预览】
- JEP 425: Virtual Threads：虚拟线程【预览】
- JEP 426: Vector API：向量 API【第四次孵化】
- JEP 427: Pattern Matching for Switch：Switch 模式匹配【第三次预览】
- JEP 428: Structured Concurrency：结构化并发【孵化】

JDK 文档查看及下载地址如下：

- Oracle OpenJDK 下载链接：https://jdk.java.net/archive/
- OpenJDK Java 19 新特性：https://openjdk.org/projects/jdk/19/
- OracleJDK Java 19 新特性：https://www.oracle.com/java/technologies/javase/19-relnote-issues.html#NewFeature
- OracleJDK 19 文档：https://docs.oracle.com/en/java/javase/19/
- JDK 不同版本之间有何差异对比（提供下载 JDK）：https://javaalmanac.io/



## X、预分配 HASH

> 作者：u013433591，JAVA19新特性 代码实战：https://blog.csdn.net/u013433591/article/details/127309450

在声明 List 或 Map 对象的同时，指定元素大小, 好处是在插入120个数据的时候不必进行多次扩容（List 默认的容量大小为10）

```java
List<String> list = new ArrayList<>(120);
Map<String, Integer> map = new HashMap<>(120);
```

根据初始化代码会认为 Map 内部提供了120个存储空间。由于 Map 内部特殊机制，实际上能够在存储90个元素的时候就会进行扩容，计算公式为：

```java
120 × 0.75 = 90 
// 0.75为 map 默认的加载因子
```

因此当需要存储120个元素时，正确初始化的姿势应该为：

```java
// for 120 mappings: 120 / 0.75 = 160
Map<String, Integer> map = new HashMap<>(160); 
```

但是，JDK 19 为解决上述问题，提供了更为简单的声明方式：

```java
Map<String, Integer> map = HashMap.newHashMap(120);
```

Java 19 newHashMap 方法内部的源码：

```java
public static <K, V> HashMap<K, V> newHashMap(int numMappings) {
    return new HashMap<>(calculateHashMapCapacity(numMappings));
}

static final float DEFAULT_LOAD_FACTOR = 0.75f;

static int calculateHashMapCapacity(int numMappings) {
    return (int) Math.ceil(numMappings / (double) DEFAULT_LOAD_FACTOR);
}
```



## 1、JEP 405: Record 模式匹配【第一次预览】

Record 是一种全新的类型，它本质上是一个 final 类，同时所有的属性都是 final 修饰，它会自动编译出 public get、hashcode 、equals、toString 等方法，减少了代码编写量。Record 在 Java 14 中被提出，在 Java 15 中二次预览，在 Java 16 中正式发布。

【操作示例】回顾一下 Record 的使用，编写一个 Dog record 类，定义 name 和 age 属性。

```java
public class Java14Record {
    public record Dog(String name, Integer age) {}
    public static void main(String[] args) {
        Dog dog1 = new Dog("牧羊犬", 1);  // Dog[name=牧羊犬, age=1]
        Dog dog2 = new Dog("田园犬", 2);  // Dog[name=田园犬, age=2]
        Dog dog3 = new Dog("哈士奇", 3);  // Dog[name=哈士奇, age=3]
        System.out.println(dog1);
        System.out.println(dog2);
        System.out.println(dog3);
    }
}
```

在 Java 17 中，为 Record 带来了增强的模式匹配，在使用 instanceof 后，可以进行类型转换。

```java
public class RecordTest {
    public record Dog(String name, Integer age) {}
    public static void main(String[] args) {
        Object dog1 = new Dog("牧羊犬", 1);
        if(dog1 instanceof Dog dogTemp){
            System.out.println(dogTemp.name()); // 牧羊犬
        }
    }
}
// ➜ ./jdk-17/bin/java RecordTest.java
// 牧羊犬
```

在 Java 19 中，甚至可以在使用 instanceof 时直接得到 Record 中的变量引用。

```java
public class RecordTest2 {
    public record Dog(String name, Integer age) {}
    public static void main(String[] args) {
        Object dog1 = new Dog("牧羊犬", 1);
        if (dog1 instanceof Dog(String name, Integer age)) {
            System.out.println(name + ":" + age);
        }
    }
}
// ➜ ./jdk-19/bin/java --enable-preview --source 19 RecordTest2.java
// 注: RecordTest2.java 使用 Java SE 19 的预览功能。
// 注: 有关详细信息，请使用 -Xlint:preview 重新编译。
// 牧羊犬:1
```

> 扩展 & 回顾：Java 14 instanceof 类型推断、Java 16 Record 关键字正式版



## 2、JEP 427: Switch 模式匹配 【第三次预览】

Switch 模式匹配在 Java 17 中已经引入，在 Java 18 中二次预览，现在在 Java 19 中进行三次预览，优化功能如下：

### 1、Switch 模式匹配

Java 17 中引进新特性，允许在 Switch 语句中编写如下代码：

```java
Object obj = "hello";
switch (obj) {
    case String s && s.length() > 5 -> System.out.println(s.toUpperCase());
    case Integer i                  -> System.out.println(i * i);
    default                         -> {}
}
```

Java 19 中为了改变 “**String s && s.length() > 5**” 这种多条件匹配代码引入了新的关键字 **when**, 需要注意的是 **when** 关键字只有在 Switch语句中才起作用，这样做的目的是不需要更改已经存在 用 when 定义好的变量。

```java
Object obj = "hello";
switch (obj) {
    case String s when s.length() > 5 -> System.out.println(s.toUpperCase());
    case Integer i                    -> System.out.println(i * i);
    default -> {}
}
```



### 2、Switch Record 模式匹配

Java 19 增强了 Record instanceof (模式匹配) 语句的语义 ，让代码更加简洁、优雅

```java
public void print(Object object) {
    if (object instanceof Position(int x, int y)) {
        System.out.println("object is a position, x = " + x + ", y = " + y);
    } 
    // else ...
}
public record Position(int x, int y) {}
```

Java 19 也增强了在 Switch 语句中记录匹配的语义

```java
// old code 
public void print(Object object) {
    switch (object) {
        case Position position -> System.out.println("x = " + position.x() + ", y = " + position.y());
        // other cases ...
        default -> {}
    }
}

// new code
public void print(Object object) {
    switch (object) {
        case Position(int x, int y) -> System.out.println("x = " + x + ", y = " + y);
        // other cases ...
        default -> {}
    }
}

public record Position(int x, int y) {}
```

3、Switch Record 嵌套模式匹配，利用上面提到的记录匹配的特性，在可以将对象输出的代码改为：

```java
public void print(Object object) {
    switch (object) {
        case Path(Position(int x1, int y1), Position(int x2, int y2))
            -> System.out.println("x1 = " + x1 + ", y1 = " + y1 + ", x2 = " + x2 + ", y2 = " + y2);
            // other cases ...
    }
}

// 定义一个方法，两个参数的类型一样，代码如下
public record Path(Position from, Position to) {}
public record Position(int x, int y) {}
```



## 3、JEP 425: 虚拟线程 【第一次预览】

**很实用**的一个新特性，从 Java 19 开始逐步的引入虚拟线程，虚拟线程是**轻量级的线程**，可以在显著的减少代码的编写，提高可维护性的同时提高系统的**吞吐量**。

这个 JEP 425 引入了虚拟线程，由 JDK 而不是 OS 提供的轻量级线程实现。虚拟线程的数量可能比操作系统线程的数量大得多。这些虚拟线程有助于提高并发应用程序的吞吐量。

举个例子： 一个平均延迟为 100ms 的应用程序运行在一个包含 10 个内核、20 个 OS 线程的 CPU 上，并发处理 20 个请求，这将充分利用 20 个 OS 线程。假设将吞吐量扩展到每秒 400 个请求。 我们要么需要并发处理 40 个请求（升级 CPU 处理器以支持 40 个 OS 线程），要么将应用程序的平均延迟降低到 50ms；操作系统线程或 CPU 处理器始终是瓶颈，这使得应用程序的吞吐量很难扩展。

在 Java 中，java.lang.Thread 的每个实例都是一个平台线程，它在底层操作系统线程上运行 Java 代码。平台线程的数量受限于操作系统线程的数量，就像上面的案例研究一样。

Virtual Thread 虚拟线程也是 java.lang.Thread 的实例，但它可以在同一个 OS 线程上运行许多个虚拟线程的 Java 代码，这使得虚拟线程的数量可以远大于 OS 线程的数量。同样一个包含10核心和20个OS线程的CPU，我们可以将平台线程转换为虚拟线程，并将并发请求增加到40个，从而达到每秒400个请求的吞吐量。



### 1、引入原因

一直以来，在 Java 并发编程中，Thread 都是十分重要的一部分，Thread 是 Java 中的并发单元，每个 Thread 线程都提供了一个堆栈来存储局部变量和方法调用，以及线程上下文等相关信息。

但问题是线程和进程一样，都是一项**昂贵的资源**，JDK 将 Thread 线程实现为操作系统线程的包装器，也就是说成本很高，而且数量有限。也因此我们会使用线程池来管理线程，同时限制线程的数量。比如常用的 Tomcat 会**为每次请求单独使用一个线程进行请求处理**，同时限制处理请求的线程数量以防止线程过多而崩溃；这很有可能**在 CPU 或网络连接没有耗尽之前，线程数量已经耗尽**，从而限制了 web 服务的吞吐量。

看到这里你可能要说了，可以放弃请求和线程一一对应的方式啊，使用异步编程来解决这个问题，把请求处理分段，在组合成顺序管道，通过一套 API 进行管理，这样就可以使用有限的线程来处理超过线程数量的请求。这当然也是可以的，但是随之而来的问题是：

- 需要额外的学习异步编程。
- 代码复杂度增加，等于放弃了语言的基本顺序组合运算。
- 堆栈上下文信息都变得难以追踪。
- DeBug 困难。
- 和 Java 平台本身的编程风格有冲突，Java 并发单元是 Thread，而这时是异步管道。



### 2、虚拟线程

基于上面的种种原因，Java 19 引入了虚拟线程，在使用体验上和 Thread 没有区别，与之前的 API 互相兼容，但是相比之下虚拟线程资源占用非常少，同时优化了硬件的使用效率，因此非常易用且**不需要被池化**。Virtual Thread 虚拟线程也是 java.lang.Thread 的实例，但它可以在同一个 OS 线程上运行许多个虚拟线程的 Java 代码，这使得虚拟线程的数量可以远大于 OS 线程的数量。

1、下面是一个示例，**创建 10 万个线程**，然后都休眠 1 秒钟最后打印耗时，如果是开传统的 Thread 线程的方式，资源十分紧张；如果是线程池的方式，必定有部分线程在等待线程释放；但是使用虚拟线程的方式，可以瞬间完成。

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
```

执行后发现 1.3 秒执行完毕，速度惊人。

```bash
$ java --enable-preview --source 19 ThreadTest.java
注: ThreadTest.java 使用 Java SE 19 的预览功能。
注: 有关详细信息，请使用 -Xlint:preview 重新编译。
耗时:1309ms
```

> 注意：虚拟线程只是增加程序的吞吐量，并不能提高程序的处理速度。

2、使用传统的OS线程池执行同样多的任务，可能需要花费8、9分钟的时间。

```java
import java.time.Duration;
import java.util.concurrent.Executors;
import java.util.stream.IntStream;

public class ThreadTest {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        try (var executor = Executors.newFixedThreadPool(20)) {
            IntStream.range(0, 10_000).forEach(i -> {
                executor.submit(() -> {
                    Thread.sleep(Duration.ofSeconds(1));
                    return i;
                });
            });
        } // executor.close() 会被自动调用
        // 提交了 10 万个虚拟线程，每个线程休眠 1 秒钟，1秒左右完成
        System.out.println("耗时:" + (System.currentTimeMillis() - start) + "ms");
    }
}
```

```bash
$ java ThreadTest.java
耗时:503500ms
```



## 4、JEP 428: 结构化并发【第一次孵化】

Structured Concurrency，结构化并发是 Java 19 的新特性，一种多线程编程方法，旨在解决多个子任务原子执行的问题，多个子任务之间要么要么全部执行成功，要么失败回滚取消任务。结构化并发的目的是为了通过结构化并发 API 来简化多线程编程，并不是为了取代java.util.concurrent，目前处于孵化器阶段。

结构化并发将不同线程中运行的多个任务视为单个工作单元，从而简化错误处理、提高可靠性并增强可观察性。也就是说，结构化并发保留了单线程代码的可读性、可维护性和可观察性。



### 1、背景问题（非结构化并发）

先看看非结构化并发方式的多线程代码（Java 5 引入的 ExecutorService 用于并行处理任务问题）：

```java
import java.time.Duration;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class JEP428Before {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        JEP428Before obj      = new JEP428Before();
        Response     response = obj.handleUnStructureAPI();
        System.out.println(response);
    }

    private Response handleUnStructureAPI() throws ExecutionException, InterruptedException {
        try (var executor = Executors.newFixedThreadPool(10)) {
            Future<String>  user     = executor.submit(this::findUser);
            Future<Integer> order    = executor.submit(this::fetchOrder);
            String          theUser  = user.get();   // Join findUser
            int             theOrder = order.get();  // Join fetchOrder
            return new Response(theUser, theOrder);
        }
    }

    private String findUser() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(1));
        return "User[name=Sam, age=18]";
    }

    private Integer fetchOrder() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(10));
        return 1;
    }
    
    private record Response(String x, int y) {}
}
```

Future 任务 findUser() 和 fetchOrder() 并行执行，每个任务都可以独立地成功或失败（抛出异常）。

- 如果 findUser() 抛出异常，fetchOrder() 将继续运行它，浪费资源。
- 如果 fetchOrder() 抛出异常， findUser() 将继续运行它，浪费资源。
- 假设 findUser() 需要 1 分钟完成，fetchOrder() 立即失败，但我们没有办法告诉 handle() 停止或取消整个 handle() 过程，handle() 仍将等待 1 分钟以处理它。

上述代码，在一般情况下能够正常执行，但是依然存在不少问题：

- 如果一个子任务发生错误，如何取消其他子任务
- 如何任务撤销，如何取消子任务
- 以上这种情况都有可能发生，整个任务的维护需要相当复杂的代码。



### 2、解决方案（结构发并发）

Java 19 中提供新的结构化并发 API，结构化并发的基本 API 是 [StructuredTaskScope](https://download.java.net/java/early_access/loom/docs/api/jdk.incubator.concurrent/jdk/incubator/concurrent/StructuredTaskScope.html)。StructuredTaskScope 支持将任务拆分为多个并发子任务，在它们自己的线程中执行，并且子任务必须在主任务继续之前完成。它的目的就是改进代码实现、可读性好、可维护性好 用以满足此类问题的场景。

结构化并发可以分成四种情况处理：

- StructuredTaskScope()【Base】：
- StructuredTaskScope.ShutdownOnSuccess()【优先返回成功子线程】
- StructuredTaskScope.ShutdownOnFailure()【优先返回失败子线程】
- StructuredTaskScope() extends Xxx【自定义模式】

【**操作示例 1**：基本用法】直接创建一个 StructuredTaskScope 对象使用：

```java
import jdk.incubator.concurrent.StructuredTaskScope;
import java.time.Duration;
import java.util.concurrent.Future;

public class JEP428After01 {
    public static void main(String[] args) throws InterruptedException {
        JEP428After01 obj      = new JEP428After01();
        Response      response = obj.handleStructureAPI();
        System.out.println(response);
    }

    private Response handleStructureAPI() throws InterruptedException {
        try (var scope = new StructuredTaskScope()) {
            // 创建并提交子任务(使用fork方法派生线程来执行子任务)
            Future<String>  user  = scope.fork(this::findUser);
            Future<Integer> order = scope.fork(this::fetchOrder);
            // 等待子任务/子线程结束
            scope.join();
            return new Response(user.resultNow(), order.resultNow());
        }
    }

    private String findUser() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(1));
        return "User[name=Sam, age=18]";
    }

    private Integer fetchOrder() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(10));
        return 1;
    }

    public record Response(String x, int y) {}
}
```

> 此类情况实际上与上述非结构化并发案例的输出结果是一直的（即便存在异常也是一样的，可以自己尝试对比）

【**操作示例 2**：优先返回成功子线程】上述代码必须等待所有子任务执行完成后，才会返回最终结果。那么有没有可能，在一些特殊的业务场景中，只需要返回任一子任务的结果即可（即谁先执行完成，先返回谁），请看如下代码：

```java
import jdk.incubator.concurrent.StructuredTaskScope;
import java.time.Duration;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

public class JEP428After02 {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        JEP428After02 obj      = new JEP428After02();
        Response      response = obj.handleUnStructureAPI();
        System.out.println(response); // Response[obj=User[name=Sam, age=18]]
    }

    Response handleUnStructureAPI() throws InterruptedException, ExecutionException {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess()) {
            // 创建并提交子任务(使用fork方法派生线程来执行子任务)
            scope.fork(this::findUser);
            scope.fork(this::fetchOrder);
            // 等待子任务, 这里需要注意因为使用ShutdownOnSuccess, 所以在findUser执行完毕后，fetchOrder也被关闭了
            scope.join();
            return new Response(scope.result().toString());
        }
    }

    private String findUser() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(1));
        return "User[name=Sam, age=18]";
    }

    private Integer fetchOrder() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(10));
        return 1;
    }

    record Response(Object obj) {}
}
```

【**操作示例 3**：优先返回失败子线程】如果其中一个子任务失败了，则会取消另外一个在运行的任务。在scope.join()之后，可以使用 resultNow() 或者exceptionNow() 来获取结果。

```java
import jdk.incubator.concurrent.StructuredTaskScope;
import java.time.Duration;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

public class JEP428After03 {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        JEP428After03 obj      = new JEP428After03();
        Response      response = obj.handleUnStructureAPI();
        System.out.println(response);
    }

    Response handleUnStructureAPI() throws InterruptedException, ExecutionException {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            // 创建并提交子任务(使用fork方法派生线程来执行子任务)
            Future<String>  user  = scope.fork(this::findUser);
            Future<Integer> order = scope.fork(this::fetchOrder);
            // 等待子任务
            scope.join();
            // 处理结果或异常(如果没有异常则会正常执行, 若有异常则直接抛异常. 
            // 注意: 即使注释了这行代码, 有异常的情况下会直接停止主线程scope)
            scope.throwIfFailed();
            return new Response(user.resultNow(), order.resultNow());
        }
    }

    private String findUser() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(1));
        return "User[name=Sam, age=18]";
    }

    private Integer fetchOrder() throws InterruptedException {
        Thread.sleep(Duration.ofSeconds(10));
        int i = 1 / 0;
        return 1;
    }

    record Response(String x, int y) {}
}
```

```bash
$ java --enable-preview --source 19 --add-modules jdk.incubator.concurrent JEP428After03.java
WARNING: Using incubator modules: jdk.incubator.concurrent
警告: 使用 incubating 模块: jdk.incubator.concurrent
1 个警告
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
        at jdk.incubator.concurrent/jdk.incubator.concurrent.StructuredTaskScope$ShutdownOnFailure.throwIfFailed(StructuredTaskScope.java:1125)
        at com.example.JEP428After03.handleUnStructureAPI(JEP428After03.java:25)
        at com.example.JEP428After03.main(JEP428After03.java:13)
Caused by: java.lang.ArithmeticException: / by zero
        at com.example.JEP428After03.fetchOrder(JEP428After03.java:37)
        at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:317)
        at java.base/java.lang.VirtualThread.run(VirtualThread.java:287)
        at java.base/java.lang.VirtualThread$VThreadContinuation.lambda$new$0(VirtualThread.java:174)
        at java.base/jdk.internal.vm.Continuation.enter0(Continuation.java:327)
        at java.base/jdk.internal.vm.Continuation.enter(Continuation.java:320)
```

【**操作示例 4**：自定义 StructuredTaskScope】创建自己的业务 Scope：我们不能扩展 ShutdownOnSuccess 或 ShutdownOnFailure，因为它是 fina l类，但是我们仍然可以包装它，组合它。但是我们可以扩展 StructuredTaskScope。假设现在我们要查询旅行社的报价，而不是天气预报，我们需要查询几个不同的报价服务器。以获取最好的价格。我们想通过scope.bestQuotation() 获取最便宜的价格，显然，JDK内置的 Scope 是没有这个方法的。

```java
import jdk.incubator.concurrent.StructuredTaskScope;
import java.time.Duration;
import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.Collection;
import java.util.Comparator;
import java.util.Random;
import java.util.concurrent.ConcurrentLinkedDeque;
import java.util.concurrent.Future;

public class JEP428After04 {
    public static void main(String[] args) throws Exception {

        try (var scope = new Quotation.QuotationScope()) {
            Instant begin = Instant.now();

            scope.fork(Quotation::readQuotationFromA);
            scope.fork(Quotation::readQuotationFromB);
            scope.fork(Quotation::readQuotationFromC);
            scope.join();

            Quotation quotation = scope.bestQuotation();

            Instant end = Instant.now();
            System.out.println("quotation = " + quotation);
            System.out.println("Time is = " + Duration.between(begin, end).toMillis() + " ms");
        }
    }

    public record Quotation(String agency, int price) {
        private static final Random random = new Random();

        /**
         * 自定义 Exception
         */
        public static class QuotationException extends RuntimeException {}

        /**
         * 自定义 Scope
         */
        public static class QuotationScope extends StructuredTaskScope<Quotation> {

            private final Collection<Quotation> quotations = new ConcurrentLinkedDeque<>();
            private final Collection<Throwable> exceptions = new ConcurrentLinkedDeque<>();

            /**
             * 当一个任务完成时，会调用此方法
             */
            @Override
            protected void handleComplete(Future<Quotation> future) {
                switch (future.state()) {
                    case RUNNING -> throw new IllegalStateException(" Future is still running... ");
                    case SUCCESS -> this.quotations.add(future.resultNow());
                    case FAILED -> this.exceptions.add(future.exceptionNow());
                    case CANCELLED -> {
                    }
                }
            }

            /**
             * 获取所有异常
             */
            public QuotationException exceptions() {
                QuotationException quotationException = new QuotationException();
                this.exceptions.forEach(quotationException::addSuppressed);
                return quotationException;
            }


            /**
             * 从队列中拿出最小的
             */
            public Quotation bestQuotation() {
                return quotations.stream()
                        .min(Comparator.comparing(Quotation::price))
                        .orElseThrow(this::exceptions);
            }

        }

        /**
         * 以下模拟请求不同报价服务器
         */
        public static Quotation readQuotationFromA() throws Exception {
            Thread.sleep(Duration.of(random.nextInt(10, 1200), ChronoUnit.MILLIS));
            return new Quotation("agencyA", random.nextInt(100, 1000));
        }

        public static Quotation readQuotationFromB() throws Exception {
            Thread.sleep(Duration.of(random.nextInt(10, 1200), ChronoUnit.MILLIS));
            return new Quotation("agencyB", random.nextInt(100, 1000));
        }

        public static Quotation readQuotationFromC() throws Exception {
            Thread.sleep(Duration.of(random.nextInt(10, 1200), ChronoUnit.MILLIS));
            return new Quotation("agencyC", random.nextInt(100, 1000));
        }
    }
}
```

```bash
$ java --enable-preview --source 19 --add-modules jdk.incubator.concurrent JEP428After04.java
WARNING: Using incubator modules: jdk.incubator.concurrent
警告: 使用 incubating 模块: jdk.incubator.concurrent
1 个警告
quotation = Quotation[agency=agencyA, price=153]
Time is = 1221 ms
```



### 3、结论分析

可以看到使用 StructuredTaskScope 进行并发编程，主要分为四步：

1. 创建并提交子任务，fork(Callable task) 方法将创建一个虚拟线程并执行任务，任务返回一个 Future 对象。

2. 等待子任务结束，join() 方法是一个阻塞调用，将等待所有子任务执行完毕；当然，StructuredTaskScope 也提供有时长限制的方法 **joinUtil(Instant deadline)**。
3. 处理结果/异常，根据不同需求来处理结果或是异常。
4. 结束 StructuredTaskScope 对象，这意味着我们将结束整个任务的调用；由于我们这里使用了 try-with-resource 这种编程模式，所以不需要手动关闭 StructuredTaskScope 对象；当然我们也可以手动结束，调用 shutdown() 方法，或者 close() 方法（AutoCloseable 接口的抽象方法）。

这个编程范式足够结构化，且高度抽象，基本能满足大部分需求。下面我们挑其中几个关键点详细介绍一下：

1. shutdown 与 close：同样是结束 scope 对象的生命周期，shutdown 与 close 实际上是不一样的，shutdown 可以翻译为中止（使 XX 中途停止），close 翻译为关闭，详细对比如下
   - shutdown() 方法将会中止 scope 中所有的子任务，同时 scope 将不会 start 那些还未执行的任务；shutdown() 只能被主线程或是子任务调用；shutdown() 方法适用于当某个子任务执行完成后，其他任务无需再继续执行的场景。
   - close() 方法首先将结束 scope（通过调用 shutdown 的具体实现 implShutdown()），然后等待所有的子任务都执行完毕；close() 方法只能被主线程调用。
2. boolean implShutdown() 将中止没有中止的 scope；中止成功返回 true，已经中止的 scope 将返回 false。
3. scope 的生命周期为：OPEN -> SHUTDOWN -> CLOSED
4. 异常处理：StructuredTaskScope 提供了两种默认的结果/异常处理策略，其 API 为：StructuredTaskScope.ShutdownOnFailure、StructuredTaskScope.ShutdownOnSuccess。
   - ShutdownOnFailure：捕获到第一个异常时将中止整个 scope，否则将等待所有的子任务执行完成（类似 ExecutorSevice.invokeAll 方法，当且仅当子任务都成功的情况）。提供方法 throwIfFailed()，当任意子任务失败时抛出 ExecutionException；当没有任务执行失败，但是有些任务已经被取消，则抛出 CancellationException，前者可以认为是任务真的执行失败的情况（join），后者则可以认为是某个任务超时的情况（joinUtil）。也提供异常包装的方法 throwIfFailed(Function<Throwable, ? extends X> esf)。
   - ShutdownOnSuccess 捕获到第一个成功执行的子任务时中止整个 scope（类似 ExecutorSevice.invokeAny 方法）。提供方法 result() 和异常包装方法 result(Function<Throwable, ? extends X> esf)。
5. 调用 result 方法就不用定义变量接收每个子任务 scope.fork 的结果，在 scope 中甚至没有一个技术对象，可以让我们更专注于业务逻辑。



### 4、参考 & 鸣谢

1. 作者：u013433591，JAVA19新特性 代码实战：https://blog.csdn.net/u013433591/article/details/127309450
2. 作者：XiumingLee，Java 19 新特性 结构化并发编程：https://blog.csdn.net/XiumingLee/article/details/126803317





## 5、JEP 422: Linux/RISC-V Port

将 JDK 移植到 Linux/RISC-V，目前仅支持 RISC-V 的 RV64GV 配置（包含向量指令的通用 64 位 ISA）。将来可能会考虑支持其他 RISC-V 配置，例如通用 32 位配置 (RV32G)。

RISC-V 是一个免费和开源的 RISC 指令集架构 (ISA)，实际上 RISC-V 是一系列相关的 ISA，现在 Java 19 开始对其进行支持。



## 6、JEP 424: 外部函数和内存 API【首次预览】

此功能引入的 API 允许 Java 开发者与 JVM 之外的代码和数据进行交互，通过调用外部函数（JVM 之外）和安全的访问外部内存（非 JVM 管理），让 Java 程序可以调用本机库并处理本机数据，而不会像 JNI 一样存在很多安全风险。

这不是一个新功能，自 Java 14 就已经引入，此次对其进行了性能、通用性、安全性、易用性上的优化。

发展历史如下：

- Java 14 [JEP 370](https://openjdk.java.net/jeps/370) 引入了外部内存访问 API（孵化器）
- Java 15 [JEP 383](https://openjdk.java.net/jeps/383) 引入了外部内存访问 API（第二孵化器）
- Java 16 [JEP 389](https://openjdk.java.net/jeps/389) 引入了外部链接器 API（孵化器）
- Java 16 [JEP 393](https://openjdk.java.net/jeps/393) 引入了外部内存访问 API（第三孵化器）
- Java 17 [JEP 412](https://openjdk.java.net/jeps/412) 引入了外部函数和内存 API（第一次孵化器）
- Java 18 [JEP 419](https://openjdk.java.net/jeps/419) 引入了外部函数和内存 API（第二次孵化器）
- Java 19 [JEP 424](https://openjdk.org/jeps/424) 引入了外部函数和内存 API（第一次预览）

这是一个孵化功能；需要添加 `--add-modules jdk.incubator.foreign` 来编译和运行 Java 代码，Java 18 改进了相关 API ，使之更加简单易用。

该特性主要的目的是为了替换笨重、易出错、执行效率慢的 JNI 本地方法调用。从 Java 14 起，后续的每个版本都有改进、迭代。下面展示一个简单的示例。

```java
import java.lang.foreign.FunctionDescriptor;
import java.lang.foreign.Linker;
import java.lang.foreign.MemorySegment;
import java.lang.foreign.SymbolLookup;
import java.lang.invoke.MethodHandle;
import static java.lang.foreign.SegmentAllocator.implicitAllocator;
import static java.lang.foreign.ValueLayout.ADDRESS;
import static java.lang.foreign.ValueLayout.JAVA_LONG;

public class FFMTest {
    public static void main(String[] args) throws Throwable {
        SymbolLookup stdlib = Linker.nativeLinker().defaultLookup();
        final MethodHandle strlen = Linker.nativeLinker().downcallHandle(
                stdlib.lookup("strlen").orElseThrow(),
                FunctionDescriptor.of(JAVA_LONG, ADDRESS));

        final MemorySegment str = implicitAllocator().allocateUtf8String("Happy Coding!");
        long len = (long) strlen.invoke(str);
        System.out.println("len = " + len);
    }
}
```

代码的效果，是调用本地方法获取字符串的长度。Java 19 新特性大部分为预览版，并不建议在生产环境中使用，因此在学习的过程中，需要增加如下参数：`–enable-preview --add-modules jdk.incubator.concurrent`



## 7、JEP 426:  Vector API 【第四次孵化】

通过将在运行时可靠地编译为支持的 CPU 架构上的向量指令的向量计算表示，与等效的标量计算相比，实现了卓越的性能。此功能已经第四次孵化，在之前 Java 16 ~ Java 18 新特性中都有介绍，这里不做赘述。



## 8、细项解读

上面列出的是大方面的特性，除此之外还有一些api的更新及废弃，主要见：[JDK 19 Release Notes](https://jdk.java.net/19/release-notes)，这里举几个例子。

### 1、添加项

- System Properties for System.out and System.err (https://bugs.openjdk.org/browse/JDK-8283620)

  > 新增stdout.encoding这两个系统属性stderr.encoding

- Support Unicode 14.0 ([JDK-8268081](https://bugs.openjdk.org/browse/JDK-8268081))

  > 新增支持unicode 14.0版本

- Additional Date-Time Formats ([JDK-8176706](https://bugs.openjdk.org/browse/JDK-8176706))

  > 以前java.time.format.DateTimeFormatter/DateTimeFormatterBuilder只支持FormatStyle.FULL/LONG/MEDIUM/SHORT这四种，现在可以自定义，比如`DateTimeFormatter.ofLocalizedPattern("yMMM")`

- Automatic Generation of the CDS Archive ([JDK-8261455](https://bugs.openjdk.org/browse/JDK-8261455))

  > 新增`-XX:+AutoCreateSharedArchive`参数可以自动创建CDS archive，比如`java -XX:+AutoCreateSharedArchive -XX:SharedArchiveFile=app.jsa -cp app.jar App`



### 2、移除项

- Remove Finalizer Implementation in SSLSocketImpl ([JDK-8212136](https://bugs.openjdk.org/browse/JDK-8212136))

  > 移除SSLSocket的finalizer实现

- Removal of Diagnostic Flag GCParallelVerificationEnabled ([JDK-8286304](https://bugs.openjdk.org/browse/JDK-8286304))

  > 移除GCParallelVerificationEnabled参数



### 3、废弃项

完整列表见[Java SE 19 deprecated-list](https://cr.openjdk.java.net/~iris/se/19/latestSpec/api/deprecated-list.html)

- java.lang.ThreadGroup Is degraded ([JDK-8284161](https://bugs.openjdk.org/browse/JDK-8284161))

  > ThreadGroup不能再被显示destroy了，它现在不再与其subgroup保持强引用

- Deprecation of Locale Class Constructors ([JDK-8282819](https://bugs.openjdk.org/browse/JDK-8282819))

  > Locale的构造器被废弃了，可用Locale.of()这个工厂方法替代



### 4、已知问题

- ForkJoinPool and ThreadPoolExecutor do not use Thread::start to Start Worker Threads ([JDK-8284161](https://bugs.openjdk.org/browse/JDK-8284161))

  > ForkJoinPool及ThreadPoolExecutor在这个版本不再使用`Thread::start来启动线程了，因而那些override无参start的工作线程可能会受影响，不过ForkJoinWorkerThread.onStart()不受影响

- InflaterInputStream.read Throws EOFException ([JDK-8292327](https://bugs.openjdk.org/browse/JDK-8292327))

  > InflaterInputStream在这个版本可能抛出EOFException

- G1 Remembered set memory footprint regression after [JDK-8286115](https://bugs.openjdk.org/browse/JDK-8286115) ([JDK-8292654](https://bugs.openjdk.org/browse/JDK-8292654))

  > [JDK-8286115](https://bugs.openjdk.org/browse/JDK-8286115)这个变动了G1的RSet的ergonomic size，会造成本地内存使用增加，可以通过增加G1RemSetArrayOfCardsEntries值来缓解，比如：

  ```bash
  -XX:+UnlockExperimentalVMOptions -XX:G1RemSetArrayOfCardsEntries=128
  ```



### 5、其他事项

- Double.toString(double) and Float.toString(float) may Return Slightly Different Results ([JDK-4511638](https://bugs.openjdk.org/browse/JDK-4511638))

  > Double.toString(2e23), 现在会返回"2.0E23", 而之前的版本会返回"1.9999999999999998E23"

- Make HttpURLConnection Default Keep Alive Timeout Configurable ([JDK-8278067](https://bugs.openjdk.org/browse/JDK-8278067))

  > 新增了http.keepAlive.time.server及http.keepAlive.time.proxy系统属性可以用于修改默认的Keep Alive Timeout

- JVM TI Changes to Support Virtual Threads ([JDK-8284161](https://bugs.openjdk.org/browse/JDK-JDK-8284161))

  > The JVM Tool Interface (JVM TI)已经更新，现可支持虚拟线程

- -Xss may be Rounded up to a Multiple of the System Page Size ([JDK-8236569](https://bugs.openjdk.org/browse/JDK-8236569))

  > 实际的java线程堆栈大小可能与-Xss命令行选项指定的值不同；它可能四舍五入为系统所需的页面大小的倍数。



## 9、参考文献 & 鸣谢

1. Java19 的新特性：https://segmentfault.com/a/1190000042529334
2. Java19 新特性详解：从虚拟线程到结构化并发：https://www.codeischeap.com/java/java19-7jeps/
3. Java 新特性文档：https://www.wdbyte.com/java-feature/
4. Java New Feature：https://javaguide.cn/java/new-features/
5. Java 的新特性：https://segmentfault.com/a/1190000004417288/
6. Java 各个版本特性：https://my.oschina.net/mdxlcj?tab=newest&catalogId=6361172/
7. Java 各个版本新特性：https://blog.csdn.net/wang_luwei/category_10482191.html/