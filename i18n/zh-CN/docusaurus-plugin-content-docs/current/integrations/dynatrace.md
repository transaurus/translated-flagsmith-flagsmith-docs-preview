---
title: Dynatrace Integration
sidebar_label: Dynatrace
hide_title: true
---

![Dynatrace](/img/integrations/dynatrace/dynatrace-logo.svg)

## 功能开关变更事件

您可以将Flagsmith与Dynatrace集成，将Flagsmith中的功能开关变更事件发送至Dynatrace事件流。

:::tip

Flagsmith JavaScript SDK 也可与 Dynatrace JavaScript SDK 进行交互。
[更多信息请参阅我们的JS文档页面](../clients/client-side/javascript.md#dynatrace-javascript-sdk-integration)。

:::

### 集成设置

1. 登录Dynatrace并创建具有以下权限的新访问令牌：
   - `API v2 作用域`：`接收事件`
2. 进入Flagsmith > 集成 > Dynatrace > 添加集成
3. 选择您要追踪的`Flagsmith环境`
4. `基础URL`取决于您的Dynatrace部署方式：
   - 托管版：`https://{您的域名}/e/{您的环境ID}/`
   - SaaS版：`https://{您的环境ID}.live.dynatrace.com/`
   - 环境ActiveGate：`https://{您的ActiveGate域名}/e/{您的环境ID}/`
5. `API密钥`即步骤1中创建的访问令牌
6. `实体选择器`用于定义要将功能开关变更事件关联到哪些Dynatrace实体
   [查阅Dynatrace文档](https://www.dynatrace.com/support/help/dynatrace-api/environment-api/entity-v2/entity-selector)
   获取更多相关信息

### 工作原理

完成设置后，尝试在配置过程中指定的环境中修改某个功能开关值。然后查看`实体选择器`中定义的任一Dynatrace实体，您将在事件面板中看到部署事件。

![Dynatrace事件](/img/integrations/dynatrace/dynatrace-events-panel.png)

## JavaScript SDK间通信

您也可以通过我们的
[JavaScript集成](../clients/client-side/javascript.md#dynatrace-javascript-sdk-integration)
将用户身份标识的功能开关值从Flagsmith发送至Dynatrace。