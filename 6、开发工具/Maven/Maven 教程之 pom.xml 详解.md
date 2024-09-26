# pom.xml 简介

## 1、pom 是什么

**POM 是 Project Object Model 的缩写，即项目对象模型。**

pom.xml 就是 maven 的配置文件，用以描述项目的各种信息。

POM 是项目对象模型（Project Object Model）的简称，它是 Maven 项目中的文件，使用 XML 表示，名称叫做 pom.xml。在 Maven 中，当谈到 Project 的时候，不仅仅是一堆包含代码的文件。一个 Project 往往包含一个配置文件，包括了与开发者有关的，缺陷跟踪系统，组织与许可，项目的 URL，项目依赖，以及其他。它包含了所有与这个项目相关的东西。事实上，在 Maven 世界中，Project 可以什么都没有，甚至没有代码，但是必须包含 pom.xml 文件。



## 2、pom 配置一览

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- The Basics, 基本设置 -->
    <parent>...</parent>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <packaging>...</packaging>
    <dependencies>...</dependencies>
    <dependencyManagement>...</dependencyManagement>
    <modules>...</modules>
    <properties>...</properties>

    <!-- Build Settings, 构建过程的设置 -->
    <build>...</build>
    <reporting>...</reporting>

    <!-- More Project Information, 项目信息设置 -->
    <name>...</name>
    <description>...</description>
    <url>...</url>
    <inceptionYear>...</inceptionYear>
    <licenses>...</licenses>
    <organization>...</organization>
    <developers>...</developers>
    <contributors>...</contributors>

    <!-- Environment Settings, 环境设置 -->
    <issueManagement>...</issueManagement>
    <ciManagement>...</ciManagement>
    <mailingLists>...</mailingLists>
    <scm>...</scm>
    <prerequisites>...</prerequisites>
    <repositories>...</repositories>
    <pluginRepositories>...</pluginRepositories>
    <distributionManagement>...</distributionManagement>
    <profiles>...</profiles>
</project>
```

# 基本配置

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.xyz</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>
</project>
```

- **project** - project 是 pom.xml 中描述符的根。
- **modelVersion** - modelVersion 指定 pom.xml 符合哪个版本的描述符。maven 2 和 3 只能为 4.0.0。

**在 maven 中，根据 `groupId`、`artifactId`、`version` 组合成 `groupId:artifactId:version` 来唯一识别一个 jar 包。**

- **groupId** - 团体、组织的标识符。团体标识的约定是，它以创建这个项目的组织名称的逆向域名(reverse domain name)开头。一般对应着 java 的包结构。
- **artifactId** - 单独项目的唯一标识符。比如我们的 tomcat、commons 等。不要在 artifactId 中包含点号(.)。
- **version** - 一个项目的特定版本。
  - maven 有自己的版本规范，一般是如下定义 major version、minor version、incremental version-qualifier ，比如 1.2.3-beta-01。要说明的是，maven 自己判断版本的算法是 major、minor、incremental 部分用数字比较，qualifier 部分用字符串比较，所以要小心 alpha-2 和 alpha-15 的比较关系，最好用 alpha-02 的格式。
  - maven 在版本管理时候可以使用几个特殊的字符串 SNAPSHOT、LATEST、RELEASE。比如 `1.0-SNAPSHOT`。各个部分的含义和处理逻辑如下说明：
    - **SNAPSHOT** - 这个版本一般用于开发过程中，表示不稳定的版本。
    - **LATEST** - 指某个特定构件的最新发布，这个发布可能是一个发布版，也可能是一个 snapshot 版，具体看哪个时间最后。
    - **RELEASE** ：指最后一个发布版。
- **packaging** - 项目的类型，描述了项目打包后的输出，默认是 jar。常见的输出类型为：pom, jar, maven-plugin, ejb, war, ear, rar, par。



## 1、project

- xmlns="http://maven.apache.org/POM/4.0.0"，定义了 pom(4.0) 命名空间（xmlns）

- xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"，创建了一个 xsi 命名空间

- xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"，定义了 xsd 文件的位置，以便 XML 解析器解析 XML 文件。
  



## 2、modelVersion

声明项目描述符遵循哪一个 POM 模型版本。模型本身的版本很少改变，虽然如此，但它仍然是必不可少的，这是为了当 Maven 引入了新的特性或者其他模型变更的时候，确保稳定性。

modelVersion 是 4.0.0，它是当前唯一同时支持 Maven2 和 Maven3 的 POM 版本。

```xml
<modelVersion>4.0.0</modelVersion>
```



## 3、groupId

公司或者组织的唯一标志，通常使用全限定的包名区分该项目和其他项目。并且构建时生成的路径也是由此生成，如 com.xyz.boot，maven 会将该项目打成的 jar 包放本地路径：/com/xyz/boot

```xml
<groupId>com.xyz.boot</groupId>
```



## 4、artifactId

构件的标识符，它和 groupId 一起唯一标识一个构件。换句话说，你不能有两个不同的项目拥有同样的 artifactId 和 groupId；在某个特定的 groupId下，artifactId 也必须是唯一的。构件是项目产生的或使用的一个东西，Maven 为项目产生的构件包括：JARs，源码，二进制发布和WARs等。

本项目的唯一ID，一个 groupId 下面可能多个项目，就是靠artifactId来区分的

```xml
<artifactId>xyz-boot</artifactId>
```



## 5、version

项目当前版本，格式为：X.Y.Z-V（主版本.次版本.修订号-限定版本号）

- X 表示主版本号，当 API 的兼容性变化时，X 需递增。

- Y 表示次版本号，当你做了向下兼容的功能更新（增加功能）时，不影响 API 的兼容性，Y 需递增。

- Z 表示修订号，当你做了向下兼容的问题修正（Bug 修复）时，不影响 API 的兼容性，Z 需递增。

- V 表示限定版本号，常用的有 SNAPSHOT - 快照版本、RELEASE - 发布版本。
  - `SNAPSHOT`: 快照版本
  - `RELEASE`: 发布版本

```xml
<version>1.0.0-SNAPSHOT</version>
```



## 6、packaging

项目产生的构件类型，例如：`jar、war、pom、maven-plugin、ejb、ear, rar、par，默认为jar`。插件可以创建他们自己的构件类型。

```xml
<packaging>jar</packaging>
```



# 依赖配置

## 1、dependencyManagement

在项目顶层的 POM 文件中，通过`dependencyManagement`元素来管理 jar 包的版本，让子项目中引用一个依赖而不用显示的列出版本号。Maven 会沿着父子层次向上走，直到找到一个拥有`dependencyManagement`元素的项目，然后它就会使用在这个`dependencyManagement`元素中指定的版本号。

