> - Spring Boot 定时任务全攻略：从入门到实战，一篇文章讲清楚！：https://juejin.cn/post/7510113574222692415
> - 玩转SpringBoot：SpringBoot的几种定时任务实现方式：https://juejin.cn/post/7343153834531946522
> - 3分钟带你掌握Spring Boot中的定时调度服务：https://juejin.cn/post/7392848248488738831

前言

在实际开发中，我们经常会遇到需要定时执行的任务场景，例如：

- 每天凌晨清理日志
- 每小时同步数据
- 每分钟检查订单状态
- 发送邮件、短信提醒等

Spring Boot 提供了非常便捷的定时任务支持 —— `@Scheduled` 注解，让我们可以轻松实现定时功能。

本文将带你从基础概念讲起，手把手教你掌握 Spring Boot 中定时任务的使用方式，并通过多个实用示例帮助你快速上手，适用于企业级项目开发。



## 一、Spring Boot 定时任务简介

### 1、什么是定时任务？

定时任务是指在特定的时间点或周期性地执行某段代码逻辑。Spring Boot 使用的是基于 **TaskScheduler** 的调度机制，默认使用 **单线程** 执行任务。



### 2、支持的定时表达式（cron）

Spring Boot 支持标准的 cron 表达式来定义任务执行频率：

```bash
秒 分 时 日 月 星期 年（可选）
```

常用示例：

| 表达式              | 含义                         |
| ------------------- | ---------------------------- |
| `0 0 1 * * ?`       | 每天凌晨1点执行              |
| `0/5 * * * * ?`     | 每隔5秒执行一次              |
| `0 0/30 9-17 * * ?` | 工作日上午每隔半小时执行一次 |
| `0 0 12 ? * SUN`    | 每周日中午12点执行           |



## 二、启用定时任务的步骤

### Step 1：添加依赖（默认已包含）

Spring Boot 默认已经集成了定时任务功能，无需额外引入依赖。

```xml
<!-- 如果是 Web 项目，通常已有以下依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

### Step 2：启用定时任务功能

在启动类或配置类上添加 `@EnableScheduling` 注解：

```java
@SpringBootApplication
@EnableScheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Step 3：编写定时任务方法

创建一个组件类（如 `ScheduledTasks.java`），并在方法上使用 `@Scheduled` 注解定义执行策略：

```java
@Component
public class ScheduledTasks {

    @Scheduled(fixedRate = 5000)
    public void runEveryFiveSeconds() {
        System.out.println("【定时任务】每5秒执行一次：" + new Date());
    }

    @Scheduled(cron = "0 0 0 * * ?") // 每天0点执行
    public void dailyCleanUp() {
        System.out.println("【定时任务】开始每日清理操作：" + new Date());
    }
}
```



##  三、@Scheduled 注解

### 1、@Scheduled 注解详解

`@Scheduled`注解是`Spring`提供的一个注解，用于标记方法作为定时任务执行。通过 `@Scheduled`注解，开发人员可以轻松地配置方法在指定的时间间隔或时间点执行，实现各种定时任务需求。

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Repeatable(Schedules.class)  
public @interface Scheduled {

    String cron() default "";

    long fixedDelay() default -1;

    long fixedRate() default -1;

    long initialDelay() default -1;
}
```

| 属性           | 描述                                             | 示例                        |
| -------------- | ------------------------------------------------ | --------------------------- |
| `fixedRate`    | 固定频率（毫秒）执行，无论上次是否执行完         | `fixedRate = 5000`（每5秒） |
| `fixedDelay`   | 上次执行完成后延迟固定时间再执行下一次           | `fixedDelay = 5000`         |
| `initialDelay` | 初始延迟时间（需配合 fixedRate/fixedDelay 使用） | `initialDelay = 10000`      |
| `cron`         | 使用 cron 表达式定义执行时间                     | `cron = "0 0 1 * * ?"`      |

 **注意：**

- fixedRate 和 `fixedDelay` 是互斥的，不能同时设置。
- @Scheduled 方法必须为无参、void 返回值。
- 默认情况下，所有任务都在同一个线程中顺序执行。



### 2、@Scheduled 注解示例

```properties
custom.scheduled.cron = 0/5 * * * * ?  
custom.scheduled.fixedRate = 5000  
custom.scheduled.fixedDelay = 10000  
custom.scheduled.initialDelay = 1000
```

```java
@Service  
public class DemoScheduledTaskService {  

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");  

    /**  
     * fixedRate属性设置每隔固定时间执行  
     */  
    @Scheduled(fixedRateString = "${custom.scheduled.fixedRate}")  
    public void reportCurrentTime() {  
        System.out.println("每隔五秒执行一次" + dateFormat.format(new Date()));  
    }  

