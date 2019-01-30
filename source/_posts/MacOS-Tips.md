---
title: MacOS_Tips
date: 2017-09-10 18:07:17
categories: tips
type: "tags"
tags:
  - 小技能
---

### MacOS/Linux下的小知识
---

#### Tip.1 更新Locate的数据库  

```
$ sudo /usr/libexec/locate.updatedb    #MacOS
# 如果依然报权限问题，先执行以下内容
$ launchctl load -wF /System/Library/LaunchDaemons/com.apple.locate.plist
$ launchctl stop com.apple.locate && launchctl start com.apple.locate

$ sudo updatedb    #Linux
```

#### Tip.2 当使用源文件编译安装，configure默认--prefix=/Applications/Xcode.app/**下面时，make失败大部分因为没有安装command line tools造成的

```
$ xcode-select --install    #MocOS
$ softwareupdate --list
$ softwareupdate --install <product name>   #安装product
$ softwareupdate --install -a   #升级all CommandLineTools
```

<!-- more -->

#### Tip.3 MacOX修改HostName  

```
$ sudo scutil --set HostName yourhostname       #MacOS
$ hostname      #查看输出是否生效
```

#### Tip.4 MocOX启动ssh服务

修改/etc/ssh/sshd_config, 打开RSAAuthentication & PubkeyAuthentication & AuthorizedKeysFile等项

~/.ssh中生成rsa key-pair, 然后启动ssh服务

```
$ sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

#### Tip.5 MocOX zsh环境变量加载顺序

特别注意，/etc/zprofile中只是调用了/usr/libexec/path_helper -s, 而path_helper会生成主要的PATH部分，path_helper主要工作是读取/etc/paths以及/etc/paths.d/目录下的文件的内容以生成PATH环境变量，如果需要修改顺序，则需要修改这两项的内容。

Global Order: etc/ ——> ~/. : zshenv, zprofile, zshrc, zlogin

![zsh环境变量加载顺序](zsh-env-order.jpeg)

#### Tip.6 Linux下docker非sudo下运行：将用户加入docker工作组  

```
$ sudo usermod -aG docker <username>			#Linux
```

#### Tip.7 chmod命令改变文件权限是有限制的,只对linux分区有效,如果是挂在目录,则命令失效

#### Tip.8 git vi报错：原因是vim版本的问题，并且git支持的是vi，更换为最高版本的vim即可正常  

```
$ git config --global core.editor vim
```

#### Tip.9 查看Ubuntu版本号

```
$ sudo lsb_release -a
```

#### Tip.10 更改时区

```
$ sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ sudo hwclock -w
```

#### Tip.11 更新pip

```
$ sudo python -m pip install --upgrade --force pip
# 用python则安装python2的pip，用python3则安装pip3
$ python/python3 -m pip install -U --force-reinstall pip
$ sudo pip install setuptools==33.1.1
```

#### Tip.12 经常出现的路径权限问题

```
$ sudo chown -R `whoami` /to/your/path
```

#### Tip.13 Python's Error -- TypeError: __cal__() takes exactly 2 arguments (1 given)

```
$ sudo vim ~*/Python/2.7/site-packages/packaging/requirements.py
```

And

```
MARKER_EXPR = originalTextFor(MARKER_EXPR())("marker")
```

Change it to

```
MARKER_EXPR = originalTextFor(MARKER_EXPR(""))("marker")
```

#### Tip.14 Mac安装软件出现"应用已损坏"的解决办法

```
$ sudo spctl --master-disable
# 完事之后再切回来
$ sudo spctl --master-enable
```

#### Tip.15 查看Cpu相关

```
# 查看物理CPU个数
$ cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
# 查看每个物理CPU中core的个数(即核数)
$ cat /proc/cpuinfo| grep "cpu cores"| uniq
# 查看逻辑CPU的个数
$ cat /proc/cpuinfo| grep "processor"| wc -l
```

#### Tip.16 MySql的my.cnf的加载顺序

```
# 以下命令可以查看加载顺序
$ mysqld --verbose --help | grep -A 1 'Default options'
```

文件名 | 目的
---|---
/etc/my.cnf | 全局选项
/etc/mysql/my.cnf | 全局选项
SYSCONFDIR/my.cnf | 全局选项
$MYSQL_HOME/my.cnf | 服务器特定选项（仅限服务器）
defaults-extra-file | 指定的文件 --defaults-extra-file，如果有的话
~/.my.cnf | 用户特定选项
~/.mylogin.cnf | 用户特定的登录路径选项（仅限客户端）

一般顺序如下:

- /etc/my.cnf
- /etc/mysql/my.cnf
- SYSCONFDIR/my.cnf  
  basedir/my.cnf
- $MYSQL_HOME/my.cnf   
  datadir/my.cnf
- 加启动指定参数  
  -defaults-file=#， 只读取指定的文件(不再读取其他配置文件)  
  -defaults-extra-file=#，从其他优先级更高的配置文件中读取全局配置后，再读取指定的配置文件  
- ~/my.cnf

#### Tip.17 缺少ps命令应该安装的工具类:

```
$ sudo apt-get udpate && sudo apt-get install procps
```

#### Tip.18 docker在中国区pull遇到的网络问题：

![设置docker的翻墙代理](docker-cross-wall.png)

#### Tip.19 查看网络带宽

```
$ nload
$ dstat -nf
```

#### Tip.20 传输大文件需网络经过跳板机，ssh无法直连时

```
# 使用nc传输，接收端
$ nc -k -l ${dest_ip} ${listen_port} > ${pwd}/filename
# nc发送端
$ nc ${dest_ip} ${dest_listen_port} < /path/${send_filename}
```

监听、发送端均可对传输文件加密

安装qpress参考qpress官网：[qpress](http://www.quicklz.com//)

qpress在linux直接安装，qpress在mac下需要自己编译，并需要修改如下代码：

```
--- a/qpress.cpp
+++ b/qpress.cpp
@@ -89,6 +89,7 @@
 #include <sys/stat.h>
 #include <stdlib.h>
 #include <stdio.h>
