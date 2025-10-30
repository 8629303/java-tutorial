> Springboot2.x 整合JWT：https://blog.csdn.net/leilei1366615/article/details/110275229
> JWT详解：https://blog.csdn.net/weixin_45070175/article/details/118559272
> SpringBoot + jwt 详解+使用案例：https://blog.csdn.net/zkcJava/article/details/119935284

# 1、Java 中使用 JWT

官网推荐了7个Java使用JWT的开源库，其中比较推荐使用的是：`java-jwt`和`jjwt-root`

## 1、java-jwt

### 1、引入依赖

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.3.0</version>
</dependency>
```

### 2、JWT生成令牌

我们构建JWT只需使用其`JWT.create()`方法，一直Build我们的参数即可：

- header：可不写，其有一个默认参数，当然您也可自行更改，详见其JWT.create()方法

- withClaim：方法即为我们的负载（payload）可多次build

- withExpiresAt：也是填充负载的一种方式

- sign：即我们的签名方式，起中班包含加密密钥,sign一定是要在最后一个build的

```java
import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import java.util.Date;
import java.util.HashMap;

public class Main {
    public static void main(String[] args) {
        // 密钥secret
        String JWT_USER_AUTH_SECRET = "!12ADAS";
        // 指定token过期时间为10秒
        Date expiresAt = new Date(System.currentTimeMillis() + 10000);

        String token = JWT.create()
                // Header
                .withHeader(new HashMap<>())
                // Payload
                .withClaim("userId", 18)
                .withClaim("userName", "baobao")
                .withExpiresAt(expiresAt)  // 过期时间
                // 签名用的secret
                .sign(Algorithm.HMAC256(JWT_USER_AUTH_SECRET));
        System.out.println(token);
    }
}
```

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyTmFtZSI6ImJhb2JhbyIsImV4cCI6MTY3ODg2MjgyNywidXNlcklkIjoxOH0.zBbWAiSkkPsaQC5vNsfHA9QqW33MSnpfSY0i85_VnIg
```

我们可以使用在线JWT解析，也可以使用接下来的代码验证。

### 3、JWT验证对象

我们服务器也可对JWT进行校验：

1. 只需要根据JWT构建时的密钥进行生成一个解析对象JWTVerifier
2. 然后将调用解析对象的verify方法，将我们的JWT传入进去，生成一个JWT解码对象
3. JWT解码对象调用不同的方法拿到对应的值（例如：getClaim("负载中某一字段名")，getClaims()拿到所有负载内容返回map）

```java
package com.xyz;

import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.DecodedJWT;
import com.auth0.jwt.interfaces.JWTVerifier;

import java.util.Date;
import java.util.HashMap;

public class Main {
    public static void main(String[] args) {
        verifyJWT(createJWT());
    }

    public static String createJWT() {
        // 密钥secret
        String JWT_USER_AUTH_SECRET = "!12ADAS";
        // 指定token过期时间为10秒
        Date expiresAt = new Date(System.currentTimeMillis() + 10000);

        String token = JWT.create()
                // Header
                .withHeader(new HashMap<>())
                // Payload
                .withClaim("userId", 18).withClaim("userName", "baobao").withExpiresAt(expiresAt)  // 过期时间
                // 签名用的secret
                .sign(Algorithm.HMAC256(JWT_USER_AUTH_SECRET));
        return token;
    }

    public static void verifyJWT(String token) {
        JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("!12ADAS")).build();
        // 使用JWT验证对象验证token,
        DecodedJWT decodedJWT = jwtVerifier.verify(token);
        System.out.println("标头:" + decodedJWT.getHeader());
        System.out.println("负载:" + decodedJWT.getPayload());
        System.out.println("签名:" + decodedJWT.getSignature());
        System.out.println("Token：" + decodedJWT.getToken());
        System.out.println("===============================");
        System.out.println("过期时间：" + decodedJWT.getExpiresAt());
        System.out.println("userId：" + decodedJWT.getClaim("userId"));
        System.out.println("username：" + decodedJWT.getClaim("userName"));
        System.out.println("负载中所有信息：" + decodedJWT.getClaims());
    }
}
```

