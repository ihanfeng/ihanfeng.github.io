---
title: 使用 GitHub Pages and Hexo 搭建免费个人博客
tags:
  - github
  - hexo
categories:
  - Hexo
abbrlink: 34009
date: 2017-09-24 14:30:26
---
# Hexo介绍

  Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

  官网：[Hexo](https://hexo.io/zh-cn/)
<!-- more  -->
# Hexo安装

## 环境安装

* [Node.js](http://nodejs.org/)
* [Git](http://git-scm.com/)

hexo运行所需环境具体安装不在本文讲解范围，不懂的朋友请通过百度或是google解决。

环境安装好，我们就可以通过npm完成hexo的安装。

```
npm install -g hexo-cli
```
## GitHub创建博客仓库

1.创建仓库

博客搭建的前提是你已经在Github上面注册账号，还没有的朋友，赶紧注册一个啦。官网：[https://github.com/](https://github.com/)

登录你的Github帐号，新建仓库，名字为【用户名.github.io】固定写法。

{% asset_img create-repo-01.png %}

{% asset_img create-repo-02.png %}

2.配置SSH-KEY

生成key,在默认位置（C:\Users\账户\.ssh）生成id_rsa.pub和id_rsa两个文件。

```
 ssh-keygen -t rsa -C "zhdevelop@gmail.com"
```
打开公钥文件 id_rsa.pub ，并把内容复制至代码托管平台上

{% asset_img github-add-key.png %}

接下来把创建好的仓库克隆岛我们本地。

## 初始化Hexo
先建立一个空文件夹blog，然后进入这个文件夹blog，输入如下命令

```
mkdir blog && cd blog
```
执行初始化命令
```
hexo init
```
然后把生成好的文件copy到我们从github克隆的分支目录里面。

## Hexo初探

### 安装hexo需要的运行环境
在本地仓库目录下执行，会发现多了个node_modules文件夹，这里面都是npm相关包，是程序运行必须的环依赖。
```
npm install
```
### 生成发布内容
博客生成需要执行如下命令
```
hexo g
```
执行完毕，我们会发现public文件夹里面生成了最后发布的文件。

## 本地预览
终端执行如下
```
hexo s
```
打开[http://localhos:4000](http://localhos:4000)进行本地预览。
