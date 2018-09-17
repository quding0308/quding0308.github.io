---
layout: post
title:  "OjbC中ARC中weak底层实现原理"
date:   2018-09-10 10:05:05 +0800
categories: runtime
---

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

