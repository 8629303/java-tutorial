# 一、Maven 配置默认的 JDK 版本

在 pom.xml 中配置默认的 JDK 版本主要通过配置 Maven 的 maven-compiler-plugin 插件来完成。可以通过指定 source 和 target 的 Java 版本来确保项目的编译使用指定的 JDK 版本。

方式一：使用 maven-compiler-plugin 显示配置

```xml
<project>
    <!-- 项目的基础配置 -->
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 在 properties 中指定 JDK 版本 -->
    <properties>
        <maven.compiler.source>17</maven.compiler.source>   <!-- 设置源代码版本 -->
        <maven.compiler.target>17</maven.compiler.target>   <!-- 设置目标代码版本 -->
        <maven.compiler.release>17</maven.compiler.release> <!-- 设置 JDK 版本 -->
    </properties>

    <!-- Maven 编译器插件配置 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version> <!-- 使用的插件版本 -->
                <configuration>
                    <source>${maven.compiler.source}</source>    <!-- 指定源码的 JDK 版本 -->
                    <target>${maven.compiler.target}</target>    <!-- 指定目标代码的 JDK 版本 -->
                    <release>${maven.compiler.release}</release> <!-- 指定 JDK 版本 -->
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

方式二：`maven-compiler-plugin` 可以通过 Maven 的属性来进行隐式配置，而不需要在 `pom.xml` 中显式声明插件的配置

```xml
<!-- 在 properties 中指定 JDK 版本 -->
<properties>
    <maven.compiler.source>17</maven.compiler.source>   <!-- 设置源代码版本 -->
    <maven.compiler.target>17</maven.compiler.target>   <!-- 设置目标代码版本 -->
    <maven.compiler.release>17</maven.compiler.release> <!-- 设置 JDK 版本 -->
    <encoding>UTF-8</encoding>                          <!-- 指定编译时的编码 -->
</properties>
```



# 二、maven-compiler-plugin 为什么可以隐式配置

在 pom.xml 中，你经常会看到仅仅通过 < properties > 配置 JDK 版本，而没有配置 < plugin >，是因为 Maven 的默认行为和插件的继承机制。



## 1、为什么只配置 < properties > 就能生效？

### 1、Maven 插件的默认继承与生命周期绑定
Maven 自带了**默认插件绑定**，其中 maven-compiler-plugin 是与编译生命周期阶段绑定的默认插件。Maven 在构建项目时，会自动调用 maven-compiler-plugin 来编译 Java 源代码，而不需要显式地在 < plugins > 部分声明它。



### 2、Maven 默认插件版本管理

如果没有显式指定 maven-compiler-plugin 版本，Maven 会根据你所使用的 Maven 版本来自动管理插件的版本（由 Maven Super POM 来定义）。因此，许多情况下，开发者不需要手动配置这个插件，Maven 会自动处理插件加载和版本选择。



### 3、< properties > 配置的作用

< properties > 部分是 Maven 用来配置一些全局属性的地方，例如 maven.compiler.source 和 maven.compiler.target。这两个属性被 Maven 默认使用的 maven-compiler-plugin 识别，所以即使没有显式声明插件，Maven 也会读取这些属性，并使用它们来控制编译器行为。

Maven 的默认行为会自动把这些属性应用到 maven-compiler-plugin 上，所以通常你只需在 < properties > 中定义它们，Maven 就会正确地应用这些配置。



### 4、release 属性的自动支持

自 Maven 3.6.0 版本和 maven-compiler-plugin 3.6.0 版本开始，release 这个属性被支持。只需要在 < properties > 中定义，Maven 也会自动应用这个配置。

自 JDK 9 引入，用于指定编译时的 Java API 版本，同时控制 source 和 target。推荐使用它来避免不必要的编译错误。



## 2、隐式配置的典型例子

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>   <!-- 使用 JDK 17 编译 -->
    <maven.compiler.target>17</maven.compiler.target>   <!-- 生成 JDK 17 兼容的字节码 -->
    <maven.compiler.release>17</maven.compiler.release> <!-- 确保与 JDK 17 API 兼容 -->
    <encoding>UTF-8</encoding>                          <!-- 指定编译时的编码 -->
</properties>
```



### 1、当你只配置 < properties > 时，Maven 实际会做什么？

- Maven 在后台使用默认的 maven-compiler-plugin，并根据你配置的 maven.compiler.source 和 maven.compiler.target，来编译你的 Java 代码。
- 这些属性会告诉 maven-compiler-plugin 要使用哪个 JDK 版本来编译源代码，以及生成的字节码应该与哪个版本的 Java 兼容。



### 2、什么时候需要显式配置 < plugin >？

虽然只配置 < properties > 通常足够，但是在以下情况下，你可能需要显式配置 < plugin >：
1. **需要使用插件的特定版本**：如果你需要使用某个特定版本的 maven-compiler-plugin（例如你需要新的功能或者修复某些问题），你可以通过 < plugins > 来指定版本。
   
2. **需要自定义插件配置**：如果你需要对编译器进行额外的自定义配置，例如指定额外的编译器参数、编码格式等，你需要在 `<plugin>` 中显式配置这些内容。例如：

```xml
<!-- 在 properties 中指定 JDK 版本 -->
<properties>
    <maven.compiler.source>17</maven.compiler.source>   <!-- 设置源代码版本 -->
    <maven.compiler.target>17</maven.compiler.target>   <!-- 设置目标代码版本 -->
    <maven.compiler.release>17</maven.compiler.release> <!-- 设置 JDK 版本 -->
    <encoding>UTF-8</encoding>                          <!-- 指定编译时的编码 -->
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.10.1</version>      <!-- 手动指定版本 -->
            <configuration>
                <source>${maven.compiler.source}</source>    <!-- 源代码的 JDK 版本 -->
                <target>${maven.compiler.target}</target>    <!-- 字节码的 JDK 版本 -->
                <release>${maven.compiler.release}</release> <!-- 兼容的 JDK API 版本 -->
                <encoding>${encoding}</encoding>             <!-- 指定编译时的编码 -->
            </configuration>
        </plugin>
    </plugins>
</build>
```



### 3、总结

- **只配置 < properties >** 是简单且常见的做法，因为 Maven 自带了默认的 maven-compiler-plugin，并且会自动应用这些配置。
- **配置 < plugin >** 主要用于自定义编译插件的行为，或指定特定版本的插件，但如果只是指定 JDK 版本，通常不需要手动配置插件。