```
标头:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
负载:eyJ1c2VyTmFtZSI6ImJhb2JhbyIsImV4cCI6MTY3ODg2NTYxMiwidXNlcklkIjoxOH0
签名:WBXAdttfQO5kfVXFz_JLqYrIHYp9eDT1Fz0VKKnmNx0
Token：eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyTmFtZSI6ImJhb2JhbyIsImV4cCI6MTY3ODg2NTYxMiwidXNlcklkIjoxOH0.WBXAdttfQO5kfVXFz_JLqYrIHYp9eDT1Fz0VKKnmNx0
===============================
过期时间：Wed Mar 15 15:33:32 CST 2023
userId：18
username："baobao"
负载中所有信息：{userName="baobao", exp=1678865612, userId=18}
```



## 2、jjwt-root

### 1、引入依赖

```xml
<!-- 老版本: 0.9.1是最后一个老版本-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

注意，现在jjwt已经更新到0.11.5，理论上说0.xx.x到0.xx.x的更新属于兼容性的，但经过尝试，发现还是有不少改动。这里也贴出来最新的版本的Maven工程依赖

```xml
<!--新版本-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

### 2、JWT工具类

新版本的jjwt中，之前的签名和验签方法都是传入密钥的字符串，已经过时。最新的方法需要传入`Key`对象

```java
public class JwtUtils {
    // token时效：24小时
    public static final long EXPIRE = 1000 * 60 * 60 * 24;
    // 签名哈希的密钥，对于不同的加密算法来说含义不同
    public static final String APP_SECRET = "ukc8BDbRigUDaY6pZFfWus2jZWLPHOsdadasdasfdssfeweee";

    /**
     * 根据用户id和昵称生成token
     * @param id  用户id
     * @param nickname 用户昵称
     * @return JWT规则生成的token
     */
    public static String getJwtToken(String id, String nickname){
        String JwtToken = Jwts.builder()
            .setSubject("baobao-user")
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + EXPIRE))
            .claim("id", id)
            .claim("nickname", nickname)
            // 传入Key对象
            .signWith(Keys.hmacShaKeyFor(APP_SECRET.getBytes(StandardCharsets.UTF_8)), SignatureAlgorithm.HS256)
            .compact();
        return JwtToken;
    }

    /**
     * 判断token是否存在与有效
     * @param jwtToken token字符串
     * @return 如果token有效返回true，否则返回false
     */
    public static Jws<Claims> decode(String jwtToken) {
        // 传入Key对象
        Jws<Claims> claimsJws = Jwts.parserBuilder().setSigningKey(Keys.hmacShaKeyFor(APP_SECRET.getBytes(StandardCharsets.UTF_8))).build().parseClaimsJws(jwtToken);
        return claimsJws;
    }
}
```



# 2、SpringBoot 整合 JWT

## 1、引入 JWT 依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.auth0/java-jwt -->
    <dependency>
        <groupId>com.auth0</groupId>
        <artifactId>java-jwt</artifactId>
        <version>4.3.0</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

## 2、创建JWT工具类

工具类中包含：创建 token，验证 token，获取用户 id 等

