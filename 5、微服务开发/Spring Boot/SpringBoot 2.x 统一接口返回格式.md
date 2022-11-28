> 参考文献：
>
> 1. SpringBoot 如何统一后端返回格式？老鸟们都是这样玩的！https://mp.weixin.qq.com/s/wLLATZqArx9Sl9xX8G0irQ
> 2. Spring Boot 无侵入式 实现API接口统一JSON格式返回，倍爽！https://mp.weixin.qq.com/s/0auGpd4RId1KaIU--xLZ_w
> 3. Spring Boot 中如何统一 API 接口响应格式？https://mp.weixin.qq.com/s/8aMz07rOF5LuclnBaI_p5g
> 4. Java项目构建基础：统一结果，统一异常，统一日志：https://mp.weixin.qq.com/s/VcNreeFhrMCCSYNosFBmzA

## 1、统一返回格式前言

在默认情况下，SpringBoot的返回格式常见的有三种：

**第一种：返回 String**

此时调用接口获取到的返回值是这样：

```java
@GetMapping("/hello")
public String getStr(){
    return "hello,javadaily";
}
```

```
hello,javadaily
```

**第二种：返回自定义对象**

```java
@GetMapping("/aniaml")
public Aniaml getAniaml(){
    Aniaml aniaml = new Aniaml(1,"pig");
    return aniaml;
}
```

```json
{
    "id": 1,
    "name": "pig"
}
```

**第三种：接口异常**

```java
@GetMapping("/wrong")
public int error(){
    int i = 9/0;
    return i;
}
```

```json
{
    "timestamp": "2021-07-08T08:05:15.423+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/wrong"
}
```

基于以上种种情况，如果你和前端开发人员联调接口她们就会很懵逼，由于我们没有给他一个统一的格式，前端人员不知道如何处理返回值。

还有甚者，有的同学比如小张喜欢对结果进行封装，他使用了Result对象，小王也喜欢对结果进行包装，但是他却使用的是Response对象，当出现这种情况时我相信前端人员一定会抓狂的。

所以我们项目中是需要定义一个统一的标准返回格式的。



## 2、定义返回标准格式

一个标准的返回格式至少包含3部分：

1. status 状态值：由后端统一定义各种返回结果的状态码

2. message 描述：本次接口调用的结果描述

3. data 数据：本次返回的数据

   ```json
   {
       "status":"100",
       "message":"操作成功",
       "data":"hello,javadaily"
   }
   ```

   当然也可以按需加入其他扩展值，比如我们就在返回对象中添加了接口调用时间

4. timestamp: 接口调用时间



### 1、定义返回对象

```java
@Data
public class ResultData<T> {
    /** 结果状态 **/
    private int status;
    /** 信息描述 **/
    private String message;
    /** 返回参数 **/
    private T data;
    /** 接口调用时间 **/
    private long timestamp;

    private ResultData() {
        this.timestamp = System.currentTimeMillis();
    }

    public static <T> ResultData<T> success(T data) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setStatus(ReturnCode.RC100.getCode());
        resultData.setMessage(ReturnCode.RC100.getMessage());
        resultData.setData(data);
        return resultData;
    }

    public static <T> ResultData<T> fail(int code, String message) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setStatus(code);
        resultData.setMessage(message);
        return resultData;
    }
}
```

### 2、定义状态码类

