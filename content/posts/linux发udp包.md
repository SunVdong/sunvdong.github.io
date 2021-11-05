+++
title = "linux发udp包"
date = 2021-11-04T19:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术'] 
tags = ['linux']
+++

Linux 直接发送UDP包


如果往本地UDP端口發送數據，那麼可以使用以下命令：


```bash
echo "hello" > /dev/udp/192.168.1.81/5060
```
​

意思是往本地192.168.1.81的5060端口發送數據包hello。


如果往遠程UDP端口發送數據，那麼可以使用以下命令：

```bash
echo "hello" | socat - udp4-datagram:192.168.1.80:5060
```


意思是往遠程192.168.1.80的5060端口發送數據包 hello 。
​

ps: 先安装 socat ，centos: `yum install -y socat`​
