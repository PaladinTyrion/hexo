---
title: yum小记
date: 2017-09-14 19:42:53
categories: OS
tags: 
  - Linux
  - Centos
---

#### yum源更换

##### yum的配置路径

```
# yum的配置路径如下:
/etc/yum.conf
/etc/yum.repos.d/

# 其中,xxx.repo中配置下载url
/etc/yum.repos.d/CentOS-Base.repo
```

<!-- more -->

xxx.repo配置格式如下, 其中[]中的内容必须唯一，可设置多个不同的[]，源地址，baseurl即为源：

```
[base]
name=CentOS-$releasever-Base
failovermethod=priority
baseurl=http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/  
gpgcheck=1
gpgkey=http://mirrors.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-7
       http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-7
```

##### 重建yum源信息

```
$ yum clean all
$ yum makecache
```

可以通过下面命令查看可下载的包，如要下载policycoreutils：

```
$ yum list --showduplicates | grep policycoreutils.x86_64 | sort
```

如返回结果如下：

```
policycoreutils.x86_64                   2.2.5-15.el7               base-vault
policycoreutils.x86_64                   2.5-11.el7_3               @updates
policycoreutils.x86_64                   2.5-11.el7_3               updates
policycoreutils.x86_64                   2.5-8.el7                  base
policycoreutils.x86_64                   2.5-9.el7                  updates
```

#### 安装

下载指定版本的包通过如下命令：

```
$ yum install policycoreutils-2.5-11.el7_3.x86_64
或者
$ yum update && yum install/upgrade policycoreutils
```

#### yum更新出现依赖冲突时的解决方案

```
$ yum install yum-utils
$ package-cleanup --dupes
$ package-cleanup --cleandupes
$ yum clean all && yum update
```
