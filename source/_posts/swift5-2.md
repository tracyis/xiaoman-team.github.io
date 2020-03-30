---
title: Swift 5.2 新特性
date: 2020-03-30 14:24:02
tags: iOS, Swift, Xcode
---

# Swift 5.2新特性

2020年3月25日，随着Xcode 11.4的开放下载，Swift 5.2也正式发布了。

*该版本专注于改善开发人员体验：*

* 提升编译器诊断（错误和警告）和代码补全效率
* 提高Debug的可靠性
* 改进了Swift Package Manager中依赖的处理
* LSP和SwiftSyntax的工具改进
* ...

## 语言更新
Swift 5.2实现了下列提案：

* [[SE-0249]](https://github.com/apple/swift-evolution/blob/master/proposals/0249-key-path-literal-function-expressions.md)通过Key Path表达式声明函数
* [[SE-0253]](https://github.com/apple/swift-evolution/blob/master/proposals/0253-callable.md)自定义可调用标称类型

以及其他语言特性：

* 下标支持默认参数
* Lazy filter顺序调整

## 改进的编译器诊断

Swift 5.2的新编译器诊断引擎解决了Swift 5.1代码错误提示中潜在的混乱消息，现在可以更好地指出需要修复的确切代码段。 

例如：

```Swift
enum E { case one, two }

func check(e: E) {
  if e != .three {
    print("okay")
  }
}
```
在Swift 5.2之前，我们会看到如下错误提示：

```
error: binary operator '!=' cannot be applied to operands of type 'E' and '_'
  if e != .three {
     ~ ^  ~~~~~~
```
在Swift 5.2中：

```
error: type 'E' has no member 'three'
  if e != .three {
          ~^~~~~
```

## 改进的代码补全

通过消除不必要的类型检查来更快地完成。对于大型文件，与Xcode 11.3.1相比，它可以将代码完成速度提高1.2倍至1.6倍，具体取决于完成位置。

为不完整的字典文字和不完整的三元表达式提供隐式成员的名称：

![](/images/code-complete-1.png)

类型出现在结果中时更易于阅读。some View尽可能使用不透明的结果类型（例如），并保留类型别名，例如在Swift 5.1.3（Xcode 11.3.1）中：

![](/images/code-complete-2.png)

在Swift 5.2（Xcode 11.4）中，现在显示为：

![](/images/code-complete-3.png)

##编译速度提升

![](/images/compilation-modes.png)

Swift 5.2编译器利用新的集中式逻辑来更有效地处理编译（尤其是增量编译）中的解析声明和相互引用，避免不必要的开销，从而提高性能和编译速度。

## SPM、LLDB改进以及SourceKit-LSP协议更新

Swift 5.2中的SPM使用一种新策略来解决依赖性以提高错误消息的质量和复杂依赖的性能。

# Swift 5.3呢？

质量和性能增强，并增加对Windows、Linux平台的支持

# Xcode 11.4 新特性

## Universal Purchase
Xcode 11.4允许开发者使用同一个Bundle ID分发macOS版本应用，以实现“通用购买”，即用户只需购买一次即可在iOS，iPadOS，macOS，watchOS和tvOS上使用应用或内购服务。

## 模拟器支持iPadOS的指针捕获

UIButton、手势添加代表“指针按钮”的Point属性

## 模拟器支持拖拽APNs文件模拟远程推送通知

也可以使用命令行：
`xcrun simctl push <device> com.example.my-app ExamplePush.apns`

>参考：
> 
[swift.org - Swift 5.2 Released!](https://swift.org/blog/swift-5-2-released/)
>
>[Apple Developer - Xcode 11.4 Release Notes](https://developer.apple.com/documentation/xcode_release_notes/xcode_11_4_release_notes)
>
>[Apple Developer - Offering Universal Purchase](https://developer.apple.com/support/universal-purchase/)
>
>[GitHub - What’s new in Swift 5.2?](https://github.com/twostraws/whats-new-in-swift-5-2)
>
>[swift by sundell - Exploring Swift 5.2’s new functional features](https://www.swiftbysundell.com/articles/exploring-swift-5-2s-new-functional-features/)
