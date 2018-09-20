---
layout: post
title:  "跨域问题总结"
date:   2018-06-22 15:40:05 +0800
categories: blog
---

* 目录
{:toc}

### 同源策略
两个页面地址中的协议、域名和端口号一致，则表示同源。

### 跨域请求
凡是发送url的scheme、host、port任一与当前页面地址不同，即为跨域。

### 为了安全，同源策略有以下限制：
- 存储在浏览器中的数据，如cookie、localStorage、indexDB不能通过脚本跨域访问。
- 不能通过脚本操作不同域下的dom
- Ajax不能跨域请求数据

***

### Ajax跨域解决方案：
- jsonp 可以跨域发起get请求
- iframe
- 后台服务器做代理(先将请求发送给后台服务器，通过服务器来发送请求，然后将请求的结果传递给前端。)
- CORS (Cross-Origin Resource Sharing) W3C官方解决方案

### CORS协议

浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。

#### CORS协议中的OPTIONS请求

OPTIONS 是 HTTP/1.1 协议中定义的方法，用以从服务器获取更多信息。    

OPTIONS Request会携带两个首部字段：   

    // 告知server，实际请求是POST
    Access-Control-Request-Method: POST
    // 告知Server，实际请求会携带两个自定义首部字段
    Access-Control-Request-Headers: X-PINGOTHER, Content-Type

OPTIONS Response响应：

    // 允许指定域名进行跨域请求
    Access-Control-Allow-Origin: http://foo.example
    // 允许实际请求发起的METHOD
    Access-Control-Allow-Methods: POST, GET, OPTIONS
    // 允许携带的header
    Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
    // 该响应有效时间，该时间内无法再次发起OPTIONS请求
    Access-Control-Max-Age: 86400

OPTIONS成功请求后，会发送实际请求。

跨域请求，会有一个请求头Origin，记录当前页面的域名。

参考
- CORS官方介绍：<https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS>
- <https://www.cnblogs.com/dojo-lzz/p/4265637.html>
- <https://blog.csdn.net/hehexiaoxia/article/details/61916737>
