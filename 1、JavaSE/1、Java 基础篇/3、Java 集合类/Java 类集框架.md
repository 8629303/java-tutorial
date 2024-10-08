# 类集简介

## 1、Java 类集框架简介

从JDK1.2开始Java引入了类集开发框架，所谓的类集指的就是一套动态对象数组的实现方案，在实际的开发之中没有任何项的开发可以离开数组，但是传统的数组实现起来非常的繁琐，而且长度是其致命伤，正是因为长度的问题，所以传统的数组是不可能大范围使用的，但是我们的开发又不可能离开数组，所以最初就只能依靠一些数据结构来实现动态的数组处理，而其中最为重要的两个结构：链表、树，但是面对这些数据结构的实现又不得不面对如下的一些问题：

1. 数据结构的代码实现困难,对于一般的开发者是无法进行使用的
2. 对于链表或二叉树当进行更新处理的时候的维护是非常麻烦的
3. 对于链表或二叉树还需要尽可能保证其操作的性能

正是因为这样的原因，所以从JDK1.2开始Java引入了类集，主要就是对常见的数据结构进行完整的实现包装，并且提供有一系列的接口与实现子类来帮助用户减少数据结构所带来的开发困难，但是最初的类集实现由于Java本身的技术所限，所以对于数据的控制并不严格，全部采用了 Object 类型进行数据接收，而在JDK1.5之后由于泛型技术的推广，所以类集本身也得到了良好的改进，可以直接利用泛型来保存相同类型的数据，并且随着数据量的不断增加，从JDK1.8开始类集中的实现算法也得到了良好的性能提升。

在整个类集框架里面提供有如下的几个核心接口：Collection、Lits、Set、Map、Iterator、ListIterator、Enumeration、Queue

![20220106212041](Java 类集框架.assets/20220106212041.png)



## 2、Collection 集合接口

java.util.Collection是（单值）集合操作的最大的父接口，在该接口中定义有所有的单值数据的处理操作，这个接口中定义了如下的核心操作方法：
单值集合：每次操作只能保存一个数据对象。

```java
public boolean add(E e);                          // 向集合保存数据
public boolean addAll(Collection<? extends E> c); // 追加一组数据
public void clear();                              // 清空集合，让根节点为空，同时执行GC处理
public boolean contains(Object o);                // 查询数据是否存在，需要equals()方法支持
public boolean remove(Object o);                  // 数据删除，需要equals()方法支持
public int size();                                // 获取数据长度，最大值为：Integer.MAX_VALUE
public Object[] toArray();                        // 将集合变为对象数组返回
public Iterator<E> iterator();                    // 将集合变为Iterator接口返回
public ListIterator<E> listIterator();            // 将集合变为ListIterator接口返回
```

在进行集合操作时，有两个方法最为常用：【增加】add()、【输出】iterator()

在JDK1.5版本之前，Collection只是一个独立的接口，但是从JDK1.5后，提供了Iterable父接口，并且在JDK1.8后针对于Iterable接口也得到了一些扩充。另外，在JDK1.2~JDK1.4的时代里面，如果要进行集合的使用往往会直接操作Collection接口，但是从JDK1.5时代开始更多的情况下选择的都是Collection的两个子接口：允许重复的List子接口、不允许重复的Set子接口；

Collection接口结构：

![image.png](Java 类集框架.assets/45d7da78aa8f4f36b2fde8db1cb0dc5b.png)



# List 集合

本节介绍了List接口的使用，在List子接口中有三个常用子类：ArrayList、Vector、LinkedList，之后会做详细说明。

## 1、List 接口简介

List是Collection的子接口，其最大的特点是允许保存有重复元素数据，该接口的定义如下：

```java
public interface List<E> extends Collection<E> {}
```

但是需要清楚的是List子接口对于Collection接口进行了方法扩充：

```java
public E get(int index);                                 // 获取指定索引上的数据
public E set(int index, E element);                      // 修改指定索引数据
public Iterator<E> iterator();                           // 返回Iterator接口对象
public ListIterator<E> listIterator();                   // 返回ListIterator接口对象
public static <E> List<E> of(E... elements);             // 将数据转为List集合（JDK1.9）
public default void forEach(Consumer<? super T> action); // 使用foreach结合消费型接口输出
```

但是List本身依然属于一个接口，那么对于接口要想使用则一定要使用子类来完成定义，在List子接口中有三个常用子类：ArrayList、Vector、LinkedList

![image.png](Java 类集框架.assets/f84b09006ff54f83acc26abe7e3e86c1.png)

操作示例：观察List中的静态方法（JDK1.9后的方法）从JDK1.9开始，List子接口里面追加有一些 statIc方法，以方便用户的处理

```java
import java.util.List;
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<String> all = List.of("Hello", "World", "饿了么？");
        System.out.println(all);
        System.out.println(all.getClass());
    }
}

// 输出内容
[Hello, World, 饿了么？]
class java.util.ImmutableCollections$ListN
```

这些操作方法并不是List的传统用法，是在新版本JDK1.9之后添加的新功能。



## 2、ArrayList 子类

ArrayList是List子接口中使用最多的一个子类，但是这个子类在使用时也是有前提要求的，所以本次来对这个类的相关定义以及源代码组成进行分析，在Java里面ArrayList类的定义如下：

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```

ArrayList的继承结构

![image.png](Java 类集框架.assets/3bc2e589f057473189b65218d960d58b.png)

操作示例 1：使用ArrayList实例化List父接口

```java
import java.util.ArrayList;
import java.util.List;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 为List父接口进行实例化
        List<String> all = new ArrayList<>();
        all.add("Hello") ;
        all.add("Hello") ;	// 重复数据
        all.add("World") ;
        all.add("XXXX") ;
        System.out.println(all);
    }
}

// 输出内容
[Hello, Hello, World, XXXX]
```

通过本程序可以发现List存储的特征：

1. 保存的顺序就是其存储顺序
2. List集合里面允许存在有重复数据

操作示例 2：利用forEach()进行输出（不是标准输出）在JDK1.8之后Iterable父接口中定义forEach()方法输出支持：

```java
import java.util.ArrayList;
import java.util.List;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 为List父接口进行实例化
        List<String> all = new ArrayList<>();
        all.add("Hello");
        all.add("Hello");    // 重复数据
        all.add("World");
        all.add("XXXX");
        all.forEach(str -> System.out.print(str + "、"));
    }
}

// 输出内容
Hello、Hello、World、XXXX、
```

需要注意的是：此种输出并不是正常开发情况下要考虑的操作形式。

操作示例 3：观察List集合的其它操作方法

```java
import java.util.ArrayList;
import java.util.List;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 为List父接口进行实例化
        List<String> all = new ArrayList<>();
        System.out.println("集合是否为空？" + all.isEmpty() + "、集合元素个数：" + all.size());
        all.add("Hello") ;
        all.add("Hello") ;	// 重复数据
        all.add("World") ;
        all.add("XXXX") ;
        all.remove("Hello") ; // 删除元素
        System.out.println("集合是否为空？" + all.isEmpty() + "、集合元素个数：" + all.size());
        all.forEach(str -> System.out.print(str + "、"));
    }
}

// 输出内容
集合是否为空？true、集合元素个数：0
集合是否为空？false、集合元素个数：3
Hello、World、XXXX、
```

如果以方法的功能为例，那么ArrayList中操作支持与之前编写的链表形式是非常相似的，但是它并不是使用链表来实现的，通过类名称实际上就已经可以清楚的发现了，ArrayList应该封装的是一个数组。



## 3、ArrayList 源码分析

ArrayList构造：public ArrayList()

```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

ArrayList构造：public ArrayList(int initialCapacity)

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}
```

通过有参构造方法可以发现，在ArrayList中所包含的数据实际上就是一个对象数组。在进行数据追加时发现ArrayList集合中保存的对象数组长度不够的时候将会开辟新的数组，同时将原始的旧数组内容拷贝到新数组中，而后数组的开辟操作：

```java
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0) ? newCapacity  : hugeCapacity(minCapacity);
}
```

如果在实例化ArrayList类对象时没有传递初始化的长度，则默认情况下会使用空数组，但是如果在进行数据增加时，发现数组容量不够，则会判断当前的增长容量与默认的容量的大小，使用较大的一个数值进行新的数组开辟，所以可以得出结论：

1. JDK1.9之前：ArrayList默认的构造实际上就会默认开辟大小为10的数组
2. JDK1.9之后：ArrayList默认的构造只会使用默认的空数组，使用的时候才会开辟数组，默认的开辟长度为10

当 ArrayList之中保存的容量不足的时候会采用成倍的方式进行增长，原始长度为10，那么下次的增长就是20，一次类推。在使用ArrayList子类的时候一定要估算出你的数据量会有多少，如果超过了10个，那么使用有参构造方法创建，以避免垃圾数组产生。



## 3、ArrayList 保存自定义类

通过之前的分析已经清楚了ArrayList子类的实现原理以及List核心操作，但是在测试的时候使用的是系统提供的String类，这是一个设计非常完善的类，而对于类集而言也可以实现自定义类对象的保存。

操作示例：实现自定义类对象的保存

```java
import java.util.ArrayList;
import java.util.List;
class Person {
    private String name ;
    private int age ;
    public Person(String name,int age) {
        this.name = name ;
        this.age = age ;
    }
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true ;
        }
        if (obj == null) {
            return false ;
        }
        if (!(obj instanceof Person)) {
            return false ;
        }
        Person per = (Person) obj ;
        return this.name.equals(per.name) && this.age == per.age ;
    }
    @Override
    public String toString() {
        return "姓名：" + this.name + "、年龄：" + this.age ;
    }
}
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<Person> all = new ArrayList<>();
        all.add(new Person("张三",30)) ;
        all.add(new Person("李四",16)) ;
        all.add(new Person("小强",78)) ;
        System.out.println(all.contains(new Person("小强",78)));
        all.remove(new Person("小强",78)) ;
        all.forEach(System.out::println);
    }
}

// 输出内容
true
姓名：张三、年龄：30
姓名：李四、年龄：16
```

在使用List保存自定义对象时，如果需要使用到contains()、remove()方法进行查询或删除处理时一定要保证类中已经覆写了equals()方法。



## 4、LinkedList 子类

在List接口中还有一个比较常用的子类：LinkedList，这个类通过名称就可以发现其特点：基于链表的实现。那么首先观察一下LinkedList的定义：

```java
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, Serializable {}
```

LinkedList 类结构如下：

![image.png](Java 类集框架.assets/a4165a4ce6ab46c9ba54442f7a759ad3.png)

操作示例 1：使用LinkedList实现集合操作

```java
import java.util.LinkedList;
import java.util.List;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<String> all = new LinkedList<>();
        all.add("Hello");
        all.add("Hello"); // 重复数据
        all.add("Wolrd");
        all.add("XXXX");
        all.forEach(System.out::println);
    }
}