```java
public enum ReturnCode {
    /**操作成功**/
    RC100(100, "操作成功"),
    /**操作失败**/
    RC999(999, "操作失败"),
    /**服务限流**/
    RC200(200, "服务开启限流保护,请稍后再试!"),
    /**服务降级**/
    RC201(201, "服务开启降级保护,请稍后再试!"),
    /**热点参数限流**/
    RC202(202, "热点参数限流,请稍后再试!"),
    /**系统规则不满足**/
    RC203(203, "系统规则不满足要求,请稍后再试!"),
    /**授权规则不通过**/
    RC204(204, "授权规则不通过,请稍后再试!"),
    /**access_denied**/
    RC403(403, "无访问权限,请联系管理员授予权限"),
    /**access_denied**/
    RC401(401, "匿名用户访问无权限资源时的异常"),
    /**服务异常**/
    RC500(500, "系统异常，请稍后重试"),

    INVALID_TOKEN(2001, "访问令牌不合法"),
    ACCESS_DENIED(2003, "没有权限访问该资源"),
    CLIENT_AUTHENTICATION_FAILED(1001, "客户端认证失败"),
    USERNAME_OR_PASSWORD_ERROR(1002, "用户名或密码错误"),
    UNSUPPORTED_GRANT_TYPE(1003, "不支持的认证模式");

    /**自定义状态码**/
    private final int code;
    /**自定义描述**/
    private final String message;

    ReturnCode(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}
```

### 3、统一返回格式

此时调用接口获取到的返回值是这样：

```java
@GetMapping("/hello")
public ResultData<String> getStr(){
    return ResultData.success("hello,javadaily");
}
```

```json
{
    "status": 100,
    "message": "操作成功",
    "data": "hello,javadaily",
    "timestamp": 1667485675343
}
```

这样确实已经实现了我们想要的结果，我在很多项目中看到的都是这种写法，在Controller层通过`ResultData.success()`对返回结果进行包装后返回给前端。看到这里我们不妨停下来想想，这样做有什么弊端呢？

**最大的弊端就是我们后面每写一个接口都需要调用`ResultData.success()`这行代码对结果进行包装，重复劳动，浪费体力；而且还很容易被其他老鸟给嘲笑。**

所以呢我们需要对代码进行优化，目标就是不要每个接口都手工制定`ResultData`返回值。



### 4、高级实现方式1（全局）

要优化这段代码很简单，我们只需要借助 SpringBoot 提供的 ResponseBodyAdvice 

ResponseBodyAdvice 作用：拦截Controller方法的返回值，统一处理返回值/响应体，一般用来统一返回格式，加解密，签名等等

先来看下 ResponseBodyAdvice 的源码：

```java
public interface ResponseBodyAdvice<T> {
    /**
     * 是否支持advice功能
     * true 支持，false 不支持
     */
    boolean supports(MethodParameter var1, Class<? extends HttpMessageConverter<?>> var2);

    /**
     * 对返回的数据进行处理
     */
    @Nullable
    T beforeBodyWrite(@Nullable T var1, MethodParameter var2, MediaType var3, Class<? extends HttpMessageConverter<?>> var4, ServerHttpRequest var5, ServerHttpResponse var6);
}
```

我们只需要编写一个具体实现类即可

```java
@RestControllerAdvice
public class ResponseAdvice implements ResponseBodyAdvice<Object> {
    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
        // 这里也可以实现动态逻辑,例如:自定义注解,判断类或者方法是否使用了 @ResponseResultBody
        // AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ANNOTATION_TYPE) || returnType.hasMethodAnnotation(ANNOTATION_TYPE);
        return true;
    }

    @SneakyThrows
    @Override
    public Object beforeBodyWrite(Object o, MethodParameter methodParameter, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        if (o instanceof String) {
            return objectMapper.writeValueAsString(ResultData.success(o));
        }
        return ResultData.success(o);
    }
}
```

需要注意两个地方：

1、@RestControllerAdvice 注解：@RestControllerAdvice是@RestController注解的增强，可以实现三个方面的功能：


1. 全局异常处理
2. 全局数据绑定
3. 全局数据预处理

2、String类型判断

```java
if (o instanceof String) {
    return objectMapper.writeValueAsString(ResultData.success(o));
} 
```

这段代码一定要加，如果Controller直接返回String的话，SpringBoot是直接返回，故我们需要手动转换成json。

经过上面的处理我们就再也不需要通过`ResultData.success()`来进行转换了，直接返回原始数据格式，SpringBoot自动帮我们实现包装类的封装。此时我们调用接口返回的数据结果为：

```java
@GetMapping("/hello")
public String getStr(){
    return "hello,javadaily";
}
```

