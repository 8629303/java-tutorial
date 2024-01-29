# Java JUC 之 CompletableFuture 实战

以前需要异步执行一个任务时，一般是用Thread或者线程池Executor去创建。如果需要返回值，则是调用Executor.submit获取Future。但是多个线程存在依赖组合，我们又能怎么办？可使用同步组件CountDownLatch、CyclicBarrier等；其实有简单的方法，就是使用线程回调 CompletableFuture。

- 线程任务的创建
- 线程任务的串行执行
- 线程任务的并行执行
- 处理任务结果和异常
- 多任务的简单组合
- 取消执行线程任务
- 任务结果的获取和完成与否判断

## 1、创建异步线程任务

1、根据 supplier 创建 CompletableFuture 任务

```java
// 使用内置线程ForkJoinPool.commonPool()，根据supplier构建执行任务
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
// 指定自定义线程，根据supplier构建执行任务
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
```

2、根据 runnable 创建 CompletableFuture 任务

```java
// 使用内置线程ForkJoinPool.commonPool()，根据runnable构建执行任务
public static CompletableFuture<Void> runAsync(Runnable runnable);
// 指定自定义线程，根据runnable构建执行任务
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);
```

3、操作示例：

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

// runAsync 的使用
CompletableFuture<Void> rFuture = CompletableFuture.runAsync(() -> System.out.println("hello siting"), executor);
// supplyAsync 的使用
CompletableFuture<String> future = CompletableFuture
        .supplyAsync(() -> {
            System.out.print("hello ");
            return "siting";
        }, executor);

// 阻塞等待，runAsync 的future 无返回值，输出null
System.out.println(rFuture.join());
// 阻塞等待
String name = future.join();
System.out.println(name);
// 线程池需要关闭
executor.shutdown();
```

```
hello siting
null
hello siting
```

4、常量值作为 CompletableFuture 返回

```java
// 有时候是需要构建一个常量的 CompletableFuture
public static <U> CompletableFuture<U> completedFuture(U value);
```

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("value");
System.out.println(future.get()); // value
```



## 2、线程任务串行执行

1、任务完成则运行action，不关心上一个任务的结果，无返回值

```java
public CompletableFuture<Void> thenRun(Runnable action);
public CompletableFuture<Void> thenRunAsync(Runnable action);
public CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor);
```

```java
CompletableFuture<Void> future = CompletableFuture
        .supplyAsync(() -> "hello siting", executor)
        .thenRunAsync(() -> System.out.println("OK"), executor);
executor.shutdown();
// --------输出结果--------
// OK
```

2、任务完成则运行action，依赖上一个任务的结果，无返回值

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action);
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor);
```

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<Void> future = CompletableFuture
        .supplyAsync(() -> "hello siting", executor)
        .thenAcceptAsync(System.out::println, executor);
executor.shutdown();
// --------输出结果--------
// hello siting
```

3、任务完成则运行fn，依赖上一个任务的结果，有返回值

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn);
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn);
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor);
```

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<String> future = CompletableFuture
        .supplyAsync(() -> "hello world", executor)
        .thenApplyAsync(data -> {
            System.out.println(data); return "OK";
        }, executor);
System.out.println(future.join());
executor.shutdown();
// --------输出结果--------
// hello world
// OK
```

4、thenCompose - 任务完成则运行fn，依赖上一个任务的结果，有返回值

- 类似 thenApply（区别是 thenCompose 的返回值是 CompletionStage，thenApply 则是返回 U），提供该方法为了和其他 CompletableFuture 任务更好地配套组合使用

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn);
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn,
  Executor executor);
```

```java
// 第一个异步任务，常量任务
CompletableFuture<String> f = CompletableFuture.completedFuture("OK");
// 第二个异步任务
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<String> future = CompletableFuture
        .supplyAsync(() -> "hello world", executor)
        .thenComposeAsync(data -> {
            System.out.println(data); return f; //使用第一个任务作为返回
        }, executor);
System.out.println(future.join());
executor.shutdown();
// --------输出结果--------
// hello world
// OK
```



## 3、线程任务并行执行

1、 两个CompletableFuture[并行]执行完，然后执行action，不依赖上两个任务的结果，无返回值

```java
public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action, Executor executor);
```

```java
// 第一个异步任务，常量任务
CompletableFuture<String> first = CompletableFuture.completedFuture("hello world");
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<Void> future = CompletableFuture
        // 第二个异步任务
        .supplyAsync(() -> "hello siting", executor)
        // () -> System.out.println("OK") 是第三个任务
        .runAfterBothAsync(first, () -> System.out.println("OK"), executor);
executor.shutdown();
// --------输出结果--------
// OK
```

2、 两个CompletableFuture【并行】执行完，然后执行action，依赖上两个任务的结果，无返回值

```java
// 第一个任务完成再运行other，fn再依赖消费两个任务的结果，无返回值
public <U> CompletableFuture<Void> thenAcceptBoth(CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action);
// 两个任务异步完成，fn再依赖消费两个任务的结果，无返回值     
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action);
// 两个任务异步完成（第二个任务用指定线程池执行），fn再依赖消费两个任务的结果，无返回值                
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action, Executor executor);
```

```java
// 第一个异步任务，常量任务
CompletableFuture<String> first = CompletableFuture.completedFuture("hello world");
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<Void> future = CompletableFuture
        // 第二个异步任务
        .supplyAsync(() -> "hello siting", executor)
        // (w, s) -> System.out.println(s) 是第三个任务
        .thenAcceptBothAsync(first, (s, w) -> System.out.println(s), executor);
