---
title: '常量变量包init函数执行顺序'
date: '2025-03-25T23:28:49+08:00'
draft: true
author: vdong
description: ''
categories:
  - 技术
tags:
  - go
  - 100-mistakes
typora-root-url: ..\..\..\static
---

## 常量、变量、包、init函数执行顺序

![执行顺序](./imgs/常量变量包init函数执行顺序/hVMYyqi6EU.png)

- 一个包中的多个文件中，可以有多个 init()，按照文件名字母排序执行
- 一个文件中，可以有多个 init()，按照声明顺序执行
- 被 import 多次，只会被准确的执行 1 次
