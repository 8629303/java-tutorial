## 1、API 管理系统介绍

> 前后端都分离了，该搞个好用的API管理系统了！
>
> 参考文献：https://mp.weixin.qq.com/s/Ahs6fnIfFVVPOn3NZpIsNA

**Swagger**： https://swagger.io/

**YApi**：http://yapi.demo.qunar.com/

**eolinker**：https://www.eolinker.com/

**易文档**：https://easydoc.xyz/



## 2、Swagger 3

### 1、OpenAPI 规范

接口文档

- 谁产生：接口开发人员，后端工程师
- 谁维护：接口开发人员，后端工程师
- 谁使用：前端同学、测试同学、产品经理

接口存在的问题：

- 接口文档不存在，靠抓包获取
- 接口更换后不及时更新
- 接口文档写错，注解写错
- 自动生成文档工具在跨语言不兼容



**OpenApi**规范：声明了用于文档的规范的版本

- 地址：https://github.com/OAI/OpenAPI-Specification

```
OpenAPI规范经过Reverb Technologies和SmartBear等公司多年的发展，OpenAPI计划拥有该规范（捐赠之后），OpenAPI Initiative在GitHub上托管社区驱动的规范。

规范是一种与语言无关的格式，用于描述RESTful Web服务，应用程序可以解释生成的文件，这样才能生成代码、生成文档并根据其描述的服务创建模拟应用。

开放API规范（OAS）是一种无需编写实际API代码就可以记录API的方法。 这是一种开放源代码格式，可以用来描述API。 在此过程中，我们可以使用JSON或YAML格式。
```

**OpenAPI**文档有三个必需的部分或对象，也可以增加其他模块：

1. openapi - OpenAPI规范版本的语义版本号
2. info - 有关API的元数据
3. paths - API的可用路径和操作



### 2、API 文档管理工具对比

**ApiDoc**

- 地址：https://apidocjs.com/
- github: https://github.com/apidoc/apidoc
- 简介：源代码中的注释直接自动生成api接口文档的工具

- 优点：不入侵代码、支持跨语言使用、界面友好简洁

- 缺点：依赖环境 node/npm

- 示例：在代码里面增加注释使用

  ```java
      /**
       * @apiGroup Product
       * @api {GET} /product/{id}  查询一个产品
       * @apiDescription 接口描述xxx
       * @apiParam {String} id 产品id(必填*)
       * @apiSuccessExample SuccessExample
       * HTTP/1.1 200
       * {
       * id: 'xxx',
       * name: 'xxx',
       * desc: 'xxxx'
       * }
       * @apiErrorExample ErrorExample
       */
      @GetMapping("/{id}")
      public Product detail(@PathVariable String id){
          return JsonData.buildSuccess();
      }
  ```



**Swagger** 丝袜哥

- 地址：https://swagger.io/tools/swagger-ui/

- 简介：在java代码里面增加注解生成接口文档

- 优点：支持SpringBoot等主流Java框架、对java代码友好、界面简洁、国内比较活跃,主要是spring社区带动、功能比较多
- 缺点：对跨语言支持不友好（可以和knife4j整合解决这个问题）、代码需要引入相关依赖包和配置、文档相对缺少

- 示例：在代码里面增加注解

  ```java
  RestController
  @RequestMapping("/api/v1/user")
  @Api(tags = "用户模块",value = "用户UserController")
  public class UserController {
  
      @Autowired
      private BannerService bannerService;
  
      @ApiOperation("分页用户列表")
      @GetMapping("list")
      public JsonData list(){
          List<BannerDO> list = bannerService.list();
          return JsonData.buildSuccess(list);
      }
  }
  ```

  

### 3、Swagger 3 快速入门

> **springfox3.0 已经支持swagger3了，之前SpringFox未支持 OpenAPI3 标准使用的是springdoc-openapi-ui依赖**

1、创建SpringBoot项目，并确保项目能够正确运行。

2、导入相关maven依赖（对比Swagger2，Swagger3只需要导入一个依赖即可）：

