---
title: Hive
date: 2017-09-10 18:24:14
categories: data
tags: 
  - hive
---

#### Hive小结

##### 首先需要搭建Hadoop，为了Hive能够使用HDFS，过程省略

##### Hive的搭建过程

```
$ tar -zxvf hive-x.y.z.tar.gz
```

设置环境变量：

```
export HIVE_HOME=/to/the/hive/path
export PATH=$HIVE_HOME/bin:$PATH
```

Hive配置：

- hive-default.xml使用默认模板即可
- hive-env.sh使用默认模板，其中修改一下HADOOP_HOME的目录
- hive-log4j2.properties使用默认模板即可
- 重点配置hive-site.xml（注释很详细，不再赘述）

<!-- more -->

```
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://0.0.0.0:3396/hive?createDatabaseIfNotExist=true&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
        <description>JDBC connect string for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
        <description>Driver class name for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
        <description>username to use against metastore database</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>tyrion</value>
        <description>password to use against metastore database</description>
    </property>
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>true</value>
    </property>
    <!-- 设置 hive仓库的HDFS上的位置 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
        <description>location of default database for the warehouse</description>
    </property>
    <!--资源临时文件存放位置 -->
    <property>
        <name>hive.downloaded.resources.dir</name>
        <value>${system:java.io.tmpdir}/${hive.session.id}_resources</value>
        <description>Temporary local directory for added resources in the remote file system.</description>
     </property>
     <!-- Hive在0.9版本之前需要设置hive.exec.dynamic.partition为true, Hive在0.9版本之后默认为true -->
    <property>
        <name>hive.exec.dynamic.partition</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.exec.dynamic.partition.mode</name>
        <value>nonstrict</value>
     </property>
    <!-- 修改日志位置 -->
    <property>
        <name>hive.exec.local.scratchdir</name>
        <value>${system:java.io.tmpdir}/${system:user.name}</value>
        <description>Local scratch space for Hive jobs</description>
    </property>
    <property>
        <name>hive.exec.scratchdir</name>
        <value>/tmp/hive</value>
        <description>HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: ${hive.exec.scratchdir}/&lt;username&gt; is created, with ${hive.scratch.dir.permission}.</description>
    </property>
    <property>
        <name>hive.querylog.location</name>
        <value>${system:java.io.tmpdir}/${system:user.name}</value>
        <description>Location of Hive run time structured log file</description>
    </property>
    <property>
        <name>hive.server2.logging.operation.log.location</name>
        <value>${system:java.io.tmpdir}/${system:user.name}/operation_logs</value>
        <description>Top level directory where operation tmp are stored if logging functionality is enabled</description>
    </property>
    <!-- 配置server2.thrift -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>0.0.0.0</value>
    </property>
    <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
    </property>
    <property>
        <name>hive.server2.thrift.http.port</name>
        <value>10001</value>
    </property>
    <property>
        <name>hive.server2.thrift.http.path</name>
        <value>cliservice</value>
    </property>
    <property>
    	<name>hive.metastore.uris</name>
    	<value>thrift://0.0.0.0:9083</value>
    	<description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>
    </property>
    <!-- HiveServer2的WEB UI -->
    <property>
        <name>hive.server2.webui.host</name>
        <value>0.0.0.0</value>
    </property>
    <property>
        <name>hive.server2.webui.port</name>
        <value>10002</value>
    </property>
    <property>
    	<name>hive.server2.webui.max.threads</name>
    	<value>30</value>
    	<description>The max HiveServer2 WebUI threads</description>
    </property>
    <property>
        <name>hive.scratch.dir.permission</name>
        <value>755</value>
    </property>
    <!-- doAs为true表示以登录hiveserver2的用户身份执行hadoop job -->
    <!-- doAs为false表示以启动hiveserver2进程的用户身份去执行hadoop job -->
    <property>
        <name>hive.server2.enable.doAs</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.auto.convert.join</name>
        <value>true</value>
    </property>
    <property>
    	<name>hive.optimize.dynamic.partition.hashjoin</name>
    	<value>true</value>
    	<description>Whether to enable dynamically partitioned hash join optimization.
      		This setting is also dependent on enabling hive.auto.convert.join
    	</description>
    </property>
    <property>
    	<name>hive.vectorized.execution.mapjoin.native.fast.hashtable.enabled</name>
    	<value>true</value>
    	<description>
      	    This flag should be set to true to enable use of native fast vector map join hash tables in
      	    queries using MapJoin.
    	</description>
    </property>
    <property>
        <name>hive.execution.engine</name>
        <value>tez</value>
        <description>
            Expects one of [mr, tez, spark].
            Chooses execution engine. Options are: mr (Map reduce, default), tez, spark. While MR
            remains the default engine for historical reasons, it is itself a historical engine
            and is deprecated in Hive 2 line. It may be removed without further warning.
        </description>
    </property>
    <property>
        <name>hive.exec.parallel</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.tez.auto.reducer.parallelism</name>
        <value>true</value>
    </property>
    <property>
        <name>tez.lib.uris</name>
        <value>hdfs://hadoop1:9000/user/tez/tez.tar.gz</value>
    </property>
    <property>
        <name>tez.am.resource.memory.mb</name>
        <value>4096</value>
    </property>
    <property>
        <name>hive.tez.container.max.java.heap.fraction</name>
        <value>0.9</value>
        <description>This is to override the tez setting with the same name</description>
    </property>
    <property>
        <name>tez.am.resource.cpu.vcores</name>
        <value>4</value>
    </property>
</configuration>
```

