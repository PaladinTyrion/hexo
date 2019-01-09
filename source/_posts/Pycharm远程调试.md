---
title: Pycharm远程调试
date: 2019-01-09 17:38:30
categories: IDE
type: "tags"
tags:
    - pycharm
---

### pycharm配置远程部署

Preferences -> Build, Execution, Deployment -> Deployment:

点击"+"新添加一个配置项：

```
# Connection
Type: SFTP
Host: 10.85.10.16
Port: 22
User Name: paladintyrion
Authentication: Key pair   #走密钥验证
Private Key Path: /Users/paladintyrion/.ssh/id_rsa
Root Path: /

# Mappings: 可以添加多个Mapping
Local Path: /Users/paladintyrion/PycharmProjects/test
Deployment Path: /data1/remotePycharm    #远程服务器对应路径
```

<!-- more -->

自动生成配置用户名: paladintyrion@10.85.10.16:22

### 配置远程调试Interpreter

Preferences -> Project: xxxx -> Project Interpreter

在Project Interpreter最右选择Add，增加一条SSH Interpreter

```
#添加类似上述配置配置，最后页配置如下:
Interpreter: /usr/bin/python3    #10.85.10.16下的python_bin
Sync folders: <Project root>->/data1/remotePycharm
```

### 原理

上述配置只配置远程调试Interpreter即可。运行代码时，本地代码会首先同步到remote path，然后启动配置的remote node的python interpreter去运行相对应代码。前提是需要remote node已经pip3 install了相应的执行包。

### 注意

这种远程调试不适合cython的开发调试，最好直接配置远程机编译并运行调试。

### 参考文献

[Pycharm远程调试](https://blog.csdn.net/five3/article/details/78615589)