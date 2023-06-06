# 1、JWT 是什么

官网地址：https://jwt.io/

1、官网译文：

JSON Web Token（JWT）是一个开放标准（rfc7519）；它定义了一种紧凑的、自包含的方式，用于在各方之间以JSON对象安全地传输信息。此信息可以验证和信任，因为它是数字签名的。jwt可以使用秘密（使用HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名

2、白话解释：

JWT是JSON Web Token的简称，是通过JSON形式作为Web应用中的令牌,用于在各方之间安全地将信息作为JSON对象传输。在数据传输过程中还可以完成对数据加密、签名等相关处理。



3、在介绍JWT之前，我们先来回顾一下利用tToken进行用户身份验证的流程：

1. 客户端使用用户名和密码请求登录

2. 服务端收到请求，验证用户名和密码
3. 验证成功后，服务端会签发一个token，再把这个token返回给客户端
4. 客户端收到token后可以把它存储起来，比如放到cookie中
5. 客户端每次向服务端请求资源时需要携带服务端签发的token，可以在cookie或者header中携带
6. 服务端收到请求，然后去验证客户端请求里面带着的token，如果验证成功，就向客户端返回请求数据

JWT 实际就是上述流程当中Token的一种具体实现方式，其全称是：JSON Web Token。

这种基于Token的认证方式相比传统的Session认证方式更节约服务器资源，并且对移动端和分布式更加友好。其优点如下：

1. 支持跨域访问：cookie是无法跨域的，而token由于没有用到cookie(前提是将token放到请求头中)，所以跨域后不会存在信息丢失问题

2. 无状态：token机制在服务端不需要存储session信息，因为token自身包含了所有登录用户的信息，所以可以减轻服务端压力
3. 更适用CDN：可以通过内容分发网络请求服务端的所有资料
4. 更适用于移动端：当客户端是非浏览器平台时，cookie是不被支持的，此时采用token认证方式会简单很多
5. 无需考虑CSRF：由于不再依赖cookie，所以采用token认证方式不会发生CSRF，所以也就无需考虑CSRF的防御



# 2、JWT 的作用

1、授权：授权是JWT在项目中使用最广的功能，例如我们在用户登录成功后，生成一个该用户的JWT令牌返回前端，前端对用户的后续请求中携带JWT，根据业务内容进行判断用户是否有访问服务和资源的权限。并且当今广泛使用单点登录功能也是要基于JWT的，因为它的开销很小并且可以在不同的域中轻松使用。

2、信息交换：JSON Web Token是在各方之间安全地传输信息的好方法。因为可以对JWT进行签名（例如：使用公钥/私钥对），所以您可以确保请求发起人是否合法；并且由于签名是使用标头和有效负载计算的，甚至能够验证您的请求内容是否被第三方拦截后进行了篡改。



# 3、为什么要使用 JWT

## 1、传统Session认证的弊端

我们知道HTTP本身是一种无状态的协议，这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，认证通过后HTTP协议不会记录下认证后的状态，那么下一次请求时，用户还要再一次进行认证，因为根据HTTP协议，我们并不知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在用户首次登录成功后，在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为Cookie，以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了，这是传统的基于Session认证的过程：

![img](SpringBoot 2.x JWT 概念篇.assets/29cfe2cc7bd13bc659227e62c3e89063.png)

传统的Session认证有如下的问题：

1. 每个用户的登录信息都会保存到服务器的Session中，随着用户的增多，服务器开销会明显增大
2. 由于Session是存在与服务器的物理内存中，所以在分布式系统中，这种方式将会失效。虽然可以将Session统一保存到Redis中，但是这样做无疑增加了系统的复杂性，对于不需要Redis的应用也会白白多引入一个缓存中间件
3. 对于非浏览器的客户端、手机移动端等不适用，因为Session依赖于Cookie，而移动端经常没有Cookie
4. 因为Session认证本质基于Cookie，所以如果Cookie被截获，用户很容易收到跨站请求伪造攻击。并且如果浏览器禁用了Cookie，这种方式也会失效
5. 前后端分离系统中更加不适用，后端部署复杂，前端发送的请求往往经过多个中间件到达后端，Cookie中关于Session的信息会转发多次
6. 由于基于Cookie，而Cookie无法跨域，所以Session的认证也无法跨域，对单点登录不适用。



## 2、JWT 认证的优势

