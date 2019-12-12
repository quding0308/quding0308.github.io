---
layout: post
title:  "iOS 构建 - xcodeproj 命令"
# date:   2018-06-23 10:05:05 +0800
categories: blog
---

* 目录
{:toc}

## pbxproject 文件

整体的结构：

``` javascript
xcode 项目中整体关系

PBXProject
    targets: [PBXNativeTarget, PBXNativeTarget]
    buildConfigurationList:[buildConfiguration, buildConfiguration] // Debug Release
    mainGroup:"29B97314FDCFA39411CA2CEA"

PBXNativeTarget
    buildConfigurationList:[buildConfiguration, buildConfiguration] // Debug Release
    buildPhases
        Sources
        Resources
        Frameworks
        Embed App Extensions
        [CP] Check Pods Manifest.lock
        [CP] Copy Pods Resources
    buildRules
    dependencies
        PBXTargetDependency
        PBXTargetDependency

添加一个文件时，
首先会添加到 PBXFileReference 中获取到 file.fileRef，然后再把 fileRef 添加到某个 PBXGroup ，就可以在 xcode 中正常展示了。
然后会添加到 PBXBuildFile ，此时获取到了 file.uuid ，之后该文件添加到 Sources、Resources 等时，使用 uuid
```

### PBXFileReference section

存储了每个真实文件的信息(a reference to the actual file)，包含 uuid、path、name。加入项目的文件首先要创建一个 PBXFileReference ，生成唯一的 uuid ，之后其他 section 使用 uuid 标识。

``` shell
1E65C106239E1DCE0046BDDA /* AppDelegate.m */ = {isa = PBXFileReference; lastKnownFileType = sourcecode.c.objc; path = AppDelegate.m; sourceTree = "<group>"; };
1E65C126239E22B40046BDDA /* NetworkExtension.framework */ = {isa = PBXFileReference; lastKnownFileType = wrapper.framework; name = NetworkExtension.framework; path = System/Library/Frameworks/NetworkExtension.framework; sourceTree = SDKROOT; };

```

### PBXGroup group
对应 Xcode 中的 group 和 group 之间的嵌套 ，每个 group 中包含对应资源或代码文件，每个 group 也有唯一的 uuid

``` shell
/* 在 PBXProject section 设置为 mainGroup，项目的根 group */
1E65C0F9239E1DCE0046BDDA = { 
    isa = PBXGroup;
    children = (
        1E65C104239E1DCE0046BDDA /* Test1 */,
        1E65C103239E1DCE0046BDDA /* Products */,
        1E65C125239E22B40046BDDA /* Frameworks */,
    );
    sourceTree = "<group>";
};
1E65C104239E1DCE0046BDDA /* Test1 */ = {
    isa = PBXGroup;
    children = (
        1E65C105239E1DCE0046BDDA /* AppDelegate.h */,
        1E65C106239E1DCE0046BDDA /* AppDelegate.m */,
        1E65C10E239E1DCE0046BDDA /* Main.storyboard */,
        1E65C111239E1DCF0046BDDA /* Assets.xcassets */,
        1E65C113239E1DCF0046BDDA /* LaunchScreen.storyboard */,
        1E65C116239E1DCF0046BDDA /* Info.plist */,
        1E65C117239E1DCF0046BDDA /* main.m */,
    );
    path = Test1;
    sourceTree = "<group>";
};
```

### PBXProject section

存储 project 的信息

``` shell
/* 在根节点设置为 rootObject = 1E65C0FA239E1DCE0046BDDA */
1E65C0FA239E1DCE0046BDDA /* Project object */ = {   
    isa = PBXProject;
    buildConfigurationList = 1E65C0FD239E1DCE0046BDDA /* Build configuration list for PBXProject "Test1" */;
    compatibilityVersion = "Xcode 9.3";
    mainGroup = 1E65C0F9239E1DCE0046BDDA;   /* 对应 PBXGroup section*/
    productRefGroup = 1E65C103239E1DCE0046BDDA /* Products */;
    projectDirPath = "";
    projectRoot = "";
    targets = ( /* 包含的 target */
        1E65C101239E1DCE0046BDDA /* Test1 */,
    );
};
```

### PBXNativeTarget section
定义了每个 target 包含的属性

