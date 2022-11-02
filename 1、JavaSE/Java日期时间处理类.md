# Java 7 六个时间类

## 时间类的介绍与对比

| 类名称                           | 时间格式         |
| -------------------------------- | ---------------- |
| java.util.Date（父类）           | 年月日时分秒     |
| java.sql.Date（子类）            | 年月日           |
| java.sql.Time（子类）            | 时分秒           |
| java.sql.Timestamp（子类）       | 年月日时分秒毫秒 |
| java.util.Calendar（日历类）     | 年月日时分秒毫秒 |
| java.text.DateFormat（格式化类） | 时间格式转换类   |

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



## 时间戳与时区的介绍

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



## java.util.Date（超类）

java.util.Date 是 java.sqlDate,Time,Timestamp 的父类，Java中的时间使用标准类库的 java.util.Date，其表示特定的时间，精确到毫秒。是用距离一个固定时间点的毫秒数（可正可负，long类型）表达一个特定的时间点。从 JDK 1.1 开始，应该使用 java.util.Calendar 类实现日期和时间字段之间转换，使用 java.text.DateFormat 类来格式化和分析日期字符串。因为 Date 的设计具有"千年虫"以及"时区"的问题，所以 Date 中的大部分方法已经不建议使用了，它们都被 java.util.Calendar 类所取代。

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
        // 默认创建一个本地时间, long类型.
        java.util.Date date2 = new java.util.Date(System.currentTimeMillis());
        // 创建一个指定毫秒数，从1970-1-1 00:00:00 开始（国内+8小时）
        java.util.Date date3 = new java.util.Date(1000L);

        /**
         * 输出时间
         */
        // 打印出北京时间
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



## java.sql.Date（子类）

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



## java.sql.Time（子类）

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



## java.sql.Timestamp（子类）

包含年月日时分秒毫秒，时间戳，适配于SQL中的 TIMESTAMP 类型而出现的，精确到纳秒级别。
只有 TimeStamp 获取得到的时间才是完整的，Date 获取到的只有日期，Time 获取到的只有时间。

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

> 为什么有了java.util.Date类还要有java.sql.Date这个类呢？
>
> 我们发现除了父类Date在util包下，它的三个子类都是在sql包下的。因为在和数据库打交道时（也就是在执行SQL语句时），我们要用到sql包下的时间类，其他情况下我们通常用util包下的Date来格式化日期或者得到当前时间。为什么和数据库打交道时不能用java.util.Date？因为PreparedStatement有三个设置时间的方法：setDate()、setTime()、setTimeStamp()，这三个方法的第二个参数分别是java.sql.Date、java.sql.Time和java.sql.TimeStamp，并没有提供参数是java.util.Date的方法。除此之外，ResultSet也有三个get()方法与之对应，且返回的时间类型也是sql包下的这三个类



## java.text.DateFormat（格式化）

虽然Date可以获取当前的日期时间，但是默认情况下Date类输出的日期时间结构并不能够被国人所习惯，那么就需要对显示的格式进行格式化的处理，为了可以格式化日期，在java.text包中提供有SimpleDateFormat程序类，该类是DateFormat的子类。其他五种均可以被格式化同种样式的时间（它拥有`date → text` 及`text → date`的能力）。在该类中提供有如下的方法：

| 方法名                                    | 类型 | 描述                                                        |
| ----------------------------------------- | ---- | ----------------------------------------------------------- |
| public final String format(Date date)     | 普通 | 将日期格式化字符串【DateFormat继承】                        |
| public Date parse(String source)          | 普通 | 将字符串转为日期【DateFormat继承】                          |
| public void applyPattern(String pattern)  | 普通 | 设置指定格式化字符串【SimpleDateFormat类的】                |
| public SimpleDateFormat(String pattern)   | 构造 | 构造指定格式的SimpleDateFormat对象                          |
| pattern参数：日期格式（描述是常用的格式） | 参数 | 年(yyyy)、月(MM)、日(dd)、时(HH)、分(mm)、秒(ss)、毫秒(SSS) |

日期格式字符串如下：

| 字母 | 日期或时间元素           | 表示              | 示例                                  |
| ---- | ------------------------ | ----------------- | ------------------------------------- |
| G    | Era                      | 标志符            | Text                                  |
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



### 1、SimpleDateFormat（使用示例）

格式化使用案例：

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



### 2、SimpleDateFormat（正确使用）

方法一：在需要执行格式化的地方都新建SimpleDateFormat实例，使用局部变量来存放SimpleDateFormat实例

```java
public static String formatDate(Date date) {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    return sdf.format(date);
}
```

这种方法可能会导致短期内创建大量的SimpleDateFormat实例，如解析一个excel表格里的字符串日期。

---

方法二：为了避免创建大量的SimpleDateFormat实例，往往会考虑把SimpleDateFormat实例设为静态成员变量，共享SimpleDateFormat对象。这种情况下就得对SimpleDateFormat添加同步。

```java
private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
public static String formatDate(Date date) {
    synchronized (sdf) {
        return sdf.format(date);
    }
}
```

这种方法的缺点也很明显，就是在高并发的环境下会导致解析被阻塞。

---

方法三（**推荐**）：要在高并发环境下能有比较好的体验，可以使用ThreadLocal来限制SimpleDateFormat只能在线程内共享，这样就避免了多线程导致的线程安全问题。

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





## java.util.Calendar（日历类）

java.util.Calendar 类是一个抽象基类，主要用于日期之间的各种计算。主要完成日期字段之间相互操作的功能，可通过setTime方法将Date类型变量转换为Calendar类型变量，通过get方法得知此刻是一年中的第几天、一月中的第几天等，或者通过set方法设置此刻时间变为一年中的指定天数等。获取Calendar实例的方法：使用 **Calendar.getInstance()** 方法或调用它子类 **GregorianCalendar** 的构造器

