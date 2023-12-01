# SpringBoot 加载后初始化的N种方式

## 0、SpringBoot 环境搭建

本人使用的是SpringBoot 2.7.18 + Java11，pom.xml 如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>SpringBootTech_1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringBootTech_1</name>
    <description>SpringBootTech_1</description>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>nexus-aliyun</id>
            <name>nexus-aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>public</id>
            <name>ali nexus</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

</project>
```



## 1、构造方法和静态代码块

构造方法和静态代码块属于 Java 自身的启动时加载方式：

1. 静态代码块：static静态代码块，在类加载的时候即自动执行。
2. 构造方法：在对象初始化时执行。执行顺序在static静态代码块之后。
3. 加载顺序：static代码块 > 构造方法

在Spring或者SpringBoot中使用时，想要在SpringBoot启动时就初始化，需要把Bean加入IOC容器管理，所以会使用到@Component等注解。

```java
import org.springframework.stereotype.Component;

@Component
public class TestConstruct {
    static {
        System.out.println("Static");
    }
    public TestPostConstruct() {
        System.out.println("Construct");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
Static
Construct
2023-12-01 13:58:09.934  INFO 85202 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-12-01 13:58:09.947  INFO 85202 --- [           main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
```



## 2、@PostConstruct 注解

PostConstruct注解修饰的方式，是在Spring容器启动时运行的。

```java
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class TestPostConstruct {
    static {
        System.out.println("Static");
    }
    public TestPostConstruct() {
        System.out.println("Construct");
    }

    @PostConstruct
    public void init() {
        System.out.println("PostConstruct");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
Static
Construct
PostConstruct
2023-12-01 13:58:09.934  INFO 85202 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-12-01 13:58:09.947  INFO 85202 --- [           main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
```



## 3、InitializingBean

InitializingBean是spring容器在启动并初始化好内部示例后调用的，用来最终为总体bean添加最后属性和操作。 官方原话：This method allows the bean instance to perform validation of its overall configuration and final initialization when all bean properties have been set.

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
public class TestInitializingBean implements InitializingBean {
    @Override
    public void afterPropertiesSet() {
        System.out.println("InitializingBean");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
```



## 4、实现 SmartInitializingSingleton 接口

SmartInitializingSingleton是Bean容器在初始化所有非懒加载的单例Bean后调用的方法。

```java
import org.springframework.beans.factory.SmartInitializingSingleton;
import org.springframework.stereotype.Component;

@Component
public class TestSmartInitializingSingleton implements SmartInitializingSingleton {
    @Override
    public void afterSingletonsInstantiated() {
        System.out.println("SmartInitializingSingleton");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
```



## 5、BeanFactoryPostProcessor.initMethod 方法

这种方法有一定的局限性，并且可能会覆盖曾经的init操作，需要慎用。

Bean在加载到Spring容器中时需要先将Bean的定义信息抽象为BeanDefinition，其中有一个属性init-method代表将来Bean初始化时要调用的方法。

我们通过BeanFactoryPostProcessor来注入init-method方法，并且该方法必须是没有参数的 。

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;

@Component
public class TestBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        // BeanDefinition beanDefinition = beanFactory.getBeanDefinition("person");
        // beanDefinition.setAttribute("setName", "123");
        // beanDefinition.setInitMethodName("run");
        System.out.println("BeanFactoryPostProcessor");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
BeanFactoryPostProcessor
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
```



## 6、监听器监听 ContextRefreshedEvent 事件

ContextRefreshedEvent：是Spring容器初始化完成后调用的事件。 ContextRefreshedEvent的父类是ApplicationContextEvent，是一个事件。所以我们通过ApplicationListener来实现。

```java
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

@Component
public class TestApplicationListenerByContextRefreshedEvent implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("ApplicationListener.ContextRefreshedEvent");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
BeanFactoryPostProcessor
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
ApplicationListener.ContextRefreshedEvent
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
```



## 7、监听器监听 ApplicationStartedEvent 事件

ApplicationStartedEvent 是 Spring Boot 中的一个事件类，它表示应用程序已经启动完成的事件。当 Spring Boot 应用程序启动并且应用程序上下文准备好接收请求时，将发布 ApplicationStartedEvent 事件。你可以监听这个事件，以执行在应用程序启动后需要进行的操作。

这个事件通常用于执行一些在应用程序完全启动后立即执行的逻辑。你可以创建一个监听器（实现 `ApplicationListener<ApplicationStartedEvent>` 接口）来响应这个事件。

```java
import org.springframework.boot.context.event.ApplicationStartedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
public class TestApplicationListenerByApplicationStartedEvent implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("ApplicationListener.ApplicationStartedEvent");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
BeanFactoryPostProcessor
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
ApplicationListener.ContextRefreshedEvent
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
ApplicationListener.ApplicationStartedEvent
```



## 8、实现 ApplicationRunner 接口（仅限SpringBoot）

ApplicationRunner 也是一个SpringBoot 接口，在应用初始化后执行，且仅会执行一次。可以用来打印项目中配置文件的参数，参数为ApplicationRunner。

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class TestApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments applicationArguments) {
        System.out.println("ApplicationRunner");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
BeanFactoryPostProcessor
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
ApplicationListener.ContextRefreshedEvent
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
ApplicationListener.ApplicationStartedEvent
ApplicationRunner
```



## 9、实现 CommandLineRunner 接口（仅限SpringBoot）

CommandLineRunner 是一个SpringBoot 接口，在应用初始化后执行，且仅会执行一次。可以用来打印项目中配置文件的参数，参数为String数组。

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class TestCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... strings) {
        System.out.println("CommandLineRunner");
    }
}
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
BeanFactoryPostProcessor
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
ApplicationListener.ContextRefreshedEvent
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
ApplicationListener.ApplicationStartedEvent
ApplicationRunner
CommandLineRunner
```

***

ApplicationRunner 和 CommandLineRunner：SpringBoot提供了两个接口来实现Spring容器启动完成后执行的功能

- 这两个接口需要实现一个run方法，将代码在run中实现即可。这两个接口功能基本一致，其区别在于run方法的入参。ApplicationRunner的run方法入参为ApplicationArguments，而CommandLineRunner的run方法入参为String数组。



## 10、实现 SpringApplicationRunListener 接口（仅限SpringBoot）

SpringBoot的生命周期事件监听方法，需要搭配resource/META-INF/spring.factories 文件使用。【SpringBoot3.0 无需定义SpringApplication和args】

```java
package com.example.springboot.tech;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringApplicationRunListener;
import org.springframework.context.ConfigurableApplicationContext;
import java.time.Duration;

public class TestSpringApplicationRunListener implements SpringApplicationRunListener {

    private final SpringApplication application;
    private final String[] args;

    public TestSpringApplicationRunListener(SpringApplication application, String[] args) {
        this.application = application;
        this.args = args;
    }

    @Override
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        System.out.println("SpringApplicationRunListener:started");
    }

    @Override
    public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        System.out.println("SpringApplicationRunListener:ready");
    }
}
```

在resources目录下添加 META-INF/spring.factories 配置文件，内容如下，将自定义的初始化器和监听器注册进去；

```properties
org.springframework.boot.SpringApplicationRunListener=com.example.springboot.tech.TestSpringApplicationRunListener
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
BeanFactoryPostProcessor
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
ApplicationListener.ContextRefreshedEvent
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
ApplicationListener.ApplicationStartedEvent
SpringApplicationRunListener:started
ApplicationRunner
CommandLineRunner
SpringApplicationRunListener:ready
```



## 11、自定义初始化器实现 ApplicationContextInitializer 接口

ApplicationContextInitializer 是 Spring Framework 提供的一个接口，用于在应用程序上下文（ApplicationContext）创建之前对其进行初始化配置。通过实现这个接口，你可以在 Spring 应用程序启动时执行一些定制化的初始化操作，例如设置一些属性、激活或禁用某些配置等。

该接口定义了一个方法 initialize，该方法接受一个 ConfigurableApplicationContext 参数，允许你对应用程序上下文进行配置。

```java
package com.example.springboot.tech;

import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;

/**
 * 自定义的初始化器
 */
public class TestApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        // 在这里进行初始化操作，可以修改上下文的配置
        System.out.println("TestApplicationContextInitializer...");
    }
}
```

在resources目录下添加 META-INF/spring.factories 配置文件，内容如下，将自定义的初始化器和监听器注册进去；

```properties
org.springframework.context.ApplicationContextInitializer=com.example.springboot.tech.TestApplicationContextInitializer
```

启动加载Spring或者SpringBoot容器时，会输出如下信息：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.7.18)

TestApplicationContextInitializer...
2023-12-01 13:58:08.576  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Starting SpringBootTechApplication using Java 11 on MacBook-Pro.local with PID 85202 (/Users/lsx/IdeaProjects/SpringBootTech_1/target/classes started by lsx in /Users/lsx/IdeaProjects/SpringBootTech_1)
2023-12-01 13:58:08.581  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : No active profile set, falling back to 1 default profile: "default"
BeanFactoryPostProcessor
2023-12-01 13:58:09.416  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-12-01 13:58:09.427  INFO 85202 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-12-01 13:58:09.427  INFO 85202 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.83]
2023-12-01 13:58:09.519  INFO 85202 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-12-01 13:58:09.520  INFO 85202 --- [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 866 ms
InitializingBean
Static
Construct
PostConstruct
SmartInitializingSingleton
2023-12-01 13:58:09.934  INFO 85202 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
ApplicationListener.ContextRefreshedEvent
2023-12-01 13:58:09.947  INFO 85202 --- [main] c.e.s.tech.SpringBootTechApplication     : Started SpringBootTechApplication in 1.853 seconds (JVM running for 2.658)
ApplicationListener.ApplicationStartedEvent
SpringApplicationRunListener:started
ApplicationRunner
CommandLineRunner
SpringApplicationRunListener:ready
```



