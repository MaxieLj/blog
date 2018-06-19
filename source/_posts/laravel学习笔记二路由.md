---
title: '''laravel学习笔记二路由'''
date: 2016-02-09 22:09:19
tags:
---
laravel框架安装好之后我们就要开始熟悉laravel框架的内部结构。
我们主要是从这个几个方便来学习laravel：

 - route 路由
 - controller 控制器
 - blade 模板
 - model 模型
  
今天我们主要讲路由。

### laravel-route
可能我们在学习laravel之前已经接触过一些框架例如：yii，tp,ci。
但是他们都没有laravel所带的路由系统强大，我们现在开始介绍laravel框架的路由。
### 闭包
最基本的路由接受的是一个闭包函数，直接返回值，例如：
```
Roure::get('/',funcition(){
return 'this is Route'})
```

我们在 **/routes/web.php** 里注册该路由即可返回 `this us Route`。当然laravel路由所能接受的请求方式不仅仅是get还要post put 等等法法，示例如下
```
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```
### 带参数访问###

```
Roure::get('/{id}',function($id){
return 'user id is '.$id
})
```
 当然也可以带多参数访问
 
```
 Route::get('/id/{id}/name/{name}',function($id,$name){
 //
 })
```
 让然也可以传递一个可选参数 ,但是我们需要用`？`来标记该参数，还给予他默认值
```
Route::get('/name/{name}?',function($name='Maxie'){
//
})
```
### 路由命名###
```
Roure::get('/',funcition(){
return 'this is Route'})->name('index')
```
我们可以给路由命名一个名字，让我们在模板中使用rul函数可以解析到我们所命名的路由`url('index')`。这些我们在模板中在细讲

### 路由群组###
路由群组是一个很好东西，我们在开发中很定会给我们的项目分不同模块。例如用户模块，商户模块。这时候我么会给他们加上不同的命名空间，我们可以通过路由群组把我们注册的路由也分类，很方便我们的访问。例如：
```
Route::groun(['namespace'=>'user'],function(){
//我们可以在此注册我们的路由，路由控制器路由会在App/Http/Controller/user 下
})
```
 当让我们可以在群组里继续注册群组，在这里我们不多做演示。
 
## 路由绑定到控制器##

这是我们最关键的地方，一般我们通过访问URI都会想传递参数给我们的控制器，当然laravel还给我们提供了中间件让我们在路路由控制器之间或者路由玉模板之前做一个参数处理或者过滤，我们在此不做演示。后边我们会详细解释，现在我们详细解释路由有绑定到控制器，想必这是你我最关系的。
路由绑定到控制器只需要：
```
Roure::get('/','UserController@showProfile')
```
这样既可绑定我么你的路由到控制器。

## 路由绑定绑定到视图##
有很多的访问不需要我们做处理，只需展示一些东西既可，我么你可以直接把路由绑定到视图

```
Route::get('/',function(){
return view('welcom')
})

这也是我们刚刚配置完laravel时候路由web.php唯一注册一个路由
```

 
 