    /**  
     * cron属性可以设置指定时间执行，cron表达式跟linux一样  
     */  
    @Scheduled(cron = "${custom.scheduled.cron}")  
    public void fixTimeExecution() {  
        System.out.println("指定时间 "+dateFormat.format(new Date())+"执行");  
    }  

    /**  
     * 上一次任务执行完成之后10秒后在执行  
     */  
    @Scheduled(fixedDelayString = "${custom.scheduled.fixedDelay}")  
    public void runWithFixedDelay() {  
        System.out.println("指定时间 "+dateFormat.format(new Date())+"执行");  
    }  

    /**  
     * 初始延迟1秒后开始，然后每10秒执行一次  
     */  
    @Scheduled(initialDelayString = "${custom.scheduled.initialDelay}", fixedDelayString = "${custom.scheduled.fixedDelay}")  
    public void executeWithInitialAndFixedDelay() {  
        System.out.println("指定时间 "+dateFormat.format(new Date())+"执行");  
    }  
}
```



## 四、SchedulingConfigurer 接口

SchedulingConfigurer 接口是 Spring 提供的一个用于定时任务配置的扩展接口，它允许开发人员更细粒度地控制定时任务的执行。通过实现SchedulingConfigurer接口，可以自定义任务调度器（TaskScheduler），配置线程池等参数，以满足不同场景下的定时任务需求



### 1、多线程执行定时任务

如果希望定时任务并发执行而不是串行执行，可以自定义任务调度器，自定义线程池配置类，提升并发性能。

```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        // 设置线程池大小为5
        taskRegistrar.setScheduler(Executors.newScheduledThreadPool(5));	
    }
}
```



### 2、定时任务线程池配置

通过实现 SchedulingConfigurer 接口，重写 configureTasks 方法，自定义任务调度器的配置。此外我们还可以配置线程池，用于控制定时任务执行时的线程数量、并发性等参数。

```java
@Bean(destroyMethod = "shutdown")  
public ThreadPoolTaskScheduler threadPoolTaskScheduler() {  
    ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();  
    scheduler.setPoolSize(5); // 设置线程池大小  
    scheduler.setThreadNamePrefix("scheduled-task-"); // 设置线程名称前缀  
    scheduler.setAwaitTerminationSeconds(60); // 设置终止等待时间  
    // 设置处理拒绝执行的任务异常
    scheduler.setRejectedExecutionHandler((r, executor) -> log.error("Task rejected", r));
    // 处理定时任务执行过程中抛出的未捕获异常
    scheduler.setErrorHandler(e -> log.error("Error in scheduled task", e));
    return scheduler;  
}
```

然后将自定义的`ThreadPoolTaskScheduler`设置到`ScheduledTaskRegistrar`中去：

```java
@Configuration  
@EnableScheduling  
public class CustomSchedulingConfig implements SchedulingConfigurer {

    @Override  
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        // 定时任务逻辑
        taskRegistrar.setTaskScheduler(threadPoolTaskScheduler());
    }
}
```



### 3、定时任务深入配置【SchedulingConfigurer 深入配置】

通过 SchedulingConfigurer 接口，可以更灵活地配置任务调度器和定时任务的执行规则，比如动态注册定时任务、动态修改任务执行规则等。

**1、动态添加定时任务**

在`SchedulingConfigurer`的`configureTasks`方法中，我们可以根据业务需求，从数据库、配置文件或其它动态来源获取定时任务的信息（如Cron表达式、任务执行类等），然后创建对应的`Runnable`或`Callable`实例，并结合`Trigger`（如`CronTrigger`）将其添加到调度器中。相比`@Scheduled`注解，这种方式能够在应用运行时随时添加新的定时任务。

```java
@Override  
public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {  
    ThreadPoolTaskScheduler scheduler = threadPoolTaskScheduler();  
    taskRegistrar.setTaskScheduler(scheduler);  

    List<CronTaskInfo> tasksFromDB = listTasksFromDatabase();  

    for (CronTaskInfo task : tasksFromDB) {  
        Runnable taskRunner = new MyTaskExecutor(task.getTaskData());  
        CronTrigger cronTrigger = new CronTrigger(task.getCronExpression());  
        scheduler.schedule(taskRunner, cronTrigger);  
    }  
}
```

> 关于这里在应用运行时，动态的添加新的任务，我们可以通过事件驱动，轮训检查，消息队列等多种方式，监听到数据库中或者配置文件中新增任务信息，然后通过`SchedulingConfigurer`接口动态创建定时任务。而这种方式是@Scheduled注解做不到的。

**2、修改定时任务规则**

当任务的执行规则需要动态变更时，同样可以在 configureTasks 方法中实现。例如，从数据库获取最新的Cron表达式，然后取消当前任务并重新添加新的任务实例。需要注意的是，取消已有任务通常需要持有对该任务的引用，例如使用 Scheduler 提供的 unschedule方法。

```java
// 假设我们有一个方法用于获取更新后的任务信息  
CronTaskInfo updatedTask = getUpdatedTaskInfoFromDatabase();  

