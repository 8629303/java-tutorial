# 什么是Stream流

1. Stream 中⽂称为 “流”，通过将集合转换为这么⼀种叫做 “流”的元素队列，通过声明性⽅式，能够对集合中的每个元素进⾏⼀系列并⾏或串⾏的流⽔线操作
2. 元素是特定类型的对象，所以元素集合看作⼀种流， 流在管道中传输， 且可以在管道的节点上进⾏处理， ⽐如 排序，聚合，过滤等操作

```
1.数据源     ==》  2.filter =》 map =》 ... =》     3.终止操作
1.获取流     ==》  2.Stream 的中间操作 =》		  3.流的终止操作
```

Stream操作三步骤：

1. 创建 Stream：一个数据源（如：集合、数组），获取一个流（串⾏流stream() 或者并⾏流parallelStream()）
2. 中间操作：一个中间操作链，对数据源的数据进行处理（可以是 排序，聚合，过滤，转换等）
3. 终止操作(终端操作)：一个终止操作，执行中间操作链，并产生结果

Stream注意事项：

1. Stream 自己不会存储元素
2. Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream
3. Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行



# 如何创建Stream

## 1、Collection 获取

Java8 中的 Collection 接口被扩展，提供了两个获取流的方法：

1. 返回一个顺序流：default Stream stream()
2. 返回一个并行流：default Stream parallelStream()

源码分析：

```java
/**
 * 使用StreamSupport.stream()就可以创建一个Stream。
 * 第一个参数是传入一个迭代器，
 * 第二个参数是true代表使用并行来进行处理。false代表串行来处理Stream。
 */
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}
```

操作示例：

```java
import java.util.List;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9);
        numbers.stream().forEach(System.out::print); // 顺序输出
        System.out.println();
        numbers.parallelStream().forEach(System.out::print); //并⾏乱序输出
    }
}

// 输出内容
123456789
657238419
```



## 2、Arrays 数组获取

Java8 中的 Arrays 的静态方法 stream() 可以获取数组流：

1. Arrays静态方法返回一个流：public static Stream stream(T[] array)

   重载形式，能够处理对应基本类型的数组：

2. public static IntStream stream(int[] array)

3. public static LongStream stream(long[] array)

4. public static DoubleStream stream(double[] array)

```java
import java.util.Arrays;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Integer[] array = new Integer[]{3,4,8,16,19,27,23,99,76,232,33,96};
        long count = Arrays.stream(array).filter(i->i>20).count();
        System.out.println(count); // 7
    }
}
```



## 3、Stream.of() 获取

Stream类静态方法返回一个流：public static Stream of(T… values)：

```java
import java.util.stream.Stream;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Stream<Integer> stream = Stream.of(2, 4, 6, 8, 10, 12, 18, 20);
        stream.forEach(System.out::print); // 246810121820
    }
}
```



## 4、Stream 函数获取

可以使用静态方法 Stream.iterate() 和Stream.generate(), 创建无限流 。

1. 迭代：public static Stream iterate(final T seed, final UnaryOperatorf)

2. 生成：public static Stream generate(Suppliers)

范例：

```java
import java.math.BigInteger;
import java.util.Arrays;
import java.util.stream.Stream;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 迭代
        Stream<BigInteger> bigIntStream = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.TEN)).limit(10);
        BigInteger[] bigIntArr = bigIntStream.toArray(BigInteger[]::new);
        System.out.println(Arrays.toString(bigIntArr)); // [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
        // 重复生产
        Stream<String> stream = Stream.generate(() -> "test").limit(8);
        String[] strArr = stream.toArray(String[]::new);
        System.out.println(Arrays.toString(strArr)); // [test, test, test, test, test, test, test]
        
        // 生产无限个随机数
         Stream<Double> generate = Stream.generate(Math::random);
    }
}
```



## 5、Stream 其他获取

1. 产生一个空元素的流：Stream.empty()

2. 两个流合并成一个流：IntStream.concat(Stream a, Stream b) 

3. build方法构造流：IntStream.builder()

   会产生一个builder用于构建stream，通过builder的add方法添加元素