```json
{
    "status": 100,
    "message": "操作成功",
    "data": "hello,javadaily",
    "timestamp": 1667486355194
}
```

是不是感觉很完美，别急，还有个问题在等着你呢。



### 5、高级实现方式2（注解动态）

我们都知道使用@ResponseBody注解会把返回Object序列化成JSON字符串,就先从这个入手吧, 大致就是在序列化前把Object赋值给Result Object就可以了, 大家可以观摩org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice和org.springframework.web.bind.annotation.ResponseBody

**1、自定义 @ResponseBody 继承类**

我们已经决定从@ResponseBody注解入手了就创建一个注解类继承@ResponseBody, 很干净什么都没有哈哈,@ResponseResultBody 可以标记在类和方法上这样我们就可以跟自由的进行使用了

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@ResponseBody
public @interface ResponseResultBody {

}
```

2、**自定义 ResponseBodyAdvice 继承类**

```java
@RestControllerAdvice
public class ResponseResultBodyAdvice implements ResponseBodyAdvice<Object> {
    private static final Class<? extends Annotation> ANNOTATION_TYPE = ResponseResultBody.class;

    /**
     * 判断类或者方法是否使用了 @ResponseResultBody
     */
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ANNOTATION_TYPE) || returnType.hasMethodAnnotation(ANNOTATION_TYPE);
    }

    /**
     * 当类或者方法使用了 @ResponseResultBody 就会调用这个方法
     */
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        // 防止重复包裹的问题出现
        if (body instanceof Result) {
            return body;
        }
        return Result.success(body);
    }
}
```

**3、RestControllerAdvice返回测试**

```java
@RestController
@ResponseResultBody
public class HelloResultController {

    private static final HashMap<String, Object> INFO;
    static {
        INFO = new HashMap<String, Object>();
        INFO.put("name", "galaxy");
        INFO.put("age", "70");
    }

    @GetMapping("hello")
    public HashMap<String, Object> hello() {
        return INFO;
    }

    /** 测试重复包裹 */
    @GetMapping("result")
    public Result<Map<String, Object>> helloResult() {
        return Result.success(INFO);
    }

    @GetMapping("helloError")
    public HashMap<String, Object> helloError() throws Exception {
        throw new Exception("helloError");
    }

