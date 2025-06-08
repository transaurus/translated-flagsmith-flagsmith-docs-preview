---
title: Deploying Flagsmith on AWS
description: Getting Started with Flagsmith on AWS
sidebar_label: AWS
sidebar_position: 70
---

## 概述

我们推荐使用以下AWS服务运行Flagsmith：

- 使用ECS/Fargate运行Docker镜像
- 使用RDS/Aurora/PostgreSQL作为数据库
- 使用应用负载均衡器分配流量

我们提供适用于AWS部署的[Pulumi](https://www.pulumi.com/)脚本。如有需要请联系我们。

## ECS

除非有特殊需求，我们建议运行[统一Docker镜像](https://hub.docker.com/repository/docker/flagsmith/flagsmith)。

建议参考我们的[docker-compose文件](https://github.com/Flagsmith/self-hosted/blob/main/docker-compose.yml)来设置基础环境变量。更多环境变量配置可[参见此处](locally-api.md#environment-variables)。

运行单个ECS服务时，建议至少部署两个Fargate实例以实现故障转移。有关Fargate规格的更多信息，请参阅我们的[扩展指南](/deployment/configuration/sizing-and-scaling)。

若使用健康检查，请确保将`/health`设为健康检查端点。

## RDS/Aurora

我们在生产环境使用PostgreSQL `11`版本和Aurora `3.x`发行版。首次启动时，应用会自动创建数据库模式。应用服务器升级时模式也会自动同步更新。

## 应用负载均衡器

我们通过AWS ALB将所有流量引导至对应的ECS服务。