executor.shutdown();
// --------输出结果--------
// hello siting
```

3、两个CompletableFuture【并行】执行完，然后执行action，依赖上两个任务的结果，有返回值

```java
// 第一个任务完成再运行other，fn再依赖消费两个任务的结果，有返回值
public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, 
  BiFunction<? super T,? super U,? extends V> fn);
// 两个任务异步完成，fn再依赖消费两个任务的结果，有返回值
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn);
// 两个任务异步完成（第二个任务用指定线程池执行），fn再依赖消费两个任务的结果，有返回值        
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn, Executor executor);
```

```java
// 第一个异步任务，常量任务
CompletableFuture<String> first = CompletableFuture.completedFuture("hello world");
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<String> future = CompletableFuture
        // 第二个异步任务
        .supplyAsync(() -> "hello siting", executor)
        // (w, s) -> System.out.println(s) 是第三个任务
        .thenCombineAsync(first, (s, w) -> {
            System.out.println(s);
            return "OK";
        }, executor);
System.out.println(future.join());
executor.shutdown();
// --------输出结果--------
// hello siting
// OK
```



## 4、线程并行执行，谁先执行完则谁触发下一任务

1、上一个任务或者other任务完成, 运行action，不依赖前一任务的结果，无返回值

```java
public CompletableFuture<Void> runAfterEither(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action);
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action, Executor executor);
```

```java
// 第一个异步任务，休眠1秒，保证最晚执行晚
CompletableFuture<String> first = CompletableFuture.supplyAsync(()->{
    try{ Thread.sleep(1000); }catch (Exception e){}
    System.out.println("hello world");
    return "hello world";
});
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<Void> future = CompletableFuture
        // 第二个异步任务
        .supplyAsync(() ->{
            System.out.println("hello siting");
            return "hello siting";
        } , executor)
        // () ->  System.out.println("OK") 是第三个任务
        .runAfterEitherAsync(first, () ->  System.out.println("OK") , executor);
executor.shutdown();
// --------输出结果--------
// hello siting
// OK
```

2、上一个任务或者other任务完成, 运行action，依赖最先完成任务的结果，无返回值

```java
public CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action);
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other,
  Consumer<? super T> action, Executor executor);
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other,
  Consumer<? super T> action, Executor executor);
```

```java
// 第一个异步任务，休眠1秒，保证最晚执行晚
CompletableFuture<String> first = CompletableFuture.supplyAsync(()->{
    try{ Thread.sleep(1000);  }catch (Exception e){}
    return "hello world";
});
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<Void> future = CompletableFuture
        // 第二个异步任务
        .supplyAsync(() -> "hello siting", executor)
        // data -> System.out.println(data) 是第三个任务
        .acceptEitherAsync(first, data -> System.out.println(data) , executor);
executor.shutdown();
// --------输出结果--------
// hello siting
```

3、上一个任务或者other任务完成, 运行fn，依赖最先完成任务的结果，有返回值

```java
public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn);
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn);
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn, Executor executor);
```

```java
// 第一个异步任务，休眠1秒，保证最晚执行晚
CompletableFuture<String> first = CompletableFuture.supplyAsync(()->{
    try{ Thread.sleep(1000);  }catch (Exception e){}
    return "hello world";
});
ExecutorService executor = Executors.newSingleThreadExecutor();
CompletableFuture<String> future = CompletableFuture
        // 第二个异步任务
        .supplyAsync(() -> "hello siting", executor)
        // data -> System.out.println(data) 是第三个任务
        .applyToEitherAsync(first, data ->  {
            System.out.println(data);
            return "OK";
        } , executor);
System.out.println(future);
executor.shutdown();
// --------输出结果--------
// hello siting
// OK
```



## 5、处理任务结果或者异常

1、exceptionally-处理异常

- 如果之前的处理环节有异常问题，则会触发exceptionally的调用相当于 try...catch

```java
public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn);
```

```java
CompletableFuture<Integer> first = CompletableFuture
    .supplyAsync(() -> {
        if (true) {
            throw new RuntimeException("main error!");
        }
        return "hello world";
    })
    .thenApply(data -> 1)
    .exceptionally(e -> {
        e.printStackTrace(); // 异常捕捉处理，前面两个处理环节的日常都能捕获
        return 0;
    });
```

2、 handle-任务完成或者异常时运行fn，返回值为fn的返回

- 如果之前的处理环节有异常问题，则会触发exceptionally的调用相当于 try...catch

```java
public <U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn, Executor executor);
```

```java
CompletableFuture<Integer> first = CompletableFuture
    .supplyAsync(() -> {
        if (true) { throw new RuntimeException("main error!"); }
        return "hello world";
    })
    .thenApply(data -> 1)
    .handleAsync((data,e) -> {
        e.printStackTrace(); // 异常捕捉处理
        return data;
    });