Java.util.Calendar 区别与 java.util.Date 的几个地方也需要注意一下 ：

1. Calendar增加了毫秒的时间段，通过它可以获取时间点的毫秒值，而java.util.Date只是精确到秒
2. 两个类是可以进行相互转换的可以使用Calendar类的 setTime(Date date) 和 getTime() 方法

Calendar的常用方法：

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



## java.util.TimeZone（时区）

Java中的java.util.TimeZone类用于表达时区（又称：时区偏移量），Date本身没有时区概念，当需要使用SimpleDateFormat解析与格式化时间，或者使用Calendar对时间需要计算的时候，才会涉及时区的概念。

### 1、Date类本身没有时区概念

查看源码可以知道 Date 对象中存储的是一个 long 型变量，这个变量的值为自 1997-01-01 00:00:00(GMT) 至 Date 对象记录时刻所经过的毫秒数，可以通过getTime()方法，获取这个变量值，且这个变量值和时区没有关系，全球任意地点同时执行 `new Date().getTime()` 获取到的值相同。

```java
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



### 2、格式化Date对象成字符串

不管是调用Date对象的toString方法，还是使用SimpleDateFormat的format方法去格式化Date对象，或者使用parse解析字符串成Date对象都会涉及到时区，也就是说Date对象没有时区概念，但是格式化Date对象，或者解析字符串成Date对象时，是有时区概念的。

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



### 3、解析字符串成Date对象

将同一个时间字符串按照不同的时区来解析，得到的Date对象值不一样，很好理解：东八区8点当然和0时区8点不一样。

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



### 4、Calendar日历类计算时

当你进行跨时区的java.util.Calendar的计算时，可以尝试使用java.util.TimeZone来连接不同时区的calendar。
注意：在Java 8中日期时间功能中，时区使用的是`java.time.ZoneId`类，但你只需要在你使用Java 8的日期时间类（像ZonedDateTime类）的时候，才需要用到ZoneId。如果你只是使用一个Calendar类，那你不妨继续使用`java.util.TimeZone`。

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



### 5、TimeZone类的可用时区

```java
public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 以下将列出TimeZone中所有可用的时区ID，你也可以通过TimeZone.getAvailableIDs()获得这个列表.
         */
        java.util.stream.Stream.of(java.util.TimeZone.getAvailableIDs()).forEach(System.out::println);
    }
}

// 输出内容
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
Africa/Asmera
....省略
```



### 6、SpringBoot解决时区问题

应用场景：有时候应用服务器的时间是UTC协调世界时间，我们用浏览器访问时就会出现日期快八小时，因为服务器的时区跟我们东八区差八个小时导致的。

1、在SpringBoot启动类中设置时区（在启动类加上 或者 在启动类run方法里加上，两者选其一即可）

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

3、如果是MySQL中出现时区问题，可在url上加上如下配置

```properties
url: jdbc:mysql://localhost:3306/xxxx?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&autoReconnect=true&serverTimezone=Asia/Shanghai
```



## Java7 时间处理类示例总结

大概可以分为以下五大类：

### 1、日期取值

获取当前系统时间（毫秒数）

```java
// 方式一
Date date = new Date();
System.out.println(date.getTime());

// 方式二
System.out.println(System.currentTimeMillis());

// 转换成指定的格式
String current = df.format(System.currentTimeMillis());
System.out.println(current);	   


