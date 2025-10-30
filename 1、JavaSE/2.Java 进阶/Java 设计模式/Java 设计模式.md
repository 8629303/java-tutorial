

# 设计模式背景介绍

## 1、什么是设计模式

设计模式（Design Pattern）是针对于软件设计中**反复出现**的问题所提出来的一个解决方案，是前人对代码开发经验的总结，它不是语法规定，学好设计模式可以开阔我们的编程思维，让我们编写的程序具备**可复用性、可扩展性、可读性、可靠性**以及安全性等；

1. 复用性：相同的功能代表，不用多次编写
2. 可扩展性：当我们需要新增的功能时，非常的方便
3. 可读性：编程规范，便于其他程序员的理解和阅读
4. 可靠性：当我们新增功能时，对援用的功能没有影响

1995 年，GoF（Gang of Four，四人组/帮）合作出版了《设计模式：可复用面向对象软件的基础》一书，共收录了 23 种设计模式，从此树立了软件设计模式领域的里程碑，人称**「GoF设计模式」**。

**总结：设计模式就是一套成熟的软件解决方案，学好设计模式能让我们开阔编程思维、编写出优异的代码**



## 2、设计模式的划分

### 1、根据目的划分

根据设计模式是用来完成什么工作来划分，这种方式可分为**创建型模式**、**结构型模式**和**行为型模式** 3 种；

1. **创建型模式：**用于描述“怎样创建对象“，它的主要特点是“将对象的创建与使用分离”。GoF 中提供了**单例、原型、工厂方法、抽象工厂、建造者**等 5 种创建型模式。
2. **结构型模式：**用于描述如何将类或对象按某种布局组成更大的结构，GoF 中提供了**代理、适配器、桥接、装饰、外观、享元、组合**等 7 种结构型模式。
3. **行为型模式：**用于描述类或对象之间怎样相互协作共同完成单个对象都无法单独完成的任务，以及怎样分配职责。GoF 中提供了**模板方法、策略、命令、职责链、状态、观察者、中介者、迭代器、访问者、备忘录、解释器**等 11 种行为型模式。

* * *

+   **创建型模式：关注怎样创建对象**
+   **结构性模式：关注怎样组合类**
+   **行为型模式：关注怎样协同类（怎样分配职责）**



### 2、根据作用范围来分

根据模式是主要用于类上还是主要用于对象上来分，这种方式可分为**类模式**和**对象模式**两种。

1. **类模式：** 用于处理类与子类之间的关系，这些关系通过继承来建立，是静态的，在编译时刻便确定下来了。GoF 中的工厂方法、（类）适配器、模板方法、解释器属于该模式。
2. **对象模式：** 用于处理对象之间的关系，这些关系可以通过组合或聚合来实现，在运行时刻是可以变化的，更具动态性。GoF 中除了以上 4 种，其他的都是对象模式。

| 范围/目的 | 创建型模式                   | 结构型模式                                          | 行为型模式                                                   |
| --------- | ---------------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| 类模式    | 工厂方法                     | （类）适配器                                        | 模板方法、解释器                                             |
| 对象模式  | 单例、原型、抽象工厂、建造者 | 代理、 （对象）适配器、桥接、装饰、外观、享元、组合 | 策略、命令、职责链、状态、观察者、中介者、迭代器、 访问者、备忘录 |

# UML统一建模语言

# 设计模式七大原则



# 创建型：单例模式

0、单例模式介绍

单例模式的介绍：

- 单例模式（Singleton Pattern）是最简单的设计模式之一，这种类型的设计模式属于创建型模式，通过单例模式的方法创建的类在当前进程中只有一个实例。
- 一般情况下，不建议使用懒汉式，建议使用饿汉式。只有在要明确实现 lazy loading 效果时，才会使用登记式。如果涉及到反序列化创建对象时，可以尝试使用枚举式。如果有其他特殊的需求，可以考虑使用双检锁方式。

单例模式的优点：

- 单例模式可以保证内存里只有一个实例，减少了内存的开销。
- 可以避免对资源的多重占用。
- 单例模式设置全局访问点，可以优化和共享资源的访问。

单例模式的缺点：

- 单例模式一般没有接口，扩展困难。如果要扩展，则除了修改原来的代码，没有第二种途径，违背开闭原则。

- 在并发测试中，单例模式不利于代码调试。在调试过程中，如果单例中的代码没有执行完，也不能模拟生成一个新的对象。
- 单例模式的功能代码通常写在一个类中，如果功能设计不合理，则很容易违背单一职责原则。

单例模式的场景：

- 创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等。
- WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来。



## 1、饿汉式「静态常量」

描述：线程很安全，没有锁机制，执行效率高，类加载初始化，可能浪费内存。这个可以用到实际生产环境。

```java
// 饿汉式
class Singleton {
    // 私有构造方法，防止外部new
    private Singleton() {}

    // 提供一个私有的静态变量对象
    private final static Singleton instance = new Singleton();

    // 向外部提供一个公开获取方法
    public static Singleton getInstance() {
        return instance;
    }
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        System.out.println(singleton1 == singleton2);
    }
}
```



## 2、饿汉式「静态代码块」

描述：线程很安全，没有锁机制，执行效率高，类加载初始化，可能浪费内存。这个可以用到实际生产环境。

```java
// 饿汉式
class Singleton {
    // 私有构造方法，防止外部new
    private Singleton() {}

    // 提供一个私有的静态变量对象
    private final static Singleton instance;

    // 在静态代码块中初始化该对象
    static {
        instance = new Singleton();
    }

    // 向外部提供一个公开获取方法
    public static Singleton getInstance() {
        return instance;
    }
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        System.out.println(singleton1 == singleton2);
    }
}
```



## 3、懒汉式「线程不安全」

描述：线程不安全，没有锁机制，执行效率高，类用时初始化，不会浪费内存。这个不建议用到实际生产环境。

```java
// 懒汉式
class Singleton {
    // 私有构造方法，防止外部new
    private Singleton() {}

    // 提供一个私有的静态变量对象
    private static Singleton instance;

    // 向外部提供一个公开获取方法
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        System.out.println(singleton1 == singleton2);
    }
}
```



## 4、懒汉式「线程安全，同步方法」

描述：线程很安全，拥有方法锁，执行效率慢，类用时初始化，不会浪费内存。这个不建议用到实际生产环境。

```java
// 懒汉式
class Singleton {
    // 私有构造方法，防止外部new
    private Singleton() {}

    // 提供一个私有的静态变量对象
    private static Singleton instance;

    // 向外部提供一个公开获取方法, 并且给此静态方式加上 synchronized
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        System.out.println(singleton1 == singleton2);
    }
}
```



## 5、双检锁「线程安全，推荐用」

描述：这种方式采用双检锁机制，类用时初始化，线程安全且在多线程情况下能保持高性能，实现方式略微复杂。

```java
// 双检锁(双重校验锁)
class Singleton {
    // 私有构造方法，防止外部new
    private Singleton() {}

    // 提供一个私有的静态变量对象
    private volatile static Singleton instance;

    // 向外部提供一个公开获取方法
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
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        System.out.println(singleton1 == singleton2);
    }
}
```



## 6、登记式「线程安全，静态内部类，推荐用」

描述：这种方式采用静态内部类，类用时初始化，线程安全且在多线程情况下能保持高性能，实现方式比较一般。

```java
// 登记式(静态内部类)
class Singleton {
    // 私有构造方法，防止外部new
    private Singleton() {}

    // 提供一个私有的静态内部类
    private static class SingletonHolder {
        private final static Singleton INSTANCE = new Singleton();
    }

    // 向外部提供一个公开获取方法
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        System.out.println(singleton1 == singleton2);
    }
}
```



## 7、枚举式「线程安全，推荐用」

描述：线程很安全，没有锁机制，执行效率高，类加载初始化，但是这种实现方式还没有被广泛采用，自动支持序列化机制，绝对防止多次实例化。

```java
// 枚举类
enum Singleton {
    INSTANCE
}

public class SingletonTest {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.INSTANCE;
        Singleton singleton2 = Singleton.INSTANCE;
        System.out.println(singleton1 == singleton2);
    }
}
```



## 8、单例模式应用

最佳实践：`java.lang.Runtime`「饿汉式的典型应用」

