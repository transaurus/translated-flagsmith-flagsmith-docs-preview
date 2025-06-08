---
title: System Administration
---

## 审计日志

Flagsmith管理后台的每个操作都会被追踪记录。这使您能够轻松追溯功能开关、用户身份和细分群体随时间推移的状态变更历史。

您可以在Flagsmith应用中查看审计日志，并通过筛选快速定位所需信息。

您还可以通过[审计日志Webhook](#audit-log-webhooks)将审计日志实时同步到自有基础设施。

## 审计日志Webhook

通过审计日志Webhook，您可以将组织的审计日志流式传输到自有基础设施。这对合规审查或与本地CI/CD系统进行交叉验证非常有用。

```json
{
 "created_date": "2020-02-23T17:30:57.006318Z",
 "log": "New Flag / Remote Config created: my_feature",
 "author": {
  "id": 3,
  "email": "user@domain.com",
  "first_name": "Kyle",
  "last_name": "Johnson"
 },
 "environment": null,
 "project": {
  "id": 6,
  "name": "Project name",
  "organisation": 1
 },
 "related_object_id": 6,
 "related_object_type": "FEATURE"
}
```

## Webhook

通过Webhook功能，您可以将Flagsmith中的事件推送到自有基础设施。Webhook在环境级别进行管理，可在环境设置页面配置。

![图片](/img/add-webhook.png)

当前以下事件会触发Webhook动作：

- 创建功能开关（事件类型为`FLAG_UPDATED`）
- 更新环境中的功能开关值/状态（事件类型为`FLAG_UPDATED`）
- 为用户身份覆盖功能开关（事件类型为`FLAG_UPDATED`）
- 为细分群体覆盖功能开关（事件类型为`FLAG_UPDATED`）

每个环境可配置任意数量的Webhook端点。Webhook管理入口位于环境设置页面。

Webhook的典型使用场景包括在服务器环境中本地缓存功能开关状态。

每个事件都会向该环境配置的所有Webhook端点发送HTTP POST请求，请求体包含以下载荷结构：

```json
{
 "data": {
  "changed_by": "Ben Rometsch",
  "new_state": {
   "enabled": true,
   "environment": {
    "id": 23,
    "name": "Development"
   },
   "feature": {
    "created_date": "2021-02-10T20:03:43.348556Z",
    "default_enabled": false,
    "description": "Show markdown in a butter bar for certain users",
    "id": 7168,
    "initial_value": null,
    "name": "butter_bar",
    "project": {
     "id": 12,
     "name": "Flagsmith Website"
    },
    "type": "CONFIG"
   },
   "feature_segment": null,
   "feature_state_value": "<strong>\nYou are using the develop environment.\n</strong>",
   "identity": null,
   "identity_identifier": null
  },
  "previous_state": {
   "enabled": false,
   "environment": {
    "id": 23,
    "name": "Development"
   },
   "feature": {
    "created_date": "2021-02-10T20:03:43.348556Z",
    "default_enabled": false,
    "description": "Show markdown in a butter bar for certain users",
    "id": 7168,
    "initial_value": null,
    "name": "butter_bar",
    "project": {
     "id": 12,
     "name": "Flagsmith Website"
    },
    "type": "CONFIG"
   },
   "feature_segment": null,
   "feature_state_value": "<strong>\nYou are using the develop environment.\n</strong>",
   "identity": null,
   "identity_identifier": null
  },
  "timestamp": "2021-06-18T07:50:26.595298Z"
 },
 "event_type": "FLAG_UPDATED"
}
```

## Webhook签名

当设置Webhook密钥后，Flagsmith会使用该密钥为每个载荷生成哈希签名。该签名会通过X-Flagsmith-Signature请求头发送，您需要在接收端进行验证。

### 签名验证

使用SHA256哈希函数计算HMAC。将请求体（原始utf-8编码字符串）作为消息，密钥（utf8编码）作为Key。以下是Python示例：

```python
import hmac

secret = "my shared secret"

expected_signature = hmac.new(
    key=secret.encode(),
    msg=request_body,
    digestmod=hashlib.sha256,
).hexdigest()

received_signature = request["headers"]["x-flagsmith-signature"]
hmac.compare_digest(expected_signature, received_signature) is True
```