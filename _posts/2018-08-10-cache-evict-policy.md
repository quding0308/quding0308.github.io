---
layout: post
title:  "Cache-缓存淘汰策略"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

缓存都会设置一个大小限制。如果超过最大缓存限制，有新数据进入后就开始淘汰旧数据

### FIFO

先进先出。判断被存储的时间，离目前最远的数据优先被淘汰。

### Random

任意选择数据淘汰

### LFU

Least Frequently Used 最不经常使用。在一段时进内，数据被使用次数最少的，优先被淘汰。

### MRU

Most Recently Used 最近最常使用

### LRU

Least Recently used 最近最少使用，根据时间来判断，最少使用的数据优先被淘汰。

LRU算法的设计原则是：如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小

### LRU的几种实现思路

#### 使用数组实现

使用数组存储，每个元素中记录下键值对和最近访问的时间戳。

这种方式需要不停地维护数据项的时间戳，查询、删除、新增数据的复杂度都是 O(n)

#### 使用链表实现

每次插入新数据，到head

每次访问数据(命中缓存)，该数据移到head。

当链表满后，丢弃链表尾部数据。

查询、删除随机数据的时候，都需要先定位数据，复杂度是 O(n)
插入(插入意味着可能删除数据)、

### 使用双向链表+HashTable实现

查询、删除随机数据 复杂度为O(1)

插入的复杂度为 O(n) 需要删除数据

使用Swift的简单实现(demo，没有实现线程安全)：

    /// LRU算法
    public class LRU<Element> {
        public typealias Node = LRUNode<Element>

        /// count limit 大于0才生效
        public var countLimit: UInt = 0
        
        /// age limit 过期时间
    //    public var ageLimit: TimeInterval
        
        private var dict = [String: Node]()
        private var head: Node?
        private var tail: Node?
        
        private func insertNodeAtHead(_ node: Node) {
            if let _ = head {
                node.next = head
                head?.pre = node
                
                head = node
            } else {
                head = node
                tail = node
            }
        }
        
        private func bringNodeToHead(_ node: Node) {
            guard node !== head else {
                return
            }
            
            if tail === node {
                tail = node.pre
                tail?.next = nil
            } else {
                node.pre?.next = node.next
                node.next?.pre = node.pre
            }
            
            node.next = head
            node.pre = nil
            head?.pre = node
            head = node
        }
        
        private func removeNode(_ node: Node) {
            node.pre?.next = node.next
            node.next?.pre = node.pre
        }
        
        private func removeTailNode() {
            tail = tail?.pre
            tail?.pre = nil
        }
        
        // MARK: public
        public func set(_ key: String, element: Element) {
            let node = LRUNode(key: key, element: element)
            node.time = NSTimeIntervalSince1970
            
            dict[node.key] = node
            insertNodeAtHead(node)
            
            if dict.count > countLimit {
                removeTailNode()
            }
        }
        
        public func element(key: String) -> Element? {
            if let node = dict[key] {
                bringNodeToHead(node)
                return node.element
            }
            
            return nil
        }
        
        public func remove(key: String) {
            if let node = dict[key] {
                removeNode(node)
                
                dict.removeValue(forKey: node.key)
            }
        }
    }

    public class LRUNode<Element> {
        typealias Node = LRUNode<Element>
        
        var key: String
        var element: Element?
        
        var pre: Node?
        var next: Node?
        
        var time: TimeInterval = TimeInterval()
        
        init(key: String, element: Element?) {
            self.key = key
            self.element = element
        }
    }


YYCache中的Objcetive C的实现，使用了双向链表 +  Dictionary实现

[YYLinkedMap对象实现了LRU算法](https://github.com/ibireme/YYCache/blob/master/YYCache/YYMemoryCache.m)


链接：
[https://github.com/quding0308/KDAlgorithmKit/blob/master/KDAlgorithmKit/Classes/DataStructure/LRU.swift](https://github.com/quding0308/KDAlgorithmKit/blob/master/KDAlgorithmKit/Classes/DataStructure/LRU.swift)

