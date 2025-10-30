> 来源：Lambda 实现超强排序：https://mp.weixin.qq.com/s/WMnQSjA91yiF6dQD6TZs_g

# 1、Lambda 排序前言

我们在系统开发过程中，对数据排序是很常见的场景。一般来说，我们可以采用两种方式：

1. 借助存储系统（SQL、NoSQL、NewSQL 都支持）的排序功能，查询的结果即是排好序的结果
2. 查询结果为无序数据，在内存中排序。

今天介绍的是第二种，在内存中实现数据排序。首先，我们定义一个基础类，后面我们将根据这个基础类演示如何在内存中排序。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    private String name;
    private int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Student student = (Student) o;
        return age == student.age && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```



# 2、基于 Comparator 排序

在 Java 8 之前，我们都是通过实现 Comparator 接口完成排序，比如：

```java
new Comparator<Student>() {
    @Override
    public int compare(Student h1, Student h2) {
        return h1.getName().compareTo(h2.getName());
    }
};
```

这里展示的是匿名内部类的定义，如果是通用的对比逻辑，可以直接定义一个实现类。使用起来也比较简单，如下就是应用：

```java
@Test
void baseSortedOrigin() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    Collections.sort(students, new Comparator<Student>() {
        @Override
        public int compare(Student h1, Student h2) {
            return h1.getName().compareTo(h2.getName());
        }
    });
    Assertions.assertEquals(students.get(0), new Student("Jerry", 12));
}
```

> 这里使用了 Junit5 实现单元测试，用来验证逻辑非常适合

因为定义的 Comparator 是使用 name 字段排序，在 Java 中，String 类型的排序是通过单字符的 ASCII 码顺序判断的，J在T的前面，所以Jerry排在第一个。



# 3、使用 Lambda 替换 Comparator 匿名内部类

1、我们可以在 Student 中定义一个静态方法：

```java
public static int compareByNameThenAge(Student s1, Student s2) {
    if (s1.name.equals(s2.name)) {
        return Integer.compare(s1.age, s2.age);
    } else {
        return s1.name.compareTo(s2.name);
    }
}
```

这个方法需要返回一个 int 类型参数，在 Java 8 中，我们可以在 Lambda 中使用该方法：

```java
@Test
void sortedUsingStaticMethod() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    students.sort(Student::compareByNameThenAge);
    Assertions.assertEquals(students.get(0), new Student("Jerry", 12));
}
```



# 4、使用 Comparator 的 comparing 方法

在 Java 8 中，Comparator 类新增了 comparing 方法，可以将传递的 Function 参数作为比较元素，比如：

```java
@Test
void sortedUsingComparator() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    students.sort(Comparator.comparing(Student::getName));
    Assertions.assertEquals(students.get(0), new Student("Jerry", 12));
}
```



# 5、多条件排序

我们在静态方法一节中展示了多条件排序，还可以在 Comparator 匿名内部类中实现多条件逻辑：

```java
@Test
void sortedMultiCondition() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12),
        new Student("Jerry", 13)
    );
    students.sort((s1, s2) -> {
        if (s1.getName().equals(s2.getName())) {
            return Integer.compare(s1.getAge(), s2.getAge());
        } else {
            return s1.getName().compareTo(s2.getName());
        }
    });
    Assertions.assertEquals(students.get(0), new Student("Jerry", 12));
}
```

从逻辑来看，多条件排序就是先判断第一级条件，如果相等，再判断第二级条件，依次类推。在 Java 8 中可以使用 comparing 和一系列 thenComparing 表示多级条件判断，上面的逻辑可以简化为：

```java
@Test
void sortedMultiConditionUsingComparator() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12),
        new Student("Jerry", 13)
    );
    students.sort(Comparator.comparing(Student::getName).thenComparing(Student::getAge));
    Assertions.assertEquals(students.get(0), new Student("Jerry", 12));
}
```

这里的 thenComparing 方法是可以有多个的，用于表示多级条件判断，这也是函数式编程的方便之处。



# 6、Stream 进行排序

Java 8 中，不但引入了 Lambda 表达式，还引入了一个全新的流式 API：Stream API，其中也有 sorted 方法用于流式计算时排序元素，可以传入 Comparator 实现排序逻辑：

```java
@Test
void streamSorted() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    final Comparator<Student> comparator = (h1, h2) -> h1.getName().compareTo(h2.getName());
    final List<Student> sortedStudents = students.stream()
        .sorted(comparator)
        .collect(Collectors.toList());
    Assertions.assertEquals(sortedStudents.get(0), new Student("Jerry", 12));
}
```

同样的，我们可以通过 Lambda 简化书写：

```java
@Test
void streamSortedUsingComparator() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    final Comparator<Student> comparator = Comparator.comparing(Student::getName);
    final List<Student> sortedStudents = students.stream()
        .sorted(comparator)
        .collect(Collectors.toList());
    Assertions.assertEquals(sortedStudents.get(0), new Student("Jerry", 12));
}
```



# 7、倒序排列

## 1、调转排序判断

排序就是根据 compareTo 方法返回的值判断顺序，如果想要倒序排列，只要将返回值取返即可：

```java
@Test
void sortedReverseUsingComparator2() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    final Comparator<Student> comparator = (h1, h2) -> h2.getName().compareTo(h1.getName());
    students.sort(comparator);
    Assertions.assertEquals(students.get(0), new Student("Tom", 10));
}
```

可以看到，正序排列的时候，我们是`h1.getName().compareTo(h2.getName())`，这里我们直接倒转过来，使用的是`h2.getName().compareTo(h1.getName())`，也就达到了取反的效果。在 Java 的 Collections中 定义了一个`java.util.Collections.ReverseComparator`内部私有类，就是通过这种方式实现元素反转。



## 2、Comparator 的 reversed 方法倒序

在 Java8 中新增了 reversed 方法实现倒序排列，用起来也是很简单：

```java
@Test
void sortedReverseUsingComparator() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    final Comparator<Student> comparator = (h1, h2) -> h1.getName().compareTo(h2.getName());
    students.sort(comparator.reversed());
    Assertions.assertEquals(students.get(0), new Student("Tom", 10));
}
```



## 3、Comparator.comparing 中定义排序反转

comparing 方法还有一个重载方法，第二个参数就可以传入`Comparator.reverseOrder()`，可以实现倒序：

```java
java.util.Comparator#comparing(java.util.function.Function<? super T,? extends U>, java.util.Comparator<? super U>)
```

```java
@Test
void sortedUsingComparatorReverse() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    students.sort(Comparator.comparing(Student::getName, Comparator.reverseOrder()));
    Assertions.assertEquals(students.get(0), new Student("Jerry", 12));
}
```



## 4、Stream 排序反转

在 Stream 中的操作与直接列表排序类似，可以反转 Comparator 定义，也可以使用 Comparator.reverseOrder() 反转。实现如下：

```java
@Test
void streamReverseSorted() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    final Comparator<Student> comparator = (h1, h2) -> h2.getName().compareTo(h1.getName());
    final List<Student> sortedStudents = students.stream()
        .sorted(comparator)
        .collect(Collectors.toList());
    Assertions.assertEquals(sortedStudents.get(0), new Student("Tom", 10));
}

