---
title: Flag Analytics
---

:::note

此功能目前可在以下 SDK 中使用：

- Javascript
- iOS
- Flutter
- Python

我们正在[努力](https://github.com/Flagsmith/flagsmith/issues/293)将此功能添加到其他 SDK 中。

:::

## 概述

功能标记分析允许您追踪 Flagsmith SDK 中各个功能标记被评估的频率。

要查看特定标记的分析数据，请浏览至相关环境并点击单个标记进行编辑。

![Image](/img/flag-analytics.png)

功能标记分析在移除 Flagsmith 中的标记时非常有用。通常，一旦标记已全面推出且团队对其在生产环境中的运行感到满意后，就可以从代码库和平台中移除这些标记。

当您从代码库中移除标记评估代码后，可以通过分析数据确认该标记的所有引用已被清除，且从 Flagsmith 中移除该标记不会引发意外问题。

该分析功能也有助于识别集成问题。有时代码中可能出现错误导致标记被多次不必要地评估，分析数据能帮助定位这类情况。

## SDK 支持

当前支持功能标记分析的 SDK 包括：

- [Javascript 和 React/React Native](/clients/javascript/)。当前版本 `1.2.2` 及以上已启用该功能。

我们正在为其他 SDK 集成此分析功能，同时欢迎提交 Pull Request！

## 工作原理

每当 SDK 中评估一个标记时（通常是通过调用类似 `flagsmith.hasFeature("myCoolFeature")` 的方法），SDK 会记录该标记名称及评估次数。

每隔 n 秒（当前 JS SDK 设置为 10 秒），SDK 会向 Flagsmith API 发送包含已评估标记列表及其次数的消息。若该时间段内无标记被评估，则不发送消息。