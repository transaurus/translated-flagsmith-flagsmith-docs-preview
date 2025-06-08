---
title: Role Based Access Control
description: Team member and group permissions.
---

Flagsmith提供细粒度的权限控制，帮助大型团队跨项目和环境管理访问权限与角色分配。

权限可分配给单个团队成员或群组。

:::caution

Flagsmith的权限/基于角色的访问控制功能_不_包含在开源版本中。如需在自托管/本地化解决方案中使用这些功能，请[联系我们](https://flagsmith.com/contact-us/)。

:::

## 群组管理

群组是管理多成员权限的便捷方式。群组可包含任意数量的团队成员，您可以通过组织设置页面创建群组。

## 组织架构

团队成员可被定义为组织管理员或普通用户。组织管理员实质上是超级用户角色，拥有对该组织内所有项目、环境、功能开关、远程配置和用户分段的完全读写权限。

非管理员的普通用户需要在相关层级手动分配权限。组织层级的可用权限如下所示。

| **Role**       | **Ability**                                                  |
| -------------- | ------------------------------------------------------------ |
| Create Project | Allows the user to create projects in the given Organisation |

![图片](/img/organisation-permissions.png)

## 项目管理

可在项目层级为团队成员和群组分配特定角色。

| **Role**           | **Ability**                                                                                |
| ------------------ | ------------------------------------------------------------------------------------------ |
| Administrator      | Full Read/Write over all Environments, Feature Flag, Remote Config, Segment and Tag values |
| View Project       | Can view the Project within their account                                                  |
| Create Environment | Can create new Environments within the Project                                             |
| Create Feature     | Can create a new Feature / Remote Config                                                   |
| Delete Feature     | Can remove an existing Feature / Remote Config entirely from the Project                   |
| Manage Segments    | Can create, delete and edit Segments within the Project                                    |

![图片](/img/project-permissions.png)

## 环境管理

可在环境层级为团队成员和群组分配特定角色。

| **Role**               | **Ability**                                               |
| ---------------------- | --------------------------------------------------------- |
| Administrator          | Can modify Feature Flag, Remote Config and Segment values |
| View Environment       | Can see the Environment within their account              |
| Update Feature State   | Update the state or value for a given feature             |
| Manage Identities      | View and update Identities                                |
| Create Change Request  | Creating a new Change Request                             |
| Approve Change Request | Approving or denying existing Change Requests             |
| View Identities        | Viewing Identities                                        |

![图片](/img/environment-permissions.png)