对比传统的session认证方式，JWT的优势是：

1. 简洁：JWT Token数据量小，传输速度也很快
2. 因为JWT Token是以JSON加密形式保存在客户端的，所以JWT是跨语言的，原则上任何Web形式都支持
3. 不需要在服务端保存会话信息，也就是说不依赖于Cookie和Session，所以没有了传统Session认证的弊端，特别适用于分布式微服务
4. 单点登录友好：使用Session进行身份认证的话，由于Cookie无法跨域，难以实现单点登录。但是使用Token进行认证的话，Ttoken可以被保存在客户端的任意位置的内存中，不一定是Cookie，所以不依赖Cookie，不会存在这些问题
5. 适合移动端应用：使用Session进行身份认证的话，需要保存一份信息在服务器端，而且这种方式会依赖到Cookie（需要 Cookie 保存 SessionId），所以不适合移动端

# 4、JWT 认证流程

JWT 的本质就是一个字符串，它是将用户信息保存到一个JSON字符串中，然后进行编码后得到一个JWT Token，并且这个JWT Token带有签名信息，接收后可以校验是否被篡改，所以可以用于在各方之间安全地将信息作为JSON对象传输。JWT 的认证流程如下：

1. 第一步：前端通过Web表单将自己的用户名和密码发送到后端的接口，这个过程一般是一个POST请求。建议的方式是通过SSL加密的传输(HTTPS)，从而避免敏感信息被嗅探
2. 第二步：后端根据用户密码查询数据库校验账户合法性后，将包含用户信息的数据作为JWT的Payload，将其与JWT Header分别进行Base64编码拼接后签名，形成一个JWT Token，形成的JWT Token就是一个如同xxx.yyy.zzz的字符串，并返回给前端
3. 第三步：前端在每次请求时将JWT Token放入HTTP请求头中的Authorization属性中（解决XSS和XSRF问题）
4. 第四步：后端检查前端传过来的JWT Token，验证其有效性，比如检查签名是否正确、是否过期、Token的接收方是否是自己等等
5. 第五步：验证通过后，后端解析出JWT Token中包含的用户信息，进行其他逻辑操作(一般是根据用户信息得到权限等)，返回结果

**整个链路图示如下：**

![image-20201128152613148](SpringBoot 2.x JWT 概念篇.assets/ff4a8a124020944e87dea9e12939c8ef.png)


# 5、JWT 结构详解

JWT（JSON WEB TOKEN）共有三部分组成：标头.负载.签名（JWTString = header.payload.singnature），在传输的时候，会将JWT的三部分分别进行Base64编码后用`.`进行连接形成最终传输的字符串。

1. 标头（Header）
2. 有效载荷（Payload）
3. 签名（Signature）

JWT通常如下所示：xxxxx.yyyyy.zzzzz 三部分，即Header.Payload.Signature

```bash
# JWT硬编码
JWTString=Base64(Header).Base64(Payload).HMACSHA256(base64UrlEncode(header)+"."+base64UrlEncode(payload),secret)
# JWT字符串
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```



## 1、标头（Header）

JWT的第一部分是标头（Header），是一个描述JWT元数据的JSON对象，标头通常由两部分组成：

1. alg：签名使用的算法，默认为HMAC SHA256（写为HS256），或者可以使用RSA
2. typ：令牌的类型，JWT令牌统一写为JWT

| 字段名 | 类型   | 说明                                                         |
| ------ | ------ | ------------------------------------------------------------ |
| alg    | String | 签名使用的算法。如：HMAC SHA256缩写为 HS256                  |
| typ    | String | 声明 JWT 的媒体类型（IANA.MediaTypes）。一般固定赋值为`JWT`。 |

