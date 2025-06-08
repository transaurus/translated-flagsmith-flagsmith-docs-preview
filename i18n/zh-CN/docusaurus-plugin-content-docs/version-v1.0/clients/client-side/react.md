---
title: Flagsmith React SDK
sidebar_label: React and React Native
description: Manage your Feature Flags and Remote Config with React and React Native Hooks.
slug: /clients/react
---

该库包含React/React Native Hooks，允许您查询各个功能开关和标志位，从而减少重复渲染。

以下提供多种React、React Native及Next.js的示例应用：

- [React应用示例](https://github.com/Flagsmith/flagsmith-js-client/tree/main/examples/react)
- [React Native应用示例](https://github.com/Flagsmith/flagsmith-js-client/tree/main/examples/react)
- [Next.js应用示例](https://github.com/Flagsmith/flagsmith-js-client/tree/main/examples/nextjs)

## 安装指南

### NPM安装

```bash
npm i flagsmith --save
```

### React Native的NPM安装

```bash
npm i react-native-flagsmith --save
```

### 通过JavaScript CDN引入

```html
<script src="https://cdn.jsdelivr.net/npm/flagsmith/index.js"></script>
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目内的单个环境（如开发或生产环境）进行初始化。![图片](/img/api-key.png)

### 步骤一：使用Flagsmith Provider包裹应用

用FlagsmithProvider组件包裹应用后，将通过React Context全局提供`useFlagsmith`和`useFlags`钩子。

```javascript
import flagsmith from 'flagsmith'
import {FlagsmithProvider} from 'flagsmith/react'

export function AppRoot() {
  <FlagsmithProvider options={{
      environmentID: "<YOUR_ENVIRONMENT_KEY>",
  }} flagsmith={flagsmith}>
    {...}
  </FlagsmithProvider>
);
```

向Flagsmith Provider传递配置选项可初始化客户端，具体API参数参考[此处](/clients/javascript#initialisation-options)。

**高级用法：在渲染FlagsmithProvider前初始化**

若需在React渲染前初始化Flagsmith客户端（如在redux或SSR中），可调用[flagsmith.init](https://docs.flagsmith.com/clients/javascript#example-initialising-the-sdk)并确保FlagsmithProvider组件不接收options属性。

### 步骤二：使用useFlags获取功能开关值与启用状态

被FlagsmithProvider包裹的组件可通过`useFlags`钩子获取功能开关值、启用状态及用户特征。

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

### 步骤三：使用useFlagsmith访问Flagsmith实例

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