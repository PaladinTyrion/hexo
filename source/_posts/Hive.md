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
        <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
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
        <value>123456</value>
        <description>password to use against metastore database</description>
    </property>
    <property>
        <name>datanucleus.autoCreateSchema</name>
        <value>true</value>
    </property>
    <property>
        <name>datanucleus.autoCreateTables</name>
        <value>true</value>
    </property>
    <property>
        <name>datanucleus.autoCreateColumns</name>
        <value>true</value>
    </property>
    <!-- 设置 hive仓库的HDFS上的位置 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/hive</value>
        <description>location of default database for the warehouse</description>
    </property>
    <!--资源临时文件存放位置 -->
    <property>
        <name>hive.downloaded.resources.dir</name>
        <value>/to/the/path/hivedata/tmp/resources</value>
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
        <value>/to/the/path/hivedata/tmp/HiveJobsLog</value>
        <description>Local scratch space for Hive jobs</description>
    </property>
    <property>
        <name>hive.downloaded.resources.dir</name>
        <value>/to/the/path/hivedata/tmp/ResourcesLog</value>
        <description>Temporary local directory for added resources in the remote file system.</description>
    </property>
    <property>
        <name>hive.querylog.location</name>
        <value>/to/the/path/hivedata/tmp/HiveRunLog</value>
        <description>Location of Hive run time structured log file</description>
    </property>
    <property>
        <name>hive.server2.logging.operation.log.location</name>
        <value>/to/the/path/hivedata/tmp/OpertitionLog</value>
        <description>Top level directory where operation tmp are stored if logging functionality is enabled</description>
    </property>
    <!-- 配置HWI接口 -->
    <property>
        <name>hive.hwi.war.file</name>
        <value>/to/the/path/hiveInstalled/lib/hive-hwi-2.1.0.jar</value>
        <description>This sets the path to the HWI war file, relative to ${HIVE_HOME}. </description>
    </property>
    <property>
        <name>hive.hwi.listen.host</name>
        <value>hostname</value>
        <description>This is the host address the Hive Web Interface will listen on</description>
    </property>
    <property>
        <name>hive.hwi.listen.port</name>
        <value>9999</value>
        <description>This is the port the Hive Web Interface will listen on</description>
    </property>
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hostname</value>
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
    <!-- HiveServer2的WEB UI -->
    <property>
        <name>hive.server2.webui.host</name>
        <value>hostname</value>
    </property>
    <property>
        <name>hive.server2.webui.port</name>
        <value>10002</value>
    </property>
    <property>
        <name>hive.scratch.dir.permission</name>
        <value>755</value>
    </property>
    <!-- 下面hive.aux.jars.path这个属性里面你这个jar包地址如果是本地的记住前面要加file://不然找不到, 而且会报org.apache.hadoop.hive.contrib.serde2.RegexSerDe错误 -->
    <property>
        <name>hive.aux.jars.path</name>
        <value>file:///to/the/path/sparkInstalled/lib/spark-assembly-2.0.1-hadoop2.7.2.jar</value>
    </property>
    <property>
        <name>hive.server2.enable.doAs</name>
        <value>false</value>
    </property>
    <property>
        <name>hive.auto.convert.join</name>
        <value>false</value>
    </property>
</configuration>
```

初始化元数据存储:

```
$ schematool --dbType mysql --initSchema
```

##### 注意：

- Hiveserver1和Hiveserver2的JDBC区别：

HiveServer version |  Connection URL | Driver Class
---|---|---
HiveServer2 | jdbc:hive2:// | org.apache.hive.jdbc.HiveDriver
HiveServer1 | jdbc:hive:// | org.apache.hadoop.hive.jdbc.HiveDriver

- JDBC连接前需要服务器开启hiveserver/hiveserver2服务才支持JDBC连接

```
$ nohup hive --service hiveserver2 -p 10002 > /dev/null 2>&1
```

- 遇到的问题，有时候jdbc连接问题:
    1. 首先检查hive机是否已开启hiveserver服务
    2. 检查hiveserver端口是否被占用(默认为10000，经常被其他进程占用)
    3. jar包版本是否对应正确
    4. Driver Class是否对应hive版本正确
    
- JDBC连接hive后，所有执行的命令相当于在hive机直接执行，特别注意文件导入hive表操作，文件应当在hive机

- 当Hive是EXTERNAL建表时候，命令执行后的meta信息保存在hive-site.xml指定的mysql或者derby中，数据保存在建表语句location指定的hdfs地址中。EXTERNAL建表无法truncate，但可以drop，drop会删除meta信息，但仍然不会删除hdfs中的内容，hdfs的内容仍需手动hadoop命令删除

- 参考资料:
    1. http://blog.csdn.net/gamer_gyt/article/details/52062460
    2. http://blog.csdn.net/sunnyyoona/article/details/51648871
