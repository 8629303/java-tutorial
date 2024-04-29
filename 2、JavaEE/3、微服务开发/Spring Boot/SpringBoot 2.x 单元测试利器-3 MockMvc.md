

## 一、MockMvc 的概念

MockMvc是由spring-test / spring-boot-starter-test包提供，实现了对http请求的模拟，能够直接使用网络的形式，转换到Controller的调用，使得测试速度快、不依赖网络环境。同时提供了一套验证的工具，结果的验证十分方便。

接口MockMvcBuilder，提供一个唯一的build方法，用来构造MockMvc。主要有两个实现：StandaloneMockMvcBuilder和DefaultMockMvcBuilder，分别对应两种测试方式，即独立安装和集成Web环境测试（并不会集成真正的Web环境，而是通过相应的Mock API进行模拟测试，无须启动服务器）

| 实例化方法 |                          实例&解释                           |
| :--------: | :----------------------------------------------------------: |
|   方法一   | mockMvc = MockMvcBuilders.standaloneSetup(new HelloWorldController()).build();<br />通过参数指定一组控制器，这样就不需要从上下文获取了 |
|   方法二   | mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();<br />指定WebApplicationContext，将会从该上下文获取相应的控制器并得到相应的MockMvc |
|   方法三   | @SpringBootTest + @AutoConfigureMockMvc + @Autowired MockMvc mockMvc |

mockMvc的基本步骤：

1. mockMvc.perform：执行一个请求
2. MockMvcRequestBuilders.get("/v1/api")：构造一个请求，返回 MockHttpServletRequestBuilder 对象
3. MockHttpServletRequestBuilder.param：添加请求传值
4. MockHttpServletRequestBuilder.accept：设置返回类型
5. ResultActions.andExpect：添加执行完成后的断言
6. ResultActions.andDo：添加一个结果处理器，表示要对结果做的事情，比如MockMvcResultHandlers.print()输出整个响应结果
7. ResultActions.andReturn：表示执行完成后返回相应的结果

MockMvcBuilder是用来构造MockMvc的构造器，其主要有两个实现：StandaloneMockMvcBuilder和DefaultMockMvcBuilder，StandaloneMockMvcBuilder继承了DefaultMockMvcBuilder。直接使用静态工厂MockMvcBuilders创建即可：
MockMvcBuilders.webAppContextSetup(WebApplicationContext context)：指定WebApplicationContext，将会从该上下文获取相应的控制器并得到相应的MockMvc；



## 二、MockMvc  代码实战

MockMvc类的使用和模拟Http请求实战：

- 增加类注解：@SpringBootTest + @AutoConfigureMockMvc
- 主要API及方法：
  - perform：执行一个RequestBuilder请求
  - andExpect：添加ResultMatcher->MockMvcResultMatchers验证规则
  - andReturn：最后返回相应的MvcResult->Response
- MockMvc 的perform方法内编写发送请求的url、编码、内容等信息，后面的.andExpect方法则是对获取返回信息的验证判断。

### 1、导入依赖

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
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

### 2、定义数据类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Integer id;
    private String username;
    private String nickName;
    private Integer age;
    private String password;
}
```

### 3、定义控制层

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;
import java.util.Collections;
import java.util.List;

@RestController
@RequestMapping("/user")
@Slf4j
public class UserController {

    /**
     * 根据name和age查询用户信息
     */
    @GetMapping
    public List<User> getUserByName(@RequestParam("username") String name, @RequestParam("age") Integer age) {
        log.info("username:{}", name);
        log.info("age:{}", age);
        User user = new User(1, name, "xiaoxx", age, "123456");
        return Collections.singletonList(user);
    }

    /**
     * 根据ID获取用户的详细信息
     */
    @GetMapping("/{id:\\d+}")
    public User getUserInfoById(@PathVariable Integer id) {
        log.info("username:{}", id);
        User user = new User(1, "zhangsan", "xiaoxx", 10, "123456");
        return user;
    }

    /**
     * 添加用户信息
     */
    @PostMapping
    public void addUser(@RequestBody User user) {
        log.info("user:{}", user);
    }

    /**
     * 根据用户ID修改用户数据
     */
    @PutMapping("/{id:\\d+}")
    public void update(@PathVariable Integer id, @RequestBody User user) {
        log.info("update user id:{}", id);
        log.info("update user:{}", user);
    }

    /**
     * 根据用户id删除
     */
    @DeleteMapping("/{id}")
    public void delete(@PathVariable Integer id) {
        log.info("delete user id:{}", id);
    }
}
```

