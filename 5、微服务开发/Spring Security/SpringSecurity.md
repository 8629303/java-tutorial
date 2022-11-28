> SpringSecurity & 源码：
>
> https://www.cnblogs.com/FondWang/category/1645775.html（流程+SpringBoot+H2）
>
> https://www.cnblogs.com/chiangchou/p/springboot-2.html
>
> [http://blog.ffspace.cn/categories/%E5%90%8E%E7%AB%AF/Spring-Security/](http://blog.ffspace.cn/categories/后端/Spring-Security/)
>
> 沈安心的客：https://blog.csdn.net/qq_22172133/category_8615344.html
>
> Jitwxs：https://blog.csdn.net/yuanlaijike/category_9283872.html
>
> http://www.tianshouzhi.com/api/tutorials/spring_security_4/265（田守真）
>
> [Delete丶94](https://blog.csdn.net/qq_42459181)
>
> https://blog.csdn.net/qq_42459181/category_8864199.html
>
> springsecurity核心要义（AuthenticationManager、ProviderManager）：
>
> https://blog.csdn.net/swordcenter/article/details/85455060
>
> https://www.cnblogs.com/xuwenjin/p/9565356.html
>
> SpringSecurity-登录认证原理：
>
> https://blog.csdn.net/abcwanglinyong/article/details/80981389
>
> https://www.cnblogs.com/zahfyy/p/9720124.html
>
> CSDN收藏：
>
> https://blog.csdn.net/u012702547/article/details/89629415
>
> https://blog.csdn.net/qq_42013035/article/details/106712074
>
> https://blog.csdn.net/qq_42013035/article/details/106681626（JJWT）
>
> SpringBoot+OAuth认证服务搭建：
>
> https://www.cnblogs.com/chiangchou/p/springboot-2.html
>
> https://blog.csdn.net/u014077048/article/details/83387695
>
> https://blog.csdn.net/WYA1993/article/details/85050120
>
> \------------------
>
> https://www.cnblogs.com/Irving/p/9343377.html
>
> https://www.jianshu.com/p/b273d53f1c27
>
> https://blog.csdn.net/w1054993544/article/details/78932614
>
> https://www.jianshu.com/p/c274f4acadb5
>
> https://www.cnblogs.com/Irving/p/9343377.html
>
> springsecurity核心要义（AuthenticationManager、ProviderManager）：
>
> https://blog.csdn.net/swordcenter/article/details/85455060
>
> https://www.cnblogs.com/xuwenjin/p/9565356.html
>
> SpringSecurity-登录认证原理：
>
> https://blog.csdn.net/abcwanglinyong/article/details/80981389
>
> https://www.cnblogs.com/zahfyy/p/9720124.html
>
> https://www.jianshu.com/p/a65f883de0c1
>
> https://www.jianshu.com/p/24c6a65c3913





## SpringSecurity 简介

一、 SpringSecurity 框架简介

**SpringSecurity** 是专门针对基于Spring项目的安全框架，充分利用了AOP和Filter来实现安全功能。它提供全面的安全性解决方案，同时在 Web 请求级和方法调用级处理身份确认和授权。他提供了强大的企业安全服务，如：认证授权机制、Web资源访问控制、业务方法调用访问控制、领域对象访问控制Access Control List（ACL）、单点登录（SSO）等等。

**核心功能**：认证（你是谁）、授权（你能干什么）、攻击防护（防止伪造身份）。

**基本原理**：SpringSecurity的核心实质是一个过滤器链，即一组Filter，所有的请求都会经过这些过滤器，然后响应返回。每个过滤器都有特定的职责，可通过配置添加、删除过滤器。过滤器的排序很重要，因为它们之间有依赖关系。有些过滤器也不能删除，如处在过滤器链最后几环的ExceptionTranslationFilter(处理后者抛出的异常)，FilterSecurityInterceptor(最后一环，根据配置决定请求能不能访问服务)。





二、比较Spring Security与Shiro

目前在java web应用安全框架中，与Spring Security形成直接竞争的就是shiro，二者在核心功能上几乎差不多，但从使用的角度各有优缺点。我认为：没有最好的，只有最合适的。



## 重写configure方法

```java

/**
     * anyRequest          |   匹配所有请求路径
     * access              |   SpringEl表达式结果为true时可以访问
     * anonymous           |   匿名可以访问
     * denyAll             |   用户不能访问
     * fullyAuthenticated  |   用户完全认证可以访问（非remember-me下自动登录）
     * hasAnyAuthority     |   如果有参数，参数表示权限，则其中任何一个权限可以访问
     * hasAnyRole          |   如果有参数，参数表示角色，则其中任何一个角色可以访问
     * hasAuthority        |   如果有参数，参数表示权限，则其权限可以访问
     * hasIpAddress        |   如果有参数，参数表示IP地址，如果用户IP和参数匹配，则可以访问
     * hasRole             |   如果有参数，参数表示角色，则其角色可以访问
     * permitAll           |   用户可以任意访问
     * rememberMe          |   允许通过remember-me登录的用户访问
     * authenticated       |   用户登录后可访问
     */
@Override
protected void configure(HttpSecurity httpSecurity) throws Exception
{

    httpSecurity
        //允许跨域
        .cors().and().anonymous().and()
        // CRSF禁用，因为不使用session
        .csrf().disable()
        // 认证失败处理类
        .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
        // 基于token，所以不需要session
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
        // 过滤请求
        .authorizeRequests()
        // 对于登录login 验证码captchaImage 允许匿名访问
        .antMatchers("/login", "/captchaImage").anonymous()
        .antMatchers(
        HttpMethod.GET,
        "/*.html",
        "/**/*.html",
        "/**/*.css",
        "/**/*.js"
    ).permitAll()
        .antMatchers("/profile/**").anonymous()
        .antMatchers("/common/download**").anonymous()
        .antMatchers("/common/download/resource**").anonymous()
        .antMatchers("/swagger-ui.html").anonymous()
        .antMatchers("/swagger-resources/**").anonymous()
        .antMatchers("/webjars/**").anonymous()
        .antMatchers("/*/api-docs").anonymous()
        .antMatchers("/druid/**").anonymous()
        // 除上面外的所有请求全部需要鉴权认证
        .anyRequest().authenticated()
        .and()
        .headers().frameOptions().disable();
    httpSecurity.logout().logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler);
    // 添加JWT filter
    httpSecurity.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

}
```



## 环境搭建

## 基于内存认证（Authentication）

## 请求授权配置（Authorization）

## 登录表单配置

## 自定义登录验证

## 注销登录



# 参考文献

- [free_java](https://blog.csdn.net/wangb_java/category_9284112.html)