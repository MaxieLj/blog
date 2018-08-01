---
title: 设计模式-builder template method
date: 2018-07-31 20:16:54
tags:builder、template method
---

## builder 模式

builder 建造者模式,又称生成器模式。

建造者模式是为了简化 使用多个建单的对象构建成一个复杂对象的 设计模式。 属于创建型模式。

代码示例

```php

    publish class SmallWheel()
    {
        
    }
    
    publish class BigWheel()
    {
        
    }
    
    publish class SmallCar()
    {
        publish $wheel;
    }
    
    publish class BigCar()
    {
        publish $wheel;
    }

```

以上为零件。

```php
    
    publish CarBuilder()
    {
        //创建小轿车
        publish function getSmallCar()
        {
            $smallCar = new SmallCar();
            $smallCar->whell = new SmallWheel();
            return $smallCar;
        }
        
        //创建大轿车
        publish function getBigCar()
        {
            $bigCar = new BigCar();
            $bigCar->whell = new BigWheel();
            return $bigCar;
        }
    }
```

调度 

```php

    $carBuilder = new CArBuilder();
    
    $smallCar = $carBuilder->getSmallCar();
    $bigCar = $carBuilder->getBigCar();
```


备注:

建造者模式与工厂模式区别。


## template method 

模板方法:定义一个抽象类或者/父类,由继承它的子类按照需求重写它的方法实现。

eg:

```php

    publish abstract class car()
    {
        //强制子类实现
        abstract protect function run();
        abstract protect function turnOnTheLight();
        //不强制
        public function palyMusic()
        {
            return 'hengheng hahei';
        }
    }
    
```
以上
