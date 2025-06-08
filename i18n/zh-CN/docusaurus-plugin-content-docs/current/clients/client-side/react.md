---
title: Flagsmith React SDK
sidebar_label: React and React Native
description: Manage your Feature Flags and Remote Config with React and React Native Hooks.
slug: /clients/react
---

该库包含React/React Native Hooks，允许您查询单个功能标记并限制重新渲染。

以下提供多种React、React Native和Next.js的示例应用：

- [React应用示例](https://github.com/Flagsmith/flagsmith-js-client/tree/main/examples/react)
- [React Native应用示例](https://github.com/Flagsmith/flagsmith-js-client/tree/main/examples/react)
- [Next.js应用示例](https://github.com/Flagsmith/flagsmith-js-client/tree/main/examples/nextjs)

## 安装指南

### NPM安装

:::tip

我们还提供flagsmith-es模块，适用于需要使用[ES模块](https://262.ecma-international.org/6.0/)的用户。

:::

```bash
npm i flagsmith --save
```

### React Native的NPM安装

:::tip

ReactNative SDK与Flagsmith核心实现完全一致，但部分底层库（如AsyncStorage）默认使用React Native兼容版本。

:::

```bash
npm i react-native-flagsmith --save
```

### 通过JavaScript CDN引入

```html
<script src="https://cdn.jsdelivr.net/npm/flagsmith/index.js"></script>
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目的特定环境（如开发或生产环境）进行初始化。![图片](/img/api-key.png)

### 第一步：用Flagsmith Provider包裹应用

使用FlagsmithProvider组件包裹应用后，可通过`useFlagsmith`和`useFlags`钩子在全局获取React Context。

```javascript
import flagsmith from 'flagsmith'
import {FlagsmithProvider} from 'flagsmith/react'

export function AppRoot() {
  <FlagsmithProvider options={{
      environmentID: "<YOUR_ENVIRONMENT_KEY>",
  }} flagsmith={flagsmith}>
    {...}
  </FlagsmithProvider>
};
```

向Flagsmith provider传递配置选项将初始化客户端，具体API参数参考[此处](/clients/javascript#initialisation-options)。

**高级用法：在渲染FlagsmithProvider前初始化**

若需在React渲染前初始化客户端（如Redux或SSR场景），可调用[flagsmith.init](https://docs.flagsmith.com/clients/javascript#example-initialising-the-sdk)方法，且不在FlagsmithProvider组件中传递options属性。

### 第二步：使用useFlags获取功能值及启用状态

被FlagsmithProvider包裹的组件可通过`useFlags`钩子获取功能值、启用状态及用户特征。

```javascript
import { useFlags } from 'flagsmith/react';

export function MyComponent() {
 const flags = useFlags(['font_size'], ['example_trait']); // only causes re-render if specified flag values / traits change
 return (
  <div className="App">
   font_size: {flags.font_size.value}
   example_trait: {flags.example_trait}
  </div>
 );
}
```

## useFlags API参考

```javascript
useFlags(requiredFlags:string[], requiredTraits?:string[])=> {[key:string]: IFlagsmithTrait  or IFlagsmithFeature}
```

## FlagsmithProvider API参考

| Property                 |                                                  Description                                                   | Required | Default Value |
| ------------------------ | :------------------------------------------------------------------------------------------------------------: | -------: | ------------: |
| `flagsmith: IFlagsmith`  |                           Defines the flagsmith instance that the provider will use.                           |  **YES** |          null |
| `options?: ` IInitConfig |    Initialisation options to use. If you don't provide this you will have to call flagsmith.init elsewhere.    |          |          null |
| `serverState?: IState`   | Used to pass an initial state, in most cases as a result of SSR flagsmith.getState(). See [Next.js and SSR](/) |          |          null |

### 第三步：使用useFlagsmith访问Flagsmith实例

被FlagsmithProvider包裹的组件可通过`useFlagsmith`钩子访问Flagsmith实例。

```javascript
import React from 'react';
import { useFlags, useFlagsmith } from 'flagsmith/react';

export function MyComponent() {
 const flags = useFlags(['font_size'], ['example_trait']); // only causes re-render if specified flag values / traits change
 const flagsmith = useFlagsmith();
 const identify = () => {
  // This will re-render the component if the user has the trait example_trait or they have a different feature value for font_size
  flagsmith.identify('flagsmith_sample_user');
 };
 const logout = () => {
  // This will re-render the component if the user has the trait example_trait or they have a different feature value for font_size
  flagsmith.logout();
 };
 return (
  <div className="App">
   font_size: {flags.font_size?.value}
   example_trait: {flags.example_trait}
   {flagsmith.identity ? <button onClick={logout}>Logout</button> : <button onClick={identify}>Identify</button>}
  </div>
 );
}
```

## useFlagsmith API参考

```javascript
useFlagsmith()=> IFlagsmith
```