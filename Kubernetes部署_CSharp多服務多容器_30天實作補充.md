# Kubernetes 部署：C# 多服務多容器 30 天實作補充

## 來源與定位

- 主線來源：30 天學習 Docker 部署你的專案：https://ithelp.ithome.com.tw/m/users/20151035/ironman/6311
- 延伸來源：Docker 部署專案：C# / ASP.NET Core + GitHub Actions + GHCR 30 天實作補充
- Kubernetes 官方 Deployment 文件：https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- Kubernetes 官方 Service 文件：https://kubernetes.io/docs/concepts/services-networking/service/
- Kubernetes 官方 Sidecar Containers 文件：https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/
- Kubernetes 官方 Liveness / Readiness Probes 文件：https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

> 這份筆記不修改前兩篇 Docker / C# GHCR 筆記，而是把「已經能 build/push image」的主線往 Kubernetes 延伸。範例是最小多服務、多容器：`orders-api`、`orders-worker`、`redis`，其中 `orders-api` Pod 還有一個 `access-log-sidecar` container 透過 shared volume 讀取 API access log，示範多容器 Pod。

## 這份補充要解決什麼

Docker Compose 可以幫你在一台機器上啟動多個 container；Kubernetes 則是讓你宣告「我希望整組服務長什麼樣子」，再由控制平面維持 Deployment、Pod、Service、Probe、ConfigMap 的狀態。這份筆記用 C# 最小專案把那條路線拆開：先有 image，再有 K8S manifest，最後知道如何觀察與排錯。

## 驗證提醒

本機已驗證 `Orders.Api` 與 `Orders.Worker` 都能 `dotnet build`。本環境沒有 `kubectl`，所以 K8S 部分做了 YAML 結構檢查，沒有實際 apply 到 cluster；筆記中會清楚標示需要 Minikube、kind、Docker Desktop Kubernetes 或雲端 K8S cluster 才能實跑。

## 主線專案

### 專案最終會長成什麼

一個最小訂單系統：API 接 HTTP request，Worker 模擬背景處理，Redis 作為基礎設施服務。K8S 中會有三個 Deployment、兩個 Service、一個 Namespace、一個 ConfigMap；API Deployment 的 Pod 內有兩個 containers，sidecar 會透過 shared `emptyDir` 讀 API 寫出的 access log。

### 需要的檔案地圖

| 檔案 | 負責的事情 |
| --- | --- |
| `src/Orders.Api/Program.cs` | API 服務，提供 `/health` 與 `/orders/{id}` |
| `src/Orders.Worker/Worker.cs` | 背景服務，輸出 heartbeat log |
| `Dockerfile.api` | 建置 API image |
| `Dockerfile.worker` | 建置 Worker image |
| `docker-compose.yml` | 本機多服務對照組 |
| `.github/workflows/k8s-demo-images.yml` | GitHub Actions build/push 兩個 image 到 GHCR |
| `k8s/orders-demo.yaml` | K8S Namespace、ConfigMap、Deployments、Services |

### 30 天交付物地圖

| Day | 交付物 |
| --- | --- |
| 1 | 理解 Compose 與 K8S 的責任差異，知道 K8S 不是另一個 docker run |
| 2 | 建立可被 probe 與 Service 驗證的 ASP.NET Core API |
| 3 | 建立不對外開 HTTP、只跑背景工作的第二個 C# 服務 |
| 4 | 加入一個現成基礎設施容器，讓範例不只是一個 API |
| 5 | 用 api、worker、redis 三個 services 先在本機理解依賴 |
| 6 | 把 ASP.NET Core API 打包成可被 K8S pull 的 image |
| 7 | 把背景服務打包成獨立 image |
| 8 | 用 matrix 一次 build/push orders-api 與 orders-worker |
| 9 | 把教學資源集中在 orders-demo namespace，方便清理與觀察 |
| 10 | 把 REDIS_HOST、WORKER_SERVICE_NAME、ASPNETCORE_HTTP_PORTS 從 image 拆出來 |
| 11 | 用 Deployment 宣告 Pod desired state，而不是手動建立 container |
| 12 | 讓 Service 找得到正確 Pod，避免流量送不到容器 |
| 13 | 在 orders-api Pod 放 api container 與 sidecar container |
| 14 | 讓 Pod 準備好才接流量 |
| 15 | 讓卡死的 API container 可以被重啟 |
| 16 | 讓 Redis 用穩定 DNS name 被 API 與 Worker 找到 |
| 17 | 用最小方式把 API 暴露到本機或測試 cluster 外部 |
| 18 | 把背景服務獨立成 Deployment，和 API 分開 scale / restart |
| 19 | 用最小 Redis Deployment 示範 stateful 元件的入門版 |
| 20 | 理解 K8S 從 registry 拉 image，不是從 GitHub repo 跑 source code |
| 21 | 知道 private GHCR image 需要 pull secret，但不要把 token 寫進 YAML |
| 22 | 知道 apply 是提交 desired state，rollout 是觀察 Deployment 更新 |
| 23 | 用 get 看狀態，用 describe 看事件與 selector |
| 24 | 從 api、sidecar、worker 三種 container 看不同 log |
| 25 | 進 container 檢查環境變數與 DNS，但不要在裡面手改系統 |
| 26 | 沒有 NodePort 或 Ingress 時，用 port-forward 做本機驗證 |
| 27 | 用 image tag 變更觸發 rollout，失敗時 rollback |
| 28 | 示範 requests/limits 與 non-root image 的責任邊界 |
| 29 | 學會刪 namespace 清掉整組練習資源 |
| 30 | 把 Docker/GHCR 主線接成 K8S 多服務多容器部署流程 |

