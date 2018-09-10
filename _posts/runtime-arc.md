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

##### weak的实现
    id objc_initWeak(id *object, id value) {
        if (!newObj) {
            *location = nil;
            return nil;
        }

        return storeWeak<false/*old*/, true/*new*/, true/*crash*/>
            (location, (objc_object*)newObj);
    }

    id objc_loadWeak(id _Nullable *location) {
        if (!*location) return nil;
        return objc_autorelease(objc_loadWeakRetained(location));
    }

    id objc_loadWeakRetained(id *object) {

        id result;

        SideTable *table;
        
    retry:
        result = *location;
        if (!result) return nil;
        
        table = &SideTables()[result];
        
        table->lock();
        if (*location != result) {
            table->unlock();
            goto retry;
        }

        result = weak_read_no_lock(&table->weak_table, location);

        table->unlock();
        return result;
    }

    id objc_storeWeak(id _Nullable *location, id obj) {
        return storeWeak(location, (objc_object *)obj)
    }

    void objc_copyWeak(id *dest, id *src) {
        id obj = objc_loadWeakRetained(src);
        objc_initWeak(dst, obj);
        objc_release(obj);
    }

    void objc_destroyWeak(id *object) {
       objc_storeWeak(object, nil);
    }









#### 实现原理



