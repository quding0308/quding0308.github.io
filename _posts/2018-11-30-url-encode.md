---
layout: post
title:  "URL编码问题"
categories: blog
---

* 目录
{:toc}

### 1. 浏览器URL 支持的字符

> 只有字母和数字[0-9a-zA-Z]、一些特殊符号"$-_.+!*'(),"[不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL。

URL中可以出现的字符： 
```
a-z、A-Z、 0-9、-_.~ 4个字符、保留字符
```

保留字符包括  
```
! * ' ( ) ; : @ & = + $ , / ? # [ ]
```

URL编码格式 采用 ASCII编码，像中文等非ASCII编码都需要转换。

不安全的字符包括规定不可以出现的ASCII符号 (空格、# % 等 保留字符) 和 非ASCII字符， 都需要做编码。

### 2.URL编码(也被称为百分号编码) 

对非ASCII字符，需要使用其他编码得到相应的字节，然后进行百分号编码。

百分号编码：使用%百分号加上两位的字符——0123456789ABCDEF——代表一个字节的十六进制形式。

例如 
```
A 在ASCII中编码为97，十六进制是 0x61 百分号编码为 %61
@ 在ASCII中编码为64，十六进制是 0x40 百分号编码为 %40
```

对于Unicode字符，RFC文档建议使用utf-8对其进行编码得到相应的字节，然后对每个字节执行百分号编码

### 3.前端URL编码处理
#### 出现浏览器编码的几种情况
- 网址路径中出现汉字
```
https://www.baidu.com/你好
```

- 查询字符串包含汉字
```
https://www.baidu.com/search?keywords=你好
```

- 网页中 发出的get或post请求，包含汉字
- Ajax调用的URL包含汉字

#### 浏览器编码-escape(),encodeURI(),encodeURIComponent()
**escape()**

escape() 不能直接用于URL编码。其作用是返回一个字符的Unicode编码。

**encodeURI()**

encodeURI() 用于对整个URL进行编码。因此除了对网址中的的保留字符不会进行编码。

例如：
```
var url = 'https://do.yzj.com/cloudwork/idx/manage/isAdmin?a=b&c=你好';
var url1 = encodeURI(url);
// url1 为 "https://do.yunzhijia.com/cloudwork/idx/manage/isAdmin?a=b&c=%E4%BD%A0%E5%A5%BD"
```

对应的解码函数为 decodeURI()

**encodeURIComponent()**

encodeURIComponent() 与 encodeURI() 的区别是，它用于对URL的组成部分进行个别编码，而不用于对整个URL进行编码。

encodeURIComponent() 会对URL中保留字符也做编码。

对应的解码函数是decodeURIComponent()

#### 结论

1. escape() 不能直接用于URL编码
2. encodeURIComponent() 不能用于整个URL编码
3. 比较好的实践：

```
param参数 使用 encodeURIComponent()编码 之后自己拼接URL
```

### 4.iOS端URL编码处理

待完成

### 参考

- [https://www.haorooms.com/post/js_escape_encodeURIComponent](https://www.haorooms.com/post/js_escape_encodeURIComponent)
- [http://www.ruanyifeng.com/blog/2010/02/url_encoding.html](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)
- [https://blog.csdn.net/zmx729618/article/details/51381655](https://blog.csdn.net/zmx729618/article/details/51381655)
