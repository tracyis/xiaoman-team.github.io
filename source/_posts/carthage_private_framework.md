---
title: 如何使用 Carthage 集成私有 framework
date: 2019-01-03 17:00:00
tags: [iOS,Carthage]
---

	行文环境
	- Xcode 10.1
	- Carthage 0.31.2
	 
## Carthage

### 简介
Carthage 是 swift 开发的代码依赖工具，与Cocoapods直接引入第三方源代码并修改 Xcode workspace 不同，Carthage 是将第三方库预先 build 好的 framework 引入工程。这样更适合大型项目，预先编译好的 framework 不需要重新编译。

### 快速使用
1. 安装 `brew install carthage`
2. 在工程目录下创建 `Cartfile`
3. 在 `Cartfile` 里添加依赖库，比如 `github "Alamofire/Alamofire" ~> 4.7.2`
4. 运行 `carthage update`
5. 会生成 `Cartfile.resolved` 和 Carthage 文件夹 包括 Build 和 Checkout
6. 添加 framework 到工程，在 Build Phases 里新建 New Run Script Phase，输入：
	`/usr/local/bin/carthage copy-frameworks`
	
	在 `Input Files` 项里填入 
	
	`$(SRCROOT)/Carthage/Build/iOS/Alamofire.framework`
	
	在 `Output Files` 项里填入 
	
	`$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/Alamofire.framework`

### 常用命令
`carthage update` 更新并重新编译 framework

`carthage bootstrap` 类似 update，一般用于新 checkout 的仓库第一次运行 carthage

`--platform iOS` build 平台参数

`--cache-builds` build 时使用缓存

`carthage build --no-skip-current` build private framework

### Cartfile
引用github库 `github "Alamofire/Alamofire" ~> 4.7.2`

指定具体tag `github "malcommac/SwiftDate" == 4.5.1` 

指定分支 `github "XiaomaniOS/RichEditorView" "xiaoman"`

私有仓库 `git "ssh://git@git.xiaoman.co:7999/im/xmprotobuf.git"`

本地仓库 `git "file:///Users/neilpa/code/project" "branch"`

### 缺点
* 第一次 build 时间很长（喝杯咖啡先）
* CI 运行 `carthage update` 费时，使用 `--cache-builds` 加快效率
* 没有 Cocoapods 的各种其他功能

## 创建私有 framework

### 新建项目

新建 **Cocoa Touch Framework** 的项目

将 framework 的 scheme 设置成 shared，这里有个坑，新创建的项目 Xcode 不会生成 `.xcscheme` 文件，carthage 需要找这个文件来选择哪些 scheme 需要 build，所以需要修改一下 scheme 让 git 里生成`.xcscheme` 文件

设置 `General ` -> `Deployment Target`

### 添加代码

将源代码加入工程，确保需要暴露的类等是 public 的，因为默认是 internal，外部无法识别

Objective-C代码需要设置header为Public
![](https://static1.squarespace.com/static/53699711e4b052b073e44f1f/t/59f26cb727ef2db556c3648f/1509059778262/target-membership.png)

私有 framework 如果有依赖库可以使用 Carthage 添加，主工程使用私有 framework 时也会 build 出依赖的 framework

### 编译

`carthage build --no-skip-current`

如果有错参考 [Resolve build failures](https://github.com/Carthage/Carthage#resolve-build-failures)

如果提示找不到 scheme，检查上面提到的 `.xcscheme` 文件有没有提交到 git

成功后打 git tag，Carthage 用 tag 来定位代码

### 使用

用上面提到的方法加入到主工程即可，注意依赖库也需要添加

## Read More
### Carthage
* [Carthage/Carthage](https://github.com/Carthage/Carthage#resolve-build-failures)
* [Carthage or CocoaPods: That is the question](http://shashikantjagtap.net/carthage-cocoapods-question/)
* [Creating a Private Framework With Carthage](https://samwize.com/2018/06/27/creating-a-private-framework-with-carthage/)
* [Local Git Repos?](https://github.com/Carthage/Carthage/issues/398)
* [Nested Frameworks?](https://github.com/Carthage/Carthage/issues/844)
* [bootstrap vs update?](https://github.com/Carthage/Carthage/issues/848)
* [Github API rate limit reached when building iOS project with Carthage](https://github.com/travis-ci/travis-ci/issues/4195)
* [Five Steps to Migrate iOS Project from CocoaPods to Carthage](http://shashikantjagtap.net/five-steps-migrate-ios-project-cocoapods-carthage/)

### 其他
* [Dynamic Versus Static Framework in iOS](https://www.ca.com/en/blog-developers/dynamic-versus-static-framework-in-ios.html)
* [Creating a Framework for iOS](https://www.raywenderlich.com/5109-creating-a-framework-for-ios)
* [Getting Started with Reusable Frameworks for iOS Development](https://medium.com/flawless-app-stories/getting-started-with-reusable-frameworks-for-ios-development-f00d74827d11)
* [JohnSundell/SwiftPlate](https://github.com/JohnSundell/SwiftPlate)
* [Making a CocoaPod](https://guides.cocoapods.org/making/making-a-cocoapod.html)
* [Improving Your Build Time in Xcode 10](https://patrickbalestra.com/blog/2018/08/27/improving-your-build-time-in-xcode-10.html)
* [How the Zalando iOS App Abandoned CocoaPods and Reduced Build Time](https://jobs.zalando.com/tech/blog/how-the-zalando-ios-app-abandoned-cocoapods-and-reduced-build-time/?gh_src=4n3gxh1)
* [Building Objective-C Frameworks](http://www.vsanthanam.com/writing/2017/10/26/building-objective-c-frameworks)
