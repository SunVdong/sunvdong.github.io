+++
title = "Git管理word文档"
date = 2023-05-17T11:47:43+08:00
draft = false
author = 'vdong'
categories = ['技术']
tags = ['git']
+++

## word 文档管理

最近写word文档有点多，并且是多人编写，然后就造成文档的标题变成了：

```txt
xxxx-1.docx
xxxx(1).docx
xxxx-0517.docx
```

总之乱七八糟。

所以就搜索了一下，word 的版本管理方案，这里简单介绍一下 Git+Pandoc 的版本管理方案（需要一定的 Git 基础）。

1. 安装 Git 和 [Pandoc](https://pandoc.org/)

2. 配置 git 的 diff 差异引擎 pandoc，其可以将 word 文件转换为 md 文件，且设置为Pandoc差异引擎的提示方式为不显示提示信息。
   ```git
   # 使用于当前仓库
   git config diff.pandoc.textconv "pandoc --to=markdown"
   git config diff.pandoc.prompt false
   
   # 全局配置
   git config --global diff.pandoc.textconv "pandoc --to=markdown"
   git config --global diff.pandoc.prompt false
   ```

3. 【可选】配置别名

   ```git
   # 以单词为单位进行比较
   git config alias.wdiff 'diff --word-diff=color --unified=1'
   ```

4. 在需要管理的仓库下新建文件 .gitattributes，加入如下内容，用于指定 Git 在比较 .docx 文件时使用 Pandoc 差异引擎。
   ```git
   *.docx diff=pandoc
   ```

##  拓展其他 diff 引擎

可以使用以下命令查看支持的 git diff/ git merge 插件

```git
git difftool --tool-help
git mergetool --tool-help
```

所以可以指定 diff 插件为 Beyond Compare ，参考链接 [Beyond Compare](https://www.scootersoftware.com/kb/vcs)。