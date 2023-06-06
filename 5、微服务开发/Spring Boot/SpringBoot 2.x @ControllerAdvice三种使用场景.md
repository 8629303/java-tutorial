# 1、@ControllerAdvice 三种使用场景

SpringBoot 中 @ControllerAdvice 有三个方面的功能：

- 全局异常处理
- 全局数据绑定
- 全局数据预处理



# 2、@ControllerAdvice 全局异常处理

1、MyGlobalException.java + @RestControllerAdvice = 返回字符串

```java
/**
 * 全局异常处理类
 */
@RestControllerAdvice
public class MyGlobalException {
    @ExceptionHandler(ArithmeticException.class)
    public String customException(Exception e) {
        return "/ by zero";
    }
}
```

2、TestController.java

```java
@RestController
public class TestController {
    @GetMapping("test")
    public Integer testException() {
        return 1 / 0;
    }
}
```

3、测试：localhost:8080/test，返回结果：/ by zero

4、MyGlobalException.java + @ControllerAdvice  = 返回视图

```java
/**
 * 全局异常处理类
 */
@ControllerAdvice 
public class MyGlobalException {
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ModelAndView customException(MaxUploadSizeExceededException e) {
        ModelAndView mv = new ModelAndView("zhuantai");
        mv.addObject("error", e.getMessage());
        return mv;
    }
}
```

其他基本一直，Controller返回到页面即可



# 3、@ControllerAdvice 全局数据绑定

1、MyGlobalData.java

```java
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ModelAttribute;
import java.util.HashMap;
import java.util.Map;

/**
 * 全局数据绑定
 */
@ControllerAdvice
public class MyGlobalData {

    // 不定义名称默认为Map
    @ModelAttribute("info")
    public Map<String,String> myData(){
        Map<String, String> info = new HashMap<>();
        info.put("username","sam");
        info.put("password","123456");
        return info;
    }
}
```

2、HelloController.java

```java
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Map;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public void hello(Model model){
        Map<String, Object> asMap = model.asMap();
        Map<String,String> info = (Map<String, String>) asMap.get("info");
        for (String s : info.keySet()){
            System.out.println("s = " + s);
        }
    }
}
```

4、运行结果：

```
s = addredd
s = username
```



# 4、@ControllerAdvice 全局数据预处理

1、MyGlobalData.java

```java
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import java.util.HashMap;
import java.util.Map;

/**
 * 全局数据绑定
 */
@ControllerAdvice
public class MyGlobalData {

    @InitBinder("b")
    public void b(WebDataBinder binder){
        // 设置一个默认的前缀
        binder.setFieldDefaultPrefix("b.");
    }

    @InitBinder("a")
    public void a(WebDataBinder binder){
        // 设置一个默认的前缀
        binder.setFieldDefaultPrefix("a.");
    }
}
```

2、BookController.java

```java
import com.zhuantai.controlleradvice.model.Author;
import com.zhuantai.controlleradvice.model.Book;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class  BookController {
    @PostMapping("/book")
    public void addBook(@ModelAttribute("b") Book book, @ModelAttribute("a") Author author){
        System.out.println("book = " + book);
        System.out.println("author = " + author);
    }
}
```

3、Book.java

```java
@Data
public class Book {
    private String name;
    private Double price;
}
```

4、Author.java

```java
@Data
public class Author {
    private String name;
    private Integer age;
}
```

4、测试：localhost:8080/book?b.name=123&b.price=20&a.name=Sam&a.age=18