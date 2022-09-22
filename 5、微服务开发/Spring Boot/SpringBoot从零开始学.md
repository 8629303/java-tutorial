

# SpringBoot 中如何读取HTTP头信息？

> 1. 如何在Spring REST Controller中读取HTTP头信息？https://mp.weixin.qq.com/s/XegHYTV40bIimDgIFIIhGg
> 2. Http Header中到底有些啥？https://mp.weixin.qq.com/s/teQPo7HrqT5Yvnx8ZjzHsA
> 3. 你还在为 HTTP 的这些概念头疼吗？https://mp.weixin.qq.com/s/EAFIAr4NtmPF7tQ7rUJNvg



## 1、Header 参数说明

1、Http协议

HTTP（HyperTextTransferProtocol）  即超文本传输协议，目前网页传输的的通用协议。HTTP协议采用了请求/响应模型，浏览器或其他客户端发出请求，服务器给与响应。就整个网络资源传输而  言，包括message-header和message-body两部分。首先传递message- header，即 **http header**消息 **。**http header 消息通常被分为4个部分：general header, request  header, response header, entity header。但是这种分法就理解而言，感觉界限不太明确。根据维基百科对http  header内容的组织形式，大体分为Request和Response两部分。

2、Header示例整理如下表所示

| 参数                | 说明                                                         | 示例                                                         |
| :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Accept-Ranges       | 表明服务器是否支持指定范围请求及哪种类型的分段请求           | Accept-Ranges: bytes                                         |
| Age                 | 从原始服务器到代理缓存形成的估算时间（以秒计，非负）         | Age: 12                                                      |
| Allow               | 对某网络资源的有效的请求行为，不允许则返回405                | Allow: GET, HEAD                                             |
| Cache-Control       | 告诉所有的缓存机制是否可以缓存及哪种类型                     | Cache-Control: no-cache                                      |
| Content-Encoding    | web服务器支持的返回内容压缩编码类型。                        | Content-Encoding: gzip                                       |
| Content-Language    | 响应体的语言                                                 | Content-Language: en,zh                                      |
| Content-Length      | 响应体的长度                                                 | Content-Length: 348                                          |
| Content-Location    | 请求资源可替代的备用的另一地址                               | Content-Location: /index.htm                                 |
| Content-MD5         | 返回资源的MD5校验值                                          | Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==                        |
| Content-Range       | 在整个返回体中本部分的字节位置                               | Content-Range: bytes 21010-47021/47022                       |
| Content-Type        | 返回内容的MIME类型                                           | Content-Type: text/html; charset=utf-8                       |
| Date                | 原始服务器消息发出的时间                                     | Date: Tue, 15 Nov 2010 08:12:31 GMT                          |
| ETag                | 请求变量的实体标签的当前值                                   | ETag: “737060cd8c284d8af7ad3082f209582d”                     |
| Expires             | 响应过期的日期和时间                                         | Expires: Thu, 01 Dec 2010 16:00:00 GMT                       |
| Last-Modified       | 请求资源的最后修改时间                                       | Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT                 |
| Location            | 用来重定向接收方到非请求URL的位置来完成请求或标识新的资源    | Location: http://www.zcmhi.com/archives/94.html              |
| Pragma              | 包括实现特定的指令，它可应用到响应链上的任何接收方           | Pragma: no-cache                                             |
| Proxy-Authenticate  | 它指出认证方案和可应用到代理的该URL上的参数                  | Proxy-Authenticate: Basic                                    |
| refresh             | 应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持） | Refresh: 5; url=         http://www.zcmhi.com/archives/94.html |
| Retry-After         | 如果实体暂时不可取，通知客户端在指定时间之后再次尝试         | Retry-After: 120                                             |
| Server              | web服务器软件名称                                            | Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)                 |
| Set-Cookie          | 设置Http Cookie                                              | Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1          |
| Trailer             | 指出头域在分块传输编码的尾部存在                             | Trailer: Max-Forwards                                        |
| Transfer-Encoding   | 文件传输编码                                                 | Transfer-Encoding:chunked                                    |
| Vary                | 告诉下游代理是使用缓存响应还是从原始服务器请求               | Vary: *                                                      |
| Via                 | 告知代理客户端响应是通过哪里发送的                           | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)                  |
| Warning             | 警告实体可能存在的问题                                       | Warning: 199 Miscellaneous warning                           |
| WWW-Authenticate    | 表明客户端请求实体应该使用的授权方案                         | WWW-Authenticate: Basic                                      |
| Date                | 请求发送的日期和时间                                         | Date: Tue, 15 Nov 2010 08:12:31 GMT                          |
| Content-Type        | 请求的与实体对应的MIME信息                                   | Content-Type: application/x-www-form-urlencoded              |
| Expect              | 请求的特定的服务器行为                                       | Expect: 100-continue                                         |
| From                | 发出请求的用户的Email                                        | From: user@email.com                                         |
| Host                | 指定请求的服务器的域名和端口号                               | Host: www.zcmhi.com                                          |
| If-Match            | 只有请求内容与实体相匹配才有效                               | If-Match: “737060cd8c284d8af7ad3082f209582d”                 |
| If-Modified-Since   | 如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码 | If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT             |
| If-None-Match       | 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 | If-None-Match: “737060cd8c284d8af7ad3082f209582d”            |
| If-Range            | 如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag | If-Range: “737060cd8c284d8af7ad3082f209582d”                 |
| If-Unmodified-Since | 只在实体在指定时间之后未被修改才请求成功                     | If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT           |
| Max-Forwards        | 限制信息通过代理和网关传送的时间                             | Max-Forwards: 10                                             |
| Pragma              | 用来包含实现特定的指令                                       | Pragma: no-cache                                             |
| Proxy-Authorization | 连接到代理的授权证书                                         | Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==      |
| Range               | 只请求实体的一部分，指定范围                                 | Range: bytes=500-999                                         |
| Referer             | 先前网页的地址，当前请求网页紧随其后,即来路                  | Referer: http://www.zcmhi.com/archives/71.html               |
| TE                  | 客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息     | TE: trailers,deflate;q=0.5                                   |
| Upgrade             | 向服务器指定某种传输协议以便服务器进行转换（如果支持）       | Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11               |
| User-Agent          | User-Agent的内容包含发出请求的用户信息                       | User-Agent: Mozilla/5.0 (Linux; X11)                         |
| Via                 | 通知中间网关或代理服务器地址，通信协议                       | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)                  |
| Warning             | 关于消息实体的警告信息                                       | Warn: 199 Miscellaneous warning                              |
| Accept              | 指定客户端能够接收的内容类型                                 | Accept: text/plain, text/html                                |
| Accept-Charset      | 浏览器可以接受的字符编码集。                                 | Accept-Charset: iso-8859-5                                   |
| Accept-Encoding     | 指定浏览器可以支持的web服务器返回内容压缩编码类型。          | Accept-Encoding: compress, gzip                              |
| Accept-Language     | 浏览器可接受的语言                                           | Accept-Language: en,zh                                       |
| Accept-Ranges       | 可以请求网页实体的一个或者多个子范围字段                     | Accept-Ranges: bytes                                         |
| Authorization       | HTTP授权的授权证书                                           | Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==            |
| Cache-Control       | 指定请求和响应遵循的缓存机制                                 | Cache-Control: no-cache                                      |
| Connection          | 表示是否需要持久连接。（HTTP 1.1默认进行持久连接）           | Connection: close                                            |
| Cookie              | HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。 | Cookie: $Version=1; Skin=new;                                |
| Content-Length      | 请求的内容长度                                               | Content-Length: 348                                          |



