---
title: Troubleshooting
sidebar_label: Troubleshooting
sidebar_position: 90
---

以下是在自托管环境中设置Flagsmith时常见的一些问题。

## 健康检查

如果使用健康检查，请确保为API和前端都使用`/health`作为健康检查端点。

## API与数据库连接

在AWS中使用RDS数据库设置时最常见的问题是API应用与RDS数据库之间缺少安全组权限。您需要确保ECS/Fargate/EC2附加的安全组允许访问RDS数据库。
[AWS在此处](https://aws.amazon.com/premiumsupport/knowledge-center/ecs-task-connect-rds-database/)和[此处](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html)提供了更多详细信息。

确保在API应用中设置了`DATABASE_URL`环境变量。

## 前端>API DNS设置

如果将API和前端作为独立应用运行，需要确保前端指向API。检查[前端环境变量](/deployment/hosting/locally-frontend#environment-variables)，特别是`API_URL`。