// 取消旧的任务（需要知道旧任务的TriggerKey）  
TriggerKey triggerKey = ...; // 获取旧任务的TriggerKey  
scheduler.unschedule(triggerKey);  

// 创建新任务并设置新的Cron表达式  
MyTaskExecutor taskExecutor = new MyTaskExecutor(updatedTask.getTaskData());  
CronTrigger updatedCronTrigger = new CronTrigger(updatedTask.getCronExpression());  

// 重新调度新任务  
scheduler.schedule(taskRunner, updatedCronTrigger);
```

另外，我们还可以通过添加任务时对其排序或设置优先级等方式间接实现设置定时任务的执行顺序。

通过实现`SchedulingConfigurer`接口，我们可以拥有对定时任务调度的更多控制权，比如自定义线程池、动态添加任务以及调整任务执行策略。还可以根据需求配置定时任务的执行规则，实现各种复杂的定时任务调度需求。这种灵活性使得在复杂环境下，特别是需要动态管理定时任务时，`SchedulingConfigurer`成为了理想的选择。



## 五、实战案例解析

1、案例一：每天凌晨清理日志文件

```java
@Scheduled(cron = "0 0 0 * * ?")
public void cleanLogs() {
    log.info("开始清理日志文件...");
    // 调用删除逻辑，比如删除30天前的日志文件
}
```

2、案例二：每小时同步远程数据

```java
@Scheduled(fixedRate = 60 * 60 * 1000)
public void syncData() {
    log.info("开始同步远程数据...");
    // 调用外部接口获取数据并保存
}
```

3、案例三：每分钟检查超时订单

```java
@Scheduled(fixedRate = 60 * 60 * 1000)
public void syncData() {
    log.info("开始同步远程数据...");
    // 调用外部接口获取数据并保存
}
```



## 六、注意事项与最佳实践

1、注意事项

| 问题                 | 解决方案                                        |
| -------------------- | ----------------------------------------------- |
| 多实例部署下重复执行 | 结合分布式锁（如 Redis 锁）控制只有一台机器执行 |
| 任务异常导致中断     | 使用 try-catch 包裹任务逻辑，记录日志           |
| 任务未执行           | 查看是否遗漏 `@EnableScheduling` 注解           |
| 时间不准确           | 注意服务器时间与时区设置一致                    |
| 任务耗时过长         | 使用异步调用或拆分任务逻辑                      |

2、总结对比表

| 功能       | 描述                                         |
| ---------- | -------------------------------------------- |
| 启用方式   | `@EnableScheduling`                          |
| 核心注解   | `@Scheduled`                                 |
| 支持类型   | fixedRate / fixedDelay / initialDelay / cron |
| 线程模型   | 单线程（默认）或自定义线程池                 |
| 适用场景   | 数据清理、同步、监控、定时通知等             |
| 高可用建议 | 结合 Redis 分布式锁避免重复执行              |

Spring Boot 的定时任务模块虽然简单，但非常实用。无论是小型项目还是大型分布式系统，都可以借助它完成各种定时触发的业务逻辑。

掌握它的基本用法和高级技巧，不仅能让你写出更健壮的后端服务，也能在面试中加分不少。



## 七、其他第三方任务调度框架

除了使用Spring框架提供的 `@Scheduled` 注解和`SchedulingConfigurer`接口外，还有许多第三方的任务调度库可供选择。这些库通常提供了更多的功能和灵活性，以满足各种复杂的任务调度需求。以下是一些常见的第三方任务调度库：

1. **Quartz Scheduler**：
   Quartz 是一个功能强大且灵活的任务调度库，具有丰富的功能，如支持基于`cron`表达式的任务调度、集群支持、作业持久化等。它可以与`Spring`框架集成，并且被广泛应用于各种类型的任务调度应用程序中。
2. **Elastic Job**：
   Elastic Job 是一个分布式任务调度框架，可以轻松实现分布式任务调度和作业执行。它提供了分布式任务执行、作业依赖关系、作业分片等功能，适用于大规模的分布式任务调度场景。
3. **xxl-job**：
   xxl-job 是一个分布式任务调度平台，提供了可视化的任务管理界面和多种任务调度方式，如单机任务、分布式任务、定时任务等。它支持任务执行日志、任务失败重试、动态调整任务执行策略等功能。
4. **PowerJob**：
   PowerJob 是一个开源的分布式任务调度框架，由阿里巴巴集团开发并开源。PowerJob 提供了分布式、高可用的任务调度能力，支持多种任务类型，如定时任务、延时任务、流程任务等。