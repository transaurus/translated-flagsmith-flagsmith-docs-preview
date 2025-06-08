---
title: Flagsmith Android/Kotlin SDK
sidebar_label: Android / Kotlin
description: Manage your Feature Flags and Remote Config in your Android applications.
slug: /clients/android
---

该SDK适用于使用Kotlin编写的Android应用程序。客户端源代码可在[Github](https://github.com/Flagsmith/flagsmith-kotlin-android-client/)上获取。

## 安装指南

### Gradle - 应用模块

在项目路径`app/build.gradle`中添加新依赖项

```groovy
//flagsmith
implementation 'com.github.Flagsmith:flagsmith-kotlin-android-client:1.0.1'
```

您可以在GitHub仓库的[发布页面](https://github.com/Flagsmith/flagsmith-kotlin-android-client/releases)找到最新版本。

### Gradle - 项目配置

在Gradle 7+版本中，更新`settings.gradle`文件以包含JitPack仓库（如尚未配置）

```groovy
repositories {
    google()
    mavenCentral()

    maven { url "https://jitpack.io" }
}
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目的单个环境进行初始化，例如开发或生产环境。您可以在环境设置页面找到客户端环境密钥。

![图片](/img/api-key.png)

## 初始化配置

### 在Activity的`onCreate()`方法中初始化

```kotlin
lateinit var flagsmith : Flagsmith

override fun onCreate(savedInstanceState: Bundle?) {
    initFlagsmith();
}

private fun initFlagsmith() {
    flagsmith = Flagsmith(environmentKey = FlagsmithConfigHelper.environmentDevelopmentKey, context = context)
}
```

## 自定义配置

Flagsmith SDK提供多种初始化参数，多数为可选配置，可根据需求灵活调整：

- `environmentKey` 从Flagsmith仪表板获取API密钥并在此传入
- `baseUrl` 默认连接Flagsmith官方后端，自托管用户可在此配置私有API地址  
- `context` 使用Flagsmith分析功能时必须传入当前上下文
- `enableAnalytics` 启用分析功能（默认true），如需禁用上下文追踪可关闭此选项
- `analyticsFlushPeriod` SDK向服务器推送分析事件的间隔时间（秒）

## 功能开关管理

完成配置后即可获取项目功能开关。例如列出并打印所有开关：

```kotlin
flagsmith.getFeatureFlags { result ->
    result.fold(
        onSuccess = { flagList ->
            Log.i("Flagsmith", "Current flags:")
            flagList.forEach { Log.i("Flagsmith", "- ${it.feature.name} - enabled: ${it.enabled} value: ${it.featureStateValue ?: "not set"}") }
        },
        onFailure = { err ->
            Log.e("Flagsmith", "Error getting feature flags", err)
        })
}
```

### 通过`featureId`获取开关对象

根据功能名称获取布尔型开关值：

```kotlin
flagsmith.hasFeatureFlag(forFeatureId = "test_feature1") { result ->
    val isEnabled = result.getOrDefault(true)
    Log.i("Flagsmith", "test_feature1 is enabled? $isEnabled")
}
```

### 为用户身份创建特征值

```kotlin
flagsmith.setTrait(Trait(key = "set-from-client", value = "12345"), identity = "test@test.com") { result ->
    result.fold(
        onSuccess = { _ ->
            Log.i("Flagsmith", "Successfully set trait")

        },
        onFailure = { err ->
            Log.e("Flagsmith", "Error setting trait", err)
        })
}
```

### 获取所有特征

根据特定身份获取特征值，详见[特征管理文档](../../basic-features/managing-identities.md#identity-traits)

```kotlin
flagsmith.getTraits(identity = "test@test.com") { result ->
    result.fold(
        onSuccess = { traits ->
            traits.forEach {
                Log.i("Flagsmith", "Trait - ${it.key} : ${it.value}")
            }
        },
        onFailure = { err ->
            Log.e("Flagsmith", "Error setting trait", err)
        })
}
```

## 覆盖默认API地址

初始化时可通过以下方式自定义API基础URL：

```kotlin
        flagsmith = Flagsmith(
            environmentKey = Helper.environmentDevelopmentKey,
            context = context,
            baseUrl = "https://hostedflagsmith.company.com/")
```