# 一、Maven 基本信息

1、什么叫做 Maven
---------

在上一篇中我们简单的介绍了一下 Maven实战这本书的简介以及相关的背景，在这一篇中我们开始详细的介绍一下这本书的内容。今天我们分享的内容主要是详细的介绍一下Maven的基本信息。例如什么叫做Maven，为什么我们需要Maven，以及如何安装它。

首先我们从英语的角度看什么是Maven。在英语的角度上看Maven可以翻译为“内行、专家”。所以当我们以后看到这个单词的时候，它除了是一项技术外，我们还学习了一个新的单词的含义。那么在技术领域的角度上看。Maven实际上是一个跨平台的项目管理工具。说是项目管理工具，是因为它支持项目的的构建、编译、打包和依赖管理等内容。（如果不理解上面说的构建、编译、打包是什么意思，也不用担心，后面的章节中将详细介绍）。说它夸平台是因为Maven是由Java语言开发的。所以在使用Maven的时候，必须安装JDK才可以使用。并且Maven也是Apache组织中的开源的项目。为什么要强调这一点呢，是因为如果一个项目是Apache的项目，大概率这样的开源项目流行的概率是比较大的。总之简单概括一下，Maven是一个跨平台的项目管理工具，它支持项目的构建、编译、打包和依赖管理等内容。



2、为什么需要 Maven
----------

介绍了这么多Maven的好处，那么我们在日常的开发中为什么需要Maven呢？在没有Maven之前我们如何开发呢。下面我们带着这个问题，看一下下面的例子，例子很简单，我们使用工具类将对象转成Json字符串并输出。并且先不使用Maven的情况下实现上面的内容呢。实现上面的方式的工具类比较多fastjson的方式实现。

我们首先创建一个普通的Java项目，具体操作如下：

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/A6855517-9206-44A9-96C0-7ACA248B520C.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/A6855517-9206-44A9-96C0-7ACA248B520C.png)

我们简单输出一句话，验证一下项目是否有问题。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/9736299C-8631-48B9-BF90-917A12E5CA0B.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/9736299C-8631-48B9-BF90-917A12E5CA0B.png)

执行上面的代码：

```java
Hello World Mazhe Maven!
```

下面我们创建一个Userinfo对象，然后输出对象的属性。具体操作如下：

*   Userinfo.java

```java
public class Userinfo {
    private String username;
    private String passworld;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassworld() {
        return passworld;
    }

    public void setPassworld(String passworld) {
        this.passworld = passworld;
    }

    @Override
    public String toString() {
        return "Userinfo{" +
            "username='" + username + '\'' +
            ", passworld='" + passworld + '\'' +
            '}';
    }
}
```

*   Mazhe.java

```java
public class Mazhe {
    public static void main(String[] args) {
        Userinfo userinfo = new Userinfo();
        userinfo.setUsername("mazhe");
        userinfo.setPassworld("maven");
        System.out.println(userinfo);
    }
}
```

输出：

```java
Userinfo{username='mazhe', password='maven'}
```

下面我们引入fastjson的依赖包，由于我们不使用Maven，所以我们使用下面的方式引用，具体操作如下：

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/D5866DF7-6C8F-498B-A74D-F1217EF6C995.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/D5866DF7-6C8F-498B-A74D-F1217EF6C995.png)

我们首先在项目的根目录中创建一个目录来存放我们引入外部的依赖包。当然我也可以不在项目的根目录中创建，在任何目录中引入都可以，只不过在项目中创建的好处是为了避免因目录改动，导致我们之前引入的依赖包找到不相应的目录，而导致引入的依赖包不可用。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/27A31C28-C2B3-4558-AD0C-6995483EBC44.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/27A31C28-C2B3-4558-AD0C-6995483EBC44.png)

然后我们将我们需要要依赖包拷贝到我们刚刚创建的目录中。如果我们这前配置过，那么这个依赖包就可以直接在项目中引入了，如果我们没有问题过，那么项目中还是使用不了我们这个依赖包的，因为它没有识别出这个目录。那以怎么判断项目是否识别同这个目录呢？很简单我们只需要看一下，这个目录中的依赖包，是否可以直接查看相应的源码即可。如果项目中显示了依赖包的目录结构，并且可以直接查看源码，那么则证明我们项目识别出了这个依赖包，可以直接在项目中引用了。反之亦然。

我们通过上图观察到，项目并没有显示出依赖包的目录结构，所以也就说明项目并没有识别出这个依赖包。下面我们通过下面的配置即可。配置的方式有很多，我们采用下面的方式实现。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/DFD938C6-12D9-49FB-9E5E-61DDB82909B0.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/DFD938C6-12D9-49FB-9E5E-61DDB82909B0.png)

我们在IDEA的File菜单中选择`Project Structure ...`

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/7588E1C4-317E-40AB-9362-44B492000D6C.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/7588E1C4-317E-40AB-9362-44B492000D6C.png)

然后选择`Libraires`

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/B5FBEF52-DF9F-49D4-9527-CFE69094536D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/B5FBEF52-DF9F-49D4-9527-CFE69094536D.png)

点击`+`选择`Java`然后选择我们之前项目中创建的目录即可。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/EF0E9CEF-F62A-4C8C-94F0-7FAB52D039D4.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/EF0E9CEF-F62A-4C8C-94F0-7FAB52D039D4.png)

这块就是我们上面所说的，我们实际上也可以不选择一个目录，而是直接选择一个依赖包，这样我们依然可以在项目中引入它。只不过选择目录好好处是，如果我有新和依赖包的要引入时，直接拷贝到这个目录中就可以直接引入了，而不需要在次配置了。这就是选择目录的好处。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/DDE2A597-BFEC-43CA-9ECE-B2291BF79F7F.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/DDE2A597-BFEC-43CA-9ECE-B2291BF79F7F.png)

我们在次查看项目中的依赖包，发现项目中可以识别出我们依赖包的里的目录结构了。下面我们修改一下代码来将我们之前说的把对象转成Json。

*   Mazhe.java

```java
import com.alibaba.fastjson.JSONObject;

public class Mazhe {
    public static void main(String[] args) {
        Userinfo userinfo = new Userinfo();
        userinfo.setUsername("mazhe");
        userinfo.setPassworld("maven");
        System.out.println(JSONObject.toJSONString(userinfo));
    }
}
```

*   输出

```java
/Users/md/IdeaProjects/md-mazhe-maven/src/Mazhe.java:8:27
java: 无法访问com.alibaba.fastjson2.util.Wrapper
  找不到com.alibaba.fastjson2.util.Wrapper的类文件
```

当我们以为大功告成的时候，发现上面的代码居然报错了。我们看上面的输出日志发现是缺少了`fastjson2`的依赖包。下面我们将这个依赖我也拷贝到项目中。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/BF2ED85C-FFB1-459A-B785-B15BF66A9610.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/BF2ED85C-FFB1-459A-B785-B15BF66A9610.png)

我们在次执行上面的代码，查看一下日志输出。

*   输出

```java
Exception in thread "main" java.lang.NoClassDefFoundError: com/alibaba/fastjson2/support/AwtRederModule
    at com.alibaba.fastjson.JSON.<clinit>(JSON.java:84)
    at Mazhe.main(Mazhe.java:8)
Caused by: java.lang.ClassNotFoundException: com.alibaba.fastjson2.support.AwtRederModule
    at java.net.URLClassLoader.findClass(URLClassLoader.java:387)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:418)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:355)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
    ... 2 more
```

我们发现日志还是报错，并且从日志中我们找到了`java.lang.ClassNotFoundException`异常的关键字，说明我们依赖包里还缺少依赖。下面我们将缺少的依赖在拷贝到项目中。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/770A4E90-DDCC-4B28-8574-CD431F5D9DDC.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/770A4E90-DDCC-4B28-8574-CD431F5D9DDC.png)

在次执行上面的代码，观察日志输出。

*   输出

```java
{"passworld":"maven","username":"mazhe"}
```

我们看日志输出发现这次项目执行成功了，我们成功的将对象转换成Json了。但这时可能有人会有些疑惑，上面的报错，为什么你能知道缺少的是哪个依赖包？这实际上也就是我们在没有使用Maven管理项目一个非常耗费时间的一个工作。因为这个是需要一点一点尝试出来的。尝试的方式，可以是直接看项目的源码，看一下这个源码中依赖的包是哪一个。或者简单一点将相同的依赖包全都拷贝到项目中，但这在实际的项目开发中是不推荐的。

除了这个问题外，还有的问题就是依赖包与依赖包冲突，因为现在依赖包的版本非常的多，通常都有一个稳定的依赖版本，如果依赖版本引入的不正确，也会导致相应的依赖包不可用。这也就是为什么我们在使用Maven的管理项目依赖的好处，它可以大在提高我们的工作效率。例如作者在书中所举的例子一样，例如我们要买一台电脑，我们现在方式相当于我们自己组装一台电脑，那么这样的工作是非常繁琐的，并且需要我们非常了解显卡、内存、主板等这方面的内容。除了这个入门比较难之外，还有一个问题是，可能会因为我们选择的配置有问题，导致不兼容最后导致电脑不可用。但这么做也有好处，就是可以让我们非常详细了解这方面的知识。那使用Maven的方式和这个方式而完全不同，相当于，我们购买了一台笔记本。它不需要我们了解上面的内容，我们只是需要选择好我们想要购买的牌子即可。这两种的效率可是截然不同的。这也就是使用Maven的好处。



3、Maven 的同类产品
-----------

下面我们了解一下，在没有Maven之前，那我们除了上面说的方式，还有哪些其它的项目管理工具可以提高我们开发效率吗？实际上也是有的，并且不只一个，下面我们分开介绍一下。

### 1、Make

Make几乎可以理解为最早的项目构建工具。它由Stuart Feldman于1977年在Bell实验室创建的。在使用Make时需要定义一个名为Makefile的脚本，Make通过该脚本来达到项目构建的目的。其它以后的项目构建工具也采取了同样的方式，例如Maven中的`pom.xml`文件。该脚本使用了Make自己定义的语法格式。语法中包括一部分规则（Rules），而规则中又包括目标（Target）、依赖（Prerequisite）和命令（Command）等信息。

下面是具体的Make的语法如下：

```bash
# 指定编译Java代码时使用的参数
JFLAGS = -g

# 指定Java编译器
JC = javac

# 指定生成Jar包时使用的参数
JARFLAGS = cvfm

# 指定生成的Jar包文件名
JARFILE = MyProgram.jar

# 定义一种依赖关系，指示如何从.java文件生成.class文件
.SUFFIXES: .java .class
.java.class:
        $(JC) $(JFLAGS) $*.java

# 定义要编译的Java源代码文件列表
CLASSES = \
        MyClass1.java \
        MyClass2.java \
        MyClass3.java

# 定义默认规则，指示执行`jar`规则
default: jar

# 定义`classes`规则，指示如何编译Java源代码
classes: $(CLASSES:.java=.class)

# 定义`jar`规则，指示如何生成Jar包
jar: classes
        jar $(JARFLAGS) $(JARFILE) Manifest.txt $(CLASSES:.java=.class)

# 定义`clean`规则，指示如何清理生成的class文件和Jar包文件
clean:
        $(RM) *.class $(JARFILE)
```

上面的内容就是Make的语法规则，由于此项目管理工具太早了，所以我们只简单了解一下即可。下面我们详细介绍一下上面具体的规则。

```bash
- default：默认规则，指定执行jar规则。

- classes：编译Java源代码生成class文件。

- jar：生成Jar包。

- clean：清理生成的class文件和Jar包文件。
```

要生成Jar包，只需在终端窗口中输入make jar命令即可。该命令会执行Makefile文件中定义的jar规则，生成Jar包。如果需要清理生成的class文件和Jar包文件，可以输入make clean命令。

* * *

### 2、Ant

在Make之后，Ant出现了。为什么会出现Ant呢？这是因为Make不支持跨平台，这对于使用Java来说是不友好的，并且Make的语法也比较难，通过编译失败都是因为多少空格导致的，但这很难提前发现问题。在Ant中使用了build.xml作为项目的构建脚本。这点和Maven比较像。相比Makefile的语法Ant的xml方式，则要方便很多。当然也有其它的构建工具，例如Gradle，它是采用Groovy编程语言来定义构建脚本的，相比xml来说，又方便了很多，所以Gradle也被移为Maven之后，下一代的构建工具。

下面我们介绍一下Ant构建脚本的具体配：

```xml
<?xml version="1.0"?>
<project name="MyProject" default="jar">
    <!-- 定义项目的属性 -->
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="build"/>
    <property name="dist.dir" value="dist"/>
    <property name="main.class" value="com.example.MyClass1"/>

    <!-- 创建目标目录 -->
    <target name="init">
        <mkdir dir="${build.dir}"/>
        <mkdir dir="${dist.dir}"/>
    </target>

    <!-- 编译Java源代码 -->
    <target name="compile" depends="init">
        <javac srcdir="${src.dir}" destdir="${build.dir}"/>
    </target>

    <!-- 生成Jar包 -->
    <target name="jar" depends="compile">
        <jar destfile="${dist.dir}/${ant.project.name}.jar" basedir="${build.dir}">
            <manifest>
                <attribute name="Main-Class" value="${main.class}"/>
            </manifest>
        </jar>
    </target>

    <!-- 清理生成的文件 -->
    <target name="clean">
        <delete dir="${build.dir}"/>
        <delete dir="${dist.dir}"/>
    </target>
</project>
```

我们通过上面的配置，第一次直观的感受就是，Ant的构建配置确实比Makefile的语法相对来说比较简洁。下面我们同样的详细介绍一下Ant的配置规则：

