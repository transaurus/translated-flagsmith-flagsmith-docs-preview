---
title: Staged Feature Rollouts
---

## 什么是分阶段功能发布

分阶段功能发布允许您在小部分用户群体中测试新功能。如果功能表现良好，您可以逐步增加可见该功能的用户比例，直至覆盖全部用户群。

这种方法能提升新功能发布的信心。若发布过程中出现问题，只需禁用功能开关即可在应用中隐藏该功能。

## 创建分阶段发布

:::important

分阶段发布**_仅_**在针对特定身份获取功能开关时生效。如果仅获取环境级别的开关而未传入身份标识，用户将永远不会被包含在"百分比分割"的区段中。

:::

您可以通过创建[区段](/basic-features/managing-segments.md)并添加定义为"百分比分割"条件的规则来实现分阶段发布。设置1到100之间的百分比值，即可定义用户群中被包含在该区段的比例。

![图示](/img/percent-rollout.png)

创建区段后，您可以按照常规[区段](/basic-features/managing-segments.md)操作将其关联到功能开关。

请注意，"百分比分割"规则可以与其他区段规则组合使用。

## 工作原理

每个身份标识和区段都有唯一ID。这两组数据经合并哈希后，会生成一个0.0到1.0之间的浮点值。该数值将与"百分比分割"规则进行比对。

### 示例说明

举例来说，对于单个身份标识的处理流程如下：

1. 获取内部区段ID和身份ID并合并为字符串
2. 对该字符串进行哈希运算
3. 基于哈希值生成0到1之间的浮点数

因此每个区段/身份组合都会生成0到1之间的值。通过采用的哈希算法，我们确保数值在0到1区间内均匀分布。

假设某身份标识的哈希值为0.351。若创建30%分割的区段，该身份将不被包含（因为0.351 > 0.3）。当将区段调整为40%分割时，该身份就会被包含（0.4 > 0.351）。由于区段ID创建后不变，终端用户能获得一致的体验。

另一个身份标识的哈希值可能是0.94。这种情况下，无论是30%还是40%的分割设置，该身份都不会被包含在目标区段中。