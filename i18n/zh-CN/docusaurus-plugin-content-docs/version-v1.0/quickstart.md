---
sidebar_position: 2
sidebar_label: Quick Start
---

# Flagsmith 快速入门指南

让我们在5分钟内完成配置。我们将按照以下步骤操作：

1. 在 [Flagsmith.com](https://flagsmith.com/) 创建账户并添加第一个功能开关
2. 将我们的 JavaScript SDK 导入网页
3. 连接 Flagsmith API 获取功能开关
4. 根据开关值更新应用程序

## 1. 创建 Flagsmith 账户

:::tip

为了快速开始，我们将使用 flagsmith.com 的托管服务，但您也可以轻松通过 [Docker 本地运行平台](deployment/docker.md)。

:::

前往 [Flagsmith](https://app.flagsmith.com/signup) 创建账户。我们将创建一个组织和一个项目。

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_1.png"/></div>

Flagsmith 通过项目管理功能开关，现在让我们创建一个：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_2.png"/></div>

Flagsmith 将项目组织到独立的环境中。创建项目时，系统会自动生成`开发`和`生产`环境。稍后我们会介绍这些环境。现在先创建第一个功能开关，这个开关将控制一个按钮是否显示在我们的简易网页上。

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_3.png"/></div>

功能开关可以是`布尔值`、`字符串值`或两者的组合。目前我们仅使用开关的`布尔值`来控制按钮显示。现在创建一个名为`show_demo_button`的开关，默认保持禁用状态：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_4.png"/></div>

## 2. 导入 JavaScript SDK

现在我们已经设置好功能开关，接下来将其集成到应用中。这里有一个（非常！）简单的网页：

```html
<!DOCTYPE html>
<html lang="en">
 <head>
  <meta charset="utf-8" />
  <title>Flagsmith Quickstart Guide</title>
 </head>
 <body>
  <h1>Here's our button!</h1>
  <div id="submit_button">
   <input type="submit" value="Flagsmith Quickstart Button!" />
  </div>
 </body>
</html>
```

页面非常简单，效果如下：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_8.png"/></div>

根据 [JavaScript 文档](clients/client-side/javascript.md)，我们将内联导入 SDK 到网页：

```html
<script src="https://cdn.jsdelivr.net/npm/flagsmith/index.js"></script>
```

## 3. 连接 Flagsmith API

现在可以连接 Flagsmith API 获取功能开关。初始化 SDK 时需要提供环境ID，这样 SDK 才能知道从哪个项目的哪个环境获取开关。进入 Flagsmith 的环境设置页面，复制 API 密钥：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_6.png"/></div>

将 API 密钥粘贴到以下代码中：

```html
<script>
 flagsmith.init({
  environmentID: '<add your API key here!>',
  onChange: (oldFlags, params) => {},
 });
</script>
```

当浏览器加载网页时，会下载 JavaScript SDK 并向`api.flagsmith.com`发起调用以获取环境开关。您可以在浏览器网络标签页中看到这个调用：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_7.png"/></div>

这里可以看到 Flagsmith API 返回的开关值为`"enabled": false`。

## 4. 集成到应用

现在将这个值绑定到按钮，让开关值控制按钮的显隐状态。

```html
<script>
 flagsmith.init({
  environmentID: '<add your API key here!>',
  onChange: (oldFlags, params) => {
   if (flagsmith.hasFeature('show_demo_button')) {
    var submit_button = document.getElementById('submit_button');
    submit_button.style.display = 'block';
   }
  },
 });
</script>
```

这段代码设置了回调函数，当 Flagsmith API 返回响应时触发。我们只需检查开关状态并根据结果设置显示属性。

现在完整网页代码如下：

```html
<!DOCTYPE html>
<html lang="en">
 <head>
  <meta charset="utf-8" />
  <title>Flagsmith Quickstart Guide</title>
  <script src="https://cdn.jsdelivr.net/npm/flagsmith/index.js"></script>
  <script>
   flagsmith.init({
    environmentID: 'ZfmJTbLQZrhZVHkVhXbsNi',
    onChange: (oldFlags, params) => {
     if (flagsmith.hasFeature('show_demo_button')) {
      var submit_button = document.getElementById('submit_button');
      submit_button.style.display = 'block';
     }
    },
   });
  </script>
 </head>
 <body>
  <h1>Here's our button!</h1>
  <div id="submit_button" style="display:none">
   <input type="submit" value="Flagsmith Quickstart Button!" />
  </div>
 </body>
</html>
```

返回浏览器刷新页面，您会发现按钮已经消失。

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_9.png"/></div>

现在，我们已经将按钮的可见性控制权交给了Flagsmith的功能开关！您可以返回Flagsmith仪表板并启用该开关：

<div style={{textAlign: 'center'}}><img width="75%" src="/img/quickstart/demo_create_10.png"/></div>

返回浏览器刷新页面，按钮将重新出现。

## 完成设置

这是一个相当简单的演示，但它涵盖了将Flagsmith集成到应用程序中的核心概念。接下来，您可能需要查阅文档中的以下部分：

- 更深入的应用概览 - [功能管理](basic-features/managing-features.md)、
  [身份管理](basic-features/managing-identities.md)和[用户分群](basic-features/managing-segments.md)。
- 关于我们的[API和SDK](clients/rest.md)的更多细节。
- 如何[自行部署Flagsmith](deployment/overview.md)或使用我们的[托管API](https://flagsmith.com/)。