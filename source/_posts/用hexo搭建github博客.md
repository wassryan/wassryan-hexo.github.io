---
title: Ubuntu+hexo搭建github博客
date: 2017-12-24 14:11:46
tags: [hexo,github]

---

使用github，hexo在ubuntu16.04上搭建静态博客
<!--more-->
# 下载必要的软件

>$ sudo apt-get install nodejs     
>$ sudo apt-get install npm

注意对软件有所更新的话一定要执行下面这两个命令
这两个命令用于安装完软件后对软件及数据库的更新，这是第一个坑！！！

>$ sudo apt update    
>$ sudo apt upgrade

安装完之后，你可以在命令行输入
>$ npm -v

出现版本号时，就说明npm顺利安装，可以进行hexo的安装，这是最简单的一步,直接输入命令
>$ sudo npm install -g hexo

只要不出现ERR都是可以接受的
到这里，恭喜你已经完成所有必备软件的安装了，算半只脚离坑了

# 初始化博客站点

在ubuntu用户目录下新建一个blog，里面相当于本地的blog及其所有的配置
>$ mkdir github\_blog (我新建的名字为blog)
>$ hexo init github\_blog (初始化)

出现INFO Start blogging with Hexo!就说明成功了
>$ cd blog
>$ npm install
>$ hexo -v

出现以下内容就说明成功了
>hexo: 3.3.8
>hexo-cli: 1.0.3
>os: Linux 4.4.0-92-generic linux x64
>http\_parser: 2.7.0
>node: 6.11.2
>v8: 5.1.281.103
>uv: 1.11.0
>zlib: 1.2.11
>ares: 1.10.1-DEV
>icu: 58.2
>modules: 48
>openssl: 1.0.21

接下来就是与github建立链接了，你需要在github上创建自己的轻型静态博客,例如我的k-miracle.github.io

# 安装配置github
## 安装git
>$ sudo apt-get install git

## 配置github(与本地的关系)
>$ git config --global user.name "k-miracle"
>$ git config --global user.eamil "1226956765@qq.com"

## 创建公钥
输入>ssh-keygen -C '1226956765@qq.com' -t rsa
这里的C必须大写，之后你可以一直按回车，直到出现一个字符型的RSA标识
之后会在用户目录 ~/.ssh/ 下建立相应的密钥文件，即 ~/.ssh/id\_rsa.pub

## 在github上添加公钥
  还是在github首页右上角点击头像，选择Settings，然后选择New SSH KEY，把上面一步id\_rsa.pub文件的秘钥复制进去就好了。(注意这步复制秘钥最好自己一行行复制，以免出问题)

## 创建项目仓库
  登录Github官网，点击右上角的+，选择New repository。
  在页面里输入github账户名.github.io只能这么填，不能改，例如我的是k-miracle.github.io填完后点击Create repository即可。(其余选项默认不用改)

# 部署到github
首先config.yml的配置