![img](https://i-blog.csdnimg.cn/blog_migrate/45e8a143e7496387c761f16e24d1b258.png)



# 创建型：工厂方法模式

## 1、工厂模式介绍

工厂模式的介绍：

- 工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

- 在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。


工厂模式的优点：

- 工厂类包含必要的逻辑判断，可以决定在什么时候创建哪一个产品的实例。客户端可以免除直接创建产品对象的职责，很方便的创建出相应的产品。工厂和产品的职责区分明确。

- 客户端无需知道所创建具体产品的类名，只需知道参数即可。
- 也可以引入配置文件，在不修改客户端代码的情况下更换和添加新的具体产品类。

工厂模式的缺点：

- 工厂模式的工厂类单一，负责所有产品的创建，职责过重，一旦异常，整个系统将受影响。且工厂类代码会非常臃肿，违背高聚合原则。

- 使用工厂模式会增加系统中类的个数（引入新的工厂类），增加系统的复杂度和理解难度。
- 系统扩展困难，一旦增加新产品不得不修改工厂逻辑，在产品类型较多时，可能造成逻辑过于复杂。
- 工厂模式使用了 static 工厂方法，造成工厂角色无法形成基于继承的等级结构。

工厂模式的场景：

- 日志记录器，记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方。
- 数据库访问，当用户不知道最后系统采用哪一类数据库，以及数据库可能有变化时。
- 设计一个连接服务器的框架，需要三个协议：“POP3”、“IMAP”、“HTTP”，可以把这三个作为产品类，共同实现一个接口。



## 2、工厂模式应用



### 1、关系依赖图

![img](https://i-blog.csdnimg.cn/blog_migrate/b13408819062dd03c4eb9196fda92373.png)

### 2、产品接口类

Shape

```java
public interface Shape {
    void draw();
}
```



### 3、产品实现类

- Rectangle

```java
public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Rectangle：绘制长方形");
    }
}
```

- Square

```java
public class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Square：绘制正方形");
    }
}
```

- Circle

```java
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Circle：绘制圆形");
    }
}
```



### 4、工厂类编写

ShapeFactory

```java
public class ShapeFactory {
    public static Shape createShape(String shapeType) {
        if ("Rectangle".equalsIgnoreCase(shapeType)) {
            return new Rectangle();
        }
        if ("Square".equalsIgnoreCase(shapeType)) {
            return new Square();
        }
        if ("Circle".equalsIgnoreCase(shapeType)) {
            return new Circle();
        }
        return null;
    }
}
```



### 5、工厂类测试

ShapeFactoryTest

```java
public class ShapeFactoryTest {
    public static void main(String[] args) {
        Shape rectangle = ShapeFactory.createShape("Rectangle");
        rectangle.draw();

        Shape square = ShapeFactory.createShape("Square");
        square.draw();

        Shape crcle = ShapeFactory.createShape("Circle");
        crcle.draw();
    }
}
```

输出内容如下：

```
Rectangle：绘制长方形
Square：绘制正方形
Circle：绘制圆形
```



## 3、工厂模式应用

最佳实践：`java.util.Calendar`

![img](https://i-blog.csdnimg.cn/blog_migrate/b0ff98e7a4ca8577a1a3241203510531.png)



# 创建型：抽象工厂模式

## 1、抽象工厂模式介绍

抽象工厂模式的介绍：

- 抽象工厂模式（Abstract Factory）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类，每个生成的工厂都能按照工厂模式提供对象。


抽象工厂模式的优点：

- 可以在类的内部对产品族中相关联的多等级产品共同管理，而不必专门引入多个新的类来进行管理。

- 当需要产品族时，抽象工厂可以保证客户端始终只使用同一个产品的产品组。
- 抽象工厂增强了程序的可扩展性，当增加一个新的产品族时，不需要修改原代码，满足开闭原则。

抽象工厂模式的缺点：

- 当产品族中需要增加一个新的产品时，所有的工厂类都需要进行修改。

- 增加了系统的抽象性和理解难度。

抽象工厂模式的场景：

- 当需要创建的对象是一系列相互关联或相互依赖的产品族时，如电器工厂中的电视机、洗衣机、空调等。

- 系统中有多个产品族，但每次只使用其中的某一族产品。如有人只喜欢穿某一个品牌的衣服和鞋。
- 系统中提供了产品的类库，且所有产品的接口相同，客户端不依赖产品实例的创建细节和内部结构。

抽象工厂模式的角色：

- 抽象工厂（Abstract Factory）：提供了创建产品的接口，它包含多个创建产品的方法 newProduct()，可以创建多个不同等级的产品。
- 具体工厂（Concrete Factory）：主要是实现抽象工厂中的多个抽象方法，完成具体产品的创建。
- 抽象产品（Product）：定义了产品的规范，描述了产品的主要特性和功能，抽象工厂模式有多个抽象产品。
- 具体产品（ConcreteProduct）：实现了抽象产品角色所定义的接口，由具体工厂来创建，它同具体工厂之间是多对一的关系。



## 2、抽象工厂模式实现

### 1、类间关系依赖图

![img](https://i-blog.csdnimg.cn/blog_migrate/d4991273cf7570021f3ed4202f9179fc.png)



### 2、图形产品接口类

- Shape

```java
public interface Shape {
    void draw();
}
```



### 3、图形产品实现类

- Rectangle

```java
public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Rectangle：绘制长方形");
    }
}
```

- Square

```java
public class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Square：绘制正方形");
    }
}
```

- Circle

```java
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Circle：绘制圆形");
    }
}
```



### 4、图形工厂类编写

- ShapeFactory

```java
public class ShapeFactory extends AbstractFactory {
    @Override
    public Shape createShape(String shapeType) {
        if ("Rectangle".equalsIgnoreCase(shapeType)) {
            return new Rectangle();
        }
        if ("Square".equalsIgnoreCase(shapeType)) {
            return new Square();
        }
        if ("Circle".equalsIgnoreCase(shapeType)) {
            return new Circle();
        }
        return null;
    }

    @Override
    public Color createColor(String color) {
        return null;
    }
}
```



### 5、颜色产品接口类

- Color

```java
public interface Color {
    void fill();
}
```



### 6、颜色产品实现类

- Red

```java
public class Red implements Color {
    @Override
    public void fill() {
        System.out.println("Red：填充红色");
    }
}
```

- Green

```java
public class Green implements Color {
    @Override
    public void fill() {
        System.out.println("Green：填充绿色");
    }
}
```

- Blue

```java
public class Blue implements Color {
    @Override
    public void fill() {
        System.out.println("Blue：填充蓝色");
    }
}
```



### 7、颜色工厂类编写

- ColorFactory

```java
public class ColorFactory extends AbstractFactory {
    @Override
    public Color createColor(String colorType) {
        if ("Red".equalsIgnoreCase(colorType)) {
            return new Red();
        }
        if ("Green".equalsIgnoreCase(colorType)) {
            return new Green();
        }
        if ("Blue".equalsIgnoreCase(colorType)) {
            return new Blue();
        }
        return null;
    }

    @Override
    public Shape createShape(String shapeType) {
        return null;
    }
}
```



### 8、产品家族抽象类

- AbstractFactory

```java
public abstract class AbstractFactory {
    public abstract Shape createShape(String shapeType);
    public abstract Color createColor(String colorType);
}
```



### 9、抽象类的工厂类

- AbstractFactoryProducer

```java
public class AbstractFactoryProducer {
    public static AbstractFactory createFactory(String choice) {
        if ("Shape".equalsIgnoreCase(choice)) {
            return new ShapeFactory();
        }
        if ("Color".equalsIgnoreCase(choice)) {
            return new ColorFactory();
        }
        return null;
    }
}
```



### 10、抽象工厂类测试

- AbstractFactoryProducerTest

```java
public class AbstractFactoryProducerTest {
    public static void main(String[] args) {
        AbstractFactory shapeFactory = AbstractFactoryProducer.createFactory("Shape");
        Shape rectangle = shapeFactory.createShape("Rectangle");
        rectangle.draw();
        Shape square = shapeFactory.createShape("Square");
        square.draw();
        Shape circle = shapeFactory.createShape("Circle");
        circle.draw();

        AbstractFactory colorFactory = AbstractFactoryProducer.createFactory("Color");
        Color red = colorFactory.createColor("Red");
        red.fill();
        Color green = colorFactory.createColor("Green");
        green.fill();
        Color blue = colorFactory.createColor("Blue");
        blue.fill();
    }
}
```

```
Rectangle：绘制长方形
Square：绘制正方形
Circle：绘制圆形
Red：填充红色
Green：填充绿色
Blue：填充蓝色
```





## 3、抽象工厂模式应用

最佳实践：`java.sql.Connection`

Connection 接口源码如下所示，其定义了 Statement、PreparedStatement、CallableStatement 三个产品等级结构。

```java
public interface Connection extends Wrapper, AutoCloseable {
    Statement createStatement() throws SQLException;
    PreparedStatement prepareStatement(String sql) throws SQLException;
    CallableStatement prepareCall(String sql) throws SQLException;
}
```



# 创建型：原型模式

## 1、原型模式介绍

原型模式的介绍：

- 原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能，这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

- 这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。


在原型模式中的克隆可以细分为“浅克隆”和“深克隆”两种，具体介绍如下：

- 浅克隆： 对于数据类型是基本数据类型的成员变量，浅克隆会直接进行值传递，也就是将该属性值复制一份给新的对象。对于数据类型是引用数据类型的成员变量，比如说成员变量是某个数组、某个类的对象等，那么浅克隆会进行引用传递，也就是只是将该成员变量的引用值（内存地址）复制一份给新的对象。因为实际上两个对象的该成员变量都指向同一个实例。在这种情况下，在一个对象中修改该成员变量会影响到另一个对象的该成员变量值。

- 深克隆： 复制对象的所有基本数据类型的成员变量值，为所有引用数据类型的成员变量申请存储空间，并复制每个引用数据类型成员变量所引用的对象，也就是说，对象进行深克隆要对整个对象（包括对象的引用类型）进行克隆。

原型模式的优点：

- Java 自带的原型模式基于内存二进制流的复制，在性能上比直接 new 一个对象更加优良。

- 可以使用深克隆方式保存对象的状态，使用原型模式将对象复制一份，并将其状态保存起来，简化了创建对象的过程，以便在需要的时候使用（例如恢复到历史某一状态），可辅助实现撤销操作。

原型模式的缺点：

- 需要为每一个类都配置一个 clone 方法，clone 方法位于类的内部，当对已有类进行改造的时候，需要修改代码，违背了开闭原则。

- 配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。
- 当实现深克隆时，需要编写较为复杂的代码，而且当对象之间存在多重嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆，实现起来会比较麻烦。因此，深克隆、浅克隆需要运用得当。
- 深克隆会破坏单例，其实防御方式很简单，单例类不要实现 Cloneable 接口即可。

原型模式的场景：

- 对象之间相同或相似，即只是个别的几个属性不同的时候。
- 创建对象成本较大，例如初始化时间长，占用CPU太多，或者占用网络资源太多等，需要优化资源。
- 创建一个对象需要繁琐的数据准备或访问权限等，需要提高性能或者提高安全性。
- 系统中大量使用该类对象，且各个调用者都需要给它的属性重新赋值。



## 2、原型模式实现（浅克隆）

- IdCard

```java
// 身份证对象
class IdCard {
    private String id;

    public IdCard(String id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return id;
    }
}
```

- Person

```java
// 代表人对象
public class Person implements Cloneable {
    private String name;
    private int age;
    private IdCard idCard;

    public Person(String name, int age, IdCard idCard) {
        this.name = name;
        this.age = age;
        this.idCard = idCard;
    }

    public IdCard getIdCard() {
        return idCard;
    }

    public void setIdCard(IdCard idCard) {
        this.idCard = idCard;
    }

    @Override
    public String toString() {
        return "Person{" +
            "personHashCode=" + this.hashCode() +
            ", name='" + name + '\'' +
            ", age=" + age +
            ", idCard=" + idCard +
            ", idCardHashCode=" + idCard.hashCode() +
            '}';
    }

    // 浅克隆已经帮我们实现了
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

- PersonTest

```java
public class PersonTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        // 创建一个对象
        Person person = new Person("张三", 20, new IdCard("10086"));

        // 克隆两个对象
        Person person1 = (Person) person.clone();
        Person person2 = (Person) person.clone();

        // 打印三人信息
        System.out.println(person);
        System.out.println(person1);
        System.out.println(person2);
    }
}
```

- 运行结果

```
Person{personHashCode=460141958, name='张三', age=20, idCard=10086, idCardHashCode=1163157884}
Person{personHashCode=1956725890, name='张三', age=20, idCard=10086, idCardHashCode=1163157884}
Person{personHashCode=356573597, name='张三', age=20, idCard=10086, idCardHashCode=1163157884}
```

- **解释说明：**我们发现可以通过实现`implements Cloneable`来完成浅拷贝，基本变量是值传递克隆，而引用对象`IdCard`则是引用传递，这不符合我们面向对象思想，每一个`Person`应该都有一个独立的`IdCard`，而不是共用一个，而要解决这种问题，我们需要使用深克隆。



## 3、原型模式实现（深克隆一）

- IdCard：

```java
// 身份证对象
class IdCard implements Cloneable {
    private String id;

    public IdCard(String id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return id;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

- Person

```java
// 代表人对象
public class Person implements Cloneable {
    private String name;
    private int age;
    private IdCard idCard;

    public Person(String name, int age, IdCard idCard) {
        this.name = name;
        this.age = age;
        this.idCard = idCard;
    }

    public IdCard getIdCard() {
        return idCard;
    }

    public void setIdCard(IdCard idCard) {
        this.idCard = idCard;
    }

    @Override
    public String toString() {
        return "Person{" +
            "personHashCode=" + this.hashCode() +
            ", name='" + name + '\'' +
            ", age=" + age +
            ", idCard=" + idCard +
            ", idCardHashCode=" + idCard.hashCode() +
            '}';
    }

    // 深克隆我们需要自己手动实现，在引用对象中也需要实现clone方法
    @Override
    protected Object clone() throws CloneNotSupportedException {
        // 完成对当前基本数据类型
        // 注：String 类型通过常量赋值时相当于基本数据类型，通过new关键字创建对象时便是引用数据类型
        Object person = super.clone();

        // 对引用类型进行单独处理
        Person p = (Person) person;
        // 实现自己的克隆
        IdCard idCard = (IdCard) p.getIdCard().clone();
        p.setIdCard(idCard);

        return p;
    }
}
```

- PersonTest

```java
public class PersonTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        // 创建一个对象
        Person person = new Person("张三", 20, new IdCard("10086"));

        // 克隆两个对象
        Person person1 = (Person) person.clone();
        Person person2 = (Person) person.clone();

        // 打印三人信息
        System.out.println(person);
        System.out.println(person1);
        System.out.println(person2);
    }
}
```

- 运行效果

```
Person{personHashCode=460141958, name='张三', age=20, idCard=10086, idCardHashCode=1163157884}
Person{personHashCode=1956725890, name='张三', age=20, idCard=10086, idCardHashCode=356573597}
Person{personHashCode=1735600054, name='张三', age=20, idCard=10086, idCardHashCode=21685669}
```

- 解释说明：


使用这种深克隆的方式，完美的解决了当数据类型为引用类型时，只是拷贝原引用对象地址而不是一个全新的引用对象的引用，但是这种实现有一个很大的弊端，需要在每一个对象中都实现clone方法，如果类全是你自己写的，那自然没问题，实现一下就行了，不过有点麻烦。但是，如果你引用的是第三方的一个类，无法修改源代码，这种方式，显然就无法实现深克隆了。



## 4、原型模式实现（深克隆二）

- IdCard

```java
// 身份证对象
class IdCard implements Serializable {
    private String id;

    public IdCard(String id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return id;
    }
}
```

- Person

```java
// 代表人对象
public class Person implements Serializable {
    private String name;
    private int age;
    private IdCard idCard;

    public Person(String name, int age, IdCard idCard) {
        this.name = name;
        this.age = age;
        this.idCard = idCard;
    }

    public IdCard getIdCard() {
        return idCard;
    }

    public void setIdCard(IdCard idCard) {
        this.idCard = idCard;
    }

    @Override
    public String toString() {
        return "Person{" +
            "personHashCode=" + this.hashCode() +
            ", name='" + name + '\'' +
            ", age=" + age +
            ", idCard=" + idCard +
            ", idCardHashCode=" + idCard.hashCode() +
            '}';
    }

    // 手动实现深克隆方法
    public Person deepClone() {
        // 创建流对象
        ByteArrayOutputStream bos = null;
        ObjectOutputStream oos = null;
        ByteArrayInputStream bis = null;
        ObjectInputStream ois = null;

        try {
            // 序列化
            bos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            //反序列化
            bis = new ByteArrayInputStream(bos.toByteArray());
            ois = new ObjectInputStream(bis);
            return (Person) ois.readObject();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        } finally {
            // 关闭流
            try {
                ois.close();
                bis.close();
                oos.close();
                bos.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

- PersonTest

```java
public class PersonTest {
    public static void main(String[] args) {
        // 创建一个对象
        Person person = new Person("张三", 20, new IdCard("10086"));

        // 克隆两个对象
        Person person1 = person.deepClone();
        Person person2 = person.deepClone();

        // 打印三人信息
        System.out.println(person);
        System.out.println(person1);
        System.out.println(person2);
    }
}
```

- 运行效果

```java
Person{personHashCode=1300109446, name='张三', age=20, idCard=10086, idCardHashCode=1020371697}
Person{personHashCode=20132171, name='张三', age=20, idCard=10086, idCardHashCode=186370029}
Person{personHashCode=2094548358, name='张三', age=20, idCard=10086, idCardHashCode=51228289}
```

- 解释说明：这种方式我们需要手动编写deepClone方法，使用Java流中的序列化与反序列化来实现深克隆，但是这种实现，需要在每一个类中都继承序列化Serializable接口，这种方式，如果你调用的是第三方类，也有可能第三方类上没有实现Serializable序列化接口，但是一般来说，大多都会实现，总的来说，这种比较推荐使用，而且效率也高，要是真的第三方类没有实现Serializable序列化接口，这里可以再提供一种思路，但是不具体实现了，可以借助第三方工具包fastjson先将对象转换为json字符串，然后再把json字符串转为对象，这样也是可以的。




## 5、原型模式应用

最佳实践：`java.util.ArrayList`

![img](https://i-blog.csdnimg.cn/blog_migrate/9219b8b343e819a847cddc8ebd81ccc3.png)



# 创建型：建造者模式

## 1、建造者模式介绍

建造者模式的介绍：

- 建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象，这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

- 它是将一个复杂的对象分解为多个简单的对象，然后一步一步构建而成。它将变与不变相分离，即产品的组成部分是不变的，但每一部分是可以灵活选择的。


建造者模式的优点：

- 封装性好，构建和表示分离。

- 扩展性好，各个具体的建造者相互独立，有利于系统的解耦。
- 客户端不必知道产品内部组成的细节，建造者可以对创建过程逐步细化，而不对其它模块产生任何影响，便于控制细节风险。

建造者模式的缺点：

- 产品的组成部分必须相同，这限制了其使用范围。

- 如果产品的内部变化复杂，如果产品内部发生变化，则建造者也要同步修改，后期维护成本较大。

建造者模式的场景：

- 隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品。

- 相同的方法，不同的执行顺序，产生不同的结果。
- 多个部件或零件，都可以装配到一个对象中，但是产生的结果又不相同。
- 产品类非常复杂，或者产品类中不同的调用顺序产生不同的作用。
- 初始化一个对象特别复杂，参数多，而且很多参数都具有默认值。

建造者模式的角色：

- 产品角色（Product）：它是包含多个组成部件的复杂对象，由具体建造者来创建其各个零部件。
- 抽象建造者（Builder）：它是一个包含创建产品各个子部件的抽象方法的接口，通常还包含一个返回复杂产品的方法 getResult()。
- 具体建造者(Concrete Builder）：实现 Builder 接口，完成复杂产品的各个部件的具体创建方法。
- 指挥者（Director）：它调用建造者对象中的部件构造与装配方法完成复杂对象的创建，在指挥者中不涉及具体产品的信息。

建造者模式在应用过程中可以根据需要改变，如果创建的产品种类只有一种，只需要一个具体建造者，这时可以省略掉抽象建造者，甚至可以省略掉指挥者角色。



## 2、建造者模式实现

### 1、关系依赖图

我们都知道盖房子是一个模式化的过程：打地基 -》砌墙面 -》盖楼顶，只不过不同类型的房子，具体的要求也会不尽相同，比如地基大小、楼的高度等等，但是大致的过程都是相似的，HouseDirector是整个工程的指挥者，由他指挥VillaHouse（别墅建造者）、HighHouse（高楼建造者）来建筑不同类型的House产品。而使用端只需要和指挥者打交道就可以了，屏蔽了产品实现的细节，构建和表示分离，有利于系统的拓展和解耦。

![img](https://i-blog.csdnimg.cn/blog_migrate/3a1cff76a3b5b68e5e2f47410b695fb1.png)



### 2、产品实现类

- House

```java
public class House {
    private String ground;
    private String wall;
    private String roofed;

    public String getGround() {
        return ground;
    }

    public void setGround(String ground) {
        this.ground = ground;
    }

    public String getWall() {
        return wall;
    }

    public void setWall(String wall) {
        this.wall = wall;
    }

    public String getRoofed() {
        return roofed;
    }

    public void setRoofed(String roofed) {
        this.roofed = roofed;
    }

    @Override
    public String toString() {
        return "House{" +
            "ground='" + ground + '\'' +
            ", wall='" + wall + '\'' +
            ", roofed='" + roofed + '\'' +
            '}';
    }
}
```



### 3、抽象建造者

- HouseBuilder

```java
public abstract class HouseBuilder {
    // 创建产品对象
    protected House house = new House();

    // 生产产品流程
    public abstract void buildGround(); // 1.打地基
    public abstract void buildWall();   // 2.砌墙面
    public abstract void buildRoofed(); // 3.盖楼顶

    // 返回产品对象
    public House getResult() {
        return house;
    }
}
```



### 4、工程指挥者

- HouseDirector

```java
public class HouseDirector {
    private HouseBuilder houseBuilder;

    public HouseDirector(HouseBuilder houseBuilder) {
        this.houseBuilder = houseBuilder;
    }

    public House build(){
        houseBuilder.buildGround();
        houseBuilder.buildWall();
        houseBuilder.buildRoofed();
        return houseBuilder.getResult();
    }
}
```



### 5、具体建造者：别墅

- VillaHouse

```java
public class VillaHouse extends HouseBuilder {
    @Override
    public void buildGround() {
        house.setGround("100平");
        System.out.println("别墅：打地基");
    }

    @Override
    public void buildWall() {
        house.setWall("每面墙高三米");
        System.out.println("别墅：砌墙面");
    }

    @Override
    public void buildRoofed() {
        house.setRoofed("欧式风格屋顶");
        System.out.println("别墅：盖楼顶");
    }
}
```



### 6、具体建造者：高楼

- HighHouse

```java
public class HighHouse extends HouseBuilder{
    @Override
    public void buildGround() {
        house.setGround("200平");
        System.out.println("高楼：打地基");
    }

    @Override
    public void buildWall() {
        house.setWall("每面墙高四米");
        System.out.println("高楼：砌墙面");
    }

    @Override
    public void buildRoofed() {
        house.setRoofed("中国复古屋顶");
        System.out.println("高楼：盖楼顶");
    }
}
```



### 7、产品使用端

- Client

```java
public class Client {
    public static void main(String[] args) {
        House house1 = new HouseDirector(new VillaHouse()).build();
        System.out.println(house1);

        System.out.println("====================");

        House house2 = new HouseDirector(new HighHouse()).build();
        System.out.println(house2);
    }
}
```

- 运行效果

```
别墅：打地基
别墅：砌墙面
别墅：盖楼顶
House{ground='100平', wall='每面墙高三米', roofed='欧式风格屋顶'}
====================
高楼：打地基
高楼：砌墙面
高楼：盖楼顶
House{ground='200平', wall='每面墙高四米', roofed='中国复古屋顶'}
```



## 3、建造者模式应用

### 1、最佳的实践

最佳实践：`org.apache.ibatis.builder.xml.XMLConfigBuilder`（mybatis 3.5.6框架源码）

### 2、产品实现类

`org.apache.ibatis.session.Configuration`

```java
public class Configuration {
    protected Environment environment;
    protected boolean safeRowBoundsEnabled;
    protected boolean safeResultHandlerEnabled;
    protected boolean mapUnderscoreToCamelCase;
    protected boolean aggressiveLazyLoading;
    protected boolean multipleResultSetsEnabled;
    protected boolean useGeneratedKeys;
    protected boolean useColumnLabel;
    protected boolean cacheEnabled;
    protected boolean callSettersOnNulls;
    protected boolean useActualParamName;
    protected boolean returnInstanceForEmptyRow;
    protected boolean shrinkWhitespacesInSql;
    protected String logPrefix;
    protected Class<? extends Log> logImpl;
    protected Class<? extends VFS> vfsImpl;
    protected Class<?> defaultSqlProviderType;
    protected LocalCacheScope localCacheScope;
    protected JdbcType jdbcTypeForNull;
    protected Set<String> lazyLoadTriggerMethods;
    protected Integer defaultStatementTimeout;
    protected Integer defaultFetchSize;
    protected ResultSetType defaultResultSetType;
    protected ExecutorType defaultExecutorType;
    protected AutoMappingBehavior autoMappingBehavior;
    protected AutoMappingUnknownColumnBehavior autoMappingUnknownColumnBehavior;
    protected Properties variables;
    protected ReflectorFactory reflectorFactory;
    protected ObjectFactory objectFactory;
    protected ObjectWrapperFactory objectWrapperFactory;
    protected boolean lazyLoadingEnabled;
    protected ProxyFactory proxyFactory;
    protected String databaseId;
    protected Class<?> configurationFactory;
    protected final MapperRegistry mapperRegistry;
    protected final InterceptorChain interceptorChain;
    protected final TypeHandlerRegistry typeHandlerRegistry;
    protected final TypeAliasRegistry typeAliasRegistry;
    protected final LanguageDriverRegistry languageRegistry;
    protected final Map<String, MappedStatement> mappedStatements;
    protected final Map<String, Cache> caches;
    protected final Map<String, ResultMap> resultMaps;
    protected final Map<String, ParameterMap> parameterMaps;
    protected final Map<String, KeyGenerator> keyGenerators;
    protected final Set<String> loadedResources;
    protected final Map<String, XNode> sqlFragments;
    protected final Collection<XMLStatementBuilder> incompleteStatements;
    protected final Collection<CacheRefResolver> incompleteCacheRefs;
    protected final Collection<ResultMapResolver> incompleteResultMaps;
    protected final Collection<MethodResolver> incompleteMethods;
    protected final Map<String, String> cacheRefMap;

    // ...此处省略很多代码
}
```

### 3、抽象建造者

`org.apache.ibatis.builder.BaseBuilder`

```java
public abstract class BaseBuilder {
    protected final Configuration configuration;
    protected final TypeAliasRegistry typeAliasRegistry;
    protected final TypeHandlerRegistry typeHandlerRegistry;

    public BaseBuilder(Configuration configuration) {
        this.configuration = configuration;
        this.typeAliasRegistry = this.configuration.getTypeAliasRegistry();
        this.typeHandlerRegistry = this.configuration.getTypeHandlerRegistry();
    }

    public Configuration getConfiguration() {
        return this.configuration;
    }

    // ...此处省略很多代码
}
```



### 4、具体建造者

`org.apache.ibatis.builder.xml.XMLConfigBuilder`

```java
public class XMLConfigBuilder extends BaseBuilder {
    private boolean parsed;
    private final XPathParser parser;
    private String environment;
    private final ReflectorFactory localReflectorFactory;

    // ...此处省略很多代码

    public Configuration parse() {
        if (this.parsed) {
            throw new BuilderException("Each XMLConfigBuilder can only be used once.");
        } else {
            this.parsed = true;
            this.parseConfiguration(this.parser.evalNode("/configuration"));
            return this.configuration;
        }
    }

    private void parseConfiguration(XNode root) {
        try {
            this.propertiesElement(root.evalNode("properties"));
            Properties settings = this.settingsAsProperties(root.evalNode("settings"));
            this.loadCustomVfs(settings);
            this.loadCustomLogImpl(settings);
            this.typeAliasesElement(root.evalNode("typeAliases"));
            this.pluginElement(root.evalNode("plugins"));
            this.objectFactoryElement(root.evalNode("objectFactory"));
            this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
            this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
            this.settingsElement(settings);
            this.environmentsElement(root.evalNode("environments"));
            this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
            this.typeHandlerElement(root.evalNode("typeHandlers"));
            this.mapperElement(root.evalNode("mappers"));
        } catch (Exception var3) {
            throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
        }
    }

    // ...此处省略很多代码
}
```

parse方法最终要返回一个Configuration对象，构建Configuration对象的建造过程都在parseConfiguration方法中，这也就是mybatis解析XML配置文件 来构建Configuration对象的主要过程，所以XMLConfigBuilder是具体建造者，复杂产品对象是Configuration，至于指挥者这里好像并没有用到。



# 结构型：代理模式

## 1、代理模式介绍

代理模式的介绍：

- 在有些情况下，一个客户不能或者不想直接访问另一个对象，这时需要找一个中介帮忙完成某项任务，这个中介就是代理对象。例如，购买火车票不一定要去火车站买，可以通过 12306 网站购买。

- 代理模式（Proxy Pattern）中，由于某些原因需要给某对象提供一个代理以控制对该对象的访问。这时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。这种类型的设计模式属于结构型模式。


根据代理的创建时期，代理模式分为静态代理和动态代理。

- 静态：由程序员创建代理类或特定工具自动生成源代码再对其编译，在程序运行前代理类的 .class 文件就已经存在了。

- 动态：在程序运行时，运用反射机制动态创建而成。

代理模式的优点：

- 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用。

- 代理对象可以扩展目标对象的功能。
- 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了程序的可扩展性。

代理模式的缺点：

- 代理模式会造成系统设计中类的数量增加。可以使用动态代理方式，降低系统设计中类的数量。

- 在客户端和目标对象之间增加一个代理对象，会造成请求处理速度变慢。
- 增加了系统的复杂度。

代理模式的场景：

- 远程代理，这种方式通常是为了隐藏目标对象存在于不同地址空间的事实，方便客户端访问。例如，用户申请某些网盘空间时，会在用户的文件系统中建立一个虚拟的硬盘，用户访问虚拟硬盘时实际访问的是网盘空间。
- 虚拟代理，这种方式通常用于要创建的目标对象开销很大时。例如，下载一幅很大的图像需要很长时间，因某种计算比较复杂而短时间无法完成，这时可以先用小比例的虚拟代理替换真实的对象，消除用户对服务器慢的感觉。
- 安全代理，这种方式通常用于控制不同种类客户对真实对象的访问权限。
- 智能指引，主要用于调用目标对象时，代理附加一些额外的处理功能。例如，增加计算真实对象的引用次数的功能，这样当该对象没有被引用时，就可以自动释放它。
- 延迟加载，指为了提高系统的性能，延迟对目标的加载。例如，Hibernate 中就存在属性的延迟加载和关联表的延时加载。

代理模式的角色：

- 抽象主题（Subject）类：通过接口或抽象类声明真实主题和代理对象实现的业务方法。

- 真实主题（Real Subject）类：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。
- 代理（Proxy）类：提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以访问、控制或扩展真实主题的功能。

![img](https://i-blog.csdnimg.cn/blog_migrate/ac8a1b119ed589105f944909239fd474.gif)



## 2、代理模式实现（静态代理）

### 1、抽象主题类

- UserDao

```java
public interface UserDao {
    public void save();

    public void delete();

    public void update();

    public void find();
}
```

### 2、真实主题类

- UserDaoImpl

```java
public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
        System.out.println("UserDaoImpl save ...");
    }

    @Override
    public void delete() {
        System.out.println("UserDaoImpl delete ...");
    }

    @Override
    public void update() {
        System.out.println("UserDaoImpl update ...");
    }

    @Override
    public void find() {
        System.out.println("UserDaoImpl find ...");
    }
}
```

### 3、代理类

- UserDaoProxy

```java
public class UserDaoProxy implements UserDao {
    private UserDao userDao;

    public UserDaoProxy(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void save() {
        userDao.save();
    }

    @Override
    public void delete() {
        System.out.println("UserDaoProxy 权限校验 ...");
        userDao.delete();
        System.out.println("UserDaoProxy 日志记录 ...");
    }

    @Override
    public void update() {
        userDao.update();
    }

    @Override
    public void find() {
        userDao.find();
    }
}
```

### 4、测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        UserDao userDao = new UserDaoImpl();
        UserDao userDaoProxy = new UserDaoProxy(userDao);
        userDaoProxy.save();
        userDaoProxy.delete();
        userDaoProxy.update();
        userDaoProxy.find();
    }
}
```

```
UserDaoImpl save ...
UserDaoProxy 权限校验 ...
UserDaoImpl delete ...
UserDaoProxy 日志记录 ...
UserDaoImpl update ...
UserDaoImpl find ...
```



## 3、代理模式实现（JDK动态代理）

### 1、抽象主题类

- UserDao

```java
public interface UserDao {
    public void save();

    public void delete();

    public void update();

    public void find();
}
```

### 2、真实主题类

- UserDaoImpl

```java
public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
        System.out.println("UserDaoImpl save ...");
    }

    @Override
    public void delete() {
        System.out.println("UserDaoImpl delete ...");
    }

    @Override
    public void update() {
        System.out.println("UserDaoImpl update ...");
    }

    @Override
    public void find() {
        System.out.println("UserDaoImpl find ...");
    }
}
```



### 3、代理类

- UserDaoProxy

```java
public class UserDaoProxy {
    private UserDao userDao;

    public UserDaoProxy(UserDao userDao) {
        this.userDao = userDao;
    }

    public UserDao createUserDao() {
        Class<? extends UserDao> clazz = userDao.getClass();
        UserDao ud = (UserDao) Proxy.newProxyInstance(
                clazz.getClassLoader(),
                clazz.getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Object result;
                        if ("delete".equals(method.getName())) {
                            System.out.println("UserDaoProxy 权限校验 ...");
                            result = method.invoke(userDao, args);
                            System.out.println("UserDaoProxy 日志记录 ...");
                        } else {
                            result = method.invoke(userDao, args);
                        }
                        return result;
                    }
                });
        return ud;
    }
}
```

### 4、测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        UserDao userDao = new UserDaoImpl();
        UserDao userDaoProxy = new UserDaoProxy(userDao).createUserDao();
        userDaoProxy.save();
        userDaoProxy.delete();
        userDaoProxy.update();
        userDaoProxy.find();
    }
}
```

```
UserDaoImpl save ...
UserDaoProxy 权限校验 ...
UserDaoImpl delete ...
UserDaoProxy 日志记录 ...
UserDaoImpl update ...
UserDaoImpl find ...
```



## 4、代理模式实现（CGLIB动态代理）

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>2.2.2</version>
</dependency>
```



### 1、抽象主题类

CGLib 动态代理方式可以无需使用抽象主题类。

### 2、真实主题类

- UserDaoImpl

```java
public class UserDaoImpl {
    public void save() {
        System.out.println("UserDaoImpl save ...");
    }

    public void delete() {
        System.out.println("UserDaoImpl delete ...");
    }

    public void update() {
        System.out.println("UserDaoImpl update ...");
    }

    public void find() {
        System.out.println("UserDaoImpl find ...");
    }
}
```

### 3、代理类

- UserDaoImplProxy

```java
public class UserDaoImplProxy {
    public UserDaoImpl createUserDaoImpl() {
        // 创建增强对象
        Enhancer enhancer = new Enhancer();
        // 设置父类对象
        enhancer.setSuperclass(UserDaoImpl.class);
        // 设置回调方法
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                Object result;
                if ("delete".equals(method.getName())) {
                    System.out.println("UserDaoImplProxy 权限校验 ...");
                    result = methodProxy.invokeSuper(proxy, args);
                    System.out.println("UserDaoImplProxy 日志记录 ...");
                } else {
                    result = methodProxy.invokeSuper(proxy, args);
                }
                return result;
            }
        });
        // 返回代理对象
        return (UserDaoImpl) enhancer.create();
    }
}
```

### 4、测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        UserDaoImpl userDaoImplProxy = new UserDaoImplProxy().createUserDaoImpl();
        userDaoImplProxy.save();
        userDaoImplProxy.delete();
        userDaoImplProxy.update();
        userDaoImplProxy.find();
    }
}
```

```
UserDaoImpl save ...
UserDaoProxy 权限校验 ...
UserDaoImpl delete ...
UserDaoProxy 日志记录 ...
UserDaoImpl update ...
UserDaoImpl find ...
```





## 5、代理模式应用

Spring 使用动态代理实现 AOP 时有两个非常重要的类，即 JdkDynamicAopProxy 类和 CglibAopProxy 类，其类图如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/6320c97996aa2ae9e0198996aa73cf03.png)

Spring 中的代理选择如下：

- 当 Bean 有实现接口时，Spring 会用 JDK 动态代理方式。

- 当 Bean 没有实现接口时，Spring 会选择 CGLib 动态代理方式。

Spring 可以通过配置强制使用 CGLib 动态代理，只需在 Spring 的配置文件中加入如下代码即可，也可以使用 Java Config 配置

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

// 可以使用 @EnableAspectJAutoProxy 注解来启用 AspectJ 自动代理，并使用 @Configuration 注解来定义配置类
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class AppConfig {
    // 在这里定义你的 Bean，例如切面和其他组件
}
```



