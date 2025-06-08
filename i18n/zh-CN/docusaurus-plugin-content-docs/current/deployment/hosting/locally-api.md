---
sidebar_label: API
title: Flagsmith REST API
sidebar_position: 10
---

## 初始设置

在运行应用程序之前，您需要为其配置数据库。具体操作步骤请参阅下文"数据库配置"章节。

```bash
virtualenv .venv
source .venv/bin/activate
pip install pip-tools
cd api
pip-sync requirements.txt requirements-dev.txt
python manage.py migrate
python manage.py createcachetable
python manage.py runserver --nostatic
```

完成配置后，您可以通过访问`http://<您的服务器域名:8000>/api/v1/users/config/init/`来创建初始超级用户并为安装配置DNS设置。

注意：如果在MacOS系统上安装依赖时遇到问题（特别是pyre2相关），可能需要执行以下命令：

```bash
brew install cmake re2
```

虽然推荐使用上述方式实现热重载，但您也可以选择使用Docker Compose在本地运行应用。只需在项目根目录下执行：

```bash
git clone https://github.com/Flagsmith/self-hosted.git
cd self-hosted
docker-compose up
```

## 数据库配置

数据库配置位于app/settings/\<环境\>.py文件中

本应用在所有环境中均使用PostgreSQL数据库。

本地开发时需运行PostgreSQL实例，最简单的方式是使用Docker命令：

`docker-compose -f docker/db.yaml up -d`

同时请确保开发机已设置POSTGRES_PASSWORD环境变量。

在Heroku类平台上运行时，应用会从名为`DATABASE_URL`的环境变量读取生产环境数据库连接，该变量应在平台应用配置中设置。

使用Docker运行时，应用会读取`app.settings.production`中的数据库配置。

### 读写分离

Flagsmith支持配置任意数量的只读副本。需通过设置`REPLICA_DATABASE_URLS`环境变量（包含以逗号分隔的数据库URL列表）来添加副本。

示例：

```
REPLICA_DATABASE_URLS: postgres://user:password@replica1.database.host:5432/flagsmith,postgres://user:password@replica2.database.host:5432/flagsmith
```

:::tip

若密码中包含逗号字符，请使用`REPLICA_DATABASE_URLS_DELIMITER`环境变量指定分隔符。

:::

## 初始化

本应用基于Django框架构建，内置管理后台（可通过`/admin/`访问）。您需要创建超级用户来访问管理界面或应用前端。根据部署方式选择以下创建方法：

### 本地环境

```bash
cd api
python manage.py createsuperuser
```

### 无直接控制台访问的环境（如Heroku、ECS）

应用部署后，访问`/api/v1/users/config/init/`可通过表单初始化平台数据。表单各字段说明如下：

| Parameter name  | Description                                                                                                                      |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Username        | A unique username to give the installation super user                                                                            |
| Email           | The email address to give the installation super user                                                                            |
| Password        | The password to give the installation super user                                                                                 |
| Site name       | A human readable name for the site, e.g. 'Flagsmith'                                                                             |
| Site domain[^1] | The domain that the FE of the site will be running on, e.g. app.flagsmith.com. This will be used for e.g. password reset emails. |

创建超级用户后，可通过`/admin/`登录管理后台，创建组织并将组织分配给管理员用户以开始使用应用。

更多管理后台使用说明请参阅[此文档](/deployment/configuration/django-admin)。

