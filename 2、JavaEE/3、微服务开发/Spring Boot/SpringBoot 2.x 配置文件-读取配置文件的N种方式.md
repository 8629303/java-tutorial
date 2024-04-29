> SpringBoot的外部化配置：http://it.hzqiuxm.com/springboot%e6%95%99%e7%a8%8b%e7%b3%bb%e5%88%973/#i

# 一、Properties 配置文件的几种读取方式

## 0、配置文件绑定变量前言

Spring Boot 中读取配置文件有以下 5 种方法：

1. 使用 @Value 读取配置文件。
2. 使用 @ConfigurationProperties 读取配置文件。
3. 使用 Environment 读取配置文件。
4. 使用 @PropertySource 读取配置文件。
5. 使用原生方式读取配置文件。

其中最常用的是前 3 种，如果读取某一个配置项可使用 @Value，如果读取一组配置项可使用 @ConfigurationProperties，如果要指定读取某一个具体的配置文件可使用 @PropertySource 来指定。

它们的具体使用方法如下，为了方便测试，我们在 Spring Boot 配置文件 application.properties 添加以下内容：

```properties
profile.name=Spring Boot Profile
profile.desc=Spring Boot Profile Desc.
```



## 1、@Value 读取配置文件

使用 @Value 可以读取单个配置项，如下代码所示：

```java
@SpringBootApplication
public class DemoApplication implements InitializingBean {
    @Value("${profile.name}")
    private String name;

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("My Profile Name：" + name);
    }
}
```

以上程序的执行结果如下所示：

```
My Profile Name：Spring Boot Profile
```



## 2、@ConfigurationProperties 读取配置文件

@ConfigurationProperties 和 @Value 的使用略微不同，@Value 是读取单个配置项的，而 @ConfigurationProperties 是读取一组配置项的，我们可以使用 @ConfigurationProperties 加实体类读取一组配置项，如下代码所示：

```java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "profile")
@Data
public class Profile {
    private String name;
    private String desc;
}
```

其中 prefix 表示读取一组配置项的根 name，相当于 Java 中的类名，最后再把此配置类，注入到某一个类中就可以使用了，如下代码所示：

```java
@SpringBootApplication
public class DemoApplication implements InitializingBean {
    @Autowired
    private Profile profile;

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Profile Object:" + profile);
    }
}
```

以上程序的执行结果如下所示：

```
Profile Object:Profile(name=Spring Boot Profile, desc=Spring Boot Profile Desc.)
```



## 3、@PropertySource 读取配置文件

使用 @PropertySource 注解可以用来指定读取某个配置文件，比如指定读取 abc.properties 配置文件的配置内容，具体实现代码如下：

```java
@SpringBootApplication
@PropertySource("classpath:abc.properties")
public class TestApplication implements InitializingBean {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
    
    @Value("${profile.name}")
    private String name;
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Name：" + name);
    }
}
```

**中文乱码：**如果配置文件中出现中文乱码的情况，可通过指定编码格式的方式来解决中文乱码的问题，具体实现如下：

```java
@PropertySource(value = "dev.properties", encoding = "utf-8")
```

**注意事项：**@PropertySource 注解默认是只支持 properties 格式配置文件的读取的。**但是我们仍然有办法，跟着我继续。**

- 我们新建一个配置文件abc.yml，用来模拟第三方jar包的额外配置文件（非application配置文件）

  ```yaml
  profile:
    name: Spring Boot Profile
    desc: Spring Boot Profile Desc.
  ```

- 通过阅读代码了解到，DefaultPropertySourceFactory是进行配置文件加载的工厂类。

- 尽管其默认不支持读取YAML格式外部配置文件，但是我们可以通过继承DefaultPropertySourceFactory ，然后对它的createPropertySource进行一下改造。就可以实现YAML的“额外”配置文件加载。

  ```java
  public class MixPropertySourceFactory extends DefaultPropertySourceFactory {
      @Override
      public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource)
          throws IOException {
          String sourceName = name != null ? name : resource.getResource().getFilename();
  
          if (sourceName != null
              &&(sourceName.endsWith(".yml") || sourceName.endsWith(".yaml"))) {
              Properties propertiesFromYaml = loadYml(resource);
              // 将YML配置转成Properties之后，再用PropertiesPropertySource绑定
              return new PropertiesPropertySource(sourceName, propertiesFromYaml);
          } else {
              return super.createPropertySource(name, resource);
          }
      }
  
      // 将YML格式的配置转成Properties配置
      private Properties loadYml(EncodedResource resource) throws IOException{
          YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
          factory.setResources(resource.getResource());
          factory.afterPropertiesSet();
          return factory.getObject();
      }
  }
  ```

