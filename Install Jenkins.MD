Install Jenkins
--

Jenkins的[官方网站](https://www.jenkins.io/doc/book/installing/)提供了多种安装Jenkins的方法，基于Docker和Kubernete的方式虽然方便，但安装出来的Jenkins执行机是位于容器内部的，和宿主机的交互不是那么丝滑。

由于设备有限，我的测试目的是在一台安装了docker和k8s集群（单节点）的Ubuntu服务器上安装Jenkins，然后由Jenkins直接使用docker和kubectl进行自动化操作，所以直接采用apt的方式安装Jenkins效果最直接，以下为具体的安装步骤。

### 安装Jenkins
> Jenkins在Linux系统上的直接安装方式受限于当时的Jenkins版本，需要配套的Java版本也是不一样的，所以下面的安装步骤仅限于目前2024年1月的安装方式，不同时期的安装方式，需要查阅[官方文档](https://www.jenkins.io/doc/book/installing/linux/)

#### 安装Java
```commandline
sudo apt update
sudo apt install fontconfig openjdk-17-jre

# 验证java
java -version
openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
```

#### 安装Jenkins
```commandline
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
  
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
  
sudo apt-get update
sudo apt-get install jenkins
```

上述安装会以systemd的方式将Jenkins作为服务安装在Linux系统中，所以可以通过下面的命令管理Jenkins服务
```commandline
# 查看Jenkins服务状态
systemctl status jenkins.service

# 关闭Jenkins服务
systemctl stop jenkins.service

# 重启Jenkins服务
systemctl restart jenkins.service

# 查看Jenkins日志
sudo journalctl -u jenkins.service
```

### 配置jenkins用户使用docker
上面说过，我用来部署Jenkins发服务器上已经安装有docker和kubectl，而要让Jenkins在build过程中能够直接使用docker和kubectl，还需要给jenkins的用户配置权限，否则会Permission Denied。

```commandline
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### 配置jenkins用户使用kubectl
```commandline
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chmod 600 /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

# 这一步貌似会报错，可以忽略
sudo usermod -aG jenkins /var/lib/jenkins/.kube/config

sudo systemctl restart jenkins
```

之后，在Jenkins的build shell里面可以直接使用docker和kubectl命令了。