@Test
void streamReverseSortedUsingComparator() {
    final List<Student> students = Lists.newArrayList(
        new Student("Tom", 10),
        new Student("Jerry", 12)
    );
    final List<Student> sortedStudents = students.stream()
        .sorted(Comparator.comparing(Student::getName, Comparator.reverseOrder()))
        .collect(Collectors.toList());
    Assertions.assertEquals(sortedStudents.get(0), new Student("Tom", 10));
}
```



# 8、null 值的判断

前面的例子中都是有值元素排序，能够覆盖大部分场景，但有时候我们还是会碰到元素中存在 null 的情况：

1. 列表中的元素是 null
2. 列表中的元素参与排序条件的字段是 null

如果还是使用前面的那些实现，我们会碰到 NullPointException 异常，即 NPE，简单演示一下：

```java
@Test
void sortedNullGotNPE() {
    final List<Student> students = Lists.newArrayList(
        null,
        new Student("Snoopy", 12),
        null
    );
    Assertions.assertThrows(NullPointerException.class,
                            () -> students.sort(Comparator.comparing(Student::getName)));
}
```

所以，我们需要考虑这些场景。



## 1、元素是 null 的笨拙实现

最先想到的就是判空：

```java
@Test
void sortedNullNoNPE() {
    final List<Student> students = Lists.newArrayList(
        null,
        new Student("Snoopy", 12),
        null
    );
    students.sort((s1, s2) -> {
        if (s1 == null) {
            return s2 == null ? 0 : 1;
        } else if (s2 == null) {
            return -1;
        }
        return s1.getName().compareTo(s2.getName());
    });

    Assertions.assertNotNull(students.get(0));
    Assertions.assertNull(students.get(1));
    Assertions.assertNull(students.get(2));
}
```

我们可以将判空的逻辑抽取出一个 Comparator，通过组合方式实现：

```java
class NullComparator<T> implements Comparator<T> {
    private final Comparator<T> real;