// 输出内容
Hello
Hello
Wolrd
XXXX
```



## 5、LinkedList 源码分析

如果现在只是观察程序的功能会发现和ArrayList使用是完全一样的，但是其内部实现机制是完全不同的，首先观察LinkedList构造方法里面并没有提供像ArrayList那样的初始化大小的方法，而只是提供了无参构造处理：“public LinkedList()”。随后观察add()方法的具体实现。

```java
/**
 * 在之前编写自定义链表时，是判断了传入数据是否为null，如果为null则不进行保存，
 * 但在LinkedList中并没有做这样的处理，而是所有的数据都可以保存,而后此方法调用了linkLast()方法（在最后一个节点后追加）
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}
```

```java
/**
 * 在LinkedList类中保存的数据都是利用Node节点进行的封装处理，同时为了提高程序执行性能，
 * 每一次都会保存上一个追加的节点（最后一个节点），就可以在增加数据的时候避免递归处理，在增加数据时要进行数据保存个数的追加
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
} 
```

通过上面的分析：可以发现LinkedList封装的就是一个链表实现。

**面试题：**请问ArrayList与LinkedList有什么区别？

1. ArrayList是数组实现的集合操作，而LinkedList是链表实现的集合操作
2. 在使用List集合中的get()方法根据索引获取数据时，ArrayList的时间复杂度为“O(1)”、而LinkedList时间复杂度为“O(n)”（n为集合的长度）
3. ArrayList在使用时默认的初始化对象数组的大小长度为10，如果空间不足则会采用2倍形式进行容量的扩充，如果保存大数据量的时候有可能会造成垃圾的产生以及性能的下降，但是这时候可以使用LinkedList类保存



## 6、Vector 子类

Vector是一个原始古老的程序类，这个类是在JDK1.0时提供的。到了JDK1.2时由于许多开发者已经习惯于使用Vector，并且许多系统类也是基于Vector实现的，考虑到其使用的广泛性，所以类集框架将其保留了下来，并让其多实现了一个List接口，观察Vector的定义结构：

```java
public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable {}
```

继承结构与ArrayList是相同的，所以来讲Vector这个类继承结构如下：

![image.png](Java 类集框架.assets/55ffc5fae2a241e28330fe62690fd68e.png)

操作示例 1：Vector的使用

```java
import java.util.List;
import java.util.Vector;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 为List父接口进行实例化
        List<String> all = new Vector<>();
        all.add("Hello");
        all.add("Hello"); // 重复数据
        all.add("World");
        all.add("XXXX");
        all.forEach(System.out::println);
    }
}

// 输出内容
Hello
Hello
World
XXXX
```

下面可以进一步的观察Vector类实现：

```java
public Vector() {
    this(10);
}
public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```

PS：Vector类如果使用的是无参构造方法，则一定会默认开辟一个10个长度的数组，而后其余的实现操作与ArrayList是相同的。通过源代码分析可以发现，Vector类中的操作方法采用的都是synchronized同步处理，而ArrayList并没有进行同步处理，所以Vector类中的方法在多线程访问的时候属于线程安全的，但是性能不如ArrayList高。



# Set 集合

本节介绍了Set接口继承关系以及特征，介绍了两个常用的子类：HashSet、TreeSet的使用。

## 1、Set 接口简介

Set集合最大的特点就是不允许保存重复元素,其也是 Collection子接口。

在JDK1.9以前Set集合与Collection集合的定义并无差别，Set继续使用了Collection接口中提供的方法进行操作，但是从JDK1.9后，Set集合也像List集合一样扩充了一些static方法，Set集合的定义如下：

```java
public interface Set<E> extends Collection<E> {}
```

需要注意的是Set集合并不像List集合那样扩充了许多的新方法，所以无法使用List集合中提供的get()方法，也就是说无法实现指定索引数据的获取，Set接口的继承关系如下。

![image.png](Java 类集框架.assets/1bc9d63abcdf44898607cd8118e527c6.png)

从JDK1.9后，Set集合也提供了像List集合中类似的of()的静态方法。下面就使用此方法进行Set集合特点的验证。

操作示例 1：验证Set集合特征

```java
import java.util.Set;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 进行Set集合数据的保存，并且设置有重复的内容
        Set<String> all = Set.of("Hello", "World", "XXXX", "Hello", "World");
        // 直接输出
        all.forEach(System.out::println);
    }
}
// 运行会报异常：因为出现重复元素
// 输出内容
Exception in thread "main" java.lang.IllegalArgumentException: duplicate element: Hello
	at java.base/java.util.ImmutableCollections$SetN.<init>(ImmutableCollections.java:587)
	at java.base/java.util.Set.of(Set.java:541)
	at JavaAPIDemo.main(JavaAPIDemo.java:5)
```

当使用of()这个新方法的时候，如果发现集合中存在重复元素则会直接抛出异常。这与传统的Set集合不保存重复元素的特点相一致，只不过自己抛出了异常而已。

Set集合的常规使用形式一定是依靠子类进行实例化的，所以Set接口之中有两个常用的子类：HashSet、TreeSet



## 2、HashSet 子类

HashSet是Set接口中使用最多的一个子类，其最大的特点就是保存的数据是无序的，而HashSet子类的继承关系如下：

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, Serializable {}
```

这种继承的形式和之前的ArrayList是非常相似的，那么现在来观察一下HashSet子类的继承结构：

![image.png](Java 类集框架.assets/41d6771c62ce4daeb5fafb7d1f0bd031.png)

操作示例 1：观察HashSet子类

```java
import java.util.HashSet;
import java.util.Set;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Set<String> all = new HashSet<>();
        all.add("XXXX");
        all.add("NiHao");
        all.add("Hello");
        all.add("Hello"); // 重复元素
        all.add("World");
        all.forEach(System.out::println);
    }
}

// 输出内容
NiHao
Hello
XXXX
World
```

通过执行结果就可以发现HashSet的操作特点：不允许保存重复元素（Set接口定义的），另外一个特点就是HashSet中保存的数据是无序的。



## 3、TreeSet 子类

Set接口的另外一个子接口就是TreeSet，与HashSet最大区别在于TreeSet集合里面保存的数据是有序的，首先来观察TreeSet类的定义：

```java
public class TreeSet<E> extends AbstractSet<E> implements NavigableSet<E>, Cloneable, Serializable {}
```

在这个子类中依然继承了AbstractSet父抽象类，同时又实现了一个NavigableSet父接口。TreeSet子类继承结果：

![image.png](Java 类集框架.assets/acee8502e0f44e76bbe821d5a88c464c.png)

操作示例 1：使用TreeSet子类

```java
import java.util.Set;
import java.util.TreeSet;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Set<String> all = new TreeSet<>();
        all.add("XXXX");
        all.add("NiHao") ;
        all.add("Hello");
        all.add("Hello"); // 重复元素
        all.add("World");
        all.forEach(System.out::println);
    }
}
// 输出结果（是有序的，默认是升序）
Hello
NiHao
World
XXXX
```

当利用TreeSet保存数据的时候，所有的数据将按照数据的升序进行自动排序处理。



## 4、TreeSet 排序说明

经过分析后发现，TreeSet子类中保存的数据是允许排序的，但是这个类必须要实现Comparable接口，只有实现了此接口才能够确认出对象的大小关系。

**提示**：TreeSet本质上是利用TreeMap子类实现的集合数据的存储，而TreeMap（树）就需要根据Comparable来确定对象的大小关系。

操作示例 1：使用自定义的类实现排序的处理操作（实现自定义类排序）

```java
import java.util.Set;
import java.util.TreeSet;

/**
 * 比较器
 */
class Person implements Comparable <Person> {
    private String name ;
    private int age ;
    public Person(String name,int age) {
        this.name = name ;
        this.age = age ;
    }
    @Override
    public String toString() {
        return "姓名：" + this.name + "、年龄：" + this.age ;
    }
    @Override
    public int compareTo(Person per) {
        if (this.age < per.age) {
            return -1 ;
        } else if (this.age > per.age) {
            return 1 ;
        } else {
            return this.name.compareTo(per.name) ;
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 为Set父接口进行实例化
        Set<Person> all = new TreeSet<>();
        all.add(new Person("张三",19)) ;
        all.add(new Person("李四",19)) ;	// 年龄相同，但是姓名不同
        all.add(new Person("王五",20)) ;	// 数据重复
        all.add(new Person("王五",20)) ;	// 数据重复
        all.add(new Person("小强",78)) ;
        all.forEach(System.out::println);
    }
}

// 输出内容
姓名：张三、年龄：19
姓名：李四、年龄：19
姓名：王五、年龄：20
姓名：小强、年龄：78
```

在使用自定义类对象进行比较处理的时候，一定要将该类中所有属性都依次进行大小关系的匹配，否则某一个或者几个属性相同的时候也会被认为是重复数据，所以TreeSet是利用了Comparable接口来确认重复数据的。

由于TreeSet在操作过程之中需要将类中的所有属性进行比对，这样的实现难度太高了，那么在实际的开发中应该首选HashSet子类进行存储。



## 5、关于重复元素说明

TreeSet类是利用了Comparable接口来实现了重复元素的判断，但是Set集合的整体特征就是不允许保存重复元素。但是HashSet判断重复元素的方式并不是利用Comparable接口完成的，它利用的是Object类中提供的方法实现的：

```java
public int hashCode();             // 对象编码
public boolean equals(Object obj); // 对象比较
```

在进行重复元素判断的时候首先利用hashCode()进行编码的匹配，如果该编码不存在则表示数据不存在，证明没有重复，如果该编码存在，则进一步进行对象比较处理，如果发现重复了，则此数据是不允许保存的。

在进行重复元素判断的时候首先利用hashCode()进行编码的匹配，如果该编码不存在，则表示数据不存在，证明没有重复，如果该编码存在，则进一步进行对象比较处理，如果发现重复了，则此数据是不允许保存的。如果使用的是IDE开发工具，则可以帮助开发者自动创建HashCode()与equals()方法。

操作示例 1：实现重复元素处理

```java
import java.util.HashSet;
import java.util.Set;

/**
 * 比较器
 */
class Person {
    private String name ;
    private int age ;
    public Person(String name,int age) {
        this.name = name ;
        this.age = age ;
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + age;
        result = prime * result + ((name == null) ? 0 : name.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null) return false;
        if (getClass() != obj.getClass())
            return false;
        Person other = (Person) obj;
        if (age != other.age)
            return false;
        if (name == null) {
            if (other.name != null)
                return false;
        } else if (!name.equals(other.name))
            return false;
        return true;
    }
    @Override
    public String toString() {
        return "姓名：" + this.name + "、年龄：" + this.age ;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        Set<Person> all = new HashSet<>();
        all.add(new Person("张三",19)) ;
        all.add(new Person("李四",19)) ;	// 年龄相同，但是姓名不同
        all.add(new Person("王五",20)) ;	// 数据重复
        all.add(new Person("王五",20)) ;	// 数据重复
        all.add(new Person("小强",78)) ;
        all.forEach(System.out::println);
    }
}

// 输出内容
姓名：小强、年龄：78
姓名：李四、年龄：19
姓名：王五、年龄：20
姓名：张三、年龄：19
```

在Java中真正的重复元素的判断处理利用的就是hashCode()和equals()两个方法共同作用完成的，而只有在排序要求的情况下（TreeSet）才会利用Comparable接口来实现。



# 集合输出

本节需要掌握集合输出的四种形式：Iterator 迭代输出、ListIterator 双向迭代输出、Enumeration 枚举输出、foreach 输出。

集合输出实际上从JDK1.8开始就在Iterable接口中提供了一个forEach()方法，但是这种方法输出并不是传统意义上集合输出形式，并且也很难在实际的开发之中出现，对于集合操作而言，一共有四种输出形式：

1. Iterator迭代输出（95%）
2. ListIterator双向迭代输出（0.1%）
3. Enumeration枚举输出（4.9%）
4. foreach输出（与Iterator相当）

## 1、Iterator 输出（核心）

通过Collection接口的继承关系可以发现，从JDK1.5开始其多继承了一个Iterable父接口，并且在这个接口里面定义有一个iterator()操作方法，通过此方法可以获取Iterator接口对象（在JDK1.5之前，这一方法直接定义在Collection接口之中）

在Iterator接口里面定义有如下的方法：

```java
public Iterator<T> iterator(); // 获取Iterator接口对象
public boolean hasNext();      // 判断是否有数据
public E next();               // 取出当前数据
default void remove();         // 删除当前数据
```

在之前使用的 java.util.Scanner 类就是 Iterator 接口的子类，所以此时（Iterator接口）类继承关系如下：

![image.png](Java 类集框架.assets/38ba0bb9fc224e6aa34c88ba2cd41334.png)

操作示例 1：使用Iterator输出

```java
import java.util.Iterator;
import java.util.Set;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Set<String> all = Set.of("Hello", "World", "XXXX");
        // 实例化Iterator接口对象
        Iterator<String> iter = all.iterator();
        while (iter.hasNext()) {
            String str = iter.next() ;
            System.out.println(str);
        }
    }
}

// 输出内容
XXXX
World
Hello
```

但是对于Iterator接口中的remove()方法的使用需要特别注意一下（如果不是必须不要使用）。实际上在Collection接口中定义有数据的删除操作方法，但是在进行迭代输出的过程中如果你使用了Collection中的remove()方法会导致迭代失败。

操作示例 2：采用Collection集合中remove()方法删除

```java
import java.util.Set;
import java.util.Iterator;
import java.util.HashSet;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Set<String> all = new HashSet<>();
        all.add("Hello");
        all.add("World");
        all.add("MLDN");
        // 实例化Iterator接口对象
        Iterator<String> iter = all.iterator();
        while (iter.hasNext()) {
            String str = iter.next();
            if ("World".equals(str)) {
                // Collection集合方法
                all.remove("World");
            }else {
                //  Hello   Exception in thread "main" java.util.ConcurrentModificationException
                System.out.println(str);
            }
        }
    }
}

// 输出内容（会报错）
Hello
Exception in thread "main" java.util.ConcurrentModificationException
	at java.base/java.util.HashMap$HashIterator.nextNode(HashMap.java:1493)
	at java.base/java.util.HashMap$KeyIterator.next(HashMap.java:1516)
	at JavaAPIDemo.main(JavaAPIDemo.java:13)
```

**PS：此时无法进行数据的删除处理操作，那么此时就只能够利用Iterator接口中的remove()方法删除。**

操作示例 3：使用Iterator接口删除方法

```java
import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Set<String> all = new HashSet<>();
        all.add("Hello");
        all.add("World");
        all.add("MLDN");
        // 实例化Iterator接口对象
        Iterator<String> iter = all.iterator();
        while (iter.hasNext()) {
            String str = iter.next();
            if ("World".equals(str)) {
                // 删除当前的数据
                iter.remove();
            } else {
                System.out.println(str);
            }
        }
        System.out.println("删除后集合： " + all);
    }

}

