Squid
---

### 参考资料
- https://www.digitalocean.com/community/tutorials/how-to-set-up-squid-proxy-on-ubuntu-20-04
- https://wiki.squid-cache.org/ConfigExamples/Authenticate/Ncsa

### 1. 安装
```commandline
sudo apt update
sudo apt install squid

# 查看squid服务验证启动成功
systemctl status squid
```

### 2. 配置
squid有很多的配置方式来保证访问的安全性，常用的是限制客户端的IP地址，但对于个人的动态IP地址或经常切换网络则不适用，所以我们使用用户名密码的方式做Basic Authentication来配置。

#### 2.1 安装密码生成工具`htpasswd`
```commandline
sudo apt install apache2-utils
sudo htpasswd -cbm /etc/squid/passwords your_squid_username your_squid_password
```
上面的命令会生成文件`/etc/squid/passwords`，里面保存了访问squid的用户名和编码后的密码, 关于`htpasswd`的具体用法，可以使用--help命令查看。

#### 2.2 修改配置文件`squid.conf`
```commandline
# 先备份原始配置文件
sudo cp /etc/squid/squid.conf /etc/squid/squid.config.ori

# 然后修改配置
sudo vim /etc/squid/squid.conf
```

##### 2.2.1 修改默认监听端口
```
http_port 8088
```

##### 2.2.2 添加认证配置
在`include /etc/squid/conf.d/*`这一行下面添加如下配置
```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/htpasswd
auth_param basic realm proxy
acl authenticated proxy_auth REQUIRED
http_access allow authenticated
```
#### 重启服务
```commandline
sudo systemctl restart squid
```

### 3. 验证
可以在本地电脑上使用curl命令进行验证
```commandline
curl -v -x http://your_squid_username:your_squid_password@your_server_ip:8088 http://www.baidu.com/
```
