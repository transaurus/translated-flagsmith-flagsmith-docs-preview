---
sidebar_label: Frontend
title: Flagsmith Frontend
sidebar_position: 20
---

## 快速开始

以下说明将帮助您在本地机器上获取项目前端副本，用于开发和测试目的。有关如何在生产环境部署项目的注意事项，请参阅运行生产环境部分。

## 先决条件

安装软件所需的工具及其安装方法

| Location                                    | Suggested Version |
| ------------------------------------------- | :---------------: |
| <a href="https://nodejs.org/en/">NodeJS</a> |     >= 16.0.0     |
| <a href="https://nodejs.org/en/">npm</a>    |     >= 8.0.0      |

## 安装指南

```bash
cd frontend
npm i
```

## 运行前端

### 开发模式

支持客户端/服务端热重载

```bash
cd frontend
npm run dev
```

### 生产环境

您可以在[Heroku](https://www.heroku.com/)和[Dokku](http://dokku.viewdocs.io/dokku/)上直接部署此应用，只需修改'/frontend/env/project_prod.js'中的API URL即可

将项目打包、压缩并添加缓存标识后输出至构建目录，并以生产模式运行node。此流程可作为部署脚本的一部分使用。

```bash
cd frontend
npm run bundle
npm start
```

## 环境变量

不同环境间的变量通过'frontend/common/project.js'中的`window.Project`全局导出，该文件会根据"ENV"环境变量（如ENV=prod）被webpack替换为'frontend/env'目录下对应的project.js文件。

您可以通过编辑'frontend/bin/env.js'单独覆盖每个变量或添加更多变量。

当前在'frontend/environment.js'和'frontend/common/project.js'之间使用的变量：

- `FLAGSMITH_API_URL`: 用于请求的API地址。例如 `https://api.flagsmith.com/api/v1/`
- `FLAGSMITH_ON_FLAGSMITH_API_KEY`: 用于管理功能特性的Flagsmith环境密钥 - [Flagsmith运行在Flagsmith上](/deployment/overview#running-flagsmith-on-flagsmith)
- `FLAGSMITH_ON_FLAGSMITH_API_URL`: Flagsmith客户端通信的API地址。Flagsmith运行在Flagsmith上。例如 `https://api.flagsmith.com/api/v1/`。如果您自托管并使用自己的Flagsmith实例管理其功能特性，通常应将其指向您自托管实例的域名。
- `ENABLE_INFLUXDB_FEATURES`: 启用依赖InfluxDB的功能。API使用图表、功能分析等。例如 `ENABLE_INFLUXDB_FEATURES=1`
- `ENABLE_FLAG_EVALUATION_ANALYTICS`: 决定Flagsmith SDK是否发送使用分析数据。如需启用功能分析，请设置此变量。例如 `ENABLE_FLAG_EVALUATION_ANALYTICS=1`
- `PROXY_API_URL`: 通过此应用程序代理API。设置为被代理API的主机名。将`/api/v1/`代理到`PROXY_API_URL`。使用此设置时，任何`FLAGSMITH_API_URL`设置将被忽略，浏览器将通过前端Node服务器发送API请求。不要添加`api/v1/`前缀 - 它会自动添加。
- `GOOGLE_ANALYTICS_API_KEY`: 用于跟踪API使用情况的Google Analytics密钥
- `CRISP_WEBSITE_ID`: Crisp聊天小部件的网站密钥
- `ALLOW_SIGNUPS`: **已弃用，推荐使用PREVENT_SIGNUP** 决定是否允许无邀请的手动注册。设置任意值以允许注册
- `PREVENT_SIGNUP`: 决定是否阻止无邀请的手动注册。设置任意值以阻止注册
- `PREVENT_FORGOT_PASSWORD`: 决定是否禁用忘记密码功能，对LDAP/SAML有用。设置任意值以禁用忘记密码功能
- `ENABLE_MAINTENANCE_MODE`: 将站点置于维护模式。设置任意值以启用维护模式
- `AMPLITUDE_API_KEY`: 用于行为跟踪的Amplitude密钥
- `MIXPANEL_API_KEY`: 用于行为跟踪的Mixpanel分析密钥
- `SENTRY_API_KEY`: 用于错误报告的Sentry密钥
- `STATIC_ASSET_CDN_URL`: 用于替换本地静态路径的CDN地址，例如https://cdn.flagsmith.com。默认为`/`，即不使用CDN
- `BASE_URL`: 用于指定在从子目录提供服务时路由忽略的基础URL路径

## 端到端测试

本项目使用[Test Cafe](https://testcafe.io/)配合Chromedriver进行自动化端到端测试

```bash
npm test
```

## 构建技术

- React
- Webpack
- Node

## 在本地运行并连接自有Flagsmith API实例

我们使用Flagsmith管理功能发布，如果您使用自己的Flagsmith环境（即通过编辑project_x.js->flagsmith），则需要复制我们的功能标志

当前生产环境中使用的功能标志和远程配置列表可[在此查看](/deployment/overview#current-flagsmith-feature-flags)