```java
import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTVerificationException;
import com.auth0.jwt.interfaces.DecodedJWT;
import com.auth0.jwt.interfaces.JWTVerifier;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.util.CollectionUtils;
import org.springframework.util.ObjectUtils;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Jwt工具类，生成JWT和认证
 */
@Slf4j
public class JwtUtil {

    /**
     * 密钥
     */
    private static final String SECRET = "xyz";

    /**
     * 过期时间（单位：秒）
     **/
    private static final long EXPIRATION = 3600L;

    /**
     * 生成用户token,设置token超时时间
     *
     * @param userId
     * @return
     */
    public static String createToken(Integer userId, String account) {
        Map<String, Object> map = new HashMap<>();
        map.put("alg", "HS256");
        map.put("typ", "JWT");
        String token = JWT.create()
                // 添加头部
                .withHeader(map)
                // 放入用户的id
                .withAudience(String.valueOf(userId))
                // 可以将基本信息放到claims中
                .withClaim("account", account)
                // 超时设置,设置过期的日期
                .withExpiresAt(new Date(System.currentTimeMillis() + EXPIRATION * 1000))
                // 签发时间
                .withIssuedAt(new Date())
                // SECRET加密
                .sign(Algorithm.HMAC256(SECRET));
        return token;
    }

    /**
     * 获取用户id
     */
    public static Integer getUserId(String token) {
        if (ObjectUtils.isEmpty(token)) {
            return null;
        }
        try {
            Algorithm algorithm = Algorithm.HMAC256(SECRET);
            JWTVerifier verifier = JWT.require(algorithm).build();
            DecodedJWT jwt = verifier.verify(token);
            if (null != jwt) {
                // 拿到我们放置在token中的信息
                List<String> audience = jwt.getAudience();
                if (!CollectionUtils.isEmpty(audience)) {
                    return Integer.parseInt(audience.get(0));
                }
            }
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (JWTVerificationException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 校验token并解析token
     */
    public static ResponseEntity verity(String token) {
        if (ObjectUtils.isEmpty(token)) {
            return new ResponseEntity<>("用户信息已过期，请重新登录", HttpStatus.UNAUTHORIZED);
        }
        try {
            Algorithm algorithm = Algorithm.HMAC256(SECRET);
            JWTVerifier verifier = JWT.require(algorithm).build();
            DecodedJWT jwt = verifier.verify(token);
            if (null != jwt) {
                // 拿到我们放置在token中的信息
                List<String> audience = jwt.getAudience();
                if (!CollectionUtils.isEmpty(audience)) {
                    return ResponseEntity.ok("认证成功");
                }
            }
        } catch (TokenExpiredException e) {
            // throw new TokenExpiredException("token is already expired", e.getExpiredOn());
            e.printStackTrace();
        } catch (AlgorithmMismatchException e) {
            // throw new AlgorithmMismatchException("签名算法不匹配");
            e.printStackTrace();
        } catch (InvalidClaimException e) {
            // throw new InvalidClaimException("校验的claims内容不匹配");
            e.printStackTrace();
        } catch (JWTVerificationException e) {
            e.printStackTrace();
        }
        return new ResponseEntity<>("用户信息已过期，请重新登录", HttpStatus.UNAUTHORIZED);
    }
}
```

我们可以看出 token 的创建用到了如下参数：

1. 算法：HS256

2. 类型：jwt
3. withAudience：向有效负载添加特定的受众（“aud”）声明，我们可以在这里放入一些用户的信息，例如：用户 id
4. withClaim：添加自定义索赔值，我们使用用户的账户和密码进行一起加密生成 jwt
5. withExpiresAt：超时时间设置，超时 token 将失效
6. withIssuedAt：签发时间，一般设置为当前时间
7. sign：签名，我们可以自定义签名和算法

JWT 的验证：

1. 首先我们从请求头中获取 token 信息，根据 key（Authorization）获取 value 值
2. 然后使用签名进行算法加密得到 jwt 的验证对象，JWTVerifier.verify(token) 用来验证 token 的正确性
3. 我们还可以从验证得到的 DecodedJWT 对象中获取我们创建 token 的时候放入的信息，例如：用户 id



## 3、自定义拦截器并注册

```java
import com.example.examplejavajwt.utils.JwtUtil;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 注册拦截规则
        registry.addInterceptor(new JwtTokenInterceptor())
                // 拦截路径，开放api请求的路径都拦截
                .addPathPatterns("/api/**")
                // 不拦截路径，如：注册、登录、忘记密码等
                .excludePathPatterns(
                        "/api/doRegister",
                        "/api/doLoginByAccount",
                        "/api/doLoginByPhone",
                        "/api/updatePasswordForget",
                        "/getToken"
                );
    }

    public static class JwtTokenInterceptor implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
            // 在拦截器中，如果请求为OPTIONS请求，则返回true，表示可以正常访问，然后就会收到真正的GET/POST请求
            if (HttpMethod.OPTIONS.toString().equals(request.getMethod())) {
                System.out.println("OPTIONS请求，放行");
                return true;
            }
            // 从请求头部中获取token信息
            String token = request.getHeader("Authorization");
            // 验证token
            ResponseEntity res = JwtUtil.verity(token);
            if (200 == res.getStatusCodeValue()) {
                return true;
            }
            // 验证不通过，返回401，表示用户未登录
            response.setStatus(401);
            return false;
        }
    }
}
```