```bash
> init：创建编译和生成Jar包所需的目录。

> compile：编译Java源代码。

> jar：生成Jar包。

> clean：清理生成的文件。
```

要生成Jar包，只需在终端窗口中输入ant jar命令即可。该命令会执行build.xml文件中定义的jar任务，生成Jar包。如果需要清理生成的文件，可以输入ant clean命令。

注意：在执行该build.xml文件之前，需要将文件中的属性值替换为您的项目属性。例如，src.dir属性指向包含Java源代码的目录，main.class属性指向包含main方法的类的类名，这将是您的程序的入口点。

通过上面的介绍我们发了一个问题，那就是不管是Make还是Ant虽然都提供了自动化管理项目的功能，但都是正对项目打包和编译的功能，并没有提供项目的依赖管理，也就是我们依赖包的管理，实际上项目开发中，这项的工作实际是最繁琐的。所以Maven的出现改变了上述的工作方式，除了上面的内容外，Maven还支持依赖管理、项目管理等内容，并且和Ant相比，Maven定义了很多额外的约定了，只要遵循这些约定，就可以很方便的管理项目。但Ant而不是，所以它也是用xml做为构建管理的，但并没有过多的约定。所以常常需要人为的约定，否则，在不同的环境中的Ant构建脚本，在换到其它环境中，则会因为目录或者其它的原因导致编译失败。



4、Maven 有哪些缺点
----------

在开始介绍Maven之前，我们先介绍一下Maven的缺点，任何技术都有它的适用之处，只有我们采用是了合适的技术，才是这项技术最佳的使用方法。

*   Maven过于复杂

说Maven复杂是相比来说的，因为它几乎帮我们处理了项目管理的全部工作，例如清理、编译、测试、打包、发布等等。但要想实现上述说的实现，难度是可想面知的，所以可以说Maven的入门还是比较难的。

因为Maven是用来管理项目的，清理、编译、测试、打包、发布，以及一些自定义的过程都是一件非常复杂的事，所以可以说的Maven入门还是比较难的。

*   Maven的仓库十分混乱

我们知道通过Maven是可以管理我们的依赖包的，因为Maven支持下载的功能，但有时会因为一些原因导致，我们下载失败，这时我们只能手动的去下载。并且Maven为了我们方便管理依赖包，提供了中央仓库的概念，但Maven的中央仓库并不完美，你们常常会发现相同的依赖包在两个不同的路径中。

*   Maven官方文档非常凌乱

我们知道作为一个开源的项目工具，官方的文档是很重要的，它是我们第一时间了解一项开源技术的唯一途经。其中有一些开源项目的管方文档确实写的比较好，例如Spring的相关文档。当有一些开源项目文档写的则比较差，其中Maven就是其中之一。Maven的官方网站的文档非常凌乱，各种插件的文档寻找非常吃力。通过官方文档，很难入门。



5、Maven 的安装
--------

介绍了这么多，我们终于来到了这一步，也就是安装，任何技术都是从安装开始的。那么Maven也是一样的，不同的地方在于，Maven在装之前需要有些额外的配置。

### 1、Windows 安装 Maven

*   安装前环境的准备

由于Maven是使用Java语言开发的，所以在使用Maven之前需要提前把Java的环境安装好，也就是需要检查是否成功的安装了JDK。具体执行的命令如下：

```bash
java -version
```

*   输出

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/CA6CBF7B-ED7B-4855-A312-FCD302AD611F.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/CA6CBF7B-ED7B-4855-A312-FCD302AD611F.png)

这样则表示我们的环境变量配置成功了。

如果本地成功的安装JDK，则会直接显示Java的版本信息。如果想查询本地JDK的安装路径可以执行下面的命令查询：

```bash
echo $JAVA_HOME
```

*   下载Maven

下面我介绍一下如何下载Maven。和其它开源软件一下，软件一定是要通过官方网站下载的。下面是Maven官方网站的地址：https://maven.apache.org

当我们访问上述官方网站时，首页直接就会看到Maven的下载地址。我们直接点击Download链接即可：

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/0338B90E-307A-4C30-92EE-CEAD41E8A38A.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/0338B90E-307A-4C30-92EE-CEAD41E8A38A.png)

当我们点击后就会看到有多种类型的安装包供我们下载，也就是如下图所示：

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/27893E5D-61E1-4F1C-99B3-D7A82CC35AAB.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/27893E5D-61E1-4F1C-99B3-D7A82CC35AAB.png)

它们具体的区别是，提供了不够平台的下载文件。如果是Linux或者是MacOS推荐使用`tar.gz`的下载文件，如果是Windows平台的推荐使用`zip`后缀的下载文件。除此之外，Maven官方还提供了，上述不同平台的md5校验和（checksum）文件和asc数字签名文件，目的是可以用来检验Maven下载包的正确性和安全性。如果你想对Maven的源码感兴趣可以下载`src`的版本文件，自己构建Maven。那由于我是MacOS系统，所以我下载tar包就可以了。

*   安装Maven

当我们下载完对应的平台的版本后，我们先直接解压然。但解压后我们还不能直接用Maven。我们需要配置Maven的环境变量之后在使用。由于不同平台的配置方式有所不同，我们主要介绍两种平台的配置。我们首先介绍一下如何在Windows平台中配置Maven的环境变量：

*   我们首先需要在系统的环境变量中新创建一个`M2_HOME`的变量，路径为上面Maven解压后的全路径，注意结尾不要加任何符号。
*   接着在系统中环境变量中查找到`Path`变量，然后在改变量的后面追加下面的配置：

```bash
%M2_HOME%\bin;
```

注意后面的分号。如果配置完成后，可以在命令窗口中查询一下配置的`M2_HOME`的变量是否配置成功。

```bash
echo %M2_HOME%
```

如果该命令输入的结果是我们Maven下载后解压的路径，则表示我们配置成功了。接着我们可以输入下面的命令来验证Maven是否安装成功

```bash
mvn-v
```
如果此命令成功的显示了Maven的安装版本则表示，我们Maven配置的成功了。
***

### 2、Mac/Linux 安装 Maven

下面我们看了解一下如何在Linux或者MacOS系统中配置Maven环境变量。首先我们在命令窗口中执行下面的命令：

```bash
sudo vim ~/.bash_profile
```

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/B5A954A8-3D85-4AF3-8DC6-01B002BFD6E5.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/B5A954A8-3D85-4AF3-8DC6-01B002BFD6E5.png)

当输入上述命令后，系统会提未你输入机器的登陆密码，当我们输入成功后，就会进入到文件当中，然后将下面的配置添加到`bash_profile`文件中。

```bash
export MAVEN_HOME=/Users/jilinwula/Downloads/apache-maven-3.6.3
export PATH=$PATH:$MAVEN_HOME/bin
```

注意，上面说的`MAVEN_HOME`对应的是系统的Maven解压后的路径。当设置完成后，输入`:wq`命令保存我们刚刚打开的文件。然后我们还需要执行下面的命令，让上述的配置立即生效。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/9B2A7FE2-1097-4591-8274-B2D31C69181A.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/9B2A7FE2-1097-4591-8274-B2D31C69181A.png)

```bash
source ~/.bash_profile
```

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/FF14151D-9EFC-42C8-B1DF-7254C39AECA1.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/FF14151D-9EFC-42C8-B1DF-7254C39AECA1.png)

由于我们已经输入完机器的登陆密码了，所以这次我们不需要在输入机器的密码了。

接着，我们输入`mvn-v`命令，看检查是否成功的返回的Maven的版本信息。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/43682BF5-DAC4-4D8E-8061-A58A699847EB.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/43682BF5-DAC4-4D8E-8061-A58A699847EB.png)

看上图中显示了我们指定的Maven版本，并且还显示出了我们Maven配置的路径，这表示我们Maven配置成功了，可以直接使用了。



6、Maven 的目录结构
----------

下面我们介绍一下Maven安装包的目录结构，通过目录结构可以让我们更加理解和掌握Maven。在Maven中主要包括下面的目录和文件下面我们详细介绍一下：

*   bin：很多的开源软件中都包括这具目录也就是目录`bin`目录。在这个目录中主要包括了Maven运行的所有脚本文件，其中没有带`.cmd`后缀的命令为Linux平台的脚本文件，带`.cmd`后缀的为Windows平台的脚本文件。在此目录中还提供了我们上述中我们介绍过的`mvn`命令脚本。除此之外Maven还提供了`mvnDebug`命令。它俩的区别是后者在运行时开启debug模式，可以允许我们调试Maven。下图为`bin`目录的相关截图：

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/81A60C12-96A3-48B9-87DA-368F67DED56C.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/81A60C12-96A3-48B9-87DA-368F67DED56C.png)

*   boot：boot`目录下有一个jar文件。该文件的作用是Maven自己实现的类加载器。和默认的Java类加载器相比，它提供了丰富的语法和配置，Maven使用此类加载器加载自己的类库。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/007AEAB9-6D77-4E7D-A4F1-2CBAC88C2568.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/007AEAB9-6D77-4E7D-A4F1-2CBAC88C2568.png)

*   conf：conf`目录中包含了一个非常重要的文件`settings.xml`文件。在后续的内容中我们会详细`settings.xml`文件。通过该文件我们可以全局地定制Maven的行为。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/1902DAE0-DD3A-467B-AE07-2F2EFA777F4D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/1902DAE0-DD3A-467B-AE07-2F2EFA777F4D.png)

*   lib：lib`目录中包括了Maven运行时依赖的所有的Java类库。这也间接的说明了Maven是使用Jave语言开发的。

[![Maven实战-第一篇（Maven的基本信息）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/A8DF0678-4A10-4CAA-B197-A7C5687F95EE.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%80%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BF%A1%E6%81%AF%EF%BC%89.resources/A8DF0678-4A10-4CAA-B197-A7C5687F95EE.png)

  

# 二、Maven 入门

通过前两篇的介绍我们基本上对Maven有了一个初入的了解，在一这一篇中，我们详细介绍一下如何在项目中使用Maven，也就是Maven的入门。

1、什么是POM
------

在Maven实战-第一篇（Maven的基本信息）中我们介绍过Make和Ant。它们都是通过Makefile或者build.xml来管理项目的。那么对于Maven也是一样的，Maven也需要一个脚本文件来执行项目的构建和管理。在Maven中采用了和Ant一样的管理方式，也就是xml文件的方式进行项目的管理。只不过在语法中有些区别。并且在Maven中命令规则为pom.xml，而不是Ant中的build.xml。

下面我们按照软件开发的惯例，创建一个Helloworld空项目。具体操作如图所示：

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C1515BC4-6512-416C-9505-F9FE04AE3A15.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C1515BC4-6512-416C-9505-F9FE04AE3A15.png)

然后我们在这个项目的根目录中创建一个`pom.xml`文件。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C466D4E9-83F2-4C7F-B608-BA36A9661523.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C466D4E9-83F2-4C7F-B608-BA36A9661523.png)

我们IDEA中提示了我们`pom.xml`文件报错了，这是因为`pom.xml`文件中有Maven约定的语法规则，我们现在的`pom.xml`文件是空的所以报错了。`pom.xml`文件中可以配置的语法非常的多，我们下面介绍一下，`pom.xml`文件中必须要设置的内容。

*   xml：指定了xml文件的版本和编码。
*   project：该标签是pom.xml项目的的根标签。
*   modelVersion：指定POM模型的版本。对于Maven3来说，必须是4.0.0。
*   groupId：指定我们创建的这个项目是属于哪个组的，通常为公司组织的域名倒写，这样保证不会重复。
*   artifactId：定义了当前Maven项目在组中的唯一标识，也就是同组唯一。
*   version：指定当前项目的版本。在Maven中有两个类型的版本，一个是正常的版本需要，还有一个为SNAPSHOT快照版本。快照版本为不稳定版本。
*   name：项目的名称。

我们先简单有一个概念，在后续的章节中，我们会详细介绍上面的内容。下面我们看一下例子，来更直观的感受到，在Maven中怎么配置上面介绍的信息。

```
<?xml version="1.0" encoding="utf-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.ma-zhe</groupId>
    <artifactId>mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven实战</name>
