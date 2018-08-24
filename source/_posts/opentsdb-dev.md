---
title: opentsdb dev
date: 2018-04-27 01:16:34
categories: tsdb
type: "tags"
tags:
  - opentsdb
---

#### Opentsdb调试

##### 生成pom.xml文件

Github上clone源码.

```
# 项目路径下，生成pom.xml
$ ./build.sh && ./build.sh pom.xml
```

生成pom.xml文件后，有提示报错，需要修改若干位置.

###### ph-javacc-maven-plugin参数javadocFriendlyComments报错，可修改ph-javacc-maven-plugin版本为2.8.2、3.0.0等尝试

###### profile: bigtable不生效，需要修改bigtable的配置，也可以删除

<!-- more -->

```
<profile>
    <!-- Build for Google Bigtable backend -->
    <id>bigtable</id>
    <activation>
        <activeByDefault>false</activeByDefault>
    </activation>

    <dependencies>
        <dependency>
            <groupId>com.pythian.opentsdb</groupId>
            <artifactId>asyncbigtable</artifactId>
            <version>0.3.0</version>
            <classifier>jar-with-dependencies</classifier>
        </dependency>
    </dependencies>
</profile>
```

###### profile: cassandra不生效，需要修改cassandra的配置，也可以删除

```
<profile>
    <!-- Build for Apache Cassandra backend -->
    <id>cassandra</id>
    <activation>
        <activeByDefault>false</activeByDefault>
    </activation>

    <dependencies>
        <dependency>
            <groupId>net.opentsdb</groupId>
            <artifactId>asynccassandra</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <classifier>jar-with-dependencies</classifier>
        </dependency>
    </dependencies>
</profile>
```

其中asynccassandra在中央仓库没有发布，需要自己去下载项目打包生成. 另外asynccassandra项目打包时要关闭doclint，否则会报错，pom.xml修改如下部分.

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.8.1</version>
    <executions>
        <execution>
            <id>attach-javadocs</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
    <!-- 添加如下内容，跳过doclint检查 -->
    <configuration>
        <encoding>UTF-8</encoding>
        <charset>UTF-8</charset>
        <additionalparam>-Xdoclint:none</additionalparam>
    </configuration>
    <!-- 添加结束 -->
</plugin>
```

打包命令

```
$ mvn install
```

###### 修改maven-surefire-plugin参数设置，减少高版本JVM报警

```
<argLine>-Xmx2048m -XX:MaxPermSize=256m</argLine>

修改为:

<argLine>-Xmx2048m -XX:MaxMetaspaceSize=256m</argLine>
```

###### 修改maven-javadoc-plugin参数设置，防止javadoc编译报错

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.8.1</version>
    ...
    <configuration>
        ...
        <!-- 增加如下内容 -->
        <additionalparam>-Xdoclint:none</additionalparam>
        <!-- 增加完成 -->
    </configuration>
</plugin>
```

###### 打包时跳过gpg过程

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-gpg-plugin</artifactId>
    ...
    <!-- 增加如下内容 -->
    <configuration>
        <skip>true</skip>
    </configuration>
    <!-- 增加完成 -->
</plugin>
```

##### 调试启动

以上6步操作完成后，mvn package查看build是否通过。遇到问题按照如下方案解决

###### 首先通过 mvn compile 编译项目

###### 发现package tsd.client中有些duplicate classes，包名为tsd.client，所以net.opentsdb.tsd.client应当去掉.

解决：在ide中将src-main/net/opentsdb/tsd/client从项目中exclude

###### net.opentsdb.tools.TSDMain启动配置文件，是从项目根目录中读取配置文件

解决：将opentsdb.conf复制到根目录下(也可以在TSDMain的运行参数中通过--config=absolute-path-to-opentsdb-conf-file来指定配置文件位置)，并修改配置

```
tsd.network.port = 4242
tsd.http.staticroot = /to/the/path/of/opentsdb/target/opentsdb-2.3.0/queryui
tsd.http.cachedir = /tmp
tsd.core.auto_create_metrics = true
# 这里需要配置bhase服务地址
tsd.storage.hbase.zk_quorum = localhost
```

###### mygnuplot.sh缺失

mvn compile后是会产生mygnuplot.sh文件的.

解决：在项目根目录下创建**src-main-resources**文件夹，将mygnuplot.sh放入src-main-resources目录下.

##### Hbase启动

###### Hbase建议使用docker镜像

```
$ docker run -d -it --net=host --privileged -p 2181:2181 -p 8080:8080 -p 8085:8085 -p 9090:9090 -p 9095:9095 -p 16000:16000 -p 16010:16010 -p 16201:16201 -p 16301:16301 harisekhon/hbase
```

###### Hbase缺失表，使用src-main/create_table.sh去Hbase环境下创建表

```
Error：Caused by: com.stumbleupon.async.DeferredGroupException: At least one of the Deferreds failed, first exception: org.hbase.async.TableNotFoundException: "tsdb-uid"
```

解决方案：

```
# 可以单独用export设定这两个环境变量，COMPRESSION不关闭，会报错压缩错误
$ env COMPRESSION=NONE HBASE_HOME=/hbase ${project_src}/create_table.sh
```

##### TSDMain运行调试

访问localhost:4242，向opentsdb添加若干数据，若tsdb监控页不能显示图，需要安装gnuplot

```
# Mac下可直接安装
$ brew install gnuplot
```

#### 参考文献
1. [opentsdb导入intellij](http://echoyuan.me/2017/08/08/running-opentsdb-with-maven-and-intellij-idea/)
2. ["tsdb-uid"报错缺失](https://ask.csdn.net/questions/363978)