# 认识反射机制

> 1. Java反射学习系列-绪论：https://hanchao.blog.csdn.net/article/details/79358924
> 2. Java-反射与注解：https://www.cnblogs.com/lishisan/p/11219404.html

Java反射机制可以让我们在编译期（Compile Time）之外的运行期（Runtime）检查类，接口，变量以及方法的信息。反射还可以让我们在运行期实例化对象，调用方法，通过调用get和set方法获取变量的值。

1、反射是框架设计的灵魂：

1. 框架：半成品软件。可以在框架的基础上进行软件开发，简化编码
2. 反射：将Java代码的各个组成部分封装为其他对象，可以在程序运行过程中操作这些对象，这就是Java的反射机制
3. 反射的好处：1：可以在程序运行过程中，操作这些对象。2：可以解耦，提高程序的可扩展性
4. **简而言之：反射是程序在运行时检查和修改对象的结构和行为的能力**

2、反射使我们能够执行以下操作：

1. 在运行时检查对象的类
2. 在运行时为类构造一个对象
3. 在运行时检查类的字段和方法
4. 在运行时调用对象的任何方法

3、在Java语言中之所以会有如此众多的开源技术支撑，很大一部分是来自于Java最大的特征—反射机制，如果不能够灵活地去使用反射机制进行项目的开发与设计，那么可以说你并未接触到Java的精髓。所有技术实现的目标只有一点：重用性。

4、对于反射技术首先考虑的是“反”与“正”的操作，所谓的“正”操作指的是使用一个类的时候，一定要先导入程序所在的包，而后根据类进行对象的实例化，并依靠对象调用类中的方法。但是“反”则是根据实例化对象反推出其类型。

- 正常方式：需要引入“包.类”名称»»»通过new实例化»»»取得实例化对象
- 反射方式：实例化对象»»»getClass方法»»»得到完整的“包.类”名称

操作示例 1：正向操作

```java
// 1、导入程序所在的包.类，知道对象的出处了
import java.util.Date;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 2、通过类产生实例化对象，「正射」
        Date date = new Date();
        // 3、根据对象调用类中的方法
        System.out.println(date.getTime());
    }
}

// 输出内容
1636077541911
```

如果要实现反的处理操作，那么首先要采用的就是Object类中所提供的一个方法：获取Class对象信息：

```java
public final Class<?> getClass();
```

操作示例 2：观察Class对象的使用

```java
package com.example;
// 1、导入程序所在的包.类，知道对象的出处了
import java.util.Date;
public class JavaAPIDemo {
    public static void main(String[] args) {
        // 2、通过类产生实例化对象
        Date date = new Date();
        //3、根据实例化对象找到对象的所属类型
        System.out.println(date.getClass());
    }
}

// 输出内容
class java.util.Date
```

PS：Object类中的getClass()方法可以帮助使用者找到对象的根源。



# 获取Class对象的方式

反射中所有的核心操作都是通过Class类对象展开的，可以说Class类是反射操作的根源所在，但是这个类如果要想获取它的实例化对象可以采用三种方式来实现。首先来观察java.lang.Class类的定义：

```java
public final class Class<T> extends Object implements Serializable, GenericDeclaration, Type, AnnotatedElement {}
```

从JDK1.5开始，Class类在定义的时可以使用泛型进行标记，这样的用法主要是希望可以避免所谓的向下转型。下面通过具体的操作讲解三种实例化形式。

| 获取Class对象方式       | 作用                         | 应用场景                                                 |
| ----------------------- | ---------------------------- | -------------------------------------------------------- |
| Class.forName("全类名") | 通过指定的字符串路径获取     | 多用于配置文件，将类名定义在配置文件中。读取文件，加载类 |
| 类名.class              | 通过类名的属性class获取      | 多用于参数的传递                                         |
| 对象.getClass()         | 通过对象的getClass()方法获取 | 多用于对象的获取字节码的方式（需要先实例化对象）         |



## 1、对象.getClass()

【**Object类支持**】Object类可以根据实例化对象获取Class对象：`public final Class<?> getClass();`

```java
public final Class<?> getClass();
```

操作示例 1：这种方式有一个不是缺点的缺点：如果只是想获得Class类对象，则必须产生指定类对象后才可以获得。

```java
package com.example;
// 采用自定义的程序类
class Person{}
public class JavaAPIDemo {
    public static void main(String[] args) {
        Person per = new Person();  // 已经存在有指定类的实例化对象
        Class<? extends Person> cls = per.getClass();
        System.out.println(cls.getName());  // 获取的是类的完整名称
    }
}

// 输出内容
com.example.Person
```



## 2、类名.class

【**JVM直接支持**】采用“类.class”的形式实例化：Object.class

操作示例 2：特点或缺点：如果想获得Class类对象，就必须导入程序所对应的开发包。

```java
package com.example;
// 采用自定义的程序类
class Person{}
public class JavaAPIDemo {
    public static void main(String[] args) {
        Class<? extends Person> cls = Person.class;
        System.out.println(cls.getName());
    }
}

// 输出内容
com.example.Person
```



## 3、Class.forName("全类名")

【**Class类支持**】在Class类中提供有一个static方法 加载类：

```java
public static Class<?> forName(String className) throws ClassNotFoundException;
```

操作示例 3：当你知道该类的全路径名时，你可以使用该方法获取 Class 类对象。

```java
package com.example;
class Person {}
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 该类必须存在
        Class<?> cls = Class.forName("com.example.Person");
        System.out.println(cls.getName());
    }
}

// 输出内容
com.example.Person
```

这种模式最大的特点：可以直接采用字符串的形式定义要使用的类型，并且程序中不需要编写任何的import语句。如果此时要使用的程序类不存在则会抛出“java.lang.ClassNotFoundException”异常。



# 反射机制与对象实例化

经过一系列分析之后可以发现虽然获得了Class类的实例化对象，但是依然觉得这个对象获取的意义不是很大，所以为了进一步理解反射的核心意义，下面将通过几个案例进行程序的说明（都是实际开发中一定会使用到的）。通过实际开发中的案例进行程序的说明，来进一步理解反射的核心意义。

## 1、反射实例化对象

获取Class对象后最大意义实际上并不是在于只是一个对象的实例化操作形式，更重要的是Class类中提供了一个对象的反射实例化方法（代替了关键字new）

1. 在JDK1.9以前的实例化：

   ```java
   @Deprecated(since="9")
   public T newInstance() throws InstantiationException, IllegalAccessException;
   ```

2. 在JDK1.9之后的实例化：

   ```java
   clazz.getDeclaredConstructor().newInstance();
   ```

操作示例 1：通过newInstance()方法实例化Person类对象

```java
package com.example;
class Person {
    // 任何情况下如果要实例化类对象，一定要调用类中的构造方法
    public Person() {
        System.out.println("*********** Person类构造方法 ************");
    }
    @Override
    public String toString() {
        return "我是一个人，一个脱离了低级趣味的好人！";
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Class.forName("com.example.Person");
        // 实例化对象，JDK1.9后被废除了
        Object obj = cls.newInstance();
        // 输出对象调用toString()方法
        System.out.println(obj);
    }
}

// 输出内容
*********** Person类构造方法 ************
我是一个人，一个脱离了低级趣味的好人！
```

操作示例 2：使用getDeclaredConstructor().newInstance()进行对象实例化

```java
package com.example;
class Person {
    // 任何情况下如果要实例化类对象，一定要调用类中的构造方法
    public Person() {
        System.out.println("*********** Person类构造方法 ************");
    }
    @Override
    public String toString() {
        return "我是一个人，一个脱离了低级趣味的好人！";
    }
}
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls=Class.forName("com.example.Person");
        // 实例化对象，JDK1.9后
        Object obj= cls.getDeclaredConstructor().newInstance();
        System.out.println(obj);
    }
}

// 输出内容
*********** Person类构造方法 ************
我是一个人，一个脱离了低级趣味的好人！
```

从JDK1.9后，newInstance()被替代了，因为默认的Class类中的newInstance()方法只能调用无参构造，所以很多开发者会认为其描述的不准确，于是将其变换了形式（构造方法会讲解）。



## 2、反射与工厂设计模式

通过案例逐步掌握工厂设计模式，通过工厂类获取指定接口的实例化对象。

如果要想进行对象的实例化处理，除了可以使用关键字new之外，还可以使用反射机制来完成，于是此时一定会思考一个问题：为什么要提供有一个反射的实例化？那么到底是使用关键字new，还是使用反射呢？

如果要想更好的理解此类问题，最好的解决方案就是通过**工厂设计模式**来解决。

**工厂模式的最大特点：**客户端的程序类不直接牵扯到对象的实例化管理，只与接口发生关联，通过工厂类获取指定接口的实例化对象。

操作示例 1：传统的工厂设计模式

```java
package com.example;

interface IMessage {
    public void send();
}
class NetMessage implements IMessage {
    @Override
    public void send() {
        System.out.println("【网络消息发送】www.xxxx.cn");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 如果直接实例化则一定会有耦合问题
        IMessage msg = new NetMessage();
    }
}
```

在实际的开发中，接口的主要作用是为不同层提供有一个操作的标准。但是如果此时直接将一个子类设置为接口实例化操作，那么一定会有耦合问题，所以使用了工厂设计模式来解决此问题。

操作示例 2：利用工厂设计模式解决

```java
package com.example;

interface IMessage {
    void send();
}
class NetMessage implements IMessage {
    @Override
    public void send() {
        System.out.println("【网络消息发送】www.xxxx.cn");
    }
}

class Factory {
    // 没有产生实例化对象的意义，所以构造方法私有化
    private Factory() {}
    public static IMessage getInstance(String className) {
        if ("netMessage".equalsIgnoreCase(className)) {
            return new NetMessage();
        }
        return null;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        IMessage msg = Factory.getInstance("netMessage");
        msg.send();
    }
}

// 输出内容
【网络消息发送】www.xxxx.cn
```

静态工厂设计模式：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/9e8300d8e1a04f7999dd979d147ea46d.png)

此种工厂设计模式属于静态工厂设计模式，也就是说如果现在要追加一个子类，则意味着工厂类一定要做出修改，因为如果不追加判断是无法获取指定接口对象的。

操作示例 3：为IMessage追加一个子类

```java
package com.example;

interface IMessage {
    void send();  //消息发送
}
class NetMessage implements IMessage {
    @Override
    public void send() {
        System.out.println("【网络消息发送】www.netMessage.cn");
    }
}
class CloudMessage implements IMessage {
    @Override
    public void send() {
        System.out.println("【云消息发送】www.cloudMessage.cn");
    }
}

class Factory {
    private Factory() {}
    public static IMessage getInstance(String className) {
        if ("netMessage".equalsIgnoreCase(className)) {
            return new NetMessage();
        } else if ("cloudMessage".equalsIgnoreCase(className)) {
            return new CloudMessage();
        }
        return null;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        IMessage netMsg = Factory.getInstance("netMessage");
        netMsg.send();
        IMessage cloud = Factory.getInstance("cloudMessage");
        cloud.send();
    }
}
```

工厂设计模式最有效的解决的是子类与客户端的耦合问题，但是解决的核心思想是在于提供了一个工厂类作为过渡端，但是随着项目的进行，IMessage接口有可能会拥有更多的子类,而且随着时间的推移，子类产生的可能会越来越多，那么此时就意味着，工厂类永远都要进行修改，并且永无停止之日。

那么这时候最好的解决方案就是不使用关键字new来完成，因为关键字在new在使用时，需要有一个明确的类存在。而newInstance()方法只需要有一个明确表示类名称的字符串即可。

操作示例 4：

```java
package com.example;

interface IMessage{
    void send();
}
class NetMessage implements IMessage{
    @Override
    public void send() {
        System.out.println("【网络消息发送】www.netMessage.cn");
    }
}
class CloudMessage implements IMessage{
    @Override
    public void send() {
        System.out.println("【云消息发送】www.cloudMessage.cn");
    }
}

class Factory {
    private Factory() {}
    public static IMessage getInstance(String className) {
        IMessage instance = null;
        try {
            instance = (IMessage) Class.forName(className).getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return instance;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        IMessage netMessage = Factory.getInstance("com.example.NetMessage");
        netMessage.send();
        IMessage cloudMessage = Factory.getInstance("com.example.CloudMessage");
        cloudMessage.send();
    }
}

// 输出内容
【网络消息发送】www.netMessage.cn
【云消息发送】www.cloudMessage.cn
```

这时可以发现，利用反射机制实现的工厂设计模式，最大的优势在于：对于接口子类的扩充将不再影响到工厂类的定义。

反射工厂设计模式：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/56ed15f351f849d890f17665dbc4c96c.png)

但现在依然需要进一步思考，因为在实际的项目开发过程中，有可能会存在有大量的接口，并且这些接口都可能需要通过工厂类实例化，所以此时的工厂设计模式不应该只为IMessage接口服务，应该为所有接口服务。

操作示例 5：

```java
package com.example;

class Factory{
    private Factory(){}
    /**
     * 获取接口实例化对象
     * @param className 接口的子类
     * @param clazz 描述的是一个接口的类型
     * @return 如果子类存在则返回指定接口的实例化对象
     */
    @SuppressWarnings("unchecked")
    public static <T> T getInstance(String className,Class<T> clazz){
        T instance = null;
        try {
            instance=(T) Class.forName(className).getDeclaredConstructor().newInstance();
        }catch (Exception e) {
            e.printStackTrace();
        }
        return instance;
    }
}

interface IService {
    void service();
}
class HouseService implements IService {
    @Override
    public void service() {
        System.out.println("【服务】为您的住宿提供服务。");
    }
}
interface IMessage {
    void send();//消息发送
}
class NetMessage implements IMessage {
    @Override
    public void send() {
        System.out.println("【网络消息发送】www.netMessage.cn");
    }
}
class CloudMessage implements IMessage {
    @Override
    public void send() {
        System.out.println("【云消息发送】www.cloudMessage.cn");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        IMessage msg = Factory.getInstance("com.example.NetMessage",IMessage.class);
        msg.send();
        IService service=Factory.getInstance("com.example.HouseService",IService.class);
        service.service();
    }
}

// 输出内容
【网络消息发送】www.netMessage.cn
【服务】为您的住宿提供服务。
```

此时的工厂模式将不再受限于指定的接口，可以为所有的接口提供实例化服务，达到可重用性。





## 3、反射与单例设计模式

通过案例引出懒汉式的单例设计模式所出现的问题，并使用使用synchronized关键字来处理解决。

单例设计模式的核心本质在于：类内部的构造方法私有化，在类的内部产生实例化对象后通过static方法获取实例化对象,进行类中的结构调用，单例设计模式一共有两类：懒汉式、饿汉式，本节主要来讨论懒汉式的单例设计模式。

操作示例 1：观察懒汉式单例设计模式的问题，单线程状态执行：