我们通过注册我们自定义的拦截器，设置了拦截路径，以 api 开头的路径都会被验证 token 信息。我们还设置了不拦截路径，例如：注册、登录、忘记密码、获取Token等不需要用户登录就能直接请求的，就不需要进行验证 token 信息

此时，我们就完成了自定义拦截器对 token 信息进行验证，比起自定义注解，这种方式更加简单，方便，我们只需要关注拦截哪些路径，不拦截哪些路径即可



## 4、新建控制层测试

```java
import com.example.examplejavajwt.utils.JwtUtil;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
    @GetMapping("/getToken")
    public ResponseEntity createToken() {
        return ResponseEntity.ok(JwtUtil.createToken(110, "Sam"));
    }
    // 在header中写入Authorization
    @GetMapping("/api/test")
    public ResponseEntity test(){
        return ResponseEntity.ok("success");
    }
}
```

测试步骤：

1. 先调用/getToken API 获取Token（正常是通过调用登录接口）：localhost:8080/getToken
2. 然后调用/api/test API，在Header中设置Authorization参数即可

```bash
$ curl -X GET -s http://localhost:8080/getToken
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxMTAiLCJleHAiOjE2Nzg5NDEyOTMsImlhdCI6MTY3ODkzNzY5MywiYWNjb3VudCI6IlNhbSJ9.Fh1Yn-nth-gkOPEX_fPzBOlSOwCJ5pqaGML0SqqqhFU

$ curl -X GET -s -H 'Authorization:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxMTAiLCJleHAiOjE2Nzg5NDEyOTMsImlhdCI6MTY3ODkzNzY5MywiYWNjb3VudCI6IlNhbSJ9.Fh1Yn-nth-gkOPEX_fPzBOlSOwCJ5pqaGML0SqqqhFU' http://localhost:8080/api/test
success
```



## 5、增强版：新增注解

1、创建自定义注解

```java
import java.lang.annotation.*;
/**
 * 自定义注解 验证 token
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface JwtToken {
}
```

- @Target(ElementType.METHOD)，Target 说明了 Annotation 所修饰的对象范围，METHOD 用于描述方法
- @Retention(RetentionPolicy.RUNTIME)，运行时注解，注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在
- @Documented，元注解，表明这个注解应该被 javadoc 工具记录

2、自定义拦截器并注册

```java
import com.example.examplejavajwt.utils.JwtUtil;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.ResponseEntity;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 注册拦截规则
        registry.addInterceptor(new JwtTokenInterceptor()).addPathPatterns("/**");
    }

    public static class JwtTokenInterceptor implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
            JwtToken annotation;
            if (handler instanceof HandlerMethod) {
                annotation = ((HandlerMethod) handler).getMethodAnnotation(JwtToken.class);
            } else {
                return true;
            }
            // 如果没有该注解，直接放行
            if (annotation == null) {
                return true;
            }
            // 从请求头部中获取token信息
            String token = request.getHeader("Authorization");
            // 验证token
            ResponseEntity res = JwtUtil.verity(token);
            if (200 == res.getStatusCodeValue()) {
                return true;
            }
            // 验证不通过，返回401，表示用户未登录
            response.setStatus(401);
            return false;
        }
    }
}
```

3、控制层测试

```java
import com.example.examplejavajwt.config.JwtToken;
import com.example.examplejavajwt.utils.JwtUtil;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
    @GetMapping("/getToken")
    public ResponseEntity createToken() {
        return ResponseEntity.ok(JwtUtil.createToken(110, "Sam"));
    }
    // 在header中写入Authorization
    @JwtToken
    @GetMapping("/api/test")
    public ResponseEntity test(){
        return ResponseEntity.ok("success");
    }
}
```

```bash
$ curl -X GET -s http://localhost:8080/getToken
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxMTAiLCJleHAiOjE2Nzg5NDEyOTMsImlhdCI6MTY3ODkzNzY5MywiYWNjb3VudCI6IlNhbSJ9.Fh1Yn-nth-gkOPEX_fPzBOlSOwCJ5pqaGML0SqqqhFU

$ curl -X GET -s -H 'Authorization:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxMTAiLCJleHAiOjE2Nzg5NDEyOTMsImlhdCI6MTY3ODkzNzY5MywiYWNjb3VudCI6IlNhbSJ9.Fh1Yn-nth-gkOPEX_fPzBOlSOwCJ5pqaGML0SqqqhFU' http://localhost:8080/api/test
success
```



