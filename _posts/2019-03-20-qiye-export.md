---
layout: post
title:  "iOS企业包相关知识"
categories: blog
---

* 目录
{:toc}

## plist文件

注意点：

- url 是 ipa的下载地址
- bundle-identifier、title、icon 是app的一些必要数据
- 该plist文件上传到服务器，必须是 https下载

最后安装包的链接：

```
itms-services:///?action=download-manifest&url=https://xxx/test.plist
```

在safari中打开链接即可下载。

```
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