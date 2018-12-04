---
title: snappy for android
date: 2018-12-04 14:01:10
tags: [android,snappy]
---

# 前言
移动端要做离线版的邮件缓存，我们需要同步客户的大量邮件数据到本地，因为数据量大，不在使用http已经json进行邮件协议传输，改用proto并配合snappy来做数据传输。

# Snappy
snappy是什么？
好吧，我也不知道，公司要用到了，我就引用一下wikipedia的内容给各位看官看下？

>Snappy（以前称Zippy）是Google基于LZ77的思路用C++语言编写的快速数据压缩与解压程序库，并在2011年开源。它的目标并非最大压缩率或与其他压缩程序库的兼容性，而是非常高的速度和合理的压缩率。使用一个运行在64位模式下的酷睿i7处理器的单个核心，压缩速度250 MB/s，解压速度500 MB/s。压缩率比gzip低20-100%。

>Snappy广泛应用在Google的项目，例如BigTable、MapReduce和Google内部RPC系统的压缩数据。它可在开源项目中使用，例如Cassandra、Hadoop、LevelDB、MongoDB、RocksDB和Lucene。[4]解压缩时会检测压缩流中是否存在错误。Snappy不使用内联汇编并且可移植。

嗯，看了下介绍，知道为啥要用它了吧？为了非常高的压缩速度和合理的压缩率。

# 相关项目

嗯，我们移动端做Android项目，肯定优先找java相关的snappy实现。
我们找到了这样的一个项目。

https://github.com/xerial/snappy-java

这个项目也是使用java代码，然后来调用so库里面的c方法来进行代码压缩和解压。但是，这个打包方式，并不适合Android项目来调用，为了解决这个问题，所以有了这文章的主角

# Snappy-Android

嗯，我们修改了 snappy-java的源码路径，并改造成 android 项目的编译方式，用以支持 android 使用 snappy 来做压缩解压。

因使用的是相同的 so 动态链接库方式，使用了最新的 NDK 来编译的相应平台的动态链接库。

## 支持平台
目前使用的 NDK 编译出来的动态链接库包含 armeabi-v7a，arm64-v8a，x86，x86_64 四大平台，
其余平台 armeabi, mips, mips64 因为新版 NDK 编译取消支持，所以编译成品中不在支持，如果有需要，可以自己下载源码，使用旧版 ndk 进行编译打包。

##  使用方式
这里不在重新写一遍啦，详见开源项目主页
https://github.com/xiaoman-team/snappy-android

# 感谢
再次感谢 [snappy](https://github.com/google/snappy) [snappy-java](https://github.com/xerial/snappy-java) 给我们做出这么好的产品用于商业使用，我们只是做了一下 android 的打包适配。

