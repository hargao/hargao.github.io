---
title: 流量代理
tags:
  - Mac
  - proxychains
categories:
  - 工具
date: 2018-08-10 23:22:28
---

公司墙了网易云音乐. 最简单的方式是换QQ音乐. 可是懒
以前装过proxychains下载youtube视频, 就想试试能不能用起来

<!-- more -->

### proxychains

```shell
// 失败
% proxychains4 open /Applications/NeteaseMusic.app

// 看了 proxychains issue 之后, 失败, 报错couldnt find configuration file: No such file or directory
% proxychains4 /Applications/NeteaseMusic.app/Contents/MacOS/NeteaseMusic

// proxychains 配置文件复制一份放在沙盒下。 启动成功, 但是没有成功代理
% proxychains4 -f ~/Library/Containers/com.netease.163music/Data/proxychains.conf /Applications/NeteaseMusic.app/Contents/MacOS/NeteaseMusic

// curl 测试, 配置是成功的
% proxychains4 /usr/local/Cellar/curl/7.61.0/bin/curl ip.cn
[proxychains] config file found: /usr/local/etc/proxychains.conf
[proxychains] preloading /usr/local/Cellar/proxychains-ng/4.13/lib/libproxychains4.dylib
[proxychains] DLL init: proxychains-ng 4.13
[proxychains] Strict chain  ...  127.0.0.1:1086  ...  ip.cn:80  ...  OK
当前 IP：42.200.116.102 来自：香港特别行政区 香港电讯
```

鉴于这个[issue](https://github.com/rofl0r/proxychains-ng/issues/181), 选择放弃.
而且面临的下一个问题是, 得把网易云流量代理到境内(海外有版权问题), 其他流量代理到境外. 起两个ss?

### PAC

新思路是, ss使用PAC模式. 把网易云的流量代理到阿里ECS上去.

#### 编辑gfwlist.js

编辑 `FindProxyForURL`
把 网易音乐 流量打到 127.0.0.1:9000

```javascript
function FindProxyForURL(url, host) {
  var netease_url = ['music.163.com','p1.music.126.net','p2.music.126.net','p3.music.126.net', 'p4.music.126.net']
  for (url in netease_url){
    if (host.indexOf(url) > -1) {
      return  'SOCKS5 127.0.0.1:9000; DIRECT;';
    }
  }

  if (defaultMatcher.matchesAny(url, host) instanceof BlockingFilter) {
    return proxy;
  }
  return direct;
}
```

#### 在ECS上搭ss

在控制台编辑规则开放端口的时候需要验证手机号. 留的旧手机, 找不着了

#### ssh 开个转发

```
ssh -D 9000 aliyun
```
看起来可以了。 周一回公司测试

-------
References:
[OS X, El Capitan, Proxychains以及强制app使用透明socks代理](https://zhuanlan.zhihu.com/p/20284019)
[Mac applications ignore proxychains after successful preloading](https://github.com/rofl0r/proxychains-ng/issues/181)