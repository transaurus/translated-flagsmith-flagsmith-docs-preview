---
description: Getting Started with Flagsmith on Docker
sidebar_position: 40
---

# Docker 部署

您可以使用 Docker 在本地搭建完整的 [Flagsmith 功能标志](https://www.flagsmith.com) 环境。只需克隆 [docker 代码库](https://github.com/Flagsmith/self-hosted) 并运行 docker-compose：

```bash
git clone https://github.com/Flagsmith/self-hosted.git
cd self-hosted
docker-compose up
```

等待镜像下载并运行后，访问 `http://localhost:8000/`。首先您需要在 [http://localhost:8000/signup](http://localhost:8000/signup) 创建新账户

## 环境变量

除了 [API](/deployment/hosting/locally-api#environment-variables) 和 [前端](/deployment/hosting/locally-frontend#environment-variables) 中指定的环境变量外，您还可以配置以下参数：

- `GUNICORN_WORKERS`: 创建的 [Gunicorn 工作进程](https://docs.gunicorn.org/en/stable/settings.html#workers) 数量
- `GUNICORN_THREADS`: 每个工作进程的 [Gunicorn 线程数](https://docs.gunicorn.org/en/stable/settings.html#threads)
- `GUNICORN_TIMEOUT`: [Gunicorn 超时](https://docs.gunicorn.org/en/stable/settings.html#timeout) 秒数
- `ACCESS_LOG_LOCATION`: 访问日志写入路径

## 平台架构支持

我们的 Docker 镜像支持以下 CPU 架构：

- `amd64`
- `linux/arm64`
- `linux/arm/v7`

## 架构说明

docker-compose 文件会运行以下容器：

### 前端仪表盘与 REST API 整合服务 - 端口 8000

网页用户界面采用 node.js 和 React 开发，用于创建账户和管理功能标志。

网页界面通过 REST 与驱动应用的 API 通信，SDK 客户端也连接至此 API。该 API 使用 Django 和 Django REST 框架开发。

创建账户和功能标志后，您可以使用任一 [Flagsmith 客户端 SDK](https://github.com/Flagsmith?q=client&type=&language=) 接入 API。需要将每个 SDK 的 API 端点覆盖指向 [http://localhost:8000/api/v1/](http://localhost:8000/api/v1/)。

您可以访问 Django 管理控制台获取对 API 核心表的 CRUD 权限。首先需要执行以下命令创建超级用户账户：

```bash
# Make sure you are in the root directory of this repository
docker-compose run --rm --entrypoint "python manage.py createsuperuser" api
```

然后即可通过 [http://localhost:8000/admin/](http://localhost:8000/admin/) 访问管理仪表盘

### Postgres 数据库

REST API 将所有数据存储在 Postgres 数据库中。升级时会通过 Django Migrations 自动执行架构变更。

## 远程访问 Flagsmith

您需要开放 Docker 主机的端口或设置反向代理来访问两项 Flagsmith 服务（仪表盘和 API）。同时需要配置仪表盘环境变量 `API_URL`，该变量用于指定 REST API 的位置。