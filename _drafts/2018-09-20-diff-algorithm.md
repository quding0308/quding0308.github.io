---
layout: post
title:  "Diff算法及应用"
categories: blog RxDataSource
---

* 目录
{:toc}

### Diff算法

需要一个数据结构能够快速反映每个元素在新、旧数据中的位置。全局维护一个set 以每个元素作为key，以entry为value
```
struct Entry {
    var oldCounter: Int = 0
    var newCounter: Int = 0
    
    var oldIndexs = Stack<Int>()
}
```

每一个新、旧的model 都分别对应一个record对象
```
struct Record {
    // entry可以让我们快速访问到数据对应的entry
    var entry: Entry?
    // index则用于记录一个新数据在旧数据中的位置（如果存在的情况下）或一个旧数据在新数据中的位置（如果存在的情况下）
    var index: Int = NSNotFound
}
```

#### 计算过程
**获取元素在新旧数据中的位置**

1. 正序遍历新数据，对应的每个entry，newCounter++，oldIndexes压入NSNotFound
2. 因为栈后进先出，倒序遍历旧数据，对应的每个entry，oldCounter++，oldIndexed压入元素在旧数据中的位置

以ADFGT和ATOXF为例，经过两次遍历，就能获得下面的map数据：

![https://pic2.zhimg.com/80/v2-2b2af757f4f8748d3e9a73dfaf17ebb1_hd.jpg](https://pic2.zhimg.com/80/v2-2b2af757f4f8748d3e9a73dfaf17ebb1_hd.jpg)


### 参考
- https://zhuanlan.zhihu.com/p/35256233
- https://xiangwangfeng.com/2017/03/16/IGListKit-diff-实现简析