# 3、JWT 实现 Token 自动续期

> JWT 实现登录认证 + Token 自动续期方案：https://mp.weixin.qq.com/s/N-6CDwaj5k8Ahz_WAiZvyQ

## 1、技术选型

要实现认证功能，很容易就会想到JWT或者session，但是两者有啥区别？各自的优缺点？应该Pick谁？夺命三连

### 1、区别

基于Session和基于JWT的方式的主要区别就是用户的状态保存的位置，Session是保存在服务端的，而JWT是保存在客户端的



### 2、认证流程

1、基于Session的认证流程

- 用户在浏览器中输入用户名和密码，服务器通过密码校验后生成一个session并保存到数据库
- 服务器为用户生成一个sessionId，并将具有sesssionId的cookie放置在用户浏览器中，在后续的请求中都将带有这个cookie信息进行访问
- 服务器获取cookie，通过获取cookie中的sessionId查找数据库判断当前请求是否有效

2、基于JWT的认证流程

- 用户在浏览器中输入用户名和密码，服务器通过密码校验后生成一个token并保存到数据库
- 前端获取到token，存储到cookie或者local storage中，在后续的请求中都将带有这个token信息进行访问
- 服务器获取token值，通过查找数据库判断当前token是否有效



### 3、优缺点

JWT保存在客户端，在分布式环境下不需要做额外工作。而Session因为保存在服务端，分布式环境下需要实现多机数据共享 Session一般需要结合Cookie实现认证，所以需要浏览器支持Cookie，因此移动端无法使用Session认证方案。



### 4、安全性

JWT的payload使用的是Base64编码的，因此在JWT中不能存储敏感数据。而Session的信息是存在服务端的，相对来说更安全。如果在JWT中存储了敏感信息，可以解码出来非常的不安全。



### 5、性能

经过编码之后JWT将非常长，Cookie的限制大小一般是4k，Cookie很可能放不下，所以JWT一般放在local storage里面。并且用户在系统中的每一次http请求都会把JWT携带在Header里面，HTTP请求的Header可能比Body还要大。而SessionId只是很短的一个字符串，因此使用JWT的HTTP请求比使用Session的开销大得多



### 6、一次性

无状态是JWT的特点，但也导致了这个问题，JWT是一次性的。想修改里面的内容，就必须签发一个新的JWT



### 7、无法废弃

一旦签发一个JWT，在到期之前就会始终有效，无法中途废弃。若想废弃，一种常用的处理手段是结合Redis



### 8、续签

如果使用JWT做会话管理，传统的Cookie续签方案一般都是框架自带的，Session有效期30分钟，30分钟内如果有访问，有效期被刷新至30分钟。一样的道理，要改变JWT的有效时间，就要签发新的JWT。

最简单的一种方式是每次请求刷新JWT，即每个HTTP请求都返回一个新的JWT。这个方法不仅暴力不优雅，而且每次请求都要做JWT的加密解密，会带来性能问题。另一种方法是在Redis中单独为每个JWT设置过期时间，每次访问时刷新JWT的过期时间。



### 9、选择JWT 或 Session

我投JWT一票，JWT有很多缺点，但是在分布式环境下不需要像Session一样额外实现多机数据共享，虽然Seesion的多机数据共享可以通过粘性Session、Session共享、Session复制、持久化Session、terracoa实现Seesion复制等多种成熟的方案来解决这个问题。但是JWT不需要额外的工作，使用JWT不香吗？且JWT一次性的缺点可以结合redis进行弥补。

所以：扬长补短，因此在实际项目中选择的是使用JWT来进行认证



## 2、功能实现

### 1、JWT 所需依赖

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.3.0</version>
</dependency>
```

### 2、JWT 工具类

```java
public class JWTUtil {
    private static final Logger logger = LoggerFactory.getLogger(JWTUtil.class);
    // 私钥
    private static final String TOKEN_SECRET = "123456";

