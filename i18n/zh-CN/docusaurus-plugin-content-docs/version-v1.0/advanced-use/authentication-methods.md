# 认证方式

:::tip

Flagsmith中的组织可以按需锁定为单一认证方式，这意味着账户只能通过指定方式创建或登录。

此配置可由超级管理员在组织层级设置。如需协助，请联系我们。

:::

除邮箱/密码及通过Google或Github的OAuth2认证外，我们还提供以下认证方式。

## SAML

Flagsmith平台可为特定组织配置SAML认证。如需为您的组织设置SAML登录，请直接联系我们协助完成配置。

请注意，通过SAML认证的用户只能归属于一个组织，即与该SAML配置绑定的组织。

要设置SAML认证，我们将为您提供SAML组织的唯一名称，您需在"单点登录"对话框中输入该名称。同时我们会提供我们的服务提供商元数据，并需要您返回身份提供商元数据。

### 信息映射

为唯一识别用户，我们会尝试从`subject-id`或`uid`声明中获取唯一标识符，或使用`NameID`属性的内容。

我们还将以下Flagsmith用户属性映射到SAML响应中的对应声明。

| Flagsmith Attribute | IdP claims                                             |
| ------------------- | ------------------------------------------------------ |
| `email`             | `mail`, `email` or `emailAddress`                      |
| `first_name`        | `gn`, `givenName` or (the first part of) `displayName` |
| `last_name`         | `sn`, `surname` or (the second part of) `displayName`  |

以下是Google的SAML应用创建流程中的配置示例。

<div style={{textAlign: 'center'}}><img width="75%" src="/img/saml-mapping-configuration.png"/></div>

## LDAP

LDAP认证功能包含在我们的[企业版](../enterprise-edition.md)中。如有需求请联系我们。我们还支持将LDAP群组同步至[Flagsmith RBAC群组](permissions.md#groups)。

## AD FS

Active Directory联合服务认证功能包含在我们的[企业版](../enterprise-edition.md)中。

## Okta

Okta集成功能包含在我们的[企业版](../enterprise-edition.md)中。