    @GetMapping("helloMyError")
    public HashMap<String, Object> helloMyError() throws Exception {
        throw new ResultException();
    }
}
```





### 6、接口异常问题

此时有个问题，由于我们没对Controller的异常进行处理，当我们调用的方法一旦出现异常，就会出现问题，比如下面这个接口

```java
@GetMapping("/wrong")
public int error(){
    int i = 9/0;
    return i;
}
```

```java
{
    "status": 100,
    "message": "操作成功",
    "data": {
        "timestamp": "2022-11-03T14:47:08Z",
        "status": 500,
        "error": "Internal Server Error",
        "path": "/wrong"
    },
    "timestamp": 1667486828991
}
```

这显然不是我们想要的结果，接口都报错了还返回操作成功的响应码。别急，接下来我们进入第二个议题，如何优雅的处理全局异常。



## 3、为何要全局异常处理器

### 1、使用全局异常处理器背景

**1、不用手写try...catch，由全局异常处理器统一捕获**

使用全局异常处理器最大的便利就是程序员在写代码时不再需要手写`try...catch`了，前面我们讲过，默认情况下SpringBoot出现异常时返回的结果是这样：

```json
{
    "timestamp": "2022-11-03T14:47:08Z",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/wrong"
}
```

这种数据格式返回给前端，前端是看不懂的，所以这时候我们一般通过`try...catch`来处理异常

```java
@GetMapping("/wrong")
public int error(){
    int i;
    try{
        i = 9/0;
    }catch (Exception e){
        log.error("error:{}",e);
        i = 0;
    }
    return i;
}
```

我们追求的目标肯定是不需要再手动写`try...catch`了，而是希望由全局异常处理器处理。

**2、对于自定义异常，只能通过全局异常处理器来处理**

```java
@GetMapping("error1")
public void empty(){
    throw  new RuntimeException("自定义异常");
}
```

3、当我们引入Validator参数校验器的时候，参数校验不通过会抛出异常，此时是无法用`try...catch`捕获的，只能使用全局异常处理器。

> SpringBoot集成参数校验请参考这篇文章[SpringBoot开发秘籍 - 集成参数校验及高阶技巧](http://mp.weixin.qq.com/s?__biz=MzAwMTk4NjM1MA==&mid=2247490888&idx=1&sn=dd7c3a4feb185abdca42362000b336d3&chksm=9ad00709ada78e1f04f0d878e03625083abccb5f72b6ae473c6816e2039edf245fe4ac6f50a6&scene=21#wechat_redirect)





### 2、如何实现全局异常处理器

```java
@Slf4j
@RestControllerAdvice
public class RestExceptionHandler {
    /**
     * 默认全局异常处理。
     * @param e the e
     * @return ResultData
     */
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResultData<String> exception(Exception e) {
        log.error("全局异常信息 ex={}", e.getMessage(), e);
        return ResultData.fail(ReturnCode.RC500.getCode(), e.getMessage());
    }
}
```

有三个细节需要说明一下：

1. @RestControllerAdvice：RestController的增强类，可用于实现全局异常处理器
2. @ExceptionHandler：统一处理某一类异常，从而减少代码重复率和复杂度，比如要获取自定义异常可以@ExceptionHandler(BusinessException.class)
3. @ResponseStatus指定客户端收到的http状态码

体验效果，这时候我们调用如下接口：

```java
@GetMapping("error1")
public void empty(){
    throw  new RuntimeException("自定义异常");
}
```

```json
{
    "status": 100,
    "message": "操作成功",
    "data": {
        "status": 500,
        "message": "自定义异常",
        "data": null,
        "timestamp": 1667487252256
    },
    "timestamp": 1667487252286
}
```

基本满足我们的需求了。但是当我们同时启用统一标准格式封装功能`ResponseAdvice`和`RestExceptionHandler`全局异常处理器时又出现了新的问题：**此时返回的结果是这样，统一格式增强功能会给返回的异常结果再次封装，所以接下来我们需要解决这个问题**。



### 3、全局异常接入返回的标准格式

要让全局异常接入标准格式很简单，因为全局异常处理器已经帮我们封装好了标准格式，我们只需要直接返回给客户端即可。

```java
@SneakyThrows
@Override
public Object beforeBodyWrite(Object o, MethodParameter methodParameter, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
    if(o instanceof String){
        return objectMapper.writeValueAsString(ResultData.success(o));
    }
    if(o instanceof ResultData){
        return o;
    }
    return ResultData.success(o);
}
```

关键代码：

```java
if(o instanceof ResultData){
    return o;
}
```

如果返回的结果是ResultData对象，直接返回即可。这时候我们再调用上面的错误方法，返回的结果就符合我们的要求了。

```java
{
    "status": 500,
    "message": "自定义异常",
    "data": null,
    "timestamp": 1625796580778
}
```

好了，今天的文章就到这里了，希望通过这篇文章你能掌握如何在你项目中友好实现统一标准格式到返回并且可以优雅的处理全局异常。

github地址：https://github.com/jianzh5/cloud-blog/



## ============分割线============





## 1、无侵入式 统一返回JSON格式

其实本没有没打算写这篇博客的，但还是要写一下写这篇博客的起因是因为，现在呆着的这家公司居然没有统一的API返回格式，询问主管他居然告诉我用HTTP状态码就够用了（fxxk），天哪HTTP状态码真的够用吗？在仔细的阅读了项目源码后发现，在API请求的是居然没有业务异常（黑人问好）。好吧 居然入坑了只能遵照项目风格了，懒得吐槽了。

因为项目已经开发了半年多了, 要是全部接口都做修改工作量还是挺大的, 只能用这种无侵入式的方案来解决.



## 2、定义接口返回的JSON格式

定义返回JSON格式，后端返回给前端一般情况下使用JSON格式, 定义如下

```json
{
    "code": 200, 
    "message": "OK", 
    "data": {}
}
```

- code: 返回状态码
- message: 返回信息的描述
- data: 返回值



## 3、定义JavaBean字段

**1、定义状态码枚举类**

```java
@ToString
@Getter
public enum ResultStatus {