// 输出内容
Hello
MLDN
删除后集合： [Hello, MLDN]
```

此时程序执行后没有出现任何的错误，并且可以成功的删除原始集合中的数据。

**面试题：请解释Collection.remove()与Iterator.remove()的区别？**

1. 在进行迭代输出的时候，如果使用了Collection.remove()则会造成并发更新的异常，导致程序删除出错
2. 而此时只能够利用Iterator接口中remove()方法实现正常的删除处理





## 2、ListIterator 输出

使用Iterator进行的迭代输出操作有一个特点：只允许由前向后输出，而如果现在需要进行双向迭代处理，那么就必须依靠Iterator的子接口：ListIterator接口来实现了。需要注意的是，如果想要获取ListIterator接口对象，Collection中并没有定义相关的处理方法，但是List子接口有，也就是说这个输出的接口是专门为List集合准备的。ListIterator 继承 Iterator：

```java
public interface ListIterator<E> extends Iterator<E> {}
```

ListIterator接口类继承结构：

![image.png](Java 类集框架.assets/ffb1cc8b24594926abca2f33d2052646.png)

ListIterator接口中定义有如下方法（如下是ListIterator比Iterator多的方法，因为ListIterator继承Iterator所以也包含了Iterator内的方法）：

```java
public boolean hasPrevious(); // 判断是否有前一个元素（迭代器当前位置，反向遍历集合是否含有元素）
public E previous();          // 获取当前元素（迭代器当前位置，反向遍历集合，下一个元素）
public int previousIndex();   // 迭代器当前位置，反向遍历集合，返回下一个元素的下标
public int nextIndex();       // 迭代器当前位置，返回下一个元素的下标
public void add(E e);         // 将指定的元素插入列表，插入位置为迭代器当前位置之前
public void set(E e);         // 迭代器返回的最后一个元素替换参数e
```

操作示例 1：实现双向迭代

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<String> all = new ArrayList<>();
        all.add("Hello");
        all.add("World");
        all.add("XXXX");
        ListIterator<String> iter = all.listIterator();
        System.out.print("由前向后输出：");
        while (iter.hasNext()) {
            System.out.print(iter.next() + "、");
        }
        System.out.print("\n由后向前输出：");
        while (iter.hasPrevious()) {
            System.out.print(iter.previous() + "、");
        }
    }
}

// 输出内容
由前向后输出：Hello、World、XXXX、
由后向前输出：XXXX、World、Hello、
```

PS：如果想实现由后向前的遍历，那么首先要实现的是由前向后实现遍历处理。

**面试题：Iterator 和 ListIterator 有什么区别？**https://blog.csdn.net/meism5/article/details/89921602

1. ListIterator 继承 Iterator
2. ListIterator 比 Iterator多6个方法：add、set、hasPrevious、previous、previousIndex、nextIndex
3. 使用范围不同，Iterator可以迭代所有集合；ListIterator 只能用于List及其子类
4. ListIterator 有 add 方法，可以向 List 中添加对象；Iterator 不能
5. ListIterator 有 hasPrevious() 和 previous() 方法，可以实现逆向遍历；Iterator不可以
6. ListIterator 有 nextIndex() 和previousIndex() 方法，可定位当前索引的位置；Iterator不可以
7. ListIterator 有 set()方法，可以实现对 List 的修改；Iterator 仅能遍历，不能修改



## 3、Enumeration 输出

Enumeration是在JDK1.0的时候就使用的输出接口，这个输出接口主要是为了Vector类提供服务的，一直到后续的JDK的发展，Enumeration依然只为Vector一个类服务，所以要想获取Enumeration接口对象，那么必须依靠Vector类提供的方法：

```java
public Enumeration<E> elements(); // 获取Enumeration
public boolean hasMoreElements(); // 判断是否有下一个元素
public E nextElement();           // 获取当前元素
```

Enumeration接口类继承结构：

![image.png](Java 类集框架.assets/8fc865f209154ea6bbfc81dc8c148031.png)

操作示例 1：使用Enumeration输出

```java
import java.util.Enumeration;
import java.util.Vector;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Vector<String> all = new Vector<>();
        all.add("Hello") ;
        all.add("World") ;
        all.add("XXXX") ;
        Enumeration<String> enu = all.elements() ;
        while (enu.hasMoreElements()) {
            String str = enu.nextElement() ;
            System.out.print(str + "、");
        }
    } 
}

// 输出内容
Hello、World、XXXX、
```

由于该接口出现的时间比较长了，所以在一些比较早的开发过程中，也有部分的方法只支持Enumeration输出操作，但随着类方法的不断完善，大部分的操作都能直接利用Iterator实现了。



## 4、Foreach 输出

除了使用迭代接口实现输出之外，从JDK1.5开始加强型for循环也可以实现集合的输出了。这种输出的形式与数组的输出操作形式类似。

操作示例：使用foreach输出

```java
import java.util.ArrayList;
import java.util.List;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<String> all = new ArrayList<>();
        all.add("Hello");
        all.add("World");
        all.add("XXXX");
        for (String str : all) {
            System.out.print(str + "、"); 
        }
    }
}

// 输出内容
Hello、World、XXXX、
```

这种输出最初出现时很多人并不建议使用，因为标准的集合操作还是以Iterator为主，但是毕竟JDK1.5都已经推出十多年了，很多语法也开始被大部分人所习惯。



# Map 集合

本节介绍了Map接口以及它的子类HashMap、LinkedHashMap、HashTable、TreeMap，需要掌握他们的基本用法。

## 1、Map 接口简介

在之前已经学习了Collection接口以及其对应的子接口，可以发现在Collection接口之中所保存的数据全部都只是单个对象，而在数据结构中除了可以进行单个对象的保存外，也可以进行二元偶对象的保存（key=value）的形式来存储，而存储二元偶对象的核心意义在于需要通过key获取对应的value。在开发中，Collection集合保存数据的目的是为了输出，Map集合保存数据的目的是为了进行key的查找。

Map接口是进行二元偶对象保存的最大父接口。该接口定义如下：

```java
public interface Map<K,V> {}
```

该接口为一个独立的父接口，并且在进行接口对象实例化的时候需要设置Key与Value的类型，也就是在整体操作的时候需要保存两个内容，在Map接口中定义有许多操作方法，但是需要记住以下的核心操作方法：

```java
public V put(K key,V vvalue)              // 向集合之中保存数据
public V get(Object key)                  // 根据key查询数据
public V remove(Object key)               // 根据key删除掉指定的数据
boolean remove(Object key, Object value)  // 当指定的 key 与 value 在映射集中存在, 且为绑定的键值对关系时, 才移除
boolean containsKey(Object key)           // 查询指定key是否存在
boolean containsValue(Object value)       // 查找指定value是否存在
public Set<Map.Entry<K,V>> entrySet()     // 将Map集合转为Set
public Set<K> keySet()                    // 将Map集合中的key转为Set集合
Collection<V> values()                    // 返回Map中所有的value
```

操作示例 1：观察Map集合的特点。从JDK1.9之后Map接口里面也扩充了一些静态方法供用户使用

```java
import java.util.Map;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // {one=1,two=2}
        Map<String, Integer> map1 = Map.of("one", 1, "two", 2);
        System.out.println(map1);

        // Exception in thread "main" java.lang.IllegalArgumentException: duplicate key: one
        Map<String, Integer> map2 = Map.of("one", 1, "two", 2, "one", 101);
        //System.out.println(map2);

        // Exception in thread "main" java.lang.NullPointerException
        Map<String, Integer> map3 = Map.of("one", 1, "two", 2, null, 0);
        System.out.println(map3);
    }
}

// 输出内容
{one=1, two=2}
Exception in thread "main" java.lang.IllegalArgumentException: duplicate key: one
	at java.base/java.util.ImmutableCollections$MapN.<init>(ImmutableCollections.java:805)
	at java.base/java.util.Map.of(Map.java:1349)
	at JavaAPIDemo.main(JavaAPIDemo.java:9)
```

在Map集合之中数据的保存就是按照“key=value”的形式存储的，并且使用of()方法操作时里面的数据是不允许重复，如果重复则会出现“IllegalArgumentException”异常，如果设置的内容为null，则会出现“NullPointerException”异常。

对于现在见到的of()方法严格意义上来说并不是Map集合的标准用法，因为正常的开发中需要通过Map集合的子类来进行接口对象的实例化，而常用的子类：HashMap、HashTable、TreeMap、LinkedHashMap。



## 2、HashMap 子类

HashMap是Map接口中最为常见的一个子类,该类的主要特点是无序存储，通过Java文档首先来观察一下HashMap子类的定义：

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {}
```

该类的定义继承形式符合之前的集合定义形式，依然提供有抽象类并且依然需要重复实现Map接口。HashMap子类结构：

![image.png](Java 类集框架.assets/debd838773f446878880a752d7768271.png)

操作示例 1：HashMap集合的使用

```java
import java.util.HashMap;
import java.util.Map;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>() ;
        map.put("one", 1) ;
        map.put("two", 2) ;
        map.put("one", 101) ;	            // key重复
        map.put(null, 0) ;		            // key为null
        map.put("zero", null) ;             // value为null
        System.out.println(map.get("one"));	// key存在
        System.out.println(map.get(null));	// key存在
        System.out.println(map.get("ten"));	// key不存在
    }
}

// 输出内容
101
0
null
```

以上的操作形式为Map集合使用的最标准的处理形式，通过代码可以发现，通过HashMap实例化的Map接口可以针对key或者value保存null的数据，同时也可以发现即便保存数据的key重复，那么也不会出现错误，而是出现内容的替换。

但是对于Map接口中提供的put()方法本身是提供有返回值的，那么这个返回值指的是在重复key的情况下返回旧的value。

操作示例 2：观察put()方法。

```java
import java.util.HashMap;
import java.util.Map;
public class JavaAPIDemo {
	public static void main(String[] args) {
		Map<String, Integer> map = new HashMap<>() ;
		System.out.println(map.put("one", 1));	// key不重复，返回null
		System.out.println(map.put("one", 101));// key重复，返回旧数据
	}
}

// 输出内容
null
1
```

可以发现：在设置了相同key的内容的时候，put()方法会返回原始的数据内容。

HashMap 源码分析：

清楚了HashMap基本功能后就研究一下HashMap中的源代码。HashMap之中肯定需要存储大量的数据，那么对于数据的存储看看HashMap是如果操作：

```java
// 当使用无参构造的时候会出现有一个loadFactor属性，并且该属性的默认内容为0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

```java
/**
 * 在使用put()方法进行数据保存的时候会调用一个putVal()方法,同时会将key进行hash处理(生成一个hash码),
 * 而对于putVal()方法里面会发现依然会提供有一个Node节点类进行数据的保存,
 * 而在使用putVal()方法操作的过程之中会调用有个resize()方法可以进行容量的扩充。
 */
public V put(K key, V value) {
	return putVal(hash(key), key, value, false, true);
}
```

**面试题 1：在进行HashMap的put()操作时，如何实现容量扩充？**

1. 在HashMap类里面提供有一个“DEFAULT_INITIAL_CAPACITY”常量，作为初始化的容量配置，而后这个常量的默认大小为16个元素，也就是说默认可以保存的最大内容是16
2. 当保存的内容的容量超过了与个阈值（DEFAULT_LOAD_FACTOR = 0.75f），相当于 “容量 * 阈值 = 12 ” 保存12个元素的时候就会进行容量的扩充
3. 在进行扩充的时候HashMap采用的是成倍的扩充模式，即：每一次都扩充2倍的容量

**面试题 2：请解释HashMap的工作原理（JDK1.8之后开始的）**

1. 在HashMap之中进行数据存在的依然是利用Node类完成的，那么这种情况下就证明可以使用的数据结构只有两种：链表（时间复杂度“O（n）”）、二叉树（时间复杂度“O（logn）”）
2. 从JDK1.8开始，HashMap的实现出现了改变，因为其要适应于大数据时代的海量数据问题，所以对其存储发生了变化，并且在HashMap类的内部提供有一个常量：“static final int TREEIFY_THRESHOLD = 8;”，在使用HashMap进行数据保存时，如果保存的数据没有超过阈值8（TREEIFY_THRESHOLD），那么会按照链表的形式进行存储，如果超过了阈值，则会将链表转为红黑树以实现树的平衡，并且利用左旋与右旋保证数据的查询性能。



## 3、LinkedHashMap 子类

HashMap虽然是Map集合中最为常用的子类，但是其本身保存的数据都是无序的（有序与否对Map没有影响），如果现在希望Map集合中的保存的数据的顺序为其增加顺序，则就可以更换子类为LinkedHashMap（基于链表实现的），观察LinkedHashMap类的定义形式：

```java
public class LinkedHashMap<K, V> extends HashMap<K, V> implements Map<K, V> {}
```

既然是链表保存，所以一般在使用LinkedHashMap类时数据量不要特别大，因为会造成时间复杂度攀升，通过继承的结构可以发现LinkedHashMap是HashMap的子类，LinkedHashMap类继承关系如下：

![image.png](Java 类集框架.assets/ca014847b22d499da92832a6917e3f3f.png)

操作示例：使用LinkedHashMap

```java
import java.util.LinkedHashMap;
import java.util.Map;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new LinkedHashMap<>() ;
        map.put("one", 1) ;
        map.put("two", 2) ;
        map.put("one", 101) ;	// key重复
        map.put(null, 0) ;		// key为null
        map.put("zero", null) ; // value为null
        System.out.println(map);	// key不存在
    }
}

// 输出内容
{one=101, two=2, null=0, zero=null}
```



## 4、HashTable 子类

HashTable类是从JDK1.0时提供的，与Vector、Enumeration属于最早的一批动态数组的实现类，后来为了将其继续保留下来，所以让其多实现了一个Map接口，HashTable类的定义如下：

```java
public class Hashtable<K, V> extends Dictionary<K, V> implements Map<K, V>, Cloneable, Serializable {}
```

HashTable的继承结构如下：

![image.png](Java 类集框架.assets/69df4f106dc64f68b6dd9b25b5fca5e4.png)

操作示例：观察HashTable子类的使用

```java
import java.util.Hashtable;
import java.util.Map;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new Hashtable<>();
        map.put("one", 1);
        map.put("two", 2);
        map.put("one", 101);
        // map.put(null, 0);       // 不能为空
        // map.put("zero",null);   //不能为空，Exception in thread "main" java.lang.NullPointerException
        System.out.println(map);   // {two=2, one=101}
    }
}

// 输出内容
{two=2, one=101}
```

