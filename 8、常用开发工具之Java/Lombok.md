## Lombok 介绍与安装

> 参考：
>
> 1. Lombok简化编码使用方法：https://blog.csdn.net/qq_38288606/article/details/80690827
> 2. Lombok常用注解：https://www.cnblogs.com/ooo0/p/12448096.html
> 3. Lombok的@Builder注解带来的两大坑：https://blog.csdn.net/belongtocode/article/details/109583053
> 3. 详解 Lombok 中的 @Builder 用法：https://mp.weixin.qq.com/s/Nwt8iwBrDjqMfdT79vsCjw

官网：https://projectlombok.org/

1、Lombok介绍：

Lombok是一个优秀的Java代码库，简化了Java的编码，为Java代码的精简提供了一种方式。每个JavaBean都会写getter、setter、equals、hashCode、toString等代码，Lombok消除Java的冗长代码，尤其是对于简单的Java对象，只要加上注解就行



2、Lombok的使用

pom.xml 添加依赖进行版本管理

```xml
<!--锁定 jdk 版本为 1.8-->
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
<!--锁定 jdk 版本为 1.8,选择其一即可-->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.7.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
<!--注意： 如果 properties 和 build 里面都有配置的话，那么 properties 会覆盖掉 build 里面的配置，即以 properties 里面的配置为准。-->

<dependencies>
    <!--https://mvnrepository.com/artifact/org.projectlombok/lombok/1.18.16-->
    <!--scope=provided，说明它只在编译阶段生效，不需要打入包中, Lombok在编译期将带Lombok注解的Java文件正确编译为完整的Class文件-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.16</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

添加IDE工具对Lombok的支持：点击File-- 》Settings界面，安装Lombok插件，IDEA里需要在设置中启用 annotation processors，然后重启IDEA



## Lombok 常用注解介绍

1. @Data：在JavaBean中使用，注解包含包含getter、setter、NoArgsConstructor注解
2. @Value注解和@Data类似，区别在于它会把所有成员变量默认定义为private final修饰，并且不会生成set方法
3. @getter和@setter：在JavaBean中使用，注解会生成对应的getter和setter方法
4. @NoArgsConstructor：在JavaBean中使用，注解会生成对应的无参构造方法
5. @AllArgsConstructor：在JavaBean中使用，注解会生成对应的有参构造方法
6. @RequiredArgsConstructor ：生成private构造方法，使用staticName选项生成指定名称的static方法。
7. @ToString：在JavaBean中使用，注解会自动重写对应的toStirng方法
8. @ToString(exclude={"column1","column2"})：排除多个column列所对应的元素
9. @ToString(of={"column1","column2"})：只生成包含多个column列所对应的元素
10. @EqualsAndHashCode：在JavaBean中使用，注解会自动重写对应的equals方法和hashCode方法
11. @Slf4j和@Log4j：在需要打印日志的类中使用，项目中使用slf4j和log4j日志框架
12. @NonNull：注解快速判断是否为空,为空抛出java.lang.NullPointerException
13. @Accessors(chain = true)：链式风格，在调用set方法时，返回这个类的实例对象
14. @Synchronized：注解自动添加到同步机制，生成的代码并不是直接锁方法,而是锁代码块， 作用范围是方法上
15. @Cleanup：注解用于确保已分配的资源被释放（IO的连接关闭）





## @Setter & @Getter

注解详解：

1. 作用类上，生成所有成员变量的getter/setter方法
2. 作用于成员变量上，生成该成员变量的getter/setter方法
3. 方法控制访问级别 set和get注解加：@Setter/@Getter(AccessLevel.PROTECTED)
4. 忽略不生成某个字段set/get方法：@Setter/@Getter(AccessLevel.NONE)
5. final成员变量只会生成get
6. static静态成员变量不会生成set/get方法



新建实体类：User.java

```java
@Setter
@Getter
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;

    public User() {
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Long getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }
}
```



更多属性示例：

新建实体类：User.java

```java
@Setter
@Getter
public class User {
    private Long id;

    /**
     * 不想生成 get方法
     */
    @Getter(AccessLevel.NONE)
    private Integer age;

