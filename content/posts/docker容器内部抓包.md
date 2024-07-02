---
title: 'Docker容器内部抓包'
date: '2024-07-02T15:16:56+08:00'
draft: false
author: vdong
description: 'Docker容器内部抓包'
categories:
  - 技术
tags:
  - docker
  - 调试
  - 命名空间
---







很多docker容器为了轻量化，都不包含一些基础命令，如ip ，address，tcpdump 等，这给调试容器的网络带来了麻烦。

其实我们可以通过 命令进入容器的网络命名空间，使用宿主机的命令调试容器网络。

用法如下：

1. 查看 docker 容器的 pid

   ```shell
   docker inspect -f {{.State.Pid}} nginx
   ```

2. 进入容器的网络命名空间

   ```shell
   nsenter -n -t <上一步中的Pid>
   
   # 退出
   exit
   ```

3. 然后就可以使用 `ip` 或者 `tcpdump` 抓包了。