- 在类的上面加上如下注解即可。通过factory属性明确的指定使用我们自定义的MixPropertySourceFactory加载yml配置文件。

  ```java
  @PropertySource(value = {"classpath:family.yml"}, factory = MixPropertySourceFactory.class)
  ```



## 4、Environment 读取配置文件

Environment 是 Spring Core 中的一个用于读取配置文件的类，将此类使用 @Autowired 注入到类中就可以使用它的 getProperty 方法来获取某个配置项的值了，如下代码所示：

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.env.Environment;

@SpringBootApplication
public class TestApplication implements InitializingBean {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

    @Autowired
    private Environment environment;    
    
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Profile Name：" + environment.getProperty("profile.name"));
    }
}
```

以上程序的执行结果如下所示：

```
Profile Name：Spring Boot Profile
```



## 5、使用原生方式读取配置文件

我们还可以使用最原始的方式 Properties 对象来读取配置文件，如下代码所示：

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.util.Properties;

@SpringBootApplication
public class TestApplication implements InitializingBean {

    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        Properties props = new Properties();
        try {
            InputStreamReader inputStreamReader = new InputStreamReader(
                this.getClass().getClassLoader().getResourceAsStream("application.properties"),
                StandardCharsets.UTF_8);
            props.load(inputStreamReader);
        } catch (IOException e1) {
            System.out.println(e1);
        }
        System.out.println("Properties Name：" + props.getProperty("profile.name"));
    }
}
```

以上程序的执行结果如下所示：

```java
Properties Name：Spring Boot Profile
```



# 二、YAML 配置文件的几种读取方式

## 1、Environment 方式读取

在 Spring 中有一个类 Environment，它可以被认为是当前应用程序正在运行的环境，它继承了 PropertyResolver 接口，因此可以作为一个属性解析器使用。先创建一个 yml 文件，属性如下：

```yaml
person:
  name: hydra
  gender: male
  age: 18
```

用起来也非常简单，直接使用 @Autowired 就可以注入到要使用的类中，然后调用它的 getProperty() 方法就可以根据属性名称取出对应的值了。除了简单的获取外，Environment 提供的方法还可以对取出的属性值进行类型转换、以及默认值的设置。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.env.Environment;
import javax.annotation.PostConstruct;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

    @Autowired
    private Environment environment;

    @PostConstruct
    public void init() {
        System.out.println(environment.getProperty("person.name"));
        System.out.println(environment.getProperty("person.gender"));

        Integer autoClose = environment.getProperty("person.age", Integer.class);
        System.out.println(autoClose);
        String defaultValue = environment.getProperty("person.other", String.class, "defaultValue");
        System.out.println(defaultValue);
    }
}
```

打印结果如下

```
hydra
male
18
defaultValue
```

除了获取属性外，还可以用来判断激活的配置文件，我们先在 application.yml 中激活 pro 文件：

```yaml
spring:
  profiles:
    active: pro
person:
  name: hydra
  gender: male
  age: 18
```

可以通过 acceptsProfiles 方法来检测某一个配置文件是否被激活加载，或者通过getActiveProfiles方法拿到所有被激活的配置文件。测试接口：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.env.Environment;

import javax.annotation.PostConstruct;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

    @Autowired
    private Environment environment;

    @PostConstruct
    public void init() {
        System.out.println(environment.acceptsProfiles("pro"));
        System.out.println(environment.acceptsProfiles("dev"));

        String[] activeProfiles = environment.getActiveProfiles();
        for (String activeProfile : activeProfiles) {
            System.out.println(activeProfile);
        }
    }
}
```

打印结果：

```
true
false
pro
```



## 2、YamlPropertiesFactoryBean 方式读取

在 Spring 中还可以使用 YamlPropertiesFactoryBean 来读取自定义配置的 yml 文件，而不用再被拘束于 application.yml 及其激活的其他配置文件。在使用过程中，只需要通过 setResources() 方法设置自定义 yml 配置文件的存储路径，再通过 getObject() 方法获取 Properties 对象，后续就可以通过它获取具体的属性，下面看一个例子：

```yaml
spring:
  profiles:
    active: pro
person2:
  name: hydra
  gender: male
  age: 18
```

