作者：fmwind；来源：https://blog.csdn.net/fmwind/article/details/81235401

作者：zhangpower1993；来源：https://blog.csdn.net/zhangpower1993/article/details/89016503

## 1、WebMvcConfigurer 简介

WebMvcConfigurer配置类其实是Spring内部的一种配置方式，采用JavaBean的形式来代替传统的xml配置文件形式进行针对框架个性化定制，可以自定义一些Handler，Interceptor，ViewResolver，MessageConverter。基于java-based方式的SpringMVC配置，需要创建一个**配置**类并实现**WebMvcConfigurer**接口；

在Spring Boot 1.5版本都是靠重写**WebMvcConfigurerAdapter**的方法来添加自定义拦截器，消息转换器等。SpringBoot 2.0 后，该类被标记为@Deprecated（弃用）。官方推荐直接实现WebMvcConfigurer或者直接继承WebMvcConfigurationSupport，方式一实现WebMvcConfigurer接口（推荐），方式二继承WebMvcConfigurationSupport类。



## 2、WebMvcConfigurer 接口

```java
public interface WebMvcConfigurer {
    void configurePathMatch(PathMatchConfigurer var1);
    void configureContentNegotiation(ContentNegotiationConfigurer var1);
    void configureAsyncSupport(AsyncSupportConfigurer var1);
    void configureDefaultServletHandling(DefaultServletHandlerConfigurer var1);
    void addFormatters(FormatterRegistry var1);
    void addInterceptors(InterceptorRegistry var1);
    void addResourceHandlers(ResourceHandlerRegistry var1);
    void addCorsMappings(CorsRegistry var1);
    void addViewControllers(ViewControllerRegistry var1);
    void configureViewResolvers(ViewResolverRegistry var1);
    void addArgumentResolvers(List<HandlerMethodArgumentResolver> var1);
    void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> var1);
    void configureMessageConverters(List<HttpMessageConverter<?>> var1);
    void extendMessageConverters(List<HttpMessageConverter<?>> var1);
    void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> var1);
    void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> var1);
    Validator getValidator();
    MessageCodesResolver getMessageCodesResolver();
}
```

常用的方法**：**

```java
/* HandlerMappings路径匹配选项，也就是可以自己定义路径的匹配规则 */
void configurePathMatch(PathMatchConfigurer var1);
/* 拦截器配置 */
void addInterceptors(InterceptorRegistry var1);
/* 视图跳转控制器 */
void addViewControllers(ViewControllerRegistry registry);
/* 静态资源处理 */
void addResourceHandlers(ResourceHandlerRegistry registry);
/* 默认静态资源处理器 */
void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer);
/* 这里配置视图解析器 */
void configureViewResolvers(ViewResolverRegistry registry);
/* 配置内容裁决的一些选项 */
void configureContentNegotiation(ContentNegotiationConfigurer configurer);
/* 解决跨域问题 */
public void addCorsMappings(CorsRegistry registry) ;
```

3、自定义实现

```java
@Configuration
public class MyWebMvcConfigurerAdapter implements WebMvcConfigurer {
    // ...重写想实现功能的方法
}
```



## 3、addInterceptors：拦截器

