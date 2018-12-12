---
layout: post
title:  "WKWebView使用"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

* 目录
{:toc}

### WKWebViewConfiguration

初始化WKWebView时，设置配置参数。

```
WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];


config.userContentController = [[WKUserContentController alloc] init];
```

### WKUserContentController

有三个作用：
#### WKUserScript 管理

向web页面注入JS

```
api:
- (void)addUserScript:(WKUserScript *)userScript;
- (void)removeAllUserScripts;

// 注入时机
WKUserScriptInjectionTimeAtDocumentStart
WKUserScriptInjectionTimeAtDocumentEnd

```

#### WKScriptMessageHandler 管理

向web页面暴露native api

```
api:
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
- (void)removeScriptMessageHandlerForName:(NSString *)name;

WKScriptMessageHandler 中接收到回调时，数据封装在 WKScriptMessage 中。

WKScriptMessage
    name // NSString
    body // NSNumber, NSString, NSDate, NSArray, NSDictionary, and NSNull
```

JS 中调用 

```
window.webkit.messageHandlers.helloNative.postMessage({title:'测试',content:'测试分享的内容',url:'http://www.baidu.com'});
```


#### WKContentRuleList 管理
从 iOS11 后提供。可以对WebView的内容添加一些自定义的过滤规则，可以通过配置规则拦截页面里的资源请求、隐藏页面里的指定元素、将http请求转换成https。

参考：https://www.jianshu.com/p/8af24e9dc82e

```
api:
- (void)addContentRuleList:(WKContentRuleList *)contentRuleList
- (void)removeContentRuleList:(WKContentRuleList *)contentRuleList

NSString *rules = @"[{\"trigger\": {\"url-filter\": \".*\" },\"action\": {\"type\": \"make-https\"}}]";
[WKContentRuleListStore.defaultStore compileContentRuleListForIdentifier:@"ContentBlockingRules" encodedContentRuleList:rules completionHandler:^(WKContentRuleList *list, NSError *error) {
    if (!error) {
        [controller addContentRuleList:list];
    }
}];

```





### 参考