```xml
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

3、Swagger3配置类

```java
/**
 * @EnableOpenApi：开启Swagger3的注解。与Swagger2启动注解不同@EnableSwagger2
 * @EnableOpenApi 不配置也可以
 */
@EnableOpenApi
@Configuration
public class Swagger3Config {

}
```

4、访问swagger3-ui地址：http://localhost:8080/swagger-ui/index.html



### 4、Swagger 3 配置详解

application.properties配置文件

```properties
# 项目名称
spring.application.name=Swagger3.x api

########### 自定义swagger配置 ###########
# 是否开启swagger，开发环境打开，线上关闭
# 项目名称：enable
# 项目版本信息：application-version
# 项目描述信息：application-description
swagger.enable=true
swagger.application-name= ${spring.application.name}
swagger.application-version=1.0
swagger.application-description=Swagger3.x api info
```

Swagger配置类

```java
/**
 * @EnableOpenApi：开启Swagger3的注解。与Swagger2启动注解不同@EnableSwagger2
 * 默认不配置也可以
 */
@EnableOpenApi
@Configuration
public class Swagger3Config {
    /**
     * 是否开启swagger，生产环境一般关闭，所以这里成变量
     */
    @Value("${swagger.enable}")
    private Boolean enable;
    /**
     * 项目应用名
     */
    @Value("${swagger.application-name}")
    private String applicationName;
    /**
     * 项目版本信息
     */
    @Value("${swagger.application-version}")
    private String applicationVersion;
    /**
     * 项目描述信息
     */
    @Value("${swagger.application-description}")
    private String applicationDescription;

