---
title: hexo多终端同步
date: 2017-12-25 16:28:39
tags: [hexo,github]

---
想在不同的终端进行github+Hexo的博客发布更新
主体的思路是将博文内容相关文件放在Github项目中master中，将Hexo配置写博客用的相关文件放在Github项目的hexo分支上
这个是关键，多终端的同步只需要对分支hexo进行操作
<!--more-->

# 本地上传博客工程
上一博客部署博客到Github以后，我们可以在Github仓库的master分支上看到我们上传的博客文件
但是这个博客文件是不包含hexo配置的，所以我们需要新建分支，使用git指令将带hexo配置的Github工程文件上传到新建的分支上

# 新建分支hexo
在github博客项目的Branch处点击输入新的分支hexo，并选择创建该分支

# git初始化
先进入到本地的博客文件夹下
>$ cd github\_blog
>$ git init

# 添加仓库地址
>$ git remote add origin https://github.com/k-miracle/k-miracle.github.io.git

# 切换到新建的hexo分支
>$ git checkout -b hexo

# 添加所有文件到git
>$ git add .

# 先把github仓库中的文件pull下来
>$ git pull origin hexo 

# 本地仓库全部push到github
>$ git commit -m "update 2017/12/25"
>$ git push origin hexo

# 另一终端完成clone和push操作
此时在另一终端更新博客，只需要将Github的hexo分支clone下来，进行初次的相关配置
PS:最关键的是，防止原电脑中的博客文件全部消失，消失了也可以同步存储在云端的博客数据，不用再重新搭建博客，这很好！
>$ git clone -b hexo git@github.com:k-miracle/k-miracle.github.io.git  //将Github中hexo分支clone到本地
>$ cd k-miracle.github.io
>$ npm install    //注意，这里一定要切换到刚刚clone的文件夹内执行，安装必要的所需组件，不用再ini
>$ hexo new post "new blog name"   //新建一个.md文件，并编辑完成自己的博客内容
>$ git add source  //经测试每次只要更新sorcerer中的文件到Github中即可，因为只是新建了一篇新博客
>$ git commit -m "提交的信息"
>$ git push origin hexo  //更新分支
>$ hexo d -g   //push更新完分支之后将自己写的博客对接到自己搭的博客网站上，同时同步了Github中的master

# 现在终于同步完两个终端，如何在两个终端更新博客，每次只需要以下６步
>$ git pull origin hexo //每次写博客前从云端同步信息
>$ hexo new post "new blog name"
>$ git add source //只更新了博客的话，就上传source下的文件即可
>$ git commit -m "更新的信息"
>$ git pushh origin hexo
>$ hexo d -g
