---
title: Flagsmith Go SDK
sidebar_label: Go
description: Manage your Feature Flags and Remote Config in your Go applications.
slug: /clients/go
---

该Go语言SDK客户端适用于[https://flagsmith.com/](https://www.flagsmith.com/)。Flagsmith能帮助您跨项目、环境和组织管理功能开关与远程配置。

客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-go-client)。

## 入门指南

```bash
go get github.com/Flagsmith/flagsmith-go-client
```

```go
import (
  "github.com/Flagsmith/flagsmith-go-client"
)
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上单个项目环境（如开发环境或生产环境）进行初始化。您可在环境设置页面获取环境密钥。

![API密钥](/img/api-key.png)

## 获取项目功能开关

完整文档请访问[https://docs.flagsmith.com](https://docs.flagsmith.com)

注册并创建账户请至[https://flagsmith.com/](https://www.flagsmith.com/)

在应用中通过API密钥初始化Flagsmith客户端

```go
fs := flagsmith.DefaultClient("<Your API Key>")
```

检查功能开关是否存在并启用：

```go
fs := flagsmith.DefaultClient("<Your API Key>")
enabled, err := fs.FeatureEnabled("cart_abundant_notification_ab_test_enabled")
if err != nil {
    log.Fatal(err)
} else {
    if (enabled) {
        fmt.Printf("Feature enabled")
    }
}
```

获取功能开关的配置值：

```go
feature_value, err := fs.GetValue("cart_abundant_notification_ab_test")
if err != nil {
    log.Fatal(err)
} else {
    fmt.Printf(feature_value)
}
```

更多示例可参阅[测试代码](https://github.com/Flagsmith/flagsmith-go-client/blob/main/client_test.go)

## 覆盖默认配置

默认情况下客户端使用标准配置，您可通过以下方式覆盖配置：

```go
fs := flagsmith.NewClient("<Your API Key>", flagsmith.Config{BaseURI: "<Your API URL>"})
```