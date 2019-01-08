---
title: hexo git github搭建博客
date: 2015-06-02 21:48:07
tags: hexo
toc: true
categories: hexo
---

------


开始
--

**
需求环境
 1. git [git下载地址][1]
 2. node.js [node.js下载地址][2]

两个安装都是一路下一步
## 验证软件正确安装 ##

    git --version
    node -v
    npm -v
如果显示版本信息，怎安装成功。
## 安装hexo ##
在这里如果被墙的可以使用淘宝镜像cnpm 具体怎么操作可以自行百度，这里不多做解释。
不过一般不会被墙，我使用npm。运行下边的命令安装hexo：
被墙请使用命令：

    npm install -g cnpm --registry=https://registry.npm.taobao.org

以下是没有被墙的命令，被墙把npm换成cnpm即可。安装hexo

    npm install hexo-cli -g

安装完成后，验证是否安装成功。

    hexo -v
新建文件夹yourblog,输入命令

    $ hexo init blog
    $ cd blog
新建博客

    $ hexo new "Hello Hexo"
生成静态页面

    $ hexo generate
运行服务

    $ hexo server

然后打开浏览器输入 localhost:4000 即可访问本地静态页面。


## 接下来我们把本地静态页面PUSH到github ##

 - 首先你要有github账号。
 - 新建一个github.io的库
 - 设置本地github配置参数
 - 经本地静态页面push到github


----------
 **github设置**

新建github账号就不在这里陈述了，如果有同学不会使用git将代码push到github的话可以参考 [廖雪峰的官网][3]
我们来直接进行第二不，创建一个github.io库。
首先登陆我们的github账号点击`new repository`,然后输入youname.github.io
**git设置**
设置git配置信息


    $ git config --global user.name "你的用户名"
    $ git config --global user.email "你的邮箱"
    

**hexo 设置**
    安装hexo git插件

    npm install hexo-deployer-git --save

然后打开博客根目录的_config.yml文件
大致内容是这个样子的

```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 和女票抢可乐
subtitle:
description:
author: MaxieLj
language:
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: yilia

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/MaxieLj/MaxieLj.github.io.git
  branch: master

jsonContent:
  meta: false
  pages: false
  posts:
    title: true
    date: true
    path: true
    text: true
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true
```
我们只需要在尾部添加：
```
deploy:
  type: git
  repository: https://github.com/MaxieLj/MaxieLj.github.io.git
  branch: master
```

然后我们运行

    $ hexo g
生成静态文件
然后推送到github

    $ hexo d
然后访问youname.github.io就可以看到生成的静态页面了
当然我们一般是在本地调试好再用 hexo d推送到github


  [1]: https://git-scm.com/downloads
  [2]: https://nodejs.org/en/
  [3]: http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000