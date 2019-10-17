---
title: Aws-ssr
date: 2018-04-02 00:14:38
categories: tools
tags: 
  - ANTI-GFW
---

#### 主要工具

- AWS
- SSR
- Client

#### AWS

- AWS新用户可免费使用一年，每月流量15G，单节点流量每月1G.

- AWS免费配置教程参考：[AWS官网](https://aws.amazon.com/cn/free/)

<!-- more -->

#### SSR

##### For 4-version SSR

- 内存要求: >=128M

1. 一键安装Shadowsocks-Python, ShadowsocksR, Shadowsocks-Go, Shadowsocks-libev版(四选一)服务端;
2. 各版本的启动脚本及配置文件名不再重合;
3. 每次运行可安装一种版本;
4. 支持以多次运行来安装多个版本，且各个版本可以共存(注意端口号需设成不同);
5. 若已安装多个版本，则卸载时也需多次运行(每次卸载一种);
6. Shadowsocks-Python和ShadowsocksR安装后不可同时启动(因为本质上都属Python版).

- 默认配置:

1. 服务器端: 自己设定(如不设定，默认从9000-19999之间随机生成)
2. 密码: 自己设定(如不设定，默认为teddysun.com)
3. 加密方式: 自己设定(如不设定，Python和libev版默认为aes-256-gcm，R和Go版默认为aes-256-cfb)
4. 协议(protocol): 自己设定(如不设定，默认为origin) (仅限ShadowsocksR版)
5. 混淆(obfs): 自己设定(如不设定，默认为plain) (仅限ShadowsocksR版)

备注：脚本默认创建单用户配置文件，如需配置多用户，请手动修改相应的配置文件后重启即可。

- 其他需知：

libev版的Shadowsocks最新版本特点是内存占用小(600k左右)，使用libev和C编写，低CPU消耗.


###### **Server安装**

root用户下执行:

```
$ wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
$ chmod +x shadowsocks-all.sh
$ ./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

###### **Server卸载**

root用户下执行:

```
$ ./shadowsocks-all.sh uninstall
```

###### **启动脚本**

启动脚本后面的参数含义，从左至右依次为：启动，停止，重启，查看状态。

Shadowsocks-Python版:

```
$ /etc/init.d/shadowsocks-python start | stop | restart | status
```

ShadowsocksR版:

```
$ /etc/init.d/shadowsocks-r start | stop | restart | status
```

Shadowsocks-Go版:

```
$ /etc/init.d/shadowsocks-go start | stop | restart | status
```

Shadowsocks-libev版:

```
$ /etc/init.d/shadowsocks-libev start | stop | restart | status
```

###### **各版本默认配置文件**

Shadowsocks-Python版:

```
/etc/shadowsocks-python/config.json
```

ShadowsocksR版:

```
/etc/shadowsocks-r/config.json
```

Shadowsocks-Go版:

```
/etc/shadowsocks-go/config.json
```

Shadowsocks-libev版:

```
/etc/shadowsocks-libev/config.json
```

##### SSR配置文件

/etc/shadowsocks-\*/config.json需知:

- 同时支持IPv4与IPv6;
- 启动多端口支持;
- 多用户配置时注意: 多个端口从防火墙(iptables或firewalld)中打开.

具体配置:

```
{
    //"server":"0.0.0.0",
    //"server_ipv6":"[::]",
    "server":["[::0]","0.0.0.0"],
    //"server_port":your_server_port,
    "local_address":"127.0.0.1",
    "local_port":1080
    //"password":"your_password",
    "port_password":{
         "9000":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":600,
    "method":"aes-256-cfb",
    //"protocol": "origin",
    //"protocol_param": "",
    //"obfs": "plain",
    //"obfs_param": "",
    //"redirect": "",
    //"dns_ipv6": false,
    //"fast_open": false,
    //"fast_open": true,
    //"workers": 1
}
```

#### Client

在iOS端使用superWingy，mac端使用ShadowsocksX-NG，其他端参考如下.

客户端: [ss客户端](https://shadowsocks.org/en/download/clients.html)

#### SS-Over-Websocket

```
$ git clone https://github.com/VincentChanX/shadowsocks-over-websocket
$ cd shadowsocks-over-websocket
# 开启服务端
$ node server.js -m aes-256-cfb -k ${mypassword} -s 0.0.0.0 -p 9988 > /dev/null &
# 客户机开启客户端
$ node local.js -m aes-256-cfb -k ${mypassword} -s ${my-vps-ip} -p 9988 -l 1081 > /dev/null &
# 客户机器socks5代理服务器为127.0.0.1:1081
```

#### 参考文献
1. [shadowsocks](https://github.com/teddysun/shadowsocks_install)
2. [ss非官方blog](https://shadowsocks.be/)
3. [ss_wiki](https://github.com/iMeiji/shadowsocks_install/wiki)
4. [ss客户端](https://shadowsocks.org/en/download/clients.html)
5. [其他参考教程](http://kr1.crossfirewall.win:8000/archives.html)
6. [shadowsocks-over-websocket](http://www.xuxiaobo.com/?p=3051)
7. [特别牛逼参考Surge3规则](https://github.com/ConnersHua/Profiles/blob/master/README.md)