    /**
     * 控制访问权限
     */
    @Getter(AccessLevel.PROTECTED)
    private int salary;

    /**
     * final 只会生成get
     */
    private final String name="Sam";

    /**
     * 下面两个静态成员变量不会生成set/get方法
     */
    static Date createTime = new Date();
    private static final String address = "广东省广州市";
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private Integer age;
    private int salary;
    private final String name = "Sam";
    static Date createTime = new Date();
    private static final String address = "广东省广州市";

    public User() {
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setSalary(int salary) {
        this.salary = salary;
    }

    public Long getId() {
        return this.id;
    }

    public String getName() {
        this.getClass();
        return "Sam";
    }

    protected int getSalary() {
        return this.salary;
    }
}
```





## @XxxArgsConstructor

> @NoArgsConstructor、@AllArgsConstructor、@RequiredArgsConstructor

@NoArgsConstructor：生成对应的无参构造方法（一般情况下不会单独使用该注解，因为默认就有无参构造）

```java
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;

    public User() {
    }
}
```



@AllArgsConstructor：生成对应的有参构造方法（全部参数）

```java
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

**注意：**一般@AllArgsConstructor和@NoArgsConstructor一起使用，因为使用@AllArgsConstructor后默认就不会生成无参构造了



@RequiredArgsConstructor：生成一个包含 "特定参数" 的构造器，特定参数指的是：

1. 加上 final 修饰词的变量们
2. 加上 @NonNull 注解的变量们
3. 注意：不能加上 @NoArgsConstructor

```java
@RequiredArgsConstructor
public class User {
    @NonNull
    private Long id;
    private final String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    @NonNull
    private Long id;
    private final String name;

    public User(@NonNull Long id, String name) {
        if (id == null) {
            throw new NullPointerException("id is marked non-null but is null");
        } else {
            this.id = id;
            this.name = name;
        }
    }
}
```



## @NonNull

@NonNull：Lombok非空判断、作用于方法上或者属性，用于非空判断，如果为空则抛异常

```java
@Setter
@Getter
public class User {
    @NonNull
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
import lombok.NonNull;

public class User {
    @NonNull
    private Long id;
    private String name;

    public User() {
    }

    public void setId(@NonNull Long id) {
        if (id == null) {
            throw new NullPointerException("id is marked non-null but is null");
        } else {
            this.id = id;
        }
    }

    public void setName(String name) {
        this.name = name;
    }

    @NonNull
    public Long getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }
}
```



## @ToString

@ToString：注解会自动重写对应的toStirng方法

@ToString(exclude={"column1","column2"})：排除多个column列所对应的元素

@ToString(of={"column1","column2"})：只生成包含多个column列所对应的元素

```java
@ToString
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;

    public User() {
    }

    public String toString() {
        return "User(id=" + this.id + ", name=" + this.name + ")";
    }
}
```



## @EqualsAndHashCode

@EqualsAndHashCode：作用于类，覆盖默认的equals和hashCode, 作用于全部属性

@EqualsAndHashCode(exclude = {"age"})：不包括某个属性

@EqualsAndHashCode(of = {"name"})：只输出某个属性

```java
@EqualsAndHashCode
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;

    public User() {
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof User)) {
            return false;
        } else {
            User other = (User)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                Object this$id = this.id;
                Object other$id = other.id;
                if (this$id == null) {
                    if (other$id != null) {
                        return false;
                    }
                } else if (!this$id.equals(other$id)) {
                    return false;
                }

                Object this$name = this.name;
                Object other$name = other.name;
                if (this$name == null) {
                    if (other$name != null) {
                        return false;
                    }
                } else if (!this$name.equals(other$name)) {
                    return false;
                }

                return true;
            }
        }
    }

    protected boolean canEqual(Object other) {
        return other instanceof User;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.id;
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        Object $name = this.name;
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        return result;
    }
}
```





## @Data & @Value

> Lombok 太多常用的注解一个个加也麻烦，所以出现了两个聚合注解：**@Data 和 @Value**

**@Data：**整合包，只要加了 @Data 这个注解，等于同时加了以下注解

- @Getter、@Setter
- @ToString
- @EqualsAndHashCode
- @RequiredArgsConstructor

元注解：

