---
layout: post
title:  "@synchronized底层实现"
# date:   2018-09-11 10:05:05 +0800
categories: ObjC, runtime
---

#### Demo代码：

    void hello()
    {
        NSObject* obj = [NSObject new];
        @synchronized (obj) {
            
        }
    }

#### 汇编代码中找到了这两行：

    bl	_objc_sync_enter
    bl	_objc_sync_exit

    实质是通过递归互斥锁mutex来实现锁，根据obj会得到一个mutex：
    mutex.lock();
    mutex.tryUnlock();

#### runtime代码中找到了对应的实现(简化后的代码）：

    int objc_sync_enter(id obj)
    {
        int result = OBJC_SYNC_SUCCESS;

        if (obj) {
            SyncData* data = id2data(obj, ACQUIRE);
            assert(data);
            data->mutex.lock();
        }

        return result;
    }

    int objc_sync_exit(id obj)
    {
        int result = OBJC_SYNC_SUCCESS;
        
        if (obj) {
            SyncData* data = id2data(obj, RELEASE); 
            if (!data) {
                result = OBJC_SYNC_NOT_OWNING_THREAD_ERROR;
            } else {
                bool okay = data->mutex.tryUnlock();
                if (!okay) {
                    result = OBJC_SYNC_NOT_OWNING_THREAD_ERROR;
                }
            }
        }
        
        return result;
    }


**static SyncData* id2data(id object, enum usage why);**

id2data函数用来获取一个跟obj相对应的SyncData对象。

##### 相关结构体：
    enum usage { ACQUIRE, RELEASE, CHECK };

    typedef struct SyncCache {
        unsigned int allocated;
        unsigned int used;
        SyncCacheItem list[0];  // 维护一个数组
    } SyncCache;

    // 一个obj对应一个SyncCacheItem对象
    typedef struct {
        SyncData *data;
        unsigned int lockCount;  // number of times THIS THREAD locked this block
    } SyncCacheItem;
    
    //每个obj对应一个SyncData对象，封装了mutex
    typedef struct SyncData {
        struct SyncData* nextData;
        DisguisedPtr<objc_object> object;
        int32_t threadCount;  // number of THREADS using this block
        recursive_mutex_t mutex;
    } SyncData;

    struct SyncList {
        SyncData *data;
        spinlock_t lock;

        SyncList() : data(nil) { }
    };

##### 全局哈希表
    /**
    StripedMap<SyncList> 会有64个SyncList对象，会根据obj的内存地址计算出数据存储到某个SyncList中。
    全局哈希表 会多线程使用，处理数据的时候需要加锁。每个SyncList对象内部有自己的锁。
    最多可以有64个读写同时发生。
    */
    // Use multiple parallel lists to decrease contention among unrelated objects.
    #define LOCK_FOR_OBJ(obj) sDataLists[obj].lock
    #define LIST_FOR_OBJ(obj) sDataLists[obj].data
    static StripedMap<SyncList> sDataLists;


##### id2data函数用来获取一个跟obj相对应的SyncData对象。
当前线程会缓存自己使用过的SyncData，如果命中缓存则直接使用，否则从全局哈希表中读取。
当前线程缓存SyncData又分两中情况
    - fastCache 快速缓存 线程缓存中只有一个SyncDataItem对象，直接读取
    - 使用SyncCache结构体维护一个SyncCache数组，SyncCache与线程一对一，用于缓存线程中处理的SyncCacheItem对象

