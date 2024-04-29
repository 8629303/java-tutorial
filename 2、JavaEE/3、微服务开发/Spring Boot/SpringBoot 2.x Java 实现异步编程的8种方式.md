> 来源：Java实现异步编程的8种方式：https://mp.weixin.qq.com/s/n8Y7yqJyZIKrSxCJFCD5uw

# 一、异步的八种实现方式

异步执行对于开发者来说并不陌生，在实际的开发过程中，很多场景多会使用到异步，相比同步执行，异步可以大大缩短请求链路耗时时间，比如：**「发送短信、邮件、异步更新等」**，这些都是典型的可以通过异步实现的场景。

我们一般可以有如下几种方式实现：

1. 线程 Thread
2. Future
3. 异步框架 CompletableFuture
4. Spring 注解 @Async
5. Spring ApplicationEvent 事件
6. 消息队列
7. 第三方异步框架，比如 Hutool 中的 ThreadUtil
8. Guava 异步



# 二、什么是异步

首先我们先看一个常见的用户下单的场景：

```
            用户下单
               ⬇
            扣减库存
               ⬇
            订单入库
               ⬇
            赠送积分
               ⬇
            发送短信
```

在同步操作中，我们执行到 **「发送短信」** 的时候，我们必须等待这个方法彻底执行完才能执行 **「赠送积分」** 这个操作，如果 **「赠送积分」** 这个动作执行时间较长，发送短信需要等待，这就是典型的同步场景。

实际上，发送短信和赠送积分没有任何的依赖关系，通过异步，我们可以实现`赠送积分`和`发送短信`这两个操作能够同时进行，比如：

```
            用户下单
               ⬇
            扣减库存
               ⬇
            订单入库
        ⬇            ⬇ 
     赠送积分      发送短信
```

这就是所谓的异步，是不是非常简单，下面就说说异步的几种实现方式吧。



# 三、异步编程

## 1、异步编程

```java
public class AsyncThread extends Thread {
    @Override
    public void run() {
        System.out.println("Current thread name:" + Thread.currentThread().getName() + " Send email success!");
    }

    public static void main(String[] args) {
        AsyncThread asyncThread = new AsyncThread();
        asyncThread.run();
    }
}
```

当然如果每次都创建一个`Thread`线程，频繁的创建、销毁，浪费系统资源，我们可以采用线程池：

```java
private ExecutorService executorService = Executors.newCachedThreadPool();

public void fun() {
    executorService.submit(()->log.info("执行业务逻辑..."));
}
```

可以将业务逻辑封装到 Runnable 或 Callable 中，交由线程池来执行



## 2、Future 异步

```java
@Slf4j
public class FutureManager {

    public String execute() throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(1);
        Future<String> future = executor.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {

                System.out.println(" --- task start --- ");
                Thread.sleep(3000);
                System.out.println(" --- task finish ---");
                return "this is future execute final result!!!";
            }
        });

        // 这里需要返回值时会阻塞主线程
        String result = future.get();
        log.info("Future get result: {}", result);
        return result;
    }

    @SneakyThrows
    public static void main(String[] args) {
        FutureManager manager = new FutureManager();
        manager.execute();
    }
}
```

```
 --- task start --- 
 --- task finish ---
 Future get result: this is future execute final result!!!
```

Future 的不足之处的包括以下几点：

1. 无法被动接收异步任务的计算结果：虽然我们可以主动将异步任务提交给线程池中的线程来执行，但是待异步任务执行结束之后，主线程无法得到任务完成与否的通知，它需要通过get方法主动获取任务执行的结果。
2. Future件彼此孤立：有时某一个耗时很长的异步任务执行结束之后，你想利用它返回的结果再做进一步的运算，该运算也会是一个异步任务，两者之间的关系需要程序开发人员手动进行绑定赋予，Future并不能将其形成一个任务流（pipeline），每一个Future都是彼此之间都是孤立的，所以才有了后面的CompletableFuture，CompletableFuture就可以将多个Future串联起来形成任务流。
3. Futrue没有很好的错误处理机制：截止目前，如果某个异步任务在执行发的过程中发生了异常，调用者无法被动感知，必须通过捕获get方法的异常才知晓异步任务执行是否出现了错误，从而在做进一步的判断处理。



