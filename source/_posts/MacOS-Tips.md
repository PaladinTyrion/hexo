---
title: MacOS_Tips
date: 2017-09-10 18:07:17
categories: tips
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
$ hostnamectl set-hostname <newhostname>        #Linux
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
# export http_proxy=http://10.39.1.44:8080
# export https_proxy=http://10.39.1.44:8080
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
$ curl https://bootstrap.pypa.io/get-pip.py | sudo python3
```

#### Tip.37 安装本地python包项目

```
$ pip3 install git+https://github.com/xxxxx/xxxx.git@master
# or
$ python3 setup.py install
```

#### Tip.38 限制允许sshd连接的ip

```
$ vim /etc/hosts.allow
sshd:10.10.10.*:allow

$ vim /etc/hosts.deny
sshd:11.11.11.*:deny

$ service sshd restart
```

至于经常sshd无法启动，一般会发现是因为默认22端口已被bind，sshd_config配置一个新的连接端口即可。

#### Tip.39 Centos设置防火墙80/8080端口的权限

```
$ firewall-cmd --zone=public --add-port=80/tcp[--add-port=8080/tcp] --permanent
$ systemctl stop firewalld.service && systemctl start firewalld.service (or $ firewall-cmd --reload)
## 删除端口
$ firewall-cmd --zone=public --remove-port=80/tcp --permanent && firewall-cmd --reload
## 设置部分ip允许权限
$ firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.88.0.0/16[10.88.88.0/24]' [protocol='tcp'] accept" && firewall-cmd --reload
## 查看zone配置
$ firewall-cmd --list-all
$ firewall-cmd --list-all-zones
```

#### Tip.40 history命令显示命令时间信息

```
$ export HISTTIMEFORMAT='%F %T '
```

#### Tip.41 Mysql更改连接权限

```
# 顺便修改配置密码
> UPDATE user SET password=PASSWORD("tyrion") WHERE user='root';
> GRANT ALL PRIVILEGES ON *.* TO 'root[$user]'@'%[$ip]' IDENTIFIED BY 'tyrion[$password]' WITH GRANT OPTION;
> FLUSH PRIVILEGES;
```

#### Tip.42 Linux查看缓存行大小

```
$ cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size
```

#### Tip.43 Linux jmap命令报错

```
## 问题往往是debuginfo没有安装
$ yum --enablerepo="*-debug*" install java-1.8.0-openjdk-debuginfo -y
```

#### Tip.44 jstatd如何使用

```
## 知晓：一般安装java-1.8.0-openjdk
## java.home目录为${java-1.8.0-openjdk}/jre目录
## 如：/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.201.b09-2.el7_6.x86_64/jre

## 开启jstatd首选需要选定配置文件
## /usr/lib/jvm/java-1.8.0-openjdk/jre/lib/security
## cp java.policy jstatd.all.policy
## 修改jstatd.all.policy中内容如下:

grant codebase "file:${java.home}/../lib/tools.jar" {
   permission java.security.AllPermission;
};

## 一般目录说明:
## /bin/java 最终-> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.201.b09-2.el7_6.x86_64/jre/bin/java，属于jre下的java
## jdk目录为:/usr/lib/jvm/java-1.8.0-openjdk
## /usr/lib/jvm/java-1.8.0-openjdk 最终-> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.201.b09-2.el7_6.x86_64 (为openjdk真正的安装目录)
## jdk目录下的lib下含有tools.jar文件

## jstatd如何启动：没啥好解释的，visualvm可用jstatd远程监控内存gc，真是神器！
## 必要时java.security.policy可以使用绝对路径指定
## java.rmi.server.logCalls打开日志,如果客户端有连接过来的请求,可以监控到,便于排错
$ jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=10.10.10.9 -p 3333 -J-Djava.rmi.server.logCalls=true &
```

#### Tip.45 git拉取远程分支

```
$ git branch -r # 假设有master和feature两个分支
$ git branch --track feature origin/feature # 将本地branch中的feature关联远程分支origin/feature
$ git fetch --all && git pull --all
```

#### Tip.46 Python3的CFLAGS等环境变量取址

```
# 查找方式：入口是python3-config --cflags;
# 直接找到python3-config命令，是一段python代码，cflags返回来自sysconfig.get_config_var('CFLAGS')
# 生成cflags参数是在_init_posix中得到的
# 真正cflags参数是硬编码在如下文件中：
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/_sysconfigdata_m_darwin_darwin.py