</project>
```

当我们配置完上述内容时，我们发现`pom.xml`中已经不显示报错信息了，这就表明我们配置的没有任何问题了，如果还是报错，而表示我们有的标签配置的还是有误。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/373798B8-7B45-41B9-9E78-F648A59E8186.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/373798B8-7B45-41B9-9E78-F648A59E8186.png)

2、代码编写
----

在我们完成上面的配置后，我们可以开始我们的代码编写了。但使用Maven编写代码时，和我们普通的Jave项目有一些区别，它也有相应的默认的约定。我们知道在普通的项目中所有Jave源码代码都写在项目的`src`目录中。在Maven中同样的对项目的源代码目录进行了约定，只不过在Maven中对普通项目的源代码称之为主代码。还约定了测试代码的目录。主代码会最终打包到构建Jar包中的代码。而测试代码只在测试运行时用到，不会被打包。在Maven中使用下面两个目录结构进行区分。

*   主代码

```
src/main/java
```

上述目录为Maven默认约束好的主代码的目录，只要按照上面的路径设置，Maven会自动编译上述目录中的代码并打包到Jar构建包中。

*   测试代码

```
src/test/java
```

上述目录为Maven默认约束好的测试代码的目录，Maven只会在测试运行时编译上述目录中的代码并且不会打包到Jar构建包中。

有一点我们要注意，上面所说的主代码或者测试代码的目录都是Maven默认约定的目录结构，我们可以通过`pom.xml`中提供的语法更改默认的目录结构，但在实际的项目中我们并不会这么做。

3、Maven命令
-------

按照上面介绍的我们先在项目中创建一个`src/main/java`目录。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/D8335C86-B392-414F-ADF6-3B3AEA05CD3B.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/D8335C86-B392-414F-ADF6-3B3AEA05CD3B.png)

然后我们在创建一个新的Java类，来测试一下，代码能否执行成功。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/64658984-780C-4E00-8F25-55A37876B4D0.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/64658984-780C-4E00-8F25-55A37876B4D0.png)

当我们要创建Java类，发现IDEA中已经没有我们想要创建Java类的那个菜单了。这是因为IDEA认为`src`才是源代码的目录。所以没有显示可以创建Java的菜单。要想可以显示出创建Java的菜单也非常的简单，也就是告诉IDEA此时`src/main/java`目录就是项目的源代码目录即可。具体配置如下：

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/A42C197B-0918-45E4-B789-738C6849287E.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/A42C197B-0918-45E4-B789-738C6849287E.png)

这样IDEA就会识别出我们现在选择的这个目录是源码目录，就允许我们创建Java文件。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/03326708-A604-4FD3-8BC8-45A60D523546.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/03326708-A604-4FD3-8BC8-45A60D523546.png)

我们首先创建一个简单的Java类，来验证一下，项目是否能成功的执行的我们的代码。

*   Mazhe

```
public class Mazhe {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven");
    }
}

```

*   输出

```
Hello World Mazhe Maven
```

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/65DA8AE5-2603-4E03-A48A-05FAF80277B4.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/65DA8AE5-2603-4E03-A48A-05FAF80277B4.png)

下面我们还是改造一下代码，按照第一篇中例子一样，我们创建一个对象，然后将对象转换成JSON字符串。

*   Userinfo

```
public class Userinfo {
    private String username;
    private String passworld;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassworld() {
        return passworld;
    }

    public void setPassworld(String passworld) {
        this.passworld = passworld;
    }

    @Override
    public String toString() {
        return "Userinfo{" +
                "username='" + username + '\'' +
                ", passworld='" + passworld + '\'' +
                '}';
    }
}
```

*   Mazhe

```
public class Mazhe {
    public static void main(String[] args) {
        Userinfo userinfo = new Userinfo();
        userinfo.setUsername("mazhe");
        userinfo.setPassworld("maven");
        System.out.println(userinfo);
    }
}
```

*   输出

```
Userinfo{username='mazhe', passworld='maven'}
```

下面我们按照Maven的方式引入依赖包，这样就不用向之前那样，我们需要在项目中手动的创建依赖包的目录和依赖包的拷贝工作了。那我们上哪去查询依赖包呢？这个不用担心，Maven为了方便我们查询，可以通过在线的网站上通过关键字搜索查询。下面是依赖包的网站的地址：

[https://mvnrepository.com](https://mvnrepository.com/)

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/05248A7B-DA39-434B-AD44-86001F9320F7.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/05248A7B-DA39-434B-AD44-86001F9320F7.png)

我们可以在上面搜索中直接搜索我们想要的依赖包。按照需求，我们需要将对象转换成json。这样的工具类很多，我们使用fastjson的方式实现。所以我们只需要在上面的网站上搜索即可。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/77ACBAFC-E67E-4FFF-9141-5B371C12BA7D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/77ACBAFC-E67E-4FFF-9141-5B371C12BA7D.png)

然后我们点击上面的链接，就会显示出这个依赖包所有可以使用的版本。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/55CDF295-25E7-45CC-8C73-918B55B1B110.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/55CDF295-25E7-45CC-8C73-918B55B1B110.png)

我们随便选择一个版本即可。在实际的软件开发中，我们需要选择相对比较稳定的版本，这样的版本问题比较少。那如何选择文档的版本呢。我们可以通过上图中的`Usages`参数来观察，这个参数越多，则表示使用的人越多。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/86B6FFB0-379F-4D82-8F97-6334D93097EB.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/86B6FFB0-379F-4D82-8F97-6334D93097EB.png)

当我们选择一个版本后，点击链接就会显示出我们使用这个依赖包具体的Maven配置信息了。并且，通过上图中我们可以观察到，除了Maven外，它还支持其它的项目管理工具，例如Gradle。我们点击这个标签就会显示Gradle的配置信息。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/FB5A27B7-83CD-4383-B1B2-BDCE26B77E2D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/FB5A27B7-83CD-4383-B1B2-BDCE26B77E2D.png)

下面我们介绍一下，如何使用上面的Maven配置。我们拷贝一下fastjson的Maven配置信息，然后粘贴到项目的pom.xml文件中。之前我们介绍过，pom.xml是项目的构建文件，所以所有和Maven相关的配置都可以在pom.xml文件中配置。

```
<?xml version="1.0" encoding="utf-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.ma-zhe</groupId>
    <artifactId>mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven实战</name>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>2.0.35</version>
    </dependency>

</project>
```

当我们将上面的配置粘贴到pom.xml后，发现文件居然报错了。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/AB3B419A-22F7-4892-AE02-30394787BBBA.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/AB3B419A-22F7-4892-AE02-30394787BBBA.png)

这是因为上面的配置不符合Maven的语法规则。具体的原因是因为Maven中需要将所有的依赖配置都放到`<dependencies></dependencies>`标签中。也就是如下配置才可以：

```
<?xml version="1.0" encoding="utf-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.ma-zhe</groupId>
    <artifactId>mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven实战</name>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.35</version>
        </dependency>
    </dependencies>

</project>
```

当我们这样配置完后成后，发现依赖包的版本显示红色了，这是因为Maven还没有查找到这个依赖包。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/7D28816D-206E-4B58-BAC8-8CB01C38E165.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/7D28816D-206E-4B58-BAC8-8CB01C38E165.png)

这是因为IDEA暂时还没有将我们这个项目识别出Maven项目。所以需要选中pom.xml文件，然后右键选择：`Add as Maven Project`

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/449F3489-C5C9-4407-9385-A1ADA99D3EE5.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/449F3489-C5C9-4407-9385-A1ADA99D3EE5.png)

当我们这样操作完成后，我们发现我们的版本已经不会报错了。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C42FAB5A-D874-4FA8-9B77-7D7E958A336D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C42FAB5A-D874-4FA8-9B77-7D7E958A336D.png)

并且这时我们看项目的依赖包，发现Maven已经自动将fastjson依赖的所有包都下载成功了。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/E424B65D-84CE-4D95-AD81-3BEEFA9ED2EA.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/E424B65D-84CE-4D95-AD81-3BEEFA9ED2EA.png)

下面我们在项目中使用一下，看看用Maven引入依赖包的方式和我们自己引用有没有什么区别。

*   Mazhe

```
import com.alibaba.fastjson.JSONObject;

public class Mazhe {
    public static void main(String[] args) {
        Userinfo userinfo = new Userinfo();
        userinfo.setUsername("mazhe");
        userinfo.setPassworld("maven");
        System.out.println(JSONObject.toJSONString(userinfo));
    }
}
```

*   输出

```
{"passworld":"maven","username":"mazhe"}
```

我们看日志输出，成功的将对象转换成json了，并且在我们使用Maven和我们自己引入依赖的方式在编写代码上没有任何区别。但在使用上，我们可以很直观的感受到使用Maven引入依赖包时，为我们带来的便捷。

* * *

*   编译

下面我们介绍一下，Maven提供的其它的功能，我们知道在项目中我们比较常见的操作就是代码测试、编译、打包、运行等操作。上面我们介绍了如何使用Maven引入我们需要的依赖包。下面我们介绍一下，如何使用Maven编译项目。

*   Mazhe

```
import com.alibaba.fastjson.JSONObject;

public class Mazhe {
    public static void main(String[] args) {
        Userinfo userinfo = new Userinfo();
        userinfo.setUsername("mazhe");
        userinfo.setPassworld("maven");
        System.out.println(JSONObject.toJSONString(userinfo));
    }
}

```

在Maven中编译项目很简单，特别是使用是IDEA这样的开发工具，我们只需要简单保存一下就可以完成代码的编译了。所以为了加深我们的理解，今天我们使用Maven命令的方式编译项目，这也是我们介绍Maven的第二个命令（第一个Maven命令为：）：

```
mvn clean compile
```

当我们在项目的根目录中输入完上面的命令后，Maven就会自动的执行项目的编译。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/05B93CD3-BE85-4DCA-965C-DFCB1D6A20EB.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/05B93CD3-BE85-4DCA-965C-DFCB1D6A20EB.png)

当我们输入完上面的Maven命令后，Maven就会开始工作了，并且通过上图中，我们也可以发现Maven执行时输出的日志信息。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/1FA00DD3-BAAA-44F2-9C13-53B0FE741FCF.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/1FA00DD3-BAAA-44F2-9C13-53B0FE741FCF.png)

当项目编译成功后就会在项目的根目录中创建target目录，然后将主代码的中编译好的class文件放到target中的classes目录下。也就上图所示。

*   测试

下面我们介绍一下如何使用Maven进行代码的测试。在Maven中如果要想执行测试代码，需要引入额外的依赖包。因为我们上面的内容中已经介绍了如何在项目中引入依赖包，所以在这里我们就不做过说的介绍了。在Maven中要执行测试，需要我们使用junit依赖。引入junit依赖的配置信息如下：

```
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
    </dependency>
</dependencies>
```

同样的，我们还是需要将上面的配置信息配置到pom.xml文件中。下面我们开始编写测试类的代码。这里有一点要我们注意，按照Maven的约定，测试类的代码是需要放在测试类代码`src/test/java`目录中的，而不是我们刚刚写的主代码的目录中，这一点要特别的注意。

同样的我们在项目中创建一个`src/test/java`目录。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/BAD137B4-CCCD-4C42-A395-6BF1EE8804F9.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/BAD137B4-CCCD-4C42-A395-6BF1EE8804F9.png)

然后我们继续创建一个测试类，但当我们右键创建时，发现了类似的问题，就是没有提供创建Java类的菜单，这个问题和上面遇到的问题一样，是因为IDEA并没有识别出我们这个测试类的目录，同样的我们使用下面的方式配置一下即可。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/D3F8DD78-7169-4303-AFAF-EBCB1D2D5A66.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/D3F8DD78-7169-4303-AFAF-EBCB1D2D5A66.png)

这里有一点选择和上面的不同，就是因为这次我们创建的是测试类的目录，所以我们要选择`Test Rources Root`，而不是之前我们选择的`Sources Root`。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/55F7EF5E-62BB-43C7-9FFA-3CCB309360BC.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/55F7EF5E-62BB-43C7-9FFA-3CCB309360BC.png)

当我们设置完成后，在右键创建时，我们发现这时IDEA可以允许我们创建Java类了。下面我们创建一个测试类，具体的代码如下：

```
import org.junit.Test;

public class MazheTest {
    @Test
    public void maven(){
        System.out.println("Hello world Mazhe Maven Test");
    }
}
```

我们简单说明一下上面的代码。因为这里面和我们平常写的代码还是有一些不一样的。例如我们之前要写可以执行的代码时，需要写`public static void main(String[] args)` 方法才可以执行。而当我们使用junit进行测试时代码时，对可以执行的方法名称并没有要求，可以随便定义符合Java规范的方法名，然后在对应的方法上添加`@Test`注解即可，这样这个方法就可以使用Maven进行测试了，也可以我们通过右键的方式直接运行。下面我们执行一下：

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/3AB8FBE8-0954-4B38-A988-8EC748B49EC6.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/3AB8FBE8-0954-4B38-A988-8EC748B49EC6.png)

我们看日志的输入成功的输出了我们代码中的结果。下面我们还是使用Maven命令的方式来执行一下我们测试的代码，要如何使用呢?很简单，我们还是通过命令的方式即可。在Maven中执行测试的命令为：

```
mvn clean test
```

我们在项目的根目录中执行一下上面的命令，来看一下输出的结果。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/617FA9C1-1BCB-4363-A1E4-851AD6F21972.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/617FA9C1-1BCB-4363-A1E4-851AD6F21972.png)

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/29656DF0-82B8-47E5-94C0-5376C8398469.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/29656DF0-82B8-47E5-94C0-5376C8398469.png)

我们看日志输出，不但成功的输出了我们测试代码中的内容，还输出了我们本次运行了多少测试，运行了多少和失败了多少等统计的信息。这也就是使用Maven非常方便的体现。下面我们将上面的代码手动抛出一个异常，然后在执行一下测试用例在看一下，这次Maven会输入什么样的信息。

*   MazheTest

```
import org.junit.Test;

public class MazheTest {
    @Test
    public void maven() {
        System.out.println("Hello world Mazhe Maven Test");
        System.out.println(1 / 0);
    }
}