使用fastCache获取缓存数据


    static SyncData* id2data(id object, enum usage why) {
        SyncData* result = NULL;
        
        /**
        快速缓存（fastCache），适用于一个线程一次只对一个对象加锁的情况，用宏SUPPORT_DIRECT_THREAD_KEYS来标识
        这种情况意味着同一时间内，线程缓存中只有一个SyncCacheItem对象
        
        SYNC_DATA_DIRECT_KEY 用来记录缓存的SyncData对象
        SYNC_COUNT_DIRECT_KEY 用来缓存lockCount
        */
        
        // Check per-thread single-entry fast cache for matching object
        //用于标识当前线程的是否已使用fastCache
        bool fastCacheOccupied = NO;
        //直接调用tls_get_direct函数获取SyncData对象
        SyncData *data = (SyncData *)tls_get_direct(SYNC_DATA_DIRECT_KEY);
        if (data) {
            //标识fastCache已被使用
            fastCacheOccupied = YES;
            //比较fastCache中的SyncData对象中的object与当前同步对象object是否为同一个对象
            if (data->object == object) {
                // Found a match in fast cache.
                uintptr_t lockCount;
                
                result = data;
                //获取当前线程对应当前SyncData对象已经加锁的次数
                lockCount = (uintptr_t)tls_get_direct(SYNC_COUNT_DIRECT_KEY);
                //判断当前操作的加锁还是解锁
                switch(why) {
                        //加锁
                    case ACQUIRE: {
                        //加锁一次
                        lockCount++;
                        //更新已加锁次数
                        tls_set_direct(SYNC_COUNT_DIRECT_KEY, (void*)lockCount);
                        break;
                    }
                        //解锁
                    case RELEASE:
                        //解锁一次
                        lockCount--;
                        //更新已加锁次数
                        tls_set_direct(SYNC_COUNT_DIRECT_KEY, (void*)lockCount);
                        //已加锁次数为0，表示当前线程对当前同步对象object达到锁平衡，因此不需要再持有当前同步对象。
                        if (lockCount == 0) {
                            // remove from fast cache
                            //将对应的SyncData对象从线程缓存中移除
                            tls_set_direct(SYNC_DATA_DIRECT_KEY, NULL);
                            // atomic because may collide with concurrent ACQUIRE
                            OSAtomicDecrement32Barrier(&result->threadCount);
                        }
                        break;
                }
                
                return result;
            }
        }
    }

使用SyncCache结构体维护一个SyncCache数组

    static SyncData* id2data(id object, enum usage why) {
        SyncData* result = NULL;
        
        /**
        使用SyncCache结构体来维护一个SyncCacheItem数组，这样一个线程就可以处理对多个同步对象。值得注意的是SyncCache与线程也是一对一的关系。
        */
        
        SyncCache *cache = fetch_cache(NO);
        if (cache) {
            unsigned int i;
            //遍历SyncCache对象中的SyncCacheItem数组，匹配当前同步对象object
            for (i = 0; i < cache->used; i++) {
                SyncCacheItem *item = &cache->list[i];
                if (item->data->object != object) continue;
                
                // Found a match.
                //当前同步对象object已存在的SyncCache中
                //获取对应的SyncData对象
                result = item->data;
                //后续操作同fastCache一样，参考fastCache的注释
                switch(why) {
                    case ACQUIRE:
                        item->lockCount++;
                        break;
                    case RELEASE:
                        item->lockCount--;
                        if (item->lockCount == 0) {
                            // remove from per-thread cache
                            cache->list[i] = cache->list[--cache->used];
                            // atomic because may collide with concurrent ACQUIRE
                            OSAtomicDecrement32Barrier(&result->threadCount);
                        }
                        break;
                    case CHECK:
                        // do nothing
                        break;
                }
                
                return result;
            }
        }
    }


