## 时序图

http://plantuml.com/zh/sequence-diagram

@startuml

participant c as "禅道"
participant jm as "jenkins master"
participant js as "jenkins slave"
participant ftp as "ftp 服务器"

c -> jm: 点击：构建 
activate jm

jm -> js: 选择 salve ，执行脚本
deactivate jm
activate js

js -> c: 开始构建

js --> js: 从禅道下载资源、证书

js --> js: 替换图片资源，替换 KDMixedCloudConfig 文件

js --> js: 读取 ios.property , 替换参数，保存到 integration.plist 中

js --> js: 导入证书，检测 group 是否一致

js --> js: 替换 info.plist 中的参数

js --> js: 多语言处理，更新 InfoPlist.strings

js --> js: 更新 entitlements ，修改 group

js --> js: 更新 project.pbxproj ，修改 bundle id，provisioning file name

js --> js: 修改 Podfile ，执行 sh pod_install 

js --> js: xcode-select -switch 命令

js --> js: xcodebuild 构建

js --> js: 修改 enterprise.plist ，准备 archive

js --> js: xcodebuild -exportArchive 打包

deactivate js

js --> jm: 把 ipa 和 plist scp 到 master

js --> ftp: ipa、plist、dsym 上传到 ftp 服务器

js -> c: 构建完成

@enduml
