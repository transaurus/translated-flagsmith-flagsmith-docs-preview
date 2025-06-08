---
title: Authentication
sidebar_label: Authentication
sidebar_position: 78
---

## SAML配置

要为Flagsmith平台上的组织启用SAML认证登录，您需要访问管理界面。具体访问方法可参考[此处文档](https://docs.flagsmith.com/deployment/configuration/django-admin)。

登录Django管理界面后，需点击左侧菜单中的'Saml Configurations'选项。此处将显示现有SAML配置实体列表。如需新建配置，请点击屏幕右上角的'ADD SAML CONFIGURATION'按钮。

您将看到类似下图的界面。

![SAML认证设置](/img/saml-auth-setup.png)

在**Organisation**下拉菜单中，选择需要配置SAML认证的Flagsmith平台组织。

在**Organisation name**旁输入不含空格的唯一标识名称。用户将通过该名称进行SAML认证，平台据此确定重定向目标。

在**Frontend url**字段填入前端应用运行地址，用户完成SAML认证后将重定向至此。

暂时保持**Idp metadata xml**字段为空。

完成上述字段填写后，点击**Save**按钮创建SAML配置。

接下来需要获取Flagsmith服务提供者元数据以配置身份提供商(IDP)集成。新建浏览器标签页访问[https://api.flagsmith.com/api/v1/auth/saml/\<组织名称\>/metadata/](https://api.flagsmith.com/api/v1/auth/saml/\<组织名称\>/metadata/)，其中_\<组织名称\>_即前文填写的名称。将网页内容复制保存为`flagsmith-sp-metadata.xml`文本文件。

注意：请勿使用浏览器"另存为"功能，否则可能导致元数据错误致使集成失效。

现在可使用该XML元数据在IDP创建集成。IDP端配置完成后，将IDP元数据填入先前留空的**Idp metadata xml**字段。