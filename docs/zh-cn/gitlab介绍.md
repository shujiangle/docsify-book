# GitLab 介绍


---

## 什么是Gitlab?
GitLab是一个开源分布式版本控制系统
开发语言： Ruby
功能： 管理项目源代码、版本控制、代码复用与查找

---
## GitLab与GitHub的不同
Github 分布式在线代码托管仓库，个人版本可直接在线免费使用，企业版本收费且需要服务器安装
Gitlab 分布式在线代码仓库托管软件，分社区免费版本与企业收费版本，都需要服务器安装

---
Gitlab的优势和应用场景
开源免费，适合中小型公司将代码放置在该系统中
差异化的版本管理，离线同步以及强大分支管理功能
便捷的GUI操作界面以及强大账号权限管理功能
集成度很高，能够集成绝大多数的开发工具
支持内置HA，保证在高并发下仍旧实现高可用性

---
GitLab主要服务构成
Nginx静态Web服务器
Gitlab-workhorse 轻量级的反向代理服务器
Gitlab-shell 用于处理Git命令和修改authorized kyes 列表
logrotate 日志文件管理工具
Postgresql 数据库
redis 缓存服务器

# GitLab的工作流程
创建并克隆项目
创建项目某Feature分支
编写代码并提交至该分支
推送该项目分支至远程Gitlab服务器
进行代码检查并提交Master主分支合并申请

# Gitlab安装配置管理
安装Omnibus Gitlab-ce package

### 1.安装Gitlab组件
```bash
yum -y install curl policycoreutils openssh-server openssh-clients postfix
```
#### 配置YUM仓库
```bash
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | bash
```

#### 启动postfix邮件服务
```bash
[root@centos-1 ~]# systemctl start postfix
[root@centos-1 ~]# systemctl enable postfix
```

#### 安装gitlab-ce 社区版本
```bash
yum install -y gitlab-ce
```

#### Gitlab等相关配置初始化并完成安装
1.证书创建与配置加载
```bash
[root@centos-1 ~]# mkdir -p /etc/gitlab/ssl
[root@centos-1 ~]# openssl genrsa -out "/etc/gitlab/ssl/gitlab.example.com.key" 2048
[root@centos-1 ~]# openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"

Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:bj
Locality Name (eg, city) [Default City]:bj
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:gitlab.example.com
Email Address []:admin@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123456
An optional company name []:

[root@centos-1 ~]# openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"
[root@centos-1 ~]# openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048
[root@centos-1 ~]# chmod 600 *  修改本地证书权限

# 修改/etc/gitlab/gitlab.rb
vim /etc/gitlab/gitlab.rb
external_url 'https://gitlab.example.com'
nginx['redirect_http_to_https'] = true
# nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
# nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
# nginx['ssl_dhparam'] = /etc/gitlab/ssl/dhparams.pem # Path to dhparams.pem


# 初始化配置
[root@centos-1 ssl]# gitlab-ctl reconfigure

# 编辑gitlab-http.conf配置文件
vim /var/opt/gitlab/nginx/conf/gitlab-http.conf
server_name gitlab.example.com;后面添加一行
rewrite ^(.*)$ https://$host$1 permanent;

#　使配置生效
[root@centos-1 ssl]# gitlab-ctl restart

#  gitlab.example.com 的密码是  woshishui

```

###　创建gitlab项目
![image_1dlf6b3u81hbq13vllg7joa4351c.png-147.4kB][1]



### windows下载git
`https://github.com/git-for-windows/git/releases/download/v2.23.0.windows.1/Git-2.23.0-64-bit.exe`

进入gitbash
```bash
shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop
$ pwd
/c/Users/shujiangle/Desktop

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop
$ mkdir repo

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop
$ cd repo

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo
$ git -c http.sslVerify=false clone https://gitlab.example.com/root/test-repo.git

$ pwd
/c/Users/shujiangle/Desktop/repo

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo
$ ls
test-repo/

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo
$ dir
test-repo

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo
$ cd test-repo/

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ ls

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ vi test.py

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ git add .
warning: LF will be replaced by CRLF in test.py.
The file will have its original line endings in your working directory

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ git commit -m "first commit"

*** Please tell me who you are.

Run

 git config --global user.email "you@example.com"
 git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'shujiangle@DESKTOP-796LNHH.(none)')

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ git config --global user.email "admin@example.com"

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ git config --global user.name "admin"

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ git commit -m "first commit"
[master (root-commit) dc97e7c] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 test.py

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
$ git -c http.sslverify=false push origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 228 bytes | 228.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://gitlab.example.com/root/test-repo.git
 * [new branch]      master -> master

shujiangle@DESKTOP-796LNHH MINGW64 ~/Desktop/repo/test-repo (master)
```
取消密码验证
```bash
git config --system --unset credential.helper
```

### 使用其他账号登录
```bash
$ git -c http.sslVerify=false clone https://gitlab.example.com/root/test-repo.git
$ cd test-repo/
# 创建一个分支
$ git checkout -b release-1.0
$ vim test.sh
hello world
this is dev test
$ git add .
$ git commit -m ""
```


github使用
```bash
shujiangle@DESKTOP-796LNHH MINGW64 /f/PYTHONSTUDY (master)
$ git add 20190927

shujiangle@DESKTOP-796LNHH MINGW64 /f/PYTHONSTUDY (master)
$ git commit -m "second commit"
[master 49212b1] second commit
 1 file changed, 27 deletions(-)
 delete mode 100644 20190927/venv/1.py

shujiangle@DESKTOP-796LNHH MINGW64 /f/PYTHONSTUDY (master)
$ git remote add origin git@github.com:shujiangle/python_study.git

```








最后效果
![image_1dlf8bctn1i55t5gumj1663ng21p.png-167.1kB][2]


  [1]: http://static.zybuluo.com/sjl--3306/em3igcmv3liqx6czz1rd49fg/image_1dlf6b3u81hbq13vllg7joa4351c.png
  [2]: http://static.zybuluo.com/sjl--3306/1ur3iv5t8f6rqo9dugwavh2k/image_1dlf8bctn1i55t5gumj1663ng21p.png

---

# Gitlab应用

 - Gitlab后台管理
 - 开发视角的Gitlab
 - 运维视角的Gitlab
 运维大部分时间都需要去后台获取相应信息，CPU利用率，内存磁盘使用率，系统的健康状况,保护
gitlab高并发的健康下，快速稳定的运转，也需要关注gitlab的权限管理
