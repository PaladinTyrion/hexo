---
title: nfs挂载
date: 2017-09-16 16:21:29
categories: OS
tags: 
  - Centos
  - Nfs
---

假设三台机器10.3.92.1，10.3.92.2，10.3.92.3，其中10.3.92.1是nfs的server，另外两台机来挂载

#### 安装nfs

```
# 三台机子分别安装，挂载机如果不做nfs server服务，可只安装rpcbind
$ yum install rpcbind nfs-utils
```

<!-- more -->

#### 开启nfs服务端

启动对应的nfs服务，rpcbind是用来做服务通信，nfs-utils包含nfs服务

```
# 设置开机启动
$ chkconfig rpcbind on
$ chkconfig nfs on
# 启动相关服务
$ service rpcbind start
$ service nfs start

# 如果上面的chkconfig命令如果失败，用下面条命令替代，
# chkconfig命令命令失败，则service也不能开启服务，需用systemctl,
# 在client端，只开启rpcbind服务即可
$ systemctl enable rpcbind
$ systemctl enable nfs 或者 systemctl enable nfs-server
$ systemctl start rpcbind
$ systemctl start nfs 或者 systemctl start nfs-server
```

##### nfs server设置nfs卷输出

提前创建好目录/data0/dir1，/data0/dir2

```
# 编辑/etc/exports，内容如下:
# /data0/dir1目录只允许10.3.92.2读写,
# /data0/dir2目录允许任何ip读写
/data0/dir1	10.3.92.2(rw,sync,no_root_squash,no_subtree_check)
/data0/dir2	*(rw,sync,no_root_squash,no_subtree_check)
```

说明:

- rw – 允许对共享目录进行读写
- ro — 只读
- sync – 同步模式，内存中数据时时写入磁盘
- async — 不同步，把内存中数据定期写入磁盘中
- no_root_squash – 允许root访问
- no_all_squash – 允许用户授权
- all_squash – 不管使用NFS的用户是谁，他的身份都会被限定成为一个指定的普通匿名用户
- no_subtree_check – 如果卷的一部分被输出，从客户端发出请求文件的一个常规的调用子目录检查验证卷的相应部分。如果是整个卷输出，禁止这个检查可以加速传输
- anonuid/anongid – 要和root_squash 以及 all_squash一同使用，用于指定使用NFS的用户限定后的uid和gid，前提是本机的/etc/passwd中存在这个uid和gid

##### 生效配置

```
$ exportfs -arv    # a表示all,r表示重新加载,v表示打印输出
```

说明:

- a ：全部挂载或者卸载；
- r ：重新挂载；
- u ：卸载某一个目录；
- v ：显示共享的目录；

#### nfs client配置nfs挂载

##### 查看可挂载的server目录

```
$ showmount -e 10.3.92.1
```

##### 挂载nfs目录

/etc/fstab格式:

1 | 2 | 3 | 4 | 5 | 6
--- | --- | --- | --- | --- | --- 
fs_spec | fs_file | fs_type | fs_options | fs_dump | fs_pass

- Fs_spec: 定义希望加载的文件系统所在的设备或远程文件系统，对于nfs则设为IP:/共享目录
- Fs_file: 本地挂载点
- Fs_type: 挂载类型
- Fs_options: 挂载参数
- Fs_dump: 该选项被“dump”命令使用来检查一个文件系统该以多快频率进行转储，若不需转储即为0
- Fs_pass: 该字段被fsck命令使用来决定在启动时需要被扫描的文件系统的顺序，根文件系统“/”对应该字段值为1,其他文件系统为2,若该文件系统无需在启动时被扫描则为0

> 挂载参数的说明

参数 |          代表意义         | 系统默认值
--- | --- | --- 
suid/nosuid |  | suid
rw/ro | 读写，只读 | rw
dev/nodev | 是否保留装置档案的特殊功能, 一般只有/dev这个目录才会有特殊的装置 | dev
exec/noexec | 是否具有执行binary file的权限 | exec
user/nouser | 是否允许使用者进行档案的挂载与卸除功能 | nouser
auto/noauto | mount -a时，会不会被挂载. 若不需要这个 partition 随时被挂载, 可设定为noauto | auto
fg/bg | 执行挂载时,挂载的行为会在前景(fg)还是在背景(bg)执行.若fg时,则mount会持续尝试挂载,直到成功或timeout为止;若bg时,则 mount会在背景持续多次进行mount,而不会影响到前景的程序操作 | fg
soft/hard | 若hard,则当两者之间有任何一部脱机,则RPC会持续的呼叫,直到对方恢复联机为止;若soft,RPC会在timeout后**重复**呼叫,而非**持续**呼叫,系统延迟会比较不明显 | hard
intr | 当使用hard方式挂载,intr表示当RPC持续呼叫中,呼叫是可以被中断的(interrupted) | 没有
rsize/wsize | 读出(rsize)与写入(wsize)的区块大小(block size) | rsize=1024,wsize=1024

```
# 编辑/etc/fstab，10.3.92.2添加如下内容
10.3.92.1:/data0/dir1 /data/localdir1 nfs auto,soft,bg,intr,rw,rsize=32768,wsize=32768 0 0
10.3.92.1:/data0/dir2 /data/localdir2 nfs auto,soft,bg,intr,rw,rsize=32768,wsize=32768 0 0
# 注意，由于server的配置的两个目录均可接受10.3.92.2读写，在10.3.92.3上不可读写/data0/dir1，特别注意
```

##### 挂载

```
$ mount -a
```

##### 查看挂载状态

```
$ df -h -T
```

#### 参考文献

- http://wiki.jikexueyuan.com/project/linux/nfs.html
- http://vinny.cc/2017/02/how-to-use-nfs-in-centos7/
- http://cn.linux.vbird.org/linux_server/0330nfs.php
- http://www.oicto.com/unix-nfs-mount/
- http://www.cnblogs.com/rootq/articles/1310888.html