---
title: Dynatrace Integration
sidebar_label: Dynatrace
hide_title: true
---

![Dynatrace](/img/integrations/dynatrace/dynatrace-logo.svg)

您可以将Flagsmith与Dynatrace集成，将Flagsmith中的功能标志变更事件发送至Dynatrace事件流。

## 集成设置

1. 登录Dynatrace并创建具有以下权限的新访问令牌：
   - `API v2 权限范围`：`事件摄取`
2. 进入Flagsmith > 集成 > Dynatrace > 添加集成
3. 选择需要跟踪的`Flagsmith环境`
4. `基础URL`取决于您的Dynatrace部署方式：
   - 托管版：`https://{您的域名}/e/{您的环境ID}/`
   - SaaS版：`https://{您的环境ID}.live.dynatrace.com/`
   - 环境ActiveGate：`https://{您的ActiveGate域名}/e/{您的环境ID}/`
5. `API密钥`即步骤1中创建的访问令牌
6. `实体选择器`用于定义功能标志变更事件关联的Dynatrace实体
   [参考Dynatrace文档](https://www.dynatrace.com/support/help/dynatrace-api/environment-api/entity-v2/entity-selector)获取详细信息

## 工作原理

完成设置后，尝试在配置的环境中修改功能标志值。随后在`实体选择器`定义的任一Dynatrace实体中，您将在事件面板看到部署事件。

![Dynatrace事件](/img/integrations/dynatrace/dynatrace-events-panel.png)