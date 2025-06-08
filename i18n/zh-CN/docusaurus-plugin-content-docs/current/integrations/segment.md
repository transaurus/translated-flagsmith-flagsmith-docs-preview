---
title: Segment Integration
description: Integrate Flagsmith with Segment
sidebar_label: Segment
hide_title: true
---

![Segment](/img/integrations/segment/segment-logo.svg)

您可以将Flagsmith与Segment集成。将用户身份特征标志状态发送至Segment平台进行下游分析。

## 集成设置

1. 获取Segment项目的API密钥。添加_HTTP API_数据源，并记录`写入密钥`。
2. 进入Flagsmith项目，点击"集成"后添加Segment集成。
3. 将步骤1中的`写入密钥`粘贴至Flagsmith的API密钥字段。选择需要发送事件的Flagsmith环境，点击保存。
4. Flagsmith SDK调用`获取身份特征标志`接口时，会将该用户的所有特征标志评估结果发送至Segment平台的[`Identify`端点](https://segment.com/docs/connections/spec/identify/)

## 工作原理

:::tip

对于包含远程配置值的特征标志，当标志处于`启用`状态时，Flagsmith会将其配置值传递给Segment。若标志没有远程配置值，则仅传递布尔状态。

:::

用户身份特征标志值将被传递至Segment平台。

以下是Flagsmith中的演示用户：

![Segment](/img/integrations/segment/segment-integration-2.png)

当我们调用Flagsmith API获取该用户的特征标志时：

```bash
curl 'https://edge.api.flagsmith.com/api/v1/identities/?identifier=development_user_123456' \
  -H 'x-environment-key: 8KzETdDeMY7xkqkSkY3Gsg'
```

在Segment仪表盘中可以查看到已发送至平台的相关用户及特征标志数据。

![Segment](/img/integrations/segment/segment-integration-1.png)

## 使用场景

完成集成配置后，您可以根据用户接触到的特征标志对Segment身份进行细分。这意味着可以通过Flagsmith分群驱动AB测试，测试数据将自动同步至Segment平台。

## 集成注意事项

必须在两个平台使用相同方式识别用户。Flagsmith的`身份ID`必须与Segment的`user_id`保持一致。