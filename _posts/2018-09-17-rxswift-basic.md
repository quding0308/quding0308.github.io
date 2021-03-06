---
layout: post
title:  "RxSwift与RxCocoa基础概念"
categories: blog RxSwift
---

* 目录
{:toc}

### Event
```
public enum Event<Element> {
    case next(Element)  // 序列产生一个新的元素
    case error(Swift.Error) // 创建序列时报错，seq会终止
    case completed  // 序列结束
}
```
### Observable-一个可被观察的序列

####  Single

- 只能发出一个元素，或一个error事件
- 不会共享状态变化

```
public enum SingleEvent<Element> {
    case success(Element)
    case error(Swift.Error)
}
```

#### Maybe

- 发出一个元素或者一个 completed 事件或者一个 error 事件
- 不会共享状态变化

#### Completable

- 不会发出元素，发出一个 completed 事件或者一个 error 事件
- 不会共享状态变化

#### Driver

为了简化UI层代码，具有以下特征：
- 不会产生 error 事件
- 一定在 MainScheduler 监听（主线程监听）
- 共享状态变化

```
.asDriver(onErrorJustReturn: [])
等价于
let safeSequence = xs
  .observeOn(MainScheduler.instance)       // 主线程监听
  .catchErrorJustReturn(onErrorJustReturn) // 无法产生错误
  .share(replay: 1, scope: .whileConnected)// 共享状态变化
return Driver(raw: safeSequence)           // 封装
```

参考：https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/rxswift_core/observable/driver.html

#### BehaviorRelay

BehaviorRelay is a wrapper for `BehaviorSubject`. Unlike `BehaviorSubject` it can't terminate with error or completed.

BehaviorRelay 是 Observable 

#### ControlEvent

专门用于描述UI控件所产生的事件，具有以下特征：

- 不会产生 error 事件
- 一定在 MainScheduler 订阅（主线程订阅）
- 一定在 MainScheduler 监听（主线程监听）
- 共享状态变化

#### Signal

Observable 有以下特性：
- 不会产生 error 
- 在主线程处理 event
- `share(scope: .whileConnected)` sharing strategy


#### Hot and Cold Observable
[官方介绍](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/HotAndColdObservables.md)

> When does an Observable begin emitting its sequence of items? It depends on the Observable. A “hot” Observable may begin emitting items as soon as it is created, and so any observer who later subscribes to that Observable may start observing the sequence somewhere in the middle. A “cold” Observable, on the other hand, waits until an observer subscribes to it before it begins to emit items, and so such an observer is guaranteed to see the whole sequence from the beginning.


Variable、ControlEvent、Driver等都是 Hot Observable 。不管有没有订阅者，都会发出 element 。Observable 中一般会有多个 element 。会共享状态变化。

Async Operation、Http Connection等是 Cold Observable 。有了订阅者后才会发出 element 。Observable 一般只有一个 element。不会共享状态变化。

### Observer

用来监听Event，然后对Event做出响应

ObserverType定义了最基本的操作
```
public protocol ObserverType {
    associatedtype E

    func on(_ event: Event<E>)
}

extension ObserverType {
    public func onNext(_ element: E) {
        on(.next(element))
    }
    
    public func onCompleted() {
        on(.completed)
    }
    
    public func onError(_ error: Swift.Error) {
        on(.error(error))
    }
}
```

#### 使用闭包创建观察者

```
// 直接使用闭包 onNext、onError、onCompleted 来创建 
tap.subscribe(onNext: { [weak self in
    print("")
}, onError: { error in
    print("error")
}, onCompleted: {
    print("completed")
})
```

#### 使用 AnyObserver 创建

```
let observer: AnyObserver<String> = AnyObserver(eventHandler: { event in
    switch event {
    case .next(let result):
        print(result)
    case .completed:
        print("completed")
    case .error(let error):
        print(error)
    default:
        break
    }
})
```

#### 使用 Binder 创建

实现了 ObserverType 协议，具有以下特点：
- 不会处理 error 和 complete 事件
- 确保绑定都是在给定 Scheduler 上执行（默认 MainScheduler）

```
let observer: Binder<Bool> = Binder(view, binding: { (v, isHidden) in
    v.isHidden = isHidden
})

// RxSwift中的使用
extension Reactive where Base: UIView {
    public var isHidden: Binder<Bool> {
        return Binder(self.base) { view, hidden in
            view.isHidden = hidden
        }
    }
}

extension Reactive where Base: UIControl {
    public var isEnabled: Binder<Bool> {
        return Binder(self.base) { control, value in
            control.isEnabled = value
        }
    }
}
```

示例：
```
let o = Observable<Int>.interval(2, scheduler: MainScheduler.instance)
    .map({ "\($0)" })

o.bind(to: label.rx.text)
    .disposed(by: disposeBag)

o.bind(to: textField.rx.text)
    .disposed(by: disposeBag)
```