## 12、总结各个初始化方法的优先级

执行优先级：ApplicationContextInitializer »» BeanFactoryPostProcessor.initMethod »» InitializingBean »» 静态代码块 »» 构造方法 »» PostConstruct注解 »» SmartInitializingSingleton »» ApplicationListener.ContextRefreshedEvent »» ApplicationListener.ApplicationStartedEvent »» SpringApplicationRunListener:started »» ApplicationRunner »» CommandLineRunner »» SpringApplicationRunListener:ready



## 13、参考文献 & 鸣谢

- Spring加载后初始化的9种方式：https://blog.csdn.net/kinbridge/article/details/131366212

# SpringBoot 加载后初始化简洁版

## 1、项目启动时行初始化方法前言

目前开发的SpringBoot项目在启动的时候需要预加载一些资源。而如何实现启动过程中执行代码，或启动成功后执行，是有很多种方式可以选择，我们可以在**static代码**块中实现，也可以在**构造方法**里实现，也可以使用**@PostConstruct**注解实现。

当然也可以去实现Spring的**ApplicationRunner**与**CommandLineRunner**接口去实现启动后运行的功能。在这里整理一下，在这些位置执行的区别以及加载顺序。

如下是Spring中常用的初始化方式

1. 实现 CommandLineRunner 接口
2. 实现 ApplicationRunner 接口
3. 通过 EventListener 监听启动事件
4. 通过 @Postconstruct 注解
5. 实现 InitializingBean 接口
6. 通过 @Bean 注解



