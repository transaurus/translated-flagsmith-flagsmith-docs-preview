# 边缘代理

Flagsmith边缘代理允许您在靠近服务器的位置运行Flagsmith引擎实例。如果您在服务器端环境中使用Flagsmith，并且需要极低延迟的响应时间，边缘代理正是您所需的解决方案！

边缘代理作为轻量级Docker容器运行。它会连接到Flagsmith API（由我们提供的api.flagsmith.com或您自行托管的实例）以获取环境特征标志和分段规则。随后您可以将Flagsmith SDK指向您的边缘代理；该代理实现了当前所有SDK端点。这意味着您能以极低延迟在基础设施和用户附近处理大量请求。查看下方的[架构说明](#architecture)。

该代理还充当本地缓存，使您无需访问核心API即可向代理发起请求。

## 性能表现

边缘代理当前在4核虚拟机上可处理约2,000次请求/秒，平均延迟约7毫秒。由于采用无状态设计，通过负载均衡器部署后几乎可实现完美扩展。

## 配置说明

可通过JSON配置文件（此处命名为`config.json`）配置边缘代理。

您可以在`config.json`中设置以下配置项来控制边缘代理行为：

- **environment_key_pairs**: 环境密钥对对象数组，例如：
  `"environment_key_pairs":[{"server_side_key":"您的服务端密钥", "client_side_key":"您的客户端环境密钥"}]`
- **[可选] api_poll_frequency(秒)**: 控制边缘代理向服务器轮询变更的频率，例如：`"api_poll_frequency":10`
- **[可选] api_url**: 若运行自托管版Flagsmith，可在此设置自托管URL以便边缘代理连接您的服务器，例如：`"api_url":"https://自托管.flagsmith.域名/api/v1"`

完成上述配置后，`config.json`文件内容示例如下：

```json
{
 "environment_key_pairs": [
  {
   "server_side_key": "your_server_side_environment_key",
   "client_side_key": "your_client_side_environment_key"
  }
 ],
 "api_poll_frequency": 10,
 "api_url": "https://api.flagsmith.com/api/v1"
}
```

### 环境变量

可通过以下环境变量配置边缘代理：

- `WEB_CONCURRENCY` [Uvicorn](https://www.uvicorn.org/)工作进程数。默认为`1`。建议设置为可用CPU核心数。

## 运行边缘代理

边缘代理作为Docker容器运行，当前已发布于[Docker Hub](https://hub.docker.com/repository/docker/flagsmith/edge-proxy)。

### 使用docker run运行

```bash
# Download the Docker Image
docker pull flagsmith/edge-proxy

# Run it
docker run \
    -v /<path-to-local>/config.json:/app/config.json \
    -p 8000:8000 \
    flagsmith/edge-proxy:latest
```

### 使用docker compose运行

```yml
version: '3.9'
services:
 edge_proxy:
  image: flagsmith/edge-proxy:latest
  volumes:
   - type: bind
     source: ./config.json
     target: /app/config.json
  ports:
   - target: 8000
   - published: 8000
```

代理现已运行并在8000端口提供服务。

## 接入边缘代理

边缘代理提供与核心API完全相同的接口方法。这意味着您只需将SDK指向边缘代理域名即可立即使用。例如按照上述说明在本地运行代理后，您可以这样操作：

```bash
curl "http://localhost:8000/api/v1/flags" -H "x-environment-key: 95DybY5oJoRNhxPZYLrxk4" | jq

[
    {
        "enabled": true,
        "feature_state_value": 5454,
        "feature": {
            "name": "feature_1",
            "id": 2,
            "type": "MULTIVARIATE"
        }
    },
    {
        "enabled": true,
        "feature_state_value": "some_value",
        "feature": {
            "name": "feature_2",
            "id": 9,
            "type": "STANDARD"
        }
    },
]
```

## 监控

边缘代理提供2个健康检查端点。

### SDK代理健康检查

向`/proxy/health`发起请求时，代理将返回HTTP `200`及`{"status": "ok"}`响应。您可将编排系统的健康检查指向此端点。该端点会验证环境文档未过期，且代理正在处理SDK请求。

### 实时标志/服务器推送事件健康检查

如果您使用代理来支持服务器发送事件(Server Sent Events)以实现实时功能标志更新，当向`/sse/health`发起请求时，代理将返回HTTP状态码`200`及响应体`{"status": "ok"}`。

## 特征管理

边缘代理存在一个注意事项：由于完全无状态设计，它无法将特征(Trait)数据持久化存储到任何数据库中。这意味着在请求特定身份(Identity)的功能标志时，必须提供完整的特征集合。我们的所有SDK都提供了相关方法来实现这一点，例如使用`curl`的示例如下：

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

请注意，当前边缘代理不会将特征数据回传至核心API。

## 架构设计

标准Flagsmith架构非常简单：

![Image](/img/edge-proxy-existing.png)

加入代理组件后，架构依然保持简洁，但显著降低了延迟：

![Image](/img/edge-proxy-proxy.png)