# 贡献指南

我们始终致力于改进本项目！只要符合以下准则，欢迎参与开源贡献。

## 拉取请求

Flagsmith团队会持续关注拉取请求。收到PR后，团队成员将根据内部用例测试代码变更并签署确认。我们会直接合并PR或提供改进建议。

### 注意事项

- 若涉及API变更，请同步更新文档
- 保持代码风格统一（缩进、换行等）
- 若提交包含多次commit，建议使用`git rebase -i`压缩提交记录以便审查
- 单行代码不超过80个字符

## 预提交规范

项目采用预提交配置（`.pre-commit-config.yaml`），在提交前自动执行`black`、`flake8`和`isort`代码格式化。

安装预提交钩子：

```bash
# From the repository root
pip install pre-commit
pre-commit install
```

也可手动全量执行代码检查：

```bash
pre-commit run --all-files
```

## 运行测试

项目使用pytest框架编写（合理运用fixture）和运行测试。执行测试前请确保`DJANGO_SETTINGS_MODULE`环境变量指向正确模块，例如`app.settings.test`。

运行测试命令：

```bash
DJANGO_SETTINGS_MODULE=app.settings.test pytest
```

## 添加依赖

添加Python依赖步骤：

- 在`requirements.in`或`requirements-dev.in`中声明依赖
- 执行`pip-compile`或`pip-compile requirements-dev.in`