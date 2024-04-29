# SpringBoot 配置外部化



## 0、配置文件加载优先级

### 1、内部配置加载顺序

SpringBoot 启动会扫描以下位置的 application.properties 或者 application.yml 文件作为 SpringBoot 的默认配置文件

1. –file:./config/：当前项目根目录下的 config 目录下（Jar 包当前目录下的 `/config` 目录）**【优先级最高】**
2. –file:./：当前项目的根目录下（Jar 包当前目录下的 `/config` 目录）
3. –classpath:/config/：resources/config 目录下
4. –classpath:/：resources 目录下**【优先级最低】【常用】**

优先级 由高到底，高优先级的配置会覆盖低优先级的配置。

- 如果低优先级存在高优先级没有的属性，则会互补配置。
- 如果同一个配置属性，在多个配置文件都配置了，默认使用第1个读取到的

这四个位置是默认位置，即 Spring Boot 启动，默认会从这四个位置按顺序去查找相关属性并加载。

我们也可以通过 spring.config.location 属性来手动的指定配置文件位置，指定完成后，系统就会自动去指定目录下查找 application.properties 文件。

```bash
java -jar properties-0.0.1-SNAPSHOT.jar --spring.config.location=classpath:/javaboy/
```

项目默认加载的配置文件名是 application ，我们可以通过属性 spring.config.name 来指定配置文件的文件名。

使用 spring.config.location 属性重新定义配置文件的位置，它会覆盖掉默认的四个位置。使用 spring.config.additional-location 这个属性自定义配置文件位置，表示在默认的四个位置的基础上，再添加几个位置，新添加的位置的优先级大于原本的位置。 在自定义配置文件位置时，值一定要以 / 结尾。例如：

```
spring.config.location=classpath:/myconfig/
```



### 2、外部配置加载顺序

SpringBoot 也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置

**（1）命令行参数 （加载时优先加载）**

- 所有的配置都可以在命令行上进行指定（ --配置）。

- 可以通过指定 --spring.config.location 来改变默认的配置文件位置（多个配置用空格分开；格式为： --配置项=值）：

  ```bash
  java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.ym
  ```

（2）来自 java:comp/env 的 JNDI 属性

（3）Java 系统属性（在命令行中使用`-D`指定），可通过 System.getProperties() 获得相关内容

```bash
java -Dspring.application.json='{"my":{"name":"test"}}' -jar myapp.jar
```

（4）操作系统环境变量

- 大多数操作系统不允许使用句点分隔的键名，使用下划线代替（例如，SPRING_CONFIG_NAME代替spring.config.name）

（5）RandomValuePropertySource 配置的 random.*属性值

**（6）Jar包外部的 application-{profile}.properties 或 application-{profile}.yml（带spring.profile）配置文件。**

**（7）Jar包内部的 application-{profile}.properties 或 application-{profile}.yml（带spring.profile）配置文件。**

**（8）Jar包外部的 application.properties 或 application.yml（不带spring.profile）配置文件。**

**（9）Jar包内部的application.properties 或 application.yml（不带spring.profile）配置文件。**

**（10）@Configuration 注解类上的 @PropertySource**

（11）通过 SpringApplication.setDefaultProperties 指定的默认属性

其中常用的为：（1）（6）（7）（8）（9）（10）

**总结：**

- 优先加载操作系统层面的配置、命令行
- 由 jar 包外向 jar 包内进行寻找，优先查找 config 目录。
- 优先加载带 profile（application-{profile}.yml）的，后加载不带 profile（application.yml）的
- 高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置



### 3、远程配置中心

以 Nacos 为例：默认远程配置优先级最高。**可以通过在远程配置中心中做如下配置，设置本地配置覆盖远程配置：**

```yaml
spring:
  cloud:
    config:
      # 是否允许本地配置覆盖远程配置
      allow-override: true
      # 是否一切以本地配置为准，默认false
      override-none: false
      # 系统环境变量或系统属性才能覆盖远程配置文件的配置
      # 本地配置文件中配置优先级低于远程配置，默认true
      override-system-properties: true
```





## 1、属性顺序

### 1、顺序

SpringBoot 使用一个非常特殊的 PropertySource 顺序，该顺序旨在允许合理地覆盖值。 属性按以下顺序考虑属性：

