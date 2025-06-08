---
id: intro
slug: /
title: Manage feature flags across web, mobile and server side applications
sidebar_position: 1
sidebar_label: Overview
---

# ![Flagsmith 文档](/img/banner-logo-dark.png)

[Flagsmith](https://flagsmith.com/) 允许您跨网页、移动端和服务器端应用程序管理功能特性。Flagsmith 是开源的，您可以选择自主托管或由我们提供托管服务。

该应用包含三个核心组件：

1. [服务端 REST API](https://github.com/Flagsmith/flagsmith/tree/main/api)
2. [前端管理界面](https://github.com/Flagsmith/flagsmith/tree/main/frontend)
3. [客户端库](/#client-libraries)

快速开始的方式有两种：直接使用 [https://flagsmith.com/](https://flagsmith.com/) 提供的第1和第2项服务，或者自行托管API与前端。完成这些组件的部署后，即可在应用中集成客户端库，开始远程管理功能特性。

## 开源版 vs SaaS版 vs 企业版

您可以自由使用开源版Flagsmith！不同版本之间存在以下差异：

- 开源版**无**API请求或身份标识限制——可随意扩展API实例集群
- 开源版**无**仪表盘用户数量限制——团队成员数量不受限
- SaaS版和企业版支持[变更请求与功能调度](advanced-use/change-requests.md)
- SaaS版和企业版具备[基于角色的访问控制](advanced-use/permissions.md)
- SaaS版和企业版提供更多[认证提供商](enterprise-edition)选项

## 安装指南

:::tip

我们还提供功能更强大的付费[企业版](enterprise-edition.md)。如需了解更多信息，欢迎随时联系我们。

:::

更多详情请参阅[自主托管页面](/deployment/overview)。

### 服务端API

源代码与安装指南详见[GitHub项目](https://github.com/flagsmith/flagsmith)。该API采用Python编写，基于Django及Django Rest框架构建。

### 前端网站

源代码与安装指南详见[GitHub项目](https://github.com/flagsmith/flagsmith-frontend)。前端网站使用React/JavaScript开发，需搭配NodeJS环境。

## 客户端库

完成前后端部署后，即可在应用中集成我们的客户端库。

| Client Side SDKs                       | Server Side SDKs               |
| -------------------------------------- | ------------------------------ |
| [Javascript](/clients/javascript)      | [Node.js](/v1.0/clients/node)  |
| [Android/Kotlin](/clients/android)     | [Java](/v1.0/clients/java)     |
| [Flutter](/clients/flutter)            | [.Net](/v1.0/clients/dotnet)   |
| [iOS/Swift](/clients/ios)              | [Python](/v1.0/clients/python) |
| [React & React Native](/clients/react) | [Ruby](/v1.0/clients/ruby)     |
| [Next.js and SSR](/clients/next-ssr)   | [Rust](/v1.0/clients/rust)     |
|                                        | [Go](/v1.0/clients/go)         |
|                                        | [Elixir](/v1.0/clients/elixir) |

## 后续步骤

查看我们的[快速入门指南](quickstart.md)，了解如何在应用中实施功能特性的高层级概览。