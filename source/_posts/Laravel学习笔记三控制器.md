---
title: 'Laravel学习笔记三控制器'
date: 2017-02-16 21:26:25
tags:
---



标签（空格分隔）： Laravel

---

关于laravel我们学习了如何配置laravel、以及路由。接下来让然是至关重要的控制器了。
在MVC框架中，C是一个很重要的环节。用来处理我们的数据逻辑。

我们之前在学习路由是后，是给Route::get()第二个参数一个闭包函数。当然能处理一些数据逻辑，但是如果我们所有的数据逻辑都放在路由文件，那么我们路由文件将会特别臃肿，而且我们的代码耦合度会特别高。这也就回到php面向过程的时代，这不是我们所想要的。

在我们的面向对象的思维中，是把不同功能的代码封装成一个一个类，各自处理自己流水线上的数据。所以我们要通过路由把必要的参数传给我们的控制器去处理。

在这里看到一个小插曲，Laravel官方文档说在上线前使用`php artisan route:cache`会让路由速度快上一百倍，我不知道这是不是吹牛逼。废话不多说我们开始我们的控制器学习吧。

###控制器到路由###

 在第一步我们需要把访问到路由的请求转到控制器。在Laravel中使用下边代码即可：
 ```Route::get('user/{id}', 'UserController@show');```
  我们可以把客户的请求跳转到 App/Http/Controller下的UserController的show方法。
  
  在这里laravel给我们提供很好的工具让我们来创建控制器。使用`php artisan make:controller NameController` 。默认会出现App/Http/Controller 路径下。并且还有一些级基础代码。
```<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class NameController extends Controller
{
    //
}
```

###传递参数到控制器###

我们看到在自动生成代码中我们看到引入了`Request` 类。这个类要讲起来其实可以单独讲，但是我们学到控制器就稍微讲一下，不然没法传递参数。如下
```
 public function index(Request $request){
    	$name = $request->input('name');

    }
```
用该方法可以获取到我们想要的参数。


