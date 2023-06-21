# Java 8 Date/Time API（JSR 310）

## Date/Time API 概述

### 1、Java 8 新日期时间新特性

Java 8 新日期时间 API 新特性【Date/Time API（JSR 310）】

- 不变性：新的日期/时间API中，所有的类都是不可变的，这对多线程很有好处。
- 关注点分离：借鉴了Joda库的一些优点，新的API将人可读的日期时间和机器时间（unix timestamp）明确分离，它为日期（Date）、时间（Time）、日期时间（DateTime）、时间戳（unix timestamp）以及时区定义了不同的类。
- 清晰：在所有的类中，方法都被明确定义用以完成相同的行为。例如要拿到当前实例我们可以使用now()方法，在所有的类中都定义了format()和parse()方法，而不是像以前那样专门有一个独立的类。为了更好的处理问题，所有的类都使用了工厂模式和策略模式，一旦你使用了其中某个类的方法，与其他类协同工作并不困难。
- 实用操作：所有新的日期/时间API类都实现了一系列方法用以完成通用的任务，如：加、减、格式化、解析、从日期/时间中提取单独部分，等等。
- 可扩展性：新的日期/时间API是工作在ISO-8601日历系统上的，但我们也可以将其应用在非IOS的日历上。



### 2、Java 8 新日期时间包结构

- java.time：父级基础包，常用的时间相关类都在这里，表示日期和时间的 API 的核心。它包括日期、时间、日期和时间组合、时区、瞬间、持续时间和时钟的类。这些类基于 ISO-8601 中定义的日历系统，并且是不可变的和线程安全的。
- java.time.chrono：日历系统包，日历相关的接口（类似Calendar）也包括提供对其他日历系统的API
- java.time.format：格式化和解析包，主要类是 DateTimeFormatter
- java.time.temporal：扩展功能包，提供细粒度的时间控制field、unit，如weeks、months、month-of-year等
- java.time.zone：时区包，支持时区、时区偏移和时区规则的类。如果使用时区，大多数开发人员将只需要使用 ZonedDateTime、ZoneId 或 ZoneOffset。



### 3、Java 8 新日期时间相关类

- 日期和时间、时钟
  1. Clock：时钟，获取其他地区时钟。比如获取目前美国纽约的时间。该类用于获取指定时区的当前日期、时间。该类可取代System类的currentTimeMillis()方法，而且提供了更多方法来获取当前日期、时间。该类提供了大量静态方法来获取Clock对象。
  2. Instant：时间戳（相对于1970年1月1日的偏移量）默认是0时区，所以需要+8小时才是北京时间，与 Date 类不同的是其精确到了纳秒级别
  3. LocalDate：只包含日期，不包含时区，比如：2016-10-20
  4. LocalTime：只包含时间，不包含时区，比如：23:12:10
  5. LocalDateTime：包含日期和时间，内部维护着一对LocalDate与LocalTime实例，同时也不包含时区，比如：2016-10-20 23:14:21
- 期限/时间段：
  1. Duration：用于计算两个“时间”间隔
  2. Period：用于计算两个“日期”间隔
- 时区与偏移量：
  1. ZoneId：该类代表一个时区
  2. ZoneRegion：该类代表地理区域，由于该类不对外开放，实际上可以理解为是 ZoneId。
  3. ZoneOffset：时区偏移量，比如：+8:00
  4. ZonedDateTime：带时区的时间
- 附加的类型：
  1. Year 存储年，如“2020”
  2. YearMonth 存储年和月，如“2020-10”，可用于信用卡上的到期
  3. Month：定义了一月到十二月的枚举值。存储一个月。如“十一月”
  4. MonthDay：存储月和日，如“12-14”，可用于存储生日
  5. DayOfWeek：存储一周中的一天，如“Tuesday”
  6. OffsetTime：存储与UTC没有日期的时间和偏移量
  7. OffsetDateTime：存储与UTC的日期时间和偏移量
- 几个特殊的接口：
  1. Temporal：日期-时间获取值的字段
  2. TemporalAdjuster：Temporal对象转换，实现自定义
  3. ChronoLocalDate：日历系统接口（下面是它的四个子类）
- Java中使用的历法是ISO 8601日历系统，它是世界民用历法（也称为公历）平年有365天，闰年是366天。此外Java8还提供了4套其他历法：
  1. ThaiBuddhistDate：泰国佛教历
  2. MinguoDate：中华民国历
  3. JapaneseDate：日本历
  4. HijrahDate：伊斯兰历



## java.time.Clock

> 1. 【Java 8 新特性】Java Clock 详解：https://blog.csdn.net/qq_31635851/article/details/117778881
> 2. JAVA8之 日期时间时区之 Clock和Instant 笔记：https://blog.csdn.net/kfepiza/article/details/115494220

### 1、Clock 类说明

Clock 表示一个带时区的时钟，Clock 的实例用于查找当前时刻，可以使用存储的时区来解释当前时刻以查找当前日期和时间，可以获取时间戳和时区。某种程度上可以使用时钟代替 System.currentTimeMillis() 和 TimeZone.getDefault()。

它是个抽象类，一共有四个子类（四个内部类）：

- SystemClock ：系统默认本地时钟【系统时间 + 系统时区】
- OffsetClock ：偏移时钟，时间偏移量的单位是Duration【它将给出过去或未来的瞬间】
- FixedClock ：始终返回相同瞬间的时钟，通常使用于测试【它是不变时钟【时刻 + 时区】】
- TickClock ：偏移时钟，时间偏移量的单位是纳秒【它是周期计时，截取时间到最接近的上个周期或下个周期的时间】

Clock 提供了下面这几个常用的方法（这几个方法在实现类里都有对应的实现）：

```java
public static Clock systemUTC();         // 获取格林尼治时钟
public static Clock systemDefaultZone(); // 获取本地时钟
public static Clock system(ZoneId zone); // 获取给定时区的时钟
public static Clock fixed(Instant fixedInstant, ZoneId zone); // 获得始终返回相同时刻的时钟
public static Clock offset(Clock baseClock, Duration offsetDuration); // 从指定时钟返回瞬时值并添加了指定的持续时间
public static Clock tick(Clock baseClock, Duration tickDuration); // 从指定时钟被截断到指定持续时间的最接近值的瞬间
public static Clock tickMinutes(ZoneId zone); // 返回指定时区的时钟,该时钟将在整分钟内立即返回当前瞬间
public static Clock tickSeconds(ZoneId zone); // 返回指定时区的时钟,该时钟将以整秒为单位返回当前瞬间
public static Clock tickMillis(ZoneId zone);  // 返回给定时区的时钟,该时钟将在整毫秒内立即返回当前瞬间,JDK9时新增

--------下面方法由上面四个子类实现---------
public abstract ZoneId	getZone();           // 获取时钟的时区
public abstract Clock withZone(ZoneId zone); // 指定时钟的时区, 并返回当前时钟的新实例
public abstract Instant instant();           // 获取时钟的当前Instant对象
public long millis();                        // 获取时钟的当前毫秒数值
```

```java
import java.time.Clock;
import java.time.Duration;
import java.time.LocalDateTime;
import java.time.ZoneId;

public class JavaDateTest {
    public static void main(String[] args) {
        // 系统时区默认时间, 通过clock.instant()方法获取当前时刻
        Clock clock = Clock.systemDefaultZone();
        System.out.println(clock); // SystemClock[Asia/Shanghai]
        System.out.println(clock.getZone()); // Asia/Shanghai
        System.out.println("当前时刻为：" + clock.instant()); // 当前时刻为：2022-02-18T08:25:12.954071700Z

        // 世界协调时UTC
        Clock clockUTC = Clock.systemUTC();
        System.out.println(clockUTC); // SystemClock[Z]
        System.out.println(clockUTC.getZone()); // Z
        System.out.println("当前时刻为：" + clockUTC.instant()); // 当前时刻为：2022-02-18T08:25:12.973084700Z

        // 获取Clock对应的毫秒数，与System.currentTimeMillis()输出相同
        System.out.println(Clock.systemDefaultZone().millis()); // 1645172712973
        System.out.println(Clock.systemUTC().millis());         // 1645172712973
        System.out.println(System.currentTimeMillis());         // 1645172712973

        // 在clock基础上增加6000秒，返回新的Clock
        Clock clockSet = Clock.offset(clockUTC, Duration.ofSeconds(6000));
        System.out.println(clockSet); // OffsetClock[SystemClock[Z],PT1H40M]
        System.out.println(clockSet.getZone()); // Z
        System.out.println("当前时刻为：" + clockSet.instant()); // 当前时刻为：2022-02-18T10:05:12.974077200Z

        // 纽约时间
        Clock clockNewYork = Clock.system(ZoneId.of("America/New_York"));
        // Current DateTime with NewYork clock: 2022-02-18T03:25:12.975086200
        System.out.println("Current DateTime with NewYork clock: " + LocalDateTime.now(clockNewYork));
        System.out.println(clockNewYork.millis()); // 1645172712977

        // 返回具有不同时区的此时钟的副本, withZone(ZoneId zoneId)
        Clock systemDefaultZone = Clock.systemDefaultZone();
        Clock withZone = systemDefaultZone.withZone(ZoneId.of("America/New_York"));
        System.out.println(systemDefaultZone); // SystemClock[Asia/Shanghai]
        System.out.println(withZone); // SystemClock[America/New_York]

        /**
         * Clock tick(Clock baseClock, Duration tickDuration):此方法获得一个时钟，该时钟返回从指定时钟被截断到指定持续时间的最接近值的瞬间
         * Clock tickMinutes(ZoneId zone):此方法获得一个时钟，该时钟使用最佳的可用系统时钟返回整分钟的当前即时滴答
         * Clock tickSeconds(ZoneId zone):此方法获得一个时钟，该时钟使用最佳可用系统时钟返回整秒的当前即时滴答。
         */
        Clock clock1 = Clock.systemDefaultZone();
        Clock clock2 = Clock.tick(clock1, Duration.ofDays(1));
        System.out.println("Clock1 : " + clock1.instant()); // Clock1 : 2022-02-18T08:25:12.977085300Z
        System.out.println("Clock2 : " + clock2.instant()); // Clock2 : 2022-02-18T00:00:00Z

        Clock clock3 = Clock.systemDefaultZone();
        Clock clock4 = Clock.tickMinutes(ZoneId.systemDefault());
        System.out.println("Clock3 : " + clock3.instant()); // Clock3 : 2022-02-18T08:25:12.977085300Z
        System.out.println("Clock4 : " + clock4.instant()); // Clock4 : 2022-02-18T08:25:00Z

        Clock clock5 = Clock.systemDefaultZone();
        Clock clock6 = Clock.tickSeconds(ZoneId.systemDefault());
        System.out.println("Clock5 : " + clock5.instant()); // Clock5 : 2022-02-18T08:25:12.978077500Z
        System.out.println("Clock6 : " + clock6.instant()); // Clock6 : 2022-02-18T08:25:12Z
    }
}
```



### 2、SystemClock

SystemClock 是系统默认的本地时钟。先查看 Clock 获取 SystemClock 实例的方法：

```java
static Clock systemUTC();         // 获取格林尼治时钟
static Clock systemDefaultZone(); // 获取本地时钟
static Clock system(ZoneId zone); // 获取指定时区时钟
```

【操作示例 1】对比获取不同时区的 SystemClock，并打印该对象，可以看出ToString方法会打印出时区，可以得出结论：SystemClock 会根据时间变化而变化。

```java
Clock now = Clock.systemDefaultZone();
Clock utc = Clock.systemUTC();
Clock hk  = Clock.system(ZoneId.of("Asia/Hong_Kong"));
System.out.println(now + " => " + utc + " => " + hk);
for (int i = 0; i < 10; i++) {
    TimeUnit.SECONDS.sleep(1);
    System.out.println(now.instant() + " => " + utc.instant() + " => " + hk.instant());
}
```

```
SystemClock[Asia/Shanghai] => SystemClock[Z] => SystemClock[Asia/Hong_Kong]
2023-05-18T09:20:08.589080900Z => 2023-05-18T09:20:08.589080900Z => 2023-05-18T09:20:08.589080900Z
2023-05-18T09:20:09.605467800Z => 2023-05-18T09:20:09.605467800Z => 2023-05-18T09:20:09.605467800Z
2023-05-18T09:20:10.620433200Z => 2023-05-18T09:20:10.620433200Z => 2023-05-18T09:20:10.620433200Z
2023-05-18T09:20:11.633925200Z => 2023-05-18T09:20:11.633925200Z => 2023-05-18T09:20:11.633925200Z
2023-05-18T09:20:12.635048100Z => 2023-05-18T09:20:12.635048100Z => 2023-05-18T09:20:12.635048100Z
2023-05-18T09:20:13.646319400Z => 2023-05-18T09:20:13.646319400Z => 2023-05-18T09:20:13.646319400Z
2023-05-18T09:20:14.661263300Z => 2023-05-18T09:20:14.661263300Z => 2023-05-18T09:20:14.661263300Z
2023-05-18T09:20:15.674768100Z => 2023-05-18T09:20:15.674768100Z => 2023-05-18T09:20:15.674768100Z
2023-05-18T09:20:16.676647500Z => 2023-05-18T09:20:16.676647500Z => 2023-05-18T09:20:16.676647500Z
2023-05-18T09:20:17.691215900Z => 2023-05-18T09:20:17.691215900Z => 2023-05-18T09:20:17.691215900Z
```

【操作示例 2】使用不同时区的 SystemClock，并打印出他们的毫秒数。输出居然完全一样。这就要看下源码了。

```java
Clock clock = Clock.systemDefaultZone();
Clock utc   = Clock.systemUTC();
Clock hk    = Clock.system(ZoneId.of("Asia/Hong_Kong"));
System.out.println(clock.millis());             // 1683885989671
System.out.println(utc.millis());               // 1683885989671
System.out.println(hk.millis());                // 1683885989671
System.out.println(System.currentTimeMillis()); // 1683885989671
```

查看源码：

1. Clock.systemDefaultZone()，用的是系统默认的时区 ZoneId.systemDefault()，可以看出最终调用的是 System.currentTimeMillis()

   ```java
   // Clock.systemDefaultZone()
   public static Clock systemDefaultZone() {
       return new SystemClock(ZoneId.systemDefault());
   }
   
   // Clock.SystemClock 内部类
   static final class SystemClock extends Clock implements Serializable {
       SystemClock(ZoneId zone) {
           this.zone = zone;
           this.offset = OFFSET_SEED;
       }
       @Override
       public long millis() {
           return System.currentTimeMillis();
       }
   }
   ```

2. Clock.systemUTC()，用的是UTC时区 ZoneOffset.UTC。可以看出最终调用的也是 System.currentTimeMillis()

   ```java
   // Clock.systemUTC()
   public static Clock systemUTC() {
       return new SystemClock(ZoneOffset.UTC);
   }
   
   // Clock.SystemClock 内部类
   static final class SystemClock extends Clock implements Serializable {
       SystemClock(ZoneId zone) {
           this.zone = zone;
           this.offset = OFFSET_SEED;
       }
       @Override
       public long millis() {
           return System.currentTimeMillis();
       }
   }
   ```

3. 结论：Clock.systemDefaultZone() 和Clock.systemUTC() 获取的 millis() 时间戳是一样的，就是对应时区的差别。



### 3、OffsetClock

OffsetClock 是偏移时钟，时间偏移量的单位是 Duration，它将给出过去或未来的瞬间。如果持续时间为负数，则该时刻将比当前时刻早。如果我们传递零持续时间，我们将得到基础时钟。由源码可知，使用 Clock.offset 方法 返回的是 OffsetClock 实例对象。

```java
public static Clock offset(Clock baseClock, Duration offsetDuration) {
    Objects.requireNonNull(baseClock, "baseClock");
    Objects.requireNonNull(offsetDuration, "offsetDuration");
    if (offsetDuration.equals(Duration.ZERO)) {
        return baseClock;
    }
    return new OffsetClock(baseClock, offsetDuration);
}
```

- baseClock：是将被添加或减去持续时间的时钟。
- offsetDuration：是添加或减去基本时钟的持续时间。
- Clock.OffsetClock：返回以基础时钟为基础的、添加或减去持续时间的时钟。

【操作示例 1】持续时间为秒，这里使用 Clock.offset + Duration.ofSeconds

```java
// 初始化baseClock
Clock baseClock = Clock.systemDefaultZone();
System.out.println(baseClock.instant());

// 获取的clock将会比baseClock快10秒
Clock clock = Clock.offset(baseClock, Duration.ofSeconds(10));
System.out.println(clock.instant());

// 获取的clock将会比baseClock慢10秒
clock = Clock.offset(baseClock, Duration.ofSeconds(-10));
System.out.println(clock.instant());

// 获取的clock将与baseClock相同
clock = Clock.offset(baseClock, Duration.ZERO);
System.out.println(clock.instant());
```

```
2023-05-19T02:52:38.739985700Z
2023-05-19T02:52:48.750369700Z
2023-05-19T02:52:28.751028800Z
2023-05-19T02:52:38.751028800Z
```

【操作示例 2】持续时间为分钟，这里使用 Clock.offset + Duration.ofMinutes

```java
// 初始化baseClock
Clock baseClock = Clock.systemDefaultZone();
System.out.println(baseClock.instant());

// 获取的clock将会比baseClock快10秒
Clock clock = Clock.offset(baseClock, Duration.ofMinutes(15));
System.out.println(clock.instant());

// 获取的clock将会比baseClock慢10秒
clock = Clock.offset(baseClock, Duration.ofMinutes(-15));
System.out.println(clock.instant());

// 获取的clock将与baseClock相同
clock = Clock.offset(baseClock, Duration.ZERO);
System.out.println(clock.instant());
```

```
2023-05-19T02:53:58.893225500Z
2023-05-19T03:08:58.907222400Z
2023-05-19T02:38:58.908222200Z
2023-05-19T02:53:58.908222200Z
```

【操作示例 3】持续时间为小时，这里使用 Clock.offset + Duration.ofHours

```java
// 初始化baseClock
Clock baseClock = Clock.systemDefaultZone();
System.out.println(baseClock.instant());

// 获取的clock将会比baseClock快10秒
Clock clock = Clock.offset(baseClock, Duration.ofHours(5));
System.out.println(clock.instant());

// 获取的clock将会比baseClock慢10秒
clock = Clock.offset(baseClock, Duration.ofHours(-5));
System.out.println(clock.instant());

// 获取的clock将与baseClock相同
clock = Clock.offset(baseClock, Duration.ZERO);
System.out.println(clock.instant());
```

```
2023-05-19T02:54:58.056963200Z
2023-05-19T07:54:58.073961200Z
2023-05-18T21:54:58.073961200Z
2023-05-19T02:54:58.073961200Z
```

【操作示例 4】持续时间为天，这里使用 Clock.offset + Duration.ofDays

```java
// 初始化baseClock
Clock baseClock = Clock.systemDefaultZone();
System.out.println(baseClock.instant());

// 获取的clock将会比baseClock快10秒
Clock clock = Clock.offset(baseClock, Duration.ofDays(8));
System.out.println(clock.instant());

// 获取的clock将会比baseClock慢10秒
clock = Clock.offset(baseClock, Duration.ofDays(-8));
System.out.println(clock.instant());

// 获取的clock将与baseClock相同
clock = Clock.offset(baseClock, Duration.ZERO);
System.out.println(clock.instant());
```

```
2023-05-19T02:55:36.584781700Z
2023-05-27T02:55:36.599465200Z
2023-05-11T02:55:36.599465200Z
2023-05-19T02:55:36.599465200Z
```



### 4、FixedClock

1、FixedClock 是时间不变时钟【时刻 + 时区】。使用 Clock.fixed() 获取，需要传递 instant 和 zone，并将返回具有固定瞬间的时钟。

```java
public static Clock fixed(Instant fixedInstant, ZoneId zone);
```

【操作示例 1】传递 instant 和 zone，验证获取的时钟会不会改变

```java
Instant instant     = Instant.now();
Clock   fixedClock1 = Clock.fixed(instant, ZoneId.of("Asia/Shanghai"));
Clock   fixedClock2 = Clock.fixed(instant, ZoneId.of("GMT"));
System.out.println("CN 时区的Clock: " + fixedClock1);
System.out.println("GMT时区的Clock: " + fixedClock2);
TimeUnit.SECONDS.sleep(10);
System.out.println("======休眠十秒后再次打印 CN 和 GMT 时区的时钟======");
System.out.println("CN 时区的Clock: " + fixedClock1);
System.out.println("GMT时区的Clock: " + fixedClock2);
```

```
CN 时区的Clock: FixedClock[2023-05-18T09:42:28.062425Z,Asia/Shanghai]
GMT时区的Clock: FixedClock[2023-05-18T09:42:28.062425Z,GMT]
======休眠十秒后再次打印 CN 和 GMT 时区的时钟======
CN 时区的Clock: FixedClock[2023-05-18T09:42:28.062425Z,Asia/Shanghai]
GMT时区的Clock: FixedClock[2023-05-18T09:42:28.062425Z,GMT]
```

由运行结果可知：

- 返回的结果是有带对应时区的，可是打印出来的时间已经是UTC时区的时间。
- Clock.fixed 创建的固定时钟，始终提供与指定相同的时刻，强制睡眠10秒，但是时刻没变。

【操作示例 2】与 Offset 方法更配，由上面可知 Clock.fixed 得到一个固定的时钟，那要添加时间或者减去时间就要用到 Offset 方法：

```java
Clock clock      = Clock.systemDefaultZone();
Clock fixedClock = Clock.fixed(clock.instant(), ZoneId.of("Asia/Shanghai"));
Clock clockAdd   = Clock.offset(clock, Duration.ofMinutes(20));
Clock clockSub   = Clock.offset(clock, Duration.ofMinutes(-10));
System.out.println("初始时间  : " + fixedClock.instant());
System.out.println("初始时间  : " + clock.instant());
System.out.println("加了20分钟: " + clockAdd.instant());
System.out.println("减了10分钟: " + clockSub.instant());
```

```
初始时间  : 2023-05-18T10:07:42.618318700Z
初始时间  : 2023-05-18T10:07:42.647308900Z
加了20分钟: 2023-05-18T10:27:42.647308900Z
减了10分钟: 2023-05-18T09:57:42.647308900Z
```



### 5、TickClock

TickDuration 是周期计时，截取时间到最接近的上个周期或下个周期的时间。注意：TickDuration 不会把当前时间点作为周期的起始时间

Clock 主要提供下面四个方法获取 TickDuration：

```java
// 构造的时钟的计时单位是自定义的偏移量单位
public static Clock tick(Clock baseClock, Duration tickDuration); 
// 构造的时钟的计时单位是分 
public static Clock tickMinutes(ZoneId zone);
// 构造的时钟的计时单位是秒
public static Clock tickSeconds(ZoneId zone);
// 构造的时钟的计时单位是毫秒, JDK9新增
public static Clock tickMillis(ZoneId zone);
```

【操作示例 1】使用 tick() 方法创建一个滴答间隔为 3s 的时钟，每 1s 钟查看一下它的时间

```java
// 系统默认时区时钟
Clock clock = Clock.systemDefaultZone();
// 滴答时间间隔为3秒的时钟,
// 当实际时间数据是 0 或 1、2秒时, 从它那里得到的读数都是 0 秒。当实际时间数据是 3或 4、5秒时, 从它那里得到的读数都是 3 秒.
Clock tick = Clock.tick(clock, Duration.ofSeconds(3));

for (int i = 0; i < 10; i++) {
    System.out.println(clock.instant()+" => "+tick.instant());
    TimeUnit.SECONDS.sleep(1);
}
```

输出结果如下：可以看到两个时钟每秒钟的计数是不同的：左边每每秒都会 +1，右边每3秒 +3。

```
2023-05-18T01:50:01.232944400Z => 2023-05-18T01:50:00Z
2023-05-18T01:50:02.297367200Z => 2023-05-18T01:50:00Z
2023-05-18T01:50:03.311686100Z => 2023-05-18T01:50:03Z
2023-05-18T01:50:04.313331600Z => 2023-05-18T01:50:03Z
2023-05-18T01:50:05.317201500Z => 2023-05-18T01:50:03Z
2023-05-18T01:50:06.331406800Z => 2023-05-18T01:50:06Z
2023-05-18T01:50:07.331837800Z => 2023-05-18T01:50:06Z
2023-05-18T01:50:08.345469300Z => 2023-05-18T01:50:06Z
2023-05-18T01:50:09.361165200Z => 2023-05-18T01:50:09Z
2023-05-18T01:50:10.374052900Z => 2023-05-18T01:50:09Z
```

【操作示例 2】tickSeconds(ZoneId zone) : 创建一个滴答间隔为 1 秒的时钟（效果与操作示例 1类似，只是间隔变为了 1s）

```java
// 系统默认时区时钟
Clock clock = Clock.systemDefaultZone();
// 获取滴答间隔为1秒的钟表, 从它那里得到的读数都是任意某1秒。当实际时间数据是当前秒数
Clock tick = Clock.tickSeconds(ZoneId.systemDefault());

for (int i = 0; i < 10; i++) {
    System.out.println(clock.instant() + " => " + tick.instant());
    TimeUnit.SECONDS.sleep(1);
}
```

输出结果如下：可以看到两个时钟每秒钟的计数基本相同：左边每每秒都会 +1，右边每1秒 +1。

```
2023-05-18T02:01:07.472099400Z => 2023-05-18T02:01:07Z
2023-05-18T02:01:08.522817600Z => 2023-05-18T02:01:08Z
2023-05-18T02:01:09.528345200Z => 2023-05-18T02:01:09Z
2023-05-18T02:01:10.534259100Z => 2023-05-18T02:01:10Z
2023-05-18T02:01:11.549237300Z => 2023-05-18T02:01:11Z
2023-05-18T02:01:12.549270300Z => 2023-05-18T02:01:12Z
2023-05-18T02:01:13.559978100Z => 2023-05-18T02:01:13Z
2023-05-18T02:01:14.575497400Z => 2023-05-18T02:01:14Z
2023-05-18T02:01:15.580159800Z => 2023-05-18T02:01:15Z
2023-05-18T02:01:16.582723500Z => 2023-05-18T02:01:16Z
```

【操作示例 3】tickMinutes(ZoneId zone) ：创建一个滴答间隔为1分钟的时钟（效果与操作示例 1类似，只是间隔变为了 1m）

```java
// 系统默认时区时钟
Clock clock = Clock.systemDefaultZone();
// 获取滴答间隔为1分钟的钟表, 当实际时间为任意秒时, 从它那里得到的读数都是 0 秒.
Clock tick = Clock.tickMinutes(ZoneId.systemDefault());

for (int i = 0; i < 10; i++) {
    System.out.println(clock.instant() + " => " + tick.instant());
    TimeUnit.SECONDS.sleep(1);
}
```

输出结果如下：可以看到右边时钟一直没变，因为没有达到1分钟，当达到1分钟时右边会+1，左边每每秒都会 +1。

```
2023-05-18T02:04:16.706961300Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:17.766889900Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:18.777887700Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:19.783824200Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:20.799448800Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:21.814170900Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:22.815569500Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:23.828352300Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:24.842314400Z => 2023-05-18T02:04:00Z
2023-05-18T02:04:25.856003100Z => 2023-05-18T02:04:00Z
```

【操作示例 4】tickMillis(ZoneId zone) ：创建一个滴答间隔为1毫秒的时钟（效果与操作示例 1类似，只是间隔变为了 1 毫秒）

```java
// 系统默认时区时钟
Clock clock = Clock.systemDefaultZone();
// 获取滴答间隔为1毫秒的钟表
Clock tick = Clock.tickMillis(ZoneId.systemDefault());
// 这里把休眠也改成了1毫秒
for (int i = 0; i < 10; i++) {
    System.out.println(clock.instant() + " => " + tick.instant());
    TimeUnit.MILLISECONDS.sleep(1);
}
```

输出结果如下：可以看到，左右两边的毫秒都有在跳动。每毫秒都有在变化

```
2023-05-18T02:11:01.402662900Z => 2023-05-18T02:11:01.402Z
2023-05-18T02:11:01.463671600Z => 2023-05-18T02:11:01.463Z
2023-05-18T02:11:01.464672400Z => 2023-05-18T02:11:01.464Z
2023-05-18T02:11:01.465672600Z => 2023-05-18T02:11:01.465Z
2023-05-18T02:11:01.467670400Z => 2023-05-18T02:11:01.467Z
2023-05-18T02:11:01.468685600Z => 2023-05-18T02:11:01.468Z
2023-05-18T02:11:01.469689900Z => 2023-05-18T02:11:01.469Z
2023-05-18T02:11:01.471736600Z => 2023-05-18T02:11:01.471Z
2023-05-18T02:11:01.473704900Z => 2023-05-18T02:11:01.473Z
2023-05-18T02:11:01.475715900Z => 2023-05-18T02:11:01.475Z
```



## java.time.Instant

> 1. JAVA8之 日期时间时区之 Clock和Instant 笔记：https://blog.csdn.net/kfepiza/article/details/115494220
> 2. Java8 Clock Instant 用法及代码示例：https://vimsky.com/examples/usage/java-8-clock-instant-method-with-examples.html
> 3. Java8 Instant时间戳使用小记：https://www.php1.cn/detail/Java8Instant_Shi_afb2f78b.html
> 4. JAVA时间戳类Instant：https://mp.weixin.qq.com/s/ncoAsCDE3BuCUSJlx99niA

### 1、Instant 类的说明

Instant 本质上表示的是“时间戳”，它是以 Unix 元年（也就是 UTC 时区从 1970 年 1 月 1 日 00:00 ）到当前时刻所经过的时间（精度可到纳秒）。这个时间并不是指北京时间或东京时间而是指世界时间。用 Instant.now() 获取当前时间戳和 System.currentTimeMillis() 效果一样，输出的就是从纪元秒到此刻的毫秒数！

Instant 与时区无关，不过 Instant 本身又带有时区，默认 UTC 的格林威治时区（0 时区），注意：它只能表示 UTC 时区，不能设置其他时区。因此其他时间类和 Instant 互转时，需要指定自己的时区。

Instant 类用于表示时间轴上的特定时刻。它继承了 Object 类并实现 Comparable 接口。类的声明如下：

```java
public final class Instant implements Temporal, TemporalAdjuster, Comparable<Instant>, Serializable {}
```

1、创建 Instant 实例，获取系统的当前时间 now

```java
// 通过Instant创建Instant实例 返回: return Clock.systemUTC().instant();
Instant now = Instant.now();
// 控制台输出：now = 2023-12-29T06:32:49.480Z （以ISO-8601格式输出)
System.out.println("now = " + now);
```

**这是输出的世界标准时间，其中T表示时分秒的开始（或者日期与时间的间隔），Z表示这是一个世界标准时间**。

Instant 是时间戳，是指世界标准时格林威治时间 1970 年 01 月 01 日 00 时 00 分 00 秒（北京时间 1970 年 01 月 01 日 08 时 00 分 00 秒）起至现在的总秒数，Instant 本身实际上是指明时区了，是 0 时区（也就是比北京时间少8小时）。

2、获取当前时区的时间（本地时间），通过方法 Instant.now().atZone(ZoneId.systemDefault()) 获取当前地区的时间

```java
ZonedDateTime zOnedDateTime = Instant.now().atZone(ZoneId.systemDefault());
System.out.println(zOnedDateTime); // 2023-05-15T14:15:35.229517400+08:00[Asia/Shanghai]
System.out.println(zOnedDateTime.getZone()); // Asia/Shanghai
```



### 2、Instant 成员变量

Instant 一共有三个静态常量和两个成员变量。

1、三个静态常量：

```java
public final class Instant implements Temporal, TemporalAdjuster, Comparable<Instant>, Serializable {
    // Constant for the 1970-01-01T00:00:00Z epoch instant.
    public static final Instant EPOCH = new Instant(0, 0);
    public static final Instant MIN = Instant.ofEpochSecond(MIN_SECOND, 0);
    public static final Instant MAX = Instant.ofEpochSecond(MAX_SECOND, 999_999_999);
    // The minimum supported epoch second.
    private static final long MIN_SECOND = -31557014167219200L;
    // The maximum supported epoch second.
    private static final long MAX_SECOND = 31556889864403199L;
}
```

- EPOCH 表示 1970-01-01T00:00:00Z (Z表示UTC)：**可看到实例化方法new Instant(0, 0); Instant本身只保存 seconds 和 nanos两个成员变量, 没有时区, 但代表UTC**。
- MAX 和 MIN 表示最大值和最小值的实例：可表示1970前后十亿年。
- MIN_SECOND 和 MAX_SECOND 表示秒范围正负范围：推断可轻松表示1970正负亿级年份。

2、两个成员变量：Instant 里有两个核心的字段，一个是秒的时间戳，另一个是纳秒的时间戳。

```java
public final class Instant implements Temporal, TemporalAdjuster, Comparable<Instant>, Serializable {
    private final long seconds; // 秒
    private final int nanos;    // 纳秒
}
```

- 只有两个成员变量 seconds 和 nanos 并且是 final 的。
- 可以看出 Instant 用**秒**和**纳秒**确定时刻，区分精度为纳秒，用 final 修饰表示不可修改，这也是它线程安全的原因。

【操作示例 1】在 Instant 时间线上存在三个重要的点位，最大点、最小点、原点也就是说小于1970-01-01的时间戳就为负数，超过1970-01-01的时间戳就为正数。

```java
// 时间线上最大点  +1000000000-12-31T23:59:59.999999999Z
System.out.println(Instant.MAX);

// 时间线上最小点  -1000000000-01-01T00:00:00Z
System.out.println(Instant.MIN);

// 时间线上原点    1970-01-01T00:00:00Z
System.out.println(Instant.EPOCH);

// 输出结果为      -8369623
System.out.println(Instant.parse("1969-09-26T03:06:17.323Z").getEpochSecond());
```

【操作示例 2】Instant 提供 toEpochMilli() 方法与 System.currentTimeMillis() 返回都是毫秒级时间戳，Instant 多了更精确的纳秒级时间戳

```java
Instant now    = Instant.now();
long    millis = System.currentTimeMillis();
System.out.println("以ISO-8601格式输出: " + now); // 以ISO-8601格式输出: now:2023-05-13T09:02:39.280503700Z
System.out.println(now.getEpochSecond()); // 秒　: 1683968559
System.out.println(now.getNano());        // 纳秒: 280503700
System.out.println(now.toEpochMilli());   // 毫秒: 1683968559280
System.out.println(millis);               // 毫秒: 1683968559280
```



### 3、Instant 成员方法

如下是 Instant 类中所有的方法，具体分为如下几类：

1. 静态方法，获取对象

   ```java
   static Instant now(); // 从系统时钟获取当前瞬间
   static Instant now(Clock clock); // 从指定时钟获取当前时刻
   static Instant ofEpochMilli(long epochMilli); // 从1970-01-01T00:00:00Z的纪元中使用毫秒获得Instant的实例
   static Instant ofEpochSecond(long epochSecond); // 使用1970-01-01T00:00:00Z时代的秒数获得Instant的实例
   static Instant ofEpochSecond(long epochSecond, long nanoAdjustment); // 使用1970-01-01T00:00:00Z和纳秒级秒的秒数获得Instant的实例
   static Instant parse(CharSequence text); // 从文本字符串中获取Instant的实例，例如2007-12-03T10:15:30.00Z
   static Instant from(TemporalAccessor temporal); // 从时态对象获取Instant的实例(TemporalAccessor不能为LocalXxx)
   
   // 还可以通过Date的Instant toInstant()方法获取Instant对象
   Instant instant = new Date().toInstant();
   ```

2. 时间计算

   ```java
   // 加
   Instant plus(long amountToAdd, TemporalUnit unit); // 返回此瞬间的副本，并添加指定的数量
   Instant plus(TemporalAmount amountToAdd); // 返回此瞬间的副本，并添加指定的数量
   Instant plusMillis(long millisToAdd); // 返回此瞬间的副本，并添加指定的持续时间(以毫秒为单位)
   Instant plusNanos(long nanosToAdd); // 返回此瞬间的副本，并添加指定的持续时间(以纳秒为单位)
   Instant plusSeconds(long secondsToAdd); // 返回此瞬间的副本，并添加指定的持续时间(以秒为单位)
   
   // 减
   Instant minus(long amountToSubtract, TemporalUnit unit); // 返回此瞬间的副本，并减去指定的数量
   Instant minus(TemporalAmount amountToSubtract); // 返回此瞬间的副本，并减去指定的数量
   Instant minusMillis(long millisToSubtract); // 返回此瞬间的副本，并减去指定的持续时间(以毫秒为单位)
   Instant minusNanos(long nanosToSubtract); // 返回此瞬间的副本，并减去指定的持续时间(以纳秒为单位)
   Instant minusSeconds(long secondsToSubtract); // 返回此瞬间的副本，并减去指定的持续时间(以秒为单位)
   
   // 自定义加减
   Instant with(TemporalAdjuster adjuster); // 返回此瞬间的调整副本
   Instant with(TemporalField field, long newValue); // 返回此瞬间的副本，并将指定的字段设置为新值
   ```

