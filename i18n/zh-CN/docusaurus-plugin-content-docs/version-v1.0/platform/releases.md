# 版本发布

## Docker 镜像标签

我们采用简单的标签策略：

- 提交到 `main` 分支会触发构建标签为 `latest` 的 Docker 镜像。
- Git 标签（例如 `2.7.1`）会触发构建以下标签的 Docker 镜像：
  - 2.7.1
  - 2.7

请查看我们的 [Github Releases](https://github.com/Flagsmith/flagsmith/releases) 页面获取详细更新信息。

## v2.23

发布于 **2022年5月17日**

- 安全审计修复
- 为请求头中未知环境返回403添加缓存
- 语义化版本和正则表达式验证

## v2.22

发布于 **2022年4月25日**

- 语义化版本运算符
- 权限改进
- 用户界面优化

## v2.21

发布于 **2022年4月12日**

- 停止支持 Python 3.6
- 变更请求功能

## v2.20

发布于 **2022年4月7日**

- 仪表板性能优化
- 分段身份采样

## v2.19

发布于 **2022年3月24日**

- 仪表板性能优化
- 用户界面微调
- ARM架构Docker镜像
- Dynatrace集成

## v2.18

发布于 **2022年2月15日**

- 开发服务端SDK密钥前端
- 改进分段规则评估
- 多项依赖更新

## v2.17

发布于 **2022年2月8日**

- Webhook分析集成
- 审计日志搜索改为服务端实现
- 标志值文本编辑器改进

## v2.16

发布于 **2022年1月31日**

- 分段覆盖权限升级
- 新增大量一键安装程序（fly.io、render、digital ocean）
- Rudderstack集成
- 为SDKv2准备的新SDK令牌

## v2.15

发布于 **2022年1月22日**

- 为InfluxDB添加主机名
- 改进SSL和`USE_X_FORWARDED_HOST`环境变量处理

## v2.14

发布于 **2022年1月4日**

- Slack集成
- 额外权限选项
- 端到端测试迁移至TestCafe

## v2.13

发布于 **2021年12月8日**

- 受保护标志
- 环境对比
- 强制双因素认证

## v2.12

发布于 **2021年12月2日**

- 环境变量重命名

## v2.11

发布于 **2021年11月18日**

- 改进Docker Compose配置
- Heap分析集成
- 多项依赖更新

## v2.10

发布于 **2021年11月2日**

- Elixir SDK
- 归档标志
- 指定标志所有者
- 数据库性能优化
- 权限改进

## v2.9

发布于 **2021年8月23日**

- 归档标志功能
- 可选GZIP响应
- 大量小错误修复

## v2.8

发布于 **2021年7月28日**

- SAML集成（仅限企业版）

## v2.7

发布于 **2021年5月5日**

- [多变量功能标志](https://flagsmith.com/blog/introducing-multivariate-flags/)
- 多项细微错误修复

## v2.6

发布于 **2021年4月9日**

- 合并功能标志与远程配置
- 深色模式
- 支持xml/json/toml/yaml格式的远程配置格式化工具
- 修复分段覆盖日期错误
- 为身份特征列表添加排序功能
- 基于FLAGSMITH_ANALYTICS环境变量启用SDK分析
- 可配置的黄油条提示
- 显示分段描述
- 新增集成支持：
  - Mixpanel
  - Heap Analytics
  - New Relic

重大变更：

- 分段覆盖使用新API

## v2.5

发布于 **2021年1月20日**

- 新增链接邀请功能
- 更多第三方集成

## v2.4

发布于 **2020年12月7日**

这是采用新品牌Flagsmith后的首个版本。

品牌重塑未引入破坏性变更，主要涉及URL和术语重构，同时我们自上个版本以来构建了多项新功能和错误修复：

- 若API使用influx，现以图表形式展示使用数据
  ![image](https://user-images.githubusercontent.com/8608314/101258233-16227880-3719-11eb-86c0-738c43d2ec0f.png)
- 全新视觉风格
- 支持批量启用/禁用标志的所有分段覆盖和用户覆盖
  ![image](https://user-images.githubusercontent.com/8608314/101258463-87aef680-371a-11eb-9e98-35c976612962.png)
- 用户页面合理分页
- 提升端到端测试稳定性
- 集成页面（该页面几乎完全由远程配置管理），支持通过您喜爱的工具增强Flagsmith。当前支持DataDog，即将支持Amplitude。
  ![image](https://user-images.githubusercontent.com/8608314/101258436-6ea64580-371a-11eb-8afe-3626eb36bbbe.png)

使用此功能的远程配置如下：

```json
integrations:

["data_dog"]
```

```json
integration_data
{
    "perEnvironment": false,
    "image": "https://xyz",
    "fields": [
      {
        "key": "base_url",
        "label": "Base URL"
      },
      {
        "key": "api_key",
        "label": "API Key"
      }
    ],
    "tags": [
      "logging"
    ],
    "title": "Data dog",
    "description": "Sends events to Data dog for when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production."
}
```

## v2.3

发布于 **2020年11月9日**

我们新增了大量功能和错误修复。

- 现可为标志添加用户自定义标签，用于管理和组织标志
- [Data Dog](https://docs.flagsmith.com/integrations/datadog/)和[Amplitude](https://docs.flagsmith.com/integrations/amplitude/)集成测试版发布
- 支持单次调用设置多个特征
- 显示特定功能被哪些身份单独覆盖
- 查看身份时显示分段并测试该身份是否属于各分段成员

## v2.2

发布于 **2020年6月12日**

- 具有分段覆盖的功能标志现基于环境层级（而非项目层级），可跨环境差异化定义分段覆盖
- 重新设计左侧导航区
- 支持按标志筛选审计日志
- 可从功能主页面跳转至标志审计日志

## v2.1

发布于 **2020年5月28日**

- Google OAuth2 集成
- 双重身份验证
- 用于API调用统计的Influx DB集成
- API现在可以在单个请求中设置多个特征
- 修复了某些场景下Webhook可能丢失数据的问题
- 主要是CSS设计调整

## v2.0

发布于 **2019年12月19日**

- 从前端应用中移除了Flagsmith主页
- 新增Webhooks功能

## v1.9

发布于 **2019年9月22日**

- 新增用户分群功能
- 新增审计日志
- 改进端到端测试