[^1]:
    也可通过`FLAGSMITH_DOMAIN`环境变量配置域名，完整配置变量列表请参见[应用环境变量](#application-environment-variables)章节。

## 部署指南

### 使用Heroku类平台（如Heroku、Dokku、Flynn）

该应用程序应能在任何类Heroku平台（如Dokku、Flynn）上运行，只需添加所需的Git仓库并推送代码即可。运行应用的代码包含在Procfile中。

要使其正常运行，您需要添加如下所述的必要配置变量。

### 使用ElasticBeanstalk

该应用程序可在ElasticBeanstalk中使用默认的Python设置运行。我们已包含.ebextensions/和.elasticbeanstalk/目录，这些将在ElasticBeanstalk上运行。

在您的环境中运行所需的更改如下：

`.elasticbeanstalk/config.yml` - 更新application_name和default_region为与您的设置相关的变量。

`.ebextensions/options.config` - 在项目根目录中，`generate.sh`将使用您选择的CI/CD添加所有所需的环境变量。或者，您可以添加自己的`options.config`。

### 使用Docker

如果您想运行整个Flagsmith平台，包括前端仪表板，请查看我们的[Flagsmith Docker仓库](https://github.com/Flagsmith/self-hosted)。

只需运行以下命令，即可配置应用程序使用Docker运行：

```bash
git clone https://github.com/Flagsmith/self-hosted.git
cd self-hosted
docker-compose up
```

这将使用项目根目录中`docker-compose.yml`文件创建的一些默认设置。这些设置应在任何生产环境中使用前进行更改。

Docker容器还接受一个参数，用于设置gunicorn的访问日志文件位置。默认情况下，此参数设置为/dev/null以保持gunicorn的默认行为。可以将其设置为`"-"`以将日志重定向到stdout，或根据需要设置为文件系统上的某个位置。

### 环境变量

应用程序依赖以下环境变量运行：

#### 数据库环境变量

- `DATABASE_URL`: (必填) 配置要连接的数据库。应为标准格式的数据库URL，例如postgres://user:password@host:port/db_name
- `REPLICA_DATABASE_URLS`: (可选) 配置任意数量的只读副本。应为标准格式的数据库URL的逗号分隔列表。例如postgres://user:password@replica1.db.host/flagsmith,postgres://user:password@replica2.db.host/flagsmith
- `REPLICA_DATABASE_URLS_DELIMITER`: (可选) 设置在使用`REPLICA_DATABASE_URLS`变量时用于分隔副本数据库URL的分隔符。默认为`,`。这在例如密码中包含逗号字符时非常有用。

您还可以提供如下所示的单独变量。请注意，如果定义了`DATABASE_URL`，它将优先，以下变量将被忽略。

- `DJANGO_DB_HOST`: 数据库主机名
- `DJANGO_DB_NAME`: 数据库名称
- `DJANGO_DB_USER`: 数据库用户名
- `DJANGO_DB_PASSWORD`: 数据库密码
- `DJANGO_DB_PORT`: 数据库端口

#### GitHub认证环境变量

- `GITHUB_CLIENT_ID`: 用于GitHub OAuth配置，在您的**OAuth Apps**设置中提供。
- `GITHUB_CLIENT_SECRET`: 用于GitHub OAuth配置，在您的**OAuth Apps**设置中提供。

#### 应用程序环境变量

- `ENV`: 表示当前运行环境的字符串，例如 'local'、'dev'、'prod'，默认为 'local'
- `DJANGO_SECRET_KEY`: Django 所需的密钥，若未提供将使用 `django.core.management.utils.get_random_secret_key` 自动生成。警告：若运行多个 API 实例，必须定义共享的 DJANGO_SECRET_KEY
- `LOG_LEVEL`: Django 日志级别，可选值为 `DEBUG`、`INFO`、`WARNING`、`ERROR`、`CRITICAL`
- `ACCESS_LOG_LOCATION`: 当以 Docker 容器运行时，存储 gunicorn 生成 web 日志的位置。若未设置则不存储日志。设为 `-` 时日志将输出到 `stdout`
- `DJANGO_SETTINGS_MODULE`: 当前环境对应的 settings 文件 Python 路径，例如 "app.settings.develop"
- `ENABLE_GZIP_COMPRESSION`: 是否启用 Django 的 HTTP 响应 GZIP 压缩，默认为 `False`
- `GOOGLE_ANALYTICS_KEY`: 如需 Google Analytics 支持，请填入跟踪代码
- `GOOGLE_SERVICE_ACCOUNT`: 用于访问 Google API 的服务账户 JSON 文件，需具备 analytics.readonly 权限范围
- `INFLUXDB_TOKEN`: 如需将 API 事件发送至 InfluxDB，请指定写入令牌
- `INFLUXDB_URL`: InfluxDB 数据库的 URL
- `INFLUXDB_ORG`: InfluxDB API 调用的组织字符串
- `GA_TABLE_ID`: 查询组织使用情况时的 GA 表格 ID（视图）
- `USER_CREATE_PERMISSIONS`: 使用逗号分隔的 djoser 或 rest_framework 权限列表设置新用户创建权限。可用于关闭自托管平台的公开用户注册。例如 `'djoser.permissions.CurrentUserOrAdmin'`，默认为 `'rest_framework.permissions.AllowAny'`
- `ALLOW_REGISTRATION_WITHOUT_INVITE`: 是否允许无邀请注册用户，默认为 True。设为 False 或 0 禁用。注意禁用时新用户必须通过邮件邀请注册
- `ENABLE_EMAIL_ACTIVATION`: 新用户注册是否启用邮件激活流程，默认为 False
- `SENTRY_SDK_DSN`: 使用 Sentry 时，请在此设置项目 DSN
- `SENTRY_TRACE_SAMPLE_RATE`: 浮点数。使用 Sentry 时设置追踪采样率，默认为 1.0
- `DEFAULT_ORG_STORE_TRAITS_VALUE`: 布尔值。设置此标志可使新组织默认不持久化特征值，适用于对数据敏感且不需要持久化特征的部署场景
- `OAUTH_CLIENT_ID`: Google OAuth 客户端 ID，用于通过 Google OAuth 访问 Django 管理页面。详见 [Django Admin SSO 包](https://pypi.org/project/django-admin-sso/) 了解如何通过 SSO 设置用户访问管理页面
- `OAUTH_CLIENT_SECRET`: Google OAuth 密钥，用于通过 Google OAuth 访问 Django 管理页面
- `ENABLE_ADMIN_ACCESS_USER_PASS`: 布尔值。设置此标志可启用使用用户名密码登录管理面板
- `USE_X_FORWARDED_HOST`: 布尔值，默认为 `False`。指定是否优先使用 X-Forwarded-Host 头而非 Host 头。仅在使用设置此头的代理时启用。[更多信息](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_X_FORWARDED_HOST)
- `SECURE_PROXY_SSL_HEADER_NAME`: 字符串。Django [`SECURE_PROXY_SSL_HEADER`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-proxy-ssl-header) 所查找的头部名称，默认为 `HTTP_X_FORWARDED_PROTO`
- `SECURE_PROXY_SSL_HEADER_VALUE`: 字符串。Django [`SECURE_PROXY_SSL_HEADER`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-proxy-ssl-header) 所查找的头部值，默认为 `https`
- `DJANGO_SECURE_REDIRECT_EXEMPT`: 列表。Django [`SECURE_REDIRECT_EXEMPT`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-redirect-exempt) 的透传，默认为空列表 `[]`
- `DJANGO_SECURE_REFERRER_POLICY`: 字符串。Django [`SECURE_REFERRER_POLICY`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-referrer-policy) 的透传，默认为 `same-origin`
- `DJANGO_SECURE_SSL_HOST`: 字符串。Django [`SECURE_SSL_HOST`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-ssl-host) 的透传，默认为 `None`
- `DJANGO_SECURE_SSL_REDIRECT`: 布尔值。Django [`SECURE_SSL_REDIRECT`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-ssl-redirect) 的透传，默认为 `False`
- [`APPLICATION_INSIGHTS_CONNECTION_STRING`](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview): 字符串。将 Flagsmith 遥测数据发送至 Azure Application Insights 的连接字符串
- [`OPENCENSUS_SAMPLING_RATE`](https://opencensus.io/tracing/sampling/probabilistic/): 浮点数。追踪器采样率
- `RESTRICT_ORG_CREATE_TO_SUPERUSERS`: 限制所有用户创建组织，除非其被[标记为超级用户](/deployment/configuration/django-admin#Authentication)
- `FLAGSMITH_CORS_EXTRA_ALLOW_HEADERS`: 跨域操作时允许的额外头部逗号分隔列表。例如 `'my-custom-header-1,my-custom-header-2'`，默认为 `'sentry-trace,'`
- `FLAGSMITH_DOMAIN`: 邮件通知中指向 Flagsmith 实例的自定义域名。注意：若设置此值，[初始配置](#environments-with-no-direct-console-access-eg-heroku-ecs) 中提供的域名将被忽略

#### 安全相关环境变量

- `ALLOWED_ADMIN_IP_ADDRESSES`: 限制访问Django管理控制台的IP地址列表（以逗号分隔，例如`127.0.0.1,127.0.0.2`）
- `DJANGO_ALLOWED_HOSTS`: 以逗号分隔的允许运行应用程序的主机列表
- `DJANGO_CSRF_TRUSTED_ORIGINS`: 以逗号分隔的允许不安全请求（POST、PUT）的主机列表。可用于在开发环境中允许localhost设置特征值
- `AXES_ONLY_USER_FAILURES`: 若为True，仅基于用户名锁定，当尝试次数超过限制时不会基于IP锁定。否则使用现有的IP和用户锁定逻辑。默认为`True`
- `AXES_FAILURE_LIMIT`: 允许的登录尝试次数整数阈值，超过后将创建失败登录记录。默认为`10`

#### 邮件相关环境变量

:::note

您可以不设置邮件服务器/网关而自托管Flagsmith。通过邀请链接即可邀请其他用户加入平台，系统在没有邮件功能的情况下也能正常运行。

:::

:::tip

Flagsmith使用`django_site`表提供邮件模板链接的域名。您需要配置该表中的记录指向您的域名，邮件链接才能正常工作。

:::

- `SENDER_EMAIL`: 发送邮件的发件人地址
- `EMAIL_BACKEND`: 可选以下之一：
  - `django.core.mail.backends.smtp.EmailBackend`
  - `sgbackend.SendGridBackend`
  - `django_ses.SESBackend`

若使用`django.core.mail.backends.smtp.EmailBackend`需配置：

- `EMAIL_HOST` = env("EMAIL_HOST", default='localhost')
- `EMAIL_HOST_USER` = env("EMAIL_HOST_USER", default=None)
- `EMAIL_HOST_PASSWORD` = env("EMAIL_HOST_PASSWORD", default=None)
- `EMAIL_PORT` = env("EMAIL_PORT", default=587)
- `EMAIL_USE_TLS` = env.bool("EMAIL_USE_TLS", default=True)

若使用`sgbackend.SendGridBackend`需配置：

- `SENDGRID_API_KEY`: Sendgrid账户的API密钥

若使用AWS SES需配置：

- `AWS_SES_REGION_NAME`: 使用Amazon SES作为邮件提供商时，指定包含已验证发件人邮箱的区域（如eu-central-1）。默认为us-east-1
- `AWS_SES_REGION_ENDPOINT`: SES区域端点，如email.eu-central-1.amazonaws.com。使用SES时必须配置
- `AWS_ACCESS_KEY_ID`: 使用Amazon SES时，作为SES凭证的一部分
- `AWS_SECRET_ACCESS_KEY`: 使用Amazon SES时，作为SES凭证的一部分

### API遥测

Flagsmith会收集自托管安装的相关信息，这有助于我们了解平台使用情况。这些数据_绝不会_在组织外共享，且设计上保持匿名。您可以通过设置`ENABLE_TELEMETRY`环境变量为`False`来选择退出发送这些遥测数据。

我们在每个API服务器实例启动时收集以下数据：

- 组织总数
- 项目总数
- 环境总数
- 功能总数
- 细分总数
- 用户总数
- Django的DEBUG变量
- Django的ENV变量
- API服务器外部IP地址

### 创建密钥

务必在您使用的平台上设置 `DJANGO_SECRET_KEY` 环境变量。若未设置，Django 会在每次应用启动时自动生成一个密钥，但这会导致意外行为，因为该密钥被 Django 用于会话令牌等数据的加密。为避免此类问题，请创建并设置 `DJANGO_SECRET_KEY` 变量。Django 建议该密钥长度至少为 50 个字符，但具体配置方式由您决定。如需了解密钥格式详情，可参考 Django 源码中的 `get_random_secret_key()` 方法。

### StatsD 集成

应用通过 Python 的 gunicorn 运行，因此可配置其将 statsd 指标发送至指定主机用于监控。使用我们的 Docker 镜像时，可通过以下环境变量进行配置：

- `STATSD_HOST`: 接收 statsd 指标的主机 URL
- `STATSD_PORT`: 可选定义主机监听 statsd 指标的端口（默认：8125）
- `STATSD_PREFIX`: 可选定义 statsd 指标的前缀（默认：flagsmith.api）

以下是配合 Datadog 使用 statsd 的 Docker Compose 配置示例。注意必须将 `DD_DOGSTATSD_NON_LOCAL_TRAFFIC` 环境变量设为 `true`，以确保 Datadog 代理能接收外部服务的指标。

```yaml
version: '3'
services:
 postgres:
  image: postgres:11.12-alpine
  environment:
   POSTGRES_PASSWORD: password
   POSTGRES_DB: flagsmith
  container_name: flagsmith_postgres
 api:
  build:
   dockerfile: Dockerfile
   context: ../../api
  environment:
   DATABASE_URL: postgres://postgres:password@postgres:5432/flagsmith
   DJANGO_SETTINGS_MODULE: app.settings.local
   STATSD_HOST: datadog
  ports:
   - '8000:8000'
  depends_on:
   - postgres
  links:
   - postgres
   - datadog
 datadog:
  image: gcr.io/datadoghq/agent:7
  environment:
   - DD_API_KEY=<API KEY>
   - DD_SITE=datadoghq.eu
   - DD_DOGSTATSD_NON_LOCAL_TRAFFIC=true
  volumes:
   - /var/run/docker.sock:/var/run/docker.sock
   - /proc/:/host/proc/:ro
   - /sys/fs/cgroup:/host/sys/fs/cgroup:ro
   - /var/lib/docker/containers:/var/lib/docker/containers:ro
```

若非通过 Docker 运行应用，可参阅 gunicorn 的 [statsd 指标文档](https://docs.gunicorn.org/en/stable/instrumentation.html)

## 缓存机制

应用在以下场景使用缓存：

1. 环境认证 - 所有使用 X-Environment-Key 请求头的端点会缓存环境对象，默认使用内存缓存。可通过下文配置项调整。
2. 环境特征标志 - 调用 /flags 接口时，返回的特征标志会缓存在内存中，缓存时长通过 `"CACHE_FLAGS_SECONDS"` 环境变量配置。
3. 项目细分 - 项目细分数据会缓存在内存中，缓存时长通过 `"CACHE_PROJECT_SEGMENTS_SECONDS"` 环境变量配置。
4. 特征标志与身份端点缓存 - 支持缓存 GET /flags 和 GET /identities 接口响应，具体缓存策略配置详见下文。

### 特征标志与身份端点缓存

要启用这两个端点（仅 GET 请求）的缓存，需设置以下环境变量：

| Environment Variable                                               | Description                                                                                                                    | Example value                                          | Default                                       |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ | --------------------------------------------- |
| <code>GET\_[FLAGS&#124;IDENTITIES]\_ENDPOINT_CACHE_SECONDS</code>  | Number of seconds to cache the response to `GET /api/v1/flags`                                                                 | `60`                                                   | `0`                                           |
| <code>GET\_[FLAGS&#124;IDENTITIES]\_ENDPOINT_CACHE_BACKEND</code>  | Python path to the django cache backend chosen. See documentation [here](https://docs.djangoproject.com/en/3.2/topics/cache/). | `django.core.cache.backends.memcached.PyMemcacheCache` | `django.core.cache.backends.dummy.DummyCache` |
| <code>GET\_[FLAGS&#124;IDENTITIES]\_ENDPOINT_CACHE_LOCATION</code> | The location for the cache. See documentation [here](https://docs.djangoproject.com/en/3.2/topics/cache/).                     | `127.0.0.1:11211`                                      | `get_flags_endpoint_cache`                    |

示例配置：在位于 `memcached-container` 的 memcached 实例中缓存特征标志和身份请求 30 秒：

```
GET_FLAGS_ENDPOINT_CACHE_SECONDS: 30
GET_FLAGS_ENDPOINT_CACHE_BACKEND: django.core.cache.backends.memcached.PyMemcacheCache
GET_FLAGS_ENDPOINT_CACHE_LOCATION: memcached-container:11211
GET_IDENTITIES_ENDPOINT_CACHE_SECONDS: 30
GET_IDENTITIES_ENDPOINT_CACHE_BACKEND: django.core.cache.backends.memcached.PyMemcacheCache
GET_IDENTITIES_ENDPOINT_CACHE_LOCATION: memcached-container:11211
```

### 环境认证缓存

每次使用 X-Environment-Key 请求头时，Flagsmith 会检索环境对象进行缓存。可通过环境变量配置共享缓存及更长超时时间。当平台中环境发生变更时，缓存会自动清除。

| Environment Variable         | Description                                                                                                                    | Example value                                          | Default                                       |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ | --------------------------------------------- |
| `ENVIRONMENT_CACHE_SECONDS`  | Number of seconds to cache the environment for                                                                                 | `60`                                                   | `86400` ( = 24h)                              |
| `ENVIRONMENT_CACHE_BACKEND`  | Python path to the django cache backend chosen. See documentation [here](https://docs.djangoproject.com/en/3.2/topics/cache/). | `django.core.cache.backends.memcached.PyMemcacheCache` | `django.core.cache.backends.dummy.DummyCache` |
| `ENVIRONMENT_CACHE_LOCATION` | The location for the cache. See documentation [here](https://docs.djangoproject.com/en/3.2/topics/cache/).                     | `127.0.0.1:11211`                                      | `environment-objects`                         |

## 前后端统一构建

可通过统一构建将 Flagsmith 作为单一应用/Docker 容器运行。这些构建版本已在 [Docker Hub](https://hub.docker.com/repository/docker/flagsmith/flagsmith) 发布，也可按以下步骤将前端集成到 Django 应用中运行：

1. `cd frontend; npm run bundledjango`
2. `cd ../api; python manage.py collectstatic`
3. `python manage.py runserver`

### 工作原理

Webpack会编译前端构建，从`api/app/templates/index.html`获取源文件。它将编译后的JS和CSS资源放置到`api/static`目录下，然后将带注释的`index.html`页面复制到`api/app/templates/webpack/index.html`。

接着，Django的`collectstatic`命令会复制Django所需的所有其他静态资源，包括`api/app/templates/webpack/index.html`，到`api/static`目录中。

## 项目开发人员须知

### 技术栈

- Python
- Django
- Django Rest Framework

### 贡献者开发环境

我们使用[pip-tools](https://github.com/jazzband/pip-tools)来管理包和依赖项。

升级包或添加新包的方法：

```bash
pip install -r requirements-dev.txt
pip-compile
```

### 使用pip-tools管理依赖

我们使用[pip-tools](https://github.com/jazzband/pip-tools)来管理依赖项。

要向项目添加新库，请编辑requirements.in文件，然后执行：

```bash
# This step will overwrite requirements.txt
pip-compile requirements.in
pip install -r requirements.txt
```