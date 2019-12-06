---
layout: post
title:  "iOS 构建 - xcodeproj 命令"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

* 目录
{:toc}

## 概述

``` shell
project
    - 



```


## Xcodeproj 命令

CocoaPods 中使用的库，用来查看、修改 project.pbxproj 文件。

github 地址：

https://github.com/CocoaPods/Xcodeproj

api 文档：

https://www.rubydoc.info/list/gems/xcodeproj/class

ruby 语法：

https://www.runoob.com/ruby/ruby-tutorial.html

安装

``` shell
gem install xcodeproj
```

## 命令行使用

``` shell

xcodeproj show 查看 project.pbxproj

把 project.pbxproj 输出到一个文件中
xcodeproj show > project.yaml
```

## 脚本

### 获取 project 信息

``` ruby 
require 'xcodeproj'

project_path = File.join(File.dirname(__FILE__), "../../../kdweibo.xcodeproj")
project = Xcodeproj::Project.open(project_path)

# project 信息
puts project.project_dir
puts project.path
puts project.build_configurations   # Debug 、Release
puts project.main_group
puts project.frameworks_group

schemes 信息
schemes = project.embedded_targets_in_native_target
puts schemes

```

### target 信息

``` ruby
target 信息 ==============
puts project.targets

main_target = nil
for target in project.targets
    # target 类型为 AbstractTarget 
    puts target.name

    if target.name == "kdweibo" 
        main_target = target
        break
    end
end
```

### 修改 bundle id 和 证书

``` ruby 
for config in main_target.build_configurations 
    # confige 为 XCBuildConfiguration 类型
    puts config.name

    if config.name == "Release" 
        map = config.build_settings
        # puts map
        # puts "PRODUCT_BUNDLE_IDENTIFIER  " + map["PRODUCT_BUNDLE_IDENTIFIER"]
        map["PRODUCT_BUNDLE_IDENTIFIER"] = "com.test"

        map["PROVISIONING_PROFILE_SPECIFIER"] = "com.test.xx"
        # PROVISIONING_PROFILE 可以不用改
        # map["PROVISIONING_PROFILE"] = "8fb5cb38-c2c9-4fff-912a-a4f3394acbc8" 
    end
end
```

### build_setting 的设置

``` ruby
for config in main_target.build_configurations 
    # confige 为 XCBuildConfiguration 类型
    puts config.name

    if config.name == "Release" 
        map = config.build_settings
        # puts map
        
        # 读取
        puts "ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES  " + map["ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES"]
        puts "OTHER_CFLAGS  " + map["OTHER_CFLAGS"]
        puts "SWIFT_VERSION  " + map["SWIFT_VERSION"]
        puts "HEADER_SEARCH_PATHS  " + map["HEADER_SEARCH_PATHS"]

        # 设值
        # 修改某个 build setting 的选项(最后要调用 project.save )
        # map["ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES"] = "NO"
    end

    if config.name == "Release" 
        # PBXFileReference 会返回 上一个 xcconfig 文件 Pods-kdweibo.release.xcconfig
        ref = config.base_configuration_reference
        puts ref.name
        puts ref.path
    end
end
```

### Linked Frameworks 中 Frameworks

``` ruby
# PBXFrameworksBuildPhase
PBXFrameworksBuildPhase = target.frameworks_build_phases

puts PBXFrameworksBuildPhase.uuid
puts PBXFrameworksBuildPhase.display_name

# 遍历 Frameworks
for tmp in PBXFrameworksBuildPhase.files_references
    # PBXFileReference
    puts tmp.name
    puts tmp.path
end

# 添加 framework
file_ref = project.new_file('Wellsign_ReaderSDK.framework')
PBXFrameworksBuildPhase.add_file_reference(file_ref)

# 删除 framework  可以通过遍历 Frameworks，根据 name 获取到 file_ref
PBXFrameworksBuildPhase.remove_file_reference(file_ref)
```

### Embed App Extensions 添加

``` ruby
embedAppExtension = target.copy_files_build_phases[0]
for tmp in embedAppExtension.files_references 
    puts tmp.name
    puts tmp.path
end

# 新增一个 appex 文件
# embedAppExtension.add_file_reference(project.new_file('PacketTunnel.appex'))
```

### Target Dependencies

``` ruby
for tmp in main_target.dependencies
    # dependency 类型为 PBXTargetDependency 
    # https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXTargetDependency
    puts tmp.target
end
```

### 其他
``` ruby 
# PBXCopyFilesBuildPhase
arrayPBXCopyFilesBuildPhase = target.copy_files_build_phases
puts arrayPBXCopyFilesBuildPhase

# shell 脚本
arrayPBXShellScriptBuildPhase = target.shell_script_build_phases

# 所有的改动最后都需要调用 project.save 才会生效
project.save
```

## 参考：
- https://www.jianshu.com/p/cca701e1d87c