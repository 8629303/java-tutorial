> 请不要自己写，Spring Boot非常实用的内置功能：https://juejin.cn/post/7417630844100231206

在 SpringBoot 框架中，内置了许多实用的功能，这些功能可以帮助开发者高效地开发和维护应用程序。

一、请求数据记录器
========

SpringBoot 提供了一个内置的日志记录解决方案，通过 AbstractRequestLoggingFilter 可以记录请求的详细信息。

AbstractRequestLoggingFilter 有两个不同的实现类，我们常用的是 CommonsRequestLoggingFilter。

```
AbstractRequestLoggingFilter
    CommonsRequestLoggingFilter, ServletRequestLoggingFilter
```

通过 CommonsRequestLoggingFilter 开发者可以自定义记录请求的参数、请求体、请求头和客户端信息。

启用方式很简单，加个配置就行了：

```java
@Configuration
public class RequestLoggingConfig {
    @Bean
    public CommonsRequestLoggingFilter logFilter() {
        CommonsRequestLoggingFilter filter = new CommonsRequestLoggingFilter();
        filter.setIncludeQueryString(true);
        filter.setIncludePayload(true);
        filter.setIncludeHeaders(true);
        filter.setIncludeClientInfo(true);
        filter.setAfterMessagePrefix("REQUEST DATA-");
        return filter;
    }
    
    @Bean
    public CommonsRequestLoggingFilter logFilter() {
        CommonsRequestLoggingFilter filter = new CommonsRequestLoggingFilter();
        // 设置是否包含客户端信息（例如IP地址）
        filter.setIncludeClientInfo(true);
        // 设置是否包含请求参数
        filter.setIncludeQueryString(true);
        // 设置是否包含请求头
        filter.setIncludeHeaders(true);
        // 设置是否包含请求体
        filter.setIncludePayload(true);
        // 设置请求体的最大长度
        filter.setMaxPayloadLength(1000);
        // 记录的前缀和后缀，可以自定义
        filter.setBeforeMessagePrefix("Request Received : ");
        filter.setAfterMessageSuffix(" [END]");
        return filter;
    }
}
```

接下来需要配置日志级别为 DEBUG，就可以详细记录请求信息：

```properties
logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=DEBUG
```

创建一个 POST 接口测试：

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class UserController {

    @PostMapping("/user")
    public String createUser(@RequestBody String user, @RequestHeader("Custom-Header") String customHeader) {
        // 可以返回传递过来的请求数据，方便测试
        return "User created with body: " + user + " and header: " + customHeader;
    }
}
```

你可以通过 curl 或 Postman 来测试发送 POST 请求。

```bash
curl -X POST http://localhost:8080/api/user \
    -H "Content-Type: application/json" \
    -H "Custom-Header: TestHeader" \
    -d '{"name":"John", "age":30}'