输出如下:
1513749728479
1513749728480
2017-12-20 14:02:08 
```



### 2、日期转换

1、日期转字符串、字符串转日期

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
try {
    //1、日期转字符串
    Calendar calendar = Calendar.getInstance();
    Date date = calendar.getTime();
    String dateStringParse = sdf.format(date);
    System.out.println(dateStringParse);
    //2、字符串转日期
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

JDK1.0中就包含java.util.Date类，但是它的大多数方法已经在JDK1.1引入Calendar类后就弃用了。但是Calendar类也面临如下问题：

1. 可变性：日期和时间这样的类应该是不可变的
2. 偏移性：Date类中的年份是从1900年开始的，而月份是从0开始的
3. 格式化：格式化只对Date有用，Calendar则不行
4. 安全性：它们都是线程不安全的，不能处理闰秒问题
4. 设计差：Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
6. 时区处理麻烦：日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题
5. 为了解决上述问题，Java8 吸收了第三方Joda - Time的精华，创建了新的 java.time



# Java 8 时间处理类

## java.time 先导篇

1、Java8 新日期时间API特性：

- 不变性：新的日期/时间API中，所有的类都是不可变的，这对多线程很有好处。
- 关注点分离：借鉴了Joda库的一些优点，新的API将人可读的日期时间和机器时间（unix timestamp）明确分离，它为日期（Date）、时间（Time）、日期时间（DateTime）、时间戳（unix timestamp）以及时区定义了不同的类。
- 清晰：在所有的类中，方法都被明确定义用以完成相同的行为。例如要拿到当前实例我们可以使用now()方法，在所有的类中都定义了format()和parse()方法，而不是像以前那样专门有一个独立的类。为了更好的处理问题，所有的类都使用了工厂模式和策略模式，一旦你使用了其中某个类的方法，与其他类协同工作并不困难。
- 实用操作：所有新的日期/时间API类都实现了一系列方法用以完成通用的任务，如：加、减、格式化、解析、从日期/时间中提取单独部分，等等。
- 可扩展性：新的日期/时间API是工作在ISO-8601日历系统上的，但我们也可以将其应用在非IOS的日历上。

---

2、Java8 新日期时间API包结构：

- java.time：父级基础包，常用的时间相关类都在这里，如LocalDate\LocalDateTime\Instant等
- java.time.chrono：日历系统包，日历相关的接口（类似Calendar）也包括提供对其他日历系统的API
- java.time.format：格式化和解析包，主要类是DateTimeFormatter
- java.time.temporal：扩展功能包，提供细粒度的时间控制field、unit，如weeks、months、month-of-year等
- java.time.zone：时区包，时区规则、本地时区等

---

3、Java8 新日期时间API类：

- 日期和时间、时钟
   1. Instant：时间戳（相对于1970年1月1日的偏移量）默认是0时区，所以需要+8小时才是北京时间，与Date类不同的是其精确到了纳秒级别
   2. LocalDate：只包含日期，不包含时区，比如：2016-10-20
   3. LocalTime：只包含时间，不包含时区，比如：23:12:10
   4. LocalDateTime：包含日期和时间，内部维护着一对LocalDate与LocalTime实例，同时也不包含时区，比如：2016-10-20 23:14:21
   5. Clock：时钟，获取其他地区时钟。比如获取目前美国纽约的时间。该类用于获取指定时区的当前日期、时间。该类可取代System类的currentTimeMillis()方法，而且提供了更多方法来获取当前日期、时间。该类提供了大量静态方法来获取Clock对象。
- 期限/时间段：
   1. Duration：用于计算两个“时间”间隔
   2. Period：用于计算两个“日期”间隔
- 时区与偏移量：
   1. ZoneId：该类代表一个时区
   2. ZoneOffset：时区偏移量，比如：+8:00
   3. ZonedDateTime：带时区的时间
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



## java.time.Instant

Instant 类在Java日期与时间功能中，表示了时间线上一个确切的点（时间戳），定义为距离初始时间的时间差（初始时间为GMT 1970年1月1日00:00），对应Java8之前使用的java.util.Date类，经测量一天有86400秒，从初始时间开始不断向前移动。

| 方法名                                                | 描述                                                        |
| ----------------------------------------------------- | ----------------------------------------------------------- |
| public static Instant now()                           | 通过以UTC（本初子午线）为基准的此刻瞬时时间返回Instannt实例 |
| public static Instant ofEpochMilli(long epochMilli)   | 通过给定毫秒数，获取Instant实例                             |
| public static Instant ofEpochSecond(long epochSecond) | 通过给定秒数，获取Instant实例                               |
| public static Instant parse(final CharSequence text)  | 通过时间字符串转换成Instant实例                             |
| public OffsetDateTime atOffset(ZoneOffset offset)     | 偏移时区，根据时区进行偏移量修正，北京时间应该+8            |
| public ZonedDateTime atZone(ZoneId zone)              | 获取系统默认时区时间的方法，ZoneId是时区的编号              |
| public long getEpochSecond()                          | 获取从1970-01-01 00:00:00到当前时间的秒值（10位秒数）       |
| public long toEpochMilli()                            | 获取从1970-01-01 00:00:00到当前时间的毫秒值（13位毫秒数）   |
| public int getNano()                                  | 把获取到的当前时间的秒数换算成纳秒                          |

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
         * 注:不建议使用Instant查看当前时间点
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



## java.time.Clock

Clock 是时钟系统，用于查找当前时刻。你可以用它来获取某个时区下当前的日期或者时间。可以用 Clock 来替代旧的 System.currentTimeInMillis() 与 TimeZone.getDefault() 方法。

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
         * Clock tickSeconds(ZoneId zone)	此方法获得一个时钟，该时钟使用最佳可用系统时钟返回整秒的当前即时滴答。
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



## java.time.LocalDate

LocalDate 用于表示 “本地日期” 无 “时间”，且不承载时区信息。可以对时间进行加减等操作并返回新副本，是不可变类且线程安全的。
默认格式：yyyy-MM-dd。它经常被用于展示`year-month-day、day-of-year、day-of-week、week-of-year`等格式的信息。

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
        System.out.println(minusLocalDate.minusYears(5)); // 2017-01-01
        System.out.println(minusLocalDate.minusMonths(60)); // 2017-01-01
        System.out.println(minusLocalDate.minusWeeks(260)); // 2017-01-07
        System.out.println(minusLocalDate.minusDays(1826)); // 2017-01-01

        /**
         * 常用日期对比方法：
         *   int compareTo(ChronoLocalDate other) ⽐较当前对象和other对象在时间上的⼤⼩，返回值如果为正，则当前对象时间较晚
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
         *   int compareTo(LocalTime other) ⽐较当前对象和other对象在时间上的⼤⼩，返回值如果为正，则当前对象时间较晚
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

LocalDateTime 用于表示 “本地日期与时间”，且不承载时区信息。可以对时间进行加减等操作并返回副本，也是线程安全并且不可变类。**这个是最常用的类。**
默认的日期格式，yyyy-MM-dd HH:mm:ss.fff，它经常被用于展示`hour-minute-second`格式的信息。

```java
import java.time.*;
import java.time.temporal.ChronoField;
import java.time.temporal.ChronoUnit;

