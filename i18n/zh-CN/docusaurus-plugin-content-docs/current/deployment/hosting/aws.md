---
title: Deploying Flagsmith on AWS
description: Getting Started with Flagsmith on AWS
sidebar_label: AWS
sidebar_position: 70
---

## 概述

我们推荐使用以下AWS服务运行Flagsmith：

- 使用ECS/Fargate运行Docker镜像
- 采用RDS/Aurora/Postgres作为数据库
- 通过应用负载均衡器分配流量

我们为AWS部署提供了[Pulumi](https://www.pulumi.com/)脚本。如有需要请联系我们获取。

## AWS基础设施架构

![AWS架构图](/img/ecs-overview.svg)

## ECS服务

除非有特殊需求，我们建议运行[统一Docker镜像](https://hub.docker.com/repository/docker/flagsmith/flagsmith)。

建议参考我们的[docker-compose文件](https://github.com/Flagsmith/self-hosted/blob/main/docker-compose.yml)来配置基础环境变量，更多环境变量设置可[参见此处](locally-api.md#environment-variables)。

建议部署单个ECS服务并运行至少两个Fargate实例以实现故障转移。有关Fargate规格的更多信息，请参阅我们的[扩容指南](/deployment/configuration/sizing-and-scaling)。

如果启用健康检查，请确保使用`/health`作为健康检查端点。

## RDS/Aurora数据库

生产环境采用PostgreSQL `11`版本和Aurora `3.x`发行版。首次启动时应用将自动创建数据库模式，后续应用服务升级时模式变更也将自动完成。

## 应用负载均衡器

我们通过AWS ALB将所有流量引导至对应的ECS服务。