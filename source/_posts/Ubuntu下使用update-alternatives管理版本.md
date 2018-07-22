---
title: Ubuntu下使用update-alternatives管理版本
date: 2018-07-09 11:16:57
categories:
- 工具
tags:
- Python
---

更新Python3.7引出来的问题。 以前蠢兮兮自己维护软链
update-alternatives 就是系统自带用来解决多版本切换的

<!-- more -->

```shell
deploy@iZwz91qlpa6f1cbkdwsnsaZ:~$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.5 1
update-alternatives: using /usr/bin/python3.5 to provide /usr/bin/python3 (python3) in auto mode

deploy@iZwz91qlpa6f1cbkdwsnsaZ:~$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 2
update-alternatives: using /usr/bin/python3.6 to provide /usr/bin/python3 (python3) in auto mode

deploy@iZwz91qlpa6f1cbkdwsnsaZ:~$ sudo update-alternatives --config python3
There are 2 choices for the alternative python3 (providing /usr/bin/python3).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.6   2         auto mode
  1            /usr/bin/python3.5   1         manual mode
  2            /usr/bin/python3.6   2         manual mode

Press <enter> to keep the current choice[*], or type selection number: 2

deploy@iZwz91qlpa6f1cbkdwsnsaZ:~$ ls -al $(which python3)
lrwxrwxrwx 1 root root 25 Jul  9 11:14 /usr/bin/python3 -> /etc/alternatives/python3
deploy@iZwz91qlpa6f1cbkdwsnsaZ:~$ ls -al /etc/alternatives/python3
lrwxrwxrwx 1 root root 18 Jul  9 13:58 /etc/alternatives/python3 -> /usr/bin/python3.6
```

-------
References:
[How to Install Python 3.6.1 in Ubuntu 16.04 LTS](http://ubuntuhandbook.org/index.php/2017/07/install-python-3-6-1-in-ubuntu-16-04-lts/)