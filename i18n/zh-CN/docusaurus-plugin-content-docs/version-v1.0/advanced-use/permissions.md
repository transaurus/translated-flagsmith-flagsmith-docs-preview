---
title: Permissions / Role Based Access Control
description: Team member and group permissions.
---

Flagsmith提供细粒度的权限控制功能，帮助大型团队跨项目和环境管理访问权限与角色分配。

权限可分配给单个团队成员或群组。

:::caution

权限/基于角色的访问控制功能不属于开源版本。如需在自托管/本地部署方案中使用这些功能，请[联系我们](https://flagsmith.com/contact-us/)。

:::

## 群组管理

群组是管理多成员权限的便捷方式。群组可包含任意数量的团队成员，您可以在「组织设置」页面创建群组。

## 组织架构

团队成员可被定义为组织管理员或普通用户。组织管理员实质上是超级用户角色，拥有对该组织内所有项目、环境、功能开关、远程配置和用户分段的完全读写权限。

非管理员的普通用户需在相应层级手动分配权限。组织层级的可用权限如下所述。

| **Role**       | **Ability**                                                  |
| -------------- | ------------------------------------------------------------ |
| Create Project | Allows the user to create projects in the given Organisation |

![Image](/img/organisation-permissions.png)

## 项目管理

可在项目层级为团队成员和群组分配独立角色。

| **Role**           | **Ability**                                                                                |
| ------------------ | ------------------------------------------------------------------------------------------ |
| Administrator      | Full Read/Write over all Environments, Feature Flag, Remote Config, Segment and Tag values |
| View Project       | Can view the Project within their account                                                  |
| Create Environment | Can create new Environments within the Project                                             |
| Create Feature     | Can create a new Feature / Remote Config                                                   |
| Delete Feature     | Can remove an existing Feature / Remote Config entirely from the Project                   |

![Image](/img/project-permissions.png)

## 环境管理

可在环境层级为团队成员和群组分配独立角色。

| **Role**         | **Ability**                                               |
| ---------------- | --------------------------------------------------------- |
| Administrator    | Can modify Feature Flag, Remote Config and Segment values |
| View Environment | Can see the Environment within their account              |

![Image](/img/environment-permissions.png)