    /**
     * 生成token，自定义过期时间 毫秒
     *
     * @param userTokenDTO
     * @return
     */
    public static String generateToken(UserTokenDTO userTokenDTO) {
        try {
            // 私钥和加密算法
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
            // 设置头部信息
            Map<String, Object> header = new HashMap<>(2);
            header.put("Type", "Jwt");
            header.put("alg", "HS256");

            return JWT.create()
                .withHeader(header)
                .withClaim("token", new ObjectMapper().writeValueAsString(userTokenDTO))
                //.withExpiresAt(date) // 因为使用了Redis自动刷新时间
                .sign(algorithm);
        } catch (Exception e) {
            logger.error("generate token occur error, error is:{}", e);
            return null;
        }
    }

    /**
     * 检验token是否正确
     *
     * @param token
     * @return
     */
    public static UserTokenDTO parseToken(String token) {
        Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
        JWTVerifier verifier = JWT.require(algorithm).build();
        DecodedJWT jwt = verifier.verify(token);
        String tokenInfo = jwt.getClaim("token").asString();
        return new ObjectMapper.readValue(tokenInfo, UserTokenDTO.class);
    }
}
```

- 生成的token中不带有过期时间，token的过期时间由Redis进行管理
- UserTokenDTO中不带有敏感信息，如password字段不会出现在token中



### 3、Redis 工具类

对RedisTemplate的简单封装

```java
public final class RedisServiceImpl implements RedisService {
    /**
     * 过期时长
     */
    private final Long DURATION = 1 * 24 * 60 * 60 * 1000L;

    @Resource
    private RedisTemplate redisTemplate;

    private ValueOperations<String, String> valueOperations;

    @PostConstruct
    public void init() {
        RedisSerializer redisSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(redisSerializer);
        redisTemplate.setValueSerializer(redisSerializer);
        redisTemplate.setHashKeySerializer(redisSerializer);
        redisTemplate.setHashValueSerializer(redisSerializer);
        valueOperations = redisTemplate.opsForValue();
    }

    @Override
    public void set(String key, String value) {
        valueOperations.set(key, value, DURATION, TimeUnit.MILLISECONDS);
        log.info("key={}, value is: {} into redis cache", key, value);
    }

    @Override
    public String get(String key) {
        String redisValue = valueOperations.get(key);
        log.info("get from redis, value is: {}", redisValue);
        return redisValue;
    }

    @Override
    public boolean delete(String key) {
        boolean result = redisTemplate.delete(key);
        log.info("delete from redis, key is: {}", key);
        return result;
    }

    @Override
    public Long getExpireTime(String key) {
        return valueOperations.getOperations().getExpire(key);
    }
}
```



## 3、业务实现

### 1、登陆功能

```java
public String login(LoginUserVO loginUserVO) {
    // 1.判断用户名密码是否正确
    UserPO userPO = userMapper.getByUsername(loginUserVO.getUsername());
    if (userPO == null) {
        throw new UserException(ErrorCodeEnum.TNP1001001);
    }
    if (!loginUserVO.getPassword().equals(userPO.getPassword())) {
        throw new UserException(ErrorCodeEnum.TNP1001002);
    }

    // 2.用户名密码正确生成token
    UserTokenDTO userTokenDTO = new UserTokenDTO();
    PropertiesUtil.copyProperties(userTokenDTO, loginUserVO);
    userTokenDTO.setId(userPO.getId());
    userTokenDTO.setGmtCreate(System.currentTimeMillis());
    String token = JWTUtil.generateToken(userTokenDTO);

    // 3.存入token至redis
    redisService.set(userPO.getId(), token);
    return token;
}
```

- 判断用户名密码是否正确
- 用户名密码正确则生成token
- 将生成的token保存至redis



### 2、登出功能

```java
public boolean loginOut(String id) {
    boolean result = redisService.delete(id);
    if (!redisService.delete(id)) {
        throw new UserException(ErrorCodeEnum.TNP1001003);
    }
    return result;
}
```

- 将Redis对应的key删除即可





### 3、更新密码功能

```java
public String updatePassword(UpdatePasswordUserVO updatePasswordUserVO) {
    // 1.修改密码
    UserPO userPO = UserPO.builder()
        .password(updatePasswordUserVO.getPassword())
        .id(updatePasswordUserVO.getId())
        .build();
    UserPO user = userMapper.getById(updatePasswordUserVO.getId());
    if (user == null) {
        throw new UserException(ErrorCodeEnum.TNP1001001);
    }

    if (userMapper.updatePassword(userPO) != 1) {
        throw new UserException(ErrorCodeEnum.TNP1001005);
    }
    // 2.生成新的token
    UserTokenDTO userTokenDTO = UserTokenDTO.builder()
        .id(updatePasswordUserVO.getId())
        .username(user.getUsername())
        .gmtCreate(System.currentTimeMillis()).build();
    String token = JWTUtil.generateToken(userTokenDTO);
    // 3.更新token
    redisService.set(user.getId(), token);
    return token;
}
```

说明：

> 更新用户密码时需要重新生成新的token，并将新的token返回给前端，由前端更新保存在local storage中的token，同时更新存储在Redis中的token，这样实现可以避免用户重新登陆，用户体验感不至于太差。

其他说明

> 在实际项目中，用户分为普通用户和管理员用户，只有管理员用户拥有删除用户的权限，这一块功能也是涉及token操作的，这块就没有做Demo了。并且在实际项目中，密码传输是加密过的



### 4、拦截器类

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
    String authToken = request.getHeader("Authorization");
    String token = authToken.substring("Bearer".length() + 1).trim();
    UserTokenDTO userTokenDTO = JWTUtil.parseToken(token);
    // 1.判断请求是否有效
    if (redisService.get(userTokenDTO.getId()) == null 
            || !redisService.get(userTokenDTO.getId()).equals(token)) {
        return false;
    }

    // 2.判断是否需要续期
    if (redisService.getExpireTime(userTokenDTO.getId()) < 1 * 60 * 30) {
        redisService.set(userTokenDTO.getId(), token);
        log.error("update token info, id is:{}, user info is:{}", userTokenDTO.getId(), token);
    }
    return true;
}
```

