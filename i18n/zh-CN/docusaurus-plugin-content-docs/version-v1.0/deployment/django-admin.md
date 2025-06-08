---
sidebar_label: Django Admin
title: Django Admin
sidebar_position: 100
---

# Django 管理后台

由于该应用基于 Django 构建，因此天然支持 Django 管理后台功能。Flagsmith 专门利用 Django 管理站点来处理平台的特定功能。如果您正在自托管部署，在某些场景下访问这些管理页面会非常有用。

## 身份验证

管理后台页面仅对标记为"超级用户"的账户开放。该权限只能在平台初始设置时或通过数据库操作授予。如果是首次使用，您可以按照[此处指南](/deployment/hosting/locally-api#Initialising)操作；否则需要通过数据库为用户设置`is_staff`和`is_superuser`权限标志位。

获得超级用户权限后，您可通过`/admin`路径访问管理后台。系统将提示您使用任意超级用户凭证登录。

## 管理页面

### 组织管理

最重要的配置页面当属平台组织管理模块。在管理后台首页中段位置，您会看到`Organisations`链接入口。通过这些页面，您可以按需管理平台上的所有组织。例如，SAML 认证配置数据就必须通过这些页面进行设置。