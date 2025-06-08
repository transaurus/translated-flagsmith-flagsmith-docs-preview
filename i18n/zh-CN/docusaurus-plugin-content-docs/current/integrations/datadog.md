---
title: Datadog Integration
sidebar_label: Datadog
hide_title: true
---

import ReactPlayer from 'react-player'

![Datadog](/img/integrations/datadog/datadog-logo.svg)

您可以通过两种方式将Flagsmith与Datadog集成：

- [1. 将Flagsmith集成至Datadog仪表盘](#1-integrate-flagsmith-into-your-datadog-dashboard)
- [2. 向Datadog发送功能标志变更事件](#2-send-flag-change-events-to-datadog)

## 1. 将Flagsmith集成至Datadog仪表盘

该集成允许您在Datadog仪表盘中添加Flagsmith小组件，无需离开Datadog应用即可查看和管理功能标志。

![Datadog仪表盘小组件](/img/integrations/datadog/datadog-dashboard-widget.png)

下方视频将逐步演示如何添加该集成：

<ReactPlayer
    playing
    controls
    width="100%"
    height="460px"
    url='https://getleda.wistia.com/medias/76558s9yj7' />

## 2. 向Datadog发送功能标志变更事件

第二种集成方式允许您将Flagsmith中的功能标志变更事件发送至Datadog事件流。

![Datadog](/img/integrations/datadog/datadog-3.png)

1. 登录Datadog并进入组织设置 > 访问 > API
2. 生成新的API密钥
3. 在Flagsmith中添加Datadog API密钥（集成 > 添加Datadog集成）
4. 在Flagsmith中配置Datadog基础URL（可选择`https://api.datadoghq.com/`或`https://api.datadoghq.eu/`）

![Datadog](/img/integrations/datadog/datadog-1.png)

![Datadog](/img/integrations/datadog/datadog-2.png)

功能标志变更事件现在将自动发送至Datadog。