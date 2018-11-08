---
title: iOS 崩溃解决方案
date: 2018-05-08 10:32:08
tags: iOS
---

	行文环境
	- Xcode 9.3
	- Swfit 4.1
	- iOS 11
	- Bugly
	 
## 崩溃分析平台（Bugly）

崩溃平台大多数问题就是缺失dsyMs文件，只要找到crash log对应的dsyMs文件上传重新符号化（Re-symbolicate）即可

### 找到dsyMs文件的方法

#### 本地文件

`mdfind "com_apple_xcode_dsym_uuids == <UUID>"`
在bundle打包的电脑上使用上面的命令查找对应UUID的dsyMs文件

#### xcarchives

在bundle打包的电脑上也可以在Xcode Orgnizer里找到对应Archive

![1](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/Screen%20Shot%202018-05-07%20at%204.26.10%20PM.png)

![2](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/Screen%20Shot%202018-05-07%20at%204.32.08%20PM.png)

![3](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/Screen%20Shot%202018-05-07%20at%204.32.16%20PM.png)

#### 下载 Bitcode dSYMs

第一种方式是在Xcode Orgnizer

![](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/WeChatWorkScreenshot_88a7d1d7-e92a-471f-bd60-4e64f91c9977.png)

第二种方式是在iTC，已发布的bundle推荐使用这种方式

![](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/WeChatWorkScreenshot_c10ee963-8e32-4dd6-9317-851825499eb5.png)

## Crash log

崩溃分析平台无法定位的崩溃可以通过直接分析crash log文件

#### 获取设备的crash log

打开Xcode Device页面，并导出crash log

![](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/Screen%20Shot%202018-05-07%20at%204.51.47%20PM.png)

![](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/WeChatWorkScreenshot_b96dc51e-8b02-405a-acdb-f1306667e0b5.png)

![](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/Screen%20Shot%202018-05-07%20at%204.55.01%20PM.png)

#### dysMs文件

打开crash log，第一行就是对应bundle的UUID
`Incident Identifier: C2FB3056-8C2E-4141-800F-1E575858998D`

使用前述的方式找到dysMs文件

#### 符号化

使用Xcode自带的一个命令行工具来符号化crash log文件

![](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/Screen%20Shot%202018-05-07%20at%205.05.12%20PM.png)

`/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/`

在这个路径找到该工具，将crash log、symbolicatecrash和dyMs文件夹zip包拷贝到同一目录下

![](http://ios-ci.oss-cn-hangzhou.aliyuncs.com/blog/Screen%20Shot%202018-05-07%20at%205.07.39%20PM.png)

cd到该目录，运行如下命令
`export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"`

符号化
`./symbolicatecrash myCrash.crash > SymbolicatedM.crash`

done

## 更多

符号化之后的崩溃分析不属于本文范围

#### 扩展阅读

- [How to symbolicate crash log with Xcode 8?
](https://stackoverflow.com/a/43539663/386151)
- [All about Missing dSYMs](https://docs.fabric.io/apple/crashlytics/missing-dsyms.html)
- [Bugly iOS 符号表配置](https://bugly.qq.com/docs/user-guide/symbol-configuration-ios/?v=20180119105842)