## 3、CompletableFuture 实现异步

```java
public class CompletableFutureCompose {
    /**
     * thenAccept子任务和父任务公用同一个线程
     */
    @SneakyThrows
    public static void thenRunAsync() {
        CompletableFuture<Integer> cf1 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread() + " cf1 do something....");
            return 1;
        });
        CompletableFuture<Void> cf2 = cf1.thenRunAsync(() -> {
            System.out.println(Thread.currentThread() + " cf2 do something...");
        });
        // 等待任务1执行完成
        System.out.println("cf1结果->" + cf1.get());
        // 等待任务2执行完成
        System.out.println("cf2结果->" + cf2.get());
    }

    public static void main(String[] args) {
        thenRunAsync();
    }
}
```

我们不需要显式使用 ExecutorService，CompletableFuture 内部使用了 ForkJoinPool 来处理异步任务，如果在某些业务场景我们想自定义自己的异步线程池也是可以的。



## 4、Spring的@Async异步

1、自定义异步线程池

```java
/**
 * 线程池参数配置，多个线程池实现线程池隔离，@Async注解，默认使用系统自定义线程池，
 * 可在项目中设置多个线程池，在异步调用的时候，指明需要调用的线程池名称，比如：@Async("taskName")
**/
@EnableAsync
@Configuration
public class TaskPoolConfig {
    /**
     * 自定义线程池
     **/
    @Bean("taskExecutor")
    public Executor taskExecutor() {
        // 返回可用处理器的Java虚拟机的数量 12
        int i = Runtime.getRuntime().availableProcessors();
        System.out.println("系统最大线程数  ： " + i);
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 核心线程数：线程池创建时候初始化的线程数
        executor.setCorePoolSize(16);
        // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
        executor.setMaxPoolSize(20);
        // 缓冲队列：用来缓冲执行任务的队列。配置队列容量，默认值为Integer.MAX_VALUE
        executor.setQueueCapacity(99999);
        // 活跃时间：允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
        executor.setKeepAliveSeconds(60);
        // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
        executor.setThreadNamePrefix("asyncServiceExecutor -");
        // 缓冲队列满了之后的拒绝策略：由调用线程处理（一般是主线程）
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy());
        // 设置此执行程序应该在关闭时阻止的最大秒数，以便在容器的其余部分继续关闭之前等待剩余的任务完成他们的执行
        executor.setAwaitTerminationSeconds(60);
        // 等待所有的任务结束后再关闭线程池
        executor.setWaitForTasksToCompleteOnShutdown(true);
        return executor;
    }
}
```

2、AsyncService

```java
public interface AsyncService {
    MessageResult sendSms(String callPrefix, String mobile, String actionType, String content);
    MessageResult sendEmail(String email, String subject, String content);
}

@Slf4j
@Service
public class AsyncServiceImpl implements AsyncService {

    @Autowired
    private IMessageHandler mesageHandler;

    @Override
    @Async("taskExecutor")
    public MessageResult sendSms(String callPrefix, String mobile, String actionType, String content) {
        try {

            Thread.sleep(1000);
            mesageHandler.sendSms(callPrefix, mobile, actionType, content);

        } catch (Exception e) {
            log.error("发送短信异常 -> ", e)
        }
    }

    @Override
    @Async("taskExecutor")
    public sendEmail(String email, String subject, String content) {
        try {

            Thread.sleep(1000);
            mesageHandler.sendsendEmail(email, subject, content);

        } catch (Exception e) {
            log.error("发送email异常 -> ", e)
        }
    }
}
```

在实际项目中， 使用@Async调用线程池，推荐等方式是是使用自定义线程池的模式，不推荐直接使用@Async直接实现异步。





## 5、Spring ApplicationEvent 事件实现异步

1、定义事件

```java
public class AsyncSendEmailEvent extends ApplicationEvent {
    /**
     * 邮箱
     **/
    private String email;

    /**
     * 主题
     **/
    private String subject;

    /**
     * 内容
     **/
    private String content;

    /**
     * 接收者
     **/
    private String targetUserId;

}
```

2、定义事件处理器