```java
 /** 
 * @see Getter
 * @see Setter
 * @see RequiredArgsConstructor
 * @see ToString
 * @see EqualsAndHashCode
 * @see lombok.Value
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Data {
    String staticConstructor() default "";
}
```

使用案例：

```java
@Data
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;

    // @RequiredArgsConstructor
    public User() {
    }

    // @Getter、@Setter
    public Long getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    // @EqualsAndHashCode
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof User)) {
            return false;
        } else {
            User other = (User)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                Object this$id = this.getId();
                Object other$id = other.getId();
                if (this$id == null) {
                    if (other$id != null) {
                        return false;
                    }
                } else if (!this$id.equals(other$id)) {
                    return false;
                }

                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name != null) {
                        return false;
                    }
                } else if (!this$name.equals(other$name)) {
                    return false;
                }

                return true;
            }
        }
    }

    protected boolean canEqual(Object other) {
        return other instanceof User;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.getId();
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        Object $name = this.getName();
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        return result;
    }

    // @ToString
    public String toString() {
        return "User(id=" + this.getId() + ", name=" + this.getName() + ")";
    }
}
```

@Data是使用频率最高的**lombok**注解，通常@Data 会在一个值可以被更新的对象上，像是日常使用的 DTO 们、或是 JPA 里的 Entity 们，就很适合加上 @Data 注解，也就是：**@Data for mutable class**





**@Value：**也是整合包，但是他会把所有的变量都设成 final 的，其他的就跟 @Data 一样，等于同时加了以下注解：

- @Getter (**注意没有@Setter**)
- @ToString
- @EqualsAndHashCode
- @RequiredArgsConstructor

元注解：

```java
/**
 * @see lombok.Getter
 * @see lombok.experimental.FieldDefaults
 * @see lombok.AllArgsConstructor
 * @see lombok.ToString
 * @see lombok.EqualsAndHashCode
 * @see lombok.Data
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Value {
   String staticConstructor() default "";
}
```

使用案例：

```java
@Value
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public final class User {
    // 给属性都设置成final
    private final Long id;
    private final String name;

    // @RequiredArgsConstructor
    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    // @Getter
    public Long getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    // @EqualsAndHashCode
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof User)) {
            return false;
        } else {
            User other = (User)o;
            Object this$id = this.getId();
            Object other$id = other.getId();
            if (this$id == null) {
                if (other$id != null) {
                    return false;
                }
            } else if (!this$id.equals(other$id)) {
                return false;
            }

            Object this$name = this.getName();
            Object other$name = other.getName();
            if (this$name == null) {
                if (other$name != null) {
                    return false;
                }
            } else if (!this$name.equals(other$name)) {
                return false;
            }

            return true;
        }
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.getId();
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        Object $name = this.getName();
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        return result;
    }

    // @ToString
    public String toString() {
        return "User(id=" + this.getId() + ", name=" + this.getName() + ")";
    }
}
```

上面那个 @Data 适合用在 POJO 或 DTO 上，而这个 @Value 注解，则是适合加在值不希望被改变的类上，像是某个类的值当创建后就不希望被更改，只希望我们读它而已，就适合加上 @Value 注解，也就是 @Value for immutable class

另外注意一下，此 lombok 的注解 @Value 和另一个 Spring 的注解 @Value 撞名，在 import 时不要 import 错了





## @Builder（建造模式）

@Builder注解：作用在类上，自动生成流式 set 值写法，从此之后再也不用写一堆 setter 了

```java
@Builder
public class User {
    private Long id;
    private String name;
}

// 使用
User build = User.builder().id(1L).name("Sam").build();
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;

    User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    public static User.UserBuilder builder() {
        return new User.UserBuilder();
    }

    public static class UserBuilder {
        private Long id;
        private String name;

        UserBuilder() {
        }

        public User.UserBuilder id(Long id) {
            this.id = id;
            return this;
        }

        public User.UserBuilder name(String name) {
            this.name = name;
            return this;
        }

        public User build() {
            return new User(this.id, this.name);
        }

        public String toString() {
            return "User.UserBuilder(id=" + this.id + ", name=" + this.name + ")";
        }
    }
}
```

