---
layout: post
title:  "RxSwift源码解析"
categories: blog RxSwift
---

* 目录
{:toc}

### obj.rx.method 如何工作？

#### Reactive

Reactive 是一个struct，调用 .rx 会返回一个 Reactive 对象

    public struct Reactive<Base> {
        /// Base object to extend.
        public let base: Base

        /// Creates extensions with base object.
        ///
        /// - parameter base: Base object.
        public init(_ base: Base) {
            self.base = base
        }
    }

#### ReactiveCompatible 
ReactiveCompatible 定义了一个property 凡是 实现了这个协议的class 都可以使用 .rx 返回一个当前类型

    public protocol ReactiveCompatible {
        /// Extended type
        associatedtype CompatibleType

        /// Reactive extensions.
        static var rx: Reactive<CompatibleType>.Type { get set }

        /// Reactive extensions.
        var rx: Reactive<CompatibleType> { get set }
    }

    /// CompatibleType 就是 Self 了
    extension ReactiveCompatible {
        /// Reactive extensions.
        ///  Class.rx  返回一个 Reactive 类型
        public static var rx: Reactive<Self>.Type {
            get {
                return Reactive<Self>.self
            }
            set {
                // this enables using Reactive to "mutate" base type
            }
        }

        /// Reactive extensions.
        /// obj.rx  返回一个 Reactive 对象
        public var rx: Reactive<Self> {
            get {
                return Reactive(self)
            }
            set {
                // this enables using Reactive to "mutate" base object
            }
        }
    }

Rx 中 NSObject 的子类都可以使用 .rx （默认实现）

    import class Foundation.NSObject

    /// Extend NSObject with `rx` proxy.
    extension NSObject: ReactiveCompatible { }

如果 Swift 类没有继承 NSObject，是没法使用Rx的。此时，我们可以让自己的类实现 ReactiveCompatible

    class PeterWang: ReactiveCompatible  {
        init() {
            // self.rx
            // PeterWang.rx
        }
    }

在上面的代码的基础上，我们如果想给某个类添加 rx 的操作 只需要添加Reactive的 extension，其中 Base: MyObject 即可。

#### UIView 示例：

例如 UIView（所有的UIView都可以使用 isHidden 和 alpha 了）

    extension Reactive where Base: UIView {
        /// Bindable sink for `hidden` property.
        public var isHidden: Binder<Bool> {
            return Binder(self.base) { view, hidden in
                view.isHidden = hidden
            }
        }

        /// Bindable sink for `alpha` property.
        public var alpha: Binder<CGFloat> {
            return Binder(self.base) { view, alpha in
                view.alpha = alpha
            }
        }
    }

#### 自己扩展Rx的示例：

MyObject.swift

    class MyObject: ReactiveCompatible {
        init() {
            //
        }
        
        func test() {
            let observable = Observable.just("hello")
            
            let disposable = observable.subscribe(self.rx.helloObject)
            disposable.dispose()
        }
    }

MyObject+Rx.swift

    extension Reactive where Base: MyObject {
        var helloObject: AnyObserver<String> {
            return AnyObserver<String>{ event in
                switch event {
                case .next(let str):
                    print(str)
                case .completed:
                    print("completed")
                case .error(let error):
                    print(error)
                }
            }
        }
    }

### DelegateProxy


### DelegateProxyType


### RxScrollViewDelegateProxy的使用


#### RxScrollViewDelegateProxy


#### RxScrollViewDelegateProxy


#### RxCollectionViewDelegateProxy


#### RxTextViewDelegateProxy


