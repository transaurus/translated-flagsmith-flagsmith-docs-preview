---
title: Flagsmith Ruby SDK
sidebar_label: Ruby
description: Manage your Feature Flags and Remote Config in your Ruby applications.
slug: /clients/ruby
---

Ruby版SDK客户端 [https://flagsmith.com/](https://www.flagsmith.com/)。Flagsmith支持跨多项目、多环境和多组织管理功能开关与远程配置。

客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-ruby-client)。

## 入门指南

本指南将帮助您在本地开发测试环境中运行项目副本。生产环境部署说明请参阅运行部署章节。

## 安装说明

通过gem安装：

`gem install flagsmith`

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目的特定环境（如开发环境或生产环境）进行初始化。环境密钥可在环境设置页面获取。

![API密钥](/img/api-key.png)

## 使用说明

### 获取项目功能开关

```ruby
require "flagsmith"

flagsmith = Flagsmith.new("<<Your API KEY>>")

if flagsmith.get_value("font_size")
  #    Do something awesome with the font size
end

if flagsmith.feature_enabled?("does_not_exist")
  #do something
else
  #do nothing, or something else
end
```

## 可用配置项

| Property  |                                            Description                                            | Required |                       Default Value |   Environment Key |
| --------- | :-----------------------------------------------------------------------------------------------: | -------: | ----------------------------------: | ----------------: |
| `api_key` |  Defines which project environment you wish to get flags for. _example ACME Project - Staging._   |  **YES** |                                null | FLAGSMITH_API_KEY |
| `url`     | Use this property to define where you're getting feature flags from, e.g. if you're self hosting. |   **NO** | <https://api.flagsmith.com/api/v1/> |     FLAGSMITH_URL |

## 可用函数

| Property                                          |                                                     Description                                                      |
| ------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------: |
| `init`                                            |                                 Initialize the sdk against a particular environment                                  |
| `feature_enabled?(key)`                           |         Get the value of a particular feature e.g. `flagsmith.feature_enabled?("powerUserFeature") // true`          |
| `feature_enabled?(key, user_id, default = false)` | Get the value of a particular feature for a user e.g. `flagsmith.feature_enabled?("powerUserFeature", 1234) // true` |
| `get_value(key)`                                  |                 Get the value of a particular feature e.g. `flagsmith.get_value("font_size") // 10`                  |
| `get_value(key, user_id, default = nil)`          |    Get the value of a particular feature for a specified user e.g. `flagsmith.get_value("font_size", 1234) // 15`    |
| `get_flags()`                                     |       Trigger a manual fetch of the environment features, if a user is identified it will fetch their features       |
| `get_flags(user_id)`                              |                       Trigger a manual fetch of the environment features with a given user id                        |
| `set_trait(user_id, trait, value)`                |                                    Set the value of a trait for the given user id                                    |

## 用户标识

通过[Flagsmith控制面板](https://www.flagsmith.com/)标识用户可实现精准用户定向。在调用`feature_enabled?`和`get_value`方法时传入可选用户标识参数，即可获取相应用户的功能开关与变量配置。