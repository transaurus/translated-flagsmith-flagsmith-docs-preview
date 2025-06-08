---
title: Feature Flags - An Overview
sidebar_label: Overview
sidebar_position: 1
---

功能标志（Feature Flags）是一种开发方法论，允许您在功能尚未完成时就部署代码。这极大促进了持续集成与持续部署（CI/CD）流程。典型工作流如下：

1. 当您准备开发新功能时（例如实现应用内的分享按钮）
2. 在Flagsmith中创建名为"sharing_button"的功能标志，在开发环境启用，在生产环境禁用
3. 开发时，所有显示该按钮的UI代码都需包裹在条件语句中，仅当"sharing_button"标志为True时显示
4. 由于按钮受标志控制，您可安全地将代码提交至生产环境，实际功能仍处于隐藏状态
5. 功能完善后，可逐步为团队成员和Beta测试者启用该标志
6. 确认无误后，在生产环境全局启用"sharing_button"标志即完成功能发布

欲深入了解功能标志技术，可参阅[Flickr 200年发表的奠基性博客文章](https://code.flickr.net/2009/12/02/flipping-out/)

## Flagsmith数据模型

以下是Flagsmith数据模型的高层概览（实际比看起来更简单）：

![图片](/img/flagsmith-model.svg)

现在让我们逐项解析：

### 组织

组织（Organisations）是团队管理项目和功能的协作单元，用户可同时属于多个组织。

### 项目

项目（Projects）包含多个环境（Environments），这些环境共享同一组功能配置。单个组织可创建任意数量的项目。

### 环境

环境（Environments）用于隔离功能配置。例如开发/预发布环境可启用某个功能，而生产环境保持关闭。单个项目可配置多个环境。

### 功能

功能（Features）在项目内所有环境间共享，但每个环境可独立修改其开关状态和参数值。

### 用户标识

用户标识（Identities）代表特定用户在某个环境中的注册实例。通过客户端注册标识，可实现用户级功能管理。例如joe@yourwebsite.com在开发环境和生产环境是不同标识，可分别配置功能开关。

详见[用户标识管理](/basic-features/managing-identities)

### 特征值

您可为用户标识存储任意数量的特征值（Traits），这些键值对可记录各类数据，例如：

- 用户登录次数
- 是否接受应用条款
- 主题偏好设置
- 特定行为记录

详见[特征值管理](/basic-features/managing-identities.md#identity-traits)

### 用户分群

用户分群（Segments）允许根据设备类型、地理位置、登录次数等特征值动态定义用户群体。

与针对单个用户类似，您也可以为特定用户群体覆盖环境默认的功能配置。例如，为"高级用户"群体展示某些特定功能。

更多信息请参阅[用户群体](/basic-features/managing-segments.md)。