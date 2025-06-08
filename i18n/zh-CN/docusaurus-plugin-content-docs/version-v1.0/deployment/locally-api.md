---
sidebar_label: API
title: Flagsmith REST API
sidebar_position: 10
---

## 开发环境

在运行应用程序之前，您需要为其配置数据库。具体操作步骤请参阅下文"数据库"章节。

```bash
virtualenv .venv
source .venv/bin/activate
pip install pip-tools
cd api
pip-sync requirements.txt requirements-dev.txt
python manage.py migrate
python manage.py runserver --nostatic
```

现在您可以访问 `http://<您的服务器域名:8000>/api/v1/users/config/init/` 来创建初始超级用户并为您的安装配置DNS设置。

注意：如果在MacOS上运行并遇到依赖安装问题（特别是pyre2相关），可能需要执行以下命令：

```bash
brew install cmake re2
```

如需使用Docker Compose本地运行应用程序，可在项目根目录执行以下命令。但建议采用上述方式运行以获得热重载功能：

```bash
git clone https://github.com/Flagsmith/self-hosted.git
cd self-hosted
docker-compose up
```

## 数据库

数据库配置位于app/settings/\<环境\>.py文件中

应用程序默认在所有环境中使用PostgreSQL数据库。

本地运行时，您需要运行本地PostgreSQL实例。最简单的方式是使用Docker，执行以下命令即可：

`docker-compose -f docker/db.yaml up -d`

同时需确保开发机上设置了POSTGRES_PASSWORD环境变量。

在Heroku类平台上运行时，应用程序会从名为`DATABASE_URL`的环境变量读取生产环境数据库连接，该变量应在Heroku类应用配置中设置。

使用Docker运行时，应用程序会读取`app.settings.production`中的数据库配置。

## 初始化

本应用基于django框架构建，自带可通过`/admin/`访问的管理界面。要使用这些功能，您需要创建超级用户。该用户既可用于访问管理界面，也可在前端应用运行时直接登录系统。根据您的安装方式，可通过以下指引创建用户：

### 本地环境

```bash
cd api
python manage.py createsuperuser
```

### 无直接控制台访问的环境（如Heroku、ECS）

应用部署后，可通过访问`/api/v1/users/config/init/`初始化安装。该页面将显示基础表单用于设置平台的初始数据，表单各参数说明如下：

| Parameter name | Description                                                                                                                      |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Username       | A unique username to give the installation super user                                                                            |
| Email          | The email address to give the installation super user                                                                            |
| Password       | The password to give the installation super user                                                                                 |
| Site name      | A human readable name for the site, e.g. 'Flagsmith'                                                                             |
| Site domain    | The domain that the FE of the site will be running on, e.g. app.flagsmith.com. This will be used for e.g. password reset emails. |

创建超级用户后，您可使用该凭证登录`/admin/`。在此可以创建组织，并选择新建用户或将组织分配给管理员用户以开始使用应用。

更多关于管理界面的信息请参阅[此处](/deployment/configuration/django-admin)。

## 部署

### 使用Heroku类平台（如Heroku、Dokku、Flynn）

本应用可通过简单的git仓库添加和代码推送运行在任何Heroku类平台（如Dokku、Flynn）上。运行逻辑包含在Procfile中。

要成功运行，您需要按照下文说明配置必要的环境变量。

### 使用ElasticBeanstalk

应用可在ElasticBeanstalk上通过默认Python设置运行。我们已包含.ebextensions/和.elasticbeanstalk/目录，这些配置可直接在ElasticBeanstalk上运行。

根据您的环境需要进行的修改如下：

`.elasticbeanstalk/config.yml` - 将 application_name 和 default_region 更新为您环境对应的变量值。

`.ebextensions/options.config` - 在项目根目录下，`generate.sh` 脚本会通过您选择的 CI/CD 流程自动注入所有必需的环境变量。您也可以手动添加自己的 `options.config` 文件。

### 使用 Docker

