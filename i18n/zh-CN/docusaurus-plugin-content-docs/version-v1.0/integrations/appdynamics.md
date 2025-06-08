---
title: AppDynamics Integration
sidebar_label: AppDynamics
hide_title: true
---

![Amplitude](/img/integrations/appdynamics/appdynamics-logo.svg)

您可以将Flagsmith与AppDynamics集成。此集成适用于自托管场景，可帮助您更详细地分析Flagsmith的性能表现。

:::note

AppDynamics是一项仅限企业版的集成功能。

:::

## 设置指南

本应用支持使用AppDynamics进行监控。请按以下步骤配置您的环境：

1. 在AppDynamics控制台使用"Getting Started Wizard - Python"向导创建应用
2. 在部署方式选择环节，请选择"uWSGI with Emperor: Module Directive"模式
3. 完成向导后，您将获得名为类似`appdynamics.template.cfg`的配置文件（包含您的应用信息）。请复制这些配置信息，并在代码库根目录创建名为`appdynamics.cfg`的文件。_注意_：AppDynamics向导存在一个bug，会将`ssl = (on)`错误配置，需手动改为`ssl = on`

## 通过docker-compose运行

使用提供的`docker-compose.yml`文件运行时，请确保`APP_DYNAMICS`参数设置为`on`，如下所示：

```yaml
api:
 build:
 context: .
 dockerfile: docker/Dockerfile
 args:
  APP_DYNAMICS: 'on'
```

执行以下命令将构建包含所有AppDynamics配置的Docker镜像

```bash
docker-compose -f docker-compose.yml build
```

随后可通过docker-compose的`up`命令在本地运行该镜像，如下所示

```bash
docker-compose -f docker-compose.yml up
```

## 其他配置选项

如需更多AppDynamics设置选项，可参考[此处](https://docs.appdynamics.com/display/PRO21/Python+Agent+Settings)列出的环境变量配置说明。