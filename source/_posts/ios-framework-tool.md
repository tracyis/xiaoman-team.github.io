---
title: 组件开发工具总结
date: 2020-03-12 10:47:18
tags: iOS
---

    作者: Bobo
    - Cocoapods 1.9.0
    - Carthage 0.34.0
    - Xcode 11.3.1
    
## 选择Carthage的理由

1. 去中心化，预编译特性
2. 更新兼容了Swift 5编译
3. 尝试同步开发的另一种选择

## 如何使用Carthage开发组件

1. 安装指引 [https://github.com/Carthage/Carthage#installing-carthage](https://github.com/Carthage/Carthage#installing-carthage)
2. framework 链接问题，因为有的target的General里没有Link选项，所以统一使用BuildPhase的Link Binary with Libraries
3. 私有库包含其他库的是否需要copy-frameworks script，不需要，因为最终链接的地方是主项目，私有库copy会导致嵌套引入
4. 同步开发，Carthage的file模式并不能很好的支持本地同步开发；[https://www.mokacoding.com/blog/carthage-no-build/](https://www.mokacoding.com/blog/carthage-no-build/) 此文章提出的“弱使用”加上submodule的引入方式根本上是使用Xcode workspace集成开发，而这种方式有不少缺点，比如名字空间冲突，build setting重复设置等等

## Carthage的问题

1. 慢，可用Terminal proxy解决
2. 版本冲突，库没有版本依赖的声明，最终在主项目使用一个版本的库，会出现代码不兼容的情况；和pod引入的同一库也存在不同版本并存的问题
3. 动态link，有的库有动态依赖的问题，编译时不能发现错误，但实际缺少库运行app会报“dyld __abort_with_payload”问题

## Cocoapods常用命令

1. 创建组件，`pod lib create`
2. 验证，`pod spec lint --sources='xxx' --allow-warnings`
3. 上传，`pod repo push OKSpecs SPECS_NAME.podspec --sources='xxx' --allow-warnings`

## SwiftPM

### 优点
1. 原生Xcode支持，没有类似Podfile，Package.resolved文件则对应lockfile
2. Package.swift没有version字段，所以私有库更新并不需要管理version/tag，也没有spec仓库管理
3. 测试方便

### 缺点
1. 不支持oc(貌似已经支持)，不支持resource
2. 慢，xcode fetch/install比pod install慢很多，包括更新，也无法单独更新，无法手动控制安装
3. 源文件相比Pods/文件夹，存在derived data，一旦删掉，需要重新fetch

### 结论
不适合复杂混编项目，适合小型项目，适合编写纯逻辑的swift库
 PS: [https://forums.swift.org/t/how-to-add-local-swift-package-as-dependency/26457/7](https://forums.swift.org/t/how-to-add-local-swift-package-as-dependency/26457/7)

## 组件的版本(tag)管理

1. 主项目使用branch模式，拉取最新代码，也可以用于feature分支开发
2. 什么时候需要更新tag，组件有互相依赖时，比如OKWorkbench依赖OKCore，这时候OKCore需要更新的话就需要升级tag
3. 组件的开发都应该优先在主项目切换为path模式做本地开发
4. 什么时候应该上传tag的更新，一个功能相关的组件全部开发完毕时，这样能保证不会过于频繁的升级组件版本

## 组件设计
1. 现阶段组件都是为主项目服务
2. 组件应尽量具有单一性，可单独测试性，应尽量避免为了使用组件的一个小部分引入整个组件的情况出现
3. 应避免组件互相之间嵌套依赖，比如OKNetwork依赖OKCore，OKWorkbench依赖OKNetwork和OKCore，此时只需要保持OKWorkbench对OKNetwork依赖即可，避免多版本冲突，第三方库同理
4. 组件内的命名应保留名字空间，单独可阅读，避免使用过长的名字空间取名，比如OKCore.A.B.C...
