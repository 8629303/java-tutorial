 

> 基于Maven的Profiles多环境配置【[怪 咖@](https://blog.csdn.net/weixin_43888891)】：https://blog.csdn.net/weixin_43888891/article/details/130794308

一个项目通常都会有多个不同的运行环境，例如开发环境，测试环境、生产环境等。而不同环境的构建过程很可能是不同的，例如数据源配置、插件、以及依赖的版本等。每次将项目部署到不同的环境时，都需要修改相应的配置，这样重复的工作，不仅浪费劳动力，还容易出错。为了解决这一问题，Maven 引入了 Profile 的概念，通过它可以为不同的环境定制不同的构建过程。



# 一、Profile 的类型

Profile 可以分为 3 个类型，它们的作用范围也各不相同。

| 类型        | 位置                                              | 有效范围                            |
| ----------- | ------------------------------------------------- | ----------------------------------- |
| Per Project | Maven 项目的 pom.xml 汇总                         | 只对当前项目有效                    |
| Per User    | 用户主目录（%USER_HOME%）/.m2/setting,xml         | 对本机上该用户所有的 Maven 项目有效 |
| Global      | Maven 安装目录（%MAVEN_HOME%）/config/setting.xml | 对本机上所有 Maven 项目有效         |

关于 Per User 有的电脑是没有的，他是 IDEA 不配置 MAVEN 情况下的一个默认 MAVEN 配置，如果 IDEA 配置了 MAVEN 可能会没有，直接忽略即可。



# 二、Profile 的声明

Maven 通过 profiles 元素来声明一组 Profile 配置，该元素下可以包含多个 profile 子元素，每个 profile 元素表示一个 Profile 配置。每个 profile 元素中通常都要包含一个 id 子元素，该元素是调用当前 Profile 的标识。

```xml
<profiles>
    <profile>
        <id>profile id</id>
        ....
    </profile>

    <profile>
        <id>profile id</id>
        ....
    </profile>
</profiles>
```

除此之外，Profile 中还可以声明一些其他的 POM 元素，但不同位置的 Profile 所能声明的 POM 元素也是不同的。

1、在 pom.xml 中声明的 Profile，由于其能够随着 pom.xml 一起存在，所以它能够修改或增加很多 POM 元素，其中常用的元素如下表：

| 一级    | 二级                   | 三级             |
| ------- | ---------------------- | ---------------- |
| project |                        |                  |
|         | repositories           |                  |
|         | pluginRepositories     |                  |
|         | dependencies           |                  |
|         | dependencyManagement   |                  |
|         | distributionManagement |                  |
|         | profiles               |                  |
|         | modules                |                  |
|         | reporting              |                  |
|         | build                  |                  |
|         |                        | plugins          |
|         |                        | pluginManagement |
|         |                        | defaultGoal      |
|         |                        | directory        |
|         |                        | finalName        |
|         |                        | resources        |
|         |                        | directories      |
|         |                        | extensions       |

2、在 setting.xml 中声明的 Profile 是无法保证能够随着 pom.xml 一起被分发的，因此 Maven 不允许用户在该类型的 Profile 修改或增加**依赖**或**插件**等配置信息，它只能声明以下范围较为宽泛的元素：

*   activation：激活 profile 的规则
*   repositories：仓库配置。
*   pluginRepositories：插件仓库配置。
*   properties：键值对，该键值对可以在 pom.xml 中使用。



# 三、需要注意的地方

我们所做的示例当中可能会经常修改 pom.xml，有时候修改的 pom.xml 并没有生效，或者是我们新增了配置文件，然后并没有生效。最好点击一次 IDEA/MAVEN 标签下的 Reload All Maven Projects。 

使用 Profile 配合 application 替换值，假如打包的时候尽量要跳过测试阶段：`mvn clean install -DskipTests`

# 四、激活 Profile

构建项目的时候，可以通过多种方式激活 Profile，以实现不同环境使用不同的配置，执行不同的构建过程。

**Profile 可以通过以下 6 种方式激活：**

*   命令行激活
*   settings.xml 文件显示激活
*   系统属性激活
*   操作系统环境激活
*   文件存在与否激活
*   默认激活

下面我们以一个 Maven 项目为例，分别对以上 6 种激活方式进行介绍。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.12</version>
        <relativePath/>
    </parent>
    <groupId>com.gzl.cn</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>demo</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <profiles>
        <!-- dev -->
        <profile>
            <id>dev</id>
            <properties>
                <username>zhangsan</username>
                <profile.active>dev</profile.active>
            </properties>
        </profile>
        <!-- test -->
        <profile>
            <id>test</id>
            <properties>
                <username>lisi</username>
                <profile.active>test</profile.active>
            </properties>
        </profile>
        <!-- on -->
        <profile>
            <id>prod</id>
            <properties>
                <username>wangwu</username>
                <profile.active>prod</profile.active>
            </properties>
        </profile>
    </profiles>

</project>
```

> 这里我们使用到了`@arg@`来取pom当中properties标签的变量值，假如项目没有继承 spring-boot-starter-parent，那么取值就应该使用`${...}`的语法，至于想了解为什么的或者是细节的可以看一下这篇文章，本篇重心是 Profiles：https://blog.csdn.net/weixin_43888891/article/details/130755755

使用@arg@语法获取pom.xml 中的 properties 的值。如下是 application.properties 文件：

```properties
spring.profiles.active=@profile.active@
username=@username@
```



## 1、命令行激活

我们可以在命令行中使用 mvn 命令行参数 `-P` 加上 Profile 的 id 来激活 Profile，多个 id 之间使用逗号隔开。例如，激活 test1 和 test2 两个 Profile, 命令如下：

```bash
mvn clean install -Ptest1,test2
```

打开命令行窗口，跳转到 pom.xml 所在的目录，执行以下 mvn 命令，激活 id 为 test 的 Profile。

```bash
mvn clean package -Ptest
```

这时候查看编译过后的 application.properties，打开 target/classes 下的 application.properties 文件：

```properties
spring.profiles.active=test
username=lisi
```



## 2、settings.xml 文件显示激活

在 Maven 安装目录的 .m/settings.xml 文件中添加如下配置，激活指定的 Profile。直接添加到 settings.xml 最下方即可。

```xml
<setting>
    <activeProfiles>
        <activeProfile>dev</activeProfile>
    </activeProfiles>
</setting>
```

这次执行命令就不需要`-P`来指定了

```bash
mvn clean package
```

这时候查看编译过后的 application.properties，打开 target/classes 下的 application.properties 文件：

```properties
spring.profiles.active=dev
username=zhangsan
```





## 3、系统属性激活

用户可以配置当某个系统属性存在时，激活指定的 Profile。例如，我们在 id 为 prod 的 profile 元素中添加以下配置，表示当系统属性 user 存在，且值等于 prod 时，自动激活该 Profile。

```xml
<activation>
    <property>
        <name>user</name>
        <value>prod</value>
    </property>
</activation>
```

```xml
<!-- on -->
<profile>
    <id>prod</id>
    <activation>
        <property>
            <name>user</name>
            <value>prod</value>
        </property>
    </activation>
    <properties>
        <username>wangwu</username>
        <profile.active>prod</profile.active>
    </properties>
</profile>
```
打开命令行窗口，跳转到 pom.xml 所在的目录。执行命令，其中参数 `-D` 选项用来指定系统临时属性。

```bash
mvn clean package -Duser=prod
```

这时候查看编译过后的 application.properties，打开 target/classes 下的 application.properties 文件：

```properties
spring.profiles.active=prod
username=wangwu
```



## 4、操作系统环境激活

Maven 还可以根据操作系统环境自动激活指定的 Profile。例如，将以下配置（本地计算机操作系统环境信息）添加到 pom.xml 文件中的 id 为 dev的 Profile 中。

```xml
<activation>
    <os>
        <name>Windows 10</name>
        <family>Windows</family>
        <arch>amd64</arch>
        <version>10.0</version>
    </os>
</activation>
```

```xml
<!-- dev -->
<profile>
    <id>dev</id>
    <activation>
        <os>
            <name>Windows 10</name>
            <family>Windows</family>
            <arch>amd64</arch>
            <version>10.0</version>
        </os>
    </activation>
    <properties>
        <username>zhangsan</username>
        <profile.active>dev</profile.active>
    </properties>
</profile>
```

这时候查看编译过后的 application.properties，打开 target/classes 下的 application.properties 文件：

```properties
spring.profiles.active=dev
username=zhangsan
```

如果真的涉及到需要根据操作系统来选择对应的配置可以使用这个实现，family 标签是操作系统，关于 family 都有哪些取值可以看源码：

- https://github.com/sonatype/plexus-utils/blob/f2beca21c75084986b49b3ab7b5f0f988021dcea/src/main/java/org/codehaus/plexus/util/Os.java#L72



## 5、文件存在与否激活

Maven 可以根据某些文件存在与否，来决定是否激活 Profile。例如，在 id 为 prod 的 Profile 中添加以下配置，该配置表示当 env.prod.properties 文件存在，且 env.test.properties 文件不存在时，激活该 Profile。

```xml
<activation>
    <file>
        <exists>./src/main/resources/env.prod.properties</exists>
        <missing>./src/main/resources/env.test.properties</missing>
    </file>
</activation>
```

```xml
<!-- dev -->
<profile>
    <id>dev</id>
    <activation>
        <file>
            <exists>./src/main/resources/env.prod.properties</exists>
            <missing>./src/main/resources/env.test.properties</missing>
        </file>
    </activation>
    <properties>
        <username>zhangsan</username>
        <profile.active>dev</profile.active>
    </properties>
</profile>
```



## 6、默认激活

我们可以在声明 Profile 时，指定其默认激活。例如，在 id 为 normal 的 Profile 中添加以下配置，指定该 Profile 默认激活。

```xml
<activation>
    <activeByDefault>true</activeByDefault>
</activation>
```

```xml
<!-- dev -->
<profile>
    <id>dev</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <username>zhangsan</username>
        <profile.active>dev</profile.active>
    </properties>
</profile>
```



## 7、根据 jdk 版本激活

```xml
<activation>
    <jdk>11</jdk>
</activation>
```

```xml
<!-- dev -->
<profile>
    <id>jdk-11</id>
    <activation>
        <jdk>11</jdk>
    </activation>
    <properties>
        <username>zhangsan</username>
        <profile.active>dev</profile.active>
    </properties>
</profile>
```



## 8、IDEA 当中指定 profiles

Maven 当中配置了多少 profiles 在这里都可以看到，而且我们还能选中后启动项目。本质上在这选中指定的 profiles 然后启动项目应该就是使用的`-P`命令，只不过 IDEA 进行了可视化。

IDEA | Maven | Profiles | 勾选需要激活的 profile-id



# 五、实战中多环境配置的几种方式

利用 Profiles 可以完成多环境配置，多环境配置又有很多种配置方式，接下来我会写几个案例供大家参考

## 1、利用Antrun插件复制配置文件

添加两个配置文件，注意这里并没有使用 application.properties

- application-dev.properties

  ```properties
  server.port=8080
  ```

- application-prod.properties

  ```properties
  server.port=8090
  ```

然后 pom.xml 配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/>
    </parent>
    <groupId>org.example</groupId>
    <artifactId>lsx-project</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>lsx-project</name>
    <description>lsx-project</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
                <!-- 排除配置文件，因为我们是利用的插件生成的application.properties配置文件，那么这些带有-环境的配置文件就可以不用携带了 -->
                <excludes>
                    <exclude>**/application*.properties</exclude>
                </excludes>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <profiles>
        <!-- test 环境配置 -->
        <profile>
            <id>dev</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-antrun-plugin</artifactId>
                        <version>3.0.0</version>
                        <executions>
                            <execution>
                                <!-- 生命周期 -->
                                <phase>compile</phase>
                                <!-- 插件目标 -->
                                <goals>
                                    <goal>run</goal>
                                </goals>
                                <configuration>
                                    <target>
                                        <!-- 输出 -->
                                        <echo>application-dev.properties, 将其配置信息复制到当前项目target\classes\application.properties中</echo>
                                        <!-- 在 target\calsses 目录下生成 application.properties -->
                                        <!-- application-dev.properties 的内容复制到 application.properties 中 -->
                                        <copy file="src/main/resources/application-dev.properties"
                                              tofile="${project.build.outputDirectory}/application.properties"/>
                                    </target>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <!-- 生产环境配置 -->
        <profile>
            <id>prod</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-antrun-plugin</artifactId>
                        <version>3.0.0</version>
                        <executions>
                            <execution>
                                <phase>compile</phase>
                                <goals>
                                    <goal>run</goal>
                                </goals>
                                <configuration>
                                    <target>
                                        <echo>application-prod.properties,将其配置信息复制到当前项目target\classes\application.properties中</echo>
                                        <!-- 在 target\calsses 目录下生成 application.properties -->
                                        <!--  application-prod.properties 的内容复制到 application.properties 中-->
                                        <copy file="src/main/resources/application-prod.properties"
                                              tofile="${project.build.outputDirectory}/application.properties"/>
                                    </target>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```
执行如下命令激活prod配置

```bash
mvn clean compile spring-boot:run -Pprod
```

启动日志如下，可以看到启动的服务端口为8090，是prod配置的。

```
2024-09-27 00:07:38.744  INFO 73613 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8090 (http) with context path ''
```

这时候我们也可以发现 target/classes 目录下只有我们生成的 application.properties，原因是我通过 resources 标签设置了那两个文件不参与编译和打包。

```properties
server.port=8090
```





## 2、利用Spring提供的profiles.active

在这个示例当中我基于上面的示例又新增了一个 application.properties。profile.active 是变量，可以取 xml 当中 profile 当中配置的 profile.active 标签。

- application.properties

  ```properties
  spring.profiles.active=@profile.active@
  ```

- application-dev.properties

  ```properties
  server.port=8080
  ```

- application-prod.properties

  ```properties
  server.port=8090
  ```

配置文件当中 spring.profiles.active 是 SpringBoot给我们提供的，假如 spring.profiles.active=dev 那么 application-dev.properties 配置文件就会生效。

关于SpringBoot多环境配置：https://blog.csdn.net/weixin_43888891/article/details/110993428

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/>
    </parent>
    <groupId>org.example</groupId>
    <artifactId>lsx-project</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>lsx-project</name>
    <description>lsx-project</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/resources/</directory>
                <filtering>true</filtering>
                <includes>
                    <include>**/application.properties</include>
                    <include>**/application-${profile.active}.properties</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources/</directory>
                <excludes>
                    <exclude>**/*.properties</exclude>
                </excludes>
            </resource>
        </resources>
    </build>
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <!-- 自定义节点profile.active -->
                <profile.active>dev</profile.active>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profile.active>prod</profile.active>
            </properties>
        </profile>
    </profiles>
</project>
```
执行如下命令激活dev配置

```bash
mvn clean compile spring-boot:run -Pdev
```

启动日志如下，可以看到启动的服务端口为8080，是 dev 配置的。

```
2024-09-27 00:07:38.744  INFO 73613 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
```

观察编译路径，当我们生效了dev，那么 prod 文件就不会被编译打包。这就是我们 xml 当中配置的 resources 标签所起到的作用！

- application.properties

  ```properties
  spring.profiles.active=dev
  ```

- application-dev.properties

  ```properties
  server.port=8080
  ```

> 可能有的人该好奇了，明明使用 spring.profiles.active 就足以完成多文件配置了，为什么还要在 pom.xml 当中配置 profile，其实我们这里配置的 profile 和 resources 标签就起了一个作用，假如选择 dev，那么 prod 就不参与打包，这样别人通过反编译就看不到我们prod的配置文件了！并且 Maven profiles 还有其他更加强大的功能。
>



## 3、直接将配置放到 POM 当中

上面两种都是以多配置文件的形式，这一种是直接将多环境配置都放到pom.xml当中。

- application.properties，继续使用@@变量取值

  ```properties
  spring.profiles.active=@profile.active@
  ```

所有相关的配置都可以放到 properties 标签下，然后以`@标签@`语法在 application.properties 当中取值

```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <server.port>8080</server.port>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <server.port>8090</server.port>
        </properties>
    </profile>
</profiles>
```

注意我们只是用来测试，所以并没有放很多配置，实际开发当中数据库相关配置都可以在`<properties>`下添加，然后通过`@@`取值即可。

```bash
mvn clean compile spring-boot:run -Pdev
```

启动日志如下，可以看到启动的服务端口为8080，是 dev 配置的。

```
2024-09-27 00:07:38.744  INFO 73613 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
```

这时候我们也可以查看 target/classes 目录下的 application.properties。

```properties
server.port=8080
```