> 参考文献：http://www.mydlq.club/article/88/

## 一、Optional 简介

Optional 是一个容器对象，可以存储对象、字符串等值，当然也可以存储 null 值。Optional 提供很多有用的方法，能帮助我们将 Java 中的对象等一些值存入其中，这样我们就不用显式进行空值检测，使我们能够用少量的代码完成复杂的流程。

比如它提供了：

- of() 方法，可以将值存入 Optional 容器中，如果存入的值是 null 则抛异常。
- ofNullable() 方法，可以将值存入 Optional 容器中，即使值是 null 也不会抛异常。
- get() 方法，可以获取容器中的值，如果值为 null 则抛出异常。
- getElse() 方法，可以获取容器中的值，如果值为 null 则返回设置的默认值。
- isPresent() 方法，该方法可以判断存入的值是否为空。
- ...等等一些其它常用方法，下面会进行介绍。

可以说，使用 Optional 可以帮助我们解决业务中，减少值动不动就抛出空指针异常问题，也减少 null 值的判断，提高代码可读性等，这里我们介绍下，如果使用这个 Optional 类。

![img](Java8之Optional类.assets/shuiyin.png)



## 二、Optional 类描述

- **Optional 类所在包：** java.util.Optional
- **Optional 类声明：** public final class Optional extends Object
- **Optional 类方法：**

| 方法名称                                                     | 方法描述                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `static <T> Optional<T> empty()`                             | 返回空的 Optional 实例                                       |
| `static <T> Optional<T> of(T value)`                         | 返回一个指定非 null 值的 Optional，如果为null会抛异常        |
| `static <T> Optional ofNullable(T value)`                    | 如果为非空，返回 Optional 描述的指定值，否则返回空的 Optional |
| T get()                                                      | 如果在这个 Optional 中包含这个值则返回值，否则抛出 NoSuchElementException 异常 |
| `boolean isPresent()`                                        | 如果值存在则方法会返回 true，否则返回 false                  |
| `boolean isEmpty()`                                          | 判断值是否为null                                             |
| `void ifPresent(Consumer<? super T> consumer)`               | 如果值存在则使用该值调用 consumer , 否则不做任何事情         |
| `Optional<T> filter(Predicate<? super T> predicate)`         | 如果有值并且满足断言条件返回包含该值的 Optional，否则返回空 Optional |
| `Optional map(Function<? super T,? extends U> mapper)`       | 如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的 Optional 作为 map 方法返回值，否则返回空 Optional。 |
| `Optional flatMap(Function<? super T,Optional<U>> mapper)`   | 如果值存在，返回基于 Optional 包含的映射方法的值，否则返回一个空的 Optional |
| `T orElse(T other)`                                          | 如果该值存在就直接返回， 否则返回指定的其它值。              |
| `T orElseGet(Supplier<? extends T> other)`                   | 如果该值存在就返回值，否则触发 other，并返回 other 调用的结果 |
| `<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)` | 如果存在该值，返回包含的值，否则抛出由 Supplier 继承的异常   |
| `boolean equals(Object obj)`                                 | 判断其他对象是否等于 Optional                                |
| `int hashCode()`                                             | 返回存在值的哈希码，如果值不存在则返回 0                     |
| `String toString()`                                          | 返回一个 Optional 的非空字符串，用来调试                     |



## 三、Optional 常用方法及使用示例

### 1、静态方法 Optional.of()

- **方法作用：** 为指定的值创建一个指定非 null 值的 Optional
- **方法描述：** of 方法通过工厂方法创建 Optional 实例，需要注意的是传入的参数不能为 null，否则抛出 NullPointerException
- **返回类型：** Optional
- **示例代码：**调用两个 Optional.of() 方法，一个传入正常参数，另一个传入 null 参数：

```java
public static void main(String[] args) {
    // 传入正常值，正常返回一个 Optional 对象
    Optional<String> optional1 = Optional.of("mydlq");

    // 传入参数为 null，抛出 NullPointerException.
    Optional optional2 = Optional.of(null);
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
Exception in thread "main" java.lang.NullPointerException
	at java.util.Objects.requireNonNull(Objects.java:203)
	at java.util.Optional.<init>(Optional.java:96)
	at java.util.Optional.of(Optional.java:108)
	at club.mydlq.OptionalExample.main(OptionalExample.java:12)
```

