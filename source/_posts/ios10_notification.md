---
title: 阿里云SDK实现iOS10推送通知
date: 2018-11-08 10:32:08
tags: iOS
---

	行文环境
	- Xcode 10.1
	- Swfit 4.1
	- iOS 12
	- 阿里云
	 
## 证书设置
[iOS推送证书设置](https://help.aliyun.com/document_detail/30071.html?spm=a2c4g.11186623.6.629.3e6e614cfnQxzL)

证书配置分为开发环境和生产环境，需要与业务服务器的开发环境（如dev/test）和正式环境区分开。例如给测试分发的Ad Hoc包使用生产环境的证书，但业务是test环境。

### Ad Hoc包为什么收不到通知
因为Ad Hoc使用生产环境证书，设备的deviceToken在不同证书环境是不同的，但测试包的业务是test环境，后台会在test环境使用开发的apns设置，导致deviceToken无法匹配到对应的设备。

## payload变化
老版本payload，只有body
```
{
    "aps": {
        "alert": {
        "your notification body",
        },
        "badge": 1,
        "sound": "default",
    },
    "key1":"value1",
    "key2":"value2"
}
```
新版payload，新增title、subtitle、body、category等字段
```
{
    "aps": {
        "alert": {
            "title": "title",
            "subtitle": "subtitle",
            "body": "body"
        },
        "badge": 1,
        "sound": "default",
        "category": "test_category",
        "mutable-content": 1
    },
    "key1":"value1",
    "key2":"value2"
}
```
业务逻辑字段方式没有变化，直接加在json里

## 通知实现

### 初始化SDK
使用配置文件`AliyunEmasServices-Info.plist`直接调用`autoInit`
```swift
func setup(_ application: UIApplication, launchOptions: [UIApplicationLaunchOptionsKey: Any]?) {
    CloudPushSDK.autoInit { result in
        guard let result = result else {
            log.error("Push SDK init failed, error: result is nil!")
            return
        }
        if result.success {
            log.debug("Push SDK init success, deviceId: \(String(describing: CloudPushSDK.getDeviceId()))")
        } else {
            log.error("Push SDK init failed, error: \(String(describing: result.error))")
        }
    }

    //...
    // 点击通知将App从关闭状态启动时，将通知打开回执上报
    CloudPushSDK.sendNotificationAck(launchOptions)
}
```

### 请求通知权限并注册远程通知
第一次安装会弹出请求通知的alert
```swift
func registerAPNS(application: UIApplication) {
    let center = UNUserNotificationCenter.current()
    center.delegate = self
    var options: UNAuthorizationOptions = [.alert, .sound, .badge]
    if #available(iOS 12.0, *) {
        options = [.alert, .sound, .badge, .providesAppNotificationSettings]
    }
    center.requestAuthorization(options: options) { (granted, error) in
        if granted {
            log.debug("User authored notification.")
            DispatchQueue.main.async {
                application.registerForRemoteNotifications()
            }
        } else {
            log.debug("User denied notification.")
        }
        if let error = error {
            log.error(error)
        }
    }
}
```

### 注册设备并上报deviceToken
```swift
func registerDevice(_ deviceToken: Data) {
    CloudPushSDK.registerDevice(deviceToken) { result in
        guard let result = result else {
            log.error("Register deviceToken failed, error:: result is nil!")
            return
        }
        if result.success {
            log.debug("Register deviceToken success, deviceToken: \(String(describing: CloudPushSDK.getApnsDeviceToken()))")
        } else {
            log.debug("Register deviceToken failed, error: \(String(describing: result.error))")
        }
    }
}

public func application(_ application: UIApplication,
                        didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    registerDevice(deviceToken)
}

// 注册失败
public func application(_ application: UIApplication,
                        didFailToRegisterForRemoteNotificationsWithError error: Error) {
    log.error("did Fail To Register For Remote Notifications With Error : \(error)")
}
```

### 注册通知类别
通知类别（category）用于给通知分类，可添加按钮或自定义UI
```swift
func createCustomNotificationCategory() {
    let action = UNNotificationAction(identifier: "actionID", title: "buttonTitle", options: [])
    let category = UNNotificationCategory(identifier: "CategoryID", actions: [action],
                                          intentIdentifiers: [],
                                          options: .customDismissAction)
    UNUserNotificationCenter.current().setNotificationCategories([category])
}
```

### UNUserNotificationCenterDelegate
`UNUserNotificationCenterDelegate`代替了`UIAppDelegate`的旧通知接收方法`didReceiveRemoteNotification`将接收通知的情况分为app开启时（foreground）和app不在前台时（background）

```swift
// app打开时调用
public func userNotificationCenter(_ center: UNUserNotificationCenter,
                                   willPresent notification: UNNotification,
                                   withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    // 业务逻辑
    handleNotification(notification)

    // 通知不弹出
    //completionHandler([])

    // 在app内弹通知
    completionHandler([.badge, .alert, .sound])
}

// app未打开时调用
public func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    let userAction = response.actionIdentifier
    if userAction == UNNotificationDefaultActionIdentifier { 
        // 点击通知栏本身
        log.debug("User opened the notification.")

        handleNotification(response.notification)

    } else if userAction == UNNotificationDismissActionIdentifier {
        // 通知dismiss，category创建时传入UNNotificationCategoryOptionCustomDismissAction才可以触发
        log.debug("User dismissed the notification.")
    } else if userAction == "actionID" {
        // 自定义按钮逻辑
    }

    completionHandler()
}

// iOS12新功能，在app系统通知设置里点击按钮跳到app内通知设置的回调
public func userNotificationCenter(_ center: UNUserNotificationCenter, openSettingsFor notification: UNNotification?) {
    log.debug("Open notification in-app setting.")
}
```

#### 扩展阅读

- [UserNotifications Document](https://developer.apple.com/documentation/usernotifications)
- [阿里云iOS SDK手册](https://help.aliyun.com/document_detail/30071.html)
- [iOS10通知适配](https://help.aliyun.com/document_detail/44666.html)
- [Ad Hoc App如何进行生产环境推送通知测试](https://help.aliyun.com/knowledge_detail/54107.html)