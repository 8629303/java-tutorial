# 一、Stream 流概述

1. Stream 中⽂称为 “流”，通过将集合转换为这么⼀种叫做 “流”的元素队列，通过声明性⽅式，能够对集合中的每个元素进⾏⼀系列并⾏或串⾏的流⽔线操作
2. 元素是特定类型的对象，所以元素集合看作⼀种流， 流在管道中传输， 且可以在管道的节点上进⾏处理， ⽐如 排序，聚合，过滤等操作

```
+--------------------+     +------+    +------+    +---+    +-------+
| stream of elements + --> |filter+ -> |sorted+ -> |map+ -> |collect|
+--------------------+     +------+    +------+    +---+    +-------+
1.数据源     »»»  2.filter »»» map »»» ... »»»     3.终止操作
1.获取流     »»»  2.Stream 流的中间操作     »»»     3.流的终止操作
```

Stream操作三步骤：

1. 创建 Stream：一个数据源（如：集合、数组），获取一个流（串⾏流stream() 或者并⾏流parallelStream()）
2. 中间操作：一个中间操作链，对数据源的数据进行处理（可以是 排序，聚合，过滤，转换等）
3. 终止操作(终端操作)：一个终止操作，执行中间操作链，并产生结果

Stream 注意事项（特性）：

1. Stream 不存储数据，而是按照特定的规则对数据进行计算，一般会输出结果
2. Stream 不会改变数据源，通常情况下会产生一个新的集合或一个值
3. Stream 具有延迟执行特性，只有调用终端操作时，中间操作才会执行

# 二、Stream 的创建

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

Java8 中的 Arrays 的静态方法 stream() 可以获取数组流（Arrays静态方法返回一个流，有重载形式，能够处理对应基本类型的数组）

1. public static Stream stream(T[] array)

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

可以使用静态方法 Stream.iterate() 和Stream.generate(), 创建无限流：

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

3. build方法构造流（会产生一个builder用于构建stream，通过builder的add方法添加元素）：IntStream.builder()

4. 除了直接创建并行流，还可以通过`parallel()`把顺序流转换成并行流

   ```java
   Optional<Integer> findFirst = list.stream().parallel().filter(x->x>6).findFirst();
   ```

5. Stream除了自己外，还有IntStream流，常用方法：IntStream.range(1, 11)，示例如下：

   ```java
   List<List<Integer>> x = IntStream.range(1, 5).mapToObj(x -> Arrays.asList(x)).collect(Collectors.toList());
   System.out.println(x); // [[1], [2], [3], [4]]
   ```



# 三、Stream 中间操作

## 1、筛选（filter）

| 方法名              | 描述                           |
| ------------------- | ------------------------------ |
| filter(Predicate p) | 接收Lambda，从流中排除某些元素 |

操作示例：filter过滤筛选方法

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> integerList = List.of(1, 2, 3, 4, 5, 6);
        List<Integer> filter = integerList.stream().filter(i -> i > 3).collect(Collectors.toList());
        System.out.println(filter); // [4, 5, 6]
    }
}
```

TODO：组合filter



## 2、映射（map/flatMap）

| 方法名                          | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| map(Function f)                 | map方法指对一个流中的值进行某种形式的转换（可以套娃）        |
| mapToDouble(ToDoubleFunction f) | 转成double类型流                                             |
| mapTOInt(ToIntFunction f)       | 转成int类型流                                                |
| mapToLong(ToLongFuntion f)      | 转成long类型流                                               |
| flatMap(Function f)             | 将流中每个值都换成另一个流，然后把所有流成一个流（扁平化，拒绝套娃） |
| flatMapToDouble(Function f)     | 同上                                                         |
| flatMapTOInt(Function f)        | 同上                                                         |
| flatMapToLong(Function f)       | 同上                                                         |

操作示例：map()映射方法

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> list = List.of(10, 20, 30, 40, 50);
        // 将Integer类型转换成String类型
        List<String> afterString = list.stream().map(i->String.valueOf(i)).collect(Collectors.toList());
        System.out.println(afterString); // [10, 20, 30, 40, 50]
    }
}
```

> Java8引入了三个原始类型流解决装箱拆箱问题：IntStream、DoubleStream、LongStream。每个接口中都有对数sum，max，min，average等方法
>

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

操作示例：flatMap()映射并转换流

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 准备两个List集合
        List<Integer> oneList = Arrays.asList(10, 20, 30);
        List<Integer> twoList = Arrays.asList(40, 50, 60);
        Map<String, List<Integer>> testMap = new HashMap<>();
        testMap.put("One", oneList);
        testMap.put("Two", twoList);

        // 返回的是一个流的集合，但是我需要的是List<Integer>这样一个集合
        List<Stream<Integer>> testList1 = testMap.values()
                .stream()
                .map(number -> number.stream())
                .collect(Collectors.toList());
        // 返回时一个List对象，包含2个List
        System.out.println(testList1); // [[10, 20, 30], [40, 50, 60]]

        // 这个时候就应该使用flatMap将多个流进行合并，然后再收集到一个集合中
        List<Integer> testList2 = testMap.values()
                .stream()
                .flatMap(number -> number.stream())
                .collect(Collectors.toList());
        System.out.println(testList2); // [10, 20, 30, 40, 50, 60]
    }
}
```



## 3、限制（limit/skip）

| 方法名              | 描述                                                   |
| ------------------- | ------------------------------------------------------ |
| limit(long maxSize) | 返回一个包含n个元素的新的流（若总长小于n则返回原始流） |
| skip(long n)        | 方法正好相反，它会丢弃掉前面的n个元素                  |

操作示例：limit和skip限制与跳过方法

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



## 4、去重（distinct）

| 方法名     | 描述                                           |
| ---------- | ---------------------------------------------- |
| distinct() | 筛选，通过流所生成的hashCode和equase去除重复的 |

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



## 5、排序（sorted）

| 方法名                  | 描述                                              |
| ----------------------- | ------------------------------------------------- |
| sorted()                | 按自然顺序排序，其中的元素必须实现Comparable 接口 |
| sorted(Comparator comp) | 按比较器Comparator顺序排序，可升序和降序          |

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
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student {
    private Integer age;
}
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Student> list = List.of(new Student(18), new Student(16), new Student(20), new Student(30));
        // 按照年龄升序排序
        list.stream().sorted(Comparator.comparing(Student::getAge)).forEach(System.out::print);
        System.out.println();
        // 按照年龄降序排序
        list.stream().sorted(Comparator.comparing(Student::getAge).reversed()).forEach(System.out::print);
        System.out.println();
        // 按照年龄降序排序 Lambda
        list.stream().sorted(Comparator.comparing((Student s) -> s.getAge()).reversed()).forEach(System.out::print);
    }
}

// 输出内容
Student(age=16)Student(age=18)Student(age=20)Student(age=30)
Student(age=30)Student(age=20)Student(age=18)Student(age=16)
Student(age=30)Student(age=20)Student(age=18)Student(age=16)
```