**注意：**虽然只要加上@Builder注解，我们就能够用流式写法快速设定对象的值，但是 setter 还是必须要写不能省略的，因为 Spring 或是其他框架有很多地方都会用到对象的 getter/setter 对他们取值/赋值

所以通常是@Data和@Builder 会一起用在同个类上，既方便我们流式写代码，也方便框架做事





## @Accessors

@Accessors：中文含义是存取器，@Accessors用于配置getter和setter方法的生成结果，下面介绍三个属性

**fluent**：中文含义是流畅的，设置为true，则getter和setter方法的方法名都是**基础属性名**，且setter方法返回当前对象

```java
@Data
@Accessors(fluent = true)
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;
    
    // 生成的getter和setter方法如下，方法体和其他方法略
    public Long id() {}
    public User id(Long id) {}
    public String name() {}
    public User name(String name) {}
}
```



**chain**：中文含义是链式的，设置为true，则setter方法返回当前对象。如下

```java
@Data
@Accessors(chain = true)
public class User {
    private Long id;
    private String name;
}
```

编译查看字节码：mvn compile

```java
public class User {
    private Long id;
    private String name;
    
    // 生成的setter方法如下，方法体和其他方法略
    public User setId(Long id) {}
    public User setName(String name) {}
}
```



**prefix**：中文含义是前缀，用于生成getter和setter方法的字段名会忽视指定前缀（遵守驼峰命名）

```java
@Data
@Accessors(prefix = "t")
public class User {
    private Long tId;
    private String tName;
}

@Data
@Accessors(prefix = {"p","t"})
public class User {
    private Long tId;
    private String pName;
}
```

编译查看字节码（如上两种都编译后都会生成如下字节码）：mvn compile

```java
public class User {
	private Long pId;
	private String pName;
	
	// 生成的getter和setter方法如下，方法体略
	public Long getId() {}
	public void setId(Long id) {}
	public String getName() {}
	public void setName(String name) {}
}
```





## @Slf4j/@Log4j（日志注解）

**@Slf4j**：自动生成该类的**log**静态常量，直接使用，不用再手动 new log 静态常量了

除了 @Slf4j 之外，lombok也提供其他日志框架的变种注解可以用，@Log4j、@Log...他们都是帮我们创建一个静态常量 log，只是使用的库不一样而已

```java
@Slf4j
public class User {
    // 编译查看字节码：mvn compile
    private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(User.class);
    
    public static void main(String[] args) {
        log.info("lombok-->Slf4j");
    }
}
```

其他日志框架：

```java
@Slf4j //对应的log语句如下
private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(User.class);

@Log4j //对应的log语句如下
private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);

@Log //对应的log语句如下
private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());
```

**注意：**必须引入对应的日志框架依赖

SpringBoot默认支持的就是 slf4j + logback 的日志框架，所以也不用再多做啥设定，直接就可以用在 SpringBoot project上，log 系列注解最常用的就是 @Slf4j





## Lombok插件原理—对比反射技术

**讲解JSR介绍和lombok原理讲解**

- 熟悉Java自定义注解的同学已经猜到是: JSR 269插件化注解处理

```
JSR 269:  Pluggable Annotation Processing API
实现在Javac编译阶段利用“Annotation Processor”对自定义的注解进行预处理后生成真正在JVM上面执行的“Class文件
地址：https://www.jcp.org/en/jsr/detail?id=269
```

- 科普

```
JSR是Java Specification Requests的缩写，意思是Java 规范提案。
是指向JCP(Java Community Process)提出新增一个标准化技术规范的正式请求。
任何人都可以提交JSR，以向Java平台增添新的API和服务。JSR已成为Java界的一个重要标准。
```

- Lombok解析流程

```
Javac 解析成AST抽象语法树后, Lombok根据自己编写的注解处理器，动态地修改 AST增加新的节点（即Lombok自定义注解所需要生成的代码）,最终生成JVM可执行的字节码Class文件

可以看编译后的在target目录下的class文件
```

- 能实现上述效果的还有一个反射技术，那两个对比如何？

```
使用Annotation Processing自定义注解是在编译阶段进行修改
JDK的反射技术是在运行时动态修改

结论：反射更加灵活一些但是带来的性能损耗更加大
```

