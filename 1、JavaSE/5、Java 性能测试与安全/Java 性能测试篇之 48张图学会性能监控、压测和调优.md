> 作者：Java中文社群；来源：https://mp.weixin.qq.com/s/IWthSkypENdnJmDitAy2Aw

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640.png)



## 一、何为压力测试

### 1、大白话解释

- 性能压测是什么：就是考察当前`软件`和`硬件`环境下，系统所能承受的`最大负荷`，并帮助找出系统的`瓶颈`所在。
- 性能压测的目的：为了系统在线上的`处理能力`和`稳定性`维持在一个`标准范围`内，做到知己知彼，百战不殆。还可以发现内存泄漏、并发与同步的问题。

### 2、性能指标

- RepsonseTime - RT：响应时间，用户从客户端发起一个请求开始计算，到客户端接收到服务端的响应结束，整个过程所耗费的时间。
- Hits Per Second - HPS：用户每秒点击次数，也就是每秒向后台发送的请求次数。
- QPS：系统每秒内处理查询的次数。
- MaxRT：最大响应时间，指用户发出请求到服务端返回响应的最大时间。
- MiniRT：最少响应时间，指用户发出请求到服务端返回响应的最少时间。
- 90%响应时间：将所有用户的响应时间进行升序排序，取 90 % 的位置。
- 性能测试关注点：
  - 吞吐量：每秒钟系统能处理的请求数、任务数。
  - 响应时间：服务处理一个请求或一个任务的耗时。
  - 错误率：一批请求中结果出过错的请求所占比例。




## 二、Jmeter 压测工具

### 1、Jmeter 工具

- 下载和安装 Jmeter 工具：下载地址：https://jmeter.apache.org/download_jmeter.cgi

我下载的版本是 apache-jmeter-5.3

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-16682591383383.png)

- 运行 JMeter 程序：打开批处理文件：\apache-jmeter-5.3\bin\jmeter.bat。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-16682591528956.png)

- 添加线程组，如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-16683151029043.png)

- 1s 内启动 200 个线程，循环次数 100 次。2 w 个请求。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-16682626529873.png)

- 测试 HTTP 请求，如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-16682626578946.png)

配置要测试的协议、服务器地址、端口号。配置信息如下：

- 协议：使用 http 协议。

- 服务器名称或 IP: www.baidu.com (只是为了演示)。

- 端口号：80 端口。


如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-16682626838559.png)

- 添加查看结果树、汇总报告和聚合报告。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826269664712.png)

- 开始压力测试：点击播放按钮就开始启动了。注意启动之前需要先设置线程组的参数配置和 HTTP 请求的配置。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826272468915.png)

- 查看每个请求结果，如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826274316218.png)

- 查看汇总报告。

  主要关心平均值和吞吐量。

  200 个线程，每个线程调用 100 次，总共 2 w 次，可以看到下图中表格中的样本列也是 2 w，请求所耗费的时间是 151 ms，吞吐量是 880 个请求每秒。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826275579921.png)

- 查看聚合报告。

  - 主要看中位数和90%百分位，


  - **中位数**是 59 ms，说明大部分请求的响应时间是 59 ms。


  - **90 % 的请求** 都是在 271 ms 以内响应完成的。

  - **异常 0.41%** 说明 2 w 个请求中有 82 个请求异常（20000 * 0.0041 = 82 ）。


  - **吞吐量 880.2/sec** 说明百度这个网站每秒能处理 880 个请求，性能一般（可能跟本地机器性能有关）。

如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826276814324.png)

- 查看汇总图：查看汇总图时，需要先勾选想要查看的信息，如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826277874027.png)

然后查看图形汇总：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826278706430.png)

可以看到勾选的几列在图表中是用不同颜色表示的，比如绿色的柱状条就是 90 % 百分位。

我们来测试下 PassJava（佳必过）的管理后台的性能，吞吐量接近 2000/s。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826286845933.png)



## 三、性能监控之 jconsole

jconsole 和 jvisualvm 是 Java JDK 的两个小工具，用来监控内存泄漏、跟踪垃圾回收、执行时的内存情况、对 CPU 进行分析、线程的分析。都可以通过命令行启动，而且可以监控本地和远程应用。而 jvisualvm 是升级版的 jconsole。我们先来看下 jconsole 的使用。

首先用 cmd 命令行的方式启动 jconsole。

### 1、启动 jconsole

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826287862836.png)



### 2、选择监控哪个应用

然后选择 passjava 项目的 question 服务。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826291904439.png)

