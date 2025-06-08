---
title: Flagsmith React SDK
sidebar_label: Next.js and SSR
description: Manage your Feature Flags and Remote Config with NextJS and SSR.
slug: /clients/next-ssr
---

该JavaScript库包含一个同构的捆绑库，允许您在服务器端获取功能标志，并将结果状态注入到您的应用程序中。

针对Next.js和服务端渲染(SSR)的示例应用可在[此处](https://github.com/flagsmith/flagsmith-js-client/tree/main/examples/nextjs)查看。

针对Svelte的示例应用可在[此处](https://github.com/flagsmith/flagsmith-js-client/tree/main/examples/svelte)查看。

Next.js中间件的示例应用可在[此处](https://github.com/flagsmith/flagsmith-js-client/tree/main/examples/nextjs-middleware)查看。

## 安装

### NPM

```bash
npm i flagsmith --save
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目内的单个环境进行初始化，例如开发环境或生产环境。您可以在环境设置页面找到客户端环境密钥。

![图片](/img/api-key.png)

## 比较SSR与客户端Flagsmith用法

SDK的初始化与使用方式与[JavaScript](/clients/javascript)和[React](/clients/react) SDK相同。主要区别在于Flagsmith应从`flagsmith/isomorphic`导入。

Next.js及任何基于JavaScript的SSR主要流程如下：

- 1: 在服务端获取标志，可选择通过[flagsmith.init({})](https://docs.flagsmith.com/clients/javascript#initialisation-options)传递身份标识
- 2: 使用[flagsmith.getState()](https://docs.flagsmith.com/clients/javascript#available-functions)将结果状态传递至客户端
- 3: 在客户端通过[flagsmith.setState(state)](https://docs.flagsmith.com/clients/javascript#available-functions)初始化flagsmith

### 示例：在Next.js中初始化SDK

基于上述流程，以下示例演示了如何在服务端获取标志并使用该状态初始化Flagsmith。

```javascript
import { FlagsmithProvider } from 'flagsmith/react';
import flagsmith from 'flagsmith/isomorphic';
function MyApp({ Component, pageProps, flagsmithState }) {
 return (
  <FlagsmithProvider flagsmith={flagsmith} serverState={flagsmithState}>
   <Component {...pageProps} />
  </FlagsmithProvider>
 );
}

MyApp.getInitialProps = async () => {
 // this could be getStaticProps too depending on your build flow
 // calls page's `getInitialProps` and fills `appProps.pageProps`
 await flagsmith.init({
  // fetches flags on the server
  environmentID: '<YOUR_ENVIRONMENT_ID>',
  identity: 'my_user_id', // optionaly specify the identity of the user to get their specific flags
 });
 return { flagsmithState: flagsmith.getState() };
};

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

此后SDK的使用方式与[React SDK指南](/clients/react)相同

### 示例：Next.js中间件中的Flagsmith

Flagsmith JS客户端包含`flagsmith/next-middleware`，可在Next.js中间件中像常规库一样使用。

```javascript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import flagsmith from 'flagsmith/next-middleware';

export async function middleware(request: NextRequest) {
 const identity = request.cookies.get('user');

 if (!identity) {
  // redirect to homepage
  return NextResponse.redirect(new URL('/', request.url));
 }

 await flagsmith.init({
  environmentID: '<YOUR_ENVIRONMENT_ID>',
  identity,
 });

 // Return a different URL based on a feature flag
 if (flagsmith.hasFeature('beta')) {
  return NextResponse.redirect(new URL(`/account-v2/`, request.url));
 }

 // Return a different URL based on a remote config
 const theme = flagsmith.getValue('colour');
 return NextResponse.redirect(new URL(`/account/${theme}`, request.url));
}

export const config = {
 matcher: '/login',
};
```

### 示例：不使用Next.js的SSR

同样的功能可以在不使用Next.js的情况下实现。

步骤1：初始化SDK并将结果状态传递至客户端。

```javascript
await flagsmith.init({
 // fetches flags on the server
 environmentID: '<YOUR_ENVIRONMENT_ID>',
 identity: 'my_user_id', // optionaly specify the identity of the user to get their specific flags
});
const state = flagsmith.getState(); // Pass this data to your client
```

步骤2：在客户端初始化SDK。

```javascript
flagsmith.setState(state); // set the state based on your
```

步骤3：可选强制客户端获取最新标志集

```javascript
flagsmith.getFlags(); // set the state based on your
```

此后SDK的使用方式与[JavaScript SDK指南](/clients/javascript)相同