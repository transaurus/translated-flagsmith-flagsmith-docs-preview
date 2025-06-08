---
title: Flagsmith Elixir SDK
sidebar_label: Elixir
description: Manage your Feature Flags and Remote Config in your Elixir applications.
slug: /clients/elixir
---

该库可用于服务端Elixir项目，客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-elixir-client)。

包含封装API功能、数据模式及响应的函数与类型。

## 安装

在`mix.exs`文件的`deps`部分添加：

### 开发期间仅使用github源

```elixir
def deps do
  [
    {:flagsmith_elixir_sdk, "~> 0.1"}
  ]
end
```

## 配置

可通过静态默认配置按环境配置SDK，例如在`config/config.exs`中添加：

```elixir
config :flagsmith_elixir_sdk, :sdk,
       environment_key: "YOUR-ENV-KEY"
```

若未使用公共API，可配置基础URL路径：

```elixir
config :flagsmith_elixir_sdk, :sdk,
       environment_key: "YOUR-ENV-KEY",
       base_url: "YOUR-BASE-URL"
```

如需运行时配置，可手动创建客户端结构体，并将其作为首个参数传递给任意SDK函数：

```elixir
{:ok, sdk_client} = Flagsmith.SDK.new("YOUR-ENV-KEY")

Flagsmith.SDK.API.flags_list(sdk_client)

### sample response ###
{:ok, [
   %Flagsmith.API.FeatureStateSerializerFull{
     enabled: false,
     environment: 11278,
     feature: %Flagsmith.API.Feature{
       created_date: ~U[2021-10-24 13:40:02Z],
       default_enabled: false,
       description: "Header Size",
       id: 13534,
       initial_value: "24px",
       name: "header_size",
       type: "MULTIVARIATE"
     },
     feature_segment: nil,
     feature_state_value: "24px",
     id: 72267,
     identity: nil
   },
   %Flagsmith.API.FeatureStateSerializerFull{
     enabled: false,
     environment: 11278,
     feature: %Flagsmith.API.Feature{
       created_date: ~U[2021-10-24 13:41:35Z],
       default_enabled: false,
       description: nil,
       id: 13535,
       initial_value: "18px",
       name: "body_size",
       type: "STANDARD"
     },
     feature_segment: nil,
     feature_state_value: "18px",
     id: 72269,
     identity: nil
   }
 ]}
```

`Flagsmith.SDK.new/2`将基础URL作为第二个参数接收。

## HTTP客户端

SDK底层使用[Tesla](https://github.com/teamon/tesla)，因此可自定义适配器。具体配置请参阅`Tesla`文档。

默认HTTP适配器为Erlang的`httpc`。若需改用例如`hackney`等支持的客户端，需在依赖中添加对应组件：

```elixir
defp deps do
  [
    # ...
    {:hackney, "~> 1.17"}
  ]
end
```

并在`config/config.exs`中配置适配器模块：

```elixir
config :tesla, adapter: Tesla.Adapter.Hackney
```