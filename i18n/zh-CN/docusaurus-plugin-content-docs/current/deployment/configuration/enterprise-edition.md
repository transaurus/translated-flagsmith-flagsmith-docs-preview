---
sidebar_position: 4
---

# 企业版

Flagsmith 还提供"企业版"，相较于开源版本具备更多功能特性：

- [基于角色的访问控制](advanced-use/permissions.md)
- [支持SAML、LDAP、ADFS和Okta认证](deployment/configuration/authentication.md)，并可锁定单一认证提供商
- 额外数据库引擎支持：Oracle、SQL Server和MySQL
- 如下文所述的额外部署与编排选项

## 部署方案

当前支持的基础设施平台包括：

- Kubernetes
- Redhat OpenShift
- 亚马逊云服务(AWS) - 通过[Amazon ECS](https://aws.amazon.com/ecs/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)部署
- 谷歌云平台(GCP) - 通过[AppEngine](https://cloud.google.com/appengine)部署
- 微软Azure - 通过[容器实例](https://azure.microsoft.com/en-gb/services/container-instances/)部署

如需其他部署方案，请联系我们。

## 编排工具

当前支持的编排方案包括：

- AWS部署专用的[Pulumi](https://www.pulumi.com/)脚本
- 适用于AWS和GCP的[Terraform](https://www.terraform.io/)脚本
- Kubernetes部署专用的[Helm Charts](https://helm.sh/)
- Kubernetes Operator部署专用的[Kubernetes Operator](https://operatorhub.io/operator/flagsmith)

如需获取相关项目源码，请联系我们。

## Docker镜像仓库

Flagsmith API企业版镜像托管于私有Docker Hub仓库。访问该仓库需提供Docker Hub账户，如需访问权限请与我们联系。

我们提供两种企业版镜像，可任选其一使用：

### "SaaS"镜像

该镜像与SaaS版本同步构建，包含额外功能包：

- SAML集成
- 工作流系统（变更请求与功能标志调度）

此镜像还将前端打包至Python应用中，无需单独运行前后端服务。

### 企业版标准镜像

该镜像在SaaS镜像基础上额外包含：

- Oracle数据库支持
- MySQL数据库支持
- Appdynamics监控集成
- LDAP认证支持

## 环境变量

### 前端环境变量

| Variable                 | Example Value                | Description                                                                                                                           |
| ------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **API_URL**              | http://localhost:8888/api/v1 | The URL of the API Backend                                                                                                            |
| **FLAGSMITH_CLIENT_API** | http://localhost:8888/api/v1 | This is where the features for the front end themselves are pulled from. Create a project within your backend and refer to flag names |

环境变量：**FLAGSMITH** 示例值：4vfqhypYjcPoGGu8ByrBaj 说明：对应`FLAGSMITH_CLIENT_API`项目的`environment id`。

### 后端环境变量

| Variable                                   | Example Value                                                                                            | Description                                                                                                                                                                                                                                                                                                                                        | Default Value                                              |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **LDAP_AUTH_URL**                          | ldap://localhost:389                                                                                     | The URL of the LDAP server                                                                                                                                                                                                                                                                                                                         | None                                                       |
| **LDAP_AUTH_USE_TLS**                      | False                                                                                                    | Setting this to true will initiate TLS on connection                                                                                                                                                                                                                                                                                               | False                                                      |
| **LDAP_AUTH_SEARCH_BASE**                  | ou=people,dc=example,dc=com                                                                              | The LDAP search base for looking up users                                                                                                                                                                                                                                                                                                          | ou=people,dc=example,dc=com                                |
| **LDAP_AUTH_OBJECT_CLASS**                 | inetOrgPerson                                                                                            | The LDAP class that represents a user                                                                                                                                                                                                                                                                                                              | inetOrgPerson                                              |
| **LDAP_AUTH_USER_FIELDS**                  | username=uid,email=email                                                                                 | User model fields mapped to the LDAP attributes that represent them.                                                                                                                                                                                                                                                                               | username=uid,email=email,first_name=givenName,last_name=sn |
| **LDAP_AUTH_ACTIVE_DIRECTORY_DOMAIN**      | DOMAIN                                                                                                   | Sets the login domain for Active Directory users.                                                                                                                                                                                                                                                                                                  | None                                                       |
| **LDAP_AUTH_CONNECT_TIMEOUT**              | 60                                                                                                       | Set connection timeouts (in seconds) on the underlying `ldap3` library.                                                                                                                                                                                                                                                                            | None                                                       |
| **LDAP_AUTH_RECEIVE_TIMEOUT**              | 60                                                                                                       | Set receive timeouts (in seconds) on the underlying `ldap3` library.                                                                                                                                                                                                                                                                               | None                                                       |
| **LDAP_AUTH_FORMAT_USERNAME**              | django_python3_ldap.<br/>utils.format_username_openldap                                                  | Path to a callable used to format the username to bind to the LDAP server                                                                                                                                                                                                                                                                          | django_python3_ldap.utils.format_username_openldap         |
| **LDAP_DEFAULT_FLAGSMITH_ORGANISATION_ID** | 1                                                                                                        | All newly created users will be added to this originisation                                                                                                                                                                                                                                                                                        | None                                                       |
| **LDAP_AUTH_SYNC_USER_RELATIONS**          | custom_auth.ldap.sync_user_groups                                                                        | Path to a callable used to sync user realtions. Note: if you are setting this value to `custom_auth.ldap.sync_user_groups` please make sure `LDAP_DEFAULT_FLAGSMITH_ORGANISATION_ID` is set.                                                                                                                                                       | django_python3_ldap.utils.sync_user_relations              |
| **LDAP_AUTH_FORMAT_SEARCH_FILTERS**        | custom_auth.ldap.login_group_search_filter                                                               | Path to a callable used to add search filters to login to restrict login to a certain group                                                                                                                                                                                                                                                        | django_python3_ldap.utils.format_search_filters            |
| **LDAP_SYNCED_GROUPS**                     | CN=Readers,CN=Roles,CN=webapp01,<br/>dc=admin,dc=com:CN=Marvel,CN=Roles,<br/>CN=webapp01,dc=admin,dc=com | colon(:) seperated list of DN's of ldap group that will be copied over to flagmsith(lazily, i.e: On user login we will create the group(s) and add the current user to the group(s) if the user is a part of them). Note: please make sure to set `LDAP_AUTH_SYNC_USER_RELATIONS` to `custom_auth.ldap.sync_user_groups` inorder for this to work. | []                                                         |
| **LDAP_LOGIN_GROUP**                       | CN=Readers,CN=Roles,CN=webapp01,<br/>dc=admin,dc=com                                                     | DN of the user allowed login user group. Note: Please make sure to set `LDAP_AUTH_FORMAT_SEARCH_FILTERS` to `custom_auth.ldap.login_group_search_filter` in order for this to work.                                                                                                                                                                | None                                                       |
| **LDAP_SYNC_USER_USERNAME**                | john                                                                                                     | Username used by [sync_ldap_users_and_groups](#sync-ldap-users-groups) in order to connect to the server.                                                                                                                                                                                                                                          | None                                                       |
| **LDAP_SYNC_USER_PASSWORD**                | password                                                                                                 | Password used by [sync_ldap_users_and_groups](#sync-ldap-users-groups) in order to connect to the server.                                                                                                                                                                                                                                          | None                                                       |

### 版本标签

`flagsmith-api-ee`的版本号与开源版本保持同步，可通过以下链接查看：

[https://github.com/Flagsmith/flagsmith/tags](https://github.com/Flagsmith/flagsmith/tags)

## AppDynamics监控

本应用支持通过AppDynamics进行监控，配置步骤如下：

:::note

AppDynamics向导存在一个bug：会将`ssl = on`错误配置为`ssl = (on)`，需手动修正此参数

:::

1. 在AppDynamics仪表盘中通过"Getting Started Wizard - Python"向导设置您的应用程序。
2. 在向导中选择部署方式时需勾选"uWSGI with Emperor: Module Directive"选项。
3. 完成向导后，系统会生成类似appdynamics.template.cfg模板的配置文件（包含您的应用信息）。请复制这些配置信息并保存为文件。

### Docker运行方式

使用传统Docker运行时，可通过以下代码片段注入AppDynamics所需的配置信息：

```shell
docker run -t {image_name} -v {config_file_path}:/etc/appdynamics.cfg -e APP_DYNAMICS=on
```

需要替换以下参数值：

- **_{image_name}_**: 所用Docker镜像的标签名称
- **_{config_file_path}_**: 系统中appdynamics.cfg配置文件的绝对路径

### docker-compose运行方式

使用提供的`docker-compose.yml`文件时，请确保将`APP_DYNAMICS`环境变量设置为`on`，如下所示：

```yaml
api:
   build:
   context: .
   dockerfile: docker/Dockerfile
   env:
      APP_DYNAMICS: "on"
   volumes:
   - {config_file_path}:/etc/appdynamics.cfg
```

将**_{config_file_path}_**替换为系统中appdynamics.cfg配置文件的绝对路径。

执行以下命令将构建包含所有AppDynamics配置的Docker镜像：

```shell
docker-compose -f docker-compose.yml build
```

随后可通过docker-compose的`up`命令在本地运行该镜像：

```shell
docker-compose -f docker-compose.yml up
```

### 附加设置

如需更多AppDynamics配置选项，可查阅[此处](https://docs.appdynamics.com/display/PRO21/Python+Agent+Settings)列出的其他环境变量。

### Oracle数据库

通过正确配置环境变量，Flagsmith可兼容Oracle数据库引擎。首先需设置`DJANGO_DB_ENGINE`为`oracle`，然后按文档要求配置其他数据库参数（`DJANGO_DB_*`）。

#### 本地测试

以下章节详述了如何使用OracleDB Docker镜像在本地运行应用。如需连接其他OracleDB实例，只需按文档正确设置环境变量即可。

:::note

**前置条件**

需要在运行Flagsmith API应用的机器上安装Oracle客户端，安装指南参见[此处](https://cx-oracle.readthedocs.io/en/latest/user_guide/installation.html)。

:::

要通过Docker在本地运行Oracle数据库，需先使用Docker Hub账号注册获取镜像访问权限。访问https://hub.docker.com/_/oracle-database-enterprise-edition完成注册后，可使用我们提供的docker-compose文件简化Oracle数据库的启动流程。

```bash
docker-compose -f docker-compose.oracle.yml up db
```

数据库启动后，需执行以下步骤正确配置数据库和用户。

首先连接到数据库：

```bash
sqlplus /nolog
```

或

```bash
docker-compose exec db bash -c "source /home/oracle/.bashrc; sqlplus /nolog"
```

连接成功后执行以下SQL命令（如需不同密码/用户组合请自行修改）：

```sql
conn sys as sysdba;
# password is blank when asked
alter session set "_ORACLE_SCRIPT"=true;
create user oracle_user identified by oracle_password;
grant dba to oracle_user;
GRANT EXECUTE ON SYS.DBMS_LOB TO oracle_user;
GRANT EXECUTE ON SYS.DBMS_RANDOM TO oracle_user;
```

### SAML认证

应用程序可使用SAML2作为认证后端运行。启用SAML2通常无需额外配置，但应用启动后，需为需要SAML2认证的组织创建相关配置实体。目前只能通过Django管理控制台的"组织"页面下的"Saml配置"模块完成。访问django管理控制台的方法详见[此处](/deployment/configuration/django-admin)。

SAML配置需要以下参数：

| Parameter name    | Description                                                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Organisation      | The flagsmith organisation entity to allow SAML login for                                                                     |
| Organisation name | A unique string for the organisation. You'll enter this when prompted by the SSO login flow and forms part of your entity ID. |
| Frontend URL      | This is the URL to redirect to on a successful SAML authentication, e.g. app.flagsmith.com.                                   |
| Idp metadata      | This is the metadata from your Identity Provider.                                                                             |

:::note

本地运行时需额外安装[xmlsec1](https://command-not-found.com/xmlsec1)。

:::

#### SAML环境变量

| Variable           | Example Value | Description                                                                            | Default Value |
| ------------------ | ------------- | -------------------------------------------------------------------------------------- | ------------- |
| **SAML_FORCE_SSL** | True          | Ignores X-Forwarded-Proto HTTP headers and forces SAML redirects to occur over `https` | False         |

### LDAP认证

通过[环境变量](#backend-environment-variables)可配置LDAP认证后端。启用后，系统会通过LDAP服务器验证用户凭据，认证成功后从LDAP服务器获取用户详情并在Django数据库中创建相应用户。

:::note

#### Microsoft Active Directory支持

默认配置支持OpenLDAP登录。连接Microsoft Active Directory需修改以下环境变量。

:::

简单用户名格式（如"username"）：

```txt
LDAP_AUTH_FORMAT_USERNAME="django_python3_ldap.utils.format_username_active_directory"
```

下级登录名格式（如"DOMAIN\username"）：

```txt
LDAP_AUTH_FORMAT_USERNAME="django_python3_ldap.utils.format_username_active_directory"
LDAP_AUTH_ACTIVE_DIRECTORY_DOMAIN="DOMAIN"
```

用户主体名称格式（如"user@domain.com"）：

```txt
LDAP_AUTH_FORMAT_USERNAME="django_python3_ldap.utils.format_username_active_directory_principal"
LDAP_AUTH_ACTIVE_DIRECTORY_DOMAIN="domain.com"
```

根据Active Directory服务器配置，以下附加设置可能比django-python3-ldap的默认配置更匹配：

```txt
LDAP_AUTH_USER_FIELDS=username=sAMAccountName,email=mail,first_name=givenName,last_name=sn
LDAP_AUTH_OBJECT_CLASS="user"
```

#### 同步LDAP用户组

运行以下命令可实现Flagsmith用户组与LDAP（目录）用户组的同步：

```bash
python manage.py sync_ldap_users_and_groups
```

该命令将执行以下操作：

- 同步删除目录中已移除的Flagsmith用户
- 同步删除目录中已移除的Flagsmith用户组
- 移除目录中不再属于该组的用户
- 为目录中新归属组的用户添加权限

:::note[运行前请确保设置以下环境变量：]

- LDAP_SYNC_USER_USERNAME
- LDAP_SYNC_USER_PASSWORD
- LDAP_SYNCED_GROUPS
- LDAP_AUTH_SYNC_USER_RELATIONS
- LDAP_DEFAULT_FLAGSMITH_ORGANISATION_ID

:::

## 负载测试

### JMeter

我们在Github公有仓库提供了[JMeter](https://jmeter.apache.org/)测试脚本：

https://github.com/Flagsmith/flagsmith/tree/main/api/jmeter-tests

### wrk

推荐使用[wrk](https://github.com/wg/wrk)测试核心SDK端点的负载性能。示例脚本如下（使用时请替换URL和环境密钥）：

```bash
# Simple get flags endpoint
wrk -t6 -c200 -d20s -H 'X-Environment-Key: iyiS5EDNDxMDuiFpHoiwzG' http://127.0.0.1:8000/api/v1/flags/

# Get flags for an identity
wrk -t6 -c200 -d20s -H 'X-Environment-Key: iyiS5EDNDxMDuiFpHoiwzG' "http://127.0.0.1:8000/api/v1/identities/?identifier=mrflags@flagsmith.com"
```

## 常见安装问题及解决方案

### 前端构建报错npm ERR! errno 137

此错误由内存不足引起，请为容器分配更多内存后重试。