```java
@Slf4j
@Component
public class AsyncSendEmailEventHandler implements ApplicationListener<AsyncSendEmailEvent> {
    @Autowired
    private IMessageHandler mesageHandler;
    
    @Async("taskExecutor")
    @Override
    public void onApplicationEvent(AsyncSendEmailEvent event) {
        if (event == null) {
            return;
        }

        String email = event.getEmail();
        String subject = event.getSubject();
        String content = event.getContent();
        String targetUserId = event.getTargetUserId();
        mesageHandler.sendsendEmailSms(email, subject, content, targerUserId);
      }
}
```

另外，可能有些时候采用ApplicationEvent实现异步的使用，当程序出现异常错误的时候，需要考虑补偿机制，那么这时候可以结合Spring Retry重试来帮助我们避免这种异常造成数据不一致问题。



## 6、消息队列实现异步

1、回调事件消息生产者

```java
@Slf4j
@Component
public class CallbackProducer {

    @Autowired
    AmqpTemplate amqpTemplate;

    public void sendCallbackMessage(CallbackDTO allbackDTO, final long delayTimes) {

        log.info("生产者发送消息，callbackDTO，{}", callbackDTO);

        amqpTemplate.convertAndSend(CallbackQueueEnum.QUEUE_GENSEE_CALLBACK.getExchange(), CallbackQueueEnum.QUEUE_GENSEE_CALLBACK.getRoutingKey(), JsonMapper.getInstance().toJson(genseeCallbackDTO), new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                //给消息设置延迟毫秒值，通过给消息设置x-delay头来设置消息从交换机发送到队列的延迟时间
                message.getMessageProperties().setHeader("x-delay", delayTimes);
                message.getMessageProperties().setCorrelationId(callbackDTO.getSdkId());
                return message;
            }
        });
    }
}
```

2、回调事件消息消费者

```java
@Slf4j
@Component
@RabbitListener(queues = "message.callback", containerFactory = "rabbitListenerContainerFactory")
public class CallbackConsumer {
    @Autowired
    private IGlobalUserService globalUserService;

    @RabbitHandler
    public void handle(String json, Channel channel, @Headers Map<String, Object> map) throws Exception {

        if (map.get("error") != null) {
            // 否认消息
            channel.basicNack((Long) map.get(AmqpHeaders.DELIVERY_TAG), false, true);
            return;
        }

        try {
            CallbackDTO callbackDTO = JsonMapper.getInstance().fromJson(json, CallbackDTO.class);
            // 执行业务逻辑
            globalUserService.execute(callbackDTO);
            // 消息消息成功手动确认，对应消息确认模式acknowledge-mode: manual
            channel.basicAck((Long) map.get(AmqpHeaders.DELIVERY_TAG), false);
        } catch (Exception e) {
            log.error("回调失败 -> {}", e);
        }
    }
}
```



## 7、Hutool 的 ThreadUtil异步工具类

```java
@Slf4j
public class ThreadUtils {
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            ThreadUtil.execAsync(() -> {
                ThreadLocalRandom threadLocalRandom = ThreadLocalRandom.current();
                int number = threadLocalRandom.nextInt(20) + 1;
                System.out.println(number);
            });
            log.info("当前第：" + i + "个线程");
        }
        log.info("task finish!");
    }
}
```



## 8、Guava 实现异步

Guava 的 ListenableFuture 顾名思义就是可以监听的Future，是对 Java 原生 Future 的扩展增强。我们知道 Future 表示一个异步计算任务，当任务完成时可以得到计算结果。如果我们希望一旦计算完成就拿到结果展示给用户或者做另外的计算，就必须使用另一个线程不断的查询计算状态。这样做，代码复杂，而且效率低下。使用**「Guava ListenableFuture」**可以帮我们检测Future是否完成了，不需要再通过get()方法苦苦等待异步的计算结果，如果完成就自动调用回调函数，这样可以减少并发程序的复杂度。

ListenableFuture 是一个接口，它从 JDK 的 Future 接口继承，添加了`void addListener(Runnable listener, Executor executor)`方法。

我们看下如何使用 ListenableFuture。首先需要定义 ListenableFuture 的实例：

```java
ListeningExecutorService executorService = MoreExecutors.listeningDecorator(Executors.newCachedThreadPool());
final ListenableFuture<Integer> listenableFuture = executorService.submit(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        log.info("callable execute...")
            TimeUnit.SECONDS.sleep(1);
        return 1;
    }
});
```

