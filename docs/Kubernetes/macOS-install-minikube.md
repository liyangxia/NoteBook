# macOS安装minikube以及优化排障

## minikube简单介绍

[minikube](https://minikube.sigs.k8s.io)是一个轻量化的本地k8s环境，非常方便作为开发、学习使用，丰富的addons几乎可以完整模拟出生产环节需要的容器集群环境，并且安装起来也非常的方便，而且得益于hyperkit驱动，在macOS上安装minikube甚至不需要庞大的Docker Desktop作为支持，所有的组建都可以使用homebrew安装使用

## 前置准备

1. 良好的网络环境，默认从国外网站下载镜像
2. 良好的设备性能，minikube运行起来后还是比较占用CPU和内存的
3. [Homebrew](https://brew.sh)包管理工具，启用[homebrew/core](https://github.com/Homebrew/homebrew-core)

## 开始安装

### 安装kubectl

```bash
brew install kubectl
```

### 安装hyperkit

如果本地有Docker Desktop环境的话可以不用安装hyperkit，但是需要在设置里关闭**启用Kubernetes**，避免和minikube冲突

```bash
brew install hyperkit
```

### 安装minikube

```bash
brew install minikube
```

一般正确执行后安装的minikube是可用的，如果遇到找不到minikube程序的错误，可以手动设置连接

```bash
brew unlink minikube
brew link minikube
```

## 调整配置

### 指定虚拟驱动

因为我的机器上只有hyperkit这一套虚拟化组件，为了指定使用hyperkit可以在启动minikube前进行调整

```bash
# 启动时指定hyperkit
minikube start --driver=hyperkit

# 设置默认启动
minikube config set driver hyperkit
```
### 添加必要环境变量

minikube会识别到默认的代理环境变量，可以提前进行设置，方便进行拉取镜像

```bash
# 全局代理配置
export http_proxy=*****
export https_proxy=*****

# Minikube 配置
export NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.99.0/24,192.168.39.0/24,192.168.64.0/24
# 下面这条是禁止emoji，可以体验下后给加上
export MINIKUBE_IN_STYLE=false
```

### 附加选择

前置条件提到minikube还是比较吃CPU和内存的，如果资源不足最好的办法是事先指定虚拟机环境的配置，减少本机资源的使用

```bash
# 设置CPU个数
minikube config set cpu 2

# 设置内存大小
minikube config set memory 16384MB

# 设置磁盘大小（默认是20G，但是对于正式的开发环境，镜像大小很容易超出限定大小，因为还不清楚是否有动态扩容机制，一劳永逸的方法是在创建虚拟机前调整到合适的空间大小）
minikube config set disk-size 50000MB
```

## 开始使用

配置完成后即可进行启动，执行`minikube start`即可启动虚拟机，完整的启动日志如下（事先拉取过镜像）：

```bash
>minikube start
* minikube v1.21.0 on Darwin 10.15.7
  - MINIKUBE_IN_STYLE=false
* Using the hyperkit driver based on user configuration
* Starting control plane node minikube in cluster minikube
* Creating hyperkit VM (CPUs=2, Memory=16384MB, Disk=50000MB) ...
* Found network options:
  - NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.99.0/24,192.168.39.0/24,192.168.64.0/24
  - http_proxy=http://159.75.114.229:10011
  - https_proxy=http://159.75.114.229:10011
# 下面这个错误不重要，我自己的网关代理配置有问题
! This VM is having trouble accessing https://k8s.gcr.io
* To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
* Preparing Kubernetes v1.20.7 on Docker 20.10.6 ...
  - env NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.99.0/24,192.168.39.0/24,192.168.64.0/24
  - env HTTP_PROXY=http://159.75.114.229:10011
  - env HTTPS_PROXY=http://159.75.114.229:10011
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

检查nodes状态

```bash
kubectl get nodes

# 示例输出
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   3m54s   v1.20.7
```

检查pods状态

```bash
kubectl get pods -A

# 示例输出
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-wjlph            1/1     Running   0          3m51s
kube-system   etcd-minikube                      1/1     Running   0          4m5s
kube-system   kube-apiserver-minikube            1/1     Running   0          4m5s
kube-system   kube-controller-manager-minikube   1/1     Running   0          4m5s
kube-system   kube-proxy-tlk6z                   1/1     Running   0          3m51s
kube-system   kube-scheduler-minikube            1/1     Running   0          4m5s
kube-system   storage-provisioner                1/1     Running   1          4m4s
```

打开dashboard

```bash
# 命令执行后会自动打开dashboard页面
minikube dashboard

# 如果只是需要链接而不是自动打开页面
minikube dashboard --url

# 示例输出
* Enabling dashboard ...
  - Using image kubernetesui/dashboard:v2.1.0
  - Using image kubernetesui/metrics-scraper:v1.0.4
* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
* Opening http://127.0.0.1:52999/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

## 结语

因为日常都是在大型服务器集群上工作，对k8s的也有了很深刻的了解，自己也用树莓派搭建过mini的k8s集群，但是真正的在自己的设备上以开发、学习为目的安装k8s环境还是第一次，挺新鲜有趣的，况且居然还不用使用完整的Docker功能，仅仅需要hyperkit就可以了，这简直就是我的幸运

不过缺点倒也是挺明显的，minikube运行起来后还是比较占用CPU和内存的，不适合长久的开启，否则电池会变的很不耐用，在需要使用的时候再开启，或者转向其他方案应该才是最终解