1. 在 home 主目录上的 [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/using-boot-devtools.html#using-boot-devtools-globalsettings)（`~/.spring-boot-devtools.properies`当 devtools 处于活动状态时）。

2. 测试的 @TestPropertySource 注解。

3. 测试的 properties 属性。在 @SpringBootTest 和测试注解@Test中可用的属性，用于测试应用程序的特定部分。

4. 命令行参数（Command line arguments）,命令行中通过 `--` 参数传递的配置项会覆盖其他配置项。

5. 来自 SPRING_APPLICATION_JSON 环境变量的属性：SPRING_APPLICATION_JSON 环境变量中的属性会被加载到应用程序的环境变量中。

6. ServletConfig 初始化参数。

7. ServletContext 初始化参数。

8. 来自 java:comp/env 的 JNDI 属性。

9. Java 系统属性（System.getProperties()）。

10. 操作系统环境变量。

11. RandomValuePropertySource 中以 random.* 为前缀的 属性。

12. jar 包外部的特定的 profile 配置（application-{profile}.properties 和 YAML 文件）。

13. jar 包内部的特定的 profile配置（application-{profile}.properties 和 YAML 文件）。

14. jar 包外部的 Application properties 文件（application.properties 和 YAML 文件）。

15. jar 包内部的 Application properties 文件（application.properties 和 YAML 文件）。

16. @Configuration注解作用类上的 @PropertySource 注解。

    注意：在刷新应用程序上下文之前，不会将此类属性源添加到环境（Environment）中。 此时置某些属性（如logging.* 和 spring.main.*）为时已晚，这些属性在刷新开始之前就已被读取。

17. 默认的属性（由 SpringApplication.setDefaultProperties 设置指定）。



### 2、示例

一个具体的示例，假设开发了一个定义了 name 属性的 @Component，如下示例：

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {
    @Value("${name}")
    private String name;
}
```

在的应用程序类路径上（例如，在 jar 内部），可以有一个 application.properties 文件，该配置文件为 name 提供合理的默认属性值。

当在一个新的环境运行时，可以在外部提供一个 application.properties文件，覆盖 name属性的值。对于一次性测试，可以使用特定的**命令行**开关启动（例如，`java -jar app.jar --name ="Spring"`）。

**备注：**

SPRING_APPLICATION_JSON 属性文件可以在带有环境变量的命令行上提供。例如，可以在 Unix shell 中使用以下行：

```bash
$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar
```

上面的示例，最终在 Spring Environment 中使用 acme.name=test。还可以在系统属性中将 JSON 作为 spring.application.json 提供，如下例所示：

```bash
$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar
```

也可以通过使用命令行参数来提供 JSON。如下示例：

```bash
$ java -jar myapp.jar --spring.application.json='{"name":"test"}'
```

也可以将 JSON 作为 JNDI 的变量来提供，如：`java:comp/env/spring.application.json`。



## 2、配置随机数

`RandomValuePropertySource`对于注入随机值（例如，在机密或测试用例中）非常有用。它可以生成整数、long、uuid或字符串，如下例所示：

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

random.int* 语法是 OPEN value (, max) CLOSE，其中 OPEN,CLOSE 是任何字符，value, max 是整数。 如果提供了max，则 value 是最小值，而 max 是最大值（不包括）。



## 3、访问命令行属性

默认情况下，SpringApplication 会转换任何命令行参数（即，以 `--`开头的参数，如，`--server.port=9000`）为属性，并将它将加入到 Spring Environment。如前所述，命令行属性始终优先于其他属性源。

如果不想将命令行属性加入到 Environment，可以使用下面设置禁用它们：

```java
SpringApplication.setAddCommandLineProperties(false);
```

操作示例：

```java
public static void main(String[] args) {
    // SpringApplication.run(FileServerApplication.class, args);
    SpringApplication springApplication = new SpringApplication(FileServerApplication.class);
    springApplication.setAddCommandLineProperties(false);
    springApplication.run(args);
}
```



## 4、应用属性文件

### 1、加载位置

SpringApplication 从以下位置查找 application.properties 文件加载属性，并将它们加入到 Spring Environment 中。

1. 当前目录的 /config 子目录。（Jar 包当前目录下的 `/config` 目录）
2. 当前目录。（Jar 包当前目录下）
3. 类路径中的 /config 目录。
4. 根类路径。

**注意：**该列表按照优先级的顺序排列（在列表中较高的位置定义的属性将会覆盖在较低位置定义的属性）

优先级由高到底，高优先级的配置会覆盖低优先级的配置。

- 如果低优先级存在高优先级没有的属性，则会互补配置。
- 如果同一个配置属性，在多个配置文件都配置了，默认使用第1个读取到的

例如：当前目录的 /config 与根类路径下都有一份相同的配置文件，则根类路径下的配置文件会覆盖当前目录 /config下的配置文件。



### 2、指定配置文件

项目默认加载的配置文件名是 application ，我们可以通过属性 spring.config.name 来指定配置文件的文件名：

```bash
$ java -jar myproject.jar --spring.config.name=myproject
```

使用 spring.config.location 属性重新定义配置文件的位置，它会覆盖掉默认的四个位置。使用 spring.config.additional-location 这个属性自定义配置文件位置，表示在默认的四个位置的基础上，再添加几个位置，新添加的位置的优先级大于原本的位置。 在自定义配置文件位置时，值一定要以 / 结尾。

```bash
# 下面的示例演示如何指定两个位置的配置文件：
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
# 属性重新定义配置文件的位置,值一定要以 / 结尾
$ java -jar myproject.jar --spring.config.location=classpath:/myconfig1/
# 追加配置文件目录,优先级大雨原本位置
$ java -jar myproject.jar --spring.config.additional-location=classpath:/myconfig2/
```

**备注：**spring.config.name 和 spring.config.location 会早早地使用，用于确定必须加载的文件。所以必须将它们定义为环境属性（通常是 OS 环境变量，系统属性或命令行参数）

如果 spring.config.location 包含目录，应以 / 结束（并且，在运行时，被附加到 spring.config.name 指定的名称后加载，包括特定的 profile 文件名）。

在 spring.config.location 中指定的文件按原样使用，不支持特定于配置文件的变量（{profile}），并且会被任何特定于配置文件（profile）的属性覆盖。



### 3、搜索顺序

#### 1、默认路径

默认情况下，配置文件的路径有 classpath:/, classpath:/config/, file:../, file:./config/，而搜索配置文件的顺序是相反的。实际的搜索顺序如下：

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/



#### 2、location

当通过 spring.config.location 自定义了配置文件的路径，则会替换认的路径。示例如下：

```properties
spring.config.location=classpath:/custom-config/,file:./custom-config/
```

上面示例的搜索顺序是：

1. file:./custom-config/
2. classpath:custom-config/



#### 3、additional-location

另外，当使用 spring.config.additional-location 设置自定义的配置路径时，自定义的的配置路径和默认的路径都会被使用。 自定义的配置路径的搜索会先于默认的配置路径。示例如下：

```properties
spring.config.additional-location=classpath:/custom-config/,file:./custom-config/
```

上面示例的搜索顺序是：

1. file:./custom-config/
2. file:custom-config/
3. file:./config/
4. file:./
5. classpath:/config/
6. classpath:/

此搜索顺序允许在一个配置文件中指定默认值，然后在另一个配置文件中有选择地覆盖这些值。

可以在默认位置之一的 application.properties（或使用 spring.config.name 选择的任何其他基本名称）中为应用程序提供默认值。然后，可以在运行时使用自定义位置之一中的的其他配置文件覆盖这些默认值。

**注意：**如果使用的是环境变量，而不是系统属性，大多数操作系统不允许使用点号分隔(`.`)的键名，但可以使用下划线（`_`）代替（示例：SPRING_CONFIG_NAME 代替 spring.config.name）。

如果应用程序运行在容器中，则可以使用 JNDI 属性（在 java:comp/env 中）或 Servlet 上下文初始化参数代替环境变量或系统属性。



## 5、Profile配置文件

除了 application.properties 文件外，还可以使用以下命名约定来定义特定于配置文件的属性：application-{profile}.properties。

Environment 有一组默认配置文件（默认情况下，`[default]`），如果未设置活动配置文件，则使用默认的。即，如果没有显式激活配置文件（profiles），则从application-default.properties 中加载属性。

加载 profiles 配置文件的位置与标准 application.properties 的位置相同，无论 profiles 配置文件是否在打包的 jar 内部或外部，profiles 配置文件的属性始终覆盖非 profiles 配置文件中的属性。

如果指定了多个 profiles 配置文件，则采用后赢策略。 例如，在通过 SpringApplication API 设置的配置文件之后，添加了spring.profiles.active属性指定的 profiles 配置文件，后者具有优先权。

**注意：**如果在 spring.config.location 中指定了任何配置文件，则不考虑使用 profiles 配置文件。如果还想使用 ，则将 profiles    配置文件存放在spring.config.location 目录中。



## 6、属性中的占位符

使用 application.properties 中的值时，它们会通过现有 Environment 进行过滤，因此可以参考以前面义的值（例如，从 System 属性中）

```properties
# a Spring Boot application
app.name=MyApp
app.description=${app.name}
```

**备注：**还可以使用此技术来创建现有 Spring Boot 属性的 简短 变体。 有关详细信息，参考：[Section 77.4, “Use ‘Short’ Command Line Arguments”](https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/howto-properties-and-configuration.html#howto-use-short-command-line-arguments)。



## 7、加密属性

Spring Boot 没有提供对属性值进行加密的任何内置支持，但是，它提供了修改 Spring Environment 中包含的值所需要的勾子。 EnvironmentPostProcessor 接口许在应用程序启动之前操做 Environment。有关详细信息，参考 [Section 76.3, “Customize the Environment or ApplicationContext Before It Starts”](https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/howto-spring-boot-application.html#howto-customize-the-environment-or-application-context)

如果正在寻找一种安全的方式来存储 **凭证** 或 **密码**，[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/) 项目为在 [HashiCorp Vault](https://www.vaultproject.io/) 中存储外部化配置提供了支持。



## 8、使用YAML文件

YAML 是 JSON 的超集，因此，其是一种用于具有层次结构来配置数据的便捷格式。 只要在类路径上具有 [SnakeYAML](https://bitbucket.org/asomov/snakeyaml) 库，SpringApplication 类就会自动支持 YAML 作为 properties 的替代方法。

**备注：**如果使用的是 SprongBoot Starters，SnakeYAML 由 spring-boot-starter 自动提供。

### 1、加载 YAML

Spring Framework 提供了两个方便的类，可用于加载 YAML 文件。YamlPropertiesFactoryBean 加载 YAML 文件做为 Properties，YamlMapFactoryBean 加载 YAML 文件作为 Map。

操作示例，如下 YAML 文件：

```yaml
environments:
	dev:
		url: https://dev.example.com
		name: Developer Setup
	prod:
		url: https://another.example.com
		name: My Cool App
```

上面示例转为 Properties 配置如下：

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

YAML 列表用 `[index]` 解引用器表示为属性键。 例如，考虑以下 YAML：

```yaml
my:
servers:
	- dev.example.com
	- another.example.com
```

上面示例转为 Properties 配置如下：

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

要使用 Spring Boot 的 Binder 工具（@ConfigurationProperties所做的）绑定到类似的属性，需要在类型为java.util.List（或Set）的目标 Bean 中拥有一个属性，或者需要提供一个 setter 或使用一个可变值对其进行初始化。 例如，以下示例绑定到前面显示的属性：

```java
@ConfigurationProperties(prefix="my")
public class Config {
	private List<String> servers = new ArrayList<String>();
	public List<String> getServers() {
		return this.servers;
	}
}
```

YamlPropertySourceLoader 类可用于在 Spring Environment 中将 YAML 公开为 PropertySource。 这样做可以让使用 @Value 注解和占位符语法来访问 YAML 属性。



### 2、多 profile YAML 文件

可以使用 spring.profiles 键在一个文件中指定多个 Profiles YAML 配置文件，以指示何时应用该文档，如以下示例所示：

```yaml
server:
	address: 192.168.1.100
---
spring:
	profiles: development
server:
	address: 127.0.0.1
---
spring:
	profiles: production & eu-central
server:
	address: 192.168.1.120
```

上面示例，如果激活了 development profile 文件，server.address属性值是 127.0.0.1。类似的，如果 production 和 eu-central profiles 激活，server.address 属性值是 192.168.1.120。如果 development, production, eu-central 都没有启用，则属性的值为 192.168.1.100。

**备注：**因此 spring.profiles 可以包含一个简单的 profile 文件名称（例如 production）或 profile 表达式。 一个 profile 表达式允许更复杂的 profile 文件逻辑，例如 `production & (eu-central | eu-west)`。 有关更多详细信息，请参阅 [参考指南](https://docs.spring.io/spring/docs/5.1.14.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。

如果在启动应用程序上下文时未显式激活任何配置，则会激活默认配置文件。 因此，在以下 YAML 中，我们为spring.security.user.password 设置了一个值，该值仅在 default 配置文件中可用：

```yaml
server:
  port: 8000
---
spring:
  profiles: default
  security:
    user:
      password: weak
```

在下面的示例中，始终设置密码，因为它没有附加到任何 *profile* 文件，并且必须在所有其他配置文件中根据需要显式重置密码：

```yaml
server:
  port: 8000
spring:
  security:
    user:
      password: weak
```

spring.profiles 元素指定的 Spring profile 文件可以通过使用 `!` 字符来表示否定。如果为单个文件同时指定了否定 (negated ) profile 文件和非否定 (non-negated) profile文件，则必须至少有一个**非否定**配置文件匹配，并且不能有**否定**配置文件匹配。



### 3、YAML 的缺点

缺点一：YAML 文件不能被 @PropertySource 注解加载。因此，如果需要以这种方式加载配置，则需要使用 properties 文件。

缺点二：在指定的 profile 文件的 YAML文件中使用多 YAML 文档语法可能会导致意外行为。 例如，考虑文件中的以下配置：

```yaml
# file-name: application-dev.yml
server:
  port: 8000
---
spring:
  profiles: "!test"
  security:
    user:
      password: "secret"
```

- 如果使用 `--spring.profiles.active=dev` 参数运行应用，期望的 security.user.password 被设置为 secret，但这种情况并非如此。
- 嵌套文档将被过滤，因为主文件名为 application-dev.yml，它已经被认为是 profile 文件的，并且嵌套文档将被忽略。
- **备注：**建议不要混用 profile 文件的 YAML 文件和多 个YAML 文档，坚持只使用其中之一。



## 9、类型安全配置属性

使用 **@Value(${property})** 注释注入配置属性有时会很麻烦，特别是在处理多个属性或数据本质上是分层的情况下。SpringBoot 提供了另一种处理属性的方法，这种方法允许强类型 Bean 控制和验证应用程序的配置，如下例所示：

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() { ... }

    public void setEnabled(boolean enabled) { ... }

    public InetAddress getRemoteAddress() { ... }

    public void setRemoteAddress(InetAddress remoteAddress) { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() { ... }

        public void setUsername(String username) { ... }

        public String getPassword() { ... }

        public void setPassword(String password) { ... }

        public List<String> getRoles() { ... }

        public void setRoles(List<String> roles) { ... }
    }
}
```

上面的 POJO 定义了如下属性：

- **acme.enable**：默认值为 false。
- **acme.remote-address**：可以使用 String 类型。
- **acme.security.username**：嵌套了 security 对象，其名称由属性名称决定。 特别是，返回类型根本不使用，可能是 `SecurityProperties`。
- **acme.security.password**。
- **acme.security.roles**：一个 String 集合。

**注意：**映射到 SpringBoot 中 @ConfigurationProperties 注解作用的类的属性（通过属性文件，YAML 文件，环境变量等进行配置）是 public API，但类本身的访问器（getter / setter）不是直接使用的。

**注意：**Getter 和 Setter 通常是必需的，因为绑定是通过标准 Java Beans 属性描述符进行的，就像 Spring MVC 一样。 在以下情况下，可以省略 Setter：

- 映射只要被初始化，就需要一个 getter，但不一定是 **setter**，因为绑定器可以对它们进行修改。

- 可以通过索引（通常使用 YAML）或使用单个逗号分隔的值（属性）访问集合和数组。在后一种情况下，setter 是必须的。官方建议始终为此类类型添加 **setter**。如果初始化集合，请确保它不是不可变的（如前一个示例所示）。

  如果嵌套的 POJO 属性已初始化（与前面示例中的 Security 字段类似），则不需要 **setter**。如果希望绑定器使用其默认构造方法动态创建实例，则需要一个 **setter**。

  有些人使用 Lombok 项目自动添加 **getters** 和 setters。 需要确保 Lombok 不会为这种类型生成任何特定的构造函数，因为容器会自动使用它来实例化该对象。

  最后，仅考虑标准 Java Bean 属性，并且不支持对静态属性的绑定。



# 参考文献 & 鸣谢

- Spring Boot 2系列(五十三)：Spring Boot 配置外部化：http://www.gxitsky.com/2020/04/18/SpringBoot-53-externalized-configuration/
- SpringBoot 配置文件加载优先级：https://blog.csdn.net/bluemysky/article/details/128097057