首先通过 MoreExecutors 类的静态方 法listeningDecorator() 方法初始化一个 ListeningExecutorService() 的方法，然后使用此实例的 submit() 方法即可初始化ListenableFuture对象。

ListenableFuture要做的工作，在Callable接口的实现类中定义，这里只是休眠了1秒钟然后返回一个数字1，有了ListenableFuture实例，可以执行此Future并执行Future完成之后的回调函数。

```java
Futures.addCallback(listenableFuture, new FutureCallback<Integer>() {
    @Override
    public void onSuccess(Integer result) {
        // 成功执行...
        System.out.println("Get listenable future's result with callback " + result);
    }
    @Override
    public void onFailure(Throwable t) {
        // 异常情况处理...
        t.printStackTrace();
    }
});
```





# 四、SpringBoot 优雅的使用多线程

> SpringBoot中如何优雅的使用多线程：https://juejin.cn/post/6844904173977206797

## 1、快速使用

SpringBoot应用中需要添加`@EnableAsync`注解，来开启异步调用，一般还会配置一个线程池，异步的方法交给特定的线程池完成，如下：

```java
@Configuration
@EnableAsync
public class AsyncConfiguration {

    @Bean("doSomethingExecutor")
    public Executor doSomethingExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 核心线程数：线程池创建时候初始化的线程数
        executor.setCorePoolSize(10);
        // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
        executor.setMaxPoolSize(20);
        // 缓冲队列：用来缓冲执行任务的队列
        executor.setQueueCapacity(500);
        // 允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
        executor.setKeepAliveSeconds(60);
        // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
        executor.setThreadNamePrefix("do-something-");
        // 缓冲队列满了之后的拒绝策略：由调用线程处理（一般是主线程）
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy());
        executor.initialize();
        return executor;
    }
}
```

使用的方式非常简单，在需要异步的方法上加`@Async`注解

```java
@RestController
public class AsyncController {

    @Autowired
    private AsyncService asyncService;

    @GetMapping("/open/something")
    public String something() {
        int count = 10;
        for (int i = 0; i < count; i++) {
            asyncService.doSomething("index = " + i);
        }
        return "success";
    }
}


@Slf4j
@Service
public class AsyncService {

    // 指定使用beanname为doSomethingExecutor的线程池
    @Async("doSomethingExecutor")
    public String doSomething(String message) {
        log.info("do something, message={}", message);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            log.error("do something error: ", e);
        }
        return message;
    }
}
```

访问：127.0.0.1:8080/open/something，日志如下：

```bash
2020-04-19 23:42:42.486  INFO 21168 --- [io-8200-exec-17] x.g.b.system.controller.AsyncController  : do something end, time 8 milliseconds
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-1] x.gits.boot.system.service.AsyncService  : do something, message=index = 0
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-5] x.gits.boot.system.service.AsyncService  : do something, message=index = 4
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-4] x.gits.boot.system.service.AsyncService  : do something, message=index = 3
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-6] x.gits.boot.system.service.AsyncService  : do something, message=index = 5
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-9] x.gits.boot.system.service.AsyncService  : do something, message=index = 8
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-8] x.gits.boot.system.service.AsyncService  : do something, message=index = 7
2020-04-19 23:42:42.488  INFO 21168 --- [do-something-10] x.gits.boot.system.service.AsyncService  : do something, message=index = 9
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-7] x.gits.boot.system.service.AsyncService  : do something, message=index = 6
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-2] x.gits.boot.system.service.AsyncService  : do something, message=index = 1
2020-04-19 23:42:42.488  INFO 21168 --- [ do-something-3] x.gits.boot.system.service.AsyncService  : do something, message=index = 2
```

由此可见已经达到异步执行的效果了，并且使用到了咱们配置的线程池。



## 2、获取异步方法返回值

当异步方法有返回值时，如何获取异步方法执行的返回结果呢？这时需要异步调用的方法带有返回值 CompletableFuture。

CompletableFuture 是对 Feature 的增强，Feature 只能处理简单的异步任务，而 CompletableFuture 可以将多个异步任务进行复杂的组合。如下：