操作示例：多字段排序

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student {
    private Integer age;
    private Integer classNum;
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 初始化数据
        List<Student> list = List.of(
                new Student(18, 1),
                new Student(16, 1),
                new Student(20, 3),
                new Student(20, 5),
                new Student(30, 2));

        // 按照年龄升序,再按照班级升序
        list.stream().sorted(Comparator.comparing(Student::getAge)
                .thenComparing(Student::getClassNum))
                .forEach(System.out::print);
        System.out.println();

        // 按照年龄升序,再按照班级降序
        // 注意：下面的reversed()在thenComparing()里面
        list.stream().sorted(Comparator.comparing(Student::getAge)
                .thenComparing(Comparator.comparing(Student::getClassNum).reversed()))
                .forEach(System.out::print);
        System.out.println();

        // 按照年龄降序,再按照班级升序
        list.stream().sorted(Comparator.comparing(Student::getAge).reversed()
                .thenComparing(Student::getClassNum))
                .forEach(System.out::print);
        System.out.println();

        // 按照年龄降序,再按照班级降序
        // 注意：下面的reversed()在thenComparing()外面 (可以省略年龄后面的reversed()方法)
        list.stream().sorted(Comparator.comparing(Student::getAge)
                        .thenComparing(Student::getClassNum).reversed())
                .forEach(System.out::print);
        System.out.println();

        /**
         * Lambda形式演示案例
         */
        // 按照年龄升序,再按照班级升序
        list.stream().sorted(Comparator.comparing((Student s) -> s.getAge())
                        .thenComparing((Student s) -> s.getClassNum()))
                .forEach(System.out::print);
        System.out.println();
        // 按照年龄升序,再按照班级降序.
        list.stream().sorted(Comparator.comparing((Student s) -> s.getAge())
                .thenComparing(Comparator.comparing((Student s) -> s.getClassNum()).reversed()))
                .forEach(System.out::print);
        System.out.println();
        // 按照年龄降序,再按照班级升序
        list.stream().sorted(Comparator.comparing((Student s) -> s.getAge()).reversed()
                        .thenComparing((Student s) -> s.getClassNum()))
                .forEach(System.out::print);
        System.out.println();
        // 按照年龄降序,再按照班级降序
        list.stream().sorted(Comparator.comparing((Student s) -> s.getAge())
                        .thenComparing((Student s) -> s.getClassNum()).reversed())
                .forEach(System.out::print);
        System.out.println();
    }
}

// 输出内容
Student(age=16, classNum=1)Student(age=18, classNum=1)Student(age=20, classNum=3)Student(age=20, classNum=5)Student(age=30, classNum=2)
Student(age=16, classNum=1)Student(age=18, classNum=1)Student(age=20, classNum=5)Student(age=20, classNum=3)Student(age=30, classNum=2)
Student(age=30, classNum=2)Student(age=20, classNum=3)Student(age=20, classNum=5)Student(age=18, classNum=1)Student(age=16, classNum=1)
Student(age=30, classNum=2)Student(age=20, classNum=5)Student(age=20, classNum=3)Student(age=18, classNum=1)Student(age=16, classNum=1)
Student(age=16, classNum=1)Student(age=18, classNum=1)Student(age=20, classNum=3)Student(age=20, classNum=5)Student(age=30, classNum=2)
Student(age=16, classNum=1)Student(age=18, classNum=1)Student(age=20, classNum=5)Student(age=20, classNum=3)Student(age=30, classNum=2)
Student(age=30, classNum=2)Student(age=20, classNum=3)Student(age=20, classNum=5)Student(age=18, classNum=1)Student(age=16, classNum=1)
Student(age=30, classNum=2)Student(age=20, classNum=5)Student(age=20, classNum=3)Student(age=18, classNum=1)Student(age=16, classNum=1)
```

操作示例：Null值参与排序

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student {
    private Integer age;
    private Integer classNum;
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 初始化数据
        List<Student> list = List.of(
                new Student(18, 1),
                new Student(null, 3),
                new Student(15, null),
                new Student(20, 5),
                new Student(30, 2));

        // 按照年龄升序,null值排在前面
        list.stream().sorted(Comparator.comparing(Student::getAge,
                        Comparator.nullsFirst(Integer::compareTo)))
                .forEach(System.out::print);
        System.out.println();

        // 按照年龄升序,null值排在后面
        list.stream().sorted(Comparator.comparing(Student::getAge,
                        Comparator.nullsLast(Integer::compareTo)))
                .forEach(System.out::print);
        System.out.println();

        // 同时Null值排序和多字段排序
        list.stream().sorted(Comparator.comparing(Student::getAge)
                        .thenComparing(Student::getClassNum,
                                Comparator.nullsFirst(Integer::compareTo)))
                .forEach(System.out::print);
        System.out.println();
    }
}
```

