# Java 7 六个时间处理类

## 时间类说明与对比

| 类名称                                    | 时间格式             |
| ----------------------------------------- | -------------------- |
| java.util.Date（日期类）                  | 年月日时分秒         |
| java.sql.Date extends java.util.Date      | 年月日，没有无参构造 |
| java.sql.Time extends java.util.Date      | 时分秒               |
| java.sql.Timestamp extends java.util.Date | 年月日时分秒毫秒     |
| java.util.Calendar（日历类）              | 年月日时分秒毫秒     |
| java.text.DateFormat（格式化类）          | 时间格式转换类       |

Date: 日期类（其三个字类与其类似，主要是为了兼容数据库字段类型）

- 用来表示瞬时时间，精确到毫秒，他表示的是某一刹那的时间。Date不存在时区概念。
- 是JDK中最早的时间处理类，由于Date类不易于实现国际化，所以逐渐不被推荐使用，并且废弃了大部分方法。

Calendar: 日历类，可以理解为计算机历

- 一方面是为了弥补Date的国际化缺陷，另一方面是将时间以日历的形式来表示和处理
- Calendar包含时区和语言的概念
- Calendar将时间细分成了年月日时分秒毫秒上午下午夏时令等概念，并提供这些概念的相关操作方法。



简单示例（如下显示输出时间08:00:00，是由于CST时区问题，东八区+8小时）：

```java
public class JavaDateTest {
    public static void main(String[] args) {
        java.util.Date date = new java.util.Date();
        java.sql.Date sqlDate = new java.sql.Date(date.getTime());
        java.sql.Time sqlTime = new java.sql.Time(date.getTime());
        java.sql.Timestamp sqlTimestamp = new java.sql.Timestamp(date.getTime());
        System.out.println("【java.util.Date】: " + date); // 【java.util.Date】: Fri Feb 11 13:12:39 CST 2022
        System.out.println("【java.sql.Date】: " + sqlDate); // 【java.sql.Date】: 2022-02-11
        System.out.println("【java.sql.Time】: " + sqlTime); // 【java.sql.Time】: 13:12:39
        System.out.println("【java.sql.Timestamp】: " + sqlTimestamp); // 【java.sql.Timestamp】: 2022-02-11 13:12:39.248

        java.util.Calendar calendar = java.util.Calendar.getInstance();
        System.out.println("【java.util.Calendar】: " + calendar.getTime()); // 【java.util.Calendar】: Fri Feb 11 13:12:39 CST 2022
        
        System.out.println(new java.util.Date(0L)); // Thu Jan 01 08:00:00 CST 1970
        System.out.println(new java.sql.Date(0L)); // 1970-01-01
        System.out.println(new java.sql.Time(0L)); // 08:00:00
        System.out.println(new java.sql.Timestamp(0L)); // 1970-01-01 08:00:00.0
    }
}
```



## 时间戳与时区说明

首先介绍计算机固定起始时间标准与时区：

