---
layout: post
title:  "RxSwift-Operation操作符"
categories: blog RxSwift
---

* 目录
{:toc}

### 创建Observable

#### just
Observable 发出唯一 event，然后complete

    let o = Observable.just(0)
    等价于 
    let id = Observable<Int>.create { observer in
        observer.onNext(0)
        observer.onCompleted()
        return Disposables.create()
    }


#### timer
Observable 每隔一段时间发送一个event(如果设置间隔时间为0，则只会发送一个event)

Element类型必须是 FixedWidthInteger 类型

    /// 初始时间 = 10  只会发一个event 之后 complete
    _ = Observable<Int>.timer(10, scheduler: MainScheduler.instance)
    
    /// 初始时间 = 8  间隔时间 = 10
    /// 发送的数据：0  1  2  3  ...
    _ = Observable<Int>.timer(8, period: 10, scheduler: MainScheduler.instance)
    

#### from 

#### repeatElement

#### create

#### deferred

#### interval

#### timer

#### empty

#### never

### 通过组合创建Observable

#### merge


#### concat



#### zip



#### combineLatest


### 转换Observable

#### map


#### flatMap


#### flatMapLatest


#### concatMap


#### scan


### 从 Observable 中发出指定元素

#### filter


#### take

#### takeWhile


#### akeWhileWithIndex

#### takeUntil

#### takeLast


#### elementAt


#### skip


#### skipWhile


#### skipWhileWithIndex



#### skipUntil


#### sample



#### debounce

#### distinctUntilChanged


#### distinctUntilChanged

#### delay

#### delaySubscription

#### amb

### 错误处理

#### timeout


#### catchErrorJustReturn


#### catchError



#### retry

### 其他


#### publish


#### replay


#### refCount


#### connect


#### do


#### error


#### subscribeOn


#### observeOn


#### single


#### groupBy


#### window


#### buffer


#### startWith



#### ignoreElements



#### materialize


#### dematerialize


#### as