```java
package com.example;
class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    public void print() {
        System.out.println("www.xxxx.cn");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        Singleton sinA = Singleton.getInstance();
        sinA.print();
    }
}

// 输出内容
www.xxxx.cn
```

操作示例 2：观察懒汉式单例设计模式的问题，多线程状态执行：

```java
package com.example;

class Singleton {
    private static Singleton instance = null;
    private Singleton() {
        System.out.println("【" + Thread.currentThread().getName() + "】****** 实例化Singleton类对象 *******");
    }
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    public void print() {
        System.out.println("www.xxxx.cn");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        for (int x = 0; x < 3; x++) {
            new Thread(() -> Singleton.getInstance().print(), "单例消费端-" + x).start();
        }
    }
}

// 输出内容
【单例消费端-2】****** 实例化Singleton类对象 *******
【单例消费端-0】****** 实例化Singleton类对象 *******
www.xxxx.cn
【单例消费端-1】****** 实例化Singleton类对象 *******
www.xxxx.cn
www.xxxx.cn
```

单例设计模式的最大特点是在整体的运行程序中只允许产生一个实例化对象，但当有了若干个线程之后，实际上当前的程序就会产生多个实例化对象了，此时就不是单例设计模式了。此时问题造成的关键在于代码本身出现了不同步的情况，而要想解决的关键就在于同步处理，也就是需要使用synchronized关键字。

单例设计模式问题：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/d6cbe2c3fcfe41f294604ab7c95cd0ec.png)

操作示例 3：修改getInstance()进行同步处理

```java
package com.example;

class Singleton {
    private static Singleton instance = null;
    private Singleton() {
        System.out.println("【" + Thread.currentThread().getName() + "】****** 实例化Singleton类对象 *******");
    }
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    public void print() {
        System.out.println("www.xxxx.cn");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        for (int x = 0; x < 3; x++) {
            new Thread(() -> Singleton.getInstance().print(), "单例消费端-" + x).start();
        }
    }
}

// 输出内容
【单例消费端-0】****** 实例化Singleton类对象 *******
www.xxxx.cn
www.xxxx.cn
www.xxxx.cn
```

这个时候的确是进行了同步处理，但这个同步处理的代价有些大，因为效率会低。因为整体代码中实际上只有一块部分需要进行同步处理，也就是instance对象的实例化处理部分。我们可以知道，之前的同步操作是有些失误的。

操作示例 4：更加合理地进行同步处理

```java
package com.example;

class Singleton {
    private static volatile Singleton instance = null;
    private Singleton() {
        System.out.println("【" + Thread.currentThread().getName() + "】****** 实例化Singleton类对象 *******");
    }
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    public void print() {
        System.out.println("www.xxxx.cn");
    }
}
```

**面试题：请编写单例设计模式**

1. 【100%】直接编写一个饿汉式的单例设计模式，并且实现构造方法私有化
2. 【120%】在Java中哪里使用到了单例设计模式？Runtime类、Pattern类、Spring框架
3. 【200%】懒汉式单例模式的问题（即上面描述所出现的问题）：
   回答：单例设计模式的最大特点是在整体的运行程序中只允许产生一个实例化对象，但当有了若干个线程之后，实际上当前的程序就会产生多个实例化对象了，此时就不是单例设计模式了。此时问题造成的关键在于代码本身出现了不同步的情况，而要想解决的关键就在于同步处理，也就是需要使用synchronized关键字。



# 反射机制与类操作

利用反射获取类的基本信息（Package），一个类的基本信息主要包括类所在的包名称、父类的定义、父接口的定义。在反射机制的处理过程之中，不仅仅只是实例化对象的处理操作，更多的情况下还有类的组成结构操作，任何一个类的基本组成结构都是父类（父接口）、包、属性、方法（构造方法、普通方法）。

## 1、获取类的基本信息

一个类的基本信息主要包括类所在的包名称、父类的定义、父接口的定义。如果此时想获得该类的一些基础信息则可以通过Class类中的如下方法：

| 方法名称                                | 描述           |
| --------------------------------------- | -------------- |
| public Package getPackage()             | 获取包名称     |
| public Class<? super T> getSuperclass() | 获取继承父类   |
| public Class<?>[] getInterfaces()       | 获取实现父接口 |

操作示例 1：定义程序类，获得包名称、父类信息、父接口信息

```java
package com.example;

interface IMessageService {
    public void send();
}

interface IChannelService {
    public boolean connect();
}

abstract class AbstractBase {
}

class Person extends AbstractBase implements IChannelService,IMessageService {
    @Override
    public boolean connect() {}
    @Override
    public void send() {}
}
// 如上是程序类的定义

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 获取指定类的Class对象
        Class<?> cls = Person.class;
        // 获取指定类的包定义
        Package pack = cls.getPackage();
        // 获取包名称
        System.out.println(pack.getName());
        // 获取指定父类的Class对象
        Class<?> parent = cls.getSuperclass();
        // 获取父类信息：package + 类名
        System.out.println(parent.getName());
        // 获取父类的父类信息
        System.out.println(parent.getSuperclass().getName());
        // 获取父类的父类的父类信息
        System.out.println(parent.getSuperclass().getSuperclass());
        // 获取父接口的Class对象
        Class<?> clazz[] = cls.getInterfaces();
        for (Class<?> temp : clazz) {
            // 获取父接口的对象名称
            System.out.println(temp.getName());
        }
    }
}

// 输出内容
com.example
com.example.AbstractBase
java.lang.Object
null
com.example.IChannelService
com.example.IMessageService
```

当获取了一个类的Class对象后，就意味着这个对象可以获取类中的一切继承结构信息。



## 2、反射调用构造方法

本节通过之前的案例介绍了反射调用构造方法和普通方法，实现反射实例化对象，并通过反射来获取类之中的全部方法。

在一个类中除了有继承的关系外，最为重要的操作就是类中的结构处理了，而类中的结构首先需要观察的就是构造方法的使用问题，实际上在之前通过反射实例化对象的时候就已经接触到构造方法的问题了：实例化方法替代：`clazz.getDeclaredConstructor().newInstance()`

所有类的构造方法的获取都可以直接通过Class类来完成，该类中定义有如下的几个方法：

| 方法名称                                                     | 描述                                     |
| ------------------------------------------------------------ | ---------------------------------------- |
| `public Constructor<?>[] getDeclaredConstructors()`          | 获取指定类所有构造方法(不包含父类, 下同) |
| `public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)` | 获取指定类指定构造方法                   |
| `public Constructor<?>[] getConstructors()`                  | 获取指定类所有public的构造方法           |
| `public Constructor<T> getConstructor(Class<?>... parameterTypes)` | 获取指定类指定public的构造方法           |

Constructor类常用方法

| 方法名称                                                     | 描述                                         |
| ------------------------------------------------------------ | -------------------------------------------- |
| `public T newInstance(Object ... initargs)`                  | 调用构造方法传入指定参数进行对象实例化       |
| `public String getName()`                                    | 获取构造方法名称                             |
| `public Type[] getGenericParameterTypes()`                   | 获取构造方法的参数类型                       |
| `public Type[] getGenericExceptionTypes()`                   | 获取构造方法抛出的异常类型                   |
| `public int getParameterCount()`                             | 获取构造方法的参数个数                       |
| `public <T extends Annotation> T getAnnotation(Class<T> annotationClass)` | 获取全部声明的Annotation                     |
| `public void setAccessible(boolean flag)`                    | 设置构造方法可见性(AccessibleObject类的方法) |

操作示例 1：获取全部构造和指定构造

```java
package com.example;
import java.lang.reflect.Constructor;

class Person {
    public Person() {}
    private Person(String name) {}
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Person.class;
        System.out.println("---------------获取本类全部构造-----------");
        // 获取本类全部构造
        Constructor<?>[] declaredConstructors = cls.getDeclaredConstructors();
        for (Constructor<?> cons : declaredConstructors) {
            System.out.println(cons);
        }
        System.out.println("---------------获取本类指定构造-----------");
        System.out.println(cls.getDeclaredConstructor(String.class));


        System.out.println("---------------获取本类全部public构造-----------");
        // 获取本类全部public构造
        Constructor<?>[] constructors = cls.getConstructors();
        for (Constructor<?> cons : constructors) {
            System.out.println(cons);
        }
        System.out.println("---------------获取本类指定public构造-----------");
        System.out.println(cls.getConstructor());
        // 如果获取private的构造则会报异常
        System.out.println(cls.getConstructor(String.class));

    }
}

// 输出内容
---------------获取本类全部构造-----------
public com.example.Person()
private com.example.Person(java.lang.String)
---------------获取本类指定构造-----------
private com.example.Person(java.lang.String)
---------------获取本类全部public构造-----------
public com.example.Person()
---------------获取本类指定public构造-----------
public com.example.Person()
Exception in thread "main" java.lang.NoSuchMethodException: com.example.Person.<init>(java.lang.String)
```

操作示例 2：反射调用指定构造实例化对象

```java
package com.example;
import java.lang.reflect.Constructor;

class Person {
    private String name;
    public Person() {}
    public Person(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "姓名：" + this.name;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Person.class;
        Constructor<?> constructor = cls.getConstructor(String.class);
        Object obj = constructor.newInstance("小强");
        System.out.println(obj);
    }
}

// 输出内容
姓名：小强
```

虽然程序代码本身允许开发者调用有参构造，但是如果从实际的开发角度出发，所有使用反射的类中最好使用无参构造，因为这样的实例化可以达到统一性。



## 3、反射调用普通方法

在进行反射处理的时候也可以通过反射来获取类之中的全部方法，但是需要提醒的是，如果想通过反射调用这些方法，必须有一个前提条件：类之中要提供实例化对象。在Class类中提供了如下的操作可以获取方法对象：

| 方法名称                                                     | 描述                   |
| ------------------------------------------------------------ | ---------------------- |
| public Method[] getMethods()                                 | 获取全部方法(包括父类) |
| public Method getMethod(String name, Class<?>... parameterTypes) | 获取指定方法(包括父类) |
| public Method[] getDeclaredMethods()                         | 获取本类全部方法       |
| public Method getDeclaredMethod(String name, Class<?>... pTypes) | 获取本类指定方法       |

通过Class类获取每一个方法信息都使用了java.lang.reflect.Method类实例描述，通过该实例可以获取方法得相关信息，也可以实现方法的反射调用，Method类常用方法如下：

| 方法名称                                                     | 描述                                |
| ------------------------------------------------------------ | ----------------------------------- |
| public Object invoke(Object obj, Object... args)             | 方法调用，等价于“实例化对象.方法()” |
| public Class<?> getReturnType()                              | 获取方法返回值类型，返回Class       |
| public String getName()                                      | 获取构造方法名称                    |
| public Type[] getGenericParameterTypes()                     | 获取构造方法的参数类型              |
| public Type[] getGenericExceptionTypes()                     | 获取构造方法抛出异常类型            |
| public Type getGenericReturnType()                           | 获取方法返回值类型，返回Type        |
| public int getParameterCount()                               | 获取构造方法的参数个数              |
| `public <T extends Annotation> T getAnnotation(Class<T> annotationClass)` | 获取指定声明的Annotation            |
| public Annotation[] getAnnotations()                         | 获取全部声明的Annotation            |
| public int getModifiers()                                    | 获取方法修饰符                      |
| public void setAccessible(boolean flag)                      | 设置方法可见性                      |

操作示例 1：获取全部方法

```java
package com.example;
import java.lang.reflect.Method;

abstract class AbstractBase {
    public void send(){}
}

class Person extends AbstractBase {
    private boolean connect() {
        return true;
    }
    @Override
    public void send(){}
    @Override
    public String toString() {
        return super.toString();
    }
}

/**
 * 反射获取全部普通方法
 */
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Person.class;
        System.out.println("---------------获取本类与父类中的所有public类型的方法-----------");
        {
            // 获取全部public类型的方法(包括父类中的public类型的方法)
            Method[] methods = cls.getMethods();
            for (Method met : methods) {
                System.out.println(met);
            }
        }
        System.out.println("---------------获取本类中所有的方法（包括private）-----------");
        {
            // 获取本类方法（包括private）
            Method[] methods = cls.getDeclaredMethods();
            for (Method met : methods) {
                System.out.println(met);
            }
        }
    }
}

// 输出内容
---------------获取本类与父类中的所有public类型的方法-----------
public java.lang.String com.example.Person.toString()
public void com.example.Person.send()
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
---------------获取本类中所有的方法（包括private）-----------
public java.lang.String com.example.Person.toString()
private boolean com.example.Person.connect()
public void com.example.Person.send()
```

PS：注意如果重写父方法，最终只会获取子类重写的方法名

操作示例 2：获取类中的方法信息（上例是依靠Method类提供的toString()方法完成的，现在使用方法信息拼接获取方法的详细信息）

```java
package com.example;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;


/**
 * AbstractBase
 */
abstract class AbstractBase {
    public void send(){}
}

/**
 * Person
 */
class Person extends AbstractBase {
    private boolean connect() {
        return true;
    }
    @Override
    public void send(){}
    @Override
    public String toString() {
        return super.toString();
    }
}

/**
 * 反射获取全部普通方法的详细信息
 */
public class JavaAPIDemo {
    public static void main(String[] args) {
        System.out.println("---------------获取本类与父类中的所有public类型的方法-----------");
        // 获取指定类的Class对象
        Class<?> cls = Person.class;
        Method[] methods = cls.getMethods();
        for (Method met : methods) {
            // 修饰符
            int mod = met.getModifiers();
            System.out.print(Modifier.toString(mod) + " ");
            // 方法返回类型
            System.out.print(met.getReturnType().getName() + " ");
            // 方法名称
            System.out.print(met.getName() + "(");
            // 获取参数类型
            Class<?>[] params = met.getParameterTypes();
            for (int x = 0; x < params.length; x++) {
                System.out.print(params[x].getName() + " " + "arg-" + x);
                if (x < params.length - 1) {
                    System.out.print(",");
                }
            }
            System.out.print(")");
            Class<?>[] exp = met.getExceptionTypes();
            if (exp.length > 0) {
                System.out.print(" throws");
            }
            for (int x = 0; x < exp.length; x++) {
                System.out.print(exp[x].getName());
                if (x < exp.length - 1) {
                    System.out.println(", ");
                }
            }
            System.out.println();  // 换行
        }
    }
}

// 输出内容
---------------获取本类与父类中的所有public类型的方法-----------
public java.lang.String toString()
public void send()
public final native void wait(long arg-0) throwsjava.lang.InterruptedException
public final void wait(long arg-0,int arg-1) throwsjava.lang.InterruptedException
public final void wait() throwsjava.lang.InterruptedException
public boolean equals(java.lang.Object arg-0)
public native int hashCode()
public final native java.lang.Class getClass()
public final native void notify()
public final native void notifyAll()
```

**操作示例 3**：反射调用类中的setter、getter方法。需要使用到最重要的方法：`public Object invoke(Object obj, Object... args)`