通过观察可以发现在HashTable中进行数据存储时设置的key或value都不允许为null，否则会出现NullPointerException异常。

**面试题：**请解释HashMap与HashTable的区别？

1. HashMap中的方法都属于异步操作（非线程安全），HashMap允许保存有null的数据
2. HashTable中的方法都属于同步方法（线程安全），HashTable不允许保存null，否则会出现NullPointerException异常



## 5、TreeMap 子类

TreeMap属于有序的Map集合类型，它可以按照key进行排序，所以来讲在使用这个子类的时候一定要配合Comparable接口共同使用，TreeMap子类的定义如下：

```java
public class TreeMap<K, V> extends AbstractMap<K, V> implements NavigableMap<K, V>, Cloneable, Serializable {}
```

TreeMap继承结构：

![20211110110856](Java 类集框架.assets/20211110110856.png)

TreeMap 默认排序规则：按照key的字典顺序来排序（升序），也可以自定义排序规则：要实现Comparator接口。操作示例：使用TreeMap实现排序

```java
import java.util.Comparator;
import java.util.Map;
import java.util.TreeMap;
public class JavaAPIDemo {
    public static void main(String[] args) {
        System.out.println("---------------- 默认 排序结果（根据key排序）-----------------");
        Map<String, Integer> map1 = new TreeMap<>();
        map1.put("C", 3);
        map1.put("A", 1);
        map1.put("B", 2);
        System.out.println(map1);

        System.out.println("---------------- 自定义 排序结果（根据key排序）-----------------");
        // TreeMap<Integer, Integer> map2 = new TreeMap<>((a, b) -> b - a); // lambda形式
        TreeMap<Integer, Integer> map2 = new TreeMap<>(new Comparator<Integer>() {
            /*
             * int compare(Object o1, Object o2) 返回一个基本类型的整型，
             * 返回负数表示：o1 小于o2，
             * 返回0 表示：o1和o2相等，
             * 返回正数表示：o1大于o2。
             */
            @Override
            public int compare(Integer a, Integer b) {
                return b - a; // key倒序排序
            }
        });
        map2.put(3, 3);
        map2.put(1, 1);
        map2.put(2, 2);

        System.out.println(map2);
    }
}

// 输出内容
{A=1, B=2, C=3}
```

在使用TreeMap的时候可以根据key实现数据的排序处理，但是对于key是不允许设置为null的（因为需要依靠Comparable接口对象中的compareTo方法排序，如果传入的对象为null，那么调用方法返回的就是NullPointerException），而value允许为null。

---

> TreeMap按照value进行排序

TreeMap底层是根据红黑树的数据结构构建的，默认是根据key的自然排序来组织（比如integer的大小，String的字典排序）。所以，TreeMap只能根据key来排序，是不能根据value来排序的（否则key来排序根本就不能形成TreeMap）。

今天有个需求，就是要根据treeMap中的value排序。所以网上看了一下，大致的思路是把TreeMap的EntrySet转换成list，然后使用Collections.sor排序。代码：

```java
import java.util.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Map map = new TreeMap<>();{{
            map.put("d", "dddd");
            map.put("a", "aaaa");
            map.put("b", "bbbb");
            map.put("c", "cccc");
        }}
        List<Map.Entry<String, String>> list = new ArrayList<>(map.entrySet());
        System.out.println("升序排序")
        Collections.sort(list, Comparator.comparing(Map.Entry::getValue)); // 升序排序
        list.forEach(e -> System.out.println(e.getKey()+":"+e.getValue()));
        System.out.println("降序排序")
        Collections.reverse(list);// 降序排序
        list.forEach(e -> System.out.println(e.getKey()+":"+e.getValue()));
    }
}

// 输出结果
升序排序
a:aaaa
b:bbbb
c:cccc
d:dddd
降序排序
d:dddd
c:cccc
b:bbbb
a:aaaa
```



## 6、Map.Entry 接口

虽然已经清楚了整个的Map集合的基本操作形式，但是依然需要有一个核心的问题要解决，Map集合里面是如何进行数据存储的？对于List而言（LinkedList子类）依靠的是链表的形式实现数据存储，那么在进行数据存储的时候一定要将数据保存在一个Node节点之中，虽然在HashMap里面也可以见到Node类型定义，通过源码定义可以发现，HashMap类中的Node内部类本身实现了Map.Entry接口：

```java
static class Node<K,V> implements Map.Entry<K,V> {}
```

所以可以得出结论：所有的key和value数据都被封装在Map.Entry接口之中，接口定义如下：

```java
public static interface Map.Entry<K,V> {}
```

并且这个接口内提供有几个重要方法：

```java
public interface Entry<K, V> {
    K getKey();	        // 获得key、
    V getValue();       // 获取value
    V setValue(V var1); //设置value
}
```

![20211110133941](Java 类集框架.assets/20211110133941.png)

在JDK1.9以前的开发版本之中，使用者基本上都不会去考虑创建Map.Entry的对象，实际上在正常的开发过程之中使用者也不需要关心Map.Entry对象创建，可是从JDK1.9之后，Map接口里面追加有一个新的方法：

```java
public static <K, V> Map.Entry<K, V> entry(K k, V v) // 创建Map.Entry对象
```

操作示例：创建Map.Entry对象

```java
import java.util.Map;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Map.Entry<String, Integer> entry = Map.entry("one", 1);
        System.out.println("获取Key：" + entry.getKey());
        System.out.println("获取Value：" + entry.getValue());
        System.out.println(entry.getClass().getName()); // 观察使用的子类
    }
}
```

通过分析可以发现在整个的Map集合里面，Map.Entry的主要作用就是作为一个Key和Value的包装类型使用，而大部分况下在进行数据存储的时候都会将key和 value包装为一个Map.Entry对象进行使用。



## 7、Map 集合的输出

对于集合的输出而言，最标准的做法就是利用Iterator接口来完成，但是需要明确一点的是在Map集合里面并没有一个方法可以直接返回Iterator接口对象，所以这种情况下就必须分析不直接提供Iterator接口实例化的方法的原因，下面对Collection与Map集合的存储结构进行一个比较：

![20211110135543](Java 类集框架.assets/20211110135543.png)

发现在Map集合里面保存的实际上是一组Map.Entry接口对象（里面包装的是Key与Value），所以整个来讲Map依然买现的是单值的保存，这样在Map集合里面提供有一个方法：将全部的Map集合转为Set集：`public Set<Map.Enty<K, V>> entrySet()`

```java
public Set<Map.Enty<K, V>> entrySet(); // 将全部的Map集合转为Set集
```

![20211110135816](Java 类集框架.assets/20211110135816.png)

经过分析可以发现如果要想使用Iterator实现Map集合的输出则必须按照如下步骤处理：

1. 利用Map接口中提供的entrySet()方法将Map集合转为Set集合
2. 利用Set接口中的iterator()方法将Set集合转为Iterator接口实例
3. 利用Iterator进行迭代输出获取每一组的Map.Entry对象，随后通过getKey()与getValue()方法获取数据

操作示例 1：利用Iterator输出Map集合

```java
import java.util.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("one", 1);
        map.put("two", 2);
        Set<Map.Entry<String,Integer>> set = map.entrySet(); // 将Map集合变为Set集合
        Iterator<Map.Entry<String,Integer>> iter = set.iterator();
        while (iter.hasNext()) {
            Map.Entry<String, Integer> me = iter.next() ;
            System.out.println(me.getKey() + " = " + me.getValue());
        }
    }
}

// 输出内容
one = 1
two = 2
```

操作示例 2：使用foreach输出

```java
import java.util.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("one", 1);
        map.put("two", 2);
        // 将Map集合变为Set集合
        Set<Map.Entry<String,Integer>> set = map.entrySet() ;
        for (Map.Entry<String, Integer> entry : set) {
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
    }
}

// 输出内容
one = 1
two = 2
```



## 8、关于 Key 的定义

在使用Map集合的时候可以发现对于Key和Value的类型实际上都可以由使用者任意决定，那么也就意味着现在依然可以使用自定义的类来进行Key类型的设置。对于自定义Key类型所在的类中一定要覆写hashCode()与equals()方法，否则无法查找到。

```java
/**
 * 在进行数据保存的时候发现会自动使用传入的key的数据生成个hash码,也就是说存储的时候是有这个Hash数值。
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

/**
 * 在根据key获取数据的时候依然要将传入的key通过hash()方法来获取其对应的hash码,
 * 那么也就证明,查询的过程之中首先要利用 hashCode()来进行数据查询,
 * 当使用 getNode()方法查询的时候还需要使用到equals()方法
 */
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

操作示例：使用自定义类作为Key类型

```java
import java.util.HashMap;
import java.util.Map;
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + age;
        result = prime * result + ((name == null) ? 0 : name.hashCode());
        return result;
    }
    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        Person other = (Person) obj;
        if (age != other.age)
            return false;
        if (name == null) {
            if (other.name != null)
                return false;
        } else if (!name.equals(other.name))
            return false;
        return true;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        Map<Person, String> map = new HashMap<>();
        map.put(new Person("小强", 78), "林弱"); // 使用自定义类作为Key
        System.out.println(map.get(new Person("小强", 78))); // 通过key找到value
    }
}

// 输出内容
林弱
```

虽然允许你使用自定义的类作为Key的类型，但是也需要注意一点，在实际的开发之中对于Map集合的Key常用的类型就是：String、Long、 Integer，尽量使用系统类。

**面试题：**如果在进行HashMap进行数据操作的时候出现了Hash冲突（Hash码相同），HashMap是如何解决的?

当出现了Hash冲突之后为了保证程序的正常执行，会在冲突的位置上将所有Hash冲突的内容转为链表保存。

![20211110140658](Java 类集框架.assets/20211110140658.png)



## 9、Java 8 Map 方法

### 1、getOrDefault()：get 不到键则返回默认值

```java
default V getOrDefault(Object key, V defaultValue)
```

- **key** ：对应Map的key
- **defaultValue** ：传入的默认值

返回值：

- 如果Map中这个 key 存在，则返回这个 key 对应的 value
- 如果Map中这个 key 不存在，则把传入的 defaultValue 作为返回值

```java
Map<String, String> map = new HashMap<>();
map.put("key1", "value1");
String value1 = map.getOrDefault("key1", "defaultValue");
System.out.println(value1); // 输出 value1
// 当不存在key2时，返回指定默认值
String value2 = map.getOrDefault("key2", "defaultValue");
System.out.println(value2); // 输出 defaultValue
```



### 2、putIfAbsent()：键不存在时执行 put 方法

```java
default V putIfAbsent(K key, V value)
```

- **key** ：对应Map的key
- **value** ：将要放入map的值

返回值：

- 如果Map中 key 对应的 value 存在，则返回value
- 如果Map中 key 对应的 value 为 null，返回null ，并且添加从key到传入的value的映射

```java
Map<String, String> map = new HashMap<>();
map.put("key1", "value1");
map.putIfAbsent("key1", "newValue");
String value = map.get("key1");
System.out.println(value); // 输出 value1
```



### 3、replace：当 key 存在时才执行 put 方法

```java
default V replace(K key, V value)
```

- **key** ：对应Map的key
- **value** ：将要放入map的值

返回值：

- 如果Map中 key 对应的 value 存在，才会执行put方法。
- 如果Map中 key 对应的 value 为 null，则返回null。

```java
Map<String, String> map = new HashMap<>();
map.put("key1", "value1");
map.replace("key1", "newValue");
System.out.println(map); // {"key1":"newValue"}
System.out.println(map.replace("key2", "value2")); // null
```

***

```java
default boolean replace(K key, V oldValue, V newValue)
```

- key：对应Map的key
- oldValue：旧的 value 值
- newValue：新的 value 值

返回值：

- 如果 oldValue 不存在，对 key 对应对 value 执行替换，返回 boolean。如果存在则替换，并返回 boolean。
- 如果 key 不存在，则不做任何操作，返回 false

```java
Map<String, String> map = new HashMap<>();
map.put("key1", "oldValue");
boolean replace1 = map.replace("key1", "xxxValue", "newValue");
System.out.println(replace1); // false
System.out.println(map);      // {key1=oldValue}
boolean replace2  = map.replace("key1", "oldValue", "newValue");
System.out.println(replace2); // true
System.out.println(map);      // {key1=newValue}
// key 不存在
boolean replace3 = map.replace("key2", "oldValue2", "newValue2");
System.out.println(replace3); // false
```



### 4、replaceAll：使用函数重写 value

```java
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function)
```

- function ：函数式接口实例，生成映射对函数

返回值：无，只会对映射集中的值做替换处理。

replaceAll() 方法将 映射集中的所有映射Value替换成给定的函数所执行的结果

```java
// 假设我们定义一个订单号对应的金额的映射
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("S2021123100001", 200);
hashMap.put("S2021123100002", 400);
hashMap.put("S2021123100003", 600);

// replaceAll之前: {S2021123100001=200, S2021123100002=400, S2021123100003=600}
System.out.println("replaceAll之前: " + hashMap);
hashMap.replaceAll((k, v) -> v == null ? -1 : v * 100);

