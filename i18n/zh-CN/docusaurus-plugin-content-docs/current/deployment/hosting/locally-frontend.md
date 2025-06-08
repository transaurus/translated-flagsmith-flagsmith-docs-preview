---
sidebar_label: Frontend
title: Flagsmith Frontend
sidebar_position: 20
---

## 入门指南

以下说明将帮助您在本地机器上获取项目前端副本并运行，用于开发和测试目的。有关在生产环境部署项目的注意事项，请参阅运行生产环境部分。

## 先决条件

安装软件所需的工具及其安装方法。安装nvm及其bash脚本可确保开发者使用相同的NodeJS版本。

| Location                                                                | Suggested Version |
| ----------------------------------------------------------------------- | :---------------: |
| <a href="https://nodejs.org/en/">NodeJS</a>                             |     >= 16.0.0     |
| <a href="https://nodejs.org/en/">npm</a>                                |     >= 8.0.0      |
| <a href="https://github.com/nvm-sh/nvm#installing-and-updating">nvm</a> |     >= 0.39.0     |
| <a href="https://github.com/nvm-sh/nvm#zsh">nvm zshrc setup</a>         |        n/a        |

## 安装步骤

```bash
cd frontend
npm i
```

## 运行前端

### 开发模式

支持客户端/服务器的热重载

```bash
cd frontend
npm run dev
```

### 生产模式

您可以在[Heroku](https://www.heroku.com/)和[Dokku](http://dokku.viewdocs.io/dokku/)上部署此应用而无需修改代码，只需更改'/frontend/env/project_prod.js'中的API URL即可

将项目打包、压缩并进行缓存破坏后输出至构建文件夹，并以生产模式运行node。此操作可作为部署脚本的一部分使用。

```bash
cd frontend
npm run bundle
npm start
```

## 环境变量

不同环境特有的变量会通过`window.Project`全局导出到'frontend/common/project.js'中。该文件会被webpack根据"ENV"环境变量（例如ENV=prod）替换为'frontend/env'目录下对应的project.js文件。

您可以通过编辑'frontend/bin/env.js'来单独覆盖每个变量或添加更多变量。

当前在'frontend/environment.js'和'frontend/common/project.js'之间使用的变量：

- `FLAGSMITH_API_URL`: 用于请求的API地址。例如 `https://edge.api.flagsmith.com/api/v1/`
- `FLAGSMITH_ON_FLAGSMITH_API_KEY`: 用于管理功能特性的Flagsmith环境密钥 - [Flagsmith运行在Flagsmith之上](/deployment/overview#running-flagsmith-on-flagsmith)
- `FLAGSMITH_ON_FLAGSMITH_API_URL`: Flagsmith客户端应通信的API URL。Flagsmith运行在Flagsmith上。例如 `https://edge.api.flagsmith.com/api/v1/`。如果您自托管并使用自己的Flagsmith实例来管理其功能特性，通常应将其指向您自己的Flagsmith实例的域名。
- `ENABLE_INFLUXDB_FEATURES`: 启用依赖InfluxDB的功能。API使用图表、功能标志分析。例如 `ENABLE_INFLUXDB_FEATURES=1`
- `ENABLE_FLAG_EVALUATION_ANALYTICS`: 确定Flagsmith SDK是否发送使用分析数据。如需启用功能标志分析，请设置此变量。例如 `ENABLE_FLAG_EVALUATION_ANALYTICS=1`
- `PROXY_API_URL`: 通过此应用程序代理API。设置为被代理API的主机名。将`/api/v1/`代理到`PROXY_API_URL`。如果使用此设置，任何对`FLAGSMITH_API_URL`的设置将被忽略，浏览器将通过前端Node服务器发送API请求。不要添加`api/v1/`前缀 - 它会自动添加。
- `GOOGLE_ANALYTICS_API_KEY`: 用于跟踪API使用的Google Analytics密钥。
- `CRISP_WEBSITE_ID`: Crisp聊天小部件的网站密钥。
- `ALLOW_SIGNUPS`: **已弃用，推荐使用PREVENT_SIGNUP** 确定是否允许无邀请的手动注册。设置为任意值以允许注册。
- `PREVENT_SIGNUP`: 确定是否阻止无邀请的手动注册。设置为任意值以阻止注册。
- `PREVENT_FORGOT_PASSWORD`: 确定是否阻止忘记密码功能，对LDAP/SAML有用。设置为任意值以阻止忘记密码功能。
- `ENABLE_MAINTENANCE_MODE`: 将站点置于维护模式。设置为任意值以启用维护。
- `AMPLITUDE_API_KEY`: 用于行为跟踪的Amplitude密钥。
- `MIXPANEL_API_KEY`: 用于行为跟踪的Mixpanel分析密钥。
- `SENTRY_API_KEY`: 用于错误报告的Sentry密钥。
- `STATIC_ASSET_CDN_URL`: 用于替换本地静态路径的CDN地址，例如https://cdn.flagsmith.com。默认为`/`，即不使用CDN。
- `BASE_URL`: 用于指定在路由时忽略的基本URL路径，如果从子目录提供服务。

## 端到端测试

本项目使用[Test Cafe](https://testcafe.io/)与Chromedriver进行自动化端到端测试。

```bash
npm test
```

## 构建工具

- React
- Webpack
- Node

## 针对您自己的Flagsmith API实例本地运行

我们使用Flagsmith来管理推出的功能特性，如果您使用自己的Flagsmith环境（即通过编辑project_x.js->flagsmith），则需要复制我们的功能标志。

我们在生产环境中当前使用的功能标志和远程配置列表可[在此找到](/deployment/overview#current-flagsmith-feature-flags)。