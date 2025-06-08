---
description: Group your users based on a set of rules, then control Feature Flags and Remote Config for those groups.
---

# 管理用户分群

用户分群功能允许您基于一组规则对用户进行分组，从而为这些群体控制功能开关和远程配置。您可以创建一个分群，然后为该分群用户覆盖功能开关状态或远程配置值。

功能开关和配置的分群覆盖是在环境级别生效的，这意味着不同环境可以定义各自的分群覆盖规则。

分群规则**_仅_**在获取特定用户身份（Identity）的功能开关时生效。如果仅检索环境级别的功能开关而不传入用户身份，您的用户将**_永远不会_**被应用到任何分群，因为缺乏必要的上下文信息。

:::tip

分群规则不会回传给客户端SDK。它们仅在仪表盘中用于覆盖功能开关值，但我们的API永远不会将分群规则返回给SDK。
[了解更多架构细节](/guides-and-examples/integration-approaches#flags-are-evaluated-server-side)。

:::

## 示例 - 测试版用户

:::important

分群定义可在**项目**或**功能开关**级别设置。**项目级**分群在项目层面定义，可被该项目下的任意功能开关使用。**功能专属**分群仅能影响其所属的功能开关。

:::

假设您希望团队所有成员自动归入`测试版用户`。当前所有登录用户都通过邮箱地址进行[身份识别](/basic-features/managing-identities.md)，并附带其他[用户特征](/basic-features/managing-identities.md#identity-traits)。

您可以新建名为`测试版用户`的分群，并定义如下规则：

- `email_address` 包含 `@flagsmith.com`

![图片](/img/edit-segment.png)

定义分群后，您可以将其关联到特定功能开关。打开目标功能开关，导航至**分群覆盖**标签页，即可选择关联分群。这将允许您为该分群用户覆盖功能开关值。当识别到的用户属于该分群时，功能开关值将被覆盖。

![图片](/img/edit-feature-with-segment.png)

对于所有与测试版功能相关的功能开关，您都可以关联这个`测试版用户`分群，并将该分群的功能开关值设为`true`。编辑功能开关时，在"分群覆盖"下拉菜单中选择对应分群即可。

此时，所有使用包含`@flagsmith.com`邮箱地址登录的用户，其所有测试版功能都将被启用。

假设您随后与另一家公司合作，需要为其开放所有测试版功能。只需修改分群规则：

- `email_address` 包含 `@flagsmith.com`
- `email_address` 包含 `@solidstategroup.com`

现在所有使用`@solidstategroup.com`邮箱地址登录的用户也将自动获得测试版功能。

## 功能专属分群

您也可以在功能开关内部创建专属分群。这意味着只有该功能开关能使用这个分群。当您确定某个分群定义只需使用一次时，功能专属分群非常有用。进入功能开关的"分群覆盖"标签页，点击"创建功能专属分群"按钮即可。

![图片](/img/feature-specific-segment.png)

## 多变量值

如果使用[多变量功能开关](managing-features.md#multi-variate-flags)，您还可以在分群覆盖中重写各个变量的权重分配。

## 规则运算符

Flagsmith 支持的全部规则运算符如下：

- `完全匹配 (=)`
- `不匹配 (!=)`
- `百分比分割`
- `>`
- `>=`
- `<`
- `<=`
- `包含`
- `不包含`
- `正则匹配`
- `已设置`（当特性属性存在时）
- `未设置`（当特性属性不存在时）

所有运算符的行为都符合预期。部分运算符还具备特殊能力！

### 语义化版本感知运算符

同时支持以下[语义化版本](https://semver.org/)运算符：

- `SemVer >`
- `SemVer >=`
- `SemVer <`
- `SemVer <=`

例如，若您采用SemVer系统管理应用版本，可将版本号作为`特性`存储在Flagsmith中，随后创建如下规则：

`version` `SemVer >=` `4.2.52`

该分段规则将包含所有运行应用版本为`4.2.52`及以上的用户。

### 规则类型处理

当您在身份信息中存储特性值时，这些值会在API中关联类型存储：

- 字符串
- 布尔值
- 整数

定义分段规则时，规则值以字符串形式存储。当分段引擎运行时，规则值将被强制转换为特性值的类型。示例如下：

存储特性的示例（JavaScript）：

```javascript
flagsmith.identify('flagsmith_sample_user');
flagsmith.setTrait('accepted_cookies', true);
```

此处您为身份信息存储了原生`布尔值`。随后可定义分段规则，如`accepted_cookies=true`。由于身份特性`accepted_cookies`为布尔类型，分段引擎会将字符串值`accepted_cookies=true`强制转换为布尔值，从而确保逻辑正确执行。

若后续将特性值改为字符串类型，分段引擎仍可正常工作，因为身份的特性值已作为字符串存储。

```javascript
flagsmith.setTrait('accepted_cookies', 'true');
```

对于布尔值评估，以下字符串值会被视为`true`：

- `True`
- `true`
- `1`

### 百分比分割运算符

:::important

百分比分割运算符**_仅_**在针对特定身份获取功能标志时生效。若仅获取环境级别的功能标志而未传入身份信息，您的用户**_永远不会_**被纳入百分比分割分段。

:::

这是唯一不需要特性的运算符。可用于驱动[A/B测试](/advanced-use/ab-testing)和[分阶段功能发布](/guides-and-examples/staged-feature-rollouts#creating-staged-rollouts)。

当在覆盖功能的段中使用百分比分割运算符时，每次为该用户评估该功能时，每个用户都将被放入相同的"分桶"中，因此他们将始终获得相同的值。不同用户会根据您设置的分割百分比获得不同值。

### 取模运算符

该运算符执行[取模运算](https://en.wikipedia.org/wiki/Modulo_operation)。规则值采用`除数|余数`格式，适用于值为`整数`或`浮点数`的特性。例如：

`userId` `%` `2|0`

此分段规则将包含所有具有`int`或`float`类型`userId`特性，且该值除以2后余数为0的身份信息。

`userId % 2 == 0`

## 功能开关与远程配置优先级

功能开关状态和远程配置值可在三个不同层级定义：

1. 功能开关/配置的默认值本身
2. 与功能开关/配置关联的用户分群(Segment)
3. 在用户身份(Identity)级别覆盖的值

例如，功能开关"显示PayPal结账按钮"可在开关层级设为`false`，在"Beta测试用户"分群中设为`true`，然后针对特定用户身份覆盖为`false`。

处理此类情况时遵循以下优先级顺序：

1. 若用户身份存在覆盖值，则优先返回该值（高于分群和功能开关/配置）
2. 若无身份覆盖值，则检查分群设置并返回有效值
3. 若既无身份覆盖也无分群覆盖，则使用默认的功能开关/配置值

简而言之，优先级顺序为：

1. 用户身份
2. 用户分群
3. 功能开关