- 计算机中时间原点：1970年1月1日 00:00:00（GMT：格林尼治标准时间）
- 时间换算单位：1秒 = 1000毫秒
- [UTC](https://baike.baidu.com/item/协调世界时/787659?fromtitle=UTC&fromid=5899996&fr=aladdin) ：统一的标准时间
- [GMT](https://baike.baidu.com/item/世界时/692237?fromtitle=GMT&fromid=6026868&fr=aladdin) ：格林尼治标准时间（Greenwich Mean Time）
- [CST](https://baike.baidu.com/item/CST/14822063) ：可视为中国、美国、澳大利亚或古巴的标准时间
  1. 美国：Central Standard Time (USA) UT-6:00
  2. 澳大利亚：Central Standard Time (Australia) UT+9:30
  3. 中国：China Standard Time UT+8:00
  4. 古巴：Cuba Standard Time UT-4:00

这个静态方法`System.currentTimeMillis()`将返回自1970年1月1日至现在的时间，以毫秒为单位返回一个long类型。例如：

```java
public class JavaDateTest {
    public static void main(String[] args) {
        // 从北京时间1970年1月1日08时00分00秒至今共计的毫秒数
        long time = System.currentTimeMillis(); // 时间戳
        System.out.println(time); // 1644500654429
    }
}
```

这就是全部需要做的了。返回的long值可以用来初始化`java.util.Date`, `java.sql.Date`, `java.sql.Timestamp`和`java.util.GregorianCalendar`对象。

**时间粒度**：事实上`System.currentTimeMillis()`方法的时间粒度是大于1毫秒的。如果你反复执行这个方法，你会发现短时间内得到的结果是相同的，随后又突然在某一次结果增加了几十毫秒（也可能更多）。这是很正常的，毕竟这个方法肯定不是世界上最精密的计时器。



## java.util.Date【超类】

java.util.Date 是 java.sql.Date、java.sql.Time、java.sql.Timestamp 的父类，Java中的时间使用标准类库的 java.util.Date，其表示特定的时间，精确到毫秒。是用距离一个固定时间点的毫秒数（可正可负，long类型）表达一个特定的时间点。从 JDK 1.1 开始，应该使用 java.util.Calendar 类实现日期和时间字段之间转换，使用 java.text.DateFormat 类来格式化和分析日期字符串。因为 Date 的设计具有"千年虫"以及"时区"的问题，所以 Date 中的大部分方法已经不建议使用了，它们都被 java.util.Calendar 类所取代。

java.util.Date 中大部分的方法都已经被标记过期了，现在常用的大致如下：

| 方法名                                 | 类型 | 描述                                                         |
| -------------------------------------- | ---- | ------------------------------------------------------------ |
| public Date()                          | 构造 | 创建一个对应当前时间的 Date                                  |
| public Date(long date)                 | 构造 | 创建指定时间，参数long time是自 1970 年 1 月 1 日 00:00:00 GMT 以来的毫秒数 |
| public long getTime()                  | 普通 | 返回自 1970 年 1 月 1 日 00:00:00 GMT 以来此 Date 对象表示的毫秒数 |
| public long setTime(long time)         | 普通 | 设置Date对象的时间（long型）                                 |
| public String toString()               | 普通 | 显示 年、月、日、时、分、秒。例：Thu Jan 15 14:51:54 CST 1970 |
| public boolean after(Date when)        | 普通 | 如果当前时间对象比when大（晚）则返回true。反之返回false      |
| public boolean before(Date when)       | 普通 | 如果当前时间对象比when小（早）则返回true。反之返回false      |
| public int compareTo(Date anotherDate) | 普通 | 当前时间对比anotherDate，如果大于anotherDate则返回 1，反则 -1，相等返回 0 |

1、首先观察构造方法，可以发现实际上都是将 long 转成 Date 类型

```java
public Date() {
    this(System.currentTimeMillis());
}
public Date(long date) {
    fastTime = date;
}
```

2、观察 toString() 方法，把此 Date 对象转换为以下形式的 String：`dow mon dd hh:mm:ss zzz yyyy`。参数详解：

- dow：是一周中的某一天 （Sun、Mon、Tue、Wed、Thu、Fri、Sat）
- mon：月份，显示 2 位十进制
- zzz：是时区（并可以反映夏令时）标准时区缩写包括方法parse识别的时区缩写。如果不提供时区信息则zzz为空，即无任何字符
- yyyy：是年份，显示为 4 位十进制数

重写了toString每次打印输出java.util.Date类型的对象时，输出的字符串都不是标准的“2018-09-12 21:09:08”这样的形式，而是这样：

```java
Thu Jan 15 14:51:54 CST 1970
```

3、基本方法使用

```java
public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 初始化 Date
         */
        java.util.Date date1 = new java.util.Date();
        // 默认创建一个本地时间, long类型. 表示距离1970年1月1日00:00:00 毫秒值
        java.util.Date date2 = new java.util.Date(System.currentTimeMillis());
        // 创建一个指定毫秒数，从1970-1-1 00:00:00 开始增加 (国内+8小时)
        java.util.Date date3 = new java.util.Date(1000L);
        // 相当于重置时间, 实际上就是: 1970-1-1 00:00:00 (国内输出会+8小时, 可以查看ToString方法)
        java.util.Date date4 = new java.util.Date(0L);

        /**
         * 输出时间
         */
        // 打印出北京时间(ToString 方法会讲时间格式化为系统默认时区的时间输出)
        System.out.println(date1); // Fri Feb 11 13:00:14 CST 2022
        // 打印出格林标准时间（该方法已经过期）
        System.out.println(date2); // Fri Feb 11 13:00:14 CST 2022
        System.out.println(date3); // Thu Jan 01 08:00:01 CST 1970
        // 返回与格林时间的时差, 以分钟计时, 正好是8个小时, 此函数输出-480，则北京时间-480分钟等于格林时间,该方法已经过期
        System.out.println(date1.getTimezoneOffset()); // -480

        /**
         * 打印毫秒数，打印出date到1970年1月1日的毫秒数
         */
        System.out.println(date1.getTime()); // 1644555614732

        /**
         * 比较时间,after()/before()返回boolean类型,compareTo()返回-1 1 0
         */
        System.out.println(date1.after(date3)); // true
        System.out.println(date1.before(date3)); // false
        System.out.println(date1.compareTo(date3)); // 1
    }
}
```

```java
// 输出内容
Fri Feb 11 13:00:14 CST 2022
Fri Feb 11 13:00:14 CST 2022
Thu Jan 01 08:00:01 CST 1970
-480
1644555614732
true
false
1
```

PS：date.getTime()方法 从UTC时间1970年1月1日00时00分00秒（对应北京时间为1970年1月1日08时00分00秒）至今，和时间戳System.currentTimeMillis()功能一致（对应格林尼治时间为1970年1月1日00时00分00秒）



## java.sql.Date【子类】

包含年月日，时分秒都被设置为 0，之所以这样设计是为了适应 SQL 中的 DATE 类型

| 方法名                               | 类型 | 描述                                                         |
| ------------------------------------ | ---- | ------------------------------------------------------------ |
| public Date(long date)               | 构造 | 创建指定date的时间（如果为0则为1970-01-01）                  |
| public String toString()             | 普通 | 格式化日期转义形式 yyyy-mm-dd 的日期，只返回日期，不返回时间 |
| public static Date valueOf(String s) | 普通 | "yyyy-mm-dd" 格式的时间字符串转换为 Date 对象： 格式错误会抛出异常 |

注意：虽然说这个类是使用年月日的，但是初始化的时候，需要一个long类型的参数，这个参数代表着January 1, 1970, 00:00:00 GMT到某个时间的毫秒数。如果是当前时间的话，可以用System.currentTimeMillis()或者new Date().getTime()获取。

```java
public class JavaDateTest {
    public static void main(String[] args) {
        // 返回的均为本地时间
        System.out.println("【java.sql.Date】: " + new java.sql.Date(System.currentTimeMillis()));
        System.out.println("【java.sql.Date】: " + new java.sql.Date(new java.util.Date().getTime()));
        System.out.println(new java.sql.Date(0));
        // 转换成格林时间和util中date输出一样
        System.out.println(new java.sql.Date(System.currentTimeMillis()).toGMTString());
        // 时间格式转换
        System.out.println(java.sql.Date.valueOf("2020-02-10").getTime());
        System.out.println(java.sql.Date.valueOf("20200210"));
    }
}
```

```java
// 输出内容
【java.sql.Date】: 2022-02-11
【java.sql.Date】: 2022-02-11
1970-01-01
11 Feb 2022 08:58:48 GMT
1581264000000
Exception in thread "main" java.lang.IllegalArgumentException
	at java.sql/java.sql.Date.valueOf(Date.java:141)
	at JavaDateTest.main(JavaDateTest.java:11)
```



## java.sql.Time【子类】

包含时分秒，这个也是为了SQL中的 TIME 类型而出现的

| 方法名                               | 类型 | 描述                                                         |
| ------------------------------------ | ---- | ------------------------------------------------------------ |
| public Time(long time)               | 构造 | 创建指定time的时间（如果为0则为 00:00:00 或者 08:00:00(系统为中国时)） |
| public String toString()             | 普通 | 对时间进行格式化，返回 hh:mm:ss 格式，只返回时间，不返回日期 |
| public static Time valueOf(String s) | 普通 | "hh:mm:ss" 格式的时间字符串转换为 Time 对象，格式错误会抛出异常 |

```java
public class JavaDateTest {
    public static void main(String[] args) {
        System.out.println("【java.sql.Time】: " + new java.sql.Time(System.currentTimeMillis()));
        System.out.println("【java.sql.Time】: " + new java.sql.Time(new java.util.Date().getTime()));
        System.out.println(new java.sql.Time(0));
        System.out.println(java.sql.Time.valueOf("20:20:20").getTime());
        System.out.println(java.sql.Time.valueOf("202020"));
    }
}

// 输出内容
【java.sql.Time】: 17:53:45
【java.sql.Time】: 17:53:45
08:00:00
44420000
Exception in thread "main" java.lang.IllegalArgumentException
	at java.sql/java.sql.Time.valueOf(Time.java:109)
	at JavaDateTest.main(JavaDateTest.java:7)
```



## java.sql.Timestamp【子类】

包含年月日时分秒毫秒，时间戳，适配于SQL中的 TIMESTAMP 类型而出现的，精确到纳秒级别。只有 TimeStamp 获取得到的时间才是完整的，Date 获取到的只有日期，Time 获取到的只有时间。

| 方法名                                    | 类型 | 描述                                                         |
| ----------------------------------------- | ---- | ------------------------------------------------------------ |
| public Timestamp(long time)               | 构造 | 创建一个对应指定时间的 Timestamp                             |
| public String toString()                  | 普通 | 返回 yyyy-mm-dd hh:mm:ss.fffffffff 格式                      |
| public static Timestamp valueOf(String s) | 普通 | "yyyy-mm-dd hh:mm:ss[.f...]"格式字符串转换为Timestamp：格式错会抛异常 |

```java
public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 返回的均为本地时间
         */
        System.out.println("【java.sql.Timestamp】: " + new java.sql.Timestamp(System.currentTimeMillis()));
        System.out.println("【java.sql.Timestamp】: " + new java.sql.Timestamp(new java.util.Date().getTime()));
        System.out.println(new java.sql.Timestamp(0));
        System.out.println(new java.sql.Timestamp(System.currentTimeMillis()).toLocalDateTime());
        /**
         * 返回一个格林瞬时时间
         */
        System.out.println(new java.sql.Timestamp(System.currentTimeMillis()).toInstant());
        /**
         * 时间格式转换
         */
        System.out.println(java.sql.Timestamp.valueOf("2020-02-10 20:20:20.110").getTime());
        System.out.println(java.sql.Timestamp.valueOf("202020 202020"));
    }
}
```

```java
// 输出日志
【java.sql.Timestamp】: 2022-02-11 16:56:00.171
【java.sql.Timestamp】: 2022-02-11 16:56:00.191
1970-01-01 08:00:00.0
2022-02-11T16:56:00.191
2022-02-11T08:56:00.201Z
1581337220110
Exception in thread "main" java.lang.IllegalArgumentException: Timestamp format must be yyyy-mm-dd hh:mm:ss[.fffffffff]
	at java.sql/java.sql.Timestamp.valueOf(Timestamp.java:224)
	at JavaDateTest.main(JavaDateTest.java:18)
```

**为什么有了java.util.Date类还要有java.sql.Date这个类呢**？

- 我们发现除了父类Date在util包下，它的三个子类都是在sql包下的。因为在和数据库打交道时（也就是在执行SQL语句时），我们要用到sql包下的时间类，其他情况下我们通常用util包下的Date来格式化日期或者得到当前时间。为什么和数据库打交道时不能用java.util.Date？因为PreparedStatement有三个设置时间的方法：setDate()、setTime()、setTimeStamp()，这三个方法的第二个参数分别是java.sql.Date、java.sql.Time和java.sql.TimeStamp，并没有提供参数是java.util.Date的方法。除此之外，ResultSet也有三个get()方法与之对应，且返回的时间类型也是sql包下的这三个类



## java.util.Calendar【日历类】

在JDK1.1后，处理时间时，推荐使用 Calendar 类，java.util.Calendar 类是一个抽象基类，主要用于日期之间的各种计算。主要完成日期字段之间相互操作的功能，可通过 setTime 方法将 Date 类型变量转换为 Calendar 类型变量，通过 get 方法得知此刻是一年中的第几天、一月中的第几天等，或者通过 set 方法设置此刻时间变为一年中的指定天数等。获取 Calendar 实例的方法：使用 **Calendar.getInstance()** 方法或调用它子类 **GregorianCalendar** 的构造器。注意：**该类不是线程安全的**。

Java.util.Calendar 区别与 java.util.Date 的几个地方也需要注意一下 ：

1. Calendar 增加了毫秒的时间段，通过它可以获取时间点的毫秒值，而 java.util.Date 只是精确到秒
2. 两个类是可以进行相互转换的可以使用 Calendar 类的 setTime(Date date) 和 getTime() 方法

Calendar的常用方法：以下仅仅是Calendar类中的一部分常用方法，更多详细的方法大家可以参考源码。

| 方法名                                   | 类型 | 描述                                                         |
| ---------------------------------------- | ---- | ------------------------------------------------------------ |
| public static Calendar getInstance()     | 静态 | 获得一个日历对象                                             |
| public int get(int field)                | 普通 | 传入field字段获得给定日历字段的值                            |
| public void set(int field, int value)    | 普通 | 设置日历对象代表的时间（Calendar类的可变性）                 |
| public void add(int field, int amount)   | 普通 | 对当前对象日期时间进行加减操作                               |
| public final Date getTime()              | 普通 | 返回一个表示此 Calendar 时间值的 Date 对象，Calendar 类 =》Date 类 |
| public final void setTime(Date date)     | 普通 | 使用给定的 Date 设置此 Calendar 的时间，Date类 =》Calendar 类 |
| public long getTimeInMillis()            | 普通 | 返回此 Calendar 的时间值，以毫秒为单位                       |
| public void setTimeInMillis(long millis) | 普通 | 用给定的 long 值设置此 Calendar 的当前时间值                 |
| public int getActualMaximum(int field)   | 普通 | 传入Calendar.Date，得到这个月可能有多少天                    |

Calendar的常用属性：

| 属性                 | 作用                     |
| -------------------- | ------------------------ |
| YEAR                 | 年份                     |
| MONTH                | 月份                     |
| DATE                 | 同DAY_OF_MONTH，日期     |
| DAY_OF_MONTH         | 同DATE，日期             |
| DAY_OF_YEAR          | 这一天是这一年中的第几天 |
| DAY_OF_WEEK_IN_MONTH | 当前月中的第几周         |
| DAY_OF_WEEK          | 一周中的第几天 / 星期    |
| HOUR                 | 12小时制的时间           |
| HOUR_OF_DAY          | 24小时制的时间           |
| MINUTE               | 分钟数                   |
| SECOND               | 秒数                     |
| MILLISECOND          | 毫秒                     |

```java
import java.util.Calendar;
import java.util.GregorianCalendar;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 实例化的两种方式：
         *   方式一：创建其子类的实例化
         *   方式二：通过调用其静态方法getInstance实例化
         */
        Calendar gregorianCalendar = new GregorianCalendar();
        Calendar calendar = Calendar.getInstance();
        System.out.println("--------------------2022-02-13--------------------");

        /**
         * get()方法的形参是Calendar类中的静态整型常量
         */
        // 获得年份:2022
        System.out.println("现在是：" + calendar.get(Calendar.YEAR) + "年");
        // 获得月份:2
        System.out.println("现在是：" + (calendar.get(Calendar.MONTH)+1) + "月");
        //获得日期(本月的第几天):13
        System.out.println("现在是：" + calendar.get(Calendar.DATE) + "号");
        System.out.println("现在是：" + calendar.get(Calendar.DAY_OF_MONTH) + "号");
        // 获得这是今年的第几天:44
        System.out.println("现在是今年第" + calendar.get(Calendar.DAY_OF_YEAR) + "天");
        // 获得今天周几:0(星期天天)
        System.out.println("现在是星期：" + (calendar.get(Calendar.DAY_OF_WEEK)-1) );
        // 获得今天是这个月的第几周:2
        System.out.println("现在是第：" + calendar.get(Calendar.DAY_OF_WEEK_IN_MONTH) + "周" );
        // 12小时制的时间:9
        System.out.println("现在是：" + calendar.get(Calendar.HOUR) + "点");
        // 24小时制的时间:21
        System.out.println("现在是：" + calendar.get(Calendar.HOUR_OF_DAY) + "点");
        // 分钟数:30
        System.out.println("现在是：" + calendar.get(Calendar.MINUTE) + "分");
        // 秒数:21
        System.out.println("现在是：" + calendar.get(Calendar.SECOND) + "秒");
        // 毫秒:338
        System.out.println("现在是：" + calendar.get(Calendar.MILLISECOND) + "毫秒");

        /**
         * set()方法中的第二个参数可以把Calendar类中的静态常量值改变
         */
        // 设置当前时间为本年度的第10天
        calendar.set(Calendar.DAY_OF_YEAR,10);
        // 再次调用get()方法后返回的是更改后的值
        System.out.println(calendar.get(Calendar.DAY_OF_YEAR)); // 10

        /**
         * add()方法是在set方法的基础上进行增减，如果要减，就把数字写为负数
         */
        // 给当前日期上加上本年度的3天（现在是本年度的第10天）
        calendar.add(Calendar.DAY_OF_YEAR,3);
        System.out.println(calendar.get(Calendar.DAY_OF_YEAR)); // 13

        /**
         * getTime()方法实现 calendar 类 -> Date类
         */
        // 得到的Date类型变量是Calendar对象经过修改后的时间
        java.util.Date time = calendar.getTime();
        System.out.println(time); // Thu Jan 13 21:12:50 CST 2022

        /**
         * setTime()方法实现 Date类 -> Calendar 类
         */
        // 通过Calendar实例对象instance调用setTime将Date类型变量转变为Calendar类型变量
        calendar.setTime(new java.util.Date());
        // 通过该Calendar实例对象调用get方法得知传入Date类型变量是一年中的第几天等信息
        System.out.println(calendar.get(Calendar.DAY_OF_MONTH)); // 13
    }
}
```



## java.text.DateFormat【格式化】

虽然 Date 可以获取当前的日期时间，但是默认情况下 Date 类输出的日期时间结构并不能够被国人所习惯，那么就需要对显示的格式进行格式化的处理，为了可以格式化日期，在 java.text 包中提供有 SimpleDateFormat 程序类，该类是 DateFormat 的子类。其他五种均可以被格式化同种样式的时间（它拥有`date → text` 及`text → date`的能力）。在该类中提供有如下的方法：

| 方法名                                    | 类型 | 描述                                                        |
| ----------------------------------------- | ---- | ----------------------------------------------------------- |
| public final String format(Date date)     | 普通 | 将日期格式化字符串【DateFormat继承】                        |
| public Date parse(String source)          | 普通 | 将字符串转为日期【DateFormat继承】                          |
| public void applyPattern(String pattern)  | 普通 | 设置指定格式化字符串【SimpleDateFormat类的】                |
| public SimpleDateFormat(String pattern)   | 构造 | 构造指定格式的SimpleDateFormat对象                          |
| pattern参数：日期格式（描述是常用的格式） | 参数 | 年(yyyy)、月(MM)、日(dd)、时(HH)、分(mm)、秒(ss)、毫秒(SSS) |



### 1、日期格式类型的匹配字符

日期格式字符串如下（英文版与中文版本）：

| Letter | Date or Time Component                           | Presentation       | Examples                              |
| :----- | :----------------------------------------------- | :----------------- | :------------------------------------ |
| G      | Era designator                                   | Text               | AD                                    |
| y      | Year                                             | Year               | 1996; 96                              |
| Y      | Week year                                        | Year               | 2009; 09                              |
| M      | Month in year (context sensitive)                | Month              | July; Jul; 07                         |
| L      | Month in year (standalone form)                  | Month              | July; Jul; 07                         |
| w      | Week in year                                     | Number             | 27                                    |
| W      | Week in month                                    | Number             | 2                                     |
| D      | Day in year                                      | Number             | 189                                   |
| d      | Day in month                                     | Number             | 10                                    |
| F      | Day of week in month                             | Number             | 2                                     |
| E      | Day name in week                                 | Text               | Tuesday; Tue                          |
| u      | Day number of week (1 = Monday, ..., 7 = Sunday) | Number             | 1                                     |
| a      | Am/pm marker                                     | Text               | PM                                    |
| H      | Hour in day (0-23)                               | Number             | 0                                     |
| k      | Hour in day (1-24)                               | Number             | 24                                    |
| K      | Hour in am/pm (0-11)                             | Number             | 0                                     |
| h      | Hour in am/pm (1-12)                             | Number             | 12                                    |
| m      | Minute in hour                                   | Number             | 30                                    |
| s      | Second in minute                                 | Number             | 55                                    |
| S      | Millisecond                                      | Number             | 978                                   |
| z      | Time zone                                        | General time zone  | Pacific Standard Time; PST; GMT-08:00 |
| Z      | Time zone                                        | RFC 822 time zone  | -0800                                 |
| X      | Time zone                                        | ISO 8601 time zone | -08; -0800; -08:00                    |

| 字母 | 日期或时间元素           | 表示              | 示例                                  |
| ---- | ------------------------ | ----------------- | ------------------------------------- |
| G    | Era designator           | 标志符            | Text                                  |
| y    | 年                       | Year              | 1996; 96                              |
| M    | 年中的月份               | Month             | July; Jul; 07                         |
| w    | 年中的周数               | Number            | 27                                    |
| W    | 月份中的周数             | Number            | 2                                     |
| D    | 年中的天数               | Number            | 189                                   |
| d    | 月份中的天数             | Number            | 10                                    |
| F    | 月份中的星期             | Number            | 2                                     |
| E    | 星期中的天数             | Text              | Tuesday; Tue                          |
| a    | am/pm 标记               | Text              | PM                                    |
| H    | 一天中的小时数（0-23）   | Number            | 0                                     |
| k    | 一天中的小时数（1-24）   | Number            | 24                                    |
| K    | am/pm 中的小时数（0-11） | Number            | 0                                     |
| h    | am/pm 中的小时数（1-12） | Number            | 12                                    |
| m    | 小时中的分钟数           | Number            | 30                                    |
| s    | 分钟中的秒数             | Number            | 55                                    |
| S    | 毫秒数                   | Number            | 978                                   |
| z    | 时区                     | General time zone | Pacific Standard Time; PST; GMT-08:00 |
| Z    | 时区                     | RFC 822 time zone | -0800                                 |

**其中的类型为：**

- **Text：**对于格式化，如果模式字母的数量是4以上，则使用完整的形式; 否则，如果有的话，使用简短或缩写形式。 对于解析，两种形式都是接受的，与模式字母的数量无关。

  ```java
  SimpleDateFormat sdf = new SimpleDateFormat("E");
  SimpleDateFormat sdf1 = new SimpleDateFormat("EE");
  SimpleDateFormat sdf2 = new SimpleDateFormat("EEE");
  SimpleDateFormat sdf3 = new SimpleDateFormat("EEEE");
  
  System.out.println("[java.text.SimpleDateFormat--E] " + sdf.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--EE] " + sdf1.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--EEE] " + sdf2.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--EEEE] " + sdf3.format(new Date()));
  
  // 输出内容:
  [java.text.SimpleDateFormat--E] 周五
  [java.text.SimpleDateFormat--EE] 周五
  [java.text.SimpleDateFormat--EEE] 周五
  [java.text.SimpleDateFormat--EEEE] 星期五
  ```

- **Number：**对于格式化，模式字母的数量是最小位数，而较短的数字将零填充到此数量。 对于解析，模式字母的数量将被忽略，除非需要分隔两个相邻的字段。

  ```java
  SimpleDateFormat sdf4 = new SimpleDateFormat("w");
  SimpleDateFormat sdf5 = new SimpleDateFormat("ww");
  SimpleDateFormat sdf6 = new SimpleDateFormat("www");
  SimpleDateFormat sdf7 = new SimpleDateFormat("wwww");
  
  System.out.println("[java.text.SimpleDateFormat--w] " + sdf4.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--ww] " + sdf5.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--www] " + sdf6.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--wwww] " + sdf7.format(new Date()));
  
  // 输出内容:
  [java.text.SimpleDateFormat--w] 22
  [java.text.SimpleDateFormat--ww] 22
  [java.text.SimpleDateFormat--www] 022
  [java.text.SimpleDateFormat--wwww] 0022
  ```

- **Year：**对于格式化，如果模式字母数为2，那么年份将被截断为2位数; 否则被解释为Number(不足补0)

  ```java
   SimpleDateFormat sdf8 = new SimpleDateFormat("y");
  SimpleDateFormat sdf9 = new SimpleDateFormat("yy");
  SimpleDateFormat sdf10 = new SimpleDateFormat("yyy");
  SimpleDateFormat sdf11 = new SimpleDateFormat("yyyy");
  SimpleDateFormat sdf12 = new SimpleDateFormat("yyyyy");
  
  System.out.println("[java.text.SimpleDateFormat--y] " + sdf8.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--yy] " + sdf9.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--yyy] " + sdf10.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--yyyy] " + sdf11.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--yyyyy] " + sdf12.format(new Date()));
  
  // 输出内容:
  [java.text.SimpleDateFormat--y] 2023
  [java.text.SimpleDateFormat--yy] 23
  [java.text.SimpleDateFormat--yyy] 2023
  [java.text.SimpleDateFormat--yyyy] 2023
  [java.text.SimpleDateFormat--yyyyy] 02023
  ```

- **Month：**如果模式字母数为3以上，则该月份被解释为text; 否则，它被解释为number

  ```java
  SimpleDateFormat sdf8 = new SimpleDateFormat("M");
  SimpleDateFormat sdf9 = new SimpleDateFormat("MM");
  SimpleDateFormat sdf10 = new SimpleDateFormat("MMM");
  SimpleDateFormat sdf11 = new SimpleDateFormat("MMMM");
  SimpleDateFormat sdf12 = new SimpleDateFormat("MMMMM");
  
  System.out.println("[java.text.SimpleDateFormat--M] " + sdf8.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--MM] " + sdf9.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--MMM] " + sdf10.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--MMMM] " + sdf11.format(new Date()));
  System.out.println("[java.text.SimpleDateFormat--MMMMM] " + sdf12.format(new Date()));
  
  // 输出内容:
  [java.text.SimpleDateFormat--M] 6
  [java.text.SimpleDateFormat--MM] 06
  [java.text.SimpleDateFormat--MMM] 6月
  [java.text.SimpleDateFormat--MMMM] 六月
  [java.text.SimpleDateFormat--MMMMM] 六月
  ```

- 其他的类型和时区有关，暂时不去做描述先知道上面的常见的几个



### 2、SimpleDateFormat 常见示例

【操作示例 1】简单时间格式化与解析

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class JavaDateTest {
    public static void main(String[] args) throws ParseException {
        // 1.默认实例化
        SimpleDateFormat sdf1 = new SimpleDateFormat();

        // 1.1 格式化：将Date类型数据转变为字符串
        Date date = new Date();
        // 将新建的日期对象传入默认实例化的SimpleDateFormat对象中，转变为默认格式的字符串 yyyy/MM/dd 上/下午hh:mm
        String format1 = sdf1.format(date);
        // 输出字符串为：2021/12/18 下午6:19
        System.out.println(format1);

        // 1.2 解析：将字符串转换为Date类型数据
        String str = "2021/11/15 下午3:41";
        // 将与SimpleDateFormat中默认日期格式相符合的字符串转换为Date类型数据
        Date parse1 = sdf1.parse(str);
        // 输出Date类型数据为：Mon Nov 15 15:41:00 CST 2021
        System.out.println(parse1);

        // 2.带参数的实例化
        SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        // 2.1 将Date类实例对象作为实参传入SimpleDateFormat重载构造器中，转换为自定义参数格式的字符串
        String format2 = sdf2.format(date);
        // 输出字符串为：2021-11-15 15:41:00
        System.out.println(format2);

        // 2.2 将与SimpleDateFormat类的实例对象中自定义参数格式相符和的字符串解析为Date类型数据
        Date parse2 = sdf2.parse("2021-11-15 16:11:27");
        // 输出Date类型数据为：Mon Nov 15 16:11:27 CST 2021
        System.out.println(parse2);
    }
}
```

> PS：SimpleDateFormat使得可以选择任何用户定义的日期-时间格式的模式。但仍然建议通过DateFormat中的 getTimeInstance、getDateInstance或getDateTimeInstance来新的创建日期-时间格式化程序
>

【操作示例 2】字符串解析为时间格式示例

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

public class JavaDateTest {
    public static void main(String[] args) throws ParseException {
        /**
         * a. 时间格式: “2015-08-28”， 模式: “yyyy-MM-dd”
         */
        {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
            Date date = dateFormat.parse("2015-08-28");
        }
        /**
         * b. 时间格式: “2015-08-28 18:28:30”， 模式: “yyyy-MM-dd HH:mm:ss”
         */
        {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            Date date = dateFormat.parse("2015-08-28 18:28:30");
        }
        /**
         * c. 时间格式: “2015-8-28”， 模式: “yyyy-M-d”
         */
        {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-M-d");
            Date date = dateFormat.parse("2015-8-28");
        }
        /**
         * d. 时间格式: “2015-8-28 18:8:30”， 模式: “yyyy-M-d H:m:s”
         */
        {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-M-d H:m:s");
            Date date = dateFormat.parse("2015-8-28 18:8:30");
        }
        /**
         * e. 时间格式: “Aug 28, 2015 6:8:30 PM”， 模式: “MMM d, yyyy h:m:s aa”
         */
        {
            SimpleDateFormat dateFormat = new SimpleDateFormat("MMM d, yyyy h:m:s aa", Locale.ENGLISH);
            Date date = dateFormat.parse("Aug 28, 2015 6:8:30 PM");
        }
        /**
         * f. 时间格式: “Fri Aug 28 18:08:30 CST 2015”， 模式: “EEE MMM d HH:mm:ss ‘CST’ yyyy”
         */
        {
            SimpleDateFormat dateFormat = new SimpleDateFormat("EEE MMM d HH:mm:ss 'CST' yyyy", Locale.ENGLISH);
            Date date = dateFormat.parse("Fri Aug 28 18:08:30 CST 2015");
        }
    }
}
```



### 2、SimpleDateFormat 最佳实践

**方法一：**在需要执行格式化的地方都新建SimpleDateFormat实例，使用局部变量来存放SimpleDateFormat实例

```java
public static String formatDate(Date date) {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    return sdf.format(date);
}
```

- 这种方法可能会导致短期内创建大量的SimpleDateFormat实例，如解析一个excel表格里的字符串日期。


**方法二：**为了避免创建大量的SimpleDateFormat实例，往往会考虑把SimpleDateFormat实例设为静态成员变量，共享SimpleDateFormat对象。这种情况下就得对SimpleDateFormat添加同步。

```java
private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
public static String formatDate(Date date) {
    synchronized (sdf) {
        return sdf.format(date);
    }
}
```

- 这种方法的缺点也很明显，就是在高并发的环境下会导致解析被阻塞。


**方法三（推荐）：**要在高并发环境下能有比较好的体验，可以使用ThreadLocal来限制SimpleDateFormat只能在线程内共享，这样就避免了多线程导致的线程安全问题。

```java
private static ThreadLocal<DateFormat> threadLocal = new ThreadLocal<DateFormat>() {
    @Override
    protected DateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
};
public static String formatDate(Date date) {
    return threadLocal.get().format(date);
}
```



## java.util.TimeZone【时区】

Java中的java.util.TimeZone类用于表达时区（又称：时区偏移量），Date 本身没有时区概念，当需要使用 SimpleDateFormat 解析与格式化时间，或者使用 Calendar 对时间需要计算的时候，才会涉及时区的概念。

### 1、Date 类本身没有时区概念

查看源码可以知道 Date 对象中存储的是一个 long 型变量，这个变量的值为自 1997-01-01 00:00:00(GMT) 至 Date 对象记录时刻所经过的毫秒数，可以通过 getTime() 方法，获取这个变量值，且这个变量值和时区没有关系，全球任意地点同时执行 new Date().getTime() 获取到的值相同。

```java
public class Date implements java.io.Serializable, Cloneable, Comparable<Date> {
    private transient long fastTime;

    /**
     * Allocates a <code>Date</code> object and initializes it so that
     * it represents the time at which it was allocated, measured to the
     * nearest millisecond.
     *
     * @see     java.lang.System#currentTimeMillis()
     */
    public Date() {
        this(System.currentTimeMillis());
    }

    /**
     * Allocates a <code>Date</code> object and initializes it to
     * represent the specified number of milliseconds since the
     * standard base time known as "the epoch", namely January 1,
     * 1970, 00:00:00 GMT.
     *
     * @param   date   the milliseconds since January 1, 1970, 00:00:00 GMT.
     * @see     java.lang.System#currentTimeMillis()
     */
    public Date(long date) {
        fastTime = date;
    }

    /**
     * Returns the number of milliseconds since January 1, 1970, 00:00:00 GMT
     * represented by this <tt>Date</tt> object.
     *
     * @return  the number of milliseconds since January 1, 1970, 00:00:00 GMT
     *          represented by this date.
     */
    public long getTime() {
        return getTimeImpl();
    }

    public String toString() {
        // "EEE MMM dd HH:mm:ss zzz yyyy";
        BaseCalendar.Date date = normalize();
        StringBuilder sb = new StringBuilder(28);
        int index = date.getDayOfWeek();
        if (index == BaseCalendar.SUNDAY) {
            index = 8;
        }
        convertToAbbr(sb, wtb[index]).append(' ');                        // EEE
        convertToAbbr(sb, wtb[date.getMonth() - 1 + 2 + 7]).append(' ');  // MMM
        CalendarUtils.sprintf0d(sb, date.getDayOfMonth(), 2).append(' '); // dd

        CalendarUtils.sprintf0d(sb, date.getHours(), 2).append(':');   // HH
        CalendarUtils.sprintf0d(sb, date.getMinutes(), 2).append(':'); // mm
        CalendarUtils.sprintf0d(sb, date.getSeconds(), 2).append(' '); // ss
        TimeZone zi = date.getZone();
        if (zi != null) {
            sb.append(zi.getDisplayName(date.isDaylightTime(), TimeZone.SHORT, Locale.US)); // zzz
        } else {
            sb.append("GMT");
        }
        sb.append(' ').append(date.getYear());  // yyyy
        return sb.toString();
    }
}
```

date.getTime()获取毫秒数，获取到的毫秒数和格式化后时间的关系。这个变量的值为自 1997-01-01 00:00:00(GMT) 至 Date 对象记录时刻所经过的毫秒数，这个毫秒数和格式化后时间是模型和视图的关系，时区（TimeZone）决定了同一模型展示成什么样的视图（格式化Date）

```java
public class JavaDateTest {
    public static void main(String[] args) {
        java.util.Date date = new java.util.Date();
        System.out.println(date); // Mon Feb 14 14:19:50 CST 2022
        System.out.println(date.getTime()); // 1644819590161
    }
}
```



### 2、SimpleDateFormat 时间格式化

**1、格式化 Date 对象成字符串**：不管是调用 Date 对象的toString方法，还是使用 SimpleDateFormat的format 方法去格式化Date对象，或者使用 parse 解析字符串成 Date 对象都会涉及到时区，也就是说 Date 对象没有时区概念，但是格式化 Date 对象，或者解析字符串成 Date对 象时，是有时区概念的。

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.TimeZone;

public class JavaDateTest {
    public static void main(String[] args) throws ParseException {
        /**
         * 格式化Date对象成字符串, 涉及时区
         */
        Date date = new Date();
        // 默认是系统时区
        System.out.println(date);
        // 修改默认时区
        TimeZone.setDefault(TimeZone.getTimeZone("GMT"));
        System.out.println(date);

        // 默认是系统时区
        SimpleDateFormat dateFormat1 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(dateFormat1.format(date));
        // 设置时区
        dateFormat1.setTimeZone(TimeZone.getTimeZone("GMT+1:00"));
        System.out.println(dateFormat1.format(date));
    }
}

// 输出内容
Mon Feb 14 14:30:07 CST 2022
Mon Feb 14 06:30:07 GMT 2022
2022-02-14 06:30:07
2022-02-14 07:30:07
```

**2、解析字符串成 Date 对象**：将同一个时间字符串按照不同的时区来解析，得到的Date对象值不一样，很好理解：东八区8点当然和0时区8点不一样。

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.TimeZone;

public class JavaDateTest {
    public static void main(String[] args) throws ParseException {
        /**
         * 解析字符串成Date对象, 涉及时区
         */
        String dateStr = "2022-01-13 08:00:00";
        // 默认是系统时区
        SimpleDateFormat dateFormat2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date date1 = dateFormat2.parse(dateStr);
        System.out.println(dateFormat2.format(date1) + "、" + date1.getTime());

        // 设置时区
        dateFormat2.setTimeZone(TimeZone.getTimeZone("GMT+1:00"));
        Date date2 = dateFormat2.parse(dateStr);
        System.out.println(dateFormat2.format(date1) + "、" + date2.getTime());
    }
}

// 输出内容
2022-01-13 08:00:00、1642032000000
2022-01-13 01:00:00、1642057200000
```



### 3、Calendar 日历类计算

当你进行跨时区的 java.util.Calendar 的计算时，可以尝试使用 java.util.TimeZone 来连接不同时区的 calendar。注意：在 Java 8 中日期时间功能中，时区使用的是 java.time.ZoneId 类，但你只需要在你使用 Java 8 的日期时间类（像 ZonedDateTime 类）的时候，才需要用到 ZoneId。如果你只是使用一个 Calendar 类，那你不妨继续使用 java.util.TimeZone。

```java
import java.util.Calendar;
import java.util.TimeZone;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 获取与设置Calendar对象的时区
         */
        Calendar calendar = Calendar.getInstance();
        TimeZone timeZone = calendar.getTimeZone(); // 获取时区
        calendar.setTimeZone(timeZone); // 设置时区

        /**
         * 创建一个TimeZone的实例对象,有两种方式获得TimeZone实例:
         *   第一种方法返回默认的时区，即系统当前时区。
         *   第二种方法返回指定的地区ID（在这个例子里是Europe/Copenhagen）的时区，在本篇的末尾将附上一个可用地区ID表。
         */
        TimeZone timeZone1 = TimeZone.getDefault();
        TimeZone timeZone2 = TimeZone.getTimeZone("Europe/Copenhagen");

        /**
         * 获取时区的ID、名字和时差(Offset),也可以展示指定TimeZone对象的时区名字、ID和时差：
         *   timeZone.getID();
         *   timeZone.getDisplayName();
         *   timeZone.getOffset(System.currentTimeMillis());
         *   getOffset(time)方法返回的是以毫秒为单位的，与世界标准时间UTC为time的时差
         *   如此段代码若在上海时区运行，将返回28800,000，即正好8个小时，符合预期中国UTC+8的时区
         */
        System.out.println(timeZone.getID()); // 中国标准时间
        System.out.println(timeZone.getDisplayName()); // Asia/Shanghai
        System.out.println(timeZone.getOffset(System.currentTimeMillis())); // 28800000

        /**
         * 不同时区之间的转换：对一个Calendar进行转时区操作
         * 注意虽然当天的时间不同了（11点和19点）但表达的时间（自1970年初到当前时间的毫秒数）是完全相同的。
         * 可见Calendar类可正确完成更改时区这个操作。
         */
        calendar = Calendar.getInstance();
        timeZone1 = TimeZone.getTimeZone("America/Los_Angeles");

        // 当前是默认时区
        long timeCPH = calendar.getTimeInMillis();
        System.out.println("timeCPH  = " + timeCPH); // 1644810416525
        System.out.println("hour     = " + calendar.get(Calendar.HOUR_OF_DAY)); // hour     = 11

        // 设置时区
        calendar.setTimeZone(timeZone1);
        long timeLA = calendar.getTimeInMillis();
        System.out.println("timeLA   = " + timeLA); // timeLA   = 1644810416525
        System.out.println("hour     = " + calendar.get(Calendar.HOUR_OF_DAY)); // hour     = 19
    }
}
```



### 4、TimeZone 类的定义

java.util.TimeZon 是一个抽象类，主要包含了对于时区的各种操作，可以进行计算时间偏移量或夏令时等操作。类声明如下：

```java
public abstract class TimeZone extends Object  implements Serializable, Cloneable {}
```



### 5、TimeZone 类常用方法

| 方法                                                         | 描述                                                      |
| :----------------------------------------------------------- | :-------------------------------------------------------- |
| static String[] getAvailableIDs()                            | 获取所有受支持的ID                                        |
| static String[] getAvailableIDs(int rawOffset)               | 根据给定的时区偏移量（以毫秒为单位）获取可用的ID          |
| static TimeZone getDefault()                                 | 为此主机获取默认的 TimeZone                               |
| String getDisplayName()                                      | 返回适合以默认语言环境呈现给用户的时区名称                |
| String getDisplayName(boolean daylight, int style)           | 返回适合以默认语言环境呈现给用户的时区名称                |
| String getDisplayName(boolean daylight, int style, Locale locale) | 返回此时区的名称，适合在指定的语言环境中呈现给用户        |
| String getDisplayName(Locale locale)                         | 返回此时区的名称，适合在指定的语言环境中呈现给用户        |
| int getDSTSavings()                                          | 返回要添加到本地标准时间以获得本地挂钟时间的时间量。      |
| String getID()                                               | 获取该时区的ID                                            |
| int getOffset(long date)                                     | 返回指定时间与UTC的时区偏移量                             |
| abstract int getRawOffset()                                  | 以毫秒为单位返回添加到UTC以获得该时区中的标准时间的时间量 |
| static TimeZone getTimeZone(String ID)                       | 此方法获取给定ID的TimeZone                                |
| boolean hasSameRules(TimeZone other)                         | 如果此区域具有与另一个区域相同的规则和偏移，则返回true    |
| abstract boolean inDaylightTime(Date date)                   | 此方法查询给定日期是否在该时区的夏令时中                  |
| static void setDefault(TimeZone zone)                        | 此方法设置由getDefault方法返回的TimeZone                  |
| void setID(String ID)                                        | 此方法设置时区ID                                          |
| abstract void setRawOffset(int offsetMillis)                 | 此方法将基准时区偏移量设置为GMT                           |
| abstract boolean useDaylightTime()                           | 此方法查询该时区是否使用夏时制                            |



### 5、TimeZone 类操作示例

【操作示例 1】getAvailableIDs()：获取Java支持的所有时区 ID。

```java
System.out.println(Arrays.toString(TimeZone.getAvailableIDs()));

// Asia/Shanghai, Asia/Chongqing, Asia/Hong_Kong, Asia/Macao, ...
```

【操作示例 2】getAvailableIDs(int rawOffset)：根据 时间偏移量 来获取时区 ID。

```java
// 东八区时间，与标准时间相差8小时
System.out.println(Arrays.toString(TimeZone.getAvailableIDs(8*60*60*1000)));

// [Asia/Brunei, Asia/Choibalsan, Asia/Chongqing, Asia/Chungking, Asia/Harbin,
// Asia/Hong_Kong, Asia/Irkutsk, Asia/Kuala_Lumpur, Asia/Kuching, Asia/Macao, 
// Asia/Macau, Asia/Makassar, Asia/Manila, Asia/Shanghai, Asia/Singapore, 
// Asia/Taipei, Asia/Ujung_Pandang, Asia/Ulaanbaatar, Asia/Ulan_Bator, 
// Australia/Perth, Australia/West, CTT, Etc/GMT-8, Hongkong, PRC, Singapore]
```

【操作示例 3】getDefault()：获取当前系统的默认时区，中国默认为东八区。

```java
System.out.println(TimeZone.getDefault()); 

// sun.util.calendar.ZoneInfo[id="GMT+08:00",offset=28800000,
// dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
```

【操作示例 4】setDefault(TimeZone zone)：设置当前系统的默认时区。

```java
TimeZone.setDefault(TimeZone.getTimeZone("Asia/Shanghai"));
System.out.println(TimeZone.getDefault());

// sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,
// dstSavings=0,useDaylight=false,transitions=19,lastRule=null]
```

【操作示例 5】getTimeZone(String ID)：根据时区 ID 来获取其对应的时区。

```java
System.out.println(TimeZone.getTimeZone("GMT+08:00"));

// sun.util.calendar.ZoneInfo[id="GMT+08:00",offset=28800000,
// dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
```

【操作示例 6】getTimeZone(ZoneId zoneId)：根据 ZoneId 对象来获取其对应的时区。

```java
System.out.println(TimeZone.getTimeZone(ZoneId.of("GMT+08:00")));

// sun.util.calendar.ZoneInfo[id="GMT+08:00",offset=28800000,
// dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
```

【操作示例 7】getDisplayName()：获取该 TimeZone 对象的时区名称。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getDisplayName()); // 中国标准时间
```

【操作示例 8】getDisplayName(Locale locale)：获取该 TimeZone 对象的时区名称，并根据 Locale 对象进行国际化。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getDisplayName()); // 中国标准时间
System.out.println(timeZone.getDisplayName(Locale.ENGL设SH)); // Chi计a Standard Time
```

【操作示例 9】getDisplayName(boolean daylight, int style)：获取该 TimeZone 对象的时区名称。

- daylight：true：指定夏令时名称。false：指定标准时间名称。
- style：TimeZone.LONG：显示全称。TimeZone.SHORT：显示简称。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getDisplayName()); // 中国标准时间
System.out.println(timeZone.getDisplayName(false, TimeZone.LONG)); // 中国标准时间
System.out.println(timeZone.getDisplayName(false, TimeZone.SHORT)); // CST (China Standard Time)
```

【操作示例 10】getDisplayName(boolean daylight, int style, Locale locale)：获取该 TimeZone 对象的时区名称，并根据 Locale 对象进行国际化。

- daylight：true：指定夏令时名称。false：指定标准时间名称。
- style：TimeZone.LONG：显示全称。TimeZone.SHORT：显示简称。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getDisplayName()); // 中国标准时间
System.out.println(timeZone.getDisplayName(false, TimeZone.LONG, Locale.ENGLISH)); // China Standard Time
```

【操作示例 11】getID()：获取该 TimeZone 对象的时区 ID。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getID()); // Asia/Shanghai
```

【操作示例 12】setID(String ID)：设置该 TimeZone 对象的时区 ID。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
timeZone.setID("Asia/Chongqing");
System.out.println(timeZone.getID()); // Asia/Chongqing
```

