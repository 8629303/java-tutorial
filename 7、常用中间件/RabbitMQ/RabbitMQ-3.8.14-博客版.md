# RabbitMQ 安装及管理

## 01、Win10 安装

> 
>安装步骤：1.安装`Erlang`=》2.安装`RabbitMQ`=》3.激活`RabbitMQ's Management Plugin`可视化插件
> 
>RabbitMQ是采用Erlang语言开发的，所以系统环境必须提供Erlang环境
> Erlang和RabbitMQ版本的按照比较: https://www.rabbitmq.com/which-erlang.html

**1、安装Erlang**

![20210314201339](https://gitee.com/liusuixing/blog_images/raw/master/20210314201339.png)

- 官网下载地址：https://www.erlang.org/downloads
- **历史版本下载**：http://erlang.org/download/
- 选择`OTP 23.2 Windows 64-bit Binary File`
- 运行`otp_win64_23.2.exe`（默认next和install ）
- 设置环境变量：
  - ERLANG_HOME：D:\xx\erl-23.2（erlang安装路径）
  - Path追加：;%ERLANG_HOME%\bin
  - 打开CMD输入`erl`, 提示版本信息: `Eshell V11.1.4 (abort with ^G)`, 说明安装成功



**2、安装RabbitMQ**

- 官网下载地址：https://www.rabbitmq.com/install-windows.html
- **历史版本下载**：https://github.com/rabbitmq/rabbitmq-server/releases
- 选择`rabbitmq-server-3.8.14.exe`，然后运行安装RabbitMQ
- 设置环境变量：
  - RABBITMQ_SERVER：D:\rabbitmq\rabbitmq-server-3.8.14（安装路径）
  - Path追加：;%RABBITMQ_SERVER%\sbin
  - 命令行输入（出现如下信息说明安装成功并且启动）：rabbitmqctl status

```cmd
D:\Environment\RabbitMQ Server\rabbitmq_server-3.8.14\sbin>rabbitmqctl status
Status of node rabbit@LAPTOP-671C76TJ ...
 [1mRuntime [0m

OS PID: 7916
OS: Windows
Uptime (seconds): 2291
Is under maintenance?: false
RabbitMQ version: 3.8.14
Node name: rabbit@LAPTOP-671C76TJ
Erlang configuration: Erlang/OTP 23 [erts-11.1.4] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1]
Erlang processes: 473 used, 1048576 limit
Scheduler run queue: 1
Cluster heartbeat timeout (net_ticktime): 60

 [1mPlugins [0m

Enabled plugin file: c:/Users/lenovo/AppData/Roaming/RabbitMQ/enabled_plugins
Enabled plugins:

 * rabbitmq_management
 * amqp_client
 * rabbitmq_web_dispatch
 * cowboy
 * cowlib
 * rabbitmq_management_agent
// ........中间省略

Interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Interface: 0.0.0.0, port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Interface: [::], port: 15672, protocol: http, purpose: HTTP API
Interface: 0.0.0.0, port: 15672, protocol: http, purpose: HTTP API
```

- 访问：http://localhost:15672/，**发现无法访问，因为还未激活管理插件**



**3、激活RabbitMQ's Management Plugin可视化插件**

> 该插件为rabbitmq提供web管理界面，可以方便管理rabbitmq，登录地址：localhost:15672，用户名和密码都为guest

- 进入/sbin目录输入命令：`rabbitmq-plugins enable rabbitmq_management`
- 出现如下信息表示成功：

```cmd
D:\Environment\RabbitMQ Server\rabbitmq_server-3.8.14\sbin>rabbitmq-plugins enable rabbitmq_management
Enabling plugins on node rabbit@LAPTOP-671C76TJ:
rabbitmq_management
The following plugins have been configured:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@LAPTOP-671C76TJ...
The following plugins have been enabled:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch

started 3 plugins.
```

- 也能通过查看插件列表判断：**rabbitmq-plugins list**（标记为E*或e*的已经启用的插件）

```shell
C:\Users\lenovo>rabbitmq-plugins list
Listing plugins with pattern ".*" ...
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status: * = running on rabbit@LAPTOP-671C76TJ
 |/
[E*] rabbitmq_management               3.8.14
// ......省略
```

- 浏览器访问：http://localhost:15672/，账号密码：guest/guest。到此windows安装RabbitMQ已全部完成



> **说明：rabbitmq默认用户guest 默认情况只能在localhost本机下访问。如果想远程访问有2种办法：**
>
> 1、添加一个远程登录的用户
>
> ```shell
> # 新增用户
> rabbitmqctl add_user admin password
> # 设置用户分配操作权限
> rabbitmqctl set_user_tags admin administrator
> # 为用户添加资源权限
> rabbitmqctl.bat set_permissions -p / admin ".*" ".*" ".*"
> ```
>
>
> 2、修改默认配置文件（查看配置文件路径两种方式：）
> 2-1、rabbitmqctl status，查找Config files即可（advanced.config）
> 2-2、登录web管理界面：Overview=》Nodes=》Node Name=》Config file（advanced.config）
>
> ```shell
> [{rabbit, [{loopback_users, [guest]}]}].
> ```





**4、如上是通过*.exe方式安装的，所以自动帮我们安装好了服务启动和服务暂时删除等**

![20210418110505](https://gitee.com/liusuixing/blog_images/raw/master/20210418110505.png)

![20210418110803](https://gitee.com/liusuixing/blog_images/raw/master/20210418110803.png)



**5、如果是通过 *.zip 文件安装的话需要手动安装RabbitMQ服务**

5.1、以应用方式启动（如何没有install安装成windows服务）

```shell
# 前台启动
rabbitmq-server
# 后台启动
rabbitmq-server -detached
```

5.2、Rabbitmq服务的启动和关闭：（以服务方式启动，只有Windows能用）

```shell
# 安装服务：
rabbitmq-service install
# 删除服务：
rabbitmq-service remove
# 启动服务：
rabbitmq-service start
# 重启服务：
rabbitmq-service restart
# 停止服务：
rabbitmq-service stop
# 使服务有效：
rabbitmq-service enable 
# 使服务无效：
rabbitmq-service disable
# 查看帮助：
rabbitmq-service help
```

> 当rabbitmq-service install后默认服务是enable的，如果这时设置服务为disable的话，rabbitmq-service start就会报错。
> 当rabbitmq-service start正常启动服务之后，使用disable是没有效果的





## 02、Linux  安装

> Erlang和RabbitMQ版本的按照比较: https://www.rabbitmq.com/which-erlang.html
> RabbitMQ是采用Erlang语言开发的，所以系统环境必须提供Erlang环境，第一步就是安装Erlang
> 环境准备：CentOS7.x+ / Erlang
> Erlang下载地址：https://github.com/rabbitmq/erlang-rpm/releases
> RabbitMQ下载地址：https://github.com/rabbitmq/rabbitmq-server/releases

方式一：下载固定版本

```bash
## 1. 首先在Linux上进行一些软件的准备工作，yum下来一些基础的软件包
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz

## 2. 下载RabbitMQ所需软件包（本神在这里使用的是 RabbitMQ3.8.14）
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v23.2.7/erlang-23.2.7-1.el7.x86_64.rpm
wget http://repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-5.el7.lux.x86_64.rpm
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.14/rabbitmq-server-3.8.14-1.el7.noarch.rpm

## 3. 安装服务命令
rpm -ivh erlang-23.2.7-1.el7.x86_64.rpm
rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm
rpm -ivh rabbitmq-server-3.8.14-1.el7.noarch.rpm
```



**RPM方式安装，查看系统版本号**：

```shell
[root@CentOS7 ~]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.8.2003 (Core)
Release:	7.8.2003
Codename:	Core
```

1、 Erlang安装（下载地址：https://www.erlang-solutions.com/downloads/）

```shell
rpm -Uvh esl-erlang_23.2-1~centos~7_amd64.rpm
# 1.要将Erlang Solutions存储库（包括用于验证已签名软件包的公共密钥）添加到您的系统，请调用以下命令：
wget https://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm
rpm -Uvh erlang-solutions-2.0-1.noarch.rpm

# 2.安装erlang（安装的是最新版本，所以用该方式安装Rabbitmq需要查看是否能匹配到erlang）
yum install -y erlang

# 3.安装成功后测试
erl -v
```

2、安装socat（EL依赖该组件）

```shell
yum install -y socat
```

3、下载安装rabbitmq

```shell
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.13/rabbitmq-server-3.8.13-1.el8.noarch.rpm
rpm -Uvh rabbitmq-server-3.8.13-1.el8.noarch.rpm
```

4、启动rabbitmq服务

```shell
############服务方式启动############
# 启动服务
systemctl start rabbitmq-server
# 重启服务
systemctl restart rabbitmq-server
# 停止服务
systemctl stop rabbitmq-server
# 查看服务状态
systemctl status rabbitmq-server
# 开机启动服务
systemctl enable rabbitmq-server

############应用方式启动############
# 前台启动
rabbitmq-server
# 后台启动
rabbitmq-server -detached
```



5、默认情况下，rabbitmq是没有安装web端的客户端插件，需要安装才可以生效

```shell
# 安装RabbitMQ管理界面插件
rabbitmq-plugins enable rabbitmq_management

# 重启服务生效
systemctl restart rabbitmq-server
```

> 说明：rabbitmq有一个默认账号和密码是：`guest` 默认情况只能在localhost本机下访问，所以需要添加一个远程登录的用户。
> 一定要记住，在对应服务器(阿里云，腾讯云等)的安全组中开放`15672`的端口。



6、添加远程登录用户

```shell
# 新增用户
rabbitmqctl add_user admin password
# 设置用户分配操作权限
rabbitmqctl set_user_tags admin administrator
# 为用户添加资源权限
rabbitmqctl.bat set_permissions -p / admin ".*" ".*" ".*"
```

用户级别：

- 1、administrator 可以登录控制台、查看所有信息、可以对rabbitmq进行管理
- 2、monitoring 监控者 登录控制台，查看所有信息
- 3、policymaker 策略制定者 登录控制台,指定策略
- 4、managment 普通管理员 登录控制台



## 03、Docker 安装

> 虚拟化容器技术—docker 的安装
>
> ```SHELL
> # 1.yum 包更新到最新
> yum update
> # 2.安装需要的软件包，yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
> yum install -y yum-utils device-mapper-persistent-data lvm2
> # 3.设置yum源为阿里云
> yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
> # 4.安装docker
> yum install -y docker-ce
> # 5.安装后查看docker版本
> docker -v
> # 6.安装加速镜像
> sudo mkdir -p /etc/docker
> sudo tee /etc/docker/daemon.json <<-'EOF'
> {
>   "registry-mirrors": ["https://0wrdwnn6.mirror.aliyuncs.com"]
> }
> EOF
> # 7.刷新配置和重启服务
> sudo systemctl daemon-reload && sudo systemctl restart docker
> ```
>
> docker-compose 的安装
>
> ```shell
> # curl方式安装
> sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
> 
> # 授权
> sudo chmod +x /usr/local/bin/docker-compose
> 
> # 查看版本
> docker-compose --version
> 
> # 命令补齐安装
> sudo curl -L https://raw.githubusercontent.com/docker/compose/1.26.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
> ```



参考：1：https://www.rabbitmq.com/download.html、2：https://registry.hub.docker.com/_/rabbitmq/

1、dockere安装rabbitmq

```shell
# 获取rabbit镜像
docker pull rabbitmq:management

# 创建并运行容器（简单创建，默认用户名密码为：guest）
docker run -itd --name=myrabbit -p 15672:15672 rabbitmq:management

# 创建并运行容器（运行时设置用户和密码）
docker run -itd --name myrabbit \
 -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=admin \
 -p 15672:15672 \
 -p 5672:5672 \
 -p 25672:25672 \
 -p 61613:61613 \
 -p 1883:1883 \
 rabbitmq:management

# 查看日志
docker logs -f myrabbit
```



2、docker-compose安装rabbitmq

```shell
# 创建目录
$ mkdir -p /usr/local/docker-compose/rabbitmq
# 编写docker-compose.yml文件
$ vi /usr/local/docker-compose/rabbitmq/docker-compose.yml
# 后台启动
$ docker-compose up -d
# rabbitmq web端默认关闭的，手动开启
$ docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_management
# 或者
$ docker-compose exec rabbitmq rabbitmq-plugins enable rabbitmq_management
```

docker-compose.yml 文件内容如下：

```yaml
version: "3.1"
services: 
  rabbitmq: 
    image: daocloud.io/library/rabbitmq:management
    container_name: rabbitmq
    restart: always
    ports: 
      - 5672:5672
      - 15672:15672
    volumes: 
      - ./data:/var/lib/rabbitmq
```



## 04、角色分类

**1、none：**不能访问management plugin

**2、management：查看自己相关节点信息**

- 列出自己可以通过AMQP登入的虚拟机
- 查看自己的虚拟机节点 virtual hosts的queues,exchanges和bindings信息
- 查看和关闭自己的channels和connections
- 查看有关自己的虚拟机节点virtual hosts的统计信息。包括其他用户在这个节点virtual hosts中的活动信息。

**3、Policymaker**

- 包含management所有权限
- 查看和创建和删除自己的virtual hosts所属的policies和parameters信息。

**4、Monitoring**

- 包含management所有权限
- 罗列出所有的virtual hosts，包括不能登录的virtual hosts。
- 查看其他用户的connections和channels信息
- 查看节点级别的数据如clustering和memory使用情况
- 查看所有的virtual hosts的全局统计信息。

**5、Administrator**

- 最高权限
- 可以创建和删除virtual hosts
- 可以查看，创建和删除users
- 查看创建permisssions
- 关闭所有用户的connections



## 05、常用命令

> **插件的启用和关闭命令：rabbitmq-plugins**

1、list命令

```shell
# -v 显示所有插件的详情（详细）
# -m 仅仅只显示插件的名称 (简约)
# -E 仅仅只显示显式启用的插件
# -e 仅仅只显示显式、隐式启用的插件
# <pattern> 表示用于过滤插件名称表达式

# 查看插件列表,每行一个（标记为E*或e*的已经启用的插件，没标记的为未启用插件）
rabbitmq-plugins list

# 显示所有的插件，并且显示插件的版本号和描述信息
rabbitmq-plugins list -v

# 显示所有名称含有 "management" 的插件
rabbitmq-plugins list -v management

# 显示所有显示或者隐式启动的插件
rabbitmq-plugins list -e rabbit
```

2、enable & disable & set 命令

```shell
--offline 仅仅修改启动的插件文件
--online 将与正在运行的代理连接失败视为致命错误
<plugin> 一个或多个插件名称

# 启动指定插件：
rabbitmq-plugins enable plugin-name
# 关闭指定插件：
rabbitmq-plugins disable plugin-name

# 禁用所有插件
rabbitmq-plugins set
# 启用management插件和它所依赖的插件，禁用其他所有插件
rabbitmq-plugins set rabbitmq_management
```



> **管理平台操作命令：rabbitmqctl：**

**节点管理：**

```bash
# 查看节点状态
rabbitmqctl status

# 启动应用（在Erlang VM运行的情况下启动RabbitMQ应用）
rabbitmqctl start_app

# 关闭应用
rabbitmqctl stop_app

# 移除所有数据（要在 rabbitmqctl stop_app 之后使用）
rabbitmqctl reset

# 清理数据并且重启
rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl start_app
```





**用户管理：**

1. 用户列表和角色：`rabbitmqctl list_users`
2. 添加用户：`rabbitmqctl add_user <username> <password>`
3. 删除用户：`rabbitmqctl delete_user <username>`
4. 修改密码：`rabbitmqctl change_password <username> <newpass>`
5. 清除密码：`rabbitmqctl clear_password <username>`
6. 验证用户：`rabbitmqctl authenticate_user <username> <passWord>`
7. 设置用户角色：`rabbitmqctl set_user_tags <username> <tag1> <tag2>...`
   tag为：none、management、policymaker、monitoring、administrator
8. 删除用户所有角色：`rabbitmqctl set_user_tags <username>`
9. 设置用户权限：`rabbitmqctl set_permissions <-p vhostpath> <username> ".*" ".*" ".*"`
10. 清除用户权限：`rabbitmqctl clear_permissions <-p vhostpath> <username>`
11. 查看用户权限：`rabbitmqctl list_user_permissions <username>`
12. 查看指定虚拟主机上的所有用户的所有权限：`rabbitmqctl list_permissions [-p vhostpath]`

> 角色说明：
>
> - none：不能访问management plugin插件
>
> - management：用户可以访问management插件
> - policymaker：用户可以访问management插件，并管理他们有权访问的vhost的策略和参数。
> - monitoring：用户可以访问管理插件，查看所有连接和通道以及与节点相关的信息。
> - administrator：所有权限
>
> 权限类型：
>
> - Configure：创建与销毁资源
> - Write：写入资源
> - Read：读取资源

```shell
# 查看用户列表：
[root@CentOS7 ~]# rabbitmqctl list_users
Listing users ...
user	tags
user_admin	[administrator, monitoring]
guest	[administrator]

# 创建用户：
[root@CentOS7 ~]# rabbitmqctl add_user user_admin password
Adding user "user_admin" ...
Done. Don't forget to grant the user permissions to some virtual hosts! See 'rabbitmqctl help set_permissions' to learn more.

# 删除用户：
[root@CentOS7 ~]# rabbitmqctl delete_user user_admin
Deleting user "user_admin" ...

# 修改密码：
[root@CentOS7 ~]# rabbitmqctl change_password user_admin 123456
Changing password for user "user_admin" ...

# 清理密码：
[root@CentOS7 ~]# rabbitmqctl clear_password user_admin
Clearing password for user "user_admin" ...

# 验证账号密码
[root@CentOS7 ~]# rabbitmqctl authenticate_user user_admin 123456
Authenticating user "user_admin" ...
Success

# 设置用户角色
[root@CentOS7 ~]# rabbitmqctl set_user_tags user_admin administrator
Setting tags for user "user_admin" to [administrator] ...

# 删除用户所有角色
[root@CentOS7 ~]# rabbitmqctl set_user_tags user_admin
Setting tags for user "user_admin" to [] ...

# 设置用户权限：
[root@CentOS7 ~]# rabbitmqctl set_permissions -p / user_admin ".*" ".*" ".*"
Setting permissions for user "user_admin" in vhost "/" ...

# 清除用户权限：
[root@CentOS7 ~]# rabbitmqctl clear_permissions -p / user_admin
Clearing permissions for user "user_admin" in vhost "/" ...

# 查看用户权限
[root@CentOS7 ~]# rabbitmqctl list_user_permissions user_admin
Listing permissions for user "user_admin" ...
vhost	configure	write	read
/	    .*	        .*	    .*

# 查看指定虚拟主机上的所有用户的所有权限
[root@CentOS7 ~]# rabbitmqctl list_permissions -p /
Listing permissions for vhost "/" ...
user	     configure	write	read
user_admin	 .*	        .*	    .*
guest	     .*	        .*	    .*
```



**virtual_host管理管理：**

1. 列出所有虚拟主机：`rabbitmqctl list_vhosts [name,tracing]`
   name：罗列出所有虚拟机，tracing：表示是否使用了 RabbitMQ 的 trace 功能
2. 创建虚拟主机：`rabbitmqctl add_vhost [vhostpath]`
3. 删除虚拟主机：`rabbitmqctl delete_vhost [vhostpath]`

```shell
# 创建/test虚拟主机
[root@CentOS7 ~]# rabbitmqctl add_vhost /test
Adding vhost "/test" ...

# 删除/test虚拟主机:
[root@CentOS7 ~]# rabbitmqctl delete_vhost /test
Deleting vhost "/test" ...

# 列出所有虚拟主机
[root@CentOS7 ~]# rabbitmqctl list_vhosts name tracing
Listing vhosts ...
name	tracing
/test	false
/	    false

# 列出虚拟主机上的所有权限:
[root@CentOS7 ~]# rabbitmqctl list_permissions -p /
Listing permissions for vhost "/" ...
user	configure	write	read
guest	.*	        .*	    .*
```



**对象管理：**

```shell
# 列出所有queue：
rabbitmqctl list_queues [-p vhost]
# 列出指定queue的信息：
rabbitmqctl list_queues [the queue name] messages_ready messages_unacknowledged
# 列出所有exchange：
rabbitmqctl list_exchanges
# 列出所有binding：
rabbitmqctl list_bindings
```



**集群管理：**

```shell
# 1、加入集群[--ram添加内存模式 默认disk模式]
rabbitmqctl join_cluster --ram rabbit@mq01

# 2、查看集群状态
rabbitmqctl cluster_status

# 3、更改节点模式[顺序 关闭运用－〉更改类型->开启运用]
rabbitmqctl stop_app  –停止运用服务
rabbitmqctl change_cluster_node_type disc/ram –更改节点为磁盘或内存节点
rabbitmqctl start_app –开启运用服务

# 4、创建策略(集群同步策略……)
set_policy [-p vhostpath] {name} {pattern} {definition} [priority]

# 5、查看策略
rabbitmqctl list_policies

# 6、移除远程offline的节点
# 1.节点2停掉应用
rabbitmqctl stop_app 
# 2.节点1执行删除
rabbitmqctl forget_cluster_node rabbit@mq02

# 7、设置集群名称
rabbitmqctl set_cluster_name cluster_name

# 8、设置镜像模式
Rabbit提供镜像功能，需要基于rabbitmq策略来实现，政策是用来控制和修改群集范围的某个vhost队列行为和Exchange行为 
set_policy [-p vhostpath] {name} {pattern} {definition} [priority]
rabbitmqctl set_policy ha-all "^ha." "{""ha-mode"":""all""}"
rabbitmqctl set_policy ha-all "^" "{""ha-mode"":""all"",""ha-sync-mode"":""automatic""}"
rabbitmqctl set_policy -p demo  ha-all "^" "{""ha-mode"":""all"",""ha-sync-mode"":""automatic""}"

# 9、手动同步queue
rabbitmqctl sync_queue name

# 10、取消queue同步
rabbitmqctl cancel_sync_queue name

# 11、查看所有队列信息
rabbitmqctl list_queues

# 12、获取队列信息
rabbitmqctl list_queues[-p vhostpath] [queueinfoitem ...]
Queueinfoitem可以为：name，durable，auto_delete，arguments，messages_ready，messages_unacknowledged，messages，consumers，memory。

# 13、获取Exchange信息
rabbitmqctl list_exchanges[-p vhostpath] [exchangeinfoitem ...]
Exchangeinfoitem有：name，type，durable，auto_delete，internal，arguments。

# 14、获取Binding信息
rabbitmqctl list_bindings[-p vhostpath] [bindinginfoitem ...] 
Bindinginfoitem有：source_name，source_kind，destination_name，destination_kind，routing_key，arguments。

# 15、获取Connection信息
rabbitmqctl list_connections [connectioninfoitem ...]
Connectioninfoitem有：recv_oct，recv_cnt，send_oct，send_cnt，send_pend等。

# 16、获取Channel信息
rabbitmqctl list_channels[channelinfoitem ...]
Channelinfoitem有consumer_count，messages_unacknowledged，messages_uncommitted，acks_uncommitted，messages_unconfirmed，prefetch_count，client_flow_blocked。
```



> 参考资料：
> https://segmentfault.com/a/1190000038393678
> https://gblfy.blog.csdn.net/article/details/105737928
> https://soulchild.cn/1774.html



## 06、默认端口及配置文件

### 1、默认相关端口

- 4369：erlang发现口
- 5672：RabbitMQ的通讯端口
- 25672：RabbitMQ的节点间的CLI通讯端口是
- 15672：后台管理界面ui端口，进入管理后台时访问url如：http://localhost:15672/
- 1883，8883：MQTT插件启动时的端口。
- 61613、61614：STOMP客户端插件启用的时候的端口。
- 15674、15675：基于webscoket的STOMP端口和MOTT端口



### 2、默认配置文件

> **官网给出的配置文件模板**：[rabbitmq.conf.example](https://github.com/rabbitmq/rabbitmq-server/blob/master/deps/rabbit/docs/rabbitmq.conf.example)、[advanced.config.example](https://github.com/rabbitmq/rabbitmq-server/blob/master/deps/rabbit/docs/advanced.config.example)
>
> **官方建议配置文件的位置：**
> 1、Linux系统：/etc/rabbitmq/rabbitmq.conf
> 2、Win10系统：C:/Users/用户名/AppData/Roaming/RabbitMQ/advanced.config
>
> **参考官网：**
> 1：https://www.rabbitmq.com/documentation.html
> 2：https://www.rabbitmq.com/configure.html#config-items



1、RabbitMQ默认情况下有一个配置文件，定义了RabbitMQ的相关配置信息，默认情况下能够满足日常的开发需求。如果需要修改需要，需要自己创建一个配置文件进行覆盖。



2、RabbitMQ有三个配置文件：

主配置文件（rabbitmq.conf），Erlang术语格式配置文件（advanced.config）、环境变量配置文件（rabbitmq-env.conf）



3、在RabbitMQ-3.8版本，默认没有配置文件。需要手动添加。默认配置文件位置：${RABBITMQ_HOME}/etc/rabbitmq/ 下

- rabbitmq-env.conf：主要配置rabbitmq的环境变量的，如配置rabbitmq.conf和advanced.config文件的地址

- rabbitmq.conf：主要配置RabbitMQ服务器和插件

- advanced.config：某些配置设置不可能或难以使用sysctl格式进行配置。因此，可以使用Erlang术语格式的其他配置文件（与rabbitmq.config相同）。该文件通常命名为advanced.config。它将与Rabbitmq.conf中提供的配置合并。



以下是三个配置文件示例的链接：
**参考文章：**https://www.cnblogs.com/masy-lucifer/p/13551067.html

**1、rabbitmq.config**：https://www.cnblogs.com/masy-lucifer/p/13551090.html

**2、advanced.config**：https://www.cnblogs.com/masy-lucifer/p/13551106.html



例子：现在想把默认端口5672改成56720，并且允许远程访问。把web管理默认端口15672改成56271。只需如下配置：

```shell
$ vim /etc/rabbitmq/rabbitmq.conf

# AMQP 0-9-1 and 1.0 port，默认5672,允许远程访问
listeners.tcp.default = 0.0.0.0:56720
# web管理，默认15672
management.tcp.port = 56721
management.tcp.ip   = 0.0.0.0
```





## 07、可视化管理页面详解



rabbitmq-management是RabbitMq web管理端，用的是erlang的cowboy框架进行开发。web页面包括Overview(概述)、Connections(连接)、Channels(通道)、Exchanges(交换器)、Queues(队列)、Admin(用户管理)。



# RabbitMQ-AMQP 简介

## 01、RabbitMQ 概述

官网：https://www.rabbitmq.com

> **什么是RabbitMQ,官方给出来这样的解释：**
>
> RabbitMQ is the most widely deployed open source message broker.
> With tens of thousands of users, RabbitMQ is one of the most popular open source message brokers. From T-Mobile to Runtastic, RabbitMQ is used worldwide at small startups and large enterprises.
> RabbitMQ is lightweight and easy to deploy on premises and in the cloud. It supports multiple messaging protocols. RabbitMQ can be deployed in distributed and federated configurations to meet high-scale, high-availability requirements.
> RabbitMQ runs on many operating systems and cloud environments, and provides a wide range of developer tools for most popular languages.
> 翻译以后：
> RabbitMQ是部署最广泛的开源消息代理。
> RabbitMQ拥有成千上万的用户，是最受欢迎的开源消息代理之一。从T-Mobile 到Runtastic，RabbitMQ在全球范围内的小型初创企业和大型企业中都得到使用。
> RabbitMQ轻巧，易于在内部和云中部署。它支持多种消息传递协议。RabbitMQ可以部署在分布式和联合配置中，以满足大规模，高可用性的要求。
> RabbitMQ可在许多操作系统和云环境上运行，并为大多数流行语言提供了广泛的开发人员工具。

简单概述：RabbitMQ是一个开源的遵循AMQP协议实现的基于Erlang语言编写，支持多种客户端（语言）。用于在分布式系统中存储消息，转发消息，具有高可用，高可扩性，易用性等特征。



## 02、什么是 AMQP

**AMQP全称：**Advanced Message Queuing Protocol(高级消息队列协议)。是应用层协议的一个开发标准，为面向消息的中间件设计。

**AMQP生产者流转过程：**

![20210418110804](https://gitee.com/liusuixing/blog_images/raw/master/20210418110804.png)

**AMQP消费者流转过程：**

![20210418110805](https://gitee.com/liusuixing/blog_images/raw/master/20210418110805.png)





## 03、RabbitMQ 核心组成

> 1、RabbitMQ的核心组成部分

![20210418110806](https://gitee.com/liusuixing/blog_images/raw/master/20210418110806.png)

核心概念：

**Server**：又称Broker ,接受客户端的连接，实现AMQP实体服务。 安装rabbitmq-server
**Connection**：连接，应用程序与Broker的网络连接 TCP/IP/ 三次握手和四次挥手
**Channel**：网络信道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道，客户端可以建立对各Channel，每个Channel代表一个会话任务。
**Message** :消息：服务与应用程序之间传送的数据，由Properties和body组成，Properties可是对消息进行修饰，比如消息的优先级，延迟等高级特性，Body则就是消息体的内容。
**Virtual Host** 虚拟地址，用于进行逻辑隔离，最上层的消息路由，一个虚拟主机理由可以有若干个Exhange和Queueu，同一个虚拟主机里面不能有相同名字的Exchange
**Exchange**：交换机，接受消息，根据路由键发送消息到绑定的队列。(==不具备消息存储的能力==)
**Bindings**：Exchange和Queue之间的虚拟连接，binding中可以保护多个routing key.
**Routing key**：是一个路由规则，虚拟机可以用它来确定如何路由一个特定消息。
**Queue**：队列：也成为Message Queue,消息队列，保存消息并将它们转发给消费者。



> 2、RabbitMQ整体架构模型

![20210418110807](https://gitee.com/liusuixing/blog_images/raw/master/20210418110807.png)



> 3、RabbitMQ的运行流程

![20210418110808](https://gitee.com/liusuixing/blog_images/raw/master/20210418110808.png)



## 04、七种队列模式

> **七种模式介绍与应用场景**
>
> - 简单模式（Hello World）：简单生产者-消费者
> - 工作队列模式（Work queues）：工作队列
> - 订阅模式（Publish/Subscribe）：发布订阅模式，广播模型
> - 路由模式（Routing）：路由模式，订阅模式Direct
> - 主题模式（Topics）：动态路由模型，订阅模式Topics
> - 远程过程调用（RPC）
> - 发布者确认（Publisher Confirms）



### 1、简单模式（Hello World）

![20210418110809](https://gitee.com/liusuixing/blog_images/raw/master/20210418110809.png)

做最简单的事情，一个生产者对应一个消费者，RabbitMQ相当于一个消息代理，负责将A的消息转发给B

**应用场景：** 将发送的电子邮件放到消息队列，然后邮件服务在队列中获取邮件并发送给收件人





### 2、工作队列模式（Work queues）

![20210418110810](https://gitee.com/liusuixing/blog_images/raw/master/20210418110810.png)

在多个消费者之间分配任务（竞争的消费者模式），一个生产者对应多个消费者，一般适用于执行资源密集型任务，单个消费者处理不过来，需要多个消费者进行处理

**应用场景：** 一个订单的处理需要10s，有多个订单可以同时放到消息队列，然后让多个消费者同时处理，这样就是并行了，而不是单个消费者的串行情况



### 3、订阅模式-fanout（Pub/Sub）

![20210418110811](https://gitee.com/liusuixing/blog_images/raw/master/20210418110811.png)

一次向许多消费者发送消息，一个生产者发送的消息会被多个消费者获取，也就是将消息将广播到所有的消费者中。

**应用场景：** 更新商品库存后需要通知多个缓存和多个数据库，这里的结构应该是：

- 一个fanout类型交换机扇出两个个消息队列，分别为缓存消息队列、数据库消息队列
- 一个缓存消息队列对应着多个缓存消费者
- 一个数据库消息队列对应着多个数据库消费者



### 4、路由模式（Routing）

![20210418110812](https://gitee.com/liusuixing/blog_images/raw/master/20210418110812.png)

有选择地（Routing key）接收消息，发送消息到交换机并且要指定路由key ，消费者将队列绑定到交换机时需要指定路由key，仅消费指定路由key的消息

**应用场景：** 如在商品库存中增加了1台iphone12，iphone12促销活动消费者指定routing key为iphone12，只有此促销活动会接收到消息，其它促销活动不关心也不会消费此routing key的消息



### 5、主题模式（Topics）

![20210418110813](https://gitee.com/liusuixing/blog_images/raw/master/20210418110813.png)

根据主题（Topics）来接收消息，将路由key和某模式进行匹配，此时队列需要绑定在一个模式上，`#`匹配一个词或多个词，`*`只匹配一个词。

**应用场景：** 同上，iphone促销活动可以接收主题为iphone的消息，如iphone12、iphone13等



### 6、远程过程调用（RPC）

![20210418110814](https://gitee.com/liusuixing/blog_images/raw/master/20210418110814.png)

如果我们需要在远程计算机上运行功能并等待结果就可以使用RPC，具体流程可以看图。应用场景：需要等待接口返回数据，如订单支付



### 7、发布者确认（Publisher Confirms）

与发布者进行可靠的发布确认，发布者确认是RabbitMQ扩展，可以实现可靠的发布。在通道上启用发布者确认后，RabbitMQ将异步确认发送者发布的消息，这意味着它们已在服务器端处理。

**应用场景：** 对于消息可靠性要求较高，比如钱包扣款



# RabbitMQ-Java 入门案例

> 参考非常详细的博主教程：https://www.cnblogs.com/dtdx/p/14362760.html
> SpringBoot+Java 版教程：https://blog.csdn.net/lgl782519197/article/details/113775569

## 00、环境搭建

> **实现步骤：**
>
> 1、IDEDA内构建一个maven工程（jdk1.8）
> 2：导入rabbitmq的maven依赖
> 3：启动rabbitmq-server服务
> 4：定义生产者、定义消费者
> 5：观察消息的在rabbitmq-server服务中的过程

1、构建一个maven工程

2、引入rabbitmq-maven依赖

```xml
<!-- Java原生依赖 -->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.10.0</version>
</dependency>
```

```xml
<!-- Spring依赖 -->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-amqp</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
```

```xml
<!-- SpringBoot依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

> 按需选择即可。
> 番外：rabbitmq和spring同属一个公司开放的产品，所以他们的支持也是非常完善，这也是为什么推荐使用rabbitmq的一个原因。



## 01、Simple 简单队列

### 简单理解

如果把使用 RabbitMQ 进行消息发送的过程比喻成邮寄邮件。那么简单队列的场景是，只有一个邮箱、一个邮局、一个投递员，。消息通过 RabbitMQ 进行一对一发送，发送过程最简单。



### 简单队列模型示意图

![20210418110841](https://gitee.com/liusuixing/blog_images/raw/master/20210418110841.png)

### 代码实战

> 一个生产者，一个消费者，一个队列（**一个默认的交换机**）

1、定义生产者

```java
package com.example.simple;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author:
 * @description: Producer 简单队列生产者
 */
public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 从连接工厂中获取
        Connection connection = connectionFactory.newConnection("生产者");
        // 3: 从连接中打开通道channel
        Channel channel = connection.createChannel();

        // 4: 通过创建交换机，声明队列，绑定关系，路由key,发送消息，和接收消息
        /*
         *  申明队列：如果队列不存在会自动创建。
         *  注意：
         *  1.Rabbitmq不允许创建两个相同的队列名称，否则会报错。
         *  2.队列声明可以放在生产者、消费者或web页面上创建。但是在消费者启动监听之前队列一定要创建好。
         *    如果要先启动消费者，建议把声明队列放在消费者端。否在消费者监听队列不存在会报异常。
         *    如果要先启动生产者，建议在生产者端声明队列。虽然发送消息时队列不存不会报错，但第一次发送时队列不存在相当于白发送了。
         *
         *  @params1： queue 队列的名称
         *  @params2： durable 队列是否持久化（即存盘），false = 非持久化 true = 持久化，非持久化会存盘吗？会存盘，但是会随从重启服务会丢失
         *  @params3： exclusive 是否排他，即是否私有的，如果为true,会对当前队列加锁，其他的通道不能访问，并且连接自动关闭
         *  @params4： autoDelete 是否自动删除，当此队列的连接数为0时，此队列会销毁（无论队列中是否还有数据）
         *  @params5： arguments 可以设置队列附加参数，设置队列的有效期，消息的最大长度，队列的消息生命周期等等
         * */
        channel.queueDeclare("simple-queue1", false, false, false, null);
        // 5： 准备发送消息的内容
        String message = "你好，消息队列！！！";
        // 6: 发送消息给队列queue1
        /*
         * @params1: 交换机exchange
         * @params2: 队列名称、路由key(routing)
         * @params3: 属性配置
         * @params4: 发送消息的内容
         **/
        // 面试题：可以存在没有交换机的队列吗？不可能，虽然没有指定交换机但是一定会存在一个默认的交换机。
        channel.basicPublish("", "simple-queue1", null, message.getBytes());
        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```

2、定义消费者

```java
package com.example.simple;

import com.rabbitmq.client.*;
import java.io.IOException;

/**
 * @author:
 * @description: Producer 简单队列消费者
 */
public class Consumer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        /*
         * 2: 从连接工厂中获取/创建连接（断点到此步可以发现web界面Connection下会出现此连接信息）
         * 3: 从连接中获取通道channel（断点到此步可以发现web界面Channel下会出现此连接信息）
         */
        try (Connection connection = connectionFactory.newConnection("消费者");
             Channel channel = connection.createChannel()){

            // 4: 通过创建交换机，声明队列，绑定关系，路由key,发送消息，和接收消息（声明队列可以在生产者或者消费者端）
            //channel.queueDeclare("queue1", false, false, false, null);

            // 接收消息，监听对应的队列名即可
            /*
             *  @params1： queue 队列的名称
             *  @params2： autoAck 指定是否自动ACK （true，接收到消息后，会立即告诉RabbitMQ）
             *  @params3： deliverCallback 指定消费回调,开启监听队列queue1
             *  @params4： cancelCallback 消费失败回调
             * */
            channel.basicConsume("simple-queue1", true, new DeliverCallback() {
                @Override
                public void handle(String consumerTag, Delivery delivery) throws IOException {
                    System.out.println("收到消息是" + new String(delivery.getBody(), "UTF-8"));
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s)  {
                    System.out.println("接受失败了...");
                }
            });

            // 让程序停止，好接收消费
            System.out.println("开始接受消息");
            System.in.read();
        }

    }
}
```

3、先运行生产者，然后运行消费者。最后消费者控制台输出：

```
开始接受消息
收到消息是你好，消息队列！！！
```



## 02、Work 工作队列

### 简单理解

在前面我们学了Simple Queue 模型，Simple Queue 模型消息生产者和消费者是一 一对应的，消息由生产者发送到队列并由消费者消费。假设有一种消息，消息生产者一次性发送了10条消息，消费者消费一条消息执行耗时1秒，如果使用简单队列模式总共耗时10秒；如果使用工作队列模式有3个消费者共同消费这些消息理想情况下只需要3秒就可以处理完这些消息。



### 工作队列模型示意图

![20210418110842](https://gitee.com/liusuixing/blog_images/raw/master/20210418110842.png)



### 代码实战-轮询模式（Round-Robin）

1、定义生产者

```java
package com.example.work.round;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 3: 声明队列
        channel.queueDeclare("work-queue1", false, false, false, null);
        // 4: 循环发送消息
        for (int i = 1; i <= 20; i++) {
            channel.basicPublish("", "work-queue1", null, ("work-轮询模式:"+ i).getBytes());
        }
        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```

2、定义消费者1

```java
package com.example.work.round;

import com.rabbitmq.client.*;

public class Work1 {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("消费者1");
        Channel channel = connection.createChannel();

        // 接收消息，监听对应的队列名即可
        channel.basicConsume("work-queue1", true, (consumerTag, delivery) ->
                System.out.println("收到消息是" + new String(delivery.getBody(), "UTF-8")),
                consumerTag  -> {
        });

    }
}
```

3、定义消费者2

```java
package com.example.work.round;

import com.rabbitmq.client.*;

public class Work2 {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("消费者2");
        Channel channel = connection.createChannel();

        // 接收消息，监听对应的队列名即可
        channel.basicConsume("work-queue1", true, (consumerTag, delivery) ->
                System.out.println("收到消息是" + new String(delivery.getBody(), "UTF-8")),
                consumerTag  -> {
        });

    }
}
```

查看控制台输出：

消费者1：

```
收到消息是work-轮询模式:1
收到消息是work-轮询模式:3
收到消息是work-轮询模式:5
收到消息是work-轮询模式:7
收到消息是work-轮询模式:9
收到消息是work-轮询模式:11
收到消息是work-轮询模式:13
收到消息是work-轮询模式:15
收到消息是work-轮询模式:17
收到消息是work-轮询模式:19
```

消费者2：

```
收到消息是work-轮询模式:2
收到消息是work-轮询模式:4
收到消息是work-轮询模式:6
收到消息是work-轮询模式:8
收到消息是work-轮询模式:10
收到消息是work-轮询模式:12
收到消息是work-轮询模式:14
收到消息是work-轮询模式:16
收到消息是work-轮询模式:18
收到消息是work-轮询模式:20
```



### 代码实战-公平分发（Fair-Dispatch）

> 也可以称为：Work模式的”能者多劳“模式

1、定义生产者

```java
package com.example.work.fairr;

import com.rabbitmq.client.*;

public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 3: 声明队列
        channel.queueDeclare("work-queue2", false, false, false, null);
        for (int i = 1; i <= 20; i++) {
            channel.basicPublish("", "work-queue2", null, ("work-公平分发:"+ i).getBytes());
        }
        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```

2、定义消费者1

```java
package com.example.work.fairr;

import com.rabbitmq.client.*;
import java.util.concurrent.TimeUnit;

public class Work1 {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("消费者1");
        Channel channel = connection.createChannel();

        // 同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            try {
                // 加一个睡眠，模拟消费者1消费慢
                TimeUnit.MILLISECONDS.sleep(1000);
                System.out.println("Work1-收到消息是" + new String(delivery.getBody(), "UTF-8"));
                // 确认消息消费，参数1：确认队列中哪个具体消息，参数2：是否开启多个消息同时确认
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };

        // 接收消息，开启手动确认消费消息机制，设置autoAck为false，防止消息一下子都进入消费者
        channel.basicConsume("work-queue2", false, deliverCallback,consumerTag  -> {
        });

    }
}
```

3、定义消费者2

```java
package com.example.work.fairr;

import com.rabbitmq.client.*;
import java.util.concurrent.TimeUnit;

public class Work2 {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("消费者2");
        Channel channel = connection.createChannel();

        // 同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            try {
                // 加一个睡眠，模拟消费者1消费慢
                TimeUnit.MILLISECONDS.sleep(100);
                System.out.println("Work2-收到消息是" + new String(delivery.getBody(), "UTF-8"));
                // 确认消息消费，参数1：确认队列中哪个具体消息，参数2：是否开启多个消息同时确认
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };

        // 接收消息，开启确认机制，设置autoAck为false
        channel.basicConsume("work-queue2", false, deliverCallback, consumerTag  -> {
        });

    }
}
```

与一般情况不同的是如下三个步骤（在消费端配置）：

```java
// 1.同一时刻服务器只会发一条消息给消费者
channel.basicQos(1);

// 2.开启这行 表示使用手动确认模式
channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);

// 3.同时改为手动确认：监听队列，false表示手动返回完成状态，true表示自动
channel.basicConsume(QUEUE_NAME, false, consumer);
```



查看控制台输出结果：

消费者1：

```
Work1-收到消息是work-公平分发:2
Work1-收到消息是work-公平分发:12
```

消费者2：

```
Work2-收到消息是work-公平分发:1
Work2-收到消息是work-公平分发:3
Work2-收到消息是work-公平分发:4
Work2-收到消息是work-公平分发:5
Work2-收到消息是work-公平分发:6
Work2-收到消息是work-公平分发:7
Work2-收到消息是work-公平分发:8
Work2-收到消息是work-公平分发:9
Work2-收到消息是work-公平分发:10
Work2-收到消息是work-公平分发:11
Work2-收到消息是work-公平分发:13
Work2-收到消息是work-公平分发:14
Work2-收到消息是work-公平分发:15
Work2-收到消息是work-公平分发:16
Work2-收到消息是work-公平分发:17
Work2-收到消息是work-公平分发:18
Work2-收到消息是work-公平分发:19
Work2-收到消息是work-公平分发:20
```



## 03、Fanout 发布订阅

### Pub/Sub模型简单理解

在前面我们学习了简单队列模型和工作队列模型，其实不管是简单队列模型还是工作队列模式只用到了 RabbitMQ 中的 Queue 队列，消息直接发送到队列中，消费者直接从队列中获取消息。

现在我们学习另一种消息的处理方式，消息生产者将消息发送到 Exchange 交换机中，再由交换机将消息投递到 Queue队列中，交换机和队列的关系是多对多的关系，表示一个交换机可以通过一定的规矩将消息投递到多个队列中，同样一个队列也可以接收多个交换机投递的消息

> **交换机投递消息到队列由哪几种方式？**
>
> - **Fanout**： 不处理路由键。你只需要简单的将队列绑定到交换机上。一个发送到交换机的消息都会被转发到与该交换机绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。Fanout交换机转发消息是最快的
> - **Direct**：处理路由键。需要将一个队列绑定到交换机上，要求该消息与一个特定的路由键**完全匹配**。这是一个完整的匹配。如果一个队列绑定到该交换机上要求路由键 `test`，则只有被标记为`test`的消息才被转发，不会转发test.aaa，也不会转发dog.123，只会转发test。
> - **Topic**： 将路由键和某模式进行匹配。此时队列需要绑定要一个模式上。符号`#`匹配一个或多个词，符号`*` 匹配不多不少一个词。因此`audit.#`能够匹配到`audit.irs.corporate`，但是`audit.*` 只会匹配到`audit.irs`
> - **Headers**：Headers类型的exchange使用的比较少，它也是忽略routingKey的一种路由方式。是使用Headers来匹配的。Headers是一个键值对，可以定义成Hashtable。发送者在发送的时候定义一些键值对，接收者也可以再绑定时候传入一些键值对，两者匹配的话，则对应的队列就可以收到消息。匹配有两种方式all和any。这两种方式是在接收端必须要用键值"x-mactch"来定义。all代表定义的多个键值对都要满足，而any则代码只要满足一个就可以了。fanout，direct，topic exchange的routingKey都需要要字符串形式的，而headers exchange则没有这个要求，因为键值对的值可以是任何类型



### 发布订阅模型示例图

![20210418110843](https://gitee.com/liusuixing/blog_images/raw/master/20210418110843.png)

### 代码实战

> 一个生产者，一个交换机，三个队列，三个消费者
>特点：发布与订阅模式，是一种广播机制，它是没有路由key的模式。

1、定义生产者

```java
package com.example.fanout;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 3: 定义队列名称-随机生成队列名、交换机名、路由key（fanout模式时路由key为空字符串）
        String exchangeName = "fanout-exchange";

        // 4-1: 声明交换机（也可通过web页面创建）
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.FANOUT);
        // 4-2: 声明队列
        channel.queueDeclare("fanout-queue1", false, false, false, null);
        channel.queueDeclare("fanout-queue2", false, false, false, null);
        channel.queueDeclare("fanout-queue3", false, false, false, null);
        // 4-3: 绑定交换机和队列
        channel.queueBind("fanout-queue1", exchangeName, "");
        channel.queueBind("fanout-queue2", exchangeName, "");
        channel.queueBind("fanout-queue3", exchangeName, "");

        // 5: @params1: 交换机名  @params2 队列/路由key @params 属性配置  @params4 消息内容
        channel.basicPublish(exchangeName, "", null, "你好，消息队列！".getBytes("UTF-8"));
        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```

2、定义消费者

```java
package com.example.fanout;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class Consumer {

    private static Runnable runnable = () -> {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        Connection connection = null;
        Channel channel = null;
        try {
            // 2: 创建连接，获取通道（消费者一般不增加自动关闭）
            connection = connectionFactory.newConnection("消费者");
            channel = connection.createChannel();
            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();

            // 6： 定义接受消息的回调
            DeliverCallback deliverCallback = (consumerTag, delivery) ->
                    System.out.println(queueName + "：收到消息是：" + new String(delivery.getBody(), "UTF-8"));

            channel.basicConsume(queueName, true, deliverCallback, consumerTag  -> {
            });
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("发送消息出现异常...");
        } finally {
            // 此处应该是需要关闭通道的，不过为了测试咱们不关闭了
        }
    };

    public static void main(String[] args) {
        // 启动三个线程去执行
        new Thread(runnable, "fanout-queue1").start();
        new Thread(runnable, "fanout-queue2").start();
        new Thread(runnable, "fanout-queue3").start();
    }
}
```

消费者控制台输出：

```
fanout-queue3：收到消息是：你好，消息队列！
fanout-queue1：收到消息是：你好，消息队列！
fanout-queue2：收到消息是：你好，消息队列！
```



## 04、Direct 路由模式

### 简单理解

Direct Exchange 消息模型 是交换机将消息投递到队列中的另一种方式，需要将交换机类型设置为 direct。那么什么是 direct 类型交换机？

> 其中**Direct**：处理路由键。需要将一个队列绑定到交换机上，要求该消息与一个特定的路由键**完全匹配**。这是一个完整的匹配。如果一个队列绑定到该交换机上要求路由键 `test`，则只有被标记为`test`的消息才被转发，不会转发test.aaa，也不会转发dog.123，只会转发test。



### 路由消息模型示意图

![20210418110844](https://gitee.com/liusuixing/blog_images/raw/master/20210418110844.png)

### Direct 消息流转过程

- 生产者向Exchange发送消息。
- 队列使用路由密钥绑定到Exchange。
- 通常，使用相同/不同的路由密钥有多个队列绑定到Exchange。
- 发送到Exchange的消息包含路由密钥。根据路由密钥，消息将转发到一个或多个队列。
- 订阅队列的使用者接收消息并进行处理。



### 代码实战

> 一个生产者，一个交换机，两个队列，两个消费者
>特点：Direct模式是fanout模式上的一种叠加，增加了路由RoutingKey的模式。

1、定义生产者

```java
package com.example.direct;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 3: 定义队列名称-随机生成队列名、交换机名、路由key（fanout模式时路由key为空字符串）
        String exchangeName = "direct-exchange";

        // 4-1: 声明交换机
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT);
        // 4-2: 声明队列
        channel.queueDeclare("direct-info", false, false, false, null);
        channel.queueDeclare("direct-error", false, false, false, null);
        // 4-3: 绑定交换机和队列
        channel.queueBind("direct-info", exchangeName, "info");
        channel.queueBind("direct-error", exchangeName, "error");

        // 5: @params1: 交换机名  @params2 队列/路由key @params 属性配置  @params4 消息内容
        channel.basicPublish(exchangeName, "info", null, "你好，消息队列！".getBytes("UTF-8"));
        channel.basicPublish(exchangeName, "error", null, "你好，消息队列！".getBytes("UTF-8"));

        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```

2、定义消费者

```java
package com.example.direct;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class Consumer {

    private static Runnable runnable = () -> {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        Connection connection = null;
        Channel channel = null;
        try {
            // 2: 创建连接，获取通道（消费者一般不增加自动关闭）
            connection = connectionFactory.newConnection("消费者");
            channel = connection.createChannel();

            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();

            // 3：定义接受消息的回调
            channel.basicConsume(queueName, true, (consumerTag, delivery) ->
                    System.out.println(queueName + "：收到消息是：" + new String(delivery.getBody()),
                    consumerTag  -> {
            });
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("发送消息出现异常...");
        } finally {
            // 此处应该是需要关闭通道的，不过为了测试咱们不关闭了
        }
    };

    public static void main(String[] args) {
        // 启动三个线程去执行
        new Thread(runnable, "direct-info").start();
        new Thread(runnable, "direct-error").start();
    }
}
```

消费者控制台输出：

```
direct-error：收到消息是：你好，消息队列！
direct-info：收到消息是：你好，消息队列！
```



## 05、Topic 主题模式

### 简单理解

Topic Exchange 消息模型需要指定路由器类型设置为`Topic`，那么什么是 `Topic` 类型？

> **Topic**： 将路由键和某模式进行匹配。此时队列需要绑定要一个模式上。符号`#`匹配一个或多个词，符号`*` 匹配不多不少一个词。因此`audit.#`能够匹配到`audit.irs.corporate`，但是`audit.*` 只会匹配到`audit.irs`



### 主题消息模型示意图

![20210418110845](https://gitee.com/liusuixing/blog_images/raw/master/20210418110845.png)

### Topic 中的路由键设置规则

- Topic Exchange中的路由关键字必须包含零个或多个由 `.` 点分隔的单词，例如`health.education`。
- Topic Exchange中的路由键通常称为**路由模式**。
- 路由键允许只包含 星号（`*`）和 井号 （`#`）的正则表达式组成
- 星号（*****）表示正好允许一个字。
- 同样，井号（**＃**）表示允许的单词数为零或更多。
- 点号（`.`）表示–单词定界符。多个关键术语用点定界符分隔。
- 如果路由模式为**`health.\*`**，则意味着以第一个单词为首的路由键运行状况发送的任何消息都将到达队列。例如，`health.education`将到达此队列，但`sports.health`将不起作用。

### Topic 中的消息流转过程

- 一个Queue队列通过路由键（P）绑定到 Exchange。
- Producer 将带有P路由键（K）的消息发送到 Topic Exchange。
- 如果P与K匹配，则消息被传递到队列。路由密钥匹配的确定如下所述。
- 订阅队列的使用者将收到消息。



### 代码实战

> 一个生产者，一个交换机，两个队列，两个消费者
>特点：Topic模式是direct模式上的一种叠加，增加了模糊路由RoutingKey的模式。
> 
>生产者创建Topic的exchange并且绑定到队列中，**绑定**可以通过\*和#关键字，对指定RoutingKey内容，编写时注意格式 xxx.xxx.xxx 去别写。\* -》代表xxx，# -》代表多个xxx.xxx，在发送消息时，指定具体的RoutingKey到底时什么
> 简单的说：* 表示一个，# 表示 0个 或者 多个

1、定义生产者

```java
package com.example.topic;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Producer {
    public static void main(String[] args) throws IOException, TimeoutException {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 3: 定义队列名称-随机生成队列名、交换机名、路由key（fanout模式时路由key为空字符串）
        String exchangeName = "topic-exchange";

        // 4-1: 声明交换机（也可通过web页面创建）
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC);

        // 4-2: 声明队列
        channel.queueDeclare("topic-queue1", false, false, false, null);
        channel.queueDeclare("topic-queue2", false, false, false, null);
        channel.queueDeclare("topic-queue3", false, false, false, null);

        // 4-3: 绑定交换机和队列
        channel.queueBind("topic-queue1", exchangeName, "#.order");
        channel.queueBind("topic-queue2", exchangeName, "*.user");

        // 5: @params1: 交换机名  @params2 队列/路由key @params 属性配置  @params4 消息内容
        // topic-queue1、topic-queue2、topic-queue3
        channel.basicPublish(exchangeName, "com.course.order", null, "routingKey：#.order".getBytes("UTF-8"));
        // topic-queue3
        channel.basicPublish(exchangeName, "com.order.user", null, "routingKey：#.user".getBytes("UTF-8"));

        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```

2、定义消费者

```java
package com.example.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class Consumer {

    private static Runnable runnable = () -> {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        Connection connection = null;
        Channel channel = null;
        try {
            // 2: 创建连接，获取通道（消费者一般不增加自动关闭）
            connection = connectionFactory.newConnection("消费者");
            channel = connection.createChannel();
            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();

            // 6： 定义接受消息的回调
            channel.basicConsume(queueName, true,
                    (consumerTag, delivery) -> System.out.println(queueName + "：收到消息是：" + new String(delivery.getBody(), "UTF-8")),
                    consumerTag  -> {
            });
        } catch (Exception e) {
            e.printStackTrace();
            System.err.println("发送消息出现异常...");
        } finally {
            // 此处应该是需要关闭通道的，不过为了测试咱们不关闭了
        }
    };

    public static void main(String[] args) {
        // 启动三个线程去执行
        new Thread(runnable, "topic-queue1").start();
        new Thread(runnable, "topic-queue2").start();
        new Thread(runnable, "topic-queue3").start();
    }
}
```

消费者控制台输出：

```
topic-queue1：收到消息是：routingKey：#.order
```



## 06、Headers 模式

Headers Exchange 模型需要指定路由器类型设置为`Headers`，那么什么是 Headers`Headers` 类型？

> **Headers**：Headers类型的exchange使用的比较少，它也是忽略routingKey的一种路由方式。是使用Headers来匹配的。Headers是一个键值对，可以定义成Hashtable。发送者在发送的时候定义一些键值对，接收者也可以再绑定时候传入一些键值对，两者匹配的话，则对应的队列就可以收到消息。匹配有两种方式all和any。这两种方式是在接收端必须要用键值"x-mactch"来定义。all代表定义的多个键值对都要满足，而any则代码只要满足一个就可以了。fanout，direct，topic exchange的routingKey都需要要字符串形式的，而headers exchange则没有这个要求，因为键值对的值可以是任何类型

### Headers 消息模型示意图

![20210418110840](https://gitee.com/liusuixing/blog_images/raw/master/20210418110840.png)

### Headers 中消息流转过程

- 一个或多个队列使用标头属性（H）绑定（链接）到标头交换。
- 生产者将带有标头属性（MH）的消息发送到此Exchange。
- 如果MH与H匹配，则消息将转发到队列。
- 监听队列的使用者接收消息并对其进行处理。



### 代码实现

1、生产者：

```java
package com.example.header;

import com.rabbitmq.client.*;

import java.util.HashMap;
import java.util.Map;

public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");

        // 2: 创建连接，获取通道
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 3: 定义队列名称-随机生成队列名、交换机名、路由key

        /**
         * 定义交换机、队列 和 绑定队列和交换机并绑定配置参数
         */
        // 4-1: 声明交换机
        channel.exchangeDeclare("header-exchange", BuiltinExchangeType.HEADERS);
        // 4-2: 声明队列
        channel.queueDeclare("header-queue-noe", false, false, false, null);
        channel.queueDeclare("header-queue-two", false, false, false, null);
        // 4-3: 绑定交换机和队列并加上配置header匹配规则
        Map<String, Object> oneArgs = new HashMap<>();
        // 匹配其中一个key/value 就能匹配成功
        oneArgs.put("x-match", "any");
        oneArgs.put("h1", "Header1");
        oneArgs.put("h2", "Header2");
        channel.queueBind("header-queue-noe", "header-exchange", "", oneArgs);
        Map<String, Object> twoArgs = new HashMap<>();
        // 必须匹配所有key/value 才能匹配成功
        twoArgs.put("x-match", "all");
        twoArgs.put("h1", "Header1");
        twoArgs.put("h2", "Header2");
        channel.queueBind("header-queue-two", "header-exchange", "", twoArgs);


        /**
         * 定义需要发送的消息和属性
         */
        Map<String, Object> headerMap = new HashMap<>();
        headerMap.put("h1", "Header1");
        headerMap.put("h3", "Header3");

        AMQP.BasicProperties basicPropertiesOne = new AMQP.BasicProperties()
                .builder().headers(headerMap).build();
        channel.basicPublish("header-exchange", "", basicPropertiesOne, "Header Exchange example 1".getBytes("UTF-8"));
        System.out.println("h1,h3：消息发送成功!");

        headerMap.put("h2", "Header2");
        AMQP.BasicProperties basicPropertiesTwo = new AMQP.BasicProperties()
                .builder().headers(headerMap).build();
        channel.basicPublish("header-exchange", "", basicPropertiesTwo, "Header Exchange example 2".getBytes("UTF-8"));
        System.out.println("h1,h2,h3：消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```



2、消费者：

```java
package com.example.header;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class Consumer {

    private static Runnable runnable = () -> {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        Connection connection = null;
        Channel channel = null;
        try {
            // 2: 创建连接，获取通道（消费者一般不增加自动关闭）
            connection = connectionFactory.newConnection("消费者");
            channel = connection.createChannel();
            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();

            // 6： 定义接受消息的回调
            channel.basicConsume(queueName, true, ((consumerTag, message) -> {
                System.out.println(queueName + "：收到消息是: " + new String(message.getBody()));
            }), consumerTag -> {
                System.out.println(consumerTag);
            });
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("发送消息出现异常...");
        } finally {
            // 此处应该是需要关闭通道的，不过为了测试咱们不关闭了
        }
    };

    public static void main(String[] args) {
        // 启动三个线程去执行
        new Thread(runnable, "header-queue-noe").start();
        new Thread(runnable, "header-queue-two").start();
    }
}
```



消费者控制台输出：

```
header-queue-noe：收到消息是: Header Exchange example 1
header-queue-two：收到消息是: Header Exchange example 2
header-queue-noe：收到消息是: Header Exchange example 2
```



# RabbitMQ-SpringBoot 案例

## 00、环境搭建

> 1、IDEA创建生产者工程：springboot-rabbitmq-producer
> 2、IDEA创建消费者工程：springboot-rabbitmq-consumer
> 3、生产者和消费者分别引入spring-boot-rabbitmq的依赖和配置连接信息
> 4、进行消息的分发和测试
> 5、查看和观察web控制台的状况

1、首先使用IDEA创建SpringBoot项目（步骤略）

2、同时引入如下依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

3、生产者：application.properties文件配置

```properties
server.port=8080
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
```

4、生产者：application.properties文件配置

```properties
server.port=8082
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
```





## 01、Simple 模式

1、创建队列（生产者消费者同时定义最好）

```java
package com.example.config;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 简单队列模式
 **/
@Configuration
public class SimpleRabbitConfig {
    @Bean
    public Queue simpleQueue() {
        /*
         * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
         * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
         * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
         * return new Queue("TestDirectQueue",true,true,false);
         * 一般设置一下队列的持久化就好,其余两个就是默认false
         */
        return new Queue("simple.queue", true);
    }
}
```

2、生产者发送消息

```java
    /**
     * 简单模式
     */
    @Test
    public void simpleSend() {
        // 简单对列的情况下routingKey即为Q名
        this.rabbitTemplate.convertAndSend("simple.queue", "点对点Simple模式测试");
    }
```

3、消费者

```java
package com.example.service;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class SimpleConsumer {
    //监听的队列名
    @RabbitListener(queues = "simple.queue")
    public void processOne(String name) {
        System.out.println("simple.queue 收到消息：" + name);
    }
}
```

4、消费者收到测试结果：

```
simple.queue 收到消息：点对点Simple模式测试
```





## 02、Work 模式

### 02-1、轮询分发模式

> **轮询分发、平均消费模式**：2个消费者队列，轮训获取队列里的消息

1、创建队列（生产者消费者同时定义最好）

```java
package com.example.config;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 工作队列模式
 **/
@Configuration
public class WorkRabbitConfig {
    @Bean
    public Queue workQueue(){
        /*
         * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
         * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
         * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
         * return new Queue("TestDirectQueue",true,true,false);
         * 一般设置一下队列的持久化就好,其余两个就是默认false
         */
        return new Queue("work.queue",true);
    }
}
```

2、生产者发送消息

```java
    /**
     * 工作队列模式
     */
    @Test
    public void workSend() {
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend("work.queue", "工作模式队列:" + i);
        }
    }
```

3、消费者-1（慢消费者）

```java
package com.example.service;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * 慢消费者，模拟性能比较差的服务器
 */
@Component
public class WorkConsumer1 {
    @RabbitListener(queues = "work.queue")
    public void processOne(String name, Channel channel, Message message) throws Exception {
        // 模拟执行每次任务需要1秒
        Thread.sleep(1000);
        System.out.println("WorkConsumer1-慢消费者 收到消息：" + name);
    }
}
```

4、消费者-2（快消费者）

```java
package com.example.service;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * 快消费者，模拟性能比较好的服务器
 */
@Component
public class WorkConsumer2 {
    @RabbitListener(queues = "work.queue")
    public void processTwo(String name, Channel channel, Message message) throws Exception {
        System.out.println("WorkConsumer2-快消费者 收到消息：" + name);
    }
}
```

5、消费者收到测试结果：

```
WorkConsumer2-快消费者 收到消息：工作模式队列:1
WorkConsumer2-快消费者 收到消息：工作模式队列:3
WorkConsumer2-快消费者 收到消息：工作模式队列:5
WorkConsumer2-快消费者 收到消息：工作模式队列:7
WorkConsumer2-快消费者 收到消息：工作模式队列:9
WorkConsumer1-慢消费者 收到消息：工作模式队列:0
WorkConsumer1-慢消费者 收到消息：工作模式队列:2
WorkConsumer1-慢消费者 收到消息：工作模式队列:4
WorkConsumer1-慢消费者 收到消息：工作模式队列:6
WorkConsumer1-慢消费者 收到消息：工作模式队列:8
```

可以看到效果，两个消费者都分别获取到了5条消息，但是问题来了，消费者-2 很快的消费完了自己的5条消息，然后就闲置了。但是消费者-1执行很慢，执行5条消息，就需要大于5秒的时间，这样就造成了性能浪费。我们应该让RabbitMQ智能一些，给每个消费者每次只给一个消息，当确认消息完成之后再下发另一个消息，这样就可以能者多劳了。


### 02-2、能者多劳模式

> **确认机制和能者多劳消费模式**：2个消费者队列，手动确认接收消息，消费者1加了睡眠时间，用来模拟消费消息慢

1、修改配置文件增加配置

```properties
# 在单个请求中处理的消息个数，每个消费者每次监听时可拉取处理的消息数量
spring.rabbitmq.listener.simple.prefetch=1
# 消费手动确认
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

**备注：**实际上不加手动ack消息确认也可以实现。不过 一般来说会两个配置同时加上

2、生产者发送消息不变

```java
    /**
     * 工作队列模式
     */
    @Test
    public void workSend() {
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend("work.queue", "工作模式队列:" + i);
        }
    }
```

2、消费者-1（慢消费者）

```java
package com.example.service;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * 慢消费者，模拟性能比较差的服务器
 */
@Component
public class WorkConsumer1 {
    @RabbitListener(queues = "work.queue")
    public void processOne(String name, Channel channel, Message message) throws Exception {
        // 模拟执行每次任务需要1秒
        Thread.sleep(1000);
        System.out.println("WorkConsumer1-慢消费者 收到消息：" + name);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
    }
}
```

3、消费者-2（快消费者）

```java
package com.example.service;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * 快消费者，模拟性能比较好的服务器
 */
@Component
public class WorkConsumer2 {
    @RabbitListener(queues = "work.queue")
    public void processTwo(String name, Channel channel, Message message) throws Exception {
        System.out.println("WorkConsumer2-快消费者 收到消息：" + name);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
    }
}
```

4、消费者收到测试结果：

```
WorkConsumer2-快消费者 收到消息：工作模式队列:1
WorkConsumer2-快消费者 收到消息：工作模式队列:2
WorkConsumer2-快消费者 收到消息：工作模式队列:3
WorkConsumer2-快消费者 收到消息：工作模式队列:4
WorkConsumer2-快消费者 收到消息：工作模式队列:5
WorkConsumer2-快消费者 收到消息：工作模式队列:6
WorkConsumer2-快消费者 收到消息：工作模式队列:7
WorkConsumer2-快消费者 收到消息：工作模式队列:8
WorkConsumer2-快消费者 收到消息：工作模式队列:9
WorkConsumer1-慢消费者 收到消息：工作模式队列:0
```

可以看到，已经达到我们想要的效果了，消费-2者消费了9条消息，而消费者-1 只消费了一条消息。





## 03、Fanout 模式

1、创建队列、交换机、绑定关系（生产者消费者同时定义最好）

```java
package com.example.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Fanout 订阅模式
 **/
@Configuration
public class FanoutRabbitMQConfig {
    /**
     * 声明队列
     */
    @Bean
    public Queue fanoutQueueOne() {
        /*
         * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
         * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
         * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
         * return new Queue("TestFanoutQueue",true,true,false);
         * 一般设置一下队列的持久化就好,其余两个就是默认false
         */
        return new Queue("fanout.queue.one", true);
    }

    @Bean
    public Queue fanoutQueueTwo() {
        return new Queue("fanout.queue.two", true);
    }
    @Bean
    public Queue fanoutQueueThree() {
        return new Queue("fanout.queue.three", true);
    }

    /**
     * 创建 Fanout 模式交换机
     */
    @Bean
    public FanoutExchange fanoutExchange() {
        /*
         * param1: 交换机名称
         * param2: durable：是否持久化，RabbitMQ关闭后，没持久化Exchange将被清除
         * param3: autoDelete：是否自动删除，，如果没有与之绑定的Queue直接删除
         */
        return new FanoutExchange("fanout_exchange", true, false);
    }

    /**
     * 绑定队列(不用指定routing key),参数名字要和bean名字一致
     */
    @Bean
    public Binding bindingFanout1() {
        return BindingBuilder.bind(fanoutQueueOne()).to(fanoutExchange());
    }
    @Bean
    public Binding bindingFanout2() {
        return BindingBuilder.bind(fanoutQueueTwo()).to(fanoutExchange());
    }
    @Bean
    public Binding bindingFanout3() {
        return BindingBuilder.bind(fanoutQueueThree()).to(fanoutExchange());
    }
}
```

2、生产者发送消息

```java
    /**
     * 发布订阅模式
     */
    @Test
    public void fanoutSend() {
        // 发布订阅模式下routingKey为空
        this.rabbitTemplate.convertAndSend("fanout_exchange","","Fanout-发布订阅模式测试");
    }
```

3、消费者

```java
package com.example.service;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class FanoutConsumer {

    @RabbitListener(queues = "fanout.queue.one")
    public void processEmail(String name) {
        System.out.println("fanout.queue.one 收到消息：" + name);
    }

    @RabbitListener(queues = "fanout.queue.two")
    public void processSms(String name) {
        System.out.println("fanout.queue.two 收到消息：" + name);
    }

    @RabbitListener(queues = "fanout.queue.three")
    public void processWeiXin(String name) {
        System.out.println("fanout.queue.three 收到消息：" + name);
    }
}
```

5、消费者收到测试结果：

```
fanout.queue.one 收到消息：Fanout-发布订阅模式测试
fanout.queue.three 收到消息：Fanout-发布订阅模式测试
fanout.queue.two 收到消息：Fanout-发布订阅模式测试
```





## 04、Direct 模式

1、创建队列、交换机、绑定关系

```java
package com.example.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Direct 路由模式
 **/
@Configuration
public class DirectRabbitMQConfig {
    /**
     * 声明队列
     */
    @Bean
    public Queue directQueueOne() {
        /*
         * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
         * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
         * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
         * return new Queue("TestDirectQueue",true,true,false);
         * 一般设置一下队列的持久化就好,其余两个就是默认false
         */
        return new Queue("direct.queue.one", true);
    }
    @Bean
    public Queue directQueueTwo() {
        return new Queue("direct.queue.two", true);
    }
    @Bean
    public Queue directQueueThree() {
        return new Queue("direct.queue.three", true);
    }

    /**
     * 创建 Direct 模式交换机
     */
    @Bean
    public DirectExchange directExchange() {
        /*
         * param1: 交换机名称
         * param2: durable：是否持久化，RabbitMQ关闭后，没持久化Exchange将被清除
         * param3: autoDelete：是否自动删除，，如果没有与之绑定的Queue直接删除
         */
        return new DirectExchange("direct_exchange", true, false);
    }

    /**
     * 绑定队列(不用指定routing key),参数名字要和bean名字一致
     */
    @Bean
    public Binding bindingDirect1() {
        return BindingBuilder.bind(directQueueOne()).to(directExchange()).with("1");
    }
    @Bean
    public Binding bindingDirect2() {
        return BindingBuilder.bind(directQueueTwo()).to(directExchange()).with("2");
    }
    @Bean
    public Binding bindingDirect3() {
        return BindingBuilder.bind(directQueueThree()).to(directExchange()).with("3");
    }
}
```



2、生产者发送消息

```java
    /**
     * 路由模式
     */
    @Test
    public void directSend() {
        // 路由模式，分别对不同的routingKey发送消息
        this.rabbitTemplate.convertAndSend("direct_exchange","1","Direct-One：" + new Date());
        this.rabbitTemplate.convertAndSend("direct_exchange","2","Direct-Two：" + new Date());
        this.rabbitTemplate.convertAndSend("direct_exchange","3","Direct-Three：" + new Date());
    }
```



3、消费者接收消息

```java
package com.example.service;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class DirectConsumer {

    @RabbitListener(queues = "direct.queue.one")
    public void processEmail(String name) {
        System.out.println("direct.queue.one 收到消息：" + name);
    }

    @RabbitListener(queues = "direct.queue.two")
    public void processSms(String name) {
        System.out.println("direct.queue.two 收到消息：" + name);
    }

    @RabbitListener(queues = "direct.queue.three")
    public void processWeiXin(String name) {
        System.out.println("direct.queue.three 收到消息：" + name);
    }
}
```

4、消费者收到测试结果：

```
direct.queue.one 收到消息：Direct-One测试
direct.queue.two 收到消息：Direct-Two测试
direct.queue.three 收到消息：Direct-Three测试
```





## 05、Topic 模式

1、创建队列、交换机、绑定关系

```java
package com.example.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Topic 广播模式
 **/
@Configuration
public class TopicRabbitMQConfig {
    /**
     * 声明队列
     */
    @Bean
    public Queue topicQueueOne() {
        /*
         * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
         * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
         * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
         * return new Queue("TestQueue",true,true,false);
         * 一般设置一下队列的持久化就好,其余两个就是默认false
         */
        return new Queue("topic.queue.one", true);
    }
    @Bean
    public Queue topicQueueTwo() {
        return new Queue("topic.queue.two", true);
    }
    @Bean
    public Queue topicQueueThree() {
        return new Queue("topic.queue.three", true);
    }

    /**
     * 创建 topic 模式交换器
     */
    @Bean
   public TopicExchange topicExchange(){
        /*
         * param1: 交换机名称
         * param2: durable：是否持久化，RabbitMQ关闭后，没持久化Exchange将被清除
         * param3: autoDelete：是否自动删除，，如果没有与之绑定的Queue直接删除
         */
        return new TopicExchange("topic_exchange", true, false);
    }

    /**
     * 绑定队列
     */
    @Bean
    public Binding bindingTopicOne(){
        return BindingBuilder.bind(topicQueueOne()).to(topicExchange()).with("#.error");
    }
    @Bean
    public Binding bindingTopicTwo(){
        return BindingBuilder.bind(topicQueueTwo()).to(topicExchange()).with("#.log");
    }
    @Bean
    public Binding bindingTopicThree(){
        return BindingBuilder.bind(topicQueueThree()).to(topicExchange()).with("good.#.timer");
    }
}
```



2、生产者发送消息

```java
    /**
     * 广播模式
     */
    @Test
    public void topicSend() {
        rabbitTemplate.convertAndSend("topic_exchange","test.error","主题模式测试");
        rabbitTemplate.convertAndSend("topic_exchange","test.log","主题模式测试");
        rabbitTemplate.convertAndSend("topic_exchange","good.test.timer","主题模式测试");
    }
```



3、消费者接收消息

```java
package com.example.service;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class TopicConsumer {

    @RabbitListener(queues = "topic.queue.one")
    public void processOne(String name) {
        System.out.println("topic.queue.one 收到消息：" + name);
    }

    @RabbitListener(queues = "topic.queue.two")
    public void processTwo(String name) {
        System.out.println("topic.queue.tw 收到消息：" + name);
    }

    @RabbitListener(queues = "topic.queue.three")
    public void processThree(String name) {
        System.out.println("topic.queue.three 收到消息：" + name);
    }

}
```

4、消费者收到测试结果：

```
topic.queue.one 收到消息：主题模式测试
topic.queue.two 收到消息：主题模式测试
topic.queue.three 收到消息：主题模式测试
```



## 06、Header 模式

1、创建队列、交换机、绑定关系

```java
package com.example.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.HeadersExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * Header 模式
 */
@Configuration
public class HeaderRabbitMQConfig {
    /**
     * Headers 模式
     * 设置header attribute参数类型的交换机，相较于 direct 和 topic 固定地使用 routing_key ,
     * headers 则是一个自定义匹配规则的类型. 在队列与交换器绑定时, 会设定一组键值对规则,
     * 消息中也包括一组键值对( headers 属性), 当这些键值对有一对, 或全部匹配时, 消息被投送到对应队列
     */
    @Bean
    public Queue headerQueueOne(){
        return  new Queue("header.queue.one",true);
    }
    @Bean
    public Queue headerQueueTwo(){
        return  new Queue("header.queue.two",true);
    }

    /**
     * 创建 Header 模式交换机
     */
    @Bean
    public HeadersExchange headersExchange(){
        /*
         * param1: 交换机名称
         * param2: durable：是否持久化，RabbitMQ关闭后，没持久化Exchange将被清除
         * param3: autoDelete：是否自动删除，，如果没有与之绑定的Queue直接删除
         */
        return new HeadersExchange("header_exchange", true, false);
    }

    @Bean
    public Binding headerBinding1(){
        Map<String, Object> map = new HashMap<>();
        map.put("header1","value1");
        map.put("header11","value11");
        // 匹配header中某一个即可
        return BindingBuilder.bind(headerQueueOne()).to(headersExchange()).whereAny(map).match();
    }

    @Bean
    public Binding headerBinding2(){
        Map<String, Object> map = new HashMap<>();
        map.put("header2","value2");
        map.put("header22","value22");
        // 必须全部匹配header
        return BindingBuilder.bind(headerQueueTwo()).to(headersExchange()).whereAll(map).match();
    }
}
```

2、生产者发送消息

```java
    /**
     * Headers模式
     */
    @Test
    public void headersSend() {
        MessageProperties messageProperties1 = new MessageProperties();
        messageProperties1.setHeader("header1","value1");
        messageProperties1.setHeader("header11","value11");
        Message message1 = new Message(("Header模式—单个匹配").getBytes(), messageProperties1);
        rabbitTemplate.convertAndSend("header_exchange","",message1);

        MessageProperties messageProperties2 = new MessageProperties();
        messageProperties2.setHeader("header2","value2");
        messageProperties2.setHeader("header22","value22");
        Message message2 = new Message(("Header模式—全部匹配").getBytes(), messageProperties2);
        rabbitTemplate.convertAndSend("header_exchange","",message2);
    }
```

3、消费者接收消息

```java
package com.example.service;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class HeadersConsumer {

    @RabbitListener(queues = "header.queue.one")
    public void processOne(byte[] bytes) {
        System.out.println("header.queue.one 收到消息：" + new String(bytes));
    }

    @RabbitListener(queues = "header.queue.two")
    public void processTwo(byte[] bytes) {
        System.out.println("header.queue.two 收到消息：" + new String(bytes));
    }
    
}
```

4、消费者收到测试结果：

```
header.queue.one 收到消息：Header模式—单个匹配
header.queue.two 收到消息：Header模式—全部匹配
```





# RabbitMQ 消息可靠性投递+消费

> 任何消息中间件发消息投递的可靠性都是开发者选择的重要参考依据。我们希望的是发送的每一条消息都是可以被消费者正确处理的。但是没有哪个消息中间件可以保证消息一定 100% 投递成功，那么如果消息投递失败我们该如何处理呢？
>
> 本文消息靠性介绍比较浅，深入可参考：
> 消息可靠投递(上)：https://www.cnblogs.com/mfrank/p/11380102.html
> 消息可靠投递(下)：https://www.cnblogs.com/mfrank/p/11442706.html
> https://www.cnblogs.com/ybyn/category/1849978.html



## 00、消息可靠性理论分析

![20210418110834](https://gitee.com/liusuixing/blog_images/raw/master/20210418110834.png)

> **1、RabbitMQ 消息投递路径**

生产者(Producer) ==> 交换机(Exchange) ==> 队列(Queue) ==> 消费者(Consumer)；这过程中消息可能丢失的三种情况：

1、**生产者投递-丢失消息**：Produer 发送消息到 Broker (Exchange) 失败，导致消息发送失败

2、**生产者投递-丢失消息**：Exchange 投递消息到 Queue 失败，导致消息丢

3、**消费者消费-丢失消息**：Consumer 从 Queue 中获取消息，但无法正确处理(消费)导致消息丢失



> **2、RabbitMQ 消息投递解决方案**

1、**消息事务机制**：RabbitMQ是支持AMQP事务机制的，在生产者确认机制之前，事务是确保消息被成功投递的唯一方法。

2、**confirmCallback确认模式**：处理生产者发送消息到Broker失败场景，生产者投递消息后，如果Broker收到消息后，会返回生产者一个ACK通知。生产者通过ACK可以确定这条消息是否正常发送到Broker

3、**returnCallback退回模式**：默认情况下交换机投递消息到队列失败是直接丢弃该消息，开启 returnCallback后，如果消息投递失败会通知消息生产者

4、**消息确认机制ACK**：消费者从RabbitMQ收到消息并处理完成后，反馈给RabbitMQ，RabbitMQ收到反馈后才将此消息从队列中删除。消费者可以确认消费该消息或者消费失败并放入队列中等待下次继续消费，或者直接拒绝消费该消息

> **建议：**开启消息确认机制以后，保证了消息的准确送达，但由于频繁的确认交互， rabbitmq 整体效率变低，吞吐量下降严重，不是非常重要的消息真心不建议用消息确认机制。



## 01、消息事务机制（不推荐）

事务机制能够解决生产者与broker之间消息确认的问题，只有消息成功被broker接受，事务才能提交成功，否则就进行事务回滚操作并进行消息重发。但是使用事务机制会降低RabbitMQ的消息吞吐量，不适用于需要发布大量消息的业务场景。**注意：事务是同步的，发送消息之后，等到接收到确认返回之后，才能发送下一条消息。**事务的实现主要是对信道（channel）的设置，主要的方法有三个：

1. channel.txSelect()：声明启动事务模式；
2. channel.txComment()：提交事务；
3. channel.txRollback()：回滚事务；

完整代码：

```java
    try {
        channel.txSelect();
        channel.basicPublish(exchange, routingKey, null, msg.getBytes());
        int result = 1 / 0;
        channel.txCommit();
    } catch (Exception e) {
        channel.txRollback();
    }
```

RabbitMQ中事务的多了四个步骤：

```
client发送Tx.Select
broker发送Tx.Select-Ok(之后publish)
client发送Tx.Commit
broker发送Tx.Commit-Ok
```

如果发生异常事务就会滚，解决了在未到达交换机之前就出现错误，只有消息被成功接收事务才能提交成功否则我们便可以在捕捉异常后进行回滚重发，但是使用事务机制的话会降低rabbit的性能所有mq提供了一个更好的方案！



**SpringBoot Config 类配置**

```java
package com.example.config;

import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.transaction.RabbitTransactionManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TransactionRabbitMQConfig {
    @Bean
    public ConnectionFactory connectionFactory(){
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setPort(5672);
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        connectionFactory.setVirtualHost("/");
        return connectionFactory;
    }

    @Bean
    public RabbitTransactionManager rabbitTransactionManager(ConnectionFactory connectionFactory){
        return new RabbitTransactionManager(connectionFactory);
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setChannelTransacted(true);// 开启事务
        return rabbitTemplate;
    }
}
```

测试

```java
    /**
     * 消息可靠性投递：事务机制
     * 如果返回异常则已经发送的消息会回滚
     */
    @Test
    @Transactional(transactionManager = "rabbitTransactionManager")
    public void transactionTest(){
        rabbitTemplate.convertAndSend("simple.queue", "test transaction");
        int nextInt = new Random().nextInt(2);
        System.out.println(nextInt);
        int i = 1/nextInt;// 模拟异常
    }
```

查看simple.queue的情况可以发现，如果发生异常，该消息是不会发送成功。可以尝试注释`@Transactional`，那么每次发送都会进入到simple.queue队列，不管后面是否发生异常。



## 02、消息确认模式（消息发送方）

### 02-1、JAVA 实现

1.普通Confirm方式

```java
package com.example.confirm_return;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * @author:
 * @description: Producer 简单队列生产者
 */
public class ProducerConfirm1 {
    public static void main(String[] args) throws Exception {

        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 从连接工厂中获取
        Connection connection = connectionFactory.newConnection("生产者");
        // 3: 从连接中打开通道channel
        Channel channel = connection.createChannel();

        // 4: 开启 confirm
        channel.confirmSelect();

        // 5: 创建交换机
         channel.exchangeDeclare("confirm-exchange", BuiltinExchangeType.DIRECT);

        // 6: 尝试发送给一个存在的exchange和一个不存在的exchange，confirm-exchange、confirm-exchange-no
        channel.basicPublish("confirm-exchange-no", "", null, "你好，消息队列！".getBytes());
        // 7: 查询是否发送成功，失败会怕抛异常
        if(channel.waitForConfirms()){
            System.out.println("生产者发布消息成功！");
        }
        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```



2.批量Confirm方式

```java
package com.example.confirm_return;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * @author:
 * @description: Producer 简单队列生产者
 */
public class ProducerConfirm2 {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 从连接工厂中获取
        Connection connection = connectionFactory.newConnection("生产者");
        // 3: 从连接中打开通道channel
        Channel channel = connection.createChannel();

        // 4: 开启 confirm
        channel.confirmSelect();

        // 5: 创建交换机
        channel.exchangeDeclare("confirm-exchange", BuiltinExchangeType.DIRECT);

        for (int i = 0; i < 10; i++) {
            // 6: 尝试发送给一个存在的exchange和一个不存在的exchange，confirm-exchange、confirm-exchange-no
            channel.basicPublish("confirm-exchange", "", null, ("你好，消息队列！"+i).getBytes());
        }
        // 7: 当你发送的全部消息，有一个失败时，就直接全部失败 并抛出IOException
        channel.waitForConfirmsOrDie();
        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```



3.异步Confirm方式

```java
package com.example.confirm_return;

import com.rabbitmq.client.*;
import java.util.concurrent.TimeUnit;

/**
 * @author:
 * @description: Producer 简单队列生产者
 */
public class ProducerConfirm3 {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 从连接工厂中获取
        Connection connection = connectionFactory.newConnection("生产者");
        // 3: 从连接中打开通道channel
        Channel channel = connection.createChannel();

        // 4: 生产者调用confirmSelect 将 channel 设置为 confirm 模式
        channel.confirmSelect();
        // 5: 开启异步回调,查询是否发送成功，失败会怕抛异常
        channel.addConfirmListener(new ConfirmListener() {
            // 成功回调
            @Override
            public void handleAck(long deliveryTag, boolean multiple) {
                System.out.println("消息发送成功，标识：" + deliveryTag + ",是否是批量" + multiple);
            }
            // 失败回调
            @Override
            public void handleNack(long deliveryTag, boolean multiple) {
                System.out.println("消息发送失败，标识：" + deliveryTag + ",是否是批量" + multiple);
            }
        });

        // 6: 创建交换机
        channel.exchangeDeclare("confirm-exchange", BuiltinExchangeType.DIRECT);

        for (int i = 0; i < 10; i++) {
            // 7: 尝试发送给一个存在的exchange和一个不存在的exchange，confirm-exchange、confirm-exchange-no
            channel.basicPublish("confirm-exchange", "", null, ("你好，消息队列！"+i).getBytes());
        }

        // 让主线程休眠，可以每次都看到控制台输出
        TimeUnit.MILLISECONDS.sleep(1000);

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```



### 02-2、SpringBoot 实现

application.properties：开启 ConfirmCallback 配置

```properties
# NONE值是禁用发布确认模式，是默认值，CORRELATED值是发布消息成功到交换器后会触发回调方法，还能选择simple
spring.rabbitmq.publisher-confirm-type=correlated
```

生产者配置代码：

```java
/**
 * 生产者投递消息后，如果Broker收到消息后，会给生产者一个ACK。
 * 生产者通过ACK，可以确认这条消息是否正常发送到Broker，这种方式是消息可靠性投递的核心
 * yaml文件中添加配置 spring.rabbitmq.publisher-confirm-type=correlated
 */
@PostConstruct
public void setConfirmCallback() {
    rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
        /**
         * @param correlationData 发送消息时指定的唯一关联数据（消息id）
         * @param ack             投递结果
         * @param cause           失败原因
         */
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
            if (ack) {
                log.info("消息投递到交换机成功：[correlationData={}]", correlationData);
            } else {
                log.error("消息投递到交换机失败：[correlationData={}，原因={}]", correlationData, cause);
            }
        }
    });
}
```

生产者发送测试：

```java
    @Autowired
    RabbitTemplate rabbitTemplate;
    @Test
    public void confirmTest(){
        // 消息内容
        Map<String,String> map = new HashMap<>();
        map.put("message","testing confirm function");
        // 设置自定义反馈消息
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId(UUID.randomUUID().toString());
        // 发送到并不存在名为“exchange-not”的exchange
        rabbitTemplate.convertAndSend("exchange-not","",map,correlationData);
    }
```

控制台输出：

```
2021-05-28 13:35:24.532 ERROR 9028 --- [nectionFactory2] c.e.c.ConfirmAndReturnCallbackConfig     : 
消息投递到交换机失败：[correlationData=CorrelationData [id=13e952a8-98e5-4eb2-831e-ffb553c48cea]，原因=channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'exchange-dog' in vhost '/', class-id=60, method-id=40)]
```





## 03、消息回退模式（消息发送方）

> Confirm只能保证消息到达 exchange，无法保证消息可以被 exchange分发到指定 queue。而且 exchange是不能持久化消息的，queue是可以持久化消息。采用 return 机制来监听消息是否从 exchange送到了指定的 queue中

![20210418110849](https://gitee.com/liusuixing/blog_images/raw/master/20210418110849.png)

当把 mandotory 参数设置为 true 时，如果交换机无法将消息进行路由时，会将该消息返回给生产者，而如果该参数设置为false，如果发现消息无法进行路由，则直接丢弃。



### 03-1、JAVA + SpringBoot 实现

> Java 示例代码：

```java
package com.example.confirm_return;

import com.rabbitmq.client.*;
import java.io.UnsupportedEncodingException;

/**
 * @author:
 * @description: Producer 简单队列生产者
 */
public class ProducerReturn {
    public static void main(String[] args) throws Exception {

        // 1: 创建连接工厂，设置连接属性
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 从连接工厂中获取
        Connection connection = connectionFactory.newConnection("生产者");
        // 3: 从连接中打开通道channel
        Channel channel = connection.createChannel();

        // 开启return机制，并在发送消息时，指定mandatory为true
        channel.addReturnListener(new ReturnListener() {
            @Override
            public void handleReturn(int replyCode, String replyText,
                                     String exchange, String routingKey,
                                     AMQP.BasicProperties properties,
                                     byte[] body) throws UnsupportedEncodingException {
                // 当消息没有送达到queue时，才会执行。
                System.out.println(new String(body,"UTF-8") + "没有送达到Queue中！！");
            }
        });

        // 6: 参数3为mandatory：开启return，发送到一个不存在queue
        channel.basicPublish("", "xxx", true, null, "你好，消息队列！".getBytes());

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```



> SpringBoot 示例代码：

1、修改application.properties：开启 returnCallback 配置

```properties
# 可以确保消息在未被队列接收时返回，如何配置了mandatory=true，可以不用配置publisher-returns
# spring.rabbitmq.publisher-returns=true

# 指定消息在没有被队列接收时是否强行退回还是直接丢弃。true退回，false丢弃.
# 或者代码rabbitTemplate.setMandatory(true);也可以
spring.rabbitmq.template.mandatory=true
```

**注意：publisher-returns 与 mandatory 配置的区别：**

spring.rabbitmq.template.mandatory属性的优先级高于spring.rabbitmq.publisher-returns的优先级
spring.rabbitmq.template.mandatory属性可能会返回三种值null、false、true,
spring.rabbitmq.template.mandatory结果为true、false时会忽略掉spring.rabbitmq.publisher-returns属性的值
spring.rabbitmq.template.mandatory结果为null（即不配置）时结果由spring.rabbitmq.publisher-returns确定

2、示例代码：

```java
/**
 * 注意下面两项必须同时配置：
 * spring.rabbitmq.publisher-returns=true
 * spring.rabbitmq.template.mandatory=true
 */
@PostConstruct
public void setQueueCallback() {
    // rabbitTemplate.setMandatory(true); // 配置文件mandatory和代码配置其一即可
    rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
        /**
         * @param returnedMessage 所有返回的信息都在里面
         */
        @Override
        public void returnedMessage(ReturnedMessage returnedMessage) {
            log.error("路由到队列失败，[消息内容：{}，交换机：{}，路由件：{}，回复码：{}，回复文本：{}]",
                    returnedMessage.getMessage(),
                    returnedMessage.getExchange(),
                    returnedMessage.getRoutingKey(),
                    returnedMessage.getReplyCode(),
                    returnedMessage.getReplyText());
        }
    });
}
```

3、测试代码：

```java
    @Autowired
    RabbitTemplate rabbitTemplate;
    @Test
    public void returnTest(){
        // 并不存在名为“dog”的routingKey，即投不到现有的queue里
        rabbitTemplate.convertAndSend("not_exist_queue","")
    }
```

4、控制台输出：

```
2021-05-28 13:50:02.004 ERROR 8692 --- [nectionFactory1] c.e.c.ConfirmAndReturnCallbackConfig     : 路由到队列失败，[消息内容：(Body:'' MessageProperties [headers=(Body:'' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0]), contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0])，交换机：，路由件：not_exist_queue，回复码：312，回复文本：NO_ROUTE]
```

可以看到，我们接收到了被退回的消息，并带上了消息被退回的原因：`NO_ROUTE`。但是要注意的是， mandatory 参数仅仅是在当消息无法被路由的时候，让生产者可以感知到这一点，只要开启了生产者确认机制，无论是否设置了 mandatory 参数，都会在交换机接收到消息时进行消息确认回调，而且通常消息的退回回调会在消息的确认回调之前。



### 03-2、备用交换机开启

有了 mandatory 参数，我们获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理。但有时候，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。

而且设置 mandatory 参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑。如果既不想丢失消息，又不想增加生产者的复杂性，该怎么做呢？

前面在设置死信队列的文章中，我们提到，可以为队列设置死信交换机来存储那些处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。

不要慌，在 RabbitMQ 中，有一种备份交换机的机制存在，可以很好的应对这个问题。

什么是备份交换机呢？备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当我们为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会将这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。

听的不太明白？没关系，看个图就知道是怎么回事了。

![20210418110850](https://gitee.com/liusuixing/blog_images/raw/master/20210418110850.png)

接下来，我们就来设置一下备份交换机（建议生产者和消费者端同时定义）：

```java
package com.example.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BackupRabbitMQConfig {
    public static final String BUSINESS_EXCHANGE_NAME = "backup.business-exchange";
    public static final String BUSINESS_QUEUE_NAME = "backup.business-queue";
    public static final String BACKUP_EXCHANGE_NAME = "backup.backup-exchange";
    public static final String BACKUP_QUEUE_NAME = "backup.backup-queue";
    public static final String BACKUP_WARNING_QUEUE_NAME = "backup.backup-warning-queue";

    // 声明业务 Exchange
    @Bean("businessExchange")
    public DirectExchange businessExchange(){
        ExchangeBuilder exchangeBuilder = ExchangeBuilder.directExchange(BUSINESS_EXCHANGE_NAME)
                .withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME)
                .durable(true);

        return (DirectExchange)exchangeBuilder.build();
    }

    // 声明业务队列
    @Bean("businessQueue")
    public Queue businessQueue(){
        return QueueBuilder.durable(BUSINESS_QUEUE_NAME).build();
    }

    // 声明业务队列绑定关系
    @Bean
    public Binding businessBinding(@Qualifier("businessQueue") Queue queue,
                                   @Qualifier("businessExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with("key");
    }

    // 声明备份 Exchange
    @Bean("backupExchange")
    public FanoutExchange backupExchange(){
        ExchangeBuilder exchangeBuilder = ExchangeBuilder.fanoutExchange(BACKUP_EXCHANGE_NAME)
                .durable(true);
        return (FanoutExchange)exchangeBuilder.build();
    }

    // 声明备份队列
    @Bean("backupQueue")
    public Queue backupQueue(){
        return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }

    // 声明备份队列绑定关系
    @Bean
    public Binding backupBinding(@Qualifier("backupQueue") Queue queue,
                                 @Qualifier("backupExchange") FanoutExchange exchange){
        return BindingBuilder.bind(queue).to(exchange);
    }

    // 声明报警队列
    @Bean("warningQueue")
    public Queue warningQueue(){
        return QueueBuilder.durable(BACKUP_WARNING_QUEUE_NAME).build();
    }

    // 声明备份报警队列绑定关系
    @Bean
    public Binding backupWarningBinding(@Qualifier("warningQueue") Queue queue,
                                        @Qualifier("backupExchange") FanoutExchange exchange){
        return BindingBuilder.bind(queue).to(exchange);
    }

}
```

这里我们使用 `ExchangeBuilder` 来创建交换机，并为其设置备份交换机：

```java
.withArgument("alternate-exchange", BUSINESS_BACKUP_EXCHANGE_NAME);
```

为业务交换机绑定了一个队列，为备份交换机绑定了两个队列，一个用来存储不可投递消息，待之后人工处理，一个专门用来做报警用途。

接下来，分别为业务交换机和备份交换机创建消费者：

```java
package com.example.service;

import com.example.config.BackupRabbitMQConfig;
import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.io.IOException;

@Slf4j
@Component
public class BackupConsumer {
    /**
     * 正常可以消费的业务队列
     */
    @RabbitListener(queues = BackupRabbitMQConfig.BUSINESS_QUEUE_NAME)
    public void receiveMsgA(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
        log.info("业务队列-收到业务消息：{}", msg);
        channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
    }

    /**
     * 备份警告队列：
     * 由于是 Fanout 模式。所以备份队列和警报队列都会收到消息。
     * 这里只监听了备份交换机中的警报队列，没有监听备份队列。
     * 可以在Web端查看到 backup.backup-queue 队列会有消息没有消费
     */
    @RabbitListener(queues = BackupRabbitMQConfig.BACKUP_WARNING_QUEUE_NAME)
    public void receiveMsgB(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
        log.error("备份警告队列-发现不可路由消息：{}", msg);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
}
```

接下来我们在发送端代码中分别发送一条可用路由消息和不可用路由消息 测试：

```java
    /**
     * 消息回退模式 + 备份交换机模式
     * 分别发送一条可路由消息和不可路由消息：
     */
    @Test
    public void backupTest(){
        // 发送可用路由
        log.info("消息id:{}, msg:{}", "1", "发送到存在的队列上");
        rabbitTemplate.convertAndSend(BackupRabbitMQConfig.BUSINESS_EXCHANGE_NAME,
                "key", "发送到存在的队列上", new CorrelationData("1"));

        // 发送不可用用路由，这样会直接转发到备用交换机中去（然后分发到指定的路由中）
        log.info("消息id:{}, msg:{}", "2", "发送到不存在的队列上");
        rabbitTemplate.convertAndSend(BackupRabbitMQConfig.BUSINESS_EXCHANGE_NAME,
                "key2","发送到不存在的队列上",new CorrelationData("2"));
    }
```

生产者查看控制台输出：

```
2021-06-02 11:39:44.691  INFO 24260 --- [main] : 消息id:1, msg:发送到存在的队列上
2021-06-02 11:39:44.911  INFO 24260 --- [main] : 消息id:2, msg:发送到不存在的队列上
```

消费者查看控制台输出：

```
2021-06-02 11:48:10.755  INFO 3452 --- [tContainer#11-1] : 业务队列-收到业务消息：发送到存在的队列上
2021-06-02 11:48:10.755 ERROR 3452 --- [tContainer#12-1] : 备份警告队列-发现不可路由消息：发送到不存在的队列上
```

这里仅仅使用 error 日志配合日志系统进行报警，如果是敏感数据，可以使用邮件、钉钉、短信、电话等报警方式来提高时效性。



> **回退模式（开启mandatory参数）与 备用交换机同时使用：**
>
> 设置 mandatory 参数会让交换机将不可路由消息退回给生产者，而备份交换机会让交换机将不可路由消息转发给它，那么如果两者同时开启，两条消息都可以收到确认成功回调，但是**不可用路由消息不会被回退给生产者，而是直接转发给备份交换机。可见备份交换机的处理优先级更高。**





## 04、消息确认机制 ACK（消息接收方）

> **消息确认机制ACK介绍：**
>
> - 消费者从RabbitMQ收到消息并处理完成后，反馈给RabbitMQ，RabbitMQ收到反馈后才将此消息从队列中删除
> - 消费者在处理消息出现了网络不稳定、服务器异常等现象，那么就不会有ACK反馈，RabbitMQ会认为这个消息没有正常消费，会将消息重新放入队列中
> - 只有当消费者正确发送ACK反馈，RabbitMQ确认收到后，消息才会从RabbitMQ服务器的数据中删除。
> - 消息的ACK确认机制默认是打开的，消息如未被进行ACK的消息确认机制，这条消息被锁定Unacked
>
> **AMQP 0-9-1 有两种消息 ACK 模式：**
>
> - 自动 ACK 模式
> - 手动 ACK 模式

RabbitMQ中消费者有3种手动确认消息方式：

- basicAck：表示成功确认，使用此回执方法后，消息会被`broker` 删除。

- basicNack ：表示失败确认，一般在消费消息业务异常时用到此方法，可以将消息重新投递入队列。

- basicReject：拒绝消息，与`basicNack`区别在于不能进行批量操作，其他用法很相似。



**开启手动确认消费配置 + 代码实战**

> 消息接收确认要比消息发送确认简单一点，因为只有一个消息回执（ack）的过程。Java的话直接使用使用`channel`操作即可。 SpringBoot的话直接在`@RabbitHandler`注解标注的方法要增加 `channel`(信道)、`message` 两个参数。

```properties
# 开启手动确认消息，如果消息重新入队，可以配置是否需要重试：retry.enabled=true
spring.rabbitmq.listener.simple.acknowledge-mode: manual
```

```java
package com.example.service;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.io.IOException;

@Component
public class AckConsumer {
    @RabbitListener(queues = "ack.queue")
    public void processOne(String msg, Message message, Channel channel) throws IOException {
        // 该条消息的消息编号，Long类型，递增的
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            /**
             * 处理成功 手动ACK回执
             * deliveryTag：表示消息投递序号，每次消费消息或者消息重新投递后，deliveryTag都会增加。
             *              手动消息确认模式下，我们可以对指定deliveryTag的消息进行ack、nack、reject等操作。
             * multiple：是否批量确认，值为 true 则会一次性 ack所有小于当前消息 deliveryTag 的消息。
             *           假设我先发送三条消息deliveryTag分别是5、6、7，可它们都没有被确认，
             *           当我发第四条消息此时deliveryTag为8，multiple设置为 true，会将5、6、7、8的消息全部进行确认
             */
            channel.basicAck(deliveryTag, false);
        } catch (Exception e) {
            // 当消息处理异常时，将消息重新放回队列，重新排队。如下两种选择其一即可
            /**
             * deliveryTag：消息编号 递增的
             * multiple：是否批量处理
             * requeue：被拒绝的是否重新入队列. true:让消息重新回到队列，false:直接丢弃
             */
            channel.basicNack(deliveryTag, false, false);
            /**
             * deliveryTag：消息编号 递增的
             * requeue：被拒绝的是否重新入队列. true:让消息重新回到队列，false:直接丢弃
             */
            channel.basicReject(deliveryTag,false);
        }
    }
}
```



## 05、消息重试机制（消息接收方）

> 参考：https://www.cnblogs.com/ybyn/p/13691058.html

```properties
# 消费者自动ack
spring.rabbitmq.listener.simple.acknowledge-mode=auto
# 开启支持重试
spring.rabbitmq.listener.simple.retry.enabled=true
# 重试次数是5次（包含自身消费的一次）
spring.rabbitmq.listener.simple.retry.max-attempts=5
# 重试最大间隔时间
spring.rabbitmq.listener.simple.retry.max-interval=10000
# 重试初始间隔时间
spring.rabbitmq.listener.simple.retry.initial-interval=2000
# 间隔时间乘子，间隔时间*乘子=下一次的间隔时间，最大不能超过设置的最大间隔时间
spring.rabbitmq.listener.simple.retry.multiplier=2
```

可以看到重试次数是5次（包含自身消费的一次），重试时间依次是2s，4s，8s，10s（上一次间隔时间*间隔时间乘子），最后一次重试时间理论上是16s，但是由于设置了最大间隔时间是10s，因此最后一次间隔时间只能是10s，和配置相符合。

> 注意：
>
> 重试并不是RabbitMQ重新发送了消息，仅仅是消费者内部进行的重试，换句话说就是重试跟mq没有任何关系；
>
> 因此上述消费者代码不能添加try{}catch(){}，一旦捕获了异常，在自动ack模式下，就相当于消息正确处理了，消息直接被确认掉了，不会触发重试的；

**TODO：还可以继续研究**





## 06、消息限流策略（消息接收方）

1：为什么要限流：若队列中消息积压过多，突然开启监听，会导致消费端崩溃。

2：如何限流：使用RabbitMQ提供的Qos（服务质量保证）功能，如果一定数目消息的未被应答前，不再接受新消息。

3：代码测试：**需要开启手动应答模式**

```properties
# 开启消费者手动消息应答
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

```java
package com.example.service;

import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.io.IOException;

@Slf4j
@Component
public class QosConsumer {
    @RabbitListener(queues = "qos-queue")
    public void revice(String msg, Message message, Channel channel) throws IOException {
        try {
            log.info("消息ID：" + message.getMessageProperties().getHeader("spring_returned_message_correlation"));
            log.info("消息标签：" + message.getMessageProperties().getDeliveryTag());
            /**
             * 设置Qos机制：
             * 参数 1 prefetchSize：单条消息的大小(0表示即无限制)
             * 参数 2 prefetchCount：每次处理消息的数量
             * 参数 3 global：是否为consumer级别（false表示仅当前channel有效）
             */
            channel.basicQos(0, 1, false);
            //手动应答消息　　第一个参数是所确认消息的标识，第二参数是是否批量确认
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
            log.error("消息ID：" + message.getMessageProperties().getHeader("spring_returned_message_correlation"));
            log.error("接收消息发送错误：" + e.getMessage());
        }
    }
}
```



## XX、消息可靠性踩坑日志

> 参考：https://juejin.cn/post/6844904205438681095

### 01、消息不确认

开启消息确认机制，消费消息别忘了`channel.basicAck`，否则消息会一直存在，导致重复消费。**这是一个非常没技术含量的坑，但却是非常容易犯错的地方。**



### 02、消息无限投递

在我最开始接触消息确认机制的时候，消费端代码就像下边这样写的，思路很简单：处理完业务逻辑后确认消息， `int a = 1 / 0` 发生异常后将消息重新投入队列。

```java
    @RabbitHandler
    public void processHandler(String msg, Channel channel, Message message) throws IOException {
        try {
            log.info("消费者 1 号收到：{}", msg);
            int a = 1 / 0;
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        }
    }
```

但是有个问题是，业务代码一旦出现 `bug` 99.9%的情况是不会自动修复，一条消息会被无限投递进队列，消费端无限执行，导致了死循环。

**TODO：还可继续深入**



# RabbitMQ 延迟队列+死信队列

> 延迟队列：https://www.cnblogs.com/mfrank/p/11260355.html
> 死信队列：https://www.cnblogs.com/mfrank/p/11184929.html

**延迟队列+死信队列 大致流程图：**

![20210418110848](https://gitee.com/liusuixing/blog_images/raw/master/20210418110848.png)

## 01、延迟队列 TTL

### 01-1、什么是延迟队列

讨论什么是延迟队列之前我们得先知道什么是TTL：time to live 消息存活时间。

延迟队列：实际上不存在真正意义上的延迟队列（简单说就是在普通队列上加过期时间就是延迟队列）。但是我们可以通过设置消息的过期时间和死信队列来模拟出延时队列。消费者监听死信交换器绑定的队列，而不要监听消息发送的队列。



### 01-2、延迟队列的使用场景

> 1. 订单在十分钟之内未支付则自动取消。
> 2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
> 3. 账单在一周内未支付，则自动结算。
> 4. 用户注册成功后，如果三天内没有登陆则进行短信提醒。
> 5. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。
> 6. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议。

这些看起来似乎都可以使用定时服务解决，确实如果在数据量不大或者时间精确度不是那么注重的情况下是可以使用定时服务解决。不过在数据量大对失效时间要求准确的情况下建议使用RabbitMQ 延迟队列 + 死信队列来解决。



### 01-3、RabbitMQ-TTL 概述

过期时间TTL表示可以对消息设置预期的时间，在这个时间内都可以被消费者接收获取；过了之后消息将自动被删除。

消息在队列的生存时间一旦超过设置的TTL值，就称为`Dead message` 被投递到死信队列， 消费者将无法再收到该消息。

RabbitMQ 支持对 **消息 和 队列** 设置TTL。目前有两种方法可以设置：

- 第一种：是通过队列设置，队列中所有消息都有相同的过期时间（延迟队列）（居多）
- 第二种：是通过消息设置，每条消息TTL可以不同

**如果上述两种同时使用，则消息的过期时间以两者之间TTL较小的那个数值为准。**



第一种是在创建队列的时候设置队列的“x-message-ttl”属性，如下：

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 6000);
channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);
```

第二种是对每条消息设置TTL，代码如下：

```java
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
    .expiration("6000")
    .build();
channel.basicPublish(exchangeName, routingKey, mandatory, properties, "msg body".getBytes());
```

但这两种方式是有区别的，**如果设置了队列的TTL属性，那么一旦消息过期，就会被队列丢弃，而第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间。**

另外，还需要注意的一点是，如果不设置TTL，表示消息永远不会过期，如果将TTL设置为0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。



### 01-4、消息过期代码实战

```java
package com.example.ttl;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import java.util.HashMap;
import java.util.Map;

/**
 * @author:
 * @description: Producer 简单队列生产者
 */
public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性，2: 从连接工厂中获取，3: 从连接中打开通道channel
        ConnectionFactory connectionFactory = new ConnectionFactory();
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 方式一：队列设置TTL
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-message-ttl",5000);// 设置队列过期时间TTL
        channel.queueDeclare("ttl-queue", false, false, false, arguments);

        // 方式二：设置消息的过期时间。一般使用其一即可。两者同时使用时按照短的时间过期
        AMQP.BasicProperties props = new AMQP.BasicProperties().builder()
                .expiration("3000") // 消息过期时间
                .build();
        // param1:交换机名，param2: 队列名/路由key，param3:属性配置附加参数，param4:消息内容
        channel.basicPublish("", "ttl-queue", props, "你好，消息队列！".getBytes());
        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```

然后可以从web页面查看该队列状况：1、可以发现队列有一个TTL的标签。2、三秒过后消息不管有没有被消费都会自动消失

![20210418110836](https://gitee.com/liusuixing/blog_images/raw/master/20210418110836.png)





SpringBoot 设置 **队列** or **消息** 的过期时间

创建队列、交换机、绑定关系

```java
package com.example.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * 过期时间+延迟队列
 **/
@Configuration
public class TTLRabbitMQConfig {
    /********************************对队列设置过期时间*****************************************/
    /**
     * 1. 声明 Direct 模式交换机
     */
    @Bean
    public DirectExchange ttlDirectExchange() {
        // param1: 交换机名、param2: 是否持久化、param3: 是否自动删除（无Queue与其绑定）
        return new DirectExchange("ttl_direct_exchange", true, false);
    }

    /**
     * 2. 队列过期时间
     */
    @Bean
    public Queue ttlDirectQueue() {
        /*
         * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
         * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
         * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
         * arguments:设置队列的属性参数
         *   x-message-ttl：设置过期时间,注意是int类型
         */
        Map<String, Object> args = new HashMap<>();
        args.put("x-message-ttl", 5000);
        return new Queue("ttl.direct.queue", true, false, false, args);
    }

    /**
     * 3. 绑定队列(不用指定routing key),参数名字要和bean名字一致
     */
    @Bean
    public Binding bindingTTLDirect() {
        return BindingBuilder.bind(ttlDirectQueue()).to(ttlDirectExchange()).with("ttl");
    }

    /********************************对消息本身设置过期时间*****************************************/
    /**
     * 1. 用上面声明的交换机
     * 2. 声明一个普通的队列
     */
    @Bean
    public Queue ttlMessageDirectQueue() {
        return new Queue("ttl.message.direct.queue", true);
    }

    /**
     * 3. 绑定队列(不用指定routing key),参数名字要和bean名字一致
     */
    @Bean
    public Binding bindingTTLMessageDirect() {
        return BindingBuilder.bind(ttlMessageDirectQueue()).to(ttlDirectExchange()).with("ttl.message");
    }
}
```

生产者发送消息

```java
    /**
     * TTL过期时间测试——对队列设置过期时间
     * 测试发送后查看web页面，5秒后消息自动消失了
     */
    @Test
    public void ttlQueueDirectSend() {
        rabbitTemplate.convertAndSend("ttl_direct_exchange","ttl","TTL-队列过期时间");
    }

    /**
     * TTL过期时间测试——对消息本身设置过期时间
     * 测试发送后查看web页面，5秒后消息自动消失了
     */
    @Test
    public void ttlMessageDirectSend() {
        MessagePostProcessor messagePostProcessor = message -> {
            MessageProperties messageProperties = message.getMessageProperties();
            messageProperties.setExpiration("5000");//字符串类型
            messageProperties.setContentEncoding("UTF-8");
            return message;
        };
        rabbitTemplate.convertAndSend("ttl_direct_exchange","ttl.message","TTL-消息过期时间", messagePostProcessor);
    }
```



### 01-5、延迟队列存在的问题

通过队列 TTL 缺点：每增加一个时间需求，就需要增一个队列。这样不太适用动态设置过期时间。

那么我们可以通过对消息本身设置过期时间，可是这也存在一个缺点（提前给结论）：**消息的过期是"同步"的**。

**解释**：第一条消息过期时间是20s，第二条是2s。第二个消息会在等第一个消息成为死信后才会“死亡“。因为这是**在同一个队列，必须前一个消费，第二个才能消费**，所以就出现了**时序问题**。



> **实战代码**

1、定义队列和交换机，并且绑定

```java
package com.example.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 过期时间+延迟队列
 **/
@Configuration
public class TTLMessagesRabbitMQConfig {

    /********************************对消息本身设置过期时间*****************************************/
    // 1. 声明 fanout 模式交换机
    @Bean
    public FanoutExchange ttlMessagesExchange(){
        return ExchangeBuilder.fanoutExchange("ttl-message.fanout-exchange").build();
    }

    // 2. 声明一个普通的队列，并设置消息
    @Bean
    public Queue ttlMessagesQueue() {
        return QueueBuilder.durable("ttl-messages.queue")
                .withArgument("x-dead-letter-exchange", "dead-ttl-message.fanout-exchange")
                .build();
    }

    // 3. 绑定队列与交换机关系
    @Bean
    public Binding bindingTTLMessagesFanout() {
        return BindingBuilder.bind(ttlMessagesQueue()).to(ttlMessagesExchange());
    }

    /********************************声明死信交换机和队列*****************************************/
    // 1. 声明死信交换机-Fanout模式
    @Bean
    public FanoutExchange deadFanoutExchange(){
        return ExchangeBuilder.fanoutExchange("dead-ttl-message.fanout-exchange").build();
    }

    // 2. 声明一个死信队列
    @Bean
    public Queue deadTTLMessagesQueue() {
        return QueueBuilder.durable("dead-ttl-messages.queue").build();
    }

    // 3. 绑定死信队列与死信交换机关系
    @Bean
    public Binding bindingDeadTTLMessagesFanout() {
        return BindingBuilder.bind(deadTTLMessagesQueue()).to(deadFanoutExchange());
    }

}
```

2、消费者监听消费：

```java
package com.example.service;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class DeadTTLMessagesConsumer {
    // 监听死信队列
    @RabbitListener(queues = "dead-ttl-messages.queue")
    public void processOne(String msg) {
        System.out.println("死信队列收到消息：" + msg);
    }
}
```

3、发送者发送消息：

```java
    /**
     * 测试对消息设置过期时间。
     * 先发送一条20s过期的消息，然后再发送一条2s过期的消息
     * 可以发现，第一条消息过期时间是20s，第二条是2s。第二个消息会在等第一个消息成为死信后才会“死亡“。
     * 结论：消息的过期是"同步"的。
     */
    @Test
    public void ttlMessagesTest(){
        rabbitTemplate.convertAndSend("ttl-message.fanout-exchange","","20s过期的消息", message -> {
            message.getMessageProperties().setExpiration("20000");
            return message;
        });
        rabbitTemplate.convertAndSend("ttl-message.fanout-exchange","","2s过期的消息", message -> {
            message.getMessageProperties().setExpiration("2000");
            return message;
        });
    }
```

4、消费端控制台输出：

```
死信队列收到消息：20s过期的消息
死信队列收到消息：2s过期的消息
```

> 可以发现TTL时间短的比TTL时间长的后“死亡”



### 02-6、利用插件实现动态TTL

> **Delayed Message 插件 及实现原理**

上文的问题确实是一个硬伤，如果不能实现在消息粒度上添加TTL，并使其在设置的TTL时间及时死亡，就无法设计成一个通用的延时队列。针对消息无序的不妨看下以下解决方案。

![20210418110856](https://gitee.com/liusuixing/blog_images/raw/master/20210418110856.png)

上面使用 DLX + TTL 的模式，消息首先会路由到一个正常的队列，根据设置的 TTL 进入死信队列，与之不同的是**通过 x-delayed-message 声明的交换机，它的消息在发布之后不会立即进入队列，先将消息保存至 Mnesia**（一个分布式数据库管理系统，适合于电信和其它需要持续运行和具备软实时特性的 Erlang 应用。目前资料介绍的不是很多）

这个插件将会尝试确认消息是否过期，首先要确保消息的延迟范围是 Delay > 0, Delay =< ?ERL_MAX_T（在 Erlang 中可以被设置的范围为 (2^32)-1 毫秒），如果消息过期通过 x-delayed-type 类型标记的交换机投递至目标队列，整个消息的投递过程也就完成了。




> **插件安装**

RabbitMQ的延迟插件下载并安装：https://www.rabbitmq.com/community-plugins.html

下载`rabbitmq_delayed_message_exchange`插件（对应RabbitMQ版本），解压到RabbitMQ的插件（plugins）目录

进入RabbitMQ安装目录的sbin目录下，使用如下命令启用延迟插件，然后重启RabbitMQ

```bash
# 启动delayed-message插件：
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
# 重启RabbitMQ
rabbitmqctl stop_app && rabbitmqctl start_app

# Liunx环境下下载
# wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.8.9/rabbitmq_delayed_message_exchange-3.8.9-0199d11c.ez
```

出现如下信息代表成功：

```bash
C:\Users\lenovo>rabbitmq-plugins enable rabbitmq_delayed_message_exchange
Enabling plugins on node rabbit@LAPTOP-671C76TJ:
rabbitmq_delayed_message_exchange
The following plugins have been configured:
  rabbitmq_delayed_message_exchange
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@LAPTOP-671C76TJ...
The following plugins have been enabled:
  rabbitmq_delayed_message_exchange

started 1 plugins.
```



> **代码实战**

1、定义交换机和队列，并绑定关系

```java
package com.example.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * 延迟队列插件
 **/
@Configuration
public class TTLDelayedPluginRabbitMQConfig {

    // 1. 延迟插件消息队列所绑定的交换机
    @Bean
    public CustomExchange customExchange(){
        // 创建一个自定义交换机，可以发送延迟消息
        Map<String, Object> args = new HashMap<>();
        args.put("x-delayed-type", "direct");
        // 参数二为类型：必须是x-delayed-message
        return new CustomExchange("ttl-delayed.exchange", "x-delayed-message", true, false, args);
    }

    // 2. 延迟插件队列（一个普通的队列）
    @Bean
    public Queue ttlDelayedQueue() {
        return QueueBuilder.durable("ttl-delayed.queue").build();
    }

    // 3. 绑定队列与交换机关系
    @Bean
    public Binding bindingTTLDelayed() {
        return BindingBuilder.bind(ttlDelayedQueue()).to(customExchange()).with("ttl-delayed.key").noargs();
    }

}
```

2、消费者监听消息

```java
package com.example.service;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class TTLDelayedPluginConsumer {
    // 监听死信队列
    @RabbitListener(queues = "ttl-delayed.queue")
    public void processOne(String msg) {
        System.out.println("死信队列收到消息：" + msg);
    }
}
```

3、发送者发送消息

```java
    /**
     * Delayed Message 插件：测试对消息设置过期时间
     */
    @Test
    public void ttlDelayedPluginTest(){
        rabbitTemplate.convertAndSend("ttl-delayed.exchange",
                "ttl-delayed.key","20s过期的消息", message -> {
            message.getMessageProperties().setDelay(20000);
            return message;
        });
        rabbitTemplate.convertAndSend("ttl-delayed.exchange",
                "ttl-delayed.key","2s过期的消息", message -> {
            message.getMessageProperties().setDelay(2000);
            return message;
        });
    }
```

4、消费端控制台输出：

```
死信队列收到消息：2s过期的消息
死信队列收到消息：20s过期的消息
```





## 02、死信队列 DLX

### 02-1、死信队列是什么

1、什么是死信队列（Dead Letter）：没有及时消费的消息存放的队列。

2、什么是死信交换机：DLX（Dead-Letter-Exchange）当消息在队列中变成死信之后，它能被重新发送到另一个交换机中，这个交换机就是DLX ，绑定DLX的队列就称之为死信队列（死信交换机与队列一样都是一个正常的交换机和队列，和一般的交换机和队列没有区别）

3、什么是死信消息（Dead-Message）：未被正常处理的消息。出现死信的情况：

- 消息被拒绝（reject、nack）并且不重新放回原队列（requeue=false）；
- 消息过期（设置了Expiration）
- 队列已满（队列达到最大长度）

**死信消息**会被RabbitMQ进行特殊处理，如果配置了死信队列，那么该消息将会被丢进死信队列中，如果没配置，则丢弃该消息。

**消息大致流转过程：**

![20210418110839](https://gitee.com/liusuixing/blog_images/raw/master/20210418110839.png)



### 02-2、如何配置死信队列

配置死信队列大概可以分为以下步骤：

1. 配置业务队列，绑定到业务交换机上
2. 为业务队列配置死信交换机和路由key
3. 为死信交换机配置死信队列

JAVA 代码实战：

```java
package com.example.dlx;

import com.rabbitmq.client.*;
import java.util.HashMap;
import java.util.Map;

/**
 * @author:
 * @description: Producer 简单队列生产者
 */
public class Producer {
    public static void main(String[] args) throws Exception {
        // 1: 创建连接工厂，设置连接属性，2: 从连接工厂中获取，3: 从连接中打开通道channel
        ConnectionFactory connectionFactory = new ConnectionFactory();
        Connection connection = connectionFactory.newConnection("生产者");
        Channel channel = connection.createChannel();

        // 创建死信交换机和队列
        channel.exchangeDeclare("dlx-exchange", BuiltinExchangeType.DIRECT);
        channel.queueDeclare("dlx", false, false, false, null);
        channel.queueBind("dlx", "dlx-exchange", "dlx.key");

        /**
         *  @params1：queue 队列的名称
         *  @params2：durable 队列是否持久化（即存盘）
         *  @params3：exclusive 是否排他，即是否私有的
         *  @params4：autoDelete 是否自动删除，当此队列的连接数为0时，此队列会销毁（无论队列中是否还有数据）
         *  @params5：arguments 可以设置队列附加参数，设置队列的有效期，消息的最大长度，队列的消息生命周期等等
         *              x-message-ttl：设置队列过期时间TTL
         *              x-max-length：设置队列最大存储消息数，超出的要么丢弃要么放进死信队列
         *              x-dead-letter-exchange：死信交换机名称
         * */
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-message-ttl",5000);
        arguments.put("x-max-length", 5);
        arguments.put("x-dead-letter-exchange", "dlx-exchange");
        arguments.put("x-dead-letter-routing-key", "dlx.key");
        channel.queueDeclare("ttl-queue", false, false, false, arguments);

        // param1:交换机名，param2: 队列名/路由key，param3:属性配置附加参数，param4:消息内容
        channel.basicPublish("", "ttl-queue", null, "你好，消息队列！".getBytes());
        System.out.println("消息发送成功!");

        // 最后关闭通关和连接
        channel.close();
        connection.close();
    }
}
```



### 02-3、死信队列代码实战

声明死信和延迟交换机 + 队列并且绑定关系

```java
package com.example.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 延迟队列 + 死信队列
 **/
@Configuration
public class DeadRabbitMQConfig {

    // 死信交换机、队列名、路由键
    public static final String DEAD_LETTER_EXCHANGE = "dead.letter.direct-exchange";
    public static final String DEAD_LETTER_QUEUE_NAME = "dead.letter.dead-queue";
    public static final String DEAD_LETTER_QUEUE_ROUTING_KEY = "dead.letter.routing-key";
    // 业务交换机、队列名、路由键
    public static final String BUSINESS_EXCHANGE_NAME = "dead.letter.business-exchange";
    public static final String BUSINESS_QUEUE_NAME = "dead.letter.business-queue";
    public static final String BUSINESS_QUEUE_ROUTING_KEY = "dead.letter.business-queue.routing-key";

    // 1. 声明死信交换机-Direct模式
    @Bean
    public DirectExchange deadDirectExchange() {
        return ExchangeBuilder.directExchange(DEAD_LETTER_EXCHANGE).build();
    }

    // 2. 声明死信队列
    @Bean
    public Queue deadDirectQueue() {
        return QueueBuilder.durable(DEAD_LETTER_QUEUE_NAME).build();
    }

    // 3. 声明死信队列与死信交换机绑定关系
    @Bean
    public Binding bindingDeadDirect() {
        return BindingBuilder.bind(deadDirectQueue()).to(deadDirectExchange()).with(DEAD_LETTER_QUEUE_ROUTING_KEY);
    }

    // 1. 声明业务Exchange-Direct模式—也可以称为 延迟交换机
    @Bean
    public DirectExchange ttlDeadDirectExchange() {
        return ExchangeBuilder.directExchange(BUSINESS_EXCHANGE_NAME).build();
    }

    // 2. 声明业务队列—也可以称为 延迟队列
    @Bean
    public Queue ttlDeadDirectQueue() {
        return QueueBuilder.durable(BUSINESS_QUEUE_NAME)
                // 设置过期时间,注意是int类型
                .withArgument("x-message-ttl", 5000)
                // 设置队列最大存储消息数，超出的要么丢弃要么放进死信队列
                .withArgument("x-max-length", 3)
                // 指定死信交换机，ttl过期以后消息自动转发到指定交换机
                .withArgument("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE)
                // 死信消息路由键
                .withArgument("x-dead-letter-routing-key", DEAD_LETTER_QUEUE_ROUTING_KEY)
                .build();
    }

    // 3  声明延迟队列与延迟交换机绑定关系
    @Bean
    public Binding bindingTTLDeadDirect() {
        return BindingBuilder.bind(ttlDeadDirectQueue()).to(ttlDeadDirectExchange()).with(BUSINESS_QUEUE_ROUTING_KEY);
    }
}
```

消息发送端测试代码：

```java
    /**
     * 延迟队列 + 死信队列测试
     * 发现消息到业务队列中，然后没有指定消费者消费或者消费是抛异常拒绝消息，
     * 就会流转到死信交换机中，然后到指定的路由中
     */
    @Test
    public void deadTest(){
        for (int i = 0; i < 5; i++) {
            String businessExchangeName = DeadRabbitMQConfig.BUSINESS_EXCHANGE_NAME;
            String businessQueueRoutingKey = DeadRabbitMQConfig.BUSINESS_QUEUE_ROUTING_KEY;
            rabbitTemplate.convertAndSend(businessExchangeName, businessQueueRoutingKey,"死信队列测试" + i);
        }
    }
```

> 查看web页面可以发现当消息超过5秒没有被消费或者消息超过3条以上时，都会被转移到死信交换机然后到指定的死信队列中。我们只需要监听指定的死信队列就可以做其他业务操作了

消费者端代码监听：

```java
package com.example.service;

import com.example.config.DeadRabbitMQConfig;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class DeadConsumer {
    // 监听死信队列
    @RabbitListener(queues = DeadRabbitMQConfig.DEAD_LETTER_QUEUE_NAME)
    public void processOne(String msg) {
        System.out.println("死信队列收到消息：" + msg);
    }
}
```

消费端监听控制台输出：

```
死信队列收到消息：死信队列测试0
死信队列收到消息：死信队列测试1
死信队列收到消息：死信队列测试2
死信队列收到消息：死信队列测试3
死信队列收到消息：死信队列测试4
```

最终所有的死信消息都会转入到死信队列中。



### 02-4、死信消息的变化

如果队列配置了参数`x-dead-letter-routing-key`，**死信**路由key将会被替换。如果不设置，则使用消息的原来的路由键值。

举个栗子：

如果原有消息的路由key是`testA`，被发送到业务Exchage中，然后被投递到业务队列QueueA中，如果该队列没有配置参数`x-dead-letter-routing-key`，则该消息成为死信后，将保留原有的路由key`testA`，如果配置了该参数，并且值设置为`testB`，那么该消息成为死信后，路由key将会被替换为`testB`，然后被抛到死信交换机中。另外，由于被抛到了死信交换机，所以消息的ExchangeName也会被替换为死信交换机的名称。

---

消息的Header中，也会添加很多奇奇怪怪的字段，修改一下上面的代码，在死信队列的消费者中添加一行日志输出：

```java
log.info("死信消息properties：{}", message.getMessageProperties());
```

然后重新运行一次，即可得到死信消息Header中被添加的信息：

```java
死信消息properties：MessageProperties [headers={x-first-death-exchange=dead.letter.business-exchange, x-death=[{reason=expired, count=1, exchange=dead.letter..business-exchange, time=Wed Jun 02 15:57:55 CST 2021, routing-keys=[dead.letter.business-queue.routing-key], queue=dead.letter.business-queue}], x-first-death-reason=expired, x-first-death-queue=dead.letter.business-queue}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=dead.letter.direct-exchange, receivedRoutingKey=dead.letter.routing-key, deliveryTag=3, consumerTag=amq.ctag-CX8-JD0orH03gXXlloL-AQ, consumerQueue=dead.letter.dead-queue]
```

Header中看起来有很多信息，实际上并不多，只是值比较长而已。下面就简单说明一下Header中的值：

| 字段名                 | 含义                                                         |
| :--------------------- | :----------------------------------------------------------- |
| x-first-death-exchange | 第一次被抛入的死信交换机的名称                               |
| x-first-death-reason   | 第一次成为死信的原因，`rejected`：消息在重新进入队列时被队列拒绝，由于`default-requeue-rejected` 参数被设置为`false`。`expired` ：消息过期。`maxlen` ： 队列内消息数量超过队列最大容量 |
| x-first-death-queue    | 第一次成为死信前所在队列名称                                 |
| x-death                | 历次被投入死信交换机的信息列表，同一个消息每次进入一个死信交换机，这个数组的信息就会被更新 |



### 02-5、死信队列总结

总结一下死信消息的生命周期：

- 业务消息被投入业务队列
- 消费者消费业务队列的消息，由于处理过程中发生异常，于是进行了nck或者reject操作
- 被nck或reject的消息由RabbitMQ投递到死信交换机中
- 死信交换机将消息投入相应的死信队列
- 死信队列的消费者消费死信消息

死信消息是RabbitMQ为我们做的一层保证，其实我们也可以不使用死信队列，而是在消息消费异常时，将消息主动投递到另一个交换机中，当你明白了这些之后，这些Exchange和Queue想怎样配合就能怎么配合。比如从死信队列拉取消息，然后发送邮件、短信、钉钉通知来通知开发人员关注。或者将消息重新投递到一个队列然后设置过期时间，来进行延时消费。





# RabbitMQ 属性参数配置详情

## 01、SpringBoot 配置参数详解

> 参考：
> https://www.chihiro.org.cn/archives/20200522150924
> https://blog.csdn.net/girlgolden/article/details/97915368

### 1、application.yml

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1 #ip
    port: 5672      #端口
    username: guest #账号
    password: guest #密码
    virtualHost:    #链接的虚拟主机
    addresses: 127.0.0.1:5672     #多个以逗号分隔，与host功能一样。
    requestedHeartbeat: 60 #指定心跳超时，单位秒，0为不指定；默认60s
    publisherConfirms: true  #发布确认机制是否启用
    publisherReturns: #发布返回是否启用
    connectionTimeout: #链接超时。单位ms。0表示无穷大不超时
    ### ssl相关
    ssl:
      enabled: #是否支持ssl
      keyStore: #指定持有SSL certificate的key store的路径
      keyStoreType: #key store类型 默认PKCS12
      keyStorePassword: #指定访问key store的密码
      trustStore: #指定持有SSL certificates的Trust store
      trustStoreType: #默认JKS
      trustStorePassword: #访问密码
      algorithm: #ssl使用的算法，例如，TLSv1.1
      verifyHostname: #是否开启hostname验证
    ### cache相关
    cache:
      channel: 
        size: #缓存中保持的channel数量
        checkoutTimeout: #当缓存数量被设置时，从缓存中获取一个channel的超时时间，单位毫秒；如果为0，则总是创建一个新channel
      connection:
        mode: #连接工厂缓存模式：CHANNEL 和 CONNECTION
        size: #缓存的连接数，只有是CONNECTION模式时生效
    ### listener
    listener:
       type: #两种类型，SIMPLE，DIRECT
       ## simple类型
       simple:
         concurrency: #最小消费者数量
         maxConcurrency: #最大的消费者数量
         transactionSize: #指定一个事务处理的消息数量，最好是小于等于prefetch的数量
         missingQueuesFatal: #是否停止容器当容器中的队列不可用
         ## 与direct相同配置部分
         autoStartup: #是否自动启动容器
         acknowledgeMode: #表示消息确认方式，其有三种配置方式，分别是none、manual和auto；默认auto
         prefetch: #指定一个请求能处理多少个消息，如果有事务的话，必须大于等于transaction数量
         defaultRequeueRejected: #决定被拒绝的消息是否重新入队；默认是true（与参数acknowledge-mode有关系）
         idleEventInterval: #container events发布频率，单位ms
         ##重试机制
         retry: 
           stateless: #有无状态
           enabled:  #是否开启
           maxAttempts: #最大重试次数,默认3
           initialInterval: #重试间隔
           multiplier: #对于上一次重试的乘数
           maxInterval: #最大重试时间间隔
       direct:
         consumersPerQueue: #每个队列消费者数量
         missingQueuesFatal:
         #...其余配置看上方公共配置
     ## template相关
     template:
       mandatory: #是否启用强制信息；默认false
       receiveTimeout: #`receive()`接收方法超时时间
       replyTimeout: #`sendAndReceive()`超时时间
       exchange: #默认的交换机
       routingKey: #默认的路由
       defaultReceiveQueue: #默认的接收队列
       ## retry重试相关
       retry: 
         enabled: #是否开启
         maxAttempts: #最大重试次数
         initialInterval: #重试间隔
         multiplier: #失败间隔乘数
         maxInterval: #最大间隔
```

### 2、application.properties

```properties
############################ base ############################
spring.rabbitmq.host: 服务Host
spring.rabbitmq.port: 服务端口
spring.rabbitmq.username: 登陆用户名
spring.rabbitmq.password: 登陆密码
spring.rabbitmq.virtual-host: 连接到rabbitMQ的vhost
spring.rabbitmq.addresses: 指定client连接到的server的地址，多个以逗号分隔(优先取addresses，然后再取host)
spring.rabbitmq.requested-heartbeat: 指定心跳超时，单位秒，0为不指定；默认60s
spring.rabbitmq.publisher-confirms: 是否启用【发布确认】
spring.rabbitmq.publisher-returns: 是否启用【发布返回】
spring.rabbitmq.connection-timeout: 连接超时，单位毫秒，0表示无穷大，不超时
 
############################ ssl ############################
spring.rabbitmq.ssl.enabled: 是否支持ssl
spring.rabbitmq.ssl.key-store: 指定持有SSL certificate的key store的路径
spring.rabbitmq.ssl.key-store-password: 指定访问key store的密码
spring.rabbitmq.ssl.trust-store: 指定持有SSL certificates的Trust store
spring.rabbitmq.ssl.trust-store-password: 指定访问trust store的密码
spring.rabbitmq.ssl.trust-store-type: JKS：Trust store 类型
spring.rabbitmq.ssl.algorithm: ssl使用的算法，默认由rabiitClient配置,例如，TLSv1.1
spring.rabbitmq.ssl.validate-server-certificate=true：是否启用服务端证书验证
spring.rabbitmq.ssl.verify-hostname=true 是否启用主机验证
 
############################ cache ############################
spring.rabbitmq.cache.channel.size: 缓存中保持的channel数量
spring.rabbitmq.cache.channel.checkout-timeout: 当缓存数量被设置时，从缓存中获取一个channel的超时时间，单位毫秒；如果为0，则总是创建一个新channel
spring.rabbitmq.cache.connection.size: 缓存的连接数，只有是CONNECTION模式时生效
spring.rabbitmq.cache.connection.mode: 连接工厂缓存模式：CHANNEL 和 CONNECTION
 
############################ listener ############################
spring.rabbitmq.listener.type=simple: 容器类型.simple或direct

spring.rabbitmq.listener.simple.auto-startup: 是否启动时自动启动容器
spring.rabbitmq.listener.simple.acknowledge-mode: 表示消息确认方式，其有三种配置方式，分别是none、manual和auto；默认auto
spring.rabbitmq.listener.simple.concurrency: 最小的消费者数量
spring.rabbitmq.listener.simple.max-concurrency: 最大的消费者数量
spring.rabbitmq.listener.simple.prefetch: 指定一个请求能处理多少个消息，如果有事务的话，必须大于等于transaction数量.
spring.rabbitmq.listener.simple.transaction-size: 指定一个事务处理的消息数量，最好是小于等于prefetch的数量.
spring.rabbitmq.listener.simple.default-requeue-rejected: 决定被拒绝的消息是否重新入队；默认是true（与参数acknowledge-mode有关系）
spring.rabbitmq.listener.simple.missing-queues-fatal=true 若容器声明的队列在代理上不可用，是否失败； 或者运行时一个多多个队列被删除，是否停止容器
spring.rabbitmq.listener.simple.idle-event-interval: 多少长时间发布空闲容器时间，单位毫秒
spring.rabbitmq.listener.simple.retry.enabled: 监听重试是否可用
spring.rabbitmq.listener.simple.retry.max-attempts: 最大重试次数
spring.rabbitmq.listener.simple.retry.initial-interval: 第一次和第二次尝试发布或传递消息之间的间隔
spring.rabbitmq.listener.simple.retry.multiplier: 应用于上一重试间隔的乘数
spring.rabbitmq.listener.simple.retry.max-interval: 最大重试时间间隔
spring.rabbitmq.listener.simple.retry.stateless: 重试是有状态or无状态

spring.rabbitmq.listener.direct.acknowledge-mode= ack模式
spring.rabbitmq.listener.direct.auto-startup=true 是否在启动时自动启动容器
spring.rabbitmq.listener.direct.consumers-per-queue= 每个队列消费者数量.
spring.rabbitmq.listener.direct.default-requeue-rejected= 默认是否将拒绝传送的消息重新入队.
spring.rabbitmq.listener.direct.idle-event-interval= 空闲容器事件发布时间间隔.
spring.rabbitmq.listener.direct.missing-queues-fatal=false若容器声明的队列在代理上不可用，是否失败.
spring.rabbitmq.listener.direct.prefetch= 每个消费者可最大处理的nack消息数量.
spring.rabbitmq.listener.direct.retry.enabled=false  是否启用发布重试机制.
spring.rabbitmq.listener.direct.retry.initial-interval=1000ms # Duration between the first and second attempt to deliver a message.
spring.rabbitmq.listener.direct.retry.max-attempts=3 # Maximum number of attempts to deliver a message.
spring.rabbitmq.listener.direct.retry.max-interval=10000ms # Maximum duration between attempts.
spring.rabbitmq.listener.direct.retry.multiplier=1 # Multiplier to apply to the previous retry interval.
spring.rabbitmq.listener.direct.retry.stateless=true # Whether retries are stateless or stateful.
 
############################ template ############################
spring.rabbitmq.template.mandatory: 启用强制信息；默认false
spring.rabbitmq.template.receive-timeout: receive() 操作的超时时间
spring.rabbitmq.template.reply-timeout: sendAndReceive() 操作的超时时间
spring.rabbitmq.template.retry.enabled: 发送重试是否可用
spring.rabbitmq.template.retry.max-attempts: 最大重试次数
spring.rabbitmq.template.retry.initial-interval: 第一次和第二次尝试发布或传递消息之间的间隔
spring.rabbitmq.template.retry.multiplier: 应用于上一重试间隔的乘数
spring.rabbitmq.template.retry.max-interval: 最大重试时间间隔
```





## 02、创建队列附加参数详解

### 1、Web端查看

![20210418110847](https://gitee.com/liusuixing/blog_images/raw/master/20210418110847.png)



### 2、代码设置

SpringBoot 创建queue设置附加参数（**附加参数类型为Map**）

```java
package com.example.config;

import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.QueueBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.Map;

/**
 * 简单队列模式
 **/
//@Configuration
public class QueueParamRabbitMQConfig {
    @Bean
    public Queue queue() {
        /**
         * durable：是否持久化,默认是false,持久化队列（内部会有一个actualName: 队列的真实名称，默认用name参数，如果name为空，则根据规则生成一个）
         * exclusive：默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
         * autoDelete：是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
         * arguments：设置队列的属性参数
         *  1、x-message-ttl：消息的过期时间，单位：毫秒；
         *  2、x-expires：队列过期时间，队列在多长时间未被访问将被删除，单位：毫秒；
         *  3、x-max-length：队列最大长度，超过该最大值，则将从队列头部开始删除消息；
         *  4、x-max-length-bytes：队列消息内容占用最大空间，受限于内存大小，超过该阈值则从队列头部开始删除消息；
         *  5、x-overflow：设置队列溢出行为。这决定了当达到队列的最大长度时消息会发生什么。有效值是drop-head、reject-publish或reject-publish-dlx。仲裁队列类型仅支持drop-head；
         *  6、x-dead-letter-exchange：死信交换器名称，过期或被删除（因队列长度超长或因空间超出阈值）的消息可指定发送到该交换器中；
         *  7、x-dead-letter-routing-key：死信消息路由键，在消息发送到死信交换器时会使用该路由键，如果不设置，则使用消息的原来的路由键值
         *  8、x-single-active-consumer：表示队列是否是单一活动消费者，true时，注册的消费组内只有一个消费者消费消息，其他被忽略，false时消息循环分发给所有消费者(默认false)
         *  9、x-max-priority：队列要支持的最大优先级数;如果未设置，队列将不支持消息优先级；
         * 10、x-queue-mode（Lazy mode）：将队列设置为延迟模式，在磁盘上保留尽可能多的消息，以减少RAM的使用;如果未设置，队列将保留内存缓存以尽可能快地传递消息；
         * 11、x-queue-master-locator：在集群模式下设置镜像队列的主节点信息。
         */
        Map<String, Object> args = new HashMap<>();
        args.put("x-message-ttl", 5000);
        Queue queue = new Queue("ttl.direct.queue", true, false, false, args);
        // 也可以用build模式创建
        Queue build = QueueBuilder.durable("test.queue")
                .exclusive()
                .autoDelete()
                .withArguments(args)
                .build();
        return build;
    }
}
```

```java
// Java 版本参数可直接参考上面SpringBoot即可
channel.queueDeclare("ttl-queue", false, false, false, arguments);
```

> arguments：设置队列的属性参数

```
1、x-message-ttl：消息的过期时间，单位：毫秒；

2、x-expires：队列过期时间，队列在多长时间未被访问将被删除，单位：毫秒；

3、x-max-length：队列最大长度，超过该最大值，则将从队列头部开始删除消息；

4、x-max-length-bytes：队列消息内容占用最大空间，受限于内存大小，超过该阈值则从队列头部开始删除消息；

5、x-overflow：设置队列溢出行为。这决定了当达到队列的最大长度时消息会发生什么。
               有效值是drop-head、reject-publish或reject-publish-dlx。仲裁队列类型仅支持drop-head；

6、x-dead-letter-exchange：死信交换器名称，过期或被删除（因队列长度超长或因空间超出阈值）的消息可指定发送到该交换器中；

7、x-dead-letter-routing-key：死信消息路由键，在消息发送到死信交换器时会使用该路由键，
                              如果不设置，则使用消息的原来的路由键值

8、x-single-active-consumer：表示队列是否是单一活动消费者，true时，注册的消费组内只有一个消费者消费消息，
                             其他被忽略，false时消息循环分发给所有消费者(默认false)

9、x-max-priority：队列要支持的最大优先级数;如果未设置，队列将不支持消息优先级；

10、x-queue-mode（Lazy mode）：将队列设置为延迟模式，在磁盘上保留尽可能多的消息，以减少RAM的使用;
                              如果未设置，队列将保留内存缓存以尽可能快地传递消息；

11、x-queue-master-locator：在集群模式下设置镜像队列的主节点信息。
```





## 03、发送消息附加参数详解

> https://blog.csdn.net/u010520146/article/details/103288443
> https://blog.csdn.net/yaomingyang/article/details/102636666

### 1、Web 端查看

![20210418110846](https://gitee.com/liusuixing/blog_images/raw/master/20210418110846.png)



### 2、AMQP.BasicProperties

**JAVA版：附加参数类型为：AMQP.BasicProperties**

RabbitMQ发布消息的方法为：

```java
void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException;
```

其中 props 参数可为消息体赋予多种的功能特性。BasicProperties的属性有以下：

```java
String contentType, //消息内容的类型
String contentEncoding, //消息内容的编码格式
Map<String,Object> headers,//header类型的交换机可以用到
Integer deliveryMode,//消息持久化 1 不持久化 2 持久化
Integer priority,//优先级
String correlationId, //关联id
String replyTo,//通常用于命名回调队列
String expiration,//设置过期消息过期时间
String messageId, //消息id
Date timestamp, //消息的时间戳
String type,  //类型
String userId, //用户ID
String appId, //应用程序id
String clusterId //集群id
```

参考示例：

```java
Map<String, Object> headers = new HashMap<String, Object>();
headers.put("arg1", "aaa");
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder()
                    .deliveryMode(2) // 传送方式
                    .contentEncoding("UTF-8") // 编码方式
                    .expiration("10000") // 过期时间
                    .headers(headers) //自定义属性
                    .build();
```

RabbitMQ内部封装的几个方法：

```java
public class MessageProperties {

    /** Empty basic properties, with no fields set */
    public static final BasicProperties MINIMAL_BASIC =
        new BasicProperties(null, null, null, null,
                            null, null, null, null,
                            null, null, null, null,
                            null, null);
    /** Empty basic properties, with only deliveryMode set to 2 (persistent) */
    public static final BasicProperties MINIMAL_PERSISTENT_BASIC =
        new BasicProperties(null, null, null, 2,
                            null, null, null, null,
                            null, null, null, null,
                            null, null);

    /** Content-type "application/octet-stream", deliveryMode 1 (nonpersistent), priority zero */
    public static final BasicProperties BASIC =
        new BasicProperties("application/octet-stream",
                            null,
                            null,
                            1,
                            0, null, null, null,
                            null, null, null, null,
                            null, null);

    /** Content-type "application/octet-stream", deliveryMode 2 (persistent), priority zero */
    public static final BasicProperties PERSISTENT_BASIC =
        new BasicProperties("application/octet-stream",
                            null,
                            null,
                            2,
                            0, null, null, null,
                            null, null, null, null,
                            null, null);

    /** Content-type "text/plain", deliveryMode 1 (nonpersistent), priority zero */
    public static final BasicProperties TEXT_PLAIN =
        new BasicProperties("text/plain",
                            null,
                            null,
                            1,
                            0, null, null, null,
                            null, null, null, null,
                            null, null);

    /** Content-type "text/plain", deliveryMode 2 (persistent), priority zero */
    public static final BasicProperties PERSISTENT_TEXT_PLAIN =
        new BasicProperties("text/plain",
                            null,
                            null,
                            2,
                            0, null, null, null,
                            null, null, null, null,
                            null, null);
}
```



### 3、MessageProperties

**SpringBoot版：附加参数类型：MessageProperties**

可以直接参考源码：org.springframework.amqp.core.MessageProperties，查看内部的字段即可，由于字段太多没有贴出来了。

参考示例：

```java
    /**
     * Headers模式
     */
    @Test
    public void headersSend() {
        MessageProperties messageProperties1 = new MessageProperties();
        messageProperties1.setHeader("header1","value1");
        messageProperties1.setHeader("header11","value11");
        Message message1 = new Message(("Header模式—单个匹配").getBytes(), messageProperties1);
        rabbitTemplate.convertAndSend("header_exchange","",message1);

        MessageProperties messageProperties2 = new MessageProperties();
        messageProperties2.setHeader("header2","value2");
        messageProperties2.setHeader("header22","value22");
        Message message2 = new Message(("Header模式—全部匹配").getBytes(), messageProperties2);
        rabbitTemplate.convertAndSend("header_exchange","",message2);
    }

    /**
     * TTL过期时间测试——对消息本身设置过期时间
     * 测试发送后查看web页面，5秒后消息自动消失了
     */
    @Test
    public void ttlMessageDirectSend() {
        MessagePostProcessor messagePostProcessor = message -> {
            MessageProperties messageProperties = message.getMessageProperties();
            messageProperties.setExpiration("5000");
            messageProperties.setContentEncoding("UTF-8");
            return message;
        };
        rabbitTemplate.convertAndSend("ttl_direct_exchange","ttl.message","TTL-消息过期时间", messagePostProcessor);
    }
```



## 04、@RabbitListener 详解

### case1：exchange_queue 已存在

```java
package com.example.config;

import org.springframework.amqp.rabbit.annotation.*;
import org.springframework.stereotype.Component;

@Component
public class AutoCreateQueueExchange {

    /**
     * 队列和交换机已经存在，只用使用queues，消费指定的queue即可.
     * 可以单独监听一个或者多个队列。如下只用其一即可
     * queues参数值为对列名(queueName)
     */
    //@RabbitListener(queues = {"exist.queue1", "exist.queue2"})
    @RabbitListener(queues = "simple.queue")
    public void receive1(String msg){
    }

    /**
     * 单独声明队列
     */
    //@RabbitListener(queuesToDeclare = {@Queue(value = "queue1"),@Queue(value = "queue2")})
    @RabbitListener(queuesToDeclare = @Queue(value = "declare.queue"))
    public void receive2(String msg){
    }
}
```



### case2：exchange_queue 不存在

```java
    /**
     * 发布/订阅模式 Fanout
     * @QueueBinding 注解的三个属性
     *   value: @Queue注解用于声明队列，value:queueName,durable: 队列是否持久化,autoDelete: 否自动删除
     *   exchange: @Exchange 注解，用于声明 exchange， type 指定消息投递策略，我们这里用的 fanout 方式
     *   key: 这个就是我们熟知的 routingKey，在fanout方式下是没有routingKey的
     */
    @RabbitHandler
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(value = "fanout.queue1",durable = "true", autoDelete = "false"),
                    exchange = @Exchange(value = "fanout.exchange1", type = ExchangeTypes.FANOUT)
            )
    })
    public void receive3(String msg){
    }

    /**
     * 路由模式 Direct
     * @QueueBinding 注解的三个属性
     *   value: @Queue注解用于声明队列，value:queueName,durable: 队列是否持久化,autoDelete: 否自动删除
     *   exchange: @Exchange 注解，用于声明 exchange， type 指定消息投递策略，我们这里用的 fanout 方式
     *   key: 这个就是我们熟知的 routingKey，在fanout方式下是没有routingKey的
     * 注意：由于@Exchange 中 type默认值就是DIRECT，所以可以省略：type = ExchangeTypes.DIRECT
     */
    @RabbitHandler
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(value = "direct.queue1", durable = "true", autoDelete = "false"),
                    exchange = @Exchange(value = "direct.exchange1"),
                    key = "direct.key1"
            )
    })
    public void receive4(String msg){
    }

    /**
     * 主题模式 Topic—单个申请
     */
    @RabbitHandler
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(value = "topic.queue1",autoDelete = "false"),
                    exchange = @Exchange(value = "topic.exchange1", type = ExchangeTypes.TOPIC),
                    key = "topic.key1"
            )
    })
    public void receive5(String msg){
    }

    /**
     * 主题模式 Topic—多个申请
     */
    @RabbitHandler
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(value = "topic.queue1",autoDelete = "false"),
                    exchange = @Exchange(value = "test.exchange1", type = ExchangeTypes.TOPIC),
                    key = "topic.key1"
            ),
            @QueueBinding(
                    value = @Queue(value = "topic.queue2",autoDelete = "false"),
                    exchange = @Exchange(value = "test.exchange1", type = ExchangeTypes.TOPIC),
                    key = "topic.key2"
            )
    })
    public void receive6(String msg){
    }

    /**
     * Headers 模式
     */
    @RabbitHandler
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(value = "headers.queue1",autoDelete = "false"),
                    exchange = @Exchange(value = "headers.exchange1", type = ExchangeTypes.HEADERS),
                    arguments = {
                            @Argument(name = "x-match", value = "any"),
                            @Argument(name = "key-one", value = "1"),
                            @Argument(name = "key-two", value = "2")
                    }
            )
    })
    public void receive7(String msg){
    }
```



### case3：延迟队列+死信队列声明

```java
    /**
     * 声明延迟队列。并绑定死信交换机。
     * 设置延迟队列消息超过5秒就过期，并且消息最大条数为3条。超过了也会成为死信消息。
     * 并且开启了 消息手动确认ack模式
     */
    @RabbitHandler
    @RabbitListener(ackMode = "MANUAL",bindings = {
            @QueueBinding(value = @Queue(value = "liveQueue", arguments = {
                          @Argument(name = "x-dead-letter-exchange", value = "deadExchange"),
                          @Argument(name = "x-dead-letter-routing-key", value = "deadKey"),
                          @Argument(name = "x-message-ttl", value = "5000", type = "java.lang.Integer"),
                          @Argument(name = "x-max-length",value = "3",type = "java.lang.Integer")
                    }),
                    exchange = @Exchange(value = "liveExchange"),
                    key = {"info", "error", "warning"}
            )
    })
    public void onMessage1(String msg, Message message, Channel channel) throws IOException {
        long consumerTag = message.getMessageProperties().getDeliveryTag();
        try {
            // 模拟消费消息, 回调接收ack是true 因为回调的ack根据exchange是否收到消息判断
            int i = 1/new Random().nextInt(1);
            System.out.println("收到消息:" + new String(message.getBody()));
            channel.basicAck(consumerTag, false);
        } catch (Exception e) {
            // 消费异常消息会添加到死信队列
            channel.basicNack(consumerTag, false, false);
        }
    }

    /**
     * 声明死信交换机和队列 并且绑定
     */
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(value = "deadQueue"),
                    exchange = @Exchange(value = "deadExchange"),
                    key = "deadKey"
            )
    })
    public void onMessage2(Message message, Channel channel) throws IOException {
        System.out.println("我是一条死信队列:" + message);
        long consumerTag = message.getMessageProperties().getDeliveryTag();
        channel.basicAck(consumerTag, true);
    }
```



### case4：手动 Ack + 并发消费

当消息很多，一个消费者吭哧吭哧的消费太慢，但是我的机器性能又杠杠的，这个时候我就希望并行消费，相当于同时有多个消费者来处理数据。要支持并行消费，如下设置即可

```java
    /**
     * 手动ack：ackMode = "MANUAL"
     * 注意：
     * @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag
     * 替代了 参数 Message message
     * long deliveryTag = message.getMessageProperties().getDeliveryTag();
     */
    @RabbitHandler
    @RabbitListener(queues = "simple.queue", ackMode = "MANUAL")
    public void consumerDoAck(String msg, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag,
                              Channel channel) throws IOException {
        if (msg.contains("ok")) {
            channel.basicAck(deliveryTag, false);
        } else {
            channel.basicNack(deliveryTag, false, true);
        }
    }

    /**
     * 并发消费。注意：concurrency = "4"属性，表示固定 4 个消费者；
     */
    @RabbitHandler
    @RabbitListener(queues = "simple.queue", concurrency = "4")
    public void multiConsumer(String data) {
        System.out.println("multiConsumer: " + data);
    }
```



## 05、@RabbitListener+@RabbitHandler

自己查资料研究：https://blog.csdn.net/weixin_43343423/article/details/103382661



# RabbitMQ 集群篇

## 00、集群架构原理

前面我们有介绍到 RabbitMQ 内部有各种基础构件，包括队列、交换器、绑定、虚拟主机等，他们组成了 AMQP 协议消息通信的基础，而这些构件以**元数据**的形式存在，它始终记录在 RabbitMQ 内部，它们分别是：

- **队列元数据**：队列名称和它们的属性
- **交换器元数据**：交换器名称、类型和属性
- **绑定元数据**：一张简单的表格展示了如何将消息路由到队列
- **vhost 元数据**：为 vhost 内的队列、交换器和绑定提供命名空间和安全属性

**PS：元数据，指的是包括队列名字属性、交换机的类型名字属性、绑定信息、vhost等基础信息，不包括队列中的消息数据。**

这些元数据，其实本质是一张查询表，里面包括了交换器名称和一个队列的绑定列表，当你将消息发布到交换器中，实际上是将你所在的信道将消息上的路由键与交换器的绑定列表进行匹配，然后将消息路由出去。

![20210418110827](https://gitee.com/liusuixing/blog_images/raw/master/20210418110827.png)

有了这个机制，那么在所有节点上传递交换器消息将简单很多，而 RabbitMQ 所做的事情就是把交换器元数据拷贝到所有节点上，因此每个节点上的每条信道都可以访问完整的交换器。

![20210418110828](https://gitee.com/liusuixing/blog_images/raw/master/20210418110828.png)

如果消息生产者所连接的是节点 2 或者节点 3，此时队列1的完整数据不在该两个节点上，那么在发送消息过程中这两个节点主要起了一个路由转发作用，根据这两个节点上的元数据转发至节点1上，最终发送的消息还是会存储至节点1的队列1上。

同样，如果消息消费者所连接的节点2或者节点3，那这两个节点也会作为路由节点起到转发作用，将会从节点1的队列1中拉取消息进行消费。



**与常见的集群主从架构模式不同在于：RabbitMQ集群模式下，仅仅只是同步元数据，每个队列内容还是在自己的服务器节点上**

这么设计主要还是基于集群本身的**性能和存储空间**上来考虑：

- **存储空间**：真正存放数据的地方是在队列里面，如果每个集群节点都拥有所有队列的完全数据拷贝，那么每个节点的存储空间会非常大，集群的消息积压能力会非常弱。例如你现在存储了 3G 队列内容，那么在另外一个只有 1G 存储空间的节点上，就会造成内存空间不足的情况，也就是无法通过集群节点的扩容提高消息积压能力。
- **性能**：消息的发布者需要将消息复制到每一个集群节点，每一条消息都会触发磁盘活动，这会导致整个集群内性能负载急剧拉升。



既然每个队列内容还是在自己的服务器节点上，同样也会带来新的问题，**那就是如果队列所在服务器挂了，那存在服务器上的队列数据是不是全部都丢失了**？

在单个节点上，RabbitMQ 存储数据有两种方案：

- **内存模式**：这种模式会将数据存储在内存当中，如果服务器突然宕机重启之后，那么附加在该节点上的队列和其关联的绑定都会丢失，并且消费者可以重新连接集群并重新创建队列；
- **磁盘模式**：这种模式会将数据存储磁盘当中，如果服务器突然宕机重启，数据会自动恢复，该队列又可以进行传输数据了，并且在恢复故障磁盘节点之前，不能在其它节点上让消费者重新连到集群并重新创建队列，如果消费者继续在其它节点上声明该队列，会得到一个 404 NOT_FOUND 错误，这样确保了当故障节点恢复后加入集群，该节点上的队列消息不会丢失，也避免了队列会在一个节点以上出现冗余的问题。

在集群中的每个节点，要么是内存节点，要么是磁盘节点，如果是内存节点，会将所有的元数据信息仅存储到内存中，**而磁盘节点则不仅会将所有元数据存储到内存上， 还会将其持久化到磁盘**。

在单节点 RabbitMQ 上，仅允许该节点是磁盘节点，这样确保了节点发生故障或重启节点之后，所有关于系统的配置与元数据信息都会从磁盘上恢复。

而在 RabbitMQ 集群上，至少有一个磁盘节点，也就是在集群环境中需要添加 2 台及以上的磁盘节点，这样其中一台发生故障了，集群仍然可以保持运行。其它节点均设置为内存节点，这样会让队列和交换器声明之类的操作会更加快速，元数据同步也会更加高效。



## 01、集群的几种模式

> 参考文章：
> https://www.cnblogs.com/rouqinglangzi/p/10584748.html
>
> RabbitMQ模式大概分为以下四种：
>
> - **普通集群模式/主备集群模式（Warren）**：默认的模式
> - **镜像队列模式（Mirror）**：实际工作用的最多
> - **远程模式（Shovel）：**属于分布式部署，这个属于早期常用的，现在很少用了
>
> - **多活模式（Federation）：**属于分布式部署，一般异地集群部署会用这种模式
>
> **RabbitMQ集群对延迟非常敏感，应该只在本地局域网中使用。**在广域网中不应该使用集群模式，而应该使用Federation或者Shovel来进行分布式部署。

 

### 01-1、主备模式（Warren）

> 也称为 Warren (兔子窝) 模式。实现 rabbitMQ 的高可用集群，一般在并发和数据量不高的情况下，这种模式非常的好用且简单。
>
> 默认情况下，RabbitMQ集群中队列的内容仅位于单个节点上（即声明该队列的节点，也就是主节点）。创建队列时，只会在某一个节点上创建队列，其它节点上并不含有队列，而只是含有创建节点的元数据(包括队列信息，绑定关系等)。如果队列所在的节点故障，则队列就崩溃了。

1、基本特征

- 交换机和队列的元数据存在于所有的节点上
- queue队列中的完整数据只存在于创建该队列的节点上
- 其他节点只保存队列的元数据信息以及指向当前队列的owner node的指针

![20210418110825](https://gitee.com/liusuixing/blog_images/raw/master/20210418110825.png)

2、数据消费

进行数据消费时随机连接到一个节点，当队列不是当前节点创建的时候，需要有一个从创建队列的实例拉取队列数据的开销。此外由于需要固定从单实例获取数据，因此会出现单实例的瓶颈。

3、优点：可以由多个节点消费单个队列的数据，提高了吞吐量

4、缺点：

- 节点实例需要拉取数据，因此集群内部存在大量的数据传输
- 可用性保障低，一旦创建队列的节点宕机，只有等到该节点恢复其他节点才能继续消费消息





### 02-2、镜像模式（Mirror）

> 集群模式非常经典的就是Mirror镜像模式，保证100%数据不丢失，在实际工作中也是用的最多的，而且实现集群也非常简单，一般互联网大厂都会构建这种镜像集群模式，原理主要是在主备的基础上进行了扩展，集群中所有的节点设备都是同步的，每一个队列，交换机里面的配置信息和我们的数据都是同步的，对于这些镜像在底层同时进行工作，前面的话采用一个负载均衡器，采用nginx或者haproxy也好，进行负载均衡。

1、基本特征

- 创建的queue，不论是元数据还是完整数据都会在每一个节点上保存一份
- 向queue中写消息时，都会自动同步到每一个节点上

![20210418110826](https://gitee.com/liusuixing/blog_images/raw/master/20210418110826.png)

2、优点：

- 保障了集群的高可用
- 配置方便，只需要在后台配置相应的策略，就可以将指定数据同步到指定的节点或者全部节点

3、缺点：

- 性能开销较大，网络带宽压力和消耗很严重，所以镜像队列的吞吐量会低于普通模式
- 无法线性扩展，例如单个queue的数据量很大，每台机器都要存储同样大量的数据



### 03-3、远程模式（Shovel）

> 远程模式可以实现双活的一种模式，简称 shovel 模式，所谓的 shovel 就是把消息进行不同数据中心的复制工作，可以跨地域的让两个MQ 集群互联，远距离通信和复制。这种模式配置比较复杂，属于RabbitMQ比较早期的模型了，现在很少使用了。
>
> 官方文档：[Shovel](https://www.rabbitmq.com/shovel.html)。

例如说一个集群，我们都会放在一个机房里面，那么如果北京的机房出现了一些事故停电，或者自然灾害，那么这个集群都会宕机了，那么在我们对数据要求极高的大型应用我们需要设置多活或者双活的模式，也就是要搭建多个数据中心，或者多套集群，那么这些集群可以一个会放在上海，一个放在北京，还有应放在广州，三个集群数据都是同步的，中间有任何一个集群出现了问题，马上灵活的切换，那么这三个集群都是可以访问的话，我们可能会按照距离，或者访问速度来进行优先选择哪组集群，或者数据中心进行访问，所有多活模式，在银行开发的时候一般也叫做容灾的机制，至少构建两套集群放在不同的地域，一个有问题了，立马进行切换，不至于整个系统宕机，这就是多活模式，在多活模式中MQ也提供了相应的实现方式，早期使用的Shovel模式，这个模式是mq自带的一种模式，主要就是可以把消息在不同的数据中心进行负载分发，主要就是可以实现跨地域的让两个mq集群进行互联。

那么这个shovel模式需要统一的版本，网络达到一个什么样的水平，配置的话也是有些复杂，这种的话目前已经淘汰了，在真正的数据复制的情况下，会使用多活模式。



### 04-4、多活模式（Federation）

> 这种模式也是实现异地数据复制的主流模式，这种模式需要依赖 rabbitMQ的 Federation 插件，本身不是mq自带的东西，可以实现持续的、可靠的 AMQP 数据通信。因此配置起来也比较容易，相当于配置简单化之后的shovel。
>
> 官方文档：[Federation](https://www.rabbitmq.com/federation.html)


这是实现异地数据复制的主流模式，现在实现异地集群的一般都是采用这种：双活 或者 多活模型。





## 02、单机多实例搭建

> **场景：**假设有三个rabbitmq节点，分别为rabbit-1、rabbit-2、rabbit-3，rabbit-1作为主节点，2和3作为从节点。
> **启动命令**：RABBITMQ_NODE_PORT={{端口号}} RABBITMQ_NODENAME={{节点名}} rabbitmq-server -detached
> **结束命令**：rabbitmqctl -n {{节点名}} stop



### 02-1、关闭RabbitMQ服务

RabbitMQ 默认是使用服务的启动的，单机多节点时需要改为手动启动，先停止运行中的RabbitMQ 服务

```bash
sudo systemctl stop rabbitmq-server
```



### 02-2、启动三个RabbitMQ节点

```bash
sudo RABBITMQ_NODE_PORT=5672 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15672}]" RABBITMQ_NODENAME=rabbit-1 rabbitmq-server -detached

sudo RABBITMQ_NODE_PORT=5673 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME=rabbit-2 rabbitmq-server -detached

sudo RABBITMQ_NODE_PORT=5674 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15674}]" RABBITMQ_NODENAME=rabbit-3 rabbitmq-server -detached
```



### 02-3、查看服务其启动状况

命令：ps aux|grep rabbitmq

```bash
[root@CentOS7 ~]# ps aux|grep rabbitmq
rabbitmq 13849 30.5  2.4 2296044 90600 ?       Sl   13:41   0:08 /usr/lib64/erlang/erts-11.1.8/bin/beam.smp -W w -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs 512 -MMmcs 30 -P 1048576 -t 5000000 -stbt db -zdbbl 128000 -sbwt none -sbwtdcpu none -sbwtdio none -B i -- -root /usr/lib64/erlang -progname erl -- -home /var/lib/rabbitmq -- -pa  -noshell -noinput -s rabbit boot -boot start_sasl -rabbitmq_management listener [{port,15672}] -lager crash_log false -lager handlers [] -noshell -noinput
rabbitmq 13853  0.0  0.0   4356   544 ?        Ss   13:41   0:00 erl_child_setup 65535
rabbitmq 13876  0.0  0.0  48912   520 ?        S    13:41   0:00 /usr/lib64/erlang/erts-11.1.8/bin/epmd -daemon
rabbitmq 13897  0.0  0.0  11592   456 ?        Ss   13:41   0:00 inet_gethost 4
rabbitmq 13898  0.0  0.0  41552  1196 ?        S    13:41   0:00 inet_gethost 4
rabbitmq 13925 50.4  2.4 2298236 90540 ?       Sl   13:41   0:08 /usr/lib64/erlang/erts-11.1.8/bin/beam.smp -W w -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs 512 -MMmcs 30 -P 1048576 -t 5000000 -stbt db -zdbbl 128000 -sbwt none -sbwtdcpu none -sbwtdio none -B i -- -root /usr/lib64/erlang -progname erl -- -home /var/lib/rabbitmq -- -pa  -noshell -noinput -s rabbit boot -boot start_sasl -rabbitmq_management listener [{port,15673}] -lager crash_log false -lager handlers [] -noshell -noinput
rabbitmq 13929  0.0  0.0   4356   532 ?        Ss   13:41   0:00 erl_child_setup 65535
rabbitmq 13973  0.0  0.0  11592   456 ?        Ss   13:41   0:00 inet_gethost 4
rabbitmq 13974  0.0  0.0  41552  1200 ?        S    13:41   0:00 inet_gethost 4
rabbitmq 14003 96.4  2.6 2306668 98156 ?       Sl   13:41   0:08 /usr/lib64/erlang/erts-11.1.8/bin/beam.smp -W w -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs 512 -MMmcs 30 -P 1048576 -t 5000000 -stbt db -zdbbl 128000 -sbwt none -sbwtdcpu none -sbwtdio none -B i -- -root /usr/lib64/erlang -progname erl -- -home /var/lib/rabbitmq -- -pa  -noshell -noinput -s rabbit boot -boot start_sasl -rabbitmq_management listener [{port,15674}] -lager crash_log false -lager handlers [] -noshell -noinput
rabbitmq 14007  0.0  0.0   4356   528 ?        Ss   13:41   0:00 erl_child_setup 65535
rabbitmq 14051  0.0  0.0  11592   456 ?        Ss   13:41   0:00 inet_gethost 4
rabbitmq 14052  0.0  0.0  41552  1196 ?        S    13:41   0:00 inet_gethost 4

```



### 02-4、加入集群

1、重置rabbit-1为主节点：

```bash
# 停止应用
> sudo rabbitmqctl -n rabbit-1 stop_app
# 目的是清除节点上的历史数据（如果不清除，无法将节点加入到集群）
> sudo rabbitmqctl -n rabbit-1 reset
# 启动应用
> sudo rabbitmqctl -n rabbit-1 start_app
```

2、将rabbit-2、rabbit-3加入到集群

```bash
# 停止应用
sudo rabbitmqctl -n rabbit-2 stop_app
# 目的是清除节点上的历史数据（如果不清除，无法将节点加入到集群）
sudo rabbitmqctl -n rabbit-2 reset
# 将rabbit2节点加入到rabbit1（主节点）集群当中【节点名@主机名】--ram 表示这是一个内存节点
sudo rabbitmqctl -n rabbit-2 join_cluster rabbit-1@`hostname -s` --ram
sudo rabbitmqctl -n rabbit-2 start_app

# rabbit-3 节点重置后加入集群
sudo rabbitmqctl -n rabbit-3 stop_app
sudo rabbitmqctl -n rabbit-3 reset
# --disc表示磁盘节点（默认不写也是磁盘节点）
sudo rabbitmqctl -n rabbit-3 join_cluster rabbit-1@`hostname -s` --disc
sudo rabbitmqctl -n rabbit-3 start_app
```



### 02-5、验证集群状态

命令：sudo rabbitmqctl cluster_status -n rabbit-1

```bash
[root@CentOS7 ~]# rabbitmqctl cluster_status -n rabbit-1
Cluster status of node rabbit-1@CentOS7 ...
Basics

Cluster name: rabbit-1@CentOS7

Disk Nodes

rabbit-1@CentOS7
rabbit-3@CentOS7

RAM Nodes

rabbit-2@CentOS7

Running Nodes

rabbit-1@CentOS7
rabbit-2@CentOS7
rabbit-3@CentOS7

Versions

rabbit-1@CentOS7: RabbitMQ 3.8.14 on Erlang 23.2.7
rabbit-2@CentOS7: RabbitMQ 3.8.14 on Erlang 23.2.7
rabbit-3@CentOS7: RabbitMQ 3.8.14 on Erlang 23.2.7

Maintenance status

Node: rabbit-1@CentOS7, status: not under maintenance
Node: rabbit-2@CentOS7, status: not under maintenance
Node: rabbit-3@CentOS7, status: not under maintenance

Alarms

(none)

Network Partitions

(none)

Listeners

Node: rabbit-1@CentOS7, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit-1@CentOS7, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit-1@CentOS7, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Node: rabbit-2@CentOS7, interface: [::], port: 15673, protocol: http, purpose: HTTP API
Node: rabbit-2@CentOS7, interface: [::], port: 25673, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit-2@CentOS7, interface: [::], port: 5673, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Node: rabbit-3@CentOS7, interface: [::], port: 15674, protocol: http, purpose: HTTP API
Node: rabbit-3@CentOS7, interface: [::], port: 25674, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit-3@CentOS7, interface: [::], port: 5674, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0

Feature flags

Flag: drop_unroutable_metric, state: enabled
Flag: empty_basic_get_metric, state: enabled
Flag: implicit_default_bindings, state: enabled
Flag: maintenance_mode_status, state: enabled
Flag: quorum_queue, state: enabled
Flag: user_limits, state: enabled
Flag: virtual_host_metadata, state: enabled

```

![20210418110851](https://gitee.com/liusuixing/blog_images/raw/master/20210418110851.png)



web页面查看：

![20210418110824](https://gitee.com/liusuixing/blog_images/raw/master/20210418110824.png)



### 02-6、解除集群

1、在自己节点重置当前节点，退出集群

```bash
# 停止服务、重置集群状态、重启服务
rabbitmqctl -n rabbit-2 stop_app && rabbitmqctl -n rabbit-2 reset && rabbitmqctl -n rabbit-2 start_app
```

2、在集群其他任意一个正常节点将节点踢出集群

```bash
# 必须先关闭当前节点才能把该节点解除集群
sudo rabbitmqctl -n rabbit-3@CentOS7 stop_app
# 在节点1上操作，把节点3踢出集群（可以在任意节点操作，不一定要在主节点）
sudo rabbitmqctl -n rabbit-1 forget_cluster_node rabbit-3@CentOS7
```



### 02-7、删除集群模式

```bash
# 1.暂停rabbitmq节点服务
sudo rabbitmqctl -n rabbit-3 stop
sudo rabbitmqctl -n rabbit-2 stop
sudo rabbitmqctl -n rabbit-1 stop
# 2.清空目录
rm -rf /var/lib/rabbitmq/mnesia/
```



> 访问web端需要新增账户：

```bash
rabbitmqctl -n rabbit-1 add_user admin admin && rabbitmqctl -n rabbit-1 set_user_tags admin administrator && rabbitmqctl -n rabbit-1 set_permissions -p / admin ".*" ".*" ".*"

rabbitmqctl -n rabbit-2 add_user admin admin && rabbitmqctl -n rabbit-2 set_user_tags admin administrator &&rabbitmqctl -n rabbit-2 set_permissions -p / admin ".*" ".*" ".*"

rabbitmqctl -n rabbit-3 add_user admin admin && rabbitmqctl -n rabbit-3 set_user_tags admin administrator &&rabbitmqctl -n rabbit-3 set_permissions -p / admin ".*" ".*" ".*"
```





## 03、多机多节点部署

> **Tips：**
> 如果采用多机部署方式，需读取其中一个节点的cookie, 并复制到其他节点（节点之间通过cookie确定相互是否可通信）。cookie存放在/var/lib/rabbitmq/.erlang.cookie。
> 例如：主机名分别为rabbit-1、rabbit-2
> 1、逐个启动各节点
> 2、配置各节点的hosts文件( vim /etc/hosts)
>  ip1：rabbit-1
>  ip2：rabbit-2
> 其它步骤雷同单机部署方式



### 03-1、环境准备

准备三台服务器CentOS 7.8：**192.168.3.12、192.168.3.13、192.168.3.14**

**`提示：如果使用虚拟机，可以在一台VM上安装好RabbitMQ后，创建快照，从快照创建链接克隆，会节省很多磁盘空间`**



### 03-2、初始化环境

1、分别修改三台主机名：

```bash
hostnamectl set-hostname rabbit-1
hostnamectl set-hostname rabbit-2
hostnamectl set-hostname rabbit-3
```

2、修改每台机器的 `/etc/hosts` 文件

```bash
cat >> /etc/hosts <<EOF
192.168.3.12 rabbit-1
192.168.3.13 rabbit-2
192.168.3.14 rabbit-3
EOF
```

3、同步三台直接的.erlang.cookie信息（拷贝.cookie时，**各节点都必须停止MQ服务**）

```bash
scp /var/lib/rabbitmq/.erlang.cookie root@rabbit-2:/var/lib/rabbitmq/
scp /var/lib/rabbitmq/.erlang.cookie root@rabbit-3:/var/lib/rabbitmq/
```

由于你可能在三台主机上使用不同的账户进行操作，为避免后面出现权限不足的问题，这里建议将 cookie 文件原来的 400 权限改为 600，命令如下：

```bash
chmod 600 /var/lib/rabbitmq/.erlang.cookie
```



### 03-3、启动服务加入集群

1、首先每台机器开启防火墙

```bash
# 添加端口
sudo firewall-cmd --zone=public --add-port=4369/tcp --permanent
sudo firewall-cmd --zone=public --add-port=5672/tcp --permanent
sudo firewall-cmd --zone=public --add-port=25672/tcp --permanent
sudo firewall-cmd --zone=public --add-port=15672/tcp --permanent

# 重启防火墙
sudo firewall-cmd --reload
```



2、启动每台机器RabbitMQ服务

```bash
sudo systemctl start rabbitmq-server
或者
rabbitmq-server -detached
```



3、重置节点1

```bash
# 停止RabbitMQ 应用
rabbitmqctl stop_app
# 重置RabbitMQ 设置
rabbitmqctl reset
# 启动RabbitMQ 应用
rabbitmqctl start_app
```

4、重置节点2 并加入集群：

```bash
# 停止RabbitMQ 应用
rabbitmqctl stop_app
# 重置RabbitMQ 设置
rabbitmqctl reset
# 加入到集群
rabbitmqctl join_cluster rabbit@rabbit-1 --ram
# 启动RabbitMQ 应用
rabbitmqctl start_app
```

5、重置节点3 并加入集群：

```bash
# 停止 RabbitMQ 应用
rabbitmqctl stop_app
# 重置 RabbitMQ 设置
rabbitmqctl reset
# 节点加入到集群
rabbitmqctl join_cluster rabbit@rabbit-3 --ram
# 启动 RabbitMQ 应用
rabbitmqctl start_app
```

6、查看集群（也可以直接通过web页面查看）

```bash
rabbitmqctl cluster_status
```



### 03-4、集群的关闭与重启

没有一个直接的命令可以关闭整个集群，需要逐一进行关闭。但是需要保证在重启时，最后关闭的节点最先被启动。如果第一个启动的不是最后关闭的节点，那么这个节点会等待最后关闭的那个节点启动，默认进行 10 次连接尝试，超时时间为 30 秒，如果依然没有等到，则该节点启动失败。

这带来的一个问题是，假设在一个三节点的集群当中，关闭的顺序为 node1，node2，node3，如果 node1 因为故障暂时没法恢复，此时 node2 和 node3 就无法启动。想要解决这个问题，可以先将 node1 节点进行剔除，命令如下：

```
rabbitmqctl forget_cluster_node rabbit@node1 --offline
```

此时需要加上 `-offline` 参数，它允许节点在自身没有启动的情况下将其他节点剔除



### 03-5、集群的解除

> 两种方式：
> 1：在当前自己节点重置退出集群
> 2：在其他任意点操作将某节点踢出集群
> 注意：被解除集群的节点必须要停止服务，不然无法解除

1、在自己节点重置当前节点，退出集群

```bash
# 1.停止服务
rabbitmqctl stop_app
# 2.重置集群状态
rabbitmqctl reset
# 3.重启服务
rabbitmqctl start_app
```

2、在集群其他任意一个正常节点将节点踢出集群

```bash
rabbitmqctl forget_cluster_node rabbit@rabbit-node3
```



### 03-6、变更节点类型

我们可以将节点的类型从RAM更改为Disk，反之亦然。假设我们想要反转rabbit@rabbit-node2和rabbit@rabbit-node1的类型，将前者从RAM节点转换为磁盘节点，而后者从磁盘节点转换为RAM节点。为此，我们可以使用change_cluster_node_type命令。必须首先停止节点。

```bash
# 1.停止服务
rabbitmqctl stop_app
# 2.变更类型 ram disc
rabbitmqctl change_cluster_node_type disc
# 3.重启服务
rabbitmqctl start_app
```

### 03-7、清除节点配置

```bash
# 如果遇到不能正常退出直接kill进程
systemctl stop rabbitmq-server
# 查看进程
ps aux|grep rabbitmq
# 清楚节点rabbitmq配置
rm -rf /var/lib/rabbitmq/mnesia
```





## 04、Dokcer 部署

### 04-1、docker

> 单机版：
> 步骤一：先启动运行三个RabbitMQ容器，这里我选择的是rabbitmq:3.8.3-management，带web插件版本的。
> 步骤二：加入RabbitMQ节点到集群（**备注**：参数“--ram”表示设置为内存节点，忽略次参数默认为磁盘节点）

```bash
==========================
--link模式网络模式：
==========================
# 启动三个服务
docker run -d --hostname rabbit1 --name myrabbit1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -p 5672:5672 -p 15672:15672 rabbitmq:3.8.3-management
docker run -d --hostname rabbit2 --name myrabbit2 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -p 5673:5672 -p 15673:15672 --link myrabbit1:rabbit1 rabbitmq:3.8.3-management
docker run -d --hostname rabbit3 --name myrabbit3 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -p 5674:5672 -p 15674:15672 --link myrabbit1:rabbit1 --link myrabbit2:rabbit2 rabbitmq:3.8.3-management


# 设置节点一
docker exec rabbitmq1 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl start_app"

# 设置节点2，加入到集群
docker exec rabbitmq2 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster --ram rabbit@rabbitmq1 && rabbitmqctl start_app"

# 设置节点3，加入到集群：
docker exec rabbitmq3 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster --ram rabbit@rabbitmq1 && rabbitmqctl start_app"
```



```bash
==========================
network自定义网络模式：
==========================
# 1、先创建自定义网络模式
docker network create rabbit_net
# 2、启动三个RabbitMQ容器，指定network为rabbit_net
docker run -d --hostname rabbitmq1 --name rabbitmq1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -e RABBITMQ_NODENAME=rabbitmq1 -p 5672:5672 -p 15672:15672 --network rabbit_net rabbitmq:3.8.3-management
docker run -d --hostname rabbitmq2 --name rabbitmq2 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -e RABBITMQ_NODENAME=rabbitmq2 -p 5673:5672 -p 15673:15672 --network rabbit_net rabbitmq:3.8.3-management
docker run -d --hostname rabbitmq3 --name rabbitmq3 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -e RABBITMQ_NODENAME=rabbitmq3 -p 5674:5672 -p 15674:15672 --network rabbit_net rabbitmq:3.8.3-management

# 设置节点一
docker exec rabbitmq1 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl start_app"

# 设置节点2，加入到集群
docker exec rabbitmq2 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster --ram rabbitmq1@rabbitmq1 && rabbitmqctl start_app"

# 设置节点3，加入到集群：
docker exec rabbitmq3 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster --ram rabbitmq1@rabbitmq1 && rabbitmqctl start_app"
```



查看集群状态（或者登录web界面查看）

```bash
docker exec rabbitmq1 bash -c "rabbitmqctl cluster_status"
```



### 04-2、docker-compose

步骤一：编写compose.yml配置文件

```yaml
version: '3.6'
services:
  rabbitmq1:
    image: rabbitmq:3.8.14-management
    container_name: rabbitmq1
    hostname: rabbitmq1
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      - RABBITMQ_ERLANG_COOKIE=CURIOAPPLICATION
      - RABBITMQ_NODENAME:rabbitmq1
    networks:
      - rabbitmq
  rabbitmq2:
    image: rabbitmq:3.8.14-management
    container_name: rabbitmq2
    hostname: rabbitmq2
    ports:
      - "5673:5672"
      - "15673:15672"
    environment:
      - RABBITMQ_ERLANG_COOKIE=CURIOAPPLICATION
      - RABBITMQ_NODENAME:rabbitmq2
    networks:
      - rabbitmq
  rabbitmq3:
    image: rabbitmq:3.8.14-management
    container_name: rabbitmq3
    restart: always
    hostname: rabbitmq3
    ports:
      - "5674:5672"
      - "15674:15672"
    environment:
      - RABBITMQ_ERLANG_COOKIE=CURIOAPPLICATION
      - RABBITMQ_NODENAME:rabbitmq3
    networks:
      - rabbitmq
networks:
  rabbitmq:
    driver: bridge
```

步骤二：设置集群

```bash
# 设置节点1
docker exec rabbitmq1 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl start_app"

# 设置节点2，加入到集群
docker exec rabbitmq2 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster --ram rabbitmq1@rabbitmq1 && rabbitmqctl start_app"

# 设置节点3，加入到集群：
docker exec rabbitmq3 bash -c "rabbitmqctl stop_app && rabbitmqctl reset && rabbitmqctl join_cluster --ram rabbitmq1@rabbitmq1 && rabbitmqctl start_app"
```



## 05、镜像模式搭建

> 镜像队列模式相比较普通模式，镜像模式会占用更多的带宽来进行同步，所以镜像队列的吞吐量会低于普通模式
>
> 普通模式不能实现高可用，某个节点挂了后，这个节点上的消息将无法被消费，需要等待节点启动后才能被消费

镜像队列是**Rabbit2.6.0**版本带来的一个新功能，允许内建双活冗余选项，与普通队列不同，镜像节点在集群中的其他节点拥有从队列拷贝，一旦主节点不可用，最老的从队列将被选举为新的主队列。

**镜像队列的工作原理：**在某种程度上你可以将镜像队列视为，拥有一个隐藏的fanout交换器，它指示者信道将消息分发到从队列上。



实现镜像模式非常简单，有3种方式：**管理平台命令创建**、**可视化管理界面手动创建**、**声明队列的时候创建**

### 1、管理平台命令行创建

```bash
# 创建镜像模式
rabbitmqctl set_policy [-p Vhost] Name Pattern Definition [Priority]
```

下面是对这些配置的解释：

- **-p Vhost**：可选参数，针对指定vhost下的queue进行设置。
- **Name**：策略名称。
- **Pattern**：队列名称的匹配规则，使用正则表达式表示；
- **Definition**：镜像队列的主体规则，json字符串，包括三个部分：ha-mode, ha-params, ha-sync-mod
  - ha-mode：指明镜像队列的模式，有效值为 all、exactly、nodes
    - all：表示在集群中所有的节点上进行镜像
    - exactly：表示在指定个数的节点上进行镜像，节点的个数由ha-params指定
    - nodes：表示在指定的节点上进行镜像，节点名称通过ha-params指定
  - ha-params：ha-mode模式需要用到的参数补充
  - ha-sync-mode：镜像消息同步方式：automatic（自动）manually（手动-默认）
    - manual手动<默认模式>.新的队列镜像将不会收到现有的消息，它只会接收新的消息。
    - automatic自动同步.当一个新镜像加入时，队列会自动同步。队列同步是一个阻塞操作



**创建镜像队列**（在任意节点上操作都可以）

```bash
# 配置所有名字开头为policy的队列进行镜像，镜像数量为2那么命令如下
rabbitmqctl set_policy -p / ha-policy "^policy_" '{"ha-mode":"exactly","ha-params":2}'

# 设置策略匹配所有名称的队列都进行高可用配置
rabbitmqctl set_policy -p / ha "^" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
```

**查看镜像队列**

```bash
# 查看所有的镜像队列
rabbitmqctl list_policies

# 查看vhost下的所有的策略（policies）
rabbitmqctl list_policies -p / 
```

**删除镜像队列**

```bash
rabbitmqctl clear_policy [-p <vhost>] <name>

# 清理名称为 ha-policy 的镜像队列
rabbitmqctl clear_policy -p / ha-policy
```



### 2、可视化管理界面手动创建

> 添加方式：进入管理页面 -> Admin -> Policies（在页面右侧）-> Add / update a policy

全部节点镜像策略创建：

![20210418110852](https://gitee.com/liusuixing/blog_images/raw/master/20210418110852.png)

指定节点数镜像策略创建：

![20210418110853](https://gitee.com/liusuixing/blog_images/raw/master/20210418110853.png)



**参数说明：**

- **name**: 策略名称，如果使用已有的名称，保存后将会修改原来的信息

- **Apply to**：策略应用到什么对象上（一般选择交换机和队列一起）

- **Pattern**：策略应用到对象时，对象名称的匹配规则（正则表达式）

- **Priority**：优先级，数值越大，优先级越高，相同优先级取最后一个

- **Definition**：策略定义的类容，对于镜像队列的配置来说，只需要包含3个部分: ha-mode 、ha-params 和 ha-sync-mode。其中，ha-sync-mode是同步的方式，自动还是手动，默认是手动。ha-mode 和 ha-params 组合使用。组合方式如下：

| ha-mode | ha-params | 说明                                                         |
| ------- | --------- | ------------------------------------------------------------ |
| all     | (empty)   | 队列镜像到集群类所有节点                                     |
| exactly | count     | 队列镜像到集群内指定数量的节点。如果集群内节点数少于此值，队列将会镜像到所有节点。<br />如果大于此值，而且一个包含镜像的节点停止，则新的镜像不会在其它节点创建。 |
| nodes   | nodename  | 队列镜像到指定节点，指定的节点不在集群中不会报错。<br />当队列申明时，如果指定的节点不在线，则队列会被创建在客户端所连接的节点上。 |



### 3、声明队列的时候创建（不推荐）

```java
// 声明队列的时候可以加入镜像队列参数（一般不推荐使用）
Map<String,Object> arguments = new HashMap<>();
// x-ha-policy：创建HA队列
arguments.put("x-ha-policy",‘"nodes");
// x-ha-nodes：HA队列的分布节点
arguments.put("x-ha-nodes","rabbit@rabbitmq1");
channel.queueDeclare("ttl-queue", false, false, false, arguments);
```





## 06、集群监控管理

> 在广大的互联网行业中RabbitMQ几乎都会有集群，那么对于集群的监控就成了企业生态中必不可少的。主要有如下四种监控：

### 1、管理界面监控

管理界面监控需要我们开启对应的插件（rabbitmq-plugins enable rabbitmq_management）访问：http://ip:15672

![20210418110829](https://gitee.com/liusuixing/blog_images/raw/master/20210418110829.png)

在管理控制台我们就可以直观的看到集群中的每一个节点是否正常,如果为红色则表示节点挂掉了，同时可以很方便的查看到各个节点的内存、磁盘等相关的信息，使用起来也是非常方便的。但是遗憾的该功能做的比较简陋,没有告警等一些列的个性化设置，同时如果想把他接入到公司其他的监控系统统一管理也是很难做到的,所以扩展性不强，一般在小型企业的小集群中使用。



### 2、Tracing日志监控

> 对于企业级的应用开发来讲,我们通常都会比较关注我们的消息，甚至很多的场景把消息的可靠性放在第一位，但是我们的MQ集群难免会出现消息异常丢失或者客户端无法发送消息等异常情况，此时为了帮助开发人员快速的定位问题，我们就可以对消息的投递和消费过程进行监控，而Tracing日志监控插件帮我们很好的实现了该功能。
>
> 消息中心的消息追踪需要使用Trace实现，Trace是Rabbitmq用于记录每一次发送的消息，方便使用Rabbitmq的开发者调试、排错。可通过插件形式提供可视化界面。Trace启动后会自动创建系统Exchange：amq.rabbitmq.trace，每个队列会自动绑定该Exchange，绑定后发送到队列的消息都会记录到Trace日志。

**1、消息追踪启用与查看**：以下是trace的相关命令和使用（要使用需要先rabbitmq启用插件，再打开开关才能使用）

| 命令集                                        | 描述                                           |
| --------------------------------------------- | ---------------------------------------------- |
| rabbitmq-plugins list                         | 查看插件列表                                   |
| rabbitmq-plugins enable rabbitmq_tracing      | rabbitmq启用trace插件                          |
| rabbitmqctl trace_on                          | 打开trace的开关                                |
| rabbitmqctl trace_on -p itcast                | 打开trace的开关（itcast为需要日志追踪的vhost） |
| rabbitmqctl trace_off                         | 关闭trace的开关                                |
| rabbitmq-plugins disable rabbitmq_tracing     | rabbitmq关闭Trace插件                          |
| rabbitmqctl set_user_tags heima administrator | 只有administrator的角色才能查看日志界面        |

安装插件并开启 trace_on 之后，会发现多个 exchange：amq.rabbitmq.trace ，类型为：topic。

![20210418110830](https://gitee.com/liusuixing/blog_images/raw/master/20210418110830.png)



**2、日志追踪**

1、发送消息

```java
rabbitTemplate.convertAndSend("spring_queue", "只发队列spring_queue的消息--01");
```

2、查看trace

![20210418110831](https://gitee.com/liusuixing/blog_images/raw/master/20210418110831.png)

3、点击Tracing查看Trace log files

![20210418110832](https://gitee.com/liusuixing/blog_images/raw/master/20210418110832.png)

4、点击xuexiangban-trace.log 确认消息轨迹正确性

![20210418110833](https://gitee.com/liusuixing/blog_images/raw/master/20210418110833.png)





### 3、定制自己的监控系统

> RabbitMQ提供了很丰富的restful风格的api接口,我们可以通过这些接口得到对应的集群数据,此时我们就可以定制我们的监控系统。

| **HTTP API URL**                      | **HTTP 请求类型** | **接口含义**                                                 |
| ------------------------------------- | ----------------- | ------------------------------------------------------------ |
| /api/connections                      | GET               | 获取当前RabbitMQ集群下所有打开的连接                         |
| /api/nodes                            | GET               | 获取当前RabbitMQ集群下所有节点实例的状态信息                 |
| /api/vhosts/{vhost}/connections       | GET               | 获取某一个虚拟机主机下的所有打开的connection连接             |
| /api/connections/{name}/channels      | GET               | 获取某一个连接下所有的管道信息                               |
| /api/vhosts/{vhost}/channels          | GET               | 获取某一个虚拟机主机下的管道信息                             |
| /api/consumers/{vhost}                | GET               | 获取某一个虚拟机主机下的所有消费者信息                       |
| /api/exchanges/{vhost}                | GET               | 获取某一个虚拟机主机下面的所有交换器信息                     |
| /api/queues/{vhost}                   | GET               | 获取某一个虚拟机主机下的所有队列信息                         |
| /api/users                            | GET               | 获取集群中所有的用户信息                                     |
| /api/users/{name}                     | GET/PUT/DELETE    | 获取/更新/删除指定用户信息                                   |
| /api/users/{user}/permissions         | GET               | 获取当前指定用户的所有权限信息                               |
| /api/permissions/{vhost}/{user}       | GET/PUT/DELETE    | 获取/更新/删除指定虚拟主机下特定用户的权限                   |
| /api/exchanges/{vhost}/{name}/publish | POST              | 在指定的虚拟机主机和交换器上发布一个消息                     |
| /api/queues/{vhost}/{name}/get        | POST              | 在指定虚拟机主机和队列名中获取消息，同时该动作会修改队列状态 |
| /api/healthchecks/node/{node}         | GET               | 获取指定节点的健康检查状态                                   |

> 更多API的相关信息和描述可以访问：http://ip:15672/api/

接下来我们使用RabbitMQ Http API接口来获取集群监控数据：

1、创建一个普通maven项目

2、引入HttpClient以及Jackson的相关Jar依赖

```xml
    <properties>
        <!--锁定 jdk 版本为 1.8-->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.3.6</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.7.4</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.7.4</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.7.4</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.16</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

3、编写监控代码：

```java
package com.example;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;
import org.apache.http.HttpEntity;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.auth.BasicScheme;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

/**
 * RabbitMQ的监控
 */
public class MonitorRabbitMQ {
    // RabbitMQ的HTTP API——获取集群各个实例的状态信息，ip替换为自己部署相应实例的
    private static String RABBIT_NODES_STATUS_REST_URL = "http://127.0.0.1:15672/api/nodes";
    // RabbitMQ的HTTP API——获取集群用户信息，ip替换为自己部署相应实例的
    private static String RABBIT_USERS_REST_URL = "http://127.0.0.1:15672/api/users";
    // rabbitmq的用户名
    private static String RABBIT_USER_NAME = "guest";
    // rabbitmq的密码
    private static String RABBIT_USER_PWD = "guest";

    public static void main(String[] args) {
        try {
            //step1.获取rabbitmq集群各个节点实例的状态信息
            Map<String, ClusterStatus> clusterMap =
                    fetchRabbtMQClusterStatus(RABBIT_NODES_STATUS_REST_URL, RABBIT_USER_NAME, RABBIT_USER_PWD);
            //step2.打印输出各个节点实例的状态信息
            for (Map.Entry entry : clusterMap.entrySet()) {
                System.out.println(entry.getKey() + " : " + entry.getValue());
            }
            //step3.获取rabbitmq集群用户信息
            Map<String, User> userMap =
                    fetchRabbtMQUsers(RABBIT_USERS_REST_URL, RABBIT_USER_NAME, RABBIT_USER_PWD);
            //step4.打印输出rabbitmq集群用户信息
            for (Map.Entry entry : userMap.entrySet()) {
                System.out.println(entry.getKey() + " : " + entry.getValue());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static Map<String, ClusterStatus> fetchRabbtMQClusterStatus(String url, String username, String password) throws IOException {
        Map<String, ClusterStatus> clusterStatusMap = new HashMap<String, ClusterStatus>();
        String nodeData = getData(url, username, password);
        JsonNode jsonNode = null;
        try {
            jsonNode = JsonUtil.toJsonNode(nodeData);
        } catch (IOException e) {
            e.printStackTrace();
        }
        Iterator<JsonNode> iterator = jsonNode.iterator();
        while (iterator.hasNext()) {
            JsonNode next = iterator.next();
            ClusterStatus status = new ClusterStatus();
            status.setDiskFree(next.get("disk_free").asLong());
            status.setFdUsed(next.get("fd_used").asLong());
            status.setMemoryUsed(next.get("mem_used").asLong());
            status.setProcUsed(next.get("proc_used").asLong());
            status.setSocketUsed(next.get("sockets_used").asLong());
            clusterStatusMap.put(next.get("name").asText(), status);
        }
        return clusterStatusMap;
    }
    public static Map<String, User> fetchRabbtMQUsers(String url, String username, String password) throws IOException {
        Map<String, User> userMap = new HashMap<String, User>();
        String nodeData = getData(url, username, password);
        JsonNode jsonNode = null;
        try {
            jsonNode = JsonUtil.toJsonNode(nodeData);
        } catch (IOException e) {
            e.printStackTrace();
        }
        Iterator<JsonNode> iterator = jsonNode.iterator();
        while (iterator.hasNext()) {
            JsonNode next = iterator.next();
            User user = new User();
            user.setName(next.get("name").asText());
            user.setTags(next.get("tags").asText());
            userMap.put(next.get("name").asText(), user);
        }
        return userMap;
    }
    public static String getData(String url, String username, String password) throws IOException {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        UsernamePasswordCredentials creds = new UsernamePasswordCredentials(username, password);
        HttpGet httpGet = new HttpGet(url);
        httpGet.addHeader(BasicScheme.authenticate(creds, "UTF-8", false));
        httpGet.setHeader("Content-Type", "application/json");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        try {
            if (response.getStatusLine().getStatusCode() != 200) {
                System.out.println("call http api to get rabbitmq data return code: " + response.getStatusLine().getStatusCode() + ", url: " + url);
            }
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                return EntityUtils.toString(entity);
            }
        } finally {
            response.close();
        }
        return null;
    }

    public static class JsonUtil {
        private static ObjectMapper objectMapper = new ObjectMapper();
        static {
            objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
            //objectMapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
        }
        public static JsonNode toJsonNode(String jsonString) throws IOException {
            return objectMapper.readTree(jsonString);
        }
    }

    @Data
    public static class User {
        private String name;
        private String tags;
    }

    @Data
    public static class ClusterStatus {
        private long diskFree;
        private long diskLimit;
        private long fdUsed;
        private long fdTotal;
        private long socketUsed;
        private long socketTotal;
        private long memoryUsed;
        private long memoryLimit;
        private long procUsed;
        private long procTotal;
    }
}
```

控制台输出：

```
rabbit-3@CentOS7 : MonitorRabbitMQ.ClusterStatus(diskFree=36523638784, diskLimit=0, fdUsed=34, fdTotal=0, socketUsed=0, socketTotal=0, memoryUsed=86605824, memoryLimit=0, procUsed=443, procTotal=0)
rabbit-1@CentOS7 : MonitorRabbitMQ.ClusterStatus(diskFree=36523638784, diskLimit=0, fdUsed=36, fdTotal=0, socketUsed=0, socketTotal=0, memoryUsed=89972736, memoryLimit=0, procUsed=444, procTotal=0)
rabbit-2@CentOS7 : MonitorRabbitMQ.ClusterStatus(diskFree=36523638784, diskLimit=0, fdUsed=34, fdTotal=0, socketUsed=0, socketTotal=0, memoryUsed=89382912, memoryLimit=0, procUsed=442, procTotal=0)
admin : MonitorRabbitMQ.User(name=admin, tags=administrator)
guest : MonitorRabbitMQ.User(name=guest, tags=administrator)
```



### 4、Zabbix监控RabbitMQ

> Zabbix是一个基于WEB界面提供分布式系统监视以及网络监视功能的企业级开源解决方案,他也可以帮助我们搭建一个MQ集群的监控系统,同时提供预警等功能，但是由于其搭建配置要求比较高一般都是由运维人员负责搭建,感兴趣的同学可以访问：https://www.zabbix.com/ 官网进行了解学习。





## 07、SpringBoot集群配置

### 1、引入相关依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 2、配置文件详细

```yaml
 rabbitmq:
    # 指定client连接到的server的地址，多个以逗号分隔(优先取addresses，然后再取host)
    addresses: 127.0.0.1:6605,127.0.0.1:6606,127.0.0.1:6705 
#    port:
    ##集群配置 addresses之间用逗号隔开
    # addresses: ip:port,ip:port
    password: admin
    username: 123456
    virtual-host: / # 连接到rabbitMQ的vhost
    requested-heartbeat: #指定心跳超时，单位秒，0为不指定；默认60s
    publisher-confirms: #是否启用 发布确认
    publisher-reurns: # 是否启用发布返回
    connection-timeout: #连接超时，单位毫秒，0表示无穷大，不超时
    cache:
      channel.size: # 缓存中保持的channel数量
      channel.checkout-timeout: # 当缓存数量被设置时，从缓存中获取一个channel的超时时间，单位毫秒；如果为0，则总是创建一个新channel
      connection.size: # 缓存的连接数，只有是CONNECTION模式时生效
      connection.mode: # 连接工厂缓存模式：CHANNEL 和 CONNECTION
    listener:
      simple.auto-startup: # 是否启动时自动启动容器
      simple.acknowledge-mode: # 表示消息确认方式，其有三种配置方式，分别是none、manual和auto；默认auto
      simple.concurrency: # 最小的消费者数量
      simple.max-concurrency: # 最大的消费者数量
      simple.prefetch: # 指定一个请求能处理多少个消息，如果有事务的话，必须大于等于transaction数量.
      simple.transaction-size: # 指定一个事务处理的消息数量，最好是小于等于prefetch的数量.
      simple.default-requeue-rejected: # 决定被拒绝的消息是否重新入队；默认是true（与参数acknowledge-mode有关系）
      simple.idle-event-interval: # 多少长时间发布空闲容器时间，单位毫秒
      simple.retry.enabled: # 监听重试是否可用
      simple.retry.max-attempts: # 最大重试次数
      simple.retry.initial-interval: # 第一次和第二次尝试发布或传递消息之间的间隔
      simple.retry.multiplier: # 应用于上一重试间隔的乘数
      simple.retry.max-interval: # 最大重试时间间隔
      simple.retry.stateless: # 重试是有状态or无状态
    template:
      mandatory: # 启用强制信息；默认false
      receive-timeout: # receive() 操作的超时时间
      reply-timeout: # sendAndReceive() 操作的超时时间
      retry.enabled: # 发送重试是否可用
      retry.max-attempts: # 最大重试次数
      retry.initial-interval: # 第一次和第二次尝试发布或传递消息之间的间隔
      retry.multiplier: # 应用于上一重试间隔的乘数
      retry.max-interval: #最大重试时间间隔
```

> 注：相关配置很多，大家只需要关注一些常用的配置即可

**对于发送方而言，需要做以下配置：**

- 1、配置CachingConnectionFactory

- 2、配置Exchange/Queue/Binding

- 3、配置RabbitAdmin创建上一步的Exchange/Queue/Binding

- 4、配置RabbitTemplate用于发送消息，RabbitTemplate通过CachingConnectionFactory获取到Connection，然后想指定Exchange发送

**对于消费方而言，需要做以下配置：**

- 1、配置CachingConnectionFactory
- 2、配置Exchange/Queue/Binding
- 3、配置RabbitAdmin创建上一步的Exchange/Queue/Binding
- 4、配置RabbitListenerContainerFactory
- 5、配置[@RabbitListener](https://github.com/RabbitListener)/[@RabbitHandler](https://github.com/RabbitHandler)用于接收消息



**在默认情况下主要的配置如下：**

![20210418110854](https://gitee.com/liusuixing/blog_images/raw/master/20210418110854.png)



### 3、Spring-AMQP主要对象

注：如果不了解AMQP请前往官网了解.

![20210418110855](https://gitee.com/liusuixing/blog_images/raw/master/20210418110855.png)



### 4、通过配置类加载的方式

```java
package com.yd.demo.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitAdmin;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.listener.RabbitListenerContainerFactory;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class RabbitConfig {
    private static final Logger logger = LoggerFactory.getLogger(RabbitConfig.class);
    public static final String RECEIVEDLXEXCHANGE="spring-ex";
    public static final String RECEIVEDLXQUEUE="spring-qu1";
    public static final String RECEIVEDLXROUTINGKEY="aa";
    public static final String DIRECTEXCHANGE="spring-ex";
    public static final String MDMQUEUE="mdmQueue";
    public static final String TOPICEXCHANGE="spring-top";
    @Value("${spring.rabbitmq.addresses}")
    private String hosts;
    @Value("${spring.rabbitmq.username}")
    private String userName;
    @Value("${spring.rabbitmq.password}")
    private String password;
    @Value("${spring.rabbitmq.virtual-host}")
    private String virtualHost;
 /*   @Value("${rabbit.channelCacheSize}")
    private int channelCacheSize;*/
//    @Value("${rabbit.port}")
//    private int port;
/*    @Autowired
    private ConfirmCallBackListener confirmCallBackListener;
    @Autowired
    private ReturnCallBackListener returnCallBackListener;*/
    @Bean
    public ConnectionFactory connectionFactory(){
        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory();
        cachingConnectionFactory.setAddresses(hosts);
        cachingConnectionFactory.setUsername(userName);
        cachingConnectionFactory.setPassword(password);
//        cachingConnectionFactory.setChannelCacheSize(channelCacheSize);
        //cachingConnectionFactory.setPort(port);
        cachingConnectionFactory.setVirtualHost(virtualHost);
        //设置连接工厂缓存模式：
        cachingConnectionFactory.setCacheMode(CachingConnectionFactory.CacheMode.CONNECTION);
        //缓存连接数
        cachingConnectionFactory.setConnectionCacheSize(3);
        //设置连接限制
        cachingConnectionFactory.setConnectionLimit(6);
        logger.info("连接工厂设置完成，连接地址{}"+hosts);
        logger.info("连接工厂设置完成，连接用户{}"+userName);
        return cachingConnectionFactory;
    }
    @Bean
    public RabbitAdmin rabbitAdmin(){
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory());
        rabbitAdmin.setAutoStartup(true);
        rabbitAdmin.setIgnoreDeclarationExceptions(true);
        rabbitAdmin.declareBinding(bindingMdmQueue());
        //声明topic交换器
        rabbitAdmin.declareExchange(directExchange());
        logger.info("管理员设置完成");
        return rabbitAdmin;
    }
    @Bean
    public RabbitListenerContainerFactory listenerContainerFactory() {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        //最小消费者数量
        factory.setConcurrentConsumers(10);
        //最大消费者数量
        factory.setMaxConcurrentConsumers(10);
        //一个请求最大处理的消息数量
        factory.setPrefetchCount(10);
        //
        factory.setChannelTransacted(true);
        //默认不排队
        factory.setDefaultRequeueRejected(true);
        //手动确认接收到了消息
        factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        logger.info("监听者设置完成");
        return factory;
    }
    @Bean
    public DirectExchange directExchange(){
        return new DirectExchange(DIRECTEXCHANGE,true,false);
    }
    @Bean
    public Queue mdmQueue(){
        Map arguments = new HashMap<>();
        // 绑定该队列到私信交换机
        arguments.put("x-dead-letter-exchange",RECEIVEDLXEXCHANGE);
        arguments.put("x-dead-letter-routing-key",RECEIVEDLXROUTINGKEY);
        logger.info("队列交换机绑定完成");
        return new Queue(RECEIVEDLXQUEUE,true,false,false,arguments);
    }
    @Bean
    Binding bindingMdmQueue() {
        return BindingBuilder.bind(mdmQueue()).to(directExchange()).with("");
    }
    @Bean
    public RabbitTemplate rabbitTemplate(){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory());
        rabbitTemplate.setMandatory(true);
        //发布确认
//        rabbitTemplate.setConfirmCallback(confirmCallBackListener);
        // 启用发布返回
//        rabbitTemplate.setReturnCallback(returnCallBackListener);
        logger.info("连接模板设置完成");
        return rabbitTemplate;
    }
  /*  @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange(TOPICEXCHANGE,true,false);
    }*/
  /*
*//**
     * @return DirectExchange
     *//*
    @Bean
    public DirectExchange dlxExchange() {
        return new DirectExchange(RECEIVEDLXEXCHANGE,true,false);
    }
*//*
*
     * @return Queue
*//*
    @Bean
    public Queue dlxQueue() {
        return new Queue(RECEIVEDLXQUEUE,true);
    }
*//*
     * @return Binding
     *//*
    @Bean
    public Binding binding() {
        return BindingBuilder.bind(dlxQueue()).to(dlxExchange()).with(RECEIVEDLXROUTINGKEY);
    }*/
}
```



### 5、总结

通过两种方式加载：1.通过配置文件、 2.通过配置类

说明：上面是通过配置文件与配置类的方式去加载,常用的配置如上所示。实际使用中要生产方与消费方要分开配置，相关配置也会有小变动，大体配置不变。更多信息可查看官网配置。



# RabbitMQ 运维篇

## 01、持久化机制

> 什么是RibbitMQ持久化：就把信息写入到磁盘的过程。
>
> **持久消息**：把消息默认放在内存中是为了加快传输和消费的速度，存入磁盘是保证消息数据的持久化。
> **非持久消息**：是指当内存不够用的时候，会把消息和数据转移到磁盘，但是重启以后非持久化队列消息就丢失。

**RabbitMQ的持久化队列分为：**1.队列持久化、2.消息持久化、3.交换机持久化

不论是持久化的消息还是非持久化的消息都可以写入到磁盘中，只不过非持久的是等内存不足的情况下才会被写入到磁盘中。

![20210418110815](https://gitee.com/liusuixing/blog_images/raw/master/20210418110815.png)



### 1、RabbitMQ队列持久化

队列的持久化是定义队列时的durable参数来实现的，Durable为true时，队列才会持久化。

```java
// 参数1：名字  
// 参数2：是否持久化，
// 参数3：独du占的queue， 
// 参数4：不使用时是否自动删除，
// 参数5：其他参数
channel.queueDeclare(queueName,true,false,false,null);
```

其中参数2：设置为true，就代表的是持久化的含义。即durable=true。持久化的队列在web控制台中有一个`D` 的标记

![20210418110816](https://gitee.com/liusuixing/blog_images/raw/master/20210418110816.png)

![20210418110818](https://gitee.com/liusuixing/blog_images/raw/master/20210418110818.png)



### 2、RabbitMQ消息持久化

消息持久化是通过消息的属性deliveryMode来设置是否持久化，在发送消息时通过basicPublish的参数传入。

```java
// 参数1：交换机的名字
// 参数2：队列或者路由key
// 参数3：是否进行消息持久化
// 参数4：发送消息的内容
channel.basicPublish(exchangeName, routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
```

通过设置参数3，把消息本体设置为持久化



### 3、RabbitMQ交换机持久化

和队列一样，交换机也需要在定义的时候设置持久化的标识，否则在rabbit-server服务重启以后将丢失。

```java
// 参数1：交换机的名字
// 参数2：交换机的类型，topic/direct/fanout/headers
// 参数3：是否持久化
channel.exchangeDeclare(exchangeName,exchangeType,true);
```

![20210418110817](https://gitee.com/liusuixing/blog_images/raw/master/20210418110817.png)

![20210418110819](https://gitee.com/liusuixing/blog_images/raw/master/20210418110819.png)





## 02、内存磁盘的监控

### 1、RabbitMQ的内存警告

当内存使用超过配置的阈值或者磁盘空间剩余空间对于配置的阈值时，RabbitMQ会暂时阻塞客户端的连接，并且停止接收从客户端发来的消息，以此避免服务器的崩溃，客户端与服务端的心态检测机制也会失效。
如下图：

![20210418110823](https://gitee.com/liusuixing/blog_images/raw/master/20210418110823.png)



### 2、RabbitMQ的内存控制

参考帮助文档：https://rabbitmq.com/memory.html
当出现警告的时候，可以通过配置去修改和调整。可以通过命令行或者修改配置文件方式配置。
命令行方式重启后就失效了，配置文件方式不会随重启失效，但是修改完配置文件需重启应用。

#### 2-1、命令的方式

```bash
rabbitmqctl set_vm_memory_high_watermark <fraction>
rabbitmqctl set_vm_memory_high_watermark absolute 50MB
```

fraction/value 为内存阈值。默认情况是：0.4/2GB，代表的含义是：当RabbitMQ的内存超过40%时，就会产生警告并且阻塞所有生产者的连接。通过此命令修改阈值在Broker重启以后将会失效，通过修改配置文件方式设置的阈值则不会随着重启而消失，但修改了配置文件一样要重启broker才会生效。



示例操作演示：

```bash
rabbitmqctl set_vm_memory_high_watermark absolute 50MB
```

![20210418110820](https://gitee.com/liusuixing/blog_images/raw/master/20210418110820.png)

![20210418110821](https://gitee.com/liusuixing/blog_images/raw/master/20210418110821.png)

> 还原可以重启服务或者重新修改成默认：`rabbitmqctl set_vm_memory_high_watermark 0.4`



#### 2-2、配置文件方式

> 当前配置文件：/etc/rabbitmq/rabbitmq.conf

```bash
# 默认值
# vm_memory_high_watermark.relative = 0.4

# 使用relative相对值进行设置fraction,建议取值在04~0.7之间，不建议超过0.7.
# 使用absolute的绝对值的方式，但是是KB,MB,GB对应的命令如下
vm_memory_high_watermark.relative = 0.6 
vm_memory_high_watermark.absolute = 2GB 
```

**注意**：relative 与 absolute 只需配置一个即可，不需要2个同时配置，无意义。



### 3、RabbitMQ的内存换页

在某个Broker节点及内存阻塞生产者之前，它会尝试将队列中的消息换页到磁盘以释放内存空间，持久化和非持久化的消息都会写入磁盘中，其中持久化的消息本身就在磁盘中有一个副本，所以在转移的过程中持久化的消息会先从内存中清除掉。

> 默认情况下，内存到达的阈值是50%时就会换页处理。
> 也就是说，在默认情况下该内存的阈值是0.4的情况下，当内存超过0.4*0.5=0.2时，会进行换页动作。

比如有1000MB内存，当内存的使用率达到了400MB,已经达到了极限，但是因为配置的换页内存0.5，这个时候会在达到极限400mb之前，会把内存中的200MB进行转移到磁盘中。从而达到稳健的运行。

可以通过设置 `vm_memory_high_watermark_paging_ratio` 来进行调整

```bash
vm_memory_high_watermark.relative = 0.4
vm_memory_high_watermark_paging_ratio = 0.7（设置小于1的值）
```

**注意：**为什么设置小于1，因为你如果你设置为1的阈值。内存都已经达到了极限了。你在去换页意义不是很大了。



### 4、RabbitMQ的磁盘预警

参考帮助文档：https://rabbitmq.com/disk-alarms.html
当磁盘的剩余空间低于确定的阈值时，RabbitMQ同样会阻塞生产者，这样可以避免因非持久化的消息持续换页而耗尽磁盘空间导致服务器崩溃。

> 默认情况下：磁盘预警为50MB的时候会进行预警。表示当前磁盘空间第50MB的时候会阻塞生产者并且停止内存消息换页到磁盘的过程。
> 这个阈值可以减小，但是不能完全的消除因磁盘耗尽而导致崩溃的可能性。比如在两次磁盘空间的检查空隙内，第一次检查是：60MB ，第二检查可能就是1MB,就会出现警告。

1、命令方式修改如下：

```bash
# disk_limit：固定单位 KB MB GB
# fraction ：是相对阈值，建议范围在:1.0~2.0之间。（相对于内存）
rabbitmqctl set_disk_free_limit <disk_limit>
rabbitmqctl set_disk_free_limit mem_relative <fraction>
```



示例测试：

```bash
rabbitmqctl set_disk_free_limit 100GB
```

![20210418110822](https://gitee.com/liusuixing/blog_images/raw/master/20210418110822.png)

> 还原可以重启服务或者重新修改成默认：`rabbitmqctl set_disk_free_limit 50MB`



2、配置文件配置如下：

```bash
disk_free_limit.relative = 3.0
disk_free_limit.absolute = 50mb
```





# RabbitMQ 面试篇

**1、Rabbitmq 为什么需要信道，为什么不是TCP直接通信**

> 1、TCP的创建和销毁，开销大，创建要三次握手，销毁要4次分手。
>
> 2、如果不用信道，那应用程序就会TCP连接到Rabbit服务器，高峰时每秒成千上万连接就会造成资源的巨大浪费，而且==底层操作系统每秒处理tcp连接数也是有限制的，==必定造成性能瓶颈。
>
> 3、信道的原理是一条线程一条信道，多条线程多条信道同用一条TCP连接，一条TCP连接可以容纳无限的信道，即使每秒成千上万的请求也不会成为性能瓶颈。



**2：queue队列到底在消费者创建还是生产者创建？**

> 1： 一般建议是在rabbitmq操作面板创建。这是一种稳妥的做法。
> 2：按照常理来说，确实应该消费者这边创建是最好，消息的消费是在这边。这样你承受一个后果，可能我生产在生产消息可能会丢失消息。
> 3：在生产者创建队列也是可以，这样稳妥的方法，消息是不会出现丢失。
> 4：如果你生产者和消费都创建的队列，谁先启动谁先创建，后面启动就覆盖前面的





# 参考资料 & 鸣谢

- [RabbitMQ与AMQP模型概览](https://blog.csdn.net/l18848956739/article/details/100075797)
- [RabbitMQ，采用当前最新版RabbitMQ 3.8.12，Erlang 23.2](https://blog.csdn.net/lgl782519197/article/details/113775569)
- [Springboot整合RabbitMQ，你只需要这一篇，学不会来打我](https://blog.csdn.net/w306026355/article/details/105467182/)
- [RabbitMQ如何保证消息的可靠性](https://www.cnblogs.com/linjiqin/p/12683076.html)
- [一个基于RabbitMQ的可复用的事务消息方案](https://www.cnblogs.com/throwable/p/12266806.html)
- [CentOS7卸载Erlang和RabbitMQ](https://www.cnblogs.com/momo-88/p/13588736.html)
- [beixuan](https://www.cnblogs.com/beixuan/tag/RabbitMQ/)
- [hh blog](https://www.cnblogs.com/dtdx/tag/rabbitmq/)
- [RabbitMQ之集群模式总结](https://www.cnblogs.com/rouqinglangzi/p/10584748.html)
- [RabbitMQ入门指南--简单集群搭建](https://mp.weixin.qq.com/s/dG-BFr6pDdxVg0Qc0at2cg)
- [RabbitMQ 高可用集群搭建](https://blog.csdn.net/qq_28533563/article/details/107932737)
- [(企业级) RabbitMQ 普通集群配置_04](https://gblfy.blog.csdn.net/article/details/104526175)
- [(企业级) RabbitMQ 镜像集群配置_05](https://gblfy.blog.csdn.net/article/details/104487406)
- [RabbitMQ单机到集群完整搭建（一）](https://blog.csdn.net/weixin_45612794/article/details/106023090)
- [RabbitMq笔记⑤——Linux中RabbitMq安装和RabbitMq集群搭建（转载）](https://blog.csdn.net/qq_35262405/article/details/102954986)
- [RabbitMQ高可用-镜像模式部署使用](https://www.cnblogs.com/huligong1234/p/13549450.html)
- [02消息队列系列-RabbitMQ的初步认识、安装、web管理端使用](https://www.cnblogs.com/lucky9322/p/13778715.html)
- [RabbitMQ管理页面各种属性详解(三)](https://blog.csdn.net/c1149884598/article/details/113970100)
- [RabbitMQ-管理界面介绍](https://www.cnblogs.com/BNTang/articles/13777477.html)
- [RabbitMQ的可视化界面进行操作](https://www.cnblogs.com/jikeyi/p/13339071.html)
- [【SpringBoot MQ 系列】RabbitListener 消费基本使用姿势介绍](https://www.cnblogs.com/yihuihui/p/12600797.html)
- [RabbitMQ集群搭建-镜像模式（Haproxy + Keepalived）](https://www.cnblogs.com/cheyunhua/p/13772623.html)

