+++
title = "nginx相关"
date = 2021-11-19T22:38:35+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['nginx']
+++
### 开启与关闭 nginx

关闭 nginx
nginx -s stop

重新加载
nginx -s relaod

### 配置文件

1. 全局配置

   影响 nginx 服务器整体运行的配置指令

   eg：worker_processes 1;  // 越大 并发越大 

2. event 块

   nginx 服务器与用户的网络连接

   eg: worker_connections 1024;  // 支持的最大连接数

3. http 块

   1. http 全局块
   2. server 块

### 配置实例

1. 反向代理

   1. 访问 http://yii2.test:80 实际访问的是 http://vdong.test:8080   

      ```conf
      server {
      	listen 80;
      	server_name yii2.test *.yii2.test;
      	root "D:/laragon/www/yii2/web/";
          
          location / {
              proxy_pass http://vdong.test:8080;
              # ...
          }
      }
      ```

   2. 根据目录 跳转 不同 服务器

      ```conf
      	
      	location ~ /yii/ {
      		proxy_pass http://yii2-base.test;
      	}
      	
      	location ~ /laravel/ {
      		proxy_pass http://laravel.test:8080;
      	}
      ```

2. 负载均衡

   ```conf
   http{
   	....
   	
       upstream php_upstream {
       	# ip_hash;
       	# fair;
           server 127.0.0.1:9001 weight=1 max_fails=1 fail_timeout=1;
           server 127.0.0.1:9002 weight=1 max_fails=1 fail_timeout=1;
       }
       ....
   }
   
   server{
   	....
   	location / {
           proxy_pass http://php_upstream;
           try_files $uri $uri/ /index.php$is_args$args;
   		autoindex on;
       }
       ....
   }
   
   ```

   负载策略：

   1. 轮询（默认）

      时间顺序 逐一分配，down 了 剔除

   2. weight 权重

      根据权重 分配

   3. ip_hash 

      解决 session 共享问题

   4. fair ( 第三方 )

      根据响应时间分配，越短越多。

3. 动静分离

   动态请求和静态请求分离

   location 根据后缀名 实现转发 ， expires 缓存时间 3d  3 天

   ```conf
   location /www/{
   	root /data/;
   	index index.html index.htm;
   }
   location /image/ {
       root /data/;
       autoindex on;
   }
   ```

   

4. 高可用集群

   两台 nginx； keepalived；虚拟 ip  

   keepalived 其实 对于两台 主备服务器的， 路由作用 

   ```shell
   yum install keepalived -y
   ```

   配置在 `etc/keepalived/keepalived.conf` 。

   1. 修改配置
   2. 在 /usr/local/src 添加检测脚本

### nginx 原理

一个 master 多个 worker

master 管理 监控， worker 争抢 client

热部署

独立进程，一个出问题，不影响其他的

io 多路复用机制， 每个 worker 都是 独立的进程，每个进程只有一个线程，通过异步非阻塞的方式来处理请求。

worker 和 cpu 核心数量相等，cpu 性能充分发挥。

worker_connection 连接数

发送一个请求， 占用了 2 / 4 个连接数。静态 不经过 php-fpm 的 。

nginx 一个 master， 4 个 worker，  每个 worker 支持最大连接数 1024，目前最大并发数是多少？

`4*1024/(2/4)`

静态访问 ： worker_connection * worker_processes / 2

动态访问： worker_connection * worker_processes / 4