可以看到传入正常参数正常返回 Optional 对象，传入 null 参数返回 NullPointerException 异常。



### 2、静态方法 Optional.ofNullable()

- **方法作用：** 为指定的值创建一个 Optional 对象，如果指定的参数为 null，不抛出异常，直接则返回一个空的 Optional 对象
- **方法描述：** ofNullable 方法是和 of 方式一样，都是用于创建 Optional 对象，只是传入的参数 null 时，会返回一个空的 Optional 对象，而不会抛出 NullPointerException 异常
- **返回类型：** Optional
- **示例代码：**调用 Optional.ofNullable() 方法，传入 null 参数：

```java
public static void main(String[] args) {
    // 传入正常值，正常返回一个 Optional 对象
    Optional<String> optional1 = Optional.ofNullable("mydlq");
    
    // 传入 null 参数，正常返回 Optional 对象
    Optional optional2 = Optional.ofNullable(null);
}
```

运行代码，可以观察到正常传入值和传入 null 值时，都没有抛出异常。



### 3、对象方法 isPresent()

- **方法作用：** 如果值存在则方法会返回 true，否则返回 false
- **方法描述：** 该方法其实就是用于判断创建 Optional 时传入参数的值是否为空，实现代码就简单一行，即 `value != null` 所以如果不为空则返回 true，否则返回 false
- **返回类型：** boolean
- **示例代码：**

```java
public static void main(String[] args) {
    // 传入正常值，正常返回一个 Optional 对象，并使用 isPresent 方法
    Optional optional1 = Optional.ofNullable("mydlq");
    System.out.println("传入正常值返回：" + optional1.isPresent());

    // 传入参数为 null 生成一个 Optional 对象，并使用 isPresent 方法
    Optional optional2 = Optional.ofNullable(null);
    System.out.println("传入 null 值返回：" + optional2.isPresent());
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
传入正常值返回：true
传入 null 值返回：false
```

可以看到传入正常参数时调用 Optional 对象的 isPresent 方法时返回 true，传入 null 参数返回 false。



### 4、对象方法 get()

- **方法作用：** 如果 Optional 有值则将其返回，否则抛出 NoSuchElementException 异常
- **方法描述：** get 方法内部实现其实就是判断 Otpional 对象中的 value 属性是否为 null，如果是就抛出 NoSuchElementException 异常，否则返回这个 value 值
- **返回类型：** T
- **示例代码：**

```java
public static void main(String[] args) {
    // 传入正常值，正常返回一个 Optional 对象，并使用 get 方法获取值
    Optional optional1 = Optional.ofNullable("mydlq");
    System.out.println(optional1.get());

    // 传入参数为 null 生成一个 Optional 对象，并使用 get 方法获取值
    Optional optional2 = Optional.ofNullable(null);
    System.out.println(optional2.get());
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
传入正常参数：mydlq
Exception in thread "main" java.util.NoSuchElementException: No value present
	at java.util.Optional.get(Optional.java:135)
	at club.mydlq.OptionalExample.main(OptionalExample.java:14)
```

可以观察到传入正常值的 Optional 调用 get 方法正常输出值，通过空的 optional 对象使用 get 方法获取值时，抛出 NoSuchElementException 异常。



### 5、对象方法 ifPresent()

- **方法作用：** 如果值存在则使用该值调用 consumer , 否则不做任何事情
- **方法描述：** 该方法 ifPresent(Consumer<? super T> consumer) 中参数接收的是 Consumer 类，它包含一个接口方法 accept()，该方法能够对传入的值进行处理，但不会返回结果。这里传入参数可以传入 Lamdda 表达式或 Consumer 对象及实现 Consumer 接口的类的对象
- **返回类型：** void
- **示例代码：**

```java
public static void main(String[] args) {
    // 创建 Optional 对象，然后调用 Optional 对象的 ifPresent 方法，传入 Lambda 表达式
    Optional optional1 = Optional.ofNullable("mydlq1");
    optional1.ifPresent((value) -> System.out.println("Optional 的值为：" + value));

    // 创建 Optional 对象，调用 Optional 对象的 ifPresent 方法，传入实现 Consumer 匿名内部类
    Optional optional2 = Optional.ofNullable("mydlq2");
    Consumer<String> consumer = new Consumer() {
        @Override
        public void accept(Object value) {
            System.out.println("Optional 的值为：" + value);
        }
    };
    optional2.ifPresent(consumer);
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
Optional 的值为：mydlq1
Optional 的值为：mydlq2
```