3. 信息获取

   ```java
   int  get(TemporalField field); // 从此瞬间获取指定字段的值作为int
   long getLong(TemporalField field); // 从此瞬间获取指定字段的值为long
   long getEpochSecond(); // 返回此瞬间到1970-01-01T00：00：00Z的秒数
   int  getNano(); // 从时间线开始，获取从第二个开始的纳秒数
   ```

4. 对象统计

   ```java
   long toEpochMilli(); //自1970-01-01T00：00：00Z时代以来的毫秒数
   ```

5. 信息判断

   ```java
   boolean isAfter(Instant otherInstant);    // 检查此瞬间是否在指定的瞬间之后
   boolean isBefore(Instant otherInstant);   // 检查此瞬间是否在指定的瞬间之前
   boolean isSupported(TemporalField field); // 检查是否支持指定的字段
   boolean isSupported(TemporalUnit unit);   // 检查指定的单元是否受支持
   int compareTo(Instant otherInstant);      // 将此瞬间与指定的瞬间进行比较
   ```

6. 对象转换

   ```java
   Temporal adjustInto(Temporal temporal);     // 调整指定的时态对象以获得此瞬间
   OffsetDateTime atOffset(ZoneOffset offset); // 将此瞬间与偏移量组合以创建OffsetDateTime
   ZonedDateTime atZone(ZoneId zone);          // 将此瞬间与偏移量组合以创建OffsetDateTime
   ```

7. 其他方法

   ```java
   String toString(); // 使用ISO-8601表示的此瞬间的字符串表示。
   long until(Temporal endExclusive, TemporalUnit unit); // 计算2各Instant时间差返回long
   <R> R query(TemporalQuery<R> query); // 使用指定的查询查询此瞬间
   ValueRange range(TemporalField field); // 获取指定字段的有效值范围
   Instant truncatedTo(TemporalUnit unit); // 将截断的Instant的副本返回到指定的单位
   ```

如下为常用方法：

| 方法名                                            | 描述                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| static Instant now()                              | 通过以UTC（本初子午线）为基准的此刻瞬时时间返回Instant实例   |
| static Instant now(Clock clock)                   | 从指定时钟获取当前时刻（Instant本身实际上是指明时区了，是0时区） |
| static Instant ofEpochMilli(long epochMilli)      | 通过毫秒数获取Instant实例（从1970-01-01T00：00：00Z 开始）   |
| static Instant ofEpochSecond(long epochSecond)    | 通过秒数获取Instant实例（从1970-01-01T00：00：00Z 开始）     |
| static Instant ofEpochSecond(long e, long na)     | 通过秒数和纳秒数获取Instant实例（从1970-01-01T00：00：00Z 开始） |
| static Instant parse(final CharSequence text)     | 通过时间字符串转换成Instant实例                              |
| public OffsetDateTime atOffset(ZoneOffset offset) | 偏移时区，根据时区进行偏移量修正，北京时间应该+8             |
| public ZonedDateTime atZone(ZoneId zone)          | 获取系统默认时区时间的方法，ZoneId是时区的编号               |
| public long getEpochSecond()                      | 获取从1970-01-01 00:00:00 到当前时间的秒值（10位秒数）       |
| public int getNano()                              | 把获取到的当前时间的秒数换算成纳秒                           |
| public long toEpochMilli()                        | 获取从1970-01-01 00:00:00 到当前时间的毫秒值（13位毫秒数）   |



### 4、Instant 代码示例

【操作示例 1】获取 Instant 实例对象的四种方式：

```java
/**
 * 实例方式一：通过静态方法now(),获得UTC(本初子午线)的此刻瞬时时间的实例对象
 * 输出内容（默认时间比北京时间相差8小时）：2023-05-14T01:51:07.586869400Z（Z表示本初子午线）
 * 注:不建议使用 Instant 查看当前时间点, 因为 Instant是没有时区的。但是 Instant 加上时区后，可以转化为 ZonedDateTime
 */
Instant instant1 = Instant.now();
System.out.println(instant1); // 2023-05-14T01:51:07.586869400Z

/**
 * 实例化方式二：通过给定毫秒数或秒数，获取Instant实例
 */
System.out.println(Instant.ofEpochMilli(Clock.systemDefaultZone().millis())); // 2023-05-14T01:51:07.667Z
System.out.println(Instant.ofEpochMilli(Clock.systemUTC().millis())); // 2023-05-14T01:51:07.667Z
System.out.println(Instant.ofEpochMilli(new Date().getTime())); // 2023-05-14T01:51:07.667Z
System.out.println(Instant.ofEpochMilli(System.currentTimeMillis())); // 2023-05-14T01:51:07.667Z
System.out.println(Instant.ofEpochSecond(System.currentTimeMillis() / 1000)); // 2023-05-14T01:51:07Z

/**
 * 实例化方式三：将字符串转换成Instant
 */
System.out.println(Instant.parse("2023-05-14T01:51:07.667Z")); // 2023-05-14T01:51:07.667Z

/**
 * 实例化方式四：从TemporalAccessor对象中获取Instant实例, TemporalAccessor不能为LocalDateTime无时区的时间
 */
System.out.println(Instant.from(OffsetDateTime.now())); // 2023-05-19T08:09:04.993788200Z
System.out.println(Instant.from(ZonedDateTime.now()));  // 2023-05-19T08:09:05.005939100Z
```

【操作示例 2】Long 型时间戳 Instant 对象

```java
// 1626796436 为秒级时间戳
Instant       ins = Instant.ofEpochSecond(1626796436);
ZonedDateTime zdt = ins.atZone(ZoneId.systemDefault());
System.out.println("秒级时间戳转化：" + zdt);    // 秒级时间戳转化：2021-07-20T23:53:56+08:00[Asia/Shanghai]
// 1626796436111l 为秒级时间戳
Instant       ins1 = Instant.ofEpochMilli(1626796436111l);
ZonedDateTime zdt1 = ins1.atZone(ZoneId.systemDefault());
System.out.println("毫秒级时间戳转化：" + zdt1); // 毫秒级时间戳转化：2021-07-20T23:53:56.111+08:00[Asia/Shanghai]
```

【操作示例 3】获取 Instant 的秒、纳秒、毫秒的时间戳。

```java
/**
 * long getEpochSecond():获取当前时间戳的秒数:
 * long toEpochMilli():获取当前时间戳的毫秒:
 * int getNano():获取当前时间点(抛开整数秒不算)的纳秒数
 *   如: 12345.12秒，抛开整数秒不算,则为0.12秒，那么instant.getNano()的结果为 0.12 * 1000_000_000 = 120_000_000
 * int  get(TemporalField field);
 */
Instant instant = Instant.now();
System.out.println("秒数 　-> " + instant.getEpochSecond()); // 秒数 　-> 1644997084
System.out.println("毫秒数 -> " + instant.toEpochMilli());   // 毫秒数 -> 1644997084046
System.out.println("纳秒数 -> " + instant.getNano());        // 纳秒数 -> 46179600

System.out.println("MilliSecond Field: " + instant.get(ChronoField.MILLI_OF_SECOND)); // 46
System.out.println("Nano of Second: " + instant.get(ChronoField.NANO_OF_SECOND));  // 46179600
instant.get(ChronoField.ERA);
// Exception:
// java.time.temporal.UnsupportedTemporalTypeException: Unsupported field: Era
```

【操作示例 4】Instant 是没有时区的，但是 Instant 加上时区后，可以转化为 ZonedDateTime。

```java
// Instant 与 时间偏移量 的相互转换, 注: 从 1970-01-01 00:00:00 开始计算偏移
Instant instant = Instant.now();
// 对时间进行时区偏移修正，北京时间应+8，输出内容可以发现->(Z变为+8:00)
OffsetDateTime odt = instant.atOffset(ZoneOffset.ofHours(8));
// 设置时区后，显示时间为北京时间了(可以发现后面带上了时区)
ZonedDateTime zdt = instant.atZone(ZoneId.systemDefault());
System.out.println(instant); // 2023-05-14T03:51:06.810011200Z
System.out.println(odt);     // 2023-05-14T11:51:06.810011200+08:00
System.out.println(zdt);     // 2023-05-14T11:51:06.810011200+08:00[Asia/Shanghai]
```

【操作示例 5】Instant 的时间加减

```java
/**
 * Instant的时间加、减：由于北京时间比UTC时间晚8小时，所以我们需要得出北京的瞬时时间，需要加8小时
 */
Instant instant = Instant.now().plus(8, ChronoUnit.HOURS);
System.out.println("原(北京瞬时)instant -> " + instant);
// 原(北京瞬时)instant -> 2023-05-14T12:03:29.245841500Z
Instant plusRes = instant.plus(1, ChronoUnit.HOURS);
System.out.println("原(北京瞬时)instant + 1小时,结果是 -> " + plusRes);
// 原(北京瞬时)instant + 1小时,结果是 -> 2023-05-14T13:03:29.245841500Z
Instant minusRes = instant.minus(2, ChronoUnit.HOURS);
System.out.println("原(北京瞬时)instant - 2小时,结果是 -> " + minusRes);
// 原(北京瞬时)instant - 2小时,结果是 -> 2023-05-14T10:03:29.245841500Z
```

【操作示例 6】比较两个 Instant 之间大小

```java
/**
 * 判断两个Instant之间谁早谁晚
 */
// 将Clock转换成Instant
Instant instantOne = Instant.now(Clock.systemDefaultZone());
// 对时间进行时区偏移修正，北京时间应+8
Instant instantTwo    = instantOne.plus(8, ChronoUnit.HOURS);
boolean isAfterResult = instantOne.isAfter(instantTwo);
boolean isBeforeResult = instantOne.isBefore(instantTwo);
System.out.println("instantOne晚于instantTwo ?  " + isAfterResult);  // instantOne晚于instantTwo ?  false
System.out.println("instantOne早于instantTwo ?  " + isBeforeResult); // instantOne早于instantTwo ?  true

// 0: ==, 1: >, -1: <
System.out.println(instantOne.compareTo(instantOne)); // 0
System.out.println(instantTwo.compareTo(instantOne)); // 1
System.out.println(instantOne.compareTo(instantTwo)); // -1
```

【操作示例 7】range() 方法获取指定字段的有效值范围

```java
Instant instant = Instant.parse("2018-10-28T19:34:50.63Z");
System.out.println("Instant: " + instant);
// Instant: 2018-10-28T19:34:50.630Z

// get range of MILLI_OF_SECOND field
// from instant using range method
ValueRange range1 = instant.range(ChronoField.MILLI_OF_SECOND);
// print range of MILLI_OF_SECOND
System.out.println("Range of MILLI_OF_SECOND: " + range1);
// Range of MILLI_OF_SECOND: 0 - 999

// get range of NANO_OF_SECOND field
// from instant using range method
ValueRange range2 = instant.range(ChronoField.NANO_OF_SECOND);
// print range of NANO_OF_SECOND
System.out.println("Range of NANO_OF_SECOND: " + range2);
// Range of NANO_OF_SECOND: 0 - 999999999

// 获取UnsupportedTemporalTypeException
ValueRange secondvalue = instant.range(ChronoField.ERA);
// Exception:
// java.time.temporal.UnsupportedTemporalTypeException: Unsupported field: Era
```

【操作示例 8】】until() 方法，用于使用TemporalUnit计算两个Instant对象之间的时间

```java
// create Instant objects
Instant instant1 = Instant.parse("2019-01-03T19:35:50.00Z");
Instant instant2 = Instant.parse("2019-01-04T13:18:59.00Z");

// 查看2个时间戳相差多少分钟
long result = instant1.until(instant2, ChronoUnit.MINUTES);
System.out.println("Result in Minutes: " + result);  // Result in Minutes: 1063

// 查看2个时间戳相差多少天
result = instant1.until(instant2, ChronoUnit.DAYS);
System.out.println("Result in DAYS: " + result);    // Result in DAYS: 0
```

【操作示例 9】toString() 使用ISO-8601表示形式返回此瞬间的字符串表示形式，格式与DateTimeFormatter.ISO_INSTANT相同。

```java
// create a Instant object
Instant instant = Instant.parse("2018-10-28T19:34:50.63Z");

// print Instant using toString()
System.out.println(instant.toString());      // 2018-10-28T19:34:50.630Z

// addition of 84000 seconds to this instant
Instant returnedValue = instant.plusSeconds(84000);

// print result Instant using toString()
System.out.println(returnedValue.toString()) // 2018-10-29T18:54:50.630Z
```

【操作示例 10】query()方法，用于使用指定的查询作为参数来查询此瞬间。

```java
// create Instant object
Instant instant = Instant.parse("2018-12-31T10:15:30.00Z");

// apply query method of Instant class
String value = instant.query(TemporalQueries.precision()).toString();
System.out.println("Precision value for Instant is " + value); // Precision value for Instant is Nanos

ZoneOffset query = instant.query(TemporalQueries.offset());
System.out.println("Zone value for Instant is " + query);      // Zone value for Instant is null
```

【操作示例 11】所有示例汇总

```java
import java.time.Clock;
import java.time.Instant;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.temporal.ChronoUnit;
import java.util.Date;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 实例方式一：通过静态方法now(),获得UTC(本初子午线)的此刻瞬时时间的实例对象
         * 输出内容（默认时间比北京时间相差8小时）：2022-02-16T07:22:12.266171900Z（Z表示本初子午线）
         * 注:不建议使用 Instant 查看当前时间点, 因为 Instant是没有时区的。但是 Instant 加上时区后，可以转化为 ZonedDateTime
         */
        Instant instant1 = Instant.now();
        System.out.println(instant1); // 2022-02-16T07:22:12.266Z

        /**
         * 实例化方式二：通过给定毫秒数或秒数，获取Instant实例
         */
        System.out.println(Instant.ofEpochMilli(Clock.systemDefaultZone().millis())); // 2022-02-16T07:22:12.266Z
        System.out.println(Instant.ofEpochMilli(Clock.systemUTC().millis())); // 2022-02-16T07:22:12.266Z
        System.out.println(Instant.ofEpochMilli(new Date().getTime())); // 2022-02-16T07:22:12.266Z
        System.out.println(Instant.ofEpochMilli(System.currentTimeMillis())); // 2022-02-16T07:22:12.266Z
        System.out.println(Instant.ofEpochSecond(System.currentTimeMillis() / 1000)); // 2022-02-16T07:22:12Z

        /**
         * 实例化方式三：将字符串转换成Instant
         */
        System.out.println(Instant.parse("2022-02-16T07:22:12.266Z")); // 2022-02-16T07:22:12.266Z

        /**
         * long getEpochSecond():获取当前时间戳的秒数:
         * long toEpochMilli():获取当前时间戳的毫秒:
         * int getNano():获取当前时间点(抛开整数秒不算)的纳秒数
         * 如: 12345.12秒，抛开整数秒不算,则为0.12秒，那么instant.getNano()的结果为 0.12 * 1000_000_000 = 120_000_000
         */
        System.out.println("秒数 -> " + Instant.now().getEpochSecond()); // 秒数 -> 1644997084
        System.out.println("毫秒数 -> " + Instant.now().toEpochMilli()); // 毫秒数 -> 1644997084046
        System.out.println("纳秒数 -> " + Instant.now().getNano());      // 纳秒数 -> 46179600

        /**
         * Instant 与 时间偏移量 的相互转换, 注:从1970-01-01 00:00:00开始计算偏移
         */
        System.out.println(Instant.now()); // 2022-02-16T07:44:36.084408900Z
        // 对时间进行时区偏移修正，北京时间应+8，输出内容可以发现->(Z变为+8:00)
        System.out.println(Instant.now().atOffset(ZoneOffset.ofHours(8))); // 2022-02-16T15:44:36.084408900+08:00
        // 设置时区后，显示时间为北京时间了->(可以发现后面带上了时区)
        System.out.println(Instant.now().atZone(ZoneId.systemDefault())); // 2022-02-16T15:44:36.084408900+08:00[Asia/Shanghai]

        /**
         * Instant的时间加、减：
         * 由于北京时间比UTC时间晚8小时，所以我们需要得出北京的瞬时时间，需要加8小时
         */
        Instant instant3 = Instant.now().plus(8, ChronoUnit.HOURS);
        // 原(北京瞬时)instant -> 2022-02-16T15:22:12.266653300Z
        System.out.println("原(北京瞬时)instant -> " + instant3);
        Instant plusRes = instant3.plus(1, ChronoUnit.HOURS); // + 1 小时
        // 原(北京瞬时)instant + 1小时,结果是 -> 2022-02-16T16:22:12.266653300Z
        System.out.println("原(北京瞬时)instant + 1小时,结果是 -> " + plusRes);
        Instant minusRes = instant3.minus(2, ChronoUnit.HOURS); // - 2 小时
        // 原(北京瞬时)instant - 2小时,结果是 -> 2022-02-16T13:22:12.266653300Z
        System.out.println("原(北京瞬时)instant - 2小时,结果是 -> " + minusRes);

        /**
         * 判断两个Instant之间谁早谁晚
         */
        // 将Clock转换成Instant
        Instant instantOne = Instant.now(Clock.systemDefaultZone());
        // 对时间进行时区偏移修正，北京时间应+8
        Instant instantTwo = instantOne.plus(8, ChronoUnit.HOURS);
        boolean isAfterResult = instantOne.isAfter(instantTwo);
        // 瞬时时间点instantOne晚于instantTwo ? --- false
        System.out.println("瞬时时间点instantOne晚于instantTwo ? --- " + isAfterResult);
        // 瞬时时间点instantOne早于instantTwo ? --- true
        boolean isBeforeResult = instantOne.isBefore(instantTwo);
        System.out.println("瞬时时间点instantOne早于instantTwo ? --- " + isBeforeResult);
    }
}
```



### 5、Instant 注意事项

1、Instant.now() 获取的时间与北京时间相差8个时区，这是一个细节，要避坑。

- 看源码，用的是UTC时间。

  ```java
  public static Instant now() {     
      return Clock.systemUTC().instant();  
  }
  ```

- 解决方案：

  ```java
  Instant now = Instant.now().plusMillis(TimeUnit.HOURS.toMillis(8));
  System.out.println("now:" + now);
  ```

2、Instant.now() 与 Instant.now(Clock clock) 输出的结果一样，无论 clock 中设置什么时区。

- 先看案例，可以得出结论：三者输出都是 UTC 时区（其实之前说过：Instant 默认 是UTC 的格林威治时区 0 时区）

  ```java
  System.out.println(Instant.now());                           // 2023-05-15T06:30:12.422773900Z
  System.out.println(Instant.now(Clock.systemUTC()));          // 2023-05-15T06:30:12.449774500Z
  System.out.println(Instant.now(Clock.systemDefaultZone()));  // 2023-05-15T06:30:12.558374100Z
  ```

- 看源码，可以看出来最终都是调用 Instant.ofEpochSecond()，并没有设置相应的时区，只是把我们对应时区转换成 UTC 的时间戳。

  ```java
  public static Instant now() {
      return Clock.systemUTC().instant();
  }
  public static Instant now(Clock clock) {
      Objects.requireNonNull(clock, "clock");
      return clock.instant();
      // 最终调用: Instant.ofEpochSecond(localOffset, adjustment);
  }
  ```

- 解决方案：

  ```java
  // 方案一: 加上对应时区的时间差
  Instant now = Instant.now().plusMillis(TimeUnit.HOURS.toMillis(8));
  System.out.println("now:" + now);
  // 方案二: 使用 Instant + 时区 ZoneId 转换成有时区的时间 ZonedDateTime
  // 设置时区后，显示时间为北京时间了(可以发现后面带上了时区)
  ZonedDateTime zdt = Instant.now().atZone(ZoneId.systemDefault());
  System.out.println("now:" + zdt);
  ```



## java.time.LocalDate

LocalDate 用于表示 “本地日期” 无 “时间”，且不承载时区信息。可以对时间进行加减等操作并返回新副本，是不可变类且线程安全的。
默认格式：yyyy-MM-dd。它经常被用于展示`year-month-day、day-of-year、day-of-week、week-of-year`等格式的信息。

LocalDate 类是不可变类且线程安全的，它继承了 Object 类并实现了 ChronoLocalDate 接口，查看类的声明

```java
public final class LocalDate extends Object implements Temporal, TemporalAdjuster, ChronoLocalDate, Serializable {} 
```

LocalDate 类方法如下：

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 将指定的时态对象调整为与此对象具有相同的日期。               |
| LocalDateTime atStartOfDay()                                 | 将此日期与午夜时间相结合，以在此日期开始时创建LocalDateTime。 |
| ZonedDateTime atStartOfDay(ZoneId zone)                      | 根据时区中的规则，在最早的有效时间返回此日期的分区日期时间。 |
| LocalDateTime atTime(int hour, int minute)                   | 将此日期与创建LocalDateTime的时间相结合。                    |
| LocalDateTime atTime(int hour, int minute, int second)       | 将此日期与创建LocalDateTime的时间相结合。                    |
| LocalDateTime atTime(int hour, int minute, int second, int nanoOfSecond) | 将此日期与创建LocalDateTime的时间相结合。                    |
| LocalDateTime atTime(LocalTime time)                         | 将此日期与创建LocalDateTime的时间相结合。                    |
| OffsetDateTime atTime(OffsetTime time)                       | 将此日期与偏移时间组合以创建OffsetDateTime。                 |
| int compareTo(ChronoLocalDate other)                         | 将此日期与另一个日期进行比较。                               |
| boolean equals(Object obj)                                   | 检查此日期是否等于另一个日期。                               |
| String format(DateTimeFormatter formatter)                   | 使用指定的格式化程序格式化此日期。                           |
| static LocalDate from(TemporalAccessor temporal)             | 从时态对象获取LocalDate的实例。                              |
| int get(TemporalField field)                                 | 从int获取指定字段的值作为int。                               |
| IsoChronology getChronology()                                | 获取此日期的年表，即ISO日历系统。                            |
| int getDayOfMonth()                                          | 获取日期字段。                                               |
| DayOfWeek getDayOfWeek()                                     | 获取星期几字段，即枚举DayOfWeek。                            |
| int getDayOfYear()                                           | 获取日期字段。                                               |
| Era getEra()                                                 | 获取此日期适用的时代。                                       |
| long getLong(TemporalField field)                            | 从此日期获取指定字段的值为long。                             |
| Month getMonth()                                             | 使用Month枚举获取月份字段。                                  |
| int getMonthValue()                                          | 获取1到12之间的月份字段。                                    |
| int getYear()                                                | 获取年份字段。                                               |
| int hashCode()                                               | 此日期的哈希码。                                             |
| boolean isAfter(ChronoLocalDate other)                       | 检查此日期是否在指定日期之后。                               |
| boolean isBefore(ChronoLocalDate other)                      | 检查此日期是否在指定日期之前。                               |
| boolean isEqual(ChronoLocalDate other)                       | 检查此日期是否等于指定日期。                                 |
| boolean isLeapYear()                                         | 根据ISO符号日历系统规则，检查年份是否为闰年。                |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                                     |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                                   |
| int lengthOfMonth()                                          | 返回此日期表示的月份长度。                                   |
| int lengthOfYear()                                           | 返回此日期表示的年份长度。                                   |
| LocalDate minus(long amountToSubtract, TemporalUnit unit)    | 返回此日期的副本，并减去指定的数量。                         |
| LocalDate minus(TemporalAmount amountToSubtract)             | 返回此日期的副本，并减去指定的数量。                         |
| LocalDate minusDays(long daysToSubtract)                     | 返回此LocalDate的副本，并减去指定的天数。                    |
| LocalDate minusMonths(long monthsToSubtract)                 | 返回此LocalDate的副本，并减去指定的月数。                    |
| LocalDate minusWeeks(long weeksToSubtract)                   | 返回此LocalDate的副本，并减去指定的周数。                    |
| LocalDate minusYears(long yearsToSubtract)                   | 返回此LocalDate的副本，并减去指定的年数。                    |
| static LocalDate now()                                       | 在默认时区中从系统时钟获取当前日期。                         |
| static LocalDate now(Clock clock)                            | 从指定的时钟获取当前日期。                                   |
| static LocalDate now(ZoneId zone)                            | 从指定时区的系统时钟获取当前日期。                           |
| static LocalDate of(int year, int month, int dayOfMonth)     | 从一年，一月和一天获得LocalDate的实例。                      |
| static LocalDate of(int year, Month month, int dayOfMonth)   | 从一年，一月和一天获得LocalDate的实例。                      |
| static LocalDate ofEpochDay(long epochDay)()                 | 从纪元日计数中获取LocalDate的实例。                          |
| static LocalDate ofYearDay(int year, int dayOfYear)          | 从一年和一年获得LocalDate的实例。                            |
| static LocalDate parse(CharSequence text)                    | 从文本字符串(如2007-12-03)获取LocalDate的实例。              |
| static LocalDate parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取LocalDate的实例。        |
| LocalDate plus(long amountToAdd, TemporalUnit unit)          | 返回此日期的副本，并添加指定的数量。                         |
| LocalDate plus(TemporalAmount amountToAdd)                   | 返回此日期的副本，并添加指定的数量。                         |
| LocalDate plusDays(long daysToAdd)                           | 返回此LocalDate的副本，并添加指定的天数。                    |
| LocalDate plusMonths(long monthsToAdd)                       | 返回此LocalDate的副本，并添加指定的月数。                    |
| LocalDate plusWeeks(long weeksToAdd)                         | 返回此LocalDate的副本，并添加指定的周数。                    |
| LocalDate plusYears(long yearsToAdd)                         | 返回此LocalDate的副本，其中添加了指定的年数。                |
| R query(TemporalQuery query)                                 | 使用指定的查询查询此日期。                                   |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                                   |
| long toEpochDay()                                            | 将此日期转换为大纪元日。                                     |
| String toString()                                            | 将此日期输出为字符串，例如:2007-12-03。                      |
| Period until(ChronoLocalDate endDateExclusive)               | 计算此日期与另一个日期之间的期间作为期间。                   |
| long until(Temporal endExclusive, TemporalUnit unit)         | 根据指定的单位计算到另一个日期的时间量。                     |
| LocalDate with(TemporalAdjuster adjuster)                    | 返回此日期的调整副本。                                       |
| LocalDate with(TemporalField field, long newValue)           | 返回此日期的副本，并将指定的字段设置为新值。                 |
| LocalDate withDayOfMonth(int dayOfMonth)                     | 返回此LocalDate的副本，其中包含日期更改。                    |
| LocalDate withDayOfYear(int dayOfYear)                       | 返回此LocalDate的副本，其中包含日期更改。                    |
| LocalDate withMonth(int month)                               | 返回此LocalDate的副本，其中包含已更改的月份。                |
| LocalDate withYear(int year)                                 | 返回此LocalDate的副本，并更改年份。                          |

【操作示例】

```java
import java.time.LocalDate;

public class JavaDateTest {
    public static void main(String[] args) {

        /**
         * 常用的两种实例化方式：
         *   1.通过静态方法, 获取系统的当前时间：LocalDate.now()
         *   2.通过静态方法, 获取自定义指定时间：LocalDate.of(2022, 02, 28)
         */
        LocalDate today = LocalDate.now(); // 获取当前日期(年/月/日) 2020-6-14 周天
        LocalDate of = LocalDate.of(2022, 02, 28);

        /**
         * 常用的getXxx()系列操作，获得日期：
         *   int getYear() 获取当前⽇期的年份
         *   Month getMonth() 获取当前⽇期的⽉份对象(返回一个 Month 枚举值)
         *   int getMonthValue() 获取当前⽇期是第⼏⽉(1-12)
         *   int getDayOfMonth() 表示该对象表示的⽇期是这个⽉第⼏天(1-31)
         *   DayOfWeek getDayOfWeek() 表示该对象表示的⽇期是星期⼏(返回一个 DayOfWeek枚举值)
         *   int getDayOfYear() 表示该对象表示的⽇期是今年第⼏天(1-366)
         */
        System.out.println("今天的⽇期：" + today); // 今天的⽇期：2020-06-14
        System.out.println("指定的⽇期：" + of); // 指定的⽇期：2022-02-28
        System.out.println("现在是哪年:" + today.getYear()); // 现在是哪年:2020
        System.out.println("现在是哪⽉(英文):" + today.getMonth()); // 现在是哪⽉(英文):JUNE
        System.out.println("现在是哪⽉(数字):" + today.getMonthValue()); // 现在是哪⽉(数字):6
        System.out.println("现在是⼏号:" + today.getDayOfMonth()); // 现在是⼏号:14
        System.out.println("现在是周⼏:" + today.getDayOfWeek()); // 现在是周⼏:SUNDAY
        System.out.println("现在是今年的第几天:" + today.getDayOfYear()); // 现在是今年的第几天:166

        /**
         * 常用的setXxx()系列操作，设置日期：
         *   LocalDate withYear(int year) 修改当前对象的年份
         *   LocalDate withMonth(int month) 修改当前对象的⽉份
         *   LocalDate withDayOfMonth(int dayOfMonth) 修改当前对象在当⽉的⽇期
         *   LocalDate withDayOfYear(int dayOfYear) 修改当前对象在当年的⽇期
         */
        LocalDate withLocalDate = LocalDate.of(2022, 01, 01);
        System.out.println("常用的setXxx()系列操作:" + withLocalDate); // 此刻时间:2022-01-01（用作对比参考）
        System.out.println(withLocalDate.withYear(2030));// 2030-01-01
        System.out.println(withLocalDate.withMonth(2)); // 2022-02-01
        System.out.println(withLocalDate.withDayOfMonth(8)); // 2022-01-08
        System.out.println(withLocalDate.withDayOfYear(10)); // 2022-01-10

        /**
         * 常用的plusXxx()系列操作，增加时间的方法：
         *   LocalDate plusYears(long yearsToAdd) 增加指定年份数
         *   LocalDate plusMonths(long monthsToAdd) 增加指定⽉份数
         *   LocalDate plusDays(long daysToAdd) 增加指定天数
         *   LocalDate plusWeeks(long weeksToAdd) 增加指定周数
         */
        LocalDate plusLocalDate = LocalDate.of(2022, 01, 01);
        System.out.println("常用的plusXxx()系列操作:" + plusLocalDate); // 此刻时间:2022-01-01（用作对比参考）
        System.out.println(plusLocalDate.plusYears(1)); // 2023-01-01
        System.out.println(plusLocalDate.plusMonths(1)); // 2022-02-01
        System.out.println(plusLocalDate.plusWeeks(1)); // 2022-01-08
        System.out.println(plusLocalDate.plusDays(6)); // 2022-01-07

        /**
         * 常用的minusXxx()系列操作，减少时间的方法:
         *   LocalDate minusYears(long yearsToSubtract) 减去指定年数
         *   LocalDate minusMonths(long monthsToSubtract) 减去注定⽉数
         *   LocalDate minusDays(long daysToSubtract) 减去指定天数
         *   LocalDate minusWeeks(long weeksToSubtract) 减去指定周数
         */
        LocalDate minusLocalDate = LocalDate.of(2022, 01, 01);
        System.out.println("常用的minusXxx()系列操作:" + minusLocalDate); // 此刻时间:2022-01-01（用作对比参考）
        System.out.println(minusLocalDate.minusYears(5));   // 2017-01-01
        System.out.println(minusLocalDate.minusMonths(60)); // 2017-01-01
        System.out.println(minusLocalDate.minusWeeks(260)); // 2017-01-07
        System.out.println(minusLocalDate.minusDays(1826)); // 2017-01-01

        /**
         * 常用日期对比方法：
         *   int compareTo(ChronoLocalDate other) ⽐较当前对象和other对象在时间上的⼤⼩,返回值为正,表示当前对象时间较晚
         *   boolean isBefore(ChronoLocalDate other) ⽐较当前对象⽇期是否在other对象⽇期之前
         *   boolean isAfter(ChronoLocalDate other) ⽐较当前对象⽇期是否在other对象⽇期之后
         *   boolean isEqual(ChronoLocalDate other) ⽐较两个⽇期对象是否相等
         *   boolean isLeapYear() 判断是否是闰年（注意是LocalDate类 和 LocalDateTime类特有的方法）
         */
        LocalDate localDateOne = LocalDate.of(2022, 01, 01);
        LocalDate localDateTwo = LocalDate.of(2000, 01, 01);
        System.out.println("compareTo: " + localDateOne.compareTo(localDateTwo)); // compareTo: 22
        System.out.println("isBefore: " + localDateOne.isBefore(localDateTwo)); // isBefore: false
        System.out.println("isAfter: " + localDateOne.isAfter(localDateTwo)); // isAfter: true
        System.out.println("isEqual: " + localDateOne.isEqual(localDateTwo)); // isEqual: false
        System.out.println("isLeapYear: " + localDateTwo.isLeapYear()); // isLeapYear: true
    }
}
```



## java.time.LocalTime

LocalTime 用于表示 “本地时间” 无 “日期”。且不承载时区信息。可以对时间进行加减等操作并返回新副本，是线程安全并且不可变类。
默认格式：HH:mm:ss.fff，它经常被用于展示`hour-minute-second`格式的信息。

LocalTime 类是一个不变且线程安全的类，它继承了Object类并实现Comparable接口，LocalTime类的声明：

```java
public final class LocalTime extends Object 
    implements Temporal, TemporalAdjuster, Comparable<LocalTime>, Serializable {} 
```

LocalTime 方法如下：

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定Temporal对象以使其具有与此对象相同的日期和时间。     |
| LocalDateTime atDate(LocalDate date)                         | 将此时间与日期相结合以创建LocalDateTime。                    |
| OffsetDateTime atOffset(ZoneOffset offset)                   | 将此时间与偏移时间相结合以创建OffsetDateTime。               |
| int compareTo(LocalTime other)                               | 这个时间与另一个时间比较。                                   |
| boolean equals(Object obj)                                   | 检查此时间是否等于另一个时间。                               |
| String format(DateTimeFormatter formatter)                   | 这次使用指定的格式化程序格式化。                             |
| static LocalTime from(TemporalAccessor temporal)             | 从temporal对象获取LocalTime的实例。                          |
| int get(TemporalField field)                                 | 从此时间获取指定字段的值作为int值。                          |
| int getHour()                                                | 获取当日时间字段。                                           |
| long getLong(TemporalField field)                            | 从此时间获取指定字段的long值。                               |
| int getMinute()                                              | 获取分钟字段的值。                                           |
| int getNano()                                                | 获取纳秒字段的值。                                           |
| int getSecond()                                              | 获取秒钟字段的值。                                           |
| int hashCode()                                               | 时间的哈希码                                                 |
| boolean isAfter(LocalTime other)                             | 检查此时间是否在指定时间之后。                               |
| boolean isBefore(LocalTime other)                            | 检查此时间是否在指定时间之前。                               |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                                     |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                                   |
| LocalTime minus(long amountToSubtract, TemporalUnit unit)    | 返回此时间的副本，并减去指定的数量。                         |
| LocalTime minus(TemporalAmount amountToSubtract)             | 返回此时间的副本，并减去指定的数量。                         |
| LocalTime minusHours(long hoursToSubtract)                   | 返回此LocalTime的副本，并减去指定的小时数。                  |
| LocalTime minusMinutes(long minutesToSubtract)               | 返回此LocalTime的副本，并减去指定的分钟数。                  |
| LocalTime minusNanos(long nanos)                             | 返回此LocalTime的副本，并减去指定的纳秒数。                  |
| LocalTime minusSeconds(long seconds)                         | 返回此LocalTime的副本，并减去指定的秒数。                    |
| static LocalTime now()                                       | 在默认时区中从系统时钟获取当前时间。                         |
| static LocalTime now(Clock clock)                            | 从指定的时钟获得当前时间。                                   |
| static LocalTime now(ZoneId zone)                            | 从指定时区的系统时钟获取当前时间。                           |
| static LocalTime of(int hour, int minute)                    | 从指定小时分钟获得LocalTime的实例。                          |
| static LocalTime of(int hour, int minute, int second)        | 从指定小时，分钟和秒钟获取LocalTime的实例。                  |
| static LocalTime of(int hour, int minute, int second, int nanoOfSecond) | 从指定小时，分钟，秒和纳秒获得LocalTime的实例。              |
| static LocalTime ofNanoOfDay(long nanoOfDay)                 | 从一天的纳米值获得LocalTime的实例。                          |
| static LocalTime ofSecondOfDay(long secondOfDay)             | 从一天的秒值获取LocalTime的实例。                            |
| static LocalTime parse(CharSequence text)                    | 从文本字符串中获取LocalTime的实例，例如2007-12-03T10:15:30。 |
| static LocalTime parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取LocalTime的实例。        |
| LocalTime plus(long amountToAdd, TemporalUnit unit)          | 返回此时间的副本，并添加指定的数量。                         |
| LocalTime plus(TemporalAmount amountToAdd)                   | 返回此时间的副本，并添加指定的数量。                         |
| LocalTime plusHours(long hoursToAdd)                         | 返回此LocalTime的副本，并添加指定的小时数。                  |
| LocalTime plusMinutes(long minutesToAdd)                     | 返回此LocalTime的副本，并添加指定的分钟数。                  |
| LocalTime plusNanos(long nanos)                              | 返回此LocalTime的副本，并添加指定的纳秒数。                  |
| LocalTime plusSeconds(long seconds)                          | 返回此LocalTime的副本，并添加指定的秒数。                    |
| R query(TemporalQuery query)                                 | 使用指定的查询进行查询的时间。                               |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                                   |
| long toNanoOfDay()                                           | 提取时间为一天的纳秒数，从0到24 * 60 * 60 * 1,000,000,000 - 1。 |
| int toSecondOfDay()                                          | 将时间提取为一天中的秒数，从0到24 * 60 * 60 - 1。            |
| String toString()                                            | 将此日期输出为String，例如10:15。                            |
| LocalTime truncatedTo(TemporalUnit unit)                     | 返回此LocalTime的副本，并截断时间。                          |
| long until(Temporal endExclusive, TemporalUnit unit)         | 根据指定的单位计算到另一个时间的时间量。                     |
| LocalTime with(TemporalAdjuster adjuster)                    | 返回此时的调整副本。                                         |
| LocalTime with(TemporalField field, long newValue)           | 返回此时间的副本，并将指定字段设置为新值。                   |
| LocalTime withHour(int hour)                                 | 返回此LocalTime的副本，并更改日期。                          |
| LocalTime withMinute(int minute)                             | 返回此LocalTime的副本，并更改了分钟。                        |
| LocalTime withNano(int nanoOfSecond)                         | 返回此LocalTime的副本，并更改了纳秒。                        |
| LocalTime withSecond(int second)                             | 返回此LocalTime的副本，并更改秒钟。                          |

