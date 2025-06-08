---
title: Amplitude Integration
sidebar_label: Amplitude
hide_title: true
---

![Amplitude](/img/integrations/amplitude/amplitude-logo.svg)

您可以将Flagsmith与Amplitude集成。通过连接Flagsmith平台与Amplitude，您可以自动化AB测试。具体流程如下：

## 集成设置

1. 从Amplitude项目页面获取API密钥（设置 > 项目 > 您的项目 > 常规选项卡）
2. 将Amplitude API密钥添加到Flagsmith中（集成 > 添加Amplitude集成）
3. Flagsmith SDK生成的所有对"获取身份特征标志"端点的API调用，都会将该用户的完整特征标志评估结果发送至Amplitude的[身份识别端点](https://developers.amplitude.com/docs/identify-api)

## 工作原理

:::tip

对于包含远程配置值的特征标志，如果标志处于"启用"状态，Flagsmith会将标志值传递给Amplitude。如果标志没有远程配置值，Flagsmith则仅传递标志的布尔状态。

:::

身份特征标志值会被传递至Amplitude。

以下是Flagsmith中的演示用户：

![Amplitude](/img/integrations/amplitude/amplitude-integration-2.png)

如果我们调用Flagsmith API获取该用户的特征标志：

```bash
curl 'https://api.flagsmith.com/api/v1/identities/?identifier=development_user_123456' \
  -H 'x-environment-key: 8KzETdDeMY7xkqkSkY3Gsg'
```

然后在Amplitude仪表板中查看，可以看到用户信息及已发送至Amplitude平台的标志数据。

![Amplitude](/img/integrations/amplitude/amplitude-integration-1.png)

## 使用场景

完成集成设置后，您可以根据用户接触到的特征标志对Amplitude身份进行细分。这意味着您可以运行由Flagsmith细分驱动的AB测试，并让数据自动显示在Amplitude中。

## 集成注意事项

必须在两个平台上使用相同方式识别用户。Flagsmith的"身份ID"必须与Amplitude的"user_id"保持一致。