

采集的数据



1. 时间
2. 该微服务容器数量
3. 该微服务的cpu利用率平均值
4. 该微服务的内存利用率平均值
5. 该微服务流入流量
6. 该微服务流出流量
7. 该微服务延迟时间（95%分位）
8. 该微服务现在访问量
9. 该微服务预测访问量



要在 Docker Swarm 中实现类似的自动扩缩容功能，可以使用 Docker 的内置服务和任务管理功能。以下是一个基本的代码示例，展示了如何通过 Python 脚本来实现 Docker Swarm 的服务扩缩容：

### 前提条件
1. 已安装并配置好 Docker 和 Docker Swarm。
2. 已创建好所需的 Docker 服务（例如 Redis 和 Online Boutique 服务）。
3. 使用 Prometheus 监控服务的指标。

### Docker Swarm 自动扩缩容示例

以下是一个 Python 脚本，它会定期从 Prometheus 获取指标数据，并根据这些数据调整 Docker 服务的副本数。

#### 导入必要的库
```python
import docker
import requests
import time
import random

# Prometheus配置
PROMETHEUS_URL = 'http://localhost:9090/'

# Docker客户端配置
client = docker.from_env()

# 服务名称和相应的指标查询
SERVICES = {
    'redis_leader': {
        'query_cpu': 'sum(irate(container_cpu_usage_seconds_total{container_label_com_docker_swarm_service_name="redis_leader"}[5m]))',
        'query_mem': 'sum(container_memory_usage_bytes{container_label_com_docker_swarm_service_name="redis_leader"})',
        'min_replicas': 1,
        'max_replicas': 5
    },
    # 添加其他服务配置
}

# 获取Prometheus指标数据
def fetch_prometheus(query):
    try:
        response = requests.get(f"{PROMETHEUS_URL}/api/v1/query", params={'query': query})
        result = response.json()['data']['result']
        return float(result[0]['value'][1]) if result else 0
    except Exception as e:
        print(f"Error fetching data from Prometheus: {e}")
        return 0

# 自动扩缩容函数
def autoscale_service(service_name, cpu_threshold, mem_threshold):
    service = client.services.get(service_name)
    query_cpu = SERVICES[service_name]['query_cpu']
    query_mem = SERVICES[service_name]['query_mem']
    
    current_cpu_usage = fetch_prometheus(query_cpu)
    current_mem_usage = fetch_prometheus(query_mem)
    
    # 假设我们根据CPU和内存的平均利用率来调整副本数
    desired_replicas = max(
        SERVICES[service_name]['min_replicas'], 
        min(
            SERVICES[service_name]['max_replicas'], 
            int((current_cpu_usage + current_mem_usage) / 2)
        )
    )
    
    current_replicas = len(service.tasks(filters={'desired-state': 'running'}))
    
    if desired_replicas != current_replicas:
        print(f"Scaling {service_name} from {current_replicas} to {desired_replicas}")
        service.update(mode={'Replicated': {'Replicas': desired_replicas}})

# 主函数
def main():
    while True:
        for service_name in SERVICES.keys():
            autoscale_service(service_name, cpu_threshold=0.7, mem_threshold=0.7)
        time.sleep(60)  # 每分钟检查一次

if __name__ == "__main__":
    main()
```

### 代码解释
1. **导入必要的库**：需要安装 `docker` 和 `requests` 库。
2. **配置 Prometheus 和 Docker 客户端**：使用 Prometheus 监控指标，并使用 Docker 客户端来与 Docker Swarm 交互。
3. **定义服务和指标查询**：在 `SERVICES` 字典中定义每个服务的指标查询和最小/最大副本数。
4. **获取 Prometheus 指标数据**：通过 `fetch_prometheus` 函数从 Prometheus 获取指定指标的数据。
5. **自动扩缩容函数**：`autoscale_service` 函数根据当前的 CPU 和内存使用情况计算所需的副本数，并更新 Docker 服务的副本数。
6. **主函数**：在 `main` 函数中循环检查和调整服务的副本数。

### 运行脚本
1. 确保 Docker Swarm 集群已启动并且 Prometheus 在运行。
2. 根据需要调整服务配置和阈值。
3. 运行 Python 脚本：`python autoscale_swarm.py`

这个示例代码展示了一个基本的实现方式，实际生产环境中可能需要更多的错误处理、日志记录和优化。你可以根据实际需求进行调整和扩展。



# 用户访问模拟

Gatling 



简化版的是只针对一个业务流程 order->storage->account 部署一个预测模型，对我自己模拟出来的访问进行训练并预测，预测的值和容器的状态例如（cpu利用率，内存利用率，数量等）然后给强化学习模型进行决策容器弹性伸缩

order:192.168.88.129:3001/order/create?userId=1&productId=1&count=10&money=100

account:192.168.88.129:3002/







```

```