# Stream的中间操作

## 1、数据筛选

| 方法名                  | 描述                                                   |
| ----------------------- | ------------------------------------------------------ |
| filter(Predicate p)     | 接收Lambda，从流中排除某些元素                         |
| limit(long maxSize)     | 返回一个包含n个元素的新的流（若总长小于n则返回原始流） |
| skip(long n)            | 方法正好相反，它会丢弃掉前面的n个元素                  |
| distinct()              | 筛选，通过流所生成的hashCode和equase去除重复的         |
| sorted()                | 按自然顺序排序，其中的元素必须实现Comparable 接口      |
| sorted(Comparator comp) | 按比较器Comparator顺序排序，可升序和降序               |

操作示例：filter方法

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> integerList = List.of(1, 2, 3, 4, 5, 6);
        List<Integer> filter = integerList.stream().filter(i -> i > 3).collect(Collectors.toList());
        System.out.println(filter); // [4, 5, 6]
    }
}
```

操作示例：limit和skip方法

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> integerList = List.of(1, 2, 3, 4, 5, 6);
        List<Integer> limit = integerList.stream().limit(3).collect(Collectors.toList());
        List<Integer> skip = integerList.stream().skip(3).collect(Collectors.toList());
        System.out.println(limit); // [1, 2, 3]
        System.out.println(skip); // [4, 5, 6]
    }
}
```

操作示例：distinct方法去重

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> integerList = List.of(1, 2, 3, 4, 5, 6, 6);
        List<Integer> distinct = integerList.stream().distinct().collect(Collectors.toList());
        System.out.println(distinct); // [1, 2, 3, 4, 5, 6]
    }
}
```

操作示例：sorted排序（默认自然排序）

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<String> stringList = List.of("SpringBoot", "Redis", "Docker", "Mybatis", "JPA");
        // 按自然顺序排序
        List<String> collect1 = stringList.stream().sorted().collect(Collectors.toList());
        System.out.println(collect1); // [Docker, JPA, Mybatis, Redis, SpringBoot]
        // 按自然顺序逆序排序
        List<String> collect2 = stringList.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());
        System.out.println(collect2); // [SpringBoot, Redis, Mybatis, JPA, Docker]
    }
}
```

操作示例：sorted排序（Comparator自定义排序）

```java
import java.util.Comparator;
import java.util.List;
class Student {
    private Integer age;
    public Student(Integer age){
        this.age = age;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "年龄：" + this.age + "、";
    }
}
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Student> list = List.of(new Student(18), new Student(16), new Student(20), new Student(30));
        // 按照年龄升序排序
        list.stream().sorted(Comparator.comparing(Student::getAge)).forEach(System.out::print);
        System.out.println();
        // 按照年龄降序排序
        list.stream().sorted(Comparator.comparing(Student::getAge).reversed()).forEach(System.out::print);
        list.stream().sorted(Comparator.comparing((Student s) -> s.getAge()).reversed()).forEach(System.out::print);
    }
}

// 输出内容
年龄：16、年龄：18、年龄：20、年龄：30、
年龄：30、年龄：20、年龄：18、年龄：16、
年龄：30、年龄：20、年龄：18、年龄：16、
```





## 2、数据映射

| 方法名                          | 描述                                             |
| ------------------------------- | ------------------------------------------------ |
| map(Function f)                 | map方法指对一个流中的值进行某种形式的转换        |
| mapToDouble(ToDoubleFunction f) | 转成double类型流                                 |
| mapTOInt(ToIntFunction f)       | 转成int类型流                                    |
| mapToLong(ToLongFuntion f)      | 转成long类型流                                   |
| flatMap(Function f)             | 将流中每个值都换成另一个流，然后把所有流成一个流 |
| flatMapToDouble(Function f)     | 同上                                             |
| flatMapTOInt(Function f)        | 同上                                             |
| flatMapToLong(Function f)       | 同上                                             |

