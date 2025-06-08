---
title: Webhook Analytics Integration
description: Integrate Flagsmith with your own infrastructure
sidebar_label: Webhook
hide_title: true
---

![Webhook](/img/integrations/webhook/webhook-logo.svg)

您可以通过Webhook集成将Flagsmith与自有数据仓库对接。该集成会自动将已识别用户的特征标志状态以Webhook形式发送至您指定的URL，用于队列分析、A/B测试等场景。具体流程如下：

## 集成配置

1. 编写一个能接收[下方JSON Schema](#webhook-json-schema)的端点
2. 在Flagsmith中添加集成，提供上一步创建的URL
3. 可额外提供Secret密钥，该密钥将被[哈希处理后包含在HTTP头信息](#webhook-signature)中，用于验证Webhook请求确实来自Flagsmith
4. 所有通过Flagsmith SDK调用`获取用户特征标志`接口时，系统会将该用户的完整特征标志评估结果、用户属性和所属用户组信息发送至您的Webhook URL

## Webhook JSON数据规范

Flagsmith会向您提供的Webhook URL发送`POST`请求，请求体包含以下数据结构：

```json
{
 "flags": [
  {
   "enabled": false,
   "environment": 2,
   "feature": {
    "created_date": "2022-02-04T14:57:39.200798Z",
    "default_enabled": false,
    "description": null,
    "id": 1,
    "initial_value": null,
    "name": "12e12e",
    "type": "STANDARD"
   },
   "feature_segment": null,
   "feature_state_value": null,
   "id": 2,
   "identity": null
  },
  {
   "enabled": true,
   "environment": 2,
   "feature": {
    "created_date": "2022-02-04T14:57:44.244575Z",
    "default_enabled": true,
    "description": null,
    "id": 2,
    "initial_value": null,
    "name": "gggg",
    "type": "STANDARD"
   },
   "feature_segment": null,
   "feature_state_value": null,
   "id": 4,
   "identity": null
  }
 ],
 "identity": "user_test",
 "segments": [
  {
   "id": 1,
   "member": true,
   "name": "test_segment"
  }
 ],
 "traits": [
  {
   "id": 4,
   "trait_key": "222",
   "trait_value": 333
  },
  {
   "id": 5,
   "trait_key": "aaa",
   "trait_value": "bbb"
  }
 ]
}
```

## 使用场景

完成集成配置后，您可以根据用户接触到的特征标志及其所属用户组，对数据仓库中的用户进行分群分析。这使您能够通过Flagsmith增强数据仓库中的信息维度。

## Webhook签名验证

当设置Webhook密钥后，Flagsmith会为每个请求生成哈希签名。该签名会通过X-Flagsmith-Signature请求头传递，您需要在接收端进行验证

### 签名验证方法

使用SHA256哈希函数计算HMAC：以原始请求体（UTF-8编码字符串）作为消息，密钥（UTF-8编码）作为密钥。以下是Python示例：

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