【操作示例 13】getOffset(long date)：获取该时间所在时区的时间偏移量。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getOffset(System.currentTimeMillis())); // 28800000
```

【操作示例 14】getDSTSavings()：在夏令时规则生效时，返回相对于标准时间提前的毫秒数。如果此时区不实施夏令时，则为 0。

```java
// 中国没有夏令时，故为0
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getDSTSavings()); // 0
```

【操作示例 15】getRawOffset()：获取时间**原始偏移量**，该值不受夏令时的影响，故称为时间原始偏移量。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.getRawOffset()); // 28800000
```

【操作示例 16】setRawOffset(int offsetMillis)：设置时间原始偏移量。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
timeZone.setRawOffset(25200000);
System.out.println(timeZone.getRawOffset()); // 25200000
```

【操作示例 17】toZoneId()：将 TimeZone 对象转换为 ZoneId 对象。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
ZoneId zoneId = timeZone.toZoneId();
System.out.println(zoneId); // Asia/Shanghai
```

【操作示例 18】useDaylightTime()：查询此时区是否使用夏令时。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.useDaylightTime()); // false
```

【操作示例 19】inDaylightTime(Date date)：查询给定的日期是否在此时区的夏令时中。

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
System.out.println(timeZone.inDaylightTime(new Date())); // false
```