### 主線端到端流程

C# API / Worker -> Dockerfile.api / Dockerfile.worker -> GitHub Actions build/push GHCR images -> K8S Deployment 建 Pod -> Service 對 Pod 做穩定入口 -> Probe 判斷健康狀態 -> kubectl logs / describe / rollout 排查。

### 主線做完後檢查

- `Orders.Api` 與 `Orders.Worker` 都能 build。
- K8S YAML 至少包含 Namespace、ConfigMap、3 個 Deployment、2 個 Service。
- Service selector 對得上 Pod labels，且 Worker 沒有宣告不存在的 HTTP Service。
- `orders-api` Pod 至少有 `api` 與 `access-log-sidecar` 兩個 containers，並共享 `emptyDir` log volume。
- 筆記沒有硬編真實 GHCR token 或 cluster credential。

## 小專案完整範例

### `src/Orders.Api/Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

var redisHost = Environment.GetEnvironmentVariable("REDIS_HOST") ?? "redis";
var workerServiceName = Environment.GetEnvironmentVariable("WORKER_SERVICE_NAME") ?? "orders-worker";
var accessLogPath = Environment.GetEnvironmentVariable("ACCESS_LOG_PATH") ?? "/var/log/orders/access.log";

app.MapGet("/health", () => Results.Ok(new
{
    Status = "ok",
    Service = "orders-api",
    RedisHost = redisHost,
    WorkerServiceName = workerServiceName,
    Time = DateTimeOffset.UtcNow
}));

app.MapGet("/orders/{id:int}", (int id) =>
{
    if (id <= 0)
    {
        return Results.BadRequest(new { Error = "Order id must be greater than 0." });
    }

    Directory.CreateDirectory(Path.GetDirectoryName(accessLogPath)!);
    File.AppendAllText(accessLogPath, $"{DateTimeOffset.UtcNow:o} order {id} received{Environment.NewLine}");

    return Results.Ok(new
    {
        Id = id,
        Status = "received",
        NextStep = "worker will process shipping event",
        RedisHost = redisHost,
        CheckedAt = DateTimeOffset.UtcNow
    });
});

app.Run();
```

### `src/Orders.Worker/Worker.cs`

```csharp
namespace Orders.Worker;

public sealed class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;
    private readonly string _redisHost;

    public Worker(ILogger<Worker> logger)
    {
        _logger = logger;
        _redisHost = Environment.GetEnvironmentVariable("REDIS_HOST") ?? "redis";
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Orders worker heartbeat. RedisHost={RedisHost}; Time={Time}", _redisHost, DateTimeOffset.UtcNow);
            await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);
        }
    }
}
```

### `Dockerfile.api`

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY src/Orders.Api/Orders.Api.csproj src/Orders.Api/
RUN dotnet restore src/Orders.Api/Orders.Api.csproj
COPY src/Orders.Api/ src/Orders.Api/
RUN dotnet publish src/Orders.Api/Orders.Api.csproj -c Release -o /app/publish --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
USER app
EXPOSE 8080
ENTRYPOINT ["dotnet", "Orders.Api.dll"]
```

### `Dockerfile.worker`

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY src/Orders.Worker/Orders.Worker.csproj src/Orders.Worker/
RUN dotnet restore src/Orders.Worker/Orders.Worker.csproj
COPY src/Orders.Worker/ src/Orders.Worker/
RUN dotnet publish src/Orders.Worker/Orders.Worker.csproj -c Release -o /app/publish --no-restore

FROM mcr.microsoft.com/dotnet/runtime:10.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
USER app
ENTRYPOINT ["dotnet", "Orders.Worker.dll"]
```

### `docker-compose.yml`

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    image: orders-api:local
    ports:
      - "8080:8080"
    environment:
      ASPNETCORE_HTTP_PORTS: "8080"
      REDIS_HOST: redis
      WORKER_SERVICE_NAME: orders-worker

  worker:
    build:
      context: .
      dockerfile: Dockerfile.worker
    image: orders-worker:local
    environment:
      REDIS_HOST: redis

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

### `.github/workflows/k8s-demo-images.yml`

```yaml
name: Build K8S demo images

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: read
  packages: write

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service:
          - name: orders-api
            dockerfile: Dockerfile.api
          - name: orders-worker
            dockerfile: Dockerfile.worker

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./${{ matrix.service.dockerfile }}
          push: true
          tags: |
            ghcr.io/${{ github.repository_owner }}/${{ matrix.service.name }}:latest
            ghcr.io/${{ github.repository_owner }}/${{ matrix.service.name }}:${{ github.sha }}
          labels: |
            org.opencontainers.image.source=${{ github.server_url }}/${{ github.repository }}
```

### `k8s/orders-demo.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: orders-demo
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: orders-config
  namespace: orders-demo
data:
  REDIS_HOST: redis
  WORKER_SERVICE_NAME: orders-worker
  ASPNETCORE_HTTP_PORTS: "8080"
  ACCESS_LOG_PATH: /var/log/orders/access.log
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-api
  namespace: orders-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: orders-api
  template:
    metadata:
      labels:
        app: orders-api
    spec:
      containers:
        - name: api
          image: ghcr.io/YOUR_GITHUB_ACCOUNT/orders-api:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: orders-config
          volumeMounts:
            - name: access-logs
              mountPath: /var/log/orders
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
        - name: access-log-sidecar
          image: busybox:1.36
          command: ["sh", "-c", "touch /var/log/orders/access.log && tail -f /var/log/orders/access.log"]
          volumeMounts:
            - name: access-logs
              mountPath: /var/log/orders
      volumes:
        - name: access-logs
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: orders-api
  namespace: orders-demo
