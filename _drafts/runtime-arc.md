---
layout: post
title:  "OjbC中ARC实现原理"
date:   2018-09-10 10:05:05 +0800
categories: runtime
---

参考：https://clang.llvm.org/docs/AutomaticReferenceCounting.html#arc-optimization-precise

#### 用途

#### Runtime Support
    id objc_autorelease(id value);
    void objc_autoreleasePoolPop(void *pool);
    void *objc_autoreleasePoolPush(void);
    id objc_autoreleaseReturnValue(id value);
    void objc_copyWeak(id *dest, id *src);
    void objc_destroyWeak(id *object);
    id objc_initWeak(id *object, id value);
    id objc_loadWeak(id *object);
    id objc_loadWeakRetained(id *object);
    void objc_moveWeak(id *dest, id *src);
    void objc_release(id value);
    id objc_retain(id value);
    id objc_retainAutorelease(id value);
    id objc_retainAutoreleaseReturnValue(id value);
    id objc_retainAutoreleasedReturnValue(id value);
    id objc_retainBlock(id value);
    id objc_storeStrong(id *object, id value);
    id objc_storeWeak(id *object, id value);

##### autorelease pool
    // 把value加到最靠近的autorelease pool中，回调用 [value autorelease]
    id objc_autorelease(id obj) {
        if (!obj) return obj;
        if (obj->isTaggedPointer()) return obj;
        return obj->autorelease();
    }

    // 创建一个pool
    void *objc_autoreleasePoolPush(void) {
        return AutoreleasePoolPage::push();
    }

    void objc_autoreleasePoolPop(void *pool) {
        AutoreleasePoolPage::pop(pool);
    }

    id objc_autoreleaseReturnValue(id value) {
        // if (prepareOptimizedReturn(ReturnAtPlus1)) return obj;

        return objc_autorelease(obj);
    }

    @autoreleasepool {
        //    
    }




#### 实现原理
#### SideTables() // 64个SideTable对象

    static StripedMap<SideTable>& SideTables() {
        // 
        return *reinterpret_cast<StripedMap<SideTable>*>(SideTableBuf);
    }


