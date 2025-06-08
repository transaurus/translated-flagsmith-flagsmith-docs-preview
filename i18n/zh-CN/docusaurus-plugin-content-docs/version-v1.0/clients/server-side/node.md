---
title: Flagsmith NodeJS SDK
sidebar_label: NodeJS
description: Manage your Feature Flags and Remote Config in your NodeJS applications.
slug: /clients/node
---

该库可用于服务端NodeJS项目。客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-nodejs-client)。

## 入门指南

以下说明将帮助您在本地开发环境获取项目副本并运行，用于开发和测试目的。生产环境部署注意事项请参阅运行指南。

## 安装说明

通过npm安装：

`npm i flagsmith-nodejs --save`

## 使用说明

### 获取项目功能开关

```javascript
const flagsmith = require('flagsmith-nodejs');

flagsmith.init({
 environmentID: '<YOUR_ENVIRONMENT_KEY>',
});

flagsmith.hasFeature('header', '<My User Id>').then((featureEnabled) => {
 if (featureEnabled) {
  //Show my awesome cool new feature to this one user
 }
});

flagsmith.hasFeature('header').then((featureEnabled) => {
 if (featureEnabled) {
  //Show my awesome cool new feature to the world
 }
});

flagsmith.getValue('header', '<My User Id').then((value) => {
 //Show some unique value to this user
});

flagsmith.getValue('header').then((value) => {
 //Show a value to the world
});
```

## 可用配置项

| Property        |                                            Description                                            | Required |                      Default Value |
| --------------- | :-----------------------------------------------------------------------------------------------: | -------: | ---------------------------------: |
| `environmentID` |  Defines which project environment you wish to get flags for. _example ACME Project - Staging._   |  **YES** |                               null |
| `onError`       |                 Callback function on failure to retrieve flags. `(error)=>{...}`                  |   **NO** |                               null |
| `api`           | Use this property to define where you're getting feature flags from, e.g. if you're self hosting. |   **NO** | <https://api.flagsmith.com/api/v1> |
| `cache`         |  Defines an object containing 3 functions (`has(k)`, `get(k)`, `set(k,v)`) to cache API results   |   **NO** |                               null |

## 可用函数

| Property                       |                                                  Description                                                   |
| ------------------------------ | :------------------------------------------------------------------------------------------------------------: |
| `init`                         |                              Initialise the sdk against a particular environment                               |
| `hasFeature(key)`              |         Get the value of a particular feature e.g. `flagsmith.hasFeature("powerUserFeature") // true`          |
| `hasFeature(key, userId)`      | Get the value of a particular feature for a user e.g. `flagsmith.hasFeature("powerUserFeature", 1234) // true` |
| `getValue(key)`                |               Get the value of a particular feature e.g. `flagsmith.getValue("font_size") // 10`               |
| `getValue(key, userId)`        | Get the value of a particular feature for a specificed user e.g. `flagsmith.getValue("font_size", 1234) // 15` |
| `getFlags()`                   |                               Trigger a manual fetch of the environment features                               |
| `getFlagsForUser(userId)`      |                     Trigger a manual fetch of the environment features for a given user id                     |
| `getUserIdentity(userId)`      |         Trigger a manual fetch of both the environment features and users' traits for a given user id          |
| `getTrait(userId, key)`        |                         Trigger a manual fetch of a specific trait for a given user id                         |
| `setTrait(userId, key, value)` |                                    Set a specific trait for a given user id                                    |

## 用户标识

用户标识功能允许您通过[Flagsmith控制台](https://www.flagsmith.com/)定位特定用户。您可以在调用`hasFeature`和`getValue`方法时传入可选的用户标识参数，以获取相应用户的功能开关和变量配置。

## 数据缓存

可通过如下方式初始化SDK：

```javascript
flagsmith.init({
 cache: {
   has:(key)=> return Promise.resolve(!!cache[key]) , // true | false
   get: (k)=> cache[k] // return flags or flags for user
   set: (k,v)=> cache[k] = v // gets called if has returns false with response from API for Identify or getFlags
  }
})
```

核心机制是当`has`返回false时，SDK会自动执行必要的API调用。缓存键名格式为`flags`或`flags_traits-${identity}`。

具体实现示例如下。

```javascript
const flagsmith = require('flagsmith-nodejs');
const redis = require('redis');

const redisClient = redis.createClient({
 host: 'localhost',
 port: 6379,
});

flagsmith.init({
 environmentID: '<Flagsmith Environment API Key>',
 cache: {
  has: (key) =>
   new Promise((resolve, reject) => {
    redisClient.exists(key, (err, reply) => {
     console.log('check ' + key + ' from cache', err, reply);
     resolve(reply === 1);
    });
   }),
  get: (key) =>
   new Promise((resolve) => {
    redisClient.get(key, (err, cacheValue) => {
     console.log('get ' + key + ' from cache');
     resolve(cacheValue && JSON.parse(cacheValue));
    });
   }),
  set: (key, value) =>
   new Promise((resolve) => {
    // Expire the key after 60 seconds
    redisClient.set(key, JSON.stringify(value), 'EX', 60, (err, reply) => {
     console.log('set ' + key + ' to cache', err);
     resolve();
    });
   }),
 },
});

router.get('/', function (req, res, next) {
 flagsmith.getValue('background_colour').then((value) => {
  res.render('index', {
   title: value,
  });
 });
});
```