```java
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.io.ClassPathResource;
import javax.annotation.PostConstruct;
import java.util.Properties;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

    @PostConstruct
    public void init() {
        YamlPropertiesFactoryBean yamlProFb = new YamlPropertiesFactoryBean();
        yamlProFb.setResources(new ClassPathResource("application2.yml"));
        Properties properties = yamlProFb.getObject();
        System.out.println(properties.get("person2.name"));
        System.out.println(properties.get("person2.gender"));
        System.out.println(properties);
    }
}
```

打印结果：

```
hydra
male
{person2.gender=male, person2.age=18, spring.profiles.active=pro, person2.name=hydra}
```

但是这样的使用中有一个问题，那就是只有在这个接口的请求中（或者这个方法）能够取到这个属性的值，如果再写一个接口（或者方法），不使用 YamlPropertiesFactoryBean 读取配置文件，即使之前的方法已经读取过这个 yml 文件一次了，第二个接口取到的仍然还是空值。来对这个过程进行一下测试：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.annotation.Order;
import org.springframework.core.io.ClassPathResource;
import javax.annotation.PostConstruct;
import java.util.Properties;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

    // 越小越先执行
    @Order(-1)
    @PostConstruct
    public void init() {
        System.out.println("@Order(-1)先执行....");
        YamlPropertiesFactoryBean yamlProFb = new YamlPropertiesFactoryBean();
        yamlProFb.setResources(new ClassPathResource("application2.yml"));
        Properties properties = yamlProFb.getObject();
        System.out.println(properties.get("person2.name"));
        System.out.println(properties.get("person2.gender"));
        System.out.println(properties);
    }

    @Value("${person2.name:null}")
    private String name;
    @Value("${person2.gender:null}")
    private String gender;

    @Order(1)
    @PostConstruct
    public void initTwo() {
        System.out.println("@Order(1)后执行....");
        System.out.println(name);
        System.out.println(gender);
    }
}
```

打印结果：

```
@Order(-1)先执行....
hydra
male
{person2.gender=male, person2.age=18, spring.profiles.active=pro, person2.name=hydra}
@Order(1)后执行....
null
null
```

想要解决这个问题也很简单，可以配合 PropertySourcesPlaceholderConfigurer 使用，它实现了 BeanFactoryPostProcessor 接口，也就是一个 bean 工厂后置处理器的实现，可以将配置文件的属性值加载到一个 Properties 文件中。使用方法如下：

```java
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.io.ClassPathResource;

@Configuration
public class PropertyConfig {
    @Bean
    public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
        PropertySourcesPlaceholderConfigurer configurer
                = new PropertySourcesPlaceholderConfigurer();
        YamlPropertiesFactoryBean yamlProFb
                = new YamlPropertiesFactoryBean();
        yamlProFb.setResources(new ClassPathResource("application2.yml"));
        configurer.setProperties(yamlProFb.getObject());
        return configurer;
    }
}
```

再次启动上面的主方法，结果如下，可以正常的取到 application2.yml 中的属性：

```
@Order(-1)先执行....
hydra
male
{person2.gender=male, person2.age=18, spring.profiles.active=pro, person2.name=hydra}
@Order(1)后执行....
hydra
male
```

除了使用 YamlPropertiesFactoryBean 将 yml 解析成 Properties 外，其实我们还可以使用 YamlMapFactoryBean 解析 yml 成为`Map`，使用方法非常类似：

```java
import org.springframework.beans.factory.config.YamlMapFactoryBean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.io.ClassPathResource;
import javax.annotation.PostConstruct;
import java.util.Map;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

    @PostConstruct
    public void init() {
        YamlMapFactoryBean yamlMapFb = new YamlMapFactoryBean();
        yamlMapFb.setResources(new ClassPathResource("application2.yml"));
        Map<String, Object> map = yamlMapFb.getObject();
        System.out.println(map);
    }
}
```

打印结果：

```
{spring={profiles={active=pro}}, person2={name=hydra, gender=male, age=18}}
```



## 3、ApplicationListener 方式读取

在上篇介绍原理的文章中，我们知道 SpringBoot 是通过监听事件的方式来加载和解析的yml文件，那么我们也可以仿照这个模式，来加载自定义的配置文件。

首先，定义一个类实现ApplicationListener接口，监听的事件类型为ApplicationEnvironmentPreparedEvent，并在构造方法中传入要解析的yml文件名，自定义的监听器中需要实现接口的`onApplicationEvent()`方法，当监听到ApplicationEnvironmentPreparedEvent事件时会被触发：

```java
import org.springframework.boot.context.event.ApplicationEnvironmentPreparedEvent;
import org.springframework.boot.env.YamlPropertySourceLoader;
import org.springframework.context.ApplicationListener;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.DefaultResourceLoader;
import org.springframework.core.io.ResourceLoader;
import java.io.IOException;
import java.util.List;

