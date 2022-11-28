> 作者：fossi/幕后哈土奇；来源：https://mp.weixin.qq.com/s/GovNaQcII54TvUSc6Nqhhg
>
> 作者：fossi/幕后哈土奇。计算机网络系列：https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI3MDc5Mjk1MA==&action=getalbum&album_id=1799572444249161729&scene=173&from_msgid=2247486571&from_itemidx=1&count=3&nolastread=1#wechat_redirect

**本篇文章主要是为了后续学习做准备，我们即将学习HTTP不能仅仅学一个理论，需要自己搭建服务器来真实感受下，那么web服务一般都是部署在linux服务器上，所以我们要先解决linux环境准备的问题。**

如果你手头没有可用的linux系统，如果你需要多台linux主机进行演示学习，但是你没有钱买服务器，那么最简单的方式就是利用虚拟机来安装多个linux操作系统。本篇文章直击VirtualBox+Centos7.x的安装和使用，一步一图进行安装和配置说明，让虚拟机玩起来不费劲。

本文环境介绍：

- 宿主机：win10系统
- linux发行版本：CentOS Linux release 7.9.2009 (Core)
- virtualbox版本：6.1

前方高能，出发！



## 一、准备活动

### 1.1、VirtualBox

VirtualBox 是一款开源虚拟机软件，由德国 Innotek 公司开发，由 Sun Microsystems 公司（就是开发 Java 语言大名鼎鼎的 Sun 公司）出品，使用 Qt 编写，在 Sun 被 Oracle 收购后正式更名成 Oracle VM VirtualBox。

VirtualBox 号称是最强的免费虚拟机软件，它不仅具有丰富的特色，性能也很优异！

你或许听说过 VMWare 虚拟机，但是我们仍然选择使用 VirtualBox ，原因是VirtualBox占用资源小、免费开源；VMWare 是收费的商业软件、占用资源较大。

virtualbox下载地址：https://www.virtualbox.org/wiki/Downloads



### 1.2、Centos7.X

操作系统我选择Centos这个发行版本，Centos由于稳定、免费在国内使用十分广泛。

centos下载，我用了国内的163镜像源：http://mirrors.163.com/centos/7/isos/x86_64/

下载速度杠杠滴（十几兆每秒），如果这个链接失效了，请去谷歌找下其他的镜像源下，相信一定比在官网下载快，比如阿里的镜像源（我登录上去发现说正在维护，所以选择了其他镜像源）

我主要下载的是centos7的Minimal版本：CentOS-7-x86_64-Minimal-2009.iso。



## 二、安装一台

首先，启动 VirtualBox 这个虚拟机软件，现在来新建一个虚拟机，点击“新建(N)”那个图标，会出现如下的窗口，在名称那一栏中填入你想给这个虚拟机起的名字，文件夹看个人选择，类型和版本可保持不变。

![图片](Virtualbox安装Centos7实战.assets/640.png)

点击“继续”，会进入如下窗口，是内存大小的配置，我这里保持不变：

![图片](Virtualbox安装Centos7实战.assets/640-16677430624793.png)

点击“继续”，会进入如下窗口，是虚拟硬盘的配置，默认选中了“现在创建虚拟硬盘”。点击“创建”即可。

![图片](Virtualbox安装Centos7实战.assets/640-16677430651456.png)

进入如下窗口，是虚拟硬盘文件类型的配置，默认选中的是“VDI（VirtualBox 磁盘映像）”，也就是以 .vdi 结尾的文件类型。

![图片](Virtualbox安装Centos7实战.assets/640-16677431056239.png)

点击“继续”，会进入如下窗口，是存储方式的配置，我们选择默认的“动态分配”方式即可，当然你如果希望用“固定大小”的方式也可以。用“动态分配”方式创建的虚拟硬盘会逐渐占用你的实际硬盘的大小，而“固定大小”的方式则是一次性占用指定的大小。

![图片](Virtualbox安装Centos7实战.assets/640-166774316293112.png)

点击“继续”，会进入如下窗口，是虚拟硬盘的文件位置和大小的配置，使用默认的配置即可。当然，你也可以修改，比如修改虚拟硬盘文件存储的路径和大小。

![图片](Virtualbox安装Centos7实战.assets/640-166774316616115.png)

点击“创建”，就创建好了这个新的虚拟机，可以在 VirtualBox 的管理器的左边栏中看到多了一个名叫 centos1 的虚拟机：

