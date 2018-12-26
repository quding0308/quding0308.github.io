---
layout: post
title:  "+load 与 +initialized 的区别"
categories: blog
---

* 目录
{:toc}

## +load

### 源码 
在 dyld 加载images的流程中，会在 *objc setup* 流程中初始化整个 runtime 的class、category、protocol等，在 read_images后，会调用 load_images。 load_images 就是 调用代码中的 +load 方法

// 判断是否 有 load 的 class 和 category
bool hasLoadMethods(const headerType *mhdr) {
    size_t count;
    // 在 Mach-O 中对应的 __objc_nlclslist 
    if (_getObjc2NonlazyClassList(mhdr, &count)  &&  count > 0) return true;
    // 在 Mach-O 中对应 __objc_nlcatlist
    if (_getObjc2NonlazyCategoryList(mhdr, &count)  &&  count > 0) return true;
    return false;
}

```
const char *
load_images(enum dyld_image_states state, uint32_t infoCount,
            const struct dyld_image_info infoList[]) {
    bool found = false;
    
    // 判断是否有 load 方法
    for (uint32_t i = 0; i < infoCount; i++) {
        // 判断是否有两个 section
        if (hasLoadMethods((const headerType *)infoList[i].imageLoadAddress)) {
            found = true;
            break;
        }
    }
    if (!found) return nil;

    recursive_mutex_locker_t lock(loadMethodLock);

    // Discover load methods
    {
        rwlock_writer_t lock2(runtimeLock);
        
        // 调用 prepare_load_methods，读取 两个section 中的数据
        found = load_images_nolock(state, infoCount, infoList);
    }

    // Call +load methods (without runtimeLock - re-entrant)
    if (found) {
        call_load_methods();
    }

    return nil;
}

void call_load_methods(void)
{
    static bool loading = NO;
    bool more_categories;

    loadMethodLock.assertLocked();

    // Re-entrant calls do nothing; the outermost call will finish the job.
    if (loading) return;
    loading = YES;

    void *pool = objc_autoreleasePoolPush();

    do {
        // 首先会把所有的 class 的 load 调用完
        // 1. Repeatedly call class +loads until there aren't any more
        while (loadable_classes_used > 0) {
            call_class_loads();
        }

        // 开始 调用 category 的 load 
        // 2. Call category +loads ONCE
        more_categories = call_category_loads();

        // 3. Run more +loads if there are classes OR more untried categories
    } while (loadable_classes_used > 0  ||  more_categories);

    objc_autoreleasePoolPop(pool);

    loading = NO;
}
```

### +load总结：
1. 重写了 +load 的 class 和 category 会在 可执行Mach-O文件中单独存储在 __DATA.__objc_nlclslist 和 __DATA___objc_nlcatlist中，runtime 会读取这两个 section，执行 +load。
2. 会优先执行所有 class 的 +load ，然后执行 catrgory 的 +load 
3. 执行 +load 的循环，会放在 autoreleasePool 中执行，不需要担心内存释放不及时


## initialized

### 源码：

![](https://raw.githubusercontent.com/quding0308/quding0308.github.io/master/res/runtime4.png)

当调用 objc_msgSend 时，会调用 lookUpImpOrForward 去查找方法。

```
IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                       bool initialize, bool cache, bool resolver) {
// ...
// Optimistic cache lookup
if (cache) {
    imp = cache_getImp(cls, sel);
    if (imp) return imp;
}
// ...
// 在这里检测 class 是否初始化过。如果没有初始化 就会调用 _class_initialize 方法
if (initialize  &&  !cls->isInitialized()) {
    _class_initialize (_class_getNonMetaClass(cls, inst));
}
// ...
```

runtime 通过 调用 _class_initialize 来 调用 +initialize ，下面是源码：
```
/***********************************************************************
* class_initialize.  Send the '+initialize' message on demand to any
* uninitialized class. Force initialization of superclasses first.
**********************************************************************/
void _class_initialize(Class cls)
{
    Class supercls;
    BOOL reallyInitialize = NO;

    // 先调用 superclass 的 inittializ
    // Make sure super is done initializing BEFORE beginning to initialize cls.
    // See note about deadlock above.
    supercls = cls->superclass;
    if (supercls  &&  !supercls->isInitialized()) {
        _class_initialize(supercls);
    }
    
    // Try to atomically set CLS_INITIALIZING.
    {
        monitor_locker_t lock(classInitLock);
        if (!cls->isInitialized() && !cls->isInitializing()) {
            cls->setInitializing();
            reallyInitialize = YES;
        }
    }
    
    if (reallyInitialize) {
        // Record that we're initializing this class so we can message it.
        _setThisThreadIsInitializingClass(cls);

        // 调用 + initialize
        // 通过 objc_msgSend 来调用
        ((void(*)(Class, SEL))objc_msgSend)(cls, SEL_initialize);

        monitor_locker_t lock(classInitLock);
        if (!supercls  ||  supercls->isInitialized()) {
            _finishInitializing(cls, supercls);
        } else {
            _finishInitializingAfter(cls, supercls);
        }
        return;
    }
    else if (cls->isInitializing()) {
        // 正在调用 initialize，wait
        if (_thisThreadIsInitializingClass(cls)) {
            return;
        } else {
            monitor_locker_t lock(classInitLock);
            while (!cls->isInitialized()) {
                classInitLock.wait();
            }
            return;
        }
    }
}
```

### +initialize总结：
1. 当 class 第一次收到 objc_msgSend 时，会调用 +initialize 方法
2. +initialize 只会调用一次，通过 objc_msgSend 的方式来调用

