---
layout: post
title:  "PromiseKit源码分析"
categories: blog ios
---

* 目录
{:toc}

### 库的作用

让异步逻辑清晰一些，避免回调地狱。

[官方网站](https://github.com/mxcl/PromiseKit)

### 如何使用
#### 定义一个Promise

``` Swift

firstly {
    login()
}.then { user in
    fetchAvatar(user.userId)
}.done { image in
    self.imageView.image = image
}.catch {   // Error 处理
    print("error")
}

func login() -> Promise<User> {
    return Promise<User> { resolver in
        let user = User(userId: "11")
        resolver.fulfill(user)
//        resolver.reject(error)
    }
}

func fetchAvatar(_ userId: String) -> Promise<UIImage> {
    return Promise<UIImage>(resolver: { resolver in
        let img = UIImage()
        resolver.fulfill(img)
    })
}

struct User: Codable {
    var userId: String
}
```

#### firstly
``` Swift
firstly {   // 执行 promise
    Promise.value("1")
}.done { value in
    print(value)
}.catch { error in
    print(error)
}
```

#### after
``` Swift
/// 延迟3s after返回的是 Guarantee 主线程执行
after(seconds: 3).done {
    print("do sth")
}
```

#### when

“且”    等待所有promise fulfilled，各个 promise 的 result 合并成一个 tuple 作为结果。

``` Swift
let promise1 = Promise.value("1")
let promise2 = Promise.value("2")

when(fulfilled: promise1, promise2)
.done { results in
    print(results)
}.catch { error in
    print(error)
}
```

#### race
“或”    多个Promise，其中有一个resolve，则调用resolve
``` Swift
let promise1 = Promise.value("1")
let promise2 = Promise.value("2")

race(promise1, promise2).done { value in
    print(value)
}
```

#### ensure

无论success 或 failure， 都会执行 ensure 的代码。返回 Promise

``` Swift 
firstly {
    UIApplication.shared.isNetworkActivityIndicatorVisible = true
    return login()
}.then {
    fetch(avatar: $0.user)
}.done {
    self.imageView = $0
}.ensure {
    // 一定会执行
    UIApplication.shared.isNetworkActivityIndicatorVisible = false
}.catch {
    //…
}
```

### finally 
最终执行的代码。相当于 ensure在最后执行。返回 Void

``` Swift 
firstly {
    foo()
}.done {
    //…
}.catch {
    //…
}.finally {
    self.spinner(visible: false)
}
```

#### map
与 Swit 中的map 含义一致
``` Swift
firstly {
    login()
}.map { user -> User in
    var tmp = user
    tmp.userId = "new"
    return tmp
}.then { user in
    fetchAvatar(user.userId)
}.done { image in
//        self.imageView.image = image
    print("")
}
```

#### compactMap

相比 map ，compactMap 可以返回nil。（compact 的含义是 装一个盒子 把值装到了 Optional 里）

加入 返回 nil，则 chain fails with PMKError.compactMap

``` Swift
firstly {
    login()
}.compactMap({ user -> User? in
    return nil
}).then { user in
    fetchAvatar(user.userId)
}.done { image in
//        self.imageView.image = image
    print("")
}
```

#### get
与 done 功能一致。会继续返回 Promise 

``` Swift 
firstly {
    foo()
}.get { foo in
    //…
}.done { foo in
    // same foo!
}
```

#### tap

使用与 get 类似，会继续返回 Promise。但 get 的参数 是 element，tap 返回的是 Result<T>

``` Swift
firstly {
    foo()
}.tap {
    print($0)
}.done {
    //…
}.catch {
    //…
}
```


### then 在 6.0之后的变化

在 6.0版本后，根据不同场景，对then 之前负责的工作做了拆分：
- 继续使用 then ，表示之后还有其他 Promise
- 使用 done 表示所有工作都已做完了
- 使用 map 表示需要对返回值转换

### 源码分析

#### Result

    public enum Result<T> {
        case fulfilled(T)
        case rejected(Error)
    }

#### Resolver
每个 Promise 中都有一个 Resolver 对象。

通过 resolver 调用 fulfill 和 reject 来确定 Promise 最后的结果。

    public class Resolver<T> {
        let box: Box<Result<T>>

        // 两个 func 实际是操作box中封装的 Result 对象
        func fulfill(_ value: T) 
        func reject(_ error: Error)
    }

#### Promise

Promise的生命周期状态

    pending -> fulfilled
    pending -> rejected


### Thenable

    /// Thenable represents an asynchronous operation that can be chained.
    public protocol Thenable: class {
        /// The type of the wrapped value
        associatedtype T

        /// `pipe` is immediately executed when this `Thenable` is resolved
        func pipe(to: @escaping(Result<T>) -> Void)

        /// The resolved result or nil if pending.
        var result: Result<T>? { get }
    }


#### Guarantee

封装了异步操作，没有error的可能

### 注意问题

#### 修改default queue

    PromiseKit.conf.Q.map = .global()
    PromiseKit.conf.Q.return = .main  //NOTE this is the default

### 参考

- https://xiaozhuanlan.com/topic/7592103684
- https://promisekit.org/reference/