    SUCCESS(HttpStatus.OK, 200, "OK"),
    BAD_REQUEST(HttpStatus.BAD_REQUEST, 400, "Bad Request"),
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, 500, "Internal Server Error"),;

    /** 返回的HTTP状态码,  符合http请求 */
    private HttpStatus httpStatus;
    /** 业务异常码 */
    private Integer code;
    /** 业务异常信息描述 */
    private String message;

    ResultStatus(HttpStatus httpStatus, Integer code, String message) {
        this.httpStatus = httpStatus;
        this.code = code;
        this.message = message;
    }
}
```

状态码和信息以及http状态码就能一一对应了便于维护, 有同学有疑问了为什么要用到http状态码呀,因为我要兼容项目以前的代码，没有其他原因, 当然其他同学不喜欢http状态码的可以吧源码中HttpStatus给删除了

**2、定义返回体类**

```java
@Getter
@ToString
public class Result<T> {
    /** 业务错误码 */
    private Integer code;
    /** 信息描述 */
    private String message;
    /** 返回参数 */
    private T data;

    private Result(ResultStatus resultStatus, T data) {
        this.code = resultStatus.getCode();
        this.message = resultStatus.getMessage();
        this.data = data;
    }

    /** 业务成功返回业务代码和描述信息 */
    public static Result<Void> success() {
        return new Result<Void>(ResultStatus.SUCCESS, null);
    }

    /** 业务成功返回业务代码,描述和返回的参数 */
    public static <T> Result<T> success(T data) {
        return new Result<T>(ResultStatus.SUCCESS, data);
    }

    /** 业务成功返回业务代码,描述和返回的参数 */
    public static <T> Result<T> success(ResultStatus resultStatus, T data) {
        if (resultStatus == null) {
            return success(data);
        }
        return new Result<T>(resultStatus, data);
    }

    /** 业务异常返回业务代码和描述信息 */
    public static <T> Result<T> failure() {
        return new Result<T>(ResultStatus.INTERNAL_SERVER_ERROR, null);
    }

    /** 业务异常返回业务代码,描述和返回的参数 */
    public static <T> Result<T> failure(ResultStatus resultStatus) {
        return failure(resultStatus, null);
    }

    /** 业务异常返回业务代码,描述和返回的参数 */
    public static <T> Result<T> failure(ResultStatus resultStatus, T data) {
        if (resultStatus == null) {
            return new Result<T>(ResultStatus.INTERNAL_SERVER_ERROR, null);
        }
        return new Result<T>(resultStatus, data);
    }
}
```

**3、Result实体返回测试**

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    private static final HashMap<String, Object> INFO;

    static {
        INFO = new HashMap<>();
        INFO.put("name", "galaxy");
        INFO.put("age", "70");
    }

    @GetMapping("/hello")
    public Map<String, Object> hello() {
        return INFO;
    }

    @GetMapping("/result")
    @ResponseBody
    public Result<Map<String, Object>> helloResult() {
        return Result.success(INFO);
    }
}
```

到这里我们已经简单的实现了统一JSON格式了, 但是我们也发现了一个问题了,想要返回统一的JSON格式需要返回Result Object才可以, 我明明返回Object可以了, 为什么要重复劳动, 有没有解决方法, 当然是有的啦, 下面我们开始优化我们的代码吧



## 4、统一返回格式进阶：全局处理(@RestControllerAdvice)

我们都知道使用@ResponseBody注解会把返回Object序列化成JSON字符串,就先从这个入手吧，大致就是在序列化前把Object赋值给Result Object就可以了, 大家可以观摩org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice和org.springframework.web.bind.annotation.ResponseBody

