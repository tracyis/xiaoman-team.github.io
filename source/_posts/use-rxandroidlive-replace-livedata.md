---
title: 使用RxAndroidLive来实现Livedata的功能
date: 2019-12-16 15:25:34
tags: [android,lifecycle,livedata]
---

# 前言
本文不讲细节，只讲功能，因为细节上，很多前人的库都已经有完美的实现了。我们只是做了整合并实现我们需要的功能
AndroidX中的生命周期组件，Livedata，是个很好用的功能。但是在Rxjava中，很多生命周期组件，都只能解绑，不能进行延迟触发与缓存，以及重订阅

# LiveData的功能

## 延迟触发
什么是延迟触发？在Livedata中，有两个方法：

```
  protected void onActive() {

  }
  protected void onInactive() {

  }
```
这两个方法，一个是在生命周期onStart之后执行，一个是在生命周期onStop执行，所谓延迟触发是，我在onCreate的时候就已经对LiveData进行订阅，但实际触发请求，是在onStart之后调用onActive之后才进行的数据请求。则为延迟触发。这里的好处是，写在Fragment的onCreate数据订阅，在onStart之后才开始请求数据，这样保证数据到达界面时，界面可用，满足在onCreate时进行数据请求能保证数据到达界面时界面可用

## 缓存
什么是缓存，Livedata的经典功能，不管订阅数据在后台如何快速发送，当界面可用的时候，都像界面层推送最后一个缓存数据。

## 重订阅
什么是重订阅？比如，代码中，需要在onStart之后开始获取数据，但是当onInactive的时候关掉数据获取，然后当我界面在可用进入onStart时，再次像数据源订阅，解决界面在压栈之后还频繁更新数据。

## 主线程管理
在Livedata订阅的数据，接收者都能在主线程直接更新UI

# RxAndroidLive
根据目前需要的需求，要实现Livedata的这些功能遍历在Rxjava上的实现，为什么要这么实现，因为Livedata没有很好的错误处理，所以我们需要在RxJava上实现这些界面需要的，我们原本使用非白白的Live组件，但是这个组件只实线了缓存功能，以及仅仅实现了Observable组件，这对我们整个系统的逻辑不完善，所以我们改进并实现了重订阅以及延迟触发机制还有线程管理。解决Android整个生命周期问题。

## 引入

```
repositories {
    jcenter()
}

dependencies {
    implementation 'cn.xiaoman.library.android:rxandroidlive:1.0.0'
}
```

## 代码使用

```
Flowable.interval(1, TimeUnit.SECONDS)
   .compose(RxAndroidLive.<Long>bindLifecycle(this, true, AndroidSchedulers.mainThread()))
   .subscribe(new Subscriber<Long>() {
        @Override
        public void onSubscribe(Subscription s) {
            addContent("Flowable onSubscribe");
            final int requestNum = 10;
            s.request(requestNum);
            addContent("Flowable request " + requestNum);
        }

        @Override
        public void onNext(Long aLong) {
            addContent("Flowable onNext value " + aLong);
        }

        @Override
        public void onError(Throwable t) {
            addContent("Flowable onError");
        }

        @Override
        public void onComplete() {
            addContent("Flowable onComplete");
        }
   });
```
## 参数解析
1. 默认实现onActive感知，只有当onStart状态之后，才会向上订阅数据
2. 当autoRestart成立时，生命周期经历stop之后取消上层订阅，对界面层订阅不中断，重新进行livedata的onstart时，重新订阅数据
3. 当界面不可用时，默认缓存最后一次数据，当界面可用时抛出最后一次数据
4. autoRestart 参数仅仅在可持续订阅数据库可用（Observable，Flowable）
5. Scheduler 参数指定rxjava后面数据流的所在线程，一般自己指定为主线程（若不指定，线程可能会因为生命周期问题，进行线程飘逸，因为生命周期属于主线程触发，后台数据上抛可能处于子线程）

# 源码
项目已开源，[源码在这](https://github.com/xiaoman-team/RxAndroidLive)


# 感谢
AndroidX LiveData

[Android 生命周期架构组件与 RxJava 完美协作](https://listenzz.github.io/android-lifecyle-works-perfectly-with-rxjava.html)