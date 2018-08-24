---
title: rsyncd服务
date: 2018-07-30 15:58:39
categories: Tools
tags:
  - Tools
---

## rsync简明使用

rsync一般可以通过配置ssh直接使用传输文件。互相有ssh公钥之后，即可完成文件传输。

```
# 目录上传
$ rsync -avz ./datadir paladintyion@10.188.25.100:/home/paladintyion/serverdir  
# 目录下载
$ rsync -avz paladintyion@10.188.25.100:/home/paladintyion/serverdir ./datadir
# 文件上传
$ rsync -avz ./datadir/* paladintyion@10.188.25.100:/home/paladintyion/serverdir 
# 文件下载
$ rsync -avz paladintyion@10.188.25.100:/home/paladintyion/serverdir/* ./datadir      
```

<!-- more -->

## rsyncd服务器配置

rsyncd需要rsync使用--daemon方式启动，默认使用/etc/rsyncd.conf配置文件，或者创建/etc/rsyncd文件夹，将rsyncd.conf放于该文件夹下。少数情况下，用户可能没有root权限，因此启动时需要指定conf文件路径及密码路径等。具体配置详见下述。

### 配置文件

假设配置文件地址为/home/paladintyrion/etc/rsyncd/rsyncd.conf。

```
###################################################################
#                    ******进程相关全局配置******
###################################################################
# = 后面的值可根据自己的实际情况更改
#    pid file 守护进程pid文件
#    port 守护进程监听端口，可更改，由xinetd允许rsyncd时忽略此参数
#    address 守护进程监听ip，由xinetd允许rsyncd时忽略此参数
pid file = /home/paladintyrion/etc/rsyncd/pid/rsyncd.pid
port = 1873
# 一般配置本机对外ip地址
address = 10.188.25.100
# rsyncd 守护进程运行系统用户全局配置，也可在具体的块中独立配置,
uid = paladintyrion
gid = paladintyrion
#允许 chroot，提升安全性，客户端连接模块，首先chroot到模块path参数指定的目录下
#chroot为yes时必须使用root权限，且不能备份path路径外的链接文件
use chroot = no
#只读
read only = no
#只写
write only = no
#允许访问rsyncd服务的ip，ip端或者单独ip之间使用空格隔开
#hosts allow = 192.168.0.1/255.255.255.0 198.162.145.1 10.0.1.0/255.255.255.0
hosts allow = *
#不允许访问rsyncd服务的ip，*是全部(不涵盖在hosts allow中声明的ip，注意和hosts allow的先后顺序)
#hosts deny = *
#客户端最大连接数
max connections = 10
#欢迎文件路径，可选的，尽量不配置吧
#motd file = /etc/rsyncd/rsyncd.motd
#日志相关
#   log file 指定rsync发送消息日志文件，而不是发送给syslog，如果不填这个参数默认发送给syslog
#   transfer logging 是否记录传输文件日志
#   log format 日志文件格式，格式参数请google
#   syslog facility rsync发送消息给syslog时的消息级别，
#   timeout连接超时时间
#lock file一定要配置一下
log file = /home/paladintyrion/etc/rsyncd/log
lock file = /home/paladintyrion/etc/rsyncd/.lock
transfer logging = yes
log format = %t %a %m %f %b
timeout = 300

###################################################################
#                    ******模块配置(多个)******
###################################################################
#模块 模块名称必须使用[]环绕，比如要访问data1,则地址应该是data1user@192.168.1.2::data1
[data]
#模块根目录，必须指定，推拉文件将以跟目录进行路径寻址，从而进行文件同步
path=/home/paladintyrion/rsync_recv
#是否允许列出模块里的内容
list=yes
#忽略错误
#ignore errors
#模块验证用户名称，可使用空格或者逗号隔开多个用户名
auth users = root,paladintyrion
#模块验证密码文件 可放在全局配置里
secrets file=/home/paladintyrion/etc/rsyncd/rsyncd.secrets
#注释
comment = private paladintyrion rsync deamon
#排除目录，多个之间使用空格隔开
exclude = test1/ test2/
```

### 密码文件

密码文件地址为/home/paladintyrion/etc/rsyncd/rsyncd.secrets，与配置文件中的配置一致。这里密码文件列出了远程用户允许同步文件的用户名和密码对。

```
root:1qaz2wsx
paladintyrion:1qaz2wsx
```

```
# 启动前务必修改rsyncd.secrets权限为600
$ chmod 600 /home/paladintyrion/etc/rsyncd/rsyncd.secrets
```

### 启动命令

命令启动后，会在/home/paladintyrion/etc/rsyncd/下创建.lock文件，跟配置中一样。

```
# 使用paladintyrion用户开启daemon进程
# -4表示只允许IPv4地址访问daemon，详情可以通过rsync --help查询
$ rsync --daemon [-4] --config=/home/paladintyrion/etc/rsyncd/rsyncd.conf
```

## client命令同步rsyncd服务器路径

### 配置文件

由于rsyncd设置了密码，rsync client也需要通过密码来访问同步文件夹。

假设有密码文件为/etc/rsyncd.passwd，修改为如下内容后，设置rsyncd.passwd权限为600，必须设置该权限。

```
1qaz2wsx
```

### client执行命令

通过rsyncd.passwd保存的密码即可正确访问。

```
# 执行后，可将服务器/home/paladintyrion/rsync_recv目录下除test1/和test2/的文件全部同步至client机/home/tyrion/目录下
$ rsync -avzP --password-file=/etc/rsyncd.passwd --port=1873 paladintyrion@10.188.25.100::data /home/tyrion/

# 执行后，可将client机/home/tyrion/data/下除去test1/和test2/的文件全部同步至服务器/home/paladintyrion/rsync_recv目录下
$ rsync -avzP --password-file=/etc/rsyncd.secret --port=1873 /home/tyrion/data paladintyrion@10.188.25.100::data
```
