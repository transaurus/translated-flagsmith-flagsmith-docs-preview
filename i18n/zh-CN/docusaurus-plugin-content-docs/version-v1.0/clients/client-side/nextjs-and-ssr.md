---
title: Flagsmith React SDK
sidebar_label: Next.js and SSR
description: Manage your Feature Flags and Remote Config with NextJS and SSR.
slug: /clients/next-ssr
---

该JavaScript库包含一个同构的捆绑库，允许您在服务器端获取功能标志，并将结果状态注入到您的应用程序中。

针对Next.js和服务端渲染(SSR)的示例应用可在此处查看：
[这里](https://github.com/flagsmith/flagsmith-js-client/tree/main/examples/nextjs)。

## 安装

### NPM

```bash
npm i flagsmith --save
```

## 基本用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)项目中的单个环境进行初始化，例如开发环境或生产环境。您可以在环境设置页面找到您的客户端环境密钥。

![Image](/img/api-key.png)

## 比较SSR与客户端Flagsmith用法

SDK的初始化与使用方式与[JavaScript](/clients/javascript)和[React](/clients/react) SDK相同。主要区别在于Flagsmith应从`flagsmith/isomorphic`导入。

与Next.js及任何基于JavaScript的SSR配合使用时，主要流程如下：

- 1: 在服务器端获取功能标志，可选地通过[flagsmith.init({})](https://docs.flagsmith.com/clients/javascript#initialisation-options)传递身份信息
- 2: 使用[flagsmith.getState()](https://docs.flagsmith.com/clients/javascript#available-functions)将结果状态传递给客户端
- 3: 在客户端通过[flagsmith.setState(state)](https://docs.flagsmith.com/clients/javascript#available-functions)初始化Flagsmith

### 示例：使用Next.js初始化SDK

基于上述流程，以下示例演示了如何在服务器端获取功能标志并使用该状态初始化Flagsmith。

```javascript
import { FlagsmithProvider } from 'flagsmith/react';
import flagsmith from 'flagsmith/isomorphic';
function MyApp({ Component, pageProps, flagsmithState }) {
    return (
        <FlagsmithProvider flagsmith={flagsmith}
                           serverState={flagsmithState}
>
            <Component {...pageProps} />
        </FlagsmithProvider>
    );
}


MyApp.getInitialProps = async () => { // this could be getStaticProps too depending on your build flow
  // calls page's `getInitialProps` and fills `appProps.pageProps`
  await flagsmith.init({ // fetches flags on the server
      environmentID: "<YOUR_ENVIRONMENT_ID>,
      identity: 'my_user_id' // optionaly specify the identity of the user to get their specific flags
  });
  return { flagsmithState: flagsmith.getState() }
}

export default MyApp;

```

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

此后，SDK的使用方式与[React SDK指南](/clients/react)相同

### 示例：不使用Next.js的SSR

不使用Next.js也可实现相同效果。

步骤1：初始化SDK并将结果状态传递给客户端。

```javascript
await flagsmith.init({ // fetches flags on the server
    environmentID: "<YOUR_ENVIRONMENT_ID>,
    identity: 'my_user_id' // optionaly specify the identity of the user to get their specific flags
});
const state = flagsmith.getState() // Pass this data to your client
```

步骤2：在客户端初始化SDK。

```javascript
flagsmith.setState(state); // set the state based on your
```

步骤3：可选地强制客户端获取一组新的功能标志

```javascript
flagsmith.getFlags(); // set the state based on your
```

此后，SDK的使用方式与[JavaScript SDK指南](/clients/javascript)相同