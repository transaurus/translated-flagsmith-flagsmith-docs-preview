---
title: Deploying Flagsmith on Google Cloud
sidebar_label: Google Cloud
sidebar_position: 75
---

## 概述

我们推荐在Google云平台上运行Flagsmith，使用以下服务：

- [Cloud Run](https://cloud.google.com/run) 作为应用服务器
- [Cloud SQL/Postgres](https://cloud.google.com/sql/postgresql) 作为数据库

## Cloud Run

除非有特殊需求，我们建议运行[统一Docker镜像](https://hub.docker.com/repository/docker/flagsmith/flagsmith)。

建议参考我们的[docker-compose文件](https://github.com/Flagsmith/self-hosted/blob/main/docker-compose.yml)来设置基础环境变量。更多环境变量配置可[参阅此处](locally-api.md#environment-variables)。

运行单个Cloud Run服务时，建议至少部署两个容器实例以实现故障转移。关于容量规划的详细信息，请参阅[扩展指南](/deployment/configuration/sizing-and-scaling)。为避免冷启动问题（特别是需要为SDK提供低延迟响应时），建议配置至少[2个最小实例](https://cloud.google.com/run/docs/configuring/min-instances)。

若使用健康检查，请确保为API和前端都配置`/health`作为健康检查端点。

## Cloud SQL/Postgres

我们支持Postgres `11+`版本。我们的SaaS平台在生产环境运行的是PostgreSQL `11`版本。初次启动时，应用会自动创建数据库模式。应用服务器升级时，模式变更也会自动完成。