## 1、简介

在这个快速教程中，我们将看看如何在Spring Rest Controller中访问HTTP头信息。

首先，我们将使用@RequestHeader注解来单独或一起读取头信息。之后，我们将深入了解@RequestHeader的属性。

## 2、访问HTTP头信息

### 2.1、获取单独头

如果我们需要访问特定的头，我们可以用头的名称配置@RequestHeader。

```java
@GetMapping("/greeting")
public ResponseEntity<String> greeting(@RequestHeader("accept-language") String language) {
    // code that uses the language variable
    return new ResponseEntity<String>(greeting, HttpStatus.OK);
}
```

然后，我们可以使用传递到我们方法中的变量访问该值。如果在请求中没有找到名为accept-language的头，方法就会返回一个 "400 Bad Request "错误。

我们的头不一定非得是字符串。例如，如果我们知道我们的头是一个数字，我们可以将我们的变量声明为数字类型。

```java
@GetMapping("/double")
public ResponseEntity<String> doubleNumber(@RequestHeader("my-number") int myNumber) {
    return new ResponseEntity<String>(String.format("%d * 2 = %d", myNumber, (myNumber * 2)), HttpStatus.OK);
}
```



### 2.2、获取所有头

如果我们不确定哪些头会出现，或者我们需要的头比我们的方法签名中需要的多，我们可以使用@RequestHeader注解，而不需要一个特定的名称。

