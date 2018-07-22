---
title: SSH端口转发
date: 2018-04-21 10:20:51
categories:
- 工具
tags:
- SSH
---

### 背景

`wormhole`: 跳板机别名, 在 config 中已配置好域名、端口、用户名和私钥; 又称为"代理", "SSH Server"
`10.0.70.20`: 内网服务器ip. 又称为"远程主机".
`10.0.70.21`: 内网数据库ip, 也是个"远程主机"
`hargao`: 用户名. 在跳板机和内网机器上都可以通过私钥登录

### 概念

```
    SSH Agent Forwarding: SSH Agent 转发
    Local Forward: 本地转发
```

<!-- more -->

### 参数

```
    -p ssh 端口, 根据实际情况设置. 不多废话. 默认22

    -T 禁止为ssh分配伪终端
    -t 强制分配伪终端, 重复使用该选项"-tt"将进一步强制. "伪终端的标准输入输出都是管道", 没有理解,
    转发时不强制分配会提示`Pseudo-terminal will not be allocated because stdin is not a terminal`,
    并且不能在终端输入

    -a 禁止转发认证代理的连接.
    -A 允许转发认证代理的连接. 可以在配置文件中对每个主机单独设定这个参数.

    -D 指定一个本地机器 “动态的” 应用程序端口转发. 本地机器上分配一个 socket 侦听端口, 转发这个端口的所有连接至跳板机

    -f fork 进程去后台执行

    -N 声明不执行远程命令. 仅用于转发端口

    -g 允许远端主机连接本地转发的端口."告诉SSH客户端，这个GatewayPorts=yes。即开启端口转发". 但我还是不懂

    -L 将本地端口转发到远程主机的端口
    -R 将远程主机端口转发到本地的指定端口
```

### 使用场景

1. 通过跳板机进入内网服务器

    ```shell
        ssh -A -t wormhole ssh hargao@10.0.70.20
    ```

    分析: SSH支持执行shell
    ```shell
        % ssh wormhole ls                                                                                                                                                                         ~
        test.txt
    ```
    所以实际上是在wormhole服务器上执行 `ssh hargao@10.0.70.20`
    -t 分配伪终端。 -A 解决代理转发,使用本机的密钥认证远程主机, 而不是当执行ssh到远程主机时在代理服务器上寻找hargao账户的密钥

2. 本地流量转发至跳板机, 访问内网服务
    ```shell
        ssh -D 18080 wormhole
    ```
    18080端口上的所有流量转发至wormhole. 用来配合 Proxy SwitchyOmega 访问内网服务

3. 本地访问内网数据库/有ip白名单限制的数据库

    ```shell
        # 都是后台运行将10.0.70.21:3306通过wormhole转发至本地3306端口.
        ssh -f wormhole -L 3306:10.0.70.21:3306 -N
        ssh -gNfL 3307:10.0.70.21:3306 wormhole

        # 但我更喜欢这样子, 前台运行, 可以执行命令, 方便管理
        ssh -L 3307:10.0.70.21:3306 wormhole

        # 连接数据库
        mysql -h 127.0.0.1 -P 3307
    ```

4. 远程转发

    "我"是内网的一台服务器("10.0.70.20"). 我能访问内网数据库、能访问外网但是没有外网ip
    有一个外网用户(host1), 想访问内网数据库("10.0.70.21:3306", host2). 但是无法主动发起连接至"10.0.70.20"
    我主动发起一个隧道， 让外网用户能访问内网数据库

    ```shell
        ssh -R 3307:10.0.70.21:3306 host1
        ssh -NfR 3307:10.0.70.21:3306 host1

        # 此时在host1上
        mysql 127.0.0.1 -P 3307
    ```

    好吧我没有使用场景, 没用过

### SSH config
```
    # 设置别名
    Host wormhole
        HostName 10.0.70.1
        Port 22
        User hargao
        IdentityFile ~/.ssh/id_rsa_hargao

    # 设置本地转发, 场景3
    Host  db
        HostName     wormhole 的 HostName
        Port         wormhole 的 Port
        User         wormhole 的 User
        LocalForward 3307 10.0.70.21:3306
```

-------
References:
[Debian man page](https://manpages.debian.org/stretch/manpages-zh/ssh.1.zh_CN.html)
[老戚的黑科技之SSH隧道技术](https://blog.csdn.net/yeyichao/article/details/51154312)