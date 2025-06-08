---
sidebar_position: 3
---

# 企业版

Flagsmith 还提供"企业版"，相较于开源版本具备更多功能特性：

- [基于角色的访问控制](advanced-use/permissions.md)
- [SAML、LDAP、ADFS 和 Okta 认证](advanced-use/authentication-methods.md)，并支持锁定单一认证提供商
- 额外数据库引擎支持：Oracle、SQL Server 和 MySQL
- 如下文所述的额外部署与编排选项

## 部署选项

当前支持的底层架构平台包括：

- Kubernetes
- Redhat OpenShift
- 亚马逊云服务(AWS) - 通过 [Amazon ECS](https://aws.amazon.com/ecs/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)
- 谷歌云平台(GCP) - 通过 [AppEngine](https://cloud.google.com/appengine)
- Azure - 通过 [容器实例](https://azure.microsoft.com/en-gb/services/container-instances/)

如需其他部署方案，请联系我们。

## 编排方案

当前支持的编排工具包括：

- AWS 部署的 [Pulumi](https://www.pulumi.com/) 脚本
- AWS 和 GCP 部署的 [Terraform](https://www.terraform.io/) 脚本
- Kubernetes 部署的 [Helm Charts](https://helm.sh/)
- Kubernetes Operator 部署的 [Kubernetes Operator](https://operatorhub.io/operator/flagsmith)

如需获取相关项目源码，请联系我们。

## Docker 镜像仓库

Flagsmith API 企业版镜像托管于私有 Docker Hub 仓库。访问该仓库需提供 Docker Hub 账户凭证，如需访问权限请与我们联系。

我们提供两种企业版镜像，您可任选其一：

### "SaaS"镜像

该镜像与我们的 SaaS 构建版本同步，包含额外功能包：

- SAML 和 LDAP 认证
- 工作流（变更请求与功能标志调度）

此镜像还将前端打包至 Python 应用内，无需单独运行前后端服务。

### 企业版镜像

该镜像在 SaaS 镜像基础上额外包含：

- Oracle 支持
- MySQL 支持
- Appdynamics 集成

## 环境变量

### 前端环境变量

| Variable                 | Example Value                | Description                                                                                                                           |
| ------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **API_URL**              | http://localhost:8888/api/v1 | The URL of the API Backend                                                                                                            |
| **FLAGSMITH_CLIENT_API** | http://localhost:8888/api/v1 | This is where the features for the front end themselves are pulled from. Create a project within your backend and refer to flag names |

环境变量: **FLAGSMITH** 示例值: 4vfqhypYjcPoGGu8ByrBaj 说明: 上述 `FLAGSMITH_CLIENT_API` 项目对应的 `environment id`。

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

### 版本标签

`flagsmith-api-ee` 的版本号与开源版本保持同步，可通过以下链接查看：

[https://github.com/Flagsmith/flagsmith/tags](https://github.com/Flagsmith/flagsmith/tags)

## AppDynamics

本应用支持使用 AppDynamics 进行监控，配置步骤如下：

:::note

AppDynamics 向导存在一个缺陷：会将 `ssl = (on)` 设为默认值，需手动修改为 `ssl = on`

:::

1. 在AppDynamics仪表板中使用"入门向导 - Python"设置您的应用程序。
2. 在向导中选择部署方法时，需选择"uWSGI with Emperor: Module Directive"。
3. 完成向导后，您将获得一个类似appdynamics.template.cfg模板的配置文件（包含您的应用信息）。请复制这些信息并保存到文件中。

### Docker运行方式

使用传统Docker运行时，可通过以下代码片段注入AppDynamics所需的配置信息

```shell
docker run -t {image_name} -v {config_file_path}:/etc/appdynamics.cfg -e APP_DYNAMICS=on
```

需要替换以下参数：

- **_{image_name}_**: 您使用的Docker镜像标签名
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

执行以下命令将构建包含所有AppDynamics配置的Docker镜像

```shell
docker-compose -f docker-compose.yml build
```

随后可通过docker-compose的`up`命令本地运行该镜像，如下所示

```shell
docker-compose -f docker-compose.yml up
```

### 附加设置

如需其他AppDynamics配置选项，可查阅[此处](https://docs.appdynamics.com/display/PRO21/Python+Agent+Settings)列出的环境变量。

### Oracle数据库

通过正确配置环境变量，Flagsmith可兼容Oracle数据库引擎。首先需确保将`DJANGO_DB_ENGINE`设为`oracle`，然后按需设置其他数据库参数（`DJANGO_DB_*`）。

#### 本地测试

以下章节详述如何使用OracleDB Docker镜像本地运行应用。若需连接其他OracleDB实例，只需按文档正确设置环境变量即可。

:::note

**前置条件**

需要在运行Flagsmith API应用的机器上安装Oracle客户端，安装指南参见[此处](https://cx-oracle.readthedocs.io/en/latest/user_guide/installation.html)。

:::

要通过Docker本地运行Oracle数据库，需先使用Docker Hub账户注册获取镜像访问权限。访问https://hub.docker.com/_/oracle-database-enterprise-edition 完成注册后，即可运行Oracle数据库。我们已准备好简化此过程的docker-compose文件。

```bash
docker-compose -f docker-compose.oracle.yml up db
```

数据库运行后，需正确设置数据库和用户，具体操作如下：

首先连接至数据库：

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

应用程序可使用SAML2作为认证后端运行。启动应用程序时通常无需额外配置SAML2，但运行后需为需要SAML2认证的组织创建相关配置实体。目前只能通过Django管理控制台操作，在"Organisation"页面的"Saml Configurations"版块完成。访问django管理控制台的更多信息请参阅[此处](/deployment/configuration/django-admin)。

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

应用程序可通过[环境变量](#backend-environment-variables)配置基于LDAP的认证后端。启用后，系统会通过LDAP服务器用用户名和密码认证用户，认证成功后从LDAP服务器获取用户详情，并在Django数据库中创建相应用户。

:::note

#### Microsoft Active Directory支持

默认配置支持OpenLDAP登录。要连接Microsoft Active Directory，需修改以下环境变量。

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

根据Active Directory服务器配置，以下附加设置可能比django-python3-ldap的默认配置更匹配您的服务器：

```txt
LDAP_AUTH_USER_FIELDS=username=sAMAccountName,email=mail,first_name=givenName,last_name=sn LDAP_AUTH_OBJECT_CLASS="user"
```

## 压力测试

### JMeter

我们在Github公共仓库中提供了[JMeter](https://jmeter.apache.org/)测试脚本：

https://github.com/Flagsmith/flagsmith/tree/main/jmeter-tests

### wrk

我们也推荐使用[wrk](https://github.com/wg/wrk)测试核心SDK端点的负载性能。示例命令如下（请替换URL和环境密钥）：

```bash
# Simple get flags endpoint
wrk -t6 -c200 -d20s -H 'X-Environment-Key: iyiS5EDNDxMDuiFpHoiwzG' http://127.0.0.1:8000/api/v1/flags/

# Get flags for an identity
wrk -t6 -c200 -d20s -H 'X-Environment-Key: iyiS5EDNDxMDuiFpHoiwzG' "http://127.0.0.1:8000/api/v1/identities/?identifier=mrflags@flagsmith.com"
```

## 常见安装问题及解决方法

### 前端构建报错 npm ERR! errno 137

此为内存不足错误，请为容器分配更多内存启动。