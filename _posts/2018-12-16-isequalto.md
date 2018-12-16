---
layout: post
title:  "isEqual ，isEqualToString ， == 三者的区别"
categories: blog
---

* 目录
{:toc}

### ==

== 用于直接比较 两个对象的内存地址。

### isEqual 与 isEqualToString: 比较

isEqual 是 NSObject 的方法 ，而 isEqualToString 是 NSString 的方法。

isEqual 是 NSObject 的方法，默认实现 等价于 “==”

我们可以 重写 isEqual 方法，同时 也建议 重写 hashCode 的实现。

类似 isEqualToString: 的方法：
```
-[NSArray isEqualToArray:]
-[NSDate isEqualToDate:]
-[NSDictionary isEqualToDictionary:]
-[NSNumber isEqualToNumber:]
-[NSSet isEqualToSet:]
-[NSString isEqualToString:]
-[NSValue isEqualToValue:]
```

在 NSString 中 isEqual 的实现：
```
- (BOOL)isEqual:(id)object {
    return [self isEqualToString:object];
}
```
所以 在 NSString 中  isEqual 与 isEqualToString 作用一样。

同理，在 NSArray 中，isEqual 与 isEqualToArray 作用一样。

### 参考

- https://stackoverflow.com/questions/1112373/implementing-hash-isequal-isequalto-for-objective-c-collections?rq=1
- https://www.jianshu.com/p/1d05bf21d8f9