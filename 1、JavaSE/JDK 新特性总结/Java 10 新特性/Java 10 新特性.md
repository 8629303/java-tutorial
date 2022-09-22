# Java 10 新特性之基于时间的发布版本控制

从 Java 10 开始，Oracle 为 Java 版本引入了严格的基于时间的版本控制模型。现在 Java 将在每六个月后发布一个主要版本。Java 10 于 2018 年 3 月发布，以后所有主要版本都计划在未来几年的 3 月和 9 月发布。发布进一步分为三大类。

- 功能发布： 功能发布包含特定于语言的功能、JVM 功能、新/改进的 API、删除/弃用 API。这些功能发布的时间是固定的，并且对要包含在特定版本中的功能没有限制。如果正在开发的功能不是最新版本的一部分，那么它将在下一个版本中计划。
- 更新版本： 更新版本包括错误修复、安全问题修复、回归修复等。每个更新版本计划在 1 月、4 月、7 月和 10 月的每个季度发布。在宣布下一个功能版本之前，每个功能版本将收到两个更新版本。
- 长期支持 (LTS) 版本：从 2018 年 9 月开始，每三年发布一次长期支持版本。Oracle 将在未来三年内为此版本提供支持和更新。此版本主要面向在生产部署中使用 Java 的企业。

JDK 版本格式，现在的版本遵循以下格式。

```shell
$FEATURE.$INTERIM.$UPDATE.$PATCH
```

- $FEATURE ： 此数字表示主要功能发布，每次功能发布后都会增加 1。对于 Java 10，它是 10。
- $INTERIM ： 此数字表示包含错误修复和增强功能的任何非功能、非更新版本。此版本没有任何不兼容的更改、任何 API 删除或对标准 API 的更改。功能发布，将此计数器设为 0。
- $UPDATE ：此数字表示在功能发布后完成的更新发布。例如，2018 年 4 月的 Java 更新版本是 JDK 10.0.1，2018 年 7 月的更新版本是 JDK 10.0.2 等等。
- $PATCH ： 此数字表示仅在紧急情况下促进关键问题时才会增加的任何紧急发布。

Java10 版本控制的示例，以下程序显示了 JAVA 10 的版本控制细节。

```java
public class Tester {
    public static void main(String[] args) {
        Runtime.Version version = Runtime.version();
        System.out.printf(" feature: %s%n interim: %s%n update: %s%n patch: %s%n",
                          version.feature(), 
                          version.interim(), 
                          version.update(), 
                          version.patch());
    }
}

// 输出内容
feature: 10
interim: 0
update: 2
patch: 0
```



# Java 10 新特性之局部变量 var

 将前端思想 var 关键字引入 java 后段，自动检测所属于类型，一种情况除外，不能为 null，因为不能判断具体类型，会报异常。

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;

public class Tester {
    public static void main(String[] args) {
        var number = 10;
        var str = "i like java";
        var list = new ArrayList<>();
        var map = new HashMap<>();
        var set = new HashSet<>();

        list.add("test var is list");
        map.put("1", "test var is map");
        set.add("test var is set");
        System.out.println(number);
        System.out.println(str);
        System.out.println(list.toString());
        System.out.println(map.toString());
        System.out.println(set.toString());
    }
}

// 输出结果
10
i like java
[test var is list]
{1=test var is map}
[test var is set]
```

var 可以用于任何类型，基本类型，集合，以及实体类等等都可以，省略了我们在前面写入具体的类型，我们尽管 var，让它自己去判断就好了。



# Java 10 新特性之新的API 和选项

## 1、创建不可修改集合的 API - copyOf()

在 java.util.List、java.util.Set、java.util.Map 新增加了一个静态方法 copyOf，这些方法按照其迭代顺序返回一个不可修改的列表、集合或映射包含了给定的元素的集合。但是如果将返回后的集合继续修改，那么报异常。

```java
import java.util.ArrayList;
import java.util.List;

public class Tester {
    public static void main(String[] args) {
        var list = new ArrayList<>();
        list.add("first");
        list.add("second");
        list.add("third");
        var result = List.copyOf(list);
        System.out.println(result.toString());
    }
}

// 输出内容
[first, second, third]
```



## 2、ByteArrayOutputStream 新增 API

java.io.ByteArrayOutputStream，重载 toString () 方法，通过使用指定的字符集编码字节，将缓冲区的内容转换为字符串，以前是默认没有参数，现在加了一个编码的字符方法。

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.util.Scanner;

public class Tester {
    public static void main(String[] args) throws Exception {
        String str = "我喜欢java";
        ByteArrayInputStream bis = new ByteArrayInputStream(str.getBytes("utf-8"));

        ByteArrayOutputStream bos = new ByteArrayOutputStream();

        int c;
        while ((c = bis.read()) != -1) {
            bos.write(c);
        }
        // bos.toString() 默认的使用的UTF-8编码
        System.out.println(bostoString ());
    }
}

// 输出内容
我喜欢java
```

这个是我们正常的格式，但是当我们修改代码，str.getBytes ("gbk") 的时候，创建的 txt 为乱码，那么我们使用 toString () 的新增方法，替换 gbk。

```
��ϲ��java
```



## 3、PrintStream、PrintWriter 新增 API

java.io.PrintStream,Java.io.PrintWriter，这两个类都有三个新的构造方法，他们需要而外的参数 charset

```java
import java.io.PrintWriter;

public class Tester {
    public static void main(String[] args) throws Exception {
        String str = "我也特别喜欢java";
        var pri  = new PrintWriter("D:\\test_out.txt");
        pri.println(str);
        pri.flush();
        pri.close();
    }
}

// 文件中输出
我也特别喜欢java
```

但是我们这个 txt 默认的都是 utf-8 格式的，如果突然改成 gbk 的话，格式会不匹配出现乱码，毕竟 txt 文件格式要和输出的格式一致才可以，这是我们可以使用一下。 我们直接在后面添加一个格式参数就可以直接解决啦

```java
var pri = new PrintWriter("D:\\test_out.txt","gbk");
```



## 4、Reader:transferTo 新增成员方法

java.io.Reader:transferTo 从这个 Reader 中读取所有字符串，并按照所读取的顺序将字符串写入给指定的 Writer

```java
import java.io.*;

public class Tester {
    public static void main(String[] args) throws Exception {
        var reader = new BufferedReader(new InputStreamReader(new FileInputStream("D:\\test.txt"), "utf-8"));
        var p = new PrintWriter("D:\\test_out.txt");
        reader.transferTo(p);
        p.flush();
        p.close();
        reader.close();
    }
}

// 最后test.txt中的内容就输出到test_out.txt文件中了
```



## 5、Formatter、Scanner 新增构造方法

 java.util.Formatter、java.util.Scanner 新增三个构造方法，除了其他参数之外，都需要一个 charset 参数，将 11.txt 内容替换成这样

```java
import java.io.FileInputStream;
import java.util.Scanner;

public class Tester {
    public static void main(String[] args) throws Exception{
        var scan = new Scanner(new FileInputStream("D:\\test.txt"), "utf-8");
        scan.useDelimiter(" |,");
        while (scan.hasNext()) {
            System.out.println(scan.next());
        }
    }
}
```

scan.useDelimiter ("|,"); 这是空格或者‘,’ 都要分割，如下结果为：

```
hello
word
!!!

```



# Java 10 新特性之删除的功能和选项

# Java 10 新特性之已弃用的功能和选项

# Java 10 新特性之

# Java 10 新特性之