# 一、语言方法新特性

## 1、接口默认方法和静态方法

### 1、接口默认方法

Java 8 引入了接口的默认方法。添加此功能是为了向后兼容，以便可以使用旧接口来利用 Java 8 的 Lambda 表达式功能。例如，List 或 Collection 接口没有 forEach 方法声明。因此，添加这样的方法只会破坏集合框架的实现。Java 8 引入了默认方法，以便 List 或  Collection 接口可以有 forEach 方法的默认实现，而实现这些接口的类不需要实现相同的方法。

并且在 JVM 中，默认方法的实现是非常高效的，并且通过字节码指令为方法调用提供了支持。默认方法允许继续使用现有的 Java 接口，而同时能够保障正常的编译过程。这方面好的例子是大量的方法被添加到 java.util.Collection 接口中去：stream()，parallelStream()，forEach()，removeIf() 等。尽管默认方法非常强大，但是在使用默认方法时我们需要小心注意一个地方：**在声明一个默认方法前，请仔细思考是不是真的有必要使用默认方法**。

【操作示例 1】默认方法实现

```java
public interface TestInterface {
    default void sameMethod() {
        System.out.println("Invoke TestInterface method！");
    }
}
```

【操作示例 2】多个默认方法，通过接口中的默认函数，有可能一个类使用相同的默认方法实现两个接口。

```java
// 测试接口 1
public interface TestInterface1 {
    default void sameMethod() { System.out.println("Invoke TestInterface1 method！"); }
}
// 测试接口 2
public interface TestInterface2 {
    default void sameMethod() { System.out.println("Invoke TestInterface2 method！"); }
}
```

- 第一个解决方案是创建自己的方法，覆盖默认实现。

  ```java
  // 继承两个接口的测试类
  public class TestObject implements TestInterface1, TestInterface2 {
      @Override
      public void sameMethod() {
          System.out.println("Invoke Object method！");
      }
  }
  ```

- 第二个解决方案是使用super调用指定接口的默认方法。

  ```java
  // 继承两个接口的测试类
  public class TestObject implements TestInterface1, TestInterface2 {
      @Override
      public void sameMethod() {
            TestInterface1.super.sameMethod();
      }
  }
  ```

注意：在 Java 8 之前，接口中的实现方法必须是 abstract 的，实现该接口的类必须重写该方法，接口只负责声明该方法。Java 8 给接口增加了default 关键词，用 default 修饰的方法可以有实现内容，实现该接口的类可以不重写用default修饰的方法，类似于继承。但这样也会带来新的问题。Java 中只能继承一个类，但是可以实现多个接口，当多个接口中有同一个方法时，以前是没问题的，因为实现类必须重写方法。但现在，当多个接口中有同一个用 default 修饰的方法时，就无法判断到底实现的是哪个接口的方法。这种情况下，就必须重写方法。还有一种情况，一个类继承的父类和实现的接口中都有同一个方法，而这个类又没有重写时，实现的是父类的方法，而不是接口中的方法。



### 2、接口静态方法

在 Java 8 中，允许在接口中增加静态方法 *(允许不构建对象而直接使用的具体方法)*。理论上讲，没有任何理由认为这是不合法的，**只是这有违将接口作为抽象规范的初衷**。

【操作示例 1】接口中声明静态方法

```java
public interface StaticInterface {
    static void method() {
        System.out.println("这是接口中的静态方法!");
    }
}
```

目前为止，通常的做法都是将静态方法放在 **伴随类**（可以理解为操作继承接口的实用工具类）中。在标准库中，你可以看到成对出现的接口和实用工具类，如 Collection / Collections 或 Path / Paths。

在 Java 11 中，Path 接口就提供了一个与之工具类 Paths.get() 等价的方法（该方法用于将一个 URI 或者字符串序列构造成一个文件或目录的路径）：

```java
public interface Path {
    public static Path of(String first, String... more) { ... }
    public static Path of(URI uri) { ... }
}
```

这样一来，Paths 类就不再是必要的了。类似地，如果实现你自己的接口时，没有理由再额外提供一个带有实用方法的工具类。

另外注意：在 Java 9 中，接口中的方法可以是 private。private 方法可以是静态方法或实例方法。由于私有方法只能在接口本身的方法中使用，所以它们的用法很有限，只能作为接口中其他方法的辅助方法。



## 2、Lambda 表达式

从 Java 8 开始为了简化使用者进行代码的开发，专门提供有 Lambda 表达式的支持（Lambda 也可称为闭包），利用此操作形式可以实现函数式的编程。对于函数式编程比较著名的语言：haskell、Scala，利用函数式编程可以避免掉面向对象编程之中的一些繁琐的处理问题。面向对象在其长期发展的过程之中一直有一部分的反对者，这些反对者认为面向对象的设计过于复杂，过于繁琐了。以一个最简单的程序为例。

【操作示例 1】观察传统开发中的问题

```java
interface IMessage {
    public void send(String str);
}
public class JavaAPIDemo {
    public static void main(String[] args) {
        IMessage msg = new IMessage() {
            @Override
            public void send(String str) {
                System.out.println("消息发送：" + str);
            }
        } ;
        msg.send("Hello Java");
    }
}
```

在这样一个程序里面，实际上核心的功能只有一行语句“System.out.println("消息发送："+ str);”，但是为了这一行的核心语句依然需要按照完整的面向对象给出的设计结构进行开发。于是这些问题随着技术的发展也是越来越突出了。

【操作示例 2】使用 Lambda 表达式实现与之前完全一样的功能

```java
interface IMessage {
    public void send(String str);
}
public class JavaAPIDemo {
    public static void main(String[] args) {
        IMessage msg = str -> System.out.println("消息发送：" + str);
        msg.send("Hello Java");
    }
}
```

现在整个程序代码里面会发现真的只是编写了一行语句，于是利用这种形式就避免了复杂的面向对象结构化的要求。

Lambda 表达式如果要想使用，那么必须有一个重要的实现要求：SAM（Single abstract method），只有一个抽象方法，以之前的 IMessage 接口为例，在这个接口里面发现只是提供有一个send方法，除此之外没有任何的其它方法定义，所以这样的接口就被称为函数式接口，而只有函数式接口才可以被 Lambda表达式所使用。

【操作示例 3】使用函数式接口注解

```java
// 函数式接口
@FunctionalInterface
interface IMessage {
    public void send(String str);
}
public class JavaDemo {
    public static void main(String args[]) {
        IMessage msg = str -> System.out.println("消息发送：" + str);
        msg.send("Hello Java");
    }
}
```

【操作示例 4】定义没有参数的方法

```java
// 函数式接口
@FunctionalInterface
interface IMessage {
	public void send() ;
}
public class JavaDemo {
	public static void main(String args[]) {
		IMessage msg = () -> System.out.println("Hello Java");
		msg.send() ;
	}
}
```

【操作示例 5】定义有参数的方法

```java
// 函数式接口
@FunctionalInterface
interface IMath {
    public int add(int x, int y);
}
public class JavaAPIDemo {
    public static void main(String[] args) {
        IMath math = (t1,t2) -> return t1 + t2;
        System.out.println(math.add(10, 20));
    }
}
```

