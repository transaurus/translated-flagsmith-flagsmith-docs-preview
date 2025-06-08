---
title: Flagsmith Python SDK
sidebar_label: Python
description: Manage your Feature Flags and Remote Config in your Python applications.
slug: /clients/python
---

该库可用于服务端Python项目。客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-python-client)。

## 入门指南

以下说明将帮助您在本地机器上获取项目副本并运行，用于开发和测试目的。有关在生产环境部署项目的注意事项，请参阅生产环境运行指南。

## 安装说明

### 通过pip安装

```bash
pip install flagsmith
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目的单个环境进行初始化，例如开发环境或生产环境。您可以在环境设置页面找到环境密钥。

![API密钥](/img/api-key.png)

## 使用方式

### 获取项目功能开关

```python
from flagsmith import Flagsmith

flagsmith = Flagsmith(environment_id="<YOUR_ENVIRONMENT_KEY>")

if flagsmith.has_feature("header"):
  if flagsmith.feature_enabled("header"):
    # Show my awesome cool new feature to the world

value = flagsmith.get_value("header", '<My User Id>')

value = flagsmith.get_value("header")

flagsmith.set_trait("accept-cookies", "true", "ben@flagsmith.com")
flagsmith.get_trait("accept-cookies", "ben@flagsmith.com")
```

### 可用配置项

| Property         | Description                                                                                       | Required |                       Default Value |
| ---------------- | :------------------------------------------------------------------------------------------------ | -------: | ----------------------------------: |
| `environment_id` | Defines which project environment you wish to get flags for. _example ACME Project - Staging._    |  **YES** |                                None |
| `api`            | Use this property to define where you're getting feature flags from, e.g. if you're self hosting. |   **NO** | <https://api.flagsmith.com/api/v1/> |

### 可用函数

| Function                                    |                                                             Description                                                              |
| ------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------: |
| `has_feature(key)`                          |                  Determine if given feature exists for an environment. `bt.has_feature("powerUserFeature") // true`                  |
| `feature_enabled(key)`                      |                  Get the value of a particular _feature flag_ e.g. `bt.feature_enabled("powerUserFeature") // true`                  |
| `feature_enabled(key, userId)`              |               Get the value of a particular _feature flag_ e.g. `bt.feature_enabled("powerUserFeature", 1234) // true`               |
| `get_value(key)`                            |                         Get the value of a particular _remote config_ e.g. `bt.get_value("font_size") // 10`                         |
| `get_value(key, userId)`                    |               Get the value of a particular feature for a specified user e.g. `bt.get_value("font_size", 1234) // 15`                |
| `set_trait(trait_key, trait_value, userId)` |              Set the value of a particular trait for a specified user e.g. `bt.set_trait("font_size", 12, 1234) // 15`               |
| `get_trait(trait_key, userId)`              |                Get the value of a particular trait for a specified user e.g. `bt.get_trait("font_size", 1234) // 12`                 |
| `get_flags()`                               |           Trigger a manual fetch of the environment features, returns a list of flag objects, see below for returned data            |
| `get_flags_for_user(1234)`                  | Trigger a manual fetch of the environment features with a given user id, returns a list of flag objects, see below for returned data |

### 用户标识

用户标识功能允许您通过[Flagsmith控制台](https://flagsmith.com/)定位特定用户。您可以在调用`has_feature`和`get_value`方法时传入可选的用户标识参数，以获取相应用户的功能开关和变量值。

### 开关数据结构

| Field               | Description                            | Type                                                                            |
| ------------------- | -------------------------------------- | ------------------------------------------------------------------------------- |
| id                  | Internal id of feature state           | Integer                                                                         |
| enabled             | Whether feature is enabled or not      | Boolean                                                                         |
| environment         | Internal ID of environment             | Integer                                                                         |
| feature_state_value | Value of the feature                   | Any - determined based on data input on [flagsmith.com](https://flagsmith.com). |
| feature             | Feature object - see below for details | Object                                                                          |

### 功能数据结构

| Field        | Description                                                        | Type     |
| ------------ | ------------------------------------------------------------------ | -------- |
| id           | Internal id of feature                                             | Integer  |
| name         | Name of the feature (sometimes referred to as key or ID)           | String   |
| description  | Description of the feature                                         | String   |
| type         | Feature Type. Can be FLAG or CONFIG                                | String   |
| created_date | Date feature was created                                           | Datetime |
| inital_value | The initial / default value set for all feature states on creation | String   |
| project      | Internal ID of the associated project                              | Integer  |