    /**
     * 配置了Swagger的Docket的bean实例：Swagger3 需要配置成 DocumentationType.OAS_30
     *  enable()：是否启动Swagger，如果为False，则Swagger不能在浏览器中访问
     *  apiInfo()：配置api文档元信息
     *  select()：选择哪些接口作为swagger的doc发布，一般和apis()一起使用
     *  apis(RequestHandlerSelectors.XXX)：配置要扫描的方式
     *      RequestHandlerSelectors.basePackage()：指定要扫描的包
     *      RequestHandlerSelectors.any()：扫描全部
     *      RequestHandlerSelectors.none()：不扫描
     *      RequestHandlerSelectors.withClassAnnotation()：扫描类上的注解，参数是注解的反射对象
     *      RequestHandlerSelectors.withMethodAnnotation()：扫描方法上的注解，参数是注解的反射对象
     *      例如：RequestHandlerSelectors.withClassAnnotation(ApiOperation.class)
     *  paths()：过滤什么路径，一般跟在select()之后，根据请求路径定义当前Docket需要包含控制器的哪些方法。
     *      PathSelectors.none()：不扫描
     *      PathSelectors.any()：扫描所有请求路径
     *      PathSelectors.ant("/*")：匹配Ant样式的路径模式
     *      PathSelectors.regex("/cat/.*")：匹配正则指定的正则表达式路径
     *  pathMapping()：默认请求都是以 / 根路径开始.如果我们的应用不是部署在根路径，
     *                 比如以/platform 部署，则可以通过一下方式设置请求的统一前缀。最终调用接口后会和paths拼接在一起.
     *                 如接口为：/v1/api/list，那么swagger-ui页面则显示/platform/v1/api/list
     *
     *  build：工厂模式
     *  注意：Docket.select().xxx.build()必须一起使用
     *        select() 返回的是一个ApiSelectorBuilder对象，而我们需要的却是Docket对象，
     *        因此，ApiSelectorBuilder类中提供了一个方法 --> build() 方法返回的是一个Docket对象。
     */
    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.OAS_30)
                .enable(enable)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build()
                .pathMapping("/");
    }

    /**
     * ApiInfo：主要返回接口和接口创建者的信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(applicationName)
                .description(applicationDescription)
                .contact(new Contact("lsx", "https://www.cnblogs.com/liusuixing/", "8629303@qq.com"))
                .version(applicationVersion)
                .build();
    }
}
```

**分组配置：**（其他配置都是一样的）

```java
@Bean
public Docket docketA(){
    return new Docket(DocumentationType.OAS_30).groupName("A");
}

@Bean
public Docket docketB(){
    return new Docket(DocumentationType.OAS_30).groupName("B");
}
```

**解决访问swaggerUI接口文档显示basic-error-controler问题：**

```java
@Bean
public Docket api() {
    return new Docket(DocumentationType.OAS_30)
        .apiInfo(apiInfo())
        .pathMapping("/")
        .select() // 选择那些路径和api会生成document
        .apis(RequestHandlerSelectors.any())// 对所有api进行监控
        // 不显示错误的接口地址
        .paths(Predicates.not(PathSelectors.regex("/error.*")))//错误路径不监控
        .paths(PathSelectors.regex("/.*"))// 对根下所有路径进行监控
        .build();
}
```







### 5、Swagger 3 常用注解

|                 注解                  |                             作用                             |
| :-----------------------------------: | :----------------------------------------------------------: |
|                 @Api                  |        用在Controller类上，对所标注的类进行描述说明。        |
|             @ApiOperation             |   用在COntroller类中的方法上，对所标注的方法进行描述说明。   |
| @ApiImplicitParams、@ApiImplicitParam | 方法的参数的说明；@ApiImplicitParams 用于指定单个参数的说明  |
|               @ApiParam               |              用于 Controller 中方法的参数说明。              |
|      @ApiResponse、@ApiResponses      |   方法返回值的说明 ；@ApiResponses 用于指定单个参数的说明    |
|     @ApiModel、@ApiModelProperty      |          用在实体类和属性上，对实体类和属性进行说明          |
|              @ApiIgnore               | 用在类、方法，参数中用来屏蔽某接口或参数，使其不在页面上显示 |

```
@Api：用在请求的类上，表示对类的说明（不配置默认是按类名驼峰变下划线显示）
    value：表示说明
    tags：也是说明，可以替代或者覆盖value，tags如果有多个值，会生成多个list
    description：对api资源的描述
    basePath：基本路径
    position：如果配置多个Api 想改变显示的顺序位置
    produces：如 “application/json, application/xml”
    consumes：如 “application/json, application/xml”
    protocols：协议类型，如: http, https, ws, wss.
    authorizations：高级特性认证时配置
    hidden：配置为true ，将在文档中隐藏

@ApiOperation：用在请求的方法上，说明方法的用途、作用
    value：用于方法描述
    tags：可以重新分组（视情况而用）
    notes：用于提示内容

@ApiImplicitParams：用在请求的方法上，表示一组参数说明
    @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面
        name：参数名
        value：参数的汉字说明、解释
        required：参数是否必须传
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · div（不常用）
            · form（不常用）    
        dataType：参数类型，默认String，其它值dataType="Integer"       
        defaultValue：参数的默认值

@ApiParam() 用于方法，参数，字段说明；表示对参数的添加元数据（说明或是否必填等）
    name：参数名
    value：参数说明
    required：是否必填
    defaultValue：默认值

@ApiResponses：用在请求的方法上，表示一组响应
    @ApiResponse：用在@ApiResponses中，一个@ApiResponse表示一种响应信息
        responseCode：字符串，例如"400"
        description：信息，例如"请求参数没填好"

@ApiModel：用于响应类上，表示一个返回响应数据的信息
（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候）
    value：自定义类的名称
    description：为类添加长文本描述信息
    
@ApiModelProperty：用在属性上，描述响应类的属性
    value：定义参数描述信息
    name：定义参数名称
    required：是否必填
```

操作实例：

```java
package com.example.swagger.controller;

import com.example.swagger.model.ApiResult;
import com.example.swagger.model.User;
import io.swagger.annotations.*;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import org.springframework.web.bind.annotation.*;

@Api(tags = "用户模块" ,description = "User module")
@RestController
@RequestMapping("/user")
public class UserController {

    /**
     * 参数形式一：@ApiImplicitParams、@ApiImplicitParam
     */
    @ApiOperation(value = "新增用户A", notes = "添加新的用户", tags = "用户模块",httpMethod = "POST")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "name", value = "用户名", required = true, dataType = "String"),
            @ApiImplicitParam(name = "age", value = "年龄", required = true, dataType = "Integer"),
            @ApiImplicitParam(name = "email", value = "邮箱", required = true, dataType = "String")
    })
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "保存成功"),
            @ApiResponse(responseCode = "400", description = "保存失败")
    })
    @PostMapping("/swagger3AnnotationA")
    public ApiResult swagger3AnnotationA(@RequestParam("name") String name,
                                         @RequestParam("age") Integer age,
                                         @RequestParam("email") String email){
        return ApiResult.builder().code(200).message("返回成功").build();
    }

    /**
     * 参数形式二：@ApiParam
     */
    @ApiOperation(value = "新增用户B", notes = "添加新的用户", tags = "用户模块",httpMethod = "POST")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "保存成功"),
            @ApiResponse(responseCode = "400", description = "保存失败")
    })
    @PostMapping("/swagger3AnnotationB")
    public ApiResult swagger3AnnotationB(
            @ApiParam(name = "name", value = "用户名") @RequestParam("name") String name,
            @ApiParam(name = "age", value = "年龄")@RequestParam("age") String age,
            @ApiParam(name = "email", value = "邮箱")@RequestParam("email") String email){
        return ApiResult.builder().code(200).message("返回成功").build();
    }

    /**
     * 参数形式三：@ApiModel、@ApiModelProperty
     */
    @ApiOperation(value = "新增用户C", notes = "添加新的用户", tags = "用户模块",httpMethod = "POST")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "保存成功"),
            @ApiResponse(responseCode = "400", description = "保存失败")
    })
    @PostMapping("/swagger3AnnotationC")
    public ApiResult swagger3AnnotationC(@RequestBody User user){
        return ApiResult.builder().code(200).message("返回成功").data(user).build();
    }
}
```

两个model类：

```java
@Data
@Builder
@ApiModel(description= "返回响应数据")
public class ApiResult {
    @ApiModelProperty(value = "是否成功",required=true)
    private boolean success = true;
    @ApiModelProperty(value = "http状态码")
    private Integer code;
    @ApiModelProperty(value = "提示信息")
    private String message;
    @ApiModelProperty(value = "数据")
    private Object data;
}
```

```java
@Data
@ApiModel(description= "User实体类参数")
public class User {
    @ApiModelProperty(value = "用户名")
    private String name;
    @ApiModelProperty(value = "年龄")
    private Integer age;
    @ApiModelProperty(value = "邮箱")
    private String email;
}
```





## 3、Swagger 2

### 1、Swagger 2 简介

> **官网：**https://swagger.io/

**优点：**

1. 可以通过Swagger给一些比较难理解的属性或者接口增加注释信息
2. 接口文档实时更新
3. 可以在线测试



### 2、Swagger 2 快速入门

1、使用IDEA创建一个SpringBoot项目

2、导入相关依赖（项目中使用Swagger需要springfox中的：swagger2、swagger-ui）

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

3、通过JavaConfig对Swagger2进行配置

```java
package com.example.swagger2.config;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.swagger2.annotations.EnableSwagger2;
//开启Swagger2
@EnableSwagger2
@Configuration
public class SwaggerConfig {

}
```

4、测试访问**Swagger-UI页面地址（Swagger2访问地址）**：http://localhost:8080/swagger-ui.html



### 3、Swagger 2 配置详解

1、Swagger2配置：

- 配置Bean实例Docket

- 扫描接口：Docket.select()：配置扫描包和路径

```Java
package com.example.swagger2.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@EnableSwagger2 //开启Swagger2
@Configuration
public class Swagger2Config {
    /**
     * 配置了Swagger的Docket的bean实例
     *  apis(RequestHandlerSelectors.XXX)：配置要扫描的方式
     *      RequestHandlerSelectors.basePackage：指定要扫描的包
     *      RequestHandlerSelectors.any()：扫描全部
     *      RequestHandlerSelectors.none()：不扫描
     *      RequestHandlerSelectors.withClassAnnotation()：扫描类上的注解，参数是一个注解的反射对象
     *      RequestHandlerSelectors.withMethodAnnotation()：扫描方法上的注解，参数是一个注解的反射对象
     *   paths()：过滤什么路径
     *   enable()：是否启动Swagger，如果为False，则Swagger不能在浏览器中访问
     */
    @Bean
    public Docket docket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .enable(false)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example"))
                .paths(PathSelectors.ant("/**"))
                .build();
    }

    /**
     * 配置Swagger信息
     */
    private ApiInfo apiInfo(){
        return new ApiInfo("SpringBoot项目Swagger的API文档",
                "只要学不死，就往死里学",
                "1.0",
                "https://liusuixing.gitee.io/",
                // 作者信息
                new Contact("lsx", "https://liusuixing.gitee.io/", "8629303@qq.com"),
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList());
    }
}
```

- **希望Swagger在生产环境中使用，正式发布环境不使用：**
  - **判断是不是生成环境**
  - **注入enable（flag）**

```java
package com.example.swagger2.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.core.env.Profiles;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;
import java.util.ArrayList;

