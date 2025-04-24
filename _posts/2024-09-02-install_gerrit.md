---
layout: post
title: "Centos 7上安装gerrit 2.14.22"
date:   2024-09-02
tags: [web]
comments: true
author: weijun23
---

```shell
[gerrit@ ~]# cat /etc/redhat-release
       CentOS Linux release 7.6.1810 (Core)
```
## 一、安装JDK
```shell
[gerrit@ ~]# sudo yum search jdk                        #查看CenOS的yum源支持的openjdk版本
[gerrit@ ~]# sudo yum install java-1.8.0-openjdk-devel

[gerrit@ ~]# java -version                              #安装后，他默认就会把新新安装的版本链接过去。
       openjdk version "1.8.0_412"
       OpenJDK Runtime Environment (build 1.8.0_412-b08)
       OpenJDK 64-Bit Server VM (build 25.412-b08, mixed mode)
[gerrit@ ~]# rpm -qa *openjdk*                          #查看已经存在的JDK版本
       java-1.8.0-openjdk-devel-1.8.0.412.b08-1.el7_9.x86_64
       java-1.8.0-openjdk-headless-1.8.0.412.b08-1.el7_9.x86_64
       java-1.8.0-openjdk-1.8.0.412.b08-1.el7_9.x86_64
```
## 二、安装Git
```shell
[gerrit@ ~]# sudo yum install gitweb           #安装gitweb服务
[gerrit@ ~]# sudo yum install epel-release     #安装第三方软件源 REPL，因为里面存有git-review的安装包
[gerrit@ ~]# sudo yum install git-review       #安装git-review，我们会用到git review命令，使用它可以轻松把本地代码push到greeit服务器上去
[gerrit@ ~]#                                   #git默认安装的版本是1.8.3，它和gerrit集成并不台友好，因此不强烈不推荐大家使用yum方式安装，建议大家使用源码安装git
[gerrit@ ~]# sudo yum remove git               #如果你已经安装了git，建议卸载掉，或者用rpm命令进行卸载
[gerrit@ ~]# sudo yum install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm #安装新版本 Git 最快的方法是通过 End Point 库。
[gerrit@ ~]# sudo yum install git
```
## 三.安装Apache服务并启动
```shell
[gerrit@ ~]# sudo yum install httpd
[gerrit@ ~]# sudo systemctl enable httpd
[gerrit@ ~]# sudo systemctl restart httpd
[gerrit@ ~]# rpm -qa | grep httpd
       httpd-tools-2.4.6-99.el7.centos.1.x86_64
       httpd-2.4.6-99.el7.centos.1.x86_64
```
## 四、安装mysql(切记安装5.6版本，5.7会连接出错)
```shell
[gerrit@ ~]# rpm -qa | grep mariadb; rpm -qa | grep mysql                                    #找到的，删除
[gerrit@ ~]# sudo yum install http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm #安装 mysql yum 源
[gerrit@ ~]# sudo yum install mysql-server                                                   #安装 mysql

[gerrit@ ~]# sudo cp -rp /var/lib/mysql ~/  #把mysql数据库移到home目录下,以后数据都放到这
[gerrit@ ~]# sudo vim /etc/my.cnf           #下面一段修改配置
       #datadir=/var/lib/mysql
       datadir=/home/gerrit/mysql
       bind-address = 127.0.0.1
       port=3306
       character-set-server=utf8
       default-storage-engine=innodb

[gerrit@ ~]# sudo systemctl restart mysqld.service

[gerrit@ ~]# mysql -uroot -p                #第一次空密码
       mysql> set password for 'root'@'localhost'=password('123456');
       mysql> FLUSH PRIVILEGES;
       mysql> CREATE DATABASE reviewdb;
       mysql> show databases;

[gerrit@ ~]# mysql --version
       mysql  Ver 14.14 Distrib 5.6.51, for Linux (x86_64) using  EditLine wrapper
```
## 五、安装gerrit (下面“<=========”的要注意修改,别的用默认,直接回车)
```shell
[gerrit@ ~]# wget https://gerrit-releases.storage.googleapis.com/gerrit-2.14.22.war
[gerrit@ ~]# java -jar gerrit-2.14.22.war init -d review
   Using secure store: com.google.gerrit.server.securestore.DefaultSecureStore
   [2024-08-31 22:31:24,219] [main] INFO  com.google.gerrit.server.config.GerritServerConfigProvider : No /home/gerrit/review/etc/gerrit.config; assuming defaults
   
   *** Gerrit Code Review 2.14.22
   ***
   
   Create '/home/gerrit/review'   [Y/n]?
   
   *** Git Repositories
   ***
   
   Location of Git repositories   [git]: /home/gerrit/git         # <========= 指定Git存储库
   
   *** SQL Database
   ***
   
   Database server type           [h2]: mysql                     # <========= 指定mysql数据库
   
   Gerrit Code Review is not shipped with MySQL Connector/J 5.1.41
   **  This library is required for your configuration. **
   Download and install it now [Y/n]?
   Downloading https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.41/mysql-connector-java-5.1.41.jar ...
    OK
   Checksum mysql-connector-java-5.1.41.jar OK
   Server hostname                [localhost]: 
   Server port                    [(mysql default)]:
   Database name                  [reviewdb]:
   Database username              [gerrit]: root                   # <========= mysql用户名
   root's password                :                                # <========= mysql密码
                 confirm password :                                # <========= mysql密码
   
   *** Index
   ***
   
   Type                           [lucene/?]:
   
   *** User Authentication
   ***
   
   Authentication method          [openid/?]: HTTP                 # <========= 认证方法输入HTTP,我们要使用反向代理
   Get username from custom HTTP header [y/N]?
   SSO logout URL                 :
   Enable signed push support     [y/N]? y                         # <========= 启用签名的推送支持
   
   *** Review Labels
   ***
   
   Install Verified label         [y/N]?
   
   *** Email Delivery
   ***
   
   SMTP server hostname           [localhost]: smtp.exmail.qq.com  # <========= 输入自动发送邮件的smtp服务器
   SMTP server port               [(default)]: 465                 # <========= 465/994时SSL协议端口后，25是非SSL协议端口号
   SMTP encryption                [none/?]: SSL                    # <========= 上一步输入的是465/994,此处输入SSL
   SMTP username                  [gerrit]: admin@nationalchip.com # <========= 发送邮件的邮箱地址
   wuwj@nationalchip.com's password :                              # <========= 输入2次咱们邮箱的授权码，而非邮箱密码
                 confirm password :                                # <========= 输入2次咱们邮箱的授权码，而非邮箱密码
   
   *** Container Process
   ***
   
   Run as                         [gerrit]:
   Java runtime                   [/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.412.b08-1.el7_9.x86_64/jre]:
   Copy gerrit-2.14.22.war to review/bin/gerrit.war [Y/n]?
   Copying gerrit-2.14.22.war to review/bin/gerrit.war
   
   *** SSH Daemon
   ***
   
   Listen on address              [*]:
   Listen on port                 [29418]:
   Generating SSH host key ... rsa... dsa... ed25519... ecdsa 256... ecdsa 384... ecdsa 521... done
   
   *** HTTP Daemon
   ***
    
   Behind reverse proxy           [y/N]? y                                    # <=========  使用发向代理
   Proxy uses SSL (https://)      [y/N]?
   Subdirectory on proxy server   [/]:
   Listen on address              [*]: localhost                              # <=========  
   Listen on port                 [8081]:
   Canonical URL                  [http://localhost/]: http://192.168.190.229 # <========= 规范url，退出gerrit后跳转的url
   
   *** Cache
   ***
   
   
   *** Plugins
   ***
   
   Installing plugins.                                                        # <========= 一路”y“ 即可
   Install plugin commit-message-length-validator version v2.14.22 [y/N]? y
   Installed commit-message-length-validator v2.14.22
   Install plugin download-commands version v2.14.22 [y/N]? y
   Installed download-commands v2.14.22
   Install plugin hooks version v2.14.22 [y/N]? y
   Installed hooks v2.14.22
   Install plugin replication version v2.14.22 [y/N]? y
   Installed replication v2.14.22
   Install plugin reviewnotes version v2.14.22 [y/N]? y
   Installed reviewnotes v2.14.22
   Install plugin singleusergroup version v2.14.22 [y/N]? y
   Installed singleusergroup v2.14.22
   Initializing plugins.
   
   Initialized /home/gerrit/review
   Executing /home/gerrit/review/bin/gerrit.sh start
   Starting Gerrit Code Review: WARNING: Could not adjust Gerrit's process for the kernel's out-of-memory killer.
            This may be caused by /home/gerrit/review/bin/gerrit.sh not being run as root.
            Consider changing the OOM score adjustment manually for Gerrit's PID=3137 with e.g.:
            echo '-1000' | sudo tee /proc/3137/oom_score_adj
   OK
   Waiting for server on 192.168.190.229:80 ... OK
   Opening http://192.168.190.229/#/admin/projects/ ...FAILED
   Open Gerrit with a JavaScript capable browser:
     http://192.168.190.229/#/admin/projects/
```
## 六、修改gerrit配置，canonicalWebUrl是退出gerrit登录后跳转的网址
```shell
[gerrit@ ~]# sudo ./review/bin/gerrit.sh restart   # 重启gerrit
[gerrit@ ~]# cat review/etc/gerrit.config
   [gerrit]
           basePath = /home/gerrit/git
           serverId = b911a634-f6a4-4c5d-baf2-8d05749ee640
           canonicalWebUrl = http://192.168.190.229
   [database]
           type = mysql
           hostname = localhost
           database = reviewdb
           username = root
   [index]
           type = LUCENE
   [auth]
           type = HTTP
   [receive]
           enableSignedPush = true
   [sendemail] #发送到本机/var/mail/gerrit文件里，不外发
           smtpServer = localhost
           smtpPort = 25
           from = gerrit@localhost.localdomain
           sendEmail = true
   [container]
           user = gerrit
           javaHome = /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.412.b08-1.el7_9.x86_64/jre
   [sshd]
           listenAddress = *:29418
   [httpd]
           listenUrl = proxy-http://localhost:8081/
   [cache]
           directory = cache
```
## 七、创建passwd文件，添加gerrit登录用户, 修改Apache的config文件httpd.conf
```shell
[gerrit@ ~]# sudo systemctl restart httpd                        #重启appache
[gerrit@ ~]# sudo htpasswd -cb /etc/httpd/passwords admin admin  #注意，"-c"参数为创建，仅限第一次添加用户时使用，用户名和密码均为admin
[gerrit@ ~]# sudo htpasswd -b /etc/httpd/passwords  wuwj wuwj    #第二次创建时不要加"-c"参数。创建一个wuwj用户，密码为"wuwj"
[gerrit@ ~]# cat /etc/httpd/passwords                            #我们查看该文件的确有2个用户，但是密码时经过加密处理的！
   admin:$apr1$Em..sIno$GMpAsQlJ4l.S4C0Qnu0iD/
   wuwj:$apr1$892G6DfX$IBkKm8paM/4wPvowppI/d1
[gerrit@ ~]# sudo vim /etc/httpd/conf/httpd.conf                 #修改Apache的config文件httpd.conf,尾部添加
   Listen 80                                                     #自定义反响代理的端口
   <VirtualHost *:80>                                            #我们只针对咱们自定义的端口进行配置
       ServerName 192.168.190.229                                #此处为你的服务器IP地址或者主机名均可，比如”192.168.190.229“
       ProxyRequests Off
       ProxyVia Off
       ProxyPreserveHost On
       <Proxy *>
           Require all granted
       </Proxy>
       <Location /login/>
           AuthType Basic
           AuthName "Gerrit Code Review"
           Require valid-user
           AuthUserFile /etc/httpd/passwords                     #我们让认证方式是基于文件方式认证
       </Location>
       AllowEncodedSlashes On
       ProxyPass / http://127.0.0.1:8081/ nocanon                #Gerrit服务的端口号。nocanon参数可以创建git多层次目录
   </VirtualHost>
```
## 八、 在网页http://192.168.190.229 将SSH密钥添加到您的Gerrit帐户,  通过Gerrit的Web UI创建相应的存储库Repositories
```shell
           # 在客户端利用git将代码push到Gerrit服务器上, 获取空项目
[gerrit@ ~]# git clone ssh://admin@192.168.190.229:29418/abc

           # 更新主分支,origin 是远程仓库的默认名称。通常在克隆仓库时，Git 会将远程仓库命名为 origin。你可以通过 git remote -v 命令查看所有远程仓库的列表
[gerrit@ ~]# git pull origin master

           # 将本地代码push到geerit服务器中
[gerrit@ ~]# git push origin HEAD:refs/for/master
[gerrit@ ~]# gitdir=$(git rev-parse --git-dir); scp -p -P 29418 admin@192.168.190.229:hooks/commit-msg ${gitdir}/hooks/
[gerrit@ ~]# git commit --amend
[gerrit@ ~]# git push origin HEAD:refs/for/master
```
## 九 、搭建repo的manifests仓库
### 1、修改Apache的config文件httpd.conf,尾部添加8000端口
```shell
[gerrit@ ~]# sudo vim /etc/httpd/conf/httpd.conf
    Listen 8000
    <VirtualHost *:8000>
        ServerName 192.168.190.229
        DocumentRoot /home/gerrit/git
        <Directory /home/gerrit/git>
            Options +Indexes +FollowSymLinks
            AllowOverride None
            Require all granted
        </Directory>
        ScriptAlias /git/ /usr/libexec/git-core/git-http-backend/
        <Location /git>
            AuthType None
            Require all granted
            SetEnv GIT_PROJECT_ROOT /home/gerrit/git
            SetEnv GIT_HTTP_EXPORT_ALL
            SetEnv REMOTE_USER=$REDIRECT_REMOTE_USER
        </Location>
        ErrorLog /var/log/httpd/git-error.log
        CustomLog /var/log/httpd/git-access.log common
    </VirtualHost>
```
### 2、 让apache可以访问gerrit创建的目录(很关键)
```shell
[gerrit@ ~]# sudo usermod -aG gerrit apache
```
### 3、 通过Gerrit的Web UI创建相应的存储库Repositories创建manifests仓库
```shell
[gerrit@ ~]# git clone ssh://wuwj@192.168.190.229:29418/mk/manifests #不要提交到HEAD:refs/for/master，会进入审核系统
[gerrit@ ~]# repo init -u http://192.168.190.229:8000/git/mk/manifests.git -b api -m gx6605s_ecos_api.xml
```
这样就可以repo使用了