```

在控制台或日志文件中，你应该能看到类似如下的日志输出：

```
Request Received : POST http://localhost:8080/api/user
Headers : [Content-Type=application/json, Custom-Header=TestHeader]
Client : [IP=127.0.0.1]
Payload : {"name":"John", "age":30}
Request Received :  [END]
```



二、请求响应包装器
==========

## 1、什么是请求和响应包装器

在 SpringBoot 中，请求和响应包装器是用于增强原生 HttpServletRequest 和 HttpServletResponse 对象的功能。这些包装器允许开发者在请求处理过程中拦截和修改请求和响应数据，从而实现一些特定的功能，如请求内容的缓存、修改、日志记录，以及响应内容的修改和增强。

### 1、请求包装器

*   ContentCachingRequestWrapper：这是 Spring 提供的一个请求包装器，用于缓存请求的输入流。它允许多次读取请求体，这在需要多次处理请求数据（如日志记录和业务处理）时非常有用。

### 2、响应包装器

*   ContentCachingResponseWrapper：这是 Spring 提供的一个响应包装器，用于缓存响应的输出流。它允许开发者在响应提交给客户端之前修改响应体，这在需要对响应内容进行后处理（如添加额外的头部信息、修改响应体）时非常有用。



## 2、使用场景

1.  **请求日志记录**：在处理请求之前和之后记录请求的详细信息，包括请求头、请求参数和请求体。
2.  **修改请求数据**：在请求到达控制器之前修改请求数据，例如添加或修改请求头。
3.  **响应内容修改**：在响应发送给客户端之前修改响应内容，例如添加或修改响应头，或者对响应体进行签名。
4.  **性能测试**：通过缓存请求和响应数据，可以进行性能测试，而不影响实际的网络 I/O 操作。



## 3、具体用法

### 请求包装器的使用

```java
import org.springframework.web.filter.OncePerRequestFilter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class RequestWrapperFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(request);
        // 可以在这里处理请求数据
        byte[] body = requestWrapper.getContentAsByteArray();
        // 处理body，例如记录日志
        //。。。
        filterChain.doFilter(requestWrapper, response);
    }
}
```

### 响应包装器的使用

```java
import org.springframework.web.filter.OncePerRequestFilter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class ResponseWrapperFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper(response);
        filterChain.doFilter(request, responseWrapper);

        // 可以在这里处理响应数据
        byte[] body = responseWrapper.getContentAsByteArray();
        // 处理body，例如添加签名
        responseWrapper.setHeader("X-Signature", "some-signature");

        // 必须调用此方法以将响应数据发送到客户端
        responseWrapper.copyBodyToResponse();
    }
}
```

在上面的案例中，OncePerRequestFilter 确保过滤器在一次请求的生命周期中只被调用一次，这对于处理请求和响应数据尤为重要，因为它避免了在请求转发或包含时重复处理数据。

通过使用请求和响应包装器，开发者可以在不改变原有业务逻辑的情况下，灵活地添加或修改请求和响应的处理逻辑。



三、单次过滤器
=======

## 1、OncePerRequestFilter 介绍

OncePerRequestFilter 是 Spring 框架提供的一个过滤器基类，它继承自 Filter 接口。这个过滤器具有以下特点：

1.  **单次执行**：OncePerRequestFilter 确保在一次请求的生命周期内，无论请求如何转发（forwarding）或包含（including），过滤器逻辑只执行一次。这对于避免重复处理请求或响应非常有用。
    
2.  **内置支持**：它内置了对请求和响应包装器的支持，使得开发者可以方便地对请求和响应进行包装和处理。
    
3.  **简化代码**：通过继承 OncePerRequestFilter，开发者可以减少重复代码，因为过滤器的执行逻辑已经由基类管理。
    
4.  **易于扩展**：开发者可以通过重写 doFilterInternal 方法来实现自己的过滤逻辑，而不需要关心过滤器的注册和执行次数。



## 2、OncePerRequestFilter 使用场景

1.  **请求日志记录**：在请求处理之前和之后记录请求的详细信息，如请求头、请求参数和请求体，而不希望在请求转发时重复记录。
    
2.  **请求数据修改**：在请求到达控制器之前，对请求数据进行预处理或修改，例如添加或修改请求头，而不希望这些修改在请求转发时被重复应用。
    
3.  **响应数据修改**：在响应发送给客户端之前，对响应数据进行后处理或修改，例如添加或修改响应头，而不希望这些修改在请求包含时被重复应用。
    
4.  **安全控制**：实现安全控制逻辑，如身份验证、授权检查等，确保这些逻辑在一次请求的生命周期内只执行一次。
    
5.  **请求和响应的包装**：使用 ContentCachingRequestWrapper 和 ContentCachingResponseWrapper 等包装器来缓存请求和响应数据，以便在请求处理过程中多次读取或修改数据。
    
6.  **性能监控**：在请求处理前后进行性能监控，如记录处理时间，而不希望这些监控逻辑在请求转发时被重复执行。
    
7.  **异常处理**：在请求处理过程中捕获和处理异常，确保异常处理逻辑只执行一次，即使请求被转发到其他处理器。
    

通过使用 OncePerRequestFilter，开发者可以确保过滤器逻辑在一次请求的生命周期内只执行一次，从而避免重复处理和潜在的性能问题。这使得 OncePerRequestFilter 成为处理复杂请求和响应逻辑时的一个非常有用的工具。

OncePerRequestFilter 的具体用法就不举例了，第二小节已经介绍过了。



四、切面 AOP 三件套
=========

在 Spring 框架中，AOP（面向切面编程）是一个强大的功能，它允许开发者在不修改源代码的情况下，对程序的特定部分进行横向切入。AopContext、AopUtils 和 ReflectionUtils 是 Spring AOP 中提供的几个实用类。



## 1、AopContext

AopContext 是 Spring 框架中的一个类，它提供了对当前 AOP 代理对象的访问，以及对目标对象的引用。

AopContext 主要用于获取当前代理对象的相关信息，以及在 AOP 代理中进行一些特定的操作。

**常见方法有两个**：

*   getTargetObject()：获取当前代理的目标对象。
*   currentProxy()：获取当前的代理对象。

其中第二个方法，在防止同一个类中注解失效的时候，可以通过该方法获取当前类的代理对象。

```java
public void noTransactionTask(String keyword) {
    // 注意这里 调用了代理类的方法, 如果直接调用会导致事务失效
    ((YourClass) AopContext.currentProxy()).transactionTask(keyword);
}

