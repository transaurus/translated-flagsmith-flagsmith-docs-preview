---
title: Flagsmith Java SDK
sidebar_label: Java
description: Manage your Feature Flags and Remote Config in your Java applications.
slug: /clients/java
---

该库可用于服务端Java、Kotlin及Android应用程序。客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-java-client)。

## 入门指南

客户端库已发布至中央Maven仓库，可通过多种工具添加到您的项目中：

### Maven

在项目的`pom.xml`中添加以下依赖项

```xml
<dependency>
  <groupId>com.flagsmith</groupId>
  <artifactId>flagsmith-java-client</artifactId>
  <version>3.1</version>
</dependency>
```

### Gradle

```groovy
implementation 'com.flagsmith:flagsmith-java-client:2.8'
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目内的单个环境进行初始化，例如开发环境或生产环境。您可以在环境设置页面找到环境密钥。

![API密钥](/img/api-key.png)

### 获取项目功能开关

请前往[https://www.flagsmith.com/](https://www.flagsmith.com/)注册并创建账户

在应用程序中使用API密钥初始化Flagsmith客户端：

```java
FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                .setApiKey("YOUR_ENV_API_KEY")
                .build();
```

检查功能开关是否存在并已启用：

```java
boolean featureEnabled = flagsmithClient.hasFeatureFlag("my_test_feature");
if (featureEnabled) {
    // run the code that executes the enabled feature
} else {
    // run the code that doesn't include the feature
}
```

获取功能开关的配置值：

```java
String myRemoteConfig = flagsmithClient.getFeatureFlagValue("my_test_feature");
if (myRemoteConfig != null) {
    // run the code that uses the remote config value
} else {
    // run the code that doesn't depend on the remote config value
}
```

### 用户标识

用户标识功能允许您通过[Flagsmith控制台](https://www.flagsmith.com/)定向特定用户。

检查指定用户上下文下的功能是否存在：

```java
FeatureUser user = new FeatureUser();
user.setIdentifier("flagsmith_sample_user");
boolean featureEnabled = flagsmithClient.hasFeatureFlag("my_test_feature", user);
if (featureEnabled) {
    // run the code that executes the enabled feature for a given user
} else {
    // run the code that doesn't include the feature
}
```

获取指定用户上下文下功能开关的配置值：

```java
String myRemoteConfig = flagsmithClient.getFeatureFlagValue("my_test_feature", user);
if (myRemoteConfig != null) {
    // run the code that uses the remote config value
} else {
    // run the code that doesn't depend on the remote config value
}
```

设置用户特征：

```java
FeatureUser user = new FeatureUser();
user.setIdentifier(identifier);

FlagsAndTraits flagsAndTraits = flagsmithClient.identifyUserWithTraits(user, Arrays.asList(
    new Trait(null, "trait1", "some value1"),
    new Trait(null, "trait2", "some value2")));

// Since version 3.0, this method returns a FlagsAndTraits object, from which you can obtain the
// returned flags and / or traits.
List<Trait> traits = flagsAndTraits.getTraits();
List<Flag> flags = flagsAndTraits.getFlags();
```

获取指定用户上下文下的用户特征：

```java
List<Trait> userTraits = flagsmithClient.getTraits(user)
if (userTraits != null && userTraits) {
    // run the code that expects the user traits
} else {
    // run the code that doesn't depend on user traits
}
```

获取指定用户上下文下特定键的用户特征：

```java
Trait userTrait = flagsmithClient.getTrait(user, "cookies_key");
if (userTrait != null) {
    // run the code that uses the user trait
} else {
    // run the code that doesn't depend on the user trait
}
```

或获取指定用户上下文下多个特定键的用户特征：

```java
List<Trait> userTraits = flagsmithClient.getTraits(user, "cookies_key", "other_trait");
if (userTraits != null) {
    // run the code that uses the user traits
} else {
    // run the code doesn't depend on user traits
}
```

更新指定用户上下文下特定键的用户特征值：

```java
Trait userTrait = flagsmithClient.getTrait(user, "cookies_key");
if (userTrait != null) {
    // update the value for a user trait
    userTrait.setValue("new value");
    Trait updated = flagsmithClient.updateTrait(user, userTrait);
} else {
    // run the code that doesn't depend on the user trait
}
```

### 功能开关与特征

或通过单次调用获取用户的功能开关和特征：

```java
FlagsAndTraits userFlagsAndTraits = flagsmithClient.getUserFlagsAndTraits(user);
// get traits
List<Trait> traits = flagsmithClient.getTraits(userFlagsAndTraits, "cookies_key");
// or get a flag value
String featureFlagValue = flagsmithClient.getFeatureFlagValue("font_size", userFlagsAndTraits);
// or get flag enabled
boolean enabled = flagsmithClient.hasFeatureFlag("hero", userFlagsAndTraits);

// see above examples on how to evaluate flags and traits
```

## 覆盖默认配置

客户端默认使用标准配置，您可按如下方式覆盖配置：

仅覆盖默认API地址：

```java
FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                .setApiKey("YOUR_ENV_API_KEY")
                .withApiUrl("http://yoururl.com")
                .build();
```

完全覆盖配置项：

```java
FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
            .setApiKey("YOUR_ENV_API_KEY")
            .withConfiguration(FlagsmithConfig.newBuilder()
                    .baseURI("http://yoururl.com")
                    .connectTimeout(200)
                    .writeTimeout(5000)
                    .readTimeout(5000)
                    .build())
            .build();

