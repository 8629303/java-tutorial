# 项目中必备的 Maven 插件

> 10个Java项目中必备的Maven插件：https://mp.weixin.qq.com/s/KqP2Tp7iJ2TW77Qw_m5rMA

Maven是Java开发人员必备的构建工具和项目管理工具。它采用简单的项目对象模型（POM），实现Java应用程序的自动化构建、测试和部署。使用Maven插件，开发人员能够进一步扩展工具的功能，可以更好地完成项目。下面介绍Java开发人员在项目中应该使用的十个必备Maven插件。



## 1、Maven 编译器插件

Maven 编译器插件是 Maven 的一个核心插件，用于编译 Java 项目中的源代码。它能够确保您的 Java 代码与所指定的 Java 版本兼容，保证您顺利地进行编译和执行。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```



## 2、Maven Surefire 插件

Maven Surefire 插件用于运行单元测试和生成测试报告。它提供了广泛的支持，适用于主流的测试框架，如 JUnit 和 TestNG。

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>3.0.0-M5</version>
</plugin>
```



## 3、Maven Failsafe 插件

Maven Failsafe 插件是专门用于运行集成测试的插件。与传统的单元测试不同，集成测试涉及多个组件之间的相互作用和协调。即使在运行过程中出现某些测试失败的情况，Maven Failsafe 插件仍能确保构建过程继续进行，不会中断整个构建流程。这为开发团队提供了一个可靠的测试环境，验证不同组件之间的集成是否正常，在构建完成后解决测试失败产生的问题。这种机制保证了项目的稳定性和可靠性，提高整体质量。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.0.0-M5</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```



## 4、Maven Checkstyle 插件

Maven Checkstyle 插件用于在项目构建过程中执行静态代码分析和检查，通过分析代码并生成相应报告，指出需要改进的地方，确保项目代码符合编码标准和最佳实践。它能够帮助开发人员维持项目代码的一致性和可读性，确保项目的可维护性和可扩展性。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.1.2</version>
    <configuration>
        <configLocation>checkstyle.xml</configLocation>
    </configuration>
</plugin>
```



## 5、Maven FindBugs 插件

Maven FindBugs 插件也是用于静态代码分析的工具，用于检测 Java 代码中的潜在错误和问题，识别出可能导致错误、漏洞或低效性能的代码模式。FindBugs 插件能够检查常见的编程错误，如空指针引用、资源未关闭、不当使用同步等，并生成详细的报告，指出需要改进的地方。帮助开发人员改进代码质量。

相比之下，Maven Checkstyle 插件更注重于代码风格、格式和命名规范的强制执行。两者在代码质量提升方面起着互补的作用，可以帮助开发人员发现并改进不同类型的代码问题。

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>findbugs-maven-plugin</artifactId>
    <version>3.0.5</version>
</plugin>
```



## 6、Maven PMD 插件

Maven PMD 插件又是另一个静态分析工具。Maven PMD 插件主要关注的是代码质量和可维护性。用于检测不良实践、潜在的问题和编码规则的违反情况。例如，它可以检查未使用的变量、未使用的方法参数、重复的代码块等。Maven PMD 插件提供全面的报告，帮助开发人员确定改进的方向和优化代码的机会。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.14.0</version>
</plugin>
```



## 7、Maven JaCoCo 插件

官网：https://www.jacoco.org/jacoco/trunk/doc/index.html

Maven JaCoCo 插件是一个代码覆盖率工具，用于衡量测试覆盖的代码量。通过使用 Maven JaCoCo 插件，开发人员可以了解测试覆盖率的情况，帮助他们确定哪些区域需要进行更多的测试。

Maven JaCoCo 插件提供丰富的报告，包括行覆盖率、分支覆盖率、类覆盖率等指标，以及可视化的代码覆盖率报告。这些报告可以帮助开发人员识别测试覆盖的不足之处，并采取相应的措施来提高测试覆盖率，更全面地测试应用程序的各个部分。最终提高应用程序的可靠性和可维护性。

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.7</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```



## 8、Maven 依赖插件

Maven 依赖插件通过提供一系列有用的任务（如分析、复制和解压依赖项）来帮助开发人员管理项目的依赖关系。通过使用该插件，可以避免依赖冲突，确保项目使用正确的依赖版本，保证应用程序顺利运行。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.2</version>
</plugin>
```



## 9、Maven Javadoc 插件 

Maven Javadoc 插件可以为您的 Java 项目生成API文档。非常实用，有助于提高代码的可读性、可理解性和可维护性，同时也为使用您的代码的其他开发人员提供了一个方便的参考资源。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <show>private</show>
        <nohelp>true</nohelp>
    </configuration>
</plugin>
```



## 10、Maven 发布插件

Maven 发布插件通过一系列任务如自动化标记、版本控制和部署到存储库等，帮助开发人员自动化执行发布相关的任务，简化了项目发布过程。这确保了一致而高效的发布流程。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-release-plugin</artifactId>
    <version>3.0.0-M4</version>
    <configuration>
        <tagNameFormat>v@{project.version}</tagNameFormat>
    </configuration>
</plugin>
```



## 11、总结

这些十个必备的 Maven 插件可以增强您的 Java 项目的构建和管理过程。通过使用这些插件，您可以提高代码质量，保持一致性，自动化测试和部署，并简化开发工作流程。凭借这些强大的工具，您将能够处理复杂的Java项目并交付高质量的应用程序。



# Maven 中的自带插件

Maven 自带了一些常用的核心插件，这些插件无需在 pom.xml 中显式声明即可使用。以下是 Maven 自带的主要插件：

1. **maven-clean-plugin**
   - 作用：用于清理构建目录（通常是 `target/` 目录），删除上一次构建生成的文件。
   - 典型用法：mvn clean

2. **maven-compiler-plugin**
   - 作用：用于编译项目的源代码。
   - 典型用法：mvn compile

3. **maven-surefire-plugin**
   - 作用：用于运行项目中的单元测试。
   - 典型用法：mvn test

4. **maven-install-plugin** 
   - 作用：将项目构建的包安装到本地 Maven 仓库中，供本地项目依赖使用。
   - 典型用法：mvn install

5. **maven-deploy-plugin**
   - 作用：将项目构建的包部署到远程 Maven 仓库中，供其他开发者或项目使用。
   - 典型用法：mvn deploy

