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

等待镜像下载并运行后，访问 `http://localhost:8000/`。首次使用时需在 [http://localhost:8000/signup](http://localhost:8000/signup) 创建新账户

## 环境变量配置

除 [API](/deployment/hosting/locally-api#environment-variables) 和 [前端](/deployment/hosting/locally-frontend#environment-variables) 文档中列出的环境变量外，还可配置以下参数：

- `GUNICORN_WORKERS`: [Gunicorn 工作进程数](https://docs.gunicorn.org/en/stable/settings.html#workers)
- `GUNICORN_THREADS`: [每个工作进程的线程数](https://docs.gunicorn.org/en/stable/settings.html#threads)
- `GUNICORN_TIMEOUT`: [Gunicorn 超时秒数](https://docs.gunicorn.org/en/stable/settings.html#timeout)
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

前端通过 REST 与驱动应用的 API 通信，SDK 客户端也连接至此 API。后端采用 Django 框架和 Django REST Framework 构建。

创建账户和功能标志后，即可使用 [Flagsmith 客户端 SDK](https://github.com/Flagsmith?q=client&type=&language=)。需要为每个 SDK 重写 API 端点指向 [http://localhost:8000/api/v1/](http://localhost:8000/api/v1/)。

可通过 Django Admin 控制台获取 API 核心表的 CRUD 权限。首先需执行以下命令创建超级用户：

```bash
# Make sure you are in the root directory of this repository
docker-compose run --rm --entrypoint "python manage.py createsuperuser" api
```

创建后可通过 [http://localhost:8000/admin/](http://localhost:8000/admin/) 访问管理仪表盘

### Postgres 数据库

REST API 所有数据存储于 Postgres 数据库，升级时会通过 Django Migrations 自动执行架构变更。

## 远程访问配置

如需远程访问 Flagsmith 服务（仪表盘和API），需开放 Docker 主机端口或设置反向代理。同时需配置前端环境变量 `API_URL` 以指定 REST API 位置。