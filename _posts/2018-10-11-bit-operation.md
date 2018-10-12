---
layout: post
title:  "位运算及使用"
categories: blog CocoaPods
---

* 目录
{:toc}

### 基础运算

    按位与(相同位都为1才为1，则为1)
        a & b
        00101
        11100
        -----
        00100

    按位或(相同位只要一个为1即为1)
        a | b
        00101
        11100
        -----
        11101

    按位异或(某位不同则该位为1, 否则该位为0)
        a ^ b
        00101
        11100
        -----
        11001

    按位取反(把0和1全部取反)
        ~ a
        00101
        -----
        11010

    左移
        a << b (把a转为二进制后左移b位 左移后的值 = a * 2^b)
        00101 << 2
        -----
        0010100   

    带符号右移
        a >> b(把a转为二进制后右移b位 有以后的值 = a / 2^b)
        00101 >> 2
        -----
        00010

    无符号右移

### 具体使用

#### 除以2

等价于 右移1位


    var a: Int = 13
    var b = a >> 1

#### 除以4

    var a: Int = 13
    var b = a >> 2

#### 除以8

    var a: Int = 13
    var b = a >> 3

#### 判断奇偶数

等价于 与0x1 按位与运算

    var a: Int = 13
    var c = a & 0x1   // c 是 0 或 1

#### 去掉最后一个1

    var a = x & (x - 1) 
    
    x = 1100
    x - 1 = 1011
    x & (x - 1) = 1000

#### 判断一个32位整数中有多少个1

一直执行 x & (x - 1) 直到返回0  执行的次数就是1的个数

    var a: Int = 25
    var count = 0
    while a > 0 {
        a = a & (a - 1)
        count += 1
    }
    print(count)

#### 检测n是否是2的幂次

如果是2的幂次，则二进制中肯定只有一个1，执行一次  x & (x - 1) 应该返回0

    var a: Int = 17
    a = a & (a - 1)
    if a == 0 {
        print("a是2的幂数")
    } else {
        print("a不是2的幂数")
    }

#### 如果要将整数A转换为B，需要改变多少个bit位？

等价于 A 和 B 有多少个位不相等。A 与 B 做异或操作，结果中1的个数就是不同的位数

    var a: Int = 25
    var b: Int = 26
    var c: Int = a ^ b
    var count = 0
    while c > 0 {
        c = c & (c - 1)
        count += 1
    }
    print(count)  // 2

#### 使用二进制进行子集枚举

使用一个正整数二进制表示的第i位是1还是0，代表集合的第i个数取或者不取。 所以从0到2^n-1总共2^n个整数，正好对应集合的2^n个子集。

    // 一共  2^n种情况
    let arr: [Int] = [1, 2, 3]
    let count = arr.count
    let num = 0x1 << count

    for index in 0..<num {
        var value = index
        var result = [Int]()
        var idx = count - 1
        
        while value != 0 {
            if value & 0x1 == 1 {
                result.append(arr[idx])
            }
            value = value >> 1
            idx -= 1
        }
        
        print("index:\(index), arr:\(result)")
    }

[参考](https://www.zhihu.com/question/38206659)

#### 数组中，只有一个数出现一次，剩下都出现两次，找出出现一次的那个数

原理  a = a ^ b ^ b

数组中所有的数做异或操作，结果就是 只出现了一次的那个数

    let arr: [Int] = [9, 2, 2, 4, 4]
    var result = 0

    for value in arr {
        result = result ^ value
    }
    print("只出现了一次的数: \(result)")

#### 数组中，有两个数出现一次，剩下都出现两次，找出出现一次的那个数

思路：数组元素做完异或操作后，结果就是 result = a ^ b，其中 a b 是只出现了一次的数。result中找到bit位为1的值，凡是这一位为1的放到一个数组，其他放到另一个数组，就转化成了上面的问题，两个数组中，分别有一个数出现了一次。

    func printNum(_ arr: [Int]) {
        var result = 0
        
        for value in arr {
            result = result ^ value
        }
        print("只出现了一次的数: \(result)")
    }

    // 找出两个只出现了一次的数
    let arr: [Int] = [9, 2, 2, 4, 4, 5, 5, 8, 66, 66, 100, 100, 99, 99, 33, 33]
    var result = 0
    for value in arr {
        result = result ^ value
    }

    let t1 = result & (result - 1) // result去掉最后的1后的数
    let t2 = t1 ^ result  // 获取result最后的1

    var arr1 = [Int]()
    var arr2 = [Int]()
    for value in arr {
        result = result ^ value
        if value & t2 != 0{
            arr1.append(value)
        } else {
            arr2.append(value)
        }
    }

    // 两个数组中分别有一个出现了一次的数，单独处理
    printNum(arr1)
    printNum(arr2)