**1、@ResponseBody继承类**

我们已经决定从@ResponseBody注解入手了就创建一个注解类继承@ResponseBody, 很干净什么都没有哈哈,@ResponseResultBody 可以标记在类和方法上这样我们就可以跟自由的进行使用了

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@ResponseBody
public @interface ResponseResultBody {

}
```

**2、ResponseBodyAdvice继承类**

```java
@RestControllerAdvice
public class ResponseResultBodyAdvice implements ResponseBodyAdvice<Object> {

    private static final Class<? extends Annotation> ANNOTATION_TYPE = ResponseResultBody.class;

    /**
     * 判断类或者方法是否使用了 @ResponseResultBody
     */
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ANNOTATION_TYPE) || returnType.hasMethodAnnotation(ANNOTATION_TYPE);
    }

    /**
     * 当类或者方法使用了 @ResponseResultBody 就会调用这个方法
     */
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        // 防止重复包裹的问题出现
        if (body instanceof Result) {
            return body;
        }
        return Result.success(body);
    }
}
```

**3、RestControllerAdvice返回测试**

```java
@RestController
@RequestMapping("/helloResult")
@ResponseResultBody
public class HelloResultController {

    private static final HashMap<String, Object> INFO;

    static {
        INFO = new HashMap<String, Object>();
        INFO.put("name", "galaxy");
        INFO.put("age", "70");
    }

    @GetMapping("hello")
    public HashMap<String, Object> hello() {
        return INFO;
    }

    /** 测试重复包裹 */
    @GetMapping("result")
    public Result<Map<String, Object>> helloResult() {
        return Result.success(INFO);
    }

    @GetMapping("helloError")
    public HashMap<String, Object> helloError() throws Exception {
        throw new Exception("helloError");
    }

    @GetMapping("helloMyError")
    public HashMap<String, Object> helloMyError() throws Exception {
        throw new ResultException();
    }
}
```

是不是很神奇, 直接返回Object就可以统一JSON格式了, 就不用每个返回都返回Result对象了,直接让SpringMVC帮助我们进行统一的管理, 简直完美。只想看接口哦, helloError和helloMyError是会直接抛出异常的接口,我好像没有对异常返回进行统一的处理哦



## 5、统一返回格式进阶：异常处理(@ExceptionHandler)

卧槽, 异常处理, 差点把这茬给忘了, 这个异常处理就有很多方法了,先看看我师傅的处理方式, 我刚拿到这个代码的时候很想吐槽, 对异常类的处理这么残暴的吗, 直接用PrintWriter直接输出结果, 果然是老师傅, 我要是有100个异常类, 不得要写100个 if else了. 赶紧改改睡吧

```java
@Configuration
public class MyExceptionHandler implements HandlerExceptionResolver {

    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,
                                         Object handler, Exception ex) {
        PrintWriter out = getPrintWrite(response);
        if (ex instanceof XXXException) {
            out.write(JsonUtil.formatJson(ResultEnum.PAY_ERROR.getCode(), ex.getMessage()));
        } else {
            out.write(JsonUtil.formatJson(ResultEnum.FAIL.getCode(), "服务器异常"));
        }
        if (null != out) {
            out.close();
        }
        return mav;
    }

    private PrintWriter getPrintWrite(HttpServletResponse response) {
        PrintWriter out = null;
        try {
            response.setHeader("Content-type", "text/html;charset=UTF-8");
            response.setCharacterEncoding("UTF-8");
            out = response.getWriter();
        } catch (IOException e) {
            log.error("PrintWriter is exception", e);
        }
        return out;
    }
}
```

上面的代码看看还是没有问题的, 别学过去哦,

**异常处理@ResponseStatus(不推荐)**@ResponseStatus用法如下,可用在Controller类和Controller方法上以及Exception类上但是这样的工作量还是挺大的

```java
@RestController
@RequestMapping("/error")
@ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR, reason = "Java的异常")
public class HelloExceptionController {