```

### 日志记录

日志功能默认禁用。如需启用，请在客户端构建器上调用`.enableLogging()`：

```java
FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                // other configuration as shown above
                .enableLogging()
                .build();
```

Flagsmith采用[SLF4J](http://www.slf4j.org)框架且仅实现其API。若项目未集成SLF4J，需引入具体实现例如：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>${slf4j.version}</version>
</dependency>
```

### 自定义HTTP头

为所有HTTP调用添加自定义头：

```java
final HashMap<String, String> customHeaders = new HashMap(){{
    put("x-custom-header", "value1");
    put("x-my-key", "value2");
}};
FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
    // other configuration as shown above
    .withCustomHttpHeaders(customHeaders)
    .build();
```

### 内存缓存

:::note

缓存功能自2.6+版本起可用

:::

如需启用内存缓存功能（默认禁用），其主要优势在于能减少获取功能标志所需的HTTP请求次数。

Flagsmith采用高性能缓存库[Caffeine](https://github.com/ben-manes/caffeine)，其缓存效率接近最优水平。

若在Flagsmith客户端启用缓存但未设置参数（如下所示），将自动应用以下默认配置：

- 最大缓存数(maxSize): 10
- 写入后过期时间(expireAfterWrite): 5分钟
- 项目级缓存默认禁用（仅当配置缓存键时启用）

```java
// use in-memory caching with Flagsmith defaults as described above
final FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                .setApiKey("YOUR_ENV_API_KEY")
                .withConfiguration(FlagsmithConfig
                        .newBuilder()
                        .baseURI("http://yoururl.com")
                        .build())
                .withCache(FlagsmithCacheConfig
                        .newBuilder()
                        .build())
                .build();
```

如需修改默认设置，可通过构建器方法进行覆盖：

```java
// use in-memory caching with custom configuration
final FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                .setApiKey("YOUR_ENV_API_KEY")
                .withConfiguration(FlagsmithConfig
                        .newBuilder()
                        .baseURI("http://yoururl.com")
                        .build())
                .withCache(FlagsmithCacheConfig
                        .newBuilder()
                        .maxSize(100)
                        .expireAfterWrite(10, TimeUnit.MINUTES)
                        .recordStats()
                        .enableEnvLevelCaching("some-key-to-avoid-clashing-with-user-identifiers")
                        .build())
                .build();
```

用户标识符将作为缓存键使用，便于实现细粒度的缓存控制。如需操作缓存：

```java
// this will return null if caching is disabled
final FlagsmithCache cache = flagsmithClient.getCache();
// you can now discard a single or all entries in the cache
cache.invalidate("user-identifier");
// or
cache.invalidateAll();
// get stats (if you have enabled them in the cache configuration, otherwise all values will be zero)
final CacheStats stats = cache.stats();
// check if flags for a user identifier are cached
final FlagsAndTraits flags = cache.getIfPresent("user-identifier");
```

由于用户标识符作为缓存键使用，启用项目级缓存需配置专属缓存键。请确保所选项目级缓存键永远不会与用户标识符重复。

```java
// use in-memory caching with Flagsmith defaults and project level caching enabled
final String projectLevelCacheKey = "some-key-to-avoid-clashing-with-user-identifiers";
final FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                .setApiKey("YOUR_ENV_API_KEY")
                .withConfiguration(FlagsmithConfig
                        .newBuilder()
                        .baseURI("http://yoururl.com")
                        .build())
                .withCache(FlagsmithCacheConfig
                        .newBuilder()
                        .enableEnvLevelCaching(projectLevelCacheKey)
                        .build())
                .build();

// if you need to access the cache directly, you can do this:
final FlagsmithCache cache = flagsmithClient.getCache();
// invalidate project level cache
cache.invalidate(projectLevelCacheKey);
// check if project level flags have been cached
final FlagsAndTraits flags = cache.getIfPresent(projectLevelCacheKey);
```

### 默认标志/配置值

标志评估始终会返回值，即使评估失败（如标志不存在于Flagsmith或HTTP调用失败）。默认情况下，错误状态下标志将评估为`false`，配置值评估为`null`。如需覆盖此行为，可使用以下方法：

```java
final FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                .setApiKey("YOUR_ENV_API_KEY")
                .setDefaultFlagPredicate(
                    (featureFlagName) -> true /* your logic for default flags */
                  )
                .setDefaultFlagValueFunction(
                  (featureFlagName) ->
                    "my-new-default-value" /* your logic for default config values */
                 )
                .build();
```

但上述代码仅适用于特定标志名的评估场景，例如`client.getFeatureFlagValue("flag-name")`或`client.hasFeatureFlag("flag-name")`。当调用`client.getFeatureFlags()`方法发生错误时，将返回空标志列表。如需修改此行为，可配置默认标志列表：

```java
final FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
                .setApiKey("YOUR_ENV_API_KEY")
                .setDefaultFeatureFlags(new HashSet<String>() {{
                    add("flag-name-1");
                    add("flag-name-2");
                    add("flag-name-3");
                    }})
                .build();
```

设置默认标志名还意味着：即使Flagsmith服务器未配置某些标志，这些标志仍会包含在返回的标志列表中（如上例所示）。