![图片](Virtualbox安装Centos7实战.assets/640-166774316981918.png)

选中 centos1 这个新建的虚拟机，点击“设置”：

![图片](Virtualbox安装Centos7实战.assets/640-166774321310024.png)

点击“存储”这个菜单，会出现如下图所示的界面，在“控制器：IDE”下面，选中“没有盘片”，然后点击右边的光盘图标，选择虚拟盘，选择刚才下载的ISO文件。

![图片](Virtualbox安装Centos7实战.assets/640-166774320935621.png)

点击OK即可：

![图片](Virtualbox安装Centos7实战.assets/640-166774322379227.png)

点击“OK”按钮之后，会回到 VirtualBox 管理器的界面。我们选中 centos1 这个虚拟机，然后点击“启动”：

![图片](Virtualbox安装Centos7实战.assets/640-166774322972730.png)

进入centos的安装界面，鼠标点进去，上下可以选中的时候，选中 install centos 7这一行，回车。

![图片](Virtualbox安装Centos7实战.assets/640-166774324682333.png)

如果你觉得窗口太小，可按照如下进行调整：

![图片](Virtualbox安装Centos7实战.assets/640-166774325098936.png)

> 鼠标点进去之后，要想出来回到宿主机，windows上只用crtl+shift。

之后会进入一个图形化界面，首先是选择语言，我这里就选择英语，选择完毕后点击右下角的继续按钮走到下一步。

![图片](Virtualbox安装Centos7实战.assets/640-166774326674239.png)

下一步有一些设置，比如下面安装目的地的设置：

![图片](Virtualbox安装Centos7实战.assets/640-166774327541142.png)

点进去选择左上角的done按钮即可，又回到刚才的页面，即可去下一步。（其他的也可自行设置，比如时区等，我这里篇幅限制，并且不重要的东西就不一一说明了）

![图片](Virtualbox安装Centos7实战.assets/640-166774333748845.png)

下一步是设置root密码和一个用户名密码，分别设置下即可：

![图片](Virtualbox安装Centos7实战.assets/640-166774334221848.png)

![图片](Virtualbox安装Centos7实战.assets/640-166774334596651.png)

![图片](Virtualbox安装Centos7实战.assets/640-166774334836454.png)

下面有一个进度条在动，默默等待其完成即可，经过一段时间等待后终于完成了：

![图片](Virtualbox安装Centos7实战.assets/640-166774336483357.png)

点击 reboot 即可，成功的话即可进入登录页面：

![图片](Virtualbox安装Centos7实战.assets/640-166774336868160.png)

输入刚才创建的用户名密码即可登录进去：

![图片](Virtualbox安装Centos7实战.assets/640-166774337182063.png)



## 三、再安装两台

好嘞，那我重复一遍以上过程。

**等等，不要冲动，有克隆功能！**

要复制之前，需要先关闭我们的虚拟机，可以在命令行输入 poweroff 命令，再按回车，来关机。当然，你也可以用图形界面的方式来操作关机。

![图片](Virtualbox安装Centos7实战.assets/640-166774341820566.png)

会出现如下的窗口，我们修改一下新虚拟电脑的名称，改为 centos2 。路径我改为了自己的路径，“MAC 地址设定”，将默认的选项改为“为所有网卡重新生成 MAC 地址”：

![图片](Virtualbox安装Centos7实战.assets/640-166774342152769.png)

就选择默认的“完整复制”即可，点击“复制”，就开始克隆了。

![图片](Virtualbox安装Centos7实战.assets/640-166774342384472.png)

克隆完之后，在 VirtualBox 管理器的左边栏就多出了一个虚拟机，名字是 centos2：

![图片](Virtualbox安装Centos7实战.assets/640-166774343883575.png)

用相同的操作来克隆出一个名叫 centos3 的虚拟机。现在，你已经准备好三个虚拟机（分别是 centos1，centos2，centos3）了，如下图所示：

![图片](Virtualbox安装Centos7实战.assets/640-166774344126978.png)



## 四、网络配置

### 4.1、方案确定

此时安装好之后，只有经过网络配置后才能进行内网和外网的访问。我们的目的是：虚拟机可以上外网，且主机与centos虚拟机也能连通，虚拟机之间也需要互相连通。