操作示例：List<Map<String, Object>> 排序

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student {
    private Integer age;
    private Integer classNum;
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Map<String, Object>> cats = new ArrayList<>();
        Map<String, Object> cat1 = new HashMap<>();
        cat1.put("name", "cat1");
        cat1.put("age", 10);
        cats.add(cat1);

        Map<String, Object> cat2 = new HashMap<>();
        cat2.put("name", "cat2");
        cat2.put("age", 2);
        cats.add(cat2);

        // 按age升序
        List<Map<String, Object>> sortedCats = cats.stream().sorted((map1, map2) -> {
            int age1 = (int) map1.getOrDefault("age", 0);
            int age2 = (int) map2.getOrDefault("age", 0);
            return age1 - age2;
        }).collect(Collectors.toList());

        sortedCats.forEach(System.out::println);
    }
}

// 输出内容
{name=cat2, age=2}
{name=cat1, age=10}
```



## 6、Map与flatMap的区别

> 参考文献：Stream流中的Map与flatMap的区别：https://blog.csdn.net/xueyijin/article/details/125901455

### 1、先说结论

1. map的作用：是将输入一种类型，转化为另一种类型，通俗来说：就是将输入类型变成另一个类型。
2. flatMap的作用：是扁平化的，比如有对象`List<List<User>> userLists`, 这属于套娃现象，而最终我们只想得到`List<User> userList`对象，即将多维集合，变成一个集合，相当于压缩，俗称扁平化。



### 2、需求

1、之前在做开发的时候，遇到如下需求：给`List<Room>`对象，需要获取到`List<String> peopleNameList`对象

```java
// 解释：一个房间有自己的房间号，里面有一堆人，每个人都有面子
@Data
@AllArgsConstructor
@NoArgsConstructor
public static class Room {
    private int number;
    private List<People> peopleList;

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class People {
        private String name;
    }
}
```



### 3、解决方案

#### 3.1、使用Map方法

1、如果只是使用普通的map方法且里面是直接getPeopleList()方法，则得到的是 `List<List<Room.People>> collect` 对象

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        final List<Room> roomList = initRoom();
        List<List<Room.People>> collect = roomList.stream()
            .map(Room::getPeopleList)
            .collect(Collectors.toList());
    }
}
```

2、通常遇到这情况，一般人解决就是在遍历一次，如下：

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        final List<Room> roomList = initRoom();
        List<List<Room.People>> collect = roomList.stream()
            .map(Room::getPeopleList)
            .collect(Collectors.toList());
        // 1.先new ArrayList<>() 准备存储String字符串
        List<String> peopleNameList = new ArrayList<>();
        // 2. 这里就得套两层的foreach了
        // 因为初始化是List<List<T>>对象
        // 第一次foreach中peopleList 是List<Room.People>对象
        collect.forEach(peopleList -> {
            // 第二次foreach中people 是People对象
            peopleList.forEach(people -> {
                peopleNameList.add(people.getName());
            });
        });
    }
}
```

3、而有的人可能会在上面的基础上，再进一步优化为如下：

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        final List<Room> roomList = initRoom();

        List<String> peopleNameList = new ArrayList<>();
        // 直接一直使用stream流的foreach
        // 虽然是比上面代码优雅多了，但两个foreach，怎么看都不舒服
        roomList.stream().forEach(room -> {
            room.getPeopleList().forEach(people -> {
                peopleNameList.add(people.getName());
            });
        });
    }
}
```



#### 3.2、使用flatMap解决

flatMap就是用于多层结构的扁平化，如下：

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        final List<Room> roomList = initRoom();

        List<String> peopleNameList = roomList.stream()
            .map(Room::getPeopleList)          // 这里跟之前一样，生成多个List
            .flatMap(Collection::stream)       // 直接使用flatMap，使其直接压缩成一个List
            .map(Room.People::getName)         // 之后再用map将其变成String对象
            .collect(Collectors.toList());
    }
}
```



#### 3.3、完整代码

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        final List<Room> roomList = initRoom();

        // Map方法
        List<String> peopleNameList = new ArrayList<>();

        roomList.stream().forEach(room -> {
            room.getPeopleList().forEach(people -> {
                peopleNameList.add(people.getName());
            });
        });

        // flatMap 方法
        List<String> peopleNameList2 = roomList.stream()
            .map(Room::getPeopleList)
            .flatMap(Collection::stream)
            .map(Room.People::getName)
            .collect(Collectors.toList());

        System.out.println(peopleNameList2);
    }

    // 初始化10条数据
    private static List<Room> initRoom() {
        return IntStream.range(1, 11)
            .mapToObj(num -> new Room(num, Arrays.asList(new Room.People("name:" + num), new Room.People("name:" + 10 * num))))
            .collect(Collectors.toList());
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    static class Room {
        private int number;
        private List<People> peopleList;


        @Data
        @NoArgsConstructor
        @AllArgsConstructor
        static class People {
            private String name;
        }
    }
}
```

输出内容

```
[name:1, name:10, name:2, name:20, name:3, name:30, name:4, name:40, name:5, name:50, name:6, name:60, name:7, name:70, name:8, name:80, name:9, name:90, name:10, name:100]
```



# 四、Stream 终止操作

## 1、遍历（forEach）

| 方法名                | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| forEach(Comparator c) | 内部迭代（使用Collection接口需要用户去做迭代称为：外部迭代） |