```java
package com.example;
import java.lang.reflect.Method;

class Person  {
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}

/**
 * 反射方法的调用（需要注意：必须要先实例化对象）
 */
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 获取指定类的Class对象
        Class<?> cls = Class.forName("com.example.Person");
        // 要设置的属性内容
        String value = "小强子";
        // 1.任何情况下如果要想保存类中的属性或者调用类中的方法都必须保证存在有实例化对象，既然不允许导入包，那么就反射实例化，调用无参构造实例化
        Object obj = cls.getDeclaredConstructor().newInstance();
        // 2.如果要想进行方法的调用，那么一定要获取方法的名称
        String setMethodName = "setName";
        // 获取指定的方法
        Method method = cls.getDeclaredMethod(setMethodName, String.class);
        // 方法调用，等价于：Person对象.setName(value);
        method.invoke(obj, value);
        String getMethodName="getName";
        // getter没有参数
        method = cls.getDeclaredMethod(getMethodName);
        // 方法调用，等价于：Person对象.getName();
        System.out.println(method.invoke(obj));
    }
}

// 输出内容
小强子
```

利用此类操作整体的形式上不会有任何明确的类对象产生，一切都是依靠反射机制处理的，这样的处理避免了与某一个类耦合问题。



## 4、反射调用成员属性

本节需要了解反射调用成员的几种操作方法，熟练掌握Field类中获取成员类型的方法。了解反射与类的操作

类结构中的最后一个核心的组成就是成员（Field），大部分情况下都会将其称为成员属性，对于成员信息的获取也是通过Class类完成，在这个类中提供了如下两组操作方法：

| 方法名                                                       | 描述                       |
| ------------------------------------------------------------ | -------------------------- |
| public Field[] getDeclaredFields() throws SecurityException  | 获取本类全部成员信息       |
| public Field getDeclaredField(String name) throws NoSuchFieldException, SecurityException | 获取指定成员属性信息       |
| public Field[] getFields() throws SecurityException          | 获取父类全部public成员信息 |
| public Field getField(String name) throws NoSuchFieldException, SecurityException | 获取父类指定public成员信息 |

反射中成员通过java.lang.reflect.Field实例描述，Field类中提供的常用方法如下：

| 方法名                                    | 描述               |
| ----------------------------------------- | ------------------ |
| public Class<?> getType()                 | 获取成员属性类型   |
| public void set(Object obj, Object value) | 设置成员属性内容   |
| public Object get(Object obj)             | 获取成员属性内容   |
| public int getModifiers()                 | 获取成员属性修饰符 |
| public void setAccessible(boolean flag)   | 设置成员属性可见性 |

操作示例 1：获取类中的成员属性信息

```java
package com.example;
import java.lang.reflect.Field;

interface IChannelService {
    public static final String NAME = "xxxxjava";
}

abstract class AbstractBase {
    protected static final String BASE = "www.xxxx.cn";
    private String info = "HELLO XXXX";
}

class Person extends AbstractBase implements IChannelService {
    private String name;
    private int age;
}

/**
 * 反射获取成员属性
 */
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Class.forName("com.example.Person");
        System.out.println("--------- 获取父类之中public类型的成员信息------------");
        {    // 获取父类之中公共成员信息
            Field[] fields = cls.getFields();
            for (Field fie : fields) {
                System.out.println(fie);
            }
        }
        System.out.println("---------获取本类中定义的成员信息------------");
        {  // 获取本类中定义的成员信息
            Field[] fields = cls.getDeclaredFields();
            for (Field fie : fields) {
                System.out.println(fie);
            }
        }
    }
}

// 输出内容
--------- 获取父类之中public类型的成员信息------------
public static final java.lang.String com.example.IChannelService.NAME
---------获取本类中定义的成员信息------------
private java.lang.String com.example.Person.name
private int com.example.Person.age
```

但是在Field类中最为重要的操作形式并不是获取全部成员，而是这三个方法：设置属性内容、获取属性内容、属性解除封装

操作示例 2：所有成员是在对象实例化后进行空间分配的，所以一定要先有实例化对象后才能对成员操作。直接调用Person类中的name私有成员

```java
package com.example;
import java.lang.reflect.Field;

/**
 * Person
 */
class Person {
    private String name;
}

/**
 * 反射设置成员属性
 */
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Class.forName("com.example.Person");
        // 实例化对象（分配成员空间）
        Object obj = cls.getDeclaredConstructor().newInstance();
        // 获取成员对象
        Field nameField = cls.getDeclaredField("name");
        // 对私有属性必须先解除封装
        nameField.setAccessible(true);
        // 设置属性内容
        nameField.set(obj,"番茄强");
        // 获取属性内容
        System.out.println(nameField.get(obj));
    }
}

// 输出内容
番茄强
```

通过一系列的分析可以发现，类中的构造、方法、成员属性都可以通过反射实现调用，但是对于成员的反射很少这样直接处理，大部分操作都应该setter和getter处理，所以对于以上的代码只能够说是反射的特色，但是不具备实际的使用能力，而对于Field类在实际开发中只有一个方法最为常用：获取成员类型

操作示例 3：获取指定成员属性的类型

```java
package com.example;
import java.lang.reflect.Field;

class Person {
    private String name;
}

/**
 * 获取成员属性类型
 */
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Class.forName("com.example.Person");
        Field nameField = cls.getDeclaredField("name");
        // 获取完整的名称，包.类：java.lang.String
        System.out.println(nameField.getType().getName());
        // 获取类名称：String
        System.out.println(nameField.getType().getSimpleName());
    }
}

// 输出和内容
java.lang.String
String
```

在以后开发中进行反射处理的时候，往往会利用Field类和Method类实现类中的setter方法的调用。

![image.png](https://ucc.alicdn.com/pic/developer-ecology/39469674e6b144e2bf8854b95dd99fa2.png)





## 5、Unsafe工具类

本节介绍了绕过JVM的相关的对象管理机制实例化对象的Unsafe工具类，巩固加深对反射的理解。

反射是Java的第一大特点，一旦打开反射的大门就可以有更加丰富的类设计形式。除了JVM本身支持的反射处理之外，在Java中也提供了一个Unsafe类（不安全的操作），这个类的主要特点是可以利用反射来获取对象，并且直接使用底层的C++来代替JVM执行，即：可以绕过JVM的相关的对象管理机制，一旦使用Unsafe，那么项目之中将无法继续使用JVM的内存管理机制以及垃圾回收处理。

但是如果要想使用Unsafe类首先就需要确认一下这个类中定义的构造方法与常量问题：

| 方法名                                               | 描述     |
| ---------------------------------------------------- | -------- |
| private Unsafe() {}                                  | 私有构造 |
| private static final Unsafe theUnsafe = new Unsafe() | 私有常量 |

但是需要注意的是，在这个Unsafe类中并没有提供static的方法，即：不能通过类似于传统的单例设计模式中提供的样式来进行操作，如果想要获得这个类的对象，就必须利用反射机制来完成。

```java
Field field = Unsafe.class.getDeclaredField("theUnsafe");
field.setAccessible(true); // 解除封装处理
Unsafe unsafeObject = (Unsafe) field.get(null); // static属性不需要传递实例化对象
```

操作示例 1：使用Unsafe类绕过实例化对象的管理

```java
package com.example;
import sun.misc.Unsafe;
import java.lang.reflect.Field;

class Singleton {
    private Singleton() {
        // 构造不执行
        System.out.println("*****Singleton类构造******");
    }
    public void print() {
        System.out.println("www.xxxx.cn");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // Unsafe三步曲
        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        Unsafe unsafeObject = (Unsafe) field.get(null);
        // 利用Unsafe类绕过了JVM管理机制，可以在没有实例化对象的情况下获取一个Singleton类实例化对象
        Singleton instance = (Singleton) unsafeObject.allocateInstance(Singleton.class);
        // 对象调用
        instance.print();
    }
}

// 输出内容
www.xxxx.cn
```

Unsafe只能说为开发提供了一些更加方便的处理机制，但是这种操作由于不受JVM的管理，所以如果不是必须的情况下不建议使用。讲解这个类的主要目的是帮助大家巩固对于反射的理解。

# 反射与简单Java类



## 1、传统属性赋值弊端

本节介绍了传统属性赋值的弊端--编码的重复性高，唯一的解决方案就是反射机制，反射机制最大的特征是可以根据其自身的特点实现相同功能类的重复操作的抽象处理。

简单Java类主要是由属性组成，并且提供有相应的setter、getter处理方法，同时简单Java类最大的特征就是通过对象保存相应的类属性内容。但是如果使用传统的简单Java类开发，那么也会面临非常麻烦的困难。

操作示例 1：传统的简单Java类操作及调用

```java
package com.example;
class Emp{
    private String ename;
    private String job;
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setJob(String job) {
        this.job = job;
    }
    public String getEname() {
        return ename;
    }
    public String getJob() {
        return job;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 更多情况下开发者关注的是内容的设置
        Emp emp = new Emp();
        emp.setEname("Sam");
        emp.setJob("Java");
        // 使用为对象设置之后
        System.out.println("姓名：" + emp.getEname() + "、职位：" + emp.getJob());
    }
}

// 输出内容
姓名：Sam、职位：Java
```

特别强调，为了方便理解，本次Emp类中定义的ename、job两个属性的类型使用的都是String类型。按照传统的做法，此时应该首先实例化Emp类的对象，而后通过实例化对象进行setter方法的调用以设置属性内容。

在整个进行Emp对象实例化并设置数据的操作过程中，设置数据的部分是最麻烦的，如果Emp类中有50个属性，那么对于整个程序将会出现有一堆的setter方法调用。或者再进一步说明，在实际的开发中，简单Java类的个数是非常多的，那么如果所有的简单的Java类都牵扯到属性赋值的时候，这种情况代码的编码的重复性将非常的高。

按照传统的直观的编程方式，所带来的问题就是代码会存在大量的重复操作，如果要想解决对象的重复处理操作，那么唯一的解决方案就是反射机制，反射机制最大的特征是可以根据其自身的特点（Object类直接操作，可以直接操作属性或方法）实现相同功能类的重复操作的抽象处理。



## 2、属性自动设置解决方案

**【本节目标】**本节介绍了属性自动赋值的实现思路，需要一个专门的ClassInstanceFactory类负责所有的反射处理，即：接收反射对象与要设置的属性内容，同时可以获取指定类的实例化对象。

经过分析后已经确认了当前简单Java类操作的问题所在，而对于开发者而言就需要想办法通过一种解决方案来实现属性内容的自动设置，那么这时的设置强烈建议采用字符串的形式来描述对应的类型。

1. 在进行程序开发时String字符串可以描述的内容有很多，并且也可以由开发者自行定义字符串的结构，下面就采用“**属性名:内容|属性名:内容|**”的形式来为简单Java类中的属性初始化。
2. 类设计的基本结构：应该由一个专门的ClassInstanceFactory类负责所有的反射处理，即：接收反射对象与要设置的属性内容，同时可以获取指定类的实例化对象。

反射与简单Java类：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/7fc10decffeb42bea773681d5d7a9b87.png)

设计的基本结构：

```java
package com.example;
class ClassInstanceFactory{
    private ClassInstanceFactory(){}

    /**
     * 实例化对象的创建方法，该对象可以根据传入的字符串结构："属性:内容|属性:内容"
     * @param clazz 要进行反射实例化的Class对象，有Class就可以反射实例化对象
     * @param value 要设置给对象的属性内容
     * @return 一个已经配置好属性内容的Java对象
     */
    public static <T> T create(Class<?> clazz,String value){
        return null;
    }
}
public class JavaAPIDemo {
    public static void main(String[] args)throws Exception{
        String value="ename:Sam|job:Java";
        Emp emp = ClassInstanceFactory.create(Emp.class, value);
        System.out.println("姓名：" + emp.getEname() + "、职位：" + emp.getJob());
    }
}
```

那么在当前的开发中，所需要完善的就是ClassInstanceFactory.create()处理方法。



## 3、单级属性赋值

本节需要掌握需要通过反射进行指定类对象的实例化处理，进行内容的设置（Field属性类型、方法名称、要设置的内容）

对于此时的Emp类中会发现所给出的数据类型都没有其他的引用关联了，只是描述了Empty本类的对象，所以这样的设置称为单级设置处理，所以此时应该处理两件事情：

1. 需要通过反射进行指定类对象的实例化处理
2. 进行内容的设置（Field属性类型、方法名称、要设置的内容）

反射与简单Java类：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/370ca16ec0f04be092e44b901753a1e2.png)

操作示例：

1. 定义StringUtils实现首字母大写功能
2. 定义BeanUtils工具类，该工具类主要实现属性的设置
3. ClassInstanceFactory负责实例化对象并且调用BeanUtils类实现属性内容的设置

```java
package com.example;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * 1、定义StringUtils实现首字母大写功能
 */
class StringUtils {
    public static String initCap(String str) {
        if (str == null || "".equals(str)) {
            return str;
        }
        if (str.length() == 1) {
            return str.toUpperCase();
        }else {
            return str.substring(0, 1).toUpperCase() + str.substring(1);
        }
    }
}

/**
 * 2、定义BeanUtils工具类，该工具类主要实现属性的设置
 */
class BeanUtils{
    private BeanUtils(){}
    /**
     * 实现指定对象的属性设置
     * @param obj 要进行反射操作的实例化对象
     * @param value 包含有指定内容的字符串，格式"属性:内容|属性:内容"
     */
    public static void setValue(Object obj,String value){
        // 按照"|"进行每一组属性的拆分
        String results [] = value.split("\\|");
        // 循环设置属性内容
        for (int x = 0; x < results.length; x ++) {
            // 获取“属性名称”和内容：attval [0]保存的是属性名称，attval [1]保存的是属性内容。
            String attval [] = results[x].split(":");
            try {
                // 获取成员
                Field field = obj.getClass().getDeclaredField(attval[0]);
                String methodName = "set" + StringUtils.initCap(attval [0]);
                Method setMethod = obj.getClass().getDeclaredMethod(methodName, field.getType());
                // 调用setter方法设置内容
                setMethod.invoke(obj, attval [1]);
            }catch (Exception e) {}
        }
    }
}

/**
 * 3、ClassInstanceFactory负责实例化对象并且调用BeanUtils类实现属性内容的设置
 */
class ClassInstanceFactory{
    private ClassInstanceFactory(){}
    /**
     * 实例化对象的创建方法，该对象可以根据传入的字符串结构："属性:内容|属性:内容"
     * @param clazz 要进行反射实例化的Class对象，有Class就可以反射实例化对象
     * @param value 要设置给对象的属性内容
     * @return 一个已经配置好属性内容的Java对象
     */
    public static <T> T create(Class<?> clazz,String value){
        // 如果要想采用反射进行简单Java类对象属性设置的时候，类中必须要有无参构造
        try {
            Object obj = clazz.getDeclaredConstructor().newInstance();
            // 通过反射设置属性
            BeanUtils.setValue(obj, value);
            // 返回对象
            return (T) obj;
        }catch (Exception e) {
            // 如果此时真的出现了错误，本质上抛出异常也没用
            e.printStackTrace();
            return null;
        }
    }
}
```

