---
description: Manage your Feature Flags and Remote Config in your REST APIs.
sidebar_label: Overview
sidebar_position: 1
---

# SDK概览

Flagsmith提供多种编程语言的SDK。如果您希望直接调用API，我们还提供[REST API](rest.md)接口。

SDK分为两种类型：**客户端**和**服务端**SDK。由于运行环境差异，这两类SDK采用不同的工作模式。

:::tip

**环境密钥**分为客户端和服务端两种类型。请根据所用SDK类型确保使用正确的密钥。

:::

## 客户端SDK

客户端SDK运行在浏览器或移动设备等不可信环境中。例如使用Javascript SDK的网页用户可创建新身份标识、查看功能开关，甚至可能[向身份标识写入特征值](../advanced-use/system-administration.md#preventing-client-sdks-to-set-traits)。

客户端SDK还受限于[可访问的数据类型](/guides-and-examples/integration-approaches#segment-and-targeting-rules-are-not-leaked-to-the-client)。

客户端环境密钥设计为公开共享，例如可嵌入发送给浏览器的HTML/JS代码中。

客户端SDK直接调用我们的[边缘API](../advanced-use/edge-api.md)获取功能开关。

各平台客户端SDK文档：

- [Javascript](/clients/javascript)
- [Android/Kotlin](/clients/android)
- [Flutter](/clients/flutter)
- [iOS/Swift](/clients/ios)
- [React & React Native](/clients/react)
- [Next.js, Svelte与SSR](/clients/next-ssr)

## 服务端SDK

[服务端SDK](/clients/server-side.md)运行在可信环境中（通常为您控制的服务器基础设施）。因此服务端环境密钥应视为机密信息，不可公开共享。

:::tip

服务端SDK支持两种工作模式：

1. `远程评估`
2. `本地评估`

请根据使用场景选择合适模式，下文将详细说明各模式的优缺点。

:::

### 1 - 远程评估

该模式下，每次获取功能开关时SDK都会向Flagsmith API发起请求。

![远程评估示意图](/img/sdk-remote-evaluation.svg)

`远程评估`是默认模式，初始化SDK后即自动启用该模式。

此模式与[客户端SDK](#client-side-sdks)的工作机制相同。

### 2 - 本地评估

该模式下所有功能开关都在您的服务器本地计算。Flagsmith SDK内置了开关引擎实现，可在您的服务器环境中运行。

![本地评估示意图](/img/sdk-local-evaluation.svg)

需手动配置SDK以启用`本地评估`模式。具体配置方法请参考[SDK配置选项](server-side.md#configuring-the-sdk)中对应语言的说明。

当SDK以`本地评估(Local Evaluation)`模式初始化时，它会从Flagsmith API获取该环境的所有相关数据。这包括该环境下的所有功能开关(Flags)、开关值、细分规则(Segment Rules)、细分覆盖(Segment Overrides)等。这些完整的环境数据使得Flagsmith SDK能够在您的服务器基础设施中_本地_且_原生_地运行功能开关引擎(Flag Engine)。

这种模式的主要优势在于降低延迟并提升性能。您的服务端代码无需在每次用户请求功能开关时访问Flagsmith API——所有开关都可以在本地计算得出，因此无需阻塞等待Flagsmith API的响应。

:::tip

SDK需要请求环境的所有数据才能运行。由于部分数据可能涉及敏感信息（例如细分规则），SDK需要使用特定的`服务端环境密钥(Server-side Environment Key)`。这与常规的`客户端环境密钥(Client-side Environment Key)`不同。`服务端环境密钥`属于敏感数据，_严禁_对外共享。

:::

为了保持环境数据的实时性，运行在`本地评估`模式的SDK会定期轮询Flagsmith API，并使用API的变更更新本地环境数据。默认情况下SDK每`60`秒轮询一次，该频率可在各语言SDK中配置。

理解[本地评估模式的优缺点](#pros-cons-and-caveats)非常重要。

### 客户端SDK

所有客户端SDK仅支持`远程评估(Remote Evaluation)`模式，无法运行在`本地评估模式`。这主要基于数据敏感性考虑。由于部分数据可能涉及敏感信息（例如细分规则），我们仅允许客户端SDK运行在`远程评估`模式。

:::info

由于客户端几乎总是远离您的服务器基础设施运行，采用`本地评估`模式几乎不会带来性能优势。

:::

## 优缺点与注意事项

### 远程评估模式

- 用户身份(Identities)会持久化存储在Flagsmith数据库中
- 支持通过仪表盘配置的身份覆盖(Identity Overrides)
- 所有集成功能均可正常使用

### 本地评估模式

- 用户身份_不会_发送至API，因此不会被持久化存储
- 由于本地模式不会为每个功能开关请求连接数据库，因此无法从API读取用户特征的完整数据。这意味着您必须为特定身份请求功能开关时提供完整的特征数据(Traits)。我们的各语言SDK都提供了相应方法来实现这一点
- [身份覆盖](../basic-features/managing-identities#identity-overrides)功能完全失效
- 基于分析数据的集成功能无法运行

本地评估模式的优势在于能够更高效地处理功能开关评估，因为所有计算都在本地完成。

:::tip

当需要针对特定身份进行定向时，您可以通过创建针对该用户的细分(Segment)，并为该细分添加覆盖规则来实现。

:::