对应的就是下面这个微服务：passjava-question。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826292730542.png)对应 passjava-question 微服务

### 3、概览

从监控界面上有 6 个菜单，首先看到的是概览功能，上面有堆内存使用量、线程数、类的使用情况、CPU 占用率，都是用趋势图来表示的，能很方便的看出当前性能的概览。注意：这些监控都是实时的。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826295645945.png)概览

### 4、内存

下面是内存的使用情况，可以从下图中看到有个下拉框，里面可以选择不同的内存维度，然后下面的图标和柱状图也会跟着选择的维度而展示不同。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826296753348.png)

### 5、线程

下面是线程的使用情况，可以看到线程峰值和活动线程的总数量，目前看到的峰值是59，活动线程数是 57。下半部分可以看到具体是哪些线程，以及线程的堆栈信息，非常详细。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826301890851.png)

### 6、类

下面是类的加载和卸载情况，已加载类总数是 10679，而已卸载的类是 1 个，所以当前已加装当前类的总数是 10679 - 1 = 10678 个。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826302938254.png)

### 7、VM 概要

我们再来看下VM（虚拟机）的情况。如下图所示，可以看到虚拟机情况，线程、类、堆的概要信息，以及 VM 的参数，是不是很方便呀~

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826305810157.png)

### 8、MBean 信息

接下来我们来看下 MBean 信息。对于 MBean，可能很多同学不知道是啥，下面做个解释：

MBean就是一种规范的JavaBean，通过集成和实现一套标准的Bean接口，这种叫MBean。MBean可以用来干嘛？就是可以有一套JDK级别的对外的服务接口。比如，你写了一个JVM允许状态辅助查询的Bean,你希望别人下载一个Jconsole就可以看到你写的杰作。那你就可以考虑用MBean规范来实现。很多垃圾收集器算法Bean就这么干的（说的就是这个类sun.management.MemoryImpl）。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826307761760.png)



## 四、性能监控之 jvisualvm

jvisualvm 比 jconsole 更强大，界面展示的信息更丰富。还可以安装插件，太赞了。

### 1、启动 jvisualvm 和概述

启动方式和 jconsole 一样，也是通过 cmd 命令行启动。还是选择 passjava-question 微服务，然后选择第一个菜单栏：概述。可以看到 JVM 的版本，启动参数等信息。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826310066463.png)

### 2、监视

监视 CPU、堆、类、线程的情况。整体显示的效果比 jconsole 更美观。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826310458066.png)

### 3、线程

再来查下线程的情况。可以看到有 5 种状态的线程：

- 运行：正在运行的线程。
- 休眠：休眠状态的线程。
- 等待：等待执行的线程。
- 驻留：线程里面的空闲线程。
- 监视：阻塞的线程，正在等待锁。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826312113769.png)

### 4、抽样器

另外我们也可以抽样器对 CPU 或内存进行抽样。如下图所示，对内存进行抽样。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826313664572.png)

### 5、插件的使用

> **安装Visual GC 插件**

安装步骤：工具->插件->可用插件->Visual GC->安装。安装完成后，重启就可以使用插件功能了。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826315691675.png)

安装完成后，就可以看到

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826318571981.png)

下图是实时监控垃圾回收的情况。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826316690278.png)



## 五、对网关的性能测试

现在我想对 Passjava 系统的 question 微服务的接口进行一个压测，该如何进行呢？

首先我们来看下 passjava 的架构是怎么样的，如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826325986184.png)

客户端分为手机端和 PC 端，http 请求先经过 API Gateway，然后再转发到 question 微服务。其中涉及到了中间件：Gateway 网关。

我们来对 Gateway 网关进行压力测试。网关的端口号是 8060，我们配置下 JMeter。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826326858487.png)

配置每秒发送 200 个请求，一直循环执行，直到手动停止压测。如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826327315590.png)

可以看下执行结果，吞吐量在 2422 个每秒，还是比较高的。

- 吞吐量：2422/s 。

- 90% 响应时间：142 毫秒。
- 99% 响应时间：281 毫秒。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826332169793.png)

我们再来看看垃圾回收的情况，Eden 区垃圾回收用时 2.7 s，用时太长了吧，看看这里怎么能优化下。

通常的优化方向是增大新生代堆内存配置。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826333458096.png)



## 六、对微服务的性能测试

根据上面的架构原理图，我们知道客户端请求都是经过 Gateway 转发了一次的，如果我们想单独看下微服务的性能该怎么测试呢？下面我来演示下如何测试 passjava-question 微服务的性能。