### 4、MockMvc 测试

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultHandlers;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

@Slf4j
@SpringBootTest
@AutoConfigureMockMvc
public class SpringBootMockMvcTests {

    @Autowired
    MockMvc mockMvc;

    /**
     * 根据用户名查询的测试用例
     */
    @Test
    public void queryUserSuccessByUserName() throws Exception {
        String responseStr = mockMvc.perform(
                        // 请求构建对象
                        MockMvcRequestBuilders
                                // 指定请求的restful api的地址,.get 就是表示发送get方法
                                .get("/user")
                                // 指定请求内容的格式
                                .contentType(MediaType.APPLICATION_JSON)
                                // 参数:姓名
                                .param("username", "zhangsan")
                                // 参数:年龄
                                .param("age", "1"))
                // 打印结果倒控制台
                .andDo(MockMvcResultHandlers.print())
                // 指定响应的预期状态码
                .andExpect(MockMvcResultMatchers.status().isOk())
                // 指定响应预期的内容, jsonPath : https://github.com/json-path/JsonPath
                .andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(1))
                // 获取到响应数据
                .andReturn().getResponse().getContentAsString();

        log.info("return string:{}", responseStr);
    }

    /**
     * 根据ID查询用户详情的测试用例
     */
    @Test
    public void queryUserSuccessByIdSuccess() throws Exception {
        String responseStr = mockMvc.perform(
                        // 请求构建对象
                        MockMvcRequestBuilders
                                // 指定请求的restful api的地址, .get 就是表示发送get方法
                                .get("/user/1")
                                // 指定请求内容的格式
                                .contentType(MediaType.APPLICATION_JSON))
                // 打印结果倒控制台
                .andDo(MockMvcResultHandlers.print())
                // 指定响应的预期状态码
                .andExpect(MockMvcResultMatchers.status().isOk())
                // 指定响应预期的内容, 要求返回的对象的用户名为：zhangsan
                .andExpect(MockMvcResultMatchers.jsonPath("$.username").value("zhangsan"))
                // 获取到响应数据
                .andReturn().getResponse().getContentAsString();

        log.info("return string:{}", responseStr);
    }

    /**
     * 根据ID查询用户信息的错误场景测试用例
     * 当如果传递的用户ID不是数据 {id:\\d+}正则表达式匹配失败的情况
     */
    @Test
    public void queryUserSuccessByIdFail() throws Exception {
        String responseStr = mockMvc.perform(
                        // 请求构建对象
                        MockMvcRequestBuilders
                                // 指定请求的restful api的地址, .get 就是表示发送get方法
                                .get("/user/mm")
                                // 指定请求内容的格式
                                .contentType(MediaType.APPLICATION_JSON))
                // 打印结果倒控制台
                .andDo(MockMvcResultHandlers.print())
                // 指定响应的预期状态码为4xx
                .andExpect(MockMvcResultMatchers.status().is4xxClientError())
                // 获取到响应数据
                .andReturn().getResponse().getContentAsString();

        log.info("return string:{}", responseStr);
    }

    /**
     * 添加的测试用例
     *
     * @throws Exception
     */
    @Test
    public void addUserSuccess() throws Exception {
        String content = "{\"username\":\"wangwu\",\"age\":25,\"nickName\":\"wuwu\",\"password\":\"123321\"}";
        mockMvc.perform(
                        MockMvcRequestBuilders
                                .post("/user")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(content))
                .andExpect(MockMvcResultMatchers.status().isOk());
    }

    /**
     * 修改的测试用例
     */
    @Test
    public void updateUserSuccess() throws Exception {
        String content = "{\"username\":\"wangwu\",\"age\":30,\"nickName\":\"wwuwu\",\"password\":\"789456\"}";
        mockMvc.perform(
                        //
                        MockMvcRequestBuilders
                                // 请求对象
                                .put("/user/1")
                                // 请求内容的个数
                                .contentType(MediaType.APPLICATION_JSON)
                                // 请求数据
                                .content(content))
                // 响应状态要求
                .andExpect(MockMvcResultMatchers.status().isOk());
    }

    /**
     * 删除的测试用例
     *
     * @throws Exception
     */
    @Test
    public void deleteUserSuccess() throws Exception {
        mockMvc.perform(
                        MockMvcRequestBuilders
                                .delete("/user/1")
                                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isOk());
    }
}
```

> JsonPath 详情可参考官方文档：https://github.com/json-path/JsonPath



## 三、MockMvc 常用属性

- MockMvcRequestBuilders：请求构建者对象

- - get、post、put、delete分别表示发送对应Method的请求，参数为对应的api地址：MockMvcRequestBuilders.get("/api")
    - param：表示要传递的请求参数；如：param("username","zhangsan")
    - contentType：指定请求数据的数据格式；如: MediaType.APPLICATION_JSON 表示请求数据为Json格式的数据
    - content：请求后端传递的数据；数据格式需要和contentType指定的对应
    - header：指定header信息

- ResultActions：对返回结果的处理对象

  - andDo：添加一个结果处理器，表示要对结果做的事情，

    - MockMvcResultHandlers.print() 表示输出整个响应结果

  - andExpect：添加期望的结果

    - MockMvcResultMatchers.status().isOk() 表示期望的响应状态码为200

    - MockMvcResultMatchers.status().is4xxClientError()表示期望的响应状态码为4xx 还可以是3xx、5xx等状态

    - MockMvcResultMatchers.jsonPath().value() 表示期望响应的json数据达到什么预期

      ```java
      .andExpect(MockMvcResultMatchers.status().isOk());
      // 表示期望响应的code值为200
      .andExpect(MockMvcResultMatchers.jsonPath("$.code").value(200));
      // 表示期望响应的JsonArray的元素个数为10个；
      .andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(10));
      // 表示期望响应的JsonObject中的username值为zhangsan
      .andExpect(MockMvcResultMatchers.jsonPath("$.username").value("zhangsan");
      // 表示期望响应的JsonObject中的data.type值为good
      .andExpect(MockMvcResultMatchers.jsonPath("$.data.type").value("good"));
      ```
  
  - andReturn：获取响应的对象
  
    - getResponse() 获取响应的response对象
    - getResponse().getContentAsString() 获取响应的文本信息



## 四、MockMvc 快速总结

现在将上面的一些琐碎的知识点汇总一下：

1. mockMvc.perform：执行一个请求
2. MockMvcRequestBuilders.get(“/XXX/get”)：构造一个请求，Post请求使用.post方法
3. contentType(MediaType.APPLICATION_JSON_VALUE)：代表发送端发送的数据格式是application/json;charset=UTF-8
4. accept(MediaType.APPLICATION_JSON)：代表客户端希望接受的数据类型为application/json;charset=UTF-8
5. header(“Authorization”,“Bearer XXXX”)：代表在报文头添加一些必须的信息，这里添加的是token
6. ResultActions.andExpect：添加执行完成后的断言
7. ResultActions.andExpect(MockMvcResult Matchers.status().isOk())：方法看请求的状态响应码是否为200如果不是则抛异常，测试不通过
8. ResultActions.andDo：添加一个结果处理器，表示要对结果做点什么事情，比如此处使用print()：输出整个响应结果信息



## 五、参考文献 & 鸣谢

1. Spring MVC服务端单元测试工具MockMvc：https://mp.weixin.qq.com/s/jQ0qrVUiOxPu-HLMkDSECw
2. springboot 单元测试Controller层方法MockMvc：https://blog.csdn.net/coolcoffee168/article/details/88638042
3. Spring Boot单元测试(Controller层的Header处有Token验证)：https://mp.weixin.qq.com/s/2_IjJU1eKQjfjlDEJwgnSA
4. SpringBoot基础之MockMvc单元测试：https://blog.csdn.net/IManiy/article/details/120833041