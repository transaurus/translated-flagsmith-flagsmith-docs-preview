---
title: Flagsmith .Net SDK
sidebar_label: .Net
description: Manage your Feature Flags and Remote Config in your .Net applications.
slug: /clients/dotnet
---

该SDK可用于.NET Core、.NET Framework、Mono、Xamarin以及通用Windows平台应用程序。

客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-dotnet-client)。

## 快速入门

客户端库可通过NuGet获取，支持多种工具添加到项目中。包地址详见  
[https://www.nuget.org/packages/Flagsmith/](https://www.nuget.org/packages/Flagsmith/)

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目的单个环境（如开发环境或生产环境）进行初始化。环境密钥可在环境设置页面获取。

![API密钥](/img/api-key.png)

## 使用指南

### 获取项目功能开关

请先注册并创建账号：[https://flagsmith.com/](https://www.flagsmith.com/)

在应用程序中，使用您的环境API密钥和API URL初始化Flagsmith客户端。

```dotnet
FlagsmithConfiguration configuration = new FlagsmithConfiguration()
{
    ApiUrl = "https://api.flagsmith.com/api/v1/",
    EnvironmentKey = "env-key-goes-here"
};

FlagsmithClient client = new FlagsmithClient(configuration);

```

此后即可通过`FlagsmithClient`类的静态变量`instance`在应用任意位置调用。

检查功能开关是否存在并启用：

```dotnet
featureEnabled = (bool)FlagsmithClient.instance.HasFeatureFlag("my_test_feature").GetAwaiter().GetResult();
if (featureEnabled) {
    // run the code to execute enabled feature
} else {
    // run the code if feature switched off
}
```

获取远程配置参数值：

```dotnet
string myRemoteConfig = FlagsmithClient.instance.GetFeatureValue("my_test_feature").GetAwaiter().GetResult();
if (myRemoteConfig != null) {
    // run the code to use remote config value
} else {
    // run the code without remote config
}
```

### 用户标识

标识用户后，您可通过[Flagsmith控制台](https://www.flagsmith.com/)定向特定用户。

检查针对特定用户的功能开关状态：

```dotnet
featureEnabled = (bool)FlagsmithClient.instance.HasFeatureFlag("my_test_feature", "my_user_id").GetAwaiter().GetResult();
if (featureEnabled) {
    // run the code to execute enabled feature for given user
} else {
    // run the code when feature switched off
}
```

获取特定用户的远程配置值：

```dotnet

string myRemoteConfig = FlagsmithClient.instance.GetFeatureValue("my_test_feature", "my_user_id").GetAwaiter().GetResult();
if (myRemoteConfig != null) {
    // run the code to use remote config value
} else {
    // run the code without remote config
}
```

获取用户特征：

```dotnet
List<Trait> userTraits = FlagsmithClient.instance.GetTraits("my_user_id").GetAwaiter().GetResult();
if (userTraits != null && userTraits) {
    // run the code to use user traits
} else {
    // run the code without user traits
}
```

获取特定用户特征：

```dotnet

string userTrait = FlagsmithClient.instance.GetTrait("my_user_id", "my_user_trait").GetAwaiter().GetResult();
bool userTrait = FlagsmithClient.instance.GetTrait("my_user_id", "my_user_bool_trait").GetAwaiter().GetResult();
int userTrait = FlagsmithClient.instance.GetTrait("my_user_id", "my_user_number_trait").GetAwaiter().GetResult();
```

获取筛选后的用户特征：

```dotnet
List<Trait> userTraits = await FlagsmithClient.instance.GetTrait("my_user_id", new List<string> { "specific_key", /* rest of elements */ });
if (userTraits != null) {
    // run the code to use user traits
} else {
    // run the code without user traits
}
```

设置或更新用户特征：

```dotnet
Trait userTrait = await FlagsmithClient.instance.SetTrait("my_user_id", "my_user_trait", "blue");
Trait userTrait = await FlagsmithClient.instance.SetTrait("my_user_id", "my_user_number_trait", 4);
Trait userTrait = await FlagsmithClient.instance.SetTrait("my_user_id", "my_user_bool_trait", true);
```

递增数值型用户特征：

```dotnet
Trait userTrait = await FlagsmithClient.instance.IncrementTrait("my_user_id", "my_user_number_trait", 1);
```

检索用户身份信息（包含功能开关及特征）：

```dotnet
Identity userIdentity = await FlagsmithClient.instance.GetUserIdentity("my_user_id");
if (userIdentity != null) {
  // Run the code to use user identity i.e. userIdentity.flags or userIdentity.traits
}
```