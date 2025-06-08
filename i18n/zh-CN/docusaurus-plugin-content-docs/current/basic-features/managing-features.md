---
title: Managing Features
description: Feature Flags allow you to ship code and features before they are finished.
---

Flagsmith中的功能标志_在项目级别创建和共享_，但_在环境级别可被覆盖_。它们还可以基于[单个身份](/basic-features/managing-identities.md)或[用户分群](/basic-features/managing-segments.md)进行覆盖。

Flagsmith中的功能标志由以下两部分组成：

- 布尔值 - `标志状态`

以及可选的：

- 字符串/整数/浮点数值 - `标志值`

或

- 选定的多变量字符串/整数/浮点数 - `标志值`

您可以自由选择在每个标志中使用`标志状态`、`标志值`或两者结合。您无需提供或使用`标志值`。如果仅需要布尔型标志，可以完全忽略`标志值`。

:::important

每个字符串值的最大容量为**_20,000字节_**。无法在标志值中存储超过此限制的数据，尝试操作时API将返回500错误。

:::

## 使用示例

这使您可以通过多种方式使用Flagsmith：

- 在应用程序中显示和隐藏功能。例如：使用布尔型`标志状态`控制应用程序中的新用户界面元素
- 配置应用程序中的环境变量/密钥。例如：使用字符串`标志值`设置API的数据库URL，或在前端配置Google Analytics API密钥
- 远程配置应用程序中使用的字符串值。例如：根据环境为应用横幅定义不同的配色方案

如果为标志提供了`标志值`，无论布尔型`标志状态`如何，该值始终会包含在[Flagsmith SDK](/clients/rest/)和API的返回结果中。

## 创建新功能标志

您可以在任意环境的Flags页面点击"Create Feature"按钮创建新功能标志。

标志默认为开启(true)或关闭(false)。您还可以选择存储和覆盖字符串及数值（整型和浮点型）。Flagsmith SDK允许对同一标志同时调用`hasFeature`和`getValue`方法，这些调用将返回布尔值以及已设置的字符串/数值。虽然不同语言存在差异，但SDK通常会在标志缺失或值未设置时返回False/Null。

![Image](/img/create-feature.png)

## 多变量功能标志

当您希望`标志值`从预定义的选项中选择时，可以创建多变量标志。项目中的每个环境都可以基于此列表定义和选择返回的值。多变量标志在两种核心场景中特别有用：

1. 需要从预设列表中控制`标志值`
2. 需要进行A/B测试。[了解更多](/advanced-use/ab-testing.md)

多变量标志值被定义为"Control"（对照组）和"Variations"（变量组）。当不传入[用户身份](/basic-features/managing-identities.md)而获取环境标志时，始终会返回Control值作为标志值。

:::important

Control和Variant的权重分配**_仅_**在获取特定身份的Flags时生效。如果仅检索环境标志而不传入身份信息，您将**_始终_**获得Control值。

:::

若为特定身份获取功能标志，Flagsmith引擎将根据环境中定义的权重比例返回对应值。

![Image](/img/multi-variate-flags.png)

如上图所示，约半数用户将获得`normal`值，约四分之一(25%)用户获得`large`，另外四分之一(25%)获得`huge`。注意可将权重设为100%以确保所有用户获得相同变体。

:::important

多变量_取值_在项目级别定义，但_权重分配_在环境级别设置。所有环境中各变体的字符串值保持一致，因此在某环境中修改变体值会同步影响项目内所有其他环境。

而各变体的_权重分配_则按环境独立配置。例如修改`development`环境的变体权重时，项目内其他环境的对应变体权重不会随之改变。

:::

### 多变量标志使用场景

多变量标志的核心应用场景是驱动[A/B测试](/advanced-use/ab-testing.md)。