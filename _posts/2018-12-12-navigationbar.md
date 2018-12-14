---
layout: post
title:  "UINavigationBar UI层级"
categories: blog
---

* 目录
{:toc}

### iOS11之后的view层级
```
UINavigationBar
    - UIBarBackground
    - UINavigationBarContentView
    - UIButtonBarStackView  // 对应 left items
        - UIButtonBarButton // item1
        - UIButtonBarButton // item2
    - UIButtonBarStackView  // 对应 right item
        - UIButtonBarButton
    - UILabel   // 对应 title
    - UITAMICAdaptorView    // 对应 titleView  设置成了UILabel
        - UILabel

```

#### 设置了 title 

![](https://raw.githubusercontent.com/quding0308/quding0308.github.io/master/res/ios_12_title.png)

#### 设置了 titleView

![](https://raw.githubusercontent.com/quding0308/quding0308.github.io/master/res/ios_12_titleview.png)

### iOS11 之前的 view 层级
```
UINavigationBar
    - UIBarBackground
    - UINavigationButton // item
    - UINavigationButton // item
    - UINavigationButton // item
    - UILabel   // 对应 titleView 设置成了UILabel
    - UINavigationItemView    // 对应 title
        - UILabel

```


#### 设置了 title 

![](https://raw.githubusercontent.com/quding0308/quding0308.github.io/master/res/ios10_3_titleview.png)

#### 设置了 titleView

![](https://raw.githubusercontent.com/quding0308/quding0308.github.io/master/res/ios10_3_title.png)

### 在 iOS11 之后 我们可以 通过重设 UIButtonBarStackView 与 UILabel 的约束，来重新调整item间距

``` Swift
重新设置 title 的约束
if let views = view.superview?.subviews, views.count >= 3 {
    if let stackView1 = views[0] as? UIStackView,
        let stackView2 = views[1] as? UIStackView,
        let label = views[2] as? UILabel {
        
        label.snp.remakeConstraints { (maker) in
            maker.left.equalTo(stackView1.snp.right).offset(5)
            maker.right.equalTo(stackView2.snp.left).offset(-5)
        }
    }
}
```