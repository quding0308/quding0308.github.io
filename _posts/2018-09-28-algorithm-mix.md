---
layout: post
title:  "mix杂"
# date:   2018-06-23 10:05:05 +0800
categories: blog algorithm
---

* 目录
{:toc}

### 数据抽象

**数据类型**指一组值和一组对这些值的操作的集合。

**抽象数据类型ADT**是一种能够对使用者隐藏数据表示的数据类型。使用的时候，只关注API描述的操作，而不关系数据的表示。

在实现**抽象数据类型**的时候，我们需要关注数据本身，并实现对该数据的各种操作。

- 以适用于各种用途的API形式准确的定义问题
- 用API的实现描述算法和数据结构

### 符号表  

英文 Symbol Table，简称 SB

符号表是一种典型的抽象数据类型 ADT 。它最主要的目的就是将一个键和一个值联系起来。

定义的api如下

    /// Symbol Table
    class ST<Key, Value> {
        func put(key: Key, value: Value)
        
        func get(key: Key) -> Value?
        
        func delete(key: Key)
        
        func contains(key: Key) -> Bool
        
        func isEmpty() -> Bool
        
        func size() -> Int
    }

### 符号表的几种实现

- 基于无序链表实现
- 基于有序数组实现
- 基于散列表实现


### 散列表 HashTable

我们通过一个数组来实现无序的符号表。 

通过散列函数hash() 将key转换为小整数，该整数作为数组的索引。

这样就可以快速访问任意key对应的value。

通过key找到索引分为两步：
1. 用散列函数将key转化为一个索引
2. 处理**碰撞冲突**

### 散列函数

假设有一个能保存M个键值的数组，我们需要一个散列函数，能够把任意key转化为在[0,M-1]范围内的索引。

具体散列函数 跟 key 的类型有关

#### 除留余数法

    // key为正整数
    hashCode = k % M

好的散列函数需要满足三个条件：
- 一致性 
- 高效性 计算速度快，简单
- 均匀性 均匀的散列所有键

### 碰撞冲突处理

#### 基于拉链法的散列表

将大小为M的数组中每个元素指向一个链表，链表中的每个节点存储了散列值为该元素的索引的键值对。






#### 基于线性探测法的散列表

用大小为M的数组保存N个键值对(M > N)。当碰撞发生时，索引加1，检测散列表中下一个位置，有3种可能：
- 命中。该位置的键好被查找的键相同
- 未命中，键为空  停止查找，没有找到元素
- 未命中，键不相等，继续查找


### 如何判断一个单链表是否有环？

#### 方式1：遍历链表 寻找是否有相同的地址

    func hasCircle2() -> Bool {
        var n = head
        while let node = n {
            var next = node.next
            if node === next {
                return true
            }
            
            while let node2 = next?.next {
                if node === node2 {
                    return true
                }   
                next = node2
            }
            n = node.next
        }
        
        return false
    }

    复杂度为n^2

#### 方式2：经典方式

    extension SinglyLinkedList {
        func hasCircle() -> Bool {
            var slowStep = head
            var quickStep = head?.next
            
            while let quick = quickStep, let slow = slowStep  {
                if quick === slow {
                    return true
                } else {
                    quickStep = quickStep?.next?.next
                    slowStep = slowStep?.next
                }
            }
            
            return false
        }
    }

    算法复杂度为O(n)

#### 求环的入口

寻找环入口的方法就是采用两个指针，一个从表头出发，一个从相遇点出发，一次都只移动一步，当二者相等时便是环入口的位置。

    func detectCycle() -> Node? {
        var slowStep = head
        var quickStep = head?.next
        
        var cross: Node? = nil
        while let quick = quickStep, let slow = slowStep  {
            if quick === slow {
                cross = quick
            } else {
                quickStep = quickStep?.next?.next
                slowStep = slowStep?.next
            }
        }
        
        var headWalker = head
        var crossWalker = cross
        while let _ = headWalker?.next, let _ = crossWalker?.next, headWalker !== crossWalker {
            crossWalker = crossWalker?.next
            headWalker = headWalker?.next
        }
        
        if headWalker === crossWalker {
            return headWalker
        }
        
        return nil
    }