即使类中的属性再多，那么也可以轻松的实现setter的调用（类对象实例化处理）



## 4、设置多种数据类型

**【本节目标】**在实际的开发中面对简单Java类中的属性类型一般的可选为：long（Long）、int（Integer）、double（Double）、String、Date（日期、日期时间），我们需要实现各种数据类型的配置。

现在已经成功的实现了单级的属性配置，但是这里面依然需要考虑一个实际的情况：当前所给定的数据类型只是String。在实际的开发中面对简单Java类中的属性类型一般的可选为：long（Long）、int（Integer）、double（Double）、String、Date（日期、日期时间），所以这时对于当前的程序代码必须做出修改，要求可以实现各种数据类型的配置。

既然要求可以实现不同类型的内容设置，并且BeanUtils类主要是完成属性赋值处理的，那么就可以在这个类中追加一系列的处理方法。

```java
package com.example;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
class Emp {
    private Long empno;
    private String ename;
    private String job;
    private double salary;
    private Date hiredate;
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setJob(String job) {
        this.job = job;
    }
    public String getEname() {
        return ename;
    }
    public String getJob() {
        return job;
    }
    public long getEmpno() {
        return empno;
    }
    public void setEmpno(Long empno) {
        this.empno = empno;
    }
    public double getSalary() {
        return salary;
    }
    public void setSalary(double salary) {
        this.salary = salary;
    }
    public Date getHiredate() {
        return hiredate;
    }
    public void setHiredate(Date hiredate) {
        this.hiredate = hiredate;
    }
}

/**
 * 1、定义StringUtils实现首字母大写功能
 */
class StringUtils {
    public static String initCap(String str) {
        if (str == null || "".equals(str)) {
            return str;
        }
        if (str.length() == 1) {
            return str.toUpperCase();
        } else {
            return str.substring(0, 1).toUpperCase() + str.substring(1);
        }
    }
}

/**
 * 2、定义BeanUtils工具类，该工具类主要实现属性的设置
 */
class BeanUtils {
    private BeanUtils() {}
    /**
     * 实现指定对象的属性设置
     * @param obj   要进行反射操作的实例化对象
     * @param value 包含有指定内容的字符串，格式"属性:内容|属性:内容"
     */
    public static void setValue(Object obj, String value) {
        // 按照"|"进行每一组属性的拆分
        String[] results = value.split("\\|");
        // 循环设置属性内容
        for (int x = 0; x < results.length; x++) {
            // 获取“属性名称”和内容：attval [0]保存的是属性名称，attval [1]保存的是属性内容
            String[] attval = results[x].split(":");
            try {
                // 获取成员
                Field field = obj.getClass().getDeclaredField(attval[0]);
                // 获得方法
                String methodName = "set" + StringUtils.initCap(attval[0]);
                Method setMethod = obj.getClass().getDeclaredMethod(methodName, field.getType());
                Object convertValue = BeanUtils.convertAttributeValue(field.getType().getName(), attval[1]);
                // 调用setter方法设置内容
                setMethod.invoke(obj, convertValue);
            } catch (Exception e) {
            }
        }
    }

    /**
     * 实现属性类型转换处理
     * @param type  属性类型，通过Field获取
     * @param value 属性的内容，传入的都是字符串，需要将其变为指定类型
     * @return 转换后的数据类型
     */
    private static Object convertAttributeValue(String type, String value) {
        if ("long".equals(type) || "java.lang.Long".equals(type)) {
            return Long.parseLong(value);
        } else if ("int".equals(type) || "java.lang.int".equals(type)) {
            return Integer.parseInt(value);
        } else if ("double".equals(type) || "java.lang.double".equals(type)) {
            return Double.parseDouble(value);
        } else if ("java.util.Date".equals(type)) {
            // 日期类型
            SimpleDateFormat sdf = null;
            if (value.matches("\\d{4}-\\d{2}-\\d{2}")) {
                sdf = new SimpleDateFormat("yyyy-MM-dd");
            } else if (value.matches("\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2}")) {
                sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            }else{
                // 日期类型
                return new Date();
            }
            try {
                return sdf.parse(value);
            } catch (ParseException e) {
                // 当前日期
                return new Date();
            }
        } else {
            return value;
        }
    }
}

/**
 * 3、ClassInstanceFactory负责实例化对象并且调用BeanUtils类实现属性内容的设置
 */
class ClassInstanceFactory {
    private ClassInstanceFactory() {
    }
    /**
     * 实例化对象的创建方法，该对象可以根据传入的字符串结构："属性:内容|属性:内容"
     * @param clazz 要进行反射实例化的Class对象，有Class就可以反射实例化对象
     * @param value 要设置给对象的属性内容
     * @return 一个已经配置好属性内容的Java对象
     */
    public static <T> T create(Class<?> clazz, String value) {
        // 如果要想采用反射进行简单Java类对象属性设置的时候，类中必须要有无参构造
        try {
            Object obj = clazz.getDeclaredConstructor().newInstance();
            // 通过反射设置属性
            BeanUtils.setValue(obj, value);
            // 返回对象
            return (T) obj;
        } catch (Exception e) {
            // 如果此时真的出现了错误，本质上抛出异常也没用
            e.printStackTrace();
            return null;
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String value = "empno:7369|ename:Smith|job:Clerk|salary:750.00|hiredate:1989-10-10";
        Emp emp = ClassInstanceFactory.create(Emp.class, value);
        System.out.println("雇员编号：" + emp.getEmpno() + "、姓名：" + emp.getEname() + "、职位：" + emp.getJob() + "、基本工资：" + emp.getSalary() + "、受雇日期：" + emp.getHiredate());
    }
}

// 输出内容
雇员编号：7369、姓名：Smith、职位：Clerk、基本工资：750.0、受雇日期：Tue Oct 10 00:00:00 CST 1989
```

此时只是列举了常用几种数据类型，如果要将其作为产品推广，必须要考虑所有可能出现的类型，同时也要考虑所有可能会出现的日期格式。



## 5、级联对象实例化

本节需要掌握实现级联对象实例化以及实现对象的级联属性设置。

如果现在给定的类对象中存在有其它的引用的级联关系的情况下，称为多级设置。例如：一个雇员属于一个部门，一个部分属于一个公司，所以这时对于简单Java类的基本关系定义如下：

```java
class Company{
    private String name;
    private Date createdate;
    public String getName() {
        return name;
    }
    public void setname(String name) {
        this.name = name;
    }
    public Date getCreatedate() {
        return createdate;
    }
    public void setCreatedate(Date createdate) {
        this.createdate = createdate;
    }
}
class Dept{
    private String dname;
    private String loc;
    private Company company;
    public String getDname() {
        return dname;
    }
    public void setDname(String dname) {
        this.dname = dname;
    }
    public String getLoc() {
        return loc;
    }
    public void setLoc(String loc) {
        this.loc = loc;
    }
    public Company getCompany() {
        return company;
    }
    public void setCompany(Company company) {
        this.company = company;
    }
}
class Emp{
    private long empno;
    private String ename;
    private String job;
    private double salary;
    private Date hiredate;
    private Dept dept;
    public Dept getDept() {
        return dept;
    }
    public void setDept(Dept dept) {
        this.dept = dept;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setJob(String job) {
        this.job = job;
    }
    public String getEname() {
        return ename;
    }
    public String getJob() {
        return job;
    }
    public long getEmpno() {
        return empno;
    }
    public void setEmpno(Long empno) {
        this.empno = empno;
    }
    public double getSalary() {
        return salary;
    }
    public void setSalary(double salary) {
        this.salary = salary;
    }
    public Date getHiredate() {
        return hiredate;
    }
    public void setHiredate(Date hiredate) {
        this.hiredate = hiredate;
    }
}
```

如果要通过Emp进行操作，则应该使用“.”作为级联关系的处理：

1. dept.dname:财务部：  Emp类实例化对象.getDept().setDname("财务部")
2. dept.company.name:XXXX：Emp类实例化对象.getDept()..getCompany().setName("XXXX")

考虑到代码的简洁性，所以应该考虑可以通过级联的配置自动实现类中属性的实例化。

```java
String value="empno:7369|ename:Sam|job:Java|salary:750.00|hiredate:1989-10-10|" + "dept.dname:财务部|dept.company.name:XXXX";
```

现在的属性存在有多级的关系，那么对于多级的关系就必须与单级的配置区分开：

```java
package com.example;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

class Company{
    private String name;
    private Date createdate;
    public String getName() {
        return name;
    }
    public void setname(String name) {
        this.name = name;
    }
    public Date getCreatedate() {
        return createdate;
    }
    public void setCreatedate(Date createdate) {
        this.createdate = createdate;
    }
}
class Dept{
    private String dname;
    private String loc;
    private Company company;
    public String getDname() {
        return dname;
    }
    public void setDname(String dname) {
        this.dname = dname;
    }
    public String getLoc() {
        return loc;
    }
    public void setLoc(String loc) {
        this.loc = loc;
    }
    public Company getCompany() {
        return company;
    }
    public void setCompany(Company company) {
        this.company = company;
    }
}
class Emp{
    private Long empno;
    private String ename;
    private String job;
    private double salary;
    private Date hiredate;
    private Dept dept;
    public Long getEmpno() {
        return empno;
    }
    public void setEmpno(Long empno) {
        this.empno = empno;
    }
    public String getEname() {
        return ename;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public String getJob() {
        return job;
    }
    public void setJob(String job) {
        this.job = job;
    }
    public double getSalary() {
        return salary;
    }
    public void setSalary(double salary) {
        this.salary = salary;
    }
    public Date getHiredate() {
        return hiredate;
    }
    public void setHiredate(Date hiredate) {
        this.hiredate = hiredate;
    }
    public Dept getDept() {
        return dept;
    }
    public void setDept(Dept dept) {
        this.dept = dept;
    }
}

class ClassInstanceFactory{
    private ClassInstanceFactory(){}
    /**
     * 实例化对象的创建方法，该对象可以根据传入的字符串结构："属性:内容|属性:内容"
     * @param clazz 要进行反射实例化的Class对象，有Class就可以反射实例化对象
     * @param value 要设置给对象的属性内容
     * @return 一个已经配置好属性内容的Java对象
     */
    public static <T> T create(Class<?> clazz,String value){
        // 如果要想采用反射进行简单Java类对象属性设置的时候，类中必须要有无参构造
        try {
            Object obj = clazz.getDeclaredConstructor().newInstance();
            // 通过反射设置属性
            BeanUtils.setValue(obj, value);
            // 返回对象
            return (T) obj;
        }catch (Exception e) {
            // 如果此时真的出现了错误，本质上抛出异常也没用
            e.printStackTrace();
            return null;
        }
    }
}

class StringUtils {
    public static String initCap(String str) {
        if (str == null || "".equals(str)) {
            return str;
        }
        if (str.length() == 1) {
            return str.toUpperCase();
        }else {
            return str.substring(0, 1).toUpperCase() + str.substring(1);
        }
    }
}

class BeanUtils {    // 进行Bean处理的类
    private BeanUtils() {}
    /**
     * 实现指定对象的属性设置
     * @param obj   要进行反射操作的实例化对象
     * @param value 包含有指定内容的字符串，格式“属性:内容|属性:内容”
     */
    public static void setValue(Object obj, String value) {
        // 按照“|”进行每一组属性的拆分
        String results[] = value.split("\\|");
        // 循环设置属性内容
        for (int x = 0; x < results.length; x++) {
            // a获取“属性名称”与内容：ttval[0]保存的是属性名称、attval[1]保存的是属性内容
            String[] attval = results[x].split(":");
            try {
                // 多级配置
                if (attval[0].contains(".")) {
                    String temp[] = attval[0].split("\\.");
                    Object currentObject = obj;
                    // 实例化对象：最后一位肯定是指定类中的属性名称，所以不在本次实例化处理的范畴之内
                    for (int y = 0; y < temp.length - 1; y++) {
                        // 调用相应的getter方法，如果getter方法返回了null表示该对象未实例化
                        Method getMethod = currentObject.getClass().getDeclaredMethod("get" + StringUtils.initCap(temp[y]));
                        Object tempObject = getMethod.invoke(currentObject);
                        // 该对象现在并没有实例化
                        if (tempObject == null) {
                            // 获取属性类型
                            Field field = currentObject.getClass().getDeclaredField(temp[y]);
                            Method method = currentObject.getClass().getDeclaredMethod("set" + StringUtils.initCap(temp[y]), field.getType());
                            Object newObject = field.getType().getDeclaredConstructor().newInstance();
                            method.invoke(currentObject, newObject);
                            currentObject = newObject;
                        } else {
                            currentObject = tempObject;
                        }
                        System.out.println(temp[y] + "--" + currentObject);
                    }
                } else {
                    // 获取成员
                    Field field = obj.getClass().getDeclaredField(attval[0]);
                    Method setMethod = obj.getClass().getDeclaredMethod("set" + StringUtils.initCap(attval[0]), field.getType());
                    Object convertValue = BeanUtils.convertAttributeValue(field.getType().getName(), attval[1]);
                    // 调用setter方法设置内容
                    setMethod.invoke(obj, convertValue);
                }
            } catch (Exception e) {
            }
        }
    }

    /**
     * 实现属性类型转换处理
     *
     * @param type  属性类型，通过Field获取的
     * @param value 属性的内容，传入的都是字符串，需要将其变为指定类型
     * @return 转换后的数据
     */
    private static Object convertAttributeValue(String type, String value) {
        if ("long".equals(type) || "java.lang.Long".equals(type)) {
            return Long.parseLong(value);
        } else if ("int".equals(type) || "java.lang.int".equals(type)) {
            return Integer.parseInt(value);
        } else if ("double".equals(type) || "java.lang.double".equals(type)) {
            return Double.parseDouble(value);
        } else if ("java.util.Date".equals(type)) {
            SimpleDateFormat sdf = null;
            if (value.matches("\\d{4}-\\d{2}-\\d{2}")) {
                sdf = new SimpleDateFormat("yyyy-MM-dd");
            } else if (value.matches("\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2}")) {
                sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            } else {
                // 当前日期
                return new Date();
            }
            try {
                return sdf.parse(value);
            } catch (ParseException e) {
                // 当前日期
                return new Date();
            }
        } else {
            return value;
        }
    }
}


public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String value = "empno:7369|ename:Sam|job:Java|salary:750.00|hiredate:1999-10-10|" + "dept.dname:财务部|dept.company.name:XXXX";
        Emp emp = ClassInstanceFactory.create(Emp.class, value);
        System.out.println("雇员编号：" + emp.getEmpno() + "、姓名：" + emp.getEname() + "、职位：" + emp.getJob() + "、基本工资：" + emp.getSalary() + "、受雇日期：" + emp.getHiredate());
        System.out.println(emp.getDept());
        System.out.println(emp.getDept().getCompany());
    }
}

// 输出内容
dept--com.example.Dept@6eebc39e
dept--com.example.Dept@6eebc39e
company--com.example.Company@f6c48ac
雇员编号：7369、姓名：Sam、职位：Java、基本工资：750.0、受雇日期：Sun Oct 10 00:00:00 CST 1999
com.example.Dept@6eebc39e
com.example.Company@f6c48ac
```



