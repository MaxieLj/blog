---
title: 如何建立一个composer私有包
date: 2017-09-28 17:11:00
tags: composer
categories: 工具
toc: true
---


最近发现分布式锁在很多项目都有所用到,所以想把之前完成的代码封装成pear包,供大家使用。
但是发现自己对composer并不是很熟悉,所以学习一番,在此记录以防自己忘记。



## 新建仓库

$ git clone git@gitlab.renrenche.com:web/jugg.git
$ cd jugg


## conmposer 初始化

composer init 

```
//包名
Package name (<vendor>/<name>) [lijian/test_composer]: jugg/jugg
//描述
Description []: 基于redis的分布式锁
//作者
Author [, n to skip]: Maxie lj <xiaolijian110@163.com>
//最小稳定版本
Minimum Stability []: dev
//遵循协议
License []: MIT

//依赖
Would you like to define your dependencies (require) interactively [yes]? yes

Search for a package: php

Enter the version constraint to require (or leave blank to use the latest version):
Using version ^7.0 for php
```


生成composer.json文件

```
{
    "name": "jugg/jugg",
    "description": "基于redis的分布式锁",
    "type": "library",
    "require": {
        "php": "^7.0"
    },
    "require-dev": {
        "php": "7"
    },
    "license": "MIT",
    "authors": [
        {
            "name": "Maxie lj",
            "email": "xiaolijian110@163.com"
        }
    ],
    "minimum-stability": "dev"
}
```

## 生成自动加载（psr-4）

在composer.json 文件里添加
```
{
    "name": "jugg/jugg",
    "description": "基于redis的分布式锁",
    "type": "library",
    "license": "MIT",
    "authors": [
        {
            "name": "Maxie lj",
            "email": "xiaolijian110@163.com"
        }
    ],
    "autoload": {
    "psr-4": {
      "Jugg\\": "jugg"
    }
    },
    "minimum-stability": "dev"
}

```
执行`composer install`

就会生成一个自动加载文件

![image](/photo/img/创建一个composer包/自动加载目录.png)

然后我们就可以按照psr-4规范开发我们pear包了。



## 测试

我们可以在测试文件里引入autoload.php文件,这样就实现了自动加载。

```php
require './vendor/autoload.php';
```

这样就可以正常的测试了


## 上传代码到gitlab或者github

当我们开完的时候,需要将我们的代码上传到远程代码库。

需要注意的事 一定不要讲.git文件上传。


## 其他项目引入

因为没有上传到pagelist,所以我们暂时是公司私有包。

如果需要再项目里引入私有包,需要在项目的composer.json文件里添加。

```
"jugg": {
        "type":"package",
        "package": {
          "name": "jugg",
          "version": "v1.1",
          "source": {
            "type": "git",
            "url": "git@gitlab.renrenche.com:web/jugg.git",
            "reference": "master"
          },
          "autoload": {
            "psr-4": {
              "Jugg\\": "jugg"
            }
          }
        }
      }
```

完整文件:

```
{
    "name": "laravel/lumen",
    "description": "The Laravel Lumen Framework.",
    "keywords": [
        "framework",
        "laravel",
        "lumen"
    ],
    "license": "MIT",
    "type": "project",
    "repositories": {
      "packagist": {
        "type": "composer",
        "url": "https://packagist.phpcomposer.com"
      },
        "0": {
            "type": "package",
            "package": {
                "name": "sdk/clusterproxy",
                "version": "1.0.3",
                "source": {
                    "type": "git",
                    "url": "git@gitlab.renrenche.com:sdk/clusterproxy.git",
                    "reference": "master"
                },
                "autoload": {
                    "psr-4": {
                        "Cluster\\": "src/Cluster"
                    }
                }
            }
        },
      "1": {
        "type": "package",
        "package": {
          "name": "sdk/dingtalk-alarm-php-sdk",
          "version": "1.0",
          "source": {
            "type": "git",
            "url": "git@gitlab.renrenche.com:sdk/dingtalk-alarm-php-sdk.git",
            "reference": "master"
          },
          "autoload": {
            "psr-4": {
              "Rrc\\": "src/Rrc/"
            }
          }
        }
      },
      "2": {
        "type":"package",
        "package": {
          "name": "sdk/prometheus-php-sdk",
          "version": "1.0",
          "source": {
            "type": "git",
            "url": "git@gitlab.renrenche.com:sdk/prometheus-php-sdk.git",
            "reference": "master"
          },
          "autoload": {
            "psr-4": {
              "Rrc\\": "src/Rrc/"
            }
          }
        }
      },
      "jugg": {
        "type":"package",
        "package": {
          "name": "jugg",
          "version": "v1.1",
          "source": {
            "type": "git",
            "url": "git@gitlab.renrenche.com:web/jugg.git",
            "reference": "master"
          },
          "autoload": {
            "psr-4": {
              "Jugg\\": "jugg"
            }
          }
        }
      }
    },
    "require": {
        "php": ">=7.0",
        "laravel/lumen-framework": "5.5.*",
        "vlucas/phpdotenv": "~2.2",
        "guzzlehttp/guzzle": "^6.2",
        "sdk/clusterproxy": "^1.0",
        "illuminate/redis": "^5.3",
        "peixinchen/mns": "^1.0",
        "mockery/mockery": "^0.9.5",
        "firebase/php-jwt": "^4.0",
        "sdk/dingtalk-alarm-php-sdk": "^1.0",
        "sdk/prometheus-php-sdk": "^1.0",
        "predis/predis": "^1.1",
        "solarium/solarium": "^3.8",
        "mongodb/mongodb": "^1.3",
        "jugg": "^1.0"
    },
    "require-dev": {
        "fzaninotto/faker": "~1.4",
        "phpunit/phpunit": "~6.0",
        "mockery/mockery": "~0.9",
        "phpstan/phpstan": "^0.9.2"
    },
    "autoload": {
        "psr-4": {
            "Rrc\\": "./"
        }
    },
    "autoload-dev": {
        "classmap": [
            "tests/",
            "database/"
        ]
    },
    "scripts": {
        "post-root-package-install": [
            "php -r \"copy('.env.example', '.env');\""
        ]
    },
    "minimum-stability": "dev",
    "prefer-stable": true,
    "config": {
        "optimize-autoloader": true
    }
}

```


然后执行`composer require jugg<包名>` 就可以引入到项目里了。 