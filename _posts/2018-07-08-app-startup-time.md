---
layout: post
title:  "App启动时间优化"
# date:   2018-07-09 10:000:00 +0800
categories: blog
---

* 目录
{:toc}

## (数据基于iPhone6s)
## 启动时间统计
启动时间 =  main之前 +  main之后
### main之前
使用 DYLD_PRINT_STATISTICS 来测量（每次重启手机，冷启动）
### main之后
main开始执行，到 didFinishLaunch结束(使用系统时间，精确到ms)
#### main之后时间 800ms
- main到didFinishLaunch  150ms
- didFinishLaunch 开始到结束 650 ms 
	- setupAppUI 耗费550ms
	- directEnterYZJ 340ms
		- 工作台延迟加载？ 

### 优化：动态库 更改为 静态库 调用

#### 优化结果
    1500ms -> 900ms ，dylib加载时间降低了500ms
##### 改动前
    Total pre-main time: 1.5 seconds (100.0%)
     		 dylib loading time: 1.0 seconds (67.2%)
     		rebase/binding time: 147.31 milliseconds (9.6%)
     			ObjC setup time:  62.66 milliseconds (4.1%)
     		   initializer time: 289.30 milliseconds (18.9%)
     		   slowest intializers :
     			 libSystem.B.dylib :   9.33 milliseconds (0.6%)
     		  libglInterpose.dylib :  97.74 milliseconds (6.4%)
     					   kdweibo : 159.88 milliseconds (10.4%)
##### 改动后	
       Total pre-main time: 886.80 milliseconds (100.0%)
     		 dylib loading time: 492.76 milliseconds (55.5%)
     		rebase/binding time:  70.12 milliseconds (7.9%)
     			ObjC setup time:  58.83 milliseconds (6.6%)
     		   initializer time: 264.95 milliseconds (29.8%)
     		   slowest intializers :
     			 libSystem.B.dylib :   7.30 milliseconds (0.8%)
     		  libglInterpose.dylib :  65.98 milliseconds (7.4%)
     		 libMTLInterpose.dylib :  29.05 milliseconds (3.2%)
     					   kdweibo : 193.56 milliseconds (21.8%)


#### 调整前，项目中一共有29个第三方动态库
![](https://leanote.com/api/file/getImage?fileId=5a98b698ab64416bd300061c)
### 调整后，还有3个
![](https://leanote.com/api/file/getImage?fileId=5a98b6aaab64416de5000592)

### 第三方Swift动态库(6)
- RxSwift、RxCocoa、RxDataSourcess
- SnapKit、PromiseKit、MenuItemKit
一共6个，打包成静态framework直接导入项目中。pod不支持Swift静态库调用，而且这几个库很少会更新，以framework方式引入，也可以减少编译时间。

### 第三方oc库(9)
- Countly、SocketRocket、SHSPhoneComponent、DACircularProgress、DGActivityIndicatorView、Masonry 通过修改podspec增加 s.static\_framework=true 来通过pod直接通过静态库方式引入项目(使用方式不变)
- MBProgressBar、AFNetworking、JsonModel 由于第三方pod有依赖，无法通过pod编译为静态库，手动把代码放到了自己的pod库中
- SDWebImage 直接通过framework导入静态库

### yzj的pod库(3)
- KDKit 不处理（自己的Swift库，会经常更新）
- KDFoundation、YZJOauthLib、KDNetwork、KDMerc修改podspec，以静态库方式引入
- 


### 第三方库版本比较
#### 更新前
    Using AFNetworking (3.1.0)
    Using AMap2DMap (4.6.0)
    Using AMapFoundation (1.4.3)
    Using AMapLocation (2.4.0)
    Using AMapSearch (5.3.0)
    Using Countly (16.06.4)
    Using DACircularProgress (2.3.1)
    Using DGActivityIndicatorView (2.1.1)
    Using Differentiator (2.0.2)
    Using FBRetainCycleDetector (0.1.4)
    Using FLAnimatedImage (1.0.12)
    Using FMDB (2.6.2)
    Using JSONModel (1.7.0)
    Using KDFoundation (0.1.1)
    Using KDKit (0.1.28)
    Using KDMerc (0.1.9)
    Using KDNetwork (0.7.0)
    Using MBProgressHUD (1.0.0)
    Using MLeaksFinder (1.0.0)
    Using Masonry (1.0.2)
    Using MenuItemKit (3.0.0)
    Using PromiseKit (4.4.4)
    Using RxCocoa (3.6.1)
    Using RxDataSources (2.0.2)
    Using RxSwift (3.6.1)
    Using SDWebImage (4.1.2)
    Using SHSPhoneComponent (2.31)
    Using SQLCipher (3.1.0)
    Using SSZipArchive (1.8.1)
    Using SafeKit (1.4.1)
    Using SnapKit (3.2.0)
    Using SocketRocket (0.4.2)
    Using WechatOpenSDK (1.7.9)
    Using YZJOauthLib (1.1.5)
    Using lottie-ios (2.1.5)

### 更新后
    Using AMap2DMap (4.6.0)
    Using AMapFoundation (1.4.3)
    Using AMapLocation (2.4.0)
    Using AMapSearch (5.3.0)
    Using Countly_static (16.06.4)
    Using DACircularProgress_static (2.3.1)  
    Using DGActivityIndicatorView_static (2.1.1)
    Using FMDB (2.6.2)
    Using KDFoundation 0.1.3
    Using KDKit (0.1.28)
    Using KDMerc (0.1.8)
    Using KDNetwork 0.7.0 
    Using Masonry_static (1.1.0) 【从1.0.2更新】
    Using SHSPhoneComponent_static (2.31)
    Using SQLCipher (3.1.0)
    Using SocketRocket_static (0.4.1)  【待更新为0.4.2】
    Using WechatOpenSDK (1.7.9)
    Using YZJOauthLib 1.1.5
    
    // 其他 手动管理的库
    Using JSONModel (1.7.0)
    Using AFNetworking (3.1.0)
    MBProgressBar (1.0.0)
    
    Differentiator.framework ()
    // RxBlocking.framework ()
    RxCocoa.framework (3.6.1)
    RxDataSources.framework (2.0.3) 【从2.0.2更新】
    RxSwift.framework (3.6.1)
    SDWebImage.framework (4.2.3)  【从4.1.2更新】

    
    
    