## 6、级联属性设置

现在已经成功的实现级联的对象实例化处理，那么随后就需要去考虑级联的属性的设置了，在之前考虑级联对象实例化处理时，循环时都是少了一位的。

```java
for (int y = 0 ; y < temp.length - 1 ; y ++) {  // 实例化
    // 调用了相应的getter方法，如果getter方法返回了null，表示该对象未实例化
    Method getMethod = obj.getClass().getDeclaredMethod("get" + StringUtils.initcap(temp[y]));
    Object tempObject = getMethod.invoke(currentObject); 
    if (tempObject == null) {    // 该对象现在并没有实例化
        Field field = currentObject.getClass().getDeclaredField(temp[y]);  // 获取属性类型
        Method method = currentObject.getClass().getDeclaredMethod("set" + StringUtils.initcap(temp[y]), field.getType());
        Object newObject = field.getType().getDeclaredConstructor().newInstance();
        method.invoke(currentObject, newObject);
        currentObject = newObject;
    }else {
        currentObject = tempObject;
    }
}
```

当此时代码循环处理完成之后，currentObject表示的就是可以进行setter方法调用的对象了，并且理论上该对象一定不可能为null，随后就可以按照我们之前的方式利用对象进行setter方法调用。

操作示例：实现对象的级联属性设置

```java
//进行属性内容的设置
Field field = currentObject.getClass().getDeclaredField(temp[temp.length - 1]);  //获取成员
Method setMethod = currentObject.getClass().getDeclaredMethod("set" + StringUtils.initcap(temp[temp.length - 1]), field.getType());
Object convertValue = BeanUtils.convertAttributeValue(field.getType().getName(), attval[1]);
setMethod.invoke(currentObject, convertValue);  //调用setter方法设置内容
```

操作示例：完整代码

```java
package com.example;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

class ClassInstanceFactory {
    private ClassInstanceFactory() {}
    /**
     * 实例化对象的创建方法，该对象可以根据传入的字符串结构“属性:内容|属性:内容”
     * @param clazz 要进行反射实例化的Class类对象，有Class就可以反射实例化对象
     * @param value 要设置给对象的属性内容
     * @return 一个已经配置好属性内容的Java类对象
     */
    public static <T> T create(Class<?> clazz, String value) {
        try {
            // 如果要想采用反射进行简单Java类对象属性设置的时候，类中必须要有无参构造
            Object obj = clazz.getDeclaredConstructor().newInstance();
            // 通过反射设置属性
            BeanUtils.setValue(obj, value);
            // 返回对象
            return (T) obj;
        } catch (Exception e) {
            // 如果此时真的出现了错误，本质上抛出异常也没用
            e.printStackTrace();
            return null;
        }
    }
}

class StringUtils {
    public static String initcap(String str) {
        if (str == null || "".equals(str)) {
            return str;
        }
        if (str.length() == 1) {
            return str.toUpperCase();
        } else {
            return str.substring(0, 1).toUpperCase() + str.substring(1);
        }
    }
}

class BeanUtils {
    private BeanUtils() {}
    /**
     * 实现指定对象的属性设置
     * @param obj   要进行反射操作的实例化对象
     * @param value 包含有指定内容的字符串，格式“属性:内容|属性:内容”
     */
    public static void setValue(Object obj, String value) {
        // 按照“|”进行每一组属性的拆分
        String results[] = value.split("\\|");
        // 循环设置属性内容
        for (int x = 0; x < results.length; x++) {
            // 获取“属性名称”与内容：attval[0]保存的是属性名称、attval[1]保存的是属性内容
            String attval[] = results[x].split(":");
            try {
                // 多级配置
                if (attval[0].contains(".")) {
                    String temp[] = attval[0].split("\\.");
                    Object currentObject = obj;
                    // 实例化对象：最后一位肯定是指定类中的属性名称，所以不在本次实例化处理的范畴之内
                    for (int y = 0; y < temp.length - 1; y++) {
                        // 调用相应的getter方法，如果getter方法返回了null表示该对象未实例化
                        Method getMethod = currentObject.getClass().getDeclaredMethod("get" + StringUtils.initcap(temp[y]));
                        Object tempObject = getMethod.invoke(currentObject);
                        // 该对象现在并没有实例化
                        if (tempObject == null) {
                            // 获取属性类型
                            Field field = currentObject.getClass().getDeclaredField(temp[y]);
                            Method method = currentObject.getClass().getDeclaredMethod("set" + StringUtils.initcap(temp[y]), field.getType());
                            Object newObject = field.getType().getDeclaredConstructor().newInstance();
                            method.invoke(currentObject, newObject);
                            currentObject = newObject;
                        } else {
                            currentObject = tempObject;
                        }
                    }
                    // 进行属性内容的设置，获取成员
                    Field field = currentObject.getClass().getDeclaredField(temp[temp.length - 1]);
                    Method setMethod = currentObject.getClass().getDeclaredMethod("set" + StringUtils.initcap(temp[temp.length - 1]), field.getType());
                    Object convertValue = BeanUtils.convertAttributeValue(field.getType().getName(), attval[1]);
                    // 调用setter方法设置内容
                    setMethod.invoke(currentObject, convertValue);
                } else {
                    // 获取成员
                    Field field = obj.getClass().getDeclaredField(attval[0]);
                    Method setMethod = obj.getClass().getDeclaredMethod("set" + StringUtils.initcap(attval[0]), field.getType());
                    Object convertValue = BeanUtils.convertAttributeValue(field.getType().getName(), attval[1]);
                    // 调用setter方法设置内容
                    setMethod.invoke(obj, convertValue);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 实现属性类型转换处理
     *
     * @param type  属性类型，通过Field获取的
     * @param value 属性的内容，传入的都是字符串，需要将其变为指定类型
     * @return 转换后的数据
     */
    private static Object convertAttributeValue(String type, String value) {
        if ("long".equals(type) || "java.lang.Long".equals(type)) {
            return Long.parseLong(value);
        } else if ("int".equals(type) || "java.lang.int".equals(type)) {
            return Integer.parseInt(value);
        } else if ("double".equals(type) || "java.lang.double".equals(type)) {
            return Double.parseDouble(value);
        } else if ("java.util.Date".equals(type)) {
            SimpleDateFormat sdf = null;
            if (value.matches("\\d{4}-\\d{2}-\\d{2}")) {
                sdf = new SimpleDateFormat("yyyy-MM-dd");
            } else if (value.matches("\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2}")) {
                sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            } else {
                // 当前日期
                return new Date();
            }
            try {
                return sdf.parse(value);
            } catch (ParseException e) {
                // 当前日期
                return new Date();
            }
        } else {
            return value;
        }
    }
}

class Company {
    private String name;
    private Date createdate;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Date getCreatedate() {
        return createdate;
    }
    public void setCreatedate(Date createdate) {
        this.createdate = createdate;
    }
}

class Dept {
    private String dname;
    private String loc;
    private Company company;
    public String getDname() {
        return dname;
    }
    public void setDname(String dname) {
        this.dname = dname;
    }
    public String getLoc() {
        return loc;
    }
    public void setLoc(String loc) {
        this.loc = loc;
    }
    public Company getCompany() {
        return company;
    }
    public void setCompany(Company company) {
        this.company = company;
    }
}

class Emp {
    private long empno;
    private String ename;
    private String job;
    private double salary;
    private Date hiredate;
    private Dept dept;
    public Dept getDept() {
        return dept;
    }
    public void setDept(Dept dept) {
        this.dept = dept;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setJob(String job) {
        this.job = job;
    }
    public String getEname() {
        return ename;
    }
    public String getJob() {
        return job;
    }
    public long getEmpno() {
        return empno;
    }
    public void setEmpno(long empno) {
        this.empno = empno;
    }
    public double getSalary() {
        return salary;
    }
    public void setSalary(double salary) {
        this.salary = salary;
    }
    public Date getHiredate() {
        return hiredate;
    }
    public void setHiredate(Date hiredate) {
        this.hiredate = hiredate;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String value = "empno:7369|ename:Smith|job:Clerk|salary:750.00|hiredate:1989-10-10|" +
                "dept.dname:财务部|dept.company.name:MLDN";
        Emp emp = ClassInstanceFactory.create(Emp.class, value);
        System.out.println("雇员编号：" + emp.getEmpno() + "姓名：" + emp.getEname() + "、职位：" + emp.getJob() + "、基本工资："
                + emp.getSalary() + "、雇佣日期：" + emp.getHiredate());
        System.out.println(emp.getDept().getDname());
        System.out.println(emp.getDept().getCompany().getName());
    }
}
```

在以后的开发中简单Java类的赋值处理将不再重复调用setter操作完成，而这种处理形式是在正规开发中普遍采用的方式。

# ClassLoader类加载器

本节介绍了ClassLoader类加载器，其中自定义的类加载器其加载的顺序是在所有系统类加载器的最后。系统类中的类加载器都是根据CLASSPATH路径进行加载的，而如果有了自定义的类加载器，就可以由开发者任意指定类的加载位置。

## 1、类加载器简介

在Java语言中提供了一个系统的环境变量：CLASSPATH，这个环境属性的作用主要是在JVM进程启动时进行类加载路径的定义，在JVM中可以根据类加载器而后进行指定路径中类的加载，也就是说找到了类的加载器就意味着找到了类的来源。ClassLoader类加载器图如下：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/316ff411884049efa150a27697f0bdae.png)



## 2、系统类加载器

如果现在要想获得类的加载器，那么一定要通过ClassLoader来获取，而要想获取ClassLoader类的对象，则必须利用Class类（反射的根源）实现，方法：`public ClassLoader getClassLoader()，`当获取了ClassLoader后还可以获取其父类的ClassLoader类对象：`public final ClassLoader getParent()`

操作示例 1：获取系统类加载器、自定义类加载器

```java
package com.example;
class Message {}
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        {
            // 获取当前类加载器（自定义类加载器）
            // 1.8：sun.misc.Launcher$AppClassLoader@6659c656
            // 1.9+：jdk.internal.loader.ClassLoaders$AppClassLoader@4f8e5cde
            System.out.println(Message.class.getClassLoader());
            // 获取父类加载器
            // 1.8：sun.misc.Launcher$ExtClassLoader@60e53b93
            // 1.9+：jdk.internal.loader.ClassLoaders$PlatformClassLoader@5d3411d
            System.out.println(Message.class.getClassLoader().getParent());
            // 获取祖父类加载器：null
            System.out.println(Message.class.getClassLoader().getParent().getParent());
        }
        {
            // 获取String类对应的类加载器：null，因为Bootstrap根加载器不是由Java编写所以只能返回null
            String str = "example";
            System.out.println(str.getClass().getClassLoader());
        }
    }
}

// 输出内容
jdk.internal.loader.ClassLoaders$AppClassLoader@4629104a
jdk.internal.loader.ClassLoaders$PlatformClassLoader@1c53fd30
null
null
```

从JDK1.8之后的版本（JDK1.9，JDK1.10）提供有一个“PlatformClassLoader”类加载器，而在JDK1.8及以前的版本中提供的加载器为“ ExtClassLoader”，因为在JDK的安装目录中提供了一个ext的目录，开发者可以将*.jar文件拷贝到此目录中，这样就可以直接执行了，但是这样的处理开发并不安全，最初的时候也是不提倡使用的，所以从JDK1.9开始将其彻底废除了，同时为了与系统类加载器和应用类加载器之间保持设计的平衡，提供有平台类加载器。

![image.png](https://ucc.alicdn.com/pic/developer-ecology/0758764a15a84fc3a4d603c72995cb31.png)

当你获得了类加载器后就可以利用类加载器来实现类的反射加载处理：

```java
protected Class<?> findClass(String name) throws ClassNotFoundException;
```



## 3、自定义类加载器

清楚了类加载器的功能后就可以根据自身的需求来实现自定义的类加载器，但是千万要记住一点：自定义的类加载器其加载的顺序是在所有系统类加载器的最后。系统类中的类加载器都是根据CLASSPATH路径进行加载的，而如果有了自定义的类加载器，就可以由开发者任意指定类的加载位置。

自定义类加载器：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/1de67f30666846518ae5564dce2fd449.png)

1、随意编写一个程序类，并且将这个类保存在D盘（D:\Message.java）

```java
package com.example;
public class Message {
    public void send() {
        System.out.println("www.example.cn");
    }
}
```

2、将此类直接拷贝到系统磁盘上（非项目路径）进行编译处理，只编译不打包：javac Message.java，此时并没有进行打包处理，所以这个类无法通过CLASSPATH正常加载。

3、自定义一个类加载器，并且继承自ClassLoader类。在ClassLoader类中提供有一个字节转换为类结构的方法：

```java
public abstract class ClassLoader {
    protected final Class<?> defineClass(String name, byte[] b, int off, int len)
        throws ClassFormatError
    {
        return defineClass(name, b, off, len, null);
    }
    //...其他方法省略
}
```

自定义类：

```java
package com.example;
import java.io.*;
public class ExampleClassLoader extends ClassLoader {
    private static final String MESSAGE_CLASS_PATH = "D:\\Message.class";
    /**
     * 进行指定类的加载
     * @param className 类的完整名称“包.类”
     * @return 返回一个指定类的Class对象
     * @throws Exception 如果类文件不存在，则无法加载
     */
    public Class<?> loadData(String className) throws Exception {
        // 读取二进制数据文件
        byte[] data = loadClassData();
        // 读取到了
        if (data != null) {
            return super.defineClass(className, data, 0, data.length); 
        }
        return null;
    }
    // 通过文件进行类的加载
    private byte[] loadClassData() throws Exception {
        InputStream input = null;
        // 将数据加载到内存之中
        ByteArrayOutputStream bos = null;
        byte data [] = null;
        try {
            // 实例化内存流
            bos = new ByteArrayOutputStream();
            // 文件加载流
            input = new FileInputStream(new File(MESSAGE_CLASS_PATH));
            // 读取数据
            input.transferTo(bos);
            // 将所有读取到的字节数取出
            data = bos.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (input != null) {
                input.close();
            }
            if (bos != null) {
                bos.close();
            }
        }
        return data;
    }
}
```

4、编写测试类实现类加载控制。

