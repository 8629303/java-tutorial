# 获取List交集、补集、并集、差集的几种方式

> 取两个集合List的交集、补集、并集、差集的几种方式：https://blog.csdn.net/whxjason/article/details/105506787

首先知道几个单词的意思：

- 交集 = intersection
- 并集 = union
- 补集 = complement
- 析取 = disjunction
- 减去 = subtract

## 一、Apache 工具包

1、导入依赖

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

2、代码如下：

```java
public class TestApache {

    public static String[] attr1 = new String[]{"A", "B", "C", "D", "E", "F", null};
    public static String[] attr2 = new String[]{"1", "2", "3", "D", "E", "F", null};
    public static List<String> list1 = Arrays.asList(attr1);
    public static List<String> list2 = Arrays.asList(attr2);

    public static void main(String[] args) {
        System.out.println("交集：" + CollectionUtils.intersection(list1, list2)); // 交集
        System.out.println("补集：" + CollectionUtils.disjunction(list1, list2)); // 补集
        System.out.println("并集：" + CollectionUtils.union(list1, list2)); // 并集
        System.out.println("list1的差集：" + CollectionUtils.subtract(list1, list2)); // list1的差集
        System.out.println("list2的差集：" + CollectionUtils.subtract(list2, list1)); // list2的差集
    }
}
```

3、输出结果如下：

```
交集：[null, D, E, F]
补集：[A, 1, B, 2, C, 3]
并集：[null, A, 1, B, 2, C, 3, D, E, F]
list1的差集：[A, B, C]
list2的差集：[1, 2, 3]
```



## 二、Hutool 工具包

1、导入依赖

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.4.2</version>
</dependency>
```

2、代码如下：

```java
public class TestHutool {
    public static String[] attr1 = new String[]{"A", "B", "C", "D", "E", "F", null};
    public static String[] attr2 = new String[]{"1", "2", "3", "D", "E", "F", null};

    public static void main(String[] args) {
        ArrayList<String> list1 = new ArrayList<>(Arrays.asList(attr1));
        ArrayList<String> list2 = new ArrayList<>(Arrays.asList(attr2));

        System.out.println("交集：" + CollectionUtil.intersection(list1, list2)); // 交集
        System.out.println("补集：" + CollectionUtil.disjunction(list1, list2)); // 补集
        System.out.println("并集：" + CollectionUtil.union(list1, list2)); //并集
        System.out.println("list1的差集"+CollectionUtil.subtract(list1,list2));
        System.out.println("list2的差集"+CollectionUtil.subtract(list2,list1));
        System.out.println("list1的差集：" + CollectionUtil.subtractToList(list1, list2));
        System.out.println("list2的差集：" + CollectionUtil.subtractToList(list2, list1));
    }
}
```

3、输出结果如下：

```
交集：[null, D, E, F]
补集：[1, A, 2, B, 3, C]
并集：[null, 1, A, 2, B, 3, C, D, E, F]
list1的差集[A, B, C]
list2的差集[1, 2, 3]
list1的差集：[A, B, C]
list2的差集：[1, 2, 3]
```

4、注意：subtract()和subtractToList()作用一样，不过处理方式不同，使用subtract()时，参数不能是Arrays.asList()的结果，否则会报异常，因为Arrays.asList()返回的对象是Arrays.ArrayList，在方法实现里面调用的是AbstractList抽象类里面的removeAll()方法，该方法代码如下：

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

    // ....

    public E remove(int index) {
        throw new UnsupportedOperationException();
    }

    // ....
}

```



## 三、Stream 方式

1、代码如下：

```java
public class TestStream {

    public static String[] attr1 = new String[]{"A", "B", "C", "D", "E", "F", null};
    public static String[] attr2 = new String[]{"1", "2", "3", "D", "E", "F", null};
    static List<String> list1 = Arrays.asList(attr1);
    static List<String> list2 = Arrays.asList(attr2);

    public static void main(String[] args) {
        List<Object> intersection = list1.stream().filter(item -> list2.contains(item)).collect(Collectors.toList());
        System.out.println("交集：" + intersection);

        List<String> subtract1 = list1.stream().filter(s -> !list2.contains(s)).collect(Collectors.toList());
        System.out.println("集合list1的差集：" + subtract1);
        List<String> subtract2 = list2.stream().filter(s -> !list1.contains(s)).collect(Collectors.toList());
        System.out.println("集合list2的差集：" + subtract2);

        List<String> union1 = list1.parallelStream().collect(Collectors.toList());
        List<String> union2 = list2.parallelStream().collect(Collectors.toList());
        union1.addAll(union2);
        List<String> union3 = union1.stream().distinct().collect(Collectors.toList());
        System.out.println("并集：" + union3);
    }
}
```

2、结果：

```
交集：[D, E, F, null]
集合list1的差集：[A, B, C]
集合list2的差集：[1, 2, 3]
并集：[A, B, C, D, E, F, null, 1, 2, 3]
```



## 四、Collection 接口中的方法

1、代码如下：

```java
public class TestCollection {

    public static String[] attr1 = new String[]{"A", "B", "C", "D", "E", "F", null};
    public static String[] attr2 = new String[]{"1", "2", "3", "D", "E", "F", null};

    public static void main(String[] args) {
        ArrayList<String> list1 = new ArrayList<>(Arrays.asList(attr1));
        ArrayList<String> list2 = new ArrayList<>(Arrays.asList(attr2));
        list1.retainAll(list2);
        System.out.println("交集：" + list1);

        ArrayList<String> list3 = new ArrayList<>(Arrays.asList(attr1));
        ArrayList<String> list4 = new ArrayList<>(Arrays.asList(attr2));
        HashSet<Object> set = new HashSet<>();
        set.addAll(list3);
        set.addAll(list4);
        System.out.println("并集：" + set);

        ArrayList<String> list5 = new ArrayList<>(Arrays.asList(attr1));
        ArrayList<String> list6 = new ArrayList<>(Arrays.asList(attr2));
        list5.removeAll(list6);
        System.out.println("集合A的差集：" + list5);
        ArrayList<String> list7 = new ArrayList<>(Arrays.asList(attr1));
        ArrayList<String> list8 = new ArrayList<>(Arrays.asList(attr2));
        list8.removeAll(list7);
        System.out.println("集合B的差集：" + list8);
    }
}
```

2、输出结果：

```
交集：[D, E, F, null]
并集：[null, A, 1, B, 2, C, 3, D, E, F]
集合A的差集：[A, B, C]
集合B的差集：[1, 2, 3]
```