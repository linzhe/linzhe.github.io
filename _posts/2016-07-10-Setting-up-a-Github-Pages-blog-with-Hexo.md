---
title: Setting up a Github Pages blog with Hexo
date: 2016-07-10 00:14:33
tags:
  - Markdown
  - github
categories:
  - github
---

### 安装 Node.js
------

1. 从[Node.js下载地址](https://nodejs.org/en/)下载相应的包进行安装
2. 在安装好Node.js，会默认安装好npm
3. 使用npm安装Hexo
   ```shell
   npm install -g hexo
   ```

### 使用Hexo
------
#### 创建文件目录
```shell
$ hexo init username.github.io
$ cd username.github.io
```
#### 修改相关的配置，使用github部署
```shell
vi _config.yml
```
在Deployment的配置中添加
```shell
deploy:
  repo: https://github.com/username/username.github.io
  type: git
```
### 本地测试
使用Hexo命令
```shell
hexo server
```
启动本地测试服务，访问[http://localhost:4000](http://localhost:4000)，可以看到默认的主题与blog文章。

### 更多Hexo命令
```shell
hexo deploy # 部署到github
hexo generate # 生成静态文件
hexo clean    # 清除生成的静态文件
```

### 更多主题

默认的主题可以在[hexo.io](http://www.hexo.io/theme)中获取，方法很简单，只要将主题的代码chechout到theme目录下，然后修改_config.yml文件中的theme的值，重启测试服务就可以，十分方便。
