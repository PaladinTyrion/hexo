---
title: docker-mesos-spark
date: 2017-09-10 18:36:18
categories: system
type: "tags"
tags: 
  - mesos
---

#### Docker

##### Docker machine  

Docker machine负责docker在本地或远程机及虚拟云上的安装，可在自己的Mac或Linux机上负责远程活本地的虚拟机上的docker安装。

<!-- more -->

###### 安装Docker machine:

```
$ curl -L https://github.com/docker/machine/releases/download/v0.8.2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
chmod +x /usr/local/bin/docker-machine

$ docker-machine version
```

注意安装远程机之前一定确保运行机可ssh登录远程机。

```
$ docker-machine create --driver generic \
--generic-ip-address=10.236.24.179 \
--generic-ssh-user=vagrant \
--generic-ssh-key ~/.ssh/id_rsa \
vagrant             #this means hostname
```

安装docker之后确保每台安装机运行:

```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

```
$ docker-machine env default  #"default为远程安装机的hostname"
 export DOCKER_TLS_VERIFY="1"
 export DOCKER_HOST="tcp://172.16.62.130:2376"
 export DOCKER_CERT_PATH="/Users/<yourusername>/.docker/machine/machines/default"
 export DOCKER_MACHINE_NAME="default"
 # Run this command to configure your shell:
 # eval "$(docker-machine env default)"
```

不使用docker-machine安装docker:

```
# Ubuntu机
$ echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
$ sudo apt-get update
$ apt-cache policy docker-engine
$ sudo apt-get install docker-engine
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

###### docker machine的运行机制:   
![docker machine](docker_machine.png)

---

##### Docker swarm

Swarm中提供了6种不同的发现（discovery）机制：token(默认) Discovery、Node Discovery、File Discovery、Consul Discovery、EtcD Discovery和Zookeeper Discovery。

###### token发现：

```
#生成全球唯一的token作为docker cluster的全球唯一id号
$ docker run swarm create 
```

```
$ docker run -d swarm join --advertise=<node_ip:2375> token://<cluster_id>
```

###### zk发现：

```
$ docker run -d swarm join --advertise=<node_ip:2375> zk://<zk1:2181>,<zk2:2181>,<zk3:2181>,.../swarm
```

###### swarm manage  
在需要作为manager机上运行：

```
$ docker run -d -p 2376:2375 --name swarm_manage swarm manage zk://<zk1:2181>,<zk2:2181>,<zk3:2181>,.../swarm
```

###### 使用swarm管理和启动docker

```
$ docker -H tcp://<swarm_ip:2376> <command>
#example
$ docker -H tcp://<swarm_ip:2376> info
```

###### Note:  
为了保证swarm manage管理的节点正常通信，需要每个节点做如下2点修改：

```
$ sudo vim /etc/default/docker
#添加如下内容
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"

#每机重启docker
$ sudo restart docker
```

对于Centos，docker的配置文件如下：

```
$ sudo vim /etc/sysconfig/docker

#最后可添加
OPTIONS="-H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock -s=devicemapper --ip-forward=true --bridge=none --iptables=false  --label ip=10.91.55.22 --label idc=aliyun --graph=/data0/docker/images"
INSECURE_REGISTRY="--insecure-registry registry.abc.xyz.com --insecure-registry registry.api.opq.com"
```

```
需要对每台机器添加各自的hosts
$sudo vim /etc/hosts
#添加如下内容
${local_ip}  ${hostname}
```

###### swarm的框架:  
![docker swarm](docker_swarm.jpg)

---

#### docker-Zookeeper

```
#或者mesoscloud/zookeeper:3.4.8-centos
$ docker pull mesoscloud/zookeeper:3.4.8-ubuntu-14.04

#在每台zk节点机上运行, MYID需要不同
#Ubuntu
$ docker run -d \
 -e MYID=1 \
 -e SERVERS=node-1,node-2,node-3 \
 --name=zookeeper --net=host --restart=always mesoscloud/zookeeper:3.4.8-ubuntu-14.04
 
#CentOS
$ docker run -d \
-e MYID=1 \
-e SERVERS=node-1,node-2,node-3 \
--name=zookeeper --net=host --restart=always mesoscloud/zookeeper:3.4.8-centos-7
```

---