```

我们手动输出了一个除以0的操作，所以上面的代码会抛出异常，我们看这时Maven的日志输出，会不会成功的输入异常上面的内容。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/619FA239-34CD-4819-8153-02C096D7516B.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/619FA239-34CD-4819-8153-02C096D7516B.png)

我们看异常上面的代码也成功的输出了，但是是执行下面的代码报错了。并且Maven已经将错误的信息输出。下面我们在看一下Maven中测试用例的统计信息，我们可以看到显示运行了一个，错误的一个。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/56F261BC-A4E9-4E48-98C6-26645D037B12.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/56F261BC-A4E9-4E48-98C6-26645D037B12.png)

* * *

下面我们详细介绍一下pom.xml中配置的标签：

*   dependencies：声明项目的依赖的父标签，也就是上面介绍的，所有依赖包的配置必须在这个配置标签下面才可以使用。
    
*   dependency：声明项目的依赖的子标签，也就是我们上面说的依赖的配置标签。
    
*   groupId：依赖的基本坐标，通常为组织的域名倒写。目的就是唯一不能重复。
    
*   artifactId：依赖的基本坐标，通常为该项目的唯一标识。不同组织下可以重复，相同项目中不可以重复。
    
*   version：依赖的基本坐标，项目的不同版本号。不同版本中依赖包的内容，可能会有很大的不同，所以我们在通过Maven使用依赖包时，版本要特别的注意，甚至不同依赖包版本与版本不同，可能会导致项目冲突的问题。
    
*   打包
    

下面我们介绍一下如何在Maven中执行项目的打包。在Maven中执行打包也非常的方便，只要执行下面的命令即可，Maven就会自动在target目录下生成相应的jar包。因为jar包是Maven默认的打包类型。所以我们不需要添加额外的配置，当然我们也可以指定其它的打包类型。在后面的章节中我在做过多的介绍。下面是Maven打包的命令：

```
mvn clean package
```

当我们执行完上面的命令后，Maven就会将打包完的jar包放到target目录下。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/4033CC58-F9D0-413E-A0F3-50B9BEA36E01.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/4033CC58-F9D0-413E-A0F3-50B9BEA36E01.png)

我们通过上面的Maven日志输出可以很方便的看到，这个jar包生成的目录地址。我们直接进入到target目录中，也可以直接看到我们刚刚打包完的jar包。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/FF4F3B0D-3935-459B-859E-1B45D3670697.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/FF4F3B0D-3935-459B-859E-1B45D3670697.png)

Maven默认的打包命名规则为当前项目的名称。当然我们也可以通过Maven中提供的`finalName`来自定义该打包文件的名称。具体配置如下：

*   pom.xml

```
<?xml version="1.0" encoding="utf-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.ma-zhe</groupId>
    <artifactId>mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven实战</name>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.35</version>
        </dependency><dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
    </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>

    </dependencies>

    <build>
        <finalName>Mazhe-Maven-1.0</finalName>
    </build>

</project>
```

然后我们继续执行`mvn clean package`命令后在看一下打包后的jar包是否为我们配置中指定的名称。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/57AF4533-3183-46E7-A0DB-25849E7C20F7.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/57AF4533-3183-46E7-A0DB-25849E7C20F7.png)

我们看上图中的结果，我们成功的指定了我们打包的名称了，这也就是`finalName`标签的作用。

*   安装

下面我们继续介绍另外的Maven命令，也就是安装命令。这里所说的安装可以理解为将打包好的jar安装到你的电脑中。这样当其它的项目相要引入这个依赖包时，只需要在对应的项目的pom.xml中配置即可。在Maven中可以使用下面命令进行安装：

```
mvn clean install 
```

当我们执行完上面的命令后，Maven依然会在项目的target中生成相应的jar包。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/BA8ED8F9-B760-4E5A-B4EF-5889052941A8.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/BA8ED8F9-B760-4E5A-B4EF-5889052941A8.png)

除此之外Maven还会将这个jar包生成到本地的仓库中，也就是上图中`Installing`输出的目录信息。由于Maven中涉及仓库的内容，我们会在后续的内容中详细的介绍，所以我们暂时可以先理解当我们执行完上面的命令后，Maven就会将这个jar包安装到我们的Maven指定的目录中，并且这个目录可以允许其它项目通过依赖配置的方式引入我们这个依赖包。

*   运行

下面我们介绍一下，如果在Maven中直接运行项目中的main方法，如果要实现上面的需求，需要我们添加额外的配置才能实现。在Maven中可以通过插件的方式完成上述的内容。插件的内容我们后续的内容中在详细的介绍，我们先简单了解如何使用。具体配置如下：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.4.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer
                                implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>Main</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

上面有一点我们要特别注意就是`mainClass`参数，我们要指定的是Mavn方法所在类的名称。并且不需要写类的后缀。当我们在pom.xml中配置了上面的信息后，然后执行打包的命令先生成新的打包文件。然后我们进入target目录中执行以下命令运行我的Jar包。

```
java -jar mazhe-maven-1.0.jar 
```

当我们输入上述命令时，就会发现成功输出了我们Main方法的内容了。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/6A46D90F-D484-4F07-9ED3-D364E8089F54.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/6A46D90F-D484-4F07-9ED3-D364E8089F54.png)

这时可能有人会怀疑，如果我们不配置上面的信息，直接打包，然后直接使用上面的命令运行我们Jar包，会显示什么结果呢？

下面我们尝试一下，将上面pom.xml中配置的build配置注释掉，然后测试一下。

*   pom.xml

```
<?xml version="1.0" encoding="utf-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.ma-zhe</groupId>
    <artifactId>mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven实战</name>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.35</version>
        </dependency>
    </dependencies>

   <!-- <build>
        <finalName>mazhe-maven-1.0</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.4.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>Mazhe</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>-->

</project>
```

下面我们执行打包命令，然后直接使用上面的命令运行我们新打包的Jar包。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/5E565436-0A82-4F82-97D8-F4674EC2DCF1.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/5E565436-0A82-4F82-97D8-F4674EC2DCF1.png)

我们看上图所示，Jar在运行时抛出了异常，上面内容指的是没有指定Main方法的类，所以Java不知道要运行哪一个。这也就间接说明了，我们上面配置的信息是没有任何问题的。

*   总结

下面我们在简单总结一下我们上面所介绍的Maven命令。上述命令就是Maven中比较常见的命令：

```
mvn clean compile // 编译项目
mvn clean test // 执行测试用例
mvn clean package // 项目打包
mvn clean install // 安装项目到本地
```

我们在执行Maven上述的命令时，如果仔细观察就会发现，上述的命令与命令之间是有依赖关系的。也就是在执行`mvn clean test`命令之前是会先执行`mvn clean compile`命令的，同样的在执行`mvn clean package`命令之前是会先执行`mvn clean test`命令的，而类似地，`mvn clean install`命令之前会执行`mvn clean package`命令。

那么Maven为什么要设置这样的规则呢，我们后续的内容中在详细介绍这方面的内容。

4、Maven如何解决重复造轮子问题
----------------

通过上面我们的介绍，我们知道按照上面步骤，知道了如果在项目中使用Maven。虽然比我们自行引入Java时，方便了许多，但还是有一些比较繁琐的问题。例如上面中介绍的那样，我们要手动的创建`pom.xml`。还要按照相应Maven的规则创建`src/main/java`和`src/test/java`等目录。如果只创建一次还好，如果有多个项目都要创建的话，上面的步骤就比较麻烦了。这也就体现不出来Maven便利之处处了。实际上Maven已经考虑到上面的问题了。于是提供了`mvn archetype:generate`命令来允许我们使用命令的方式初始化Maven项目的骨架。

下面我们测试一下，使用`mvn archetype:generate`。生成Maven项目骨架时，都会自动给我们生成哪些内容。我们首先选择一个空项目，然后执行上面的命令。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/08D6DF07-BDD7-44A9-AC3E-B0D73B66479F.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/08D6DF07-BDD7-44A9-AC3E-B0D73B66479F.png)

输入`mvn archetype:generate`命令。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/CCC03F5D-AB95-460C-BE1C-4A2D35F14071.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/CCC03F5D-AB95-460C-BE1C-4A2D35F14071.png)

当我们输入完上面的命令后，Maven就会自动运行创建我们想要的项目骨架。但运行中时，需要我们做出人为的选择。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/D7B57C39-4868-4F4B-A6F4-1C04162A44E0.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/D7B57C39-4868-4F4B-A6F4-1C04162A44E0.png)

上面提示的2065，指的就是Maven会安装下面的方式创建项目骨架。那为什么Maven提供的了这么多选项呢？这是因为不同的项目，需要生成的项目骨架是不一样的，例如Java Web的项目的和普通Java项目就是不一样的，所以Maven为了兼容这种情况，提供了很多的选择，我们在使用`mvn archetype:generate`创建项目时，通常按照默认的方式创建即可，除非你有特殊的创建需求，在选择指定的创建规则。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/1F3CCBEB-C453-46A9-AD91-5CCC3C12378C.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/1F3CCBEB-C453-46A9-AD91-5CCC3C12378C.png)

我们回车后，让Maven继续创建，也就是默认选择了2065创建。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/2D5A3ED8-0B24-4F12-9F4D-F9A5166842DE.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/2D5A3ED8-0B24-4F12-9F4D-F9A5166842DE.png)

当我们选择后，会提示让我们选择相应的版本，我们通常选择比较稳定的版本即可，也就是不待任何修饰的版本，我们选择最后一个，也就是默认的选项8。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/EB9A60D0-9300-4C52-B2EE-DE605C2F81A7.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/EB9A60D0-9300-4C52-B2EE-DE605C2F81A7.png)

继续后，Maven又提示了，这回没有选择项了，需要我们手动的填入了，也就是`groupId`参数，这个参数上面的内容介绍过，项目的基本坐标，不要重复，我们随便写`cn-mazhe.maven`然后继续。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/FAC53E4D-26DC-4652-A2EC-ABC6FE41AF74.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/FAC53E4D-26DC-4652-A2EC-ABC6FE41AF74.png)

Maven又让我们写`artifactId`参数，这个参数我们也介绍过，项目的唯一标识，我们写`mazhe-maven-quickstart`然后继续。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/7D2CEF14-2C89-4520-A5BD-00EB4AEE1093.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/7D2CEF14-2C89-4520-A5BD-00EB4AEE1093.png)

这时又提示需要我们指定项目的版本了，并且通过上面的提示，我们可以看到，Maven在提示我们版本时，这个版本号的后面添加了SNAPSHOT修饰，这是什么意思呢？

在Maven中后缀添加SNAPSHOT的为快照版本，它本普通的版本在使用上没有任何区别，但是在下载更新时还是有区别的，我们后续的内容中在详细的介绍有关Maven版本的内容，我们继续。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/B468B81E-41BA-4D41-86B6-8F52386D2F8C.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/B468B81E-41BA-4D41-86B6-8F52386D2F8C.png)

Maven提示我们这个项目的包路径是不是我们上面指定的那个`groupId`参数路径。这个我们可以随便修改。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/3F4C6FFC-032A-4954-B3E2-347B29E77A04.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/3F4C6FFC-032A-4954-B3E2-347B29E77A04.png)

当我们执行完成后，Maven提示了我们项目构建成功了。下面我们用IDEA打开这个项目看一样，Maven都会我们生成哪些内容了。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/19CD1C03-F556-4C49-AC6D-DD187F739681.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/19CD1C03-F556-4C49-AC6D-DD187F739681.png)

如上图所示，使用Maven中的`mvn archetype:generate`创建项目骨架时，生成了我们使用Maven默认约定的配置。也就是我们上面内容中手动创建的内容。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C6DAA9ED-1C93-4F23-819D-49A964CC83E2.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/C6DAA9ED-1C93-4F23-819D-49A964CC83E2.png)

并且通过观察我们还发现了在pom.xml中还为我们生成了，很多我们没介绍过的标签内容。这里先不用担心，我们后续的内容中，都会详细的介绍。我们下面执行一下上面生成的代码，看一下项目是否能够运行。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/96C4DEFE-8125-49A3-9E3B-465B43EEDC71.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/96C4DEFE-8125-49A3-9E3B-465B43EEDC71.png)

当我们执行时，我们发现，IDEA中并没有显示可以运行的按钮，这是什么原因呢。这是因为我们是在一个空的项目中创建了上面的另外的一个项目，所以IDEA并没有将这个项目识别出是一个Maven项目，已经这是我们创建空项目的一个目录。要想解决这个问题，有两种途径，一种是用IDEA,直接打开我们生成的这个项目。还有一个办法，就是如下图所示，我们设置一下。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/30351783-E23E-47B0-BDC3-72BAE6888E83.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/30351783-E23E-47B0-BDC3-72BAE6888E83.png)

选中pom.xml文件，然后我们右键选择`Add as Maven Project`即可。

[![Maven实战-第二篇（Maven的入门）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/4AC91109-E7C8-47B6-A615-E61BC1DD13D5.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%8C%E7%AF%87%EF%BC%88Maven%E7%9A%84%E5%85%A5%E9%97%A8%EF%BC%89.resources/4AC91109-E7C8-47B6-A615-E61BC1DD13D5.png)

这样我们就可以执行上图中自动生成的代码了。

  

我们在上一篇中我们介绍过，如果想通过Maven添加项目的依赖时，就需要在pom.xml中引入项目的相关配置，也就是上一篇中说过的，如果要在项目中引入fastjson依赖，需要添加以下的配置：

```
 <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>2.0.35</version>