【操作示例 20】如果两时区仅时区 ID 不同，但具有相同的规则和时间偏移量，则返回 true。如果另一个时区为空，则返回 false。

```java
TimeZone timeZone1 = TimeZone.getTimeZone("Asia/Shanghai");
TimeZone timeZone2 = TimeZone.getTimeZone("Asia/Chongqing");
System.out.println(timeZone1.hasSameRules(timeZone2)); // true
```





### 6、SpringBoot 设置时区

应用场景：有时候应用服务器的时间是UTC协调世界时间，我们用浏览器访问时就会出现日期快八小时，因为服务器的时区跟我们东八区差八个小时导致的。

1、在启动类中设置时区（在启动类加上 或者 在启动类run方法里加上，两者选其一即可）

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        // 2、在启动类 启动run方法里加上也可以。两种选择其一即可
        // TimeZone.setDefault(TimeZone.getTimeZone("Asia/Shanghai"));
        SpringApplication.run(Application.class, args);
    }
    
    // 1、在启动类加上。两种选择其一即可
    @PostConstruct
    void started() {
        TimeZone.setDefault(TimeZone.getTimeZone("Asia/Shanghai"));
        // TimeZone.setDefault(TimeZone.getTimeZone("GMT+8"));
        // TimeZone.setDefault(TimeZone.getTimeZone("UTC"));
    }
}
```

2、在application.properties加上如下配置

```properties
spring.jackson.date-format = yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone = Asia/Shanghai 
```

3、如果是MySQL（或者其他数据库）中出现时区问题，可在url上加上如下配置

```properties
url: jdbc:mysql://localhost:3306/xxxx?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&autoReconnect=true&serverTimezone=Asia/Shanghai
```



## 时间处理类示例汇总

> 大概可以分为以下五大类
>

### 1、日期取值

获取当前系统时间（毫秒数）

```java
// 方式一
Date date = new Date();
System.out.println(date.getTime()); // 1513749728479

// 方式二
System.out.println(System.currentTimeMillis()); // 1513749728480

// 转换成指定的格式
String current = df.format(System.currentTimeMillis());
System.out.println(current); // 2017-12-20 14:02:08 
```



### 2、日期转换

1、日期转字符串、字符串转日期

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
try {
    // 1、日期转字符串
    Calendar calendar = Calendar.getInstance();
    Date date = calendar.getTime();
    String dateStringParse = sdf.format(date);
    System.out.println(dateStringParse);
    // 2、字符串转日期
    String dateString = "2017-12-20 14:02:08";
    Date dateParse = sdf.parse(dateString);
    System.out.println(dateParse);
} catch (ParseException e) {
    e.printStackTrace();
}
```

2、将日期转换成中文年月日时分秒

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH时mm分ss秒");
try {
    Date date = new Date();
    String dateStringParse = sdf.format(date);
    System.out.println(dateStringParse);
} catch (Exception e) {
    e.printStackTrace();
}
```

3、将指定日期转换成带周的格式

```java
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date d1 = df.parse("2017-12-20 12:19:19");
Calendar cal = df.getCalendar(); // 指定日期
Calendar cas = Calendar.getInstance(); // 当前时间
int year = cal.get(Calendar.YEAR); // 获取年份
int month=cal.get(Calendar.MONTH); // 获取月份
int day=cal.get(Calendar.DATE); // 获取日
int hour=cal.get(Calendar.HOUR); // 小时
int minute=cal.get(Calendar.MINUTE); // 分
int second=cal.get(Calendar.SECOND); // 秒
int WeekOfYear = cal.get(Calendar.DAY_OF_WEEK); // 一周的第几天
System.out.println("现在时间：" + year + "年" + month + "月" + day + "日," + hour + "时" + minute + "分" + second+"秒,星期"+WeekOfYear);
```

4、获取当前时间显示，上午，下午

```java
Date date = new Date();
DateFormat df1 = DateFormat.getDateInstance(); // 日期格式，精确到日
System.out.println(df1.format(date));
DateFormat df2 = DateFormat.getDateTimeInstance(); // 可以精确到时分秒
System.out.println(df2.format(date));
DateFormat df3 = DateFormat.getTimeInstance(); // 只显示出时分秒
System.out.println("只显示出时分秒:"+df3.format(date));
DateFormat df4 = DateFormat.getDateTimeInstance(DateFormat.FULL,DateFormat.FULL); // 显示日期，周，上下午，时间(精确到秒)
System.out.println("显示日期，周，上下午，时间(精确到秒):"+df4.format(date));
DateFormat df5 = DateFormat.getDateTimeInstance(DateFormat.LONG,DateFormat.LONG); // 显示日期,上下午，时间(精确到秒)
System.out.println("显示日期,上下午，时间(精确到秒):"+df5.format(date));
DateFormat df6 = DateFormat.getDateTimeInstance(DateFormat.SHORT,DateFormat.SHORT); // 显示日期，上下午,时间(精确到分)
System.out.println("显示日期，上下午,时间(精确到分):"+df6.format(date));
DateFormat df7 = DateFormat.getDateTimeInstance(DateFormat.MEDIUM,DateFormat.MEDIUM); // 显示日期，时间(精确到分)
System.out.println("显示日期，时间(精确到分):"+df7.format(date));
```

5、时间秒转化为多少天小时分秒

```java
// 秒转化为天小时分秒字符串
public static String formatSeconds(long seconds) {
    String timeStr = seconds + "秒";
    if (seconds > 60) {
        long second = seconds % 60;
        long min = seconds / 60;
        timeStr = min + "分" + second + "秒";
        if (min > 60) {
            min = (seconds / 60) % 60;
            long hour = (seconds / 60) / 60;
            timeStr = hour + "小时" + min + "分" + second + "秒";
            if (hour > 24) {
                hour = ((seconds / 60) / 60) % 24;
                long day = (((seconds / 60) / 60) / 24);
                timeStr = day + "天" + hour + "小时" + min + "分" + second + "秒";
            }
        }
    }
    return timeStr;    
}
```



### 3、设置时间

推荐 使用java.util.Calendar 类来进行操作，因为java.util.Date类很多方法都过时了，Calendar 类有很多重载的设置时间的方法，可以针对于某一项进行设置，也可以同时进行很多设置

1. Calendar.getInstance().getTime()：即可获取一个Date对象
2. Calendar.getInstance().add()：时间的一个部分，正数代表加，负数代表减

```java
// 将给定的日历字段设置为给定值。
set(int field, int value);
// 设置日历字段 YEAR、MONTH 和 DAY_OF_MONTH 的值。
void set(int year, int month, int date); 
// 设置日历字段 YEAR、MONTH、DAY_OF_MONTH、HOUR_OF_DAY 和 MINUTE 的值
void set(int year, int month, int date, int hourOfDay, int minute);
// 设置字段 YEAR、MONTH、DAY_OF_MONTH、HOUR、MINUTE 和 SECOND 的值
void set(int year, int month, int date, int hourOfDay, int minute, int second);
```

第一个就是给特定的字段设值，后面三个就是针对多个字段赋值



### 4、日期加减

通常来说，我们会对日期做两种加减操作：

> 一、以某个日期为基准，计算其几天前/后、几年前/后，或者其他时间单位前后的日期

A、获取当前时间的前一年时间

```java
// 根据现在时间计算
Calendar now = Calendar.getInstance();
now.add(Calendar.YEAR, 1); // 现在时间是1年后
print(now);
now.add(Calendar.YEAR, -1); // 现在时间是1年前
print(now);

