---
layout: post
title:  "AutoLayout 使用"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

* 目录
{:toc}


## AutoLayout 系统 API

### 使用官方 API 设置约束
```
let v2 = UIView()
view.addSubview(v2)

v2.translatesAutoresizingMaskIntoConstraints = false
let constraint1 = NSLayoutConstraint(item: v2, attribute: .left,
                                        relatedBy: .equal,
                                        toItem: view, attribute: .left,
                                        multiplier: 1, constant: 0)

let constraint2 = NSLayoutConstraint(item: v2, attribute: .width,
                                        relatedBy: .equal,
                                        toItem: nil, attribute: .notAnAttribute,
                                        multiplier: 1, constant: 100)

let constraint3 = NSLayoutConstraint(item: v2, attribute: .top,
                                        relatedBy: .equal,
                                        toItem: view, attribute: .top,
                                        multiplier: 1, constant: 100)

let constraint4 = NSLayoutConstraint(item: v2, attribute: .height,
                                        relatedBy: .equal,
                                        toItem: nil, attribute: .notAnAttribute,
                                        multiplier: 1, constant: 100)

NSLayoutConstraint.activate([constraint1, constraint2, constraint3, constraint4])
```

### Content Hugging Priority

Content Hugging Priority 默认值为 250

> Hugging => content does not want to grow

使用场景：同一行有两个label，内容少的时候，会根据 ContentHuggingPriority 来确定 label 的 width。优先级高的优先根据内容确定宽度，优先级低的 label 会被拉长。

``` Swift
let label1 = UILabel()
label1.backgroundColor = .red
view.addSubview(label1)
label1.snp.updateConstraints { make in
    make.left.bottom.equalToSuperview()
    
    make.height.equalTo(100)
}

label1.setContentHuggingPriority(UILayoutPriority.defaultLow, for: UILayoutConstraintAxis.horizontal)

let label2 = UILabel()
label2.backgroundColor = .red
view.addSubview(label2)
label2.snp.updateConstraints { make in
    make.right.bottom.equalToSuperview()
    make.height.equalTo(100)
    make.left.equalTo(label1.snp.right).offset(10)
}

label2.setContentHuggingPriority(UILayoutPriority.defaultHigh, for: UILayoutConstraintAxis.horizontal)

label1.text = "123"
label2.text = "abc"
```

### Content Compression Resistance Priority

内容压缩阻力优先级， 默认值为 750

> Compression* Resistance => content does not want to shrink

使用场景：同一行有两个label，内容都太多显示不完全时，则会根据 ContentCompressionResistancePriority 来确定两个 label 的宽度。优先级高的优先显示内容，优先级低的会被压缩。

``` Swift
let label1 = UILabel()
label1.backgroundColor = .red
view.addSubview(label1)
label1.snp.updateConstraints { make in
    make.left.bottom.equalToSuperview()
    
    make.height.equalTo(100)
}
        
label1.setContentCompressionResistancePriority(UILayoutPriority.defaultHigh, for: UILayoutConstraintAxis.horizontal)

let label2 = UILabel()
label2.backgroundColor = .red
view.addSubview(label2)
label2.snp.updateConstraints { make in
    make.right.bottom.equalToSuperview()
    make.height.equalTo(100)
    make.left.equalTo(label1.snp.right).offset(10)
}
        
label2.setContentCompressionResistancePriority(UILayoutPriority.defaultLow, for: UILayoutConstraintAxis.horizontal)
        
label1.text = "你发货发大水念佛fasdfasdfa发..."
label2.text = "佛按时发偶发啊121212123456789"
```

参考：https://stackoverflow.com/questions/15850417/cocoa-autolayout-content-hugging-vs-content-compression-resistance-priority

### firstBaseline 和 lastBaseline

参考：https://stackoverflow.com/questions/33932846/first-baseline-and-last-baseline-properties-in-uistackview

使用场景：多行文本的textView对齐的时候。使用 firstBaseline 第一行文本对齐，使用 lastBaseline 最后一行文本对齐。

### Margin


## 参考

- https://www.jianshu.com/p/de2446b195ce
- 