我们的变量类型有几种选择：Map、MultiValueMap、HttpHeaders 对象

首先，让我们以Map的形式获取请求头。

```java
@GetMapping("/listHeaders")
public ResponseEntity<String> listAllHeaders(@RequestHeader Map<String, String> headers) {
    headers.forEach((key, value) -> {
        log.info(String.format("Header '%s' = %s", key, value));
    });
    return new ResponseEntity<String>(String.format("Listed %d headers", headers.size()), HttpStatus.OK);
}
```

如果我们使用一个Map，并且其中一个头有多个值，我们将只得到第一个值。这相当于在一个MultiValueMap上使用getFirst方法。

如果我们的headers可能有多个值，我们可以以MultiValueMap的方式获取它们。

```java
@GetMapping("/multiValue")
public ResponseEntity<String> multiValue(@RequestHeader MultiValueMap<String, String> headers) {
    headers.forEach((key, value) -> {
        log.info(String.format("Header '%s' = %s", key, value.stream().collect(Collectors.joining("|"))));
    });
    return new ResponseEntity<String>(String.format("Listed %d headers", headers.size()), HttpStatus.OK);
}
```

我们也可以以HttpHeaders对象的形式获得我们的头信息。

```java
@GetMapping("/getBaseUrl")
public ResponseEntity<String> getBaseUrl(@RequestHeader HttpHeaders headers) {
    InetSocketAddress host = headers.getHost();
    String url = "http://" + host.getHostName() + ":" + host.getPort();
    return new ResponseEntity<String>(String.format("Base URL = %s", url), HttpStatus.OK);
}
```

HttpHeaders对象有常见应用头的访问器。

当我们从Map、MultiValueMap或HttpHeaders对象中通过名称访问一个头时，如果它不存在，我们将得到一个null。



## 3、HTTP头注解属性

现在我们已经介绍了使用@RequestHeader注解访问请求头的基础知识，让我们仔细看看它的属性。

当我们为我们的头文件专门命名时，我们已经隐式地使用了名称或值属性。

```java
public ResponseEntity<String> greeting(@RequestHeader("accept-language") String language) {}
```

我们可以通过使用名称属性来实现同样的目的。

```java
public ResponseEntity<String> greeting(\@RequestHeader(name = "accept-language") String language) {}
```

接下来，让我们以完全相同的方式使用值属性。

```java
public ResponseEntity<String> greeting(@RequestHeader(value = "accept-language") String language) {}
```

当我们特别命名一个头时，默认情况下，头是必需的。如果在请求中没有找到头，控制器就会返回一个400错误。

让我们使用required属性来表示我们的头不是必需的。

```java
@GetMapping("/nonRequiredHeader")
public ResponseEntity<String> evaluateNonRequiredHeader(
    @RequestHeader(value = "optional-header", required = false) String optionalHeader) {
    return new ResponseEntity<String>(String.format(
        "Was the optional header present? %s!",
        (optionalHeader == null ? "No" : "Yes")), HttpStatus.OK);
}
```

由于如果请求中不存在头，我们的变量将为空，因此我们需要确保进行适当的空检查。

让我们使用defaultValue属性来为我们的头提供一个默认值。

```java
@GetMapping("/default")
public ResponseEntity<String> evaluateDefaultHeaderValue(
    @RequestHeader(value = "optional-header", defaultValue = "3600") int optionalHeader) {
    return new ResponseEntity<String>(String.format("Optional Header is %d", optionalHeader), HttpStatus.OK);
}
```







# Spring Boot Actuator 服务监控

> [SpringBoot 服务监控，健康检查，线程信息，JVM堆信息，指标收集，运行情况监控等！](https://mp.weixin.qq.com/s/J6mWLHKo2jvJs-KObSJ2aw)

# Spring Boot 集成 JUnit5

> [Spring Boot 集成 JUnit5，更优雅单元测试！ (qq.com)](https://mp.weixin.qq.com/s/w_oECKcRLpBoKxT1uM7NMA)



# SpringBoot 写入导出 PDF、EXCEL 

> [JAVA实现PDF、EXCEL生成和数据动态插入以及导出](https://mp.weixin.qq.com/s/3SOARI5A_1EOlDAN6cjZ0g)