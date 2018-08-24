---
title: centos升级docker版本
date: 2018-06-04 14:21:21
categories: docker
tags:
  - Centos
  - Docker
---

#### yum依赖python的问题

centos如果版本过老，如centos 6.5，安装docker会存在依赖性问题，按照docker官网的安装，会要求升级linux-head等等，服务器一般是不能随便重启的，因而只能降低docker升级的版本要求。在centos 6.8之前，docker最高可升级到1.7.1版本。

centos当python升级后，yum的安装会提示Not found yum module，因此我们需要回滚yum命令的python版本。

```
## 将#!/usr/bin/python 修改为 #!/usr/bin/python2.6
$ vim /usr/bin/yum
$ vim /usr/bin/yum-config-manager
```

<!-- more -->

#### 清空yum缓存

```
$ sudo yum clean all
```

#### 更换yum源

一般更换阿里云的yum源，修改/etc/yum.repos.d/CentOS-Base.repo的内容。

```
# CentOS-Base.repo
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6

#released updates
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-6

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
```

#### 更换docker安装的yum源

```
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

命令在/etc/yum.repos.d/下生成docker.repo源文件，如果里面的centos是7的源，需更改源地址内容。

```
[dockerrepo]
name=Docker Repository
#原来可能是baseurl=https://yum.dockerproject.org/repo/main/centos/7/ 
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
```

Tips：更换完源地址之后，需要为yum清空cache，并重新生成cache。

#### 生成yum本地缓存

将服务器上的软件包信息在本地缓存,以提高搜索安装软件的速度，也可以不生成cache。

生成的cache文件在/var/cache/yum/x86_64/$releasever/目录下。

```
$ sudo yum makecache
```

#### 卸载旧版本docker后安装新版docker

```
$ yum list docker-engine.x86_64 --showduplicates | sort -r ##可以查看可安装版本
$ yum install -y docker-engine  ##或者docker-io
```

#### 修改docker配置文件

Centos docker配置文件位置：/etc/sysconfig/docker; Ubuntu/MacOS docker配置文件位置：/etc/default/docker

配置文件的内容如下：

```
DOCKER_OPTS="-H 0.0.0.0:2375 -H unix:///var/run/docker.sock --selinux-enabled"
```

配置的复杂一下，可以参考如下的配置：

```
DOCKER_OPTS="-H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock -s=devicemapper --ip-forward=true [--bridge=none] --iptables=false  --label ip=10.0.10.10 --label idc=aliyun --graph=/data1/docker/images"
INSECURE_REGISTRY="--insecure-registry registry.intra.xxx.com --insecure-registry registry.api.xxx.com"
```

#### 重启docker服务

```
$ service docker restart
```
