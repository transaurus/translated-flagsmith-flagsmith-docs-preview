---
title: Importing and Exporting
sidebar_position: 110
---

:::tip

需运行Flagsmith版本`2.28.2`或更高才能导出和导入数据。

:::

Flagsmith提供实用工具支持在不同实例间迁移数据。

例如从自托管迁移至SaaS版本的流程如下：

- **步骤1.** 联系Flagsmith客服确认迁移意向
- **步骤2.** 从自托管实例生成JSON文件（详见下文）
- **步骤3.** 将JSON文件发送给Flagsmith客服
- **步骤4.** 客服会将数据导入云端服务
- **步骤5.** 重新注册用户并设置密码（客服需为导入的组织至少分配一名管理员）

## 导出范围

**包含**以下实体：

- 功能开关
- 用户分群
- 用户标识
- 集成配置

**不包含**以下实体：

- 登录管理后台的Flagsmith用户
- 审计日志
- 变更请求
- 定时开关变更

## 导出流程

需通过终端命令执行导出操作，可选择以下方式：
1. 在已运行的容器内执行
2. 启动新容器连接相同数据库

执行前需获取组织ID，可通过Django管理界面查询（访问方法参见[此文档](/deployment/configuration/django-admin.md)）。在管理界面的"Organisations"菜单中可见类似信息：

![](/img/organisations-admin.png)

括号内的数字即为所需ID（图示案例中为1）。

获取组织ID后，可选择两种导出方式：



### 选项1 - 本地文件系统

```bash
python manage.py dumporganisationtolocalfs <organisation-id> <local-file-system-path>
```

例如：

```bash
python manage.py dumporganisationtolocalfs 1 /tmp/organisation-1.json
```

需挂载Docker卷以获取导出文件，参考下方示例docker-compose文件：

```yml
version: '3'
services:
 postgres:
  image: postgres:11.12-alpine
  environment:
   POSTGRES_PASSWORD: password
   POSTGRES_DB: flagsmith
  container_name: flagsmith_postgres
  ports:
   - '5434:5432'

 flagsmith:
  build:
   dockerfile: ./Dockerfile
   context: .
  environment:
   DJANGO_ALLOWED_HOSTS: '*'
   DATABASE_URL: postgresql://postgres:password@postgres:5432/flagsmith
   ENV: prod
   ALLOW_REGISTRATION_WITHOUT_INVITE: 'True'
  ports:
   - '8000:8000'
  depends_on:
   - postgres
  links:
   - postgres

 flagsmith-fs-exporter:
  build:
   dockerfile: ./Dockerfile
   context: .
  environment:
   DATABASE_URL: postgresql://postgres:password@postgres:5432/flagsmith
  command:
   - 'dump-organisation-to-local-fs'
   - '1'
   - '/tmp/flagsmith-exporter/org-1.json'
  depends_on:
   - postgres
   - flagsmith
  links:
   - postgres
  volumes:
   - '/tmp/flagsmith-exporter:/tmp/flagsmith-exporter'
```

### 选项2 - S3存储桶

需使用以下变体命令：

```bash
python manage.py dumporganisationtos3 <organisation-id> <bucket-name> <key>
```

例如：

```bash
python manage.py dumporganisationtos3 1 my-export-bucket exports/organisation-1.json
```

需确保应用具有AWS账户访问权限。若在AWS环境运行，请确认容器角色具有指定S3桶的读写权限，或通过环境变量`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`配置具有权限的IAM用户。