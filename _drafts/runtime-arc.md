---
layout: post
title:  "OjbC中ARC实现原理"
# date:   2018-06-23 10:05:05 +0800
categories: runtime
---

参考：https://clang.llvm.org/docs/AutomaticReferenceCounting.html#arc-optimization-precise

#### 用途

#### runtime暴露的api

    id objc_initWeak(id *object, id value) {
        *object = nil;
        return objc_storeWeak(object, value);
    }


    id objc_loadWeak(id _Nullable *location)

    id objc_loadWeakRetained(id *object)

    id objc_storeWeak(id _Nullable *location, id obj)

    void objc_copyWeak(id *dest, id *src)











#### 实现原理



