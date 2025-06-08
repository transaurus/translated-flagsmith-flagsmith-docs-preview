---
title: Sizing and Scaling
description: Sizing and Scaling Flagsmith
sidebar_position: 80
---

## 概述

Flagsmith采用极简架构设计，在高负载场景下仍能保持清晰可控的服务能力。

## 前端仪表盘

该组件通常不会承受高负载压力。如需扩展可通过负载均衡实现，且无需会话保持功能。

## API服务

API服务采用完全无状态设计，可完美配合负载均衡横向扩展。例如在AWS ECS/Fargate环境运行时我们采用以下配置：

- `cpu=1024`
- `memory=2048`

关于自动扩展策略，建议基于`ECSServiceAverageCPUUtilization`监控指标，设置`target_value`为`50`并配置30秒冷却时长。

## 数据库

我们建议首先通过升级单机配置进行垂直扩展。

当API集群占满数据库连接数后，可通过添加只读副本来解决数据库连接瓶颈问题。

同时推荐在环境中测试使用[pgBouncer](https://www.pgbouncer.org/)，该工具能有效优化数据库连接管理并降低数据库负载。