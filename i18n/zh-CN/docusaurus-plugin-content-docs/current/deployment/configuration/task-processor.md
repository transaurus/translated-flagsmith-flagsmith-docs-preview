# 异步任务处理器

Flagsmith 具备通过独立任务处理器服务消费异步任务的能力。若未启用异步处理器，Flagsmith API 将在非托管的独立线程中运行异步任务。

## 运行处理器

任务处理器可通过 flagsmith/flagsmith-api 镜像运行，但需使用稍有不同的入口点。其应指向与 API 容器相同的数据库。为使 API 能将任务发送至处理器，必须在运行 Flagsmith 应用的容器中设置环境变量 `TASK_RUN_METHOD` 为 `TASK_PROCESSOR`。

基础的 docker-compose 配置示例如下：

```yaml
    postgres:
        image: postgres:11.12-alpine
        environment:
            POSTGRES_PASSWORD: password
            POSTGRES_DB: flagsmith
        container_name: flagsmith_postgres

    flagsmith:
        image: flagsmith/flagsmith-api
        environment:
            DJANGO_ALLOWED_HOSTS: '*'
            DATABASE_URL: postgresql://postgres:password@postgres:5432/flagsmith
            ENV: prod
            TASK_RUN_METHOD: TASK_PROCESSOR
        ports:
            - '8000:8000'
        depends_on:
            - postgres
        links:
            - postgres

   flagsmith_processor:
       build:
           dockerfile: api/Dockerfile
           context: .
       environment:
           DATABASE_URL: postgresql://postgres:password@postgres:5432/flagsmith
       command:
           - "python"
           - "manage.py"
           - "runprocessor"
       depends_on:
           - flagsmith
           - postgres
```

## 配置处理器

该处理器提供多项配置选项，可根据需求或环境进行调优。这些配置选项通过启动处理器时的命令行参数传递。

| Argument            | Description                                                               | Default |
| ------------------- | ------------------------------------------------------------------------- | ------- |
| `--sleepintervalms` | The amount of ms each worker should sleep between checking for a new task | 2000    |
| `--numthreads`      | The number of worker threads to run per task processor instance           | 5       |
| `--graceperiodms`   | The amount of ms before a worker thread is considered 'stuck'.            | 3000    |

## 监控

可通过多种方式监控任务处理器的运行状态。

### 检查线程/工作进程健康状态

任务处理器包含一个管理命令，用于检查运行任务的工作线程健康状况。

```
python manage.py checktaskprocessorthreadhealth
```

该命令将以成功退出码（0）或失败退出码（1）终止。

### API 至任务处理器的健康检查

为监控 API 能否向处理器发送任务及任务是否成功执行，可在通用健康端点（`GET /health?format=json`）启用自定义健康检查。需手动启用此功能，通过在 Flagsmith 应用容器（非任务处理器容器）中设置环境变量 `ENABLE_TASK_PROCESSOR_HEALTH_CHECK` 为 `True`。注意此健康检查不视为"关键"检查，因此无论任务处理器是否成功处理任务，端点均会返回 200 OK 状态。

### 任务统计

API 内提供返回处理器已消费/待消费任务统计数据的 JSON 格式端点，访问路径为 `GET /processor/monitoring`。该端点返回以下数据结构：

```json
{
 "waiting": 1
}
```