```java
package com.example;
import java.lang.reflect.Method;
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 实例化自定义类加载器
        ExampleClassLoader classLoader = new ExampleClassLoader();
        // 进行类的加载
        Class<?> cls = classLoader.loadData("com.example.Message");
        // 由于Message类不在CLASSPATH中，所以此时无法直接将对象转为Message类型，只能通过反射调用
        Object obj = cls.getDeclaredConstructor().newInstance(); // 实例化对象
        Method method = cls.getDeclaredMethod("send"); // 获取方法
        method.invoke(obj); // 方法调用
    }
}

// 输出内容
www.example.cn
```

如果在以后结合网络程序开发的话，就可以通过一个远程的服务器来确定一个类的功能。

![image.png](https://ucc.alicdn.com/pic/developer-ecology/f4770756ef7d4943955cc7170db96f41.png)

5、观察当前的Message类的加载器的情况

```java
package com.example;
public class JavaAPIDemo {
    public static void main(String[] args)throws Exception{
        // 实例化自定义类加载器
        ExampleClassLoader classLoader = new ExampleClassLoader();
        Class<?> cls=classLoader.loadData("com.example.Message");
        // com.example.ExampleClassLoader@27f674d
        System.out.println(cls.getClassLoader());
        // jdk.internal.loader.ClassLoaders$AppClassLoader@2437c6dc
        System.out.println(cls.getClassLoader().getParent());
        // jdk.internal.loader.ClassLoaders$PlatformClassLoader@58ceff1
        System.out.println(cls.getClassLoader().getParent().getParent());
    }
}

// 输出内容
com.example.ExampleClassLoader@27f674d
jdk.internal.loader.ClassLoaders$AppClassLoader@2437c6dc
jdk.internal.loader.ClassLoaders$PlatformClassLoader@58ceff1
```

如果现在定义了一个类：java.lang.String，并且利用了自定义的类加载器进行加载处理，这个类将不会被加载，Java之中针对于类加载器提供有双亲加载机制，如果现在要加载的程序类是由系统提供的类则会由系统类加载器进行加载，在开发者定义的类与系统类名称相同，那么为了保证系统的安全性绝对不会加载。



# 反射与代理设计模式

## 1、静态代理设计模式

本节介绍了静态代理设计模式的弊端以及解决方案：实现动态代理机制。

代理设计模式是在程序开发中使用最多的设计模式，代理设计模式的核心是有真实业务实现类和代理业务实现类，并且代理类要完成比真实业务更多的处理操作。

> 传统代理设计模式的弊端

所有的代理设计模式如果按照设计要求来说，必须是基于接口的设计，也就是说需要首先定义出核心接口的组成。

操作示例 1：模拟一个消息发送的代理操作结构（传统代理设计）

```java
package com.example;
/**
 * 传统代理设计必须有接口
 */
interface IMessage{
    public void send();
}
class MessageReal implements IMessage {
    @Override
    public void send() {
        System.out.println("【发送消息】www.xxxx.cn");
    }
}

/**
 * 代理类
 */
class MessageProxy implements IMessage {
    // 代理对象，一定是业务接口实例
    private IMessage message;
    public MessageProxy(IMessage message) {
        this.message = message;
    }
    @Override
    public void send() {
        if(this.connect()){
            // 消息的发送处理
            this.message.send();
            this.close();
        }
    }
    public boolean connect(){
        System.out.println("【消息代理】进行消息发送通道的连接。");
        return true;
    }
    public void close(){
        System.out.println("【消息代理】关闭消息通道。");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        IMessage message = new MessageProxy(new MessageReal());
        message.send();
    }
}

// 输出内容
【消息代理】进行消息发送通道的连接。
【发送消息】www.xxxx.cn
【消息代理】关闭消息通道。
```

传统代理设计：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/2d7906b1be244235bfa7407efad0f361.png)

以上的操作代码是一个最为标准的代理设计，但是如果要进一步的去思考会发现客户端的接口与具体的子类产生了耦合问题，所以这样的操作如果从实际的开发来讲，最好再引入工厂设计模式进行代理对象的获取。

以上的代理设计模式为静态代理设计，这种静态代理涉及的特点在于：一个代理类只为一个接口服务，如果现在准备有3000个业务接口，则按照此种做法就意味着需要编写3000个代理类，并且这些代理类操作形式类似。

所以现在需要解决的问题在于：如何可以让一个代理类满足于所有的业务接口操作要求。



## 2、动态代理设计模式

通过静态代理设计模式的缺陷可以发现，最好的做法是为所有功能一致的业务操作接口提供有统一的代理处理操作，而这就可以通过动态代理机制来实现，但是在动态代理机制中需要考虑到如下几点问题：

1. 不管是动态代理类还是静态代理类都一定要接收真实业务实现子类对象
2. 由于动态代理类不再与某一个具体的接口进行捆绑，所以应该可以动态获取类的接口信息

动态代理设计模式：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/77c6189e21d3434188ef44e5d1174581.png)

在进行动态代理实现的操作中，首先需要关注的就是一个InvocationHandler接口，这个接口规定了代理方法的执行。

```java
public interface InvocationHandler{
    /**
     * 代理方法调用，代理主体类中执行的方法最终都是此方法
     * @param proxy 要代理的对象
     * @param method 要执行的接口方法名称
     * @param args 传递的参数
     * @return 某一个方法的返回值
     * @throws Throwable 方法调用时出现的错误继续向上抛出
     */
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

在进行动态代理设计时，对于动态对象的创建是由JVM底层完成的，此时主要依靠的是java.lang.reflect.Proxy程序类，而这个类中只提供了一个核心方法：

代理对象：`public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`

1. ClassLoader loader：获取当前真实主体类的ClassLoader
2. Class<?>[] interfaces：代理是围绕接口进行的，所以一定要获取真实主体类的接口信息
3. InvocationHandler h：代理处理的方法

代理方法：

![image.png](https://ucc.alicdn.com/pic/developer-ecology/60725c9ee29941aea3f8641bdf8de23f.png)

操作示例 2：实现动态代理机制

```java
package com.example;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 传统代理设计必须有接口
 */
interface IMessage {
    void send();
}
class MessageReal implements IMessage {
    @Override
    public void send() {
        System.out.println("【发送消息】www.xxxx.cn");
    }
}

class ExampleProxy implements InvocationHandler {
    // 保存真实业务对象
    private Object target;
    /**
     * 进行真实业务对象与代理业务对象之间的绑定处理
     * @param target 真实业务对象
     * @return Proxy生成的代理业务对象
     */
    public Object bind(Object target) {
        this.target = target;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }
    public boolean connect() {
        System.out.println("【消息代理】进行消息发送通道的连接。");
        return true;
    }
    public void close() {
        System.out.println("【消息代理】关闭消息通道。");
    }

    @Override
    public Object invoke(Object pro, Method method, Object[] args) throws Throwable {
        System.out.println("*****【执行方法: 】" + method);
        Object returnData = null;
        if (this.connect()) {
            returnData = method.invoke(this.target, args);
            this.close();
        }
        return returnData;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        IMessage msg = (IMessage) new ExampleProxy().bind(new MessageReal());
        msg.send();
    }
}

// 输出内容
【消息代理】进行消息发送通道的连接。
【发送消息】www.xxxx.cn
【消息代理】关闭消息通道。
```

如果认真观察系统中提供的Proxy.newProxyInstance()方法，会发现该方法会使用大量的底层机制来进行代理对象的动态创建，所有的代理类是符合所有相关功能需求的操作功能类，它不再代表具体的接口，这样在处理时就必须依赖于类加载器与接口进行代理对象的伪造。



## 3、CGLIB实现代理设计模式

本节介绍了利用第三方包：CGLIB开发包来实现代理设计模式。

从Java的官方来讲，已经明确的要求了如果想要实现代理设计模式，那么一定是基于接口的应用，所以在官方给出的Proxy类创建代理对象时，都需要传递该对象所有的接口信息：

```java
Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),this);
```

但是这个时候有一部分开发者就认为不应该强迫性的基于接口实现代理设计，所以一部分开发者就开发出了一个CGLIB开发包，利用这个开发包就可以实现基于类的代理设计模式。

1、CGLIB是一个第三方的程序包，需要单独在IDE中进行配置，或者直接在pom.xml引入依赖

```xml
<!-- https://mvnrepository.com/artifact/cglib/cglib-nodep -->
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib-nodep</artifactId>
    <version>3.3.0</version>
</dependency>
```

2、编写程序类，该类不实现任何接口

```java
package com.example;
class Message {
    public void send() {
        System.out.println("【发送消息】www.xxxx.cn");
    }
}
```

3、利用CGLIB编写代理类，但是这个代理类需要做一个明确，此时相当于使用了类的形式实现了代理设计的处理，所以该代理设计需要通过CGLIB来生成代理对象，定义一个代理类：

```java
package com.example;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

class Message {
    public void send() {
        System.out.println("【发送消息】www.xxxx.cn");
    }
}

/**
 * 拦截器配置
 */
class ExampleProxy implements MethodInterceptor {
    // 保存真实主题对象
    private Object target;
    public ExampleProxy(Object target) {
        this.target = target;
    }
    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        Object returnData = null;
        if (this.connect()) {
            returnData = method.invoke(this.target, args);
            this.close();
        }
        return returnData;
    }
    public boolean connect() {
        System.out.println("【消息代理】进行消息发送通道的连接。");
        return true;
    }
    public void close() {
        System.out.println("【消息代理】关闭消息通道。");
    }
}
```

4、此时如果想创建代理类对象，则就必须进行一系列的CGLIB处理

```java
package com.example;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

class Message {
    public void send() {
        System.out.println("【发送消息】www.xxxx.cn");
    }
}

/**
 * 拦截器配置
 */
class ExampleProxy implements MethodInterceptor {
    // 保存真实主题对象
    private Object target;
    public ExampleProxy(Object target) {
        this.target = target;
    }
    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        Object returnData = null;
        if (this.connect()) {
            returnData = method.invoke(this.target, args);
            this.close();
        }
        return returnData;
    }
    public boolean connect() {
        System.out.println("【消息代理】进行消息发送通道的连接。");
        return true;
    }
    public void close() {
        System.out.println("【消息代理】关闭消息通道。");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        // 真实主体对象
        Message realObject = new Message();
        // 负责代理操作的程序类
        Enhancer enhancer = new Enhancer();
        // 假定一个父类
        enhancer.setSuperclass(realObject.getClass());
        // 设置代理类
        enhancer.setCallback(new ExampleProxy(realObject));
        // 创建代理对象
        Message proxyObject = (Message) enhancer.create();
        proxyObject.send();
    }
}

// 输出内容
【消息代理】进行消息发送通道的连接。
【发送消息】www.xxxx.cn
【消息代理】关闭消息通道。
```

在进行代理设计模式定义的时候，除了使用可以接口之外，还可以不受接口限制而实现基于类的代理设计，但从正常的设计角度来讲，强烈建议还是基于接口的设计会比较合理。



# 注解Annotation详解

> 菜鸟教程：https://www.runoob.com/w3cnote/java-annotation.html

## 1、什么是注解

1. Annotation是从JDK5.0（JDK1.5）开始引入的新技术
2. Annotation的作用：1）可以对程序作出解释（这点和注释没什么区别）2）可以被其他程序读取（比如编译器等）
3. Annotation的格式：注解是以“@注释名”在代码中存在的，还可添加参数值，例如：@SuppressWarnings(value="unchecked")
4. Annotation如何使用：可以附加在package，class，method，field等上面，相当于给他们添加额外的辅助信息，我们可以通过反射机制访问这些元数据

官方定义：

注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。注解用处主要如下：

1. 提供信息给编译器：编译器可以利用注解来探测错误和警告信息。

2. 编译阶段时的处理：软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。

3. 运行时的处理：某些注解可以在程序运行的时候接受代码的提取

4. 值得注意的是，注解不是代码本身的一部分。


我们通俗一点理解就是：注解就相当于超市里面商品的标签，它不属于商品，它只是为商品做一些解释说明，而注解就是为我们的代码作解释说明，并不属于我们代码本身的部分。




## 2、Java内置注解

Java 定义了一套注解，主要在 java.lang 与 java.lang.annotation 中。

作用在代码的注解是，Java从1.5版本以后默认内置三个标注：

1. @Override：定义在java.lang.Override，此注释只适用于修辞方法，表示一个方法声明打算重写超类中的另一个方法声明。

2. @Deprecated：定义在java.lang.Deprecated，建议别人不要使用旧的API的时候用的，编译的时候会用产生警告信息，可以设定在程序里的所有的元素上

3. @SuppressWarnings：定义在java.lang.SuppressWarnings，用来抑制编译时的警告信息。

   与之前两个注释有所不同，你需要添加一个参数才能正确使用，这些参数都是已经定义好了的：

   - @SuppressWarnings("all")：压制所有的警告
   - @SuppressWarnings("unchecked")：告诉编译器忽略 unchecked 警告信息，如使用List，ArrayList等未进行参数化产生的警告信息
   - @SuppressWarnings(value={"unused","deprecation")：使用了不赞成使用的类或方法时的警告、未使用的变量。压制了2中警告

```java
@SuppressWarnings("all")         // 所有类型的警告
@SuppressWarnings("unchecked")   // 执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型。
@SuppressWarnings("unused")      // 未使用的变量
@SuppressWarnings("resource")    // 有泛型未指定类型
@SuppressWarnings("path")        // 在类路径、源文件路径等中有不存在的路径时的警告
@SuppressWarnings("deprecation") // 使用了不赞成使用的类或方法时的警告
@SuppressWarnings("fallthrough") // 当 Switch 程序块直接通往下一种情况而没有 break; 时的警告
@SuppressWarnings("serial")      // 某类实现Serializable(序列化)， 但没有定义 serialVersionUID 时的警告
@SuppressWarnings("rawtypes")    // 没有传递带有泛型的参数
@SuppressWarnings("try")         // 没有catch时的警告
@SuppressWarnings("finally")     // 任何finally子句不能正常完成时的警告