首先需要在 passjava-question 微服务中添加一个测试方法：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-166826335025199.png)

有两种方式测试这个 api 是否添加正确。

第一种：用 postman 测试下这个请求是否能正确响应，返回 “test” 则表示响应正确。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263380320105.png)

第二种：通过浏览器进行测试。浏览器地址栏输入以下链接后，回车，看下浏览器窗口是否显示 “test”，是则表示响应正确。

然后我们需要用 Jmeter 压测工具来测试这个微服务下的 api 的性能究竟如何。（单独压测微服务的 api的结果）

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263370458102.png)

- 吞吐量：3542/s 。

- 90% 响应时间：100 毫秒。

- 99% 响应时间：152 毫秒。



## 七、对网关+微服务的性能测试

如果我们想对这个整个请求链路进行性能测试该怎么做？

首先请求需要先经过网关，然后由网关转发到微服务。在之前的文章中，我已经将网关配置好了，所以要想通过网关转发到 test 请求，只需要对请求路径稍作修改即可，如下所示：

```
http://localhost:8060/api/question/v1/admin/question/test
```

然后在浏览器输入该网址，返回 “test” 即表示响应正确。

然后我们还是用 Jmeter 压测工具测试下 test api 的性能。测试结果如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263441215108.png)

从结果可以看到：

- 吞吐量：982/s 。

- 90% 响应时间：437 毫秒。

- 99% 响应时间：790毫秒。

这里做个横向对比：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263465291111.png)

说明微服务 api 经过网关转发一次后，性能至少下降了一半。可以得出结果：中间件越多，性能损失越大，大部分损失都是网络交互导致的。可以通过增强网络通信质量来减少网络的延迟。



## 八、对数据库查询进行优化

一般情况下，出现性能问题更多的是业务中查询数据库的耗时。接下来看下如何优化数据的查询。

下面是一个查询问题列表的 api：通过问题类型 type 字段过滤问题列表。api 路径如下：

```
http://localhost:11000/question/v1/admin/question/list?type=5
```

这个 api 的代码如下，很容易看懂（查询问题列表的 api）

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263481977114.png)

我们加些测试代码：统计查询数据库的耗时。如下所示（耗时统计）：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263488142117.png)

然后重启 passjava-question 服务，再次测试这个 api，耗时 43 ms

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263503330120.png)

怎么对查询进行优化呢？很容易想到加索引，我们来试下加在 question 表加索引后的效果。给 type 字段加上普通索引，如下图所示：

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668297452649132.png)

我们再来看下加了索引后的耗时情况：耗时 18 ms，确实比之前的 43 ms 快了很多。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263555639123.png)



## 九、优化垃圾回收

我们可以通过 jvisulavm工具查看垃圾回收的情况，Eden 区频繁发生 GC，短时间（1分钟）内共造成了 480 次 stop the world。另外从压测工具中也可以看到，吞吐量为 275/s。

原因是 Eden 区的内存分配得太小了，只有 32 M，我们来调大一点。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263564408126.png)



### 1、增大 Eden 区大小

通过在 IDEA 工具中配置以下参数，调整堆内存最大为 1024 M，新生代内存为 512 M。

```
-Xmx1024m -Xms1024m -Xmn512m
```

然后可以观察到在相当长的短时间（1分钟）内只进行了 92 次垃圾回收，说明垃圾回收的频率降低了。应用程序的性能也提升了。另外从压测工具中也可以看到，吞吐量为 347/s，吞吐量也有较大提升。

![图片](Java 性能测试篇之 48张图学会性能监控、压测和调优.assets/640-1668263590641129.png)



## 十、总结

本文通过压测工具 Jmeter 讲解压测如何实施，然后用性能监控工具 jconsole 和 jvisualvm 来监控 Java 应用程序的性能，以及如何用工具来优化开源项目 passjava 的性能，并且非常详细地介绍了每一步以及执行结果，通过对比的方式，更加清晰地知道如何做性能优化。

下面是对系统性能的常规优化手段：

- 中间件较多时，优化网络通信质量。
- 数据库查询耗时时，需要对查询进行优化，比如添加索引。
- 模板的渲染速度，可以通过设置模板缓存。
- 静态资源的获取，可以通过 Nginx 动静分离来解决。（下期再讲）
- 日志太多，需要减少不必要的打 log 操作。



参考资料 & 鸣谢：

- https://blog.csdn.net/u010833547/article/details/92806510
- https://www.bilibili.com/video/BV1np4y1C7Yf
- https://github.com/Jackson0714/PassJava-Platform
- www.passjava.cn