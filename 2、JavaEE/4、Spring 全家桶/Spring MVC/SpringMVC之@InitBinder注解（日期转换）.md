# Spring MVC之@InitBinder注解（日期转换）

> 作者：[YD_1989](https://blog.csdn.net/m0_58680865)、来源：SpringMVC之@InitBinder注解（日期转换）：https://blog.csdn.net/m0_58680865/article/details/125006308

## @InitBinder 注解作用

SpringMVC 并不是能对所有类型的参数进行绑定的，如果对日期 Date 类型参数进行绑定，就会报错 IllegalStateException 错误。所以需要注册一些类型绑定器用于对参数进行绑定。InitBinder注解就有这个作用。

代码示例：

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import java.util.Date;

@RestController
public class InitBinderController {

    @RequestMapping(value = "/testInitBinder", method = RequestMethod.GET)
    public String testInitBinder(Date date) {
        System.out.println("date = " + date);
        return "testInitBinder";
    }
}
```

测试 Controller API：

```bash
curl -X GET localhost:8080/testInitBinder?date=2019-05-15
{"timestamp":"2024-08-26T03:47:59.076+00:00","status":400,"error":"Bad Request","path":"/testInitBinder"}
```

控制台报错：

```java
2024-08-26 11:47:59.068  WARN 36167 --- [nio-8080-exec-3] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.web.method.annotation.MethodArgumentTypeMismatchException: Failed to convert value of type 'java.lang.String' to required type 'java.util.Date'; nested exception is org.springframework.core.convert.ConversionFailedException: Failed to convert from type [java.lang.String] to type [java.util.Date] for value '2019-05-15'; nested exception is java.lang.IllegalArgumentException]
```

报错提示不能把 String 类型转换为 Date 类型报错。此时就需要一个日期类型转换器。

```java
import org.springframework.format.datetime.DateFormatter;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import java.util.Date;

@RestController
public class InitBinderController {

    @RequestMapping(value = "/testInitBinder", method = RequestMethod.GET)
    private String testInitBinder(Date date) {
        System.out.println("date = " + date);
        return "testInitBinder";
    }

    @InitBinder
    public void dateTypeBinder(WebDataBinder webDataBinder) {
        // 往数据绑定器中添加一个DateFormatter日期转化器。
        webDataBinder.addCustomFormatter(new DateFormatter("yyyy-mm-dd"));
    }
}
```

测试 Controller API：

```bash
curl -X GET localhost:8080/testInitBinder?date=2019-05-15
testInitBinder
```

控制台输出：

```java
date = Tue Jan 15 00:05:00 CST 2019
```



## @InitBinder 注解源码

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface InitBinder {
    // 指定参数名，这个不知控制器方法上形参的参数名，而是请求参数名，
    // 可以指定多个。指定后只有这些参数需要用到该转换器。如果不指定，默认所有。
    String[] value() default {};
}
```

***注意：并且使用InitBinder 注册的绑定器只有在当前 Controller 中才有效，不会作用于其他 Controller。***



## @ControllerAdvice 全局注册

此时，就需要用到 @ControllerAdvice 注解定义全局绑定器。使不同 Controller 的方法都能作用到。

```java
import org.springframework.format.datetime.DateFormatter;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.InitBinder;

@ControllerAdvice
public class InitConfig {

    @InitBinder
    public void dateTypeBinder(WebDataBinder webDataBinder) {
        // 往数据绑定器中添加一个DateFormatter日期转化器。
        webDataBinder.addCustomFormatter(new DateFormatter("yyyy-mm-dd"));
    }
}
```



## 使用其他格式转化器

我们可以自定义格式转化器，实现 Formatter 接口就可。还可以添加验证器等等。

```java
public class StringFormatter implements Formatter<String> {
    private static final String PREFIX = "convertString == ";

    @Override
    public String parse(String text, Locale locale) throws ParseException {
        return rPREFIX + text;
    }

    @Override
    public String print(String object, Locale locale) {
        return object;
    }
}
```

添加

```java
import org.springframework.format.datetime.DateFormatter;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.InitBinder;

@ControllerAdvice
public class InitConfig {

    @InitBinder
    public void dateTypeBinder(WebDataBinder webDataBinder) {
        // 往数据绑定器中添加一个DateFormatter日期转化器。
        webDataBinder.addCustomFormatter(new DateFormatter("yyyy-mm-dd"));

        // 添加一个string类型的数据绑定器，作用是加个前缀
        webDataBinder.addCustomFormatter(new StringFormatter());
    }
}
```

测试 Controller API：

```bash
curl -X GET localhost:8080/testInitBinder?date=2019-05-15
testInitBinder
```

打印结果：

```java
name = convertString == 刘亦菲
```