// 以下是源码引用中见到的，但实际很少用到的
@SuppressWarnings("FragmentNotInstantiable")
@SuppressWarnings("ReferenceEquality")
@SuppressWarnings("WeakerAccess")
@SuppressWarnings("UnusedParameters")
@SuppressWarnings("NullableProblems")
@SuppressWarnings("SameParameterValue")
@SuppressWarnings("PointlessBitwiseExpression")
```

作用在其他注解的注解（元注解）：

1. @Retention：标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问
2. @Documented：标记这些注解是否包含在用户文档中
3. @Target：标记这个注解应该是哪种 Java 成员
4. @Inherited：标记这个注解是继承于哪个注解类（默认注解并没有继承于任何子类）
5. @Repeatable：Java8开始支持，标识某注解可以在同一个声明上使用多次

从Java7后开始，额外添加了个注解：

1. @SafeVarargs：Java7开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告
1. @FunctionalInterface：函数式接口注解，这个是 Java 1.8 版本引入的新特性。



## 3、Java的元注解

元注解的作用就是负责注解其他注解，Java定义了5个标准的meta-annotation类型，他们被用来提供对其他annotation类型作说明。这些类型和它们所支持的类在java.lang.annotation包中可以找到：@Retention、@Documented、@Target、@Inherited、@Repeatable（该注解Java8开始支持）

1、@Retention：表示在什么级别保存该注解信息，描述注解的生命周期（RUNTIME>CLASS>SOURCE ）如设置RUNTIME，在CLASS和SOURCE都有效

| 范围（生命周期类型）    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| RetentionPolicy.SOURCE  | 注解仅存在于源码中(*.java)，编译后不保存在class字节码文件中  |
| RetentionPolicy.CLASS   | 默认的保留策略，在源码(*.java)和class文件中都存在，但运行时不存在，即运行时无法获得 |
| RetentionPolicy.RUNTIME | 注解存在于class字节码文件中，运行时可以获取（可以通过反射机制获取到） |
| 如上是通俗的解释：      | 如下是官方解释：                                             |
| RetentionPolicy.SOURCE  | 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视（即源文件保留） |
| RetentionPolicy.CLASS   | 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中（即class保留） |
| RetentionPolicy.RUNTIME | 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们（即运行时保留） |

2、@Target：用于描述注解的使用范围（即：被描述的注解的使用位置，用来说明该注解可以被声明在那些类型元素之前）

| Target类型                   | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| ElementType.TYPE             | 应用于类、接口（包括注解类型）、枚举                         |
| ElementType.FIELD            | 应用于属性（包括枚举中的常量）                               |
| ElementType.METHOD           | 应用于方法                                                   |
| ElementType.PARAMETER        | 应用于方法的形参                                             |
| ElementType.CONSTRUCTOR      | 应用于构造函数                                               |
| ElementType.LOCAL_VARIABLE   | 应用于局部变量                                               |
| ElementType.ANNOTATION_TYPE  | 应用于注解类型                                               |
| ElementType.PACKAGE          | 应用于包                                                     |
| ElementType.TYPE_PARAMETER   | 1.8 新增，应用于类型变量                                     |
| ElementType.TYPE_USE         | 1.8 新增，应用于任何使用类型的语句中（例如声明语句、泛型和强制转换语句中的类型） |
| ElementType.MODULE           | Java 9 新增，应用于Module类型                                |
| ElementType.RECORD_COMPONENT | Java 16 新增，用于Record 类及获取 Record 类信息              |

3、@Documented：表明该注解标记的元素可以被Javadoc或类似的工具文档化

4、@Inherited：表明使用了@Inherited 注解的注解，所标记的类的子类也会拥有这个注解。

Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。说的比较抽象。代码来解释。

```java
/// 自定义注解
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}
// 自定义Java类
@Test
public class A {}
public class B extends A {}
```

PS：注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A，类 B 也拥有 Test 这个注解。

5、@Repeatable：标识某注解可以在同一个声明上使用多次， Java1.8增加的，所以算是一个新的特性。

案例：一个人他既是程序员又是产品经理，同时他还是个画家

```java
@interface Persons {
    Person[]  value();
}
@Repeatable(Persons.class)
@interface Person{
    String role default "";
}
@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan {}
```

注意上面的代码，@Repeatable 注解了 Person。而 @Repeatable 后面括号中的类相当于一个容器注解。

什么是容器注解呢？就是用来存放其它注解的地方。它本身也是一个注解。我们再看看代码中的相关容器注解。

```java
@interface Persons {
    Person[]  value();
}
```

按照规定，它里面必须要有一个 value 的属性，属性类型是一个被 @Repeatable 注解过的注解数组，注意它是数组。

如果不好理解的话，可以这样理解。Persons 是一张总的标签，上面贴满了 Person 这种同类型但内容不一样的标签。把 Persons 给一个 SuperMan 贴上，相当于同时给他贴了程序员、产品经理、画家的标签。

我们可能对于 @Person(role=”PM”) 括号里面的内容感兴趣，它其实就是给 Person 这个注解的 role 属性赋值为 PM ，大家不明白正常，马上就讲到注解的属性这一块。



## 4、注解的属性

注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。参数成员只能用基本类型byte、short、char、int、long、float、double、boolean八种基本数据类型和String、Enum、Class、annotations等数据类型，及这一些类型的数组。

1、在注解中定义属性时它的类型必须是 8 种基本数据类型外加类、接口、注解及它们的数组

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation {
    public int id();
    public String msg();
}

@TestAnnotation(id = 3, msg = "hello annotation")
public class JavaAPIDemo {
}
```

2、注解中属性可以有默认值，默认值需要用 default 关键值指定

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation {
    // id 属性默认值为 -1，msg 属性默认值为 ok
    public int id() default -1;
    public String msg() default "ok";
}

// 因为有默认值，所以无需要再在 @TestAnnotation 后面的括号里面进行赋值了，这一步可以省略
@TestAnnotation()
public class JavaAPIDemo {
}
```

3、在Annotation定义中，如果只有一个需要用户设置的必要属性时，可以使用value作为属性名称，这样在进行内容设置时可以不写属性名称

```java
@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation {
    public String value();
    public String msg() default "";
}

/**
 * 1.TestAnnotation 注解只有 value 这个属性。所以可以这样应用
 * 2.TestAnnotation 注解有其他属性值时，加上默认值也能这样应用
 */
@TestAnnotation("ok")
public class JavaAPIDemo {
}
```

4、最后：还需要注意的一种情况是一个注解没有任何属性。那么在应用这个注解的时候，括号都可以省略。

```java
@interface TestAnnotation {}
@TestAnnotation
public class JavaAPIDemo {}
```





## 4、自定义注解

使用@interface自定义注解分析：

1. Annotation型定义为@interface，所有的Annotation会自动继承java.lang.Annotation接口，并且不能再去继承别的类或是接口
2. 参数成员只能用 public 或默认这两个访问权修饰，可以通过default设置成员默认值。也可以不定义成员
3. 其中的每一个方法实际上是声明一个配置参数，方法名称就是参数名称
4. 返回类型就是参数的类型（返回值只能是8种基本数据类型 和 String、Class、Enum、Annotations等数据类型）
5. 如果只有一个成员变量，一般参数名为value。注解元素必须要有值，我们定义注解元素时，经常使用空字符串，0做为默认值
6. 要获取类方法和字段的注解信息，必须通过Java的反射技术来获取 Annotation 对象，因为你除此之外没有别的获取注解对象的方法

```java
import java.lang.annotation.*;

@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation1{
    // 注解的参数：参数类型 + 参数名();
    String name() default "";
    int age() default 0;
    int id() default -1; // 如果默认值为-1，代表不存在
    String[] schools() default {"北京大学","清华大学"};
}

@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation2{
    // 如果只有一个参数成员，一般参数名为value，在调用的时候可以省略参数名称
    String value();
}

public class JavaAPIDemo {
    // 注解开源显示复制，如果没有默认值，我们就必须给注解赋值
    @TestAnnotation1(name="123",age = 2)
    public void test1(){}

    @TestAnnotation2("")
    public void test2(){}
}
```



# 反射与注解Annotation

本节介绍了获取Annotation信息的方法，以及Annotation的运行策略，通过案例解释如何实现自定义的Annotation。从JDK1.5后Java开发提供了Annotation技术支持，这种技术为项目的编写带来了新的模型，而后经过了十多年的发展，Annotation的技术得到了非常广泛的应用，并且在所有的项目开发中都会存在。



## 1、反射获取注解信息

在进行类或方法定义时，都可以使用一系列的Annotation进行声明，于是如果要想获得这些Annotation信息，那么可以直接通过反射来完成。在java.lang.reflect里面有一个AccessibleObject类，在本类中提供有获取Annotation类的方法：

| 方法名                                                       | 描述                                     |
| ------------------------------------------------------------ | ---------------------------------------- |
| public Annotation[] getAnnotations()                         | 获取全部Annotation信息                   |
| public Annotation[] getDeclaredAnnotations()                 | 获取全部Annotation信息(忽略继承的注解)   |
| public < T extends Annotation > T getAnnotation(Class< T > annotationClass) | 获取指定类型的Annotation                 |
| public < T extends Annotation > T getDeclaredAnnotation(Class< T > annotationClass) | 获取指定类型的Annotation(忽略继承的注解) |
| public < T extends Annotation > T[] getAnnotationsByType(Class< T > annotationClass) | 返回该元素指定类型的注解                 |
| public < T extends Annotation > T[] getDeclaredAnnotationsByType(Class< T > annotationClass) | 返回直接存在该元素上某类型的注释         |
| public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) | 是否存在指定的Annotation                 |

![image.png](https://ucc.alicdn.com/pic/developer-ecology/fa527e5dc53d4c4a953235a51de6a5a7.png)

操作示例 1：获取接口和接口子类上的Annotation信息

```java
package com.example;
import java.io.Serializable;
import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

// 该接口有2个Annotation，程序执行时可以获取
@FunctionalInterface
@Deprecated(since = "1.0")
interface IMessage {
    void send(String msg);
}
// 无法在程序执行时获取
@SuppressWarnings("serial")
class MessageImpl implements IMessage, Serializable {
    // 无法在程序执行时获取
    @Override
    public void send(String msg) {
        System.out.println("【消息发送】" + msg);
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        {
            // 获取接口上的全部Annotation及信息
            Annotation annotations [] = IMessage.class.getAnnotations();
            for (Annotation temp : annotations) {
                System.out.println(temp);
                // @java.lang.FunctionalInterface()
                // @java.lang.Deprecated(forRemoval=false, since="1.0")
            }
        }
        System.out.println("-----------------------");
        {
            // 获取MessageImpl子类上的Annotation信息
            Annotation[] annotations = MessageImpl.class.getAnnotations();
            for (Annotation temp : annotations) {
                System.out.println(temp);
            }
        }
        System.out.println("-----------------------");
        {
            // 获取MessageImpl.toString()方法上的Annotation信息
            Method method = MessageImpl.class.getDeclaredMethod("send", String.class);
            Annotation[] annotations = method.getAnnotations();
            for (Annotation temp : annotations) {
                System.out.println(temp);
            }
        }
    }
}

// 输出内容
@java.lang.FunctionalInterface()
@java.lang.Deprecated(forRemoval=false, since="1.0")
-----------------------
-----------------------
```

本程序的主要目的是要进行接口、类、方法上的Annotation信息获取，通过结果可以发现，程序最终只获得了在接口上定义的两个Annotation信息，之所以无法获取某些Annotation，主要和Annotation的定义范围有关。

@FunctionalInterface（运行时）：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```

@SuppressWarnings（源代码）：

```java
@Retention(RetentionPolicy.SOURCE)
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
public @interface SuppressWarnings {}
```

现在发现“@FunctionalInterface”是在运行时生效的Annotation，所以程序执行时可以获取Annotation；而“@SuppressWarnings”是在源代码编写时有效。在RetentionPolicy枚举类中还有一个class的定义，指的是在类定义时生效。

RetentionPolicy的三个范围：

| 范围    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| SOURCE  | 注解仅存在于源码中(*.java)，编译后不保存在class字节码文件中  |
| CLASS   | 默认的保留策略，在源码(*.java)和class文件中都存在，但运行时不存在，即运行时无法获得 |
| RUNTIME | 注解存在于class字节码文件中，运行时可以获取（可以通过反射机制获取到） |



## 2、获取自定义注解信息

现在已经清楚了Annotation的获取，以及Annotation的运行策略，但是最为关键性的因素是如何实现自定义的Annotation呢？为此在Java中提供了新的语法，使用“@interface”来定义Annotation。

@Target标记Annotation操作范围：

| 范围                        | 描述                               |
| --------------------------- | ---------------------------------- |
| ElementType.ANNOTATION_TYPE | 只能用在注解声明上                 |
| ElementType.CONSTRUCTOR     | 只能用在构造方法声明上             |
| ElementType.FIELD           | 只能用在字段声明上（包括枚举常量） |
| ElementType.LOCAL_VARIABLE  | 只能用在局部变量声明上             |
| ElementType.METHOD          | 只能用在方法的声明上               |
| ElementType.PACKAGE         | 只能用在包的声明上                 |
| ElementType.PARAMETER       | 只能用在方法参数的声明上           |
| ElementType.TYPE            | 只能用在类、接口、枚举类型上       |

操作示例 1：自定义Annotation

```java
package com.example;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.reflect.Method;

/**
 * 自定义的Annotation
 * RetentionPolicy.RUNTIME：定义Annotation的运行策略
 */
@Retention(RetentionPolicy.RUNTIME)
@interface DefaultAnnotation {
    public String title();
    public String url() default "www.example.cn";
}

class Message {
    @DefaultAnnotation(title = "example")
    public void send(String msg) {
        System.out.println("【消息发送】" + msg);
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        // 获取指定方法
        Method method = Message.class.getMethod("send", String.class);
        // 获取指定的Annotation
        DefaultAnnotation annotation = method.getAnnotation(DefaultAnnotation.class);
        // 直接调用Annotation中的方法，获取消息内容
        String msg = annotation.title() + "（" + annotation.url() + ")";
        method.invoke(Message.class.getDeclaredConstructor().newInstance(), msg);
    }
}

// 输出内容
【消息发送】example（www.example.cn)
```

操作示例 2：使用默认属性名称（属性简化设置）

在Annotation定义中，如果只有一个需要用户设置的必要属性时，可以使用value作为属性名称，这样在进行内容设置时可以不写属性名称。

```java
@Retention(RetentionPolicy.RUNTIME)
@interface DefaultAnnotation {
    public String value();
    public String url() default "www.example.cn";
}
class Message {
    // 也可以使用：@DefaultAnnotation(value = "example")
    @DefaultAnnotation("example")
    public void send(String msg) {
        System.out.println("【消息发送】" + msg);
    }
}
// 由于value属性名称为系统内定名称，所以在开发中可以省略此属性名称，这一点日后开发中较为常见
```

使用Annotation之后的最大特点是可以结合反射机制实现程序的处理。



## 3、整合工厂设计模式与Annotation

现在已经清楚了Annotation的整体作用，但是Annotation到底在开发中能做哪些事情呢？为了进一步理解Annotation的处理目的，下面将结合工厂设计模式来应用Annotation操作。

操作示例：Annotation整合工厂设计模式与代理设计模式

