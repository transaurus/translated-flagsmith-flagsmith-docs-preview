---
sidebar_label: Django Admin
title: Django Admin
sidebar_position: 100
---

# Django 管理后台

由于应用基于 Django 构建，因此天然支持 Django 管理后台功能。Flagsmith 平台部分功能正是通过 Django 管理站点实现的。如果您正在自托管部署，在某些场景下访问这些管理页面会很有帮助。

## 身份验证

管理后台页面仅对标记为"超级用户"的账户开放。该权限只能在平台初始化时或通过数据库操作进行设置。初次部署时，您可以按照[此处指南](/deployment/hosting/locally-api#Initialising)操作；对于已部署环境，则需要通过数据库为用户设置`is_staff`和`is_superuser`权限标志。

获得超级用户权限后，您可通过`/admin/`路径访问管理后台，系统将提示使用超级用户凭证登录。

:::info[若登录页面仅显示"使用SSO登录"选项]
您可能需要设置`ENABLE_ADMIN_ACCESS_USER_PASS`环境变量。具体说明请参阅[环境变量列表](http://localhost:3000/deployment/hosting/locally-api#application-environment-variables)。:::

## 管理功能页面

### 组织管理

平台配置中最常使用的当属组织管理功能。在管理后台首页中段位置可找到`Organisations`入口，通过该页面可对平台上的组织进行全方位管理。例如，SAML 配置数据就必须通过这些页面进行设置。