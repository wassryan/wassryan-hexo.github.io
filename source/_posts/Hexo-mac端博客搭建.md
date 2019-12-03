---
title: Hexo-mac端博客搭建
date: 2018-12-23 15:53:01
categories: Hexo
tags: [hexo]

---
#  安装必备文件

### 本地站点搭建

1、安装git

mac osx中自带git不需要安装

2、安装Node.js

从官网直接下载按步骤安装即可；

安装完之后使用以下命令查看版本号

```
node -v
npm -v
```

3、安装hexo

```
sudo npm install -g hexo
```

### 在mac上配置github

1、输入命令(该邮箱是github注册时的账号邮箱)

```
ssh-keygen -t rsa -C 邮箱地址
```

2、复制上一步在本地产生的ssh key，使用vim打开，并复制全部内容，然后登陆github，将密钥复制到Settings/SSH and GPG keys，new一个新的密钥

```
vim ~/.ssh/id_rsa.pub
```

3、关联

```
git config --global user.name "github用户名"
git config --global user.email "邮箱地址"
```

好，至此本地与github关联完成

### 同步云端博客到mac本地

```
git clone -b hexo git@github.com:k-miracle/k-miracle.github.io.git

cd k-miracle.github.io
npm install

hexo new post "mac同步博客"//新建一个md文件，编写博客内容
git add source//每次只要更新sorcerer中的文件到Github中即可，因为只是新建了一篇新博客
git commit -m "提交的信息"
git push origin hexo//更新分支
hexo d -g//push更新完分支之后将自己写的博客对接到自己搭的博客网站上，同时同步了Github中的master

```

### 如果在ubuntu和mac都要写博客时，按照以下6步

```
git pull origin hexo //每次写博客前从云端同步信息
hexo new post “new blog name”
git add source //只更新了博客的话，就上传source下的文件即可
git commit -m “更新的信息”
git pushh origin hexo
hexo d -g
```

ps：运行git add source时，可能会出现warning: unable to access '/Users/kary/.config/git/ignore': Permission denied，这是因为kary用户没有.config的访问权限

只需运行以下命令即可更改权限

`sudo chown kary .config`


