---
title: Flagsmith Rust SDK
sidebar_label: Rust
description: Manage your Feature Flags and Remote Config in your Rust applications.
slug: /clients/rust
---

适用于 Rust 的 SDK 客户端 [https://flagsmith.com/](https://www.flagsmith.com/)。Flagsmith 允许您跨多个项目、环境和组织管理功能开关与远程配置。

客户端源代码托管于 [Github](https://github.com/flagsmith/flagsmith-rust-client)。

该客户端 SDK 已发布至 [Crates](https://crates.io/crates/flagsmith) 平台

## 基础用法

SDK 需针对 [https://flagsmith.com](https://flagsmith.com) 上某个项目内的特定环境（例如开发环境或生产环境）进行初始化。您可以在环境设置页面找到对应的环境密钥。

![API密钥](/img/api-key.png)

## 使用方法

### 获取项目的功能开关

在应用程序中通过API密钥初始化Flagsmith客户端

```rust
let flagsmith = flagsmith::Client::new("<Your API Key>");
```

检查功能开关是否存在并已启用：

```rust
let flagsmith = flagsmith::Client::new("<Your API Key>");
if flagsmith.feature_enabled("cart_abundant_notification_ab_test_enabled")? {
    println!("Feature enabled");
}
```

获取功能开关的配置值：

```rust
use flagsmith::{Client,Value};

let flagsmith = Client::new("<Your API Key>");

if let Some(Value::String(s)) = bt.get_value("cart_abundant_notification_ab_test")? {
    println!("{}", s);
}
```

更多示例可参考[测试代码](https://github.com/Flagsmith/flagsmith-rust-client/blob/main/tests/integration_test.rs)

## 覆盖默认配置

客户端默认使用预设配置，您可通过以下方式覆盖配置：

```rust
let flagsmith = flagsmith::Client {
    api_key: String::from("secret key"),
    base_uri: String::from("https://features.on.my.own.server/api/v1/"),
};
```