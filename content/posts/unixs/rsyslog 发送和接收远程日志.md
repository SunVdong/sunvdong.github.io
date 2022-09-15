+++
title = " rsyslog 发送和接收远程日志"
date = 2022-09-15T15:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['unix', 'linux', 'syslog']

+++

## 配置本机可以接收远端的syslog日志

根据需要启用udp或tcp模块，并设置端口，使其能够接收消息。

```shell
# vim /etc/rsyslog.conf
$ModLoad imudp
$UDPServerRun 514
```

重启 rsyslog 服务

```bash
systemctl restart rsyslog
```

防火墙放行端口

```bash
iptables -A INPUT -p udp --dport 514  -j ACCEPT
```

监听放行的端口

```bash
# -l listen -p source-port -4 ipv4 -u udp
nc -l -p 514 -4 -u
```

在远端的主机上发送日志测试

```shell
logger "hello logs" --server {{hostname}} --port 514
```

以上监听将会显示类似如下信息：

```shell
<5>Sep 15 15:36:00 root: hello logs
```

## 配置本机将syslog日志发送远端

修改配置文件

```shell
# 旧版本语法 

# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
#$ActionQueueFileName fwdRule1 # unique name prefix for spool files
#$ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
#$ActionQueueSaveOnShutdown on # save messages to disk on shutdown
#$ActionQueueType LinkedList   # run asynchronously
#$ActionResumeRetryCount -1    # infinite retries if host is down
# Facility.Severity *.* 表示所有  @表示传输协议（@表示udp，@@表示tcp），后面是ip和端口。
*.* @@remote-host:514

# 新版本语法
*.* action(type="omfwd" target="192.0.2.1" port="10514" protocol="tcp")
*.* action(type="omfwd" #使用 omfwd udp 和 tcp 转发插件
queue.type="LinkedList"  # 启用 LinkedList 内存队列，queue_type 可以是 direct、linkedlist 或者 fixedarray（它们是内存队列）或者磁盘
action.resumeRetryCount="-1" # 远程主机关闭时重试次数 -1 无限次
queue.size="10000" # 内存中队列的数量，非字节大小，防止峰值
queue.saveonshutdown="on" # 本地主机关闭时，是否保存内存中的队列到磁盘
target="10.43.138.1" Port="10514" Protocol="tcp")
```

重启 rsyslog 服务

```shell
systemctl restart rsyslog
```

对于 rsyslog 你也可以定义一个配置文件，重新启动一个新的进程。

```shell
rsyslogd -i /var/run/rsyslog_reindexer.pid -f /home/me/rsyslog_reindexer.conf
```

----------------

参考：

- [如何使用 tcp 和 udp 端口将 syslog 日志发送到远程服务器上](https://www.onitroad.com/jc/faq/how-to-send-log-messages-to-remote-server-using-tcp-and-udp-ports-in-red-hat-linux.html)
- [syslog 详解和配置](https://www.cnblogs.com/haimeng/p/10823699.html)
- [使用 rsyslog 重排 es 数据](https://www.rsyslog.com/using-rsyslog-to-reindexmigrate-elasticsearch-data/)
- [rsys文档](https://www.rsyslog.com/doc/v8-stable/configuration/modules/index.html)
- [man 手册](https://linux.die.net/man/8/rsyslogd)