### 4、 其他：

重启gerrit方式
```shell
 ~/review/bin/gerrit.sh stop;java -jar gerrit-2.14.22.war reindex -d review;sudo ~/review/bin/gerrit.sh restart
```
 安装postfix，没保存注册邮箱，提交不了代码
```shell
sudo yum install postfix
sudo vi /etc/postfix/main.cf
    myhostname = localhost.localdomain                                # 主机名用本机
    mydestination = localhost, localhost.localdomain, 192.168.190.229 # 允许的邮件来源
sudo systemctl start postfix
sudo systemctl enable postfix

[gerrit@localhost ~]$ sudo tail -f /var/log/maillog #查看发送邮件日志
[gerrit@localhost ~]$ postqueue -p                  #查看发送队列
[gerrit@localhost ~]$ sudo postcat -q ECB8A4270D50  #查看邮件，要带上邮件ID
[gerrit@localhost ~]$ sudo postsuper -d ALL         #清空发送队列
```
查看邮件
```shell
cat /var/mail/gerrit
sudo echo "" > cat /var/mail/gerrit #清空
```
### 5、 QA：怎么添加用户？
```shell
sudo htpasswd -b /etc/httpd/passwords  wuwj wuwj
cat /var/mail/gerrit                #注册邮箱
sudo echo "" > cat /var/mail/gerrit #清空
```
