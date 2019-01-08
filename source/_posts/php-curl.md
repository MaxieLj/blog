---
title: php curl
date: 2015-07-18 20:28:10
tags: php
categories: php
toc: true
---


### 参数1

curl_setopt 
CURLOPT_RETURNTRANSFER 表示是否直接输出到控制台
 eg:
 
```php
$curl = curl_init();
curl_setopt($curl, CURLOPT_URL, 'http://baidu.com');
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($curl, CURLOPT_HEADER, 0);
// curl_setopt($curl, CURLOPT_POST, 1);
$ret = curl_exec($curl);
// var_dump($ret);
```

输出结果为:

![image](/photo/img/php-curl/DingTalk20180718203726.png)

打开参数

```
<?php
$curl = curl_init();
curl_setopt($curl, CURLOPT_URL, 'http://baidu.com');
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 0);
curl_setopt($curl, CURLOPT_HEADER, 0);
// curl_setopt($curl, CURLOPT_POST, 1);
$ret = curl_exec($curl);
// var_dump($ret);
```

输出结果

![image](/photo/img/php-curl/DingTalk20180718204055.png)


### 参数2

CURLOPT_HEADER
CURLOPT_HEADER 表示是否输出头信息

```
<?php
$curl = curl_init();
curl_setopt($curl, CURLOPT_URL, 'http://baidu.com');
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 0);
curl_setopt($curl, CURLOPT_HEADER, 1);
// curl_setopt($curl, CURLOPT_POST, 1);
$ret = curl_exec($curl);
// var_dump($ret);
```
返回结果
![image](/photo/img/php-curl/DingTalk20180718204343.png)

```
<?php
$curl = curl_init();
curl_setopt($curl, CURLOPT_URL, 'http://baidu.com');
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 0);
curl_setopt($curl, CURLOPT_HEADER, 0);
// curl_setopt($curl, CURLOPT_POST, 1);
$ret = curl_exec($curl);
// var_dump($ret);
```
返回结果
![image](/photo/img/php-curl/DingTalk20180718204411.png)


## get 与 post

`get`
```
//初始化
$curl = curl_init();
//设置url
curl_setopt($curl, CURLOPT_URL, 'http://baidu.com');
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 0);
curl_setopt($curl, CURLOPT_HEADER, 0);
$ret = curl_exec($curl);
//关闭
curl_close($curl);
```

`post`

```
//初始化
$curl = curl_init();
//设置url
curl_setopt($curl, CURLOPT_URL, 'http://baidu.com');
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 0);
curl_setopt($curl, CURLOPT_HEADER, 0);
curl_setopt($curl, CURLOPT_POST, 1);
$ret = curl_exec($curl);
//关闭
curl_close($curl);
```