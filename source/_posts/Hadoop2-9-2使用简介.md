---
title: Hadoop2.9.2使用简介
date: 2019-03-06 11:38:49
categories: hadoop
tags:
  - hadoop
---

本文主要介绍Hadoop搭建以及若干性能参数说明。

### Hadoop搭建

#### 选机器

所有机器要修改/etc/hosts

- 10.16.10.1 hadoop1 #这台做master
- 10.16.10.2 hadoop2
- 10.16.10.3 hadoop3

<!-- more -->

#### SSH环境

集群所有机器ssh互相免密登陆，设置略。ssh端口可以自己设置，并在hadoop-env.sh中export HADOOP_SSH_OPTS="-p $ssh_port"

#### 配置hadoop安装目录etc/hodoop下多份配置文件

##### hadoop-env.sh

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/jre
export HADOOP_SSH_OPTS="-p 2222"
export HADOOP_NAMENODE_OPTS="-XX:+UseParallelGC"
export HADOOP_LOG_DIR=/data0/hadoop/log
export HADOOP_HEAPSIZE=1536
export YARN_RESOURCEMANAGER_HEAPSIZE=1024
export YARN_NODEMANAGER_HEAPSIZE=1024
export YARN_PROXYSERVER_HEAPSIZE=1024
export HADOOP_JOB_HISTORYSERVER_HEAPSIZE=1024

export HADOOP_HOME=/data0/thd/hadoop-2.9.2
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
```

##### yarn-env.sh & mapred-env.sh

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/jre
```

##### core-site.xml

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop1:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data0/hadoop/tmp</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>1048576</value>
    </property>
    <property>
        <name>fs.trash.interval</name>
        <value>1440</value>
    </property>
    <property>
        <name>fs.trash.checkpoint.interval</name>
        <value>1440</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>
</configuration>
```

##### hdfs-site.xml

```
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///data0/hadoop/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///data0/hadoop/data</value>
    </property>
    <property>
	<name>dfs.blocksize</name>
	<value>268435456</value>
    </property>
    <property>
        <name>dfs.namenode.handler.count</name>
        <value>30</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop1:50090</value>
    </property>
    <property>
    	<name>dfs.webhdfs.enabled</name>
    	<value>true</value>
    </property>
    <property>
        <name>dfs.support.append</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.support.broken.append</name>
        <value>true</value>
    </property>
</configuration>
```

##### mapred-site.xml

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
	<final>true</final>
    </property>
    <property>
	<name>mapreduce.map.memory.mb</name>
	<value>2048</value>
    </property>
    <property>
        <name>mapreduce.reduce.memory.mb</name>
        <value>4096</value>
    </property>
    <property>
 	<name>mapreduce.reduce.shuffle.parallelcopies</name>
	<value>10</value>
    </property>
    <property>
	<name>mapreduce.task.io.sort.mb</name>
	<value>256</value>
    </property>
    <property>
	<name>mapreduce.task.io.sort.factor</name>
	<value>10</value>
    </property>
    <property>
        <name>mapreduce.map.java.opts</name>
        <value>-Xmx1792m</value>
    </property>
    <property>
        <name>mapreduce.reduce.java.opts</name>
        <value>-Xmx3584m</value>
    </property>
    <property>
        <name>mapreduce.reduce.input.buffer.percent</name>
        <value>0.0</value>
    </property>
    <property>
        <name>mapreduce.reduce.shuffle.input.buffer.percent</name>
        <value>0.9</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>0.0.0.0:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>0.0.0.0:19888</value>
    </property>
</configuration>
```

##### yarn-site.xml

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
    </property>
    <property>
        <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>hadoop1:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>hadoop1:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>hadoop1:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>hadoop1:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>hadoop1:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop1</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>8192</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>4096</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>4</value>
    </property>
</configuration>
```

##### slaves

```
hadoop1
hadoop2
hadoop3
```

### 说明

#### 环境变量调用路径

- HADOOP_HEAPSIZE: hadoop -> hadoop-config.sh -> hadoop-env.sh
- hdfs -> hdfs-config.sh -> hadoop-config.sh -> hadoop-env.sh
- YARN_HEAPSIZE: yarn -> yarn-config.sh -> hadoop-config.sh -> hadoop-env.sh

#### 节点角色分配

- hadoop1(master+slave): ResourceManager(yarn), NodeManager(yarn), NameNode(master-dfs), SecondaryNameNode(master-dfs), DataNode(hdfs)
- hadoop2(slave):  NodeManager(yarn), DataNode(hdfs)
- hadoop3(slave):  NodeManager(yarn), DataNode(hdfs)

#### 启动集群及关闭集群

设置过ssh免密登陆，可简单启动和关闭。

```
# 第一次使用namenode服务，需要在NN-master作格式化
$ hdfs namenode -format <cluster_name>
# 先启hdfs再yarn
$ start-dfs.sh && start-yarn.sh
$ $HADOOP_HOME/sbin/mr-jobhistory-daemon.sh start historyserver

# 先关yarn再hdfs
$ stop-yarn.sh && stop-dfs.sh
```

### 参考文献

- [CentOS7下搭建Hadoop2.9分布式集群](https://www.linuxidc.com/Linux/2018-11/155328.htm)
- [Hadoop Cluster Setup](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)
- [Hadoop安装](https://www.jianshu.com/p/4c81a1e32161)
- [YARN内存参数终极详解](https://blog.51cto.com/meiyiprincess/1747113)
- [Yarn内存分配管理机制](https://blog.csdn.net/suifeng3051/article/details/45477773)
- [Yarn下Mapreduce的内存参数理解](https://segmentfault.com/a/1190000003777237)