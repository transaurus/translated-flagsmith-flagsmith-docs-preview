---
title: Deployment Overview
sidebar_label: Overview
sidebar_position: 1
---

## 付费托管服务

如果您希望跳过自托管步骤，直接开始将Flagsmith集成到您的应用程序中，可以立即使用[https://flagsmith.com/](https://flagsmith.com/)。我们提供[适合初创企业和大型企业的多种付费方案](https://flagsmith.com/pricing)。

## 一键部署方案

[![部署至DigitalOcean](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/flagsmith/flagsmith/tree/main)

[![部署至Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/flagsmith/flagsmith/tree/main)

[![Railway平台部署](https://railway.app/button.svg)](https://railway.app/new/template?template=https%3A%2F%2Fgithub.com%2FFlagsmith%2Fflagsmith&plugins=postgresql&envs=DJANGO_ALLOWED_HOSTS%2CGUNICORN_THREADS%2CGUNICORN_WORKERS%2CPORT%2Cenv&DJANGO_ALLOWED_HOSTSDefault=%5B%27*%27%5D&GUNICORN_THREADSDefault=1&GUNICORN_WORKERSDefault=1&PORTDefault=8000&envDefault=prod)

![Fly.io](/img/logos/fly.io.svg)

我们非常推荐[Fly.io](https://Fly.io)平台！您可以通过以下方式轻松部署：

```bash
git clone git@github.com:Flagsmith/flagsmith.git
cd flagsmith
flyctl postgres create --name flagsmith-flyio-db
flyctl apps create flagsmith-flyio
flyctl postgres attach --postgres-app flagsmith-flyio-db
flyctl deploy
```

由于Fly.io采用全局应用命名空间，您可能需要修改[`fly.toml`](https://github.com/Flagsmith/flagsmith/blob/main/fly.toml)文件中定义的应用程序名称以及上述命令。

### Caprover部署

您也可以通过[Caprover服务器](https://caprover.com/)的[一键应用功能](https://caprover.com/docs/one-click-apps.html)进行部署。

## 自托管概览

完成自托管需要执行以下步骤：

1. 创建Postgres数据库用于存储Flagsmith数据
2. 部署API服务并配置DNS解析。若使用健康检查，请确保将`/health`设为健康检查端点
3. 访问`http://<您的服务器域名:8000>/api/v1/users/config/init/`创建初始超级用户并为平台配置DNS信息
4. 部署前端仪表盘并配置DNS解析。通过环境变量将仪表盘指向API服务。若使用健康检查，请确保将`/health`设为健康检查端点
5. 通过仪表盘创建新组织(Organisation)、项目(Project)、环境(Environment)和功能开关(Flags)
6. 使用SDK时需重写默认API地址（默认指向付费版API`https://api.flagsmith.com/api/v1`），具体操作请参考对应SDK文档

## 部署选项

我们推荐使用[Docker容器化部署](/deployment/hosting/docker)，同时支持[Kubernetes集群部署](/deployment/hosting/kubernetes)和[RedHat OpenShift平台部署](/deployment/hosting/openshift)。

## 系统架构

Flagsmith采用基于REST API的架构设计，SDK客户端和仪表盘前端Web应用均通过该API进行交互。

![系统架构图](/img/architecture.svg)

## 依赖组件

运行API服务需要以下核心依赖：

- Postgres数据库（我们测试过11.12版本，但其他版本也可兼容）—— 作为主数据存储

API还可选集成以下第三方服务：

- Google Analytics - 用于API分析
- InfluxDB - 用于API分析
- SendGrid - 用于事务性邮件
- AWS S3 - 存储Django静态资源
- GitHub - OAuth认证提供商
- Google - OAuth认证提供商

## InfluxDB配置

Flagsmith对InfluxDB存在软依赖以存储时间序列数据。平台运行无需配置Influx，但若未正确设置，SDK流量和功能开关分析将无法工作。启动docker-compose后：

1. 在influxdb创建用户账户。访问http://localhost:8086/完成操作。创建名为`flagsmith_api`的初始存储桶
2. 进入Data > Buckets创建第二个存储桶`flagsmith_api_downsampled_15m`
3. 进入Data > Tokens获取访问令牌
4. 编辑`docker-compose.yml`文件，在api服务中添加以下环境变量以连接InfluxDB：
   - `INFLUXDB_TOKEN`: 上一步获取的令牌
   - `INFLUXDB_URL`: `http://influxdb`
   - `INFLUXDB_ORG`: 组织ID - 可参考[此文档](https://docs.influxdata.com/influxdb/v2.0/organizations/view-orgs/)
   - `INFLUXDB_BUCKET`: `flagsmith_api`
5. 重启`docker-compose`
6. 登录InfluxDB，新建名为`flagsmith_api_downsampled_15m`的存储桶
7. 创建定时任务并设置以下查询语句。该任务会将毫秒级数据降采样为15分钟区块以加速查询。设定每15分钟执行一次：

```text
option task = {name: "Downsample", every: 15m}

data = from(bucket: "flagsmith_api")
	|> range(start: -duration(v: int(v: task.every) * 2))
	|> filter(fn: (r) =>
		(r._measurement == "api_call"))

data
	|> aggregateWindow(fn: sum, every: 15m)
	|> filter(fn: (r) =>
		(exists r._value))
	|> to(bucket: "flagsmith_api_downsampled_15m")
```

任务执行后，组织API使用情况区域将开始显示数据

需通过SDK生成功能开关评估数据，才能在Influx数据库中看到[功能开关分析](/advanced-use/flag-analytics.md)数据

## API遥测

Flagsmith会收集自托管安装的匿名使用数据，这些数据仅用于分析平台使用情况且绝不对外共享。可通过设置环境变量`ENABLE_TELEMETRY`为`False`来禁用遥测功能

我们会在启动时及之后每8小时收集以下数据（按API服务器实例统计）：

- 组织总数
- 项目总数
- 环境总数
- 功能总数
- 用户分群总数
- 用户总数
- Django的DEBUG变量状态
- Django的ENV变量值
- API服务器外部IP地址

## 基于Flagsmith自托管Flagsmith

Flagsmith前端仪表板通过Flagsmith自身实现功能控制。自托管时可能会遇到功能灰显，或需要禁用特定功能（如Google/Github登录）。若使用自有Flagsmith环境，需复制我们的功能开关配置以实现相同控制

操作步骤：
1. 在自托管实例中新建项目（用于控制该实例的功能）
2. 将前端仪表板指向此项目

创建项目后，需在[前端](https://github.com/Flagsmith/flagsmith-frontend)配置以下环境变量：

- `FLAGSMITH_ON_FLAGSMITH_API_KEY`
  - 用于管理功能的Flagsmith客户端环境密钥——Flagsmith运行在Flagsmith之上。这将是您按上述指引创建项目的API密钥。
- `FLAGSMITH_ON_FLAGSMITH_API_URL`
  - Flagsmith前端仪表板应与之通信的API URL。这很可能是您自托管Flagsmith API的域名：Flagsmith运行在Flagsmith之上。例如，对于我们的SaaS托管平台，该变量为`https://api.flagsmith.com/api/v1/`。如果您使用标准[docker-compose设置](https://github.com/Flagsmith/flagsmith-docker)在本地运行所有内容，则应使用`http://localhost:8000/api/v1/`

完成此设置后，您应该能看到Flagsmith前端从API请求其自身的功能标志（可在浏览器开发者控制台中查看）。现在您可以开始创建标志并覆盖平台的默认行为。例如，若要禁用Google OAuth认证，您需要创建一个名为`oauth_google`的标志并将其禁用。

### 当前Flagsmith功能标志

以下是我们在生产环境中当前使用的标志和远程配置列表：

| Flag Name              | Description                                                                   | Text Value                                                                                                     |
| ---------------------- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `try_it`               | Whether to show the try it buttons                                            | None                                                                                                           |
| `butter_bar`           | markdown to show at the top of the dashboard page                             | None                                                                                                           |
| `disable_create_org`   | Turning this on will prevent users from creating any additional organisations | None                                                                                                           |
| `scaleup_audit`        | Disables audit log for anyone under scale-up plan                             | None                                                                                                           |
| `integration_data`     | Configures integrations                                                       | [See Below](#integration_data)                                                                                 |
| `integrations`         | Whether to show the try it buttons                                            | `["amplitude","datadog","dynatrace","new-relic","segment","rudderstack","webhook","slack", "heap","mixpanel"]` |
| `plan_based_access`    | Controls RBAC and 2FA based on organisation plan                              | None                                                                                                           |
| `flag_analytics`       | Flag usage chart - requires InfluxDB                                          | None                                                                                                           |
| `usage_chart`          | Organisation Analytics usage chart - requires InfluxDB                        | None                                                                                                           |
| `dark_mode`            | Enables Dark Mode in UI [See Below](#dark-mode)                               | None                                                                                                           |
| `scaleup_audit`        | Disables audit log for anyone under scale-up plan                             | None                                                                                                           |
| `serverside_sdk_keys`  | Enable Server-side Environment Keys                                           | None                                                                                                           |
| `compare_environments` | Compare feature flag changes across environments                              | None                                                                                                           |
| `segment_operators`    | Determines what rules are shown when creating a segment                       | [See Below](#segment_operators)                                                                                |
| `oauth_github`         | Disables audit log for anyone under scale-up plan                             | [See Below](#oauth_github)                                                                                     |
| `oauth_google`         | Disables audit log for anyone under scale-up plan                             | [See Below](#oauth_google)                                                                                     |

### `integration_data`

```json
{
 "datadog": {
  "perEnvironment": false,
  "image": "https://app.flagsmith.com/static/images/integrations/datadog.svg",
  "docs": "https://docs.flagsmith.com/integrations/datadog/",
  "fields": [
   {
    "key": "base_url",
    "label": "Base URL"
   },
   {
    "key": "api_key",
    "label": "API Key"
   }
  ],
  "tags": ["logging"],
  "title": "Datadog",
  "description": "Sends events to Datadog for when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production."
 },
 "dynatrace": {
  "perEnvironment": true,
  "image": "/static/images/integrations/dynatrace.svg",
  "docs": "https://docs.flagsmith.com/integrations/dynatrace/",
  "fields": [
   {
    "key": "base_url",
    "label": "Base URL"
   },
   {
    "key": "api_key",
    "label": "API Key"
   },
   {
    "key": "entity_selector",
    "label": "Entity Selector"
   }
  ],
  "tags": ["logging"],
  "title": "Dynatrace",
  "description": "Sends events to Dynatrace for when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production."
 },
 "slack": {
  "perEnvironment": true,
  "isOauth": true,
  "image": "https://app.flagsmith.com/static/images/integrations/slack.svg",
  "docs": "https://docs.flagsmith.com/integrations/slack/",
  "tags": ["messaging"],
  "title": "Slack",
  "description": "Sends messages to Slack when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production."
 },
 "amplitude": {
  "perEnvironment": true,
  "image": "https://app.flagsmith.com/static/images/integrations/amplitude.svg",
  "docs": "https://docs.flagsmith.com/integrations/amplitude/",
  "fields": [
   {
    "key": "api_key",
    "label": "API Key"
   }
  ],
  "tags": ["analytics"],
  "title": "Amplitude",
  "description": "Sends data on what flags served to each identity."
 },
 "new-relic": {
  "perEnvironment": false,
  "image": "https://app.flagsmith.com/static/images/integrations/new_relic.svg",
  "docs": "https://docs.flagsmith.com/integrations/newrelic",
  "fields": [
   {
    "key": "base_url",
    "label": "New Relic Base URL"
   },
   {
    "key": "api_key",
    "label": "New Relic API Key"
   },
   {
    "key": "app_id",
    "label": "New Relic Application ID"
   }
  ],
  "tags": ["analytics"],
  "title": "New Relic",
  "description": "Sends events to New Relic for when flags are created, updated and removed."
 },
 "segment": {
  "perEnvironment": true,
  "image": "https://app.flagsmith.com/static/images/integrations/segment.svg",
  "docs": "https://docs.flagsmith.com/integrations/segment",
  "fields": [
   {
    "key": "api_key",
    "label": "API Key"
   }
  ],
  "tags": ["analytics"],
  "title": "Segment",
  "description": "Sends data on what flags served to each identity."
 },
 "rudderstack": {
  "perEnvironment": true,
  "image": "https://app.flagsmith.com/static/images/integrations/rudderstack.svg",
  "docs": "https://docs.flagsmith.com/integrations/rudderstack",
  "fields": [
   {
    "key": "base_url",
    "label": "Rudderstack Data Plane URL"
   },
   {
    "key": "api_key",
    "label": "API Key"
   }
  ],
  "tags": ["analytics"],
  "title": "Rudderstack",
  "description": "Sends data on what flags served to each identity."
 },
 "webhook": {
  "perEnvironment": true,
  "image": "https://app.flagsmith.com/static/images/integrations/webhooks.svg",
  "docs": "https://docs.flagsmith.com/integrations/webhook",
  "fields": [
   {
    "key": "url",
    "label": "Your Webhook URL Endpoint"
   },
   {
    "key": "secret",
    "label": "Your Webhook Secret"
   }
  ],
  "tags": ["analytics"],
  "title": "Webhook",
  "description": "Sends data on what flags served to each identity to a Webhook Endpoint you provide."
 },
 "heap": {
  "perEnvironment": true,
  "image": "https://app.flagsmith.com/static/images/integrations/heap.svg",
  "docs": "https://docs.flagsmith.com/integrations/heap",
  "fields": [
   {
    "key": "api_key",
    "label": "API Key"
   }
  ],
  "tags": ["analytics"],
  "title": "Heap Analytics",
  "description": "Sends data on what flags served to each identity."
 },
 "mixpanel": {
  "perEnvironment": true,
  "image": "https://app.flagsmith.com/static/images/integrations/mp.svg",
  "docs": "https://docs.flagsmith.com/integrations/mixpanel",
  "fields": [
   {
    "key": "api_key",
    "label": "Project Token"
   }
  ],
  "tags": ["analytics"],
  "title": "Mixpanel",
  "description": "Sends data on what flags served to each identity."
 }
}
```

### `segment_operators`

```json
[
 {
  "value": "EQUAL",
  "label": "Exactly Matches (=)"
 },
 {
  "value": "NOT_EQUAL",
  "label": "Does not match (!=)"
 },
 {
  "value": "PERCENTAGE_SPLIT",
  "label": "% Split"
 },
 {
  "value": "GREATER_THAN",
  "label": ">"
 },
 {
  "value": "GREATER_THAN_INCLUSIVE",
  "label": ">="
 },
 {
  "value": "LESS_THAN",
  "label": "<"
 },
 {
  "value": "LESS_THAN_INCLUSIVE",
  "label": "<="
 },
 {
  "value": "GREATER_THAN:semver",
  "label": "SemVer >",
  "append": ":semver"
 },
 {
  "value": "GREATER_THAN_INCLUSIVE:semver",
  "label": "SemVer >=",
  "append": ":semver"
 },
 {
  "value": "LESS_THAN:semver",
  "label": "SemVer <",
  "append": ":semver"
 },
 {
  "value": "LESS_THAN_INCLUSIVE:semver",
  "label": "SemVer <=",
  "append": ":semver"
 },

 {
  "value": "CONTAINS",
  "label": "Contains"
 },
 {
  "value": "NOT_CONTAINS",
  "label": "Does not contain"
 },
 {
  "value": "REGEX",
  "label": "Matches regex"
 }
]
```

### `oauth_github`

```json
{
 "url": "<Your github redirect URL>"
}
```

### `oauth_google`

```json
{
 "clientId": "<Your Google oAuth Client ID>",
 "apiKey": "<Your Google redirect URL>"
}
```

### 暗黑模式

我们还通过一个细分规则管理UI暗黑模式：

细分名称：`dark_mode` 细分规则：特征`dark_mode`完全匹配`True`

然后使用此规则覆盖`dark_mode`功能标志。

## 集成配置

部分[集成功能](/integrations/overview)需要额外配置

### Slack集成

创建一个私有Slack应用。随后需在API端配置以下环境变量：

- `SLACK_CLIENT_ID`
- `SLACK_CLIENT_SECRET`

这些值可从Slack获取。您还需添加以下权限范围：

- channels:read
- chat:write
- chat:write.public

## 手动安装

如需更灵活的环境配置，可手动安装前端和API组件。

### 服务端API

源代码与安装指南请参阅[GitHub项目](https://github.com/Flagsmith/flagsmith/tree/main/api)。该API采用Python编写，基于Django和Django Rest框架构建。服务端API依赖PostgreSQL数据库存储数据，并需要Redis作为缓存。

### 前端网站

源代码与安装指南请参阅[GitHub项目](https://github.com/Flagsmith/flagsmith/tree/main/frontend)。前端网站采用React/JavaScript开发，需要NodeJS运行环境。