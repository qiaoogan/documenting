Kubernetes - Setup : 2024

### 安装目标
1. 在纯净的Linux系统（Debian 12 或者Ubuntu 22）上安装K8s

### 参考资料
- https://acytoo.com/ladder/debian12-kubernetes-installation-and-config-master-worker/
- https://blog.csdn.net/qq_15045983/article/details/127678302
- [Linux系统镜像源](https://developer.aliyun.com/article/1146440)

### 1. 更新source.list镜像:
替换`/etc/apt/source.list`文件内容为以下国内源：

#### 1.a Debian 12
```commandline
# 阿里镜像源
deb https://mirrors.aliyun.com/debian bookworm main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian bookworm-updates main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian bookworm-proposed-updates main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian bookworm-backports main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian bookworm-backports-sloppy main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian bookworm main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian bookworm-updates main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian bookworm-proposed-updates main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian bookworm-backports main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian bookworm-backports-sloppy main non-free non-free-firmware contrib

# 阿里安全更新镜像源
deb https://mirrors.aliyun.com/debian-security bookworm-security main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian-security bookworm-security main non-free non-free-firmware contrib
```

#### 1.b Ubuntu 22
```commandline
# 阿里镜像站
deb https://mirrors.aliyun.com/ubuntu jammy main multiverse restricted universe
deb https://mirrors.aliyun.com/ubuntu jammy-backports main multiverse restricted universe
deb https://mirrors.aliyun.com/ubuntu jammy-proposed main multiverse restricted universe
deb https://mirrors.aliyun.com/ubuntu jammy-security main multiverse restricted universe
deb https://mirrors.aliyun.com/ubuntu jammy-updates main multiverse restricted universe
deb-src https://mirrors.aliyun.com/ubuntu jammy main multiverse restricted universe
deb-src https://mirrors.aliyun.com/ubuntu jammy-backports main multiverse restricted universe
deb-src https://mirrors.aliyun.com/ubuntu jammy-proposed main multiverse restricted universe
deb-src https://mirrors.aliyun.com/ubuntu jammy-security main multiverse restricted universe
deb-src https://mirrors.aliyun.com/ubuntu jammy-updates main multiverse restricted universe
```

### 2. 系统准备
#### 2.1 添加用户到sudo
```commandline
> apt-get install sudo
> sudo adduser $USERNAME sudo

# 如果sudo adduser报错，就切换成root执行下面的操作
> su
> /sbin/adduser ariman sudo
> /sbin/visudo
```
`visudoe`命令会打开一个配置文件，在其“%sudo   ALL=(ALL:ALL) ALL”下面添加“[username]   ALL=(ALL:ALL) ALL”，然后保存退出：Ctrl+X, y, Endter.

#### 2.2 更新安装源
```commandline
> sudo apt update && sudo apt upgrade -y
> sudo apt autoremove
> sudo apt autoclean
```

#### 2.3 安装常用工具
```commandline
> sudo apt-get install net-tools
```

#### 2.4 禁用Swap交换分区
```commandline
> free -h  # 查看swap的使用情况
> sudo swapoff -a
> sudo vim /etc/fstab  #注释掉swap的那一行
> sudo reboot

# 重启后验证swap的使用情况
> free -h
```
> swap的禁用可能不能持久化，相应的可以参考[How to Permanently Disable Swap in Linux](https://www.tecmint.com/disable-swap-partition-in-centos-ubuntu/)


#### 2.5 配置网络
```commandline
> cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

> sudo modprobe overlay
> sudo modprobe br_netfilter

> cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

> sudo sysctl --system
```

#### 2.6 配置域名
在`/etc/hosts`文件中添加`本地IP地址`的域名，域名在后面配置k8s集群时会用到，可以任意取名
```
xx.xx.xx.xx   k8s-master.ariman.com
```

### 3. 安装运行时 - docker & cri-dockerd
常用的K8s运行时（Runtime）有三种，`containerd`、`CRI-O`和`docker`，这里选择docker作为运行时来使用。当使用docker作为K8s的运行时时，除了要安装docker外，还需要安装`cri-dockerd`作为shim.
详细的安装介绍：[Get Docker Engine - Community for Debian \| Docker Documentation](https://docs.docker.com/install/linux/docker-ce/debian/)

#### 3.1 卸载非官方安装的docker版本
```commandline
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

#### 3.2 安装docker
##### 3.2.1.a debian上安装docker
```commandline
> sudo apt-get update
> sudo apt-get install ca-certificates curl gnupg apt-transport-https software-properties-common
> sudo install -m 0755 -d /etc/apt/keyrings
> curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
> sudo chmod a+r /etc/apt/keyrings/docker.gpg

> echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

> sudo apt-get update
> sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
##### 3.2.1.b ubuntu上安装docker
```commandline
> sudo apt-get update
> sudo apt-get install ca-certificates curl gnupg apt-transport-https software-properties-common
> sudo install -m 0755 -d /etc/apt/keyrings
> curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
> sudo chmod a+r /etc/apt/keyrings/docker.gpg

> echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

> sudo apt-get update
> sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
##### 3.2.2 修改docker配置：
- 设置cgroup driver 为systemd（kubernetes V1.22.2版本及其以后，要求容器的cgroup driver为systemd，但是docker默认的cgroup driver是cgroupfs），这一步非常重要；
- 配置镜像加速器，使用阿里云的[镜像服务器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)；
> 阿里云的镜像服务器是绑定到个人账号的，每个人的URI地址是不同的，需要注册一个阿里云账号
```commandline
> cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "registry-mirrors": ["https://zvkzilnb.mirror.aliyuncs.com"]
}
EOF
```
##### 3.2.3 重启docker并配置开机启动
```commandline
> sudo systemctl daemon-reload
> sudo systemctl restart docker
> sudo systemctl enable docker
```

#### 3.3 安装cri-dockerd
cri-dockerd的安装方式有两种：
- 使用cri-dockerd在github上发布的二进制安装包直接安装，这种方式会自动安装和配置开机服务，最方便；
- 直接使用源代码在Linux系统上编译安装；

#### 3.3.1.a debian上安装cri-dockerd
目前（2023-12-26）cri-dockerd v0.3.8版本的发布安装包里面，只有针对debian 11的安装包，没有debian 12的安装包，所以这里使用源码进行编译安装。

##### 3.3.1.a.1 安装go
编译安装cri-dockerd需要使用go，所以要先安装go
```commandline
> sudo apt install golang
```
使用apt安装的golang可能不是最新的版本，如果与cri-dockerd的需求版本不匹配会报错，目前cri-dockerd需要1.20版本的go，但apt安装的是1.19版本的go，所以需要再安装一次1.20版本的go
```commandline
> sudo apt install golang-1.20
```
> 安装完成后使用`go version`查询版本，如果还是1.19的版本，需要手动替换go的链接文件，详细信息参加debian的[官方文档](https://packages.debian.org/sid/golang-1.20)

##### 3.3.1.a.2 安装cri-dockerd并设置开机启动服务
```commandline
> git clone https://github.com/Mirantis/cri-dockerd.git
> cd cri-dockerd
> make cri-dockerd
> sudo mkdir -p /usr/local/bin
> sudo install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
> sudo install packaging/systemd/* /etc/systemd/system
> sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
> sudo systemctl daemon-reload
> sudo systemctl enable cri-docker.service
> sudo systemctl enable --now cri-docker.socket
```

#### 3.3.1.b ubuntu上安装cri-dockerd
目前cri-dockerd v0.3.8版本有发布适用于ubuntu 22的二进制安装包，所以直接使用安装包进行安装。
```commandline
> wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.8/cri-dockerd_0.3.8.3-0.ubuntu-jammy_amd64.deb
> sudo apt install ./cri-dockerd_0.3.8.3-0.ubuntu-jammy_amd64.deb
```

#### 查看cri-dockerd服务
```commandline
> systemctl status cri-docker
```
服务应该是正常`Running`。


### 4. 安装Kubenetes
K8s的安装主要就是安装`kubeadm`、`kubectl`和`kubelet`这三个工具

#### 4.1 安装`kubeadm`、`kubectl`和`kubelet`
```commandline
> sudo apt-get update
> sudo apt-get install -y apt-transport-https curl

# 安装证书
> curl -s http://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

# 配置源
> cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

# 更新源      
> sudo apt-get update

# 安装工具
> sudo apt-get install -y kubelet kubeadm kubectl

# 关闭这三个程序的自动更新
> sudo apt-mark hold kubelet kubeadm kubectl

# 验证
> kubeadm version
> kubectl version
> kubelet --version
```

#### 4.2 启动kubelet和设置开启服务
```commandline
> sudo systemctl daemon-reload
> sudo systemctl start kubelet
> sudo systemctl enable kubelet
```

#### 4.3 生成和修改初始化配置文件
上面的4.2完成之后，k8s的工具安装就完成了，接下来就是配置和初始化集群。集群的初始化使用`kubeadm init`命令，一般有两种方式：
- 使用命令参数进行初始化，例如`sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock --control-plane-endpoint master-node-01.acytoo.net`, 完整的初始化命令参数参考[官方文档](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)；
- 使用配置文件进行初始化，例如`sudo kubeadm init --config=init.yaml`, 使用配置文件方面追溯和修改配置，所以最好使用配置文件的方式来初始化；

##### 4.3.1 获取默认的配置文件
kubeadm可以生成默认的配置文件，然后再基于默认的配置文件修改我们需要的配置
```commandline
> kubeadm config print init-defaults > init.default.yaml
```

##### 4.3.2 修改配置参数
```commandline
# 使用默认文件复制出初始化文件
> cp init.default.yaml init.config.yaml
```
以下是修改后的init-config.yaml文件
```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 0.0.0.0     //修改1, 通常为master节点的IP地址，不能使用域名，使用0.0.0.0表示监听所有网络地址
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/cri-dockerd.sock //修改2，使用cri-dockerd套接字，而不是默认的containerd
  imagePullPolicy: IfNotPresent
  name: k8s-master.ariman.com    //修改3，work节点node的名称，因为是单节点集群，直接使用master的域名
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers    //修改4，更换成阿里云的镜像仓库
kind: ClusterConfiguration
kubernetesVersion: 1.28.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 192.168.0.0/16           //修改5，添加podSubnet，也可以不添加，因为192.168.0.0/16是其默认值
scheduler: {}
controlPlaneEndpoint: k8s-master.ariman.com:6443   //修改6，添加control-plane-endpoint，使用域名，在/etc/hosts配置
```
> 上述类容中的注释仅仅是说明，在保存修改的时候需要去掉，因为yaml文件不允许添加注释。

对于`advertiseAddress`, 如果是生产环境或者有固定IP地址的服务器，可以使用固定的IP地址，但如果是个人电脑的测试环境，可能经常更换Wifi网络，则可以使用`0.0.0.0`，表示监听任意的IP地址。

##### 4.3.3 拉取镜像文件
使用`kubeadm init`初始化集群时，需要拉取必要的镜像文件，但最好是在初始化之前先手动拉取镜像文件，避免在初始化的时候出错。
```commandline
> kubeadm config images pull --config=init.config.yaml
```
> 拉取镜像的时候，如果出现`nework is unreachable`之类的问题，可以单独使用`docker image pull`命令测试看能不能正常拉取镜像，如果也报错同样的问题，就需要单独解决，这就不是k8s的问题，而是docker自己的网络问题，比如dns配置不当，参见[这里](https://blog.csdn.net/yukai08008/article/details/121512212)

##### 4.3.4 初始化集群
```commandline
> sudo kubeadm init --config=init.config.yaml
```

##### 4.3.5 重置集群（可选）
在初始化的过程中如果出错，可以重置集群，然后重新初始化
```commandline
# reset
> sudo kubeadm reset --cri-socket unix:///var/run/cri-dockerd.sock
> sudo rm -rf /etc/kubernetes
> sudo rm ~/.kube/config

# re-init
> sudo kubeadm init --config=init.config.yaml
```

复制配置文件到普通用户的目录下（必须，不然会报错找不到apierver)
```commandline
> mkdir -p $HOME/.kube
> sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $HOME/.kube/config
> export KUBECONFIG=$HOME/.kube/config
```

可以验证kubectl命令了
```commandline
kubectl get nodes
```

#### 4.4 安装CNI网络插件（网络插件安装之前，Master node的状态会是NotReady）
至此，k8s集群的基本安装已经完成，但还需要安装CNI网络插件，CNI网络插件有多种不同的插件，这里使用`calio`：
> 下面的命令中使用的是v3.27.0版本，这是目前的最新版本。安装时具体的版本和命令可以参考calio的[官方文档](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-50-nodes-or-less)，其中，calio有`operator`和`manifest`两种安装方式，自己玩儿的话，使用manifest就足够了。
```commandline
> wget https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
> kubectl apply -f calico.yaml
```
calio构建需要一些时间，可以使用watch命令来持续查看资源的状态：
```commandline
watch kubectl get pods -A
```
当所以pod的状态都变成Running时，如下所示，K8s集群master节点的安装就完成了。
```
NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5fc7d6cf67-vwq5g        1/1     Running   0          6m55s
kube-system   calico-node-47zjf                               1/1     Running   0          6m55s
kube-system   coredns-66f779496c-6tbwp                        1/1     Running   0          11m
kube-system   coredns-66f779496c-kggkl                        1/1     Running   0          11m
kube-system   etcd-k8s-master.ariman.com                      1/1     Running   0          12m
kube-system   kube-apiserver-k8s-master.ariman.com            1/1     Running   0          12m
kube-system   kube-controller-manager-k8s-master.ariman.com   1/1     Running   0          12m
kube-system   kube-proxy-vcjxq                                1/1     Running   0          11m
kube-system   kube-scheduler-k8s-master.ariman.com            1/1     Running   0          12m
```

### 5. 异常处理
不出问题就不是K8S！！！！

#### IP地址变更
当使用个人电脑进行测试安装时，开关机或者切换Wifi网络都可能造成本机IP地址发生变更，虽然我们在执行`kubeadm init`命令时使用了`/etc/hosts`里面配置的域名来作为control-plane-endpoint，但kubeadm在初始化时，还是会将其域名解析成对应的IP地址（也就是`/etc/hosts`里面配置的IP地址）来使用，这就导致更改IP地址后，kubeclt无法找到kube-apiserver，从而出现类似下面的报错：
```
The connection to xxx.xxx.xxx.xxx:6443 was refused - did you specify right host or port?
```
当出现这样的问题时，可以首先验证集群中使用的IP地址是否确实是老的地址，查看下面的配置文件:
```commandline
> su
> cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

这个文件记录了kube-apiserver的具体配置信息，它是由`kubeadm init`命令生成的。然而，单独更改kube-apiserver.yaml文件里面的IP地址是不能修复这个问题的，因为apiserver已经运行并且在监听老的IP地址，所以永远无法工作，唯一的办法只能是重置整个集群：
```commandline
> sudo kubeadm reset --cri-socket unix:///var/run/cri-dockerd.sock
> sudo rm -rf /etc/kubernetes/
> rm ~/.kube/config

# 重新初始化集群
> sudo kubeadm init --config=init.config.yaml
```

#### kubeadm init失败
在初始化集群时，如果链接的网络不能访问外网，那么十有八九会出现下面的错误：
```
Unfortunately, an error has occurred:
timed out waiting for the condition

This error is likely caused by:

    The kubelet is not running
    The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
    ‘systemctl status kubelet’
    ‘journalctl -xeu kubelet’
    
Additionally, a control plane component may have crashed or exited when started by the container runtime.
To troubleshoot, list all containers using your preferred container runtimes CLI, e.g. docker.
Here is one example how you may list all Kubernetes containers running in docker:

    ‘docker ps -a | grep kube | grep -v pause’
Once you have found the failing container, you can inspect its logs with:
    ‘docker logs CONTAINERID’
error execution phase wait-control-plane: couldn’t initialize a Kubernetes cluster
```

具体的错误原因使用下面的命令查看：
```commandline
# 一定要sudo，否则日志是空的
> sudo journalctl -xeu kubelet
```

如果日志里面包含`failed pulling image \"k8s.gcr.io/pause:x.x\"`，就说明在拉取`pause:x.x`镜像时出错。原因在于k8s.gcr.io这个仓库在国内几乎是访问不到的。虽然我们在之前的init.config.yaml文件中使用了阿里云的镜像仓库，但这个pause镜像比较特殊，会单独使用k8s.gcr.io这个仓库，解决这个问题的方法如下：

先手动从阿里云的镜像仓库中拉取相同版本的pause镜像：
```commandline
> docker image pull registry.aliyuncs.com/google_containers/pause:x.x
```

然后给它打一个k8s.gcr.io的tag：
```commandline
> docker tag registry.aliyuncs.com/google_containers/pause:x.x k8s.gcr.io/pause:x.x
```
这样kubeadm在拉取时就会首先使用本地的`k8s.gcr.io/pause:x.x`镜像，而不会去访问外网了。

