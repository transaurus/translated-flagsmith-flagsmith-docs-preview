---
title: Deploying Flagsmith on Google Cloud
sidebar_label: Google Cloud
sidebar_position: 75
---

## 概述

我们推荐使用以下Google云平台服务运行Flagsmith：

- [Cloud Run](https://cloud.google.com/run) 作为应用服务器
- [Cloud SQL/Postgres](https://cloud.google.com/sql/postgresql) 作为数据库

## Cloud Run

除非有特殊需求，我们建议运行[统一Docker镜像](https://hub.docker.com/repository/docker/flagsmith/flagsmith)。

建议参考我们的[docker-compose文件](https://github.com/Flagsmith/self-hosted/blob/main/docker-compose.yml)来配置基础环境变量，更多环境变量配置可[参阅此处](locally-api.md#environment-variables)。

运行单个Cloud Run服务时，建议至少保持两个容器实例以实现故障转移。关于容量规划详情，请参阅我们的[扩展指南](/deployment/configuration/sizing-and-scaling)。为规避冷启动问题（特别是需要为SDK提供低延迟响应时），建议配置至少[2个最小实例](https://cloud.google.com/run/docs/configuring/min-instances)。

若启用健康检查，请确保对API和前端均使用`/health`作为健康检查端点。

## Cloud SQL/Postgres

我们支持Postgres `11+`版本。SaaS生产环境运行于PostgreSQL 11版本。初次启动时，应用将自动创建数据库模式，后续应用服务器升级时模式变更也将自动完成。