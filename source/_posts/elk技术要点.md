---
title: elk技术要点
date: 2017-11-20 11:50:50
categories: framework
type: "tags"
tags:
  - elk
---

### Architecture

- Elasticsearch Arch:

<!-- more -->

![Elasticsearch Arch](elasticsearch.jpeg)

- Logstash:

每个logstash都是独立的进程，每机可开多个logstash运行，logstash之间没有需要协同工作的状态，属于单机运行程序。

![Logstash RUN](logstash.jpeg)

- Kibana:

Kibana是用nodejs开发的web界面，其中如果开启Kibana，kibana会在elasticsearch中生成一个指定名配置的名的索引，因而如果多个kibana连接多台es_master机器，所图标等的保存也会被记录下来。

工作图：kibana => elasticsearch

- Elk Arch

![Arch](arch.jpeg)

机器限制，所以对elk的部署做了四件主要工作：

1. 将elk制作了imgae，为了支持多节点部署，需要相应名字的指定环境变量来传递集群成员的信息，启动时将修改elasticsearch和logstash配置，另外image启动之后均使用宿主机的ip

2. 限制了ES container运行机host的5601端口，防止脚本注入攻击，只开放了对若干ip的访问权限，另外单独部署了nginx-proxy负载均衡地访问kibana界面，加了登陆验证功能

3. 单独部署了多台机器的kafka集群，接收数据作为logstash的message输入端。kafka集群单独运维，单独调参，container可通过环境变量传入来进行启动设置。另外kafka也有单独的监控，制作了image。

4. 多台机器开启了nfs文件映射。kafka的数据和日志，elk的数据和日志，都通过卷映射到nfs server的相应目录。由于es不再支持root启动，nfs的映射采用匿名映射方式。

**Note**：nfs server在此套系统中比其他节点的负载率高出5左右

---

### Elasticsearch

配置项(摘了主要):

```
cluster.name: logging
node.name: ES_${HOSTNAME}
node.master: true
node.data: true
node.ingest: false
bootstrap.memory_lock: true
network.host: 0.0.0.0
discovery.zen.ping.unicast.hosts: ["0.0.0.0:9300"]
discovery.zen.minimum_master_nodes: 1
discovery.zen.ping_timeout: 20s

thread_pool:
    generic:
        keep_alive: 2m
    index:
        size: 9
        queue_size: 3000
```

http://${elasticsearch_ip}:9200/_cluster/health?pretty
可以读取集群状态

Elasticsearch不再支持root权限，需要使用自定义用户启动。

---

### Logstash

配置项(摘了主要):

```
node.name: "logstash"
pipeline.workers: 6
pipeline.batch.size: 1500
pipeline.batch.delay: 5  #ms
path.config: /etc/logstash/conf.d/
config.reload.automatic: true
queue.type: memory
http.host: "0.0.0.0"
```

```
input {
  kafka {
    bootstrap_servers => "0.0.0.0:9092"
    topics => ["servlogging"]
    auto_commit_interval_ms => "5000"
    session_timeout_ms => "15000"
  }
}

filter {
  ……
  date {
    match => [ "logtime", "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
  mutate {
    remove_field => [ "message" ]
  }
}

output {
  elasticsearch {
    hosts => ["0.0.0.0:9200"]
    action => "index"
    manage_template => false
    index => "servlogging-%{+YYYY-MM-dd}"
    document_type => "servlogging"
  }
}

```

---

### Kibana

配置项(摘了主要):

```
server.host: "0.0.0.0"
elasticsearch.url: "http://0.0.0.0:9200"
kibana.index: ".kibana"
elasticsearch.pingTimeout: 1500   #ms
elasticsearch.requestTimeout: 30000  #ms
```

http://${kibana_ip}:5601/status 可以读本机状态。

---

### Improvement

优化主要需要在五方面上下功夫：操作系统limit，内存，Cpu(为处理开多线程)，IO，JVM

```
## 操作系统limit
# elasticsearch
sysctl -w vm.max_map_count=262144
ulimit -l unlimited  #(locked memory)
ulimit -n 65536
# logstash
ulimit -n 16384
# kibana
没有

## 内存
# elasticsearch
-Xms10g
-Xmx10g
# logstash
-Xms4g
-Xmx4g
# kibana
留空1g内存左右，会缓存查询的数据记录，防止访问es过频繁

## Cpu
# elasticsearch
thread_pool.index.size <= available cores + 1
# logstash
pipeline.workers: 3  #为pipeline开多进程
input=>kafka:
auto_commit_interval_ms要设，保证断了重连可以offset继续读
reconnect_backoff_ms要设，防止超时read rebalance
client_id尽量设，可以保证多个logstash不会产生冲突，好查错
# kibana
界面配置

## JVM
# elasticsearch
-XX:+UseConcMarkSweepGC
-XX:+DisableExplicitGC  #禁掉系统调用gc
-Duser.timezone=GMT+8  #启动时需要加时区，es默认UTC标准时间
# logstash
同上设置
# kibana
没有要求，不走JVM

## 日志
使用rotate作业
保留5天日志，不压缩，触发bg压缩会耗费计算
关掉不需要的print，level高于warn再输出
```

---

### Usage

1. 参考官网： [elastic](https://www.elastic.co/guide/index.html)

2. 主要使用kibana和elasticsearch API

kibana作图：[kibana dashboard](https://www.elastic.co/guide/en/kibana/current/dashboard.html)

elasticsearch query：[elasticsearch query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html)

---

### References

[elastic官网](https://www.elastic.co/guide/index.html)  
[elk-docker](https://github.com/PaladinTyrion/elk-docker)

