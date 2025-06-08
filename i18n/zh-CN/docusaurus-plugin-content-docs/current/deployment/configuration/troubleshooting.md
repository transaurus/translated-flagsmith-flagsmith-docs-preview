---
title: Troubleshooting
sidebar_label: Troubleshooting
sidebar_position: 90
---

以下是在自托管环境中设置Flagsmith时常见的一些问题。

## 健康检查

如果使用健康检查功能，请确保为API和前端都配置`/health`作为健康检查端点。

## API与数据库连接

在AWS中使用RDS数据库时，最常见的问题是API应用与RDS数据库之间的安全组权限缺失。您需要确保ECS/Fargate/EC2实例所关联的安全组允许访问RDS数据库。
[AWS相关文档可参考此处](https://aws.amazon.com/premiumsupport/knowledge-center/ecs-task-connect-rds-database/)及[此处](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html)。

请确认已在API应用中设置`DATABASE_URL`环境变量。

## 前端>API的DNS配置

若将API与前端作为独立应用运行，需确保前端正确指向API地址。请检查[前端环境变量](/deployment/hosting/locally-frontend#environment-variables)，特别是`API_URL`参数。