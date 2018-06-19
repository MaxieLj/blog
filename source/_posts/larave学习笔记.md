layout: '''laravel'
title: laravel学习笔记
date: 2016-02-08 22:10:05
tags:
---
# laravel-  学习笔记
------

laravel最近新冒出来的一个框架，在国外已经风靡，但是在国内却并没有流行起来。我想造成这种原因可能有几点：
 

 1. 国内从业人员英语基础普遍偏差，没有系统阅读文档的能力
 2. 主动学习积极性较差
 3. 固守于已经学会的知识
 
 以此为鉴

## laravel 配置篇 ##
lavel 有三种安装方法：

 - 使用composer安装
 - 使用laravel安装器安装
 - 下载解压安装
 
###  composer ###
先说第一种，使用composer安装，使用 composer create-project 进行安装
```
composer create-project --prefer-dist laravel/laravel blog
```
### laravel安装器### 

 使用laravel安装器首先需要安装laravel安装器，让我们来使用composer安装laravel安装器

```
composer global require "laravel/installer"
```

当然我们在使用laravel安装器安装laravel之前，需要配置PATH环境变量，不然系统找不到laravel安装器
laravel安装器默认位置
Windows:
`C:\Users\admin\AppData\Roaming\Composer\vendor\bin`
linux:
`~/.composer/vendor/bin`


### 下载解压安装 ###
因为composer可能被墙，所以golaravel给出了已经集成了所有以来的laravel意见安装包
下载地址为`http://www.golaravel.com/download/`


# 配置
laravel所有的配置文件都在根目录conf文件夹下
当然我们不能忽视的一个很重的一点，就是要设置laravel的应用key，如果不加以设置，我的会话和其他需要加密的信息将会不安全，让然我们通过composer或者laravel安装器安装的laravel不需要进行配置，因为在我们安装laravel时候composer和laravel已经给我们配置好了，所以通过第三种方法安装laravel的，需要为laravel配置新的key，生成key的方法是，在laravel根目录输入`key:generate`.
laravel所有的配置都是conf文件夹下的，但是为了方便配置，laravel调用根目录的.env,我们配置框架最关键的数据库信息卸载根目录的.env文件里

# 开发服务器

laravel为我们提供了一个简单的开发服务器，当我需要时只需要在跟路面输入命令`php artisan serve`即可。但是这毕竟是开发使用过的，不能再生产环境中使用。
#laravel其他
laravel还为我们提供很多其他功能，具体我就不在此赘述，需要请看官网。