System.out.println(first.join());
// --------输出结果--------
// java.util.concurrent.CompletionException: java.lang.RuntimeException: main error!
//  ... 5 more
// null
```

3、whenComplete-任务完成或者异常时运行action，有返回值

- whenComplete 与 handle 的区别在于，它不参与返回结果的处理，把它当成监听器即可
- 即使异常被处理，在 CompletableFuture 外层，异常也会再次复现
- 使用 whenCompleteAsync 时，返回结果则需要考虑多线程操作问题，毕竟会出现两个线程同时操作一个结果

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T, ? super Throwable> action);
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action);
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action, Executor executor);
```

```java
CompletableFuture<AtomicBoolean> first = CompletableFuture
    .supplyAsync(() -> {
        if (true) {  throw new RuntimeException("main error!"); }
        return "hello world";
    })
    .thenApply(data -> new AtomicBoolean(false))
    .whenCompleteAsync((data,e) -> {
        // 异常捕捉处理, 但是异常还是会在外层复现
        System.out.println(e.getMessage());
    });
first.join();
// --------输出结果--------
// java.lang.RuntimeException: main error!
// Exception in thread "main" java.util.concurrent.CompletionException: java.lang.RuntimeException: main error!
//  ... 5 more
```



## 6、多个任务的简单组合

```
TaskA
TaskB
TaskC
```

```java
// 所有任务需要执行完毕才会触发next Task的执行
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs);
// 任意一任务执行完毕都会触发next Task的执行
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs);
```

```java
CompletableFuture<Void> future = CompletableFuture
    .allOf(CompletableFuture.completedFuture("A"),
           CompletableFuture.completedFuture("B"));
// 全部任务都需要执行完
future.join();
CompletableFuture<Object> future2 = CompletableFuture
    .anyOf(CompletableFuture.completedFuture("C"),
           CompletableFuture.completedFuture("D"));
// 其中一个任务行完即可
future2.join();
```



## 7、取消执行线程任务

```java
// mayInterruptIfRunning 无影响；如果任务未完成,则返回异常
public boolean cancel(boolean mayInterruptIfRunning);
// 任务是否取消
public boolean isCancelled();
```

```java
CompletableFuture<Integer> future = CompletableFuture
    .supplyAsync(() -> {
        try { Thread.sleep(1000);  } catch (Exception e) { }
        return "hello world";
    })
    .thenApply(data -> 1);

System.out.println("任务取消前:" + future.isCancelled());
// 如果任务未完成,则返回异常,需要对使用exceptionally，handle 对结果处理
future.cancel(true);
System.out.println("任务取消后:" + future.isCancelled());
future = future.exceptionally(e -> {
    e.printStackTrace();
    return 0;
});
System.out.println(future.join());
// --------输出结果--------
// 任务取消前:false
// 任务取消后:true
// java.util.concurrent.CancellationException
//  at java.util.concurrent.CompletableFuture.cancel(CompletableFuture.java:2276)
//  at Test.main(Test.java:25)
// 0
```



## 8、任务的获取和完成与否判断

```java
// 任务是否执行完成
public boolean isDone();
//阻塞等待 获取返回值
public T join();
// 阻塞等待 获取返回值,区别是get需要返回受检异常
public T get();
// 等待阻塞一段时间，并获取返回值
public T get(long timeout, TimeUnit unit);
// 未完成则返回指定value
public T getNow(T valueIfAbsent);
// 未完成，使用value作为任务执行的结果，任务结束。需要future.get获取
public boolean complete(T value);
// 未完成，则是异常调用,返回异常结果，任务结束
public boolean completeExceptionally(Throwable ex);
// 判断任务是否因发生异常结束的
public boolean isCompletedExceptionally();
// 强制地将返回值设置为value，无论该之前任务是否完成；类似complete
public void obtrudeValue(T value);
// 强制地让异常抛出，异常返回，无论该之前任务是否完成；类似completeExceptionally
public void obtrudeException(Throwable ex);
```

```java
CompletableFuture<Integer> future = CompletableFuture
    .supplyAsync(() -> {
        try { Thread.sleep(1000);  } catch (Exception e) { }
        return "hello world";
    })
    .thenApply(data -> 1);

System.out.println("任务完成前:" + future.isDone());
future.complete(10);
System.out.println("任务完成后:" + future.join());
// --------输出结果--------
// 任务完成前:false
// 任务完成后:10
```



# 参考文献 & 鸣谢

1. 基础篇：异步编程不会？我教你啊！CompeletableFuture：https://mp.weixin.qq.com/s/siixcALGcVDPGqwNpJdc1w
2. 奇淫巧技，CompletableFuture 异步多线程是真的优雅：https://mp.weixin.qq.com/s/Af6Y9kx3RBPfCpzf5w0OCA
3. CompletableFuture 入门学习：https://www.cnblogs.com/pingoo/p/14238516.html
4. CompletableFuture使用详解：https://blog.csdn.net/sermonlizhi/article/details/123356877

