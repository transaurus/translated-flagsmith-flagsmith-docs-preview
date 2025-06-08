---
title: Testing Push Notifications
---

## 问题背景

精准无误地发送推送通知（以及电子邮件）是出了名的困难。考虑到这些通知在不同开发/测试/生产环境中可能极其敏感且容易出错，加之一旦误发便无法撤回的特性，这个领域足以让最资深的工程师也感到胆战心惊。部署可能向数百万用户发送消息的代码绝非轻松体验。

通过[Firebase FCM](https://firebase.google.com/docs/cloud-messaging)发送推送通知时，虽然可以通过设备令牌向特定设备测试消息，但配置过程技术门槛较高，且这些令牌在应用中并不直观可见。此外，测试营销活动时通常需要向多个设备发送通知，以便在QA、产品负责人、开发人员和营销团队之间验证行为并修复问题。

## 解决方案

我们需要在生产环境全面铺开前，测试营销推送通知的端到端流程。这往往涉及跨营销、产品和工程团队的协作，过程复杂且易出错。提前测试是确保流程可靠的最佳方式。

通过结合使用Flagsmith和Firebase FCM主题可以简化这一过程。当向主题发送推送通知时，Firebase会将该通知投递给订阅该命名主题的所有用户。我们最终会使用这些主题向全体用户发送营销消息，但也可以先利用它们进行团队内部测试。

我们将在Flagsmith中创建名为`fcm_marketing_beta`的功能标志。希望任何启用该标志的用户自动订阅`marketing` FCM主题，后续通过该主题发送消息。

在React Native应用中采用以下代码实现：

```javascript
const isInMarketingBeta = flagsmith.hasFeature('fcm_marketing_beta');

if (isInMarketingBeta) {
 messaging().subscribeToTopic('marketing');
}
```

### 添加单个用户

代码部署后，可以开始为应用用户逐个启用该标志。利用Flagsmith的[用户身份](/basic-features/managing-identities.md)功能，我们既可以为自己的应用账户启用该标志，也可以为团队成员批量启用。

![为用户覆盖功能标志](/img/guides/fcm-user-override.png)

刷新应用后，`messaging().subscribeToTopic('marketing');`代码将执行，将设备添加至FCM主题。

![FCM订阅状态](/img/guides/fcm-subscribed.png)

此时设备已出现在该主题中，可开始用于测试营销消息。

### 通过用户分群批量添加

进一步利用[Flagsmith用户分群](/basic-features/managing-segments.md)功能，可将整个团队纳入该FCM测试组。

创建分群规则（本例通过电子邮件域名特征匹配），并覆盖该分群的`fcm_marketing_beta`标志：

![为分群覆盖功能标志](/img/guides/fcm-segment.png)

现在无需额外代码或部署，即可与整个团队测试这些消息！