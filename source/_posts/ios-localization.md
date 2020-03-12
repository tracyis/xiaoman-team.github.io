---
title: iOS多语言开发探讨
date: 2020-03-12 10:29:39
tags: iOS
---
    作者: Hackice

## 国际化与本地化

>国际化是指在设计软件，将软件与特定语言及地区脱钩的过程。当软件被移植到不同的语言及地区时，软件本身不用做内部工程上的改变或修正。本地化则是指当移植软件时，加上与特定区域设置有关的信息和翻译文件的过程。
——《维基百科》

国际化意味着产品有适用于任何地方的“潜力”；本地化则是为了更适合于“特定”地方的使用。

## 项目中国际化方式
1. 静态字符串（.strings），通过调用Bundle的`localizedString(forKey: String, value: String, table: String)`方法实现国际化
2. 资源：xib/asset/info.plist
3. 语言包：xcloc

## New
- **iOS 13中用户可以针对每一个应用单独的设置语言选项**![](/images/ios_localization_01.jpg)
- **获取当前语言设置**

```
// 系统设置中的语言偏好列表
Locale.preferredLanguages
// 应用支持语言的偏好选项
Bundle.main.preferredLocalizations.first
// 指定范围语言中的偏好
Bundle.preferredLocalizations(from: [String]).first
```

- **Test Plan导出包含上下文的语言包信息**![](/images/ios_localization_02.jpg)


## 问题

1. 当前国际化依赖手动维护各个语种的字符串文件（.strings），从调用难易度、容错、可维护性，有没有更好的方案
2. Interface Builder中国际化的处理，比如文本和图片
3. app语言切换


### 参考：
>[Creating Great Localized Experiences with Xcode 11](https://developer.apple.com/videos/play/wwdc2019/403/) WWDC19
>
>[Clean iOS Localizable Files](https://buildingvts.com/clean-ios-localizable-files-8b910413b985) medium
>
>[How to keep your iOS localizable files clean](https://medium.com/stash-engineering/how-to-keep-your-ios-localizable-files-clean-swift-script-edition-a01bc649ef1d) medium
>
>[String Localization](https://www.objc.io/issues/9-strings/string-localization/) objc.io
