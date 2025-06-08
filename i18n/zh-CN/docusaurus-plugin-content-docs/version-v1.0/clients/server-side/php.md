---
title: Flagsmith PHP SDK
sidebar_label: PHP
description: Manage your Feature Flags and Remote Config in your PHP applications.
slug: /clients/php
---

该SDK客户端适用于PHP[https://flagsmith.com/](https://www.flagsmith.com/)。Flagsmith使您能够跨多个项目、环境和组织管理功能开关与远程配置。

客户端源代码托管于[Github](https://github.com/flagsmith/flagsmith-php-client)。

## 安装指南

`composer require flagsmith/flagsmith-php-client`

要求PHP 7.4或更高版本，并内置GuzzleHTTP支持。

您可选择自行实现PSR-18和PSR-16规范

> 您还需要安装[PSR-18](https://packagist.org/providers/psr/http-client-implementation)和
> [PSR-17](https://packagist.org/providers/psr/http-factory-implementation)的实现，例如
> [Guzzle](https://packagist.org/packages/guzzlehttp/guzzle)以及
> [PSR-16](https://packagist.org/providers/psr/simple-cache-implementation)实现，例如
> [Symfony缓存组件](https://packagist.org/packages/symfony/cache)。示例：

`composer require flagsmith/flagsmith-php-client guzzlehttp/guzzle symfony/cache`

或

`composer require flagsmith/flagsmith-php-client symfony/http-client nyholm/psr7 symfony/cache`

## 使用说明

Flagsmith PHP客户端采用不可变设计模式，每次修改设置时都会返回自身的克隆实例。

```php
$flagsmith = new Flagsmith('apiToken');
$flagsmithWithCache = $flagsmith->withCache(/** PSR-16 Cache Interface  **/);
```

若您自行托管Flagsmith实例，可通过Flagsmith类的第二个参数指定主机地址（需包含完整路径）

```php
$flagsmith = new Flagsmith('apiToken', 'https://api.flagsmith.com/api/v1/');
```

### 缓存应用

```php
$flagsmith = new Flagsmith('apiToken');
$flagsmithWithCache = $flagsmith
  ->withCache(/** PSR-16 Cache Interface  **/)
  ->withTimeToLive(15); //15 seconds
```

### 获取所有功能开关

获取全部功能开关，返回结果为`Flagsmith\Models\Flag`模型对象

#### 全局获取

```php
$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->getFlags();
```

#### 按用户标识获取

```php
$identity = new \Flagsmith\Models\Identity('identity');

$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->getFlagsByIdentity($identity);
```

### 获取单个功能开关

单个功能开关将返回为`Flagsmith\Models\Flag`模型对象

#### 全局获取单个开关

```php
$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->getFlag('name');
```

#### 按用户标识获取单个开关

```php
$identity = new \Flagsmith\Models\Identity('identity');

$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->getFlagByIdentity($identity, 'name');
```

### 检查功能是否启用

验证特定功能是否处于启用状态

#### 全局检查功能状态

```php
$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->isFeatureEnabled('name');
```

#### 按用户标识检查功能状态

```php
$identity = new \Flagsmith\Models\Identity('identity');

$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->isFeatureEnabledByIdentity($identity, 'name');
```

### 获取功能值

获取特定功能的配置值

#### 全局获取功能值

```php
$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->getFeatureValue('name', 'default value');
```

#### 按用户标识获取功能值

```php
$identity = new \Flagsmith\Models\Identity('identity');

$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->getFeatureValueByIdentity($identity, 'name', 'default value');
```

### 用户特征应用

您可选择在用户标识模型上声明特征属性

```php
$identity = new \Flagsmith\Models\Identity('identity');

$identityTrait = (new \Flagsmith\Models\IdentityTrait('Foo'))->withValue('Bar');

$identity = $identity->withTrait($identityTrait);

$flagsmith = new \Flagsmith\Flagsmith('apiToken');
$flagsmith->getFlagsByIdentity($identity);
```