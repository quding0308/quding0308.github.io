---
layout: post
title:  "iOS中文件管理和沙盒机制"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

####  锁分段技术 
    锁分段 就是进一步对一组独立的对象进行分解。
    关于提高并发性 可以使用 锁分离(lock striping) 和 锁分拆(lock splitging) 两种方式。   
    如果一个锁守护多个独立的不同变量，可以通过分拆锁，使不同的锁守护不同变量，从而降低锁的请求频率。

    例如Java中的ConcurrentHashMap的实现中使用了一个包含16个锁的数组，每个锁保护所有散列值的1/16，第N个散列桶有第(N mod 16)个锁来保护，所以这个并发集合可以支持最多16个并发的写入器。
    
例如在ObjC的runtime中使用的StripedMap中包含了64个锁的数组。

    /**
    下面是runtime库中的代码
    */
    template<typename T>
    class StripedMap {
        enum { StripeCount = 64 };
        static unsigned int indexForPointer(const void *p) {
            // c++ 从指针类型 转换成一个足够大的整数类型
            uintptr_t addr = reinterpret_cast<uintptr_t>(p);
            // 右移后 取 异或运算，再取余 （最终得到一个小于StripeCount的随机数）
            return ((addr >> 4) ^ (addr >> 9)) % StripeCount;
        }
        
        T array[StripCount];
        
    public:
        // 重写了[]运算 map[指针地址] 即可获取到一个对象
        T& operator[] (const void *p) {
            return array[indexForPointer(p)].value;
        }
    };


在runtime库中的用法

    //里面放了64个spinklock对象，用于获取lock
    static StripedMap<spinlock_t> PropertyLocks;  
    // 使用
    spinlock_t& slotlock = PropertyLocks[slot];
    slotlock.lock();
        oldValue = *slot;
        *slot = newValue;
    slotlock.unlock();




#### spinlock_t的使用
    class spinlock_t {
        os_lock_handoff_s mLock;
    public:
        spinlock_t() : mLock(OS_LOCK_HANDOFF_INIT) { }
        
        void lock() { os_lock_lock(&mLock); }
        void unlock() { os_lock_unlock(&mLock); }
        bool trylock() { return os_lock_trylock(&mLock); }


        // Address-ordered lock discipline for a pair of locks.
        // 两个指针指向的内容互换时，需要先锁定两个地址指向的内容
        static void lockTwo(spinlock_t *lock1, spinlock_t *lock2) {
            if (lock1 > lock2) { // lock1 lock2 锁有先后顺序？
                lock1->lock();
                lock2->lock();
            } else {
                lock2->lock();
                if (lock2 != lock1) lock1->lock(); 
            }
        }

        static void unlockTwo(spinlock_t *lock1, spinlock_t *lock2) {
            lock1->unlock();
            if (lock2 != lock1) lock2->unlock();
        }
    };

#### SideTables/RefcountMap/weak_table_t的使用
    struct SideTable {
        spinlock_t slock;   // 锁保证 atomic

        /**
        RefcountMap disguises its pointers because we 
        don't want the table to act as a root for `leaks`.
        对象具体的引用计数存储在这里
        */
        RefcountMap refcnts;

        /**
        * The global weak references table. 
        * Stores object ids as keys, and weak_entry_t structs as their values.
        */
        weak_table_t weak_table;

        void lock() { slock.lock(); }
        void unlock() { slock.unlock(); }
        bool trylock() { return slock.trylock(); }

        // Address-ordered lock discipline for a pair of side tables.

        template<bool HaveOld, bool HaveNew>
        static void lockTwo(SideTable *lock1, SideTable *lock2);
        template<bool HaveOld, bool HaveNew>
        static void unlockTwo(SideTable *lock1, SideTable *lock2);
    };

   
    struct weak_table_t {
        weak_entry_t *weak_entries; // 数组 存储了指向某个对象的所有弱引用的变量指针
        size_t    num_entries;
        uintptr_t mask;
        uintptr_t max_hash_displacement;
    };

    /**
    
    */
    struct weak_entry_t {
        DisguisedPtr<objc_object> referent; // 某个对象的指针
        weak_referrer_t *referrers; // 数组 对这个对象的弱引用的变量地址存在这里
    };

    /// The address of a __weak object reference
    typedef objc_object ** weak_referrer_t;  

###




#### 参考：
- os_lock_handoff_s参考：https://opensource.apple.com/source/libmalloc/libmalloc-116.30.3/src/locking.h.auto.html
- 
