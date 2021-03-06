---
title: 基于redis的分布式锁
tags:
---

## 锁
当业务中存在多个竞争者对统一资源进行插入/更新的时候,且当前资源只允许一个操作者更改/插入时,就会存在竞争关系。如果不妥善处理竞争会导致
数据混乱。

eg:

已支付为例,现在存在A B两个操作者,账户余额为100元。

- A 减少账户50元。
- B 扣账户致账户100元。

在扣除账户余额之前,会对账户余额进行校验,如果账户余额不足,应该拒绝操作。如果此时不对资源（账户）加锁 ,会存在如下情况:

1 A操作首先对账户校验, 当前账户余额100元, 允许操作, 进行扣除操作。
2 B操作对账户进行校验, 当前账户余额100元, 允许操作, 进行扣除操作。
3 A执行扣除完毕,当前账户余额50元。
4 B执行扣除完毕,当前账户余额-50元。


在这种场景会导致当前账户余额为负数,这肯定输不允许出现的。
所以需要资源加锁。


## 分布式锁

顾名思义, 锁是分布式的, 多个服务同时公用一个锁。


## 如何实现:


锁是对某种资源的占有。

加锁:

基于redis生成一个带有过期时间的key

```php
$redis->set($lock->sourceName, $lock->key, ['nx', 'ex' => $lock->expire])

```
- `expire`为过期时间,如果超过过期时间,则自动解锁。
- `key` 生成当前锁的key,随机生成。
解锁:

具体动作: 比对当前锁和当前key是否匹配,如果匹配则解锁（删除当前redis里的key）

实现:

```php
$script = '
        if redis.call("get",KEYS[1]) == ARGV[1]
        then
            return redis.call("del",KEYS[1])
        else
            return 0
        end
    ';
$redis->eval($script,[$locks, $keys],$numKeys)
```
- 其中locks为锁名。
- $keys为秘钥。

## 异常

如果进程异常,我们需要解锁当前锁。
具体实现:

```php
 register_shutdown_function(function(){
            $lock->release();
        });
```

php中程序异常退出会调用register_shutdown_function()所注册函数,所以只要在程序异常时,解锁当前注册锁就可以了。