# cython在编译中gcc-8会直接读取这部分cflags参数直接使用
```

#### Tip.47 CentOS更改docker启动service配置

```
$ vim /usr/lib/systemd/system/docker.service
# 修改这句
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --graph=/data1/docker/images --insecure-registry hub.docker.com
```

#### Tip.48 nc命令直接传输文件目录

```
## 接收端
$ nc -l 10.10.10.22 19995 | tar xfvz -
## 发送端
$ tar cfz - * | nc 10.10.10.22 19995
```

#### Tip.49 垃圾Centos,只能升级gcc

```
$ yum -y update
$ yum -y install bzip2 wget gmp-devel mpfr-devel libmpc-devel
$ wget http://mirrors-usa.go-parts.com/gcc/releases/gcc-8.2.0/gcc-8.2.0.tar.gz
$ tar -zxvf gcc-8.2.0.tar.gz
$ mkdir gcc-8.2.0-build
$ cd gcc-8.2.0-build
$ ../gcc-8.2.0/configure --enable-languages=c,c++ --disable-multilib
$ make -j$(nproc)
$ make install
$ gcc --version
```

#### Tip.50 查看硬盘与分区的有用命令

```
$ lsblk -a
$ blkid
$ sfdisk -l /dev/sda
$ cfdisk -Ps /dev/sda
$ ls -l /dev/disk/by-uuid/

$ hdparm -i /dev/sda
$ smartctl -a /dev/sda

$ swapon -s 或者 cat /proc/swaps ## 查看交换分区信息
```

#### Tip.51 ping无效

```
## 查看gateway是否正确
$ vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

#### Tip.52 读取网卡UUID

```
$ yum -y install NetworkManager && service NetworkManager start
$ nmcli con
```

打印信息：

```
NAME         UUID                                  TYPE      DEVICE
docker0      26907f61-70bd-4b02-a130-80256e16ca3d  bridge    docker0
eth1         c56addc9-e5d4-4cfb-87f5-32b416a1b5ef  ethernet  eth1
System eth0  5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet  eth0
System eth1  9c92fad9-6ecb-3e6c-eb4d-8a47c6f50c04  ethernet  --
System eth2  3a73717e-65ab-93e8-b518-24f5af32dc0d  ethernet  --
System eth3  c5ca8081-6db2-4602-4b46-d771f4330a6d  ethernet  --
```

#### Tip.53 查询网卡状态

```
$ mii-tool ethx
$ ethtool ethx
```

#### Tip.54 vagrant+virtualbox添加usb支持

首先安装virtualbox扩展：[Extension下载地址](https://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html#extpack)

```
$ VBoxManage list usbhost
```

```
Host USB Devices:

UUID: 7c078866-a1ef-43fc-a67c-df06ae84e4ac
VendorId: 0x04f2 (04F2)
ProductId: 0xb40a (B40A)
Revision: 105.100 (105100)
Port: 0
USB version/speed: 2/High
Manufacturer: Chicony Electronics Co., Ltd
Address: {36fc9e60-c465-11cf-8056-444553540000}\0008
Current State: Busy

UUID: dec2a0f4-33db-4a42-9af7-ee7beb034c77
VendorId: 0x13d3 (13D3)
ProductId: 0x3362 (3362)
Revision: 0.1 (0001)
Port: 0
USB version/speed: 1/Full
Manufacturer: Atheros Communications
Product: Bluetooth USB Host Controller
SerialNumber: Alaska Day 2006
Address: {e0cbf06c-cd8b-4647-bb8a-263b43f0f974}\0000
Current State: Busy

UUID: 2f9068df-3345-477b-8de9-d0272bbbaf15
VendorId: 0x1a86 (1A86)
ProductId: 0x7523 (7523)
Revision: 2.84 (0284)
Port: 0
USB version/speed: 1/Full
Manufacturer: QinHeng Electronics
Product: USB2.0-Serial
Address: {4d36e978-e325-11ce-bfc1-08002be10318}\0001
Current State: Busy
```

Vagrantfile添加配置内容：

```
vb.customize ["modifyvm", :id, "--usb", "on"]
vb.customize ['usbfilter', 'add', '0', '--target', :id, '--name', 'usb extension', '--vendorid', '0x1a86', '--productid', '0x7523']
```

#### Tip.55 内存/CPU监控

```
$ vmstat 2 1000     ## 每2秒执行，执行1000次
$ vmstat -s -S M    ## 打印内存监控信息，单位MB
$ htop    ## 用F5看到进程里面的线程，树形目录父子关系
$ memstat -p pid   ## 查看内存
$ memstat -w
```

### 参考文献

- [python3安装第三方包](https://www.jianshu.com/p/9acc85d0ff16)
- [vagrant+virtualbox连接usb](https://sonnguyen.ws/connect-usb-from-virtual-machine-using-vagrant-and-virtualbox)
- [fio读写性能测试](https://www.alibabacloud.com/help/zh/doc-detail/25382.htm)
- [Linux cpu/mem监控命令](https://www.cnblogs.com/arnoldlu/p/9462221.html)
- [Linux内存监控命令](https://linux.cn/article-4836-2.html)
- [Maven配置文件详解](https://blog.csdn.net/u012152619/article/details/51485297)
