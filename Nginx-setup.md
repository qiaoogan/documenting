在Debian11上安装配置Nginx服务
--

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

### 配置Nginx服务

