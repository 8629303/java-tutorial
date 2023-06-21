# 日期时间转换及封装工具类

## 1、其他时间类型转 Date

```java
final Timestamp timestamp = new Timestamp(date.getTime());
final Calendar calendar = Calendar.getInstance();
final Instant instant = Instant.now();
final LocalDateTime localDateTime = LocalDateTime.now();
final ZonedDateTime zonedDateTime = ZonedDateTime.now();
final LocalDate localDate = LocalDate.now();

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
final Date date = new Date();
final Calendar calendar = Calendar.getInstance();
final Instant instant = Instant.now();
final LocalDateTime localDateTime = LocalDateTime.now();
final ZonedDateTime zonedDateTime = ZonedDateTime.now();
final LocalDate localDate = LocalDate.now();

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
final Date date = new Date();
final Timestamp timestamp = new Timestamp(date.getTime());
final Instant instant = Instant.now();
final LocalDateTime localDateTime = LocalDateTime.now();
final ZonedDateTime zonedDateTime = ZonedDateTime.now();
final LocalDate localDate = LocalDate.now();

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
final Date date = new Date();
final Timestamp timestamp = new Timestamp(date.getTime());
final Calendar calendar = Calendar.getInstance();
final LocalDateTime localDateTime = LocalDateTime.now();
final ZonedDateTime zonedDateTime = ZonedDateTime.now();
final LocalDate localDate = LocalDate.now();

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
final Date date = new Date();
final Timestamp timestamp = new Timestamp(date.getTime());
final Calendar calendar = Calendar.getInstance();
final Instant instant = Instant.now();
final LocalDateTime localDateTime = LocalDateTime.now();
final ZonedDateTime zonedDateTime = ZonedDateTime.now();

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
final Date date = new Date();
final Timestamp timestamp = new Timestamp(date.getTime());
final Calendar calendar = Calendar.getInstance();
final Instant instant = Instant.now();
final ZonedDateTime zonedDateTime = ZonedDateTime.now();
final LocalDate localDate = LocalDate.now();

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
final Date date = new Date();
final Timestamp timestamp = new Timestamp(date.getTime());
final Calendar calendar = Calendar.getInstance();
final Instant instant = Instant.now();
final LocalDateTime localDateTime = LocalDateTime.now();
final LocalDate localDate = LocalDate.now();

// Date 转 ZonedDateTime
ZonedDateTime dateZonedDateTime = ZonedDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
ZonedDateTime dateZonedDateTime = date.toInstant().atZone(ZoneId.systemDefault());

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



# 日期时间类创建解析速度测试

> JAVA日期时间类创建解析速度测试：https://blog.csdn.net/kfepiza/article/details/115448573

```java
public class 日期时间类创建解析速度测试 {

    long totalAmount001 = 10_000_000L;

    void p(Object o) {
        System.out.println(o);
    }

    @Test
    public void testSystemCurrentTimeMillisSpeed() {
        long v = 0;
        for (long i = 0; i < totalAmount001; i++) {
            v = System.currentTimeMillis();
        }
        p(v);
    }

    @Test
    public void testInstantNowSpeed() {
        Instant instant = null;
        for (long i = 0; i < totalAmount001; i++) {
            instant = Instant.now();
        }
        p(instant);
    }

    @Test
    public void testLocalDateNowSpeed() {
        LocalDate localDate = null;
        for (long i = 0; i < totalAmount001; i++) {
            localDate = LocalDate.now();
        }
        System.out.println(localDate);
    }

    @Test
    public void testLocalTimeNowSpeed() {
        LocalTime localTime = null;
        for (long i = 0; i < totalAmount001; i++) {
            localTime = LocalTime.now();
        }
        System.out.println(localTime);
    }

    @Test
    public void testLocalDateTimeNowSpeed() {
        LocalDateTime localDateTime = null;
        for (long l = 0; l < totalAmount001; l++) {
            localDateTime = LocalDateTime.now();
        }
        System.out.println(localDateTime);
    }

    @Test
    public void testLocalDateTimeOfIntSpeed() {
        LocalDateTime localDateTime = null;
        for (long l = 0; l < totalAmount001; l++) {
            localDateTime = LocalDateTime.of(2021, 4, 4, 8, 0, 0);
        }
        System.out.println(localDateTime);
    }

    @Test
    public void testLocalDateTimeOfIntegerValueOfStringSpeed() {
        LocalDateTime localDateTime = null;
        for (long l = 0; l < totalAmount001; l++) {
            localDateTime = LocalDateTime.of(
                    Integer.valueOf("2021"), Integer.valueOf("4"), Integer.valueOf("5")
                    , Integer.valueOf("0"), Integer.valueOf("0"), Integer.valueOf("0")
            );
        }
        System.out.println(localDateTime);
    }

    @Test
    public void testLocalDateTimeParseStringSpeed() {
        LocalDateTime localDateTime = null;
        for (long l = 0; l < totalAmount001; l++) {
            localDateTime = LocalDateTime.parse("2021-04-04T08:08:08");
        }
        System.out.println(localDateTime);
    }

    @Test
    public void testNewUtilDateSpeed() {
        java.util.Date utilDate = null;
        for (long l = 0; l < totalAmount001; l++) {
            utilDate = new java.util.Date();
        }
        System.out.println(utilDate);
    }

    @Test
    public void testNewTimestampLongSpeed() {
        Timestamp timestamp = null;
        for (long l = 0; l < totalAmount001; l++) {
            timestamp = new Timestamp(0);
        }
        p(timestamp);
    }

    @Test
    public void testNewTimestampSystemCurrenttTimeMillisSpeed() {
        Timestamp timestamp = null;
        for (long l = 0; l < totalAmount001; l++) {
            timestamp = new Timestamp(System.currentTimeMillis());
        }
        p(timestamp);
    }

    @Test
    public void testTimestampValueOfStringSpeed() {
        Timestamp timestamp = null;
        for (long l = 0; l < totalAmount001; l++) {
            timestamp = Timestamp.valueOf("2021-04-05 01:18:18");
        }
        p(timestamp);
    }
}
```

- System.currentTimeMillis() 比 Instant.now() 快3倍
- new Date() 比 LocalDateTime.now() 快10倍
- LocalDateTime.of(int , int …) 速度还是挺快的
- LocalDateTime.now()比LocalDateTime.of(int) 慢了近50倍
- LocalDateTime.parse(字符串)比LocalDateTime.now() 还慢20倍
- LocalDateTime.parse(“2021-04-05T00:00:00”)中间的 T 不能少
- 用官方方式解析字符串获得时间真的很慢：
  - Timestamp.valueOf(String)比 new Timestamp(long) 慢60倍
  - LocalDateTime.parse(String)比LocalDateTime.of(int,int…)慢800倍
  - LocalDateTime.parse(String)比Timestamp.valueOf(String)慢6倍
  - LocalDateTime.of(2021, 4, 8, 8, 8); 比 LocalDateTime.parse(“2021-04-04T08:08:08”) 快 200 倍!


