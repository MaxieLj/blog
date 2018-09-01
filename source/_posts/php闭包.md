---
title: php闭包
date: 2018-08-25 14:50:55
tags:
---
在看guzzle包的时候看大到了一段有意思的代码, 简化以后类似于下边:

```php
<?php
/**
* 
*/
class test 
{
	public  $handler;
	public function __construct(callable $handler = null)
	{
		$this->handler = $handler;
	}

	public static function create(callable $handler)
	{
		return new self($handler);
	}

	public function __invoke($request,$options)
	{
		$func = $this->handler;
	    return $func($request, $options);
	}


	public function test1()
	{
		return [$this, 'exec1'];
	}

	public function exec1($a,$b)
	{
		echo ($a+$b);
	}
}

$test = new test();



$obj = test::create($test->test1());

$obj(1,4);


```

- php文档是这样描述闭包的


`PHP是将函数以string形式传递的。 可以使用任何内置或用户自定义函数，但除了语言结构例如：array()，echo，empty()，eval()，exit()，isset()，list()，print 或 unset()。
 
 一个已实例化的 object 的方法被作为 array 传递，下标 0 包含该 object，下标 1 包含方法名。 在同一个类里可以访问 protected 和 private 方法。
 
 静态类方法也可不经实例化该类的对象而传递，只要在下标 0 中包含类名而不是对象。自 PHP 5.2.3 起，也可以传递 'ClassName::methodName'。
 
 除了普通的用户自定义函数外，也可传递 匿名函数 给回调参数。`
 
 除了上边的这些可以作为callable参数的类型,还有一个是拥有__invoke()的实例化对象。


所以实例化的对象可以使用 `[$obj,'functionName']` 当做闭包去传递。
`$obj = test::create($test->test1());`将我们的`test1`方法赋值给新实例化的对象里。


- 魔术方法`__invoke()`表示:

当尝试以调用函数的方式调用一个对象时，__invoke() 方法会被自动调用。


所以我们调用`$obj()`函数时会调用到__invoke()里,最后调用到我们一开始注册的函数。



ps
```
class test 
{
	public  $handler;
	public function __construct(callable $handler = null)
	{
		$this->handler = $handler;
	}

	public static function create(callable $handler)
	{
		return new self($handler);
	}

	public function __invoke($request,$options)
	{
		$func = $this->handler;
	    return $func($request, $options);
	}


	public function test1()
	{
		return [$this, 'exec1'];
	}

	public function exec1($a,$b)
	{
		echo ($a+$b);
	}
}

$test = new test();



$blj = test::create($test->test1());
$blj(1,4);
//测试对象是否可以作为callable类型参数


$test = test::create(new test());
```