- addInterceptor：需要一个实现HandlerInterceptor接口的拦截器实例
- addPathPatterns：用于设置拦截器的过滤路径规则；`addPathPatterns("/**")`对所有请求都拦截
- excludePathPatterns：用于设置不需要拦截的过滤规则
- 拦截器主要用途：进行用户登录状态的拦截，日志的拦截等。

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    super.addInterceptors(registry);
    registry.addInterceptor(new TestInterceptor())
        .addPathPatterns("/**")
        .excludePathPatterns("/emp/toLogin","/emp/login","/js/**","/css/**","/images/**");
}
```



## 4、addViewControllers：页面跳转

以前写SpringMVC的时候，如果需要访问一个页面，必须要写Controller类，然后再写一个方法跳转到页面，感觉好麻烦，其实重写WebMvcConfigurer中的addViewControllers方法即可达到效果了

```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/toLogin").setViewName("login");
}
```

值的指出的是，在这里重写addViewControllers方法，并不会覆盖**WebMvcAutoConfiguration**（Springboot自动配置）中的addViewControllers（在此方法中，Spring Boot将“/”映射至index.html），这也就意味着自己的配置和SpringBoot的自动配置同时有效，这也是我们推荐添加自己的MVC配置的方式。



## 5、addResourceHandlers：静态资源

比如，我们想自定义静态资源映射目录的话，只需重写addResourceHandlers方法即可。注意：如果继承WebMvcConfigurationSupport类实现配置时必须要重写该方法，具体见其它文章。

```java
/**
 * 配置静态访问资源
 */
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/my/**").addResourceLocations("classpath:/my/");
}
```

- addResoureHandler：指的是对外暴露的访问路径
- addResourceLocations：指的是内部文件放置的目录



## 6、configureDefaultServletHandling：默认静态资源处理器

```java
@Override
public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
    configurer.enable("defaultServletName");
}
```

此时会注册一个默认的Handler：DefaultServletHttpRequestHandler，这个Handler也是用来处理静态文件的，它会尝试映射/。当DispatcherServelt映射/时（/ 和/ 是有区别的），并且没有找到合适的Handler来处理请求时，就会交给DefaultServletHttpRequestHandler 来处理。注意：这里的静态资源是放置在web根目录下，而非WEB-INF 下。

可能这里的描述有点不好懂（我自己也这么觉得），所以简单举个例子，例如：在webroot目录下有一个图片：1.png 我们知道Servelt规范中web根目录（webroot）下的文件可以直接访问的，但是由于DispatcherServlet配置了映射路径是：/ ，它几乎把所有的请求都拦截了，从而导致1.png 访问不到，这时注册一个DefaultServletHttpRequestHandler 就可以解决这个问题。其实可以理解为DispatcherServlet破坏了Servlet的一个特性（根目录下的文件可以直接访问），DefaultServletHttpRequestHandler是帮助回归这个特性的。



## 7、configureViewResolvers：视图解析器

这个方法是用来配置视图解析器的，该方法的参数ViewResolverRegistry 是一个注册器，用来注册你想自定义的视图解析器等。ViewResolverRegistry 常用的几个方法：https://blog.csdn.net/fmwind/article/details/81235401

```java
/**
 * 配置请求视图映射
 */
@Bean
public InternalResourceViewResolver resourceViewResolver() {
    InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
    // 请求视图文件的前缀地址
    internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
    // 请求视图文件的后缀
    internalResourceViewResolver.setSuffix(".jsp");
    return internalResourceViewResolver;
}

/**
 * 视图配置
 */
