---
title: Hexo-配置主题
date: 2017-12-26 17:15:24
categories: Hexo
tags: [hexo]

---

搭建完博客之后，进行UI美化将博客主题更改为Next主题(旧)，Polk(新)
<!--more-->

# 新版
后续更换成了`polk`主题：地址在[polk](https://github.com/chunqiuyiyu/hexo-theme-polk)
按照它博客的做法，即能替换
注意的一点在`_config.yml`中要设置每页展示的博客数，是在`Pagination`字段下，设置`per_page`

# 旧版
>这里注意区分两个配置文件：
>站点配置文件：是你的 hexo 博客目录下面的 \_config.yml 文件。
>主题配置文件：是 themes/next 目录下的 \_config.yml 文件。

# 下载主题
>$ cd github\_blog/
>$ git clone https://github.com/iissnan/hexo-theme-next themes/next

# 启用主题
>$ vim \_config.yml

theme字段修改为next

# 主题设定
>$ vim /github\_blog/theme/next/\_config.yml

scheme修改为Pisces

# 设置语言
>$ vim \_config.yml

language修改为 zh-Hans

# 设置菜单
>$ vim /github\_blog/theme/next/\_config.yml

修改如下：
>menu:
>  home: /
>  archives: /archives
>  #about: /about
>  #categories: /categories
>  tags: /tags
>  #commonweal: /404.html


# 设置头像
>$ vim /github\_blog/theme/next/\_config.yml

修改为avatar: /images/touxiang.jpeg
图片需要从网上下载并保存到theme/next/source/images中

# 侧边社交链接
>$ vim /github\_blog/theme/next/\_config.yml

修改如下：
\# Social links
　　social:
　　GitHub: https://github.com/your-user-name
　　Twitter: https://twitter.com/your-user-name
　　微博: http://weibo.com/your-user-name
　　豆瓣: http://douban.com/people/your-user-name
　　知乎: http://www.zhihu.com/people/your-user-name
　　\# 等等

# 设定标签链接
\# Social Icons
　　social\_icons:
　　enable: true
\# Icon Mappings
　　GitHub: github
　　Twitter: twitter
　　微博: weibo

# 站点建立时间
修改如下：since: 2017

# 添加标签页面
新建「标签」页面，并在菜单中显示「标签」链接。「标签」页面将展示站点的所有标签，若你的所有文章都未包含标签，此页面将是空的

你需要在文章的顶部tags的属性后添加
如：tags: [hexo,github]

这样是赋予每个文章他的标签，然后要创建一个单独的标签页面展示所有博客的标签
>$ hexo new page tags

打开source/tags/index.md编辑
>title: 标签
>date: 2014-12-22 12:39:04
>type: "tags"
>comments: false

# 添加分类页面
同标签页面
每篇文章的categories后添加文章的分类

创建分类页面
>$ hexo new page categories

打开/source/categories/index.md编辑
>title: 分类
>date: 2014-12-22 12:39:04
>type: "categories"
>comments: false

