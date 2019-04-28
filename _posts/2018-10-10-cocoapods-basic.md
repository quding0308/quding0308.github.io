---
layout: post
title:  "CocoaPods基础知识"
categories: blog CocoaPods
---


* 目录
{:toc}

### RubyGems  

Ruby dependency management

命令：

    gem install xx
    gem install cocoapods -v 0.35.0   // 安装指定版本

    gem uninstall xx

### Bundler

Bundler creates a consistent application environment for your application, by 
allowing you to specify the version of libraries.

命令：

    Gemfile 中配置 lib 和 版本号

    bundle install 命令执行后，生成Gemfile.lock

    别人调用bundle install 后 会使用Gemfile.lock中指定的版本


    bundle update 可以根据Gemfile重新生成Gemfile.lock文件

### 安装 bundle 和 对应的 CocoaPods

在执行 sh pod_install.sh 之前需要先执行两个命令
```
// 安装 bundler
gem install bundler
// 安装对应的 CocoaPods
bundle install
```

#### 使用统一的CocoaPods版本号

在项目中通过使用Gemfile 我们可以指定CocoaPods或fastlane的版本号，保证团队在某个项目中都使用统一的版本。

新建Gemfile

    source 'https://rubygems.org'
    gem 'cocoapods', '~> 1.5.3'

执行完 bundle install 后，会生成Gemfile.lock文件。

团队内其他成员使用时，也需要执行一次 bundle install。

通过exec命令，使用Gemfile中定义的CocoaPods版本

    bundle exec pod update 
    bundle exec pod install

    如果直接执行 pod install 会使用机器上安装的最新版本

#### 使用官方master分支的CocoaPods

Gemfile

    source 'https://rubygems.org'

    # gem 'cocoapods', '~> 1.5.3'
    # gem 'cocoapods-keys'

    gem 'cocoapods', :git => 'http://github.com/CocoaPods/CocoaPods.git'
    gem 'cocoapods-core', :git => 'https://github.com/CocoaPods/Core.git'
    gem 'xcodeproj', :git => 'https://github.com/CocoaPods/Xcodeproj.git'

    gem 'cocoapods-keys', :git => 'https://github.com/orta/cocoapods-keys.git'

    gem 'cocoapods-binary'

    gem 'xcpretty'
    gem 'shenzhen'
    gem 'sbconstants'
    gem 'fastlane'

### CocoaPods

由Ruby语言开发，整个思路也跟Bundle类似。Podfile对应Gemfile，Podfile.lock 对应 Gemfile.lock

Cocoapods项目具体由下面几个gem组成：
- CocoaPods
- Cocoapods-Core
- Xcodeproj 
- Cocoapods-Downloader
- CLAide
- Molinillo

官方文档：

[https://guides.cocoapods.org/contributing/components.html](https://guides.cocoapods.org/contributing/components.html)

其中，Xcodeproj 命令 可以用来修改Xcode项目的基本配置，包括 xcworkspace 和 xcccofign文件

### 如何为CocoaPods贡献代码，以及自定义源码？

官方文档：

[https://guides.cocoapods.org/contributing/dev-environment.html](https://guides.cocoapods.org/contributing/dev-environment.html)