【操作示例】

```java
import java.time.LocalTime;

public class JavaDateTest {
    public static void main(String[] args) {

        /**
         * 常用的两种实例化方式：
         *   1.通过静态方法, 获取系统的当前时间：LocalTime.now()
         *   2.通过静态方法, 获取自定义指定时间：LocalTime.of(21, 30, 59, 11001);
         */
        LocalTime today = LocalTime.now();
        LocalTime of = LocalTime.of(21, 30, 59, 11001);
        System.out.println("指定的时间：" + of); // 指定的时间：18:24:31.761102500

        /**
         * 常用的getXxx()系列操作，获得日期：
         *   int getHour() 获取当前时间小时数
         *   int getMinute() 获取当前时间分钟数
         *   int getSecond() 获取当前时间秒值
         *   int getNano() 把获取到的当前时间的秒数换算成纳秒
         */
        System.out.println("今天的时间：" + today); // 今天的时间：18:24:31.761102500
        System.out.println("现在是几时:" + today.getHour()); // 现在是几时:18
        System.out.println("现在是几分:" + today.getMinute()); // 现在是几分:24
        System.out.println("现在是几秒:" + today.getSecond()); // 现在是几秒:31
        System.out.println("现在是⼏纳秒:" + today.getNano()); // 现在是⼏纳秒:761102500

        /**
         * 常用的setXxx()系列操作，设置日期：
         *   LocalTime withHour(int hour) 修改当前对象的小时数
         *   LocalTime withMinute(int minute) 修改当前对象的分钟数
         *   LocalTime withSecond(int second) 修改当前对象在当⽉的秒数
         *   LocalTime withNano(int nanoOfSecond) 修改当前对象在当年的纳秒数
         */
        LocalTime withLocalTime = LocalTime.of(13, 8, 20, 123456789);
        System.out.println("常用的setXxx()系列操作:" + withLocalTime); // 此刻时间:13:08:20.123456789（用作对比参考）
        System.out.println(withLocalTime.withHour(5)); // 05:08:20.123456789
        System.out.println(withLocalTime.withMinute(10)); // 13:10:20.123456789
        System.out.println(withLocalTime.withSecond(8)); // 13:08:08.123456789
        System.out.println(withLocalTime.withNano(100000001)); // 13:08:20.100000001

        /**
         * 常用的plusXxx()系列操作，增加时间的方法：
         *   LocalTime plusHours(long hoursToAdd) 增加指定小时
         *   LocalTime plusMinutes(long minutesToAdd) 增加指定分钟
         *   LocalTime plusSeconds(long secondstoAdd) 增加指定秒
         *   LocalTime plusNanos(long nanosToAdd) 增加指定纳秒
         */
        LocalTime plusLocalTime = LocalTime.of(13, 8, 20, 123456789);
        System.out.println("常用的plusXxx()系列操作:" + plusLocalTime); // 此刻时间:13:08:20.123456789（用作对比参考）
        System.out.println(plusLocalTime.plusHours(1)); // 14:08:20.123456789
        System.out.println(plusLocalTime.plusMinutes(1)); // 13:09:20.123456789
        System.out.println(plusLocalTime.plusSeconds(1)); // 13:08:21.123456789
        System.out.println(plusLocalTime.plusNanos(6)); // 13:08:20.123456795

        /**
         * 常用的minusXxx()系列操作，减少时间的方法:
         *   LocalTime minusHours(long hoursToSubtract) 减去指定年数
         *   LocalTime minusMinutes(long minutesToSubtract) 减去注定⽉数
         *   LocalTime minusSeconds(long secondsToSubtract) 减去指定天数
         *   LocalTime minusNanos(long nanosToSubtract) 减去指定周数
         */
        LocalTime minusLocalTime = LocalTime.of(13, 8, 20, 123456789);
        System.out.println("常用的minusXxx()系列操作:" + minusLocalTime); // 此刻时间:13:08:20.123456789（用作对比参考）
        System.out.println(minusLocalTime.minusHours(1)); // 12:08:20.123456789
        System.out.println(minusLocalTime.minusMinutes(60)); // 12:08:20.123456789
        System.out.println(minusLocalTime.minusSeconds(3600)); // 12:08:20.123456789
        System.out.println(minusLocalTime.minusNanos(9)); // 13:08:20.123456780

        /**
         * 常用日期对比方法：
         *   int compareTo(LocalTime other) ⽐较当前对象和other对象在时间上的⼤⼩,返回值如果为正,则当前对象时间较晚
         *   boolean isBefore(LocalTime other) ⽐较当前对象时间是否在other对象时间之前
         *   boolean isAfter(LocalTime other) ⽐较当前对象时间是否在other对象时间之后
         */
        LocalTime localTimeOne = LocalTime.of(13, 8, 20, 123456789);
        LocalTime localTimeTwo = LocalTime.of(10, 8, 20, 123456789);
        System.out.println("compareTo: " + localTimeOne.compareTo(localTimeTwo)); // compareTo: 1
        System.out.println("isBefore: " + localTimeOne.isBefore(localTimeTwo)); // isBefore: false
        System.out.println("isAfter: " + localTimeOne.isAfter(localTimeTwo)); // isAfter: true
    }
}
```



## java.time.LocalDateTime

LocalDateTime 用于表示 “本地日期与时间”，且不承载时区信息。可以对时间进行加减等操作并返回副本，也是线程安全并且不可变类。**这个是最常用的类。**默认的日期格式，yyyy-MM-dd HH:mm:ss.fff，它经常被用于展示`hour-minute-second`格式的信息。

LocalDateTime 是线程安全并且不可变类，它继承了Objct类并实现了ChronoLocalDateTime接口。LocalDateTime 类声明：

```java
public final class LocalDateTime extends Object 
    implements Temporal, TemporalAdjuster, ChronoLocalDateTime<LocalDate>, Serializable {}
```

LocalDateTime 类方法如下：

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定的时态对象以使其具有与此对象相同的日期和时间。       |
| OffsetDateTime atOffset(ZoneOffset offset)                   | 将此日期时间与偏移时间组合以创建OffsetDateTime。             |
| ZonedDateTime atZone(ZoneId zone)                            | 将此日期时间与时区组合以创建ZonedDateTime。                  |
| int compareTo(ChronoLocalDateTime other)                     | 将此日期时间与另一个日期时间进行比较。                       |
| boolean equals(Object obj)                                   | 检查此日期时间是否等于另一个日期时间。                       |
| String format(DateTimeFormatter formatter)                   | 使用指定的格式化程序格式化此日期时间。                       |
| static LocalDateTime from(TemporalAccessor temporal)         | 从时态对象获取LocalDateTime的实例。                          |
| int get(TemporalField field)                                 | 从此日期时间获取指定字段的值作为int类型值。                  |
| int getDayOfMonth()                                          | 获取日期字段。                                               |
| DayOfWeek getDayOfWeek()                                     | 获取星期几字段，即枚举DayOfWeek。                            |
| int getDayOfYear()                                           | 获取日期字段。                                               |
| int getHour()                                                | 获取当日时间字段。                                           |
| long getLong(TemporalField field)                            | 从此日期时间获取指定字段的值为long值。                       |
| Month getMinute()                                            | 获取分钟字段。                                               |
| Month getMonth()                                             | 使用Month枚举获取月份字段。                                  |
| int getMonthValue()                                          | 获取1到12之间的月份字段。                                    |
| int getNano()                                                | 获取纳秒级字段。                                             |
| int getSecond()                                              | 获取秒钟字段。                                               |
| int getYear()                                                | 获取年份字段。                                               |
| int hashCode()                                               | 此日期时间的哈希码。                                         |
| boolean isAfter(ChronoLocalDateTime other)                   | 检查此日期时间是否在指定的日期时间之后。                     |
| boolean isBefore(ChronoLocalDateTime other)                  | 检查此日期时间是否在指定的日期时间之前。                     |
| boolean isEqual(ChronoLocalDateTime other)                   | 检查此日期时间是否等于指定的日期时间。                       |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                                     |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                                   |
| LocalDateTime minus(long amountToSubtract, TemporalUnit unit) | 返回此日期时间的副本，并减去指定的数量。                     |
| LocalDateTime minus(TemporalAmount amountToSubtract)         | 返回此日期时间的副本，并减去指定的数量。                     |
| LocalDateTime minusDays(long daysToSubtract)                 | 返回此LocalDateTime的副本，并减去指定的天数。                |
| LocalDateTime minusHours(long hoursToSubtract)               | 返回此LocalDateTime的副本，并减去指定的小时数。              |
| LocalDateTime minusMinutes(long minutesToSubtract)           | 返回此LocalDateTime的副本，并减去指定的分钟数。              |
| LocalDateTime minusMonths(long monthsToSubtract)             | 返回此LocalDateTime的副本，并减去指定的月数。                |
| LocalDateTime minusNanos(long nanos)                         | 返回此LocalDateTime的副本，减去指定的纳秒数。                |
| LocalDateTime minusSeconds(long seconds)                     | 返回此LocalDateTime的副本，并减去指定的秒数。                |
| LocalDateTime minusWeeks(long weeksToSubtract)               | 返回此LocalDateTime的副本，并减去指定的周数。                |
| LocalDateTime minusYears(long yearsToSubtract)               | 返回此LocalDateTime的副本，并减去指定的年数。                |
| static LocalDateTime now()                                   | 从默认时区中的系统时钟获取当前日期时间。                     |
| static LocalDateTime now(Clock clock)                        | 从指定的时钟获得当前日期时间。                               |
| static LocalDateTime now(ZoneId zone)                        | 从指定时区的系统时钟获取当前日期时间。                       |
| static LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute) | 从年，月，日，小时和分钟获得LocalDateTime的实例，将秒钟和纳秒设置为零。 |
| static LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second) | 从年，月，日，小时，分钟和秒获得LocalDateTime的实例，将纳秒设置为零。 |
| static LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond) | 从年，月，日，小时，分钟，秒和纳秒获得LocalDateTime的实例。  |
| static LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute, int second) | 从年，月，日，小时，分钟和秒获得LocalDateTime的实例，将纳秒设置为零。 |
| static LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond) | 从年，月，日，小时，分钟，秒和纳秒获得LocalDateTime的实例。  |
| static LocalDateTime of(LocalDate date, LocalTime time)      | 从日期和时间获取LocalDateTime的实例。                        |
| static LocalDateTime ofEpochSecond(long epochSecond, int nanoOfSecond, ZoneOffset offset) | 从1970-01-01T00:00:00Z的纪元获得LocalDateTime的实例。        |
| static LocalDateTime ofInstant(Instant instant, ZoneId zone) | 从Instant和区域ID获取LocalDateTime的实例。                   |
| static LocalDateTime parse(CharSequence text)                | 从文本字符串中获取LocalDateTime的实例，例如2007-12-03 T10:15:30。 |
| static LocalDateTime parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取LocalDateTime的实例。    |
| LocalDateTime plus(long amountToAdd, TemporalUnit unit)      | 返回此日期时间的副本，并添加指定的数量。                     |
| LocalDateTime plus(TemporalAmount amountToAdd)               | 返回此日期时间的副本，并添加指定的数量。                     |
| LocalDateTime plusDays(long daysToAdd)                       | 返回此LocalDateTime的副本，并添加指定的天数。                |
| LocalDateTime plusHours(long hoursToAdd)                     | 返回此LocalDateTime的副本，并添加指定的小时数。              |
| LocalDateTime plusMinutes(long minutesToAdd)                 | 返回此LocalDateTime的副本，并添加指定的分钟数。              |
| LocalDateTime plusMonths(long monthsToAdd)                   | 返回此LocalDateTime的副本，并添加指定的月份数。              |
| LocalDateTime plusNanos(long nanos)                          | 返回此LocalDateTime的副本，其中添加了指定的纳秒数。          |
| LocalDateTime plusSeconds(long seconds)                      | 返回此LocalDateTime的副本，并添加指定的秒数。                |
| LocalDateTime plusWeeks(long weeksToAdd)                     | 返回此LocalDateTime的副本，并添加指定的周数。                |
| LocalDateTime plusYears(long yearsToAdd)                     | 返回此LocalDateTime的副本，其中添加了指定的年数。            |
| R query(TemporalQuery query)                                 | 使用指定的查询查询此日期时间。                               |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                                   |
| LocalDate toLocalDate()                                      | 获取此日期时间的LocalDate部分。                              |
| LocalTime toLocalTime()                                      | 获取此日期时间的LocalTime部分。                              |
| String toString()                                            | 将此日期输出为字符串，例如2007-12-03T10:15:30。              |
| LocalDateTime truncatedTo(TemporalUnit unit)                 | 返回此LocalDateTime的副本，并截断时间。                      |
| long until(Temporal endExclusive, TemporalUnit unit)         | 根据指定的单位计算到另一个日期时间的时间量。                 |
| LocalDateTime with(TemporalAdjuster adjuster)                | 返回此日期时间的调整副本。                                   |
| LocalDateTime with(TemporalField field, long newValue)       | 返回此日期时间的副本，并将指定字段设置为新值。               |
| LocalDateTime withDayOfMonth(int dayOfMonth)                 | 返回此LocalDateTime的副本，其中包含每日更改的日期。          |
| LocalDateTime withDayOfYear(int dayOfYear)                   | 返回此LocalDateTime的副本，其中包含日期更改。                |
| LocalDateTime withHour(int hour)                             | 返回此LocalDateTime的副本，并更改日期。                      |
| LocalDateTime withMinute(int minute)                         | 返回此LocalDateTime的副本，并更改了分钟。                    |
| LocalDateTime withMonth(int month)                           | 返回此LocalDateTime的副本，其中包含已更改的年份。            |
| LocalDateTime withNano(int nanoOfSecond)                     | 返回此LocalDateTime的副本，并更改了纳秒。                    |
| LocalDateTime withSecond(int second)                         | 返回此LocalDateTime的副本，并更改秒钟。                      |
| LocalDateTime withYear(int year)                             | 返回此LocalDateTime的副本，其中年份已更改。                  |

【操作示例】

```java
import java.time.*;
import java.time.temporal.ChronoField;
import java.time.temporal.ChronoUnit;

public class JavaDateTest {
    public static void main(String[] args) {

        /**
         * 常用的两种实例化方式：
         *   1.通过静态方法, 获取系统的当前时间：LocalDateTime.now()
         *   2.通过静态方法, 获取自定义指定时间：
         *     1.LocalDateTime.of(LocalDate.now(), LocalTime.now())
         *     2.LocalDateTime.of(2020,02,8,21, 30, 59, 11001)
         *   3.通过LocalDateTime转换LocalDate、LocalTime
         */
        LocalDateTime today = LocalDateTime.now();
        LocalDateTime of1 = LocalDateTime.of(LocalDate.now(), LocalTime.now());
        LocalDateTime of2 = LocalDateTime.of(2020,02,8,21, 30, 59, 11001);

        /**
         * 转换的方法:
         *   LocalDate toLocalDate()：将LocalDateTime转换为相应的LocalDate对象
         *   LocalTime toLocalTime()：将LocalDateTime转换为相应的LocalTime对象
         */
        LocalDate localDate = today.toLocalDate();
        LocalTime localTime = today.toLocalTime();

        /**
         * 常用的getXxx()系列操作，获得日期：
         *   int getYear() 获取当前日期的年份
         *   Month getMonth() 获取当前日期的月份对象(返回一个Month枚举值)
         *   int getMonthValue() 获取当前日期是第几月(1-12)
         *   int getDayOfMonth() 表示该对象表示的⽇期是这个⽉第⼏天(1-31)
         *   DayOfWeek getDayOfWeek() 表示该对象表示的日期是星期几(返回一个DayOfWeek枚举值)
         *   int getDayOfYear() 表示该对象表示的日期是今年第几天(1-366)
         *   int getHour() 获取当前时间小时数
         *   int getMinute() 获取当前时间分钟数
         *   int getSecond() 获取当前时间秒值
         *   int getNano() 把获取到的当前时间的秒数换算成纳秒
         *   int get(TemporalField field) 获取指定字段的日期时间，通过ChronoField枚举类
         */
        System.out.println("今天的时间:" + today); // 今天的时间:2022-02-17T19:13:38.682102600
        System.out.println("现在是哪年:" + today.getYear()); // 现在是哪年:2022
        System.out.println("现在是哪⽉(英文):" + today.getMonth()); // 现在是哪⽉(英文):FEBRUARY
        System.out.println("现在是哪⽉(数字):" + today.getMonthValue()); // 现在是哪⽉(数字):2
        System.out.println("现在是⼏号:" + today.getDayOfMonth()); // 现在是⼏号:17
        System.out.println("现在是周⼏:" + today.getDayOfWeek()); // 现在是周⼏:THURSDAY
        System.out.println("现在是该年的第几天:" + today.getDayOfYear()); // 现在是该年的第几天:48
        System.out.println("现在是几时:" + today.getHour()); // 现在是几时:19
        System.out.println("现在是几分:" + today.getMinute()); // 现在是几分:13
        System.out.println("现在是几秒:" + today.getSecond()); // 现在是几秒:38
        System.out.println("现在是⼏纳秒:" + today.getNano()); // 现在是⼏纳秒:682102600
        System.out.println("现在是⼏号:" + today.get(ChronoField.DAY_OF_MONTH)); // 现在是⼏号:19

        /**
         * 常用的setXxx()系列操作，设置日期：
         *   LocalDateTime withYear(int year) 指定对象的年份
         *   LocalDateTime withMonth(int month) 指定对象的月份
         *   LocalDateTime withDayOfMonth(int dayOfMonth) 指定对象在当月的日期
         *   LocalDateTime withDayOfYear(int dayOfYear) 指定对象在当年的日期
         *   LocalDateTime withHour(int hour) 指定对象的小时数
         *   LocalDateTime withMinute(int minute) 指定对象的分钟数
         *   LocalDateTime withSecond(int second) 指定对象在当⽉的秒数
         *   LocalDateTime withNano(int nanoOfSecond) 指定对象在当年的纳秒数
         *   LocalDateTime with(TemporalField field, long newValue) 指定对象的日期时间，通过ChronoField枚举类
         */
        LocalDateTime withLocalDateTime = LocalDateTime.of(2020,02,8,21, 30, 59, 123456789);
        System.out.println("常用的setXxx()系列操作:" + withLocalDateTime); // 常用的setXxx()系列操作:2020-02-08T21:30:59.123456789
        System.out.println(withLocalDateTime.withYear(2030));// 2030-02-08T21:30:59.123456789
        System.out.println(withLocalDateTime.withMonth(2)); // 2020-02-08T21:30:59.123456789
        System.out.println(withLocalDateTime.withDayOfMonth(8)); // 2020-02-08T21:30:59.123456789
        System.out.println(withLocalDateTime.withDayOfYear(10)); // 2020-01-10T21:30:59.123456789
        System.out.println(withLocalDateTime.withHour(5)); // 2020-02-08T05:30:59.123456789
        System.out.println(withLocalDateTime.withMinute(10)); // 2020-02-08T21:10:59.123456789
        System.out.println(withLocalDateTime.withSecond(8)); // 2020-02-08T21:30:08.123456789
        System.out.println(withLocalDateTime.withNano(100000001)); // 2020-02-08T21:30:59.100000001
        System.out.println(withLocalDateTime.with(ChronoField.DAY_OF_MONTH, 1)); // 2020-02-01T21:30:59.123456789

        /**
         * 常用的plusXxx()系列操作，增加时间的方法：
         *   LocalDateTime plusYears(long years) 增加指定年份数
         *   LocalDateTime plusMonths(long months) 增加指定⽉份数
         *   LocalDateTime plusDays(long days) 增加指定天数
         *   LocalDateTime plusWeeks(long weeks) 增加指定周数
         *   LocalDateTime plusHours(long hours) 增加指定小时
         *   LocalDateTime plusMinutes(long minutes) 增加指定分钟
         *   LocalDateTime plusSeconds(long seconds) 增加指定秒
         *   LocalDateTime plusNanos(long nanos) 增加指定纳秒
         *   plus(long amountToAdd, TemporalUnit unit) 指定增加的字段的日期时间
         */
        LocalDateTime plusLocalDateTime = LocalDateTime.of(2020,02,8,21, 30, 59, 123456789);
        System.out.println("常用的plusXxx()系列操作:" + plusLocalDateTime); // 常用的plusXxx()系列操作:2020-02-08T21:30:59.123456789
        System.out.println(plusLocalDateTime.plusYears(1)); // 2021-02-08T21:30:59.123456789
        System.out.println(plusLocalDateTime.plusMonths(1)); // 2020-03-08T21:30:59.123456789
        System.out.println(plusLocalDateTime.plusDays(6)); // 2020-02-14T21:30:59.123456789
        System.out.println(plusLocalDateTime.plusWeeks(1)); // 2020-02-15T21:30:59.123456789
        System.out.println(plusLocalDateTime.plusHours(1)); // 2020-02-08T22:30:59.123456789
        System.out.println(plusLocalDateTime.plusMinutes(1)); // 2020-02-08T21:31:59.123456789
        System.out.println(plusLocalDateTime.plusSeconds(1)); // 2020-02-08T21:31:00.123456789
        System.out.println(plusLocalDateTime.plusNanos(6)); // 2020-02-08T21:30:59.123456795
        System.out.println(plusLocalDateTime.plus(1, ChronoUnit.DAYS)); // 2020-02-09T21:30:59.123456789

        /**
         * 常用的minusXxx()系列操作，减少时间的方法:
         *   LocalDateTime minusYears(long years) 减去指定年份
         *   LocalDateTime minusMonths(long months) 减去指定月份
         *   LocalDateTime minusDays(long days) 减去指定天数
         *   LocalDateTime minusWeeks(long weeks) 减去指定周数
         *   LocalDateTime minusHours(long hours) 减去指定小时
         *   LocalDateTime minusMinutes(long minutes) 减去指定分钟
         *   LocalDateTime minusSeconds(long seconds) 减去指定秒
         *   LocalDateTime minusNanos(long nanos) 减去指定纳秒
         *   LocalDateTime minus(long amountToSubtract, TemporalUnit unit) 减少指定字段的日期时间
         */
        LocalDateTime minusLocalDateTime = LocalDateTime.of(2020,02,8,21, 30, 59, 123456789);
        System.out.println("常用的minusXxx()系列操作:" + minusLocalDateTime); // 常用的minusXxx()系列操作:2020-02-08T21:30:59.123456789
        System.out.println(minusLocalDateTime.minusYears(5)); // 2015-02-08T21:30:59.123456789
        System.out.println(minusLocalDateTime.minusMonths(60)); // 2015-02-08T21:30:59.123456789
        System.out.println(minusLocalDateTime.minusDays(1826)); // 2015-02-08T21:30:59.123456789
        System.out.println(minusLocalDateTime.minusWeeks(260)); // 2015-02-14T21:30:59.123456789
        System.out.println(minusLocalDateTime.minusHours(1)); // 2020-02-08T20:30:59.123456789
        System.out.println(minusLocalDateTime.minusMinutes(60)); // 2020-02-08T20:30:59.123456789
        System.out.println(minusLocalDateTime.minusSeconds(3600)); // 2020-02-08T20:30:59.123456789
        System.out.println(minusLocalDateTime.minusNanos(9)); // 2020-02-08T21:30:59.123456780
        System.out.println(minusLocalDateTime.minus(1, ChronoUnit.HOURS)); // 2020-02-08T20:30:59.123456789

        /**
         * 常用日期对比方法：
         *   int compareTo(localDateTimeOne other) 比较当前对象和other对象在时间上的大小，返回值如果为正，则当前对象时间较晚
         *   boolean isBefore(localDateTimeOne other) 比较当前对象时间是否在other对象时间之前
         *   boolean isAfter(localDateTimeOne other) 比较当前对象时间是否在other对象时间之后
         *   boolean isEqual(ChronoLocalDateTime other) 比较两个日期对象是否相等
         */
        LocalDateTime localDateTimeOne = LocalDateTime.of(2022,02,8,21, 30, 59, 123456789);
        LocalDateTime localDateTimeTwo = LocalDateTime.of(2020,02,8,21, 30, 59, 123456789);
        System.out.println("compareTo: " + localDateTimeOne.compareTo(localDateTimeTwo)); // compareTo: 2
        System.out.println("isBefore: " + localDateTimeOne.isBefore(localDateTimeTwo)); // isBefore: false
        System.out.println("isAfter: " + localDateTimeOne.isAfter(localDateTimeTwo)); // isAfter: true
        System.out.println("isEqual: " + localDateTimeOne.isEqual(localDateTimeTwo)); // isEqual: false

        /**
         * 从LocalDateTime实例获取时间戳
         * 从LocalDateTime获取时间戳稍微有点麻烦，需先把LocalDateTime实例转为Instant实例，再调用Instant实例的toEpochMilli方法获得对应的时间戳。
         * 下面示例从本地日期时间实例获取对应的时间戳
         */
        LocalDateTime now = LocalDateTime.now();
        Instant instant1 = now.toInstant(ZoneOffset.ofHours(8));
        System.out.println("timeFromLocal1 = " + instant1.toEpochMilli()); // timeFromLocal1 = 1645272488866
        /**
         * 上面获取代码基于北京时间，所以转为Instant实例时使用了东八区。
         * 倘若在东八区以外的其他地区运行上述代码，就无法得到正确的当地时间戳，此时要先设置当地的默认时区，再将LocalDateTime实例转为Instant实例
         */
        Instant instant2 = now.atZone(ZoneId.systemDefault()).toInstant();
        System.out.println("timeFromLocal2 = " + instant2.toEpochMilli()); // timeFromLocal2 = 1645272488866

        /**
         * 当前日期时间替换成指定的日期时间, 这里会用到一个方法adjustInto()
         * Temporal adjustInto(Temporal temporal) 将指定的时间对象调整为具有与此对象相同的日期和时间
         * 不常用（该方法不学也罢）
         */
        // 获取当前时间
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println("转换前的时间:" + localDateTime); // 转换前的时间:2022-02-19T22:25:20.859059200
        // 使用adjustInto()方法后
        LocalDateTime localDateTimeOf = LocalDateTime.of(2020,12,12,12,00,01);
        localDateTime = (LocalDateTime)localDateTimeOf.adjustInto(localDateTime);
        System.out.println("转换后的时间:" + localDateTime); // 转换后的时间:2020-12-12T12:00:01
    }
}
```

> **LocalDateTime、LocalDate、LocalTime 的常用方法总结**：
>
> | 方法名                                                       | 描述                                      |
> | ------------------------------------------------------------ | ----------------------------------------- |
> | now、now(ZoneId zone)                                        | 根据当前时间创建对象、指定时区创建对象    |
> | of(int year,int month,int dOfM,int hour,int minute,int second,int nanoOfSecond) | 估计指定日期时间创建对象，没有偏移量      |
> | toLocalDate()、toLocalTime()                                 | 将LocalDateTime转换为LocalDate或LocalTime |
> | getYear()                                                    | 获得年份                                  |
> | getMonth()、getMonthValue()                                  | 获得月份Month枚举和数字（1-12）           |
> | getDayOfMonth()、getDayOfYear()                              | 获得月份天数（1-31）和年份天数（1-366）   |
> | getDayOfWeek()                                               | 获得星期几DayOfWeek枚举类                 |
> | getHour()、getMinute()、getSecond()、getNano()               | 获得时、分、秒、纳秒                      |
> | withYear()、withMonth()、withDayOfMonth()、withDayOfYear()   | 设置指定年、月、当月的日、当年的日        |
> | withHour()、withMinute()、withSecond()、withNano()           | 设置指定时、分、秒、纳秒                  |
> | plusYears()、plusMonths()、plusDays()、plusWeeks()           | 增加指定年、月、日、周                    |
> | plusHours()、plusMinutes()、plusSeconds()、plusNanos()       | 增加指定时、分、秒、纳秒                  |
> | minusYears()、minusMonths()、minusDays()、minusWeeks()       | 减少指定年、月、日、周                    |
> | minusHours()、minusMinutes()、minusSeconds()、minusNanos()   | 减少指定时、分、秒、纳秒                  |
> | get(TemporalField field)                                     | 获取指定字段时间                          |
> | LocalDateTime with(TemporalField field, long newValue)       | 指定特殊时间                              |
> | plus(long amountToAdd, TemporalUnit unit)                    | 增加特殊时间                              |
> | minus(long amountToSubtract, TemporalUnit unit)              | 减少特殊时间                              |
>
> LocalDateTime、LocalDate、LocalTime 的静态方法 of() 参数取值范围：
>
> | 参数       | 说明                              |
> | ---------- | --------------------------------- |
> | year       | 从-999 999 999到999 999 999的年份 |
> | month      | 一年中的月份，从1到12             |
> | dayOfMonth | 月中的天，从1到31                 |
> | hour       | 从0到23表示的时                   |
> | minute     | 从0到59表示的分                   |
> | second     | 从0到59表示的秒                   |
>



## java.time.Year

Year 类代表**年份**，如 2014。这个类只与年份有关。我们可以用它来判断闰年（leap-year）或获取当前年份。它也是不可变日期时间对象。它继承了Object类并实现Comparable接口。如下是 Year 类的声明：

```java
public final class Year extends Object implements Temporal, TemporalAdjuster, Comparable<Year>, Serializable {}
```

Year 类的方法如下：

| 方法                                                         | 描述                                             |
| :----------------------------------------------------------- | :----------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定的时间对象以获得年份。                   |
| LocalDate atDay(int dayOfYear)                               | 将年份与年中的日期相结合，创建一个LocalDate。    |
| YearMonth atMonth(int month)                                 | 将年份与一个月份结合起来创造一个YearMonth。      |
| YearMonth atMonth(Month month)                               | 将年份与一个月份结合起来创造一个YearMonth。      |
| LocalDate atMonthDay(MonthDay monthDay)                      | 将年份与月份相结合以创建LocalDate。              |
| int compareTo(Year other)                                    | 将此年份到另一年份比较。                         |
| boolean equals(Object otherYear)                             | 检查引年份是否等于指定的年份。                   |
| String format(DateTimeFormatter formatter)                   | 此年份使用指定的格式化程序格式化。               |
| static Year from(TemporalAmount amount)                      | 从时间量获得Year的实例。                         |
| int get(TemporalField field)                                 | 从int获取此年份指定字段的值。                    |
| long getLong(TemporalField field)                            | 获取所请求单位的值。                             |
| int getValue()                                               | 获取年份值。                                     |
| int hashCode()                                               | 此年份的哈希码。                                 |
| boolean isAfter(Year other)                                  | 检查此年份是否在指定的年份之后。                 |
| boolean isBefore(Year other)                                 | 检查此年是否在指定年份之前。                     |
| boolean isLeap()                                             | 根据ISO符号日历系统规则，检查年份是否为闰年。    |
| boolean isLeap(long year)                                    | 根据ISO符号日历系统规则，检查年份是否为闰年。    |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                         |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                       |
| boolean isValidMonthDay(MonthDay monthDay)                   | 检查月份是否对此年份有效。                       |
| int length()                                                 | 以天为单位获取此年份的长度。                     |
| Year minus(long amountToSubtract, TemporalUnit unit)         | 返回此年份的副本，减去指定的总量。               |
| Year minus(TemporalAmount amountToSubtract)                  | 返回此年份的副本，并减去指定的年份。             |
| Year minusYears(long yearsToSubtract)                        | 返回此年份的副本，减去指定的年份。               |
| static Year now()                                            | 在默认时区中从系统时钟获取当前年份。             |
| static Year now(Clock clock)                                 | 从指定的时钟获得当前年份。                       |
| static Year now(ZoneId zone)                                 | 从指定时区的系统时钟获取当前年份。               |
| static Year of(int years)                                    | 获得Year的实例。                                 |
| static Year parse(CharSequence text)                         | 从文本字符串(例如2007)获取Year值。               |
| static Year parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取Year的实例。 |
| Year plus(long amountToAdd, TemporalUnit unit)               | 返回添加了指定年份的本年份副本。                 |
| Year plus(TemporalAmount amountToAdd)                        | 返回添加了指定年份的本年份副本。                 |
| Year plusMonths(long monthsToAdd)                            | 返回指定月份的本年份副本。                       |
| Year plusYears(long yearsToAdd)                              | 返回添加了指定年份的本年份副本。                 |
| R query(TemporalQuery query)                                 | 使用指定Query查询当前年份                        |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                       |
| String toString()                                            | 将此年份输出为String。                           |
| long until(Temporal endExclusive, TemporalUnit unit)         | 按指定单位计算直到另一年的时间量。               |
| Year with(TemporalAdjuster adjuster)                         | 返回此年份的调整副本。                           |

【操作示例】

```java
import java.time.Year;
public class YearDemo {
    public static void main(String[] args) {
        System.out.println("Year.now():" + Year.now()); // Year.now():2023
        System.out.println("Year.MAX_VALUE:" + Year.MAX_VALUE); // Year.MAX_VALUE:999999999
        System.out.println("Year.isLeap(2014):" + Year.isLeap(2014)); // Year.isLeap(2014):false
        System.out.println("Year.isLeap(2016):" + Year.isLeap(2016)); // Year.isLeap(2016):true
    }
}
```



## java.time.YearMonth

YearMonth 是代表**年份和月份组合**的类，如 2014-09。此类提供了获取给定日期字符串的年和月的方法。它也是不可变日期时间对象。它继承了Object类并实现Comparable接口。如下是 YearMonth 类的声明：

```java
public final class YearMonth extends Object 
    implements Temporal, TemporalAdjuster, Comparable<YearMonth>, Serializable {}
```

YearMonth 类的方法如下：

