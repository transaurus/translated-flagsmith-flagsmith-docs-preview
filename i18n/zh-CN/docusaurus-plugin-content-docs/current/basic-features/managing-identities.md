---
title: Managing Identities
description: Manage user traits and properties independently of your application.
---

功能标志虽好，但往往过于简单粗暴，仅能对整个用户群体进行全局启用或禁用。若想实现更精准的用户定向、执行[分阶段功能发布](/guides-and-examples/staged-feature-rollouts.md)或进行[A/B测试与多变量测试](/advanced-use/ab-testing.md)，您需要对用户进行身份标识。

身份标识会在客户端SDK首次识别用户时自动创建。通常当用户登录您的应用/网站时，您会调用一个包含唯一字符串/ID的标识方法。随后SDK将向Flagsmith API发送包含相关身份信息的请求。

:::tip

Flagsmith API的SDK部分设计为公开访问。环境密钥本身是公开可查的，这在客户端集成时很容易获取。因此标识用户时，务必使用不易被猜测的身份值。例如，若使用自增整数作为用户标识，攻击者只需枚举该整数即可获取所有用户身份，这将导致与用户关联的所有特征数据意外泄露。

我们强烈建议采用不可预测且无关联性的标识键（如[GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)）来标识用户，以防止身份特征数据意外泄露。

:::

## 身份覆盖规则

当用户被唯一标识后，您可以针对该用户覆盖环境默认的功能配置。例如：您已将新功能部署至生产环境，但相关功能标志仍对全体用户隐藏。此时您可以为自己账号单独启用该功能进行测试。验证无误后，再通过启用全局标志向所有用户开放该功能。

身份标识在项目内的每个环境中都是独立存在的。例如joe@yourwebsite.com在开发环境和生产环境中会被视为不同身份，可以为每个环境分别配置不同的功能开关。

## 身份级功能标志

默认情况下，身份标识会继承所属环境的默认标志。其主要用途在于支持按身份覆盖标志配置。您可以通过"用户管理"页面定位目标用户，并修改其专属标志配置。

## 身份特征

您还可以使用Flagsmith存储身份相关的"特征"数据。特征即与特定环境下单个身份关联的键值对，主要有两大用途（详见下文），但核心应用场景是驱动[用户分群](managing-segments.md)。

:::important

每个特征值的最大存储容量为**_2000字节_**。若尝试存储超过此限制的数据，API将返回500错误。

:::

### 使用特征驱动分群

假设您正在开发移动应用，需要根据用户使用的应用版本控制功能。集成Flagsmith SDK时，您可以将应用版本号作为特征键值对传递给平台：

```java
FeatureUser user = new FeatureUser();
user.setIdentifier("user_512356");

FlagsAndTraits flagsAndTraits = flagsmithClient.identifyUserWithTraits(FeatureUser user, Arrays.asList(
    trait(null, "app_version", Application.getVersion());
```

这里我们设置特征键`app_version`的值为`Application.getVersion()`。随后您可创建基于应用版本的[分群](managing-segments.md)，实现按版本号管理功能。

特征数据完全自由格式。您可以在平台中存储任意数量的特征及其关联信息，并通过分群机制基于这些特征值控制功能。

### 将特征作为数据存储

特征还可用于存储那些在应用主数据库中维护成本较高的用户附加数据。典型应用场景包括：

- 存储用户是否已接受新版服务条款
- 存储应用最后浏览页面，实现跨设备续览功能

通常对于用户非核心的低价值信息，存储在Flagsmith中比存入主应用更为简便

特征值原生支持三种数据类型：数值型、字符串型或布尔型

![Image](/img/identity-details.png)

## 驱动用户分群的特征

特征值既可在应用内使用，也可用于驱动[用户分群](/basic-features/managing-segments.md)功能

## 特征值数据类型

特征值支持以下四种数据类型存储：

- 布尔型
- 字符串型（最大长度2000字节）
- 整型（32位有符号）
- 浮点型（典型取值范围1E-307至1E+308，精度至少15位）

如需存储64位整数或高精度浮点数，建议转为字符串存储，通过SDK进行类型转换

## 批量导入用户与特征

Flagsmith采用延迟创建用户机制。若需在非用户会话场景导入数据，可通过`POST`请求调用`identities`接口实现：

```bash
curl -X "POST" "https://edge.api.flagsmith.com/api/v1/identities/?identifier=<identity_id>" \
     -H 'X-Environment-Key: <Your Environment Key>' \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "traits":   "traits": [
    {
      "trait_key": "this_key",
      "trait_value": "this_value"
    },
    {
      "trait_key": "this_key2",
      "trait_value": "this_value2"
    }
  ],
  "identifier": "<identity_id>"
}'
```