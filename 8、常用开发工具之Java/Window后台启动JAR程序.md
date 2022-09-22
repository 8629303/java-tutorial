## Windows 后台执行Jar

我们都知道Linux下可用命令：nohup optjdk1.8.0_131binjava -jar xxx.jar & 来后台执行JAR

如果是Windows环境，要如何在后台执行呢

新建一个bat文件，输入：

1. 普通的启动： java -jar xxx.jar
2. 后台启动：javaw -jar xxx.jar

系统需要退出的时候，可以查看任务管理器。从命令行中找到你启动的项目（javaw.exe），进而杀掉。

## 引入外部日志配置文件启动

引入外部日志配置文件启动： java(javaw) -jar -Dlogging.config=xxxlogback.xml test.jar

## SpringBoot打包为瘦jar时启动

java(javaw) -Dloader.path=lib -jar xxx.jar application.properties

-Dloader.path：指定外部jar包目录