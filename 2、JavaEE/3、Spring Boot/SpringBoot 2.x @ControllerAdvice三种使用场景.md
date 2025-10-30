# @ControllerAdvice 几种使用场景

SpringBoot 中 @ControllerAdvice 有三个方面的功能：

- 全局异常处理
- 全局数据绑定
- 全局数据预处理



# 1、全局异常处理

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

---

4、MyGlobalException.java + @ControllerAdvice  = 返回视图

```java
/**
 * 全局异常处理类
 */
@ControllerAdvice 
public class MyGlobalException {
    // 处理视图请求的异常
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ModelAndView customException(MaxUploadSizeExceededException e) {
        ModelAndView mv = new ModelAndView("error");
        mv.addObject("error", e.getMessage());
        return mv;
    }
}
```

其他基本一致，Controller 返回到页面即可。

**注意**：@RestControllerAdvice + @ExceptionHandler = @ControllerAdvice + @ExceptionHandler + @ResponseBody







# 2、全局数据绑定

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



# 3、全局数据预处理

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
    
    /**
     * 它会自动把 yyyy-MM-dd 格式的字符串转换为 Date 类型。
     * 可以防止出现 HTTP 400 - Bad Request 这样的错误。
     */
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        // 注册自定义编辑器，将字符串转换为日期
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
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



# 4、处理 REST API 的异常

针对 REST API，我们可以返回标准化的错误响应。

```java
@ControllerAdvice
public class RestExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse response = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "资源未找到",
                ex.getMessage(),
                LocalDateTime.now()
        );
        return new ResponseEntity<>(response, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthenticationException(AuthenticationException ex) {
        ErrorResponse response = new ErrorResponse(
                HttpStatus.UNAUTHORIZED.value(),
                "认证失败",
                ex.getMessage(),
                LocalDateTime.now()
        );
        return new ResponseEntity<>(response, HttpStatus.UNAUTHORIZED);
    }
}

// 错误响应类
@Data
class ErrorResponse {
    private int status;
    private String error;
    private String message;
    private LocalDateTime timestamp;
}
```



# 5、指定控制器范围

```java
@ControllerAdvice(annotations = RestController.class)        // 只对 @RestController 生效
// @ControllerAdvice(basePackages = "com.example.admin")     // 只对特定包生效
// @ControllerAdvice(assignableTypes = UserController.class) // 只对特定控制器生效
public class RestControllerAdvice {
    // 具体实现
}
```



# 6、与 AOP 搭配混合使用

与AOP搭配使用，如下示例只能拦截控制程的注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodLog {
    String value() default "";
    String[] ignoreParams() default {};
}

@RestControllerAdvice
public class LogAdvice {
    @Around("@annotation(com.example.annotation.MethodLog)")
    public Object logMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        // 获取目标方法
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        MethodLog annotation = method.getAnnotation(MethodLog.class);
        
        // 记录方法信息
        String methodName = method.getDeclaringClass().getName() + "." + method.getName();
        log.info("Method: {} - Start", methodName);
        
        // 记录方法入参（脱敏处理）
        Object[] args = joinPoint.getArgs();
        String[] parameterNames = signature.getParameterNames();
        Object[]脱敏Args = desensitizeArgs(args, parameterNames, annotation.ignoreParams());
        log.info("Method: {} - Params: {}", methodName, Arrays.toString(脱敏Args));
        
        // 执行方法并记录返回值
        long startTime = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long executionTime = System.currentTimeMillis() - startTime;
        
        log.info("Method: {} - Result: {} (耗时: {}ms)", 
                methodName, result, executionTime);
                
        return result;
    }
    
    private Object[] desensitizeArgs(Object[] args, String[] parameterNames, String[] ignoreParams) {
        // 实现参数脱敏逻辑
        List<String> ignoreList = Arrays.asList(ignoreParams);
        Object[]脱敏Args = new Object[args.length];
        
        for (int i = 0; i < args.length; i++) {
            if (ignoreList.contains(parameterNames[i])) {
                脱敏Args[i] = "******"; // 脱敏处理
            } else {
                脱敏Args[i] = args[i];
            }
        }
        return 脱敏Args;
    }
}
```



# 7、与 ResponseEntityExceptionHandler 结合使用

继承 `ResponseEntityExceptionHandler` 类，能够对 Spring MVC 的标准异常进行自定义处理。

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid (
            MethodArgumentNotValidException ex,
            HttpHeaders headers,
            HttpStatus status,
            WebRequest request) {
        Map<String, Object> body = new LinkedHashMap<>();
        body.put("timestamp", LocalDateTime.now());
        body.put("status", status.value());

        // 获取所有错误
        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(x -> x.getField() + ": " + x.getDefaultMessage())
                .collect(Collectors.toList());

        body.put("errors", errors);

        return new ResponseEntity<>(body, headers, status);
    }
}
```

- 当请求参数验证失败时，会返回一个包含详细错误信息的 JSON 响应。
- 错误信息格式为 `{"timestamp": "...", "status": 400, "errors": ["field: 错误信息"]}`。





