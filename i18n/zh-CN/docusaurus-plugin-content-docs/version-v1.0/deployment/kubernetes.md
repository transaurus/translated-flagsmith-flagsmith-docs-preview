---
title: Kubernetes
sidebar_position: 50
---

## 概述

请查看我们在GitHub上的[Kubernetes Chart仓库](https://github.com/Flagsmith/flagsmith-charts)以及[已发布的Helm Charts](https://flagsmith.github.io/flagsmith-charts/)。

## 快速开始

```bash
helm repo add flagsmith https://flagsmith.github.io/flagsmith-charts/
helm install -n flagsmith --create-namespace flagsmith flagsmith/flagsmith
kubectl -n flagsmith port-forward svc/flagsmith-frontend 8080:8080
```

然后在浏览器中访问`http://localhost:8080`。这将使用默认选项在新命名空间`flagsmith`中完成安装。

### 入口配置

上述方式是快速访问Flagsmith的简易方案，但多数情况下需要配置入口控制器才能正常工作。

#### 端口转发

在终端运行：

```bash
kubectl -n [flagsmith-namespace] port-forward svc/[flagsmith-release-name]-frontend 8080:8080
```

随后在浏览器访问`http://localhost:8080`。

#### 在已部署入口控制器的集群中使用前端代理

此配置下，API请求由前端代理转发。配置更简单，但会引入额外延迟。

为flagsmith设置以下值，根据您的入口控制器需求调整，并确保完成相关DNS变更。

例如：

```yaml
ingress:
 frontend:
  enabled: true
  hosts:
   - host: flagsmith.[MYDOMAIN]
     paths:
      - /
```

完成集群外DNS或CDN配置后，即可在浏览器访问`https://flagsmith.[我的域名]`。

#### 在已部署入口控制器的集群中为前端和API分别配置入口

为flagsmith设置以下值，根据入口控制器需求调整，并确保完成DNS变更。同时设置`API_URL`环境变量，确保浏览器访问前端时可解析该URL。

例如：

```yaml
ingress:
 frontend:
  enabled: true
  hosts:
   - host: flagsmith.[MYDOMAIN]
     paths:
      - /
 api:
  enabled: true
  hosts:
   - host: flagsmith.[MYDOMAIN]
     paths:
      - /api/
      - /health/

frontend:
 extraEnv:
  API_URL: 'https://flagsmith.[MYDOMAIN]/api/v1/'
```

完成集群外DNS或CDN配置后，即可在浏览器访问`https://flagsmith.[我的域名]`。

#### Minikube入口配置

（详见https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/）

若使用minikube，需先启用入口插件：`minikube addons enable ingress`。

然后为flagsmith设置以下值：

```yaml
ingress:
 frontend:
  enabled: true
  hosts:
   - host: flagsmith.local
     paths:
      - /
```

应用后将创建两个入口资源。

运行`minikube ip`获取IP，将其与`flagsmith.local`添加到`/etc/hosts`文件，例如：

```txt
192.168.99.99 flagsmith.local
```

随后在浏览器访问`http://flagsmith.local`。

### 数据库配置

默认情况下，chart会在集群内创建PostgreSQL服务器。

如需连接外部PostgreSQL服务器，请设置`databaseExternal`下的值，例如：

```yaml
postgresql:
 enabled: false # turn off the chart-managed postgres

databaseExternal:
 enabled: true
 # Can specify the full URL
 url: 'postgres://myuser:mypass@myhost:5432/mydbname'
 # Or can specify each part (url takes precedence if set)
 type: postgres
 host: myhost
 port: 5432
 database: mydbname
 username: myuser
 password: mypass
 # Or can specify a pre-existing k8s secret containing the database URL
 urlFromExistingSecret:
  enabled: true
  name: my-precreated-db-config
  key: DB_URL
```

### 环境变量

chart已处理大部分必需环境变量，完整配置选项请参阅[API说明文档](https://github.com/Flagsmith/flagsmith-api/blob/main/readme.md#environment-variables)。可通过`api.extraEnv`设置，例如：

```yaml
api:
 extraEnv:
  LOG_LEVEL: DEBUG
```

### 资源分配

默认未设置资源限制或请求。

待办：推荐默认配置

### 副本数

默认使用1个前端和1个API副本。

待办：推荐默认配置。

待办：考虑自动扩展方案。

待办：创建 Pod 中断预算

### InfluxDB

默认情况下，Flagsmith 使用 InfluxDB 存储时间序列数据。当前主要用于记录：

- SDK API 流量
- SDK 功能标志评估数据

[InfluxDB 的详细配置方法请参阅文档](/deployment/overview#influxdb)。

### PgBouncer

默认配置下 Flagsmith 直接连接数据库（集群内或外部）。可通过设置 `pgbouncer.enabled: true` 启用 PgBouncer，此时 Flagsmith 将连接至 PgBouncer，再由 PgBouncer 连接数据库。

### 一体化 Docker 镜像

https://hub.docker.com/r/flagsmith/flagsmith/ 提供的 Docker 镜像同时包含 API 和前端组件。使用时需配置以下参数：

```yaml
api:
 image:
  repository: flagsmith/flagsmith # or some other repository hosting the combined image
  tag: 2.14 # or some other tag that exists in that repository
 separateApiAndFrontend: false
```

此配置会停用前端组件的 Kubernetes 部署，但保留 ingress 和 service 配置，所有请求将由 API 部署处理。

## 配置参数

下表列出了图表可配置参数及其默认值：

| Parameter                                          | Description                                                    | Default                        |
| -------------------------------------------------- | -------------------------------------------------------------- | ------------------------------ |
| `api.image.repository`                             | docker image repository for flagsmith api                      | `flagsmith/flagsmith-api`      |
| `api.image.tag`                                    | docker image tag for flagsmith api                             | appVersion                     |
| `api.image.imagePullPolicy`                        |                                                                | `IfNotPresent`                 |
| `api.image.imagePullSecrets`                       |                                                                | `[]`                           |
| `api.separateApiAndFrontend`                       | Set to false if using flagsmith/flagsmith image for the api    | `true`                         |
| `api.replicacount`                                 | number of replicas for the flagsmith api                       | 1                              |
| `api.resources`                                    | resources per pod for the flagsmith api                        | `{}`                           |
| `api.podLabels`                                    | additional labels to apply to pods for the flagsmith api       | `{}`                           |
| `api.extraEnv`                                     | extra environment variables to set for the flagsmith api       | `{}`                           |
| `api.nodeSelector`                                 |                                                                | `{}`                           |
| `api.tolerations`                                  |                                                                | `[]`                           |
| `api.affinity`                                     |                                                                | `{}`                           |
| `api.podSecurityContext`                           |                                                                | `{}`                           |
| `api.defaultPodSecurityContext.enabled`            | whether to use the default security context                    | `true`                         |
| `api.livenessProbe.failureThreshold`               |                                                                | 5                              |
| `api.livenessProbe.initialDelaySeconds`            |                                                                | 10                             |
| `api.livenessProbe.periodSeconds`                  |                                                                | 10                             |
| `api.livenessProbe.successThreshold`               |                                                                | 1                              |
| `api.livenessProbe.timeoutSeconds`                 |                                                                | 2                              |
| `api.readinessProbe.failureThreshold`              |                                                                | 10                             |
| `api.readinessProbe.initialDelaySeconds`           |                                                                | 10                             |
| `api.readinessProbe.periodSeconds`                 |                                                                | 10                             |
| `api.readinessProbe.successThreshold`              |                                                                | 1                              |
| `api.readinessProbe.timeoutSeconds`                |                                                                | 2                              |
| `api.dbWaiter.image.repository`                    |                                                                | `willwill/wait-for-it`         |
| `api.dbWaiter.image.tag`                           |                                                                | `latest`                       |
| `api.dbWaiter.image.imagePullPolicy`               |                                                                | `IfNotPresent`                 |
| `api.dbWaiter.timeoutSeconds`                      | Time before init container will retry                          | 30                             |
| `frontend.enabled`                                 | Whether the flagsmith frontend is enabled                      | `true`                         |
| `frontend.image.repository`                        | docker image repository for flagsmith frontend                 | `flagsmith/flagsmith-frontend` |
| `frontend.image.tag`                               | docker image tag for flagsmith frontend                        | appVersion                     |
| `frontend.image.imagePullPolicy`                   |                                                                | `IfNotPresent`                 |
| `frontend.image.imagePullSecrets`                  |                                                                | `[]`                           |
| `frontend.replicacount`                            | number of replicas for the flagsmith frontend                  | 1                              |
| `frontend.resources`                               | resources per pod for the flagsmith frontend                   | `{}`                           |
| `frontend.apiProxy.enabled`                        | proxy API requests to the API service within the cluster       | `true`                         |
| `frontend.extraEnv`                                | extra environment variables to set for the flagsmith frontend  | `{}`                           |
| `frontend.nodeSelector`                            |                                                                | `{}`                           |
| `frontend.tolerations`                             |                                                                | `[]`                           |
| `frontend.affinity`                                |                                                                | `{}`                           |
| `api.podSecurityContext`                           |                                                                | `{}`                           |
| `api.defaultPodSecurityContext.enabled`            | whether to use the default security context                    | `true`                         |
| `frontend.livenessProbe.failureThreshold`          |                                                                | 20                             |
| `frontend.livenessProbe.initialDelaySeconds`       |                                                                | 20                             |
| `frontend.livenessProbe.periodSeconds`             |                                                                | 10                             |
| `frontend.livenessProbe.successThreshold`          |                                                                | 1                              |
| `frontend.livenessProbe.timeoutSeconds`            |                                                                | 10                             |
| `frontend.readinessProbe.failureThreshold`         |                                                                | 20                             |
| `frontend.readinessProbe.initialDelaySeconds`      |                                                                | 20                             |
| `frontend.readinessProbe.periodSeconds`            |                                                                | 10                             |
| `frontend.readinessProbe.successThreshold`         |                                                                | 1                              |
| `frontend.readinessProbe.timeoutSeconds`           |                                                                | 10                             |
| `postgresql.enabled`                               | if `true`, creates in-cluster PostgreSQL database              | `true`                         |
| `postgresql.serviceAccount.enabled`                | creates a serviceaccount for the postgres pod                  | `true`                         |
| `nameOverride`                                     |                                                                | `flagsmith-postgres`           |
| `postgresqlDatabase`                               |                                                                | `flagsmith`                    |
| `postgresqlUsername`                               |                                                                | `postgres`                     |
| `postgresqlPassword`                               |                                                                | `flagsmith`                    |
| `databaseExternal.enabled`                         | use an external database. Specify database URL, or all parts.  | `false`                        |
| `databaseExternal.url`                             | See https://github.com/kennethreitz/dj-database-url#url-schema |                                |
| `databaseExternal.type`                            | Note: Only postgres supported by default images.               | `postgres`                     |
| `databaseExternal.port`                            |                                                                | 5432                           |
| `databaseExternal.database`                        | Name of the database within the server                         |                                |
| `databaseExternal.username`                        |                                                                |                                |
| `databaseExternal.password`                        |                                                                |                                |
| `databaseExternal.urlFromExistingSecret.enabled`   | Reference an existing secret containing the database URL       |                                |
| `databaseExternal.urlFromExistingSecret.name`      | Name of referenced secret                                      |                                |
| `databaseExternal.urlFromExistingSecret.key`       | Key within the referenced secrt to use                         |                                |
| `influxdb.enabled`                                 |                                                                | `true`                         |
| `influxdb.nameOverride`                            |                                                                | `influxdb`                     |
| `influxdb.image.repository`                        | docker image repository for influxdb                           | `quay.io/influxdb/influxdb`    |
| `influxdb.image.tag`                               | docker image tag for influxdb                                  | `v2.0.2`                       |
| `influxdb.image.imagePullPolicy`                   |                                                                | `IfNotPresent`                 |
| `influxdb.image.imagePullSecrets`                  |                                                                | `[]`                           |
| `influxdb.adminUser.organization`                  |                                                                | `influxdata`                   |
| `influxdb.adminUser.bucket`                        |                                                                | `default`                      |
| `influxdb.adminUser.user`                          |                                                                | `admin`                        |
| `influxdb.adminUser.password`                      |                                                                | randomly generated             |
| `influxdb.adminUser.token`                         |                                                                | randomly generated             |
| `influxdb.persistence.enabled`                     |                                                                | `false`                        |
| `influxdb.resources`                               | resources per pod for the influxdb                             | `{}`                           |
| `influxdb.nodeSelector`                            |                                                                | `{}`                           |
| `influxdb.tolerations`                             |                                                                | `[]`                           |
| `influxdb.affinity`                                |                                                                | `{}`                           |
| `influxdbExternal.enabled`                         | Use an InfluxDB not managed by this chart                      | `false`                        |
| `influxdbExternal.url`                             |                                                                |                                |
| `influxdbExternal.bucket`                          |                                                                |                                |
| `influxdbExternal.organization`                    |                                                                |                                |
| `influxdbExternal.token`                           |                                                                |                                |
| `influxdbExternal.tokenFromExistingSecret.enabled` | Use reference to a k8s secret not managed by this chart        | `false`                        |
| `influxdbExternal.tokenFromExistingSecret.name`    | Referenced secret name                                         |                                |
| `influxdbExternal.tokenFromExistingSecret.key`     | Key within the referenced secret to use                        |                                |
| `pgbouncer.enabled`                                |                                                                | `false`                        |
| `pgbouncer.image.repository`                       |                                                                | `bitnami/pgbouncer`            |
| `pgbouncer.image.tag`                              |                                                                | `1.16.0`                       |
| `pgbouncer.image.imagePullPolicy`                  |                                                                | `IfNotPresent`                 |
| `pgbouncer.image.imagePullSecrets`                 |                                                                | `[]`                           |
| `pgbouncer.replicaCount`                           |                                                                | 1                              |
| `pgbouncer.podAnnotations`                         |                                                                | `{}`                           |
| `pgbouncer.resources`                              |                                                                | `{}`                           |
| `pgbouncer.podLabels`                              |                                                                | `{}`                           |
| `pgbouncer.extraEnv`                               |                                                                | `{}`                           |
| `pgbouncer.nodeSelector`                           |                                                                | `{}`                           |
| `pgbouncer.tolerations`                            |                                                                | `[]`                           |
| `pgbouncer.affinity`                               |                                                                | `{}`                           |
| `pgbouncer.podSecurityContext`                     |                                                                | `{}`                           |
| `pgbouncer.securityContext`                        |                                                                | `{}`                           |
| `pgbouncer.defaultSecurityContext.enabled`         |                                                                | `true`                         |
| `pgbouncer.defaultSecurityContext`                 |                                                                | `{}`                           |
| `pgbouncer.livenessProbe.failureThreshold`         |                                                                | 5                              |
| `pgbouncer.livenessProbe.initialDelaySeconds`      |                                                                | 5                              |
| `pgbouncer.livenessProbe.periodSeconds`            |                                                                | 10                             |
| `pgbouncer.livenessProbe.successThreshold`         |                                                                | 1                              |
| `pgbouncer.livenessProbe.timeoutSeconds`           |                                                                | 2                              |
| `pgbouncer.readinessProbe.failureThreshold`        |                                                                | 10                             |
| `pgbouncer.readinessProbe.initialDelaySeconds`     |                                                                | 1                              |
| `pgbouncer.readinessProbe.periodSeconds`           |                                                                | 10                             |
| `pgbouncer.readinessProbe.successThreshold`        |                                                                | 1                              |
| `pgbouncer.readinessProbe.timeoutSeconds`          |                                                                | 2                              |
| `service.influxdb.externalPort`                    |                                                                | `8080`                         |
| `service.api.type`                                 |                                                                | `ClusterIP`                    |
| `service.api.port`                                 |                                                                | `8000`                         |
| `service.frontend.type`                            |                                                                | `ClusterIP`                    |
| `service.frontend.port`                            |                                                                | `8080`                         |
| `ingress.frontend.enabled`                         |                                                                | `false`                        |
| `ingress.frontend.ingressClassName`                |                                                                |                                |
| `ingress.frontend.annotations`                     |                                                                | `{}`                           |
| `ingress.frontend.hosts[].host`                    |                                                                | `chart-example.local`          |
| `ingress.frontend.hosts[].paths`                   |                                                                | `[]`                           |
| `ingress.frontend.tls`                             |                                                                | `[]`                           |
| `ingress.api.enabled`                              |                                                                | `false`                        |
| `ingress.api.ingressClassName`                     |                                                                |                                |
| `ingress.api.annotations`                          |                                                                | `{}`                           |
| `ingress.api.hosts[].host`                         |                                                                | `chart-example.local`          |
| `ingress.api.hosts[].paths`                        |                                                                | `[]`                           |
| `ingress.api.tls`                                  |                                                                | `[]`                           |

---

## 开发与贡献

### 环境要求

helm 版本 > 3.0.2

### 本地运行

在 OSX 系统可通过 `minikube` 按以下方式测试运行：

```bash
# Install Docker for Desktop and then:

brew install minikube
minikube start --memory 8192 --cpus 4
helm install flagsmith --debug ./flagsmith
minikube dashboard
```

### 测试安装图表

不打包直接安装图表：

```bash
helm install flagsmith --debug ./flagsmith
```

运行模板并检查生成的 Kubernetes 资源：

```bash
helm template flagsmith flagsmith --debug -f flagsmith/values.yaml
```

### 构建图表包

执行以下命令构建图表包：

```bash
helm package ./flagsmith
```