对于 Lambda 表达式而言，提供有如下几种格式：

1. 方法没有参数：() ->{};
2. 方法有参数：(参数, 参数) -> {};
3. 如果现在只有一行语句返回：(参数, 参数)->语句

以下是 Lambda 表达式的重要特征:

- 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
- 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。
- 可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
- 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

Lambda 表达式的最大作用是简化了匿名内部类的使用，是的代码看起来更加简洁。（注意：Lambda 表达式并不能提升代码执行的效率）。
Lambda 表达式还有一个作用就是推动了 Java 中的函数化编程，使得可以将一个函数作为参数传给方法。（之前必须传一个对象的引用给方法，然后再通过这个对象引用调用具体的方法）。



## 3、方法引用

引用数据类型最大的特点是可以进行内存的指向处理，但是在传统的开发之中一直所使用的只是对象引用操作，而从JDK1.8之后也提供有方法的引用，即：不同的方法名称可以描述同一个方法。如果要进行方法的引用在Java中提供有如下的四种形式：

1. 引用构造方法：类名称 :: new（ClassName::new）
2. 引用静态方法：类名称 :: static 方法名称（ContainingClass::staticMethodName）
3. 引用实例方法：实例化对象 :: 普通方法（containingObject::instanceMethodName）
4. 引用特定类型的方法：特定类 :: 普通方法（ContainingType::methodName）

下面的例子中，**Tester** 类是不同方法引用的例子，可以帮助读者区分四种类型的方法引用。

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Supplier;

public class Tester {
    public static Tester create(final Supplier<Tester> supplier) {
        return supplier.get();
    }

    public static void collide(final Tester tester) {
        System.out.println("Collided " + tester.toString());
    }

    public void follow(final Tester another) {
        System.out.println("Following the " + another.toString());
    }

    public void repair() {
        System.out.println("Repaired " + this);
    }
}
```

第一种方法引用的类型是构造器引用，语法是 **Class::new**，或者更一般的形式：**Class< T>::new**。注意：这个构造器没有参数。

```java
Tester       tester  = Tester.create(Tester::new);
List<Tester> testers = Arrays.asList(tester);
```

第二种方法引用的类型是静态方法引用，语法是**Class::static_method**。注意：这个方法接受一个 Tester 类型的参数。

```java
List<Tester> testers = Arrays.asList(Tester.create(Tester::new));
testers.forEach(Tester::collide);
```

第三种方法引用的类型是某个实例对象的成员方法的引用，语法是**instance::method**。注意：这个方法接受一个 Tester 类型的参数：

```java
Tester tester = Tester.create(Tester::new);
List<Tester> testers = Arrays.asList(Tester.create(Tester::new));
testers.forEach(tester::follow);
```

第四种方法引用的类型是某个类的成员方法的引用，语法是**Class::method**，注意，这个方法没有定义入参：

```java
List<Tester> testers = Arrays.asList(Tester.create(Tester::new));
testers.forEach(Tester::repair);
```

在进行方法应用的时候也可以引用特定类中的一些操作方法，在 String 类里面提供有一个字符串大小关系比较：public int compareTo(String anotherString); 这是一个普通方法，如果要引用普通，则往往都需要实例化对象，但是如果说现在你不想给出实例化对象，只是想引用这个方法，则就可以使用特定类来进行引用处理。

```java
Comparator<String> comparator = String::compareTo;
System.out.println(comparator.compare("a", "b"));
```



## 4、函数式接口

函数式接口（Functional Interface）就是一个**有且仅有一个抽象方法，但是可以有多个非抽象方法的接口**。函数式接口可以被隐式转换为 Lambda 表达式。Java 8 API 包含了很多内建的函数式接口，在老 Java 中常用到的比如 Comparator 或者 Runnable 接口，这些接口都增加了 @FunctionalInterface 注解以便能用在 Lambda 上。

Java 8 提供有一个 java.util.function 的开发包，里面可以直接使用函数式接口，在这个包下面一共有如下的几个核心接口供使用：

```java
// 1.功能性函数型接⼝：有⼊参，有返回值
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

// 2.消费型接⼝：有⼊参，⽆返回值
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
    
// 3.供给型接⼝：⽆⼊参，有返回值
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

// 4.断⾔型接⼝：有⼊参，有返回值，返回值类型确定是boolean
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

```java
import java.util.function.*;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 功能性函数式接口, 例如: String 类中的方法: public boolean startsWith(String str);
        Function<String,Boolean> fun = "**Hello" :: startsWith;
        System.out.println(fun.apply("**")) ;
        
        // 消费型接⼝, 例如：在进行系统数据输出的时候使用的是：System.out.println();
        Consumer<String> con = System.out :: println;
        con.accept("hello java") ;	
        
        // 供给型接⼝, 例如: String 类中的方法: public String toLowerCase();
        Supplier<String> sup = "HELLO java" :: toLowerCase;
        System.out.println(sup.get());
        
        // 断⾔型接⼝, 例如: String类中的方法: public boolean equalsIgnoreCase();
        Predicate<String> pre = "xxxx" :: equalsIgnoreCase;
        System.out.println(pre.test("XXXX"));
    }
}
```

有一点需要注意：**默认方法和静态方法**不会破坏函数式接口的定义，因此如下的代码是合法的。

```java
@FunctionalInterface
public interface FunctionalDefaultMethods {
    void method();
    default void defaultMethod() {}
}
```



### 1、Function（功能型接口）

