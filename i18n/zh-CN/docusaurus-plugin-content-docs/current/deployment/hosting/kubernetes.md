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

上述方法是一种快速简单的访问Flagsmith的方式，但在许多情况下需要配置入口以与入口控制器配合使用。

#### 端口转发

在终端中运行：

```bash
kubectl -n [flagsmith-namespace] port-forward svc/[flagsmith-release-name]-frontend 8080:8080
```

然后在浏览器中访问`http://localhost:8080`。

#### 在具有入口控制器的集群中，使用前端代理

在此配置中，API请求由前端代理转发。这种配置更简单，但会引入一些延迟。

为flagsmith设置以下值，根据需要调整以适应您的入口控制器及相关的DNS变更。

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

然后，在完成集群外DNS或CDN变更后，在浏览器中访问`https://flagsmith.[MYDOMAIN]`。

#### 在具有入口控制器的集群中，为前端和API分别配置入口

为flagsmith设置以下值，根据需要调整以适应您的入口控制器及相关的DNS变更。同时，设置`FLAGSMITH_API_URL`环境变量，确保该URL可从访问前端的浏览器中访问。

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
  FLAGSMITH_API_URL: 'https://flagsmith.[MYDOMAIN]/api/v1/'
```

然后，在完成集群外DNS或CDN变更后，在浏览器中访问`https://flagsmith.[MYDOMAIN]`。

#### Minikube入口

（更多详情请参阅https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/。）

如果使用minikube，通过`minikube addons enable ingress`启用入口。

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

并应用。这将创建两个入口资源。

运行`minikube ip`。将该IP和`flagsmith.local`添加到您的`/etc/hosts`文件中，例如：

```txt
192.168.99.99 flagsmith.local
```

然后在浏览器中访问`http://flagsmith.local`。

### 提供的数据库配置