// replaceAll之后: {S2021123100001=20000, S2021123100002=40000, S2021123100003=60000}
System.out.println("replaceAll之后: " + hashMap);
```



### 5、compute：根据 value 来重写计算

```java
default V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```

- **key** ：对应Map的key
- **remappingFunction** ：重新映射函数，用于重新计算值

返回值：

- 如果 key 对应的 value 不存在，则返回该 null。
- 如果 key 对应的 value 存在，则返回通过 remappingFunction 重新计算值，如果计算后的值newValue不为null ，则把计算后的值newValue返回，并且把计算后的值newValue更新到映射中；如果计算后的值newValue为null ，则返回null，并且把当前key 对应的映射从集合中删除。

```java
// 假设我们定义一个订单号对应的金额的映射
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("S2021123100001", 200);
hashMap.put("S2021123100002", 400);
hashMap.put("S2021123100003", 600);

System.out.println("----------------模拟操作不存在的订单----------------");
// 模拟操作不存在的订单
Integer noneExistValue = hashMap.compute("S2021123100004", (key, value) -> {
    if (value == null) {
        return value;
    }
    return value * 80 / 100;
});
System.out.println("操作一个不存在的key结果：" + noneExistValue);
System.out.println("最终hashMap的值：" + hashMap);

System.out.println("----------------模拟订单存在，总价打个8折----------------");
// 模拟订单存在，总价打个8折
Integer existValue = hashMap.compute("S2021123100001", (key, value) -> {
    if (value == null) {
        return value;
    }
    return value * 80 / 100;
});
System.out.println("操作一个存在的key,计算结果不为空的结果：" + existValue);
System.out.println("最终hashMap的值：" + hashMap);

System.out.println("----------------模拟操作返回null----------------");
// 模拟操作返回null
Integer nullValue = hashMap.compute("S2021123100003", (key, value) -> null);
System.out.println("操作一个不存在的key，计算结果为空的结果：" + nullValue);
System.out.println("最终hashMap的值：" + hashMap);
```

```java
----------------模拟操作不存在的订单----------------
操作一个不存在的key结果：null
最终hashMap的值：{S2021123100001=200, S2021123100002=400, S2021123100003=600}
----------------模拟订单存在，总价打个8折----------------
操作一个存在的key,计算结果不为空的结果：160
最终hashMap的值：{S2021123100001=160, S2021123100002=400, S2021123100003=600}
----------------模拟操作返回null----------------
操作一个不存在的key，计算结果为空的结果：null
最终hashMap的值：{S2021123100001=160, S2021123100002=400}
```



### 6、computeIfAbsent：当 value 不存在时重新计算

```java
default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) 
```

- **key** ：对应Map的key
- **mappingFunction** ：重新映射函数，用于重新计算值，得到newValue

返回值：

- 如果Map中 key 对应的 value 存在，返回 value
- 如果Map中 key 对应的 value 不存在，则通过remappingFunction 重新计算值，如果新计算的值newValue不为null，并保存从key 到计算后的值newValue的一个映射；如果新计算的值newValue为null，则不会加入映射

有这样一个常见的应用场景，当一批用户按年龄分组之后，新用户怎么加入到分组中？通常我们这样做的：

```java
// 假设我们定义水果英文和中文的映射
Map<String, String> map = new HashMap<>();
map.put("Apple", "苹果");
map.put("Orange", "橘子");
map.put("Banana", "香蕉");

System.out.println("----------------模拟key存在----------------");
String orange = map.computeIfAbsent("Orange", key -> "西瓜");
System.out.println("操作存在的key的结果：" + orange);
System.out.println("最终map的值：" + map);

System.out.println("----------------模拟key不存在，计算后的值不为null（增加映射）----------------");
String watermelon = map.computeIfAbsent("watermelon", key -> "西瓜");
System.out.println("操作不存在的key,计算后的值不为null的结果：" + watermelon);
System.out.println("最终map的值：" + map);

System.out.println("----------------模拟key不存在，计算后的值为null----------------");
String strawberry = map.computeIfAbsent("strawberry", key -> null);
System.out.println("操作不存在的key,计算后的值为null的结果：" + strawberry);
System.out.println("最终map的值：" + map);
```

```java
----------------模拟key存在----------------
操作存在的key的结果：橘子
最终map的值：{Apple=苹果, Orange=橘子, Banana=香蕉}
----------------模拟key不存在，计算后的值不为null（增加映射）----------------
操作不存在的key,计算后的值不为null的结果：西瓜
最终map的值：{Apple=苹果, watermelon=西瓜, Orange=橘子, Banana=香蕉}
----------------模拟key不存在，计算后的值为null----------------
操作不存在的key,计算后的值为null的结果：null
最终map的值：{Apple=苹果, watermelon=西瓜, Orange=橘子, Banana=香蕉}
```



### 7、computeIfPresent：当 value 存在时重新计算

```java
default V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```

- **key** ：对应Map的key
- **mappingFunction** ：重新映射函数，用于重新计算值，得到newValue

返回值：

- 如果Map中 key 对应的 value 不存在，则返回该 null
- 如果Map中 key 对应的 value 存在，则通过 remappingFunction 重新计算值，如果新计算的值不为null，则更新从key 到计算后的值newValue的一个映射；如果新计算的值为null，则删除key 到value的映射

```java
// 假设我们定义一个订单号对应的金额的映射
Map<String, Integer> map = new HashMap<>();
map.put("S2021123100001", 200);
map.put("S2021123100002", 400);
map.put("S2021123100003", 600);

System.out.println("----------------模拟key存在----------------");
// 假设抽取到的订单，就给总价打个8折
Integer existValue = map.computeIfPresent("S2021123100001", (key, value) -> value * 80 / 100);
System.out.println("操作一个存在的key结果：" + existValue);
System.out.println("最终map的值：" + map);

System.out.println("----------------模拟key不存在，，计算结果不为null----------------");
Integer noneExistValue = map.computeIfPresent("S2021123100004", (key, value) -> value * 80 / 100);
System.out.println("操作一个不存在的key,计算结果不为null结果：" + noneExistValue);
System.out.println("最终map的值：" + map);

System.out.println("----------------模拟key存在，计算结果为null（删除映射）----------------");
Integer existKeyWithNullValue = map.computeIfPresent("S2021123100002", (key, value) -> null);
System.out.println("操作一个存在的key,计算结果为null结果：" + existKeyWithNullValue);
System.out.println("最终map的值：" + map);
```

```java
----------------模拟key存在----------------
操作一个存在的key结果：160
最终map的值：{S2021123100001=160, S2021123100002=400, S2021123100003=600}
----------------模拟key不存在，，计算结果不为null----------------
操作一个不存在的key,计算结果不为null结果：null
最终map的值：{S2021123100001=160, S2021123100002=400, S2021123100003=600}
----------------模拟key存在，计算结果为null（删除映射）----------------
操作一个存在的key,计算结果为null结果：null
最终map的值：{S2021123100001=160, S2021123100003=600}
```



### 8、merge：合并新旧两个值

```java
default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction)
```

- **key** ：对应Map的key
- **value** ：使用者传入的值
- **mappingFunction** ：重新映射函数，用于重新计算值，得到newValue

返回值：

- 如果Map中不存在指定的key时，便将传入的value设置为key的值，
- 如果Map中key存在value时，执行一个方法该方法接收key的旧值oldValue和传入的value，执行自定义的方法

```java
Employee[] employeeArray = {
    new Employee("BAT001", "胡昊天", "销售部", 28, 3500),
    new Employee("BAT002", "王大锤", "销售部", 27, 3000),
    new Employee("BAT003", "唐二鹏", "研发部", 32, 9900),
    new Employee("BAT004", "王一林", "研发部", 30, 9000),
    new Employee("BAT005", "梁南生", "研发部", 27, 8000),
    new Employee("BAT006", "包三雅", "财务部", 25, 6000),
    new Employee("BAT007", "罗考聪", "测试部", 35, 7400),
    new Employee("BAT008", "徐浪生", "测试部", 24, 7000),
    new Employee("BAT009", "胡俊伟", "研发部", 24, 4500),
    new Employee("BAT010", "胡健儿", "人事部", 23, 4000),
    new Employee("BAT011", "陶建伟", "运维部", 25, 8000),
    new Employee("BAT012", "张萌萌", "行政部", 20, 3500)
};
List<Employee> list = Arrays.asList(employeeArray);
Map<String, Double> hashMap = new HashMap<>();
System.out.println("-------------------java8 写法（易理解）-------------------");
list.stream().forEach(studentScore -> 
    hashMap.merge(studentScore.getDepartment(), studentScore.getSalary(), (oldValue, newValue) -> oldValue + newValue));
System.out.println("最终hashMap的值：" + hashMap.toString());
System.out.println("-------------------java8 精简写法-------------------");
Map<String, Double> map = new HashMap<>();
list.stream().forEach(studentScore -> map.merge(studentScore.getDepartment(), studentScore.getSalary(), Double::sum));
System.out.println("最终map的值：" + map.toString());
```

```java
-------------------java8 写法（易理解）-------------------
最终hashMap的值：{销售部=6500.0, 测试部=14400.0, 财务部=6000.0, 人事部=4000.0, 研发部=31400.0, 行政部=3500.0, 运维部=8000.0}
-------------------java8 精简写法-------------------
最终map的值：{销售部=6500.0, 测试部=14400.0, 财务部=6000.0, 人事部=4000.0, 研发部=31400.0, 行政部=3500.0, 运维部=8000.0}
```



### 9、Java8 中 Map 的 10 个常用新方法

```java
import java.util.HashMap;
import java.util.Map;
import java.util.function.BiFunction;

/**
 * Java8中的Map的新方法
 */
