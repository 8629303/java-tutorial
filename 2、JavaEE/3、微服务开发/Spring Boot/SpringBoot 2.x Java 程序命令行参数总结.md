> 来源：Java程序命令行参数总结：https://blog.csdn.net/blueheart20/article/details/103019782

# 1、基于命令行输入参数

1、测试程序如下：

```java
import java.util.Arrays;
public class Main {
    public static void main(String[] args) {
        System.out.println("System args: " + System.getenv("feature.flag") + ", " + System.getProperty("feature"));
        System.out.println("Program args:" + Arrays.toString(args));
    }
}
```

2、基于命令行的输入：

```bash
java -Dfeature=module1 -Dfeature.flag=true -Xms128M -Xmx256M com.x.y,z.Main first second third
```

输出结果为：

```
System args: true, module1
Program args:[first, second, third]
```

从上述示例中可知：

1. `-Dfeature=module1 -Dfeature.flag=true`，设置了系统属性信息，被正确地解析出来。系统属性的设置位于Java程序之前。
2. `first second third`，作为Java程序的args参数，需要放在程序之后才可以正确被解析。
3. `-Xms128 -Xmx256M`： 为设置JVM的参数，这里设置了其对内存的大小。

3、如果是Spring Boot应用的话，则通过两个‘- - ’作为命令行参数的前缀。，可以在命令行设置：

```bash
--server.port=9091 --spring.profiles.active=dev
```



4、基于命令行输入系统属性：-D：表示其为系统属性

```bash
java -Dfeature=module1 org.spb.data.Main
```



5、JVM参数设置：JVM参数是设置JVM的设置，其遵守JVM设置的参数类型。

```bash
java -Xms128M -Xmx256M org.spb.data.Main
```



# 2、环境变量 vs 系统属性

当程序中需要使用与操作系统相关的变量（例如：文件分隔符、换行符）时，Java 提供了 System 类的静态方法 getenv() 和 getProperty() 用于返回系统相关的变量与属性，getenv()方法返回的变量大多于系统相关，getProperty()方法返回的变量大多与 Java 程序有关。

系统属性和环境变量都是名称与值之间的映射。两种机制都能用来将用户定义的信息传递给 Java 进程。环境变量产生更多的全局效应，因为它们不仅对Java子进程可见，而且对于定义它们的进程的所有子进程都是可见的。在不同的操作系统上，它们的语义有细微的差别，比如，不区分大小写。因此环境变量更可能有意料不到的副作用。程序中尽可能使用系统属性。环境变量应该在需要全局效应的时候使用，或者在外部系统接口要求使用环境变量时使用（比如 PATH）

从上述示例可知，系统属性由于其与程序紧密相关，则可以基于命令行进行设置。但是，环境变量则无法通过程序进行设置，需要基于IDE和环境变量进行设置。

Java 命令行的帮助信息：

```bash
$ java -h
用法：java [options] <主类> [args...]
           （执行类）
   或  java [options] -jar <jar 文件> [args...]
           （执行 jar 文件）
   或  java [options] -m <模块>[/<主类>] [args...]
       java [options] --module <模块>[/<主类>] [args...]
           （执行模块中的主类）
   或  java [options] <源文件> [args]
           （执行单个源文件程序）

 将主类、源文件、-jar <jar 文件>、-m 或
 --module <模块>/<主类> 后的参数作为参数
 传递到主类。

 其中，选项包括：

    -cp <目录和 zip/jar 文件的类搜索路径>
    -classpath <目录和 zip/jar 文件的类搜索路径>
    --class-path <目录和 zip/jar 文件的类搜索路径>
                  使用 ; 分隔的, 用于搜索类文件的目录, JAR 档案
                  和 ZIP 档案列表。
    -p <模块路径>
    --module-path <模块路径>...
                  用 ; 分隔的目录列表, 每个目录
                  都是一个包含模块的目录。
    --upgrade-module-path <模块路径>...
                  用 ; 分隔的目录列表, 每个目录
                  都是一个包含模块的目录, 这些模块
                  用于替换运行时映像中的可升级模块
    --add-modules <模块名称>[,<模块名称>...]
                  除了初始模块之外要解析的根模块。
                  <模块名称> 还可以为 ALL-DEFAULT, ALL-SYSTEM,
                  ALL-MODULE-PATH.
    --list-modules
                  列出可观察模块并退出
    -d <module name>
    --describe-module <模块名称>
                  描述模块并退出
    --dry-run     创建 VM 并加载主类, 但不执行 main 方法。
                  此 --dry-run 选项对于验证诸如
                  模块系统配置这样的命令行选项可能非常有用。
    --validate-modules
                  验证所有模块并退出
                  --validate-modules 选项对于查找
                  模块路径中模块的冲突及其他错误可能非常有用。
    -D<名称>=<值>
                  设置系统属性
    -verbose:[class|module|gc|jni]
                  启用详细输出
    -version      将产品版本输出到错误流并退出
    --version     将产品版本输出到输出流并退出
    -showversion  将产品版本输出到错误流并继续
    --show-version
                  将产品版本输出到输出流并继续
    --show-module-resolution
                  在启动过程中显示模块解析输出
    -? -h -help
                  将此帮助消息输出到错误流
    --help        将此帮助消息输出到输出流
    -X            将额外选项的帮助输出到错误流
    --help-extra  将额外选项的帮助输出到输出流
    -ea[:<程序包名称>...|:<类名>]
    -enableassertions[:<程序包名称>...|:<类名>]
                  按指定的粒度启用断言
    -da[:<程序包名称>...|:<类名>]
    -disableassertions[:<程序包名称>...|:<类名>]
                  按指定的粒度禁用断言
    -esa | -enablesystemassertions
                  启用系统断言
    -dsa | -disablesystemassertions
                  禁用系统断言
    -agentlib:<库名>[=<选项>]
                  加载本机代理库 <库名>, 例如 -agentlib:jdwp
                  另请参阅 -agentlib:jdwp=help
    -agentpath:<路径名>[=<选项>]
                  按完整路径名加载本机代理库
    -javaagent:<jar 路径>[=<选项>]
                  加载 Java 编程语言代理, 请参阅 java.lang.instrument
    -splash:<图像路径>
                  使用指定的图像显示启动屏幕
                  自动支持和使用 HiDPI 缩放图像
                  (如果可用)。应始终将未缩放的图像文件名 (例如, image.ext)
                  作为参数传递给 -splash 选项。
                  将自动选取提供的最合适的缩放
                  图像。
                  有关详细信息, 请参阅 SplashScreen API 文档
    @argument 文件
                  一个或多个包含选项的参数文件
    -disable-@files
                  阻止进一步扩展参数文件
    --enable-preview
                  允许类依赖于此发行版的预览功能
要为长选项指定参数, 可以使用 --<名称>=<值> 或
--<名称> <值>。
```