// 根据某个特定的时间 date (Date 型) 计算
Calendar specialDate = Calendar.getInstance();
specialDate.setTime(new Date()); // 注意在此处将 specialDate 的值改为特定日期
specialDate.add(Calendar.YEAR, 1); // 特定时间的1年后
print(specialDate);
specialDate.add(Calendar.YEAR, -1); // 特定时间的1年前
print(specialDate); 
```

B、根据时间然后对小时，分钟，秒数进行相加

```java
// 取当前的时分-30
SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm");
Calendar nowTime2 = Calendar.getInstance();
nowTime2.add(Calendar.MINUTE, -30); // 30分钟前的时间
String currentTime = df.format(nowTime2.getTime());
System.out.println("30分钟前的时间" + currentTime);

// 取当前小时减一个小时
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.HOUR_OF_DAY, calendar.get(Calendar.HOUR_OF_DAY) - 1);
SimpleDateFormat dff = new SimpleDateFormat("yyyy-MM-dd HH");
System.out.println("一个小时前的时间：" + dff.format(calendar.getTime()));
System.out.println("当前的时间：" + dff.format(new Date()));

// 取当前的时分+5
Calendar nowTime = Calendar.getInstance();
nowTime.add(Calendar.MINUTE, 5);
String currentTimes = df.format(nowTime.getTime());
System.out.println("当前时间加5分钟" +currentTimes);

// 前一天的时间
Date dNow = new Date(); // 当前时间
Calendar calendars = Calendar.getInstance(); // 得到日历
calendars.setTime(dNow);// 把当前时间赋给日历
calendars.add(Calendar.DAY_OF_MONTH, -1); // 设置为前一天
Date dBefore = calendars.getTime(); // 得到前一天的时间

String defaultStartDate = df.format(dBefore); // 格式化前一天
String defaultEndDate = df.format(dNow); // 格式化当前时间

System.out.println("前一天的时间是：" + defaultStartDate);
System.out.println("生成的时间是：" + defaultEndDate);
```

C、对时分秒进行累计，比如说视频通话，第一次，通了10分钟，第二次就得把上次的时间进行累计

```java
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
// 上次挂断电话时间
Date d1 = df.parse("2017-12-20 12:19:19");
// 这次通话时长
Date d2 = df.parse("2017-12-20 11:40:34");
long nd = 1000 * 24 * 60 * 60;
long nh = 1000 * 60 * 60;
long nm = 1000 * 60;
long ns = 1000;
// 获得两个时间的毫秒时间差异
long diff = d1.getTime() - d2.getTime();
// 计算差多少天
long day = diff / nd;
// 计算差多少小时
long hour = diff % nd / nh;
// 计算差多少分钟
long min = diff % nd % nh / nm;
// 计算差多少秒/输出结果
long sec = diff % nd % nh % nm / ns;
// 根据上次挂断电话时间然后得出这次通话时长
System.out.println(day + "天" + hour + "小时" + min + "分钟"+ sec + "秒");

// 累计通话时长
String cur = "10:15:12";
SimpleDateFormat myFormatter = new SimpleDateFormat("HH:mm:ss");
Calendar cal = Calendar.getInstance();
cal.setTime(myFormatter.parse(cur));
int shi = (int)cal.get(Calendar.HOUR_OF_DAY)+(int)hour;
int fendo = (int)cal.get(Calendar.MINUTE)+(int)min;
int miao = (int)cal.get(Calendar.SECOND)+(int)sec;

// 以下算法有些问题
if(miao>60){
    miao=00;
    if(fendo>60){
        fendo = 00;
        shi = shi+1;
    }else {
        fendo = fendo + 1;
    }
}
System.out.println(shi+":"+fendo+":"+miao);
```

> 二、计算两个时间的间隔

A、例如计算 2017 年 1 月 1 日距离现在有多少天

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String dateString = "2017-01-01 11:11:11";
Calendar calendar = Calendar.getInstance();
long nowDate = calendar.getTime().getTime(); // Date.getTime() 获得毫秒型日期
try {
    long specialDate = sdf.parse(dateString).getTime();
    long betweenDate = (specialDate - nowDate) / (1000 * 60 * 60 * 24); // 计算间隔多少天，则除以毫秒到天的转换公式
    System.out.print(betweenDate);
} catch (ParseException e) {
    e.printStackTrace();
}
```

B、求出两个日期相差多少小时，分钟，毫秒

```java
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date d1 = df.parse("2017-12-20 12:19:19");
Date d2 = df.parse("2017-12-20 11:40:34");
long nd = 1000 * 24 * 60 * 60;
long nh = 1000 * 60 * 60;
long nm = 1000 * 60;
long ns = 1000;
// 获得两个时间的毫秒时间差异
long diff = d1.getTime() - d2.getTime();
// 计算差多少天
long day = diff / nd;
// 计算差多少小时
long hour = diff % nd / nh;
// 计算差多少分钟
long min = diff % nd % nh / nm;
// 计算差多少秒//输出结果
long sec = diff % nd % nh % nm / ns;
System.out.println(day + "天" + hour + "小时" + min + "分钟"+ sec + "秒");
```





### 5、日期比较

日期比较一般有两种方法，对于 java.util.Date 或者 java.util.Calendar 都是通用的。一种是通过 after() 与 before() 方法进行比较，一种是通过 compareTo() 方法进行比较。

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String dateString_01 = "2017-12-20 11:11:11";
String dateString_02 = "2017-12-21 11:11:11";
try {
    Date date_01 = sdf.parse(dateString_01);
    Date date_02 = sdf.parse(dateString_02);
    System.out.println(date_01.before(date_02)); // true，当 date_01 小于 date_02 时，为 true，否则为 false
    System.out.println(date_02.after(date_01)); // true，当 date_02 大于 date_01 时，为 true，否则为 false
    System.out.println(date_01.compareTo(date_02)); // -1，当 date_01 小于 date_02 时，为 -1
    System.out.println(date_02.compareTo(date_01)); // 1，当 date_02 大于 date_01 时，为 1
    System.out.println(date_02.compareTo(date_02)); // 0，当两个日期相等时，为 0
} catch (ParseException e) {
    e.printStackTrace();
}
```



## Date or Calendar 面临的问题

JDK1.0 中就包含 java.util.Date 类，但是它的大多数方法已经在 JDK1.1 引入 Calendar 类后就弃用了。但 Calendar 类也面临如下问题：

1. 可变性：日期和时间这样的类应该是不可变的
2. 偏移性：Date 类中的年份是从1900 年开始的，而月份是从0开始的
3. 格式化：格式化只对 Date 有用，Calendar 则不行
4. 安全性：它们都是线程不安全的，不能处理闰秒问题
4. 设计差：Java 的日期/时间类的定义并不一致，在 java.util 和 java.sql 的包中都有日期类，此外用于格式化和解析的类在 java.text 包中定义。java.util.Date 同时包含日期和时间，而 java.sql.Date 仅包含日期，将其纳入 java.sql 包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
6. 时区处理麻烦：日期类并不提供国际化，没有时区支持，因此 Java 引入了 java.util.Calendar 和j ava.util.TimeZone 类，但他们同样存在上述所有的问题。
5. 为了解决上述问题，Java 8 吸收了第三方 Joda - Time 的精华，创建了新的 java.time。





# 日期时间转换及封装工具类

```java
  final Date date = new Date();
  final Timestamp timestamp = new Timestamp(date.getTime());
  final Calendar calendar = Calendar.getInstance();
  final Instant instant = Instant.now();
  final LocalDateTime localDateTime = LocalDateTime.now();
  final ZonedDateTime zonedDateTime = ZonedDateTime.now();
  final LocalDate localDate = LocalDate.now();
```



## 1、其他时间类型转 Date

```java
final Date date = new Date();

// Timestamp 转 Date
Date timestampDate = new Date(timestamp.getTime());

// Calendar 转 Date
Date calendarDate = calendar.getTime();

// Instant 转 Date
Date instantDate = Date.from(instant);

// LocalDate 转 Date
Date localDateDate = Date.from(localDate.atStartOfDay(ZoneId.systemDefault()).toInstant());

// LocalDateTime 转 Date
Date localDateTimeDate = Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());

// ZonedDateTime 转 Date
Date zonedDateTimeDate = Date.from(zonedDateTime.toInstant());
```



## 2、其他时间类型转 Timestamp

```java
final Timestamp timestamp = new Timestamp(date.getTime());

// Date 转 Timestamp
Timestamp dateTimestamp = new Timestamp(date.getTime());

// Calendar 转 Timestamp
Timestamp calendarTimestamp = new Timestamp(calendar.getTimeInMillis());

// Instant 转 Timestamp
Timestamp instantTimestamp = Timestamp.from(instant);

// LocalDate 转 Timestamp
Timestamp localDateTimestamp = Timestamp.valueOf(localDate.atStartOfDay());

// LocalDateTime 转 Timestamp
Timestamp localDateTimeTimestamp = Timestamp.valueOf(localDateTime);

// ZonedDateTime 转 Timestamp
Timestamp zonedDateTimeTimestamp = Timestamp.from(zonedDateTime.toInstant());
```



## 3、其他时间类型转 Calendar

```java
final Calendar calendar = Calendar.getInstance();

// Date 转 Calendar
Calendar dateCalendar = Calendar.getInstance();
dateCalendar.setTime(date);

// Timestamp 转 Calendar
Calendar timestampCalendar = Calendar.getInstance();
timestampCalendar.setTimeInMillis(timestamp.getTime());

// Instant 转 Calendar
Calendar instantCalendar = GregorianCalendar.from(ZonedDateTime.ofInstant(instant, ZoneId.systemDefault()));

// LocalDate 转 Calendar
Calendar localDateCalendar = GregorianCalendar.from(localDate.atStartOfDay(ZoneId.systemDefault()));

// LocalDateTime 转 Calendar
Calendar localDateTimeCalendar = GregorianCalendar.from(ZonedDateTime.of(localDateTime, ZoneId.systemDefault()));

// ZonedDateTime 转 Calendar
Calendar zonedDateTimeInstantCalendar = GregorianCalendar.from(zonedDateTime);
```



## 4、其他时间类型转 Instant

```java
final Instant instant = Instant.now();

// Date 转 Instant
Instant dateInstant = date.toInstant();

// Timestamp 转 Instant
Instant timestampInstant = timestamp.toInstant();

// Calendar 转 Instant
Instant calendarInstant = calendar.toInstant();

// LocalDateTime 转 Instant
Instant localDateTimeInstant = localDateTime.atZone(ZoneId.systemDefault()).toInstant();

// ZonedDateTime 转 Instant
Instant zonedDateTimeInstant = zonedDateTime.toInstant();

// LocalDate 转 Instant
Instant localDateInstant = localDate.atStartOfDay(ZoneId.systemDefault()).toInstant();
```



## 5、其他时间类型转 LocalDate

```java
final LocalDate localDate = LocalDate.now();

// Date 转 LocalDate (jdk11)
LocalDate dateLocalDate = LocalDate.ofInstant(date.toInstant(), ZoneId.systemDefault());

LocalDate dateLocalDate = LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault()).toLocalDate();

// Timestamp 转 LocalDate
LocalDate timestampLocalDate = timestamp.toLocalDateTime().toLocalDate();

// Calendar 转 LocalDate (jdk11)
LocalDate calendarLocalDate = LocalDate.ofInstant(calendar.toInstant(), ZoneOffset.systemDefault());
LocalDate calendarLocalDate = LocalDateTime.ofInstant(calendar.toInstant(), ZoneOffset.systemDefault()).toLocalDate();

// Instant 转 LocalDate (jdk11)
LocalDate instantLocalDate = LocalDate.ofInstant(instant, ZoneId.systemDefault());
LocalDate instantLocalDate = LocalDateTime.ofInstant(instant, ZoneId.systemDefault()).toLocalDate();

// LocalDateTime 转 LocalDate
LocalDate localDateTimeLocalDate = localDateTime.toLocalDate();

// ZonedDateTime 转 LocalDate
LocalDate zonedDateTimeLocalDate = zonedDateTime.toLocalDate();
```



## 6、其他时间类型转 LocalDateTime

```java
final LocalDateTime localDateTime = LocalDateTime.now();

// Date 转 LocalDateTime
LocalDateTime dateLocalDateTime = LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
LocalDateTime dateLocalDateTime = date.toInstant().atOffset(ZoneId.systemDefault()).toLocalDateTime();

// Timestamp 转 LocalDateTime
LocalDateTime timestampLocalDateTime = timestamp.toLocalDateTime();

// Calendar 转 LocalDateTime
LocalDateTime calendarLocalDateTime = LocalDateTime.ofInstant(calendar.toInstant(), ZoneOffset.systemDefault());

// Instant 转 LocalDateTime
LocalDateTime instantLocalDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());

// LocalDate 转 LocalDateTime
LocalDateTime localDateLocalDateTime = localDate.atStartOfDay();
LocalDateTime localDateLocalDateTime = localDate.atTime(LocalTime.now());

// ZonedDateTime 转 LocalDateTime
LocalDateTime zonedDateTimeLocalDateTime = zonedDateTime.toLocalDateTime();
```



## 7、其他时间类型转 ZonedDateTime

```java
final ZonedDateTime zonedDateTime = ZonedDateTime.now();

// Date 转 ZonedDateTime
ZonedDateTime dateZonedDateTime = ZonedDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());

// Timestamp 转 ZonedDateTime
ZonedDateTime timestampZonedDateTime = ZonedDateTime.ofInstant(timestamp.toInstant(), ZoneId.systemDefault());

// Calendar 转 ZonedDateTime
ZonedDateTime calendarZonedDateTime = ZonedDateTime.ofInstant(calendar.toInstant(), ZoneId.systemDefault());

// Instant 转 ZonedDateTime
ZonedDateTime instantZonedDateTime = instant.atZone(ZoneId.systemDefault());

// LocalDateTime 转 ZonedDateTime
ZonedDateTime localDateTimeZonedDateTime = localDateTime.atZone(ZoneId.systemDefault());