</dependency>
```

上面的配置，实际上最关键的就是`groupId`、`artifactId`和`version`参数。这三个主要的参数就可以理解为Maven中的坐标。下面在详细介绍Maven坐标之前，我们先回想一下在我们之前学平面几何的知识，我们知道如果要描述一个坐标，我们可以用x轴和y轴来表示。那么在现实世界中，我们可以通过经纬度来定位一个城市的位置。所以在Maven世界中，由于构建的组件非常多，于是也采用了同样的方式，也就是坐标来定位一个组件。但有唯一不同之处，就是相对x轴和y轴、经纬度来说，Maven的坐标中还包括版本等其它额外的信息。

在之前的文章中我们介绍过，我们没有使用Maven之前如果我们想要使用组件时，我们需要去相应的网站中找寻找，然后去下载才可以使用。但这个过程比较繁琐，并且非常不容易找到。如果项目依赖的组件比较多时，这项工作非常的浪费时间。并且在之前的文章中，我们也介绍过，这样常常会导致我们引入的依赖包，还依赖其它的依赖，这样我们很不方便的一次性全部寻找全。所以Maven为了解决上述的问题，方便我们使用和管理依赖。于是在Maven中定义了这样一组规则：世界上任何一个构件也就是依赖包都可以使用Maven坐标唯一标识。通过这个坐标标识，我们可以下载任何的组件。在Maven坐标中主要包括以下元素：groupId、artifactId、version、packaging、classifier等。下面我们就详细介绍一下这些元素的作用及其使用。



# 三、Maven 中的坐标

一、groupId
---------

groupId会被定义为Maven所属的项目名称。不过这个项目名称通常会被命令为公司或者组织的域名反向的命名。因为这样基本上是可以保证唯一的。只不过本书中并不推荐使用公司或者组织命名，因为在Maven中一个项目会被划分为多个模块，如果多个模块都按照此方式命名。那么在Maven中就无法通过groupId坐标来区分不同的模块了，当然我们也可以指定其它的坐标参数来区分，但这不是推荐的方式。所以groupId最好使用公司或者组织在加上项目名称来区分。例如大名鼎鼎的Spring Boot项目就是采用下面的方式定义groupId的：`spring-boot`的groupId就是使用`org.springframework.boot`来定义的。具体配置如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.5</version>
    <type>pom</type>
</dependency>
```

上面的配置中，我们看有了一新的配置也就是type参数，这个我们称不用了解，后续内容中，我们在详细的介绍。这样做除了上面说的好处外，还有一个目的，就是Maven在执行安装命令时，也就是install，会将此依赖安装到本地的仓库中，并且是按照是groupId的目录进行区分的，例如上面的`org.springframework.boot`配置，Maven在安装时会生成`org/springframework/boot`目录。这样如果我们添加了项目的后缀，在本地仓库寻找相应的Jar包就会容易很多。

二、artifactId  
--------------

artifactId元素会被定义为Maven中的项目名称，但由于现有的软件开发架构中，通常都会把Maven划分为多个模块。所以常常将artifactId元素指定为模块的名称。并且推荐加上groupId元素的前缀。这样的好处是，在Maven库中可以很方便的通过artifactId元素的名字来找到我们想要的Jar包。因为就是上面说过的在Maven的库中groupId的命名会被解析为目录，而artifactId的命名才是组件真正的名称。如果我们不加前缀可能会有很多不同的groupId的artifactId的命名重复。虽然不会影响我们使用，但可能会影响我们在本地仓库中寻找。当然Maven也可以通过额外的参数设置来更改构建的名称。也就是这前介绍过的。`finalName`参数。

三、version
---------

version元素定义Maven中的项目的版本。不同的版本可能会有很大的不同，并且不同的版本所依赖其它依赖包的版本也可能是不同的。在Maven中有两个版本类型。一个是正常的版本信息，也就是我们之前看到了1.0这类的。还有一个是快照版本，也就是SNAPSHOT版本。SNAPSHOT版本，在特殊的场景中有不同的效果，我们后续的内容在做详细的介绍。我们现在需要知道，如果我们要引入任何一个依赖包时，必须指定相应的版本，如果我们版本信息有指定的不对时，Maven则不会下载相应的依赖包。

四、packaging
-----------

packaging元素定义为Maven中的打包方式。在Maven中打包方式指的就是项目构建完组件的扩展名。在Maven中默认的打包方式为jar。也就是上一篇中我们介绍过的，我们什么都没有指定，默认Maven打包的方式就是jar包。也就是当然我们忽略这个参数不指定时，Maven会自动构建出jar类型的组件。在Maven中除了jar打包方式外，还有war和pom类型。其中war类型的组建为网站的类型，需要通过应用服务器才可以运行。也就是我们常说的的Java Web项目。而pom类型，则是Maven中父项目的类型。因为Maven中可以划分为多个模块，但一定是需要一个父模块来管理其它的子模块。这个父模块的类型，我们就需要设置为pom类型。

五、classifier
------------

classifier元素定义Maven中的附属构建。我们知道在Maven中可以使用groupId、artifactId和version元素来定位一个组件。但在一些其它的情况下，只使用上面的元素是定位不到组件的。因为有些组件并不是按照上面3个元素构建的。所以只指定上在面三个元素是下载不下来的。这时就需要classifier元素了。例如有些组件需要兼容不的jdk版本，这时我们就需要将classifier元素配置为指定的jdk版本。例如下面的组件配置。

```xml
<dependency>
    <groupId>net.sf.json-lib</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.4</version>
</dependency>
```

按照我们上面介绍的，一般的组件我们只要按照上面的方式配置就可以直接下载这个组件的Jar包。但上面的组件比较特殊，因为它需要指定相应的jdk版本才可以下载。也就是要按照下面的方式才可以。

```xml
<dependency>
    <groupId>net.sf.json-lib</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.4</version>
    <classifier>jdk15</classifier>
</dependency>
```

上面介绍的是Maven中比较常见的元素，当然还有很多其它的元素，需要我们学习，我们先了解上面的内容，后续的章节中我们在慢慢介绍。

 

# 四、Maven 的依赖配置

在之前文章中，我们已经介绍过了，如果要在Maven中添加新的依赖则需要使用dependencies标签来配置Maven的依赖，并且我们已经介绍过了很多有关依赖元素的配置。例如：`groupId、artifactId、version`等。但上面的内容只是我们比较常见的配置，实际上Maven为了满足我们很多需求场景，提供了很多额外的配置，具体的配置如下：

```xml
＜dependencies＞
    ＜dependency＞
        ＜groupId＞……＜/groupId＞
        ＜artifactId＞……＜/artifactId＞
        ＜version＞……＜/version＞
        ＜type＞……＜/type＞
        ＜scope＞……＜/scope＞
        ＜optional＞……＜/optional＞
        ＜exclusions＞
            ＜exclusion＞……＜/exclusion＞
        ＜/exclusions＞
    ＜/dependency＞
＜/dependencies＞
```

上面的部分标签我们在之前的文章中已经介绍过了，下面我们重点介绍一下没有介绍过的标签的作用：

一、type
------

依赖的类型，也就是引入其它依赖组件packaging指定的类型。所以该标签可以选择的参数为`jar、war、pom`。且在Maven中默认的type类型为`jar`。也就是将改依赖的jar包引入到项目中，`war`参数也是如些。下面我们验证一下。

首先我们先创建一个Maven的项目，因为之前的内容已经详细介绍过怎么创建了，所以我们这里就不做过多的介绍了。我们直接创建一个`midai-mazhe-maven`项目。

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/C670B226-14EF-4D5D-B015-AB75ACD30D93.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/C670B226-14EF-4D5D-B015-AB75ACD30D93.png)

然后我们写一个测试类测试一下项目。

*   MaZheMaven

```java
public class MaZheMaven {
    public static void main(String[] args) {
        System.out.println("Hello World MaZhe Maven");
    }
}
```

*   输出

```
Hello World MaZhe Maven
```

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/747B5844-BEF3-4BD4-952D-B3458A10E606.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/747B5844-BEF3-4BD4-952D-B3458A10E606.png)

通过上图证明了我们刚刚创建的项目是没有任何问题的。下面我们还是和之前一样创建一个对象然后使用fastjson工具类转成json。

*   Userinfo

```java
public class Userinfo {
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "Userinfo{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}

```

*   MaZheMaven

```java
import com.alibaba.fastjson.JSONObject;

public class MaZheMaven {
    public static void main(String[] args) {
        System.out.println(JSONObject.toJSONString("Hello World MaZhe Maven"));
        printJson();
    }

    public static void printJson(){
        Userinfo userinfo = new Userinfo();
        userinfo.setUsername("mazhe");
        userinfo.setPassword("maven");
        System.out.println(JSONObject.toJSONString(userinfo));
    }
}
```

由于上面的代码非常的简单，我们就不详细介绍了。下面我们将上面的项目打成包。也就是使用Maven中的`install`命令。下面我们在项目的根目录中执行下面的命令：

```bash
mvn clean install
```

上面实际上是两个命令，`install`命令我们介绍过，就是将依赖包安装到达本地仓库中。而`clean`命令的作用就是清理的意思。我们知道当执行Maven命令编译或者打包时，当执行完成后，会在项目的根目录中创建一个target目录，然后Maven将相应的jar包生成到这个目录中。而`clean`命令的作用就是删除这个目录。如果我们不执行`clean`命令，而直接执行`install`命令而Maven并不会自动删除之前target目录下生成的内容。所以为了方便，也为了我们不会使用错jar包。我们实际的工作中基本上会联合使用上面的命令。

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/36678299-745D-49D4-B0EB-8AF346CAD66B.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/36678299-745D-49D4-B0EB-8AF346CAD66B.png)

我们之前也介绍过，当使用`install`命令安装成功后，会将这个依赖包安装到本地的仓库中，这样的好处是其它项目也可以通过Maven依赖的配置引入到自己的项目中。下面我们在创建一个新的项目，来测试一下。

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/85654A5C-6146-44DD-8124-441F7667EDCE.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/85654A5C-6146-44DD-8124-441F7667EDCE.png)

这次我们添加一个测试类测试一下代码。为了我们可以使用测试用例，所以我们需要添加`junit`依赖包，具体配置如下：

*   pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.ma-zhe</groupId>
    <artifactId>mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>
    </dependencies>

</project>
```

下面我们创建一个测试类，简单输出一句话，来验证，我们引入的依赖包，没有任何问题。

*   MazheMaveTest

```java
import org.junit.Test;

public class MazheMavenTest {
    @Test
    public void test(){
        System.out.println("Hello World Mazhe Maven");
    }
}
```

*   输出

```
Hello World Mazhe Maven
```

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/1AE3A4B7-8F98-4689-B592-828967594B5D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/1AE3A4B7-8F98-4689-B592-828967594B5D.png)

看上图所示，我们成功引入了junit依赖了。下面我们引入我们上一个项目的依赖，看一下能否直接使用。具体配置如下：

*   pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.ma-zhe</groupId>
    <artifactId>mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>midai-mazhe-maven</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

下面我们直接在测试类中使用上一个项目的代码，看一下是否能直接使用。

*   MazheMavenTest

```java
import org.junit.Test;

public class MazheMavenTest {
    @Test
    public void test(){
        MaZheMaven.printJson();
    }
}
```

*   输出

```
{"password":"maven","username":"mazhe"}
```

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/A5B4F8F6-B61A-4DEB-9718-D605A8A21027.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/A5B4F8F6-B61A-4DEB-9718-D605A8A21027.png)

我们看上图所示，我们成功了引入了上个项目的依赖。因为默认`type`的参数的值为`jar`。所以我们上述的配置实际上就是验证了引入外部依赖包的功能。那么对于`war`的参数配置也是一样。下面我们验证一下如果将`type`的参数设置为`pom`，然后我们在从另外的项目中引入会有什么效果呢？下面我们尝试一下。首先在`midai-mazhe-maven`项目中将`packaging`参数设置为`pom`。配置如下：

*   pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.ma-zhe</groupId>
    <artifactId>midai-mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.6</version>
        </dependency>
    </dependencies>

</project>
```

下面我们在修改一下mazhe-maven项目的pom.xml文件。将刚刚引入的midai-mazhe-maven项目的依赖配置的`type`参数配置成`pom`。  
然后我们在观察一下测试类的代码，看看有啥不同。

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/60CAC658-D6FD-44EC-9DF0-06ADF37FA9F4.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/60CAC658-D6FD-44EC-9DF0-06ADF37FA9F4.png)

我们看midai-mazhe-maven项目中的MaZheMaven显示报错了，这就表明了，我们这个项目中并没有成功的引入这个类的依赖。这个就是`type`参数配置成`pom`的左右。当然这样配置时，Maven并不会引入这个项目本身的任何依赖类，所以我们这个项目中也就引入不到MaZheMaven类了。那这时可能有人想问了，那这样我还引入这个依赖包干什么呢？当这样配置时，虽然Maven不会引入这个依赖包项目本身的代码，但是它将这个项目的配置依赖引入到新项目中，可以简单的理解为将这个项目的pom.xml中的配置引入过来。因为midai-mazhe-maven项目的pom.xml中引入了fastjson依赖，下面我们看一下，我们直接在mazhe-maven项目中是否可以直接使用这个依赖呢？如果可以则说明了，当将`type`参数配置成`pom`时，确实会引入依赖配置。下面我们尝试一下。

*   MazheMavenTest

```java
import com.alibaba.fastjson.JSONObject;
import org.junit.Test;

public class MazheMavenTest {
    @Test
    public void test(){
//        MaZheMaven.printJson();
        System.out.println(JSONObject.toJSONString("Hello World Mazhe Maven"));
    }
}

```

*   输出

```
"Hello World Mazhe Maven"
```

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/09D38332-45B2-4889-8AF8-064A619D3B0B.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/09D38332-45B2-4889-8AF8-064A619D3B0B.png)

我们看上图所示，成功的使用了fastjson的工具类了，虽然我们项目本身没有引入fastjson的依赖，但是因为我们引入了midai-mazhe-maven项目，该项目配置了fastjson的依赖。所以本项目可以直接使用。这就是type参数的作用。

二、scope
-------

依赖的范围。通过此标签可以指定Maven的打包行为。下面我们介绍一下，scope参数都有哪些配置参数：

*   compile