> `-D<名称>=<值>`就是设置系统属性信息。



# 3、Maven 运行 SpringBoot

1、程序代码如下：

```java
@Slf4j
@SpringBootApplication
public class AppRunnerApplication {
    public static void main(String[] args) throws InterruptedException {
        SpringApplication.run(AppRunnerApplication.class, args);
        // Thread.currentThread().join();
        log.info("args:{}", Arrays.toString(args));
        log.info("feature:" + System.getProperty("feature"));
        log.info("feature.flag:" + System.getenv("feature.flag"));
    }
}
```

2、基于命令行的输入：

```bash
mvn spring-boot:run -Dmaven.test.skip=true -Pdev -Dspring-boot.run.arguments="first seond third" -Dspring-boot.run.jvmArguments="-Dfeature=module11"
```

- -Dfeature=module11设置系统属性信息。
- arguments：设置Java 程序main的入口参数

3、对于环境变量，则需要设置pom.xml文件中的plugin配置项：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <systemPropertyVariables>
                    <feature>moduleXXX</feature>
                </systemPropertyVariables>
                <environmentVariables>
                    <feature.flag>true</feature.flag>
                </environmentVariables>
            </configuration>
        </plugin>
    </plugins>
</build>
```

运行结果为：

```shell
2019-11-11 23:08:22.764  INFO 18717 --- [  restartedMain] o.s.d.apprunner.cmdline.AppLineRunner1   : App LineRunner1:first seond third
2019-11-11 23:08:22.765  INFO 18717 --- [  restartedMain] o.s.d.apprunner.cmdline.AppLineRunner2   : App LineRunner2:[first seond third]
2019-11-11 23:08:22.766  INFO 18717 --- [  restartedMain] o.s.data.apprunner.AppRunnerApplication  : args:[first seond third]
2019-11-11 23:08:22.766  INFO 18717 --- [  restartedMain] o.s.data.apprunner.AppRunnerApplication  : feature:module11
2019-11-11 23:08:22.766  INFO 18717 --- [  restartedMain] o.s.data.apprunner.AppRunnerApplication  : feature.flag:true
```

4、如果在命令行不进行jvmArguments的参数设置：

```bash
mvn spring-boot:run -Dmaven.test.skip=true -Pdev -Dspring-boot.run.arguments=“first seond third”
```

输出结果信息如下：

```shell
2019-11-11 23:11:55.399  INFO 18757 --- [  restartedMain] o.s.d.apprunner.cmdline.AppLineRunner2   : App LineRunner2:[first seond third]
2019-11-11 23:11:55.400  INFO 18757 --- [  restartedMain] o.s.data.apprunner.AppRunnerApplication  : args:[first seond third]
2019-11-11 23:11:55.400  INFO 18757 --- [  restartedMain] o.s.data.apprunner.AppRunnerApplication  : feature:moduleXXX
2019-11-11 23:11:55.400  INFO 18757 --- [  restartedMain] o.s.data.apprunner.AppRunnerApplication  : feature.flag:true
```