如需运行完整的 Flagsmith 平台（包括前端仪表盘），请参阅我们的 [Flagsmith Docker 仓库](https://github.com/Flagsmith/self-hosted)。

只需运行以下命令即可通过 Docker 配置并启动应用：

```bash
git clone https://github.com/Flagsmith/self-hosted.git
cd self-hosted
docker-compose up
```

这将使用项目根目录下 `docker-compose.yml` 文件中预设的默认配置。在生产环境中使用前请务必修改这些配置。

Docker 容器还支持通过参数设置 gunicorn 的访问日志存储路径。默认值为 /dev/null 以保持 gunicorn 的原始行为。可设置为 `"-"` 将日志重定向到标准输出，或指定为文件系统路径。

### 环境变量

应用运行依赖以下环境变量：

#### 数据库环境变量

- `DATABASE_URL`: 开发和生产环境必需，需为标准格式的数据库连接字符串，例如 postgres://用户:密码@主机:端口/数据库名

您也可以单独配置以下变量。注意：若已定义 `DATABASE_URL`，则以下变量将被忽略。

- `DJANGO_DB_HOST`: 数据库主机名
- `DJANGO_DB_NAME`: 数据库名称  
- `DJANGO_DB_USER`: 数据库用户名
- `DJANGO_DB_PASSWORD`: 数据库密码
- `DJANGO_DB_PORT`: 数据库端口

#### 应用环境变量

- `ENV`: 字符串表示当前运行环境，例如 'local'、'dev'、'prod'，默认为 'local'
- `DJANGO_SECRET_KEY`: Django 所需的密钥，若未提供则会通过 `django.core.management.utilsget_random_secret_key` 自动生成
- `LOG_LEVEL`: Django 日志级别，可选值为 `DEBUG`、`INFO`、`WARNING`、`ERROR`、`CRITICAL`
- `ACCESS_LOG_LOCATION`: Docker 容器运行时存储 gunicorn 生成的 Web 日志的位置。若未设置则不存储日志，设为 `-` 则输出到 `stdout`
- `DJANGO_SETTINGS_MODULE`: 当前环境对应的 Python 设置文件路径，例如 "app.settings.develop"
- `ENABLE_GZIP_COMPRESSION`: 是否启用 Django 的 HTTP 响应 Gzip 压缩，默认为 `False`
- `GOOGLE_ANALYTICS_KEY`: 如需 Google Analytics 支持，请填入跟踪代码
- `GOOGLE_SERVICE_ACCOUNT`: 访问 Google API 的服务账号 JSON 文件，用于获取组织使用情况 - 需具备 analytics.readonly 权限
- `INFLUXDB_TOKEN`: 如需将 API 事件发送至 InfluxDB，请指定写入令牌
- `INFLUXDB_URL`: InfluxDB 数据库 URL
- `INFLUXDB_ORG`: InfluxDB API 调用的组织标识符
- `GA_TABLE_ID`: 查询组织使用情况时的 GA 表格 ID（视图）
- `USER_CREATE_PERMISSIONS`: 通过逗号分隔的 djoser 或 rest_framework 权限列表设置新用户创建权限。可用于关闭自托管环境的公开用户注册，例如 `'djoser.permissions.CurrentUserOrAdmin'`，默认为 `'rest_framework.permissions.AllowAny'`
- `ALLOW_REGISTRATION_WITHOUT_INVITE`: 是否允许无邀请注册用户，默认为 True。设为 False 或 0 禁用。注意禁用后新用户必须通过邮件邀请注册
- `ENABLE_EMAIL_ACTIVATION`: 新用户注册是否启用邮件激活流程，默认为 False
- `SENTRY_SDK_DSN`: 使用 Sentry 时请设置项目 DSN
- `SENTRY_TRACE_SAMPLE_RATE`: 浮点数。使用 Sentry 时设置跟踪采样率，默认为 1.0
- `DEFAULT_ORG_STORE_TRAITS_VALUE`: 布尔值。设置此标志可使新组织默认不持久化特征，适用于对数据敏感且不需要持久化特征的部署场景
- `OAUTH_CLIENT_ID`: 用于通过 Google OAuth 访问 Django 管理页面的客户端 ID。参见 [Django Admin SSO 包](https://pypi.org/project/django-admin-sso/) 了解如何配置用户通过 SSO 访问管理页面
- `OAUTH_CLIENT_SECRET`: 用于通过 Google OAuth 访问 Django 管理页面的客户端密钥
- `ENABLE_ADMIN_ACCESS_USER_PASS`: 布尔值。设置此标志可启用通过用户名密码登录管理面板
- `USE_X_FORWARDED_HOST`: 布尔值。默认为 `False`。指定是否优先使用 X-Forwarded-Host 头而非 Host 头。仅当使用设置此头的代理时应启用。[更多信息](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_X_FORWARDED_HOST)
- `SECURE_PROXY_SSL_HEADER_NAME`: 字符串。Django [`SECURE_PROXY_SSL_HEADER`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-proxy-ssl-header) 所查找的头部名称，默认为 `HTTP_X_FORWARDED_PROTO`
- `SECURE_PROXY_SSL_HEADER_VALUE`: 字符串。Django [`SECURE_PROXY_SSL_HEADER`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-proxy-ssl-header) 所查找的头部值，默认为 `https`
- `DJANGO_SECURE_REDIRECT_EXEMPT`: 列表。透传 Django 的 [`SECURE_REDIRECT_EXEMPT`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-redirect-exempt)，默认为空列表 `[]`
- `DJANGO_SECURE_REFERRER_POLICY`: 字符串。透传 Django 的 [`SECURE_REFERRER_POLICY`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-referrer-policy)，默认为 `same-origin`
- `DJANGO_SECURE_SSL_HOST`: 字符串。透传 Django 的 [`SECURE_SSL_HOST`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-ssl-host)，默认为 `None`
- `DJANGO_SECURE_SSL_REDIRECT`: 布尔值。透传 Django 的 [`SECURE_SSL_REDIRECT`](https://docs.djangoproject.com/en/4.0/ref/settings/#secure-ssl-redirect)，默认为 `False`
- [`APPLICATION_INSIGHTS_CONNECTION_STRING`](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview): 字符串。用于配置 Flagsmith 向 Azure Application Insights 发送遥测数据的连接字符串
- [`OPENCENSUS_SAMPLING_RATE`](https://opencensus.io/tracing/sampling/probabilistic/): 浮点数。跟踪器采样率

#### 安全相关环境变量

- `ALLOWED_ADMIN_IP_ADDRESSES`: 限制访问Django管理控制台的IP地址列表（以逗号分隔，例如`127.0.0.1,127.0.0.2`）
- `DJANGO_ALLOWED_HOSTS`: 指定应用在特定环境下可运行的主机列表（以逗号分隔）
- `DJANGO_CSRF_TRUSTED_ORIGINS`: 允许发起非安全请求（POST/PUT）的主机列表（以逗号分隔）。适用于开发环境中允许localhost设置特征值。
- `AXES_ONLY_USER_FAILURES`: 若为True，仅基于用户名锁定，超过尝试次数后永不基于IP锁定。否则使用现有的IP和用户锁定逻辑。默认为`True`。
- `AXES_FAILURE_LIMIT`: 允许的登录尝试失败次数阈值，超过后将创建失败记录。默认为`10`。

#### 邮件相关环境变量

:::note

您可以在不配置邮件服务器/网关的情况下自托管Flagsmith。通过邀请链接添加平台用户，系统无需邮件功能即可正常运行。

:::

:::tip

Flagsmith使用`django_site`表提供邮件模板链接的域名。需配置该表中的记录指向您的域名，邮件链接才能生效。

:::

- `SENDER_EMAIL`: 发送邮件的发件地址
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

- `AWS_SES_REGION_NAME`: 使用Amazon SES时指定包含已验证发件邮箱的区域（如eu-central-1）。默认为us-east-1
- `AWS_SES_REGION_ENDPOINT`: SES区域端点（如email.eu-central-1.amazonaws.com）。使用SES时必须配置
- `AWS_ACCESS_KEY_ID`: 使用Amazon SES时作为凭证组成部分
- `AWS_SECRET_ACCESS_KEY`: 使用Amazon SES时作为凭证组成部分

### API遥测

Flagsmith会收集自托管实例的使用信息以分析平台使用情况。这些数据_绝不_对外共享，且设计上保持匿名。可通过设置`ENABLE_TELEMETRY`环境变量为`False`来禁用遥测功能。

每个API服务器实例启动时收集以下数据：

- 组织总数
- 项目总数
- 环境总数
- 功能总数
- 用户细分总数
- 用户总数
- Django的DEBUG变量值
- Django的ENV变量值
- API服务器外部IP地址

### 创建密钥

务必在您所使用的平台上设置 `DJANGO_SECRET_KEY` 环境变量。若未设置，Django 会在每次应用启动时自动生成一个密钥，但这会导致意外行为，因为该密钥被 Django 用于加密会话令牌等数据。为避免这些问题，请手动设置 `DJANGO_SECRET_KEY` 变量。Django 建议该密钥长度至少为 50 个字符，但具体配置方式由您决定。如需了解密钥格式详情，可参考 Django 源码中的 `get_random_secret_key()` 方法。

## 运行测试

本应用使用 pytest 编写（合理运用 fixtures）和运行测试。运行测试前请确保 `DJANGO_SETTINGS_MODULE` 环境变量指向正确的模块，例如 `app.settings.test`。

运行测试：

```bash
DJANGO_SETTINGS_MODULE=app.settings.test pytest
```

## 预提交钩子

项目通过预提交配置（`.pre-commit-config.yaml`）在提交前自动运行 `black`、`flake8` 和 `isort` 代码格式化工具。

安装预提交钩子：

```bash
# From the repository root
pip install pre-commit
pre-commit install
```

也可手动对整个代码库运行所有检查：

```bash
pre-commit run --all-files
```

## 添加依赖

添加 Python 依赖时，请将其当前版本号写入 requirements.txt 或 requirements-dev.txt。

## 缓存机制

应用在以下场景使用缓存：

1. 环境认证 - 所有使用 X-Environment-Key 请求头的端点会缓存环境对象至内存
2. 环境特征标志 - 调用 /flags 接口返回的特征标志会缓存至内存，缓存时长可通过环境变量 `"CACHE_FLAGS_SECONDS"` 配置
3. 项目用户分群 - 项目分群数据会缓存至内存，缓存时长可通过环境变量 `"CACHE_PROJECT_SEGMENTS_SECONDS"` 配置

## 前后端统一构建

您可通过统一构建将 Flagsmith 作为单一应用/Docker 容器运行。这些构建已发布至 [Docker Hub](https://hub.docker.com/repository/docker/flagsmith/flagsmith)，也可将前端作为 Django 应用的一部分运行。步骤如下：

1. `cd frontend; npm run bundledjango`
2. `cd ../api; python manage.py collectstatic`
3. `python manage.py runserver`

### 实现原理

Webpack 编译前端构建时，会以 `api/app/templates/index.html` 为模板，将编译后的 JS 和 CSS 资源输出至 `api/static` 目录，同时将注解后的 `index.html` 复制到 `api/app/templates/webpack/index.html`。

Django 的 `collectstatic` 命令随后将所有静态资源（包括 `api/app/templates/webpack/index.html`）收集至 `api/static` 目录。

## 开发者指南

### 技术栈

- Python
- Django
- Django Rest Framework

### 贡献者开发环境

我们使用 [pip-tools](https://github.com/jazzband/pip-tools) 管理包依赖。

升级或新增依赖包：

```bash
pip install -r requirements-dev.txt
pip-compile
```

### pip-tools 依赖管理

本项目使用 [pip-tools](https://github.com/jazzband/pip-tools) 管理依赖关系。

添加新库时，请先编辑 requirements.in 文件，然后执行：

```bash
# This step will overwrite requirements.txt
pip-compile requirements.in
pip install -r requirements.txt
```