    NullComparator(Comparator<? super T> real) {
        this.real = (Comparator<T>) real;
    }

    @Override
    public int compare(T a, T b) {
        if (a == null) {
            return (b == null) ? 0 : 1;
        } else if (b == null) {
            return -1;
        } else {
            return (real == null) ? 0 : real.compare(a, b);
        }
    }
}
```

在 Java8 中已经为我们准备了这个实现。



## 2、Comparator.nullsLast & Comparator.nullsFirst

1、使用 Comparator.nullsLast 实现 null 在结尾：

```java
@Test
void sortedNullLast() {
    final List<Student> students = Lists.newArrayList(
        null,
        new Student("Snoopy", 12),
        null
    );
    students.sort(Comparator.nullsLast(Comparator.comparing(Student::getName)));
    Assertions.assertNotNull(students.get(0));
    Assertions.assertNull(students.get(1));
    Assertions.assertNull(students.get(2));
}
```

2、使用 Comparator.nullsFirst 实现 null 在开头：

```java
@Test
void sortedNullFirst() {
    final List<Student> students = Lists.newArrayList(
        null,
        new Student("Snoopy", 12),
        null
    );
    students.sort(Comparator.nullsFirst(Comparator.comparing(Student::getName)));
    Assertions.assertNull(students.get(0));
    Assertions.assertNull(students.get(1));
    Assertions.assertNotNull(students.get(2));
}
```

是不是很简单，接下来我们看下如何实现排序条件的字段是 null 的逻辑。



## 3、排序条件的字段是 null

这个就是借助 Comparator 的组合了，就像是套娃实现了，需要使用两次 Comparator.nullsLast，这里列出实现：

```java
@Test
void sortedNullFieldLast() {
    final List<Student> students = Lists.newArrayList(
        new Student(null, 10),
        new Student("Snoopy", 12),
        null
    );
    final Comparator<Student> nullsLast = Comparator.nullsLast(
        Comparator.nullsLast( // 1
            Comparator.comparing(
                Student::getName,
                Comparator.nullsLast( // 2
                    Comparator.naturalOrder() // 3
                )
            )
        )
    );
    students.sort(nullsLast);
    Assertions.assertEquals(students.get(0), new Student("Snoopy", 12));
    Assertions.assertEquals(students.get(1), new Student(null, 10));
    Assertions.assertNull(students.get(2));
}
```

代码逻辑如下：

1. 代码 1 是第一层 null-safe 逻辑，用于判断元素是否为 null；
2. 代码 2 是第二层 null-safe 逻辑，用于判断元素的条件字段是否为 null；
3. 代码 3 是条件 Comparator，这里使用了 Comparator.naturalOrder()，是因为使用了 String 排序，也可以写为`String::compareTo`。如果是复杂判断，可以定义一个更加复杂的 Comparator，组合模式就是这么好用，一层不够再套一层。