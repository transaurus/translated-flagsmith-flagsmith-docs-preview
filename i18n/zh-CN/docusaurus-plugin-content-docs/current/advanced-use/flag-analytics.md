---
title: Flag Analytics
---

## 概述

功能标志分析允许您追踪Flagsmith SDK中各功能标志被评估的频率。

要查看特定标志的分析数据，请浏览至相应环境并点击单个标志进行编辑。

![Image](/img/flag-analytics.png)

当需要从Flagsmith中移除标志时，标志分析功能尤为实用。通常而言，在功能标志已全面上线且团队对其在生产环境运行状态感到满意后，即可从代码库和平台中移除这些标志。

当您从代码库中移除标志评估逻辑后，可通过分析数据确认该标志的所有引用是否已彻底清除，从而确保从Flagsmith平台删除该标志不会引发意外问题。

标志分析也有助于识别集成问题。某些情况下，代码中可能出现错误导致标志被重复无效评估，分析数据可帮助定位这类异常情况。

## SDK支持

We currently support Flag Analytics with the following SDKs:

- [JavaScript及React/React Native](/clients/javascript/)。当前版本`1.2.2`及以上已支持该功能。

我们正在为其他SDK集成标志分析功能，同时欢迎提交Pull Request！

## 工作原理

每当SDK中对标志进行评估时（通常通过调用类似`flagsmith.hasFeature("myCoolFeature")`的方法），SDK会记录该标志名称及评估次数。

每隔n秒（当前JavaScript SDK设置为10秒），SDK会向Flagsmith API发送包含已评估标志列表及其次数的消息。若该时间段内无标志被评估，则不发送消息。