# 参考文献 && 鸣谢

1. Win10 使用minikube搭建k8s集群 并运行hello-minikube实例：https://blog.csdn.net/fuyuande/article/details/118632723
1. CentOS 7安装minikube：https://www.cnblogs.com/harmful-chan/p/12731014.html
2. Centos7 使用Minikube搭建Kubernetes集群：https://www.cnblogs.com/cxt618/p/14985589.html
3. K8S学习之CentOS7环境安装minikube：https://blog.csdn.net/weixin_42586723/article/details/121895012
4. Kubeadm初始化遇到的问题：https://www.cnblogs.com/zwb-19981125/p/14372106.html、https://blog.csdn.net/weixin_38146924/article/details/117519139
5. 使用 minikube 安装指定版本的 k8s 集群：https://www.cnblogs.com/tssc/p/15119756.html
6. Kubernetes 环境搭建：https://www.cnblogs.com/cjsblog/p/11877014.html



# Minikube 搭建 K8S 步骤

```shell
# 国内阿里镜像v1.23.1：
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.23.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
curl -LO http://kubernetes.oss-cn-hangzhou.aliyuncs.com/kubernetes-release/release/v1.23.1/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/kubectl

# 国外官网镜像
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

# windows10 的启动方式
minikube start \
    --vm-driver=hyperv 
    --hyperv-virtual-switch="MinikubeSwitch" \
    --cpus=4 \
    --memory=4096mb \
    --image-mirror-country=cn \
    --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
    --registry-mirror="https://docker.mirrors.ustc.edu.cn" \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.7.3.iso

# Linux-CentOS7的启动方式
minikube start \
    --image-mirror-country=cn \
    --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
    --registry-mirror="https://docker.mirrors.ustc.edu.cn" \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.9.0.iso \
    --vm-driver=docker 
    --force

# 参数详解
# --registry-mirror：设置为自己的阿里云镜像加速器
# --iso-ur：指定阿里云做的国内iso镜像 官宣
# --image-repository 指定使用阿里云的镜像仓库

====================================================================
# 安装正常过得到步骤与版本
sudo yum -y install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io

# 步骤1 下载minikube
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.18.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
# 步骤2 下载kubectl
curl -LO https://kubernetes.oss-cn-hangzhou.aliyuncs.com/kubernetes-release/release/v1.19.0/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
# 步骤3：检查版本（Docker version 18.09.9、kubectl version v1.19.0、minikube version: v1.18.1）
docker -v && kubectl version --client && minikube version

# 步骤4：安装bash-completion命令补全以及 安装conntrack
yum -y install bash-completion && source /etc/profile.d/bash_completion.sh
yum install -y conntrack

# 步骤5：启动
minikube start --vm-driver=none --image-mirror-country=cn --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --force

# 步骤6：启动 dashboard 控制台，并映射访问端口
minikube dashboard
kubectl proxy --port=8001 --address='192.168.3.111' --accept-hosts='^.*' &
```



