---
title: Managing Identities
description: Manage user traits and properties independently of your application.
---

功能标志非常实用，但它们可能是一个较为粗放的工具，仅允许您在整个用户群体中启用或禁用标志。为了更精准地定位用户，并能够执行[分阶段功能发布](/guides-and-examples/staged-feature-rollouts.md)或[A/B及多变量测试](/advanced-use/ab-testing.md)，您需要_识别您的用户_。

身份（Identities）在Flagsmith中首次通过客户端SDK识别时会自动创建。通常，您会在用户登录您的应用/网站时调用一个带有唯一字符串/ID的方法来识别用户。随后，SDK会向Flagsmith API发送包含相关身份信息的API请求。

:::tip

Flagsmith API的SDK部分设计为公开是经过深思熟虑的。环境密钥（Environment Key）设计为公开且易于在构建客户端集成时查看/找到。在识别用户时，使用一个不易猜测的身份值非常重要。例如，如果您使用递增的整数来识别用户，那么通过枚举这个整数来请求身份将变得轻而易举。这将有效地公开与用户相关的任何特征数据。

我们强烈建议使用不可猜测、不可识别的身份密钥，例如[GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)，以防止无意中泄露身份特征数据。

:::

## 身份覆盖

一旦您唯一识别了一个用户，您就可以为该用户从环境默认值中覆盖功能。例如，您已将某个功能推送到生产环境，但相关的功能标志仍对所有用户隐藏该功能。您现在可以为自己的用户覆盖该标志，并测试该功能。当您对一切满意后，可以通过启用标志本身将该功能推广给所有用户。

身份在您的项目中的每个环境中都是特定且独立的。例如，joe@yourwebsite.com在开发环境中的身份将与生产环境中的身份不同，并且可以为每个环境启用不同的功能。

## 身份功能标志

默认情况下，身份会接收其环境的默认标志。身份的主要用例是能够基于每个身份覆盖标志和配置。您可以通过导航到“用户”页面，找到用户并修改其标志来实现这一点。

## 身份特征

您还可以使用Flagsmith存储与身份关联的“特征”（Traits）。特征只是与特定环境中单个身份关联的键/值对。特征有两个主要用途，如下所述，但主要用例是驱动[分段](managing-segments.md)。

:::important

每个特征值的最大大小为**_2000字节_**。您不能在单个特征中存储更多数据，如果尝试这样做，API将返回500错误。

:::

### 使用特征驱动分段

假设您正在开发一个移动应用，并且希望基于身份使用的应用程序版本控制某个功能。在集成Flagsmith SDK时，您可以将应用程序版本号作为特征键/值对传递给Flagsmith平台：

```java
FeatureUser user = new FeatureUser();
user.setIdentifier("user_512356");

FlagsAndTraits flagsAndTraits = flagsmithClient.identifyUserWithTraits(FeatureUser user, Arrays.asList(
    trait(null, "app_version", Application.getVersion());
```

这里我们将特征键`app_version`设置为`Application.getVersion()`的值。您现在可以创建一个基于应用程序版本的[分段](managing-segments.md)，并根据应用程序版本管理功能。

特征是完全自由的。您可以在平台中存储任意数量的特征，包含任何您认为相关的信息，然后使用分段基于这些特征值控制功能。

### 使用特征作为数据存储

特征还可以用于存储关于用户的额外数据，这些数据在您的应用程序中存储可能会很繁琐。特征的一些可能用途包括：

- 存储用户是否已接受新版条款与条件。
- 存储应用内最后浏览的页面位置，以便跨设备恢复用户进度。

通常对于用户相关低价值信息，存储在Flagsmith中比存入核心应用更简便。

特征值原生支持三种数据类型：数值型、字符串型或布尔型。

![Image](/img/identity-details.png)

## 驱动用户分组的特征

特征既可在应用程序中使用，也可用于构建[用户分组](/basic-features/managing-segments.md)。

## 特征值数据类型

特征值支持四种存储类型：

- 布尔型
- 字符串型（最大长度2000字节）
- 整型（32位有符号）
- 浮点型（典型取值范围约1E-307至1E+308，精度至少15位）

如需存储64位整数或超高精度浮点数，建议转为字符串存储，并在SDK内进行类型转换。