# 结构型：适配器模式

## 1、适配器模式介绍

适配器模式的介绍：

- 适配器模式（Adapter Pattern）是作为两个不兼容接口之间的桥梁，这种类型的设计模式属于结构型模式。一些书籍也称接口适配器模式为缺省适配器模式（Default Adapter Pattern）。适配器模式主要分为三类：类适配器模式、对象适配器模式、接口适配器模式「**默认适配器**」。以生活中手机充电为例来讲解适配器模式，手机本身并不能直接用220V交流电，需要将220V的交流电转换为5V的直流电，在这个过程中，充电器本身相当于Adapter（适配器），220V交流电相当于Adaptee （适配者），5V直流电则是我们的Target（目标）。


适配器模式的优点：

- 客户端通过适配器可以透明地调用目标接口。

- 复用了现存的类，程序员不需要修改原有代码而重用现有的适配者类。
- 将目标类和适配者类解耦，解决了目标类和适配者类接口不一致的问题。
- 在很多业务场景中符合开闭原则。

适配器模式的缺点：

- 适配器编写过程需要结合业务场景全面考虑，可能会增加系统的复杂性。

- 增加代码阅读难度，降低代码可读性，过多使用适配器会使系统代码变得凌乱。

适配器模式的场景：

- 以前开发的系统存在满足新系统功能需求的类，但其接口同新系统的接口不一致。

- 使用第三方提供的组件，但组件接口定义和自己要求的接口定义不同。

适配器模式的角色：

- 目标（Target）：当前系统业务所期待的接口，它可以是抽象类或接口。就好比上述例子中的5V直流电。
- 适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口。就好比上述例子中的220V交流电。
- 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口/抽象类的格式访问适配者。



## 2、适配器模式实现（类适配器）

> 描述：看看怎么将220V适配成5V，这样220V也能当5V来用。因为，现在5V这个接口，我们没有合适的实现类可以用，所以需要适配器。

### 1、关系依赖图

![img](https://i-blog.csdnimg.cn/blog_migrate/e5bcbbaa26e95baa2745c072c2aa4310.png)

### 2、创建交流电

- Voltage220V「旧系统中已存在的接口或类，如果是接口，最后也需要拿到它的实现类来使用」

```java
public class Voltage220V {
    public int output220V() {
        System.out.println("Voltage220V output220V ...");
        return 220;
    }
}
```



### 3、定义直流电

- Voltage5V「新系统中的接口」

```java
public interface Voltage5V {
    // 定义一个标准，充电器来实现
    public int output5V();
}
```



### 4、创建充电器

- VoltageAdapter

```java
public class VoltageAdapter extends Voltage220V implements Voltage5V {
    @Override
    public int output5V() {
        // 获取220V交流电
        int output220V = output220V();
        // 转换为5V直流电
        int output5V = output220V / 44;

        System.out.println("VoltageAdapter output5V ...");
        return output5V;
    }
}
```



### 5、创建手机类

- Phone

```java
public class Phone {
    // 充电
    public void charging(Voltage5V voltage5V) {
        if (voltage5V.output5V() == 5) {
            System.out.println("电压刚好5V, 可以充电");
        } else if (voltage5V.output5V() > 5) {
            System.out.println("电压大于5V, 不能充电");
        }
    }
}
```



### 6、测试充电器

- PhoneTest

```java
public class PhoneTest {
    public static void main(String[] args) {
        // 创建一部手机
        Phone phone = new Phone();
        // 用充电器充电
        phone.charging(new VoltageAdapter());
    }
}
```

```
Voltage220V output220V ...
VoltageAdapter output5V ...
电压刚好5V, 可以充电
```



### 7、点评该模式

- Java是单继承机制，所以类适配器需要继承适配者（Adaptee，指Voltage220V）类，这点算是一个缺点，除此之外还必须要求目标（Target，指Voltage5V）必须是接口，有一定局限性；
- 适配者Voltage220V类的方法在适配器VoltageAdapter类中都会暴露出来，也增加了使用的成本。但是由于其继承了适配者Voltage220V类，所以它可以根据需求重写该类的方法，使得适配器VoltageAdapter类的灵活性增强了。



## 3、适配器模式实现（对象适配器）

### 1、关系依赖图

基本思路和类适配器模式相同，只是将`VoltageAdapter`类作修改，不是继承`Voltage220V`类，而是持有`Voltage220V`类的实例，以解决兼容性的问题。

根据“合成复用原则”，在系统中尽量使用关联关系（聚合）来替代继承关系。

![img](https://i-blog.csdnimg.cn/blog_migrate/cf83b9e9fb7975584ffdee55218518e6.png)

### 2、创建交流电

- Voltage220V

```java
public class Voltage220V {
    public int output220V() {
        System.out.println("Voltage220V output220V ...");
        return 220;
    }
}
```



### 3、定义直流电

- Voltage5V

```java
public interface Voltage5V {
    // 定义一个标准，充电器来实现
    public int output5V();
}
```



### 4、创建充电器

- VoltageAdapter

```java
public class VoltageAdapter implements Voltage5V {
    private Voltage220V voltage220V;

    public VoltageAdapter(Voltage220V voltage220V) {
        this.voltage220V = voltage220V;
    }

    @Override
    public int output5V() {
        // 获取220V交流电
        int output220V = voltage220V.output220V();
        // 转换为5V直流电
        int output5V = output220V / 44;

        System.out.println("VoltageAdapter output5V ...");
        return output5V;
    }
}
```



### 5、创建手机类

- Phone

```java
public class Phone {
    // 充电
    public void charging(Voltage5V voltage5V) {
        if (voltage5V.output5V() == 5) {
            System.out.println("电压刚好5V, 可以充电");
        } else if (voltage5V.output5V() > 5) {
            System.out.println("电压大于5V, 不能充电");
        }
    }
}
```



### 6、测试充电器

- PhoneTest

```java
public class PhoneTest {
    public static void main(String[] args) {
        // 创建一部手机
        Phone phone = new Phone();
        // 用充电器充电（传入220V交流电，传出5V直流电）
        phone.charging(new VoltageAdapter(new Voltage220V()));
    }
}
```

```
Voltage220V output220V ...
VoltageAdapter output5V ...
电压刚好5V, 可以充电
```



### 7、点评该模式

- 对象适配器和类适配器其实算是同一种思想，只不过实现方式不同。

- 根据合成复用原则，使用组合替代继承， 所以它解决了类适配器中VoltageAdapter必须继承Voltage220V的局限性问题，也不再强制要求Voltage5V必须是接口。使用成本更低，更灵活。因此，对象适配器模式是适配器模式常用的一种。





## 4、适配器模式实现（接口适配器）

### 1、关系依赖图

接口适配器介绍：当不需要全部实现接口提供的方法时，可先设计一个抽象类实现接口，并为该接口中每个方法提供 一个默认实现（空方法），那么该抽象类的子类可有选择地覆盖父类的某些方法来实现需求。

![img](https://i-blog.csdnimg.cn/blog_migrate/8356a86399f76fcc40fc823116399092.png)

### 2、创建动画接口

- Animation

```java
public interface Animation {
    public void fadeIn();
    public void fadeInUp();
    public void fadeInDown();
    public void fadeInLeft();
    public void fadeInRight();
    public void fadeOut();
    public void fadeOutUp();
    public void fadeOutDown();
    public void fadeOutLeft();
    public void fadeOutRight();
}
```



### 3、接口的适配器

- AnimationAdapter

```java
public class AnimationAdapter implements Animation {
    @Override
    public void fadeIn() {}
    @Override
    public void fadeInUp() {}
    @Override
    public void fadeInDown() {}
    @Override
    public void fadeInLeft() {}
    @Override
    public void fadeInRight() {}
    @Override
    public void fadeOut() {}
    @Override
    public void fadeOutUp() {}
    @Override
    public void fadeOutDown() {}
    @Override
    public void fadeOutLeft() {}
    @Override
    public void fadeOutRight() {}
}
```



### 4、适配器的子类

- JFrameAnimation

```java
public class JFrameAnimation extends AnimationAdapter {
    @Override
    public void fadeIn() {
        System.out.println("JFrameAnimation fadeIn ...");
    }

    @Override
    public void fadeOut() {
        System.out.println("JFrameAnimation fadeOut ...");
    }
}
```



### 5、客户端测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        Animation animation = new JFrameAnimation();
        animation.fadeIn();
        animation.fadeOut();
    }
}
```

```
JFrameAnimation fadeIn ...
JFrameAnimation fadeOut ...
```



### 6、该模式的实践

`javax.servlet.http.HttpServlet`继承`javax.servlet.GenericServlet`空实现于`javax.servlet.Servlet`

![img](https://i-blog.csdnimg.cn/blog_migrate/34f322962bca988a990f8f6b43c0afc9.png)



## 5、适配器模式应用

最佳实践：`适配器模式在 SpringMVC 框架应用`

由于`Controller`的类型不同，有多重实现方式，那么调用方式就不是确定的，如果需要直接调用`Controller`方法，需要调用的时候就得不断是使用`if else`来进行判断是哪一种子类然后执行。那么如果后面要扩展`Controller`，就得修改原来的代码，这样违背了 OCP 原则。

因此`SpringMVC`定义了一个适配接口`HandlerAdapter`，使得每一种`Controller`有一种对应的适配器实现类， 让适配器代替`Controller`执行相应的方法。这样在扩展`Controller`时，只需要增加一个适配器类就完成了`SpringMVC`的扩展了。

`org.springframework.web.servlet.HandlerAdapter`，相当于适配器模式中的目标（Target）

```java
public interface HandlerAdapter {
    boolean supports(Object handler);

    @Nullable
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

    long getLastModified(HttpServletRequest request, Object handler);
}
```

![img](https://i-blog.csdnimg.cn/blog_migrate/011e04277a1218d2a9c6aee9307b3822.png)

`org.springframework.web.servlet.DispatcherServlet`

```java
public class DispatcherServlet extends FrameworkServlet {
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        // 通过request可以得到请求对应的控制器handler
        mappedHandler = getHandler(processedRequest);
        // 根据handler得到对应的适配器 
        HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
        // 通过适配器去调用对应controller的方法，并返回ModelAndView
        mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    }
}
```

- getHandlerAdapter 方法：

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    if (this.handlerAdapters != null) {
        for (HandlerAdapter adapter : this.handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
    }
    throw new ServletException("No adapter for handler [" + handler + "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

我们随便找一个`HandlerAdapter`的实现类，相当于适配器模式中的适配器（Adapter），其中`Object handler`为适配者（Adaptee）

```java
public class HttpRequestHandlerAdapter implements HandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof HttpRequestHandler);
    }

    @Override
    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        ((HttpRequestHandler) handler).handleRequest(request, response);
        return null;
    }

    @Override
    public long getLastModified(HttpServletRequest request, Object handler) {
        if (handler instanceof LastModified) {
            return ((LastModified) handler).getLastModified(request);
        }
        return -1L;
    }
}
```



# 结构型：桥接模式

## 1、桥接模式介绍

桥接模式的介绍：

- 在现实生活中，某些类具有两个或多个维度的变化，如图形既可按形状分，又可按颜色分。如何设计类似于 Photoshop 这样的软件，能画不同形状和不同颜色的图形呢？如果用继承方式，m 种形状和 n 种颜色的图形就有 m×n 种，不但对应的子类很多，而且扩展困难。

- 当然，这样的例子还有很多，如不同颜色和字体的文字、不同品牌和功率的汽车，如果用桥接模式就能很好地解决这些问题，那么桥接模式到底是什么呢？
- 桥接模式（Bridge Pattern）的定义如下：将抽象与实现分离，使它们可以独立变化。它是用组合关系代替继承关系来实现，从而降低了抽象和实现这两个可变维度的耦合度。桥接模式遵循了里氏替换原则和依赖倒置原则，最终实现了开闭原则，对修改关闭，对扩展开放。

桥接模式的优点：

- 抽象与实现分离，扩展能力强。

- 符合开闭原则。
- 符合合成复用原则。
- 其实现细节对客户透明。

桥接模式的缺点：

- 由于聚合关系建立在抽象层，要求开发者针对抽象化进行设计与编程，能正确地识别出系统中两个独立变化的维度，这增加了系统的理解与设计难度。

- 桥接模式要求正确识别出系统中两个独立变化的维度(抽象、和实现)，因此其使用范围有一定的局限性，即需要有这样的应用场景。

桥接模式的场景：

- 当一个类存在两个独立变化的维度，且这两个维度都需要进行扩展时。

- 当一个系统不希望使用继承或因为多层次继承导致系统类的个数急剧增加时。
- 当一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性时。

桥接模式的角色：

- 抽象化（Abstraction）角色：定义抽象类，并包含一个对实现化对象的引用。
- 扩展抽象化（Refined Abstraction）角色：是抽象化角色的子类，实现父类中的业务方法，并通过组合关系调用实现化角色中的业务方法。
- 实现化（Implementor）角色：定义实现化角色的接口，供扩展抽象化角色调用。
- 具体实现化（Concrete Implementor）角色：给出实现化角色接口的具体实现。

**核心思想：用组合代替继承，使得抽象部分和实现部分可以独立变化。**

![img](https://i-blog.csdnimg.cn/blog_migrate/4ea425a85a9bfb30c030435b624963d6.gif)



## 2、桥接模式实现「实现一」

### 1、关系依赖图

![img](https://i-blog.csdnimg.cn/blog_migrate/25e94411bccd2c27bc94c0c531984393.png)

### 2、绘图接口类

- DrawAPI

```java
public interface DrawAPI {
    public void drawCircle(int x, int y, int radius);
}
```



### 3、绘图实现类

- RedCircle

```java
public class RedCircle implements DrawAPI {
    @Override
    public void drawCircle(int x, int y, int radius) {
        System.out.print("Drawing Red Circle");
        System.out.print(",x=" + x);
        System.out.print(",y=" + y);
        System.out.print(",radius=" + radius);
        System.out.println();
    }
}
```

- GreenCircle

```java
public class GreenCircle implements DrawAPI {
    @Override
    public void drawCircle(int x, int y, int radius) {
        System.out.print("Drawing Green Circle");
        System.out.print(",x=" + x);
        System.out.print(",y=" + y);
        System.out.print(",radius=" + radius);
        System.out.println();
    }
}
```



### 4、图形抽象类

- Shape

```java
public abstract class Shape {
    protected DrawAPI drawAPI;

    protected Shape(DrawAPI drawAPI) {
        this.drawAPI = drawAPI;
    }

    public abstract void draw();
}
```



### 5、图形实现类

- Circle

```java
public class Circle extends Shape {
    private int x;
    private int y;
    private int radius;

    public Circle(int x, int y, int radius, DrawAPI drawAPI) {
        super(drawAPI);
        this.x = x;
        this.y = y;
        this.radius = radius;
    }

    public void draw() {
        drawAPI.drawCircle(radius, x, y);
    }
}
```



### 6、最终测试类

- BridgePatternDemo

```java
public class BridgePatternDemo {
    public static void main(String[] args) {
        Shape redCircle = new Circle(100,100, 10, new RedCircle());
        Shape greenCircle = new Circle(100,100, 10, new GreenCircle());

        redCircle.draw();
        greenCircle.draw();
    }
}
```

```
Drawing Red Circle,x=10,y=100,radius=100
Drawing Green Circle,x=10,y=100,radius=100
```



## 3、桥接模式实现「实现二」

### 1、需求背景

**需求背景「用桥接模式设计不同品牌的消息发送系统」**：假设我们要设计一个消息发送系统，它需要支持：

- **不同类型的消息**：普通消息、加急消息、特急消息等。
- **不同的发送渠道**：短信、邮件、微信等。

如果我们采用传统的继承方式：

- SmsNormalMessage
- SmsUrgentMessage
- SmsVeryUrgentMessage
- EmailNormalMessage
- EmailUrgentMessage
- EmailVeryUrgentMessage
- WeChatNormalMessage
- WeChatUrgentMessage
- WeChatVeryUrgentMessage

**问题：类爆炸，难以维护！**

**使用桥接模式优化**：我们可以将 **“消息类型”** 作为**抽象层**，**“发送渠道”** 作为**实现层**，两者独立扩展，不会相互影响。



### 2、代码实现

```java
// 1. 实现部分接口（消息发送方式）
interface MessageSender {
    void send(String message, String toUser);
}

// 2. 具体实现部分（短信发送）
class SmsSender implements MessageSender {
    @Override
    public void send(String message, String toUser) {
        System.out.println("【短信发送】" + message + " -> " + toUser);
    }
}

// 3. 具体实现部分（邮件发送）
class EmailSender implements MessageSender {
    @Override
    public void send(String message, String toUser) {
        System.out.println("【邮件发送】" + message + " -> " + toUser);
    }
}

// 4. 抽象部分（消息类型）
abstract class AbstractMessage {
    // 组合而非继承
    protected MessageSender messageSender;

    public AbstractMessage(MessageSender messageSender) {
        this.messageSender = messageSender;
    }

    public abstract void sendMessage(String message, String toUser);
}

// 5. 扩展抽象部分（普通消息）
class NormalMessage extends AbstractMessage {
    public NormalMessage(MessageSender messageSender) {
        super(messageSender);
    }

    @Override
    public void sendMessage(String message, String toUser) {
        messageSender.send("[普通] " + message, toUser);
    }
}

// 6. 扩展抽象部分（加急消息）
class UrgentMessage extends AbstractMessage {
    public UrgentMessage(MessageSender messageSender) {
        super(messageSender);
    }

    @Override
    public void sendMessage(String message, String toUser) {
        messageSender.send("[加急] " + message, toUser);
    }
}