```java
Stream.of("Java", "Python", "C#", "Vue").forEach(System.out::print);  // JavaPythonC#Vue
```



## 2、匹配（find/match）

| 方法名                 | 描述                     |
| ---------------------- | ------------------------ |
| findFirst()            | 返回第一个元素           |
| findAny()              | 返回当前流中任意元素     |
| allMatch(Predicate p)  | 检查是否匹配所有元素     |
| anyMatch(Predicate p)  | 检查是否至少匹配一个元素 |
| noneMatch(Predicate p) | 检查是否没有匹配所有元素 |

操作示例：allMatch()、noneMatch()、anyMatch()、findFirst()、findAny()

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

        // 匹配第一个
        Optional<Integer> findFirst = list.stream().filter(x -> x > 6).findFirst();
        // 匹配任意（适用于并行流）
        Optional<Integer> findAny = list.parallelStream().filter(x -> x > 6).findAny();
        System.out.println("匹配集合中第一个值：" + findFirst.get());
        System.out.println("匹配集合中任意一个值：" + findAny.get());
    }
}

// 输出内容
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
集合中是否有一个小于10的元素：true
集合中是否全部小于12：true
集合中是否全部不小于12：false
匹配集合中第一个值：2
匹配集合中任意一个值：7
```



## 3、聚合（max/min/count/sum）

| 方法名            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| max(Comparator c) | 返回流中最大值                                               |
| min(Comparator c) | 返回流中最小值                                               |
| count()           | 返回流中元素总数                                             |
| sum()             | 返回流中元素的和，只适用于IntStream。x.mapToInt(Student::getAge).sum() |

操作示例：max()、min()、count()

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student {
    private int age;
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Student> students = Arrays.asList(new Student(32), 
                                               new Student(33), 
                                               new Student(21), 
                                               new Student(29), 
                                               new Student(18));
        // 方式一:自然排序
        Optional<Student> max = students.stream().max(Comparator.comparingInt(Student::getAge));
        System.out.println(max.get().getAge()); // 32
        Optional<Student> min = students.stream().min(Comparator.comparingInt(Student::getAge));
        System.out.println(min.get().getAge()); // 18

        // 方式二:自定义排序（从小到大排序）,需要注意：自然排序是先排序后取值,如果调换return中两个参数的位置,max就变成min了
        // 最⼤
        Optional<Student> optional1 = students.stream().max((s1, s2) -> {
            // return s1.getAge() - s1.getAge();
            return Integer.compare(s1.getAge(), s2.getAge());
        });
        System.out.println(optional1.get().getAge()); // 32
        //最⼩
        Optional<Student> optional2 = students.stream().min((s1, s2) -> {
            // return Integer.compare(s2.getAge(), s1.getAge()); // 调换参数后结果就相反了
            return Integer.compare(s1.getAge(), s2.getAge());
        });
        System.out.println(optional2.get().getAge()); //18

        System.out.println(students.stream().count()); // 5
        
        // 总和
        Integer sum = list.stream().mapToInt(Student::getAge).sum();
        System.out.println(sum); // 76
    }
}

// 输出内容
33
18
33
18
5
76
```



## 4、规约（reduce）

归约，也称缩减，顾名思义，是把一个流缩减成一个值，能实现对集合求和、求乘积和求最值操作。

| 方法名                           | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| reduce(T iden, BinaryOperator b) | 可以将流中元素反复结合起来，得到一个值。返回T。iden 为初始值 |
| reduce(BinaryOperator b)         | 可以将流中元素反复结合起来，得到一个值。返回Optional         |

操作示例：

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 3, 2, 8, 11, 4);
        // 求和方式1
        Optional<Integer> sum = list.stream().reduce((x, y) -> x + y);
        // 求和方式2
        Optional<Integer> sum2 = list.stream().reduce(Integer::sum);
        // 求和方式3,这里设置初始值为1,总数会在List总和上+1
        Integer sum3 = list.stream().reduce(1, Integer::sum);

        // 求乘积
        Optional<Integer> product = list.stream().reduce((x, y) -> x * y);

        // 求最大值方式1
        Optional<Integer> max = list.stream().reduce((x, y) -> x > y ? x : y);
        // 求最大值写法2
        Integer max2 = list.stream().reduce(1, Integer::max);

        // 求最小值方式1,并且设置了初始值
        Optional<Integer> min = list.stream().reduce((x, y) -> x > y ? y : x);
        // 求最小值写法2,并且设置了初始值
        Integer min2 = list.stream().reduce(0, Integer::min);

        System.out.println("list求和：" + sum.get() + "," + sum2.get() + "," + sum3);
        System.out.println("list求积：" + product.get());
        System.out.println("list求最大值：" + max.get() + "," + max2);
        System.out.println("list求最小值：" + min.get() + "," + min2);
    }
}