// LocalDate 转 ZonedDateTime
ZonedDateTime localDateZonedDateTime = localDate.atStartOfDay(ZoneId.systemDefault());
```



## 8、其他时间类型转时间戳

```java
// Date 转 时间戳
long date = new Date().getTime();

// Timestamp 转 时间戳
long timestamp = new Timestamp(System.currentTimeMillis()).getTime();

// Calendar 转 时间戳
long calendar = Calendar.getInstance().getTimeInMillis();

// Instant 转 时间戳
long instant = Instant.now().toEpochMilli();

// LocalDate 转 时间戳
long localDate = LocalDate.now().atStartOfDay().toInstant(ZoneOffset.of("+8")).toEpochMilli();
long localDate = LocalDate.now().atStartOfDay().atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();

// LocalDateTime 转 时间戳
long localDateTime = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();
long localDateTime = LocalDateTime.now().atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();

// ZonedDateTime 转 时间戳
long zonedDateTime = ZonedDateTime.now().toInstant().toEpochMilli();
```



## 9、时间戳转其他时间类型

```java
long millis = System.currentTimeMillis();

// 时间戳 转 Date
Date date = new Date(millis);

// 时间戳 转 Timestamp
Timestamp timestamp = new Timestamp(millis);

// 时间戳 转 Calendar
Calendar calendar = Calendar.getInstance();
calendar.setTimeInMillis(millis);

// 时间戳 转 Instant
Instant instant = Instant.ofEpochMilli(millis);

// 时间戳 转 LocalDate
LocalDate localDate = LocalDate.ofInstant(Instant.ofEpochMilli(millis), ZoneId.systemDefault());

// 时间戳 转 LocalDateTime
LocalDateTime localDateTime = LocalDateTime.ofInstant(Instant.ofEpochMilli(millis), ZoneId.systemDefault());

// 时间戳 转 ZonedDateTime
ZonedDateTime zonedDateTime = ZonedDateTime.ofInstant(Instant.ofEpochMilli(millis), ZoneId.systemDefault());
```









## 10、Java 7 时间工具类

```java
import java.sql.Timestamp;
import java.text.ParseException;
import java.text.ParsePosition;
import java.text.SimpleDateFormat;
import java.util.*;

/**
 * @version: V1.0
 * @author: fendo
 * @className: DateUtil
 * @packageName: test
 * @description: 时间操作工具类
 * @data: 2017-12-20 15:36
 **/
public class DateUtil {

    public static String pattern = "yyyy-MM-dd";
    public static SimpleDateFormat formatter = new SimpleDateFormat(pattern);
    public static SimpleDateFormat formatter2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    /**
     * 获取现在时间
     *
     * @return 返回时间类型 yyyy-MM-dd HH:mm:ss
     */
    public static Date getNowDate() {
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateString = formatter.format(new Date());
        ParsePosition pos = new ParsePosition(8);
        Date currentTime_2 = formatter.parse(dateString, pos);
        return currentTime_2;
    }

    /**
     * 获取现在时间
     *
     * @return返回短时间格式 yyyy-MM-dd
     */
    public static Date getNowDateShort() {
        String dateString = formatter.format(new Date());
        ParsePosition pos = new ParsePosition(8);
        Date currentTime_2 = formatter.parse(dateString, pos);
        return currentTime_2;
    }

    /**
     * 获取现在时间
     *
     * @return返回字符串格式 yyyy-MM-dd HH:mm:ss
     */
    public static String getStringDate() {
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateString = formatter.format(new Date());
        return dateString;
    }

    /**
     * 获取现在时间
     *
     * @return返回字符串格式 yyyyMMddHHmmss
     */
    public static String getStringAllDate() {
        SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMddHHmmssSSS");
        String dateString = formatter.format(new Date());
        return dateString;
    }

    /**
     * 获取现在时间
     *
     * @return 返回短时间字符串格式yyyy-MM-dd
     */
    public static String getStringDateShort() {
        String dateString = formatter.format(new Date());
        return dateString;
    }

    /**
     * 获取时间 小时:分;秒 HH:mm:ss
     *
     * @return
     */
    public static String getTimeShort() {
        SimpleDateFormat formatter = new SimpleDateFormat("HH:mm:ss");
        String dateString = formatter.format(new Date());
        return dateString;
    }

    /**
     * 将长时间格式字符串转换为时间 yyyy-MM-dd HH:mm:ss
     *
     * @param strDate
     * @return
     */
    public static Date strToDateLong(String strDate) {
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        ParsePosition pos = new ParsePosition(0);
        Date strtodate = formatter.parse(strDate, pos);
        return strtodate;
    }

    /**
     * 将长时间格式时间转换为字符串 yyyy-MM-dd HH:mm:ss
     *
     * @param dateDate
     * @return
     */
    public static String dateToStrLong(java.util.Date dateDate) {
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateString = formatter.format(dateDate);
        return dateString;
    }

    /**
     * 将短时间格式时间转换为字符串 yyyy-MM-dd
     *
     * @param dateDate
     * @return
     */
    public static String dateToStr(Date dateDate) {
        String dateString = formatter.format(dateDate);
        return dateString;
    }


    /**
     * 将短时间格式字符串转换为时间 yyyy-MM-dd
     *
     * @param strDate
     * @return
     */
    public static Date strToDate(String strDate) {
        ParsePosition pos = new ParsePosition(0);
        Date strtodate = formatter.parse(strDate, pos);
        return strtodate;
    }

    /**
     * 将短时间格式字符串转换为时间 yyyy-MM-dd HH:mm:ss
     *
     * @param strDate
     * @return
     */
    public static Timestamp strToDateSql(String strDate) {
        ParsePosition pos = new ParsePosition(0);
        Date strtodate = formatter2.parse(strDate, pos);
        return new Timestamp(strtodate.getTime());
    }

    /**
     * 得到现在时间
     *
     * @return
     */
    public static Date getNow() {
        Date currentTime = new Date();
        return currentTime;
    }

    /**
     * 提取一个月中的最后一天
     *
     * @param day
     * @return
     */
    public static Date getLastDate(long day) {
        Date date = new Date();
        long date_3_hm = date.getTime() - 3600000 * 34 * day;
        Date date_3_hm_date = new Date(date_3_hm);
        return date_3_hm_date;
    }

    /**
     * 得到现在时间
     *
     * @return 字符串 yyyyMMdd HHmmss
     */
    public static String getStringToday() {
        Date currentTime = new Date();
        SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMdd HHmmss");
        String dateString = formatter.format(currentTime);
        return dateString;
    }

    /**
     * 功能：<br/>
     *
     * @author Tony
     * @version 2016年12月16日 下午4:41:51 <br/>
     */
    public static String getTodayShort() {
        Date currentTime = new Date();
        SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMdd");
        String dateString = formatter.format(currentTime);
        return dateString;
    }

    /**
     * @param @param  value
     * @param @return
     * @return String
     * @throws
     * @Description: 输入一个整数类型的字符串, 然后转换成时分秒的形式输出
     * 例如：输入568
     * 返回结果为：00:09:28
     * 输入null或者“”
     * 返回结果为:00:00:00
     * @author Tony 鬼手卡卡
     * @date 2016-4-20
     */
    public static String getHHMMSS(String value) {
        String hour = "00";
        String minute = "00";
        String second = "00";
        if (value != null && !value.trim().equals("")) {
            int v_int = Integer.valueOf(value);
            hour = v_int / 3600 + "";//获得小时;
            minute = v_int % 3600 / 60 + "";//获得小时;
            second = v_int % 3600 % 60 + "";//获得小时;
        }
        return (hour.length() > 1 ? hour : "0" + hour) + ":" + (minute.length() > 1 ? minute : "0" + minute) + ":" + (second.length() > 1 ? second : "0" + second);
    }

    /**
     * 得到现在小时
     */
    public static String getHour() {
        Date currentTime = new Date();
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateString = formatter.format(currentTime);
        String hour;
        hour = dateString.substring(11, 13);
        return hour;
    }

    /**
     * 得到现在分钟
     *
     * @return
     */
    public static String getTime() {
        Date currentTime = new Date();
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateString = formatter.format(currentTime);
        String min;
        min = dateString.substring(14, 16);
        return min;
    }

    /**
     * 根据用户传入的时间表示格式，返回当前时间的格式 如果是yyyyMMdd，注意字母y不能大写。
     *
     * @param sformat yyyyMMddhhmmss
     * @return
     */
    public static String getUserDate(String sformat) {
        Date currentTime = new Date();
        SimpleDateFormat formatter = new SimpleDateFormat(sformat);
        String dateString = formatter.format(currentTime);
        return dateString;
    }

    /**
     * 二个小时时间间的差值,必须保证二个时间都是"HH:MM"的格式，返回字符型的分钟
     */
    public static String getTwoHour(String st1, String st2) {
        String[] kk = null;
        String[] jj = null;
        kk = st1.split(":");
        jj = st2.split(":");
        if (Integer.parseInt(kk[0]) < Integer.parseInt(jj[0]))
            return "0";
        else {
            double y = Double.parseDouble(kk[0]) + Double.parseDouble(kk[1]) / 60;
            double u = Double.parseDouble(jj[0]) + Double.parseDouble(jj[1]) / 60;
            if ((y - u) > 0)
                return y - u + "";
            else
                return "0";
        }
    }

    /**
     * 得到二个日期间的间隔天数
     */
    public static String getTwoDay(String sj1, String sj2) {
        long day = 0;
        try {
            java.util.Date date = formatter.parse(sj1);
            java.util.Date mydate = formatter.parse(sj2);
            day = (date.getTime() - mydate.getTime()) / (24 * 60 * 60 * 1000);
        } catch (Exception e) {
            return "";
        }
        return day + "";
    }

    /**
     * 时间前推或后推分钟,其中JJ表示分钟.
     */
    public static String getPreTime(String sj1, String jj) {
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String mydate1 = "";
        try {
            Date date1 = format.parse(sj1);
            long Time = (date1.getTime() / 1000) + Integer.parseInt(jj) * 60;
            date1.setTime(Time * 1000);
            mydate1 = format.format(date1);
        } catch (Exception e) {
        }
        return mydate1;
    }

    /**
     * 得到一个时间延后或前移几天的时间,nowdate(yyyy-mm-dd)为时间,delay为前移或后延的天数
     */
    public static String getNextDay(String nowdate, String delay) {
        try {
            String mdate = "";
            Date d = strToDate(nowdate);
            long myTime = (d.getTime() / 1000) + Integer.parseInt(delay) * 24 * 60 * 60;
            d.setTime(myTime * 1000);
            mdate = formatter.format(d);
            return mdate;
        } catch (Exception e) {
            return "";
        }
    }

    /**
     * 功能：<br/> 距离现在几天的时间是多少
     * 获得一个时间字符串，格式为：yyyy-MM-dd HH:mm:ss
     * day  如果为整数，表示未来时间
     * 如果为负数，表示过去时间
     *
     * @author Tony
     * @version 2016年11月29日 上午11:02:56 <br/>
     */
    public static String getFromNow(int day) {
        Date date = new Date();
        long dateTime = (date.getTime() / 1000) + day * 24 * 60 * 60;
        date.setTime(dateTime * 1000);
        return formatter2.format(date);
    }

    /**
     * 判断是否润年
     *
     * @param ddate
     * @return
     */
    public static boolean isLeapYear(String ddate) {

        /**
         * 详细设计： 1.被400整除是闰年，否则： 2.不能被4整除则不是闰年 3.能被4整除同时不能被100整除则是闰年
         * 3.能被4整除同时能被100整除则不是闰年
         */
        Date d = strToDate(ddate);
        GregorianCalendar gc = (GregorianCalendar) Calendar.getInstance();
        gc.setTime(d);
        int year = gc.get(Calendar.YEAR);
        if ((year % 400) == 0)
            return true;
        else if ((year % 4) == 0) {
            if ((year % 100) == 0)
                return false;
            else
                return true;
        } else
            return false;
    }

    /**
     * 返回美国时间格式 26 Apr 2006
     *
     * @param str
     * @return
     */
    public static String getEDate(String str) {
        ParsePosition pos = new ParsePosition(0);
        Date strtodate = formatter.parse(str, pos);
        String j = strtodate.toString();
        String[] k = j.split(" ");
        return k[2] + k[1].toUpperCase() + k[5].substring(2, 4);
    }

    /**
     * 获取一个月的最后一天
     *
     * @param dat
     * @return
     */
    public static String getEndDateOfMonth(String dat) {// yyyy-MM-dd
        String str = dat.substring(0, 8);
        String month = dat.substring(5, 7);
        int mon = Integer.parseInt(month);
        if (mon == 1 || mon == 3 || mon == 5 || mon == 7 || mon == 8 || mon == 10 || mon == 12) {
            str += "31";
        } else if (mon == 4 || mon == 6 || mon == 9 || mon == 11) {
            str += "30";
        } else {
            if (isLeapYear(dat)) {
                str += "29";
            } else {
                str += "28";
            }
        }
        return str;
    }

    /**
     * 判断二个时间是否在同一个周
     *
     * @param date1
     * @param date2
     * @return
     */
    public static boolean isSameWeekDates(Date date1, Date date2) {
        Calendar cal1 = Calendar.getInstance();
        Calendar cal2 = Calendar.getInstance();
        cal1.setTime(date1);
        cal2.setTime(date2);
        int subYear = cal1.get(Calendar.YEAR) - cal2.get(Calendar.YEAR);
        if (0 == subYear) {
            if (cal1.get(Calendar.WEEK_OF_YEAR) == cal2.get(Calendar.WEEK_OF_YEAR))
                return true;
        } else if (1 == subYear && 11 == cal2.get(Calendar.MONTH)) {
            // 如果12月的最后一周横跨来年第一周的话则最后一周即算做来年的第一周
            if (cal1.get(Calendar.WEEK_OF_YEAR) == cal2.get(Calendar.WEEK_OF_YEAR))
                return true;
        } else if (-1 == subYear && 11 == cal1.get(Calendar.MONTH)) {
            if (cal1.get(Calendar.WEEK_OF_YEAR) == cal2.get(Calendar.WEEK_OF_YEAR))
                return true;
        }
        return false;
    }

    /**
     * 产生周序列,即得到当前时间所在的年度是第几周
     *
     * @return
     */
    public static String getSeqWeek() {
        Calendar c = Calendar.getInstance(Locale.CHINA);
        String week = Integer.toString(c.get(Calendar.WEEK_OF_YEAR));
        if (week.length() == 1)
            week = "0" + week;
        String year = Integer.toString(c.get(Calendar.YEAR));
        return year + week;
    }