| 方法                                                         | 描述                                                  |
| :----------------------------------------------------------- | :---------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定的时间对象以具有此年-月。                     |
| LocalDate atDay(int dayOfMonth)                              | 将此年份和月份结合起来创建LocalDate                   |
| LocalDate atEndOfMonth()                                     | 返回月末的LocalDate。                                 |
| int compareTo(YearMonth other)                               | 此年月和另一个年月相比较。                            |
| boolean equals(Object otherYearMonth)                        | 检查此YearMonth是否等于指定的YearMonth。              |
| String format(DateTimeFormatter formatter)                   | 使用指定的格式化程序格式化此年月。                    |
| static YearMonth from(TemporalAmount amount)                 | 从时间量获得YearMonth的实例。                         |
| int get(TemporalField field)                                 | 获取此年份中指定字段的int值。                         |
| long getLong(TemporalField field)                            | 获取所请求单位的值。                                  |
| Month getMonth()                                             | 使用Month枚举获取月份字段。                           |
| int getMonthValue()                                          | 获取月份字段1到12之间的值。                           |
| int getYear()                                                | 获取年份字段的值。                                    |
| int hashCode()                                               | 此YearMonth的哈希码。                                 |
| boolean isAfter(YearMonth other)                             | 检查此年月是否在指定的年月之后。                      |
| boolean isBefore(YearMonth other)                            | 检查此年月是否在指定的年月之前。                      |
| boolean isLeapYear()                                         | 根据ISO符号日历系统规则，检查年份是否为闰年。         |
| boolean isLeap(long year)                                    | 根据ISO符号日历系统规则，检查年份是否为闰年。         |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                              |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                            |
| boolean isValidDay(int dayOfMonth)                           | 检查每月的日期是否有效。                              |
| int lengthOfMonth()                                          | 返回月份的长度。                                      |
| int lengthOfYear()                                           | 返回年份的长度。                                      |
| YearMonth minus(long amountToSubtract, TemporalUnit unit)    | 返回此年份的副本，减去指定的金额。                    |
| YearMonth minus(TemporalAmount amountToSubtract)             | 返回此YearMonth的副本，并减去指定的YearMonth。        |
| YearMonth minusMonths(long monthsToSubtract)                 | 返回此YearMonth的副本，并减去指定的月份。             |
| YearMonth minusYears(long yearsToSubtract)                   | 返回此YearMonth的副本，并减去指定的年份。             |
| static YearMonth now()                                       | 从默认时区中的系统时钟获取当前年月。                  |
| static YearMonth now(Clock clock)                            | 从指定的时钟获得当前年月。                            |
| static YearMonth now(ZoneId zone)                            | 从指定时区的系统时钟获取当前年月。                    |
| static YearMonth of(int years, int month)                    | 从指定年份和月份获得YearMonth的实例。                 |
| static YearMonth of(int years, Month month)                  | 从指定年份和月份获得YearMonth的实例。                 |
| static YearMonth parse(CharSequence text)                    | 从文本字符串(例如2007-12)获取YearMonth。              |
| static YearMonth parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取YearMonth的实例。 |
| YearMonth plus(long amountToAdd, TemporalUnit unit)          | 返回此YearMonth的副本，并添加指定的YearMonth。        |
| YearMonth plus(TemporalAmount amountToAdd)                   | 返回此YearMonth的副本，并添加指定的YearMonth。        |
| YearMonth plusMonths(long monthsToAdd)                       | 返回此YearMonth的副本，并添加指定的月份。             |
| YearMonth plusYears(long yearsToAdd)                         | 返回此YearMonth的副本，并添加指定的年份。             |
| R query(TemporalQuery query)                                 | 使用指定的查询来查询此年月。                          |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                            |
| String toString()                                            | 将此年份作为字符串输出。                              |
| long until(Temporal endExclusive, TemporalUnit unit)         | 根据指定的单位计算直到另一年月的时间量。              |
| YearMonth with(TemporalAdjuster adjuster)                    | 返回此年份的调整副本。                                |
| YearMonth with(TemporalField field, long newValue)           | 返回此年月的副本，并将指定的字段设置为新值。          |
| YearMonth withMonth(int month)                               | 返回此YearMonth的副本，其中年份已更改。               |
| YearMonth withYear(int year)                                 | 返回此YearMonth的副本，并更改年份。                   |

【操作示例】

```java
import java.time.YearMonth;
public class YearMonthDemo {
    public static void main(String[] args) {
        System.out.println("YearMonth.now():" + YearMonth.now()); // YearMonth.now():2023-05
        System.out.println("getMonthValue():" + YearMonth.parse("2014-09").getMonthValue()); // getMonthValue():9
        System.out.println("getYear():" + YearMonth.parse("2014-09").getYear()); // getYear():2014
        System.out.println("isLeapYear():" + YearMonth.parse("2014-09").isLeapYear()); // isLeapYear():false
    }
}
```



## java.time.MonthDay

MonthDay 表示**月份和日期的组合**。这个类不提供年份。它也是不可变日期时间对象。它继承了Object类并实现Comparable接口。如下是 MonthDay 类的声明：

```java
public final class MonthDay extends Object 
    implements TemporalAccessor, TemporalAdjuster, Comparable<MonthDay>, Serializable {}
```

MonthDay 类的方法如下：

| 方法                                                         | 描述                                                 |
| :----------------------------------------------------------- | :--------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定的时间对象以具有此月-日。                    |
| LocalDate atYear(int year)                                   | 将此月-日与一年组合以创建LocalDate。                 |
| int compareTo(MonthDay other)                                | 将这个月份的日期与另一个月份的日期进行比较。         |
| boolean equals(Object obj)                                   | 检查此月-日是否等于另一个月-日。                     |
| String format(DateTimeFormatter formatter)                   | 使用指定的格式化程序在本月创建格式。                 |
| static MonthDay from(TemporalAccessor temporal)              | 从时态对象获取MonthDay的实例。                       |
| int get(TemporalField field)                                 | 获取此月-日指定字段的int值。                         |
| int getDayOfMonth()                                          | 获取日期字段。                                       |
| long getLong(TemporalField field)                            | 从此时间为long获取指定字段的值。                     |
| Month getMonth()                                             | 使用Month枚举获取月份字段。                          |
| int getMonthValue()                                          | 获取1到12之间的月份字段。                            |
| int hashCode()                                               | 这个月的哈希码。                                     |
| boolean isAfter(MonthDay other)                              | 检查此月-日是否在指定的月-日之后。                   |
| boolean isBefore(MonthDay other)                             | 检查此月-日是否在指定的月-日之前。                   |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                             |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                           |
| boolean isValidYear(int year)                                | 检查年份是否适用于本月。                             |
| static MonthDay now()                                        | 在默认时区中从系统时钟获取当前时间。                 |
| static MonthDay now(Clock clock)                             | 从指定的时钟获得当前时间。                           |
| static MonthDay now(ZoneId zone)                             | 从指定时区的系统时钟获取当前时间。                   |
| static MonthDay of(int month, int dayOfMonth)                | 获得MonthDay的实例。                                 |
| static MonthDay of(Month month, int dayOfMonth)              | 获得MonthDay的实例。                                 |
| static MonthDay parse(CharSequence text)                     | 从文本字符串(例如：12-03)获取MonthDay的实例。        |
| static MonthDay parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取MonthDay的实例。 |
| R query(TemporalQuery query)                                 | 使用指定的查询进行查询。                             |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                           |
| String toString()                                            | 将此日期输出为字符串，例如：12-03。                  |
| MonthDay with(Month month)                                   | 返回此MonthDay的副本，其中月份已更改。               |
| MonthDay withDayOfMonth(int dayOfMonth)                      | 返回此MonthDay的副本，并更改日期。                   |
| MonthDay withMonth(int month)                                | 返回此MonthDay的副本，其中月份已更改。               |

【操作示例】在这个例子中，我展示了 MonthDay 的一些用途和工作。

```java
import java.time.MonthDay;
public class MonthDayDemo {
    public static void main(String[] args) {
        MonthDay monthDay = MonthDay.now();
        System.out.println(monthDay.getDayOfMonth()); // 23
        System.out.println(monthDay.getMonth()); // MAY
        System.out.println(monthDay.atYear(2021)); // 2021-05-23
    }
} 
```



## java.time.Month【枚举】

Month 是代表一年中12个**月份的枚举**。除文本枚举名称外，每年的每个月都有一个int值。它的声明如下：

```java
public enum Month extends Enum<Month> implements TemporalAccessor, TemporalAdjuster {}
```

Month 枚举类的枚举变量及方法如下：

| Enum Constant | 描述                                 |
| :------------ | :----------------------------------- |
| JANUARY       | 一月份的单例实例，为31天。           |
| FEBRUARY      | 二月的单例实例为28天，或闰年为29天。 |
| MARCH         | 三月份的单例实例，为31天。           |
| APRIL         | 四月份的单例实例，为期30天。         |
| MAY           | 五月份的单例实例，为31天。           |
| JUNE          | 六月份的单例实例，为期30天。         |
| JULY          | 七月份的单例实例，为31天。           |
| AUGUST        | 八月份的单身实例，为31天。           |
| SEPTEMBER     | 九月份的单例实例，为期30天。         |
| OCTOBER       | 十月份的单例实例，为31天。           |
| NOVEMBER      | 十一月份的单例实例，为期30天。       |
| DECEMBER      | 十二月份的单例实例，为31天。         |

| 方法                                                  | 描述                                               |
| :---------------------------------------------------- | :------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                | 将指定的时间对象调整为与该对象的月份相同。         |
| int firstDayOfYear(boolean leapYear)                  | 获取与本月第一天相对应的年份。                     |
| Month firstMonthOfQuarter()                           | 获取与该季度的第一个月相对应的月份。               |
| static Month from(TemporalAccessor temporal)          | 从时间对象获取Month的实例。                        |
| int get(TemporalField field)                          | 从一年中的这个月中获取指定字段的值作为一个整数。   |
| String getDisplayName(TextStyle style, Locale locale) | 获取文本表示形式，例如“ Jan”或“ December”。        |
| long getLong(TemporalField field)                     | 从一年的这个月中获取指定字段的值。                 |
| int getValue()                                        | 获取当月int值。                                    |
| boolean isSupported(TemporalField field)              | 检查是否支持指定的字段。                           |
| int length(boolean leapYear)                          | 获取本月的天数。                                   |
| int maxLength()                                       | 获取本月的最大长度（天）。                         |
| int minLength()                                       | 获取以天为单位的该月的最小长度。                   |
| Month minus(long months)                              | 返回一年中的月份，即该月份之前的指定月份。         |
| static Month of(int month)                            | 从int值获取Month的实例。                           |
| Month plus(long months)                               | 返回一年中的月份，该月份是该月份之后指定的季度数。 |
| R query(TemporalQuery query)                          | 使用指定的查询查询此偏移量。                       |
| ValueRange range(TemporalField field)                 | 获取指定字段的有效值范围。                         |
| static Month valueOf(String name)                     | 返回具有指定名称的此类型的枚举常量。               |
| static Month values()                                 | 以声明顺序返回包含此枚举类型的常量的数组。         |

【操作示例】

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.Year;

public class MonthDemo {
    public static void main(String[] args) {
        System.out.println(Month.MARCH); // MARCH
        System.out.println(Month.MARCH.getValue()); // 3
        System.out.println(Month.of(3)); // MARCH
        System.out.println(Month.valueOf("MARCH")); // MARCH

		Month month = Month.from(LocalDate.of(2022,02,02));
		System.out.println(month.getValue()); // 2
		System.out.println(month.name()); // FEBRUARY

		LocalDate localdate = Year.of(2022).atMonth(month).atDay(1);
		month= localdate.getMonth();
		System.out.println(month.getValue()); // 2
		System.out.println(month.name()); // FEBRUARY
	}
} 
```



## java.time.DayOfWeek【枚举】

DayOfWeek 表示**一周中的一天的枚举**，如 MONDAY，TUESDAY 等。除了日期名称也有一个数字值。它的声明如下：

```java
public enum DayOfWeek implements TemporalAccessor, TemporalAdjuster {}
```

DayOfWeek 枚举类的枚举变量及方法如下：

| Enum Constant | 描述                       |
| :------------ | :------------------------- |
| MONDAY        | 星期一的星期几的单例实例。 |
| TUESDAY       | 星期二的星期几的单例实例。 |
| WEDNESDAY     | 星期三的星期几的单例实例。 |
| THURSDAY      | 星期四的星期几的单例实例。 |
| FRIDAY        | 星期五的星期几的单例实例。 |
| SATURDAY      | 星期六的星期几的单例实例。 |
| SUNDAY        | 星期日的星期几的单例实例。 |

| 方法                                                  | 描述                                             |
| :---------------------------------------------------- | :----------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                | 调整指定的时态对象以具有此星期几。               |
| static DayOfWeek from(TemporalAccessor temporal)      | 从时态对象获得 DayOfWeek 的实例。                |
| int get(TemporalField field)                          | 从星期几获取指定字段的值，作为 int 。            |
| String getDisplayName(TextStyle style, Locale locale) | 获取文本表示，例如“Mon”或“Friday”。              |
| long getLong(TemporalField field)                     | 从 long 获取此星期几中指定字段的值。             |
| int getValue()                                        | 获取星期几 int 值。                              |
| boolean isSupported(TemporalField field)              | 检查是否支持指定的字段。                         |
| DayOfWeek minus(long days)                            | 返回星期几，即此前一天的指定天数。               |
| static DayOfWeek of(int dayOfWeek)                    | 从 int 值获得 DayOfWeek 的实例。                 |
| DayOfWeek plus(long days)                             | 返回星期几，即此后的指定天数。                   |
| R query(TemporalQuery query)                          | 使用指定的查询查询此星期几。                     |
| ValueRange range(TemporalField field)                 | 获取指定字段的有效值范围。                       |
| static DayOfWeek valueOf(String name)                 | 返回具有指定名称的此类型的枚举常量。             |
| static DayOfWeek[] values()                           | 按照声明的顺序返回一个包含此枚举类型常量的数组。 |

【操作示例】

```java
import java.time.DayOfWeek;
import java.time.format.TextStyle;
import java.util.Locale;

public class DayOfWeekDemo {
    public static void main(String[] args) {
        // DayOfWeek 是枚举类型，由 7 个常量组成，代表了一周七天
        System.out.println("DayOfWeek.MONDAY : " + DayOfWeek.MONDAY); // DayOfWeek.MONDAY : MONDAY
        System.out.println("DayOfWeek.TUESDAY : " + DayOfWeek.TUESDAY); // DayOfWeek.TUESDAY : TUESDAY

        // 整型值范围[1,7]，可以通过加减数字推算
        System.out.println("周二往前退4天 : " + DayOfWeek.TUESDAY.minus(4)); // 周二往前退4天 : FRIDAY
        System.out.println("周五往后进3天 : " + DayOfWeek.FRIDAY.plus(3)); // 周五往后进3天 : MONDAY

        // getDisplayName(TextStyle, Locale) 方法定制字符串内容
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.FULL, Locale.CHINESE)); // 星期日
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.SHORT, Locale.CHINESE)); // 周日
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.NARROW, Locale.CHINESE)); // 日

        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.FULL, Locale.ENGLISH)); // Sunday
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.SHORT, Locale.ENGLISH)); // Sun
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.NARROW, Locale.ENGLISH)); // S

        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.FULL, Locale.FRENCH)); // dimanche
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.SHORT, Locale.FRENCH)); // dim.
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.NARROW, Locale.FRENCH)); // D

        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.FULL, Locale.GERMANY)); // Sonntag
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.SHORT, Locale.GERMANY)); // So.
        System.out.println(DayOfWeek.SUNDAY.getDisplayName(TextStyle.NARROW, Locale.GERMANY)); // S
    }
}
```



## java.time 时区先导篇

首先介绍：LocalDateTime、OffsetDateTime 和 ZoneDateTime 之间的关系，且与 ZoneOffset（偏移量）和 ZoneId（时区） 的概念。

```
┌─────────────┬─────────────┬────────────┬────────────┐
│  LocalDate  │  LocalTime  │ ZoneOffset │   ZoneId   │
└─────────────┴─────────────┴────────────┴────────────┘
┌───────────────────────────┐
│       LocalDateTime       │
└───────────────────────────┘
              ┌──────────────────────────┐
              │        OffsetTime        │
              └──────────────────────────┘
┌────────────────────────────────────────┐
│             OffsetDateTime             │
└────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│                    ZonedDateTime                    │
└─────────────────────────────────────────────────────┘

分割时间(ZonedDateTime): 2022-02-18T20:41:14.164538200+08:00[Asia/Shanghai]
LocalDate：2022-02-18
LocalTime：T20:41:14.164538200
ZoneOffset：+08:00
ZoneId：[Asia/Shanghai]
```

```java
import java.time.*;
public class JavaDateTest {
    public static void main(String[] args) {
        System.out.println(LocalDate.now()); // 2022-02-19
        System.out.println(LocalTime.now()); // 18:52:15.279221600
        System.out.println(LocalDateTime.now()); // 2022-02-19T18:52:15.279221600
        System.out.println(ZoneOffset.ofHours(8)); // +08:00
        System.out.println(ZoneId.systemDefault()); // Asia/Shanghai
        System.out.println(OffsetDateTime.now()); // 2022-02-19T18:52:15.279221600+08:00
        System.out.println(ZonedDateTime.now()); // 2022-02-19T18:52:15.279221600+08:00[Asia/Shanghai]
    }
}
```



## java.time.ZoneId

> JAVA8之 日期时间时区之 ZoneId[ZoneOffset, ZoneRegion] 笔记：https://blog.csdn.net/kfepiza/article/details/115433132

### 1、ZoneId 类说明

ZoneId 是 java.time 引入的新的时区类，注意和旧的 java.util.TimeZone 区别。时区从基准 UTC 开始的一个固定偏移。ZoneId 该类中包含了所有的时区信息，并提供在 Instant 和 LocalDateTime 之间进行转换的规则。

其中每个时区都对应着 ID，ID有两种不同的类型：

- 固定的偏移量：相对于UTC格林威治标准时间的完全解析的偏移量，它对所有本地日期时间都使用相同的偏移量。
- 地理区域：适用于查找与UTC /格林威治的偏移量的一组特定规则的区域

ZoneId 是抽象类，它有两个继承实现类 ZoneOffset 与 ZoneRegion：

- ZoneOffset 表示时区偏移量：代表了从伦敦格林威治零度子午线开始的时间偏移，也就是时差（例如：+08:00）。ZoneOffset 可以表示大多数固定偏移量。在任何ZoneId上调用normalized() 将确保将固定的偏移量ID表示为ZoneOffset。
- ZoneRegion 表示地理区域：格式是：洲（州、国家）/城市。最常见的区域分类是时区数据库（TZDB：Europe/Paris和Asia/Tokyo来区分地区）。ID的格式为 “区域/城市” 。例如 ：Asia/Shanghai 等。ZoneRules 定义了描述偏移量何时更改以及如何更改的实际规则，此类仅是用于获取基础规则的ID。之所以采用这种方法，是因为规则是由政府定义且频繁更改的，而该ID是稳定的。

为什么要定义 ZoneRegion 和 ZoneOffset 两个类来表示时区呢？这是因为人们可以通过两种方式来定义时区。

- 第一种方式是通过城市或地区来定义时区。世界上有很多城市和地区，这些城市或地区都处在某个特定的时区当中，因此如果指定了一个城市或地区，就能确定它所在的时区。ZoneRegion就是代表城市或地区的类，当一个ZoneRegion类对象被创建后，ZoneRegion类对象所代表的那个城市或地区的时区也会被确定，所以说一个ZoneRegion类对象就代表一个时区。

- 第二种定义时区的方式是指定时差。我们知道：全球共分为24个时区，人们把英国格林尼治时间天文台所在的那个时区称为“零时区”或“中时区”，其他各个时区与零时区都有时差。因此只需要指出与零时区的时差，也能确定一个时区。ZoneOffset 就是代表时差的类，但必须说明：ZoneOffset 类对象所代表的时差并不是任意两个地区的时差，而是特指与零时区的时差。ZoneOffset 类对象中包含了与零时区的时差数值，所以ZoneOffset类对象也能代表一个时区。



### 2、ZoneId 时区ID类型

时区ID在系统内是唯一的。时区 ID 有三种类型：

- 第一种最简单的ID类型是ZoneOffset中的ID，它由’Z’和以’+‘或’-'开头的ID组成。例如：+08:00、-080000、UTC+08:00:00、Z
- 第二种ID是带有某种形式的前缀的偏移样式ID，例如’GMT +2’或’UTC +01:00’。可识别的前缀为’UTC’，‘GMT’和’UT’。后缀将在创建过程中进行规范化。可以使用 normalized() 将这些ID规范化为ZoneOffset。
- 第三种ID类型是基于区域的ID。基于区域的ID必须包含两个或多个字符，并且不能以’UTC’，‘GMT’，‘UT’，’+‘或’-'开头。基于区域的ID由配置定义，请参见ZoneRulesProvider。提供从ID到底层ZoneRules的查找。

时区规则的定义及使用：

- 时区规则由政府定义并经常更改。有许多组织（在此称为组）来监视时区更改并进行整理。默认组是IANA时区数据库（TZDB）。其他组织包括IATA （航空业团体）和Microsoft。

- 每个组为其提供的区域ID定义其自己的格式.TZDB组定义诸如欧洲/伦敦’‘或美国/纽约’'的ID.TZDB ID优先于其他组。

- 强烈建议将组名包括在TZDB以外的其他组提供的所有ID中，以免发生冲突。例如，国际航空运输协会（IATA）航空公司的时区区域ID通常与三个字母的机场代码相同，但是乌得勒支机场的代码为’UTC’，这显然是冲突的.TZDB以外的其他组的区域ID的推荐格式是’group〜region’。因此，如果定义了IATA数据，乌得勒支机场将是’IATA〜UTC’。



### 3、ZoneId 类方法及示例

| 方法                                                     | 描述                                                 |
| :------------------------------------------------------- | :--------------------------------------------------- |
| boolean equals(Object obj)                               | 检查此时区ID是否等于另一个时区ID                     |
| static ZoneId from(TemporalAccessor temporal)            | 从时态对象获取ZoneId的实例                           |
| static Set getAvailableZoneIds()                         | 获取可用区域ID的集合                                 |
| String getDisplayName(TextStyle style, Locale locale)    | 获取区域的文本表示，例如:British Time或+02:00        |
| abstract String getId()                                  | 获取唯一的时区ID                                     |
| abstract ZoneRules getRules()                            | 获取此ID的时区规则，允许执行计算                     |
| int hashCode()                                           | 此时区ID的哈希码                                     |
| ZoneId normalized()                                      | 规范化时区ID，尽可能返回ZoneOffset                   |
| static ZoneId of(String zoneId)                          | 从ID获取ZoneId的实例，确保该ID有效且可供使用         |
| static ZoneId of(String zoneId, Map aliasMap)            | 使用别名映射使用其ID获取ZoneId的实例以补充标准区域ID |
| static ZoneId ofOffset(String prefix, ZoneOffset offset) | 获得包装偏移量的ZoneId实例                           |
| static ZoneId systemDefault()                            | 获取系统默认时区                                     |
| String toString()                                        | 使用ID将此区域输出为String                           |

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Set;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 常用API：
         *   systemDefault()：获取系统默认时区的ID
         *   of(String zoneName)：根据各个地区的时区ID名创建对象
         *   getAvailableZoneIds()：获取世界各个地方的时区的集合
         */
        System.out.println(ZoneId.systemDefault()); // Asia/Shanghai
        System.out.println(ZoneId.of("Asia/Kolkata")); // Asia/Kolkata
        System.out.println(ZoneId.of("Asia/Tokyo")); // Asia/Tokyo
        System.out.println(ZoneId.of("+08:00:00")); // +08:00
        System.out.println(ZoneId.of("UTC+08:00:00")); // UTC+08:00
        System.out.println(LocalDateTime.now(ZoneId.of("Asia/Kolkata"))); // 2022-02-19T17:24:39.643968100
        System.out.println(LocalDateTime.now(ZoneId.of("Asia/Kolkata"))); // 2022-02-19T17:24:39.643968100
        Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
        System.out.println(availableZoneIds);
        // [Asia/Aden, America/Cuiaba, Etc/GMT+9, Etc/GMT+8, ...省略后面输出
    }
}
```



### 4、ZoneId 类的实例化详解

ZoneId 的实例可能是 ZoneOffset 或 ZoneRegion，我们查看 of 静态方法实例化的源码：

```java
static ZoneId of(String zoneId, boolean checkAvailable) {
    Objects.requireNonNull(zoneId, "zoneId");
    if (zoneId.length() <= 1 || zoneId.startsWith("+") || zoneId.startsWith("-")) {
        return ZoneOffset.of(zoneId);
    } else if (zoneId.startsWith("UTC") || zoneId.startsWith("GMT")) {
        return ofWithPrefix(zoneId, 3, checkAvailable);
    } else if (zoneId.startsWith("UT")) {
        return ofWithPrefix(zoneId, 2, checkAvailable);
    }
    return ZoneRegion.ofId(zoneId, checkAvailable);
}
```

从源码中可以看出：

- 如果以加或减{+|-}开头，则创建ZoneOffset实例
- 如果以{GMT|UTC|UT}开头，则创建ZoneRegion实例
- ZoneId.of(“UT+8”) 得到 ZoneRegion
- ZoneId.of("+8") 得到 ZoneOffset
- ZoneId.of("+08:00") 得到 ZoneOffset
- ZoneId.of(“Asia/Shanghai”) 得到 ZoneRegion

【操作示例】我们自己使用案例验证

```java
import java.time.ZoneId;

public class Main {
    public static void main(String[] args) {
        System.out.println(ZoneId.of("+08:00:00").getClass().getName()); // java.time.ZoneOffset
        System.out.println(ZoneId.of("UTC+08:00:00").getClass().getName()); // java.time.ZoneRegion
        System.out.println(ZoneId.of("Asia/Shanghai").getClass().getName()); // java.time.ZoneOffset
        System.out.println(ZoneId.systemDefault().getClass().getName()); // java.time.ZoneOffset
    }
}
```



### 5、ZoneId 与 TimeZone 互转

有意思的是ZoneId.systemDefault()就是通过TimeZone.getDefault().toZoneId()来获得，我们查看源码：

```java
    public static ZoneId systemDefault() {
        return TimeZone.getDefault().toZoneId();
    }

    private transient ZoneId zoneId;
    public ZoneId toZoneId() {
        ZoneId zId = zoneId;
        if (zId == null) {
            zoneId = zId = toZoneId0();
        }
        return zId;
    }
    private ZoneId toZoneId0() {
        String id = getID();
        TimeZone defaultZone = defaultTimeZone;
        // are we not defaultTimeZone but our id is equal to default's?
        if (defaultZone != this &&
            defaultZone != null && id.equals(defaultZone.getID())) {
            // delegate to default TZ which is effectively immutable
            return defaultZone.toZoneId();
        }
        // derive it ourselves
        if (ZoneInfoFile.useOldMapping() && id.length() == 3) {
            if ("EST".equals(id))
                return ZoneId.of("America/New_York");
            if ("MST".equals(id))
                return ZoneId.of("America/Denver");
            if ("HST".equals(id))
                return ZoneId.of("America/Honolulu");
        }
        return ZoneId.of(id, ZoneId.SHORT_IDS);
    }
```

【操作示例 1】TimeZone 转 ZoneId【**注意**：TimeZone可以用"CTT" , ZoneId不可以】

```java
ZoneId zoneId1 = TimeZone.getTimeZone("GMT+8").toZoneId(); // GMT+08:00
ZoneId zoneId2 = TimeZone.getTimeZone("GMT+8:00").toZoneId(); // GMT+08:00
ZoneId zoneId3 = TimeZone.getTimeZone("GMT+08:00").toZoneId(); // GMT+08:00
ZoneId zoneId4 = TimeZone.getTimeZone("Asia/Shanghai").toZoneId(); // Asia/Shanghai
ZoneId zoneId5 = TimeZone.getTimeZone("Asia/Chongqing").toZoneId(); // Asia/Chongqing
ZoneId zoneId6 = TimeZone.getTimeZone("Asia/Chungking").toZoneId(); // Asia/Chungking
ZoneId zoneId7 = TimeZone.getTimeZone("Asia/Hong_Kong").toZoneId(); // Asia/Hong_Kong
ZoneId zoneId8 = TimeZone.getTimeZone("CTT").toZoneId(); // Asia/Shanghai
// TimeZone可以用"CTT" , ZoneId不可以
```

【操作示例 2】ZoneId 转 TimeZone

```java
TimeZone timeZone = TimeZone.getTimeZone(ZoneId.of("GMT+8"));
System.out.println(timeZone);
// sun.util.calendar.ZoneInfo[id="GMT+08:00",offset=28800000,dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
```



ZoneId 创建 ZoneOffset、ZoneId 创建 ZoneRegion



## java.time.ZoneOffset

### 1、ZoneOffset 类说明

1、java.time.ZoneOffset 类用于表示距UTC时区的固定区域偏移量。它继承了 ZoneId 类并实现 Comparable 接口。类定义如下：

```java
public final class ZoneOffset extends ZoneId 
    implements TemporalAccessor, TemporalAdjuster, Comparable<ZoneOffset>, Serializable {}
```

2、ZoneOffset 类声明三个常量【ZoneOffset的取值范围为正负18小时】：

```java
/** The time-zone offset for UTC, with an ID of 'Z'. */
public static final ZoneOffset UTC = ZoneOffset.ofTotalSeconds(0);
/** Constant for the minimum supported offset. */
public static final ZoneOffset MIN = ZoneOffset.ofTotalSeconds(-MAX_SECONDS);
/** Constant for the maximum supported offset. */
public static final ZoneOffset MAX = ZoneOffset.ofTotalSeconds(MAX_SECONDS);
```

- MAX：这是支持的最大区域偏移量【正18小时】
- MIN：这是支持的最小区域偏移量【负18小时】
- UTC：这是UTC的时区偏移常数【中心偏移量 Z】

3、MAX_SECONDS 取值范围（正负18小时），这表示 ZoneOffset 的取值范围为正负18小时

```java
/**
 * The abs maximum seconds.
 */
private static final int MAX_SECONDS = 18 * SECONDS_PER_HOUR;
```

4、两个ConcurrentMap作为秒缓存和id缓存

```java
/** Cache of time-zone offset by offset in seconds. */
private static final ConcurrentMap<Integer, ZoneOffset> SECONDS_CACHE = new ConcurrentHashMap<>(16, 0.75f, 4);
/** Cache of time-zone offset by ID. */
private static final ConcurrentMap<String, ZoneOffset> ID_CACHE = new ConcurrentHashMap<>(16, 0.75f, 4);
```



### 2、ZoneOffset 类方法及示例

| 方法                                                         | 描述                                           |
| :----------------------------------------------------------- | :--------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定时态对象以使其具有与此对象相同的偏移量 |
| static ZoneOffset from(TemporalAccessor temporal)            | 从temporal对象获取ZoneOffset的实例             |
| int get(TemporalField field)                                 | 从此偏移量中获取指定字段的int值                |
| String getId()                                               | 获取规范化区域偏移ID                           |
| long getLong(TemporalField field)                            | 从此偏移量中获取指定字段的long值               |
| int getTotalSeconds()                                        | 获取以秒为单位的总区域偏移量                   |
| static ZoneOffset of(String offsetId)                        | 使用ID获取ZoneOffset的实例                     |
| static ZoneOffset ofHours(int hours)                         | 使用以小时为单位的偏移量获取ZoneOffset的实例   |
| static ZoneOffset ofHoursMinutes(int hours, int minutes)     | 使用小时和分钟的偏移量获取ZoneOffset的实例     |
| static ZoneOffset ofHoursMinutesSeconds(int hours, int minutes, int seconds) | 使用小时，分钟和秒的偏移量获取ZoneOffset的实例 |
| static ZoneOffset ofTotalSeconds(int totalSeconds)           | 指定以秒为单位的总偏移量获取 ZoneOffset 实例   |

```java
import java.time.*;
import java.time.temporal.Temporal;

public class JavaDateTest {
    public static void main(String[] args) {
        System.out.println(ZoneOffset.MAX); // +18:00
        System.out.println(ZoneOffset.MIN); // -18:00
        System.out.println(ZoneOffset.UTC); // Z
        // System.out.println(ZoneOffset.of("+100")); // 超出最大范围, 会抛异常
        
       
        Temporal temp = ZoneOffset.UTC.adjustInto(ZonedDateTime.now());
        System.out.println(temp); // 2022-02-19T18:55:40.567526+08:00[Asia/Shanghai]
        ZoneOffset zone = ZoneOffset.from(ZonedDateTime.now());
        System.out.println(zone); // +08:00
        
        System.out.println(ZoneOffset.MAX.getId()); // +18:00
        System.out.println(ZoneOffset.MAX.getTotalSeconds()); // 64800
        System.out.println(ZoneOffset.MAX.get(ChronoField.OFFSET_SECONDS)); // 64800
        System.out.println(ZoneOffset.MAX.getLong(ChronoField.OFFSET_SECONDS)); // 64800
        
        // 获取X小时偏移量的ZoneOffset对象
        System.out.println(ZoneOffset.ofHours(5)); // +05:05
        System.out.println(ZoneOffset.ofHoursMinutes(5, 5)); // +05:05
        System.out.println(ZoneOffset.ofHoursMinutesSeconds(5, 5, 5)); // +05:05:05
        System.out.println(ZoneOffset.ofTotalSeconds(64800)); // +18:00
    }
}
```



### 3、ZoneOffset 实例化两种方式

1、【方式一】使用 ZoneId 创建 ZoneOffset 对象，如果给 ZoneId.of() 方法传递的字符串参数代表与零时区的时差，ZoneId.of() 方法将会创建一个 ZoneOffset 类的对象。

```java
ZoneId zid1 = ZoneId.of("+08:00");       // ①
ZoneId zid2 = ZoneId.of("+8");           // ②
ZoneId zid3 = ZoneId.of("+080000");      // ③
ZoneId zid4 = ZoneId.of("+08:30:00");    // ④
ZoneId zid5 = ZoneId.of("UTC+08:00:00"); // ⑤
ZoneId zid6 = ZoneId.of("GMT+08:00:00"); // ⑥
System.out.println("时区信息: " + zid1.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("时区信息: " + zid2.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("时区信息: " + zid3.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("时区信息: " + zid4.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:30]
System.out.println("时区信息: " + zid5.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("时区信息: " + zid6.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
```

- 其中语句①为of()方法所传递的参数为“+08:00”，它表示比零时区的时间早8小时0分。参数中“+”表示比零时区的时间更早，如果换成“-”，则表示比零时区的时间更晚。语句②为of()方法所传递的参数为“+8”，这是一种简略的写法，也表示比零时区的时间早8小时。需要注意：只有在省略分钟的情况下才能省略数字8之前的0，也就是说不能把参数写为“+8:00”。语句③为of()方法传递的参数为“080000”，它表示比零时区的时间早8小时0分0秒，实际上，这个参数的标准写法是“08:00:00”，Java语言规定：如果参数中时、分、秒都用两位数来表示，那么它们之间的冒号(:)可以省略。语句④所创建的对象表示比零时区的时间早8小时30分。这说明ZoneOffset对象所代表的时区与零时区的时差并不一定是1小时的整数倍，它可以与零时区相差任意长度的时间，只要这个时差不超过18小时即可。语句⑤和⑥中，字符串参数中出现了“UTC”和“GMT”，它们代表了两种时间系统。“GMT”表示格林尼治时间，它是通过计算地球自转周期所确定的时间系统，而“UTC”表示协调世界时，它是通过计算原子吸收或释放能量时发出电磁波的频率所确定的时间系统。

2、【方式二】可以直接通过 ZoneOffset 类中所定义的 of() 方法来创建 ZoneOffset 对象，需要注意：这个 ZoneOffset.of() 方法的参数中不能带有“UTC”或“GMT”。除此之外，ZoneOffset类还定义了一系列以of开头的方法用于创建ZoneOffset类的对象，并且还定义了getTotalSeconds()方法用于计算与零时区的时差是多少秒。

```java
// 设置时差为8小时
ZoneOffset zo1 = ZoneOffset.of("+08:00");
ZoneOffset zo2 = ZoneOffset.of("+8");
ZoneOffset zo3 = ZoneOffset.of("+080000");
ZoneOffset zo4 = ZoneOffset.of("+08:30:00");
// 设置时差为3600秒(1小时)
ZoneOffset zo5 = ZoneOffset.ofTotalSeconds(3600);
// 设置时差为3小时
ZoneOffset zo6 = ZoneOffset.ofHours(3);
// 设置时差为3小时30分
ZoneOffset zo7 = ZoneOffset.ofHoursMinutes(3, 30);
// 设置时差为4小时20分45秒
ZoneOffset zo8 = ZoneOffset.ofHoursMinutesSeconds(4, 20, 45);

System.out.println("时区信息: " + zo1.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("时区信息: " + zo2.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("时区信息: " + zo3.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("时区信息: " + zo4.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+08:30]
System.out.println("时区信息: " + zo5.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+01:00]
System.out.println("时区信息: " + zo6.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+03:00]
System.out.println("时区信息: " + zo7.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+03:30]
System.out.println("时区信息: " + zo8.getRules()); // 时区信息: ZoneRules[currentStandardOffset=+04:20:45]
```

3、ZoneOffset.of() 与 ZoneId.of() 字符串参数的区别：

