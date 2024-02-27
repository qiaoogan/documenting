在Debian11上安装配置Nginx服务
--

### 安装Nginx服务
```commandline
sudo apt update
sudo apt install nginx
```

#### 配置防火墙
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

### 配置Nginx服务
