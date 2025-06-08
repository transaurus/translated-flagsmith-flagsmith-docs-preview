---
title: Authentication
sidebar_label: Authentication
sidebar_position: 78
---

:::tip

Flagsmith中的组织可按需锁定为单一认证方式，这意味着账户只能通过指定方式创建或登录。

此配置需由超级管理员在组织层级完成。如需协助，请随时联系我们。

:::

除邮箱/密码及通过Google或Github的OAuth2认证外，我们还提供以下认证方式。

## SAML - SaaS版

可为指定组织配置Flagsmith平台使用SAML认证。如需为您的组织设置SAML登录，请直接联系我们协助完成配置。

配置流程大致如下：

1. 您联系我们为组织启用SAML功能
2. 我们生成XML文件并提供SAML组织名称
3. 您将该XML提供给SAML身份提供商(IDP)，并接收返回的XML（可能需要配置[属性映射](#attribute-mapping-information)）
4. 您发回该XML，我们将其添加至Flagsmith系统
5. 用户访问https://app.flagsmith.com/login，点击"单点登录"并输入第2步获取的SAML组织名称即可登录

请注意：通过SAML认证的用户仅能属于与其SAML配置绑定的单一组织。

设置SAML认证时，我们将为您提供唯一的SAML组织名称（需在"单点登录"对话框中输入），同时提供我们的服务提供商元数据，并需要您返回IDP元数据。

## SAML - 企业本地部署版

:::tip

SAML认证仅在企业版计划中提供。如有需求，请[联系我们](https://flagsmith.com/contact-us/)！

:::

要为Flagsmith平台上的组织启用SAML登录，需访问管理员界面。访问方法详见[此文档](https://docs.flagsmith.com/deployment/configuration/django-admin)。

登录Django管理员界面后，点击左侧菜单中的'Saml Configurations'选项。在现有SAML配置列表页面，点击右上角的'ADD SAML CONFIGURATION'按钮创建新配置。

您将看到类似下图的界面：

![SAML认证设置](/img/saml-auth-setup.png)

在**Organisation**下拉菜单中选择需要配置SAML认证的Flagsmith平台组织。

在**Organisation name**字段输入不含空格的唯一组织标识名称。用户将通过该名称进行SAML认证以确定重定向目标。

在**Frontend url**字段填入您前端应用的运行URL，用户SAML认证成功后将被重定向至此地址。

暂时保持**Idp metadata xml**字段为空。

如需从IDP发起认证请求（IDP初始化登录），请勾选**Allow IdP-initiated (unsolicited) login**选项。

完成上述字段填写后，点击**Save**按钮创建SAML配置。

现在我们需要获取Flagsmith的服务提供者元数据，以便在您的身份提供商(IDP)上配置集成。请打开新浏览器标签页，访问`https://<您的Flagsmith API地址>/api/v1/auth/saml/<组织名称>/metadata/`，其中`<组织名称>`是您之前提供的名称。将网页显示的内容复制粘贴到新建的文本文件中，保存为`flagsmith-sp-metadata.xml`或类似名称。

注意：不要使用浏览器的"另存为"功能，这可能导致元数据不正确，致使集成失效。

现在您可以使用此XML元数据在IDP上创建集成。在IDP端配置完成后，将IDP的元数据填入之前留空的**Idp metadata xml**字段。

## 属性映射信息

为唯一识别用户，我们会尝试从`subject-id`或`uid`声明中获取唯一标识符，或使用`NameID`属性的内容。

我们还将以下Flagsmith用户属性映射到SAML响应中的对应声明：

| Flagsmith Attribute | IdP claims                                             |
| ------------------- | ------------------------------------------------------ |
| `email`             | `mail`, `email` or `emailAddress`                      |
| `first_name`        | `gn`, `givenName` or (the first part of) `displayName` |
| `last_name`         | `sn`, `surname` or (the second part of) `displayName`  |

以下是Google SAML应用创建流程的配置示例：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/saml-mapping-configuration.png"/></div>

## SAML群组同步

您可以配置Flagsmith，根据SAML响应在用户登录时将其添加或移出群组。

例如：假设您的系统中存在名为`team`的群组，当用户登录Flagsmith时，若该群组存在于相关SAML属性中（但用户尚未加入），则将其添加至对应的Flagsmith群组；若群组不存在于SAML属性中（但用户已加入），则将其移出。

### 配置方法

需在Flagsmith中创建对应群组，并使用SAML群组名称作为外部ID。例如：若您的SAML属性如下方XML片段所示（且需同步`team`群组）：

```xml
<saml2:Attribute Name="groups">
    <saml2:AttributeValue
        xmlns:xs="http://www.w3.org/2001/XMLSchema"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="xs:anyType">team
    </saml2:AttributeValue>
    <saml2:AttributeValue
        xmlns:xs="http://www.w3.org/2001/XMLSchema"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="xs:anyType">for_saml_group_test
    </saml2:AttributeValue>
</saml2:Attribute>
```

则需在Flagsmith中创建如下对应群组：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/saml-group-sync-external-id.png"/></div>

## OAuth认证

### Google

配置Google OAuth的步骤：

- [设置OAuth 2.0](https://support.google.com/cloud/answer/6158849?hl=en)
- 按[此处](/deployment/overview#oauth_google)说明创建Flagsmith功能开关

### Github

配置前提：确保已设置[Flagsmith自托管实例](/deployment/overview#running-flagsmith-on-flagsmith)

需配置以下环境变量：

- `GITHUB_CLIENT_ID`
- `GITHUB_CLIENT_SECRET`

配置Github OAuth的步骤：

- [创建OAuth Github应用](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app)
- 回调URL填写：`https://<您的Flagsmith域名>/oauth/github`
- 按[此处](/deployment/overview#oauth_github)说明创建Flagsmith功能开关

完成后即可看到GitHub单点登录选项。

<div style={{textAlign: 'center'}}><img width="75%" src="/img/Flagsmith_GitHub_SignUp.png"/></div>

## LDAP认证

LDAP认证功能仅限[企业版](/deployment/configuration/enterprise-edition.md)。如有需求请联系我们。我们还支持将LDAP群组同步至[Flagsmith RBAC群组](/advanced-use/permissions.md#groups)。

## AD FS

Active Directory 联合身份验证服务 (AD FS) 可在我们的[企业版](/deployment/configuration/enterprise-edition.md)中使用。

## Okta

Okta 集成可在我们的[企业版](/deployment/configuration/enterprise-edition.md)中使用。