6. **maven-site-plugin**
   - 作用：生成项目的站点文档，包括 Javadoc、测试报告等。
   - 典型用法：mvn site

7. **maven-jar-plugin**
   - 作用：用于打包项目为一个 JAR 文件。
   - 典型用法：mvn package

8. **maven-resources-plugin**
   - 作用：用于复制并过滤项目的资源文件（例如 `src/main/resources` 文件夹中的文件）到输出目录。
   - 典型用法：mvn resources:resources

9. **maven-war-plugin**
   - 作用：用于打包 Web 应用程序为 WAR 文件，适用于 Java EE 项目。
   - 典型用法：mvn package（在 WAR 项目中）

10. **maven-pmd-plugin**
    - 作用：用于代码质量检查，帮助检测常见的编程错误和潜在问题。
    - 典型用法：mvn pmd:pmd

这些插件是 Maven 核心的一部分，并且自动在 Maven 构建生命周期中使用，不需要显式配置。通过这些插件，Maven 可以方便地执行清理、编译、测试、打包等常见的构建任务。







# Maven 插件之测试覆盖率 Jacoco

> Jacoco 单元测试配置：https://blog.csdn.net/wwtg9988/article/details/140671047

编写单元测试是开发健壮程序的有效途径，单元测试写的好不好可以从多个指标考量，其中一个就是单元测试的覆盖率。单元测试覆盖率可以看到我们的单元测试覆盖了多少代码行、类、分支等。查看单元测试覆盖率可以使用一些工具帮助我们计算，如 Idea 自带的单元测试工具、Jacoco 等。[Jacoco](https://www.jacoco.org/jacoco/) 是一款开源的单元测试工具，可以方便地整合到 Maven 中，提供了更丰富的配置及通过规则，可以生成单元测试报告，也可以方便地和自动化流水线整合。所以更推荐使用 Jacoco 作为单元测试工具。 



1、配置详解
--------------------------------------------------------------

### 1、Maven 配置

可以通过 Maven 插件的方式整合 Jacoco，这样使用 `mvn test` 命令就可以生成 Jacoco 的测试报告了。要配置 Jacoco，在 build 中添加 Jacoco 插件即可。

```xml
<build>
    <plugins>
        <!-- Jacoco -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <configuration>
                <!-- 排除文件 -->
                <excludes>
                    <exclude>**/*Test.class</exclude>
                    <exclude>**/*Configuration.class</exclude>
                    <exclude>**/*Properties.class</exclude>
                </excludes>
            </configuration>
            <executions>
                <execution>
                    <id>jacoco-prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>jacoco-report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                    <configuration>
                        <!--定义输出的文件夹-->
                        <outputDirectory>target/test-report</outputDirectory>
                        <footer>xyz</footer>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>

```

上面定义了两个执行，prepare-prepare-agent 是准备 Jacoco 的运行时代理，而 jacoco-report 则是生成报告。

然后执行 `mvn test` 或通过 IDE 的 Maven 面板执行测试，完成后就会在指定目录下生成 jacoco 的报告。默认会生成 csv 和 html 格式的，csv 可以用于自动化流水线整合，而 html 的可以用浏览器打开查看。



### 2、定义通过规则

我们可以使用 Jacoco 方便地配置通过规则，只有满足特定规则才能通过测试，否则 test 执行会失败。添加一个 execution 即可，在 rule 中定义我们的通过规则。

```xml
<execution>
    <id>jacoco-check</id>
    <goals>
        <goal>check</goal>
    </goals>
    <configuration>
        <rules>
            <rule>
                <element>BUNDLE</element>
                <excludes>
                    <exclude>**/*Test.class</exclude>
                    <exclude>**/*Configuration.class</exclude>
                    <exclude>**/*Properties.class</exclude>
                </excludes>
                <limits>
                    <limit>
                        <counter>LINE</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.8</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</execution>

```

上面定义了一条规则，包含一个限制：行覆盖率最低为 80%，[这里](https://github.com/52jing/wang-template-backend/blob/master/pom.xml)可查看完整案例。

其中 rule 标签定义了一条规则，element 定义了规则对应的范围，可选值有：

*   BUNDLE：表示整个模块，是默认值
*   PACKAGE：表示代码包
*   CLASS：表示类
*   SOURCEFILE：源文件
*   METHOD：表示方法

每个规则可以定义多条限制（limit），每个限制有一个特定的指标：

*   INSTRUCTION：字节码指令，是最细粒度的指标，默认值
*   LINE：代码行，一行代码可能有多个字节码指令
*   BRANCH：分支，if 或 switch 包含了多个分支
*   COMPLEXITY：圈复杂度，是代码复杂度的衡量标准，简单来说越大越复杂，需要的测试用例越多，详细算法可参考[百科](https://baike.baidu.com/item/%E5%9C%88%E5%A4%8D%E6%9D%82%E5%BA%A6/828737)
*   METHOD：方法
*   CLASS：类

为每个指标的值定义一个最大值或最小值。

*   TOTALCOUNT：总数
*   COVEREDCOUNT：覆盖的数量
*   MISSEDCOUNT：未覆盖的数量
*   COVEREDRATIO：覆盖率，范围从 0.0 到 1.0， 默认值
*   MISSEDRATIO：未覆盖率，范围从 0.0 到 1.0

excludes 标签定义要排除的文件。

例如，下面的规则定义了 INSTRUCTION 的覆盖率至少 80%，且没有类未被覆盖。

```xml
<rules>
    <rule>
        <element>BUNDLE</element>
        <limits>
            <limit>
                <counter>INSTRUCTION</counter>
                <value>COVEREDRATIO</value>
                <minimum>0.80</minimum>
            </limit>
            <limit>
                <counter>CLASS</counter>
                <value>MISSEDCOUNT</value>
                <maximum>0</maximum>
            </limit>
        </limits>
    </rule>
</rules>
```

规则的详细配置可参考[官方文档](https://www.jacoco.org/jacoco/trunk/doc/check-mojo.html)。



### 3、IDEA 配置

Idea IDE 除了支持自带的单元测试工具外，也支持使用 Jacoco。在配置中搜索 jacoco，Java 覆盖率中选择运行程序为 Jacoco 即可（不同的版本可能略有差别）。

路径：File Settings | Build, Execution, Deployment | Coverage | Java Coverage | Choose coverage runner | JaCoCo

配置后在 IDE 中执行单元测试时，选择第三项“使用覆盖率运行…”，运行完成后即可打开覆盖率面板，可查看当前单元测试的覆盖率，代码编辑器中也会以颜色标记。



### 4、排除测试

如果在代码中使用 Lombok，会有很多生成的代码，这些往往不需要测试覆盖，但如果不覆盖的话会影响测试覆盖率。Jacoco 可以自动排除 Lombok 的 @Generated 注解标记的类或方法。要让 Lombok 为生成的代码添加注解，可在项目的根目录中添加配置文件 `lombok.config`。然后添加一行配置。

```properties
config.stopbubbling = true
lombok.addLombokGeneratedAnnotation = true
```

这样生成的代码就会带上 @Generated 注解，被 Jacoco 排除计算了。当然，如果自己的代码不用测试，又不想影响覆盖率，也可以添加这个注解。



### 5、多模块聚合报告

如果我们的项目有多个模块，Jacoco 会在每个模块下生成一个单独的报告，能不能生成一个聚合的报告呢？Jacoco 也贴心地为我们提供了生成聚合报告的方法。

首先添加一个模块，专门用于生成聚合报告，可以只有一个 pom.xml 文件。

在该模块的 pom 中添加如下配置，report-aggregate 用于生成聚合的报告。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <executions>
                <execution>
                    <id>jacoco-report-aggregate</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report-aggregate</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>target/test-report</outputDirectory>
                        <footer>xyz</footer>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

也不要忘记在 pom 的 dependencies 中添加其他模块作为依赖。[这里](https://github.com/52jing/wang-template-backend/blob/master/report/pom.xml)可查看完整案例。这样就可以获得我们项目的整体单元测试覆盖率了哦。



2、理解报告
----------------------------------------------------------------

| Element | Missed Instructions Cov | Missed Branches Cov | Missed | Cxty | Missed | Lines | Missed | Methods | Missed | Classes |
| ------- | ----------------------- | ------------------- | ------ | ---- | ------ | ----- | ------ | ------- | ------ | ------- |
| com.xyz | 65%                     | 60%                 | 72     | 116  | 56     | 108   | 27     | 83      | 2      | 24      |

如果能理解规则定义，报告就非常容易理解了。如上面的截图所示，报告表格各个列的意思为：

*   Element：对应配置的 element，例如 BUNDLE 则是模块名
*   Missed Instructions Cov：字节码指令覆盖率（下面的数字是未覆盖数量 of 总数）
*   Missed Branches Cov：分支覆盖率（下面的数字是未覆盖数量 of 总数）
*   Missed / Cxty：未覆盖的以及总的圈复杂度
*   Missed / Lines：未覆盖的以及总的行数
*   Missed / Methods：未覆盖的以及总的方法
*   Missed / Classes：未覆盖的以及总的类

详情中可以看到每个文件的覆盖情况。其中，行的颜色代表的当前行的覆盖情况。红色代表未覆盖，黄色代码部分覆盖（分支条件没有全部执行到），绿色代码已覆盖。





# Maven 插件之代码检查

## 1、maven-checkstyle-plugin

**maven-checkstyle-plugin**：用于检测代码中不符合规范的地方。

1、定义 checkstyle.xml

```xml
<!DOCTYPE module PUBLIC
  "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
  "https://checkstyle.org/dtds/configuration_1_3.dtd">

<!-- Generated by RHY @will_awoke -->

<module name="Checker">

    <property name="charset" value="UTF-8"/>
    <property name="severity" value="warning"/>

    <!-- Checks for Size Violations.  -->
    <!-- 检查文件的长度（行） default max=2000 -->
    <module name="FileLength">
        <property name="max" value="2500"/>
    </module>

    <!-- Checks that property files contain the same keys. -->
    <!-- 检查**.properties配置文件 是否有相同的key
  <module name="Translation">
  </module>
  -->

    <module name="TreeWalker">

        <!-- Checks for imports    -->
        <!-- 必须导入类的完整路径，即不能使用*导入所需的类 -->
        <module name="AvoidStarImport"/>

        <!-- 检查是否从非法的包中导入了类 illegalPkgs: 定义非法的包名称-->
        <module name="IllegalImport"/> <!-- defaults to sun.* packages -->

        <!-- 检查是否导入了不必显示导入的类-->
        <module name="RedundantImport"/>

        <!-- 检查是否导入的包没有使用-->
        <module name="UnusedImports"/>

        <!-- Checks for whitespace
    <module name="EmptyForIteratorPad"/>
    <module name="MethodParamPad"/>
    <module name="NoWhitespaceAfter"/>
    <module name="NoWhitespaceBefore"/>
    <module name="OperatorWrap"/>
    <module name="ParenPad"/>
    <module name="TypecastParenPad"/>
    <module name="WhitespaceAfter"/>
    <module name="WhitespaceAround"/>
    -->

        <!-- 检查类和接口的javadoc 默认不检查author 和version tags
      authorFormat: 检查author标签的格式
            versionFormat: 检查version标签的格式
            scope: 可以检查的类的范围，例如：public只能检查public修饰的类，private可以检查所有的类
            excludeScope: 不能检查的类的范围，例如：public，public的类将不被检查，但访问权限小于public的类仍然会检查，其他的权限以此类推
            tokens: 该属性适用的类型，例如：CLASS_DEF,INTERFACE_DEF -->
        <module name="JavadocType">
            <property name="authorFormat" value="\S"/>
            <property name="scope" value="protected"/>
            <property name="tokens" value="CLASS_DEF,INTERFACE_DEF"/>
        </module>

        <!-- 检查方法的javadoc的注释
            scope: 可以检查的方法的范围，例如：public只能检查public修饰的方法，private可以检查所有的方法
            allowMissingParamTags: 是否忽略对参数注释的检查
            allowMissingThrowsTags: 是否忽略对throws注释的检查
            allowMissingReturntags: 是否忽略对return注释的检查 -->
        <module name="JavadocMethod">
            <property name="scope" value="private"/>
            <property name="allowMissingParamTags" value="false"/>
            <property name="allowMissingThrowsTags" value="false"/>
            <property name="allowMissingReturnTag" value="false"/>
            <property name="tokens" value="METHOD_DEF"/>
            <property name="allowUndeclaredRTE" value="true"/>
            <property name="allowThrowsTagsForSubclasses" value="true"/>
            <!--允许get set 方法没有注释-->
            <property name="allowMissingPropertyJavadoc" value="true"/>
        </module>

        <!-- 检查类变量的注释
            scope: 检查变量的范围，例如：public只能检查public修饰的变量，private可以检查所有的变量 -->
        <module name="JavadocVariable">
            <property name="scope" value="private"/>
        </module>

        <!--option: 定义左大括号'{'显示位置，eol在同一行显示，nl在下一行显示
      maxLineLength: 大括号'{'所在行行最多容纳的字符数
      tokens: 该属性适用的类型，例：CLASS_DEF,INTERFACE_DEF,METHOD_DEF,CTOR_DEF -->
        <module name="LeftCurly">
            <property name="option" value="nl"/>
        </module>

        <!-- NeedBraces 检查是否应该使用括号的地方没有加括号
      tokens: 定义检查的类型 -->
        <module name="NeedBraces"/>

        <!-- Checks the placement of right curly braces ('}') for  else, try, and catch tokens. The policy to verify is specified using property  option.
      option: 右大括号是否单独一行显示
      tokens: 定义检查的类型  -->
        <module name="RightCurly">
            <property name="option" value="alone"/>
        </module>

        <!-- 检查在重写了equals方法后是否重写了hashCode方法 -->
        <module name="EqualsHashCode"/>

        <!--  Checks for illegal instantiations where a factory method is preferred.
      Rationale: Depending on the project, for some classes it might be preferable to create instances through factory methods rather than calling the constructor.
      A simple example is the java.lang.Boolean class. In order to save memory and CPU cycles, it is preferable to use the predefined constants TRUE and FALSE. Constructor invocations should be replaced by calls to Boolean.valueOf().
      Some extremely performance sensitive projects may require the use of factory methods for other classes as well, to enforce the usage of number caches or object pools. -->
        <module name="IllegalInstantiation">
            <property name="classes" value="java.lang.Boolean"/>
        </module>

        <!-- Checks for Naming Conventions.   命名规范   -->
        <!-- local, final variables, including catch parameters -->
        <module name="LocalFinalVariableName"/>

        <!-- local, non-final variables, including catch parameters-->
        <module name="LocalVariableName"/>

        <!-- static, non-final fields -->
        <module name="StaticVariableName">
            <property name="format" value="(^[A-Z0-9_]{0,19}$)"/>
        </module>

        <!-- packages -->
        <module name="PackageName">
            <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$"/>
        </module>

        <!-- classes and interfaces -->
        <module name="TypeName">
            <property name="format" value="(^[A-Z][a-zA-Z0-9]{0,19}$)"/>
        </module>

        <!-- methods -->
        <module name="MethodName">
            <property name="format" value="(^[a-z][a-zA-Z0-9]{0,19}$)"/>
        </module>

        <!-- non-static fields -->
        <module name="MemberName">
            <property name="format" value="(^[a-z][a-z0-9][a-zA-Z0-9]{0,19}$)"/>
        </module>

        <!-- parameters -->
        <module name="ParameterName">
            <property name="format" value="(^[a-z][a-zA-Z0-9_]{0,19}$)"/>
        </module>

        <!-- constants (static,  final fields) -->
        <module name="ConstantName">
            <property name="format" value="(^[A-Z0-9_]{0,19}$)"/>
        </module>

        <!-- 代码缩进   -->
        <module name="Indentation">
        </module>

        <!-- Checks for redundant exceptions declared in throws clause such as duplicates, unchecked exceptions or subclasses of another declared exception.
      检查是否抛出了多余的异常
    <module name="RedundantThrows">
        <property name="logLoadErrors" value="true"/>
        <property name="suppressLoadErrors" value="true"/>
    </module>
    -->

        <!--  Checks for overly complicated boolean expressions. Currently finds code like  if (b == true), b || true, !false, etc.
      检查boolean值是否冗余的地方
      Rationale: Complex boolean logic makes code hard to understand and maintain. -->
        <module name="SimplifyBooleanExpression"/>

        <!--  Checks for overly complicated boolean return statements. For example the following code
       检查是否存在过度复杂的boolean返回值
       if (valid())
          return false;
       else
          return true;
       could be written as
          return !valid();
       The Idea for this Check has been shamelessly stolen from the equivalent PMD rule. -->
        <module name="SimplifyBooleanReturn"/>

        <!-- Checks that a class which has only private constructors is declared as final.只有私有构造器的类必须声明为final-->
        <module name="FinalClass"/>

        <!--  Make sure that utility classes (classes that contain only static methods or fields in their API) do not have a public constructor.
      确保Utils类（只提供static方法和属性的类）没有public构造器。
      Rationale: Instantiating utility classes does not make sense. Hence the constructors should either be private or (if you want to allow subclassing) protected. A common mistake is forgetting to hide the default constructor.
      If you make the constructor protected you may want to consider the following constructor implementation technique to disallow instantiating subclasses:
      public class StringUtils // not final to allow subclassing
      {
          protected StringUtils() {
              throw new UnsupportedOperationException(); // prevents calls from subclass
          }
          public static int count(char c, String s) {
              // ...
          }
      }
   <module name="HideUtilityClassConstructor"/>
   -->

        <!--  Checks visibility of class members. Only static final members may be public; other class members must be private unless property protectedAllowed or packageAllowed is set.
      检查class成员属性可见性。只有static final 修饰的成员是可以public的。其他的成员属性必需是private的，除非属性protectedAllowed或者packageAllowed设置了true.
       Public members are not flagged if the name matches the public member regular expression (contains "^serialVersionUID$" by default). Note: Checkstyle 2 used to include "^f[A-Z][a-zA-Z0-9]*$" in the default pattern to allow CMP for EJB 1.1 with the default settings. With EJB 2.0 it is not longer necessary to have public access for persistent fields, hence the default has been changed.
       Rationale: Enforce encapsulation. 强制封装 -->
        <module name="VisibilityModifier"/>

        <!-- 每一行只能定义一个变量 -->
        <module name="MultipleVariableDeclarations">
        </module>

        <!-- Checks the style of array type definitions. Some like Java-style: public static void main(String[] args) and some like C-style: public static void main(String args[])
      检查再定义数组时，采用java风格还是c风格，例如：int[] num是java风格，int num[]是c风格。默认是java风格-->
        <module name="ArrayTypeStyle">
        </module>

        <!-- Checks that there are no "magic numbers", where a magic number is a numeric literal that is not defined as a constant. By default, -1, 0, 1, and 2 are not considered to be magic numbers.
    <module name="MagicNumber">
    </module>
    -->

        <!-- A check for TODO: comments. Actually it is a generic regular expression matcher on Java comments. To check for other patterns in Java comments, set property format.
       检查是否存在TODO（待处理） TODO是javaIDE自动生成的。一般代码写完后要去掉。
     -->
        <module name="TodoComment"/>

        <!--  Checks that long constants are defined with an upper ell. That is ' L' and not 'l'. This is in accordance to the Java Language Specification,  Section 3.10.1.
      检查是否在long类型是否定义了大写的L.字母小写l和数字1（一）很相似。
      looks a lot like 1. -->
        <module name="UpperEll"/>

        <!--  Checks that switch statement has "default" clause. 检查switch语句是否有‘default’从句
       Rationale: It's usually a good idea to introduce a default case in every switch statement.
       Even if the developer is sure that all currently possible cases are covered, this should be expressed in the default branch,
        e.g. by using an assertion. This way the code is protected aginst later changes, e.g. introduction of new types in an enumeration type. -->
        <module name="MissingSwitchDefault"/>

        <!--检查switch中case后是否加入了跳出语句，例如：return、break、throw、continue -->
        <module name="FallThrough"/>

        <!-- Checks the number of parameters of a method or constructor. max default 7个. -->
        <module name="ParameterNumber">
            <property name="max" value="5"/>
        </module>

        <!-- 每行字符数 -->
        <module name="LineLength">
            <property name="max" value="200"/>
        </module>

        <!-- Checks for long methods and constructors. max default 150行. max=300 设置长度300 -->
        <module name="MethodLength">
            <property name="max" value="300"/>
        </module>

        <!-- ModifierOrder 检查修饰符的顺序，默认是 public,protected,private,abstract,static,final,transient,volatile,synchronized,native -->
        <module name="ModifierOrder">
        </module>

        <!-- 检查是否有多余的修饰符，例如：接口中的方法不必使用public、abstract修饰  -->
        <module name="RedundantModifier">
        </module>

        <!--- 字符串比较必须使用 equals() -->
        <module name="StringLiteralEquality">
        </module>

        <!-- if-else嵌套语句个数 最多4层 -->
        <module name="NestedIfDepth">
            <property name="max" value="3"/>
        </module>

        <!-- try-catch 嵌套语句个数 最多2层 -->
        <module name="NestedTryDepth">
            <property name="max" value="2"/>
        </module>

        <!-- 返回个数 -->
        <module name="ReturnCount">
            <property name="max" value="5"/>
            <property name="format" value="^$"/>
        </module>

    </module>
</module>
```

2、配置 `pom.xml`：

```xml

<project>
    ...
    <properties>
        <checkstyle.config.location>config/maven_checks.xml</checkstyle.config.location>
    </properties>
    ...
    <reporting>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>3.0</version>
                <executions>
                    <execution>
                        <!-- 绑定pmd:pmd到validate生命周期，在validate时会自动进行代码规范检查 -->
                        <id>validate</id>
                        <phase>validate</phase>
                        <configuration>
                            <!-- 配置文件的路径，在style文件夹下 -->
                            <configLocation>style/checkstyle.xml</configLocation>
                            <encoding>UTF-8</encoding>
                            <consoleOutput>true</consoleOutput>
                            <failsOnError>true</failsOnError>
                            <includeTestSourceDirectory>false</includeTestSourceDirectory>
                        </configuration>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jxr-plugin</artifactId>
                <version>2.3</version>
            </plugin>
        </plugins>
    </reporting>
    ...
</project>
```

其中可以修改使用的检查规则文件路径，插件默认提供了四个规则文件可以直接使用，无需手动下载：

- config/sun_checks.xml - Sun Microsystems Definition (default).
- config/maven_checks.xml - Maven Development Definitions.
- config/turbine_checks.xml - Turbine Development Definitions.
- config/avalon_checks.xml - Avalon Development Definitions.

配置好后，可以执行 `mvn clean checkstyle:check` 检查代码。



## 2、maven-pmd-plugin

**maven-pmd-plugin**：是阿里编程规范检查插件。

1、配置 `pom.xml`：参考 https://github.com/alibaba/p3c/blob/master/p3c-pmd/pom.xml 配置。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <sourceEncoding>${project.build.sourceEncoding}</sourceEncoding>
        <targetJdk>${maven.compiler.target}</targetJdk>
        <printFailingErrors>true</printFailingErrors>
        <rulesets>
            <ruleset>rulesets/java/ali-comment.xml</ruleset>
            <ruleset>rulesets/java/ali-concurrent.xml</ruleset>
            <ruleset>rulesets/java/ali-constant.xml</ruleset>
            <ruleset>rulesets/java/ali-exception.xml</ruleset>
            <ruleset>rulesets/java/ali-flowcontrol.xml</ruleset>
            <ruleset>rulesets/java/ali-naming.xml</ruleset>
            <ruleset>rulesets/java/ali-oop.xml</ruleset>
            <ruleset>rulesets/java/ali-orm.xml</ruleset>
            <ruleset>rulesets/java/ali-other.xml</ruleset>
            <ruleset>rulesets/java/ali-set.xml</ruleset>
        </rulesets>
        <printFailingErrors>true</printFailingErrors>
    </configuration>
    <executions>
        <execution>
            <phase>verify</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.p3c</groupId>
            <artifactId>p3c-pmd</artifactId>
            <version>2.0.0</version>
        </dependency>
    </dependencies>
</plugin>
</plugins>
```

2、配置好后，可以执行 `mvn clean pmd:check` 检查代码。



## 3、参考资料

- https://maven.apache.org/plugins/maven-checkstyle-plugin/
- https://maven.apache.org/jxr/maven-jxr-plugin/
- https://www.jianshu.com/p/557b975ae40d
- 阿里巴巴编程规范
  - https://github.com/alibaba/p3c
  - https://github.com/alibaba/p3c/blob/master/p3c-pmd/pom.xml





# Maven 插件之自定义开发

> - 怎么写一个maven插件来提升生产效率？https://mp.weixin.qq.com/s/ESAhXzcckmF07a2R3edHeQ
> - 现学现用，写个Maven插件用下：https://mp.weixin.qq.com/s/ek9R2GGOfzgYINO4uay3hA

## 1、自定义插件开发入门

如果你的项目是基于maven构建，那么在项目的pom文件中，经常会看到这样的配置

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

在上面的代码中，我们使用了插件`maven-compiler-plugin`，这个插件的作用是将编写的java代码编译成指定的版本与编码的 class 文件。



### 1、Maven 插件的组成

- goal

Maven 中的插件是由很多目标（goal）组成的，开发插件，实际上就是去编写插件中目标的具体代码。每个目标对应一个 Java 类，这个类在 Maven 中叫做 MOJO，Maven 提供了一个 Mojo 的接口，我们开发插件也就是去实现这个接口

```
org.apache.maven.plugin.Mojo
```

通过 goal 我们可以定义插件在 Maven 哪个生命周期中执行，比如上面的`maven-compiler-plugin`在`compile`阶段执行。

- configuration

插件执行阶段，我们可以通过配置定义各种参数，这样就能根据不同参数按需求执行插件，比如上面的`maven-compiler-plugin`中，在configuration 下的配置

```xml
<configuration>
    <source>1.8</source>
    <target>1.8</target>
    <encoding>UTF-8</encoding>
</configuration>
```

maven-compiler-plugin 的配置信息中，source 和 target 属性指定了编译的 Java 版本，encoding 属性指定了编译结果的编码。



### 2、Maven 插件的分类

Maven 插件分为两种：

- 核心插件：Maven 自带的插件，如 maven-compiler-plugin，maven-surefire-plugin 等。
- 非核心插件：第三方开发的插件，如 sonar-maven-plugin，pmd-maven-plugin 等。





### 3、插件的生命周期

Maven插件的生命周期分为三个阶段：

- 初始化阶段：在这个阶段，Maven 会将插件的配置信息加载到一个`org.apache.maven.plugin.Plugin`对象中，这个对象中包含了插件的配置信息，如插件的id，version，dependencies 等。
- 执行阶段：在这个阶段，Maven 会调用插件的`execute()`方法，这个方法就是插件的核心，这个方法的执行是有顺序的，先执行`initialize()`方法，然后执行`execute()`方法
- 销毁阶段：在这个阶段，Maven会调用插件的`cleanup()`方法，这个方法的执行是无顺序的，先执行`execute()`方法，然后执行`cleanup()`方法。



### 4、Maven 插件的执行

Maven插件的执行分为两种：

- 手动执行：在命令行中，使用 mvn 插件的 id，如 mvn compiler:compile，这个命令会先执行插件的初始化阶段，然后执行插件的执行阶段，最后执行插件的销毁阶段。
- 自动执行：在 pom.xml 文件中配置插件的执行，如 maven-compiler-plugin，这个配置会先执行插件的初始化阶段，然后执行插件的执行阶段，最后执行插件的销销毁阶段。

比如基于 IDEA 时，我们可以在 Maven 工具栏，通过 Lifecycle 菜单中，选择相应的生命周期函数，然后点击 Run 按钮，即可执行插件。比如插件配置的在 compile 阶段执行，那么配置在 compile 对应的 goal 都会触发。

同样我们可以直接通过 mvn 命令调用插件，这样插件就不需要依赖于 Maven 的生命周期函数去执行了。

```bash
# mvn groupId:artifactId:goal -Dprop=value
mvn com.sucls.blog:build-maven-plugin:printDate -Dname=XX
```



### 5、Maven 插件的编写

下面通过一个简单的示例来演示如何写一个 Maven 插件。

1. 创建一个 Maven 项目
2. 添加相关依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-plugin-api</artifactId>
        <version>${maven-plugin-api.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.maven.plugin-tools</groupId>
        <artifactId>maven-plugin-annotations</artifactId>
        <version>${maven-plugin-annotations.version}</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

1. 编写插件代码

```java
@Mojo(name = "printDate")
public class PrintDatePlugin extends AbstractMojo {

    @Parameter(property = "name",defaultValue = "printDate")
    private String name;

    @Parameter(property = "description",defaultValue = "打印日期插件")
    private String description;

    @Parameter(property = "format",defaultValue = "yyyy-MM-dd HH:mm:ss")
    private String format;

    @Override
    public void execute() throws MojoExecutionException, MojoFailureException {
        Date date = new Date();
        Log log = getLog();
        log.info(StringUtils.repeat("=",20));
        log.info(String.format("开始执行插件：%s", name));
        log.info(String.format("插件信息：%s", description));
        log.info(String.format("执行插件：%s", DateFormatUtils.format(date,format)));
        log.info(StringUtils.repeat("=",20));
    }
}
```

1. 打包插件

```bash
# 执行下面的命令 或者通过 idea中maven工具栏的package直接打包
mvn clean package
```

1. 使用插件 创建一个新的项目，在pom.xml中添加插件的配置信息

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.sucls.blog</groupId>
            <artifactId>build-maven-plugin</artifactId>
            <version>${project.version}</version>
            <executions>
                <execution>
                    <id>print-date</id>
                    <phase>compile</phase>
                    <goals>
                        <goal>printDate</goal>
                    </goals>
                    <configuration>
                        <name>PD</name>
                        <format>yyyy-MM-dd</format>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

1. 执行插件

```bash
mvn clean complie
```

观察控制台可以看到这样的信息：

```bash
[INFO] --- build-maven-plugin:1.0-SNAPSHOT:printDate (print-date) @ build-maven-plugin ---
[INFO] 
[INFO] ===============
[INFO] 开始执行插件：PD
[INFO] 插件信息：打印日期插件
[INFO] 执行插件：2024-09-25
[INFO] ===============
```

上面的示例中，主要做了下面的事情：

1. 定义了一个打印日期的插件 也就是一个 Maven 项目。
2. 为插件定义 goal 一个 goal 也就是一个 org.apache.maven.plugin.Mojo 类，通过 @Mojo 注解，定义了插件的名称和goal，一个插件中可以包含多个 goal，在配置时同样可以对应多个。
3. 为插件添加参数 在每一个 Mojo 中，通过 @Parameter 定义插件相关参数，这样在插件执行阶段即可通过参数完成对应逻辑。
4. 在其他项目中引入插件 在 pom 文件中，像其他插件一样通过 groupId:artifactId:version 引入自己定义的插件，配置 Maven 的哪个生命周期阶段执行插件的 goal ，以及插件的参数。
5. 使用插件 Maven 生命周期中，各个阶段都会找到所关联的插件并执行。

**总结**：通过 Maven 插件，可以根据我们的项目，定义一些重复，但耗时的编码工作，通过插件的形式，可以自动化完成这些工作，从而提高开发效率。









## 2、自动创建 Module 插件

**Maven 插件实践**

上一次简单介绍了如何编写一个Maven插件，并且如何将插件的执行与Maven生命周期绑定，这样通过调用maven生命周期方法时，则会在配置的阶段按照插件的目标来执行代码。

今天通过一个具体的插件来熟悉在项目中的使用。

一般公司的项目结构或者代码结构都是非常固定的，有一些框架针对这种固化的代码结构或约定的规范，在开发前会严格对项目进行模块划分，对各个模块的代码结构也会严格要求。那么我们则可以根据这种约定的规范，通过工具来自动化的生成代码，从而减少开发人员的工作量。



### 1、示例项目

比如我们的项目一般都会由多个模块组成，比如下面的示例：

```bash
   test-module
   ├ system
   │ ├ account
   │   ├ entity
   │   ├ dao
   │   └ service
   │   └ web
   │ ├ role
   │   ├ ...
   │ ├ permission
   │   ├ ...    
   │ └ pom.xml
   ├ monitor
   │ ├ database
   │   ├ ...
   │ ├ disk
   │   ├ ...
   │ ├ memory
   │   ├ ...    
   │ └ pom.xml
   └ pom.xml
```

在上面的例子中，我们项目包括了system、monitor等多个模块，其中system模块包含了account、role、permission三个子模块， monitor模块包含了database、disk、memory三个子模块，每个子模块又包含了特定的代码结构. 这个属于我们约定的模块划分规则，那么基于这样的规则，我们完全可以通过开发一个插件来自动生成这种约定结构的空项目。



### 2、实现步骤

假设插件名称为 `module-create-plugin`，将来我们会通过该插件实现项目模块文件夹和一些通用文件的自动生成。插件大概配置如下：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.example</groupId>
            <artifactId>module-create-plugin</artifactId>
            <version>1.0-SNAPSHOT</version>
            <configuration>
                <project>test-module</project>
                <basedir>${project.basedir}</basedir>
                <modules>
                    <module>system/account</module>
                    <module>system/role</module>
                    <module>monitor/database</module>
                    <module>monitor/disk</module>
                    <module>monitor/memory</module>
                </modules>
            </configuration>
        </plugin>
    </plugins>
</build>
```

根据我们预期要求，来设想插件的开发过程。



#### 1、定义一个 Maven 插件项目

创建一个maven插件项目，在pom.xml中添加如下配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sucls.blog.plugin</groupId>
    <artifactId>module-create-plugin</artifactId>
    <version>1.0.0</version>
    <packaging>maven-plugin</packaging>

    <dependencies>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-plugin-api</artifactId>
            <version>3.8.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-core</artifactId>
            <version>3.8.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugin-tools</groupId>
            <artifactId>maven-plugin-annotations</artifactId>
            <version>3.8.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.32</version>
        </dependency>
    </dependencies>
</project>
```



#### 2、创建一个 Mojo 根据配置添加实现逻辑

goal 的名字设计成`modules`，由于最终项目是基于 Maven 构建，所以会生成 pom.xml，插件中的几个参数就是为了生成 pom 而设计的。

```java
package org.example;

import org.apache.commons.lang3.StringUtils;
import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.logging.Log;
import org.apache.maven.plugins.annotations.Mojo;
import org.apache.maven.plugins.annotations.Parameter;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.*;
import java.util.stream.Stream;

@Mojo(name = "run")
public class ModulesCreatePlugin extends AbstractMojo {

    private final Log log = getLog();

    public static final String POM_XML = "pom.xml";
    public static final String POM_FTL = "pom.ftl";

    @Parameter(property = "basedir", defaultValue = "${project.basedir}")
    private String basedir;
    @Parameter
    private String project;

    @Parameter(property = "groupId", defaultValue = "${project.groupId}")
    private String groupId;
    @Parameter(property = "artifactId", defaultValue = "${project.artifactId}")
    private String artifactId;
    @Parameter(property = "version", defaultValue = "${project.version}")
    private String version;

    @Parameter
    private List<String> modules;


    private ModuleTemplateHelper moduleTemplateHelper;

    public ModulesCreatePlugin() {
        init();
    }

    public void init() {
        moduleTemplateHelper = new ModuleTemplateHelper();
    }

    @Override
    public void execute() {
        log.info(StringUtils.repeat("=", 50));
        try {
            createModules();
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        log.info(StringUtils.repeat("=", 50));
    }

    private void createModules() throws IOException {
        if (modules != null && !modules.isEmpty()) {
            log.info(modules.toString());

            String path = Paths.get(basedir).getParent().toString();
            String projectPath = path + File.separatorChar + project;

            List<File> moduleFiles = new ArrayList<>();

            // 先清理目录
            deleteDirectory(Paths.get(projectPath));
            // 创建目录
            for (String module : modules) {
                Path directory = Files.createDirectories(Paths.get(projectPath, module));
                moduleFiles.add(directory.toFile());
            }

            Set<String> parentModules = new HashSet<>();

            // 添加POM.XML
            for (File module : moduleFiles) {
                File parent = module.getParentFile();
                parentModules.add(parent.getName());
                // 上级POM
                ModuleEntity parentModuleEntity = new ModuleEntity(groupId, artifactId, version, parent.getName());
                if (!new File(parent, POM_XML).exists()) {
                    moduleTemplateHelper.process(POM_FTL, parentModuleEntity, parent.getAbsolutePath() + "/pom.xml");
                }
                // 追加MODULE
                XmlUtils.appendModule(new File(parent, POM_XML), module.getName());

                // 模块POM
                ModuleEntity moduleEntity = new ModuleEntity(groupId, parent.getName(), version, module.getName());
                moduleTemplateHelper.process(POM_FTL, moduleEntity, module.getAbsolutePath() + "/pom.xml");
                Files.createDirectories(module.toPath().resolve("src/main/java"));
                Files.createDirectories(module.toPath().resolve("src/main/resources"));
                Files.createDirectories(module.toPath().resolve("src/test/java"));
                Files.createDirectories(module.toPath().resolve("src/test/resources"));
            }

            // 项目POM.XML追加MODULE
            if (Files.exists(Paths.get(projectPath, POM_XML))) {
                parentModules.forEach(parentModule -> XmlUtils.appendModule(new File(projectPath, POM_XML), parentModule));
            }
        }
    }

    /**
     * 递归删除指定目录下的所有文件及目录
     * @param path
     * @throws IOException
     */
    public static void deleteDirectory(Path path) throws IOException {
        if (Files.exists(path) && Files.isDirectory(path)) {
            Stream<Path> walk = Files.walk(path);
            // 先删除子目录和文件，再删除主目录
            walk.sorted(Comparator.reverseOrder()).forEach(p -> {
                try {
                    Files.delete(p);
                } catch (IOException e) {
                    throw new IllegalArgumentException("Failed to delete " + p, e);
                }
            });
            walk.close();
        }
    }
}
```



#### 3、添加模版类及工具类

上面我们通过`ModuleTemplateHelper`辅助类结合`freemaker`框架，最后为各个模块生成对应的pom.xml文件；通过自定义的XmlUtils工具类结合JDK Documentation API，实现父级模块中`modules`节点的添加子module；

1、ModuleEntity

```java
package org.example;

public class ModuleEntity {
    private String groupId;
    private String artifactId;
    private String version;
    private String name;

    public ModuleEntity() {
    }

    public ModuleEntity(String groupId, String artifactId, String version, String name) {
        this.groupId = groupId;
        this.artifactId = artifactId;
        this.version = version;
        this.name = name;
    }

    public String getGroupId() {
        return groupId;
    }

    public void setGroupId(String groupId) {
        this.groupId = groupId;
    }

    public String getArtifactId() {
        return artifactId;
    }

    public void setArtifactId(String artifactId) {
        this.artifactId = artifactId;
    }

    public String getVersion() {
        return version;
    }

    public void setVersion(String version) {
        this.version = version;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

2、ModuleTemplateHelper

```java
package org.example;


import freemarker.cache.ClassTemplateLoader;
import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;

import java.io.FileWriter;
import java.io.IOException;

public class ModuleTemplateHelper {

    private final Configuration configuration;

    public ModuleTemplateHelper() {
        configuration = new Configuration(Configuration.VERSION_2_3_22);
        configuration.setTemplateLoader(new ClassTemplateLoader(this.getClass(), "/templates"));
        configuration.setDefaultEncoding("UTF-8");
    }

    public void process(String tpl, Object module, String outputPath) {
        try {
            Template template = configuration.getTemplate(tpl);
            template.process(module, new FileWriter(outputPath));
        } catch (IOException | TemplateException e) {
            throw new IllegalArgumentException("Error processing template: " + tpl, e);
        }
    }
}
```

3、XmlUtils

```java
package org.example;

import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.OutputKeys;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.File;

public class XmlUtils {

    private XmlUtils(){}

    static DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();
    static TransformerFactory transformerFactory = TransformerFactory.newInstance();
    static DocumentBuilder documentBuilder;

    static {
        try {
            documentBuilder = documentBuilderFactory.newDocumentBuilder();
        } catch (ParserConfigurationException e) {
            throw new IllegalArgumentException(e);
        }
    }


    /**
     * @param pomXml
     * @param moduleName
     */
    public static void appendModule(File pomXml, String moduleName) {
        try {
            Document document = documentBuilder.parse(pomXml);
            NodeList modules = document.getElementsByTagName("modules");
            Node modulesNode = null;
            if (modules.getLength() > 0) {
                modulesNode = modules.item(0);
            }
            if (modulesNode == null) {
                modulesNode = document.createElement("modules");
                Node projectNode = document.getElementsByTagName("project").item(0); // 获取 <project> 节点
                projectNode.appendChild(modulesNode); // 将 modules 节点添加到 project 节点下
            }
            // 追加
            Element module = document.createElement("module");
            module.setTextContent(moduleName);
            modulesNode.appendChild(module);

            // 保存并且格式化
            Transformer transformer = transformerFactory.newTransformer();
            // 如下为格式化代码
//            transformer.setOutputProperty(OutputKeys.INDENT, "yes");
//            transformer.setOutputProperty("{https://xml.apache.org/xslt}indent-amount", "4"); // 设置缩进大小
            transformer.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, "no"); // 输出声明
            transformer.setOutputProperty(OutputKeys.METHOD, "xml");
            transformer.setOutputProperty(OutputKeys.ENCODING, "UTF-8"); // 设置编码格式
            // 设置格式化输出
            transformer.transform(new DOMSource(document), new StreamResult(pomXml));
        } catch (Exception e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```



#### 4、打包生成插件

执行下面的命令即可生成插件jar，并安装到本地仓库

```bash
mvn clean install
```



#### 5、在项目中调用插件

1、新建一个空目录，目录名为：init-project，然后添加pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>module-init</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.example</groupId>
                <artifactId>module-create-plugin</artifactId>
                <version>1.0-SNAPSHOT</version>
                <configuration>
                    <project>test-module</project>
                    <basedir>${project.basedir}</basedir>
                    <modules>
                        <module>system/account</module>
                        <module>system/role</module>
                        <module>monitor/database</module>
                        <module>monitor/disk</module>
                        <module>monitor/memory</module>
                    </modules>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

2、执行如下命令启动插件

```bash
mvn module-create:run
```

3、最终项目结构如下：

```
test-module
   ├ system
   │ ├ account
   │   └ pom.xml
   │ ├ role
   │   └ pom.xml
   │ ├ permission
   │   └ pom.xml  
   │ └ pom.xml
   ├ monitor
   │ ├ database
   │   └ pom.xml
   │ ├ disk
   │   └ pom.xml
   │ ├ memory
   │   └ pom.xml  
   │ └ pom.xml
```

> 也可以直接在IDEA中可视化操作：在项目中引入插件，并且按照需要的模块添加配置，最后在IDEA右侧则可以看到该插件，双击运行。


