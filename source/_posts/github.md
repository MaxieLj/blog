---
title: github
date: 2015-06-26 22:04:09
tags: git
toc: true
categories: git
------

## Git的安装和使用 ##
 准备工作：
> * 下载git 客户端 [链接地址][1]

下载完毕后，安装一路一下一步。
安装完成之后我们开始使用Git
使用前我们需要告诉Git，我们是谁。所以需要配置一些信息，来确定我们的身份。

 ```
 git config –global user.name “用户名”
 git config –global user.email “邮箱”
 ```
第一条命令用来告诉 git 我们的名字（以后链接github）
第二条命令用来告诉git 我们的邮箱

到此我们就可以使用Git了

## 链接github ##
如果我们希望把自己的项目托管在github我们就需要再做一些配置

首先我们需要生成ssh,使用以下命令
```
ssh-keygen –t rsa –C
```

 1. 然后他提示的默认路径生成一些文件，或者你自己配置路径。 在生成过程中会让你输入密码，可以不输入，一路回车
    生成完毕后，会在你选择路径下生成两个文件 一个是 .rsa 一个是 .pub 我们这里只需要.pub里的内容 复制里边的内容
 2. 接下来我们需要把文件里的key添加到我们的github账号里。 首先我们登陆我们的github账号。 然后在设置里 NEW SSH KEY 添加描述然后把我们的key复制到里边即可。

不过在提交之前我们还需要做一些配置来确定我们需要把当前库的代码推送到github账号的哪个库里
```
git remote add origin git@github.com:aaa/xxx
```
在上边的配置的命令中，aaa表示的我们的github账号。xxx使我们的库文件。
以后就可以使用git来进行版本管理和推送到github里了。
正常操作代码
```
git status
git add .
git commit -a 'message'
git push
```
提交完毕。

如果我们需要在其他电脑上使用我们托管的代码，我们需要使用
```
git pull

```
或者
```
git colne 'address'
```
来从github上获取我们所需要的代码

如果在配置过程中出现错误，按照提示的代码解决即可。
当然在此如果我们直接使用了
```
git commit -a
```
会弹出来vim窗口，在这里写一些vim的命令
在非insert状态下我们使用ctrl+d来批量选择#，然后按d删除

在飞插入模式写:wq 保存并推出
  [1]: https://git-scm.com/download/win