```java
@RestController
public class AsyncController {

    @Autowired
    private AsyncService asyncService;

    @SneakyThrows
    @ApiOperation("异步 有返回值")
    @GetMapping("/open/somethings")
    public String somethings() {
        CompletableFuture<String> createOrder = asyncService.doSomething1("create order");
        CompletableFuture<String> reduceAccount = asyncService.doSomething2("reduce account");
        CompletableFuture<String> saveLog = asyncService.doSomething3("save log");

        // 等待所有任务都执行完
        CompletableFuture.allOf(createOrder, reduceAccount, saveLog).join();
        // 获取每个任务的返回结果
        String result = createOrder.get() + reduceAccount.get() + saveLog.get();
        return result;
    }
}


@Slf4j
@Service
public class AsyncService {

    @Async("doSomethingExecutor")
    public CompletableFuture<String> doSomething1(String message) throws InterruptedException {
        log.info("do something1: {}", message);
        Thread.sleep(1000);
        return CompletableFuture.completedFuture("do something1: " + message);
    }

    @Async("doSomethingExecutor")
    public CompletableFuture<String> doSomething2(String message) throws InterruptedException {
        log.info("do something2: {}", message);
        Thread.sleep(1000);
        return CompletableFuture.completedFuture("; do something2: " + message);
    }

    @Async("doSomethingExecutor")
    public CompletableFuture<String> doSomething3(String message) throws InterruptedException {
        log.info("do something3: {}", message);
        Thread.sleep(1000);
        return CompletableFuture.completedFuture("; do something3: " + message);
    }
}
```

访问接口：

```bash
C:\Users\Administrator>curl -X GET "http://localhost:8200/open/somethings" -H "accept: */*"
do something1: create order; do something2: reduce account; do something3: save log
```

控制台上关键日志如下：

```bash
2020-04-20 00:27:42.238  INFO 5672 --- [ do-something-3] x.gits.boot.system.service.AsyncService  : do something3: save log
2020-04-20 00:27:42.238  INFO 5672 --- [ do-something-2] x.gits.boot.system.service.AsyncService  : do something2: reduce account
2020-04-20 00:27:42.238  INFO 5672 --- [ do-something-1] x.gits.boot.system.service.AsyncService  : do something1: create order
```



## 3、注意事项

`@Async`注解会在以下几个场景失效，也就是说明明使用了`@Async`注解，但就没有走多线程。

- 异步方法使用 static 关键词修饰；
- 异步类不是一个 Spring 容器的 Bean（一般使用注解`@Component`和`@Service`，并且能被 Spring 扫描到）；
- SpringBoot 应用中没有添加`@EnableAsync`注解；
- 在同一个类中，一个方法调用另外一个有 @Async 注解的方法，注解不会生效。原因是 @Async 注解的方法，是在代理类中执行的。

需要注意的是： 异步方法使用注解 @Async 的返回值只能为 void 或者 Future 及其子类，当返回结果为其他类型时，方法还是会异步执行，但是返回值都是null，部分源码如下：**AsyncExecutionInterceptor#invoke**

```java
@Override
@Nullable
public Object invoke(final MethodInvocation invocation) throws Throwable {
    Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);
    Method specificMethod = ClassUtils.getMostSpecificMethod(invocation.getMethod(), targetClass);
    final Method userDeclaredMethod = BridgeMethodResolver.findBridgedMethod(specificMethod);

    AsyncTaskExecutor executor = determineAsyncExecutor(userDeclaredMethod);
    if (executor == null) {
        throw new IllegalStateException(
            "No executor specified and no default executor set on AsyncExecutionInterceptor either");
    }

    Callable<Object> task = () -> {
        try {
            Object result = invocation.proceed();
            // 重点看这里：如果返回类型不是Future的实现，那么最后会返回null
            if (result instanceof Future) {
                return ((Future<?>) result).get();
            }
        }
        catch (ExecutionException ex) {
            handleError(ex.getCause(), userDeclaredMethod, invocation.getArguments());
        }
        catch (Throwable ex) {
            handleError(ex, userDeclaredMethod, invocation.getArguments());
        }
        // 重点看这里：如果返回类型不是Future的实现，那么最后会返回null
        return null;
    };

    return doSubmit(task, executor, invocation.getMethod().getReturnType());
}
```

通过上边几个示例，@Async 实际还是通过 Future 或 CompletableFuture 来异步执行的，Spring 又封装了一下，让我们使用的更方便。



