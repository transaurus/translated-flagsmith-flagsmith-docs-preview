---
title: Flagsmith iOS SDK
sidebar_label: iOS / Swift
description: Manage your Feature Flags and Remote Config in your iOS applications.
slug: /clients/ios
---

This library can be used with iOS and Mac applications. The source code for the client is available on
[Github](https://github.com/flagsmith/flagsmith-ios-client).

## Installation

### CocoaPods

[CocoaPods](https://cocoapods.org) is a dependency manager for Cocoa projects. For usage and installation instructions,
visit their website. To integrate Flagsmith into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
pod 'FlagsmithClient', '~> 1.0'
```

### Swift Package Manager

The Swift Package Manager is a tool for automating the distribution of Swift code and is integrated into the swift
compiler. You can use it to install Flagsmith by adding the description to your `Package.swift` file:

```swift
dependencies: [
    .package(url: "https://github.com/Flagsmith/flagsmith-ios-client.git", from: "1.1.1"),
]
```

## Basic Usage

The SDK is initialised against a single environment within a project on [https://flagsmith.com](https://flagsmith.com),
for example the Development or Production environment. You can find your Client-side Environment Key in the Environment
settings page.

![Image](/img/api-key.png)

### Initialization

Within your application delegate (usually _AppDelegate.swift_) add:

```swift
import FlagsmithClient
```

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

Flagsmith.shared.apiKey = "<YOUR_API_KEY>"
// The rest of your launch method code
}
```

Now you are all set to retrieve feature flags from your project. For example to list and print all flags:

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

Note that you can use:

- `flag.value?.stringValue`
- `flag.value?.intValue`
- `flag.value?.floatValue`

Based on your desired type.

To retrieve a feature flag boolean value by its name:

```swift
Flagsmith.shared.hasFeatureFlag(withID: "test_feature1", forIdentity: nil) { (result) in
    print(result)
}
```

To retrieve a config value by its name:

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

These methods can also specify a particular identity to retrieve the values for a user registration. See
[Identities](https://docs.flagsmith.com/managing-identities/) , using the **forIdentity** parameter.

To retrieve a trait for a particular identity (see
[Traits](https://docs.flagsmith.com/managing-identities/#identity-traits)):

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

### Swift Concurrency

When running with Swift version 5.5.2 and greater (Xcode 13.2), `async` versions of the Flagsmith API become available.
These are provided using the generic
[`withCheckedThrowingContinuation(function:_:)`](https://developer.apple.com/documentation/swift/3814989-withcheckedthrowingcontinuation)
Swift api, to wrap the closure based syntax. The `async`/`await` syntax provides a streamlined execution flow leading to
greater code clarity. For example:

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

## Override default configuration

By default, the client uses a default configuration. You can override the configuration as follows:

Override just the default API URI with your own:

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

Flagsmith.shared.apiKey = "<YOUR_API_KEY>"
Flagsmith.shared.baseURL = "https://<your-self-hosted-api>/api/v1/"
// The rest of your launch method code
}
```