操作示例：map()

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> list = List.of(10, 20, 30, 40, 50);
        // 将Integer类型转换成String类型
        List<String> afterString = list.stream().map(i->String.valueOf(i)).collect(Collectors.toList());
        System.out.println(afterString); // [10, 20, 30, 40, 50]
    }
}

 // 输出内容
[10, 20, 30, 40, 50]
```

Java8引入了三个原始类型流解决装箱拆箱问题：IntStream、DoubleStream、LongStream。每个接口中都有对数sum，max，min，average等方法

操作示例：mapToXxx()

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        // Integer占用的内存比int多很多，在Stream流操作中会自动装修和拆箱操作
        Integer[] arr = {1, 2, 3, 5, 6, 8};
        List<Integer> integerList = Stream.of(arr).filter(i -> i > 0).collect(Collectors.toList());
        System.out.println(integerList); // [1, 2, 3, 5, 6, 8]

        // 为了提高程序代码的效率，我们可以先将流中Integer数据转换为int数据，然后再操作
        IntStream intStream = Stream.of(arr).mapToInt(Integer::intValue);
        intStream.filter(i -> i > 3).forEach(System.out::println); // 5 6 8

        // 更重要的是可以用来统计方法的操作
        int sum = Stream.of(arr).mapToInt(Integer::intValue).sum();
        OptionalInt max = Stream.of(arr).mapToInt(d -> d.intValue()).max();
        OptionalInt min = Stream.of(arr).mapToInt(d -> d.intValue()).min();
        OptionalDouble average = Stream.of(arr).mapToInt(d -> d.intValue()).average();
        System.out.println("和：" + sum + "、最大值：" + max.getAsInt() + "、最小值：" + min.getAsInt() + "、平均数：" + average.getAsDouble());
    }
}

// 输出内容
[1, 2, 3, 5, 6, 8]
5
6
8
和：25、最大值：8、最小值：1、平均数：4.166666666666667
```

操作示例：flatMap()

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Integer> oneList = Arrays.asList(10, 20, 30);
        List<Integer> twoList = Arrays.asList(40, 50, 60);
        Map<String, List<Integer>> testMap = new HashMap<>();
        testMap.put("1", oneList);
        testMap.put("2", twoList);

        // 返回的是一个流的集合，但是我需要的是List<Integer>这样一个集合
        List<Stream<Integer>> testList1 = testMap.values().stream().map(number -> number.stream()).collect(Collectors.toList());
        System.out.println(testList1); // 返回时一个List对象，包含2个Stream

        // 这个时候就应该使用flatMap将多个流进行合并，然后再收集到一个集合中
        List<Integer> testList2 = testMap.values().stream().flatMap(number -> number.stream()).collect(Collectors.toList());
        System.out.println(testList2); // [10, 20, 30, 40, 50, 60]
    }
}

// 输出内容
[java.util.stream.ReferencePipeline$Head@6e8dacdf, java.util.stream.ReferencePipeline$Head@7a79be86]
[10, 20, 30, 40, 50, 60]
```



# Stream 的终止操作

## 1、查找与匹配

| 方法                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| forEach(Comparator c)  | 内部迭代（使用Collection接口需要用户去做迭代称为：外部迭代） |
| allMatch(Predicate p)  | 检查是否匹配所有元素                                         |
| anyMatch(Predicate p)  | 检查是否至少匹配一个元素                                     |
| noneMatch(Predicate p) | 检查是否没有匹配所有元素                                     |
| findFirst()            | 返回第一个元素                                               |
| findAny()              | 返回当前流中任意元素                                         |
| max(Comparator c)      | 返回流中最大值                                               |
| min(Comparator c)      | 返回流中最小值                                               |
| count()                | 返回流中元素总数                                             |

操作示例：allMatch()、noneMatch()、anyMatch()

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> list = Stream.iterate(1, n -> n + 1).limit(10).collect(Collectors.toList());
        System.out.println(list);
        boolean isHas = list.parallelStream().anyMatch(i -> i < 10);
        boolean allHas = list.parallelStream().allMatch(i -> i < 12);
        boolean noHas = list.parallelStream().noneMatch(i -> i < 12);
        System.out.println("集合中是否有一个小于10的元素：" + isHas);
        System.out.println("集合中是否全部小于12：" + allHas);
        System.out.println("集合中是否全部不小于12：" + noHas);
    }
}

// 输出内容
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
集合中是否有一个小于10的元素：true
集合中是否全部小于12：true
集合中是否全部不小于12：false
```