拦截器中主要做两件事，一是对token进行校验，二是判断token是否需要进行续期

**Token校验：**

- 判断id对应的token是否不存在，不存在则token过期
- 若token存在则比较token是否一致，保证同一时间只有一个用户操作

**Token自动续期：**

- 为了不频繁操作Redis，只有当离过期时间只有30分钟时才更新过期时间



### 5、拦截器配置类

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthenticateInterceptor())
                .excludePathPatterns("/logout/**")
                .excludePathPatterns("/login/**")
                .addPathPatterns("/**");
    }
}
```



# 4、JWT Token 自动续签讨论

> JWT之token机制与双token详解：https://blog.csdn.net/qq_41634455/article/details/112591318

有网友说JWT的特性之一就是服务器不需要存储，只需校验token内容是否有效。如果需要服务器存储，就是把JWT当成session在用。也有说可以每次访问判断，当距离过期时间还剩15分钟就开始续签，也有使用双Token Refresh 的。个人认为根据项目情况而定。最简单的方式就是每次请求都刷新Token（这样只是增加了一些开销）

Token 过期自动续费方案总结：

1. 方案1：每一次请求都进行重新生成一个新的token【频率过高，性能不好】

2. 方案2：登录过后给前端进行返回token并设置了过期时间30分钟，每次请求的时候前端把token存在请求头里面进行发请求，后端接收请求的时候获取请求头出来进行jwt解析判断过期时间是否小于10分钟，如果小于10分钟就生成新的token在responseHearde进行返回即可

3. 方案3：每次登录的时候生成两个token给前端进行返回，一个是用于鉴别用户身份的access token，另外一个refresh token则是用于刷新token用的。

   双token检验流程：首先进行正常的登录操作，在后台服务器验证账号密码成功之后返回2个token：accessToken和refreshToken。在进行服务器请求的时候，先将Token发送验证，如果accessToken有效，则正常返回请求结果；如果accessToken无效，则验证refreshToken。此时如果refreshToken有效则返回请求结果和新的accessToken和新的refreshToken。如果refreshToken无效，则提示用户进行重新登陆操作。



# 5、JWT 双 Token 自动续签

JWT 的双 Token 自动续签机制是现代 Web 应用中保障用户体验与系统安全性的重要手段。下面为你详细解析其原理、优势，并给出完整的 Java 实现示例。



## 1、什么是双 Token 模式？

双 Token 模式指的是登录成功后，服务端签发两个 Token：

| **Token 类型**    | **存在形式** | **用途**              | **生命周期**     |
| ----------------- | ------------ | --------------------- | ---------------- |
| **Access Token**  | JWT 字符串   | 用于访问受保护的接口  | 短（如 15 分钟） |
| **Refresh Token** | JWT / UUID   | 用于刷新 Access Token | 长（如 7 天）    |



## 2、为什么使用双 Token？

安全性提升

- Access Token 过期时间短，即使泄露也影响较小；
- Refresh Token 存储在服务器或加密形式，不能用于访问业务接口。

用户体验好

- 用户在 Access Token 过期后，无需重新登录；
- 客户端可透明地通过 Refresh Token 获取新 Access Token。

可控性更强

- 可随时注销（失效）某个 Refresh Token，从而使该用户完全退出登录。



## 3、实现流程图

```
┌─────────────┐
│  用户登录    │
└────┬────────┘
     │
     ▼
