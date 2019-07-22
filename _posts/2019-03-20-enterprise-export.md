---
layout: post
title:  "iOS企业包相关知识"
categories: blog
---

* 目录
{:toc}

## 下载地址

1. 把 plist 文件上传到服务器(必须是https)
2. 下载的地址，在 手机的 safari 中打开 （可以先把url生成二维码）
```
itms-services:///?action=download-manifest&url=https://xxx/test.plist
```
3. 在safari中打开链接即可下载

## plist文件

注意点：

- url 是 ipa的下载地址
- bundle-identifier、title、icon 是app的一些基础数据，是必须的
- 该plist文件上传到服务器，必须是 https下载

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>md5</key>
					<string>a04fe3a12eecd6520f5273da8ea7624a</string>
					<key>url</key>
					<string>https://staic.aliyun.com/xxx_ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>needs-shine</key>
					<false/>
					<key>url</key>
					<string>https://staic.aliyun.com/xxx/icon@2x.png</string>
				</dict>
			</array>

			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>com.xxx.www</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>xxx</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```

## 构建企业包注意事项

```
// 构建
/usr/bin/xcodebuild -scheme target -workspace target.xcworkspace -configuration Release archive -archivePath ${archivePath} DEVELOPMENT_TEAM=${teamId}

// build 完后，会生成 archive 文件，然后 根据 archive 文件导出 ipa 包
/usr/bin/xcodebuild -UseModernBuildSystem=NO -exportArchive -archivePath ${WORKSPACE}/build/Release-iphoneos/target.xcarchive -exportPath ${WORKSPACE}/../ipa -exportOptionsPlist ${WORKSPACE}/../ipa/ExportOptions.plist
```

在导出 ipa 的过程中，需要 ExportOptions.plist 文件，来指定导出 ipa 的一些规则：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>compileBitcode</key>
	<false/>
	<key>method</key>
	<!-- 企业包 如果是 appstore ，则是 appstore -->
	<string>enterprise</string>
	<key>provisioningProfiles</key>
	<dict>
		<key>com.xx.www</key>
		<string>xx</string>
		<key>com.xx.www.CallExtension</key>
		<string>xxcallextension</string>
		<key>com.xx.www.extension</key>
		<string>xxextension</string>
		<key>com.xx.www.widget</key>
		<string>xxwidget</string>
	</dict>
	<key>signingCertificate</key>
	<string>iPhone Distribution</string>
	<key>signingStyle</key>
	<string>manual</string>
	<key>stripSwiftSymbols</key>
	<true/>
	<key>teamID</key>
	<string>254KGT3Y5Z</string>
	<key>thinning</key>
	<string>&lt;none&gt;</string>
</dict>
</plist>
```

其中要注意 provisioningProfiles 中的设置：

key 为 bundle id ，value 为对应的 provisioningProfile 的名称。

provisioningProfile 从下面截图中找：

![](/assets/img/provisioningfile-name.jpeg)



