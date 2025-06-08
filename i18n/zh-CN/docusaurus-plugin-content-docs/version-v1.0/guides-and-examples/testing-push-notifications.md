---
title: Testing Push Notifications
---

## 问题背景

精准无误地发送推送通知（及电子邮件）是业界公认的难题。考虑到这些通知在不同开发/测试/生产环境中极易出错，且一旦误发无法撤回，即便是经验最丰富的工程师也会对此心生畏惧。向数百万用户推送消息的代码部署过程，绝非轻松体验。

通过[Firebase FCM](https://firebase.google.com/docs/cloud-messaging)发送推送通知时，虽然可以通过设备令牌向特定设备测试消息，但配置过程技术门槛较高，且这些令牌在应用中不易获取。此外，测试营销活动时通常需要向多个设备发送通知，以便在QA、产品负责人、开发人员和营销团队之间验证行为并修复缺陷。

## 解决方案

我们需要在生产环境全面推送前，测试营销通知的端到端流程。这往往涉及跨营销、产品和工程团队的协作，过程复杂且易出错。提前测试是确保流程可靠的最佳实践。

通过结合使用Flagsmith和Firebase FCM主题可简化此流程。当向主题发送推送通知时，Firebase会将该通知投递给订阅该主题的所有用户。我们最终会使用这些主题向全体用户推送营销消息，但也可以先用于团队内部测试。

我们将在Flagsmith中创建名为`fcm_marketing_beta`的功能开关。任何启用此开关的用户都将自动订阅`marketing` FCM主题，后续消息将通过该主题推送。

在React Native应用中采用以下实现代码：

```javascript
const isInMarketingBeta = flagsmith.hasFeature('fcm_marketing_beta');

if (isInMarketingBeta) {
 messaging().subscribeToTopic('marketing');
}
```

### 添加单个用户

代码部署后，可通过Flagsmith的[用户标识](/basic-features/managing-identities.md)功能为指定用户启用该开关。在应用界面为团队成员逐个启用此功能。

![为用户覆盖功能开关](/img/guides/fcm-user-override.png)

应用刷新后将执行`messaging().subscribeToTopic('marketing');`代码，完成FCM主题订阅。

![FCM订阅状态](/img/guides/fcm-subscribed.png)

设备成功加入主题后，即可通过该主题发送测试营销消息。

### 通过用户分群批量添加

利用[Flagsmith用户分群](/basic-features/managing-segments.md)功能，可将整个团队纳入FCM测试组。

创建分群并设置匹配规则（本例使用邮箱域名特征匹配），然后为该分群覆盖`fcm_marketing_beta`开关：

![为分群覆盖功能开关](/img/guides/fcm-segment.png)

现在无需额外编码或部署，即可实现全团队成员的营销消息测试！