## 2、规约

| 方法                             | 描述                                                 |
| -------------------------------- | ---------------------------------------------------- |
| reduce(T iden, BinaryOperator b) | 可以将流中元素反复结合起来，得到一个值。返回T        |
| reduce(BinaryOperator b)         | 可以将流中元素反复结合起来，得到一个值。返回Optional |



## 3、收集

| 方法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| collect(Collector c) | 将流转为其他形式。接收一个Collector接口的实现，用于给Stream中元素汇总的方法 |

Collector 接口中方法的实现决定了如何对流执行收集操作(如收集到 List、 Set、 Map)。但是 Collectors 实用类提供了很多静态方法，可以方便地创建常见收集器实例， 具体方法与实例如下表：

| 方法               | 返回类型                  | 作用                                                         |
| ------------------ | ------------------------- | ------------------------------------------------------------ |
| toList             | List\<T\>                 | 把流中元素收集到List                                         |
| toSet              | Set\<T\>                  | 把流中元素收集到Set                                          |
| toCollection       | Collection\<T\>           | 把流中元素收集到创建的集合                                   |
| counting           | Long                      | 计算流中元素的个数                                           |
| summingInt         | Integer                   | 对流中元素的整数属性求和                                     |
| averagingInt       | Double                    | 计算流中元素Integer属性的平均值                              |
| summarizingInt     | IntSummaryStatistics      | 收集流中Integer属性的统计值.如平均值                         |
| joining            | String                    | 连接流中每个字符串                                           |
| maxBy              | Optional\<T\>             | 根据比较器选择最大值                                         |
| minBy              | Optional\<T\>             | 根据比较器选择最小值                                         |
| reducing           | 规约产生的类型            | 从一个作为累加器的初始值开始，利用BinaryOperator与流中元素逐个结合，从而归约成单个值 |
| collectiongAndThen | 转换函数返回的类型        | 包裹另一个收集器，对其结果转换函数                           |
| groupingBy         | Map\<K, List\<T\>\>       | 根据某属性值对流分组，属性为K,结果为V                        |
| partitioningBy     | Map\<Boolean, List\<T\>\> | 根据true或者false进行分区                                    |

1、收集到集合

```java
import java.util.*;
import java.util.stream.Collectors;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> hereList = Arrays.asList(10, 20, 30, 30);
        // 把流中元素收集到List
        List<Integer> thereList = hereList.stream().collect(Collectors.toList());
        System.out.println(thereList); // [10, 20, 30, 30]
        // 把流中元素收集到Set
        Set<Integer> thereSet = hereList.stream().collect(Collectors.toSet());
        System.out.println(thereSet); // [20, 10, 30]
        // 把流中元素收集到创建的集合
        TreeSet<Integer> treeSet = hereList.stream().collect(Collectors.toCollection(TreeSet::new));
        System.out.println(treeSet); // [10, 20, 30]
    }
}

// 输出内容
[10, 20, 30, 30]
[20, 10, 30]
[10, 20, 30]
```

2、将结果集收集到Map

```java
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Student{
    private String name;
    private Integer age;
    private String className;
    public Student(String name, Integer age, String className) {
        this.name = name;
        this.age = age;
        this.className = className;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
    public String getClassName() {
        return className;
    }
    public void setClassName(String className) {
        this.className = className;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "二班"),
                new Student("王五", 15, "三班"));
        Map<String, String> map = students.stream().collect(Collectors.toMap(Student::getName, Student::getClassName));
        System.out.println(map);
    }
}

// 输出内容
{李四=二班, 张三=一班, 王五=三班}
```

3、拼接