    private static final HashMap<String, Object> INFO;

    static {
        INFO = new HashMap<String, Object>();
        INFO.put("name", "galaxy");
        INFO.put("age", "70");
    }

    @GetMapping()
    public HashMap<String, Object> helloError() throws Exception {
        throw new Exception("helloError");
    }

    @GetMapping("helloJavaError")
    @ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR, reason = "Java的异常")
    public HashMap<String, Object> helloJavaError() throws Exception {
        throw new Exception("helloError");
    }

    @GetMapping("helloMyError")
    public HashMap<String, Object> helloMyError() throws Exception {
        throw new MyException();
    }
}

@ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR, reason = "自己定义的异常")
class MyException extends Exception {

}
```

**全局异常处理@ExceptionHandler（推荐）**把ResponseResultBodyAdvice类进行改造一下,代码有点多了 主要参考了org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler#handleException()方法, 有空可以看一下

```java
@Slf4j
@RestControllerAdvice
public class ResponseResultBodyAdvice implements ResponseBodyAdvice<Object> {

    private static final Class<? extends Annotation> ANNOTATION_TYPE = ResponseResultBody.class;

    /** 判断类或者方法是否使用了 @ResponseResultBody */
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ANNOTATION_TYPE) || returnType.hasMethodAnnotation(ANNOTATION_TYPE);
    }

    /** 当类或者方法使用了 @ResponseResultBody 就会调用这个方法 */
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        if (body instanceof Result) {
            return body;
        }
        return Result.success(body);
    }


    /**
     * 提供对标准Spring MVC异常的处理
     *
     * @param ex      the target exception
     * @param request the current request
     */
    @ExceptionHandler(Exception.class)
    public final ResponseEntity<Result<?>> exceptionHandler(Exception ex, WebRequest request) {
        log.error("ExceptionHandler: {}", ex.getMessage());
        HttpHeaders headers = new HttpHeaders();
        if (ex instanceof ResultException) {
            return this.handleResultException((ResultException) ex, headers, request);
        }
        // TODO: 2019/10/05 galaxy 这里可以自定义其他的异常拦截
        return this.handleException(ex, headers, request);
    }

    /** 对ResultException类返回返回结果的处理 */
    protected ResponseEntity<Result<?>> handleResultException(ResultException ex, HttpHeaders headers, WebRequest request) {
        Result<?> body = Result.failure(ex.getResultStatus());
        HttpStatus status = ex.getResultStatus().getHttpStatus();
        return this.handleExceptionInternal(ex, body, headers, status, request);
    }

    /** 异常类的统一处理 */
    protected ResponseEntity<Result<?>> handleException(Exception ex, HttpHeaders headers, WebRequest request) {
        Result<?> body = Result.failure();
        HttpStatus status = HttpStatus.INTERNAL_SERVER_ERROR;
        return this.handleExceptionInternal(ex, body, headers, status, request);
    }

    /**
     * org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler#handleExceptionInternal(java.lang.Exception, java.lang.Object, org.springframework.http.HttpHeaders, org.springframework.http.HttpStatus, org.springframework.web.context.request.WebRequest)
     * <p>
     * A single place to customize the response body of all exception types.
     * <p>The default implementation sets the {@link WebUtils#ERROR_EXCEPTION_ATTRIBUTE}
     * request attribute and creates a {@link ResponseEntity} from the given
     * body, headers, and status.
     */
    protected ResponseEntity<Result<?>> handleExceptionInternal(
        Exception ex, Result<?> body, HttpHeaders headers, HttpStatus status, WebRequest request) {

        if (HttpStatus.INTERNAL_SERVER_ERROR.equals(status)) {
            request.setAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE, ex, WebRequest.SCOPE_REQUEST);
        }
        return new ResponseEntity<>(body, headers, status);
    }
}
```

> 参考文献：
>
> 1. https://www.toutiao.com/i6694404645827117572/ 
> 2. https://blog.csdn.net/qq_36722039/article/details/80825117 
> 3. http://www.imooc.com/article/260354 