---
title: '''laravel学习笔记四中间件'''
date: 2016-03-02 19:44:12
tags:
---
 # Laravel 学习笔记四Http中间件

标签（空格分隔）： laravel

---
 
 
 ### 简介###
  关于中间件，从字面意思上我们很好理解中间件的意思。就是存在两个节点之间的节点。这里我们所描述的节点是`Route`和`Controller`。
  
  
  一次正产给的请求是客户端发送请求，服务端接受请求，然后对参数做一些判断然后处理逻辑。任何请求都会从进入控制器开始就直接进入真正的逻辑处理，我们会先判断请求参数是否正确，以防止黑客攻击。比较常见的处理逻辑反馈有403、503页面。这些是我们在接受参数后对参数进行处理，从而反馈到客户端。如果我们传递参数较多，对参数筛选很严格的话，就会出现很多对参数判断的代码，以及当一些对参数判断的代码需要复用时候我们无法将其从控制器中分离出来，这些都是我们平常工作中饭经常遇到的问题。这是`中间件`就能很好地处理这个问题了。
  
  间件的作用是在请求从`Route`进入控制器是时，我们可以先将请求引入到中间件中，对数据做些处理，然后才引入到控制器中，或者直接反馈给客户端。这样我们的控制器可以专心于数据逻辑，从而代码简洁，以及参数判断代码会得到复用，很大的提升工作效率。


 
 ### 创建一个控制器###

第一步我们来创建一个中间件。在`larval`中，自带了一条命令用来创建中间件。`php artisan make:middleware CheckAge` 。我们可以用此命令创建一个`CheckAge`的中间，用来过滤用户的年龄。穿件代码如下：

```
<?php

namespace App\Http\Middleware;

use Closure;

class CheckAge
{
    /**
     * 运行请求过滤器。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }

        return $next($request);
    }

}
```
 当然在这里少不了路由的代码，我们的请求先到路由，然后到中间件最后到控制器。
```
Route::get('getuser/user/{id}/age/{age}',UserController@getUser)->middleware('CheckAge');
```
 在这段代码中我们把所有请求age大于200的重定向到`home`中，佛则就继续，这样就是实现了参数过滤。这就是中间件的基本作用。
 
# 注册中间件###

 
 我们的中间件在使用前，是需要注册的。否则我们无法正常调用到我们的中间件。这个注册文件在`app/Http/Kernel.php`中。我们打开这个文件。


```
<?php

namespace App\Http;

use Illuminate\Foundation\Http\Kernel as HttpKernel;

class Kernel extends HttpKernel
{
    // 这里注册是全局的中间件，在所有的都需要经过这个中间件来过滤信息，无论是来之web的请求还是API的///请求
    protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
    ];

    // 中间件群组，在这里我们可以注册中间件群组，用来给请求批量的添加空间。这里laravel中web请求默认     //都必须经过web群组的中间的过滤，api请求需要经过api群组的中间件过滤。使用实例在下边已经给出。
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
        ],

        'api' => [
            'throttle:60,1',
        ],
    ];
    
    // 非全局的中间件，单独给某些路由指定下面的中间件（用户自己编写）。
    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'can' => \Illuminate\Foundation\Http\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];
}
```
群组使用
```
Route::group(['middleware' => ['web']], function () {
    //
})
```

我们可以看出这个`Kernel.php`文件中有是哪个属性，分别为 `protected $middleware` `protected $middlewareGroups` `protected $routeMiddleware` 这三个属性分别用来注册`全局中间件` `群组中间件` `和自定义中间件`。

### 特殊中间件###

在中间件中用连个比较特殊的中间件，分别为`前置中间件` `和后置中间件`。顾名思义，前置中间件就是在请求前运作的，后置中间件就是在请求后运作的。这有什么用呢？我们可以在这里举一个栗子：

在正常的项目中我们肯定要记录一些数据操作记录，用来跟踪每次请求的逻辑处理。方便我们在项目出现bug时候跟踪数据。
在laravel中我们用下面函数来进行sql语句的记录：
```
DB::enableQueryLog();
DB::getQueryLog();
```

我们可以将`DB::enableQueryLog()` 放在需要记录sql语句的请求的前置中间件中，用来开启sql记录。将`DB::getQueryLog()`放在需要记录请求的后置中间件中，记录所执行的sql。

这就是中间件的作用。


### 中间件参数###

在调取中间件时，我们可以穿的附加参数。例如：
```
<?php

namespace App\Http\Middleware;

use Closure;

class CheckRole
{
    /**
     * 处理传入的请求
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```
传递指定参数可以:隔开
```
Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
```

好了，本节就到此为止了。