```java
import java.util.*;
import java.util.stream.Collectors;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<String> stringList = Arrays.asList("1", "2", "3", "4", "5", "6", "7");

        // 将字流中的字符串连接并收集起来。
        String resultString1 = stringList.stream().collect(Collectors.joining());
        System.out.println(resultString1); // 1234567

        // 在将流中的字符串连接并收集起来时，想在元素中介添加分隔符，传递个joining方法即可
        String resultString2 = stringList.stream().collect(Collectors.joining(","));
        System.out.println(resultString2); // 1,2,3,4,5,6,7

        // 复杂玩法
        List<Integer> intList = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
        String ResultString3 = intList.stream().map(String::valueOf).collect(Collectors.joining(","));
        System.out.println(ResultString3); // 1,2,3,4,5,6,7
    }
}

// 输出内容
1234567
1,2,3,4,5,6,7
1,2,3,4,5,6,7
```

4、收集聚合

```java
import java.util.*;
import java.util.stream.Collectors;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> integerList = Arrays.asList(1, 11, 3, 8, 7, 9, 10);
        // 总和
        int sum = integerList.stream().collect(Collectors.summingInt(Integer::intValue));
        // 平均值
        Double ave = integerList.stream().collect(Collectors.averagingInt(Integer::intValue));
        // 最大值
        Integer max = integerList.stream().collect(Collectors.maxBy(Integer::compare)).get();
        // 最小值
        Integer min = integerList.stream().collect(Collectors.minBy(Integer::compare)).get();
        // 输出：sum:49,ave:7.0,max:11,min:1
        System.out.println("sum:" + sum + ",ave:" + ave + ",max:" + max + ",min:" + min);
    }
}

// 输出内容
sum:49,ave:7.0,max:11,min:1
```

5、一次性收集流中的结果，聚合为一个总和，平均值，最大值或最小值的对象。

```java
import java.util.*;
import java.util.stream.Collectors;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> integerList = Arrays.asList(1, 11, 3, 8, 7, 9, 10);
        IntSummaryStatistics summaryStatistics = integerList.stream().collect(Collectors.summarizingInt(Integer::intValue));
        // 输出：IntSummaryStatistics{count=7, sum=49, min=1, average=7.000000, max=11}
        System.out.println(summaryStatistics);
        System.out.println(summaryStatistics.getSum());
        System.out.println(summaryStatistics.getCount());
        System.out.println(summaryStatistics.getMax());
        System.out.println(summaryStatistics.getMax());
        System.out.println(summaryStatistics.getAverage());
    }
}

// 输出内容
IntSummaryStatistics{count=7, sum=49, min=1, average=7.000000, max=11}
49
7
11
11
7.0
```



## 4、分组分片

在一个集合中，对具有相同特性的值进行分组是一个很常见的功能，在Stream的API中也提供了相应的方法。

范例：分组，groupingBy

```java
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;
class Student{
    private String name;
    private Integer age;
    private String className;
    public Student(String name, Integer age, String className) {
        this.name = name;
        this.age = age;
        this.className = className;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
    public String getClassName() {
        return className;
    }
    public void setClassName(String className) {
        this.className = className;
    }
    @Override
    public String toString() {
        return "Student{name='" + this.name +"', age=" + this.age + ", className='" + this.className + "'}";
    }
}
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 12, "二班"),
                new Student("小李", 12, "三班"),
                new Student("王五", 15, "三班"));
        Map<String, List<Student>> collect = students.stream().collect(Collectors.groupingBy(Student::getClassName));
        System.out.println(collect);
    }
}

// 输出内容
{一班=[
    Student{name='张三', age=10, className='一班'}, 
    Student{name='李四', age=12, className='一班'}], 
 二班=[
     Student{name='小吴', age=12, className='二班'}, 
     Student{name='小刘', age=12, className='二班'}], 
 三班=[
     Student{name='小李', age=12, className='三班'}, 
     Student{name='王五', age=15, className='三班'}]}
```



当分类函数是一个返回布尔值的函数时，流元素会被分为两组列表：一组是返回true的元素集合，另一组是返回false的元素集合。这种情况适用partitoningBy方法会比groupingBy更有效率。

操作示例：分片或者分区，partitoningBy

