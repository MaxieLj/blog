---
title: 一次HTTP请求
date: 2017-11-20 20:09:23
tags:
---

#一次完整的HTTP请求，以及请求方式的区别

-----
今天在对接业务组mas时候遇到一个很少遇到的问题，发送一个一个不但参数的POST的请求。
在我的印象里传参好像必带名字，因为就像一个你不认识的人要来你家，你肯定不会让他进来的。
所以重新缕了一下POST请求，冲冲能。

##一次HTTP请求的过程

访问某个网址，例如我们访问www.baidu.com
浏览器会和百度的服务器进行三次握手（中间的域名解析等暂不考虑）
然后是程序层面的信息请求与提交

三次握手->get或者post请求

1） Client首先发送一个连接试探，ACK=0 表示确认号无效，SYN = 1 表示这是一个连接请求或连接接受报文，同时表示这个数据报不能携带数据，seq = x 表示Client自己的初始序号（seq = 0 就代表这是第0号包），这时候Client进入syn_sent状态，表示客户端等待服务器的回复

2） Server监听到连接请求报文后，如同意建立连接，则向Client发送确认。TCP报文首部中的SYN 和 ACK都置1 ，ack = x + 1表示期望收到对方下一个报文段的第一个数据字节序号是x+1，同时表明x为止的所有数据都已正确收到（ack=1其实是ack=0+1,也就是期望客户端的第1个包），seq = y 表示Server 自己的初始序号（seq=0就代表这是服务器这边发出的第0号包）。这时服务器进入syn_rcvd，表示服务器已经收到Client的连接请求，等待client的确认。

3） Client收到确认后还需再次发送确认，同时携带要发送给Server的数据。ACK 置1 表示确认号ack= y + 1 有效（代表期望收到服务器的第1个包），Client自己的序号seq= x + 1（表示这就是我的第1个包，相对于第0个包来说的），一旦收到Client的确认之后，这个TCP连接就进入Established状态，就可以发起http请求了。

## post请求
HTTP 协议是以 ASCII 码传输，建立在TCP/IP协议之上的应用层规范，规范把 HTTP 请求分为三个部分：状态行、请求头、消息主体。类似于下面这样：
<method> <request-url> <version>
<headers>
<entity-body></entity-body></headers></version></request-url></method>
协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。

但是，数据发送出去，还要服务端解析成功才有意义。一般服务端语言如 php、python 等，以及它们的 framework，都内置了自动解析常见数据格式的功能。服务端通常是根据请求头（headers）中的 Content-Type 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。所以说到 POST 提交数据方案，包含了 Content-Type 和消息主体编码方式两部分。下面就正式开始介绍它们。


header
Content-Type，内容类型，一般是指网页中存在的Content-Type，用于定义网络文件的类型和网页的编码，决定文件接收方将以什么形式、什么编码读取这个文件.
当然还有UA等其他头信息，

Content-Type对照表：http://tool.oschina.net/commons

说几种常见的Content-Type

1.application/x-www-form-urlencoded
这个就是我们常见form表格提的方式

2.multipart/form-data

这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 form 的 enctyped 等于这个值。直接来看一个请求示例

3.application/json

application/json 这个 Content-Type 作为响应头大家肯定不陌生。实际上，现在越来越多的人把它作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串。由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。

so 用这种格式就可以传递非Key Val的数据
