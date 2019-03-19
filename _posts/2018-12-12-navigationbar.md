---
layout: post
title:  "UINavigationBar 相关问题"
categories: blog
---

* 目录
{:toc}

## UINavigationBar UI层级

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

![](/assets/img/ios_12_title.png)

#### 设置了 titleView

![](/assets/img/ios_12_titleview.png)

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

![](/assets/img/ios10_3_titleview.png)

#### 设置了 titleView

![](/assets/img/ios10_3_title.png)

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

#### UIBarBackground的层级

```
UINavigationBar : UIView
    - _UIBarBackground
        - UIImageView
        - UIVisualEffectView
          - UIVisualEffectBackdropView
          - UIVisualEffectSubView
          - UIVisualEffectSubView
    - _UINavigationBarContentView
        - _UIButtonBarButton
        - UILabel // title
```

### 设置完全透明：

代码：

```
[self.navigationController.navigationBar setBackgroundImage:[UIImage new] forBarMetrics:UIBarMetricsDefault];    
self.navigationController.navigationBar.shadowImage = [UIImage new];
self.navigationController.navigationBar.translucent = YES;
```

去掉了模糊效果，变透明后的层级：

```
UINavigationBar : UIView
    - _UIBarBackground
        - UIImageView
    - _UINavigationBarContentView
        - _UIButtonBarButton
        - UILabel // title
```

### 设置背景色

#### 方式1

```
[navigationBar setBarTintColor:[UIColor whiteColor]];

// 此时背景色默认带半透明 0.85 
UI层级：
    - _UIBarBackground
        - UIImageView
        - UIVisualEffectView
          - UIVisualEffectBackdropView
          - UIVisualEffectSubView
          - UIVisualEffectSubView // 设置的 tint color ，透明度 0.85
```

#### 方式2

```
[navigationBar setBarTintColor:[UIColor whiteColor]];
[navigationBar setTranslucent:NO];
UI层级：
    - _UIBarBackground  // 背景色为 tintColor 没有半透明 (_UIBarBackground 没有 subview)
```
