---
title: Integration Approaches
---

## 客户端SDK的Flag端点均为公开

我们SDK调用的API端点都是公开的。您的环境API密钥也应视为公开信息，其性质类似于Google Analytics跟踪密钥。

### 用户分群与定向规则不会泄露至客户端

如果在客户端SDK（网页浏览器、移动应用）内评估功能开关，则需要将基于用户分群的全部定向规则发送至客户端。鉴于这些端点默认公开，我们认为这可能导致敏感信息泄露。最佳实践是在我们的服务器或您的服务器上评估功能开关，而非客户端。

### 通过简单HTTP GET即可获取功能开关

无需运行复杂的规则评估逻辑，只需调用我们的端点即可获取功能开关。您不会收到任何关于用户分群或发布规则的信息，这是刻意设计。若需在应用中自行实现HTTP客户端，仅需发起简单的HTTP GET请求即可。

### 构建时获取功能开关

:::tip

我们提供[Flagsmith CLI工具](https://github.com/Flagsmith/flagsmith-cli)可简化此流程！

:::

更高级的技术方案是在构建时从Flagsmith API获取功能开关默认值，并将其嵌入应用构建。具体步骤如下：

1. 将代码推送至Git仓库
2. 触发自动化构建流水线
3. 在流水线阶段中调用`/flags`端点获取当前默认开关状态，并将JSON响应存入应用构建
4. 应用启动时优先读取内嵌的JSON文件获取初始开关配置
5. 异步调用Flagsmith API获取最新开关配置

## 本地缓存功能开关

此方案取决于应用运行时能否在主机操作系统持久化数据。在应用环境中本地缓存开关可确保后续启动时无需阻塞等待Flagsmith API调用。典型工作流如下：

1. 使用合理默认值构建应用
2. 启动时先使用默认值，同时异步调用Flagsmith API获取最新开关
3. 获取最新开关后将其存入本地存储
4. 后续启动时优先检查本地存储中的开关配置
5. 始终异步调用Flagsmith API保持配置更新

官方[Javascript客户端](/clients/javascript/)已内置可选缓存功能。

## 服务器端缓存功能开关

:::tip

注意：我们的服务端SDK也支持[本地评估开关](../clients/overview.md)。

:::

在服务器环境中运行时，SDK难以自动感知可用的缓存基础设施。因此服务端缓存需要手动集成，但实现非常简单：

1. 服务器启动时从Flagsmith API获取开关配置，此时配置存在于运行时内存中
2. 若有可用缓存设施（如memcache、redis等），可将环境开关存入缓存
3. 在Flagsmith中设置[Web Hook](/advanced-use/system-administration.md#web-hooks)，将开关变更事件推送至服务器
4. 编写接收开关变更事件的API端点，将更新存入本地缓存
5. 后续可直接依赖本地缓存获取最新开关配置