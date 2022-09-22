# 常用的测试工具对比

LoadRunner：性能稳定，压测结果及细粒度大，可以自定义脚本进行压力，但是太过于重大，功能比较繁多

Apache Benchmark（ab）：单接口压测最方便。模拟多线程并发请求，ab命令对发出负责的计算机要求很低，既不会占用很多CPU，也不会占用太多的内存，但却给目标的服务器造成巨大的负载，简单DDOS攻击等

Webbench：webbench首先fork出多个子进程，每个紫禁城都循环做web访问测试，子进程把访问的结果通过pipe告诉父进程，父进程做最终的统计结果

Jmeter：开源免费，功能强大，在还联网公司普遍使用

- 压测不同的协议和应用

  - Web-》HTTP，HTTPS（java、NodeJS、PHP、ASP.NET,...）
  - SOAP / REST Webservices
  - FTP
  - Database via JDBC
  - LDAP 轻量目录访问协议
  - Message-oriented middleware (MOM) via JMS
  - Mail - SMTP(S)，POP3(s) and IMAP(S)
  - TCP等等

- 使用从场景及优点：功能测试、压力测试、分布式压力测试、纯Java开发、上手容易性能高、提供测试数据分析、各种报表数据图形展示

  



# Apache Benchmark

Apache Benchmark（简称ab） 是Apache安装包中自带的压力测试工具 ，简单易用。单接口压测最方便。

模拟多线程并发请求,ab命令对发出负载的计算机要求很低，既不会占用很多CPU，也不会占用太多的内存，但却会给目标服务器造成巨大的负载, 简单DDOS攻击等

## 1、ab 的安装

```shell
[root@CentOS7 ~]# yum install -y httpd-tools

# 查看是否安装成功，查看版本号
[root@CentOS7 ~]# ab -V
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
```

> windows可以去http://httpd.apache.org/download.cgi下载，或者从命令行安装

## 2、ab 测试的命令参数

命令： ab 或者 ab -help

```shell
# 查看帮助文档
[root@CentOS7 ~]# ab -help
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
    -b windowsize   Size of TCP send/receive buffer, in bytes
    -B address      Address to bind to when making outgoing connections
    -p postfile     File containing data to POST. Remember also to set -T
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header to use for POST/PUT data, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
    -v verbosity    How much troubleshooting info to print
    -w              Print out results in HTML tables
    -i              Use HEAD instead of GET
    -x attributes   String to insert as table attributes
    -y attributes   String to insert as tr attributes
    -z attributes   String to insert as td or th attributes
    -C attribute    Add cookie, eg. 'Apache=1234'. (repeatable)
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'
                    Inserted after all normal header lines. (repeatable)
    -A attribute    Add Basic WWW Authentication, the attributes
                    are a colon separated username and password.
    -P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password.
    -X proxy:port   Proxyserver and port number to use
    -V              Print version number and exit
    -k              Use HTTP KeepAlive feature
    -d              Do not show percentiles served table.
    -S              Do not show confidence estimators and warnings.
    -q              Do not show progress when doing more than 150 requests
    -g filename     Output collected data to gnuplot format file.
    -e filename     Output CSV file with percentages served
    -r              Don't exit on socket receive errors.
    -h              Display usage information (this message)
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    -f protocol     Specify SSL/TLS protocol
                    (SSL3, TLS1, TLS1.1, TLS1.2 or ALL)
```

压测命令：ab [可选的参数选项] 需要进行压力测试的url

参数说明：

```shell
此外，我们再根据上面的用法介绍界面来详细了解每个参数选项的作用。

-n	即requests，用于指定压力测试总共的执行次数。
-c	即concurrency，用于指定的并发数。
-t	即timelimit，等待响应的最大时间(单位：秒)。
-b	即windowsize，TCP发送/接收的缓冲大小(单位：字节)。
-p	即postfile，发送POST请求时需要上传的文件，此外还必须设置-T参数。
-u	即putfile，发送PUT请求时需要上传的文件，此外还必须设置-T参数。
-T	即content-type，用于设置Content-Type请求头信息，例如：application/x-www-form-urlencoded，默认值为text/plain。
-v	即verbosity，指定打印帮助信息的冗余级别。
-w	以HTML表格形式打印结果。
-i	使用HEAD请求代替GET请求。
-x	插入字符串作为table标签的属性。
-y	插入字符串作为tr标签的属性。
-z	插入字符串作为td标签的属性。
-C	添加cookie信息，例如："Apache=1234"(可以重复该参数选项以添加多个)。
-H	添加任意的请求头，例如："Accept-Encoding: gzip"，请求头将会添加在现有的多个请求头之后(可以重复该参数选项以添加多个)。
-A	添加一个基本的网络认证信息，用户名和密码之间用英文冒号隔开。
-P	添加一个基本的代理认证信息，用户名和密码之间用英文冒号隔开。
-X	指定使用的和端口号，例如:"126.10.10.3:88"。
-V	打印版本号并退出。
-k	使用HTTP的KeepAlive特性。
-d	不显示百分比。
-S	不显示预估和警告信息。
-g	输出结果信息到gnuplot格式的文件中。
-e	输出结果信息到CSV格式的文件中。
-r	指定接收到错误信息时不退出程序。
-h	显示用法信息，其实就是ab -help。
```

