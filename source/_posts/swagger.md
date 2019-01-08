---
title: swagger
date: 2018-08-06 15:30:13
tags: pear
toc: true
---

swagger 是什么鬼? 
swagger像是个接口生成、管理、mock、测试的框架。主要功能:
 - 支持API自动生成同步的在线文档
 - 这些文档可用于项目内部API审核
 - 方便测试人员了解API
 - 这些文档可作为客户产品文档的一部分进行发布
 - 可以mock接口方便调试
支持API规范生成代码，生成的客户端和服务器端骨架代码可以加速开发和测试速度
## 
swagger-ui 一个是一个管理接口的服务,它可以根据swagger.json模拟接口返回。
### 搭建swagger-ui

克隆swagger-ui

`git clone https://github.com/swagger-api/swagger-ui.git`

配置nginx 服务

```
 server {
        listen       8090;
        server_name  www.swagger-ui.com;
        autoindex on;
        #charset koi8-r;

       # access_log  logs/host.access.log  main;
        root  /Users/lijian/Desktop/code/web_code/swagger/swagger-ui/dist;
        location / {

            index  index.php index.html index.htm;
            try_files $uri $uri/ /index.php?$query_string;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location ~ \.php$ {

            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info    ^(.+\.php)(/.+)$;
            fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

    }
```

重启NGINX 
`nginx -s reload`

样子如下:

![image](/photo/img/swagger-ui/WX20180807-095949.png)


## swagger.json

swagger.json,swagger-ui能够将swagger.json解析为对应的可处理的接口文档。

效果如上图所示。
最终要的一点在于 `http://localhost:8091/swagger.json`该url为被解析的json地址。

当然如果我们需要管理不同项目的项目,可以单独配置个服务用来管理swagger.json。


## swagger-php

github 地址: `https://github.com/zircote/swagger-php`

首先我们需要在项目里引入swagger-php 扩展包。

`composer global require zircote/swagger-php`

按照swagger文档写swagger备注:
eg:

```SWG
    /**
     * @SWG\Post(
     *     path="/guestbook/appmsg",
     *     summary="访客留言",
     *     tags={"new", "guests"},
     *     description="访客留言",
     *     operationId="appmsg",
     *     @SWG\Parameter(
     *         description="msg",
     *         format="string",
     *         in="formData",
     *         name="msg",
     *         required=true,
     *         type="string",
     *     ),
     *     @SWG\Parameter(
     *         description="email",
     *         format="string",
     *         in="formData",
     *         name="email",
     *         required=true,
     *         type="string",
     *
     *     ),
     *     consumes={"multipart/form-data", "application/x-www-form-urlencoded"},
     *     produces={"application/json"},
     *     @SWG\Response(
     *         response="200",
     *         description="返回成功",
     *     ),
     * )
     *
     */
    
    /**
     *   @SWG\Get(
     *     path="/get/feedback",
     *     summary="留言表",
     *     tags={"getList"},
     *     descriptionId="appmsglist",
     *     @SWG\Parameter(
     *          description="Id",
     *          format="integer",
     *          in="formData",
     *          name="user_id",
     *          required="true",
     *          type="integer"
        *      ),
     *      @SWG\Parameter(
     *          description="phone",
     *          format="integer",
     *          in="formData",
     *          name="user_id",
     *          required="true",
     *          type="integer"
     *        ),
     *     consumes={"multipart/form-data", "application/x-www-form-urlencode"},
     *     produces={"application/json"},
     *     @SWG\Response(
     *        response="200",
     *        description="返回成功",
     *     )
     * )
     *
     */
```

写完以后运行:

`./vendor/zircote/swagger-php/bin/swagger app/Business/Hera/HeraMockApi.php -o ./../../swagger_json/rrc/swagger.json
`
第一个为swagger.json的生成器,第二个为目标文件,第三个为生成文件。

然后我们在swagger-ui 引入生成的json即可。