``` shell
1E65C101239E1DCE0046BDDA /* Test1 */ = {
    isa = PBXNativeTarget;
    buildConfigurationList = 1E65C11B239E1DCF0046BDDA /* Build configuration list for PBXNativeTarget "Test1" */;
    buildPhases = ( /* 定义了 target 包含的 文件、库、资源 */
        1E65C0FE239E1DCE0046BDDA /* Sources */,
        1E65C0FF239E1DCE0046BDDA /* Frameworks */,
        1E65C100239E1DCE0046BDDA /* Resources */,
    );
    buildRules = (
    );
    dependencies = (
    );
    name = Test1;
    productName = Test1;
    productReference = 1E65C102239E1DCE0046BDDA /* Test1.app */;
    productType = "com.apple.product-type.application";
};
```

### PBXFrameworksBuildPhase section

每个 section 定义一组 framework， 在 framework 的 buildPhases 中使用。    

如果想新增加一个target，需要在 

``` shell
1E65C0FF239E1DCE0046BDDA /* Frameworks */ = {
    isa = PBXFrameworksBuildPhase;
    buildActionMask = 2147483647;
    files = (
        1E65C127239E22B40046BDDA /* NetworkExtension.framework in Frameworks */,
    );
    runOnlyForDeploymentPostprocessing = 0;
};
```

### PBXSourcesBuildPhase section

对应项目中 Build Phases -> Compile Sources 中要编译的文件

``` shell
1E65C0FE239E1DCE0046BDDA /* Sources */ = {
    isa = PBXSourcesBuildPhase;
    buildActionMask = 2147483647;
    files = (
        1E65C10D239E1DCE0046BDDA /* ViewController.m in Sources */,
        1E65C107239E1DCE0046BDDA /* AppDelegate.m in Sources */,
        1E65C118239E1DCF0046BDDA /* main.m in Sources */,
        1E65C10A239E1DCE0046BDDA /* SceneDelegate.m in Sources */,
    );
    runOnlyForDeploymentPostprocessing = 0;
};
```

### PBXResourcesBuildPhase section

对应项目中 Build Phases -> Copy Bundle Resources 中的资源

``` shell
1E65C100239E1DCE0046BDDA /* Resources */ = {
    isa = PBXResourcesBuildPhase;
    buildActionMask = 2147483647;
    files = (
        1E65C115239E1DCF0046BDDA /* LaunchScreen.storyboard in Resources */,
        1E65C112239E1DCF0046BDDA /* Assets.xcassets in Resources */,
        1E65C110239E1DCE0046BDDA /* Main.storyboard in Resources */,
    );
    runOnlyForDeploymentPostprocessing = 0;
};
```

### XCBuildConfiguration section
定义了一组 build config，默认为 Debug 或 Release

``` shell
1E65C119239E1DCF0046BDDA /* Debug */ = {
    isa = XCBuildConfiguration;
    buildSettings = {
        ALWAYS_SEARCH_USER_PATHS = NO;
        CLANG_CXX_LANGUAGE_STANDARD = "gnu++14";
        CLANG_CXX_LIBRARY = "libc++";
    };
    name = Debug;
};
```

### XCConfigurationList section

XCBuildConfiguration 的组合，在 PBXProject 或 PBXTarget 中使用

``` shell
1E65C0FD239E1DCE0046BDDA /* Build configuration list for PBXProject "Test1" */ = {
    isa = XCConfigurationList;
    buildConfigurations = (
        1E65C119239E1DCF0046BDDA /* Debug */,
        1E65C11A239E1DCF0046BDDA /* Release */,
    );
    defaultConfigurationIsVisible = 0;
    defaultConfigurationName = Release;
};
```

### PBXBuildFile section

A PBXFileReference is a reference to the actual file. It's the object that backs up the files that appear in the left-hand project view.

A PBXBuildFile is a file in a target. It wraps a PBXFileReference and adds certain attributes like per-file compiler flags. If a file is added to a target, it will be listed in both sections. If a file is in more than one target, it will have more than one PBXBuildFile in the build files section.

包含了对每个文件编译的额外属性(Compiler Flags)，例如 -fno-objc-arc 和 -fobjc-arc

``` shell
1E65C10D239E1DCE0046BDDA /* ViewController.m in Sources */ = {isa = PBXBuildFile; fileRef = 1E65C10C239E1DCE0046BDDA /* ViewController.m */; settings = {COMPILER_FLAGS = -fno-objc-arc; }; };
1E65C110239E1DCE0046BDDA /* Main.storyboard in Resources */ = {isa = PBXBuildFile; fileRef = 1E65C10E239E1DCE0046BDDA /* Main.storyboard */; };
```

## cordova-node-xcode

### 介绍

