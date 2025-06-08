# 边缘API

[Flagsmith架构](/guides-and-examples/integration-approaches#flags-are-evaluated-server-side)基于服务端标志评估引擎。这种设计带来诸多优势，但也会增加延迟——尤其是当调用方地理位置远离我们当前API所在的欧盟区域时。

为解决这一问题，我们开发了全球边缘API。该API旨在为全球任意位置的SDK请求提供100毫秒内的响应。为实现这一目标，我们采用了以下AWS组件：

## 启用边缘API

:::tip

边缘API目前处于测试阶段。如需加入测试计划，请通过[Discord](https://discord.gg/hFhxNtXzgm)、页面底部聊天组件或邮件support@flagsmith.com联系我们

:::

加入测试后，您只需将SDK指向新的Flagsmith边缘API URL（该URL将指向我们的边缘Cloudfront CDN）即可完成配置。

例如在Java SDK中，只需添加`withApiUrl`配置：

```java
FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
        .setApiKey("aaa"))
        .withApiUrl("https://edge.api.flagsmith.com/api/v1/")
        .build();
```

具体URL前缀覆盖方法请查阅各语言SDK文档。

## 当前限制

测试阶段平台存在以下限制，这些限制将在正式版发布前解决：

### 身份数据仅从核心API同步至边缘API

当您在核心API中写入身份数据时，该数据会同步至边缘API存储。但反向同步尚未实现：边缘API的写入操作不会同步回核心API。

### 增量/减量端点已弃用

不过您可能原本就不知道这些端点的存在？

### API响应已精简

我们移除了核心API响应中SDK未使用的冗余字段。虽然不影响SDK功能，但直接使用REST API时需注意以下字段变更：

```txt
trait.id
flag.feature.created_data
flag.feature.description
flag.feature.initial_value
flag.feature.default_enabled
flag.environment
flag.identity
flag.feature_segment
```

## 工作原理

### Lambda@Edge

我们将核心[规则引擎](https://github.com/Flagsmith/flagsmith-engine)从REST API中解耦，使其可同时作为Flagsmith API和Lambda函数的依赖组件。通过将SDK客户端指向全局CDN域名`edge.api.flagsmith.com`，您的请求将由距离最近的AWS数据中心内的Lambda函数处理，从而实现低延迟响应。

### DynamoDB全局表

系统状态数据（包括项目环境配置和身份信息）存储在DynamoDB全局表中，这些表格通过全球复制实现数据同步。

当前仅实现环境数据的全局同步（参见前文限制说明），身份数据同步功能即将推出。

Lambda函数会连接最近的DynamoDB表格来获取环境及身份数据。