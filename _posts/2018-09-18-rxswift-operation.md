---
layout: post
title:  "RxSwift-Operation操作符"
categories: blog RxSwift
---

* 目录
{:toc}

### 创建Observable

#### create
自定义一个Observable的逻辑

    let observable = Observable<Int>.create { observer in
        observer.onNext(0)
        observer.onNext(1)
        observer.onCompleted()
        
        return Disposables.create()
    }

#### just
Observable 发出唯一 event，然后complete

    let o = Observable.just(0)
    等价于 
    let id = Observable<Int>.create { observer in
        observer.onNext(0)
        observer.onCompleted()
        return Disposables.create()
    }

#### from 

    // 数组 转化为 Observable
    _ = Observable.from(["1", "2", "3"])
            
    // 一个optional 有值 则发送next  没有值 直接调用complete
    let opt: String?
    opt = "22"
    _ = Observable.from(optional: opt)

#### of
使用给的一系列元素 生成一个 Observable

        _ = Observable.of(1, 2, 3, 4)
        
        _ = Observable.of("n1", "n2", "n3")

#### timer
Observable 每隔一段时间发送一个event(如果设置间隔时间为0，则只会发送一个event)

Element类型必须是 FixedWidthInteger 类型

    /// 初始时间 = 10  只会发一个event 之后 complete
    _ = Observable<Int>.timer(10, scheduler: MainScheduler.instance)
    
    /// 初始时间 = 8  间隔时间 = 10
    /// 发送的数据：0  1  2  3  ...
    _ = Observable<Int>.timer(8, period: 10, scheduler: MainScheduler.instance)
    
#### interval

    _ = Observable<Int>.interval(3, scheduler: MainScheduler.instance)
    等价于 
    _ = Observable<Int>.timer(3, period: 3, scheduler: MainScheduler.instance)

#### deferred
deferred 操作符将等待观察者订阅它，才创建一个 Observable。每次订阅的时候，都会创建一个新的Observable对象

    let observable = Observable<Int>.deferred { () -> Observable<Int> in
        let tmp = Observable.just(1, scheduler: MainScheduler.instance)
        
        return tmp
    }

#### empty
创建一个空 Observable：

    let id = Observable<Int>.empty()
    等价于：
    let id = Observable<Int>.create { observer in
        observer.onCompleted()
        return Disposables.create()
    }

#### repeatElement

    // 一直循环发送一个element
    let observable = Observable.repeatElement("11")
    observable
        .subscribe(onNext: { [weak self] value in
            guard let `self` = self else { return }
            
            print("111")
        })

#### never
创建一个不会产生任何事件的 Observable：

    let id = Observable<Int>.never()
    等价于：
    let id = Observable<Int>.create { observer in
        return Disposables.create()
    }

### 通过组合创建Observable

#### merge

将 [Observable] 数组 合并成一个Observabel序列。当某一个 Observable 发出一个元素时，他就将这个元素发出。

某一个Observable发出error，则这个序列就发出error
所有的Observable都complete，这个序列才complete

    let observable1 = Observable<Int>.interval(3, scheduler: MainScheduler.instance)
    let observable2 = Observable<Int>.interval(5, scheduler: MainScheduler.instance)
    
    Observable.merge([observable1, observable2])
        .subscribe(onNext: { value in
            
            print("value \(value)")
        })

#### concat
多个Observabel串行连接起来，只有前一个 Observable complete 后，后一个 Observable 才开始发送元素

    let observable1 = Observable<Int>.timer(1, scheduler: MainScheduler.instance)
    let observable2 = Observable<Int>.timer(10, scheduler: MainScheduler.instance)
    
    Observable.concat([observable2, observable1])
        .subscribe(onNext: { value in
            print("111 \(value)")
        }, onError: { err in
            print("err")
        }, onCompleted: {
            print("1")
        })

#### zip

将多个 Observable 的元素组合成一个 tuple 作为新 Observable 的元素发出来。新Observable的第n个tuple中的元素由各个Observable的第n个元素组合而成。所以 它的元素数量等于源 Observable 中元素数量最少的那个。

    let observable1 = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
    let observable2 = Observable<Int>.timer(10, scheduler: MainScheduler.instance)
    
    Observable.zip([observable2, observable1])
        .subscribe(onNext: { value in
            print("111 \(value)")
        }, onError: { err in
            print("err")
        }, onCompleted: {
            print("1")
        })


#### combineLatest

将多个 Observable 的最新的元素组合成tuple 作为新Observable的元素发出来。

能发出元素的前提是  每个 Observable 都有至少一个元素。

当有一个 Observable 有新元素，就会从每个 Observable 中取最新的元素组合成 tuple 发出来

Observable 之间没有依赖关系

    let observable1 = Observable<Int>.timer(10, scheduler: MainScheduler.instance)
    let observable2 = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
    
    Observable.combineLatest([observable2, observable1])
        .subscribe(onNext: { value in
            print("111 \(value)")
        }, onError: { err in
            print("err")
        }, onCompleted: {
            print("1")
        })

### 转换Observable

#### map

    Observable.of(1, 2, 3)
        .map { $0 * 10 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

#### flatMap


#### flatMapLatest


#### concatMap


#### scan
对每个元素 遍历操作 应用一个函数。

这个函数有一个初始值，之后会把每个元素 和 上次的结果作为参数传进去。

该操作 跟 reduce、accumulator 有点类似，入参都一样。但 reduce 最后只返回一个累积的结果，scan 是对每个元素操作再返回一个元素苏亮相等的Observable

    let observable2 = Observable<Int>.interval(1, scheduler: MainScheduler.instance)

    observable2.scan(100) { (first, element) in
            return first + element
        }.subscribe(onNext: { value in
            print("111 \(value)")
        })

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