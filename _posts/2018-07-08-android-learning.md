---
layout: post
title:  "Android基础"
# date:   2018-07-09 10:000:00 +0800
categories: blog
---

 

### Android Api Level

    Android 4.0,4.0.1,4.0.2 ~ 14
    Android 4.0.3,4.0.4 ~ 15
    Android 4.1,4.1.1 ~ 16
    Android 5.0 ~ 21
    Android 5.1 ~ 22
    Android 6.0 ~ 23
    Android 7.0 ~ 24
    Android 7.1 ~ 25
    Android 8.0 ~ 26
    Android 8.1 ~ 27

### 指令集
指令集架构是对实际处理器的抽象。只要执行模型一样，不同的处理器实现也可以执行同样的机器代码，而又提供不同的开销和性能。

ARM公司 通过出售芯片技术授权而盈利。ARM设计了高性能、廉价、低耗能的RISC处理器模型，同样的ARM架构提供了一致的 处理器指令集。我们的代码编译后生成机器代码，会支持某一种ARM处理器指令集。Android目前只需要支持armeabi-v7a。高通、联发科等是处理器制造商，需要使用ARM授权的专利。ARM出方案的公司，高通用ARM的方案出CPU，高通同时在此基础上加自己定制的内容，比如一二三级缓存、核心数量、主频等不同。

 

### ABI 
应用程序二进制接口，它定义了一套规则，允许编译好的二进制目标代码在所有兼容该ABI的操作系统和硬件平台上执行。
不同的CPU和操作系统，支持的ABI不同。

常见的ABI类型：armeabi  armeabi-v7  x86  x86_64  mips  mips64  arm64-v8a

- x86系列主要是模拟器，分为 x86和 x86_64
- MIPS系列 多在网关、猫、机顶盒等使用
- armeabi系列 属于ARM
    - armabi 是针对普通的ARM v5 CPU
    - armabi-v7a 是针对ARM v7 CPU
    - arm64-v8a 是针对最新的ARM v8a CPU

### Service层
官方介绍：https://developer.android.com/guide/components/services.html?hl=zh-cn
启动一个service可以通过bindService或startService两种方式。

1.startService
每次调用startService，service都会调用onStartCommand方法获取intent。这种Service只有在程序调用stop Context.stopService() or stopSelf() 才会停止。

2.bindService
调用bindService是，第三个参数一般是  Context.BIND_AUTO_CREATE  如果没有Service则自动创建一个。service在onBind返回一个IBinder对象，client端通过ServiceConnection拿到IBinder对象，转化为一个ServiceStub或者是Message对象，以此来实现通信。

 

参考：http://gityuan.com/2017/05/01/binder_exception/
 

### Intent和IntentFilter
Intent用于调用组件(Activity/Service/BroadcastReceiver)，并且可以作为消息的载体。

Intent分为显示和隐式两种

- 显示Intent。通过类型指定调用的组件。
- 隐式Intent。不直接指定组件类型，而通过IntentFilter设置的规则 去寻找组件。如果多个匹配，则系统会弹框让用户选择使用哪个组件。

Intent相关属性

- action
- data
- category
- type
- extra
 