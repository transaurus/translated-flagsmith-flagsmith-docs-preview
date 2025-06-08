---
title: Datadog Integration
sidebar_label: Datadog
hide_title: true
---

![Datadog](/img/integrations/datadog/datadog-logo.svg)

您可以将Flagsmith与Datadog集成，将功能标志变更事件从Flagsmith发送至Datadog事件流。

## 集成设置

![Datadog](/img/integrations/datadog/datadog-3.png)

1. 登录Datadog并进入 集成 > API
2. 生成一个API密钥
3. 将Datadog API密钥填入Flagsmith（集成 > 添加Datadog集成）
4. 在Flagsmith中配置Datadog URL（可选`https://api.datadoghq.com/`或`https://api.datadoghq.eu/`）

![Datadog](/img/integrations/datadog/datadog-1.png)

![Datadog](/img/integrations/datadog/datadog-2.png)

功能标志变更事件现在将自动推送至Datadog。