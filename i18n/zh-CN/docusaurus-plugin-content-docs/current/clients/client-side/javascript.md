---
title: Flagsmith JavaScript SDK
sidebar_label: JavaScript
description: Manage your Feature Flags and Remote Config in your JavaScript applications.
slug: /clients/javascript
---

该库可应用于纯JavaScript、React（及所有主流框架/库）和React Native项目。客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-js-client)。

以下提供多种JavaScript框架的示例应用，包括React、Vue、Angular以及React Native：

- [Flagsmith框架示例](https://github.com/flagsmith/flagsmith-js-client/tree/main/examples)

## 安装

### NPM

:::tip

如需使用[ES](https://262.ecma-international.org/6.0/)模块，我们另提供flagsmith-es版本。

:::

```bash
npm i flagsmith --save
```

### React Native版NPM

:::tip

ReactNative SDK与Flagsmith核心实现完全一致，但其底层库（如AsyncStorage）默认采用React Native兼容方案。

:::

```bash
npm i react-native-flagsmith --save
```

### 通过JavaScript CDN

```html
<script src="https://cdn.jsdelivr.net/npm/flagsmith/index.js"></script>
```

## 基础用法

SDK需针对[https://flagsmith.com](https://flagsmith.com)项目中单个环境（如开发或生产环境）初始化。客户端环境密钥可在环境设置页面获取。

![图片](/img/api-key.png)

### 示例：初始化SDK

```javascript
import flagsmith from 'flagsmith or react-native-flagsmith'; //Add this line if you're using flagsmith via npm

flagsmith.init({
 environmentID: '<YOUR_CLIENT_SIDE_ENVIRONMENT_KEY>',
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
   // Do something!
  }
 },
});
```

### 设置默认功能开关

初始化SDK时可定义默认开关值，确保在[无法获取API响应时](/guides-and-examples/defensive-coding)应用仍能按预期运行。

```javascript
import flagsmith from 'flagsmith or react-native-flagsmith'; //Add this line if you're using flagsmith via npm

flagsmith.init({
    environmentID: '<YOUR_CLIENT_SIDE_ENVIRONMENT_KEY>',
    defaultFlags: {
        feature_a: { enabled: false},
        font_size: { enabled: true, value: 12 },
    }
    onChange: (oldFlags, params) => {
        ...
    },
});
```

### 通过CI/CD设置默认开关

可在构建流水线中使用[CLI工具](https://github.com/Flagsmith/flagsmith-cli)为前端应用自动配置默认开关。

主要实施步骤：

1. 安装CLI `npm i flagsmith-cli --save-dev`
2. 通过npm postinstall调用CLI生成`flagsmith.json`（每次执行`npm install`时触发）：
   - 使用环境变量 `export FLAGSMITH_ENVIRONMENT=<您的客户端环境密钥> flagsmith get`
   - 或手动指定环境密钥 `flagsmith get <您的客户端环境密钥>`
3. 在应用中通过结果JSON初始化Flagsmith，将在调用API前优先加载默认开关：`flagsmith.init({environmentID: json.environmentID, state:json})`

完整示例参见[此处](https://github.com/Flagsmith/flagsmith-cli/tree/main/example)，CLI命令列表参阅[文档](https://github.com/Flagsmith/flagsmith-cli)。

## 用户识别

识别用户后可通过Flagsmith控制台定向配置功能开关和用户特征。此操作可在项目初始化前后调用，后续调用将重新从API获取特征。

可通过初始化时配置或后续调用`flagsmith.identify`函数实现用户识别。

用户功能管理入口：[https://flagsmith.com](https://flagsmith.com)对应项目的用户模块。![图片](/img/user-features.png)

### 示例：初始化后识别用户

未指定用户身份初始化时，客户端将获取环境默认开关（除非设置`preventFetch:true`参数）。

```javascript
import flagsmith from 'flagsmith';

flagsmith.init({
 environmentID: '<YOUR_CLIENT_SIDE_ENVIRONMENT_KEY>',
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
   // Do something!
  }
 },
});

/*
Can be called either after you're done initialising the project or in flagsmith.init with its identity and trait properties 
to prevent flags being fetched twice.
*/
flagsmith.identify('flagsmith_sample_user'); //This will create a user in the dashboard if they don't already exist
```

### 示例：初始化包含用户的SDK

使用身份属性初始化客户端将获取该用户的特性标志，而非环境默认值。此时还可指定特征值，这些特征可能通过分段覆盖决定返回的标志。

```javascript
import flagsmith from 'flagsmith';

flagsmith.init({
 environmentID: '<YOUR_CLIENT_SIDE_ENVIRONMENT_KEY>',
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

所有函数及属性类型可查阅[此处](https://github.com/Flagsmith/flagsmith-js-client/blob/main/types.d.ts#L35)。

### 初始化选项

| Property                                                                       |                                                                                                              Description                                                                                                               | Required |                          Default Value |
| ------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | -------: | -------------------------------------: |
| `environmentID: string`                                                        |                                                                     Defines which project environment you wish to get flags for. _example ACME Project - Staging._                                                                     |  **YES** |                                   null |
| `onChange?: (previousFlags:IFlags, params:IRetrieveInfo)=> void`               |                                     Your callback function for when the flags are retrieved `(previousFlags,{isFromServer:true/false,flagsChanged: true/false, traitsChanged:true/false})=>{...}`                                      |  **YES** |                                   null |
| `onError?: (res:{message:string}) => void`                                     |                                                                                    Callback function on failure to retrieve flags. `(error)=>{...}`                                                                                    |          |                                   null |
| `asyncStorage?:any`                                                            | Needed for cacheFlags option, used to tell the library what implementation of AsyncStorage your app uses, i.e. ReactNative.AsyncStorage vs @react-native-community/async-storage, for web this defaults to an internal implementation. |          |                                   null |
| `cacheFlags?: boolean`                                                         |                                      Any time flags are retrieved they will be cached, flags and identities will then be retrieved from local storage before hitting the API (see cache options)                                       |          |                                   null |
| `cacheOptions?: {ttl?:number, skipAPI?:boolean}`                               |                                                     A ttl in ms (default to 0 which is infinite) and option to skip hitting the API in flagsmith.init if there's cache available.                                                      |          |                 {ttl:0, skipAPI:false} |
| `enableAnalytics?: boolean`                                                    |                                                               [Enable sending flag analytics](/advanced-use/flag-analytics.md) for getValue and hasFeature evaluations.                                                                |          |                                  false |
| `enableLogs?: boolean`                                                         |                                                                                                Enables logging for key Flagsmith events                                                                                                |          |                                   null |
| `defaultFlags?: {flag_name: {enabled: boolean, value: string,number,boolean}}` |                                                                    Allows you define default features, these will all be overridden on first retrieval of features.                                                                    |          |                                   null |
| `preventFetch?: boolean`                                                       |                                                                                     If you want to disable fetching flags and call getFlags later.                                                                                     |          |                                  false |
| `state?: IState`                                                               |                                                                                   Set a predefined state, useful for SSR / isomorphic applications.                                                                                    |          |                                  false |
| `api?: string`                                                                 |                                                                   Use this property to define where you're getting feature flags from, e.g. if you're self hosting.                                                                    |          | https://edge.api.flagsmith.com/api/v1/ |
| `eventSourceUrl?: string`                                                      |                                                 Use this property to define where you're getting real-time flag update events (server sent events) from, e.g. if you're self hosting.                                                  |          | https://edge.api.flagsmith.com/api/v1/ |
| `identity?: string`                                                            |                                                                           Specifying an identity will fetch flags for that identity in the initial API call.                                                                           |  **YES** |                                   null |
| `traits?:Record<string, string or number or boolean>`                          |                                                                           Specifying traits will send the traits for that identity in the initial API call.                                                                            |  **YES** |                                   null |

### 可用函数

| Property                                                                                                                                        |                                                                                                                                Description                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| <code>init(initialisationOptions)=> Promise&lt;void&gt;</code>                                                                                  |                                                                                                            Initialise the sdk against a particular environment                                                                                                            |
| <code>hasFeature(key:string)=> boolean</code>                                                                                                   |                                                                                       Get the value of a particular feature e.g. `flagsmith.hasFeature("powerUserFeature") // true`                                                                                       |
| <code>getValue<T=string&#124;number&#124;boolean&#124;null>(key:string,{ json?:boolean, fallback?:T })=> string&#124;number&#124;boolean</code> |                                                  Get the value of a particular feature e.g. `flagsmith.getValue("font_size", { fallback: 12 }) // 10`, specifying json:true will automatically parse the value as JSON.                                                   |
| <code>getTrait(key:string)=> string&#124;number&#124;boolean</code>                                                                             |                                                               Once used with an identified user you can get the value of any trait that is set for them e.g. `flagsmith.getTrait("accepted_cookie_policy")`                                                               |
| <code>getAllTraits()=> Record&lt;string,string&#124;number&#124;boolean&gt;</code>                                                              |                                                                      Once used with an identified user you can get a key value pair of all traits that are set for them e.g. `flagsmith.getTraits()`                                                                      |
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

[1.5及以上版本](https://github.com/Flagsmith/flagsmith-js-client/releases/tag/1.5.0)支持创建多个Flagsmith SDK实例。此功能适用于需在应用中同时识别多个用户，并保留对各用户getValue、hasFeature等方法的调用权限的场景。

类型：

```javascript
export function createFlagsmithInstance (): IFlagsmith
```

用法：

```javascript
import { createFlagsmithInstance } from 'flagsmith';
const flagsmith = createFlagsmithInstance();
const flagsmithB = createFlagsmithInstance();

// now you can use flagsmith as before but in its own instance
```

## JSON特性值

Flagsmith JavaScript客户端支持JSON格式的远程配置/特性值。调用`flagsmith.getValue`时指定`json:true`将尝试将特性值解析为JSON，若解析失败则回退至`fallback`值。

```javascript
const json = flagsmith.getValue('json_value', {
 json: true,
 fallback: { foo: null, bar: null },
});
```

## TypeScript支持

Flagsmith为JavaScript客户端提供完整的TypeScript支持，主类型定义文件参见[此处](https://github.com/Flagsmith/flagsmith-js-client/blob/main/types.d.ts#L35)。还可通过泛型强制约束特性名与特征名的类型安全：

给定已类型化的标志和特征：

```typescript
type FlagOptions = 'font_size' | 'hero';
type TraitOptions = 'example_trait';
```

现可强制实施这些类型：

```typescript
// enforces you passing the correct key to flagsmith.getValue(flag:FlagOptions), flagsmith.getTrait(trait:TraitOptions)
import flagsmith from 'flagsmith';
const typedFlagsmith = flagsmith as IFlagsmith<FlagOptions, TraitOptions>;

// Similarly for the useFlagsmith hook is typed with useFlagsmith(flags:FlagOptions[],traits:TraitOptions[])
const flagsmith = useFlagsmith<FlagOptions, TraitOptions>(); // enforces flagsmith.getValue()

// for useFlags this will ensure you only can pass correct keys also
const flags = useFlags<FlagOptions, TraitOptions>(['font_size'], ['example_trait']);

// for getting JSON values this will type the return
const json = flagsmith.getValue<{ foo: string | null; bar: string | null }>('json_value', {
 json: true,
 fallback: { foo: null, bar: null },
});
console.log(json.foo); // typed as {foo: string|null, bar: string|null}

// If a type is not specified for getValue it will asume it from the type of fallback. In this case, a number.
const font_size = flagsmith.getValue('font_size', { fallback: 12 });
```

## Datadog RUM JavaScript SDK集成

:::caution

此功能在Datadog中仍处于测试阶段。启用下方集成前请先联系Datadog代表。

:::

Flagsmith JavaScript SDK可配置为：将特性启用状态和远程配置存储为[Datadog RUM特性标志](https://docs.datadoghq.com/real_user_monitoring/guide/setup-feature-flag-data-collection/?tab=npm#analyze-your-feature-flag-performance-in-rum)，用户特征存储为[Datadog用户会话属性](https://docs.datadoghq.com/real_user_monitoring/browser/modifying_data_and_context/?tab=npm#addoverride-user-session-property)。该集成需要已初始化的Datadog `datadogRum`客户端。

### 步骤1：使用feature_flags实验特性初始化Datadog RUM SDK

要开始收集特性标志数据，需初始化Datadog RUM SDK并在初始化参数enableExperimentalFeatures中配置["feature_flags"]。

```typescript
import { datadogRum } from '@datadog/browser-rum';

// Initialize Datadog Browser SDK
datadogRum.init({
    enableExperimentalFeatures: ["feature_flags"],
    ...
});
```

### 步骤2：通过配置初始化Flagsmith SDK

使用datadogRum选项初始化Flagsmith SDK。可选配置客户端将Flagsmith特征通过``datadogRum.setUser()``发送至Datadog。

```typescript
import { datadogRum } from '@datadog/browser-rum';
...
// Initialize the Flagsmith SDK
flagsmith.init({
    datadogRum: {
        client: datadogRum,
        trackTraits: true,
    },
    ...
})
```

### 步骤3：后续流程

- 当代码中_评估_标志值时，这些值会作为用户事件发送至Datadog
- 若启用发送特征选项，SDK获取标志时会将特征键值对发送至Datadog

该功能会以如下格式将远程配置和特性启用状态作为特性标志进行追踪

```bash
flagsmith_value_<FEATURE_NAME> // remote config
flagsmith_enabled_<FEATURE_NAME> // enabled state
```

此外，该集成还会以如下格式将Flagsmith用户特征存储至Datadog用户会话中：

```bash
flagsmith_trait_<FEATURE_NAME> // remote config
```

集成示例可参见
[此处](https://github.com/Flagsmith/flagsmith-js-client/blob/main/examples/datadog-realtime-user-monitoring/src/index.tsx)。

## Dynatrace JavaScript SDK集成

Flagsmith JavaScript SDK可配置为将功能开关状态、远程配置值及用户特征存储为Dynatrace会话属性。该集成要求预先配置好Dynatrace的`dtrum`对象。

### 步骤1：在`flagsmith.init()`中传入`enableDynatrace`

配置SDK时需传入已初始化的
[dtrum实例](https://www.dynatrace.com/support/help/how-to-use-dynatrace/real-user-monitoring/basic-concepts/js-tag-api)。

```javascript
// Initialize the Flagsmith SDK
flagsmith.init({
 //...Initialisation properties,
 enableDynatrace: true,
});
```

当`flagsmith.init`中设置`enableDynatrace`为true时，Flagsmith将通过
[dtrum.sendSessionProperties()](https://www.dynatrace.com/support/doc/javascriptapi/interfaces/dtrum_types.DtrumApi.html#sendSessionProperties)
发送以下会话属性：

- 功能开关状态以**shortString**类型发送，前缀为`flagsmith_enabled_`
  - 示例：`flagsmith_enabled_hero: "true"`
- 远程配置值以`flagsmith_value_`为前缀，数值类型为**javaDouble**，其他类型为**shortString**
  - 示例：`flagsmith_value_font_size: 21`, `flagsmith_value_hero_colour: "blue"`
- 用户特征以`flagsmith_trait_`为前缀，数值类型为**javaDouble**，其他类型为**shortString**
  - 示例：`flagsmith_trait_age: 21`, `flagsmith_trait_favourite_colour: "blue"`

### 步骤2：在Dynatrace应用设置中添加会话属性

[与其他Dynatrace会话属性类似](https://www.dynatrace.com/support/help/how-to-use-dynatrace/real-user-monitoring/setup-and-configuration/web-applications/additional-configuration/define-user-action-and-session-properties)，需在RUM应用设置中定义这些属性。

也可通过
[Dynatrace API](https://www.dynatrace.com/support/help/dynatrace-api/configuration-api/rum/mobile-custom-app-configuration/user-action-and-session-properties/post-property)
添加这些属性。

### Dynatrace配置截图

定义Dynatrace属性：

![图片](/img/dynatrace_1.png)

定义Flagsmith与Dynatrace属性：

![图片](/img/dynatrace_2.png)

按Flagsmith功能开关筛选：

![图片](/img/dynatrace_3.png)

## 常见问题

**如何同时调用`identify`、`setTraits`和`init`？**

- `init`应在应用中仅调用一次，建议在所有flagsmith操作前调用
- `init`默认会获取功能开关，可通过`preventFetch`参数禁用（适用于后续立即执行用户身份识别的情况）

**何时触发onChange回调？**

`onChange`在以下场景触发功能开关获取时调用：

- init初始化
- setTrait设置特征
- incrementTrait递增特征
- getFlags获取开关
- identify身份识别
- 通过本地存储评估开关

最佳实践是将`onChange`与应用程序的状态管理结合使用，例如通过`onChange`触发一个动作，利用`hasFeature`和`getValue`重新评估功能开关状态。

但如果这与您的开发模式不匹配，上述所有Flagsmith函数均会返回一个Promise，该Promise会在获取到最新功能开关状态时解析。

例如执行以下操作：

```javascript
await flagsmith.setTrait('age', 21);
const hasFeature = flagsmith.hasFeature('my_feature');
```

`onChange`回调时会携带变更信息，您可利用此机制避免不必要的重新渲染。

```javascript
onChange(this.oldFlags, {
 isFromServer: true, // flags have come from the server or local storage
 flagsChanged: deepEqualsCheck(oldFlags, newFlags),
 traitsChanged: deepEqualsCheck(oldFlags, newFlags),
});
```

**功能开关缓存如何工作？**

若在`init`时设置`cacheFlags`为true，SDK会将功能开关评估结果缓存至本地异步存储中。当浏览器重新加载时，会立即触发携带本地存储开关状态的`onChange`事件。具体流程如下：

1. 调用`init`

2. 若启用`cacheFlags`，则检查本地存储中是否存在缓存的开关和用户特征

3. 若发现本地存储中存在开关，则立即用存储值触发`onChange`

4. 同时会异步获取最新开关状态，获取成功后再次触发`onChange`回调

5. 每次获取到新开关状态时都会更新本地存储

默认情况下这些开关会永久保存，您可通过删除`localStorage`中的`"BULLET_TRAIN_DB"`来清除缓存。

**为何会出现`ReferenceError: XMLHttpRequest未定义`错误？**

Flagsmith JavaScript客户端使用[fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)处理REST调用。某些框架（如Manifest和Nuxt）默认不支持此API。

解决方案是为Flagsmith SDK提供自定义fetch实现，示例代码可参考[此处](https://github.com/Flagsmith/flagsmith-js-client/blob/main/examples/nuxt/plugins/flagsmith-plugin.ts#L9)。