```java
import java.time.*;

public class JavaDateTest {
    public static void main(String[] args) {
        System.out.println(ZoneId.of("UTC+8"));
        System.out.println(ZoneOffset.of("+08:00"));

        System.out.println(ZoneId.of("+08:00"));
        System.out.println(ZoneOffset.of("UTC+8"));
        // java.time.DateTimeException: Invalid ID for ZoneOffset, non numeric characters found: UTC+8
    }
}
```

- ZoneOffset 的静态 of(String)方法，String以 +或- 开头（查看源码可以得知：开头必须为正或负号：+ 或 -）否则报错。
- ZoneId 可以以 {GMT|UTC|UT} 开头，这样则创建 ZoneRegion 实例，不过可以使用 normalized() 将这些ID规范化为ZoneOffset。



## java.time.ZoneRegion

### 1、ZoneRegion 类说明

1、ZoneRegion 不是对外使用的，修饰符是default（只能同包下类调用）。只能通过ZoneId来操作。查看 ZoneRegion 的声明：

```java
/**
 * 适用于相同时区规则的地理区域。
 * 
 * 时区信息归类为一组规则，这些规则定义了何时以及如何更改与UTC /格林威治的偏移量。 
 * 可使用基于地理区域（例如国家或州）的标识符来访问这些规则。
 * 最常见的区域分类是时区数据库（TZDB），时区数据库定义了“欧洲/巴黎”和“亚洲/东京”等区域。
 * 
 * 由此类建模的区域标识符与由ZoneRules建模的基础规则不同，这些规则由政府定义且经常更改，相比之下，区域标识符定义明确且寿命长，这种分隔也使规则能够 适当时在区域之间共享。 
 * 
 * @implSpec
 * This class is immutable and thread-safe.
 * 此类不可变,且线程安全
 * @since 1.8
 */
final class ZoneRegion extends ZoneId implements Serializable {}
```

2、ZoneRegion de构造方法也没有public，只能由同包下的类调用

```java
ZoneRegion(String id, ZoneRules rules) {
    this.id = id;
    this.rules = rules;
}
```

3、ZoneRegion 的成员变量，可以看到 ZoneRegion 保存一个String id 和 ZoneRules，这里又牵扯到了 ZoneRules（Zone规则）

```java
/**
 * The time-zone ID, not null.
  */
private final String id;
/**
 * The time-zone rules, null if zone ID was loaded leniently.
 */
private final transient ZoneRules rules;
```

4、ZoneRule 的类定义

```java
/**
 * 定义区域偏移量在单个时区中如何变化的规则。
 * 
 * 该规则为时区建模了所有历史和未来的过渡。 ZoneOffsetTransition用于已知的过渡，通常是历史性的。 
 * ZoneOffsetTransitionRule用于基于算法结果的将来过渡。
 * 
 * 这些规则通过ZoneRulesProvider使用ZoneId加载。
 * 相同的规则可以在多个区域ID之间内部共享。
 * 
 * 序列化ZoneRules实例将存储整个规则集。
 * 它不存储区域ID，因为它不是该对象状态的一部分。
 * 
 * 规则实现可能存储或不存储有关历史和未来过渡的完整信息，并且存储的信息仅与规则提供者提供给实现的信息一样准确。
 * 应用程序应将提供的数据视为代表可用于实现此规则的最佳信息。 。 
 *
 * @implSpec
 * This class is immutable and thread-safe.
 *
 * @since 1.8
 */
public final class ZoneRules implements Serializable {}
```

5、ZoneRegion的ofId(String zoneId, boolean checkAvailable) 方法

```java
/**
 * Obtains an instance of {@code ZoneId} from an identifier.
 *
 * @param zoneId  the time-zone ID, not null
 * @param checkAvailable  whether to check if the zone ID is available
 * @return the zone ID, not null
 * @throws DateTimeException if the ID format is invalid
 * @throws ZoneRulesException if checking availability and the ID cannot be found
 */
static ZoneRegion ofId(String zoneId, boolean checkAvailable) {
    Objects.requireNonNull(zoneId, "zoneId");
    checkName(zoneId);
    ZoneRules rules = null;
    try {
        // always attempt load for better behavior after deserialization
        rules = ZoneRulesProvider.getRules(zoneId, true);
    } catch (ZoneRulesException ex) {
        if (checkAvailable) {
            throw ex;
        }
    }
    return new ZoneRegion(zoneId, rules);
}
```



### 2、ZoneRegion 实例化

可以通 过ZoneId 类的 of() 静态方法来创建它的子类对象，如果为 of() 方法传递的字符串参数代表一个城市或地区，那么 of() 方法将会创建出一个 ZoneRegion 类的对象。此外，还可以调用 systemDefault() 方法来创建一个表示系统默认地区的 ZoneRegion 类的对象。

【操作示例 1】使用 ZoneId 创建 ZoneRegion 类对象

```java
ZoneId zid1 = ZoneId.systemDefault();
ZoneId zid2 = ZoneId.of("Europe/Paris");
System.out.println("zid1的值: " + zid1); // zid1的值: Asia/Shanghai
System.out.println("zid2的值: " + zid2); // zid2的值: Europe/Paris
System.out.println("zid1类型: " + zid1.getClass().getName()); // zid1的类型: java.time.ZoneRegion
System.out.println("zid2类型: " + zid2.getClass().getName()); // zid2的类型: java.time.ZoneRegion
System.out.println("zid1时区信息: " + zid1.getRules()); // zid1的时区信息: ZoneRules[currentStandardOffset=+08:00]
System.out.println("zid2时区信息: " + zid2.getRules()); // zid2的时区信息: ZoneRules[currentStandardOffset=+01:00]
```

- 需要说明：ZoneRegion不是一个public类，所以在程序中无法直接访问这个类，也创建不出这个类型的引用。由于ZoneRegion和ZoneOffset都代表时区，所以程序中一般都统一使用ZoneId类的引用指向这两个类的对象。此外还可以看出：直接打印ZoneRegion类对象只能输出ZoneRegion类对象所代表的城市或地区，而操作系统默认的时区用ZoneRegion类对象表示出来是“Asia/Shanghai”，意为“亚洲/上海”，并不是人们常用的北京。如果希望打印ZoneRegion类对象的时区信息，需要调用getRules()方法。zid1的时区信息为“ZoneRules[currentStandardOffset=+08:00]”，其中“+08:00”表示这个时区比零时区的时间早8个小时。同理，zid2时区信息中的“+01:00”表示法国巴黎所在的时区比零时区早1个小时。

【操作示例 2】获取所有可用区域ID的集合

```java
Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
System.out.println(availableZoneIds);
// [Asia/Aden, America/Cuiaba, Etc/GMT+9, Etc/GMT+8, ...省略后面输出
```

- 需要注意：并不是任意一个大洲的名称加上城市的名称所组成的字符串都可以成为of()方法的参数，例如“Asia/Beijing”就不可以当作of()方法的参数，如果以这个字符串作为参数，程序在执行of()方法时会抛出异常。所有能够作为of()方法参数的字符串都被存放到一个集合中，程序员可以通过ZoneId类定义的getAvailableZoneIds()静态方法来获得这个集合，遍历这个集合就可以得到所有可以作为of()方法参数的字符串。



## java.time.OffsetDateTime

### 1、OffsetDateTime 类说明

OffsetDateTime 与 OffseTime 都属于 ISO-8601 日历系统中与 UTC 偏移量有关的日期时间。

OffsetDateTime 类是一个**带有偏移量**的日期时间类型（不包含基于 ZoneRegion 的时间偏移量）。用于存储日期和时间字段，以及偏移量，精度为纳秒。可以简单理解为 OffsetDateTime = LocalDateTime + ZoneOffset。

OffsetDateTime、ZonedDateTime 和 Instant 它们三都能在时间线上以纳秒精度存储一个瞬间（注意：LocalDateTime 是不行的），也可理解我某个时刻。OffsetDateTime 和 Instant 可用于模型的字段类型，因为它们都表示瞬间值并且还不可变，所以适合网络传输或者数据库持久化。【**PS**：ZonedDateTime 不适合网络传输/持久化，因为即使同一个 ZoneId 时区，不同地方获取到瞬时值也有可能不一样】

OffsetDateTime 类是不可变表示形式，它继承了 Object 类并实现 Comparable 接口。OffsetDateTime 类声明：

```java
public final class OffsetDateTime extends Object 
    implements Temporal, TemporalAdjuster, Comparable<OffsetDateTime>, Serializable {}
```

比如现在我们在UTC+8:00 的位置，所以当地时间是 2021-06-09 15:00:00，用OffsetDateTime来表示就是 2021-06-09 07:00:00 -8:00。



### 2、OffsetDateTime 类方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定的时态对象以使其具有与此对象相同的日期和时间。       |
| ZonedDateTime atZoneSameInstant(ZoneId zone)                 | 将此日期时间与时区组合以创建ZonedDateTime，以确保结果具有相同的时刻。 |
| ZonedDateTime atZoneSimilarLocal(ZoneId zone)                | 将此日期时间与时区组合以创建ZonedDateTime，以尝试保持相同的本地日期和时间。 |
| int compareTo(OffsetDateTime other)                          | 将此日期时间与另一个日期时间进行比较。                       |
| boolean equals(Object obj)                                   | 检查此日期时间是否等于另一个日期时间。                       |
| String format(DateTimeFormatter formatter)                   | 使用指定的格式化程序格式化此日期时间。                       |
| static OffsetDateTime from(TemporalAccessor temporal)        | 从时态对象获取OffsetDateTime的实例。                         |
| int get(TemporalField field)                                 | 从此日期时间获取指定字段作为int值。                          |
| int getDayOfMonth()                                          | 获取日期字段。                                               |
| DayOfWeek getDayOfWeek()                                     | 获取星期几字段，即枚举DayOfWeek。                            |
| int getDayOfYear()                                           | 获取日期字段。                                               |
| int getHour()                                                | 获取当日时间字段。                                           |
| long getLong(TemporalField field)                            | 从此日期时间获取指定字段转为long的值。                       |
| Month getMinute()                                            | 获取分钟字段。                                               |
| Month getMonth()                                             | 使用Month枚举获取月份字段。                                  |
| int getMonthValue()                                          | 获取1到12之间的月份字段。                                    |
| int getNano()                                                | 获取纳秒级字段。                                             |
| ZoneOffset getOffset()                                       | 获取区域偏移量，例如"+01:00"。                               |
| int getSecond()                                              | 获取秒钟字段。                                               |
| int getYear()                                                | 获取年份字段。                                               |
| int hashCode()                                               | 此日期时间的哈希码。                                         |
| boolean isAfter(OffsetDateTime other)                        | 检查此日期时间是否在指定的日期时间之后。                     |
| boolean isBefore(OffsetDateTime other)                       | 检查此日期时间是否在指定的日期时间之前。                     |
| boolean isEqual(OffsetDateTime other)                        | 检查此日期时间是否等于指定的日期时间。                       |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                                     |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                                   |
| OffsetDateTime minus(long amountToSubtract, TemporalUnit unit) | 返回此日期时间的副本，并减去指定的数量。                     |
| OffsetDateTime minus(TemporalAmount amountToSubtract)        | 返回此日期时间的副本，并减去指定的数量。                     |
| OffsetDateTime minusDays(long daysToSubtract)                | 返回此OffsetDateTime的副本，并减去指定的天数。               |
| OffsetDateTime minusHours(long hoursToSubtract)              | 返回此OffsetDateTime的副本，并减去指定的小时数。             |
| OffsetDateTime minusMinutes(long minutesToSubtract)          | 返回此OffsetDateTime的副本，并减去指定的分钟数。             |
| OffsetDateTime minusMonths(long monthsToSubtract)            | 返回此OffsetDateTime的副本，并减去指定的月数。               |
| OffsetDateTime minusNanos(long nanos)                        | 返回此OffsetDateTime的副本，并减去指定的纳秒数。             |
| OffsetDateTime minusSeconds(long seconds)                    | 返回此OffsetDateTime的副本，并减去指定的秒数。               |
| OffsetDateTime minusWeeks(long weeksToSubtract)              | 返回此OffsetDateTime的副本，并减去指定的周数。               |
| OffsetDateTime minusYears(long yearsToSubtract)              | 返回此OffsetDateTime的副本，并减去指定的年数。               |
| static OffsetDateTime now()                                  | 从默认时区中的系统时钟获取当前日期时间。                     |
| static OffsetDateTime now(Clock clock)                       | 从指定的时钟获得当前日期时间。                               |
| static OffsetDateTime now(ZoneId zone)                       | 从指定时区的系统时钟获取当前日期时间。                       |
| static OffsetDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond, ZoneOffset offset) | 从年，月，日，小时，分钟，秒，纳秒和偏移量获得OffsetDateTime的实例。 |
| static OffsetDateTime of(LocalDate date, LocalTime time, ZoneOffset offset) | 从日期，时间和偏移量中获取OffsetDateTime的实例。             |
| static OffsetDateTime of(LocalDateTime dateTime, ZoneOffset offset) | 从日期时间和偏移量获得OffsetDateTime的实例。                 |
| static OffsetDateTime ofInstant(Instant instant, ZoneId zone) | 从Instant和区域ID获取OffsetDateTime的实例。                  |
| static OffsetDateTime parse(CharSequence text)               | 从文本字符串中获取OffsetDateTime的实例，例如:2007-12-03T10:15:30。 |
| static OffsetDateTime parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取OffsetDateTime的实例。   |
| OffsetDateTime plus(long amountToAdd, TemporalUnit unit)     | 返回此日期时间的副本，并添加指定的数量。                     |
| OffsetDateTime plus(TemporalAmount amountToAdd)              | 返回此日期时间的副本，并添加指定的数量。                     |
| OffsetDateTime plusDays(long daysToAdd)                      | 返回此OffsetDateTime的副本，并添加指定的天数。               |
| OffsetDateTime plusHours(long hoursToAdd)                    | 返回此OffsetDateTime的副本，并添加指定的小时数。             |
| OffsetDateTime plusMinutes(long minutesToAdd)                | 返回此OffsetDateTime的副本，并添加指定的分钟数。             |
| OffsetDateTime plusMonths(long monthsToAdd)                  | 返回此OffsetDateTime的副本，并添加指定的月份数。             |
| OffsetDateTime plusNanos(long nanos)                         | 返回此OffsetDateTime的副本，其中添加了指定的纳秒数。         |
| OffsetDateTime plusSeconds(long seconds)                     | 返回此OffsetDateTime的副本，并添加指定的周数。               |
| OffsetDateTime plusWeeks(long weeksToAdd)                    | 返回此OffsetDateTime的副本，并添加指定的周数。               |
| OffsetDateTime plusYears(long yearsToAdd)                    | 返回此OffsetDateTime的副本，其中添加了指定的年数。           |
| R query(TemporalQuery query)                                 | 使用指定的查询查询此日期时间。                               |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                                   |
| static Comparator timeLineOrder()                            | 获取一个比较器，它仅根据瞬间比较两个OffsetDateTime实例。     |
| long toEpochSecond()                                         | 将此日期时间转换为1970-01-01T00:00:00Z纪元的秒数。           |
| Instant toInstant()                                          | 将此日期时间转换为即时。                                     |
| LocalDate toLocalDate()                                      | 获取此日期时间的LocalDate部分。                              |
| LocalDateTime toLocalDateTime()                              | 获取此日期时间的LocalDateTime部分。                          |
| LocalTime toLocalTime()                                      | 获取此日期时间的LocalTime部分。                              |
| OffsetTime toOffsetTime()                                    | 获取此日期时间的OffsetTime部分。                             |
| String toString()                                            | 将此日期输出为字符串，例如2007-12-03T10:15:30。              |
| ZonedDateTime toZonedDateTime()                              | 使用偏移量作为区域ID将此日期时间转换为ZonedDateTime。        |
| OffsetDateTime truncatedTo(TemporalUnit unit)                | 返回此OffsetDateTime的副本，并截断时间。                     |
| long until(Temporal endExclusive, TemporalUnit unit)         | 根据指定的单位计算到另一个日期时间的时间量。                 |
| OffsetDateTime with(TemporalAdjuster adjuster)               | 返回此日期时间的调整副本。                                   |
| OffsetDateTime with(TemporalField field, long newValue)      | 返回此日期时间的副本，并将指定字段设置为新值。               |
| OffsetDateTime withDayOfMonth(int dayOfMonth)                | 返回此OffsetDateTime的副本，并更改日期。                     |
| OffsetDateTime withDayOfYear(int dayOfYear)                  | 返回此OffsetDateTime的副本，其中包含日期更改。               |
| OffsetDateTime withHour(int hour)                            | 返回此OffsetDateTime的副本，并更改日期。                     |
| OffsetDateTime withMinute(int minute)                        | 返回此OffsetDateTime的副本，并更改了分钟。                   |
| OffsetDateTime withMonth(int month)                          | 返回此OffsetDateTime的副本，其中年份已更改。                 |
| OffsetDateTime withNano(int nanoOfSecond)                    | 返回此OffsetDateTime的副本，并更改了纳秒。                   |
| OffsetDateTime withOffsetSameInstant(ZoneOffset offset)      | 返回具有指定偏移量的此OffsetDateTime的副本，确保结果处于同一时刻。 |
| OffsetDateTime withOffsetSameLocal(ZoneOffset offset)        | 返回具有指定偏移量的此OffsetDateTime的副本，确保结果具有相同的本地日期时间。 |
| OffsetDateTime withSecond(int second)                        | 返回此OffsetDateTime的副本，并在第二分钟更改。               |
| OffsetDateTime withYear(int year)                            | 返回此OffsetDateTime的副本，并更改年份。                     |



### 3、OffsetDateTime 类示例

```java
import java.time.*;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 最大/最小值
         * 偏移量的最大值是+18，最小值是-18，这是由ZoneOffset内部的限制决定的。
         */
        OffsetDateTime min = OffsetDateTime.MIN;
        OffsetDateTime max = OffsetDateTime.MAX;
        System.out.println("OffsetDateTime最小值: " + min);
        System.out.println("OffsetDateTime最大值: " + max); 
        System.out.println(min.getOffset() + ":" + min.getYear() + "-" + min.getMonthValue() + "-" + min.getDayOfMonth()); 
        System.out.println(max.getOffset() + ":" + max.getYear() + "-" + max.getMonthValue() + "-" + max.getDayOfMonth()); 
        // 输出内容:
        // OffsetDateTime最小值: -999999999-01-01T00:00+18:00
        // OffsetDateTime最大值: +999999999-12-31T23:59:59.999999999-18:00
        // +18:00:-999999999-1-1
        // -18:00:999999999-12-31

        /**
         * OffsetDateTime的实例化
         */
        OffsetDateTime offsetDateTime1 = OffsetDateTime.now(); // 当前北京时间 2023-05-25T15:12:27
        OffsetDateTime offsetDateTime2 = OffsetDateTime.now(ZoneId.of("America/New_York"));
        OffsetDateTime offsetDateTime3 = OffsetDateTime.now(ZoneId.of("UTC+08:00:00"));
        OffsetDateTime offsetDateTime4 = OffsetDateTime.of(LocalDateTime.now(), ZoneOffset.of("-04:00"));
        System.out.println("默认时区偏移量的本地时间: " + offsetDateTime1);  
        System.out.println("纽约时区偏移量的本地时间: " + offsetDateTime2);  
        System.out.println("偏移量+8(上海)的本地时间: " + offsetDateTime3); 
        System.out.println("偏移量-4(纽约)的本地时间: " + offsetDateTime4); 
        // 输出内容:
        // 默认时区偏移量的本地时间: 2023-05-25T15:12:27.821601600+08:00
        // 纽约时区偏移量的本地时间: 2023-05-25T03:12:27.825588200-04:00
        // 偏移量+8(上海)的本地时间: 2023-05-25T15:12:27.828593500+08:00
        // 偏移量-4(纽约)的本地时间: 2023-05-25T15:12:27.828593500-04:00

        /**
         * 转换：LocalDateTime -> OffsetDateTime
         * 通过此例值得注意的是：LocalDateTime#atOffset()/atZone()只是增加了偏移量/时区，本地时间是并没有改变的。
         * 若想实现本地时间到其它偏移量的对应的时间只能通过其ofInstant()系列构造方法。
         */
        LocalDateTime  localDateTime1  = LocalDateTime.of(2022, 12, 12, 18, 00, 00);
        OffsetDateTime offsetDateTime5 = OffsetDateTime.of(localDateTime1, ZoneOffset.ofHours(-4));
        OffsetDateTime offsetDateTime6 = localDateTime1.atOffset(ZoneOffset.ofHours(-4));
        OffsetDateTime offsetDateTime7 = OffsetDateTime.ofInstant(localDateTime1.toInstant(ZoneOffset.ofHours(8)), ZoneOffset.ofHours(-4));
        System.out.println("当前时区(北京)时间为: " + localDateTime1);
        System.out.println("-4偏移量地方的晚上18点(方式一): " + offsetDateTime5);
        System.out.println("-4偏移量地方的晚上18点(方式二): " + offsetDateTime6);
        System.out.println("当前地区(北京)对应的-4地方的时间: " + offsetDateTime7);
        // 输出内容:
        // 当前时区(北京)时间为: 2022-12-12T18:00
        // -4偏移量地方的晚上18点(方式一): 2022-12-12T18:00-04:00
        // -4偏移量地方的晚上18点(方式二): 2022-12-12T18:00-04:00
        // 当前地区(北京)对应的-4地方的时间: 2022-12-12T06:00-04:00

        /**
         * 转换：OffsetDateTime -> LocalDateTime 
         * 注意：转为LocalDateTime 注意: 时间还是未变的哦
         */
        LocalDateTime  localDateTime2  = LocalDateTime.now();
        OffsetDateTime offsetDateTime8 = OffsetDateTime.of(localDateTime2, ZoneOffset.ofHours(-4));
        LocalDateTime  localDateTime3  = offsetDateTime8.toLocalDateTime();
        System.out.println("当前时区(北京)时间为: " + localDateTime2);
        System.out.println("-4偏移量时间为: " + offsetDateTime8);
        System.out.println("LocalDateTime的表示形式: " + localDateTime3);
        // 输出内容:
        // 当前时区(北京)时间为: 2023-05-25T15:25:40.973201400
        // -4偏移量时间为: 2023-05-25T15:25:40.973201400-04:00
        // LocalDateTime的表示形式: 2023-05-25T15:25:40.973201400
        
        /**
         * OffsetDateTime -> ZonedDateTime
         */
        OffsetDateTime offsetDateTime = OffsetDateTime.of(LocalDateTime.now(), ZoneOffset.ofHours(-4));
        System.out.println("-4偏移量时间为：" + offsetDateTime); 
        // 转换为ZonedDateTime的表示形式
        ZonedDateTime zonedDateTime1 = offsetDateTime.toZonedDateTime();
        ZonedDateTime zonedDateTime2 = offsetDateTime.atZoneSameInstant(ZoneId.of("America/New_York"));
        ZonedDateTime zonedDateTime3 = offsetDateTime.atZoneSimilarLocal(ZoneId.of("America/New_York"));
        System.out.println(zonedDateTime1);
        System.out.println(zonedDateTime2);
        System.out.println(zonedDateTime3);
        // 输出内容:
        // -4偏移量时间为：2023-06-04T16:46:01.148810900-04:00
        // 2023-06-04T16:46:01.148810900-04:00
        // 2023-06-04T16:46:01.148810900-04:00[America/New_York]
        // 2023-06-04T16:46:01.148810900-04:00[America/New_York]
    }
}
```

本例有值得关注的点：

- atZoneSameInstant()：将此日期时间与时区结合起来创建ZonedDateTime，以确保结果具有相同的Instant。所有偏移量-4 -> -5，时间点也从19 -> 18，确保了Instant保持一致嘛
- atZoneSimilarLocal：将此日期时间与时区结合起来创建ZonedDateTime，以确保结果具有相同的本地时间。所以直接效果和toLocalDateTime()是一样的，但是它会尽可能的保留偏移量（所以你看-4变为了-5，保持了真实的偏移量）

这里需要注意 OffsetDateTime.now() 与 OffsetDateTime.of() 使用区别：

- OffsetDateTime.now() 与 OffsetDateTime.now(ZoneId.of("UTC+08:00:00") 及 OffsetDateTime.of()返回时间相同，都是返回本届默认时区的时间，但是偏移量会显示设置的偏移量
- 而 OffsetDateTime.now(ZoneId.of("America/New_York")) 则返回当前设置时区的时间
- 如果想要转换不同时区或者偏移量的时间，可以使用 OffsetDateTime.ofInstant() 方法。



## java.time.OffsetTime

### 1、OffsetTime 类说明

OffsetDateTime 与 OffseTime 都属于 ISO-8601 日历系统中与 UTC 偏移量有关的日期时间。

OffsetTime 类是一个**带有偏移量**的时间类型（不包含基于 ZoneRegion 的时间偏移量）。该偏移量可以被视为小时，分钟，秒的偏移量（hour-minute-second-offset）。可以简单理解为 OffsetTime = LocalTime + ZoneOffset。

OffsetTime 类是一个不变的日期时间对象，它继承了 Object 类并实现Comparable 接口。类声明如下：

```java
public final class OffsetTime extends Object 
    implements Temporal, TemporalAdjuster, Comparable<OffsetTime>, Serializable {}
```



### 2、OffsetTime 类方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定的时态对象以使其具有与此对象相同的日期和时间。       |
| OffsetDateTime atDate(LocalDate date)                        | 将此时间与日期组合以创建OffsetDateTime。                     |
| int compareTo(OffsetTime other)                              | 这个时间与另一个时间比较。                                   |
| boolean equals(Object obj)                                   | 检查此时间是否等于另一个指定的时间。                         |
| String format(DateTimeFormatter formatter)                   | 使用指定的格式化程序格式化。                                 |
| static OffsetTime from(TemporalAccessor temporal)            | 从时态对象获取OffsetTime的实例。                             |
| int get(TemporalField field)                                 | 从此时间获取指定字段的int值。                                |
| int getHour()                                                | 获取当天小时时间字段。                                       |
| long getLong(TemporalField field)                            | 从此时间为long获取指定字段的值。                             |
| int getMinute()                                              | 获取分钟字段。                                               |
| int getNano()                                                | 获取纳秒级字段。                                             |
| ZoneOffset getOffset()                                       | 获取区域偏移量，例如"+01:00"。                               |
| int getSecond()                                              | 获取秒钟字段。                                               |
| int hashCode()                                               | 获取秒钟字段。                                               |
| boolean isAfter(OffsetTime other)                            | 检查此OffsetTime的瞬间是否在指定时间之后，将两次应用于共同日期。 |
| boolean isBefore(OffsetTime other)                           | 检查此OffsetTime的瞬间是否在指定时间之前，将两次应用于共同日期。 |
| boolean isEqual(OffsetTime other)                            | 检查此OffsetTime的瞬间是否等于指定时间的瞬间，将两次应用于共同日期。 |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                                     |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                                   |
| OffsetTime minus(long amountToSubtract, TemporalUnit unit)   | 返回此时间的副本，并减去指定的数量。                         |
| OffsetTime minus(TemporalAmount amountToSubtract)            | 返回此时间的副本，并减去指定的数量。                         |
| OffsetTime minusHours(long hoursToSubtract)                  | 返回此OffsetTime的副本，并减去指定的小时数。                 |
| OffsetTime minusMinutes(long minutesToSubtract)              | 返回此OffsetTime的副本，并减去指定的分钟数。                 |
| OffsetTime minusNanos(long nanos)                            | 返回此OffsetTime的副本，并减去指定的纳秒数。                 |
| OffsetTime minusSeconds(long seconds)                        | 返回此OffsetTime的副本，并减去指定的秒数。                   |
| static OffsetTime now()                                      | 在默认时区中从系统时钟获取当前时间。                         |
| static OffsetTime now(Clock clock)                           | 从指定的时钟获得当前时间。                                   |
| static OffsetTime now(ZoneId zone)                           | 从指定时区的系统时钟获取当前时间。                           |
| static OffsetTime of(int hour, int minute, int second, int nanoOfSecond, ZoneOffset offset) | 从小时，分钟，秒，纳秒和偏移量获得OffsetTime的实例。         |
| static OffsetTime of(LocalTime time, ZoneOffset offset)      | 从本地时间和偏移量获取OffsetTime的实例。                     |
| static OffsetTime ofInstant(Instant instant, ZoneId zone)    | 从Instant和区域ID获取OffsetTime的实例。                      |
| static OffsetTime parse(CharSequence text)                   | 从文本字符串(例如10:15:30 + 01:00)获取OffsetTime的实例。     |
| static OffsetTime parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式化程序从文本字符串中获取OffsetTime的实例。       |
| OffsetTime plus(long amountToAdd, TemporalUnit unit)         | 返回此时间的副本，并添加指定的数量。                         |
| OffsetTime plus(TemporalAmount amountToAdd)                  | 返回此时间的副本，并添加指定的数量。                         |
| OffsetTime plusHours(long hoursToAdd)                        | 返回此OffsetTime的副本，并添加指定的小时数。                 |
| OffsetTime plusMinutes(long minutesToAdd)                    | 返回此OffsetTime的副本，并添加指定的分钟数。                 |
| OffsetTime plusNanos(long nanos)                             | 返回此OffsetTime的副本，并添加指定的纳秒数。                 |
| OffsetTime plusSeconds(long seconds)                         | 返回此OffsetTime的副本，并添加指定的秒数。                   |
| R query(TemporalQuery query)                                 | 这次使用指定的查询进行查询。                                 |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                                   |
| LocalTime toLocalTime()                                      | 获取此日期时间的LocalTime部分。                              |
| String toString()                                            | 将此日期输出为String，例如10:15。                            |
| OffsetTime truncatedTo(TemporalUnit unit)                    | 返回此OffsetTime的副本，并截断时间。                         |
| long until(Temporal endExclusive, TemporalUnit unit)         | 根据指定的单位计算到另一个时间的时间量。                     |
| OffsetTime with(TemporalAdjuster adjuster)                   | 返回此时的调整副本。                                         |
| OffsetTime with(TemporalField field, long newValue)          | 返回此时间的副本，并将指定字段设置为新值。                   |
| OffsetTime withHour(int hour)                                | 返回此OffsetTime的副本，并更改日期。                         |
| OffsetTime withMinute(int minute)                            | 返回此OffsetTime的副本，并更改了分钟。                       |
| OffsetTime withNano(int nanoOfSecond)                        | 返回此OffsetTime的副本，并更改纳秒。                         |
| OffsetTime withOffsetSameInstant(ZoneOffset offset)          | 返回具有指定偏移量的此OffsetTime的副本，确保结果在隐含日期的同一时刻。 |
| OffsetTime withOffsetSameLocal(ZoneOffset offset)            | 返回具有指定偏移量的此OffsetTime的副本，确保结果具有相同的本地时间。 |
| OffsetTime withSecond(int second)                            | 返回此OffsetTime的副本，并更改秒钟。                         |



### 3、OffsetTime 类示例

```java
import java.time.*;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 最大/最小值
         * 偏移量的最大值是+18，最小值是-18，这是由ZoneOffset内部的限制决定的。
         */
        OffsetTime min = OffsetTime.MIN;
        OffsetTime max = OffsetTime.MAX;
        System.out.println("OffsetTime最小值: " + min);
        System.out.println("OffsetTime最大值: " + max);
        System.out.println(min.getOffset() + ":" + min.getHour() + "-" + min.getMinute() + "-" + min.getSecond());
        System.out.println(max.getOffset() + ":" + max.getHour() + "-" + max.getMinute() + "-" + max.getSecond());
        // 输出内容:
        // OffsetTime最小值: 00:00+18:00
        // OffsetTime最大值: 23:59:59.999999999-18:00
        // +18:00:0-0-0
        // -18:00:23-59-59

        /**
         * OffsetTime 的实例化
         */
        OffsetTime offsetTime1 = OffsetTime.now(); // 当前北京时间 2023-05-25T15:12:27
        OffsetTime offsetTime2 = OffsetTime.now(ZoneId.of("America/New_York"));
        OffsetTime offsetTime3 = OffsetTime.now(ZoneId.of("UTC+08:00:00"));
        OffsetTime offsetTime4 = OffsetTime.of(LocalTime.now(), ZoneOffset.of("-04:00"));
        System.out.println("默认时区偏移量的本地时间: " + offsetTime1);
        System.out.println("纽约时区偏移量的本地时间: " + offsetTime2);
        System.out.println("偏移量+8(上海)的本地时间: " + offsetTime3);
        System.out.println("偏移量-4(纽约)的本地时间: " + offsetTime4);
        // 输出内容:
        // 默认时区偏移量的本地时间: 17:43:58.872763900+08:00
        // 纽约时区偏移量的本地时间: 05:43:58.874758500-04:00
        // 偏移量+8(上海)的本地时间: 17:43:58.876759200+08:00
        // 偏移量-4(纽约)的本地时间: 17:43:58.876759200-04:00

        /**
         * 转换：LocalTime -> OffsetTime
         * 通过此例值得注意的是：LocalTime#atOffset()/atZone()只是增加了偏移量/时区，本地时间是并没有改变的。
         * 若想实现本地时间到其它偏移量的对应的时间只能通过其ofInstant()系列构造方法。
         */
        LocalTime  localTime1  = LocalTime.of(18, 00, 00);
        OffsetTime offsetTime5 = OffsetTime.of(localTime1, ZoneOffset.ofHours(-4));
        OffsetTime offsetTime6 = localTime1.atOffset(ZoneOffset.ofHours(-4));
        OffsetTime offsetTime7 = OffsetTime.ofInstant(localTime1.atDate(LocalDate.of(2022,2,2)).toInstant(ZoneOffset.ofHours(8)), ZoneOffset.ofHours(-4));
        System.out.println("当前时区(北京)时间为: " + localTime1);
        System.out.println("-4偏移量地方的晚上18点(方式一): " + offsetTime5);
        System.out.println("-4偏移量地方的晚上18点(方式二): " + offsetTime6);
        System.out.println("当前地区(北京)对应的-4地方的时间: " + offsetTime7);
        // 输出内容:
        // 当前时区(北京)时间为: 18:00
        // -4偏移量地方的晚上18点(方式一): 18:00-04:00
        // -4偏移量地方的晚上18点(方式二): 18:00-04:00
        // 当前地区(北京)对应的-4地方的时间: 06:00-04:00

        /**
         * 转换：OffsetTime -> LocalTime
         */
        LocalTime  localTime2  = LocalTime.now();
        OffsetTime offsetTime8 = OffsetTime.of(localTime2, ZoneOffset.ofHours(-4));
        LocalTime  localTime3  = offsetTime8.toLocalTime();
        System.out.println("当前时区(北京)时间为: " + localTime2);
        System.out.println("-4偏移量时间为: " + offsetTime8);
        System.out.println("LocalTime的表示形式: " + localTime3);
        // 输出内容:
        // 当前时区(北京)时间为: 17:43:58.878768
        // -4偏移量时间为: 17:43:58.878768-04:00
        // LocalTime的表示形式: 17:43:58.878768
    }
}
```



## java.time.ZonedDateTime

### 1、ZonedDateTime 类说明

ZonedDateTime 类是带有时区的日期时间的不可变表示形式，用于存储所有日期和时间字段（精度为纳秒），以及时区，其时区偏移量用于处理模棱两可的本地日期时间。

ZonedDateTime 其实就是带有时区的 LocalDateTime，可以理解为：ZonedDateTime = LocalDateTime + ZoneId。其实 LocalDate、LocalTime、LocalDateTime、Year、YearMonth、MonthDay、Month、DayOfWeek 都是不带时区的。

它继承了 Object 类并实现了 ChronoZonedDateTime 接口。ZonedDateTime 声明如下：

```java
public final class ZonedDateTime extends Object implements Temporal, ChronoZonedDateTime<LocalDate>, Serializable {}
```



### 2、ZonedDateTime 类方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| boolean equals(Object obj)                                   | 检查此日期时间是否等于另一个日期时间。                       |
| String format(DateTimeFormatter formatter)                   | 使用指定的格式化程序格式化此日期时间。                       |
| static ZonedDateTime from(TemporalAccessor temporal)         | 从temporal对象获取ZonedDateTime的实例。                      |
| int get(TemporalField field)                                 | 从此日期时间获取指定字段的int值。                            |
| int getDayOfMonth()                                          | 获取日期字段。                                               |
| DayOfWeek getDayOfWeek()                                     | 获取星期几字段，即枚举DayOfWeek。                            |
| int getDayOfYear()                                           | 获取日期字段。                                               |
| int getHour()                                                | 获取小时字段。                                               |
| long getLong(TemporalField field)                            | 从此日期时间获取指定字段的long值。                           |
| Month getMinute()                                            | 获取分钟字段。                                               |
| Month getMonth()                                             | 使用Month枚举获取月份字段。                                  |
| int getMonthValue()                                          | 获取1到12之间的月份字段。                                    |
| int getNano()                                                | 获取纳秒级字段。                                             |
| ZoneOffset getOffset()                                       | 获取区域偏移量，例如:+01:00。                                |
| int getSecond()                                              | 获取秒钟字段。                                               |
| int getYear()                                                | 获取年份字段。                                               |
| ZoneId getZone()                                             | 获取时区，例如:"Europe/Paris"。                              |
| int hashCode()                                               | 此日期时间的哈希码。                                         |
| boolean isSupported(TemporalField field)                     | 检查是否支持指定的字段。                                     |
| boolean isSupported(TemporalUnit unit)                       | 检查指定的单元是否受支持。                                   |
| ZonedDateTime minus(long amountToSubtract, TemporalUnit unit) | 返回此日期时间的副本，并减去指定的数量。                     |
| ZonedDateTime minus(TemporalAmount amountToSubtract)         | 返回此ZonedDateTime的副本，并减去指定的天数。                |
| ZonedDateTime minusDays(long daysToSubtract)                 | 返回此ZonedDateTime的副本，并减去指定的天数。                |
| ZonedDateTime minusHours(long hoursToSubtract)               | 返回此ZonedDateTime减去指定的小时数的副本。                  |
| ZonedDateTime minusMinutes(long minutesToSubtract)           | 返回此ZonedDateTime减去指定的分钟数的副本。                  |
| ZonedDateTime minusMonths(long monthsToSubtract)             | 返回此ZonedDateTime减去指定的月数的副本。                    |
| ZonedDateTime minusNanos(long nanos)                         | 返回此ZonedDateTime减去指定的纳秒数的副本。                  |
| ZonedDateTime minusSeconds(long seconds)                     | 返回此ZonedDateTime减去指定的秒数的副本。                    |
| ZonedDateTime minusWeeks(long weeksToSubtract)               | 返回此ZonedDateTime并减去指定的周数的副本。                  |
| ZonedDateTime minusYears(long yearsToSubtract)               | 返回此ZonedDateTime减去指定的年数的副本。                    |
| static ZonedDateTime now()                                   | 从默认时区中的系统时钟获取当前日期时间。                     |
| static ZonedDateTime now(Clock clock)                        | 从指定的时钟获得当前日期时间。                               |
| static ZonedDateTime now(ZoneId zone)                        | 从指定时区的系统时钟获取当前日期时间。                       |
| static ZonedDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond, ZoneId zone) | 从年，月，日，小时，分钟，秒，纳秒和时区获得ZonedDateTime的实例。 |
| static ZonedDateTime of(LocalDate date, LocalTime time, ZoneId zone) | 从日期和时间获取ZonedDateTime的实例。                        |
| static ZonedDateTime of(LocalDateTime date, ZoneId zone)     | 从本地日期时间获取ZonedDateTime的实例。                      |
| static ZonedDateTime ofInstant(Instant instant, ZoneId zone) | 从Instant和区域ID获取ZonedDateTime的实例。                   |
| static ZonedDateTime ofInstant(LocalDateTime localDateTime, ZoneOffset offset, ZoneId zone) | 从通过组合本地日期时间和偏移量形成的瞬间获得ZonedDateTime的实例。 |
| static ZonedDateTime ofLocal(LocalDateTime localDateTime, ZoneId zone, ZoneOffset preferredOffset) | 如果可能，使用首选偏移量从本地日期时间获取ZonedDateTime的实例。 |
| static ZonedDateTime ofStrict(LocalDateTime localDateTime, ZoneOffset offset, ZoneId zone) | 获得ZonedDateTime的实例，严格验证本地日期时间，偏移量和区域ID的组合。 |
| static ZonedDateTime parse(CharSequence text)                | 从文本字符串中获取ZonedDateTime的实例，例如2007-12-03T10:15:30+01:00[Europe/Paris]。 |
| static ZonedDateTime parse(CharSequence text, DateTimeFormatter formatter) | 使用特定格式器从文本字符串获取ZonedDateTime的实例。          |
| ZonedDateTime plus(long amountToAdd, TemporalUnit unit)      | 返回此日期时间添加指定的数量的副本。                         |
| ZonedDateTime plus(TemporalAmount amountToAdd)               | 返回此日期时间添加指定的数量的副本。                         |
| ZonedDateTime plusDays(long daysToAdd)                       | 返回此ZonedDateTime添加指定的天数的副本。                    |
| ZonedDateTime plusHours(long hoursToAdd)                     | 返回此ZonedDateTime添加指定的小时数的副本。                  |
| ZonedDateTime plusMinutes(long minutesToAdd)                 | 返回此ZonedDateTime添加指定的分钟数的副本。                  |
| ZonedDateTime plusMonths(long monthsToAdd)                   | 返回此ZonedDateTime并添加指定的月数的副本。                  |
| ZonedDateTime plusNanos(long nanos)                          | 返回此ZonedDateTime添加了指定的纳秒数的副本。                |
| ZonedDateTime plusSeconds(long seconds)                      | 返回此ZonedDateTime添加指定的秒数的副本。                    |
| ZonedDateTime plusWeeks(long weeksToAdd)                     | 返回此ZonedDateTime添加指定的周数的副本。                    |
| ZonedDateTime plusYears(long yearsToAdd)                     | 返回此ZonedDateTime添加指定的年数的副本。                    |
| R query(TemporalQuery query)                                 | 使用指定的查询查询此日期时间。                               |
| ValueRange range(TemporalField field)                        | 获取指定字段的有效值范围。                                   |
| LocalDate toLocalDate()                                      | 获取此日期时间的LocalDate部分。                              |
| LocalTime toLocalTime()                                      | 获取此日期时间的LocalTime部分。                              |
| OffsetDateTime toOffsetDateTime()                            | 将此日期时间转换为OffsetDateTime。                           |
| String toString()                                            | 将此日期输出为String，例如:"2007-12-03T10:15:30+01:00[Europe/Paris]" |
| ZonedDateTime truncatedTo(TemporalUnit unit)                 | 返回此ZonedDateTime截断时间的副本。                          |
| long until(Temporal endExclusive, TemporalUnit unit)         | 根据指定的单位计算到另一个日期时间的时间量。                 |
| ZonedDateTime with(TemporalAdjuster adjuster)                | 返回此日期时间的调整副本。                                   |
| ZonedDateTime with(TemporalField field, long newValue)       | 返回此日期时间将指定字段设置为新值的副本。                   |
| ZonedDateTime withDayOfMonth(int dayOfMonth)                 | 返回此ZonedDateTime更改日期的副本。                          |
| ZonedDateTime withDayOfYear(int dayOfYear)                   | 返回此ZonedDateTime包含日期更改的副本。                      |
| ZonedDateTime withEarlierOffsetAtOverlap()                   | 返回此日期时间将区域偏移更改为本地时间线重叠处的两个有效偏移中较早的一个的副本。 |
| ZonedDateTime withFixedOffsetZone()                          | 返回此日期时间将区域ID设置为偏移量的副本。                   |
| ZonedDateTime withHour(int hour)                             | 返回此ZonedDateTime更改日期的副本。                          |
| ZonedDateTime withLaterOffsetAtOverlap()                     | 返回此日期时间，将区域偏移更改为本地时间线重叠处的两个有效偏移中的较晚者的副本。 |
| ZonedDateTime withMinute(int minute)                         | 返回此ZonedDateTime更改了分钟的副本。                        |
| ZonedDateTime withMonth(int month)                           | 返回此ZonedDateTime已更改年份的副本。                        |
| ZonedDateTime withNano(int nanoOfSecond)                     | 返回此ZonedDateTime更改了纳秒的副本。                        |
| ZonedDateTime withSecond(int second)                         | 返回此ZonedDateTime更改秒钟的副本。                          |
| ZonedDateTime withYear(int year)                             | 返回此ZonedDateTime更改年份后的副本。                        |
| ZonedDateTime withZoneSameInstant(ZoneId zone)               | 使用不同的时区返回此日期时间保留该瞬间的副本。               |
| ZonedDateTime withZoneSameLocal(ZoneId zone)                 | 使用不同的时区返回此日期时间的副本，如果可能保留本地日期时间。 |



