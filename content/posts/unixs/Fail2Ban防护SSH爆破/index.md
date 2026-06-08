+++
title = "Fail2Ban防护SSH爆破"
date = 2023-11-27T11:10:25+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['unix', 'fail2ban']

+++

今天发现早上登录服务器，查看日志有大量ssh登录的爆破，所以先限制root用户不允许远程登录，再使用Fail2Ban自动封锁ip限制一下。

记录如下：

## 限制root用户不允许远程登录

```shell
vi /etc/ssh/sshd_config
```

修改 `PermitRootLogin yes` 为   `PermitRootLogin no` 。

然后重启 ssh 服务

```shell
systemctl restart ssh
```

## 使用Fail2Ban自动封锁ip

```shell
// install
yum install fail2ban -y
```

修改配置文件

```shell
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

vi /etc/fail2ban/jail.local
```

根据需要进行配置：

- `ignoreip`: 允许指定一组 IP 地址，这些地址不会被 `fail2ban` 封锁。
- `bantime`: 定义 IP 地址被封锁的时间（秒）。
- `maxretry`: 指定在多少次失败尝试后触发封锁。
- `findtime`: 定义检查失败尝试的时间窗口。
- `backend`: 设置 `fail2ban` 用于封锁 IP 的后端（例如 iptables）。

须在 `ssh` 选项下增加 `enabled=true` 如下：

```shell
[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
enabled = true
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

设置开机自动，同时现在启动

```shell
systemctl enable --now fail2ban.service
```

常用命令：

```shell
# 查看状态 serviceName 对应配置文件的中 [sshd] 选项
fail2ban-client status <serviceName>

# 手动 ban ip
fail2ban-client set sshd banip 222.186.16.210

# 手动删除被 ban 的 ip
fail2ban-client set sshd delignoreip 222.186.16.210

# 查看日志
tail /var/log/fail2ban.log
```

ps：当然也可以限制只允许密钥登录。

参考：

[centos8与Fail2Ban联合使用](https://www.sujx.net/2021/10/29/fail2ban/index.html)

[ssh只允许密钥登录](http://blog.51yip.com/linux/1838.html)

[fail2ban防御ddos攻击](https://www.kpromise.top/use-fail2ban-against-ddos-attack/amp/)