可以观察到，调用 ifPresent 使用 lambda 或者内部匿名类方法，都是为了再执行 Optional 对象的 ifPresent 方法时，执行一段代码逻辑。



### 6、对象方法 orElse()

- **方法作用：** 如果该值存在就直接返回， 否则返回指定的其它值
- **方法描述：** orElse 方法实现很简单，就是使用三目表达式对传入的参数值进行 null 验证，即 `value != null ? value : other;` 如果为 null 则返回 true，否则返回 false
- **返回类型：** T
- **示例代码：**

```java
public static void main(String[] args) {
    // 传入正常参数，获取一个 Optional 对象，并使用 orElse 方法设置默认值
    Optional optional1 = Optional.ofNullable("mydlq");
    Object object1 = optional1.orElse("默认值");
    System.out.println("如果值不为空："+object1);

    // 传入 null 参数，获取一个 Optional 对象，并使用 orElse 方法设置默认值
    Optional optional2 = Optional.ofNullable(null);
    Object object2 = optional2.orElse("默认值");
    System.out.println("如果值为空："+object2);
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
如果值不为空：mydlq
如果值为空：默认值
```

可以观察到，如果 Optional 的值为空，则返回 orElse() 方法设置的默认值，否则返回 Optional 中的值。



### 7、对象方法 orElseGet()

- **方法作用：** 如果该值存在就返回值，否则触发 other，并返回 other 调用的结果
- **方法描述：** orElseGet 方法和 orElse 方法类似，都是在 Optional 值为空时，返回一个默认操作，只不过 orElse 返回的是默认值，而 orElseGet 是执行 lambda 表达式，然后返回 lambda 表达式执行后的结果
- **返回类型：** T
- **示例代码：**

```java
public static void main(String[] args) {
    // 传入正常参数，获取一个 Optional 对象，并使用 orElse 方法设置默认值
    Optional optional1 = Optional.ofNullable("mydlq");
    Object object1 = optional1.orElseGet(() -> {
        String defaultVal = "执行逻辑和生成的默认值";
        return defaultVal;
    });
    System.out.println("输出的值为："+object1);

    // 传入 null 参数，获取一个 Optional 对象，并使用 orElse 方法设置默认值
    Optional optional2 = Optional.ofNullable(null);
    Object object2 = optional2.orElseGet(() -> {
        String defaultVal = "执行逻辑和生成的默认值";
        return defaultVal;
    });
    System.out.println("输出的值为："+object2);
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
输出的值为：mydlq
输出的值为：执行逻辑和生成的默认值
```

可也观察到，当 Optional 值为不为空时正常返回带值的 Optional，如果 Optional 为空则返回 orElseGet 方法中 lambda 表达式执行后生成的值。



### 8、对象方法 orElseThrow()

- **方法作用：** 如果 Optional 存在该值，返回包含的值，否则抛出由 Supplier 继承的异常
- **方法描述：** orElseThrow 方法其实就是判断创建 Optional 时传入的参数是否为 null，如果是非 null 则返回传入的值，否则抛出异常
- **返回类型：** T
- **示例代码：**

```java
public static void main(String[] args) {
    // 传入正常参数，获取一个 Optional 对象，并使用 orElseThrow 方法
    try {
        Optional optional1 = Optional.ofNullable("mydlq");
        Object object1 = optional1.orElseThrow(() -> {
                    System.out.println("执行逻辑，然后抛出异常");
                    return new RuntimeException("抛出异常");
                }
        );
        System.out.println("输出的值为：" + object1);
    } catch (Throwable throwable) {
        throwable.printStackTrace();
    }
    
    // 传入 null 参数，获取一个 Optional 对象，并使用 orElseThrow 方法
    try {
        Optional optional2 = Optional.ofNullable(null);
        Object object2 = optional2.orElseThrow(() -> {
                    System.out.println("执行逻辑，然后抛出异常");
                    return new RuntimeException("抛出异常");
                }
        );
        System.out.println("输出的值为：" + object2);
    } catch (Throwable throwable) {
        throwable.printStackTrace();
    }
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
值为不为空输出的值：mydlq
执行逻辑，然后抛出异常
java.lang.RuntimeException: 抛出异常
	at club.mydlq.OptionalExample.lambda$main$1(OptionalExample.java:25)
	at java.util.Optional.orElseThrow(Optional.java:290)
	at club.mydlq.OptionalExample.main(OptionalExample.java:23)
```