    /**
     * 获得一个日期所在的周的星期几的日期，如要找出2002年2月3日所在周的星期一是几号
     *
     * @param sdate
     * @param num
     * @return
     */
    public static String getWeek(String sdate, String num) {
        // 再转换为时间
        Date dd = DateUtil.strToDate(sdate);
        Calendar c = Calendar.getInstance();
        c.setTime(dd);
        if (num.equals("1")) // 返回星期一所在的日期
            c.set(Calendar.DAY_OF_WEEK, Calendar.MONDAY);
        else if (num.equals("2")) // 返回星期二所在的日期
            c.set(Calendar.DAY_OF_WEEK, Calendar.TUESDAY);
        else if (num.equals("3")) // 返回星期三所在的日期
            c.set(Calendar.DAY_OF_WEEK, Calendar.WEDNESDAY);
        else if (num.equals("4")) // 返回星期四所在的日期
            c.set(Calendar.DAY_OF_WEEK, Calendar.THURSDAY);
        else if (num.equals("5")) // 返回星期五所在的日期
            c.set(Calendar.DAY_OF_WEEK, Calendar.FRIDAY);
        else if (num.equals("6")) // 返回星期六所在的日期
            c.set(Calendar.DAY_OF_WEEK, Calendar.SATURDAY);
        else if (num.equals("0")) // 返回星期日所在的日期
            c.set(Calendar.DAY_OF_WEEK, Calendar.SUNDAY);
        return new SimpleDateFormat("yyyy-MM-dd").format(c.getTime());
    }

    /**
     * 根据一个日期，返回是星期几的字符串
     *
     * @param sdate
     * @return
     */
    public static String getWeek(String sdate) {
        // 再转换为时间
        Date date = DateUtil.strToDate(sdate);
        Calendar c = Calendar.getInstance();
        c.setTime(date);
        // int hour=c.get(Calendar.DAY_OF_WEEK);
        // hour中存的就是星期几了，其范围 1~7
        // 1=星期日 7=星期六，其他类推
        return new SimpleDateFormat("EEEE").format(c.getTime());
    }

    public static String getWeekStr(String sdate) {
        String str = "";
        str = DateUtil.getWeek(sdate);
        if ("1".equals(str)) {
            str = "星期日";
        } else if ("2".equals(str)) {
            str = "星期一";
        } else if ("3".equals(str)) {
            str = "星期二";
        } else if ("4".equals(str)) {
            str = "星期三";
        } else if ("5".equals(str)) {
            str = "星期四";
        } else if ("6".equals(str)) {
            str = "星期五";
        } else if ("7".equals(str)) {
            str = "星期六";
        }
        return str;
    }

    /**
     * 两个时间之间的天数
     *
     * @param date1
     * @param date2
     * @return
     */
    public static long getDays(String date1, String date2) {
        if (date1 == null || date1.equals(""))
            return 0;
        if (date2 == null || date2.equals(""))
            return 0;
        // 转换为标准时间
        java.util.Date date = null;
        java.util.Date mydate = null;
        try {
            date = formatter.parse(date1);
            mydate = formatter.parse(date2);
        } catch (Exception e) {
        }
        long day = (date.getTime() - mydate.getTime()) / (24 * 60 * 60 * 1000);
        return day;
    }

    /**
     * 形成如下的日历 ， 根据传入的一个时间返回一个结构 星期日 星期一 星期二 星期三 星期四 星期五 星期六 下面是当月的各个时间
     * 此函数返回该日历第一行星期日所在的日期
     *
     * @param sdate
     * @return
     */
    public static String getNowMonth(String sdate) {
        // 取该时间所在月的一号
        sdate = sdate.substring(0, 8) + "01";

        // 得到这个月的1号是星期几
        Date date = DateUtil.strToDate(sdate);
        Calendar c = Calendar.getInstance();
        c.setTime(date);
        int u = c.get(Calendar.DAY_OF_WEEK);
        String newday = DateUtil.getNextDay(sdate, (1 - u) + "");
        return newday;
    }

    /**
     * 取得数据库主键 生成格式为yyyymmddhhmmss+k位随机数
     *
     * @param k 表示是取几位随机数，可以自己定
     */

    public static String getNo(int k) {

        return getUserDate("yyyyMMddhhmmss") + getRandom(k);
    }

    /**
     * 返回一个随机数
     *
     * @param i
     * @return
     */
    public static String getRandom(int i) {
        Random jjj = new Random();
        if (i == 0)
            return "";
        String jj = "";
        for (int k = 0; k < i; k++) {
            jj = jj + jjj.nextInt(9);
        }
        return jj;
    }

    public static boolean RightDate(String date) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        if (date == null)
            return false;
        if (date.length() > 10) {
            sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        } else {
            sdf = new SimpleDateFormat("yyyy-MM-dd");
        }
        try {
            sdf.parse(date);
        } catch (ParseException pe) {
            return false;
        }
        return true;
    }

    /***************************************************************************
     * //nd=1表示返回的值中包含年度 //yf=1表示返回的值中包含月份 //rq=1表示返回的值中包含日期 //format表示返回的格式 1
     * 以年月日中文返回 2 以横线-返回 // 3 以斜线/返回 4 以缩写不带其它符号形式返回 // 5 以点号.返回
     **************************************************************************/
    public static String getStringDateMonth(String sdate, String nd, String yf, String rq, String format) {
        Date currentTime = new Date();
        String dateString = formatter.format(currentTime);
        String s_nd = dateString.substring(0, 4); // 年份
        String s_yf = dateString.substring(5, 7); // 月份
        String s_rq = dateString.substring(8, 10); // 日期
        String sreturn = "";
        //roc.util.MyChar mc = new roc.util.MyChar();
        //if (sdate == null || sdate.equals("") || !mc.Isdate(sdate)) { // 处理空值情况
        if (sdate == null || sdate.equals("")) {
            if (nd.equals("1")) {
                sreturn = s_nd;
                // 处理间隔符
                if (format.equals("1"))
                    sreturn = sreturn + "年";
                else if (format.equals("2"))
                    sreturn = sreturn + "-";
                else if (format.equals("3"))
                    sreturn = sreturn + "/";
                else if (format.equals("5"))
                    sreturn = sreturn + ".";
            }
            // 处理月份
            if (yf.equals("1")) {
                sreturn = sreturn + s_yf;
                if (format.equals("1"))
                    sreturn = sreturn + "月";
                else if (format.equals("2"))
                    sreturn = sreturn + "-";
                else if (format.equals("3"))
                    sreturn = sreturn + "/";
                else if (format.equals("5"))
                    sreturn = sreturn + ".";
            }
            // 处理日期
            if (rq.equals("1")) {
                sreturn = sreturn + s_rq;
                if (format.equals("1"))
                    sreturn = sreturn + "日";
            }
        } else {
            // 不是空值，也是一个合法的日期值，则先将其转换为标准的时间格式
            sdate = getOKDate(sdate);
            s_nd = sdate.substring(0, 4); // 年份
            s_yf = sdate.substring(5, 7); // 月份
            s_rq = sdate.substring(8, 10); // 日期
            if (nd.equals("1")) {
                sreturn = s_nd;
                // 处理间隔符
                if (format.equals("1"))
                    sreturn = sreturn + "年";
                else if (format.equals("2"))
                    sreturn = sreturn + "-";
                else if (format.equals("3"))
                    sreturn = sreturn + "/";
                else if (format.equals("5"))
                    sreturn = sreturn + ".";
            }
            // 处理月份
            if (yf.equals("1")) {
                sreturn = sreturn + s_yf;
                if (format.equals("1"))
                    sreturn = sreturn + "月";
                else if (format.equals("2"))
                    sreturn = sreturn + "-";
                else if (format.equals("3"))
                    sreturn = sreturn + "/";
                else if (format.equals("5"))
                    sreturn = sreturn + ".";
            }
            // 处理日期
            if (rq.equals("1")) {
                sreturn = sreturn + s_rq;
                if (format.equals("1"))
                    sreturn = sreturn + "日";
            }
        }
        return sreturn;
    }

    public static String getNextMonthDay(String sdate, int m) {
        sdate = getOKDate(sdate);
        int year = Integer.parseInt(sdate.substring(0, 4));
        int month = Integer.parseInt(sdate.substring(5, 7));
        month = month + m;
        if (month < 0) {
            month = month + 12;
            year = year - 1;
        } else if (month > 12) {
            month = month - 12;
            year = year + 1;
        }
        String smonth = "";
        if (month < 10)
            smonth = "0" + month;
        else
            smonth = "" + month;
        return year + "-" + smonth + "-10";
    }

    /**
     * 功能：<br/>
     *
     * @author Tony
     * @version 2015-3-31 上午09:29:31 <br/>
     */
    public static String getOKDate(String sdate) {
        if (sdate == null || sdate.equals(""))
            return getStringDateShort();

//    if (!VeStr.Isdate(sdate)) {
//     sdate = getStringDateShort();
//    }
//    // 将“/”转换为“-”
//    sdate = VeStr.Replace(sdate, "/", "-");
        // 如果只有8位长度，则要进行转换
        if (sdate.length() == 8)
            sdate = sdate.substring(0, 4) + "-" + sdate.substring(4, 6) + "-" + sdate.substring(6, 8);
        ParsePosition pos = new ParsePosition(0);
        Date strtodate = formatter.parse(sdate, pos);
        String dateString = formatter.format(strtodate);
        return dateString;
    }

    /**
     * 获取当前时间的前一天时间
     *
     * @param cl
     * @return
     */
    private static String getBeforeDay(Calendar cl) {
        //使用roll方法进行向前回滚
        //cl.roll(Calendar.DATE, -1);
        //使用set方法直接进行设置
        // int day = cl.get(Calendar.DATE);
        cl.add(Calendar.DATE, -1);
        return formatter.format(cl.getTime());
    }

    /**
     * 获取当前时间的后一天时间
     *
     * @param cl
     * @return
     */
    private static String getAfterDay(Calendar cl) {
        //使用roll方法进行回滚到后一天的时间
        //cl.roll(Calendar.DATE, 1);
        //使用set方法直接设置时间值
        //int day = cl.get(Calendar.DATE);
        cl.add(Calendar.DATE, 1);
        return formatter.format(cl.getTime());
    }

    private static String getDateAMPM() {
        GregorianCalendar ca = new GregorianCalendar();
        //结果为“0”是上午     结果为“1”是下午
        int i = ca.get(GregorianCalendar.AM_PM);
        return i == 0 ? "AM" : "PM";
    }

    private static int compareToDate(String date1, String date2) {
        return date1.compareTo(date2);
    }

    private static int compareToDateString(String date1, String date2) {
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        int i = 0;
        try {
            long ldate1 = formatter.parse(date1).getTime();
            long ldate2 = formatter.parse(date2).getTime();
            if (ldate1 > ldate2) {
                i = 1;
            } else if (ldate1 == ldate2) {
                i = 0;
            } else {
                i = -1;
            }

        } catch (ParseException e) {
            e.printStackTrace();
        }
        return i;
    }

    public static String[] getFiveDate() {
        String[] dates = new String[2];
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(new Date());
        String five = " 05:00:00";

        if (getDateAMPM().equals("AM") && compareToDateString(getStringDate(), getStringDateShort() + five) == -1) {
            dates[0] = getBeforeDay(calendar) + five;
            dates[1] = getStringDateShort() + five;
        } else {
            dates[0] = getStringDateShort() + five;
            dates[1] = getAfterDay(calendar) + five;
        }

        return dates;
    }

    public static String getFiveDate2() {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(new Date());
        String five = " 05:00:00";
        String reStr = "";
        if (getDateAMPM().equals("AM") && compareToDateString(getStringDate(), getStringDateShort() + five) == -1) {
            reStr = getBeforeDay(calendar);
        } else {
            reStr = getStringDateShort();
        }
        return reStr;
    }
}
```



## 11、Java 8 时间工具类

```java
public class LocalDateUtils {

    /**
     * 显示年月日时分秒，例如 2015-08-11 09:51:53.
     */
    public static final String DATETIME_PATTERN = "yyyy-MM-dd HH:mm:ss";

    /**
     * 仅显示年月日，例如 2015-08-11.
     */
    public static final String DATE_PATTERN = "yyyy-MM-dd";

    /**
     * 仅显示时分秒，例如 09:51:53.
     */
    public static final String TIME_PATTERN = "HH:mm:ss";

    /**
     * 显示年月日时分秒(无符号)，例如 20150811095153.
     */
    public static final String UNSIGNED_DATETIME_PATTERN = "yyyyMMddHHmmss";

    /**
     * 仅显示年月日(无符号)，例如 20150811.
     */
    public static final String UNSIGNED_DATE_PATTERN = "yyyyMMdd";

    /**
     * 春天;
     */
    public static final Integer SPRING = 1;

    /**
     * 夏天;
     */
    public static final Integer SUMMER = 2;

    /**
     * 秋天;
     */
    public static final Integer AUTUMN = 3;

    /**
     * 冬天;
     */
    public static final Integer WINTER = 4;

    /**
     * 星期日;
     */
    public static final String SUNDAY = "星期日";

    /**
     * 星期一;
     */
    public static final String MONDAY = "星期一";

    /**
     * 星期二;
     */
    public static final String TUESDAY = "星期二";

    /**
     * 星期三;
     */
    public static final String WEDNESDAY = "星期三";

    /**
     * 星期四;
     */
    public static final String THURSDAY = "星期四";

    /**
     * 星期五;
     */
    public static final String FRIDAY = "星期五";

    /**
     * 星期六;
     */
    public static final String SATURDAY = "星期六";

    /**
     * 年
     */
    private static final String YEAR = "year";

    /**
     * 月
     */
    private static final String MONTH = "month";

    /**
     * 周
     */
    private static final String WEEK = "week";

    /**
     * 日
     */
    private static final String DAY = "day";

    /**
     * 时
     */
    private static final String HOUR = "hour";

    /**
     * 分
     */
    private static final String MINUTE = "minute";

    /**
     * 秒
     */
    private static final String SECOND = "second";

    /**
     * 获取当前日期和时间字符串.
     *
     * @return String 日期时间字符串，例如 2015-08-11 09:51:53
     */
    public static String getLocalDateTimeStr() {
        return format(LocalDateTime.now(), DATETIME_PATTERN);
    }

    /**
     * 获取当前日期字符串.
     *
     * @return String 日期字符串，例如2015-08-11
     */
    public static String getLocalDateStr() {
        return format(LocalDate.now(), DATE_PATTERN);
    }

    /**
     * 获取当前时间字符串.
     *
     * @return String 时间字符串，例如 09:51:53
     */
    public static String getLocalTimeStr() {
        return format(LocalTime.now(), TIME_PATTERN);
    }

    /**
     * 获取当前星期字符串.
     *
     * @return String 当前星期字符串，例如 星期二
     */
    public static String getDayOfWeekStr() {
        return format(LocalDate.now(), "E");
    }