public class YmlListener implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {
    private String ymlFilePath;

    public YmlListener(String ymlFilePath) {
        this.ymlFilePath = ymlFilePath;
    }

    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        ConfigurableEnvironment environment = event.getEnvironment();
        ResourceLoader loader = new DefaultResourceLoader();
        YamlPropertySourceLoader ymlLoader = new YamlPropertySourceLoader();
        try {
            List<PropertySource<?>> sourceList = ymlLoader.load(ymlFilePath, loader.getResource(ymlFilePath));
            for (PropertySource<?> propertySource : sourceList) {
                environment.getPropertySources().addLast(propertySource);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

上面的代码中，主要实现了：

- 获取当前环境Environment，当ApplicationEnvironmentPreparedEvent事件被触发时，已经完成了Environment的装载，并且能够通过event事件获取
- 通过YamlPropertySourceLoader加载、解析配置文件
- 将解析完成后的OriginTrackedMapPropertySource添加到Environment中

修改启动类，在启动类中加入这个监听器：

```java
import com.example.test.config.YmlListener;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import javax.annotation.PostConstruct;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(TestApplication.class);
        application.addListeners(new YmlListener("classpath:/application2.yml"));
        application.run(args);
    }

    @Value("${person2.name:null}")
    private String name;
    @Value("${person2.gender:null}")
    private String gender;
    @PostConstruct
    public void init() {
        System.out.println(name);
        System.out.println(gender);
    }
}
```

> 在向environment中添加propertySource前加一个断点，查看环境的变化：main.run() »» prepareEnvironment() »» configurePropertySources »» environment.getPropertySources()，执行完成后，可以看到配置文件源已经被添加到了环境中。

启动完成后再调用一下接口，查看结果：

```
hydra
male
```

能够正确的取到配置文件中的值，说明自定义的监听器已经生效。



## 4、SnakeYml 方式读取

前面介绍的几种方式，在 Spring 环境下无需引入其他依赖就可以完成的，接下来要介绍的SnakeYml在使用前需要引入依赖，但是同时也可以脱离Spring环境单独使用。先引入依赖坐标：

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.23</version>
</dependency>
```

准备一个yml配置文件：

```yaml
person1:
  name: hydra
  gender: male
person2:
  name: susan
  gender: female
```

在使用SnakeYml解析yml时，最常使用的就是load、loadlAll、loadAs方法，这三个方法可以加载yml文件或字符串，最后返回解析后的对象。我们先从基础的load方法开始演示：

```java
import org.yaml.snakeyaml.Yaml;
import java.util.Map;

public class TestMain {
    public static void main(String[] args) {
        Yaml yaml = new Yaml();
        Map<String, Object> map = yaml.load(TestMain.class.getClassLoader().getResourceAsStream("snake1.yml"));
        System.out.println(map);
    }
}
```

运行上面的代码，打印Map中的内容：

```
{person1={name=hydra, gender=male}, person2={name=susan, gender=female}}
```

接下来看一下loadAll方法，它可以用来加载yml中使用`---`连接符连接的多个文档，将上面的yml文件进行修改：

```yaml
person1:
  name: hydra
  gender: male
---
person2:
  name: susan
  gender: female
```

在添加了连接符后，尝试再使用load方法进行解析，报错如下显示发现了另一段yml文档从而无法正常解析：

```shell
Exception in thread "main" expected a single document in the stream
 in 'reader', line 1, column 1:
    person1:
    ^
but found another document
 in 'reader', line 4, column 1:
    ---
    ^

	at org.yaml.snakeyaml.composer.Composer.getSingleNode(Composer.java:114)
	at org.yaml.snakeyaml.constructor.BaseConstructor.getSingleData(BaseConstructor.java:139)
	at org.yaml.snakeyaml.Yaml.loadFromReader(Yaml.java:524)
	at org.yaml.snakeyaml.Yaml.load(Yaml.java:452)
	at com.example.test.TestMain.main(TestMain.java:9)
```

这时候修改上面的代码，使用loadAll方法：

```java
import org.yaml.snakeyaml.Yaml;

public class TestMain {
    public static void main(String[] args) {
        Yaml yaml = new Yaml();
        Iterable<Object> objects = yaml.loadAll(TestMain.class.getClassLoader().getResourceAsStream("snake2.yml"));
        for (Object object : objects) {
            System.out.println(object);
        }
    }
}
```

执行结果如下：

```
{person1={name=hydra, gender=male}}
{person2={name=susan, gender=female}}
```

可以看到，loadAll方法返回的是一个对象的迭代，里面的每个对象对应yml中的一段文档，修改后的yml文件就被解析成了两个独立的Map。

接下来再来看一下loadAs方法，它可以在yml解析过程中指定类型，直接封装成一个对象。我们直接复用上面的`snake1.yml`，在解析前先创建两个实体类对象用于接收：

```java
@Data
public class Person {
    SinglePerson person1;
    SinglePerson person2;
}

@Data
public class SinglePerson {
    String name;
    String gender;
}
```

下面使用loadAs方法加载yml，注意方法的第二个参数，就是用于封装yml的实体类型。

```java
import org.yaml.snakeyaml.Yaml;

public class TestMain {
    public static void main(String[] args) {
        Yaml yaml=new Yaml();
        Person person = yaml
            .loadAs(TestMain.class.getClassLoader().getResourceAsStream("snake1.yml"), Person.class);
        System.out.println(person.toString());
    }
}
```

查看执行结果：

```
Person(person1=SinglePerson(name=hydra, gender=male), person2=SinglePerson(name=susan, gender=female))
```

实际上，如果想要将yml封装成实体对象，也可以使用另一种方法。在创建Yaml对象的时候，传入一个指定实体类的构造器对象，然后直接调用load方法就可以实现：

```java
import org.yaml.snakeyaml.Yaml;
import org.yaml.snakeyaml.constructor.Constructor;

public class TestMain {
    public static void main(String[] args) {
        Yaml yaml = new Yaml(new Constructor(Person.class));
        Person person = yaml.load(TestMain.class.getClassLoader().getResourceAsStream("snake1.yml"));
        System.out.println(person.toString());
    }
}
```

执行结果与上面相同：

```
Person(person1=SinglePerson(name=hydra, gender=male), person2=SinglePerson(name=susan, gender=female))
```

SnakeYml其实实现了非常多的功能，这里就不一一列举了，有兴趣的小伙伴可以自己查看一下文档。如果你看了上一篇的文章后跟着翻阅了一下源码，那么你会发现，其实在SpringBoot的底层，也是借助了SnakeYml来进行的yml的解析操作。



## 5、jackson-dataformat-yaml 方式读取

相比大家平常用jackson比较多的场景是用它来处理json，其实它也可以用来处理yml，使用前需要引入依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
    <version>2.12.3</version>
</dependency>
```

使用jackson读取yml也非常简单，这里用到了常用的ObjectMapper，在创建ObjectMapper对象时指定使用YAML工厂，之后就可以简单的将yml映射到实体：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;
import lombok.Data;
import java.io.FileInputStream;
import java.io.InputStream;

public class TestMain {
    public static void main(String[] args) throws Exception {
        // 读取操作
        ObjectMapper objectMapper = new ObjectMapper(new YAMLFactory());
        InputStream input = new FileInputStream("D:\\work\\Test\\src\\main\\resources\\snake1.yml");
        Person person = objectMapper.readValue(input, Person.class);
        System.out.println(person.toString());
    }
}
@Data
class Person {
    SinglePerson person1;
    SinglePerson person2;
}
@Data
class SinglePerson {
    String name;
    String gender;
}
```

运行结果：

```
Person(person1=SinglePerson(name=hydra, gender=male), person2=SinglePerson(name=susan, gender=female))
```

如果想要生成yml文件的话，可以调用ObjectMapper的writeValue方法实现：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.io.File;
import java.util.HashMap;
import java.util.Map;

public class TestMain {
    public static void main(String[] args) throws Exception {
        Map<String, Object> map = new HashMap<>();
        SinglePerson person1 = new SinglePerson("Trunks", "male");
        SinglePerson person2 = new SinglePerson("Goten", "male");
        Person person = new Person(person1, person2);
        map.put("person", person);

        ObjectMapper objectMapper = new ObjectMapper(new YAMLFactory());
        objectMapper.writeValue(new File("jackson-gen.yml"), map);
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class Person {
    SinglePerson person1;
    SinglePerson person2;
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class SinglePerson {
    String name;
    String gender;
}
```

查看生成的yml文件，可以看到 jackson 对字符串类型严格的添加了引号，还在文档的开头添加了 yml 的链接符。至于其他 jackson 读写 yml 的复杂功能，大家可以在工作中自己去探索使用。

```yaml
---
person:
  person1:
    name: "Trunks"
    gender: "male"
  person2:
    name: "Goten"
    gender: "male"
```

