
## CocoaPods在项目中使用

### 基础知识
##### 什么是CocoaPods
a dependency manager for Swift and Objective-C Cocoa projects

##### workspace与project

##### 静态库与动态库

##### Swift中的umbrella与oc的framework

##### .framework/ .a 的区别

### 常用命令

#### pod init 

在一个Xcode项目的根目录执行，生成Podfile文件

#### pod install

初始化一个项目

- 生成xcworkspace文件
- Podfile.lock
- Pods目录，里面会有默认的文件

使用：
- 已经在Podfile.lock中的库，不会尝试更新版本(不修改Podfile的情况下)。
- 会更新版本的情况
    - 新增或删除的库后，会重新修改Podfile.lock文件。对其他库的版本不会做操作
    - 写死的版本号，更改版本后 会更新
    - 注意：直接指向某个分支，只有第一次会主动拉取，之后无法更新到最新的代码

#### pod update

1. 更新pod本地仓库
2. 根据Podfile更新第三方库

pod update // update local specs repositories  慢
pod update --no-repo-update 

#### pod outdated
CocoaPods会列出所有在Podfile.lock中的有新版本的pod库。这意味着当你对这些pod使用pod update PODNAME时，他们会更新（只要新版本仍然遵守你在Podfile中做的类似于pod 'MyPod', '~>x.y'这样的限制）

#### pod repo

管理pod仓库

    pod repo list
    pod repo add name url
    pod repo remove name
    pod repo push name file.podspec
    pod repo update name


### Podfile中管理依赖

#### 依赖使用：

    pod 'AFNetworking', '~> 3.0.0'  // 会升级到 3.0.4
    pod 'AFNetworking', '~> 3.0'    // 会升级到3.2.1
    pod 'AFNetworking', '3.0'    // 会下载3.0.0
    pod 'AFNetworking', '= 3.0'    // 会升级到3.0

    // local
    pod 'Alamofire', :path => '~/Documents/Alamofire'

    // default master branch
    pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git' 
    
    pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :branch => 'dev'
    
    pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :tag => '3.1.1'

    pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :commit => '0f506b1c45'

    pod 'Reveal-SDK', :configurations => ['Debug']

    // Subspecs
    pod 'AFNetworking', :subspecs => ['NSURLSession', 'Reachability', 'Security', 'Serialization']

    // 加载不在中央仓库中的库
    pod 'KDDebugKit', :podspec => 'http://172.20.10.91/quding/KDDebugKit/blob/master/KDDebugKit.podspec'


#### 动态库&静态库管理
   
    use_frameworks!

### 使用Pod管理自己的代码

#### 创建自己的私有仓库

1. 创建一个空的git仓库
2. 直接push Pod库即可

#### 创建一个Pod，更快的改代码

1.直接依赖本地代码，改代码

    修改好后，最后push到中央库：
    pod repo push 10-mobile-pod_specs KDMerc.podspec --use-libraries --allow-warnings

##### Podspec配置
    
    // 依赖库  注意：如果依赖静态库，则该lib也只能为静态库；静态库 可以 依赖 动态库吗？？auth库依赖mb？
    s.dependency 'MBProgressHUD'
    #  s.dependency 'KDFoundation', '0.1.1'
    s.dependency 'KDFoundation'

    // 静态库方式引用
    s.static_framework = true




### 其他问题：

#### 还有别的工具可以使用吗？
CocoaPods与Carthage、git submodule、自己手动管理

##### CocoaPods与Carthage、git submodule使用的区别

- 复杂度，使用是否方便
    - 团队是否都掌握了
    - 修改代码，升级版本
- 编译时间消耗(Pod编译，iMac下大概25s) 

经常遇到的警告：

    The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation."

    可以暂时注释 检查的脚本，先运行项目。



### 参考：

- https://reallifeprogramming.com/carthage-vs-cocoapods-vs-git-submodules-9dc341ec6710