+#include <unistd.h>
 #include "aio.hpp"
 #include <stdarg.h>
 #include <string>
```

然后make编译即可使用

参考网站：[Fix_qpress_bug](http://blog.liudongkai.com/2017/02/11/qpress_error_isatty/)

nc传输压缩文件流：

```
# -k表示当client断开发送时server接收不断开
$ nc -k -l ${dest_ip} ${listen_port} | qpress -dio > ${pwd}/filename
$ qpress -o /path/${send_filename} | nc ${dest_ip} ${dest_listen_port}
```

#### Tip.21 添加用户并分配用户权限

```
$ groupadd [-r] -g 20999 xijia
$ useradd [-r] -c "xijia" -m -g xijia -u 20999 -s /bin/bash xijia
## 更改/etc/ssh/sshd_config中的PasswordAuthentication yes
$ vim /etc/ssh/sshd_config
$ service sshd restart
## 设置密码
$ passwd xijia
## 添加sudo权限
## 添加xijia ALL=(ALL) NOPASSWD: ALL到/etc/sudoers最后一行
$ vim /etc/sudoers
## 添加xijia所属组
$ usermod -aG root xijia

## 登陆服务器
$ ssh -l xijia -p 22 10.11.11.10
```

#### Tip.22 python升级版本后yum失效

```
## 将#!/usr/bin/python 修改为 #!/usr/bin/python2.6
$ vim /usr/bin/yum
$ vim /usr/bin/yum-config-manager
```

#### Tip.23 设置代理

可以修改.bashrc或者/etc/profile

```
export http_proxy=http://username:password@proxyserver:port/
export https_proxy=http://username:password@proxyserver:port/
export ftp_proxy=http://username:password@proxyserver:port/
# export http_proxy=http://10.93.1.44:8080
# export https_proxy=http://10.93.1.44:8080
```

yum可以修改/etc/yum.conf

```
proxy=http://username:password@proxyserver:port
```

#### Tip.24 最新版MacOS防火墙开启允许任何

```
$ sudo spctl --master-disable
```

#### Tip.25 scp传文件

```
# 本地文件传到远端服务器
$ scp -P 33 ${local_file} root@10.10.10.10:/data0/paladintyrion/${to_remote_file}
# 远程文件复制到本地:${remote_file}复制到本地/data0目录下
$ scp –P 34 root@10.10.10.10:/data0/paladintyrion/${remote_file} /data0
```

#### Tip.26 Linux设置docker代理

```
$ mkdir -p /etc/systemd/system/docker.service.d/ && touch /etc/systemd/system/docker.service.d/http-proxy.conf
```

在http-proxy.conf中添加如下内容：

```
[Service]
Environment="HTTP_PROXY=http://10.39.1.44:8080/"
```

then重启docker服务：

```
$ sudo systemctl daemon-reload
$ sudo systemctl show --property Environment docker
$ sudo systemctl restart docker
```

#### Tip.27 Linux永久修改hostname

```
# 另外把{hostname}放到/etc/hostname文件中
$ hostnamectl set-hostname paladintyrion
```

#### Tip.28 IntelliJ HTTP_PROXY setting

设置项: Preferences -> Appearance and Behavior -> System Settings -> HTTP Proxy

#### Tip.29 git每次需要密码的解决

```
$ git config --global credential.helper store
```

#### Tip.30 IntelliJ解决两个JVM环境的问题

```
# help->edit custom properties
idea.no.launcher=true
```

#### Tip.31 Linux反选文件删除

```
$ shopt -s extglob  ## 打开extglob模式
$ rm -fr !(*.bak)  ## 删除非*.bak的文件
```

#### Tip.32 批量重命名

```
$ for i in `ls`; do mv -f $i `echo $i | sed 's/\.bak$//'`; done
```

#### Tip.33 CentOS懒人安装py

```
$ yum install epel-release -y
$ yum install https://centos7.iuscommunity.org/ius-release.rpm -y
$ yum install python36u -y
$ ln -s /bin/python3.6 /bin/python3
$ yum install python36u-pip -y
$ ln -s /bin/pip3.6 /bin/pip3
```

#### Tip.34 百度云命令上传/下载

```
$ pip3 install bypy --user
$ bypy info

$ bypy list                #显示文档
$ bypy upload filename -v  #上传某文件，显示进度
$ bypy -c                  #取消令牌文件。一段时间后要重新授权
$ bypy downdir filename    #下载
$ bypy compare             #比较本地目录和网盘目录
```

#### Tip.35 pip升级所有包

```
$ pip3 freeze --local | grep -v '^\-e' | cut -d = -f 1  | xargs pip3 install -U
```

#### Tip.36 修复pip丢失

```
$ curl https://bootstrap.pypa.io/get-pip.py | sudo python
```

#### Tip.37 安装本地python包项目

```
$ pip3 install git+https://github.com/xxxxx/xxxx.git@master
# or
$ python3 setup.py install
```

### 参考文献

- [python3安装第三方包](https://www.jianshu.com/p/9acc85d0ff16)