最好的办法就是使用两块网卡，nat(虚拟机访问互联网，使用10.0.2.x段)和host-only(虚拟机和主机互相通信，使用192.168.56.x段)。

为什么是192.168.56.x段呢？我们去看下就知道了，去`控制面板\网络和 Internet\网络连接`看下`virtualbox`网络：

![图片](Virtualbox安装Centos7实战.assets/640-166774348040481.png)

看下本地主机IP，在CMD工具输入ipconfig可查看 IPv4地址：

![图片](Virtualbox安装Centos7实战.assets/640-166774348450784.png)



### 4.2、网络配置

管理-》全局设定-》网络，添加网络：

![图片](Virtualbox安装Centos7实战.assets/640-166774350174487.png)

以centos2为例，点开设置，打开网络选项，设置两个网卡，分别是nat网络和host-only网络两种模式。

![图片](Virtualbox安装Centos7实战.assets/640-166774350484890.png)

![图片](Virtualbox安装Centos7实战.assets/640-166774350746793.png)

注意刷新下mac地址，以防冲突。下面启动centos2，进去设置。首先`ip addr`看下当前网络情况，发现网卡连对应的ip地址都没有，下面去设置。

![图片](Virtualbox安装Centos7实战.assets/640-166774350977096.png)

进入目录：`/etc/sysconfig/network-scripts/`：

![图片](Virtualbox安装Centos7实战.assets/640-166774351263499.png)

首先编辑文件：`vi ifcfg-enp0s3`网卡对应的信息：

![图片](Virtualbox安装Centos7实战.assets/640-1667743515025102.png)

把`ifcfg-enp0s3`文件拷贝下：`cp ifcfg-enp0s3 ifcfg-enp0s8`，编辑文件：`vi ifcfg-enp0s8`：

![图片](Virtualbox安装Centos7实战.assets/640-1667743530830105.png)

OK，大功告成，需要重启网络：

```
systemctl restart network
```

此时再次`ip addr`来查看网卡：

![图片](Virtualbox安装Centos7实战.assets/640-1667743538351108.png)

下面测试各种情况（我提前按照centos2的方式设置好了centos1主机，centos的ip设置为192.168.56.100）：

![图片](Virtualbox安装Centos7实战.assets/640-1667743541123111.png)

![图片](Virtualbox安装Centos7实战.assets/640-1667743545472114.png)

下面按照相同的方式设置centos即可，我这里三台虚拟的情况就是：

![图片](Virtualbox安装Centos7实战.assets/640-1667743549944117.png)



## 五、xshell去远程登录

因为ssh需要22端口，索性先关闭掉防火墙吧，centos7关闭防火墙：

![图片](Virtualbox安装Centos7实战.assets/640-1667743622645120.png)

下面就是打开工具`xshell`，新建会话，比如我这里要连接centos2，点击连接：

![图片](Virtualbox安装Centos7实战.assets/640-1667743626072123.png)

下面按照提示一路确定、输入用户名密码即可：

![图片](Virtualbox安装Centos7实战.assets/640-1667743628954126.png)

![图片](Virtualbox安装Centos7实战.assets/640-1667743631365129.png)

![图片](Virtualbox安装Centos7实战.assets/640-1667743633392132.png)

登录后我安装了`ifconfig`（可自行百度或谷歌哦），查看了下ip：

![图片](Virtualbox安装Centos7实战.assets/640-1667743636022135.png)

发现还是`xshell`来操作简单多了。那么本篇文章就要告一段落啦。



## 六、可能存在的问题

- **问题1：**virtualbox启动虚拟机报错Failed to open/create the internal network 'HostInterfaceNetworking-VirtualBox Host

在window系统中找到对应的虚拟网卡，禁用设备并重新启用即可。

![图片](Virtualbox安装Centos7实战.assets/640-1667743664863138.png)

- **问题2：**执行ping命令的时候，提示name or service not known

这是DNS服务器没有配置好，我们索性改为电信的114DNS服务器吧。

```
vi /etc/resolv.conf
```

修改为：

```
nameserver 114.114.114.114
```

- **问题3：**我还是不会配置网络，文字看的很烦

要不移步这里，这个几分钟的视频可参考：https://www.bilibili.com/s/video/BV1rT4y137vD

- **问题4：**我还是搞不定，出现了很多其他问题

**搞不定是常态，希望可以坚持不懈，一定可以的。**