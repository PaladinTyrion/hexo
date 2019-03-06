---
title: grpc
date: 2017-09-10 18:28:35
categories: rpc
tags: 
  - rpc
---

#### grpc入门

##### 安装grpc

- 官方标准安装：

```
 $ git clone -b $(curl -L http://grpc.io/release) https://github.com/grpc/grpc
 $ cd grpc
 $ git submodule update --init
 $ make
 $ [sudo] make install
```

<!-- more -->

- MacOS安装：

```
$ brew install --with-plugins grpc
```

- 以上两种安装，会生成如下可执行文件：

```
grpc_cli                 grpc_csharp_plugin       grpc_objective_c_plugin  grpc_python_plugin
grpc_cpp_plugin          grpc_node_plugin         grpc_php_plugin          grpc_ruby_plugin
```

- 补充go的plugin：

```
go get -u github.com/golang/protobuf/{protoc-gen-go,proto} // 前者是plugin;后者是go的依赖库
```

- 补充java的plugin(protoc-gen-grpc-java)的编译：

如果要编译Java工程依赖的jar包：

```
$ cd $GRPC_JAVA_ROOT/
$ ./gradlew build  #在对应的文件夹(all, netty, okhttp, protobuf等)的build文件夹里找到生成的lib
# 如果想把编译好的jar添加到你的本地库，以方便后续工程的依赖引用
$ ./gradlew install
```

如果只想编译使用codegen插件：

```
$ git clone https://github.com/grpc/grpc-java.git

# 进入grpc-java的工程根目录:
$ cd $GRPC_JAVA_ROOT/compiler

# 编译插件
$ ../gradlew java_pluginExecutable

# 测试插件是否ok
$ ../gradlew test
```

上述成功，则插件成功生成，可以在$GRPC_JAVA_ROOT/compiler/build/exe/java_plugin中看到"protoc-gen-grpc-java"。

另外，如果想把编译好的codegen插件添加到你的本地库，以方便后续工程的依赖引用，可以执行以下命令：

```
$ cd $GRPC_JAVA_ROOT/compiler
$ ../gradlew install
```

- grpc编译

```
#编译仍然使用protoc

#对于官方标准安装提供的plugin，还需要加如下参数
--plugin=grpc_$language_plugin
或者
--plugin=protoc-gen-grpc=$(which grpc_$language_plugin)

--$language_out=/to/the/output/path
或者
--grpc-$language_out=/to/the/output/path

#对java
--proto_path=/src/path
--plugin=protoc-gen-grpc-java=/path/to/protoc-gen-grpc-java
--grpc-java_out=/to/the/output/path

#对golang
#安装go plugin后只要填写go_out可自动调用protoc-gen-go来生成 xxxx.pb.go
--go_out=plugins=grpc:/to/the/output/path
或者
--go_out=/to/the/output/path
```

- grpc-java的maven编译方式

Java项目的pom.xml添加如下内容：

```
<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.5.0.Final</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.5.0</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:3.3.0:exe:${os.detected.classifier}</protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.4.0:exe:${os.detected.classifier}</pluginArtifact>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

当mvn编译工程时，使用如下命令：

```
$ mvn protobuf:compile-custom
```

在生成的target/generated-resource/protobuf目录下，分别生成的java-grpc和java文件夹，分别存放生成的grpc代码以及protobuf生成代码。

##### 参考文献
1. [protobuf-maven-plugin](https://www.xolstice.org/protobuf-maven-plugin/)
2. [grpc-java](https://github.com/grpc/grpc-java/)