@Transactional
void transactionTask(String keyword) {
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        // error tracking
    }
    System.out.println(keyword);
}
```

同一个类中两个方法，noTransactionTask 方法调用 transactionTask 方法，为了使事务注解不失效，就可以使用 AopContext.currentProxy() 去获取当前代理对象。

注意：AopContext 访问代理对象，需要写注解 @EnableAspectJAutoProxy(exposeProxy = true)，开启代理对象访问，exposeProxy 注解属性默认是 false。



## 2、AopUtils

AopUtils 提供了一些静态方法来处理与 AOP 相关的操作，如获取代理对象、获取目标对象、判断代理类型等。

**常见方法有三个**：

*   getTargetObject()：从代理对象中获取目标对象。
*   isJdkDynamicProxy(Object obj)：判断是否是 JDK 动态代理。
*   isCglibProxy(Object obj)：判断是否是 CGLIB 代理。

```java
import org.springframework.aop.framework.AopProxyUtils;
import org.springframework.aop.support.AopUtils;

public class AopUtilsExample {
    public static void main(String[] args) {
        MyService myService = ...;
        // 假设 myService 已经被代理
        if (AopUtils.isCglibProxy(myService)) {
            System.out.println("这是一个 CGLIB 代理对象");
        }
    }
}
```



## 3、ReflectionUtils

ReflectionUtils 提供了一系列反射操作的便捷方法，如设置字段值、获取字段值、调用方法等。这些方法封装了 Java 反射 API 的复杂性，使得反射操作更加简单和安全。

**常见方法**：

*   makeAccessible(Field field)：使私有字段可访问。
*   getField(Field field, Object target)：获取对象的字段值。
*   invokeMethod(Method method, Object target, Object... args)：调用对象的方法。

```java
import org.springframework.util.ReflectionUtils;

import java.lang.reflect.Field;
import java.util.Map;

public class ReflectionUtilsExample {
    public static void main(String[] args) throws Exception {
        ExampleBean bean = new ExampleBean();
        bean.setMapAttribute(new HashMap<>());

        Field field = ReflectionUtils.findField(ExampleBean.class, "mapAttribute");
        ReflectionUtils.makeAccessible(field);

        Object value = ReflectionUtils.getField(field, bean);
        System.out.println(value);
    }

    static class ExampleBean {
        private Map<String, String> mapAttribute;

        public void setMapAttribute(Map<String, String> mapAttribute) {
            this.mapAttribute = mapAttribute;
        }
    }
}
```
