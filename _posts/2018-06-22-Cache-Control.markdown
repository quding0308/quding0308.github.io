---
layout: post
title:  "Cache-Control使用总结"
date:   2018-06-22 10:05:05 +0800
categories: blog http
---
目录
<!-- TOC -->

- [Request中设置Cache-Control](#request%E4%B8%AD%08%E8%AE%BE%E7%BD%AEcache-control)
- [Response中设置Cache-control](#response%E4%B8%AD%E8%AE%BE%E7%BD%AEcache-control)
- [iOS中Request设置cachePolicy](#ios%E4%B8%ADreques%08t%E8%AE%BE%E7%BD%AEcachepolicy)
- [WKWebvView与NSURLRequestCachePolicy配合使用](#wkwebvview%E4%B8%8Ensurlrequestcachepolicy%08%E9%85%8D%E5%90%88%E4%BD%BF%E7%94%A8)
    - [第一种情况](#%E7%AC%AC%E4%B8%80%E7%A7%8D%E6%83%85%E5%86%B5)
    - [第二种情况](#%E7%AC%AC%E4%BA%8C%E7%A7%8D%E6%83%85%E5%86%B5)
    - [第三中情况](#%E7%AC%AC%E4%B8%89%E4%B8%AD%E6%83%85%E5%86%B5)
- [ETag的使用](#etag%E7%9A%84%E4%BD%BF%E7%94%A8)
- [Last-Modified 和 If-Modified-Since](#last-modified-%E5%92%8C-if-modified-since)

<!-- /TOC -->
***

### Request中设置Cache-Control
- no-cache
- no-store
- max-age
- max-stale
- min-fresh
- no-transform
- only-if-cached
- cache-extension

### Response中设置Cache-control
- public
- private
- no-cache
- no-store
- no-transform
- max-age(单位s)
- must-revalidate
- s-maxage
- cache-extension

### iOS中Request设置cachePolicy

    // NSURLRequestCachePolicy
    // 使用网络协议中实现的缓存策略(Cache-Control)
    NSURLRequestUseProtocolCachePolicy

    // 不使用本地缓存，从server加载。如果一个资源设置了 setHeader('Cache-Control', 'public, max-age=3600')，可以用这种策略强制重新加载
    NSURLRequestReloadIgnoringLocalCacheData

    // 无论缓存是否过期，先使用本地缓存。如果没有缓存，才从server加载
    NSURLRequestReturnCacheDataElseLoad

    // 无论缓存是否过期，先使用本地缓存数据。如果缓存中没有请求所对应的数据，那么放弃从原始地址加载数据，请求视为失败
    NSURLRequestReturnCacheDataDontLoad


### WKWebvView与NSURLRequestCachePolicy配合使用   
#### 第一种情况   
html资源设置Cache-Control='public, max-age=3600'，此时WKWebView第一次加载该html时会使用本地缓存，重新load时，会忽略Cache-Control，从server请求。WKWebView的request如果设置为NSURLRequestReloadIgnoringLocalCacheData，则html的第一次请求也会忽略本地缓存，从server请求。
#### 第二种情况
html资源设置Cache-Control='public, max-age=0'，此时WKWebView默认每次都会从server请求。如果WKWebView的请求设置为NSURLRequestReturnCacheDataElseLoad，则第一次请求会本地缓存，之后重新reload会请求server。
#### 第三中情况
WKWebView的request设置为NSURLRequestReturnCacheDataDontLoad，如果本地没有缓存则第一次加载失败，reload会从server请求。如果本地有缓存，第一次会使用本地缓存，之后reload会请求server。

**WKWebView中请求一个url，第一次加载会使用Cache-Control策略，之后在同一个WKWebView中在此加载这个url时Cache-Control策略无效，会请求server**

### ETag的使用
response中的header中返回的字段"ETag"="af-k59S5WU6TaRZGFycLip4kzlBkQ8"，由server产生，作为资源实体的hash值 返回给client。

发起reqeust时，可以在header中设置If-None-Match字段，把之前的ETag字段传给server，server比较ETag字段，如果相等则返回304，告知client使用本地缓存（不用返回数据，节省流量，加快请求速度）。

ETag的优先级高于Last-Modified

304 Not Modified

### Last-Modified 和 If-Modified-Since

Last-Modified与Etag类似，Last-Modified表示响应资源在服务器最后修改时间。

response 返回Last-Modified，资源的最后修改时间。
request 在header中带参数If-Modified-Since，把上一次获取的Last-Modified数据返回给server，server
通过比较时间，决定是否返回304.

Last-Modified相比ETag有比几个不足：
- Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间
- 如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存



参考：     
- <http://nshipster.cn/nsurlcache/>
- <https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9>
- <https://www.cnblogs.com/softidea/p/5986339.html>