可以观察到，当创建 Optional 时如果传入的参数为空则执行 Lambda 表达式代码逻辑后抛出异常信息，否则返回传入的参数值。



### 9、对象方法 map()

- **方法作用：** 如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的 Optional 作为 map 方法返回值，否则返回空 Optional
- **方法描述：** map 方法主要用于获取某个对象中的某个属性值的 Optional 对象时使用。map 方法调用时，首先验证传入的映射函数是否为空，如果为空则抛出异常。然后，再检测 Optional 的 value 是否为空，如果是，则返回一个空 value 的 Optional 对象。如果传入的映射函数和 Optinal 的 value 都不为空，则返回一个带 value 对象属性的 Optional 对象
- **返回类型：** `Optional<U>`
- **示例代码：**

**示例1：** 创建 Map 集合，存储一些键值对信息，通过 Optional 操作 Map 获取值，然后观察：

```java
public static void main(String[] args) {
    // 创建 map 对象
    Map<String, String> userMap = new HashMap<>();
    userMap.put("name1", "mydlq");
    userMap.put("name2", null);

    // 传入 Map 对象参数，获取一个 Optional 对象，获取 name1 属性
    Optional<String> optional1 = Optional.of(userMap).map(value -> value.get("name1"));

    // 传入 Map 对象参数，获取一个 Optional 对象，获取 name2 属性
    Optional<String> optional2 = Optional.of(userMap).map(value -> value.get("name2"));

    // 获取 Optional 的值
    System.out.println("获取的 name1 的值：" + optional1.orElse("默认值"));
    System.out.println("获取的 name2 的值：" + optional2.orElse("默认值"));
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
获取的 Optional 的值：mydlq
获取的 Optional 的值：默认值
```



**示例2：** 创建一个用户类，使用 Optional 操作用户对象，获取其 name 参数，结合 Optional 的 map 方法获取值，进行观察：