// 7. 客户端测试代码
public class BridgePatternDemo {
    public static void main(String[] args) {
        // 发送普通短信
        AbstractMessage message1 = new NormalMessage(new SmsSender());
        message1.sendMessage("会议通知", "张三");

        // 发送加急邮件
        AbstractMessage message2 = new UrgentMessage(new EmailSender());
        message2.sendMessage("服务器故障", "李四");
    }
}
```

```
【短信发送】[普通] 会议通知 -> 张三
【邮件发送】[加急] 服务器故障 -> 李四
```



### 3、代码解析

- MessageSender（接口）定义了**消息发送方式**，SmsSender 和 EmailSender 具体实现它。
- AbstractMessage（抽象类）定义了**消息类型**，如普通消息、加急消息。
- NormalMessage 和 UrgentMessage 继承 AbstractMessage，并通过**组合方式**引用 MessageSender，使得消息类型和发送方式**解耦**。
- **扩展性高**：如果未来要增加 **“微信发送”**，只需新增 WeChatSender，不影响已有代码。



### 4、使用总结

桥接模式的关键点：

1. 用组合代替继承，抽象部分与实现部分分离，避免类爆炸。
2. 各部分可以独立扩展，新增消息类型或发送方式时，互不影响。
3. 提高代码可维护性，减少代码耦合。

适用场景：

- 当一个类有多个变化维度（如“消息类型” & “发送方式”），并且这些维度需要独立扩展时。
- 需要避免继承导致的类爆炸，用组合来代替继承。
- JDBC 的 DriverManager 也有桥接模式的影子（不同数据库驱动解耦）。



## 4、桥接模式实现「实现三，升级实现二」

需求背景：在实现二的基础上，补充如下2点需求：

- **如果你要为 “加急消息” 添加一个新的功能，比如 “是否已读” 记录状态，你会怎么做？**
- **如果要扩展支持 “特急消息” 和 “企业微信发送”，如何最小改动代码？**

需求拆解我们将在原有 桥接模式的基础上，进行以下扩展：

- **为“加急消息”添加“是否已读”状态**。
- **新增“特急消息”类型**，继承抽象消息类并添加相应逻辑。
- **新增“企业微信发送”方式**，实现消息发送接口。

```java
import java.util.HashMap;
import java.util.Map;

// 1. 发送方式接口（Implementor）
interface MessageSender {
    void send(String message, String toUser);
}

// 2. 具体实现部分（短信发送）
class SmsSender implements MessageSender {
    @Override
    public void send(String message, String toUser) {
        System.out.println("【短信发送】" + message + " -> " + toUser);
    }
}

// 3. 具体实现部分（邮件发送）
class EmailSender implements MessageSender {
    @Override
    public void send(String message, String toUser) {
        System.out.println("【邮件发送】" + message + " -> " + toUser);
    }
}

// 4. 新增：企业微信发送
class WeChatSender implements MessageSender {
    @Override
    public void send(String message, String toUser) {
        System.out.println("【企业微信发送】" + message + " -> " + toUser);
    }
}

// 5. 抽象消息类（Abstraction）
abstract class AbstractMessage {
    protected MessageSender messageSender; // 组合实现方式

    public AbstractMessage(MessageSender messageSender) {
        this.messageSender = messageSender;
    }

    public abstract void sendMessage(String message, String toUser);
}

// 6. 普通消息类（Refined Abstraction）
class NormalMessage extends AbstractMessage {
    public NormalMessage(MessageSender messageSender) {
        super(messageSender);
    }

    @Override
    public void sendMessage(String message, String toUser) {
        messageSender.send("[普通] " + message, toUser);
    }
}

// 7. 加急消息类（Refined Abstraction），增加“已读”状态
class UrgentMessage extends AbstractMessage {
    private Map<String, Boolean> readStatus = new HashMap<>(); // 存储已读状态

    public UrgentMessage(MessageSender messageSender) {
        super(messageSender);
    }

    @Override
    public void sendMessage(String message, String toUser) {
        messageSender.send("[加急] " + message, toUser);
        readStatus.put(toUser, false); // 默认未读
    }

    public void markAsRead(String toUser) {
        if (readStatus.containsKey(toUser)) {
            readStatus.put(toUser, true);
            System.out.println("用户 " + toUser + " 已读加急消息");
        } else {
            System.out.println("用户 " + toUser + " 未收到加急消息");
        }
    }
}

// 8. 新增特急消息（Refined Abstraction）
class VeryUrgentMessage extends AbstractMessage {
    public VeryUrgentMessage(MessageSender messageSender) {
        super(messageSender);
    }

    @Override
    public void sendMessage(String message, String toUser) {
        messageSender.send("[特急] ❗❗ " + message, toUser);
    }
}

// 9. 测试代码
public class BridgePatternEnhanced {
    public static void main(String[] args) {
        // 发送普通短信
        AbstractMessage normalSms = new NormalMessage(new SmsSender());
        normalSms.sendMessage("会议通知", "张三");

        // 发送加急邮件，并标记已读
        UrgentMessage urgentEmail = new UrgentMessage(new EmailSender());
        urgentEmail.sendMessage("服务器故障", "李四");
        urgentEmail.markAsRead("李四");

        // 发送特急消息到企业微信
        AbstractMessage veryUrgentWeChat = new VeryUrgentMessage(new WeChatSender());
        veryUrgentWeChat.sendMessage("数据库崩溃，请立即处理！", "王五");
    }
}
```

```
【短信发送】[普通] 会议通知 -> 张三
【邮件发送】[加急] 服务器故障 -> 李四
用户 李四 已读加急消息
【企业微信发送】[特急] ❗❗ 数据库崩溃，请立即处理！ -> 王五
```

- 实现“加急消息”添加已读功能：使用 Map<String, Boolean> 存储 **收件人已读状态**，并提供 markAsRead() 方法标记已读。
- 扩展支持“特急消息”：新增 VeryUrgentMessage，在消息前增加“❗❗”提高紧急程度。
- 扩展支持“企业微信发送”：新增 WeChatSender 实现 MessageSender 接口，支持企业微信消息发送。



## 5、桥接模式应用

### 1、JDBC 桥接模式

我们对Driver接口一定不陌生，如果从桥接模式来看，Driver就是一个接口，下面可以有MySQL的Driver，Oracle的Driver，这些就可以当做实现接口类。

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    //...
    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
    //...
}
```

```java
public class DriverManager {
    //...
    public static Connection getConnection(String url, String user, String password) throws SQLException {
        java.util.Properties info = new java.util.Properties();

        if (user != null) {
            info.put("user", user);
        }
        if (password != null) {
            info.put("password", password);
        }

        return (getConnection(url, info, Reflection.getCallerClass()));
    }
    //...
}
```

上面是简化的代码，可以看到需要返回Connection对象，在Java中通过Connection提供给各个数据库一样的操作接口，这里的Connection可以看作抽象类，可以说我们用来操作不同数据库的方法都是相同的，不过MySQL有自己的实现类，同样Oracle也有对应的实现类，这里Driver和Connection之间是通过DriverManager类进行桥接的，不是像我们上面说的那样用组合关系来进行桥接。

所以到了这里我们必须要明白，设计模式固然很好，现在也已经形成了一定的流程步骤，但是在早期，可能程序语言的开发者都没有意识到自己用的这是桥接模式，因此，有时候框架中的一些设计模式并不是一定和我们学习的是一样的结构。



### 2、JDBC 是否真正属于桥接模式？

Driver 和 Connection 通过 DriverManager 桥接，并不完全属于桥接模式（Bridge Pattern），但它确实具备某些桥接模式的特征。下面从 **桥接模式的定义** 和 **JDBC 结构分析** 来解释它的设计模式特性。

1、桥接模式（Bridge Pattern）的定义

桥接模式 主要用于解耦抽象部分和实现部分，使它们可以独立变化。通常包含以下两个核心点：

- 抽象部分（Abstraction）：定义高层接口，不依赖具体实现。
- 实现部分（Implementation）：提供具体实现，通常通过组合方式连接抽象部分，而不是继承。
- **关键点：抽象和实现解耦，使其可以独立扩展。**

***

2、JDBC 结构分析

在 JDBC 体系 中，DriverManager 充当了 Driver（驱动程序） 和 Connection（数据库连接） 之间的管理桥梁：

- Driver（驱动接口）：不同数据库厂商提供的 JDBC 驱动，例如 MySQL Driver、PostgreSQL Driver。
- DriverManager（驱动管理器）：负责管理多个 Driver，并根据 URL 选择合适的驱动进行数据库连接。
- Connection（数据库连接接口）：表示数据库连接，由具体驱动实现，如 MySQLConnection、OracleConnection。

***

3、JDBC 是否属于桥接模式？

从设计模式的角度来看，JDBC 的 DriverManager **确实有桥接模式的特性**，但并不完全符合标准桥接模式：

- **抽象部分（Abstraction）：DriverManager**
- DriverManager 负责管理 Driver，使上层代码不直接依赖具体的数据库驱动。
- **实现部分（Implementation）：不同的 Driver 实现**
- 每个数据库厂商（MySQL、PostgreSQL、Oracle）提供自己的 Driver，它们实现了 Driver 接口。
- **解耦关系**：DriverManager 让应用程序**不直接依赖具体 Driver**，只需要通过 JDBC 规范与 DriverManager 交互，进而获取 Connection。

**但与标准桥接模式不同的是**：

- 桥接模式通常是 **组合关系**，而 DriverManager 更像 **一个管理中心**，它通过 **注册驱动、查找合适驱动并返回 Connection** 来运作，而不是直接持有 Driver 或 Connection 的实例。
- DriverManager 更像是 **工厂模式 + 代理模式**，因为它主要负责**驱动的注册与选择**，而不是简单的抽象与实现分离。



# 结构型：装饰器模式

## 1、装饰器模式介绍

装饰器模式的介绍：

- 在软件开发过程中，有时想用一些现存的组件，这些组件可能只是完成了一些核心功能，但在不改变其结构的情况下，可以动态地扩展其功能，所有这些都可以釆用装饰器模式来实现。装饰器模式（Decorator Pattern）的定义：指在不改变现有对象结构的情况下，动态地给该对象增加一些职责（即增加其额外功能）的模式，它属于对象结构型模式。


装饰器模式的优点：

- 装饰器是继承的有力补充，比继承灵活，在不改变原有对象的情况下，动态的给一个对象扩展功能，即插即用。
- 通过使用不用装饰类及这些装饰类的排列组合，可以实现不同效果。
- 装饰器模式完全遵守开闭原则。

装饰器模式的缺点：

- 装饰器模式会增加许多子类，过度使用会增加程序得复杂性。
- 多层装饰比较复杂。

装饰器模式的场景：

- 当需要给一个现有类添加附加职责，而又不能采用生成子类的方法进行扩充时。例如，该类被隐藏或者该类是终极类或者采用继承方式会产生大量的子类。
- 当需要通过对现有的一组基本功能进行排列组合而产生非常多的功能时，采用继承关系很难实现，而采用装饰器模式却很好实现。
- 当对象的功能要求可以动态地添加，也可以再动态地撤销时。

装饰器模式的角色：

- 抽象构件（Component）角色：定义一个抽象接口以规范准备接收附加责任的对象。
- 具体构件（ConcreteComponent）角色：实现抽象构件，通过装饰角色为其添加一些职责。
- 抽象装饰（Decorator）角色：继承抽象构件，并包含具体构件的实例，可以通过其子类扩展具体构件的功能。
- 具体装饰（ConcreteDecorator）角色：实现抽象装饰的相关方法，并给具体构件对象添加附加的责任。

```
Component（抽象组件）
    ↑
ConcreteComponent（具体组件）
    ↑
Decorator（抽象装饰者）
    ↑
ConcreteDecorator（具体装饰者）
```

![img](https://i-blog.csdnimg.cn/blog_migrate/fd70dfa9fc7f69928182cc1da9fb3cff.gif)

## 2、装饰器模式实现「一」

### 1、关系依赖图

![img](https://i-blog.csdnimg.cn/blog_migrate/0379003f72fc556d3ca34e3722ef24da.png)

### 2、图形接口类

- Shape

```java
public interface Shape {
    void draw();
}
```



### 3、图形实现类

- Circle

```java
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Shape: Circle");
    }
}
```

- Rectangle

```java
public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Shape: Rectangle");
    }
}
```



### 4、图形修饰者接口类

- ShapeDecorator

```java
public abstract class ShapeDecorator implements Shape {
    protected Shape decoratedShape;

    public ShapeDecorator(Shape decoratedShape) {
        this.decoratedShape = decoratedShape;
    }

    public void draw() {
        decoratedShape.draw();
    }
}
```



### 5、图形修饰者实现类

- RedShapeDecorator

```java
public class RedShapeDecorator extends ShapeDecorator {
    public RedShapeDecorator(Shape decoratedShape) {
        super(decoratedShape);
    }

    @Override
    public void draw() {
        decoratedShape.draw();
        setRedBorder(decoratedShape);
    }

    private void setRedBorder(Shape decoratedShape) {
        System.out.println("Add Shape Border Color: Red");
    }
}
```



### 6、最终测试类

- DecoratorPatternDemo

```java
public class DecoratorPatternDemo {
    public static void main(String[] args) {
        Shape circle = new Circle();
        circle.draw();
        System.out.println("====================");

        ShapeDecorator redCircle = new RedShapeDecorator(new Circle());
        redCircle.draw();
        System.out.println("====================");

        Shape rectangle = new Rectangle();
        rectangle.draw();
        System.out.println("====================");

        ShapeDecorator redRectangle = new RedShapeDecorator(new Rectangle());
        redRectangle.draw();
    }
}
```

```
Shape: Circle
====================
Shape: Circle
Add Shape Border Color: Red
====================
Shape: Rectangle
====================
Shape: Rectangle
Add Shape Border Color: Red
```



## 3、装饰器模式实现「二」

**需求：咖啡加料系统**，场景：我们要实现一个咖啡系统，支持**基础咖啡**，但用户可以选择额外添加 **牛奶、糖、巧克力**等配料，每种配料都会增加价格。

### 1、定义抽象组件（咖啡）

```java
// 抽象组件：咖啡
interface Coffee {
    String getDescription(); // 获取描述
    double getCost(); // 获取价格
}
```



### 2、具体组件（基础咖啡）

```java
// 具体组件：黑咖啡（基础咖啡）
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "黑咖啡";
    }

    @Override
    public double getCost() {
        return 10.0; // 基础价格
    }
}
```



### 3、抽象装饰者

```java
// 装饰者抽象类（必须继承 Coffee）
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee; // 被装饰的咖啡对象

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    @Override
    public String getDescription() {
        return coffee.getDescription(); // 调用被装饰者的方法
    }

    @Override
    public double getCost() {
        return coffee.getCost();
    }
}
```



### 4、具体装饰者

```java
// 具体装饰者：牛奶
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + " + 牛奶";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 2.0; // 额外加 2 元
    }
}

// 具体装饰者：糖
class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + " + 糖";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 1.5; // 额外加 1.5 元
    }
}
```



### 5、测试代码

```java
public class DecoratorPatternDemo {
    public static void main(String[] args) {
        // 1. 只有黑咖啡
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.getDescription() + " 价格: " + coffee.getCost());

        // 2. 黑咖啡 + 牛奶
        coffee = new MilkDecorator(coffee);
        System.out.println(coffee.getDescription() + " 价格: " + coffee.getCost());

        // 3. 黑咖啡 + 牛奶 + 糖
        coffee = new SugarDecorator(coffee);
        System.out.println(coffee.getDescription() + " 价格: " + coffee.getCost());
    }
}
```

```
黑咖啡 价格: 10.0
黑咖啡 + 牛奶 价格: 12.0
黑咖啡 + 牛奶 + 糖 价格: 13.5
```



## 4、装饰器模式应用

> 真实应用场景：
>
> Java I/O流：
>
> - InputStream -> FilterInputStream -> BufferedInputStream
> - Reader -> BufferedReader
>
> Spring AOP（切面编程）：
>
> - 使用代理模式实现动态增强功能（例如日志、权限、事务）。
>
> GUI 组件：
>
> - JComponent -> JScrollPane（为组件增加滚动条）。

### 1、FilterInputStream 装饰者模式

![img](https://i-blog.csdnimg.cn/blog_migrate/b018f957ee5961595b6c3ef4c1b35097.png)

在`java.io`包中的`FilterInputStream`就是一个装饰者类，`InputStream in`即被装饰者。

```java
public class FilterInputStream extends InputStream {
    protected volatile InputStream in; // 被装饰者（被包装的流）

    protected FilterInputStream(InputStream in) { // 依赖注入 InputStream
        this.in = in;
    }

    @Override
    public int read() throws IOException {
        return in.read(); // 直接调用被装饰者的 read() 方法
    }
    //...
}
```

- `DataInputStream`是`FilterInputStream`子类，则是具体的装饰者类。

```java
public class DataInputStream extends FilterInputStream implements DataInput {
    public DataInputStream(InputStream in) {
        super(in);
    }
    //...
}
```

- `BufferedInputStream`也是`FilterInputStream`子类，则是具体的装饰者类。

```java
public class BufferedInputStream extends FilterInputStream {
    private static final int DEFAULT_BUFFER_SIZE = 8192;
    private byte[] buffer; // 额外的缓冲区

    public BufferedInputStream(InputStream in) {
        super(in);
        this.buffer = new byte[DEFAULT_BUFFER_SIZE]; // 初始化缓冲
    }

    @Override
    public int read() throws IOException {
        // 从缓冲区读取，而不是直接访问底层流
        return super.read();
    }
}
```

```java
InputStream fileStream = new FileInputStream("test.txt"); 
InputStream bufferedStream = new BufferedInputStream(fileStream); // 用装饰者包装
bufferedStream.read(); // 先读取缓冲区，再访问底层流
```

- fileStream 是 **被装饰者（Concrete Component）**。
- bufferedStream 是 **装饰者（Concrete Decorator）**，它增强了 read() 方法，使其支持**缓冲读取**。



### 2、为什么不用继承？

- 如果 BufferedInputStream 直接继承 FileInputStream，它只能增强**文件流**，但无法增强其他 InputStream（如 ByteArrayInputStream）。
- 使用 **装饰者模式**，我们可以动态地**包装任何 InputStream**，增强它的功能，具有**更好的扩展性**。



### 3、最终结论

- **正确**：FilterInputStream 确实是 InputStream 的**装饰者**，而 in 变量是**被装饰者**。
- **符合装饰者模式**：它允许**动态增强** InputStream 的功能，如**缓冲读取、数据转换、加密等**。
- **优势**：相比继承方式，装饰者模式**更灵活、可扩展性更强**，可以随时组合不同功能的流。



# 结构型：组合模式

## 1、组合模式介绍

组合模式的介绍：

- 组合模式（Composite Pattern）又叫“整体-部分”模式，是用于把一组相似的对象当作一个单一的对象。它是一种将对象组合成树状的层次结构的模式，用来表示“整体-部分”的关系，使用户对单个对象和组合对象具有一致的访问性，属于结构型设计模式。组合模式分为透明式的组合模式和安全式的组合模式。

- 组合模式一般用来描述整体与部分的关系，它将对象组织到树形结构中，顶层的节点被称为根节点，根节点下面可以包含树枝节点和叶子节点，树枝节点下面又可以包含树枝节点和叶子节点，树形结构图如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/86416e4e87bcffcce120d2fdd0badb91.png)

- 由上图可以看出，其实根节点和树枝节点本质上属于同一种数据类型，可以作为容器使用，而叶子节点与树枝节点在语义上不属于同一种类型。但是在组合模式中，会把树枝节点和叶子节点看作属于同一种数据类型（用统一接口定义），让它们具备一致行为。

组合模式的优点：

- 组合模式使得客户端代码可以一致地处理单个对象和组合对象，无须关心自己处理的是单个对象，还是组合对象，这简化了客户端代码。

- 更容易在组合体内加入新的对象，客户端不会因为加入了新的对象而更改源代码，满足“开闭原则”。

组合模式的缺点：

- 设计较复杂，客户端需要花更多时间理清类之间的层次关系。

- 不容易限制容器中的构件。
- 不容易用继承的方法来增加构件的新功能。

组合模式的场景：

- 算术表达式包括操作数、操作符和另一个操作数，其中，另一个操作符也可以是操作数、操作符和另一个操作数。

- 在 Java awt 和 swing 中，对于 Button 和 Checkbox 是树叶，而 Container 是树枝。

组合模式的角色：

- 抽象构件（Component）角色：它的主要作用是为树叶构件和树枝构件声明公共接口，并实现它们的默认行为。在透明式的组合模式中抽象构件还声明访问和管理子类的接口；在安全式的组合模式中不声明访问和管理子类的接口，管理工作由树枝构件完成。

- 树叶构件（Leaf）角色：是组合中的叶节点对象，它没有子节点，用于继承或实现抽象构件。
- 树枝构件（Composite）角色 / 中间构件：是组合中的分支节点对象，它有子节点，用于继承和实现抽象构件。它的主要作用是存储和管理子部件，通常包含 Add()、Remove()、GetChild() 等方法。



## 2、组合模式实现（透明式）

### 1、透明式介绍

在该方式中，由于抽象构件声明了所有子类中的全部方法，所以客户端无须区别树叶对象和树枝对象，对客户端来说是透明的。但其缺点是：树叶构件本来没有 Add()、Remove() 及 GetChild() 方法，却要实现它们（空实现或抛异常），这样会带来一些安全性问题。其结构如下图：

![img](https://i-blog.csdnimg.cn/blog_migrate/3edae0dd5cf8d79b71f35d9bc45b8651.gif)

### 2、抽象构件

- Component

```java
public interface Component {
    public void add(Component c);

    public void remove(Component c);

    public Component getChild(int i);

    public void operation();
}
```



### 3、树叶构件

- Leaf

```java
public class Leaf implements Component {
    private String name;

    public Leaf(String name) {
        this.name = name;
    }

    public void add(Component c) {
    }

    public void remove(Component c) {
    }

    public Component getChild(int i) {
        return null;
    }

    public void operation() {
        System.out.println("树叶" + name + "：被访问！");
    }
}
```



### 4、树枝构件

- Composite

```java
public class Composite implements Component {
    private ArrayList<Component> children = new ArrayList<>();

    public void add(Component c) {
        children.add(c);
    }

    public void remove(Component c) {
        children.remove(c);
    }

    public Component getChild(int i) {
        return children.get(i);
    }

    public void operation() {
        for (Object obj : children) {
            ((Component) obj).operation();
        }
    }
}
```



### 5、客户端测试

- CompositePattern

```java
public class CompositePattern {
    public static void main(String[] args) {
        Component c0 = new Composite();
        Component c1 = new Composite();
        Component leaf1 = new Leaf("1");
        Component leaf2 = new Leaf("2");
        Component leaf3 = new Leaf("3");
        c0.add(leaf1);
        c0.add(c1);
        c1.add(leaf2);
        c1.add(leaf3);
        c0.operation();
    }
}
```

```
树叶1：被访问！
树叶2：被访问！
树叶3：被访问！
```



## 3、组合模式实现（安全式）

### 1、安全式介绍

在该方式中，将管理子构件的方法移到树枝构件中，抽象构件和树叶构件没有对子对象的管理方法，这样就避免了上一种方式的安全性问题，但由于叶子和分支有不同的接口，客户端在调用时要知道树叶对象和树枝对象的存在，所以失去了透明性。其结构如下图：

![img](https://i-blog.csdnimg.cn/blog_migrate/e846133860421ad7538e3e68a05874af.gif)

### 2、抽象构件

- Component

```java
public interface Component {
    public void operation();
}
```



### 3、树叶构件

- Leaf

```java
public class Leaf implements Component {
    private String name;

    public Leaf(String name) {
        this.name = name;
    }

    public void operation() {
        System.out.println("树叶" + name + "：被访问！");
    }
}
```



### 4、树枝构件

- Composite

```java
public class Composite implements Component {
    private ArrayList<Component> children = new ArrayList<>();

    public void add(Component c) {
        children.add(c);
    }

    public void remove(Component c) {
        children.remove(c);
    }

    public Component getChild(int i) {
        return children.get(i);
    }

    public void operation() {
        for (Object obj : children) {
            ((Component) obj).operation();
        }
    }
}
```



### 5、客户端测试

- CompositePattern

```java
public class CompositePattern {
    public static void main(String[] args) {
        Composite c0 = new Composite();
        Composite c1 = new Composite();
        Component leaf1 = new Leaf("1");
        Component leaf2 = new Leaf("2");
        Component leaf3 = new Leaf("3");
        c0.add(leaf1);
        c0.add(c1);
        c1.add(leaf2);
        c1.add(leaf3);
        c0.operation();
    }
}
```



## 4、组合模式应用

HashMap中有一个putAll方法，参数是一个Map，这就是一种组合模式的体现：

```java
public void putAll(Map<? extends K, ? extends V> m) {
    putMapEntries(m, true);
}
```

ArrayList中有一个addAll方法，参数是一个Collection，这就是一种组合模式的体现：

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    modCount++;
    int numNew = a.length;
    if (numNew == 0)
        return false;
    Object[] elementData;
    final int s;
    if (numNew > (elementData = this.elementData).length - (s = size))
        elementData = grow(s + numNew);
    System.arraycopy(a, 0, elementData, s, numNew);
    size = s + numNew;
    return true;
}
```



# 结构型：外观模式

## 1、外观模式介绍

外观模式的介绍：

- 在现实生活中，常常存在办事较复杂的例子，如办房产证或注册一家公司，有时要同多个部门联系，这时要是有一个综合部门能解决一切手续问题就好了。

- 软件设计也是这样，当一个系统的功能越来越强，子系统会越来越多，客户对系统的访问也变得越来越复杂。这时如果系统内部发生改变，客户端也要跟着改变，这违背了“开闭原则”，也违背了“迪米特法则”，所以有必要为多个子系统提供一个统一的接口，从而降低系统的耦合度，这就是外观模式的目标。

- 外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。


外观模式的优点：

- 降低了子系统与客户端之间的耦合度，使得子系统的变化不会影响调用它的客户类。

- 对客户屏蔽了子系统组件，减少了客户处理的对象数目，并使得子系统使用起来更加容易。
- 降低了大型软件系统中的编译依赖性，简化了系统在不同平台之间的移植过程，因为编译一个子系统不会影响其他的子系统，也不会影响外观对象。

外观模式的缺点：

- 不能很好地限制客户使用子系统类，很容易带来未知风险。

- 增加新的子系统可能需要修改外观类或客户端的源代码，违背了“开闭原则”。

外观模式的场景：

- 对分层结构系统构建时，使用外观模式定义子系统中每层的入口点可以简化子系统之间的依赖关系。

- 当一个复杂系统的子系统很多时，外观模式可以为系统设计一个简单的接口供外界访问。
- 当客户端与多个子系统之间存在很大的联系时，引入外观模式可将它们分离，从而提高子系统的独立性和可移植性。

外观模式的角色：

- 外观（Facade）角色：为多个子系统对外提供一个共同的接口。
- 子系统（Sub System）角色：实现系统的部分功能，客户可以通过外观角色访问它。
- 客户（Client）角色：通过一个外观角色访问各个子系统的功能。

![img](https://i-blog.csdnimg.cn/blog_migrate/153bbda437d8047836d7dfbe799c34e2.gif)

## 2、外观模式实现

### 1、关系依赖图

![img](https://i-blog.csdnimg.cn/blog_migrate/eeb42f916076cea751142f82e4bbca8b.png)

### 2、图形接口类

- Shape

```java
public interface Shape {
    void draw();
}
```



### 3、图形实现类

- Circle

```java
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Circle::draw()");
    }
}
```

- Rectangle

```java
public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Rectangle::draw()");
    }
}
```

- Square

```java
public class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Square::draw()");
    }
}
```



### 4、外观角色类

- ShapeMaker

```java
public class ShapeMaker {
    private Shape circle;
    private Shape rectangle;
    private Shape square;

