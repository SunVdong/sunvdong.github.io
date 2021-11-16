+++
title = "git 常用命令"
date = 2021-11-04T19:12:06+08:00
draft = false
author = 'vdong'
categories = ['技术'] 
tags = ['git']
+++

### 常用命令

#### clone 特定分支
```shell
git clone -b v5.12 git@gitee.com:xxx/xxx.git
```

#### 日志查看
```shell
git log --graph --pretty=oneline --abbrev-commit
```

#### 添加远程库
```shell
git remote add origin git@github.com:michaelliao/learngit.git
```

#### 第一次推送
```shell
git push -u origin master
```

#### 比较差异
```shell
git diff readme.txt
```

#### 配置别名
```shell
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.lg "log --graph --pretty=oneline --abbrev-commit"
git config --global alias.st "status"
git config --global alias.ck "checkout"
git config --global alias.br "branch"
```

#### 标签
- 命令`git tag <tagname>`用于新建一个标签，默认为`HEAD`，也可以指定一个commit id；
- 命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息；
- 命令`git tag`可以查看所有标签。
- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

#### 获取今天的日志
```shell
git log --since="6am"
```

### 错误记录
#### pull 或 merge 时错误
fatal: refusing to merge unrelated histories

解决方案：
在你操作命令后面加--allow-unrelated-histories
eg:
git merge master --allow-unrelated-histories
git pull origin master --allow-unrelated-histories


#### git如何忽略已经提交的文件 (.gitignore文件无效)
解决方案：

1. `git rm -r --cached 要忽略的文件` (如: `git rm -r --cached build/*`, 如修改列表中的内容全部是不需要的, 那么你可以使用最最简单的命令搞定`git rm -r --cached .`)
1. `git add .`
1. `git commit -m " commet for commit ....."`
1. `git push`



#### 文件名大小写问题
默认对大小写不敏感
文件名相同 造成 Changes not staged for commit 错误

解决方案：

```shell
git config --global core.ignorecase false
```

#### 换行符问题
win上会提示CRLF will be replaced by LF


解决方案：
[https://stackmirror.com/questions/5834014](https://stackmirror.com/questions/5834014)
```shell
git config --global core.autocrlf input
```

#### 修改最后一次commit的message
```shell
git commit --amend
```

#### 文件名中文 \xxx 八进制中文乱码
```shell
git config core.quotepath false  --global
```

#### 删除远程分支, 后还显示
```shell
git push origin -d 
git remote prune origin
```


### 撤销

#### 场景1 ：未 add ，但是想放弃修改
```shell
git checkout fileName
// or
git checkout .
```

#### 场景2：执行了 add , 但是想放弃修改
```shell
git reset HEAD <fileName>
```

#### 场景3：commit 了，想修改，且不想产生新的提交
```shell
git add fileName 
git commit -amend -m '说明'
```
####  场景4：commit 了，想撤销到某次 commit
```shell
git reset [--hard|soft|mixed|merge|keep] [commit|HEAD]
```

#### 场景5： 已经 push 了，撤销制定文件到指定版本
```shell
# 查看文件版本
git log <filename>
git checkout <commitID> <fileName>
```

#### 场景6： 撤销最后一次提交
```shell
git revert HEAD // 是反做了目标版本，产生一个新的commits

git reset --hard HEAD^ // 会删除目标版本后的版本
```

### 学习网站
[learngitbranching](https://learngitbranching.js.org/)

[图解git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)

[githowto](https://githowto.com/)
​
[git飞行规则](https://github.com/k88hudson/git-flight-rules/blob/master/README_zh-CN.md)​