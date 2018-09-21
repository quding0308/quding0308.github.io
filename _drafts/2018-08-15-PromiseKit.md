---
layout: post
title:  "PromiseKit源码分析"
categories: blog ios
---

* 目录
{:toc}

### 库的作用

[官方网站](https://github.com/mxcl/PromiseKit)

### 如何使用
#### 定义一个Promise

    func executePromise() -> Promise<String> {
        Promise<String> { seal in
            /// async do sth
            
            /// result
            seal.fulfill("fulfill string")
            //seal.reject(error)
        }
    }

#### firstly

    firstly {   // 执行 promise
        Promise.value("1")
    }.done { value in
        print(value)
    }.catch { error in
        print(error)
    }

#### after

    /// 延迟3s after返回的是 Guarantee
    after(seconds: 3).done {
        print("do sth")
    }

#### race
多个Promise，其中有一个resolve，则调用resolve

    let promise1 = Promise.value("1")
    let promise2 = Promise.value("2")
    
    race(promise1, promise2).done { value in
        print(value)
    }

#### when
等待所有promise fulfilled，各个 promise 的 result 合并成一个 tuple 作为结果

    let promise1 = Promise.value("1")
    let promise2 = Promise.value("2")
    
    when(fulfilled: promise1, promise2)
    .done { results in
        print(results)
    }.catch { error in
        print(error)
    }
    
        

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