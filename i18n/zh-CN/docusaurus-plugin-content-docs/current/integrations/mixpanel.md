---
title: Mixpanel Analytics Integration
description: Integrate Flagsmith with Mixpanel
sidebar_label: Mixpanel
hide_title: true
---

![Mixpanel](/img/integrations/mixpanel/mixpanel-logo.svg)

您可以将Flagsmith与Mixpanel集成。该集成会自动将已识别用户的特征标志状态发送至Mixpanel，用于群体分析、A/B测试等功能。具体流程如下：

## 集成设置

1. 从Mixpanel管理项目页面获取项目环境ID（项目设置 > 项目令牌）
2. 在Flagsmith中添加项目令牌（集成 > 添加Mixpanel集成）
3. Flagsmith SDK调用`获取身份特征标志`接口时，会将该用户的所有特征标志评估结果作为[用户档案](https://developer.mixpanel.com/reference/user-profiles)发送至Mixpanel

## 工作原理

:::tip

对于包含远程配置值的特征标志，当标志处于`启用`状态时，Flagsmith会将其值传递给Mixpanel。若标志无远程配置值，则仅传递布尔状态。

:::

身份特征标志值会被传递至Mixpanel。

以下是Flagsmith中的演示用户：

![Mixpanel](/img/integrations/mixpanel/mixpanel-integration-2.png)

如果我们调用Flagsmith API获取该用户的特征标志：

```bash
curl 'https://edge.api.flagsmith.com/api/v1/identities/?identifier=development_user_123456' \
  -H 'x-environment-key: 8KzETdDeMY7xkqkSkY3Gsg'
```

然后在Mixpanel仪表盘中，您可以看到已发送至Mixpanel平台的用户及其特征标志数据。

![Mixpanel](/img/integrations/mixpanel/mixpanel-integration-1.png)

## 使用场景

集成完成后，您可以根据用户接触到的特征标志对Mixpanel中的身份进行分群。这意味着您可以运行由Flagsmith分群驱动的A/B测试，并让数据自动同步至Mixpanel。

## 集成注意事项

必须在两个平台使用相同方式识别用户。Flagsmith的`身份ID`需与Mixpanel的`identity`保持一致。

## 底层实现原理

每次`身份`向Flagsmith API请求特征标志时，Flagsmith会向`https://api.mixpanel.com/engage#profile-set`发送包含以下JSON负载的`POST`请求：

```json
{
 "$token": "<YOUR_MIXPANEL_PROJECT_TOKEN>",
 "$distinct_id": "<FLAGSMITH_IDENTITY_ID>",
 "$set": {
  "<FLAG_1_ID>": "<FLAG_1_STATE>",
  "<FLAG_2_ID>": "<FLAG_2_STATE>",
  "...": "..."
 }
}
```