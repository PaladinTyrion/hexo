---
title: docker-mesos-demo
date: 2017-09-10 18:40:59
categories: system
type: "tags"
tags: 
  - mesos
---

#### 主要参考运行示例：

<!-- more -->

```
#每台安装过docker的node运行
$ docker run -d swarm join --advertise=10.236.24.156:2375 zk://10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181/swarm

#其中任何一台node运行
$ docker run -d -p 2376:2375 --name swarm_manage swarm manage zk://10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181/swarm

#选三台node运行docker-zk，每台MYID需要不同
docker run -d \
 -e MYID=3 \
 -e SERVERS=10.236.24.143,10.236.24.140,10.236.24.138 \
 --name=zookeeper --net=host --restart=always mesoscloud/zookeeper:3.4.8-ubuntu-14.04

#3个mesos-master运行
$ docker run -d -e MESOS_QUORUM=2 \
 -e MESOS_ZK=zk://10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181/mesos \
 --name=mesos-master \
 --net=host \
 mesoscloud/mesos-master:0.28.1-ubuntu

#每个mesos-slave运行
$ docker run -d -p 80:80 -e MESOS_MASTER=zk://10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181/mesos -v /sys/fs/cgroup:/sys/fs/cgroup  -v /var/run/docker.sock:/var/run/docker.sock  -v /vagrant/vagrant:/vagrant/vagrant  --net=host  --privileged  --restart always mesoscloud/mesos-slave:0.28.1-ubuntu 

#任何一台node运行marathon
$ docker run -d -p 8089:8080 \
 --name marathon --net host \
 -e MARATHON_MASTER=zk://10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181/mesos \
 -e MARATHON_ZK=zk://10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181/marathon \
 --restart always mesoscloud/marathon:1.1.1-ubuntu
 
#任何一台node运行chronos
$ docker run -d --net=host \
 -e CHRONOS_HTTP_PORT=4400 \
 -e CHRONOS_MASTER=zk://10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181/mesos \
 -e CHRONOS_ZK_HOSTS=10.236.24.143:2181,10.236.24.140:2181,10.236.24.138:2181 \
 --name chronos --restart always mesoscloud/chronos:2.4.0-ubuntu-14.04
```

```
#spark测试demo,这里是我自己上传的应用
docker run --rm --net=host \
 -p 4040:4040 \
 -p 8080:8080 \
 --name spark-demo \
 -v /vagrant/vagrant/examples:/example \
 paladintyrion/mesos1.0.2-spark2.0.0:latest \
 spark-submit --master=mesos://zk://10.236.23.65:2181,10.236.23.31:2181,10.236.23.67:2181/mesos \
 --deploy-mode=client \
 --conf spark.mesos.coarse=true \
 --conf spark.mesos.executor.docker.image=paladintyrion/mesos1.0.2-spark2.0.0:latest \
 --conf spark.executor.memory=768m \
 --class Main \
 /example/hello-1.0-SNAPSHOT.jar \
 200
 
 #spark测试demo
 docker run --net=host \
 --name spark-demo1 \
 -v /vagrant/vagrant/examples:/example \
 paladintyrion/mesos1.0.2-spark2.0.0:latest \
 spark-submit --master=mesos://zk://10.236.23.65:2181,10.236.23.31:2181,10.236.23.67:2181/mesos \
 --deploy-mode=client \
 --conf spark.mesos.coarse=true \
 --conf spark.mesos.executor.docker.image=paladintyrion/mesos1.0.2-spark2.0.0:latest \
 --conf spark.executor.memory=768m \
 /example/src/main/python/pi.py \
 200
 
 #spark测试demo，去掉image运行
 docker run --net=host \
 --name spark-demo1 \
 -v /vagrant/vagrant/examples:/example \
 -v /vagrant/vagrant:/sparkhome \
 paladintyrion/mesos1.0.2-spark2.0.0:latest \
 spark-submit --master=mesos://zk://10.236.23.65:2181,10.236.23.31:2181,10.236.23.67:2181/mesos \
 --deploy-mode=client \
 --conf spark.mesos.coarse=true \
 --conf spark.executor.memory=768m \
 --conf spark.executor.uri=/sparkhome/spark-2.0.1-bin-hadoop2.7.tgz \
 --class Main \
 /example/hello-1.0-SNAPSHOT.jar \
 150
 
 docker run --net=host \
 --name spark-demo1 \
 -v /vagrant/vagrant/examples:/example \
 -v /vagrant/vagrant:/sparkhome \
 paladintyrion/mesos1.0.2-spark2.0.0:latest \
 spark-submit --master=mesos://zk://10.236.23.65:2181,10.236.23.31:2181,10.236.23.67:2181/mesos \
 --deploy-mode=client \
 --conf spark.mesos.coarse=true \
 --conf spark.executor.memory=1g \
 --conf spark.executor.uri=/sparkhome/spark-2.0.1-bin-hadoop2.7.tgz \
 /example/src/main/python/pi.py \
 100
```

```
docker run --net=host  --name spark-demo1  -v /vagrant/vagrant/examples:/example  -v /vagrant/vagrant:/sparkhome  paladintyrion/mesos1.0.2-spark2.0.0:latest  spark-submit --master=mesos://zk://10.236.23.65:2181,10.236.23.31:2181,10.236.23.67:2181/mesos  --deploy-mode=client  --conf spark.mesos.coarse=true  --conf spark.executor.memory=768m  --conf spark.executor.uri=/sparkhome/spark-2.0.1-bin-hadoop2.7.tgz  --class JavaWordCount /example/jwc-1.0-SNAPSHOT.jar file:///sparkhome/Vagrantfile

docker run --net=host  --name spark-demo  -v /vagrant/vagrant/examples:/example  -v /vagrant/vagrant:/sparkhome  paladintyrion/mesos1.0.2-spark2.0.0:latest  spark-submit --master=mesos://zk://10.236.23.65:2181,10.236.23.31:2181,10.236.23.67:2181/mesos  --deploy-mode=client  --conf spark.mesos.coarse=true  --conf spark.executor.memory=1g --conf spark.eventLog.dir=file:///vagrant/vagrant/eventLogs/ --conf spark.sql.warehouse.dir=file:/// --conf spark.executor.uri=/sparkhome/spark-2.0.1-bin-hadoop2.7.tgz  --class org.apache.spark.examples.JavaWordCount /sparkhome/spark-2.0.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.0.1.jar file:///sparkhome/Vagrantfile

spark-submit --master=spark://paladin:7077 --deploy-mode=client --conf spark.mesos.coarse=true --conf spark.driver.memory=1g --class org.apache.spark.examples.JavaWordCount /Users/paladintyrion/src/src/Spark/spark-2.0.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.0.1.jar file:///Users/paladintyrion/derby.log
```
