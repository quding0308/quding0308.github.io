---
layout: post
title:  "从fir下载安装包ipa"
categories: blog
---

* 目录
{:toc}

### 找到下载地址

例如：

```
https://fir.im/yzj1015
```
### 查找 id 和 token

从刚才的下载地址找到 "yzj1015" 拼接新的地址
```
https://download.fir.im/yzj1015
```

这个地址获取到的是 json 数据 从里面找到 id 和 token 

### 下载另一个文件

https://download.fir.im/v2/app/install/appid?download_token=token


### 打开下载的文件(文本) 查找到 ipa 链接 

```
<string><![CDATA[https://pro-bd.fir.im/6e84728d0dbd8ddde8f96ebbb96dfd1c030ca7bf?auth_key=1544670514-0-9b1583649b4a4026bdaa53857e31cd6e-e222907055f72430a75bca4e9f96e60a]]></string>

```