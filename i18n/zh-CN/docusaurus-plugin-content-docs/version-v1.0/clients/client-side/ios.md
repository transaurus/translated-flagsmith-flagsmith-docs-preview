---
title: Flagsmith iOS/Swift SDK
sidebar_label: iOS
description: Manage your Feature Flags and Remote Config in your iOS applications.
slug: /clients/ios
---

该库可用于iOS和Mac应用程序。客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-ios-client)。

## 安装

### CocoaPods

[CocoaPods](https://cocoapods.org)是Cocoa项目的依赖管理工具。访问其官网查看使用说明。要通过CocoaPods将Flagsmith集成到Xcode项目中，请在`Podfile`中声明：

```ruby
pod 'FlagsmithClient', '~> 1.0'
```

### Swift Package Manager

Swift Package Manager是用于自动化分发Swift代码的工具，已集成到Swift编译器中。您可以通过在`Package.swift`文件中添加描述来安装Flagsmith：

```swift
dependencies: [
    .package(url: "https://github.com/Flagsmith/flagsmith-ios-client.git", from: "1.1.1"),
]
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目的单个环境进行初始化，例如开发环境或生产环境。您可以在环境设置页面找到环境密钥。

![图片](/img/api-key.png)

### 初始化

在应用程序委托中（通常是_AppDelegate.swift_）添加：

```swift
import FlagsmithClient
```

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

Flagsmith.shared.apiKey = "<YOUR_API_KEY>"
// The rest of your launch method code
}
```

现在您已准备好从项目中获取功能开关。例如列出并打印所有开关：

```swift
Flagsmith.shared.getFeatureFlags() { (result) in
    switch result {
    case .success(let flags):
        for flag in flags {
            let name = flag.feature.name
            let value = flag.value?.stringValue
            let enabled = flag.enabled
            print(name, "= enabled:", enabled, "value:", value ?? "nil")
        }
    case .failure(let error):
        print(error)
    }
}
```

注意您可以使用：

- `flag.value?.stringValue`
- `flag.value?.intValue`
- `flag.value?.floatValue`

根据您需要的类型进行选择。

通过名称获取功能开关的布尔值：

```swift
Flagsmith.shared.hasFeatureFlag(withID: "test_feature1", forIdentity: nil) { (result) in
    print(result)
}
```

通过名称获取配置值：

```swift
Flagsmith.shared.getFeatureValue(withID: "test_feature2", forIdentity: nil) { (result) in
    switch result {
    case .success(let value):
        print(value ?? "nil")
    case .failure(let error):
        print(error)
    }
}
```

这些方法还可通过**forIdentity**参数为特定用户身份获取值。详见[身份管理](https://docs.flagsmith.com/managing-identities/)。

获取特定身份的特征值（参见[特征管理](https://docs.flagsmith.com/managing-identities/#identity-traits)）：

```swift
Flagsmith.shared.getTraits(forIdentity: "test_user@test.com") {(result) in
    switch result {
    case .success(let traits):
        for trait in traits {
            let name = trait.key
            let value = trait.value
            print(name, "=", value)
        }
    case .failure(let error):
        print(error)
    }
}
```

## 覆盖默认配置

默认情况下客户端使用标准配置。您可按如下方式覆盖配置：

仅覆盖默认API地址：

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

Flagsmith.shared.apiKey = "<YOUR_API_KEY>"
Flagsmith.shared.baseURL = "https://<your-self-hosted-api>/api/v1/"
// The rest of your launch method code
}
```