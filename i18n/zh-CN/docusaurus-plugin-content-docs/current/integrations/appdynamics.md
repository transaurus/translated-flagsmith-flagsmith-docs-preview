---
title: AppDynamics Integration
sidebar_label: AppDynamics
hide_title: true
---

![Amplitude](/img/integrations/appdynamics/appdynamics-logo.svg)

您可以将Flagsmith与AppDynamics集成。该集成适用于自托管场景，可帮助您更详细地分析Flagsmith的性能表现。

:::note

AppDynamics属于企业专属集成方案。

:::

## 配置指南

本应用支持使用AppDynamics进行监控。请按以下步骤配置环境：

1. 在AppDynamics控制台通过"Getting Started Wizard - Python"向导创建应用
2. 选择部署方式时需勾选"uWSGI with Emperor: Module Directive"
3. 向导完成后会生成名为`appdynamics.template.cfg`的配置文件（含您的应用信息）。请复制配置内容并在代码库根目录创建`appdynamics.cfg`文件。_注意_：AppDynamics向导存在将`ssl = (on)`错误配置的bug，需手动改为`ssl = on`

## Docker-compose运行方式

使用提供的`docker-compose.yml`文件时，请确保`APP_DYNAMICS`参数设置为`on`，如下所示：

```yaml
api:
 build:
 context: .
 dockerfile: docker/Dockerfile
 args:
  APP_DYNAMICS: 'on'
```

执行以下命令将构建包含AppDynamics配置的Docker镜像：

```bash
docker-compose -f docker-compose.yml build
```

随后可通过docker-compose的`up`命令本地运行该镜像：

```bash
docker-compose -f docker-compose.yml up
```

## 高级配置

如需其他AppDynamics配置选项，可参考[官方文档](https://docs.appdynamics.com/display/PRO21/Python+Agent+Settings)设置环境变量。