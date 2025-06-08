# Edge API

:::info

Edge API仅在我们的SaaS平台中提供，不属于开源项目的一部分。

:::

[Flagsmith架构](/guides-and-examples/integration-approaches#flags-are-evaluated-server-side)基于服务端标志计算引擎。这带来诸多优势，但也可能增加延迟——尤其是当调用来自距离我们当前API所在地（欧盟地区）较远的位置时。同时，这也构成了单一故障点，若遭遇AWS区域级中断将影响服务可用性。

Edge API可同时解决这两个问题。它提供跨8个AWS区域复制的数据存储和边缘计算API，支持基于延迟的路由选择，并在区域中断时实现全局故障转移。

Edge API服务覆盖以下AWS区域：

- 欧洲（伦敦） - eu-west-2
- 美国东部（俄亥俄） - us-east-2
- 美国西部（北加利福尼亚） - us-west-1
- 亚太（孟买） - ap-south-1
- 亚太（悉尼） - ap-southeast-2
- 南美（圣保罗） - sa-east-1
- 亚太（首尔） - ap-northeast-2
- 亚太（新加坡） - ap-southeast-1

## 启用Edge API

:::tip

在**2022年6月7日**之前创建的现有组织，其项目默认部署在核心API上。您可通过访问项目设置页面点击"开始迁移"按钮，将项目迁移至Edge API。迁移耗时将根据项目包含的身份数据量在1分钟至1小时之间不等。

迁移期间及迁移完成后，核心API仍可正常运作。详见[迁移步骤](#migration-steps)获取更多信息。

:::

项目完成Edge迁移后，您只需将SDK指向新的Flagsmith Edge API URL：`edge.api.flagsmith.com`。该域名指向我们的边缘CDN，仅此而已！

最简便的方式是升级至您所用编程语言的最新版Flagsmith SDK。

若无法升级，您可手动修改现有SDK配置指向Edge API端点。例如在Java SDK中，我们只需添加`withApiUrl`参数：

```java
FlagsmithClient flagsmithClient = FlagsmithClient.newBuilder()
        .setApiKey("aaa"))
        .withApiUrl("https://edge.api.flagsmith.com/api/v1/")
        .build();
```

请注意Edge API URL为：`https://edge.api.flagsmith.com/api/v1/`。

请查阅对应语言SDK文档了解如何重写端点URL前缀。

## 迁移步骤

迁移过程会将您的`Identity`数据从核心API单向同步至Edge API。所有身份数据仍会保留在核心API中，您也可以选择继续向核心API写入身份数据。

最终目标是让所有应用程序接入Edge API，从而享受全球低延迟优势，并获得多区域故障转移和容错能力。

### 步骤1 - 准备应用程序

将应用程序配置为指向Flagsmith Edge API，即将`api.flagsmith.com`更改为`edge.api.flagsmith.com`。您可以选择在SDK中显式设置，或直接确保使用最新版SDK——默认情况下最新版SDK会自动指向`edge.api.flagsmith.com`。

### 步骤2 - 迁移数据

您现在可以通过Flagsmith仪表板为每个项目触发单向数据同步。只需进入项目设置页面点击"开始迁移"按钮，即可将项目迁移至Edge API。该操作将启动一个耗时1分钟至1小时的作业（具体时长取决于数据量）。迁移完成后，核心API中的所有身份数据都将同步至Edge API。

迁移过程中及迁移完成后，核心API仍可正常运作。

若您拥有类似移动应用的产品（与网页应用不同），无法立即强制所有用户升级，在此期间很可能仍会向旧版核心API写入身份数据。

在迁移期间及迁移完成后，若系统收到针对核心API身份端点的写入请求，我们将在核心API保存数据的同时将该请求重放至Edge API。您可以逐步将API端点/SDK更新至Edge API，为用户迁移至新版本应用争取时间。

请注意：未来仍可向核心API写入数据，但这些数据不会在核心平台与边缘平台之间同步。

### 步骤3 - 部署应用程序

当数据完成复制至Edge API数据存储后，您即可部署指向新端点`edge.api.flagsmith.com`的应用程序，享受全球低延迟优势！

## 注意事项

### 部分安全身份端点已变更

若您使用REST API管理身份数据，请注意部分端点已更新。如需帮助请联系我们。

### 新增身份覆盖仅适用于Edge API

迁移完成后，针对特定身份设置的任何新标记值覆盖将仅作用于Edge API。

### 增量/减量端点已弃用

不过您可能原本就不知道这些端点存在吧？

### 批量特征端点已弃用

您仍可通过POST /identities端点实现相同功能。

### API响应已精简

我们移除了核心API响应中SDK未使用的冗余字段。虽然不影响SDK运作，但若您直接使用REST API获取这些值，请注意以下字段已被移除：

```txt
trait.id
flag.feature.created_data
flag.feature.description
flag.feature.initial_value
flag.feature.default_enabled
flag.environment
flag.identity
flag.feature_segment
```

## 架构设计

### 仅核心API模式

![Image](/img/core-api-now.svg)

### 核心+边缘API模式

![Image](/img/edge-api-now.svg)

## 工作原理

### Lambda@Edge

我们的核心[规则引擎](https://github.com/Flagsmith/flagsmith-engine)已从REST API解耦，使其既能作为Flagsmith API的依赖项，也可用于处理SDK API调用的Lambda函数集。将SDK客户端指向我们的全球CDN节点`edge.api.flagsmith.com`后，请求将由距离客户端最近的AWS数据中心内的Lambda函数处理，从而实现低延迟。

### DynamoDB全局表

我们的API中存储了两种状态数据：与您项目环境相关的数据，以及这些环境内身份(Identities)的数据。边缘架构设计采用DynamoDB全局表进行数据写入，这些表格会在全球范围内同步复制。我们的Lambda函数随后会连接最近的DynamoDB表，以获取环境数据和身份数据。