public class MainTest {
    public static void main(String[] args) {
        //map 新特性
        Map<Integer,String> map = new HashMap<Integer,String>();
        for(int i=0; i<6; i++){
            map.put(i,"val_"+i);
        }
        map.put(10,null);

        //1:遍历
        map.forEach((key,value) -> System.out.println(key+":"+value));

        //2:V getOrDefault(key,defaultValue):获取key值,如果key不存在则用defaultValue
        System.out.println("3-->"+map.getOrDefault(3,"val_66"));//3-->val_3
        System.out.println("10-->"+map.getOrDefault(10,"val_66"));//10-->null
        System.out.println("11-->"+map.getOrDefault(11,"val_66"));//11-->val_66

        //3:V putIfAbsent(K key, V value):根据key匹配Node,如果匹配不到则增加key-value,返回null,如果匹配到Node,如果oldValue不等于null则不进行value覆盖，返回oldValue
        System.out.println(map.putIfAbsent(3,"val_66"));//val_3
        System.out.println(map.putIfAbsent(10,"val_66"));//null
        System.out.println(map.putIfAbsent(11,"val_66"));//null
        System.out.println(map.get(3)+"--"+map.get(10)+"--"+map.get(11));//val_3--val_66--val_66

        //4:boolean remove(Object key, Object value):根据key匹配node，如果value也相同则删除
        System.out.println(map.size());//8
        map.remove(10,"66");
        map.remove(11,"val_66");
        System.out.println(map.size());//7
        System.out.println(map.toString());//{0=val_0, 1=val_1, 2=val_2, 3=val_3, 4=val_4, 5=val_5, 10=val_66}

        //5:boolean replace(K key, V oldValue, V newValue):根据key匹配node,如果value也相同则使用newValue覆盖返回true，否则返回false
        map.put(11,null);
        map.replace(3,"3","33");
        map.replace(10,"val_66","val_666666");
        map.replace(11,null,"val_11");
        map.replace(11,null,"val_11");
        System.out.println(map.toString());//{0=val_0, 1=val_1, 2=val_2, 3=val_3, 4=val_4, 5=val_5, 10=val_666666, 11=val_11}
        /** 6:
         * void replaceAll(BiFunction function)：调用此方法时重写BiFunction的Object apply(Object o, Object o2)方法，
         * 其中o为key，o2为value，根据重写方法逻辑进行重新赋值。
         */
        map.replaceAll((key,value) -> {
            if(key == 2){
                return value+"222";
            }
            return value;
        });
        System.out.println(map.toString());//{0=val_0, 1=val_1, 2=val_2222, 3=val_3, 4=val_4, 5=val_5, 10=val_666666, 11=val_11}
        /** 7:
         * V compute(K key,BiFunction remappingFunction)：根据key做匹配，根据BiFunction的apply返回做存储的value。
         * 匹配到Node做value替换，匹配不到新增node。apply的返回值如果为null则删除该节点，否则即为要存储的value。
         */
        System.out.println("---------------------- compute -----------------------");
        System.out.println(map.compute(3,new BiFunction() {
            @Override
            public Object apply(Object key, Object value) {
                return key+":"+value;
            }
        }));//3:val_3 -》用返回值覆盖原来的值，这里用了java7的编码方式，以下均采用java8的lanbda表达式
        System.out.println(map.compute(10,(key,value) -> {return value.split("_")[1];}));//666666 -》用返回值覆盖原来的值
        System.out.println(map.compute(6,(key,value) ->  null));//null -》返回值为null，则删除该key值
        System.out.println(map.toString());//{0=val_0, 1=val_1, 2=val_2, 3=3:val_3, 4=val_4, 5=val_5, 10=666666, 11=val_11}
        /** 8:
         * merge(K key, V value,BiFunctionsuper V, ? super V, ? extends V> remappingFunction):
         * 功能大部分与compute相同，不同之处在于BiFunction中apply的参数，入参为oldValue、value，
         * 调用merge时根据两个value进行逻辑处理并返回value。
         */
        System.out.println(map.merge(3,"val_3",(value,newValue) -> newValue));//val_3  --》返回值覆盖原来的value
        System.out.println(map.merge(10,"33334",(a,b) -> (Integer.valueOf(a)+Integer.valueOf(b))+""));//700000
        System.out.println(map.merge(8,"88",(oldValue,newValue) -> oldValue+newValue));//88 -》key不存在则新增
        System.out.println(map.merge(11,"11",(old,newValue) -> null));//null -》返回值为null,删除该节点
        System.out.println(map.toString());//{0=val_0, 1=val_1, 2=val_2, 3=val_3, 4=val_4, 5=val_5, 8=88, 10=700000}
        /** 9:
         * computeIfAbsent(K key,Functionsuper K, ? extends V> mappingFunction):
         * 根据key做匹配Node，（匹配不到则新建然后重排）
         * 如果Node有value，则直接返回oldValue，
         * 如果没有value则根据Function接口的apply方法获取value，返回value。
         * Function接口的apply的入参为key，调用computeIfAbsent时重写Function接口可以根据key进行逻辑处理，
         * apply的返回值即为要存储的value。
         */
        System.out.println("----------------------computeIfAbsent------------------------");
        map.put(8,null);
        System.out.println(map.toString());//{0=val_0, 1=val_1, 2=val_2, 3=val_3, 4=val_4, 5=val_5, 8=null, 10=700000}
        System.out.println(map.computeIfAbsent(0,key -> key+"000"));//val_0  -》key值存在，直接返回oldValue
        System.out.println(map.computeIfAbsent(7,key -> "value_"+key));//value_7 -》key匹配不到，直接新增，返回值为value
        System.out.println(map.computeIfAbsent(8,key -> "88"));//88 -》key匹配到了，value为null，返回值作为value
        System.out.println(map.toString());//{0=val_0, 1=val_1, 2=val_2, 3=val_3, 4=val_4, 5=val_5, 7=value_7, 8=88, 10=700000}
        /** 10:
         * V computeIfPresent(K key,BiFunction remappingFunction)：
         * 根据key做匹配，如果匹配不上则返回null,匹配上根据BiFunction的apply方法获取value，返回value。
         * BiFunction接口的apply的入参为key、oldValue，调用computeIfPresent时重写Function接口
         * 可以根据key和oldValue进行逻辑处理，apply的返回值如果为null则删除该节点，否则即为要存储的value。
         */
        map.remove(7);
        map.remove(8);
        map.replace(10,null);
        map.remove(0,"val_0");//value匹配到了删除
        map.remove(1,"val_0");//value匹配失败，不会删除
        System.out.println(map.toString());//{1=val_1, 2=val_2, 3=val_3, 4=val_4, 5=val_5, 10=null}
        System.out.println(map.computeIfPresent(3,(key,value) -> key+":"+value));//3:val_3 -》key存在，根据返回值修改value
        System.out.println(map.computeIfPresent(0,(key, value) -> "0000"));//null -》key不存在，返回null,不做任何操作
        System.out.println(map.computeIfPresent(1,(key, value) -> null));//null -》key存在，根据返回值修改value
        System.out.println(map.computeIfPresent(10,(key,value) -> "val_10"));//null -》oldValue值为null，删除节点
        System.out.println(map.toString());//{2=val_2, 3=3:val_3, 4=val_4, 5=val_5, 10=null}
        /** 比较
         * compute：根据key做匹配，key,value为参数，匹配到Node做value替换，匹配不到新增node。apply的返回值为null则删除该节点。
         * merge：oldValue，newValue作为为参数，其它功能于compute类似
         * computeIfAbsent：根据key匹配，参数为key,存在且value不为null，不做修改，为null用返回值作为value，不存在则新增
         * computeIfPresent：key,value作为参数，存在,原来的值为null不做操作，否则返回值作为新的value覆盖原来；不存在，不做操作；返回值为null删除该节点
         *
         */
    }
}
```



# 栈与队列

## 1、Stack 栈

栈是一种先进后出的数据结构。例如：在文本编辑器上都有撤销功能，那么每次使用的时候你会发现，最后一次的编辑操作永远是最先撤销，那么这个功能就是利用栈来实现的，栈的基本操作形式如下。

![20211110141417](Java 类集框架.assets/20211110141417.png)

```java
public class Stack<E> extends Vector<E> {}
```

可以发现Stack是Vector子类，但是它使用的并不是Vector类之中所提供的方法，而是采用如下的两个方法：

```java
public E push(E item); // 数据入栈
public E pop(); // 数据出栈（弹出）
```

操作示例：Stack操作

```java
import java.util.Stack;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Stack<String> all = new Stack<>() ;
        all.push("A") ;
        all.push("B") ;
        all.push("C") ;
        System.out.println(all.pop());
        System.out.println(all.pop());
        System.out.println(all.pop());
        System.out.println(all.pop()); // 无数据、EmptyStackException
    }
}

// 输出内容
C
B
A
Exception in thread "main" java.util.EmptyStackException
	at java.base/java.util.Stack.peek(Stack.java:102)
	at java.base/java.util.Stack.pop(Stack.java:84)
	at JavaAPIDemo.main(JavaAPIDemo.java:11)
```

通过此时的操作可以发现，所有的保存之后将按照倒序的形式进行弹出，如果栈已经空了，则会抛出空栈异常。

![20211110143816](Java 类集框架.assets/20211110143816.png)



## 2、Queue 队列

Queue描述的是一个队列，而队列的主要特点是实现先进先出的操作形式。其基本的操作形式如下：

![20211110150302](Java 类集框架.assets/20211110150302.png)

如果将队列应用在多线程的“生产者与消费者”的模型处理上，那么对于生产者过快的情况下就没有必要等待消费者获取数据了，可以将所有的内容直接保存在队列之中，队列的实现可以使用LinkedList子类来完成，观察这个类的定义。

```java
public interface Queue<E> extends Collection<E> {}
```

![20211110150358](Java 类集框架.assets/20211110150358.png)

队列的使用主要依靠Queue接口之中提供的方法来处理，提供有如下的方法：

```java
public boolean offer(E e); // 向队列中追加数据，可以直接使用add()方法
public E poll();           // 通过队列获取数据，弹出后删除数据
```

操作示例 1：实现队列操作

```java
import java.util.LinkedList;
import java.util.Queue;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<>() ;
        // queue.add("X"); // 也可以使用父类的add()方法
        queue.offer("X") ;	// 追加队列数据，通过队尾追加
        queue.offer("A") ;	// 追加队列数据，通过队尾追加
        queue.offer("Z") ;	// 追加队列数据，通过队尾追加
        System.out.println(queue.poll()); // 弹出数据、X
        System.out.println(queue.poll()); // 弹出数据、A
        System.out.println(queue.poll()); // 弹出数据、Z
    }
}

// 输出内容
X
A
Z
```

除了Linkedlist子类之外，还有一个优先级队列的概念，可以使用PriorityQueue实现优先级队列（比较功能）

![20211110152736](Java 类集框架.assets/20211110152736.png)

操作示例 2：使用优先级队列

```java
import java.util.PriorityQueue;
import java.util.Queue;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Queue<String> queue = new PriorityQueue<>() ;
        queue.offer("X") ;	// 追加队列数据，通过队尾追加
        queue.offer("A") ;	// 追加队列数据，通过队尾追加
        queue.offer("Z") ;	// 追加队列数据，通过队尾追加
        System.out.println(queue.poll()); // 弹出数据、X
        System.out.println(queue.poll()); // 弹出数据、A
        System.out.println(queue.poll()); // 弹出数据、Z
    }
}

// 输出内容
A
X
Z
```

对于队列的选用原则也是需要根据实际的项目环境来决定的。

# Properties 工具类

在之前讲解国际化程序的时候讲解过资源文件（*.properties），那么这类文件的存储结构是按照“key= value”的形式存储的，而这种结构的保存形式与Map集合很相似的，但是唯一的区别在于其所保存的内容只能够是字符串，所以为了可以方便的描述属性的定义，java.util包里面提供有Properties类型，此类是Hashtable的子类。

```java
public class Properties extends Hashtable<Object, Object> {}
```

可以发现在继承Hashtable的时候为Hashtable中定义的泛型为Object，Properties是不需要操作泛型的，因为它可以操作的类型只能是String类型，在Properties之中如果要想实现属性的操作可以采用如下的方法：

```java
public Object setProperty(String key, String value);        // 设置属性
public String getProperty(String key);                      // 取得属性，key不存在返回null
public String getProperty(String key, String defaultValue); // 取得属性，不存在返回默认值
public void store(OutputStream, String comments);           // 输出属性内容（到输出流中）
public void load(InputStream instream);                     // 通过输入流读取属性内容
```

操作示例 1：观察属性的设置和取得

```java
import java.util.Properties;
public class JavaAPIDemo {
    public static void main(String[] args) {
        Properties prop = new Properties();
        // 设置的内容只允许是字符串
        prop.setProperty("name", "Sam");
        prop.setProperty("age", "18");
        System.out.println(prop.getProperty("name"));
        System.out.println(prop.getProperty("work"));
        System.out.println(prop.getProperty("work", "Java"));
    }
}

// 输出内容
Sam
null
Java
```

通过代码可以发现Properties里面可以像Map集合那样进行内容的设置与获取，但是唯一的差别是它只能够操作String类型，另外需要注意的是，之所以会提供有Properties类还有一个最重要的功能是它可以通过输出流输出属性，也可以使用输入流读取属性内容。

操作示例 2：将属性内容保存在文件之中

```java
import java.io.File;
import java.io.FileOutputStream;
import java.util.Properties;
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Properties prop = new Properties();
        // 设置的内容只允许是字符串
        prop.setProperty("name", "Sam");
        prop.setProperty("age", "18");
        prop.setProperty("address", "中国");
        prop.store(new FileOutputStream("D:\\info.properties"), "中文注释会被编码-English");
    }
}

// 输出到文件中的内容
#\u4E2D\u6587\u6CE8\u91CA\u4F1A\u88AB\u7F16\u7801-English
#Wed Nov 10 15:36:57 CST 2021
address=\u4E2D\u56FD
name=Sam
age=18
```

通过程序的执行可以发现,的确可以实现资源文件的输入处理，但是如果输出的是中文则自动帮助用户进行处理。

操作示例 3：读取资源文件

```java
import java.io.FileInputStream;
import java.util.Properties;
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Properties prop = new Properties() ;
        prop.load(new FileInputStream("D:\\info.properties"));
        System.out.println(prop.getProperty("name"));
    }
}

// 输出内容
Sam
```

使用Properties类型的最大特点是可以进行资源内容的输入与输出的处理操作，但是在实际的开发之中Properties往往用于读取配置资源的信息，这一点主要是在标准设计之中做程序初始化准备的时候使用。



# Collections 工具类

> 参考：https://www.cnblogs.com/guweiwei/p/6511974.html
>
> - https://mp.weixin.qq.com/s/kYKfGibLLwE-gylsclSuvg
> - https://mp.weixin.qq.com/s/K8PAFANbS_rBim0Uy9oOmQ

Collections是java提供的一组集合数据的操作工具类，也就是说利用它可以实现各个集合的操作。

![20211110154329](Java 类集框架.assets/20211110154329.png)

**Collections工具类常用方法：**

## 1、排序操作

（主要针对List接口相关）

```java
public static <T extends Comparable<? super T>> void sort(List<T> list); // 对List里的元素根据自然升序排序
public static <T> void sort(List<T> list, Comparator<? super T> c); // 自定义比较器进行排序
public static void reverse(List<?> list); // 反转指定List集合中元素的顺序
public static void shuffle(List<?> list); // 使用默认随机源对List中的元素进行随机排序（洗牌）
public static void swap(List<?> list, int i, int j); // 将指定List集合中i处元素和j出元素进行交换
public static void rotate(List<?> list, int distance); // 将所有元素向右移位指定长度，如果distance等于size那么结果不变

/**********************简单方法名**************************/
sort(List list)：对List里的元素根据自然升序排序
sort(List list, Comparator c)：自定义比较器进行排序
reverse(List list)：反转指定List集合中元素的顺序
shuffle(List list)：使用默认随机源对List中的元素进行随机排序（洗牌）
swap(List list, int i, int j)：将指定List集合中i处元素和j出元素进行交换
rotate(List list, int distance)：移动列表中的元素，负数向左移动，正数向右移动（集合中的元素向后移m个位置，在后面被遮盖的元素循环到前面来）
```

操作示例：

```java
import java.util.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(List.of("C", "B", "A", "D" ,"F", "E"));
        // 默认正序排序
        Collections.sort(list);
        System.out.println(list); // [A, B, C, D, E, F]
        // 自定义倒序排序
        Collections.sort(list,Collections.reverseOrder()); // [F, E, D, C, B, A]
        System.out.println(list);
        // 反转集合
        Collections.reverse(list);
        System.out.println(list); // [A, B, C, D, E, F]
        // 替换集合中左边0与5的位置
        Collections.swap(list, 0, 5);
        System.out.println(list); // [F, B, C, D, E, A]
        // 将所有元素向右移位1个位置，如果参数为负数，则是向左移动
        Collections.rotate(list, 1);
        System.out.println(list); // [A, F, B, C, D, E]
        // 随机排序集合，每次输出都不一样
        Collections.shuffle(list);
        System.out.println(list); // [D, B, E, A, C, F]
    }
}