服务端签发：AccessToken（短）+ RefreshToken（长）
     │
     ▼
客户端持 token 访问接口
     │
     ▼
┌───────────────────────────────────────────────┐
│ AccessToken 是否有效?                          │
│   └→ 是: 正常处理请求                           │
│   └→ 否: 尝试用 RefreshToken 获取新 AccessToken │
└───────────────────────────────────────────────┘
```



## 4、Java 实现示例

1、JWT 工具类（使用 io.jsonwebtoken）

```java
public class JwtUtils {

    private static final String SECRET = "my_secret_key";
    private static final long ACCESS_TOKEN_EXPIRE = 15 * 60 * 1000; // 15分钟
    private static final long REFRESH_TOKEN_EXPIRE = 7 * 24 * 60 * 60 * 1000L; // 7天

    // 生成 Access Token
    public static String createAccessToken(Long userId) {
        return Jwts.builder()
            .setSubject(userId.toString())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + ACCESS_TOKEN_EXPIRE))
            .signWith(SignatureAlgorithm.HS256, SECRET)
            .compact();
    }

    // 生成 Refresh Token
    public static String createRefreshToken(Long userId) {
        return Jwts.builder()
            .setSubject(userId.toString())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + REFRESH_TOKEN_EXPIRE))
            .signWith(SignatureAlgorithm.HS256, SECRET)
            .compact();
    }

    // 解析 Token
    public static Claims parseToken(String token) {
        return Jwts.parser()
            .setSigningKey(SECRET)
            .parseClaimsJws(token)
            .getBody();
    }

    // 是否过期
    public static boolean isExpired(String token) {
        try {
            return parseToken(token).getExpiration().before(new Date());
        } catch (Exception e) {
            return true;
        }
    }
}
```

2、登录接口示例

```java
@PostMapping("/login")
public R<Map<String, String>> login(@RequestBody LoginRequest req) {
    // 校验用户名密码（略）

    Long userId = 1L; // 示例

    Map<String, String> result = new HashMap<>();
    result.put("accessToken", JwtUtils.createAccessToken(userId));
    result.put("refreshToken", JwtUtils.createRefreshToken(userId));

    return R.ok(result);
}
```

3、刷新 Token 接口

```java
@PostMapping("/refresh-token")
public R<Map<String, String>> refresh(@RequestBody Map<String, String> req) {
    String refreshToken = req.get("refreshToken");

    if (JwtUtils.isExpired(refreshToken)) {
        return R.error("Refresh Token 已过期，请重新登录");
    }

    Claims claims = JwtUtils.parseToken(refreshToken);
    Long userId = Long.valueOf(claims.getSubject());

    Map<String, String> result = new HashMap<>();
    result.put("accessToken", JwtUtils.createAccessToken(userId));

    return R.ok(result);
}
```



## 5、前端配合逻辑（流程说明）

- 登录后保存 accessToken 和 refreshToken
- 调用接口时若遇到 401（token 失效），自动调用 /refresh-token 刷新
- 如果刷新成功，替换本地 accessToken 并重发原请求
- 若刷新也失败，则跳转登录页



## 6、补充建议

- Refresh Token 建议存在服务端（如 Redis）用于强制注销；
- 可记录 Refresh Token 的 jti（唯一ID）用于防止重放；
- 所有返回数据用统一响应体封装，方便前端判断异常处理。