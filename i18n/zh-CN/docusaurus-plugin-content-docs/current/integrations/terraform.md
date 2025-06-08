---
title: Terraform Provider
sidebar_label: Terraform
hide_title: true
---

![Terraform](/img/integrations/terraform/terraform-logo.svg)

您可以将Flagsmith与Terraform集成。使用我们的
[Terraform provider](https://registry.terraform.io/providers/Flagsmith/flagsmith) 将功能标志管理纳入您的基础设施即代码工具链中。

:::tip

您可以在Hashicorp官方文档中查阅Flagsmith提供商的最新使用指南
[此处](https://registry.terraform.io/providers/Flagsmith/flagsmith/latest/docs)。

:::

:::tip

部分API操作需要引用对象的UUID/ID。您可以通过账户设置页面启用[JSON视图](../clients/rest.md#json-view)功能，这将帮助您获取这些变量。

:::

## 前提条件

### [Terraform API密钥](../advanced-use/system-administration.md#terraform-api-keys-for-organisations):

配置Flagsmith Terraform提供商需要API密钥。请前往组织设置页面，点击`创建Terraform API密钥`生成密钥。

:::info

生成Terraform API密钥需要具备组织管理员权限。

:::

## 使用Flagsmith Terraform提供商

获取Terraform提供商密钥后，您可以创建如下所示的Terraform配置文件：

```hcl
terraform {
  required_providers {
    flagsmith = {
      source = "Flagsmith/flagsmith"
      version = "0.3.0" # or whatever the latest version is
    }
  }
}

provider "flagsmith" {
  # or omit this for master_api_key to be read from environment variable
  master_api_key = "<Your Terraform API Key>"
}

# the feature that you want to manage
resource "flagsmith_feature" "new_standard_feature" {
  feature_name = "new_standard_feature"
  project_uuid = "10421b1f-5f29-4da9-abe2-30f88c07c9e8"
  description  = "This is a new standard feature"
  type         = "STANDARD"
}

```

现在只需运行`terraform apply`命令即可创建功能。

```bash
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # flagsmith_feature.new_standard_feature will be created
  + resource "flagsmith_feature" "new_standard_feature" {
      + default_enabled = (known after apply)
      + description     = "This is a new standard feature"
      + feature_name    = "new_standard_feature"
      + id              = (known after apply)
      + initial_value   = (known after apply)
      + is_archived     = (known after apply)
      + project_id      = (known after apply)
      + project_uuid    = "10421b1f-5f29-4da9-abe2-30f88c07c9e8"
      + type            = "STANDARD"
      + uuid            = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

flagsmith_feature.new_standard_feature: Creating...
flagsmith_feature.new_standard_feature: Creation complete after 2s

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

假设您需要更新功能描述：

```hcl
# the feature that you want to manage
resource "flagsmith_feature" "new_standard_feature" {
  feature_name = "new_standard_feature"
  project_uuid = "10421b1f-5f29-4da9-abe2-30f88c07c9e8"
  description  = "New description"
  type         = "STANDARD"
}
```

运行`terraform apply`命令应用变更：

```bash
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # flagsmith_feature.new_standard_feature will be updated in-place
  ~ resource "flagsmith_feature" "new_standard_feature" {
      ~ description     = "This is a new standard feature" -> "New description"
        id              = 574
        # (7 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

flagsmith_feature.new_standard_feature: Modifying...
flagsmith_feature.new_standard_feature: Modifications complete after 1s

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

要将现有的Flagsmith功能导入Terraform（并开始跟踪其状态），您可以[导入](https://registry.terraform.io/providers/Flagsmith/flagsmith/latest/docs/resources/feature#import)该资源。