    public ShapeMaker() {
        circle = new Circle();
        rectangle = new Rectangle();
        square = new Square();
    }

    public void drawCircle() {
        circle.draw();
    }

    public void drawRectangle() {
        rectangle.draw();
    }

    public void drawSquare() {
        square.draw();
    }
}
```



### 5、最终测试类

- FacadePatternDemo

```java
public class FacadePatternDemo {
    public static void main(String[] args) {
        ShapeMaker shapeMaker = new ShapeMaker();

        shapeMaker.drawCircle();
        shapeMaker.drawRectangle();
        shapeMaker.drawSquare();
    }
}
```

```
Circle::draw()
Rectangle::draw()
Square::draw()
```



## 3、外观模式应用

在`Mybatis`源码`org.apache.ibatis.session.Configuration`类中以 `new` 开头的方法，下图以`newExecutor`方法为例：

![img](https://i-blog.csdnimg.cn/blog_migrate/59f1243d3cd606f7b14018cdf88fa5c4.png)

# 结构型：享元模式

## 1、享元模式介绍

享元模式的介绍：

- 在面向对象程序设计过程中，有时会面临要创建大量相同或相似对象实例的问题。创建那么多的对象将会耗费很多的系统资源，它是系统性能提高的一个瓶颈。例如，围棋和五子棋中的黑白棋子，教室里的桌子和凳子等。这些对象有很多相似的地方，如果能把它们相同的部分提取出来共享，则能节省大量的系统资源，这就是享元模式的产生背景。

- 享元模式（Flyweight Pattern）也叫蝇量模式，主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。享元模式的定义提出了两个要求，细粒度和共享对象。因为要求细粒度，所以不可避免地会使对象数量多且性质相近，此时我们就将这些对象的信息分为两个部分：内部状态和外部状态。
  - 内部状态指对象共享出来的信息，存储在享元信息内部，并且不会随环境的改变而改变。例如围棋的棋子颜色：黑色、白色。
  - 外部状态指对象得以依赖的一个标记，随环境的改变而改变，不可共享。例如围棋的棋子位置：x横坐标、y纵坐标。

享元模式的优点：

- 相同对象只要保存一份，这降低了系统中对象的数量，从而降低了系统中细粒度对象给内存带来的压力。

享元模式的缺点：

- 为了使对象可以共享，需要将一些不能共享的状态外部化，这将增加程序的复杂性。

- 读取享元模式的外部状态会使得运行时间稍微变长。

享元模式的场景：

- 系统中存在大量相同或相似的对象，这些对象耗费大量的内存资源。

- 大部分的对象可以按照内部状态进行分组，且可将不同部分外部化，这样每一个组只需保存一个内部状态。
- 由于享元模式需要额外维护一个保存享元的数据结构，所以应当在有足够多的享元实例时才值得使用享元模式。

享元模式的角色：

- 抽象享元（Flyweight）角色：是所有的具体享元类的基类，为具体享元规范需要实现的公共接口，非享元的外部状态以参数的形式通过方法传入。

- 具体享元（Concrete Flyweight）角色：实现抽象享元角色中所规定的接口。
- 非享元（Unsharable Flyweight）角色：是不可以共享的外部状态，它以参数的形式注入具体享元的相关方法中。
- 享元工厂（Flyweight Factory）角色：负责创建和管理享元角色。当客户对象请求一个享元对象时，享元工厂检査系统中是否存在符合要求的享元对象，如果存在则提供给客户；如果不存在的话，则创建一个新的享元对象。

![img](https://i-blog.csdnimg.cn/blog_migrate/7c06b3ab39f37ab3250c15ec8ca0d8be.png)

## 2、享元模式实现

### 1、非享元对象

- Point

```java
public class Point {
    private int x;  // 外部状态：代表棋子横坐标
    private int y;  // 外部状态：代表棋子纵坐标

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public void setX(int x) {
        this.x = x;
    }

    public int getY() {
        return y;
    }

    public void setY(int y) {
        this.y = y;
    }
}
```



### 2、抽象享元对象

- ChessPieces

```java
public interface ChessPieces {
    void downPieces(Point p); // 落子方法
}
```



### 3、具体享元对象

- WhiteChessPieces

```java
public class WhiteChessPieces implements ChessPieces {
    private String color;   // 内部状态：代表棋子颜色

    public WhiteChessPieces() {
        this.color = "白";
    }

    @Override
    public void downPieces(Point p) {
        System.out.println(color + "子落入横坐标" + p.getX() + "，纵坐标" + p.getY());
    }
}
```

- BlackChessPieces

```java
public class BlackChessPieces implements ChessPieces {
    private String color;   // 内部状态：代表棋子颜色

    public BlackChessPieces() {
        this.color = "黑";
    }

    @Override
    public void downPieces(Point p) {
        System.out.println(color + "子落入横坐标" + p.getX() + "，纵坐标" + p.getY());
    }
}
```



### 4、享原工厂对象

- ChessPiecesFactory

```java
public class ChessPiecesFactory {
    private HashMap<String, ChessPieces> pools = new HashMap<>();

    public ChessPieces getChessPieces(String color) {
        if (!pools.containsKey(color)) {
            if ("白".equals(color)) {
                pools.put(color, new WhiteChessPieces());
            }
            if ("黑".equals(color)) {
                pools.put(color, new BlackChessPieces());
            }
        }
        return pools.get(color);
    }
}
```



### 5、最终测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        ChessPiecesFactory chessPiecesFactory = new ChessPiecesFactory();

        ChessPieces cp1 = chessPiecesFactory.getChessPieces("白");
        ChessPieces cp2 = chessPiecesFactory.getChessPieces("白");
        ChessPieces cp3 = chessPiecesFactory.getChessPieces("黑");
        ChessPieces cp4 = chessPiecesFactory.getChessPieces("黑");

        Point p1 = new Point(1, 1);
        Point p2 = new Point(2, 2);
        Point p3 = new Point(3, 3);
        Point p4 = new Point(4, 4);

        cp1.downPieces(p1);
        cp2.downPieces(p2);
        cp3.downPieces(p3);
        cp4.downPieces(p4);

        System.out.println("====================");

        System.out.println(cp1 == cp2);
        System.out.println(cp3 == cp4);
    }
}
```

```
白子落入横坐标1，纵坐标1
白子落入横坐标2，纵坐标2
黑子落入横坐标3，纵坐标3
黑子落入横坐标4，纵坐标4
====================
true
true
```



## 3、享元模式应用

如果 Integer.valueOf(x) 中的x在 `-128 ~ 127` 之间，就是使用享元模式返回，如果不在该范围，则仍然 new。

```java
@IntrinsicCandidate
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

我们可以单独使用一个测试类来进行测试：

```java
Integer a1 = Integer.valueOf(12);
Integer a2 = Integer.valueOf(12);
System.out.println(a1 == a2);

Integer b1 = Integer.valueOf(128);
Integer b2 = Integer.valueOf(128);
System.out.println(b1 == b2);
```

```
true
false
```



# 行为型：模板方法模式

## 1、模板模式介绍

模板模式的介绍：

- 模板模式（Template Pattern）定义了一个算法的执行框架，**固定不可变部分**在父类中实现，**可变部分**由子类继承实现。这种模式属于**行为型模式**。
- 银行业务办理流程：
  - **固定部分**（父类实现）：取号 → 排队 → 评分。
  - **变化部分**（子类实现）：办理业务（存款、取款、转账）

模板模式的优点：

- 复用父类代码，减少重复，提高**代码复用性**。
- 允许子类扩展部分行为，符合**开闭原则**。

模板模式的缺点：

- 每个不同实现都需要创建**子类**，可能增加系统复杂度。
- 子类影响父类执行，可能**提高代码阅读难度**。
- 由于继承关系自身的缺点，如果父类添加新的抽象方法，则所有子类都要改一遍。

模板模式的场景：

- **固定流程**，部分步骤可变，如数据解析、工作流管理等。
- **多个子类有相同逻辑**，可抽取公共部分。
- **控制子类扩展**，只允许在特定步骤修改行为。

模板模式的角色：

- **抽象模板类（Abstract Class）**：定义模板方法和基本方法。
  - **模板方法**：固定算法结构，调用基本方法「按某种顺序调用其包含的基本方法。」。
  - **基本方法**：抽象方法（子类实现）、具体方法（父类实现）、钩子方法（子类可选重写）。
- **具体子类（Concrete Class）**：实现抽象方法，定义可变逻辑。

![img](https://i-blog.csdnimg.cn/blog_migrate/15bcb0fa3ca512dba0fd27b8350b57dd.gif)



## 2、模板模式实现

### 1、抽象父类

- Game

```java
public abstract class Game {
    abstract void initialize();
    abstract void startPlay();
    abstract void endPlay();

    // 模板方法
    public final void play() {
        initialize(); // 初始化游戏
        startPlay();  // 开始游戏
        endPlay();    // 结束游戏
    }
}
```



### 2、具体子类「Football、Basketball」

```java
public class Football extends Game {
    @Override
    void initialize() {
        System.out.println("Football Game Initialized.");
    }

    @Override
    void startPlay() {
        System.out.println("Football Game Started.");
    }

    @Override
    void endPlay() {
        System.out.println("Football Game Finished.");
    }
}
```

```java
public class Basketball extends Game {
    @Override
    void initialize() {
        System.out.println("Basketball Game Initialized.");
    }

    @Override
    void startPlay() {
        System.out.println("Basketball Game Started.");
    }

    @Override
    void endPlay() {
        System.out.println("Basketball Game Finished.");
    }
}
```



### 3、测试类「Client」

```java
public class Client {
    public static void main(String[] args) {
        Game g1 = new Football();
        g1.play();

        Game g2 = new Basketball();
        g2.play();
    }
}
```

```
Football Game Initialized.
Football Game Started.
Football Game Finished.
Basketball Game Initialized.
Basketball Game Started.
Basketball Game Finished.
```



## 3、模板模式应用

在 Servlet 中，每一个 Servlet 类都必须要实现 HttpServlet 接口。GenericServlet 是个通用的、不特定于任何协议的 Servlet，它实现了 Servlet 接口。而 HttpServlet 继承于 GenericServlet，为 Servlet 接口提供了处理 HTTP 协议的通用实现，所以我们定义 Servlet 时只需要继承 HttpServlet 即可。

```java
public abstract class HttpServlet extends GenericServlet {
    // 以上代码部分省略...
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
        throws ServletException, IOException
    {
        // 省略...
    }

    protected void doHead(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        // 省略...
    }


    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        // 省略...
    }

    protected void doPut(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        // 省略...
    }

    protected void doDelete(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        // 省略...
    }

    protected void doOptions(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        // 省略...
    }

    protected void doTrace(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        // 省略...
    }


    protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException 
    {

        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                } catch (IllegalArgumentException iae) {
                    // Invalid date header - proceed as if none was set
                    ifModifiedSince = -1;
                }
                if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);

        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);

        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);

        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);

        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);

        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);

            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }

    // 以下部分代码省略...
}
```

在 HttpServlet 的 service 方法中，首先获取请求的方法名，然后根据方法名调用对应的 doXXX 方法，比如说请求方法为 GET，那么就去调用 doGet 方法；请求方法为 POST，那么就去调用 doPost 方法。

这里，HttpServlet 就相当于定义了一套处理 HTTP 请求的模板。service 方法为模板方法，定义了处理 HTTP 请求的基本流程；doXXX 等方法为基本方法，根据请求方法做相应的处理，子类可重写这些方法。HttpServletRequest 中的 Method 则起到钩子方法的作用。

比如实现一个输出 Hello World 的简单 Servlet，代码如下：

```java
public class HelloWorld extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response) 
        throws ServletException, IOException 
    {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<h1>Hello World!</h1>");
    }
}
```



# 行为型：职责链模式

## 1、责任链模式介绍

责任链模式的介绍：

- 责任链模式（Chain of Responsibility Pattern）通过**链式处理请求**，实现请求的**发送者**与**接收者**解耦。每个处理者包含对下一个处理者的引用，若无法处理请求，则**传递给下一个处理者**，直到请求被处理或到达链尾。
- 公司请假审批流程：
  - **部门主管** → **副总经理** → **总经理**。
  - 每个领导能批准的天数不同，无法处理时交给上一级审批。

责任链模式的优点：

- **降低耦合**：请求发送者无需关心具体处理者。
- **增强扩展性**：可动态增加/调整处理者，符合**开闭原则**。
- **职责明确**：每个处理者只处理自身职责范围内的请求。

责任链模式的缺点：

- **请求可能无法处理**，导致请求到达链尾仍无响应。
- **职责链过长**会影响系统性能，增加处理时间。
- **客户端需确保链的合理性**，避免**循环调用**等错误。

责任链模式的场景：

- 可动态指定一组对象处理请求，或添加新的处理者。
- 多个对象可以处理一个请求，但具体由哪个对象处理该请求在运行时自动确定。
- 需要在不明确指定请求处理者的情况下，向多个处理者中的一个提交请求。

责任链模式的角色：

| 角色                               | 说明                                               |
| ---------------------------------- | -------------------------------------------------- |
| **抽象处理者（Handler）**          | 定义请求处理接口，包含处理方法和下一个处理者引用。 |
| **具体处理者（Concrete Handler）** | 判断是否能处理请求，若不能则传递给下一个处理者。   |
| **客户端（Client）**               | 创建处理链并提交请求，无需关心请求如何被处理。     |

![img](https://i-blog.csdnimg.cn/blog_migrate/ed563a96931e0cfc9e8c6d00cbecb0d3.gif)

![img](https://i-blog.csdnimg.cn/blog_migrate/0380f18b26204d532249620ba867bbd3.gif)



## 2、责任链模式实现「一」

**需求背景**：采购员采购教学器材，请设计程序完成采购审批项目。

- 如果金额小于等于 5000，由主任审批 （0<=x<=5000）
- 如果金额小于等于 10000，由院长审批（5000<x<=10000）
- 如果金额小于等于 30000，由副校长审批（10000<x<=30000）
- 如果金额超过 30000 以上，有校长审批（30000<x）



### 1、抽象处理者「Approver」

```java
public abstract class Approver {
    protected Approver approver; // 下一个处理者
    protected String name;

    public Approver(String name) {
        this.name = name;
    }

    public Approver getNext() {
        return approver;
    }

    public void setNext(Approver approver) {
        this.approver = approver;
    }

    public abstract void handleRequest(PurchaseRequest purchaseRequest);
}
```



### 2、具体处理者「XxxApprover」

```java
// 主任审批
public class DepartmentApprover extends Approver {
    public DepartmentApprover(String name) {
        super(name);
    }

    @Override
    public void handleRequest(PurchaseRequest purchaseRequest) {
        if (purchaseRequest.getPrice() < 0 && purchaseRequest.getPrice() <= 5000) {
            System.out.println("本次采购请求被 " + this.name + " 处理");
        } else {
            if (this.approver != null) {
                approver.handleRequest(purchaseRequest);
            }
        }
    }
}
```

```java
// 院长审批
public class DeanApprover extends Approver {
    public DeanApprover(String name) {
        super(name);
    }