编译的依赖范围。该依赖访问是一个比较强的依赖，配置此依赖的依赖包，将参与项目的编译、测试、运行等操作，并且在项目打包时，会将此依赖包部署到打包文件中。该依赖也是Maven中scope参数的默认配置。

下面我们通过示例验证一下，上面说的内容。首先我们引入一个依赖包也就是之前介绍过的fastjson依赖包，具体配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.ma-zhe</groupId>
    <artifactId>midai-mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.6</version>
        </dependency>
    </dependencies>

</project>
```

按照上面介绍的，当我们什么都不指定的时候，scope参数的默认配置为：`compile`。下面我们创建两个类，一个在项目的主代码目录中，一个在项目的测试目录中。

*   MaZheMaven

```java
import com.alibaba.fastjson.JSONObject;

public class MaZheMaven {
    public static void main(String[] args) {
        System.out.println(JSONObject.toJSONString("Hello World MaZhe Maven"));
    }
}
```

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/6EECB137-6B50-4CFF-9E88-65AE2835AD8B.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/6EECB137-6B50-4CFF-9E88-65AE2835AD8B.png)

*   MaZheMavenTest

```java
import com.alibaba.fastjson.JSONObject;

public class MaZheMavenTest {
    public static void main(String[] args) {
        System.out.println(JSONObject.toJSONString("Hello World MaZhe Maven Test"));
    }
}
```

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/C5C61E8F-7E2D-4B63-A329-6CC08A5479C6.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/C5C61E8F-7E2D-4B63-A329-6CC08A5479C6.png)

由于`compile`依赖范围的是针对编译、测试和运行都是有效的，所以上面的代码均可运行，都可以引入fastjson的依赖包。

*   test

测试的依赖范围。顾名思义，该依赖只对测试环境有效，所以当配置为此依赖时，并不会参与项目的编译和运行，打包时也不会将此依赖部署到打包文件中。该依赖最常使用的场景就是测试用例的依赖。

同样样我们也验证一下test的配置范围。我们将之前引入的fastjson依赖包的scope的参数修改为test。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.ma-zhe</groupId>
    <artifactId>midai-mazhe-maven</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.6</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

然后我们什么不都修改，在看一下上面的两具类的代码有啥变化。

*   MaZheMaven

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/B561BC5A-A555-46D2-9B3D-2E3B78D997CC.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/B561BC5A-A555-46D2-9B3D-2E3B78D997CC.png)

*   MaZheMavenTest

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/8ED3BBA3-83C5-465F-AE14-B5FE8ED96072.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/8ED3BBA3-83C5-465F-AE14-B5FE8ED96072.png)

如上图所示，我们看到MaZheMaven类报错了。这是因为上面所说的，test的依赖范围只对测试环境有效，其它环境均是无效的。因为MaZheMaven类在主代码目录中，所以报错了。下面我们打包一下这个项目，看一下test范围的依赖包会不会生成在打包文件中。在执行打包命令前，需要将MaZheMaven类的代码注释，否则会报编译失败错误。

*   runntime

运行时依赖范围。该依赖范围适用于运行和测试范围。在编译的环境下不会被使用。最常使用的场景就是数据库的驱动依赖，因为只有项目运行时才会被使用。

我们继续将scope参数修改为runntime看一下代码的效果。

1、pom.xml

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/D31597E5-650A-428F-A95A-13E281BF7A2A.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/D31597E5-650A-428F-A95A-13E281BF7A2A.png)

2、MaZheMaven

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/75115829-2C68-4101-BDBD-D88168FC0230.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/75115829-2C68-4101-BDBD-D88168FC0230.png)

3、MaZheMavenTest

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/31B0863E-9113-4231-8DE3-1A3913BDCE0D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/31B0863E-9113-4231-8DE3-1A3913BDCE0D.png)

我们看上图所示，因为runntime依赖范围是对运行和测试有效的，所以主代码中的依赖依赖报错了，而测试目录中代码则没有任何问题。

*   provided

此依赖范围适用于编译和测试范围。项目打包时并不会打包此依赖范围的依赖。最常用的使用场景是servlet相应的依赖。因为该依赖范围，必须在容器中运行，容器中已经有了相关的jar包所以不需要额外添加了。

我们继续将scope参数修改为provided看一下代码的效果。

1、pom.xml

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/5888D027-CBBA-4CB0-985A-5D9FF59C167B.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/5888D027-CBBA-4CB0-985A-5D9FF59C167B.png)

2、MaZheMaven

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/86F136A3-689B-48D1-8C7D-40C30DA27BC8.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/86F136A3-689B-48D1-8C7D-40C30DA27BC8.png)

3、MaZheMavenTest

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/D263B5DD-F6FD-4C0C-94B6-E0785B46A4B4.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/D263B5DD-F6FD-4C0C-94B6-E0785B46A4B4.png)

还是如图所示，我们看这次主代码的和测试代码都没有报错。这是因为provided参数的依赖范围是针对编译和测试环境的，所以主代码报错的问题也就不存在了。

*   system

此依赖范围适用于编译和测试范围。和provided参数一致，不同的地方在于，此依赖配置需要添加额外的配置，因为此依赖配置不是从Maven中获取依赖的，于是通过系统中配置的路径获取依赖的。具体的配置如下：

```xml
<scope>system</scope>
<systemPath>../fastjson-2.0.6.jar</systemPath>
```

我们继续将scope参数修改为system看一下代码的效果。

1、pom.xml

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/E1A4EDED-53AE-4777-B3B2-26BF6CB53961.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/E1A4EDED-53AE-4777-B3B2-26BF6CB53961.png)

2、MaZheMaven

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/DA67FF71-999A-462A-B5AF-5AF7447C6301.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/DA67FF71-999A-462A-B5AF-5AF7447C6301.png)

3、MaZheMavenTest

[![Maven实战-第四篇（Maven的依赖配置）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/5385D040-5A8F-4096-926B-7E4DC3F6704D.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%9B%9B%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE%EF%BC%89.resources/5385D040-5A8F-4096-926B-7E4DC3F6704D.png)

还是如图所示，我们看这次主代码的和测试代码都没有报错。这是因为provided参数的依赖范围是针对编译和测试环境的，所以主代码报错的问题也就不存在了。

* * *



三、optional
----------

下面我们介绍一下optional参数，该参数的作用是标记依赖是否可选。当参数设置为true时，则表示子项目不会依赖此组件。也可以理解为子项目并不会继承这个依赖。由于我们后续的内容中会详细的介绍这个参数的作用，所以这里就不在做过多的介绍和项目演示了了。



四、exclusions
------------

exclusions参数的目的是排除传递性依赖。由于传递性依赖的内容，我们也将在后续的内容中在做详细介绍，所以本篇也不在做项目演示了。



# 五、Maven 的传递性依赖

今天我们分享的内容是Maven中的传递性依赖。在介绍这个内容之前，在使用Maven时会不会有这样疑惑。也就是当我们引入一个组件时，需要在pom.xml中配置相关依赖的标签。但这个组件可能还会依赖其它的组件。如果我们每引入一个依赖时，都需要将这个依赖所依赖的组件配置写到pom.xml中这样会导致，我们需要配置的依赖的标签就会非常的多，并且我们也很难知道一个组件和依赖的其它的组件到底是什么。且这样也违背了Maven通过坐标引入组件的目的了。因为它不够简单与便捷。为了解决上述的问题，Maven引入了传递性依赖。所谓的传递性依赖的，指的就是当我们引入一个组件时，如果这个组件还依赖其它的组件，那么Maven就会自动将其它的组件也一同下载下来，并且并不需要我们引入这些额外的依赖。

* * *

下面我们通过代码验证一下，还是我们之前的代码事例，也就是引入`fastjson`依赖包，来验证一下上面所说。具体配置如下：

*   pom.xml

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>2.0.26</version>
</dependency>
```

我们创建一个新的项目，并在项目中添加上面的依赖。

