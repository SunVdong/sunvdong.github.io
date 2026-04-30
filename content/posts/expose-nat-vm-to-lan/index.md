---
title: 'Expose Nat Vm to Lan'
date: '2026-04-30T17:52:05+08:00'
draft: false
author: vdong
description: ''
categories:
  - 技术
tags:
  - 折腾
  - 端口转发
  - 内网穿透
---


## 场景

局域网（10.2.15.x）

​        ├── 电脑A（你）10.2.15.239

​        │      └── 虚拟机 192.168.48.128（NAT 私网）

​        └── 电脑B（同事）10.2.15.200

现在需要 电脑B 可以 访问 虚拟机的服务

## 解决方案

方法1,  虚拟机改成桥接，配置一个 10.2.15.x 网段的 ip

方法2, 通过 电脑A 转发

bash

```bash
iptables -t nat -A PREROUTING -p tcp --dport 5678 -j DNAT --to-destination 192.168.48.128:5678
```

bash

```bash
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=5678 connectaddress=192.168.48.128 connectport=5678
```

结果是： 电脑B → [http://10.2.15.239:5678](http://10.2.15.239:5678/) → 转发 → 虚拟机
