---
layout: post
title:  "多线程&并发"
categories: blog
---

* 目录
{:toc}


### 原子性 Atomic

原子性是指一个操作是不可中断的，要么全部执行成功要么全部执行失败。即使多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程所干扰。

    int a = 10;  // 是原子操作。只有赋值一个操作
    int b = a;  // 不是原子操作。包含两个操作：读取a；给b赋值
    a++; // 不是原子操作，包含三个操作：读取a；进行 a + 1 操作；给a赋值 

非原子性操作，在多线程操作中是不安全的。

线程不安全就是多线程环境下，执行可能不会得到正确的结果。

### 并发


### 同步、异步、阻塞、非阻塞

同步和异步关注的是**消息通信机制**。
**同步**：发出一个调用时，没有得到结果时，该调用就不会返回。

**异步**：发出一个调用时，调用直接返回，结果不会立刻返回，而是通过通知、回调来返回。

例如Node.js的异步编程模型

阻塞和非阻塞关注的是**程序在等待调用结果（消息，返回值）时的状态**.

**阻塞**：是指调用结果返回之前，当前线程会被挂起。

**非阻塞**：该调用不会阻塞当前线程。结果会通过block返回。

    // 会阻塞当前线程，直到block执行完
    dispatch_sync(queue, ^{
        // do sth
    });

    // 不会阻塞当前线程，block会放到queue中 
    dispatch_async(queue, ^{
        // do sth
    });

### iOS中Dispatch库

#### 介绍

GCD是iOS上一套用于编写并发代码的API，可以充分利用多核cpu硬件。

#### semaphore

#### dispatch barrier

dispatch barrier 可以在一个concurrent dispatch queue中，创建一个sync的点。当执行一个barrier时，这个concurrent queue会保证queue里的block都执行完，然后在执行barrier block。barrier block执行完后，concurrent dispatch queue会继续并发执行queue中的block

示例
    
    /*
     代码执行顺序：
     [queue中的block全部执行完毕：block1、block2] -> barrier block -> [开始并发queue中的block: block3、block4]
     */
    dispatch_queue_t concurrent_queue = dispatch_queue_create("concurrent_queue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(concurrent_queue, ^{
        printf("block1");
    });
    dispatch_async(concurrent_queue, ^{
        printf("block2");
    });
    dispatch_barrier_async(concurrent_queue, ^{
        printf("barrier block");
    });
    dispatch_async(concurrent_queue, ^{
        printf("block3");
    });
    dispatch_async(concurrent_queue, ^{
        printf("block4");
    });
    
实际案例：单一资源，多读单写

    dispatch_queue_t concurrent_queue = dispatch_queue_create("concurrent_queue", DISPATCH_QUEUE_CONCURRENT);
    - (void)setObject:(id)object forKey:(NSString *)key {
        key = [key copy];
        dispatch_barrier_async(concurrent_queue, ^{
            if (key && object) {
                [_dic setObject:object forKey:key];
            }
        });
    }

    - (id)getSafeObjectForKey:(NSString *)key {
        __block id result = nil;
        dispatch_sync(concurrent_queue, ^{  // 如果遇到写，同步等待写完再读
            result = [_dic objectForKey:key];
        });
        return result;
    }