[![Maven实战-第五篇（Maven的传递性依赖）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/1D97A450-8E90-4231-ACFD-377454B3ADEC.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/1D97A450-8E90-4231-ACFD-377454B3ADEC.png)

我们展开`External Libraries`目录，就可以现在项目中引入的所有外部的依赖包。

[![Maven实战-第五篇（Maven的传递性依赖）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/615D1434-0572-4E52-B1B0-A47040A5A928.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/615D1434-0572-4E52-B1B0-A47040A5A928.png)

我们看上图所示，项目引入了3个依赖包，但我们依赖的配置只配置了一个依赖，这就是Maven传递性依赖的作用，如果没有传递性依赖，则上面只有显示一个依赖包，并且需要我们手动的将其它的依赖包配置到项目中，才可以使用。

* * *

除了通过上面的方式查看项目的依赖外，IDEA中还提供了以下的方式查看依赖，并且这种方式更直观。

首先，我们选中项目中的pom.xml。然后右键选择`Maven`然后选择`Show Diagram ...`

[![Maven实战-第五篇（Maven的传递性依赖）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/B0571435-751E-4946-B70C-0EE00760F108.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/B0571435-751E-4946-B70C-0EE00760F108.png)

当我们点击后就会看到下图所示：

[![Maven实战-第五篇（Maven的传递性依赖）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/2953FEEE-BED8-4A17-AB0B-9711712F4115.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%BA%94%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BC%A0%E9%80%92%E6%80%A7%E4%BE%9D%E8%B5%96%EF%BC%89.resources/2953FEEE-BED8-4A17-AB0B-9711712F4115.png)

它显示了我们项目所有的依赖包，并且会按照我们传递性依赖的顺序显示。按照上图显示，我们知道，虽然，我们只引入了`fastjson`依赖，但是因为它依赖了`fastjson-extension`。所以Maven会继续引入这个依赖包。又因为`fastjson-extension`依赖了`fastjson2`，所以Maven又会引入这个依赖。这也就是传递性依赖引入依赖包的顺序。

* * *

除此之外，Maven也提供了相应的命令来查看我们项目中的依赖。具体命令如下：

*   mvn dependency:list

使用上述命令可以列出我们项目所有的依赖包。

*   mvn dependency:tree

此命令和上面的命令一样，也可以列出我们项目所有的依赖包，不同之处就是这个命令可以按照树的方式展示，方便我们查看依赖关系。而上面的命令是同一级显示的。

*   mvn dependency:analyze

使用此命令Maven可以分析当前项目的依赖。

* * *

由于后续的内容中有专门的章节详细介绍上面的命令，所以这里就不做过多的介绍了。上面的内容就是Maven中传递性依赖的相关内容，它也是Maven可以非常方便我们管理依赖包的因素之一。



# 六、Maven 的依赖范围

今天我们分享一下Maven中的依赖范围。因为在之前的`Maven实战-第四篇（Maven的依赖配置）`文章中已经对一部分内容进行了详细的介绍，所以本篇中涉及到上述的内容将不在过多的介绍了。

一、一般依赖范围
--------

在Maven中依赖范围的目的控就是控制项目中classpath的关系。在Maven中主要有三种classpath。分别是编译的classpath、测试的classpath和运行的classpath。下面我们看一下不同依赖范围对上述三种classpath的影响。

1、compile

编译依赖范围，也是Maven默认的依赖范围。使用此依赖范围时表示，此组件对编译、测试和运行都有效。所以此依赖范围，对于上面的三个classpath都是有效的。

* * *

2、test

测试依赖范围。使用此依赖范围表示，此组件只对测试的classpath有效。对编译的和运行时无法使用此依赖。比较典型的组件为：Junit组件，具体配置如下：

```
 <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
```

因为`Maven实战-第四篇（Maven的依赖配置）`文章已经详细介绍过了，所以这里就不在进行演示了。

* * *

3、provided

使用此依赖范围。组件将对编译和测试有效，而对运行时无效。比较典型的组件为：servlet-api。此组件我们编译和测试的时需要依赖，但在运行时，由于应用服务器已经提供了此组件的支持，所以不需要我们在项目中添加此依赖了。同样的因为之前文章中已介绍过了，所以这里就不在过多介绍了。

* * *

4、runtime

运行时依赖范围。也就是只对运行时才有效，但对测试也是有效的。典型的组件为数据库的驱动。只有项目运行时才需要。对其它的classpath是无效的。

* * *

5、system

系统依赖范围。system依赖范围和provided的依赖范围一致。唯一不同的之处就是在使用system依赖范围时，我们需要指定systemPath参数。因为system依赖范围并不是通过Maven仓库解析的，而是通过systemPath参数指定的路径解析的。具体的配置如下：

```
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    ＜scope＞system＜/scope＞
    ＜systemPath＞/Users/jilinwula/.m2/repository/junit/junit/4.13.2/junit-4.12.jar＜/systemPath＞
</dependency>
```

比较典型的场景为加密项目中可能会使用。它会读取我们指定目录中的依赖包加载到项目中。例如从外部设备中读取。

* * *

6、import

导入依赖范围。此依赖范围和其它的依赖不同，它并不会影响编译、测试和运行的classpath。而是通过此依赖范围，将其它项目的依赖引入本项目中。所以在使用此依赖时，需要引入的依赖类型必须为pom类型。

* * *

下面是依赖范围和classpath之间的关系。

| 依赖范围（scope） | 编译（classpath） | 测试（classpath） | 运行（classpath） | 例子        |
| ----------------- | ----------------- | ----------------- | ----------------- | ----------- |
| compile           | Y                 | Y                 | Y                 | spring-core |
| test              | \-                | Y                 | Y                 | junit       |
| provided          | Y                 | Y                 | \-                | servlet-api |
| runtime           | \-                | Y                 | Y                 | JDBC驱动    |
| system            | Y                 | Y                 | \-                | 本地的组件  |

横轴表示classpath，纵轴则表示Maven中的依赖范围。表示`Y`则表示对该classpath有效果，反之`-`则表示无效。

二、传递性依赖范围
---------

上面介绍的是一般的依赖范围。但我们知道也就是上篇中介绍的，Maven中有传递性依赖，那么这个依赖会不会和一般的依赖范围有些不同呢。答案是肯定的。在Maven中依赖范围不仅可以控制依赖与classpath的关系，还对传递性依赖会产生影响。下面内容是依赖范围影响传递性依赖的关系：

|          | compile  | test | provided | runtime  |
| -------- | -------- | ---- | -------- | -------- |
| compile  | compile  | \-   | \-       | runtime  |
| test     | test     | \-   | \-       | test     |
| provided | provided | \-   | provided | provided |
| runtime  | runtime  | \-   | \-       | runtime  |

在Maven中如果项目A依赖于B,项目B又依赖于项目C，我们会说A对于B是第一直接依赖，B对于C是第二直接依赖，而A对于C则是传递性依赖。也就早篇中介绍的内容。所以在上面的表格中最左边的一列表示第一直接依赖范围，最上面一行表示的是第二直接依赖范围，中间的交叉单元格则表示传递性依赖的范围。

* * *

下面我们通过一个简单的例子来解释一下上面说的内容。假如我们有一个项目叫做account-email。该项目有一个com.icegreen:greenmail:1.3.1b的直接依赖，我们说这是第一直接依赖，并且假如该依赖范围是test。而greenmail又有一个javax.mail:mail:1.4的直接依赖，我们说这是第二直接依赖，其依赖范围是compile。显然javax.mail:mail:1.4是account-email的传递性依赖。按照上图所示，当第一直接依赖范围为test，第二直接依赖范围是compile的时候，传递性依赖的范围是test，因此javax.mail:mail:1.4是account-email的一个范围是test的传递性依赖。

* * *

我们通过下面的例子来验证一下上面说的内容。为了能够直观的观察第一依赖与第二依赖以及传递性依赖的关系。我们创建三个项目然后依次依赖，来验证上面说的内容。

1、mazhe-maven-user

*   pomx.ml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-user</artifactId>
</project>
```

*   MazheMavenUser

```
public class MazheMavenUser {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven User!");
    }
}
```

2、mazhe-maven-order

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-order</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-user</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

*   MazheMavenOrder

```
public class MazheMavenOrder {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven Order!");
    }
}
```

3、mazhe-maven-mall

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-mall</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-order</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

*   MazheMavenMall

```
public class MazheMavenMall {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven Mall!");
    }
}
```

我们创建了三个项目`MazheMavenUser、mazhe-maven-order、mazhe-maven-mall`。期中项目`mazhe-maven-mall`依赖于项目`mazhe-maven-order`，项目`mazhe-maven-order`又依赖于项目`MazheMavenUser`，所以项目`MazheMavenUser`就是项目`mazhe-maven-mall`的传递性依赖。下面我们按照上面的依赖关系来验证一下，当第一依赖是`compile`，第二依赖也是`compile`时，传递性依赖是不是也是`compile`的依赖范围。因为Maven中`compile`依赖范围为默认的依赖范围，所以我们不需要修改任何`pom.xml`直接在项目中验证一下。因为我们项目中引入了`mazhe-maven-order`依赖。所以因为传递性依赖的关系，Maven也会引入`MazheMavenUser`依赖。所以我们直接在MazheMavenMall中调用这两个项目的代码看看是否能成功的引用。

compile&compile
---------------

*   MazheMavenMall

```

public class MazheMavenMall {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven Mall!");
        MazheMavenOrder.main(args);
        MazheMavenUser.main(args);
    }
}
```

\*输出

```
Hello World Mazhe Maven Mall!
Hello World Mazhe Maven Order!
Hello World Mazhe Maven User!
```

我们看上面输出的日志，成功的验证了当第一依赖为`compile`,并且第二依赖也是`compile`的时候，传递性依赖的范围也是`compile`。

|         | compile |
| ------- | ------- |
| compile | compile |

[![Maven实战-第六篇（Maven的依赖范围）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/196E9F69-E40C-4F0F-A801-855F4EA266B9.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/196E9F69-E40C-4F0F-A801-855F4EA266B9.png)

* * *

compile&test
------------

下面我们验证一下，当然第一依赖范围为`compile`，第二依赖范围为`test`的时候，传递性依赖的范围。因为是要修改的第二依赖范围，所以我们需要将mazhe-maven-order项目中的依赖范围修改为`test`。

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-order</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-user</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

我们再次看一下mazhe-maven-mall项目中，是否有问题。

[![Maven实战-第六篇（Maven的依赖范围）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/F1164E1D-4F50-4504-B8C4-AD9EF7AADA8C.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/F1164E1D-4F50-4504-B8C4-AD9EF7AADA8C.png)

如上图所示，我们看到MazheMavenMall类中的调用MazheMavenUser代码报错，这是因为没有找到这个类的依赖所至。下面我们在mazhe-maven-mall项目中创建一个测试类，来验证一下测试代码是否能够成功的引用。

*   MazheMavenMallTest

```
public class MazheMavenMallTest {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven Mall Test!");
        MazheMavenOrder.main(args);
        MazheMavenUser.main(args);
    }
}
```

[![Maven实战-第六篇（Maven的依赖范围）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/7241C5B3-C6AB-4BE7-B4B6-38626551BCB0.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/7241C5B3-C6AB-4BE7-B4B6-38626551BCB0.png)

我们看测试的代码也显示出无法引用MazheMavenUser类，这也间接证明了，当第一依赖范围为`compile`，第二依赖范围为`test`时候，Maven不会进行传递性依赖。

|         | compile | test |
| ------- | ------- | ---- |
| compile | compile | \-   |

* * *

compile&provided
----------------

下面我们同样的验证一下，当然第一依赖范围为`compile`，第二依赖范围为`provided`的时候，传递性依赖的范围。我们同样的将mazhe-maven-order项目中的依赖范围修改为`provided`。

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-order</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-user</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

我们同样的在看一下MazheMavenMall类的代码是否能成功的引入MazheMavenUser类。

[![Maven实战-第六篇（Maven的依赖范围）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/27644D9A-A1CC-47F6-8123-EC6110C8E9C3.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/27644D9A-A1CC-47F6-8123-EC6110C8E9C3.png)

我们在看一下MazheMavenMallTest类的代码是否能引入。

[![Maven实战-第六篇（Maven的依赖范围）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/97400CB3-CE00-4BA8-BACF-47C22EF576CF.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/97400CB3-CE00-4BA8-BACF-47C22EF576CF.png)

我们看上图所示，不管是主代码中还是测试代码中，都无法引入MazheMavenUser类，这也就说明了，当第一依赖范围为`compile`，第二依赖范围为`provided`的时候，Maven不会进行传递性依赖。

|         | compile | test | provided |
| ------- | ------- | ---- | -------- |
| compile | compile | \-   | \-       |

* * *

compile&runtime
---------------

下面我们继续验证，当然第一依赖范围为`compile`，第二依赖范围为`runtime`的时候，传递性依赖的范围。我们继续的将mazhe-maven-order项目中的依赖范围修改为`runtime`。

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-order</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-user</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
</project>
```

我们继续看MazheMavenMall类中的代码。