Function 接口有一个参数并且返回一个结果，并附带了一些可以和其他函数组合的默认方法（compose, andThen）。源代码及使用示例如下：

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);
backToString.apply("123");     // "123"
```

更多的 Function 接口

- `BiFunction ：R apply(T t, U u);`: 接受两个参数，返回一个值，代表一个二元函数；
- `DoubleFunction ：R apply(double value);`: 只处理double类型的一元函数；
- `IntFunction ：R apply(int value);`: 只处理int参数的一元函数；
- `LongFunction ：R apply(long value);`: 只处理long参数的一元函数；
- `ToDoubleFunction：double applyAsDouble(T value);`: 返回double的一元函数；
- `ToDoubleBiFunction：double applyAsDouble(T t, U u);`: 返回double的二元函数；
- `ToIntFunction：int applyAsInt(T value);`: 返回int的一元函数；
- `ToIntBiFunction：int applyAsInt(T t, U u);`: 返回int的二元函数；
- `ToLongFunction：long applyAsLong(T value);`: 返回long的一元函数；
- `ToLongBiFunction：long applyAsLong(T t, U u);`: 返回long的二元函数；
- `DoubleToIntFunction：int applyAsInt(double value);`: 接受double返回int的一元函数；
- `DoubleToLongFunction：long applyAsLong(double value);`: 接受double返回long的一元函数；
- `IntToDoubleFunction：double applyAsDouble(int value);`: 接受int返回double的一元函数；
- `IntToLongFunction：long applyAsLong(int value);`: 接受int返回long的一元函数；
- `LongToDoubleFunction：double applyAsDouble(long value);`: 接受long返回double的一元函数；
- `LongToIntFunction：int applyAsInt(long value);`: 接受long返回int的一元函数；



### 2、Consumer（消费型接口）

Consumer 接口表示执行在单个参数上的操作。源代码及使用示例如下：

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

更多的 Consumer 接口：

- `BiConsumer：void accept(T t, U u);`: 接受两个参数的二元函数
- `DoubleConsumer：void accept(double value);`: 接受一个double参数的一元函数
- `IntConsumer：void accept(int value);`: 接受一个int参数的一元函数
- `LongConsumer：void accept(long value);`: 接受一个long参数的一元函数
- `ObjDoubleConsumer：void accept(T t, double value);`: 接受一个泛型参数一个double参数的二元函数
- `ObjIntConsumer：void accept(T t, int value);`: 接受一个泛型参数一个int参数的二元函数
- `ObjLongConsumer：void accept(T t, long value);`: 接受一个泛型参数一个long参数的二元函数



### 3、Supplier（供应型接口）

Supplier 接口是不需要参数并返回一个任意范型的值。其简洁的声明，会让人以为不是函数。这个抽象方法的声明，同 Consumer 相反，是一个只声明了返回值，不需要参数的函数。也就是说 Supplier 其实表达的不是从一个参数空间到结果空间的映射能力，而是表达一种生成能力，因为我们常见的场景中不止是要consume（Consumer）或者是简单的map（Function），还包括了 new 这个动作。而 Supplier 就表达了这种能力。源代码及使用示例如下：

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

更多 Supplier 接口

- `BooleanSupplier：boolean getAsBoolean();`: 返回boolean的无参函数
- `DoubleSupplier：double getAsDouble();`: 返回double的无参函数
- `IntSupplier：int getAsInt();`: 返回int的无参函数
- `LongSupplier：long getAsLong();`: 返回long的无参函数



### 4、Predicate（断言型接口）

Predicate 接口只有一个参数，返回 boolean 类型。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：**与**，**或**，**非**）。Stream 的 filter 方法就是接受 Predicate 作为入参的。这个具体在后面使用 Stream 的时候再分析深入。源代码及使用示例如下：

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

```java
Predicate<String> predicate = (s) -> s.length() > 0;
predicate.test("foo");            // true
predicate.negate().test("foo");     // false
Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

更多的 Predicate 接口

- `BiPredicate：boolean test(T t, U u);`: 接受两个参数的二元断言函数
- `DoublePredicate：boolean test(double value);`: 入参为double的断言函数
- `IntPredicate：boolean test(int value);`: 入参为int的断言函数
- `LongPredicate：boolean test(long value);`: 入参为long的断言函数



### 5、Comparator（比较器接口）

Comparator 是老 Java中 的经典接口， Java 8 在此之上添加了多种默认方法。源代码及使用示例如下：

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

```java
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);
Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");
comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```



### 6、Operator（函数的补充）

Operator 其实就是 Function，函数有时候也叫作算子。算子在 Java 8 中接口描述更像是函数的补充，和上面的很多类型映射型函数类似。算子 Operator 包括：UnaryOperator 和 BinaryOperator。分别对应单（一）元算子和二元算子。

