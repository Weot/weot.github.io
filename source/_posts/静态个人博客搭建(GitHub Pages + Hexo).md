---
title: 静态个人博客搭建(GitHub Pages + Hexo)
date: 2019-01-11 12:27
updated: 2019-01-18 18:21
categories:
- 编程开发
- 网站开发
tags: [编程, Hexo, GitHub Pages, Blog]
---
# 静态个人博客搭建(GitHub Pages + Hexo)

## 前言

> 什么是GitHub Pages?

GitHub Pages 本来是用来托管项目静态文档页面的. 因此我们可以将它作为托管静态个人博客的仓库. [GitHub Pages 官方文档](https://pages.github.com/)


> 什么是Hexo?

Hexo 是一个快速、简洁且高效的基于NodeJS的博客框架. Hexo 使用 Markdown（或其他渲染引擎）解析文章, 在几秒内, 即可利用靓丽的主题生成静态网页.
[Hexo 官方文档](https://hexo.io/zh-cn/docs/)


因此, 我们可以利用Hexo生成我们博客的静态页面, 然后通过Git工具推送到我们创建的GitHub Pages仓库中, 通过GitHub的二级域名访问网站, 当然最重要的是还可以绑定自己的顶级域名.

## Hexo框架

### 安装前提


- [node.js](https://nodejs.org/en/)
- [Git](https://git-scm.com/)

由于Hexo是基于node.js的框架, 所以首先要安装好node.js, 安装Git是为了之后部署到Github上使用.
nodejs和Git直接在官网上下载相应的安装包安装就可以了.

### 安装Hexo

直接使用以下命令就可以安装了
```bsah
npm install -g hexo-cli
```

检查是否安装成功, 运行命令:
```
hexo -v
```
如果现实类似下面的输出, 就是安装成功了
```sh
D:\Code\Blog\Hexo
λ hexo -v
hexo: 3.8.0
hexo-cli: 1.1.0
os: Windows_NT 10.0.17134 win32 x64
http_parser: 2.8.0
node: 10.13.0
v8: 6.8.275.32-node.36
uv: 1.23.2
zlib: 1.2.11
ares: 1.14.0
modules: 64
nghttp2: 1.34.0
napi: 3
openssl: 1.1.0i
icu: 62.1
unicode: 11.0
cldr: 33.1
tz: 2018e
```

### 尝试运行

初始化博客项目文件夹
```
hexo init <folder>
```
运行命令后会在指定文件夹下生成hexo的项目文件, 不使用floder参数的话, 会默认在当前文件夹下生成项目文件, 但是要保证文件夹是空的, 才可以初始化成功. 这个过程会持续2-3分钟的时间. 会有类似下面的输出
```
D:\Code\Blog
λ hexo init test
INFO  Cloning hexo-starter to D:\Code\Blog\test
Cloning into 'D:\Code\Blog\test'...
remote: Enumerating objects: 68, done.
remote: Total 68 (delta 0), reused 0 (delta 0), pack-reused 68
Unpacking objects: 100% (68/68), done.
Submodule 'themes/landscape' (https://github.com/hexojs/hexo-theme-landscape.git) registered for path 'themes/landscape'
Cloning into 'D:/Code/Blog/test/themes/landscape'...
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 867 (delta 0), reused 0 (delta 0), pack-reused 866
Receiving objects: 100% (867/867), 2.55 MiB | 1.04 MiB/s, done.
Resolving deltas: 100% (459/459), done.
Submodule path 'themes/landscape': checked out '73a23c51f8487cfcd7c6deec96ccc7543960d350'
INFO  Install dependencies
npm WARN deprecated titlecase@1.1.2: no longer maintained
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.6 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.6: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

added 421 packages from 500 contributors and audited 4699 packages in 43.612s
found 0 vulnerabilities

INFO  Start blogging with Hexo!
```

进入项目文件夹, 运行命令, 生成静态文件(每次有内容或者结构改变的时候, 需要重新生成下, 第一次运行之前也可以不用生成)
```
hexo g
```

在本地运行, 通过浏览器查看运行结果
```
hexo s
```
运行后在命令行应该有类似下面输出
```
D:\Code\Blog\test
λ hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

浏览器访问`http://localhost:4000`, 应该有类似下图的结果
![首次运行预览图](https://img3.doubanio.com/view/status/raw/public/f547be57c538dc0.jpg)

### 配置与自定义

其实安装配置的内容, 官网的文档已经写的很详细, 这里不再过多赘述


## GitHub Pages

GitHub Pages的使用, 官网也有很好的文档.

首先, 创建一个新的仓库, 形式如`username.github.io`, 这个仓库会用来默认当做托管静态页面的仓库, 这时候就可以通过`https://username.github.io`来访问你的GitHub Pages页面了

## 部署页面导GitHub Pages

### 修改Hexo的配置文件

`_confi.yml`文件中许多可以自定义的配置参数, 在文件的最底部是部署的设置

初始的配置应该是下面的样子
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type:
```

将你自己的仓库, 分支等参数加入进去, type为Git, repository为GitHub Pages仓库, branch为部署的分支
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:Weot/weot.github.com.git
  branch: master
```

### 配置Git

然后配置Git的用户名和邮箱
```
git config user.name "username"
git config user.email "XXXX@XXX.com"
```

使用命令一键部署到GitHub Pages
```
hexo deploy
```

### 其他配置

#### 绑定GitHub Pages到自己的域名

- 在Hexo项目文件夹`source`下新建一个CNAME文件, 里面填入要绑定的域名
- Ping自己GitHub Pages的地址, 得到IP地址
- 使用DNS工具CNAME映射到上一步得到的IP地址
