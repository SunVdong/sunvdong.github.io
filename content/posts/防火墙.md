+++
title = "防火墙"
date = 2021-11-19T22:48:36+08:00
draft = true
author = 'vdong'
categories = ['技术']
tags = ['防火墙']

+++

firewall-cmd --list-all

firewall-cmd --add-service=http --premanent
firewall-cmd --add-port=80/tcp --premanent

ps: premanent 是永久生效

重新加载
firewall-cmd --reload

查看状态
firewall-cmd --state

停止firewall
systemctl stop firewalld.service

禁止firewall开机启动
systemctl disable firewalld.service 