算子的接口声明如下：

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {

    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

Operator 只需声明一个泛型参数 T 即可。对应的使用示例如下：

```java
UnaryOperator<Integer> increment = x -> x + 1;
System.out.println("递增:" + increment.apply(2)); // 输出 递增:3

BinaryOperator<Integer> add = (x, y) -> x + y;
System.out.println("相加:" + add.apply(2, 3)); // 输出 相加:5

BinaryOperator<Integer> min = BinaryOperator.minBy((o1, o2) -> o1 - o2);
System.out.println("最小值:" + min.apply(2, 3)); // 输出 最小值:2
```

更多的 Operator 接口

- `LongUnaryOperator：long applyAsLong(long operand);`: 对long类型做操作的一元算子
- `IntUnaryOperator：int applyAsInt(int operand);`: 对int类型做操作的一元算子
- `DoubleUnaryOperator：double applyAsDouble(double operand);`: 对double类型做操作的一元算子
- `DoubleBinaryOperator：double applyAsDouble(double left, double right);`: 对double类型做操作的二元算子
- `IntBinaryOperator：int applyAsInt(int left, int right);`: 对int类型做操作的二元算子
- `LongBinaryOperator：long applyAsLong(long left, long right);`: 对long类型做操作的二元算子



### 7、其他函数式接口

- `java.lang.Runnable`
- `java.util.concurrent.Callable`
- `java.security.PrivilegedAction`
- `java.io.FileFilter`
- `java.nio.file.PathMatcher`
- `java.lang.reflect.InvocationHandler`
- `java.beans.PropertyChangeListener`
- `java.awt.event.ActionListener`
- `javax.swing.event.ChangeListener`



### 8、预定义函数式接口总结

| 接口                 | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| BiConsumer           | 代表了一个接受两个输入参数的操作，并且不返回任何结果         |
| BiFunction           | 代表了一个接受两个输入参数的方法，并且返回一个结果           |
| BinaryOperator       | 代表了一个作用于于两个同类型操作符的操作，并且返回了操作符同类型的结果 |
| BiPredicate          | 代表了一个两个参数的 boolean 值方法                          |
| BooleanSupplier      | 代表了 boolean 值结果的提供方                                |
| Consumer             | 代表了接受一个输入参数并且无返回的操作                       |
| DoubleBinaryOperator | 代表了作用于两个 double 值操作符的操作，并且返回了一个 double 值的结果。 |
| DoubleConsumer       | 代表一个接受 double 值参数的操作，并且不返回结果。           |
| DoubleFunction       | 代表接受一个 double 值参数的方法，并且返回结果               |
| DoublePredicate      | 代表一个拥有 double 值参数的 boolean 值方法                  |
| DoubleSupplier       | 代表一个 double 值结构的提供方                               |
| DoubleToIntFunction  | 接受一个 double 类型输入，返回一个 int 类型结果。            |
| DoubleToLongFunction | 接受一个 double 类型输入，返回一个 long 类型结果             |
| DoubleUnaryOperator  | 接受一个参数同为类型 double, 返回值类型也为 double 。        |
| Function             | 接受一个输入参数，返回一个结果。                             |
| IntBinaryOperator    | 接受两个参数同为类型 int, 返回值类型也为 int 。              |
| IntConsumer          | 接受一个 int 类型的输入参数，无返回值 。                     |
| IntFunction          | 接受一个 int 类型输入参数，返回一个结果 。                   |
| IntPredicate         | 接受一个 int 输入参数，返回一个布尔值的结果。                |
| IntSupplier          | 无参数，返回一个 int 类型结果。                              |
| IntToDoubleFunction  | 接受一个 int 类型输入，返回一个 double 类型结果 。           |
| IntToLongFunction    | 接受一个 int 类型输入，返回一个 long 类型结果。              |
| IntUnaryOperator     | 接受一个参数同为类型 int, 返回值类型也为 int 。              |
| LongBinaryOperator   | 接受两个参数同为类型 long, 返回值类型也为 long。             |
| LongConsumer         | 接受一个 long 类型的输入参数，无返回值。                     |
| LongFunction         | 接受一个 long 类型输入参数，返回一个结果。                   |
| LongPredicate        | 接受一个 long 输入参数，返回一个布尔值类型结果。             |
| LongSupplier         | 无参数，返回一个结果 long 类型的值。                         |
| LongToDoubleFunction | 接受一个 long 类型输入，返回一个 double 类型结果。           |
| LongToIntFunction    | 接受一个 long 类型输入，返回一个 int 类型结果。              |
| LongUnaryOperator    | 接受一个参数同为类型 long, 返回值类型也为 long。             |
| ObjDoubleConsumer    | 接受一个 object 类型和一个 double 类型的输入参数，无返回值。 |
| ObjIntConsumer       | 接受一个 object 类型和一个 int 类型的输入参数，无返回值。    |
| ObjLongConsumer      | 接受一个 object 类型和一个 long 类型的输入参数，无返回值。   |
| Predicate            | 接受一个输入参数，返回一个布尔值结果。                       |
| Supplier             | 无参数，返回一个结果。                                       |
| ToDoubleBiFunction   | 接受两个输入参数，返回一个 double 类型结果                   |
| ToDoubleFunction     | 接受一个输入参数，返回一个 double 类型结果                   |
| ToIntBiFunction      | 接受两个输入参数，返回一个 int 类型结果。                    |
| ToIntFunction        | 接受一个输入参数，返回一个 int 类型结果。                    |
| ToLongBiFunction     | 接受两个输入参数，返回一个 long 类型结果。                   |
| ToLongFunction       | 接受一个输入参数，返回一个 long 类型结果。                   |
| UnaryOperator        | 接受一个参数为类型 T, 返回值类型也为 T。                     |



## 5、重复式注解

自从 Java 5 引入了注解机制，这一特性就变得非常流行并且广为使用。然而，使用注解的一个限制是相同的注解在同一位置只能声明一次，不能声明多次。Java 8 打破了这条规则，引入了重复注解机制，这样相同的注解可以在同一地方声明多次。

重复注解机制本身必须用 @Repeatable 注解。事实上，这并不是语言层面上的改变，更多的是编译器的技巧，底层的原理保持不变。让我们看一个快速入门的例子：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

public class RepeatingAnnotations {

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Filters {
        Filter[] value();
    }

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Repeatable(Filters.class)
    public @interface Filter {
        String value();
    };

    @Filter("filter1")
    @Filter("filter2")
    public interface Filterable {
    }

    public static void main(String[] args) {
        for(Filter filter: Filterable.class.getAnnotationsByType(Filter.class)) {
            System.out.println(filter.value());
        }
    }
}
```

```
filter1
filter2
```

正如我们看到的，这里有个使用 @Repeatable(Filters.class) 注解的注解类 Filter，Filters 仅仅是 Filter 注解的数组，但 Java 编译器并不想让程序员意识到 Filters 的存在。这样，接口 Filterable 就拥有了两次 Filter（并没有提到Filters）注解。

同时反射相关的API提供了新的函数getAnnotationsByType()来返回重复注解的类型（请注意Filterable.class.getAnnotation(Filters.class)经编译器处理后将会返回Filters的实例）。



## 6、扩展注解的支持

Java 8 扩展了注解的上下文。**现在几乎可以为任何东西添加注解：局部变量、泛型类、父类与接口的实现，就连方法的异常也能添加注解**。下面演示几个例子：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.Collection;

public class Annotations {
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ ElementType.TYPE_USE, ElementType.TYPE_PARAMETER })
    public @interface NonEmpty {
    }

    public static class Holder<@NonEmpty T> extends @NonEmpty Object {
        public void method() throws @NonEmpty Exception {
        }
    }

    @SuppressWarnings("unused")
    public static void main(String[] args) {
        final Holder<String> holder = new @NonEmpty Holder<String>();
        @NonEmpty Collection<@NonEmpty String> strings = new ArrayList<>();
    }
}
```

**ElementType.TYPE_USER** 和 **ElementType.TYPE_PARAMETER** 是 Java 8 新增的两个注解，用于描述注解的使用场景。Java 语言也做了对应的改变，以识别这些新增的注解。



## 7、更好的类型推测机制

Java 8 在类型推测方面有了很大的提高。在很多情况下，编译器可以推测出确定的参数类型，这样就能使代码更整洁。看如下例子：

```java
public class Value<T> {
    public static<T> T defaultValue() {
        return null;
    }

    public T getOrDefault(T value, T defaultValue) {
        return (value != null) ? value : defaultValue;
    }
}
```

这里是`Value<String>`类型的用法。

```java
public class TypeInference {
    public static void main(String[] args) {
        final Value<String> value = new Value<>();
        value.getOrDefault("22", Value.defaultValue());
        // Java 7 表示如下
        // value.getOrDefault("22", Value.<String>defaultValue());
    }
}
```

Value.defaultValue() 的参数类型可以被推测出，所以就不必明确给出。在 Java 7 中，相同的例子将不会通过编译，正确的书写方式是：

```java
value.getOrDefault("22", Value.<String>defaultValue());
```



## 8、forEach 方法

Java 提供了一个新的 forEach() 方法来迭代元素。它在 Iterable 和 Stream 接口中定义。这是 Iterable 接口中定义的默认方法。扩展 Iterable 接口的集合类可以使用forEach循环来迭代元素。

此方法采用单个参数，该参数是功能接口。因此，您可以将 Lambda 表达式作为参数传递。

```java
default void forEach(Consumer<super T>action);
```

【操作示例 1】

```java
import java.util.ArrayList;  
import java.util.List;  
public class ForEachExample {  
    public static void main(String[] args) {  
        List<String> gamesList = new ArrayList<String>();  
        gamesList.add("Football");  
        gamesList.add("Cricket");  
        gamesList.add("Chess");  
        gamesList.add("Hocky");  
        gamesList.forEach(games -> System.out.println(games));  
    }  
}
```

```java
Football
Cricket
Chess
Hocky
```

【操作示例 2】

```java
import java.util.ArrayList;  
import java.util.List;  
public class ForEachExample {  
    public static void main(String[] args) {  
        List<String> gamesList = new ArrayList<String>();  
        gamesList.add("Football");  
        gamesList.add("Cricket");  
        gamesList.add("Chess");  
        gamesList.add("Hocky");  
        gamesList.forEach(System.out::println);  
    }  
}
```

```
Football
Cricket
Chess
Hocky
```



# 二、工具库类新特性

## 1、Base64

### 1、Base64 编解码介绍

Java 提供了一个 Base64 类来处理加密。您可以使用提供的方法对数据进行加密和解密。您需要在源文件中导入java.util.Base64才能使用其方法。此类提供了三种不同的编码器和解码器，用于在每个级别上加密信息。您可以在以下级别使用这些方法。

1、基本编码和解码：Base64这个类使用Java在RFC 4648和RFC 2045中指定的Base64字母表进行编码和解码操作。编码器不添加任何行分隔符。解码器不允许包含base64字母之外的字符的数据。

2、URL 和文件名的编码解码：Base64这个类使用Java在RFC 4648中指定的Base64字母表进行编码和解码操作。编码器不添加任何行分隔符。解码器不允许包含base64字母之外的字符的数据。

3、MIME：Base64 这个类使用 RFC 2045 中指定的 Base64 字母表进行编码和解码操作。编码后的输出必须以不超过 76 个字符的行表示，并使用回车符'\r'和紧跟换行符'\n'作为行分隔符。没有行分隔符添加到编码的输出的末尾。在base64字母表中找不到的所有行分隔符或其他字符在解码操作中将被忽略。

在 Java 8 中，Base64 编码已经成为 Java 类库的标准。它的使用十分简单，无⽤引包，编解码效率远⼤于 sun.misc 和 Apache Commons Codec。



### 2、Base64 类方法介绍

1、Base64 的内部类

| 类名           | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| Base64.Decoder | 此类实现了一个解码器，用于使用RFC 4648和RFC 2045中指定的Base64编码方案来解码字节数据。 |
| Base64.Encoder | 此类实现了一个编码器，用于使用RFC 4648和RFC 2045中指定的Base64编码方案对字节数据进行编码。 |

2、Base64 的方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| public static Base64.Decoder getDecoder()                    | 返回一个Base64.Decoder，它使用Basic类型的base64编码方案进行解码。 |
| public static Base64.Encoder getEncoder()                    | 返回一个Base64.Encoder，它使用Basic类型的base64编码方案进行编码。 |
| public static Base64.Decoder getUrlDecoder()                 | 返回一个Base64.Decoder，该URL使用URL和Filename安全类型base64编码方案进行解码。 |
| public static Base64.Decoder getMimeDecoder()                | 返回一个使用MIME类型base64解码方案进行解码的Base64.Decoder。 |
| public static Base64.Encoder getMimeEncoder()                | 返回一个使用MIME类型base64编码方案进行编码的Base64.Encoder。 |
| public static Base64.Encoder getMimeEncoder(int lineLength, byte[] lineSeparator) | 返回一个Base64.Encoder，它使用具有指定行长和行分隔符的MIME类型base64编码方案进行编码。 |
| public static Base64.Encoder getUrlEncoder()                 | 返回一个Base64.Encoder，它使用URL和Filename安全类型base64编码方案进行编码。 |

3、Base64.Decoder 的方法

| 方法                                        | 描述                                                         |
| :------------------------------------------ | :----------------------------------------------------------- |
| public byte[] decode(byte[] src)            | 使用Base64编码方案解码输入字节数组中的所有字节，并将结果写入新分配的输出字节数组中。返回的字节数组具有所得字节的长度。 |
| public byte[] decode(String src)            | 使用Base64编码方案将Base64编码的String解码为新分配的字节数组。 |
| public int decode(byte[] src, byte[] dst)   | 使用Base64编码方案对输入字节数组中的所有字节进行解码，并将结果从偏移量0开始写入给定的输出字节数组。 |
| public ByteBuffer decode(ByteBuffer buffer) | 使用Base64编码方案对输入字节缓冲区中的所有字节进行解码，并将结果写入新分配的ByteBuffer中。 |
| public InputStream wrap(InputStream is)     | 返回用于解码Base64编码的字节流的输入流。                     |

4、Base64.Encoder 的方法

| 方法                                        | 描述                                                         |
| :------------------------------------------ | :----------------------------------------------------------- |
| public byte[] encode(byte[] src)            | 使用Base64编码方案将指定字节数组中的所有字节编码为新分配的字节数组。返回的字节数组具有所得字节的长度。 |
| public int encode(byte[] src, byte[] dst)   | 使用Base64编码方案对指定字节数组中的所有字节进行编码，并将结果字节从偏移量0开始写入给定的输出字节数组。 |
| public String encodeToString(byte[] src)    | 使用Base64编码方案将指定的字节数组编码为String。             |
| public ByteBuffer encode(ByteBuffer buffer) | 使用Base64编码方案将指定字节缓冲区中的所有剩余字节编码为新分配的ByteBuffer。返回时，源缓冲区的位置将更新到其极限；其限制将不会更改。返回的输出缓冲区的位置将为零，其限制将为生成的编码字节数。 |
| public OutputStream wrap(OutputStream os)   | 包装输出流，以使用Base64编码方案对字节数据进行编码。         |
| public Base64.Encoder withoutPadding()      | 返回一个编码器实例，该实例与该实例等效，但在编码的字节数据的末尾不添加任何填充字符。 |



### 3、Base64 的实操案例

1、基本编码和解码

```java
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class Base64s {
    public static void main(String[] args) {
        final String text = "Base64 finally in Java 8!";

        final String encoded = Base64.getEncoder().encodeToString(text.getBytes(StandardCharsets.UTF_8));
        System.out.println(encoded);

        final String decoded = new String(Base64.getDecoder().decode(encoded), StandardCharsets.UTF_8);
        System.out.println(decoded);
    }
}
```

```
QmFzZTY0IGZpbmFsbHkgaW4gSmF2YSA4IQ==
Base64 finally in Java 8!
```

2、URL 编码和解码

```java
import java.util.Base64;  
public class Base64BasicEncryptionExample {
    public static void main(String[] args) {
        // Getting encoder
        Base64.Encoder encoder = Base64.getUrlEncoder();  
        // Encoding URL
        String eStr = encoder.encodeToString("https://baidu.com".getBytes());
        System.out.println("Encoded URL: "+eStr);
        // Getting decoder
        Base64.Decoder decoder = Base64.getUrlDecoder();  
        // Decoding URl
        String dStr = new String(decoder.decode(eStr));  
        System.out.println("Decoded URL: "+dStr);  
    }  
}
```

```
Encoded URL: aHR0cHM6Ly9iYWlkdS5jb20=
Decoded URL: https://baidu.com
```

3、MIME 编码和解码

```java
import java.util.Base64;

public class Base64BasicEncryptionExample {
    public static void main(String[] args) {
        // Getting MIME encoder  
        Base64.Encoder encoder = Base64.getMimeEncoder();
        String message = "Hello, \nYou are informed regarding your inconsistency of work";
        String eStr    = encoder.encodeToString(message.getBytes());
        System.out.println("Encoded MIME message: " + eStr);

        // Getting MIME decoder  
        Base64.Decoder decoder = Base64.getMimeDecoder();
        // Decoding MIME encoded message  
        String dStr = new String(decoder.decode(eStr));
        System.out.println("Decoded message: " + dStr);
    }
}  
```

```
Encoded MIME message: SGVsbG8sIApZb3UgYXJlIGluZm9ybWVkIHJlZ2FyZGluZyB5b3VyIGluY29uc2lzdGVuY3kgb2Yg
d29yaw==
Decoded message: Hello, 
You are informed regarding your inconsistency of work
```



## 2、Stream

> 请参考本人教程：Java 8 新特性之 Stream 流

## 3、Optional

> 请参考本人教程：Java 8 新特性之 Optional 类

## 4、Date/Time API（JSR 310）

> 请参考本人教程：Java 8 新特性之 Date/Time API（JSR 310）



## 5、StringJoiner 类

Java 8 在 java.util 包中添加了一个新的最终类 StringJoiner。它用于构造由定界符分隔的字符序列。现在，我们可以通过传递诸如：comma（，），hyphen（-）等分隔符来创建字符串。还可以将前缀和后缀传递给 char 序列。

1、StringJoiner 类的构造方法：

| 构造方法                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| public StringJoiner(CharSequence delimiter)                  | 构造一个不带任何字符，没有前缀或后缀的StringJoiner，以及提供的定界符的副本。如果分隔符为null，则抛出NullPointerException。 |
| public StringJoiner(CharSequence delimiter,CharSequence prefix,CharSequence suffix) | 使用提供的前缀，定界符和后缀的副本构造一个不带任何字符的StringJoiner。如果前缀，分隔符或后缀为null，则抛出NullPointerException。 |

2、StringJoiner 类的成员方法：

| 方法                                                       | 描述                                                         |
| :--------------------------------------------------------- | :----------------------------------------------------------- |
| public StringJoiner add(CharSequence newElement)           | 将给定CharSequence值的副本添加为StringJoiner值的下一个元素。如果newElement为null，则添加“ null”。 |
| public StringJoiner merge(StringJoiner other)              | 如果它不是空的，它将添加不带前缀和后缀的给定StringJoiner的内容作为下一个元素。如果给定的StringJoiner为空，则该调用无效。 |
| public int length()                                        | 返回此StringJoiner的String表示形式的长度。                   |
| public StringJoiner setEmptyValue(CharSequence emptyValue) | 设置确定此StringJoiner的字符串表示形式时要使用的字符序列，并且尚未添加任何元素（即，当它为空时）。 |

【操作示例 1】增加分隔符

```java
import java.util.StringJoiner;
public class StringJoinerExample {
    public static void main(String[] args) {
        // passing comma(,) as delimiter
        StringJoiner joinNames = new StringJoiner(",");
        
        // Adding values to StringJoiner
        joinNames.add("Rahul");
        joinNames.add("Raju");
        joinNames.add("Peter");
        joinNames.add("Raheem");
        
        System.out.println(joinNames);
        // Rahul,Raju,Peter,Raheem
    }
}
```

【操作示例 2】增加分隔符，并且添加前缀和后缀

```java
import java.util.StringJoiner;
public class StringJoinerExample {
    public static void main(String[] args) {
        // passing comma(,) and square-brackets as delimiter
        StringJoiner joinNames = new StringJoiner(",", "[", "]");
        
        // Adding values to StringJoiner
        joinNames.add("Rahul");
        joinNames.add("Raju");
        joinNames.add("Peter");
        joinNames.add("Raheem");
        
        System.out.println(joinNames);
        // [Rahul,Raju,Peter,Raheem]
    }
}  
```

【操作示例 3】合并两个 StringJoiner，merge() 方法合并两个StringJoiner对象，但不包括第二个StringJoiner对象的前缀和后缀。

```java
import java.util.StringJoiner;
public class StringJoinerExample {
    public static void main(String[] args) {
        // passing comma(,) and square-brackets as delimiter
        StringJoiner joinNames = new StringJoiner(",", "[", "]");

        // Adding values to StringJoiner
        joinNames.add("Rahul");
        joinNames.add("Raju");

        // Creating StringJoiner with :(colon) delimiter
        // passing colon(:) and square-brackets as delimiter
        StringJoiner joinNames2 = new StringJoiner(":", "[", "]");

        // Adding values to StringJoiner
        joinNames2.add("Peter");
        joinNames2.add("Raheem");

        // Merging two StringJoiner
        StringJoiner merge = joinNames.merge(joinNames2);
        System.out.println(merge);
        // [Rahul,Raju,Peter:Raheem]
    }
}
```

【操作示例 4】StringJoiner 方法总结

```java
import java.util.StringJoiner;

public class StringJoinerExample {
    public static void main(String[] args) {
        // passing comma(,) as delimiter
        StringJoiner joinNames = new StringJoiner(",");

        // Prints nothing because it is empty
        System.out.println(joinNames);

        // We can set default empty value.
        joinNames.setEmptyValue("It is empty");
        System.out.println(joinNames);


        // Adding values to StringJoiner
        joinNames.add("Rahul");
        joinNames.add("Raju");
        System.out.println(joinNames);

        // Returns length of StringJoiner
        int length = joinNames.length();
        System.out.println("Length: " + length);

        // Returns StringJoiner as String type
        String str = joinNames.toString();
        System.out.println(str);

        // Now, we can apply String methods on it
        char ch = str.charAt(3);
        System.out.println("Character at index 3: " + ch);

        // Adding one more element
        joinNames.add("Sorabh");
        System.out.println(joinNames);

        // Returns length
        int newLength = joinNames.length();
        System.out.println("New Length: " + newLength);
    }
}
```

```
It is empty
Rahul,Raju
Length: 10
Rahul,Raju
Character at index 3: u
Rahul,Raju,Sorabh
New Length: 17
```



## 6、并行数组

Java 8 版本新增了很多新的方法，用于支持并行数组处理。最重要的方法是 parallelSort() ，可以显著加快多核机器上的数组排序。

Java8 并行数组排序的方法：

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| public static void parallelSort(byte[] a)                    | 将指定的数组按升序排列。                                     |
| public static void parallelSort(byte[] a, int fromIndex, int toIndex) | 将指定范围的数组按升序排列。要排序的范围从索引fromIndex（含）到索引toIndex（不包含）。如果fromIndex == toIndex，则要排序的范围为空。 |
| public static void parallelSort(char[] a)                    | 将指定的数组按升序排列。                                     |
| public static void parallelSort(char[] a, int fromIndex, int toIndex) | 将指定范围的数组按升序排列。要排序的范围从索引fromIndex（含）到索引toIndex（不包含）。如果fromIndex == toIndex，则要排序的范围为空。 |
| public static void parallelSort(double[] a)                  | 将指定的数组按升序排列。                                     |
| public static void parallelSort(double[] a, int fromIndex, int toIndex) | 将指定范围的数组按升序排列。要排序的范围从索引fromIndex（含）到索引toIndex（不包含）。如果fromIndex == toIndex，则要排序的范围为空。 |
| public static void parallelSort(float[] a)                   | 将指定的数组按升序排列。                                     |
| public static void parallelSort(float[] a, int fromIndex, int toIndex) | 将指定范围的数组按升序排列。要排序的范围从索引fromIndex（含）到索引toIndex（不包含）。如果fromIndex == toIndex，则要排序的范围为空。 |
| public static void parallelSort(int[] a)                     | 将指定的数组按升序排列。                                     |
| public static void parallelSort(int[] a,int fromIndex, int toIndex) | 将指定范围的数组按升序排列。要排序的范围从索引fromIndex（含）到索引toIndex（不包含）。如果fromIndex == toIndex，则要排序的范围为空。 |
| public static void parallelSort(long[] a)                    | 将指定的数组按升序排列。                                     |
| public static void parallelSort(long[] a, int fromIndex, int toIndex) | 将指定范围的数组按升序排列。要排序的范围从索引fromIndex（含）到索引toIndex（不包含）。如果fromIndex == toIndex，则要排序的范围为空。 |
| public static void parallelSort(short[] a)                   | 将指定的数组按升序排列。                                     |
| public static void parallelSort(short[] a,int fromIndex,int toIndex) | 将指定范围的数组按升序排列。要排序的范围从索引fromIndex（含）到索引toIndex（不包含）。如果fromIndex == toIndex，则要排序的范围为空。 |
| public static <T extends Comparable<? super T>> void parallelSort(T[] a) | 根据对象的自然顺序，将指定对象数组按升序排序。数组中的所有元素必须实现Comparable接口。此外，数组中的所有元素都必须相互可比（即e1.compareTo（e2）不得对数组中的任何元素e1和e2抛出ClassCastException）。 |
| public static <T7gt; void parallelSort(T[] a,Comparator<? super T> cmp) | 根据指定的比较器引发的顺序对指定的对象数组进行排序。数组中的所有元素必须可以通过指定的比较器相互比较（即c.compare（e1，e2）不得对数组中的任何元素e1和e2抛出ClassCastException。 |
| public static <T extends Comparable<? super T>> void parallelSort(T[] a,int fromIndex, int toIndex) | 根据其元素的自然顺序，它将指定对象数组的指定范围按升序排序。要排序的范围从索引fromIndex（包括）到索引toIndex（不包括）。（如果fromIndex == toIndex，则要排序的范围为空。）此范围中的所有元素必须实现Comparable接口。此外，此范围内的所有元素都必须相互可比较（即e1.compareTo（e2）不得对数组中的任何元素e1和e2抛出ClassCastException）。 |
| public static <T> void parallelSort(T[] a, int fromIndex, int toIndex, Comparator<? super T> cmp) | 根据由指定比较器引起的顺序对指定对象数组的指定范围进行排序。要排序的范围从索引fromIndex（包括）到索引toIndex（不包括）。（如果fromIndex == toIndex，则要排序的范围为空。）该范围内的所有元素必须可以通过指定的比较器相互比较（即c.compare（e1，e2）不得对任何元素e1抛出ClassCastException和范围内的e2）。 |

【操作示例】下面我们使用 parallelSetAll() 方法生成20000个随机数，然后使用 parallelSort() 方法进行排序。这个程序会输出乱序数组和排序数组的前10个元素。

```java
import java.util.Arrays;
import java.util.concurrent.ThreadLocalRandom;

public class ParallelArrays {
    public static void main(String[] args) {
        long[] arrayOfLong = new long[20000];

        Arrays.parallelSetAll(arrayOfLong, index -> ThreadLocalRandom.current().nextInt(1000000));
        Arrays.stream(arrayOfLong).limit(10).forEach(i -> System.out.print(i + " "));
        System.out.println();

        Arrays.parallelSort(arrayOfLong);
        Arrays.stream(arrayOfLong).limit(10).forEach(i -> System.out.print(i + " "));
        System.out.println();
    }
}
```

```
Unsorted: 591217 891976 443951 424479 766825 351964 242997 642839 119108 552378 
Sorted: 39 220 263 268 325 607 655 678 723 793
```



## 7、并发性（Concurrency）

在新增 Stream 机制与 Lambda 的基础之上，在 java.util.concurrent.ConcurrentHashMap 中加入了一些新方法来支持聚集操作。同时也在 java.util.concurrent.ForkJoinPool 类中加入了一些新方法来支持共有资源池（common pool）。

新增的 java.util.concurrent.locks.StampedLock 类提供一直基于容量的锁，这种锁有三个模型来控制读写操作（它被认为是不太有名的 java.util.concurrent.locks.ReadWriteLock 类的替代者）。

在 java.util.concurrent.atomic 包中还增加了下面这些类：DoubleAccumulator、DoubleAdder、LongAccumulator、LongAdder。



## 8、Nashorn JavaScript 引擎

Java 8 提供了一个新的 Nashorn Javascript 引擎，它允许我们在 JVM 上运行特定的 Javascript 应用。Nashorn Javascript 引擎只是 javax.script.ScriptEngine 另一个实现，而且规则也一样，允许 Java 和 JavaScript 互相操作。示例代码如下：

```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("JavaScript");

System.out.println(engine.getClass().getName());
System.out.println("Result:" + engine.eval("function f(){return 1;}; f() + 1;"));
```

```
jdk.nashorn.api.scripting.NashornScriptEngine
Result: 2
```



## 9、JavaFX 归化

JavaFX 是一个强大的图形和多媒体处理工具包集合，它允许开发者来设计、创建、测试、调试和部署富客户端程序，并且和 Java 一样跨平台。从 Java 8 开始，JavaFx 已经内置到了 JDK 中。关于 JavaFx 更详细的文档可参考 JavaFX 中文文档。



# 三、命令行工具新特性

## 1、Nashorn引擎：jjs

jjs 是一个基于标准 Nashorn 引擎的命令行工具，可以接受 js 源码并执行。例如，我们写一个 func.js 文件，内容如下：

```javascript
function f() { 
    return 1; 
}; 

print( f() + 1 );
```

可以在命令行中执行这个命令：jjs func.js，控制台输出结果是：

```
2
```

更多的细节可以参考：[官方文档](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/jdeps.html)。



## 2、类依赖分析器：jdeps

Jdep s是一个功能强大的命令行工具，它可以帮我们显示出包层级或者类层级java类文件的依赖关系。它接受 class 文件、目录、jar 文件作为输入，默认情况下，jdeps 会输出到控制台。

我们可以分析下 Spring Framework 库，为了让结果少一点，仅仅分析一个JAR文件：org.springframework.core-3.0.5.RELEASE.jar：

```bash
jdeps org.springframework.core-3.0.5.RELEASE.jar
```

这个命令会输出很多结果，我们仅看下其中的一部分：依赖关系按照包分组，如果在classpath上找不到依赖，则显示"not found".

```bash
org.springframework.core-3.0.5.RELEASE.jar -> C:\Program Files\Java\jdk1.8.0\jre\lib\rt.jar
   org.springframework.core (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.io
      -> java.lang
      -> java.lang.annotation
      -> java.lang.ref
      -> java.lang.reflect
      -> java.util
      -> java.util.concurrent
      -> org.apache.commons.logging                         not found
      -> org.springframework.asm                            not found
      -> org.springframework.asm.commons                    not found
   org.springframework.core.annotation (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.lang
      -> java.lang.annotation
      -> java.lang.reflect
      -> java.util
```

更多的细节可以参考：[官方文档](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/jdeps.html)。



# 四、编译器相关特性

## 1、获取方法的参数名称

为了在运行时获得 Java 程序中方法的参数名称，老一辈的 Java 程序员必须使用不同方法，例如：Paranamer liberary。Java 8 终于将这个特性规范化，在语言层面（使用反射 API 和 Parameter.getName() 方法）和字节码层面（使用新的 javac 编译器以及**-parameters**参数）提供支持。

```java
import java.lang.reflect.Method;
import java.lang.reflect.Parameter;

public class ParameterNames {
    public static void main(String[] args) throws Exception {
        Method method = ParameterNames.class.getMethod( "main", String[].class );
        for( final Parameter parameter: method.getParameters() ) {
            System.out.println( "Parameter: " + parameter.getName() );
        }
    }
}
```

在 Java 8 中这个特性是默认关闭的，因此如果不带 **-parameters** 参数编译上述代码并运行，则会输出如下结果：

```
Parameter: arg0
```

如果带 **-parameters** 参数，则会输出如下结果（正确的结果）：

```
Parameter: args
```

如果你使用Maven进行项目管理，则可以在**maven-compiler-plugin**编译器的配置项中配置**-parameters**参数：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <compilerArgument>-parameters</compilerArgument>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```



# 五、JVM 的新特性

PermGen 空间被移除了，取而代之的是 Metaspace（JEP 122）。

在JVM参数方面，使用 -XX:MetaSpaceSize和-XX:MaxMetaspaceSize 代替原来的 -XX:PermSize 和 -XX:MaxPermSize。

区别：

1. 元空间并不在虚拟机中，而是使用本地内存
2. 默认情况下，元空间的大小仅受本地内存限制
3. 也可以通过-XX：MetaspaceSize指定元空间大小



# 六、其他的新特性

## 1、JDBC4.2 规范

### 1、JDBC 改进

在 Java 8 中，Java 在 JDBC API 中进行了两项重大更改：

1、JDBC-ODBC桥已被删除。

- Oracle不支持JDBC-ODBC桥。Oracle建议您使用数据库供应商提供的JDBC驱动程序，而不要使用JDBC-ODBC Bridge。

2、在 JDBC 4.2 中添加了一些新功能，Java JDBC 4.2引入了以下功能：：

- 增加了对 REF Cursor 的支持
- 修改返回值大小范围（update count）
- 增加了 java.sql.DriverAction 接口
- 增加了 java.sql.SQLType 接口
- 增加了 java.sql.JDBCtype 枚举
- 增加了对 java.time 包时间类型的支持



### 3、JDBC 驱动程序操作

当要通过 DriverManager 通知驱动程序时，必须实现此接口。它添加在 java.sql 包中，仅包含一个抽象方法

1、DriverAction方法

| 方法              | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| void deregister() | DriverManager.deregisterDriver（Driver）调用此方法以通知JDBC驱动程序已注销。 |

- 取消注册方法仅供JDBC驱动程序使用，而不能由应用程序使用。
- 建议不要使用JDBC驱动程序在公共类中实现DriverAction。
- 如果在调用注销方法时存在到数据库的活动连接，则具体取决于连接是关闭还是允许继续。调用此方法后，驱动程序是否可能会限制创建与数据库的新连接，调用其他Driver方法或引发SQLException的能力，这取决于实现。

2、JDBC4.2 DriverAction 操作示例

```java
import java.sql.*;

class JdbcExample implements DriverAction {
    // implementing deregister method of DriverAction interface  
    @Override
    public void deregister() {
        System.out.println("Driver deregistered");
    }

    public static void main(String args[]) {
        try {
            // Creating driver instance  
            Driver driver = new com.mysql.jdbc.Driver();
            // Creating Action Driver  
            DriverAction da = new JdbcExample();
            // Registering driver by passing driver and driverAction  
            DriverManager.registerDriver(driver, da);
            // Creating connection  
            Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/student", "root", "mysql");
            //Here student is database name, root is username and password is mysql  
            Statement stmt = con.createStatement();
            // Executing SQL query  
            ResultSet rs = stmt.executeQuery("select * from user");
            while (rs.next()) {
                System.out.println(rs.getInt(1) + "" + rs.getString(2) + "" + rs.getString(3));
            }
            // Closing connection  
            con.close();
            // Calling deregisterDriver method  
            DriverManager.deregisterDriver(driver);
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

```
1  Arun  25
2  irfan  22
3  Neraj kumar  25
Driver deregistered
```



## 2、HashMap 性能提升

Java 8 中，HashMap 内部实现又引入了红黑树，使得 HashMap 的总体性能相较于 Java 7 有比较明显的提升。以下是对 Hash 均匀和不均匀的情况下的性能对比

Hash较均匀的情况：

| map size 大小            | 10   | 100  | 1000 | 10000 | 100000 | 1000000 | 10000000 |
| ------------------------ | ---- | ---- | ---- | ----- | ------ | ------- | -------- |
| JDK7 get方法平均时间(ns) | 900  | 540  | 570  | 285   | 55     | 6.9     | 8.1      |
| JDK8 get方法平均时间(ns) | 705  | 400  | 120  | 68    | 15     | 6.25    | 6.8      |

Hash极不均匀的情况：

| map size 大小            | 10   | 100   | 1000 | 10000 | 100000 | 1000000 | 10000000 |
| ------------------------ | ---- | ----- | ---- | ----- | ------ | ------- | -------- |
| JDK7 get方法平均时间(ns) | 2100 | 12960 | 3700 | 21000 | 17200  | 36000   | ---      |
| JDK8 get方法平均时间(ns) | 1960 | 3340  | 1470 | 720   | 190    | 230     | 220      |

想要了解更多 HashMap 请参考：[传送门](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247486169&idx=2&sn=9818c995d51a19cd4a40c2605bdcfa5d&scene=21#wechat_redirect)



## 3、IO/NIO 的改进

Java 8  对 IO/NIO 也做了一些改进。主要包括：改进了 java.nio.charset.Charset 的实现，使编码和解码的效率得以提升，也精简了jre/lib/charsets.jar 包；优化了`String(byte[], *)`构造方法和`String.getBytes()`方法的性能；还增加了一些新的 IO/NIO 方法，使用这些方法可以从文件或者输入流中获取流（java.util.stream.Stream），通过对流的操作，可以简化文本行处理、目录遍历和文件查找。

新增的 API 如下：

- BufferedReader.line()：返回文本行的流 Stream< String>
- File.lines(Path, Charset)：返回文本行的流 Stream< String>
- File.list(Path)：遍历当前目录下的文件和目录
- File.walk(Path, int, FileVisitOption)：遍历某一个目录下的所有文件和指定深度的子目录
- File.find(Path, int, BiPredicate, FileVisitOption...)：查找相应的文件

下面就是用流式操作列出当前目录下的所有文件和目录：

```java
Files.list(new File(".").toPath()).forEach(System.out::println);
```



# 七、参考文献 & 鸣谢

1. 【**史上最全 Java 8 新特性**】Java 8 新特性大全：https://moonce.blog.csdn.net/article/details/111146637
2. 一点教程-Java8教程：http://www.yiidian.com/java8/
3. 全网最通透的Java8版本特性讲解：https://mp.weixin.qq.com/s/muKflQV9Erz857Ym8jjrZA
4. Java 8 新特性-终极指南：https://blog.csdn.net/wang_luwei/article/details/125274881
5. Java 8 新特性：https://www.lmlphp.com/user/8405/article/item/591956/
6. Java 8新特性-Lambda表达式：https://thinkwon.blog.csdn.net/article/details/113764085
7. Java8新特性详解（精编-史上最全版）：https://blog.csdn.net/weixin_45101064/article/details/104444768