    @Override
    public void handleRequest(PurchaseRequest purchaseRequest) {
        if (purchaseRequest.getPrice() < 5000 && purchaseRequest.getPrice() <= 10000) {
            System.out.println("本次采购请求被 " + this.name + " 处理");
        } else {
            if (this.approver != null) {
                approver.handleRequest(purchaseRequest);
            }
        }
    }
}
```

```java
// 副校长审批
public class ViceSchoolMasterApprover extends Approver {
    public ViceSchoolMasterApprover(String name) {
        super(name);
    }

    @Override
    public void handleRequest(PurchaseRequest purchaseRequest) {
        if (purchaseRequest.getPrice() < 10000 && purchaseRequest.getPrice() <= 30000) {
            System.out.println("本次采购请求被 " + this.name + " 处理");
        } else {
            if (this.approver != null) {
                approver.handleRequest(purchaseRequest);
            }
        }
    }
}
```

```java
// 校长审批
public class SchoolMasterApprover extends Approver {
    public SchoolMasterApprover(String name) {
        super(name);
    }

    @Override
    public void handleRequest(PurchaseRequest purchaseRequest) {
        if (purchaseRequest.getPrice() > 30000) {
            System.out.println("本次采购请求被 " + this.name + " 处理");
        } else {
            if (this.approver != null) {
                approver.handleRequest(purchaseRequest);
            }
        }
    }
}
```



### 3、最终测试类「Client」

```java
// 请求参数类
class PurchaseRequest {
    private int price;
    public PurchaseRequest(int price) {
        this.price = price;
    }
    public int getPrice() {
        return this.price;
    }
}

public class Client {
    public static void main(String[] args) {
        // 创建一个请求
        PurchaseRequest purchaseRequest = new PurchaseRequest(31000);
        // 创建审批人员
        DepartmentApprover departmentApprover = new DepartmentApprover("张主任");
        DeanApprover deanApprover = new DeanApprover("李院长");
        ViceSchoolMasterApprover viceSchoolMasterApprover = new ViceSchoolMasterApprover("王副校");
        SchoolMasterApprover schoolMasterApprover = new SchoolMasterApprover("赵校长");
        // 设置审批链
        departmentApprover.setNext(deanApprover);
        deanApprover.setNext(viceSchoolMasterApprover);
        viceSchoolMasterApprover.setNext(schoolMasterApprover);
        // 开始处理请求
        departmentApprover.handleRequest(purchaseRequest);
    }
}
```

```
本次采购请求被 赵校长 处理
```



## 3、责任链模式实现「二」

```
+------------------+      +-------------------+      +-------------------+
| Client (请求者)    | --> | Handler (处理者)   | ---> | ConcreteHandlerA  |
+------------------+      +-------------------+      +-------------------+
                                  |
                                  v
                          +-------------------+
                          | ConcreteHandlerB  |
                          +-------------------+
```

- Handler（抽象处理者）： 定义处理请求的接口，并持有下一个处理者的引用。
- ConcreteHandler（具体处理者）： 继承 Handler，实现具体的处理逻辑，如果无法处理，就传递给下一个处理者。
- Client（请求者）： 发送请求，触发责任链。

**需求背景**：员工提交请假申请，审批流程如下：

- 普通员工（Leader） 可审批 1-3 天假期。
- 经理（Manager） 可审批 4-7 天假期。
- 总监（Director） 可审批 7 天以上的假期。



### 1、定义处理者接口

```java
// 处理者抽象类
abstract class LeaveHandler {
    protected LeaveHandler nextHandler; // 下一个处理者

    public void setNextHandler(LeaveHandler nextHandler) {
        this.nextHandler = nextHandler;
    }

    public abstract void handleRequest(int leaveDays);
}
```



### 2、具体处理者实现

```java
// 普通主管（审批 1-3 天）
class Leader extends LeaveHandler {
    @Override
    public void handleRequest(int leaveDays) {
        if (leaveDays <= 3) {
            System.out.println("Leader: 批准 " + leaveDays + " 天假期");
        } else if (nextHandler != null) {
            nextHandler.handleRequest(leaveDays);
        }
    }
}

// 经理（审批 4-7 天）
class Manager extends LeaveHandler {
    @Override
    public void handleRequest(int leaveDays) {
        if (leaveDays <= 7) {
            System.out.println("Manager: 批准 " + leaveDays + " 天假期");
        } else if (nextHandler != null) {
            nextHandler.handleRequest(leaveDays);
        }
    }
}

// 总监（审批 7 天以上）
class Director extends LeaveHandler {
    @Override
    public void handleRequest(int leaveDays) {
        System.out.println("Director: 批准 " + leaveDays + " 天假期");
    }
}
```



### 3、组装责任链

```java
public class ChainOfResponsibilityDemo {
    public static void main(String[] args) {
        // 创建责任链对象
        LeaveHandler leader = new Leader();
        LeaveHandler manager = new Manager();
        LeaveHandler director = new Director();

        // 组装责任链
        leader.setNextHandler(manager);
        manager.setNextHandler(director);

        // 测试不同天数的请假审批
        leader.handleRequest(2);  // Leader: 批准 2 天假期
        leader.handleRequest(5);  // Manager: 批准 5 天假期
        leader.handleRequest(10); // Director: 批准 10 天假期
    }
}
```

```
Leader: 批准 2 天假期
Manager: 批准 5 天假期
Director: 批准 10 天假期
```



## 4、责任链模式应用

在Java中，Servlet容器是责任链模式的经典应用。如在tomcat的包中，带有Filter、FilterChain结尾的类都是使用的是责任链模式。

`javax.servlet.Filter`类使用的就是责任链模式来实现对请求的过滤，执行任一请求到资源过滤任务。源码如下：

```java
public interface Filter {
    public default void init(FilterConfig filterConfig)
        throws ServletException {}
    
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException;
    
    public default void destroy() {}
}
```

`javax.Servlet.FilterChain`类也是通过责任链模式来进行过滤链中过滤器的调用。源码如下：

```java
public interface FilterChain {
    public void doFilter(ServletRequest request, ServletResponse response)
        throws IOException, ServletException;
}
```

- 处理请求的多个对象（多个 Filter）。
- 对象之间解耦（Filter 之间无须直接交互）。
- 按顺序传递请求（FilterChain.doFilter() 传递给下一个 Filter）。
- 支持部分拦截或终止请求（某个 Filter 可以决定不调用 chain.doFilter()）。



# 行为型：策略模式

## 1、策略模式介绍

策略模式的介绍：

- 在现实生活中常常遇到实现某种目标存在多种策略可供选择的情况，例如，出行旅游可以乘坐飞机、乘坐火车、骑自行车或自己开私家车等，超市促销可以釆用打折、送商品、送积分等方法。

- 在软件开发中也常常遇到类似的情况，当实现某一个功能存在多种算法或者策略，我们可以根据环境或者条件的不同选择不同的算法或者策略来完成该功能，如数据排序策略有冒泡排序、选择排序、插入排序、二叉树排序等。

- 如果使用多重条件转移语句实现（即硬编码），不但使条件语句变得很复杂，而且增加、删除或更换算法要修改原代码，不易维护，违背开闭原则，如果采用策略模式就能很好解决该问题。

- 在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象的改变而改变的 context 对象。策略对象改变 context 对象的执行算法。


策略模式的优点：

- 多重条件语句不易维护，而使用策略模式可以避免使用多重条件语句，如 if…else 语句、switch…case 语句。

- 策略模式提供了一系列的可供重用的算法族，恰当使用继承可以把算法族的公共代码转移到父类里面，从而避免重复的代码。
- 策略模式可以提供相同行为的不同实现，客户可以根据不同时间或空间要求选择不同的。
- 策略模式提供了对开闭原则的完美支持，可以在不修改原代码的情况下，灵活增加新算法。
- 策略模式把算法的使用放到环境类中，而算法的实现移到具体策略类中，实现了二者的分离。

策略模式的缺点：

- 客户端必须理解所有策略算法的区别，以便适时选择恰当的算法类。

- 策略模式造成很多的策略类，增加维护难度。

策略模式的场景：

- 一个系统需要动态地在几种算法中选择一种时，可将每个算法封装到策略类中。

- 一个类定义了多种行为，并且这些行为在这个类的操作中以多个条件语句的形式出现，可将每个条件分支移入它们各自的策略类中以代替这些条件语句。
- 系统中各算法彼此完全独立，且要求对客户隐藏具体算法的实现细节时。
- 系统要求使用算法的客户不应该知道其操作的数据时，可使用策略模式来隐藏与算法相关的数据结构。
- 多个类只区别在表现行为不同，可以使用策略模式，在运行时动态选择具体要执行的行为。

策略模式的角色：

- 抽象策略（Strategy）角色：定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，环境角色使用这个接口调用不同的算法，一般使用接口或抽象类实现。
- 具体策略（Concrete Strategy）角色：实现了抽象策略定义的接口，提供具体的算法实现。
- 环境（Context）角色：持有一个策略类的引用，最终给客户端调用。

![img](https://i-blog.csdnimg.cn/blog_migrate/644e600dfcddde48ba50022242c1baa3.gif)



## 2、策略模式实现

### 1、抽象策略

- Strategy

```java
public interface Strategy {
    void strategyMethod();
}
```



### 2、具体策略

- ConcreteStrategyA

```java
public class ConcreteStrategyA implements Strategy {
    public void strategyMethod() {
        System.out.println("具体策略A的策略方法被访问！");
    }
}
```

- ConcreteStrategyB

```java
public class ConcreteStrategyB implements Strategy {
    public void strategyMethod() {
        System.out.println("具体策略B的策略方法被访问！");
    }
}
```



### 3、环境类

- Context

```java
public class Context {
    private Strategy strategy;

    public Strategy getStrategy() {
        return strategy;
    }

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public void strategyMethod() {
        strategy.strategyMethod();
    }
}
```



### 4、测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        Context c = new Context();

        c.setStrategy(new ConcreteStrategyA());
        c.strategyMethod();

        c.setStrategy(new ConcreteStrategyB());
        c.strategyMethod();
    }
}
```

```
具体策略A的策略方法被访问！
具体策略B的策略方法被访问！
```



## 3、策略模式应用

JDK中Arrays这个类中的sort方法，Arrays就是一个环境角色类，这个sort方法你可以传一个新策略让Arrays根据这个方法来进行排序。

```java
public class Arrays {
    public static <T> void sort(T[] a, Comparator<? super T> c) {
        if (c == null) {
            sort(a);
        } else {
            if (LegacyMergeSort.userRequested)
                legacyMergeSort(a, c);
            else
                TimSort.sort(a, 0, a.length, c, null, 0, 0);
        }
    }    
}
```



# 行为型：命令模式

## 1、命令模式介绍

命令模式的介绍：

- 在软件开发系统中，“方法的请求者”与“方法的实现者”之间经常存在紧密的耦合关系，这不利于软件功能的扩展与维护。例如，想对方法进行“撤销、重做、记录”等处理都很不方便，因此“如何将方法的请求者与实现者解耦”变得很重要，命令模式就能很好地解决这个问题。

- 在现实生活中，命令模式的例子也很多。比如看电视时，我们只需要轻轻一按遥控器就能完成频道的切换，这就是命令模式，将换台请求和换台处理完全解耦了。电视机遥控器（命令发送者）通过按钮（具体命令）来遥控电视机（命令接收者）。

- 命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。


命令模式的优点：

- 通过引入中间件（抽象接口）降低系统的耦合度。

- 扩展性良好，增加或删除命令非常方便。采用命令模式增加与删除命令不会影响其他类，且满足“开闭原则”。
- 可以实现宏命令。命令模式可以与组合模式结合，将多个命令装配成一个组合命令，即宏命令。
- 方便实现 Undo 和 Redo 操作。命令模式可以与后面介绍的备忘录模式结合，实现命令的撤销与恢复。
- 可以在现有命令的基础上，增加额外功能。比如日志记录，结合装饰器模式会更加灵活。

命令模式的缺点：

- 可能产生大量具体的命令类。因为每一个具体操作都需要设计一个具体命令类，这会增加系统的复杂性。

- 命令模式的结果其实就是接收方的执行结果，但是为了以命令的形式进行架构、解耦请求与实现，引入了额外类型结构（引入了请求方与抽象命令接口），增加了理解上的困难。不过这也是设计模式的通病，抽象必然会额外增加类的数量，代码抽离肯定比代码聚合更加难理解。

命令模式的场景：

- 请求调用者需要与请求接收者解耦时，命令模式可以使调用者和接收者不直接交互。

- 系统随机请求命令或经常增加、删除命令时，命令模式可以方便地实现这些功能。
- 当系统需要执行一组操作时，命令模式可以定义宏命令来实现该功能。
- 当系统需要支持命令的撤销（Undo）操作和恢复（Redo）操作时，可以将命令对象存储起来，采用备忘录模式来实现。

命令模式的角色：

- 抽象命令类（Command）角色：声明执行命令的接口，拥有执行命令的抽象方法 execute()。
- 具体命令类（Concrete Command）角色：是抽象命令类的具体实现类，它拥有接收者对象，并通过调用接收者的功能来完成命令要执行的操作。
- 实现者/接收者（Receiver）角色：执行命令功能的相关操作，是具体命令对象业务的真正实现者。
- 调用者/请求者（Invoker）角色：是请求的发送者，它通常拥有很多的命令对象，并通过访问命令对象来执行相关请求，它不直接访问接收者。

![img](https://i-blog.csdnimg.cn/blog_migrate/affe44de6891ac4a12ce5b16610c7a6b.gif)



## 2、命令模式实现

### 1、抽象命令类

- Command

```java
public interface Command {
    // 执行动作(操作)
    public void execute();

    // 撤销动作(操作)
    public void undo();
}
```



### 2、具体命令类

- NoCommand

```java
public class NoCommand implements Command {
    @Override
    public void execute() {

    }

    @Override
    public void undo() {

    }
}
```

- TVOnCommand

```java
public class TVOnCommand implements Command {
    TVReceiver tv;

    public TVOnCommand(TVReceiver tv) {
        this.tv = tv;
    }

    @Override
    public void execute() {
        tv.on();
    }

    @Override
    public void undo() {
        tv.off();
    }
}
```

- TVOffCommand

```java
public class TVOffCommand implements Command {
    TVReceiver tv;

    public TVOffCommand(TVReceiver tv) {
        this.tv = tv;
    }

    @Override
    public void execute() {
        tv.off();
    }

    @Override
    public void undo() {
        tv.on();
    }
}
```



### 3、命令接收者

- TVReceiver

```java
public class TVReceiver {
    public void on() {
        System.out.println("电视机打开了... ");
    }

    public void off() {
        System.out.println("电视机关闭了... ");
    }
}
```



### 4、命令请求者

- RemoteControllerOne「复杂版本」

```java
public class RemoteController {
    Command[] onCommands;   // 开命令集合
    Command[] offCommands;  // 关命令集合
    Command undoCommand;    // 撤销命令

    public RemoteController() {
        onCommands = new Command[5];
        offCommands = new Command[5];

        for (int i = 0; i < 5; i++) {
            onCommands[i] = new NoCommand();
            offCommands[i] = new NoCommand();
        }
    }

    // 设置命令按钮
    public void setCommand(int no, Command onCommand, Command offCommand) {
        onCommands[no] = onCommand;
        offCommands[no] = offCommand;
    }

    // 按下开按钮
    public void onButtonWasPushed(int no) {
        // 找到你按下的开的按钮，并调用对应方法
        onCommands[no].execute();
        // 记录这次的操作，用于撤销
        undoCommand = onCommands[no];
    }

    // 按下关按钮
    public void offButtonWasPushed(int no) {
        // 找到你按下的关的按钮，并调用对应方法
        offCommands[no].execute();
        // 记录这次的操作，用于撤销
        undoCommand = offCommands[no];
    }

    // 按下撤销按钮
    public void undoButtonWasPushed() {
        undoCommand.undo();
    }
}
```

- RemoteControllerTwo「简单版本」

```java
// 调用者（遥控器）
class RemoteControl {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressButton() {
        command.execute();
    }

    public void pressUndo() {
        command.undo();
    }
}
```



### 5、最终测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        // 创建远程遥控器
        RemoteController remoteController = new RemoteController();
        // 创建电视机接收者
        TVReceiver tvReceiver = new TVReceiver();
        TVOffCommand tvOffCommand = new TVOffCommand(tvReceiver);
        TVOnCommand tvOnCommand = new TVOnCommand(tvReceiver);
        remoteController.setCommand(0, tvOnCommand, tvOffCommand);
        // 调用电视机的操作
        remoteController.onButtonWasPushed(0);
        remoteController.offButtonWasPushed(0);
        remoteController.undoButtonWasPushed();
    }
}
```

```
电视机打开了... 
电视机关闭了... 
电视机打开了... 
```

- CommandPatternDemo

```java
public class CommandPatternDemo {
    public static void main(String[] args) {
        TVReceiver tVReceiver = new TVReceiver(); // 创建接收者（电视）
        RemoteControl remote = new RemoteControl(); // 创建调用者（遥控器）

        // 绑定 "打开电视" 命令
        Command tVOn = new TVOnCommand(tVReceiver);
        remote.setCommand(tVOn);
        remote.pressButton(); // 电视打开了！
        remote.pressUndo();   // 电视关闭了！

        // 绑定 "关闭电视" 命令
        Command tVOff = new TVOffCommand(light);
        remote.setCommand(tVOff);
        remote.pressButton(); // 电视关闭了！
        remote.pressUndo();   // 电视打开了！
    }
}
```

```
电视机打开了... 
电视机关闭了... 
电视机打开了... 
电视机关闭了... 
```



## 3、命令模式应用

Spring 框架的 JdbcTemplate 就使用到了命令模式，不过并不是命令模式的标准实现，只是用到了命令模式的思想。

StatementCallback充当了抽象命令类。

```java
@FunctionalInterface
public interface StatementCallback<T> {
    @Nullable
    T doInStatement(Statement stmt) throws SQLException, DataAccessException;
}
```

以下图片中是实现了StatementCallback的所有具体命令类。

![img](https://i-blog.csdnimg.cn/blog_migrate/6d0d60e135b8cf4b1abf09202acd2a17.png)

这里QueryStatementCallback使用到了匿名内部类而不是单独写成一个类文件，它实现了命令接口， 同时也充当命令接收者。

```java
@Nullable
public <T> T query(final String sql, final ResultSetExtractor<T> rse) throws DataAccessException {
    Assert.notNull(sql, "SQL must not be null");
    Assert.notNull(rse, "ResultSetExtractor must not be null");
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Executing SQL query [" + sql + "]");
    }

    class QueryStatementCallback implements StatementCallback<T>, SqlProvider {
        QueryStatementCallback() {}

        @Nullable
        public T doInStatement(Statement stmt) throws SQLException {
            ResultSet rs = null;

            Object var3;
            try {
                rs = stmt.executeQuery(sql);
                var3 = rse.extractData(rs);
            } finally {
                JdbcUtils.closeResultSet(rs);
            }

            return var3;
        }

        public String getSql() {
            return sql;
        }
    }

    return this.execute(new QueryStatementCallback(), true);
}
```

命令调用者是 JdbcTemplate，其中 execute(StatementCallback<T> action, boolean closeResources) 方法中，调用 action.doInStatement(stmt) 方法，不同的实现 StatementCallback 接口的对象，对应不同的 doInStatemnt 实现逻辑。

```java
@Nullable
private <T> T execute(StatementCallback<T> action, boolean closeResources) throws DataAccessException {
    Assert.notNull(action, "Callback object must not be null");
    Connection con = DataSourceUtils.getConnection(this.obtainDataSource());
    Statement stmt = null;

    Object var12;
    try {
        stmt = con.createStatement();
        this.applyStatementSettings(stmt);
        T result = action.doInStatement(stmt);
        this.handleWarnings(stmt);
        var12 = result;
    } catch (SQLException var10) {
        String sql = getSql(action);
        JdbcUtils.closeStatement(stmt);
        stmt = null;
        DataSourceUtils.releaseConnection(con, this.getDataSource());
        con = null;
        throw this.translateException("StatementCallback", sql, var10);
    } finally {
        if (closeResources) {
            JdbcUtils.closeStatement(stmt);
            DataSourceUtils.releaseConnection(con, this.getDataSource());
        }

    }

    return var12;
}
```



# 行为型：状态模式

## 1、状态模式介绍

状态模式的介绍：

- 在软件开发过程中，应用程序中的部分对象可能会根据不同的情况做出不同的行为，我们把这种对象称为有状态的对象，而把影响对象行为的一个或多个动态变化的属性称为状态。当有状态的对象与外部事件产生互动时，其内部状态就会发生改变，从而使其行为也发生改变。如人都有高兴和伤心的时候，不同的情绪有不同的行为，当然外界也会影响其情绪变化。

- 对这种有状态的对象编程，传统的解决方案是：将这些所有可能发生的情况全都考虑到，然后使用 if-else 或 switch-case 语句来做状态判断，再进行不同情况的处理。但是显然这种做法对复杂的状态判断存在天然弊端，条件判断语句会过于臃肿，可读性差，且不具备扩展性，维护难度也大。且增加新的状态时要添加新的 if-else 语句，这违背了“开闭原则”，不利于程序的扩展。

- 以上问题如果采用“状态模式”就能很好地得到解决。在状态模式（State Pattern）中，类的行为是基于它的状态改变的。这种类型的设计模式属于行为型模式。在状态模式中，我们创建表示各种状态的对象和一个行为随着状态对象改变而改变的 context 对象。


状态模式的优点：

- 结构清晰，状态模式将与特定状态相关的行为局部化到一个状态中，并且将不同状态的行为分割开来，满足“单一职责原则”。

- 将状态转换显示化，减少对象间的相互依赖。将不同状态引入独立对象中会使得状态转换变得更加明确，且减少对象间的相互依赖。
- 状态类职责明确，有利于程序的扩展。通过定义新的子类很容易地增加新的状态和转换。

状态模式的缺点：

- 状态模式的使用必然会增加系统的类与对象的个数。

- 状态模式的结构与实现都较为复杂，如果使用不当会导致程序结构和代码的混乱。
- 状态模式对开闭原则的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源码，否则无法切换到新增状态，而且修改某个状态类的行为也需要修改对应类的源码。

状态模式的场景：

- 当一个对象的行为取决于它的状态，并且它必须在运行时根据状态改变它的行为时，就可以考虑使用状态模式。

- 当一个操作中含有庞大的分支结构，并且这些分支决定于对象的状态时。

状态模式的角色：

- 环境类（Context）角色：也称为上下文，它定义了客户端需要的接口，内部维护一个当前状态，并负责具体状态的切换。
- 抽象状态（State）角色：定义一个接口，用以封装环境对象中的特定状态所对应的行为，可以有一个或多个行为。
- 具体状态（Concrete State）角色：实现抽象状态所对应的行为，并且在需要的情况下进行状态切换。

![img](https://i-blog.csdnimg.cn/blog_migrate/fa016e293232cbc2ff6c227659e8c7c6.gif)



## 2、状态模式实现

### 1、环境类

- Context

```java
public class Context {
    private State state;

