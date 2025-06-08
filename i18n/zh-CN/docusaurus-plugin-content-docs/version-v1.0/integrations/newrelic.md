---
title: New Relic Analytics Integration
description: Integrate Flagsmith with New Relic
sidebar_label: New Relic
hide_title: true
---

![Image](/img/integrations/newrelic/newrelic-logo.svg)

您可以将Flagsmith与New Relic集成。将Flagsmith中的功能标志变更事件发送至New Relic事件流。

## 集成设置

1. 登录New Relic并导航至需要集成的APM项目，记录下`App ID`。
2. 生成New Relic API密钥：账户设置 > API密钥 > 创建密钥，密钥类型选择`用户`。
3. 进入Flagsmith项目，点击"集成"添加New Relic集成。输入New Relic的`API密钥`和`App ID`。API URL需填写：
   - 美国数据中心：`https://api.newrelic.com/`
   - 欧盟数据中心：`https://api.eu.newrelic.com/`
4. 点击保存。

功能标志变更事件将以`部署`形式发送至New Relic平台。