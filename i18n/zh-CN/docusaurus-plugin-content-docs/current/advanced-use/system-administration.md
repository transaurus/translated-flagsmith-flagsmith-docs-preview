---
title: System Administration
---

## 身份验证

Flagsmith 仪表板支持多种身份验证登录方式。

### 开源版本

- 用户名和密码
- Github OAuth（需配置）
- Google OAuth（需配置）

### SaaS免费版及初创计划

- 用户名和密码
- Github OAuth（需配置）
- Google OAuth（需配置）

### SaaS扩容计划

该计划支持启用双重认证(2FA)。

- 用户名和密码
- Github OAuth（需配置）
- Google OAuth（需配置）
- SAML

### 企业计划（SaaS/本地部署/私有云）

该计划支持启用双重认证(2FA)。

企业版还可限制组织仅通过特定认证方式登录。

- 用户名和密码
- Github OAuth（需配置）
- Google OAuth（需配置）
- SAML
- Okta
- LDAP
- Microsoft ADFS

## 组织的Terraform API密钥

您可以在组织设置页面创建Terraform API密钥，当前用于我们的[Terraform Provider](../integrations/terraform.md)。可创建具有组织全局访问权限的密钥，以满足Terraform运行所需的端点调用。

## 禁止客户端SDK设置用户特征

某些场景下可能需要禁止客户端SDK设置用户特征。例如：当您通过特征`plan=silver`控制功能开关时，恶意用户可能通过客户端SDK将特征篡改为`plan=gold`来解锁未付费功能。

您可通过关闭"允许客户端SDK设置用户特征"选项来防范（默认开启）。关闭后，必须使用[服务端SDK及服务端密钥](../clients/overview.md)才能写入特征。

此为按环境配置的选项。

## API使用统计

Flagsmith会追踪SDK发起的API调用并存储统计数据。您可以在组织设置页面的**使用情况**中查看，也可下钻到具体项目和环境。系统追踪以下请求类型：

1. 获取功能开关
2. 获取身份特征开关
3. 设置身份特征
4. 获取环境文档（用于本地评估型SDK）

![API使用统计](/img/api-usage.png)

## 审计日志

Flagsmith管理端的所有操作都会被追踪记录，便于追溯功能开关、身份特征和用户分群的历史变更。

您可在应用内查看审计日志，并通过筛选定位所需信息。

还支持通过[审计日志Webhook](#audit-log-webhooks)将日志流式传输至自有基础设施。

## 审计日志Webhook

通过审计日志Webhook可将组织级审计日志流式传输至自有基础设施，适用于合规审查或与本地CI/CD系统对接。

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

通过Webhook可将Flagsmith事件推送至自有基础设施。Webhook在环境级别进行管理，配置入口位于环境设置页面。

![图示](/img/add-webhook.png)

当前以下事件会触发Web Hook动作：

- 创建功能开关（事件类型为`FLAG_UPDATED`）
- 在环境中更新功能开关的值/状态（事件类型为`FLAG_UPDATED`）
- 为特定身份覆盖功能开关（事件类型为`FLAG_UPDATED`）
- 为特定用户群组覆盖功能开关（事件类型为`FLAG_UPDATED`）

您可以为每个环境定义任意数量的Web Hook端点。Web Hook可通过环境设置页面进行管理。

Web Hook的典型使用场景是：如果您希望在服务器环境中本地缓存功能开关状态。

每个事件都会向该环境中定义的所有Web Hook端点发送HTTP POST请求，请求体包含以下载荷：

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
    "description": "Show html in a butter bar for certain users",
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
    "description": "Show html in a butter bar for certain users",
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

## Webhook签名验证

当设置Webhook密钥后，Flagsmith会使用该密钥为每个载荷创建哈希签名。该签名会通过X-Flagsmith-Signature请求头随每次请求发送，您需要在接收端进行验证。

### 签名验证方法

使用SHA256哈希函数计算HMAC。将请求体（原始utf-8编码字符串）作为消息，密钥（utf8编码）作为密钥。以下是Python示例：

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

## 环境标识横幅

您可以在每个环境设置页面中为环境配置彩色标识横幅。这有助于您在切换功能开关前识别敏感环境！

![环境标识横幅](/img/environment-banner.png)