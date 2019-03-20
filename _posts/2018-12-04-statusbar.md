---
layout: post
title:  "iOS中 Status Bar 设置"
categories: blog
---

* 目录
{:toc}


### Info.plist 文件设置
#### Status bar is initially hidden  

设置status bar的初始状态，包括在LaunchScreen时，状态栏是否显示。

```
<key>UIStatusBarHidden</key>
<true/>
```

#### Status bar style

设置默认status bar样式
```
<key>UIStatusBarStyle</key>
<string>UIStatusBarStyleDefault</string>
```

#### View controller-based status bar appearance

在 ViewController页面，VC是否可以控制 status bar的状态。 true 为可以，则prefersStatusBarHidden、preferredStatusBarStyle生效。

```
<key>UIViewControllerBasedStatusBarAppearance</key>
<true/>
```

**注意一个坑：**

如果Status bar is initially hidden为true，当View controller-based status bar appearance 为 true时， vc的status bar 默认是显示的。

### status bar 随 vc 变化
#### 方式1

**View controller-based status bar appearance 必须设置为YES，才可以使用VC中的下面方法**

```
- (BOOL)prefersStatusBarHidden {
    return YES;
}

- (UIStatusBarStyle)preferredStatusBarStyle {
    return UIStatusBarStyleDefault; // 文字为黑色
    // return UIStatusBarStyleLightContent;    // 文字为白色
}

- (UIStatusBarAnimation)preferredStatusBarUpdateAnimation {
    return UIStatusBarAnimationFade;
}
```

**一个坑**

当vc被push出来，则上面的方法不会调用vc的，而是调用 nav vc的。所以想要改变status bar，需要设置 nav vc。可以通过 自定义 UINavigationController 子类来实现。

#### 方式2

**View controller-based status bar appearance 必须设置为NO，才可以使用下面的方法**

**方式1与方式2是互斥的，只能选择一种**

**下面的方法已经被废弃了，建议使用方式1**

```
[[UIApplication sharedApplication] setStatusBarHidden:YES];
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];        
```

### status bar 设置颜色

1.跟随导航栏改变颜色
status bar 的颜色默认跟随导航栏的颜色变化，所以一般设置 导航栏颜色即可

```
[self.navigationController.navigationBar setBarTintColor:[UIColor redColor]];
```

2.自定义颜色
```
/**
 设置状态栏背景颜色
 @param color 设置颜色
 */
- (void)setStatusBarBackgroundColor:(UIColor *)color {
    UIView *statusBar = [[[UIApplication sharedApplication] valueForKey:@"statusBarWindow"] valueForKey:@"statusBar"];
    if ([statusBar respondsToSelector:@selector(setBackgroundColor:)]) {
        statusBar.backgroundColor = color;
    }
}
```

### 主动刷新 status bar

当 status bar 的 hidene、style 有变化时，可以主动刷新

```
UIViewController
setNeedsStatusBarAppearanceUpdate
```

**一个坑**

如果 vc是被push出来的 则需要调用 nav vc 的 setNeedsStatusBarAppearanceUpdate 才能生效

```
[self.navigationController setNeedsStatusBarAppearanceUpdate];
```

### Bug记录
在 webview 播放视频，全屏横屏播放 然后点击关闭视频窗口，可能会导致 webview 的vc的status bar 与 navigation bar 重合。当检测到视频窗口关闭后，应该重新刷新 navigation vc 的status bar 的状态。

```
// 监听关闭
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(avplayerDidClosed:) name:UIWindowDidBecomeHiddenNotification object:nil];

- (void)avplayerDidClosed:(NSNotification *)notif {
    self.isStatubBarHidden = YES;
    [self.navigationController setNeedsStatusBarAppearanceUpdate];
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        self.isStatubBarHidden = NO;
        [self.navigationController setNeedsStatusBarAppearanceUpdate];
    });
}
```

### 设置维度

App级别 -> VC级别

#### App级别

Info.plist 中设置 

```
<key>UIStatusBarHidden</key>
<true/>
<key>UIStatusBarStyle</key>
<string>UIStatusBarStyleLightContent</string>
```

或者使用 UIApplication 的方法：

```
[[UIApplication sharedApplication] setStatusBarHidden:YES];
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];        
```

VC级别：

```
- (BOOL)prefersStatusBarHidden {
    return YES;
}

- (UIStatusBarStyle)preferredStatusBarStyle {
    return UIStatusBarStyleDefault; // 文字为黑色
    // return UIStatusBarStyleLightContent;    // 文字为白色
}

- (UIStatusBarAnimation)preferredStatusBarUpdateAnimation {
    return UIStatusBarAnimationFade;
}
```

### 参考

- https://www.jianshu.com/p/534054a8c897