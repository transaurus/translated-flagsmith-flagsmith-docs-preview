---
title: Heap Analytics Integration
sidebar_label: Heap Analytics
hide_title: true
---

![Heap Analytics](/img/integrations/heap/heap-logo.svg)

您可以将Flagsmith与Heap Analytics集成。该集成会自动将已识别用户的特征标志状态发送至Heap平台，用于群体分析、A/B测试等功能。具体流程如下：

## 集成设置

1. 从Heap管理项目页面获取项目环境ID（路径：账户 > 管理 > 项目 > 环境 > 获取ID）
2. 将该ID密钥添加到Flagsmith（路径：集成 > 添加Heap集成）
3. Flagsmith SDK调用`获取身份特征标志`接口时，会将该用户的所有特征标志评估结果作为[事件](https://developers.heap.io/reference#track-1)发送至Heap

## 工作原理

:::tip

对于包含远程配置值的特征标志：
- 若标志为`启用`状态，Flagsmith会将该标志值传递给Heap
- 若标志无远程配置值，则仅传递布尔状态

:::

身份特征标志值会被传递至Heap平台。

以下是Flagsmith中的示例用户：

![Heap Analytics](/img/integrations/heap/heap-integration-2.png)

当我们调用Flagsmith API获取该用户的特征标志时：

```bash
curl 'https://edge.api.flagsmith.com/api/v1/identities/?identifier=development_user_123456' \
  -H 'x-environment-key: 8KzETdDeMY7xkqkSkY3Gsg'
```

在Heap仪表盘中，您可以看到已发送至Heap平台的用户及其特征标志数据：

![Heap Analytics](/img/integrations/heap/heap-integration-1.png)

## 使用场景

完成集成配置后，您可以根据用户接触到的特征标志在Heap中进行人群细分。

这意味着您可以通过Flagsmith的细分规则驱动A/B测试，相关数据会自动同步至Heap。

## 使用Heap和Flagsmith进行多变量测试

集成运行数日后，特征标志值会作为"自定义事件属性"出现在Heap中，这些属性隶属于"Flagsmith特征标志"事件。以下示例演示如何评估启用特征标志的效果（以我们平台中"深色模式"开关为例）：

### 第一步 - 创建自定义事件定义

在Heap中进入：定义 > 新建定义 > 新建事件。按以下示例配置事件：

![Heap Analytics Step 1](/img/integrations/heap/heap-mv-step-1.png)

### 第二步 - 基于该自定义事件属性创建细分群体

在Heap中进入：定义 > 新建定义 > 新建细分。按以下示例配置细分：

![Heap Analytics Step 2](/img/integrations/heap/heap-mv-step-2.png)

### 第三步 - 创建分析报告

创建基于Flagsmith标志值的细分后，您可以在Heap报告中应用该细分。下图展示了深色模式与非深色模式用户的转化率对比（注意页面底部的"群体分析-转化率"）：

![Heap Analytics Step 3](/img/integrations/heap/heap-mv-step-3.png)

## 集成注意事项

必须保证两个平台的用户识别方式一致。Flagsmith的`身份ID`需与Heap的`identity`保持相同。