直接从全局变量读取：

    static SyncData* id2data(id object, enum usage why)
    {
        //从全局哈希表sDataLists中获取object对应的SyncList对象
        //lockp指针指向SyncList对象中自旋锁
        //listp指向一条SyncData链表
        spinlock_t *lockp = &LOCK_FOR_OBJ(object);
        SyncData **listp = &LIST_FOR_OBJ(object);
        SyncData* result = NULL;
        
        /**
        如果当前线程中的缓存中没有找到当前同步对象对应的SyncData对象，则在全局哈希表中查找
        因为全局哈希表是多个线程共享的数据结构，因此需要进行加锁处理
        从全局哈希表中找是否已经缓存了obj对应的SyncData对象
        */
        lockp->lock();
        {
            SyncData* p;
            SyncData* firstUnused = NULL;
            //遍历当前同步对象obejct在全局哈希表中的SyncData链表。这里之所以使用链表，是因为哈希表的hash算法不能确保hash的唯一性，存在多个对象对应一个hash值的情况。
            for (p = *listp; p != NULL; p = p->nextData) {
                //哈希表中存在对应的SyncData对象
                if ( p->object == object ) {
                    result = p;
                    // atomic because may collide with concurrent RELEASE
                    //此函数为原子操作函数，确保线程安全，用于对32位的threadCount整形变量执行加一操作，表示占用当前同步对象的线程数加1。
                    OSAtomicIncrement32Barrier(&result->threadCount);
                    goto done;
                }
                //用于标记一个空闲的SyncData对象
                if ( (firstUnused == NULL) && (p->threadCount == 0) )
                    firstUnused = p;
            }
            
            // no SyncData currently associated with object
            //由于此时同步对象object没有对应的SyncData对象，因此RELEASE与CHECK都属于无效操作
            if ( (why == RELEASE) || (why == CHECK) )
                goto done;
            
            // an unused one was found, use it
            //如果没有找到匹配的SyncData对象且存在空闲的SyncData对象，则直接使用，不需要创建新的SyncData，以提高效率。
            if ( firstUnused != NULL ) {
                result = firstUnused;
                //关联当前同步对象
                result->object = (objc_object *)object;
                //重置占用线程为1
                result->threadCount = 1;
                goto done;
            }
        }
        
        // malloc a new SyncData and add to list.
        // XXX calling malloc with a global lock held is bad practice,
        // might be worth releasing the lock, mallocing, and searching again.
        // But since we never free these guys we won't be stuck in malloc very often.
        
        //到这一步说明需要新建一个SyncData对象
        result = (SyncData*)calloc(sizeof(SyncData), 1);
        result->object = (objc_object *)object;
        result->threadCount = 1;
        //创建递归互斥锁
        new (&result->mutex) recursive_mutex_t();
        //以“入栈”的方式加入当前同步对象object对应的SyncData链表
        result->nextData = *listp;
        *listp = result;
        
    done:
        //对全局哈希表的操作结束，解锁
        lockp->unlock();
        if (result) {
            // Only new ACQUIRE should get here.
            // All RELEASE and CHECK and recursive ACQUIRE are
            // handled by the per-thread caches above.
            //只有ACQUIRE才需要新建SyncData对象
            if (why == RELEASE) {
                // Probably some thread is incorrectly exiting
                // while the object is held by another thread.
                return nil;
            }
            if (why != ACQUIRE) _objc_fatal("id2data is buggy");
            if (result->object != object) _objc_fatal("id2data is buggy");
            
            //fastCache缓存模式
    #if SUPPORT_DIRECT_THREAD_KEYS
            if (!fastCacheOccupied) {
            } else
    #endif
                //SyncCache缓存模式，则直接加入SyncCacheItem数组中
            {
                // Save in thread cache
                if (!cache) cache = fetch_cache(YES);
                cache->list[cache->used].data = result;
                cache->list[cache->used].lockCount = 1;
                cache->used++;
            }
        }
        
        return result;
    }


#### 总结 
- @synchronized方便使用，但也是多线程同步机制中最慢的一种
- 底层使用递归mutex来做同步，可以写@synchronized嵌套的代码，不会导致死锁
- @synchronized(nil) 不起任何作用。意味着 如果obj的生命周期结束了，同步代码也就失效了
- obj的作用：synchronized中传入的object的内存地址，被用作key，通过hash map对应的一个系统维护的递归锁
- 不要使用self作为obj，而是应该声明一个私有的obj来作为key
- @synchronized如果控制好精度，也不会很慢。精度控制 就是 对obj的使用（如果所有的 @synchronized都使用一个obj，则就会很慢了）。不同数据应该使用不同的obj来控制在最细的粒度



#### 参考：
- <http://mrpeak.cn/blog/synchronized/>





