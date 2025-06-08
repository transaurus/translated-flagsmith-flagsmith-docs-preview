---
title: Sizing and Scaling
description: Sizing and Scaling Flagsmith
sidebar_position: 80
---

## 概述

Flagsmith采用极简架构设计，在高负载场景下具有清晰可预测的性能表现。

## 前端控制台

该组件通常不会承受显著负载。如需负载均衡可不启用会话保持功能。

## API服务

API服务完全无状态设计，可近乎线性地通过负载均衡器横向扩展。例如在AWS ECS/Fargate环境中的典型配置为：

- `cpu=1024`
- `memory=2048`

对于自动伸缩策略，建议基于`ECSServiceAverageCPUUtilization`指标进行配置，将`target_value`设为`50`并设置30秒冷却时长。

## 数据库

我们建议首先通过升级单节点服务器配置进行垂直扩展。

当API集群占满数据库连接数后，可通过添加只读副本来解决数据库连接瓶颈。

同时推荐在环境中测试使用[pgBouncer](https://www.pgbouncer.org/)，该工具能有效优化数据库连接并降低数据库负载。