`dependencyManagement` 主要是为了统一管理依赖包的版本，确保所有子项目使用的版本一致，类似的还有`plugins`和`pluginManagement`。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson-spring-boot-starter</artifactId>
            <version>${redisson.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```



## 2、dependencies

定义本项目的依赖关系，一个`dependencies`中包含多个`dependency`标签

```xml
<dependencies>
    <dependency>
        <!-- 依赖的组ID: 通常是Maven库或项目的组织或提供者 -->
        <groupId>org.apache.maven</groupId>
        <!-- 依赖的Artifact ID: 标识依赖的具体项目或库 -->
        <artifactId>maven-embedder</artifactId>
        <!-- 依赖的版本号: 指定所需的依赖版本 -->
        <version>2.0</version>
        <!-- 依赖的类型: 指定依赖的文件类型，常见的包括jar、war、pom等 -->
        <type>jar</type>
        <!-- 依赖的范围: 可选值包括：compile、provided、runtime、test、system、import -->
        <scope>test</scope>
        <systemPath></systemPath>
        <optional>true</optional>
        <classifier>sources</classifier>
        <exclusions>
            <exclusion>
                <groupId>org.apache.maven</groupId>
                <artifactId>maven-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

- **groupId**, **artifactId**, **version** - 和基本配置中的 `groupId`、`artifactId`、`version` 意义相同。
- **type** - 对应 `packaging` 的类型，如果不使用 `type` 标签，maven 默认为 jar。
- **scope** - 此元素指的是任务的类路径（编译和运行时，测试等）以及如何限制依赖关系的传递性。有 5 种可用的限定范围：
  - **compile** - 如果没有指定 `scope` 标签，maven 默认为这个范围。编译依赖关系在所有 classpath 中都可用。此外，这些依赖关系被传播到依赖项目。
  - **provided** - 与 compile 类似，但是表示您希望 jdk 或容器在运行时提供它。它只适用于编译和测试 classpath，不可传递。
  - **runtime** - 此范围表示编译不需要依赖关系，而是用于执行。它是在运行时和测试 classpath，但不是编译 classpath。
  - **test** - 此范围表示正常使用应用程序不需要依赖关系，仅适用于测试编译和执行阶段。它不是传递的。
  - **system** - 此范围与 provided 类似，除了您必须提供明确包含它的 jar。该 artifact 始终可用，并且不是在仓库中查找。
- **systemPath** - 仅当依赖范围是系统时才使用。否则，如果设置此元素，构建将失败。该路径必须是绝对路径，因此建议使用 `propertie` 来指定特定的路径，如\$ {java.home} / lib。由于假定先前安装了系统范围依赖关系，maven 将不会检查项目的仓库，而是检查库文件是否存在。如果没有，maven 将会失败，并建议您手动下载安装。
- **optional** - `optional` 让其他项目知道，当您使用此项目时，您不需要这种依赖性才能正常工作。
- **exclusions** - 包含一个或多个排除元素，每个排除元素都包含一个表示要排除的依赖关系的 `groupId` 和 `artifactId`。与可选项不同，可能或可能不会安装和使用，排除主动从依赖关系树中删除自己。



### 2.1、type

**含义**: 定义了依赖项的类型，默认为 jar。这决定了构件的打包方式，如 JAR 包或 WAR 包等。它通常表示依赖的文件的扩展名，但也有例外。一个类型可以被映射成另外一个扩展名或分类器。类型经常和使用的打包方式对应，尽管这也有例外。一些类型的例子：jar，war，ejb-client 和 test-jar。如果设置 extensions 为 true，就可以在 plugin 里定义新的类型。

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>example-project</artifactId>
        <version>1.0.0</version>
        <type>war</type>
    </dependency>
</dependencies>
```





### 2.2、scope

Maven 认为，程序对外部的依赖会随着程序的所处阶段和应用场景而变化，所以 Maven 中的依赖关系有作用域（scope）的限制。scope 包含如下的取值：

| 范围                   | 作用                                                         |
| ---------------------- | ------------------------------------------------------------ |
| compile（编译范围）    | 如果没有指定 scope 标签，maven 默认为这个范围。编译依赖关系在所有 classpath 中都可用。此外，这些依赖关系被传播到依赖项目。 |
| provided（已提供范围） | 与 compile 类似，但是表示您希望 jdk 或容器在运行时提供它。它只适用于编译和测试 classpath，不可传递。 |
| runtime（运行时范围）  | 此范围表示编译不需要依赖关系，而是用于执行。它是在运行时和测试 classpath，但不是编译 classpath。 |
| test（测试范围）       | 此范围表示正常使用应用程序不需要依赖关系，仅适用于测试编译和执行阶段。它不是传递的。 |
| system（系统范围）     | 此范围与 provided 类似，除了您必须提供明确包含它的 jar。该 artifact 始终可用，并且不是在仓库中查找。 |

```xml
<dependencies> 
    <dependency>
        <groupId>com.xyz.trade</groupId>  
        <artifactId>trade-test</artifactId>  
        <version>1.0.0-SNAPSHOT</version>  
        <scope>test</scope> 
    </dependency>
</dependencies> 
```



### 2.3、systemPath

仅当依赖范围是system时才使用。否则，如果设置此元素，构建将失败。该路径必须是绝对路径，因此建议使用 propertie 来指定特定的路径，如\$ {java.home} / lib。由于假定先前安装了系统范围依赖关系，maven 将不会检查项目的仓库，而是检查库文件是否存在。如果没有，maven 将会失败，并建议您手动下载安装

```xml
<dependency>
    <groupId>com.xyz.trade</groupId>  
    <artifactId>trade-test</artifactId>  
    <version>1.0.0-SNAPSHOT</version>  
    <scope>system</scope> 
    <systemPath>{java.home}/lib</systemPath>
</dependency>
```



### 2.4、optional

设置指依赖是否可选，默认为`false`，即子项目默认都继承。设置为`true`，则子项目必需显示的引入，与`dependencyManagement`里定义的依赖类似。

```xml
<dependencies> 
    <dependency>
        <groupId>com.winner.trade</groupId>  
        <artifactId>trade-test</artifactId>  
        <version>1.0.0-SNAPSHOT</version>  
        <optional>false</optional> 
    </dependency>
</dependencies> 
```



### 2.5、classifier

**含义**：classifier 用来区分构件的不同构建版本（如 sources, javadoc 等）。。可以理解为对 artifactId 和 version 的进一步细化。

**典型使用场景**：

1. **源码和 Javadoc JAR 包**：当你希望为某个库添加其源码或 Javadoc JAR 包时，通常会使用 classifier 来区分

   ```xml
   <dependency>
       <groupId>org.example</groupId>
       <artifactId>my-library</artifactId>
       <version>1.0.0</version>
       <classifier>sources</classifier>
   </dependency>
   ```

   这个依赖会下载包含源码的 JAR 包（即带有 -sources.jar 后缀的文件），而不是运行时库。

2. **特定平台的构件**：有些项目会为不同的平台生成不同的构建（例如 Linux、Windows、macOS），这些构件可能具有相同的 groupId、artifactId 和 version，但会通过 classifier 来区分。

   ```xml
   <dependency>
       <groupId>org.example</groupId>
       <artifactId>my-library</artifactId>
       <version>1.0.0</version>
       <classifier>windows</classifier>
   </dependency>
   ```

   这个依赖指向的是为 Windows 构建的 JAR 文件，比如 my-library-1.0.0-windows.jar。

3. **附加资源**：你可以通过 classifier 来添加附加资源，例如测试 JAR 文件。例如：

   ```xml
   <dependency>
       <groupId>org.example</groupId>
       <artifactId>my-library</artifactId>
       <version>1.0.0</version>
       <classifier>tests</classifier>
   </dependency>
   ```

   这个依赖会下载一个仅包含测试资源的 JAR 文件，比如 my-library-1.0.0-tests.jar。

***

如果你想为某个依赖添加源码和 Javadoc 文件，可以这样声明依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.0</version>
    <classifier>sources</classifier> <!-- 下载源码 JAR -->
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.0</version>
    <classifier>javadoc</classifier> <!-- 下载 Javadoc JAR -->
</dependency>
```

这样你就可以获得 Spring Boot 的源码和 Javadoc 文件。



### 2.6、exclusions

屏蔽依赖关系。 比如项目中使用的libA依赖某个库的1.0版，libB依赖某个库的2.0版，现在想统一使用2.0版，就应该屏蔽掉对1.0版的依赖

```xml
<dependencies> 
    <dependency>
        <groupId>com.xyz.trade</groupId>  
        <artifactId>trade-test</artifactId>  
        <version>1.0.0-SNAPSHOT</version>  
        <optional>false</optional> 
        <exclusions>  
            <exclusion>  
                <groupId>org.slf4j</groupId>  
                <artifactId>slf4j-api</artifactId>  
            </exclusion>  
        </exclusions>  
    </dependency>
</dependencies> 
```



***

注意：`dependencyManagement`与`dependencies`区别

- `dependencies`即使在子项目中不写该依赖项，那么子项目仍然会从父项目中继承该依赖项（全部继承）
- `dependencyManagement`里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且 version 和 scope 都读取自父 POM。另外如果子项目中指定了版本号，那么会使用子项目中指定的 jar 版本。



## 3、parent

父项目的坐标。如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。 坐标包括 groupId，artifactId 和 version。maven 支持继承功能。子 POM 可以使用 parent 指定父 POM ，然后继承其配置。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.xyz.parent</groupId>
        <artifactId>my-parent</artifactId>
        <version>2.0</version>
        <relativePath>../my-parent</relativePath>
    </parent>

    <artifactId>my-project</artifactId>
</project>
```

- **relativePath**：注意 `relativePath` 元素。在搜索本地和远程存储库之前，它不是必需的，但可以用作 maven 的指示符，以首先搜索给定该项目父级的路径。



### 3.1、relativePath

父项目的 pom.xml 文件的相对路径。相对路径允许你选择一个不同的路径。默认值是…/pom.xml。Maven 首先在构建当前项目的地方寻找父项目的 pom，其次在文件系统的这个位置（relativePath 位置），然后在本地仓库，最后在远程仓库寻找父项目的 pom。

```xml
<parent>
    <artifactId>xyz-boot-project</artifactId>
    <groupId>com.xyz.boot</groupId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath>../xyz-boot-parent</relativePath>
</parent
```



## 4、modules

模块（有时称作子项目） 被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的相对路径。

modules 标签用于声明当前 Maven 项目包含的模块子项目，每个子项目都是一个独立的 Maven 项目，具有自己的 pom.xml 文件，可以进行独立构建和测试。在父项目的 pom.xml 文件中，使用 标签来列出所有子项目的名称，如下所示：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.xyz.parent</groupId>
    <artifactId>my-parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>my-project</module>
        <module>another-project</module>
        <module>third-project/pom-example.xml</module>
    </modules>
</project>
```



## 5、properties

属性列表。定义的属性可以在 pom.xml 文件中任意处使用。使用方式为 `${propertie}` 。

```xml
<properties>
    <!-- 设置 JDK 源代码版本和目标版本为 17 -->
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>

    <!-- 可选，指定使用的 Java 版本 -->
    <maven.compiler.release>17</maven.compiler.release>

    <!-- 指定项目的源代码文件（包括 .java 文件和资源文件）的字符编码 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- 指定 Maven 生成的报告（如 site 插件生成的项目报告或文档）的输出编码 -->
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
```

# 构建配置

## 1、build

分类：build 可以分为 "project build" 和 "profile build"。

```xml
<!-- "Project Build" contains more elements than just the BaseBuild set -->
<build>...</build>

<profiles>
    <profile>
        <!-- "Profile Build" contains a subset of "Project Build"s elements -->
        <build>...</build>
    </profile>
</profiles>
```

基本构建配置：

```xml
<build>
    <defaultGoal>install</defaultGoal>
    <directory>${basedir}/target</directory>
    <finalName>${artifactId}-${version}</finalName>
    <filters>
        <filter>filters/filter1.properties</filter>
    </filters>
    ...
</build>
```



### 1、defaultGoal

**defaultGoal**：默认执行目标或阶段。如果给出了一个目标，它应该被定义为它在命令行中（如 jar：jar）。如果定义了一个阶段（如 install），也是如此。

当项目没有规定目标（Maven2 叫做阶段`phase`）时的默认值，必须跟命令行上的参数相同例如 jar:jar，或者与某个阶段`phase`相同例如`install`、`compile`等



### 2、directory

构建时的输出路径。默认为：`${basedir}/target` 。



### 3、finalName

这是项目的最终构建名称，产生的构件的文件名，默认值是`${artifactId}-${version}`，不包括文件扩展名。例如：my-project-1.0。



4、filter

定义 `* .properties` 文件，其中包含适用于接受其设置的资源的属性列表（如下所述）。换句话说，过滤器文件中定义的“name = value”对在代码中替换 ${name} 字符串。



### 5、resources

资源的配置。资源文件通常不是代码，不需要编译，而是在项目需要捆绑使用的内容。

```xml
<build>
    <resources>
        <resource>
            <targetPath>META-INF/plexus</targetPath>
            <filtering>false</filtering>
            <directory>${basedir}/src/main/plexus</directory>
            <includes>
                <include>configuration.xml</include>
            </includes>
            <excludes>
                <exclude>**/*.properties</exclude>
            </excludes>
        </resource>
    </resources>
    <testResources>
        <testResource>  
            <targetPath/>  
            <filtering/>  
            <directory/>  
            <includes/>  
            <excludes/>  
        </testResource>  
    </testResourc
</build>
```

- **resources**: 资源元素的列表，每个资源元素描述与此项目关联的文件和何处包含文件。
- **targetPath**: 指定从构建中放置资源集的目录结构。目标路径默认为基本目录。将要包装在 jar 中的资源的通常指定的目标路径是 META-INF。
- **filtering**: 值为 true 或 false。表示是否要为此资源启用过滤。请注意，该过滤器 `* .properties` 文件不必定义为进行过滤 - 资源还可以使用默认情况下在 POM 中定义的属性（例如 ${project.version}），并将其传递到命令行中“-D”标志（例如，“-Dname = value”）或由 properties 元素显式定义。过滤文件覆盖上面。
- **directory**: 值定义了资源的路径。构建的默认目录是`${basedir}/src/main/resources`。
- **includes**: 一组文件匹配模式，指定目录中要包括的文件，使用\*作为通配符。
- **excludes**: 与 `includes` 类似，指定目录中要排除的文件，使用\*作为通配符。注意：如果 `include` 和 `exclude` 发生冲突，maven 会以 `exclude` 作为有效项。
- **testResources**: `testResources` 与 `resources` 功能类似，区别仅在于：`testResources` 指定的资源仅用于 test 阶段，并且其默认资源目录为：`${basedir}/src/test/resources` 。



#### 5.1、targetPath

描述了资源的目标路径。该路径相对`target/classes`目录（例如`${project.build.outputDirectory}`）



#### 5.2、filtering

Maven 中的 filtering 标签用于控制资源文件（如 .properties、.xml 等）中的占位符替换。Maven 支持通过 filtering 将占位符替换为项目中定义的属性（如 pom.xml 中的属性或环境变量）。这通常用于在不同的构建环境（开发、测试、生产）中动态设置配置文件的值。值为 `true` 或 `false`。表示是否要为此资源启用过滤。

**filtering 标签的作用**：

- **功能**：启用或禁用对资源文件的占位符替换。
- **作用范围**：它可以在 pom.xml 的 < resources > 和 < testResources > 中使用，控制编译和测试时的资源文件的过滤行为。

**典型占位符**

- Maven 使用 ${...} 形式的占位符进行变量替换。占位符可以引用 pom.xml 中定义的属性、系统环境变量、Maven 插件中的属性等。
- ${project.version} 会被替换为 pom.xml 中定义的项目版本。
- ${env.HOME} 会被替换为操作系统的环境变量 HOME。

***

**使用场景**

1、在 pom.xml 中定义属性：

你可以在 pom.xml 中通过 < properties > 标签定义一些属性，然后在资源文件中引用这些属性。Maven 会在构建过程中替换资源文件中的占位符。

```xml
<properties>
    <app.environment>development</app.environment>
</properties>
```

2、在资源文件中使用占位符：

在你的资源文件（如 application.properties）中，你可以使用占位符引用这些属性。

```properties
app.environment=${app.environment}
```

在构建过程中，${app.environment} 将会被替换为 development。

***

**基本使用：pom.xml 中的 filtering 配置**

以下是一个典型的 Maven 项目中的 filtering 配置示例。

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

- < directory >: 指定资源文件所在的目录，在这里是 src/main/resources。
- < filtering >: 如果设置为 true，则启用过滤功能，Maven 会替换资源文件中的占位符。

当 filtering 设置为 true 时，Maven 会扫描 src/main/resources 目录下的所有文件，识别出占位符并进行替换。

**示例：pom.xml 配置和资源文件过滤**

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>filtering-demo</artifactId>
    <version>1.0.0</version>

    <!-- 定义属性 -->
    <properties>
        <app.environment>development</app.environment>
        <app.version>${project.version}</app.version>
    </properties>

    <build>
        <!-- 配置资源过滤 -->
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering> <!-- 启用过滤 -->
            </resource>
        </resources>
    </build>
</project>
```

application.properties 示例（位于 src/main/resources）

```properties
app.environment=${app.environment}
app.version=${app.version}
```

当你执行 Maven 构建（如 mvn package）时，application.properties 中的占位符会被替换为 pom.xml 中定义的值：

- ${app.environment} 被替换为 development
- ${app.version} 被替换为 1.0.0（项目版本）

构建后的 application.properties 文件内容可能如下：

```properties
app.environment=development
app.version=1.0.0
```

***

**更高级的过滤——使用过滤文件**

有时，你可能希望将一些属性值放在单独的过滤文件中，而不是在 pom.xml 中定义。Maven 提供了 filtering 文件的支持。

**步骤 1：创建过滤文件**

创建一个过滤文件 config.properties，位于项目根目录的 src/main/filters/：

```properties
app.environment=production
app.version=${project.version}
```

**步骤 2：修改 pom.xml 以使用过滤文件**

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <testResources> <!-- 也可以为测试资源配置过滤 -->
        <testResource>
            <directory>src/test/resources</directory>
            <filtering>true</filtering>
        </testResource>
    </testResources>

    <!-- 指定过滤文件 -->
    <filters>
        <filter>src/main/filters/config.properties</filter>
    </filters>
</build>
```

**其他 filtering 选项**

1、**环境变量和系统属性的引用**：除了在 pom.xml 中定义的属性，你也可以在资源文件中引用环境变量和系统属性:

- ${env.HOME}: 引用系统环境变量 HOME。
- ${user.name}: 引用 JVM 系统属性 user.name。

3、示例，文件 config.properties，位于项目根目录的 src/main/filters/：

```properties
user.home=${env.HOME}
user.name=${user.name}
```

***

**总结**

- filtering 主要用于在构建过程中自动替换资源文件中的占位符。
- 可以通过在 pom.xml 中定义属性，或使用外部的过滤文件，实现占位符的替换。
- 常用于不同环境的配置管理（如开发、测试、生产环境），使得构建过程更加灵活。



#### 5.3、directory

值定义了资源的路径。构建的默认目录`${basedir}/src/main/resources`



#### 5.4、includes

一组文件匹配模式，指定目录中要包括的文件，使用*作为通配符。

```xml
<includes>  
    <include>**/*.properties</include>  
    <include>**/*.xml</include>  
</includes>  
```



#### 5.5、excludes

排除的模式列表 如果与划定的范围存在冲突，以为准

```xml
<excludes>  
    <exclude>jdbc.properties</exclude>  
</excludes>  
```



#### 5.6、testResources

单元测试相关的所有资源路径，配置方法与 resources 类似



### 6、plugins

位置：project > build > plugins

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>2.6</version>
            <extensions>false</extensions>
            <inherited>true</inherited>
            <configuration>
                <classifier>test</classifier>
            </configuration>
            <dependencies>...</dependencies>
            <executions>...</executions>
        </plugin>
    </plugins>
</build>
```

- **groupId**, **artifactId**, **version** ：和基本配置中的 `groupId`、`artifactId`、`version` 意义相同。

- **extensions** ：值为 true 或 false。是否加载此插件的扩展名。默认为 false。

- **inherited** ：值为 true 或 false。这个插件配置是否应该适用于继承自这个插件的 POM。默认值为 true。

- **configuration** - 这是针对个人插件的配置，这里不扩散讲解。

- **dependencies** ：这里的 `dependencies` 是插件本身所需要的依赖。

- **executions** ：需要记住的是，插件可能有多个目标。每个目标可能有一个单独的配置，甚至可能将插件的目标完全绑定到不同的阶段。执行配置插件的目标的执行。
  - **id**: 执行目标的标识。
  - **goals**: 像所有多元化的 POM 元素一样，它包含单个元素的列表。在这种情况下，这个执行块指定的插件目标列表。
  - **phase**: 这是执行目标列表的阶段。这是一个非常强大的选项，允许将任何目标绑定到构建生命周期中的任何阶段，从而改变 maven 的默认行为。
  - **inherited**: 像上面的继承元素一样，设置这个 false 会阻止 maven 将这个执行传递给它的子代。此元素仅对父 POM 有意义。
  - **configuration**: 与上述相同，但将配置限制在此特定目标列表中，而不是插件下的所有目标。



#### 6.1、extensions

该标签用于告诉 Maven 是否启用插件的扩展功能。如果设置为 true，插件可以扩展 Maven 的生命周期、类型或其他核心功能。通常默认设置为 false，除非确实需要。



#### 6.2、inherited

当你有多模块项目时，子模块会继承父 pom.xml 中定义的插件。如果 < inherited > 设置为 false，则子模块不会继承该插件配置。值为 true 或 false。默认值为 true。



#### 6.3、configuration

这是针对个人插件的配置

**功能**：用于配置插件的特定行为。< configuration > 标签内的元素是插件特有的参数。

- < classifier >：配置生成的 JAR 文件的分类。如果 classifier 被设置为 test，意味着打包输出的 JAR 文件名将包含 -test 后缀。例如，如果项目生成的主 JAR 名为 my-app-1.0.jar，则通过这个插件生成的测试 JAR 文件名将是 my-app-1.0-test.jar。classifier 通常用于生成特殊用途的 JAR 文件，比如测试或源代码 JAR 文件。



#### 6.4、dependencies

这里的 dependencies 是插件本身所需要的依赖。

**功能**：可以在插件中定义依赖项。这些依赖是插件在执行时需要使用的。虽然插件本身是一个依赖，但插件可以自己包含所需的其他依赖来支持它的功能。

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>custom-lib</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```



#### 6.5、executions

需要记住的是，插件可能有多个目标。每个目标可能有一个单独的配置，甚至可能将插件的目标完全绑定到不同的阶段。执行配置插件的目标的执行。

**功能**：用于配置插件的执行行为。你可以定义插件在构建生命周期中的哪些阶段执行、执行多少次、是否有特定的 ID 等。

- < id >：执行的唯一标识符，用于区分不同的执行块。
- < phase >：插件在 Maven 构建生命周期中的哪个阶段执行，比如 compile、package、test 等。
- < goals >：要执行的具体目标（插件的功能），比如 jar、test-jar 等。

```xml
<executions>
    <execution>
        <id>create-jar</id> <!-- 执行的 ID -->
        <phase>package</phase> <!-- 在 package 阶段执行 -->
        <goals>
            <goal>jar</goal> <!-- 插件的目标是打包 JAR 文件 -->
        </goals>
    </execution>
</executions>
```

**完整示例**：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>2.6</version>
            <!-- 不扩展 Maven 核心功能 -->
            <extensions>false</extensions>
            <!-- 子项目继承此配置 -->
            <inherited>true</inherited>
            <configuration>
                <!-- 将 JAR 文件标记为测试用途 -->
                <classifier>test</classifier> 
            </configuration>
            <dependencies>
                <!-- 此插件依赖 custom-lib 依赖包 -->
                <dependency>
                    <groupId>org.example</groupId>
                    <artifactId>custom-lib</artifactId>
                    <version>1.0</version>
                </dependency>
            </dependencies>
            <executions>
                <execution>
                    <id>create-test-jar</id>
                    <!-- 在 package 阶段执行 -->
                    <phase>package</phase>
                    <goals>
                        <!-- 执行打包 JAR 文件的目标 -->
                        <goal>jar</goal> 
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**执行过程：**

- 在 package 阶段，maven-jar-plugin 插件会被执行。
- 插件使用依赖的 custom-lib 库。
- 执行 jar 目标，生成一个标记为 test 的 JAR 文件（例如 my-app-1.0-test.jar）。



##### 6.5.1、id

执行目标的唯一标识符



##### 6.5.2、phase

插件在 Maven 构建生命周期中的哪个阶段执行，比如 compile、package、test 等。

Maven 构建生命周期分为多个阶段（phase），每个阶段表示构建过程中的一个步骤。这些阶段按顺序执行，通常会根据项目的状态和目标自动选择从哪个阶段开始以及执行到哪个阶段。

**Maven 构建生命周期的主要阶段：**

- **validate**：验证项目是否正确，确保所有必要的信息和配置文件都可用。
- **compile**：编译项目的源代码。
- **test**：使用适当的单元测试框架（如 JUnit）测试已编译的代码。
- **package**：将编译好的代码打包成可分发的格式，如 JAR、WAR。
- **verify**：运行集成测试，检查包的质量并验证其是否满足质量标准。
- **install**：将生成的包安装到本地 Maven 仓库，供本地其他项目使用。
- **deploy**：将最终的构件复制到远程仓库，供其他开发者或系统使用。

**phase 的作用：**

phase 表示该插件的目标将在 Maven 构建生命周期中的哪个阶段执行。例如：

```xml
<phase>package</phase>
```

表示插件的执行会在 Maven 构建生命周期的 package 阶段进行。如果你在这个阶段指定插件目标，Maven 会在执行 package 时调用插件，确保该阶段完成时生成所需的构件（如 JAR 文件）。



**关键点：**

- phase 是 Maven 构建的一个执行节点，表示一个构建步骤。
- 如果在 package 阶段执行，Maven 会在到达 package 阶段时触发相关的插件目标。
- 如果你运行一个高阶段，如 install，Maven 会从 validate 阶段开始执行，直到 install 阶段结束。



##### 6.5.3、goals

要执行的具体目标（插件的功能），比如 jar、test-jar 等

Maven 插件提供了一个或多个目标（goal），每个目标代表了插件所能执行的一个具体任务。插件的 goal 定义了它实际要做的工作，比如编译代码、运行测试、生成 JAR 文件等。

**goal 的作用：**

- goal 是插件执行的具体任务。例如，对于 maven-jar-plugin 插件，jar 是一个常用的目标，它表示生成一个 JAR 文件。

每个插件有自己的特定目标列表，以下是一些常用插件的目标示例：

- **maven-compiler-plugin**
  - compile：编译主源代码。
  - testCompile：编译测试代码。
- **maven-surefire-plugin**
  - test：运行单元测试。
- **maven-jar-plugin**
  - jar：创建一个 JAR 文件。

示例：

```xml
<goals>
    <goal>jar</goal>
</goals>
```

这表示在执行时，maven-jar-plugin 将运行其 jar 目标来打包 JAR 文件。

**关键点：**

- goal 是插件的具体任务。
- 一个插件可以有多个 goal，每个 goal 完成不同的任务。
- 你可以在同一个插件中定义多个 goal，它们在同一阶段中按顺序执行。

***

**phase 与 goal 的关系：**

Maven 在执行时，首先确定要执行的构建阶段（phase），然后它会调用与该阶段对应的插件目标（goal）。在构建生命周期的每个阶段，Maven 会自动执行所有已绑定到该阶段的目标。



##### 6.5.4、inherited

像上面的继承元素一样，设置这个 false 会阻止 maven 将这个执行传递给它的子代。此元素仅对父 POM 有意义



##### 6.5.5、configuration

与上述相同，但将配置限制在此特定目标列表中，而不是插件下的所有目标。

**示例代码**：这段代码使用了 Maven 的 maven-antrun-plugin 插件来执行一个 Ant 任务（在这里是一个简单的 echo 输出任务）。maven-antrun-plugin 插件允许你在 Maven 构建过程中执行 Ant 任务，这种方式特别适合在 Maven 构建中添加自定义的行为。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>1.1</version>
            <executions>
                <execution>
                    <id>echodir</id>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <phase>verify</phase>
                    <inherited>false</inherited>
                    <configuration>
                        <tasks>
                            <echo>Build Dir: ${project.build.directory}</echo>
                        </tasks>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



#### 6.6、summary

- < groupId > 和 < artifactId > 定义插件的来源。
- < version > 指定插件的版本。
- < extensions > 控制是否扩展 Maven 核心功能。
- < inherited > 控制子项目是否继承插件的配置。
- < configuration > 用于配置插件的行为和参数，比如 classifier 用于区分生成的构件类型。
- < dependencies > 指定插件需要的额外依赖。
- < executions > 配置插件的执行阶段和目标。



### 7、pluginManagement

与 `dependencyManagement` 很相似，在当前 POM 中仅声明插件，而不是实际引入插件。子 POM 中只配置 `groupId` 和 `artifactId` 就可以完成插件的引用，且子 POM 有权重写 pluginManagement 定义。

它的目的在于统一所有子 POM 的插件版本。



### 8、directories

```xml
<build>
    <sourceDirectory>${basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${basedir}/src/test/java</testSourceDirectory>
    <outputDirectory>${basedir}/target/classes</outputDirectory>
    <testOutputDirectory>${basedir}/target/test-classes</testOutputDirectory>
</build>
```

目录元素集合存在于 `build` 元素中，它为整个 POM 设置了各种目录结构。由于它们在配置文件构建中不存在，所以这些不能由配置文件更改。

如果上述目录元素的值设置为绝对路径（扩展属性时），则使用该目录。否则，它是相对于基础构建目录：`${basedir}`。



#### 8.1、sourceDirectory

项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。

```xml
<sourceDirectory>${basedir}/src/main/java</sourceDirectory>
```



#### 8.2、scriptSourceDirectory

项目脚本源码目录，该目录和源码目录不同， <!-- 绝大多数情况下，该目录下的内容会被拷贝到输出目录(因为脚本是被解释的，而不是被编译的)。

```xml
<scriptSourceDirectory>${basedir}/src/main/scripts</scriptSourceDirectory
```



#### 8.3、testSourceDirectory

目单元测试使用的源码目录，当测试项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。

```xml
<testSourceDirectory>${basedir}/src/test/java</testSourceDirectory>
```



#### 8.4、outputDirectory

被编译过的应用程序class文件存放的目录。

```xml
<outputDirectory>${basedir}/target/classes</outputDirectory> 
```



#### 8.5、testOutputDirectory

被编译过的测试class文件存放的目录。

```xml
<testOutputDirectory>${basedir}/target/test-classes</testOutputDirectory>
```



### 9、extensions

扩展是在此构建中使用的 artifacts 的列表。它们将被包含在运行构建的 classpath 中。它们可以启用对构建过程的扩展（例如为 Wagon 传输机制添加一个 ftp 提供程序），并使活动的插件能够对构建生命周期进行更改。简而言之，扩展是在构建期间激活的 artifacts。扩展不需要实际执行任何操作，也不包含 Mojo。因此，扩展对于指定普通插件接口的多个实现中的一个是非常好的。

```xml
<build>
    <extensions>
        <extension>
            <groupId>org.apache.maven.wagon</groupId>
            <artifactId>wagon-ftp</artifactId>
            <version>1.0-alpha-3</version>
        </extension>
    </extensions>
</build>
```



## 2、reporting

描述使用报表插件产生报表的规范，特定的 maven 插件能输出相应的定制和配置报表。当用户执行`mvn site`，这些报表就会运行，在页面导航栏能看到所有报表的链接。

```xml
<reporting>  
    <excludeDefaults />  
    <outputDirectory />  
    <plugins>  
        <plugin>  
            <groupId />  
            <artifactId />  
            <version />  
            <inherited />  
            <configuration>  
                <links>
                    </link>  
                </links>  
            </configuration>  
        <reportSets>  
            <reportSet>  
                <id>sunlink</id>  
                <configuration />
                <inherited />  
                <reports>  
                    <report>javadoc</report>  
                </reports>  
            </reportSet>  
        </reportSets>  
        </plugin>  
    </plugins>  
</reporting>
```

报告包含特定针对 `site` 生成阶段的元素。某些 maven 插件可以生成 `reporting` 元素下配置的报告，例如：生成 javadoc 报告。`reporting` 与 `build` 元素配置插件的能力相似。

- 明显的区别在于：在执行块中插件目标的控制不是细粒度的，报表通过配置 `reportSet` 元素来精细控制。而微妙的区别在于 `reporting` 元素下的 `configuration` 元素可以用作 `build` 下的 `configuration` ，尽管相反的情况并非如此（ `build` 下的 `configuration` 不影响 `reporting` 元素下的 `configuration` ）。
- 另一个区别就是 `plugin` 下的 `outputDirectory` 元素。在报告的情况下，默认输出目录为 `${basedir}/target/site`。

```xml
<reporting>
    <plugins>
        <plugin>
            <reportSets>
                <reportSet>
                    <id>sunlink</id>
                    <reports>
                        <report>javadoc</report>
                    </reports>
                    <inherited>true</inherited>
                    <configuration>
                        <links>
                            <link>http://java.sun.com/j2se/1.5.0/docs/api/</link>
                        </links>
                    </configuration>
                </reportSet>
            </reportSets>
        </plugin>
    </plugins>
</reporting>
```



### 2.1、excludeDefaults

设置为 true，则网站不包括默认的报表。这包括“项目信息”菜单中的报表。



### 2.2、outputDirectory

所有产生的报表存放到哪里。默认值是`${project.build.directory}/site`。



### 2.3、plugin

#### 2.3.1、groupId

报表插件在仓库里的group ID



#### 2.3.2、artifactId

报表插件在仓库里的artifact ID



#### 2.3.3、version

报表插件在仓库里的artifact ID



#### 2.3.4、inherited

任何配置是否被传播到子项目



#### 2.3.5、configuration

报表插件的配置

```xml
<configuration>  
    <links>  
        <link>http://java.sun.com/j2se/1.5.0/docs/api/</link>  
    </links>  
</configuration>  
```



#### 2.3.6、reportSet

组报表的多重规范，每个规范可能有不同的配置。

```xml
<reportSets>  
    <reportSet>  
        <id>sunlink</id>  
        <configuration />  
        <inherited />  
        <reports>  
            <report>javadoc</report>  
        </reports>  
    </reportSet>  
</reportSets>
```



# 项目信息

项目信息相关的这部分标签**都不是必要的**，也就是说完全可以不填写。它的作用仅限于描述项目的详细信息。

下面的示例是项目信息相关标签的清单：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 项目信息 begin -->

    <!--项目名-->
    <name>maven-notes</name>

    <!--项目描述-->
    <description>maven 学习笔记</description>

    <!--项目url-->
    <url>https://github.com/dunwu/maven-notes</url>

    <!--项目开发年份-->
    <inceptionYear>2017</inceptionYear>

    <!--开源协议-->
    <licenses>
        <license>
            <name>Apache License, Version 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>repo</distribution>
            <comments>A business-friendly OSS license</comments>
        </license>
    </licenses>

    <!--组织信息(如公司、开源组织等)-->
    <organization>
        <name>...</name>
        <url>...</url>
    </organization>

    <!--开发者列表-->
    <developers>
        <developer>
            <id>victor</id>
            <name>Zhang Peng</name>
            <email>forbreak at 163.com</email>
            <url>https://github.com/dunwu</url>
            <organization>...</organization>
            <organizationUrl>...</organizationUrl>
            <roles>
                <role>architect</role>
                <role>developer</role>
            </roles>
            <timezone>+8</timezone>
            <properties>...</properties>
        </developer>
    </developers>

    <!--代码贡献者列表-->
    <contributors>
        <contributor>
            <!--标签内容和<developer>相同-->
        </contributor>
    </contributors>

    <!-- 项目信息 end -->
</project>
```

这部分标签都非常简单，基本都能做到顾名思义，且都属于可有可无的标签，所以这里仅简单介绍一下：

- **name** - 项目完整名称

- **description** - 项目描述

- **url** - 一般为项目仓库的 host

- **inceptionYear** - 开发年份

- **licenses** - 开源协议

- **organization** - 项目所属组织信息

- **developers** - 项目开发者列表

- **contributors** - 项目贡献者列表，`<contributor>` 的子标签和 `<developer>` 的完全相同。



## 1、name

项目的名称, Maven产生的文档用

```xml
<name>maven-test </name>
```



## 2、description

项目的详细描述, Maven 产生的文档用。 当这个元素能够用HTML格式描述时。例如，CDATA中的文本会被解析器忽略，就可以包含HTML标签），不鼓励使用纯文本描述。

```xml
<description>A maven project to study maven. </description>
```



## 3、url

项目主页的 URL，Maven 产生的文档用

```xml
<url>http://www.test.com/</url>
```



## 4、inceptionYear

项目创建年份，4位数字。当产生版权信息时需要使用这个值。



## 5、licenses

该元素描述了项目所有License列表。应该只列出该项目的license列表，不要列出依赖项目的license列表。

```xml
<licenses>
    <license>
        <!-- license 用于法律上的名称 -->  
        <name> Apache 2 </name>
        <!-- 官方的 license 正文页面的URL -->  
        <url>http://www.test.com/LICENSE-2.0.txt </url>
        <!-- 项目分发的主要方式： repo，可以从 Maven 库下载 Manual， 用户必须手动下载和安装依赖 -->  
        <distribution> repo</distribution>
        <!-- 关于 license 的补充信息 -->  
        <comments> Abusiness-friendly OSS license </comments>
    </license>
</licenses>
```



## 6、organization

描述项目所属组织的各种属性。Maven产生的文档用

```xml
<!-- 描述项目所属组织的各种属性。Maven产生的文档用 -->  
<organization>  
    <!-- 组织的全名 -->  
    <name> demo </name>  
    <!-- 组织主页的URL -->  
    <url> http://www.test.com/</url>  
</organization>   
```



## 7、developers

项目开发者列表

```xml
<!--项目开发者列表 -->  
<developers>  

    <!--某个项目开发者的信息 -->  
    <developer>  

        <!--SCM里项目开发者的唯一标识符 -->  
        <id> HELLO WORLD </id>  

        <!--项目开发者的全名 -->  
        <name> banseon </name>  

        <!--项目开发者的email -->  
        <email> banseon@126.com</email>  

        <!--项目开发者的主页的URL -->  
        <url />  

        <!--项目开发者在项目中扮演的角色，角色元素描述了各种角色 -->  
        <roles>  
            <role> Project Manager</role>  
            <role>Architect </role>  
        </roles>  

        <!--项目开发者所属组织 -->  
        <organization> demo</organization>  

        <!--项目开发者所属组织的URL -->  
        <organizationUrl>http://hi.test.com/ </organizationUrl>  

        <!--项目开发者属性，如即时消息如何处理等 -->  
        <properties>  
            <dept> No </dept>  
        </properties>  

        <!--项目开发者所在时区， -11到12范围内的整数。 -->  
        <timezone> +5 </timezone>  

    </developer>  

</developers>  
```



## 8、contributors

项目的其他贡献者列表

```xml
<contributors>
    <!--项目的其他贡献者。参见developers/developer元素 -->
    <contributor>  
        <name />
        <email />
        <url />
        <organization />
        <organizationUrl />
        <roles />
        <timezone />
        <properties />
    </contributor>
</contributors>
```

# 环境配置

## 1、issueManagement

这定义了所使用的缺陷跟踪系统（Bugzilla，TestTrack，ClearQuest，Jira，Scarab 等）。虽然没有什么可以阻止插件使用这些信息的东西，但它主要用于生成项目文档。

```xml
<issueManagement>
    <!-- 问题管理系统（例如jira）的名字， -->  
    <system> jira </system>  
    <!--该项目使用的问题管理系统的URL -->  
    <url> http://jira.test.com/</url>  
</issueManagement> 
```

```xml
<issueManagement>
    <system>Bugzilla</system>
    <url>http://127.0.0.1/bugzilla/</url>
</issueManagement>
```



## 2、ciManagement

CI 构建系统配置，主要是指定通知机制以及被通知的邮箱。

```xml
<ciManagement>
    <!-- 持续集成系统的名字，例如 continuum -->
    <system>continuum</system>
    <!-- 该项目使用的持续集成系统的URL（如果持续集成系统有web接口的话） -->
    <url>http://127.0.0.1:8080/continuum</url>
    <!-- 构建完成时，需要通知的开发者/用户的配置项。包括被通知者信息和通知条件（错误，失败，成功，警告） -->
    <notifiers>
        <!-- 配置一种方式，当构建中断时，以该方式通知用户/开发者 -->
        <notifier>
            <!-- 传送通知的途径 -->
            <type>mail</type>
            <!-- 发生错误时是否通知 -->
            <sendOnError>true</sendOnError>
            <!-- 构建失败时是否通知 -->
            <sendOnFailure>true</sendOnFailure>
            <!-- 构建成功时是否通知 -->
            <sendOnSuccess>false</sendOnSuccess>
            <!--发生警告时是否通知 -->
            <sendOnWarning>false</sendOnWarning>
            <!-- 通知发送到哪里 -->
            <address>continuum@127.0.0.1</address>
            <!-- 扩展配置项 -->
            <configuration></configuration>
        </notifier>
    </notifiers>
</ciManagement>
```



## 3、mailingLists

项目相关邮件列表信息

```xml
<mailingLists>
    <!-- 该元素描述了项目相关的所有邮件列表。自动产生的网站引用这些信息 -->  
    <mailingList>
        <!-- 邮件的名称 -->
        <name>User List</name>
        <!-- 发送邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
        <subscribe>user-subscribe@127.0.0.1</subscribe>
        <!-- 订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
        <unsubscribe>user-unsubscribe@127.0.0.1</unsubscribe>
        <!-- 取消订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
        <post>user@127.0.0.1</post>
        <!-- 你可以浏览邮件信息的URL -->
        <archive>http://127.0.0.1/user/</archive>
        <otherArchives>
            <otherArchive>http://base.google.com/base/1/127.0.0.1</otherArchive>
        </otherArchives>
    </mailingList>
</mailingLists>
```



## 4、scm

SCM（软件配置管理，也称为源代码/控制管理或简洁的版本控制）。供 Maven Web 站点和其它插件使用。常见的 scm 有 svn 和 git 。

SCM 又叫 Software Configuration Management，即软件配置管理， 与我们以前提到过的版本控制有关，是 Maven 中用于指定源代码版本控制信息的标签。它可以帮助 Maven 获取源代码并将构建生成的二进制文件提交到版本控制系统中。SCM 标签通常用于指定源代码管理系统的类型、URL、开发者连接等详细信息。示例如下：

```xml
<scm>
    <!-- 指定连接到SCM的URL，可以使用HTTP或者SSH协议 -->
    <connection>scm:git:git://github.com/username/repo.git</connection>
    <!-- 指定开发者连接到SCM的URL，通常使用SSH协议 -->
    <developerConnection>scm:git:ssh://github.com/username/repo.git</developerConnection>
    <!-- 指定SCM的web页面URL，方便开发者查看SCM信息 -->
    <url>http://github.com/username/repo/tree/master</url>
    <!-- 当前Maven构件的版本在SCM中对应的标记 -->
    <tag>1.0.0</tag>
</scm>
```

```xml
<scm>
    <connection>scm:svn:http://127.0.0.1/svn/my-project</connection>
    <developerConnection>scm:svn:https://127.0.0.1/svn/my-project</developerConnection>
    <tag>HEAD</tag>
    <url>http://127.0.0.1/websvn/my-project</url>
</scm>
```

- connection：SCM 的 URL，该 URL 描述了版本库和如何连接到版本库。欲知详情，请看 SCMs 提供的 URL 格式和列表。该连接只读。
- developerConnection：给开发者使用的，类似 connection 元素。即该连接不仅仅只读。
- tag：当前代码的标签，在开发阶段默认为 HEAD。
- url：指向项目的可浏览 SCM 库（例如 ViewVC 或者 Fisheye）的URL。

developerConnection 与 connection 有些许不同，connection 可以指向一个本地文件系统路径，也可以是一个远程代码仓库的 URL；developerConnection 则是开发者使用的版本控制系统的连接URL，例如 connection 指向只读代码仓库，而 developerConnection 则指向可写代码仓库。

通过在 POM 文件中添加 scm 标签，Maven 可以获取源代码并构建项目，同时还可以自动将构建生成的文件提交到版本控制系统中，方便管理代码版本和协同开发



## 5、prerequisites

POM 执行的预设条件。描述了这个项目构建环境中的前提条件。

```xml
<prerequisites>
    <!-- 构建该项目或使用该插件所需要的Maven的最低版本 --> 
    <maven>2.0.6</maven>
</prerequisites>
```



## 6、repositories

`repositories` 是遵循 Maven 存储库目录布局的 artifacts 集合。默认的 Maven 中央存储库位于：https://repo.maven.apache.org/maven2/上。

```xml
<repositories>
    <!-- 包含需要连接到远程仓库的信息 -->
    <repository>
        <!-- 如何处理远程仓库里发布版本的下载 -->
        <releases>
            <!-- 表示该仓库是否为下载某种类型构件（发布版，快照版）开启, 值为true或者false -->
            <!-- 默认值为 true -->
            <enabled>false</enabled>
            <!-- 该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳 -->
            <!-- 这里的选项是：always（一直：默认值），daily（每日），interval:X（这里X是以分钟为单位的时间间隔），never（从不） -->
            <updatePolicy>always</updatePolicy>
            <!-- 当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告，默认值） -->
            <checksumPolicy>warn</checksumPolicy>
        </releases>
        <!-- 如何处理远程仓库里快照版本的下载。有了这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略 -->
        <snapshots>
            <!-- 默认值为 false -->
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
        <!-- 远程仓库唯一标识 -->
        <id>codehausSnapshots</id>
        <!-- 远程仓库名称 -->
        <name>Codehaus Snapshots</name>
        <!-- 远程仓库URL，按protocol://hostname/path形式 -->
        <url>http://snapshots.maven.codehaus.org/maven2</url>
        <!-- 用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留） -->
        <layout>default</layout>
    </repository>
</repositories>
```



### 6.1、releases/snapshots

如何处理远程仓库里`发布版本`和`快照版本`的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。



### 6.2、enabled

true 或者 false 表示该仓库是否为下载某种类型构件（发布版，快照版）开启。

默认值：

- < releases >< enabled >：true
- < snapshots >< enabled >：false



### 6.3、updatePolicy

该元素指定更新发生的频率。Maven 会比较本地 POM 和远程 POM 的时间戳。这里的选项是：

- always（一直）默认值
- daily（默认，每日）
- interval：X（这里X是以分钟为单位的时间间隔）
- never（从不）



### 6.4、checksumPolicy

当 Maven 验证构件校验文件失败时该怎么做

- ignore（忽略）
- fail（失败）
- warn（警告）



#### 6.5、id

远程仓库唯一标识符。可以用来匹配在`settings.xml`文件里配置的远程仓库



### 6.6、name

远程仓库名称



### 6.7、url

远程仓库URL，按`protocol://hostname/path`形式



### 6.8、layout

用于定位和排序构件的仓库布局类型-可以是`default（默认）`或者`legacy（遗留`）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是 default（默认）还是 legacy（遗留）。



## 7、pluginRepositories

与 repositories 差不多。发现插件的远程仓库列表，这些插件用于构建和报表。包含需要连接到远程插件仓库的信息.参见repositories/repository元素

```xml
<!-- 发现插件的远程仓库列表，这些插件用于构建和报表 -->
<pluginRepositories>
    <!-- 包含需要连接到远程插件仓库的信息.参见repositories/repository元素 -->
    <pluginRepository>
        <releases>
            <enabled></enabled>
            <updatePolicy></updatePolicy>
            <checksumPolicy></checksumPolicy>
        </releases>
        <snapshots>
            <enabled></enabled>
            <updatePolicy></updatePolicy>
            <checksumPolicy></checksumPolicy>
        </snapshots>
        <id></id>
        <name></name>
        <url></url>
        <layout></layout>
    </pluginRepository>
</pluginRepositories>
```



## 8、distributionManagement

它管理在整个构建过程中生成的 artifact 和支持文件的分布。项目分发信息，在执行`mvn deploy`后表示要发布的位置。有了这些信息就可以把网站部署到远程服务器或者把构件部署到远程仓库。

```xml
<!-- 项目分发信息，在执行mvn deploy后表示要发布的位置。有了这些信息就可以把网站部署到远程服务器或者把构件部署到远程仓库 -->
<distributionManagement>
    <!-- 部署项目产生的正式版本构件到远程仓库需要的信息 -->
    <repository>
        <!-- 是分配给快照一个唯一的版本号（由时间戳和构建流水号）？还是每次都使用相同的版本号？ -->
        <!-- uniqueVersion 元素用于控制在 Maven 中发布 快照（snapshot） 版本时，生成的构件是否附带一个唯一的时间戳标识 -->
        <!-- 它的默认值是 true，表示在上传版本时，Maven 会在生成的构件文件名中添加一个时间戳和构建编号，以确保每次部署的版本是唯一的 -->
        <uniqueVersion>false</uniqueVersion><!-- 在 releases 中通常不使用 uniqueVersion -->
        <id>banseon-maven2</id>
        <name>banseon maven2</name>
        <url>file://${basedir}/target/deploy</url>
        <layout>default</layout>
    </repository>

    <!-- 部署项目产生的快照版本构件到远程仓库需要的信息 -->
    <snapshotRepository>
        <!-- 是分配给快照一个唯一的版本号（由时间戳和构建流水号）？还是每次都使用相同的版本号？ -->
        <!-- uniqueVersion 元素用于控制在 Maven 中发布 快照（snapshot） 版本时，生成的构件是否附带一个唯一的时间戳标识 -->
        <!-- 它的默认值是 true，表示在上传快照版本时，Maven 会在生成的构件文件名中添加一个时间戳和构建编号，以确保每次部署的版本是唯一的 -->
        <uniqueVersion>true</uniqueVersion><!-- 在 snapshot 中通常会使用 uniqueVersion, 默认true, 所以可不配置 -->
        <id>banseon-maven2</id>
        <name>Banseon-maven2 Snapshot Repository</name>
        <url>scp://svn.baidu.com/banseon:/usr/local/maven-snapshot</url>
        <layout>default</layout>
    </snapshotRepository>

    <!-- 部署项目的网站需要的信息 -->
    <site>
        <!-- 部署位置的唯一标识符，用来匹配站点和settings.xml文件里的配置 -->
        <id>banseon-site</id>
        <!-- 部署位置的名称 -->
        <name>business api website</name>
        <!-- 部署位置的URL，按protocol://hostname/path形式 -->
        <url>scp://svn.baidu.com/banseon:/var/www/localhost/banseon-web</url>
    </site>

    <!-- 项目下载页面的URL。如果没有该元素，用户应该参考主页。使用该元素的原因是：帮助定位那些不在仓库里的构件（由于license限制） -->
    <downloadUrl/>

    <!-- 如果构件有了新的group ID和artifact ID（构件移到了新的位置），这里列出构件的重定位信息。 -->
    <relocation>
        <!-- 构件新的group ID -->
        <groupId></groupId>
        <!-- 构件新的artifact ID -->
        <artifactId></artifactId>
        <!-- 构件新的版本号 -->
        <version></version>
        <!-- 显示给用户的，关于移动的额外信息，例如原因。 -->
        <message></message>
    </relocation>

    <!-- 给出该构件在远程仓库的状态。不得在本地项目中设置该元素，因为这是工具自动更新的 -->
    <!-- 有效的值有：none（默认），
                   converted（仓库管理员从Maven 1 POM转换过来），
                   partner（直接从伙伴Maven 2仓库同步过来），
                   deployed（从Maven 2实例部署），
                   verified（被核实时正确的和最终的）-->
    <status>deployed</status>
</distributionManagement>
```

- **repository/snapshotRepository** - 与 `repositories` 相似

- **site** - 站点信息

- **downloadUrl** - 项目下载页面的URL。如果没有该元素，用户应该参考主页。使用该元素的原因是：帮助定位那些不在仓库里的构件（由于license限制）

- **relocation** - 项目迁移位置

- **status** - 用于指定项目当前的发布状态，以帮助用户理解项目的发布级别。通常用于标识项目是处于“发布”（release）还是“未发布”（in-progress）状态。

***

### 1、简洁配置

```xml
<distributionManagement>
    <!-- 部署项目产生的正式版本构件到远程仓库需要的信息 -->
    <repository>
        <id>banseon-maven2</id>
        <url>file://${basedir}/target/deploy</url>
    </repository>
    <!-- 部署项目产生的快照版本构件到远程仓库需要的信息 -->
    <snapshotRepository>
        <id>banseon-maven2</id>
        <url>scp://svn.baidu.com/banseon:/usr/local/maven-snapshot</url>
    </snapshotRepository>
</distributionManagement>
```



### 2、高阶玩法

1、项目中的 ${basedir}/pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.xyz</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>

    <distributionManagement>
        <!-- 部署项目产生的正式版本构件到远程仓库需要的信息 -->
        <repository>
            <id>xyz-maven2</id>
            <url>file://${basedir}/target/deploy</url>
        </repository>
        <!-- 部署项目产生的快照版本构件到远程仓库需要的信息 -->
        <snapshotRepository>
            <id>xyz-maven2</id>
            <url>scp://svn.baidu.com/banseon:/usr/local/maven-snapshot</url>
        </snapshotRepository>
    </distributionManagement>
</project>
```

2、配置自定义 setting.xml。${basedir}/maven/setting.xml

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">

    <!-- 配置服务端的一些设置。一些设置如安全证书不应该和pom.xml一起分发。这种类型的信息应该存在于构建服务器上的settings.xml文件中 -->
    <servers>
        <!--服务器元素包含配置服务器时需要的信息 -->
        <server>
            <id>xyz-maven2</id> <!-- distributionManagement中repository元素的id相匹配 -->
            <username>my_username</username> <!-- 鉴权用户 -->
            <password>my_password</password> <!-- 鉴权密码 -->
        </server>
    </servers>

    <profiles>
        <profile>
            <id>profile_id</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>

            <repositories>
                <repository>
                    <id>nexus-aliyun</id>
                    <name>nexus-aliyun</name>
                    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>

            <pluginRepositories>
                <pluginRepository>
                    <id>public</id>
                    <name>aliyun nexus</name>
                    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
</settings>
```

注意：在这个命令中，Maven 将**仅加载** /path/to/custom-settings.xml，并忽略默认的 ~/.m2/settings.xml 和 Maven 全局配置文件 conf/settings.xml。

3、执行命令打包到私服仓库：

```bash
mvn deploy -s maven/setting.xml -P profile_id
```



## 9、profiles

**作用**：根据环境参数来调整构建配置的列表。< profile > 中可以定义 < project > 下所有的标签，如果与 < project > 定义相同，相当于覆盖/重写。

```xml
<profiles>
    <profile>
        <!-- profile的唯一标识 -->
        <id>test</id>
        <!-- 自动触发profile的条件逻辑 -->
        <activation />
        <!-- 扩展属性列表 -->
        <properties />
        <!-- 远程仓库列表 -->
        <repositories />
        <!-- 插件仓库列表 -->
        <pluginRepositories />

        <!-- The Basics, 基本设置 -->
        <dependencies>...</dependencies>
        <dependencyManagement>...</dependencyManagement>
        <modules>...</modules>

        <!-- Build Settings, 构建过程的设置 -->
        <build/>
        <reporting/>

        <!-- Environment Settings, 环境设置 -->
        <distributionManagement/>

        <!-- Other Settings, 其他设置 -->
        <!-- 用于定义项目的核心元数据，但很少在 profiles 中使用 -->
        <project/>
        <!-- 用于定义报告生成的插件配置，但在 Maven3 中已不再常见 -->
        <reports/>
    </profile>
</profiles>
```



### 1、id

profile 的唯一标识，激活指定的profile，也是通过 id 来找到对应的 profile 的。



### 2、activation

**作用**：自动触发 `profile` 的条件逻辑。

profile 的作用在于它能够在某些特定的环境中自动使用某些特定的值；这些环境通过 activation 元素指定。

activation 元素并不是激活 profile 的唯一方式。settings.xml 文件中的 activeProfile 元素可以包含 profile 的 id。profile 也可以通过在命令行，使用 -P 标记和逗号分隔的列表来显式的激活（如，-P test）。

```xml
<profiles>
    <profile>
        <id>test</id>
        <activation>
            <!-- profile默认是否激活的标志 -->
            <activeByDefault>false</activeByDefault>
            <!-- 当匹配的jdk被检测到，profile被激活 -->
            <jdk>1.5</jdk>
            <!-- 当匹配的操作系统属性被检测到，profile被激活 -->
            <os>
                <name>Windows XP</name>
                <family>Windows</family>
                <arch>x86</arch>
                <version>5.1.2600</version>
            </os>
            <!-- 检测到某一个属性其拥有对应的名称和值，profile被激活 -->
            <property>
                <name>sparrow-type</name>
                <value>African</value>
            </property>
            <!-- 如果指定的文件存在或不存在，则激活profile -->
            <file>
                <exists>${basedir}/file2.properties</exists>
                <missing>${basedir}/file1.properties</missing>
            </file>
        </activation>
    </profile>
</profiles>
```



### 3、properties

**作用**：对应`profile`的扩展属性列表。与 project>properties 同理。

maven 属性和 ant 中的属性一样，可以用来存放一些值。这些值可以在 `pom.xml` 中的任何地方使用标记 ${X} 来使用，这里 X 是指属性的名称。属性有五种不同的形式，并且都能在 pom.xml 文件中访问。

```xml
<!--
  1. env.X: 在一个变量前加上"env."的前缀，会返回一个shell环境变量。例如,"env.PATH"指代了$path环境变量（在Windows上是%PATH%）。
  2. project.x：指代了POM中对应的元素值。例如: <project><version>1.0</version></project>通过${project.version}获得version的值。
  3. settings.x: 指代了settings.xml中对应元素的值。例如：<settings><offline>false</offline></settings>通过 ${settings.offline}获得offline的值。
  4. Java System Properties: 所有可通过java.lang.System.getProperties()访问的属性都能在POM中使用该形式访问，例如 ${java.home}。
  5. x: 在<properties/>元素中，或者外部文件中设置，以${someVar}的形式使用。
 -->
<properties>
    <user.install>${user.home}/our-project</user.install>
</properties>
```

> 注：如果该 profile 被激活，则可以在`pom.xml`中使用 ${user.install}。



### 4、repositories

**作用**：远程仓库列表，它是 maven 用来填充构建系统本地仓库所使用的一组远程仓库。与 project>repositories 同理。

```xml
<repositories>
    <!-- 包含需要连接到远程仓库的信息 -->
    <repository>
        <!-- 远程仓库唯一标识 -->
        <id>codehausSnapshots</id>
        <!-- 远程仓库名称 -->
        <name>Codehaus Snapshots</name>
        <!-- 如何处理远程仓库里发布版本的下载 -->
        <releases>
            <!-- true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
            <enabled>false</enabled>
            <!-- 该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
            <updatePolicy>always</updatePolicy>
            <!-- 当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告）。 -->
            <checksumPolicy>warn</checksumPolicy>
        </releases>
        <!-- 如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->
        <snapshots>
            <enabled />
            <updatePolicy />
            <checksumPolicy />
        </snapshots>
        <!-- 远程仓库URL，按protocol://hostname/path形式 -->
        <url>http://snapshots.maven.codehaus.org/maven2</url>
        <!-- 用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
        <layout>default</layout>
    </repository>
</repositories>
```

- id：远程仓库唯一标识
- name：远程仓库唯一标识
- layout：用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）
- releases、snapshots：这是对于工件的类型的限制。
- enabled：表示这个仓库是否允许这种类型的工件
- updatePolicy：表示多久尝试更新一次。可选值有always、daily、interval:minutes（表示每多久更新一次）和never。
- checksumPolicy：当Maven在部署项目到仓库的时候会连同校验文件一起提交，checksumPolicy表示当这个校验文件缺失或不正确的时候该如何处理，可选项有ignore、fail和warn



### 5、pluginRepositories

**作用**：发现插件的远程仓库列表。与 project>pluginRepositories 同理。

和 `repository` 类似，只是 `repository` 是管理 jar 包依赖的仓库，`pluginRepositories` 则是管理插件的仓库。
maven 插件是一种特殊类型的构件。由于这个原因，插件仓库独立于其它仓库。`pluginRepositories` 元素的结构和 `repositories` 元素的结构类似。每个 `pluginRepository` 元素指定一个 Maven 可以用来寻找新插件的远程地址。

```xml
<pluginRepositories>
    <!-- 包含需要连接到远程插件仓库的信息.参见profiles/profile/repositories/repository元素的说明 -->
    <pluginRepository>
        <releases>
            <enabled />
            <updatePolicy />
            <checksumPolicy />
        </releases>
        <snapshots>
            <enabled />
            <updatePolicy />
            <checksumPolicy />
        </snapshots>
        <id />
        <name />
        <url />
        <layout />
    </pluginRepository>
</pluginRepositories>
```



### 6、dependencies

**作用**：定义本项目的依赖关系。主要用来新增或重写 project>dependencies>dependencie。



### 7、dependencyManagement

**作用**：主要是为了统一管理依赖包的版本，确保所有子项目使用的版本一致。主要用来新增或重写 project>dependencyManagement。



### 8、modules

**作用**：模块（有时称作子项目） 被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的相对路径。主要用来重写 project>modules。



### 9、build

**作用**：项目分发信息，在执行`mvn deploy`后表示要发布的位置。构件部署到远程仓库。主要用来新增或重写 project>build。



### 10、reporting

**作用**：主要用来新增或重写 project>reporting。



### 11、distributionManagement

**作用**：主要用来重写 project>distributionManagement。



### 12、project/reports

- < project > **标签**：用于定义项目的核心元数据，但很少在 profiles 中使用。
- < reports > **标签**：用于定义报告生成的插件配置，但在 Maven 3 中已不再常见



# 内置属性

Maven 中有一些常用的内置属性，它们可以帮助你访问项目相关的信息。以下是一些常见的内置属性：



## 1、项目相关属性

   - **`${project.basedir}`**: 指向项目的根目录（即包含 `pom.xml` 文件的目录）。
   - **`${project.groupId}`**: 项目的 `groupId`，用于标识项目所在的组织或公司。
   - **`${project.artifactId}`**: 项目的 `artifactId`，是项目的唯一标识符。
   - **`${project.version}`**: 当前项目的版本号。
   - **`${project.packaging}`**: 项目的打包类型，通常是 `jar` 或 `war`。
   - **`${project.build.directory}`**: 指向项目构建过程中生成文件的目录，通常是 `target` 目录。
   - **`${project.build.outputDirectory}`**: 项目编译后的类文件输出目录，通常是 `target/classes`。



## 2、环境相关属性

   - **`${basedir}`**: 项目执行的基准目录，通常等同于 `${project.basedir}`。
   - **`${settings.localRepository}`**: Maven 本地仓库的路径。
   - **`${env.[var]}`**: 用于访问系统环境变量，例如 `${env.PATH}` 访问系统的 `PATH` 变量。



## 3、依赖相关属性

   - **`${project.dependencies}`**: 列出当前项目中的所有依赖项。
   - **`${project.dependencyManagement}`**: 项目中的依赖管理部分，用于定义依赖的版本、范围等。
   - **`${project.parent}`**: 访问项目的父 `POM`，用于继承配置。



## 4、构建和插件相关属性

   - **`${project.build.finalName}`**: 项目构建后的最终文件名，通常是 `artifactId-version`。
   - **`${project.build.sourceDirectory}`**: 项目的源代码目录，通常是 `src/main/java`。
   - **`${project.build.testSourceDirectory}`**: 项目的测试源代码目录，通常是 `src/test/java`。
   - **`${project.build.plugins}`**: 项目中配置的所有插件。



## 5、文件路径相关属性

   - **`${project.build.resources}`**: 项目中的资源文件路径，通常是 `src/main/resources`。
   - **`${project.build.testResources}`**: 项目中的测试资源文件路径，通常是 `src/test/resources`。



## 6、其他常用属性

   - **`${java.home}`**: 当前 JVM 的安装目录。
   - **`${maven.repo.local}`**: Maven 本地仓库的路径。
   - **`${os.name}`**: 当前操作系统的名称。
   - **`${user.home}`**: 当前用户的主目录。

这些内置属性广泛应用于 `pom.xml` 配置中，用于获取各种项目和环境的相关信息，帮助配置依赖、插件和构建过程等。

# 完整配置

如下是完整版本的 pom.xml 文件配置。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- 父项目的坐标。如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。
         坐标包括group ID，artifact ID和 version。 -->
    <parent>
        <!-- 被继承的父项目的构件标识符 -->
        <artifactId>com.xyz</artifactId>

        <!-- 被继承的父项目的全球唯一标识符 -->
        <groupId>parent</groupId>

        <!-- 被继承的父项目的版本 -->
        <version>1.0.0</version>

        <!-- 父项目的pom.xml文件的相对路径。相对路径允许你选择一个不同的路径。默认值是../pom.xml。
             Maven首先在构建当前项目的地方寻找父项目的pom，其次在文件系统的这个位置（relativePath位置），
             然后在本地仓库，最后在远程仓库寻找父项目的pom。 -->
        <relativePath>xxx</relativePath>
    </parent>

    <!-- 声明项目描述符遵循哪一个POM模型版本。模型本身的版本很少改变，虽然如此，但它仍然是必不可少的，
         这是为了当Maven引入了新的特性或者其他模型变更的时候，确保稳定性。 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- 项目的全球唯一标识符，通常使用全限定的包名区分该项目和其他项目。并且构建时生成的路径也是由此生成，
         如com.myCompany.app生成的相对路径为：/com/myCompany/app -->
    <groupId>com.xyz</groupId>

    <!-- 构件的标识符，它和group ID一起唯一标识一个构件。换句话说，你不能有两个不同的项目拥有同样的artifact ID
         和groupID；在某个特定的group ID下，artifact ID也必须是唯一的。构件是项目产生的或使用的一个东西，Maven
         为项目产生的构件包括：JARs，源码，二进制发布和WARs等。 -->
    <artifactId>example-maven</artifactId>

    <!-- 项目产生的构件类型，例如jar、war、ear、pom。插件可以创建他们自己的构件类型，所以前面列的不是全部构件类型 -->
    <packaging>jar</packaging>

    <!-- 项目当前版本，格式为:主版本.次版本.增量版本-限定版本号 -->
    <version>1.0-SNAPSHOT</version>

    <!-- 项目的名称, Maven产生的文档用 -->
    <name>xxx-maven</name>

    <!-- 项目主页的URL, Maven产生的文档用 -->
    <url>https://maven.apache.org</url>

    <!-- 项目的详细描述, Maven 产生的文档用。 当这个元素能够用HTML格式描述时（例如，CDATA中的文本会被解析器忽略，
         就可以包含HTML标签）， 不鼓励使用纯文本描述。如果你需要修改产生的web站点的索引页面，你应该修改你自己的
         索引页文件，而不是调整这里的文档。 -->
    <description>A maven project to study maven.</description>

    <!-- 以值替代名称，Properties可以在整个POM中使用，也可以作为触发条件（见settings.xml配置文件里
         activation元素的说明）。格式是<name>value</name>。 -->
    <properties>
        <name>value</name>
        <version1>1.0</version1>
    </properties>

    <!-- 描述了这个项目构建环境中的前提条件。 -->
    <prerequisites>
        <!-- 构建该项目或使用该插件所需要的Maven的最低版本 -->
        <maven>2.0.3</maven>
    </prerequisites>

    <!-- 项目的问题管理系统(Bugzilla, Jira, Scarab,或任何你喜欢的问题管理系统)的名称和URL，本例为 jira -->
    <issueManagement>
        <!-- 问题管理系统（例如jira）的名字， -->
        <system>jira</system>

        <!-- 该项目使用的问题管理系统的URL -->
        <url>https://jira.baidu.com/banseon</url>
    </issueManagement>

    <!-- 项目持续集成信息 -->
    <ciManagement>
        <!-- 持续集成系统的名字，例如continuum -->
        <system></system>

        <!-- 该项目使用的持续集成系统的URL（如果持续集成系统有web接口的话）。 -->
        <url></url>

        <!-- 构建完成时，需要通知的开发者/用户的配置项。包括被通知者信息和通知条件（错误，失败，成功，警告） -->
        <notifiers>
            <!-- 配置一种方式，当构建中断时，以该方式通知用户/开发者 -->
            <notifier>
                <!-- 传送通知的途径 -->
                <type></type>

                <!-- 发生错误时是否通知 -->
                <sendOnError></sendOnError>

                <!-- 构建失败时是否通知 -->
                <sendOnFailure></sendOnFailure>

                <!-- 构建成功时是否通知 -->
                <sendOnSuccess></sendOnSuccess>

                <!-- 发生警告时是否通知 -->
                <sendOnWarning></sendOnWarning>

                <!-- 不赞成使用。通知发送到哪里 -->
                <address></address>

                <!-- 扩展配置项 -->
                <configuration></configuration>
            </notifier>
        </notifiers>
    </ciManagement>

    <!-- 项目创建年份，4位数字。当产生版权信息时需要使用这个值。 -->
    <inceptionYear>1994</inceptionYear>

    <!-- 项目相关邮件列表信息 -->
    <mailingLists>
        <!-- 该元素描述了项目相关的所有邮件列表。自动产生的网站引用这些信息。 -->
        <mailingList>
            <!-- 邮件的名称 -->
            <name>Demo</name>

            <!-- 发送邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
            <post>banseon@126.com</post>

            <!-- 订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
            <subscribe>banseon@126.com</subscribe>

            <!-- 取消订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
            <unsubscribe>banseon@126.com</unsubscribe>

            <!-- 你可以浏览邮件信息的URL -->
            <archive>http:/hi.baidu.com/banseon/demo/dev/</archive>
        </mailingList>
    </mailingLists>

    <!-- 项目开发者列表 -->
    <developers>
        <!-- 某个项目开发者的信息 -->
        <developer>
            <!-- SCM里项目开发者的唯一标识符 -->
            <id>HELLO WORLD</id>

            <!-- 项目开发者的全名 -->
            <name>banseon</name>

            <!-- 项目开发者的email -->
            <email>banseon@126.com</email>

            <!-- 项目开发者的主页的URL -->
            <url>https://blog.csdn.net/banseon</url>

            <!-- 项目开发者在项目中扮演的角色，角色元素描述了各种角色 -->
            <roles>
                <role>Project Manager</role>
                <role>Architect</role>
                <role>Development</role>
            </roles>

            <!-- 项目开发者所属组织 -->
            <organization>xyz</organization>

            <!-- 项目开发者所属组织的URL -->
            <organizationUrl>https://hi.baidu.com/xxx</organizationUrl>

            <!-- 项目开发者属性，如即时消息如何处理等 -->
            <properties>
                <dept>No</dept>
            </properties>

            <!-- 项目开发者所在时区， -11到12 范围内的整数。 -->
            <timezone>+8</timezone>
        </developer>
    </developers>

    <!-- 项目的其他贡献者列表 -->
    <contributors>
        <!-- 项目的其他贡献者。参见developers/developer元素 -->
        <contributor>
            <!-- 项目贡献者的全名 -->
            <name></name>

            <!-- 项目贡献者的email -->
            <email></email>

            <!-- 项目贡献者的主页的URL -->
            <url></url>

            <!-- 项目贡献者所属组织 -->
            <organization></organization>

            <!-- 项目贡献者所属组织的URL -->
            <organizationUrl></organizationUrl>

            <!-- 项目贡献者在项目中扮演的角色，角色元素描述了各种角色 -->
            <roles>
                <role>Project Manager</role>
                <role>Architect</role>
            </roles>

            <!-- 项目贡献者所在时区， -11到12范围内的整数。 -->
            <timezone></timezone>

            <!-- 项目贡献者属性，如即时消息如何处理等 -->
            <properties>
                <dept>No</dept>
            </properties>
        </contributor>
    </contributors>

    <!-- 该元素描述了项目所有License列表。 应该只列出该项目的license列表，不要列出依赖项目的 license列表。
         如果列出多个license，用户可以选择它们中的一个而不是接受所有license。 -->
    <licenses>
        <!-- 描述了项目的license，用于生成项目的web站点的license页面，其他一些报表和validation也会用到该元素。 -->
        <license>
            <!-- license用于法律上的名称 -->
            <name>Apache 2</name>

            <!-- 官方的license正文页面的URL -->
            <url>https://www.baidu.com/banseon/LICENSE-2.0.txt</url>

            <!-- 项目分发的主要方式：
                    repo，可以从Maven库下载
                    manual， 用户必须手动下载和安装依赖 -->
            <distribution>repo</distribution>

            <!-- 关于license的补充信息 -->
            <comments>A business-friendly OSS license</comments>
        </license>
    </licenses>

    <!-- SCM(Source Control Management)标签允许你配置你的代码库，供Maven web站点和其它插件使用。 -->
    <scm>
        <!-- SCM的URL,该URL描述了版本库和如何连接到版本库。欲知详情，请看SCMs提供的URL格式和列表。该连接只读。 -->
        <connection>
            scm:svn:https://svn.baidu.com/banseon/maven/banseon/banseon-maven2-trunk(dao-trunk)
        </connection>

        <!-- 给开发者使用的，类似connection元素。即该连接不仅仅只读 -->
        <developerConnection>
            scm:svn:https://svn.baidu.com/banseon/maven/banseon/dao-trunk
        </developerConnection>

        <!-- 当前代码的标签，在开发阶段默认为HEAD -->
        <tag></tag>

        <!-- 指向项目的可浏览SCM库（例如ViewVC或者Fisheye）的URL。 -->
        <url>https://svn.baidu.com/banseon</url>
    </scm>

    <!-- 描述项目所属组织的各种属性。Maven产生的文档用 -->
    <organization>
        <!-- 组织的全名 -->
        <name>xyz</name>

        <!-- 组织主页的URL -->
        <url>https://www.baidu.com/banseon</url>
    </organization>

    <!-- 构建项目需要的信息 -->
    <build>
        <!-- 该元素设置了项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对
             于pom.xml的相对路径。 -->
        <sourceDirectory>${basedir}/src/main/java</sourceDirectory>

        <!-- 该元素设置了项目脚本源码目录，该目录和源码目录不同：绝大多数情况下，该目录下的内容会
             被拷贝到输出目录(因为脚本是被解释的，而不是被编译的)。 -->
        <scriptSourceDirectory>${basedir}/src/main/scripts</scriptSourceDirectory>

        <!-- 该元素设置了项目单元测试使用的源码目录，当测试项目的时候，构建系统会编译目录里的源码。
             该路径是相对于pom.xml的相对路径。 -->
        <testSourceDirectory>${basedir}/src/test/java</testSourceDirectory>

        <!-- 被编译过的应用程序class文件存放的目录。 -->
        <outputDirectory>${basedir}/target/classes</outputDirectory>

        <!-- 被编译过的测试class文件存放的目录。 -->
        <testOutputDirectory>${basedir}/target/test-classes</testOutputDirectory>

        <!-- 使用来自该项目的一系列构建扩展 -->
        <extensions>
            <!-- 描述使用到的构建扩展。 -->
            <extension>
                <!-- 构建扩展的groupId -->
                <groupId>org.apache.maven.wagon</groupId>

                <!-- 构建扩展的artifactId -->
                <artifactId>wagon-ftp</artifactId>

                <!-- 构建扩展的版本 -->
                <version>1.0-alpha-3</version>
            </extension>
        </extensions>

        <!-- 当项目没有规定目标（Maven2 叫做阶段）时的默认值 -->
        <defaultGoal>install</defaultGoal>

        <!-- 这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在
             最终的打包文件里。 -->
        <resources>
            <!-- 这个元素描述了项目相关或测试相关的所有资源路径 -->
            <resource>
                <!-- 描述了资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。
                     举个例子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为
                    org/apache/maven/messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。 -->
                <targetPath>META-INF/plexus</targetPath>

                <!-- 是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素
                     里列出。 -->
                <filtering>false</filtering>

                <!-- 描述存放资源的目录，该路径相对POM路径, 也可以绝对路径${basedir}/src/main/java -->
                <directory>src/main/java</directory>

                <!-- 包含的模式列表，例如**/*.xml. -->
                <includes>
                    <include>**/*.xml</include>
                </includes>

                <!-- 排除的模式列表，例如**/*.xml -->
                <excludes>
                    <exclude>**/*.properties</exclude>
                </excludes>
            </resource>
        </resources>

        <!-- 这个元素描述了单元测试相关的所有资源路径，例如和单元测试相关的属性文件。 -->
        <testResources>
            <!-- 这个元素描述了测试相关的所有资源路径，参见build/resources/resource元素的说明 -->
            <testResource>
                <!-- 描述了测试相关的资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。
                     举个例子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为
                    org/apache/maven/messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。 -->
                <targetPath>META-INF/plexus</targetPath>

                <!-- 是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素
                     里列出。 -->
                <filtering>false</filtering>

                <!-- 描述存放测试相关的资源的目录，该路径相对POM路径 -->
                <directory>src/test/java</directory>

                <!-- 包含的模式列表，例如**/*.xml. -->
                <includes>
                    <include>**/*.xml</include>
                </includes>

                <!-- 排除的模式列表，例如**/*.xml -->
                <excludes>
                    <exclude>**/*.properties</exclude>
                </excludes>
            </testResource>
        </testResources>

        <!-- 构建产生的所有文件存放的目录, 默认为：${basedir}/target  -->
        <directory>${basedir}/target</directory>

        <!-- 产生的构件的文件名，默认值是${artifactId}-${version} 不包括文件扩展名 -->
        <finalName>${artifactId}-${version}</finalName>

        <!-- 当filtering开关打开时，使用到的过滤器属性文件列表 -->
        <filters>
            <filter>filters/filter1.properties</filter>
        </filters>

        <!-- 子项目可以引用的默认插件信息。该插件配置项直到被引用时才会被解析或绑定到生命周期。给定插件的任何本
             地配置都会覆盖这里的配置 -->
        <pluginManagement>
            <!-- 使用的插件列表 。 -->
            <plugins>
                <!-- plugin元素包含描述插件所需要的信息。 -->
                <plugin>
                    <!-- 插件在仓库里的group ID -->
                    <groupId>com.xyz</groupId>

                    <!-- 插件在仓库里的artifact ID -->
                    <artifactId>plugin</artifactId>

                    <!-- 被使用的插件的版本（或版本范围） -->
                    <version>1.0.0</version>

                    <!-- 是否从该插件下载Maven扩展（例如打包和类型处理器），由于性能原因，只有在真需要下载时，该
                         元素才被设置成enabled。 -->
                    <extensions>true/false</extensions>

                    <!-- 在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。 -->
                    <executions>
                        <!-- execution元素包含了插件执行需要的信息 -->
                        <execution>
                            <!-- 执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标 -->
                            <id>echodir</id>

                            <!-- 绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段 -->
                            <phase>verify</phase>

                            <!-- 配置的执行目标 -->
                            <goals>
                                <goal>run</goal>
                            </goals>

                            <!-- 配置是否被传播到子POM -->
                            <inherited>true/false</inherited>

                            <!-- 作为DOM对象的配置 -->
                            <configuration>
                                <tasks>
                                    <echo>Build Dir: ${project.build.directory}</echo>
                                </tasks>
                            </configuration>
                        </execution>
                    </executions>

                    <!-- 项目引入插件所需要的额外依赖 -->
                    <dependencies>
                        <!-- 参见dependencies/dependency元素 -->
                        <dependency>
                            <groupId>com.xyz</groupId>
                            <artifactId>plugin</artifactId>
                            <version>1.0.0</version>
                        </dependency>
                    </dependencies>

                    <!-- 任何配置是否被传播到子项目 -->
                    <inherited>true/false</inherited>

                    <!-- 作为DOM对象的配置 -->
                    <configuration>
                        <classifier>test</classifier>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>

        <!-- 该项目使用的插件列表 。 -->
        <plugins>
            <!-- plugin元素包含描述插件所需要的信息。 -->
            <plugin>
                <!-- 插件在仓库里的group ID -->
                <groupId>com.xyz</groupId>

                <!-- 插件在仓库里的artifact ID -->
                <artifactId>plugin</artifactId>

                <!-- 被使用的插件的版本（或版本范围） -->
                <version>1.0.0</version>

                <!-- 是否从该插件下载Maven扩展（例如打包和类型处理器），由于性能原因，只有在真需要下载时，该
                     元素才被设置成enabled。 -->
                <extensions>true/false</extensions>

                <!-- 在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。 -->
                <executions>
                    <!-- execution元素包含了插件执行需要的信息 -->
                    <execution>
                        <!-- 执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标 -->
                        <id>id</id>

                        <!-- 绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段 -->
                        <phase>compile</phase>

                        <!-- 配置的执行目标 -->
                        <goals>
                            <goal>run</goal>
                        </goals>

                        <!-- 配置是否被传播到子POM -->
                        <inherited>true/false</inherited>

                        <!-- 作为DOM对象的配置 -->
                        <configuration>
                            <classifier>test</classifier>
                        </configuration>
                    </execution>
                </executions>

                <!-- 项目引入插件所需要的额外依赖 -->
                <dependencies>
                    <!-- 参见dependencies/dependency元素 -->
                    <dependency>
                        <groupId>com.xyz</groupId>
                        <artifactId>xxx-maven</artifactId>
                        <version>1.0.0</version>
                    </dependency>
                </dependencies>

                <!-- 任何配置是否被传播到子项目 -->
                <inherited>true/false</inherited>

                <!-- 作为DOM对象的配置 -->
                <configuration>
                    <classifier>test</classifier>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <!-- 在列的项目构建profile，如果被激活，会修改构建处理 -->
    <profiles>
        <!-- 根据环境参数或命令行参数激活某个构建处理 -->
        <profile>
            <!-- 构建配置的唯一标识符。即用于命令行激活，也用于在继承时合并具有相同标识符的profile。 -->
            <id>test</id>

            <!-- 自动触发profile的条件逻辑。Activation是profile的开启钥匙。profile的力量来自于它能够
                 在某些特定的环境中自动使用某些特定的值；这些环境通过activation元素指定。activation元
                 素并不是激活profile的唯一方式。 -->
            <activation>
                <!-- profile默认是否激活的标志 -->
                <activeByDefault>true/false</activeByDefault>

                <!-- 当匹配的jdk被检测到，profile被激活。例如，1.4激活JDK1.4，1.4.0_2，而!1.4激活所有版本
                     不是以1.4开头的JDK。 -->
                <jdk>jdk版本，如:1.7</jdk>

                <!-- 当匹配的操作系统属性被检测到，profile被激活。os元素可以定义一些操作系统相关的属性。 -->
                <os>
                    <!-- 激活profile的操作系统的名字 -->
                    <name>Windows XP</name>

                    <!-- 激活profile的操作系统所属家族(如 'windows') -->
                    <family>Windows</family>

                    <!-- 激活profile的操作系统体系结构 -->
                    <arch>x86</arch>

                    <!-- 激活profile的操作系统版本 -->
                    <version>5.1.2600</version>
                </os>

                <!-- 如果Maven检测到某一个属性（其值可以在POM中通过${名称}引用），其拥有对应的名称和值，Profile
                     就会被激活。如果值字段是空的，那么存在属性名称字段就会激活profile，否则按区分大小写方式匹
                     配属性值字段 -->
                <property>
                    <!-- 激活profile的属性的名称 -->
                    <name>mavenVersion</name>

                    <!-- 激活profile的属性的值 -->
                    <value>2.0.3</value>
                </property>

                <!-- 提供一个文件名，通过检测该文件的存在或不存在来激活profile。missing检查文件是否存在，如果不存在则激活
                     profile。另一方面，exists则会检查文件是否存在，如果存在则激活profile。 -->
                <file>
                    <!-- 如果指定的文件存在，则激活profile。 -->
                    <exists>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</exists>

                    <!-- 如果指定的文件不存在，则激活profile。 -->
                    <missing>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</missing>
                </file>
            </activation>

            <!-- 构建项目所需要的信息。参见project>build元素 -->
            <build>
                <defaultGoal/>
                <resources>
                    <resource>
                        <targetPath></targetPath>
                        <filtering></filtering>
                        <directory></directory>
                        <includes>
                            <include></include>
                        </includes>
                        <excludes>
                            <exclude></exclude>
                        </excludes>
                    </resource>
                </resources>
                <testResources>
                    <testResource>
                        <targetPath></targetPath>
                        <filtering></filtering>
                        <directory></directory>
                        <includes>
                            <include></include>
                        </includes>
                        <excludes>
                            <exclude></exclude>
                        </excludes>
                    </testResource>
                </testResources>
                <directory></directory>
                <finalName></finalName>
                <filters></filters>
                <pluginManagement>
                    <plugins>
                        <!-- 参见build/pluginManagement/plugins/plugin元素 -->
                        <plugin>
                            <groupId></groupId>
                            <artifactId></artifactId>
                            <version></version>
                            <extensions>true/false</extensions>
                            <executions>
                                <execution>
                                    <id></id>
                                    <phase></phase>
                                    <goals></goals>
                                    <inherited>true/false</inherited>
                                    <configuration></configuration>
                                </execution>
                            </executions>
                            <dependencies>
                                <!-- 参见dependencies/dependency元素 -->
                                <dependency>
                                </dependency>
                            </dependencies>
                            <goals></goals>
                            <inherited>true/false</inherited>
                            <configuration></configuration>
                        </plugin>
                    </plugins>
                </pluginManagement>
                <plugins>
                    <!-- 参见build/pluginManagement/plugins/plugin元素 -->
                    <plugin>
                        <groupId></groupId>
                        <artifactId></artifactId>
                        <version></version>
                        <extensions>true/false</extensions>
                        <executions>
                            <execution>
                                <id></id>
                                <phase></phase>
                                <goals></goals>
                                <inherited>true/false</inherited>
                                <configuration></configuration>
                            </execution>
                        </executions>
                        <dependencies>
                            <!-- 参见dependencies/dependency元素 -->
                            <dependency>
                            </dependency>
                        </dependencies>
                        <goals></goals>
                        <inherited>true/false</inherited>
                        <configuration></configuration>
                    </plugin>
                </plugins>
            </build>

            <!-- 模块（有时称作子项目） 被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的
                 相对路径 -->
            <modules>
                <!--子项目相对路径-->
                <module>common-user</module>
                <module>common-util</module>
            </modules>

            <!-- 发现依赖和扩展的远程仓库列表。 -->
            <repositories>
                <!-- 参见repositories/repository元素 -->
                <repository>
                    <releases>
                        <enabled></enabled>
                        <updatePolicy></updatePolicy>
                        <checksumPolicy></checksumPolicy>
                    </releases>
                    <snapshots>
                        <enabled></enabled>
                        <updatePolicy></updatePolicy>
                        <checksumPolicy></checksumPolicy>
                    </snapshots>
                    <id></id>
                    <name></name>
                    <url></url>
                    <layout></layout>
                </repository>
            </repositories>

            <!-- 发现插件的远程仓库列表，这些插件用于构建和报表 -->
            <pluginRepositories>
                <!-- 包含需要连接到远程插件仓库的信息.参见repositories/repository元素 -->
                <pluginRepository>
                    <releases>
                        <enabled></enabled>
                        <updatePolicy></updatePolicy>
                        <checksumPolicy></checksumPolicy>
                    </releases>
                    <snapshots>
                        <enabled></enabled>
                        <updatePolicy></updatePolicy>
                        <checksumPolicy></checksumPolicy>
                    </snapshots>
                    <id></id>
                    <name></name>
                    <url></url>
                    <layout></layout>
                </pluginRepository>
            </pluginRepositories>

            <!-- 该元素描述了项目相关的所有依赖。 这些依赖组成了项目构建过程中的一个个环节。它们自动从项目定义的
                 仓库中下载。要获取更多信息，请看项目依赖机制。 -->
            <dependencies>
                <!-- 参见dependencies/dependency元素 -->
                <dependency>
                    <groupId>com.xyz</groupId>
                    <artifactId>xxx</artifactId>
                    <version>1.0.0</version>
                </dependency>
            </dependencies>

            <!-- 不赞成使用. 现在Maven忽略该元素. -->
            <reports></reports>

            <!-- 该元素包括使用报表插件产生报表的规范。当用户执行“mvn site”，这些报表就会运行。 在页面导航栏能看
                 到所有报表的链接。参见reporting元素 -->
            <reporting></reporting>

            <!-- 参见dependencyManagement元素 -->
            <dependencyManagement>
                <dependencies>
                    <!-- 参见dependencies/dependency元素 -->
                    <dependency>
                        <groupId>com.xyz</groupId>
                        <artifactId>xxx</artifactId>
                        <version>1.0.0</version>
                    </dependency>
                </dependencies>
            </dependencyManagement>

            <!-- 参见distributionManagement元素 -->
            <distributionManagement>
            </distributionManagement>

            <!-- 参见properties元素 -->
            <properties/>
        </profile>
    </profiles>

    <!-- 模块（有时称作子项目） 被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的相对路径 -->
    <modules>
        <!--子项目相对路径-->
        <module>common-user</module>
        <module>common-util</module>
    </modules>

    <!-- 发现依赖和扩展的远程仓库列表。 -->
    <repositories>
        <!-- 包含需要连接到远程仓库的信息 -->
        <repository>
            <!-- 如何处理远程仓库里发布版本的下载 -->
            <releases>
                <!-- true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
                <enabled>true/false</enabled>

                <!-- 该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），
                     daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
                <updatePolicy>never</updatePolicy>

                <!-- 当Maven验证构件校验文件失败时该怎么做：ignore（忽略），fail（失败），或者warn（警告）。 -->
                <checksumPolicy>warn</checksumPolicy>
            </releases>

            <!-- 如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，
                 为每种类型的构件采取不同的策略。例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->
            <snapshots>
                <enabled>true/false</enabled>
                <updatePolicy>never</updatePolicy>
                <checksumPolicy>warn</checksumPolicy>
            </snapshots>

            <!-- 远程仓库唯一标识符。可以用来匹配在settings.xml文件里配置的远程仓库 -->
            <id>banseon-repository-proxy</id>

            <!-- 远程仓库名称 -->
            <name>banseon-repository-proxy</name>

            <!-- 远程仓库URL，按protocol://hostname/path形式 -->
            <url>http://192.168.1.169:9999/repository/</url>

            <!-- 用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认
                 的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
            <layout>default</layout>
        </repository>
    </repositories>

    <!-- 发现插件的远程仓库列表，这些插件用于构建和报表 -->
    <pluginRepositories>
        <!-- 包含需要连接到远程插件仓库的信息.参见repositories/repository元素 -->
        <pluginRepository>
            <releases>
                <enabled>true/false</enabled>
                <updatePolicy>never</updatePolicy>
                <checksumPolicy>warn</checksumPolicy>
            </releases>
            <snapshots>
                <enabled>true/false</enabled>
                <updatePolicy>never</updatePolicy>
                <checksumPolicy>warn</checksumPolicy>
            </snapshots>
            <id>banseon-plugin-repository-proxy</id>
            <name>banseon-plugin-repository-proxy</name>
            <url>http://192.168.1.169:9999/repository/</url>
            <layout>default</layout>
        </pluginRepository>
    </pluginRepositories>

    <!-- 该元素描述了项目相关的所有依赖。 这些依赖组成了项目构建过程中的一个个环节。它们自动从项目定义的仓库中下载。
         要获取更多信息，请看项目依赖机制。 -->
    <dependencies>
        <dependency>
            <!-- 依赖的group ID -->
            <groupId>org.apache.maven</groupId>

            <!-- 依赖的artifact ID -->
            <artifactId>maven-artifact</artifactId>

            <!-- 依赖的版本号。 在Maven 2里, 也可以配置成版本号的范围。 -->
            <version>3.8.1</version>

            <!-- 依赖类型，默认类型是jar。它通常表示依赖的文件的扩展名，但也有例外。一个类型可以被映射成另外一个扩展
                 名或分类器。类型经常和使用的打包方式对应，尽管这也有例外。一些类型的例子：jar，war，ejb-client和test-jar。
                 如果设置extensions为 true，就可以在plugin里定义新的类型。所以前面的类型的例子不完整。 -->
            <type>jar</type>

            <!-- 依赖的分类器。分类器可以区分属于同一个POM，但不同构建方式的构件。分类器名被附加到文件名的版本号后面。例如，
                 如果你想要构建两个单独的构件成JAR，一个使用Java 1.4编译器，另一个使用Java 6编译器，你就可以使用分类器来生
                 成两个单独的JAR构件。
                 用来区分不同构建变体的分类器（如 sources, javadoc, tests 等）-->
            <classifier>sources</classifier>

            <!-- 依赖范围。在项目发布过程中，帮助决定哪些构件被包括进来。欲知详情请参考依赖机制。
                - compile ：默认范围，用于编译
                - provided：类似于编译，但支持你期待jdk或者容器提供，类似于classpath
                - runtime: 在执行时需要使用
                - test: 用于test任务时使用
                - system: 需要外在提供相应的元素。通过systemPath来取得
                - systemPath: 仅用于范围为system。提供相应的路径
                - optional: 当项目自身被依赖时，标注依赖是否传递。用于连续依赖时使用 -->
            <scope>test</scope>

            <!-- 仅供system范围使用。注意，不鼓励使用这个元素，并且在新的版本中该元素可能被覆盖掉。该元素为依赖规定了文件
                 系统上的路径。需要绝对路径而不是相对路径。推荐使用属性匹配绝对路径，例如${java.home}。 -->
            <systemPath>${java.home}/xxx/xyz.jar</systemPath>

            <!-- 当计算传递依赖时， 从依赖构件列表里，列出被排除的依赖构件集。即告诉maven你只依赖指定的项目，不依赖项目的
                 依赖。此元素主要用于解决版本冲突问题 -->
            <exclusions>
                <exclusion>
                    <artifactId>spring-core</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>

            <!-- 可选依赖，如果你在项目B中把C依赖声明为可选，你就需要在依赖于B的项目（例如项目A）中显式的引用对C的依赖。
                 可选依赖阻断依赖的传递性。 -->
            <optional>true</optional>
        </dependency>
    </dependencies>

    <!-- 不赞成使用. 现在Maven忽略该元素. -->
    <reports></reports>

    <!-- 该元素描述使用报表插件产生报表的规范。当用户执行“mvn site”，这些报表就会运行。 在页面导航栏能看到所有报表的链接。 -->
    <reporting>
        <!-- true，则，网站不包括默认的报表。这包括“项目信息”菜单中的报表。 -->
        <excludeDefaults/>

        <!-- 所有产生的报表存放到哪里。默认值是${project.build.directory}/site。 -->
        <outputDirectory/>

        <!-- 使用的报表插件和他们的配置。 -->
        <plugins>
            <!-- plugin元素包含描述报表插件需要的信息 -->
            <plugin>
                <!-- 报表插件在仓库里的group ID -->
                <groupId>com.xyz</groupId>
                <!-- 报表插件在仓库里的artifact ID -->
                <artifactId>xxx</artifactId>

                <!-- 被使用的报表插件的版本（或版本范围） -->
                <version>1.0.0</version>

                <!-- 任何配置是否被传播到子项目 -->
                <inherited>true/false</inherited>

                <!-- 报表插件的配置 -->
                <configuration></configuration>

                <!-- 一组报表的多重规范，每个规范可能有不同的配置。一个规范（报表集）对应一个执行目标 。例如，
                     有1，2，3，4，5，6，7，8，9个报表。1，2，5构成A报表集，对应一个执行目标。2，5，8构成B报
                     表集，对应另一个执行目标 -->
                <reportSets>
                    <!-- 表示报表的一个集合，以及产生该集合的配置 -->
                    <reportSet>
                        <!-- 报表集合的唯一标识符，POM继承时用到 -->
                        <id></id>

                        <!-- 产生报表集合时，被使用的报表的配置 -->
                        <configuration></configuration>

                        <!-- 配置是否被继承到子POMs -->
                        <inherited>true/false</inherited>

                        <!-- 这个集合里使用到哪些报表 -->
                        <reports></reports>
                    </reportSet>
                </reportSets>
            </plugin>
        </plugins>
    </reporting>

    <!-- 继承自该项目的所有子项目的默认依赖信息。这部分的依赖信息不会被立即解析,而是当子项目声明一个依赖
        （必须描述group ID和artifact ID信息），如果group ID和artifact ID以外的一些信息没有描述，则通过
            group ID和artifact ID匹配到这里的依赖，并使用这里的依赖信息。 -->
    <dependencyManagement>
        <dependencies>
            <!-- 参见dependencies/dependency元素 -->
            <dependency>
                <groupId>com.xyz</groupId>
                <artifactId>xxx</artifactId>
                <version>1.0.0</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- 项目分发信息，在执行mvn deploy后表示要发布的位置。有了这些信息就可以把网站部署到远程服务器或者
         把构件部署到远程仓库。 -->
    <distributionManagement>
        <!-- 部署项目产生的正式版本构件到远程仓库需要的信息 -->
        <repository>
            <!-- 是分配给快照一个唯一的版本号（由时间戳和构建流水号）？还是每次都使用相同的版本号？
                 uniqueVersion 元素用于控制在 Maven 中发布 快照（snapshot） 版本时，生成的构件是否附带一个唯一的时间戳标识。
                 它的默认值是 true，表示在上传快照版本时，Maven 会在生成的构件文件名中添加一个时间戳和构建编号，以确保每次部署的版本是唯一的 -->
            <uniqueVersion>false</uniqueVersion><!-- 在 releases 中通常不使用 uniqueVersion -->
            <id>banseon-maven2</id>
            <name>banseon maven2</name>
            <url>file://${basedir}/target/deploy</url>
            <layout>default</layout>
        </repository>

        <!-- 部署项目产生的快照版本构件到远程仓库需要的信息 -->
        <snapshotRepository>
            <!-- 是分配给快照一个唯一的版本号（由时间戳和构建流水号）？还是每次都使用相同的版本号？
                 uniqueVersion 元素用于控制在 Maven 中发布 快照（snapshot） 版本时，生成的构件是否附带一个唯一的时间戳标识。
                 它的默认值是 true，表示在上传快照版本时，Maven 会在生成的构件文件名中添加一个时间戳和构建编号，以确保每次部署的版本是唯一的 -->
            <uniqueVersion>true</uniqueVersion><!-- 在 snapshots 中通常启用 uniqueVersion -->
            <id>banseon-maven2</id>
            <name>Banseon-maven2 Snapshot Repository</name>
            <url>scp://svn.baidu.com/banseon:/usr/local/maven-snapshot</url>
            <layout>default</layout>
        </snapshotRepository>

        <!-- 部署项目的网站需要的信息 -->
        <site>
            <!-- 部署位置的唯一标识符，用来匹配站点和settings.xml文件里的配置 -->
            <id>banseon-site</id>

            <!-- 部署位置的名称 -->
            <name>business api website</name>

            <!-- 部署位置的URL，按protocol://hostname/path形式 -->
            <url>
                scp://svn.baidu.com/banseon:/var/www/localhost/banseon-web
            </url>
        </site>

        <!-- 项目下载页面的URL。如果没有该元素，用户应该参考主页。使用该元素的原因是：帮助定位
             那些不在仓库里的构件（由于license限制）。 -->
        <downloadUrl/>

        <!-- 如果构件有了新的group ID和artifact ID（构件移到了新的位置），这里列出构件的重定位信息。 -->
        <relocation>
            <!-- 构件新的group ID -->
            <groupId></groupId>

            <!-- 构件新的artifact ID -->
            <artifactId></artifactId>

            <!-- 构件新的版本号 -->
            <version></version>

            <!-- 显示给用户的，关于移动的额外信息，例如原因。 -->
            <message></message>
        </relocation>

        <!-- 给出该构件在远程仓库的状态。不得在本地项目中设置该元素，因为这是工具自动更新的。有效的值
             有：none（默认），converted（仓库管理员从Maven 1 POM转换过来），partner（直接从伙伴Maven
             2仓库同步过来），deployed（从Maven 2实例部署），verified（被核实时正确的和最终的）。 -->
        <status>deployed</status>
    </distributionManagement>
</project>
```



# 参考资料

- Maven POM Reference：https://maven.apache.org/pom.html
- 【SpringBoot】基于 Maven 的 pom.xml 配置详解【[奥特迦](https://blog.csdn.net/weixin_42201180)】：https://blog.csdn.net/weixin_42201180/article/details/127121085
- 关于pom.xml配置详解【[喝酸奶要舔盖儿](https://blog.csdn.net/qq_39361915)】：https://blog.csdn.net/qq_39361915/article/details/122621032
- Maven的配置文件pom.xml标签详解(含常用plugin)：https://blog.csdn.net/weixin_49999835/article/details/141468708
- Maven 教程之 pom.xml 详解：https://github.com/dunwu/blog/blob/master/source/_posts/01.Java/11.%E8%BD%AF%E4%BB%B6/01.%E6%9E%84%E5%BB%BA/01.Maven/02.Maven%E6%95%99%E7%A8%8B%E4%B9%8Bpom.xml%E8%AF%A6%E8%A7%A3.md