---
title: '保留提交记录和tag的git仓库迁移'
date: '2024-07-16T14:33:25+08:00'
draft: false
author: vdong
description: '保留提交记录和tag的git仓库迁移'
categories:
  - 技术
tags:
  - git
typora-root-url: '..\..\..\static'
---



## 问题场景1

将 Git 仓库从一个服务器迁移到另一个服务器，并保留所有的分支和提交记录。例如：我有一个git仓库（1.1.1.5:8888/a.git），有多个分支且做了多次提交了，现在需要通过u盘迁移至另一个仓库(2.2.2..5:8888/b.git)，我应该怎么操作呢。

## 操作步骤

1. 克隆源仓库到本地

   ```git
   git clone --mirror http://1.1.1.5:8888/a.git
   ```

   使用 `--mirror` 参数会确保你克隆的是镜像仓库，包括所有的分支、标签和提交记录。

2. 将仓库复制到目标服务器

3. 在目标服务器上初始化新的仓库

   ```git
   cd a.git
   git remote set-url origin http://2.2.2.5:8888/b.git
   git push --mirror
   ```

   `git remote set-url origin` 命令将远程仓库的地址更改为新的地址。`git push --mirror` 命令会将所有的分支、标签和提交记录推送到新的远程仓库。

## 问题场景2

一个项目中有多个 git 仓库，我希望合并到一起，一个 git 仓库一个目录， 同时我希望保留提交记录和tags 等。

## 操作步骤

1. 在源机器上

   ```git
   git clone --bare https://example.com/repo1.git repo1.git
   git clone --bare https://example.com/repo2.git repo2.git
   
   cp -r repo1.git /path/to/usb-drive/
   cp -r repo2.git /path/to/usb-drive/
   ```

2. 在目标机器上

   ```git
   cp -r /path/to/usb-drive/repo1.git /path/to/temp-directory/
   cp -r /path/to/usb-drive/repo2.git /path/to/temp-directory/
   
   mkdir combined-repo
   cd combined-repo
   git init
   
   git remote add -f repo1 /path/to/temp-directory/repo1.git
   git fetch repo1
   
   git remote add -f repo2 /path/to/temp-directory/repo2.git
   git fetch repo2
   
   git checkout -b repo1-master repo1/master
   git checkout master
   git merge --allow-unrelated-histories repo1-master
   mkdir repo1
   git mv * repo1/
   git commit -m "Move repo1 content to repo1 directory"
   
   git checkout -b repo2-master repo2/master
   git checkout master
   git merge --allow-unrelated-histories repo2-master
   mkdir repo2
   git mv * repo2/
   git commit -m "Move repo2 content to repo2 directory"
   ```

   这样，你就可以通过U盘将多个仓库合并到一个目标仓库中，并保留所有的提交记录和标签。