### 3、ZonedDateTime 类示例

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public class JavaDateTest {
    public static void main(String[] args) {
        System.out.println("系统的默认时区编号" + ZoneId.systemDefault()); // 系统的默认时区编号Asia/Shanghai
        System.out.println("本地时区的日期时间:" + LocalDateTime.now()); // 本地时区的日期时间:2022-02-18T19:17:37.416345200

        /**
         * 实例化 ZonedDateTime 对象：
         * 一种是通过now()方法返回当前时间，一种是通过of()方法放回指定时间。对会带上时区ZoneId对象
         *   ZonedDateTime now()
         *   ZonedDateTime now(ZoneId zone)
         *   ZonedDateTime now(Clock clock)
         *   ZonedDateTime of(LocalDate date, LocalTime time, ZoneId zone)
         *   ZonedDateTime of(LocalDateTime localDateTime, ZoneId zone)
         *   ZonedDateTime of(int year, int month, int dayOfMonth,int hour, int minute, int second, int nanoOfSecond, ZoneId zone)
         */
        ZonedDateTime zbj1 = ZonedDateTime.now(); // 默认时区
        ZonedDateTime zny1 = ZonedDateTime.now(ZoneId.of("America/New_York")); // 指定时区
        ZonedDateTime zdf1 = ZonedDateTime.now(Clock.systemDefaultZone()); // 默认时区
        System.out.println(zbj1); // 2022-02-18T20:41:14.164538200+08:00[Asia/Shanghai]
        System.out.println(zny1); // 2022-02-18T07:41:14.164538200-05:00[America/New_York]
        System.out.println(zdf1); // 2022-02-18T20:41:14.164538200+08:00[Asia/Shanghai]

        /**
         * 另一种方式是通过给一个LocalDateTime附加一个ZoneId，就可以变成ZonedDateTime：
         */
        LocalDateTime ldt = LocalDateTime.of(2019, 9, 15, 15, 16, 17);
        ZonedDateTime zbj2 = ldt.atZone(ZoneId.systemDefault());
        ZonedDateTime zny2 = ldt.atZone(ZoneId.of("America/New_York"));
        // 以这种方式创建的ZonedDateTime，它的日期和时间与LocalDateTime相同，但附加的时区不同，因此是两个不同的时刻：
        System.out.println(zbj2); // 2019-09-15T15:16:17+08:00[Asia/Shanghai]
        System.out.println(zny2); // 2019-09-15T15:16:17-04:00[America/New_York]

        /**
         * 对比LocalDateTime和ZonedDateTime都设置时区的情况
         */
        // 根据时区，获得指定时区的当前时间（可以理解为还是本地时区，所以输出不显示时区）
        LocalDateTime localDateTime = LocalDateTime.now(ZoneId.of("America/Phoenix"));
        System.out.println(localDateTime); // 2022-02-18T04:17:37.431349500
        // 获取指定时区，获得指定时区的当前时间（这个是把当前时间指定到固定的时区）
        ZonedDateTime zonedDateTime = LocalDateTime.now().atZone(ZoneId.of("Europe/Monaco"));
        System.out.println(zonedDateTime); // 2022-02-18T19:17:37.416345200+01:00[Europe/Monaco]

        /**
         * 时区转换:
         * 要转换时区,通过ZonedDateTime的withZoneSameInstant()将关联时区转换到另一个时区,转换后日期和时间都会相应调整。
         * 下面的代码演示了如何将北京时间转换为纽约时间：
         * 要特别注意，时区转换的时候，由于夏令时的存在，不同的日期转换的结果很可能是不同的。这是北京时间9月15日的转换结果：
         * 下面两次转换后的纽约时间有1小时的夏令时时差（ 涉及到时区时，千万不要自己计算时差，否则难以正确处理夏令时）
         */
        // 设置中国时区时间9月15日:
        ZonedDateTime zbj3 = ZonedDateTime.of(2020, 9, 15, 15, 16, 17, 00, ZoneId.of("Asia/Shanghai"));
        // 转换为纽约时间:
        ZonedDateTime zny3 = zbj3.withZoneSameInstant(ZoneId.of("America/New_York"));
        System.out.println(zbj3); // 2020-09-15T15:16:17+08:00[Asia/Shanghai]
        System.out.println(zny3); // 2020-09-15T03:16:17-04:00[America/New_York]

        // 设置中国时区时间11月15日:
        ZonedDateTime zbj4 = ZonedDateTime.of(2020, 11, 15, 15, 16, 17, 00, ZoneId.of("Asia/Shanghai"));
        // 转换为纽约时间:
        ZonedDateTime zny4 = zbj4.withZoneSameInstant(ZoneId.of("America/New_York"));
        System.out.println(zbj4); // 2020-11-15T15:16:17+08:00[Asia/Shanghai]
        System.out.println(zny4); // 2020-11-15T02:16:17-05:00[America/New_York]


        /**
         * 访问与设置ZonedDateTime对象的时间（与LocalDateTime用法基本一致, 案例直接参考LocalDateTime）
         *
         * 1.常用的getXxx()系列操作，获得日期：
         *   int getYear() 获取当前日期的年份
         *   Month getMonth() 获取当前日期的月份对象(返回一个Month枚举值)
         *   int getMonthValue() 获取当前日期是第几月(1-12)
         *   int getDayOfMonth() 表示该对象表示的⽇期是这个⽉第⼏天(1-31)
         *   DayOfWeek getDayOfWeek() 表示该对象表示的日期是星期几(返回一个DayOfWeek枚举值)
         *   int getDayOfYear() 表示该对象表示的日期是今年第几天(1-366)
         *   int getHour() 获取当前时间小时数
         *   int getMinute() 获取当前时间分钟数
         *   int getSecond() 获取当前时间秒值
         *   int getNano() 把获取到的当前时间的秒数换算成纳秒
         *
         * 2.常用的setXxx()系列操作，设置日期：
         *   ZonedDateTime withYear(int year) 指定对象的年份
         *   ZonedDateTime withMonth(int month) 指定对象的月份
         *   ZonedDateTime withDayOfMonth(int dayOfMonth) 指定对象在当月的日期
         *   ZonedDateTime withDayOfYear(int dayOfYear) 指定对象在当年的日期
         *   ZonedDateTime withHour(int hour) 指定对象的小时数
         *   ZonedDateTime withMinute(int minute) 指定对象的分钟数
         *   ZonedDateTime withSecond(int second) 指定对象在当⽉的秒数
         *   ZonedDateTime withNano(int nanoOfSecond) 指定对象在当年的纳秒数
         *
         * 3.常用的plusXxx()系列操作，增加时间的方法：
         *   ZonedDateTime plusYears(long years) 增加指定年份数
         *   ZonedDateTime plusMonths(long months) 增加指定⽉份数
         *   ZonedDateTime plusDays(long days) 增加指定天数
         *   ZonedDateTime plusWeeks(long weeks) 增加指定周数
         *   ZonedDateTime plusHours(long hours) 增加指定小时
         *   ZonedDateTime plusMinutes(long minutes) 增加指定分钟
         *   ZonedDateTime plusSeconds(long seconds) 增加指定秒
         *   ZonedDateTime plusNanos(long nanos) 增加指定纳秒
         *
         * 4.常用的minusXxx()系列操作，减少时间的方法:
         *   ZonedDateTime minusYears(long years) 减去指定年份
         *   ZonedDateTime minusMonths(long months) 减去指定月份
         *   ZonedDateTime minusDays(long days) 减去指定天数
         *   ZonedDateTime minusWeeks(long weeks) 减去指定周数
         *   ZonedDateTime minusHours(long hours) 减去指定小时
         *   ZonedDateTime minusMinutes(long minutes) 减去指定分钟
         *   ZonedDateTime minusSeconds(long seconds) 减去指定秒
         *   ZonedDateTime minusNanos(long nanos) 减去指定纳秒
         */
        // 案例直接参考LocalDateTime
    }
}
```

LocalDateTime.now()也是获取默认时区的当前日期和时间，有什么区别呢？LocalDateTime内部不会记录时区信息，只会单纯记录年月日时分秒等信息，而ZonedDateTime除了记录日历信息，还会记录时区，它的其他大部分构建方法都需要显式传递时区，比如：

```java
// 根据Instant和时区构建ZonedDateTime
public static ZonedDateTime ofInstant(Instant instant, ZoneId zone);

// 根据LocalDate, LocalTime和ZoneId构造
public static ZonedDateTime of(LocalDate date, LocalTime time, ZoneId zone);
```

ZonedDateTime可以直接转换为Instant，比如：

```java
ZonedDateTime ldt = ZonedDateTime.now();
Instant now = ldt.toInstant();
```



## JSR 310 常用计算工具

Java 8 JSR-310 中提供了几个常用于计算的类：

- java.timeDuration：表示秒和纳秒的时间量。
- java.timePeriod：表示年月日的时间量。
- java.time.temporal.TemporalUnit：日期时间的基本单位。
- java.time.temporal.TemporalField：日期时间的属性。
- java.time.temporal.ValueRange：表示取值范围。
- java.time.temporal.TemporalAdjuster：日期矫正器。
- java.time.chrono.Chronology：判断是否闰年



## java.time.Duration

Java 8 中提供了两个工具类 Duration 与 Period：计算日期时间差。它们最大的作用就不需要你自己复杂的计算关于两个年月日之间的相差的时间或日期啦。

- Period ：基于日期值，对应使用 LocalDate ，它们的作用范围域都是日期（年/月/日）
- Duration：基于时间值，对应使用 Instant /LocalTime/LocalDateTime（不包含LocalDate，会报异常），它们的作用范围域都是时间（天/时/分/秒/毫秒/纳秒）注意：Duration 不包含毫秒这个属性

### 1、Duration 类说明

Duration 类用于以秒和纳秒为单位测量时间（Duration 是 TemporalAmount 的实现类，类里包含两个变量 seconds 和 nanos ，所以 Duration 是由秒和纳秒组成的时间量）。Duration 适合处理较短的时间，需要更高的精确性。

它继承了 Object 类并实现 Comparable 接口。类声明如下：

```java
public final class Duration extends Object implements TemporalAmount, Comparable<Duration>, Serializable {}
```



### 2、Duration 类方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Duration abs()                                               | 返回此持续时间的副本，长度为正。                             |
| Temporal addTo(Temporal temporal)                            | 将此持续时间添加到指定的时间对象。                           |
| static Duration between(Temporal startInclusive, Temporal endExclusive) | 获得一个Duration，表示两个时间对象之间的持续时间。           |
| int compareTo(Duration otherDuration)                        | 将此持续时间与指定的持续时间进行比较。                       |
| Duration dividedBy(long divisor)                             | 返回此持续时间的副本除以指定值。                             |
| boolean equals(Object otherDuration)                         | 检查此持续时间是否等于指定的持续时间。                       |
| static Duratio from(TemporalAmount amount)                   | 从时间量获取Duration的实例。                                 |
| long get(TemporalUnit unit)                                  | 获取所请求单位的值。                                         |
| int getNano()                                                | 获取此持续时间中每秒内的纳秒数。                             |
| long getSeconds()                                            | 获取此持续时间的秒数。                                       |
| List getUnits()                                              | 获取此持续时间支持的单位集。                                 |
| int hashCode()                                               | 此持续时间的哈希码。                                         |
| boolean isNegative()                                         | 检查此持续时间是否为负（不包括零）。                         |
| boolean isZero()                                             | 检查此持续时间是否为零长度。                                 |
| Duration minus(Duration duration)                            | 返回此持续时间的副本，减去指定的持续时间。                   |
| Duration minus(long amountToSubtract, TemporalUnit unit)     | 返回此持续时间的副本，减去指定的持续时间。                   |
| Duration minusDays(long daysToSubtract)                      | 返回此持续时间的副本，并减去标准24小时内的指定持续时间。     |
| Duration minusHours(long hoursToSubtract)                    | 返回此持续时间的副本，其中减去指定的持续时间（以小时为单位）。 |
| Duration minusMillis(long millisToSubtract)                  | 返回此持续时间的副本，其中减去指定的持续时间（以毫秒为单位）。 |
| Duration minusMinutes(long minutesToSubtract)                | 返回此持续时间的副本，其中减去指定的持续时间（以分钟为单位）。 |
| Duration minusNanos(long nanosToSubtract)                    | 返回此持续时间的副本，其中减去指定的持续时间（以纳秒为单位）。 |
| Duration minusSeconds(long secondsToSubtract)                | 返回此持续时间的副本，减去指定的持续时间（以秒为单位）。     |
| Duration multipliedBy(long multiplicand)                     | 返回此持续时间乘以标量的副本。                               |
| Duration negated()                                           | 返回此持续时间的副本，长度取反。                             |
| static Duration of(long amount, TemporalUnit unit)           | 获取一个以指定单位表示金额的持续时间。                       |
| static Duration ofDays(long days)                            | 获得一个持续时间，表示多个标准的24小时工作日。               |
| static Duration ofHours(long hours)                          | 获得一个代表多个标准小时数的持续时间。                       |
| static Duration ofMillis(long millis)                        | 获得一个表示毫秒数的持续时间。                               |
| static Duration ofMinutes(long minutes)                      | 获得一个代表多个标准分钟的持续时间。                         |
| static Duration ofNanos(long nanos)                          | 获得一个持续时间，表示多个纳秒。                             |
| static Duration ofSeconds(long seconds)                      | 获得一个表示秒数的持续时间。                                 |
| static Duration ofSeconds(long seconds, long nanoAdjustment) | 获得一个表示秒数的持续时间，并以纳秒为单位进行调整。         |
| static Duration parse(CharSequence text)                     | 从文本字符串（例如PnDTnHnMn.nS）获取持续时间。               |
| Duration plus(Duration duration)                             | 返回此持续时间的副本，其中添加了指定的持续时间。             |
| Duration plus(long amountToAdd, TemporalUnit unit)           | 返回此持续时间的副本，其中添加了指定的持续时间。             |
| Duration plusDays(long daysToAdd)                            | 返回此持续时间的副本，并添加指定的持续时间（标准24小时）。   |
| Duration plusHours(long hoursToAdd)                          | 返回此持续时间的副本，并添加指定的持续时间（以小时为单位）。 |
| Duration plusMillis(long millisToAdd)                        | 返回此持续时间的副本，并添加指定的持续时间（以毫秒为单位）。 |
| Duration plusMinutes(long minutesToAdd)                      | 返回此持续时间的副本，并添加指定的持续时间（以分钟为单位）。 |
| Duration plusNanos(long nanosToAdd)                          | 返回此持续时间的副本，并添加指定的持续时间（以纳秒为单位）。 |
| Duration plusSeconds(long secondsToAdd)                      | 返回此持续时间的副本，并添加指定的持续时间（以秒为单位）。   |
| Temporal subtractFrom(Temporal temporal)                     | 从指定的时间对象中减去此持续时间。                           |
| long toDays()                                                | 获取此持续时间中的天数。                                     |
| long toHours()                                               | 获取此持续时间中的小时数。                                   |
| long toMillis()                                              | 将此持续时间转换为总长度（以毫秒为单位）。                   |
| long toMinutes()                                             | 获取此持续时间中的分钟数。                                   |
| long toNanos()                                               | 将此持续时间转换为表示为long的总长度（以纳秒为单位）。       |
| String toString()                                            | 此持续时间的字符串表示形式，使用基于ISO-8601秒的表示形式，例如PT8H6M12.345S。 |
| Duration withNanos(int nanoOfSecond)                         | 返回此持续时间的副本，以指定的纳秒为单位。                   |
| Duration withSeconds(long seconds)                           | 返回此持续时间的副本，具有指定的秒数。                       |

Duration 类常用方法归类总结：

1. static between(to, from)：计算两个时间的间隔，默认是秒
2. static ofDays()、ofHours()、ofMinutes()、ofSeconds()、ofMillis()、ofNanos()：计算指定时间段
3. static of(int amount, ChronoUnit.Xxx) 计算指定时间段
4. static from(TemporalAmount amount)：从时间量获取Duration的实例
5. long toDays()、toHours()、toMinutes()、toSeconds()、toMillis()、toNanos()：将时间转换为以天/时/分/秒/毫秒/纳秒为单位
6. int toXxxPart()：跟如上toXxx方法基本一样，区别就是返回类型由long变成了int，个人认为可能是节约资源
7. getXxx()：获取当前对象时间中某个单位的值
8. withXxx()：设置好当前时间段的固定的时间
9. plusXxx()：给当前时间段加上固定的时间
10. minusXxx()：给当前时间段减少固定的时间



### 3、Duration 类示例

```java
import java.time.Duration;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 实例化指定单位的持续时间对象
         * 注意: 默认打印为ISO国际标准化组织规定的日期格式: PT2H中 H表示Hour小时, M代表Minute分钟，S代表Second秒数
         */
        Duration durationDays1 = Duration.ofDays(10); // 10天
        Duration durationDays2 = Duration.of(10, ChronoUnit.DAYS); // 10天
        System.out.println(durationDays1); // PT240H
        System.out.println(durationDays2); // PT240H
        Duration durationHours1 = Duration.ofHours(1); // 1小时
        Duration durationHours2 = Duration.of(1, ChronoUnit.HOURS); // 1小时
        System.out.println(durationHours1); // PT1H
        System.out.println(durationHours2); // PT1H
        Duration durationMinutes1 = Duration.ofMinutes(1); // 1分
        Duration durationMinutes2 = Duration.of(1, ChronoUnit.MINUTES); // 1分
        System.out.println(durationMinutes1); // PT1M
        System.out.println(durationMinutes2); // PT1M
        Duration durationSeconds1 = Duration.ofSeconds(1); // 1秒
        Duration durationSeconds2 = Duration.of(1, ChronoUnit.SECONDS); // 1秒
        System.out.println(durationSeconds1); // PT1S
        System.out.println(durationSeconds2); // PT1S
        Duration durationMillis1 = Duration.ofMillis(1000); // 1000毫秒
        Duration durationMillis2 = Duration.of(1000, ChronoUnit.MILLIS); // 1000毫秒
        System.out.println(durationMillis1); // PT1S
        System.out.println(durationMillis2); // PT1S
        Duration durationNanos1 = Duration.ofNanos(100000000); // 10000000纳秒
        Duration durationNanos2 = Duration.of(100000000, ChronoUnit.NANOS); // 10000000纳秒
        System.out.println(durationNanos1); // PT0.1S
        System.out.println(durationNanos2); // PT0.1S
        Duration durationFrom = Duration.from(ChronoUnit.DAYS.getDuration());
        System.out.println(durationFrom); // PT24H

        /**
         * 转换成指定单位的持续时间
         *   long toDays() 这段时间的总天数
         *   long toHours() 这段时间的小时数
         *   long toMinutes() 这段时间的分钟数
         *   long toSeconds() 这段时间的秒数
         *   long toMillis() 这段时间的毫秒数
         *   long toNanos() 这段时间的纳秒数
         *   String toString() 此持续时间的字符串表示形式,使用基于ISO-8601秒*的表示形式,例如 PT8H6M12.345S
         */
        Duration durationOne = Duration.ofDays(1); // 设置1天的时间
        System.out.println("toDay天 = "+ durationOne.toDays()); // toDay时间 = 1
        System.out.println("toHours时 = "+ durationOne.toHours()); // toHours时间 = 24
        System.out.println("toMinutes分 = "+ durationOne.toMinutes()); // toMinutes时间 = 1440
        System.out.println("toMinutes秒 = "+ durationOne.toSeconds()); // toMinutes秒 = 86400
        System.out.println("toMillis毫秒 = "+ durationOne.toMillis()); // toMillis时间 = 86400000
        System.out.println("toNanos纳秒 = "+ durationOne.toNanos()); // toNanos时间 = 86400000000000
        System.out.println("toString格式 = "+ durationOne.toString()); // toString时间 = PT24H

        /**
         * 获取2个时间点之间差值的持续时间
         * Duration.between()方法比较两个瞬间的差, 注意这个天数是可以负数
         */
        LocalDateTime from = LocalDateTime.of(2017, 01, 1, 00, 0, 0);   // 2017-01-01 00:00:00
        LocalDateTime to = LocalDateTime.of(2019, 9, 12, 14, 28, 0); // 2019-09-12 14:28:00
        Duration duration1 = Duration.between(from, to);  // 表示从 from 到 to 这段时间（第⼆个参数减第⼀个参数）
        System.out.println(duration1.toDays()); // 984
        System.out.println(duration1.toHours()); // 23630
        System.out.println(duration1.toMinutes()); // 1417828
        System.out.println(duration1.getSeconds()); // 85069680
        System.out.println(duration1.toMillis()); // 85069680000
        System.out.println(duration1.toNanos()); // 85069680000000000

        /**
         * Duration时间的加减，可以参考LocalDateTime中的plusXxx、minusXxx 和 withXxx() 系列的方法
         * Duration 包含两部分：seconds秒，nanos纳秒
         * 它们的组合表达了时间长度。所以withXxx()只有withSeconds()和withNanos()方法
         */
        System.out.println(Duration.ofDays(4).withSeconds(360).toHours()); // 加8小时(4天8小时)：输出：0
        System.out.println(Duration.ofDays(4).plusHours(8).toHours()); // 加8小时(4天8小时)：输出：104
        System.out.println(Duration.ofDays(4).minusHours(8).toHours()); // 加8小时(4天8小时)：输出：88

        /**
         * Duration可以接收：LocalDate、LocalTime、LocalDateTime、Instant
         * Duration只能处理两个Instant、LocalTime, LocalDateTime, ZonedDateTime,
         * 参数不能混搭,混搭会报异常，如果传入的是LocalDate，将会抛出异常
         */
        Duration.between(LocalTime.now(), LocalTime.now());
        Duration.between(LocalDateTime.now(), LocalDateTime.now());
        Duration.between(Instant.now(), Instant.now());
    }
}
```



## java.time.Period

### 1、Period 类说明

Period 在概念上和 Duration 类似，区别在于 Period 是以年月日来衡量一个时间段（Period 是 ChronoPeriod 的实现类，类里包含两个变量 years、months 和 days ，所以 Period 是由年，月和日组成的时间量），Period 所有日期单位都被认为是估计的（非精确的）。

Period 它继承了 Object 类并实现了 ChronoPeriod 接口。类的声明如下：

```java
public final class Period extends Object implements ChronoPeriod, Serializable {}
```



### 2、Period 类方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Temporal addTo(Temporal temporal)                            | 将此Period添加到指定的时态对象                               |
| static Period between(LocalDate startInclusive, LocalDate endExclusive) | 获得一个包含两个日期之间的年数，月数和天数的Period。         |
| boolean equals(Object otherPeriod)                           | 检查此Period是否等于指定的Period。                           |
| static Duratio from(TemporalAmount amount)                   | 从时间量获得Period的实例。                                   |
| long get(TemporalUnit unit)                                  | 获取所请求单元的值。                                         |
| IsoChronology getChronology()                                | 获取此Period的年表，即ISO日历系统。                          |
| int getDays()                                                | 获取此Period的天数。                                         |
| int getMonths()                                              | 获取此Period的月数。                                         |
| List getUnits()                                              | 获取此Period支持的单位集。                                   |
| int hashCode()                                               | 此Period的哈希码。                                           |
| boolean isNegative()                                         | 检查此期间是否为负数，不包括零。                             |
| boolean isZero()                                             | 检查此Period是否为零长度。                                   |
| Period minus(TemporalAmount amountToSubtract)                | 返回此Period的副本，并减去指定的Period。                     |
| Period minusDays(long daysToSubtract)                        | 返回此Period的副本，并减去指定的天数。                       |
| Period minusMonths(long months)                              | 返回此Period的副本，并减去指定的月份。                       |
| Period minusYears(long years)                                | 返回此Period的副本，并减去指定的年份。                       |
| Period multipliedBy(long multiplicand)                       | 返回此Period的副本乘以标量。                                 |
| Period negated()                                             | 返回此Period的副本，其长度为negated。                        |
| Period normalized()                                          | 返回此Period的副本，其中年份和月份已标准化。                 |
| static Period of(int years, int months, int days)            | 获得表示若干年，月和日的时段。                               |
| static Period ofDays(int days)                               | 获得表示天数的Period。                                       |
| static Period ofMonths(int months)                           | 获得表示若干月份的Period。                                   |
| static Period ofWeeks(int weeks)                             | 获得代表若干周的Period。                                     |
| static Period ofYears(int years)                             | 获得代表若干周的Period。                                     |
| static Period parse(CharSequence text)                       | 从文本字符串(如PnYnMnD)获取Period。                          |
| Period plus(TemporalAmount amountToAdd)                      | 返回此Period的副本，并添加指定的Period。                     |
| Period plusDays(long daysToAdd)                              | 返回此Period的副本，并添加指定的天数。                       |
| Period plusMonths(long monthsToAdd)                          | 返回此Period的副本，并添加指定的月份。                       |
| Period plusYears(long yearsToAdd)                            | 返回此Period的副本，并添加指定年份。                         |
| Temporal subtractFrom(Temporal temporal)                     | 从指定的时态对象中减去此Period。                             |
| String toString()                                            | 使用基于ISO-8601秒的表示形式的此Period的字符串表示形式，例如PT8H6M12.345S。 |
| long toTotalMonths()                                         | 获取此Period的总月数。                                       |
| Period withDays(int days)                                    | 返回具有指定天数的此Period的副本。                           |
| Period withMonths(int months)                                | 返回具有指定月份数的此Period的副本。                         |
| Period withYears(int years)                                  | 返回具有指定年数的此Period的副本。                           |

Period 类常用 API：

- static between()：计算两个日期之间的间隔获取Period实例
- static ofYears()、ofMonths()、ofDays()、ofWeeks()：计算指定时间段获取Period实例
- static of(int years, int months, int days)：计算指定时间段获取Period实例
- 静态方法 from(TemporalAmount amount)：从时间量获取Period的实例
- getYears()、getMonths()、getDays()：获取年份、月份、天数
- withXxx()：设置好当前时间段的固定的时间
- plusXxx()：给当前时间段加上固定的时间
- minusXxx()：给当前时间段减少固定的时间

注意：Duration 用于计算两个时间间隔，Period 用于计算两个日期间隔，所以 between() 方法只能接收 LocalDate 类型的参数。



### 3、Period 类示例

```java
import java.time.Duration;
import java.time.LocalDate;
import java.time.Period;
import java.time.temporal.ChronoUnit;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 实例化指定单位的持续日期对象
         * 注意: 默认的打印结果为ISO国际标准化组织规定的日期格式，P3Y2M1D中的，Y表示Year年，M代表Month月，D代表Day天
         */
        System.out.println(Period.between(LocalDate.now(), LocalDate.now())); // P0D
        System.out.println(Period.of(1,2,3)); // P1Y2M3D
        System.out.println(Period.ofYears(1)); // P1Y
        System.out.println(Period.ofMonths(2)); // P2M
        System.out.println(Period.ofDays(25)); // P25D
        System.out.println(Period.ofWeeks(4)); // P28D
        System.out.println(Period.from(Period.of(3, 2, 1))); // P3Y2M1D

        /**
         * 获取指定单位的持续时间
         */
        Period periodYears = Period.ofYears(1); // 设置1年的时间
        System.out.println(periodYears.getYears()); // 1
        System.out.println(periodYears.getMonths()); // 0
        System.out.println(periodYears.getDays()); // 0
        System.out.println(periodYears.get(ChronoUnit.YEARS)); // 1
        System.out.println(periodYears.get(ChronoUnit.MONTHS)); // 0
        System.out.println(periodYears.get(ChronoUnit.DAYS)); // 0
        System.out.println(periodYears.getChronology()); // 获取此Period的年表，即ISO日历系统：ISO
        System.out.println(periodYears.getUnits()); // 查看支持的枚举类型：[Years, Months, Days]

        /**
         * Period时间的加减，可以参考LocalDateTime中的plusXxx、minusXxx和withXxx()系列的方法
         */
        System.out.println(Period.of(1,2,3).withYears(8).getYears()); // 3
        System.out.println(Period.of(1,2,3).withMonths(8).getMonths()); // 3
        System.out.println(Period.of(1,2,3).withDays(8).getDays()); // 8
        System.out.println(Period.of(1,2,3).plusYears(8).getYears()); // 3
        System.out.println(Period.of(1,2,3).plusMonths(8).getMonths()); // 3
        System.out.println(Period.of(1,2,3).plusDays(8).getDays()); // 11
        System.out.println(Period.of(1,2,3).minusYears(8).getYears()); // 3
        System.out.println(Period.of(1,2,3).minusMonths(8).getMonths()); // 3
        System.out.println(Period.of(1,2,3).minusDays(8).getDays()); // -5
        
        /**
         * Period获取2个时间点之间差值的持续时间
         */
        LocalDate from = LocalDate.of(2017, 01, 01); // 2017-01-01
        LocalDate to = LocalDate.of(2019, 9, 12); // 2019-09-12
        Period period = Period.between(from, to);  // 表示从 from 到 to 这段时间（第⼆个参数减第⼀个参数）
        System.out.println(period); // P2Y8M11D
        System.out.println(period.getDays()); // 11
        System.out.println(period.getMonths()); // 8
        System.out.println(period.getYears()); // 2
        System.out.println(period.get(ChronoUnit.YEARS)); // 2
        System.out.println(period.getUnits()); // [Years, Months, Days]
        System.out.println(period.getChronology()); // ISO
    }
}
```



## java.time.temporal.TemporalUnit【接口】

### 1、TemporalUnit 与 ChronoUnit 类说明

TemporalUnit 是一个**时间字段单位**的接口，我们主要用它的实现类 ChronoUnit【枚举】，里面定义了很多表示日期/时间的枚举值，提供基于单元的访问来操纵日期，时间或日期时间。

```java
public interface TemporalUnit {
    Duration getDuration();
    boolean isDurationEstimated();
    boolean isDateBased();
    boolean isTimeBased();
    default boolean isSupportedBy(Temporal temporal) {
        // ...
    }
    <R extends Temporal> R addTo(R temporal, long amount);
    long between(Temporal temporal1Inclusive, Temporal temporal2Exclusive);
    @Override
    String toString();
}
```

每个 ChronoUnit 会维护一个 name 和一个 Duration 类型的实例。其中 ChronoUnit 枚举了标准的日期时间单位集合，就是常用的年、月、日、小时、分钟、秒、毫秒、微秒、纳秒，**这些时间单位的时间量到底是多少，代表多长的时间**，在该枚举类中都有定义。

```java
public enum ChronoUnit implements TemporalUnit {
    NANOS("Nanos", Duration.ofNanos(1)), // 纳秒
    MICROS("Micros", Duration.ofNanos(1000)), // 微秒
    MILLIS("Millis", Duration.ofNanos(1000_000)), // 毫秒
    SECONDS("Seconds", Duration.ofSeconds(1)), // 秒
    MINUTES("Minutes", Duration.ofSeconds(60)), // 分钟
    HOURS("Hours", Duration.ofSeconds(3600)), // 小时
    HALF_DAYS("HalfDays", Duration.ofSeconds(43200)), // 半天
    DAYS("Days", Duration.ofSeconds(86400)), // 天
    WEEKS("Weeks", Duration.ofSeconds(7 * 86400L)), // 周
    MONTHS("Months", Duration.ofSeconds(31556952L / 12)),  // 月
    YEARS("Years", Duration.ofSeconds(31556952L)), // 年
    DECADES("Decades", Duration.ofSeconds(31556952L * 10L)), // 十年
    CENTURIES("Centuries", Duration.ofSeconds(31556952L * 100L)), // 世纪（即百年）
    MILLENNIA("Millennia", Duration.ofSeconds(31556952L * 1000L)), // 世纪（即百年）
    ERAS("Eras", Duration.ofSeconds(31556952L * 1000_000_000L)), // 十亿年
    FOREVER("Forever", Duration.ofSeconds(Long.MAX_VALUE, 999_999_999)); // 月

