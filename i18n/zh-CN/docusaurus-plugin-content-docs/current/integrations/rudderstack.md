---
title: RudderStack Integration
description: Integrate Flagsmith with RudderStack
sidebar_label: RudderStack
hide_title: true
---

![RudderStack](/img/integrations/rudderstack/rudderstack-logo.svg)

您可以将Flagsmith与RudderStack集成。将身份特征标志状态发送至RudderStack进行下游分析。

## 集成设置

1. 获取RudderStack项目的API密钥。添加_HTTP API_数据源，并记录`写入密钥`。
2. 记录RudderStack的_数据平面URL_。
3. 进入Flagsmith项目，点击"集成"。添加RudderStack集成。
4. 将步骤1中的`写入密钥`粘贴至Flagsmith的API密钥字段。选择需要发送事件的环境，点击保存。
5. Flagsmith SDK调用`获取身份标志`接口时，会将该用户的完整标志评估结果发送至RudderStack。

## 工作原理

:::tip

对于包含远程配置值的特征标志，当标志处于`启用`状态时，Flagsmith会将其值传递给RudderStack。若标志无远程配置值，则仅传递布尔状态。

:::

身份特征标志值将被传递至RudderStack。

以下是Flagsmith中的示例用户：

![RudderStack](/img/integrations/segment/segment-integration-2.png)

当我们调用Flagsmith API获取该用户的标志时：

```bash
curl 'https://edge.api.flagsmith.com/api/v1/identities/?identifier=development_user_123456' \
  -H 'x-environment-key: 8KzETdDeMY7xkqkSkY3Gsg'
```

在RudderStack控制台中，可以查看到已发送至平台的用户及其标志数据。

![RudderStack](/img/integrations/rudderstack/rudderstack-integration-1.png)

## 使用场景

集成完成后，您可以根据用户接触到的特征标志对RudderStack身份进行细分。这意味着可以通过Flagsmith分群驱动A/B测试，数据将自动同步至RudderStack。

## 集成注意事项

需确保两个平台的用户标识方式一致。Flagsmith的`身份ID`必须与RudderStack的`user_id`保持相同。