public class JavaDateTest {
    public static void main(String[] args) {

        /**
         * 常用的两种实例化方式：
         *   1.通过静态方法, 获取系统的当前时间：LocalTime.now()
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



## LocalXxx API Summary

1、LocalDateTime、LocalDate、LocalTime 的常用方法总结：

| 方法名                                                       | 描述                                      |
| ------------------------------------------------------------ | ----------------------------------------- |
| now、now(ZoneId zone)                                        | 根据当前时间创建对象、指定时区创建对象    |
| of(int year,int month,int dayOfMonth,int hour,int minute,int second,int nanoOfSecond) | 估计指定日期时间创建对象，没有偏移量      |
| toLocalDate()、toLocalTime()                                 | 将LocalDateTime转换为LocalDate或LocalTime |
| getYear()                                                    | 获得年份                                  |
| getMonth()、getMonthValue()                                  | 获得月份Month枚举和数字（1-12）           |
| getDayOfMonth()、getDayOfYear()                              | 获得月份天数（1-31）和年份天数（1-366）   |
| getDayOfWeek()                                               | 获得星期几DayOfWeek枚举类                 |
| getHour()、getMinute()、getSecond()、getNano()               | 获得时、分、秒、纳秒                      |
| withYear()、withMonth()、withDayOfMonth()、withDayOfYear()   | 设置指定年、月、当月的日、当年的日        |
| withHour()、withMinute()、withSecond()、withNano()           | 设置指定时、分、秒、纳秒                  |
| plusYears()、plusMonths()、plusDays()、plusWeeks()           | 增加指定年、月、日、周                    |
| plusHours()、plusMinutes()、plusSeconds()、plusNanos()       | 增加指定时、分、秒、纳秒                  |
| minusYears()、minusMonths()、minusDays()、minusWeeks()       | 减少指定年、月、日、周                    |
| minusHours()、minusMinutes()、minusSeconds()、minusNanos()   | 减少指定时、分、秒、纳秒                  |
| get(TemporalField field)                                     | 获取指定字段时间                          |
| LocalDateTime with(TemporalField field, long newValue)       | 指定特殊时间                              |
| plus(long amountToAdd, TemporalUnit unit)                    | 增加特殊时间                              |
| minus(long amountToSubtract, TemporalUnit unit)              | 减少特殊时间                              |

2、LocalDateTime、LocalDate、LocalTime 的静态方法 of() 参数取值范围：

| 参数       | 说明                              |
| ---------- | --------------------------------- |
| year       | 从-999 999 999到999 999 999的年份 |
| month      | 一年中的月份，从1到12             |
| dayOfMonth | 月中的天，从1到31                 |
| hour       | 从0到23表示的时                   |
| minute     | 从0到59表示的分                   |
| second     | 从0到59表示的秒                   |



## 日期时间相关枚举类

日期时间相关枚举类（日期时间获取设置与修改时会用到）括号中的类是他们的父接口

| 枚举                                            | 描述           |
| ----------------------------------------------- | -------------- |
| Month（TemporalAccessor, TemporalAdjuster）     | 代表月份       |
| DayOfWeek（TemporalAccessor, TemporalAdjuster） | 代表星期几     |
| ChronoField（TemporalField）                    | 代表时间字段   |
| ChronoUnit（TemporalUnit）                      | 代表字段的单位 |



## java.time 时区先导篇

首先介绍：LocalDateTime、OffsetDateTime 和 ZoneDateTime 之间的关系，且与 ZoneOffset（偏移量）和 ZoneId（时区） 的概念。

```
┌─────────────┬─────────────┬────────────┬────────────┐
│  LocalDate  │  LocalTime  │ ZoneOffset │   ZoneId   │
└─────────────┴─────────────┴────────────┴────────────┘
┌───────────────────────────┐
│       LocalDateTime       │
└───────────────────────────┘
┌────────────────────────────────────────┐
│             OffsetDateTime             │
└────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│                    ZonedDateTime                    │
└─────────────────────────────────────────────────────┘

分割时间（ZonedDateTime）：2022-02-18T20:41:14.164538200+08:00[Asia/Shanghai]
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

ZoneId 是 java.time 引入的新的时区类，注意和旧的 java.util.TimeZone 区别。时区从基准 UTC 开始的一个固定偏移。ZoneId 的子类 ZoneOffset，代表了这种从伦敦格林威治零度子午线开始的时间偏移，也就是时差。并提供在 Instant 和 LocalDateTime 之间进行转换的规则。其中每个时区都对应着 ID，ID的格式为 “区域/城市” 。例如 ：Asia/Shanghai 等。ZoneId：该类中包含了所有的时区信息。

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
        System.out.println(LocalDateTime.now(ZoneId.of("Asia/Kolkata"))); // 2022-02-19T17:24:39.643968100
        System.out.println(LocalDateTime.now(ZoneId.of("Asia/Kolkata"))); // 2022-02-19T17:24:39.643968100
        Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
        // [Asia/Aden, America/Cuiaba, Etc/GMT+9, Etc/GMT+8, ...省略后面输出
        System.out.println(availableZoneIds);
    }
}
```



## java.time.ZoneOffset

java.time.ZoneOffset 类用于表示距UTC时区的固定区域偏移量。它继承了 ZoneId 类并实现 Comparable 接口。ZoneOffset 类声明三个常量：

- MAX：这是支持的最大区域偏移量
- MIN：这是支持的最小区域偏移量
- UTC：这是UTC的时区偏移常数

| 方法                                                         | 描述                                                 |
| :----------------------------------------------------------- | :--------------------------------------------------- |
| Temporal adjustInto(Temporal temporal)                       | 调整指定的时态对象以使其具有与此对象相同的偏移量     |
| static ZoneOffset from(TemporalAccessor temporal)            | 从temporal对象获取ZoneOffset的实例                   |
| int get(TemporalField field)                                 | 从此偏移量中获取指定字段的int值                      |
| String getId()                                               | 获取规范化区域偏移ID                                 |
| long getLong(TemporalField field)                            | 从此偏移量中获取指定字段的long值                     |
| int getTotalSeconds()                                        | 获取以秒为单位的总区域偏移量                         |
| static ZoneOffset of(String offsetId)                        | 使用ID获取ZoneOffset的实例                           |
| static ZoneOffset ofHours(int hours)                         | 使用以小时为单位的偏移量获取ZoneOffset的实例         |
| static ZoneOffset ofHoursMinutes(int hours, int minutes)     | 使用小时和分钟的偏移量获取ZoneOffset的实例           |
| static ZoneOffset ofHoursMinutesSeconds(int hours, int minutes, int seconds) | 使用小时，分钟和秒的偏移量获取ZoneOffset的实例       |
| static ZoneOffset ofTotalSeconds(int totalSeconds)           | 获取ZoneOffset的实例，该实例指定以秒为单位的总偏移量 |

```java
import java.time.*;
import java.time.temporal.Temporal;

public class JavaDateTest {
    public static void main(String[] args) {
        System.out.println(ZoneOffset.MAX); // +18:00
        System.out.println(ZoneOffset.MIN); // -18:00
        ZoneOffset zone = ZoneOffset.UTC;
        System.out.println(zone); // Z
        Temporal temp = zone.adjustInto(ZonedDateTime.now());
        System.out.println(temp); // 2022-02-19T18:55:40.567526+08:00[Asia/Shanghai]
        // 获取5小时偏移量的ZoneOffset对象
        System.out.println(ZoneOffset.ofHours(5)); // +05:00
    }
}
```



## java.time.OffsetDateTime

ISO-8601日历系统中与UTC偏移量有关的日期时间。OffsetDateTime是一个**带有偏移量**的日期时间类型。存储有精确到纳秒的日期时间，以及偏移量。可以简单理解为 OffsetDateTime = LocalDateTime + ZoneOffset。

OffsetDateTime、ZonedDateTime 和 Instant它们三都能在时间线上以纳秒精度存储一个瞬间（注意：LocalDateTime是不行的），也可理解我某个时刻。OffsetDateTime 和 Instant可用于模型的字段类型，因为它们都表示瞬间值并且还不可变，所以适合网络传输或者数据库持久化。

> PS：ZonedDateTime 不适合网络传输/持久化，因为即使同一个 ZoneId 时区，不同地方获取到瞬时值也有可能不一样

```java
import java.time.LocalDateTime;
import java.time.OffsetDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 最大/最小值
         * 偏移量的最大值是+18，最小值是-18，这是由ZoneOffset内部的限制决定的。
         */
        OffsetDateTime min = OffsetDateTime.MIN;
        OffsetDateTime max = OffsetDateTime.MAX;
        // OffsetDateTime最小值：-999999999-01-01T00:00+18:00
        System.out.println("OffsetDateTime最小值：" + min);
        // OffsetDateTime最大值：+999999999-12-31T23:59:59.999999999-18:00
        System.out.println("OffsetDateTime最大值：" + max);
        // +18:00:-999999999-1-1
        System.out.println(min.getOffset() + ":" + min.getYear() + "-" + min.getMonthValue() + "-" + min.getDayOfMonth());
        // -18:00:999999999-12-31
        System.out.println(max.getOffset() + ":" + max.getYear() + "-" + max.getMonthValue() + "-" + max.getDayOfMonth());

        /**
         * ZoneOffset的实例化
         */
        // 当前位置偏移量的本地时间：2022-02-19T22:31:13.539368500+08:00
        System.out.println("当前位置偏移量的本地时间：" + OffsetDateTime.now());
        // 偏移量-4（纽约）的本地时间：：2022-02-19T22:31:13.539368500-04:00
        System.out.println("偏移量-4（纽约）的本地时间：：" + OffsetDateTime.of(LocalDateTime.now(), ZoneOffset.of("-4")));
        // 纽约时区的本地时间：2022-02-19T09:31:13.539368500-05:00
        System.out.println("纽约时区的本地时间：" + OffsetDateTime.now(ZoneId.of("America/New_York")));

        /**
         * 转换：LocalDateTime -> OffsetDateTime
         * 通过此例值得注意的是：LocalDateTime#atOffset()/atZone()只是增加了偏移量/时区，本地时间是并没有改变的。
         * 若想实现本地时间到其它偏移量的对应的时间只能通过其ofInstant()系列构造方法。
         */
        LocalDateTime localDateTime = LocalDateTime.of(2021, 12, 12, 18, 00, 00);
        // 当前时区（北京）时间为：2021-12-12T18:00
        System.out.println("当前时区（北京）时间为：" + localDateTime);
        // 转换为偏移量为 -4的OffsetDateTime时间：(-4地方的晚上18点)
        // -4偏移量地方的晚上18点(方式一)：2021-12-12T18:00-04:00
        System.out.println("-4偏移量地方的晚上18点(方式一)：" + OffsetDateTime.of(localDateTime, ZoneOffset.ofHours(-4)));
        // -4偏移量地方的晚上18点(方式二)：2021-12-12T18:00-04:00
        System.out.println("-4偏移量地方的晚上18点(方式二)：" + localDateTime.atOffset(ZoneOffset.ofHours(-4)));
        // 转换为偏移量为 -4的OffsetDateTime时间：(北京时间晚上18:00 对应的-4地方的时间点)
        // 当前地区对应的-4地方的时间：2021-12-12T06:00-04:00
        System.out.println("当前地区对应的-4地方的时间：" + OffsetDateTime.ofInstant(localDateTime.toInstant(ZoneOffset.ofHours(8)), ZoneOffset.ofHours(-4)));

        /**
         * 转换：OffsetDateTime -> LocalDateTime
         */
        OffsetDateTime offsetDateTime = OffsetDateTime.of(LocalDateTime.now(), ZoneOffset.ofHours(-4));
        // -4偏移量时间为：2022-02-19T22:39:14.442577-04:00
        System.out.println("-4偏移量时间为：" + offsetDateTime);
        // 转为LocalDateTime 注意：时间还是未变的哦
        // LocalDateTime的表示形式：2022-02-19T22:39:14.442577
        System.out.println("LocalDateTime的表示形式：" + offsetDateTime.toLocalDateTime());
    }
}
```



## java.time.ZonedDateTime

ZonedDateTime 其实就是带有时区的 LocalDateTime，其实就是 LocalDateTime + ZoneId，LocalDate、LocalTime、LocalDateTime 是不带时区的。

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public class JavaDateTest {
    public static void main(String[] args) {
        // 获取系统的默认时区编号
        System.out.println(ZoneId.systemDefault());
        // 获取本地默认时区国家的的日期
        System.out.println("本地时区的日期时间:" + LocalDateTime.now()); // 本地时区的日期时间:2022-02-18T19:17:37.416345200

        /**
         * 实例化ZonedDateTime对象：
         * 一种是通过now()方法返回当前时间，一种是通过of()方法放回指定时间。对会带上时区ZoneId对象
         *   ZonedDateTime now()
         *   ZonedDateTime now(ZoneId zone)
         *   ZonedDateTime now(Clock clock)
         *   ZonedDateTime of(LocalDate date, LocalTime time, ZoneId zone)
         *   ZonedDateTime of(LocalDateTime localDateTime, ZoneId zone)
         *   ZonedDateTime of(int year, int month, int dayOfMonth,int hour, int minute, int second, int nanoOfSecond, ZoneId zone)
         */
        ZonedDateTime zbj1 = ZonedDateTime.now(); // 默认时区
        ZonedDateTime zny1 = ZonedDateTime.now(ZoneId.of("America/New_York")); // 用指定时区获取当前时间
        System.out.println(zbj1); // 2022-02-18T20:41:14.164538200+08:00[Asia/Shanghai]
        System.out.println(zny1); // 2022-02-18T07:41:14.164538200-05:00[America/New_York]


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
         * 要转换时区，通过ZonedDateTime的withZoneSameInstant()将关联时区转换到另一个时区，转换后日期和时间都会相应调整。
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



## java.time.Duration/Period

JDK8中提供了两个工具类 Duration/Period：计算日期时间差。它们最大的作用就不需要你自己复杂的计算关于两个年月日之间的相差的时间或日期啦。

- Period ：基于日期值，对应使用 LocalDate ，它们的作用范围域都是日期（年/月/日）
- Duration：基于时间值，对应使用 Instant /LocalTime/LocalDateTime（不包含LocalDate，会报异常），它们的作用范围域都是时间（天/时/分/秒/毫秒/纳秒）注意：Duration 不包含毫秒这个属性

---

Duration 类常用API：

1. 静态方法 between(to, from)：计算两个时间的间隔，默认是秒
2. 静态方法 ofDays()、ofHours()、ofMinutes()、ofSeconds()、ofMillis()、ofNanos()：计算指定时间段
3. 静态方法 of(int amount, ChronoUnit.Xxx) 计算指定时间段
4. 静态方法 from(TemporalAmount amount)：从时间量获取Duration的实例 
5. toDays()：将时间转换为以天为单位的
6. toHours()：将时间转换为以时为单位的
7. toMinutes()：将时间转换为以分钟为单位的
8. toSeconds()：将时间转换为以秒为单位的
9. toMillis()：将时间转换为以毫秒为单位的
10. toNanos()：将时间转换为以纳秒为单位的
11. toXxxPart()：跟如上toXxx方法基本一样，区别就是返回类型由long变成了int，个人认为可能是节约资源
12. withXxx()：设置好前时间段的固定的时间
13. plusXxx()：给当前时间段加上固定的时间
6. minusXxx()：给当前时间段减少固定的时间

Duration 示例代码（表示一个时间段）：

```java
import java.time.Duration;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;

