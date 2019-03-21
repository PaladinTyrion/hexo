---
title: Flink1.7.2简明使用
date: 2019-03-21 10:58:53
categories: framework
tags:
  - flink
---

### Flink依赖环境

#### java

安装java-1.8.0-openjdk，过程略

#### ssh

配置ssh可免密互访，过程略

为了防止ssh的22端口被占用，一般可以自己重启一个sshd，关注/etc下的配置文件

```
$ vim /etc/hosts.allow
sshd: ALL
#sshd:10.10.10.*:allow

$ vim /etc/ssh/sshd_config && service sshd restart
# 关注以下几点配置
Port 26387  #ssh的配置端口号
AllowUsers root@10.10.10.1   #允许连接的远程用户
AllowUsers root@10.10.10.2
AllowUsers root@10.10.10.3
AllowUsers root@10.10.10.4
AllowUsers root@10.10.10.5
PubkeyAuthentication yes
AuthorizedKeysFile  .ssh/authorized_keys
PasswordAuthentication no

# 调试ssh时可以才用debug模式来获得ssh连接有什么报错信息，主要查看server的调试信息
$ sshd -d -p 2222   #开server，-d表示调试模式，-p代表占用端口
$ ssh -v root@kafka2 -p 2222 #开client，-v表示输出打印栈信息
```

<!-- more -->

#### hadoop集群

提前配置好远程hadoop集群，假设namenode为hdfs://10.99.99.1:9000，配置过程略

#### zookeeper集群

提前配置好zk集群，假设zk集群10.99.98.1:2181,10.99.98.2:2181,10.99.98.3:2181，配置过程略

### FLink配置及启动

#### 节点信息及角色分配

下载flink-1.7.2-bin-hadoop28-scala_2.12.tgz，选用hadoop版本主要是为了支持hdfs连接，元信息及checkpoint需要hdfs，否则选用nfs也可以，但hdfs更好。另外flink-1.7.2-bin-scala_2.12.tgz仅支持single jobmanager，所以弃用。

#### 配置环境变量

```
$ export FLINK_HOME=/to/flink/path
$ export PATH=$PATH:$FLINK_HOME/bin
```

#### 配置/etc/hosts

```
# 假设5台机器，3个zk，3个jobmanager，5个taskmanager
10.10.10.1   flink1
10.10.10.2   flink2
10.10.10.3   flink3
10.10.10.4   flink4
10.10.10.5   flink5
```

#### flink配置文件

需要修改conf下的masters，slaves，flink-conf.yaml，修改bin/config.sh

##### flink1.7.2/bin/config.sh

```
# 添加下面两个
export FLINK_SSH_OPTS="-p 26387"
export JAVA_HOME="/usr/lib/jvm/jre-1.8.0-openjdk"
```

##### flink1.7.2/conf/masters

```
flink1:8081
flink2:8081
flink3:8081
```

##### flink1.7.2/conf/slaves

```
flink1
flink2
flink3
flink4
flink5
```

##### flink1.7.2/conf/flink-conf.yaml

```
env.java.home: /usr/lib/jvm/jre-1.8.0-openjdk
env.log.dir: /data0/flink/log

#jobmanager.rpc.address: 0.0.0.0    #不要配置，ha mode此项无效，此项仅在flink only的single jobmanager情况下work
#jobmanager.rpc.port: 6123     #不要配置，ha mode此项无效，此项仅在flink only的single jobmanager情况下work
jobmanager.heap.size: 2048m    #主要负责调度Task、协调检查点、协调失效恢复，与Client和TaskManager交互，Client将JobGraph提交给JobManager，然后其将JobGraph转换为ExecutionGraph，并分发到TaskManager上执行(也就是负责执行计划生成，并把task下发到taskmanager，并监控任务执行)
taskmanager.heap.size: 16384m   #tm堆内存
taskmanager.numberOfTaskSlots: 8  #每个slot可以分得2g内存
parallelism.default: 2
#fs.default-scheme: hdfs://10.99.99.1:9000   #后面的文件都对应此基路径的hdfs存储，不设则默认是file:///，影响如io.tmp.dirs等

high-availability: zookeeper
high-availability.storageDir: hdfs://10.99.99.1:9000/flink/recovery/
high-availability.zookeeper.quorum: 10.99.98.1:2181,10.99.98.2:2181,10.99.98.3:2181
high-availability.zookeeper.path.root: /flink  #zk的存储根路径
high-availability.cluster-id: /cluster_one
high-availability.zookeeper.client.acl: open

state.backend: filesystem   #可选'jobmanager', 'filesystem', 'rocksdb'，其中'jobmanager'就是把状态信息都存在jobmanager堆内存中
state.checkpoints.dir: hdfs://10.99.99.1:9000/flink/checkpoints/
state.savepoints.dir: hdfs://10.99.99.1:9000/flink/checkpoints/
state.backend.incremental: false  #rocksdb可以设为true
state.checkpoints.num-retained: 1

rest.address: 0.0.0.0
web.address: 0.0.0.0
web.tmpdir: /data0/flink/tmp
rest.port: 8081
web.submit.enable: true

io.tmp.dirs: /data0/flink/tmp
taskmanager.memory.preallocate: false   #只做纯的批处理任务，可以true此项，streaming不建议开始。使用堆外内存时候，要开启此项。开启后，streaming不再使用taskmanager的内存管理，rocksdb自己管理内存，memory和filesystem需要明确的保持数据，以节省序列化损耗。
classloader.resolve-order: child-first   #先加载class
taskmanager.network.memory.fraction: 0.1   #10%tm堆内存用于网络buffer
taskmanager.network.memory.min: 64mb   #最小不能小于64mb
taskmanager.network.memory.max: 1gb
taskmanager.rpc.port: 50100-50300
taskmanager.memory.fraction: 0.7    #for sorting, hash tables以及缓存中间结果使用
taskmanager.memory.off-heap: false
taskmanager.memory.size: 0

jobmanager.archive.fs.dir: hdfs://10.99.99.1:9000/flink/completed-jobs/
jobmanager.execution.attempts-history-size: 32
historyserver.web.address: 0.0.0.0
historyserver.web.port: 8082
historyserver.archive.fs.dir: hdfs://10.99.99.1:9000/flink/completed-jobs/
historyserver.archive.fs.refresh-interval: 10000
historyserver.web.tmpdir: /data0/flink/tmp
```

#### Flink启动

```
$ $FLINK_HOME/bin/start-cluster.sh
```

### 参考文献

- [Flink架构及工作原理介绍](http://lionheartwang.github.io/blog/2018/03/05/flink-framwork-introduction/)