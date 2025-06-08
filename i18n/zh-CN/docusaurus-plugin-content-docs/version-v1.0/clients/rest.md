---
description: Manage your Feature Flags and Remote Config in your REST APIs.
sidebar_label: REST
sidebar_position: 1
---

# 直接访问API

Flagsmith采用客户端/服务器架构设计。REST API服务器不仅可通过管理前端访问，也支持SDK客户端调用。这种解耦设计使您能够按需以编程方式访问完整API功能。

我们提供[Postman集合](https://www.postman.com/flagsmith/workspace/flagsmith/overview)，您可通过该集合体验API操作并熟悉其工作原理。

[![在Postman中运行](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/14712118-a638325a-f1f4-4570-8b4d-fd2841218dfa?action=collection%2Ffork&collection-url=entityId%3D14712118-a638325a-f1f4-4570-8b4d-fd2841218dfa%26entityType%3Dcollection%26workspaceId%3D452554eb-f581-4754-b5b8-0deabdce9f4b#?env%5BFlagsmith%20Environment%5D=W3sia2V5IjoiRmxhZ3NtaXRoIEVudmlyb25tZW50IEtleSIsInZhbHVlIjoiOEt6RVRkRGVNWTd4a3FrU2tZM0dzZyIsImVuYWJsZWQiOnRydWV9LHsia2V5IjoiYmFzZVVybCIsInZhbHVlIjoiaHR0cHM6Ly9hcGkuZmxhZ3NtaXRoLmNvbS9hcGkvdjEvIiwiZW5hYmxlZCI6dHJ1ZX0seyJrZXkiOiJJZGVudGl0eSIsInZhbHVlIjoicG9zdG1hbl91c2VyXzEyMyIsImVuYWJsZWQiOnRydWV9XQ==)

您也可以使用[curl](https://curl.haxx.se/)或[httpie](https://httpie.org/)等工具直接调用API，或者通过我们尚未提供官方SDK的编程语言客户端进行访问。

## API浏览器

可通过Swagger查看API文档：[https://api.flagsmith.com/api/v1/docs/](https://api.flagsmith.com/api/v1/docs/)，或获取OpenAPI格式的[JSON](https://api.flagsmith.com/api/v1/docs/?format=.json)及[YAML](https://api.flagsmith.com/api/v1/docs/?format=.yaml)文件。

## 环境密钥

公开API调用需在每次请求中提供环境密钥。该密钥需通过HTTP头部传递，头部名称为`X-Environment-Key`，值为您可在Flagsmith管理界面找到的环境API密钥。

### Curl示例

```bash
curl 'https://api.flagsmith.com/api/v1/flags/' -H 'X-Environment-Key: TijpMX6ajA7REC4bf5suYg'
```

### httpie示例

```bash
http GET 'https://api.flagsmith.com/api/v1/flags/' 'X-Environment-Key':'TijpMX6ajA7REC4bf5suYg'
```

## 私有端点

通过API您可以执行包括创建新功能开关、环境配置、切换功能状态等所有管理前端支持的操作。

认证需使用账户关联的令牌。该令牌可在右上角导航面板的"账户"页面获取。以下示例展示如何通过API创建新环境：

```bash
curl 'https://api.flagsmith.com/api/v1/environments/' \
    -H 'content-type: application/json' \
    -H 'authorization: Token <TOKEN FROM PREVIOUS STEP>' \
    --data-binary '{"name":"New Environment","project":"<Project ID>"}'
```

完整端点列表可通过Swagger REST API查看：[https://api.flagsmith.com/api/v1/docs/](https://api.flagsmith.com/api/v1/docs/)。

## 常用SDK端点

### 提交带特征的标识并获取功能开关

以下`curl`命令可一次性完成完整的SDK工作流程：

1. 创建用户身份
2. 为该身份设置特征值
3. 获取该身份对应的功能开关配置

```bash
curl --request POST 'https://api.flagsmith.com/api/v1/identities/' \
--header 'X-Environment-Key: <Your Env Key>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "identifier":"identifier_5",
    "traits": [
        {
            "trait_key": "my_trait_key",
            "trait_value": 123.5
        },
        {
            "trait_key": "my_other_key",
            "trait_value": true
        }
    ]
}'
```