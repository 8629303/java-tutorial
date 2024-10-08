> 转载与参考：
>
> 1. Linux中curl详解：https://blog.csdn.net/lemon_TT/article/details/129101461
> 2. curl命令大全：https://www.cnblogs.com/cangqiongbingchen/p/10180535.html
> 3. CURL 发送POST请求：https://www.cnblogs.com/fan-gx/p/12321351.html
> 4. curl 命令详解（超详细）：https://blog.csdn.net/m0_51504545/article/details/123278429

# Curl 的使用

## 1、URL访问

1、访问一个网页时，可以使用`curl`命令后加上要访问的网址（我们就看到所访问网址的页面源码）：

```bash
[root@CentOS7 ~]# curl itbilu.com 
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.12.2</center>
</body>
</html>
```

2、重定向跟踪：在上面示例中，页面使用了[301](https://itbilu.com/nodejs/core/4yMyt38M.html)重定向，这时我们可以添加`-L`参数来跟踪URL重定向（这样会返回页面所有HTML）：

```bash
$ curl -L itbilu.com
```

3、页面保存：如果需要将页面源码保存到本地，可以使用`-o`参数：

```bash
$ curl -o 文件名称 itbilu.com

[root@CentOS7 ~]# curl -o xxx.html itbilu.com
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   185  100   185    0     0   3038      0 --:--:-- --:--:-- --:--:--  3083
[root@CentOS7 ~]# ll
-rw-r--r-- 1 root root  185 Oct 14 10:11 xxx.html
```

4、查看头信息：如果需要查看访问页面的可以添加`-i`或`--include`参数：

```bash
$ curl -i itbilu.com
```

```bash
[root@CentOS7 ~]# curl -i itbilu.com
HTTP/1.1 301 Moved Permanently
Server: nginx/1.12.2
Date: Thu, 14 Oct 2021 02:12:26 GMT
Content-Type: text/html
Content-Length: 185
Connection: keep-alive
Location: https://itbilu.com/

<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.12.2</center>
</body>
</html>
```

添加`-i`参数后，页面响应头会和页面源码（响应体）一块返回。如果只想查看响应头，可以使用`-I`或`--head`参数：

```bash
$ curl -I itbilu.com
HTTP/1.1 301 Moved Permanently
Server: nginx/1.6.2
Date: Sun, 25 Jun 2017 02:03:45 GMT
Content-Type: text/html
Content-Length: 184
Connection: keep-alive
Location: https://itbilu.com/
```

 

## 2、表单提交

通过Form表单，可以将Web页面的表单数据提交到服务端。提交表单时，可以使用GET或POST提交方法。

Curl同样支持表单数据提交，也可以使用GET或POST提交方法。

1、GET数据提交，当全用GET表单数据提交时，提交数据会被附加到请求URL的后面。类型如下：

```bash
$ curl 'https://itbilu.com/?keyword=linux&page=3'
```

使用`curl`进行`GET`数据提交时，也可以直接把提交数据添加到URL后面：

```bash
curl https://itbilu.com/?keyword=linux&page=3
```



2、POST数据提交，Curl使用POST提交表单数据时，除了`-X`参数指定请求方法外，还要使用 `-d` 或 `--data`参数添加提交数据：

参数内容：

1. -X：请求协议
2. -H：请求头
3. -d：POST body内容

```bash
$ curl -X POST --data 'keyword=linux' itbilu.com
```

 ```bash
 curl -H "Content-Type: application/json" -X POST -d '{"user_id": "123", "coin":100, "success":1, "msg":"OK!" }' "https://jsonplaceholder.typicode.com/posts"
 ```

使用`-d`参数以后，HTTP 请求会自动加上标头`Content-Type : application/x-www-form-urlencoded`。并且会自动将请求转为 POST 方法，因此可以省略`-X POST`。

`-d`参数可以读取本地文本文件的数据，向服务器发送。

```shell
$ curl -d '@data.txt' https://jsonplaceholder.typicode.com/posts
```





## 3、其它HTTP请求方法

目前为止，我们使用`GET`和`POST`两种HTTP请求。`curl`支持所有[HTTP请求方法](https://itbilu.com/other/relate/EkwKysXIl.html)，只要通过`-X`参数指定即可。

1、使用`DELETE`请求：

```bash
$ curl -X DELETE itbilu.com/examlple.html
```

2、使用`PUT`请求，并指定请求数据：

```bash
$ curl -X PUT --data 'keyword=linux' itbilu.com
```

 

## 4、 文件上传

Curl支持文件上传，上传文件时使用`-T`或`--upload-file`参数：

```bash
$ curl -T ./index.html www.uploadhttp.com/receive.cgi
```

 

## 5、HTTPS支持

对于使用了`SSL/TLS`加密的HTTPS协议，可以使用`curl`直接访问：

```bash
$ curl https://itbilu.com
```

如果使用的本地`ssl证书`认证的HTTPS，可以通过`-E`或`--cert`参数指定本地证书：

```bash
$ curl -E mycert.pem https:/itbilu.com
```

 

## 6、添加请求头

有时在进行HTTP请求时，需要自定义请求头。在Curl中，可以通过`-H`或`--header`参数来指定请求头。多次使用`-H`或`--header`参数可指定多个请求头。

1、指定`Content-Type`及`Authorization`请求头：

```bash
$ curl -H 'Content-Type:application/json' -H 'Authorization: bearer eyJhbGciOiJIUzI1NiJ9' itbilu.com
```

 

## 7、Cookie支持

[Cookie](https://itbilu.com/other/relate/4J4n8fIPe.html)是一种常用的保持服务端会话信息的方法，Crul也支持使用Cookie。

可以通过`--cookie`参数指定发送请求时的`Cookie`值，也可以通过`-b [文件名]`来指定一个存储了`Cookie`值的本地文件：

```bash
$ curl -b stored_cookies_in_file itbilu.com
```

`Cookie`值可能会被服务器所返回的值所修改，并应用于下次HTTP请求。这时，可以能过`-c`参数指定存储服务器返回`Cookie`值的存储文件：

```bash
$ curl -b cookies.txt -c newcookies.txt itbilu.com
```

 除以上用法外，Curl还可以设置用户代理（客户端）信息、使用代理服务器、指定认证用户名／密码等。详见：Curl语法及选项

```bash
curl --silent -H "Host: www.test.com" "192.168.0.1/xxx/xxx/t.php"
curl "http://www.test.com/LiveUserCount.ac " -x 127.0.0.1:1080
```



# Curl 语法及选项

Curl语法结构如下：

```bash
curl [options...] <url>
```

 参数选项：`curl`（`7.29.0`）所支持的选项（`options`）参数如下：

```bash
在以下选项中，(H) 表示仅适用 HTTP/HTTPS ，(F) 表示仅适用于 FTP
    --anyauth       选择 "any" 认证方法 (H)
-a, --append        添加要上传的文件 (F/SFTP)
    --basic         使用HTTP基础认证（Basic Authentication）(H)
    --cacert FILE   CA 证书，用于每次请求认证 (SSL)
    --capath DIR    CA 证书目录 (SSL)
-E, --cert CERT[:PASSWD] 客户端证书文件及密码 (SSL)
    --cert-type TYPE 证书文件类型 (DER/PEM/ENG) (SSL)
    --ciphers LIST  SSL 秘钥 (SSL)
    --compressed    请求压缩 (使用 deflate 或 gzip)
-K, --config FILE   指定配置文件
    --connect-timeout SECONDS  连接超时设置
-C, --continue-at OFFSET  断点续转
-b, --cookie STRING/FILE  Cookies字符串或读取Cookies的文件位置 (H)
-c, --cookie-jar FILE  操作结束后，要写入 Cookies 的文件位置 (H)
    --create-dirs   创建必要的本地目录层次结构
    --crlf          在上传时将 LF 转写为 CRLF
    --crlfile FILE  从指定的文件获得PEM格式CRL列表
-d, --data DATA     HTTP POST 数据 (H)
    --data-ascii DATA  ASCII 编码 HTTP POST 数据 (H)
    --data-binary DATA  binary 编码 HTTP POST 数据 (H)
    --data-urlencode DATA  url 编码 HTTP POST 数据 (H)
    --delegation STRING GSS-API 委托权限
    --digest        使用数字身份验证 (H)
    --disable-eprt  禁止使用 EPRT 或 LPRT (F)
    --disable-epsv  禁止使用 EPSV (F)
-D, --dump-header FILE  将头信息写入指定的文件
    --egd-file FILE  为随机数据设置EGD socket路径(SSL)
    --engine ENGINGE  加密引擎 (SSL). "--engine list" 指定列表
-f, --fail          连接失败时不显示HTTP错误信息 (H)
-F, --form CONTENT  模拟 HTTP 表单数据提交（multipart POST） (H)
    --form-string STRING  模拟 HTTP 表单数据提交 (H)
    --ftp-account DATA  帐户数据提交 (F)
    --ftp-alternative-to-user COMMAND  指定替换 "USER [name]" 的字符串 (F)
    --ftp-create-dirs  如果不存在则创建远程目录 (F)
    --ftp-method [MULTICWD/NOCWD/SINGLECWD] 控制 CWD (F)
    --ftp-pasv      使用 PASV/EPSV 替换 PORT (F)
-P, --ftp-port ADR  使用指定 PORT 及地址替换 PASV (F)
    --ftp-skip-pasv-ip 跳过 PASV 的IP地址 (F)
    --ftp-pret      在 PASV 之前发送 PRET (drftpd) (F)
    --ftp-ssl-ccc   在认证之后发送 CCC (F)
    --ftp-ssl-ccc-mode ACTIVE/PASSIVE  设置 CCC 模式 (F)
    --ftp-ssl-control ftp 登录时需要 SSL/TLS (F)
-G, --get           使用 HTTP GET 方法发送 -d 数据  (H)
-g, --globoff       禁用的 URL 队列 及范围使用 {} 和 []
-H, --header LINE   要发送到服务端的自定义请求头 (H)
-I, --head          仅显示响应文档头
-h, --help          显示帮助
-0, --http1.0       使用 HTTP 1.0 (H)
    --ignore-content-length  忽略 HTTP Content-Length 头
-i, --include       在输出中包含协议头 (H/F)
-k, --insecure      允许连接到 SSL 站点，而不使用证书 (H)
    --interface INTERFACE  指定网络接口／地址
-4, --ipv4          将域名解析为 IPv4 地址
-6, --ipv6          将域名解析为 IPv6 地址
-j, --junk-session-cookies 读取文件中但忽略会话cookie (H)
    --keepalive-time SECONDS  keepalive 包间隔
    --key KEY       私钥文件名 (SSL/SSH)
    --key-type TYPE 私钥文件类型 (DER/PEM/ENG) (SSL)
    --krb LEVEL     启用指定安全级别的 Kerberos (F)
    --libcurl FILE  命令的libcurl等价代码
    --limit-rate RATE  限制传输速度
-l, --list-only    只列出FTP目录的名称 (F)
    --local-port RANGE  强制使用的本地端口号
-L, --location      跟踪重定向 (H)
    --location-trusted 类似 --location 并发送验证信息到其它主机 (H)
-M, --manual        显示全手动
    --mail-from FROM  从这个地址发送邮件
    --mail-rcpt TO  发送邮件到这个接收人(s)
    --mail-auth AUTH  原始电子邮件的起始地址
    --max-filesize BYTES  下载的最大文件大小 (H/F)
    --max-redirs NUM  最大重定向数 (H)
-m, --max-time SECONDS  允许的最多传输时间
    --metalink      处理指定的URL上的XML文件
    --negotiate     使用 HTTP Negotiate 认证 (H)
-n, --netrc         必须从 .netrc 文件读取用户名和密码
    --netrc-optional 使用 .netrc 或 URL; 将重写 -n 参数
    --netrc-file FILE  设置要使用的 netrc 文件名
-N, --no-buffer     禁用输出流的缓存
    --no-keepalive  禁用 connection 的 keepalive
    --no-sessionid  禁止重复使用 SSL session-ID (SSL)
    --noproxy       不使用代理的主机列表
    --ntlm          使用 HTTP NTLM 认证 (H)
-o, --output FILE   将输出写入文件，而非 stdout
    --pass PASS     传递给私钥的短语 (SSL/SSH)
    --post301       在 301 重定向后不要切换为 GET 请求 (H)
    --post302       在 302 重定向后不要切换为 GET 请求 (H)
    --post303       在 303 重定向后不要切换为 GET 请求 (H)
-#, --progress-bar  以进度条显示传输进度
    --proto PROTOCOLS  启用/禁用 指定的协议
    --proto-redir PROTOCOLS  在重定向上 启用/禁用 指定的协议
-x, --proxy [PROTOCOL://]HOST[:PORT] 在指定的端口上使用代理
    --proxy-anyauth 在代理上使用 "any" 认证方法 (H)
    --proxy-basic   在代理上使用 Basic 认证  (H)
    --proxy-digest  在代理上使用 Digest 认证 (H)
    --proxy-negotiate 在代理上使用 Negotiate 认证 (H)
    --proxy-ntlm    在代理上使用 NTLM 认证 (H)
-U, --proxy-user USER[:PASSWORD]  代理用户名及密码
     --proxy1.0 HOST[:PORT]  在指定的端口上使用 HTTP/1.0 代理
-p, --proxytunnel   使用HTTP代理 (用于 CONNECT)
    --pubkey KEY    公钥文件名 (SSH)
-Q, --quote CMD     在传输开始前向服务器发送命令 (F/SFTP)
    --random-file FILE  读取随机数据的文件 (SSL)
-r, --range RANGE   仅检索范围内的字节
    --raw           使用原始HTTP传输，而不使用编码 (H)
-e, --referer       Referer URL (H)
-J, --remote-header-name 从远程文件读取头信息 (H)
-O, --remote-name   将输出写入远程文件
    --remote-name-all 使用所有URL的远程文件名
-R, --remote-time   将远程文件的时间设置在本地输出上
-X, --request COMMAND  使用指定的请求命令
    --resolve HOST:PORT:ADDRESS  将 HOST:PORT 强制解析到 ADDRESS
    --retry NUM   出现问题时的重试次数
    --retry-delay SECONDS 重试时的延时时长
    --retry-max-time SECONDS  仅在指定时间段内重试
-S, --show-error    显示错误. 在选项 -s 中，当 curl 出现错误时将显示
-s, --silent        Silent模式。不输出任务内容
    --socks4 HOST[:PORT]  在指定的 host + port 上使用 SOCKS4 代理
    --socks4a HOST[:PORT]  在指定的 host + port 上使用 SOCKSa 代理
    --socks5 HOST[:PORT]  在指定的 host + port 上使用 SOCKS5 代理
    --socks5-hostname HOST[:PORT] SOCKS5 代理，指定用户名、密码
    --socks5-gssapi-service NAME  为gssapi使用SOCKS5代理服务名称 
    --socks5-gssapi-nec  与NEC Socks5服务器兼容
-Y, --speed-limit RATE  在指定限速时间之后停止传输
-y, --speed-time SECONDS  指定时间之后触发限速. 默认 30
    --ssl           尝试 SSL/TLS (FTP, IMAP, POP3, SMTP)
    --ssl-reqd      需要 SSL/TLS (FTP, IMAP, POP3, SMTP)
-2, --sslv2         使用 SSLv2 (SSL)
-3, --sslv3         使用 SSLv3 (SSL)
    --ssl-allow-beast 允许的安全漏洞，提高互操作性(SSL)
    --stderr FILE   重定向 stderr 的文件位置. - means stdout
    --tcp-nodelay   使用 TCP_NODELAY 选项
-t, --telnet-option OPT=VAL  设置 telnet 选项
     --tftp-blksize VALUE  设备 TFTP BLKSIZE 选项 (必须 >512)
-z, --time-cond TIME  基于时间条件的传输
-1, --tlsv1         使用 => TLSv1 (SSL)
    --tlsv1.0       使用 TLSv1.0 (SSL)
    --tlsv1.1       使用 TLSv1.1 (SSL)
    --tlsv1.2       使用 TLSv1.2 (SSL)
    --trace FILE    将 debug 信息写入指定的文件
    --trace-ascii FILE  类似 --trace 但使用16进度输出
    --trace-time    向 trace/verbose 输出添加时间戳
    --tr-encoding   请求压缩传输编码 (H)
-T, --upload-file FILE  将文件传输（上传）到指定位置
    --url URL       指定所使用的 URL
-B, --use-ascii     使用 ASCII/text 传输
-u, --user USER[:PASSWORD]  指定服务器认证用户名、密码
    --tlsuser USER  TLS 用户名
    --tlspassword STRING TLS 密码
    --tlsauthtype STRING  TLS 认证类型 (默认 SRP)
    --unix-socket FILE    通过这个 UNIX socket 域连接
-A, --user-agent STRING  要发送到服务器的 User-Agent (H)
-v, --verbose       显示详细操作信息
-V, --version       显示版本号并退出
-w, --write-out FORMAT  完成后输出什么
    --xattr        将元数据存储在扩展文件属性中
-q                 .curlrc 如果作为第一个参数无效
```

 