// 输出内容
list求和：29,29,30
list求积：2112
list求最大值：11,11
list求最小值：1,0
```



## 5、收集（collect）

| 方法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| collect(Collector c) | 将流转为其他形式。接收一个Collector接口的实现，用于给Stream中元素汇总的方法 |

Collector 接口中方法的实现决定了如何对流执行收集操作(如收集到 List、 Set、 Map)。但是 Collectors 实用类提供了很多静态方法，可以方便地创建常见收集器实例， 具体方法与实例如下表：

| 方法名            | 返回类型                  | 作用                                                   |
| ----------------- | ------------------------- | ------------------------------------------------------ |
| toList            | List\<T\>                 | 把流中元素收集到List                                   |
| toSet             | Set\<T\>                  | 把流中元素收集到Set                                    |
| toCollection      | Collection\<T\>           | 把流中元素收集到创建的集合                             |
| toMap             | Map\<T\>                  | 把流中元素收集到Map                                    |
| counting          | Long                      | 计算流中元素的个数                                     |
| maxBy             | Optional\<T\>             | 根据比较器选择最大值                                   |
| minBy             | Optional\<T\>             | 根据比较器选择最小值                                   |
| summingInt        | Integer                   | 对流中元素的Integer属性求和                            |
| summingLong       | Long                      | 对流中元素的Long属性求和                               |
| summingDouble     | Double                    | 对流中元素的Double属性求和                             |
| averagingInt      | Double                    | 计算流中元素Integer属性的平均值                        |
| averagingLong     | Double                    | 计算流中元素Long属性的平均值                           |
| averagingDouble   | Double                    | 计算流中元素Double属性的平均值                         |
| summarizingInt    | IntSummaryStatistics      | 收集流中Integer属性的统计值.如平均值                   |
| summarizingLong   | LongSummaryStatistics     | 收集流中Long属性的统计值.如平均值                      |
| summarizingDouble | DoubleSummaryStatistics   | 收集流中Double属性的统计值.如平均值                    |
| joining           | String                    | 连接流中每个字符串                                     |
| reducing          | 规约产生的类型            | 利用BinaryOperator与流中元素逐个结合，从而归约成单个值 |
| groupingBy        | Map\<K, List\<T\>\>       | 根据某属性值对流分组，属性为K,结果为V                  |
| partitioningBy    | Map\<Boolean, List\<T\>\> | 根据true或者false进行分区                              |
| collectingAndThen | 转换函数返回的类型        | 包裹另一个收集器，对其结果转换函数                     |

每个方法对应的使用示例如下表所示：

| 方法              | 使用示例                                                     |
| ----------------- | ------------------------------------------------------------ |
| toList            | List employees = list.stream().collect(Collectors.toList())  |
| toSet             | Set employees = list.stream().collect(Collectors.toSet())    |
| toCollection      | Collection employees = list.stream().collect(Collectors.toCollection(ArrayList::new)) |
| toMap             |                                                              |
| counting          | long count = list.stream().collect(Collectors.counting())    |
| summingInt        | int total=list.stream().collect(Collectors.summingInt(Employee::getSalary)) |
| averagingInt      | double avg= list.stream().collect(Collectors.averagingInt(Employee::getSalary)) |
| summarizingInt    | IntSummaryStatistics iss= list.stream().collect(Collectors.summarizingInt(Employee::getSalary)) |
| joining           | String str= list.stream().map(Employee::getName).collect(Collectors.joining()); |
| maxBy             | Optionalmax= list.stream().collect(Collectors.maxBy(comparingInt(Employee::getSalary)))； |
| minBy             | Optional min = list.stream().collect(Collectors.minBy(comparingInt(Employee::getSalary))); |
| reducing          | int total=list.stream().collect(Collectors.reducing(0, Employee::getSalar, Integer::sum)); |
| collectingAndThen | int how= list.stream().collect(Collectors.collectingAndThen(Collectors.toList(), List::size)); |
| groupingBy        | Map<Emp.Status, List> map= list.stream() .collect(Collectors.groupingBy(Employee::getStatus)); |
| partitioningBy    | Map<Boolean,List>vd= list.stream().collect(Collectors.partitioningBy(Employee::getManage)); |



### 1、归集（toList/toSet/toMap）

| 方法名       | 返回类型        | 作用                       |
| ------------ | --------------- | -------------------------- |
| toList       | List\<T\>       | 把流中元素收集到List       |
| toSet        | Set\<T\>        | 把流中元素收集到Set        |
| toCollection | Collection\<T\> | 把流中元素收集到创建的集合 |
| toMap        | Map\<T\>        | 把流中元素收集到Map        |

1、将结果集收集到集合List、Set、TreeSet

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

2、将结果集收集到Map（List 转 Map）

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "二班"),
                new Student("王五", 15, "三班"));
        Map<String, String> map = students.stream()
            .collect(Collectors.toMap(Student::getName, Student::getClassName));
        System.out.println(map);
    }
}

// 输出内容
{李四=二班, 张三=一班, 王五=三班}
```