```java
package com.example;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

@Retention(RetentionPolicy.RUNTIME)
@interface UseMessage {
    public Class<?> clazz() ;
}

/**
 * 利用Annotation实现了类的使用
 */
@UseMessage(clazz=NetMessageImpl.class)
class MessageService {
    private IMessage message;
    public MessageService() {
        UseMessage use = MessageService.class.getAnnotation(UseMessage.class);
        // 直接通过Annotation获取
        this.message = (IMessage) Factory.getInstance(use.clazz());
    }
    public void send(String msg) {
        this.message.send(msg);
    }
}
class Factory {
    private Factory() {}
    // 直接返回一个实例化对象
    public static <T> T getInstance(Class<T> clazz) {
        try {
            return (T) new MessageProxy().bind(clazz.getDeclaredConstructor().newInstance());
        } catch (Exception e) {
            e.printStackTrace();
            return null ;
        }
    }
}
class MessageProxy implements InvocationHandler {
    private Object target ;
    public Object bind(Object target) {
        this.target = target ;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }
    public boolean connect() {
        System.out.println("〖代理操作〗进行消息发送通道的连接。");
        return true ;
    }
    public void close() {
        System.out.println("〖代理操作〗关闭连接通道。");
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            if (this.connect()) {
                return method.invoke(this.target, args);
            } else {
                throw new Exception("【ERROR】消息无法进行发送！");
            }
        } finally {
            this.close();
        }
    }
}
interface IMessage {
    public void send(String msg) ;
}
class MessageImpl implements IMessage {
    @Override
    public void send(String msg) {
        System.out.println("【消息发送】" + msg);
    }
}
class NetMessageImpl implements IMessage {
    @Override
    public void send(String msg) {
        System.out.println("【网络消息发送】" + msg);
    }
}

public class JavaAPIDemo {
    public static void main(String[] args)  {
        MessageService messageService = new MessageService();
        messageService.send("www.example.cn");
    }
}

// 输出内容
〖代理操作〗进行消息发送通道的连接。
【网络消息发送】www.example.cn
〖代理操作〗关闭连接通道。
```

更换：@UserMessage(clazz =NetMessageImpl.class )，输出内容如下：

```
〖代理操作〗进行消息发送通道的连接。
【消息发送】www.example.cn
〖代理操作〗关闭连接通道。
```

由于Annotation的存在，所以面向接口的编程处理将可以直接利用Annotation的属性完成控制，从而使得整体代码变得整洁。



## 4、使用反射动态修改注解属性值

```java
package com.example;

import java.lang.annotation.*;
import java.lang.reflect.*;
import java.util.Map;

@Target({ElementType.FIELD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@interface Name {
    String value() default "Unknown";
}

@Name("typeName")
class Member {
    @Name ("fieldName")
    private String value;
}

public class JavaAPIDemo {
    public static void main(String ...args) throws NoSuchFieldException, IllegalAccessException {
        {
            // 通过反射修改运行中的方法上的注解属性
            Field field = Member.class.getDeclaredField("value"); // 获取Member的name字段
            Name name = field.getAnnotation(Name.class); // 获取的Name注解实例
            InvocationHandler h = Proxy.getInvocationHandler(name); // 获取name这个代理实例所持有的 InvocationHandler
            Field hField = h.getClass().getDeclaredField("memberValues");  // 获取 AnnotationInvocationHandler 的 memberValues 字段
            hField.setAccessible(true); // 因为这个字段是 private final 修饰，所以要打开权限
            Map memberValues = (Map) hField.get(h); // 获取 memberValues
            memberValues.put("value", "customFiledName"); // 修改 value 属性值
            System.out.println(name.value()); // 获取 name 的 value 属性值
        }
        System.out.println("---------------------------分割线---------------------------");
        {
            // 通过反射修改运行中的类上的注解属性
            Name name = Member.class.getDeclaredAnnotation(Name.class);
            InvocationHandler h = Proxy.getInvocationHandler(name); // 获取代理实例所持有的 InvocationHandler
            Field hField = h.getClass().getDeclaredField("memberValues");  // 获取 AnnotationInvocationHandler 的 memberValues 字段
            hField.setAccessible(true); // 因为这个字段是 private final 修饰，所以要打开权限
            Map memberValues = (Map) hField.get(h); // 获取 memberValues
            memberValues.put("value", "customTypeName"); // 修改 value 属性值
            System.out.println(name.value()); // 获取 name 的 value 属性值
        }
        System.out.println("---------------------------分割线---------------------------");
        {
            // 验证
            Member member = new Member();
            System.out.println(member.getClass().getDeclaredAnnotation(Name.class).value());
            System.out.println(member.getClass().getDeclaredField("value").getDeclaredAnnotation(Name.class).value());
        }
    }
}

// 输出内容
customFiledName
---------------------------分割线---------------------------
customTypeName
---------------------------分割线---------------------------
customTypeName
customFiledName
```

PS：membervalues属性为JDK中存储所有成员变量值的Map



# 反射系列方法大全汇总

> 来源：Java反射系列--方法大全：https://blog.csdn.net/feiying0canglang/article/details/126878437

## 1、Class 相关方法

| 方法                               | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| getName()                          | 返回String形式的该类的名称。                                 |
| newInstance()                      | 根据某个Class对象产生其对应类的实例，它调用的是此类的默认构造方法(没有默认无参构造器会报错) |
| getClassLoader()                   | 返回该Class对象对应的类的类加载器。                          |
| getSuperClass()                    | 返回某子类所对应的直接父类所对应的Class对象                  |
| getComponentType()                 | 如果当前类表示一个数组，则返回表示该数组组件的 Class 对象，否则返回 null。 |
| getConstructor(Class[])            | 返回当前 Class 对象表示的类的指定的公有构造子对象。          |
| getComponentType()                 | 如果当前类表示一个数组，则返回表示该数组组件的 Class 对象，否则返回 null。 |
| getConstructor(Class[])            | 返回当前 Class 对象表示的类的指定的公有构造子对象。          |
| getDeclaredConstructors()          | 返回当前 Class 对象表示的类的所有已说明的构造子对象数组。    |
| getDeclaredField(String)           | 返回当前 Class 对象表示的类或接口的指定已说明的一个域对象（获取指定的一个公共的字段，包括继承的） |
| getDeclaredFields()                | 返回当前 Class 对象表示的类或接口的所有已说明的域对象数组。获取公共和私有的属性不包括继承 |
| getDeclaredMethod(String, Class[]) | 返回当前 Class 对象表示的类或接口的指定已说明的一个方法对象。 |
| getDeclaredMethods()               | 返回 Class 对象表示的类或接口的所有已说明的方法数组。        |
| getField(String)                   | 返回当前 Class 对象表示的类或接口的指定的公有成员域对象。    |
| getFields()                        | 返回当前 Class 对象表示的类或接口的所有可访问的公有域对象数组。可以获取自身和继承公共的属性 |
| getInterfaces()                    | 返回当前对象表示的类或接口实现的接口。                       |
| getMethod(String, Class[])         | 返回当前 Class 对象表示的类或接口的指定的公有成员方法对象。  |
| getMethods()                       | 返回当前 Class 对象表示的类或接口的所有公有成员方法对象数组，包括已声明的和从父类继承的方法。 |
| isArray()                          | 判定此Class对象所对应的是否是一个数组对象                    |
| isInstance(Object)                 | 此方法是 Java 语言 instanceof 操作的动态等价方法。           |
| isInterface()                      | 判定指定的 Class 对象是否表示一个接口类型                    |
| isPrimitive()                      | 判定指定的 Class 对象是否表示一个 Java 的基类型。            |



## 2、获取接口/父类/构造方法

| **方法**                                    | **说明**       |
| ------------------------------------------- | -------------- |
| Class<?>[] c1 = c.getInterfaces();          | 取得实现的接口 |
| Class<?> c2 = c.getSuperclass();            | 取得父类       |
| Constructor<?> con[] = c.getConstructors(); | 取得构造函数   |



## 3、获取方法数据

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Method getMethod(String name, Class<?>... parameterTypes)    | 通过方法名和参数列表获得方法                                 |
| Method[] getMethods()                                        | 获得所有方法。反映此 Class 对象所表示的类或接口（包括父类或接口） |
| Method getDeclaredMethod(String name, Class<?>... parameterTypes) | 反映此 Class 对象表示的类或接口声明的指定方法（不包括继承的方法） |
| Method[] getDeclaredMethods()                                | 反映此 Class 对象表示的类或接口声明的所有方法（不包括继承的方法） |

详细方法：

| 方法                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| int getModifiers()             | 取得本方法的访问修饰符。取得后通过Modefier.toString(xxx)还原 |
| String getName()               | 取得方法的名称                                               |
| Class<?>[] getParameterTypes() | 得到方法的全部参数类型                                       |
| String getName()               | 取得方法的名称                                               |
| Class<?>[] getParameterTypes() | 得到方法的全部参数类型                                       |

```java
package org.example.a;

import java.lang.reflect.Method;
import java.lang.reflect.Modifier;

interface ICategory {
    public static final String CATEGORY_NAME = "Human";

    public void sayCategory();

    public String sayHello(String name, Integer age);
}

class Human implements ICategory {
    private String name;
    private Integer age;

    public Human() {
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

    @Override
    public void sayCategory() {
        System.out.println(CATEGORY_NAME);
    }

    @Override
    public String sayHello(String name, Integer age) {
        return "名字:" + name + "年龄:" + age;
    }
}

public class Demo {
    public static void main(String[] args) {
        Class<?> c = null;
        try {
            c = Class.forName("org.example.a.Human");
        } catch (Exception e) {
            e.printStackTrace();
        }

        Method m[] = c.getMethods();
        for (int i = 0; i < m.length; i++) {
            Class<?> r = m[i].getReturnType();
            Class<?> p[] = m[i].getParameterTypes();
            int xx = m[i].getModifiers();
            System.out.print(Modifier.toString(xx) + " ");
            System.out.print(r.getName() + " ");
            System.out.print(m[i].getName());
            System.out.print("(");
            for (int x = 0; x < p.length; x++) {
                System.out.print(p[x].getName() + " " + "arg" + x);
                if (x < p.length - 1) {
                    System.out.print(", ");
                }
            }
            System.out.print(")");

            Class<?> ex[] = m[i].getExceptionTypes();
            if (ex.length > 0) {
                System.out.print(" throws ");
                for (int j = 0; j < ex.length; j++) {
                    System.out.print(ex[j].getName());
                    if (j < ex.length - 1) {
                        System.out.print(",");
                    }
                }
            }
            System.out.println();
        }
    }
}
```

```java
public java.lang.String getName()
public void setName(java.lang.String arg0)
public java.lang.String sayHello(java.lang.String arg0, java.lang.Integer arg1)
public java.lang.Integer getAge()
public void setAge(java.lang.Integer arg0)
public void sayCategory()
public final void wait() throws java.lang.InterruptedException
public final void wait(long arg0, int arg1) throws java.lang.InterruptedException
public final native void wait(long arg0) throws java.lang.InterruptedException
public boolean equals(java.lang.Object arg0)
public java.lang.String toString()
public native int hashCode()
public final native java.lang.Class getClass()
public final native void notify()
public final native void notifyAll()
```



## 4、获取属性

| 方法                                | 说明                                      |
| ----------------------------------- | ----------------------------------------- |
| Field getField(String name)         | 根据属性名称获取public属性                |
| Field[] getFields()                 | 获得该类所有的public属性                  |
| Field getDeclaredField(String name) | 根据属性名称获取属性。private属性也可获取 |
| Field[] getDeclaredFields()         | 获得该类所有的属性。private属性也可获取   |

详细方法：

| 方法                                                         | 说明                           |
| ------------------------------------------------------------ | ------------------------------ |
| Object get(Object obj)                                       | 获得一个对象中属性的具体内容   |
| void set(Object obj,Object value)                            | 设置指定对象中属性的具体内容   |
| int RetModifiers()                                           | 获得属性的修饰符               |
| String getName()                                             | 获得此属性的名称               |
| boolean isAccessible()                                       | 判断此属性是否可被外部访问     |
| void setAccessible(boolean flag)                             | 设置一个属性是否可被外部访问   |
| static void setAccessibie(AccessibIeObject[] array,boolean flag) | 将一组属性设置是否可被外部访问 |
| String toString()                                            | 返回此Field类的信息            |

```java
package org.example.a;
 
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
 
interface ICategory {
    public static final String CATEGORY_NAME = "Human";
 
    public void sayCategory();
 
    public String sayHello(String name, Integer age);
}
 
class Human implements ICategory {
    private String name;
    private Integer age;
 
    public Human() {
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
 
 
    @Override
    public void sayCategory() {
        System.out.println(CATEGORY_NAME);
    }
 
    @Override
    public String sayHello(String name, Integer age) {
        return "名字:" + name + "年龄:" + age;
    }
}
 
public class Demo {
    public static void main(String[] args) {
        Class<?> c = null;
        try {
            c = Class.forName("org.example.a.Human");
        } catch (Exception e) {
            e.printStackTrace();
        }
 
        Field f[] = c.getDeclaredFields();
        System.out.println("本类属性： ");
        for (int i = 0; i < f.length; i++) {
            Class<?> r = f[i].getType();
            int mo = f[i].getModifiers();
            String priv = Modifier.toString(mo);
            System.out.print(priv + " ");
            System.out.print(r.getName() + " ");
            System.out.print(f[i].getName());
            System.out.println(";");
        }
 
        System.out.println("-------------------------------------------");
 
        Field f1[] = c.getDeclaredFields();
        System.out.println("公共属性： ");
        for (int i = 0; i < f1.length; i++) {
            Class<?> r = f1[i].getType();
            int mo = f1[i].getModifiers();
            String priv = Modifier.toString(mo);
            System.out.print(priv + "");
            System.out.print(r.getName() + " ");
            System.out.print(f1[i].getName());
            System.out.println(";");
        }
    }
}
```

```java
本类属性： 
private java.lang.String name;
private java.lang.Integer age;
-------------------------------------------
公共属性： 
privatejava.lang.String name;
privatejava.lang.Integer age;
```



## 5、获得注解

Method、Constructor、Field这三个类对象都可以判断是否有特定的注解类，方法是：

```java
boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)
```

本处使用枚举类示例，普通类类似的。

```java
package org.example.a;

import java.lang.reflect.Field;

enum Color{
    @Deprecated
    RED,
    GREEN,
    BLUE
}

public class Demo {
    public static void main(String[] args) {
        for (Color value : Color.values()) {
            Field field = null;
            try {
                field = Color.class.getField(value.name());
            } catch (NoSuchFieldException e) {
                e.printStackTrace();
            }
            if (field.isAnnotationPresent(Deprecated.class)) {
                System.out.println(value.name() + "上有注解类：" + Deprecated.class.getName());
            }
        }
    }
}
```

```
RED上有注解类：java.lang.Deprecated 
```



## 6、操作数组

| **方法**                    | **说明**                |
| --------------------------- | ----------------------- |
| Class<?> getComponentType() | 取得一个数组的Class对象 |

详细方法

| **方法**                                                     | **说明**                           |
| ------------------------------------------------------------ | ---------------------------------- |
| static Object get(Object array,int index)                    | 根据下标取得数组内容               |
| static Object newlnstance(Class<?> componentType, int length) | 根据己有的数组类型开辟新的数组对象 |
| static void set(Object array,int index,Object value)         | 修改指定位置的内容                 |

