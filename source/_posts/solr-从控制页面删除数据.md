---
title: solr-增删改查
date: 2018-06-21 20:21:18
tags: solr
toc: true
---
## 背景
solr 版本 5.3.1
### 1.删除数据
solr 提供多重删除数据方式, 从控制页删除数据是一种方式,主要用于人工删除少量数据
eg:
![image](/photo/img/solr-增删改查/DingTalk20180621202319.png)

执行删除操作之前:

![image](/photo/img/solr-增删改查/执行删除前数据.png)

删除操作只需在控制页Document Type 选择XML选项,然后输入一下内容提交即可。

```
<commit/>
<delete><query>id:change.me1</query></delete>
<commit/>
```

具体执行,以及http请求:

![image](/photo/img/solr-增删改查/执行删除操作.png)

最后为执行删除后的结果:

![image](/photo/img/solr-增删改查/执行删除操作结果.png)


### 新增数据

#### json 方式添加数据

同样在solr的控制页面,选在Document Type为json。

输入:

```
{"id":"ceshi2","title":"这是一个测试"}
```


![image](/photo/img/solr-增删改查/新增数据.png)

可以看出来solr是已http请求的方式请求 solr server,所以我们用程序去查询solr数据是,也可json的方式去查询。
当然solr支持xml等多种功能。（solr支持从数据库导入数据）
具体查询规则（目前还没找到,找到后补充）:

执行结果

![image](/photo/img/solr-增删改查/执行结果.png)


### 更新数据

第一个问题:solr不支持局部更新。
所有的更新操作对于solr来说都是一次数据的删除和插入,例如:

![image](/photo/img/solr-增删改查/更新操作.png)

同事又说支持局部更新,但是我还没有找到相关文档,这个带求证