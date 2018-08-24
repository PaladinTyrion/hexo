---
title: elk小结
date: 2017-10-31 00:42:48
categories: framework
type: "tags"
tags:
  - elk
---

### Elk多机部署小结
---

#### Elasticsearch

elasticsearch一般多机部署性能好，另外节点也不宜过多，因为需要在多机之间维持心跳。

elasticsearch多机也遵循quorum，一般需要在每个节点创建*.yml文件进行配置。

elasticsearch主要配置文件也就**elasticsearch.yml,jvm.options,log4j2.properties**这三样。

<!-- more -->

下面详解几个重要的配置：

```
cluster.name: 多节点时，此项必须配置，并且要保证多机的clustername一致;

# 此四项决定了node的作用，参考[1]
node.master: true
node.data: true
node.ingest: false
tribe.*

discovery.zen.ping.unicast.hosts: 配合cluster.name，这项也需要设置，列出所有的节点hosts,例如["192.168.10.10:9300"],9300为tcp通信端口，可通过transport.tcp.port进行设置
discovery.zen.minimum_master_nodes: 配合上一项设置，需要满足quorum

# 此两项防止内存交换，会很好的提速。需要配合ulimit设置项
bootstrap.memory_lock: true
bootstrap.system_call_filter: true

node.max_local_storage_nodes: 可设置单机启动elasticsearch的上限

# threadpool配置要满足文档，参考[2]
thread_pool.index.size <= 1 + # of available processors

# GC需要设置为CMSGC
-XX:+UseConcMarkSweepGC

# 程序启动时区设置可适当修改，否则默认为UTC
-Duser.timezone=GMT+8

# Xmx不要超过一半儿的物理内存，也不要超过32g，设置更多也没有意义，参考[3]
-Xms32g
-Xmx32g

# maximum map count至少262144
sysctl -w vm.max_map_count=262144

# ulimit设置
ulimit -l unlimited
ulimit -n 65536

# 另外plugins值得研究，参考[4]
```

**特别重要：elaticsearch在5.0之后不再支持root用户权限！**

---

#### Logstash

logstash是一个非常有趣的日志流处理工具，功能强大，plugins也丰富。

logstash主要配置文件主要有**logstash.yml,jvm.options,log4j2.properties**这三样。

另外需要单独配置一下input, filter, output过程，可以随意安装依赖的插件。

下面详解几个重要的配置：

```
path.config: 此项非常关键，是input, filter, output过程的配置文件地址，可以是多个文件，会依次被执行。

# GC需要设置为CMSGC
-XX:+UseConcMarkSweepGC

# 程序启动时区设置可适当修改，否则默认为UTC
-Duser.timezone=GMT+8

# 设置成一致，当然设置大致总归会快
-Xms4g
-Xmx4g

# memory会快很多
queue.type: memory
queue.max_bytes: 1024mb

# http.host:http.port可以查看logstash的相关信息
http.port: 9600

# 默认是hostname
node.name

# logstash用户可以设置自己的文件连接数
ulimit -n 16384

# 各种配置项，参考[5]

# 各种插件相当有用，参考[6]
```

可使用单独的logstash用户来启动logstash

---

#### Kibana

kibana只设置一个就可以，但必须连接elasticsearch的master节点或者coordinate节点。单点故障需要自己去保证。

kibana配置非常简单，下面详解几个重要的配置：

```
# 最重要的一项，url为master节点或者coordinate节点
elasticsearch.url: "http://${master_ip}:9200"

# 可查看kibana状态
http://${kibana_ip}:5601/status

# 其他配置参考[7]
```

---

#### docker启动命令

注意这个镜像，把节点配置项都打进image里了，使用话的，可自行修改参考[8]Dockerfile.

```
$ docker run -d  --net=host --privileged -p 5601:5601 -p 9200:9200 -p 9300:9300 -p 5044:5044 -p 9600:9600 -it --name elk \
    -v /data0/elk/elasticsearch/data:/var/lib/elasticsearch \
    -v /data0/elk/elasticsearch/log:/var/log/elasticsearch \
    -v /data0/elk/elasticsearch/tmp:/tmp/elasticsearch \
    -v /data0/elk/logstash/data:/var/lib/logstash \
    -v /data0/elk/logstash/log:/var/log/logstash \
    -v /data0/elk/logstash/tmp:/tmp/logstash \
    -v /data0/elk/kibana/log:/var/log/kibana \
    -v /data0/elk/kibana/tmp:/tmp/kibana \
    paladintyrion/elk
```

---

#### 参考文献

[1] [elasticsearch node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)  
[2] [elasticsearch threadpool](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)  
[3] [elasticsearch mem](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)  
[4] [elasticsearch plugins](https://www.elastic.co/guide/en/elasticsearch/plugins/5.6/index.html)  
[5] [logstash config](https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html)  
[6] [logstash plugins](https://www.elastic.co/guide/en/logstash/current/working-with-plugins.html)  
[7] [kibana config](https://www.elastic.co/guide/en/kibana/current/settings.html)  
[8] [elk Dockerfile](https://github.com/PaladinTyrion/elk-docker)