// 输出内容
[A, B, C, D, E, F]
[F, E, D, C, B, A]
[A, B, C, D, E, F]
[F, B, C, D, E, A]
[A, F, B, C, D, E]
[D, B, E, A, C, F]
```



## 2、查找替换

查找和替换（主要针对Collection接口相关）

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src); // 将集合src中的元素全部复制到dest中,并且覆盖相应索引的元素
public static int indexOfSubList(List<?> source, List<?> target); // 查找target在source中首次出现位置的索引
public static int lastIndexOfSubList(List<?> source, List<?> target); // 查找target在source中末次出现位置的索引
public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll); // 返回最大元素
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp); // 根据自定义比较器，返回最大元素
public static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll); // 返回最小元素
public static <T> T min(Collection<? extends T> coll, Comparator<? super T> comp); // 根据自定义比较器，返回最小元素
public static <T> void fill(List<? super T> list, T obj); // 使用指定对象填充
public static int frequency(Collection<?> c, Object o); // 返回指定集合中指定对象出现的次数
public static <T> boolean replaceAll(List<T> list, T oldVal, T newVal); // 替换集合中指定元素
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key); // 查找指定集合中的元素,返回该元素的索引(二分法)
    
/**********************简单方法名**************************/
copy(List dest, List src)：将集合src中的元素全部复制到dest中,并且覆盖相应索引的元素
indexOfSubList(List source, List target)：查找target在source中首次出现位置的索引
lastIndexOfSubList(List source, List target)：查找target在source中末次出现位置的索引
max(Collection coll)：返回最大元素
max(Collection coll, Comparator comp)：根据自定义比较器，返回最大元素
min(Collection coll)：返回最小元素
min(Collection coll, Comparator comp)：根据自定义比较器，返回最小元素
fill(List list, Object obj)：使用指定对象填充（用对象obj替换集合list中的所有元素）
frequency(Collection Object o)：返回指定集合中指定对象出现的次数
replaceAll(List list, Object old, Object new)：替换集合中指定元素
binarySearch(List list, Object key)：使用二分搜索法，以获得指定对象在List中的索引，前提是集合已经排序
```

操作示例：

```java
import java.util.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        {
            // 将集合src中的元素全部复制到dest中,并且覆盖相应索引的元素
            List<String> dest = new ArrayList<>(List.of("C", "B", "A", "D" ,"F", "E"));
            List<String> src = new ArrayList<>(List.of("1", "2", "3"));
            Collections.copy(dest, src);
            System.out.println(dest); // [1, 2, 3, D, F, E]
        }
        {
            // 查找subList在list中首次出现位置的索引
            List<String> initList = new ArrayList<>(List.of("C", "B", "A", "D" ,"F", "E"));
            System.out.println(Collections.indexOfSubList(initList, List.of("A", "D"))); // 2
            System.out.println(Collections.indexOfSubList(initList, List.of("A", "F"))); // -1
            // 查找subList在list中末次次出现位置的索引
            System.out.println(Collections.indexOfSubList(initList, List.of("A", "D"))); // 2
            System.out.println(Collections.indexOfSubList(initList, List.of("A", "F"))); // -1
        }
        {
            // 返回集合中最大和最小元素
            List<String> initList = new ArrayList<>(List.of("C", "B", "A", "D" ,"F", "E"));
            System.out.println(Collections.min(initList)); // A
            System.out.println(Collections.max(initList)); // F
            // 返回指定集合中指定对象出现的次数
            System.out.println(Collections.frequency(initList, "A")); // 1
            // 查找指定集合中的元素，返回所查找元素的索引，PS：前提是集合已经排序
            Collections.sort(initList);
            System.out.println(Collections.binarySearch(initList, "C")); // 2
        }
        {
            // 用对象obj替换集合list中的所有元素
            List<String> initList = new ArrayList<>(List.of("C", "B", "A", "D" ,"F", "E"));
            Collections.fill(initList, "fill");
            System.out.println(initList); // [fill, fill, fill, fill, fill, fill]
        }
        {
            // 替换批定元素为某元素,若要替换的值存在刚返回true,反之返回false
            List<String> initList = new ArrayList<>(List.of("C", "B", "A", "D" ,"E", "E"));
            Collections.replaceAll(initList, "E", "X");
            System.out.println(initList); // [C, B, A, D, X, X]
        }
    }
}
```



## 3、同步控制

同步控制Collections：https://blog.csdn.net/weixin_38192427/article/details/116330109

同步控制Collections工具类中提供了多个synchronizedXxx()方法，该方法返回指定集合对象对应的同步对象，从而解决多线程并发访问集合时线程的安全问题。HashSet、ArrayList、HashMap都是线程不安全的，如果需要考虑同步，则使用这些方法。这些方法主要有：

```java
public static <T> Collection<T> synchronizedCollection(Collection<T> c);
public static <T> List<T> synchronizedList(List<T> list);
public static <T> Set<T> synchronizedSet(Set<T> s);
public static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s);
public static <T> NavigableSet<T> synchronizedNavigableSet(NavigableSet<T> s);
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m);
public static <K,V> NavigableMap<K,V> synchronizedNavigableMap(NavigableMap<K,V> m);

SynchronizedXxx(Xxx x); // 创建一个SynchronizedXxx，并将指定的集合x作为内部集合,此方法会将SynchronizedXxx实例本身作为互斥体（mutex）
SynchronizedXxx(Xxx x, Object mutex); // 创建一个SynchronizedXxx，并将指定的集合作为内部集合，将指定的对象作为互斥体（mutex）
```

SynchronizedMap类解读：

> 1、SynchronizedMap概述
>
> - SynchronizedMap是集合工具类Collections的一个静态内部类
> - SynchronizedMap实现了Map接口，同时实现了接口Serializable，支持序列化
> - 在多线程的情况下，能够保证线程的安全
>
> 2、SynchronizedMap的获取示例
>
> ```java
> Map<Integer, String> map = new HashMap<>();
> Map<Integer, String> synchronizedMap = Collections.synchronizedMap(map);
> 
> // Collections 调用的 synchronizedMap 方法如下
> public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
>     return new SynchronizedMap<>(m);
> }
> ```
>
> 3、SynchronizedMap源码解读
>
> ```java
> private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable {
>     // 1、类的基本属性
>     private final Map<K,V> m;     // 被 final 修饰的普通 Map 对象
>     final Object      mutex;        // 互斥锁对象
> 
>     // 2、构造函数
>     // 不指定锁对象
>     SynchronizedMap(Map<K,V> m) {
>         // 不指定锁对象时，锁对象就是 SynchronizedMap 本身
>         this.m = Objects.requireNonNull(m);
>         mutex = this;
>     }
>     // 自定义锁对象
>     SynchronizedMap(Map<K,V> m, Object mutex) {
>         // 锁对象为传入的 mutex
>         this.m = m;
>         this.mutex = mutex;
>     }
> 
>     // 3、常见的 API 方法
>     public int size() {
>         synchronized (mutex) {return m.size();}
>     }
>     public boolean isEmpty() {
>         synchronized (mutex) {return m.isEmpty();}
>     }
>     public boolean containsKey(Object key) {
>         synchronized (mutex) {return m.containsKey(key);}
>     }
>     public boolean containsValue(Object value) {
>         synchronized (mutex) {return m.containsValue(value);}
>     }
>     public V get(Object key) {
>         synchronized (mutex) {return m.get(key);}
>     }
>     public V put(K key, V value) {
>         synchronized (mutex) {return m.put(key, value);}
>     }
>     // ...等等方法
> }
> ```
>
> - 可以发现它的所有方法都是加了synchronized互斥锁关键字
> - 此外SynchronizedMap还重写了Map中的一些方法，也都是加了synchronized互斥锁关键字
>
> 
>
> 4、SynchronizedMap，Hashtable与ConcurrentHashMap 对比：
>
> - 相同点：1）三者都实现了Map接口。2）在多线程的情况下，都可以保证线程的安全
> - 不同点：
>   1. 底层使用的数据结构：Hashtable使用了 数组 + 链表，ConcurrentHashMap 使用了数组 + 链表 + 红黑树
>   2. 默认的初始容量：HashTable是 11，ConcurrentHashMap是 16
>   3. 保证线程安全的机制：SynchronizedMap和Hashtable都是为每个方法加了synchronized互斥锁关键字，ConcurrentHashMap使用了 Synchronized + CAS + Volatile机制来保证线程安全
>   4. 多线程的情况下执行效率：ConcurrentHashMap执行效率要高于SynchronizedMap和Hashtable

SynchronizedList类解读：

> 1、SynchronizedList 概述
>
> - SynchronizedList是集合工具类Collections的一个静态内部类
> - SynchronizedList实现了List接口
> - 在多线程的情况下，能够保证线程的安全
>
> 2、SynchronizedList的获取示例
>
> ```java
> List<Integer> newList = new ArrayList<>();
> List<Integer> synchronizedList = Collections.synchronizedList(newList);
> 
> // Collections 调用 synchronizedList 方法如下
> public static <T> List<T> synchronizedList(List<T> list) {
>     return (list instanceof RandomAccess ?
>             new SynchronizedRandomAccessList<>(list) :
>             new SynchronizedList<>(list));
> }
> ```
>
> 3、SynchronizedList源码解读
>
> ```java
> static class SynchronizedList<E>
>         extends SynchronizedCollection<E>
>         implements List<E> {
>         private static final long serialVersionUID = -7754090372962971524L;
> 
>     // 1、类的基本属性
>     final List<E> list;
> 
>     // 2、构造函数
>     SynchronizedList(List<E> list) {
>         super(list);
>         this.list = list;
>     }
>     // 指定互斥锁对象，并调用父类 SynchronizedCollection 的构造函数
>     SynchronizedList(List<E> list, Object mutex) {
>         super(list, mutex);
>         this.list = list;
>     }
> 
>     // 3、常用API
>     public E get(int index) {
>         synchronized (mutex) {return list.get(index);}
>     }
>     public E set(int index, E element) {
>         synchronized (mutex) {return list.set(index, element);}
>     }
>     public void add(int index, E element) {
>         synchronized (mutex) {list.add(index, element);}
>     }
>     public E remove(int index) {
>         synchronized (mutex) {return list.remove(index);}
>     }
>     // 其父类 SynchronizedCollection 的方法
>     public int size() {
>         synchronized (mutex) {return c.size();}
>     }
>     // ...等等方法
> }
> ```
>
> - 可以看到添加、删除、修改、查询等操作都通过 synchronized 实现了同步锁来保证这些操作是线程安全的
> - 在迭代器操作是没有添加同步锁的，因此使用迭代器进行操作使会出现线程不安全的情况，但我们也可以手动添加同步锁来实现迭代的时候线程安全
>
> 
>
> 4、SynchronizedList，Vector与CopyOnWriteArrayList 对比：
>
> - 相同点：三者都是线程安全的List
> - 不同点：
>   1. 扩容机制：Vector 每次扩容的大小都是原来数组大小的 2 倍，而 CopyOnWriteArrayList 不需要扩容，通过 COW 思想就能使数组容量满足要求
>   2. 保证线程安全机制：SynchronizedList 和 Vector 的每个方法都进行了加锁，而 CopyOnWriteArrayList 的读操作是不加锁的，因此 CopyOnWriteArrayList 的读性能远高于 SynchronizedList 和 Vector
>   3. 在多线程的读多写少的情况下执行效率：CopyOnWriteArrayList 要高于 SynchronizedList 和 Vector

SynchronizedSet类解读：

> 1、synchronizedSet 概述
>
> - SynchronizedSet 是集合工具类 Collections 的一个静态内部类
> - SynchronizedSet 实现了 Set 接口
> - 在多线程的情况下，能够保证线程的安全
>
> 2、SynchronizedSet 的获取示例
>
> ```java
> Set<String> set = new HashSet<>();
> Set<String> synchronizedSet = Collections.synchronizedSet(set);
> 
> // Collections 调用 synchronizedSet 方法如下
> public static <T> Set<T> synchronizedSet(Set<T> s) {
>     return new SynchronizedSet<>(s);
> }
> ```
>
> 3、SynchronizedSet 源码解读
>
> ```java
> static class SynchronizedSet<E> extends SynchronizedCollection<E> implements Set<E> {
>     SynchronizedSet(Set<E> s) {
>         super(s);
>     }
>     SynchronizedSet(Set<E> s, Object mutex) {
>         super(s, mutex);
>     }
>     public boolean equals(Object o) {
>         if (this == o)
>             return true;
>         synchronized (mutex) {return c.equals(o);}
>     }
>     public int hashCode() {
>         synchronized (mutex) {return c.hashCode();}
>     }
> }
> 
> // 在 synchronizedSet 中只有 equals() 和 hashCode() 方法，所以我们看其父类 SynchronizedCollection的方法
> public int size() {
> 	synchronized (mutex) {return c.size();}
> }
> public boolean isEmpty() {
> 	synchronized (mutex) {return c.isEmpty();}
> }
> public boolean contains(Object o) {
> 	synchronized (mutex) {return c.contains(o);}
> }
> public Iterator<E> iterator() {
> 	return c.iterator(); 
> }
> public boolean add(E e) {
> 	synchronized (mutex) {return c.add(e);}
> }      
> public boolean remove(Object o) {
> 	synchronized (mutex) {return c.remove(o);}
> }
> public boolean addAll(Collection<? extends E> coll) {
> 	synchronized (mutex) {return c.addAll(coll);}
> }
> public boolean removeAll(Collection<?> coll) {
> 	synchronized (mutex) {return c.removeAll(coll);}
> }     
> @Override
> public void forEach(Consumer<? super E> consumer) {
> 	synchronized (mutex) {c.forEach(consumer);}
> }
> @Override
> public Stream<E> stream() {
> 	return c.stream(); 
> }
> ```
>
> - 可以看到添加、删除等操作都通过 synchronized 实现了同步锁来保证这些操作是线程安全的
> - 在迭代器操作是没有添加同步锁的，因此使用迭代器进行操作使会出现线程不安全的情况，但我们也可以手动添加同步锁来实现迭代的时候线程安全
>
> 4、SynchronizedSet 与 CopyOnWriteArraySet 对比：
>
> - 相同点：二者都是线程安全的Set
> - 不同点：
>   1. 实现机制：CopyOnWriteArraySet 是通过 CopyOnWriteArrayList 实现的。而CopyOnWriteArrayList 本质是个动态数组，所以 CopyOnWriteArraySet 相当于通过动态数组实现的 Set，而 SynchronizedSet 的实现依据于如何创建 SynchronizedSet 的实例
>   2. 保证线程安全的机制：CopyOnWriteArraySet 通过 volatile 和 ReentrantLock 来实现的，而 SynchronizedSet 是通过给添加、删除等方法添加 synchronized 同步锁来保证的
>   3. 在多线程的读多写少的情况下执行效率：CopyOnWriteArraySet 要高于 SynchronizedSet





