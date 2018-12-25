---
layout: post
title:  "AutoLayout 使用"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

* 目录
{:toc}


## 系统 API

### 使用官方 API 设置约束
```
view.addSubview(v2)
v2.translatesAutoresizingMaskIntoConstraints = false
let c1 = NSLayoutConstraint(
            item: v2, attribute: .left,
            relatedBy: .equal,
            toItem: view, attribute: .left,
            multiplier: 1, constant: 0)

let c2 = NSLayoutConstraint(
            item: v2, attribute: .width,
            relatedBy: .equal,
            toItem: nil, attribute: .notAnAttribute,
            multiplier: 1, constant: 100)

let c3 = NSLayoutConstraint(
            item: v2, attribute: .top,
            relatedBy: .equal,
            toItem: view, attribute: .top,
            multiplier: 1, constant: 100)

let c4 = NSLayoutConstraint(
            item: v2, attribute: .height,
            relatedBy: .equal,
            toItem: nil, attribute: .notAnAttribute,
            multiplier: 1, constant: 100)

// 约束生效
NSLayoutConstraint.activate([c1, c2, c3, c4])

// 约束失效
NSLayoutConstraint.deactivate([c1, c2]) // 效率更高
// 等价于
c1.isActive = false
c2.isActive = false
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


### firstBaseline 和 lastBaseline

使用场景：多行文本的textView对齐的时候。使用 firstBaseline 第一行文本对齐，使用 lastBaseline 最后一行文本对齐。

firstBaseline:

![](https://raw.githubusercontent.com/quding0308/quding0308.github.io/master/res/firstbaseline.png)

lastBaseLine:

![](https://raw.githubusercontent.com/quding0308/quding0308.github.io/master/res/lastbaseline.png)

### Margin

当 superView 的 margin 改变时，所有依赖 margin 的 subviews 的布局都会调整。

margin 如果设置为负数，会默认为默认为0来处理。

``` Swift
view.layoutMargins = UIEdgeInsetsMake(0, 0, 20, 0)

let label2 = UILabel()
label2.backgroundColor = .red
view.addSubview(label2)
label1.text = "123"
label2.snp.updateConstraints { make in
    make.left.right.equalToSuperview()
    make.height.equalTo(200)

    make.bottomMargin.equalToSuperview()
    // make.centerYWithinMargins.equalToSuperview()
}

after(seconds: 3).done {
    UIView.animate(withDuration: 2, animations: {
        self.view.layoutMargins = UIEdgeInsetsMake(0, 0, 120, 0)
        self.view.layoutIfNeeded()
    })
}
```

### safeArea 和 topLayoutGuide 和 bottomLayoutGuide

我们可以 通过 safeArea 来获取 navigation bar 和 tabbar 占用的高度，从而 设置 subview 的高度（iOS11以下使用topLayoutGuide、bottomLayoutGuide）

safeArea：

```
self.edgesForExtendedLayout = UIRectEdge.all

带刘海的屏幕 safeArea： 
UIEdgeInsets(top: 88.0, left: 0.0, bottom: 34.0, right: 0.0)

不带刘海的屏幕 safeArea：
UIEdgeInsets(top: 64.0, left: 0.0, bottom: 0.0, right: 0.0)
```

在 VC 中 根据 safeArea 设置 view 大小：

``` Swift
override func viewDidLayoutSubviews() {
    v1.snp.updateConstraints { make in
        if #available(iOS 11.0, *) {
            make.top.equalToSuperview().offset(self.view.safeAreaInsets.top)
        } else {
            // Fallback on earlier versions
            make.top.equalToSuperview().offset(self.topLayoutGuide.length)
        }
    }

    v2.snp.updateConstraints { make in
        if #available(iOS 11.0, *) {
            make.bottom.equalToSuperview().offset(-self.view.safeAreaInsets.bottom)
        } else {
            // Fallback on earlier versions
            make.bottom.equalToSuperview().offset(-self.bottomLayoutGuide.length)
        }
    }
}
```

## 参考

- [https://www.jianshu.com/p/de2446b195ce](https://www.jianshu.com/p/de2446b195ce)
- [https://stackoverflow.com/questions/33932846/first-baseline-and-last-baseline-properties-in-uistackview](https://stackoverflow.com/questions/33932846/first-baseline-and-last-baseline-properties-in-uistackview)
- [https://stackoverflow.com/questions/15850417/cocoa-autolayout-content-hugging-vs-content-compression-resistance-priority](https://stackoverflow.com/questions/15850417/cocoa-autolayout-content-hugging-vs-content-compression-resistance-priority)