    private final String name;
    private final Duration duration;
    private ChronoUnit(String name, Duration estimatedDuration) {
        this.name = name;
        this.duration = estimatedDuration;
    }
}
```

| Enum Constant | 描述                                       |
| ------------- | ------------------------------------------ |
| CENTURIES     | 代表一个世纪概念的单位。                   |
| DAYS          | 代表一天概念的单位。                       |
| DECADES       | 代表十年概念的单位。                       |
| ERAS          | 代表一个时代概念的单位。                   |
| FOREVER       | 代表永恒概念的人工单位。                   |
| HALF_DAYS     | 代表AM / PM中使用的半天概念的单位。        |
| HOURS         | 表示一小时概念的单位。                     |
| MICROS        | 表示微秒概念的单位。                       |
| MILLENNIA     | 代表千年概念的单位。                       |
| MILLIS        | 表示毫秒概念的单位。                       |
| MINUTES       | 表示一分钟概念的单位。                     |
| MONTHS        | 代表一个月概念的单位。                     |
| NANOS         | 代表纳秒概念的单位，是支持的最小时间单位。 |
| SECONDS       | 秒                                         |
| MONTHS        | 代表一个月概念的单位。                     |
| NANOS         | 代表纳秒概念的单位，是支持的最小时间单位。 |
| SECONDS       | 秒                                         |



### 2、ChronoUnit 类方法及示例

| 方法                                   | 描述                                             |
| :------------------------------------- | :----------------------------------------------- |
| Duration getDuration()                 | 获取ISO日历系统中此单元的估计持续时间。          |
| boolean isDateBased()                  | 检查此单位是否为日期单位。                       |
| boolean isDurationEstimated()          | 检查单位的持续时间是否为估计值。                 |
| boolean isTimeBased()                  | 检查此单位是否为时间单位。                       |
| static ChronoUnit valueOf(String name) | 返回具有指定名称的此类型的枚举常量。             |
| static ChronoUnit[] values()           | 按照声明的顺序返回一个包含此枚举类型常量的数组。 |

```java
// valueOf()
ChronoUnit chronoUnit = ChronoUnit.valueOf("DAYS");
System.out.println(chronoUnit); // Days

// values()
ChronoUnit[] chronoUnits = ChronoUnit.values();
System.out.println(Arrays.toString(chronoUnits));
// [Nanos, Micros, Millis, Seconds, Minutes, Hours, HalfDays, Days, Weeks, Months, Years, Decades, Centuries, Millennia, Eras, Forever]

// getDuration()
Duration getDurationAttribute = chronoUnit.getDuration();
System.out.println(getDurationAttribute); // PT24H

// isDurationEstimated() ChronoUnit 中的所有时间单位都被认为是准确的,而所有日期单位都被认为是估计的。
System.out.println(ChronoUnit.DAYS.isDurationEstimated());  // true
System.out.println(ChronoUnit.NANOS.isDurationEstimated()); // false

// isDateBased() 用于检查此ChronoUnit是否为日期单位。从天到时代的所有ChronoUnit都是基于日期的。
System.out.println(ChronoUnit.DAYS.isDateBased());  // true
System.out.println(ChronoUnit.NANOS.isDateBased()); // false

// isTimeBased() 用于检查此ChronoUnit是否为时间单位。从nanos到half-days的所有ChronoUnit都是基于时间的。
System.out.println(ChronoUnit.DAYS.isTimeBased());  // false
System.out.println(ChronoUnit.NANOS.isTimeBased()); // true

LocalDateTime today     = LocalDateTime.now(); // now: 2023-05-26 11:49
LocalDateTime birthDate = LocalDateTime.of(2000, 5, 21, 0, 0, 0, 0);

System.out.println("相差年数: " + ChronoUnit.YEARS.between(birthDate,today)); // 相差年数: 23
System.out.println("相差月数: " + ChronoUnit.MONTHS.between(birthDate,today)); // 相差月数: 276
System.out.println("相差周数: " + ChronoUnit.WEEKS.between(birthDate,today)); // 相差周数: 1200
System.out.println("相差天数: " + ChronoUnit.DAYS.between(birthDate,today)); // 相差天数: 8405
System.out.println("相差时数: " + ChronoUnit.HOURS.between(birthDate, today)); // 相差时数: 201731
System.out.println("相差分数: " + ChronoUnit.MINUTES.between(birthDate, today)); // 相差分数: 12103910
System.out.println("相差秒数: " + ChronoUnit.SECONDS.between(birthDate,today)); // 相差秒数: 726234611
System.out.println("相差毫秒数: " + ChronoUnit.MILLIS.between(birthDate,today)); // 相差毫秒数: 726234611842
System.out.println("相差微秒数: " + ChronoUnit.MICROS.between(birthDate,today)); // 相差微秒数: 726234611842343
System.out.println("相差纳秒数: " + ChronoUnit.NANOS.between(birthDate, today)); // 相差纳秒数: 726234611842343400
System.out.println("相差半天数: " + ChronoUnit.HALF_DAYS.between(birthDate, today)); // 相差半天数: 16810
System.out.println("相差十年数: " + ChronoUnit.DECADES.between(birthDate,today)); // 相差十年数: 2
System.out.println("相差世纪有年数: " + ChronoUnit.CENTURIES.between(birthDate,today)); // 相差世纪有年数: 0
System.out.println("相差千年数: " + ChronoUnit.MILLENNIA.between(birthDate,today)); // 相差千年数: 0
System.out.println("相差纪元数: " + ChronoUnit.ERAS.between(birthDate,today)); // 相差纪元数: 0
```



## java.time.temporal.TemporalField【接口】

### 1、TemporalField 与 ChronoField 类说明

TemporalField 是一个**时间域（时间字段）**接口，表示时间字段，他的实现类为 ChronoField，也是一个枚举。

```java
public interface TemporalField {
    default String getDisplayName(Locale locale) {
        Objects.requireNonNull(locale, "locale");
        return toString();
    }
    TemporalUnit getBaseUnit();
    TemporalUnit getRangeUnit();
    ValueRange range();
    boolean isDateBased();
    boolean isTimeBased();
    boolean isSupportedBy(TemporalAccessor temporal);
    ValueRange rangeRefinedBy(TemporalAccessor temporal);
    long getFrom(TemporalAccessor temporal);
    <R extends Temporal> R adjustInto(R temporal, long newValue);
    default TemporalAccessor resolve(
            Map<TemporalField, Long> fieldValues,
            TemporalAccessor partialTemporal,
            ResolverStyle resolverStyle) {
        return null;
    }
    @Override
    String toString()
}
```

TemporalField 主要实现类是枚举类型 ChronoField。一个 ChronoField 成员会维护一个字符串名字属性 name、一个 TemporalUnit 的基础单位 baseUnit、一个 TemporalUnit 的表示范围的单位 rangeUnit 和一个 ValueRange 类型的 range 用于表示当前属性的范围。

```java
public enum ChronoField implements TemporalField {
    NANO_OF_SECOND("NanoOfSecond", NANOS, SECONDS, ValueRange.of(0, 999_999_999)), // 一秒钟的纳秒数
    SECOND_OF_MINUTE("SecondOfMinute", SECONDS, MINUTES, ValueRange.of(0, 59), "second"), // 一分钟的秒数
    MINUTE_OF_HOUR("MinuteOfHour", MINUTES, HOURS, ValueRange.of(0, 59), "minute"), // 一个小时的分钟数
    CLOCK_HOUR_OF_AMPM("ClockHourOfAmPm", HOURS, HALF_DAYS, ValueRange.of(1, 12)), // 一上午或者一下午有多少个小时
    CLOCK_HOUR_OF_DAY("ClockHourOfDay", HOURS, DAYS, ValueRange.of(1, 24)), // 一天的小时数
    AMPM_OF_DAY("AmPmOfDay", HALF_DAYS, DAYS, ValueRange.of(0, 1), "dayperiod"), // 上午还是下午
    DAY_OF_WEEK("DayOfWeek", DAYS, WEEKS, ValueRange.of(1, 7), "weekday"), // 一周的第几天
    DAY_OF_MONTH("DayOfMonth", DAYS, MONTHS, ValueRange.of(1, 28, 31), "day"), // 当前月的天数
    DAY_OF_YEAR("DayOfYear", DAYS, YEARS, ValueRange.of(1, 365, 366)), // 当前年的天数
    ALIGNED_WEEK_OF_MONTH("AlignedWeekOfMonth", WEEKS, MONTHS, ValueRange.of(1, 4, 5)), // 当前月的周数
    ALIGNED_WEEK_OF_YEAR("AlignedWeekOfYear", WEEKS, YEARS, ValueRange.of(1, 53)), // 当前年的周数
    MONTH_OF_YEAR("MonthOfYear", MONTHS, YEARS, ValueRange.of(1, 12), "month"), // 当前年的月数
    // ...
    ;

    private final TemporalUnit baseUnit;
    private final String name;
    private final TemporalUnit rangeUnit;
    private final ValueRange range;
    private final String displayNameKey;
    // ...
}
```

| Enum Constant                | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| ALIGNED_DAY_OF_WEEK_IN_MONTH | 一个月内对齐的星期几。                                       |
| ALIGNED_DAY_OF_WEEK_IN_YEAR  | 一年内对齐的星期几。                                         |
| ALIGNED_WEEK_OF_MONTH        | 一个月内对齐的一周。                                         |
| ALIGNED_WEEK_OF_YEAR         | 一年内对齐的一周。                                           |
| AMPM_OF_DAY                  | 当天的AM / PM，从0（AM）到1（PM）                            |
| CLOCK_HOUR_OF_AMPM           | 计算AM / PM内的小时数，从1到12.这是在标准的12小时模拟挂钟上观察到的小时数 |
| CLOCK_HOUR_OF_DAY            | 小时，从1到24.这是在24小时模拟挂钟上观察到的小时             |
| DAY_OF_MONTH                 | 天。当月第几天                                               |
| DAY_OF_WEEK                  | 星期几，例如星期二。                                         |
| DAY_OF_YEAR                  | 天。当年第几天                                               |
| EPOCH_DAY                    | 该纪元日，基于1970-01-01（ISO）的Java时代。                  |
| ERA                          | 时代。                                                       |
| HOUR_OF_AMPM                 | 计算AM / PM内的小时数，12小时制，从0到11                     |
| HOUR_OF_DAY                  | 一天中的小时，24小时制，从0到23。这是在标准的24小时数字时钟上观察到的小时 |
| INSTANT_SECONDS              | 即时纪元。                                                   |
| MICRO_OF_DAY                 | 计算当天的微秒，从0到（24 * 60 * 60 * 1,000,000） - 1.此字段对于所有日历系统具有相同的含义 |
| MICRO_OF_SECOND              | 计算在秒内的微秒，从0到999,999。 该字段对于所有日历系统具有相同的含义 |
| MILLI_OF_DAY                 | 当天的毫秒数，从0到（24 * 60 * 60 * 1,000） - 1              |
| MILLI_OF_SECOND              | 毫秒，从0到999.此字段对于所有日历系统具有相同的含义          |
| MINUTE_OF_DAY                | 一天中的分钟，从0到（24 * 60） - 1                           |
| MINUTE_OF_HOUR               | 分钟，从0到59                                                |
| MONTH_OF_YEAR                | 一年中的月份，例如三月。                                     |
| NANO_OF_DAY                  | 计算当天的纳秒，从0到（24 * 60 * 60 *1,000,000,000） - 1.该字段对于所有日历系统具有相同的含义 |
| NANO_OF_SECOND               | 计算在秒内的纳秒，从0到999,999,999。 该字段对于所有日历系统具有相同的含义。 |
| OFFSET_SECONDS               | 与UTC / Greenwich的偏差。                                    |
| PROLEPTIC_MONTH              | 基于预感月，从0年开始按月计数。                              |
| SECOND_OF_DAY                | 计算当天的秒数，从0到（24 * 60 * 60） - 1                    |
| SECOND_OF_MINUTE             | 计算分钟内的秒数，从0到59                                    |
| YEAR                         | 即将到来的一年，如2012年。                                   |
| YEAR_OF_ERA                  | 这个时代的一年。                                             |



### 2、ChronoField 类方法及示例

| 方法                                    | 描述                                             |
| :-------------------------------------- | :----------------------------------------------- |
| int checkValidIntValue(long value)      | 检查指定的值是否有效并且适合整数 int 。          |
| long checkValidValue(long value)        | 检查指定的值是否对此字段有效。                   |
| boolean isDateBased()                   | 检查此字段是否表示日期的组成部分。               |
| boolean isTimeBased()                   | 检查此字段是否表示时间的组成部分。               |
| ValueRange range()                      | 获取该字段的有效值范围。                         |
| static ChronoField valueOf(String name) | 返回具有指定名称的此类型的枚举常量。             |
| static ChronoField[] values()           | 按照声明的顺序返回一个包含此枚举类型常量的数组。 |

```java
// valueOf()
ChronoField chronoField = ChronoField.valueOf("HOUR_OF_DAY");
System.out.println(chronoField); // HourOfDay

// checkValidIntValue()
int validInt = chronoField.checkValidIntValue(21);
System.out.println(validInt); // 21

// checkValidIntValue()
long validLong = chronoField.checkValidIntValue(21);
System.out.println(validLong); // 21

// isDateBased() 用于检查此ChronoField是否为日期单位。从天到时代的所有ChronoUnit都是基于日期的。
System.out.println(ChronoField.HOUR_OF_DAY.isDateBased()); // false
System.out.println(ChronoField.YEAR_OF_ERA.isDateBased()); // true

// isTimeBased() 用于检查此ChronoField是否为时间单位。从nanos到half-days的所有ChronoUnit都是基于时间的。
System.out.println(ChronoField.HOUR_OF_DAY.isTimeBased()); // true
System.out.println(ChronoField.YEAR_OF_ERA.isTimeBased()); // false

// range()
System.out.println(ChronoField.HOUR_OF_DAY.range()); // 0 - 23

// values()
System.out.println(Arrays.toString(ChronoField.values()));
// [NanoOfSecond, NanoOfDay, MicroOfSecond, MicroOfDay, MilliOfSecond, MilliOfDay, SecondOfMinute, SecondOfDay, MinuteOfHour, MinuteOfDay, HourOfAmPm, ClockHourOfAmPm, HourOfDay, ClockHourOfDay, AmPmOfDay, DayOfWeek, AlignedDayOfWeekInMonth, AlignedDayOfWeekInYear, DayOfMonth, DayOfYear, EpochDay, AlignedWeekOfMonth, AlignedWeekOfYear, MonthOfYear, ProlepticMonth, YearOfEra, Year, Era, InstantSeconds, OffsetSeconds]
```



## java.time.temporal.ValueRange

### 1、ValueRange 类说明

ValueRange 表示日期时间字段的有效值范围。这个类是不可变的和线程安全的。

所有 TemporalField 实例都具有有效的值范围。 例如，ISO日期从1到28到31之间的某个位置运行。此类捕获该有效范围。重要的是要意识到这门课程的局限性。 仅提供最小值和最大值。 外部范围内可能存在无效值。 例如，奇怪的字段可以具有有效值1,2,4,6,7，因此具有范围'1-7'，尽管值3和5是无效的。此类的实例不依赖于特定字段。

ValueRange 类的声明如下：

```java
public final class ValueRange implements Serializable {
    /**
     * The smallest minimum value.最小值
     */
    private final long minSmallest;
    /**
     * The largest minimum value.最大可能最小值
     */
    private final long minLargest;
    /**
     * The smallest maximum value.最小可能最大值
     */
    private final long maxSmallest;
    /**
     * The largest maximum value.最大值
     */
    private final long maxLargest;
    // ...
}
```



### 2、ValueRange 类方法及示例

| 方法                                                         | 描述                                                        |
| :----------------------------------------------------------- | :---------------------------------------------------------- |
| int checkValidIntValue(long value, TemporalField field)      | 检查指定的值是否有效并且适合 int 。                         |
| long checkValidValue(long value, TemporalField field)        | 检查指定的值是否有效。                                      |
| boolean equals(Object obj)                                   | 检查此范围是否等于另一个范围。                              |
| long getLargestMinimum()                                     | 获取字段可以采用的最大可能最小值。                          |
| long getMaximum()                                            | 获取字段可以采用的最大值。                                  |
| long getMinimum()                                            | 获取字段可以采用的最小值。                                  |
| long getSmallestMaximum()                                    | 获取字段可以采用的最小可能最大值。                          |
| int hashCode()                                               | 此范围的哈希码。                                            |
| boolean isFixed()                                            | 值范围是否固定且完全已知。                                  |
| boolean isIntValue()                                         | 检查范围内的所有值是否适合 int 。                           |
| boolean isValidIntValue(long value)                          | 检查该值是否在有效范围内，并且该范围内的所有值都适合 int 。 |
| boolean isValidValue(long value)                             | 检查值是否在有效范围内。                                    |
| static ValueRange of(long min, long max)                     | 获得固定值范围。                                            |
| static ValueRange of(long min, long maxSmallest, long maxLargest) | 获得可变值范围。                                            |
| static ValueRange of(long minSmallest, long minLargest, long maxSmallest, long maxLargest) | 获得完全可变的值范围。                                      |
| String toString()                                            | 输出此范围为 String 。                                      |

```java
ValueRange valueRange = ValueRange.of(1L, 10000L);
System.out.println(valueRange); // 1 - 10000
valueRange = ValueRange.of(1L, 5L, 10000L, 50000L);
System.out.println(valueRange); // 1/5 - 10000/50000

LocalDateTime localDateTime = LocalDateTime.of(2021, 8, 30, 23, 14, 20);
valueRange = localDateTime.range(ChronoField.DAY_OF_MONTH);
System.out.println(valueRange.getMinimum()); // 1
System.out.println(valueRange.getMaximum()); // 32
System.out.println(valueRange.getLargestMinimum());  // 1
System.out.println(valueRange.getSmallestMaximum()); // 31
```



## java.time.temporal.TemporalAdjuster【接口】

LocalDateTime、Duration、Instant等，尽管已经提供了很多日期加减的方法，但还是比较局限性，如“下个星期天”，“下个结婚生日”，"下个月的第一天"等需求便难以满足。这时我们通过时间校正器效果可能会更好。

- TemporalAdjuster：日期校准器
- TemporalAdjusters：通过该类静态方法提供了大量的常用 TemporalAdjuster 的实现

TemporalAdjusters 中包含许多静态方法，可以直接调用，以下列举一些：

| 方法名                         | 描述                                                 |
| :----------------------------- | :--------------------------------------------------- |
| dayOfWeekInMonth()             | 返回同一个月中每周的第几天                           |
| firstDayOfMonth()              | 返回当月的第一天                                     |
| firstDayOfNextMonth()          | 返回下月的第一天                                     |
| firstDayOfNextYear()           | 返回下一年的第一天                                   |
| firstDayOfYear()               | 返回本年的第一天                                     |
| firstInMonth()                 | 返回同一个月中第一个星期几                           |
| lastDayOfMonth()               | 返回当月的最后一天                                   |
| lastDayOfYear()                | 返回本年的最后一天                                   |
| lastInMonth()                  | 返回同一个月中最后一个星期几                         |
| next(DayOfWeek week)           | 返回后一个给定的星期几                               |
| previous(DayOfWeek week)       | 返回前一个给定的星期几                               |
| nextOrSame(DayOfWeek week)     | 返回后一个给定的星期几，如果这个值满足条件，直接返回 |
| previousOrSame(DayOfWeek week) | 返回前一个给定的星期几，如果这个值满足条件，直接返回 |

```java
import java.time.DayOfWeek;
import java.time.LocalDateTime;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class JavaDateTest {
    public static void main(String[] args) {
        LocalDateTime now = LocalDateTime.now(); // 首先获取当前时间
        System.out.println("当前时间:"+now); // 当前时间:2022-02-17T22:01:45.718728600
        /**
         * 方式一：使用TemporalAdjuster接口自定义日期方式实现
         */
        TemporalAdjuster adJuster = (temporal) -> {
            LocalDateTime dateTime = (LocalDateTime) temporal;
            DayOfWeek dayOfWeek = dateTime.getDayOfWeek(); // 先获取周几
            if (DayOfWeek.FRIDAY.equals(dayOfWeek)) {
                return dateTime.plusDays(3); // 周五加三天等于工作日
            } else if (DayOfWeek.SATURDAY.equals(dayOfWeek)) {
                return dateTime.plusDays(2); // 周六加两天
            }
            return dateTime.plusDays(1); // 其他均加一天
        };
        // 下一个工作日:2022-02-18T22:01:45.718728600
        System.out.println("下一个工作日:" + now.with(adJuster));

        /**
         * 方式二：使用TemporalAdjusters工具类提供的日期来实现
         */
        LocalDateTime with1 = now.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));
        System.out.println("下周日:" + with1); // 下周日:2022-02-20T22:01:45.718728600
        LocalDateTime with2 = now.with(TemporalAdjusters.firstDayOfMonth());
        System.out.println("这个月的第一天:" + with2); // 这个月的第一天:2022-02-01T22:01:45.718728600
        LocalDateTime with3 = now.with(TemporalAdjusters.firstDayOfNextMonth());
        System.out.println("下个月的第一天:" + with3); // 下个月的第一天:2022-03-01T22:01:45.718728600
    }
}
```

```java
import java.time.DayOfWeek;
import java.time.LocalDateTime;
import java.time.temporal.TemporalAdjusters;

public class JavaDateTest {
    public static void main(String[] args) {
        LocalDateTime now = LocalDateTime.of(2021, 9, 8, 0, 20, 13);
        System.out.println("当前时间：" + now + " ==> " + now.getDayOfWeek());
        System.out.println("下一个周一：" + now.with(TemporalAdjusters.next(DayOfWeek.MONDAY)));
        System.out.println("上一个周一：" + now.with(TemporalAdjusters.previous(DayOfWeek.MONDAY)));
        System.out.println("下一个周五：" + now.with(TemporalAdjusters.nextOrSame(DayOfWeek.FRIDAY)));
        System.out.println("上一个周五：" + now.with(TemporalAdjusters.previousOrSame(DayOfWeek.FRIDAY)));
        System.out.println("本月最后一个周五：" + now.with(TemporalAdjusters.lastInMonth(DayOfWeek.FRIDAY)));
        System.out.println("本月第一个周五：" + now.with(TemporalAdjusters.firstInMonth(DayOfWeek.FRIDAY)));
        System.out.println("本月第一天：" + now.with(TemporalAdjusters.firstDayOfMonth()));
        System.out.println("本月最后一天：" + now.with(TemporalAdjusters.lastDayOfMonth()));
        System.out.println("下月的第一天：" + now.with(TemporalAdjusters.firstDayOfNextMonth()));
        System.out.println("今年的第一天：" + now.with(TemporalAdjusters.firstDayOfYear()));
        System.out.println("今年的最后一天：" + now.with(TemporalAdjusters.lastDayOfYear()));
        System.out.println("下一年的第一天：" + now.with(TemporalAdjusters.firstDayOfNextYear()));
        System.out.println("本月的第二个周五：" + now.with(TemporalAdjusters.dayOfWeekInMonth(2, DayOfWeek.FRIDAY)));
        System.out.println("两周后：" + now.with(TemporalAdjusters.ofDateAdjuster(date -> date.plusWeeks(2))));
        // 输出内容:
        // 当前时间：2021-09-08T00:20:13 ==> WEDNESDAY
        //下一个周一：2021-09-13T00:20:13
        //上一个周一：2021-09-06T00:20:13
        //下一个周五：2021-09-10T00:20:13
        //上一个周五：2021-09-03T00:20:13
        //本月最后一个周五：2021-09-24T00:20:13
        //本月第一个周五：2021-09-03T00:20:13
        //本月第一天：2021-09-01T00:20:13
        //本月最后一天：2021-09-30T00:20:13
        //下月的第一天：2021-10-01T00:20:13
        //今年的第一天：2021-01-01T00:20:13
        //今年的最后一天：2021-12-31T00:20:13
        //下一年的第一天：2022-01-01T00:20:13
        //本月的第二个周五：2021-09-10T00:20:13
        //两周后：2021-09-22T00:20:13
    }
}
```



## java.time.chrono.Chronology【判断闰年】

判断是否闰年是由年表 Chronology类 提供的，通常情况下，我们使用ISO下的年表，是 IsoChronology。看下代码实现：

```java
@Override
public boolean isLeapYear(long prolepticYear) {
    return ((prolepticYear & 3) == 0) && ((prolepticYear % 100) != 0 || (prolepticYear % 400) == 0);
}
```

闰年的基本判定方法：

1. 非整百年：能被4整除的为闰年。（如2004年就是闰年,2001年不是闰年）
2. 整百年：能被400整除的是闰年。(如2000年是闰年，1900年不是闰年)

```java
((prolepticYear & 3) == 0) && ((prolepticYear % 100) != 0 || (prolepticYear % 400) == 0);
```

这段代码用了两个条件，这两个条件都符合，才是闰年。

- **(prolepticYear & 3) == 0**
- **(prolepticYear % 100) != 0 || (prolepticYear % 400) == 0**

(prolepticYear & 3) == 0 用了与运算符“&”,其使用规律如下：

- 两个操作数中位都为1，结果才为1，否则结果为0。
- 3 的二进制是011 ，prolepticYear & 3 目的是保留最后2位二进制数，然后判断是否最后两位二进制数等于0。如果等于0，证明能被4整除。闰年一定要满足是4的倍数的条件；

(prolepticYear % 100) != 0 || (prolepticYear % 400) == 0 这个就比较好理解了，看是不是100的倍数或者是不是400 倍数。

而且发现 java.time.Year#isLeap() 用的实现代码逻辑是一样的：

```java
public static boolean isLeap(long year) {
    return ((year & 3) == 0) && ((year % 100) != 0 || (year % 400) == 0);
}
```

上面 IsoChronology 是对 Chronology 接口接口的 isLeapYear 实现，MinguoChronology 等实现类的 isLeapYear，互用了 IsoChronology 的 isLeapYear 方法。

```java
// MinguoChronology 
public boolean isLeapYear(long prolepticYear) {
    return IsoChronology.INSTANCE.isLeapYear(prolepticYear + YEARS_DIFFERENCE);
}
```

巨佬是有考虑复用的，在MinguoChronology等实现类已经有复用了。

java.time.Year#isLeap() 的优先级高，因为它是静态方法。isoChronology 可以引 Year.isLeap，Year 不可以引 Chronology.isLeapYear

【操作示例】

```java
int year = 2020;
System.out.println(Year.isLeap(year)); // true
System.out.println(IsoChronology.INSTANCE.isLeapYear(year)); // true

LocalDate localDate = LocalDate.of(2021, 9, 7);
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println(localDate.isLeapYear()); // false
System.out.println(localDateTime.toLocalDate().isLeapYear()); // false
```



## 日期时间比较与日期时间间隔计算

### 1、比较日期时间的先后

基本上都有这四个比较方法：：compareTo()、isBefore()、isAfter()、和equals()

【操作示例 1】LocalDate 比较

```java
LocalDate localDate1 = LocalDate.of(2021, 8, 14);
// 比较指定日期和参数日期，返回正数，那么指定日期时间较晚（数字较大）:13
int i = localDate1.compareTo(LocalDate.of(2021, 8, 1));
System.out.println(i); // 13
// 比较指定日期是否比参数日期早（true为早）:true
System.out.println(localDate1.isBefore(LocalDate.of(2021, 8, 31))); // true
// 比较指定日期是否比参数日期晚（true为晚）:false
System.out.println(localDate1.isAfter(LocalDate.of(2021, 8, 31))); // false
// 比较两个日期是否相等:true
System.out.println(localDate1.isEqual(LocalDate.of(2021, 8, 14))); // true
```

【操作示例 2】LocalTime 比较

```java
LocalTime localTime1 = LocalTime.of(23, 26, 30);
LocalTime localTime2 = LocalTime.of(23, 26, 32);
// 两个时间进行比较 大返回1，小就返回-1，一样就返回0:-1
System.out.println(localTime1.compareTo(localTime2)); // -1

// 比较指定时间是否比参数时间早（true为早）:true
System.out.println(localTime1.isBefore(localTime2)); // true
// 比较指定时间是否比参数时间晚（true为晚）:false
System.out.println(localTime1.isAfter(localTime2)); // false
// 比较两个时间是否相等:true
System.out.println(localTime1.equals(LocalTime.of(23, 26, 30))); // true
```

【操作示例 3】OffsetDateTime 比较

```java
LocalDateTime  localDateTime1  = LocalDateTime.of(2021, 8, 15, 13, 14, 20);
OffsetDateTime offsetDateTime1 = OffsetDateTime.of(localDateTime1, ZoneOffset.ofHours(8));
OffsetDateTime offsetDateTime3 = OffsetDateTime.of(localDateTime1, ZoneOffset.ofHours(8));

LocalDateTime  localDateTime2  = LocalDateTime.of(2021, 8, 15, 13, 14, 30);
OffsetDateTime offsetDateTime2 = OffsetDateTime.of(localDateTime2, ZoneOffset.ofHours(8));

// 两个时间进行比较 大返回1，小就返回-1，一样就返回0:-1
System.out.println(offsetDateTime1.compareTo(offsetDateTime2)); //-1

// 比较指定时间是否比参数时间早（true为早）:true
System.out.println(offsetDateTime1.isBefore(offsetDateTime2)); // true
// 比较指定时间是否比参数时间晚（true为晚）:false
System.out.println(offsetDateTime1.isAfter(offsetDateTime2)); // false
// 比较两个时间是否相等:true
System.out.println(offsetDateTime1.equals(offsetDateTime3)); // true
```

【操作示例 4】OffsetTime 比较

```java
LocalTime  localTime1  = LocalTime.of(13, 14, 20);
OffsetTime offsetTime1 = OffsetTime.of(localTime1, ZoneOffset.ofHours(8));
OffsetTime offsetTime3 = OffsetTime.of(localTime1, ZoneOffset.ofHours(8));

LocalTime  localTime2  = LocalTime.of(13, 14, 30);
OffsetTime offsetTime2 = OffsetTime.of(localTime2, ZoneOffset.ofHours(8));
// 两个时间进行比较 大返回1，小就返回-1，一样就返回0:-1
System.out.println(offsetTime1.compareTo(offsetTime2)); // -1

// 比较指定时间是否比参数时间早（true为早）:true
System.out.println(offsetTime1.isBefore(offsetTime2)); // true
// 比较指定时间是否比参数时间晚（true为晚）:false
System.out.println(offsetTime1.isAfter(offsetTime2)); // false
// 比较两个时间是否相等:true
System.out.println(offsetTime1.equals(offsetTime3)); // true
```

【操作示例 5】ZonedDateTime 比较

```java
LocalDateTime localDateTime1 = LocalDateTime.of(2021, 8, 15, 13, 14, 20);
ZonedDateTime zonedDateTime1 = ZonedDateTime.of(localDateTime1, ZoneOffset.ofHours(8));

ZonedDateTime zonedDateTime3 = ZonedDateTime.of(localDateTime1, ZoneOffset.ofHours(8));

LocalDateTime localDateTime2 = LocalDateTime.of(2021, 8, 15, 13, 14, 30);
ZonedDateTime zonedDateTime2 = ZonedDateTime.of(localDateTime2, ZoneOffset.ofHours(8));

// 两个时间进行比较 大返回1，小就返回-1，一样就返回0:-1
System.out.println(zonedDateTime1.compareTo(zonedDateTime2)); // -1

