---
title: '127.0.0.1, 127.0.1.1和 localhost'
date: 2017-03-07 20:37
categories:
  - 走在写码的路上
tags:
  - 假装是个DevOps
---

本文测试环境：
```shell
$ uname -a
Linux xxx 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

起因是修改服务器 `/etc/hostname` 和 `/etc/hosts`

同事坚持 A
```
127.0.0.1 localhost
127.0.1.1 <host_name>
```
而我试了一下 B
```
127.0.0.1 localhost <host_name>
```

<!-- more -->

明明也是没有问题的嘛。 随手查[文档](http://www.debian.org/doc/manuals/debian-reference/ch05.en.html#_the_hostname_resolution)。 注意这两句

>The IP address 127.0.1.1 in the second line of this example may not be found on some other Unix-like systems. The [Debian Installer](http://en.wikipedia.org/wiki/Debian-Installer) creates this entry for a system without a permanent IP address as a ***workaround*** for some software (e.g., GNOME) as documented in the [bug #719621](http://bugs.debian.org/719621).

>For a system with a permanent IP address, that permanent IP address should be used here instead of 127.0.1.1.


127.0.0.1 和 127.0.1.1 都属于 127.0.0.0/8  loopback地址， 本质没有区别。 和127.255.255.254 也没区别。

结论是，如果你需要一个单独的入口， 又没有真实的固定IP， 那就用A。 至于为什么默认A， 因为 GNOME.
如果有固定ip, `127.0.1.1` 替换成固定ip, 结束

同时也兼容各种getfqdn()不会get到localhost, [这里](https://onebitbug.me/2014/06/25/settings-fqdn-in-linux/)