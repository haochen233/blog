---
title: "使用git Page+hugo搭建博客"
date: 2021-06-30T13:28:33+08:00
draft: true
author: haochen
categories:
tags:
- github
- hugo
---

### 1.使用环境准备：
1. 安装go：hugo需要版本大于等于1.16。可以从[Go官网](https://golang.google.cn/dl/)下载所需要的go版本。
    - 下载完成解压后需要配置环境变量（PATH、GOAPTH等）
    - 检查go开发环境是否配置正确：`$ go version`
2. 安装git：可以从[git官网](https://git-scm.com/downloads)下载你所需要的系统和版本,这一步就略过了。

### 2.安装hugo：
#### 使用go get 命令自动获取并安装
- 开启go mod: `go env -w GOMODULE111=on`
- 配置国内因为GFW的原因不能下载第三方包，所以需要配置代理来进行访问：`go env -w GOPROXY=https://goproxy.io,direct`
- 使用**go get**自动安装hugo: `go get github.com/gohugoio/hugo`
- 检查是否安装成功: `hugo -h`

### 3.为什么使用git page
1. 免费托管
2. 搭建简单

### 4.在github上创建blog仓库、git page仓库
为什么要创建两个仓库？  
- **blog**仓库是静态网站的开发环境
- **git page**仓库是生产环境用来发布网站(仓库名必须是[username.github.io](https://haochen233.github.io),到时可以通过这个域名直接访问)。 

例如(开发环境仓库)：
![development](/img/post/使用git-page+hugo搭建博客/blog.png)
例如(生产环境仓库)：
![production](/img/post/使用git-page+hugo搭建博客/haochen233.png)


### 5.使用hugo生成新站点

- `hugo new site [your site name]`

![hugo new site](/img/post/使用git-page+hugo搭建博客/1.png)
- 使用git 添加远程仓库: `git remote add origin https://github.com/haochen233/blog`

- 从[hugo theme](https://themes.gohugo.io/)网站挑选一个心仪的主题,
![hugo new site](/img/post/使用git-page+hugo搭建博客/mainroad.png)
点击**download**跳转到仓库首页，复制仓库地址。

- 进入刚才blog仓库下的themes目录下，将主题仓库添加为子模块：`git submodule add https://github.com/Vimux/Mainroad.git`

然后blog仓库根目录下的config.toml,后面追加一行:`theme = "mainroad"`，表示使用[mainroad](https://themes.gohugo.io/mainroad/)主题。

- 添加第一个博客文件, 例如 first.md：`hugo new post/first.md`

- 在本地运行博客服务器: `hugo server -D`，浏览器输入https://127.0.0.1:1313即可访问到博客。

### 6.将hugo静态网站部署到git page上。
- 使用hugo 发布静态资源：`hugo -D` ,此时会在blog仓库下的根目录下生成public目录，这个目录就包含了博客需要的所有静态资源。
![hugo new site](/img/post/使用git-page+hugo搭建博客/public.png)

- 因为每发布一次新博客，public目录就会更新,所以需要将public作为生产环境的仓库**git page**，需要先删除掉之前的public目录，将**git page**仓库添加到public目录下: `rm -rf public`, `git submodule add https://github.com/haochen233/haochen233.github.io.git public`

- 然后hugo发布用的public就是**git page**仓库了，每次`hugo -D`后public目录下的子模块就可以commit和push到**git page**仓库了。

### 7.访问站点
在将public下的文件推送到github上后，就可以通过https://username.github.io/来访问博客站点了。