初始化元数据存储:

用mysql会遇到执行失败，一般报错是JDBC组件not found。解决方案是自己去下个mysql-connector-java-8.0.15.jar之类的mysql JDBC jar包放到$HIVE_HOME/lib里。

```
$ schematool --dbType mysql --initSchema
```

##### 注意：

- Mysql中要创建相应配置的JDBC连接用户(如hive用户，也可以使用root用户)

```
CREATE USER 'hive'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%' WITH GRANT OPTION;
flush privileges;
```

- Hiveserver1和Hiveserver2的JDBC区别：

HiveServer version |  Connection URL | Driver Class
---|---|---
HiveServer2 | jdbc:hive2:// | org.apache.hive.jdbc.HiveDriver
HiveServer1 | jdbc:hive:// | org.apache.hadoop.hive.jdbc.HiveDriver

- JDBC连接前需要服务器开启hiveserver/hiveserver2服务才支持JDBC连接

```
$ nohup hive --service hiveserver2 >> hiveserv.log 2>&1 &
$ nohup hive --service metastore >> hivemeta.log 2>&1 &
# 在本机访问时不填写host:port,否则应为jdbc:hive2://$ip:10000
# 远程访问时，需要hadoop core-site中配置过hadoop.proxyuser.xxx.hosts/groups项。
$ beeline -u jdbc:hive2://  #beeline -u jdbc:hive2://10.10.10.10:10000 -n hive/root
```

- 遇到的问题，有时候jdbc连接问题:
    1. 首先检查hive机是否已开启hiveserver服务
    2. 检查hiveserver端口是否被占用(默认为10000，经常被其他进程占用)
    3. jar包版本是否对应正确
    4. Driver Class是否对应hive版本正确
    
- JDBC连接hive后，所有执行的命令相当于在hive机直接执行，特别注意文件导入hive表操作，文件应当在hive机

- 当Hive是EXTERNAL建表时候，命令执行后的meta信息保存在hive-site.xml指定的mysql或者derby中，数据保存在建表语句location指定的hdfs地址中。EXTERNAL建表无法truncate，但可以drop，drop会删除meta信息，但仍然不会删除hdfs中的内容，hdfs的内容仍需手动hadoop命令删除

- metastore的使用方法

metastore可以meta server机器起，远程连接meta服务。

假设server1已经通过**hive --service metastore -p 9803**启动了metastore服务，server2的hive-site.xml只需配置如下，即可远程连接server1的metastore服务。当mysql+metastore为远程服务时可这样配置。

```
<configuration>
    <property>
       <name>hive.metastore.uris</name>
       <value>thrift://server1:9083</value><!-- 此处是服务器1的ip -->
    </property>
</configuration>
```

