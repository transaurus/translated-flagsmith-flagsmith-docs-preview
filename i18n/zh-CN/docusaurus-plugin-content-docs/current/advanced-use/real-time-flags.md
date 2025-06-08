# 实时功能开关

:::tip

实时功能开关是我们的SaaS Scale-Up和企业版计划的一部分。

:::

:::info

实时功能开关目前处于测试阶段。请联系我们加入测试计划！

:::

## 概述

实时功能开关允许您将功能开关的变更从Flagsmith实时推送到已连接的客户端。

## 工作原理

我们的SDK会发起一个长连接请求（实际上是
[服务器发送事件](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)），连接到
我们的实时基础设施。该连接将在整个SDK会话期间保持开放，监听来自API的事件。

当通过Flagsmith仪表板或API更新某个环境时，所有连接到该环境流的客户端都会收到一条消息（实际上是一个时间戳），提醒它们刷新功能开关状态。

随后需要您在SDK集成中，将新的功能开关状态反映到应用程序中。

## 定价

实时功能开关包含在Scale-Up和企业版计划中。

## SDK支持

我们正在为所有SDK构建支持。我们将优先考虑客户端SDK而非服务端SDK，但也欢迎提交Pull Request！

### 客户端

| Language       | Support |
| -------------- | ------- |
| Javascript     | ✅      |
| iOS/Swift      | ❌      |
| Android/Kotlin | ❌      |
| Dart/Flutter   | ❌      |

### 服务端

| Language | Support |
| -------- | ------- |
| Node.js  | ❌      |
| Java     | ❌      |
| .Net     | ❌      |
| Python   | ❌      |
| Ruby     | ❌      |
| Rust     | ❌      |
| Go       | ❌      |
| Elixir   | ❌      |

## 注意事项

### 基于Identity的覆盖不会触发更新

该功能正在开发中，预计将于2023年第二季度发布。

### Identity特征更新不会触发更新

该功能正在开发中，预计将于2023年第二季度发布。

## 底层实现原理

当SDK启用实时流功能时，客户端将尝试连接到：
`eventSourceUrl + "sse/environments/" + environmentID + "/stream"`。该流服务基于
[服务器发送事件](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)构建，而非WebSocket！

默认情况下，`eventSourceUrl`设置为`https://realtime.flagsmith.com/`。

每次通过REST API获取功能开关时（通过identify、get flags、set traits等操作），SDK内部会更新一个时间戳，记录功能开关的新鲜度。

当连接到流服务时，如果Flagsmith环境状态发生变化，客户端会收到新的时间戳事件。若该时间戳值大于内部记录的时间戳，则重新获取功能开关。