spec:
  type: NodePort
  selector:
    app: orders-api
  ports:
    - name: http
      port: 80
      targetPort: 8080
      nodePort: 30080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-worker
  namespace: orders-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: orders-worker
  template:
    metadata:
      labels:
        app: orders-worker
    spec:
      containers:
        - name: worker
          image: ghcr.io/YOUR_GITHUB_ACCOUNT/orders-worker:latest
          imagePullPolicy: Always
          envFrom:
            - configMapRef:
                name: orders-config
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: orders-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7-alpine
          ports:
            - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: orders-demo
spec:
  type: ClusterIP
  selector:
    app: redis
  ports:
    - name: redis
      port: 6379
      targetPort: 6379
```

## Day 1 - 從 Docker Compose 走到 Kubernetes

### 這篇文章主要在講什麼

這一天聚焦在「從 Docker Compose 走到 Kubernetes」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：理解 Compose 與 K8S 的責任差異，知道 K8S 不是另一個 docker run。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 從 Docker Compose 走到 Kubernetes，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `docker-compose.yml、k8s/orders-demo.yaml` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`docker-compose.yml、k8s/orders-demo.yaml`。
- 資料與控制流：需求 -> 多容器本機組合 -> K8S objects -> cluster desired state。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「從 Docker Compose 走到 Kubernetes」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`docker-compose.yml、k8s/orders-demo.yaml`。
- 資料怎麼流：需求 -> 多容器本機組合 -> K8S objects -> cluster desired state。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `docker-compose.yml` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
docker compose up --build
# 對照：kubectl apply -f k8s/orders-demo.yaml
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「從 Docker Compose 走到 Kubernetes」段落。

具體流程：
1. 打開或建立 `docker-compose.yml`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「從 Docker Compose 走到 Kubernetes」。
2. 修改或檢查 `docker-compose.yml、k8s/orders-demo.yaml`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 1 的重點是：用「從 Docker Compose 走到 Kubernetes」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 2 - 建立 Orders.Api 最小服務

### 這篇文章主要在講什麼

這一天聚焦在「建立 Orders.Api 最小服務」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：建立可被 probe 與 Service 驗證的 ASP.NET Core API。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 建立 Orders.Api 最小服務，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `src/Orders.Api/Program.cs` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`src/Orders.Api/Program.cs`。
- 資料與控制流：HTTP request -> Service -> Pod -> api container -> JSON response。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「建立 Orders.Api 最小服務」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`src/Orders.Api/Program.cs`。
- 資料怎麼流：HTTP request -> Service -> Pod -> api container -> JSON response。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `src/Orders.Api/Program.cs` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
dotnet run --project src/Orders.Api/Orders.Api.csproj --urls http://localhost:5000
curl http://localhost:5000/health
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「建立 Orders.Api 最小服務」段落。

具體流程：
1. 打開或建立 `src/Orders.Api/Program.cs`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「建立 Orders.Api 最小服務」。
2. 修改或檢查 `src/Orders.Api/Program.cs`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 2 的重點是：用「建立 Orders.Api 最小服務」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 3 - 建立 Orders.Worker 背景服務

### 這篇文章主要在講什麼

這一天聚焦在「建立 Orders.Worker 背景服務」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：建立不對外開 HTTP、只跑背景工作的第二個 C# 服務。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 建立 Orders.Worker 背景服務，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `src/Orders.Worker/Worker.cs` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`src/Orders.Worker/Worker.cs`。
- 資料與控制流：Pod start -> worker container -> log heartbeat。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「建立 Orders.Worker 背景服務」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`src/Orders.Worker/Worker.cs`。
- 資料怎麼流：Pod start -> worker container -> log heartbeat。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `src/Orders.Worker/Worker.cs` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
dotnet run --project src/Orders.Worker/Orders.Worker.csproj
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「建立 Orders.Worker 背景服務」段落。

具體流程：
1. 打開或建立 `src/Orders.Worker/Worker.cs`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「建立 Orders.Worker 背景服務」。
2. 修改或檢查 `src/Orders.Worker/Worker.cs`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 3 的重點是：用「建立 Orders.Worker 背景服務」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 4 - 用 Redis 當第三個服務

### 這篇文章主要在講什麼

這一天聚焦在「用 Redis 當第三個服務」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：加入一個現成基礎設施容器，讓範例不只是一個 API。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 用 Redis 當第三個服務，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `redis:7-alpine image、redis Service` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`redis:7-alpine image、redis Service`。
- 資料與控制流：worker/api env -> redis DNS name -> redis Pod。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「用 Redis 當第三個服務」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`redis:7-alpine image、redis Service`。
- 資料怎麼流：worker/api env -> redis DNS name -> redis Pod。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `redis:7-alpine image` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
docker run --rm -p 6379:6379 redis:7-alpine
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「用 Redis 當第三個服務」段落。

具體流程：
1. 打開或建立 `redis:7-alpine image`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「用 Redis 當第三個服務」。
2. 修改或檢查 `redis:7-alpine image、redis Service`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 4 的重點是：用「用 Redis 當第三個服務」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 5 - 先用 Compose 對齊多服務心智模型

### 這篇文章主要在講什麼

這一天聚焦在「先用 Compose 對齊多服務心智模型」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：用 api、worker、redis 三個 services 先在本機理解依賴。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 先用 Compose 對齊多服務心智模型，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `docker-compose.yml` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`docker-compose.yml`。
- 資料與控制流：compose up -> api container + worker container + redis container。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「先用 Compose 對齊多服務心智模型」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`docker-compose.yml`。
- 資料怎麼流：compose up -> api container + worker container + redis container。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `docker-compose.yml` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
docker compose up --build
curl http://localhost:8080/orders/1
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「先用 Compose 對齊多服務心智模型」段落。

具體流程：
1. 打開或建立 `docker-compose.yml`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「先用 Compose 對齊多服務心智模型」。
2. 修改或檢查 `docker-compose.yml`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 5 的重點是：用「先用 Compose 對齊多服務心智模型」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 6 - 替 API 寫 Dockerfile

### 這篇文章主要在講什麼

這一天聚焦在「替 API 寫 Dockerfile」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：把 ASP.NET Core API 打包成可被 K8S pull 的 image。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 替 API 寫 Dockerfile，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Dockerfile.api` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Dockerfile.api`。
- 資料與控制流：source -> dotnet publish -> aspnet runtime image。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「替 API 寫 Dockerfile」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Dockerfile.api`。
- 資料怎麼流：source -> dotnet publish -> aspnet runtime image。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Dockerfile.api` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
docker build -f Dockerfile.api -t orders-api:local .
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「替 API 寫 Dockerfile」段落。

具體流程：
1. 打開或建立 `Dockerfile.api`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「替 API 寫 Dockerfile」。
2. 修改或檢查 `Dockerfile.api`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 6 的重點是：用「替 API 寫 Dockerfile」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 7 - 替 Worker 寫 Dockerfile

### 這篇文章主要在講什麼

這一天聚焦在「替 Worker 寫 Dockerfile」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：把背景服務打包成獨立 image。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 替 Worker 寫 Dockerfile，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Dockerfile.worker` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Dockerfile.worker`。
- 資料與控制流：source -> dotnet publish -> runtime image。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「替 Worker 寫 Dockerfile」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Dockerfile.worker`。
- 資料怎麼流：source -> dotnet publish -> runtime image。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Dockerfile.worker` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
docker build -f Dockerfile.worker -t orders-worker:local .
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「替 Worker 寫 Dockerfile」段落。

具體流程：
1. 打開或建立 `Dockerfile.worker`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「替 Worker 寫 Dockerfile」。
2. 修改或檢查 `Dockerfile.worker`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 7 的重點是：用「替 Worker 寫 Dockerfile」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 8 - 用 GitHub Actions 建兩個 image

### 這篇文章主要在講什麼

這一天聚焦在「用 GitHub Actions 建兩個 image」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：用 matrix 一次 build/push orders-api 與 orders-worker。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 用 GitHub Actions 建兩個 image，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `.github/workflows/k8s-demo-images.yml` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`.github/workflows/k8s-demo-images.yml`。
- 資料與控制流：git push -> Actions matrix -> two Docker builds -> GHCR。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「用 GitHub Actions 建兩個 image」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`.github/workflows/k8s-demo-images.yml`。
- 資料怎麼流：git push -> Actions matrix -> two Docker builds -> GHCR。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `.github/workflows/k8s-demo-images.yml` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
git push origin main
# GitHub Actions -> Build K8S demo images
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「用 GitHub Actions 建兩個 image」段落。

具體流程：
1. 打開或建立 `.github/workflows/k8s-demo-images.yml`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「用 GitHub Actions 建兩個 image」。
2. 修改或檢查 `.github/workflows/k8s-demo-images.yml`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 8 的重點是：用「用 GitHub Actions 建兩個 image」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 9 - Namespace

### 這篇文章主要在講什麼

這一天聚焦在「Namespace」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：把教學資源集中在 orders-demo namespace，方便清理與觀察。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Namespace，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Namespace/orders-demo` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Namespace/orders-demo`。
- 資料與控制流：kubectl apply -> namespace -> namespaced resources。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Namespace」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Namespace/orders-demo`。
- 資料怎麼流：kubectl apply -> namespace -> namespaced resources。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Namespace/orders-demo` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl apply -f k8s/orders-demo.yaml
kubectl get ns orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Namespace」段落。

具體流程：
1. 打開或建立 `Namespace/orders-demo`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Namespace」。
2. 修改或檢查 `Namespace/orders-demo`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 9 的重點是：用「Namespace」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 10 - ConfigMap

### 這篇文章主要在講什麼

這一天聚焦在「ConfigMap」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：把 REDIS_HOST、WORKER_SERVICE_NAME、ASPNETCORE_HTTP_PORTS 從 image 拆出來。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 ConfigMap，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `ConfigMap/orders-config` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`ConfigMap/orders-config`。
- 資料與控制流：ConfigMap -> envFrom -> container environment。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「ConfigMap」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`ConfigMap/orders-config`。
- 資料怎麼流：ConfigMap -> envFrom -> container environment。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `ConfigMap/orders-config` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get configmap orders-config -n orders-demo -o yaml
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「ConfigMap」段落。

具體流程：
1. 打開或建立 `ConfigMap/orders-config`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「ConfigMap」。
2. 修改或檢查 `ConfigMap/orders-config`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 10 的重點是：用「ConfigMap」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 11 - Deployment 基礎

### 這篇文章主要在講什麼

這一天聚焦在「Deployment 基礎」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：用 Deployment 宣告 Pod desired state，而不是手動建立 container。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Deployment 基礎，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Deployment/orders-api` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Deployment/orders-api`。
- 資料與控制流：Deployment -> ReplicaSet -> Pod。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Deployment 基礎」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Deployment/orders-api`。
- 資料怎麼流：Deployment -> ReplicaSet -> Pod。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Deployment/orders-api` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get deploy orders-api -n orders-demo
kubectl get pods -n orders-demo -l app=orders-api
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Deployment 基礎」段落。

具體流程：
1. 打開或建立 `Deployment/orders-api`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Deployment 基礎」。
2. 修改或檢查 `Deployment/orders-api`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 11 的重點是：用「Deployment 基礎」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 12 - Labels 與 Selectors

### 這篇文章主要在講什麼

這一天聚焦在「Labels 與 Selectors」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：讓 Service 找得到正確 Pod，避免流量送不到容器。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Labels 與 Selectors，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `metadata.labels、selector.matchLabels` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`metadata.labels、selector.matchLabels`。
- 資料與控制流：Service selector -> Pod labels -> endpoints。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Labels 與 Selectors」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`metadata.labels、selector.matchLabels`。
- 資料怎麼流：Service selector -> Pod labels -> endpoints。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `metadata.labels` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get endpoints orders-api -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Labels 與 Selectors」段落。

具體流程：
1. 打開或建立 `metadata.labels`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Labels 與 Selectors」。
2. 修改或檢查 `metadata.labels、selector.matchLabels`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 12 的重點是：用「Labels 與 Selectors」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 13 - 多容器 Pod 與 Sidecar

### 這篇文章主要在講什麼

這一天聚焦在「多容器 Pod 與 Sidecar」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：在 orders-api Pod 放 api container 與 sidecar container。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 多容器 Pod 與 Sidecar，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `containers: api + access-log-sidecar` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`containers: api + access-log-sidecar`。
- 資料與控制流：one Pod -> shared lifecycle -> two containers。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「多容器 Pod 與 Sidecar」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`containers: api + access-log-sidecar`。
- 資料怎麼流：one Pod -> shared lifecycle -> two containers。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `containers: api + access-log-sidecar` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get pod -n orders-demo -l app=orders-api -o jsonpath="{.items[0].spec.containers[*].name}"
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「多容器 Pod 與 Sidecar」段落。

具體流程：
1. 打開或建立 `containers: api + access-log-sidecar`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「多容器 Pod 與 Sidecar」。
2. 修改或檢查 `containers: api + access-log-sidecar`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 13 的重點是：用「多容器 Pod 與 Sidecar」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 14 - Readiness Probe

### 這篇文章主要在講什麼

這一天聚焦在「Readiness Probe」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：讓 Pod 準備好才接流量。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Readiness Probe，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `readinessProbe /health` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`readinessProbe /health`。
- 資料與控制流：kubelet -> HTTP /health -> ready endpoint。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Readiness Probe」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`readinessProbe /health`。
- 資料怎麼流：kubelet -> HTTP /health -> ready endpoint。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `readinessProbe /health` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl describe pod -n orders-demo -l app=orders-api
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Readiness Probe」段落。

具體流程：
1. 打開或建立 `readinessProbe /health`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Readiness Probe」。
2. 修改或檢查 `readinessProbe /health`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 14 的重點是：用「Readiness Probe」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 15 - Liveness Probe

### 這篇文章主要在講什麼

這一天聚焦在「Liveness Probe」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：讓卡死的 API container 可以被重啟。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Liveness Probe，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `livenessProbe /health` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`livenessProbe /health`。
- 資料與控制流：kubelet -> failed health check -> restart container。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Liveness Probe」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`livenessProbe /health`。
- 資料怎麼流：kubelet -> failed health check -> restart container。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `livenessProbe /health` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl describe pod -n orders-demo -l app=orders-api
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Liveness Probe」段落。

具體流程：
1. 打開或建立 `livenessProbe /health`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Liveness Probe」。
2. 修改或檢查 `livenessProbe /health`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 15 的重點是：用「Liveness Probe」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 16 - Service ClusterIP

### 這篇文章主要在講什麼

這一天聚焦在「Service ClusterIP」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：讓 Redis 用穩定 DNS name 被 API 與 Worker 找到。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Service ClusterIP，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Service/redis` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Service/redis`。
- 資料與控制流：redis service name -> cluster IP -> selected Redis Pod。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Service ClusterIP」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Service/redis`。
- 資料怎麼流：redis service name -> cluster IP -> selected Redis Pod。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Service/redis` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get svc redis -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Service ClusterIP」段落。

具體流程：
1. 打開或建立 `Service/redis`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Service ClusterIP」。
2. 修改或檢查 `Service/redis`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 16 的重點是：用「Service ClusterIP」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 17 - Service NodePort

### 這篇文章主要在講什麼

這一天聚焦在「Service NodePort」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：用最小方式把 API 暴露到本機或測試 cluster 外部。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Service NodePort，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Service/orders-api type NodePort` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Service/orders-api type NodePort`。
- 資料與控制流：nodeIP:30080 -> Service -> targetPort 8080。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Service NodePort」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Service/orders-api type NodePort`。
- 資料怎麼流：nodeIP:30080 -> Service -> targetPort 8080。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Service/orders-api type NodePort` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get svc orders-api -n orders-demo
curl http://localhost:30080/health
# Minikube/kind 若 localhost 不通，改用 port-forward：
# kubectl port-forward service/orders-api -n orders-demo 8080:80
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Service NodePort」段落。

具體流程：
1. 打開或建立 `Service/orders-api type NodePort`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Service NodePort」。
2. 修改或檢查 `Service/orders-api type NodePort`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 17 的重點是：用「Service NodePort」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 18 - Worker Deployment

### 這篇文章主要在講什麼

這一天聚焦在「Worker Deployment」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：把背景服務獨立成 Deployment，和 API 分開 scale / restart。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Worker Deployment，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Deployment/orders-worker` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Deployment/orders-worker`。
- 資料與控制流：Deployment -> worker Pod -> log heartbeat。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Worker Deployment」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Deployment/orders-worker`。
- 資料怎麼流：Deployment -> worker Pod -> log heartbeat。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Deployment/orders-worker` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl logs deploy/orders-worker -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Worker Deployment」段落。

具體流程：
1. 打開或建立 `Deployment/orders-worker`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Worker Deployment」。
2. 修改或檢查 `Deployment/orders-worker`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 18 的重點是：用「Worker Deployment」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 19 - Redis Deployment

### 這篇文章主要在講什麼

這一天聚焦在「Redis Deployment」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：用最小 Redis Deployment 示範 stateful 元件的入門版。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Redis Deployment，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `Deployment/redis` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`Deployment/redis`。
- 資料與控制流：Deployment -> redis Pod -> redis Service。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Redis Deployment」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`Deployment/redis`。
- 資料怎麼流：Deployment -> redis Pod -> redis Service。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `Deployment/redis` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get deploy redis -n orders-demo
kubectl get svc redis -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Redis Deployment」段落。

具體流程：
1. 打開或建立 `Deployment/redis`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Redis Deployment」。
2. 修改或檢查 `Deployment/redis`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 19 的重點是：用「Redis Deployment」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 20 - Image Pull 與 GHCR

### 這篇文章主要在講什麼

這一天聚焦在「Image Pull 與 GHCR」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：理解 K8S 從 registry 拉 image，不是從 GitHub repo 跑 source code。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Image Pull 與 GHCR，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `image: ghcr.io/...` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`image: ghcr.io/...`。
- 資料與控制流：node kubelet -> container runtime -> GHCR image。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Image Pull 與 GHCR」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`image: ghcr.io/...`。
- 資料怎麼流：node kubelet -> container runtime -> GHCR image。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `image: ghcr.io/...` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl describe pod -n orders-demo -l app=orders-api | findstr Image
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Image Pull 與 GHCR」段落。

具體流程：
1. 打開或建立 `image: ghcr.io/...`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Image Pull 與 GHCR」。
2. 修改或檢查 `image: ghcr.io/...`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 20 的重點是：用「Image Pull 與 GHCR」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 21 - Secrets 與 imagePullSecrets 觀念

### 這篇文章主要在講什麼

這一天聚焦在「Secrets 與 imagePullSecrets 觀念」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：知道 private GHCR image 需要 pull secret，但不要把 token 寫進 YAML。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Secrets 與 imagePullSecrets 觀念，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl create secret docker-registry` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl create secret docker-registry`。
- 資料與控制流：secret -> imagePullSecrets -> kubelet auth。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Secrets 與 imagePullSecrets 觀念」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl create secret docker-registry`。
- 資料怎麼流：secret -> imagePullSecrets -> kubelet auth。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl create secret docker-registry` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl create secret docker-registry ghcr-pull-secret --docker-server=ghcr.io --docker-username=YOUR_ACCOUNT --docker-password=YOUR_TOKEN -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Secrets 與 imagePullSecrets 觀念」段落。

具體流程：
1. 打開或建立 `kubectl create secret docker-registry`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Secrets 與 imagePullSecrets 觀念」。
2. 修改或檢查 `kubectl create secret docker-registry`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 21 的重點是：用「Secrets 與 imagePullSecrets 觀念」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 22 - kubectl apply 與 rollout

### 這篇文章主要在講什麼

這一天聚焦在「kubectl apply 與 rollout」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：知道 apply 是提交 desired state，rollout 是觀察 Deployment 更新。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 kubectl apply 與 rollout，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl apply、kubectl rollout status` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl apply、kubectl rollout status`。
- 資料與控制流：manifest -> API server -> controller -> pods updated。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「kubectl apply 與 rollout」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl apply、kubectl rollout status`。
- 資料怎麼流：manifest -> API server -> controller -> pods updated。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl apply` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl apply -f k8s/orders-demo.yaml
kubectl rollout status deployment/orders-api -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「kubectl apply 與 rollout」段落。

具體流程：
1. 打開或建立 `kubectl apply`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「kubectl apply 與 rollout」。
2. 修改或檢查 `kubectl apply、kubectl rollout status`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 22 的重點是：用「kubectl apply 與 rollout」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 23 - kubectl get / describe

### 這篇文章主要在講什麼

這一天聚焦在「kubectl get / describe」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：用 get 看狀態，用 describe 看事件與 selector。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 kubectl get / describe，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl get、kubectl describe` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl get、kubectl describe`。
- 資料與控制流：object summary -> events -> troubleshooting。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「kubectl get / describe」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl get、kubectl describe`。
- 資料怎麼流：object summary -> events -> troubleshooting。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl get` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get all -n orders-demo
kubectl describe deployment/orders-api -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「kubectl get / describe」段落。

具體流程：
1. 打開或建立 `kubectl get`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「kubectl get / describe」。
2. 修改或檢查 `kubectl get、kubectl describe`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 23 的重點是：用「kubectl get / describe」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 24 - kubectl logs

### 這篇文章主要在講什麼

這一天聚焦在「kubectl logs」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：從 api、sidecar、worker 三種 container 看不同 log。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 kubectl logs，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl logs deploy/orders-api -c api` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl logs deploy/orders-api -c api`。
- 資料與控制流：Pod -> container log stream -> diagnosis。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「kubectl logs」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl logs deploy/orders-api -c api`。
- 資料怎麼流：Pod -> container log stream -> diagnosis。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl logs deploy/orders-api -c api` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl logs deploy/orders-api -n orders-demo -c api
kubectl logs deploy/orders-api -n orders-demo -c access-log-sidecar
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「kubectl logs」段落。

具體流程：
1. 打開或建立 `kubectl logs deploy/orders-api -c api`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「kubectl logs」。
2. 修改或檢查 `kubectl logs deploy/orders-api -c api`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 24 的重點是：用「kubectl logs」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 25 - kubectl exec

### 這篇文章主要在講什麼

這一天聚焦在「kubectl exec」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：進 container 檢查環境變數與 DNS，但不要在裡面手改系統。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 kubectl exec，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl exec` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl exec`。
- 資料與控制流：operator -> shell -> env/nslookup/curl。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「kubectl exec」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl exec`。
- 資料怎麼流：operator -> shell -> env/nslookup/curl。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl exec` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl exec deploy/orders-api -n orders-demo -c api -- printenv REDIS_HOST
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「kubectl exec」段落。

具體流程：
1. 打開或建立 `kubectl exec`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「kubectl exec」。
2. 修改或檢查 `kubectl exec`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 25 的重點是：用「kubectl exec」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 26 - Port-forward

### 這篇文章主要在講什麼

這一天聚焦在「Port-forward」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：沒有 NodePort 或 Ingress 時，用 port-forward 做本機驗證。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Port-forward，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl port-forward service/orders-api 8080:80` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl port-forward service/orders-api 8080:80`。
- 資料與控制流：localhost -> kubectl tunnel -> Service -> Pod。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Port-forward」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl port-forward service/orders-api 8080:80`。
- 資料怎麼流：localhost -> kubectl tunnel -> Service -> Pod。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl port-forward service/orders-api 8080:80` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl port-forward service/orders-api -n orders-demo 8080:80
curl http://localhost:8080/health
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Port-forward」段落。

具體流程：
1. 打開或建立 `kubectl port-forward service/orders-api 8080:80`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Port-forward」。
2. 修改或檢查 `kubectl port-forward service/orders-api 8080:80`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 26 的重點是：用「Port-forward」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 27 - Rolling Update 與回滾

### 這篇文章主要在講什麼

這一天聚焦在「Rolling Update 與回滾」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：用 image tag 變更觸發 rollout，失敗時 rollback。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 Rolling Update 與回滾，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl set image、rollout undo` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl set image、rollout undo`。
- 資料與控制流：new image -> new ReplicaSet -> traffic shift。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「Rolling Update 與回滾」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl set image、rollout undo`。
- 資料怎麼流：new image -> new ReplicaSet -> traffic shift。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl set image` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl set image deployment/orders-api api=ghcr.io/YOUR_GITHUB_ACCOUNT/orders-api:<sha> -n orders-demo
kubectl rollout undo deployment/orders-api -n orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「Rolling Update 與回滾」段落。

具體流程：
1. 打開或建立 `kubectl set image`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「Rolling Update 與回滾」。
2. 修改或檢查 `kubectl set image、rollout undo`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 27 的重點是：用「Rolling Update 與回滾」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 28 - 資源限制與最小安全線

### 這篇文章主要在講什麼

這一天聚焦在「資源限制與最小安全線」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：示範 requests/limits 與 non-root image 的責任邊界。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 資源限制與最小安全線，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `resources、USER app` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`resources、USER app`。
- 資料與控制流：scheduler -> resource request -> node placement。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「資源限制與最小安全線」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`resources、USER app`。
- 資料怎麼流：scheduler -> resource request -> node placement。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `resources` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
# 在 Deployment container 下加入 resources.requests / resources.limits
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「資源限制與最小安全線」段落。

具體流程：
1. 打開或建立 `resources`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「資源限制與最小安全線」。
2. 修改或檢查 `resources、USER app`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 28 的重點是：用「資源限制與最小安全線」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 29 - 清理與成本意識

### 這篇文章主要在講什麼

這一天聚焦在「清理與成本意識」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：學會刪 namespace 清掉整組練習資源。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 清理與成本意識，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `kubectl delete namespace orders-demo` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`kubectl delete namespace orders-demo`。
- 資料與控制流：namespace deletion -> all namespaced objects cleaned。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「清理與成本意識」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`kubectl delete namespace orders-demo`。
- 資料怎麼流：namespace deletion -> all namespaced objects cleaned。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `kubectl delete namespace orders-demo` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl delete namespace orders-demo
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「清理與成本意識」段落。

具體流程：
1. 打開或建立 `kubectl delete namespace orders-demo`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「清理與成本意識」。
2. 修改或檢查 `kubectl delete namespace orders-demo`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 29 的重點是：用「清理與成本意識」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。

## Day 30 - 端到端復盤

### 這篇文章主要在講什麼

這一天聚焦在「端到端復盤」，把 Docker / GHCR 主線往 Kubernetes 的一個責任層推進。今天的交付物是：把 Docker/GHCR 主線接成 K8S 多服務多容器部署流程。

### 為什麼需要這個概念

K8S 的難點不是 YAML 很長，而是每個 object 有不同責任。若你不知道今天在處理 端到端復盤，就很容易把 image pull、Service selector、Pod health、DNS、container log 全部混成同一個錯誤。

### 學完這篇你應該會做到什麼

你應該能說出 `所有範例檔案` 在整個部署流程中的位置，並能用至少一個 `kubectl` 或本機命令驗證它是否生效。

### 核心重點

- 會動到：`所有範例檔案`。
- 資料與控制流：C# services -> images -> GHCR -> K8S Deployments -> Services for API/Redis -> health check。
- K8S 是 desired state 模型；你提交 manifest，controller 讓實際狀態往目標狀態靠近。

### 真實工作流程例子

- 工作任務：團隊已有 C# Docker image，現在要把訂單系統部署到 Kubernetes，今天負責「端到端復盤」。
- 你先判斷：先分清楚這次是在改 C# 程式、image build、K8S Deployment、Service、ConfigMap、probe、還是觀察指令；不要一口氣改整份 manifest。
- 會動到：`所有範例檔案`。
- 資料怎麼流：C# services -> images -> GHCR -> K8S Deployments -> Services for API/Redis -> health check。
- 流程路線圖：需求 -> 選 K8S object -> 修改 `所有範例檔案` -> apply 或 build -> kubectl / curl / log 驗證。
- 工作中會寫 / 檢查的片段：

```bash
kubectl get all -n orders-demo
curl http://localhost:30080/health
# 或：kubectl port-forward service/orders-api -n orders-demo 8080:80
```

- 交付前驗證：至少看 object 是否存在、selector 是否對到 Pod、log 是否合理；若是對外入口，再用 curl 驗證。
- 常見卡點：看到 Pod 不通時先看 `kubectl describe` events，再看 logs；不要直接重建整個 cluster。

### 主線專案銜接

今天接到主線專案的「端到端復盤」段落。

具體流程：
1. 打開或建立 `所有範例檔案`，確認今天要改哪個 object 或程式檔。
2. 只套用今天相關的最小設定，保留 image account、token、domain 為 placeholder。
3. 用本日命令確認狀態，並記錄成功輸出與錯誤訊息。

具體檢查：
- labels / selectors 是否一致。
- namespace 是否正確。
- command 驗證的 object 是否就是今天改的 object。

### 當天做完後檢查

- 可以說明今天處理的是 Pod、Deployment、Service、ConfigMap、Probe、Registry 還是觀察工具。
- 可以指出一個成功狀態與一個常見失敗狀態。
- 沒有把真實 token、kubeconfig、cluster endpoint 寫進筆記或 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `ConfigMap`、image tag、env、labels | 提供 container 啟動所需設定與識別資訊 |
| 流程或邏輯 | `Deployment`、controller、probe | 維持 Pod 數量、健康與更新流程 |
| 使用端或呈現 | `Service`、`kubectl logs`、`curl /health` | 提供存取入口與排查訊號 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天主題是「端到端復盤」。
2. 修改或檢查 `所有範例檔案`。
3. 如果是程式或 image，先本機 build；如果是 manifest，先檢查 YAML 結構與 selector。
4. 套到 cluster 後，用 `kubectl get`、`describe`、`logs`、`rollout status` 或 `curl` 驗證。
5. 將失敗情境寫回筆記，特別是 image pull、selector、port、probe 的錯誤。

注意事項：
- 最小範例可以用 NodePort；Docker Desktop 常能直接打 `localhost:30080`，Minikube/kind 可能要改用 node IP、`minikube service` 或 `kubectl port-forward`。
- Redis 在這份筆記是教學用 Deployment；正式資料庫或 stateful service 要重新評估 StatefulSet、PVC、備份與 HA。
- `latest` 方便學習；本範例搭配 `imagePullPolicy: Always` 避免重推 latest 後節點沿用快取。正式部署仍應優先使用 commit sha 或版本 tag。

### 如果結果和預期不同

- Pod Pending：查 node resource、image pull secret、namespace。
- Pod Running 但 Service 不通：查 selector、targetPort、containerPort、readiness。
- Rollout 卡住：查 `kubectl describe deployment` 與新 ReplicaSet 的 Pod events。

### 負面例子 / 錯誤用法

錯誤做法：把所有服務塞進同一個 Pod，或 Service selector 隨便寫，然後只靠重新 apply 解決問題。

問題：
- API、Worker、Redis 生命週期不同，全部塞一起會難以 scale 與排錯。
- selector 錯誤時 Service 沒 endpoints，看起來像網路壞掉，其實是 labels 對不上。
- 沒有 readiness probe 時，Pod 還沒準備好也可能收到流量。

修正方向：一個責任一個 Deployment；Service 用明確 selector；health endpoint 要和 probe 對齊。

### 小練習

把今天的主題故意改錯一次，例如改錯 label、port 或 image tag，再用本日命令找出錯誤落在哪一層。

### Junior 常見誤解

常見誤解是把 K8S 當成「比較複雜的 Docker Compose」。K8S 真正重要的是 controller、selector、Service discovery、probe 與 rollout 這些維持狀態的機制。

### 一句話總結

Day 30 的重點是：用「端到端復盤」把 C# 多服務系統往 Kubernetes 可觀察、可更新、可排錯的部署方式推進一步。
