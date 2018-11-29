---
title: 使用guzzle并行请求
date: 2018-09-28 17:10:05
tags: 压测
---

## 背景

由于拍卖项目特有的点特, 会导致在开拍是有瞬时的高流量。
后端系统会每个出价排序, 由于高qps导致排序会有重复的情况,。为了解决遮掩样的问题, 
就写了一个分布式锁, 用于解决排序冲突。用guzzle写了一个压测脚本。


## 实现

首先压测是为了同一时刻发送指定数量的请求到目标接口。
所以串行话的请求是无法
guzzle 支持用pool实现并行的请求。
官方文档 :

[官方文档支持](https://guzzle-cn.readthedocs.io/zh_CN/latest/quickstart.html?highlight=Pool)

文档中并没有提示如何发送POST请求, 顾琢磨一下:


并行方式请求:

```php
  $client = new Client(['base_uri' => '10.171.22.45:5005', 'timeout' => 10]);
    
        $requests = function ($total) {
            $uri = 'api/v1/bid';
            for ($i = 0; $i < $total; $i++) {
                yield new Request('POST', $uri, ['Content-type' => 'application/json'],json_encode([
                    'car_dealer_id'=> (string)rand(1,20000),
                    'bid_price' => "4",
                    'car_id' => "206431344",
                    'fr' => 'bd_other',
                    'platform' => 'MO',
                    'source' => 'C2B竞价',
                    'sale_type' => '卖车',
                    'car_dealer_city' => '深圳',
                    'app_version' => "3.2"
                ]));
            }
        };
    
        $pool = new Pool($client, $requests(100), [
            'concurrency' => 5,
            'fulfilled' => function ($response, $index) {
                var_dump($response->getBody()->getContents());
            },
            'rejected' => function ($reason, $index) {
                var_dump($reason->getMessage());
            },
        ]);
    
        $promise = $pool->promise();
        $promise->wait();
```
![image](/photo/img/guzzle压测/并行.png)
![image](/photo/img/guzzle压测/逻辑上并行-100次.png)
![image](/photo/img/guzzle压测/逻辑上并行-100次结束.png)


串行:

```php
        for ($i =1 ; $i<20;$i++) {
            echo time();
            $org_res[] = $this->client->request('POST', 'api/v1/bid',['form_params'=>
                [
                    'car_dealer_id'=> rand(1,20000),
                    'bid_price' => 4,
                    'car_id' => 206431332,
                    'fr' => 'bd_other',
                    'platform' => 'MO',
                    'source' => 'C2B竞价',
                    'sale_type' => '卖车',
                    'car_dealer_city' => '深圳',
                    'app_version' => 3.2
                ]]);
        }

        foreach ($org_res as $val) {
            $resdata = $val->getBody()->getContents();
        }
        var_dump($resdata);
        die;
```

请求结果:

![image](/photo/img/guzzle压测/串行.png)

![image](/photo/img/guzzle压测/逻辑上的串行-100次.png)

## 两者区别

- 从性能上来看,20次请求, 并行请求耗时2秒, 串行请求3秒。
- 当请求加到100次的时候, 并行请求耗时6秒, 串行请求耗时27秒,这个时候就能体现出并行和串行的却别相差21秒。


为什么会有这么差距,再上一张图和代码,打印出来开始结束时间。

并行:
```php
       并行请求
        echo '总的开始:'.microtime().'<br>';
        $client = new Client(['base_uri' => '10.171.22.45:5005', 'timeout' => 10]);
        $requests = function ($total) {
            $uri = 'api/v1/bid';
            for ($i = 0; $i < $total; $i++) {
                $car_dealer_id = (string)rand(1,1000000);
                echo '开始'.microtime().' car_dealer_id:'.$car_dealer_id."<br>";
                yield new Request('POST', $uri, ['Content-type' => 'application/json'],json_encode([
                    'car_dealer_id'=> (string)$car_dealer_id,
                    'bid_price' => "4",
                    'car_id' => "206431344",
                    'fr' => 'bd_other',
                    'platform' => 'MO',
                    'source' => 'C2B竞价',
                    'sale_type' => '卖车',
                    'car_dealer_city' => '深圳',
                    'app_version' => "3.2"
                ]));
            }
        };

        $pool = new Pool($client, $requests(100), [
            'concurrency' => 5,
            'fulfilled' => function ($response, $index) {
                var_dump($response->getBody()->getContents());
                echo microtime()."<br>";
            },
            'rejected' => function ($reason, $index) {
                var_dump($reason->getMessage());
                echo microtime();
            },
        ]);

        $promise = $pool->promise();

        $promise->wait();
        echo '总的结束:'.microtime().'<br>';
```

100次请求的返回结果

![image](/photo/img/guzzle压测/带时间的并行请求结果.png)

串行:

```php
     for ($i =1 ; $i<100;$i++) {
            echo '开始时间'.microtime()."<br>";
            $ret = $this->client->request('POST', 'api/v1/bid',['form_params'=>
                [
                    'car_dealer_id'=> rand(1,2000000),
                    'bid_price' => 4,
                    'car_id' => 206431332,
                    'fr' => 'bd_other',
                    'platform' => 'MO',
                    'source' => 'C2B竞价',
                    'sale_type' => '卖车',
                    'car_dealer_city' => '深圳',
                    'app_version' => 3.2
                ]]);
            $org_res[] = $ret;
            echo '结束时间'.microtime()."<br>";
            var_export($ret->getBody()->getContents());
        }

//        foreach ($org_res as $val) {
//            $resdata[] = $val->getBody()->getContents();
//        }
//        var_dump($resdata);
        die;
```

100次请求的返回结果

![image](/photo/img/guzzle压测/带时间的串行请求结果.png)


- 所以可以看出,并行请求是先生成所有请求（这个上限和concurrency 参数有关）,再等待结果。如果达到当前并行的上线,就等待请求结束再生成新的请求。
- 串行请求为请求,等待结果然后在请求。


所以如果请求量很大的话,尽量还是用并行请求。