常用的ab的参数说明：

```shell

-n  需要执行的请求次数
-c  并发的数量
-t   等待返回的最长时间
-b  TCP收发缓冲区的大小，单位（byte）
-p 使用post (同时需要定义-T参数)
-u 使用put (同时需要定义-T参数)
-T content-type, 例如application/x-www-form-urlencoded, 默认为text/plain
-w 把结果打印在html的表格里
-x  表格的属性
-y  tr行属性
-z  td列属性
-C 设置cookie 例如Apach=1234
-H header行， 例如Accept-Encoding:gzip
-k 是否标示位HTTP Keep Alive 
```



## 3、ab 的使用

模拟并发请求100次，总共请求10000次。命令模板：ab -c 100 -n 10000 待测试网站的完整路径

参数解释：

1. -n:进行http请求的总个数
2. -c:请求的client个数，也就是请求并发数
3. 统计qps：qps即每秒并发数，request per second

返回内容解释：

```shell
Server Software:        nginx/1.10.2 (服务器软件名称及版本信息)
Server Hostname:        192.168.1.106(服务器主机名)
Server Port:            80 (服务器端口)

Document Path:          /index1.html. (供测试的URL路径)
Document Length:        3721 bytes (供测试的URL返回的文档大小)

Concurrency Level:      1000 (并发数)
Time taken for tests:   2.327 seconds (压力测试消耗的总时间)
Complete requests:      5000 (的总次数)
Failed requests:        688 (失败的请求数)
Write errors:           0 (网络连接写入错误数)
Total transferred:      17402975 bytes (传输的总数据量)
HTML transferred:       16275725 bytes (HTML文档的总数据量)
Requests per second:    2148.98 [#/sec] (mean) (平均每秒的请求数) 这个是非常重要的参数数值，服务器的吞吐量 
Time per request:       465.338 [ms] (mean) (所有并发用户(这里是1000)都请求一次的平均时间)
Time  request:       0.247 [ms] (mean, across all concurrent requests) (单个用户请求一次的平均时间)
Transfer rate:          7304.41 [Kbytes/sec] received 每秒获取的数据长度 (传输速率，单位：KB/s)
...
Percentage of the requests served within a certain time (ms)
  50%    347  ## 50%的请求在347ms内返回 
  66%    401  ## 60%的请求在401ms内返回 
  75%    431
  80%    516
  90%    600
  95%    846
  98%   1571
  99%   1593
 100%   1619 (longest request)
```



## 4、压力测试

操作示例 1：

```shell
# GET请求
[root@CentOS7 ~]# ab -c 100 -n 5000 http://localhost/index
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking ourchem.com (be patient)
Completed 500 requests
Completed 1000 requests
Completed 1500 requests
Completed 2000 requests
Completed 2500 requests
Completed 3000 requests
Completed 3500 requests
Completed 4000 requests
Completed 4500 requests
Completed 5000 requests
Finished 5000 requests


Server Software:        nginx/1.16.1
Server Hostname:        localhost
Server Port:            80

Document Path:          /index
Document Length:        69316 bytes

Concurrency Level:      100                                         # 请求的并发数
Time taken for tests:   102.862 seconds
Complete requests:      5000                                        # 完成了5000个请求
Failed requests:        0
Write errors:           0
Total transferred:      348555000 bytes
HTML transferred:       346580000 bytes
Requests per second:    48.61 [#/sec] (mean)
Time per request:       2057.243 [ms] (mean)
Time per request:       20.572 [ms] (mean, across all concurrent requests)
Transfer rate:          3309.15 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        3    3   0.6      3      25
Processing:    36 2034 1833.7   1412   15617
Waiting:       30 2028 1833.7   1406   15611
Total:         39 2038 1833.7   1415   15620

Percentage of the requests served within a certain time (ms)
  50%   1415
  66%   2084
  75%   2673
  80%   3230
  90%   4453
  95%   5756
  98%   7338
  99%   8482
 100%  15620 (longest request)

```

操作示例 2：

```shell
# POST请求
# 创建一个postfile文本，然后在文本中添加json字符串内容，例如：
{
  "username": "大鹏",
  "age": 18,
  "description": "屌丝男士",
  "aspirations": "拜托屌丝"
}
[root@CentOS7 ~]# ab -n 100 -c 100 -p postfile -T 'application/json' http://localhost/insert
```

## 5、注意事项

1. 测试机与被测试机要分开
2. 不要对线上的服务器做压力测试
3. 观察测试工具ab所在机器，以及被测试的前端机的CPU、内存、网络等都不超过最高限度的75%

## 6、文章推荐

1. [使用ab进行页面的压力测试(可参考解释)](https://www.cnblogs.com/yjf512/archive/2011/05/24/2055723.html)
2. [apache性能测试工具ab使用详解](http://www.jb51.net/article/59469.htm)
3. 本文转载：https://blog.csdn.net/u011415782/article/details/78501799





# Jmeter

> 参考：https://www.cnblogs.com/chenyanbin/p/13332068.html