[![Maven实战-第六篇（Maven的依赖范围）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/3CC1DF91-B8DE-4BC5-9BE1-1F0FCC046071.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/3CC1DF91-B8DE-4BC5-9BE1-1F0FCC046071.png)

我们看上图所示，因为我们第二依赖范围是runtime，所以直接调用MazheMavenUser报错了。下面我们验证一下，如果项目运行时，会不会可以直接调用MazheMavenUser类。为了验证项目运行，我们需要通过反射的主式，来调用MazheMavenUser类，下面为具体的代码：

*   MazheMavenMall

```
import java.lang.reflect.Method;

public class MazheMavenMall {
    public static void main(String[] args) throws Exception {
        System.out.println("Hello World Mazhe Maven Mall!");
        MazheMavenOrder.main(args);
        Class<?> clazz = Class.forName("MazheMavenUser");
        Method method = clazz.getMethod("main", String[].class);
        method.invoke(null, (Object) args);
    }
}
```

*   日志

```
Hello World Mazhe Maven Mall!
Hello World Mazhe Maven Order!
Hello World Mazhe Maven User!
```

我们看通过Java反射的方式，我们成功的调用了MazheMavenUser的方法，这也就说明了当第一依赖范围为compile，第二依赖范围为runtime的时候。Maven的传递性依赖范围为runtime。下面我们验证一下测试类的代码。

*   MazheMavenMallTest

```
public class MazheMavenMallTest {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven Mall Test!");
        MazheMavenOrder.main(args);
        MazheMavenUser.main(args);
    }
}
```

[![Maven实战-第六篇（Maven的依赖范围）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/9BCCDD27-3D97-4BC0-9D38-AC0C9521E56A.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E5%85%AD%E7%AF%87%EF%BC%88Maven%E7%9A%84%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4%EF%BC%89.resources/9BCCDD27-3D97-4BC0-9D38-AC0C9521E56A.png)

我们看上图所示，测试类中的代码可以直接调用MazheMavenUser类的方法。这是因为runtime依赖范围是对测试和运行同样有效的，所以不会报错，我们也修改一下测试类的代码，来验证一下，测试类中运行是否有问题。

*   MazheMavenMallTest

```
import java.lang.reflect.Method;

public class MazheMavenMallTest {
    public static void main(String[] args) throws Exception {
        System.out.println("Hello World Mazhe Maven Mall Test!");
        MazheMavenOrder.main(args);
        MazheMavenUser.main(args);
        Class<?> clazz = Class.forName("MazheMavenUser");
        Method method = clazz.getMethod("main", String[].class);
        method.invoke(null, (Object) args);
    }
}
```

*   日志```
    Hello World Mazhe Maven Mall Test!
    Hello World Mazhe Maven Order!
    Hello World Mazhe Maven User!
    Hello World Mazhe Maven User!
    ```
    

这就证明了当第一依赖范围为compile，第二依赖范围为runtime的时候。Maven的传递性依赖范围为runtime。

|         | compile | test | provided | runtime |
| ------- | ------- | ---- | -------- | ------- |
| compile | compile | \-   | \-       | runtime |

* * *

其它依赖范围的测试，由于和上面内容类似，我们就不依依验证了，我们只要参考上面的表格就可以知道传递性依赖的范围了。所以通过上面的表格我们可以发现这样的规律：

*   当第二直接依赖的范围是compile的时候，传递性依赖的范围与第一直接依赖的范围一致。
*   当第二直接依赖的范围是test的时候，Maven不会进行传递性依赖。
*   当第二直接依赖的范围是provided的时候，只传递第一直接依赖范围也为provided的依赖，且传递性依赖的范围同样为provided。
*   当第二直接依赖的范围是runtime的时候，传递性依赖的范围与第一直接依赖的范围一致，但compile例外，此时传递性依赖的范围为runtime。

  



# 七、Maven 中的依赖冲突

一、什么是Maven中的依赖冲突
----------------

我们在第五篇`Maven实战-第五篇（Maven的传递性依赖）`中介绍了Maven中传递性依赖的内容。并且通过上一篇的内容我们也了解到了传递性依赖为我们使用Maven所带来的好处。但是同样的传递性依赖也为我们带来了使用的弊端，也就是Maven中的依赖冲突。那什么是依赖冲突呢？假如我们有一个项目A。它依赖于项目X。还有一个项目B也依赖于项目X。但是呢这两个项目所依赖X的版本不一致。但是Maven中又支持传递性依赖，所以按照正常的逻辑，这两个X的版本的项目都会引入到项目中，但这样就会造成依赖冲突。因为项目中有两个一样的依赖只是版本不同，所以Maven就不知道引入哪个依赖了，这就是Maven中的依赖冲突。

二、怎么解决Maven的依赖冲突
----------------

所以在Maven中为了解决上面说的依赖冲突引入了一个概念就是依赖调节，其实也就是解决依赖冲突的一个默认规则。因为我们知道，依赖冲突的本质是传递性依赖的版本不一致导致的。所以我们只要制定一个规则，来让Maven可以选择一个传递性依赖的版本就可以了。Maven就是通过这样的方式解决依赖冲突的。下面我们详细介绍一下这个规则到底是什么。

### 规则一

* * *

这个规则就是引入依赖的路径近者优先。举例来说就是如果项目的依赖路径是这样的：A->B-C-X（1.0）。然后还有一个项目依赖路径为A->D->X（2.0）。按照上面引入依赖的路径来判断，我们知道后者的路径较短。所以Maven在执行传递性依赖时会按照上述的规则引入X（2.0）的版本依赖。

* * *

下面我们通过一个例子来验证一上上面所说的规则。我们以上个项目为例，创建4个项目它们分别是：`mazhe-maven-mall、mazhe-maven-order、mazhe-maven-user、mazhe-maven-payment`。其中它们的依赖关系是`mazhe-maven-mall`依赖于`mazhe-maven-order`和`mazhe-maven-payment`，而`mazhe-maven-order`又依赖于`mazhe-maven-user`。这样我们就模拟出来依赖路径不一样的，来验证规则一。具体配置如下:

1、mazhe-maven-mall

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-mall</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-order</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-payment</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

* * *

2、mazhe-maven-order

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-order</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-user</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

* * *

3、mazhe-maven-payment

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-payment</artifactId>
    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.32</version>
        </dependency>
    </dependencies>
</project>
```

* * *

4、mazhe-maven-user

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-user</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.35</version>
        </dependency>
    </dependencies>

</project>
```

* * *

通过上面的配置，我们分别在`mazhe-maven-payment`和`mazhe-maven-user`中引入了`fastjson`的依赖，并且为了模拟依赖冲突，我们将这两个项目所依赖的`fastjson`的版本设置成了不一样的版本，其中`mazhe-maven-payment`的版本为：`2.0.32`，而`mazhe-maven-user`的版本为：`2.0.35`。下面我们验证一下，当这样的依赖冲突后，我们`mazhe-maven-mall`项目依赖的版本到底是多少。我们还是通过`Show Deagram ...`的方式来查看`mazhe-maven-mall`项目的依赖情况。

[![Maven实战-第七篇（Maven中的依赖冲突）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/A134F0CD-9BF5-4AA5-9043-1564191D8967.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/A134F0CD-9BF5-4AA5-9043-1564191D8967.png)

通过上图我们也发现IDEA中已经将这两个依赖冲突的版本显示出来了并且非常直观的标识出了`mazhe-maven-user`项目依赖的`fastjson`的依赖已经指向了`mazhe-maven-payment`的`fastjson`版本。所以此时`mazhe-maven-mall`项目通过传递性依赖所引入的`fastjson`版本就是`2.0.32`。如果我们将`mazhe-maven-payment`和`mazhe-maven-user`依赖的版本进行一下对换。在查看一下就会发现，`mazhe-maven-mall`项目所依赖的版本就会变成`2.0.35`了。

[![Maven实战-第七篇（Maven中的依赖冲突）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/DBA38F89-79DE-4C42-AA6D-D343566C7011.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/DBA38F89-79DE-4C42-AA6D-D343566C7011.png)

这就是Maven中解决依赖冲突的第一个规则，最短路径原则，因为项目`mazhe-maven-payment`依赖的`fastjson`的路径最短，所以传递性依赖以`mazhe-maven-payment`项目配置的版本为准。

* * *

### 规则二

当然这不能解决所有依赖冲突的问题，因为如果项目依赖路径相同时，上面的规则就不起作用了。所以为了解决这个问题。Maven又引入了第二条规则。当第一条规则不能解决依赖冲突时，第二条规则则会起作用。在Maven中规定，如果项目依赖的路径相同时，Maven以依赖声明的顺序决定依赖冲突。也就是说，如果项目依赖的路径相同，则谁先引入这个依赖，Maven就会默认引入这个项目的传递性依赖。

* * *

同样的我们还是通过例子的方式来验证一下上面说的内容。我们将上面的代码进行一下修改，直接在`mazhe-maven-order`项目中引入`fastjson`的依赖，并且它两个项目的版本配置的不一样的。

1、mazhe-maven-order

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-order</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.35</version>
        </dependency>
    </dependencies>

</project>
```

* * *

2、mazhe-maven-payment

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-payment</artifactId>
    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.32</version>
        </dependency>
    </dependencies>
</project>
```

* * *

3、mazhe-maven-mall

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-mall</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-order</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-payment</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

* * *

下面我们用同样的方式，看一下`mazhe-maven-mall`项目的依赖。

[![Maven实战-第七篇（Maven中的依赖冲突）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/A16931EA-A4F9-411A-8975-3AB14EA0248E.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/A16931EA-A4F9-411A-8975-3AB14EA0248E.png)

如图所示,Maven引入的传递性依赖`fastjson`的版本为：`2.0.35`。下面我们将`mazhe-maven-mall`项目中依赖的顺序调换一下。

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-mall</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-payment</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-order</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

我们在查看一下`mazhe-maven-mall`项目的依赖情况。

[![Maven实战-第七篇（Maven中的依赖冲突）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/A919345C-976A-4314-8C73-A95D7C13A509.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/A919345C-976A-4314-8C73-A95D7C13A509.png)

我们看这次`mazhe-maven-mall`项目的依赖版本就变成了：`2.0.32`了。这也就是Maven为了解决依赖冲突制定的第二个规则。按照配置依赖的顺序决定传递性依赖的版本。

三、Maven中怎么排除依赖
--------------

就是因为传递性依赖的原因，在使用Maven时常常会引入很多额外的依赖包。Maven这么做的目的当然是为了方便了我们依赖管理。但这样也会导致一些其它的问题。例如如果传递性依赖引入了一些其它版本的依赖包。但因为有些原因，实际的的项目中却不能引入这个版本的依赖包，需要引入其它版本的依赖包。那该如何解决呢？在Maven中为了解决这样的问题，引入了排除传递性依赖的方式，也就是在Maven中可以使用`exclusions`标签来排除传递性依赖。当通过下面的方式配置后，Maven将不在进行传递性依赖了。具体的配置如下：

```
<dependencies>
    <dependency>
      <groupId></groupId>
      <artifactId></artifactId>
      <version></version>
      <exclusions>
        <exclusion>
            <groupId></groupId>
            <artifactId></artifactId>
        </exclusion>
      </exclusions>
    </dependency>
</dependencies>
```

我们还是以刚刚的例子为例，因为我们在项目中有引入了`mazhe-maven-payment`依赖，所以在路径相同时，Maven会直接引入这个项目的依赖包。我们通过下面的配置将`mazhe-maven-payment`项目依赖`fastjson`的包排除掉，然后在看一下依赖情况。具体配置如下：

1、mazhe-maven-mall

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-mall</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-payment</artifactId>
            <version>1.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>com.alibaba</groupId>
                    <artifactId>fastjson</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-order</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

我们在查看一下`mazhe-maven-mall`项目的依赖情况。

[![Maven实战-第七篇（Maven中的依赖冲突）](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/D3E69763-2463-4861-9748-86A4965EDD8C.png)](http://ma-zhe.cn/wp-content/uploads/maven/Maven%E5%AE%9E%E6%88%98-%E7%AC%AC%E4%B8%83%E7%AF%87%EF%BC%88Maven%E4%B8%AD%E7%9A%84%E4%BE%9D%E8%B5%96%E5%86%B2%E7%AA%81%EF%BC%89.resources/D3E69763-2463-4861-9748-86A4965EDD8C.png)

我们看上图所示，IDEA中已经不会提示依赖冲突了，因为当前传递性依赖中只有一个版本的`fastjson`依赖，所以也就不会提示依赖冲突了，这就是Maven中提供排除依赖的目的。

* * *

我们通过上面的配置发现Maven在排除依赖的时候，是不需要我们指定排除的版本的。有人知道这是为什么吗？这是因为我们传递性依赖所传递的版本已经确定了，因为它会在依赖包中指定我们依赖的版本，所以在排除时，就不需要我们指定版本了，因为一定只有一个版本。



# 八、Maven 中的依赖查询

在之前的内容中我们介绍了两种查看Maven中依赖关系的方法。一个是通过`External Libraries`查看。另外一个就是通过`Show Deagram ...`查看，但不管以上是哪种方法，本质都是依赖IDEA查看的。实际上Maven提供了项目中查看依赖的命令，可以帮助我们非常方便的查询项目的依赖的情况。下面我们分别介绍一下。

一、mvn dependency:list
---------------------

使用上述命令可以列出我们项目所有的依赖包。我们还是以上一篇的中的项目为例子，通过上面的命令查看一下项目的所有依赖包。

1、mazhe-maven-mall

*   pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>midai-mazhe-maven</artifactId>
        <groupId>cn.ma-zhe</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mazhe-maven-mall</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-payment</artifactId>
            <version>1.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>com.alibaba</groupId>
                    <artifactId>fastjson</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>cn.ma-zhe</groupId>
            <artifactId>mazhe-maven-order</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

我们进入到这个项目的根目录中，然后执行下面命令：

```
mvn dependency:list
```

*   输出

```
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< cn.ma-zhe:mazhe-maven-mall >---------------------
[INFO] Building mazhe-maven-mall 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- dependency:3.6.0:list (default-cli) @ mazhe-maven-mall ---
[INFO] 
[INFO] The following files have been resolved:
[INFO]    cn.ma-zhe:mazhe-maven-payment:jar:1.0-SNAPSHOT:compile
[INFO]    cn.ma-zhe:mazhe-maven-order:jar:1.0-SNAPSHOT:compile
[INFO]    cn.ma-zhe:mazhe-maven-user:jar:1.0-SNAPSHOT:compile
[INFO]    com.alibaba:fastjson:jar:2.0.35:compile
[INFO]    com.alibaba.fastjson2:fastjson2-extension:jar:2.0.35:compile
[INFO]    com.alibaba.fastjson2:fastjson2:jar:2.0.35:compile
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.516 s
[INFO] Finished at: 2023-07-24T22:40:06+08:00
[INFO] ------------------------------------------------------------------------

```

我们看上面的日志输出了我们这个项目中所有依赖包的信息，并且还包括传递性依赖的信息。这就是`mvn dependency:list`命令的作用。

二、mvn dependency:tree
---------------------

下面我们看第二个可以查看项目依赖的命令。此命令和上面的命令一样，也可以列出我们项目中所有的依赖包，不同之处就是这个命令可以按照树的方式展示，方便我们查看依赖关系。而`mvn dependency:list`命令则是同一级显示的。

*   输出

```
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< cn.ma-zhe:mazhe-maven-mall >---------------------
[INFO] Building mazhe-maven-mall 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- dependency:3.6.0:tree (default-cli) @ mazhe-maven-mall ---
[INFO] cn.ma-zhe:mazhe-maven-mall:jar:1.0-SNAPSHOT
[INFO] +- cn.ma-zhe:mazhe-maven-payment:jar:1.0-SNAPSHOT:compile
[INFO] \- cn.ma-zhe:mazhe-maven-order:jar:1.0-SNAPSHOT:compile
[INFO]    \- cn.ma-zhe:mazhe-maven-user:jar:1.0-SNAPSHOT:compile
[INFO]       \- com.alibaba:fastjson:jar:2.0.35:compile
[INFO]          \- com.alibaba.fastjson2:fastjson2-extension:jar:2.0.35:compile
[INFO]             \- com.alibaba.fastjson2:fastjson2:jar:2.0.35:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.375 s
[INFO] Finished at: 2023-07-24T22:47:15+08:00
[INFO] ------------------------------------------------------------------------
```

我们看上面的日志，依然输出了我们项目中所有的依赖信息，并且它是按照树形方式展示的，这样可以让我们非常方便的查看出传递性依赖的关系。通过上面日志我们知道，因为项目依赖了`mazhe-maven-order`，又因为该项目又依赖了`mazhe-maven-user`。所以Maven才会将项目`mazhe-maven-mall`依赖的`fastjson`引入到项目中。

三、mvn dependency:analyze
------------------------

下面我们介绍一下非常重要的命令，使用此命令Maven可以分析当前项目的依赖，我们可以通过分析的结果来判断出来我们项目引入了哪些依赖，并且知道哪些依赖我们并没有使用。我们继续在项目的根目录中执行上面的命令查看一下日志输出

*   日志

```
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< cn.ma-zhe:mazhe-maven-mall >---------------------
[INFO] Building mazhe-maven-mall 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] >>> dependency:3.6.0:analyze (default-cli) > test-compile @ mazhe-maven-mall >>>
[INFO] 
[INFO] --- resources:3.3.1:resources (default-resources) @ mazhe-maven-mall ---
[INFO] skip non existing resourceDirectory /Users/md/Documents/midai-mazhe-maven/mazhe-maven-mall/src/main/resources
[INFO] 
[INFO] --- compiler:3.11.0:compile (default-compile) @ mazhe-maven-mall ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- resources:3.3.1:testResources (default-testResources) @ mazhe-maven-mall ---
[INFO] skip non existing resourceDirectory /Users/md/Documents/midai-mazhe-maven/mazhe-maven-mall/src/test/resources
[INFO] 
[INFO] --- compiler:3.11.0:testCompile (default-testCompile) @ mazhe-maven-mall ---
[INFO] No sources to compile
[INFO] 
[INFO] <<< dependency:3.6.0:analyze (default-cli) < test-compile @ mazhe-maven-mall <<<
[INFO] 
[INFO] 
[INFO] --- dependency:3.6.0:analyze (default-cli) @ mazhe-maven-mall ---
[WARNING] Unused declared dependencies found:
[WARNING]    cn.ma-zhe:mazhe-maven-payment:jar:1.0-SNAPSHOT:compile
[WARNING]    cn.ma-zhe:mazhe-maven-order:jar:1.0-SNAPSHOT:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.634 s
[INFO] Finished at: 2023-07-24T22:58:45+08:00
[INFO] ------------------------------------------------------------------------
```

我们看上面的日志，发现有警告的信息，为什么会出现警告信息呢，这是因为我们虽然引入了上述的依赖，但是我们并没有在项目中使用。下面我们在项目中调用一下`MazheMavenOrder`类。

*   MazheMavenOrder

```
public class MazheMavenOrder {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven Order!");
    }
}
```

*   MazheMavenMall

```
public class MazheMavenMall {
    public static void main(String[] args) {
        System.out.println("Hello World Mazhe Maven Mall!");
        MazheMavenOrder.main(args);
    }
}
```

然后我们在执行一下`mvn dependency:analyze`命令，在观察一下日志输出。

*   日志

```
[WARNING] Unused declared dependencies found:
[WARNING]    cn.ma-zhe:mazhe-maven-payment:jar:1.0-SNAPSHOT:compile
```

我们看日志输出，这次就只会显示`mazhe-maven-payment`项目未使用了，因为`mazhe-maven-order`项目已经在项目中引用了，Maven就不会提示这个错误警告了。

 