### Observable & Observer 既是可被监听的序列也是观察者

#### AsyncSubject

将在源 Observable 产生完成事件后，发出最后一个元素（仅仅只有最后一个元素），如果源 Observable 没有发出任何元素，只有一个完成事件。那 AsyncSubject 也只有一个完成事件。

```
输出： 
subject 3

let subject = AsyncSubject<Int>()

subject.subscribe(onNext: { element in
    print("subject \(element)")
}).disposed(by: disposeBag)

Observable.from([1, 2, 3])
    .subscribe(subject)
    .disposed(by: disposeBag)
```

#### PublishSubject

PublishSubject 将对观察者发送订阅后产生的元素，而在订阅前发出的元素将不会发送给观察者。

```
输出：
间隔4s
subject 0 
间隔4s
subject 1
间隔4s
subject 2
...

let subject = PublishSubject<Int>()

subject.subscribe(onNext: { element in
    print("subject \(element)")
}).disposed(by: disposeBag)

Observable<Int>.interval(4, scheduler: MainScheduler.instance)
    .subscribe(subject)
    .disposed(by: disposeBag)
```

#### ReplaySubject

ReplaySubject 将对观察者发送全部的元素，无论观察者是何时进行订阅的。可以指定队列中event的最大数量

    let sub: ReplaySubject<String> = ReplaySubject<String>.createUnbounded()

    let sub: ReplaySubject<String> = ReplaySubject<String>.create(bufferSize: 8)


#### BehaviorSubject

当观察者对 BehaviorSubject 进行订阅时，它会将源 Observable 中最新的元素发送出来（如果不存在最新的元素，就发出默认元素）。然后将随后产生的元素发送出来。

```
输出：
subject 123 
间隔4s
subject 0 
间隔4s
subject 1
间隔4s
subject 2
...

let subject = BehaviorSubject(value: 123)

subject.subscribe(onNext: { element in
    print("subject \(element)")
}).disposed(by: disposeBag)

Observable<Int>.interval(4, scheduler: MainScheduler.instance)
    .subscribe(subject)
    .disposed(by: disposeBag)

```

#### ControlProperty

专门用于描述 UI 控件属性的，它具有以下特征：

- 不会产生 error 事件
- 一定在 MainScheduler 订阅（主线程订阅）
- 一定在 MainScheduler 监听（主线程监听）
- 共享状态变化


### Disposable-管理订阅的生命周期

每一次绑定 都会生成一个 Disposable 对象。我们需要把 Disposable 对象放到一个 DisposeBag 中。

#### 推荐使用 DisposeBag
当 DisposeBag 对象被销毁调用  deinit 时，会遍历调用每个 Disposable 对象的 dispose 方法。RxSwift中使用的 Disposable 对象为 AnonymousDisposable

    // 用于清除资源
    public protocol Disposable {
        /// Dispose resource.
        func dispose()
    }

    // 每次调用这个方法 把 Disposable 对象加入到 bag中
    extension Disposable {
        public func disposed(by bag: DisposeBag) {
            bag.insert(self)
        }
    }

#### 手动管理 Disposable 对象


    let disposable = viewModel.image?
        .bind(to: imgView.rx.image)

    // 自己来手动 dispose 后，订阅会被取消，内部资源会被释放
    disposable.dispose()
    
#### takeUntil 管理订阅周期

    /// takeUntil 操作符将镜像源 Observable，它同时观测第二个 Observable。一旦第二个 Observable 发出一个元素或者产生一个终止事件，那个镜像的 Observable 将立即终止。
    viewModel.image?
        .takeUntil(viewModel.sections)
        .bind(to: imgView.rx.image)

### Scheduler

#### 使用 subscribeOn

决定数据序列的构建函数在哪个 Scheduler 上运行

#### 使用 observeOn

决定在哪个 Scheduler 监听这个数据序列

    tableView.rx.itemSelected
        .subscribeOn(ConcurrentDispatchQueueScheduler.init(qos: .userInitiated))
        .subscribe(onNext: { indexPath in
            print("select 后执行")
        })
        .disposed(by: disposeBag)
    
    tableView.rx.willDisplayCell
        .subscribeOn(ConcurrentDispatchQueueScheduler.init(qos: .userInitiated))
        .observeOn(MainScheduler.instance)
        .subscribe(onNext: { (cell, indexPath) in
            //
        }).disposed(by: disposeBag)

#### MainScheduler
#### ConcurrentMainScheduler
#### CurrentThreadScheduler

#### SerialDispatchQueueScheduler
#### ConcurrentDispatchQueueScheduler

#### OperationQueueScheduler
#### HistoricalScheduler



