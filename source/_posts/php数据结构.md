---
title: php数据结构
date: 2018-08-19 20:30:34
tags: php源码
categories: php源码学习
toc: true
---

## 变量
   
php是如何统一实现各种变量类型的? 这和php变量类型的实现是密不可分的。
一个变量主要有三个要素:`变量名``变量类型``变量值`,他们在php变量类型实现中主要对应 zval zend_value 和
zend的各种数据类型。
php中用$去声明一个变量,在声明的时候可以进行初始化,当然也可以不声明直接使用。一般来说,一个变量的定义包含
两步:变量定义和变量初始化,在php中只定义不初始化是可以的。

## 变量结构体

```c
// php zval结构
struct _zval_struct {
	zend_value        value; 指向具体的value			/* value */
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar    type,			/* active type */
				zend_uchar    type_flags,
				zend_uchar    const_flags,
				zend_uchar    reserved)	    /* call info for EX(This) */
		} v;
		uint32_t type_info;
	} u1;
	union {
		uint32_t     var_flags;
		uint32_t     next;                 /* hash collision chain */
		uint32_t     cache_slot;           /* literal cache slot */
		uint32_t     lineno;               /* line number (for ast nodes) */
		uint32_t     num_args;             /* arguments number for EX(This) */
		uint32_t     fe_pos;               /* foreach position */
		uint32_t     fe_iter_idx;          /* foreach iterator index */
	} u2;
};
```


## 变量类型
其中type 表示当前变量的类型,例如string array int等,以下为php中全部变量类型。
其中type 是一个无符号类型的char,他的定义是这样的
```c
typedef unsigned char zend_uchar;
```


```c
typedef union _zend_value {
    	zend_long         lval;				/* long value */
    	double            dval;				/* double value */
    	zend_refcounted  *counted;
    	zend_string      *str;
    	zend_array       *arr;
    	zend_object      *obj;
    	zend_resource    *res;
    	zend_reference   *ref;
    	zend_ast_ref     *ast;
    	zval             *zv;
    	void             *ptr;
    	zend_class_entry *ce;
    	zend_function    *func;
    	struct {
    		uint32_t w1;
    		uint32_t w2;
    	} ww;
    } zend_value;
```

- 在这些类型里不存在boolean类型,因为在php7中已经将boolean拆分为true和false.直接保存在type中,也就是说
boolean没有_zend_value结构,直接通过type字段就可以确定boolean的值。
- 从上边的结构体可以看出来 `zend_long`、`double` 类型不是指针类型,也就是说整形和浮点型正能进行深拷贝,
并不能和其他说句类型一样进行 `引用计数`和`写时复制`。因为有`引用计数`和`写时复制`在变量赋值且不做修改时
才能大量节省内存。


以string类型为例,它在php中的结构类型应该是这样的:

![image](/photo/img/php数据结构/php数据结构.png)

_zend_string:
```c
struct _zend_string {
	zend_refcounted_h gc;
	zend_ulong        h;                /* hash value */
	size_t            len;
	char              val[1];          /*字符串起始地址*/
};
```

- 其中`gc` 字段表示这个结构体被引用的次数,垃圾回收的时候会用到。
- h 字符串通过Times33计算出来的hashcode
- len 字符串长度
- val 字符串内容

我们在这里都会有疑问,为什么存储字符串内容的时候没有使用char*类型,而用了一个数组类型来存储。zend_string
在申请内存的时候会会申请malloc(sizeof(zend_string)+字符串长度),且val[0]存的是字符串最后一个字符安“\0”

zend_string结构在内存中如下所示:


![image](/photo/img/php数据结构/zend_stirng内存中结构.png)

当时也在char val[1]的地方疑惑了很久,再往上找了很多文章,其中
Nikita Popov(nikic，PHP 官方开发组成员，柏林科技大学的学生) 这样解释

```
如果你对 C 语言了解的不是很深入的话，可能会觉得 val 的定义有些奇怪：这个声明只有一个元素，但是显然我们想存储的字符串长度肯定大于一个字符的长度。这里其实使用的是结构体的一个『黑』方法：在声明数组时只定义一个元素，但是实际创建 zend_string 时再分配足够的内存来存储整个字符串。这样我们还是可以通过 val 访问完整的字符串。
```


## 参考:

- [Internal value representation in PHP 7 - Part 2 ](https://nikic.github.io/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html)
- [[译]变量在 PHP7 内部的实现（二）](https://0x1.im/blog/php/Internal-value-representation-in-PHP-7-part-2.html)
- [php内核分析——2.1 变量的内部实现](https://www.kancloud.cn/nickbai/php7/363268)