数学推到：

[参考](https://blog.csdn.net/sinat_35261315/article/details/79205157)

![图片](http://pc5ouzvhg.bkt.clouddn.com/WechatIMG187.jpeg)

#### 求环的长度 

知道环的入口后，继续往下走  直到下次到入口走的距离就是 环的长度



### 约瑟夫问题

问题描述：

N个人围成一圈，从第一个开始报数，第M个将被杀掉，最后剩下一个，其余人都将被杀掉。

例如N=6，M=5，被杀掉的顺序是：5，4，6，2，3，1。

#### 解法

用循环单链模拟整个过程，时间复杂度为O(m*n)

    class Node {
        var next: Node?
        var index: Int
        
        init(index: Int) {
            self.index = index
            print(index)
        }
    }

    func createCycle(n: Int) -> Node? {
        let head = Node(index: 1)
        
        var node = head
        var count = 1
        while count < n {
            count += 1
            
            let next = Node(index: count)
            
            node.next = next
            node = next
        }
        node.next = head // 单链环
        
        return head
    }

    func run(total: Int, tag: Int) {
        let head = createCycle(n: total)
        
        let start = 1
        var index = start
        var node: Node? = head
        var pre: Node?
        while node !== node?.next {
            if index == tag {
                print("index \(node?.index ?? -1)")
                pre?.next = node?.next
                node = pre?.next
                
                index = start
            } else {
                pre = node
                node = node?.next
                index += 1
            }
        }
        print("last one \(node?.index ?? -1)")
    }


    run(total: 6, tag: 5)
    tag = 1 时 死循环了

[参考](https://zh.wikipedia.org/wiki/%E7%BA%A6%E7%91%9F%E5%A4%AB%E6%96%AF%E9%97%AE%E9%A2%98#C++%E7%89%88%E6%9C%AC)


### 斐波那契数列

又称为 黄金分割数列 或 兔子数列

黄金分割
随着数列项数的增加，前一项与后一项之比越来越逼近黄金分割的数值0.6180339887..

指的是这样一个数列： 
    
    0、1、1、2、3、5、8、13、21、34、...

数学表示：

    f(1) = 1
    f(2) = 1
    f(n) = f(n -1) + f(n - 2)


程序表示：

    /// 递归表示 斐波那契数列
    /// 时间复杂度为 O(2^n)
    ///
    /// - Parameter n: 数列中第n位的值
    /// - Returns:
    func fibonacci_recursion(n: UInt) -> UInt {
        if n == 0 {
            return 0
        } else if n == 1 {
            return 1
        }
        
        return fibonacci_recursion(n: n - 1) + fibonacci_recursion(n: n - 2)
    }

    复杂度计算：f(4)的表示

                    f(4)
                    /   \ 
                 f(3)    f(2) 
               /    \    /   \ 
            f(2)   f(1) f(1) f(0) 
            /   \        
           f(1) f(0) 

    一个k层满二叉树的节点数为 2^k - 1
    f(n)是一个树高为n的二叉树 近似计算复杂度为 O(2^n)


    /// 迭代计算 斐波切纳数列
    /// 时间复杂度为 O(n) 空间复杂度为 O(1)
    /// - Parameter n:
    /// - Returns:
    func fibonacci_iterator(n: UInt) -> UInt {
        if n == 0 {
            return 0
        } else if n == 1 {
            return 1
        } else {
            var a: UInt = 1
            var b: UInt = 1
            var c: UInt = 1
            
            for _ in 2..<n {
                c = a + b
                a = b
                b = c
            }
            
            return c
        }
    }

    /// 还有一种使用矩阵计算的思路，复杂度为O(log(n)


### 求最大公约数

    int gcd(int a,int b) {
        if(a%b)
            return gcd(b,a%b);
        return b;
    }