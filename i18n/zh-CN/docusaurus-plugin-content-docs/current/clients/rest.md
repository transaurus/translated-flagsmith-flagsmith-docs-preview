---
description: Manage your Feature Flags and Remote Config in your REST APIs.
sidebar_label: REST
sidebar_position: 2
---

# 直接访问API

:::tip

部分API操作需要引用对象的UUID/ID。您可以在账户设置页面启用[JSON视图](#json-view)，这将帮助您获取这些变量。

:::

Flagsmith采用客户端/服务器架构设计。REST API服务器既可供SDK客户端访问，也可供管理前端调用。这种解耦设计意味着您可以根据需要以编程方式访问完整API。

我们提供了[Postman集合](https://www.postman.com/flagsmith/workspace/flagsmith/overview)，您可以通过该集合体验API功能并了解其工作原理。

[![在Postman中运行](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/14712118-a638325a-f1f4-4570-8b4d-fd2841218dfa?action=collection%2Ffork&collection-url=entityId%3D14712118-a638325a-f1f4-4570-8b4d-fd2841218dfa%26entityType%3Dcollection%26workspaceId%3D452554eb-f581-4754-b5b8-0deabdce9f4b#?env%5BFlagsmith%20Environment%5D=W3sia2V5IjoiRmxhZ3NtaXRoIEVudmlyb25tZW50IEtleSIsInZhbHVlIjoiOEt6RVRkRGVNWTd4a3FrU2tZM0dzZyIsImVuYWJsZWQiOnRydWV9LHsia2V5IjoiYmFzZVVybCIsInZhbHVlIjoiaHR0cHM6Ly9hcGkuZmxhZ3NtaXRoLmNvbS9hcGkvdjEvIiwiZW5hYmxlZCI6dHJ1ZX0seyJrZXkiOiJJZGVudGl0eSIsInZhbHVlIjoicG9zdG1hbl91c2VyXzEyMyIsImVuYWJsZWQiOnRydWV9XQ==)

您也可以直接使用[curl](https://curl.haxx.se/)或[httpie](https://httpie.org/)等工具访问API，或者通过我们尚未提供SDK支持的语言客户端进行调用。

## API浏览器

您可以通过Swagger查看API文档：[https://api.flagsmith.com/api/v1/docs/](https://api.flagsmith.com/api/v1/docs/)，或获取OpenAPI格式的[JSON](https://api.flagsmith.com/api/v1/docs/?format=.json)和[YAML](https://api.flagsmith.com/api/v1/docs/?format=.yaml)文件。

## 环境密钥

公开访问的API调用需要在每个请求中提供环境密钥。该密钥通过HTTP标头传递，标头名称为`X-Environment-Key`，值为您可以在Flagsmith管理界面找到的环境密钥。

## Curl示例

以下是使用API实现SDK功能所需调用的两个主要端点。

### 获取环境特征开关

```bash
curl 'https://edge.api.flagsmith.com/api/v1/flags/' -H 'X-Environment-Key: <Your Env Key>'
```

### 提交带有特征的标识并获取特征开关

该命令将通过单次调用完成完整的SDK标识工作流：

1. 惰性创建标识
2. 设置标识特征
3. 获取该标识的特征开关

```bash
curl --request POST 'https://edge.api.flagsmith.com/api/v1/identities/' \
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

## 私有端点

您还可以通过API执行诸如创建新特征开关、环境配置、切换特征状态等操作，实际上管理前端支持的所有功能都可以通过API实现。

要进行身份验证，您可以使用账户关联的令牌。该令牌可在右上角导航面板的"账户"页面中找到。将此令牌用于API调用。例如，要创建新环境：

```bash
curl 'https://api.flagsmith.com/api/v1/environments/' \
    -H 'content-type: application/json' \
    -H 'authorization: Token <TOKEN FROM PREVIOUS STEP>' \
    --data-binary '{"name":"New Environment","project":"<Project ID>"}'
```

您可以通过Swagger REST API在[https://api.flagsmith.com/api/v1/docs/](https://api.flagsmith.com/api/v1/docs/)查看完整的端点列表。

## JSON视图

您可以在账户设置页面启用JSON视图。这将使您能够在仪表板的Flag区域访问相关对象元数据：

![JSON视图](/img/json-view.png)

## 代码示例

以下是一些使用Python通过REST API实现特定操作的简单示例。

### 创建功能

```python
import os

from requests import Session

API_URL = os.environ.get("API_URL", "https://api.flagsmith.com/api/v1")  # update this if self-hosting
PROJECT_ID = os.environ["PROJECT_ID"]  # obtain this from the URL on your dashboard
TOKEN = os.environ["API_TOKEN"]  # obtain this from the account page in your dashboard
FEATURE_NAME = os.environ["FEATURE_NAME"]  # name of the feature to create

session = Session()
session.headers.update({"Authorization": f"Token {TOKEN}"})

create_feature_url = f"{API_URL}/projects/{PROJECT_ID}/features/"
data = {"name": FEATURE_NAME}
response = session.post(create_feature_url, json=data)
```

### 更新环境中功能的值/状态

```python
import json
import os

import requests

TOKEN = os.environ.get("API_TOKEN")  # obtained from Account section in dashboard
ENV_KEY = os.environ.get("ENV_KEY")  # obtained from environment settings in dashboard
BASE_URL = "https://api.flagsmith.com/api/v1"  # update this if self hosting
FEATURE_STATES_URL = f"{BASE_URL}/environments/{ENV_KEY}/featurestates"
FEATURE_NAME = os.environ.get("FEATURE_NAME")

session = requests.Session()
session.headers.update(
    {"Authorization": f"Token {TOKEN}", "Content-Type": "application/json"}
)

# get the existing feature state id based on the feature name
get_feature_states_response = session.get(
    f"{FEATURE_STATES_URL}/?feature_name={FEATURE_NAME}"
)
feature_state_id = get_feature_states_response.json()["results"][0]["id"]

# update the feature state
data = {"enabled": True, "feature_state_value": "new value"}  # `feature_state_value` can be str, int, bool or float
update_feature_state_response = session.patch(
    f"{FEATURE_STATES_URL}/{feature_state_id}/", data=json.dumps(data)
)
```

### 创建分段和分段覆盖

```python
import os

from requests import Session

API_URL = os.environ.get("API_URL", "https://api.flagsmith.com/api/v1")  # update this if self-hosting
SEGMENT_NAME = os.environ["SEGMENT_NAME"]  # define the name of the segment here
PROJECT_ID = os.environ["PROJECT_ID"]  # obtain this from the URL on your dashboard
TOKEN = os.environ["API_TOKEN"]  # obtain this from the account page in your dashboard
FEATURE_ID = os.environ.get("FEATURE_ID")  # obtain this from the URL on your dashboard when viewing a feature
IS_FEATURE_SPECIFIC = os.environ.get("IS_FEATURE_SPECIFIC", default=False) == "True"  # set this to True to create a feature specific segment
ENVIRONMENT_ID = os.environ["ENVIRONMENT_ID"]  # must (currently) be obtained by inspecting the request to /api/v1/environments in the network console

# set these values to create a segment override for the segment, feature, environment combination
ENABLE_FOR_SEGMENT = os.environ.get("ENABLE_FOR_SEGMENT", default=False) == "True"
VALUE_FOR_SEGMENT = os.environ.get("VALUE_FOR_SEGMENT")

SEGMENT_DEFINITION = {
    "name": SEGMENT_NAME,
    "feature": FEATURE_ID if IS_FEATURE_SPECIFIC else None,
    "project": PROJECT_ID,
    "rules": [
        {
            "type": "ALL",
            "rules": [  # add extra rules here to build up 'AND' logic
                {
                    "type": "ANY",
                    "conditions": [  # add extra conditions here to build up 'OR' logic
                        {
                            "property": "my_trait",  # specify a trait key that you want to match on, e.g. organisationId
                            "operator": "EQUAL",  # specify the operator you want to use (one of EQUAL, NOT_EQUAL, GREATER_THAN, LESS_THAN, GREATER_THAN_INCLUSIVE, LESS_THAN_INCLUSIVE, CONTAINS, NOT_CONTAINS, REGEX, PERCENTAGE_SPLIT, IS_SET, IS_NOT_SET)
                            "value": "my-value"  # the value to match against, e.g. 103
                        }
                    ]
                }
            ]
        }
    ]
}

session = Session()
session.headers.update({"Authorization": f"Token {TOKEN}"})

# first let's create the segment
create_segment_url = f"{API_URL}/projects/{PROJECT_ID}/segments/"
create_segment_response = session.post(create_segment_url, json=SEGMENT_DEFINITION)
assert create_segment_response.status_code == 201
segment_id = create_segment_response.json()["id"]

if not any(key in os.environ for key in ("ENABLE_FOR_SEGMENT", "VALUE_FOR_SEGMENT")):
    print("Segment created! Not creating an override as no state / value defined.")
    exit(0)

# next we need to create a feature segment (a flagsmith internal entity)
create_feature_segment_url = f"{API_URL}/features/feature-segments/"
feature_segment_data = {
    "feature": FEATURE_ID,
    "segment": segment_id,
    "environment": ENVIRONMENT_ID
}
create_feature_segment_response = session.post(create_feature_segment_url, json=feature_segment_data)
assert create_feature_segment_response.status_code == 201
feature_segment_id = create_feature_segment_response.json()["id"]

# finally, we can create the segment override
create_segment_override_url = f"{API_URL}/features/featurestates/"
feature_state_data = {
    "feature": FEATURE_ID,
    "feature_segment": feature_segment_id,
    "environment": ENVIRONMENT_ID,
    "enabled": ENABLE_FOR_SEGMENT,
    "feature_state_value": {
        "type": "unicode",
        "string_value": VALUE_FOR_SEGMENT
    }
}
create_feature_state_response = session.post(create_segment_override_url, json=feature_state_data)
assert create_feature_state_response.status_code == 201
```

### 更新分段规则

```python
import os

from requests import Session

API_URL = os.environ.get("API_URL", "https://api.flagsmith.com/api/v1")  # update this if self-hosting
PROJECT_ID = os.environ["PROJECT_ID"]  # obtain this from the URL on your dashboard
TOKEN = os.environ["API_TOKEN"]  # obtain this from the account page in your dashboard
SEGMENT_ID = os.environ.get("SEGMENT_ID")  # obtain this from the URL on your dashboard when viewing a segment

SEGMENT_RULES_DEFINITION = {
    "rules": [
        {
            "type": "ALL",
            "rules": [
                {
                    "type": "ANY",
                    "conditions": [  # add as many conditions here to build up a segment
                        {
                            "property": "my_trait",  # specify a trait key that you want to match on, e.g. organisationId
                            "operator": "EQUAL",  # specify the operator you want to use (one of EQUAL, NOT_EQUAL, GREATER_THAN, LESS_THAN, GREATER_THAN_INCLUSIVE, LESS_THAN_INCLUSIVE, CONTAINS, NOT_CONTAINS, REGEX, PERCENTAGE_SPLIT, IS_SET, IS_NOT_SET)
                            "value": "my-value"  # the value to match against, e.g. 103
                        }
                    ]
                }
            ]
        }
    ]
}

session = Session()
session.headers.update({"Authorization": f"Token {TOKEN}"})

update_segment_url = f"{API_URL}/projects/{PROJECT_ID}/segments/{SEGMENT_ID}/"
session.patch(update_segment_url, json=SEGMENT_RULES_DEFINITION)
```

### 遍历身份标识

有时遍历您的身份标识以检查分段覆盖等情况会很有用。

```python
import os

import requests

TOKEN = os.environ.get("API_TOKEN")  # obtained from Account section in dashboard
ENV_KEY = os.environ.get("ENV_KEY")  # obtained from Environment settings in dashboard
BASE_URL = "https://api.flagsmith.com/api/v1"  # update this if self hosting
IDENTITIES_PAGE_URL = f"{BASE_URL}/environments/{ENV_KEY}/edge-identities/?page_size=20"

session = requests.Session()
session.headers.update(
    {"Authorization": f"Token {TOKEN}", "Content-Type": "application/json"}
)

# get the existing feature state id based on the feature name
page_of_identities = session.get(f"{IDENTITIES_PAGE_URL}")
print(page_of_identities.json())

for identity in page_of_identities.json()['results']:
    print(str(identity))
    IDENTITY_UUID = identity['identity_uuid']
    IDENTITY_URL = f"{BASE_URL}/environments/{ENV_KEY}/edge-identities/{IDENTITY_UUID}/edge-featurestates/all/"
    identity_data = session.get(f"{IDENTITY_URL}")
    print(identity_data.json())
```