## 2、Java 自身的启动时加载方式

1、static代码块：static静态代码块，在类加载的时候即自动执行。

2、构造方法：在对象初始化时执行。执行顺序在static静态代码块之后。

3、加载顺序：static代码块 > 构造方法



## 3、SpringBoot 启动时加载方式

1、@PostConstruct注解：PostConstruct注解使用在方法上，这个方法在对象依赖注入初始化之后执行。

2、ApplicationRunner 和 CommandLineRunner：SpringBoot提供了两个接口来实现Spring容器启动完成后执行的功能

这两个接口需要实现一个run方法，将代码在run中实现即可。这两个接口功能基本一致，其区别在于run方法的入参。ApplicationRunner的run方法入参为ApplicationArguments，而CommandLineRunner的run方法入参为String数组。

何为 ApplicationArguments

官方文档解释为：`Provides access to the arguments that were used to run a SpringApplication.`

在Spring应用运行时使用的访问应用参数。即我们可以获取到`SpringApplication.run(…)`的应用参数。

当有多个类实现了`CommandLineRunner`和`ApplicationRunner`接口时，可以通过在类上添加**@Order**注解来设定运行顺序。



## 4、启动时初始化方法代码测试

> 为了测试启动时运行的效果和顺序，编写几个测试代码来运行看看。

1、TestPostConstruct

```java
@Component
public class TestPostConstruct {

    static {
        System.out.println("static");
    }
    public TestPostConstruct() {
        System.out.println("constructer");
    }

    @PostConstruct
    public void init() {
        System.out.println("PostConstruct");
    }
}
```

2、TestApplicationRunner

```java
@Component
@Order(1)
public class TestApplicationRunner implements ApplicationRunner{
    @Override
    public void run(ApplicationArguments applicationArguments) throws Exception {
        System.out.println("order1:TestApplicationRunner");
    }
}
```

3、TestCommandLineRunner

```java
@Component
@Order(2)
public class TestCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... strings) throws Exception {
        System.out.println("order2:TestCommandLineRunner");
    }
}
```

3、执行结果

```
static
constructer
PostConstruct
order1:TestApplicationRunner
order2:TestCommandLineRunner
```



## 5、启动时初始化加载顺序总结

SpringBoot应用启动过程中，肯定是要自动扫描有@Component注解的类，加载类并初始化对象进行自动注入。加载类时首先要执行static静态代码块中的代码，之后再初始化对象时会执行构造方法。

在对象注入完成后，调用带有@PostConstruct注解的方法。当容器启动成功后，再根据@Order注解的顺序调用CommandLineRunner和ApplicationRunner接口类中的run方法。

因此加载顺序为：static > constructer > @PostConstruct > CommandLineRunner 和 ApplicationRunner



