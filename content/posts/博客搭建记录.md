---
title: "博客搭建记录"
date: 2021-11-03T15:46:20+08:00
lastmod: 2022-05-30T20:00:00+08:00
draft: false
author: vdong
categories: 
- 技术
tags:
- hugo
- 折腾
---

这是一个静态 bolg 网站。

记录搭建过程。

## 技术栈：

hugo , git , github 。

- hugo：是一个开源的 web 框架，使用 go 语言开发，可以将 markdown 文件快速的建构成静态网站。

- git: 作为版本管理工具。

- github: 作为代码仓库，使用 github 的 actions 做自动的建构和部署，分别部署在 github pages 和 阿里云上。

## 搭建过程

### 本地安装 hugo

不同操作系统安装差异略有不同，具体请参考：[安装文档](https://gohugo.io/getting-started/installing/)

本人使用如下命令安装

```shell
# Hugo extended 支持 Sass/SCSS
scoop install hugo-extended
```

### 创建站点，测试

参考[快速开始](https://gohugo.io/getting-started/quick-start/)创建站点，并测试。

本人选择的主题是 [Doit](https://github.com/HEIGE-PCloud/DoIt)，并作简单配置（修改 `config.toml` 文件）即可，先跑起来。

至此博客已经搭建完成，你可以添加内容，在本地浏览博客，生成静态文件（默认 `public/` 目录下），上传 `public/` 目录的下的内容到服务器，就是一个静态的`blog` 网站。

### 添加版本管理

本地：进入站点目录，`git init` 将该目录变成一个仓库；之后正常做版本管理。

github线上: 创建仓库，推荐命名 `USER_NAME.github.io` （之后设置 github pages 时，网址没有路径）。推送站点仓库到 github 。

### 配置持续集成

使用 `github actions` ，自动建构 `hugo` 并发布到 `github pages` 和 第三方服务器。

- github actions ： 是一些持续集成的操作，包括但不局限于：抓取代码、运行测试、登录远程服务器，发布到第三方服务等。github 允许开发者，把每个操作协程独立的脚本，存放到代码仓库，供他人调用。[官方市场](https://github.com/marketplace?category=&query=&type=actions&verification=)，[awesome-actions](https://github.com/sdras/awesome-actions)
- github pages ：是一个静态网址托管服务，`github` 允许你为你的每一个仓库，制作一个静态网页。

#### 配置 github actions

你可以在 actions 选项卡内新建 workflow ；也可以在项目根目录手动创建`.github\workflows\XXX.yml` 然后提交， action 的配置文件叫做 workflow 文件。 以 `.yml` 为后缀。

```yml
name: Hugo build and deploy
on:
  # 当 main 分支被 push 时，此 workflow 被触发
  push:
    branches: [ main ]

  # 允许你在 actions 选项卡中手动执行此 workflow 
  workflow_dispatch:
jobs:
  Hugo-build-deploy:
    runs-on: ubuntu-latest # 基于 ubuntu 最新版
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v2 # 检出代码
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo # 安装 hugo 
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build # 建构 hugo
        run: hugo --minify
        
      - name: Deploy Github Pages # 部署到 github pages
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
      - name: 📂 Sync files # ftp 同步到 第三方服务器，如无第三方服务器可以省略此步。
        uses: SamKirkland/FTP-Deploy-Action@4.1.0
        with:
          server: ${{ secrets.FTP_URL }}
          username: ${{ secrets.FTP_USER }}
          password: ${{ secrets.FTP_PWD }}
          local-dir: ./public/
```

ps: 敏感字段需要在 `Settings/Secrets` 选项卡中设置，如上的 `${{ secrets.FTP_PWD }}` 等。

ps：可以在 actions 选项卡中查看 action 运行状态，并调试错误。

#### 配置 github pages

当以上 actions 成功运行一次后（push 操作后，或 手动运行），你会发现多了一个 `gh-pages` 分支。

进入选项卡 `Settings/Pages` 配置 `source` ，`branch` 为 `gh-pages`，目录为 `/(root)`。

成功后会提示你 `Your site is published at https://sunvdong.github.io/`  （如果你的仓库名是 `你的用户名.github.io`，则网址后没有路径）。

### 增加评论功能

参考官方文档：[Waline 快速上手](https://waline.js.org/guide/get-started.html)

1. LeanCloud 设置 (准备数据库)

   1. 注册 LeanCloud  国际版 （用于存储数据）
   2. 在 LeanCloud 创建应用
   3. 记录 id ，密钥等。

2. Vercel 部署 (服务端)

   1. github 登录 Vercel
   2. 新建项目，初始化仓库
   3. 设置环境变量，步骤1（第3小步）中记录的id等
   4. 重新部署，使环境变量生效
   5. 配置域名（默认域名 xxxxxx.vercel.app 国内不能访问）
   6. 域名服务器商处添加新的 `CNAME` 解析记录

3. 修改 hugo 配置文件  `config.toml`：

   ```toml
   [params.page]
       # 评论系统设置
       [params.page.comment]
         enable = true
         # Waline 评论系统设置 (https://waline.js.org)
         [params.page.comment.waline]
           enable = true
           serverURL = "https://comment.vdong.xyz/" # 此处为上边配置的域名
           pageview = true
           comment = true
           emoji = ['https://i.whaoot.com/emojis/qq/'] # emoji 使用了个人七牛云存储
           # emoji = ['https://cdn.jsdelivr.net/gh/walinejs/emojis/weibo']
           # meta = ['nick', 'mail', 'link']
           # requiredMeta = []
           # login = 'enable'
           # wordLimit = 0
           # pageSize = 10
           # imageUploader = false
           # highlighter = false
           # texRenderer = false
   ```

### 增加搜索功能

DoIt 支持多种搜索方式，这里使用 Fuse.js ,只需要修改 hugo 的配置文教 `config.toml` 即可：

```toml
 [params.search]
    enable = true
    # 搜索引擎的类型 ("lunr", "algolia", "fuse")
    type = "fuse"
    # 文章内容最长索引长度
    contentLength = 4000
    # 搜索框的占位提示语
    placeholder = "搜索文章标题或内容..."
    # DoIt 新增 | 0.2.1 最大结果数目
    maxResultLength = 10
    # DoIt 新增 | 0.2.3 结果内容片段长度
    snippetLength = 50
    # DoIt 新增 | 0.2.1 搜索结果中高亮部分的 HTML 标签
    highlightTag = "em"
    # DoIt 新增 | 0.2.4 是否在搜索索引中使用基于 baseURL 的绝对路径
    absoluteURL = false
    [params.search.fuse]
      # DoIt 新增 | 0.2.12 https://fusejs.io/api/options.html
      isCaseSensitive = false
      minMatchCharLength = 2
      findAllMatches = false
      location = 0
      threshold = 0.3
      distance = 100
      ignoreLocation = false
      useExtendedSearch = false
      ignoreFieldNorm = false
```

### 配置文章修改时间

hugo 配置文件中新增如下：

- `:git` 从文件的 git 提交记录获取
- `lastmod` 从文件中的 `lastmod` 字段获取
- `:fileModTime` 从文件修改时间获取

```toml
enableGitInfo = true # 获取每个文件的git修改信息
[frontmatter]
  lastmod = [':git', 'lastmod', ':fileModTime', ':default']
```

## 问题记录

### 子模块未初始化或更新导致，hugo server -D 不能预览网站问题

原因是：子模块未初始化，导致模板缺失，从而不能生成 html 文件。
```shell
# 初始化子模块
git submodule init

# 更新子模块
git submodule update --remote

# 查看更新状态
git submodule status

```

### 使用 Typora 编辑 md 文件时的图片设置问题

1. 文件 -> 偏好设置 -> 图像 -> 插入图片时..  选择 复制到指定路径，并指定路径为，`hugo\static\imgs\${filename}\` 。 
2. 格式( Format ) -> 图像( Image ) -> 设置图片根目录( Use Image Root Path )，目录选择到 imgs 的上一级 static 目录，这时 Typora 会增加一个 Markdown 文件顶部的元数据（yml格式） `typora-root-url: "..\\..\\..\\static"`（每个文件都得设置，不推荐）。
3. 直接修改，`archetypes/default.md` 模板文件，增加元数据 yml格式：`typora-root-url: ../../../static/` 或 toml格式：`typora-root-url = "..\\..\\..\\static"`，之后在新建 md 文件时就自动增加了。（ `../` 的个数取决于你的目录层级）。

## 成果

源码：[https://github.com/SunVdong/sunvdong.github.io](https://github.com/SunVdong/sunvdong.github.io)

博客地址: [https://www.vdong.xyz](https://www.vdong.xyz) or [https://sunvdong.github.io](https://sunvdong.github.io)

## 参考

[Hugo官网](https://gohugo.io/)

[GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

[使用 Hugo + Github 搭建个人博客](https://zhuanlan.zhihu.com/p/105021100)

[hugo与typora图片路径统一问题](https://blog.bitllion.top/posts/hugo%E4%B8%8Etypora%E5%9B%BE%E7%89%87%E8%B7%AF%E5%BE%84%E7%BB%9F%E4%B8%80%E9%97%AE%E9%A2%98/)