3、toMap解决key重复后异常，如果key重复会报异常，这时得指定重复用哪一个，得加上(oldValue, newValue) -> newValue)

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "二班"),
                new Student("老刘", 13, "二班"),
                new Student("王五", 15, "三班"));
        Map<String, String> map = students.stream().collect(Collectors.toMap(
                        Student::getClassName,
                        Student::getName,
                        (oldValue, newValue) -> newValue));
        System.out.println(map);
    }
}
// 输出内容
{一班=张三, 二班=老刘, 三班=王五}
```





### 2、统计（max/min/count/sum/avg）

| 方法名            | 返回类型                | 作用                                 |
| ----------------- | ----------------------- | ------------------------------------ |
| counting          | Long                    | 计算流中元素的个数                   |
| maxBy             | Optional\<T\>           | 根据比较器选择最大值                 |
| minBy             | Optional\<T\>           | 根据比较器选择最小值                 |
| summingInt        | Integer                 | 对流中元素的Integer属性求和          |
| summingLong       | Long                    | 对流中元素的Long属性求和             |
| summingDouble     | Double                  | 对流中元素的Double属性求和           |
| averagingInt      | Double                  | 计算流中元素Integer属性的平均值      |
| averagingLong     | Double                  | 计算流中元素Long属性的平均值         |
| averagingDouble   | Double                  | 计算流中元素Double属性的平均值       |
| summarizingInt    | IntSummaryStatistics    | 收集流中Integer属性的统计值.如平均值 |
| summarizingLong   | LongSummaryStatistics   | 收集流中Long属性的统计值.如平均值    |
| summarizingDouble | DoubleSummaryStatistics | 收集流中Double属性的统计值.如平均值  |

Collectors提供了一系列用于数据统计的静态方法：

1. 计数：counting
2. 最值：maxBy、minBy
3. 求和：summingInt、summingLong、summingDouble
4. 平均值：averagingInt、averagingLong、averagingDouble
5. 统计以上所有：summarizingInt、summarizingLong、summarizingDouble

操作示例：统计个数、平均数、数总额、最大最小值

```java
public class JavaAPIDemo {
    public static void main(String[] args) {

        // 初始化数据
        List<Integer> integerList = Arrays.asList(1, 11, 3, 8, 7, 9, 10);
        List<Long> longList = Arrays.asList(1L, 11L, 3L, 8L, 7L, 9L, 10L);
        List<Double> doubleList = Arrays.asList(1.0, 11.1, 3.3, 8.2, 7.5, 9.0, 10.0);

        // 总数(个数)
        Long integerCount = integerList.stream().collect(Collectors.counting());
        Long longCount = longList.stream().collect(Collectors.counting());
        Long douCount = doubleList.stream().collect(Collectors.counting());
        System.out.println("总数: integerCount: " + integerCount + "、longCount: " + longCount + "、douCount: " + douCount);

        // 最大值、最小值
        Integer integerMax = integerList.stream().collect(Collectors.maxBy(Integer::compare)).get();
        Integer integerMin = integerList.stream().collect(Collectors.minBy(Integer::compare)).get();
        Long longMax = longList.stream().collect(Collectors.maxBy(Long::compare)).get();
        Long longMin = longList.stream().collect(Collectors.minBy(Long::compare)).get();
        Double doubleMax = doubleList.stream().collect(Collectors.maxBy(Double::compare)).get();
        Double doubleMin = doubleList.stream().collect(Collectors.minBy(Double::compare)).get();
        System.out.println("最大值: integerMax: " + integerMax + "、longMax: " + longMax + "、doubleMax: " + doubleMax);
        System.out.println("最小值: integerMin: " + integerMin + "、longMin: " + longMin + "、doubleMin: " + doubleMin);

        // 总和
        Integer integerSum = integerList.stream().collect(Collectors.summingInt(Integer::intValue));
        Long longSum = longList.stream().collect(Collectors.summingLong(Long::intValue));
        Double doubleSum = doubleList.stream().collect(Collectors.summingDouble(Double::intValue));
        System.out.println("总和: integerSum: " + integerSum + "、longSum: " + longSum + "、doubleSum: " + doubleSum);

        // 平均值
        Double integerAve = integerList.stream().collect(Collectors.averagingInt(Integer::intValue));
        Double longAve = longList.stream().collect(Collectors.averagingLong(Long::intValue));
        Double doubleAve = doubleList.stream().collect(Collectors.averagingDouble(Double::intValue));
        System.out.println("平均值: integerSum: " + integerSum + "、longSum: " + longSum + "、doubleSum: " + doubleSum);

        // 统计所有信息
        IntSummaryStatistics intSummaryStatistics = integerList.stream().collect(Collectors.summarizingInt(Integer::intValue));
        LongSummaryStatistics longSummaryStatistics = longList.stream().collect(Collectors.summarizingLong(Long::intValue));
        DoubleSummaryStatistics doubleSummaryStatistics = doubleList.stream().collect(Collectors.summarizingDouble(Double::intValue));
        System.out.println("统计所有信息: intSummaryStatistics: " + intSummaryStatistics);
        System.out.println("统计所有信息: longSummaryStatistics: " + longSummaryStatistics);
        System.out.println("统计所有信息: doubleSummaryStatistics: " + doubleSummaryStatistics);
        // 分别获取总和，平均值，最大值或最小值信息
        System.out.println(summaryStatistics.getSum());
        System.out.println(summaryStatistics.getCount());
        System.out.println(summaryStatistics.getMax());
        System.out.println(summaryStatistics.getMix());
        System.out.println(summaryStatistics.getAverage());
    }
}

// 输出内容
总数: integerCount: 7、longCount: 7、douCount: 7
最大值: integerMax: 11、longMax: 11、doubleMax: 11.1
最小值: integerMin: 1、longMin: 1、doubleMin: 1.0
总和: integerSum: 49、longSum: 49、doubleSum: 49.0
平均值: integerSum: 49、longSum: 49、doubleSum: 49.0
统计所有信息: intSummaryStatistics: IntSummaryStatistics{count=7, sum=49, min=1, average=7.000000, max=11}
统计所有信息: longSummaryStatistics: LongSummaryStatistics{count=7, sum=49, min=1, average=7.000000, max=11}
统计所有信息: doubleSummaryStatistics: DoubleSummaryStatistics{count=7, sum=49.000000, min=1.000000, average=7.000000, max=11.000000}
49
7
11
1
7.000000
```

操作示例：统计对象的个数、平均数、数总额、最大最小值

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student {
    private Integer age;
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 初始化数据
        List<Student> list = List.of(
                new Student(10),
                new Student(12),
                new Student(12),
                new Student(14),
                new Student(13),
                new Student(15));

        /**
         * 实际上可以简写成如下:
         * long count = list.stream().count();
         */
        long count = list.stream().collect(Collectors.counting());
        System.out.println(count);

        /**
         * 最大值、最小值
         * 实际上可以简写成如下:
         * Optional<Student> max1 = list.stream().max(Comparator.comparing(Student::getAge));
         * Optional<Student> max2 = list.stream().max(Comparator.comparingInt(Student::getAge));
         */
        Optional<Student> max1 = list.stream().collect(Collectors.maxBy(Comparator.comparing(Student::getAge)));
        Optional<Student> max2 = list.stream().collect(Collectors.maxBy((x, y) -> x.getAge() - y.getAge()));
        Optional<Student> min1 = list.stream().collect(Collectors.minBy(Comparator.comparing(Student::getAge)));
        Optional<Student> min2 = list.stream().collect(Collectors.minBy((x, y) -> x.getAge() - y.getAge()));

        /**
         * 总和
         * 实际上可以简写成如下:
         * Integer integer = list.stream().mapToInt(Student::getAge).sum();
         */
        Integer sum = list.stream().collect(Collectors.summingInt(Student::getAge));
        System.out.println(sum);

        /**
         * 平均数
         */
        Double avg = list.stream().collect(Collectors.averagingInt(Student::getAge));
        System.out.println(avg);

        /**
         * 收集流中Integer属性所有的统计值.
         */
        IntSummaryStatistics summaryStatistics = list.stream().collect(Collectors.summarizingInt(Student::getAge));
        System.out.println(summaryStatistics);
        System.out.println(summaryStatistics.getSum());
        System.out.println(summaryStatistics.getCount());
        System.out.println(summaryStatistics.getMax());
        System.out.println(summaryStatistics.getMax());
        System.out.println(summaryStatistics.getAverage());
    }
}

// 输出内容
6
76
12.666666666666666
IntSummaryStatistics{count=6, sum=76, min=10, average=12.666667, max=15}
76
6
15
15
12.666666666666666
```