@Override
public void configureViewResolvers(ViewResolverRegistry registry) {
    super.configureViewResolvers(registry);
    registry.viewResolver(resourceViewResolver());
    /*registry.jsp("/WEB-INF/jsp/",".jsp");*/
}
```



## 8、configureContentNegotiation：配置内容裁决的一些参数

上面我们讲了configureViewResolvers 方法，假如在该方法中我们启用了内容裁决解析器，那么configureContentNegotiation(ContentNegotiationConfigurer configurer) 这个方法是专门用来配置内容裁决的一些参数的。这个比较简单，我们直接通过一个例子看：

```java
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    /* 是否通过请求Url的扩展名来决定media type */
    configurer.favorPathExtension(true)  
        /* 不检查Accept请求头 */
        .ignoreAcceptHeader(true)
        .parameterName("mediaType")
        /* 设置默认的media yype */
        .defaultContentType(MediaType.TEXT_HTML)
        /* 请求以.html结尾的会被当成MediaType.TEXT_HTML*/
        .mediaType("html", MediaType.TEXT_HTML)
        /* 请求以.json结尾的会被当成MediaType.APPLICATION_JSON*/
        .mediaType("json", MediaType.APPLICATION_JSON);
}
```

到这里我们就可以举个例子来进一步熟悉下我们上面讲的知识了，假如我们MVC的配置如下：

```java
@EnableWebMvc
@Configuration
public class MyWebMvcConfigurerAdapte extends WebMvcConfigurerAdapter {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/jsp/", ".jsp");
        registry.enableContentNegotiation(new MappingJackson2JsonView());
    }

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.favorPathExtension(true)
            .ignoreAcceptHeader(true)
            .parameterName("mediaType")
            .defaultContentType(MediaType.TEXT_HTML)
            .mediaType("html", MediaType.TEXT_HTML)
            .mediaType("json", MediaType.APPLICATION_JSON);
    }
}
```

controller的代码如下：

```java
@Controller
public class ExampleController {
    @RequestMapping("/test")
    public ModelAndView test() {
        Map<String, String> map = new HashMap();
        map.put("哈哈", "哈哈哈哈");
        map.put("呵呵", "呵呵呵呵");
        return new ModelAndView("test", map);
    }
}
```

在WEB-INF/jsp目录下创建一个test.jsp文件，内容随意。现在启动tomcat，在浏览器输入以下链接：http://localhost:8080/test.json，浏览器内容返回如下：

```json
{
    "哈哈":"哈哈哈哈",
    "呵呵":"呵呵呵呵"
}
```

在浏览器输入http://localhost:8080/test 或 http://localhost:8080/test.html，内容返回：this is test.jsp

显然，两次使用了不同的视图解析器，那么底层到底发生了什么？在配置里我们注册了两个视图解析器：
ContentNegotiatingViewResolver 和 InternalResourceViewResolver，还有一个默认视图：MappingJackson2JsonView。controller执行完毕之后返回一个ModelAndView，其中视图的名称为example1。

1. 返回首先会交给ContentNegotiatingViewResolver 进行视图解析处理，而ContentNegotiatingViewResolver 会先把视图名example1交给它持有的所有ViewResolver尝试进行解析（本实例中只有InternalResourceViewResolver），
2. 根据请求的mediaType，再将example1.mediaType（这里是example1.json 和example1.html）作为视图名让所有视图解析器解析一遍，两步解析完毕之后会获得一堆候选的List<View> 再加上默认的MappingJackson2JsonView ，
3. 根据请求的media type从候选的List<View> 中选择一个最佳的返回，至此视图解析完毕。

现在就可以理解上例中为何请求链接加上.json 和不.json 结果会不一样。当加上.json 时，表示请求的media type 为MediaType.APPLICATION_JSON，而InternalResourceViewResolver 解析出来的视图的ContentType与其不符，而与MappingJackson2JsonView 的ContentType相符，所以选择了MappingJackson2JsonView 作为视图返回。当不加.json 请求时，默认的media type 为MediaType.TEXT_HTML，所以就使用了InternalResourceViewResolver解析出来的视图作为返回值了。我想看到这里你已经大致可以自定义视图了。


## 9、addCorsMappings：跨域

```java
@Override
public void addCorsMappings(CorsRegistry registry) {
    super.addCorsMappings(registry);
    registry.addMapping("/cors/**")
        .allowedHeaders("*")
        .allowedMethods("POST","GET")
        .allowedOrigins("*");
}
```



## 10、configureMessageConverters：信息转换器

```java
/**
* 消息内容转换配置
 * 配置fastJson返回json转换
 * @param converters
 */
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    // 调用父类的配置
    super.configureMessageConverters(converters);
    // 创建fastJson消息转换器
    FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
    // 创建配置类
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    // 修改配置返回内容的过滤
    fastJsonConfig.setSerializerFeatures(
        SerializerFeature.DisableCircularReferenceDetect,
        SerializerFeature.WriteMapNullValue,
        SerializerFeature.WriteNullStringAsEmpty
    );
    fastConverter.setFastJsonConfig(fastJsonConfig);
    // 将fastjson添加到视图消息转换器列表内
    converters.add(fastConverter);
}
```



## 11、configurePathMatch：路径匹配规则

```java
@Override
public void configurePathMatch(PathMatchConfigurer configurer) {
    // 是否存在尾\\来进行匹配  /user和/user/等效的，同样可以进行匹配
    configurer.setUseTrailingSlashMatch(true);

    // 这个配置需要传入一个UrlPathHelper对象，UrlPathHelper是一个处理url地址的帮助类，
    // 他里面有一些优化url的方法
    // 比如：getSanitizedPath，就是将// 换成/  所以我们在输入地址栏的时候，//也是没有问题的，
    // 这里使用springmvc默认的就可以了，如果想要深入了解，那么我们后续在深入
    UrlPathHelper urlPathHelper = new UrlPathHelper();
    configurer.setUrlPathHelper(urlPathHelper);

    // 路径匹配器 PathMatcher是一个接口，springmvc默认使用的是AntPathMatcher
    // 这里也就不深入了，使用springmvc默认的就可以，如果想要深入了解，那么我们后续在深入，查看AntPathMatcher的源码
    // configurer.setPathMatcher();

    // 配置路径前缀
    // 下面这样写的意思是：对含有AdminController注解的controller添加/admin地址前缀
    configurer.addPathPrefix("admin", c -> c.isAnnotationPresent(AdminController.class));
    configurer.addPathPrefix("app", c -> c.isAnnotationPresent(AppController.class));
    // 当然这里也不一定非得用注解来实现：也可以用分包：
    // configurer.addPathPrefix("app", c -> c.getPackage().getName().contains("com.osy.controller.admin"));
    // configurer.addPathPrefix("app", c -> c.getPackage().getName().contains("com.osy.controller.app"));
}
```