```java
public class User {
    private String name;
    public User(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

使用 Optional 的 map 方法对值处理：

```java
public static void main(String[] args) {
    // 创建一个对象，设置姓名属性而不设置性别，这时候性别为 null
    User user1 = new User("测试名称");
    User user2 = new User();

    // 使用 Optional 存储 User 对象
    Optional<User> optional1 = Optional.ofNullable(user1);
    Optional<User> optional2 = Optional.ofNullable(user2);

    // 获取对象的 name 属性值
    String name1 = optional1.map(User::getName).orElse("未填写");
    String name2 = optional2.map(User::getName).orElse("未填写");

    // 输出结果
    System.out.println("获取的名称：" + name1);
    System.out.println("获取的名称：" + name2);
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
获取的名称：测试名称
获取的名称：未填写
```

**总结：**

通过上面两个示例观察到，通过 Optional 对象的 map 方法能够获取映射对象中的属，创建 Optional 对象，并以此属性充当 Optional 的值，结合 orElse 方法，如果获取的属性的值为空，则设置个默认值。



### 10、对象方法 flatMap()

- **方法作用：** 如果值存在，返回基于 Optional 包含的映射方法的值，否则返回一个空的 Optional
- **方法描述：** flatMap 方法和 map 方法类似，唯一的不同点就是 map 方法会对返回的值进行 Optional 封装，而 flatMap 不会，它需要手动执行 Optional.of 或 Optional.ofNullable 方法对 Optional 值进行封装（map方法的mapping函数返回值可以是任何类型T，而flatMap方法的mapping函数必须是Optional）
- **返回类型：** `Optional<U>`
- **示例代码：**

```java
public static void main(String[] args) {
    // 创建 map 对象
    Map<String, String> userMap = new HashMap<>();
    userMap.put("name", "mydlq");
    userMap.put("sex", "男");

    // 传入 Map 对象参数，获取一个 Optional 对象
    Optional<Map<String, String>> optional1 = Optional.of(userMap);

    // 使用 Optional 的 flatMap 方法，获取 Map 中的 name 属性
    // 然后通过获取的值手动创建一个新的 Optional 对象
    Optional optional2 = optional1.flatMap(value -> Optional.ofNullable(value.get("name")));

    // 获取 Optional 的 value
    System.out.println("获取的 Optional 的值：" + optional2.get());
}
```

运行代码，可以观察到控制台输出内容如下：

```fallback
获取的 Optional 的值：mydlq
```

根据结果观察，可以看到 flatMap 和 map 方法没有什么区别，但是仔细看，代码中调用 flatMap 后，需要手动执行 of 或 ofNullable 方法创建了 Optional 对象。





### 11、对象方法 filter()

- **方法作用：** 如果有值并且满足断言条件返回包含该值的 Optional，否则返回空 Optional
- **方法描述：** filter 方法通过传入的限定条件对 Optional 实例的值进行过滤，如果 Optional 值不为空且满足限定条件就返回包含值的 Optional，否则返回空的 Optional。这里设置的限定条件需要使用实现了 Predicate 接口的 lambda 表达式来进行配置
- **返回类型：** Optional<T>
- **示例代码：**

```java
public static void main(String[] args) {
    // 创建一个测试的 Optional 对象
    Optional<String> optional = Optional.ofNullable("mydlq");
    // 调用 Optional 的 filter 方法，设置一个满足的条件，然后观察获取的 Optional 对象值是否为空
    Optional optional1 =optional.filter((value) -> value.length() > 2);
    System.out.println("Optional 的值不为空：：" + optional.isPresent());

    // 调用 Optional 的 filter 方法，设置一个不满足的条件，然后观察获取的 Optional 对象值是否为空
    Optional optional2 =optional.filter((value) -> value.length() <2);
    System.out.println("Optional 的值不为空：：" + optional2.isPresent());
}
```

运行代码，可以观察到控制台输出内容如下：

```bash
Optional 的值不为空：true
Optional 的值不为空：false
```

根据结果可以观察到，可以通过 filter 设置一个条件来判断 Optional 的值，如果满足条件就返回带值的 Optional，否则返回空的 Optional。



## 四、Optional 最佳实践

在介绍一栏中已经说过 Optional 是个容器，它可用保存类型的 T 的值，即使 T 为 null 也可以使用 Optional 存储，这样我就不用显示进行空值检测，防止空指针异常。

上面也介绍了 Optional 的各种方法，在实际使用中这些方法常常组合使用。且很多方法也常与 Lambda 表达式结合，获取我们想要的结果的值。

下面是常用的示例，可以作为参考：

### 1、对集合中的对象属性进行过滤

创建一个 User 对象实体类，里面包含 name 属性：

```java
public class User {
    private String name;
    public User(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

创建一个使用 main 方法的类，创建几个 User 对象且设置不同的值，有的对象为 null 有的属性不设置，然后通过 Optional 获取 name 属性值加入集合，进行测试：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class OptionalExample {

    /**
     * 测试的 main 方法
     */
    public static void main(String[] args) {
        // 创建一个测试的用户集合
        List<User> userList = new ArrayList<>();

        // 创建几个测试用户
        User user1 = new User("abc");
        User user2 = new User("efg");
        User user3 = null;

        // 将用户加入集合
        userList.add(user1);
        userList.add(user2);
        userList.add(user3);

        // 创建用于存储姓名的集合
        List<String> nameList = new ArrayList();
        // 循环用户列表获取用户信息，值获取不为空且用户以 a 开头的姓名，
        // 如果不符合条件就设置默认值，最后将符合条件的用户姓名加入姓名集合
        for (User user : userList) {
            nameList.add(Optional.ofNullable(user)
                         .map(User::getName)
                         .filter(value -> value.startsWith("a")).orElse("未填写"));
        }

        // 输出名字集合中的值
        System.out.println("通过 Optional 过滤的集合输出：");
        nameList.stream().forEach(System.out::println);
    }

}
```

输出运行结果：

```bash
通过 Optional 过滤的集合输出：
abc
未填写
未填写
```

通过上面，可以观察到，使用 Optional 有时候可以很方便的过滤一些属性，而且它的方法可以通过链式调用，方法间相互组合使用，使我们用少量的代码就能完成复杂的逻辑。



### 2、有异常建议用 ofNullable() + orElseThrow()

### 3、无异常建议用 ofNullable() + orElse()

### 4、看情况搭配 isPresent() or ifPresent()