public class JavaDateTest {
    public static void main(String[] args) {
        /**
         * 实例化指定单位的持续时间对象
         * 注意: 默认的打印结果为ISO国际标准化组织规定的日期格式，PT2H中的H，表示Hour小时，M代表Minute分钟，S代表Second秒数
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
         * 获取指定单位的持续时间
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
         *   Duration.between()方法创建Duration对象,注意这个天数是可以负数,意味着如果开始时间比结束时间更后面就会得到负数天数
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
         * Duration时间的加减，可以参考LocalDateTime中的plusXxx、minusXxx和withXxx()系列的方法
         * 注意: Duration包含两部分：seconds秒，nanos纳秒,它们的组合表达了时间长度。所以withXxx()只有withSeconds()和withNanos()方法
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

---

Period 在概念上和 Duration 类似，区别在于 Period 是以年月日来衡量一个时间段。Duration 用于计算两个时间间隔，Period 用于计算两个日期间隔，所以 between() 方法只能接收 LocalDate 类型的参数。

Period 类常用API：

- 静态方法 between()：计算两个日期之间的间隔获取Period实例
- 静态方法 ofYears()、ofMonths()、ofDays()、ofWeeks()：计算指定时间段获取Period实例
- 静态方法 of(int years, int months, int days)：计算指定时间段获取Period实例
- 静态方法 from(TemporalAmount amount)：从时间量获取Period的实例
- getYears()：获取年份
- getMonths()：获取月份
- getDays()：获取天数

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
    }
}
```



## TemporalAdjuster 矫正器

LocalDateTime、Duration、Instant等，尽管已经提供了很多日期加减的方法，但还是比较局限性，如“下个星期天”，“下个结婚生日”，"下个月的第一天"等需求便难以满足。这时我们通过时间校正器效果可能会更好。

- TemporalAdjuster：时间校正器
- TemporalAdjusters：通过该类静态方法提供了大量的常用 TemporalAdjuster 的实现

TemporalAdjusters 中包含许多静态方法，可以直接调用，以下列举一些：

| 方法名                                                      | 描述                                                        |
| :---------------------------------------------------------- | :---------------------------------------------------------- |
| dayOfWeekInMonth                                            | 返回同一个月中每周的第几天                                  |
| firstDayOfMonth                                             | 返回当月的第一天                                            |
| firstDayOfNextMonth                                         | 返回下月的第一天                                            |
| firstDayOfNextYear                                          | 返回下一年的第一天                                          |
| firstDayOfYear                                              | 返回本年的第一天                                            |
| firstInMonth                                                | 返回同一个月中第一个星期几                                  |
| lastDayOfMonth                                              | 返回当月的最后一天                                          |
| lastDayOfNextMonth                                          | 返回下月的最后一天                                          |
| lastDayOfNextYear                                           | 返回下一年的最后一天                                        |
| lastDayOfYear                                               | 返回本年的最后一天                                          |
| lastInMonth                                                 | 返回同一个月中最后一个星期几                                |
| next(DayOfWeek week) / previous(DayOfWeek week)             | 返回后一个/前一个给定的星期几                               |
| nextOrSame(DayOfWeek week) / previousOrSame(DayOfWeek week) | 返回后一个/前一个给定的星期几，如果这个值满足条件，直接返回 |
| DayOfWeek 这个枚举类有七个常量值                            | 分别为周一到周日，如 MONDAY, TUESDAY, WEDNESDAY 等          |

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
        // 下周日:2022-02-20T22:01:45.718728600
        System.out.println("下周日:" + with1);

        LocalDateTime with2 = now.with(TemporalAdjusters.firstDayOfMonth());
        // 这个月的第一天:2022-02-01T22:01:45.718728600
        System.out.println("这个月的第一天:" + with2);

        LocalDateTime with3 = now.with(TemporalAdjusters.firstDayOfNextMonth());
        // 下个月的第一天:2022-03-01T22:01:45.718728600
        System.out.println("下个月的第一天:" + with3);
    }
}
```



## java.time.format.DateTimeFormatter

Java8 后开始使用 java.time.format.DateTimeFormatter 类用于解析日期字符串和格式化日期输出。格式化字符串的使用方式与 Java8 之前的日期时间处理类 java.text.DateFormat 完全一致。

java.time.format.DateTimeFormatter 类：该类提供了三种格式化方法：

- 预定义的标准格式（系统提供好了枚举类）
- 语言环境相关的格式（设置指定的语言环境Locale，如：Locale.CHINA、Locale.US）
- 自定义的格式（自定义设置格式，例如：DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm")）

DateTimeFormatter 类包含一系列预定义常量的实例，可以解析和格式化一些标准时间格式。这将让你免除麻烦的时间格式定义，类中预定义实例如下：

```java
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
```

**DateTimeFormatter 常用API** ：

- DateTimeFormatter ofPattern(“yyyy-MM-dd”)：静态方法，通过给定格式获取对象
- String format(TemporalAccessor temporal)：把一个日期对象的**默认格式**格式化成**指定的格式**的**字符串**
- TemporalAccessor parse(CharSequence text)：把一个字符串格式化成指定格式的 TemporalAccessor 对象（一般不建议用）

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

**注意**：虽然 DateTimeFormatter 类可以进行日期时间格式化输出 & 字符串解析，但是我们更常用的还是 java.time 下每个日期类对应的静态 parse() 和 format() 方法来格式化输出 & 字符串解析。



## 日期时间格式化输出 & 字符串解析

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



## 日期时间格式化模板汇总

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




## Java8 的日期时间API总结

前面详细介绍了Java 8的日期时间API，现在进行简单总结一下。新的时间与日期 API 中很重要的一点是，它定义清楚了基本的时间与日期的概念，比方说日期、时间、瞬时时间、持续时间、时区及时间段。它们都是基于 ISO8601 日历系统，它是世界民用历法，也就是我们所说的公历。

java.time包下主要包含下面几个主要的类：

- LocalDate：表示不带时间的日期，比如：2016-10-20
- LocalTime：表示不带日期的时间，比如：23:12:10
- LocalDateTime：日期时间，比如：2016-10-20 23:14:21
- TemporalAdjuster : 时间调节器
- TemporalAdjusters：获得指定日期时间等，如当月的第一天、今年的最后一天等
- Duration：持续时间，计算两个“时间”的间隔
- Period：日期间隔，计算两个“日期”的间隔
- Instant：Unix 时间，它代表的是时间戳，比如 2018-01-14T02:20:13.592Z
- Clock：时钟，获取某个时区下的瞬时时间
- ZoneId：时区id，例如 Asia/Shanghai等
- ZonedDateTime：带时区的日期时间
- DateTimeFormatter：时间格式化

新的 API 区分各种日期时间概念并且各个概念使用相似的方法定义模式，这种相似性非常有利于 API 的学习。总结一下一般的方法规律：

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



## 新老日期时间类型转换表及对比

| 类                                         | To遗留类                         | From遗留类                  |
| ------------------------------------------ | -------------------------------- | --------------------------- |
| java.time.Instant java.util.Date           | Date.from(instant)               | date.toInstant()            |
| java.time.Instant java.sql.Timestamp       | Timestamp.from(instant)          | timestamp.toInstant()       |
| java.time.LocalDate java.sql.Date          | Date.valueOf(localDate)          | Date.toLocalDate()          |
| java.time.LocalTime java.sql.Time          | Time.valueOf(localTime)          | Time.toLocalTime()          |
| java.time.LocalDateTime java.sql.Timestamp | Timestamp.valueOf(localDateTime) | Timestamp.toLocalDateTime() |

| Java.time ISO Calendar                                    | Java.util Caalendar                                        |
| --------------------------------------------------------- | ---------------------------------------------------------- |
| Instant                                                   | Date                                                       |
| LocalDate、LocalTime、LocalDateTime                       | Calendar                                                   |
| ZonedDateTime                                             | Calendar                                                   |
| OffsetDateTime、OffsetTime                                | Calendar                                                   |
| ZoneId、ZoneOffset、ZoneRules                             | TimeZone                                                   |
| Week Starts on Monday(1..7) enum MONDAY, TUESDAY,..SUNDAY | Week Starts on Sunday(1..7) enum SUNDAY, MONDAY,..SATURDAY |
| 12 Months(1..12)  enum JANUARY, FUBRUARY,..DECEMBER       | 12 Months(0..11) int values JANUARY, FUBRUARY,..DECEMBER   |

`LocalDateTime`，`ZoneId`，`Instant`，`ZonedDateTime`和`long`都可以互相转换：

```
┌─────────────┐
│LocalDateTime│────┐
└─────────────┘    │    ┌─────────────┐
                   ├───>│ZonedDateTime│
┌─────────────┐    │    └─────────────┘
│   ZoneId    │────┘           ▲
└─────────────┘      ┌─────────┴─────────┐
                     │                   │
                     ▼                   ▼
              ┌─────────────┐     ┌─────────────┐
              │   Instant   │<───>│    long     │
              └─────────────┘     └─────────────┘
```



# Java 的时间工具类

## LocalDateTime和时间戳互转

```java
   /**
    * 获取到毫秒级时间戳
    * @param localDateTime 具体时间
    * @return long 毫秒级时间戳
    */
    public static long toEpochMilli(LocalDateTime localDateTime){
        return localDateTime.toInstant(ZoneOffset.of("+8")).toEpochMilli();
    }

    /**
     * 毫秒级时间戳转 LocalDateTime
     * @param epochMilli 毫秒级时间戳
     * @return LocalDateTime
     */
    public static LocalDateTime ofEpochMilli(long epochMilli){
        return LocalDateTime.ofInstant(Instant.ofEpochMilli(epochMilli), ZoneOffset.of("+8"));
    }

    /**
     * 获取到秒级时间戳
     * @param localDateTime 具体时间
     * @return long 秒级时间戳
     */
    public static long toEpochSecond(LocalDateTime localDateTime){
        return localDateTime.toEpochSecond(ZoneOffset.of("+8"));
    }

    /**
     * 秒级时间戳转 LocalDateTime
     * @param epochSecond 秒级时间戳
     * @return LocalDateTime
     */
    public static LocalDateTime ofEpochSecond(long epochSecond){
        return LocalDateTime.ofEpochSecond(epochSecond, 0,ZoneOffset.of("+8"));
    }
```



## LocalDateTime和Date互转

```java
    /**
     * Date时间类转LocalDateTime
     * @param date Date时间类
     * @return LocalDateTime
     */
    public static LocalDateTime ofDate(Date date){
        return date.toInstant().atOffset(ZoneOffset.of("+8")).toLocalDateTime();
    }

    /**
     * LocalDateTime时间类转 Date时间类
     * @param localDateTime LocalDateTime时间类
     * @return Date时间类
     */
    public static Date toDate(LocalDateTime localDateTime){
        return Date.from(localDateTime.atZone(ZoneOffset.of("+8")).toInstant());
    }
```



## LocalDateTime和字符串互转

```java
    /**
     * LocalDateTime转时间格式字符串
     * @param localDateTime 时间
     * @return string 
     */
    public static String formatToString(LocalDateTime localDateTime){
        String format  = "yyyy:MM:dd HH:mm:ss";
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(format);
        return localDateTime.format(dateTimeFormatter);
    }

    /**
     *  时间字符串 转LocalDateTime
     * @param localDateTime  时间字符串
     * @return LocalDateTime
     */
    public static LocalDateTime stringToFormat(String localDateTime){
        String format  = "yyyy:MM:dd HH:mm:ss";
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(format);
        return LocalDateTime.parse(localDateTime,dateTimeFormatter);
    }
```





## Java 7 时间工具类

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



## Java 8 时间工具类

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

> Java7 时间类：
>
> 1. Java的日期与时间（目录）：https://blog.csdn.net/tjgykhulj/article/details/68952451
> 2. 时间类_开发者-CSDN博客：https://blog.csdn.net/N_ZSX/article/details/121331104
>
> Java8 时间类：
>
> 1. Java方向盘-日期时间（CSDN）：https://blog.csdn.net/f641385712/category_10749009.html
> 1. Java方向盘-日期时间（公众号）：https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzI0MTUwOTgyOQ==&scene=24&album_id=1696358010555547649&count=3#wechat_redirect
> 2. ThinkWon：https://thinkwon.blog.csdn.net/article/details/111087199
> 3. java8时间使用（概念）：https://blog.csdn.net/xj80231314/article/details/86711095
> 5. Java日期时间教程 - 一点教程 (yiidian.com)：http://www.yiidian.com/java-date