    public Context() {
        this.state = new ConcreteStateA();
    }

    // 设置状态
    public void setState(State state) {
        this.state = state;
    }

    // 读取状态
    public State getState() {
        return (state);
    }

    // 请求处理
    public void Handle() {
        state.Handle(this);
    }
}
```



### 2、抽象状态类

- State

```java
public abstract class State {
    public abstract void Handle(Context context);
}

```



3、具体状态类

- ConcreteStateA

```java
public class ConcreteStateA extends State {
    public void Handle(Context context) {
        System.out.println("当前状态是 A.");
        context.setState(new ConcreteStateB());
    }
}
```

- ConcreteStateB

```java
public class ConcreteStateB extends State {
    public void Handle(Context context) {
        System.out.println("当前状态是 B.");
        context.setState(new ConcreteStateA());
    }
}
```



### 4、测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        Context context = new Context();
        context.Handle();
        context.Handle();
        context.Handle();
        context.Handle();
    }
}
```

```
当前状态是 A.
当前状态是 B.
当前状态是 A.
当前状态是 B.
```



## 4、状态模式应用

多线程存在 5 种状态，分别为新建状态、就绪状态、运行状态、阻塞状态和死亡状态，各个状态当遇到相关方法调用或事件触发时会转换到其它状态，其状态转换规律如下图所示：

![img](https://i-blog.csdnimg.cn/blog_migrate/7b92bc9344581ea7d01c71518fdea75f.gif)

现在先定义一个抽象状态类（TheadState），然后为上图所示的每个状态设计一个具体状态类，它们是新建状态（New）、就绪状态（Runnable ）、运行状态（Running）、阻塞状态（Blocked）和死亡状态（Dead），每个状态中有触发它们转变状态的方法，环境类（ThreadContext）中先生成一个初始状态（New），并提供相关触发方法，下线程状态转换程序的结构如下图所示。

![img](https://i-blog.csdnimg.cn/blog_migrate/95a86cabadee51ac59aa1bc261ae188a.gif)

### 1、环境类

- ThreadContext

```java
public class ThreadContext {
    private ThreadState state;

    ThreadContext() {
        state = new New();
    }

    public void setState(ThreadState state) {
        this.state = state;
    }

    public ThreadState getState() {
        return state;
    }

    public void start() {
        ((New) state).start(this);
    }

    public void getCPU() {
        ((Runnable) state).getCPU(this);
    }

    public void suspend() {
        ((Running) state).suspend(this);
    }

    public void stop() {
        ((Running) state).stop(this);
    }

    public void resume() {
        ((Blocked) state).resume(this);
    }
}
```



### 2、抽象状态类

- ThreadState

```java
public abstract class ThreadState {
    protected String stateName;
}
```



### 3、具体状态类

- New

```java
public class New extends ThreadState {
    public New() {
        stateName = "新建状态";
        System.out.println("当前线程处于：新建状态.");
    }

    public void start(ThreadContext hj) {
        System.out.print("调用start()方法-->");
        if (stateName.equals("新建状态")) {
            hj.setState(new Runnable());
        } else {
            System.out.println("当前线程不是新建状态，不能调用start()方法.");
        }
    }
}
```

- Runnable

```java
public class Runnable extends ThreadState {
    public Runnable() {
        stateName = "就绪状态";
        System.out.println("当前线程处于：就绪状态.");
    }

    public void getCPU(ThreadContext hj) {
        System.out.print("获得CPU时间-->");
        if (stateName.equals("就绪状态")) {
            hj.setState(new Running());
        } else {
            System.out.println("当前线程不是就绪状态，不能获取CPU.");
        }
    }
}
```

- Running

```java
public class Running extends ThreadState {
    public Running() {
        stateName = "运行状态";
        System.out.println("当前线程处于：运行状态.");
    }

    public void suspend(ThreadContext hj) {
        System.out.print("调用suspend()方法-->");
        if (stateName.equals("运行状态")) {
            hj.setState(new Blocked());
        } else {
            System.out.println("当前线程不是运行状态，不能调用suspend()方法.");
        }
    }

    public void stop(ThreadContext hj) {
        System.out.print("调用stop()方法-->");
        if (stateName.equals("运行状态")) {
            hj.setState(new Dead());
        } else {
            System.out.println("当前线程不是运行状态，不能调用stop()方法.");
        }
    }
}
```

- Blocked

```java
public class Blocked extends ThreadState {
    public Blocked() {
        stateName = "阻塞状态";
        System.out.println("当前线程处于：阻塞状态.");
    }

    public void resume(ThreadContext hj) {
        System.out.print("调用resume()方法-->");
        if (stateName.equals("阻塞状态")) {
            hj.setState(new Runnable());
        } else {
            System.out.println("当前线程不是阻塞状态，不能调用resume()方法.");
        }
    }
}
```

- Dead

```java
public class Dead extends ThreadState {
    public Dead() {
        stateName = "死亡状态";
        System.out.println("当前线程处于：死亡状态.");
    }
}
```



### 4、测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        ThreadContext context = new ThreadContext();
        context.start();
        context.getCPU();
        context.suspend();
        context.resume();
        context.getCPU();
        context.stop();
    }
}
```

```
当前线程处于：新建状态.
调用start()方法-->当前线程处于：就绪状态.
获得CPU时间-->当前线程处于：运行状态.
调用suspend()方法-->当前线程处于：阻塞状态.
调用resume()方法-->当前线程处于：就绪状态.
获得CPU时间-->当前线程处于：运行状态.
调用stop()方法-->当前线程处于：死亡状态.
```



# 行为型：观察者模式

## 1、观察者模式介绍

观察者模式的介绍：

- 观察者模式（Observer Pattern）**定义一对多的依赖关系**，当**一个对象状态变化**时，所有**依赖它的对象**都会收到通知并自动更新。常用于**发布-订阅**（Pub-Sub）模式、模型-视图模式，它是对象行为型模式。
- 气象站向多个网站提供温度、湿度、气压数据：
  - 气象站（主题） → 多个网站（观察者）
  - 数据变化时，所有网站都会收到更新通知。

观察者模式的优点：

- **降低耦合**：目标与观察者解耦，符合**依赖倒置原则**。
- **自动触发**：观察者无需主动查询，目标对象状态变化时**自动通知**。

观察者模式的缺点：

- 目标与观察者之间**可能存在循环引用**，导致死循环问题。
- 大量观察者可能导致**通知耗时**，影响性能。

观察者模式的场景：

- **一对多依赖**：一个对象的状态变化会影响多个对象。
- **解耦通知机制**：无需直接维护所有订阅者，仅需发布事件。
- **实现广播机制**：多个对象监听同一数据源，自动更新状态。
- **多层级触发**：事件链式触发，适用于跨模块或跨系统的消息传递。

观察者模式的角色：

| 角色                                | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| **抽象主题（Subject）**             | **多层级触发**：事件链式触发，适用于跨模块或跨系统的消息传递。 |
| **具体主题（Concrete Subject）**    | 当状态改变时，**通知所有观察者**。                           |
| **抽象观察者（Observer）**          | 定义**更新方法**，用于接收通知。                             |
| **具体观察者（Concrete Observer）** | 实现更新方法，以便在**接收通知时更新自身状态**。             |

![img](https://i-blog.csdnimg.cn/blog_migrate/9431bf9a0764bf4d1cb697386e43ab6d.gif)



## 2、观察者模式实现

### 1、抽象主题类

- Subject

```java
public class Subject {
    private boolean changed = false;
    private List<Observer> observers = new ArrayList<>();

    // 添加一个观察者
    public void addObserver(Observer o) {
        observers.add(o);
    }

    // 删除一个观察者
    public void deleteObserver(Observer o) {
        observers.remove(o);
    }

    // 通知所有观察者
    public void notifyObservers(Object arg) {
        // 如果数据没改变，那就不需要通知
        if (changed == false) {
            return;
        }
        // 通知所有的观察者来进行数据更新
        for (Observer observer : observers) {
            ((Observer) observer).update(this, arg);
        }
        // 更新完成以后，清除当前主题状态
        this.clearChanged();
    }

    public void setChanged() {
        changed = true;
    }

    public void clearChanged() {
        changed = false;
    }
}
```



### 2、具体主题类

- Weather

```java
public class Weather {
    private float temperature;  // 温度
    private float humidity;     // 湿度
    private float pressure;     // 压力

    public Weather() {}

    public Weather(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
    }

    public float getTemperature() {
        return temperature;
    }

    public void setTemperature(float temperature) {
        this.temperature = temperature;
    }

    public float getHumidity() {
        return humidity;
    }

    public void setHumidity(float humidity) {
        this.humidity = humidity;
    }

    public float getPressure() {
        return pressure;
    }

    public void setPressure(float pressure) {
        this.pressure = pressure;
    }
}
```

- WeatherSubject

```java
public class WeatherSubject extends Subject {
    private Weather weather;

    public Weather getWeather() {
        return weather;
    }

    public void setWeather(Weather weather) {
        this.weather = weather;
        setChanged();
        notifyObservers(weather);
    }
}
```



### 3、抽象观察者

- Observer

```java
public interface Observer {
    void update(Subject s, Object arg);
}
```



### 4、具体观察者

- WebSite1

```java
public class WebSite1 implements Observer {
    private Weather weather;

    public void display() {
        System.out.println("***** WebSite1 *****");
        System.out.println("温度：" + weather.getTemperature());
        System.out.println("湿度：" + weather.getHumidity());
        System.out.println("压力：" + weather.getPressure());
        System.out.println("********************");
    }

    @Override
    public void update(Subject s, Object arg) {
        this.weather = (Weather) arg;
        display();
    }
}
```

- WebSite2

```java
public class WebSite2 implements Observer {
    private Weather weather;

    public void display() {
        System.out.println("***** WebSite2 *****");
        System.out.println("温度：" + weather.getTemperature());
        System.out.println("湿度：" + weather.getHumidity());
        System.out.println("压力：" + weather.getPressure());
        System.out.println("********************");
    }

    @Override
    public void update(Subject s, Object arg) {
        this.weather = (Weather) arg;
        display();
    }
}
```



### 5、最终测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        WeatherSubject ws = new WeatherSubject();
        ws.addObserver(new WebSite1());
        ws.addObserver(new WebSite2());
        ws.setWeather(new Weather(10f, 100f, 30.3f));
    }
}
```

```
***** WebSite1 *****
温度：10.0
湿度：100.0
压力：30.3
********************
***** WebSite2 *****
温度：10.0
湿度：100.0
压力：30.3
********************
```



## 3、观察者模式应用

在 Java 中，通过 java.util.Observable 类和 java.util.Observer 接口定义了观察者模式，只要实现它们就可以编写观察者模式实例。

### 1、具体主题类

- Weather

```java
public class Weather {
    private float temperature;  //温度
    private float humidity;     //湿度
    private float pressure;     //压力

    public Weather() {}

    public Weather(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
    }

    public float getTemperature() {
        return temperature;
    }

    public void setTemperature(float temperature) {
        this.temperature = temperature;
    }

    public float getHumidity() {
        return humidity;
    }

    public void setHumidity(float humidity) {
        this.humidity = humidity;
    }

    public float getPressure() {
        return pressure;
    }

    public void setPressure(float pressure) {
        this.pressure = pressure;
    }
}
```

- WeatherObservable

```java
import java.util.Observable;

public class WeatherObservable extends Observable {
    private Weather weather;

    public Weather getWeather() {
        return weather;
    }

    public void setWeather(Weather weather) {
        this.weather = weather;
        setChanged();
        notifyObservers(weather);
    }
}
```



### 2、具体观察者

- WebSite1

```java
import java.util.Observable;
import java.util.Observer;

public class WebSite1 implements Observer {
    private Weather weather;

    public void display() {
        System.out.println("***** WebSite1 *****");
        System.out.println("温度：" + weather.getTemperature());
        System.out.println("湿度：" + weather.getHumidity());
        System.out.println("压力：" + weather.getPressure());
        System.out.println("********************");
    }

    @Override
    public void update(Observable o, Object arg) {
        this.weather = (Weather) arg;
        display();
    }
}
```

- WebSite2

```java
import java.util.Observable;
import java.util.Observer;

public class WebSite2 implements Observer {
    private Weather weather;

    public void display() {
        System.out.println("***** WebSite2 *****");
        System.out.println("温度：" + weather.getTemperature());
        System.out.println("湿度：" + weather.getHumidity());
        System.out.println("压力：" + weather.getPressure());
        System.out.println("********************");
    }

    @Override
    public void update(Observable o, Object arg) {
        this.weather = (Weather) arg;
        display();
    }
}
```



### 3、最终测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        WeatherObservable wo = new WeatherObservable();
        wo.addObserver(new WebSite1());
        wo.addObserver(new WebSite2());
        wo.setWeather(new Weather(10f, 100f, 30.3f));
    }
}
```

```
***** WebSite2 *****
温度：10.0
湿度：100.0
压力：30.3
********************
***** WebSite1 *****
温度：10.0
湿度：100.0
压力：30.3
********************
```



# 行为型：中介者模式

## 1、中介者模式介绍

中介者模式的介绍：

- 在现实生活中，多个对象之间可能存在复杂的交互关系，如飞机管制、聊天室、工作流管理等。这种交互关系如果是“网状结构”，每个对象都必须知道它需要交互的所有对象，导致耦合度高，维护困难。
- 中介者模式（Mediator Pattern）是用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。中介者模式属于行为型模式。
- 传统模式：每个对象都需要维护一组交互对象的引用，类似于每个人都记住所有朋友的电话号码。
- 中介者模式：引入一个中介对象，所有的对象都通过中介者进行通信，就像使用通讯录来管理联系人。

中介者模式的优点：

- 类之间各司其职，符合迪米特法则。

- 降低了对象之间的耦合性，使得对象易于独立地被复用。
- 将对象间的一对多关联转变为一对一的关联，提高系统的灵活性，使得系统易于维护和扩展。

中介者模式的缺点：

- 中介者可能过于复杂，承担大量协调工作，可能成为系统的性能瓶颈。当同事类越多时，中介者就会越臃肿，变得复杂且难以维护。
- 可读性下降，由于所有交互都经过中介者，不如直接调用清晰。且中介者承担了较多责任，一旦中介者出错，整个系统会受到影响。

中介者模式的场景：

- 复杂对象交互：对象之间存在复杂的网状关系，需要解耦。
- 事件驱动系统：如GUI组件、消息队列等。
- 多个子系统交互：如 MVC 架构中的 Controller 作为中介者管理 Model 和 View。

中介者模式的角色：

| 角色                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| **抽象中介者（Mediator）**           | 它是中介者的接口，提供了同事对象注册与转发同事对象信息的抽象方法。 |
| **具体中介者（Concrete Mediator）**  | 实现中介者接口，定义一个 List 来管理同事对象，协调各个同事角色之间的交互关系，因此它依赖于同事角色。 |
| **抽象同事类（Colleague）**          | 定义同事类的接口，保存中介者对象，提供同事对象交互的抽象方法，实现所有相互影响的同事类的公共功能。 |
| **具体同事类（Concrete Colleague）** | 是抽象同事类的实现者，当需要与其他同事对象交互时，由中介者对象负责后续的交互。 |

![img](https://i-blog.csdnimg.cn/blog_migrate/afc8ffa89e573e0a513f0f8a9cdb4a60.gif)



## 2、中介者模式实现

### 1、抽象中介者「Mediator」

```java
public abstract class Mediator {
    public abstract void register(Colleague colleague);
    public abstract void relay(Colleague colleague); // 负责转发消息
}
```



### 2、具体中介者「ConcreteMediator」

```java
public class ConcreteMediator extends Mediator {
    private List<Colleague> colleagues = new ArrayList<>();

    public void register(Colleague colleague) {
        if (!colleagues.contains(colleague)) {
            colleagues.add(colleague);
            colleague.setMediator(this);
        }
    }

    public void relay(Colleague colleague) {
        for (Colleague c : colleagues) {
            if (!c.equals(colleague)) {
                ((Colleague) c).receive();
            }
        }
    }
}
```



### 3、抽象同事类「Colleague」

```java
public abstract class Colleague {
    protected Mediator mediator;

    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    public abstract void receive();
    public abstract void send();
}
```



### 4、具体同事类「ConcreteColleagueX」

```java
public class ConcreteColleague1 extends Colleague {
    public void receive() {
        System.out.println("具体同事类1收到请求...");
    }

    public void send() {
        System.out.println("具体同事类1发出请求...");
        mediator.relay(this); // 请中介者转发
    }
}
```

```java
public class ConcreteColleague2 extends Colleague {
    public void receive() {
        System.out.println("具体同事类2收到请求...");
    }

    public void send() {
        System.out.println("具体同事类2发出请求...");
        mediator.relay(this); // 请中介者转发
    }
}
```



### 5、最终测试类「Client」

```java
public class Client {
    public static void main(String[] args) {
        Mediator mediator = new ConcreteMediator();
        Colleague c1 = new ConcreteColleague1();
        Colleague c2 = new ConcreteColleague2();
        mediator.register(c1);
        mediator.register(c2);
        c1.send();
        c2.send();
    }
}
```

```
具体同事类1发出请求...
具体同事类2收到请求...
具体同事类2发出请求...
具体同事类1收到请求...
```



## 3、中介者模式应用

打开 JDK 中的 Timer 类，查看 Timer 类的结构，会发现 Timer 类中有很多 schedule() 重载方法，如下图所示：

![img](https://i-blog.csdnimg.cn/blog_migrate/90af699ee4515cabbc2eb43ed7ecbf34.png)

任意点开其中一个方法，会发现所有方法最终都调用了私有的 sched() 方法，源码如下：

```java
public void schedule(TimerTask task, long delay) {
    if (delay < 0)
        throw new IllegalArgumentException("Negative delay.");
    sched(task, System.currentTimeMillis()+delay, 0);
}
```

不论是什么样的任务加入到队列中，我们都把这个队列中的对象称为“同事”。查看 sched() 方法的源码可以看出，所有的任务（task）都放到了 Timer 类维护的 task 队列中，同事之间的通信都是通过 Timer 来协调完成的，所以，Timer 承担了中介者的角色，而 task 队列内的任务就是具体同事对象。

```java
private void sched(TimerTask task, long time, long period) {
    if (time < 0)
        throw new IllegalArgumentException("Illegal execution time.");

    // Constrain value of period sufficiently to prevent numeric
    // overflow while still being effectively infinitely large.
    if (Math.abs(period) > (Long.MAX_VALUE >> 1))
        period >>= 1;

    synchronized(queue) {
        if (!thread.newTasksMayBeScheduled)
            throw new IllegalStateException("Timer already cancelled.");

        synchronized(task.lock) {
            if (task.state != TimerTask.VIRGIN)
                throw new IllegalStateException(
                "Task already scheduled or cancelled");
            task.nextExecutionTime = time;
            task.period = period;
            task.state = TimerTask.SCHEDULED;
        }

        queue.add(task);
        if (queue.getMin() == task)
            queue.notify();
    }
}
```



# 行为型：迭代器模式

## 1、迭代器模式介绍

迭代器模式的介绍：

- 在现实生活以及程序设计中，经常要访问一个聚合对象中的各个元素，如“数据结构”中的链表遍历，通常的做法是将链表的创建和遍历都放在同一个类中，但这种方式不利于程序的扩展，如果要更换遍历方法就必须修改程序源代码，这违背了“开闭原则”。

- 迭代器模式（Iterator Pattern）是 Java 和 .Net 编程环境中非常常用的设计模式。这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。迭代器模式属于行为型模式。


迭代器模式的优点：

- 访问一个聚合对象的内容而无须暴露它的内部表示。

- 遍历任务交由迭代器完成，这简化了聚合类。
- 它支持以不同方式遍历一个聚合，甚至可以自定义迭代器的子类以支持新的遍历。
- 增加新的聚合类和迭代器类都很方便，无须修改原有代码。
- 封装性良好，为遍历不同的聚合结构提供一个统一的接口。

迭代器模式的缺点：

- 增加了类的个数，这在一定程度上增加了系统的复杂性。


迭代器模式的场景：

- 当需要为聚合对象提供多种遍历方式时。

- 当需要为遍历不同的聚合结构提供一个统一的接口时。
- 当访问一个聚合对象的内容而无须暴露其内部细节的表示时。

迭代器模式的角色：

| 角色                                | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| **抽象聚合（Aggregate）**           | 定义存储、添加、删除聚合对象以及创建迭代器对象的接口。       |
| **具体聚合（Concrete Aggregate）**  | 实现抽象聚合类，返回一个具体迭代器的实例。                   |
| **抽象迭代器（Iterator）**          | 定义访问和遍历聚合元素的接口，通常包含 hasNext()、first()、next() 等方法。 |
| **具体迭代器（Concrete lterator）** | 实现抽象迭代器接口中所定义的方法，完成对聚合对象的遍历，记录遍历的当前位置。 |

![img](https://i-blog.csdnimg.cn/blog_migrate/cd82155e8461bd53b917f2379877c277.gif)



## 2、迭代器模式实现

### 1、抽象迭代器「Iterator」

```java
public interface Iterator {
    Object first();
    Object next();
    boolean hasNext();
}
```



### 2、具体迭代器「ConcreteIterator」

```java
public class ConcreteIterator implements Iterator {
    private List<Object> list;
    private int index = -1;

