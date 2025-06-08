---
title: Heap Analytics Integration
sidebar_label: Heap Analytics
hide_title: true
---

![Heap Analytics](/img/integrations/heap/heap-logo.svg)

您可以将Flagsmith与Heap Analytics集成。该集成会自动将已识别用户的特征标志状态发送至Heap平台，用于群体分析、A/B测试等场景。具体流程如下：

## 集成设置

1. 从Heap项目管理页面获取项目环境ID（路径：Account > Manage > Projects > Environment > Get ID）
2. 将该ID密钥添加至Flagsmith（路径：Integrations > Add Heap Integration）
3. 当Flagsmith SDK调用`Get Identity Flags`接口时，系统会自动将该用户的所有特征标志评估结果作为[事件](https://developers.heap.io/reference#track-1)发送至Heap

## 工作原理

:::tip

对于包含远程配置值的特征标志：
- 若标志处于`启用`状态，Flagsmith会将其配置值发送至Heap
- 若标志无远程配置值，则仅传递布尔状态

:::

用户身份特征标志值将被传递至Heap平台。

以下是Flagsmith中的示例用户：

![Heap Analytics](/img/integrations/heap/heap-integration-2.png)

当我们调用Flagsmith API获取该用户的特征标志时：

```bash
curl 'https://api.flagsmith.com/api/v1/identities/?identifier=development_user_123456' \
  -H 'x-environment-key: 8KzETdDeMY7xkqkSkY3Gsg'
```

在Heap仪表盘中，您可以看到已发送至Heap平台的用户数据及特征标志信息。

![Heap Analytics](/img/integrations/heap/heap-integration-1.png)

## 使用场景

完成集成后，您可以根据用户接触到的特征标志对Heap身份进行细分。

这意味着您可以通过Flagsmith细分驱动A/B测试，相关数据将自动同步至Heap平台。

## 使用Heap与Flagsmith进行多变量测试

集成运行数日后，特征标志值会作为"Flagsmith Feature Flags"事件的子属性出现在Heap的"自定义事件属性"中。以下示例演示如何评估"深色模式"标志的启用效果：

### 步骤1 - 创建自定义事件定义

在Heap中进入Definitions > New Definition > New Event，按如下方式配置事件：

![Heap Analytics Step 1](/img/integrations/heap/heap-mv-step-1.png)

### 步骤2 - 基于自定义事件属性创建细分群体

在Heap中进入Definitions > New Definition > New Segment，按如下方式配置细分：

![Heap Analytics Step 2](/img/integrations/heap/heap-mv-step-2.png)

### 步骤3 - 创建分析报告

基于特征标志值创建细分后，可在Heap报告中应用该细分。下图展示了深色模式与非深色模式用户的转化率对比（注意页面底部的"群体分析-转化率"）：

![Heap Analytics Step 3](/img/integrations/heap/heap-mv-step-3.png)

## 集成注意事项

需确保两个平台的用户标识一致，Flagsmith的`Identity ID`必须与Heap的`identity`保持相同。