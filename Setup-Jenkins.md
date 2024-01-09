Setup Jenkins
---

#### 在Jenkins服务器上克隆git代码
由于众所周知的原因，在国内的环境克隆github上面的代码会经常遇到下面的报错，原因是使用https进行`git clone`时超时。
```commandline
Failed to connect to github.com port 443 after 129479 ms: Connection timed out
```
目前最靠谱的解决方案是使用ssh替换https来拉取代码。

> 官方的操作步骤参加[官方文档](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

简单来说就两步，以jenkins的用户生成公钥，然后将公钥添加到github的个人账号的设置中去：

##### 1. 生成jenkins用户的公钥
```commandline
# 在服务器上切换成jenkins用户
sudo su - jenkins

# 生成公钥，这里的your_email@example.com要改为github的登录账号
ssh-keygen -t ed25519 -C "your_email@example.com"

# 获取公钥类容
cat ~/.ssh/id_ed25519.pub
```

##### 2. 将公钥添加到github的个人账号的设置里面
将上面得到的`id_ed25519.pub`的文件内容复制到github -> settings -> SSH and GPS keys里面即可，然后就可以使用ssh拉取代码，例如：
```commandline
git clone git@github.com:qiaoogan/fastaping.git
```