#### docker-Mesos  
Mesos同样通过zk服务发现保证HA。  
Mesos HA模式参考[这里](http://mesos.apache.org/documentation/latest/high-availability/)

- 3个mesos master

```
docker run -d \
 -e MESOS_QUORUM=2 \
 -e MESOS_ZK=zk://node1-ip:2181,node2-ip:2181,node3-ip:2181/mesos \
 --name=mesos-master \
 --net=host \
 mesoscloud/mesos-master:0.28.1-ubuntu
```

- 每台node的mesos slave

```
docker run -d \
 -e MESOS_MASTER=zk://node1-ip:2181,node2-ip:2181,node3-ip:2181/mesos \
 -v /sys/fs/cgroup:/sys/fs/cgroup \
 -v /var/run/docker.sock:/var/run/docker.sock \
 --net host \
 --privileged \
 --restart always \
 mesoscloud/mesos-slave:0.28.1-ubuntu
```

- mesos cluster框架  
![mesos cluster](mesos_cluster.jpg)

- mesos job  
![mesos job](mesos_job.jpg)

---

#### docker-marathon

```
#这个docker-marathon没有expose 8080端口,如果需要，则制作继承镜像
$ docker run -d --name marathon --net host \
 -e MARATHON_HOSTNAME=ip.address \
 -e MARATHON_HTTPS_ADDRESS=ip.address \
 -e MARATHON_HTTP_ADDRESS=ip.address \
 -e MARATHON_MASTER=zk://node-1:2181,node-2:2181,node-3:2181/mesos \
 -e MARATHON_ZK=zk://node-1:2181,node-2:2181,node-3:2181/marathon \
 --restart always mesoscloud/marathon:1.1.1-ubuntu
 
#marathon开启应用，需要写json启动文件
如nginx.json:
{
  "id":"nginx",
  "cpus":0.4,
  "mem":200.0,
  "instances": 1,
  "container": {
    "type":"DOCKER",
    "docker": {
      "parameters": [
        {
          "key": "name",
          "value": "nginxx"
        }
      ],
      "image": "nginx",
      "network": "BRIDGE",
      "portMappings": [{
         "containerPort": 80, 
         "hostPort": 0,"servicePort": 0, 
         "protocol": "tcp"
      }]
    }
  }
}
#启动命令:
curl -X POST http://10.236.24.140:8080/v2/apps -d @/Users/paladintyrion/nginx.json -H "Content-type: application/json"
```

---

#### Mesos-Chronos

```
docker run -d --net=host \
 -e CHRONOS_HTTP_PORT=4400 \
 -e CHRONOS_MASTER=zk://node-1:2181,node-2:2181,node-3:2181/mesos \
 -e CHRONOS_ZK_HOSTS=node-1:2181,node-2:2181,node-3:2181 \
 --name chronos --restart always mesoscloud/chronos:2.4.0-ubuntu-14.04
```

---

#### mesos-spark
- spark框架  
![spark struct](spark.jpg)

- spark-mesos run  
![spark-mesos run](spark_run.png)

- spark-mesos demo

```
$ docker run --net=host \
 --name spark-demo \
 -v /vagrant/vagrant/examples:/example \
 paladintyrion/mesos1.0.2-spark2.0.0:latest \
 spark-submit --master=mesos://zk://node1:2181,node2:2181,node3:2181/mesos \
 --deploy-mode=client \
 --conf spark.mesos.coarse=true \
 --conf spark.mesos.executor.docker.image=paladintyrion/mesos1.0.2-spark2.0.0:latest \
 --conf spark.executor.memory=768m \
 /example/src/main/python/pi.py \
 200
 
 #或者非镜像运行spark
$ docker run --net=host \
 --name spark-demo1 \
 -v /vagrant/vagrant/examples:/example \
 -v /vagrant/vagrant:/sparkhome \
 paladintyrion/mesos1.0.2-spark2.0.0:latest \
 spark-submit --master=mesos://zk://node1:2181,node2:2181,node3:2181/mesos \
 --deploy-mode=client \
 --conf spark.mesos.coarse=true \
 --conf spark.executor.memory=768m \
 --conf spark.executor.uri=/sparkhome/spark-2.0.1-bin-hadoop2.7.tgz \
 /example/src/main/python/pi.py \
 200
```

---

#### 待补充
Compose
Jekins

---

#### 参考资料:
[docker-machine的命令文档](https://docs.docker.com/machine/reference/)  
[swarm discovery](https://docs.docker.com/swarm/discovery/)  
[mesos cluster](https://github.com/mesosphere/docker-containers/tree/master/mesos)

#### 示例:
[demo](../docker-mesos-demo)
