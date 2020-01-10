---
layout: post
title:  "iOS Runtime 相关"
categories: blog
---

* 目录
{:toc}

## 编译运行 objc4

参考：

https://www.imooc.com/article/268031

使用 Xcode 10.1 运行：

https://github.com/quding0308/objc4-750


## objc_getClass 与 [obj class]] 方法获取有什么别区别？

objc_getClass(name) 会去存储 class 的全局哈希表中根据 name 查找 class

[Object class] 会调用  object_getClass(obj) 会返回 isa 指针指向的类


## isEqual、==、isEqualToString

== 用于比较基础数据的值，或者 比较对象的内存地址。两个对象只有指向一个地址，== 才为 YES。两个 nil 比较，结果为 YES

isEqualToString 用于比较两个字符串的值是否相等，如果两个字符串为 nil ，结果为 NO

isEqual 用于检测两个 Object 是否相同，可以重写 isEqual 实现自己的逻辑

``` java
NSString *str1  = [[NSString alloc] initWithString:@"hello"];
NSString *str2  = [[NSString alloc] initWithString:@"hello"];
NSString *str3  = [[NSMutableString alloc] initWithString:@"hello"];
NSString *str4  = [[NSMutableString alloc] initWithString:@"hello"];

if(str1 == str2) { // YES  两个对象指向同一个字符串常量
    NSLog(@"== ");
}

if(str1 == str3) { // NO
    NSLog(@"== ");
}

if(str3 == str4) { // NO
    NSLog(@"== ");
}

if([str1 isEqual:str2]){ // YES  等价于 isEqualToString
    NSLog(@" isEqual ");
}

if([str3 isEqual:str4]){    // YES  等价于 isEqualToString
    NSLog(@"isEqual ");
}

```

## Dictionary 的 key 需要满足什么条件

The key is copied (using copyWithZone:; keys must conform to the NSCopying protocol). 

key 是一个 id 类型，实际是一个指向对象的指针。该对象的类必须实现 NSCopying ，重写 copyWithZone: 方法

Object 作为 key 时，会通过 isEqual: 来判断 key 是否相同。如果 isEqual: 相同，则会覆盖之前的值

key 和 value 不可以为 nil ，否则会闪退

value 可以使用 [NSNull null] 来表示 nil

## 深拷贝、浅拷贝 copy mutableCopy

- Shallow Copy 浅拷贝值复制对象本身，对象里的属性、包含的对象不做复制
- Deep Copy 深拷贝则既复制对象本身，对象的属性也会复制一份

对普通对象 ObjectA 进行 copy，无论浅拷贝还是深拷贝，都会复制出一个新的对象 ObjectB，只是浅拷贝时 ObjectA 与 ObjectB 中的 textColor 指针还指向同一个 NSColor 对象，而深拷贝时 ObjectA 和 ObjectB 中的 textColor 指针分别指向各自的 NSColor 对象（NSColor 对象被复制了一份）。

Foundation中支持复制的类，默认是浅复制。常用的可复制对象有：NSNumber、NSString、NSMutableString、NSArray、NSMutableArray、NSDictionary、NSMutableDictionary。

对象拥有复制特性，须实现NSCopying，NSMutableCopying协议，实现该协议的CopyWithZone：方法或MutableCopyWithZone：方法。

runtime 源码中的默认实现：
``` java
- (struct _NSZone *)zone {
    return (struct _NSZone *)_objc_rootZone(self);
}
- (id)mutableCopyWithZone:(struct _NSZone *)zone {
    return (id)self;
}
+ (id)copyWithZone:(struct _NSZone *)zone {
    return (id)self;
}
- (id)copy {
    return [(id)self copyWithZone:nil];
}
- (id)mutableCopy {
    return [(id)self mutableCopyWithZone:nil];
}
```

## NSString NSMutableString 使用 strong copy

写 property 的时候 NSString 使用 copy ，NSMutableString 使用 strong

NSString 类型的属性用 strong 修饰，当给它传一个 NSMutableString 类型的数据时，它的指针指向了该数据，之后会随 NSMutableString 的变化而变化

NSMutableString 如果使用 copy 修饰，在 setter 中会有 [name copy] 的操作，实际是一个不可变的 string 。如果调用 appendString 会直接闪退。 


## isSubclassOfClass isMemberOfClass conformsToProtocol

``` java
+ (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = object_getClass((id)self); tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}
- (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = [self class]; tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}
+ (BOOL)isMemberOfClass:(Class)cls {
    return object_getClass((id)self) == cls;
}
- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}
```

测试：

```
BOOL kindObj = [[NSObject class] isKindOfClass:[NSObject class]];  // YES
BOOL kindTemp = [[Student class] isKindOfClass:[Student class]];    // NO
[[Student class] isKindOfClass:[NSObject class]];       // YES
```

参考：https://www.jianshu.com/p/0b14c53fde22

## performSelector 带多个参数

performSelector 是对 objc_msgSend 的简单封装

runtime 源码：
``` java
- (id)performSelector:(SEL)sel withObject:(id)obj {
    if (!sel) [self doesNotRecognizeSelector:sel];
    return ((id(*)(id, SEL, id))objc_msgSend)(self, sel, obj);
}
```

我们实现3个参数

```
- (id)performSelector:(SEL)sel withObject:(id)obj withObject2:(id)obj2 withObject3:(id)obj3 {
    if (!sel) [self doesNotRecognizeSelector:sel];
    return ((id(*)(id, SEL, id, id, id))objc_msgSend)(self, sel, obj, obj2, obj3);
}
```

## 在子线程中执行 performSelector:withObject:afterDelay

```
[self performSelector:@selector(test111) withObject:nil afterDelay:3];
[[NSRunLoop currentRunLoop] runUntilDate:NSDate.distantFuture];
```


## isProxy 作用


## NSObject 的方法


## runtime_initialize方法与load方法的区别

https://blog.csdn.net/shorewb/article/details/52081178