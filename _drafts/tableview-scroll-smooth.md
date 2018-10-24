---
layout: post
title:  "TableView优化"
categories: blog
---

* 目录
{:toc}

### 首次加载慢优化

首次只加载部分cell


#### 滑动时不渲染，停止后渲染当前屏幕的cell

渲染过程中 可能会出现白屏。



#### 刷新tableview时，通过比较，只更新 有变化的cell

RXDatasource中的 Differentiator 会计算出需要刷新的cell，不需要全量更新


#### 多层view合成为image再渲染，减少层级



#### 高度计算、富文本计算后的结果 缓存下来