@EnableSwagger2 //开启Swagger2
@Configuration
public class Swagger2Config {
    @Autowired
    Environment environment;
    @Bean
    public Docket docket() {
        // 设置要显示的Swagger环境,通过environment.acceptsProfiles 判断是否处在自己设定的环境当中
        Profiles profiles = Profiles.of("dev", "text");
        boolean flag = environment.acceptsProfiles(profiles);
        System.out.println(flag);
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .enable(flag)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example"))
                .paths(PathSelectors.ant("/**"))
                .build();
    }

    /**
     * 配置Swagger信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfo("SpringBoot项目Swagger的API文档",
                "只要学不死，就往死里学",
                "1.0",
                "https://liusuixing.gitee.io/",
                // 作者信息
                new Contact("lsx", "https://liusuixing.gitee.io/", "8629303@qq.com"),
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList());
    }
}
```



**配置Api文档分组**

```java
@Bean
public Docket docket1(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("A");
}

@Bean
public Docket docket2(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("B");
}
```



### 4、Swagger 2 常用注解

> **参考：**Swagger3.X常用注解即可.

```java
@Api：用在请求的类上，表示对类的说明（不配置默认是按类名驼峰变下划线显示）
    value：该参数没什么意义，在UI界面上也看到，所以不需要配置"
    tags：说明该类的作用，可以在UI界面上看到的注解
    description：对api资源的描述
    basePath：基本路径
    position：如果配置多个Api 想改变显示的顺序位置
    produces：如 “application/json, application/xml”
    consumes：如 “application/json, application/xml”
    protocols：协议类型，如: http, https, ws, wss.
    authorizations：高级特性认证时配置
    hidden：配置为true ，将在文档中隐藏

@ApiOperation：用在请求的方法上，说明方法的用途、作用
    value：说明方法的用途、作用
    tags：如果设置这个值、value的值会被覆盖
    notes：方法的备注说明
    description：对api资源的描述

@ApiImplicitParams：用在请求的方法上，表示一组参数说明
    @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面
        name：参数名
        value：参数的汉字说明、解释
        required：参数是否必须传
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · div（不常用）
            · form（不常用）    
        dataType：参数类型，默认String，其它值dataType="Integer"       
        defaultValue：参数的默认值

@ApiParam() 用于方法，参数，字段说明；表示对参数的添加元数据（说明或是否必填等）
    name：参数名
    value：参数说明
    required：是否必填

@ApiResponses：用在请求的方法上，表示一组响应
    @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：抛出异常的类

@ApiModel：用于响应类上，表示一个返回响应数据的信息
            （这种一般用在post创建的时候，使用@RequestBody这样的场景，
            请求参数无法使用@ApiImplicitParam注解进行描述的时候）
@ApiModelProperty：用在属性上，描述响应类的属性
```





#### 1、@Api

@Api 注解用于标注一个Controller（Class）。在默认情况下，Swagger-Core只会扫描解析具有@Api注解的类，而会自动忽略其他类别资源（JAX-RS endpoints，Servlets等等）的注解。

主要属性如下：

| 属性           | 描述                                             |
| -------------- | ------------------------------------------------ |
| value          | url的路径值                                      |
| tags           | 如果设置这个值、value的值会被覆盖                |
| description    | 对api资源的描述（）                              |
| basePath       | 基本路径可以不配置                               |
| position       | 如果配置多个Api 想改变显示的顺序位置             |
| produces       | For example, "application/json, application/xml" |
| consumes       | For example, "application/json, application/xml" |
| protocols      | Possible values: http, https, ws, wss.           |
| authorizations | 高级特性认证时配置                               |
| hidden         | 配置为true 将在文档中隐藏                        |

示例：

```java
@Api(tags = "用户模块" ,description = "user module")
@RestController
@RequestMapping("/user")
public class UserController {
}
```



#### 2、@ApiOperation

@ApiOperation 注解在用于对一个操作或HTTP方法进行描述。具有相同路径的不同操作会被归组为同一个操作对象。不同的HTTP请求方法及路径组合构成一个唯一操作。

主要属性：

| 属性              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| value             | url的路径值                                                  |
| tags              | 如果设置这个值、value的值会被覆盖                            |
| notes             | 操作的详细描述                                               |
| description       | 对api资源的描述                                              |
| basePath          | 基本路径可以不配置                                           |
| position          | 如果配置多个Api 想改变显示的顺序位置                         |
| produces          | For example, "application/json, application/xml"             |
| consumes          | For example, "application/json, application/xml"             |
| protocols         | Possible values: http, https, ws, wss.                       |
| authorizations    | 高级特性认证时配置                                           |
| hidden            | 配置为true 将在文档中隐藏                                    |
| response          | 返回的对象                                                   |
| responseContainer | 这些对象是有效的 "List", "Set" or "Map".，其他无效           |
| httpMethod        | "GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS" and "PATCH" |
| code              | http的状态码 默认 200                                        |
| extensions        | 扩展属性                                                     |

示例：

```java
    // @ApiOperation("分页用户列表")
    @ApiOperation(value = "分页用户列表", notes = "对用户列表进行分页")
    @GetMapping("/pageOne")
    public ResponseEntity pageOne(){
        return ResponseEntity.ok("分页用户列表");
    }
```



#### 3、@ApiParam

@ApiParam作用于请求方法上，定义api参数的注解。

主要属性：

| 属性            | 描述         |
| --------------- | ------------ |
| name            | 属性名称     |
| value           | 属性值       |
| defaultValue    | 默认属性值   |
| allowableValues | 可以不配置   |
| required        | 是否属性必填 |
| access          | 不过多描述   |
| allowMultiple   | 默认为false  |
| hidden          | 隐藏该属性   |
| example         | 举例子       |

实例：

```java
    @ApiOperation(value = "分页用户列表", notes = "对用户列表进行分页")
    @GetMapping("/pageTwo")
    public ResponseEntity pageTwo(
            @ApiParam(value = "当前页数", required = true) @RequestParam("page") Integer page,
            @ApiParam(value = "每页显示数", required = true) @RequestParam("limit") Integer limit){
        return ResponseEntity.ok("分页用户列表");
    }
```





#### 4、@ApiImplicitParams、@ApiImplicitParam

@ApiImplicitParams、@ApiImplicitParam也可以定义参数.

- @ApiImplicitParams：用在请求的方法上，包含一组参数说明
- @ApiImplicitParam：对单个参数的说明

主要属性：

| 属性         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| name         | 参数名                                                       |
| value        | 参数的说明、描述                                             |
| required     | 参数是否必须必填                                             |
| paramType    | 参数放在哪个地方 query --> 请求参数的获取：@RequestParam header --> 请求参数的获取：@RequestHeader path（用于restful接口）--> 请求参数的获取：@PathVariable body（请求体）--> @RequestBody User user form（普通表单提交） |
| dataType     | 参数类型，默认String，其它值dataType="Integer"               |
| defaultValue | 参数的默认值                                                 |

实例：

```java
    @ApiOperation(value = "用户查询", notes = "根据用户信息查询用户")
    @ApiImplicitParams({
            @ApiImplicitParam(name="mobile",value="手机号",required=true,paramType="form"),
            @ApiImplicitParam(name="password",value="密码",required=true,paramType="form"),
            @ApiImplicitParam(name="age",value="年龄",required=true,paramType="form",dataType="Integer")
    })
    @PostMapping("/findUserByMobileAndPwdAndAge")
    public ResponseEntity findUserByMobileAndPwdAndAge(@RequestParam String mobile,
                                  @RequestParam String password,
                                  @RequestParam Integer age){
        return ResponseEntity.ok("用户查询");
    }
```





#### 5、@ApiResponses、@ApiResponse

@ApiResponses、@ApiResponse进行方法返回对象的说明

主要属性：

| 属性     | 描述                       |
| -------- | -------------------------- |
| code     | 数字，例如400              |
| message  | 信息，例如"请求参数没填好" |
| response | 抛出异常的类               |

实例：

```java
    @ApiOperation(value = "查询所有用户", notes = "查询所有用户信息")
    @ApiResponses({
            @ApiResponse(code = 200, message = "查询成功"),
            @ApiResponse(code = 400, message = "查询失败")
    })
    @GetMapping("/findAll")
    public ResponseEntity findAll(){
        HashMap<String, String > map  = new HashMap<String, String>(){{
            put("code","200");
        }};
        return ResponseEntity.ok().body(map);
    }
```





#### 6、@ApiModel、@ApiModelProperty

@ApiModel用于描述一个Model的信息（一般用在使用@RequestBody的场景，请求参数无法使用@ApiImplicitParam描述）

@ApiModelProperty用来描述一个Model的属性。

ApiModel：主要属性

| 属性名称      | 属性类型 | 默认值 | 作用                   |
| ------------- | -------- | ------ | ---------------------- |
| value()       | String   | 空     | 自定义类的名称         |
| description() | String   | 空     | 为类添加长文本描述信息 |

ApiModelProperty：主要属性

| 属性名称          | 属性类型 | 默认值 | 作用                 |
| ----------------- | -------- | ------ | -------------------- |
| value()           | String   | 空     | 定义参数描述信息     |
| name()            | String   | 空     | 定义参数名称         |
| required()        | boolean  | false  | 定义参数是否必传     |
| hidden()          | boolean  | false  | 定义参数是否隐藏     |
| allowEmptyValue() | boolean  | false  | 定义参数是否允许为空 |

实例：

```java
@Data
@ApiModel(description= "返回响应数据")
public class ApiResult {
    @ApiModelProperty(value = "是否成功",required=true)
    private boolean success = true;

    @ApiModelProperty(value = "http状态码")
    private Integer code;

    @ApiModelProperty(value = "提示信息")
    private String message;

    @ApiModelProperty(value = "数据")
    private Object data;
}
```





## 4、Swagger 增强神器 Knife4j

> 参考：https://mp.weixin.qq.com/s/4HGYX0wpBk8-p3AfFbbBVg

简单使用（在使用Swagger3的基础上）添加依赖：

```xml
<!-- https://mvnrepository.com/artifact/com.github.xiaoymin/knife4j-spring-boot-starter -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>3.0.2</version>
</dependency>
```

经过以上简单的依赖添加之后，无需配置任何内容，我们就可以成功的使用 Knife4j 了。访问 Knife4j 的主页： http://localhost:8080/doc.html 



## X、参考文献 & 鸣谢

1. Swagger常用注解【博客园】https://www.cnblogs.com/three-fighter/p/12346184.html
1. 也许是最全面的 Swagger 注解【CSDN】https://blog.csdn.net/dejunyang/article/details/89527586