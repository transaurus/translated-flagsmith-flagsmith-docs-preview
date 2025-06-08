---
sidebar_position: 3
---

# 版本对比

您可以自由选择运行Flagsmith的开源版本！以下是开源版、SaaS托管版和企业版之间的主要区别：

- 开源版本**没有**API请求或身份标识限制——您可以根据需要在集群中运行任意数量的API实例。
- 开源版本**没有**仪表盘用户数量限制——团队成员数量不受限制。
- SaaS版和企业版支持[变更请求与功能标志调度](advanced-use/change-requests.md)。
- SaaS版和企业版提供[基于角色的访问控制](advanced-use/permissions.md)。
- SaaS版和企业版额外支持以下[认证提供商](/deployment/configuration/enterprise-edition)：
  - Okta
  - LDAP
  - SAML
  - AD FS

## SaaS版优势

我们的SaaS平台具有以下核心优势：

- 可立即快速部署使用
- 全球分布的[边缘API](advanced-use/edge-api.md)确保低延迟特性交付，全球范围内标志请求响应时间控制在150毫秒内
- 实时推送标志变更至客户端，仪表盘修改即刻生效
- 由我们负责平台升级、安全补丁、扩展备份等运维工作

### SaaS架构

![架构图](/img/saas-architecture.svg)

## 企业版优势

企业版支持本地化部署，也可为您组织提供专属私有云实例。

- [基于角色的访问控制](advanced-use/permissions.md)
- 支持[SAML、LDAP、ADFS和Okta认证](deployment/configuration/authentication.md)，并可锁定单一认证提供商
- 额外数据库引擎支持：Oracle、SQL Server和MySQL
- 下文详述的更多部署与编排选项

## 开源版优势

- 完全免费！
- 开源版本**没有**API请求或身份标识限制——集群中可无限扩展API实例
- 开源版本**没有**仪表盘用户数量限制——团队成员数量不受限制
- 支持一键部署至多种[IaaS和PaaS提供商](deployment/overview#one-click-installers)