最后使用 Base64 编码组成 JWT 结构的第一部分。一般我们都会使用默认标头：

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```



## 2、有效载荷（Payload）

令牌的第二部分是有效负载（Payload），其中包含声明。声明是有关实体（通常是用户）和其他数据的声明。同样的，它会使用 Base64 编码组成 JWT 结构的第二部分。

这一部分是JWT的主体内容部分，也是一个JSON对象，包含需要传递的数据。 JWT指定七个默认字段供选择：

```
iss：发行人
exp：到期时间
sub：主题
aud：用户
nbf：在此之前不可用
iat：发布时间
jti：JWT ID用于标识该JWT
```

| 字段名 | 类型   | 说明                   |
| ------ | ------ | ---------------------- |
| iss    | String | 发行人，可填充应用标识 |
| exp    | Long   | 到期时间，是一个时间戳 |
| sub    | String | 主题，JWT面向的用户    |
| aud    | String | 用户，JWT的接收方      |
| nbf    | Long   | 在此时间前JWT不可用    |
| iat    | Long   | 发布时间，时间戳       |
| jti    | String | JWT ID，即JWT的标识ID  |

这些预定义的字段并不要求强制使用。除以上默认字段外，还可以自定义私有字段，一般会把包含用户信息的数据放到payload中：

```json
{
    "iss": "https://auth.brozen.top",
    "aud": "WeChatApp",
    "sub": "123123",
    "iat": 1655893632,
    "nbf": 1655893632,
    "jti": "testSessionId",
    "groups": ["WeChatApp"],
    "nonce": "testSessionId",
    "exp": 1655980032,
    "name": "Sam",
    "admin": true
}
```

> 注意：默认情况下JWT是未加密的，因为只是采用Base64算法，拿到JWT字符串后可以转换回原本的JSON数据，任何人都可以解读其内容，因此不要构建隐私信息字段，比如用户的密码一定不能保存到JWT中，以防止信息泄露。JWT只是适合在网络中传输一些非敏感的信息



## 3、签名（Signature）

令牌的第三部是签名（Signature），签名需要使用编码后的 header 和 payload 以及我们提供的一个密钥（该密码仅仅为保存在服务器中，并且不能向用户公开），然后使用 header 中指定的签名算法（HS256）进行签名。签名的作用是防止JWT篡改。

根据以下公式生成签名：

```javascript
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret);
```

**签名公式：加密算法（标头base64编码 + "." + 负载base64编码，密钥）**

签名目的：

最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。

在计算出签名哈希后，JWT头、有效载荷和签名哈希的三个部分组合成一个字符串，每个部分用`.`分隔，就构成整个JWT对象。



## 4、Base64URL

前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。



## 5、JWT 使用注意事项

注意JWT每部分的作用，在服务端接收到客户端发送过来的JWT Token之后：

1. header 和 payload 可以直接利用 Base64 解码出原文，从header中获取哈希签名的算法，从payload中获取有效数据
2. signature由于使用了不可逆的加密算法，无法解码出原文，它的作用是校验token有没有被篡改。服务端获取header中的加密算法之后，利用该算法加上secretKey对header、payload进行加密，比对加密后的数据和客户端发送过来的是否一致。注意secretKey只能保存在服务端，而且对于不同的加密算法其含义有所不同，一般对于MD5类型的摘要加密算法，secretKey实际上代表的是盐值



# 6、JWT 的种类

> 关于 JWT、JWS、JWE（**推荐**）：https://blog.csdn.net/u012503481/article/details/125982508

其实JWT（JSON Web Token）指的是一种规范，这种规范允许我们使用JWT在两个组织之间传递安全可靠的信息，JWT的具体实现可以分为以下几种：

- `nonsecure JWT`：未经过签名，不安全的JWT
- `JWS`：经过签名的JWT
- `JWE`：`payload`部分经过加密的JWT

## 1、nonsecure JWT

未经过签名，不安全的JWT。其`header`部分没有指定签名算法，并且也没有`Signature`部分。

```json
{
    "alg": "none",
    "typ": "JWT"
}
```



## 2、JWS

JWS ，也就 是JWT Signature，其结构就是在之前nonsecure JWT的基础上，在头部声明签名算法，并在最后添加上签名。**创建签名，是保证JWT不能被他人随意篡改**。我们通常使用的JWT一般都是JWS。

为了完成签名，除了用到 header 信息和 payload 信息外，还需要算法的密钥，也就是secretKey。加密的算法一般有2类：

- 对称加密：secretKey 指加密密钥，可以生成签名与验签

- 非对称加密：secretKey 指私钥，只用来生成签名，不能用来验签（验签用的是公钥）

JWT的密钥或者密钥对，一般统一称为JSON Web Key，也就是JWK

到目前为止，JWT的签名算法有三种：

- HMAC【哈希消息验证码(对称)】：HS256/HS384/HS512
- RSASSA【RSA签名算法(非对称)】（RS256/RS384/RS512）
- ECDSA【椭圆曲线数据签名算法(非对称)】（ES256/ES384/ES512）
