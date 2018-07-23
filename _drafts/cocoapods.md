
## CocoaPods在项目中使用

### 基础知识
##### 什么是CocoaPods
a dependency manager for Swift and Objective-C Cocoa projects

##### workspace与project

##### 静态库与动态库

##### Swift中的umbrella与oc的framework

##### .framework/ .a 的区别

### 常用命令

##### pod install
- 已经在Podfile.lock中的库，不会尝试更新版本。
- 新增或删除的库后，会重新修改Podfile.lock文件。对其他库的版本不会做操作

##### pod update

pod update // update local specs repositories
pod update --no-repo-update 

##### pod outdated
CocoaPods会列出所有在Podfile.lock中的有新版本的pod库。这意味着当你对这些pod使用pod update PODNAME时，他们会更新（只要新版本仍然遵守你在Podfile中做的类似于pod 'MyPod', '~>x.y'这样的限制）


##### pod repo

    pod repo list
    pod repo add name url
    pod repo remove name
    pod repo push name file.podspec
    pod repo update name


### Podfile中管理依赖



#### 不同的依赖方式



### 使用Pod管理自己的代码

#### 创建自己的私有仓库

#### 创建一个Pod，更快的改代码



### 其他问题：

#### 还有别的工具可以使用吗？
CocoaPods与Carthage、git submodule、自己手动管理

##### CocoaPods与Carthage、git submodule使用的区别



### 参考：

- https://reallifeprogramming.com/carthage-vs-cocoapods-vs-git-submodules-9dc341ec6710


