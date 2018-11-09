---
title: php内存管理
date: 2018-09-12 21:28:00
tags:
---

php是如何实现内存管理的?内存管理无非包括内存分配、内存回收、以及内存使用优化。

- 内存使用的优化
- 垃圾回收机制
- 底层内存分配


## 内存使用的优化

### 引用计数

php的引用中有个引用结构体

```php
```c
struct _zend_reference {
    zend_refcondted_h gc;
    zval              val;  指向原来的value.
};
```
```
其中`zend_refcondted_h` 便是gc便是当前变量被引用的次数,这个参数会在变量回收的时候用到。


zend_refcondted_h :

```c
typedef struct _zend_refcounted_h {
	uint32_t         refcount;			/* reference counter 32-bit */
	union {
		struct {
			ZEND_ENDIAN_LOHI_3(
				zend_uchar    type,
				zend_uchar    flags,    /* used for strings & objects */
				uint16_t      gc_info)  /* keeps GC root number (or 0) and color */
		} v;
		uint32_t type_info;
	} u;
} zend_refcounted_h;
```

在实际中这个结构体到底是什么样的? 具体可以举例来看。

```php
$a = 'this is string'; // zend_array (refcount = 1)  只有$a引用了zend_array
$b = &$a; //   zend_array (refcount = 2)  $a、$b引用了zend_array
$c = $b; // zend_array (refcount = 3)  $a、$b、$c引用了zend_array
unset($b); // zend_array (refcount = 2)  $a、$c引用了zend_array
```

> 并不是所有的变量类型都会使用引用计数, 例如 `整形`、`浮点型`、`布尔型`、`NUll`(在php中这是一个变量类型)等采用了深拷贝,
即只要这几种变量赋值, 就会申请一款一块内存写入一个新的变量结构（注意这里不是引用结构）,当然这些类型不会公用value。



### 写时复制

当然只有引用计数是不够的, 因为变量会发生赋值的情况。所以在更改某一个变量时, 会对原来的变量进行拷贝并赋值。

举个栗子:

```
$foo = time();
$bar = &$b;
$si = $a;

$c = '123';
```

具体数据结构的引用计数情况如下图:



![image](/photo/img/php内存管理/写时复制.png)

## 内存回收

### 自动gc
在zend数据接口中有一个gc.refount,他是自动gc的关键。

在自动gc机制中,如果zval不在指向value且当前value的gc.refount为0时,会直接释放value。这种情况多发生于函数返回时（销毁所有局部变量）、变量修改时、以及unset操作的时候。


### 辣鸡回收

除了自动gc,还有一种是自动gc无法处理的垃圾, 这种情况称为`循环引用`。顾名思义也就是自身内部变量引用了自身, 这种情况常出现与object和array。当我该变量zval被销毁是, 与其对应的value gc.refcount -1,
但是因为有自身变量指向自身, 所以就陷入了一个循环:如果不销毁自身变量 value gc.refcount就无法自动gc, 只有自动gc才会销毁value。

```php

$a = [1];
$a[] = &$a;
unset($a);

```

![image](/photo/img/php内存管理/自身引用.png)

`unset($a)`执行以后

![image](/photo/img/php内存管理/释放.png)

由于上述情况导致无法自动gc,所以需要引入另外一钟垃圾回收机制-垃圾回收器。
现在会存在两种情况的数据需要回收：
- 当value的gc.refcount =0 是需要回收。
- 当value的gc.refcount 减少不等于0，但是存在循环引用时。

### 回收机制

当发生value gc.refcount减少时，垃圾回收机制会把可能是垃圾的value存起来， 当可能是辣鸡的value到达一定数量的时候启动垃圾鉴别程序，统一处理垃圾。当然这里的value类型只有object和array。

垃圾兼备程序：
其实垃圾鉴别程序很简单，递归遍历自身value，查看是否存在指向自身的即可。