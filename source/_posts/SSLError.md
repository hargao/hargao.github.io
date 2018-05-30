---
title: SSLError
date: 2017-03-02 22:55:00
tags:
  - 假装是个DevOps
categories:
  - 走在写码的路上
---

不敢说自己懂了，记录遇到的问题和解决

接入华为推送遇到

```
SSLError(SSLError("bad handshake: Error([('SSL routines', SSL3_GET_SERVER_CERTIFICATE', 'certificate verify failed')],)",),)`
```

确定对方的证书没有问题， 所以问题定位在服务器的证书链上

最偷懒的办法当然是 `verify=False` 啦， 但是基本操守还是要有的


### openssl

```bash
openssl s_client -connect 'login.vmall.com:443' -showcerts

SSL-Session:
  ....
  Verify return code: 20 (unable to get local issuer certificate)

```

```
openssl s_client -CApath /etc/ssl/certs/ -connect 'login.vmall.com:443' -showcerts

SSL-Session:
  ....
  Verify return code: 0 (ok)
```

伟大的StackOverflows说， openssl 不默认信任任何证书， 需要由用户指定, 这就是不默认`/etc/ssl/certs/`为`CApath`的原因

### 解决方法 0.1
既然认为是CApath的原因， 那就显式的 `requests.post(.., verify="/etc/ssl/certs/")`, stage 环境测试通过， 奔走相告

### 高兴的太早了
上prod后，集中报 `CA directories not supported in older Pythons`, requests和python版本明明是一样的

### 走头无路查阅文档
得到以下信息:
>- python 2 (2.7.9 版本以下) 的 ssl 不安全, [看这里](https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings)
-  [解决办法](https://urllib3.readthedocs.io/en/latest/user-guide.html#ssl-py2) 是配合pyopenssl食用
- 如果安装了pyopenssl, `Requests` will then automatically inject pyopenssl into urllib3
- 从 Requests 2.4.0 版之后，如果系统中装了 `certifi` 包，`Requests` 会试图使用它里边的证书

 推测问题出在上面某个部分， 我相信这么常见的场景， requests不可能不支持。
那就重走一遍安装过程
```
pip install requests[security]
sudo apt-get update
sudo apt-get install libffi-dev libssl-dev
```

结论是是libssl-dev 没装好
也真是..坑

-------

真·END
