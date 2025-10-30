> 作者：Java中文社群；链接：https://mp.weixin.qq.com/s/CAVgFuDwazMfU6nXk1gWNw
>

## 一、前言介绍

在实际的项目开发中，我们往往需要根据不同的环境做出不同的配置，例如：在开发环境下，我们会使用内存数据库以便快速启动服务并进行开发调试，在test环境、生产环境，会使用对应环境的数据库。

如果我们的应用程序可以根据自身的环境做一些这样的适配，那么我们的程序开发无疑将更加灵活、高效。

在过去的应用程序开发中，我们常常会将这些环境变量写在某个指定的配置文件中，每次服务器启动的时候，会读取服务器中指定的配置文件，从而实现根据不同的环境，应用程序能做出对应的适配。

但是这样的工作，对于运维来说，非常苦逼，尤其是应用程序到达50个以上的时候，会非常不好维护，每次上线改配置，全靠人肉，想想都觉得反人类～

当我们在使用SpringBoot来开发应用程序的时候，这些工作量将大大简化。

SpringBoot为开发者提供了三种可选的条件装配方式。

- Profile
- Conditional
- ConditionalOnProperty

下面，我们一起来了解一下具体的应用实践。



## 二、代码实践

### 1、Profile

SpringBoot 为应用程序提供了 Profile 这一概念，用来表示不同的环境。例如，我们分别定义开发、测试和生产这3个环境

- dev：开发环境
- test：测试环境
- production：生产环境

以上传文件为例，在开发环境下，我们将文件上传到本地，而在测试环境、生产环境，我们将文件上传到云端服务商。

1、首先编写两套上传服务

```java
/**
 * 上传文件到本地
 * @since 2021-06-13
 */
public class FileUploader implements Uploader {

    @Override
    public String upload(File file) {
        // 上传文件到本地,并返回绝对路径
        return null;
    }
}
/**
 * 上传文件到OSS
 * @since 2021-06-13
 */
public class OSSUploader implements Uploader {

    @Override
    public String upload(File file) {
        // 上传文件到云端,并返回绝对路径
        return null;
    }
}
```

2、然后编写一个服务配置类，根据不同的环境，创建不同的实现类

```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("dev")
    public Uploader initFileUploader() {
        System.out.println("初始化一个上传到本地的bean");
        return new FileUploader();
    }

    @Bean
    @Profile("!dev")
    public Uploader initOSSUploader() {
        System.out.println("初始化一个上传到云端的bean");
        return new OSSUploader();
    }
}
```

3、最后，运行程序。在运行程序时，加上JVM参数`-Dspring.profiles.active=dev`就可以指定以dev环境启动。

如果当前Profile为dev，则Spring容器会调用initFileUploader()创建FileUploader，否则，调用initOSSUploader()创建OSSUploader

注意：`@Profile("!dev")`表示非dev环境。

当然，你还可以在application.properties文件中加上如下配置，一样可以指定环境进行运行。

```properties
spring.profiles.active=dev
```



### 2、Conditional

除了可以根据@Profile条件来决定是否创建某个Bean外，Spring还可以根据@Conditional决定是否创建某个Bean。

以发短信为例，在生产环境，我们会提供发短信服务，而在其他环境，我们不会向运营商发短信。

1、创建一个条件配置类：SMSEnvCondition.java

```java
public class SMSEnvCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return "true".equalsIgnoreCase(context.getEnvironment().getProperty("enable.sms"));
    }
}
```

2、创建一个发短信的服务

```java
@Component
@Conditional(SMSEnvCondition.class)
public class SendMessageService {
    // do someing
}
```

3、在application.properties文件中，添加配置变量enable.sms

```properties
enable.sms=true
```

当enable.sms为true的时候，会创建SendMessageService对象，否则不创建。



### 3、ConditionalOnProperty

Spring提供的条件装配@Conditional，灵活性非常强，但是具体判断逻辑还需要我们自己实现，比较麻烦。

实际上，SpringBoot 为开发者提供了很多使用起来更简单的条件注解，例如：

- ConditionalOnProperty：如果有指定的配置，条件生效
- ConditionalOnBean：如果有指定的Bean，条件生效
- ConditionalOnMissingBean：如果没有指定的Bean，条件生效
- ConditionalOnMissingClass：如果没有指定的Class，条件生效
- ConditionalOnWebApplication：在Web环境中条件生效
- ConditionalOnExpression：根据表达式判断条件是否生效

我们以最常用的@ConditionalOnProperty注解为例，将上面的代码改成如下方式即可实现按照条件进行加载。

```java
@Component
@ConditionalOnProperty(name="enable.sms", havingValue="true")
public class SendMessageService {
    // do someing
}
```

当enable.sms的值等于true时，会实例化SendMessageService对象；反之，不会创建对象。

当然@ConditionalOnProperty的参数还不仅仅限于此，以上面上传文件为例，在开发环境，我们总是上传到本地；在测试环境、生产环境，我们将文件上传到云端，改造过程如下：

```java
@Component
@ConditionalOnProperty(name = "file.storage", havingValue = "file", matchIfMissing = true)
public class FileUploader implements Uploader {
    @Override
    public String upload(File file) {
        // 上传文件到本地,并返回绝对路径
        return null;
    }
}

@Component
@ConditionalOnProperty(name = "file.storage", havingValue = "oss")
public class OSSUploader implements Uploader {
    @Override
    public String upload(File file) {
        // 上传文件到云端,并返回绝对路径
        return null;
    }
}
```

当file.storage配置值为file，会加载FileUploader类；当file.storage配置值为`oss`，会加载OSSUploader类。

其中@ConditionalOnProperty中的matchIfMissing参数表示，当没有找到对应配置参数时，会默认加载当前类，也就是FileUploader类。



## 三、文末总结

虽然：@Profile、@Conditional、@ConditionalOnProperty 三个注解都能实现按照条件进行适配，但是@Profile注解控制比较粗糙，很难实现精细化控制。在实际的使用过程中，使用最多的是@Conditional、@ConditionalOnProperty，可以很灵活的实现条件装配。

其中，@ConditionalOnProperty是@Conditional的一种具体扩展实现，提供了很多非常实用的操作，在使用中，推荐大家使用@ConditionalOnProperty。如果不够，可以根据@Conditional条件装配，编写一套控制开关实现类。
