# SpringBoot API 日志记录

> Spring Boot 3实战案例详解：全方位掌握API日志记录：https://baijiahao.baidu.com/s?id=1834473981973589810&wfr=spider&for=pc
>

本篇笔记涵盖在 Spring Boot 3 项目中全方位记录 HTTP 请求与响应日志的多种方式，特别强调：

- 请求与响应体的完整记录
- 日志脱敏处理
- TraceId 链路追踪集成
- 响应体读取的安全处理（避免多次读取）



## 1. 使用 Filter 记录日志

1、配置类注册 Filter

```java
@Configuration
public class LogFilterConfig {
    @Bean
    public FilterRegistrationBean<LogFilter> logFilter() {
        FilterRegistrationBean<LogFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new LogFilter());
        registrationBean.addUrlPatterns("/*");
        registrationBean.setName("logFilter");
        registrationBean.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return registrationBean;
    }
}
```

2、日志过滤器实现

```java
@Slf4j
public class LogFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        // 响应体记录（基于 ContentCachingResponseWrapper，确保只读取一次）；
        ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(req);
        ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper((HttpServletResponse) response);

        String traceId = UUID.randomUUID().toString();
        MDC.put("traceId", traceId);

        chain.doFilter(requestWrapper, responseWrapper);

        log.info("[Filter] [{}] {} Headers: {} Body: {} Response: {}",
                req.getMethod(),
                req.getRequestURI(),
                getHeaders(req),
                getRequestBody(requestWrapper),
                getResponseBody(responseWrapper));

        responseWrapper.copyBodyToResponse(); // 确保响应输出一次
        MDC.remove("traceId");
    }

    private String getRequestBody(ContentCachingRequestWrapper requestWrapper) {
        try {
            return new String(requestWrapper.getContentAsByteArray(), requestWrapper.getCharacterEncoding());
        } catch (Exception e) {
            return "[UNREADABLE]";
        }
    }

    private String getResponseBody(ContentCachingResponseWrapper responseWrapper) {
        try {
            return new String(responseWrapper.getContentAsByteArray(), responseWrapper.getCharacterEncoding());
        } catch (Exception e) {
            return "[UNREADABLE]";
        }
    }

    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> headers = new HashMap<>();
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            headers.put(name, request.getHeader(name));
        }
        return headers;
    }
}
```





## 2. 使用 HandlerInterceptor 拦截日志

1、拦截器实现

```java
@Slf4j
@Component
public class LogInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        MDC.put("traceId", UUID.randomUUID().toString());
        log.info("[Interceptor] [{}] {} Headers: {} Params: {}",
                request.getMethod(),
                request.getRequestURI(),
                getHeaders(request),
                request.getParameterMap());
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        MDC.remove("traceId");
    }

    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> headers = new HashMap<>();
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            headers.put(name, request.getHeader(name));
        }
        return headers;
    }
}
```

2、使用  WebMvcConfigurer 注册：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private LogInterceptor logInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(logInterceptor);
    }
}
```



## 3. 使用 AOP 切面记录日志

1、自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable {
    String value() default "";
}
```

2、切面实现

```java
@Slf4j
@Aspect
@Component
public class LogAspect {

    @Pointcut("@annotation(loggable)")
    public void pointcut(Loggable loggable) {}

    @Around("pointcut(loggable)")
    public Object around(ProceedingJoinPoint joinPoint, Loggable loggable) throws Throwable {
        ServletRequestAttributes attrs = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attrs.getRequest();
        MDC.put("traceId", UUID.randomUUID().toString());

        log.info("[AOP] {} {} Headers: {} Args: {} Annotation: {}",
                request.getMethod(),
                request.getRequestURI(),
                getHeaders(request),
                Arrays.toString(joinPoint.getArgs()),
                loggable.value());

        Object result = joinPoint.proceed();
        log.info("[AOP] Response: {}", maskSensitiveData(result));

        MDC.remove("traceId");
        return result;
    }

    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> headers = new HashMap<>();
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            headers.put(name, request.getHeader(name));
        }
        return headers;
    }

    private Object maskSensitiveData(Object response) {
        if (response instanceof String str && str.contains("password")) {
            return str.replaceAll("\\"password\\":\\".*?\\"", "\"password\":\"***\"");
        }
        return response;
    }
}
```

3、或者直接记录指定包下所有接口请求和响应

```java
@Aspect
@Component
public class ApiLogAspect {

    private static final Logger log = LoggerFactory.getLogger(ApiLogAspect.class);

    @Pointcut("execution(* com.example.demo.controller..*.*(..))")
    public void apiLogPointcut() {}

    @Around("apiLogPointcut()")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        String method = joinPoint.getSignature().toShortString();
        log.info("[AOP] Entering: {}", method);
        Object result = joinPoint.proceed();
        log.info("[AOP] Exiting: {}", method);
        return result;
    }
}
```





## 4. ServletRequestHandledEvent 监听

```java
@Slf4j
@Component
public class ServletRequestLogListener implements ApplicationListener<ServletRequestHandledEvent> {
    @Override
    public void onApplicationEvent(ServletRequestHandledEvent event) {
        log.info("[Event] [{}] {} Duration={}ms TraceId={}",
                event.getMethod(),
                event.getRequestUrl(),
                event.getProcessingTimeMillis(),
                MDC.get("traceId"));
    }
}
```

从 Spring 官方文档和类结构看，`ServletRequestHandledEvent` 主要提供**请求处理完成后的 “元数据”** ，核心方法 / 信息包括：

- getMethod()：HTTP 方法（如 GET/POST ）。
- getRequestUrl()：请求 URL。
- getProcessingTimeMillis()：处理耗时。
- getClientAddress()：客户端 IP 等。

这些信息是**请求的 “上下文结果”** ，但不包含请求参数、请求体这类**请求内容级数据** ，因为设计初衷是 “记录请求生命周期事件”，而非 “完整请求内容”。



## 5. 使用 Logbook 第三方组件

1、Maven 依赖

```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>logbook-spring-boot-starter</artifactId>
    <version>3.3.0</version>
</dependency>
```

2、application.yml 配置（含注释）

```yaml
logbook:
  format.style: http  # 输出格式，可选：http, json
  include:
    - request
    - response
  obfuscate:
    headers:
      - Authorization  # 关键字段脱敏
      - Cookie
    parameters:
      - password
  filter.enabled: true
```

自动记录所有 HTTP 请求响应，包括路径、方法、状态码、头部、body，支持脱敏字段配置。





## 6. 自定义 HandlerMethod 记录日志

```java
@Slf4j
@Component
public class LogHandlerMapping extends RequestMappingHandlerAdapter {
    @Override
    protected void handleInternal(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) {
        log.info("[HandlerMethod] {} {} Headers: {} Params: {} Handler: {}",
                request.getMethod(),
                request.getRequestURI(),
                getHeaders(request),
                request.getParameterMap(),
                handlerMethod.getMethod().getName());
    }

    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> headers = new HashMap<>();
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            headers.put(name, request.getHeader(name));
        }
        return headers;
    }
}
```

---

**附加建议**

- **链路追踪 TraceId**：推荐使用 [MDC](https://logback.qos.ch/manual/mdc.html) 搭配日志格式输出 %X{traceId}，配合 Nginx / Gateway 做端到端追踪。
- **响应体读取**：务必使用 ContentCachingResponseWrapper 包装响应，避免 getInputStream() 被提前消费。
- **脱敏处理建议**：在 AOP 层或 Logbook 配置中统一处理 password、token、phone 等敏感字段。