### 3、拼接（joining）

| 方法名  | 返回类型 | 作用               |
| ------- | -------- | ------------------ |
| joining | String   | 连接流中每个字符串 |

操作示例：`joining`可以将stream中的元素用特定的连接符（没有的话，则直接连接）连接成一个字符串。

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



### 4、归约（reducing）

| 方法名                                | 返回类型       | 作用                                                         |
| ------------------------------------- | -------------- | ------------------------------------------------------------ |
| reducing(BinaryOperator b)            | 规约产生的类型 | 利用BinaryOperator与流中元素逐个结合，返回Optional类型       |
| reducing(T iden, BinaryOperator b)    | 规约产生的类型 | 从一个作为累加器的初始值开始，<br />利用BinaryOperator与流中元素逐个结合，从而归约成单个值 |
| reducing(T, Function, BinaryOperator) | 规约产生的类型 | 同上                                                         |

`Collectors`类提供的`reducing`方法，相比于`stream`本身的`reduce`方法，增加了对自定义归约的支持。`reducing`有多个重载方法

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student {
    private Integer age;
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 初始化数据
        List<Student> list = Arrays.asList(new Student(12), new Student(10), new Student(35), new Student(28));

        // 使用 方法引用 + lambda 方式
        // 1.reducing(BinaryOperator b)
        Integer sum1 = list.stream().map(x -> x.getAge()).collect(Collectors.reducing(Integer::sum)).get();
        Integer sum2 = list.stream().map(x -> x.getAge()).collect(Collectors.reducing((x, y) -> x + y)).get();
        System.out.println(sum1 + "、" + sum2); // 85、85

        // 2.reducing(T iden, BinaryOperator b)
        Integer sum3 = list.stream().map(x -> x.getAge()).collect(Collectors.reducing(0, Integer::sum));
        Integer sum4 = list.stream().map(x -> x.getAge()).collect(Collectors.reducing(0, (x, y) -> x + y));
        System.out.println(sum3 + "、" + sum4); // 85、85

        // reducing(T iden, Function function, BinaryOperator b)
        Integer sum5 = list.stream().collect(Collectors.reducing(0, Student::getAge, Integer::sum));
        Integer sum6 = list.stream().collect(Collectors.reducing(0, x->x.getAge(), (x,y)-> x + y));
        System.out.println(sum5 + "、" + sum6); // 85、85
    }
}

// 输出内容
85、85
85、85
85、85
```



### 5、分组分片（groupingBy/partitioningBy）

| 方法名         | 返回类型                  | 作用                                   |
| -------------- | ------------------------- | -------------------------------------- |
| groupingBy     | Map\<K, List\<T\>\>       | 根据某属性值对流分组，属性为K，结果为V |
| partitioningBy | Map\<Boolean, List\<T\>\> | 根据true或者false进行分区              |

操作示例：groupingBy 分组。在一个集合中，对具有相同特性的值进行分组是一个很常见的功能，在Stream的API中也提供了相应的方法

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
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
{一班=[Student{name='张三', age=10, className='一班'},  Student{name='李四', age=12, className='一班'}], 
 二班=[Student{name='小吴', age=12, className='二班'}, Student{name='小刘', age=12, className='二班'}], 
 三班=[Student{name='小李', age=12, className='三班'}, Student{name='王五', age=15, className='三班'}]}
```

操作示例：partitoningBy 分片或者分区，当分类函数是一个返回布尔值的函数时，流元素会被分为两组列表：一组是返回true的元素集合，另一组是返回false的元素集合。这种情况适用partitoningBy方法会比groupingBy更有效率。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

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

// 输出内容
{false=[Student{name='张三', age=10, className='一班'}, 
        Student{name='李四', age=12, className='一班'}, 
        Student{name='小吴', age=12, className='二班'}], 
 true=[Student{name='小刘', age=14, className='二班'}, 
       Student{name='小李', age=13, className='三班'}, 
       Student{name='王五', age=15, className='三班'}]}
```



### 6、分组分区扩展（counting/summingInt..）

下面要介绍的这些方法功能，无论是groupingBy方法还是partitioningBy方法都是支持的。

1、Collectors.counting() 方法会返回收集元素的总个数

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

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

// 输出内容
{10=1, 12=2, 13=1, 14=1, 15=1}
```

2、Collectors.summingInt(ToIntFunction mapper)、Collectors.summingLong())、Collectors.summingDouble()，来计算总和

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

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

