> - gitlab+runner持续集成springboot项目之docker模式【3】https://www.cnblogs.com/hukey/p/17183582.html
> - [Docker部署gitlab-runner并使用Docker命令构建Docker镜像](https://zhuanlan.zhihu.com/p/597132392)

# 一、介绍

前面的文章已经介绍了如何以 shell的方式来进行gitlab的持续集成，gitlab的CI/CD功能需要通过gitlab-runner来完成，本文采用docker来部署gitlab-runner。

总的来说想要在CI/CD中使用Docker命令，你可以通过如下三种方式进行：

- [The shell executor](https://link.zhihu.com/?target=https%3A//docs.gitlab.com/ee/ci/docker/using_docker_build.html%23use-the-shell-executor)
- [Docker-in-Docker](https://link.zhihu.com/?target=https%3A//docs.gitlab.com/ee/ci/docker/using_docker_build.html%23use-docker-in-docker)
- [Docker socket binding](https://link.zhihu.com/?target=https%3A//docs.gitlab.com/ee/ci/docker/using_docker_build.html%23use-docker-socket-binding)

**第一种，**通过shell execotor可以不必将你的gitlab-runner以docker的方式运行，同时，需要你将gitlab-runner加入到docker组，这样gitlab-runner就可以执行docker的命令了。

**第二种**，Docker in Docker的方式需要docker container获取privilege的权限。他的含义是①executor的类型是docker②runner通过docker提供的镜像来执行docker命令。dockers镜像需要拥有所有的docker工具，并且以privilege的模式在镜像的上下文中执行docker命令。

在这种模式下官方推荐需要使用TLS，查看：[GitLab.com shared runners](https://link.zhihu.com/?target=https%3A//docs.gitlab.com/ee/ci/runners/index.html).

**第三种**，可以不用priviledge的方式，就是Docker socket binding的方式。这种方式是通过将宿主机/var/run/docker.sock挂载到容器的/var/run/docker.sock上，这样，gitlab-runner中执行的docker命令最终会在宿主机上的docker daemon中执行。

本文采用 **第三种** 方式来介绍。



# 二、docker 运行 runner

## 1、启动 runner

通过docker的方式运行 runner

```bash
docker run -itd --restart=always  \
  --name gitlab-runner \
  -v /etc/gitlab-runner:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock  \
  gitlab/gitlab-runner:alpine
```

在上面的命令中， 我们进行了两处挂载：

```bash
-v /etc/gitlab-runner:/etc/gitlab-runner：这个挂载是将gitlab-runner的配置文件挂载到宿主机上，这样我们可以通过修改宿主机上的这个配置文件对gitlab-runner进行配置

-v /var/run/docker.sock:/var/run/docker.sock:这个挂载是将宿主机上的docker socket挂载到了容器内，这样容器内执行的docker命令会被宿主机docker daemon最终执行。
```



## 2、注册 runner

当容器启动成功后，需要将这个runner注册到gitlab的CI/CD上，runner有几种类型，这里采用全局注册的方式。

![image-20230306112736350](https://img2023.cnblogs.com/blog/828019/202303/828019-20230306134947076-1682532652.png)

左侧有俩个重要的信息，一个是URL，就是gitlab的url，另一个是token，我们获取到这两个信息后，通过以下命令进行注册：

```bash
docker exec -it gitlab-runner  gitlab-runner register -n \
  --url http://192.168.199.104 \	### 注册gitlab地址
  --registration-token Hz1igGoTfo9YeERHk3iM \	###注册tokan
  --tag-list=dockersock,docker \
  --description "dockersock" \
  --docker-privileged=true \
  --docker-pull-policy="if-not-present" \
  --docker-image "docker:20.10.7" \	###最好和本地运行的docker版本同步
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
  --docker-volumes /root/m2:/root/.m2 \		### 共享maven缓存多次使用
  --executor docker
```

操作完成后，在你的gitlab项目中就会出现这个runner了：

![image-20230306113010974](https://img2023.cnblogs.com/blog/828019/202303/828019-20230306134946724-744681746.png)



# 三、编写配置文件

通过docker构建，需要编写两个配置文件，如下：

- Dockerfile
- .gitlab-ci.yml

## 1、Dockerfile

Dockerfile 根据项目自行进行编写，下面是我本项目使用的一个Dockerfile

```dockerfile
FROM openjdk:8-jdk-alpine
COPY target/system-monitoring-1.0-release/system-monitoring-1.0 /app
CMD ["java","-jar","/app/lib/system-monitoring-1.0.jar","--spring.config.location=/app/config/application.yml"]
EXPOSE 8105
```



## 2、gitlab-ci.yml

```yaml
image: docker:20.10.7
stages:
  - compile
  - build
variables:
  MAVEN_OPTS: "-Dmaven.repo.local=/root/.m2"
compile:
  image: maven:3.6.2-jdk-8
  stage: compile
  tags:
    - docker
  script:
    - mvn clean package -Dmaven.test.skip=true
  artifacts:
    expire_in: 1 days
    paths:
      - target
  only:
    - main
build:
  stage: build
  tags:
    - docker
  script:
    - docker build -t test ./
    - docker rm -f monitor
    - docker run --name monitor -d -p 8105:8105 test
  only:
    - main
```

配置文件详解：

```yaml
image: docker:20.10.7
说明：Runner执行器设置为docker, 所以这里需要指定docker的版本

stages:
  - compile
  - build
说明：定义了两个阶段：compile和build

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=/root/.m2"
说明：定义个变量, 指定maven下载的jar包存放的位置，方便重复利用，不用多次重复下载


compile:
  image: maven:3.6.2-jdk-8
  stage: compile
说明：第一阶段，打包用到了maven, 所有需要拉取maven镜像。stage：指定阶段

  tags:
    - docker
说明：拥有标签为 docker 的runner执行，没有则不执行

  script:
    - mvn clean package --Dmaven.test.skip=true
说明：执行的打包命令

  artifacts:
    expire_in: 1 days
    paths:
      - target
说明：编译后有产物,所以要指定下过期时间和路径, 以供于其他阶段使用

  only:
    - main
说明：指定仅在 main 分支上运行

build:
  stage: build
  tags:
    - docker
说明：第二阶段及标签

  script:
    - docker build -t test ./
    - docker rm -f monitor
    - docker run --name monitor -d -p 8105:8105 test
说明：执行容器的更替

  only:
    - main
说明：指定仅在 main 分支上运行
```



# 四、触发 CI/CD

## 1、提交 git

```bash
git add .
git commit -m 'test ci/cd'
git push origin main
```



## 2、流水线触发

![image-20230306114906946](https://img2023.cnblogs.com/blog/828019/202303/828019-20230306134946354-2133075104.png)



## 3、查看服务部署

```bash
root@localhost(192.168.199.105)~/system-monitoring>ds
CONTAINER ID   IMAGE                         COMMAND                  CREATED       STATUS       PORTS                                       NAMES
9d773cd4213c   test                          "java -jar /app/lib/…"   2 hours ago   Up 2 hours   0.0.0.0:8105->8105/tcp, :::8105->8105/tcp   monitor
3358767da9e1   gitlab/gitlab-runner:alpine   "/usr/bin/dumb-init …"   2 hours ago   Up 2 hours                                               gitlab-runner

```



## 4、浏览器访问测试

![image-20230306115014711](https://img2023.cnblogs.com/blog/828019/202303/828019-20230306134945888-393629093.png)



## 5、版本迭代

这里模拟修改代码内容进行测试是否能够做到自动部署启动

```bash
sed -i 's/hello china!/hello beijing!/g' ./src/main/java/com/jiusuo/kj/data/system/service/impl/SystemMonitoringServiceImpl.java
git add .
git commit -m 'test'
git push origin main
```

等待1-2分钟后，刷新浏览器查看：

![image-20230306134524062](https://img2023.cnblogs.com/blog/828019/202303/828019-20230306134945173-1048748954.png)



# 五、问题总汇

## 1、gitlab-runner 拉取代码错误

![image](https://img2023.cnblogs.com/blog/828019/202305/828019-20230511165820967-2050483652.png)

**上面的问题是因为 注册 runner的用户未添加到该项目中。根据我自己理解，当gitlab多个仓库都需要做CI/CD时，可以创建一个gitlab-runner用户，然后在每个项目中添加该用户作为Reporter，然后通过 gitlab-runner 用户中添加 runner，这样所有的项目CI/CD都可以拉取到，就不会出现如上的报错信息。**





# 六、docker-compose 启动 Gitlab 与 Runner

```yaml
version: '3.8'  # 使用较新的 compose 版本，支持更多特性
services:
  gitlab:
    image: 'gitlab/gitlab-ce:16.8.0-ce.0'  # 固定版本，避免自动更新
    restart: unless-stopped  # 除手动停止外均重启（比 always 更合理）
    container_name: gitlab
    hostname: '192.168.2.132'  # 移除端口，仅保留主机IP/域名
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.2.132:8001'  # 外部访问地址（含端口）
        gitlab_rails['gitlab_shell_ssh_port'] = 2222  # SSH端口映射
        # 性能优化：根据CPU核心数调整（示例：2核CPU设为3）
        puma['worker_processes'] = 3
        # 数据库连接池优化（默认较小，可根据并发需求调整）
        postgresql['max_connections'] = 100
        # 关闭未使用的服务（如不需要可以禁用）
        # prometheus['enable'] = false
        # grafana['enable'] = false
    ports:
      - '8001:80'  # HTTP端口映射
      # - '8443:443'  # 如暂不使用HTTPS，可注释掉
      - '2222:22'  # SSH端口映射
    volumes:
      - /etc/localtime:/etc/localtime:ro  # 只读挂载，增强安全性
      - ./config:/etc/gitlab
      - ./data:/var/opt/gitlab
      - ./logs:/var/log/gitlab
    # 资源限制：根据服务器配置调整（示例：4核8G服务器）
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
        reservations:
          cpus: '2'
          memory: 4G
    # 健康检查：监控服务是否就绪
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]  # 检查内部80端口
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 5m  # GitLab启动慢，预留足够时间

  gitlab-runner:
    image: gitlab/gitlab-runner:v16.8.0  # 与GitLab版本匹配
    restart: unless-stopped
    hostname: gitlab-runner
    container_name: gitlab-runner
    extra_hosts:
      - git.imlcs.top:192.168.2.132  # 若内部使用该域名，保留
    depends_on:
      - gitlab
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./runner:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock  # 允许Runner调用主机Docker
    # 资源限制：Runner资源需求较低
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 1G
```

执行`docker-compose up -d`运行容器。访问`http://192.168.2.132:8001` 并设置好密码。

> **注意：目录权限预处理**：启动前执行以下命令，避免容器内权限错误：
>
> ```bash
> sudo chmod -R 777 ./config ./data ./logs ./runner  # 生产环境可细化权限（如指定用户组）
> ```

配置 GitLab Runner：

- 方式一：

```bash
docker exec -it gitlab-runner  gitlab-runner register -n \
  --url http://192.168.2.132:8001/ \
  --registration-token WE6zAWHJniXzWDxq2qPU \
  --description "dockersock" \
  --docker-privileged=true \
  --docker-pull-policy="if-not-present" \
  --docker-image "docker:latest" \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
  --docker-volumes /root/m2:/root/.m2 \
  --executor docker
```

- 方式二：

```bash
[root@localhost gitlab]# docker exec -it gitlab-runner gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=17 revision=a998cacd version=13.2.2
Running in system-mode.
# 输入GitLab地址
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.2.132:8001/
# 输入GitLab Token
Please enter the gitlab-ci token for this runner:
WE6zAWHJniXzWDxq2qPU
# 输入 Runner 的说明(可以为空,以后可修改)
Please enter the gitlab-ci description for this runner:
[gitlab-runner]:
# 设置 Tag 可以用于指定在构建规定的 tag 时触发 ci(可后期修改)
Please enter the gitlab-ci tags for this runner (comma separated):
docker,maven
# 选择 runner 执行器，这里我们选择的是 docker
Please enter the executor: shell, docker+machine, docker-ssh+machine, kubernetes, custom, docker-ssh, parallels, ssh, virtualbox, docker:
docker
Please enter the default Docker image (e.g. ruby:2.6):
docker:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