```java
package cn.mldn.demo;

import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));
        Map<Boolean, List<Student>> collect = students.stream().
                collect(Collectors.partitioningBy(stu -> stu.getAge() > 12));
        System.out.println(collect);
    }
}
@Data
class Student{
    private String name;
    private Integer age;
    private String className;
    public Student(String name, Integer age, String className) {
        this.name = name;
        this.age = age;
        this.className = className;
    }
}
```

**output：**

```java
{false=[Student{name='张三', age=10, className='一班'}, 
        Student{name='李四', age=12, className='一班'}, 
        Student{name='小吴', age=12, className='二班'}], 
 true=[Student{name='小刘', age=14, className='二班'}, 
       Student{name='小李', age=13, className='三班'}, 
       Student{name='王五', age=15, className='三班'}]}
```



5、扩展功能

下面要介绍的这些方法功能，无论是groupingBy方法还是partitioningBy方法都是支持的。

**counting方法**会返回收集元素的总个数。

```java
package cn.mldn.demo;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));
        Map<Integer, Long> countMap  = students.stream().
                collect(Collectors.groupingBy(Student::getAge, Collectors.counting()));
        System.out.println(countMap);
    }
}
.....
```

**output：**

```java
{10=1, 12=2, 13=1, 14=1, 15=1}
```



**summing(Int|Long|Double)方法**接受一个取值函数作为参数，来计算总和。

```java
package cn.mldn.demo;
import java.util.*;
import java.util.stream.Collectors;

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));
        Map<String, Integer> sumMap = students.stream().
                collect(Collectors.groupingBy(Student::getClassName,
                        Collectors.summingInt(Student::getAge)));
        System.out.println(sumMap);
    }
}
....
```

output：

```java
{一班=22, 二班=26, 三班=28}
```



**maxBy方法和minBy方法**接受比较器作为参数来计算最大值和最小值。取出分组中年龄中最大和最小的学生

```java
package cn.mldn.demo;

import java.util.*;
import java.util.stream.Collectors;

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));
        // maxBy
        Map<String, Optional<Student>> maxMap  = students.stream().
                collect(Collectors.groupingBy(Student::getClassName,
                        Collectors.maxBy(Comparator.comparing(Student::getAge))));
        // minBy
        Map<String, Optional<Student>> minMap  = students.stream().
                collect(Collectors.groupingBy(Student::getClassName,
                        Collectors.minBy(Comparator.comparing(Student::getAge))));
        System.out.println(maxMap);
        System.out.println(minMap);
    }
}
.....
```

**output：**

```java
{一班=Optional[Student{name='李四', age=12, className='一班'}], 
 二班=Optional[Student{name='小刘', age=14, className='二班'}], 
 三班=Optional[Student{name='王五', age=15, className='三班'}]}

{一班=Optional[Student{name='张三', age=10, className='一班'}], 
 二班=Optional[Student{name='小吴', age=12, className='二班'}], 
 三班=Optional[Student{name='小李', age=13, className='三班'}]}
```





> **mapping方法**会将结果应用到另一个收集器上。

取出分组中年龄最大的学生的年龄

```java
package cn.mldn.demo;
import java.util.*;
import java.util.stream.Collectors;

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));
        Map<String, Optional<Integer>> collect = students.stream().collect(Collectors.groupingBy(Student::getClassName,
                Collectors.mapping(Student::getAge, Collectors.maxBy(Comparator.comparing(Integer::valueOf)))));
        System.out.println(collect);
    }
}
.....
```

output：

```java
{一班=Optional[12], 二班=Optional[14], 三班=Optional[15]}
```





# 参考文献 & 鸣谢

1. 波波烤鸭，CSDN：https://dpb-bobokaoya-sm.blog.csdn.net/article/details/117591234
2. 纪莫，博客园：https://www.cnblogs.com/jimoer/p/10995574.html
2. 【Java8新特性】面试官：谈谈Java8中的Stream API有哪些终止操作？https://blog.csdn.net/l1028386804/article/details/106416723
2. Java 8 Stream API学习总结：https://blog.csdn.net/qq_28336351/article/details/106153368