## 4、不可变集合

设置不可变集合，Collections有三类方法可返回一个不可变集合：

1. emptyXxx()：返回一个空的不可变的集合对象
2. singletonXxx()：返回一个只包含指定对象的，不可变的集合对象
3. unmodifiableXxx()：返回指定集合对象的不可变视图

```java
// emptyXxx()：返回一个空的不可变的集合
public static <T> ListIterator<T> emptyListIterator();
public static <T> Enumeration<T> emptyEnumeration();
public static final <T> Set<T> emptySet();
public static <E> SortedSet<E> emptySortedSet();
public static <E> NavigableSet<E> emptyNavigableSet();
public static final <T> List<T> emptyList();
public static final <K,V> Map<K,V> emptyMap();
public static final <K,V> SortedMap<K,V> emptySortedMap();
public static final <K,V> NavigableMap<K,V> emptyNavigableMap();

// singletonXxx()：返回一个只包含指定对象的，不可变的集合
public static <T> Set<T> singleton(T o);
public static <T> List<T> singletonList(T o);
public static <K,V> Map<K,V> singletonMap(K key, V value);

// unmodifiableXxx()：返回指定集合对象的不可变视图
public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c);
public static <T> Set<T> unmodifiableSet(Set<? extends T> s);
public static <T> SortedSet<T> unmodifiableSortedSet(SortedSet<T> s);
public static <T> NavigableSet<T> unmodifiableNavigableSet(NavigableSet<T> s);
public static <T> List<T> unmodifiableList(List<? extends T> list);
public static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m);
public static <K,V> SortedMap<K,V> unmodifiableSortedMap(SortedMap<K, ? extends V> m);
public static <K,V> NavigableMap<K,V> unmodifiableNavigableMap(NavigableMap<K, ? extends V> m);
```

操作示例：

```java
import java.util.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 1、emptyXxx()：返回一个空的不可变的集合
        try {
            List<String> emptyList = Collections.emptyList();
            // 抛出：java.lang.UnsupportedOperationException
            emptyList.add("X");
        } catch (UnsupportedOperationException e) {
            e.printStackTrace();
        }

        // 2、singletonXxx()：返回一个只包含指定对象的，不可变的集合
        try {
            List<String> singletonList = Collections.singletonList("X");
            System.out.println(singletonList.get(0));
            // 抛出：java.lang.UnsupportedOperationException
            singletonList.add("");
        } catch (UnsupportedOperationException e) {
            e.printStackTrace();
        }

        // 3、unmodifiableXxx()：返回指定集合对象的不可变视图
        try {
            // 初始化集合
            List<String> list = new ArrayList<>(List.of("C", "B", "A", "D" ,"F", "E"));
            List<String> unmodList = Collections.unmodifiableList(list);
            // 抛出：java.lang.UnsupportedOperationException
            unmodList.add("再加个试试！");
        } catch (UnsupportedOperationException e) {
            e.printStackTrace();
        }
    }
}

// 输出内容
java.lang.UnsupportedOperationException
	at java.base/java.util.AbstractList.add(AbstractList.java:153)
	at java.base/java.util.AbstractList.add(AbstractList.java:111)
	at JavaAPIDemo.main(JavaAPIDemo.java:8)
X
java.lang.UnsupportedOperationException
	at java.base/java.util.AbstractList.add(AbstractList.java:153)
	at java.base/java.util.AbstractList.add(AbstractList.java:111)
	at JavaAPIDemo.main(JavaAPIDemo.java:18)
java.lang.UnsupportedOperationException
	at java.base/java.util.Collections$UnmodifiableCollection.add(Collections.java:1060)
	at JavaAPIDemo.main(JavaAPIDemo.java:29)
```



## 5、其它常用方法

```java
public static <T> boolean addAll(Collection<? super T> c, T... elements); // 将所有指定元素添加(追加)到指定collection中
public static boolean disjoint(Collection<?> c1, Collection<?> c2); // 如果两个指定collection中没有相同的元素则返回true
public static <T> List<T> nCopies(int n, T o); // 返回大小为n的List，其中的所有引用都指向o，List是不可变得（与fill方法类似）
public static <T> Comparator<T> reverseOrder(); // 返回一个比较器，它强行反转指定比较器的顺序
```

操作示例：

```java
import java.util.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 初始化集合List
        List<String> list = new ArrayList<>(List.of("你好"));
        List<String> list1 = new ArrayList<>(List.of("list1"));
        List<String> list2 = new ArrayList<>();

        // addAll增加变长参数
        Collections.addAll(list1, "大家好", "你好","我也好");
        Collections.addAll(list2, "大家好", "a李四","我也好");
        System.out.println(list1); // [list1, 大家好, 你好, 我也好]
        System.out.println(list2); // [大家好, a李四, 我也好]

        // disjoint检查两个Collection是否的交集
        boolean b1 = Collections.disjoint(list, list1);
        boolean b2 = Collections.disjoint(list, list2);
        System.out.println(b1 + "\t" + b2); // false	true

        // 生产大小为5，内容全为 中国 的不可变的List集合
        System.out.println(Collections.nCopies(5, "中国")); // [中国, 中国, 中国, 中国, 中国]
        
        // 利用reverseOrder倒序
        Collections.sort(list1, Collections.reverseOrder());
        System.out.println(list1); // [我也好, 大家好, 你好, list1]
    }
}
```

**面试题：**请解释Collection与Collections的区别？

1. Collection 是集合接口，允许保存单值对象
2. Collections 是集合操作的工具类



## 6、参考文献 & 鸣谢

1. Java集合的详细研究：https://www.cnblogs.com/guweiwei/category/960786.html
1. 集合源码入门教程：https://www.cnblogs.com/chanshuyi/category/1366321.html



# Java 集合汇总篇

## 1、List接口与接口实现类

> 有序、不唯一

ArrayList：

1. 内部是通过数组实现的，允许对元素进行快速随机访问。
2. 当数组大小不满足时需要增加存储能力，就要将已经有数组的数据复制到新的存储空间中。
3. 适合随机查找和遍历，不适合插入和删除。当从 ArrayList 的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。

LinkedList：

1. LinkedList 是用链表结构存储数据的。
2. 很适合数据的动态插入和删除，随机访问和遍历速度比较慢。
3. 有List 接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。

Vector（线程安全）：

- 通过数组实现的。
- 默认情况下 Vector 每次扩容时容量都会翻倍。
- 支持线程的同步，即某一时刻只有一个线程能够写 Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此访问它比 ArrayList 慢

CopyOnWriteArrayList：

1. 写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。
2. 写操作需要加锁，防止并发写入时导致写入数据丢失。
3. 写操作结束之后需要把原始数组指向新的复制数组。
4. 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。
5. 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右。
6. 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。



## 2、Set接口与接口实现类

> 无序、不可重复

 HashSet：

1. 哈希表边存放的是哈希值。
2. HashSet这个类实现了Set集合，实际为一个HashMap的实例。
3. HashSet 存储元素的顺序并不是按照存入时的顺序。而是按照哈希值来存的所以取数据也是按照哈希值取得。元素的哈希值是通过元素的hashcode 方法来获取的, HashSet 首先判断两个元素的哈希值，如果哈希值一样（哈希冲突，也叫哈希碰撞），接着会比较equals 方法 如果 equls 结果为 true，HashSet 就视为同一个元素。如果 equals 为 false 就不是同一个元素。HashSet 通过 hashCode 值来确定元素在内存中的位置。 一个 hashCode 位置上可以存放多个元素。

LinkedHashSet：

1. 能保证怎么存就怎么取的set集合对象。
2. LinkedHashSet是一个哈希表和链表的结合，且是一个双向链表。
3. LinkedHashSet 底层使用 LinkedHashMap 来保存所有元素。
4. 继承与 HashSet，其所有的方法操作上又与 HashSet 相同。

TreeSet：

1. TreeSet是使用二叉树的原理对新添加的对象按照指定的顺序排序（升序、降序），每增加一个对象都会进行排序，将对象插入的二叉树指定的位置。
2. Integer 和 String 对象都可以进行默认的 TreeSet 排序，而自定义类的对象是不可以的， 自己定义的类必须实现 Comparable 接口，并且覆写相应的 compareTo()函数，才可以正常使用。
3. 在覆写 compare()函数时，要返回相应的值才能使 TreeSet 按照一定的规则来排序。
4. 比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则分别返回负整数、零或正整数。



## 3、Map接口与接口实现类

> 不能包含重复的键、每个键最多只能映射到一个值

LinkedHashMap：

1. 迭代顺序可以是插入顺序。
2. Key和Value都允许空，Key重复会覆盖、Value允许重复。
3. LinkedHashMap 是 HashMap 的一个子类，保存了记录的插入顺序，在用 Iterator 遍历。
4. LinkedHashMap 时，先得到的记录肯定是先插入的，也可以在构造时带参数，按照访问次序排序。

HashMap：

1. HashMap 根据键的 hashCode 值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。
2. HashMap 最多只允许一条记录的键为 null，允许多条记录的值为 null。
3. HashMap 非线程安全，即任一时刻可以有多个线程同时写 HashMap，可能会导致数据的不一致。
4. 如果需要满足线程安全，可以用 Collections 的 synchronizedMap 方法使 HashMap 具有线程安全的能力，或者使用 ConcurrentHashMap。

TreeMap：

1. TreeMap 实现 SortedMap 接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用 Iterator 遍历 TreeMap 时，得到的记录是排过序的。如果使用排序的映射，建议使用 TreeMap。
2. key 必须实现 Comparable 接口或者在构造 TreeMap 传入自定义的Comparator，否则会在运行时抛出 java.lang.ClassCastException 类型的异常。

Hashtable（线程安全）：

1. Hashtable 是遗留类，很多映射的常用功能与 HashMap 类似。
2. 承自 Dictionary 类。
3. 是线程安全的，任一时间只有一个线程能写 Hashtable。
4. 并发性不如 ConcurrentHashMap，因为 ConcurrentHashMap 引入了分段锁。 Hashtable 不建议在新代码中使用，不需要线程安全的场合可以用 HashMap 替换，需要线程安全的场合可以用 ConcurrentHashMap 替换。

ConcurrentHashMap 线程安全（Segment 继承 ReentrantLock 加锁）：

1. ConcurrentHashMap 和 HashMap 思路是差不多的。
2. 支持并发操作。
3. 整个 ConcurrentHashMap 由一个个 Segment 组成， Segment 代表”部分“或”一段“的意思，所以很多地方都会将其描述为分段锁。注意，行文中，我很多地方用了“槽”来代表一个segment。



## 4、线程安全处理方法

```java
// Collections.synchronizedXxx()
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);

Map<String,String> map = new HashMap<>();
Map<String,String> synMap = Collections.synchronizedMap(map);

Set<String> set = new HashSet<>();
Set<String> synset = Collections.synchronizedSet(set);
```



## 5、ArrayList、LinkedList和Vector的区别

- ArrayList: 可以看作是能够自动增长容量的数组。
- ArrayList底层的实现是Array, 数组扩容实现。
- ArrayList根据下标查询快，但是根据值查数据速度和LinkList一样。
- ArrayList线程不安全，效率高。

- LinkList是一个双链表,在添加和删除元素时具有比ArrayList更好的性能。
- LinkList线程不安全，效率高。

- LinkList是一个双链表,在添加和删除元素时具有比ArrayList更好的性能。
- LinkList线程不安全，效率高。



## 6、HashMap和HashTable的区别

1. HashMap是线程不安全的，所以效率远远高于Hashtable，当需要多线程操作的时候可以使用线程安全的ConcurrentHashMap，ConcurrentHashMap虽然也是线程安全的，但是它的效率比Hashtable要高好多倍。因为ConcurrentHashMap使用了分段锁，并不对整个数据进行锁定。
2. HashMap：key可以为null，但是这样的key只能有一个，因为必须保证key的唯一性可以有多个key值对应的value为null。

3. Hashtable是线程安全的，它的每个方法上都有synchronized 关键字。
4. Hashtable：key和value都不能为null。



## 7、HashMap和HashTable的区别

- List：有序、不唯一。
- Set ：无序、唯一。
- Map：不能包含重复的键、每个键最多只能映射到一个值、元素存储循序是无序的、以键值对存储数据。

