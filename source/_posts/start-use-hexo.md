---
title: 初始化使用hexo
date: 2018-12-03 16:10:38
tags:
---
# 前言
团队博客的驱型基本做好了，接下来是要写一篇交大家使用的文章，这样才能让大家安心写些文档不是？
目前使用的博客系统，是基于nodejs的hexo静态博客生成组件。
好处是什么呢？是我们直接用github来托管就好了，不用自己管理空间哈。
接下来我们说说怎么使用我们的博客系统

# 平台支持初始化
公司用的比较多的是mac，下面就基于苹果环境来做教程啦，你们相应的做修改哈。
先安装nodejs环境和博客平台环境。

## 安装nodejs
这个嘛，到官网下载就好了。https://nodejs.org/，作者目前使用的是11.3.0哈，安装好之后，就会有npm指令了。

## 安装hexo
```npm install -g hexo-cli```

这样，博客支持程序就安装好了。

# 配置网站本地环境

## 下载网站源码
```git clone git@github.com:xiaoman-team/xiaoman-team.github.io.git```

## 获取网站源码分支
为啥要获取，因为静态网站使用的是master分支，这个分支是试用hexo发布的，平时我们管理的文章，在source分支，为啥用这个分支？嗯，其实你也可以用其他分支不是？自己换就好了啦。

```
cd xiaoman-team.github.io
git checkout -b source origin/source
```

## 初始化hexo程序

```npm install hexo --save```

到目前为止，，你就搞定了本地环境配置拉

# 文章编写
接下来，是不是改写文章了呢？好吧，我们用hexo生成一片新文章好不好？

## 新文章编写
比如这篇文章的指令是
```hexo new start-use-hexo```
当然，start-use-hexo就是文章的链接名啦，文件会生成在 _post 目录下，是 markdown 格式的。

## 进入文章编写
怎么写？？去_post目录下去写你的文章啦，，markdown怎么用？这不在本文讨论范围内哦，你自己去试试？

## 预览文章

使用一下命令生成静态文件来预览文章啦
```hexo g```

使用一下命令来打开预览服务器
```hexo s```

他会提示你本地打开路径，你自己打开就能预览你写好的文章啦。
如果有时候没生效，试试
`hexo clean`
在
`hexo g`

## 发布文章
也是很简单
```
hexo clean
hexo g
hexo d
```
请你在预览好之后，在执行 d 的命令哈。
该命令执行完成之后，会推送到master分支上，然后我们的网站 
 https://xiaoman-team.github.io.git 就会更新了。
 
 
# 保存文章源码
最后，最重要的，当然是把你的文章源码保存下来了啦。
```
git add source/_posts/start-use-hexo.md
git commit -m 'add start use hexo post'
git push
```

# 结语
好了，目前源码也已经保存了，文章也发布了，教程就是这么简单啦。有啥不懂的，可以提出来我们在探讨一下啦。