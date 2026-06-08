+++
title = "Postgresql15 的安装和初始化"
date = 2022-10-30T20:11:52+08:00
draft = true
author = 'vdong'
categories = ['技术']
tags = ['数据库', 'postgresql']

+++

## 安装

1. 查看系统环境

   ```shell
    cat /etc/os-release  # 或 lsb_release -a
   
    # VERSION="3 (Soaring Falcon)"
    # ID="alinux"
    # ID_LIKE="rhel fedora centos anolis"
    # VERSION_ID="3"
    # PLATFORM_ID="platform:al8"
    # PRETTY_NAME="Alibaba Cloud Linux 3 (Soaring Falcon)"
    # ANSI_COLOR="0;31"
    # HOME_URL="https://www.aliyun.com/"
   ```

2. [进入官网](https://www.postgresql.org/)，下载页，选择合适的系统版本

   阿里云的 `Alibaba Cloud Linux 3` 相当于 `centos 8`，故选择 `Red Hat Enterprise, Rocky, or Oracle version 8`  平台，命令如下。 

   ```shell
   # 1.Install the repository RPM:
   sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   
   # 2. Disable the built-in PostgreSQL module:
   sudo dnf -qy module disable postgresql
   
   # 3. Install PostgreSQL:
   sudo dnf install -y postgresql15-server
   
   # 4. Optionally initialize the database and enable automatic start:
   sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
   sudo systemctl enable postgresql-15
   sudo systemctl start postgresql-15
   ```

   第2步，会出现 `Failed to download metadata for repo 'pgdg-common'` 错误，原因是：

   > 第三方DNF源仅适配CentOS 8发行版，而在Alibaba Cloud Linux 3中，系统的releasever值与CentOS 8不同，导致DNF解析后的地址无效，从而下载RPM包失败。

   解决方案-1：

   `vim /etc/yum.repos.d/pgdg-redhat-all.repo`

   全局替换 `$releasever` 为 `8 `,  vim 命令为：`%s/$releasever/8/g`

   继续执行剩余步骤。
   
   解决方案-2：
   
   ```shell
   cd /etc/yum.repos.d # 找到postgres的.repo, 复制文件名称
   
   vim /etc/yum/pluginconf.d/releasever-adapter.conf 
   
   # 如果没有pluginconf.d 使用下面命令
   dnf install dnf-plugin-releasever-adapter --repo alinux3-plus
   
   # 将上面复制的repo文件名添加到include后面，用 `,` 隔开
   [main]
   enabled=1
   
   [releasevermapping]
   release_dict={'2.1903' : '7', '3' : '8'}
   
   [reposlist]
   include=docker-ce.repo, epel.repo, pgdg-redhat-all.repo
   ```
   
   

## 初始化配置

PostgreSQL安装成功之后，会默认创建一个名为postgres的Linux用户，初始化数据库后，会有名为postgres的数据库，来存储数据库的基础信息，例如用户信息等等，相当于MySQL中默认的名为mysql数据库。

postgres数据库中会初始化一名超级用户`postgres`

为了方便我们使用postgres账号进行管理，我们可以修改该账号的密码

1. 修改密码

   ```shell
   # 切换用户
   su postgres
   
   # 启动 sql shell  
   psql
   
   # 修改密码
   ALTER USER postgres WITH PASSWORD 'BLa:47ZvP#E*=*KvRMqv';
   ```

2. 配置远程连接

   ```shell
   # 修改可以ip绑定
   vi /var/lib/pgsql/15/data/postgresql.conf
   
   # 监听地址修改为 * 默认listen_addresses配置是注释掉的，所以可以直接在配置文件开头加入该行
   listen_addresses='*'
   
   # 允许所有IP访问
   vi /var/lib/pgsql/15/data/pg_hba.conf
   
   # 尾部添加如下 允许所有IP访问
   host  all  all 0.0.0.0/0 md5
   
   # 重启 
   systemctl restart postgresql-12
   ```

## 创建数据库、用户、权限配置

通常我们会为每个项目和每个用户单独使用一个数据库。

```shell
# 创建数据库
createdb mydb

#新建用户
CREATE USER test WITH PASSWORD 'test';

#赋予指定账户指定数据库所有权限
GRANT ALL PRIVILEGES ON DATABASE mydb TO test;

#移除指定账户指定数据库所有权限
REVOKE ALL PRIVILEGES ON DATABASE mydb TO test
```



参考链接：

> https://ken.io/note/centos7-postgresql12-install-and-configuration
