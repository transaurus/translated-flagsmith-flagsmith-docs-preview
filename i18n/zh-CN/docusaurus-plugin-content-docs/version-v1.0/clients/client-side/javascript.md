---
title: Flagsmith Javascript SDK
sidebar_label: Javascript
description: Manage your Feature Flags and Remote Config in your Javascript applications.
slug: /clients/javascript
---

该库可用于纯JavaScript、React（及所有其他主流框架/库）和React Native项目。客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-js-client)。

以下提供多种JavaScript框架的示例应用，包括React、Vue、Angular以及React Native：

- [Flagsmith框架示例](https://github.com/flagsmith/flagsmith-js-client/tree/main/examples)

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

SDK需针对[https://flagsmith.com](https://flagsmith.com)上项目内的单个环境进行初始化，例如开发环境或生产环境。环境密钥可在环境设置页面获取。

![图片](/img/api-key.png)

### 示例：初始化SDK

```javascript
import flagsmith from 'flagsmith or react-native-flagsmith'; //Add this line if you're using flagsmith via npm

flagsmith.init({
 environmentID: '<YOUR_ENVIRONMENT_KEY>',
 // api:"http://localhost:8000/api/v1/" set this if you are self hosting, and point it to your API
 cacheFlags: true, // stores flags in localStorage cache
 enableAnalytics: true, // See https://docs.flagsmith.com/flag-analytics/ for more info
 onChange: (oldFlags, params) => {
  //Occurs whenever flags are changed
  const { isFromServer } = params; //determines if the update came from the server or local cached storage

  //Check for a feature
  if (flagsmith.hasFeature('my_cool_feature')) {
   myCoolFeature();
  }

  //Or, use the value of a feature
  const bannerSize = flagsmith.getValue('banner_size');

  //Check whether value has changed
  const bannerSizeOld = oldFlags['banner_size'] && oldFlags['banner_size'].value;
  if (bannerSize !== bannerSizeOld) {
  }
 },
});
```

## 用户标识

标识用户可让您从Flagsmith仪表板定向特定用户，并配置功能开关和用户特征。该操作可在项目初始化前后调用，初始化后调用将重新从API获取功能开关。

可通过初始化客户端时或之后调用`flagsmith.identify`函数来标识用户。

用户功能管理请访问[https://flagsmith.com](https://flagsmith.com)对应项目的用户页面。![图片](/img/user-features.png)

### 示例：初始化客户端后标识用户

若初始化客户端时未指定用户标识，将获取环境默认功能开关（除非设置`preventFetch:true`参数）。

```javascript
import flagsmith from 'flagsmith';

flagsmith.init({
 environmentID: 'QjgYur4LQTwe5HpvbvhpzK',
 onChange: (oldFlags, params) => {
  //Occurs whenever flags are changed

  const { isFromServer } = params; //determines if the update came from the server or local cached storage

  //Set a trait against the Identity
  flagsmith.setTrait('favourite_colour', 'blue'); //This save the trait against the user, it can be queried with flagsmith.getTrait

  //Check for a feature
  if (flagsmith.hasFeature('my_power_user_feature')) {
   myPowerUserFeature();
  }

  //Check for a trait
  if (!flagsmith.getTrait('accepted_cookie_policy')) {
   showCookiePolicy();
  }

  //Or, use the value of a feature
  const myPowerUserFeature = flagsmith.getValue('my_power_user_feature');

  //Check whether value has changed
  const myPowerUserFeatureOld = oldFlags['my_power_user_feature'] && oldFlags['my_power_user_feature'].value;
  if (myPowerUserFeature !== myPowerUserFeatureOld) {
  }
 },
});

/*
Can be called either after you're done initialising the project or in flagsmith.init with its identity and trait properties 
to prevent flags being fetched twice.
*/
flagsmith.identify('flagsmith_sample_user'); //This will create a user in the dashboard if they don't already exist
```

### 示例：携带用户初始化SDK

通过identity属性初始化客户端将获取该用户专属功能开关（而非环境默认值）。此时还可指定用户特征，这些特征可能基于分段规则影响返回的功能开关。

```javascript
import flagsmith from 'flagsmith';

flagsmith.init({
 environmentID: 'QjgYur4LQTwe5HpvbvhpzK',
 identity: 'flagsmith_sample_user',
 traits: { age: 21, country: 'England' }, // these will add to the user's existing traits
 onChange: (oldFlags, params) => {
  //Occurs whenever flags are changed

  const { isFromServer } = params; //determines if the update came from the server or local cached storage

  //Set a trait against the Identity
  flagsmith.setTrait('favourite_colour', 'blue'); //This save the trait against the user, it can be queried with flagsmith.getTrait

  //Check for a feature
  if (flagsmith.hasFeature('my_power_user_feature')) {
   myPowerUserFeature();
  }

  //Check for a trait
  if (!flagsmith.getTrait('accepted_cookie_policy')) {
   showCookiePolicy();
  }

  //Or, use the value of a feature
  const myPowerUserFeature = flagsmith.getValue('my_power_user_feature');

  //Check whether value has changed
  const myPowerUserFeatureOld = oldFlags['my_power_user_feature'] && oldFlags['my_power_user_feature'].value;
  if (myPowerUserFeature !== myPowerUserFeatureOld) {
  }
 },
});
```

## API参考

完整类型定义参见[此处](https://github.com/Flagsmith/flagsmith-js-client/blob/main/index.d.ts#L35)。

### 初始化选项

| Property                                                         |                                                                                                              Description                                                                                                               | Required |                     Default Value |
| ---------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | -------: | --------------------------------: |
| `environmentID: string`                                          |                                                                     Defines which project environment you wish to get flags for. _example ACME Project - Staging._                                                                     |  **YES** |                              null |
| `onChange?: (previousFlags:IFlags, params:IRetrieveInfo)=> void` |                                     Your callback function for when the flags are retrieved `(previousFlags,{isFromServer:true/false,flagsChanged: true/false, traitsChanged:true/false})=>{...}`                                      |  **YES** |                              null |
| `onError?: (res:{message:string}) => void`                       |                                                                                    Callback function on failure to retrieve flags. `(error)=>{...}`                                                                                    |          |                              null |
| `asyncStorage?:any`                                              | Needed for cacheFlags option, used to tell the library what implementation of AsyncStorage your app uses, i.e. ReactNative.AsyncStorage vs @react-native-community/async-storage, for web this defaults to an internal implementation. |          |                              null |
| `cacheFlags?: boolean`                                           |                                    Any time flags are retrieved they will be cached, flags and identities will then be retrieved from local storage before hitting the API (see cache options) ```                                     |          |                              null |
| `cacheOptions?: {ttl?:number, skipAPI?:boolean}`                 |                                                   A ttl in ms (default to 0 which is infinite) and option to skip hitting the API in flagsmith.init if there's cache available. ```                                                    |          |            {ttl:0, skipAPI:false} |
| `enableAnalytics?: boolean`                                      |                                                               [Enable sending flag analytics](/advanced-use/flag-analytics.md) for getValue and hasFeature evaluations.                                                                |          |                             false |
| `enableLogs?: boolean`                                           |                                                                                                Enables logging for key Flagsmith events                                                                                                |          |                              null |
| `defaultFlags?: IFlags`                                          |                                                                    Allows you define default features, these will all be overridden on first retrieval of features.                                                                    |          |                              null |
| `preventFetch?: boolean`                                         |                                                                                     If you want to disable fetching flags and call getFlags later.                                                                                     |          |                             false |
| `state?: IState`                                                 |                                                                                   Set a predefined state, useful for SSR / isomorphic applications.                                                                                    |          |                             false |
| `api?: string`                                                   |                                                                   Use this property to define where you're getting feature flags from, e.g. if you're self hosting.                                                                    |          | https://api.flagsmith.com/api/v1/ |
| `identity?: string`                                              |                                                                           Specifying an identity will fetch flags for that identity in the initial API call.                                                                           |  **YES** |                              null |
| `traits?:Record<string, string or number or boolean>`            |                                                                           Specifying traits will send the traits for that identity in the initial API call.                                                                            |  **YES** |                              null |

### 可用函数

| Property                                                                                                                                        |                                                                                                                                Description                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| <code>init(initialisationOptions)=> Promise&lt;void&gt;</code>                                                                                  |                                                                                                            Initialise the sdk against a particular environment                                                                                                            |
| <code>hasFeature(key:string)=> boolean</code>                                                                                                   |                                                                                       Get the value of a particular feature e.g. `flagsmith.hasFeature("powerUserFeature") // true`                                                                                       |
| <code>getValue<T=string&#124;number&#124;boolean&#124;null>(key:string,{ json?:boolean, fallback?:T })=> string&#124;number&#124;boolean</code> |                                                  Get the value of a particular feature e.g. `flagsmith.getValue("font_size", { fallback: 12 }) // 10`, specifying json:true will automatically parse the value as JSON.                                                   |
| <code>getTrait(key:string)=> string&#124;number&#124;boolean</code>                                                                             |                                                               Once used with an identified user you can get the value of any trait that is set for them e.g. `flagsmith.getTrait("accepted_cookie_policy")`                                                               |
| <code>getState()=>IState</code>                                                                                                                 |                                                                               Retrieves the current state of flagsmith, useful in NextJS / isomorphic applications. `flagsmith.getState()`                                                                                |
| <code>setTrait(key:string, value:string&#124;number&#124;boolean)=> Promise&lt;IFlags&gt;</code>                                                |                                                              Once used with an identified user you can set the value of any trait relevant to them e.g. `flagsmith.setTrait("accepted_cookie_policy", true)`                                                              |
| <code>setTraits(values:Record<string, string&#124;number&#124;boolean>)=> Promise&lt;IFlags&gt;</code>                                          |                                                           Set multiple traits e.g. `flagsmith.setTraits({foo:"bar",numericProp:1,boolProp:true})`. Setting a value of null for a trait will remove that trait.                                                            |
| <code>incrementTrait(key:string, value:number)=> Promise&lt;IFlags&gt;</code>                                                                   |                                                                                You can also increment/decrement a particular trait them e.g. `flagsmith.incrementTrait("click_count", 1)`                                                                                 |
| <code>startListening(ticks=1000:number)=>void</code>                                                                                            |                                                                                                               Poll the api for changes every x milliseconds                                                                                                               |
| <code>stopListening()=>void</code>                                                                                                              |                                                                                                                           Stop polling the api                                                                                                                            |
| <code>getFlags()=> Promise&lt;IFlags&gt;</code>                                                                                                 |                                                         Trigger a manual fetch of the environment features, if a user is identified it will fetch their features. Resolves a promise when the flags are updated.                                                          |
| <code>identify(userId:string, traits?:Record<string, string or number or boolean>)=> Promise&lt;IFlags&gt;</code>                               | Identify as a user, optionally with traits e.g. `{foo:"bar",numericProp:1,boolProp:true}`. This will create a user for your environment in the dashboard if they don't exist, it will also trigger a call to `getFlags()`, resolves a promise when the flags are updated. |
| <code>logout()=>Promise&lt;IFlags&gt;</code>                                                                                                    |                                                                                                   Stop identifying as a user, this will trigger a call to `getFlags()`                                                                                                    |

## 多SDK实例

[1.5版本及以上](https://github.com/Flagsmith/flagsmith-js-client/releases/tag/1.5.0)支持创建多个Flagsmith SDK实例。此功能适用于需要同时标识多个用户，并保留对各用户`getValue`、`hasFeature`等方法的调用能力。

类型声明：

```javascript
export function createFlagsmithInstance (): IFlagsmith
```

使用示例：

```javascript
import { createFlagsmithInstance } from 'flagsmith';
const flagsmith = createFlagsmithInstance();
const flagsmithB = createFlagsmithInstance();

// now you can use flagsmith as before but in its own instance
```

## 常见问题

**如何协调`identify`、`setTraits`等函数与`init`的调用顺序？**

- `init`应在应用中仅调用一次，建议优先于其他flagsmith函数调用
- `init`默认会获取功能开关，可通过`preventFetch`参数禁用此行为。这在您确定随后将立即标识用户时特别有用

**何时触发onChange回调？**

`onChange`在以下情况触发功能开关获取时回调：

- init
- setTrait
- incrementTrait
- getFlags
- identify
- 通过本地存储评估的flags

最佳实践是将onChange与应用程序的状态管理结合使用，例如通过onChange触发一个动作来重新用`hasFeature`和`getValue`评估flags。

但如果这与您的开发模式不符，上述所有flagsmith函数都会返回一个Promise，该Promise会在获取到最新flags时解析。

例如通过以下方式：

```javascript
await flagsmith.setTrait('age', 21);
const hasFeature = flagsmith.hasFeature('my_feature');
```

OnChange回调时会携带变更信息，您可利用此特性避免不必要的重新渲染。

```javascript
onChange(this.oldFlags, {
 isFromServer: true, // flags have come from the server or local storage
 flagsChanged: deepEqualsCheck(oldFlags, newFlags),
 traitsChanged: deepEqualsCheck(oldFlags, newFlags),
});
```

**flags缓存机制如何工作？**

若在`init`时设置`cacheFlags`为true，SDK会将flags评估结果缓存至本地异步存储。当浏览器重新加载时，会立即触发携带本地存储flags的onChange事件。具体流程如下：

1. 调用`init`

2. 若启用`cacheFlags`，则检查本地存储中是否存在缓存的flags和traits

3. 若发现本地存储的flags，则立即用这些flags触发`onChange`

4. 同时会获取最新flags，这将导致再次触发`onChange`回调

5. 每次获取到新flags时都会更新本地存储

默认情况下这些flags会永久保存，您可以通过删除`localStorage`中的`"BULLET_TRAIN_DB"`来清除缓存。