npm地址：https://www.npmjs.com/package/xcode

cordova 项目中使用的工具库，用于操作 xpbproject 文件

基本使用 (node 运行) 
``` javascript
var xcode = require('xcode'),
    fs = require('fs'),
    projectPath = '../Test1.xcodeproj/project.pbxproj',
    myProj = xcode.project(projectPath)

// 解析
myProj.parseSync()

// ... 具体操作

// 修改完后保存
fs.writeFileSync(projectPath, myProj.writeSync());
```

上面初始化的 myProj 对象结构如下图所示：
![](/assets/img/xpb1.png)

xpbproject 中的内容都保存在 myProj.hash.project 对象中：

![](/assets/img/xpb2.png)

### 具体使用

#### 获取 project

``` javascript
let project = myProj.getFirstProject()['firstProject']

let targets = project.targets
let mainGroup = project.mainGroup
```

#### 根据 Target 名称获取 Target uuid

``` javascript
function getTargetUUIDWithName(targetName) {
    let targets = myProj.getFirstProject()['firstProject']['targets']

    for (var i=0; i < targets.length; i++) {
        var item = targets[i]

        if (item['comment'] == targetName) {
            return item['value']
        }
    }

    return null
}
```

#### 添加代码文件

``` javascript
let mainGroupUUID = myProj.getFirstProject()['firstProject']['mainGroup']
let mainGroupObj = myProj.getPBXGroupByKey(mainGroupUUID)

let firstGroupUUID = mainGroupObj['children'][0]['value']

// firstGroupUUID 是默认添加到了第一个group中，group 层级对应 xcode 项目中代码层级
myProj.addHeaderFile('ViewController1.h', {}, firstGroupUUID)
myProj.addSourceFile('ViewController1.m', {}, firstGroupUUID)
```

#### 获取某个 target 的 frameworks

``` javascript
let targetUUID = getTargetUUIDWithName('Test1')

let buildFramework = myProj.pbxFrameworksBuildPhaseObj(targetUUID)
```

#### 添加  framework

``` javascript
// system framework
myProj.addFramework('System/Library/Frameworks/NetworkExtension.framework')

// 添加第三方静态库
// myProj.addStaticLibrary(aPath)
```

#### 添加 App Extension

动态添加 vpn 的 appex

``` javascript
function test() {
    myProj.parseSync()

    addVpnAppExtension('../vpn/PacketTunnel.appex')

    fs.writeFileSync(projectPath, myProj.writeSync());
}

function addVpnAppExtension(path) {
    // add to PBXFileReference and PBXGroup
    let file = myProj.addFile(path, getFirstGroupUUID())
    file.uuid = myProj.generateUuid();
    // add to PBXBuildFile
    myProj.addToPbxBuildFileSection(file)

    let targetUUID = getTargetUUIDWithName('Test1')
    var copyFileSection = myProj.hash.project.objects['PBXCopyFilesBuildPhase']

    var buildPhase = myProj.buildPhase('Embed App Extensions', targetUUID);

    for (key in copyFileSection) {
        // select the proper buildPhase
        if (buildPhase && buildPhase == key) {
            sectionKey = key.split('_comment')[0];
            
            let appexExtensions = copyFileSection[sectionKey]
            appexExtensions['files'].push({
                value: `${file['uuid']}`,
                comment: `${file['uuid']}_comment`
            })
        }
    }
}
```

## Xcodeproj 命令

### 介绍

Xcodeproj 是 CocoaPods 中使用的库，用来查看、修改 project.pbxproj 文件，比 cordeva 更强大，但使用 ruby 编写，会导致脚本更复杂

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

命令行使用

``` shell

xcodeproj show 查看 project.pbxproj

把 project.pbxproj 输出到一个文件中
xcodeproj show > project.yaml
```

### 具体使用

#### 获取 project 信息

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

#### target 信息

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

#### 修改 bundle id 和 证书

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

#### build_setting 的设置

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

#### Linked Frameworks 中 Frameworks

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

#### Embed App Extensions 添加

``` ruby
embedAppExtension = target.copy_files_build_phases[0]
for tmp in embedAppExtension.files_references 
    puts tmp.name
    puts tmp.path
end

# 新增一个 appex 文件
# embedAppExtension.add_file_reference(project.new_file('PacketTunnel.appex'))
```

#### Target Dependencies

``` ruby
for tmp in main_target.dependencies
    # dependency 类型为 PBXTargetDependency 
    # https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXTargetDependency
    puts tmp.target
end
```

#### 其他
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