# 一、问题背景

项目中免不了需要读取文件，如果文件用绝对路径读取，就需要配置或写死路径，非常不便。如果我们读取类路径上的文件，就不会这么麻烦。

比如要读取的文件位于类路径下：`java/main/resources/myfile.txt`，这个文件在项目打成Jar包的时候，也会压缩在里头，非常方便。

用如下两种方式获得文件路径或输入流：

```java
URL url = this.getClass().getResource("resourcePath");
InputStream inputStream = this.getClass().getResourceAsStream("resourcePath");
```

上面2个方法**就是从类路径上读取文件的**，但有个大坑，就是你在IDEA里调试得好好的，但打成Jar包，启动项目后发现可能会无法读取到这个文件。详细如下

- 打成jar包在linux启动，**但是读取输入流为null**
- 打成jar包在windows启动，能读取到输入流 (很奇葩，linux不能windows居然能)
- 在 IDEA 里：能读取到这个输入流



# 二、问题复现与源码

## 1、问题复现

开发需求读取一个：resources/txt下的name.txt，开发环境运行了很正常，部署到线上环境就滚犊子了。代码报错信息：

```java
java.io.FileNotFoundException: file:/Users/lsx/Downloads/springboot-out-properties-0.0.1-SNAPSHOT.jar!/BOOT-INF/classes!/txt/name.txt (No such file or directory)
```

这里报错的意思是很明显了，也就是说name.txt文件找不到。在本地环境运行的时候可以看到地址是这个：

```
/Users/lsx/Downloads/springboot-out-properties-0.0.1-SNAPSHOT.jar!/BOOT-INF/classes!/txt/name.txt
```

这个路径是一个有效的路径。这又是一个我本地明明没问题，到线上就有问题，锅是甩不出去了。



## 2、源码

看下实现的源码，在resources/txt下有一个name.txt，读取name.txt的代码：

```java
import java.io.*;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.util.*;

public class NameGenerator {

    private static final String NOUNS_FILE = "/txt/name.txt";
    private static List<String> nouns = new ArrayList<>();

    static {
        readLinesByPath();
    }

    public static String generate() {
        Random random = new Random();
        String noun = nouns.get(random.nextInt(nouns.size()));
        return noun;
    }

    private static void readLinesByPath() {
        URL url = NameGenerator.class.getResource(NOUNS_FILE);
        File file = new File(url.getPath());
        System.out.println(file.getPath());
        nouns.addAll(readLinesByFile(file.getPath()));
        System.out.println("名称个数："+ nouns.size());
    }

    private static List<String> readLinesByFile(String filePath) {
        List<String> lines = new ArrayList<>();
        try (InputStream is = new FileInputStream(filePath);
             InputStreamReader isr = new InputStreamReader(is, StandardCharsets.UTF_8);
             BufferedReader reader = new BufferedReader(isr)) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return lines;
    }
}
```

这里核心实现的代码如下，通过文件流FileInutStream读取了文件。：

```java
InputStream is = new FileInputStream(filePath)
```



# 三、问题分析与解决

## 1、问题分析

JAVA是一门静态语言，先编译再运行也就是先将Java文件编译成class文件，然后在用虚拟机来执行class文件的。SpringBoot 在编译打包后会生成target目录，class文件，资源文件还有Jar包都会被放在这个目录下。如下图所示：

```
├─target
    ├─classes
       │  application.yml
       ├─com
       │  └─example
       └─txt
          └─name.txt
```

其中所有的class文件以及资源文件都放在了classes文件夹中。在本地运行时：`NameGenerator.class.getResource("/").getPath()`读取到的路径是`/Users/lsx/Documents/workspace/workspace-idea/springboot-out-properties/target/classes/`，classes文件夹所在的路径也就是我们的classpath路径 。

而通过jar包来运行时，上面的代码读取的是jar的绝对路径，而jar是一个压缩包，直接读取其包内的绝对路径是有问题的。也就是会报上面的错误。



## 2、问题解决

既然不能通过路径的方式来获取jar中文件，那么该通过何种方式来获取呢？这里有两种写法。

1. 解决方式一：

   ```java
   InputStream is = NameGenerator.class.getResourceAsStream(NOUNS_FILE);
   ```

2. 解决方式二：

   ```java
   InputStream is = new ClassPathResource(NOUNS_FILE).getInputStream();
   ```

这样的写法部署到线上的话，就可以正常的运行了。最终的源码如下：

```java
import org.springframework.core.io.ClassPathResource;

import java.io.*;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.util.*;

public class NameGenerator {
    private static final String NOUNS_FILE = "/txt/name.txt";
    private static List<String> nouns = new ArrayList<>();

    static {
        readLinesByPath();
    }

    public static String generate() {
        Random random = new Random();
        String noun = nouns.get(random.nextInt(nouns.size()));
        return noun;
    }


    private static void readLinesByPath() {
        nouns.addAll(readLinesByFile());
        System.out.println("名称个数："+ nouns.size());
    }


    private static List<String> readLinesByFile() {
        List<String> lines = new ArrayList<>();
        // 写法一：NameGenerator.class.getResourceAsStream(NOUNS_FILE)
        // 写法二：new ClassPathResource(NOUNS_FILE).getInputStream()
        try (InputStream is = new ClassPathResource(NOUNS_FILE).getInputStream();
             InputStreamReader isr = new InputStreamReader(is, StandardCharsets.UTF_8);
             BufferedReader reader = new BufferedReader(isr)) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return lines;
    }
}
```



# 四、问题解决总结

在读取resouces/下的文件的时候，不要使用new FileInutStream()的方式，不然就会出现本地开发环境是正常的，线上环境报错的问题，尽量使用java提供的读取流的方式，具体的实现有两种：

1. 写法一：

   ```java
   InputStream is = NameGenerator.class.getResourceAsStream(NOUNS_FILE);
   ```

2. 写法二（强烈推荐）：

   ```java
   InputStream is = new ClassPathResource(NOUNS_FILE).getInputStream();
   ```

> ```java
> @PostConstruct
> public void getResources() throws IOException {
>        String      fileName = "/templates/name.txt";
>        String      fileName2 = "templates/name.txt";
>        InputStream is1      = this.getClass().getClassLoader().getResourceAsStream(fileName);
>        InputStream is2      = this.getClass().getResourceAsStream(fileName);
>        InputStream is3      = GetResources.class.getClassLoader().getResourceAsStream(fileName);
>        InputStream is4      = GetResources.class.getResourceAsStream(fileName);
>        InputStream is5      = new ClassPathResource(fileName).getInputStream();
>        System.out.println(is1);
>        System.out.println(is2);
>        System.out.println(is3);
>        System.out.println(is4);
>        System.out.println(is5);
> }
> ```
>
>  在上面的5中方式中，1、3、5使用的是相对路径，2、4使用的是根路径。另外，在很多文章中有提到3、4不推荐使用，理由是GetResources.class.getClassLoader()会返回空，使用3、4会报NPE，具体原因倒没有看到解释，有路过旁友可以补充链接或说明。



