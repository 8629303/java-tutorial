# Liunx 环境启动、停止、重启脚本

1、单一启动脚本

```bash
vi start.sh
```

```shell
#!/bin/bash
echo starting
java -jar joke.jar > log.file 2>&1 &
```

2、停止服务脚本：停止脚本的流程就是，通过ps查找到该项目进程id，然后kill掉。

```bash
vi stop.sh
```

```shell
#!/bin/bash
   PID=$(ps -ef | grep joke-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{ print $2 }')
 
if [ -z "$PID" ]
then
    echo Application is already stopped
else
    echo kill $PID
    kill $PID
fi
```

3、重启脚本：重启脚本流程好理解，依次调用停止，启动即可。

```bash
vi restart.sh
```

```shell
#!/bin/bash
echo stop application
source stop.sh
echo start application
source start.sh
```

4、启动 & 重启脚本：

```shell
#!/bin/sh
# Jar包名称，注意：不要带有.jar
APP1_NAME=sdk-cms-1.0.1-beta
 
# 启动服务前，如果服务依然在运行，找到服务的PID
# grep -v 只打印没有匹配的，而匹配的反而不打印
tpid1=`ps -ef|grep $APP1_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
 
# 如果服务存在，就杀掉
if [ $tpid1 ]; then
kill -9 $tpid1
fi
 
# 指定配置文件，启动Jar，控制台默认输出到nohup.out文件
nohup java -jar $APP1_NAME.jar --spring.profiles.active=pro &
```



# Windows 环境启动、停止、重启脚本

1、启动脚本：start.bat（如下2个任选其一即可）：

```bash
# 后台启动
@echo off
START "app" javaw -jar app-0.0.1-SNAPSHOT.jar
```

```bash
# 前台启动
@echo off
START "app" java -jar app-0.0.1-SNAPSHOT.jar
pause
```

2、停止脚本：shutdown.bat

```bash
@echo off
# 项目启动后，会占用的端口
set port=8080
for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%port%"') do (
    echo kill the process %%m who use the port %port%
    # 根据 进程id pid 信息，杀掉进程
    taskkill /f /pid %%m
)
```

3、重启脚本：restart.bat

```bash
@echo off
call ./shutdown.bat
call ./start.bat
```

如果不在同一文件夹可以使用如下脚本：

```bash
@echo off
start "call shutdown..." cmd /k call D:\shutdown.bat
start "call start..." cmd /k call D:\start.bat
```



# Windows JAR包启动停止和日志分割脚本

> Windows平台下JAR包的启动,停止和日志分割脚本：https://glory.blog.csdn.net/article/details/110004829

## 1、JAR 包的启动脚本

### 1、JAR包的启动脚本

启动脚本：start.bat

```bash
@echo off
echo.
echo WebVR START...
echo.


:: 进入目录文件夹
d:
cd D:\JT\java\workspace\webvr-end\deploy


:: 执行多个javaw命令，但是无法输出日志
echo  webvr-auth START...
start javaw -jar -Dfile.encoding=utf-8 -Xms800m -Xmx1024m  webvr-auth-1.0.5.jar

echo  webvr-gateway START...
start javaw -jar -Dfile.encoding=utf-8 -Xms800m -Xmx1024m  webvr-gateway-1.0.5.jar

echo  webvr-system START...
start javaw -jar -Dfile.encoding=utf-8 -Xms800m -Xmx1024m  webvr-system-1.0.5.jar

echo  webvr-vr START...
start javaw -jar -Dfile.encoding=utf-8 -Xms800m -Xmx1024m  webvr-vr-1.0.5.jar

pause
```

**存在问题：**无法输出日志



### 2、单个JAR包启动并输出日志

注意：这里依赖cronolog组件

启动脚本：start.bat

```bash
@echo off
echo.
echo WebVR START...[1/4]
echo.

:: 应用存放路径
set deploy_path = D:\JT\java\workspace\webvr-end\deploy

:: 进入目录文件夹
d:
echo cd deploy_path %deploy_path%
cd %deploy_path%


:: 执行命令
echo  webvr-auth START...
javaw -D'file.encoding=utf-8' -jar webvr-auth-1.0.5.jar 2>&1>webvr-auth.log

pause
```

> **存在问题：**日志文件越来越大,问题定位不方便

### 3、单个JAR包启动并输出按天分割的日志

启动脚本：start.bat

```bash
@echo off
echo.
echo WebVR START...[4/4]
echo.

:: 应用存放路径
set deploy_path = D:\JT\java\workspace\webvr-end\deploy

:: 进入目录文件夹
d:
echo cd deploy_path %deploy_path%
cd %deploy_path%


:: 执行命令
echo  webvr-vr START...
:: javaw -D'file.encoding=utf-8' -jar webvr-vr-1.0.5.jar 2>&1>webvr-vr.log
javaw -D'file.encoding=utf-8' -jar webvr-vr-1.0.5.jar | D:\JT\java\workspace\webvr-end\deploy\cronolog\cronolog.exe D:\JT\java\workspace\webvr-end\deploy\log\webvr-vr-%%Y%%m%%d.log

pause
```

**还待解决的问题：**过去多少天（如3个月）的日志，并没有多大意义，需要清理



## 2、JAR 包的停止脚本

原理:

1. 查找一下XXX端口的进程：netstat -ano | findstr :9200
2. 结束进程: taskkill /pid 2044 -t -f

注意：执行停止脚本会报错。

- **其中报错信息"错误,没有找到进程XXX"，并不影响，因为对应的进程已经再上面被杀掉**



### 1、停止一个端口

停止脚本：shutdown.bat

```bash
@echo off
set port=20812
for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%port%"') do (
    echo kill the process %%m who use the port %port%
    taskkill /pid %%m
)
```



### 2、停止多个端口

停止脚本：shutdown.bat（注意：port 设置成变量，并用空白字符分割）

```bash
@echo off
set port=7910 9200 9201 9203
for /f "tokens=1-5" %%a in ('netstat -ano^|findstr ":%port%"') do (
    echo PID:%%e is running,kill the process [%%e] who use the port [%%b]
    taskkill /F /pid %%e
)
pause
```



### 3、停止多个连续端口

停止脚本：shutdown.bat

```bash
@echo off
for /l %%n in (20801,1,2085) do (
    @echo find the process which use port [%%n]
    for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%%n"') do (
        tasklist /FI "PID eq %%m"|find /i "PID" && (
        echo PID:%%m 运行中,kill the process [%%m] who use the port [%%n]
        taskkill /F /pid %%m
        ) || echo PID:%%m 未运行 
    )
)
```

> for /l %%n in (20801,1,2085)，表示操作：20801,20802,20803,20804,20805 的端口



## 3、批处理脚本静默运行

1、打开文本编辑器

```vbscript
DIM objShell
set objShell=createobject("wscript.shell")
result=objShell.Run("bat路径",0,true)
```

2、将文件的后缀名改为.vbs 双击即可运行

```vbscript
DIM objShell
set objShell=createobject("wscript.shell")
result=objShell.Run("D:\JT\java\workspace\webvr-end\deploy\StartWebVR.bat",0,true)
```

