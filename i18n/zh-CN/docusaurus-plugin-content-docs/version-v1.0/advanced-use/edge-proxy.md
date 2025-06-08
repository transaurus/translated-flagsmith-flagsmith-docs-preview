# Edge 代理

:::tip

Edge 代理仅作为企业版计划的一部分提供。如果您对此感兴趣，请[联系我们](https://flagsmith.com/contact-us/)！

:::

Flagsmith Edge 代理允许您在靠近服务器的位置运行 Flagsmith 引擎实例。如果您在服务器端环境中运行 Flagsmith，并且需要极低延迟的响应时间，Edge 代理正是您所需的解决方案！

Edge 代理以轻量级 Docker 容器形式运行。它连接到核心 Flagsmith API（由我们托管的 api.flagsmith.com 或您自行托管的实例）以获取环境标志和分段规则。随后，您可以将 Flagsmith SDK 指向您的 Edge 代理；它实现了所有当前 SDK 端点。这意味着您可以在靠近基础设施和用户的位置以极低延迟处理大量请求。请参阅下方的[架构说明](#architecture)。

该代理还充当本地缓存，允许您在不触及核心 API 的情况下向代理发起请求。

## 性能表现

在单台 4 核虚拟机上，Edge 代理当前可处理约每秒 2,000 次请求，平均延迟约 7 毫秒。由于采用无状态设计，在负载均衡器后部署时几乎具备完美的可扩展性。

## 运行 Edge 代理

Edge 代理以 Docker 容器形式运行，当前可在 [Docker Hub](https://hub.docker.com/repository/docker/flagsmith/edge-proxy) 获取。

:::tip

`flagsmith/edge-proxy` 镜像是私有镜像——如需访问权限，请[联系我们](https://flagsmith.com/contact-us/)。

:::

```bash
# Download the Docker Image
docker pull flagsmith/edge-proxy

# Run it
docker run \
    -e FLAGSMITH_API_URL='https://api.flagsmith.com/api/v1/' \
    -e FLAGSMITH_API_TOKEN='<API Token - see below>' \
    -e ENVIRONMENT_API_KEYS='<Flagsmith Environment Key - see below>' \
    -p 8000:8000 \
    flagsmith/edge-proxy:latest
```

代理现已运行并在 8000 端口提供服务。

### 环境变量

您可通过以下环境变量配置 Edge 代理：

- `FLAGSMITH_API_URL` **必填**。被代理的 API 地址，例如 `https://api.flagsmith.com/api/v1/`，若您自行托管 Flagsmith API 则填写您的域名。
- `ENVIRONMENT_API_KEYS` **必填**。要服务的环境密钥列表。以逗号分隔的字符串形式提供，例如 `4vfqhypYjcPoGGu8ByrBaj, EmJFz265Q6CAXuGRZYnkeE, "8KzETdDeMY7xkqkSkY3Gsg`
- `FLAGSMITH_API_TOKEN` **必填**。可按[此说明](/clients/rest.md##private-endpoints)获取。
- `API_POLL_FREQUENCY` 以秒为单位设置。轮询 `FLAGSMITH_API_URL` 以更新当前使用环境前的等待秒数。默认为 `10` 秒。
- `WEB_CONCURRENCY` [Uvicorn](https://www.uvicorn.org/) 工作进程数。默认为 `1`。建议设置为可用 CPU 核心数。

## 使用 Edge 代理

Edge 代理提供与核心 API 完全相同的 API 方法集。这意味着您只需将 SDK 指向 Edge 代理域名即可立即使用。例如，假设您按照上述说明在本地运行代理，只需执行以下操作：

```bash
curl "http://localhost:8000/api/v1/flags" -H "x-environment-key: 95DybY5oJoRNhxPZYLrxk4" | jq

{
  "flags": [
    {
      "id": 78978,
      "feature": {
        "id": 15058,
        "name": "string_feature",
        "created_date": "2021-11-29T17:15:51.694223Z",
        "description": null,
        "initial_value": "foo",
        "default_enabled": true,
        "type": "STANDARD"
      },
      "feature_state_value": "foo",
      "enabled": true,
      "environment": 12561,
      "identity": null,
      "feature_segment": null
    },
    {
      "id": 78980,
      "feature": {
        "id": 15059,
        "name": "integer_feature",
        "created_date": "2021-11-29T17:16:11.288134Z",
        "description": null,
        "initial_value": "1234",
        "default_enabled": true,
        "type": "STANDARD"
      },
      "feature_state_value": 1234,
      "enabled": true,
      "environment": 12561,
      "identity": null,
      "feature_segment": null
    }
  ]
}
```

## 管理特征值

Edge 代理存在一个注意事项：由于完全无状态设计，它无法将特征数据持久化到任何存储中。这意味着在请求特定身份的标志时，您必须提供完整的特征值集合。我们的 SDK 均提供了实现此功能的相关方法。使用 `curl` 的示例如下：

```bash
curl -X "POST" "http://localhost:8000/api/v1/identities/?identifier=do_it_all_in_one_go_identity" \
     -H 'X-Environment-Key: n9fbf9h3v4fFgH3U3ngWhb' \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "traits": [
    {
      "trait_value": 123.5,
      "trait_key": "my_trait_key"
    },
    {
      "trait_value": true,
      "trait_key": "my_other_key"
    }
  ],
  "identifier": "do_it_all_in_one_go_identity"
}'
```

请注意，当前 Edge 代理不会将特征数据回传至核心 API。

## 架构

标准Flagsmith架构非常简单：

![Image](/img/edge-proxy-existing.png)

加入代理后，架构依然简洁，只是延迟更低！

![Image](/img/edge-proxy-proxy.png)