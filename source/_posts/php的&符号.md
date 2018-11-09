---
title: 'php的引用'
date: 2018-08-12 22:18:07
tags:
---

### &

&是php中的引用,他并不一种独立的数据类型,而是指向其他数据类型的数据结构,和C语言的指针类似。

具体作用为:当我们修改引用类型变量是,其修改的将作用于实际引用的变量上。

如果当我们使用&符号生成一个引用变量时,首先会为&操作生成一个zend_reference结构,zend_reference就是引用
类型的结构体,他内嵌一个zval,然后此结构的体zval的value指向原来zval的value上。然后将原来zval的变量
类型变为IS_REFERENCE,原来zval的value执行新创建zend_reference结构。
索引&的作用就是新生成zend_reference,然后将其指向原来的zval的value上,顺便把原来zval的变量类型变化为引用类型。

```c
struct _zend_reference {
    zend_refcondted_h gc;
    zval              val;  指向原来的value.
};
```

### 举个栗子

```
    $a = time(); //步骤1
    $b = &$a;    //步骤2
```

**步骤1**

现在`$a = time()`是他们的数据结构指向为:

![image](/photo/img/php引用/未引用前.png)

此时数据结构,一直指针指向是这个样子。

**步骤2**

![image](/photo/img/php引用/引用后.png)


也就是说PHP的引用在使用后新生成一个引用类型机构,然后将引用类型结构指向越来的value,最后将原来zval的value
指向新生成的结构体,并将其变量类型变为引用类型,IS_REFERENCE。当然随之改变的还有计数。