    /**
     * 获取指定日期是星期几
     *
     * @param localDate 日期
     * @return String 星期几
     */
    public static String getDayOfWeekStr(LocalDate localDate) {
        String[] weekOfDays = {MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY};
        int dayOfWeek = localDate.getDayOfWeek().getValue() - 1;
        return weekOfDays[dayOfWeek];
    }

    /**
     * 获取日期时间字符串
     *
     * @param temporal 需要转化的日期时间
     * @param pattern  时间格式
     * @return String 日期时间字符串，例如 2015-08-11 09:51:53
     */
    public static String format(TemporalAccessor temporal, String pattern) {
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern);
        return dateTimeFormatter.format(temporal);
    }

    /**
     * 日期时间字符串转换为日期时间(java.time.LocalDateTime)
     *
     * @param localDateTimeStr 日期时间字符串
     * @param pattern          日期时间格式 例如DATETIME_PATTERN
     * @return LocalDateTime 日期时间
     */
    public static LocalDateTime parseLocalDateTime(String localDateTimeStr, String pattern) {
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern);
        return LocalDateTime.parse(localDateTimeStr, dateTimeFormatter);
    }

    /**
     * 日期字符串转换为日期(java.time.LocalDate)
     *
     * @param localDateStr 日期字符串
     * @param pattern      日期格式 例如DATE_PATTERN
     * @return LocalDate 日期
     */
    public static LocalDate parseLocalDate(String localDateStr, String pattern) {
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern);
        return LocalDate.parse(localDateStr, dateTimeFormatter);
    }

    /**
     * 获取指定日期时间加上指定数量日期时间单位之后的日期时间.
     *
     * @param localDateTime 日期时间
     * @param num           数量
     * @param chronoUnit    日期时间单位
     * @return LocalDateTime 新的日期时间
     */
    public static LocalDateTime plus(LocalDateTime localDateTime, int num, ChronoUnit chronoUnit) {
        return localDateTime.plus(num, chronoUnit);
    }

    /**
     * 获取指定日期时间减去指定数量日期时间单位之后的日期时间.
     *
     * @param localDateTime 日期时间
     * @param num           数量
     * @param chronoUnit    日期时间单位
     * @return LocalDateTime 新的日期时间
     */
    public static LocalDateTime minus(LocalDateTime localDateTime, int num, ChronoUnit chronoUnit) {
        return localDateTime.minus(num, chronoUnit);
    }

    /**
     * 根据ChronoUnit计算两个日期时间之间相隔日期时间
     *
     * @param start      开始日期时间
     * @param end        结束日期时间
     * @param chronoUnit 日期时间单位
     * @return long 相隔日期时间
     */
    public static long getChronoUnitBetween(LocalDateTime start, LocalDateTime end, ChronoUnit chronoUnit) {
        return Math.abs(start.until(end, chronoUnit));
    }

    /**
     * 根据ChronoUnit计算两个日期之间相隔年数或月数或天数
     *
     * @param start      开始日期
     * @param end        结束日期
     * @param chronoUnit 日期时间单位,(ChronoUnit.YEARS,ChronoUnit.MONTHS,ChronoUnit.WEEKS,ChronoUnit.DAYS)
     * @return long 相隔年数或月数或天数
     */
    public static long getChronoUnitBetween(LocalDate start, LocalDate end, ChronoUnit chronoUnit) {
        return Math.abs(start.until(end, chronoUnit));
    }

    /**
     * 获取本年第一天的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfYearStr() {
        return getFirstDayOfYearStr(LocalDateTime.now());
    }

    /**
     * 获取本年最后一天的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfYearStr() {
        return getLastDayOfYearStr(LocalDateTime.now());
    }

    /**
     * 获取指定日期当年第一天的日期字符串
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfYearStr(LocalDateTime localDateTime) {
        return getFirstDayOfYearStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期当年最后一天的日期字符串
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfYearStr(LocalDateTime localDateTime) {
        return getLastDayOfYearStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期当年第一天的日期字符串,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @param pattern       日期时间格式
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfYearStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.withDayOfYear(1).withHour(0).withMinute(0).withSecond(0), pattern);
    }

    /**
     * 获取指定日期当年最后一天的日期字符串,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @param pattern       日期时间格式
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfYearStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.with(TemporalAdjusters.lastDayOfYear()).withHour(23).withMinute(59).withSecond(59), pattern);
    }

    /**
     * 获取本月第一天的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfMonthStr() {
        return getFirstDayOfMonthStr(LocalDateTime.now());
    }

    /**
     * 获取本月最后一天的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfMonthStr() {
        return getLastDayOfMonthStr(LocalDateTime.now());
    }

    /**
     * 获取指定日期当月第一天的日期字符串
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getFirstDayOfMonthStr(LocalDateTime localDateTime) {
        return getFirstDayOfMonthStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期当月最后一天的日期字符串
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfMonthStr(LocalDateTime localDateTime) {
        return getLastDayOfMonthStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期当月第一天的日期字符串,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfMonthStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.withDayOfMonth(1).withHour(0).withMinute(0).withSecond(0), pattern);
    }

    /**
     * 获取指定日期当月最后一天的日期字符串,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @param pattern       日期时间格式
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfMonthStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.with(TemporalAdjusters.lastDayOfMonth()).withHour(23).withMinute(59).withSecond(59), pattern);
    }

    /**
     * 获取本周第一天的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfWeekStr() {
        return getFirstDayOfWeekStr(LocalDateTime.now());
    }

    /**
     * 获取本周最后一天的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfWeekStr() {
        return getLastDayOfWeekStr(LocalDateTime.now());
    }

    /**
     * 获取指定日期当周第一天的日期字符串,这里第一天为周一
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfWeekStr(LocalDateTime localDateTime) {
        return getFirstDayOfWeekStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期当周最后一天的日期字符串,这里最后一天为周日
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfWeekStr(LocalDateTime localDateTime) {
        return getLastDayOfWeekStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期当周第一天的日期字符串,这里第一天为周一,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @param pattern       日期时间格式
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getFirstDayOfWeekStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.with(DayOfWeek.MONDAY).withHour(0).withMinute(0).withSecond(0), pattern);
    }

    /**
     * 获取指定日期当周最后一天的日期字符串,这里最后一天为周日,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @param pattern       日期时间格式
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getLastDayOfWeekStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.with(DayOfWeek.SUNDAY).withHour(23).withMinute(59).withSecond(59), pattern);
    }

    /**
     * 获取今天开始时间的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getStartTimeOfDayStr() {
        return getStartTimeOfDayStr(LocalDateTime.now());
    }

    /**
     * 获取今天结束时间的日期字符串
     *
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getEndTimeOfDayStr() {
        return getEndTimeOfDayStr(LocalDateTime.now());
    }

    /**
     * 获取指定日期开始时间的日期字符串
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 00:00:00
     */
    public static String getStartTimeOfDayStr(LocalDateTime localDateTime) {
        return getStartTimeOfDayStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期结束时间的日期字符串
     *
     * @param localDateTime 指定日期时间
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getEndTimeOfDayStr(LocalDateTime localDateTime) {
        return getEndTimeOfDayStr(localDateTime, DATETIME_PATTERN);
    }

    /**
     * 获取指定日期开始时间的日期字符串,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @param pattern       日期时间格式
     * @return String 格式：yyyy-MM-dd HH:mm:ss
     */
    public static String getStartTimeOfDayStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.withHour(0).withMinute(0).withSecond(0), pattern);
    }

    /**
     * 获取指定日期结束时间的日期字符串,带日期格式化参数
     *
     * @param localDateTime 指定日期时间
     * @param pattern       日期时间格式
     * @return String 格式：yyyy-MM-dd 23:59:59
     */
    public static String getEndTimeOfDayStr(LocalDateTime localDateTime, String pattern) {
        return format(localDateTime.withHour(23).withMinute(59).withSecond(59), pattern);
    }

    /**
     * 切割日期。按照周期切割成小段日期段。例如： <br>
     *
     * @param startDate 开始日期（yyyy-MM-dd）
     * @param endDate   结束日期（yyyy-MM-dd）
     * @param period    周期（天，周，月，年）
     * @return 切割之后的日期集合
     * <li>startDate="2019-02-28",endDate="2019-03-05",period="day"</li>
     * <li>结果为：[2019-02-28, 2019-03-01, 2019-03-02, 2019-03-03, 2019-03-04, 2019-03-05]</li><br>
     * <li>startDate="2019-02-28",endDate="2019-03-25",period="week"</li>
     * <li>结果为：[2019-02-28,2019-03-06, 2019-03-07,2019-03-13, 2019-03-14,2019-03-20,
     * 2019-03-21,2019-03-25]</li><br>
     * <li>startDate="2019-02-28",endDate="2019-05-25",period="month"</li>
     * <li>结果为：[2019-02-28,2019-02-28, 2019-03-01,2019-03-31, 2019-04-01,2019-04-30,
     * 2019-05-01,2019-05-25]</li><br>
     * <li>startDate="2019-02-28",endDate="2020-05-25",period="year"</li>
     * <li>结果为：[2019-02-28,2019-12-31, 2020-01-01,2020-05-25]</li><br>
     */
    public static List<String> listDateStrs(String startDate, String endDate, String period) {
        List<String> result = new ArrayList<>();
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(DATE_PATTERN);
        LocalDate end = LocalDate.parse(endDate, dateTimeFormatter);
        LocalDate start = LocalDate.parse(startDate, dateTimeFormatter);
        LocalDate tmp = start;
        switch (period) {
            case DAY:
                while (start.isBefore(end) || start.isEqual(end)) {
                    result.add(start.toString());
                    start = start.plusDays(1);
                }
                break;
            case WEEK:
                while (tmp.isBefore(end) || tmp.isEqual(end)) {
                    if (tmp.plusDays(6).isAfter(end)) {
                        result.add(tmp.toString() + "," + end);
                    } else {
                        result.add(tmp.toString() + "," + tmp.plusDays(6));
                    }
                    tmp = tmp.plusDays(7);
                }
                break;
            case MONTH:
                while (tmp.isBefore(end) || tmp.isEqual(end)) {
                    LocalDate lastDayOfMonth = tmp.with(TemporalAdjusters.lastDayOfMonth());
                    if (lastDayOfMonth.isAfter(end)) {
                        result.add(tmp.toString() + "," + end);
                    } else {
                        result.add(tmp.toString() + "," + lastDayOfMonth);
                    }
                    tmp = lastDayOfMonth.plusDays(1);
                }
                break;
            case YEAR:
                while (tmp.isBefore(end) || tmp.isEqual(end)) {
                    LocalDate lastDayOfYear = tmp.with(TemporalAdjusters.lastDayOfYear());
                    if (lastDayOfYear.isAfter(end)) {
                        result.add(tmp.toString() + "," + end);
                    } else {
                        result.add(tmp.toString() + "," + lastDayOfYear);
                    }
                    tmp = lastDayOfYear.plusDays(1);
                }
                break;
            default:
                break;
        }
        return result;
    }

    public static void main(String[] args) {
        System.out.println(getLocalDateTimeStr());
        System.out.println(getLocalDateStr());
        System.out.println(getLocalTimeStr());
        System.out.println(getDayOfWeekStr());
        System.out.println(getDayOfWeekStr(LocalDate.now()));

        System.out.println("========");
        System.out.println(format(LocalDate.now(), UNSIGNED_DATE_PATTERN));

        System.out.println("========");
        System.out.println(parseLocalDateTime("2020-12-13 11:14:12", DATETIME_PATTERN));
        System.out.println(parseLocalDate("2020-12-13", DATE_PATTERN));

        System.out.println("========");
        System.out.println(plus(LocalDateTime.now(), 3, ChronoUnit.HOURS));
        System.out.println(minus(LocalDateTime.now(), 4, ChronoUnit.DAYS));

        System.out.println("========");
        System.out.println(getChronoUnitBetween(LocalDateTime.now(), parseLocalDateTime("2020-12-12 12:03:12", DATETIME_PATTERN), ChronoUnit.MINUTES));
        System.out.println(getChronoUnitBetween(LocalDate.now(), parseLocalDate("2021-12-12", DATE_PATTERN), ChronoUnit.WEEKS));

        System.out.println("========");
        System.out.println(getFirstDayOfYearStr());
        System.out.println(getFirstDayOfYearStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getFirstDayOfYearStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println(getLastDayOfYearStr());
        System.out.println(getLastDayOfYearStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getLastDayOfYearStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println("========");
        System.out.println(getFirstDayOfMonthStr());
        System.out.println(getFirstDayOfMonthStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getFirstDayOfMonthStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println(getLastDayOfMonthStr());
        System.out.println(getLastDayOfMonthStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getLastDayOfMonthStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println("========");
        System.out.println(getFirstDayOfWeekStr());
        System.out.println(getFirstDayOfWeekStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getFirstDayOfWeekStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println(getLastDayOfWeekStr());
        System.out.println(getLastDayOfWeekStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getLastDayOfWeekStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println("========");
        System.out.println(getStartTimeOfDayStr());
        System.out.println(getStartTimeOfDayStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getStartTimeOfDayStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println(getEndTimeOfDayStr());
        System.out.println(getEndTimeOfDayStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN)));
        System.out.println(getEndTimeOfDayStr(parseLocalDateTime("2021-12-12 12:03:12", DATETIME_PATTERN), UNSIGNED_DATETIME_PATTERN));

        System.out.println("========");
        List<String> dateStrs = listDateStrs("2019-01-30", "2020-12-13", YEAR);
        for (String dateStr : dateStrs) {
            System.out.println(dateStr);
        }

        System.out.println("========");
        List<String> dateStrs1 = listDateStrs("2019-01-30", "2020-12-13", MONTH);
        for (String dateStr : dateStrs1) {
            System.out.println(dateStr);
        }

        System.out.println("========");
        List<String> dateStrs2 = listDateStrs("2020-12-01", "2020-12-13", DAY);
        for (String dateStr : dateStrs2) {
            System.out.println(dateStr);
        }
    }

}
```





# 参考文献 & 鸣谢

Java 时间类：

1. Java的日期与时间（目录）：https://blog.csdn.net/tjgykhulj/article/details/68952451
2. Java方向盘-日期时间（CSDN）：https://blog.csdn.net/f641385712/category_10749009.html
3. Java方向盘-日期时间（公众号）：https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzI0MTUwOTgyOQ==&scene=24&album_id=1696358010555547649&count=3#wechat_redirect
4. ThinkWon：https://thinkwon.blog.csdn.net/article/details/111087199
5. Java日期时间教程 - 一点教程 (yiidian.com)：http://www.yiidian.com/java-date