- 参考资料:
    1. [hiveserver/hiveserver2的比较](http://blog.csdn.net/gamer_gyt/article/details/52062460)
    2. [Hive踩过的坑](http://blog.csdn.net/sunnyyoona/article/details/51648871)
    3. [Hive on Tez](https://blog.csdn.net/duguyiren3476/article/details/46349177)
    4. [Hive on tez](https://blog.csdn.net/sinat_37690778/article/details/80594571)
    5. [Tez优化参考](https://www.cnblogs.com/mobiwangyue/p/8405780.html)
--- 

- 附录资料：

Mysql神器脚本：

```
-- mysql.fix.utf8.sql
-- 先use相应的 database。执行失败的地方略过继续往下执行
alter database hive default character set utf8;
alter table BUCKETING_COLS default character set utf8;
alter table CDS default character set utf8;
alter table COLUMNS_V2 default character set utf8;
alter table DATABASE_PARAMS default character set utf8;
alter table DBS default character set utf8;
alter table FUNCS default character set utf8;
alter table FUNC_RU default character set utf8;
alter table GLOBAL_PRIVS default character set utf8;
alter table PARTITIONS default character set utf8;
alter table PARTITION_KEYS default character set utf8;
alter table PARTITION_KEY_VALS default character set utf8;
alter table PARTITION_PARAMS default character set utf8;
-- alter table PART_COL_STATS default character set utf8;
alter table ROLES default character set utf8;
alter table SDS default character set utf8;
alter table SD_PARAMS default character set utf8;
alter table SEQUENCE_TABLE default character set utf8;
alter table SERDES default character set utf8;
alter table SERDE_PARAMS default character set utf8;
alter table SKEWED_COL_NAMES default character set utf8;
alter table SKEWED_COL_VALUE_LOC_MAP default character set utf8;
alter table SKEWED_STRING_LIST default character set utf8;
alter table SKEWED_STRING_LIST_VALUES default character set utf8;
alter table SKEWED_VALUES default character set utf8;
alter table SORT_COLS default character set utf8;
alter table TABLE_PARAMS default character set utf8;
alter table TAB_COL_STATS default character set utf8;
alter table TBLS default character set utf8;
alter table VERSION default character set utf8;
alter table BUCKETING_COLS convert to character set utf8;
alter table CDS convert to character set utf8;
alter table COLUMNS_V2 convert to character set utf8;
alter table DATABASE_PARAMS convert to character set utf8;
alter table DBS convert to character set utf8;
alter table FUNCS convert to character set utf8;
alter table FUNC_RU convert to character set utf8;
alter table GLOBAL_PRIVS convert to character set utf8;
alter table PARTITIONS convert to character set utf8;
alter table PARTITION_KEYS convert to character set utf8;
alter table PARTITION_KEY_VALS convert to character set utf8;
alter table PARTITION_PARAMS convert to character set utf8;
-- alter table PART_COL_STATS convert to character set utf8;
alter table ROLES convert to character set utf8;
alter table SDS convert to character set utf8;
alter table SD_PARAMS convert to character set utf8;
alter table SEQUENCE_TABLE convert to character set utf8;
alter table SERDES convert to character set utf8;
alter table SERDE_PARAMS convert to character set utf8;
alter table SKEWED_COL_NAMES convert to character set utf8;
alter table SKEWED_COL_VALUE_LOC_MAP convert to character set utf8;
alter table SKEWED_STRING_LIST convert to character set utf8;
alter table SKEWED_STRING_LIST_VALUES convert to character set utf8;
alter table SKEWED_VALUES convert to character set utf8;
alter table SORT_COLS convert to character set utf8;
alter table TABLE_PARAMS convert to character set utf8;
alter table TAB_COL_STATS convert to character set utf8;
alter table TBLS convert to character set utf8;
alter table VERSION convert to character set utf8;
-- alter table PART_COL_STATS convert to character set utf8;
SET character_set_client = utf8 ;
-- SET character_set_connection = utf8 ;
-- alter table PART_COL_STATS convert to character set utf8;
SET character_set_database = utf8 ;
SET character_set_results = utf8 ;
SET character_set_server = utf8 ;
-- SET collation_connection = utf8 ;
-- SET collation_database = utf8 ;
-- SET collation_server = utf8 ;
SET NAMES 'utf8';
SET NAMES 'utf8';
```