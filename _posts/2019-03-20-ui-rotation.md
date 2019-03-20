---
layout: post
title:  "iOS中横竖屏相关知识"
categories: blog
---

* 目录
{:toc}

## App级别设置

App级别的设置 是在 Info.plist中：

Info.plist 设置 了App支持的方向。自动旋转的方向只会在这几个里面。
```
// iPhone
	<key>UISupportedInterfaceOrientations</key>
	<array>
		<string>UIInterfaceOrientationPortrait</string>
		<string>UIInterfaceOrientationLandscapeRight</string>
		<string>UIInterfaceOrientationLandscapeLeft</string>
	</array>

// iPad
	<key>UISupportedInterfaceOrientations~ipad</key>
	<array>
		<string>UIInterfaceOrientationPortrait</string>
		<string>UIInterfaceOrientationPortraitUpsideDown</string>
		<string>UIInterfaceOrientationLandscapeLeft</string>
		<string>UIInterfaceOrientationLandscapeRight</string>
	</array>
```


## Window级别设置

在AppDelegate中的回调方法中，可以为 某个Window设置
```
- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
    return UIInterfaceOrientationMaskPortrait | UIInterfaceOrientationMaskLandscapeRight;
}
```

## UIViewController级别设置

```
// VC
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskLandscape;
}

- (BOOL)shouldAutorotate {
    return NO;
}
```

### 指定某一个VC横屏

使用案例：视屏全屏播放、全屏查看图标等信息

使用场景：整体App中不支持自动旋转，希望某个 VC 可以横屏

使用限制：该VC必须是 某个Window的rootVC 或者 presentedVC

```
// VC
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskLandscape;
}
```

### App支持自适应旋转屏幕 


#### Info.plist 设置

Info.plist 设置 了App支持的方向。自动旋转的方向只会在这几个里面。
```
	<key>UISupportedInterfaceOrientations</key>
	<array>
		<string>UIInterfaceOrientationPortrait</string>
		<string>UIInterfaceOrientationLandscapeRight</string>
		<string>UIInterfaceOrientationLandscapeLeft</string>
	</array>
```

#### 设置VC支持自动旋转：

设置 VC 支持自动旋转屏幕
```
- (BOOL)shouldAutorotate {
    return YES;
}

// 基于 Info.plist 的设置，可以设置仅支持更小的子集
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskLandscape;
}
```

#### 设置VC不支持自动旋转

// 适用于：App整体支持自动旋转，具体某个VC不支持
```
- (BOOL)shouldAutorotate {
    return NO;
}
```

## 总结 

从3个维度设置 横竖屏、自动旋转屏幕 

App -> window -> VC

- 默认情况下，会根据3个维度的设置 决定 VC的横竖屏。
- 如果VC设置了屏幕方向，则以VC为准

## 注意：

- 如果Info.plist支持多个方向，则vc的shouldAutorotate默认为YES
- 如果App或window只支持一个方向，则vc的shouldAutorotate默认为NO