// 输出内容
{一班=22, 二班=26, 三班=28}
```

3、Collectors.maxBy(Comparator comparator)、CollectorsminBy()方法接受比较器作为参数来计算最大值和最小值

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 初始化数据
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));
        // 取出分组中年龄中最大和最小的学生
        // maxBy
        Map<String, Optional<Student>> maxMap  = students.stream().
                collect(Collectors.groupingBy(Student::getClassName,
                        Collectors.maxBy(Comparator.comparing(Student::getAge))));
        System.out.println(maxMap);
        
        // minBy
        Map<String, Optional<Student>> minMap  = students.stream().
                collect(Collectors.groupingBy(Student::getClassName,
                        Collectors.minBy(Comparator.comparing(Student::getAge))));
        System.out.println(minMap);
    }
}

// 输出内容
{一班=Optional[Student{name='李四', age=12, className='一班'}], 
 二班=Optional[Student{name='小刘', age=14, className='二班'}], 
 三班=Optional[Student{name='王五', age=15, className='三班'}]}

{一班=Optional[Student{name='张三', age=10, className='一班'}], 
 二班=Optional[Student{name='小吴', age=12, className='二班'}], 
 三班=Optional[Student{name='小李', age=13, className='三班'}]}
```

4、Collectors.mapping() 方法会将结果应用到另一个收集器上。取出分组中年龄最大的学生的年龄

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 初始化数据
        List<Student> students = Arrays.asList(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));
        // 取出分组中年龄最大的学生的年龄
        Map<String, Optional<Integer>> collect = students.stream().collect(Collectors.groupingBy(Student::getClassName,
                Collectors.mapping(Student::getAge, Collectors.maxBy(Comparator.comparing(Integer::valueOf)))));
        System.out.println(collect);
    }
}

// 输出内容
{一班=Optional[12], 二班=Optional[14], 三班=Optional[15]}
```



### 7、CollectingAndThen

| 方法名            | 返回类型           | 作用                               |
| ----------------- | ------------------ | ---------------------------------- |
| collectingAndThen | 转换函数返回的类型 | 包裹另一个收集器，对其结果转换函数 |

| 方法名            | 示例                                                         |
| ----------------- | ------------------------------------------------------------ |
| collectingAndThen | Int count = Menu.getMenus.stream().collect(collectingAndThen(toList(),List::size)); |

根据函数名称可以简单理解为先collect然后再对其结果进行后续操作。

```java
public static<T,A,R,RR> Collector<T,A,RR> collectingAndThen(Collector<T,A,R> downstream,Function<R,RR> finisher)
```

操作示例：简单案例

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<String> list1 = List.of("1", "2", "3", "4", "100");

        // averagingInt 后返回的是Double类型, 这里通过Double.intValue() 转换成 int 类型
        int avg1 = list1.stream().collect(Collectors.averagingInt(Integer::parseInt)).intValue();
        // 使用collectingAndThen, 参数1收集collector结果, 参数二
        int avg2 = list1.stream().collect(Collectors.collectingAndThen(Collectors.averagingInt(Integer::parseInt), a -> a.intValue()));
        System.out.println(avg1 + "、" + avg2); // 22、22

        /**
         * 1.首先对 [1, 2, 3, 4] 每个值 * 2, 返回 [2, 4, 6, 8]
         * 2.然后averagingLong取平均值, 返回: 5
         * 3.通过参数2的lambda s * s = 5 * 5 返回: 25
         */
        List<Integer> list2 = List.of(1, 2, 3, 4);
        Double result = list2.stream().collect(
                Collectors.collectingAndThen(
                        Collectors.averagingLong(v -> v * 2), s -> s * s));
        System.out.println(result); // 25
    }
}

// 输出内容
22、22
25
```

操作示例：复杂案例

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Student{
    private String name;
    private Integer age;
    private String className;
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 初始化数据
        List<Student> students = List.of(
                new Student("张三", 10, "一班"),
                new Student("李四", 12, "一班"),
                new Student("小吴", 12, "二班"),
                new Student("小刘", 14, "二班"),
                new Student("小李", 13, "三班"),
                new Student("王五", 15, "三班"));

        // 根据className对集合的结果进行去重
        List<Student> list = students.stream().collect(Collectors.collectingAndThen(
                        Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(Student::getClassName))),
                        ArrayList::new));
        System.out.println(list);



        // 查找age最大学生的姓名
        // 注意为什么会使用Comparator.comparing(Student::getAge),因为他会返回Student对象,如果直接使用Student::getAge则返回Integer
        // Optional<Student> collect = students.stream().collect(Collectors.maxBy(Comparator.comparing(Student::getAge)));
        String collect = students.stream()
                .collect(Collectors.collectingAndThen(
                        Collectors.maxBy(Comparator.comparing(Student::getAge)),
                        (Optional<Student> student) -> student.map(Student::getName).orElse(null)));
        System.out.println(collect);
    }
}

// 输出内容
[Student(name=张三, age=10, className=一班), Student(name=小李, age=13, className=三班), Student(name=小吴, age=12, className=二班)]
王五
```



# 五、参考文献 & 鸣谢

1. Java8 Stream 详细总结目录：https://blog.csdn.net/winterking3/article/details/116310016
2. Java8 Stream：2万字20个实例，玩转集合的筛选、归约、分组、聚合：https://blog.csdn.net/mu_wind/article/details/109516995
3. 波波烤鸭，CSDN：https://dpb-bobokaoya-sm.blog.csdn.net/article/details/117591234
4. 纪莫，博客园：https://www.cnblogs.com/jimoer/p/10995574.html
5. 【Java8新特性】面试官：谈谈Java8中的Stream API有哪些终止操作？https://blog.csdn.net/l1028386804/article/details/106416723
6. Java 8 Stream API学习总结：https://blog.csdn.net/qq_28336351/article/details/106153368
