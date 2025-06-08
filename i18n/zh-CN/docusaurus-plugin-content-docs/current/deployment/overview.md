---
title: Deployment Overview
sidebar_label: Overview
sidebar_position: 1
---

## 付费托管服务

若您希望跳过自托管环节直接集成Flagsmith到应用，可立即使用[https://flagsmith.com/](https://flagsmith.com/)。我们提供[适合初创企业和大型企业的付费方案](https://flagsmith.com/pricing)。

## 一键部署方案

[![部署至DigitalOcean](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/flagsmith/flagsmith/tree/main)

[![部署至Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/flagsmith/flagsmith/tree/main)

[![Railway部署](https://railway.app/button.svg)](https://railway.app/new/template?template=https%3A%2F%2Fgithub.com%2FFlagsmith%2Fflagsmith&plugins=postgresql&envs=DJANGO_ALLOWED_HOSTS%2CGUNICORN_THREADS%2CGUNICORN_WORKERS%2CPORT%2Cenv&DJANGO_ALLOWED_HOSTSDefault=%5B%27*%27%5D&GUNICORN_THREADSDefault=1&GUNICORN_WORKERSDefault=1&PORTDefault=8000&envDefault=prod)

![Fly.io](/img/logos/fly.io.svg)

我们强烈推荐[Fly.io](https://Fly.io)平台，您可轻松完成部署：

```bash
git clone git@github.com:Flagsmith/flagsmith.git
cd flagsmith
flyctl postgres create --name flagsmith-flyio-db
flyctl apps create flagsmith-flyio
flyctl postgres attach --postgres-app flagsmith-flyio-db
flyctl deploy
```

Fly.io采用全局应用命名空间，您可能需要修改[`fly.toml`](https://github.com/Flagsmith/flagsmith/blob/main/fly.toml)中定义的应用名称及上述命令。

### Caprover

您也可以通过[Caprover服务器](https://caprover.com/)的[一键应用](https://caprover.com/docs/one-click-apps.html)进行部署。

## 自托管概览

完成自托管需执行以下步骤：

1. 创建Postgres数据库存储Flagsmith数据
2. 部署API并配置DNS。若启用健康检查，请确保使用`/health`作为检查端点
3. 访问`http://<您的服务器域名:8000>/api/v1/users/config/init/`创建超级用户并为平台配置DNS信息
4. 部署前端仪表盘并配置DNS。通过环境变量将仪表盘指向API。健康检查同样需使用`/health`端点
5. 通过仪表盘创建新组织、项目、环境及功能开关
6. 使用SDK时需重写默认API地址（默认指向付费API`https://edge.api.flagsmith.com/api/v1`），具体操作请参考对应SDK文档

## 部署选项

我们推荐使用[Docker](/deployment/hosting/docker)运行Flagsmith，同时支持[Kubernetes](/deployment/hosting/kubernetes)和[RedHat OpenShift](/deployment/hosting/openshift)部署。

## 系统架构

Flagsmith采用基于REST API的架构，SDK客户端与仪表盘前端Web应用均通过该API进行交互。

![应用架构图](/img/architecture.svg)

## 依赖项

运行API需要以下核心依赖：

- Postgres数据库：主数据存储。已测试兼容Postgres v11.12，其他版本亦可运行

API还可选集成以下第三方服务：

- Google Analytics - 用于API分析
- InfluxDB - 用于API分析
- SendGrid - 用于事务性邮件
- AWS S3 - 存储Django静态资源
- GitHub - OAuth认证提供商
- Google - OAuth认证提供商

## 功能开关分析

Flagsmith会存储两类时间序列数据：

1. 功能开关分析数据
2. API流量报告数据

可通过以下三种方式配置存储和处理这些数据：

1. 存储至Postgres
2. 存储至InfluxDB
3. 完全不存储

推荐采用第一种方案。

### 通过Postgres存储时间序列数据

需为Flagsmith API服务添加以下环境变量：

```bash
# Set Postgres to store the data
USE_POSTGRES_FOR_ANALYTICS=True

# Configure the postgres datastore:
# Either
ANALYTICS_DATABASE_URL (e.g. postgresql://postgres:password@postgres:5432/flagsmith)
# Or
DJANGO_DB_HOST_ANALYTICS (e.g. postgres.db)
DJANGO_DB_NAME_ANALYTICS (e.g. flagsmith)
DJANGO_DB_USER_ANALYTICS (e.g. postgres_user)
DJANGO_DB_PASSWORD_ANALYTICS (e.g. postgres_password)
DJANGO_DB_PORT_ANALYTICS (e.g. 5432)
```

注意：可与核心Flagsmith数据库使用不同的数据库或数据库服务器。

同时需要运行[任务处理器](https://docs.flagsmith.com/deployment/task-processor)才能实现数据降采样，并使统计数据显示在仪表盘中。此过程最长可能需要1小时。

## API遥测

Flagsmith会收集自托管安装的相关信息，用于分析平台使用情况。这些数据_绝不会_对外共享，且设计上为匿名采集。可通过设置`ENABLE_TELEMETRY`环境变量为`False`来退出遥测数据上报。

我们会在启动时及之后每8小时（按API服务器实例）收集以下数据：

- 组织总数
- 项目总数
- 环境总数
- 功能总数
- 用户分群总数
- 用户总数
- Django的DEBUG变量值
- Django的ENV变量值
- API服务器外部IP地址

## 基于Flagsmith自托管Flagsmith

Flagsmith前端仪表盘的功能由Flagsmith自身控制。自托管平台时，可能会发现某些功能显示为灰色，或需要禁用特定功能（例如通过Google/Github登录）。若使用自托管环境，需复制我们的功能开关配置才能控制这些功能。

操作步骤：首先在自托管Flagsmith应用中新建项目，该项目将用于控制自托管实例的功能。然后将自托管前端仪表盘指向此Flagsmith项目，从而实现对功能可见性的控制。

创建项目后，需配置以下[前端](https://github.com/Flagsmith/flagsmith-frontend)环境变量：

- `FLAGSMITH_ON_FLAGSMITH_API_KEY`
  - 用于管理功能的Flagsmith客户端环境密钥（Flagsmith自托管场景）。此处填写前述步骤中创建项目的API密钥。
- `FLAGSMITH_ON_FLAGSMITH_API_URL`
  - 前端仪表盘应连接的API地址。通常为自托管Flagsmith API的域名，例如我们的SaaS平台使用`https://edge.api.flagsmith.com/api/v1/`。若使用标准[docker-compose方案](https://github.com/Flagsmith/flagsmith-docker)本地运行，则应设置为`http://localhost:8000/api/v1/`

完成上述设置后，您应该能看到Flagsmith前端从API请求自身的功能标志（可通过浏览器开发者控制台查看）。此时您可以开始创建标志并覆盖平台的默认行为。例如，若需禁用Google OAuth认证，只需创建名为`oauth_google`的标志并将其禁用。

### 当前Flagsmith功能标志

以下列出我们当前在生产环境中使用的功能标志及远程配置：

| Flag Name                     | Description                                                                              | Text Value                                                                                                    |
| ----------------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `4eyes`                       | Whether to enable the [Change Requests](../advanced-use/change-requests.md) feature      | None                                                                                                          |
| `butter_bar`                  | markdown to show at the top of the dashboard page                                        | None                                                                                                          |
| `case_sensitive_flags`        | Enables the project setting to allow case sensitive flags                                | None                                                                                                          |
| `compare_environments`        | Compare feature flag changes across environments                                         | None                                                                                                          |
| `dark_mode`                   | Enables Dark Mode in UI [See Below](#dark-mode)                                          | None                                                                                                          |
| `disable_create_org`          | Turning this on will prevent users from creating any additional organisations            | None                                                                                                          |
| `feature_name_regex`          | Enables the project setting to add a regex matcher to validate feature names             | None                                                                                                          |
| `flag_analytics`              | Flag usage chart - requires InfluxDB                                                     | None                                                                                                          |
| `force_2fa`                   | Enables the organisation setting to force 2 factor authentication                        | None                                                                                                          |
| `integration_data`            | Configures integrations                                                                  | [See Below](#integration_data)                                                                                |
| `integrations`                | Which third party integrations are displayed                                             | `["amplitude","datadog","dynatrace","heap","mixpanel","new-relic","rudderstack","segment","slack","webhook"]` |
| `oauth_github`                | Configure Github OAuth                                                                   | [See Below](#oauth_github)                                                                                    |
| `oauth_google`                | Configure Google OAuth                                                                   | [See Below](#oauth_google)                                                                                    |
| `rotate_api_token`            | Enables the ability to rotate a user's access token                                      | [See Below](#oauth_google)                                                                                    |
| `saml`                        | Enables SAML authentication                                                              | [See Below](#oauth_google)                                                                                    |
| `scaleup_audit`               | Disables audit log for anyone under scale-up plan                                        | None                                                                                                          |
| `segment_associated_features` | Enables the ability to see features associated with a segment                            | None                                                                                                          |
| `segment_operators`           | Determines what rules are shown when creating a segment                                  | [See Below](#segment_operators)                                                                               |
| `serverside_sdk_keys`         | Enable Server-side Environment Keys                                                      | None                                                                                                          |
| `sso_idp`                     | Set this to your configured SAML organisation name to automatically redirect to your IdP | None                                                                                                          |
| `tag_environments`            | Enables an environment setting to add a UI hint to your environments (e.g. for prod)     | None                                                                                                          |
| `try_it`                      | Whether to show the try it buttons                                                       | None                                                                                                          |
| `usage_chart`                 | Organisation Analytics usage chart - requires InfluxDB                                   | None                                                                                                          |

### `integration_data`

```json
{
 "datadog": {
  "perEnvironment": false,
  "image": "/static/images/integrations/datadog.svg",
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
  "image": "/static/images/integrations/slack.svg",
  "docs": "https://docs.flagsmith.com/integrations/slack/",
  "tags": ["messaging"],
  "title": "Slack",
  "description": "Sends messages to Slack when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production."
 },
 "amplitude": {
  "perEnvironment": true,
  "image": "/static/images/integrations/amplitude.svg",
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
  "image": "/static/images/integrations/new_relic.svg",
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
  "image": "/static/images/integrations/segment.svg",
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
  "image": "/static/images/integrations/rudderstack.svg",
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
  "image": "/static/images/integrations/webhooks.svg",
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
  "image": "/static/images/integrations/heap.svg",
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
  "image": "/static/images/integrations/mp.svg",
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
 },
 {
  "value": "MODULO",
  "label": "Modulo operation"
 },
 {
  "value": "IS_SET",
  "label": "Is set",
  "hideValue": true
 },
 {
  "value": "IS_NOT_SET",
  "label": "Is not set",
  "hideValue": true
 }
]
```

### `oauth_github`

GitHub认证配置指南请参阅[此处](../deployment/configuration/authentication#github)。

在GitHub开发者控制台创建OAuth应用后，按以下格式提供标志值：

- 按如下方式创建Flagsmith控制标志，替换您的`client_id`和`redirect_uri`

```json
{
 "url": "https://github.com/login/oauth/authorize?scope=user&client_id=<your client_id>&redirect_uri=<your url encoded redirect uri>"
}
```

例如，我们的SaaS配置值如下（客户ID已脱敏）

```json
{
 "url": "https://github.com/login/oauth/authorize?scope=user&client_id=999999999999&redirect_uri=https%3A%2F%2Fapp.flagsmith.com%2Foauth%2Fgithub"
}
```

### `oauth_google`

在Google开发者控制台创建OAuth应用后，提供以下标志值：

```json
{
 "clientId": "<Your Google oAuth Client ID>",
 "apiKey": "<Your Google oAuth Client secret>"
}
```

### 暗黑模式

我们通过以下用户分片管理UI暗黑模式：

分片名称：`dark_mode` 分片规则：特征值`dark_mode`完全匹配`True`

随后使用此规则覆盖`dark_mode`功能标志。

## 集成配置

部分[集成功能](/integrations/overview)需要额外配置

### Slack集成

创建私有Slack应用后，需在API服务端配置以下环境变量：

- `SLACK_CLIENT_ID`
- `SLACK_CLIENT_SECRET`

这些值可从Slack获取，同时需添加以下权限范围：

- `channels:read`
- `chat:write`
- `chat:write.public`

还需配置应用的重定向URL。详细流程请参考Slack官方文档关于创建应用及OAuth流程的说明。以下生产环境Flagsmith应用清单可作为模板：

```json
{
 "display_information": {
  "name": "Flagsmith Bot",
  "description": "Get notified in Slack whenever changes are made to your Flagsmith Environments",
  "background_color": "#000000",
  "long_description": "Use our application for Slack to receive Flagsmith state changes directly in your Slack channels. Whenever you create, update or delete a Flag within Flagsmith, our application for Slack will send a message into a Slack channel of your choosing.\r\n\r\nFlagsmith is an open source, fully featured, Feature Flag and Remote Config service. Use our hosted API, deploy to your own private cloud, or run on-premise."
 },
 "features": {
  "bot_user": {
   "display_name": "Flagsmith Bot",
   "always_online": false
  }
 },
 "oauth_config": {
  "redirect_urls": [
   "https://api.flagsmith.com/api/v1/environments",
   "https://api-staging.flagsmith.com/api/v1/environments"
  ],
  "scopes": {
   "bot": ["channels:read", "chat:write", "chat:write.public"]
  }
 },
 "settings": {
  "org_deploy_enabled": false,
  "socket_mode_enabled": false,
  "token_rotation_enabled": false
 }
}
```

### 通过InfluxDB存储时间序列数据

Flagsmith对InfluxDB存在软依赖关系以存储时间序列数据。平台运行时默认将数据存入Postgres，无需强制配置InfluxDB。若您运行高流量负载，可考虑部署InfluxDB方案。

1. 在InfluxDB中创建用户账户。访问 http://localhost:8086/
2. 进入 Data > Buckets 创建三个新存储桶，分别命名为 `default`、`default_downsampled_15m` 和 `default_downsampled_1h`
3. 进入 Data > Tokens 获取访问令牌
4. 编辑 `docker-compose.yml` 文件，在api服务中添加以下环境变量以连接InfluxDB：
   - `INFLUXDB_TOKEN`: 上一步获取的令牌
   - `INFLUXDB_URL`: `http://influxdb`
   - `INFLUXDB_ORG`: 组织ID - 可在此处查找
     [文档](https://docs.influxdata.com/influxdb/v2.0/organizations/view-orgs/)
   - `INFLUXDB_BUCKET`: `default`
5. 重启 `docker-compose`
6. 创建新任务并执行以下查询。该操作会将毫秒级API请求数据降采样为15分钟区块以加速查询。设置每15分钟运行一次。

```text
option task = {name: "Downsample (API Requests)", every: 15m}

data = from(bucket: "default")
	|> range(start: -duration(v: int(v: task.every) * 2))
	|> filter(fn: (r) =>
		(r._measurement == "api_call"))

data
	|> aggregateWindow(fn: sum, every: 15m)
	|> filter(fn: (r) =>
		(exists r._value))
	|> to(bucket: "default_downsampled_15m")
```

任务运行后，您将在组织API使用情况区域看到数据流入。

7. 创建另一个新任务并执行以下查询。该操作会将毫秒级功能标志评估数据降采样为15分钟区块以加速查询。设置每15分钟运行一次。

```text
option task = {name: "Downsample (Flag Evaluations)", every: 15m}

data = from(bucket: "default")
	|> range(start: -duration(v: int(v: task.every) * 2))
	|> filter(fn: (r) =>
		(r._measurement == "feature_evaluation"))

data
	|> aggregateWindow(fn: sum, every: 15m)
	|> filter(fn: (r) =>
		(exists r._value))
	|> to(bucket: "default_downsampled_15m")
```

任务运行后，且您已启用分析功能进行标志评估（相关文档见[此处](/advanced-use/flag-analytics.md)），您将在仪表板的每个功能"分析"选项卡中看到数据。

8. 创建另一个新任务并执行以下查询。该操作会将毫秒级API请求数据降采样为1小时区块以加速查询。设置每小时运行一次。

```text
option task = {name: "Downsample API 1h", every: 1h}

data = from(bucket: "default")
	|> range(start: -duration(v: int(v: task.every) * 2))
	|> filter(fn: (r) =>
		(r._measurement == "api_call"))

data
	|> aggregateWindow(fn: sum, every: 1h)
    |> filter(fn: (r) =>
      (exists r._value))
	|> to(bucket: "default_downsampled_1h")
```

9. 创建另一个新任务并执行以下查询。该操作会将毫秒级功能标志评估数据降采样为1小时区块以加速查询。设置每小时运行一次。

```text
option task = {name: "Downsample API 1h - Flag Analytics", every: 1h}

data = from(bucket: "default")
	|> range(start: -duration(v: int(v: task.every) * 2))
	|> filter(fn: (r) =>
		(r._measurement == "feature_evaluation"))
	|> filter(fn: (r) =>
		(r._field == "request_count"))
	|> group(columns: ["feature_id", "environment_id"])

data
	|> aggregateWindow(fn: sum, every: 1h)
    |> filter(fn: (r) =>
      (exists r._value))
	|> set(key: "_measurement", value: "feature_evaluation")
	|> set(key: "_field", value: "request_count")
	|> to(bucket: "default_downsampled_1h")
```

## 手动安装

如需更可配置的环境，您可以手动安装前端和API。

### 服务端API

源代码和安装指南可在[GitHub项目](https://github.com/Flagsmith/flagsmith/tree/main/api)获取。该API使用Python编写，基于Django和Django Rest框架。服务端API依赖PostgreSQL存储数据，并使用Redis作为缓存。

### 前端网站

源代码和安装指南可在[GitHub项目](https://github.com/Flagsmith/flagsmith/tree/main/frontend)获取。前端网站使用React/JavaScript编写，需要NodeJS环境。