    public ConcreteIterator(List<Object> list) {
        this.list = list;
    }

    @Override
    public boolean hasNext() {
        if (index < list.size() - 1) {
            return true;
        } else {
            return false;
        }
    }

    @Override
    public Object first() {
        index = 0;
        Object obj = list.get(index);
        return obj;
    }

    @Override
    public Object next() {
        Object obj = null;
        if (this.hasNext()) {
            obj = list.get(++index);
        }
        return obj;
    }
}
```



### 3、抽象聚合类「Aggregate」

```java
public interface Aggregate {
    void add(Object obj);
    void remove(Object obj);
    Iterator getIterator();
}
```



### 4、具体聚合类「ConcreteAggregate」

```java
public class ConcreteAggregate implements Aggregate {
    private List<Object> list = new ArrayList<>();

    public void add(Object obj) {
        list.add(obj);
    }

    public void remove(Object obj) {
        list.remove(obj);
    }

    public Iterator getIterator() {
        return new ConcreteIterator(list);
    }
}
```



### 5、最终测试类「Client」

```java
public class Client {
    public static void main(String[] args) {
        Aggregate ag = new ConcreteAggregate();
        ag.add("张三");
        ag.add("李四");
        ag.add("王五");
        Iterator it = ag.getIterator();
        while (it.hasNext()) {
            Object object = it.next();
            System.out.println(object.toString());
        }
    }
}
```

```
张三
李四
王五
```



## 3、迭代器模式应用

JDK 的 ArrayList 集合中就使用了迭代器模式。

```java
public class ArrayListTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张三");
        list.add("李四");
        list.add("王五");
        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            Object object = it.next();
            System.out.println(object);
        }
    }
}
```

```
张三
李四
王五
```



# 行为型：访问者模式

## 1、访问者模式介绍

访问者模式的介绍：

- 在现实生活中，有些集合对象存在多种不同的元素，且每种元素也存在多种不同的访问者和处理方式。例如，公园中存在多个景点，也存在多个游客，不同的游客对同一个景点的评价可能不同。

- 访问者模式（Visitor Pattern）尽可能地将数据结构与数据操作分离，我们使用一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变。这种类型的设计模式属于行为型模式。


访问者模式的优点：

- 扩展性好，可以在不修改元素类的情况下，为元素类添加新的功能。

- 复用性好，通过访问者定义通用功能，提高系统复用程度。
- 灵活性好，将数据结构与操作解耻，让操作集合可相对自由地发展而不影响数据结构。
- 符合单一责任原则，访问者将相关行为封装在一起，每个访问者功能比较单一。

访问者模式的缺点：

- 增加新元素类困难，需要在每个访问者中添加相应操作，违背“开闭原则”。

- 破坏封装，元素类对访问者公开细节，不便于隐藏内部实现。
- 违反了依赖倒置原则。访问者模式依赖了具体类，而没有依赖抽象类。

访问者模式的场景：

- 对象结构相对稳定，但操作算法常变的情况。
- 对象结构中的对象需要提供多种不同且不相关的操作，而且要避免让这些操作的变化影响对象的结构。
- 对象结构包含很多类型的对象，希望对这些对象实施一些依赖于其具体类型的操作。

访问者模式的角色：

| 角色                                  | 说明                                                 |
| ------------------------------------- | ---------------------------------------------------- |
| **抽象访问者角色（Visitor）**         | 定义 visit() 方法，每个元素类对应一个 visit() 操作。 |
| **具体访问者角色（ConcreteVisitor）** | 实现抽象访问者定义的操作。                           |
| **抽象元素角色（Element）**           | 定义 accept() 方法，接受访问者。                     |
| **具体元素角色（ConcreteElement）**   | 实现 accept() 方法，主要转迁到 visit(this) 。        |
| **对象结构角色（Object Structure）**  | 包含元素类，并提供访问者遍历所有元素的方法。         |

![img](https://i-blog.csdnimg.cn/blog_migrate/9ae8d7d518f502857b2176553d2ba732.gif)



## 2、访问者模式实现

### 1、抽象访问者类

- ComputerPartVisitor

```java
public interface ComputerPartVisitor {
    void visit(Mouse mouse);
    void visit(Keyboard keyboard);
}
```



### 2、具体访问者类

- MIComputerPartVisitor

```java
public class MIComputerPartVisitor implements ComputerPartVisitor {
    @Override
    public void visit(Mouse mouse) {
        System.out.println("Displaying XiaoMi Mouse ...");
    }

    @Override
    public void visit(Keyboard keyboard) {
        System.out.println("Displaying XiaoMi Keyboard ...");
    }
}
```

- HPComputerPartVisitor

```java
public class HPComputerPartVisitor implements ComputerPartVisitor {
    @Override
    public void visit(Mouse mouse) {
        System.out.println("Displaying HuiPu Mouse ...");
    }

    @Override
    public void visit(Keyboard keyboard) {
        System.out.println("Displaying HuiPu Keyboard ...");
    }
}
```



### 3、抽象元素类

- ComputerPart

```java
public interface ComputerPart {
    void accept(ComputerPartVisitor computerPartVisitor);
}
```



### 4、具体元素类

- Mouse

```java
public class Mouse implements ComputerPart {
    @Override
    public void accept(ComputerPartVisitor computerPartVisitor) {
        computerPartVisitor.visit(this);
    }
}
```

- Keyboard

```java
public class Keyboard implements ComputerPart {
    @Override
    public void accept(ComputerPartVisitor computerPartVisitor) {
        computerPartVisitor.visit(this);
    }
}
```



### 5、对象结构类

- ObjectStructure

```java
public class ObjectStructure {
    private List<ComputerPart> computerParts = new ArrayList<>();

    public void attach(ComputerPart computerPart) {
        computerParts.add(computerPart);
    }

    public void detach(ComputerPart computerPart) {
        computerParts.remove(computerPart);
    }

    public void display(ComputerPartVisitor computerPartVisitor) {
        for (ComputerPart computerPart : computerParts) {
            computerPart.accept(computerPartVisitor);
        }
    }
}
```



### 6、最终测试类

- Client

```java
public class Client {
    public static void main(String[] args) {
        ObjectStructure os = new ObjectStructure();
        os.attach(new Mouse());
        os.attach(new Keyboard());
        os.display(new MIComputerPartVisitor());
        os.display(new HPComputerPartVisitor());
    }
}
```

```
Displaying XiaoMi Mouse ...
Displaying XiaoMi Keyboard ...
Displaying HuiPu Mouse ...
Displaying HuiPu Keyboard ...
```



## 3、访问者模式应用

在早期的 Java 版本中，如果要对指定目录下的文件进行遍历，必须用递归的方式来实现，这种方法复杂且灵活性不高。

Java 7 版本后，Files 类提供了 walkFileTree() 方法，该方法可以很容易的对目录下的所有文件进行遍历，需要 Path、FileVisitor 两个参数。其中，Path 是要遍历文件的路径，FileVisitor 则可以看成一个文件访问器。源码如下：

```java
public final class Files {
    // 以上代码部分省略...
    
    public static Path walkFileTree(Path start, FileVisitor<? super Path> visitor)
        throws IOException
    {
        return walkFileTree(start,
                            EnumSet.noneOf(FileVisitOption.class),
                            Integer.MAX_VALUE,
                            visitor);
    }
    
    // 以下代码部分省略...
}
```

FileVisitor 提供了递归遍历文件树的支持，这个接口的方法表示了遍历过程中的关键过程，允许在文件被访问、目录将被访问、目录已被访问、发生错误等过程中进行控制。换句话说，这个接口在文件被访问前、访问中和访问后，以及产生错误的时候都有相应的钩子程序进行处理。FileVisitor 主要提供了 4 个方法，且返回结果的都是 FileVisitResult 对象值，用于决定当前操作完成后接下来该如何处理。FileVisitResult 是一个枚举类，代表返回之后的一些后续操作。源码如下：

```java
public interface FileVisitor<T> {
    FileVisitResult preVisitDirectory(T dir, BasicFileAttributes attrs) throws IOException;
    FileVisitResult visitFile(T file, BasicFileAttributes attrs) throws IOException;
    FileVisitResult visitFileFailed(T file, IOException exc) throws IOException;
    FileVisitResult postVisitDirectory(T dir, IOException exc) throws IOException;
}
```

FileVisitResult 主要包含 4 个常见的操作：

- FileVisitResult.CONTINUE：这个访问结果表示当前的遍历过程将会继续。

- FileVisitResult.SKIP_SIBLINGS：这个访问结果表示当前的遍历过程将会继续，但是要忽略当前文件/目录的兄弟节点。
- FileVisitResult.SKIP_SUBTREE：这个访问结果表示当前的遍历过程将会继续，但是要忽略当前目录下的所有节点。
- FileVisitResult.TERMINATE：这个访问结果表示当前的遍历过程将会停止。

```java
public enum FileVisitResult {
    CONTINUE,
    TERMINATE,
    SKIP_SUBTREE,
    SKIP_SIBLINGS;
}
```

通过访问者去遍历文件树会比较方便，比如查找文件夹内符合某个条件的文件或者某一天内所创建的文件，这个类中都提供了相对应的方法。它的实现也非常简单，代码如下：

```java
public class SimpleFileVisitor<T> implements FileVisitor<T> {
    protected SimpleFileVisitor() {}

    @Override
    public FileVisitResult preVisitDirectory(T dir, BasicFileAttributes attrs)
        throws IOException
    {
        Objects.requireNonNull(dir);
        Objects.requireNonNull(attrs);
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult visitFile(T file, BasicFileAttributes attrs)
        throws IOException
    {
        Objects.requireNonNull(file);
        Objects.requireNonNull(attrs);
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult visitFileFailed(T file, IOException exc)
        throws IOException
    {
        Objects.requireNonNull(file);
        throw exc;
    }

    @Override
    public FileVisitResult postVisitDirectory(T dir, IOException exc)
        throws IOException
    {
        Objects.requireNonNull(dir);
        if (exc != null)
            throw exc;
        return FileVisitResult.CONTINUE;
    }
}
```



# 行为型：备忘录模式

## 1、备忘录模式介绍

备忘录模式的介绍：

- 人们在日常生活中可能会犯错误，希望能够“后悔”或“撤销”某些操作。在计算机系统中，我们也经常遇到类似的需求，如文本编辑器的 撤销（Undo）、数据库事务的 回滚（Rollback）、游戏的 存档与恢复 等。
- 备忘录模式（Memento Pattern） 提供了一种机制，可以在不破坏对象封装性的前提下，**保存和恢复对象的内部状态**。
- 这种模式属于 行为型模式，主要用于保存对象的历史状态，并在需要时进行恢复。

备忘录模式的优点：

- 状态回滚：提供了一种可以恢复状态的机制，使得对象能够恢复到某个历史状态。
- 封装性：除了创建备忘录的 发起人（Originator） 之外，其他对象无法访问备忘录内部状态。
- 简化发起人：发起人只需创建备忘录，管理状态交由 管理者（Caretaker） 处理。

备忘录模式的缺点：

- 资源消耗大：如果状态对象很大或变化频繁，存储多个备忘录可能会占用大量内存。
- 管理复杂：需要额外的 管理者（Caretaker） 来管理备忘录，可能导致系统复杂度增加。

备忘录模式的场景：

- 游戏存档：保存游戏进度，用户可以随时恢复。
- 文本编辑器：按 Ctrl+Z 可撤销上一步操作。
- 数据库事务：支持回滚到特定的事务状态。
- 网页导航：浏览器的 后退/前进 功能。

备忘录模式的角色：

| 角色                     | 职责                                                |      |
| ------------------------ | --------------------------------------------------- | ---- |
| **发起人（Originator）** | 记录当前状态，创建/恢复备忘录                       |      |
| **备忘录（Memento）**    | 存储发起人的状态，提供回溯数据                      |      |
| **管理者（Caretaker）**  | 负责管理多个备忘录（存储 & 取回），但不能修改其内容 |      |

![img](https://i-blog.csdnimg.cn/blog_migrate/5f4ddeb145b2460aa546152f36173639.gif)



## 2、备忘录模式实现

### 1、发起人「Originator」

```java
// 发起人：拥有要存储的状态，并创建/恢复备忘录
public class Originator {
    private String state;

    public void setState(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }

    // 创建备忘录，存储当前状态
    public Memento createMemento() {
        return new Memento(state);
    }

    // 从备忘录恢复状态
    public void restoreMemento(Memento memento) {
        this.state = memento.getState();
    }
}
```



### 2、备忘录「Memento」

```java
// 备忘录：存储 Originator 的状态
public class Memento {
    private final String state; // 只读，不允许外部修改

    public Memento(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }
}
```



### 3、管理者「Caretaker」

```java
import java.util.ArrayList;
import java.util.List;

// 管理者：负责存储和恢复备忘录
public class Caretaker {
    private final List<Memento> history = new ArrayList<>();

    public void addMemento(Memento memento) {
        history.add(memento);
    }

    public Memento getMemento(int index) {
        return history.get(index);
    }
}
```



### 4、测试类「Client」

```java
public class Client {
    public static void main(String[] args) {
        Originator originator = new Originator();
        Caretaker caretaker = new Caretaker();

        // 设置初始状态并存档
        originator.setState("状态 #1");
        caretaker.addMemento(originator.createMemento());

        // 修改状态并存档
        originator.setState("状态 #2");
        caretaker.addMemento(originator.createMemento());

        // 继续修改状态
        originator.setState("状态 #3");
        System.out.println("当前状态：" + originator.getState()); // 输出: 状态 #3

        // 恢复到状态 #1
        originator.restoreMemento(caretaker.getMemento(0));
        System.out.println("恢复后状态：" + originator.getState()); // 输出: 状态 #1
    }
}
```

```
当前状态：状态 #3
恢复后状态：状态 #1
```



## 3、备忘录模式应用

由于 JDK、Spring、MyBatis 中 很少直接 使用备忘录模式，因此不做源码分析。但在 文本编辑器、游戏、数据库事务、工作流引擎 等场景中，该模式应用广泛。

**如果你的应用需要回溯历史状态，备忘录模式是一个不错的选择！** 



# 行为型：解释器模式

## 1、解释器模式介绍

什么是解释器模式？

- 解释器模式（Interpreter Pattern） 主要用于解析和执行特定领域的语言或表达式。它定义了一种语法解析机制，并提供一个解释器来处理该语法。

解释器模式的结构「解释器模式通常包含以下组件」

| 角色                                    | **说明**                                       |
| --------------------------------------- | ---------------------------------------------- |
| 抽象表达式（Expression）                | 定义解释器的通用接口                           |
| 终结符表达式（TerminalExpression）      | 处理最基本的操作（如数字）                     |
| 非终结符表达式（NonTerminalExpression） | 组合终结符表达式，表示复杂规则（如加法、乘法） |
| 上下文（Context）                       | 存储全局信息，如变量表                         |
| 客户端（Client）                        | 负责构建语法树，并调用解释器执行               |

解释器模式的优点：

- 可扩展性强：新增运算符（如 *、/）时，只需添加新的 Expression 类，无需修改原代码。
- 符合开闭原则：可以轻松扩展新的解释规则。
- 易于实现 DSL（领域特定语言）：适用于规则引擎、数学计算等。

解释器模式的缺点：

- 性能低：递归解析表达式，效率比编译型方案（如 ANTLR）差。
- 难以维护：当语法复杂时，表达式类会大量增加，难以维护。
- 不适合复杂计算：更适合简单规则解析，而非复杂的 SQL 解析。

适用场景：

- 需要解析和执行特定的语言、表达式或规则（如数学表达式、SQL 解析、规则引擎）。
- 语法相对稳定，但需要灵活扩展解析规则。
- 递归结构的表达式计算（如加法、减法、乘法、除法）。



## 2、解释器模式实现

解释器模式示例：数学表达式计算器

需求背景：实现一个简单的数学表达式解析器，支持 数字 + 数字 和 数字 - 数字 运算，例如：

```
"5 + 3 - 2"  ->  结果: 6
```



### 1、定义抽象表达式接口

```java
// 抽象表达式：定义解释器接口
interface Expression {
    int interpret();
}
```



### 2、终结符表达式「数字」

```java
// 终结符表达式（具体的数字）
class NumberExpression implements Expression {
    private int number;

    public NumberExpression(int number) {
        this.number = number;
    }

    @Override
    public int interpret() {
        return number;
    }
}
```

- NumberExpression 表示一个整数（如 5、3）。
- interpret() 方法直接返回该数值。



### 3、非终结符表达式「加减法」

```java
// 非终结符表达式（加法）
class AddExpression implements Expression {
    private Expression left, right;

    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}

// 非终结符表达式（减法）
class SubtractExpression implements Expression {
    private Expression left, right;

    public SubtractExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() - right.interpret();
    }
}
```

- AddExpression 计算 left + right。
- SubtractExpression 计算 left - right。
- interpret() 方法递归调用左右子表达式。



### 4、解析并执行表达式

```java
import java.util.Stack;

public class InterpreterPatternDemo {
    public static void main(String[] args) {
        String expression = "5 + 3 - 2";  // 输入表达式
        Expression result = parseExpression(expression);
        System.out.println(expression + " = " + result.interpret());  // 输出: 6
    }

    // 解析输入字符串
    public static Expression parseExpression(String input) {
        Stack<Expression> stack = new Stack<>();
        String[] tokens = input.split(" ");

        for (int i = 0; i < tokens.length; i++) {
            if (tokens[i].equals("+")) {
                Expression left = stack.pop();
                Expression right = new NumberExpression(Integer.parseInt(tokens[++i]));
                stack.push(new AddExpression(left, right));
            } else if (tokens[i].equals("-")) {
                Expression left = stack.pop();
                Expression right = new NumberExpression(Integer.parseInt(tokens[++i]));
                stack.push(new SubtractExpression(left, right));
            } else {
                stack.push(new NumberExpression(Integer.parseInt(tokens[i])));
            }
        }
        return stack.pop();
    }
}
```

```
5 + 3 - 2 = 6
```





## 3、解释器模式实现

尽管 JDK、Spring、MyBatis 很少直接使用解释器模式，但它仍然适用于一些场景：

| **应用场景**   | **示例**                |
| -------------- | ----------------------- |
| 数学表达式解析 | 计算器（如 5 + 3 * 2）  |
| SQL 解析       | MyBatis 动态 SQL 解析   |
| 正则表达式     | java.util.regex.Pattern |
| EL 表达式      | Spring SpEL、OGNL       |
| 规则引擎       | Drools 规则解析         |

解释器模式使用总结：

- 解释器模式适用于解析和执行特定领域的语言、表达式或规则。
- 适合 DSL（领域特定语言）、规则引擎、表达式计算，但在实际开发中更常用**现成的解析器（如 ANTLR、SpEL）**。
- 如果规则复杂，建议使用编译方式（如 AST、字节码优化）代替解释器模式，以提高性能。
- 你可以将解释器模式用于业务规则计算、动态表达式解析等场景，但在实际开发中，通常更推荐使用现成的解析工具（如 Spring SpEL、OGNL、ANTLR）。 

# xx、参考文献 & 鸣谢

1. 《23种设计模式系列教程》｜緑水長流*z：https://coder1024.blog.csdn.net/article/details/127095961
2. 设计模式 | 轻松的小希：https://blog.csdn.net/qq_38490457/category_10837977.html
3. 设计模式目录 | 二木成林：https://blog.csdn.net/cnds123321/article/details/119045245
4. 字详解 GoF 23 种设计模式（多图、思维导图、模式对比），让你一文全面理解：https://blog.csdn.net/penriver/article/details/118571991
5. 小威要向诸佬学习呀：https://blog.csdn.net/qq_53847859/article/details/130594428
