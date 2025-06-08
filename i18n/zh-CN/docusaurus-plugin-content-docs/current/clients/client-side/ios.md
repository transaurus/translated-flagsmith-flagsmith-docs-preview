---
title: Flagsmith iOS SDK
sidebar_label: iOS / Swift
description: Manage your Feature Flags and Remote Config in your iOS applications.
slug: /clients/ios
---

该库可用于iOS和Mac应用程序。客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-ios-client)。

## 安装方式

### CocoaPods

[CocoaPods](https://cocoapods.org)是Cocoa项目的依赖管理工具。访问其官网查看使用说明。要通过CocoaPods将Flagsmith集成到Xcode项目，请在`Podfile`中声明：

```ruby
pod 'FlagsmithClient', '~> 1.0'
```

### Swift包管理器

Swift包管理器是用于自动化分发Swift代码的工具，已集成到Swift编译器中。您可以通过在`Package.swift`文件中添加描述来安装Flagsmith：

```swift
dependencies: [
    .package(url: "https://github.com/Flagsmith/flagsmith-ios-client.git", from: "1.1.1"),
]
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目的单个环境（如开发或生产环境）进行初始化。客户端环境密钥可在环境设置页面获取。

![图片](/img/api-key.png)

### 初始化

在应用程序委托文件（通常为_AppDelegate.swift_）中添加：

```swift
import FlagsmithClient
```

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

Flagsmith.shared.apiKey = "<YOUR_API_KEY>"
// The rest of your launch method code
}
```

完成设置后即可从项目中获取功能开关。例如列出并打印所有开关：

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

注意可使用以下方式获取值：

- `flag.value?.stringValue`
- `flag.value?.intValue`
- `flag.value?.floatValue`

根据所需类型选择对应方法。

通过名称获取布尔型功能开关值：

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

这些方法还可通过**forIdentity**参数为特定用户身份获取值，详见[身份管理](https://docs.flagsmith.com/managing-identities/)。

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

### Swift并发支持

当使用Swift 5.5.2及以上版本（Xcode 13.2）时，Flagsmith API将提供`async`版本。这些API通过Swift的通用方法[`withCheckedThrowingContinuation(function:_:)`](https://developer.apple.com/documentation/swift/3814989-withcheckedthrowingcontinuation)实现，对基于闭包的语法进行封装。`async`/`await`语法可提供更清晰的代码执行流，例如：

```swift
/// (Example) Setup the app based on the available feature flags.
func determineAppConfiguration() async throws {
    let flagsmith = Flagsmith.shared

    if try await flagsmith.hasFeatureFlag(withID: "ab_test_enabled") {
        if let theme = try await flagsmith.getFeatureValue(withID: "app_theme") {
            setTheme(theme)
        } else {
            let flags = try await flagsmith.getFeatureFlags()
                processFlags(flags)
        }
    } else {
        let trait = Trait(key: "selected_tint_color", value: "orange")
        let identity = "4DDBFBCA-3B6E-4C59-B107-954F84FD7F6D"
        try await flagsmith.setTrait(trait, forIdentity: identity)
    }
}
```

## 覆盖默认配置

客户端默认使用标准配置，可按以下方式覆盖：

仅覆盖默认API地址：

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

Flagsmith.shared.apiKey = "<YOUR_API_KEY>"
Flagsmith.shared.baseURL = "https://<your-self-hosted-api>/api/v1/"
// The rest of your launch method code
}
```