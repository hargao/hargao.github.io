---
title: ctypes
tags:
---
> ctypes与系统编程
    ctypes作为一种轻量并且内置的c语言“代理”，使得python极大地增强了系统编程的能力。
    从此，系统编程的代码也可以变得更加优雅。
    真实场景：
        sdn/vpc方案需要对内核协议栈做较多的调整，从管理的层面上，网络配置由中央控制并下发。因此，host上存在一个daemon，一方面要接受zookeeper的配置变更通知，另一方面要把配置解析后通过netlink与内核通信。
        这个daemon大概几乎没有人会用python去做。但是我看到iotop里用到ctypes对netlink接口的封装，惊为天人，并且python更加适合对配置解析与处理。我斗胆用python实现了这个daemon，调试起来如丝般顺滑，然后就减少了好几个月的加班。

> cffi更好用，还通用 pypy

> 我原来在公司的时候也是使用ctypes接口解决python加解密慢的问题,确实好使.只不过ctypes和C++的兼容性并不好!