// 比较指定时间是否比参数时间早（true为早）:true
System.out.println(zonedDateTime1.isBefore(zonedDateTime2)); // true
// 比较指定时间是否比参数时间晚（true为晚）:false
System.out.println(zonedDateTime1.isAfter(zonedDateTime2)); // false
// 比较两个时间是否相等:true
System.out.println(zonedDateTime1.equals(zonedDateTime3)); // true
```



### 2、计算日期时间的间隔

Duration 和 Period  都有 between  方法

【操作示例 1】Duration#between()

```java
LocalDateTime from = LocalDateTime.of(2017, 01, 1, 00, 0, 0);   // 2017-01-01 00:00:00
LocalDateTime to = LocalDateTime.of(2019, 9, 12, 14, 28, 0); // 2019-09-12 14:28:00
Duration duration = Duration.between(from, to);  // 表示从 from 到 to 这段时间（第⼆个参数减第⼀个参数）
System.out.println(duration.toDays()); // 984
System.out.println(duration.toHours()); // 23630
System.out.println(duration.toMinutes()); // 1417828
System.out.println(duration.getSeconds()); // 85069680
System.out.println(duration.toMillis()); // 85069680000
System.out.println(duration.toNanos()); // 85069680000000000
```

【操作示例 2】Period#between()

```java
LocalDate from = LocalDate.of(2017, 01, 01); // 2017-01-01
LocalDate to = LocalDate.of(2019, 9, 12); // 2019-09-12
Period period = Period.between(from, to);  // 表示从 from 到 to 这段时间（第⼆个参数减第⼀个参数）
System.out.println(period); // P2Y8M11D
System.out.println(period.getDays()); // 11
System.out.println(period.getMonths()); // 8
System.out.println(period.getYears()); // 2
System.out.println(period.get(ChronoUnit.YEARS)); // 2
```



## java.time.format.DateTimeFormatter

### 1、DateTimeFormatter 类说明

Java 8 后开始使用 java.time.format.DateTimeFormatter 类用于解析日期字符串和格式化日期输出。格式化字符串的使用方式与 Java 8 之前的日期时间处理类 java.text.DateFormat 完全一致。

java.time.format.DateTimeFormatter 类：该类提供了三种格式化方法：

- 预定义的标准格式（系统提供好了枚举类）
- 语言环境相关的格式（设置指定的语言环境Locale，如：Locale.CHINA、Locale.US）
- 自定义的格式（自定义设置格式，例如：DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm")）

DateTimeFormatter 类包含一系列预定义常量的实例，可以解析和格式化一些标准时间格式。这将让你免除麻烦的时间格式定义，类中预定义实例如下：

```java
public final class DateTimeFormatter {
    public static final DateTimeFormatter BASIC_ISO_DATE;
    public static final DateTimeFormatter ISO_LOCAL_DATE;
    public static final DateTimeFormatter ISO_LOCAL_TIME;
    public static final DateTimeFormatter ISO_LOCAL_DATE_TIME;
    public static final DateTimeFormatter ISO_OFFSET_DATE;
    public static final DateTimeFormatter ISO_OFFSET_TIME;
    public static final DateTimeFormatter ISO_OFFSET_DATE_TIME;
    public static final DateTimeFormatter ISO_ZONED_DATE_TIME;
    public static final DateTimeFormatter ISO_INSTANT;
    public static final DateTimeFormatter ISO_DATE;
    public static final DateTimeFormatter ISO_TIME;
    public static final DateTimeFormatter ISO_DATE_TIME;
    public static final DateTimeFormatter ISO_ORDINAL_TIME;
    public static final DateTimeFormatter ISO_WEEK_DATE;
    public static final DateTimeFormatter RFC_1123_DATE_TIME;
}
```



### 2、DateTimeFormatter 类方法及示例

| 方法                                             | 描述                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| static DateTimeFormatter ofPattern(“yyyy-MM-dd”) | 静态方法，通过给定格式获取对象                               |
| String format(TemporalAccessor temporal)         | 把一个日期对象的**默认格式**格式化成**指定的格式**的**字符串** |
| TemporalAccessor parse(CharSequence text)        | 把一个字符串格式化成指定格式的 TemporalAccessor 对象（不建议用） |

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAccessor;
import java.util.Locale;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * DateTimeFormatter 类的两种转换方式
         *   Date -> String：String format(TemporalAccessor temporal)
         *   String -> Date：TemporalAccessor parse(CharSequence text)
         */
        LocalDateTime localDateTime = LocalDateTime.of(2020, 12, 12, 12, 12);
        // DateTimeFormatter类的：Date -> String
        System.out.println(DateTimeFormatter.ISO_DATE.format(localDateTime)); // 2020-12-12
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        System.out.println(dateTimeFormatter.format(localDateTime)); // 2020-12-12 12:12:00
        DateTimeFormatter dateTimeFormatter1 = DateTimeFormatter.ofPattern("yyyy-MMM-dd HH:mm:ss E", Locale.CHINA);
        System.out.println(dateTimeFormatter1.format(localDateTime)); // 2020-12月-12 12:12:00 周六
        DateTimeFormatter dateTimeFormatter2 = DateTimeFormatter.ofPattern("EEEE, MMMM/dd/yyyy HH:mm", Locale.US);
        System.out.println(dateTimeFormatter2.format(localDateTime)); // Saturday, December/12/2020 12:12

        /**
         * DateTimeFormatter类的：String -> Date
         * Java DateTimeFormatter.parse()的坑：
         * 直接转换是无法转换的,运行时的具体类型是：java.time.format.Parsed类型，无法类型转换为java.io下面的时间类
         * 参考：https://blog.csdn.net/D___H/article/details/121479052
         */
        TemporalAccessor parse = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").parse("2020-12-12 12:12:12");
        System.out.println(parse); // {},ISO resolved to 2020-12-12T12:12:12
        // 直接(强)转换是无法转换的，所以只能使用对应日期类的from()方法获得对应的日期类型
        LocalDateTime from = LocalDateTime.from(parse);
        System.out.println(from); // 2020-12-12T12:12:12
        // 直接强转会报错：java.time.format.DateTimeParseException: Text '2020-12-12 12:12:12' could not be parsed at index 5
        LocalDateTime parseLocalDateTime = (LocalDateTime) dateTimeFormatter1.parse("2020-12-12 12:12:12");
    }
}
```



### 3、DateTimeFormatter 类的坑

1、在正常配置按照标准格式的字符串日期，是能够正常转换的。如果月，日，时，分，秒在不足两位的情况需要补0，否则的话会转换失败，抛出异常。

```java
DateTimeFormatter DATE_TIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS");
LocalDateTime dt1 = LocalDateTime.parse("2021-7-20 23:46:43.946", DATE_TIME_FORMATTER);
System.out.println(dt1);
// java.time.format.DateTimeParseException: Text '2021-7-20 23:46:43.946' could not be parsed at index 5
```

分析原因：是格式字符串与实际的时间不匹配

- “yyyy-MM-dd HH:mm:ss.SSS”
- “2021-7-20 23:46:43.946”
- 中间的月份格式是MM，实际时间是7

解决方案：保持格式字符串与实际的时间匹配

```java
DateTimeFormatter DATE_TIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS");
LocalDateTime dt1 = LocalDateTime.parse("2021-07-20 23:46:43.946", DATE_TIME_FORMATTER);
System.out.println(dt1); // 2021-07-20T23:46:43.946
```



2、YYYY 和 DD 谨慎使用

```java
LocalDate date = LocalDate.of(2020, 12, 31);
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("YYYYMM");
System.out.println(formatter.format(date)); // 结果是 202112
```

YYYY 是取的当前周所在的年份，week-based year 是 ISO 8601 规定的。2020年12月31号，周算年份，就是2021年

> Java’s DateTimeFormatter pattern “YYYY” gives you the week-based-year, (by default, ISO-8601 standard) the year of the Thursday of that week.

```java
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;

public class JavaDateTest {
    private static void tryit(int Y, int M, int D, String pat) {
        DateTimeFormatter fmt = DateTimeFormatter.ofPattern(pat);
        LocalDate         dat = LocalDate.of(Y, M, D);
        String            str = fmt.format(dat);
        System.out.printf("Y=%04d M=%02d D=%02d " + "formatted with " + "\"%s\" -> %s\n", Y, M, D, pat, str);
    }
    public static void main(String[] args) {
        tryit(2020, 01, 20, "MM/DD/YYYY");
        tryit(2020, 01, 21, "DD/MM/YYYY");
        tryit(2020, 01, 22, "YYYY-MM-DD");
        tryit(2020, 03, 17, "MM/DD/YYYY");
        tryit(2020, 03, 18, "DD/MM/YYYY");
        tryit(2020, 03, 19, "YYYY-MM-DD");
    }
}
```

```
Y=2020 M=01 D=20 formatted with "MM/DD/YYYY" -> 01/20/2020
Y=2020 M=01 D=21 formatted with "DD/MM/YYYY" -> 21/01/2020
Y=2020 M=01 D=22 formatted with "YYYY-MM-DD" -> 2020-01-22
Y=2020 M=03 D=17 formatted with "MM/DD/YYYY" -> 03/77/2020
Y=2020 M=03 D=18 formatted with "DD/MM/YYYY" -> 78/03/2020
Y=2020 M=03 D=19 formatted with "YYYY-MM-DD" -> 2020-03-79
```

最后三个日期是有问题的，因为大写的**DD**代表的是处于这一年中那一天，不是处于这个月的那一天，但是**dd**就没有问题。所以建议使用yyyy和dd。例子参考于：https://www.cnblogs.com/tonyY/p/12153335.html



3、DateTimeFormatter.format(Instant) 会报错

```
java.time.temporal.UnsupportedTemporalTypeException: Unsupported field: YearOfEra
```

解决方案：先把date类型转化为LocalDateTime类型，然后再进行DateTimeFormatter.format(LocalDateTime)的格式化。

参考于：https://xiaoxuzhu.blog.csdn.net/article/details/120477317



### 4、DateTimeFormatterBuilder

1、DateTimeFormatter 的所有格式化器都是用 DateTimeFormatterBuilder 建造器类创建的。看下面两个**ofPattern** 源码：

```java
public static DateTimeFormatter ofPattern(String pattern) {
    return new DateTimeFormatterBuilder().appendPattern(pattern).toFormatter();
}

public static DateTimeFormatter ofPattern(String pattern, Locale locale) {
    return new DateTimeFormatterBuilder().appendPattern(pattern).toFormatter(locale);
}
```

2、解析风格配置：官方提供了四种解析风格的配置，如下枚举 SettingsParser ：

```java
static enum SettingsParser implements DateTimePrinterParser {
    SENSITIVE, // 大小写敏感
    INSENSITIVE, // 大小写不敏感
    STRICT, // 严格
    LENIENT; // 宽松
    // ...
}
```

3、对应 DateTimeFormatterBuilder 类中的方法源码，可以看出，都是调用 appendInternal 方法：

```java
public final class DateTimeFormatterBuilder {
    // 大小写敏感
    public DateTimeFormatterBuilder parseCaseSensitive() {
        appendInternal(SettingsParser.SENSITIVE);
        return this;
    }
    // 大小写不敏感
    public DateTimeFormatterBuilder parseCaseInsensitive() {
        appendInternal(SettingsParser.INSENSITIVE);
        return this;
    }
    // 严格
    public DateTimeFormatterBuilder parseStrict(){
        appendInternal(SettingsParser.INSENSITIVE);
        return this;
    }
    // 宽松
    public DateTimeFormatterBuilder parseLenient(){
        appendInternal(SettingsParser.LENIENT);
        return this;
    }
}
```

4、接着往下看 appendInternal 源码：

```java
private int appendInternal(DateTimePrinterParser pp) {
    Objects.requireNonNull(pp, "pp");
    if (active.padNextWidth > 0) {
        if (pp != null) {
            pp = new PadPrinterParserDecorator(pp, active.padNextWidth, active.padNextChar);
        }
        active.padNextWidth = 0;
        active.padNextChar = 0;
    }
    active.printerParsers.add(pp);
    active.valueParserIndex = -1;
    return active.printerParsers.size() - 1;
}
```

- 其中active 是一个DateTimeFormatterBuilder 实例，且这个DateTimeFormatterBuilder 实例内部有一个列表 List< DateTimePrinterParser > ，看了源码可知，真正做解析工作的是DateTimePrinterParser 对应的实例来做的。

5、DateTimePrinterParser 的源码，源码有一共有16个 DateTimePrinterParser 的实例。

```java
interface DateTimePrinterParser {
    // 解析于格式化方法
    boolean format(DateTimePrintContext context, StringBuilder buf);
    int parse(DateTimeParseContext context, CharSequence text, int position);

    // 1.Composite printer and parser.
    static final class CompositePrinterParser implements DateTimePrinterParser {...}

    // 2.Pads the output to a fixed width.
    static final class PadPrinterParserDecorator implements DateTimePrinterParser {...}

    // 3.Enumeration to apply simple parse settings.
    static enum SettingsParser implements DateTimePrinterParser{...}

    // 4. Defaults a value into the parse if not currently present.
    static class DefaultValueParser implements DateTimePrinterParser {...}

    // 5.Prints or parses a character literal.
    static final class CharLiteralPrinterParser implements DateTimePrinterParser {...}

    // 6.Prints or parses a string literal.
    static final class StringLiteralPrinterParser implements DateTimePrinterParser {...}

    // 7.Prints and parses a numeric date-time field with optional padding.
    static class NumberPrinterParser implements DateTimePrinterParser {...}

    // 8.Prints and parses a numeric date-time field with optional padding.
    static final class FractionPrinterParser implements DateTimePrinterParser {...}

    // 9.Prints or parses field text.
    static final class TextPrinterParser implements DateTimePrinterParser  {...}

    // 10.Prints or parses an ISO-8601 instant.
    static final class InstantPrinterParser implements DateTimePrinterParser  {...}

    // 11.Prints or parses an offset ID.
    static final class OffsetIdPrinterParser implements DateTimePrinterParser   {...}

    // 12.Prints or parses an offset ID.
    static final class LocalizedOffsetIdPrinterParser implements DateTimePrinterParser {...}

    // 13.Prints or parses a zone ID.
    static class ZoneIdPrinterParser implements DateTimePrinterParser {...}

    // 14. Prints or parses a chronology.
    static final class ChronoPrinterParser implements DateTimePrinterParser {...}

    // 15.Prints or parses a localized pattern.
    static final class LocalizedPrinterParser implements DateTimePrinterParser {...}

    // 16.Prints or parses a localized pattern from a localized field.
    static final class WeekBasedFieldPrinterParser implements DateTimePrinterParser {...}
}
```



### 5、DateTimeFormatter 注意事项

**注意**：虽然 DateTimeFormatter 类可以进行日期时间格式化输出 & 字符串解析，但是我们更常用的还是 java.time 下每个日期类对应的静态 parse() 和 format() 方法来格式化输出 & 字符串解析。

大家有没有注意到，格式化 parse 方法 是放在 LocalDateTime 类中的，而不是 DateTimeFormatter 类中 。这样的设计符合正常的思路想法，想解析出 LocalDateTime 的日期时间，那就用 LocalDateTime 。想解析其他的 JSR-310 的日期时间对象，那就用对应的日期时间对象去解析。

我把常用的日期时间API都看了，这些里面除了 Clock （时钟不需要解析的），其他都有实现 parse 方法 。如下类支持 parse() 方法”

- java.time.Instant
- java.time.LocalTime
- java.time.LocalDate
- java.time.LocalDateTime
- java.time.OffsetDateTime
- java.time.OffsetTime
- java.time.ZonedDateTime



## 日期时间格式化与解析

java.time.format.DateTimeFormatter 能够进行 TemporalAccessor 类型（包括：LocalDate、LocalTime、LocalDateTime、ZonedDateTime、OffsetDateTime）的格式化输出。同时 LocalDate、LocalTime、LocalDateTime、ZonedDateTime、OffsetDateTime 提供了静态的 parse 方法，能够进行字符串解析。

| 类型           | 默认格式示例                             |
| -------------- | ---------------------------------------- |
| Instant        | 2017-11-23T10:15:30.00Z                  |
| LocalDate      | 2017-11-23                               |
| LocalTime      | 10:15:30                                 |
| LocalDateTime  | 2017-11-23T10:15:30                      |
| OffsetDateTime | 2017-11-23T10:15:30+01:00                |
| ZonedDateTime  | 2017-11-23T10:15:30+01:00[Asia/Shanghai] |

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Locale;

public class JavaDateTest {
    public static void main(String[] args) {

        /**
         * 日期时间类（LocalDateTime...等）自带的两种转换方式
         *   Date -> String：String format(DateTimeFormatter formatter)
         */
        LocalDateTime now = LocalDateTime.of(2020, 12, 12, 12, 12, 12);
        // Date -> String
        System.out.println("----------Date -> String----------");
        // 2020-12-12T12:12:12
        System.out.println(now.format(DateTimeFormatter.ISO_DATE_TIME));
        // 2020-12月-12 12:12:12 周六
        System.out.println(now.format(DateTimeFormatter.ofPattern("yyyy-MMM-dd HH:mm:ss E", Locale.CHINA)));
        // Saturday, December/12/2020 12:12
        System.out.println(now.format(DateTimeFormatter.ofPattern("EEEE, MMMM/dd/yyyy HH:mm", Locale.US)));
        // 2020-12-12 12:12:12
        System.out.println(now.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        
        /**
         * 日期时间类（LocalDateTime...等）自带的两种转换方式
         * String -> Date：LocalDateTime parse(CharSequence text, DateTimeFormatter formatter)
         */
        System.out.println("----------String -> Date----------");
        // 2020-12-12T12:12:12
        System.out.println(LocalDateTime.parse("2020-12-12T12:12:12", DateTimeFormatter.ISO_DATE_TIME));
        // 2020-12-12T12:12
        System.out.println(LocalDateTime.parse("Saturday, December/12/2020 12:12", DateTimeFormatter.ofPattern("EEEE, MMMM/dd/yyyy HH:mm", Locale.US)));
        // 2020-12-12T12:12:12
        System.out.println(LocalDateTime.parse("2020-12-12 12:12:12", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }
}
```



## 日期时间格式化与解析模板

格式化的模式由指定的字符串组成，未加引号的大写/小写字母（A-Z a-z）代表特定模式，用来表示模式含义，若想**原样输出**可以用单引号’'包起来，除了英文字母其它均不解释原样输出/匹配。下面是它规定的模式字母（其它字母原样输出）：

| 字母 | 含义                           | 匹配类型     | 示例                                                  |
| :--- | :----------------------------- | :----------- | :---------------------------------------------------- |
| y    | 年                             | Year         | 2020,20                                               |
| M    | 月                             | Month        | July; Jul; 07                                         |
| d    | 月中的天数（俗称日，最大值31） | Number       | 10                                                    |
| H    | 小时(0-23)                     | Number       | 0,23                                                  |
| m    | 分钟(0-59)                     | Number       | 30,59                                                 |
| s    | 秒(0-59)                       | Number       | 30,59                                                 |
| ---  | ---                            | ---          | yyyy-MM-dd HH:mm:ss（分隔符可以是任意字符，甚至汉字） |
| Y    | 当前周所在的年份               | Year         | 2020（不建议使用，周若跨年有坑）                      |
| S    | 毫秒数(1-999)                  | Number       | 999                                                   |
| a    | am/pm                          | Text         | PM、AM; 上午、下午                                    |
| z    | 时区                           | 通用时区     | Pacific Standard Time; PST; GMT-08:00                 |
| Z    | 时区                           | RFC 822时区  | -0800,+0800                                           |
| X    | 时区                           | ISO 8601时区 | -08; -0800; -08:00                                    |
| G    | 年代                           | Text         | AD(公元)、BC(公元前)                                  |
| D    | 年中的天数(1-366)              | Number       | 360                                                   |
| w    | 年中的周数(1-54)               | Number       | 27                                                    |
| W    | 月中的周数(1-5)                | Number       | 3                                                     |
| E    | 星期几名称                     | Text         | Tuesday; Tue                                          |
| u    | 星期几数字(1=Monday...)        | Number       | 1                                                     |
| k    | 小时(1-24)                     | Number       | 不建议使用                                            |
| K/h  | am/pm小时数字                  | Number       | 一般配合a一起使用                                     |

这个表格里出现了一些“特殊”的匹配类型，做如下解释：

- **Text**：格式化(Date -> String)，如果模式字母的数目是4个或更多，则使用完整形式；否则，如果可能的话，使用简短或缩写形式。对于解析（String -> Date），这两种形式都一样，与模式字母的数量无关（注意：a不能再DateTimeFormatter使用多个，可以在SimpleDateFormat使用多个）

  ```java
  import java.time.LocalDateTime;
  import java.time.format.DateTimeFormatter;
  import java.util.Locale;
  
  public class JavaDateTest {
      public static void main(String[] args) {
          /**
           * 观察打印结果，除了符合模式规则外，还能在String -> Date解析时总结出两点结论：
           *   1.英文单词，不分区大小写。如SatUrday sunDay都是没问题，但是不能有拼写错误
           *   2.若有多个part表示一个意思，那么last win。如Sat SatUrday sunDay最后一个生效
           */
          LocalDateTime now = LocalDateTime.now();
          System.out.println("↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓中文地区模式↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓");
          DateTimeFormatter dateTimeFormatter1 = DateTimeFormatter.ofPattern("G yyyy-MM-dd HH:mm a E EE EEEEE", Locale.CHINA);
          System.out.println("====================Date->String====================");
          System.out.println(now.format(dateTimeFormatter1));
          System.out.println("====================String->Date====================");
          System.out.println(LocalDateTime.parse("公元 2022-02-20 17:18 下午 周日 周日 日", dateTimeFormatter1));
  
          System.out.println("↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓英文地区模式↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓");
          DateTimeFormatter dateTimeFormatter2 = DateTimeFormatter.ofPattern("G yyyy-MM-dd HH:mm a E EE EEEEE", Locale.US);
          System.out.println("====================Date->String====================");
          System.out.println(now.format(dateTimeFormatter2));
          System.out.println("====================String->Date====================");
          System.out.println(LocalDateTime.parse("AD 2022-02-20 17:22 PM Sun Sun S", dateTimeFormatter2));
      }
  }
  
  // 输出内容
  ====================Date->String====================
  公元 2022-02-20 17:22 下午 周日 周日 日
  ====================String->Date====================
  2022-02-20T17:18
  ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓英文地区模式↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
  ====================Date->String====================
  AD 2022-02-20 17:22 PM Sun Sun S
  ====================String->Date====================
  2022-02-20T17:22
  ```

- **Number**：格式化(Date -> String)，模式字母的数量是数字的【最小】数量，较短的数字被零填充到这个数量。对于解析(String -> Date)，模式字母的数量将被忽略，除非需要分隔两个相邻的字段

- **Year**：对于格式化和解析，如果模式字母的数量是4个或更多，则使用特定于日历的长格式。否则，使用日历特定的简短或缩写形式

- **Month**：如果模式字母的数量是3个或更多，则被解释为文本；否则，它将被解释为一个数字。

- **通用时区**：如果该时区有名称，如Pacific Standard Time、PST、CST等那就用名称，否则就用GMT规则的字符串，如：GMT-08:00

- **RFC 822时区**：遵循RFC 822格式，向下兼容通用时区（名称部分除外）

- **ISO 8601时区**：对于格式化，如果与GMT的偏移值为0（也就是格林威治时间喽），则生成“Z”；如果模式字母的数量为1，则忽略小时的任何分数。例如，如果模式是“X”，时区是“GMT+05:30”，则生成“+05”。在进行解析时，“Z”被解析为UTC时区指示符。一般时区不被接受。如果模式字母的数量是4个或更多，在构造SimpleDateFormat或应用模式时抛出IllegalArgumentException。

  - 这个规则理解起来还是比较费劲的，在开发中一般不太建议使用此种模式。若要使用请务必本地做好测试



## JSR-310 几个主要接口的说明

### 1、java.time.temporal.TemporalAccessor

什么是 TemporalAccessor？

1. Temporal 表示是：时间的
2. Accessor 表示是：访问方法，访问器

连在一起也就是时间的访问方法，说起时间，这个词的范围太广太大了，什么各种日历系统，什么夏令时，什么时区，很是复杂，因此从这个接口简单地都可以看得出，设计API的作者是希望我们能够用比较简单的方式来访问时间，降低时间概念本身的复杂性，那怎么简单法呢？让我们来具体看看TemporalAccessor接口的方法吧：

```java
boolean isSupported(TemporalField field);

long getLong(TemporalField field);

default ValueRange range(TemporalField field);

default int get(TemporalField field);

default <R> R query(TemporalQuery<R> query);
```

一共就5个方法，其中只有2个是抽象方法，还有3个default方法，就2个抽象方法，这已经抽象的很简单了，大家注意看这5个方法中用的最多的就是另一个接口TemporalField：**时间字段**。毕竟TemporalAccessor大量使用了TemporalField，所以要明白TemporalAccessor怎么表示时间，可能需要先明白什么是TemporalField【请参考之前的教程】。

结论：TemporalAccessor接口完全就是一个**只读**的接口，用于各种维度的去描述时间，但是可以知道的是，其实我们日常还是会有很多对于时间做运算的场景，比如知道今天的日期，算明天的日期，知道今天的日期，算1年2个月后的日期等，所以只有只读的接口抽象显然也是很不完整的



### 2、java.time.temporal.Temporal

> Java8日期时间时区之 Temporal接口 笔记：https://blog.csdn.net/kfepiza/article/details/115586668

Temporal 是 Java 8 日期时间框架的一个接口，Instant、LocalDate、LocalTime、LocalDateTime、ZonedDateTime、OffsetDateTime 都implements 实现了它。

1、Temporal 的一些实现类：

```java
public final class Instant implements Temporal, TemporalAdjuster, Comparable<Instant>, Serializable {}
public final class LocalDate implements Temporal, TemporalAdjuster, ChronoLocalDate, Serializable {}
public final class LocalDate implements Temporal, TemporalAdjuster, ChronoLocalDate, Serializable {}
public final class LocalDate implements Temporal, TemporalAdjuster, ChronoLocalDate, Serializable {}
public final class ZonedDateTime implements Temporal, ChronoZonedDateTime<LocalDate>, Serializable {}
public final class OffsetDateTime implements Temporal, TemporalAdjuster, Comparable<OffsetDateTime>, Serializable {}
public final class OffsetTime implements Temporal, TemporalAdjuster, Comparable<OffsetTime>, Serializable {}
public final class Year implements Temporal, TemporalAdjuster, Comparable<Year>, Serializable {}
public final class YearMonth implements Temporal, TemporalAdjuster, Comparable<YearMonth>, Serializable {}
```

2、它继承自 TemporalAccessor 接口：

```java
public interface Temporal extends TemporalAccessor {}
```

3、Temporal 接口说明及翻译：

框架级别的接口，定义对时间对象（例如日期，时间，偏移量或它们的某种组合）的读写访问。

这是日期，时间和偏移量对象的基本接口类型，这些对象足够完整以至于可以使用加号和减号进行操作【所以它们都有 with plus minus until 方法】。它是由可以以字段或查询形式提供和操作信息的类实现的。有关此版本的只读版本，请参见 TemporalAccessor 界面。

大多数日期和时间信息都可以用数字表示。这些信息是使用 TemporalField 建模的，其中数字使用长号来处理较大的值。年，月和月中的日期是字段的简单示例，但它们还包含即时和偏移量。有关标准字段集，请参阅 ChronoField。

日期/时间信息不能用数字来表示，日期和时区这两个信息可以通过使用 TemporalQuery 定义的静态方法通过查询来访问。

此接口是框架级别的接口，不应在应用程序代码中广泛使用。相反，应用程序应该创建并绕过具体类型的实例，例如LocalDate。这有很多原因，部分原因是该接口的实现可能在 ISO 以外的日历系统中。有关 java.time.chrono.ChronoLocalDate 的信息，请参见 java.time.chrono.ChronoLocalDate 对问题进行更充分的讨论。

> 何时实施 or 如何实施：
>

如果一个类满足以下三个条件，则应实现此接口：

- 根据 TemporalAccessor，它提供对日期/时间/偏移信息的访问
- 一组字段从最大到最小是连续的
- 一组字段是完整的，因此不需要其他字段来定义表示的字段的有效值范围

有四个例子可以说明这一点：

- LocalDate实现此接口，因为它表示从天到永远都是连续的字段集，不需要外部信息即可确定每个日期的有效性。因此，它能够正确实现加/减。
- LocalTime实现此接口，因为它表示从纳秒到数天之内连续的一组字段，不需要外部信息即可确定有效性。通过绕一整天，它能够正确地实现加/减。
- MonthDay（年月和月日的组合）未实现此接口。虽然组合是连续的，但在几年中从几天到几个月不等，所以该组合没有足够的信息来定义月中某天的值的有效范围。因此，它无法正确实现加/减。
- 周日和月日（“ 13日星期五”）的组合不应实现此接口。它不代表连续的字段集，因为几天到几周重叠几天到几月。



### 3、java.time.temporal.TemporalAmount

TemporalAmount 是 java.time 包下的一个接口，表示一段具体的时间（如：1年、2个月、5天、1小时、3分钟等），时间可以是负数。TemporalAmount 有两个常见的实现类：Duration 和 Period。并且在学习 TemporalAmount 接口及实现类之前，先了解 TemporalUnit、TemporalField 接口及其实现类 ChronoUnit、ChronoField。TemporalAmount 接口定义如下：

```java
public interface TemporalAmount {
    long get(TemporalUnit unit);
    List<TemporalUnit> getUnits();
    Temporal addTo(Temporal temporal);
    Temporal subtractFrom(Temporal temporal);
}
```



### 4、java.time.temporal.TemporalUnit

> 请参考之前的教程

### 5、java.time.temporal.TemporalField

> 请参考之前的教程

### 6、java.time.temporal.TemporalAdjuster

> 请参考之前的教程




## Java 8 Date/Time JSR-310 总结

前面详细介绍了 Java 8 的日期时间 API，现在进行简单总结一下。新的时间与日期 API 中很重要的一点是，它定义清楚了基本的时间与日期的概念，比方说日期、时间、瞬时时间、持续时间、时区及时间段。它们都是基于 ISO8601 日历系统，它是世界民用历法，也就是我们所说的公历。

1、日期时间相关枚举类（日期时间获取设置与修改时会用到）括号中的类是他们的父接口

| 日期时间抽象类与接口                                         | 描述                             |
| ------------------------------------------------------------ | -------------------------------- |
| java.time.temporal.TemporalAccessor                          | 只读接口，用于各维度描述数据     |
| java.time.temporal.Temporal extends TemporalAccessor         | 包装了一层只读接口，增加时间加减 |
| java.time.temporal.TemporalAdjuster                          | 调整Temporal对象的策略【调节器】 |
| java.time.temporal.TemporalAmount                            | 表示定义时间的数量/间隔          |
| java.time.temporal.TemporalUnit                              | 表示日期时间的基本单位           |
| java.time.temporal.TemporalField                             | 表示日期时间的属性               |
| **日期时间类**                                               | **描述**                         |
| java.time.Instant implements Temporal, TemporalAdjuster      | 表示时间戳，默认0时区            |
| java.time.LocalDate implements Temporal, TemporalAdjuster    | 表示日期，不包含时区             |
| java.time.LocalTime implements Temporal, TemporalAdjuster    | 表示时间，不包含时区             |
| java.time.LocalDateTime implements Temporal, TemporalAdjuster | 表示日期和时间，不包含时区       |
| java.time.Year implements Temporal, TemporalAdjuster         | 表示年份                         |
| java.time.YearMonth implements Temporal, TemporalAdjuster    | 表示年份和月份组合               |
| java.time.MonthDay implements TemporalAccessor, TemporalAdjuster | 表示月份和日期的组合             |
| java.time.OffsetDateTime                                     | 表示带偏移量的日期与时间         |
| java.time.OffsetTime                                         | 表示带偏移量的时间               |
| java.time.ZonedDateTime                                      | 表示带时区的日期和时间           |
| **枚举**                                                     | **描述**                         |
| java.time.Month implements TemporalAccessor, TemporalAdjuster | 代表月份                         |
| java.time.DayOfWeek implements TemporalAccessor, TemporalAdjuster | 代表星期几                       |
| java.time.temporal.ChronoField implements TemporalField      | 代表时间字段                     |
| java.time.temporal.ChronoUnit implements TemporalUnit        | 代表字段的单位                   |

java.time 包下主要包含下面几个主要的类：

- Instant：Unix 时间，它代表的是时间戳，比如 2018-01-14T02:20:13.592Z
- Clock：时钟，获取某个时区下的瞬时时间
- LocalDate：表示不带时间的日期，比如：2016-10-20
- LocalTime：表示不带日期的时间，比如：23:12:10
- LocalDateTime：日期时间，比如：2016-10-20 23:14:21
- TemporalAdjuster : 时间调节器
- TemporalAdjusters：获得指定日期时间等，如当月的第一天、今年的最后一天等
- Duration：持续时间，计算两个“时间”的间隔
- Period：日期间隔，计算两个“日期”的间隔
- ZoneId：时区id，例如 Asia/Shangha i等
- ZoneRegion：该类代表地理区域，由于该类不对外开放，实际上就是 ZoneId。例如 Asia/Shanghai 等
- ZoneOffset：时区偏移量，比如：+8:00
- ZonedDateTime：带时区的日期时间
- OffsetTime：存储与UTC没有日期的时间和偏移量
- OffsetDateTime：存储与UTC的日期时间和偏移量
- DateTimeFormatter：时间格式化
- Year 存储年，如“2020”
- YearMonth 存储年和月，如“2020-10”，可用于信用卡上的到期
- Month：定义了一月到十二月的枚举值。存储一个月。如“十一月”
- MonthDay：存储月和日，如“12-14”，可用于存储生日
- DayOfWeek：存储一周中的一天，如“Tuesday”

新的 API 区分各种日期时间概念并且各个概念使用相似的方法定义模式，这种相似性非常有利于 API 的学习。总结一般的方法规律：

- of：静态工厂方法，用于创建实例
- now：静态工厂方法，用当前时间创建实例
- parse：静态工厂方法，从字符串解析得到对象实例
- get：获取对象的部分状态
- is：检查某些东西的是否是 true，例如比较时间前后
- with：返回一个部分状态改变了的时间日期对象拷贝(单独一个with方法,参数为TemporalAdjusters类型)
- plus：返回一个时间增加了的时间日期对象拷贝
- minus：返回一个时间减少了的时间日期对象拷贝
- to：把当前时间日期对象转换成另外一个，可能会损失部分状态。
- at：把这个对象与另一个对象组合起来，例如：date.atTime(time)
- format：将时间日期格式化为字符串



## 新旧日期时间类型转换表及对比

| 类                                         | To遗留类                         | From遗留类                  |
| ------------------------------------------ | -------------------------------- | --------------------------- |
| java.time.Instant java.util.Date           | Date.from(instant)               | date.toInstant()            |
| java.time.Instant java.sql.Timestamp       | Timestamp.from(instant)          | timestamp.toInstant()       |
| java.time.LocalDate java.sql.Date          | Date.valueOf(localDate)          | Date.toLocalDate()          |
| java.time.LocalTime java.sql.Time          | Time.valueOf(localTime)          | Time.toLocalTime()          |
| java.time.LocalDateTime java.sql.Timestamp | Timestamp.valueOf(localDateTime) | Timestamp.toLocalDateTime() |

| Java.time ISO Calendar                              | Java.util Caalendar                                      |
| --------------------------------------------------- | -------------------------------------------------------- |
| Instant                                             | Date                                                     |
| LocalDate、LocalTime、LocalDateTime                 | Calendar                                                 |
| ZonedDateTime                                       | Calendar                                                 |
| OffsetDateTime、OffsetTime                          | Calendar                                                 |
| ZoneId、ZoneOffset、ZoneRules                       | TimeZone                                                 |
| Week Starts on Monday(1..7) enum MONDAY, ..SUNDAY   | Week Starts on Sunday(1..7) enum SUNDAY, ..SATURDAY      |
| 12 Months(1..12)  enum JANUARY, FUBRUARY,..DECEMBER | 12 Months(0..11) int values JANUARY, FUBRUARY,..DECEMBER |

LocalDateTime、ZoneId、Instant、ZonedDateTime 和 long 都可以互相转换：

```
┌─────────────┐                                           ┌─────────────┐ 
│LocalDateTime│────────┐                             ┌────│   Instant   │
└─────────────┘        │       ┌─────────────┐       │    └─────────────┘
                       ├──────>│ZonedDateTime│<──────┤           ↕
┌─────────────┐        │       └─────────────┘       │    ┌─────────────┐
│   ZoneId    │────────┘                             └────│    long     │
└─────────────┘                                           └─────────────┘
```

```
          ┌────────────────────> ┌───────────────┐ <─────────────────┐
          ⬆                      │ LocalDateTime │                   ⬆
          ⬆    ┌───────────────  └───────────────┘  ─────────────┐   ⬆
          ⬆    ⬇               atZone ⬇    ⬆ toLocalDateTime     ⬇   ⬆
          ⬆    ⬇                      ⬇    ⬆         toLocalDate ⬇   ⬆ atTime
   atDate ⬆    ⬇ toLocalTime          ⬇    ⬆      atStartOfDay   ⬇   ⬆
       ┌───────────┐             ┌───────────────┐ ───────> ┌───────────┐
       │ LocalDate │ <────────── │ ZonedDateTime │          │ LocalTime │
       └───────────┘ toLocalTime └───────────────┘ <─────── └───────────┘
                                      ⬇    ⬆      toLocalDate
                                      ⬇    ⬆
 Date.from(zonedDateTime.toInstant()) ⬇    ⬆ toInstant().atZone()
                                    ┌─────────┐
                                    │  Date   │
                                    └─────────┘
```



# 参考文献 & 鸣谢

1. JSR-310（java8 新日期时间API）【**最全教程**】：https://blog.csdn.net/shi_hong_fei_hei/article/details/120477317
2. Java日期时间教程【一点教程】http://www.yiidian.com/java-date
3. java.time包教程【易百教程】https://www.yiibai.com/javatime
4. Java的日期与时间（目录）https://blog.csdn.net/tjgykhulj/article/details/68952451
5. Java方向盘-日期时间：https://blog.csdn.net/f641385712/category_10749009.html、[[YourBatman]-日期时间-公众号目录](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzI0MTUwOTgyOQ==&scene=24&album_id=1696358010555547649&count=3#wechat_redirect)
6. Java8日期时间API【ThinkWon】：https://thinkwon.blog.csdn.net/article/details/111087199
7. **日期时间时区等**：https://blog.csdn.net/kfepiza/category_10948478.html