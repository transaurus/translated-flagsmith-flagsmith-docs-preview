---
title: Integration Approaches
---

我们投入了大量工作以确保Flagsmith API快速、稳定、可靠且具备容错能力。尽管如此，仍有一些简单技巧可进一步提升性能，为用户提供最佳体验。

## SDK标志接口是公开的

所有SDK调用的API端点均为公开接口。您的环境API密钥也应视为公开信息，其性质类似于Google Analytics跟踪密钥。

## 标志在服务端评估

Flagsmith与其他功能标志服务的不同之处在于，我们会在服务器端执行标志规则评估。这种设计具有以下优势：

### 用户分群与定向规则不会泄露至客户端

若在SDK内评估标志，则需将基于用户分群等的完整定向规则集发送至客户端。鉴于这些端点默认公开，我们认为这可能导致敏感信息泄露。标志评估的最佳场所理应是我们的服务器。

### 通过单一HTTP GET请求即可获取标志

您无需执行复杂的规则评估流程，仅需调用我们的端点即可获取标志。根据设计，您不会收到任何关于用户分群或灰度发布规则的信息。若需在应用中自行实现HTTP客户端，仅需发起简单的HTTP GET请求即可完成标志获取。

### 我们的SDK极其轻量

我们设计的SDK保持最小化依赖。由于无需在客户端执行规则评估，我们的SDK代码库得以精简，避免引入不必要的依赖项。

## 合理的默认值

无论您的应用是移动应用还是服务端渲染的Web平台，采用合理且安全的标志默认值都是明智之举。以下两种实现方式值得推荐：

### 硬编码默认值

在代码中存储标志默认值是最简单的实现方式。例如在Java中可采用如下方式：

```java
    public static boolean FF_FREEZE_DELINQUENT_ACCOUNTS = false;
    public static boolean FF_KYC_BUTTON = true;
    public static int FF_TWILIO_IMPORT_DAYS_TO_PROCESS = 45;
    public static boolean FF_YOTI_INCLUDE_LIVENESS = true;
    public static String FF_YOTI_UPLOAD_TYPE = "CAMERA";

    public static void setup() {

    FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
            .setApiKey(Play.configuration.getProperty("flagsmith.apikey"))
            .build();

    FF_FREEZE_DELINQUENT_ACCOUNTS = flagsmithClient.hasFeatureFlag("freeze_delinquent_accounts");
    FF_KYC_BUTTON = flagsmithClient.hasFeatureFlag("kyc_button");
    FF_YOTI_INCLUDE_LIVENESS = flagsmithClient.hasFeatureFlag("yoti_include_liveness");
    FF_YOTI_UPLOAD_TYPE = flagsmithClient.getFeatureFlagValue("yoti_upload_type");
```

这样当Flagsmith客户端因故无法连接API并超时时，您的应用仍能基于合理的默认值正常运行。

### 构建时标志获取

更高级的技术是在构建时从Flagsmith API获取标志默认值，并将其包含在应用构建中。具体流程可能如下：

1. 将代码推送至Git仓库
2. 触发自动化构建流水线
3. 流水线中包含从`/flags`端点获取当前默认标志状态，并将JSON响应存入应用构建的环节
4. 应用启动时首先读取内嵌的JSON文件获取合理默认标志配置
5. 异步调用Flagsmith API获取最新标志与配置值

## 本地标志缓存

此方案取决于应用是否具备在运行时向宿主操作系统持久化数据的能力。在应用环境中本地缓存标志可确保后续启动时无需阻塞等待Flagsmith API调用。典型工作流如下：

1. 使用合理默认值构建应用
2. 启动应用时先使用默认值，同时异步调用Flagsmith API获取最新标志
3. 获取最新标志后将其存储在本地
4. 后续应用启动时检查本地存储是否存在可用标志，若存在则立即加载
5. 异步调用Flagsmith API获取最新标志

官方[Javascript客户端](/clients/javascript/)在SDK中内置了可选的缓存功能。

## 在服务器端缓存功能标志

当在服务器环境中运行Flagsmith SDK时，SDK难以自动判断可用的缓存基础设施类型。因此，在服务器环境中缓存功能标志需要手动集成，但实现起来相当简单：

1. 服务器启动时，从Flagsmith API获取功能标志。此时标志会驻留在服务器运行时内存中。
2. 若存在缓存基础设施（如memcache、redis等），可将该环境的功能标志存储至缓存系统。
3. 在Flagsmith中配置[Web钩子](/advanced-use/system-administration.md#web-hooks)，用于向服务器基础设施发送标志变更事件。
4. 在基础设施中编写API端点接收标志变更事件，并将其存储至本地缓存。
5. 此后即可依赖本地缓存获取最新功能标志。