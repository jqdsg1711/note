是的，你可以将 Prometheus 部署到其中一台服务器（例如 `docker01`），然后配置 Prometheus 来监控 `docker01` 和 `docker02` 上的 Docker 容器。以下是具体的步骤：

### 1. 在 `docker01` 上部署 Prometheus

#### 步骤：
1. **创建配置文件**：
   在 `docker01` 上创建一个 `prometheus.yml` 文件，配置文件如下：
   ```yaml
   global:
     scrape_interval: 15s
   
   scrape_configs:
     - job_name: 'docker01-containers'
       static_configs:
         - targets: ['docker01:9323']  # cadvisor on docker01
     - job_name: 'docker02-containers'
       static_configs:
         - targets: ['192.168.88.128:9323']  # cadvisor on docker02
     - job_name: 'docker03-containers'
       static_configs:
         - targets: ['192.168.88.127:9323']  # cadvisor on docker03
   ```

2. **启动 Prometheus 容器**：
   
   ```sh
   docker run -d \
     -p 9090:9090 \
     -v /root/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
     --name prometheus \
     prom/prometheus
   
   ```

### 2. 在 `docker01` 和 `docker02` 上部署 cAdvisor

#### 步骤：
1. **在 `docker01` 上启动 cAdvisor**：
   ```sh
   docker run -d \
     --name=cadvisor \
     --volume=/:/rootfs:ro \
     --volume=/var/run:/var/run:rw \
     --volume=/sys:/sys:ro \
     --volume=/var/lib/docker/:/var/lib/docker:ro \
     --publish=9323:8080 \
     google/cadvisor:latest
   ```

2. **在 `docker02` 上启动 cAdvisor**：
   ```sh
   docker run -d \
     --name=cadvisor \
     --volume=/:/rootfs:ro \
     --volume=/var/run:/var/run:rw \
     --volume=/sys:/sys:ro \
     --volume=/var/lib/docker/:/var/lib/docker:ro \
     --publish=9323:8080 \
     google/cadvisor:latest
   ```

### 3. 验证配置

#### 步骤：
1. **检查 cAdvisor**：
   在 `docker01` 和 `docker02` 上访问 `http://192.168.88.129:9323` 和 `http://192.168.88.128:9323`，确保 cAdvisor 正常工作并返回监控数据。

2. **访问 Prometheus**：
   在浏览器中访问 `http://192.168.88.129:9090`，确保 Prometheus 正常运行。

3. **查看目标状态**：
   在 Prometheus 的 Web 界面中，导航到 `Status` -> `Targets`，查看 `docker01-containers` 和 `docker02-containers` 是否被成功监控。

### 完整的 Prometheus 配置示例 (`prometheus.yml`)

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'docker01-containers'
    static_configs:
      - targets: ['192.168.88.129:9323']
  - job_name: 'docker02-containers'
    static_configs:
      - targets: ['192.168.88.128:9323']
```

这样，你就可以在 `docker01` 上部署 Prometheus 并监控 `docker01` 和 `docker02` 上的 Docker 容器了。如果有任何问题或需要进一步的帮助，请告诉我。

使用 Grafana 来可视化 Prometheus 收集的数据是一个常见的组合。以下是如何在 `docker01` 上设置和使用 Grafana 的步骤。

### 1. 安装并启动 Grafana

首先，你需要在 `docker01` 上安装并启动 Grafana 容器。



#### 启动 Grafana 容器：

```sh
docker run -d \
  -p 3000:3000 \
  --name=grafana \
  -v /root/grafana/grafana.ini:/etc/grafana/grafana.ini \
  -v /root/grafana/defaults.ini:/usr/share/grafana/conf/defaults.ini \
  grafana/grafana
```

### 2. 访问 Grafana Web 界面

在浏览器中访问 `http://<docker01_ip>:3000`，你应该会看到 Grafana 登录页面。

#### 默认登录信息：

- 用户名: `admin`
- 密码: `admin`

第一次登录时，Grafana 会提示你更改密码。

### 3. 添加 Prometheus 数据源

#### 步骤：

1. 登录到 Grafana 后，点击左侧边栏的 **齿轮图标**（Configuration），然后选择 **Data Sources**。
2. 点击 **Add data source** 按钮。
3. 在搜索框中输入 "Prometheus"，然后选择 **Prometheus**。
4. 在 **HTTP** 部分，配置以下信息：
   - **URL**: `http://<docker01_ip>:9090`（这是 Prometheus 的地址）
5. 向下滚动并点击 **Save & Test**，确保连接成功。

### 4. 创建仪表盘

#### 步骤：

1. 在 Grafana 左侧边栏，点击 **+**（加号图标），然后选择 **Dashboard**。
2. 点击 **Add new panel**。
3. 在 **Query** 部分，选择你刚刚添加的 Prometheus 数据源。
4. 在查询框中输入一个 Prometheus 查询，例如：

   ```promql
   node_cpu_seconds_total{instance="192.168.0.101:9323", mode="idle"}
   ```

   这将显示 `docker01` 上的 CPU 使用情况。你可以根据你的需求输入其他查询。

5. 配置图表的显示设置，然后点击 **Apply**。
6. 重复以上步骤，添加更多的面板和查询，创建完整的仪表盘。

### 5. 保存仪表盘

当你配置好仪表盘后，点击右上角的 **Save** 图标（保存图标），为你的仪表盘命名，并保存。

### 6. 设置告警（可选）

你还可以在 Grafana 中设置告警，以便在数据超出阈值时收到通知。

#### 步骤：

1. 在面板的编辑模式下，点击 **Alert** 选项卡。
2. 配置告警条件和阈值。
3. 配置通知渠道（如电子邮件、Slack 等）。
4. 点击 **Save**。

### 总结

通过以上步骤，你可以成功设置 Grafana 来可视化和监控 Prometheus 收集的数据。Grafana 提供了丰富的可视化选项和告警机制，使你能够更好地了解和监控你的系统。如果你有任何问题或需要进一步的帮助，请告诉我。