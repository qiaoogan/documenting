在Debian11上安装配置Nginx服务做反向代理
--
Nginx是一个广泛使用的代理或者文件服务器，这里在Debian 11上原生安装配置（不使用容器）Nginx来做反向代理，目的是对外暴露服务器上面运行在K8S集群中的服务。

### 安装Nginx服务
```commandline
sudo apt update
sudo apt install nginx
```

#### 配置防火墙(可选）
> 默认情况下，nginx安装完成后，会自动打开HTTP端口，所以下面的步骤是可选的

Nginx服务一般用于托管外部接入的服务，所以通常都是监听HTTP的80端口和HTTPS的443端口，这就需要在防火墙中打开响应的端口。
```commandline
sudo ufw app list
```
使用上面的`ufw`命令会显示当前系统防火墙中配置了端口的服务，例如：
```commandline
Output
Available applications:
...
  Nginx Full
  Nginx HTTP
  Nginx HTTPs
  OpenSSH
...
```
其中Nginx Full表示同时使用80和443端口，Nginx HTTP表示仅开放80端口，Ngnix HTTPS表示仅开放443端口。开放端口的命令如下：
```commandline
sudo ufw allow 'Nginx HTTP'
```
然后可以查询端口情况：
```commandline
sudo ufw status

// 输出类似
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```
#### 查看Nginx服务状态
```commandline
systemctl status nginx

// 输出类似
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-02-27 20:22:47 CST; 12min ago
       Docs: man:nginx(8)
    Process: 3197685 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 3197686 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 3197776 (nginx)
      Tasks: 3 (limit: 4327)
     Memory: 8.7M
        CPU: 31ms
     CGroup: /system.slice/nginx.service
             ├─3197776 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─3197779 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             └─3197780 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
```
### 管理Nginx服务
管理Nginx服务的常用方式都是通过systemctl命令来完成的，比如：
```commandline
sudo systemctl start nginx     // 启动nginx服务
sudo systemctl stop nginx      // 停止ngin服务
sudo systemctl restart nginx   // 从新启动nginx服务
sudo systemctl reload nginx    // 更新配置后重新加载
sudo systemctl disable nginx   // 禁止在系统启动时自动启动nginx
sudo systemctl enable nginx    // 设置在系统启动时自动启动nginx
```
查看服务运行日志（一定要sudo）
```commandline
sudo journalctl -xeu nginx
```
Nigx服务的日志位于：
- `/var/log/nginx/access.log`, 访问日志
- `/var/log/nginx/error.log`, 错误日志

  
### 配置Nginx反向代理
Nginx服务安装完成后，配置文件默认都位于`/etc/nginx`目录下。
```commandline
cd /etc/nginx
```
备份原始配置文件
```commandline
sudo cp nginx.conf nginx.conf.ori
```

修改`nginx.conf`配置文件，在http区块（block）中添加如下内容：
```config
server {
	    listen 		80;
	    server_name		hw.dogger.instance localhost;

	    location / {
        	proxy_pass		http://k8s-master.ariman.com:31100;
		proxy_set_header	Host $host;
		proxy_set_header	X-Real-IP $remote_addr;
		proxy_set_header	X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header	X-Forwarded-Proto $scheme;
	    }

	    location /dashbackend/ {
	    	proxy_pass		http://k8s-master.ariman.com:31102;
	    }
}
```
其中，`hw.dogger.instance`是nginx服务所在服务器的公网域名，也可以是公网IP地址在本机上面设置的DNS域名，`http://k8s-master.ariman.com`则是我在/etc/hosts里面配置的本地服务器的私有IP地址的域名，因为K8S集群绑定在该私有IP地址上，31100和31102则是K8S服务所在的NodePort端口，可以被集群所在的服务器内部访问。

### 最后
反向代理只是Nginx的一种使用方式而已，更多的使用方式可以参加Nginx的官网，或者[这里](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-11)

