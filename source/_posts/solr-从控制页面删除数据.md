---
title: solr-从控制页面删除数据
date: 2018-06-21 20:21:18
tags:
---

# 1.xml

![image](/photo/img/2018-06-21/DingTalk20180621202319.png)

```
<delete><query>*:*</query></delete>
<commit/>
```