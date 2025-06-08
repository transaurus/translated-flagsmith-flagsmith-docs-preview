---
title: Flagsmith Flutter SDK
sidebar_label: Flutter
description: Manage your Feature Flags and Remote Config in your Flutter applications.
slug: /clients/flutter
---

该SDK可用于Flutter应用程序。客户端源代码托管于
[Github](https://github.com/flagsmith/flagsmith-flutter-client)。

Flagsmith Flutter SDK支持iOS、Android及Web平台。

## 开始使用

客户端库可通过[https://pub.dev/packages/flagsmith](https://pub.dev/packages/flagsmith)获取：

```dart
dependencies:
  flagsmith:
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)项目中单个环境进行初始化，
例如开发环境或生产环境。您可以在环境设置页面找到环境密钥。

![图片](/img/api-key.png)

### 获取项目特性开关

请先在[https://flagsmith.com/](https://flagsmith.com/)注册并创建账户

在应用程序中，使用您的API密钥初始化Flagsmith客户端：

```dart
import 'package:flagsmith/flagsmith.dart';

final flagsmithClient = FlagsmithClient(
        apiKey: 'YOUR_ENV_API_KEY'
        config: config,
        seeds: <Flag>[
            Flag.seed('feature', enabled: true),
        ],
    );
await flagsmithClient.initialize();
await flagsmithClient.getFeatureFlags(reload: true) // fetch updates from api
```

若偏好异步初始化，可使用：

```dart
import 'package:flagsmith/flagsmith.dart';

final flagsmithClient = await FlagsmithClient.init(
        apiKey: 'YOUR_ENV_API_KEY',
        config: config,
        seeds: <Flag>[
            Flag.seed('feature', enabled: true),
        ],
    );
await flagsmithClient.getFeatureFlags(reload: true) // fetch updates from api
```

检查特性开关是否存在并启用：

```dart
bool featureEnabled = await flagsmithClient.hasFeatureFlag("my_test_feature");
if (featureEnabled) {
    // run the code to execute enabled feature
} else {
    // run the code if feature switched off
}
```

获取特性开关的配置值：

```dart
final myRemoteConfig = await flagsmithClient.getFeatureFlagValue("my_test_feature");
if (myRemoteConfig != null) {
    // run the code to use remote config value
} else {
    // run the code without remote config
}
```

监听获取请求状态：

```dart
flagsmithClient.loading.listen((state){
    // FlagsmithLoading.loading
    // FlagsmithLoading.loaded
});
```

监听特性开关变更：

```dart
flagsmithClient.stream("my_test_feature").listen((value){
    // call to action
});
```

```dart
StreamBuilder(
    stream: flagsmithClient.stream("my_test_feature"),
    builder: (context, AsyncSnapshot<String> snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
            return CircularProgressIndicator();
        }
        return TextButton(
            onPressed: snapshot.data.enabled ? (){} : null,
            child: Text('Call to Action'),);
    },
),
```

## 缓存开关

您可以使用缓存替代async/await：

```dart
final config = FlagsmithConfig(
    baseURI: 'http://yoururl.com/',
    connectTimeout: 200,
    receiveTimeout: 500,
    sendTimeout: 500,
    storeType = StoreType.inMemory,
    caches: true, // mandatory if you want to use caches
);
await flagsmithClient.initialize();

final flagsmithClient = await FlagsmithClient.init(
        apiKey: 'YOUR_ENV_API_KEY',
        config: config,
        seeds: <Flag>[
            Flag.seed('feature', enabled: true),
        ],
    );

await flagsmithClient.getFeatureFlags(reload: true); // fetch updates from api
bool isFeatureEnabled = flagsmithClient.hasCachedFeatureFlag('feature');
```

### 用户识别

用户识别功能允许您通过[Flagsmith控制台](https://flagsmith.com/)定位特定用户。

检查特性是否对指定用户标识存在：

```dart
final user = Identity(identifier: 'flagsmith_sample_user');
bool featureEnabled = await flagsmithClient.hasFeatureFlag('my_test_feature', user: user);
if (featureEnabled) {
    // run the code to execute enabled feature for given user
} else {
    // run the code when feature switched off
}
```

获取指定用户标识的特性开关配置值：

```dart
final myRemoteConfig = await flagsmithClient.getFeatureFlagValue('my_test_feature', user: user);
if (myRemoteConfig != null) {
    // run the code to use remote config value
} else {
    // run the code without remote config
}
```

获取指定用户标识的所有用户特征：

```dart
final userTraits = await flagsmithClient.getTraits(user)
if (userTraits != null && userTraits) {
    // run the code to use user traits
} else {
    // run the code without user traits
}
```

获取指定用户标识的特定特征键值：

```dart
final userTrait = await flagsmithClient.getTrait(user, 'cookies_key');
if (userTrait != null) {
    // run the code to use user trait
} else {
    // run the code without user trait
}
```

或获取指定用户标识的多个特征键值：

```dart
final userTraits = await flagsmithClient.getTraits(user, keys: ['cookies_key', 'other_trait']);
if (userTraits != null) {
    // run the code to use user traits
} else {
    // run the code without user traits
}
```

更新指定用户标识的特征：

```dart
final userTrait = await flagsmithClient.getTrait(user, 'cookies_key');
if (userTrait != null) {
    // update value for user trait
    var updatedTrait = userTrait.copyWith(value: 'new value');
    Trait updated = await flagsmithClient.updateTrait(user, updatedTrait);
} else {
    // run the code without user trait
}
```

## 重置存储

重置存储并重新初始化默认值：

```dart
await flagsmithClient.reset();
```

## 覆盖默认配置

默认情况下，客户端使用标准配置。您可按如下方式覆盖配置：

```dart
final flagsmithClient = FlagsmithClient(
      config: FlagsmithConfig(
          baseURI: 'http://yoururl.com/'
      ), apiKey: 'YOUR_ENV_API_KEY');
```

使用自定义配置覆盖默认值：

```dart
final flagsmithClient = FlagsmithClient(
      config: FlagsmithConfig(
          baseURI: 'http://yoururl.com/',
          connectTimeout: 200,
          receiveTimeout: 500,
          sendTimeout: 500,
          storeType = StoreType.inMemory,
          caches: true,
      ), apiKey: 'YOUR_ENV_API_KEY');
```