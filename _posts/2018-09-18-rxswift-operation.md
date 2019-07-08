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

### 组合 Observable

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

```
let disposeBag = DisposeBag()
private func test() {
    let o1 = Observable<Int>.create({ observer in
        after(seconds: Double(2)).done {
            observer.onNext(999)
            observer.onCompleted()
        }
        
        return Disposables.create()
    })
    
    let o2 = Observable<Int>.timer(0, period: 3, scheduler: MainScheduler.instance)

    // 输出： 0、999、1、2、3 ...
    Observable.merge(o1, o2)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
    
    // 输出： 999、0、1、2、3 ...
    Observable.concat(o1, o2)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
    
    // 输出： (999，0)
    Observable.zip(o1, o2)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    // 输出： (999，0)、(999，1)、(999，2) ...
    Observable.combineLatest(o1, o2)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

### 转换Observable

#### map
```
    Observable.of(1, 2, 3)
        .map { $0 * 10 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
```
#### flatMap

flatMap 闭包会返回 Observable，然后所有 Observable 合称为一个 Observable 作为结果

```
Observable.repeatElement(3)
            .flatMap({ value in
                return Observable.just(value)
            })
            .subscribe(onNext: { value in
                print(value)
            })
            .disposed(by: bag)
```

#### flatMapLatest

![](/assets/img/flatMapLatest.png)

参考：https://github.com/baconjs/bacon.js/wiki/Diagrams

#### concatMap

将源 Observable 的每一个元素应用一个转换方法，将他们转换成 Observables。然后让这些 Observables 按顺序的发出元素，当前一个 Observable 元素发送完毕后，后一个 Observable 才可以开始发出元素。等待前一个 Observable 产生完成事件后，才对后一个 Observable 进行订阅。

```
let o1 = Observable.of(1, 2, 3)
let o2 = Observable.of("a", "b", "c")
o1.concatMap({ v1 in
        return o2.map({ (v2) in
            return "\(v1)" + v2
        })
    })
    .subscribe(onNext: { value in
        print(value)
    })
    .disposed(by: bag)
```



```
输出结果为：
=== flatMap 1
=== concatMap 2
=== flatMapFirst 2
=== flatMap 2
=== concatMap 1
=== flatMapLatest 5
=== flatMap 5
=== concatMap 5


let disposeBag = DisposeBag()
private func test() {
    let o = Observable<Int>.create() { observer in
        observer.onNext(2)
        observer.onNext(1)
        observer.onNext(5)
        observer.onCompleted()
        
        return Disposables.create()
    }
    
    o.flatMap({ element in
        return Observable<String>.create({ observer in
            after(seconds: Double(element)).done {
                observer.onNext("\(element)")
            }
            
            return Disposables.create()
        })
    }).subscribe(onNext: { element in
        print("=== flatMap \(element)")
    }).disposed(by: disposeBag)
    
    o.flatMapFirst({ element in
        return Observable<String>.create({ observer in
            after(seconds: Double(element)).done {
                observer.onNext("\(element)")
            }
            
            return Disposables.create()
        })
    }).subscribe(onNext: { element in
        print("=== flatMapFirst \(element)")
    }).disposed(by: disposeBag)
    
    o.flatMapLatest({ element in
        return Observable<String>.create({ observer in
            after(seconds: Double(element)).done {
                observer.onNext("\(element)")
            }
            
            return Disposables.create()
        })
    }).subscribe(onNext: { element in
        print("=== flatMapLatest \(element)")
    }).disposed(by: disposeBag)
    
    o.concatMap({ element in
        return Observable<String>.create({ observer in
            after(seconds: Double(element)).done {
                observer.onNext("\(element)")
                // concat 必须有 oncompleted，否则后续的 Observable 一直在等待
                observer.onCompleted()
            }
            
            return Disposables.create()
        })
    }).subscribe(onNext: { element in
        print("=== concatMap \(element)")
    }).disposed(by: disposeBag)
}
```

#### scan
对每个元素 遍历操作 应用一个函数。

这个函数有一个初始值，之后会把每个元素 和 上次的结果作为参数传进去。

该操作 跟 reduce、accumulator 有点类似，入参都一样。但 reduce 最后只返回一个累积的结果，scan 会返回一个 Observable

```
// 输出：0 1 3 6 10 15 21 28 ..
let observable2 = Observable<Int>.interval(1, scheduler: MainScheduler.instance)

observable2.scan(0) { (first, element) in
    return first + element
    }.subscribe(onNext: { value in
        print("\(value)")
    })
```

### 从 Observable 中发出指定元素

#### filter

过滤数据
```
let o2 = Observable<Int>.timer(0, period: 2, scheduler: MainScheduler.instance)
o2.filter() { $0 % 2 == 0 }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### take

仅仅从 Observable 中发出头 n 个元素

```
输出： 0 1 2

let o2 = Observable<Int>.timer(0, period: 2, scheduler: MainScheduler.instance)
o2.take(3)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### takeLast

只发出尾部 n 个元素。并且忽略掉前面的元素。

```
输出：3 2 1

Observable.of(1, 2, 3, 4, 3, 2, 1)
    .takeLast(3)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### takeWhile

镜像一个 Observable 直到某个元素的判定为 false

```
输出：1 2 3
Observable.of(1, 2, 3, 4, 3, 2, 1)
    .takeWhile { $0 < 4 }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

```

#### takeUntil

将镜像源 Observable，它同时观测第二个 Observable。一旦第二个 Observable 发出一个元素或者产生一个终止事件，那个镜像的 Observable 将立即终止。

```
输出：0 1 2

let o1 = Observable<Int>.create({ observer in
    after(seconds: Double(5)).done {
        observer.onNext(999)
        observer.onCompleted()
    }
    
    return Disposables.create()
})

let o2 = Observable<Int>.timer(0, period: 2, scheduler: MainScheduler.instance)

o2.takeUntil(o1)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### elementAt

只发出 Observable 中的第 n 个元素

#### debounce

过滤掉高频产生的元素

```
输出： 0

Observable.of(1, 2, 3, 4, 3, 2, 1)
    .debounce(0.5, scheduler: MainScheduler.instance)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### throttle

限流  在一定时间内，只输出 第一条和最后一条，中间的忽略。

适用于：输入框搜索限制发送请求。

```
let subject = PublishSubject<Int>()
        subject.throttle(2, scheduler: MainScheduler.instance)
        .subscribe {
            print($0)
        }
        .disposed(by: bag)
        
        subject.onNext(1)
        subject.onNext(2)
        subject.onNext(3)
        subject.onNext(4)
        
        after(seconds: 1).done {
            subject.onNext(5)
        }
        
        after(seconds: 3).done {
            subject.onNext(6)
        }
        
输出 1 5 6
```

#### skip

跳过 Observable 中头 n 个元素

```
// 输出 3 4 3 2 1

Observable.of(1, 2, 3, 4, 3, 2, 1)
    .skip(2)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### skipWhile

跳过 Observable 中头几个元素，直到元素的判定为否

```
// 输出 3 4 3 2 1
Observable.of(1, 2, 3, 4, 3, 2, 1)
    .skipWhile() { $0 < 3 }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### skipUntil

跳过 Observable 中头几个元素，直到另一个 Observable 发出一个元素

```
 // 输出  3 4 5 ..
o2.skipUntil(o1)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

```

#### distinctUntilChanged

阻止 Observable 发出相同的元素。如果前后两个元素相同，则后面的元素不会发出来

```
Observable.of("1", "1", "2", "3", "4", "4")
    .distinctUntilChanged()
    .subscribe {
        print($0)
    }
    .disposed(by: bag)

输出：1 2 3 4                                                
```

#### delay

将 Observable 的每一个元素拖延一段时间后发出

```
// 延迟 2 秒，输出 999
Observable.just(999, scheduler: MainScheduler.instance)
    .delay(2, scheduler: MainScheduler.instance)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

#### amb

在多个源 Observables 中，取第一个发出元素或产生事件的 Observable，然后只发出它的元素

![](/assets/img/amb.png)

### 错误处理

#### retry

如果源 Observable 产生一个错误事件，重新对它进行订阅

![](/assets/img/retry.png)

```
输出：999 999 999 test(错误)
let o1 = Observable<Int>.create({ observer in
    after(seconds: Double(5)).done {
        observer.onNext(999)
        observer.onError(TestError.test)
    }
    
    return Disposables.create()
})

o1.retry(3)
    .subscribe(onNext: { print($0) }, onError: { print($0) })
    .disposed(by: disposeBag)

```

#### timeout

如果 Observable 在一段时间内没有产生元素，timeout 操作符将使它发出一个 error 事件。

```
// 会走 onError 逻辑
o1.timeout(1, scheduler: MainScheduler.instance)
    .subscribe(onNext: { print($0) }, onError: { print($0) })
    .disposed(by: disposeBag)

```

#### catchErrorJustReturn

```
输出：999 10

let o1 = Observable<Int>.create({ observer in
    after(seconds: Double(5)).done {
        observer.onNext(999)
        observer.onError(TestError.test)
    }
    
    return Disposables.create()
})

o1.catchErrorJustReturn(10)
            .subscribe(onNext: { print($0) }, onError: { print($0) })
            .disposed(by: disposeBag)
```

#### catchError

从一个错误事件中恢复，将错误事件替换成一个备选序列


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