默认情况下，该chart会在集群内创建自己的PostgreSQL服务器，引用[https://github.com/helm/charts/tree/master/stable/postgresql](https://github.com/helm/charts/tree/master/stable/postgresql)作为服务。

:::caution

我们建议在生产环境中运行外部托管的数据库，可以通过在集群中部署自己的Postgres实例，或使用类似[AWS RDS](https://aws.amazon.com/rds/postgresql/)的服务。

:::

您可以通过修改值来为Postgres数据库提供配置选项，例如以下更改了max_connections：

```yaml
postgresql:
 enabled: true

 postgresqlConfiguration:
  max_connections: '200' # override the default max_connections of 100
```

### 外部数据库配置

要将Flagsmith API连接到外部PostgreSQL服务器，请在`databaseExternal`下设置值，例如：

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

:::caution

在 Kubernetes 中运行时，必须在 Helm Chart 中定义 [`secretKey`](https://docs.djangoproject.com/zh-hans/4.1/ref/settings/#std-setting-SECRET_KEY) 值。请使用密码管理器生成随机哈希值并设置该值，以确保所有 API 节点使用相同的 `DJANGO_SECRET_KEY`。

如果使用我们的 Helm Chart 且未提供 `secretKey`，系统会自动生成一个密钥并在运行的 Pod 之间共享，但重新部署时该密钥会变更，这通常不是期望的行为。

:::

该 Chart 已处理大部分必需的环境变量，完整配置选项请参阅 [API 说明文档](https://docs.flagsmith.com/deployment/hosting/locally-api#environment-variables)。可通过 `api.extraEnv` 设置这些变量，例如：

```yaml
api:
 extraEnv:
  LOG_LEVEL: DEBUG
```

### 资源分配

默认未设置资源限制或请求。

待办：推荐默认配置

### 副本数量

默认情况下，前端和 API 各使用 1 个副本。

待办：推荐默认配置。

待办：考虑自动伸缩方案。

待办：创建 Pod 中断预算

### 部署策略

对于每个部署，可设置 `deploymentStrategy`。默认未设置此值（采用 Kubernetes 默认行为），也可设置为对象进行调整。详见 https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#strategy。

例如：

```yaml
api:
 deploymentStrategy:
  type: RollingUpdate
  rollingUpdate:
   maxUnavailable: 1
   maxSurge: '50%'
```

### PgBouncer

默认情况下 Flagsmith 直接连接数据库（集群内或外部数据库）。可通过设置 `pgbouncer.enabled: true` 启用 PgBouncer，使 Flagsmith 连接至 PgBouncer，再由 PgBouncer 连接数据库。

### 一体化 Docker 镜像

https://hub.docker.com/r/flagsmith/flagsmith/ 提供的 Docker 镜像同时包含 API 和前端。使用时需设置以下值：

```yaml
api:
 image:
  repository: flagsmith/flagsmith # or some other repository hosting the combined image
  tag: 2.14 # or some other tag that exists in that repository
 separateApiAndFrontend: false
```

这将关闭前端的 Kubernetes 部署，但保留 Ingress 和 Service 配置，所有请求由 API 部署处理。

## 配置

下表列出该 Chart 的可配置参数及其默认值。

| Parameter                                          | Description                                                    | Default                        |
| -------------------------------------------------- | -------------------------------------------------------------- | ------------------------------ |
| `api.image.repository`                             | docker image repository for flagsmith api                      | `flagsmith/flagsmith-api`      |
| `api.image.tag`                                    | docker image tag for flagsmith api                             | appVersion                     |
| `api.image.imagePullPolicy`                        |                                                                | `IfNotPresent`                 |
| `api.image.imagePullSecrets`                       |                                                                | `[]`                           |
| `api.separateApiAndFrontend`                       | Set to false if using flagsmith/flagsmith image for the api    | `true`                         |
| `api.replicacount`                                 | number of replicas for the flagsmith api, `null` to unset      | 1                              |
| `api.deploymentStrategy`                           | See "Deployment strategy" above                                |                                |
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
| `frontend.replicacount`                            | number of replicas for the flagsmith frontend, `null` to unset | 1                              |
| `frontend.deploymentStrategy`                      | See "Deployment strategy" above                                |                                |
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
| `pgbouncer.replicaCount`                           | number of replicas for pgbouncer, `null` to unset              | 1                              |
| `pgbouncer.deploymentStrategy`                     | See "Deployment strategy" above                                |                                |
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
| `api.statsd.enabled`                               | Enable statsd metric reporting from gunicorn.                  | `false`                        |
| `api.statsd.host`                                  | Host URL to receive statsd metrics                             | `null`                         |
| `api.statsd.hostFromNodeIp`                        | Set as true to use the node IP as the statsd host instead      | `false`                        |
| `api.statsd.port`                                  | Host port to receive statsd metrics                            | `8125`                         |
| `api.statsd.prefix`                                | Prefix to add to metric ids                                    | `flagsmith.api`                |

---

### InfluxDB

默认情况下 Flagsmith 使用 Postgres 存储时间序列数据。也可选用 Influx 来追踪：

- SDK API 流量
- SDK 功能标志评估

[配置 InfluxDB 需执行额外步骤](/deployment/overview#influxdb)。

## 开发与贡献

### 环境要求

helm 版本 > 3.0.2

### 本地运行

在 OSX 上可通过 `minikube` 按以下方式测试和运行应用：

```bash
# Install Docker for Desktop and then:

brew install minikube
minikube start --memory 8192 --cpus 4
helm install flagsmith --debug ./flagsmith
minikube dashboard
```

### 测试安装 Chart

不构建包直接安装 Chart：

```bash
helm install flagsmith --debug ./flagsmith
```

运行模板并检查 Kubernetes 资源生成：

```bash
helm template flagsmith flagsmith --debug -f flagsmith/values.yaml
```

### 构建 Chart 包

执行以下命令构建 Chart 包：

```bash
helm package ./flagsmith
```