+++
title = "PHP 和 Nginx 通信"
date = 2021-11-04T19:11:06+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['php']
+++

## 一个请求的生命周期

### 0. 启动服务 准备工作

启动 php-fpm，通信模式， TCP socket， Unix scoket。

PHP-FPM 启动两种进程， master：监控端口、分配任务，管理 worker 进程， worker： 就是 php 的cgi 程序，解释和编译执行 php 脚本。

启动 nginx。载入 ngx_http_fastcgi_module 模块，初始化 FastCGI 执行环境，实现 FastCGI 协议 请求代理。

### 1. request => nginx

nginx 接受请求，基于 location 配置，选择 handle，即 代理 PHP 的 handle。

### 2. nginx => PHP-FPM

nginx 将请求翻译成 fastcgi 请求。

通过 tcp scoket /unix scoket 发送给 PHP-FPM 的 master 进程

### 3. PHP-FMP 的 master => worker

master 分配 worker 执行 php 脚本，没空闲 worker 返回 502。

worker （php-cgi）进程执行 php 超时 返回 504

处理解释返回结果

### 4. worker => master => nginx

worker 返回处理结果给 master，关闭连接，等待下一个请求

master 进程返回结果给 nginx

nginx 根据 handle 顺序，将响应一步一步返回给客户端

## php解释执行的机制

1. 初始化 ，启动 zend 引擎，加载注册的拓展模块 （php-fpm 启动时）
2. 读取脚本文件，词法分析和语法分析 生成语法树
3. zend 引擎编译语法树， 生成 opcode
4. 执行 opcode，返回结果

步骤 2-4 每个请求都要执行一次，极大的浪费。

## opcache

将编译好的操作码放入共享内存，提供给其他进程访问。

主要缓存

1. opcode
2. interned string ， 如注释，变量名

## php-fpm 三种运行模式

1. ondemand 内存优先

php-fpm 启动时候不会创建 worker 进程，链接进来时按需创建

2. static 静态池

启动时 创建固定数量的 worker 进程，也有1 秒的定时器，统计进程状态

3. dynamic 服务优先

启动时候创建一部分进程，运行过程中动态调整worker 数量


ps： [https://segmentfault.com/a/1190000022549643](https://segmentfault.com/a/1190000022549643)
